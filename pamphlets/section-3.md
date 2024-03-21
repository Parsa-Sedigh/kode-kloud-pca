# Section 3 - Prometheus Fundamentals

## 4-1 - Prometheus Use Case 0219
Let's say we have a couple of different data centers scattered across the country and we also have some services running on aws or
a cloud provider. With prometheus we can collect metrics from these services and have them all presented on one dashboard.
Prometheus has builtin dashboard utilities and also builtin alerting.

With alerting, prometheus can track various metrics and when those metrics cross specific threshold, they can generate alerts and send
those alerts to different notifiers.

1. collecting metrics and show them on dashboard
2. alerting: if certain threshold crossed, create an alert and send it
3. for example: we can create a chart from the average file size of uploads and average latency per request and we can use these two to
cross-reference and find out where is the exact filesize point where latency starts to get high. We can set Prometheus to scrape and
collect the average filesize as well as average latency and then we can collect that data and plot it using built in dashboarding and
visualization tools within Prometheus

## 5-2 - Prometheus Basics 0221

## 6-3 - Prometheus Architecture 0830
Prometheus has 3 components in it's server. But overall, there are more components.

Prometheus has 3 main components:
- data retrival worker: responsible for collecting metrics from targets. So it sends HTTP reqs to targets and collect metrics
Once it collects the metrics, it's gonna store them within a time series DB.
- time series DB
- http server: to allow us to retrieve the data that we stored in the DB in order to see the metrics and visualize it. It accepts
PromQL queriesSo we send HTTP Req to this server and in the req, we send a query using promql.

Components outside of Prometheus server:
- exporters: little processes that run on the targets and are responsible for serving up the metrics so that the retrival node is able to
actually poll the metrics. Because by default your targets(apps) don't automatically present the metrics in a consumable format required
by Prometheus. So exporters are responsible for taking internal data and converting it into metrics that Prometheus will understand.
So targets do not send the data(push) to retrival node. However, there are times where you have short-lived jobs that 
are not going to be able to work with the pull-based model of Prometheus and that's where we have the `pushgateway` component of Prometheus.
- pushgateway: anytime you have a short-lived job where we need to be able to push data to prometheus because the job doesn't 
last long enough for Prometheus to be able to scrape it, we send the metrics to pushgateway and then prometheus can query the metrics
from pushgateway like any other target.
- service discovery: prometheus expects you to hard-code all of the targets that it needs to scrape. It needs to know about all the targets.
You can configure this within the configuration file of prometheus. However for dynamic environments like k8s or cloud provider envs where
you have ec2 instances spinning up and down, you wanna be able to dynamically update the list of targets that you wanna scrape and that's
where service discovery comes into play. Service discovery is all about providing a list of targets for prometheus to scrape, so you don't have
to hardcode those values.
- alert manager: prometheus can also be used to generate alerts. However, prometheus doesn't actually send the notifications. It's not 
responsible for sending emails and sms. prometheus is used to triggering alerts. So we need a separate entity called alert manager which 
handles sending sms, email, smtp integration. When an alert gets triggered, prometheus pushes the alert to a process called alert manager
and alert manager is responsible for sending the alert through email or ... .
- prometheus web ui/grafana: we use promql to query the data from the http server

We saw that prometheus supports exporters for some common apps like Mysql, HAProxy and ... . But what if we wanted to collect metrics from
our own **custom apps** or expose custom metrics? Like the number of errors and exceptions and latency of reqs and ... .
We use prometheus client libs.

Prometheus is pull based system. In push based system the server has no idea about the targets.

Prometheus is not meant for collecting events, it's used exclusively for collecting numeric metrics. So it's pull based.

## 7-4 - Prometheus Installation 0529
Promtool: help verify if config is acceptable and everything is gonna work once you start Prometheus.

When you install Prometheus and run `up` query, you see one target in up state. That's because even though we didn't configure any target yet,
Prometheus by default is configured to scrape itself. So it's going to monitor it's own metrics as well.

```shell
brew services start prometheus
```
Now visit localhost:9090

## 8-5 - Prometheus Installation systemd 0550
When we start Prometheus server, it creates some issues:
- it starts the Prometheus process in foreground. So if we close the terminal, we lose the process, Prometheus goes down.
- it doesn't automatically start up on boot. So if we reboot the server, we're gonna have to open a terminal and start Prometheus again

Instead, we want to create a systemd unit for Prometheus, so it automatically starts it on boot and runs it in background.

For this:
1. create a user called prometheus. This user is responsible for running the Prometheus process
With --no-create-home option of useradd command, we make sure the user can't log in and that's because this user is exclusively for
starting the prometheus process. So it's not a normal user that we would use to log in as

In linux, /etc folder is usually for configuration.

In linux, we move the executables to /usr/local/bin .

The actual command to start prometheus after copying files to the mentioned
destination in slides(in order to start prometheus with systemd) is:
```shell
# With -u flag, we change the user that's running the command
sudo -u prometheus /usr/local/bin/prometheus \ ... # (look at the slide for the rest)
```

Then create a systemd unit service file for prometheus service:
```shell
sudo vi /etc/systemd/system/prometheus.service
```
Look at the slide ... .

Since we changed the unit file, we have to do this:
```shell
sudo systemctl daemon-reload
```

To make prometheus start automatically on boot, we also have to run this:
```shell
sudo systemctl enable prometheus
```

## 9-6 - Prometheus Installation Demo 0647
Setting up a unit in systemd to run Prometheus.

