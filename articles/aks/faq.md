---
title: Frequently asked questions for Azure Kubernetes Service (AKS)
description: Find answers to some of the common questions about Azure Kubernetes Service (AKS).
ms.topic: conceptual
ms.date: 06/17/2022
ms.custom: references_regions

---

# Frequently asked questions about Azure Kubernetes Service (AKS)

This article addresses frequent questions about Azure Kubernetes Service (AKS).

## Which Azure regions currently provide AKS?

For a complete list of available regions, see [AKS regions and availability][aks-regions].

## Can I spread an AKS cluster across regions?

No. AKS clusters are regional resources and can't span regions. See [best practices for business continuity and disaster recovery][bcdr-bestpractices] for guidance on how to create an architecture that includes multiple regions.

## Can I spread an AKS cluster across availability zones?

Yes. You can deploy an AKS cluster across one or more [availability zones][availability-zones] in [regions that support them][az-regions].

## Can I limit who has access to the Kubernetes API server?

Yes. There are two options for limiting access to the API server:

- Use [API Server Authorized IP Ranges][api-server-authorized-ip-ranges] if you want to maintain a public endpoint for the API server but restrict access to a set of trusted IP ranges.
- Use [a private cluster][private-clusters] if you want to limit the API server to *only* be accessible from within your virtual network.

## Can I have different VM sizes in a single cluster?

Yes, you can use different virtual machine sizes in your AKS cluster by creating [multiple node pools][multi-node-pools].

## Are security updates applied to AKS agent nodes?

AKS patches CVEs that have a "vendor fix" every week. CVEs without a fix are waiting on a "vendor fix" before it can be remediated.  The AKS images will get automatically updated inside of 30 days. We recommend you apply an updated Node Image on a regular cadence to ensure that latest patched images and OS patches are all applied and current. You can do this using one of the following methods:

- Manually, through the Azure portal or the Azure CLI.
- By upgrading your AKS cluster. The cluster upgrades [cordon and drain nodes][cordon-drain] automatically and then bring a new node online with the latest Ubuntu image and a new patch version or a minor Kubernetes version. For more information, see [Upgrade an AKS cluster][aks-upgrade].
- By using [node image upgrade](node-image-upgrade.md).

## What is the size limit on a container image in AKS?

AKS does not set a limit on the container image size. However, it is important to understand that the larger the image, the higher the memory demand. A larger size could potentially exceed resource limits or the overall available memory of worker nodes. By default, memory for VM size Standard_DS2_v2 for an AKS cluster is set to 7 GiB.

When a container image is excessively large, as in the Terabyte (TBs) range, kubelet might not be able to pull it from your container registry to a node due to lack of disk space.

### Windows Server nodes

For Windows Server nodes, Windows Update does not automatically run and apply the latest updates. On a regular schedule around the Windows Update release cycle and your own validation process, you should perform an upgrade on the cluster and the Windows Server node pool(s) in your AKS cluster. This upgrade process creates nodes that run the latest Windows Server image and patches, then removes the older nodes. For more information on this process, see [Upgrade a node pool in AKS][nodepool-upgrade].

### Are there security threats targeting AKS that customers should be aware of?

Microsoft provides guidance for other actions you can take to secure your workloads through services like [Microsoft Defender for Containers](../defender-for-cloud/defender-for-containers-introduction.md?tabs=defender-for-container-arch-aks). The following security threat is related to AKS and Kubernetes that customers should be aware of:

