---
title: Utiliser le Générateur d’images Azure avec une galerie d’images pour machines virtuelles Windows (préversion)
description: Créer des versions d'images Azure Shared Gallery à l'aide d'Azure Image Builder et d'Azure PowerShell.
author: cynthn
ms.author: cynthn
ms.date: 01/14/2020
ms.topic: article
ms.service: virtual-machines-windows
manager: gwallace
ms.openlocfilehash: d5856780d0d9f1a1943bca1c2f076bb3ec914e1d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76263351"
---
# <a name="preview-create-a-windows-image-and-distribute-it-to-a-shared-image-gallery"></a>Aperçu : Créer une image Windows et la distribuer sur une bibliothèque d’images partagées 

Cet article explique comment utiliser Azure Image Builder et Azure PowerShell pour créer une version d'image dans une instance de [Shared Image Gallery](shared-image-galleries.md) avant de distribuer l'image dans le monde entier. Vous pouvez également effectuer cette opération à l'aide d'[Azure CLI](../linux/image-builder-gallery.md).

Nous utiliserons un modèle .json.pour configurer l’image. Le fichier .json utilisé est le suivant : [armTemplateWinSIG.json](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/1_Creating_a_Custom_Win_Shared_Image_Gallery_Image/armTemplateWinSIG.json). Nous allons télécharger et modifier une version locale du modèle. Cet article part donc du principe qu'une session PowerShell locale est utilisée.

