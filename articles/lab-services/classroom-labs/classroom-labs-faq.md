---
title: Labs de salle de classe dans Azure Lab Services - FAQ | Microsoft Docs
description: Cet article offre des réponses aux questions fréquentes sur les labs de salle de classe dans Azure Lab Services.
services: lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/14/2020
ms.author: spelluru
ms.openlocfilehash: 8d1ed128181d036af0026ae273c2c5bf1d3a066e
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77443497"
---
# <a name="classroom-labs-in-azure-lab-services--frequently-asked-questions-faq"></a>Labs de salle de classe dans Azure Lab Services - Forum aux questions (FAQ)
Obtenez des réponses aux questions les plus fréquemment posées sur les labs de salle de classe dans Azure Lab Services. 

## <a name="quotas"></a>Quotas

### <a name="is-the-quota-per-user-or-per-week-or-per-entire-duration-of-the-lab"></a>Le quota est-il appliqué par utilisateur, par semaine ou pour toute la durée du lab ? 
Le quota que vous définissez pour un lab s’applique à chaque étudiant pour toute la durée du lab. Le [temps d’exécution planifié des machines virtuelles](how-to-create-schedules.md) ne compte pas dans le quota alloué à un utilisateur. Le quota s’applique au temps pendant lequel un étudiant utilise les machines virtuelles en dehors des heures planifiées.  Pour plus d’informations sur les quotas, consultez [Définir des quotas pour les utilisateurs](how-to-configure-student-usage.md#set-quotas-for-users).

### <a name="if-professor-turns-on-a-student-vm-does-that-affect-the-student-quota"></a>Si le professeur active la machine virtuelle d’un étudiant, cela affecte-t-il le quota de l’étudiant ? 
Non. Non. Quand le professeur active la machine virtuelle d’un étudiant, cela n’affecte pas le quota alloué à l’étudiant. 

## <a name="schedules"></a>Planifications

### <a name="do-all-vms-in-the-lab-start-automatically-when-a-schedule-is-set"></a>Toutes les machines virtuelles du lab démarrent-elles automatiquement quand un calendrier est défini ? 
Non. Cela ne concerne pas toutes les machines virtuelles, mais uniquement celles qui sont attribuées à des utilisateurs selon un calendrier. Les machines virtuelles qui ne sont pas attribuées à un utilisateur ne démarrent pas automatiquement. Ce comportement est normal. 

## <a name="lab-accounts"></a>Comptes lab

### <a name="why-am-i-not-able-to-create-a-lab-because-of-unavailability-of-the-address-range"></a>Pourquoi ne puis-je pas créer un lab en cas d’indisponibilité de la plage d’adresses ? 
Les labs de salle de classe peuvent créer des machines virtuelles de lab dans une plage d’adresses IP que vous spécifiez lors de la création de votre compte lab dans le Portail Azure. Quand une plage d’adresses est fournie, 512 adresses IP sont allouées à chaque lab créé par la suite pour les machines virtuelles de lab. La plage d’adresses du compte lab doit être suffisamment étendue pour prendre en charge tous les labs que vous envisagez de créer sous le compte lab. 

Par exemple, si vous avez un bloc de /19 - 10.0.0.0/19, cette plage d’adresses prend en charge 8 192 adresses IP et 16 labs (8 192/512 = 16 labs). Dans ce cas, la création du 17e lab échouera.

### <a name="what-port-ranges-should-i-open-on-my-organizations-firewall-setting-to-connect-to-lab-virtual-machines-via-rdpssh"></a>Quelles plages de ports dois-je ouvrir sur le paramètre de pare-feu de mon organisation pour permettre la connexion aux machines virtuelles d’un lab via RDP/SSH ?

Les ports sont les suivants : 49152–65535. Les labos de salle de classe se trouvent derrière un équilibreur de charge. Chaque labo dispose d’une adresse IP publique unique et chaque machine virtuelle du labo possède un port unique. 

Vous pouvez également voir l’adresse IP privée de chaque machine virtuelle sous l’onglet **Pool de machines virtuelles** de la page d’accueil du labo dans le portail Azure. Si vous republiez un labo, son adresse IP publique ne change pas. Cependant, l’adresse IP privée et le numéro de port de chaque machine virtuelle dans le labo peuvent changer. Pour en savoir plus, consultez l’article [Paramètres de pare-feu pour Azure Lab Services](how-to-configure-firewall-settings.md).

### <a name="what-public-ip-address-range-should-i-open-on-my-organizations-firewall-settings-to-connect-to-lab-virtual-machines-via-rdpssh"></a>Quelles plages d’adresses IP publiques dois-je ouvrir sur le paramètre de pare-feu de mon organisation pour permettre la connexion aux machines virtuelles d’un lab via RDP/SSH ?
Consultez [Plages d’adresses IP et balises de service Azure - Cloud public](https://www.microsoft.com/download/details.aspx?id=56519), qui fournit la plage d’adresses IP publiques pour les centres de données dans Azure. Vous pouvez ouvrir les adresses IP des régions où se trouvent vos comptes de laboratoire.

## <a name="virtual-machine-images"></a>Images de machine virtuelle

### <a name="as-a-lab-creator-why-cant-i-enable-additional-image-options-in-the-virtual-machine-images-dropdown-when-creating-a-new-lab"></a>En tant que créateur de labo, pourquoi je ne peux pas activer d’autres options d’image dans la liste déroulante des images de machine virtuelle quand je crée un labo ?

Quand un administrateur vous ajoute à un compte lab comme créateur de labo, vous êtes autorisé à créer des labos. Toutefois, vous ne disposez pas des autorisations nécessaires pour modifier les paramètres du compte lab, notamment la liste des images de machine virtuelle activées. Pour activer des images supplémentaires, demandez à l’administrateur de votre compte lab de le faire pour vous ou demandez-lui de vous ajouter au compte lab avec un rôle de contributeur. Le rôle de contributeur vous donne les autorisations nécessaires pour modifier la liste des images de machine virtuelle dans le compte lab.

## <a name="users"></a>Utilisateurs

### <a name="how-many-users-can-be-in-a-classroom-lab"></a>Combien peut-il y avoir d’utilisateurs dans un laboratoire de classe ?
Vous pouvez ajouter jusqu’à 400 utilisateurs à un laboratoire de classe. 

## <a name="blog-post"></a>Billet de blog
Abonnez-vous au [blog Azure Lab Services](https://azure.microsoft.com/blog/tag/azure-lab-services/).

## <a name="update-notifications"></a>Notifications de mise à jour
Abonnez-vous aux [mises à jour de Lab Services](https://azure.microsoft.com/updates/?product=lab-services) pour rester informé des nouvelles fonctionnalités de Lab Services.

## <a name="general"></a>Général
### <a name="what-if-my-question-isnt-answered-here"></a>Que dois-je faire si je n’ai pas trouvé de réponse à ma question ici ?
Si votre question n’est pas répertoriée ici, faites-le-nous savoir pour que nous puissions vous aider à trouver une réponse.

- Postez une question à la fin de ce Forum aux questions. 
- Pour atteindre un public plus large, postez une question sur le [forum Stack Overflow consacré à Azure Lab Services](https://stackoverflow.com/questions/tagged/azure-lab-services). 
- Concernant les demandes de fonctionnalité, transmettez vos questions et idées à [Azure Lab Services - User Voice](https://feedback.azure.com/forums/320373-lab-services?category_id=352774).

