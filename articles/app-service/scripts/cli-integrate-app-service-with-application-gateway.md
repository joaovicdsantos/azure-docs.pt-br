---
title: Exemplo de script da CLI do Azure – integrar o Serviço de Aplicativo com o Gateway de Aplicativo | Microsoft Docs
description: Exemplo de script da CLI do Azure – integrar o Serviço de Aplicativo com o Gateway de Aplicativo
services: appservice
documentationcenter: appservice
author: madsd
manager: ccompy
editor: ''
tags: azure-service-management
ms.assetid: 6c16d6f8-3c08-4a59-858e-684a2ceccb5f
ms.service: app-service
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: web
ms.date: 12/09/2019
ms.author: madsd
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 3820e7bf00f99a846dd2be0edeaf4248e0dfd8ad
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/29/2021
ms.locfileid: "97006068"
---
# <a name="integrate-app-service-with-application-gateway-using-cli"></a>Integrar o Serviço de Aplicativo com o Gateway de Aplicativo usando a CLI

Este script de exemplo cria um aplicativo Web do Serviço de Aplicativo do Azure, uma Rede Virtual do Azure e um Gateway de Aplicativo. Em seguida, ele restringe o tráfego para o aplicativo Web de modo que só tenha origem na sub-rede do Gateway de Aplicativo.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial exige a versão 2.0.74 ou posterior da CLI do Azure. Se você está usando o Azure Cloud Shell, a versão mais recente já está instalada.

## <a name="sample-script"></a>Exemplo de script

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/integrate-with-app-gateway/integrate-with-app-gateway.sh "Integrate with Application Gateway")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicação sobre o script

Este script usa os comandos a seguir para criar um grupo de recursos, um aplicativo do Serviço de Aplicativo, o Cosmos DB e todos os recursos relacionados. Cada comando da tabela é vinculado à documentação específica do comando.

| Comando | Observações |
|---|---|
| [`az group create`](/cli/azure/group#az-group-create) | Cria um grupo de recursos no qual todos os recursos são armazenados. |
| [`az network vnet create`](/cli/azure/network/vnet#az-network-vnet-create) | Cria uma rede virtual. |
| [`az network public-ip create`](/cli/azure/network/public-ip#az-network-public-ip-create) | Cria um endereço IP público. |
| [`az network public-ip show`](/cli/azure/network/public-ip#az-network-public-ip-show) | Mostrar detalhes de um endereço IP público. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az-appservice-plan-create) | Cria um Plano do Serviço de Aplicativo. |
| [`az webapp create`](/cli/azure/webapp#az-webapp-create) | Cria um aplicativo Web do Serviço de Aplicativo. |
| [`az webapp show`](/cli/azure/webapp#az-webapp-show) | Mostrar detalhes de um aplicativo Web do Serviço de Aplicativo. |
| [`az webapp config access-restriction add`](/cli/azure/webapp/config/access-restriction#az-webapp-config-access-restriction-add) | Adiciona uma restrição de acesso ao aplicativo Web do Serviço de Aplicativo. |
| [`az network application-gateway create`](/cli/azure/network/application-gateway#az-network-application-gateway-create) | Cria um Gateway de Aplicativo. |
| [`az network application-gateway http-settings update`](/cli/azure/network/application-gateway/http-settings#az-network-application-gateway-http-settings-update) | Atualiza as configurações de HTTP do Gateway de Aplicativo. |

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre a CLI do Azure, veja a [documentação da CLI do Azure](/cli/azure).

Os exemplos de script da CLI do Serviço de Aplicativo adicionais podem ser encontrados na [documentação do Serviço de Aplicativo do Azure](../samples-cli.md).
