---
title: Docker pull du conteneur Analyse des sentiments
titleSuffix: Azure Cognitive Services
description: Commande Docker pull du conteneur Analyse des sentiments
services: cognitive-services
author: IEvangelist
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/12/2019
ms.author: dapine
ms.openlocfilehash: df00d469052fa30c3f2aaa5afe1881ef74587f9a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "70966661"
---
#### <a name="docker-pull-for-the-sentiment-analysis-container"></a>Docker pull du conteneur Analyse des sentiments

Utilisez la commande [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir de Microsoft Container Registry.

Pour obtenir une description complète des balises disponibles pour les conteneurs Analyse de texte, consultez le conteneur [Analyse des sentiments](https://go.microsoft.com/fwlink/?linkid=2018654) dans le Docker Hub.

```
docker pull mcr.microsoft.com/azure-cognitive-services/sentiment:latest
```
