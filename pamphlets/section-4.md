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
## 28-Lab – Operators, Vector Matching, Aggregators
## 29-9 - Functions
## 30-10 - Subquery
## 31-11 - Histogram & Summary
Lab – Functions, subqueries, Histogram, Summary
## 32-12 - Recording Rules
Lab – Recording Rules
## 33-14 - HTTP API