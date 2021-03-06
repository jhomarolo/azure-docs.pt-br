---
title: Introdução aos aplicativos móveis do Azure para aplicativos Xamarin Android
description: Siga este tutorial para começar a usar os Aplicativos Móveis do Azure para desenvolvimento Android Xamarin
services: app-service\mobile
documentationcenter: xamarin
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 81649dd3-544f-40ff-b9b7-60c66d683e60
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-android
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: crdun
ms.openlocfilehash: 29efa963a254913e3d4744ade1d161c5c8ce42e4
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "62127889"
---
# <a name="create-a-xamarinandroid-app"></a>Criar um Aplicativo Xamarin.Android
[!INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## <a name="overview"></a>Visão geral
Este tutorial mostra como adicionar um serviço de back-end baseado em nuvem a um aplicativo Xamarin Android. Para saber mais, confira [O que são Aplicativos Móveis](app-service-mobile-value-prop.md).

Uma captura de tela do aplicativo completo está disponível abaixo:

![][0]

A conclusão deste tutorial é um pré-requisito para todos os outros tutoriais de Aplicativos Móveis para aplicativos Xamarin.Android.

## <a name="prerequisites"></a>Pré-requisitos
Para concluir este tutorial, você precisará dos seguintes pré-requisitos:

* Uma conta ativa do Azure. Se você não tiver uma conta, inscreva-se para uma avaliação do Azure e obtenha até 10 aplicativos móveis gratuitos. Para obter detalhes, consulte [Avaliação gratuita do Azure](https://azure.microsoft.com/pricing/free-trial/).
* Visual Studio com Xamarin. Veja [Setup and Install for Visual Studio and Xamarin](/visualstudio/cross-platform/setup-and-install) (Configuração e instalação para Visual Studio e Xamarin).

## <a name="create-an-azure-mobile-app-backend"></a>Criar um back-end de aplicativo móvel do Azure
Siga estas etapas para criar um back-end de aplicativo móvel.

[!INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]

Você acabou de provisionar um back-end do aplicativo móvel do Azure que pode ser usado pelos aplicativos móveis clientes. Em seguida, baixe um projeto do servidor para um back-end simples da "lista de tarefas" e publique-o no Azure.

## <a name="configure-the-server-project"></a>Configurar o projeto de servidor
[!INCLUDE [app-service-mobile-configure-new-backend.md](../../includes/app-service-mobile-configure-new-backend.md)]

## <a name="download-and-run-the-xamarinandroid-app"></a>Baixar e executar o aplicativo Xamarin.Android
1. Em **Baixar e executar seu projeto Xamarin.Android**, clique no botão **Baixar**.

      Salve o arquivo do projeto compactado em seu computador local e anote onde ele foi salvo.
2. Pressione a tecla **F5** para compilar o projeto e iniciar o aplicativo.
3. No aplicativo, digite um texto significativo, como *Concluir o tutorial* e depois clique no botão **Adicionar**.

    ![][10]

    Os dados da solicitação são inseridos na tabela TodoItem. Itens armazenados na tabela são retornados pelo back-end do aplicativo móvel e os dados aparecem na lista.

   > [!NOTE]
   > Você pode examinar o código que acessa o back-end do aplicativo móvel para consultar e inserir dados que estão localizados no arquivo ToDoActivity.cs C#.
   >
   >

## <a name="troubleshooting"></a>solução de problemas
Se você tiver problemas para compilar a solução, execute o gerenciador de pacotes NuGet e atualize os pacotes de suporte do `Xamarin.Android`. Os projetos de Início Rápido nem sempre incluem as últimas versões.

Observe que todos os pacotes de suporte referenciados em seu projeto devem ter a mesma versão. O [pacote NuGet de Aplicativos Móveis do Azure](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/) depende de `Xamarin.Android.Support.CustomTabs` para a plataforma Android e, portanto, se o projeto usar pacotes de suporte mais recentes, será necessário instalar diretamente esse pacote com a versão necessária para evitar conflitos.

## <a name="next-steps"></a>Próximas etapas
* [Adicionar sincronização offline ao seu aplicativo](app-service-mobile-xamarin-android-get-started-offline-data.md)
* [Adicionar autenticação ao seu aplicativo](app-service-mobile-xamarin-android-get-started-users.md)
* [Adicionar notificações por push ao seu aplicativo Xamarin.Android](app-service-mobile-xamarin-android-get-started-push.md)
* [Como usar o cliente gerenciado para aplicativos móveis do Azure](app-service-mobile-dotnet-how-to-use-client-library.md)

<!-- Images. -->
[0]: ./media/app-service-mobile-xamarin-android-get-started/mobile-quickstart-completed-android.png
[6]: ./media/app-service-mobile-xamarin-android-get-started/mobile-portal-quickstart-xamarin.png
[8]: ./media/app-service-mobile-xamarin-android-get-started/mobile-xamarin-project-android-vs.png
[9]: ./media/app-service-mobile-xamarin-android-get-started/mobile-xamarin-project-android-xs.png
[10]: ./media/app-service-mobile-xamarin-android-get-started/mobile-quickstart-startup-android.png

<!-- URLs. -->
[Azure Portal]: https://azure.portal.com/
[Visual Studio]: https://go.microsoft.com/fwLink/p/?LinkID=534203
