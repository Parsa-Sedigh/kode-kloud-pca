
## 58-1-Introduction
One alert manager can support multiple prometheus servers.

### For
Let's say there was a small network issue, so the targets are down. We don't want to trigger an alert because it would give us a
false positive. For this, we can use `for` clause. So for example if the expr evaluates to true for 5m, we wanna trigger an alert.
This is good for things like high cpu metrics. We don't care about quick cpu spikes.

## 59-2-Labels & Annotations
## 60-3-Alertmanager Architecture
## 61-4-Alertmanager Installation
## 62-5-Alertmanager Installation Systemd
## 63-6-Lab – Alertmanager Installation
## 64-7-Configuration
## 65-8-Receivers & Notifiers
## 66-9-Alertmanager Demo
## 67-10-Silences
## 68-11-Lab – Alertmanager
