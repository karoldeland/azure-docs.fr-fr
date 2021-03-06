---
title: FAQ Azure Security Center – Collecte de données et agents
description: Forum aux questions sur la collecte de données, les agents et les espaces de travail pour Azure Security Center, produit qui vous aide à prévenir, détecter et répondre aux menaces
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 8317a13b9ef87679836f55627268deefa4500dce
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79225313"
---
# <a name="faq---questions-about-data-collection-agents-and-workspaces"></a>FAQ – Question relatives à la collecte de données, aux agents et aux espaces de travail

Azure Security Center collecte des données à partir de vos machines virtuelles Azure, groupes de machines virtuelles identiques, conteneurs IaaS et ordinateurs autres qu’Azure (y compris les ordinateurs locaux) pour surveiller les menaces et vulnérabilités de sécurité. Les données sont collectées à l’aide de Microsoft Monitoring Agent, qui lit divers journaux d’événements et configurations liées à la sécurité de la machine et copie les données dans votre espace de travail à des fins d’analyse.


## <a name="am-i-billed-for-azure-monitor-logs-on-the-workspaces-created-by-security-center"></a>Est-ce que je suis facturé pour Journaux Azure Monitor sur des espaces de travail créés par Security Center ?

Non. Les espaces de travail créés par Security Center, bien qu’ils soient configurés pour une facturation Journaux Azure Monitor par nœud, n’entraînent pas de frais de journaux Azure Monitor. La facturation Security Center est toujours basée sur la stratégie de sécurité Security Center et les solutions installées sur l’espace de travail :

- **Niveau Gratuit** : Security Center active la solution « SecurityCenterFree » sur l’espace de travail par défaut. Vous n’êtes pas facturé pour le niveau Gratuit.

- **Niveau Standard** : Security Center active la solution « Security » sur l’espace de travail par défaut.

Pour plus d’informations sur la tarification, consultez la page de [tarification de Security Center](https://azure.microsoft.com/pricing/details/security-center/).

> [!NOTE]
> Le niveau tarifaire Log Analytics des espaces de travail créés par Security Center n’affecte pas la facturation Security Center.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]


## <a name="what-qualifies-a-vm-for-automatic-provisioning-of-the-microsoft-monitoring-agent-installation"></a>Qu’est-ce qui rend une machine virtuelle apte pour le provisionnement automatique de l’installation de Microsoft Monitoring Agent ?

Les machines virtuelles Windows ou Linux IaaS sont retenues dans les cas suivants :

- L’extension Microsoft Monitoring Agent n’est pas actuellement installée sur la machine virtuelle.
- La machine virtuelle est en cours d’exécution.
- L’[agent de machine virtuelle Azure](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows) pour Windows ou Linux est installé.
- La machine virtuelle n’est pas utilisée comme pare-feu d’applications web ou comme pare-feu de nouvelle génération.


## <a name="can-i-delete-the-default-workspaces-created-by-security-center"></a>Puis-je supprimer les espaces de travail par défaut créés par Security Center ?

**Il n’est pas recommandé de supprimer l’espace de travail par défaut.** Security Center utilise les espaces de travail par défaut pour stocker les données de sécurité de vos machines virtuelles. Si vous supprimez un espace de travail, Security Center n’est pas en mesure pas collecter ces données, et certaines recommandations de sécurité et alertes ne sont pas disponibles.

Pour effectuer une récupération, supprimez Microsoft Monitoring Agent sur les machines virtuelles connectées à l’espace de travail supprimé. Security Center réinstalle l’agent et crée des nouveaux espaces de travail par défaut.

## <a name="how-can-i-use-my-existing-log-analytics-workspace"></a>Comment puis-je utiliser mon espace de travail Log Analytics existant ?

Vous pouvez sélectionner un espace de travail Log Analytics existant pour stocker les données collectées par Security Center. Pour utiliser votre espace de travail Log Analytics existant :

- L’espace de travail doit être associé à votre abonnement Azure sélectionné.
- Au minimum, vous devez avoir des autorisations de lecture pour accéder à l’espace de travail.

Pour sélectionner un espace de travail Log Analytics existant :

1. Dans **Stratégie de sécurité : collecte de données**, sélectionnez **Use another workspace** (Utiliser un autre espace de travail).

    ![Utiliser un autre espace de travail][4]

