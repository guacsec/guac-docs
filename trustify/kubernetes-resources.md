---
title: Resource requirements
layout: page
permalink: /trustify/resources/
nav_order: 3
parent: Kubernetes
has_children: true
---

# Resource requirements & configuration

Configuring resource requests and limitations in Kubernetes is important to
ensure the application runs stable and performs as expected. Therefore, there
are some reasonable resource requests in the Helm charts, which get applied by
default. The default resource request is 1 CPU and 8 GiB of RAM, for both the
importer and API server deployment. There are no resource limits by default.

You can either reduce the resource requirements, at the cost of stability, or
give more resources to the cluster, supporting the workload. Pods can fail to
start, or become stuck in a "Pending" state, if resource requirements are not
meet.

{: .note }

To learn more about Kubernetes resource management, read
[Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/).

## Modifying the resource requests and limits

You can configure resource requirements by updating Trusted Profile Analyzer’s
Helm chart "values" file. After making changes to the "values" file, you need to
apply those changes by running the `helm update` command.

The following example shows the relevant sections of the values file:

```
modules:
  importer:
    resources:
      requests:
        memory: "12Gi"
  server:
    resources:
      requests:
        memory: "16Gi"
        cpu: 4
      limits:
        cpu: 8
```

The pod’s container uses these resources as defined under the resources section
for the importer feature and the API server.

{: .warning } Reducing the resource requests below the default values can lead
to degraded performance or instability of the application.
