---
title: Exemples de types de classes dans Azure Lab Services | Microsoft Docs
description: Cite certains des types de classes pour lesquels il est possible de configurer des laboratoires à l’aide d’Azure Lab Services.
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/30/2019
ms.author: spelluru
ms.openlocfilehash: 27619a69a1f7fbded8ce6430afc2b8e9a8b4a00c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79296725"
---
# <a name="class-types-overview---azure-lab-services"></a>Vue d’ensemble des types de classes - Azure Lab Services

Azure Lab Services vous permet de configurer rapidement un environnement lab de salle de classe dans le cloud. Les articles de cette section fournissent des conseils sur la configuration de plusieurs types de laboratoires de classe à l’aide d’Azure Lab Services.

## <a name="deep-learning-in-natural-language-processing"></a>Deep Learning pour le traitement en langage naturel

Vous pouvez configurer un laboratoire axé sur le Deep Learning dans le cadre du traitement en langage naturel (NLP) à l’aide d’Azure Lab Services. Le traitement en langage naturel (NLP) est une forme d’intelligence artificielle (IA) qui fournit aux ordinateurs des fonctionnalités de traduction, de reconnaissance vocale et d’autres fonctionnalités de compréhension de la langue. Les étudiants qui suivent un cours de traitement en langage naturel disposent d’une machine virtuelle Linux pour apprendre à appliquer des algorithmes de réseau neuronal afin de développer des modèles de deep learning utilisés pour l’analyse de la langue humaine écrite.

Pour plus d’informations sur la configuration de ce type de labo, consultez [Configurer un labo axé sur le Deep Learning pour le traitement en langage naturel à l’aide d’Azure Lab Services](class-type-deep-learning-natural-processing.md).

## <a name="shell-scripting-on-linux"></a>Scripts shell sur Linux

Vous pouvez configurer un laboratoire pour enseigner la création de scripts shell sur Linux. Dans le cadre de l’administration système, l’écriture de scripts permet aux administrateurs d’éviter les tâches répétitives. Dans cet exemple de scénario, les scripts bash traditionnels et les scripts améliorés sont abordés. Les scripts améliorés sont des scripts qui associent des commandes bash et Ruby. Cette approche permet à Ruby de passer des données et fournit des commandes bash pour interagir avec le shell.

Les étudiants qui suivent ces cours d’écriture de scripts disposent d’une machine virtuelle Linux pour apprendre les bases de Linux et également se familiariser avec les scripts de shell bash. La machine virtuelle Linux est fournie avec un accès au Bureau à distance activé. En outre, les éditeurs de texte [gedit](https://help.gnome.org/users/gedit/stable/) et [Visual Studio Code](https://code.visualstudio.com/) y sont installés.

Pour plus d’informations sur la configuration de ce type de labo, consultez [Scripts shell sur Linux](class-type-shell-scripting-linux.md).

## <a name="ethical-hacking"></a>Piratage éthique

Vous pouvez configurer un labo de classe qui se concentre sur l’analyse forensique du piratage éthique. Des tests d’intrusion, une pratique utilisée par la communauté de piratage éthique, sont effectués quand quelqu’un tente d’accéder au système ou au réseau pour détecter les vulnérabilités qu’un attaquant malveillant pourrait exploiter.

Dans une classe sur le piratage éthique, les étudiants apprennent les techniques modernes de défense face aux vulnérabilités. Chaque étudiant a accès à une machine virtuelle hôte Windows Server qui comporte deux machines virtuelles imbriquées : une avec une image [Metasploitable3](https://github.com/rapid7/metasploitable3) et une autre avec une image [Kali Linux](https://www.kali.org/). La machine virtuelle Metasploitable est utilisée à des fins d’exploitation.  La machine virtuelle Kali Linux permet d’accéder aux outils nécessaires pour exécuter des tâches d’investigation.

Pour plus d’informations sur la configuration de ce type de labo, consultez [Configurer un labo pour enseigner le piratage éthique](class-type-ethical-hacking.md).

## <a name="database-management"></a>Gestion de bases de données
Les concepts des bases de données constituent l’un des cours d’introduction abordés dans la plupart des départements informatiques à l’université. Vous pouvez configurer un labo pour un cours élémentaire de gestion de bases de données dans Azure Lab Services. Par exemple, vous pouvez configurer un modèle de machine virtuelle dans un labo avec un serveur de base de données [MySQL](https://www.mysql.com/) ou un serveur [SQL Server 2019](https://www.microsoft.com/sql-server/sql-server-2019).

Pour des informations détaillées sur la configuration de ce type de labo, consultez [Configurer un labo pour enseigner la gestion des bases de données pour les bases de données relationnelles](class-type-database-management.md).

## <a name="python-and-jupyter-notebooks"></a>Python et Jupyter Notebooks
Vous pouvez configurer une machine modèle dans Azure Lab Services avec les outils nécessaires pour enseigner aux élèves comment utiliser [Jupyter Notebooks](http://jupyter-notebook.readthedocs.io). Jupyter Notebooks est un projet open source qui vous permet de combiner facilement du texte enrichi et du code source [Python](https://www.python.org/) exécutable sur un seul canevas appelé notebook. L’exécution d’un notebook produit un enregistrement linéaire des entrées et des sorties.  Ces sorties peuvent inclure du texte, des tables d’informations, des nuages de points, etc.

Pour des informations détaillées sur la configuration de ce type de labo, consultez [Configurer un labo pour enseigner la science des données avec Python et Jupyter Notebooks](class-type-jupyter-notebook.md).

## <a name="mobile-app-development-with-android-studio"></a>Développement d’applications mobiles avec Android Studio
Vous pouvez configurer un labo dans Azure Lab Services pour enseigner un cours d’introduction sur le développement d’applications mobiles. Ce cours est axé sur les applications mobiles Android pouvant être publiées sur [Google Play Store](https://play.google.com/store/apps).  Les étudiants apprennent à utiliser [Android Studio](https://developer.android.com/studio) pour créer des applications.  [Visual Studio Emulator pour Android](https://visualstudio.microsoft.com/vs/msft-android-emulator/) est utilisé pour tester l’application localement.

Pour des informations détaillées sur la configuration de ce type de labo, consultez [Configurer un labo pour enseigner le développement d’applications mobiles avec Android Studio](class-type-mobile-dev-android-studio.md).


## <a name="next-steps"></a>Étapes suivantes

Voir les articles suivants :

- [Configurer un laboratoire axé sur le Deep Learning dans le cadre du traitement en langage naturel (NLP) à l’aide d’Azure Lab Services](class-type-deep-learning-natural-processing.md)
- [Scripts shell sur Linux](class-type-shell-scripting-linux.md)
- [Piratage éthique](class-type-ethical-hacking.md)