---
title: Custom classifier model - Form Recognizer
titleSuffix: Azure Applied AI Services
description: Use the custom classifier model to train a model to identify and split the documents you process within your application.
author: vkurpad
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

# Custom classifier model

**This article applies to:** ![Form Recognizer v3.0 checkmark](media/yes-icon.png) **Form Recognizer v3.0**.

Custom classifier models are deep-learning-model types that combine layout and language features to accurately detect and identify documents you process within your application. Custom classifier models can classify each page in an input file to identify the document(s) within and can also identify multiple documents or multiple instances of a single document within an input file.

## Model capabilities

Custom classifier models can analyze a single- or multi-file documents to identify if any of the trained document types are contained within an input file. Here are the currently supported scenarios:

* A single file containing one document. For instance, a loan application form.

* A single file containing multiple documents. For instance, a loan application package containing a loan application form, payslip, and bank statement.

* A single file containing multiple instances of the same document. For instance, a collection of scanned invoices.

Training a custom classifier model requires at least two distinct classes and a minimum of five samples per class.

### Compare custom classifier and composed models

A custom classifier model can replace [a composed model](concept-composed-models.md) in some scenarios but there are a few differences to be aware of:

| Capability | Custom classifier process | Composed model process |
|--|--|--|
|Analyze a single document of unknown type belonging to one of the types trained for extraction model processing.| &#9679; Requires multiple calls. </br> &#9679; Call the classifier models based on the document class. This step allows for a confidence-based check before invoking the extraction model analysis.</br> &#9679; Invoke the extraction model. | &#9679; Requires a single call to a composed model containing the model corresponding to the input document type. |
 |Analyze a single document of unknown type belonging to several types trained for extraction model processing.| &#9679;Requires multiple calls.</br> &#9679; Make a call to the classifier that ignores documents not matching a designated type for extraction.</br> &#9679; Invoke the extraction model. | &#9679;  Requires a single call to a composed model. The service selects a custom model within the composed model with the highest match.</br> &#9679; A composed model can't ignore documents.|
|Analyze a file containing multiple documents of known or unknown type belonging to one of the types trained for extraction model processing.| &#9679; Requires multiple calls. </br> &#9679; Call the extraction model for each identified document in the input file.</br> &#9679; Invoke the extraction model. | &#9679;  Requires a single call to a composed model.</br> &#9679; The composed model invokes the component model once on the first instance of the document. </br> &#9679;The remaining documents are ignored. |

## Language support

Classifier models currently only support English language documents.

## Best practices

Custom classifier models require a minimum of five samples per class to train. If the classes are similar, adding extra training samples improves model accuracy.

## Training a model

Custom classifier models are only available in the [v3.0 API](v3-migration-guide.md) starting with API version ```2023-02-28-preview```. [Form Recognizer Studio](https://formrecognizer.appliedai.azure.com/studio) provides a no-code user interface to interactively train a custom classifier.

When using the REST API, if you've organized your documents by folders, you can use the ```azureBlobSource``` property of the request to train a classifier model.

```rest
https://{endpoint}/formrecognizer/documentClassifiers:build?api-version=2023-02-28-preview

{
  "classifierId": "demo2.1",
  "description": "",
  "docTypes": {
    "car-maint": {
        "azureBlobSource": {
            "containerUrl": "SAS URL to container",
            "prefix": "sample1/car-maint/"
            }
    },
    "cc-auth": {
        "azureBlobSource": {
            "containerUrl": "SAS URL to container",
            "prefix": "sample1/cc-auth/"
            }
    },
    "deed-of-trust": {
        "azureBlobSource": {
            "containerUrl": "SAS URL to container",
            "prefix": "sample1/deed-of-trust/"
            }
    }
  }
}

```

Alternatively, if you have a flat list of files or only plan to use a few select files within each folder to train the model, you can use the ```azureBlobFileListSource``` property to train the model. This step requires a ```file list``` in [JSON Lines](https://jsonlines.org/) format. For each class, add a new file with a list of files to be submitted for training.

```rest
{
  "classifierId": "demo2",
  "description": "",
  "docTypes": {
    "car-maint": {
      "azureBlobFileListSource": {
        "containerUrl": "SAS URL to container",
        "fileList": "sample1/car-maint.jsonl"
      }
    },
    "cc-auth": {
      "azureBlobFileListSource": {
        "containerUrl": "SAS URL to container",
        "fileList": "sample1/cc-auth.jsonl"
      }
    },
    "deed-of-trust": {
      "azureBlobFileListSource": {
        "containerUrl": "SAS URL to container",
        "fileList": "sample1/deed-of-trust.jsonl"
      }
    }
  }
}

```

File list `car-maint.jsonl` contains the following files.

```json
{"file":"sample1/car-maint/Commercial Motor Vehicle - Adatum.pdf"}
{"file":"sample1/car-maint/Commercial Motor Vehicle - Fincher.pdf"}
{"file":"sample1/car-maint/Commercial Motor Vehicle - Lamna.pdf"}
{"file":"sample1/car-maint/Commercial Motor Vehicle - Liberty.pdf"}
{"file":"sample1/car-maint/Commercial Motor Vehicle - Trey.pdf"}
```

## Next steps

Learn to create custom classifier models:

> [!div class="nextstepaction"]
> [**Build a custom classifier model**](how-to-guides/build-a-custom-classifier.md)
> [**Custom models overview**](concept-custom.md)
