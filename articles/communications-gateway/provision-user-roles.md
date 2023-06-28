---
title: Set up user roles for Azure Communications Gateway
description: Learn how to configure the user roles required to deploy, manage and monitor your Azure Communications Gateway
author: rcdun
ms.author: rdunstan
ms.service: communications-gateway
ms.topic: how-to
ms.date: 06/02/2023
---

# Set up user roles for Azure Communications Gateway

This article will guide you through how to configure the permissions required for staff in your organization to:

- Deploy Azure Communications Gateway through the portal
- Raise customer support requests (support tickets)
- Monitor Azure Communications Gateway
- Use the API Bridge Number Management Portal for provisioning

## Prerequisites

Familiarize yourself with the Azure user roles relevant to Azure Communications Gateway by reading [Azure roles, Azure AD roles, and classic subscription administrator roles](../role-based-access-control/rbac-and-directory-admin-roles.md).

A list of all available defined Azure roles is available in [Azure built-in roles](../role-based-access-control/built-in-roles.md).

## 1. Understand the user roles required for Azure Communications Gateway

Your staff might need different user roles, depending on the tasks they need to carry out.

|Task  | Required user roles or access |
|---------|---------|
| Deploying Azure Communications Gateway |**Contributor** access to your subscription|
| Raising support requests |**Owner**, **Contributor** or **Support Request Contributor** access to your subscription or a custom role with `Microsoft.Support/*` access at the subscription level|
|Monitoring logs and metrics | **Reader** access to your subscription|
|Using the API Bridge Number Management Portal|**NumberManagement.Read**, **NumberManagement.Write**, **PartnerSettings.Read**, and **PartnerSettings.Write** permissions for the Project Synergy enterprise application and **Reader** permissions to the Azure portal for your subscription|

## 2. Configure user roles

You need to use the Azure portal to configure user roles.

### 2.1 Prepare to assign a user role

1. Read through [Steps to assign an Azure role](../role-based-access-control/role-assignments-steps.md) and ensure that you:
    - Know who needs access.
    - Know the appropriate user role or roles to assign them.
    - Are signed in with a user that is assigned a role that has role assignments write permission, such as **Owner** or **User Access Administrator** for the subscription.
1. If you're managing access to the API Bridge Number Management Portal, ensure that you're signed in with a user that can change permissions for enterprise applications. For example, you could be a Global Administrator, Cloud Application Administrator or Application Administrator. For more information, see [Assign users and groups to an application](../active-directory/manage-apps/assign-user-or-group-access-portal.md).

### 2.2 Assign a user role

1. Follow the steps in [Assign a user role using the Azure portal](../role-based-access-control/role-assignments-portal.md) to assign the permissions you determined in [1. Understand the user roles required for Azure Communications Gateway](#1-understand-the-user-roles-required-for-azure-communications-gateway).
1. If you're managing access to the API Bridge Number Management Portal, follow [Assign users and groups to an application](../active-directory/manage-apps/assign-user-or-group-access-portal.md) to assign **NumberManagement.Read**, **NumberManagement.Write**, **PartnerSettings.Read**, and **PartnerSettings.Write** permissions for each user in the Project Synergy application.

## Next steps

- Learn how to remove access to the Azure Communications Gateway subscription by [removing Azure role assignments](../role-based-access-control/role-assignments-remove.md).