1. Dans le menu déroulant, sélectionnez un espace de travail pour stocker les données collectées.

    > [!NOTE]
    > Dans le menu déroulant, seuls les espaces de travail auxquels vous avez accès et se trouvant dans votre abonnement Azure sont affichés.

1. Sélectionnez **Enregistrer**. Vous serez invité à reconfigurer les machines virtuelles surveillées.

    - Sélectionnez **Non** si vous souhaitez que les nouveaux paramètres de l’espace de travail **s’appliquent uniquement aux nouvelles machines virtuelles**. Les nouveaux paramètres de l’espace de travail s’appliquent uniquement aux nouvelles installations d’agent ; machines virtuelles nouvellement détectées qui n’ont pas Microsoft Monitoring Agent installé.
    - Sélectionnez **Oui** si vous souhaitez que les nouveaux paramètres de l’espace de travail **s’appliquent à toutes les machines virtuelles**. En outre, chaque machine virtuelle connectée à un espace de travail créé par Security Center est reconnectée au nouvel espace de travail cible.

    > [!NOTE]
    > Si vous sélectionnez **Oui**, ne supprimez pas les espaces de travail créés par Security Center tant que toutes les machines virtuelles n’ont pas été reconnectées au nouvel espace de travail cible. Cette opération échoue si un espace de travail est supprimé trop tôt.

    - Pour annuler l’opération, sélectionnez **Annuler**.

## <a name="what-if-the-microsoft-monitoring-agent-was-already-installed-as-an-extension-on-the-vm"></a>Que se passe-t-il si l’agent Microsoft Monitoring Agent est déjà installé en tant qu’extension sur la machine virtuelle ?<a name="mmaextensioninstalled"></a>

Lorsque l’Agent Monitoring est installé en tant qu’extension, la configuration de l’extension permet de rendre compte à un seul espace de travail. Security Center n’écrase pas les connexions existantes des espaces de travail utilisateur. Security Center stocke les données de sécurité à partir d’une machine virtuelle dans un espace de travail qui est déjà connecté, sous réserve que la solution « Security » ou « SecurityCenterFree » y soit installée. Security Center peut mettre à niveau la version de l'extension vers la dernière version lors de ce processus.

