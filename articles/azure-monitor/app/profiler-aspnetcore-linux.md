---
title: Criar o perfil de Aplicativos Web ASP.NET Core Azure Linux com o Application Insights Profiler | Microsoft Docs
description: Uma visão geral conceitual e um tutorial passo a passo sobre como usar o Application Insights Profiler.
services: application-insights
documentationcenter: ''
author: cweining
manager: carmonm
ms.reviewer: mbullwin
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 02/23/2018
ms.author: cweining
ms.openlocfilehash: 35789cc1e516fb24d5e985e12b44fe3cd01b795d
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60306443"
---
# <a name="profile-aspnet-core-azure-linux-web-apps-with-application-insights-profiler"></a>Criar o perfil de aplicativos web ASP.NET Core Azure Linux com o Application Insights Profiler

Esse recurso está atualmente na visualização.

Descubra quanto tempo é gasto em cada método do seu aplicativo Web em tempo real ao usar o [Application Insights](../../azure-monitor/app/app-insights-overview.md). O Application Insights Profiler agora está disponível para aplicativos de web do ASP.NET core hospedados no Linux no Serviço de Aplicativo do Azure. Este guia fornece instruções passo a passo sobre como os rastreamentos do Profiler podem ser coletados para aplicativos web do ASP.NET Core Linux.

Depois de concluir este passo a passo, seu aplicativo poderá coletar rastreamentos de Profiler como os rastreamentos que são mostrados na imagem. Neste exemplo, o rastreamento do Profiler indica que uma solicitação da web específica está lenta por causa do tempo gasto na espera. O *afunilamento* no código que está reduzindo a velocidade do aplicativo é marcado por um ícone de chamas. O método **Sobre** na seção **HomeController** seção está reduzindo a velocidade do aplicativo web porque está chamando a função **Thread.Sleep**.

![Rastreamentos do Criador de Perfil](./media/profiler-aspnetcore-linux/profiler-traces.png)

## <a name="prerequisites"></a>Pré-requisitos
As seguintes instruções se aplicam a todos os ambientes de desenvolvimento do Windows, Linux e Mac:

