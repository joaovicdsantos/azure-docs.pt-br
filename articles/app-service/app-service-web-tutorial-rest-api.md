---
title: 'Tutorial: Hospedar uma API RESTful com o CORS'
description: Saiba como o Serviço de Aplicativo do Azure ajuda a hospedar suas APIs RESTful com suporte a CORS. O Serviço de Aplicativo pode hospedar aplicativos Web front-end e APIs de back-end.
ms.assetid: a820e400-06af-4852-8627-12b3db4a8e70
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/28/2020
ms.custom: devx-track-csharp, mvc, devcenter, seo-javascript-september2019, seo-javascript-october2019, seodec18
ms.openlocfilehash: 9481b6d2740d27b8c3d1309e205edda6017868fa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/29/2021
ms.locfileid: "96005735"
---
# <a name="tutorial-host-a-restful-api-with-cors-in-azure-app-service"></a>Tutorial: Hospedar uma API RESTful com CORS no Serviço de Aplicativo do Azure

O [Serviço de Aplicativo do Azure](overview.md) fornece um serviço de hospedagem na Web altamente escalonável e com aplicação automática de patches. Além disso, o Serviço de Aplicativo tem suporte interno a [CORS (Compartilhamento de Recursos entre Origens)](https://wikipedia.org/wiki/Cross-Origin_Resource_Sharing) para APIs RESTful. Este tutorial mostra como implantar um aplicativo de API do ASP.NET Core no Serviço de Aplicativo com suporte a CORS. Configure o aplicativo usando as ferramentas de linha de comando e implante o aplicativo usando o Git. 

Neste tutorial, você aprenderá como:

> [!div class="checklist"]
> * Criar recursos do Serviço de Aplicativo usando a CLI do Azure
> * Implantar uma API RESTful no Azure usando o Git
> * Habilitar o suporte de CORS do Serviço de Aplicativo

Você pode seguir as etapas deste tutorial no macOS, no Linux e no Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Pré-requisitos

Para concluir este tutorial:

* <a href="https://git-scm.com/" target="_blank">Instalar o Git</a>
 * <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Instalar o SDK do .NET Core 3.1 mais recente</a>

## <a name="create-local-aspnet-core-app"></a>Criar um aplicativo ASP.NET Core local

Nesta etapa, você configura o projeto ASP.NET Core local. O Serviço de Aplicativo dá suporte ao mesmo fluxo de trabalho para APIs codificadas em outras linguagens.

### <a name="clone-the-sample-application"></a>Clonar o aplicativo de exemplo

Na janela do terminal, `cd` para um diretório de trabalho.  

Execute o comando a seguir para clonar o repositório de exemplo. 

```bash
git clone https://github.com/Azure-Samples/dotnet-core-api
```

Esse repositório contém um aplicativo que é criado com base no seguinte tutorial: [Páginas de ajuda da API Web do ASP.NET Core usando o Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger?tabs=visual-studio). Ele usa um gerador de Swagger para servir a [interface do usuário do Swagger](https://swagger.io/swagger-ui/) e o ponto de extremidade JSON do Swagger.

### <a name="run-the-application"></a>Executar o aplicativo

Execute os comandos a seguir para instalar os pacotes necessários, executar as migrações de banco de dados e iniciar o aplicativo.

```bash
cd dotnet-core-api
dotnet restore
dotnet run
```

Navegue até `http://localhost:5000/swagger` em um navegador para trabalhar com a interface do usuário do Swagger.

![API do ASP.NET Core em execução localmente](./media/app-service-web-tutorial-rest-api/azure-app-service-local-swagger-ui.png)

Navegue até `http://localhost:5000/api/todo` e veja uma lista de itens JSON de tarefas pendentes.

Navegue até `http://localhost:5000` e experimente o aplicativo de navegador. Posteriormente, você vai apontar o aplicativo de navegador para uma API remota no Serviço de Aplicativo a fim de testar a funcionalidade CORS. O código do aplicativo de navegador é encontrado no diretório _wwwroot_ do repositório.

Para parar o ASP.NET Core a qualquer momento, pressione `Ctrl+C` no terminal.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-app-to-azure"></a>Implantar o aplicativo no Azure

Nesta etapa, você implantará seu aplicativo .NET Core conectado ao Banco de Dados SQL no Serviço de Aplicativo.

### <a name="configure-local-git-deployment"></a>Configurar a implantação do git local

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-a-resource-group"></a>Criar um grupo de recursos

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)]

### <a name="create-an-app-service-plan"></a>Criar um plano de Serviço de Aplicativo

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-no-h.md)]

### <a name="create-a-web-app"></a>Criar um aplicativo Web

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-dotnetcore-win-no-h.md)] 

### <a name="push-to-azure-from-git"></a>Enviar do Git para o Azure

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Enumerating objects: 83, done.
Counting objects: 100% (83/83), done.
Delta compression using up to 8 threads
Compressing objects: 100% (78/78), done.
Writing objects: 100% (83/83), 22.15 KiB | 3.69 MiB/s, done.
Total 83 (delta 26), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '509236e13d'.
remote: Generating deployment script.
remote: Project file path: .\TodoApi.csproj
remote: Generating deployment script for ASP.NET MSBuild16 App
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling ASP.NET Core Web Application deployment with MSBuild16.
remote: .
remote: .
remote: .
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Triggering recycle (preview mode disabled).
remote: Deployment successful.
To https://&lt;app_name&gt;.scm.azurewebsites.net/&lt;app_name&gt;.git
 * [new branch]      master -> master
