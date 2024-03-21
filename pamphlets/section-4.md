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
3 - Modifiers
4 - PromQL Demo
6 - Operators
7 - Vector Matching
8 - Aggregation
9 - Functions
10 - Subquery
11 - Histogram & Summary
12 - Recording Rules
14 - HTTP API