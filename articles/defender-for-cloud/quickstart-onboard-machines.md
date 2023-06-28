---
title: Connect your non-Azure machines to Microsoft Defender for Cloud with Azure Arc
description: Learn how to connect your non-Azure machines to Microsoft Defender for Cloud
ms.topic: quickstart
ms.date: 06/01/2023
author: dcurwin
ms.author: dacurwin
zone_pivot_groups: non-azure-machines
ms.custom: mode-other
---
# Quickstart: Connect your non-Azure machines to Microsoft Defender for Cloud with Azure Arc

Defender for Cloud can monitor the security posture of your non-Azure computers, but first you need to connect them to Azure.

You can connect your non-Azure computers in any of the following ways:

- Onboarding with Azure Arc
  - [Using Azure Arc-enabled servers](#add-non-azure-machines-with-azure-arc) (**recommended**)
  - [From Defender for Cloud's pages in the Azure portal](#add-non-azure-machines-from-the-azure-portal) (**Getting started** and **Inventory**)
- [Onboarding directly with Defender for Endpoint](onboard-machines-with-defender-for-endpoint.md)

Each of these is described on this page.

> [!TIP]
> If you're connecting machines from other cloud providers, see [Connect your AWS accounts](quickstart-onboard-aws.md) or [Connect your GCP projects](quickstart-onboard-gcp.md). Defender for Cloud's multicloud connectors for AWS and GCP transparently handles the Azure Arc deployment for you.

::: zone pivot="azure-arc"

## Add non-Azure machines with Azure Arc

The preferred way of adding your non-Azure machines to Microsoft Defender for Cloud is with [Azure Arc-enabled servers](../azure-arc/servers/overview.md).

A machine with Azure Arc-enabled servers becomes an Azure resource and - when you've installed the Log Analytics agent on it - appears in Defender for Cloud with recommendations like your other Azure resources.

In addition, Azure Arc-enabled servers provides enhanced capabilities such as the option to enable guest configuration policies on the machine, simplify deployment with other Azure services, and more. For an overview of the benefits, see [Supported cloud operations](../azure-arc/servers/overview.md#supported-cloud-operations).

> [!NOTE]
> Defender for Cloud's auto-deploy tools for deploying the Log Analytics agent works with machines running Azure Arc however this capability is currently in preview . When you've connected your machines using Azure Arc, use the relevant Defender for Cloud recommendation to deploy the agent and benefit from the full range of protections offered by Defender for Cloud:
>
> - [Log Analytics agent should be installed on your Linux-based Azure Arc machines](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/720a3e77-0b9a-4fa9-98b6-ddf0fd7e32c1)
> - [Log Analytics agent should be installed on your Windows-based Azure Arc machines](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/27ac71b1-75c5-41c2-adc2-858f5db45b08)

Learn more about [Azure Arc-enabled servers](../azure-arc/servers/overview.md).

**To deploy Azure Arc:**

- For one machine, follow the instructions in [Quickstart: Connect hybrid machines with Azure Arc-enabled servers](../azure-arc/servers/learn/quick-enable-hybrid-vm.md).
- To connect multiple machines at scale to Azure Arc-enabled servers, see [Connect hybrid machines to Azure at scale](../azure-arc/servers/onboard-service-principal.md)

::: zone-end

::: zone pivot="azure-portal"

## Add non-Azure machines from the Azure portal

1. From Defender for Cloud's menu, open the **Getting started** page.
1. Select the **Get started** tab.
1. Below **Add non-Azure servers**, select **Configure** .

    :::image type="content" source="./media/quickstart-onboard-machines/onboarding-get-started-tab.png" alt-text="Get Started tab in the Getting started page." lightbox="./media/quickstart-onboard-machines/onboarding-get-started-tab.png":::

    > [!TIP]
    > You can also open add machines from the **inventory** page's **Add non-Azure servers** button.
    >
    > :::image type="content" source="./media/quickstart-onboard-machines/onboard-inventory.png" alt-text="Adding non-Azure machines from the asset inventory page.":::

    A list of your Log Analytics workspaces is shown. The list includes, if applicable, the default workspace created for you by Defender for Cloud when automatic provisioning was enabled. Select this workspace or another workspace you want to use.

    You can add computers to an existing workspace or create a new workspace.

1. Optionally, to create a new workspace, select  **Create new workspace**.

1. From the list of workspaces, select **Add Servers** for the relevant workspace.

    The **Agents management** page appears.

    From here, choose the relevant procedure below depending on the type of machines you're onboarding:

    - [Onboard your Linux machines](#onboard-your-linux-machines)
    - [Onboard your Windows machines](#onboard-your-windows-machines)

### Onboard your Linux machines

To add Linux machines, you need the WGET command from the **Agents management** page.

1. From the **Agents management** page, copy the **WGET** command into Notepad. Save this file to a location that can be accessible from your Linux computer.
1. On your Linux computer, open the file with the WGET command. Select the entire content and copy and paste it into a terminal console.
1. When the installation completes, you can validate that the `omsagent` is installed by running the `pgrep` command. The command will return the `omsagent` PID.

    The logs for the Agent can be found at: `/var/opt/microsoft/omsagent/\<workspace id>/log/`. It might take up to 30 minutes for the new Linux machine to appear in Defender for Cloud.

### Onboard your Windows machines

To add Windows machines, you need the information on the **Agents management** page and to download the appropriate agent file (32/64-bit).

1. Select the **Download Windows Agent** link applicable to your computer processor type to download the setup file.
1. From the **Agents management** page, copy the **Workspace ID** and **Primary Key** into Notepad.
1. Copy the downloaded setup file to the target computer and run it.
1. Follow the installation wizard (**Next**, **I Agree**, **Next**, **Next**).
    1. On the **Azure Log Analytics** page, paste the **Workspace ID** and **Workspace Key (Primary Key)** that you copied into Notepad.
    1. If the computer should report to a Log Analytics workspace in Azure Government cloud, select **Azure US Government** from the **Azure Cloud** dropdown list.
    1. If the computer needs to communicate through a proxy server to the Log Analytics service, select **Advanced** and provide the URL and port number of the proxy server.
    1. When you've entered all of the configuration settings, select **Next**.
    1. From the **Ready to Install** page, review the settings to be applied and select **Install**.
    1. On the **Configuration completed successfully** page, select **Finish**.

When complete, the **Microsoft Monitoring agent** appears in **Control Panel**. You can review your configuration there and verify that the agent is connected.

For further information on installing and configuring the agent, see [Connect Windows machines](../azure-monitor/agents/agent-windows.md#install-the-agent).

::: zone-end

## Verifying

Congratulations! Now you can see your Azure and non-Azure machines together in one place. Open the [asset inventory page](asset-inventory.md) and filter to the relevant resource types. These icons distinguish the types:

  ![Defender for Cloud icon for non-Azure machine.](./media/quickstart-onboard-machines/security-center-monitoring-icon1.png) Non-Azure machine

  ![Defender for Cloud icon for Azure machine.](./media/quickstart-onboard-machines/security-center-monitoring-icon2.png) Azure VM

  ![Defender for Cloud icon for Azure Arc server.](./media/quickstart-onboard-machines/arc-enabled-machine-icon.png) Azure Arc-enabled server

## Next steps

This page showed you how to add your non-Azure machines to Microsoft Defender for Cloud. To monitor their status, use the inventory tools as explained in the following page:

- [Explore and manage your resources with asset inventory](asset-inventory.md)