```shell
sudo useradd --no-create-home --shell /bin/false prometheus

sudo mkdir /etc/prometheus # to store the configuration file
sudo mkdir /var/lib/prometheus # to store the data

sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus

sudo cp prometheus /usr/local/bin/ # copy the executable
sudo cp promtool /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

sudo cp -r consoles/ etc/prometheus
sudo cp -r console_libraries/ etc/prometheus/

sudo chown -R prometheus:prometheus /etc/prometheus/consoles 
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries

sudo cp prometheus.yml /etc/prometheus
sudo chown prometheus:prometheus /etc/prmetheus/prometheus.yml

# create the service file
sudo vi /etc/systemd/system/prometheus.service
```

prometheus.service:
`
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /etc/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \

[Install]
WantedBy=multi-user.target
`

```shell
sudo systemctl daemon-reload

sudo systemctl start prometheus
systemctl status prometheus

# enable it on startup
sudo systemctl enable prometheus
```

## 10-7 - Node Exporter
Install exporter on one of the targets that we wanna collect metrics on.

This exporter is responsible for collecting metrics on a linux host and presenting it so that prometheus can then scrape it.

To verify it works after running node exporter:
```shell
curl localhost:9100/metrics
```

## 11-8 - Node Exporter systemd
TODO

## 12-9 - Prometheus Configuration
Now that we have exporters like node exporters exposing the metrics, we need to configure Prometheus to be aware of those nodes to scrape
them, remember we follow a pull-based model. We configure it in prometheus.yml .

In prometheus.yml, all config sections gonna inherit the `global` section and in those config sections you can override the global configs.

Job: a collection of instances that need to be scraped.

**When installed with brew, the prometheus.yml is in `opt/homebrew/etc/`.**

```shell
sudo systemctl restart prometheus
```

Prometheus web ui -> status -> targets

So we added a new target for prometheus to scrape which in this case is a linux machine running node_exporter.

## 13-10-Lab – Prometheus Installation

## 14-11 - Authentication-Encryption
### First issue
Without requiring authentication, anybody that has access to the target, can also scrape the metrics exposed by
the exporter of that target. We don't want the metrics to be exposed out in the open like anybdoy can access it.
This is sth authentication solves

### Second issue
When prometheus sends data to and from your target, if somebody is able to capture the packets with a sniffer as it traverses the
network with, they'll be able to read all that data in plain text. So even if we set up auth, if anybody captures the packets, then
they can see all the data. This is solved by encryption. Now if they sniff it, they can't find out what the data is.

### Setting up encryption with TLS
First create certificates. There are a couple of ways of doing this. We can use `openssl` to generate self-signed certs. There is also
lets encrypt and ... .

We're gonna get 2 files: node_export.cert and node_export.key .

Then create config.yml for the node exporter:
```yaml
tls_server_config:
  cert_file: node_export.cert
  key_file: node_export.key
```

Update node exporter process to make the use of the new config.yml file:
```shell
./node_exporter --web.config=config.yml
```

Now when we send req to /metrics of the node, we get tls error. Since we used self-signed certs, if we used certs from lets encrypt,
we wouldn't get this issue. So only because we used self-signed certs, when we send curl, we have to pass `-k` flag to allow an
insecure connection.

### Prometheus TLS Config
We have setup the config on the node exporter to use TLS, we now have to do the same thing on Prometheus, because Prometheus right now is
still gonna send a req not using TLS but still using http, so there's gonna be a mismatch in config.

So copy the generated .cert file of node_exporter to Prometheus server. We can use scp to copy that file over.

Since we're using self-signed certs, we wanna set `insecure_skip_verify: true`. If you got the certs from somewhere else then you wouldn't
have to do this. This setting is equivalent of that -k flag we used on curl.

### Prometheus authentication
When setting up authentication with Prometheus, first create a hash of the password.

To do this, you can use apache2-utils or httpd-tools.

Or you can use a programming lang.

## 15-12-Lab – Authentication-Encryption
TODO

## 16-13 - Metrics
Timeseries: Stream of timestamped values sharing the same metric and set of labels.

In histogram, the bigger buckets gonna include the values that fell into smaller buckets(it's accumulated).

Summary is gonna give us percentages. For example in response time metric, histogram is gonna calculate how many response times
were under .2s, how many under .5s(which also include under .2s) and how many under 1s?

But summary is gonna give us that 20% of total reqs were finished less than .2s, 50% were under .8s and 80% of all reqs finished less than
1s.

Do not use colons in metric names, they are for recording rules.

Every metric is automatically assigned 2 labels by default which are `instance` and `job`.

## 17-14 - Metrics Quiz

## 15 - Exploring Expression Browser
Expression Browser: is a web UI for prometheus server to query the server. localhost:9090

`up`: This query returns a metric called up which is gonna tell us which targets are in up state and which are down(which ones can be reached
and which ones can't). 1 is up, 0 is down.

Once you have a grafana instance installed, then you can build out dashboards using a separate tool so that you don't have to
interact with the expression browser and you can create more customized dashboards.

## 16 - Prometheus in Docker Container
We saw how to install Prometheus on a VM or bare-metal server.

Use bind mounts on docker image so that you can sync prometheus.yml from the host machine to the container.

## 17 - Intro to PromTools

## 18 - Monitoring Containers
We can collect:
- docker engine metrics
- container metrics using cadvisor

### docker engine metrics
To enable docker engine metrics, go to host machine and create daemon.json in /etc/docker and put the content in the slid

If you want info/metrics about **a container**, use cadvisor.

To enable cadvisor, we're gonna use a container to run the cAdvisor process and it's gonna responsible for getting the container's
info

## 19 - Lab – Monitoring Containers

Feedback – Prometheus Certified Associate