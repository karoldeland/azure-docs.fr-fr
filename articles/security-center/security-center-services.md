---
title: Fonctionnalités prises en charge disponibles dans Azure Security Center | Microsoft Docs
description: Ce document fournit la liste des services pris en charge par Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 870ebc8d-1fad-435b-9bf9-c477f472ab17
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/01/2020
ms.author: memildin
ms.openlocfilehash: 9d3fa1e0b62ea6f4762c3df6ac7da310d5703807
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79225241"
---
# <a name="feature-coverage-for-machines"></a>Couverture des fonctionnalités pour les machines

Les tableaux ci-dessous présentent les fonctionnalités Azure Security Center disponibles pour les machines virtuelles et les serveurs.

## <a name="supported-features-for-virtual-machines-and-servers"></a>Fonctionnalités prises en charge pour les machines virtuelles et les serveurs <a name="vm-server-features"></a>

### <a name="windows-machines"></a>[Ordinateurs Windows](#tab/features-windows)

|||||||||
|----|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|**Fonctionnalité**|**Machines virtuelles Azure**|**Groupes de machines virtuelles identiques Azure**|**Machines non-Azure**|**Tarification**
|[Intégration de Microsoft Defender ATP](security-center-wdatp.md)|✔</br>(sur les versions prises en charge)|✔</br>(sur les versions prises en charge)|✔|standard|
|[Analytique comportementale des machines virtuelles (et alertes de sécurité)](threat-protection.md)|✔|✔|✔|Recommandations (Gratuit) </br></br> Alertes de sécurité (Standard)|
|[Alertes de sécurité sans fichier](alerts-reference.md#alerts-windows)|✔|✔|✔|standard|
|[Alertes de sécurité réseau](threat-protection.md#network-layer)|✔|✔|-|standard|
|[Accès juste-à-temps aux machines virtuelles](security-center-just-in-time.md)|✔|-|-|standard|
|[Évaluation native des vulnérabilités](built-in-vulnerability-assessment.md)|✔|-|-|standard|
|[Supervision de l’intégrité des fichiers](security-center-file-integrity-monitoring.md)|✔|✔|✔|standard|
|[Contrôles d’application adaptative](security-center-adaptive-application.md)|✔|-|✔|standard|
|[Mappage réseau](security-center-network-recommendations.md#network-map)|✔|✔|-|standard|
|[Durcissement réseau adaptatif](security-center-adaptive-network-hardening.md)|✔|-|-|standard|
|Contrôles réseau adaptatifs|✔|✔|-|standard|
|[Tableau de bord et rapports de conformité à la réglementation](security-center-compliance-dashboard.md)|✔|✔|✔|standard|
|Recommandations et protection contre les menaces sur les conteneurs IaaS hébergés dans Docker|-|-|-|standard|
|Évaluation des correctifs de système d’exploitation manquants|✔|✔|✔|Gratuit|
|Évaluation des erreurs de configuration de la sécurité|✔|✔|✔|Gratuit|
|[Évaluation de la protection des points de terminaison](security-center-services.md#supported-endpoint-protection-solutions-)|✔|✔|✔|Gratuit|
|Évaluation du chiffrement des disques|✔|✔|-|Gratuit|
|Évaluation des vulnérabilités tierces|✔|-|-|Gratuit|
|[Évaluation de la sécurité réseau](security-center-network-recommendations.md)|✔|✔|-|Gratuit|


### <a name="linux-machines"></a>[Ordinateurs Linux](#tab/features-linux)

|||||||||
|----|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|**Fonctionnalité**|**Machines virtuelles Azure**|**Groupes de machines virtuelles identiques Azure**|**Machines non-Azure**|**Tarification**
|[Intégration de Microsoft Defender ATP](security-center-wdatp.md)|-|-|-|standard|
|[Analytique comportementale des machines virtuelles (et alertes de sécurité)](security-center-alerts-iaas.md)|✔</br>(sur les versions prises en charge)|✔</br>(sur les versions prises en charge)|✔|Recommandations (Gratuit) </br></br> Alertes de sécurité (Standard)|
|[Alertes de sécurité sans fichier](alerts-reference.md#alerts-windows)|-|-|-|standard|
|[Alertes de sécurité réseau](threat-protection.md#network-layer)|✔|✔|-|standard|
|[Accès juste-à-temps aux machines virtuelles](security-center-just-in-time.md)|✔|-|-|standard|
|[Évaluation native des vulnérabilités](built-in-vulnerability-assessment.md)|✔|-|-|standard|
|[Supervision de l’intégrité des fichiers](security-center-file-integrity-monitoring.md)|✔|✔|✔|standard|
|[Contrôles d’application adaptative](security-center-adaptive-application.md)|✔|-|✔|standard|
|[Mappage réseau](security-center-network-recommendations.md#network-map)|✔|✔|-|standard|
|[Durcissement réseau adaptatif](security-center-adaptive-network-hardening.md)|✔|-|-|standard|
|Contrôles réseau adaptatifs|✔|✔|-|standard|
|[Tableau de bord et rapports de conformité à la réglementation](security-center-compliance-dashboard.md)|✔|✔|✔|standard|
|Recommandations et protection contre les menaces sur les conteneurs IaaS hébergés dans Docker|✔|✔|✔|standard|
|Évaluation des correctifs de système d’exploitation manquants|✔|✔|✔|Gratuit|
|Évaluation des erreurs de configuration de la sécurité|✔|✔|✔|Gratuit|
|[Évaluation de la protection des points de terminaison](security-center-services.md#supported-endpoint-protection-solutions-)|-|-|-|Gratuit|
|Évaluation du chiffrement des disques|✔|✔|-|Gratuit|
|Évaluation des vulnérabilités tierces|✔|-|-|Gratuit|
|[Évaluation de la sécurité réseau](security-center-network-recommendations.md)|✔|✔|-|Gratuit|

--- 


> [!TIP]
>Pour expérimenter les fonctionnalités uniquement disponibles dans le niveau tarifaire standard, les utilisateurs du niveau gratuit peuvent s’inscrire à un essai de 30 jours. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/security-center/).


## <a name="supported-endpoint-protection-solutions"></a>Solutions de protection du point de terminaison prises en charge <a name="endpoint-supported"></a>

Le tableau suivant fournit une matrice de ce qui suit :

 - Si vous pouvez utiliser Azure Security Center pour installer chaque solution.
 - Les solutions de protection du point de terminaison que Security Center peut détecter. Si l’une des solutions de protection du point de terminaison de cette liste est détectée, Security Center vous déconseillera d'en installer une autre.

Pour plus d’informations sur le moment où les recommandations sont générées pour chacune de ces protections, consultez [Évaluation de la protection de point de terminaison et recommandations](security-center-endpoint-protection.md).

| Protection du point de terminaison| Plateformes | Installation du centre de sécurité | Détection du centre de sécurité |
|------|------|-----|-----|
| Windows Defender (logiciel anti-programme malveillant de Microsoft)| Windows Server 2016| Non, intégré au système d’exploitation| Oui |
| System Center Endpoint Protection (logiciel anti-programme malveillant de Microsoft) | Windows Server 2012 R2, 2012, 2008 R2 (voir la remarque ci-dessous) | Via l’extension | Oui |
| Trend Micro : toutes les versions* | Gamme Windows Server  | Non | Oui |
| Symantec v12.1.1100+| Gamme Windows Server  | Non | Oui |
| McAfee v10+ | Gamme Windows Server  | Non | Oui |
| McAfee v10+ | Famille de serveurs Linux  | Non | Oui **\*** |
| Sophos V9+| Famille de serveurs Linux  | Non | Oui  **\***  |

 **\*** L’état de couverture et les données de prise en charge sont actuellement disponibles uniquement dans l’espace de travail Log Analytics associé à vos abonnements protégés. Il n’est pas reflété dans le portail Azure Security Center.

> [!NOTE]
> - La détection de System Center Endpoint Protection (SCEP) sur une machine virtuelle Windows Server 2008 R2 requiert l’installation de SCEP après celle de PowerShell 3.0 (ou d’une version ultérieure).
> - La détection de la protection Trend Micro est prise en charge pour les agents Deep Security.  Les agents OfficeScan ne sont pas pris en charge.


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Security Center collecte les données et l'agent Log Analytics Agent](security-center-enable-data-collection.md).
- Découvrez comment [Security Center gère et protège les données](security-center-data-security.md).
- [Découvrez comment planifier l’adoption d’Azure Security Center et prenez connaissance des considérations relatives à la conception](security-center-planning-and-operations-guide.md).
- Passez en revue les [plateformes qui prennent en charge Security Center](security-center-os-coverage.md).
- Découvrez la [protection contre les menaces pour les ordinateurs Windows et Linux dans Azure Security Center](threat-protection.md#windows-machines).
- Accédez aux [Questions fréquentes (FAQ) sur Azure Security Center](faq-general.md).