* Instale o [SDK 2.1.2 do .NET Core ou posterior](https://dotnet.microsoft.com/download/archives).
* Instale o Git seguindo as instruções em [Introdução - Instalação do Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

## <a name="set-up-the-project-locally"></a>Configurar o projeto localmente

1. Abra uma janela do prompt de comando no seu computador. As seguintes instruções são compatíveis com todos os ambientes de desenvolvimento do Windows, Linux e Mac.

1. Crie um aplicativo Web MVC ASP.NET Core:

    ```
    dotnet new mvc -n LinuxProfilerTest
    ```

1. Altere o diretório de trabalho para a pasta raiz do projeto.

1. Adicione o pacote do NuGet para coletar os rastreamentos do Profiler:

    ```shell
    dotnet add package Microsoft.ApplicationInsights.Profiler.AspNetCore
    ```

1. Habilite o Application Insights em Program.cs:

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseApplicationInsights() // Add this line of code to Enable Application Insights
            .UseStartup<Startup>();
    ```
    
1. Habilite o Profiler em Startup.cs:

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddServiceProfiler(); // Add this line of code to Enable Profiler
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
    }
    ```

1. Adicione uma linha de código na seção **HomeController.cs** para atrasar aleatoriamente alguns segundos:

    ```csharp
        using System.Threading;
        ...

        public IActionResult About()
            {
                Random r = new Random();
                int delay = r.Next(5000, 10000);
                Thread.Sleep(delay);
                return View();
            }
    ```

1. Salve e confirme suas alterações no repositório local:

    ```
        git init
        git add .
        git commit -m "first commit"
    ```

## <a name="create-the-linux-web-app-to-host-your-project"></a>Criar o aplicativo web para Linux para hospedar seu projeto

1. Crie o ambiente de aplicativo web usando o Serviço de Aplicativo no Linux:

    ![Criar o aplicativo web para Linux](./media/profiler-aspnetcore-linux/create-linux-appservice.png)

2. Crie as credenciais de implantação:

    > [!NOTE]
    > Registre sua senha para usar posteriormente ao implantar seu aplicativo web.

    ![Criar as credenciais de implantação](./media/profiler-aspnetcore-linux/create-deployment-credentials.png)

3. Escolha as opções de implantação. Configure um repositório Git local no aplicativo web seguindo as instruções no Portal do Azure. Um repositório Git é criado automaticamente.

    ![Configurar o repositório Git](./media/profiler-aspnetcore-linux/setup-git-repo.png)

Para obter mais opções de implantação, consulte [este artigo](https://docs.microsoft.com/azure/app-service/containers/choose-deployment-type).

## <a name="deploy-your-project"></a>Implante o seu projeto

1. Na janela do prompt de comando, navegue até a pasta raiz de seu projeto. Adicione um repositório remoto de Git para apontar para o repositório no Serviços de Aplicativos:

    ```
    git remote add azure https://<username>@<app_name>.scm.azurewebsites.net:443/<app_name>.git
    ```

    * Use o **username** que você usou para criar as credenciais de implantação.
    * Use o **nome do aplicativo** que você usou para criar o aplicativo web por meio do Serviço de Aplicativo no Linux.

2. Implante o projeto efetuando push nas alterações para o Azure:

    ```
    git push azure master
    ```

Você deverá ver uma saída semelhante ao exemplo a seguir:

    ```
    Counting objects: 9, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (8/8), done.
    Writing objects: 100% (9/9), 1.78 KiB | 911.00 KiB/s, done.
    Total 9 (delta 3), reused 0 (delta 0)
    remote: Updating branch 'master'.
    remote: Updating submodules.
    remote: Preparing deployment for commit id 'd7369a99d7'.
    remote: Generating deployment script.
    remote: Running deployment command...
    remote: Handling ASP.NET Core Web Application deployment.
    remote: ......
    remote:   Restoring packages for /home/site/repository/EventPipeExampleLinux.csproj...
    remote: .
    remote:   Installing Newtonsoft.Json 10.0.3.
    remote:   Installing Microsoft.ApplicationInsights.Profiler.Core 1.1.0-LKG
    …

    ```

## <a name="add-application-insights-to-monitor-your-web-apps"></a>Adicionar o Application Insights para monitorar seus aplicativos web

1. [Crie um recurso Application Insights](./../../azure-monitor/app/create-new-resource.md ).

2. Copie o valor de **iKey** do recurso do Application Insights e defina as seguintes configurações nos seus aplicativos web:

    ```
    APPINSIGHTS_INSTRUMENTATIONKEY: [YOUR_APPINSIGHTS_KEY]
    ```

    Quando as configurações de aplicativo são alteradas, o site é reiniciado automaticamente. Depois que as novas configurações são aplicadas, o Profiler é executado imediatamente por dois minutos. O Profiler, em seguida, é executado por dois minutos a cada hora.

3. Gere algum tráfego para seu site. Você pode gerar tráfego atualizando a página **Sobre** do site algumas vezes.

4. Aguarde de dois a cinco minutos para os eventos serem agregados ao Application Insights.

5. Navegue até o painel de **Desempenho** do Application Insights no Portal do Azure. Você pode exibir os rastreamentos do Profiler no canto inferior direito do painel.

    ![Exibir rastreamentos do Profiler](./media/profiler-aspnetcore-linux/view-traces.png)

## <a name="known-issues"></a>Problemas conhecidos

### <a name="profile-now-button-doesnt-work-for-linux-profiler"></a>Agora o botão não funciona para Linux Profiler de perfil
A versão do Linux do criador de perfil do App Insights ainda não dá suporte sob demanda de criação de perfil usando o perfil agora botão.


## <a name="next-steps"></a>Próximos passos
Se você usar contêineres personalizados hospedados pelos Serviço de Aplicativo do Azure, siga as instruções em [ Habilitar o Criador de Perfil de Serviço para um aplicativo em contêineres do ASP.NET Core](https://github.com/Microsoft/ApplicationInsights-Profiler-AspNetCore/tree/master/examples/EnableServiceProfilerForContainerApp) para habilitar o Application Insights Profiler.

Relate quaisquer problemas ou sugestões ao repositório do GitHub do Application Insights: [ApplicationInsights-Profiler-AspNetCore: Problemas](https://github.com/Microsoft/ApplicationInsights-Profiler-AspNetCore/issues).
