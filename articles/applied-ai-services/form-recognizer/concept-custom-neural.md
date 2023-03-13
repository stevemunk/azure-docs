---
title: Custom neural document model - Form Recognizer
titleSuffix: Azure Applied AI Services
description: Use the custom neural document model to train a model to extract data from structured, semistructured, and unstructured documents.
author: laujan
manager: nitinme
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 03/03/2023
ms.author: lajanuar
ms.custom: references_regions
monikerRange: 'form-recog-3.0.0'
recommendations: false
---

# Custom neural document model

**This article applies to:** ![Form Recognizer v3.0 checkmark](media/yes-icon.png) **Form Recognizer v3.0**.

Custom neural document models or neural models are a deep learned model type that combines layout and language features to accurately extract labeled fields from documents. The base custom neural model is trained on various document types that makes it suitable to be trained for extracting fields from structured, semi-structured and unstructured documents. The table below lists common document types for each category:

|Documents | Examples |
|---|--|
|structured| surveys, questionnaires|
|semi-structured | invoices, purchase orders |
|unstructured | contracts, letters|

Custom neural models share the same labeling format and strategy as [custom template](concept-custom-template.md) models. Currently custom neural models only support a subset of the field types supported by custom template models.

## Model capabilities

Custom neural models currently only support key-value pairs and selection marks and structured fields (tables), future releases include support for signatures.

| Form fields | Selection marks | Tabular fields | Signature | Region |
|:--:|:--:|:--:|:--:|:--:|
| Supported | Supported | Supported | Unsupported | Supported <sup>1</sup> |

<sup>1</sup> Region labels in custom neural models use the results from the Layout API for specified region. This feature is different from template models where, if no value is present, text is generated at training time.

### Build mode

The build custom model operation has added support for the *template* and *neural* custom models. Previous versions of the REST API and SDKs only supported a single build mode that is now known as the *template* mode.

Neural models support documents that have the same information, but different page structures. Examples of these documents include United States W2 forms, which share the same information, but may vary in appearance across companies. For more information, *see* [Custom model build mode](concept-custom.md#build-mode).

## Language support

1. Neural models now support added languages in the ```2023-02-28-preview``` API.

| Languages | API version |
|:--:|:--:|
| English | `2022-08-31` (GA), `2023-02-28-preview`|
| German |  `2023-02-28-preview`|
| Italian |  `2023-02-28-preview`|
| French |  `2023-02-28-preview`|
| Spanish |  `2023-02-28-preview`|
| Dutch |  `2023-02-28-preview`|

## Tabular fields

With the release of API versions **2022-06-30-preview** and  later, custom neural models will support tabular fields (tables):

* Models trained with API version 2022-08-31,  or later will accept tabular field labels.
* Documents analyzed with custom neural models using API version 2022-06-30-preview or later will produce tabular fields aggregated across the tables.
* The results can be found in the ```analyzeResult``` object's ```documents``` array that is returned following an analysis operation.

Tabular fields support **cross page tables** by default:

* To label a table that spans multiple pages, label each row of the table across the different pages in a single table.
* As a best practice, ensure that your dataset contains a few samples of the expected variations. For example, include samples where the entire table is on a single page and where tables span two or more pages.

Tabular fields are also useful when extracting repeating information within a document that isn't recognized as a table. For example, a repeating section of work experiences in a resume can be labeled and extracted as a tabular field.

## Supported regions

As of October 18, 2022, Form Recognizer custom neural model training will only be available in the following Azure regions until further notice:

* Australia East
* Brazil South
* Canada Central
* Central India
* Central US
* East Asia
* East US
* East US2
* France Central
* Japan East
* South Central US
* Southeast Asia
* UK South
* West Europe
* West US2
* US Gov Arizona
* US Gov Virginia



> [!TIP]
> You can [copy a model](disaster-recovery.md#copy-api-overview) trained in one of the select regions listed to **any other region** and use it accordingly.
>
> Use the [**REST API**](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-2022-08-31/operations/CopyDocumentModelTo) or [**Form Recognizer Studio**](https://formrecognizer.appliedai.azure.com/studio/custommodel/projects) to copy a model to another region.

## Best practices

Custom neural models differ from custom template models in a few different ways. The custom template or model relies on a consistent visual template to extract the labeled data. Custom neural models support structured, semi-structured, and unstructured documents to extract fields. When you're choosing between the two model types, start with a neural model, and test to determine if it supports your functional needs.

### Dealing with variations

Custom neural models can generalize across different formats of a single document type. As a best practice, create a single model for all variations of a document type. Add at least five labeled samples for each of the different variations to the training dataset.

### Field naming

When you label the data, labeling the field relevant to the value improves the accuracy of the key-value pairs extracted. For example, for a field value containing the supplier ID, consider naming the field "supplier_id". Field names should be in the language of the document.

### Labeling contiguous values

Value tokens/words of one field must be either

* Consecutive sequence in natural reading order without interleaving with other fields
* In a region that don't cover any other fields

### Representative data

Values in training cases should be diverse and representative. For example, if a field is named "date", values for this field should be a date. synthetic value like a random string can affect model performance.

## Current Limitations

* The model doesn't recognize values split across page boundaries.
* Custom neural models are only trained in English. Model performance is lower for documents in other languages.
* If a dataset labeled for custom template models is used to train a custom neural model, the unsupported field types are ignored.
* Custom neural models are limited to 10 build operations per month. Open a support request if you need the limit increased.

## Training a model

Custom neural models are only available in the [v3 API](v3-migration-guide.md).

| Document Type | REST API | SDK | Label and Test Models|
|--|--|--|--|
| Custom document | [Form Recognizer 3.0 ](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-2022-08-31/operations/AnalyzeDocument)| [Form Recognizer SDK](quickstarts/get-started-sdks-rest-api.md?view=form-recog-3.0.0&preserve-view=true)| [Form Recognizer Studio](https://formrecognizer.appliedai.azure.com/studio)

The build operation to train model supports a new ```buildMode``` property, to train a custom neural model, set the ```buildMode``` to ```neural```.

```REST
https://{endpoint}/formrecognizer/documentModels:build?api-version=2022-08-31

{
  "modelId": "string",
  "description": "string",
  "buildMode": "neural",
  "azureBlobSource":
  {
    "containerUrl": "string",
    "prefix": "string"
  }
}
```

## Next steps

Learn to create and compose custom models:

> [!div class="nextstepaction"]
> [**Build a custom model**](how-to-guides/build-a-custom-model.md)
> [**Compose custom models**](how-to-guides/compose-custom-models.md)
