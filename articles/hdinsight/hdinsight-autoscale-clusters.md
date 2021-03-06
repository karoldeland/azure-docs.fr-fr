---
title: Mettre à l’échelle automatiquement les clusters Azure HDInsight
description: Utiliser la fonction Azure HDInsight de mise à l’échelle automatique pour les clusters Apache Hadoop
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 03/05/2020
ms.openlocfilehash: 68bc30d08d95fe8e3d20a8ecb7af6c9710951921
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78399712"
---
# <a name="automatically-scale-azure-hdinsight-clusters"></a>Mettre à l’échelle automatiquement les clusters Azure HDInsight

> [!Important]
> La fonctionnalité de mise à l’échelle automatique d’Azure HDInsight a été mise à la disposition générale le 7 novembre 2019 pour les clusters Spark et Hadoop. Elle incluait des améliorations non disponibles dans la préversion de la fonctionnalité. Si vous avez créé un cluster Spark avant le 7 novembre 2019 et que vous souhaitez utiliser la fonctionnalité de mise à l’échelle automatique dessus, l’approche recommandée consiste à créer un nouveau cluster et à activer la mise à l’échelle automatique sur le nouveau cluster.
>
> La mise à l’échelle automatique pour les clusters Interactive Query (LLAP) et HBase est toujours en préversion. La mise à l’échelle automatique est disponible uniquement sur les clusters Spark, Hadoop, Interactive Query et HBase.

La fonctionnalité de mise à l’échelle automatique de cluster d’Azure HDInsight met automatiquement à l’échelle le nombre de nœuds Worker dans un cluster. Les autres types de nœuds du cluster ne peuvent pas être mis à l’échelle actuellement.  Lors de la création d’un cluster HDInsight, il est possible de définir un nombre minimum et un nombre maximum de nœuds Worker. La mise à l’échelle automatique surveille ensuite les besoins en ressources de la charge analytique et augmente ou diminue le nombre de nœuds Worker. L’utilisation de cette fonctionnalité n’entraîne aucun coût supplémentaire.

## <a name="cluster-compatibility"></a>Compatibilité du cluster

Le tableau suivant décrit les types de cluster et les versions qui sont compatibles avec la fonctionnalité de mise à l’échelle automatique.

| Version | Spark | Hive | LLAP | hbase | Kafka | Storm | ML |
|---|---|---|---|---|---|---|---|
| HDInsight 3.6 sans ESP | Oui | Oui | Oui | Oui* | Non | Non | Non |
| HDInsight 4.0 sans ESP | Oui | Oui | Oui | Oui* | Non | Non | Non |
| HDInsight 3.6 avec ESP | Oui | Oui | Oui | Oui* | Non | Non | Non |
| HDInsight 4.0 avec ESP | Oui | Oui | Oui | Oui* | Non | Non | Non |

\* Les clusters HBase peuvent uniquement être configurés pour une mise à l’échelle basée sur la planification, et non basée sur la charge.

## <a name="how-it-works"></a>Fonctionnement

Vous pouvez choisir entre une mise à l’échelle de votre cluster HDInsight basée sur la charge ou sur la planification. Une mise à l’échelle basée sur la charge modifie le nombre de nœuds dans votre cluster, selon une plage que vous définissez, pour assurer une utilisation optimale du processeur et réduire les coûts d’exécution.

Une mise à l’échelle basée sur la planification modifie le nombre de nœuds de votre cluster selon des conditions prenant effet à des moments donnés. Ces conditions mettent le cluster à l’échelle vers le nombre de nœuds de votre choix.

### <a name="metrics-monitoring"></a>Supervision des métriques

La mise à l’échelle automatique supervise en permanence le cluster et collecte les métriques suivantes :

