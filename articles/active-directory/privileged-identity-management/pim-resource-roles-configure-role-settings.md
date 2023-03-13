---
title: Configure Azure resource role settings in PIM - Azure Active Directory
description: Learn how to configure Azure resource role settings in Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: amsliu
manager: amycolannino
ms.service: active-directory
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 01/27/2023
ms.author: amsliu
ms.custom: pim
ms.collection: M365-identity-device-management
---

# Configure Azure resource role settings in Privileged Identity Management

In Privileged Identity Management (PIM) in Azure Active Directory (Azure AD), part of Microsoft Entra, role settings define role assignment properties: MFA and approval requirements for activation, assignment maximum duration, notification settings, and more. Use the following steps to configure role settings and setup the approval workflow to specify who can approve or deny requests to elevate privilege.

You need to have Owner or User Access Administrator role to manage PIM role settings for the resource. Role settings are defined per role and per resource: all assignments for the same role follow the same role settings. Role settings of one role are independent from role settings of another role. Role settings of one resource are independent from role settings of another resource, and role settings configured on a higher level, such as "Subscription" for example, are not inherited on a lower level, such as "Resource Group" for example.

PIM role settings are also known as “PIM Policies”.

## Open role settings

Follow these steps to open the settings for an Azure resource role.

1. [Sign in to the Azure portal](https://portal.azure.com/)

1. Select **Azure AD Privileged Identity Management -> Azure Resources**. On this page you can see list of Azure resources discovered in PIM. Use Resource type filter to select all required resource types.

     :::image type="content" source="media/pim-resource-roles-configure-role-settings/resources-list.png" alt-text="Screenshot of the list of Azure resources discovered in PIM." lightbox="media/pim-resource-roles-configure-role-settings/resources-list.png":::

1. Select the resource that you need to configure PIM role settings for.

1. Select **Settings**. View list of PIM policies for a selected resource.

    :::image type="content" source="media/pim-resource-roles-configure-role-settings/resources-role-settings.png" alt-text="Screenshot of the list of PIM policies for a selected resource." lightbox="media/pim-resource-roles-configure-role-settings/resources-role-settings.png":::

1. Select the role or policy that you want to configure.

1.	Select Edit to update role settings.

1.	Once finished, select Update.

## Role settings 

### Activation maximum duration

Use the **Activation maximum duration** slider to set the maximum time, in hours, that an activation request for a role assignment remains active before it expires. This value can be from one to 24 hours.

### On activation, require multi-factor authentication 

You can require users who are eligible for a role to prove who they are using Azure AD Multi-Factor Authentication before they can activate. Multi-factor authentication ensures that the user is who they say they are with reasonable certainty. Enforcing this option protects critical resources in situations when the user account might have been compromised.

User may not be prompted for multi-factor authentication if they authenticated with strong credential or provided multi-factor authentication earlier in this session.

For more information, see [Multifactor authentication and Privileged Identity Management](pim-how-to-require-mfa.md).

### On activation, require Azure AD Conditional Access authentication context (Public Preview)

You can require users who are eligible for a role to satisfy Conditional Access policy requirements: use specific authentication method enforced through Authentication Strengths, elevate the role from Intune compliant device, comply with Terms of Use, and more. 

To enforce this requirement, you need to:

1.	Create Conditional Access authentication context.
1.	Configure Conditional Access policy that would enforce requirements for this authentication context.
1.	Configure authentication context in PIM settings for the role.

:::image type="content" source="media/pim-resource-roles-configure-role-settings/resources-role-setting-details.png" alt-text="Screenshot of the Edit role settings Attestation Reader page." lightbox="media/pim-resource-roles-configure-role-settings/resources-role-setting-details.png":::

To learn more about Conditional Access authentication context, see [Conditional Access: Cloud apps, actions, and authentication context](../conditional-access/concept-conditional-access-cloud-apps.md#authentication-context).

### Require justification on activation

You can require users to enter a business justification when they activate the eligible assignment.

### Require ticket information on activation

You can require users to enter a support ticket number when they activate the eligible assignment. This is information-only field and correlation with information in any ticketing system is not enforced.

### Require approval to activate

You can require approval for activation of eligible assignment. Approver doesn’t have to have any roles. When using this option, you have to select at least one approver (we recommend to select at least two approvers), there are no default approvers.

To learn more about approvals, see [Approve or deny requests for Azure AD roles in Privileged Identity Management](azure-ad-pim-approval-workflow.md).

### Assignment duration

You can choose from two assignment duration options for each assignment type (eligible and active) when you configure settings for a role. These options become the default maximum duration when a user is assigned to the role in Privileged Identity Management.

You can choose one of these **eligible** assignment duration options:

| Setting | Description |
| --- | --- |
| Allow permanent eligible assignment | Resource administrators can assign permanent eligible assignment. |
| Expire eligible assignment after | Resource administrators can require that all eligible assignments have a specified start and end date. |

And, you can choose one of these **active** assignment duration options:

| Setting | Description |
| --- | --- |
| Allow permanent active assignment | Resource administrators can assign permanent active assignment. |
| Expire active assignment after | Resource administrators can require that all active assignments have a specified start and end date. |

> [!NOTE]
> All assignments that have a specified end date can be renewed by Global admins and Privileged role admins. Also, users can initiate self-service requests to [extend or renew role assignments](pim-resource-roles-renew-extend.md).

### Require multi-factor authentication on active assignment

You can require that administrator provides multi-factor authentication when they create an active (as opposed to eligible) assignment. Privileged Identity Management can't enforce multi-factor authentication when the user uses their role assignment because they are already active in the role from the time that it is assigned.

Administrator may not be prompted for multi-factor authentication if they authenticated with strong credential or provided multi-factor authentication earlier in this session.

### Require justification on active assignment

You can require that users enter a business justification when they create an active (as opposed to eligible) assignment.

In the **Notifications** tab on the role settings page, Privileged Identity Management enables granular control over who receives notifications and which notifications they receive.

-	**Turning off an email**</br>
You can turn off specific emails by clearing the default recipient check box and deleting any other recipients.
-	**Limit emails to specified email addresses**</br>
You can turn off emails sent to default recipients by clearing the default recipient check box. You can then add other email addresses as recipients. If you want to add more than one email address, separate them using a semicolon (;).
-	**Send emails to both default recipients and more recipients**</br>
You can send emails to both default recipient and another recipient by selecting the default recipient checkbox and adding email addresses for other recipients.
-	**Critical emails only**</br>
For each type of email, you can select the check box to receive critical emails only. What this means is that Privileged Identity Management will continue to send emails to the specified recipients only when the email requires an immediate action. For example, emails asking users to extend their role assignment will not be triggered while emails requiring admins to approve an extension request will be triggered.

## Next steps

- [Assign Azure resource roles in Privileged Identity Management](pim-resource-roles-assign-roles.md)
- [Configure security alerts for Azure resource roles in Privileged Identity Management](pim-resource-roles-configure-alerts.md)
