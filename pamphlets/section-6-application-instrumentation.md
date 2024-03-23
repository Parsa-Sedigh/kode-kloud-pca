## 40-1-Introduction
We've setup prometheus to gather and monitor metrics from our infra. Things like our node machines(linux, windows), docker engines and
docker containers. But we also want to collect metrics from our apps that are running on the infra. This is where prometheus client
libraries come in.

## 41-2-Instrumentation-basics

## 42-3-Labels

## 43-4-Histogram/Summary

## 44-5-Gauge

## 45-6-Best Practice
### naming metrics
- container_docker_restarts => docker_container_restarts . docker is kinda the library or the package, so it should be at the beginning 
- http_requests_sum => http_requests
- nginx_disk_free_kilobytes => nginx_disk_free_bytes
- dotnet_queue_waiting_time => dotnet_queue_waiting_time_seconds

## 46-7-Lab – Application Instrumentation