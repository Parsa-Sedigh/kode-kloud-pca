## 53-1-Introduction
Batch job will run and then once it completes, it will push the metrics to push gtw. Now push gtw has the metrics stored in there
and then prometheus will scrape the push gtw like any other target or instance. It has no idea that it's talking to a push gtw. 
It's not gonna be configured any differently than any other instance with just one minor difference:
```yaml
honor_labels: true
```
This is for when prometheus scrapes metrics from push gtw, it's gonna automatically set the instance and the job labels to be the
push gateway server, which is not helpful because we want information of the original jobs, we don't care about the push gtw. This property
allows the metrics to specify custom labels for the instance and job labels. So the job1 can push a metric to push gtw and
set the instance='job1' and job='job1'.

## 54-1-Installation

## 55-1-Pushing Metrics
### HTTP
The <label>/<value> do two things:
- will get added to the metrics like what we had before
- they'll act as a grouping key with the job_name which are in the url path of pushing to gtw. Grouping key groups 
metrics together so we can update and delete multiple metrics at once.

EX: cat << EOF | curl --data-binary @- http://localhost:9091/metrics/job/archive/db/mysql

`<some metric name> {job="archive", db="mysql"}`

### PUT
EX: In: `metric_one{db="mysql",instance="",job="archive",label="val1"} 11
metric_two{db="mysql",instance="",job="archive"} 100
metric_one{app="web",instance="",job="archive",label="val1"} 22
metric_two{app="web",instance="",job="archive"} 200
metric_three{app="web",instance="",job="archive"} 300`

We have 2 groups(because groups are created based on job label and other labels(having an additional label can not differentiate)):
- group with db="mysql"
- group with app="web"

## 56-1-Client Library
## 57-1-Lab – Push Gateway
Feedback – Prometheus Certified Associate
