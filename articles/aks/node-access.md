---
title: Connect to Azure Kubernetes Service (AKS) cluster nodes
description: Learn how to connect to Azure Kubernetes Service (AKS) cluster nodes for troubleshooting and maintenance tasks.
ms.topic: article
ms.date: 04/26/2023
ms.reviewer: mattmcinnes
ms.custom: contperf-fy21q4

#Customer intent: As a cluster operator, I want to learn how to connect to virtual machines in an AKS cluster to perform maintenance or troubleshoot a problem.
---

# Connect to Azure Kubernetes Service (AKS) cluster nodes for maintenance or troubleshooting

Throughout the lifecycle of your Azure Kubernetes Service (AKS) cluster, you might need to access an AKS node. This access could be for maintenance, log collection, or troubleshooting operations. You can securely authenticate against AKS Linux and Windows nodes using SSH, and you can also [connect to Windows Server nodes using remote desktop protocol (RDP)][aks-windows-rdp]. For security reasons, the AKS nodes aren't exposed to the internet. To connect to the AKS nodes, you use `kubectl debug` or the private IP address. 

This article shows you how to create a connection to an AKS node and update the SSH key on an existing AKS cluster.

## Before you begin

This article assumes you have an SSH key. If not, you can create an SSH key using [macOS or Linux][ssh-nix] or [Windows][ssh-windows]. Make sure you save the key pair in an OpenSSH format, other formats like .ppk aren't supported.

You also need the Azure CLI version 2.0.64 or later installed and configured. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI][install-azure-cli].

## Create an interactive shell connection to a Linux node

To create an interactive shell connection to a Linux node, use the `kubectl debug` command to run a privileged container on your node. To list your nodes, use the `kubectl get nodes` command:

```bash
kubectl get nodes -o wide
```

The following example resembles output from the command:

```output
NAME                                STATUS   ROLES   AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION     CONTAINER-RUNTIME
aks-nodepool1-12345678-vmss000000   Ready    agent   13m     v1.19.9   10.240.0.4    <none>        Ubuntu 18.04.5 LTS               5.4.0-1046-azure   containerd://1.4.4+azure
aks-nodepool1-12345678-vmss000001   Ready    agent   13m     v1.19.9   10.240.0.35   <none>        Ubuntu 18.04.5 LTS               5.4.0-1046-azure   containerd://1.4.4+azure
aksnpwin000000                      Ready    agent   87s     v1.19.9   10.240.0.67   <none>        Windows Server 2019 Datacenter   10.0.17763.1935    docker://19.3.1
```

Use the `kubectl debug` command to run a container image on the node to connect to it. The following command starts a privileged container on your node and connects to it.

```bash
kubectl debug node/aks-nodepool1-12345678-vmss000000 -it --image=mcr.microsoft.com/dotnet/runtime-deps:6.0
```

The following example resembles output from the command:

```output
Creating debugging pod node-debugger-aks-nodepool1-12345678-vmss000000-bkmmx with container debugger on node aks-nodepool1-12345678-vmss000000.
If you don't see a command prompt, try pressing enter.
root@aks-nodepool1-12345678-vmss000000:/#
```

This privileged container gives access to the node.

> [!NOTE]
> You can interact with the node session by running `chroot /host` from the privileged container.

### Remove Linux node access

When done, `exit` the interactive shell session. After the interactive container session closes, delete the pod used for access with `kubectl delete pod`.

```bash
kubectl delete pod node-debugger-aks-nodepool1-12345678-vmss000000-bkmmx
```

## Create the SSH connection to a Windows node

At this time, you can't connect to a Windows Server node directly by using `kubectl debug`. Instead, you need to first connect to another node in the cluster, then connect to the Windows Server node from that node using SSH. Alternatively, you can [connect to Windows Server nodes using remote desktop protocol (RDP) connections][aks-windows-rdp] instead of using SSH.

To connect to another node in the cluster, use the `kubectl debug` command. For more information, see [Create an interactive shell connection to a Linux node][ssh-linux-kubectl-debug].