* [New large-scale campaign targets Kubeflow](https://techcommunity.microsoft.com/t5/azure-security-center/new-large-scale-campaign-targets-kubeflow/ba-p/2425750) - June 8, 2021

## How does the managed Control Plane communicate with my Nodes?

AKS uses a secure tunnel communication to allow the api-server and individual node kubelets to communicate even on separate virtual networks. The tunnel is secured through TLS encryption. The current main tunnel that is used by AKS is [Konnectivity, previously known as apiserver-network-proxy](https://kubernetes.io/docs/tasks/extend-kubernetes/setup-konnectivity/). Verify all network rules follow the [Azure required network rules and FQDNs](limit-egress-traffic.md).

## Why are two resource groups created with AKS?

AKS builds upon many Azure infrastructure resources, including Virtual Machine Scale Sets, virtual networks, and managed disks, which enables you to apply many of the core capabilities of the Azure platform within the managed Kubernetes environment provided by AKS. For example, most Azure virtual machine types can be used directly with AKS and Azure Reservations can be used to receive discounts on those resources automatically.

To enable this architecture, each AKS deployment spans two resource groups:

1. You create the first resource group. This group contains only the Kubernetes service resource. The AKS resource provider automatically creates the second resource group during deployment. An example of the second resource group is *MC_myResourceGroup_myAKSCluster_eastus*. For information on how to specify the name of this second resource group, see the next section.
2. The second resource group, known as the *node resource group*, contains all of the infrastructure resources associated with the cluster. These resources include the Kubernetes node VMs, virtual networking, and storage. By default, the node resource group has a name like *MC_myResourceGroup_myAKSCluster_eastus*. AKS automatically deletes the node resource group whenever the cluster is deleted, so it should only be used for resources that share the cluster's lifecycle.

## Can I provide my own name for the AKS node resource group?

Yes. By default, AKS will name the node resource group *MC_resourcegroupname_clustername_location*, but you can also provide your own name.

To specify your own resource group name, install the [aks-preview][aks-preview-cli] Azure CLI extension version *0.3.2* or later. When you create an AKS cluster by using the [`az aks create`][az-aks-create] command, use the `--node-resource-group` parameter and specify a name for the resource group. If you [use an Azure Resource Manager template][aks-rm-template] to deploy an AKS cluster, you can define the resource group name by using the *nodeResourceGroup* property.

* The secondary resource group is automatically created by the Azure resource provider in your own subscription.
* You can specify a custom resource group name only when you're creating the cluster.

As you work with the node resource group, keep in mind that you can't:

* Specify an existing resource group for the node resource group.
* Specify a different subscription for the node resource group.
* Change the node resource group name after the cluster has been created.
* Specify names for the managed resources within the node resource group.
* Modify or delete Azure-created tags of managed resources within the node resource group. (See additional information in the next section.)

## Can I modify tags and other properties of the AKS resources in the node resource group?

If you modify or delete Azure-created tags and other resource properties in the node resource group, you could get unexpected results, such as scaling and upgrading errors. AKS allows you to create and modify custom tags created by end users, and you can add those tags when [creating a node pool](use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool). You might want to create or modify custom tags, for example, to assign a business unit or cost center. Another option is to create Azure Policies with a scope on the managed resource group.

However, modifying any **Azure-created tags** on resources under the node resource group in the AKS cluster is an unsupported action, which breaks the service-level objective (SLO). For more information, see [Does AKS offer a service-level agreement?](#does-aks-offer-a-service-level-agreement)

## What Kubernetes admission controllers does AKS support? Can admission controllers be added or removed?

AKS supports the following [admission controllers][admission-controllers]:

- *NamespaceLifecycle*
- *LimitRanger*
- *ServiceAccount*
- *DefaultStorageClass*
- *DefaultTolerationSeconds*
- *MutatingAdmissionWebhook*
- *ValidatingAdmissionWebhook*
- *ResourceQuota*
- *PodNodeSelector*
- *PodTolerationRestriction*
- *ExtendedResourceToleration*

Currently, you can't modify the list of admission controllers in AKS.

## Can I use admission controller webhooks on AKS?

Yes, you may use admission controller webhooks on AKS. It's recommended you exclude internal AKS namespaces, which are marked with the **control-plane label.** For example:

```
namespaceSelector:
    matchExpressions:
    - key: control-plane
      operator: DoesNotExist
```

AKS firewalls the API server egress so your admission controller webhooks need to be accessible from within the cluster.

## Can admission controller webhooks impact kube-system and internal AKS namespaces?

To protect the stability of the system and prevent custom admission controllers from impacting internal services in the kube-system, namespace AKS has an **Admissions Enforcer**, which automatically excludes kube-system and AKS internal namespaces. This service ensures the custom admission controllers don't affect the services running in kube-system.

If you have a critical use case for deploying something on kube-system (not recommended) in support of your custom admission webhook, you may add the following label or annotation so that Admissions Enforcer ignores it.

Label: ```"admissions.enforcer/disabled": "true"``` or Annotation: ```"admissions.enforcer/disabled": true```

## Is Azure Key Vault integrated with AKS?

[Azure Key Vault Provider for Secrets Store CSI Driver][aks-keyvault-provider] provides native integration of Azure Key Vault into AKS.

## Can I run Windows Server containers on AKS?

Yes, Windows Server containers are available on AKS. To run Windows Server containers in AKS, you create a node pool that runs Windows Server as the guest OS. Windows Server containers can use only Windows Server 2019. To get started, see [Create an AKS cluster with a Windows Server node pool](./learn/quick-windows-container-deploy-cli.md).

Windows Server support for node pool includes some limitations that are part of the upstream Windows Server in Kubernetes project. For more information on these limitations, see [Windows Server containers in AKS limitations][aks-windows-limitations].

## Does AKS offer a service-level agreement?

AKS provides SLA guarantees in the [Standard pricing tier with the Uptime SLA feature][pricing-tiers].

The Free pricing tier doesn't have an associated Service Level *Agreement*, but has a Service Level *Objective* of 99.5%. Transient connectivity issues are observed if there was an upgrade, unhealthy underlay nodes, platform maintenance, or an application overwhelms the API Server with requests, etc. For mission-critical and production workloads, or if your workload doesn't tolerate API Server restarts, we recommend using the Standard tier, which includes Uptime SLA.

## Can I apply Azure reservation discounts to my AKS agent nodes?

AKS agent nodes are billed as standard Azure virtual machines. If you've purchased [Azure reservations][reservation-discounts] for the VM size that you're using in AKS, those discounts are automatically applied.

## Can I move/migrate my cluster between Azure tenants?

Moving your AKS cluster between tenants is currently unsupported.

## Can I move/migrate my cluster between subscriptions?

Movement of clusters between subscriptions is currently unsupported.

## Can I move my AKS clusters from the current Azure subscription to another?

Moving your AKS cluster and its associated resources between Azure subscriptions isn't supported.

## Can I move my AKS cluster or AKS infrastructure resources to other resource groups or rename them?

Moving or renaming your AKS cluster and its associated resources isn't supported.

## Why is my cluster delete taking so long?

Most clusters are deleted upon user request; in some cases, especially where customers are bringing their own Resource Group, or doing cross-RG tasks deletion can take more time or fail. If you have an issue with deletes, double-check that you do not have locks on the RG, that any resources outside of the RG are disassociated from the RG, and so on.

## Can I restore my cluster after deleting it?

No, you're unable to restore your cluster after deleting it. When you delete your cluster, the associated resource group and all its resources will also be deleted. If you want to keep any of your resources, move them to another resource group before deleting your cluster. If you have the **Owner** or **User Access Administrator** built-in role, you can lock Azure resources to protect them from accidental deletions and modifications. For more information, see [Lock your resources to protect your infrastructure][lock-azure-resources].

## If I have pod / deployments in state 'NodeLost' or 'Unknown' can I still upgrade my cluster?

You can, but we don't recommend it. Upgrades should be performed when the state of the cluster is known and healthy.

## If I have a cluster with one or more nodes in an Unhealthy state or shut down, can I perform an upgrade?

No, delete/remove any nodes in a failed state or otherwise removed from the cluster prior to upgrading.

## I ran a cluster delete, but see the error `[Errno 11001] getaddrinfo failed`

Most commonly, this is caused by users having one or more Network Security Groups (NSGs) still in use and associated with the cluster.  Remove them and attempt the delete again.

## I ran an upgrade, but now my pods are in crash loops, and readiness probes fail?

Confirm your service principal hasn't expired.  See: [AKS service principal](./kubernetes-service-principal.md) and [AKS update credentials](./update-credentials.md).

## My cluster was working, but suddenly can't provision LoadBalancers, mount PVCs, etc.?

Confirm your service principal hasn't expired.  See: [AKS service principal](./kubernetes-service-principal.md)  and [AKS update credentials](./update-credentials.md).

## Can I scale my AKS cluster to zero?

You can completely [stop a running AKS cluster](start-stop-cluster.md), saving on the respective compute costs. Additionally, you may also choose to [scale or autoscale all or specific `User` node pools](scale-cluster.md#scale-user-node-pools-to-0) to 0, maintaining only the necessary cluster configuration.
You can't directly scale [system node pools](use-system-pools.md) to zero.

## Can I use the Virtual Machine Scale Set APIs to scale manually?

No, scale operations by using the Virtual Machine Scale Set APIs aren't supported. Use the AKS APIs (`az aks scale`).

## Can I use Virtual Machine Scale Sets to manually scale to zero nodes?

No, scale operations by using the Virtual Machine Scale Set APIs aren't supported. You can use the AKS API to scale to zero non-system node pools or [stop your cluster](start-stop-cluster.md) instead.

## Can I stop or de-allocate all my VMs?

While AKS has resilience mechanisms to withstand such a config and recover from it, this isn't a supported configuration. [Stop your cluster](start-stop-cluster.md) instead.

## Can I use custom VM extensions?

No, AKS is a managed service, and manipulation of the IaaS resources isn't supported. To install custom components, use the Kubernetes APIs and mechanisms. For example, use DaemonSets to install required components.

## Does AKS store any customer data outside of the cluster's region?

No, all data is stored in the cluster's region.

## Are AKS images required to run as root?

The following images have functional requirements to "Run as Root" and exceptions must be filed for any policies:

- *mcr.microsoft.com/oss/kubernetes/coredns*
- *mcr.microsoft.com/azuremonitor/containerinsights/ciprod*
- *mcr.microsoft.com/oss/calico/node*
- *mcr.microsoft.com/oss/kubernetes-csi/azuredisk-csi*

## What is Azure CNI Transparent Mode vs. Bridge Mode?

Starting with version 1.2.0, Azure CNI sets Transparent mode as default for single tenancy Linux CNI deployments. Transparent mode is replacing bridge mode. In this section, we will discuss more about the differences about both modes and what are the benefits/limitation for using Transparent mode in Azure CNI.

### Bridge mode

As the name suggests, bridge mode Azure CNI, in a "just in time" fashion, will create an L2 bridge named "azure0". All the host side pod `veth` pair interfaces will be connected to this bridge. So Pod-Pod intra VM communication and the remaining traffic goes through this bridge. The bridge in question is a layer 2 virtual device that on its own cannot receive or transmit anything unless you bind one or more real devices to it. For this reason, eth0 of the Linux VM has to be converted into a subordinate to "azure0" bridge. This creates a complex network topology within the Linux VM and as a symptom CNI had to take care of other networking functions like DNS server update and so on.

:::image type="content" source="media/faq/bridge-mode.png" alt-text="Bridge mode topology":::

The following example shows what the ip route setup looks like in Bridge mode. Regardless of how many pods the node has, there will only ever be two routes. The first one saying, all traffic excluding local on azure0 will go to the default gateway of the subnet through the interface with ip "src 10.240.0.4" (which is Node primary IP) and the second one saying "10.20.x.x" Pod space to kernel for kernel to decide.

```bash
default via 10.240.0.1 dev azure0 proto dhcp src 10.240.0.4 metric 100
10.240.0.0/12 dev azure0 proto kernel scope link src 10.240.0.4
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
root@k8s-agentpool1-20465682-1:/#
```

### Transparent mode

Transparent mode takes a straight forward approach to setting up Linux networking. In this mode, Azure CNI won't change any properties of eth0 interface in the Linux VM. This minimal approach of changing the Linux networking properties helps reduce complex corner case issues that clusters could face with Bridge mode. In Transparent Mode, Azure CNI will create and add host-side pod `veth` pair interfaces that will be added to the host network. Intra VM Pod-to-Pod communication is through ip routes that the CNI will add. Essentially Pod-to-Pod communication is over layer 3 and pod traffic is routed by L3 routing rules.

:::image type="content" source="media/faq/transparent-mode.png" alt-text="Transparent mode topology":::

The following example shows a ip route setup of transparent mode. Each Pod's interface will get a static route attached so that traffic with dest IP as the Pod will be sent directly to the Pod's host side `veth` pair interface.

```bash
10.240.0.216 dev azv79d05038592 proto static
10.240.0.218 dev azv8184320e2bf proto static
10.240.0.219 dev azvc0339d223b9 proto static
10.240.0.222 dev azv722a6b28449 proto static
10.240.0.223 dev azve7f326f1507 proto static
10.240.0.224 dev azvb3bfccdd75a proto static
168.63.129.16 via 10.240.0.1 dev eth0 proto dhcp src 10.240.0.4 metric 100
169.254.169.254 via 10.240.0.1 dev eth0 proto dhcp src 10.240.0.4 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
```

### Benefits of transparent mode

- Provides mitigation for `conntrack` DNS parallel race condition and avoidance of 5-sec DNS latency issues without the need to set up node local DNS (you may still use node local DNS for performance reasons).
- Eliminates the initial 5-sec DNS latency CNI bridge mode introduces today due to "just in time" bridge setup.
- One of the corner cases in bridge mode is that the Azure CNI can't keep updating the custom DNS server lists users add to either VNET or NIC. This results in the CNI picking up only the first instance of the DNS server list. Solved in Transparent mode as CNI doesn't change any eth0 properties. See more [here](https://github.com/Azure/azure-container-networking/issues/713).
- Provides better handling of UDP traffic and mitigation for UDP flood storm when ARP times out. In bridge mode, when bridge doesn't know a MAC address of destination pod in intra-VM Pod-to-Pod communication, by design, this results in storm of the packet to all ports. Solved in Transparent mode as there are no L2 devices in path. See more [here](https://github.com/Azure/azure-container-networking/issues/704).
- Transparent mode performs better in Intra VM Pod-to-Pod communication in terms of throughput and latency when compared to bridge mode.

## How to avoid permission ownership setting slow issues when the volume has a lot of files?

Traditionally if your pod is running as a non-root user (which you should), you must specify a `fsGroup` inside the pod’s security context so that the volume can be readable and writable by the Pod. This requirement is covered in more detail in [here](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).

But one side-effect of setting `fsGroup` is that, each time a volume is mounted, Kubernetes must recursively `chown()` and `chmod()` all the files and directories inside the volume - with a few exceptions noted below. This happens even if group ownership of the volume already matches the requested `fsGroup`, and can be expensive for larger volumes with lots of small files, which causes pod startup to take a long time. This scenario has been a known problem before v1.20, and the workaround is setting the Pod run as root:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 0
    fsGroup: 0
```

The issue has been resolved with Kubernetes version 1.20. For more information, see [Kubernetes 1.20: Granular Control of Volume Permission Changes](https://kubernetes.io/blog/2020/12/14/kubernetes-release-1.20-fsgroupchangepolicy-fsgrouppolicy/).

## Can I use FIPS cryptographic libraries with deployments on AKS?

FIPS-enabled nodes are now supported on Linux-based node pools. For more information, see [Add a FIPS-enabled node pool](use-multiple-node-pools.md#add-a-fips-enabled-node-pool).

## Can I configure NSGs with AKS?

AKS doesn't apply Network Security Groups (NSGs) to its subnet and doesn't modify any of the NSGs associated with that subnet. AKS only modifies the network interfaces NSGs settings. If you're using CNI, you also must ensure the security rules in the NSGs allow traffic between the node and pod CIDR ranges. If you're using kubenet, you must also ensure the security rules in the NSGs allow traffic between the node and pod CIDR. For more information, see [Network security groups](concepts-network.md#network-security-groups).

## How does Time synchronization work in AKS?

AKS nodes run the "chrony" service, which pulls time from the localhost.  Containers running on pods get the time from the AKS nodes.  Applications launched inside a container use time from the container of the pod.

## How are AKS addons updated?

Any patch, including security patches, is automatically applied to the AKS cluster. Anything bigger than a patch, like major or minor version changes (which can have breaking changes to your deployed objects), is updated when you update your cluster if a new release is available. You can find when a new release is available by visiting the [AKS release notes](https://github.com/Azure/AKS/releases). 

<!-- LINKS - internal -->

[aks-upgrade]: ./upgrade-cluster.md
[aks-cluster-autoscale]: ./cluster-autoscaler.md
[aks-advanced-networking]: ./configure-azure-cni.md
[aks-rbac-aad]: ./azure-ad-integration-cli.md
[node-updates-kured]: node-updates-kured.md
[aks-preview-cli]: /cli/azure/aks
[az-aks-create]: /cli/azure/aks#az-aks-create
[aks-rm-template]: /azure/templates/microsoft.containerservice/2022-09-01/managedclusters
[aks-cluster-autoscaler]: cluster-autoscaler.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[aks-windows-cli]: windows-container-cli.md
[aks-windows-limitations]: ./windows-faq.md
[reservation-discounts]:../cost-management-billing/reservations/save-compute-costs-reservations.md
[api-server-authorized-ip-ranges]: ./api-server-authorized-ip-ranges.md
[multi-node-pools]: ./use-multiple-node-pools.md
[availability-zones]: ./availability-zones.md
[private-clusters]: ./private-clusters.md
[bcdr-bestpractices]: ./operator-best-practices-multi-region.md#plan-for-multiregion-deployment
[availability-zones]: ./availability-zones.md
[az-regions]: ../availability-zones/az-region.md
[pricing-tiers]: ./free-standard-pricing-tiers.md
[aks-keyvault-provider]: ./csi-secrets-store-driver.md

<!-- LINKS - external -->
[aks-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[auto-scaler]: https://github.com/kubernetes/autoscaler
[cordon-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[admission-controllers]: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
[private-clusters-github-issue]: https://github.com/Azure/AKS/issues/948
[csi-driver]: https://github.com/Azure/secrets-store-csi-driver-provider-azure
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[lock-azure-resources]: ../azure-resource-manager/management/lock-resources.md
