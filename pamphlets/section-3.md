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
sydo systemctl daemon-reload

sudo systemctl start prometheus
systemctl status prometheus

# enable it on startup
sudo systemctl enable prometheus
```

## 10-7 - Node Exporter