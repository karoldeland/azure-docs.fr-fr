---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: d36d2be59010c47348a8e196b28d87e5b967868e
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "67176934"
---
### <a name="step-1-navigate-to-the-virtual-network-gateway"></a>Étape 1 : Accès à la passerelle de réseau virtuel

1. Dans le [Portail Azure](https://portal.azure.com), accédez à **Toutes les ressources**. 
2. Pour ouvrir le page Passerelle de réseau virtuel, accédez à la passerelle de réseau virtuel que vous souhaitez supprimer et cliquez dessus.

### <a name="step-2-delete-connections"></a>Étape 2 : Suppression des connexions

1. Dans la page de votre passerelle de réseau virtuel, cliquez sur **Connexions** pour afficher toutes les connexions à la passerelle.
2. Cliquez sur **'...'** sur la ligne du nom de la connexion, puis sélectionnez **Supprimer** dans la liste déroulante.
3. Cliquez sur **Oui** pour confirmer que vous souhaitez supprimer la connexion. Si vous avez plusieurs connexions, supprimez chaque connexion.

### <a name="step-3-delete-the-virtual-network-gateway"></a>Étape 3 : Supprimez la passerelle de réseau virtuel

À savoir : Si vous avez une configuration P2S sur ce réseau virtuel en plus de votre configuration S2S, la suppression de la passerelle de réseau virtuel déconnecte automatiquement tous les clients P2S sans avertissement.

1. Dans la page Passerelle de réseau virtuel, cliquez sur **Vue d’ensemble**.
2. Dans la page **Vue d’ensemble**, cliquez sur **Supprimer** pour supprimer la passerelle.
