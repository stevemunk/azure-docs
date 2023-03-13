---
title: include file
description: Send email.js sdk include file
author: natekimball-msft
manager: koagbakp
services: azure-communication-services
ms.author: natekimball
ms.date: 04/15/2022
ms.topic: include
ms.service: azure-communication-services
ms.custom: mode-other
---

Get started with Azure Communication Services by using the Communication Services JS Email client library to send Email messages.

## Understanding the email object model

The following classes and interfaces handle some of the major features of the Azure Communication Services Email Client library for JavaScript.

| Name | Description |
| ---- |-------------|
| EmailAddress | This class contains an email address and an option for a display name. |
| EmailAttachment | This class creates an email attachment by accepting a unique ID, email attachment mime type string, and binary data for content. |
| EmailClient | This class is needed for all email functionality. You instantiate it with your connection string and use it to send email messages. |
| EmailClientOptions | This class can be added to the EmailClient instantiation to target a specific API version. |
| EmailContent | This class contains the subject and the body of the email message. You have to specify at least one of PlainText or Html content. |
| EmailCustomHeader | This class allows for the addition of a name and value pair for a custom header. Email importance can also be specified through these headers using the header name 'x-priority' or 'x-msmail-priority'. |
| EmailMessage | This class combines the sender, content, and recipients. Custom headers, attachments, and reply-to email addresses can optionally be added, as well. |
| EmailRecipients | This class holds lists of EmailAddress objects for recipients of the email message, including optional lists for CC & BCC recipients. |
| EmailSendResult | This class holds the results of the email send operation. It has an operation ID, operation status and error object (when applicable). |
| EmailSendStatus | This class represents the set of statuses of an email send operation. |

EmailSendResult returns the following status on the email operation performed.

| Status Name | Description |
| ----------- | ------------|
| isStarted | Returns true if the email send operation is currently in progress and being processed. |
| isCompleted | Returns true if the email send operation has completed without error and the email is out for delivery. Any detailed status about the email delivery beyond this stage can be obtained either through Azure Monitor or through Azure Event Grid. [Learn how to subscribe to email events](../handle-email-events.md) |
| result | Property that exists if the email send operation has concluded. |
| error | Property that exists if the email send operation wasn't successful and encountered an error. The email wasn't sent. The result contains an error object with more details on the reason for failure or cancellation. |
| isCanceled | True if the email send operation was canceled before it could complete. The email wasn't sent. The result contains an error object with more details on the reason for failure or cancellation.|

## Prerequisites

