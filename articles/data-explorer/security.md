---
title: Sécuriser les clusters Azure Data Explorer dans Azure
description: En savoir plus sur la sécurisation des clusters dans Azure Data Explorer.
author: saguiitay
ms.author: itsagui
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 01/06/2020
ms.openlocfilehash: 786950011f10e25d6bcb72061212c1878e79d45a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77373358"
---
# <a name="secure-azure-data-explorer-clusters-in-azure"></a>Sécuriser les clusters Azure Data Explorer dans Azure

Cet article fournit une introduction à la sécurité dans Azure Data Explorer qui vous aide à protéger vos données et ressources dans le cloud et à répondre aux besoins de sécurité de votre entreprise. Il est important de sécuriser vos clusters. La sécurisation de vos clusters inclut une ou plusieurs fonctionnalités Azure qui comprennent un accès et un stockage sécurisés. Cet article fournit des informations pour vous aider à sécuriser vos clusters.

## <a name="managed-identities-for-azure-resources"></a>Identités gérées pour les ressources Azure

La gestion des informations d’identification dans votre code pour s’authentifier auprès des services cloud constitue un défi courant lors de la génération d’applications cloud. La sécurisation de ces informations d’identification est une tâche importante. Les informations d’identification ne doivent pas être stockées dans les stations de travail des développeurs ni être archivées dans le contrôle de code source. Azure Key Vault permet de stocker en toute sécurité des informations d’identification, secrets, et autres clés, mais votre code doit s’authentifier sur Key Vault pour les récupérer.

La fonctionnalité des identités managées Azure Active Directory (Azure AD) pour les ressources Azure résout ce problème. Elle fournit aux services Azure une identité système administrée automatiquement dans Azure AD. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, y compris Key Vault, sans avoir d’informations d’identification dans votre code. Pour plus d’informations sur ce service, consultez la page de vue d’ensemble des [identités managées pour les ressources Azure](/azure/active-directory/managed-identities-azure-resources/overview).

## <a name="data-encryption"></a>Chiffrement des données

### <a name="azure-disk-encryption"></a>Azure Disk Encryption

[Azure Disk Encryption](/azure/security/azure-security-disk-encryption-overview) vous aide à protéger et à préserver vos données de façon à répondre aux engagements de votre entreprise en matière de sécurité et de conformité. Il fournit un chiffrement de volume pour le système d’exploitation et les disques de données des machines virtuelles de votre cluster. Azure Disk Encryption est également intégré à [Azure Key Vault](/azure/key-vault/), ce qui vous permet de contrôler et de gérer les secrets et les clés de chiffrement de disque et de garantir que toutes les données figurant sur les disques des machines virtuelles sont chiffrées. 

### <a name="customer-managed-keys-with-azure-key-vault"></a>Clés gérées par le client avec Azure Key Vault

Par défaut, les données sont chiffrées avec des clés managées par Microsoft Pour plus de contrôle sur les clés de chiffrement, vous pouvez fournir des clés gérées par le client à utiliser pour le chiffrement des données. Vous pouvez gérer le chiffrement de vos données au niveau du stockage à l’aide de vos propres clés. Une clé gérée par le client est utilisée pour protéger et contrôler l’accès à la clé de chiffrement racine, laquelle est utilisée pour chiffrer et déchiffrer toutes les données. Les clés managées par le client offrent plus de flexibilité pour créer, permuter, désactiver et révoquer des contrôles d’accès. Vous pouvez également effectuer un audit sur les clés de chiffrement utilisées pour protéger vos données.

