---
title: "Cluster Version Operator's Lifecycle"
date: 2022-10-20T23:19:06+05:30
tags: ['openshift', 'cluster version operator']
---

<span style="font-size:25px; font-family:'Kalam'">
<br>OpenShift's cluster version operator is quite an interesting and important one. In this blog, we are going to talk about how it manages the cluster-version. 

Before we move ahead, I expect that you are familiar with the following:
1. Worked on OpenShift 4.x clusters with admin privileges.
2. Have atleast basic idea about cluster version operator (CVO) and it's importance in OpenShift 4.x.
3. Tried/Observed OpenShift 4.x's upgrade.

As you are still reading, I believe you've met atleast one of the above pre-requisites, or, have a knack for technologies (Lovely!).

<br>

➤ **Cluster Version Operator (CVO)'s roles & responsibilities**

CVO is responsible to manage the lifecycle of default cluster Operators, and to monitor the version consistency throughout the cluster. 

I personally use the following command (which fetches the details from CVO) as first confirmation about cluster's current health:

```
$ oc get clusterversion
```

If any of the default cluster operators (installed with OpenShift's installation) is suffering, or not Available, the above command reports about the same.

In addition to this, CVO periodically checks for valid updates available based on current component versions. Let's talk about how this is managed in the next section.

<br>

➤ **CVO checks for available updates**

Fortunately, upstream contributors are really helpful to document such exciting topics, and one such example is the documentation maintained by [Cincinnati in OpenShift](https://github.com/openshift/cincinnati/blob/master/docs/design/openshift.md#update-process) about CVO's update-check cycle.

Here's a flow chart that could help in understanding the complete flow quickly (ref: [Cincinnati in OpenShift](https://github.com/openshift/cincinnati/blob/master/docs/design/openshift.md#update-process)).
<img src="/images/post/cvo_flow_chart.png" alt="cvo_update_process" class="center">

<br>The above helps us to understand the sequence of events, but once I understood this, I was quite curious to know for exactly how long the CVO sleeps before checking for update on upstream again? 

To find answer of this, I took a dive into the [codebase](https://github.com/openshift/cluster-version-operator/blob/master/cmd/start.go).

<br>

➤ **CVO sleeps for minResyncPeriod**

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

So, by default, the minimum reconcile period for CVO is **2 minutes**.

Now, if we revisit the flow chart, CVO sleeps for 2 minutes before checking with upstream again, when any of the following occurs:
- No new update available, or,
- Invalid image is downloaded, or,
- All the operators have finally updated to the `desiredVersion`. 

<br>I hope this blog helps you with a quick overview of OpenShift's Cluster version Operator's role during the update process. In case any of the point is confusing, or seems "odd one out", I'd love to know hear about it :D.
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