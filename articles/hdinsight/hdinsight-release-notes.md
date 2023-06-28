---
title: Release notes for Azure HDInsight 
description: Latest release notes for Azure HDInsight. Get development tips and details for Hadoop, Spark, Hive, and more.
ms.custom: references_regions
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/12/2023
---

# Azure HDInsight release notes

This article provides information about the **most recent** Azure HDInsight release updates. For information on earlier releases, see [HDInsight Release Notes Archive](hdinsight-release-notes-archive.md).

## Summary

Azure HDInsight is one of the most popular services among enterprise customers for open-source analytics on Azure.
Subscribe to the [HDInsight Release Notes](./subscribe-to-hdi-release-notes-repo.md) for up-to-date information on HDInsight and all HDInsight versions.

To subscribe, click the “watch” button in the banner and watch out for [HDInsight Releases](https://github.com/Azure/HDInsight/releases).

## Release date: May 08, 2023

This release applies to HDInsight 4.x and 5.x HDInsight release will be available to all regions over several days. This release is applicable for image number **2304280205**. [How to check the image number?](./view-hindsight-cluster-image-version.md)

HDInsight uses safe deployment practices, which involve gradual region deployment. it may take up to 10 business days for a new release or a new version to be available in all regions.

**OS versions**

* HDInsight 4.0: Ubuntu 18.04.5 LTS Linux Kernel 5.4
* HDInsight 5.0: Ubuntu 18.04.5 LTS Linux Kernel 5.4

For workload specific versions, see 

* [HDInsight 5.x component versions](./hdinsight-5x-component-versioning.md)
* [HDInsight 4.x component versions](./hdinsight-40-component-versioning.md)

![Icon showing update with text.](media/hdinsight-release-notes/new-icon-for-updated.png)

1. Azure HDInsight 5.1 updated with

    1. Apache HBase 2.4.11
    1. Apache Phoenix 5.1.2
    1. Apache Hive 3.1.2
    1. Apache Spark 3.3.1
    1. Apache Tez 0.9.1
    1. Apache Zeppelin 0.10.1
    1. Apache Livy 0.5
    1. Apache Kafka 3.2.0

    > [!NOTE]
    > * All components are integrated with Hadoop 3.3.4 & ZK 3.6.3
    > * All above upgraded components are now available in non-ESP clusters for public preview.

![Icon showing new features with text.](media/hdinsight-release-notes/new-icon-for-new-feature.png)

1. **Enhanced Autoscale for HDInsight**

   Azure HDInsight has made notable improvements stability and latency on Autoscale, The essential changes include improved feedback loop for scaling decisions, significant improvement on latency for scaling and support for recommissioning the decommissioned nodes, Learn [more](https://techcommunity.microsoft.com/t5/analytics-on-azure-blog/enhanced-autoscale-capabilities-in-hdinsight-clusters/ba-p/3811271) about the enhancements, how to custom configure and migrate your cluster to enhanced autoscale. The enhanced Autoscale capability is available effective 17 May, 2023 across all supported regions.
    
1. **Azure HDInsight ESP for Apache Kafka 2.4.1 is now Generally Available**.

   Azure HDInsight ESP for Apache Kafka 2.4.1 has been in public preview since April 2022. After notable improvements in CVE fixes and stability, Azure HDInsight ESP Kafka 2.4.1 now becomes generally available and ready for production workloads, learn the detail about the [how to configure](./domain-joined/apache-domain-joined-run-kafka.md) and [migrate](./kafka/migrate-versions.md).

1. **Quota Management for HDInsight**

    HDInsight currently allocates quota to customer subscriptions at a regional level. The cores allocated to customers are generic and not classified at a VM family level (For example, Dv2, Ev3, Eav4, etc.).
    
    HDInsight introduced an improved view, which provides a detail and classification of quotas for family-level VMs, this feature allows customers to view current and remaining quotas for a region at the VM family level. With the enhanced view, customers have richer visibility, for planning quotas, and a better user experience. This feature is currently available on HDInsight 4.x and 5.x for East US EUAP region. Other regions to follow later.

    For more information, see [Cluster capacity planning in Azure HDInsight | Microsoft Learn](./hdinsight-capacity-planning.md#view-quota-management-for-hdinsight)
    
![Icon showing new regions added with text.](media/hdinsight-release-notes/new-icon-for-new-regions-added.png) 

* Poland Central

## Coming soon

* The max length of cluster name will be changed to 45 from 59 characters, to improve the security posture of clusters.
* Cluster permissions for secure storage  
  * Customers can specify (during cluster creation) whether a secure channel should be used for HDInsight cluster nodes to contact the storage account. 
* In-line quota update.
   * Request quotas increase directly from the My Quota page, which will be a direct API call, which is faster. If the API call fails, then customers need to create a new support request for quota increase.
* HDInsight Cluster Creation with Custom VNets.
  * To improve the overall security posture of the HDInsight clusters, HDInsight clusters using custom VNETs will need to ensure that the user needs to have permission for `Microsoft Network/virtualNetworks/subnets/join/action` to perform create operations. Customers would need to plan accordingly as this would be a mandatory check to avoid cluster creation failures.
* Basic and Standard A-series VMs Retirement.
   * On 31 August 2024, we'll retire Basic and Standard A-series VMs. Before that date, you need to migrate your workloads to Av2-series VMs, which provide more memory per vCPU and faster storage on solid-state drives (SSDs). To avoid service disruptions, [migrate your workloads](https://aka.ms/Av1retirement) from Basic and Standard A-series VMs to Av2-series VMs before 31 August 2024.
* Non-ESP ABFS clusters [Cluster Permissions for World Readable] 
  * Plan to introduce a change in non-ESP ABFS clusters, which restricts non-Hadoop group users from executing Hadoop commands for storage operations. This change to improve cluster security posture. Customers need to plan for the updates.

If you have any more questions, contact [Azure Support](https://ms.portal.azure.com/#view/Microsoft_Azure_Support/HelpAndSupportBlade/~/overview).

You can always ask us about HDInsight on [Azure HDInsight - Microsoft Q&A](https://learn.microsoft.com/answers/tags/168/azure-hdinsight)

You’re welcome to add more proposals and ideas and other topics here and vote for them - [HDInsight Community (azure.com)](https://feedback.azure.com/d365community/search/?q=HDInsight) and follow us for more  updates on [twitter](https://twitter.com/AzureHDInsight)

 > [!NOTE]
 > We advise customers to use to latest versions of HDInsight [Images](./view-hindsight-cluster-image-version.md) as they bring in the best of open source updates,  Azure updates and security fixes. For more information, see [Best practices](./hdinsight-overview-before-you-start.md).

### Next steps
* [Azure HDInsight: Frequently asked questions](./hdinsight-faq.yml)
* [Configure the OS patching schedule for Linux-based HDInsight clusters](./hdinsight-os-patching.md)
