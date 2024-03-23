# Section 

## 36-1-Introduction
Prometheus's builtin dashboarding & visulization tools.

## 37-2-Expression Browser

## 38-3-Console Templates
Alternative builtin visualization tool of prometheus.

Go to http://localhost:9090/consoles/index.html.example

```shell
# installed using binaries
cd /etc/prometheus/consoles

# if installed with homebrew
cd /opt/homebrew/Cellar/prometheus/2.50.1/libexec/consoles

touch demo.html
```

Note: In go templates written for prometheus dashboard, `prom_query_drilldown` takes us to the expression browser of prometheus browser.

## 39-4-Lab – Console Templates

Feedback – Prometheus Certified Associate