</pre>

### <a name="browse-to-the-azure-app"></a>Navegar até o aplicativo do Azure

Navegue até `http://<app_name>.azurewebsites.net/swagger` em um navegador e experimente a interface do usuário do Swagger.

![API do ASP.NET Core em execução no Serviço de Aplicativo do Azure](./media/app-service-web-tutorial-rest-api/azure-app-service-browse-app.png)

Navegue até `http://<app_name>.azurewebsites.net/swagger/v1/swagger.json` para ver o _swagger.json_ da sua API implantada.

Navegue até `http://<app_name>.azurewebsites.net/api/todo` para ver sua API implantada funcionando.

## <a name="add-cors-functionality"></a>Adicionar funcionalidade CORS

Em seguida, habilite o suporte interno de CORS no Serviço de Aplicativo para a sua API.

### <a name="test-cors-in-sample-app"></a>Testar CORS no aplicativo de exemplo

No seu repositório local, abra _wwwroot/index.html_.

Na linha 51, defina a variável `apiEndpoint` como a URL da sua API implantada (`http://<app_name>.azurewebsites.net`). Substitua _\<appname>_ pelo nome do aplicativo no Serviço de Aplicativo.

Na janela do terminal local, execute novamente o aplicativo de exemplo.

```bash
dotnet run
```

Navegue até o aplicativo de navegador em `http://localhost:5000`. Abra a janela de ferramentas de desenvolvedor no navegador (`Ctrl`+`Shift`+`i` no Chrome para Windows) e inspecione a guia **Console**. Agora, você verá a mensagem de erro `No 'Access-Control-Allow-Origin' header is present on the requested resource`.

![Erro de CORS no cliente de navegador](./media/app-service-web-tutorial-rest-api/azure-app-service-cors-error.png)

Devido à incompatibilidade de domínio entre o aplicativo de navegador (`http://localhost:5000`) e o recurso remoto (`http://<app_name>.azurewebsites.net`), e ao fato de sua API no Serviço de Aplicativo não estar enviando o cabeçalho `Access-Control-Allow-Origin`, seu navegador impediu que o conteúdo entre domínios fosse carregado no aplicativo de navegador.

Na produção, o aplicativo de navegador teria uma URL pública em vez da URL de localhost, mas a maneira de habilitar o CORS para uma URL de localhost é a mesma de uma URL pública.

### <a name="enable-cors"></a>Habilitar CORS 

No Cloud Shell, habilite o CORS para URL do cliente usando o comando [`az webapp cors add`](/cli/azure/webapp/cors#az-webapp-cors-add). Substitua o espaço reservado _&lt;app-name>_ .

```azurecli-interactive
az webapp cors add --resource-group myResourceGroup --name <app-name> --allowed-origins 'http://localhost:5000'
```

Você pode definir mais de uma URL de cliente `properties.cors.allowedOrigins` (`"['URL1','URL2',...]"`). Você também pode habilitar todas as URLs de cliente com `"['*']"`.

> [!NOTE]
> Se o seu aplicativo exigir credenciais, como o envio de cookies ou tokens de autenticação, o navegador poderá exigir o cabeçalho `ACCESS-CONTROL-ALLOW-CREDENTIALS` na resposta. Para habilitar isso no Serviço de Aplicativo, defina `properties.cors.supportCredentials` como `true` em sua configuração de CORS. Isso não pode ser habilitado quando `allowedOrigins` inclui `'*'`.

### <a name="test-cors-again"></a>Testar CORS novamente

Atualize o aplicativo de navegador em `http://localhost:5000`. A mensagem de erro na janela **Console** desapareceu e você pode ver os dados da API implantada e interagir com eles. Sua API remota agora dá suporte a CORS para o aplicativo de navegador em execução localmente. 

![Êxito do CORS no cliente de navegador](./media/app-service-web-tutorial-rest-api/azure-app-service-cors-success.png)

Parabéns, você estiver executando uma API no Serviço de Aplicativo do Azure com suporte de CORS.

## <a name="app-service-cors-vs-your-cors"></a>CORS do Serviço de Aplicativo comparado com seu CORS

Você pode usar seus próprios utilitários CORS em vez do CORS do Serviço de Aplicativo para obter mais flexibilidade. Por exemplo, convém especificar diferentes origens permitidas para rotas ou métodos diferentes. Como o CORS do Serviço de Aplicativo permite especificar um conjunto de origens aceitas para todas as rotas e métodos de API, é melhor usar seu próprio código CORS. Veja como o ASP.NET Core faz isso em [Habilitando solicitações entre origens (CORS)](/aspnet/core/security/cors).

> [!NOTE]
> Não tente usar CORS do Serviço de Aplicativo e seu próprio CORS juntos. Quando usados juntos, o CORS do Serviço de Aplicativo terá precedência e seu próprio código CORS não terá nenhum efeito.
>
>

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Próximas etapas

O que você aprendeu:

> [!div class="checklist"]
> * Criar recursos do Serviço de Aplicativo usando a CLI do Azure
> * Implantar uma API RESTful no Azure usando o Git
> * Habilitar o suporte de CORS do Serviço de Aplicativo

Vá para o próximo tutorial e aprenda a autenticar e autorizar os usuários.

> [!div class="nextstepaction"]
> [Tutorial: autenticar e autorizar usuários de ponta a ponta](tutorial-auth-aad.md)