To create the SSH connection to the Windows Server node from another node, use the SSH keys provided when you created the AKS cluster and the internal IP address of the Windows Server node.

> [!IMPORTANT]
>
> The following steps for creating the SSH connection to the Windows Server node from another node can only be used if you created your AKS cluster using the Azure CLI and the `--generate-ssh-keys` parameter. If you didn't use this method to create your cluster, use a password instead of an SSH key. To do this, see [Create the SSH connection to a Windows node using a password](#create-the-ssh-connection-to-a-windows-node-using-a-password)

Open a new terminal window and use the `kubectl get pods` command to get the name of the pod started by `kubectl debug`.

```bash
kubectl get pods
```

The following example resembles output from the command:

```output
NAME                                                    READY   STATUS    RESTARTS   AGE
node-debugger-aks-nodepool1-12345678-vmss000000-bkmmx   1/1     Running   0          21s
```

In the above example, *node-debugger-aks-nodepool1-12345678-vmss000000-bkmmx* is the name of the pod started by `kubectl debug`.

Use the `kubectl port-forward` command to open a connection to the deployed pod:

```bash
kubectl port-forward node-debugger-aks-nodepool1-12345678-vmss000000-bkmmx 2022:22
```

The following example resembles output from the command:

```output
Forwarding from 127.0.0.1:2022 -> 22
Forwarding from [::1]:2022 -> 22
```

The above example begins forwarding network traffic from port 2022 on your development computer to port 22 on the deployed pod. When using `kubectl port-forward` to open a connection and forward network traffic, the connection remains open until you stop the `kubectl port-forward` command.

Open a new terminal and run the command `kubectl get nodes` to show the internal IP address of the Windows Server node:

```bash
kubectl get nodes -o wide
```

The following example resembles output from the command:

```output
NAME                                STATUS   ROLES   AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION     CONTAINER-RUNTIME
aks-nodepool1-12345678-vmss000000   Ready    agent   13m     v1.19.9   10.240.0.4    <none>        Ubuntu 18.04.5 LTS               5.4.0-1046-azure   containerd://1.4.4+azure
aks-nodepool1-12345678-vmss000001   Ready    agent   13m     v1.19.9   10.240.0.35   <none>        Ubuntu 18.04.5 LTS               5.4.0-1046-azure   containerd://1.4.4+azure
aksnpwin000000                      Ready    agent   87s     v1.19.9   10.240.0.67   <none>        Windows Server 2019 Datacenter   10.0.17763.1935    docker://19.3.1
```

In the above example, *10.240.0.67* is the internal IP address of the Windows Server node.

Create an SSH connection to the Windows Server node using the internal IP address, and connect to port 22 through port 2022 on your development computer. The default username for AKS nodes is *azureuser*. Accept the prompt to continue with the connection. You're then provided with the bash prompt of your Windows Server node:

```bash
ssh -o 'ProxyCommand ssh -p 2022 -W %h:%p azureuser@127.0.0.1' azureuser@10.240.0.67
```

The following example resembles output from the command:

```output
The authenticity of host '10.240.0.67 (10.240.0.67)' can't be established.
ECDSA key fingerprint is SHA256:1234567890abcdefghijklmnopqrstuvwxyzABCDEFG.
Are you sure you want to continue connecting (yes/no)? yes

[...]

Microsoft Windows [Version 10.0.17763.1935]
(c) 2018 Microsoft Corporation. All rights reserved.

azureuser@aksnpwin000000 C:\Users\azureuser>
```

> [!NOTE]
> If you prefer to use password authentication, include the parameter `-o PreferredAuthentications=password`. For example:
>
> ```console
>  ssh -o 'ProxyCommand ssh -p 2022 -W %h:%p azureuser@127.0.0.1' -o PreferredAuthentications=password azureuser@10.240.0.67
> ```

### Create the SSH connection to a Windows node using a password

If you didn't create your AKS cluster using the Azure CLI and the `--generate-ssh-keys` parameter, you'll use a password instead of an SSH key to create the SSH connection. To do this with Azure CLI, use the following steps:

