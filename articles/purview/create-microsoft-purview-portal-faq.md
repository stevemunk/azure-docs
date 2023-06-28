---
title: Create an exception to deploy Microsoft Purview
description: This article describes how to create an exception to deploy Microsoft Purview while leaving existing Azure policies in place to maintain security.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.topic: how-to
ms.date: 06/02/2023
---

# Create an exception to deploy Microsoft Purview

Many subscriptions have [Azure Policies](../governance/policy/overview.md) in place that restrict the creation or update of some resources. This is to maintain subscription security and cleanliness. However, Microsoft Purview accounts will deploy an Azure Storage account when it's created. It'll be managed by Azure, so you don't need to maintain it, but it is necessary for Microsoft Purview to run correctly. Existing policies may block this deployment, and you may receive an error when attempting to create a Microsoft Purview account.

Microsoft Purview also regularly updates its Azure Storage account after creation, so any policies blocking updates to this storage account will cause errors during scanning.

To maintain your policies in your subscription, but still allow the creation and updates to these managed resources, you can create an exception.

## Create an Azure policy exception for Microsoft Purview

1. Navigate to the [Azure portal](https://portal.azure.com) and search for **Policy**

    :::image type="content" source="media/create-purview-portal-faq/search-for-policy.png" alt-text="Screenshot showing the Azure portal search bar, searching for Policy keyword.":::

1. Follow [Create a custom policy definition](../governance/policy/tutorials/create-custom-policy-definition.md) or modify existing policy to add two exceptions with `not` operator and `resourceBypass` tag:

    ```json
    {
    "mode": "All",
      "policyRule": {
        "if": {
          "anyOf": [
          {
            "allOf": [
            {
              "field": "type",
              "equals": "Microsoft.Storage/storageAccounts"
            },
            {
              "not": {
                "field": "tags['<resourceBypass>']",
                "exists": true
              }
            }]
          },
          {
            "allOf": [
            {
              "field": "type",
              "equals": "Microsoft.EventHub/namespaces"
            },
            {
              "not": {
                "field": "tags['<resourceBypass>']",
                "exists": true
              }
            }]
          }]
        },
        "then": {
          "effect": "deny"
        }
      },
      "parameters": {}
    }
    ```
  
    > [!Note]
    > The tag could be named anything beside `resourceBypass`. You'll create the tag either when creating Microsoft Purview, or you can add it to a previously created account in the **Tags** page on the resource in the Azure portal. The name can be whatever you choose, but when you define the tag the value needs to be 'allowed'.

    :::image type="content" source="media/create-catalog-portal/policy-definition.png" alt-text="Screenshot showing how to create policy definition.":::

1. [Create a policy assignment](../governance/policy/assign-policy-portal.md) using the custom policy created.

    :::image type="content" source="media/create-catalog-portal/policy-assignment.png" alt-text="Screenshot showing how to create policy assignment" lightbox="./media/create-catalog-portal/policy-assignment.png":::

> [!Note] 
> If you have **Azure Policy** and need to add exception as in **Prerequisites**, you need to add the correct tag. For example, you can add `resourceBypass` tag. The name can be whatever you like, as long as the tag name matches the name in the policy, but the value must be "allowed".
> :::image type="content" source="media/create-catalog-portal/add-purview-tag.png" alt-text="Add tag to Microsoft Purview account.":::

## Next steps

To set up Microsoft Purview by using Private Link, see [Use private endpoints for your Microsoft Purview account](./catalog-private-link.md).
