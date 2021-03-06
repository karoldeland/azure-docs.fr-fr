---
title: Sécurité et authentification Azure Event Grid
description: Détaille Azure Event Grid et ses concepts.
services: event-grid
author: banisadr
manager: timlt
ms.service: event-grid
ms.topic: conceptual
ms.date: 05/22/2019
ms.author: babanisa
ms.openlocfilehash: 03bc2f9de6f50f08c9f62f86a3d1791a067cecd0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78899003"
---
# <a name="authorizing-access-to-event-grid-resources"></a>Autorisation de l’accès aux ressources Event Grid
Azure Event Grid vous permet de contrôler le niveau d’accès offert aux utilisateurs, leur permettant d’effectuer différentes opérations de gestion telles que répertorier et créer des abonnements aux événements et générer des clés. Event Grid utilise le contrôle d’accès en fonction du rôle (RBAC) d’Azure.

## <a name="operation-types"></a>Types d’opération

Event Grid prend en charge les actions suivantes :

* Microsoft.EventGrid/*/read
* Microsoft.EventGrid/*/write
* Microsoft.EventGrid/*/delete
* Microsoft.EventGrid/eventSubscriptions/getFullUrl/action
* Microsoft.EventGrid/topics/listKeys/action
* Microsoft.EventGrid/topics/regenerateKey/action

Les trois dernières opérations retournent des informations potentiellement confidentielles qui sont filtrées lors d’opérations de lecture normales. Il est recommandé de restreindre l’accès à ces opérations. 

## <a name="built-in-roles"></a>Rôles intégrés

Event Grid fournit deux rôles intégrés pour gérer les abonnements aux événements. Ceux-ci sont importants lors de l’implémentation de [domaines d’événements](event-domains.md) car ils fournissent aux utilisateurs les autorisations nécessaires pour s’abonner aux rubriques de votre domaine d’événements. Ces rôles sont centrés sur les abonnements aux événements et ne donnent pas accès aux actions telles que la création de rubriques.

Vous pouvez [affecter ces rôles à un utilisateur ou groupe](../role-based-access-control/quickstart-assign-role-user-portal.md).

**Contributeur EventGrid EventSubscription** : gère les opérations relatives aux abonnements Event Grid.

```json
[
  {
    "Description": "Lets you manage EventGrid event subscription operations.",
    "IsBuiltIn": true,
    "Id": "428e0ff05e574d9ca2212c70d0e0a443",
    "Name": "EventGrid EventSubscription Contributor",
    "IsServiceRole": false,
    "Permissions": [
      {
        "Actions": [
          "Microsoft.Authorization/*/read",
          "Microsoft.EventGrid/eventSubscriptions/*",
          "Microsoft.EventGrid/topicTypes/eventSubscriptions/read",
          "Microsoft.EventGrid/locations/eventSubscriptions/read",
          "Microsoft.EventGrid/locations/topicTypes/eventSubscriptions/read",
          "Microsoft.Insights/alertRules/*",
          "Microsoft.Resources/deployments/*",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Support/*"
        ],
        "NotActions": [],
        "DataActions": [],
        "NotDataActions": [],
        "Condition": null
      }
    ],
    "Scopes": [
      "/"
    ]
  }
]
```

**Lecteur EventGrid EventSubscription** : lit les abonnements Event Grid.

```json
[
  {
    "Description": "Lets you read EventGrid event subscriptions.",
    "IsBuiltIn": true,
    "Id": "2414bbcf64974faf8c65045460748405",
    "Name": "EventGrid EventSubscription Reader",
    "IsServiceRole": false,
    "Permissions": [
      {
        "Actions": [
          "Microsoft.Authorization/*/read",
          "Microsoft.EventGrid/eventSubscriptions/read",
          "Microsoft.EventGrid/topicTypes/eventSubscriptions/read",
          "Microsoft.EventGrid/locations/eventSubscriptions/read",
          "Microsoft.EventGrid/locations/topicTypes/eventSubscriptions/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read"
        ],
        "NotActions": [],
        "DataActions": [],
        "NotDataActions": []
       }
    ],
    "Scopes": [
      "/"
    ]
  }
]
```

## <a name="custom-roles"></a>Rôles personnalisés

Si vous avez besoin de spécifier des autorisations autres que les rôles intégrés, vous pouvez créer des rôles personnalisés.

Voici des exemples de définitions de rôle dans Event Grid permettant aux utilisateurs d’effectuer différentes actions. Ces rôles personnalisés sont différents des rôles intégrés car ils confèrent un accès qui va au-delà des abonnements aux événements.

**EventGridReadOnlyRole.json** : pour autoriser uniquement les opérations en lecture seule.

```json
{
  "Name": "Event grid read only role",
  "Id": "7C0B6B59-A278-4B62-BA19-411B70753856",
  "IsCustom": true,
  "Description": "Event grid read only role",
  "Actions": [
    "Microsoft.EventGrid/*/read"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<Subscription Id>"
  ]
}
```

**EventGridNoDeleteListKeysRole.json** : pour autoriser des actions de publication limitées, et interdire les actions de suppression.

```json
{
  "Name": "Event grid No Delete Listkeys role",
  "Id": "B9170838-5F9D-4103-A1DE-60496F7C9174",
  "IsCustom": true,
  "Description": "Event grid No Delete Listkeys role",
  "Actions": [
    "Microsoft.EventGrid/*/write",
    "Microsoft.EventGrid/eventSubscriptions/getFullUrl/action"
    "Microsoft.EventGrid/topics/listkeys/action",
    "Microsoft.EventGrid/topics/regenerateKey/action"
  ],
  "NotActions": [
    "Microsoft.EventGrid/*/delete"
  ],
  "AssignableScopes": [
    "/subscriptions/<Subscription id>"
  ]
}
```

**EventGridContributorRole.json**: pour autoriser toutes les actions dans Event Grid.

```json
{
  "Name": "Event grid contributor role",
  "Id": "4BA6FB33-2955-491B-A74F-53C9126C9514",
  "IsCustom": true,
  "Description": "Event grid contributor role",
  "Actions": [
    "Microsoft.EventGrid/*/write",
    "Microsoft.EventGrid/*/delete",
    "Microsoft.EventGrid/topics/listkeys/action",
    "Microsoft.EventGrid/topics/regenerateKey/action",
    "Microsoft.EventGrid/eventSubscriptions/getFullUrl/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/<Subscription id>"
  ]
}
```

Vous pouvez créer des rôles personnalisés avec [PowerShell](../role-based-access-control/custom-roles-powershell.md), [Azure CLI](../role-based-access-control/custom-roles-cli.md) et [REST](../role-based-access-control/custom-roles-rest.md).



### <a name="encryption-at-rest"></a>Chiffrement au repos

Tous les événements ou données écrits sur le disque par le service Event Grid sont chiffrés à l’aide d’une clé managée par Microsoft, ce qui garantit un chiffrement au repos. En outre, la durée maximale de conservation des événements ou données est de 24 heures, conformément à la [stratégie de nouvelles tentatives Event Grid](delivery-and-retry.md). Event Grid supprime automatiquement tous les événements ou données après 24 heures, ou la durée de vie de l’événement, selon la valeur la plus faible.

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Event Grid, consultez [Event Grid](overview.md)
