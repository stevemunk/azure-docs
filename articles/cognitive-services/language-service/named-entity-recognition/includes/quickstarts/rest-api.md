---
services: cognitive-services
author: jboback
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-service
ms.topic: include
ms.date: 10/31/2022
ms.author: jboback
ms.custom: ignite-fall-2021
---

[Reference documentation](/rest/api/language/2022-05-01/text-analysis-runtime/analyze-text)

Use this quickstart to send Named Entity Recognition (NER) requests using the REST API. In the following example, you will use cURL to identify [recognized entities](../../concepts/named-entity-categories.md) in text.

[!INCLUDE [Use Language Studio](../../../includes/use-language-studio.md)]

## Prerequisites

* The current version of [cURL](https://curl.haxx.se/).
* Once you have your Azure subscription, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="Create a Language resource"  target="_blank">create a Language resource </a> in the Azure portal to get your key and endpoint. After it deploys, select **Go to resource**.
    * You will need the key and endpoint from the resource you create to connect your application to the API. You'll paste your key and endpoint into the code below later in the quickstart.
    * You can use the free pricing tier (`Free F0`) to try the service, and upgrade later to a paid tier for production.

> [!NOTE]
> * The following BASH examples use the `\` line continuation character. If your console or terminal uses a different line continuation character, use that character.
> * You can find language specific samples on [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code).
> * Go to the Azure portal and find the key and endpoint for the Language resource you created in the prerequisites. They will be located on the resource's **key and endpoint** page, under **resource management**. Then replace the strings in the code below with your key and endpoint.
To call the API, you need the following information:


|parameter  |Description  |
|---------|---------|
|`-X POST <endpoint>`     | Specifies your endpoint for accessing the API.        |
|`-H Content-Type: application/json`     | The content type for sending JSON data.          |
|`-H "Ocp-Apim-Subscription-Key:<key>`    | Specifies the key for accessing the API.        |
|`-d <documents>`     | The JSON containing the documents you want to send.         |

The following cURL commands are executed from a BASH shell. Edit these commands with your own resource name, resource key, and JSON values.

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=REST API&Pillar=Language&Product=Named-entity-recognition&Page=quickstart&Section=Prerequisites" target="_target">I ran into an issue</a>



## Named Entity Extraction (NER)

[!INCLUDE [REST API quickstart instructions](../../../includes/rest-api-instructions.md)]

```bash
curl -i -X POST https://<your-language-resource-endpoint>/language/:analyze-text?api-version=2022-05-01 \
-H "Content-Type: application/json" \
-H "Ocp-Apim-Subscription-Key:<your-language-resource-key>" \
-d \
'
{
    "kind": "EntityRecognition",
    "parameters": {
        "modelVersion": "latest"
    },
    "analysisInput":{
        "documents":[
            {
                "id":"1",
                "language": "en",
                "text": "I had a wonderful trip to Seattle last week."
            }
        ]
    }
}
'
```

### JSON response

```json
{
	"kind": "EntityRecognitionResults",
	"results": {
		"documents": [{
			"id": "1",
			"entities": [{
				"text": "trip",
				"category": "Event",
				"offset": 18,
				"length": 4,
				"confidenceScore": 0.74
			}, {
				"text": "Seattle",
				"category": "Location",
				"subcategory": "GPE",
				"offset": 26,
				"length": 7,
				"confidenceScore": 1.0
			}, {
				"text": "last week",
				"category": "DateTime",
				"subcategory": "DateRange",
				"offset": 34,
				"length": 9,
				"confidenceScore": 0.8
			}],
			"warnings": []
		}],
		"errors": [],
		"modelVersion": "2021-06-01"
	}
}
```

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=REST API&Pillar=Language&Product=Named-entity-recognition&Page=quickstart&Section=Named-entity-recognition" target="_target">I ran into an issue</a>
