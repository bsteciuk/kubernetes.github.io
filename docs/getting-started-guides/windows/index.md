---
title: Windows Server Containers
---

Kubernetes version 1.5 introduces support for Windows Server Containers. Currently, the Kubernetes control plane (API Server, Scheduler, Controller Manager, etc) continue to run on Linux, while the kubelet can be run on Windows Server.

**Note:** Windows Server Containers on Kubernetes is an Alpha feature, currently working towards Beta.

## Prerequisites
In Kubernetes version 1.5 or later, Windows Server Containers for Kubernetes is supported using the following:

1. Kubernetes control plane running on existing Linux infrastructure (version 1.5 or later)
2. Windows Server 2016 (RTM version 10.0.14393 or later)
3. Docker Version 1.12.2-cs2-ws-beta or later for Windows Server nodes (Linux nodes and Kubernetes control plane can run any Kubernetes supported Docker Version)

## Networking
Because third-party networking plugins (e.g. flannel, calico, etc) don't natively work on Windows Server, cluster networking is currently achieved using Open vSwitch.  In this L3 networking approach, a /16 subnet is chosen for the cluster wide pod address space, and a /24 subnet within it is assigned to each node. All pods on a given worker node will be connected to the /24 subnet. Open vSwitch handles pod to pod routing and external access to/from the containers. 

## Setup
A walkthrough of setting up a cluster can be found at https://github.com/apprenda/kubernetes-ovn-heterogeneous-cluster/ .  It walks through the setup of the a cluster with a Windows worker and deploying a Windows workload to it.

## Scheduling Pods on Windows
Because your cluster has both Linux and Windows nodes, you must explicitly set the nodeSelector constraint to be able to schedule Pods to Windows nodes. You must set nodeSelector with the label beta.kubernetes.io/os to the value windows; see the following example:

```
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "iis",
    "labels": {
      "name": "iis"
    }
  },
  "spec": {
    "containers": [
      {
        "name": "iis",
        "image": "microsoft/iis",
        "ports": [
          {
            "containerPort": 80
          }
        ]
      }
    ],
    "nodeSelector": {
      "beta.kubernetes.io/os": "windows"
    }
  }
}
```

## Links
Sig-Windows community page - https://github.com/kubernetes/community/tree/master/sig-windows 

Sig-Windows onboarding page - https://github.com/apprenda/sig-windows

Heterogeneous Cluster Demo - https://github.com/apprenda/kubernetes-ovn-heterogeneous-cluster


## Known Limitations:
1. There is no network namespace in Windows and as a result currently only one container per pod is supported
2. Secrets as volumes do not work (though secrets can be used as environment variables) because of a bug in Windows Server Containers described [here](https://github.com/docker/docker/issues/28401)
3. Similarly ConfigMaps as volumes has the same problem, but work when used as environment variables in the container.

