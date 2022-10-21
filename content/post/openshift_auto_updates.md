---
title: "Openshift_auto_updates"
date: 2022-10-21T21:37:04+05:30
tags: ['openshift', 'auto update', 'cluster version operator']
---

<span style="font-size:25px; font-family:'Kalam'">
<br>Ever been in a situation where the OpenShift cluster has undergone an upgrade to the next version without admin's knowledge or any manual interference? If yes, congratulations! You've finally landed on a very relevant blog. 

Before we proceed, I expect that you are familiar with the cluster-version operator's lifecycle. If not, please feel free to take a look into [CVO's update process](https://apoorvajagtap.github.io/openshift_cvo_update_process/) before reading further.

Now that you are well aware of CVO's routine, let's get back to our surprise factor of this blog.
Anyone who has ever been in the above stated situation, must have wondered, was this an automatic upgrade? If yes, then what exactly triggered this automatic upgrade?; how can we control it?; or many more...<br><br>

➤ **Was it CVO ??**

The answer to the primary question: Yes, it is an automated upgrade managed via cluster-version operator (CVO).

The **`--enable-auto-update`** option if set to **`true`** does the magic of allowing CVO to update to the next available version for the respective channel (e.g. stable-4.x/fast-4.x/candidate-4.x). This option is basically specified as a container argument in `deployment.apps/cluster-version-operator`. 

```JSON
$ oc get deployment cluster-version-operator -ojson -n openshift-cluster-version | jq .spec.template.spec.containers[].args
[
  "start",
  "--release-image=quay.io/openshift-release-dev/ocp-release@sha256:3820c168791c2372cae687a022e009a8beafeff01dbb61f419d29d546cde02d9",
  "--enable-auto-update=true",
  "--enable-default-cluster-version=true",
  "--listen=0.0.0.0:9099",
  "--serving-cert-file=/etc/tls/serving-cert/tls.crt",
  "--serving-key-file=/etc/tls/serving-cert/tls.key",
  "--v=2"
]
```
<br>

➤ **What if CVO wasn't at fault ??**

There could be times, when **`--enable-auto-update`** is set to **`false`**, but you are still unaware of how & when was the upgrade triggered. In such situation, **audit logs** could come to rescue. 

[OpenShift auditing](https://docs.openshift.com/container-platform/4.11/security/audit-log-view.html) logs all the API requests coming to the server.

Filtering the audit logs with `"verb":"update"` and `"resource":"clusterversions"`, we can list the details on who triggered the upgrade.

<br>So, next time your cluster undergoes an abrupt upgrade, don't worry, the following got you covered:

- Check for the value set to --enable-auto-update.
- Scrutinize the OpenShift's audit logs. (The best one!)

</span>