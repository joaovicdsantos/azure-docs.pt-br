---
title: Integração com o Serviço do Azure SignalR
titleSuffix: Azure Digital Twins
description: Veja como transmitir telemetria do Azure digital gêmeos para clientes usando o Azure Signalr
author: dejimarquis
ms.author: aymarqui
ms.date: 02/12/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 89bd77c30ec52a72087598b86f22e85659fa1b0e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2021
ms.locfileid: "102203888"
---
# <a name="integrate-azure-digital-twins-with-azure-signalr-service"></a>Integrar o gêmeos digital do Azure ao serviço de Signaler do Azure

Neste artigo, você aprenderá a integrar o Azure digital gêmeos ao [serviço de signaler do Azure](../azure-signalr/signalr-overview.md).

A solução descrita neste artigo permite enviar dados telemétricos digitais para clientes conectados, como uma única página da Web ou um aplicativo móvel. Como resultado, os clientes são atualizados com métricas em tempo real e status de dispositivos IoT, sem a necessidade de sondar o servidor ou enviar novas solicitações HTTP para atualizações.

## <a name="prerequisites"></a>Pré-requisitos

Aqui estão os pré-requisitos que você deve concluir antes de continuar:

* Antes de integrar sua solução com o serviço de Signaler do Azure neste artigo, você deve concluir o tutorial do Azure digital gêmeos [_**: conectar uma solução de ponta a ponta**_](tutorial-end-to-end.md), pois este artigo de instruções se baseia nele. O tutorial orienta você pela configuração de uma instância do gêmeos digital do Azure que funciona com um dispositivo IoT virtual para disparar atualizações de atualização de cópia digital. Este artigo de instruções conectará essas atualizações a um aplicativo Web de exemplo usando o serviço de Signaler do Azure.

* Você precisará dos seguintes valores do tutorial:
  - Tópico da grade de eventos
  - Resource group
  - Serviço de aplicativo/nome do aplicativo de funções
    
