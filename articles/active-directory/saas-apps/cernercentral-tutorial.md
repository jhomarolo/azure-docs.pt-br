---
title: 'Tutorial: Integração do Azure Active Directory com o Cerner Central | Microsoft Docs'
description: Saiba como configurar o logon único entre o Azure Active Directory e o Cerner Central.
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.assetid: d2bc549d-d286-4679-854e-bb67c62b0475
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2018
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: c097fb045db1afe65a84a2a96dc202c57e8a449e
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60428402"
---
# <a name="tutorial-azure-active-directory-integration-with-cerner-central"></a>Tutorial: Integração do Azure Active Directory com o Cerner Central

Neste tutorial, você aprenderá a integrar o Cerner Central ao Azure AD (Azure Active Directory).

A integração do Cerner Central ao Azure AD oferece os seguintes benefícios:

- No Azure AD, é possível controlar quem tem acesso ao Cerner Central
- Permitir que seus usuários façam logon automaticamente no Cerner Central (logon único) com as respectivas contas do Azure AD
- Você pode gerenciar suas contas em um única localização: o Portal do Azure

Para conhecer mais detalhadamente a integração de aplicativos de SaaS ao Azure AD, consulte [o que é o acesso a aplicativos e logon único com o Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Pré-requisitos

Para configurar a integração do Azure AD ao Cerner Central, você precisa dos seguintes itens:

- Uma assinatura do Azure AD
- Uma Conta Sistema aprovada do Cerner Central

> [!NOTE]
> Para testar as etapas deste tutorial, nós não recomendamos o uso de um ambiente de produção.

Para testar as etapas deste tutorial, você deve seguir estas recomendações:

- Não use o ambiente de produção, a menos que seja necessário.
- Se não tiver um ambiente de avaliação do AD do Azure, você pode obter uma versão de avaliação de um mês [aqui](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Descrição do cenário
Neste tutorial, você testará o logon único do Azure AD em um ambiente de teste.  O cenário descrito neste tutorial consiste em dois blocos de construção principais:

1. Adicionar o Cerner Central da galeria
1. configurar e testar o logon único do AD do Azure

## <a name="adding-cerner-central-from-the-gallery"></a>Adicionar o Cerner Central da galeria
Para configurar a integração do Cerner Central ao Azure AD, você precisa adicionar o Cerner Central à sua lista de aplicativos SaaS gerenciados por meio da galeria.

**Para adicionar o Cerner Central da galeria, execute as seguintes etapas:**

1. No **[Portal do Azure](https://portal.azure.com)**, no painel navegação à esquerda, clique no ícone **Azure Active Directory**. 

    ![Active Directory][1]

1. Navegue até **aplicativos empresariais**. Em seguida, vá para **todos os aplicativos**.

    ![Aplicativos][2]

1. Para adicionar o novo aplicativo, clique no botão **Novo aplicativo** na parte superior da caixa de diálogo.

    ![Aplicativos][3]

1. Na caixa de pesquisa, digite **Cerner Central**.

    ![Criação de um usuário de teste do AD do Azure](./media/cernercentral-tutorial/tutorial_cernercentral_search.png)

1. No painel de resultados, selecione **Cerner Central** e clique no botão **Adicionar** para adicionar o aplicativo.

    ![Criação de um usuário de teste do AD do Azure](./media/cernercentral-tutorial/tutorial_cernercentral_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>configurar e testar o logon único do AD do Azure
Nesta seção, você configurará e testará o logon único do Azure AD com o Cerner Central, com base em um usuário de teste chamado "Brenda Fernandes".

Para que o logon único funcione, o Azure AD precisa saber qual usuário do Cerner Central é equivalente a um usuário do Azure AD. Em outras palavras, é necessário estabelecer uma relação de vínculo entre um usuário do Azure AD e o usuário relacionado do Cerner Central.

Para configurar e testar o logon único do Azure AD com o Cerner Central, você precisa concluir os seguintes blocos de construção:

1. **[Configuring Azure AD Single Sign-On](#configuring-azure-ad-single-sign-on)** - para habilitar seus usuários a usar esse recurso.
1. **[Criação de um usuário de teste do AD do Azure](#creating-an-azure-ad-test-user)** – para testar o logon único do AD do Azure com Brenda Fernandes.
1. **[Criação de um usuário de teste do Cerner Central](#creating-a-cerner-central-test-user)** – para ter um equivalente de Brenda Fernandes no Cerner Central que esteja vinculado à representação do usuário no Azure AD.
1. **[Atribuição do usuário de teste do AD do Azure](#assigning-the-azure-ad-test-user)** – para permitir que Brenda Fernandes use o logon único do AD do Azure.
1. **[Teste do logon único](#testing-single-sign-on)** : para verificar se a configuração funciona.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuração do logon único do Azure AD

Nesta seção, você habilitará o logon único do Azure AD no Portal do Azure e configurará o logon único em seu aplicativo Cerner Central.

**Para configurar o logon único do Azure AD com o Cerner Central, execute as seguintes etapas:**

1. No Portal do Azure, na página de integração do aplicativo **Cerner Central**, clique em **Logon único**.

    ![Configurar o logon único][4]

1. Na caixa de diálogo **Logon único**, selecione **Modo** como **Logon baseado em SAML** para habilitar o logon único.

    ![Configurar o logon único](./media/cernercentral-tutorial/tutorial_cernercentral_samlbase.png)

1. Na seção **URLs e Domínio do Cerner Central**, execute as seguintes etapas:

    ![Configurar o logon único](./media/cernercentral-tutorial/tutorial_cernercentral_url.png)

     a. Na caixa de texto **Identificador**, digite o valor usando os seguintes padrões:

    | |
    |--|
    | `https://<instancename>.cernercentral.com/session-api/protocol/saml2/metadata` |
    | `https://<instancename>.sandboxcernercentral.com/session-api/protocol/saml2/metadata` |
    
    b. Na caixa de texto **URL de Resposta** , digite uma URL nos seguintes padrões:
    
    | |
    |--|
    | `https://<instancename>.cernercentral.com/session-api/protocol/saml2/sso` |
    | `https://<instancename>.sandboxcernercentral.com/session-api/protocol/saml2/sso` |

    > [!NOTE]
    > Esses não são os valores reais. Atualize esses valores com o Identificador e a URL de Resposta reais. Entre em contato com a [equipe de suporte do Cerner Central](https://wiki.ucern.com/display/CernerCentral/Contacting+Cloud+Operations) para obter esses valores.

1. Na seção **Certificado de Autenticação SAML**, clique no botão copiar para copiar a **URL de metadados de federação do aplicativo** e cole-a no bloco de notas.

    ![Configurar o logon único](./media/cernercentral-tutorial/tutorial_metadataurl.png)

1. Clique no botão **Salvar** .

    ![Configurar o logon único](./media/cernercentral-tutorial/tutorial_general_400.png)

1. Para configurar logon único no **Cerner Central**, é necessário enviar a **URL de metadados de federação do aplicativo**  para o [suporte do Cerner Central](https://wiki.ucern.com/display/CernerCentral/Contacting+Cloud+Operations). Eles configuram o SSO no lado do aplicativo para concluir a integração.

### <a name="creating-an-azure-ad-test-user"></a>Criação de um usuário de teste do AD do Azure
O objetivo desta seção é criar um usuário de teste no Portal do Azure chamado Brenda Fernandes.

![Criar um usuário do AD do Azure][100]

**Para criar um usuário de teste no AD do Azure, execute as seguintes etapas:**

1. No **Portal do Azure**, no painel de navegação esquerdo, clique no ícone **Azure Active Directory**.

    ![Criação de um usuário de teste do AD do Azure](./media/cernercentral-tutorial/create_aaduser_01.png) 

1. Vá para **Usuários e grupos** e clique em **Todos os usuários** para exibir a lista de usuários.

    ![Criação de um usuário de teste do AD do Azure](./media/cernercentral-tutorial/create_aaduser_02.png) 

1. Para abrir a caixa de diálogo **Usuário**, clique em **Adicionar**.

    ![Criação de um usuário de teste do AD do Azure](./media/cernercentral-tutorial/create_aaduser_03.png) 

1. Na página do diálogo **Usuário**, execute as seguintes etapas:

    ![Criação de um usuário de teste do AD do Azure](./media/cernercentral-tutorial/create_aaduser_04.png) 

    a. Na caixa de texto **Nome**, digite **Brenda Fernandes**.

    b. Na caixa de texto **Nome de usuário**, digite o **endereço de email** da conta de Brenda Fernandes.

    c. Selecione **Mostrar senha** e anote o valor de **senha**.

    d. Clique em **Criar**.

### <a name="creating-a-cerner-central-test-user"></a>Criar um usuário de teste do Cerner Central

O aplicativo **Cerner Central** permite a autenticação por meio de qualquer provedor de identidade federada. Se um usuário conseguir fazer logon na home page do aplicativo, ele será federado e não precisará de nenhum provisionamento manual. Você pode encontrar mais detalhes [ aqui ](cernercentral-provisioning-tutorial.md) sobre como configurar o provisionamento automático de usuários.

### <a name="assigning-the-azure-ad-test-user"></a>Atribuição do usuário de teste do AD do Azure

Nesta seção, ao conceder a Brenda Fernandes acesso ao Cerner Central, você permite que ela use o logon único do Azure.

![Atribuir usuário][200]

**Para atribuir Brenda Fernandes ao Cerner Central, execute as seguintes etapas:**

1. No Portal do Azure, abra a exibição de aplicativos e, em seguida, navegue até a exibição de diretório e vá para **Aplicativos Empresariais** e clique em **Todos os aplicativos**.

    ![Atribuir usuário][201]

1. Na lista de aplicativos, selecione **Cerner Central**.

    ![Configurar o logon único](./media/cernercentral-tutorial/tutorial_cernercentral_app.png)

1. No menu à esquerda, clique em **usuários e grupos**.

    ![Atribuir usuário][202]

1. Clique no botão **Adicionar**. Em seguida, selecione **usuários e grupos** na **Adicionar atribuição** caixa de diálogo.

    ![Atribuir usuário][203]

1. Em **usuários e grupos** caixa de diálogo, selecione **Britta Simon** na lista de usuários.

1. Clique em **selecione** botão **usuários e grupos** caixa de diálogo.

1. Clique em **atribuir** botão **Adicionar atribuição** caixa de diálogo.

### <a name="testing-single-sign-on"></a>Teste do logon único

Nesta seção, você testará sua configuração de logon único do Azure AD usando o Painel de Acesso.

Ao clicar no bloco do Cerner Central no Painel de Acesso, você deverá ser conectado automaticamente a seu aplicativo do Cerner Central. Para saber mais sobre o Painel de Acesso, veja [Introdução ao Painel de Acesso](../user-help/active-directory-saas-access-panel-introduction.md).

## <a name="additional-resources"></a>Recursos adicionais

* [Lista de tutoriais sobre como integrar aplicativos SaaS com o Active Directory do Azure](tutorial-list.md)
* [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Configurar Provisionamento de Usuário](cernercentral-provisioning-tutorial.md)

<!--Image references-->

[1]: ./media/cernercentral-tutorial/tutorial_general_01.png
[2]: ./media/cernercentral-tutorial/tutorial_general_02.png
[3]: ./media/cernercentral-tutorial/tutorial_general_03.png
[4]: ./media/cernercentral-tutorial/tutorial_general_04.png

[100]: ./media/cernercentral-tutorial/tutorial_general_100.png

[200]: ./media/cernercentral-tutorial/tutorial_general_200.png
[201]: ./media/cernercentral-tutorial/tutorial_general_201.png
[202]: ./media/cernercentral-tutorial/tutorial_general_202.png
[203]: ./media/cernercentral-tutorial/tutorial_general_203.png
