---
title: Maintenir la connexion dans Azure Active Directory B2C
description: Découvrez comment configurer la fonctionnalité « Maintenir la connexion » dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 02/27/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9a27487fa69888b02883c3d9a2151887f41afc45
ms.sourcegitcommit: 225a0b8a186687154c238305607192b75f1a8163
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/29/2020
ms.locfileid: "78189376"
---
# <a name="enable-keep-me-signed-in-kmsi-in-azure-active-directory-b2c"></a>Activer la fonctionnalité « Maintenir la connexion » dans Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Vous pouvez activer la fonctionnalité Maintenir la connexion pour les utilisateurs de vos applications web et native qui ont des comptes locaux dans votre annuaire Azure AD B2C (Azure Active Directory B2C). Cette fonctionnalité accorde l’accès aux utilisateurs qui retournent dans votre application sans avoir à entrer une nouvelle fois leur nom d’utilisateur et leur mot de passe. Cet accès est révoqué lorsque l’utilisateur se déconnecte.

Vous ne devez pas activer cette option sur les ordinateurs publics.

![Exemple de page de connexion avec la case à cocher Maintenir la connexion](./media/custom-policy-keep-me-signed-in/kmsi.PNG)

## <a name="prerequisites"></a>Prérequis

- Un locataire Azure AD B2C configuré pour permettre la connexion à un compte local. La fonctionnalité Maintenir la connexion n’est pas prise en charge pour les comptes des fournisseurs d’identité externes.
- Suivez les étapes décrites dans [Bien démarrer avec les stratégies personnalisées dans Azure Active Directory B2C](custom-policy-get-started.md).

## <a name="configure-the-page-identifier"></a>Configurer l’identificateur de page

Pour activer la fonctionnalité « Maintenir la connexion », définissez l’élément `DataUri` de définition de contenu sur l’[identificateur de page](contentdefinitions.md#datauri) `unifiedssp` et la [version de page](page-layout.md) *1.1.0* ou ultérieure.

1. Ouvrez le fichier d’extension de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/` **`TrustFrameworkExtensions.xml`** </em>. Ce fichier d’extension est un des fichiers de stratégie inclus dans le pack de démarrage des stratégies personnalisées, que vous avez dû obtenir en suivant la rubrique prérequise [Bien démarrer avec les stratégies personnalisées](custom-policy-get-started.md).
1. Recherchez l’élément **BuildingBlocks**. Si l’élément n’existe pas, ajoutez-le.
1. Ajoutez l’élément **ContentDefinitions** à l’élément **BuildingBlocks** de la stratégie.

    Votre stratégie personnalisée doit se présenter comme l’extrait de code suivant :

    ```xml
    <BuildingBlocks>
      <ContentDefinitions>
        <ContentDefinition Id="api.signuporsignin">
          <DataUri>urn:com:microsoft:aad:b2c:elements:unifiedssp:1.1.0</DataUri>
        </ContentDefinition>
      </ContentDefinitions>
    </BuildingBlocks>
    ```

1. Enregistrez le fichier d’extensions.



## <a name="configure-a-relying-party-file"></a>Configurer un fichier de partie de confiance

Mettez à jour le fichier de partie de confiance qui lance le parcours utilisateur que vous avez créé.

1. Ouvrez votre fichier de stratégie personnalisée. Par exemple *SignUpOrSignin.xml*.
1. S’il n’existe pas déjà, ajoutez un nœud enfant `<UserJourneyBehaviors>` au nœud `<RelyingParty>`. Il doit être placé immédiatement après `<DefaultUserJourney ReferenceId="User journey Id" />`, par exemple : `<DefaultUserJourney ReferenceId="SignUpOrSignIn" />`.
1. Ajoutez le nœud suivant en tant qu’enfant de l’élément `<UserJourneyBehaviors>`.

    ```XML
    <UserJourneyBehaviors>
      <SingleSignOn Scope="Tenant" KeepAliveInDays="30" />
      <SessionExpiryType>Absolute</SessionExpiryType>
      <SessionExpiryInSeconds>1200</SessionExpiryInSeconds>
    </UserJourneyBehaviors>
    ```

    - **SessionExpiryType** – Indique comment la session est étendue par le délai spécifié dans `SessionExpiryInSeconds` et `KeepAliveInDays`. La valeur `Rolling` (valeur par défaut) indique que la session est étendue chaque fois que l’utilisateur effectue une authentification. La valeur `Absolute` indique que l’utilisateur est obligé de se réauthentifier après le délai spécifié.

    - **SessionExpiryInSeconds** – Durée de vie des cookies de session lorsque l’option *Maintenir la connexion* n’est pas activée ou si un utilisateur ne sélectionne pas *Maintenir la connexion*. La session expire une fois le délai `SessionExpiryInSeconds` écoulé ou le navigateur fermé.

    - **KeepAliveInDays** – Durée de vie des cookies de session lorsque l’option *Maintenir la connexion* est activée et que l’utilisateur sélectionne *Maintenir la connexion*.  La valeur de `KeepAliveInDays` est prioritaire sur la valeur de `SessionExpiryInSeconds`, et détermine le délai d’expiration de la session. Si un utilisateur ferme et rouvre le navigateur, il peut toujours se connecter en mode silencieux à condition de le faire dans le délai KeepAliveInDays.

    Pour plus d’informations, consultez [Comportements de parcours utilisateur](relyingparty.md#userjourneybehaviors).

Nous vous recommandons de définir la valeur de SessionExpiryInSeconds sur une courte période (1 200 secondes), tandis que vous pouvez définir la valeur de KeepAliveInDays sur une période relativement longue (30 jours), comme l’illustre l’exemple suivant :

```XML
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <UserJourneyBehaviors>
    <SingleSignOn Scope="Tenant" KeepAliveInDays="30" />
    <SessionExpiryType>Absolute</SessionExpiryType>
    <SessionExpiryInSeconds>1200</SessionExpiryInSeconds>
  </UserJourneyBehaviors>
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
      <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

4. Enregistrez vos modifications, puis chargez le fichier.
5. Pour tester la stratégie personnalisée que vous avez chargée, dans le portail Azure, accédez à la page Stratégie, puis sélectionnez **Exécuter maintenant**.

Vous pouvez trouver l’exemple de stratégie [ici](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/keep%20me%20signed%20in).
