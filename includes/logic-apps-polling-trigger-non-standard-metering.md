---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 11/09/2018
ms.openlocfilehash: 89c2467843d7abc7c005804fd5263fe3beb668b6
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "74793419"
---
Pour réaliser des estimations de coûts d’utilisation plus précises, ne basez pas vos calculs juste sur l’intervalle d’interrogation, mais prenez plutôt en compte le nombre possible de messages ou d’événements qui pourraient arriver chaque jour. Lorsqu’un événement ou un message répond aux critères de déclencheurs, plusieurs déclencheurs essaient immédiatement de lire tous les autres événements ou messages en attente qui répondent aux critères. Ce comportement signifie que le déclencheur se lance selon le nombre d’événements ou de messages en attente qualifiés pour le démarrage de workflows, même si vous sélectionnez un intervalle d’interrogation plus étendue. Les déclencheurs qui adoptent ce comportement incluent Azure Service Bus et Azure Event Hub.

Par exemple, imaginons que vous configuriez un déclencheur qui vérifie un point de terminaison par jour. Lorsqu’il vérifie le point de terminaison et trouve 15 événements qui répondent aux critères, il se lance et exécute le workflow correspondant 15 fois. Logic Apps mesure toutes les actions réalisées par ces 15 workflows, dont les requêtes du déclencheur. Pour calculer les coûts éventuels, essayez la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/).