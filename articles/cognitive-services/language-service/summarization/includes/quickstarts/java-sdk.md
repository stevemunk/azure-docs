---
author: jboback
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-service
ms.topic: include
ms.date: 12/12/2022
ms.custom: devx-track-java, ignite-fall-2021
ms.author: aahi
---

[Reference documentation](/java/api/overview/azure/ai-textanalytics-readme?preserve-view=true&view=azure-java-preview) | [Additional samples](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/textanalytics/azure-ai-textanalytics/src/samples) | [Package (Maven)](https://mvnrepository.com/artifact/com.azure/azure-ai-textanalytics/5.2.0-beta.3) | [Library source code](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/textanalytics/azure-ai-textanalytics)

Use this quickstart to create a text summarization application with the client library for Java. In the following example, you will create a Java application that can summarize documents.

[!INCLUDE [Use Language Studio](../use-language-studio.md)]

## Prerequisites

* Azure subscription - [Create one for free](https://azure.microsoft.com/free/cognitive-services)
* [Java Development Kit](https://www.oracle.com/technetwork/java/javase/downloads/index.html) (JDK) with version 8 or above
* Once you have your Azure subscription, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="Create a Language resource"  target="_blank">create a Language resource </a> in the Azure portal to get your key and endpoint.  After it deploys, select **Go to resource**.
    * You will need the key and endpoint from the resource you create to connect your application to the API. You'll paste your key and endpoint into the code below later in the quickstart.
    * You can use the free pricing tier (`Free F0`) to try the service, and upgrade later to a paid tier for production.
* To use the Analyze feature, you will need a Language resource with the standard (S) pricing tier.

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=REST API&Pillar=Language&Product=Summarization&Page=quickstart&Section=Clean-up-resources" target="_target" target="_target">I ran into an issue</a>

## Setting up

### Add the client library

Create a Maven project in your preferred IDE or development environment. Then add the following dependency to your project's *pom.xml* file. You can find the implementation syntax [for other build tools](https://mvnrepository.com/artifact/com.azure/azure-ai-textanalytics/5.2.0-beta.3) online.

```xml
<dependencies>
     <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-ai-textanalytics</artifactId>
        <version>5.2.0-beta.3</version>
    </dependency>
</dependencies>
```

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=CSHARP&Pillar=Language&Product=Summarization&Page=quickstart&Section=Set-up-the-environment" target="_target">I ran into an issue</a>

## Code example

Create a Java file named `Example.java`. Open the file and copy the below code. Remember to replace the `key` variable with the key for your resource, and replace the `endpoint` variable with the endpoint for your resource. Then run the code.  

[!INCLUDE [find the key and endpoint for a resource](../../../includes/find-azure-resource-info.md)]

```java
import com.azure.core.credential.AzureKeyCredential;
import com.azure.ai.textanalytics.models.*;
import com.azure.ai.textanalytics.TextAnalyticsClientBuilder;
import com.azure.ai.textanalytics.TextAnalyticsClient;
import java.util.ArrayList;
import java.util.List;
import com.azure.core.util.polling.SyncPoller;
import com.azure.ai.textanalytics.util.*;

public class Example {

    private static String KEY = "replace-with-your-key-here";
    private static String ENDPOINT = "replace-with-your-endpoint-here";

    public static void main(String[] args) {
        TextAnalyticsClient client = authenticateClient(KEY, ENDPOINT);
        summarizationExample(client);
    }
    // Method to authenticate the client object with your key and endpoint
    static TextAnalyticsClient authenticateClient(String key, String endpoint) {
        return new TextAnalyticsClientBuilder()
                .credential(new AzureKeyCredential(key))
                .endpoint(endpoint)
                .buildClient();
    }
    // Example method for summarizing text
    static void summarizationExample(TextAnalyticsClient client) {
        List<String> documents = new ArrayList<>();
        documents.add(
                "The extractive summarization feature uses natural language processing techniques "
                + "to locate key sentences in an unstructured text document. "
                + "These sentences collectively convey the main idea of the document. This feature is provided as an API for developers. "
                + "They can use it to build intelligent solutions based on the relevant information extracted to support various use cases. "
                + "In the public preview, extractive summarization supports several languages. "
                + "It is based on pretrained multilingual transformer models, part of our quest for holistic representations. "
                + "It draws its strength from transfer learning across monolingual and harness the shared nature of languages "
                + "to produce models of improved quality and efficiency.");
    
        SyncPoller<AnalyzeActionsOperationDetail, AnalyzeActionsResultPagedIterable> syncPoller =
                client.beginAnalyzeActions(documents,
                        new TextAnalyticsActions().setDisplayName("{tasks_display_name}")
                                .setExtractSummaryActions(
                                        new ExtractSummaryAction()),
                        "en",
                        new AnalyzeActionsOptions());
    
        syncPoller.waitForCompletion();
    
        syncPoller.getFinalResult().forEach(actionsResult -> {
            System.out.println("Extractive Summarization action results:");
            for (ExtractSummaryActionResult actionResult : actionsResult.getExtractSummaryResults()) {
                if (!actionResult.isError()) {
                    for (ExtractSummaryResult documentResult : actionResult.getDocumentsResults()) {
                        if (!documentResult.isError()) {
                            System.out.println("\tExtracted summary sentences:");
                            for (SummarySentence summarySentence : documentResult.getSentences()) {
                                System.out.printf(
                                        "\t\t Sentence text: %s, length: %d, offset: %d, rank score: %f.%n",
                                        summarySentence.getText(), summarySentence.getLength(),
                                        summarySentence.getOffset(), summarySentence.getRankScore());
                            }
                        } else {
                            System.out.printf("\tCannot extract summary sentences. Error: %s%n",
                                    documentResult.getError().getMessage());
                        }
                    }
                } else {
                    System.out.printf("\tCannot execute Extractive Summarization action. Error: %s%n",
                            actionResult.getError().getMessage());
                }
            }
        });
    }
}

```

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=CSHARP&Pillar=Language&Product=Summarization&Page=quickstart&Section=Code-example" target="_target">I ran into an issue</a>

### Output

```console
Extractive Summarization action results:
	Extracted summary sentences:
		 Sentence text: The extractive summarization feature uses natural language processing techniques to locate key sentences in an unstructured text document., length: 138, offset: 0, rank score: 1.000000.
		 Sentence text: This feature is provided as an API for developers., length: 50, offset: 206, rank score: 0.510000.
		 Sentence text: In the public preview, extractive summarization supports several languages., length: 75, offset: 378, rank score: 0.410000.
```