- [Node.js (~14)](https://nodejs.org/download/release/v14.19.1/).
- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- An Azure Email Communication Services resource created and ready with a provisioned domain. [Get started with creating an Email Communication Resource](../create-email-communication-resource.md).
- An active Azure Communication Services resource connected to an Email Domain and its connection string. [Get started by connecting an Email Communication Resource with a Azure Communication Resource](../../create-communication-resource.md).

Completing this quick start incurs a small cost of a few USD cents or less in your Azure account.

> [!Note]
> We can also send an email from our own verified domain. [Add custom verified domains to Email Communication Service](../add-azure-managed-domains.md).

### Prerequisite check

- In a terminal or command window, run `node --version` to check that Node.js is installed.
- To view the domains verified with your Email Communication Services resource, sign in to the [Azure portal](https://portal.azure.com/), locate your Email Communication Services resource and open the **Provision domains** tab from the left navigation pane.

## Set up the application environment

### Create a new Node.js Application
First, open your terminal or command window, create a new directory for your app, and navigate to it.

```console
mkdir email-quickstart && cd email-quickstart
```

Run `npm init -y` to create a **package.json** file with default settings.

```console
npm init -y
```

Use a text editor to create a file called **send-email.js** in the project root directory. Change the "main" property in **package.json** to "send-email.js". The following section demonstrates how to add the source code for this quickstart to the newly created file.

### Install the package
Use the `npm install` command to install the Azure Communication Services Email client library for JavaScript.

```console
npm install @azure/communication-email --save
```

The `--save` option lists the library as a dependency in your **package.json** file.

## Creating the email client with authentication

### Option 1: Authenticate using a connection string

Import the **EmailClient** from the client library and instantiate it with your connection string.

The following code retrieves the connection string for the resource from an environment variable named `COMMUNICATION_SERVICES_CONNECTION_STRING` using the dotenv package. Use the `npm install` command to install the dotenv package. Learn how to [manage your resource's connection string](../../create-communication-resource.md#store-your-connection-string).

```console
npm install dotenv
```

Add the following code to **send-email.js**:

```javascript
const { EmailClient } = require("@azure/communication-email");
require("dotenv").config();

// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];
const emailClient = new EmailClient(connectionString);
```

### Option 2: Authenticate using Azure Active Directory

You can also authenticate with Azure Active Directory using the [Azure Identity library](https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/identity/identity). To use the [DefaultAzureCredential](https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/identity/identity#defaultazurecredential) provider shown below, or other credential providers provided with the Azure SDK, please install the [`@azure/identity`](https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/identity/identity) package:

```bash
npm install @azure/identity
```

The [`@azure/identity`](https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/identity/identity) package provides a variety of credential types that your application can use to do this. The README for `@azure/identity` provides more details and samples to get you started.
`AZURE_CLIENT_SECRET`, `AZURE_CLIENT_ID` and `AZURE_TENANT_ID` environment variables are needed to create a `DefaultAzureCredential` object.

```typescript
import { DefaultAzureCredential } from "@azure/identity";
import { EmailClient } from "@azure/communication-email";
const endpoint = "https://<resource-name>.communication.azure.com";
let credential = new DefaultAzureCredential();
const client = new EmailClient(endpoint, credential);
```

For simplicity, this quickstart uses connection strings, but in production environments, we recommend using [service principals](../../../quickstarts/identity/service-principal.md).

## Basic email sending 

### Send an email message

To send an email message, call the `beginSend` function from the EmailClient. This method returns a poller that checks on the status of the operation and retrieves the result once it's finished.

```javascript

async function main() {
  try {
    const message = {
      senderAddress: "<donotreply@xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.azurecomm.net>",
      content: {
        subject: "Welcome to Azure Communication Services Email",
        plainText: "This email message is sent from Azure Communication Services Email using the JavaScript SDK.",
      },
      recipients: {
        to: [
          {
            address: "<emailalias@emaildomain.com>",
            displayName: "Customer Name",
          },
        ],
      },
    };

    const poller = await emailClient.beginSend(message);
    const response = await poller.pollUntilDone();
  } catch (e) {
    console.log(e);
  }
}

main();
```

Make these replacements in the code:

- Replace `<emailalias@emaildomain.com>` with the email address you would like to send a message to.
- Replace `<donotreply@xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.azurecomm.net>` with the MailFrom address of your verified domain.

### Run the code

use the node command to run the code you added to the send-email.js file.

```console
node ./send-email.js
```
### Sample code

You can download the sample app from [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/send-email)

## Advanced sending

### Send an email message to multiple recipients

To send an email message to multiple recipients, add an object for each recipient type and an object for each recipient. These addresses can be added as `To`, `CC`, or `BCC` recipients.

```javascript
const message = {
  senderAddress: "<donotreply@xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.azurecomm.net>",
  content: {
    subject: "Welcome to Azure Communication Service Email.",
    plainText: "<This email message is sent from Azure Communication Service Email using JS SDK.>"
  },
  recipients: {
    to: [
      {
        address: "customer1@domain.com",
        displayName: "Customer Name 1",
      },
      {
        address: "customer2@domain.com",
        displayName: "Customer Name 2",
      }
    ],
    cc: [
      {
        address: "ccCustomer1@domain.com",
        displayName: " CC Customer 1",
      },
      {
        address: "ccCustomer2@domain.com",
        displayName: "CC Customer 2",
      }
    ],
    bcc: [
      {
        address: "bccCustomer1@domain.com",
        displayName: " BCC Customer 1",
      },
      {
        address: "bccCustomer2@domain.com",
        displayName: "BCC Customer 2",
      }
    ]
  }
};

```

You can download the sample app demonstrating this action from [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/send-email-advanced/send-email-multiple-recipients)


### Send an email message with attachments

We can add an attachment by defining an attachment object and adding it to our message. Read the attachment file and encode it using Base64.

```javascript
const filePath = "<path-to-your-file>";

const message = {
  sender: "<donotreply@xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.azurecomm.net>",
  content: {
    subject: "Welcome to Azure Communication Service Email.",
    plainText: "<This email message is sent from Azure Communication Service Email using JavaScript SDK.>"
  },
  recipients: {
    to: [
      {
        address: "<emailalias@emaildomain.com>",
        displayName: "Customer Name",
      }
    ]
  },
  attachments: [
    {
      name: path.basename(filePath),
      contentType: "text/plain",
      contentInBase64: readFileSync(filePath, "base64"),
    }
  ]
};

const response = await emailClient.send(message);
```

You can download the sample app demonstrating this action from [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/send-email-advanced/send-email-attachments)
