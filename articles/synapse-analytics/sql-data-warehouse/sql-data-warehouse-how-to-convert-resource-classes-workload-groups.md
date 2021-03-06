---
title: Convertir la classe de ressources en groupe de charge de travail
description: Découvrez comment créer un groupe de charge de travail similaire à une classe de ressources dans Azure SQL Data Warehouse.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.subservice: ''
ms.topic: conceptual
ms.date: 11/4/2019
ms.author: rortloff
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: d20a811270b89772ab75bc298544a549caa01041
ms.sourcegitcommit: 8a9c54c82ab8f922be54fb2fcfd880815f25de77
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "80350444"
---
# <a name="convert-resource-classes-to-workload-groups"></a>Convertir les classes de ressources en groupes de charge de travail
Les groupes de charges de travail fournissent un mécanisme permettant d’isoler et de contenir des ressources système.  En outre, les groupes de charges de travail vous permettent de définir des règles d’exécution pour les requêtes qui s’y exécutent.  Une règle d’exécution du délai d’expiration de la requête permet d’annuler les pertes de contrôle de requête sans intervention de l’utilisateur.  Cet article explique comment prendre une classe de ressource existante et créer un groupe de charge de travail avec une configuration similaire.  En outre, une règle facultative de délai d’attente de requête est ajoutée.

## <a name="understanding-the-existing-resource-class-configuration"></a>Comprendre la configuration de la classe de ressources existante
Les groupes de charge de travail requièrent un paramètre appelé `REQUEST_MIN_RESOURCE_GRANT_PERCENT` qui spécifie le pourcentage de ressources système globales allouées par requête.  L’allocation des ressources est effectuée pour les [classes de ressources](https://docs.microsoft.com/azure/sql-data-warehouse/resource-classes-for-workload-management#what-are-resource-classes) en allouant des emplacements de concurrence.  Pour déterminer la valeur à spécifier pour `REQUEST_MIN_RESOURCE_GRANT_PERCENT`, utilisez la vue de gestion dynamique sys.dm_workload_management_workload_groups_stats <link tbd>.  Par exemple, la requête ci-dessous retourne une valeur qui peut être utilisée pour le paramètre `REQUEST_MIN_RESOURCE_GRANT_PERCENT` pour créer un groupe de charge de travail similaire à staticrc40.   

```sql
SELECT Request_min_resource_grant_percent = Effective_request_min_resource_grant_percent
  FROM sys.dm_workload_management_workload_groups_stats
  WHERE name = 'staticrc40'
```

> [!NOTE]
> Les groupes de charges de travail fonctionnent selon le pourcentage de ressources système globales.  

Étant donné que les groupes de charges de travail fonctionnent selon le pourcentage de ressources système globales, à mesure que vous montez ou descendez en puissance, le pourcentage de ressources allouées aux classes de ressources statiques par rapport aux modifications globales des ressources système est modifié.  Par exemple, staticrc40 avec Dw1000c alloue 9,6 % des ressources système globales.  19,2 % sont alloués avec DW2000c.  Ce modèle est similaire que vous souhaitiez mettre à l’échelle à des fins concurrentielles ou allouer plus de ressources par requête.   

## <a name="create-workload-group"></a>Créer le groupe de charge de travail
Avec le `REQUEST_MIN_RESOURCE_GRANT_PERCENT` connu, vous pouvez utiliser la syntaxe <link> CREATE WORKLOAD GROUP pour créer le groupe de charge de travail.  Vous pouvez éventuellement spécifier un `MIN_PERCENTAGE_RESOURCE` supérieur à zéro pour isoler les ressources pour le groupe de charge de travail.  En outre, vous pouvez éventuellement spécifier `CAP_PERCENTAGE_RESOURCE` inférieur à 100 pour limiter la quantité de ressources que le groupe de charge de travail peut consommer.  

L’exemple ci-dessous définit la `MIN_PERCENTAGE_RESOURCE` pour dédier 9,6 % des ressources système à `wgDataLoads` et garantit qu’une requête pourra être exécutée à chaque fois.  En outre, `CAP_PERCENTAGE_RESOURCE` a la valeur 38,4 % et limite ce groupe de charge de travail à quatre requêtes simultanées.  En définissant le paramètre `QUERY_EXECUTION_TIMEOUT_SEC` sur 3 600, toute requête qui s’exécute pendant plus d’une heure est automatiquement annulée.

```sql
CREATE WORKLOAD GROUP wgDataLoads WITH  
( REQUEST_MIN_RESOURCE_GRANT_PERCENT = 9.6
 ,MIN_PERCENTAGE_RESOURCE = 9.6
 ,CAP_PERCENTAGE_RESOURCE = 38.4
 ,QUERY_EXECUTION_TIMEOUT_SEC = 3600)
```

## <a name="create-the-classifier"></a>Créer le classifieur
Précédemment, le mappage des requêtes aux classes de ressources a été effectué avec [sp_addrolemember](https://docs.microsoft.com/azure/sql-data-warehouse/resource-classes-for-workload-management#change-a-users-resource-class).  Pour obtenir les mêmes fonctionnalités et mapper les demandes aux groupes de charge de travail, utilisez la syntaxe [CREATE WORKLOAD CLASSIFIER](https://docs.microsoft.com/sql/t-sql/statements/create-workload-classifier-transact-sql) (CRÉER UN CLASSIFIEUR DE CHARGE DE TRAVAIL).  L’utilisation de sp_addrolemember vous permettait uniquement de mapper des ressources à une requête basée sur une connexion.  Un classifieur fournit des options supplémentaires en plus de la connexion, telles que :
    - label
    - session
    - temps L’exemple ci-dessous affecte des requêtes à partir de la connexion `AdfLogin` qui dispose également de l’[ÉTIQUETTE OPTION](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-develop-label) définie sur `factloads` sur le groupe de charge de travail `wgDataLoads` créé ci-dessus.

```sql
CREATE WORKLOAD CLASSIFIER wcDataLoads WITH  
( WORKLOAD_GROUP = 'wgDataLoads'
 ,MEMBERNAME = 'AdfLogin'
 ,WLM_LABEL = 'factloads')
```
## <a name="test-with-a-sample-query"></a>Tester avec un exemple de requête
Voici un exemple de requête et une requête DMV pour vérifier que le groupe de charge de travail et le classifieur sont configurés correctement.

```sql
SELECT SUSER_SNAME() --should be 'AdfLogin'

--change to a valid table AdfLogin has access to
SELECT TOP 10 * 
  FROM nation 
  OPTION (label='factloads')

SELECT request_id, [label], classifier_name, group_name, command 
  FROM sys.dm_pdw_exec_requests 
  WHERE [label] = 'factloads'
  ORDER BY submit_time DESC
```

## <a name="next-steps"></a>Étapes suivantes

Isolation de la charge de travail : lien à déterminer Comment créer un groupe de charge de travail : lien à déterminer