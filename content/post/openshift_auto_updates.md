---
title: "CVO during OpenShift upgrade"
date: 2022-10-07T21:23:08+05:30
tags: ['openshift', 'auto update', 'cluster version operator']
draft: true
---

<span style="font-size:25px; font-family:'Kalam'">
<br>While working with OpenShift end-users, I've observed several instances where the OpenShift cluster has initiated an upgrade to the latest version without the knowledge/interference of the admin. This is most likely to happen when an automatic upgrade option is set to true in CVO's (cluster-version operator) deployment. And it would lead to a real mess if the admin is unaware about this option.
Such events made me curious about CVO's plan of action w.r.t cluster versions, and then I started my research on how CVO keeps the track & manages the cluster's version throughout.   

Fortunately, upstream contributors are really helpful to document such exciting topics, and during my hunt I landed on the documentation maintained by [Cincinnati in OpenShift](https://github.com/openshift/cincinnati/blob/master/docs/design/openshift.md#update-process) about update process's desigm.

Here's a flow chart that could help to understand the complete flow of CVO quickly (ref: [Cincinnati in OpenShift](https://github.com/openshift/cincinnati/blob/master/docs/design/openshift.md#update-process)).
<img src="/images/post/cvo_flow_chart.png" alt="cvo_update_process" class="center">


<br>The above helps us to understand the sequence of events, but what about the time duration for which CVO sleeps/waits before checking for next update-graph? To find answer of this, I took a dive into the [codebase](https://github.com/openshift/cluster-version-operator/blob/master/cmd/start.go).

<br>➤ **CVO sleeps for a set duration of time**

With the start of CVO container, we initialize the controllers and attempt to load the payload information using [NewControllerContext()](https://github.com/openshift/cluster-version-operator/blob/master/pkg/start/start.go#L432). The `NewControllerContext()` initializes the default context and while doing so, it passes `resyncPeriod(o.ResyncInterval)()` as the [**minimumUpdateCheckInterval**](https://github.com/openshift/cluster-version-operator/blob/master/pkg/cvo/cvo.go#L192) (i.e. the minimum duration to check for updates from the upstream).  

The value of `resyncPeriod(o.ResyncInterval)()` is determined using a constant [**minResyncPeriod**](https://github.com/openshift/cluster-version-operator/blob/master/pkg/start/start.go#L47), which is set to '2 Minutes'.

```GO
const (
	...
	minResyncPeriod = 2 * time.Minute
)
...
func NewOptions() *Options {
	return &Options{
		...
		ResyncInterval:  minResyncPeriod,
		...
	}
}
...
func (o *Options) NewControllerContext(cb *ClientBuilder, startingFeatureSet string) *Context {
    ...
		CVO: cvo.New(
			...
			resyncPeriod(o.ResyncInterval)(),
			...
        )
  ...
}
``` 
<!-- <img src="/images/post/cvo_start_go.png" alt="start.go" class="center"> -->
<!-- ![start.go](/images/post/cvo_start_go.png) -->

So, by default, the minimun reconcile period for CVO is **2 minutes**.

<br>Now, the next question that pops up is how CVO confirms whether the auto-upgrade is enabled or not?

➤ **CVO checks for auto-update**

The **`--enable-auto-update`** option if set to **`true`** enables the `autoupdate controller` and, hence informs CVO that the auto-update is enabled. This option is basically specified as a container argument in `deployment.apps/cluster-version-operator`. 

```JSON
$ oc get deployment cluster-version-operator -ojson -n openshift-cluster-version | jq .spec.template.spec.containers[].args
[
  "start",
  "--release-image=quay.io/openshift-release-dev/ocp-release@sha256:3820c168791c2372cae687a022e009a8beafeff01dbb61f419d29d546cde02d9",
  "--enable-auto-update=false",
  "--enable-default-cluster-version=true",
  "--listen=0.0.0.0:9099",
  "--serving-cert-file=/etc/tls/serving-cert/tls.crt",
  "--serving-key-file=/etc/tls/serving-cert/tls.key",
  "--v=2"
]
```

<!-- ![CVO_container_args](/images/post/cvo_container_args.png) -->


<br>I hope this blog helps you with a better understanding of OpenShift's Cluster version Operator's role during the update process.
</span>

<!-- 
The **`--enable-auto-update`** option set to **`true`** does the magic of allowing cluster-version operator to check for available versions and update to the newest version of the respective channel, periodically.


useful links: https://github1s.com/openshift/cluster-version-operator/blob/HEAD/cmd/start.go#L38-L39 (description of option) 
https://github.com/openshift/cluster-version-operator/blob/master/pkg/cvo/cvo.go#L93-L95
https://github.com/openshift/cluster-version-operator/blob/master/pkg/cvo/cvo.go#L161-L192
https://github.com/openshift/cluster-version-operator/blob/master/pkg/start/start.go#L451-L456
https://github.com/openshift/cluster-version-operator/blob/master/pkg/start/start.go#L102
https://github.com/openshift/cluster-version-operator/blob/master/pkg/start/start.go#L47
https://github.com/openshift/cluster-version-operator/blob/master/pkg/cvo/cvo.go#L561-L634

FROM THE CVO LOGS ():
I1007 18:34:28.656423       1 cvo.go:331] Starting ClusterVersionOperator with minimum reconcile period 2m50.956499648s
>> ref: https://github.com/openshift/cluster-version-operator/blob/master/pkg/cvo/cvo.go#L364
Now why the period in logs has nanoseconds too? ans >> https://github.com/openshift/cluster-version-operator/blob/master/pkg/start/start.go#L338-L343

// just to compare two various instances.. (exact nanoseconds) 
I1007 18:36:58.145132       1 cvo.go:331] Starting ClusterVersionOperator with minimum reconcile period 2m50.956499648s
-->