Pour distribuer l’image à une galerie d’images partagées, le modèle utilise [sharedImage](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#distribute-sharedimage) comme valeur de la section `distribute` du modèle.

Azure Image Builder exécute automatiquement sysprep pour généraliser l'image. Il s'agit d'une commande générique sysprep, que vous pouvez [écraser](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md#vms-created-from-aib-images-do-not-create-successfully) si nécessaire. 

Tenez compte du nombre de personnalisations que vous effectuez. Vous pouvez exécuter la commande Sysprep jusqu'à 8 fois sur une même image Windows. Après avoir exécuté Sysprep 8 fois, vous devrez recréer votre image Windows. Pour plus d'informations, consultez [Nombre d'exécutions maximum de Sysprep](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation#limits-on-how-many-times-you-can-run-sysprep). 

> [!IMPORTANT]
> Le Générateur d’images Azure est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="register-the-features"></a>Inscrire les fonctionnalités
Pour utiliser le Générateur d’images Azure pendant la préversion, vous devez inscrire la nouvelle fonctionnalité.

```powershell
Register-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages
```

Vérifiez l’état d’inscription de la fonctionnalité.

```powershell
Get-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages
```

Attendez que `RegistrationState` soit `Registered` avant de passer à l'étape suivante.

Vérifiez les inscriptions de votre fournisseur. Assurez-vous que chacun renvoie `Registered`.

```powershell
Get-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages | Format-table -Property ResourceTypes,RegistrationState
Get-AzResourceProvider -ProviderNamespace Microsoft.Storage | Format-table -Property ResourceTypes,RegistrationState 
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute | Format-table -Property ResourceTypes,RegistrationState
Get-AzResourceProvider -ProviderNamespace Microsoft.KeyVault | Format-table -Property ResourceTypes,RegistrationState
```

S'ils ne renvoient pas `Registered`, utilisez ce qui suit pour inscrire les fournisseurs :

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
```

## <a name="create-variables"></a>Créer des variables

Nous allons utiliser certains éléments d’information à plusieurs reprises, donc nous allons créer des variables pour les stocker. Remplacez la valeur des variables, comme `username` et `vmpassword`, par vos propres informations.

```powershell
# Get existing context
$currentAzContext = Get-AzContext

# Get your current subscription ID. 
$subscriptionID=$currentAzContext.Subscription.Id

# Destination image resource group
$imageResourceGroup="aibwinsig"

# Location
$location="westus"

# Image distribution metadata reference name
$runOutputName="aibCustWinManImg02ro"

# Image template name
$imageTemplateName="helloImageTemplateWin02ps"

# Distribution properties object name (runOutput).
# This gives you the properties of the managed image on completion.
$runOutputName="winclientR01"
```



## <a name="create-the-resource-group"></a>Créer le groupe de ressources

Créez un groupe de ressources et accordez à Azure Image Builder l'autorisation de créer des ressources dans ce groupe de ressources.

```powershell
New-AzResourceGroup `
   -Name $imageResourceGroup `
   -Location $location
New-AzRoleAssignment `
   -ObjectId ef511139-6170-438e-a6e1-763dc31bdf74 `
   -Scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup `
   -RoleDefinitionName Contributor
```



## <a name="create-the-shared-image-gallery"></a>Créer la galerie Shared Image Gallery

Pour utiliser Image Builder avec une galerie d’images partagées, vous devez disposer d’une bibliothèque d’images et d’une définition d’image existantes. Image Builder ne va pas créer la Galerie d’images et la définition d’image pour vous.

Si vous n’avez pas encore de définition d’image et de galerie à utiliser, commencez par les créer. Tout d’abord, créez une galerie d’images.

```powershell
# Image gallery name
$sigGalleryName= "myIBSIG"

# Image definition name
$imageDefName ="winSvrimage"

# additional replication region
$replRegion2="eastus"

# Create the gallery
New-AzGallery `
   -GalleryName $sigGalleryName `
   -ResourceGroupName $imageResourceGroup  `
   -Location $location

# Create the image definition
New-AzGalleryImageDefinition `
   -GalleryName $sigGalleryName `
   -ResourceGroupName $imageResourceGroup `
   -Location $location `
   -Name $imageDefName `
   -OsState generalized `
   -OsType Windows `
   -Publisher 'myCompany' `
   -Offer 'WindowsServer' `
   -Sku 'WinSrv2019'
```



## <a name="download-and-configure-the-template"></a>Télécharger et configurer le modèle

Téléchargez le modèle .json et configurez-le avec vos variables.

```powershell

$templateFilePath = "armTemplateWinSIG.json"

Invoke-WebRequest `
   -Uri "https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/1_Creating_a_Custom_Win_Shared_Image_Gallery_Image/armTemplateWinSIG.json" `
   -OutFile $templateFilePath `
   -UseBasicParsing

(Get-Content -path $templateFilePath -Raw ) `
   -replace '<subscriptionID>',$subscriptionID | Set-Content -Path $templateFilePath
(Get-Content -path $templateFilePath -Raw ) `
   -replace '<rgName>',$imageResourceGroup | Set-Content -Path $templateFilePath
(Get-Content -path $templateFilePath -Raw ) `
   -replace '<runOutputName>',$runOutputName | Set-Content -Path $templateFilePath
(Get-Content -path $templateFilePath -Raw ) `
   -replace '<imageDefName>',$imageDefName | Set-Content -Path $templateFilePath
(Get-Content -path $templateFilePath -Raw ) `
   -replace '<sharedImageGalName>',$sigGalleryName | Set-Content -Path $templateFilePath
(Get-Content -path $templateFilePath -Raw ) `
   -replace '<region1>',$location | Set-Content -Path $templateFilePath
(Get-Content -path $templateFilePath -Raw ) `
   -replace '<region2>',$replRegion2 | Set-Content -Path $templateFilePath

```


## <a name="create-the-image-version"></a>Créer la version de l’image

Votre modèle doit être envoyé au service. Cela permettra de télécharger tous les artefacts dépendants, comme les scripts, et de les stocker dans le groupe de ressources intermédiaire, avec le préfixe *IT_* .

```powershell
New-AzResourceGroupDeployment `
   -ResourceGroupName $imageResourceGroup `
   -TemplateFile $templateFilePath `
   -api-version "2019-05-01-preview" `
   -imageTemplateName $imageTemplateName `
   -svclocation $location
```

Pour générer l'image, vous devez appeler « Run » sur le modèle.

```powershell
Invoke-AzResourceAction `
   -ResourceName $imageTemplateName `
   -ResourceGroupName $imageResourceGroup `
   -ResourceType Microsoft.VirtualMachineImages/imageTemplates `
   -ApiVersion "2019-05-01-preview" `
   -Action Run
```

La création de l’image et sa réplication dans les deux régions peuvent prendre un certain temps. Attendez la fin de cette partie pour passer à la création d’une machine virtuelle.

Pour plus d'informations sur les options d'automatisation de l'obtention de l'état de génération d'image, consultez le fichier [Lisez-moi](https://github.com/danielsollondon/azvmimagebuilder/blob/master/quickquickstarts/1_Creating_a_Custom_Win_Shared_Image_Gallery_Image/readme.md#get-status-of-the-image-build-and-query) de ce modèle sur GitHub.


## <a name="create-the-vm"></a>Création de la machine virtuelle

Créez une machine virtuelle à partir de la version de l’image créée par le Générateur d’images Azure.

Procurez-vous la version de l'image que vous avez créée.
```powershell
$imageVersion = Get-AzGalleryImageVersion `
   -ResourceGroupName $imageResourceGroup `
   -GalleryName $sigGalleryName `
   -GalleryImageDefinitionName $imageDefName
```

Créez la machine virtuelle dans la deuxième région où l'image a été répliquée.

```powershell
$vmResourceGroup = "myResourceGroup"
$vmName = "myVMfromImage"

# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create a resource group
New-AzResourceGroup -Name $vmResourceGroup -Location $replRegion2

# Network pieces
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $vmResourceGroup -Location $replRegion2 `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $vmResourceGroup -Location $replRegion2 `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $vmResourceGroup -Location $replRegion2 `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName $vmResourceGroup -Location $replRegion2 `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using $imageVersion.Id to specify the shared image
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1_v2 | `
Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred | `
Set-AzVMSourceImage -Id $imageVersion.Id | `
Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM -ResourceGroupName $vmResourceGroup -Location $replRegion2 -VM $vmConfig
```

## <a name="verify-the-customization"></a>Vérifier la personnalisation
Créez une connexion Bureau à distance à la machine virtuelle avec le nom d’utilisateur et le mot de passe définis lors de la création de la machine virtuelle. Dans la machine virtuelle, ouvrez une invite de commande et tapez :

```console
dir c:\
```

Le répertoire nommé `buildActions`, créé pendant la personnalisation de l’image, devrait apparaître.


## <a name="clean-up-resources"></a>Nettoyer les ressources
Si vous souhaitez maintenant essayer de repersonnaliser la version de l’image pour créer une nouvelle version de la même image, **ignorez cette étape** et passez à [Utiliser le Générateur d’images Azure pour créer une autre version de l’image](image-builder-gallery-update-image-version.md).


L’image créée ainsi que tous les autres fichiers de ressources seront ainsi supprimés. Terminez ce déploiement avant de supprimer les ressources.

Supprimez d'abord le modèle de groupe de ressources, sinon le groupe de ressources intermédiaire (*IT_* ) utilisé par AIB ne sera pas nettoyé.

Procurez-vous l'ID de ressource du modèle d'image. 

```powerShell
$resTemplateId = Get-AzResource -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2019-05-01-preview"
```

Supprimez le modèle d'image.

```powerShell
Remove-AzResource -ResourceId $resTemplateId.ResourceId -Force
```

Supprimez le groupe de ressources.

```powerShell
Remove-AzResourceGroup $imageResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment mettre à jour la version de l’image créée, voir [Utiliser le Générateur d’images Azure pour créer une autre version de l’image](image-builder-gallery-update-image-version.md).
