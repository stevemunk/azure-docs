---
title: Configure a Dev Box with Azure Image Builder
titleSuffix: Microsoft Dev Box Preview
description: 'Learn how to create a custom image with Azure Image Builder, then create a Dev box with the image.'
services: dev-box
ms.service: dev-box
ms.custom: devx-track-azurepowershell
author: RoseHJM
ms.author: rosemalcolm
ms.date: 11/17/2022
ms.topic: how-to
---

# Configure a Dev Box with Azure Image Builder

By using standardized virtual machine (VM) images, your organization can more easily migrate to the cloud and help ensure consistency in your deployments. Images ordinarily include predefined security, configuration settings, and any necessary software. Setting up your own imaging pipeline requires time, infrastructure, and many other details. With Azure VM Image Builder, you can create a configuration that describes your image and submit it to the service, where the image is built and then distributed to a dev box project. In this article, you will create a customized dev box using a template which includes a customization step to install Visual Studio Code. 

Although it's possible to create custom VM images by hand or by using other tools, the process can be cumbersome and unreliable. VM Image Builder, which is built on HashiCorp Packer, gives you the benefits of a managed service. 

To reduce the complexity of creating VM images, VM Image Builder: 

- Removes the need to use complex tooling, processes, and manual steps to create a VM image. VM Image Builder abstracts out all these details and hides Azure-specific requirements, such as the need to generalize the image (Sysprep). And it gives more advanced users the ability to override such requirements. 

- Can be integrated with existing image build pipelines for a click-and-go experience. To do so, you can either call VM Image Builder from your pipeline or use an Azure VM Image Builder service DevOps task (preview). 

- Can fetch customization data from various sources, which removes the need to collect them all from one place. 

- Can be integrated with Compute Gallery, which creates an image management system with which to distribute, replicate, version, and scale images globally. Additionally, you can distribute the same resulting image as a VHD or as one or more managed images, without having to rebuild them from scratch. 

## Prerequisites
To provision a custom image you've creating by using VM Image Builder, you need: 
- Owner or Contributor permissions on an Azure Subscription or a specific resource group.
- A resource group 
- A dev center with an attached network connection. 
  If you don't have a dev center with an attached network connection, follow these steps to attach the network connection: [Create a network connection](./quickstart-configure-dev-box-service.md#create-a-network-connection).

## Create a Windows image and distribute it to an Azure Compute Gallery 
The next step is to use Azure VM Image Builder and Azure PowerShell to create an image version in an Azure Compute Gallery (formerly Shared Image Gallery) and then distribute the image globally. You can also do this by using the Azure CLI. 

1. To use VM Image Builder, you need to register the features. 

    Check your provider registrations. Make sure that each one returns Registered. 
    
    ```powershell
        Get-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages | Format-table -Property ResourceTypes,RegistrationState 
        Get-AzResourceProvider -ProviderNamespace Microsoft.Storage | Format-table -Property ResourceTypes,RegistrationState  
        Get-AzResourceProvider -ProviderNamespace Microsoft.Compute | Format-table -Property ResourceTypes,RegistrationState 
        Get-AzResourceProvider -ProviderNamespace Microsoft.KeyVault | Format-table -Property ResourceTypes,RegistrationState 
        Get-AzResourceProvider -ProviderNamespace Microsoft.Network | Format-table -Property ResourceTypes,RegistrationState 
    ```
        
    
    If they don't return Registered, register the providers by running the following commands: 
    ```powershell
        Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages  
        Register-AzResourceProvider -ProviderNamespace Microsoft.Storage  
        Register-AzResourceProvider -ProviderNamespace Microsoft.Compute  
        Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault  
        Register-AzResourceProvider -ProviderNamespace Microsoft.Network 
    ```
    
2. Install PowerShell modules: 

    ```powershell
    'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}
    ```

3.  Create variables to store information that you'll use more than once. 

    Copy the sample code and replace the Resource group with the resource group you have used to create the dev center. 
    
    ```powershell
    # Get existing context 
    $currentAzContext = Get-AzContext  
    # Get your current subscription ID.  
    $subscriptionID=$currentAzContext.Subscription.Id  
    # Destination image resource group  
    $imageResourceGroup="<Resource group>"  
    # Location  
    $location="eastus2"  
    # Image distribution metadata reference name  
    $runOutputName="aibCustWinManImg01"  
    # Image template name  
    $imageTemplateName="vscodeWinTemplate"  
    ```

