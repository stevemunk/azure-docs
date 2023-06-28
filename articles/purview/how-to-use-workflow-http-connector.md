---
title: Use external APIs in Workflows
description: Work with external APIs using HTTP connector and Parse JSON in Microsoft Purview workflows.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-workflows
ms.topic: how-to
ms.date: 05/16/2023
ms.custom: template-how-to
---

# Use external APIs in Workflows

[!INCLUDE [feature-in-preview](includes/feature-in-preview.md)]

You can use [workflows](concept-workflow.md) to automate some business processes through Microsoft Purview, and the [HTTP connector](#http-connector) and [parse JSON action](#parse-json-action) allow you to integrate your workflows with external applications.

## HTTP connector

HTTP connectors use Representational State Transfer (REST) architecture, which allows Microsoft Purview workflows to interact directly with third party applications by using web requests. 

HTTP connector is available in all workflow templates.

>[!NOTE]
> To create or edit a workflow, you need the [workflow admin role](catalog-permissions.md) in Microsoft Purview. You can also contact the workflow admin in your collection, or reach out to your collection administrator, for permissions.

1. To add an HTTP connector, click on the **+** icon in the template where you want to add and select HTTP connector.

    :::image type="content" source="./media/how-to-use-workflow-http-connector/add-http-connector.png" alt-text="Screenshot of how to add HTTP connector.":::

1. Once you select HTTP connector, you see the following parameters:
    1. Host - Request URL you want to call when this connector is executed.
    1. Method - Select one of the following methods. GET, PUT, PATCH, POST and DELETE. These methods correspond to create, read, update and delete operations.
    1. Path - Optionally you can enter request URL Path. You can use dynamic content for this parameter.
    1. Headers - Optionally, you can enter HTTP headers. HTTP headers let the client and the server pass additional information with an HTTP request or response
    1. Queries - Optionally, you can pass queries. 
    1. Body - Optionally, you can pass HTTP body while invoking the URL
    1. Authentication - HTTP connector is integrated with Purview credentials. Depending on the URL, you may invoke the endpoint with None (no authentication) or you can use credentials to create a basic authentication. To learn more about credentials see the [Microsoft Purview credentials article](manage-credentials.md).

    :::image type="content" source="./media/how-to-use-workflow-http-connector/add-http-properties.png" alt-text="Screenshot of how to add HTTP connector properties.":::

1. By default, secure settings are turned on for HTTP connectors. To turn OFF secure inputs and outputs select the ellipsis icon (**...**) to go to settings.

    :::image type="content" source="./media/how-to-use-workflow-http-connector/add-http-settings.png" alt-text="Screenshot of how to add HTTP connector settings."::: 

1. You are now presented with the settings for HTTP connector and you can turn secure inputs and outputs OFF.

    :::image type="content" source="./media/how-to-use-workflow-http-connector/add-http-secure.png" alt-text="Screenshot of how to add HTTP connector secure input and outputs.":::
    
## Parse JSON action

The parse JSON action in workflows allows you to take an incoming JSON from HTTP (or any other action/connector), and parse the JSON to extract values for use in your workflow.

The parse JSON action is available in all workflows.

:::image type="content" source="./media/how-to-use-workflow-http-connector/parse-json-action.png" alt-text="Screenshot of the workflows parse JSON action, showing its parameters filled out with sample data.":::

The parse JSON action has two parameters:

- Content - a variable that should contain the JSON you want to parse.
- Schema - the schema of the incoming JSON, which allows the workflow to parse the incoming information. You can supply your own, or use the **Generate from sample** button. If you generate from a sample, you'll enter a sample JSON payload and a schema will be automatically generated for you.

Actions and connectors in the workflow after the parse JSON action will be able to use the values extracted from the JSON by selecting **Add dynamic content** for any parameters.

:::image type="content" source="./media/how-to-use-workflow-http-connector/add-dynamic-content.png" alt-text="Screenshot of another connector with values from the Parse JSON action added.":::


## Next steps

For more information about workflows, see these articles:

- [Workflows in Microsoft Purview](concept-workflow.md)
- [Approval workflow for business terms](how-to-workflow-business-terms-approval.md)
- [Manage workflow requests and approvals](how-to-workflow-manage-requests-approvals.md)

