---
title: Livraison sécurisée de webhooks à l’aide d’Azure AD dans Azure Event Grid
description: Décrit comment livrer des événements aux points de terminaison HTTPS protégés par Azure Active Directory à l’aide d’Azure Event Grid
services: event-grid
author: banisadr
ms.service: event-grid
ms.topic: conceptual
ms.date: 11/18/2019
ms.author: babanisa
ms.openlocfilehash: 074378668b0516936e11968ea8c800d3daa667bb
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74931553"
---
# <a name="publish-events-to-azure-active-directory-protected-endpoints"></a>Publier des événements sur des points de terminaison protégés par Azure Active Directory

Cet article explique comment tirer parti d’Azure Active Directory pour sécuriser la connexion entre votre abonnement aux événements et votre point de terminaison webhook. Pour obtenir une vue d’ensemble des applications Azure AD et des principaux de service, consultez [Présentation de la plateforme d’identités Microsoft (v2.0)](https://docs.microsoft.com/azure/active-directory/develop/v2-overview).

Cet article utilise le Portail Azure à des fins de démonstration, mais la fonctionnalité peut également être activée à l’aide de l’interface CLI, de PowerShell ou des Kits de développement logiciel (SDK).

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="create-an-azure-ad-application"></a>Créer une application Azure AD

Commencez par créer une application Azure AD pour votre point de terminaison protégé. Consultez https://docs.microsoft.com/azure/active-directory/develop/scenario-protected-web-api-overview.
    - Configurez votre API protégée pour qu’elle soit appelée par une application démon.
    
## <a name="enable-event-grid-to-use-your-azure-ad-application"></a>Activer Event Grid pour utiliser votre application Azure AD

Utilisez le script PowerShell ci-dessous pour créer un rôle et un principe de service dans votre application Azure AD. Vous aurez besoin de l’ID de locataire et de l’ID d’objet de votre application Azure AD :

    > [!NOTE]
    > You must be a member of the [Azure AD Application Administrator role](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles#available-roles) to execute this script.
    
1. Modifiez le $myTenantId du script PowerShell pour utiliser votre ID de locataire Azure AD.
1. Modifiez le $myAzureADApplicationObjectId du script PowerShell pour utiliser l’ID d’objet de votre application Azure AD.
1. Exécutez le script modifié.

```PowerShell
# This is your Tenant Id. 
$myTenantId = "<the Tenant Id of your Azure AD Application>"

Connect-AzureAD -TenantId $myTenantId
    
# This is your Azure AD Application's ObjectId. 
$myAzureADApplicationObjectId = "<the Object Id of your Azure AD Application>"
    
# This is the "Azure Event Grid" Azure Active Directory AppId
$eventGridAppId = "4962773b-9cdb-44cf-a8bf-237846a00ab7"
    
# This is the name of the new role we will add to your Azure AD Application
$eventGridRoleName = "AzureEventGridSecureWebhook"
    
# Create an application role of given name and description
Function CreateAppRole([string] $Name, [string] $Description)
{
    $appRole = New-Object Microsoft.Open.AzureAD.Model.AppRole
    $appRole.AllowedMemberTypes = New-Object System.Collections.Generic.List[string]
    $appRole.AllowedMemberTypes.Add("Application");
    $appRole.DisplayName = $Name
    $appRole.Id = New-Guid
    $appRole.IsEnabled = $true
    $appRole.Description = $Description
    $appRole.Value = $Name;
    return $appRole
}
    
# Get my Azure AD Application, it's roles and service principal
$myApp = Get-AzureADApplication -ObjectId $myAzureADApplicationObjectId
$myAppRoles = $myApp.AppRoles
$eventGridSP = Get-AzureADServicePrincipal -Filter ("appId eq '" + $eventGridAppId + "'")

Write-Host "App Roles before addition of new role.."
Write-Host $myAppRoles
    
# Create the role if it doesn't exist
if ($myAppRoles -match $eventGridRoleName)
{
    Write-Host "The Azure Event Grid role is already defined.`n"
}
else
{
    $myServicePrincipal = Get-AzureADServicePrincipal -Filter ("appId eq '" + $myApp.AppId + "'")
    
    # Add our new role to the Azure AD Application
    $newRole = CreateAppRole -Name $eventGridRoleName -Description "Azure Event Grid Role"
    $myAppRoles.Add($newRole)
    Set-AzureADApplication -ObjectId $myApp.ObjectId -AppRoles $myAppRoles
}
    
# Create the service principal if it doesn't exist
if ($eventGridSP -match "Microsoft.EventGrid")
{
    Write-Host "The Service principal is already defined.`n"
}
else
{
    # Create a service principal for the "Azure Event Grid" Azure AD Application and add it to the role
    $eventGridSP = New-AzureADServicePrincipal -AppId $eventGridAppId
}
    
New-AzureADServiceAppRoleAssignment -Id $myApp.AppRoles[0].Id -ResourceId $myServicePrincipal.ObjectId -ObjectId $eventGridSP.ObjectId -PrincipalId $eventGridSP.ObjectId
    
Write-Host "My Azure AD Tenant Id" + $myTenantId
Write-Host "My Azure AD Application Id" + $myAzureADApplicationObjectId
Write-Host "My Azure AD Application ($myApp.ObjectId): " + $myApp.ObjectId
Write-Host "My Azure AD Application's Roles"
Write-Host $myApp.AppRoles
```
    
## <a name="configure-the-event-subscription"></a>Configurer l’abonnement aux événements

Dans le flux de création de votre abonnement aux événements, sélectionnez le type de point de terminaison « Webhook ». Une fois que vous avez donné votre URI de point de terminaison, cliquez sur l’onglet Fonctionnalités supplémentaires en haut du panneau Créer des abonnements aux événements.

![Sélectionner un webhook de type de point de terminaison](./media/secure-webhook-delivery/select-webhook.png)

Sous l’onglet Fonctionnalités supplémentaires, cochez la case « Utiliser l’authentification AAD » et configurez l’ID de locataire et l’ID d’application :

* Copiez l’ID de locataire Azure AD à partir de la sortie du script et entrez-le dans le champ ID de locataire AAD.
* Copiez l’ID d’application Azure AD à partir de la sortie du script et entrez-le dans le champ ID d’application AAD.

    ![Action de webhook sécurisé](./media/secure-webhook-delivery/aad-configuration.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’information sur la surveillance des remises des événements, consultez [Surveiller la remise des messages Event Grid](monitor-event-delivery.md).
* Pour plus d’informations sur la clé d’authentification, consultez la page [Sécurité et authentification Azure Event Grid](security-authentication.md).
* Pour plus d’informations sur la création d’un abonnement Azure Event Grid, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).
