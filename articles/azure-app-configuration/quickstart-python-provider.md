---
title: Quickstart for using Azure App Configuration with Python apps | Microsoft Learn
description: In this quickstart, create a Python app with the Azure App Configuration to centralize storage and management of application settings separate from your code.
services: azure-app-configuration
author: maud-lv
ms.service: azure-app-configuration
ms.devlang: python
ms.topic: quickstart
ms.custom: devx-track-python, mode-other, engagement-fy23
ms.date: 03/10/2023
ms.author: malev
#Customer intent: As a Python developer, I want to manage all my app settings in one place.
---
# Quickstart: Create a Python app with Azure App Configuration

In this quickstart, you will use the Python provider for Azure App Configuration to centralize storage and management of application settings using the [Azure App Configuration Python provider client library](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/appconfiguration/azure-appconfiguration-provider).

The Python App Configuration provider is a library running on top of the [Azure SDK for Python](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/appconfiguration/azure-appconfiguration), helping Python developers easily consume the App Configuration service. It enables configuration settings to be used like a dictionary.

## Prerequisites

- Azure subscription - [create one for free](https://azure.microsoft.com/free/)
- Python 3.6 or later - for information on setting up Python on Windows, see the [Python on Windows documentation](/windows/python/)

## Create an App Configuration store

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

9. Select **Configuration Explorer** > **Create** > **Key-value** to add the following key-value pairs:

    | Key            | Value             | Label       | Content type       |
    |----------------|-------------------|-------------|--------------------|
    | *message*      | *Hello*           | Leave empty | Leave empty        |
    | *test.message* | *Hello test*      | Leave empty | Leave empty        |
    | *my_json*      | *{"key":"value"}* | Leave empty | *application/json* |

10. Select **Apply**.

## Set up the Python app

1. Create a new directory for the project named *app-configuration-quickstart*.

    ```console
    mkdir app-configuration-quickstart
    ```

1. Switch to the newly created *app-configuration-quickstart* directory.

    ```console
    cd app-configuration-quickstart
    ```

1. Install the Azure App Configuration provider by using the `pip install` command.

    ```console
    pip install azure-appconfiguration-provider
    ```

1. Create a new file called *app-configuration-quickstart.py* in the *app-configuration-quickstart* directory and add the following code:

    ```python
    from azure.appconfiguration.provider import (
        load,
        SettingSelector
    )
    import os

    connection_string = os.environ.get("AZURE_APPCONFIG_CONNECTION_STRING")

    # Connect to Azure App Configuration using a connection string.
    config = load(connection_string=connection_string)

    # Find the key "message" and print its value.
    print(config["message"])
    # Find the key "my_json" and print the value for "key" from the dictionary.
    print(config["my_json"]["key"])

    # Connect to Azure App Configuration using a connection string and trimmed key prefixes.
    trimmed = {"test."}
    config = load(connection_string=connection_string, trim_prefixes=trimmed)
    # From the keys with trimmed prefixes, find a key with "message" and print its value.
    print(config["message"])

    # Connect to Azure App Configuration using SettingSelector.
    selects = {SettingSelector(key_filter="message*", label_filter="\0")}
    config = load(connection_string=connection_string, selects=selects)

   # Print True or False to indicate if "message" is found in Azure App Configuration.
    print("message found: " + str("message" in config))
    print("test.message found: " + str("test.message" in config))
    ```

## Configure your App Configuration connection string

1. Set an environment variable named **AZURE_APPCONFIG_CONNECTION_STRING**, and set it to the connection string of your App Configuration store. At the command line, run the following command:

    ### [Windows command prompt](#tab/windowscommandprompt)

    To build and run the app locally using the Windows command prompt, run the following command and replace `<app-configuration-store-connection-string>` with the connection string of your app configuration store:

    ```cmd
    setx AZURE_APPCONFIG_CONNECTION_STRING "connection-string-of-your-app-configuration-store"
    ```

    ### [PowerShell](#tab/powershell)

    If you use Windows PowerShell, run the following command and replace `<app-configuration-store-connection-string>` with the connection string of your app configuration store:

    ```azurepowershell
    $Env:AZURE_APPCONFIG_CONNECTION_STRING = "<app-configuration-store-connection-string>"
    ```

    ### [macOS](#tab/unix)

    If you use macOS, run the following command and replace `<app-configuration-store-connection-string>` with the connection string of your app configuration store:

    ```console
    export AZURE_APPCONFIG_CONNECTION_STRING='<app-configuration-store-connection-string>'
    ```

    ### [Linux](#tab/linux)

    If you use Linux, run the following command and replace `<app-configuration-store-connection-string>` with the connection string of your app configuration store:

    ```console
    export AZURE_APPCONFIG_CONNECTION_STRING='<app-configuration-store-connection-string>'
   ```

1. Restart the command prompt to allow the change to take effect. Print out the value of the environment variable to validate that it is set properly with the command below.

    ### [Windows command prompt](#tab/windowscommandprompt)

    Using the Windows command prompt, run the following command:

    ```cmd
    printenv AZURE_APPCONFIG_CONNECTION_STRING
    ```

    ### [PowerShell](#tab/powershell)

    If you use Windows PowerShell, run the following command:

    ```azurepowershell
    $Env:AZURE_APPCONFIG_CONNECTION_STRING
    ```

    ### [macOS](#tab/unix)

    If you use macOS, run the following command:

    ```console
    echo "$AZURE_APPCONFIG_CONNECTION_STRING"
    ```

    ### [Linux](#tab/linux)

    If you use Linux, run the following command:

    ```console
    echo "$AZURE_APPCONFIG_CONNECTION_STRING"

1. After the build successfully completes, run the following command to run the app locally:

    ```python
    python app-configuration-quickstart.py
    ```

    You should see the following output:

    ```Output
    Hello
    value
    Hello test
    message found: True
    test.message found: False
    ```

## Clean up resources

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## Next steps

In this quickstart, you created a new App Configuration store and learned how to access key-values from a Python app.

For additional code samples, visit:

> [!div class="nextstepaction"]
> [Django Sample](https://github.com/Azure/AppConfiguration/tree/main/examples/Python/python-django-webapp-sample)
> [Flask Sample](https://github.com/Azure/AppConfiguration/tree/main/examples/Python/python-flask-webapp-sample)
> [Azure App Configuration Python provider](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/appconfiguration/azure-appconfiguration-provider)