* Você precisará de [**Node.js**](https://nodejs.org/) instalado em seu computador.

Você também deve entrar no [portal do Azure](https://portal.azure.com/) com sua conta do Azure.

## <a name="solution-architecture"></a>Arquitetura da solução

Você estará anexando o serviço de Signalr do Azure ao gêmeos digital do Azure por meio do caminho abaixo. As seções A, B e C no diagrama são extraídas do diagrama da arquitetura do [pré-requisito do tutorial de ponta a ponta](tutorial-end-to-end.md). Neste artigo de instruções, você criará a seção D na arquitetura existente.

:::image type="content" source="media/how-to-integrate-azure-signalr/signalr-integration-topology.png" alt-text="Uma exibição dos serviços do Azure em um cenário de ponta a ponta. Descreve dados que fluem de um dispositivo para o Hub IoT, por meio de uma função do Azure (seta B) para uma instância do gêmeos digital do Azure (seção A) e, em seguida, pela grade de eventos para outra função do Azure para processamento (seta C). A seção D mostra os dados que fluem da mesma grade de eventos na seta C para uma função do Azure rotulada ' difusão '. ' Broadcast ' comunica-se com outra função do Azure rotulada como ' Negotiate ', e ' Broadcast ' e ' Negotiate ' se comunicam com dispositivos de computador." lightbox="media/how-to-integrate-azure-signalr/signalr-integration-topology.png":::

## <a name="download-the-sample-applications"></a>Baixar os aplicativos de exemplo

Primeiro, baixe os aplicativos de exemplo necessários. Você precisará dos seguintes itens:
* [**Exemplos de ponta a ponta do Azure digital gêmeos**](/samples/azure-samples/digital-twins-samples/digital-twins-samples/): Este exemplo contém um *AdtSampleApp* que contém duas funções do Azure para mover dados em uma instância do Azure digital gêmeos (você pode aprender sobre esse cenário mais detalhadamente no [*tutorial: conectar uma solução de ponta a ponta*](tutorial-end-to-end.md)). Ele também contém um aplicativo de exemplo *DeviceSimulator* que simula um dispositivo IOT, gerando um novo valor de temperatura a cada segundo.
    - Se você ainda não baixou o exemplo como parte do tutorial em [*pré-requisitos*](#prerequisites), navegue até o [link](/samples/azure-samples/digital-twins-samples/digital-twins-samples/) de exemplo e selecione o botão *Procurar código* abaixo do título. Isso levará você a um repositório GitHub para obter exemplos que poderão ser baixados como um arquivo *.zip* clicando no botão *Código*, depois em *Baixar arquivo .zip*.

        :::image type="content" source="media/includes/download-repo-zip.png" alt-text="Uma exibição do repositório de gêmeos digitais de exemplo no GitHub. Após clicar no botão Código, uma pequena caixa de diálogo será exibida, na qual o botão Baixar arquivo .zip estará realçado." lightbox="media/includes/download-repo-zip.png":::

    Isso baixará uma cópia do repositório de exemplo em seu computador, como **digital-twins-samples-master.zip**. Descompacte a pasta.
* [**Exemplo de aplicativo Web de integração do signalr**](/samples/azure-samples/digitaltwins-signalr-webapp-sample/digital-twins-samples/): Este é um aplicativo Web de reação de exemplo que consumirá dados de telemetria do gêmeos digital do Azure de um serviço de Signaler do Azure.
    -  Navegue até o link de exemplo e use o mesmo processo de download para baixar uma cópia do exemplo em seu computador, como _**digitaltwins-signalr-webapp-sample-main.zip**_. Descompacte a pasta.

[!INCLUDE [Create instance](../azure-signalr/includes/signalr-quickstart-create-instance.md)]

Deixe a janela do navegador aberta na portal do Azure, pois você a usará novamente na próxima seção.

## <a name="publish-and-configure-the-azure-functions-app"></a>Publicar e configurar o aplicativo Azure Functions

Nesta seção, você configurará duas funções do Azure:
* **Negotiate** -uma função de gatilho http. Ele usa a associação de entrada *SignalRConnectionInfo* para gerar e retornar informações de conexão válidas.
* **Broadcast** -uma função de gatilho de [grade de eventos](../event-grid/overview.md) . Ele recebe dados de telemetria do gêmeos digital do Azure por meio da grade de eventos e usa a associação de saída da instância do *signalr* criada na etapa anterior para transmitir a mensagem a todos os aplicativos cliente conectados.

Inicie o Visual Studio (ou outro editor de código de sua escolha) e abra a solução de código na pasta *digital-gêmeos-Samples-master > ADTSampleApp* . Em seguida, execute as seguintes etapas para criar as funções:

1. No projeto *SampleFunctionsApp* , crie uma nova classe C# chamada **SignalRFunctions. cs**.

1. Substitua o conteúdo do arquivo de classe pelo código a seguir:
    
    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/signalRFunction.cs":::

1. Na janela do *console do Gerenciador de pacotes* do Visual Studio ou em qualquer janela de comando em seu computador, navegue até a pasta *digital-Twins-Samples-master\AdtSampleApp\SampleFunctionsApp* e execute o seguinte comando para instalar o `SignalRService` pacote NuGet no projeto:
    ```cmd
    dotnet add package Microsoft.Azure.WebJobs.Extensions.SignalRService --version 1.2.0
    ```

    Isso deve resolver quaisquer problemas de dependência na classe.

1. Publique sua função no Azure, usando as etapas descritas na [seção *publicar o aplicativo*](tutorial-end-to-end.md#publish-the-app) do tutorial *conectar uma solução de ponta a ponta* . Você pode publicá-lo no mesmo serviço de aplicativo/aplicativo de funções que você usou no [pré-requisito](#prerequisites)do tutorial de ponta a ponta ou criar um novo — mas talvez queira usar o mesmo para minimizar a duplicação. 

Em seguida, configure as funções para se comunicar com sua instância do Signalr do Azure. Você começará coletando a **cadeia de conexão** da instância do signalr e, em seguida, a adicionará às configurações do aplicativo functions.

1. Vá para a [portal do Azure](https://portal.azure.com/) e procure o nome da sua instância do signalr na barra de pesquisa na parte superior do Portal. Selecione a instância para abri-la.
1. Selecione **chaves** no menu instância para exibir as cadeias de conexão para a instância do serviço signalr.
1. Selecione o ícone de *cópia* para copiar a cadeia de conexão primária.

    :::image type="content" source="media/how-to-integrate-azure-signalr/signalr-keys.png" alt-text="Captura de tela da portal do Azure que mostra a página de chaves para a instância do Signalr. O ícone ' copiar para área de transferência ' ao lado da cadeia de conexão primária é realçado." lightbox="media/how-to-integrate-azure-signalr/signalr-keys.png":::

1. Por fim, adicione a cadeia de **conexão** do signalr do Azure às configurações do aplicativo da função, usando o comando CLI do Azure a seguir. Além disso, substitua os espaços reservados pelo seu grupo de recursos e pelo nome do aplicativo/função do serviço de aplicativo do [pré-requisito do tutorial](how-to-integrate-azure-signalr.md#prerequisites). O comando pode ser executado no [Azure cloud Shell](https://shell.azure.com)ou localmente se você tiver o CLI do Azure [instalado em seu computador](/cli/azure/install-azure-cli):
 
    ```azurecli-interactive
    az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "AzureSignalRConnectionString=<your-Azure-SignalR-ConnectionString>"
    ```

    A saída desse comando imprime todas as configurações de aplicativo configuradas para sua função do Azure. Procure `AzureSignalRConnectionString` na parte inferior da lista para verificar se ela foi adicionada.

    :::image type="content" source="media/how-to-integrate-azure-signalr/output-app-setting.png" alt-text="Trecho da saída em uma janela de comando, mostrando um item de lista chamado ' AzureSignalRConnectionString '":::

#### <a name="connect-the-function-to-event-grid"></a>Conectar a função à Grade de Eventos

Em seguida, assine a *transmissão* do Azure function para o **tópico da grade de eventos** criado durante o pré-requisito do [tutorial](how-to-integrate-azure-signalr.md#prerequisites). Isso permitirá que os dados de telemetria fluam do thermostat67 para cima através do tópico da grade de eventos e da função. A partir daqui, a função pode transmitir os dados para todos os clientes.

Para fazer isso, você criará uma **assinatura de evento** do tópico da grade de eventos para sua função de *difusão* do Azure como um ponto de extremidade.

No [portal do Azure](https://portal.azure.com/), navegue até o tópico da grade de eventos pesquisando pelo nome dele na barra de pesquisa superior. Selecione *+ Assinatura de Evento*.

:::image type="content" source="media/how-to-integrate-azure-signalr/event-subscription-1b.png" alt-text="Portal do Azure: assinatura de evento da Grade de Eventos":::

Na página *Criar Assinatura de Evento*, preencha os campos da seguinte maneira (os campos preenchidos por padrão não são mencionados):
* *DETALHES DA ASSINATURA DE EVENTO* > **Nome**: dê um nome à assinatura de evento.
* *DETALHES DO PONTO DE EXTREMIDADE* > **Tipo de Ponto de Extremidade**: selecione *Função do Azure* nas opções de menu.
* *DETALHES DO PONTO DE EXTREMIDADE* > **Ponto de Extremidade**: clique no link *Selecionar um ponto de extremidade*. Isso abrirá uma janela *Selecionar Função do Azure*:
    - Preencha sua **assinatura**, **grupo de recursos**, **aplicativo de funções** e **função** (*difusão*). Alguns deles poderão ser preenchidos automaticamente após a seleção da assinatura.
    - Clique em **Confirmar seleção**.

:::image type="content" source="media/how-to-integrate-azure-signalr/create-event-subscription.png" alt-text="Portal do Azure exibição da criação de uma assinatura de evento. Os campos acima são preenchidos e os botões ' confirmar seleção ' e ' criar ' são realçados.":::

Na página *Criar Assinatura de Evento*, selecione **Criar**.

Neste ponto, você deve ver duas assinaturas de evento na página de *Tópicos da grade de eventos* .

:::image type="content" source="media/how-to-integrate-azure-signalr/view-event-subscriptions.png" alt-text="Portal do Azure exibição de duas assinaturas de evento na página de tópicos da grade de eventos." lightbox="media/how-to-integrate-azure-signalr/view-event-subscriptions.png":::

## <a name="configure-and-run-the-web-app"></a>Configurar e executar o aplicativo Web

Nesta seção, você verá o resultado em ação. Primeiro, configure o **aplicativo Web do cliente de exemplo** para se conectar ao fluxo do Azure signalr que você configurou. Em seguida, você iniciará o **aplicativo de exemplo de dispositivo simulado** que envia dados de telemetria por meio da instância do gêmeos digital do Azure. Depois disso, você exibirá o aplicativo Web de exemplo para ver os dados do dispositivo simulado atualizando o aplicativo Web de exemplo em tempo real.

### <a name="configure-the-sample-client-web-app"></a>Configurar o aplicativo Web do cliente de exemplo

Em seguida, você configurará o aplicativo Web do cliente de exemplo. Comece coletando a **URL do ponto de extremidade http** da função *Negotiate* e, em seguida, use-a para configurar o código do aplicativo em seu computador.

1. Vá para a página de [aplicativos da função](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Web%2Fsites/kind/functionapp) do portal do Azure e selecione seu aplicativo de funções na lista. No menu do aplicativo, selecione *funções* e escolha a função *Negotiate* .

    :::image type="content" source="media/how-to-integrate-azure-signalr/functions-negotiate.png" alt-text="Portal do Azure exibição do aplicativo de funções, com ' Functions ' realçado no menu. A lista de funções é mostrada na página e a função ' Negotiate ' também é realçada.":::

1. Pressione *obter URL da função* e copie o valor **para cima por meio de _/API_ (não inclua o último _/Negotiate?_)**. Você usará isso na próxima etapa.

    :::image type="content" source="media/how-to-integrate-azure-signalr/get-function-url.png" alt-text="Portal do Azure exibição da função ' Negotiate '. O botão ' obter URL da função ' está realçado e a parte da URL do início por meio de '/API '":::

1. Usando o Visual Studio ou qualquer editor de código de sua escolha, abra a pasta descompactada _**digitaltwins-signaler-webapp-Sample-Main**_ que você baixou na seção [*baixar aplicativos de exemplo*](#download-the-sample-applications) .

1. Abra o arquivo *src/App.js* e substitua a URL da função `HubConnectionBuilder` por pela URL do ponto de extremidade http da função **Negotiate** que você salvou na etapa anterior:

    ```javascript
        const hubConnection = new HubConnectionBuilder()
            .withUrl('<Function URL>')
            .build();
    ```
1. No *prompt de comando do desenvolvedor* do Visual Studio ou em qualquer janela de comando em seu computador, navegue até a pasta *digitaltwins-signalr-webapp-Sample-main\src* Execute o seguinte comando para instalar os pacotes de nó dependente:

    ```cmd
    npm install
    ```

Em seguida, defina as permissões em seu aplicativo de funções no portal do Azure:
1. Na página de [aplicativos da função](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Web%2Fsites/kind/functionapp) do portal do Azure, selecione sua instância do aplicativo de funções.

1. Role para baixo no menu de instância e selecione *CORS*. Na página CORS, adicione `http://localhost:3000` como uma origem permitida inserindo-a na caixa vazia. Marque a caixa *habilitar acesso-controle-permitir-credenciais* e clique em *salvar*.

    :::image type="content" source="media/how-to-integrate-azure-signalr/cors-setting-azure-function.png" alt-text="Configuração de CORS no Azure function":::

### <a name="run-the-device-simulator"></a>Executar o simulador de dispositivo

Durante o pré-requisito do tutorial de ponta a ponta, você [configurou o simulador de dispositivo](tutorial-end-to-end.md#configure-and-run-the-simulation) para enviar dados por meio de um hub IOT e para sua instância de gêmeos digital do Azure.

Agora, tudo o que você precisa fazer é iniciar o projeto de simulador, localizado em *digital-gêmeos-Samples-master > DeviceSimulator > DeviceSimulator. sln*. Se você estiver usando o Visual Studio, poderá abrir o projeto e executá-lo com esse botão na barra de ferramentas:

:::image type="content" source="media/how-to-integrate-azure-signalr/start-button-simulator.png" alt-text="O botão de início do Visual Studio (projeto DeviceSimulator)":::

Uma janela do console será aberta e exibirá mensagens da telemetria de temperatura simulada. Eles estão sendo enviados por meio de sua instância do gêmeos digital do Azure, em que eles são coletados pelo Azure Functions e Signalr.

Você não precisa fazer mais nada neste console, mas deixá-lo em execução enquanto conclui a próxima etapa.

### <a name="see-the-results"></a>Confira os resultados

Para ver os resultados em ação, inicie o **exemplo de aplicativo Web de integração do signalr**. Você pode fazer isso em qualquer janela de console no local *digitaltwins-signalr-webapp-Sample-main\src* , executando este comando:

```cmd
npm start
```

Isso abrirá uma janela do navegador que executa o aplicativo de exemplo, que exibe um medidor de temperatura Visual. Depois que o aplicativo estiver em execução, você deverá começar a ver os valores de telemetria de temperatura do simulador de dispositivo que se propagam por meio do Azure digital gêmeos sendo refletido pelo aplicativo Web em tempo real.

:::image type="content" source="media/how-to-integrate-azure-signalr/signalr-webapp-output.png" alt-text="Trecho do aplicativo Web do cliente de exemplo, mostrando um medidor de temperatura Visual. A temperatura refletida é 67,52":::

## <a name="clean-up-resources"></a>Limpar recursos

Se você não precisar mais dos recursos criados neste artigo, siga estas etapas para excluí-los. 

Usando o CLI do Azure de Azure Cloud Shell ou local, você pode excluir todos os recursos do Azure em um grupo de recursos com o comando [AZ Group Delete](/cli/azure/group#az-group-delete) . Remover o grupo de recursos também será removido...
* a instância do gêmeos digital do Azure (do tutorial de ponta a ponta)
* o Hub IoT e o registro do dispositivo de Hub (do tutorial de ponta a ponta)
* o tópico da grade de eventos e as assinaturas associadas
* o aplicativo Azure Functions, incluindo todas as três funções e recursos associados, como armazenamento
* a instância do Signalr do Azure

> [!IMPORTANT]
> A exclusão de um grupo de recursos é irreversível. O grupo de recursos e todos os recursos contidos nele são excluídos permanentemente. Não exclua acidentalmente o grupo de recursos ou os recursos incorretos. 

```azurecli-interactive
az group delete --name <your-resource-group>
```

Por fim, exclua as pastas de exemplo do projeto que você baixou para o computador local (*digital-twins-samples-master.zip*, *digitaltwins-signalr-webapp-sample-main.zip* e suas contrapartes descompactadas).

## <a name="next-steps"></a>Próximas etapas

Neste artigo, você configura o Azure Functions com o Signalr para difundir eventos de telemetria do gêmeos digital do Azure para um aplicativo cliente de exemplo.

Em seguida, saiba mais sobre o serviço do Azure Signalr:
* [*O que é o Serviço do Azure SignalR?*](../azure-signalr/signalr-overview.md)

Ou Leia mais sobre a autenticação do serviço de Signaler do Azure com o Azure Functions:
* [*Autenticação do Serviço Azure SignalR*](../azure-signalr/signalr-tutorial-authenticate-azure-functions.md)