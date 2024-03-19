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