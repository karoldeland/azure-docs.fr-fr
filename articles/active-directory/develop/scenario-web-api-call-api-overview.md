---
title: Générer une API web appelant des API web - Plateforme d’identités Microsoft | Azure
description: Apprenez à créer une API web qui appelle des API web en aval (vue d'ensemble).
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 467ff2f789cc83bc5651d831838da0b5c922c839
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76701737"
---
# <a name="scenario-a-web-api-that-calls-web-apis"></a>Scénario : Une API web qui appelle des API web

Découvrez tout ce que vous devez savoir pour générer une API web qui appelle des API web.

## <a name="prerequisites"></a>Prérequis

Ce scénario, dans lequel une API web protégée appelle des API web, repose sur le scénario « Protéger une API web ». Pour en savoir plus sur ce scénario de base, consultez la section [Scénario : API web protégée](scenario-protected-web-api-overview.md).

## <a name="overview"></a>Vue d’ensemble

- Un client d’application web, de bureau, mobile ou monopage (non représenté sur le diagramme) appelle une API web protégée et fournit un jeton du porteur JSON Web Token (JWT) dans son en-tête HTTP « Autorisation ».
- L’API web protégée valide le jeton et utilise la méthode de la bibliothèque d’authentification Microsoft (MSAL) `AcquireTokenOnBehalfOf` pour demander un autre jeton à Azure Active Directory (Azure AD) afin que l’API web protégée puisse appeler une deuxième API web, ou une API web en aval, pour le compte de l’utilisateur.
- Cette API web protéger peut également appeler `AcquireTokenSilent`ultérieurement afin de demander des jetons pour les autres API en aval pour le compte du même utilisateur. `AcquireTokenSilent` actualise le jeton, si besoin.

![Diagramme d’une API web appelant une API web](media/scenarios/web-api.svg)

## <a name="specifics"></a>Spécificités

La partie de l’inscription d’application liée aux autorisations de l’API est classique. La configuration de l'application implique l’utilisation du flux OBO OAuth 2.0 afin d'échanger le jeton du porteur JWT contre un jeton pour une API en aval. Ce jeton est ajouté au cache de jetons, où il est disponible dans les contrôleurs des API web, et peut ensuite obtenir un jeton en mode silencieux pour appeler des API en aval.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Inscription d’application](scenario-web-api-call-api-app-registration.md)
