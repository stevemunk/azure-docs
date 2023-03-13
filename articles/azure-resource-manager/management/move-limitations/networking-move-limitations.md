---
title: Move Azure Networking resources to new subscription or resource group
description: Use Azure Resource Manager to move virtual networks and other networking resources to a new resource group or subscription.
ms.topic: conceptual
ms.date: 10/28/2022
---

# Move networking resources to new resource group or subscription

This article describes how to move virtual networks and other networking resources to a new resource group or Azure subscription.

During the move, your networking resources will operate without interruption.

If you want to move networking resources to a new region, see [Tutorial: Move Azure VMs across regions](../../../resource-mover/tutorial-move-region-virtual-machines.md).

## Dependent resources

> [!NOTE]
> Any resource, including a VPN Gateway, that is associated with a public IP Standard SKU address can't be moved across subscriptions. For virtual machines, you can [disassociate the public IP address](../../../virtual-network/ip-services/remove-public-ip-address-vm.md) before moving across subscriptions.

When moving a resource, you must also move its dependent resources (for example - public IP addresses, virtual network gateways, all associated connection resources). The virtual network assigned to the AKS instance can also be moved, and local network gateways can be in a different resource group.

> [!WARNING]
> Please refrain from moving the virtual network for an AKS cluster. The AKS cluster will stop working if its virtual network is moved.

To move a virtual machine with a network interface card to a new subscription, you must move all dependent resources. Move the virtual network for the network interface card, all other network interface cards for the virtual network, and the VPN gateways.

For more information, see [Scenario for move across subscriptions](../move-resource-group-and-subscription.md#scenario-for-move-across-subscriptions).

## Peered virtual network

To move a peered virtual network, you must first disable the virtual network peering. Once disabled, you can move the virtual network. After the move, reenable the virtual network peering.

## VPN Gateways

You cannot move VPN Gateways across resource groups or subscriptions if they are of Basic SKU. Basic SKU is only meant for test environment usage and doesn't support resource move operation.

## Subnet links

You can't move a virtual network to a different subscription if the virtual network contains a subnet with resource navigation links. For example, if an Azure Cache for Redis resource is deployed into a subnet, that subnet has a resource navigation link.

## Private endpoints

The following [private-link resources](../../../private-link/private-endpoint-overview.md#private-link-resource) support move:

* Microsoft.aadiam/privateLinkForAzureAD
* Microsoft.DocumentDB/databaseAccounts
* Microsoft.Kusto/clusters
* Microsoft.Search/searchServices
* Microsoft.SignalRService/SignalR
* Microsoft.SignalRService/webPubSub
* Microsoft.Sql/servers
* Microsoft.StorageSync/storageSyncServices
* Microsoft.Synapse/workspaces
* Microsoft.Synapse/privateLinkHubs

All other private-link resources don't support move.

## Next steps

For commands to move resources, see [Move resources to new resource group or subscription](../move-resource-group-and-subscription.md).
