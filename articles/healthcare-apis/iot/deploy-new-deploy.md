---
title: Manual deployment and post-deployment of the MedTech service using the Azure portal - Azure Health Data Services
description: In this article, you'll learn how to manually create a deployment and post-deployment of the MedTech service in the Azure portal.
author: msjasteppe
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 03/10/2023
ms.author: jasteppe
---

# Quickstart: Part 3: Manual deployment and post-deployment of the MedTech service

> [!NOTE]
> [Fast Healthcare Interoperability Resources (FHIR&#174;)](https://www.hl7.org/fhir/) is an open healthcare specification.

When you're satisfied with your configuration and it has been successfully validated, you can complete the deployment and post-deployment process.

## Create your manual deployment

1. Select the **Create** button to begin the deployment.

2. The deployment process may take several minutes. The screen will display a message saying that your deployment is in progress.

3. When Azure has finished deploying, a message will appear will say, "Your Deployment is complete" and will also display the following information:

- Deployment name
- Subscription
- Resource group
- Deployment details

Your screen should look something like this:

   :::image type="content" source="media\deploy-new-deploy\created-medtech-service.png" alt-text="Screenshot of the MedTech service deployment completion." lightbox="media\deploy-new-deploy\created-medtech-service.png":::

## Manual post-deployment requirements

There are two post-deployment steps you must perform or the MedTech service can't:

1. Read device data from the device message event hub. 
2. Read or write to the FHIR service. 

These steps are:

1. Grant access to the device message event hub.
2. Grant access to the FHIR service.

These two other steps are needed because MedTech service uses [Azure role-based access control (Azure RBAC)](../../role-based-access-control/overview.md) and a [system-assigned managed identity](../../active-directory/managed-identities-azure-resources/overview.md) for extra security and control of your MedTech service assets.

### Grant access to the device message event hub

Follow these steps to grant access to the device message event hub:

1. In the **Search** bar at the top center of the Azure portal, enter and select the name of your **Event Hubs Namespace** that was previously created for your MedTech service device messages.

2. Select the **Event Hubs** button under **Entities**.

3. Select the event hub that will be used for your MedTech service device messages. For this example, the device message event hub is named **devicedata**.

4. Select the **Access control (IAM)** button.

5. Select the **Add role assignment** button.

6. On the **Add role assignment** page, select the **View** button directly across from the **Azure Event Hubs Data Receiver** role. The Azure Event Hubs Data Receiver role allows the MedTech service to receive device message data from this event hub. For more information about application roles, see [Authentication & Authorization for Azure Health Data Services](.././authentication-authorization.md).

7. Select the **Select role** button.

8. Select the **Next** button.

9. In the **Add role assignment** page, select **Managed identity** next to **Assign access to** and **+ Select members** next to **Members**. 

10. When the **Select managed identities** box opens, under the **Managed identity** box, select **MedTech service,** and find your MedTech service system-assigned managed identity under the **Select** box. Once the system-assigned managed identity for your MedTech service is found, select it, and then select the **Select** button.

    The system-assigned managed identify name for your MedTech service is a concatenation of the workspace name and the name of your MedTech service, using the format: **"your workspace name"/"your MedTech service name"** or **"your workspace name"/iotconnectors/"your MedTech service name"**. For example: **azuredocsdemo/mt-azuredocsdemo** or **azuredocsdemo/iotconnectors/mt-azuredocsdemo**.

11. On the **Add role assignment** page, select the **Review + assign** button.

12. On the **Add role assignment** confirmation page, select the **Review + assign** button.

13. After the role assignment has been successfully added to the event hub, a notification will display on your screen with a green check mark. This notification indicates that your MedTech service can now read from your device message event hub. It should look like this:

    :::image type="content" source="media\deploy-new-deploy\validate-medtech-service-managed-identity-added-to-event-hub.png" alt-text="Screenshot of the MedTech service system-assigned managed identity being successfully granted access to the event hub with a red box around the message." lightbox="media\deploy-new-deploy\validate-medtech-service-managed-identity-added-to-event-hub.png":::

For more information about authorizing access to Event Hubs resources, see [Authorize access with Azure Active Directory](../../event-hubs/authorize-access-azure-active-directory.md).  

### Grant access to the FHIR service

The process for granting your MedTech service system-assigned managed identity access to your **FHIR service** requires the same 13 steps that you used to grant access to your device message event hub. There are two exceptions. The first is that, instead of navigating to the **Access Control (IAM)** menu from within your event hub (as outlined in steps 1-4), you should navigate to the equivalent **Access Control (IAM)** menu from within your **FHIR service**. The second exception is that, in step 6, your MedTech service system-assigned managed identity will require you to select the **View** button directly across from **FHIR Data Writer** access instead of the button across from **Azure Event Hubs Data Receiver**.

The **FHIR Data Writer** role provides read and write access to your FHIR service, which your MedTech service uses to access or persist data. Because the MedTech service is deployed as a separate resource, the FHIR service will receive requests from the MedTech service. If the FHIR service doesn’t know who's making the request, it will deny the request as unauthorized.

For more information about assigning roles to the FHIR service, see [Configure Azure Role-based Access Control (RBAC)](.././configure-azure-rbac.md).

For more information about application roles, see [Authentication & Authorization for Azure Health Data Services](.././authentication-authorization.md).

Now that you have granted access to the device message event hub and the FHIR service, your manual deployment is complete. You're MedTech service is now ready to receive data from a device and process it into a FHIR Observation resource.

## Next steps

In this article, you learned how to perform the manual deployment and post-deployment steps to implement your MedTech service. 

To learn about other methods for deploying the MedTech service, see

> [!div class="nextstepaction"]
> [Choose a deployment method for the MedTech service](deploy-new-choose.md)

FHIR&#174; is a registered trademark of Health Level Seven International, registered in the U.S. Trademark Office and is used with their permission.
