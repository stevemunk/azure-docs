---
title: 'Quickstart: Diagnose a VM network traffic filter problem - Azure CLI'
titleSuffix: Azure Network Watcher
description: Learn how to use Azure CLI to diagnose a virtual machine network traffic filter problem using the IP flow verify capability of Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: halkazwini
tags: azure-resource-manager
ms.service: network-watcher
ms.topic: quickstart
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 11/02/2022
ms.author: halkazwini
ms.custom: mvc, devx-track-azurecli, mode-api, engagement-fy23
#Customer intent: I need to diagnose a virtual machine (VM) network traffic filter problem that prevents communication to and from a VM.
---

# Quickstart: Diagnose a virtual machine network traffic filter problem - Azure CLI

In this quickstart, you deploy a virtual machine (VM) and then check communications to and from an IP address and to a URL. You determine the cause of a communication failure and how to resolve it.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](~/articles/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

- This quickstart requires version 2.0 or later of the Azure CLI. If you are using Azure Cloud Shell, the latest version is already installed. 

- The Azure CLI commands in this quickstart are formatted to run in a Bash shell.

## Create a VM

1. Before you can create a VM, you must create a resource group to contain the VM. Create a resource group with [az group create](/cli/azure/group). The following example creates a resource group named *myResourceGroup* in the *eastus* location:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

2. Create a VM with [az vm create](/cli/azure/vm). If SSH keys don't already exist in a default key location, the command creates them. To use a specific set of keys, use the `--ssh-key-value` option. The following example creates a VM named *myVm*:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

The VM takes a few minutes to create. Don't continue with the remaining steps until the VM is created and the Azure CLI returns the output.

## Test network communication

To test network communication with Network Watcher, you must first enable a network watcher in the region the VM that you want to test is in, and then use Network Watcher's IP flow verify capability to test communication.

### Enable network watcher

If you already have a network watcher enabled in the East US region, skip to [Use IP flow verify](#use-ip-flow-verify). Use the [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) command to create a network watcher in the EastUS region:

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### Use IP flow verify

When you create a VM, Azure allows and denies network traffic to and from the VM, by default. You might override Azure's defaults later, allowing or denying additional types of traffic. To test whether traffic is allowed or denied to different destinations and from a source IP address, use the [az network watcher test-ip-flow](/cli/azure/network/watcher#az-network-watcher-test-ip-flow) command.

Test outbound communication from the VM to one of the IP addresses for www.bing.com:

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 13.107.21.200:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

After several seconds, the result returned informs you that access is allowed by a security rule named **DenyAllOutBound**.

Test outbound communication from the VM to 172.31.0.100:

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 172.31.0.100:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

The result returned informs you that access is denied by a security rule named **DenyAllOutBound**.

Test inbound communication to the VM from 172.31.0.100:

```azurecli-interactive
az network watcher test-ip-flow \
  --direction inbound \
  --local 10.0.0.4:80 \
  --protocol TCP \
  --remote 172.31.0.100:60000 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

The result returned informs you that access is denied because of a security rule named **DenyAllInBound**. Now that you know which security rules are allowing or denying traffic to or from a VM, you can determine how to resolve the problems.

## View details of a security rule

To determine why the rules in [Use IP flow verify](#use-ip-flow-verify) are allowing or preventing communication, review the effective security rules for the network interface with the [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) command:

```azurecli-interactive
az network nic list-effective-nsg \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

The output includes the following text for the **AllowInternetOutbound** rule that allowed outbound access to www.bing.com in a previous step under [Use IP flow verify](#use-ip-flow-verify):

```console
{
 "access": "Allow",
 "additionalProperties": {},
 "destinationAddressPrefix": "Internet",
 "destinationAddressPrefixes": [
  "Internet"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": [
  "1.0.0.0/8",
  "2.0.0.0/7",
  "4.0.0.0/6",
  "8.0.0.0/7",
  "11.0.0.0/8",
  "12.0.0.0/6",
  ...
 ],
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/AllowInternetOutBound",
 "priority": 65001,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

You can see in the previous output that **destinationAddressPrefix** is **Internet**. It's not clear how 13.107.21.200 relates to **Internet** though. You see several address prefixes listed under **expandedDestinationAddressPrefix**. One of the prefixes in the list is **12.0.0.0/6**, which encompasses the 12.0.0.1-15.255.255.254 range of IP addresses. Since 13.107.21.200 is within that address range, the **AllowInternetOutBound** rule allows the outbound traffic. Additionally, there are no higher priority (lower number) rules shown in the previous output that override this rule. To deny outbound communication to an IP address, you could add a security rule with a higher priority, that denies port 80 outbound to the IP address.

When you ran the [az network watcher test-ip-flow](/cli/azure/network/watcher#az-network-watcher-test-ip-flow) command to test outbound communication to 172.131.0.100 in [Use IP flow verify](#use-ip-flow-verify), the output informed you that the **DenyAllOutBound** rule denied the communication. The **DenyAllOutBound** rule equates to the **DenyAllOutBound** rule listed in the following output from the [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) command:

```console
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllOutBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
}
```

The rule lists **0.0.0.0/0** as the **destinationAddressPrefix**. The rule denies the outbound communication to 172.131.0.100 because the address is not within the **destinationAddressPrefix** of any of the other outbound rules in the output from the [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) command. To allow the outbound communication, you could add a security rule with a higher priority, that allows outbound traffic to port 80 at 172.131.0.100.

When you ran the [az network watcher test-ip-flow](/cli/azure/network/watcher#az-network-watcher-test-ip-flow) command in [Use IP flow verify](#use-ip-flow-verify) to test inbound communication from 172.131.0.100, the output informed you that the **DenyAllInBound** rule denied the communication. The **DenyAllInBound** rule equates to the **DenyAllInBound** rule listed in the following output from the [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) command:

```console
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Inbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllInBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

The **DenyAllInBound** rule is applied because, as shown in the output, no other higher priority rule exists in the output from the [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) command that allows port 80 inbound to the VM from 172.131.0.100. To allow the inbound communication, you could add a security rule with a higher priority that allows port 80 inbound from 172.131.0.100.

The checks in this quickstart tested Azure configuration. If the checks return the expected results and you still have network problems, ensure that you don't have a firewall between your VM and the endpoint you're communicating with and that the operating system in your VM doesn't have a firewall that is allowing or denying communication.

## Clean up resources

When no longer needed, you can use [az group delete](/cli/azure/group) to remove the resource group and all of the resources it contains:

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## Next steps

- Learn more about [security rules](../virtual-network/network-security-groups-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) and how to [create security rules](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule).
- Even with the proper network traffic filters in place, communication to a VM can still fail, due to routing configuration. To learn how to diagnose VM network routing problems, see [Diagnose VM routing problems](diagnose-vm-network-routing-problem-cli.md).
- [Learn more](network-watcher-connectivity-cli.md) about Connection troubleshoot to diagnose outbound routing, latency, and traffic filtering problems.
