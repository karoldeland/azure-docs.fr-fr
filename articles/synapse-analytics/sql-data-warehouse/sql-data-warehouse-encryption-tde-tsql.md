---
title: Chiffrement transparent des données (T-SQL)
description: Chiffrement transparent des données (TDE, Transparent Data Encryption) dans Azure Synapse Analytics (T-SQL)
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/30/2019
ms.author: jrasnick
ms.reviewer: rortloff
ms.custom: seo-lt-2019
ms.openlocfilehash: 14058a91ee21bd792f9be1a64d554092c4f5aedd
ms.sourcegitcommit: 8a9c54c82ab8f922be54fb2fcfd880815f25de77
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "80351506"
---
# <a name="get-started-with-transparent-data-encryption-tde"></a>Prise en main du chiffrement transparent des données (TDE)
> [!div class="op_single_selector"]
> * [Présentation de la sécurité](sql-data-warehouse-overview-manage-security.md)
> * [Authentification](sql-data-warehouse-authentication.md)
> * [Chiffrement (portail)](sql-data-warehouse-encryption-tde.md)
> * [Chiffrement (T-SQL)](sql-data-warehouse-encryption-tde-tsql.md)
> 
> 

## <a name="required-permissions"></a>Autorisations requises
Pour activer le chiffrement transparent des données (TDE), vous devez être un administrateur ou un membre du rôle dbmanager.

## <a name="enabling-encryption"></a>Activation du chiffrement
Procédez comme suit pour activer le TDE :

1. Connectez-vous à la base de données *master* sur le serveur hébergeant la base de données à l'aide d'identifiants de connexion administrateurs ou membres du rôle **dbmanager** dans la base de données master
2. Exécutez l'instruction suivante pour chiffrer la base de données.

```sql
ALTER DATABASE [AdventureWorks] SET ENCRYPTION ON;
```

## <a name="disabling-encryption"></a>Désactivation du chiffrement
Procédez comme suit pour désactiver le TDE :

1. Connectez-vous à la base de données *master* à l'aide d'identifiants de connexion administrateurs ou membres du rôle **dbmanager** dans la base de données master
2. Exécutez l'instruction suivante pour chiffrer la base de données.

```sql
ALTER DATABASE [AdventureWorks] SET ENCRYPTION OFF;
```

> [!NOTE]
> Un pool SQL suspendu doit reprendre avant que des modifications ne puissent être apportées aux paramètres TDE.
> 
> 

## <a name="verifying-encryption"></a>Vérification de chiffrement
Pour vérifier l’état du chiffrement, procédez comme suit :

1. Connectez-vous à la base de données *master* ou d’instance à l'aide d'identifiants de connexion administrateurs ou membres du rôle **dbmanager** dans la base de données master
2. Exécutez l'instruction suivante pour chiffrer la base de données.

```sql
SELECT
    [name],
    [is_encrypted]
FROM
    sys.databases;
```

Un résultat de ```1``` indique une base de données chiffrée, ```0``` indique une base de données non chiffrée.

## <a name="encryption-dmvs"></a>DMV de chiffrement
* [sys.databases][sys.databases] 
* [sys.dm_pdw_nodes_database_encryption_keys][sys.dm_pdw_nodes_database_encryption_keys]

<!--Anchors-->
[Transparent Data Encryption (TDE)]: https://msdn.microsoft.com/library/bb934049.aspx
[sys.databases]: https://msdn.microsoft.com/library/ms178534.aspx  
[sys.dm_pdw_nodes_database_encryption_keys]: https://msdn.microsoft.com/library/mt203922.aspx  

<!--Image references-->

<!--Link references-->