1. Create a root user called `azureuser`.

    ```azurecli
    az vmss update -g <nodeRG> -n <vmssName> --set virtualMachineProfile.osProfile.adminUsername=azureuser
    ```

2. Create a password for the new root user.

    ```azurecli
    az vmss update -g <nodeRG> -n <vmssName> --set virtualMachineProfile.osProfile.adminPassword=<new password>
    ```

3. Update the instances to use the above changes.

    ```azurecli
    az vmss update-instances -g <nodeRG> -n <vmssName> --instance-ids '*'
    ```

4. Reimage the affected nodes so you can connect using your new credentials.

    ```azurecli
    az vmss reimage -g <nodeRG> -n <vmssName> --instance-id <affectedNodeInstanceId>
    ```

5. Use `kubectl debug` to connect to another node.

    ```azurecli
    kubectl debug node/<nodeName> -it --image=mcr.microsoft.com/dotnet/runtime-deps:6.0
    ```

6. Open a second terminal to use port forwarding to connect the debug pod to your local computer.

    ```azurecli
    kubectl port-forward <debugPodName> 2022:22
    ```

7. Open a third terminal to get the `INTERNAL-IP` of the affected node to initiate the SSH connection. You can get this with `kubectl get nodes -o wide`. Once you have it, use the following command to connect.

    ```azurecli
     ssh -o 'ProxyCommand ssh -p 2022 -W %h:%p azureuser@127.0.0.1' azureuser@<affectedNodeIp>
    ```

8. Enter your password.

### Remove SSH access

When done, `exit` the SSH session, stop any port forwarding, and then `exit` the interactive container session. After the interactive container session closes, delete the pod used for SSH access using the `kubectl delete pod` command.

```bash
kubectl delete pod node-debugger-aks-nodepool1-12345678-vmss000000-bkmmx
```

## Update SSH key on an existing AKS cluster (preview)

### Prerequisites

* Before you start, ensure the Azure CLI is installed and configured. If you need to install or upgrade, see [Install Azure CLI][install-azure-cli].
* The aks-preview extension version 0.5.111 or later. To learn how to install an Azure extension, see [How to install extensions][how-to-install-azure-extensions].

> [!NOTE]
> Updating of the SSH key is supported on Azure virtual machine scale sets with AKS clusters.

Use the [az aks update][az-aks-update] command to update the SSH key on the cluster. This operation updates the key on all node pools. You can either specify the key or a key file using the `--ssh-key-value` argument.

```azurecli
az aks update --name myAKSCluster --resource-group MyResourceGroup --ssh-key-value <new SSH key value or SSH key file>
```

Examples:
In the following example, you can specify the new SSH key value for the `--ssh-key-value` argument.

```azurecli
az aks update --name myAKSCluster --resource-group MyResourceGroup --ssh-key-value 'ssh-rsa AAAAB3Nza-xxx'
```

In the following example, you specify a SSH key file.

```azurecli
az aks update --name myAKSCluster --resource-group MyResourceGroup --ssh-key-value .ssh/id_rsa.pub
```

> [!IMPORTANT]
> During this operation, all virtual machine scale set instances are upgraded and re-imaged to use the new SSH key.


## Next steps

If you need more troubleshooting data, you can [view the kubelet logs][view-kubelet-logs] or [view the Kubernetes master node logs][view-master-logs].

<!-- INTERNAL LINKS -->
[view-kubelet-logs]: kubelet-logs.md
[view-master-logs]: monitor-aks-reference.md#resource-logs
[install-azure-cli]: /cli/azure/install-azure-cli
[aks-windows-rdp]: rdp.md
[ssh-nix]: ../virtual-machines/linux/mac-create-ssh-keys.md
[ssh-windows]: ../virtual-machines/linux/ssh-from-windows.md
[ssh-linux-kubectl-debug]: #create-an-interactive-shell-connection-to-a-linux-node
[az-aks-update]: /cli/azure/aks#az-aks-update
[how-to-install-azure-extensions]: /cli/azure/azure-cli-extensions-overview#how-to-install-extensions

                                              
