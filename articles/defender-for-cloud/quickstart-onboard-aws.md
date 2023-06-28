---
title: Connect your AWS account to Microsoft Defender for Cloud
description: Defend your AWS resources with Microsoft Defender for Cloud
ms.topic: quickstart
ms.date: 06/26/2023
author: dcurwin
ms.author: dacurwin
zone_pivot_groups: connect-aws-accounts
ms.custom: mode-other, ignite-2022
---
# Quickstart: Connect your AWS accounts to Microsoft Defender for Cloud

With cloud workloads commonly spanning multiple cloud platforms, cloud security services must do the same. Microsoft Defender for Cloud protects workloads in Azure, Amazon Web Services (AWS), Google Cloud Platform (GCP), GitHub and Azure DevOps (ADO).

To protect your AWS-based resources, you can connect an AWS account with either:

- **Native cloud connector** (recommended) - Provides an agentless connection to your AWS account that you can extend with Defender for Cloud's Defender plans to secure your AWS resources:

  - [**Cloud Security Posture Management (CSPM)**](overview-page.md) assesses your AWS resources according to AWS-specific security recommendations and reflects your security posture in your secure score. The [asset inventory](asset-inventory.md) gives you one place to see all of your protected AWS resources. The [regulatory compliance dashboard](regulatory-compliance-dashboard.md) shows your compliance with built-in standards specific to AWS, including AWS CIS, AWS PCI DSS, and AWS Foundational Security Best Practices.
  - [**Microsoft Defender for Servers**](defender-for-servers-introduction.md) brings threat detection and advanced defenses to [supported Windows and Linux EC2 instances](supported-machines-endpoint-solutions-clouds-servers.md?tabs=tab/features-multicloud).
  - [**Microsoft Defender for Containers**](defender-for-containers-introduction.md) brings threat detection and advanced defenses to [supported Amazon EKS clusters](supported-machines-endpoint-solutions-clouds-containers.md).
  - [**Microsoft Defender for SQL**](defender-for-sql-introduction.md) brings threat detection and advanced defenses to your SQL Servers running on AWS EC2, AWS RDS Custom for SQL Server.

- **Classic cloud connector** - Requires configuration in your AWS account to create a user that Defender for Cloud can use to connect to your AWS environment.

