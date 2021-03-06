---
title: Fichier Include
description: Fichier Include
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/14/2019
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 4e699707db02de07f3d1ebb7d1fa8d0575a10aa3
ms.sourcegitcommit: c2065e6f0ee0919d36554116432241760de43ec8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/26/2020
ms.locfileid: "67836920"
---
Vous pouvez déployer le protocole FTP et Git local sur une application web Azure en faisant appel à un *utilisateur de déploiement*. Une fois que vous avez créé votre utilisateur de déploiement, vous pouvez l’utiliser pour tous vos déploiements Azure. Votre nom d’utilisateur et votre mot de passe de déploiement au niveau du compte sont différents de vos informations d’identification de l’abonnement Azure. 

Pour configurer l’utilisateur de déploiement, exécutez la commande [az webapp deployment user set](/cli/azure/webapp/deployment/user?view=azure-cli-latest#az-webapp-deployment-user-set) dans Azure Cloud Shell. Remplacez \<username> et \<password> par un nom d’utilisateur et un mot de passe de déploiement. 

- Le nom d’utilisateur doit être unique dans Azure et, pour les push Git locaux, ne doit pas contenir le symbole « @ ». 
- Le mot de passe doit comporter au moins huit caractères et inclure deux des trois éléments suivants : lettres, chiffres et symboles. 

```azurecli-interactive
az webapp deployment user set --user-name <username> --password <password>
```

La sortie JSON affiche le mot de passe comme étant `null`. Si vous obtenez une erreur `'Conflict'. Details: 409`, modifiez le nom d’utilisateur. Si vous obtenez une erreur `'Bad Request'. Details: 400`, utilisez un mot de passe plus fort. 

Enregistrez le nom d’utilisateur et le mot de passe à utiliser pour déployer vos applications web.
