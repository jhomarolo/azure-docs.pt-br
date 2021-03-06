---
title: 'Tutorial: Integração do Azure Active Directory ao Clarizen | Microsoft Docs'
description: Saiba como configurar o logon único entre o Active Directory do Azure e o Clarizen.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 28acce3e-22a0-4a37-8b66-6e518d777350
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/21/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 48955354baf86d84c0ddc6618a99f7e806c26c2b
ms.sourcegitcommit: 5839af386c5a2ad46aaaeb90a13065ef94e61e74
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/19/2019
ms.locfileid: "57840862"
---
# <a name="tutorial-azure-active-directory-integration-with-clarizen"></a>Tutorial: Integração do Active Directory do Azure ao Clarizen

Neste tutorial, você aprenderá a integrar o Clarizen ao Azure AD (Azure Active Directory).
A integração do Clarizen ao Azure AD oferece os seguintes benefícios:

* No Azure AD, é possível controlar quem tem acesso ao Clarizen.
* Você pode permitir que seus usuários entrem automaticamente no Clarizen (logon único) usando suas contas do Azure AD.
* Você pode gerenciar suas contas em um único local central – o portal do Azure.

Para conhecer mais detalhadamente a integração de aplicativos de SaaS ao AD do Azure, consulte [O que é o acesso a aplicativos e logon único com o Active Directory do Azure](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Se você não tiver uma assinatura do Azure, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.

## <a name="prerequisites"></a>Pré-requisitos

Para configurar a integração do Azure AD ao Clarizen, você precisará dos seguintes itens:

* Uma assinatura do Azure AD. Se não tiver um ambiente do Azure AD, você pode obter uma versão de avaliação de um mês [aqui](https://azure.microsoft.com/pricing/free-trial/)
* Uma assinatura do Clarizen habilitada para logon único

## <a name="scenario-description"></a>Descrição do cenário

Neste tutorial, você configurará e testará o logon único do Azure AD em um ambiente de teste.

* O Clarizen é compatível com o SSO iniciado por **IDP**

## <a name="adding-clarizen-from-the-gallery"></a>Adicionando o Clarizen da galeria

Para configurar a integração do Clarizen ao Azure AD, você precisará adicionar o Clarizen da galeria à sua lista de aplicativos SaaS gerenciados.

**Para adicionar o Clarizen da galeria, execute as seguintes etapas:**

1. No **[Portal do Azure](https://portal.azure.com)**, no painel navegação à esquerda, clique no ícone **Azure Active Directory**.

    ![O botão Azure Active Directory](common/select-azuread.png)

2. Navegue até **Aplicativos Empresariais** e, em seguida, selecione a opção **Todos os Aplicativos**.

    ![A folha Aplicativos empresariais](common/enterprise-applications.png)

3. Clique no botão **Novo aplicativo** na parte superior da caixa de diálogo para adicionar o novo aplicativo.

    ![O botão Novo aplicativo](common/add-new-app.png)

4. Na caixa de pesquisa, digite **Clarizen**, selecione **Clarizen** no painel de resultados e, em seguida, clique no botão **Adicionar** para adicionar o aplicativo.

     ![Clarizen na lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurar e testar logon único do Azure AD

Nesta seção, você configurará e testará o logon único do Azure AD com o Clarizen, com base em um usuário de teste chamado **Brenda Fernandes**.
Para que o logon único funcione, é necessário estabelecer uma relação de vínculo entre um usuário do Azure AD e o usuário relacionado do Clarizen.

Para configurar e testar o logon único do Azure AD com o Clarizen, você precisa concluir os seguintes blocos de construção:

1. **[Configurar o logon único do Azure AD](#configure-azure-ad-single-sign-on)** – para habilitar seus usuários a usar esse recurso.
2. **[Configurar o logon único do Clarizen](#configure-clarizen-single-sign-on)** – para definir as configurações de logon único no lado do aplicativo.
3. **[Criar um usuário de teste do Azure AD](#create-an-azure-ad-test-user)** – para testar o logon único do Azure AD com Brenda Fernandes.
4. **[Atribuir o usuário de teste do Azure AD](#assign-the-azure-ad-test-user)** – para permitir que Brenda Fernandes use o logon único do Azure AD.
5. **[Criar um usuário de teste do Clarizen](#create-clarizen-test-user)** – para ter um equivalente de Brenda Fernandes no Clarizen que esteja vinculado à representação do usuário no Azure AD.
6. **[Teste o logon único](#test-single-sign-on)** – para verificar se a configuração funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configurar o logon único do Azure AD

Nesta seção, você habilitará o logon único do Azure AD no portal do Azure.

Para configurar o logon único do Azure AD com o Clarizen, execute as seguintes etapas:

1. No [portal do Azure](https://portal.azure.com/), na página de integração do aplicativo **Clarizen**, clique em **Logon único**.

    ![Link Configurar logon único](common/select-sso.png)

2. Na caixa de diálogo **Selecionar um método de logon único**, selecione o modo **SAML/WS-Fed** para habilitar o logon único.

    ![Modo de seleção de logon único](common/select-saml-option.png)

3. Na página **Definir logon único com SAML**, clique no ícone **Editar** para abrir a caixa de diálogo **Configuração básica do SAML**.

    ![Editar a Configuração Básica de SAML](common/edit-urls.png)

4. Na página **Configurar Logon Único com SAML**, execute as seguintes etapas:

    ![Informações de logon único de Domínio e URLs do Clarizen](common/idp-intiated.png)

     a. Na caixa de texto **Identificador**, digite um valor: `Clarizen`

    b. No **URL de resposta** caixa de texto, digite uma URL usando o seguinte padrão: `https://.clarizen.com/Clarizen/Pages/Integrations/SAML/SamlResponse.aspx`

    > [!NOTE]
    > Esses não são os valores reais. Você precisa usar o identificador real e a URL de resposta. Aqui, sugerimos que você use o valor exclusivo de uma cadeia de caracteres como o identificador. Para obter os valores reais, entre em contato com a [equipe de suporte do Clarizen](https://success.clarizen.com/hc/en-us/requests/new).

4. Na página **Configurar logon único com SAML**, na seção **Certificado de Autenticação SAML**, clique em **Fazer o download** para fazer o download do **Certificado (Base64)** usando as opções fornecidas de acordo com seus requisitos e salve-o no computador.

    ![O link de download do Certificado](common/certificatebase64.png)

6. Na seção **Configurar o Clarizen**, copie a URL apropriada de acordo com suas necessidades.

    ![Copiar URLs de configuração](common/copy-configuration-urls.png)

    a. URL de logon

    b. Identificador do Azure Ad

    c. URL de logoff

### <a name="configure-clarizen-single-sign-on"></a>Configurar o logon único do Clarizen

1. Em outra janela do navegador da Web, entre em seu site de empresa do Clarizen como administrador.

1. Clique no nome de usuário e em **Configurações**.

    ![Clicar em "Configurações" em seu nome de usuário](./media/clarizen-tutorial/tutorial_clarizen_001.png "Configurações")

1. Clique na guia **Configurações Globais**. Em seguida, próximo a **Autenticação Federada**, clique em **editar**.

    ![Guia "Configurações Globais"](./media/clarizen-tutorial/tutorial_clarizen_002.png "Configurações Globais")

1. Na caixa de diálogo **Autenticação Federada**, execute as seguintes etapas:

    ![Caixa de diálogo "Autenticação Federada"](./media/clarizen-tutorial/tutorial_clarizen_003.png "Autenticação Federada")

     a. Selecione **Habilitar Autenticação Federada**.

    b. Clique em **Carregar** para carregar o certificado baixado.

    c. Na caixa de texto **URL de Entrada**, insira o valor da **URL de Logon** da janela de configuração de aplicativo do Azure AD.

    d. Na caixa de texto **URL de Entrada**, insira o valor da **URL de Logon** da janela de configuração de aplicativo do Azure AD.

    e. Selecione **Usar POST**.

    f. Clique em **Salvar**.

### <a name="create-an-azure-ad-test-user"></a>Criar um usuário de teste do Azure AD 

O objetivo desta seção é criar um usuário de teste no Portal do Azure chamado Brenda Fernandes.

1. No Portal do Azure, no painel esquerdo, selecione **Azure Active Directory**, selecione **Usuários** e, em seguida, **Todos os usuários**.

    ![Os links “Usuários e grupos” e “Todos os usuários”](common/users.png)

2. Selecione **Novo usuário** na parte superior da tela.

    ![Botão Novo usuário](common/new-user.png)

3. Nas Propriedades do usuário, execute as etapas a seguir.

    ![A caixa de diálogo Usuário](common/user-properties.png)

    a. No campo **Nome**, insira **BrendaFernandes**.
  
    b. No campo **Nome de usuário**, digite **brendafernandes\@domíniodaempresa.extensão**  
    Por exemplo, BrittaSimon@contoso.com

    c. Marque a caixa de seleção **Mostrar senha** e, em seguida, anote o valor exibido na caixa Senha.

    d. Clique em **Criar**.

### <a name="assign-the-azure-ad-test-user"></a>Atribuir o usuário de teste do Azure AD

Nesta seção, você permitirá que Brenda Fernandes use o logon único do Azure, concedendo a ela acesso ao Clarizen.

1. No portal do Azure, selecione **Aplicativos Empresariais**, **Todos os aplicativos** e, em seguida, selecione **Clarizen**.

    ![Folha de aplicativos empresariais](common/enterprise-applications.png)

2. Na lista de aplicativos, selecione **Clarizen**.

    ![O link do Clarizen na lista de Aplicativos](common/all-applications.png)

3. No menu à esquerda, selecione **Usuários e grupos**.

    ![O link “Usuários e grupos”](common/users-groups-blade.png)

4. Escolha o botão **Adicionar usuário** e, em seguida, escolha **Usuários e grupos** na caixa de diálogo **Adicionar Atribuição**.

    ![O painel Adicionar Atribuição](common/add-assign-user.png)

5. Na caixa de diálogo **Usuários e grupos**, escolha **Brenda Fernandes** na lista Usuários e clique no botão **Selecionar** na parte inferior da tela.

6. Se você estiver esperando um valor de função na declaração SAML, na caixa de diálogo **Selecionar função**, escolha a função de usuário apropriada na lista e clique no botão **Selecionar** na parte inferior da tela.

7. Na caixa de diálogo **Adicionar atribuição**, clique no botão **Atribuir**.

### <a name="create-clarizen-test-user"></a>Criar um usuário de teste do Clarizen

O objetivo desta seção é criar um usuário chamado Brenda Fernandes no Clarizen.

**Se você precisar criar um usuário manualmente, siga estas etapas:**

Para permitir que os usuários do Azure AD entrem no Clarizen, você deverá provisionar contas de usuário. No caso do Clarizen, o provisionamento é uma tarefa manual.

1. Entre em seu site de empresa do Clarizen como administrador.

2. Clique em **Pessoas**.

    ![Clicar em "Pessoas"](./media/clarizen-tutorial/create_aaduser_001.png "Pessoas")

3. Clique em **Convidar Usuário**.

    ![Botão "Convidar Usuário"](./media/clarizen-tutorial/create_aaduser_002.png "Convidar Usuários")

1. Na página **Convidar Pessoas**, execute as seguintes etapas:

    ![Caixa de diálogo "Convidar Pessoas"](./media/clarizen-tutorial/create_aaduser_003.png "Convidar Pessoas")

     a. Na caixa **Email**, digite o endereço de email da conta de Brenda Fernandes.

    b. Clique em **Convidar**.

    > [!NOTE]
    > O titular da conta do Active Directory do Azure receberá um email e seguirá um link para confirmar a conta antes que ela se torne ativa.


### <a name="test-single-sign-on"></a>Testar logon único 

Nesta seção, você testará sua configuração de logon único do Azure AD usando o Painel de Acesso.

Ao clicar no bloco do Clarizen no Painel de Acesso, você deverá ser conectado automaticamente ao Clarizen, no qual configurou o SSO. Para saber mais sobre o Painel de Acesso, veja [Introdução ao Painel de Acesso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionais

- [Lista de tutoriais sobre como integrar aplicativos SaaS com o Active Directory do Azure](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [O que é o acesso condicional no Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
