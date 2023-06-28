---
title: Custom template document model - Form Recognizer
titleSuffix: Azure Applied AI Services
description: Use the custom template document model to train a model to extract data from structured or templated forms.
author: laujan
manager: nitinme
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 05/23/2023
ms.author: lajanuar
---

# Custom template document model

::: moniker range="form-recog-3.0.0"
[!INCLUDE [applies to v3.0](includes/applies-to-v3-0.md)]
::: moniker-end

::: moniker range="form-recog-2.1.0"
[!INCLUDE [applies to v2.1](includes/applies-to-v2-1.md)]
::: moniker-end

Custom template (formerly custom form) is an easy-to-train document model that accurately extracts labeled key-value pairs, selection marks, tables, regions, and signatures from documents. Template models use layout cues to extract values from documents and are suitable to extract fields from highly structured documents with defined visual templates.

::: moniker range="form-recog-3.0.0"

Custom template models share the same labeling format and strategy as custom neural models, with support for more field types and languages.

::: moniker-end

## Model capabilities

Custom template models support key-value pairs, selection marks, tables, signature fields, and selected regions.

| Form fields | Selection marks | Tabular fields (Tables) | Signature | Selected regions |
|:--:|:--:|:--:|:--:|:--:|
| Supported| Supported | Supported | Supported| Supported |

::: moniker range="form-recog-3.0.0"

## Tabular fields

With the release of API versions **2022-06-30-preview** and  later, custom template models will add support for **cross page** tabular fields (tables):  

* To label a table that spans multiple pages, label each row of the table across the different pages in a single table.
* As a best practice, ensure that your dataset contains a few samples of the expected variations. For example, include samples where the entire table is on a single page and where tables span two or more pages if you expect to see those variations in documents.

Tabular fields are also useful when extracting repeating information within a document that isn't recognized as a table. For example, a repeating section of work experiences in a resume can be labeled and extracted as a tabular field.

::: moniker-end

## Dealing with variations

Template models rely on a defined visual template, changes to the template results in lower accuracy. In those instances, split your training dataset to include at least five samples of each template and train a model for each of the variations. You can then [compose](concept-composed-models.md) the models into a single endpoint. For subtle variations, like digital PDF documents and images, it's best to include at least five examples of each type in the same training dataset.

## Training a model

::: moniker range="form-recog-3.0.0"

Custom template models are generally available with the [v3.0 API](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-2022-08-31/operations/BuildDocumentModel). If you're starting with a new project or have an existing labeled dataset, use the v3 API with Form Recognizer Studio to train a custom template model.

| Model | REST API | SDK | Label and Test Models|
|--|--|--|--|
| Custom template  | [Form Recognizer 3.0 ](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-2022-08-31/operations/AnalyzeDocument)| [Form Recognizer SDK](quickstarts/get-started-sdks-rest-api.md?view=form-recog-3.0.0&preserve-view=true)| [Form Recognizer Studio](https://formrecognizer.appliedai.azure.com/studio)|

With the v3.0 API, the build operation to train model supports a new ```buildMode``` property, to train a custom template model, set the ```buildMode``` to ```template```.

```REST
https://{endpoint}/formrecognizer/documentModels:build?api-version=2022-08-31

{
  "modelId": "string",
  "description": "string",
  "buildMode": "template",
  "azureBlobSource":
  {
    "containerUrl": "string",
    "prefix": "string"
  }
}
```

::: moniker-end

::: moniker range="form-recog-2.1.0"

Custom (template) models  are generally available with the [v2.1 API](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeWithCustomForm).

| Model | REST API | SDK | Label and Test Models|
|--|--|--|--|
| Custom model (template) | [Form Recognizer 2.1 ](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeWithCustomForm)| [Form Recognizer SDK](quickstarts/get-started-v2-1-sdk-rest-api.md?pivots=programming-language-python)| [Form Recognizer Sample labeling tool](https://fott-2-1.azurewebsites.net/)|

::: moniker-end

## Next steps

Learn to create and compose custom models:

::: moniker range="form-recog-3.0.0"

> [!div class="nextstepaction"]
> [**Build a custom model**](how-to-guides/build-a-custom-model.md)
> [**Compose custom models**](how-to-guides/compose-custom-models.md)

::: moniker-end

::: moniker range="form-recog-2.1.0"

> [!div class="nextstepaction"]
> [**Build a custom model**](concept-custom.md#build-a-custom-model)
> [**Compose custom models**](concept-composed-models.md#development-options)

::: moniker-end
