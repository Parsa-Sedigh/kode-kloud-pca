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

## 70-1-Installing Helm Chart
## 70-1-Prometheus Chart Overview
## 70-1-Connecting To Prometheus
## 70-1-Prometheus Configuration
## 70-1-Deploy Demo Application
## 70-1-Additional Scrape Configs
## 70-1-Service Monitors
## 70-1-Adding Rules
## 70-1-Alertmanager Rules
## 70-1-Lab – Kubernetes & Prometheus
## 70-1-Feedback – Prometheus Certified Associate