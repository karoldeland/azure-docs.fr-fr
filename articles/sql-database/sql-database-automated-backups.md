---
title: Sauvegardes géoredondantes automatiques
description: SQL Database crée automatiquement une sauvegarde de base de données locale toutes les cinq minutes et utilise le stockage géoredondant avec accès en lecture pour fournir la géoredondance.
services: sql-database
ms.service: sql-database
ms.subservice: backup-restore
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, carlrab, danil
manager: craigg
ms.date: 12/13/2019
ms.openlocfilehash: 16ee8c1e271f0aa3e6565322f9a4a422dd90b8b8
ms.sourcegitcommit: 6ee876c800da7a14464d276cd726a49b504c45c5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/19/2020
ms.locfileid: "77461768"
---
# <a name="automated-backups"></a>Sauvegardes automatisées

SQL Database crée automatiquement des sauvegardes de base de données, qui sont conservées pour la durée de période de rétention configurée, en tirant parti du [stockage géoredondant avec accès en lecture seule (RA-GRS)](../storage/common/storage-redundancy.md) d’Azure pour garantir leur conservation, même en cas d’indisponibilité du centre de données. Ces sauvegardes sont créées automatiquement. Les sauvegardes de base de données sont une partie essentielle de toute stratégie de continuité d’activité ou de récupération d’urgence, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles. Si vos règles de sécurité nécessitent que vos sauvegardes soient disponibles pendant une période prolongée (jusqu’à 10 ans), vous pouvez configurer une stratégie de [conservation à long terme](sql-database-long-term-retention.md) dans des bases de données unique et des pools élastiques.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="what-is-a-sql-database-backup"></a>Qu’est-ce qu’une sauvegarde SQL Database ?

