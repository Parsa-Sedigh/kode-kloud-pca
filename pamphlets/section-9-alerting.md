
## 58-1-Introduction
One alert manager can support multiple prometheus servers.

### For
Let's say there was a small network issue, so the targets are down. We don't want to trigger an alert because it would give us a
false positive. For this, we can use `for` clause. So for example if the expr evaluates to true for 5m, we wanna trigger an alert.
This is good for things like high cpu metrics. We don't care about quick cpu spikes.

## 59-2-Labels & Annotations
### Annotations
You can't use annotations to set up rules on matching certain alerts to trigger notifications.

Firing sample value: the value of that metric when the alert was triggered

## 60-3-Alertmanager Architecture
When an alert arrives on alert manager, the dispatcher is gonna first receive it and then it's gonna forward it to inhibition node.
Inhibition node does one thing specifically which is: it allows you to specify rules where you can suppress certain alerts if other alerts
exist. So we can say: If alert x exists, we don't want to do anything with alert y, we want to supress alert y.

Then in silencing node it allows you to mute alerts. For example if you're performing some maintenance on one of the servers,
we know that we're gonna trigger some alerts and since it's expected, we can supress them as well, so we can set up a silence rule.

Routing is responsible for figuring out what alert gets sent to who through what integration. For example if alert x occurs,
who(which team) do we sent this to?

Notification engine has all of those 3rd party integrations like email, pagerduty and chat and it's responsible for sending notifs out.

## 61-4-Alertmanager Installation
After setting up alert manager, we have to set up prometheus to use that alert manager. So add alertmanagers property there according to the
slide.

## 62-5-Alertmanager Installation Systemd
Just like what we did with prometheus and node exporter, we wanna set up alert manager so it's managed by systemd.

```shell
# creating a group and then a user on mac
# All daemon users are prefixed with an underscore, such as _www.
sudo dscl . -create /Users/_alertmanager UniqueID 300
sudo dscl . -create /Users/_alertmanager PrimaryGroupID 300
sudo dscl . -create /Users/_alertmanager UserShell /usr/bin/false
```

Then when changing the ownership of folders, use create group and username which in this case was _alertmanager:
```shell
sudo chown -R _alertmanager:_alertmanager /etc/alertmanager/
```

Note: This is for enabling the service on reboot: `sudo systemctl enable alertmanager`.

On my machine, to run alertmanager:
```shell
sudo alertmanager --config.file=/etc/alertmanager/alertmanager.yml --storage.path=/var/lib/alertmanager
```

## 63-6-Lab – Alertmanager Installation

## 64-7-Configuration
A receiver is responsible for sending out the notif t oend user.

### Route
A route is gonna have a match statement and the way matchers work is you provide a bunch of labels that you look for in an **alert**.
So if an alert has those labels in the match statement, then it's gonna route that alert to the provided receiver.

## 65-8-Receivers & Notifiers
### global config
If we have the same config across multiple receivers, we can put them in this section.

## 66-9-Alertmanager Demo
Prometheus and alert manager don't have to run on the same server.

Look at `my-alerts` group in rules.yml .

In prometheus web ui, go to `Alerts` page.

The default grouping is based on `alertname`. So we're gonna have a separate notification for each `alertname`.

This:
```yaml
group_by: ['team', 'env']
```
Means all of the alerts that have the same team and env labels, are gonna be grouped into one notification. 

## 67-10-Silences

## 68-11-Lab – Alertmanager
