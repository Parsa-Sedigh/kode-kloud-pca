## 69-1-Introduction
Instead of having the prometheus server running on a separate server, it's better to set up and install prometheus in the 
k8s cluster itself. There are a couple of benefits to this:
1. you wanna deploy your prometheus server as close to your targets as possible. If your targets are gonna be apps running on a
k8s cluster, it's better to have the prometheus server running as close to them as possible(meaning it would be in the cluster)
2. we can make sure of pre-existing k8s infra. So we don't need a separate server or vm to host the prometheus server, we can run it
on the underlying k8s infra

### kube-state-metrics
By default, we are unable to collect k8s-level metrics like pods, deployments or ... , k8s doesn't expose those metrics.
To get access to those metrics, we have to install `kube-state-metrics` container into our k8s cluster and then that container
is gonna responsible for making those metrics available for our prometheus server. So we have to deploy a container to actually
get access to that information.

### node exporter
There are a couple ways of running a node_exporter on every node:
- we can manually go into each node and install and run a node exporter
- we can customize the base image that is running on nodes, so that base image contains the node_exporter program
- better option: since we're using k8s, we can use daemonset which makes it that a pod will run on every single node in the cluster.
So we can set up a pod that contains the node_exporter process and have it automatically run on every node. That way, if we add a
new node to the cluster, we don't have to remember to install node_exporter and run it. K8s daemonset will automatically create a new
pod for that new node that's gonna run the node_exporter process.

### service discovery
With k8s, we're gonna be making use of service discovery in this way:

We access the k8s api to discover the list of all of the targets that we need to scrape which are gonna be:
- all of the k8s components 
- all of the node_exporters that we have installed
- kube-state-metrics

These will all automatically discovered by service discovery. So you don't have to manually go in and enter the /metrics config of
each target in prometheus.yml .

### Kube-prometheus-stack
K8s operator manages the entire life-cycle of your app, it's gonna handle the initialization, configuration and ... . For example
it will help installing prometheus and automatically restarting the prometheus server if we changed the prometheus config and other 
helpful abilities to manage the prometheus instance.

The prometheus helm chart is gonna make use of prometheus-operator. Note that you can run the operators independently outside of
a helm chart.

With prometheus-operator, we get access to custom resources like Prometheus, Alertmanager and ... . 

## 70-2-Installing Helm Chart

## 71-3-Prometheus Chart Overview
```shell
kubectl get all
```
The prometheus helm chart created 2 stateful sets. The `statefulset.apps/prometheus-prometheus-kube-prometheus-prometheus` is
our prometheus server. We have another stateful set that's responsible for running
alert manager `statefulset.apps/alertmanager-prometheus-kube-prometheus-alertmanager`.

It created 3 deployments:
- It creates the grafana deployment.
- it creates `deplyment.apps/prometheus-kube-prometheus-operator`: This is the operator that's gonna manage the lifecycle of our
prometheus instance.

We get one daemon in the k8s cluster named `prometheus-prometheus-node-exporter`. This daemon set is responsible for deploying a
node-exporter pod on every single node in the k8s cluster. A daemon set takes an img and it makes sure every node in the k8s cluster
will have a pod running this img on it.

All of the created services by the helm chart is gonna be ClusterIP, so nothing is exposed to outside of the cluster. To access
the apps, we either have to set up an ingress or modify the services to be a load balancer or just setting up a proxy to play around.

```shell
kubectl describe statefulset prometheus-prometheus-kube-prometheus-prometheus > prometheus.yaml
```

Note: When it comes to modifying the rules.yml, prometheus operator gives us high level abstractions to manipulate the 
rules file with them instead of manually update that file.

## 72-4-Connecting To Prometheus
There are a couple of ways to access to prometheus.

The service created for prometheus server is a clusterIP, so we don't have access to it outside of cluster, only inside of cluster.
We can update the config of that service to be load balancer or NodePort. Or setting up an ingress.