SQL Database utilise la technologie SQL Server pour créer des sauvegardes [complètes](https://docs.microsoft.com/sql/relational-databases/backup-restore/full-database-backups-sql-server) (chaque semaine), [différentielles](https://docs.microsoft.com/sql/relational-databases/backup-restore/differential-backups-sql-server) (toutes les 12 heures) et du [journal des transactions](https://docs.microsoft.com/sql/relational-databases/backup-restore/transaction-log-backups-sql-server) (toutes les 5 à 10 minutes). Les sauvegardes sont stockées dans des [objets blob de stockage RA-GRS](../storage/common/storage-redundancy.md) répliqués dans un [centre de données associé](../best-practices-availability-paired-regions.md) afin de proposer une protection contre toute panne du centre de données. Quand vous restaurez une base de données, le service identifie les sauvegardes nécessitant une restauration (complète, différentielle ou journal des transactions).

Vous pouvez utiliser ces sauvegardes aux fins suivantes :

- **Restaurer une base de données existante dans le temps**, à un moment situé pendant la période de rétention, à l’aide du portail Microsoft Azure, de Microsoft Azure PowerShell, de Microsoft Azure CLI ou de l’API REST. Dans une base de données unique et des pools élastiques, cette opération crée une base de données au sein du même serveur que la base de données d’origine. Dans Managed Instance, cette opération peut créer une copie de la base de données, ou une instance Managed Instance, identique ou non, pour le même abonnement.
- **Restaurer une base de données supprimée sur le moment où elle été supprimée,** ou tout moment pendant la période de rétention. La base de données supprimée ne peut être restaurée que sur le serveur logique ou Managed Instance sur lequel la base de données d’origine a été créée.
- **Restaurer une base de données dans une autre région géographique**. La géorestauration vous permet de procéder à la récupération après un sinistre géographique lorsque vous ne pouvez pas accéder à votre serveur, ni à la base de données. Cette opération crée une base de données sur n’importe quel serveur existant dans le monde entier.
- **Restaurer une base de données à partir d’une sauvegarde à long terme spécifique** dans une base de données unique ou un pool élastique, si la base de données a été configurée avec une stratégie de conservation à long terme (LTR). La conservation à long terme (LTR) vous permet de restaurer une ancienne version de la base de données à l’aide du [portail Microsoft Azurel](sql-database-long-term-backup-retention-configure.md#using-azure-portal) ou de [Microsoft Azure PowerShell](sql-database-long-term-backup-retention-configure.md#using-powershell) pour répondre à une requête de conformité ou exécuter une ancienne version de l’application. Pour plus d’informations, consultez [Rétention à long terme](sql-database-long-term-retention.md).
- Pour effectuer une restauration, consultez [Restauration de la base de données à partir de la sauvegarde](sql-database-recovery-using-backups.md).

> [!NOTE]
> Dans le stockage Azure, le terme *réplication* fait référence à la copie de fichier d’un emplacement à un autre. La *réplication de base de données* de SQL fait référence à la gestion de la synchronisation de plusieurs bases de données secondaires avec une base de données primaire.

Vous pouvez essayer certaines de ces opérations en utilisant les exemples suivants :

| | Le portail Azure | Azure PowerShell |
|---|---|---|
| Modifier la rétention des sauvegardes | [Base de données unique](sql-database-automated-backups.md?tabs=managed-instance#change-pitr-backup-retention-period-using-azure-portal) <br/> [Managed Instance](sql-database-automated-backups.md?tabs=managed-instance#change-pitr-backup-retention-period-using-azure-portal) | [Base de données unique](sql-database-automated-backups.md#change-pitr-backup-retention-period-using-powershell) <br/>[Managed Instance](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlinstancedatabasebackupshorttermretentionpolicy) |
| Modifier la rétention des sauvegardes à long terme | [Base de données unique](sql-database-long-term-backup-retention-configure.md#configure-long-term-retention-policies)<br/>Managed Instance - N/A  | [Base de données unique](sql-database-long-term-backup-retention-configure.md)<br/>Managed Instance - N/A  |
| Restaurer la base de données dans le temps | [Base de données unique](sql-database-recovery-using-backups.md#point-in-time-restore) | [Base de données unique](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase) <br/> [Managed Instance](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqlinstancedatabase) |
| Restauration d’une base de données supprimée | [Base de données unique](sql-database-recovery-using-backups.md) | [Base de données unique](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldeleteddatabasebackup) <br/> [Managed Instance](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldeletedinstancedatabasebackup)|
| Restauration d’une base de données à partir du Stockage Blob Azure | Base de données unique - N/A <br/>Managed Instance - N/A  | Base de données unique - N/A <br/>[Managed Instance](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-get-started-restore) |


## <a name="backup-frequency"></a>Fréquence de sauvegarde

### <a name="point-in-time-restore"></a>Restauration dans le temps

SQL Database prend en charge la limite de restauration dans le temps en libre-service en créant automatiquement une sauvegarde complète, des sauvegardes différentielles et des sauvegardes du journal des transactions. Les sauvegardes de base de données complètes sont créées chaque semaine, les sauvegardes différentielles généralement toutes les 12 heures, et les sauvegardes de journal des transactions généralement toutes les 5 à 10 minutes, la fréquence variant selon la taille de calcul et l’activité de la base de données. La première sauvegarde complète est planifiée immédiatement après la création d’une base de données. Elle s’exécute généralement en 30 minutes, mais elle peut nécessiter davantage de temps s’il s’agit d’une base de données de taille considérable. Par exemple, la sauvegarde initiale peut prendre davantage de temps sur une base de données restaurée ou une copie de base de données. Après la première sauvegarde complète, toutes les sauvegardes sont planifiées automatiquement et gérées en mode silencieux en arrière-plan. Le moment exact de toutes les sauvegardes de base de données est déterminé par le service SQL Database en fonction de l’équilibrage de la charge de travail globale du système. Vous ne pouvez pas modifier ou désactiver des travaux de sauvegarde.

Les sauvegardes avec récupération jusqu’à une date et heure sont protégées par un stockage géoredondant. Pour plus d’informations, consultez [Redondance de Stockage Azure](../storage/common/storage-redundancy.md).

Pour plus d'informations, consultez [Limite de restauration dans le temps](sql-database-recovery-using-backups.md#point-in-time-restore)

### <a name="long-term-retention"></a>Rétention à long terme

Les bases de données uniques et mises en pool permettent de configurer une conservation à long terme des sauvegardes complètes allant jusqu’à 10 ans dans le service Stockage Blob Azure. Si la stratégie de conservation à long terme est activée, les sauvegardes complètes hebdomadaires sont automatiquement copiées vers un autre conteneur de stockage RA-GRS. Pour répondre aux différentes exigences de conformité, vous pouvez sélectionner plusieurs périodes de rétention pour les sauvegardes hebdomadaires, mensuelles ou annuelles. La consommation du stockage dépend de la fréquence sélectionnée des sauvegardes et des périodes de conservation. Vous pouvez utiliser la [calculatrice de prix LTR](https://azure.microsoft.com/pricing/calculator/?service=sql-database) pour estimer le coût du stockage de conservation à long terme.

Comme les sauvegardes avec récupération jusqu’à une date et heure, les sauvegardes avec conservation à long terme sont protégées par un stockage géoredondant. Pour plus d’informations, consultez [Redondance de Stockage Azure](../storage/common/storage-redundancy.md).

Pour plus d’informations, consultez [Conservation des sauvegardes à long terme](sql-database-long-term-retention.md).

## <a name="backup-storage-consumption"></a>Consommation de stockage de sauvegarde 

Pour les bases de données uniques, l’utilisation totale du stockage de sauvegarde est calculée comme suit :   
`Total backup storage size = (size of full backups + size of differential backups + size of log backups) – database size`.

Pour les pools élastiques, la taille totale de stockage de sauvegarde est agrégée au niveau du pool et calculée comme suit :   
`Total backup storage size = (total size of all full backups + total size of all differential backups + total size of all log backups) - allocated pool data storage`. 

Les sauvegardes antérieures à la période de rétention sont automatiquement purgées en fonction de leur horodatage. Étant donné que les sauvegardes différentielles et les sauvegardes de fichiers journaux requièrent une sauvegarde complète antérieure, elles sont purgées en blocs hebdomadaires. 

Azure SQL Database calcule votre stockage de sauvegarde total en rétention en tant que valeur cumulée. Toutes les heures, cette valeur est consignée dans le pipeline de facturation Azure, qui est responsable de l’agrégation de cette utilisation horaire afin de calculer votre consommation à la fin de chaque mois. Une fois la base de données supprimée, la consommation diminue à mesure que les sauvegardes vieillissent. Une fois que les sauvegardes sont antérieures à la période de rétention, la facturation s’arrête. 

   > [!IMPORTANT]
   > Les sauvegardes d’une base de données sont conservées pendant la période de rétention spécifiée, même si la base de données a été supprimée. Si le fait de supprimer et de recréer fréquemment une base de données peut permettre d’économiser sur les coûts de stockage et de calcul, cela peut augmenter les coûts de stockage des sauvegardes, car nous conservons une sauvegarde pendant la période de rétention spécifiée (sept jours au minimum) pour chaque base de données supprimée et à chaque suppression. 



### <a name="monitor-consumption"></a>Surveiller la consommation

Chaque type de sauvegarde (complète, différentielle et journal) est signalé dans le panneau de surveillance de la base de données sous la forme d’une mesure distincte. Le diagramme suivant montre comment surveiller la consommation de stockage des sauvegardes pour une base de données unique. Cette fonctionnalité n’est pas disponible actuellement pour les instances gérées.

![Surveiller la consommation de sauvegarde de base de données dans le panneau de surveillance de la base de données du Portail Azure](media/sql-database-automated-backup/backup-metrics.png)

### <a name="fine-tune-the-backup-storage-consumption"></a>Ajuster la consommation de stockage de sauvegarde

Cette consommation de stockage de sauvegarde excessive dépend de la charge de travail et de la taille des bases de données individuelles. Vous pouvez envisager d’implémenter certaines des techniques de paramétrage suivantes pour réduire davantage votre consommation de stockage de sauvegarde :

* Réduisez la [période de rétention de la sauvegarde](#change-pitr-backup-retention-period-using-azure-portal) au minimum possible pour vos besoins.
* Évitez d’effectuer des opérations d’écriture volumineuses plus fréquemment que nécessaire, comme les reconstructions d’index.
* Pour les opérations de chargement de données volumineuses, envisagez d’utiliser des [index columnstore en cluster](https://docs.microsoft.com/sql/database-engine/using-clustered-columnstore-indexes), de réduire le nombre d’index non cluster et de prendre en compte les opérations de chargement en masse avec environ 1 million de lignes.
* Dans le niveau de service Usage général, le stockage de données provisionné est moins cher que le prix du stockage de sauvegarde excédentaire, ce qui fait que les clients dont les coûts de stockage de sauvegarde excédentaire sont continuellement excédentaires peuvent envisager d'augmenter le stockage de données afin d'économiser sur le stockage de sauvegarde.
* Utilisez TempDB dans votre logique ETL pour le stockage des résultats temporaires, au lieu de tables permanentes (applicables uniquement à l’instance managée).
* Envisagez de désactiver le chiffrement TDE pour les bases de données qui ne contiennent pas de données sensibles (bases de données de développement ou de test, par exemple). Les sauvegardes des bases de données non chiffrées sont généralement compressées avec un taux de compression supérieur.

> [!IMPORTANT]
> Pour les charges de travail de DataMart/de l’entrepôt de données et analytiques, il est fortement recommandé d’[utiliser des index columnstore en cluster](https://docs.microsoft.com/sql/database-engine/using-clustered-columnstore-indexes), de réduire le nombre d’index non cluster et de considérer également les opérations de chargement en masse en prévoyant le traitement de 1 million de lignes environ afin de réduire la consommation de stockage de sauvegarde excessive.


## <a name="storage-costs"></a>Coûts de stockage

Le prix du stockage varie selon que vous utilisez le modèle DTU ou le modèle vCore. 

### <a name="dtu-model"></a>Modèle DTU

Aucun frais supplémentaire n’est facturé pour le stockage de sauvegarde pour les bases de données et les pools élastiques à l’aide du modèle DTU. 

### <a name="vcore-model"></a>Modèle vCore

Pour les bases de données uniques, une quantité minimale de stockage de sauvegarde égale à 100 % de la taille de la base de données est fournie sans frais supplémentaires. Pour les pools élastiques et les instances managées, une quantité minimale de stockage de sauvegarde égale à 100 % du stockage de données alloué pour le pool ou la taille d’instance respectivement est fournie sans frais supplémentaires. Toute consommation supérieure de stockage de sauvegarde est facturée en Go/mois. Cette consommation supplémentaire dépend de la charge de travail et de la taille des bases de données individuelles.

Azure SQL DB calcule votre stockage de sauvegarde total en rétention en tant que valeur cumulée. Toutes les heures, cette valeur est consignée dans le pipeline de facturation Azure, qui est responsable de l’agrégation de cette utilisation horaire afin d’obtenir votre consommation à la fin de chaque mois. Une fois la base de données supprimée, nous réduisons la consommation à mesure que les sauvegardes vieillissent. Une fois qu’elles sont antérieures à la période de rétention, la facturation s’arrête. Étant donné que toutes les sauvegardes de fichier journal et les sauvegardes différentielles sont conservées pendant la période de rétention complète, les bases de données qui sont fortement modifiées engendrent des frais de sauvegarde plus élevés. 

Supposons que la base de données ait accumulé 744 Go de stockage de sauvegarde et que cette quantité reste constante tout au long d’un mois. Pour convertir cette consommation de stockage cumulée en une utilisation horaire, nous la divisons par 744,0 (31 jours par mois * 24 heures par jour). Ainsi, SQL DB signale que la base de données a consommé 1 Go de sauvegarde PITR chaque heure. La facturation Azure regroupera cet agrégat et affichera une utilisation de 744 Go pour le mois entier et le coût en fonction du taux €/Go/mois dans votre région. 

À présent, un exemple plus complexe. Supposons que la durée de rétention de la base de données ait augmenté de 14 jours au milieu du mois et que cela (de façon hypothétique) double la taille du stockage de sauvegarde (1 488 Go). SQL DB signale 1 Go d’utilisation pour les heures 1 à 372, puis signale l’utilisation de 2 Go pour les heures 373 à 744. Il s’agit d’une facture finale de 1116 Go/mois. 

### <a name="monitor-costs"></a>Superviser les coûts

Pour comprendre les coûts de stockage des sauvegardes, accédez à **Azure Cost Management + facturation** dans le Portail Azure, sélectionnez **Cost Management** (Gestion des coûts), puis sélectionnez **Cost analysis** (Analyse des coûts). Sélectionnez l’abonnement souhaité comme **Scope** (Étendue), puis filtrez la période et le service qui vous intéressent. 

Ajoutez un filtre pour **Service name** (Nom de service), puis choisissez **sql database** dans la liste déroulante. Utilisez le filtre **Meter subcategory** (Sous-catégorie du compteur) pour choisir le compteur de facturation pour votre service. Pour une base de données unique ou un pool élastique, choisissez **single/elastic pool pitr backup storage**. Pour une instance gérée, choisissez **mi pitr backup storage**. Les sous-catégories **Storage** (Stockage) et **Compute** (Calcul) peuvent vous également intéresser, bien qu’elles ne soient pas associées à des coûts de stockage de sauvegarde. 

![Analyse du coût du stockage de sauvegarde](./media/sql-database-automated-backup/check-backup-storage-cost-sql-mi.png)


## <a name="backup-retention"></a>Rétention des sauvegardes

Toutes les bases de données Azure SQL (uniques, mises en pool et Managed Instance), présentent une période de rétention des sauvegardes de **sept** jours. Vous pouvez [modifier la période de rétention des sauvegardes jusqu’à 35 jours](#change-pitr-backup-retention-period).

Si vous supprimez une base de données, SQL Database conserve les sauvegardes de la même façon que s’il s’agissait d’une base de données en ligne. Par exemple, si vous supprimez une base de données De base dont la période de conservation est de sept jours, une sauvegarde datant de quatre jours est enregistrée pendant encore trois jours.

Si vous souhaitez conserver les sauvegardes plus longtemps que la période de conservation maximale, vous pouvez modifier les propriétés de sauvegarde pour ajouter une ou plusieurs périodes de conservation à long terme à votre base de données. Pour plus d’informations, consultez [Rétention à long terme](sql-database-long-term-retention.md).

> [!IMPORTANT]
> Si vous supprimez le serveur Azure SQL Server qui héberge les bases de données SQL, tous les pools élastiques et bases de données qui appartiennent au serveur sont également supprimés, sans pouvoir être restaurés. Vous ne pouvez pas restaurer un serveur supprimé. Mais si vous avez configuré la conservation à long terme, les sauvegardes des bases de données avec conservation à long terme ne seront pas supprimées et peuvent être restaurées.

## <a name="encrypted-backups"></a>Sauvegardes chiffrées

Si votre base de données est chiffrée à l’aide de TDE, les sauvegardes sont automatiquement chiffrées au repos, y compris les sauvegardes LTR. Lorsque TDE est activé pour une base de données Azure SQL, les sauvegardes sont également chiffrées. TDE est configuré par défaut sur l’ensemble des nouvelles bases de données Azure SQL. Pour en savoir plus sur le TDE, consultez [Transparent Data Encryption avec Azure SQL Database](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql).

## <a name="backup-integrity"></a>Intégrité de la sauvegarde

L’équipe d’ingénieurs Azure SQL Database teste régulièrement et automatiquement la restauration des sauvegardes automatisées de bases de données placées sur des serveurs logiques et dans des pools élastiques (non disponible dans Managed Instance). Lors d’une restauration à un moment donné, les bases de données subissent également des vérifications d’intégrité à l’aide de DBCC CHECKDB.

Managed Instance effectue une sauvegarde initiale automatique avec `CHECKSUM` des bases de données restaurées à l’aide de la commande `RESTORE` native ou du service de migration des données à l’issue de la migration.

Tout problème détecté lors de la vérification d’intégrité est traduit par une alerte envoyée à l’équipe d’ingénieurs. Pour plus d’informations sur l’intégrité des données dans Azure SQL Database, consultez l’article [Intégrité des données dans Azure SQL Database](https://azure.microsoft.com/blog/data-integrity-in-azure-sql-database/).

## <a name="compliance"></a>Conformité

Lorsque vous migrez votre base de données à partir d’un niveau de service basé sur DTU vers un niveau de service basé sur vCore, la conservation PITR est préservée pour garantir que la stratégie de récupération de données de votre application n’est pas compromise. Si la conservation par défaut ne répond pas à vos exigences de conformité, vous pouvez modifier la période de conservation PITR à l’aide de PowerShell ou de l’API REST. Consultez [Modification de la période de conservation](#change-pitr-backup-retention-period) pour plus d’informations.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="change-pitr-backup-retention-period"></a>Modifier la période de conservation des sauvegardes PITR

Vous pouvez modifier la période de rétention des sauvegardes PITR par défaut à l’aide du portail Microsoft Azure, de PowerShell ou de l’API REST. Les exemples suivants illustrent comment modifier la conservation de récupération jusqu’à une date et heure pour la faire passer à 28 jours.

> [!WARNING]
> Si vous réduisez la période de rétention actuelle, toutes les sauvegardes antérieures à la nouvelle période de rétention ne seront plus disponibles. Si vous augmentez la période de rétention actuelle, SQL Database conserve les sauvegardes existantes jusqu’à ce que la période de rétention plus longue soit atteinte.

> [!NOTE]
> Ces API impactent uniquement la période de conservation PITR. Si vous avez configuré la conservation à long terme pour votre base de données, elle ne sera pas affectée. Consultez [Conservation à long terme](sql-database-long-term-retention.md) pour en savoir plus sur la modification des périodes de conservation à long terme.

### <a name="change-pitr-backup-retention-period-using-azure-portal"></a>Changer la période de conservation des sauvegardes PITR à l’aide du portail Azure

Pour changer la période de rétention des sauvegardes PITR dans le portail Microsoft Azure, accédez à l’objet serveur dont vous souhaitez changer la période de rétention dans le portail, puis sélectionnez l’option appropriée, selon l’objet serveur que vous modifiez.

#### <a name="single-database--elastic-pools"></a>[Base de données unique et pools élastiques](#tab/single-database)

Le changement de la conservation des sauvegardes PITR pour les bases de données Azure SQL uniques est effectué au niveau du serveur. Les changements effectués au niveau du serveur s’appliquent aux bases de données sur le serveur concerné. Pour changer le processus PITR pour un serveur Azure SQL Database à partir du portail Azure, accédez au panneau de vue d’ensemble du serveur, cliquez sur Gérer les sauvegardes dans le menu de navigation, puis cliquez sur Configurer la rétention dans la barre de navigation.

![Modification PITR sur le Portail Azure](./media/sql-database-automated-backup/configure-backup-retention-sqldb.png)

#### <a name="managed-instance"></a>[Managed Instance](#tab/managed-instance)

La modification de la conservation des sauvegardes PITR pour une instance managée SQL Database est effectuée au niveau d’une base de données individuelle. Pour changer la conservation des sauvegardes PITR pour une base de données d’instance à partir du portail Azure, accédez au panneau de vue d’ensemble de la base de données concernée, puis cliquez sur Configurer la conservation de sauvegarde dans la barre de navigation.

![Modification PITR sur le Portail Azure](./media/sql-database-automated-backup/configure-backup-retention-sqlmi.png)

---

### <a name="change-pitr-backup-retention-period-using-powershell"></a>Modifier la période de conservation des sauvegardes PITR et à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

```powershell
Set-AzSqlDatabaseBackupShortTermRetentionPolicy -ResourceGroupName resourceGroup -ServerName testserver -DatabaseName testDatabase -RetentionDays 28
```

### <a name="change-pitr-retention-period-using-rest-api"></a>Modifier la période de conservation PITR à l’aide de l’API REST

#### <a name="sample-request"></a>Exemple de demande

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/resourceGroup/providers/Microsoft.Sql/servers/testserver/databases/testDatabase/backupShortTermRetentionPolicies/default?api-version=2017-10-01-preview
```

#### <a name="request-body"></a>Corps de la requête

```json
{
  "properties":{
    "retentionDays":28
  }
}
```

#### <a name="sample-response"></a>Exemple de réponse

Code d’état : 200

```json
{
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/providers/Microsoft.Sql/resourceGroups/resourceGroup/servers/testserver/databases/testDatabase/backupShortTermRetentionPolicies/default",
  "name": "default",
  "type": "Microsoft.Sql/resourceGroups/servers/databases/backupShortTermRetentionPolicies",
  "properties": {
    "retentionDays": 28
  }
}
```

Pour plus d’informations, consultez [API REST de conservation des sauvegardes](https://docs.microsoft.com/rest/api/sql/backupshorttermretentionpolicies).

## <a name="next-steps"></a>Étapes suivantes

- Les sauvegardes de base de données sont une partie essentielle de toute stratégie de continuité d’activité ou de récupération d’urgence, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles. Pour en savoir plus sur les autres solutions de continuité des activités Azure SQL Database, consultez [Vue d’ensemble de la continuité des activités](sql-database-business-continuity.md).
- Pour effectuer une restauration à un point dans le temps à l’aide du portail Azure, consultez [Restauration d’une base de données à un point dans le temps à l’aide du portail Azure](sql-database-recovery-using-backups.md).
- Pour effectuer une restauration à un point dans le temps à l’aide de PowerShell, consultez [Restauration d’une base de données à un point dans le temps à l’aide de PowerShell](scripts/sql-database-restore-database-powershell.md).
- Pour configurer, gérer et restaurer depuis la conservation à long terme de sauvegardes automatisées dans Stockage Blob Azure avec le portail Azure, consultez [Gestion de la rétention des sauvegardes à long terme à l’aide du Portail Azure](sql-database-long-term-backup-retention-configure.md).
- Pour configurer, gérer et restaurer des données à partir d’une conservation à long terme de sauvegardes automatisées dans Stockage Blob Azure avec PowerShell, voir [Gestion de la conservation des sauvegardes à long terme avec PowerShell](sql-database-long-term-backup-retention-configure.md).