|Métrique|Description|
|---|---|
|Total Pending CPU|Nombre total de cœurs nécessaires pour commencer l’exécution de tous les conteneurs en attente.|
|Total Pending Memory|Mémoire totale (en Mo) nécessaire pour commencer l’exécution de tous les conteneurs en attente.|
|Total Free CPU|Somme de tous les cœurs inutilisés sur les nœuds Worker actifs.|
|Total Free Memory|Somme de la mémoire inutilisée (en Mo) sur les nœuds Worker actifs.|
|Used Memory per Node|Charge sur un nœud Worker. Un nœud Worker sur lequel 10 Go de mémoire sont utilisés est considéré comme étant plus sollicité qu’un nœud avec 2 Go de mémoire utilisés.|
|Number of Application Masters per Node|Nombre de conteneurs Application Master (AM) en cours d’exécution sur un nœud Worker. Un nœud Worker hébergeant 2 conteneurs AM est considéré comme plus important qu’un nœud Worker hébergeant 0 conteneur AM.|

Les métriques ci-dessus sont contrôlées toutes les 60 secondes. La fonction de mise à l’échelle automatique prend des décisions de montée en puissance ou de descente en puissance en fonction de ces métriques.

### <a name="load-based-scale-conditions"></a>Conditions de mise à l’échelle basée sur la charge

Lorsque les conditions suivantes sont détectées, la mise à l’échelle automatique émet une requête de mise à l’échelle :

|Scale-up|Scale-down|
|---|---|
|« Total pending CPU » est supérieur à la valeur de « Total free CPU » pendant plus de 3 minute.|« Total pending CPU » est inférieur à la valeur de « Total free CPU » pendant plus de 10 minutes.|
|« Total pending memory » est supérieur à la valeur de « Total free memory » pendant plus de 3 minute.|« Total pending memory » est inférieur à la valeur de « Total free memory » pendant plus de 10 minutes.|

Pour un scale-up, le service HDInsight calcule combien de nouveaux nœuds Worker sont utiles pour répondre aux besoins actuels en UC et en mémoire, puis il émet une requête de scale-up pour ajouter le nombre de nœuds nécessaires.

Pour un scale-down, selon le nombre de conteneurs AM par nœud et les besoins actuels en UC et en mémoire, la fonction de mise à l’échelle automatique émet une requête de suppression d’un certain nombre de nœuds. Le service détecte également les nœuds à supprimer en fonction de l’exécution des travaux en cours. L’opération de descente en puissance désactive tout d’abord les nœuds, puis les supprime du cluster.

## <a name="get-started"></a>Bien démarrer

### <a name="create-a-cluster-with-load-based-autoscaling"></a>Créer un cluster avec une mise à l’échelle automatique basée sur la charge

Pour activer la fonctionnalité de mise à l’échelle automatique basée sur la charge, procédez comme suit dans le cadre d’un processus normal de création de cluster :

1. Sous l’onglet **Configuration + tarification**, cochez la case **Activer la mise à l’échelle automatique**.
1. Sélectionnez **Basé sur la charge** sous **Type de mise à l’échelle automatique**.
1. Entrez les valeurs souhaitées pour les propriétés suivantes :  

    * **Nombre de nœuds** initial pour le **Nœud Worker**.
    * Nombre **min** de nœuds Worker.
    * Nombre **max** de nœuds Worker.

    ![Activation de l’option de mise à l’échelle automatique basée sur la charge du nœud Worker](./media/hdinsight-autoscale-clusters/azure-portal-cluster-configuration-pricing-autoscale.png)

