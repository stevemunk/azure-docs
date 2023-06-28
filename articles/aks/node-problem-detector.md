---
title:  Node Problem Detector (NPD) in Azure Kubernetes Service (AKS) nodes
description: Learn about how AKS uses Node Problem detector to expose issues with the node.
ms.topic: conceptual
ms.date: 05/31/2023
---

# Node Problem Detector (NPD)

[Node Problem Detector (NPD)](https://github.com/kubernetes/node-problem-detector) is a Kubernetes add-on that detects node-related problems and reports on them. It runs as a systemd serviced on each node in the cluster and collects various metrics and system information, such as CPU usage, disk usage, and network connectivity. When it detects a problem, it generates an Events and/or Node Conditions. NPD is used in AKS (Azure Kubernetes Service) to monitor and manage nodes in a Kubernetes cluster running on the Azure cloud platform. NPD is enabled by default as part of the AKS Linux Extension.


## Node Conditions
AKS uses the following Node conditions from NPD to expose permanent problems on the node. In addition to these node conditions, corresponding kubernetes events are also emitted. Node conditions indicate a permanent problem that makes the node unavailable.

|Problem Daemon type| NodeCondition | Reason |  
| --- | --- | --- | 
|CustomPluginMonitor| FilesystemCorruptionProblem | FilesystemCorruptionDetected |
|CustomPluginMonitor| KubeletProblem | KubeletIsDown |
|CustomPluginMonitor| ContainerRuntimeProblem | ContainerRuntimeIsDown |
|CustomPluginMonitor| VMEventScheduled | VMEventScheduled |
|CustomPluginMonitor| FrequentUnregisterNetDevice | UnregisterNetDevice|
|CustomPluginMonitor|FrequentKubeletRestart|FrequentKubeletRestart|
|CustomPluginMonitor|FrequentContainerdRestart|FrequentContainerdRestart|
|CustomPluginMonitor|FrequentDockerRestart|FrequentDockerRestart|
|SystemLogMonitor|KernelDeadlock|DockerHung|
|SystemLogMonitor|ReadonlyFilesystem |FilesystemIsReadOnly|

## Events 
In few temporary scenarios, Events are emitted with relevant information to be able to diagnose the underlying issue.

|Problem Daemon type| Reason  | 
|---|---|
|CustomPluginMonitor|FilesystemCorruptionDetected|
|CustomPluginMonitor|KubeletIsDown|
|CustomPluginMonitor|ContainerRuntimeIsDown|
|CustomPluginMonitor|FreezeScheduled|
|CustomPluginMonitor|RebootScheduled|
|CustomPluginMonitor|RedeployScheduled|
|CustomPluginMonitor|TerminateScheduled|
|CustomPluginMonitor|PreemptScheduled|
|CustomPluginMonitor|DNSProblem|
|CustomPluginMonitor|PodIPProblem|
|SystemLogMonitor|OOMKilling|
|SystemLogMonitor|TaskHung|
|SystemLogMonitor|UnregisterNetDevice|
|SystemLogMonitor| KernelOops|
|SystemLogMonitor| DockerSocketCannotConnect|
|SystemLogMonitor| KubeletRPCDeadlineExceeded|
|SystemLogMonitor|KubeletRPCNoSuchContainer|
|SystemLogMonitor|CNICannotStatFS|
|SystemLogMonitor|PLEGUnhealthy|
|SystemLogMonitor|KubeletStart|
|SystemLogMonitor|DockerStart|
|SystemLogMonitor|ContainerdStart|

## Check the node conditions and events
 ```azurecli-interactive
    kubectl describe node my-aks-node
```
The output is clipped to only show the relevant parts
```output
...
...

Conditions:
  Type                          Status  LastHeartbeatTime                 LastTransitionTime                Reason                          Message
  ----                          ------  -----------------                 ------------------                ------                          -------
  VMEventScheduled              False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   NoVMEventScheduled              VM has no scheduled event
  FrequentContainerdRestart     False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   NoFrequentContainerdRestart     containerd is functioning properly
  FrequentDockerRestart         False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   NoFrequentDockerRestart         docker is functioning properly
  FilesystemCorruptionProblem   False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   FilesystemIsOK                  Filesystem is healthy
  FrequentUnregisterNetDevice   False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   NoFrequentUnregisterNetDevice   node is functioning properly
  ContainerRuntimeProblem       False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:40 +0000   ContainerRuntimeIsUp            container runtime service is up
  KernelDeadlock                False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   KernelHasNoDeadlock             kernel has no deadlock
  FrequentKubeletRestart        False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   NoFrequentKubeletRestart        kubelet is functioning properly
  KubeletProblem                False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   KubeletIsUp                     kubelet service is up
  ReadonlyFilesystem            False   Thu, 01 Jun 2023 19:14:25 +0000   Thu, 01 Jun 2023 03:57:41 +0000   FilesystemIsNotReadOnly         Filesystem is not read-only
  NetworkUnavailable            False   Thu, 01 Jun 2023 03:58:39 +0000   Thu, 01 Jun 2023 03:58:39 +0000   RouteCreated                    RouteController created a route
  MemoryPressure                True    Thu, 01 Jun 2023 19:16:50 +0000   Thu, 01 Jun 2023 19:16:50 +0000   KubeletHasInsufficientMemory    kubelet has insufficient memory available
  DiskPressure                  False   Thu, 01 Jun 2023 19:16:50 +0000   Thu, 01 Jun 2023 03:57:22 +0000   KubeletHasNoDiskPressure        kubelet has no disk pressure
  PIDPressure                   False   Thu, 01 Jun 2023 19:16:50 +0000   Thu, 01 Jun 2023 03:57:22 +0000   KubeletHasSufficientPID         kubelet has sufficient PID available
  Ready                         True    Thu, 01 Jun 2023 19:16:50 +0000   Thu, 01 Jun 2023 03:57:23 +0000   KubeletReady                    kubelet is posting ready status. AppArmor enabled
...
...
...
Events:
  Type    Reason                   Age                  From     Message
  ----    ------                   ----                 ----     -------
  Normal  NodeHasSufficientMemory  94s (x176 over 15h)  kubelet  Node aks-agentpool-40622340-vmss000009 status is now: NodeHasSufficientMemory

```
These events are also available in [Container Insights](/azure/azure-monitor/containers/container-insights-overview) through [KubeEvents](/azure/azure-monitor/reference/tables/kubeevents).
