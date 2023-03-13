---
title: Manage Enterprise IoT plans on Azure subscriptions
description: Manage Defender for IoT plans for Enterprise IoT monitoring on your Azure subscriptions.
ms.date: 07/06/2022
ms.topic: how-to
ms.custom: enterprise-iot
---

# Manage Defender for IoT plans for Enterprise IoT security monitoring

Enterprise IoT security monitoring with Defender for IoT is managed by an Enterprise IoT plan on your Azure subscription. While you can view your plan in Microsoft Defender for IoT, onboarding and canceling a plan is done with [Microsoft Defender for Endpoint](/microsoft-365/security/defender-endpoint/) in Microsoft 365 Defender.

For each monthly price plan, you'll be asked to define the number of [committed devices](billing.md#defender-for-iot-committed-devices). Committed devices are the approximate number of devices that will be monitored in your enterprise.

For information about OT networks, see [Manage Defender for IoT plans for OT security monitoring](how-to-manage-subscriptions.md).

## Prerequisites

Before performing the procedures in this article, make sure that you have:

- A Microsoft Defender for Endpoint P2 license

- An Azure subscription. If you need to, [sign up for a free account](https://azure.microsoft.com/free/).

- The following user roles:

    - **In Azure Active Directory**: [Global administrator](../../active-directory/roles/permissions-reference.md#global-administrator) for your Microsoft 365 tenant

    - **In Azure RBAC**:  [Security admin](../../role-based-access-control/built-in-roles.md#security-admin), [Contributor](../../role-based-access-control/built-in-roles.md#contributor), or [Owner](../../role-based-access-control/built-in-roles.md#owner) for the Azure subscription that you'll be using for the integration

## Calculate committed devices for Enterprise IoT monitoring

If you're adding an Enterprise IoT plan with a monthly commitment, you'll be asked to enter the number of committed devices.

We recommend that you make an initial estimate of your committed devices when onboarding your plan. You can skip this procedure if you're adding a [trial plan](billing.md#free-trial).

**To calculate committed devices:**:

1. In the navigation pane of the [https://security.microsoft.com](https://security.microsoft.com/) portal, select **Assets** \> **Devices** to open the **Device inventory** page.

1. Add the total number of devices listed on both the **Network devices** and **IoT devices** tabs.

    For example:

    :::image type="content" source="media/how-to-manage-subscriptions/eiot-calculate-devices.png" alt-text="Screenshot of network device and IoT devices in the device inventory in Microsoft Defender for Endpoint." lightbox="media/how-to-manage-subscriptions/eiot-calculate-devices.png":::

1. Round up your total to a multiple of 100.

For example:

- In the Microsoft 365 Defender **Device inventory**, you have *473* network devices and *1206* IoT devices.
- Added together, the total is *1679* devices.
- Rounded up to a multiple of 100 is **1700**.

Use **1700** as the estimated number of committed devices.

For more information, see the [Defender for Endpoint Device discovery overview](/microsoft-365/security/defender-endpoint/device-discovery).

> [!NOTE]
> Devices listed on the **Computers & Mobile** tab, including those managed by Defender for Endpoint or otherwise, are not included in the number of committed devices for Defender for IoT.

## Onboard an Enterprise IoT plan

This procedure describes how to add an Enterprise IoT plan to your Azure subscription from Microsoft 365 Defender.

**To add an Enterprise IoT plan**:

1. In the navigation pane of the [https://security.microsoft.com](https://security.microsoft.com/) portal, select **Settings** \> **Device discovery** \> **Enterprise IoT**.

1. Select the following options for your plan:

    - **Select an Azure subscription**: Select the Azure subscription that you want to use for the integration. You'll need a [Security admin](../../role-based-access-control/built-in-roles.md#security-admin), [Contributor](../../role-based-access-control/built-in-roles.md#contributor), or [Owner](../../role-based-access-control/built-in-roles.md#owner) role for the subscription.

        > [!TIP]
        > If your subscription isn't listed, check your account details and confirm your permissions with the subscription owner.

    - **Price plan**: Select a trial or monthly commitment.

        Microsoft Defender for IoT provides a 30-day free trial for evaluation purposes, with an unlimited number of devices. For more information, see the [Microsoft Defender for IoT pricing page](https://azure.microsoft.com/pricing/details/iot-defender/).

        Monthly commitments require that you enter the number of committed devices that you'd calculated earlier.

1. Select the **I accept the terms and conditions** option and then select **Save**.

    For example:

    :::image type="content" source="media/enterprise-iot/defender-for-endpoint-onboard.png" alt-text="Screenshot of the Enterprise IoT tab in Defender for Endpoint." lightbox="media/enterprise-iot/defender-for-endpoint-onboard.png":::

After you've onboarded your plan, you'll see it listed in [Defender for IoT](https://ms.portal.azure.com/#view/Microsoft_Azure_IoT_Defender/IoTDefenderDashboard/~/Getting_started) in the Azure portal. Go to the Defender for IoT **Plans and pricing** page and find your subscription with the new **Enterprise IoT** plan listed. For example:

:::image type="content" source="media/enterprise-iot/eiot-plan-in-azure.png" alt-text="Screenshot of an Enterprise IoT plan showing in the Defender for IoT Plans and pricing page.":::

## Edit your Enterprise IoT plan

To edit your plan, such as to edit your commitment level or the number of committed devices, first [cancel the plan](#cancel-your-enterprise-iot-plan) and then [onboard a new plan](#onboard-an-enterprise-iot-plan).

## Cancel your Enterprise IoT plan

You'll need to cancel your plan if you want to edit the details of your plan, such as the price plan or the number of committed devices, or if you no longer need the service.

You'd also need to cancel your plan and onboard again if you need to work with a new payment entity or Azure subscription.

**To cancel your Enterprise IoT plan**:

1. In the navigation pane of the [https://security.microsoft.com](https://security.microsoft.com/) portal, select **Settings** \> **Device discovery** \> **Enterprise IoT**.

1. Select **Cancel plan**. For example:

    :::image type="content" source="media/enterprise-iot/defender-for-endpoint-cancel-plan.png" alt-text="Screenshot of the Cancel plan option on the Microsoft 365 Defender page.":::

After you cancel your plan, the integration stops and you'll no longer get added security value in Microsoft 365 Defender, or detect new Enterprise IoT devices in Defender for IoT.

The cancellation takes effect one hour after confirming the change.  This change will appear on your next monthly statement, and you will be charged based on the length of time the plan was in effect.

If you're canceling your plan as part of an [editing procedure](#edit-your-enterprise-iot-plan), make sure to [onboard a new plan](#onboard-an-enterprise-iot-plan) back with the new details.

> [!IMPORTANT]
>
> If you've [registered an Enterprise IoT network sensor](eiot-sensor.md) (Public preview), device data collected by the sensor remains in your Microsoft 365 Defender instance. If you're canceling the Enterprise IoT plan because you no longer need the service, make sure to manually delete data from Microsoft 365 Defender as needed.

## Next steps

For more information, see:

- [Defender for IoT subscription billing](billing.md)

- [Manage sensors with Defender for IoT in the Azure portal](how-to-manage-sensors-on-the-cloud.md)

- [Create an additional Azure subscription](../../cost-management-billing/manage/create-subscription.md)

- [Upgrade your Azure subscription](../../cost-management-billing/manage/upgrade-azure-subscription.md)