Le nombre initial de nœuds Worker doit se situer entre le minimum et le maximum inclus. Cette valeur définit la taille initiale du cluster lors de sa création. Le nombre minimal de nœuds Worker doit être défini sur au moins trois. La mise à l’échelle de votre cluster à moins de trois nœuds peut entraîner le blocage en mode sans échec en raison d’une réplication de fichiers insuffisante.  Pour plus d’informations, consultez [Blocage en mode sans échec](./hdinsight-scaling-best-practices.md#getting-stuck-in-safe-mode).

### <a name="create-a-cluster-with-schedule-based-autoscaling"></a>Créer un cluster avec une mise à l’échelle automatique basée sur la planification

Pour activer la fonctionnalité de mise à l’échelle automatique basée sur la planification, procédez comme suit dans le cadre d’un processus normal de création de cluster :

1. Sous l’onglet **Configuration + tarification**, activez la case à cocher **Activer la mise à l’échelle automatique**.
1. Entrez le **Nombre de nœuds** pour le **Nœud Worker**, qui contrôle la limite de la mise à l’échelle du cluster.
1. Sélectionnez l’option **Basée sur la planification** sous **Type de mise à l’échelle**.
1. Sélectionnez **Configurer** pour ouvrir la fenêtre **Configuration de la mise à l’échelle automatique**.
1. Sélectionnez votre fuseau horaire, puis cliquez sur **+ Ajouter une condition**
1. Sélectionnez les jours de la semaine pour lesquels la nouvelle condition doit s’appliquer.
1. Modifiez l’heure à laquelle la condition doit prendre effet, ainsi que le nombre de nœuds du cluster à mettre à l’échelle.
1. Ajoutez d’autres conditions si nécessaire.

    ![Activation de l’option de création d’un nœud Worker basée sur la planification](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-schedule-creation.png)

Le nombre de nœuds doit être compris entre 3 et le nombre de nœuds Worker maximal que vous avez entré avant d’ajouter des conditions.

### <a name="final-creation-steps"></a>Dernières étapes de la création

Pour les mises à l’échelle basées sur la charge et la planification, sélectionnez le type de machine virtuelle pour les nœuds Worker en sélectionnant une machine virtuelle dans la liste déroulante sous **Taille du nœud**. Après avoir sélectionné le type de machine virtuelle pour chaque type de nœud, vous pouvez voir la fourchette de coûts estimée pour l’ensemble du cluster. Ajustez les types de machine virtuelle selon votre budget.

![Activation de l’option de mise à l’échelle d’un nœud worker basée sur la planification - Taille du nœud](./media/hdinsight-autoscale-clusters/azure-portal-cluster-configuration-pricing-vmsize.png)

Votre abonnement a un quota de capacité pour chaque région. Le nombre total de cœurs de vos nœuds principaux, combiné au nombre maximum de nœuds Worker ne peut dépasser le quota de capacité. Toutefois, ce quota est une limite logicielle ; vous pouvez toujours créer un ticket de support pour l’augmenter aisément.

> [!Note]  
> Si vous dépassez la limite totale de quota de base, vous recevrez un message d’erreur disant indiquant que le nœud maximum a dépassé les noyaux disponibles dans cette région et que vous devez choisir une autre région ou contacter le support pour augmenter le quota (« the maximum node exceeded the available cores in this region, please choose another region or contact the support to increase the quota »).

Pour plus d’informations sur la création de clusters HDInsight à l’aide du portail Azure, consultez [Créer des clusters Linux dans HDInsight à l’aide du portail Azure](hdinsight-hadoop-create-linux-clusters-portal.md).  

### <a name="create-a-cluster-with-a-resource-manager-template"></a>Création d’un cluster avec un modèle Resource Manager

#### <a name="load-based-autoscaling"></a>Mise à l’échelle automatique basée sur la charge

Vous pouvez créer un cluster HDInsight avec la mise à l’échelle basée sur la charge d’un modèle Azure Resource Manager en ajoutant un nœud `autoscale` à la section `computeProfile` > `workernode` avec les propriétés `minInstanceCount` et `maxInstanceCount`, comme indiqué dans l’extrait de code json indiqué ci-dessous.

```json
{
  "name": "workernode",
  "targetInstanceCount": 4,
  "autoscale": {
      "capacity": {
          "minInstanceCount": 3,
          "maxInstanceCount": 10
      }
  },
  "hardwareProfile": {
      "vmSize": "Standard_D13_V2"
  },
  "osProfile": {
      "linuxOperatingSystemProfile": {
          "username": "[parameters('sshUserName')]",
          "password": "[parameters('sshPassword')]"
      }
  },
  "virtualNetworkProfile": null,
  "scriptActions": []
}
```

Pour plus d’informations sur la création de clusters avec des modèles Resource Manager, consultez [Création de clusters Apache Hadoop dans HDInsight à l’aide de modèles Azure Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md).  

#### <a name="schedule-based-autoscaling"></a>Mise à l’échelle automatique basée sur la planification

Vous pouvez créer un cluster HDInsight avec la mise à l’échelle basée sur la planification d’un modèle Azure Resource Manager en ajoutant un nœud `autoscale` à la section `computeProfile` > `workernode`. Le nœud `autoscale` contient un élément `recurrence` qui a un élément `timezone` et un élément `schedule`. Ils vous permettent d’indiquer quand la modification doit avoir lieu.

```json
{
  "autoscale": {
    "recurrence": {
      "timeZone": "Pacific Standard Time",
      "schedule": [
        {
          "days": [
            "Monday",
            "Tuesday",
            "Wednesday",
            "Thursday",
            "Friday"
          ],
          "timeAndCapacity": {
            "time": "11:00",
            "minInstanceCount": 10,
            "maxInstanceCount": 10
          }
        },
      ]
    }
  },
  "name": "workernode",
  "targetInstanceCount": 4,
}
```

### <a name="enable-and-disable-autoscale-for-a-running-cluster"></a>Activation et désactivation de la mise à l’échelle automatique d’un cluster en cours d’exécution

#### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Pour activer la mise à l’échelle automatique sur un cluster en cours d’exécution, sélectionnez **Taille du cluster** sous **Paramètres**. Ensuite, sélectionnez **Activer la mise à l’échelle automatique**. Choisissez un type de mise à l’échelle automatique, puis entrez les options de mise à l’échelle basée sur la planification ou la charge. Enfin, sélectionnez **Enregistrer**.

![Activation de l’option de mise à l’échelle d’un nœud worker basée sur la planification - Cluster en cours d’exécution](./media/hdinsight-autoscale-clusters/azure-portal-settings-autoscale.png)

#### <a name="using-the-rest-api"></a>Utilisation de l’API REST

Pour activer ou désactiver la mise à l’échelle automatique sur un cluster en cours d’exécution à l’aide de l’API REST, envoyez une requête POST au point de terminaison de la mise à l’échelle automatique comme indiqué dans l’extrait de code ci-dessous :

```
https://management.azure.com/subscriptions/{subscription Id}/resourceGroups/{resourceGroup Name}/providers/Microsoft.HDInsight/clusters/{CLUSTERNAME}/roles/workernode/autoscale?api-version=2018-06-01-preview
```

Utilisez les paramètres appropriés dans la charge utile de la requête. La charge utile JSON ci-dessous peut être utilisée pour activer la mise à l’échelle automatique. Utilisez la charge utile `{autoscale: null}` pour désactiver la mise à l’échelle automatique.

```json
{ autoscale: { capacity: { minInstanceCount: 3, maxInstanceCount: 2 } } }
```

Consultez la section précédente sur l’[activation de la mise à l’échelle automatique basée sur la charge](#load-based-autoscaling) pour obtenir une description complète de tous les paramètres de charge utile.

## <a name="best-practices"></a>Meilleures pratiques

### <a name="choosing-load-based-or-schedule-based-scaling"></a>Choix de la mise à l’échelle basée sur la planification ou la charge

Avant de prendre une décision sur le mode à choisir, tenez compte des facteurs suivants :

* Activez la mise à l’échelle automatique lors de la création du cluster.
* Le nombre minimal de nœuds doit être au moins égal à trois.
* Variance de la charge : la charge du cluster suit un modèle cohérent à des moments spécifiques, des jours spécifiques. Si ce n’est pas le cas, nous vous recommandons d’opter pour une mise à l’échelle basée sur la charge.
* Exigences du Contrat de niveau de service : La mise à l’échelle automatique est réactive, et non pas prédictive. Le délai entre le début de l’augmentation de la charge et le moment où le cluster doit atteindre sa taille prévue est-il suffisant ? S’il existe des exigences de contrat SLA strictes et que la charge est un modèle connu fixe, nous vous recommandons d’opter pour une mise à l’échelle basée sur la planification.

### <a name="consider-the-latency-of-scale-up-or-scale-down-operations"></a>Prendre en compte la latence de la montée ou de la descente en puissance

Une opération de mise à l’échelle peut prendre entre 10 et 20 minutes. Intégrez ce délai lorsque vous configurez une planification personnalisée. Par exemple, si vous avez besoin que la taille du cluster soit de 20 à 09h00, définissez le déclencheur de planification sur une heure antérieure, comme 08h30, pour que l’opération de mise à l’échelle se termine à 09h00.

### <a name="preparation-for-scaling-down"></a>Préparation pour la descente en puissance

Au cours de descente en puissance du cluster, la mise à l’échelle automatique désactivera les nœuds pour atteindre à la taille cible. Si des tâches sont en cours d’exécution sur ces nœuds, la mise à l’échelle automatique attendra jusqu’à ce qu’elles soient terminées. Dans la mesure où chaque nœud Worker joue également un rôle dans HDFS, les données temporaires seront décalées vers les nœuds restants. Par conséquent, nous vous conseillons de vérifier que les nœuds restants disposent d’assez d’espace de stockage pour héberger toutes les données temporaires.

Les travaux en cours d’exécution continueront jusqu’à leur conclusion. Les travaux en attente attendrons d’être planifiée comme d’habitude avec moins de nœuds Worker disponibles.

### <a name="minimum-cluster-size"></a>Taille minimale du cluster

Ne mettez pas à l’échelle votre cluster à moins de trois nœuds. La mise à l’échelle de votre cluster à moins de trois nœuds peut entraîner le blocage en mode sans échec en raison d’une réplication de fichiers insuffisante.  Pour plus d’informations, consultez [Blocage en mode sans échec](./hdinsight-scaling-best-practices.md#getting-stuck-in-safe-mode).

## <a name="monitoring"></a>Surveillance

### <a name="cluster-status"></a>État du cluster

L’état du cluster répertorié dans le Portail Microsoft Azure peut vous aider à surveiller les activités de mise à l’échelle automatique.

![Activation de l’option de mise à l’échelle automatique basée sur la charge du nœud worker - État du cluster](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-cluster-status.png)

La liste ci-dessous présente les messages d’état de cluster susceptibles de s’afficher.

| État du cluster | Description |
|---|---|
| Exécution en cours | Le cluster fonctionne normalement. Toutes les activités de mise à l’échelle automatique précédentes sont réussies. |
| Mise à jour  | La configuration de la mise à l’échelle automatique du cluster est en cours de mise à jour.  |
| Configuration de HDInsight  | Une opération de montée ou de descente en puissance de cluster est en cours.  |
| Erreur de mise à jour  | HDInsight a rencontré des problèmes pendant la mise à jour de la configuration de la mise à l’échelle automatique. Les clients peuvent choisir de réessayer la mise à jour ou de désactiver la mise à l’échelle automatique.  |
| Error  | Une erreur est survenue dans le cluster. Ce dernier n’est plus utilisable. Supprimez ce cluster et créez-en un autre.  |

Pour afficher le nombre actuel de nœuds dans votre cluster, accédez au graphique **Taille du cluster** dans la page **Vue d’ensemble** de votre cluster, ou sélectionnez **Taille du cluster** sous **Paramètres**.

### <a name="operation-history"></a>Historique de l’opération

Vous pouvez afficher l’historique de Scale up/Scale down du cluster dans le cadre des métriques de celui-ci. Vous pouvez également dresser la liste de toutes les actions de mise à l’échelle au cours de la journée, de la semaine ou d’une autre période.

Sous **Supervision**, **Métriques**. Ensuite, sélectionnez **Ajouter une métrique** et **Nombre de Workers actifs** dans la zone de liste déroulante **Métrique**. Sélectionnez le bouton en haut à droite pour modifier l’intervalle de temps.

![Activation de l’option de mise à l’échelle d’un nœud worker basée sur la planification - Métrique](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-chart-metric.png)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les meilleures pratiques de mise à l’échelle manuelle des clusters, lisez [Scaling best practices](hdinsight-scaling-best-practices.md) (Meilleures pratiques de la mise à l’échelle).
