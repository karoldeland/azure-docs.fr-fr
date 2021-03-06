---
title: Mise à niveau du système d’exploitation pour SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Effectuer la mise à niveau du système d’exploitation pour SAP HANA sur Azure (grandes instances)
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/04/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 3a0a5d39a7cb2162186291ea534a623ef45c40d4
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78675631"
---
# <a name="operating-system-upgrade"></a>Mise à niveau du système d’exploitation
Ce document décrit en détail les mises à niveau du système d’exploitation sur les grandes instances HANA.

>[!NOTE]
>La mise à niveau du système d’exploitation relève de la responsabilité du client. Le support des opérations Microsoft peut vous indiquer les éléments clés à surveiller pendant la mise à niveau. Consultez également le fournisseur de votre système d’exploitation avant de planifier une mise à niveau.

Pendant le provisionnement de l’unité HLI, l’équipe des opérations Microsoft installe le système d’exploitation.
Au fil du temps, il vous appartient d’effectuer la maintenance du système d’exploitation sur l’unité HLI : mise à jour corrective, réglage, mise à niveau, etc.

Avant d’apporter des modifications majeures au système d’exploitation (par exemple, une mise à niveau de SP1 vers SP2), vous devez contacter l’équipe des opérations de Microsoft en ouvrant un ticket de support.

Veuillez inclure dans votre ticket :

* Votre ID d’abonnement HLI.
* Le nom de votre serveur.
* Le niveau de correctif que vous envisagez d'appliquer.
* La date à laquelle vous envisagez cette modification. 

Nous vous conseillons d'ouvrir ce ticket au moins une semaine avant la date de mise à niveau souhaitée pour permettre à l'équipe de vérifier si une mise à niveau du microprogramme est requise dans le panneau de votre serveur.


