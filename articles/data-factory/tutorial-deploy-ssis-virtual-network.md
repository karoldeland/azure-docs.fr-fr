---
title: Didacticiel pour configurer un runtime d’intégration Azure-SSIS pour joindre un réseau virtuel
description: Découvrez comment joindre un runtime d’intégration Azure-SSIS à un réseau virtuel Azure.
services: data-factory
documentationcenter: ''
author: chugugrace
ms.author: chugu
ms.reviewer: ''
manager: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 1/10/2020
ms.openlocfilehash: 065610a9de4898d012cef8a16849c09a81f0774c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76841095"
---
# <a name="configure-an-azure-sql-server-integration-services-ssis-integration-runtime-ir-to-join-a-virtual-network"></a>Configurer un runtime d’intégration (IR) Azure-SQL Server Integration Services (SSIS) pour joindre un réseau virtuel

Ce didacticiel décrit les étapes de base de l’utilisation du Portail Azure pour configurer un runtime d’intégration (IR) Azure-SQL Server Integration Services (SSIS) afin de joindre un réseau virtuel.

Procédez comme suit :

- Configurez un réseau virtuel.
- Joignez Azure-SSIS IR à un réseau virtuel à partir du portail Azure Data Factory.

## <a name="prerequisites"></a>Conditions préalables requises

- **Runtime d’intégration Azure SSIS**. Si vous n’avez pas de runtime d’intégration Azure-SSIS, [approvisionnez un runtime d’intégration Azure-SSIS dans Azure Data Factory](tutorial-deploy-ssis-packages-azure.md) avant de commencer.

