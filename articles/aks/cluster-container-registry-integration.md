---
title: Intégrer Azure Container Registry avec Azure Kubernetes Service
description: Découvrez comment intégrer Azure Kubernetes Service (AKS) à Azure Container Registry (ACR)
services: container-service
manager: gwallace
ms.topic: article
ms.date: 02/25/2020
ms.openlocfilehash: f83faf05eb7099557d5b653e0b24591062c44d11
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79368449"
---
# <a name="authenticate-with-azure-container-registry-from-azure-kubernetes-service"></a>S’authentifier auprès d’Azure Container Registry à partir d’Azure Kubernetes Service

Quand vous utilisez Azure Container Registry (ACR) avec Azure Kubernetes Service (AKS), vous avez besoin d’un mécanisme d’authentification. Cet article fournit des exemples de configuration de l’authentification entre ces deux services Azure.

Vous pouvez configurer l’intégration de AKS à ACR à l’aide de quelques commandes simples avec Azure CLI.

## <a name="before-you-begin"></a>Avant de commencer

Ces exemples requièrent les éléments suivants :

* Le rôle **Propriétaire** ou **Administrateur de compte Azure** sur **l’abonnement Azure**
* Azure CLI version 2.0.73 ou ultérieure

Pour ne pas devoir utiliser un rôle **Propriétaire** ou **Administrateur de compte Azure**, vous pouvez configurer un principal de service manuellement ou utiliser un principal de service existant afin d'authentifier ACR depuis AKS. Pour plus d’informations, consultez [Authentification ACR à l’aide de principaux de service](../container-registry/container-registry-auth-service-principal.md) ou [S’authentifier à partir de Kubernetes avec un secret de tirage (pull)](../container-registry/container-registry-auth-kubernetes.md).

## <a name="create-a-new-aks-cluster-with-acr-integration"></a>Créer un nouveau cluster AKS avec l’intégration ACR

Vous pouvez configurer l’intégration d’AKS et d’ACR lors de la création initiale de votre cluster AKS.  Pour permettre à un cluster AKS d’interagir avec ACR, un **principal du service** Azure Active Directory est utilisé. La commande CLI suivante vous permet d’autoriser un ACR existant dans votre abonnement et de configurer le rôle **ACRPull** approprié pour le principal du service. Fournissez des valeurs valides pour vos paramètres ci-dessous.

```azurecli
# set this to the name of your Azure Container Registry.  It must be globally unique
MYACR=myContainerRegistry

# Run the following line to create an Azure Container Registry if you do not already have one
az acr create -n $MYACR -g myContainerRegistryResourceGroup --sku basic

# Create an AKS cluster with ACR integration
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr $MYACR
```

Vous pouvez également spécifier le nom ACR à l’aide d’un ID de ressource ACR, dont le format est :

`/subscriptions/\<subscription-id\>/resourceGroups/\<resource-group-name\>/providers/Microsoft.ContainerRegistry/registries/\<name\>` 

```azurecli
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr /subscriptions/<subscription-id>/resourceGroups/myContainerRegistryResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry
```

Cette étape peut prendre plusieurs minutes.

## <a name="configure-acr-integration-for-existing-aks-clusters"></a>Configurer une intégration ACR pour les clusters AKS existants

Intégrez un ACR existant à des clusters AKS existants en fournissant des valeurs valides pour **acr-name** ou **acr-resource-id** comme indiqué ci-dessous.

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acrName>
```

ou

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-resource-id>
```

Vous pouvez également supprimer l’intégration entre ACR et un cluster AKS à l’aide de ce qui suit

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --detach-acr <acrName>
```

or

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --detach-acr <acr-resource-id>
```

## <a name="working-with-acr--aks"></a>Utilisation d’ACR et AKS

### <a name="import-an-image-into-your-acr"></a>Importer une image dans votre instance ACR

Importez une image à partir du hub Docker dans votre instance ACR en exécutant la commande suivante :


```azurecli
az acr import  -n <myContainerRegistry> --source docker.io/library/nginx:latest --image nginx:v1
```

### <a name="deploy-the-sample-image-from-acr-to-aks"></a>Déployer l’exemple d’image depuis ACR vers AKS

Vérifiez que vous disposez des informations d’identification AKS appropriées.

```azurecli
az aks get-credentials -g myResourceGroup -n myAKSCluster
```

Créez un fichier appelé **acr-nginx.yaml** contenant les éléments suivants :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx0-deployment
  labels:
    app: nginx0-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx0
  template:
    metadata:
      labels:
        app: nginx0
    spec:
      containers:
      - name: nginx
        image: <replace this image property with you acr login server, image and tag>
        ports:
        - containerPort: 80
```

Ensuite, exécutez ce déploiement dans votre cluster AKS :

```console
kubectl apply -f acr-nginx.yaml
```

Vous pouvez surveiller le déploiement en exécutant :

```console
kubectl get pods
```

Vous devez avoir deux pods en cours d’exécution.

```output
NAME                                 READY   STATUS    RESTARTS   AGE
nginx0-deployment-669dfc4d4b-x74kr   1/1     Running   0          20s
nginx0-deployment-669dfc4d4b-xdpd6   1/1     Running   0          20s
```

<!-- LINKS - external -->
[AKS AKS CLI]:  https://docs.microsoft.com/cli/azure/aks?view=azure-cli-latest#az-aks-create
