---
title: Créer un réseau virtuel pour les applications multiniveaux - Exemple de script Azure CLI
description: Exemple de script Azure CLI - Créer un réseau pour les applications multiniveau.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: kumud
ms.openlocfilehash: 39a993324a476f0e1f1d17858ae126742e4f69f9
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/24/2020
ms.locfileid: "74083858"
---
# <a name="create-a-network-for-multi-tier-applications-script-sample"></a>Créer un réseau pour des exemples de script d’applications multiniveau

Cet exemple de script permet de créer un réseau virtuel avec des sous-réseaux frontaux et principaux. Le trafic vers le sous-réseau frontal est limité à HTTP et SSH, tandis que le trafic vers le sous-réseau principal est limité à MySQL, port 3306. Après avoir exécuté le script, vous obtenez deux machines virtuelles, une dans chaque sous-réseau sur laquelle que vous pouvez déployer le serveur web et le logiciel MySQL.

Vous pouvez exécuter le script à partir d’Azure [Cloud Shell](https://shell.azure.com/bash) ou à partir d’une installation Azure CLI locale. Si vous utilisez l’interface CLI localement, ce script nécessite que vous exécutiez la version 2.0.28 ou une version ultérieure. Pour trouver la version installée, exécutez `az --version`. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli). Si vous exécutez la CLI localement, vous devez également exécuter `az login` pour créer une connexion avec Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>Exemple de script


[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/virtual-network-multi-tier-application/virtual-network-multi-tier-application.sh  "Virtual network for multi-tier application")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement 

Exécutez la commande suivante pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées :

```azurecli
az group delete --name MyResourceGroup --yes
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes pour créer un groupe de ressources, un réseau virtuel et les groupes de sécurité réseau. Chaque commande du tableau suivant renvoie à une documentation spécifique :

| Commande | Notes |
|---|---|
| [az group create](/cli/azure/group) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az network vnet create](/cli/azure/network/vnet) | Crée un réseau virtuel et un sous-réseau frontal Azure. |
| [az network subnet create](/cli/azure/network/vnet/subnet) | Crée un sous-réseau principal. |
| [az network public-ip create](/cli/azure/network/public-ip) | Crée une adresse IP publique pour accéder à la machine virtuelle à partir d’Internet. |
| [az network nic create](/cli/azure/network/nic) | Crée des interfaces réseau virtuelles et les attache aux sous-réseaux frontaux et principaux du réseau virtuel. |
| [az network nsg create](/cli/azure/network/nsg) | Crée des groupes de sécurité réseau (NSG) associés aux sous-réseaux frontaux et principaux. |
| [az network nsg rule create](/cli/azure/network/nsg/rule) |Crée des règles NSG qui autorisent ou bloquent des ports spécifiques sur des sous-réseaux donnés. |
| [az vm create](/cli/azure/vm) | Crée des machines virtuelles et associe une carte d’interface réseau à chacune d’elles. Cette commande spécifie également l’image de machine virtuelle à utiliser ainsi que les informations d’identification d’administration. |
| [az group delete](/cli/azure/group) | Supprime un groupe de ressources, ainsi que toutes ses ressources. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

Vous trouverez des exemples supplémentaires de scripts CLI pour réseau virtuel dans [Exemples CLI pour réseau virtuel](../cli-samples.md).
