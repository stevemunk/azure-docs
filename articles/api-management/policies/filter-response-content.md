---
title: Azure API management policy sample - Filter response content | Microsoft Docs
description: Azure API management policy sample - Demonstrates how to filter data elements from the response payload based on the product associated with the request.
services: api-management
documentationcenter: ''
author: dlepow
manager: cfowler
editor: ''

ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 10/13/2017
ms.author: danlep
---

# Filter response content

This article shows an Azure API management policy sample that demonstrates how to filter data elements from the response payload based on the product associated with the request. To set or edit a policy code, follow the steps described in [Set or edit a policy](../set-edit-policies.md). To see other examples, see [policy samples](/azure/api-management/policies).

## Policy

Paste the code into the **outbound** block.

[!code-xml[Main](../../../api-management-policy-samples/examples/Filter response content based on product name.policy.xml)]

## Next steps

Learn more about API Management policies:

+ [Transformation policies](../api-management-transformation-policies.md)
+ [Policy samples](/azure/api-management/policies)