Utilisez Azure Key Vault pour stocker vos clés gérées par le client. Vous pouvez créer vos propres clés et les stocker dans un coffre de clés, ou utiliser une API d’Azure Key Vault pour générer des clés. Le cluster Azure Data Explorer et l’instance Azure Key Vault se trouver dans la même région, mais ils peuvent appartenir à des abonnements différents. Pour plus d’informations sur Azure Key Vault, consultez [Qu’est-ce qu’Azure Key Vault ?](/azure/key-vault/key-vault-overview). Pour obtenir une explication détaillée sur les clés gérées par le client, consultez [Clés gérées par le client avec Azure Key Vault](/azure/storage/common/storage-service-encryption). Configurer des clés gérées par le client dans votre cluster Azure Data Explorer à l’aide du langage [C#](/azure/data-explorer/customer-managed-keys-csharp) ou du [modèle Azure Resource Manager](/azure/data-explorer/customer-managed-keys-resource-manager)

> [!Note]
> Les clés managées par le client s’appuient sur des identités managées pour ressources Azure, une fonctionnalité d’Azure Active Directory (Azure AD). Pour configurer des clés gérées par le client dans le Portail Azure, vous devez configurer une identité managée **SystemAssigned** sur votre cluster, comme indiqué dans [Configurer des identités managées pour votre cluster Azure Data Explorer](/azure/data-explorer/managed-identities).

#### <a name="store-customer-managed-keys-in-azure-key-vault"></a>Stocker les clés gérées par le client dans Azure Key Vault

Pour activer les clés gérées par le client sur un cluster, utilisez une instance Azure Key Vault pour stocker vos clés. Vous devez activer les propriétés **Suppression réversible** et **Ne pas vider** sur le coffre de clés. Le coffre de clés doit se trouver dans le même abonnement que le cluster. Azure Data Explorer utilise des identités managées afin que les ressources Azure s’authentifient auprès du coffre de clés pour les opérations de chiffrement et de déchiffrement. Les identités managées ne prennent pas en charge les scénarios sur plusieurs répertoires.

#### <a name="rotate-customer-managed-keys"></a>Permuter des clés gérées par le client

Vous pouvez permuter une clé gérée par le client dans Azure Key Vault en fonction de vos stratégies de conformité. Une fois la clé permutée, vous devez mettre à jour le cluster pour utiliser le nouvel URI de clé. La permutation de la clé ne déclenche pas le rechiffrement des données dans le cluster. 

#### <a name="revoke-access-to-customer-managed-keys"></a>Révoquer l’accès aux clés gérées par le client

Pour révoquer l’accès aux clés gérées par le client, utilisez PowerShell ou Azure CLI. Pour plus d’informations, consultez [Azure Key Vault PowerShell](/powershell/module/az.keyvault/) ou [Interface de ligne de commande Azure Key Vault](/cli/azure/keyvault). La révocation de l’accès bloque l’accès à toutes les données situées au niveau de stockage du cluster, car la clé de chiffrement est donc inaccessible à Azure Data Explorer.

> [!Note]
> Quand Azure Data Explorer identifie que l’accès à une clé gérée par le client est révoqué, il suspend automatiquement le cluster pour supprimer toutes les données mises en cache. Une fois que l’accès à la clé est restauré, la reprise du cluster doit être faite manuellement.

## <a name="role-based-access-control"></a>Contrôle d’accès en fonction du rôle

Grâce au [contrôle d’accès en fonction du rôle (RBAC)](/azure/role-based-access-control/overview), vous pouvez séparer les tâches au sein de votre équipe et accorder aux utilisateurs du cluster uniquement les accès nécessaires. Plutôt que de donner à tous des autorisations illimitées sur le cluster, vous pouvez autoriser uniquement certaines actions. Vous pouvez configurer le [contrôle d’accès pour les bases de données](/azure/data-explorer/manage-database-permissions) dans le [Portail Azure](/azure/role-based-access-control/role-assignments-portal), à l’aide d’[Azure CLI](/azure/role-based-access-control/role-assignments-cli) ou d’[Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell).

## <a name="next-steps"></a>Étapes suivantes

* [Sécuriser votre cluster dans Azure Data Explorer – Portail](manage-cluster-security.md) en activant le chiffrement au repos.
* [Configurer des identités managées pour votre cluster Azure Data Explorer](managed-identities.md)
* [Configurer des clés gérées par le client à l’aide du modèle Azure Resource Manager](customer-managed-keys-resource-manager.md)
* [Configurer des clés gérées par le client à l’aide de C#](customer-managed-keys-csharp.md)

