---
title: Configuration du service – QnA Maker
description: Comprendre comment et où configurer les ressources.
ms.topic: reference
ms.date: 02/21/2020
ms.openlocfilehash: 8ef6fecbfeb119d0c68ec5bc3bbc90ec449dbb7d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77651803"
---
# <a name="service-configuration"></a>Configuration de service

QnA Maker utilise plusieurs ressources Azure (services), dont Recherche cognitive, App Service, Plan App Service et Application Insights.

Toutes les personnalisations apportées à ces paramètres qui sont prises en charge par QnA Maker sont répertoriées ci-dessous.

## <a name="app-service"></a>App Service

QnA Maker utilise App Service pour fournir le runtime de requête utilisé par l’[API generateAnswer](https://docs.microsoft.com/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer).


Ces paramètres sont disponibles dans le Portail Azure, pour App Service. Les paramètres sont disponibles en sélectionnant **Paramètres**, puis **Configuration**.

Vous pouvez définir un paramètre individuel via la liste des paramètres de l’application ou modifier plusieurs paramètres en sélectionnant **Modification avancée**.

|Ressource|Paramètre|
|--|--|
|AzureSearchAdminKey|Recherche cognitive : utilisé pour le stockage des ensembles QnA et le classement n °1|
|AzureSearchName|Recherche cognitive : utilisé pour le stockage des ensembles QnA et le classement n °1|
|DefaultAnswer|Texte de réponse quand aucune correspondance n’est trouvée|
|UserAppInsightsAppId|Journal et télémétrie de conversation|
|UserAppInsightsKey|Journal et télémétrie de conversation|
|UserAppInsightsName|Journal et télémétrie de conversation|

Découvrez [comment ajouter la modification du service Recherche cognitive](./how-to/set-up-qnamaker-service-azure.md#configure-qna-maker-to-use-different-cognitive-search-resource) à votre service.

Vous devez **redémarrer** le service à partir de la page **Vue d’ensemble** du Portail Azure, une fois que vous avez apporté des modifications.

## <a name="qna-maker-service"></a>Service d’établissement QnA

Le service QnA Maker fournit une configuration permettant aux utilisateurs suivants de collaborer sur un seul service QnA Maker, ainsi que toutes ses bases de connaissances.

Découvrez [comment ajouter des collaborateurs](./how-to/collaborate-knowledge-base.md) à votre service.

## <a name="application-insights"></a>Application Insights

Application Insights n’a aucun paramètre de configuration spécifique à QnA Maker.

## <a name="app-service-plan"></a>Plan App Service

Plan App Service n’a aucun paramètre de configuration spécifique à QnA Maker.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [formats](reference-document-format-guidelines.md) pour les documents et les URL que vous souhaitez importer dans une base de connaissances.