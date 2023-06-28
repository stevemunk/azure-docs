---
title: Manage access to Log Analytics workspaces
description: This article explains how you can manage access to data stored in a Log Analytics workspace in Azure Monitor by using resource, workspace, or table-level permissions.
ms.topic: conceptual
author: guywi-ms
ms.author: guywild
ms.reviewer: MeirMen
ms.date: 10/06/2022
ms.custom: devx-track-azurepowershell

---

# Manage access to Log Analytics workspaces

 The data in a Log Analytics workspace that you can access is determined by a combination of the following factors:

- The settings on the workspace itself.
- The access to resources sending data to the workspace.
- The method used to access the workspace.

This article describes how access is managed and how to perform any required configuration.

## Overview

The factors that define the data you can access are described in the following table. Each factor is further described in the sections that follow.

| Factor | Description |
|:---|:---|
| [Access mode](#access-mode) | Method used to access the workspace. Defines the scope of the data available and the access control mode that's applied. |
| [Access control mode](#access-control-mode) | Setting on the workspace that defines whether permissions are applied at the workspace or resource level. |
| [Azure role-based access control (RBAC)](#azure-rbac) | Permissions applied to individuals or groups of users for the workspace or resource sending data to the workspace. Defines what data you have access to. |
| [Table-level Azure RBAC](#set-table-level-read-access) | Optional permissions that define specific data types in the workspace that you can access. Can be applies to all access modes or access control modes. |

## Access mode

The *access mode* refers to how you access a Log Analytics workspace and defines the data you can access during the current session. The mode is determined according to the [scope](scope.md) you select in Log Analytics.

There are two access modes:

- **Workspace-context**: You can view all logs in the workspace for which you have permission. Queries in this mode are scoped to all data in tables that you have access to in the workspace. This access mode is used when logs are accessed with the workspace as the scope, such as when you select **Logs** on the **Azure Monitor** menu in the Azure portal.
 - **Resource-context**: When you access the workspace for a particular resource, resource group, or subscription, such as when you select **Logs** from a resource menu in the Azure portal, you can view logs for only resources in all tables that you have access to. Queries in this mode are scoped to only data associated with that resource. This mode also enables granular Azure RBAC. Workspaces use a resource-context log model where every log record emitted by an Azure resource is automatically associated with this resource.

Records are only available in resource-context queries if they're associated with the relevant resource. To check this association, run a query and verify that the [_ResourceId](./log-standard-columns.md#_resourceid) column is populated.

There are known limitations with the following resources:

- **Computers outside of Azure**: Resource-context is only supported with [Azure Arc for servers](../../azure-arc/servers/index.yml).
- **Application Insights**: Supported for resource-context only when using a [workspace-based Application Insights resource](../app/create-workspace-resource.md).
- **Azure Service Fabric**

### Compare access modes

The following table summarizes the access modes:

| Issue | Workspace-context | Resource-context |
|:---|:---|:---|
| Who is each model intended for? | Central administration.<br>Administrators who need to configure data collection and users who need access to a wide variety of resources. Also currently required for users who need to access logs for resources outside of Azure. | Application teams.<br>Administrators of Azure resources being monitored. Allows them to focus on their resource without filtering. |
| What does a user require to view logs? | Permissions to the workspace.<br>See "Workspace permissions" in [Manage access using workspace permissions](./manage-access.md#azure-rbac). | Read access to the resource.<br>See "Resource permissions" in [Manage access using Azure permissions](./manage-access.md#azure-rbac). Permissions can be inherited from the resource group or subscription or directly assigned to the resource. Permission to the logs for the resource will be automatically assigned. The user doesn't require access to the workspace.|
| What is the scope of permissions? | Workspace.<br>Users with access to the workspace can query all logs in the workspace from tables they have permissions to. See [Set table-level read access](./manage-access.md#set-table-level-read-access). | Azure resource.<br>Users can query logs for specific resources, resource groups, or subscriptions they have access to in any workspace, but they can't query logs for other resources. |
| How can a user access logs? | On the **Azure Monitor** menu, select **Logs**.<br><br>Select **Logs** from **Log Analytics workspaces**.<br><br>From Azure Monitor [workbooks](../best-practices-analysis.md#azure-workbooks). | Select **Logs** on the menu for the Azure resource. Users will have access to data for that resource.<br><br>Select **Logs** on the **Azure Monitor** menu. Users will have access to data for all resources they have access to.<br><br>Select **Logs** from **Log Analytics workspaces**, if users have access to the workspace.<br><br>From Azure Monitor [workbooks](../best-practices-analysis.md#azure-workbooks). |

## Access control mode

The *access control mode* is a setting on each workspace that defines how permissions are determined for the workspace.

* **Require workspace permissions**. This control mode doesn't allow granular Azure RBAC. To access the workspace, the user must be [granted permissions to the workspace](#azure-rbac) or to [specific tables](#set-table-level-read-access).

    If a user accesses the workspace in [workspace-context mode](#access-mode), they have access to all data in any table they've been granted access to. If a user accesses the workspace in [resource-context mode](#access-mode), they have access to only data for that resource in any table they've been granted access to.

    This setting is the default for all workspaces created before March 2019.

* **Use resource or workspace permissions**. This control mode allows granular Azure RBAC. Users can be granted access to only data associated with resources they can view by assigning Azure `read` permission.

    When a user accesses the workspace in [workspace-context mode](#access-mode), workspace permissions apply. When a user accesses the workspace in [resource-context mode](#access-mode), only resource permissions are verified, and workspace permissions are ignored. Enable Azure RBAC for a user by removing them from workspace permissions and allowing their resource permissions to be recognized.

    This setting is the default for all workspaces created after March 2019.

    > [!NOTE]
    > If a user has only resource permissions to the workspace, they can only access the workspace by using resource-context mode assuming the workspace access mode is set to **Use resource or workspace permissions**.

### Configure access control mode for a workspace

# [Azure portal](#tab/portal)

View the current workspace access control mode on the **Overview** page for the workspace in the **Log Analytics workspace** menu.

![Screenshot that shows the workspace access control mode.](media/manage-access/view-access-control-mode.png)

Change this setting on the **Properties** page of the workspace. If you don't have permissions to configure the workspace, changing the setting is disabled.

![Screenshot that shows changing workspace access mode.](media/manage-access/change-access-control-mode.png)

# [PowerShell](#tab/powershell)

Use the following command to view the access control mode for all workspaces in the subscription:

```powershell
Get-AzResource -ResourceType Microsoft.OperationalInsights/workspaces -ExpandProperties | foreach {$_.Name + ": " + $_.Properties.features.enableLogAccessUsingOnlyResourcePermissions}
```

The output should resemble the following:

```
DefaultWorkspace38917: True
DefaultWorkspace21532: False
```

A value of `False` means the workspace is configured with *workspace-context* access mode. A value of `True` means the workspace is configured with *resource-context* access mode.

> [!NOTE]
> If a workspace is returned without a Boolean value and is blank, this result also matches the results of a `False` value.
>

Use the following script to set the access control mode for a specific workspace to *resource-context* permission:

```powershell
$WSName = "my-workspace"
$Workspace = Get-AzResource -Name $WSName -ExpandProperties
if ($Workspace.Properties.features.enableLogAccessUsingOnlyResourcePermissions -eq $null)
    { $Workspace.Properties.features | Add-Member enableLogAccessUsingOnlyResourcePermissions $true -Force }
else
    { $Workspace.Properties.features.enableLogAccessUsingOnlyResourcePermissions = $true }
Set-AzResource -ResourceId $Workspace.ResourceId -Properties $Workspace.Properties -Force
```

Use the following script to set the access control mode for all workspaces in the subscription to *resource-context* permission:

```powershell
Get-AzResource -ResourceType Microsoft.OperationalInsights/workspaces -ExpandProperties | foreach {
if ($_.Properties.features.enableLogAccessUsingOnlyResourcePermissions -eq $null)
    { $_.Properties.features | Add-Member enableLogAccessUsingOnlyResourcePermissions $true -Force }
else
    { $_.Properties.features.enableLogAccessUsingOnlyResourcePermissions = $true }
Set-AzResource -ResourceId $_.ResourceId -Properties $_.Properties -Force
}
```

# [Resource Manager](#tab/arm)

To configure the access mode in an Azure Resource Manager template, set the **enableLogAccessUsingOnlyResourcePermissions** feature flag on the workspace to one of the following values:

* **false**: Set the workspace to *workspace-context* permissions. This setting is the default if the flag isn't set.
* **true**: Set the workspace to *resource-context* permissions.

---

## Azure RBAC

Access to a workspace is managed by using [Azure RBAC](../../role-based-access-control/role-assignments-portal.md). To grant access to the Log Analytics workspace by using Azure permissions, follow the steps in [Assign Azure roles to manage access to your Azure subscription resources](../../role-based-access-control/role-assignments-portal.md).

### Workspace permissions

Each workspace can have multiple accounts associated with it. Each account can have access to multiple workspaces. The following table lists the Azure permissions for different workspace actions:

|Action |Azure permissions needed |Notes |
|-------|-------------------------|------|
| Change the pricing tier. | `Microsoft.OperationalInsights/workspaces/*/write` | 
| Create a workspace in the Azure portal. | `Microsoft.Resources/deployments/*` <br> `Microsoft.OperationalInsights/workspaces/*` |
| View workspace basic properties and enter the workspace pane in the portal. | `Microsoft.OperationalInsights/workspaces/read` |
| Query logs by using any interface. | `Microsoft.OperationalInsights/workspaces/query/read` |
| Access all log types by using queries. | `Microsoft.OperationalInsights/workspaces/query/*/read` |
| Access a specific log table - legacy method | `Microsoft.OperationalInsights/workspaces/query/<table_name>/read` |
| Read the workspace keys to allow sending logs to this workspace. | `Microsoft.OperationalInsights/workspaces/sharedKeys/action` |
| Add and remove monitoring solutions. | `Microsoft.Resources/deployments/*` <br> `Microsoft.OperationalInsights/*` <br> `Microsoft.OperationsManagement/*` <br> `Microsoft.Automation/*` <br> `Microsoft.Resources/deployments/*/write`<br><br>These permissions need to be granted at resource group or subscription level. |
| View data in the **Backup** and **Site Recovery** solution tiles. | Administrator/Co-administrator<br><br>Accesses resources deployed by using the classic deployment model. |
| Run a search job. | `Microsoft.OperationalInsights/workspaces/tables/write` <br> `Microsoft.OperationalInsights/workspaces/searchJobs/write`|
| Restore data from archived table. | `Microsoft.OperationalInsights/workspaces/tables/write` <br> `Microsoft.OperationalInsights/workspaces/restoreLogs/write`|

### Built-in roles

Assign users to these roles to give them access at different scopes:

* **Subscription**: Access to all workspaces in the subscription
* **Resource group**: Access to all workspaces in the resource group
* **Resource**: Access to only the specified workspace

Create assignments at the resource level (workspace) to assure accurate access control. Use [custom roles](../../role-based-access-control/custom-roles.md) to create roles with the specific permissions needed.

> [!NOTE]
> To add and remove users to a user role, you must have `Microsoft.Authorization/*/Delete` and `Microsoft.Authorization/*/Write` permission.

#### Log Analytics Reader

Members of the Log Analytics Reader role can view all monitoring data and monitoring settings, including the configuration of Azure diagnostics on all Azure resources.

Members of the Log Analytics Reader role can:

- View and search all monitoring data.
- View monitoring settings, including viewing the configuration of Azure diagnostics on all Azure resources.

The Log Analytics Reader role includes the following Azure actions:

| Type    | Permission | Description |
| ------- | ---------- | ----------- |
| Action | `*/read`   | Ability to view all Azure resources and resource configuration.<br>Includes viewing:<br>- Virtual machine extension status.<br>- Configuration of Azure diagnostics on resources.<br>- All properties and settings of all resources.<br><br>For workspaces, allows full unrestricted permissions to read the workspace settings and query data. See more granular options in the preceding list. |
| Action | `Microsoft.Support/*` | Ability to open support cases. |
|Not Action | `Microsoft.OperationalInsights/workspaces/sharedKeys/read` | Prevents reading of workspace key required to use the data collection API and to install agents. This prevents the user from adding new resources to the workspace. |
| Action | `Microsoft.OperationalInsights/workspaces/analytics/query/action` | Deprecated. |
| Action | `Microsoft.OperationalInsights/workspaces/search/action` | Deprecated. |

#### Log Analytics Contributor

Members of the Log Analytics Contributor role can:

- Read all monitoring data granted by the Log Analytics Reader role.
- Edit monitoring settings for Azure resources, including:
  - Adding the VM extension to VMs.
  - Configuring Azure diagnostics on all Azure resources.
- Create and configure Automation accounts. Permission must be granted at the resource group or subscription level.
- Add and remove management solutions. Permission must be granted at the resource group or subscription level.
- Read storage account keys.
- Configure the collection of logs from Azure Storage.
- Configure data export rules.
- [Run a search job.](search-jobs.md)
- [Restore archived logs.](restore.md)

> [!WARNING]
> You can use the permission to add a virtual machine extension to a virtual machine to gain full control over a virtual machine.

The Log Analytics Contributor role includes the following Azure actions:

| Permission | Description |
| ---------- | ----------- |
| `*/read`     | Ability to view all Azure resources and resource configuration.<br><br>Includes viewing:<br>- Virtual machine extension status.<br>- Configuration of Azure diagnostics on resources.<br>- All properties and settings of all resources.<br><br>For workspaces, allows full unrestricted permissions to read the workspace settings and query data. See more granular options in the preceding list. |
| `Microsoft.Automation/automationAccounts/*` | Ability to create and configure Azure Automation accounts, including adding and editing runbooks. |
| `Microsoft.ClassicCompute/virtualMachines/extensions/*` <br> `Microsoft.Compute/virtualMachines/extensions/*` | Add, update, and remove virtual machine extensions, including the Microsoft Monitoring Agent extension and the OMS Agent for Linux extension. |
| `Microsoft.ClassicStorage/storageAccounts/listKeys/action` <br> `Microsoft.Storage/storageAccounts/listKeys/action` | View the storage account key. Required to configure Log Analytics to read logs from Azure Storage accounts. |
| `Microsoft.Insights/alertRules/*` | Add, update, and remove alert rules. |
| `Microsoft.Insights/diagnosticSettings/*` | Add, update, and remove diagnostics settings on Azure resources. |
| `Microsoft.OperationalInsights/*` | Add, update, and remove configuration for Log Analytics workspaces. To edit workspace advanced settings, user needs `Microsoft.OperationalInsights/workspaces/write`. |
| `Microsoft.OperationsManagement/*` | Add and remove management solutions. |
| `Microsoft.Resources/deployments/*` | Create and delete deployments. Required for adding and removing solutions, workspaces, and automation accounts. |
| `Microsoft.Resources/subscriptions/resourcegroups/deployments/*` | Create and delete deployments. Required for adding and removing solutions, workspaces, and automation accounts. |

### Resource permissions

When users query logs from a workspace by using [resource-context access](#access-mode), they'll have the following permissions on the resource:

| Permission | Description |
| ---------- | ----------- |
| `Microsoft.Insights/logs/*/read` | Ability to view all log data for the resource  |
| `Microsoft.Insights/logs/<tableName>/read`<br>Example:<br>`Microsoft.Insights/logs/Heartbeat/read` | Ability to view specific table for this resource - legacy method  |
| `Microsoft.Insights/diagnosticSettings/write` | Ability to configure diagnostics setting to allow setting up logs for this resource |

The `/read` permission is usually granted from a role that includes _\*/read or_ _\*_ permissions, such as the built-in [Reader](../../role-based-access-control/built-in-roles.md#reader) and [Contributor](../../role-based-access-control/built-in-roles.md#contributor) roles. Custom roles that include specific actions or dedicated built-in roles might not include this permission.

### Custom role examples

In addition to using the built-in roles for a Log Analytics workspace, you can create custom roles to assign more granular permissions. Here are some common examples.

**Example 1: Grant a user permission to read log data from their resources.**

- Configure the workspace access control mode to *use workspace or resource permissions*.
- Grant users `*/read` or `Microsoft.Insights/logs/*/read` permissions to their resources. If they're already assigned the [Log Analytics Reader](../../role-based-access-control/built-in-roles.md#reader) role on the workspace, it's sufficient.


**Example 2: Grant a user permission to read log data from their resources and run a search job.**

- Configure the workspace access control mode to *use workspace or resource permissions*.
- Grant users `*/read` or `Microsoft.Insights/logs/*/read` permissions to their resources. If they're already assigned the [Log Analytics Reader](../../role-based-access-control/built-in-roles.md#reader) role on the workspace, it's sufficient.
- Grant users the following permissions on the workspace:
  - `Microsoft.OperationalInsights/workspaces/tables/write`: Required to be able to create the search results table (_SRCH).
  - `Microsoft.OperationalInsights/workspaces/searchJobs/write`: Required to allow executing the search job operation. 


**Example 3: Grant a user permission to read log data from their resources and configure their resources to send logs to the Log Analytics workspace.**

- Configure the workspace access control mode to *use workspace or resource permissions*.
- Grant users the following permissions on the workspace: `Microsoft.OperationalInsights/workspaces/read` and `Microsoft.OperationalInsights/workspaces/sharedKeys/action`. With these permissions, users can't perform any workspace-level queries. They can only enumerate the workspace and use it as a destination for diagnostic settings or agent configuration.
- Grant users the following permissions to their resources: `Microsoft.Insights/logs/*/read` and `Microsoft.Insights/diagnosticSettings/write`. If they're already assigned the [Log Analytics Contributor](../../role-based-access-control/built-in-roles.md#contributor) role, assigned the Reader role, or granted `*/read` permissions on this resource, it's sufficient.

**Example 4: Grant a user permission to read log data from their resources, but not to send logs to the Log Analytics workspace or read security events.**

- Configure the workspace access control mode to *use workspace or resource permissions*.
- Grant users the following permissions to their resources: `Microsoft.Insights/logs/*/read`.
- Add the following NonAction to block users from reading the SecurityEvent type: `Microsoft.Insights/logs/SecurityEvent/read`. The NonAction shall be in the same custom role as the action that provides the read permission (`Microsoft.Insights/logs/*/read`). If the user inherits the read action from another role that's assigned to this resource or to the subscription or resource group, they could read all log types. This scenario is also true if they inherit `*/read` that exists, for example, with the Reader or Contributor role.

**Example 5: Grant a user permission to read log data from their resources and all Azure AD sign-in and read Update Management solution log data in the Log Analytics workspace.**

- Configure the workspace access control mode to *use workspace or resource permissions*.
- Grant users the following permissions on the workspace:
  - `Microsoft.OperationalInsights/workspaces/read`: Required so the user can enumerate the workspace and open the workspace pane in the Azure portal
  - `Microsoft.OperationalInsights/workspaces/query/read`: Required for every user that can execute queries
  - `Microsoft.OperationalInsights/workspaces/query/SigninLogs/read`: To be able to read Azure AD sign-in logs
  - `Microsoft.OperationalInsights/workspaces/query/Update/read`: To be able to read Update Management solution logs
  - `Microsoft.OperationalInsights/workspaces/query/UpdateRunProgress/read`: To be able to read Update Management solution logs
  - `Microsoft.OperationalInsights/workspaces/query/UpdateSummary/read`: To be able to read Update Management logs
  - `Microsoft.OperationalInsights/workspaces/query/Heartbeat/read`: Required to be able to use Update Management solutions
  - `Microsoft.OperationalInsights/workspaces/query/ComputerGroup/read`: Required to be able to use Update Management solutions
- Grant users the following permissions to their resources: `*/read`, assigned to the Reader role, or `Microsoft.Insights/logs/*/read`

**Example 6: Restrict a user from restoring archived logs.**

- Configure the workspace access control mode to *use workspace or resource permissions*.
- Assign the user to the [Log Analytics Contributor](../../role-based-access-control/built-in-roles.md#contributor) role.
- Add the following NonAction to block users from restoring archived logs: `Microsoft.OperationalInsights/workspaces/restoreLogs/write`


## Set table-level read access

Table-level access allows you to let specific people read data only from a specific set of tables. It applies both for workspace-context and resource-context. There are two methods to define table-level permissions:
* By assigning permissions to the table sub-resource under the workspace resource - this is the recommended method that is described in this section. This method is currently in **preview**.
* By assigning special actions that contain table name to the workspace resource - this is the legacy method that is described in the next section. It has some limitations around custom log tables.

Table-level RBAC is applied during query execution. It does not apply to metadata retrieval calls. For that reason, tables will appear in the list of tables even if they are not available to the user.

> [!NOTE]
> The recommended table-level access method described here does not apply during preview to Microsoft Sentinel Detection Rules. These rules might have access to more tables than intended.

In order to apply table-level RBAC for a user, two assignments shall be made:

1. Assign the user the ability to read the workspace details and to run a query without granting the ability to run a query on tables. This is done by assigning a special custom role on the workspace that has only the following actions:
            - `Microsoft.OperationalInsights/workspaces/read`
            - `Microsoft.OperationalInsights/workspaces/query/read`
            - `Microsoft.OperationalInsights/workspaces/analytics/query/action`
            - `Microsoft.OperationalInsights/workspaces/search/action`
        
2. Assign the user a read permissions on the specific table sub-resource. Any role that has */read will be sufficient such as **Reader** role or **Log Analytics Reader** role. As table is a sub-resource of workspace, the workspace admins can also perform action on a specific table.

> [!WARNING]
> If the user has other assignments on the workspace, directly or via inheritence (e.g. user has Reader on the subscription that contains the workspace), the user will be able to access all tables in the workspace.



To create a [custom role](../../role-based-access-control/custom-roles.md) that lets specific users or groups read data from specific tables in a workspace:

1. Create a custom role that grants users permission to execute queries in the Log Analytics workspace, based on the built-in Azure Monitor Logs **Reader** role:

    1. Navigate to your workspace and select **Access control (IAM)** > **Roles**.

    1. Right-click the **Reader** role and select **Clone**.

       :::image type="content" source="media/manage-access/access-control-clone-role.png" alt-text="Screenshot that shows the Roles tab of the Access control screen with the clone button highlighted for the Reader role." lightbox="media/manage-access/access-control-clone-role.png":::        

       This opens the **Create a custom role** screen.

    1. On the **Basics** tab of the screen, enter a **Custom role name** value and, optionally, provide a description.

        :::image type="content" source="media/manage-access/manage-access-create-custom-role.png" alt-text="Screenshot that shows the Basics tab of the Create a custom role screen with the Custom role name and Description fields highlighted." lightbox="media/manage-access/manage-access-create-custom-role.png":::

    1. Select the **JSON** tab > **Edit**::

        1. In the `"actions"` section, add:

            - `Microsoft.OperationalInsights/workspaces/read`
            - `Microsoft.OperationalInsights/workspaces/query/read`
            - `Microsoft.OperationalInsights/workspaces/analytics/query/action`
            - `Microsoft.OperationalInsights/workspaces/search/action`

        1. In the `"not actions"` section, add `Microsoft.OperationalInsights/workspaces/sharedKeys/read`.

        :::image type="content" source="media/manage-access/manage-access-create-custom-role-json.png" alt-text="Screenshot that shows the JSON tab of the Create a custom role screen with the actions section of the JSON file highlighted." lightbox="media/manage-access/manage-access-create-custom-role-json.png":::    

    1. Select **Save** > **Review + Create** at the bottom of the screen, and then **Create** on the next page.   

1. Assign your custom role to the relevant users or groups:
    1. Select **Access control (AIM)** > **Add** > **Add role assignment**.

       :::image type="content" source="media/manage-access/manage-access-add-role-assignment-button.png" alt-text="Screenshot that shows the Access control screen with the Add role assignment button highlighted." lightbox="media/manage-access/manage-access-add-role-assignment-button.png":::

    1. Select the custom role you created and select **Next**.

       :::image type="content" source="media/manage-access/manage-access-add-role-assignment-screen.png" alt-text="Screenshot that shows the Add role assignment screen with a custom role and the Next button highlighted." lightbox="media/manage-access/manage-access-add-role-assignment-screen.png":::


       This opens the **Members** tab of the **Add custom role assignment** screen.   

    1. Click **+ Select members** to open the **Select members** screen.

        :::image type="content" source="media/manage-access/manage-access-add-role-assignment-select-members.png" alt-text="Screenshot that shows the Select members screen." lightbox="media/manage-access/manage-access-add-role-assignment-select-members.png":::

    1. Search for and select the relevant user or group and click **Select**.
    1. Select **Review and assign**.

1. Grant the users or groups read access to specific tables in a workspace by calling the `https://management.azure.com/batch?api-version=2020-06-01` POST API and sending the following details in the request body:

    ```json
    {
        "requests": [
            {
                "content": {
                    "Id": "<GUID_1>",
                    "Properties": {
                        "PrincipalId": "<user_object_ID>",
                        "PrincipalType": "User",
                        "RoleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7",
                        "Scope": "/subscriptions/<subscription_ID>/resourceGroups/<resource_group_name>/providers/Microsoft.OperationalInsights/workspaces/<workspace_name>/Tables/<table_name>",
                        "Condition": null,
                        "ConditionVersion": null
                    }
                },
                "httpMethod": "PUT",
                "name": "<GUID_2>",
                "requestHeaderDetails": {
                    "commandName": "Microsoft_Azure_AD."
                },
                "url": "/subscriptions/<subscription_ID>/resourceGroups/<resource_group_name>/providers/Microsoft.OperationalInsights/workspaces/<workspace_name>/Tables/<table_name>/providers/Microsoft.Authorization/roleAssignments/<GUID_1>?api-version=2020-04-01-preview"
            }
        ]
    }
    ```

    Where:
    - You can generate a GUID for `<GUID 1>` and `<GUID 2>` using any GUID generator.
    - `<user_object_ID>` is the object ID of the user to which you want to grant table read access.
    - `<subscription_ID>` is the ID of the subscription related to the workspace.
    - `<resource_group_name>` is the resource group of the workspace.
    - `<workspace_name>` is the name of the workspace.
    - `<table_name>` is the name of the table to which you want to assign the user or group permission to read data from.

### Legacy method of setting table-level read access

The legacy method of table-level also uses [Azure custom roles](../../role-based-access-control/custom-roles.md) to let you grant specific users or groups access to specific tables in the workspace. Azure custom roles apply to workspaces with either workspace-context or resource-context [access control modes](#access-control-mode) regardless of the user's [access mode](#access-mode).

To define access to a particular table, create a [custom role](../../role-based-access-control/custom-roles.md):

* Set the user permissions in the **Actions** section of the role definition. 
* Use `Microsoft.OperationalInsights/workspaces/query/*` to grant access to all tables.
* To exclude access to specific tables when you use a wildcard in **Actions**, list the tables excluded tables in the **NotActions** section of the role definition.

#### Examples

Here are examples of custom role actions to grant and deny access to specific tables.

Grant access to the _Heartbeat_ and _AzureActivity_ tables:

```
"Actions":  [
    "Microsoft.OperationalInsights/workspaces/read",
    "Microsoft.OperationalInsights/workspaces/query/read",
    "Microsoft.OperationalInsights/workspaces/query/Heartbeat/read",
    "Microsoft.OperationalInsights/workspaces/query/AzureActivity/read"
  ],
```

Grant access to only the _SecurityBaseline_ table:

```
"Actions":  [
    "Microsoft.OperationalInsights/workspaces/read",
    "Microsoft.OperationalInsights/workspaces/query/read",
    "Microsoft.OperationalInsights/workspaces/query/SecurityBaseline/read"
],
```


Grant access to all tables except the _SecurityAlert_ table:

```
"Actions":  [
    "Microsoft.OperationalInsights/workspaces/read",
    "Microsoft.OperationalInsights/workspaces/query/read",
    "Microsoft.OperationalInsights/workspaces/query/*/read"
],
"notActions":  [
    "Microsoft.OperationalInsights/workspaces/query/SecurityAlert/read"
],
```

#### Custom tables

Custom tables store data you collect from data sources such as [text logs](../agents/data-sources-custom-logs.md) and the [HTTP Data Collector API](data-collector-api.md). To identify the table type, [view table information in Log Analytics](./log-analytics-tutorial.md#view-table-information).

> [!NOTE]
> Tables created by the [Logs ingestion API](../essentials/../logs/logs-ingestion-api-overview.md) don't yet support table-level RBAC.

Using the legacy method of table-level access, you can't grant access to individual custom log tables at the table level, but you can grant access to all custom log tables. To create a role with access to all custom log tables, create a custom role by using the following actions:

```
"Actions":  [
    "Microsoft.OperationalInsights/workspaces/read",
    "Microsoft.OperationalInsights/workspaces/query/read",
    "Microsoft.OperationalInsights/workspaces/query/Tables.Custom/read"
],
```

### Considerations regarding table-level access

- If a user is granted global read permission with the standard Reader or Contributor roles that include the _\*/read_ action, it will override the per-table access control and give them access to all log data.
- If a user is granted per-table access but no other permissions, they can access log data from the API but not from the Azure portal. To provide access from the Azure portal, use Log Analytics Reader as its base role.
- Administrators and owners of the subscription will have access to all data types regardless of any other permission settings.
- Workspace owners are treated like any other user for per-table access control.
- Assign roles to security groups instead of individual users to reduce the number of assignments. This practice will also help you use existing group management tools to configure and verify access.

## Next steps

* See [Log Analytics agent overview](../agents/log-analytics-agent.md) to gather data from computers in your datacenter or other cloud environment.
* See [Collect data about Azure virtual machines](../vm/monitor-virtual-machine.md) to configure data collection from Azure VMs.
