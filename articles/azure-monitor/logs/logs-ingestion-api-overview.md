---
title: Logs Ingestion API in Azure Monitor
description: Send data to a Log Analytics workspace by using a REST API.
ms.topic: conceptual
ms.date: 06/27/2022

---

# Logs Ingestion API in Azure Monitor

The Logs Ingestion API in Azure Monitor lets you send data to a Log Analytics workspace from any REST API client. By using this API, you can send data from almost any source to [supported Azure tables](#supported-tables) or to [custom tables that you create](../logs/create-custom-table.md#create-a-custom-table). You can even [extend the schema of Azure tables with custom columns](../logs/create-custom-table.md#add-or-delete-a-custom-column).

> [!NOTE]
> The Logs Ingestion API was previously referred to as the custom logs API.

## Basic operation

Your application sends data to a [data collection endpoint (DCE)](../essentials/data-collection-endpoint-overview.md), which is a unique connection point for your subscription. The payload of your API call includes the source data formatted in JSON. The call:

- Specifies a [data collection rule (DCR)](../essentials/data-collection-rule-overview.md) that understands the format of the source data.
- Potentially filters and transforms it for the target table.
- Directs it to a specific table in a specific workspace.

You can modify the target table and workspace by modifying the DCR without any change to the REST API call or source data.

:::image type="content" source="media/data-ingestion-api-overview/data-ingestion-api-overview.png" lightbox="media/data-ingestion-api-overview/data-ingestion-api-overview.png" alt-text="Diagram that shows an overview of logs ingestion API.":::

> [!NOTE]
> To migrate solutions from the [Data Collector API](data-collector-api.md), see [Migrate from Data Collector API and custom fields-enabled tables to DCR-based custom logs](custom-logs-migrate.md).

## Supported tables

### Custom tables

The Logs Ingestion API can send data to any custom table that you create and to certain Azure tables in your Log Analytics workspace. The target table must exist before you can send data to it. Custom tables must have the `_CL` suffix.

### Azure tables

The Logs Ingestion API can send data to the following Azure tables. Other tables may be added to this list as support for them is implemented. 

- [CommonSecurityLog](/azure/azure-monitor/reference/tables/commonsecuritylog)
- [SecurityEvents](/azure/azure-monitor/reference/tables/securityevent)
- [Syslog](/azure/azure-monitor/reference/tables/syslog)
- [WindowsEvents](/azure/azure-monitor/reference/tables/windowsevent)

> [!NOTE]
> Column names must start with a letter and can consist of up to 45 alphanumeric characters and the characters `_` and `-`. The following are reserved column names: `Type`, `TenantId`, `resource`, `resourceid`, `resourcename`, `resourcetype`, `subscriptionid`, `tenanted`. Custom columns you add to an Azure table must have the suffix `_CF`.

## Authentication

Authentication for the Logs Ingestion API is performed at the DCE, which uses standard Azure Resource Manager authentication. A common strategy is to use an application ID and application key as described in [Tutorial: Add ingestion-time transformation to Azure Monitor Logs](tutorial-logs-ingestion-portal.md).

## Source data

The source data sent by your application is formatted in JSON and must match the structure expected by the DCR. It doesn't necessarily need to match the structure of the target table because the DCR can include a [transformation](../essentials//data-collection-transformations.md) to convert the data to match the table's structure.

## Data collection rule

[Data collection rules](../essentials/data-collection-rule-overview.md) define data collected by Azure Monitor and specify how and where that data should be sent or stored. The REST API call must specify a DCR to use. A single DCE can support multiple DCRs, so you can specify a different DCR for different sources and target tables.

The DCR must understand the structure of the input data and the structure of the target table. If the two don't match, it can use a [transformation](../essentials/data-collection-transformations.md) to convert the source data to match the target table. You can also use the transformation to filter source data and perform any other calculations or conversions.

## Send data

To send data to Azure Monitor with the Logs Ingestion API, make a POST call to the DCE over HTTP. Details of the call are described in the following sections.

### Endpoint URI

The endpoint URI uses the following format, where the `Data Collection Endpoint` and `DCR Immutable ID` identify the DCE and DCR. `Stream Name` refers to the [stream](../essentials/data-collection-rule-structure.md#custom-logs) in the DCR that should handle the custom data.

```
{Data Collection Endpoint URI}/dataCollectionRules/{DCR Immutable ID}/streams/{Stream Name}?api-version=2021-11-01-preview
```

> [!NOTE]
> You can retrieve the immutable ID from the JSON view of the DCR. For more information, see [Collect information from the DCR](tutorial-logs-ingestion-portal.md#collect-information-from-the-dcr).

### Headers

| Header | Required? | Value | Description |
|:---|:---|:---|:---|
| Authorization     | Yes | Bearer (bearer token obtained through the client credentials flow)  | |
| Content-Type      | Yes | `application/json` | |
| Content-Encoding  | No  | `gzip` | Use the gzip compression scheme for performance optimization. |
| x-ms-client-request-id | No | String-formatted GUID |  Request ID that can be used by Microsoft for any troubleshooting purposes.  |

### Body

The body of the call includes the custom data to be sent to Azure Monitor. The shape of the data must be a JSON object or array with a structure that matches the format expected by the stream in the DCR. Additionally, it is important to ensure that the request body is properly encoded in UTF-8 to prevent any issues with data transmission.

## Sample call

For sample data and an API call using the Logs Ingestion API, see either [Send custom logs to Azure Monitor Logs using the Azure portal](tutorial-logs-ingestion-portal.md) or [Send custom logs to Azure Monitor Logs using Resource Manager templates](tutorial-logs-ingestion-api.md).

## Limits and restrictions

For limits related to the Logs Ingestion API, see [Azure Monitor service limits](../service-limits.md#logs-ingestion-api).

## Next steps

- [Walk through a tutorial sending custom logs using the Azure portal](tutorial-logs-ingestion-portal.md)
- [Walk through a tutorial sending custom logs using Resource Manager templates and REST API](tutorial-logs-ingestion-api.md)
- Get guidance on using the client libraries for the Logs ingestion API for [.NET](/dotnet/api/overview/azure/Monitor.Ingestion-readme), [Java](/java/api/overview/azure/monitor-ingestion-readme), [JavaScript](/javascript/api/overview/azure/monitor-ingestion-readme), or [Python](/python/api/overview/azure/monitor-ingestion-readme).
