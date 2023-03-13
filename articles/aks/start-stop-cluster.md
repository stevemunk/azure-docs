---
title: Start and Stop an Azure Kubernetes Service (AKS)
description: Learn how to stop or start an Azure Kubernetes Service (AKS) cluster.
ms.topic: article
ms.date: 08/09/2021
author: palma21
---

# Stop and Start an Azure Kubernetes Service (AKS) cluster

Your AKS workloads may not need to run continuously, for example a development cluster that is used only during business hours. This leads to times where your Azure Kubernetes Service (AKS) cluster might be idle, running no more than the system components. You can reduce the cluster footprint by [scaling all the `User` node pools to 0](scale-cluster.md#scale-user-node-pools-to-0), but your [`System` pool](use-system-pools.md) is still required to run the system components while the cluster is running.
To optimize your costs further during these periods, you can completely turn off (stop) your cluster. This action will stop your control plane and agent nodes altogether, allowing you to save on all the compute costs, while maintaining all your objects (except standalone pods) and cluster state stored for when you start it again. You can then pick up right where you left of after a weekend or to have your cluster running only while you run your batch jobs.

## Before you begin

This article assumes that you have an existing AKS cluster. If you need an AKS cluster, see the AKS quickstart [using the Azure CLI][aks-quickstart-cli], [using Azure PowerShell][aks-quickstart-powershell], or [using the Azure portal][aks-quickstart-portal].

### Limitations

When using the cluster start/stop feature, the following restrictions apply:

- This feature is only supported for Virtual Machine Scale Sets backed clusters.
- The cluster state of a stopped AKS cluster is preserved for up to 12 months. If your cluster is stopped for more than 12 months, the cluster state cannot be recovered. For more information, see the [AKS Support Policies](support-policies.md).
- You can only start or delete a stopped AKS cluster. To perform any operation like scale or upgrade, start your cluster first.
- The customer provisioned PrivateEndpoints linked to private cluster need to be deleted and recreated again when you start a stopped AKS cluster.
- Because the stop process drains all nodes, any standalone pods (i.e. pods not managed by a Deployment, StatefulSet, DaemonSet, Job, etc.) will be deleted.

## Stop an AKS Cluster

### [Azure CLI](#tab/azure-cli)

You can use the `az aks stop` command to stop a running AKS cluster's nodes and control plane. The following example stops a cluster named *myAKSCluster*:

```azurecli-interactive
az aks stop --name myAKSCluster --resource-group myResourceGroup
```

You can verify when your cluster is stopped by using the [az aks show][az-aks-show] command and confirming the `powerState` shows as `Stopped` as on the below output:

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Stopped"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

If the `provisioningState` shows `Stopping` that means your cluster hasn't fully stopped yet.

### [Azure PowerShell](#tab/azure-powershell)

You can use the [Stop-AzAksCluster][stop-azakscluster] cmdlet to stop a running AKS cluster's nodes and control plane. The following example stops a cluster named *myAKSCluster*:

```azurepowershell-interactive
Stop-AzAksCluster -Name myAKSCluster -ResourceGroupName myResourceGroup
```

You can verify your cluster is stopped using the [Get-AzAksCluster][get-azakscluster] cmdlet and confirming the `ProvisioningState` shows as `Succeeded` as shown in the following output:

```Output
ProvisioningState       : Succeeded
MaxAgentPools           : 100
KubernetesVersion       : 1.20.7
...
```

If the `ProvisioningState` shows `Stopping` that means your cluster hasn't fully stopped yet.

---

> [!IMPORTANT]
> If you are using [Pod Disruption Budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) the stop operation can take longer as the drain process will take more time to complete.

## Start an AKS Cluster

> [!CAUTION]
> It is important that you don't repeatedly start/stop your cluster. Repeatedly starting/stopping your cluster may result in errors. Once your cluster is stopped, you should wait 15-30 minutes before starting it up again. 

### [Azure CLI](#tab/azure-cli)

You can use the `az aks start` command to start a stopped AKS cluster's nodes and control plane. The cluster is restarted with the previous control plane state and number of agent nodes.
The following example starts a cluster named *myAKSCluster*:

```azurecli-interactive
az aks start --name myAKSCluster --resource-group myResourceGroup
```

You can verify when your cluster has started by using the [az aks show][az-aks-show] command and confirming the `powerState` shows `Running` as on the below output:

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Running"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

If the `provisioningState` shows `Starting` that means your cluster hasn't fully started yet.

### [Azure PowerShell](#tab/azure-powershell)

You can use the [Start-AzAksCluster][start-azakscluster] cmdlet to start a stopped AKS cluster's nodes and control plane. The cluster is restarted with the previous control plane state and number of agent nodes.
The following example starts a cluster named *myAKSCluster*:

```azurepowershell-interactive
Start-AzAksCluster -Name myAKSCluster -ResourceGroupName myResourceGroup
```

You can verify when your cluster has started using the [Get-AzAksCluster][get-azakscluster] cmdlet and confirming the `ProvisioningState` shows `Succeeded` as shown in the following output:

```Output
ProvisioningState       : Succeeded
MaxAgentPools           : 100
KubernetesVersion       : 1.20.7
...
```

If the `ProvisioningState` shows `Starting` that means your cluster hasn't fully started yet.

---

> [!NOTE]
> When you start your cluster back up, the following is expected behavior:
>
> * The IP address of your API server may change.
> * If you are using cluster autoscaler, when you start your cluster back up your current node count may not be between the min and max range values you set. The cluster starts with the number of nodes it needs to run its workloads, which isn't impacted by your autoscaler settings. When your cluster performs scaling operations, the min and max values will impact your current node count and your cluster will eventually enter and remain in that desired range until you stop your cluster.

## Next steps

- To learn how to scale `User` pools to 0, see [Scale `User` pools to 0](scale-cluster.md#scale-user-node-pools-to-0).
- To learn how to save costs using Spot instances, see [Add a spot node pool to AKS](spot-node-pool.md).
- To learn more about the AKS support policies, see [AKS support policies](support-policies.md).

<!-- LINKS - external -->

<!-- LINKS - internal -->
[aks-quickstart-cli]: ./learn/quick-kubernetes-deploy-cli.md
[aks-quickstart-portal]: ./learn/quick-kubernetes-deploy-portal.md
[aks-quickstart-powershell]: ./learn/quick-kubernetes-deploy-powershell.md
[install-azure-cli]: /cli/azure/install-azure-cli
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[az-aks-show]: /cli/azure/aks#az_aks_show
[kubernetes-walkthrough-powershell]: kubernetes-walkthrough-powershell.md
[stop-azakscluster]: /powershell/module/az.aks/stop-azakscluster
[get-azakscluster]: /powershell/module/az.aks/get-azakscluster
[start-azakscluster]: /powershell/module/az.aks/start-azakscluster