4. Create a user-assigned identity and set permissions on the resource group 

    VM Image Builder uses the provided user-identity to inject the image into Azure Compute Gallery. In this example, you create an Azure role definition with specific actions for distributing the image. The role definition is then assigned to the user identity. 
    
    ```powershell
    # setup role def names, these need to be unique 
    $timeInt=$(get-date -UFormat "%s") 
    $imageRoleDefName="Azure Image Builder Image Def"+$timeInt 
    $identityName="aibIdentity"+$timeInt 
     
    ## Add an Azure PowerShell module to support AzUserAssignedIdentity 
    Install-Module -Name Az.ManagedServiceIdentity 
     
    # Create an identity 
    New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName -Location $location
     
    $identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id 
    $identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId
    ```

5. Assign permissions for the identity to distribute the images 

    Use this command to download an Azure role definition template, and then update it with the previously specified parameters. 
    
    ```powershell
    $aibRoleImageCreationUrl="https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json" 
    $aibRoleImageCreationPath = "aibRoleImageCreation.json" 
    
    # Download the configuration 
    Invoke-WebRequest -Uri $aibRoleImageCreationUrl -OutFile $aibRoleImageCreationPath -UseBasicParsing 
    ((Get-Content -path $aibRoleImageCreationPath -Raw) -replace '<subscriptionID>',$subscriptionID) | Set-Content -Path $aibRoleImageCreationPath 
    ((Get-Content -path $aibRoleImageCreationPath -Raw) -replace '<rgName>', $imageResourceGroup) | Set-Content -Path $aibRoleImageCreationPath 
    ((Get-Content -path $aibRoleImageCreationPath -Raw) -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $aibRoleImageCreationPath 
    
    # Create a role definition 
    New-AzRoleDefinition -InputFile  ./aibRoleImageCreation.json 
    # Grant the role definition to the VM Image Builder service principal 
    New-AzRoleAssignment -ObjectId $identityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup" 
    ```
    
## Create an Azure Compute Gallery 

To use VM Image Builder with an Azure Compute Gallery, you need to have an existing gallery and image definition. VM Image Builder doesn't create the gallery and image definition for you. The definition created below will have Trusted Launch as security type and meets the windows 365 image requirements. 

```powershell
# Gallery name 
$galleryName= "devboxGallery" 

# Image definition name 
$imageDefName ="vscodeImageDef" 

# Additional replication region 
$replRegion2="eastus" 

# Create the gallery 
New-AzGallery -GalleryName $galleryName -ResourceGroupName $imageResourceGroup -Location $location 

$SecurityType = @{Name='SecurityType';Value='TrustedLaunch'}  
$features = @($SecurityType) 

# Create the image definition
New-AzGalleryImageDefinition -GalleryName $galleryName -ResourceGroupName $imageResourceGroup  -Location $location  -Name $imageDefName  -OsState generalized  -OsType Windows  -Publisher 'myCompany'  -Offer 'vscodebox'  -Sku '1-0-0' -Feature $features -HyperVGeneration "V2" 
```

