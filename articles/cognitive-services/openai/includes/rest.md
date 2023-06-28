---
title: 'Quickstart: Use the OpenAI Service to make your first completions call with the REST API'
titleSuffix: Azure OpenAI Service
description: Walkthrough on how to get started with Azure OpenAI and make your first completions call with the REST API. 
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: openai
ms.topic: include
ms.date: 02/02/2023
keywords: 
---

## Prerequisites

- An Azure subscription - <a href="https://azure.microsoft.com/free/cognitive-services" target="_blank">Create one for free</a>
- Access granted to Azure OpenAI in the desired Azure subscription

    Currently, access to this service is granted only by application. You can apply for access to the Azure OpenAI service by completing the form at <a href="https://aka.ms/oai/access" target="_blank">https://aka.ms/oai/access</a>. Open an issue on this repo to contact us if you have an issue.
- <a href="https://www.python.org/" target="_blank">Python 3.7.1 or later version</a>
- The following Python libraries: os, requests, json
- An Azure OpenAI resource with a model deployed. For more information about model deployment, see the [resource deployment guide](../how-to/create-resource.md).

## Set up

### Retrieve key and endpoint

To successfully make a call against Azure OpenAI, you'll need the following:

|Variable name | Value |
|--------------------------|-------------|
| `ENDPOINT`               | This value can be found in the **Keys & Endpoint** section when examining your resource from the Azure portal. Alternatively, you can find the value in **Azure OpenAI Studio** > **Playground** > **Code View**. An example endpoint is: `https://docs-test-001.openai.azure.com/`.|
| `API-KEY` | This value can be found in the **Keys & Endpoint** section when examining your resource from the Azure portal. You can use either `KEY1` or `KEY2`.|
| `DEPLOYMENT-NAME` | This value will correspond to the custom name you chose for your deployment when you deployed a model. This value can be found under **Resource Management** > **Deployments** in the Azure portal or alternatively under **Management** > **Deployments** in Azure OpenAI Studio.|

Go to your resource in the Azure portal. The **Endpoint and Keys** can be found in the **Resource Management** section. Copy your endpoint and access key as you'll need both for authenticating your API calls. You can use either `KEY1` or `KEY2`. Always having two keys allows you to securely rotate and regenerate keys without causing a service disruption.

:::image type="content" source="../media/quickstarts/endpoint.png" alt-text="Screenshot of the overview blade for an OpenAI Resource in the Azure portal with the endpoint & access keys location circled in red." lightbox="../media/quickstarts/endpoint.png":::

Create and assign persistent environment variables for your key and endpoint.

### Environment variables

# [Command Line](#tab/command-line)

```CMD
setx AZURE_OPENAI_KEY "REPLACE_WITH_YOUR_KEY_VALUE_HERE" 
```

```CMD
setx AZURE_OPENAI_ENDPOINT "REPLACE_WITH_YOUR_ENDPOINT_HERE" 
```

# [PowerShell](#tab/powershell)

```powershell
[System.Environment]::SetEnvironmentVariable('AZURE_OPENAI_KEY', 'REPLACE_WITH_YOUR_KEY_VALUE_HERE', 'User')
```

```powershell
[System.Environment]::SetEnvironmentVariable('AZURE_OPENAI_ENDPOINT', 'REPLACE_WITH_YOUR_ENDPOINT_HERE', 'User')
```

# [Bash](#tab/bash)

```Bash
echo export AZURE_OPENAI_KEY="REPLACE_WITH_YOUR_KEY_VALUE_HERE" >> /etc/environment && source /etc/environment
```

```Bash
echo export AZURE_OPENAI_ENDPOINT="REPLACE_WITH_YOUR_ENDPOINT_HERE" >> /etc/environment && source /etc/environment
```
---

> [!div class="nextstepaction"]
> [I ran into an issue with the setup.](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=REST&Pillar=AOAI&Product=gpt&Page=quickstart&Section=Set-up)

## Create a new Python application

Create a new Python file called quickstart.py. Then open it up in your preferred editor or IDE.

1. Replace the contents of quickstart.py with the following code.

    ```python
    import os
    import requests
    import json

    api_key = os.getenv("AZURE_OPENAI_KEY")
    base_url = os.getenv("AZURE_OPENAI_ENDPOINT") 
    deployment_name ="REPLACE_WITH_YOUR_DEPLOYMENT_NAME_HERE"

    url = base_url + "/openai/deployments/" + deployment_name + "/completions?api-version=2023-05-15"
    prompt = "Once upon a time"
    payload = {        
        "prompt":prompt
        }

    r = requests.post(url, 
          headers={
            "api-key": api_key,
            "Content-Type": "application/json"
          },
          json = payload
        )

    response = json.loads(r.text)
    formatted_response = json.dumps(response, indent=4)

    print(formatted_response)
    ```

    > [!IMPORTANT]
    > For production, use a secure way of storing and accessing your credentials like [Azure Key Vault](../../../key-vault/general/overview.md). For more information about credential security, see the Cognitive Services [security](../../security-features.md) article.

1. Run the application with the `python` command on your quickstart file:

    ```console
    python quickstart.py
    ```

## Output

The output from the completions API will look as follows.

```json
{
    "id": "ID of your call",
    "object": "text_completion",
    "created": 1675444965,
    "model": "text-davinci-002",
    "choices": [
        {
            "text": " there lived in a little village a woman who was known as the meanest",
            "index": 0,
            "finish_reason": "length",
            "logprobs": null
        }
    ],
    "usage": {
        "completion_tokens": 16,
        "prompt_tokens": 3,
        "total_tokens": 19
    }
}
```

> [!div class="nextstepaction"]
> [I ran into an issue when running the code sample.](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=REST&Pillar=AOAI&Product=gpt&Page=quickstart&Section=Create-sample-code)

The Azure OpenAI Service also performs content moderation on the prompt inputs and generated outputs. The prompts or responses may be filtered if harmful content is detected. For more information, see the [content filter](../concepts/content-filter.md) article.

## Clean up resources

If you want to clean up and remove an OpenAI resource, you can delete the resource or resource group. Deleting the resource group also deletes any other resources associated with it.

- [Portal](../../cognitive-services-apis-create-account.md#clean-up-resources)
- [Azure CLI](../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## Next steps

* Learn more about how to generate the best completion in our [How-to guide on completions](../how-to/completions.md).
* For more examples check out the [Azure OpenAI Samples GitHub repository](https://aka.ms/AOAICodeSamples).