We can do port forwarding for debugging as well:
```shell
kubectl port-forward <pod name> <port we wanna connect to>
```
In this case, we wanna connect to port 9090 because the ClusterIP service of prometheus is listening on that.
To access prometheus server pod from outside of cluster:
```shell
kubectl port-forward pod/prometheus-prometheus-kube-prometheus-prometheus-0 9090
```

## 73-5-Prometheus Configuration
All of these configs are set up by helm chart, so we didn't have to manually define all the relabel configs for getting access
to specific targets that we wanna scrape.

## 74-6-Deploy Demo Application

## 75-7-Additional Scrape Configs
There are two ways of changing prometheus server config. First way is less ideal.
1. values file which allows tweaking the config of the helm chart. Here, we update the `AdditionalScrapeConfig` property. 
Like before, we provide a list of jobs and then run: 
`helm upgrade <release name - here promehteus> <chart name here: prometheus-community...>` -f values.yaml 
2. using service monitors to add new targets to prometheus

We wanna use service monitor, so that we can be more declarative when applying new scrape configs.

### First approach
```shell
# Then look for AdditionalScrapeConfigs
helm show values prometheus-community/kube-prometheus-stack > values.yaml
```

## 76-8-Service Monitors
There are 2 CRDs that we care in this case:
- promethium: prometheuses.monitoring.coreos.com , this is responsible for creating prometheus instance. releaseLabel is defined
here. So the new service monitros that we create, should match the releaseLabel here so that they get automatically registered by prometheus,
which makes it that the services that they point, get scraped by prometheus
- service monitor: servicemonitors.monitoring.coreos.com , so that we can add more targets

Service monitor is a custom resource.

To scrape a target, we know we would have our app, a service and deployment. First we create a ServiceMonitor. A serviceMonitor is gonna
reference a service that is in some place in our app. If we wanna create an app, we have to reference the service that we created for that app,
in the ServiceMonitor.

There are 3 important things in serviceMonitor definition to point it to the service of the app we wanna scrape:
1. in selector.matchLabels we provide a label(key: value pair) of the label that we gave to the service(labels section of service)
2. specify the port name in endpoints.port(the port of the app we wanna scrape)
3. job label of the scrape we wanna be doing. By default the job label here would be the name of the k8s service of the app we wanna scrape.
We can change the value fo this label using `spec.jobLabel`. The value of `spec.jobLabel` should match with a key in labels section of the service.
In the slide, job is the value of jobLabel in ServiceMonitor and a key in labels section of the service.

Notes:
- The endpoint.interval is the interval of scrape.
- endpoints.path by default is /metrics

With this config, prometheus will know it should scrape the app that is behind the service definition.

### Release label
By default, prometheus doesn't know which service monitors to look for, but if you give the service monitor the release label
that is specified in `serviceMonitorSelector`, prometheus will automatically add the service that that service monitor is pointing to,
to the target list to scrape.

## 77-9-Adding Rules
We saw how to add new targets using service monitor. Now how to add rules?

## 78-10-Alertmanager Rules
Adding rules to alert manager in k8s env. 

### Config differences
An alertmanager.yml config that isn't deployed on k8s is a bit different than alertmanager.yml for the custom resource that is 
defined within k8s using the prometheus operator.

`alertmanagerConfigSelector` by default doesn't have any releases to that we can use it in alertmanager.yml so that prometheus pick it
by default. So we need to change the helm chart. Sth like:
```yaml
alertmanagerConfigSelector:
  matchLabels:
    # could be any arbitrary key-value pair
    resource: prometheus
```

Then run:
```shell
helm upgrade prometheus prometheus-community/kube-prometheus-stack -f <path to alert manager config>
```

Then apply the alert manager config with the specified selector that you chose in the previous step(resource: prometheus).
So in rules.yaml for alert manager, we would have:
```yaml
metadata:
  name: ...
  labels:
    resource: prometheus
```

Then to verify it was applied:
```shell
kubectl get alertmanagerconfig

# set up port forwarding
kubectl port-forward service/alertmanager-operated 9093
# Now see if the new rules for alert manager is applied
```

## 79-11-Lab – Kubernetes & Prometheus
## 80-12-Feedback – Prometheus Certified Associate