---
title: Guia de Início Rápido – C# para fluxos de dispositivos do Hub IoT do Azure para SSH e RDP
description: Neste início rápido, você executará dois aplicativos C# de exemplo que habilitam cenários de SSH e RDP em um fluxo de dispositivos do Hub IoT.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: quickstart
ms.custom: references_regions
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: 12e26818f86fc4abdc1873d031182fd994c04687
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/29/2021
ms.locfileid: "98624364"
---
# <a name="quickstart-enable-ssh-and-rdp-over-an-iot-hub-device-stream-by-using-a-c-proxy-application-preview"></a>Início Rápido: Habilitar o SSH e o RDP em fluxos de dispositivos do Hub IoT usando o aplicativo proxy do C# (versão prévia)

[!INCLUDE [iot-hub-quickstarts-4-selector](../../includes/iot-hub-quickstarts-4-selector.md)]

Atualmente, o Hub IoT do Microsoft Azure dá suporte a fluxos de dispositivos como uma [versão prévia do recurso](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Os [fluxos de dispositivos do Hub IoT](iot-hub-device-streams-overview.md) permitem que aplicativos de serviço e dispositivo se comuniquem de maneira segura e simples para o firewall. Este guia de início rápido envolve dois aplicativos C# que permitem que o tráfego de aplicativo de cliente-servidor, como o SSH (Secure Shell) e o RDP (protocolo RDP), seja enviado em um fluxo de dispositivos estabelecido por meio de um hub IoT. Para obter uma visão geral da configuração, confira [Amostra de aplicativo proxy local para o SSH ou RDP](iot-hub-device-streams-overview.md#local-proxy-sample-for-ssh-or-rdp).

Este artigo descreve primeiro a configuração do SSH (usando a porta 22) e, em seguida, descreve como modificar a porta da configuração do RDP. Como os fluxos de dispositivos são independentes de protocolo e de aplicativo, a mesma amostra pode ser modificada para acomodar outros tipos de tráfego do aplicativo. Essa modificação geralmente envolve apenas a alteração da porta de comunicação para aquela usada pelo aplicativo pretendido.

## <a name="prerequisites"></a>Pré-requisitos

* Atualmente, a versão prévia dos fluxos de dispositivos só é compatível com hubs IoT criados nas seguintes regiões:

  * Centro dos EUA
  * EUA Central EUAP
  * Sudeste Asiático
  * Norte da Europa

* Os dois aplicativos de exemplo executados neste início rápido foram escritos em C#. É necessário ter o SDK do .NET Core 2.1.0 ou posterior no computador de desenvolvimento.

    Você pode baixar o [SDK do .NET Core para várias plataformas a partir do .NET](https://www.microsoft.com/net/download/all).

    Verifique a versão atual do C# no computador de desenvolvimento usando o seguinte comando:

    ```
    dotnet --version
    ```

* [Baixe os exemplos de C# do Azure IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip) e extraia o arquivo ZIP.

* Uma conta de usuário e uma credencial válidas no dispositivo (Windows ou Linux) usadas para autenticar o usuário.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="how-it-works"></a>Como ele funciona

A figura a seguir ilustra como os aplicativos proxy locais do dispositivo e do serviço nesta amostra permitem a conectividade de ponta a ponta entre os processos do cliente SSH e do daemon SSH. Aqui, presumimos que o daemon esteja em execução no mesmo dispositivo do aplicativo proxy local do dispositivo.

![Configuração do aplicativo proxy local](./media/quickstart-device-streams-proxy-csharp/device-stream-proxy-diagram.png)

1. O aplicativo proxy local do serviço se conecta ao hub IoT e inicia um fluxo de dispositivos para o dispositivo de destino.

1. O aplicativo proxy local do dispositivo conclui o handshake de inicialização do fluxo e estabelece um túnel de streaming de ponta a ponta por meio do ponto de extremidade de streaming do hub IoT até o lado do serviço.

1. O aplicativo proxy local do dispositivo conecta-se ao daemon SSH que escuta na porta 22 do dispositivo. Essa configuração é definível, conforme descrito na seção "Executar o aplicativo proxy local do dispositivo".

1. O aplicativo proxy local do serviço aguarda novas conexões SSH de um usuário por meio da escuta em uma porta designada que, nesse caso, é a porta 2222. Essa configuração é definível, conforme descrito na seção "Executar o aplicativo proxy local do serviço". Quando o usuário se conecta por meio do cliente SSH, o túnel permite que o tráfego de aplicativo SSH seja transferido entre os aplicativos de cliente e servidor SSH.

> [!NOTE]
> O tráfego SSH enviado por um fluxo de dispositivos é passado por um túnel pelo ponto de extremidade de streaming do hub IoT, em vez de ser enviado diretamente entre o serviço e o dispositivo. Para obter mais informações, confira os [benefícios do uso de fluxos de dispositivos do Hub IoT](iot-hub-device-streams-overview.md#benefits).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-an-iot-hub"></a>Crie um hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar um dispositivo

Um dispositivo deve ser registrado no hub IoT antes de poder se conectar. Neste início rápido, você usará o Azure Cloud Shell para registrar um dispositivo simulado.

1. Para criar a identidade do dispositivo, execute o seguinte comando no Cloud Shell:

   > [!NOTE]
   > * Substitua o espaço reservado *YourIoTHubName* com o nome escolhido para o hub IoT.
   > * Para o nome do dispositivo que você está registrando, é recomendável usar *MyDevice* conforme mostrado. Se você escolher um nome diferente para seu dispositivo, use esse nome ao longo deste artigo e atualize o nome do dispositivo nos aplicativos de exemplo antes de executá-los.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

1. Para obter a *cadeia de conexão de dispositivo* referente ao dispositivo que você acabou de registrar, execute os seguintes comandos no Cloud Shell:

   > [!NOTE]
   > Substitua o espaço reservado *YourIoTHubName* com o nome escolhido para o hub IoT.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name {YourIoTHubName} --device-id MyDevice --output table
    ```

    Anote a cadeia de conexão de dispositivo retornada para uso posterior neste início rápido. Ela se parece com o seguinte exemplo:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyDevice;SharedAccessKey={YourSharedAccessKey}`

1. Para se conectar ao hub IoT e estabelecer um fluxo de dispositivos, você também precisa da *cadeia de conexão de serviço* do hub IoT para habilitar o aplicativo do lado do serviço. O comando a seguir recupera esse valor para o Hub IoT:

   > [!NOTE]
   > Substitua o espaço reservado *YourIoTHubName* com o nome escolhido para o hub IoT.

    ```azurecli-interactive
    az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
    ```

    Anote a cadeia de conexão de serviço retornada para uso posterior neste início rápido. Ela se parece com o seguinte exemplo:

   `"HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"`

## <a name="ssh-to-a-device-via-device-streams"></a>SSH para um dispositivo por fluxos de dispositivos

Nesta seção, você estabelecerá um fluxo de ponta a ponta para encapsular o tráfego SSH.

### <a name="run-the-device-local-proxy-application"></a>Executar o aplicativo de proxy no local do dispositivo

Em uma janela do terminal local, navegue até o diretório `device-streams-proxy/device` na pasta descompactada do projeto. Mantenha as seguintes informações acessíveis:

| Nome do argumento | Valor do argumento |
|----------------|-----------------|
| `DeviceConnectionString` | A cadeia de conexão do dispositivo que você criou anteriormente. |
| `targetServiceHostName` | O endereço IP no qual o servidor SSH escuta. O endereço será `localhost` se ele for o mesmo IP no qual o aplicativo proxy local do dispositivo está em execução. |
| `targetServicePort` | A porta usada pelo protocolo de aplicativo (para o SSH, por padrão, essa será a porta 22).  |

Compile e execute o código com os seguintes comandos:

```
cd ./iot-hub/Quickstarts/device-streams-proxy/device/

# Build the application
dotnet build

# Run the application
# In Linux or macOS
dotnet run ${DeviceConnectionString} localhost 22

# In Windows
dotnet run {DeviceConnectionString} localhost 22
```

### <a name="run-the-service-local-proxy-application"></a>Executar o aplicativo de proxy no local do serviço

Em outra janela de terminal local, navegue até `iot-hub/quickstarts/device-streams-proxy/service` na pasta descompactada do projeto. Mantenha as seguintes informações acessíveis:

| Nome do parâmetro | Valor do parâmetro |
|----------------|-----------------|
| `ServiceConnectionString` | A cadeia de conexão de serviço do seu Hub IoT. |
| `MyDevice` | O identificador do dispositivo que você criou anteriormente. |
| `localPortNumber` | Uma porta local à qual o cliente SSH se conectará. Usamos a porta 2222 nesta amostra, mas você pode usar outros números arbitrários. |

Compile e execute o código com os seguintes comandos:

```
cd ./iot-hub/Quickstarts/device-streams-proxy/service/

# Build the application
dotnet build

# Run the application
# In Linux or macOS
dotnet run ${ServiceConnectionString} MyDevice 2222

# In Windows
dotnet run {ServiceConnectionString} MyDevice 2222
```

### <a name="run-the-ssh-client"></a>Executar o cliente SSH

Agora, use o aplicativo cliente SSH e se conecte ao aplicativo proxy local do serviço na porta 2222 (em vez de no daemon SSH diretamente).

```
ssh {username}@localhost -p 2222
```

Neste ponto, a janela de entrada do SSH solicitará que você insira suas credenciais.

Saída do console no lado do serviço (o aplicativo proxy local do serviço escuta na porta 2222):

![Saída do aplicativo proxy local do serviço](./media/quickstart-device-streams-proxy-csharp/service-console-output.png)

Saída do console no aplicativo proxy local do dispositivo que se conecta ao daemon SSH em *IP_address:22*:

![Saída do aplicativo proxy local do dispositivo](./media/quickstart-device-streams-proxy-csharp/device-console-output.png)

Saída do console do aplicativo de cliente SSH. O cliente SSH se comunica com o daemon SSH conectando-se à porta 22, na qual o aplicativo proxy local do serviço escuta:

![Saída do aplicativo cliente SSH](./media/quickstart-device-streams-proxy-csharp/ssh-console-output.png)

## <a name="rdp-to-a-device-via-device-streams"></a>RDP para um dispositivo por fluxos de dispositivos

A configuração do RDP é semelhante à configuração do SSH (descrita acima). Em vez disso, use o IP de destino do RDP e a porta 3389 e o cliente RDP (em vez do cliente SSH).

### <a name="run-the-device-local-proxy-application-rdp"></a>Executar o aplicativo proxy local do dispositivo (RDP)

Em uma janela do terminal local, navegue até o diretório `device-streams-proxy/device` na pasta descompactada do projeto. Mantenha as seguintes informações acessíveis:

| Nome do argumento | Valor do argumento |
|----------------|-----------------|
| `DeviceConnectionString` | A cadeia de conexão do dispositivo que você criou anteriormente. |
| `targetServiceHostName` | O nome do host ou o endereço IP em que o servidor RDP é executado. O endereço será `localhost` se ele for o mesmo IP no qual o aplicativo proxy local do dispositivo está em execução. |
| `targetServicePort` | A porta usada pelo protocolo de aplicativo (para o RDP, por padrão, essa será a porta 3389).  |

Compile e execute o código com os seguintes comandos:

```
cd ./iot-hub/Quickstarts/device-streams-proxy/device

# Run the application
# In Linux or macOS
dotnet run ${DeviceConnectionString} localhost 3389

# In Windows
dotnet run {DeviceConnectionString} localhost 3389
```

### <a name="run-the-service-local-proxy-application-rdp"></a>Executar o aplicativo proxy local do serviço (RDP)

Em outra janela de terminal local, navegue até `device-streams-proxy/service` na pasta descompactada do projeto. Mantenha as seguintes informações acessíveis:

| Nome do parâmetro | Valor do parâmetro |
|----------------|-----------------|
| `ServiceConnectionString` | A cadeia de conexão de serviço do seu Hub IoT. |
| `MyDevice` | O identificador do dispositivo que você criou anteriormente. |
| `localPortNumber` | Uma porta local à qual o cliente SSH se conectará. Usamos a porta 2222 neste exemplo, mas você poderia modificar para outros números arbitrários. |

Compile e execute o código com os seguintes comandos:

```
cd ./iot-hub/Quickstarts/device-streams-proxy/service/

# Build the application
dotnet build

# Run the application
# In Linux or macOS
dotnet run ${ServiceConnectionString} MyDevice 2222

# In Windows
dotnet run {ServiceConnectionString} MyDevice 2222
```

### <a name="run-rdp-client"></a>Executar o cliente RDP

Agora, use o aplicativo cliente RDP e se conecte ao aplicativo proxy local do serviço na porta 2222 (essa era uma porta disponível arbitrária que você escolheu anteriormente).

![O RDP se conecta ao aplicativo proxy local do serviço](./media/quickstart-device-streams-proxy-csharp/rdp-screen-capture.png)

## <a name="clean-up-resources"></a>Limpar os recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Próximas etapas

Neste início rápido, você configurou um hub IoT, registrou um dispositivo, implantou aplicativos proxy locais do dispositivo e do serviço para estabelecer um fluxo de dispositivos pelo hub IoT e usou os aplicativos proxy para passar o tráfego SSH ou RDP por um túnel. O mesmo paradigma pode acomodar outros protocolos de cliente-servidor, em que o servidor é executado no dispositivo (por exemplo, o daemon SSH).

Para saber mais sobre fluxos de dispositivos, confira:

> [!div class="nextstepaction"]
> [Visão geral dos fluxos de dispositivos](./iot-hub-device-streams-overview.md)
