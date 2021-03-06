---
title: Résolution des problèmes relatifs aux comptes Automation
description: Découvrez comment détecter et résoudre les problèmes liés à un compte Azure.
services: automation
author: mgoedtel
ms.author: magoedte
ms.date: 03/24/2020
ms.topic: conceptual
ms.service: automation
manager: carmonm
ms.openlocfilehash: c66b1728144b8517f6ac444059b3a8def956c6e1
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80299197"
---
# <a name="automation-account-troubleshooting"></a>Résolution des problèmes relatifs aux comptes Automation

Cet article traite des solutions aux problèmes que vous pourriez rencontrer en utilisant un compte Automation. Les sections suivantes mettent en évidence des messages d’erreur spécifiques et des résolutions possibles pour chacun d’eux. Pour des informations générales sur les comptes Automation, voir [Créer un compte Azure](../automation-quickstart-create-account.md).

## <a name="scenario-unable-to-register-automation-resource-provider-for-subscriptions"></a><a name="rp-register"></a>Scénario : Impossible d’inscrire le fournisseur de ressources Automation pour les abonnements

### <a name="issue"></a>Problème

Lorsque vous utilisez des solutions de gestion dans votre compte Automation, vous rencontrez l’erreur suivante :

```error
Error details: Unable to register Automation Resource Provider for subscriptions:
```

### <a name="cause"></a>Cause

Le fournisseur de ressources Automation n’est pas inscrit dans l’abonnement.

### <a name="resolution"></a>Résolution

Pour inscrire le fournisseur de ressources Automation, effectuez les opérations suivantes dans le portail Azure :

1. À partir de votre navigateur, accédez au [portail Azure](https://portal.azure.com).

2. Accédez à **Abonnements**, puis sélectionnez votre abonnement dans la page Abonnements.   

3. Sous **Paramètres**, sélectionnez **Fournisseurs de ressources**.

4. Dans la liste des fournisseurs de ressources, vérifiez que le fournisseur de ressources **Microsoft.Automation** est inscrit.

5. S’il ne l’est pas, inscrivez le **fournisseur Microsoft.Automation** en suivant les étapes de la section [Résoudre les erreurs d’inscription du fournisseur de ressources](/azure/azure-resource-manager/resource-manager-register-provider-errors).

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou que vous ne pouvez pas le résoudre, utilisez un des canaux suivants pour obtenir une l’aide supplémentaire :

* Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
* Connectez-vous avec [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client en connectant la communauté Azure aux ressources appropriées (réponses, support et experts).
* Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.