---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.custom: seo-python-october2019
ms.openlocfilehash: 23b93f507ef6abe19a0202b28afa31d28490b2b3
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80151221"
---
## <a name="what-is-queue-storage"></a>Présentation du stockage File d’attente

Les files d’attente de stockage Azure sont un service permettant de stocker un grand nombre de messages accessibles depuis n’importe où dans le monde via des appels authentifiés avec HTTP ou HTTPS. Un simple message de file d’attente peut avoir une taille de 64 Ko et une file d’attente peut contenir des millions de messages, jusqu’à la limite de capacité totale d’un compte de stockage. Le stockage File d’attente est couramment utilisé pour créer un backlog de travail à traiter de façon asynchrone.

## <a name="queue-service-concepts"></a>Concepts du service File d’attente

Le service de File d’attente Azure contient les composants suivants :

![Composants du service File d’attente Azure](./media/storage-queue-concepts-include/azure-queue-service-components.png)

* **Format d’URL** : les files d’attente sont adressables à l’aide du format d’URL suivant : http://`<storage account>`.queue.core.windows.net/`<queue>`
  
    L'URL suivante désigne une file d'attente du schéma :  
  
    `http://myaccount.queue.core.windows.net/incoming-orders`

* **Compte de stockage :** Tous les accès à Azure Storage passent par un compte de stockage. Pour plus d’informations sur les comptes de stockage, consultez [Vue d’ensemble des comptes de stockage](../articles/storage/common/storage-account-overview.md).
* **File d’attente :** Une file d’attente contient un ensemble de messages. Tous les messages doivent être dans une file d’attente. Notez que le nom de la file d’attente doit être en minuscules. Pour plus d'informations sur l’affectation de noms à des files d’attente, consultez [Affectation de noms pour les files d'attente et les métadonnées](https://msdn.microsoft.com/library/azure/dd179349.aspx).
* **Message :** Message dans n’importe quel format d’une taille maximale de 64 Ko. La durée maximale pendant laquelle un message peut rester dans la file d’attente est de 7 jours. Pour les versions du 29 juillet 2017 ou ultérieures, la durée de vie maximale peut être n’importe quel nombre positif, ou -1 indiquant que le message n’expire pas. Si ce paramètre est omis, la valeur par défaut de la durée de vie est de sept jours.

