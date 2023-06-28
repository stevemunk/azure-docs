---
title: SMS  metrics definitions for Azure Communication Service
titleSuffix: An Azure Communication Services concept document
description: This document covers definitions of SMS metrics available in the Azure portal.
author: mkhribech
manager: timmitchell
services: azure-communication-services
ms.author: mkhribech
ms.date: 06/26/2023
ms.topic: conceptual
ms.service: azure-communication-services
ms.subservice: data
---
# SMS  metrics overview

Azure Communication Services currently provides metrics for all ACS primitives. [Azure Metrics Explorer](../../../azure-monitor\essentials\metrics-getting-started.md) can be used to plot your own charts, investigate abnormalities in your metric values, and understand your API traffic by using the metrics data that Chat and SMS requests emit.

## Where to find metrics

Primitives in Azure Communication Services emit metrics for API requests. These metrics can be found in the Metrics tab under your Communication Services resource. You can also create permanent dashboards using the workbooks tab under your Communication Services resource.

## Metric definitions

All API request metrics contain three dimensions that you can use to filter your metrics data. These dimensions can be aggregated together using the `Count` aggregation type and support all standard Azure Aggregation time series including `Sum`, `Average`, `Min`, and `Max`.

More information on supported aggregation types and time series aggregations can be found [Advanced features of Azure Metrics Explorer](../../../azure-monitor/essentials/metrics-charts.md#aggregation).

- **Operation** - All operations or routes that can be called on the Azure Communication Services Chat gateway.
- **Status Code** - The status code response sent after the request.
- **StatusSubClass** - The status code series sent after the response. 
- 
### SMS API requests

The following operations are available on SMS API request metrics:

| Operation / Route    | Description                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| SMSMessageSent       | Sends an SMS message. |
| SMSDeliveryReportsReceived     | Gets SMS Delivery Reports |
| SMSMessagesReceived      | Gets SMS messages. |


:::image type="content" source="../media/sms-metric.png" alt-text="SMS API Request Metric.":::
