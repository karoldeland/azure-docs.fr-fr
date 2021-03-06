---
title: Passer en production une application monopage - Plateforme d’identités Microsoft | Azure
description: En savoir plus sur la création d’une application monopage (passer en production)
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 01b923e0d013fab1815456e55eac6036ded772bb
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76701839"
---
# <a name="single-page-application-move-to-production"></a>Application monopage : Passer en production

Maintenant que vous savez comment acquérir un jeton pour appeler des API web, découvrez comment passer en production.

## <a name="improve-your-app"></a>Améliorer votre application

[Activez la journalisation](msal-logging.md) pour préparer votre application à la production.

## <a name="test-your-integration"></a>Tester votre intégration

Testez votre intégration en suivant la [check-list de l’intégration à la plateforme d’identités Microsoft](identity-platform-integration-checklist.md).

## <a name="next-steps"></a>Étapes suivantes

Présentation approfondie de l’exemple de démarrage rapide, avec explication du code permettant de connecter des utilisateurs et d’obtenir un jeton d’accès pour appeler l’API Microsoft Graph à l’aide de MSAL.js :

> [!div class="nextstepaction"]
> [Didacticiel JavaScript SPA](./tutorial-v2-javascript-spa.md)

Exemple qui montre comment obtenir des jetons pour vos propres API web de serveur principal à l’aide de MSAL.js :

> [!div class="nextstepaction"]
> [SPA avec serveur principal ASP.NET](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2)

Exemple qui montre comment utiliser MSAL.js pour connecter des utilisateurs dans une application inscrite auprès d’Azure Active Directory B2C (Azure AD B2C) :

> [!div class="nextstepaction"]
> [SPA avec Azure AD B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp)