- **Autorisation utilisateur**. L’utilisateur qui crée Azure-SSIS IR doit disposer de l’[attribution de rôle](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal#list-role-assignments-for-a-user-at-a-scope) au moins sur la ressource Azure Data Factory avec l’une des options ci-dessous :

    - Utilisez le rôle Contributeur de réseaux intégré. Ce rôle inclut l’autorisation _Microsoft.Network/\*_ , mais dispose d’une étendue plus large que nécessaire.
    - Créez un rôle personnalisé qui inclut uniquement l’autorisation _Microsoft.Network/virtualNetworks/\*/join/action_ nécessaire. Si vous voulez également apporter vos propres adresses IP publiques pour Azure-SSIS IR tout en le joignant à un réseau virtuel Azure Resource Manager, incluez également l’autorisation _Microsoft.Network/publicIPAddresses/*/join/action_ dans le rôle.

- **Réseau virtuel**.

    - Si vous n’avez pas de réseau virtuel, [créez un réseau virtuel à l’aide du Portail Azure](https://docs.microsoft.com/azure/virtual-network/quick-create-portal).

    - Vérifiez que le groupe de ressources du réseau virtuel peut créer et supprimer certaines ressources réseau Azure.
    
        Le runtime d’intégration Azure-SSIS doit créer certaines ressources réseau sous le même groupe de ressources que le réseau virtuel. Ces ressources incluent :
        - Un équilibreur de charge Azure, avec le nom *\<<GUID>-azurebatch-cloudserviceloadbalancer*
        - Un groupe de sécurité réseau, avec le nom *\<GUID>-azurebatch-cloudservicenetworksecuritygroup
        - Une adresse IP publique Azure, avec le nom -azurebatch-cloudservicepublicip
    
        Ces ressources sont créées au démarrage de votre Azure-SSIS IR. Elles sont supprimées lorsqu’il est arrêté. Pour éviter de bloquer l’arrêt du runtime d’intégration Azure-SSIS IR, ne réutilisez pas ces ressources réseau dans vos autres ressources.

    - Vérifiez qu’il n’existe aucun [verrou de ressource](https://docs.microsoft.com/azure/azure-resource-manager/management/lock-resources) sur le groupe de ressources/abonnement auquel appartient le réseau virtuel. Si vous configurez un verrou en lecture seule ou de suppression, le démarrage et l’arrêt du runtime d’intégration Azure-SSIS IR échoueront ou celui-ci ne répondra plus.

    - Vérifiez qu’aucune stratégie Azure n’empêche la création des ressources suivantes sous le groupe de ressources/abonnement auquel appartient le réseau virtuel :
        - Microsoft.Network/LoadBalancers
        - Microsoft.Network/NetworkSecurityGroups

- Les scénarios de **configuration réseau** ci-dessous ne sont pas abordés dans ce didacticiel :
    - Si vous apportez vos propres adresses IP publiques pour Azure-SSIS IR.
    - Si vous utilisez votre propre serveur DNS (Domain Name System).
    - Si vous utilisez un groupe de sécurité réseau sur le sous-réseau.
    - Si vous utilisez Azure ExpressRoute ou une route définie par l’utilisateur (UDR).
    - Si vous utilisez un Azure-SSIS IR personnalisé.
    
    Pour plus d’informations, consultez [Configuration du réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md#virtual-network-configuration).

## <a name="configure-a-virtual-network"></a>Configurer un réseau virtuel

Utilisez le portail Azure pour configurer un réseau virtuel avant de tenter d’y joindre une instance Azure-SSIS IR.

1. Démarrez Microsoft Edge ou Google Chrome. Actuellement, seuls les navigateurs web prennent en charge l’interface utilisateur.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Plus de services**. Filtrez et sélectionnez **Réseaux virtuels**.

1. Filtrez et sélectionnez votre réseau virtuel dans la liste.

1. Dans la page **Réseau virtuel**, sélectionnez **Propriétés**.

1. Sélectionnez le bouton Copier au niveau de **ID DE RESSOURCE** pour copier l’ID de ressource du réseau virtuel dans le Presse-papiers. Enregistrez dans OneNote ou un fichier l’ID se trouvant dans le Presse-papiers.

1. Dans le menu de gauche, sélectionnez **Sous-réseaux**.

    - Vérifiez que le sous-réseau sélectionné dispose de suffisamment d’espace d’adressage pour le runtime d’intégration Azure-SSIS IR. Les adresses IP disponibles doivent représenter au moins le double du nombre de nœuds de runtime d’intégration. Azure réserve des adresses IP dans chaque sous-réseau. Ces adresses ne peuvent pas être utilisées. Les première et dernière adresse IP des sous-réseaux sont réservées à la conformité du protocole, et trois adresses supplémentaires sont utilisées pour les services Azure. Pour plus d’informations, consultez [Existe-t-il des restrictions sur l’utilisation des adresses IP au sein de ces sous-réseaux ?](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)
    - Ne sélectionnez pas le GatewaySubnet pour déployer un runtime d’intégration Azure-SSIS IR. Il est réservé aux passerelles de réseau virtuel.
    - N’utilisez pas un sous-réseau qui est exclusivement occupé par d’autres services Azure (par exemple, instance managée SQL Database, App Service, etc.).

1. Vérifiez que le fournisseur Azure Batch est bien inscrit dans l’abonnement Azure qui contient le réseau virtuel. Si ce n’est pas le cas, inscrivez le fournisseur Azure Batch. Si vous possédez déjà un compte Azure Batch dans votre abonnement, ce dernier est inscrit pour Azure Batch. (Si vous créez le runtime d’intégration Azure SSIS dans le portail Data Factory, le fournisseur Azure Batch est inscrit automatiquement pour vous.)

   1. Dans le menu de gauche du Portail Azure, sélectionnez **Abonnements**.

   1. Sélectionnez votre abonnement.

   1. À gauche, sélectionnez **Fournisseurs de ressources**, puis confirmez que **Microsoft.Batch** est un fournisseur inscrit.

   ![Confirmation de l’état « Inscrit »](media/join-azure-ssis-integration-runtime-virtual-network/batch-registered-confirmation.png)

   Si **Microsoft.Batch** ne figure pas dans la liste, il vous faut l’inscrire. Pour cela, [créez un compte Azure Batch vide](../batch/batch-account-create-portal.md) dans votre abonnement. Vous pourrez le supprimer par la suite.

## <a name="join-the-azure-ssis-ir-to-a-virtual-network"></a>Joindre le runtime d’intégration Azure-SSIS à un réseau virtuel

Après avoir configuré votre réseau virtuel Azure Resource Manager ou votre réseau virtuel classique, vous pouvez joindre le runtime d’intégration Azure-SSIS IR au réseau virtuel :

1. Démarrez Microsoft Edge ou Google Chrome. Actuellement, seuls les navigateurs web prennent en charge l’interface utilisateur.

1. Dans le menu de gauche du [Portail Azure](https://portal.azure.com), sélectionnez **Fabriques de données**. Si vous ne voyez pas **Fabriques de données** dans le menu, sélectionnez **Autres services**, puis sélectionnez **Fabriques de données** dans la section **INTELLIGENCE + ANALYSE**.

   ![Liste de fabriques de données](media/join-azure-ssis-integration-runtime-virtual-network/data-factories-list.png)

1. Dans la liste, sélectionnez votre fabrique de données avec Azure-SSIS IR. La page d’accueil de votre fabrique de données apparaît. Sélectionnez la vignette **Créer et surveiller**. L’interface utilisateur de Data Factory apparaît sous un onglet séparé.

   ![Page d’accueil Data Factory](media/join-azure-ssis-integration-runtime-virtual-network/data-factory-home-page.png)

1. Dans l’interface utilisateur de Data Factory, basculez vers l’onglet **Modifier**, sélectionnez **Connexions**, puis basculez vers l’onglet **Runtimes d’intégration**.

   ![Onglet « Runtimes d’intégration »](media/join-azure-ssis-integration-runtime-virtual-network/integration-runtimes-tab.png)

1. Si votre runtime d’intégration Azure-SSIS IR est en cours d’exécution, dans la liste **Runtimes d’intégration**, au niveau de la colonne **Actions**, sélectionnez le bouton **Arrêter** pour votre runtime d’intégration Azure-SSIS IR. Vous ne pouvez pas modifier un runtime d’intégration Azure-SSIS IR tant que vous ne l’arrêtez pas.

   ![Arrêter le runtime d’intégration](media/join-azure-ssis-integration-runtime-virtual-network/stop-ir-button.png)

1. Dans la liste **Runtimes d’intégration**, au niveau de la colonne **Actions**, sélectionnez le bouton **Modifier** pour votre runtime d’intégration Azure-SSIS IR.

   ![Modifier le runtime d’intégration](media/join-azure-ssis-integration-runtime-virtual-network/integration-runtime-edit.png)

1. Dans le panneau de configuration du runtime d’intégration, passez les sections **Paramètres généraux** et **Paramètres SQL** en sélectionnant le bouton **Suivant**.

1. Dans la section **Paramètres avancés** :
   1. Activez la case à cocher **Sélectionner un réseau virtuel auquel joindre votre Azure-SSIS Integration Runtime, autoriser ADF à créer certaines ressources réseau, et éventuellement apporter vos propres adresses IP publiques statiques**.

   1. Dans **Abonnement**, sélectionnez l’abonnement Azure possédant votre réseau virtuel.

   1. Pour **Emplacement**, sélectionnez le même emplacement que celui de votre runtime d’intégration.

   1. Pour **Type**, sélectionnez le type de votre réseau virtuel : classique ou Azure Resource Manager. Nous vous recommandons de sélectionner un réseau virtuel Azure Resource Manager, car les réseaux virtuels classiques seront bientôt dépréciés.

   1. Pour **Nom du réseau virtuel**, sélectionnez le nom de votre réseau virtuel. Il doit être identique à celui utilisé pour votre serveur Azure SQL Database avec des points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB. Il peut également s’agir de celui connecté à votre réseau local. Sinon, vous pouvez utiliser n’importe quel réseau virtuel pour apporter vos propres adresses IP publiques statiques pour Azure-SSIS IR.

   1. Pour **Nom du sous-réseau**, sélectionnez le nom du sous-réseau de votre réseau virtuel. Il doit être identique à celui utilisé pour votre serveur Azure SQL Database avec des points de terminaison de service de réseau virtuel pour héberger le catalogue SSISDB. Il peut également s’agir d’un autre sous-réseau que celui utilisé pour votre instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB. Sinon, vous pouvez utiliser n’importe quel sous-réseau pour apporter vos propres adresses IP publiques statiques pour Azure-SSIS IR.

   1. Sélectionnez **Validation de réseau virtuel**. Si la validation réussit, sélectionnez **Continuer**.

   ![Paramètres avancés avec un réseau virtuel](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-vnet.png)

1. Dans la section **Résumé**, passez en revue tous les paramètres de votre runtime d’intégration Azure-SSIS IR. Sélectionnez ensuite **Mettre à jour**.

1. Démarrez votre Azure-SSIS IR en sélectionnant le bouton **Démarrer** situé dans la colonne **Actions** en regard de votre Azure-SSIS IR. Le démarrage de votre Azure-SSIS IR qui joint un réseau virtuel prend 20 à 30 minutes environ.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [jonction d’Azure-SSIS IR à un réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md).