1. Copy the ARM Template for Azure Image Builder. This template indicates the source image and also the customizations applied. With this template, we are installing choco and vscode. It also indicates where the image will be distributed.

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "imageTemplateName": {
            "type": "string"
          },
          "api-version": {
            "type": "string"
          },
          "svclocation": {
            "type": "string"
          }
        },
        "variables": {},
        "resources": [
          {
            "name": "[parameters('imageTemplateName')]",
            "type": "Microsoft.VirtualMachineImages/imageTemplates",
            "apiVersion": "[parameters('api-version')]",
            "location": "[parameters('svclocation')]",
            "dependsOn": [],
            "tags": {
              "imagebuilderTemplate": "win11multi",
              "userIdentity": "enabled"
            },
            "identity": {
              "type": "UserAssigned",
              "userAssignedIdentities": {
                "<imgBuilderId>": {}
              }
            },
            "properties": {
              "buildTimeoutInMinutes": 100,
              "vmProfile": {
                "vmSize": "Standard_DS2_v2",
                "osDiskSizeGB": 127
              },
            "source": {
                "type": "PlatformImage",
                "publisher": "MicrosoftWindowsDesktop",
                "offer": "Windows-11",
                "sku": "win11-21h2-avd",
                "version": "latest"
            },
              "customize": [
                {
                    "type": "PowerShell",
                    "name": "Install Choco and Vscode",
                    "inline": [
                        "Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))",
                        "choco install -y vscode"
                    ]
                }
              ],
                "distribute": 
                [
                    {   
                        "type": "SharedImage",
                        "galleryImageId": "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<sharedImageGalName>/images/<imageDefName>",
                        "runOutputName": "<runOutputName>",
                        "artifactTags": {
                            "source": "azureVmImageBuilder",
                            "baseosimg": "win11multi"
                        },
                        "replicationRegions": [
                          "<region1>",
                          "<region2>"
                        ]
                    }
                ]
            }
          }
        ]
      }
    ```
2. Configure the template with your variables. 
    ```powershell
    $templateFilePath = <Template Path>
    
    (Get-Content -path $templateFilePath -Raw ) -replace '<subscriptionID>',$subscriptionID | Set-Content -Path $templateFilePath 
    (Get-Content -path $templateFilePath -Raw ) -replace '<rgName>',$imageResourceGroup | Set-Content -Path $templateFilePath 
    (Get-Content -path $templateFilePath -Raw ) -replace '<runOutputName>',$runOutputName | Set-Content -Path $templateFilePath  
    (Get-Content -path $templateFilePath -Raw ) -replace '<imageDefName>',$imageDefName | Set-Content -Path $templateFilePath  
    (Get-Content -path $templateFilePath -Raw ) -replace '<sharedImageGalName>',$galleryName| Set-Content -Path $templateFilePath  
    (Get-Content -path $templateFilePath -Raw ) -replace '<region1>',$location | Set-Content -Path $templateFilePath  
    (Get-Content -path $templateFilePath -Raw ) -replace '<region2>',$replRegion2 | Set-Content -Path $templateFilePath  
    ((Get-Content -path $templateFilePath -Raw) -replace '<imgBuilderId>',$identityNameResourceId) | Set-Content -Path $templateFilePath 
    ```
3. Create the image version 

    Your template must be submitted to the service. The following commands will download any dependent artifacts, such as scripts, and store them in the staging resource group, which is prefixed with IT_. 
    
    ```powershell
    New-AzResourceGroupDeployment  -ResourceGroupName $imageResourceGroup  -TemplateFile $templateFilePath  -Api-Version "2020-02-14"  -imageTemplateName $imageTemplateName  -svclocation $location 
    ```
    To build the image, invoke 'Run' on the template. 
    ```powershell
    Invoke-AzResourceAction  -ResourceName $imageTemplateName  -ResourceGroupName $imageResourceGroup  -ResourceType Microsoft.VirtualMachineImages/imageTemplates  -ApiVersion "2020-02-14"  -Action Run 
    ```
    Creating the image and replicating it to both regions can take a few moments. Before you begin creating a dev box definition, wait until this part is finished.
    ```powershell
    Get-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup | Select-Object -Property Name, LastRunStatusRunState, LastRunStatusMessage, ProvisioningState 
    ```
    Alternatively, you can navigate to the Azure portal to your compute gallery > image definition to view the provisioning state of your image.
    
![Provisioning state of the customized image version](./media/how-to-customize-devbox-azure-image-builder/image-version-provisioning-state.png)

## Configure the Azure Compute Gallery 

Once your custom image has been provisioned within the compute gallery, you can configure the gallery to use the images within the dev center. More details here: 

[Configure an Azure Compute Gallery](./how-to-configure-azure-compute-gallery.md)

## Setting up Dev Box Service with Custom Image

Once the compute gallery images are available in the dev center. You can use the custom image with the dev box service. More details here:

[Configure the Microsoft Dev Box Service](./quickstart-configure-dev-box-service.md)

## Next steps
- [Create dev box definitions](./quickstart-configure-dev-box-service.md#create-a-dev-box-definition)
- [Configure an Azure Compute Gallery](./how-to-configure-azure-compute-gallery.md)