Pour consulter la matrice de prise en charge des différentes versions SAP HANA avec les différentes versions Linux, reportez-vous à [Remarque SAP n° 2235581](https://launchpad.support.sap.com/#/notes/2235581).


## <a name="known-issues"></a>Problèmes connus

Voici quelques problèmes affectant couramment la mise à niveau :
- Dans la référence SKU de classe de type II, le logiciel SFS est supprimé après la mise à niveau du système d’exploitation. Vous devez réinstaller le logiciel SFS compatible après la mise à niveau du système d’exploitation.
- Une version antérieure des pilotes de cartes Ethernet (ENIC et FNIC) est restaurée. Vous devez réinstaller la version compatible des pilotes après la mise à niveau.

## <a name="sap-hana-large-instance-type-i-recommended-configuration"></a>Configuration recommandée pour les grandes instances SAP HANA (type I)

Au fil du temps, la configuration du système d’exploitation peut s’écarter des paramètres recommandés en raison des mises à jour correctives, des mises à niveau du système et des modifications apportées par les clients. De plus, Microsoft identifie les mises à jour nécessaires pour que les systèmes existants soient configurés de manière optimale pour offrir les meilleurs niveaux de performance et de résilience possibles. Les instructions suivantes décrivent les suggestions relatives au niveau de performance réseau, à la stabilité du système et au niveau de performance optimal de HANA.

### <a name="compatible-enicfnic-driver-versions"></a>Versions compatibles des pilotes eNIC/fNIC
  Pour bénéficier d’un bon niveau de performance réseau et d’une stabilité du système adéquate, il est recommandé de vérifier que la version propre au système d’exploitation des pilotes eNIC et fNIC est installée comme indiqué dans la tableau de compatibilité suivant. Les serveurs sont fournis aux clients avec des versions compatibles. Dans certains cas, notez que la mise à jour corrective du système d’exploitation et/ou du noyau peut entraîner la restauration de la version par défaut des pilotes. Vérifiez que les pilotes s’exécutant après les opérations de mise à jour corrective du système d’exploitation et/ou du noyau correspondent aux bonnes versions.
       
      
  |  Fournisseur du système d’exploitation    |  Version du package du système d’exploitation     |  Version du microprogramme  |  Pilote eNIC |  Pilote fNIC | 
  |---------------|-------------------------|--------------------|--------------|--------------|
  |   SuSE        |  SLES 12 SP2            |   3.1.3h           |  2.3.0.40    |   1.6.0.34   |
  |   SuSE        |  SLES 12 SP3            |   3.1.3h           |  2.3.0.44    |   1.6.0.36   |
  |   Red Hat     |  RHEL 7.2               |   3.1.3h           |  2.3.0.39    |   1.6.0.34   |
 

### <a name="commands-for-driver-upgrade-and-to-clean-old-rpm-packages"></a>Commandes de mise à niveau des pilotes et de nettoyage des anciens packages rpm
```
rpm -U driverpackage.rpm
rpm -e olddriverpackage.rpm
```

#### <a name="commands-to-confirm"></a>Commandes à confirmer
```
modinfo enic
modinfo fnic
```

### <a name="suse-hlis-grub-update-failure"></a>Échec de la mise à jour de GRUB pour les HLI SuSE
Les grandes instances SAP HANA sur Azure (type I) peuvent être dans un état qui les empêchent de démarrer après une mise à niveau. La procédure ci-dessous corrige ce problème.
#### <a name="execution-steps"></a>Étapes d’exécution


*   Exécutez la commande `multipath -ll`.
*   Obtenez l’ID du numéro d’unité logique (« LUN ID ») dont la taille avoisine 50G ou utilisez la commande `fdisk -l | grep mapper`
*   Mettez à jour le fichier `/etc/default/grub_installdevice` avec la ligne `/dev/mapper/<LUN ID>`. Exemple :/dev/mapper/3600a09803830372f483f495242534a56
>[!NOTE]
>L’ID du numéro d’unité logique varie d’un serveur à l’autre.


### <a name="disable-edac"></a>Désactiver EDAC 
   Le module EDAC (Error Detection And Correction) vise à détecter et corriger les erreurs de mémoire. Or, le matériel sous-jacent pour les grandes instances SAP HANA sur Azure (type I) assure déjà la même fonction. Le fait que la même fonctionnalité soit activée au niveau du matériel et du système d’exploitation peut occasionner des conflits et entraîner l’arrêt occasionnel et inopiné du serveur. Par conséquent, il est recommandé de désactiver le module du système d’exploitation.

#### <a name="execution-steps"></a>Étapes d’exécution

* Vérifiez si le module EDAC est activé. Si une sortie est retournée par la commande ci-dessous, cela signifie que le module est activé. 
```
lsmod | grep -i edac 
```
* Désactivez les modules en ajoutant les lignes suivantes au fichier `/etc/modprobe.d/blacklist.conf`
```
blacklist sb_edac
blacklist edac_core
```
Un redémarrage est nécessaire pour que les modifications soient prises en compte. Exécutez la commande `lsmod` et vérifiez que le module n’est pas présent dans la sortie.


### <a name="kernel-parameters"></a>Paramètres de noyau
   Vérifiez que les paramètres `transparent_hugepage`, `numa_balancing`, `processor.max_cstate`, `ignore_ce` et `intel_idle.max_cstate` sont correctement définis.

* intel_idle.max_cstate=1
* processor.max_cstate=1
* transparent_hugepage=never
* numa_balancing=disable
* mce=ignore_ce


#### <a name="execution-steps"></a>Étapes d’exécution

* Ajoutez ces paramètres à la ligne `GRB_CMDLINE_LINUX` du fichier `/etc/default/grub`
```
intel_idle.max_cstate=1 processor.max_cstate=1 transparent_hugepage=never numa_balancing=disable mce=ignore_ce
```
* Créez un nouveau fichier grub.
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```
* Relancez le système.


## <a name="next-steps"></a>Étapes suivantes
- Pour la classe de références SKU de type I de sauvegarde du système d’exploitation, consultez [Sauvegarde et restauration](hana-overview-high-availability-disaster-recovery.md).
- Pour la classe SKU Type II, consultez [Sauvegarde du système d’exploitation pour les SKU Type II des tampons Révision 3](os-backup-type-ii-skus.md).
