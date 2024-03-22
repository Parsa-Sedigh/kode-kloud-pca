# Section 4 - PromQL

## 20-1 - Introduction
When you run a promql expression, what gets return the prometheus server can be one of four types:
- string
- scalar
- instant vector
- range vector

Every combination of metric name and unique labels is considered one timeseries.

### range vector
If we run `node_cpu_seconds_total[3m]`: get the metric data not just for a single point in time but for the past 3 minutes.

## 21-2 - Selectors & Matchers
Label matchers allow us to match on specific labels of timeseries that we're actually interested in.

Note: In label selector: <label_name>="<value>", the value has to be in quotes(single or double).

### Range vector selectors
Get the node_arp_entries timeseries from node1 for the past 2 minutes:
`node_arp_entries{instance=node1}[2m]`

This is a range vector selector, so it returns **multiple** values. So we get a value for each scrape of the last 2 minutes.

## 22-3 - Modifiers
### Offset modifier
When we use both @ modifier and an offset modifier, the order that you use them does not matter.

Note: When using [2m] alone, it means get me the data of last 2 minutes. But when combined with @ modifier, it's gonna give us
2 minutes worth of data(range of values) at the specified time.

`node_filesystem_files{mountpoint=~"/System.*"}`

## 23-4 - PromQL Demo

## 24-5-Lab – PromQL Selectors, Matchers, and Modifiers

## 25-6 - Operators
### Arithmetic operators
When you modify any metric(like performing a mathematical op on it), what's returned is no longer a metric. So it removes the metric name.
So when you make any changes to the metric, the metric is no longer the same original metric, so it doesn't keep the metric name.

## 26-7 - Vector Matching
### Vector matching
Let's say we wanna know what percentage of our reqs got error? We have:
- http_errors with 2 labels
- http_requests with 1 label

But we have 1 metric that has 2 labels and other side has 1 label. But we can't because we know all the labels must match in order to do
ops. We can use the ignoring keyword.

Exactly match example: `node_filesystem_avail_bytes / node_filesystem_size_bytes * 100`.

`ignoring()` and `on()` are opposite. Instead of saying which labels to ignore, we can say which labels to match on using on().

ex: `http_errors_total / ignoring(error) group_left http_requests_total`. http_errors_total is the `many` side.

## 27-8 - Aggregation
Another form of operators like aggregation operators.

When we do an aggregation op, it's gonna perform it on every single entry in the instant vector.

`by` clause: groups entries based on the provided label. Ex: `sum by(path) (http_requests)`: sum the entries that have the same `path` label.

### aggregation operator
`without` is the opposite of `by`.

`sum without(path) (http_requests)`: aggregate based off all labels except `path` and sum them.

`sum by (mode) (node_cpu_seconds_total)`

## 28-Lab – Operators, Vector Matching, Aggregators

## 29-9 - Functions
### Changing type
If the instant vector doesn't have exactly one element and we call scalar(<metric name>), it returns Nan. Like the `node_cpu_seconds_total` example.

### rate
We know a counter always go up, we expect it to always go up, so plotting it doesn't give us useful info. Instead, we'd like to get the
rage of the increase.

Q: We wanna know what is the rate in which the errors are increasing.

### rate()
A: `rate(http_errors[1m])`. [1m] normally means we wanna get the timeseries in the last 1 minute, but when used inside rate(), it's different.
Each yellow circle shows at that point in time(horizontal axios), there was x errors(vertical axis). The values are not correct since
number of errors in decimal is impossible!

Here, 1m means how we're gonna group data together. So we group together all the scraped values over 1m window.

Then in each group, we subtract the last data point from the first one and then we divide the result by 60s. 

We're plotting the increase rate!

### irate()
Instead of taking the first and last data point of each group to calculate the rate, we use the last two data points of each group.

When using rate() in prometheus ui, use the graph tab instead of table to see the changes of the increase rate. With rate(), we can see
the rate has increases and also decreases, so it's more valuable info than a plain counter metric.

## 30-10 - Subquery
This won't work: `max_over_time(rate(http_requests_total[10m]))`

There are a couple of issues:
- max_over_time() expects a range vector but rate() returns an instant vector. In a range vector, you have multiple values and the
max_over_time() is gonna return the largest value.
- we know when we have rate(), the [10m] represents we're gonna group the data in 10m groups, not that it's gonna get the data for 
past 10m

To address this specific use case, we use subqueries.

Note: `node_network_transmit_bytes_total` metric returns total transmitted bytes on network interfaces of the targets.

Q: Give me the transmitted bytes in past 1hr in 30s query interval.

A: `rate(node_network_transmit_bytes_total[1m]) [1h:30s]` gives the data points for the past 1hr and each sample is gathered in
30s interval.

Q: What was the highest rate of transmitted bytes on network interfaces over the past 5 hours?

A: This won't work: `max_over_time(rate(node_network_transmit_bytes_total[1m]))`. Because of the two reasons we mentioned.
We need subquery => `max_over_time(rate(node_network_transmit_bytes_total[1m]) [5h:30s])`

## 31-11 - Histogram & Summary


## 32-12-Lab – Functions, subqueries, Histogram, Summary

## 33-13 - Recording Rules
Lab – Recording Rules
## 34-14 - HTTP API