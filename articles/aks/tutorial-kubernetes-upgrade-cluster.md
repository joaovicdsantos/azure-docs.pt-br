---
title: Tutorial do Kubernetes no Azure - Atualizar um cluster
description: Neste tutorial do AKS (Serviço de Kubernetes do Azure), você aprenderá a atualizar um cluster AKS existente para a versão de Kubernetes mais recente disponível.
services: container-service
ms.topic: tutorial
ms.date: 01/12/2021
ms.custom: mvc
ms.openlocfilehash: baf82ee1393214f96f11de8f6d838c41222fa359
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/13/2021
ms.locfileid: "107377184"
---
# <a name="tutorial-upgrade-kubernetes-in-azure-kubernetes-service-aks"></a>Tutorial: Atualizar o Kubernetes no AKS (Serviço de Kubernetes do Azure)

Como parte do aplicativo e do ciclo de vida do cluster, convém atualizar para a versão mais recente disponível do Kubernetes e usar os novos recursos. Um cluster AKS (Serviço de Kubernetes do Azure) pode ser atualizado usando a CLI do Azure.

Neste tutorial, parte sete de sete, é feito o upgrade de um cluster Kubernetes. Você aprenderá como:

> [!div class="checklist"]
> * Identificar as versões atuais e disponíveis do Kubernetes
> * Fazer upgrade dos nós Kubernetes
> * Validar um upgrade bem-sucedido

## <a name="before-you-begin"></a>Antes de começar

Nos tutoriais anteriores, um aplicativo foi empacotado em uma imagem de contêiner. Essa imagem foi carregada no Registro de Contêiner do Azure e você criou um cluster do AKS. O aplicativo foi então implantado no cluster do AKS. Se você ainda não executou essas etapas e deseja continuar acompanhando, comece com o [Tutorial 1 – Criar imagens de contêiner][aks-tutorial-prepare-app].

Este tutorial exige a execução da CLI do Azure versão 2.0.53 ou posterior. Execute `az --version` para encontrar a versão. Se você precisa instalar ou atualizar, consulte [Instalar a CLI do Azure][azure-cli-install].

## <a name="get-available-cluster-versions"></a>Obter versões de cluster disponíveis

Antes de atualizar um cluster, use o comando [az aks get-upgrades][] para verificar quais versões do Kubernetes estão disponíveis para upgrade:

```azurecli
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster
```

No exemplo a seguir, a versão atual é *1.18.10*, e as versões disponíveis são mostradas em *atualizações*.

```json
{
  "agentPoolProfiles": null,
  "controlPlaneProfile": {
    "kubernetesVersion": "1.18.10",
    ...
    "upgrades": [
      {
        "isPreview": null,
        "kubernetesVersion": "1.19.1"
      },
      {
        "isPreview": null,
        "kubernetesVersion": "1.19.3"
      }
    ]
  },
  ...
}
```

## <a name="upgrade-a-cluster"></a>Atualizar um cluster

Para minimizar as interrupções nos aplicativos em execução, os nós do AKS são cuidadosamente isolados e drenados. Nesse processo, as seguintes etapas são executadas:

1. O agendador do Kubernetes impede o agendamento de pods adicionais em um nó que deve ser atualizado.
1. Os pods em execução no nó são agendados em outros nós do cluster.
1. Um nó que executa os últimos componentes do Kubernetes é criado.
1. Quando o novo nó fica pronto e é ingressado no cluster, o agendador do Kubernetes começa a executar pods nele.
1. O nó antigo é excluído e o próximo nó no cluster inicia o processo de isolamento e drenagem.

Use o comando [az aks upgrade][] para fazer upgrade do cluster do AKS.

```azurecli
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --kubernetes-version KUBERNETES_VERSION
```

> [!NOTE]
> Você só pode atualizar uma versão secundária por vez. Por exemplo, você pode atualizar a versão *1.14.x* para a *1.15.x*, mas não a versão *1.14.x* para a *1.16.x* diretamente. Para atualizar a versão *1.14.x* para a *1.16.x*, primeiro, atualize a versão *1.14.x* para a *1.15.x* e, em seguida, faça outra atualização da versão *1.15.x* para a *1.16.x*.

A seguinte saída de exemplo condensada mostra o resultado da atualização para *1.19.1*. Observe que *kubernetesVersion* agora relata *1.19.1*:

```json
{
  "agentPoolProfiles": [
    {
      "count": 3,
      "maxPods": 110,
      "name": "nodepool1",
      "osType": "Linux",
      "storageProfile": "ManagedDisks",
      "vmSize": "Standard_DS1_v2",
    }
  ],
  "dnsPrefix": "myAKSClust-myResourceGroup-19da35",
  "enableRbac": false,
  "fqdn": "myaksclust-myresourcegroup-19da35-bd54a4be.hcp.eastus.azmk8s.io",
  "id": "/subscriptions/<Subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster",
  "kubernetesVersion": "1.19.1",
  "location": "eastus",
  "name": "myAKSCluster",
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

## <a name="validate-an-upgrade"></a>Validar uma atualização

Confirme se o upgrade teve êxito usando o comando [az aks show][] da seguinte forma:

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --output table
```

A seguinte saída de exemplo mostra que o cluster do AKS executa *KubernetesVersion 1.19.1*:

```output
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------------------------
myAKSCluster  eastus      myResourceGroup  1.19.1               Succeeded            myaksclust-myresourcegroup-19da35-bd54a4be.hcp.eastus.azmk8s.io
```

## <a name="delete-the-cluster"></a>Excluir o cluster

Como este tutorial é a última parte da série, talvez você deseje excluir o cluster do AKS. Devido aos nós Kubernetes serem executados em VMs (máquinas virtuais) do Azure, eles continuarão a incorrer em encargos mesmo se você não usar o cluster. Use o comando [az group delete][az-group-delete] para remover o grupo de recursos, o serviço de contêiner e todos os recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

> [!NOTE]
> Quando você excluir o cluster, a entidade de serviço do Azure Active Directory usada pelo cluster do AKS não será removida. Para obter as etapas para remover a entidade de serviço, confira [Considerações sobre a entidade de serviço do AKS e sua exclusão][sp-delete]. Se você usou a identidade gerenciada, ela é gerenciada pela plataforma e não exige provisionamento ou giro de nenhum segredo.

## <a name="next-steps"></a>Próximas etapas

Neste tutorial, você fez upgrade do Kubernetes em um cluster do AKS. Você aprendeu a:

> [!div class="checklist"]
> * Identificar as versões atuais e disponíveis do Kubernetes
> * Fazer upgrade dos nós Kubernetes
> * Validar um upgrade bem-sucedido

Para obter mais informações sobre o AKS, confira [Visão geral do AKS][aks-intro]. Para obter diretrizes sobre como criar soluções completas com o AKS, confira [Diretrizes sobre a solução AKS][aks-solution-guidance].

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-intro]: ./intro-kubernetes.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az aks show]: /cli/azure/aks#az-aks-show
[az aks get-upgrades]: /cli/azure/aks#az-aks-get-upgrades
[az aks upgrade]: /cli/azure/aks#az-aks-upgrade
[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-delete]: /cli/azure/group#az-group-delete
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[aks-solution-guidance]: /azure/architecture/reference-architectures/containers/aks-start-here?WT.mc_id=AKSDOCSPAGE
