---
title: Workflow connectors and actions
description: This article describes how to use connectors and actions in Microsoft Purview workflows
author: zeinam
ms.author: zeinam
ms.service: purview
ms.subservice: purview-workflows
ms.topic: how-to
ms.date: 05/15/2023
ms.custom: template-how-to
---

# Workflow connectors and actions

[!INCLUDE [feature-in-preview](includes/feature-in-preview.md)]

You can use [workflows](concept-workflow.md) to automate some business processes through Microsoft Purview. A Connector in a workflow provides a way to connect to different systems and leverage a set of prebuilt actions and triggers.

## Current workflow connectors and actions

Currently the following connectors are available for a workflow in Microsoft Purview:

|Connector Type  |Functionality  |Parameters  |Customizable  |Workflow templates |
|---------|---------|---------|---------|---------|
|Apply to each |Apply an action or set of actions to all returned values in an output. | -Output to process <br> -Actions|- Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow|All workflows templates|
|Check data source registration for data use governance     |Validate if data source has been registered with Data Use Management enabled.         |None         | - Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow         |Data access request         |
|Condition     |Evaluate a value to true or false. Based on the evaluation the workflow will be re-directed to different branches         |- Add row <br> - Title <br> - Add group         |- Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow         |All workflows templates         |
|Create Glossary Term     |Create a new glossary term         |None         |- Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow         |Create glossary term template         |
|Create task and wait for task completion     |Creates, assigns, and tracks a task to a user or Azure Active Directory group as part of a workflow. <br> - Reminder settings - You can set reminders to periodically remind the task owner till they complete the task. <br> - Expiry settings - You can set an expiration or deadline for the task activity. Also, you can set who needs to be notified (user/AAD group) after the expiry.        |- Assigned to <br> - Task title <br> - Task body         |- Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow         |All workflows templates         |
|Delete glossary term     |Delete an existing glossary term         |None         |- Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow         |Delete glossary term         |
|Grant access     |Create an access policy to grant access to the requested user.         |None         |- Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow         |Data access request         |
|Http     |Integrate with external applications through http or https call. <br> For more information, see [Workflows HTTP connector](how-to-use-workflow-http-connector.md)         |- Host <br> - Method <br> - Path <br> - Headers <br> - Queries <br> - Body <br> - Authentication         |- Renamable: Yes <br> - Deletable: Yes <br> - Settings: Secured Input and Secure outputs (Enabled by default) <br> - Multiple per workflow         |All workflows templates         |
|Import glossary terms     |Import one or more glossary terms         |None        |- Renamable: Yes <br> - Deletable: No <br> - Multiple per workflow         |Import terms         |
|Parse JSON    |Parse an incoming JSON to extract parameters       |- Content <br> - Schema <br>        |- Renamable: Yes <br> - Deletable: No <br> - Multiple per workflow         |All workflows templates         |
|Send email notification     |Send email notification to one or more recipients         |- Subject <br> - Message body <br> - Recipient         |- Renamable: Yes <br> - Deletable: Yes <br> - Settings: Secured Input and Secure outputs (Enabled by default) <br> - Multiple per workflow         |All workflows templates         |
|Start and wait for an approval     |Generates approval requests and assign the requests to individual users or Microsoft Azure Active Directory groups. Microsoft Purview workflow approval connector currently supports two types of approval types:  <br> - First to Respond – This implies that the first approver's outcome (Approve/Reject) is considered final. <br> - Everyone must approve – This implies everyone identified as an approver must approve the request for the request to be considered approved. If one approver rejects the request, regardless of other approvers, the request is rejected. <br> - Reminder settings - You can set reminders to periodically remind the approver till they approve or reject. <br> - Expiry settings - You can set an expiration or deadline for the approval activity. Also, you can set who needs to be notified (user/AAD group) after the expiry. |- Approval Type <br> - Title <br> - Assigned To         |- Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow         |All workflows templates         | 
|Update glossary term     |Update an existing glossary term         |None         |- Renamable: Yes <br> - Deletable: Yes <br> - Multiple per workflow         |Update glossary term         |
|When term creation request is submitted     |Triggers a workflow with all term details when a new term request is submitted         |None         |- Renamable: Yes <br> - Deletable: No <br> - Only one per workflow         |Create glossary term template         |
|When term deletion request is submitted     |Triggers a workflow with all term details when a request to delete an existing term is submitted         |None         |- Renamable: Yes <br> - Deletable: No <br> - Only one per workflow         |Delete glossary term         |
|When term Import request is submitted     |Triggers a workflow with all term details in a csv file, when a request to import terms is submitted         |None         |- Renamable: Yes <br> - Deletable: No <br> - Only one per workflow         |Import terms         |
|When term update request is submitted     |Triggers a workflow with all term details when a request to update an existing term is submitted         |None         |- Renamable: Yes <br> - Deletable: No <br> - Only one per workflow         |Update glossary term       |

## Next steps

For more information about workflows, see these articles:

- [Workflows in Microsoft Purview](concept-workflow.md)
- [Approval workflow for business terms](how-to-workflow-business-terms-approval.md)
- [Manage workflow requests and approvals](how-to-workflow-manage-requests-approvals.md)
