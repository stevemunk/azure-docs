---
title: What's new in Microsoft Sentinel
description: This article describes new features in Microsoft Sentinel from the past few months.
author: yelevin
ms.author: yelevin
ms.topic: conceptual
ms.date: 06/22/2023
---

# What's new in Microsoft Sentinel

This article lists recent features added for Microsoft Sentinel, and new features in related services that provide an enhanced user experience in Microsoft Sentinel.

The listed features were released in the last three months. For information about earlier features delivered, see our [Tech Community blogs](https://techcommunity.microsoft.com/t5/azure-sentinel/bg-p/AzureSentinelBlog/label-name/What's%20New).

See these [important announcements](#announcements) about recent changes to features and services.

> [!TIP]
> Get notified when this page is updated by copying and pasting the following URL into your feed reader:
>
> `https://aka.ms/sentinel/rss`

[!INCLUDE [reference-to-feature-availability](includes/reference-to-feature-availability.md)]

## June 2023

- [Content Hub is now generally available](#content-hub-is-now-generally-available)
- [Windows Forwarded Events connector is now generally available](#windows-forwarded-events-connector-is-now-generally-available)
- [Connect multiple SAP System Identifiers via the UI](#connect-multiple-sap-system-identifiers-via-the-ui-preview)
- [Classic alert automation due for deprecation](#classic-alert-automation-due-for-deprecation) (see Announcements)
- [Microsoft Sentinel solution for SAP® applications: new systemconfig.json file](#microsoft-sentinel-solution-for-sap-applications-new-systemconfigjson-file)

### Content Hub is now generally available

Along with the general availability (GA) of content hub, the [content hub centralization changes announced in February](#out-of-the-box-content-centralization-changes) have also taken effect. For more information on these changes and their impact, including more details about the tool provided to reinstate **IN USE** gallery templates, see [Out-of-the-box (OOTB) content centralization changes](sentinel-content-centralize.md). 

As part of the deployment for GA, the default view of the content hub is now the **List view**. The install process is streamlined as well. When selecting **Install** or **Install/Update**, the experience behaves like bulk installation. For more information on the content hub GA, see this [blog post](https://aka.ms/sentinelcontenthubga-blogpost).

### Windows Forwarded Events connector is now generally available

The Windows Forwarded Events connector is now generally available. The connector is available in both the Azure Commercial and Azure Government clouds. Review the [connector information](data-connectors/windows-forwarded-events.md).

### Connect multiple SAP System Identifiers via the UI (Preview)

You can now connect multiple SAP System Identifiers (SID) via the connector page in the UI, and gain insights to the connectivity health status of each. To gain access to this feature, **first [complete the sign-up form](https://aka.ms/SentinelSAPMultiSIDUX)**.

Learn more about how to [deploy the container and SAP systems via the UI](sap/deploy-data-connector-agent-container.md) and how to [monitor the health of your SAP systems](monitor-sap-system-health.md).

### Microsoft Sentinel solution for SAP® applications: new systemconfig.json file

Microsoft Sentinel solution for SAP® applications uses the new *[systemconfig.json](sap/reference-systemconfig-json.md)* file from agent versions deployed on June 22 and later. For previous agent versions, you must still use the *[systemconfig.ini file](sap/reference-systemconfig.md)*.

## May 2023

- Connect your threat intelligence platform or custom solution to Microsoft Sentinel with the new [upload indicators API](#connect-threat-intelligence-with-the-upload-indicators-api)
- [Use Hunts to conduct end-to-end proactive threat hunting in Microsoft Sentinel](#use-hunts-to-conduct-end-to-end-proactive-threat-hunting)
- [Audit and track incident task activity](#audit-and-track-incident-task-activity)
- Updated the announcement for [Out-of-the-box content centralization changes](#out-of-the-box-content-centralization-changes) to include information on the **Next Steps** tab in data connectors that's deprecated.

### Connect threat intelligence with the upload indicators API

There's a new and improved API for connecting your threat intelligence platform or custom solution to add Indicators of Compromise (IOCs) into Microsoft Sentinel. The data connector and the API it relies on offer the following improvements:
- The threat indicator fields use the standardized format of the STIX specification.
- The Azure Active Directory (Azure AD) application registration only requires the Microsoft Sentinel Contributor role.
- The API request endpoint is scoped to the workspace level and the Azure AD application permissions required allow granular assignment at the workspace level.

Learn more about the [TI upload indicators API data connector](connect-threat-intelligence-upload-api.md).
Learn more about the underlying [TI upload indicators API](upload-indicators-api.md).

The [Threat Intelligence Platform data connector](connect-threat-intelligence-tip.md) is now on a path for deprecation. More details will be published on the precise timeline. New Microsoft Sentinel solutions should use the upload indicators API instead of the Microsoft Graph threat intelligence indicator API. 

### Use Hunts to conduct end-to-end proactive threat hunting

Take your hunts to the next level. Stay organized and keep track of new, active and closed hunts. Don't wait to react, proactively track down detection gaps on specific MITRE ATT&CK techniques or your own hypotheses. Collect evidence, investigate entities, annotate your findings and share them with your team all in one screen. 

Learn more about [Hunts (Preview)](hunts.md).

### Audit and track incident task activity

Thanks to newly available information in the *SecurityIncident* table, you can now inspect the history and status of open tasks in your incidents, even on incidents that have been closed. Use the information to ensure your SOC's efficient and proper functioning.

Learn more about [auditing and tracking incident tasks](audit-track-tasks.md).

## April 2023

- [RSA announcements](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/rsac-2023-microsoft-sentinel-empowering-the-soc-with-next-gen/ba-p/3803613)
- [Manage multiple workspaces with workspace manager](#manage-multiple-workspaces-with-workspace-manager-preview)

### Manage multiple workspaces with workspace manager (Preview)

Centrally manage Microsoft Sentinel at scale with Workspace Manager. Whether you're working across workspaces or Azure AD tenants, workspace manager reduces the complexity.

Learn more about [Microsoft Sentinel workspace manager](workspace-manager.md). 

## March 2023

- [Microsoft Sentinel for SAP® BTP solution (Preview)](#microsoft-sentinel-solution-for-sap-btp-preview)
- [Work with the Microsoft Sentinel Solution for SAP® applications across multiple workspaces (Preview)](#work-with-the-microsoft-sentinel-solution-for-sap-applications-across-multiple-workspaces-preview)
- [Monitoring the configuration of static SAP security parameters](#monitoring-the-configuration-of-static-sap-security-parameters-preview)
- [Stream log data from the Google Cloud Platform into Microsoft Sentinel (Preview)](#stream-log-data-from-the-google-cloud-platform-into-microsoft-sentinel-preview)
- [Microsoft Defender Threat Intelligence data connector (Preview)](#microsoft-defender-threat-intelligence-data-connector-preview)
- [Microsoft Defender Threat Intelligence solution (Preview)](#microsoft-defender-threat-intelligence-solution-preview)
- [Automatically update the SAP data connector agent](#automatically-update-the-sap-data-connector-agent)

### Microsoft Sentinel Solution for SAP® BTP (Preview)

The Microsoft Sentinel Solution for SAP BTP monitors and protects your SAP Business Technology Platform (BTP) system, by collecting audits and activity logs from the BTP infrastructure and BTP based apps, and detecting threats, suspicious activities, illegitimate activities, and more. 

The solution includes the **SAP BTP** connector, [built-in analytics rules](sap/sap-btp-security-content.md#built-in-analytics-rules) for identity management and low-code application development scenarios, and the [BTP activity workbook](sap/sap-btp-security-content.md#sap-btp-workbook), which provides a dashboard overview of subaccounts and a grid of identity management events.

[Learn more about the solution](sap/sap-btp-solution-overview.md).

### Work with the Microsoft Sentinel solution for SAP® applications across multiple workspaces (Preview)

You can now [work with the Microsoft Sentinel solution for SAP® applications across multiple workspaces](sap/cross-workspace.md) in different scenarios. This feature allows improved flexibility for managed security service providers (MSSPs) or a global or federated SOC, data residency requirements, organizational hierarchy/IT design, and insufficient role-based access control (RBAC) in a single workspace. One common use case is the need for collaboration between the security operations center (SOC) and SAP teams in your organization. Read about [the scenarios that address this use case](sap/cross-workspace.md).

### Monitoring the configuration of static SAP security parameters (Preview)

To secure the SAP system, SAP has identified security-related parameters that need to be monitored for changes. With the ["SAP - (Preview) Sensitive Static Parameter has Changed" analytics rule](sap/sap-solution-security-content.md#monitoring-the-configuration-of-static-sap-security-parameters-preview), the Microsoft Sentinel solution for SAP® applications tracks [over 52 security-related parameters](sap/sap-suspicious-configuration-security-parameters.md) in the SAP system, and triggers an alert once these parameters are changed not according to the policy.

For the Microsoft Sentinel solution for SAP® applications to successfully monitor the SAP security parameters, the solution needs to successfully monitor the SAP PAHI table at regular intervals. [Verify that the solution can successfully monitor the PAHI table](sap/preparing-sap.md#verify-that-the-pahi-table-history-of-system-database-and-sap-parameters-is-updated-at-regular-intervals).

### Stream log data from the Google Cloud Platform into Microsoft Sentinel (Preview)

You can now [stream audit log data from the Google Cloud Platform (GCP) into Microsoft Sentinel](connect-google-cloud-platform.md) using the **GCP Pub/Sub Audit Logs** connector, based on our [Codeless Connector Platform](create-codeless-connector.md?tabs=deploy-via-arm-template%2Cconnect-via-the-azure-portal) (CCP). The new connector ingests logs from your GCP environment using the GCP [Pub/Sub capability](https://cloud.google.com/pubsub/docs/overview). 

### Microsoft Defender Threat Intelligence data connector (Preview)

Now bring the high fidelity indicators of compromise (IOC) generated by Microsoft Defender Threat Intelligence (MDTI) into your Microsoft Sentinel workspace with the [Microsoft Defender Threat Intelligence data connector](connect-mdti-data-connector.md). 

### Microsoft Defender Threat Intelligence solution (Preview)

A collection of playbooks is available in the new Microsoft Defender Threat Intelligence *solution*. Utilize the playbooks to enrich entities (Domains, Hosts and IPs) associated with Microsoft Sentinel **Incidents**. The enrichment uses comprehensive Threat Intelligence (TI) data to add risk scoring, useful tags, analyst insights, and links to published TI articles. What makes the TI data so comprehensive and compelling? Along with traditional TI datasets of DNS, reverse DNS, WHOIS, SSL certificates and subdomains, the MDTI enrichment works with advanced TI datasets like trackers, web components, host pairs, and cookies.

Enabling this solution helps your security team achieve the following goals:
- accelerate investigations
- increase visibility 
- respond more effectively to threats
- maximize impact of existing security incident response

Check out the [Tech Community blog](https://techcommunity.microsoft.com/t5/microsoft-defender-threat/what-s-new-at-microsoft-secure/ba-p/3773576) for more information on announcements from Microsoft Secure.

### Automatically update the SAP data connector agent

You can now [enable automatic updates for the SAP data connector agent](sap/update-sap-data-connector.md#automatically-update-the-sap-data-connector-agent-preview). You can choose to apply automatic updates to all existing containers or to a specific container.

## February 2023

- [Out-of-the-box content centralization changes](#out-of-the-box-content-centralization-changes) (in [Announcements](#announcements) section below)
- [New CloudWatch data type for the AWS S3 connector (Preview)](#new-cloudwatch-data-type-for-the-aws-s3-connector)
- [Audit and monitor the health of your analytics rules (Preview)](#audit-and-monitor-the-health-of-your-analytics-rules-preview)
- [New behavior for alert grouping in analytics rules](#new-behavior-for-alert-grouping-in-analytics-rules) (in [Announcements](#announcements) section below)
- [Microsoft 365 Defender data connector is now generally available](#microsoft-365-defender-data-connector-is-now-generally-available)
- [Advanced scheduling for analytics rules (Preview)](#advanced-scheduling-for-analytics-rules-preview)

### New CloudWatch data type for the AWS S3 connector

The Microsoft Sentinel AWS S3 connector now supports [CloudWatch logs](connect-aws.md) in addition to the supported CloudTrail, VPC Flow, and Guard Duty logs. Logs from AWS CloudWatch provide operational information from different AWS sources, which enables Microsoft Sentinel customers with AWS footprints to better understand and operate their AWS systems and applications.  

The CloudWatch data type has the ability to perform the same data transformation functions as the other data types within the AWS S3 connector. Learn how to [transform your data for CloudWatch](../azure-monitor/logs/tutorial-workspace-transformations-portal.md).

### Audit and monitor the health of your analytics rules (Preview)

Microsoft Sentinel's **health monitoring feature is now available for analytics rules** in addition to automation rules, playbooks, and data connectors. Also now available for the first time, and currently only for analytics rules, is Microsoft Sentinel's **audit feature**. The audit feature collects information about any changes made to Sentinel resources (analytics rules) so that you can discover any unauthorized actions or tampering with the service.

Learn more about [auditing and health monitoring in Microsoft Sentinel](health-audit.md):
- [Turn on auditing and health monitoring for Microsoft Sentinel (preview)](enable-monitoring.md)
- [Monitor the health and audit the integrity of your analytics rules](monitor-analytics-rule-integrity.md)
- Explore the new [Analytics Health & Audit workbook](monitor-analytics-rule-integrity.md#use-the-auditing-and-health-monitoring-workbook).

### Microsoft 365 Defender data connector is now generally available

Microsoft 365 Defender incidents, alerts, and raw event data can be ingested into Microsoft Sentinel using this connector. It also enables the bi-directional synchronization of incidents between Microsoft 365 Defender and Microsoft Sentinel. This integration allows you to manage all of your incidents in Microsoft Sentinel, while taking advantage of Microsoft 365 Defender's specialized tools and capabilities to investigate those incidents that originated in Microsoft 365.

- Learn more about [Microsoft 365 Defender integration with Microsoft Sentinel](microsoft-365-defender-sentinel-integration.md).
- Learn how to [connect Microsoft 365 Defender to Microsoft Sentinel](connect-microsoft-365-defender.md).

### Advanced scheduling for analytics rules (Preview)

To give you more flexibility in scheduling your analytics rule execution times and to help you avoid potential conflicts, Microsoft Sentinel now allows you to determine when newly created analytics rules will run for the first time. The default behavior is as it has been: for them to run immediately upon creation.

[Learn more about advanced scheduling](detect-threats-custom.md#query-scheduling-and-alert-threshold).


## Announcements

- [Classic alert automation due for deprecation](#classic-alert-automation-due-for-deprecation)
- [When disconnecting and connecting the MDI alerts connector - UniqueExternalId field is not populated (use the AlertName field)](#when-disconnecting-and-connecting-the-mdi-alerts-connector---uniqueexternalid-field-is-not-populated-use-the-alertname-field)
- [Microsoft Defender for Identity alerts will no longer refer to the MDA policies in the Alert ExternalLinks properties](#microsoft-defender-for-identity-alerts-will-no-longer-refer-to-the-mda-policies-in-the-alert-externallinks-properties)
- [WindowsEvent table enhancements](#windowsevent-table-enhancements)
- [Out-of-the-box content centralization changes](#out-of-the-box-content-centralization-changes)
- [New behavior for alert grouping in analytics rules](#new-behavior-for-alert-grouping-in-analytics-rules)
- [Microsoft 365 Defender now integrates Azure Active Directory Identity Protection (AADIP)](#microsoft-365-defender-now-integrates-azure-active-directory-identity-protection-aadip)
- [Account enrichment fields removed from Azure AD Identity Protection connector](#account-enrichment-fields-removed-from-azure-ad-identity-protection-connector)
- [Name fields removed from UEBA UserPeerAnalytics table](#name-fields-removed-from-ueba-userpeeranalytics-table)

### Classic alert automation due for deprecation

Automated responses to alerts, in the form of playbooks, can be run in one of two ways:
- **Classic:** adding the playbook to the list under **Alert automation (classic)** in the **Automated response** tab of the analytics rule that produced the alert.

- **Automation rule:** creating an automation rule to run in response to the creation of alerts, and the automation rule will run the playbook. This method has [several advantages, as described here](migrate-playbooks-to-automation-rules.md).

As of **June 2023**, you can no longer add playbooks to be run using the **Classic** method; rather, you must use [automation rules](automate-incident-handling-with-automation-rules.md).

Playbooks in the existing **Classic** lists will continue to run until this method's scheduled deprecation in **March 2026**.

We strongly encourage you to migrate any remaining playbooks in your **Classic** lists to run from automation rules instead. [Learn how to migrate playbooks to automation rules](migrate-playbooks-to-automation-rules.md).

### When disconnecting and connecting the MDI alerts connector - UniqueExternalId field is not populated (use the AlertName field) 

The Microsoft Defender for Identity alerts now support the Government Community Cloud (GCC). To enable this support, there is a change to the way alerts are sent to Microsoft Sentinel. 

For customers connecting and disconnecting the MDI alerts connector, the `UniqueExternalId` field is no longer populated. The `UniqueExternalId` represents the alert, and was formerly located in the`ExtendedProperties` field. You can now obtain the ID through the `AlertName` field, which contains the alert’s name. 

Review the [complete mapping between the alert names and unique external IDs](/defender-for-identity/alerts-overview#security-alert-name-mapping-and-unique-external-ids).

### Microsoft Defender for Identity alerts will no longer refer to the MDA policies in the Alert ExternalLinks properties

Microsoft Defender for Identity alerts will no longer refer to the MDA policies in the Alert ExternalLinks properties due to a change in infrastructure performed on MDIs. Alerts will no longer contain any MDA links under **ExtendedLinks** with a **Label** that starts with **Defender for Cloud Apps**. This change will take effect April 30th, 2023. [Read more about this change](/defender-for-identity/whats-new#defender-for-identity-release-2198). 

### WindowsEvent table enhancements

The WindowsEvent schema has been expanded to include new fields, such as `Keywords`, `Version`, `Opcode`, `Correlation`, `SystemProcessId`, `SystemThreadId` and `EventRecordId`. 

These additions allow for more comprehensive analysis and for more information to be extracted and parsed from the event. 

If you aren't interested in ingesting the new fields, use ingest-time transformation in the AMA DCR to filter and drop the fields, while still ingesting the events. To ingest the events, add the following to your DCRs: 

```kusto
"transformKql": "source | project-away TimeCreated, SystemThreadId, EventRecordId, SystemProcessId, Correlation, Keywords, Opcode, SystemUserId, Version"
```
Learn more about [ingest-time transformations](../azure-monitor/essentials/data-collection-transformations.md).

### Out-of-the-box content centralization changes
A new banner has appeared in Microsoft Sentinel gallery pages! This informational banner has rolled out to all tenants to explain upcoming changes regarding out-of-the-box (OOTB) content. In short, the **Content hub** will be the central source whether you're looking for standalone content or packaged solutions. Banners appear in the templates section of **Workbooks**, **Hunting**, **Automation**, **Analytics** and **Data connectors** galleries. Here's an example of the banner in the **Workbooks** gallery. 

:::image type="complex" source="media/whats-new/example-content-central-change-banner.png" alt-text="Screenshot shows an example informational banner in the **Workbooks** gallery." lightbox="media/whats-new/example-content-central-change-banner.png":::
The banner reads, 'All Workbook templates, and additional out-of-the-box (OOTB) content are now centrally available in Content hub. Starting Q2 2023, only Workbook templates installed from the content hub will be available in this gallery. Learn more about the OOTB content centralization changes.'
:::image-end:::

As part of this centralization change, the **Next Steps** tab on data connector pages [has been deprecated](sentinel-content-centralize.md#data-connector-page-change).

For all the details on what these upcoming changes will mean for you, see [Microsoft Sentinel out-of-the-box content centralization changes](sentinel-content-centralize.md).

### New behavior for alert grouping in analytics rules

Starting **February 6, 2023** and continuing through the end of February, Microsoft Sentinel is rolling out a change in the way that incidents are created from analytics rules with certain event and alert grouping settings, and also the way that such incidents are updated by automation rules. This change is being made in order to produce incidents with more complete information and to simplify automation triggered by the creating and updating of incidents.

The affected analytics rules are those with both of the following two settings:
- **Event grouping** is set to **Trigger an alert for each event** (sometimes referred to as "alert per row" or "alert per result").
- **Alert grouping** is enabled, in any one of the [three possible configurations](detect-threats-custom.md#alert-grouping).

#### The problem

Rules with these two settings generate unique alerts for each event (result) returned by the query. These alerts are then all grouped together into a single incident (or a small number of incidents, depending on the alert grouping configuration choice).

The problem is that the incident is created as soon as the first alert is generated, so at that point the incident contains only the first alert. The remaining alerts are joined to the incident, one after the other, as they are generated. So you end up with a *single running of an analytics rule* resulting in:
- One incident creation event *and*
- Up to 149 incident update events

These circumstances result in unpredictable behavior when evaluating the conditions defined in automation rules or populating the incident schema in playbooks:

- **Incorrect evaluation of an incident's conditions by automation rules:**

    Automation rules on this incident will run immediately on its creation, even with just the one alert included. So the automation rule will only consider the incident's status as containing the first alert, even though other alerts are being created nearly simultaneously (by the same running of the analytics rule) and will continue being added while the automation rule is running. So you end up with a situation where the automation rule's evaluation of the incident is incomplete and likely incorrect.

    If there are automation rules defined to run when the incident is *updated*, they will run again and again as each subsequent alert is added to the incident (even though the alerts were all generated by the same running of the analytics rule). So you'll have alerts being added and automation rules running, each time possibly incorrectly evaluating the conditions of the incident.

    Automation rules' conditions might ignore entities that only later become part of the incident but weren't included in the first alert/creation of the incident.

    In these cases, incorrect evaluation of an incident's condition may cause automation rules to run when they shouldn't, or not to run when they should. The result of this would be that the wrong actions would be taken on an incident, or that the right actions would not be taken.

- **Information in later alerts being unavailable to playbooks run on the incident:**

    When an automation rule calls a playbook, it passes the incident's detailed information to the playbook. Because of the behavior mentioned above, a playbook might only receive the details (entities, custom details, and so on) of the first alert in an incident, but not those from subsequent alerts. This means that the playbook's actions would not have access to all the information in the incident.

#### The solution

Going forward, instead of creating the incident as soon as the first alert is generated, Microsoft Sentinel will wait until a single running of an analytics rule has generated all of its alerts, and then create the incident, adding all the alerts to it at once. So instead of an incident creation event and a whole bunch of incident update events, you have only the incident creation event.

Now, automation rules that run on the creation of an incident can evaluate the complete incident with all of its alerts (as well as entities and custom details) and its most updated properties, and any playbooks that run will similarly have the complete details of the incident.



The following table describes the change in the incident creation and automation behaviors:

| When incident created/updated with multiple alerts | Before the change | After the change |
| -- | -- | -- |
| **Automation rule** conditions are evaluated based on... | The first alert generated by the current running of the analytics rule. | All alerts and entities resulting from the current running of the analytics rule. |
| **Playbook input** includes... | - Alerts list containing only the first alert of the incident.<br>- Entities list containing only entities from the first alert of the incident. | - Alerts list containing all the alerts triggered by this rule execution and grouped to this incident.<br>- Entities list containing the entities from all the alerts triggered by this rule execution and grouped to this incident. |
| **SecurityIncident** table in Log Analytics shows... | -&nbsp;One&nbsp;row&nbsp;for&nbsp;*incident&nbsp;created*&nbsp;with&nbsp;one&nbsp;alert.<br>- Multiple&nbsp;events&nbsp;of&nbsp;*alert&nbsp;added*. | One row for *incident created* only after all alerts triggered by this rule execution have been added and grouped to this incident. |

### Microsoft 365 Defender now integrates Azure Active Directory Identity Protection (AADIP)

As of **October 24, 2022**, [Microsoft 365 Defender](/microsoft-365/security/defender/) integrates [Azure Active Directory Identity Protection (AADIP)](../active-directory/identity-protection/index.yml) alerts and incidents. Customers can choose between three levels of integration:

- **Show high-impact alerts only (Default)** includes only alerts about known malicious or highly suspicious activities that might require attention. These alerts are  chosen by Microsoft security researchers and are mostly of Medium and High severities.
- **Show all alerts** includes all AADIP alerts, including activity that might not be unwanted or malicious.
- **Turn off all alerts** disables any AADIP alerts from appearing in your Microsoft 365 Defender incidents.

Microsoft Sentinel customers (who are also AADIP subscribers) with [Microsoft 365 Defender integration](microsoft-365-defender-sentinel-integration.md) enabled now automatically receive AADIP alerts and incidents in their Microsoft Sentinel incidents queue. Depending on your configuration, this may affect you as follows:

- If you already have your AADIP connector enabled in Microsoft Sentinel, and you've enabled incident creation, you may receive duplicate incidents. To avoid this, you have a few choices, listed here in descending order of preference:

    | Preference | Action in Microsoft 365 Defender | Action in Microsoft Sentinel |
    | - | - | - |
    | **1** | Keep the default AADIP integration of **Show high-impact alerts only**. | Disable any [**Microsoft Security** analytics rules](detect-threats-built-in.md) that create incidents from AADIP alerts. |
    | **2** | Choose the **Show all alerts** AADIP integration. | Create automation rules to automatically close incidents with unwanted alerts.<br><br>Disable any [**Microsoft Security** analytics rules](detect-threats-built-in.md) that create incidents from AADIP alerts. |
    | **3** | Don't use Microsoft 365 Defender for AADIP alerts:<br>Choose the **Turn off all alerts** option for AADIP integration. | Leave enabled those [**Microsoft Security** analytics rules](detect-threats-built-in.md) that create incidents from AADIP alerts. |

    See the [Microsoft 365 Defender documentation for instructions](/microsoft-365/security/defender/investigate-alerts#configure-aad-ip-alert-service) on how to take the prescribed actions in Microsoft 365 Defender.

- If you don't have your [AADIP connector](data-connectors/azure-active-directory-identity-protection.md) enabled, you must enable it. Be sure **not** to enable incident creation on the connector page. If you don't enable the connector, you may receive AADIP incidents without any data in them.

- If you're first enabling your Microsoft 365 Defender connector now, the AADIP connection was made automatically behind the scenes. You won't need to do anything else.

### Account enrichment fields removed from Azure AD Identity Protection connector

As of **September 30, 2022**, alerts coming from the **Azure Active Directory Identity Protection connector** no longer contain the following fields:

- CompromisedEntity
- ExtendedProperties["User Account"]
- ExtendedProperties["User Name”]

We are working to adapt Microsoft Sentinel's built-in queries and other operations affected by this change to look up these values in other ways (using the *IdentityInfo* table).

In the meantime, or if you've built any custom queries or rules directly referencing these fields, you'll need another way to get this information. Use the following two-step process to have your queries look up these values in the *IdentityInfo* table:

1. If you haven't already, **enable the UEBA solution** to sync the *IdentityInfo* table with your Azure AD logs. Follow the instructions in [this document](enable-entity-behavior-analytics.md).  
(If you don't intend to use UEBA in general, you can ignore the last instruction  about selecting data sources on which to enable entity behavior analytics.)

1. Incorporate the query below in your existing queries or rules to look up this data by joining the *SecurityAlert* table with the *IdentityInfo* table.

    ```kusto
    SecurityAlert
    | where TimeGenerated > ago(7d)
    | where ProductName == "Azure Active Directory Identity Protection"
    | mv-expand Entity = todynamic(Entities)
    | where Entity.Type == "account"
    | extend AadTenantId = tostring(Entity.AadTenantId)
    | extend AadUserId = tostring(Entity.AadUserId)
    | join kind=inner (
        IdentityInfo
        | where TimeGenerated > ago(14d)
        | distinct AccountTenantId, AccountObjectId, AccountUPN, AccountDisplayName
        | extend UserAccount = AccountUPN
        | extend UserName = AccountDisplayName
        | where isnotempty(AccountDisplayName) and isnotempty(UserAccount)
        | project AccountTenantId, AccountObjectId, UserAccount, UserName
        )
        on
        $left.AadTenantId == $right.AccountTenantId,
        $left.AadUserId == $right.AccountObjectId
    | extend CompromisedEntity = iff(CompromisedEntity == "N/A" or isempty(CompromisedEntity), UserAccount, CompromisedEntity)
    | project-away AadTenantId, AadUserId, AccountTenantId, AccountObjectId
    ```

For information on looking up data to replace enrichment fields removed from the UEBA UserPeerAnalytics table, See [Name fields removed from UEBA UserPeerAnalytics table](#name-fields-removed-from-ueba-userpeeranalytics-table) for a sample query.

### Name fields removed from UEBA UserPeerAnalytics table

As of **September 30, 2022**, the UEBA engine no longer performs automatic lookups of user IDs and resolves them into names. This change resulted in the removal of four name fields from the *UserPeerAnalytics* table: 

- UserName
- UserPrincipalName
- PeerUserName
- PeerUserPrincipalName 

The corresponding ID fields remain part of the table, and any built-in queries and other operations execute the appropriate name lookups in other ways (using the IdentityInfo table), so you shouldn’t be affected by this change in nearly all circumstances. 

The only exception to this is if you’ve built custom queries or rules directly referencing any of these name fields. In this scenario, you can incorporate the following lookup queries into your own, so you can access the values that would have been in these name fields. 

The following query resolves **user** and **peer identifier fields**: 

```kusto
UserPeerAnalytics 
| where TimeGenerated > ago(24h) 
// join to resolve user identifier fields 
| join kind=inner ( 
    IdentityInfo  
    | where TimeGenerated > ago(14d) 
    | distinct AccountTenantId, AccountObjectId, AccountUPN, AccountDisplayName 
    | extend UserPrincipalNameIdentityInfo = AccountUPN 
    | extend UserNameIdentityInfo = AccountDisplayName 
    | project AccountTenantId, AccountObjectId, UserPrincipalNameIdentityInfo, UserNameIdentityInfo 
) on $left.AADTenantId == $right.AccountTenantId, $left.UserId == $right.AccountObjectId 
// join to resolve peer identifier fields 
| join kind=inner ( 
    IdentityInfo  
    | where TimeGenerated > ago(14d) 
    | distinct AccountTenantId, AccountObjectId, AccountUPN, AccountDisplayName 
    | extend PeerUserPrincipalNameIdentityInfo = AccountUPN 
    | extend PeerUserNameIdentityInfo = AccountDisplayName 
    | project AccountTenantId, AccountObjectId, PeerUserPrincipalNameIdentityInfo, PeerUserNameIdentityInfo 
) on $left.AADTenantId == $right.AccountTenantId, $left.PeerUserId == $right.AccountObjectId 
```
If your original query referenced the user or peer names (not just their IDs), substitute this query in its entirety for the table name (“UserPeerAnalytics”) in your original query.

## Next steps

> [!div class="nextstepaction"]
>[On-board Azure Sentinel](quickstart-onboard.md)

> [!div class="nextstepaction"]
>[Get visibility into alerts](get-visibility.md)
