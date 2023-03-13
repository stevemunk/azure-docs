---
title: Deprovision an Azure disk pool (preview)
description: Learn how to deprovision, stop, and delete and Azure disk pool.
author: roygara
ms.date: 02/28/2023
ms.topic: conceptual
ms.author: rogarana
ms.service: storage
ms.subservice: disks
ms.custom: ignite-fall-2021, devx-track-azurepowershell
---

# Deprovision an Azure disk pool (preview)

> [!IMPORTANT]
> Disk pools are being retired soon. If you're looking for an alternative solution, see either [Azure Elastic SAN (preview)](../storage/elastic-san/elastic-san-introduction.md) or [Azure NetApp Files](../aks/azure-netapp-files.md).

This article covers the deletion process for an Azure disk pool (preview) and how to disable iSCSI support.

## Stop a disk pool

You can stop a disk pool to save costs and preserve all configurations. When a disk pool is stopped, you can no longer connect to it over iSCSI. The managed resources deployed to support the disk pool will not be deleted. You must disconnect all clients with iSCSI connections to the disk pool first before stopping a disk pool. You can start a disk pool at any time. This will reactivate the iSCSI target exposed on this disk pool.
# [Portal](#tab/azure-portal)

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Navigate to your disk pool, and select **Stop**.

# [PowerShell](#tab/azure-powershell)

```azurepowershell
Stop-AzDiskPool -Name '<yourDiskPool>' -ResourceGroupName '<yourResourceGroup>'
```

# [Azure CLI](#tab/azure-cli)

```azurecli
az disk-pool stop --name "<yourDiskPool>" --resource-group "<yourResourceGroup>"
```
---

## Disable iSCSI support

If you disable iSCSI support on a disk pool, you can no longer connect to a disk pool.

When you first enable iSCSI support on a disk pool, an iSCSI target is created as the endpoint for the iSCSI connection. You can disable iSCSI support on the disk pool by deleting the iSCSI target. Each disk pool can only have one iSCSI target configured.

You can re-enable iSCSI support on an existing disk pool. iSCSI support cannot be disabled on the disk pool if there are outstanding iSCSI connections to the disk pool.

# [Portal](#tab/azure-portal)

1. Search for **Disk pool** and select your disk pool.
1. Select **iSCSI** under **Settings**.
1. Clear the **Enable iSCSI** checkbox and select **Save**.    

# [PowerShell](#tab/azure-powershell)

```azurepowershell
Remove-AzDiskPoolIscsiTarget -DiskPoolName "<yourDiskpoolName>" -Name "<youriSCSITargetName>" -ResourceGroupName "yourResourceGroup>"
```

# [Azure CLI](#tab/azure-cli)

```azurecli
az disk-pool iscsi-target delete --disk-pool-name "<yourDiskPool>" --name "<yourIscsiTarget>" --resource-group "<yourResourceGroup>"
```
---

## Delete a disk pool

When you delete a disk pool, all the resources in the managed resource group are also deleted. If there are outstanding iSCSI connections to the disk pool, you cannot delete the disk pool. You must disconnect all clients with iSCSI connections to the disk pool first. Disks that have been added to the disk pool will not be deleted.

# [Portal](#tab/azure-portal)

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Search for **Disk pool** and select it, then select the disk pool you want to delete.
1. Select **Delete** at the top of the pane.

# [PowerShell](#tab/azure-powershell)

```azurepowershell
Stop-AzDiskPool -Name "<yourDiskPoolName>" -ResourceGroupName "<yourResourceGroup>"

Remove-AzDiskPool -Name "<yourDiskPoolName>" -ResourceGroupName "<yourResourceGroup>
Remove-AzDiskPool -Name "<yourDiskpoolName>" -ResourceGroupName "<yourResourceGroup>"
```

# [Azure CLI](#tab/azure-cli)

```azurecli
az disk-pool delete --name "<yourDiskPool>" --resource-group "<yourResourceGroup>"
```

---

## Next steps

Learn about [Azure managed disks](managed-disks-overview.md).
