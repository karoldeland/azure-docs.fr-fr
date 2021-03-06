---
title: 'Machine à vecteurs de support à deux classes : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module **Two-Class Support Vector Machine** (Machine à vecteurs de support à deux classes) dans Azure Machine Learning pour créer un modèle basé sur l’algorithme de machine à vecteurs de support.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: ba788518951e72c1701d99decf46350e8665dbae
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79455806"
---
# <a name="two-class-support-vector-machine-module"></a>Module Two-Class Support Vector Machine (Machine à vecteurs de support à deux classes)

Cet article décrit un module dans le concepteur Azure Machine Learning (version préliminaire).

Utilisez ce module pour créer un modèle basé sur l’algorithme de machine à vecteurs de support. 

Les machines à vecteurs de support (SVM, support vector machines) représentent une classe bien étudiée de méthodes d’apprentissage supervisées. Cette implémentation particulière est adaptée à la prédiction de deux résultats possibles en fonction de variables continues ou catégorielles.

Après avoir défini les paramètres de modèle, effectuez l’apprentissage du modèle en utilisant les modules de formation et en fournissant un *jeu de données avec balises* qui inclut une colonne d’étiquettes ou de résultats.

## <a name="about-support-vector-machines"></a>À propos des machines à vecteurs de support

Les machines à vecteurs de support font partie des premiers algorithmes d’apprentissage automatique, et les modèles SVM ont été utilisés dans de nombreuses applications, de la récupération d’informations à la classification de texte et d’images. Les SVM peuvent être utilisées pour les tâches de classification et de régression.

Ce modèle de SVM est un modèle d’apprentissage supervisé qui requiert des données étiquetées. Dans le cadre du processus d’apprentissage, l’algorithme analyse les données d’entrée et reconnaît des modèles dans un espace de caractéristiques multidimensionnel appelé *hyperplan*.  Tous les exemples d’entrée sont représentés sous forme de points dans cet espace et sont mappés aux catégories de sortie de sorte que les catégories soient séparées par un écart aussi large et clair que possible.

Pour la prédiction, l’algorithme SVM attribue les nouveaux exemples dans une catégorie ou dans l’autre en les mappant au même espace. 

## <a name="how-to-configure"></a>Comment configurer 

Pour ce type de modèle, il est recommandé de normaliser le jeu de données avant de l’utiliser pour effectuer l’apprentissage du classifieur.
  
1.  Ajoutez le module **Machine à vecteurs de support à deux classes** à votre pipeline.  
  
2.  Spécifiez le mode d’apprentissage du modèle en définissant l’option **Créer un mode d’apprentissage**.  
  
    -   **Single Parameter** (Paramètre unique) : si vous savez comment vous voulez configurer le modèle, vous pouvez fournir un ensemble spécifique de valeurs comme arguments.  

    -   **Plage de paramètres** : en cas de doute sur les paramètres à utiliser, utilisez le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md). Vous fournissez une plage de valeurs, et le mode d'apprentissage effectue une itération sur plusieurs combinaisons de paramètres pour déterminer la combinaison de valeurs qui produit le meilleur résultat.

3.  Pour **Nombre d’itérations**, tapez un nombre indiquant le nombre d’itérations utilisées lors de la génération du modèle.  
  
     Ce paramètre peut être utilisé pour contrôler le compromis entre la précision et la vitesse de formation.  
  
4.  Pour **Lambda**, tapez une valeur à utiliser en tant que poids de la régularisation L1.  
  
     Ce coefficient de régularisation permet d’ajuster le modèle. Des valeurs plus élevées pénalisent les modèles plus complexes.  
  
5.  Sélectionnez l’option **Normalize features** (Normaliser les caractéristiques) si vous souhaitez normaliser les caractéristiques avant la formation.
  
     Si vous appliquez la normalisation, avant l’apprentissage, les points de données sont centrés sur la moyenne et mis à l’échelle de façon à présenter une unité d’écart type.
  
6.  Sélectionnez l’option **Project to the unit sphere** (Projeter sur la sphère d’unité) pour normaliser les coefficients.
  
     La projection des valeurs vers un espace d’unité signifie qu’avant l’apprentissage, les points de données sont centrés sur 0 et mis à l’échelle de façon à présenter une unité d’écart type.
  
7.  Sous **Random number seed** (Valeur de départ aléatoire), tapez une valeur entière à utiliser comme valeur de départ si vous souhaitez garantir la reproductibilité entre les exécutions.  Sinon, une valeur d’horloge système est utilisée comme valeur de départ, ce qui peut entraîner des résultats légèrement différents entre les exécutions.
  
9. Connectez un jeu de données étiqueté et l’un des [modules de formation](module-reference.md) :
  
    -   Si vous définissez **Créer un mode d’apprentissage** sur **Paramètre unique**, utilisez le module [Entraîner le du modèle](train-model.md).
  
10. Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Pour enregistrer un instantané du modèle entraîné, sélectionnez l’onglet **Sorties** dans le panneau droit du module **Entraîner le modèle**. Sélectionnez l’icône **Inscrire le jeu de données** pour enregistrer le modèle en tant que module réutilisable.

+ Pour utiliser le modèle à des fins de scoring, ajoutez le module **Modèle de scoring** à un pipeline.


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 