Pour plus d’informations, voir [Approvisionnement automatique en cas d’installation d’un agent préexistant](security-center-enable-data-collection.md#preexisting).



## <a name="what-if-a-microsoft-monitoring-agent-is-directly-installed-on-the-machine-but-not-as-an-extension-direct-agent"></a>Que se passe-t-il si Microsoft Monitoring Agent est installé directement sur la machine, mais pas en tant qu’extension (Direct Agent) ?<a name="directagentinstalled"></a>

Si l’agent Microsoft Monitoring Agent est installé directement sur la machine virtuelle (pas en tant qu’extension Azure), Security Center installe l’extension Microsoft Monitoring Agent, et peut mettre à niveau Microsoft Monitoring Agent vers la dernière version.

L’agent installé continue de rendre compte à ses espaces de travail déjà configurés et à l’espace de travail configuré dans Security Center (multihébergement pris en charge sur les machines Windows).

Si l’espace de travail configuré est un espace de travail utilisateur (pas un espace de travail par défaut de Security Center), vous devez y installer la solution « Security/SecurityCenterFree » pour que Security Center démarre le traitement des événements à partir des machines virtuelles et ordinateurs rendant compte à cet espace de travail.

Pour les machines Linux, le multihébergement d’agent n’est pas encore pris en charge, par conséquent, si une installation existante de l’agent est détectée, l’approvisionnement automatique n’a pas lieu et la configuration de la machine n’est pas modifiée.

Pour des machines existantes sur des abonnements intégrés à Security Center avant le 17 mars 2019, lors de la détection d’un agent existant, l’extension Microsoft Monitoring Agent n’est pas installée et la machine n’est pas affectée. Pour ces machines, consultez la recommandation « Résoudre les problèmes d’intégrité de l’agent d’analyse sur vos machines » pour résoudre les problèmes d’installation de l’agent sur ces machines.

Pour plus d’informations, consultez la prochaine section, [Que se passe-t-il si l’agent direct System Center Operations Manager ou OMS est déjà installé sur ma machine virtuelle ?](#scomomsinstalled)

## <a name="what-if-a-system-center-operations-manager-agent-is-already-installed-on-my-vm"></a>Que se passe-t-il si un agent Operations Manager est déjà installé sur ma machine virtuelle ?<a name="scomomsinstalled"></a>

Security Center installe l’extension Microsoft Monitoring Agent parallèlement à l’agent System Center Operations Manager existant. L’agent existant continue de rendre compte normalement au serveur System Center Operations Manager. Notez que l’agent Operations Manager et Microsoft Monitoring Agent partagent des bibliothèques runtime communes, qui sont mises à jour vers la dernière version lors de ce processus. Remarque : si la version 2012 de l’agent Operations Manager est installée, n’activez pas l’approvisionnement automatique (les fonctionnalités de facilité de gestion peuvent être perdues si la version du serveur Operations Manager date également de 2012).


## <a name="what-is-the-impact-of-removing-these-extensions"></a>Quel est l’impact de la suppression de ces extensions ?

Si vous supprimez l’extension Microsoft Monitoring, Security Center n’est pas en mesure de collecter des données de sécurité sur la machine virtuelle et certaines recommandations de sécurité et les alertes ne sont pas disponibles. Sous 24 heures, Security Center détermine que l’extension est absente de la machine virtuelle et la réinstalle.


## <a name="how-do-i-stop-the-automatic-agent-installation-and-workspace-creation"></a>Comment empêcher l’installation automatique de l’agent et la création de l’espace de travail ?

Vous pouvez désactiver l’approvisionnement automatique pour vos abonnements dans la stratégie de sécurité, mais ce n’est pas recommandé. La désactivation de l’approvisionnement automatique a pour effet de limiter les alertes et recommandations de Security Center. Pour désactiver l’approvisionnement automatique :

1. Si votre abonnement est configuré pour le niveau Standard, ouvrez la stratégie de sécurité de cet abonnement, puis sélectionnez le niveau **Gratuit**.

   ![Niveau tarifaire][1]

1. Ensuite, désactivez l’approvisionnement automatique en sélectionnant **Non** sur le page **Stratégie de sécurité : collecte de données**.
   ![Collecte de données][2]


## <a name="should-i-opt-out-of-the-automatic-agent-installation-and-workspace-creation"></a>Dois-je refuser l’installation automatique de l’agent et la création de l’espace de travail ?

> [!NOTE]
> Veillez à consulter les sections [Quelles sont les implications d’un refus ?](#what-are-the-implications-of-opting-out-of-automatic-provisioning) et [Étapes recommandées en cas de refus](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning) si vous choisissez de refuser le provisionnement automatique.

Vous pouvez souhaiter refuser le provisionnement automatique si les points suivants s’appliquent à vous :

- L’installation automatique de l’agent par Security Center s’applique à l’ensemble de l’abonnement. Vous ne pouvez pas appliquer l’installation automatique à un sous-ensemble de machines virtuelles. Si des machines virtuelles critiques ne peuvent pas être installées avec Microsoft Monitoring Agent, vous devez refuser le provisionnement automatique.
- L’installation de l’extension Microsoft Monitoring Agent (MMA) met à jour la version de l’agent. Cela s’applique à un agent direct et un agent System Center Operations Manager (dans le dernier cas, les bibliothèques Operations Manager et MMA partagent des bibliothèques runtime communes, qui sont mises à jour dans le processus). Si l’agent Operations Manager installé est en version 2012 et qu’il est mis à niveau, les fonctionnalités de facilité de gestion peuvent être perdues quand le serveur Operations Manager est également en version 2012. Vous pouvez refuser le provisionnement automatique si l’agent Operations Manager installé est en version 2012.
- Si vous disposez d’un espace de travail personnalisé externe à l’abonnement (un espace de travail centralisé), vous devez refuser l’approvisionnement automatique. Vous pouvez installer manuellement l’extension Microsoft Monitoring Agent et la connecter à votre espace de travail sans que Security Center ne remplace la connexion.
- Si vous souhaitez éviter de créer plusieurs espaces de travail par abonnement et que vous disposez de votre propre espace de travail personnalisé dans l’abonnement, deux options s’offrent à vous :

   1. Vous pouvez refuser le provisionnement automatique. Après la migration, définissez les paramètres d’espace de travail par défaut comme décrit dans [Comment puis-je utiliser mon espace de travail Log Analytics existant ?](#how-can-i-use-my-existing-log-analytics-workspace)

   1. Vous pouvez aussi autoriser l’exécution de la migration, l’installation de l’agent Microsoft Monitoring Agent sur les machines virtuelles et la connexion des machines virtuelles à l’espace de travail créé. Sélectionnez ensuite votre propre espace de travail personnalisé en définissant le paramètre d’espace de travail par défaut avec l’activation de la reconfiguration des agents déjà installés. Pour plus d’informations, consultez [Comment puis-je utiliser mon espace de travail Log Analytics existant ?](#how-can-i-use-my-existing-log-analytics-workspace)


## <a name="what-are-the-implications-of-opting-out-of-automatic-provisioning"></a>Quelles sont les implications d’un refus du provisionnement automatique ?

Une fois la migration terminée, Security Center n’est pas en mesure de collecter des données de sécurité sur la machine virtuelle, et certaines recommandations de sécurité et alertes ne sont pas disponibles. En cas de refus, installez Microsoft Monitoring Agent manuellement. Consultez les [étapes recommandées en cas de refus](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning).


## <a name="what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning"></a>Quelles sont les étapes recommandées en cas de refus du provisionnement automatique ?

Installez manuellement l’extension Microsoft Monitoring Agent pour que Security Center puisse collecter des données de sécurité sur vos machines virtuelles et fournir des suggestions et des alertes. Consultez [Installation de l’agent pour les machines virtuelles Windows](../virtual-machines/extensions/oms-windows.md) ou [Installation de l’agent pour les machines virtuelles Linux](../virtual-machines/extensions/oms-linux.md) pour obtenir des instructions sur l’installation.

Vous pouvez connecter l’agent à n’importe quel espace de travail personnalisé existant ou à l’espace de travail créé par Security Center. Si les solutions « Security » ou « SecurityCenterFree » ne sont pas activées pour un espace de travail personnalisé, vous devez appliquer une solution. Pour ce faire, sélectionnez l’abonnement ou l’espace de travail personnalisé, puis appliquez un niveau tarifaire via la page **Stratégie de sécurité – Niveau tarifaire**.

   ![Niveau tarifaire][1]

Security Center active la solution appropriée sur l’espace de travail en fonction du niveau tarifaire sélectionné.


## <a name="how-do-i-remove-oms-extensions-installed-by-security-center"></a>Comment supprimer des extensions OMS installées par Security Center ?<a name="remove-oms"></a>

Vous pouvez supprimer manuellement l’agent Microsoft Monitoring Agent. Ce n’est pas recommandé car cela limite les recommandations de Security Center et les alertes.

> [!NOTE]
> Si la collecte de données est activée, Security Center réinstalle l’agent après l’avoir supprimé.  Vous devez désactiver la collecte de données avant de supprimer manuellement l’agent. Consultez Comment empêcher l’installation automatique de l’agent et la création de l’espace de travail ? pour obtenir des instructions sur la désactivation de la collecte de données.

Pour supprimer manuellement l’agent :

1.  Dans le portail, ouvrez **Log Analytics**.

1.  Sur la page Log Analytics, sélectionnez un espace de travail :

1.  Sélectionnez les machines virtuelles que vous ne souhaitez pas surveiller, puis choisissez **Déconnecter**.

   ![Supprimer l’agent][3]

> [!NOTE]
> Si une machine virtuelle Linux possède déjà un agent OMS qui n’est pas un agent d’extension, la suppression de l’extension a pour effet de supprimer également l’agent. Vous devrez alors le réinstaller.


## <a name="how-do-i-disable-data-collection"></a>Comment désactiver la collecte des données ?

L’approvisionnement automatique est désactivé par défaut. Vous pouvez désactiver l’approvisionnement automatique à partir des ressources à tout moment en désactivant ce paramètre dans la stratégie de sécurité. L’approvisionnement automatique est fortement recommandé si vous souhaitez obtenir des alertes de sécurité et des recommandations sur les mises à jour système, les vulnérabilités du système d’exploitation et la protection du point de terminaison.

Pour désactiver la collecte des données, [connectez-vous au Portail Azure](https://portal.azure.com), sélectionnez **Parcourir**, **Centre de sécurité**, puis **Sélectionner une stratégie**. Sélectionnez l’abonnement pour lequel vous souhaitez désactiver l’approvisionnement automatique. Lorsque vous sélectionnez un abonnement, **Stratégie de sécurité - Collecte de données** s’ouvre. Sous **Auto provisioning** (Approvisionnement automatique), sélectionnez **Off** (Désactivé).


## <a name="how-do-i-enable-data-collection"></a>Comment activer la collecte des données ?

Vous pouvez activer la collecte des données pour votre abonnement Azure dans la stratégie de sécurité. Pour activer la collecte de données. [Connectez-vous au portail Azure](https://portal.azure.com), sélectionnez **Parcourir**, **Centre de sécurité**, puis **Stratégie de sécurité**. Sélectionnez l’abonnement pour lequel vous souhaitez activer l’approvisionnement automatique. Lorsque vous sélectionnez un abonnement, **Stratégie de sécurité - Collecte de données** s’ouvre. Sous **Auto provisioning** (Approvisionnement automatique), sélectionnez **On** (Activé).


## <a name="what-happens-when-data-collection-is-enabled"></a>Que se passe-t-il quand la collecte des données est activée ?

Lorsque l’approvisionnement automatique est activé, Security Center approvisionne Microsoft Monitoring Agent pour toutes les machines virtuelles Azure prises en charge et toutes celles nouvellement créées. L’approvisionnement automatique est recommandé. Toutefois, l’installation manuelle de l’agent est également disponible. [Découvrez comment installer l’extension Microsoft Monitoring Agent](../azure-monitor/learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension) 

L’agent active l’événement de création de processus 4688 et le champ *CommandLine* à l’intérieur de l’événement 4688. Les nouveaux processus créés sur la machine virtuelle sont enregistrés par le journal des événements et analysés par les services de détection Security Center. Pour plus d’informations sur les détails enregistrés pour chaque nouveau processus, voir [Description des champs 4688](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688#fields). L’agent collecte également les événements 4688 créés sur la machine virtuelle et les stocke dans la recherche.

L’agent permet également d’activer la collecte de données pour les [Contrôles d’application adaptatifs](security-center-adaptive-application.md), Security Center configure une stratégie AppLocker locale en mode Audit pour autoriser toutes les applications. Cette stratégie amène AppLocker à générer des événements qui sont ensuite recueillis et exploités par Security Center. Il est important de noter que cette stratégie ne sera configurée sur aucun ordinateur sur lequel une stratégie AppLocker est déjà configurée. 

Lorsque Security Center détecte une activité suspecte sur la machine virtuelle, le client est averti par e-mail si [des informations de contact de sécurité](security-center-provide-security-contact-details.md) ont été fournies. Une alerte est également visible dans le tableau de bord d’alertes de sécurité de Security Center.


## <a name="will-security-center-work-using-an-oms-gateway"></a>Security Center fonctionne-t-il avec une passerelle OMS ?

Oui. Azure Security Center utilise Azure Monitor pour collecter des données à partir de machines virtuelles et de serveurs Azure, à l’aide de Microsoft Monitoring Agent.
Pour collecter les données, chaque machine virtuelle et chaque serveur doivent se connecter à Internet à l’aide du protocole HTTPS. Il peut s’agir d’une connexion directe, d’une connexion obtenue par un proxy ou bien établie via la [passerelle OMS](../azure-monitor/platform/gateway.md).


## <a name="does-the-monitoring-agent-impact-the-performance-of-my-servers"></a>Est-ce que l’agent d’analyse a un impact sur les performances de mes serveurs ?

L’agent utilise une quantité minime de ressources système et n’a donc qu’un faible impact sur les performances. Pour en savoir plus sur l’impact sur les performances, l’agent et l’extension, consultez le [Guide de planification et de fonctionnement](security-center-planning-and-operations-guide.md#data-collection-and-storage).


## <a name="where-is-my-data-stored"></a>Où sont stockées mes données ?

Les données collectées à partir de cet agent sont stockées dans un espace de travail Log Analytics existant associé à votre abonnement Azure ou dans un nouvel espace de travail. Pour plus d’informations, consultez [Sécurité des données](security-center-data-security.md).


<!--Image references-->
[1]: ./media/security-center-platform-migration-faq/pricing-tier.png
[2]: ./media/security-center-platform-migration-faq/data-collection.png
[3]: ./media/security-center-platform-migration-faq/remove-the-agent.png
[4]: ./media/security-center-platform-migration-faq/use-another-workspace.png