> [!NOTE]
> The option to select the classic connector is only available if you previously onboarded an AWS account using the classic connector.
>
> If you have classic cloud connectors, we recommend that you [delete these connectors](#remove-classic-connectors), and use the native connector to reconnect to the account. Using both the classic and native connectors can produce duplicate recommendations.

For a reference list of all the recommendations Defender for Cloud can provide for AWS resources, see [Security recommendations for AWS resources - a reference guide](recommendations-reference-aws.md).

This screenshot shows AWS accounts displayed in Defender for Cloud's [overview dashboard](overview-page.md).

:::image type="content" source="./media/quickstart-onboard-aws/aws-account-in-overview.png" alt-text="Four AWS projects listed on Defender for Cloud's overview dashboard" lightbox="./media/quickstart-onboard-aws/aws-account-in-overview.png":::

You can learn more by watching this video from the Defender for Cloud in the Field video series:

- [New AWS connector](episode-one.md)

::: zone pivot="env-settings"

## Availability

|Aspect|Details|
|----|:----|
|Release state:|General Availability (GA)|
|Pricing:|The **[Defender for SQL](defender-for-sql-introduction.md)** plan is billed at the same price as Azure resources.<br>The **[Defender for Containers](defender-for-containers-introduction.md)** plan is free during the preview. After which, it will be billed for AWS at the same price as for Azure resources.<br>For every AWS machine connected to Azure, the **Defender for Servers** plan is billed at the same price as the [Microsoft Defender for Servers](defender-for-servers-introduction.md) plan for Azure machines.<br>Learn more about [Defender plan pricing and billing](enhanced-security-features-overview.md#faq---pricing-and-billing)|
|Required roles and permissions:|**Contributor** permission for the relevant Azure subscription. <br> **Administrator** on the AWS account.|
|Clouds:|:::image type="icon" source="./media/icons/yes-icon.png"::: Commercial clouds<br>:::image type="icon" source="./media/icons/no-icon.png"::: National (Azure Government, Azure China 21Vianet)|

## Prerequisites

The native cloud connector requires:

- Access to an AWS account.

- **To enable the Defender for Containers plan**, you'll need:
  - At least one Amazon EKS cluster with permission to access to the EKS K8s API server. If you need to create a new EKS cluster, follow the instructions in [Getting started with Amazon EKS – eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html).
  - The resource capacity to create a new SQS queue, Kinesis Fire Hose delivery stream, and S3 bucket in the cluster's region.

- **To enable the Defender for SQL plan**, you'll need:

  - Microsoft Defender for SQL enabled on your subscription. Learn how to [enable protection on all of your databases](quickstart-enable-database-protections.md).

    - An active AWS account, with EC2 instances running SQL server or RDS Custom for SQL Server.

    - Azure Arc for servers installed on your EC2 instances/RDS Custom for SQL Server.
      - (Recommended) Use the auto provisioning process to install Azure Arc on all of your existing and future EC2 instances.

          Auto provisioning is managed by AWS Systems Manager (SSM) using the SSM agent. Some Amazon Machine Images (AMIs) already have the SSM agent pre-installed. If you already have the SSM agent pre-installed, the AMIs are listed in [AMIs with SSM Agent preinstalled](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-technical-details.html#ami-preinstalled-agent). If your EC2 instances don't have the SSM Agent, you'll need to install it using either of the following relevant instructions from Amazon:
            - [Install SSM Agent for a hybrid environment (Windows)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-win.html)
            Ensure that your SSM agent has the managed policy ["AmazonSSMManagedInstanceCore"](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonSSMManagedInstanceCore.html) that enables AWS Systems Manager service core functionality.

        > [!NOTE]
        > To enable the Azure Arc auto-provisioning, you'll need **Owner** permission on the relevant Azure subscription.

    - Other extensions should be enabled on the Arc-connected machines:
      - Microsoft Defender for Endpoint
      - VA solution (TVM/Qualys)
      - Log Analytics (LA) agent on Arc machines or Azure Monitor agent (AMA)

           Make sure the selected LA workspace has security solution installed. The LA agent and AMA are currently configured in the subscription level. All of your AWS accounts and GCP projects under the same subscription will inherit the subscription settings for the LA agent and AMA.

        Learn more about [monitoring components](monitoring-components.md) for Defender for Cloud.

- **To enable the Defender for Servers plan**, you'll need:

  - Microsoft Defender for Servers enabled on your subscription. Learn how to enable plans in [Enable enhanced security features](enable-enhanced-security.md).

    - An active AWS account, with EC2 instances.

    - Azure Arc for servers installed on your EC2 instances.
      - (Recommended) Use the auto provisioning process to install Azure Arc on all of your existing and future EC2 instances.

           Auto provisioning is managed by AWS Systems Manager (SSM) using the SSM agent. Some Amazon Machine Images (AMIs) already have the SSM agent pre-installed. If that is the case, their AMIs are listed in [AMIs with SSM Agent preinstalled](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-technical-details.html#ami-preinstalled-agent). If your EC2 instances don't have the SSM Agent, you'll need to install it using either of the following relevant instructions from Amazon:
            - [Install SSM Agent for a hybrid environment (Windows)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-win.html)
            - [Install SSM Agent for a hybrid environment (Linux)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-linux.html)
            Ensure that your SSM agent has the managed policy ["AmazonSSMManagedInstanceCore"](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonSSMManagedInstanceCore.html) that enables AWS Systems Manager service core functionality.

        > [!NOTE]
        > To enable the Azure Arc auto-provisioning, you'll need an **Owner** permission on the relevant Azure subscription.

        - If you want to manually install Azure Arc on your existing and future EC2 instances, use the [EC2 instances should be connected to Azure Arc](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/231dee23-84db-44d2-bd9d-c32fbcfb42a3) recommendation to identify instances that don't have Azure Arc installed.

    - Other extensions should be enabled on the Arc-connected machines:
      - Microsoft Defender for Endpoint
      - VA solution (TVM/Qualys)
      - Log Analytics (LA) agent on Arc machines or Azure Monitor agent (AMA)

           Make sure the selected LA workspace has security solution installed. The LA agent and AMA are currently configured in the subscription level. All of your AWS accounts and GCP projects under the same subscription will inherit the subscription settings for the LA agent and AMA.

        Learn more about [monitoring components](monitoring-components.md) for Defender for Cloud.

        > [!NOTE]
        > Defender for Servers assigns tags to your AWS resources to manage the auto-provisioning process. You must have these tags properly assigned to your resources so that Defender for Cloud can manage your resources:
        **AccountId**, **Cloud**, **InstanceId**, **MDFCSecurityConnector**

## Connect your AWS account

**To connect your AWS account to Defender for Cloud with a native connector**:

1. If you have any classic connectors, [remove them](#remove-classic-connectors).

    Using both the classic and native connectors can produce duplicate recommendations.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Navigate to **Defender for Cloud** > **Environment settings**.

1. Select **Add environment** > **Amazon Web Services**.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-environment-settings.png" alt-text="Connecting an AWS account to an Azure subscription.":::

1. Enter the details of the AWS account, including the location where you'll store the connector resource. You can also scan specific AWS regions or all available regions (default) on AWS public cloud.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-details.png" alt-text="Step 1 of the add AWS account wizard: Enter the account details.":::

   (Optional) Select **Management account** to create a connector to a management account. Connectors will be created for each member account discovered under the provided management account. Auto-provisioning will be enabled for all of the newly onboarded accounts.

    > [!NOTE]
    > Defender for Cloud can be connected to each AWS account or management account only once.

1. Select **Next: Select plans**.<a name="cloudtrail-implications-note"></a>

    > [!NOTE]
    > Each plan has its own requirements for permissions, and might incur charges.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-plans-selection.png" alt-text="The select plans tab is where you choose which Defender for Cloud capabilities to enable for this AWS account." lightbox="media/quickstart-onboard-aws/add-aws-account-plans-selection.png":::

    > [!IMPORTANT]
    > To present the current status of your recommendations, the CSPM plan queries the AWS resource APIs several times a day. These read-only API calls incur no charges, but they *are* registered in CloudTrail if you've enabled a trail for read events. As explained in [the AWS documentation](https://aws.amazon.com/cloudtrail/pricing/), there are no additional charges for keeping one trail. If you're exporting the data out of AWS (for example, to an external SIEM), this increased volume of calls might also increase ingestion costs. In such cases, We recommend filtering out the read-only calls from the Defender for Cloud user or role ARN: `arn:aws:iam::[accountId]:role/CspmMonitorAws` (this is the default role name, confirm the role name configured on your account).

1. By default the **Servers** plan is set to **On**. This is necessary to extend Defender for server's coverage to your AWS EC2. Ensure you've fulfilled the [network requirements for Azure Arc](../azure-arc/servers/network-requirements.md?tabs=azure-cloud).

    - (Optional) Select **Configure**, to edit the configuration as required.

    > [!NOTE]
    > The respective Azure Arc servers for EC2 instances or GCP virtual machines that no longer exist (and the respective Azure Arc servers with a status of ["Disconnected" or "Expired"](https://learn.microsoft.com/azure/azure-arc/servers/overview)) will be removed after 7 days. This process removes irrelevant Azure ARC entities, ensuring only Azure Arc servers related to existing instances are displayed.

1. By default the **Containers** plan is set to **On**. This is necessary to have Defender for Containers protect your AWS EKS clusters. Ensure you've fulfilled the [network requirements](./defender-for-containers-enable.md?pivots=defender-for-container-eks&source=docs&tabs=aks-deploy-portal%2ck8s-deploy-asc%2ck8s-verify-asc%2ck8s-remove-arc%2caks-removeprofile-api#network-requirements) for the Defender for Containers plan.

    > [!Note]
    > Azure Arc-enabled Kubernetes, the Defender Arc extension, and the Azure Policy Arc extension should be installed. Use the dedicated Defender for Cloud recommendations to deploy the extensions (and Arc, if necessary) as explained in [Protect Amazon Elastic Kubernetes Service clusters](defender-for-containers-enable.md?tabs=defender-for-container-eks).

    - (Optional) Select **Configure**, to edit the configuration as required. If you choose to disable this configuration, the `Threat detection (control plane)` feature will be disabled. Learn more about the [feature availability](supported-machines-endpoint-solutions-clouds-containers.md).

1. By default the **Databases** plan is set to **On**. This is necessary to extend Defender for SQL's coverage to your AWS EC2 and RDS Custom for SQL Server.

    - (Optional) Select **Configure**, to edit the configuration as required. We recommend you leave it set to the default configuration.

1. Select **Next: Configure access**.

    a. Choose deployment type, **Default access** or **Least privilege access**.

    - Default access - Allows Defender for Cloud to scan your resources and automatically include future capabilities.
    - Least privileged access - Grants Defender for Cloud access only to the current permissions needed for the selected plans. If you select the least privileged permissions, you'll receive notifications on any new roles and permissions that are required to get full functionality on the connector health section.

    b. Choose deployment method: **AWS CloudFormation** or **Terraform**.

    :::image type="content" source="media/quickstart-onboard-aws/aws-configure-access.png" alt-text="Screenshot showing the configure access and its deployment options and instructions.":::

1. Follow the on-screen instructions for the selected deployment method to complete the required dependencies on AWS. If you're onboarding a management account, you'll need to run the CloudFormation template both as Stack and as StackSet. Connectors will be created for the member accounts up to 24 hours after the onboarding.

1. Select **Next: Review and generate**.

1. Select **Create**.

Defender for Cloud will immediately start scanning your AWS resources and you'll see security recommendations within a few hours. For a reference list of all the recommendations Defender for Cloud can provide for AWS resources, see [Security recommendations for AWS resources - a reference guide](recommendations-reference-aws.md).

## AWS authentication process

Federated authentication is used between Microsoft Defender for Cloud and AWS. All of the resources related to the authentication are created as a part of the CloudFormation template deployment, including:

- An identity provider (OpenID connect)
- Identity and Access Management (IAM) roles with a federated principal (connected to the identity providers).

The architecture of the authentication process across clouds is as follows:

:::image type="content" source="media/quickstart-onboard-aws/architecture-authentication-across-clouds.png" alt-text="diagram showing architecture of authentication process across clouds." lightbox="media/quickstart-onboard-aws/architecture-authentication-across-clouds.png":::

1. Microsoft Defender for Cloud CSPM service acquires an Azure AD token with a validity life time of 1 hour that is signed by the Azure AD using the RS256 algorithm.

1. The Azure AD token is exchanged with AWS short living credentials and Defender for Cloud's CPSM service assumes the CSPM IAM role (assumed with web identity).

1. Since the principle of the role is a federated identity as defined in a trust relationship policy, the AWS identity provider validates the Azure AD token against the Azure AD through a process that includes:

    - audience validation
    - signing of the token
    - certificate thumbprint

1. The Microsoft Defender for Cloud CSPM role is assumed only after the validation conditions defined at the trust relationship have been met. The conditions defined for the role level are used for validation within AWS and allows only the Microsoft Defender for Cloud CSPM application (validated audience) access to the specific role (and not any other Microsoft token).

1. After the Azure AD token is validated by the AWS identity provider, the AWS STS exchanges the token with AWS short-living credentials which CSPM service uses to scan the AWS account.

## CloudFormation deployment source

As part of connecting an AWS account to Microsoft Defender for Cloud, a CloudFormation template should be deployed to the AWS account. This CloudFormation template creates all of the required resources necessary for Microsoft Defender for Cloud to connect to the AWS account.

The CloudFormation template should be deployed using Stack (or StackSet if you have a management account).

When deploying the CloudFormation template, the Stack creation wizard offers the following options:

:::image type="content" source="media/quickstart-onboard-aws/cloudformation-template.png" alt-text="Screenshot showing stack creation wizard." lightbox="media/quickstart-onboard-aws/cloudformation-template.png":::

1. **Amazon S3 URL** – upload the downloaded CloudFormation template to your own S3 bucket with your own security configurations. Enter the URL to the S3 bucket in the AWS deployment wizard.

1. **Upload a template file** – AWS will automatically create an S3 bucket that the CloudFormation template will be saved to. The automation for the S3 bucket will have a security misconfiguration that will cause the `S3 buckets should require requests to use Secure Socket Layer` recommendation to appear. You can remediate this recommendation by applying the following policy:

    ```json
    {  
      "Id": "ExamplePolicy",  
      "Version": "2012-10-17",  
      "Statement": [  
        {  
          "Sid": "AllowSSLRequestsOnly",  
          "Action": "s3:*",  
          "Effect": "Deny",  
          "Resource": [  
            "<S3_Bucket ARN>",  
            "<S3_Bucket ARN>/*"  
          ],  
          "Condition": {  
            "Bool": {  
              "aws:SecureTransport": "false"  
            }  
          },  
          "Principal": "*"  
        }  
      ]  
    }  
    ```

### Remove 'classic' connectors

If you have any existing connectors created with the classic cloud connectors experience, remove them first:

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Navigate to **Defender for Cloud** > **Environment settings**.

1. Select the option to switch back to the classic connectors experience.

    :::image type="content" source="media/quickstart-onboard-gcp/classic-connectors-experience.png" alt-text="Switching back to the classic cloud connectors experience in Defender for Cloud.":::

1. For each connector, select the three-dot button **…** at the end of the row, and select **Delete**.

1. On AWS, delete the role ARN, or the credentials created for the integration.

::: zone-end

::: zone pivot="classic-connector"

## Availability

|Aspect|Details|
|----|:----|
|Release state:|General availability (GA)|
|Pricing:|Requires [Microsoft Defender for Servers Plan 2](plan-defender-for-servers-select-plan.md#plan-features)|
|Required roles and permissions:|**Owner** on the relevant Azure subscription<br>**Contributor** can also connect an AWS account if an owner provides the service principal details|
|Clouds:|:::image type="icon" source="./media/icons/yes-icon.png"::: Commercial clouds<br>:::image type="icon" source="./media/icons/no-icon.png"::: National (Azure Government, Azure China 21Vianet)|

## Connect your AWS account

Follow the steps below to create your AWS cloud connector.

### Step 1. Set up AWS Security Hub

1. To view security recommendations for multiple regions, repeat the following steps for each relevant region.

    > [!IMPORTANT]
    > If you're using an AWS management account, repeat the following three steps to configure the management account and all connected member accounts across all relevant regions

    1. Enable [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html).
    1. Enable [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-settingup.html).
    1. Verify that data is flowing to the Security Hub. When you first enable Security Hub, it might take several hours for data to be available.

### Step 2. Set up authentication for Defender for Cloud in AWS

There are two ways to allow Defender for Cloud to authenticate to AWS:

- **Create an IAM role for Defender for Cloud** (Recommended) - The most secure method
- **AWS user for Defender for Cloud** - A less secure option if you don't have IAM enabled

#### Create an IAM role for Defender for Cloud

1. From your Amazon Web Services console, under **Security, Identity & Compliance**, select **IAM**.
    :::image type="content" source="./media/quickstart-onboard-aws/aws-identity-and-compliance.png" alt-text="AWS services.":::

1. Select **Roles** and **Create role**.
1. Select **Another AWS account**.
1. Enter the following details:

    - **Account ID** - enter the Microsoft Account ID (**158177204117**) as shown in the AWS connector page in Defender for Cloud.
    - **Require External ID** - should be selected
    - **External ID** - enter the subscription ID as shown in the AWS connector page in Defender for Cloud.

1. Select **Next**.
1. In the **Attach permission policies** section, select the following [AWS managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html):

    - SecurityAudit (`arn:aws:iam::aws:policy/SecurityAudit`)
    - AmazonSSMAutomationRole (`arn:aws:iam::aws:policy/service-role/AmazonSSMAutomationRole`)
    - AWSSecurityHubReadOnlyAccess (`arn:aws:iam::aws:policy/AWSSecurityHubReadOnlyAccess`)

1. Optionally add tags. Adding Tags to the user doesn't affect the connection.
1. Select **Next**.

1. In The Roles list, choose the role you created

1. Save the Amazon Resource Name (ARN) for later.

#### Create an AWS user for Defender for Cloud

1. Open the **Users** tab and select **Add user**.
1. In the **Details** step, enter a username for Defender for Cloud and ensure that you select **Programmatic access** for the AWS Access Type.
1. Select **Next Permissions**.
1. Select **Attach existing policies directly** and apply the following policies:
    - SecurityAudit
    - AmazonSSMAutomationRole
    - AWSSecurityHubReadOnlyAccess

1. Select **Next: Tags**. Optionally add tags. Adding Tags to the user doesn't affect the connection.
1. Select **Review**.
1. Save the automatically generated **Access key ID** and **Secret access key** CSV file for later.
1. Review the summary and select **Create user**.

### Step 3. Configure the SSM Agent

AWS Systems Manager is required for automating tasks across your AWS resources. If your EC2 instances don't have the SSM Agent, follow the relevant instructions from Amazon:

- [Installing and Configuring SSM Agent on Windows Instances](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-win.html)
- [Installing and Configuring SSM Agent on Amazon EC2 Linux Instances](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-agent.html)

### Step 4. Complete Azure Arc prerequisites

1. Make sure the appropriate [Azure resources providers](../azure-arc/servers/prerequisites.md#azure-resource-providers) are registered:
    - Microsoft.HybridCompute
    - Microsoft.GuestConfiguration

1. Create a Service Principal for onboarding at scale. As an **Owner** on the subscription you want to use for the onboarding, create a service principal for Azure Arc onboarding as described in [Create a Service Principal for onboarding at scale](../azure-arc/servers/onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).

### Step 5. Connect AWS to Defender for Cloud

1. From Defender for Cloud's menu, open **Environment settings** and select the option to switch back to the classic connectors experience.

    :::image type="content" source="media/quickstart-onboard-gcp/classic-connectors-experience.png" alt-text="Switching back to the classic cloud connectors experience in Defender for Cloud.":::

1. Select **Add AWS account**.
    :::image type="content" source="./media/quickstart-onboard-aws/add-aws-account.png" alt-text="Add AWS account button on Defender for Cloud's multicloud connectors page":::
1. Configure the options in the **AWS authentication** tab:
    1. Enter a **Display name** for the connector.
    1. Confirm that the subscription is correct. It's the subscription that will include the connector and AWS Security Hub recommendations.
    1. Depending on the authentication option, you chose in [Step 2. Set up authentication for Defender for Cloud in AWS](#step-2-set-up-authentication-for-defender-for-cloud-in-aws):
        - Select  **Assume Role** and paste the ARN from [Create an IAM role for Defender for Cloud](#create-an-iam-role-for-defender-for-cloud).

            :::image type="content" source="./media/quickstart-onboard-aws/paste-arn-in-portal.png" alt-text="Pasting the ARN file in the relevant field of the AWS connection wizard in the Azure portal.":::

            OR

        - Select **Credentials** and paste the **access key** and **secret key** from the .csv file you saved in [Create an AWS user for Defender for Cloud](#create-an-aws-user-for-defender-for-cloud).
1. Select **Next**.
1. Configure the options in the **Azure Arc Configuration** tab:

    Defender for Cloud discovers the EC2 instances in the connected AWS account and uses SSM to onboard them to Azure Arc.

    > [!TIP]
    > For the list of supported operating systems, see [What operating systems for my EC2 instances are supported?](faq-general.yml) in the common questions.

    1. Select the **Resource Group** and **Azure Region** that the discovered AWS EC2s will be onboarded to in the selected subscription.
    1. Enter the **Service Principal ID** and **Service Principal Client Secret** for Azure Arc as described here [Create a Service Principal for onboarding at scale](../azure-arc/servers/onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale)
    1. If the machine is connecting to the internet via a proxy server, specify the proxy server IP address, or the name and port number that the machine uses to communicate with the proxy server. Enter the value in the format ```http://<proxyURL>:<proxyport>```
    1. Select **Review + create**.

        Review the summary information

        The Tags sections will list all Azure Tags that will be automatically created for each onboarded EC2 with its own relevant details to easily recognize it in Azure.

        Learn more about Azure Tags in [Use tags to organize your Azure resources and management hierarchy](../azure-resource-manager/management/tag-resources.md).

### Step 6. Confirmation

When the connector is successfully created, and AWS Security Hub has been configured properly:

- Defender for Cloud scans the environment for AWS EC2 instances, onboarding them to Azure Arc, enabling to install the Log Analytics agent and providing threat protection and security recommendations.
- The Defender for Cloud service scans for new AWS EC2 instances every 6 hours and onboards them according to the configuration.
- The AWS CIS standard will be shown in the Defender for Cloud's regulatory compliance dashboard.
- If Security Hub policy is enabled, recommendations will appear in the Defender for Cloud portal and the  regulatory compliance dashboard 5-10 minutes after onboard completes.

::: zone-end

:::image type="content" source="./media/quickstart-onboard-aws/aws-resources-in-recommendations.png" alt-text="AWS resources and recommendations in Defender for Cloud's recommendations page" lightbox="./media/quickstart-onboard-aws/aws-resources-in-recommendations.png":::

## Monitoring your AWS resources

As you can see in the previous screenshot, Defender for Cloud's security recommendations page displays your AWS resources. You can use the environments filter to enjoy Defender for Cloud's multicloud capabilities: view the recommendations for Azure, AWS, and GCP resources together.

To view all the active recommendations for your resources by resource type, use Defender for Cloud's asset inventory page and filter to the AWS resource type in which you're interested:

:::image type="content" source="./media/quickstart-onboard-aws/aws-resource-types-in-inventory.png" alt-text="screenshot of the asset inventory page's resource type filter showing the AWS options.":::

## Learn more

You can check out the following blogs:

- [Ignite 2021: Microsoft Defender for Cloud news](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/ignite-2021-microsoft-defender-for-cloud-news/ba-p/2882807).
- [Security posture management and server protection for AWS and GCP](https://techcommunity.microsoft.com/t5/microsoft-defender-for-cloud/security-posture-management-and-server-protection-for-aws-and/ba-p/3271388)

## Next steps

Connecting your AWS account is part of the multicloud experience available in Microsoft Defender for Cloud. For related information, see the following pages:

- [Security recommendations for AWS resources - a reference guide](recommendations-reference-aws.md).
- [Connect your GCP projects to Microsoft Defender for Cloud](quickstart-onboard-gcp.md)
- [Troubleshoot your multicloud connectors](troubleshooting-guide.md#troubleshooting-the-native-multicloud-connector)
- Check out [common questions](faq-general.yml) about onboarding your AWS account.
