---
title: Haute disponibilité des machines virtuelles Azure pour SAP NW sur Windows avec Azure NetApp Files (SMB) | Microsoft Docs
description: Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur Windows avec Azure NetApp Files (SMB) pour les applications SAP
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/29/2019
ms.author: radeltch
ms.openlocfilehash: b41db629c5308348f632b3dc51c75822ba361c60
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77591351"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-windows-with-azure-netapp-filessmb-for-sap-applications"></a>Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur Windows avec Azure NetApp Files (SMB) pour les applications SAP

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md

Cet article décrit comment déployer, configurer les machines virtuelles, installer l’infrastructure de cluster et installer un système SAP NetWeaver 7.50 à haute disponibilité sur des machines virtuelles Windows, à l’aide de [SMB](https://docs.microsoft.com/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) sur [Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-introduction/).  

La couche de base de données n’est pas couverte en détail dans cet article. Nous supposons que le [réseau virtuel](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) Azure a déjà été créé.  

Commencez par lire les notes et publications SAP suivantes :

* [Documentation Azure NetApp Files][anf-azure-doc] 
* La note SAP [1928533][1928533], qui contient :  
  * une liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP
  * des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure
  * les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données
  * la version du noyau SAP requise pour Windows sur Microsoft Azure
* La note SAP [2015553][2015553] répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure.
* La note SAP [2178632][2178632] contient des informations détaillées sur toutes les métriques de surveillance rapportées pour SAP sur Azure.
* La note SAP [1999351][1999351] contient des informations de dépannage supplémentaires pour l’extension d’analyse Azure améliorée pour SAP.
* La note SAP [2287140](https://launchpad.support.sap.com/#/notes/2287140) répertorie les conditions préalables pour la fonctionnalité d’autorité de certification prise en charge par SAP du protocole SMB 3.x.
* La note SAP [2802770](https://launchpad.support.sap.com/#/notes/2802770) contient des informations sur la résolution des problèmes liés à l’exécution lente de la transaction SAP AL11 sur Windows 2012 et 2016.
* La note SAP [1911507](https://launchpad.support.sap.com/#/notes/1911507) contient des informations sur la fonctionnalité de basculement transparent pour un partage de fichiers sur Windows Server avec le protocole SMB 3.0.
* La note SAP [662452](https://launchpad.support.sap.com/#/notes/662452) offre une recommandation (désactivation de la génération de noms 8.3) pour résoudre les erreurs/performances médiocres du système de fichiers lors des accès aux données.
* [Installer la haute disponibilité SAP NetWeaver sur un cluster de basculement Windows et un partage de fichiers pour des instances SAP ASCS/SCS sur Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-installation-wsfc-file-share) 
* [Scénarios et architecture de haute disponibilité de machines virtuelles Azure pour SAP NetWeaver](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-architecture-scenarios)
* [Ajouter un port de sondage dans la configuration du cluster ASCS](sap-high-availability-installation-wsfc-file-share.md)
* [Installation d’une instance (A)SCS sur un cluster de basculement](https://www.sap.com/documents/2017/07/f453332f-c97c-0010-82c7-eda71af511fa.html)
* [Créer un volume SMB pour Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-volumes-smb#requirements-for-active-directory-connections)
* [Applications SAP NetApp su Microsoft Azure avec Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Vue d’ensemble

SAP a développé une nouvelle approche et une alternative aux disques partagés pour le clustering d’une instance SAP ASCS/SCS sur un cluster de basculement Windows. Au lieu d’utiliser des disques partagés de cluster, il est possible d’utiliser un partage de fichiers SMB pour déployer des fichiers d’hôte global SAP. Azure NetApp Files prend en charge SMBv3 (ainsi que NFS) avec la liste de contrôle d’accès NTFS à l’aide d’Active Directory. Azure NetApp Files est automatiquement hautement disponible (car il s’agit d’un service PaaS). Ces fonctionnalités font d’Azure NetApp Files l’option idéale pour héberger le partage de fichiers SMB pour SAP global.  
[Azure Active Directory (AD) Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/overview) et [Active Directory Domain Services (AD DS)](https://docs.microsoft.com/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) sont tous deux pris en charge. Vous pouvez utiliser des contrôleurs de domaine Active Directory existants avec Azure NetApp Files. Les contrôleurs de domaine peuvent figurer dans Azure en tant que machines virtuelles ou en local via ExpressRoute ou S2S VPN. Dans cet article, nous allons utiliser un contrôleur de domaine dans une machine virtuelle Azure.  
La haute disponibilité pour les services centraux SAP Netweaver nécessite un stockage partagé. Pour parvenir à cela sur Windows, il était nécessaire de créer un cluster SOFS ou d’utiliser un logiciel de disque partagé de cluster comme SIOS. Maintenant, il est possible d’atteindre la haute disponibilité SAP Netweaver à l’aide d’un stockage partagé, déployé sur Azure NetApp Files. L’utilisation d’Azure NetApp Files pour le stockage partagé élimine le besoin de recourir à SOFS ou SIOS.  

> [!NOTE]
> Le clustering d’instances SAP ASCS/SCS avec le partage de fichiers est pris en charge pour SAP NetWeaver 7.40 (et versions ultérieures), avec SAP Kernel 7.49 (et versions ultérieures).  

![Architecture à haute disponibilité SAP ASCS/SCS avec partage SMB](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb.png)

Conditions préalables pour un partage de fichiers SMB :
* Protocole SMB 3.0 (ou version ultérieure)
* Possibilité de définir les listes de contrôle d’accès (ACL, access control list) Active Directory pour les groupes d’utilisateurs Active Directory et l’objet d’ordinateur computer$.
* La haute disponibilité doit être activée pour le partage de fichiers.

Le partage pour les services centraux SAP de cette architecture de référence est proposé par Azure NetApp Files :

![Architecture à haute disponibilité SAP ASCS/SCS avec partage SMB](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-detail.png)

## <a name="create-and-mount-smb-volume-for-azure-netapp-files"></a>Créer et monter un volume SMB pour Azure NetApp Files

Procédez comme suit pour préparer l’utilisation d’Azure NetApp Files.  

1. Suivez les étapes pour vous [inscrire à Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register)  
2. Créez un compte Azure NetApp, en suivant les étapes décrites dans [Créer un compte NetApp](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-netapp-account)  
3. Configurez le pool de capacité, en suivant les instructions de la rubrique [Configurer un pool de capacité](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-set-up-capacity-pool)
4. Les ressources d’Azure NetApp Files doivent résider dans un sous-réseau délégué. Suivez les instructions de la rubrique [Déléguer un sous-réseau à Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-delegate-subnet) pour créer un sous-réseau délégué.  

> [!IMPORTANT]
> Vous devez créer des connexions Active Directory avant de créer un volume SMB. Passez en revue la [configuration requise pour les connexions Active Directory](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-volumes-smb#requirements-for-active-directory-connections).  

5. Créez la connexion Active Directory, comme décrit dans [Créer une connexion Active Directory](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-volumes-smb#create-an-active-directory-connection)  
6. Créez un volume SMB Azure NetApp Files, en suivant les instructions fournies dans [Ajouter un volume SMB](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-volumes-smb#add-an-smb-volume)  
7. Montez le volume SMB sur votre machine virtuelle Windows.

> [!TIP]
> Vous trouverez des instructions sur la façon de monter le volume Azure NetApp Files, si vous naviguez dans le [Portail Azure](https://portal.azure.com/#home) vers l’objet Azure NetApp Files, cliquez sur le panneau **Volumes**, puis **Instructions de montage**.  

## <a name="prepare-the-infrastructure-for-sap-ha-by-using-a-windows-failover-cluster"></a>Préparer l’infrastructure pour la haute disponibilité SAP à l’aide d’un cluster de basculement Windows 

1. [Définissez l’adresse IP DNS requise](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-infrastructure-wsfc-shared-disk#b22d7b3b-4343-40ff-a319-097e13f62f9e)  
2. [Définissez des adresses IP statiques pour les machines virtuelles SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-infrastructure-wsfc-shared-disk#84c019fe-8c58-4dac-9e54-173efd4b2c30).
3. [Définissez une adresse IP statique pour l’équilibrage de charge interne Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-infrastructure-wsfc-shared-disk#7a8f3e9b-0624-4051-9e41-b73fff816a9e).
4. [Définissez des règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibrage de charge interne Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-infrastructure-wsfc-shared-disk#f19bd997-154d-4583-a46e-7f5a69d0153c).
5. [Changer les règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibreur de charge interne Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-infrastructure-wsfc-shared-disk#fe0bd8b5-2b43-45e3-8295-80bee5415716).
6. [Ajoutez des machines virtuelles Windows au domaine](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-infrastructure-wsfc-shared-disk#e69e9a34-4601-47a3-a41c-d2e11c626c0c).
7. [Ajoutez des entrées de registre aux deux nœuds de cluster de l’instance SAP ASCS/SCS](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-infrastructure-wsfc-shared-disk#661035b2-4d0f-4d31-86f8-dc0a50d78158)
8. [Configurez un cluster de basculement Windows Server pour une instance SAP ASCS/SCS](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-infrastructure-wsfc-shared-disk#0d67f090-7928-43e0-8772-5ccbf8f59aab)
9. Si vous utilisez Windows Server 2016, nous vous recommandons de configurer un [témoin Azure Cloud](https://docs.microsoft.com/windows-server/failover-clustering/deploy-cloud-witness).


## <a name="install-sap-ascs-instance-on-both-nodes"></a>Installer l’instance SAP ASCS sur les deux nœuds

Vous avez besoin des logiciels suivants de SAP :
   * Outil d’installation SAP Software Provisioning Manager (SWPM) version SPS25 (ou ultérieure).
   * Noyau SAP 7.49 ou version ultérieure
   * Créez un nom d’hôte virtuel (nom de réseau en cluster) pour l’instance SAP ASCS/SCS en cluster, comme décrit dans [Créer un nom d’hôte virtuel pour l’instance SAP ASCS/SCS en cluster](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-installation-wsfc-shared-disk#a97ad604-9094-44fe-a364-f89cb39bf097).

> [!NOTE]
> Le clustering d’instances SAP ASCS/SCS avec le partage de fichiers est pris en charge pour SAP NetWeaver 7.40 (et versions ultérieures), avec SAP Kernel 7.49 (et versions ultérieures).  

### <a name="install-an-ascsscs-instance-on-the-first-ascsscs-cluster-node"></a>Installer une instance ASCS/SCS sur le premier nœud de cluster ASCS/SCS

1. Installez une instance ASCS/SCS SAP sur le premier nœud de cluster. Démarrez l’outil d’installation de SAP SWPM, puis accédez à : **Product (Produit)**  > **DBMS (SGBD)** > Installation > Application Server ABAP (ou Java) > High-Availability System (Système haute disponibilité) > ASCS/SCS instance (Instance ASCS/SCS) > First cluster node (Premier nœud de cluster).  

2. Sélectionnez **File Share Cluster (Cluster de partage de fichiers)** en tant que configuration de partage de cluster dans SWPM.  
3. Lorsque vous y êtes invité à l’étape **SAP System Cluster Parameters (Paramètres du cluster du système SAP)** , entrez le nom d’hôte du partage SMB Azure NetApp Files que vous avez déjà créé comme **File Share Host Name (Nom d’hôte du partage de fichiers)** .  Dans cet exemple, le nom d’hôte du partage SMB est **anfsmb-9562**. 

> [!IMPORTANT]
> Si les résultats de l’outil de vérification de la configuration requise dans SWPM indiquent que la fonctionnalité de disponibilité continue n’est pas remplie, vous pouvez résoudre ce problème en suivant les instructions figurant dans [Retard de message d’erreur lorsque vous essayez d’accéder à un dossier partagé qui n’existe plus dans Windows](https://support.microsoft.com/help/2820470/delayed-error-message-when-you-try-to-access-a-shared-folder-that-no-l).  

> [!TIP]
> Si les résultats de l’outil de vérification de la configuration requise dans SWPM indiquent qu’une condition de taille d’échange n’est pas remplie, vous pouvez ajuster la taille d’échange en accédant à Poste de travail > Propriétés système > Paramètres de performances > Options avancées > Mémoire virtuelle > Modifier.  

4. Configurez une ressource de cluster SAP, le port de sonde `SAP-SID-IP`, à l’aide de PowerShell. Exécutez cette configuration sur l’un des nœuds de cluster SAP ASCS/SCS, comme décrit dans [Configurer le port de sonde](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-installation-wsfc-shared-disk#10822f4f-32e7-4871-b63a-9b86c76ce761).

### <a name="install-an-ascsscs-instance-on-the-second-ascsscs-cluster-node"></a>Installer une instance ASCS/SCS sur le deuxième nœud de cluster ASCS/SCS

1. Installez une instance ASCS/SCS SAP sur le deuxième nœud de cluster. Démarrez l’outil d’installation SAP SWPM, puis accédez à **Product (Produit)**  > **DBMS (SGBD)** > Installation > Application Server ABAP (ou Java) > High-Availability System (Système à haute disponibilité) > ASCS/SCS instance (Instance ASCS/SCS) > Additional cluster node (Noeud de cluster supplémentaire).  

### <a name="install-a-dbms-instance-and-sap-application-servers"></a>Installer une instance de SGBD et les serveurs d’applications SAP

Finalisez votre installation SAP en installant :

   * Une instance de SGBD  
   * Un serveur d’applications SAP principal  
   * Un serveur d’applications SAP supplémentaire  

## <a name="test-the-sap-ascsscs-instance-failover"></a>Testez le basculement de l’instance SAP ASCS/SCS 

### <a name="fail-over-from-cluster-node-a-to-cluster-node-b-and-back"></a>Basculement du nœud de cluster A vers le nœud de cluster B et inversément
Dans ce scénario de test, nous faisons référence au nœud de cluster sapascs1 comme nœud A et au nœud de cluster sapascs2 en tant que nœud B.

1. Vérifiez que les ressources du cluster s’exécutent sur le nœud A. ![Figure 1 : Ressources du cluster de basculement Windows Server exécutées sur le nœud A avant le test de basculement](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-1.png)  

2. Redémarrez le nœud de cluster A. Les ressources de cluster SAP sont déplacées vers le nœud de cluster B. ![Figure 2 : Ressources du cluster de basculement Windows Server exécutées sur le nœud B après le test de basculement](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-2.png)  


## <a name="lock-entry-test"></a>Test d’entrée de verrou

1. Vérifiez que le serveur ERS (Enqueue Replication Server) est actif  
2. Connectez-vous au système SAP, exécutez la transaction SU01 et ouvrez un ID utilisateur en mode modification. Cela génère une entrée de verrou SAP.  
3. Lorsque vous êtes connecté au système SAP, affichez l’entrée de verrou, en accédant à la transaction ST12.  
4. Basculez les ressources ASCS du nœud de cluster A vers le nœud de cluster B.  
5. Vérifiez que l’entrée de verrou, générée avant le basculement des ressources de cluster SAP ASCS/SCS, est conservée.  

![Figure 3 : L’entrée de verrou est conservée après le test de basculement](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-3.png)  

Pour plus d’informations, consultez [Troubleshooting for Enqueue Failover in ASCS with ERS](https://wiki.scn.sap.com/wiki/display/SI/Troubleshooting+for+Enqueue+Failover+in+ASCS+with+ERS) (en anglais)
## <a name="next-steps"></a>Étapes suivantes

* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* Pour découvrir comment établir la haute disponibilité et planifier la récupération d’urgence de SAP 
* HANA sur Azure (grandes instances), consultez [Haute disponibilité et récupération d’urgence SAP HANA (grandes instances) sur Azure](hana-overview-high-availability-disaster-recovery.md).
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur des machines virtuelles Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha]
