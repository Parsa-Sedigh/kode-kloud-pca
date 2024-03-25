## 47-1-Introduction
Why do we need service discovery?

Over time, we add more targets or we decommission(remove) some of them. Then we have to update prometheus.yml each time.
We don't want to do this. So if we're in a very highly dynamic env where servers are continuously, having to keep updating the configuration
is hard. That's where service discovery comes in.

Prometheus can keep track of all of the ec2 instances and other servers in cloud envs.

### Static
Sth we've been working with. This is one form of service discovery.

## 48-2-File
A service discovery mechanism. This is still mostly static.

## 49-3-AWS
How to setup ec2 service discovery? We just configure the ec2_sd_configs block in prometheus.yml .

With AmazonEC2ReadOnlyAccess permission, prometheus server can read the list of ec2 instances on our account.

Why aws doesn't provide public ip for the in list of ec2 instances that is returned by ec2 service discovery?
1. aws is expecting that prometheus is running within your aws cloud env as well. But if it isn't and you need access to the public ip,
there is a label that's gonna contain the public ip. So then you can map that to the `instance` label if you want to. So that you can
scrape the public ip.
2. another reason that it uses private ip for the value of instance label, is that not all ec2 instances have public ips.

In status > service discovery tab you can see the discovered targets to scrape.

With ec2 service discovery, as new ec2 instances get made and some of them get deleted, prometheus will regularly update the 
discovered services so you don't have to do anything.

## 50-4-Re-Labeling
Let's say we have some sort of service discovery set up to learn about the targets and find the new ones and remove the old ones from
the scrape list and let's say we don't want to keep all of the targets, we wanna filter out some of them because we wanna scrape
targets that are maybe in the production environment. For this, we can set up a relabeling rule so that 
we don't scrape specific targets that we learned from our service discovery mechanism. And in addition to that, if we get certain
metrics and labels that we got from a scraped target, we can set up relabeling rules to relabel or rename or a label. Let's say we got
this label: `instance="node1:9100"`. We wanna remove the port. For this, we can set up a relabeling rule. Another ex: `region="us-east-1"`.
Here, we wanna drop the label altogether. So we can drop labels and metrics as well.

Relabeling allows us to:
- filter out targets that we don't want to scrape
- rename 
- drop metrics and labels.

In relabeling we have two options:
- `relabel_configs`: this relabeling occurs before a scrape and only has labels added by service discovery mechanism
- `metric_relabel_configs`: this relabeling occurs after the scrape. So you have access to all of the metrics and labels that it got after
scrape

The `Target Labels` that you see in service discovery tab, are the labels that will be added to every metric that we receive from that target.

The `source_labels` specifies what labels you wanna look at to make a decision.

With `regex` property, we specify the value. We say if the specified label in `source_labels` has this label, we're gonna do sth.
With these, we can determine whether or not you wanna keep specific targets to scrape or rename a label.

Any label with two underscores at the beginning will get dropped after the relabeling process. So they're just metadata info, but you have
access to them in the `relabel_configs` section.

### Target labels
The example slide: Let's say we wanna save the __address__label as a target label, so that we have it for all the timeseries that we
collect from that target that matches the config we set in prometheus.yml . But we wanna first rename it and also remove the port.

Note: `.*:.*` means: <anything>:<anything>. Now in `(.*):.*` , by using parentheses in regex, we can reference that 
part later(group) using $1.

### Dropping label
Note: If you wanna keep specified labels and drop everything else, use `labelkeep`. But if you wanna drop just the specified labels
but keep everything else, use `labeldrop`.

### labelmap
If you wanna keep some of the labels with __ at the beginning of them, we can use action: labelmap

### metric_relabel_config
Let's say we wanna drop the **metric** called http_errors_total. How do we do that?

A: We know the name of the metric is technically a label called __name__.

Note: To drop any label with a specified value, we don't need to specify the `source_labels` property anymore.

## 51-5-Re-Labeling Demo
TODO

## 52-6-Lab – Re-Labeling
