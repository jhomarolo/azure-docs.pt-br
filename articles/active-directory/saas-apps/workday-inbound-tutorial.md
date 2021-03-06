---
title: 'Tutorial: Configurar o Workday para provisionamento automático do usuário com Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar o Azure Active Directory para provisionar e descontinuar automaticamente as contas de usuário para o Workday.
services: active-directory
author: cmmdesai
documentationcenter: na
manager: daveba
ms.assetid: 1a2c375a-1bb1-4a61-8115-5a69972c6ad6
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/19/2019
ms.author: chmutali
ms.collection: M365-identity-device-management
ms.openlocfilehash: d34bd9d7f80f72b3c6c0821ad48e6be1fd260be9
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60385061"
---
# <a name="tutorial-configure-workday-for-automatic-user-provisioning"></a>Tutorial: Configurar o Workday para provisionamento automático do usuário

O objetivo deste tutorial é mostrar as etapas necessárias para importar perfis de trabalho do Workday para o Active Directory e o Azure Active Directory, com write-back opcional de alguns endereços de email para o Workday.

## <a name="overview"></a>Visão geral

O [serviço de provisionamento de usuário do Azure Active Directory](../manage-apps/user-provisioning.md) integra-se com a [API de Recursos Humanos do Workday](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) para provisionar contas de usuários. O Azure AD utiliza essa conexão para habilitar os seguintes fluxos de trabalho de provisionamento de usuário:

* **Provisionamento de usuários para o Active Directory** - provisionar conjuntos de usuários selecionados do Workday em um ou mais domínios do Active Directory.

* **Provisionamento de usuários somente na nuvem para o Azure Active Directory** - Nos cenários em que o Active Directory local não for usado, os usuários poderão ser provisionados diretamente do Workday para o Azure Active Directory usando o serviço de provisionamento de usuário do Microsoft Azure AD.

* **Fazer write-back de endereços de email para Workday** - o serviço de provisionamento de usuário do Microsoft Azure AD pode gravar os endereços de email dos usuários do Microsoft Azure AD de volta no Workday.

### <a name="what-human-resources-scenarios-does-it-cover"></a>Quais cenários de recursos humanos ele cobre?

Os fluxos de trabalho de provisionamento de usuário do Workday com suporte do serviço de provisionamento de usuário do Azure AD habilitam a automação dos seguintes cenários de gerenciamento do ciclo de vida de identidade e recurso humanos:

* **Contratação de novos funcionários** - quando um novo funcionário é adicionado ao Workday, uma conta de usuário é criada automaticamente no Active Directory, no Azure Active Directory e, opcionalmente, no Office 365 e [ outros aplicativos SaaS suportados pelo Azure AD](../manage-apps/user-provisioning.md), com write-back do endereço de e-mail para Workday.

* **Atualizações de perfil e atributo de funcionário** - Quando um registro de funcionário é atualizado no Workday (como seu nome, cargo ou gerente), sua conta de usuário será atualizada automaticamente no Active Directory, no Azure Active Directory e, opcionalmente, no Office 365 e [outros aplicativos SaaS com suporte do Azure AD](../manage-apps/user-provisioning.md).

* **Rescisão de funcionário** - Quando um funcionário é rescindido no Workday, sua conta de usuário será desabilitada automaticamente no Active Directory, no Azure Active Directory e, opcionalmente, no Office 365 e [outros aplicativos SaaS com suporte do Azure AD](../manage-apps/user-provisioning.md).

* **Recontratação de funcionário** - quando um funcionário é recontratado no Workday, sua conta antiga poderá ser reativada ou reprovisionada automaticamente (dependendo da sua preferência) no Active Directory, Azure Active Directory e, opcionalmente, Office 365 e [outros aplicativos SaaS com suporte do Azure AD](../manage-apps/user-provisioning.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>Para quem é mais recomendada essa solução de provisionamento de usuário?

Essa solução de provisionamento de usuário do Workday é ideal para:

* Organizações que desejam uma solução predefinida, baseados em nuvem para o provisionamento de usuário do Workday

* Organizações que precisam de provisionamento de usuário direto do Workday para o Active Directory ou Azure Active Directory

* Organizações que precisam que os usuários sejam provisionados usando os dados obtidos do módulo HCM do Workday (consulte [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html))

* As organizações que precisam que os usuários adicionados, transferidos e excluídos sejam sincronizados com um ou mais florestas, domínios e unidades organizacionais do Active Directory com base somente em alterar informações detectadas no módulo HCM Workday (consulte [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html))

* Organizações que usam o Office 365 para email

## <a name="solution-architecture"></a>Arquitetura da solução

Esta seção descreve a arquitetura da solução de provisionamento do usuário de ponta a ponta para ambientes híbridos comuns. Há dois fluxos relacionados:

* **Fluxo de Dados de RH Autoritativo – do Workday para o Active Directory local:** Nesse fluxo, os eventos de trabalho (como Novas Contratações, Transferências, Demissões) ocorrem primeiro no locatário de RH do Workday na nuvem e, em seguida, os dados do evento fluem para o Active Directory local por meio do Azure AD e do Agente de Provisionamento. Dependendo do evento, pode levar a operações de criar/atualizar/habilitar/desabilitar no AD.
* **Fluxo de Write-Back de Email – do Active Directory local para o Workday:** Depois que a criação da conta for concluída no Active Directory, ela será sincronizada com o Azure AD por meio do Azure AD Connect e o atributo de email originado do Active Directory poderá ser gravado novamente no Workday.

![Visão geral](./media/workday-inbound-tutorial/wd_overview.png)

### <a name="end-to-end-user-data-flow"></a>Fluxo de dados do usuário de ponta a ponta

1. A equipe de RH realiza transações de trabalho (acréscimos/movimentações/saídas ou novas contratações/transferências/desligamentos) no HCM do Workday
2. O Serviço de Provisionamento do Azure AD executa sincronizações agendadas de identidades de RH do Workday e identifica alterações que precisam ser processadas para sincronização com o Active Directory local.
3. O Serviço de Provisionamento do Azure AD invoca o Agente de Provisionamento do AAD Connect local com um conteúdo de solicitação que contém operações de criar/atualizar/habilitar/desabilitar contas do AD.
4. O Agente de Provisionamento do Azure AD Connect usa uma conta de serviço para adicionar/atualizar dados de conta do AD.
5. O mecanismo do Azure AD Connect/AD Sync executa uma sincronização delta para efetuar pull das atualizações no AD.
6. As atualizações do Active Directory são sincronizadas com o Azure Active Directory.
7. Se o conector de Write-back do Workday estiver configurado, ele fará write-back do atributo de email para o Workday com base no atributo correspondente usado.

## <a name="planning-your-deployment"></a>Planejamento da implantação

Antes de iniciar sua integração no Workday, verifique os pré-requisitos abaixo e leia as seguintes orientações sobre como corresponder sua arquitetura do Active Directory atual e os requisitos de provisionamento de usuário com a(s) solução(ões) fornecida(s) pelo Azure Active Directory. Um [plano de implantação](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-deployment-plans) abrangente com planilhas de planejamento também está disponível para ajudar você a colaborar com seu parceiro de integração do Workday e stakeholders de RH.

Esta seção cobre os seguintes aspectos de planejamento:

* [Pré-requisitos](#prerequisites)
* [Selecionar aplicativos de conectores de provisionamento para implantação](#selecting-provisioning-connector-apps-to-deploy)
* [Planejar a implantação do Agente de Provisionamento do Azure AD Connect](#planning-deployment-of-azure-ad-connect-provisioning-agent)
* [Integração a vários domínios do Active Directory](#integrating-with-multiple-active-directory-domains)
* [Planejar mapeamento e transformações de atributos do usuário do Workday para o Active Directory](#planning-workday-to-active-directory-user-attribute-mapping-and-transformations)

### <a name="prerequisites"></a>Pré-requisitos

O cenário descrito neste tutorial pressupõe que você já tem os seguintes itens:

* Uma assinatura do Azure AD Premium P1 ou superior válida com acesso de administrador global
* Um locatário de implementação do Workday para fins de integração e teste
* Permissões de administrador no Workday para criar um usuário de integração de sistema e fazer alterações para testar dados do funcionário para fins de teste
* Para provisionamento do usuário no Active Directory, é necessário um servidor executando o Windows Server 2012 ou superior com tempo de execução .NET 4.7.1+ para hospedar o [agente de provisionamento local](https://go.microsoft.com/fwlink/?linkid=847801)
* [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) para a sincronização de usuários entre o Active Directory e o Azure AD

### <a name="selecting-provisioning-connector-apps-to-deploy"></a>Selecionar aplicativos de conectores de provisionamento para implantação

Para facilitar os fluxos de trabalho de provisionamento entre o Workday e o Active Directory, o Azure AD fornece vários aplicativos de conectores de provisionamento que podem ser adicionados na galeria de aplicativos do Azure AD:

![Galeria de Aplicativos do AAD](./media/workday-inbound-tutorial/wd_gallery.png)

* **Provisionamento de Usuário do Workday para o Active Directory** - esse aplicativo facilita o provisionamento de contas de usuário do Workday para um único domínio do Active Directory. Se você tiver vários domínios, será possível adicionar uma instância desse aplicativo da galeria de aplicativos do Azure AD para cada domínio do Active Directory que precisar ser provisionado.

* **Provisionamento de Usuário do Workday para o Azure AD** - embora o AAD Connect seja a ferramenta que deve ser utilizada para sincronizar usuários do Active Directory ao Azure Active Directory, esse aplicativo pode ser utilizado para facilitar o provisionamento de usuários somente na nuvem do Workday para um locatário único do Azure Active Directory.

* **Write-back do Workday** – este aplicativo facilita o write-back de endereços de email do usuário do Azure Active Directory para o Workday.

> [!TIP]
> O aplicativo "Workday" regular é utilizado para configurar logon único entre o Workday e o Azure Active Directory.

Use o fluxograma de decisão abaixo para identificar quais aplicativos de provisionamento do Workday são relevantes para seu cenário.
    ![Fluxograma de decisão](./media/workday-inbound-tutorial/wday_app_flowchart.png "Fluxograma de decisão")

Use o sumário para ir até a seção relevante deste tutorial.

### <a name="planning-deployment-of-azure-ad-connect-provisioning-agent"></a>Planejar a implantação do Agente de Provisionamento do Azure AD Connect

> [!NOTE]
> Esta seção é relevante apenas se você planeja implantar o aplicativo de Provisionamento de Usuário do Workday para o Active Directory. Você pode ignorá-la se estiver implantando o write-back de Workday ou o aplicativo de Provisionamento de Usuário do Workday para o Azure AD.

A solução de Provisionamento de Usuário do Workday para AD exige a implantação de um ou mais Agentes de Provisionamento em servidores que executam o Windows 2012 R2 ou superior com no mínimo de 4 GB de RAM e o tempo de execução do .NET 4.7.1+. As considerações a seguir devem ser levadas em conta antes de instalar o Agente de Provisionamento:

* Certifique-se de que o servidor de host que executa o Agente de Provisionamento tenha acesso à rede do domínio de destino do AD
* O Assistente de Configuração do Agente de Provisionamento registra o agente em seu locatário do Azure AD e o processo de registro requer acesso a *. msappproxy.net na porta SSL 443. Certifique-se de que as regras de firewall de saída em vigor permitam essa comunicação. O agente é compatível com a [configuração de proxy de saída HTTPS](#how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication).
* O Agente de Provisionamento usa uma conta de serviço para se comunicar com os domínios do AD locais. Antes da instalação do agente, é recomendável que você crie uma conta de serviço com permissões de administrador de domínio e uma senha que não expire.  
* Durante a configuração do Agente de Provisionamento, você pode selecionar os controladores de domínio que devem lidar com as solicitações de provisionamento. Se você tiver vários controladores de domínio distribuídos geograficamente, instale o Agente de Provisionamento no mesmo site que seus controladores de domínio preferidos para aumentar a confiabilidade e o desempenho da solução de ponta a ponta
* Para obter alta disponibilidade, você pode implantar mais de um Agente de Provisionamento e registrá-lo para lidar com o mesmo conjunto de domínios do AD locais.

> [!IMPORTANT]
> Em ambientes de produção, a Microsoft recomenda que você tenha no mínimo três Agentes de Provisionamento configurados com seu locatário do Azure AD para alta disponibilidade.

### <a name="integrating-with-multiple-active-directory-domains"></a>Integração a vários domínios do Active Directory

> [!NOTE]
> Esta seção é relevante apenas se você planeja implantar o aplicativo de Provisionamento de Usuário do Workday para o Active Directory. Você pode ignorá-la se estiver implantando o write-back de Workday ou o aplicativo de Provisionamento de Usuário do Workday para o Azure AD.

Dependendo da sua topologia do Active Directory, você precisará decidir o número de aplicativos conectores de Provisionamento de Usuário e o número de Agentes de Provisionamento a configurar. Abaixo estão alguns dos padrões comuns de implantação que você pode utilizar ao planejar sua implantação.

#### <a name="deployment-scenario-1--single-workday-tenant---single-ad-domain"></a>Cenário de implantação 1: Locatário individual do Workday -> Domínio individual do AD

Nesse cenário, você tem um locatário do Workday e deseja provisionar usuários para um único domínio do AD de destino. Aqui está a configuração de produção recomendada para essa implantação.

|   |   |
| - | - |
| Não. de agentes de provisionamento a serem implantados localmente | 3 (para alta disponibilidade e failover) |
| Não. de aplicativos de Provisionamento de Usuário do Workday para o AD para configurar no portal do Azure | 1 |

  ![Cenário 1](./media/workday-inbound-tutorial/dep_scenario1.png)

#### <a name="deployment-scenario-2--single-workday-tenant---multiple-child-ad-domains"></a>Cenário de implantação 2: Locatário individual do Workday -> Vários domínios filhos do AD

Este cenário envolve o provisionamento de usuários do Workday para vários domínios filhos de destino do AD em uma floresta. Aqui está a configuração de produção recomendada para essa implantação.

|   |   |
| - | - |
| Não. de agentes de provisionamento a serem implantados localmente | 3 (para alta disponibilidade e failover) |
| Não. de aplicativos de Provisionamento de Usuário do Workday para o AD para configurar no portal do Azure | um aplicativo por domínio filho |

  ![Cenário 2](./media/workday-inbound-tutorial/dep_scenario2.png)

#### <a name="deployment-scenario-3--single-workday-tenant---disjoint-ad-forests"></a>Cenário de implantação 3: Locatário único do Workday -> Florestas do AD não contíguas

Este cenário envolve o provisionamento de usuários do Workday para domínios em florestas não contíguas do AD. Aqui está a configuração de produção recomendada para essa implantação.

|   |   |
| - | - |
| Não. de agentes de provisionamento a serem implantados localmente | 3 por floresta do AD não contígua |
| Não. de aplicativos de Provisionamento de Usuário do Workday para o AD para configurar no portal do Azure | um aplicativo por domínio filho |

  ![Cenário 3](./media/workday-inbound-tutorial/dep_scenario3.png)

### <a name="planning-workday-to-active-directory-user-attribute-mapping-and-transformations"></a>Planejar mapeamento e transformações de atributos do usuário do Workday para o Active Directory

> [!NOTE]
> Esta seção é relevante apenas se você planeja implantar o aplicativo de Provisionamento de Usuário do Workday para o Active Directory. Você pode ignorá-la se estiver implantando o write-back de Workday ou o aplicativo de Provisionamento de Usuário do Workday para o Azure AD.

Antes de configurar o provisionamento de usuário para um domínio do Active Directory, considere as seguintes perguntas. As respostas a essas perguntas determinarão como seus filtros de escopo e os mapeamentos de atributo precisam ser definidos.

* **O que os usuários no Workday precisam para serem provisionados a essa floresta do Active Directory?**

  * *Exemplo: Usuários em que o atributo "Empresa" do Workday contém o valor "Contoso" e o atributo "Worker_Type" contém "Regular"*

* **Como os usuários são roteados em diferentes OUs (unidades organizacionais)?**

  * *Exemplo: Os usuários são roteados para UOs que correspondem a um escritório, conforme definido nos atributos "Município" e "Country_Region_Reference" do Workday*

* **Como os atributos a seguir devem ser preenchidos no Active Directory?**

  * Nome Comum (cn)
    * *Exemplo: Use o valor User_ID do Workday, conforme definido pelos recursos humanos*

  * ID de funcionário (employeeId)
    * *Exemplo: Use o valor Worker_ID do Workday*

  * Nome de conta SAM (sAMAccountName)
    * *Exemplo: Use o valor User_ID do Workday, filtrado por uma expressão de provisionamento do Azure AD para remover caracteres inválidos*

  * Nome principal do usuário (userPrincipalName)
    * *Exemplo: Use o valor User_ID do Workday, com uma expressão de provisionamento do Azure AD para acrescentar um nome de domínio*

* **Como os usuários devem corresponder entre o Workday e o Active Directory?**

  * *Exemplo: Os usuários com um valor "Worker_ID" específico do Workday correspondem com usuários do Active Directory em que "employeeID" tem o mesmo valor. Se o valor de Worker_ID não for encontrado no Active Directory, crie um novo usuário.*
  
* **A floresta do Active Directory já contém as IDs de usuário necessárias para a lógica de correspondência funcionar?**

  * *Exemplo: Se essa configuração for uma nova implantação do Workday, é recomendável que o Active Directory seja preenchido previamente com os valores de Worker_ID do Workday corretos (ou valor de ID exclusivo de escolha) para manter a lógica de correspondência o mais simples possível.*

Como instalar e configurar esses aplicativos de conectores de provisionamento especiais é o assunto das próximas seções deste tutorial. Os aplicativos que você optar por configurar dependerão de quais sistemas precisam ser provisionados e de quantos locatários dos domínios do Active Directory e do Azure AD estão em seu ambiente.

## <a name="configure-integration-system-user-in-workday"></a>Configurar o usuário do sistema de integração no Workday

Um requisito comum de todos os conectores de provisionamento do Workday é que eles exigem credenciais de um usuário do sistema de integração do Workday para conectar-se à API de Recursos Humanos do Workday. Esta seção descreve como criar um usuário do sistema de integração no Workday e possui as seguintes seções:

* [Criar um usuário do sistema de integração](#creating-an-integration-system-user)
* [Criar um grupo de segurança de integração](#creating-an-integration-security-group)
* [Configurar permissões de política de segurança do domínio](#configuring-domain-security-policy-permissions)
* [Configurar permissões da política de segurança de processo empresarial](#configuring-business-process-security-policy-permissions)
* [Ativar alterações de política de segurança](#activating-security-policy-changes)

> [!NOTE]
> É possível ignorar esse procedimento e utilizar uma conta Administrador global do Workday como a conta de integração do sistema. Isso pode funcionar aparentemente bem para demonstrações, mas não é recomendável para implementações de produção.

### <a name="creating-an-integration-system-user"></a>Criação de um usuário do sistema de integração

**Para criar um usuário do sistema de integração:**

1. Entre no locatário do Workday utilizando uma conta Administrador. No **Aplicativo Workday**, insira “criar usuário” na caixa de pesquisa e clique em **Criar Usuário do Sistema de Integração**.

    ![Criar Usuário](./media/workday-inbound-tutorial/wd_isu_01.png "Criar Usuário")
2. Conclua a tarefa **Criar Usuário do Sistema de Integração** fornecendo um nome de usuário e senha para um novo Usuário do Sistema de Integração.  
  
* Deixe a opção **Exigir Nova Senha na Próxima Entrada** desmarcada, porque esse usuário efetuará logon por meio de programação.
* Deixe **Minutos de Tempo Limite da Sessão** com seu valor padrão de 0, o que impedirá que as sessões do usuário expirem prematuramente.
* Selecione a opção **Não Permitir Sessões de Interface do Usuário**, pois ela fornece uma camada adicional de segurança que impede que usuários com a senha do sistema de integração façam logon no Workday.

    ![Criar Usuário do Sistema de Integração](./media/workday-inbound-tutorial/wd_isu_02.png "Criar Usuário do Sistema de Integração")

### <a name="creating-an-integration-security-group"></a>Criar um grupo de segurança de integração

Nesta etapa, você criará um grupo de segurança do sistema de integração irrestrito ou restrito no Workday e atribuirá o usuário do sistema de integração criado na etapa anterior a esse grupo.

**Para criar um grupo de segurança:**

1. Insira Criar grupo de segurança na caixa de pesquisa e clique em **Criar Grupo de Segurança**.

    ![Criar Grupo de Segurança](./media/workday-inbound-tutorial/wd_isu_03.png "Criar Grupo de Segurança")
2. Conclua a tarefa **Criar Grupo de Segurança**. 

   * Há dois tipos de grupos de segurança no Workday:
     * **Irrestrito:** Todos os membros do grupo de segurança podem acessar todas as instâncias de dados protegidas pelo grupo de segurança.
     * **Restrito:** Todos os membros do grupo de segurança têm acesso contextual a um subconjunto das instâncias de dados (linhas) que o grupo de segurança pode acessar.
   * Entre em contato com seu parceiro de integração do Workday para selecionar o tipo de grupo de segurança adequado para a integração.
   * Após saber o tipo de grupo, selecione **Grupo de Segurança do Sistema de Integração (irrestrito)** ou **Grupo de Segurança do Sistema de Integração (restrito)** no menu suspenso **Tipo de Grupo de Segurança com Locatário**.

     ![Criar Grupo de Segurança](./media/workday-inbound-tutorial/wd_isu_04.png "Criar Grupo de Segurança")

3. Após a criação bem-sucedida do Grupo de segurança, você verá uma página na qual pode atribuir membros ao Grupo de segurança. Adicione o novo usuário do sistema de integração criado na etapa anterior a esse grupo de segurança. Se você estiver usando o grupo de segurança *restrito*, você também precisará selecionar o escopo da organização apropriado.

    ![Editar Grupo de Segurança](./media/workday-inbound-tutorial/wd_isu_05.png "Editar Grupo de Segurança")

### <a name="configuring-domain-security-policy-permissions"></a>Configurar permissões de política de segurança do domínio

Nesta etapa, você concederá permissões de política de “segurança de domínio” para os dados de trabalho ao grupo de segurança.

**Para configurar permissões de política de segurança do domínio:**

1. Insira a **Configuração de Segurança de Domínio** na caixa de pesquisa e, em seguida, clique no link **Relatório de Configuração de Segurança de Domínio**.  

    ![Políticas de Segurança de Domínio](./media/workday-inbound-tutorial/wd_isu_06.png "Políticas de Segurança de Domínio")  
2. Na caixa de texto **Domínio**, pesquise pelos seguintes domínios e adicione-os ao filtro individualmente.  
   * *Provisionamento de conta externa*
   * *Dados do Trabalhador: Relatórios do Trabalhador Público*
   * *Dados Pessoais: Informações de Contato Comerciais*
   * *Dados do Trabalhador: Todos os Cargos*
   * *Dados do Trabalhador: Informação da Equipe Atual*
   * *Dados do Trabalhador: Título Comercial no Perfil do Trabalhador*

     ![Políticas de Segurança de Domínio](./media/workday-inbound-tutorial/wd_isu_07.png "Políticas de Segurança de Domínio")  

     ![Políticas de Segurança de Domínio](./media/workday-inbound-tutorial/wd_isu_08.png "Políticas de Segurança de Domínio") 

     Clique em **OK**.

3. No relatório que aparece, selecione as reticências (...) que aparecem ao lado de **Provisionamento de Conta Externa** e clique na opção de menu **Domínio -> Editar Permissões da Política de Segurança**

    ![Políticas de Segurança de Domínio](./media/workday-inbound-tutorial/wd_isu_09.png "Políticas de Segurança de Domínio")  

4. Na página **Editar Permissões da Política de Segurança de Domínio**, role para baixo até a seção **Permissões de Integração**. Clique no sinal "+" para adicionar o grupo do sistema de integração à lista de grupos de segurança com as permissões de integração **Get** e **Put**.

    ![Editar Permissão](./media/workday-inbound-tutorial/wd_isu_10.png "Editar Permissão")  

5. Clique no sinal "+" para adicionar o grupo do sistema de integração à lista de grupos de segurança com as permissões de integração **Get** e **Put**.

    ![Editar Permissão](./media/workday-inbound-tutorial/wd_isu_11.png "Editar Permissão")  

6. Repita as etapas 3 a 5 acima para cada uma dessas políticas de segurança restantes:

   | Operação | Política de segurança de domínio |
   | ---------- | ---------- |
   | Get e Put | Dados do Trabalhador: Relatórios do Trabalhador Público |
   | Get e Put | Dados Pessoais: Informações de Contato Comerciais |
   | Obter | Dados do Trabalhador: Todos os Cargos |
   | Obter | Dados do Trabalhador: Informação da Equipe Atual |
   | Obter | Dados do Trabalhador: Título Comercial no Perfil do Trabalhador |

### <a name="configuring-business-process-security-policy-permissions"></a>Configurar permissões da política de segurança de processo empresarial

Nesta etapa, você concederá permissões de política de “segurança de processo empresarial” para os dados de trabalho ao grupo de segurança. Essa etapa é necessária para configurar o conector do aplicativo de write-back do Workday.

**Para configurar permissões da política de segurança de processo empresarial:**

1. Insira **Política de processo empresarial** na caixa de pesquisa e, em seguida, clique no link da tarefa **Editar Política de Segurança de Processo Empresarial**.  

    ![Políticas de Segurança de Processo Empresarial](./media/workday-inbound-tutorial/wd_isu_12.png "Políticas de Segurança de Processo Empresarial")  

2. Na caixa de texto **Tipo de Processo Empresarial**, pesquise por *Contato*, selecione o processo empresarial **Mudar contato** e clique em **OK**.

    ![Políticas de Segurança de Processo Empresarial](./media/workday-inbound-tutorial/wd_isu_13.png "Políticas de Segurança de Processo Empresarial")  

3. Na página **Editar Política de Segurança de Processo Empresarial**, role até a seção **Manter informações de contato (serviço Web)**.

    ![Políticas de Segurança de Processo Empresarial](./media/workday-inbound-tutorial/wd_isu_14.png "Políticas de Segurança de Processo Empresarial")  

4. Selecione e adicione o novo grupo de segurança do sistema de integração à lista de grupos de segurança que podem iniciar a solicitação de serviços Web. Clique em **Concluído**. 

    ![Políticas de Segurança de Processo Empresarial](./media/workday-inbound-tutorial/wd_isu_15.png "Políticas de Segurança de Processo Empresarial")  

### <a name="activating-security-policy-changes"></a>Ativando alterações de política de segurança

**Para ativar alterações de política de segurança:**

1. Digite Ativar na caixa de pesquisa e, em seguida, clique no link **Ativar alterações de política de segurança pendentes**.

    ![Ativar](./media/workday-inbound-tutorial/wd_isu_16.png "Ativar")

1. Inicie a tarefa Ativar Alterações de Política de Segurança Pendentes inserindo um comentário para fins de auditoria e clique em **OK**.
1. Conclua a tarefa na próxima tela marcando a caixa de seleção **Confirmar** e clique em **OK**.

    ![Ativar Segurança Pendente](./media/workday-inbound-tutorial/wd_isu_18.png "Ativar Segurança Pendente")  

## <a name="configuring-user-provisioning-from-workday-to-active-directory"></a>Configurando o provisionamento de usuário do Workday para o Active Directory

Esta seção fornece as etapas para o provisionamento de conta do usuário do Workday para cada domínio do Active Directory no escopo de sua integração.

* [Instalar e configurar Agentes de Provisionamento locais](#part-1-install-and-configure-on-premises-provisioning-agents)
* [Adicionar o aplicativo conector de provisionamento e criar a conexão com o Workday](#part-2-adding-the-provisioning-connector-app-and-creating-the-connection-to-workday)
* [Configurar mapeamentos de atributos](#part-3-configure-attribute-mappings)
* [Habilitar e iniciar o provisionamento de usuário](#enable-and-launch-user-provisioning)

### <a name="part-1-install-and-configure-on-premises-provisioning-agents"></a>Parte 1: Instalar e configurar Agentes de Provisionamento local

Para provisionar no Active Directory local, um agente deverá ser instalado em um servidor que tenha o .NET Framework 4.7.1+ e acesso à rede para o(s) domínio(s) do Active Directory desejado(s).

> [!TIP]
> Você pode verificar a versão do .NET Framework em seu servidor usando as instruções fornecidas [aqui](https://docs.microsoft.com/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed).
> Se o servidor não tiver o .NET 4.7.1 ou superior instalado, você poderá baixá-lo [aqui](https://support.microsoft.com/help/4033342/the-net-framework-4-7-1-offline-installer-for-windows).  

Após ter implantado o .NET 4.7.1+, você poderá baixar o **[agente de provisionamento local aqui](https://go.microsoft.com/fwlink/?linkid=847801)** e seguir as etapas fornecidas abaixo para concluir a configuração do agente.

1. Faça logon no Windows Server em que você deseja instalar o novo agente.
2. Inicie o Instalador do Agente de Provisionamento, aceite os termos e clique no botão **Instalar**.

   ![Tela Instalar](./media/workday-inbound-tutorial/pa_install_screen_1.png "Tela Instalar")
3. Após a instalação ser concluída, o assistente será iniciado e você verá a tela **Conectar o Azure AD**. Clique no botão **Autenticar** para se conectar à sua instância do Azure AD.

   ![Conectar o Azure AD](./media/workday-inbound-tutorial/pa_install_screen_2.png "Conectar o Azure AD")
1. Autentique-se na instância do Azure AD usando Credenciais de Administrador Global.

   ![Autenticação de Administrador](./media/workday-inbound-tutorial/pa_install_screen_3.png "Autenticação de Administrador")

> [!NOTE]
> As credenciais de administrador do AD do Azure são usadas apenas para se conectar ao seu locatário do Azure AD. O agente não armazena as credenciais localmente no servidor.

1. Após a autenticação bem-sucedida no Azure AD, você verá a tela **Conectar o Active Directory**. Nesta etapa, insira seu nome de domínio do AD e clique no botão **Adicionar Diretório**.

   ![Adicionar Diretório](./media/workday-inbound-tutorial/pa_install_screen_4.png "Adicionar Diretório")
  
1. Agora, será solicitado que você insira as credenciais necessárias para se conectar ao domínio do AD. Na mesma tela, você pode usar **Selecionar prioridade do controlador de domínio** para especificar os controladores de domínio que o agente deve usar para enviar solicitações de provisionamento.

   ![Credenciais do domínio](./media/workday-inbound-tutorial/pa_install_screen_5.png)
1. Depois de configurar o domínio, o instalador exibe uma lista de domínios configurados. Nessa tela, você pode repetir as etapas 5 e 6 para adicionar mais domínios ou clicar em **Avançar** para prosseguir para o registro do agente.

   ![Domínios Configurados](./media/workday-inbound-tutorial/pa_install_screen_6.png "Domínios Configurados")

   > [!NOTE]
   > Se você tiver vários domínios do AD (por exemplo, na.contoso.com, emea.contoso.com), adicione cada domínio individualmente à lista. Não é suficiente apenas adicionar o domínio pai (por exemplo, contoso.com). Você deve registrar cada domínio filho com o agente.
1. Examine os detalhes de configuração e clique em **Confirmar** para registrar o agente.
  
   ![Tela Confirmar](./media/workday-inbound-tutorial/pa_install_screen_7.png "Tela Confirmar")
1. O assistente de configuração exibe o progresso do registro do agente.
  
   ![Registro do Agente](./media/workday-inbound-tutorial/pa_install_screen_8.png "Registro do Agente")
1. Depois que o registro do agente for bem-sucedido, você poderá clicar em **Sair** para sair do Assistente.
  
   ![Tela Sair](./media/workday-inbound-tutorial/pa_install_screen_9.png "Tela Sair")
1. Verifique a instalação do Agente e certifique-se de que ele esteja em execução abrindo o Snap-In de “Serviços” e procurando pelo Serviço chamado “Agente de Provisionamento do Microsoft Azure AD Connect”
  
   ![Serviços](./media/workday-inbound-tutorial/services.png)

### <a name="part-2-adding-the-provisioning-connector-app-and-creating-the-connection-to-workday"></a>Parte 2: Adicionar o aplicativo de conector de provisionamento e criar a conexão com o Workday

**Para configurar o Workday para provisionamento do Active Directory:**

1. Acesse <https://portal.azure.com>

2. Na barra de navegação esquerda, selecione **Azure Active Directory**

3. Selecione **Aplicativos Empresariais** e, em seguida, **Todos os Aplicativos**.

4. Selecione **Adicionar um aplicativo** e a categoria **Todos**.

5. Pesquise por **Provisionamento do Workday para Active Directory** e adicione esse aplicativo da galeria.

6. Após adicionar o aplicativo e a tela de detalhes do aplicativo for exibida, selecione **Provisionamento**

7. Altere o **Modo de** **Provisionamento** para **Automático**

8. Conclua a seção **Credenciais de Administrador**, conforme a seguir:

   * **Nome de Usuário Administrador**  – digite o nome de usuário da conta do sistema de integração do Workday com o nome de domínio do locatário acrescentado. Ele deve ser semelhante: **nome de usuário\@tenant_name**

   * **Senha do administrador –** digite a senha da conta do sistema de integração do Workday

   * **URL do locatário –** digite a URL para o ponto de extremidade de serviços Web do Workday para seu locatário. Esse valor deve ser semelhante a: https://wd3-impl-services1.workday.com/ccx/service/contoso4, onde *contoso4* é substituído pelo nome do locatário correto e *wd3-impl* é substituído pela cadeia de caracteres de ambiente correta.

   * **Floresta do Active Directory –** o “Nome” de seu domínio do Active Directory, conforme registrado com o agente. Use a lista suspensa para selecionar o domínio de destino para o provisionamento. Geralmente, esse valor é uma cadeia de caracteres como: *contoso.com*

   * **Contêiner do Active Directory –** insira o DN do contêiner em que o agente deve criar contas de usuário por padrão.
        Exemplo: *OU=Standard Users,OU=Users,DC=contoso,DC=test*
     > [!NOTE]
     > Essa configuração só entra em vigor para criações de contas de usuário quando o atributo *parentDistinguishedName* não está configurado nos mapeamentos de atributo. Essa configuração não é usada para operações de atualização e pesquisa de usuários. A subárvore de todo o domínio se enquadra no escopo da operação de pesquisa.

   * **Email de notificação –** digite seu endereço de email e marque a caixa de seleção “enviar email se ocorrer falha”.

> [!NOTE]
> O Serviço de Provisionamento do Azure AD envia uma notificação por email quando o trabalho de provisionamento entra no estado de [quarentena](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning#quarantine).

   * Clique no botão **Conexão de Teste**. Se o teste de conexão for bem-sucedido, clique no botão **Salvar** na parte superior. Se ele falhar, verifique se as credenciais do Workday e as credenciais do AD configuradas na instalação de agente são válidas.

     ![Portal do Azure](./media/workday-inbound-tutorial/wd_1.png)

   * Depois que as credenciais são salvas com êxito, a seção **Mapeamentos** exibirá o mapeamento padrão **Sincronizar trabalhadores do Workday com o Active Directory local**

### <a name="part-3-configure-attribute-mappings"></a>Parte 3: Configurar mapeamentos de atributos

Nesta seção, você irá configurar o fluxo de dados de usuário do Workday para o Active Directory.

1. Na guia Provisionamento, em **Mapeamentos**, clique em **Sincronizar trabalhadores do Workday com o Active Directory local**.

2. No campo **Escopo do objeto de origem** é possível selecionar quais conjuntos de usuários no Workday devem estar no escopo de provisionamento para AD, definindo um conjunto de filtros baseados em atributo. O escopo padrão é "todos os usuários no Workday". Filtros de exemplo:

   * Exemplo: Escopo para usuários com IDs de trabalho entre 1000000 e 2000000 (exceto 2000000)

      * Atributo: WorkerID

      * Operador: Coincidir EXREG

      * Valor: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Exemplo: Apenas funcionários e trabalhadores não contingentes

      * Atributo: EmployeeID

      * Operador: IS NOT NULL

> [!TIP]
> Quando você estiver configurando o aplicativo de provisionamento de aplicativo pela primeira vez, você precisará testar e verificar seus mapeamentos de atributo e expressões para certificar-se de que ele está dando a você o resultado desejado. A Microsoft recomenda usar os filtros de escopo sob **Escopo do objeto de origem** para testar seus mapeamentos com alguns usuários de teste do Workday. Após ter verificado que os mapeamentos funcionam, você pode remover o filtro ou expandi-lo gradualmente para incluir mais usuários.

1. No campo **Ações do objeto de destino** é possível filtrar globalmente quais ações são executadas no Active Directory. **Criar** e **Atualizar** são as mais comuns.

1. Na seção **Mapeamentos de atributos** é possível definir como os atributos individuais do Workday mapeiam atributos do Active Directory.

1. Clique em um mapeamento de atributo existente para atualizá-lo ou clique em **Adicionar novo mapeamento** na parte inferior da tela para adicionar novos mapeamentos. Um mapeamento de atributo individual dá suporte para essas propriedades:

      * **Tipo de mapeamento**

         * **Direto** – grava o valor do atributo do Workday no atributo AD, sem alterações

         * **Constante** - grava um valor de cadeia de caracteres constante estático para o atributo do AD

         * **Expressão** – Permite gravar um valor personalizado para o atributo do AD baseado em um ou mais atributos do Workday. [Para obter mais informações, consulte este artigo sobre expressões](../manage-apps/functions-for-customizing-application-data.md).

      * **Atributo de origem** - O atributo de usuário do Workday. Se o atributo que você está procurando não estiver presente, consulte [Personalizar a lista de atributos de usuário do Workday](#customizing-the-list-of-workday-user-attributes).

      * **Valor padrão** – Opcional. Se o atributo de origem tiver um valor vazio, o mapeamento irá gravar esse valor.
            A configuração mais comum é deixar em branco.

      * **Atributo de destino** –  o atributo de usuário no Active Directory.

      * **Corresponder objetos utilizando esse atributo** – Se esse mapeamento deverá ou não ser utilizado para identificar usuários do Workday e Active Directory com exclusividade. Normalmente, esse valor é definido no campo ID do Trabalhador para o Workday que geralmente é mapeado para um dos atributos do ID do Funcionário no Active Directory.

      * **Precedência de correspondência** – vários atributos de correspondência podem ser definidos. Quando houver múltiplos, os atributos serão avaliados na ordem definida por esse campo. Assim que uma correspondência for encontrada, nenhum outro atributo correspondente será avaliado.

      * **Aplicar esse mapeamento**

         * **Sempre** – aplicar esse mapeamento nas ações de criação e atualização do usuário

         * **Somente durante a criação** – aplicar esse mapeamento somente nas ações de criação de usuário

1. Para salvar seus mapeamentos, clique em **Salvar** na parte superior da seção Mapeamento de Atributos.

   ![Portal do Azure](./media/workday-inbound-tutorial/wd_2.png)

#### <a name="below-are-some-example-attribute-mappings-between-workday-and-active-directory-with-some-common-expressions"></a>A seguir, estão alguns exemplos de mapeamentos de atributos entre o Workday e o Active Directory, com algumas expressões comuns

* A expressão que mapeia para o atributo *parentDistinguishedName* é utilizada para provisionar um usuário para OUs diferentes baseado em um ou mais atributos de origem do Workday. Este exemplo coloca os usuários em OUs diferentes com base na cidade que eles estão.

* O atributo *userPrincipalName* no Active Directory é gerado usando a função de eliminação de duplicação [SelectUniqueValue](../manage-apps/functions-for-customizing-application-data.md#selectuniquevalue) que verifica a existência de um valor gerado no domínio do AD de destino e apenas define se ele é exclusivo.  

* [Há documentação sobre expressões de gravação aqui](../manage-apps/functions-for-customizing-application-data.md). Esta seção inclui exemplos sobre como remover caracteres especiais.

| ATRIBUTO WORKDAY | ATRIBUTO DO ACTIVE DIRECTORY |  ID DE CORRESPONDÊNCIA? | CRIAR / ATUALIZAR |
| ---------- | ---------- | ---------- | ---------- |
| **WorkerID**  |  EmployeeID | **Sim** | Gravado na criação somente |
| **PreferredNameData**    |  cn    |   |   Gravado na criação somente |
| **SelectUniqueValue (Join ("\@", Junte-se (".", \[FirstName\], \[LastName\]), "contoso.com"), Join ("\@", Junte-se (".", Mid (\[FirstName\], 1, 1 () \[LastName\]), "contoso.com"), Join ("\@", Join (".", Mid (\[FirstName\], 1, 2), \[LastName\]), "contoso.com"))**   | userPrincipalName     |     | Gravado na criação somente 
| **Replace(Mid(Replace(\[UserID\], , "(\[\\\\/\\\\\\\\\\\\\[\\\\\]\\\\:\\\\;\\\\\|\\\\=\\\\,\\\\+\\\\\*\\\\?\\\\&lt;\\\\&gt;\])", , "", , ), 1, 20), , "([\\\\.)\*\$](file:///\\.)*$)", , "", , )**      |    sAMAccountName            |     |         Gravado na criação somente |
| **Switch(\[Active\], , "0", "True", "1", "False")** |  accountDisabled      |     | Criar + atualizar |
| **Nome**   | givenName       |     |    Criar + atualizar |
| **Sobrenome**   |   sn   |     |  Criar + atualizar |
| **PreferredNameData**  |  displayName |     |   Criar + atualizar |
| **Empresa**         | company   |     |  Criar + atualizar |
| **SupervisoryOrganization**  | department  |     |  Criar + atualizar |
| **ManagerReference**   | manager  |     |  Criar + atualizar |
| **BusinessTitle**   |  título     |     |  Criar + atualizar | 
| **AddressLineData**    |  streetAddress  |     |   Criar + atualizar |
| **Município**   |   l   |     | Criar + atualizar |
| **CountryReferenceTwoLetter**      |   co |     |   Criar + atualizar |
| **CountryReferenceTwoLetter**    |  c  |     |         Criar + atualizar |
| **CountryRegionReference** |  st     |     | Criar + atualizar |
| **WorkSpaceReference** | physicalDeliveryOfficeName    |     |  Criar + atualizar |
| **PostalCode**  |   postalCode  |     | Criar + atualizar |
| **PrimaryWorkTelephone**  |  telephoneNumber   |     | Criar + atualizar |
| **Fax**      | facsimileTelephoneNumber     |     |    Criar + atualizar |
| **Móvel**  |    Serviço Móvel       |     |       Criar + atualizar |
| **LocalReference** |  preferredLanguage  |     |  Criar + atualizar |                                               
| **Switch(\[Municipality\], "OU=Standard Users,OU=Users,OU=Default,OU=Locations,DC=contoso,DC=com", "Dallas", "OU=Standard Users,OU=Users,OU=Dallas,OU=Locations,DC=contoso,DC=com", "Austin", "OU=Standard Users,OU=Users,OU=Austin,OU=Locations,DC=contoso,DC=com", "Seattle", "OU=Standard Users,OU=Users,OU=Seattle,OU=Locations,DC=contoso,DC=com", “London", "OU=Standard Users,OU=Users,OU=London,OU=Locations,DC=contoso,DC=com")**  | parentDistinguishedName     |     |  Criar + atualizar |

Uma vez concluída a configuração do mapeamento de atributo, você pode agora [habilitar e iniciar o serviço de provisionamento de usuário](#enable-and-launch-user-provisioning).

## <a name="configuring-user-provisioning-to-azure-ad"></a>Configurar o provisionamento de usuário para Azure AD

As seções a seguir descrevem as etapas para configurar o provisionamento de usuário do Workday para o Azure AD para implantações somente em nuvem.

* [Adicionar o aplicativo conector de provisionamento do Azure AD e criar a conexão com o Workday](#part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday)
* [Configurar mapeamentos de atributo do Workday e do Azure AD](#part-2-configure-workday-and-azure-ad-attribute-mappings)
* [Habilitar e iniciar o provisionamento de usuário](#enable-and-launch-user-provisioning)

> [!IMPORTANT]
> Siga o procedimento abaixo apenas se você tiver usuários somente na nuvem que precisam ser provisionados para o Azure AD e não para o Active Directory local.

### <a name="part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday"></a>Parte 1: Adicionar o aplicativo do conector de provisionamento do Azure AD e criar a conexão com o Workday

**Para configurar o provisionamento do Workday to Azure Active Directory para usuários somente na nuvem:**

1. Vá para <https://portal.azure.com>.

2. Na barra de navegação esquerda, selecione **Azure Active Directory**

3. Selecione **Aplicativos Empresariais** e, em seguida, **Todos os Aplicativos**.

4. Selecione **Adicionar um aplicativo** e, em seguida, selecione a categoria **Todos**.

5. Pesquise por **Workday para provisionamento do Azure AD** e adicione esse aplicativo da galeria.

6. Após adicionar o aplicativo e a tela de detalhes do aplicativo for exibida, selecione **Provisionamento**

7. Altere o **Modo de** **Provisionamento** para **Automático**

8. Conclua a seção **Credenciais de Administrador**, conforme a seguir:

   * **Nome de Usuário Administrador**  – Digite o nome de usuário da conta do sistema de integração do Workday com o nome de domínio do locatário acrescentado. Deve ser semelhante a: username@contoso4

   * **Senha do administrador –** digite a senha da conta do sistema de integração do Workday

   * **URL do locatário –** digite a URL para o ponto de extremidade de serviços Web do Workday para seu locatário. Esse valor deve ser semelhante a: https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources, onde *contoso4* é substituído pelo nome do locatário correto e *wd3-impl* é substituído pela cadeia de caracteres de ambiente correta. Se essa URL não for conhecida, trabalhe com seu representante de suporte ou parceiro de integração do Workday para determinar a URL correta a ser usada.

   * **Email de notificação –** digite seu endereço de email e marque a caixa de seleção “enviar email se ocorrer falha”.

   * Clique no botão **Conexão de Teste**.

   * Se o teste de conexão for bem-sucedido, clique no botão **Salvar** na parte superior. Se falhar, verifique novamente se o URL do Workday e as credenciais são válidas no Workday.

### <a name="part-2-configure-workday-and-azure-ad-attribute-mappings"></a>Parte 2: Configurar mapeamentos de atributo do Workday e do Azure AD

Nesta seção, você irá configurar o fluxo dos dados do usuário do Workday para o Azure Active Directory de usuários somente na nuvem.

1. Na guia Provisionamento em **Mapeamentos**, clique em **Sincronizar Trabalhos para Azure AD**.

2. No campo **Escopo do objeto de origem** é possível selecionar quais conjuntos de usuários no Workday devem estar no escopo de provisionamento para Azure AD, definindo um conjunto de filtros baseados em atributo. O escopo padrão é “todos os usuários no Workday”. Filtros de exemplo:

   * Exemplo: Escopo para usuários com IDs do Trabalhador entre 1000000 e 2000000

      * Atributo: WorkerID

      * Operador: Coincidir EXREG

      * Valor: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Exemplo: Apenas trabalhadores contingentes e funcionários não regulares

      * Atributo: ContingentID

      * Operador: IS NOT NULL

3. No campo **Ações do objeto de destino** é possível filtrar globalmente quais ações podem ser realizadas no Azure AD. **Criar** e **Atualizar** são as mais comuns.

4. Na seção **Mapeamentos de atributos** é possível definir como os atributos individuais do Workday mapeiam atributos do Active Directory.

5. Clique em um mapeamento de atributo existente para atualizá-lo ou clique em **Adicionar novo mapeamento** na parte inferior da tela para adicionar novos mapeamentos. Um mapeamento de atributo individual dá suporte para essas propriedades:

   * **Tipo de mapeamento**

      * **Direto** – grava o valor do atributo do Workday no atributo AD, sem alterações

      * **Constante** - grava um valor de cadeia de caracteres constante estático para o atributo do AD

      * **Expressão** – Permite gravar um valor personalizado para o atributo do AD baseado em um ou mais atributos do Workday. [Para obter mais informações, consulte este artigo sobre expressões](../manage-apps/functions-for-customizing-application-data.md).

   * **Atributo de origem** - O atributo de usuário do Workday. Se o atributo que você está procurando não estiver presente, consulte [Personalizar a lista de atributos de usuário do Workday](#customizing-the-list-of-workday-user-attributes).

   * **Valor padrão** – Opcional. Se o atributo de origem tiver um valor vazio, o mapeamento irá gravar esse valor.
            A configuração mais comum é deixar em branco.

   * **Atributo de destino** – O atributo do usuário no Azure AD.

   * **Corresponder objetos utilizando esse atributo** – se esse mapeamento deverá ou não ser utilizado para identificar usuários entre o Workday e o Azure AD com exclusividade. Normalmente, esse valor é definido no campo ID do Trabalhador para o Workday que normalmente é mapeado para o atributo ID do Funcionário (novo) ou um atributo de extensão no Azure AD.

   * **Precedência de correspondência** – vários atributos de correspondência podem ser definidos. Quando houver múltiplos, os atributos serão avaliados na ordem definida por esse campo. Assim que uma correspondência for encontrada, mais nenhum atributo correspondente será avaliado.

   * **Aplicar esse mapeamento**

     * **Sempre** – aplicar esse mapeamento nas ações de criação e atualização do usuário

     * **Somente durante a criação** – aplicar esse mapeamento somente nas ações de criação de usuário

6. Para salvar seus mapeamentos, clique em **Salvar** na parte superior da seção Mapeamento de Atributos.

Uma vez concluída a configuração do mapeamento de atributo, você pode agora [habilitar e iniciar o serviço de provisionamento de usuário](#enable-and-launch-user-provisioning).

## <a name="configuring-writeback-of-email-addresses-to-workday"></a>Configurando write-back de endereços de email para Workday

Siga essas instruções para configurar o write-back de endereços de email do usuário do Azure Active Directory para Workday.

* [Adicionar o aplicativo conector de write-back e criar a conexão com o Workday](#part-1-adding-the-writeback-connector-app-and-creating-the-connection-to-workday)
* [Configurar mapeamentos de atributos de write-back](#part-2-configure-writeback-attribute-mappings)
* [Habilitar e iniciar o provisionamento de usuário](#enable-and-launch-user-provisioning)

### <a name="part-1-adding-the-writeback-connector-app-and-creating-the-connection-to-workday"></a>Parte 1: Adicionar o aplicativo conector de write-back e criar a conexão com o Workday

**Para configurar o conector de Write-back do Workday:**

1. Acesse <https://portal.azure.com>

2. Na barra de navegação esquerda, selecione **Azure Active Directory**

3. Selecione **Aplicativos Empresariais** e, em seguida, **Todos os Aplicativos**.

4. Selecione **Adicionar um aplicativo** e, em seguida, selecione a categoria **Todos**.

5. Pesquise por **Fazer write-back de Workday** e adicione esse aplicativo da galeria.

6. Após adicionar o aplicativo e a tela de detalhes do aplicativo for exibida, selecione **Provisionamento**

7. Altere o **Modo de** **Provisionamento** para **Automático**

8. Conclua a seção **Credenciais de Administrador**, conforme a seguir:

   * **Nome de Usuário Administrador**  – Digite o nome de usuário da conta do sistema de integração do Workday com o nome de domínio do locatário acrescentado. Deve ser parecido com: *nome de usuário\@contoso4*

   * **Senha do administrador –** digite a senha da conta do sistema de integração do Workday

   * **URL do locatário –** Digite a URL para o ponto de extremidade de serviços Web do Workday para seu locatário. Esse valor deve ser semelhante a: https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources, onde *contoso4* é substituído pelo seu nome de locatário correto e *wd3-impl* é substituído pela cadeia de caracteres de ambiente correta (se necessário).

   * **Email de notificação –** digite seu endereço de email e marque a caixa de seleção “enviar email se ocorrer falha”.

   * Clique no botão **Conexão de Teste**. Se o teste de conexão for bem-sucedido, clique no botão **Salvar** na parte superior. Se falhar, verifique novamente se o URL do Workday e as credenciais são válidas no Workday.

### <a name="part-2-configure-writeback-attribute-mappings"></a>Parte 2: Configurar mapeamentos de atributos de write-back

Nesta seção, você irá configurar como os atributos de write-back fluem do Azure AD para o Workday.

1. Na guia Provisionamento em **Mapeamentos**, clique em **Sincronizar usuários do Azure Active Directory para Workday**.

2. No campo **Escopo do objeto de Origem** é possível, opcionalmente, filtrar quais conjuntos de usuários no Azure Active Directory devem fazer write-back dos endereços de email para o Workday. O escopo padrão é "todos os usuários no Azure AD".

3. Na seção **Mapeamentos de atributo**, atualize a ID correspondente para indicar o atributo no Azure Active Directory em que a ID de trabalho do Workday ou a ID de funcionário é armazenada. Um método de correspondência popular é sincronizar o ID de Trabalho do Workday ou o ID de Funcionário para extensionAttribute1-15 no Azure AD e, em seguida, utilizar esse atributo no Azure AD para corresponder os usuários no Workday.

4. Para salvar seus mapeamentos, clique em **Salvar** na parte superior da seção Mapeamento de Atributos.

Uma vez concluída a configuração do mapeamento de atributo, você pode agora [habilitar e iniciar o serviço de provisionamento de usuário](#enable-and-launch-user-provisioning). 

## <a name="enable-and-launch-user-provisioning"></a>Habilitar e iniciar o provisionamento de usuário

Depois que as configurações do aplicativo de provisionamento do Workday tiverem sido concluídas, você pode ativar o serviço de provisionamento no portal do Azure.

> [!TIP]
> Por padrão, quando você ativa o serviço de provisionamento, ele iniciará as operações de provisionamento para todos os usuários no escopo. Se houver erros em problemas de dados de mapeamento ou Workday, o trabalho de provisionamento pode falhar e prosseguir para o estado de quarentena. Para evitar isso, como uma prática recomendada, recomendamos configurar o filtro **Escopo do objeto de origem** e testar seus mapeamentos de atributos com alguns usuários de teste antes de iniciar a sincronização completa para todos os usuários. Após ter verificado que os mapeamentos funcionam e estão fornecendo os resultados desejados, você pode remover o filtro ou expandi-lo gradualmente para incluir mais usuários.

1. Na guia **Provisionamento**, defina o **Status de Provisionamento** para **Em**.

2. Clique em **Salvar**.

3. Essa operação dará início à sincronização inicial, o que poderá demorar algumas horas dependendo de quantos usuários estiverem no locatário do Workday. 

4. A qualquer momento, verifique os **Logs de auditoria** no portal do Azure para ver as ações que o serviço de provisionamento executou. Os logs de auditoria listam todos os eventos de sincronização realizados pelo serviço de provisionamento, tais como aqueles em que os usuários estão sendo lidos do Workday e, posteriormente adicionados ou atualizados no Active Directory. Consulte a seção Solução de problemas para obter instruções sobre como examinar os logs de auditoria e corrigir erros de provisionamento.

5. Após a sincronização inicial ser concluída, um relatório de resumo de auditoria será gravado na guia **Provisionamento** conforme mostrado abaixo.

   ![Portal do Azure](./media/workday-inbound-tutorial/wd_3.png)

## <a name="frequently-asked-questions-faq"></a>Perguntas frequentes (FAQ)

* **Perguntas do recurso de solução**
  * [Ao processar uma nova contratação do Workday, como a solução define a senha para a nova conta de usuário no Active Directory?](#when-processing-a-new-hire-from-workday-how-does-the-solution-set-the-password-for-the-new-user-account-in-active-directory)
  * [A solução dá suporte a envio de notificações por email após a conclusão de operações de provisionamento?](#does-the-solution-support-sending-email-notifications-after-provisioning-operations-complete)
  * [Como gerenciar a entrega de senhas para novas contratações e fornecer um mecanismo seguro para redefinir sua senha?](#how-do-i-manage-delivery-of-passwords-for-new-hires-and-securely-provide-a-mechanism-to-reset-their-password)
  * [A solução armazena em cache os perfis de usuário do Workday na nuvem do Azure AD ou na camada de agente de provisionamento?](#does-the-solution-cache-workday-user-profiles-in-the-azure-ad-cloud-or-at-the-provisioning-agent-layer)
  * [A atribuição dá suporte a atribuição de grupos locais do AD para o usuário?](#does-the-solution-support-assigning-on-premises-ad-groups-to-the-user)
  * [Quais APIs do Workday a solução utiliza para consultar e atualizar perfis de trabalhadores do Workday?](#which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles)
  * [Posso configurar meu locatário do HCM do Workday com dois locatários do Azure AD?](#can-i-configure-my-workday-hcm-tenant-with-two-azure-ad-tenants)
  * [Por que o aplicativo de provisionamento do usuário do “Workday para Azure AD” não tem suporte se implantamos o Azure AD Connect?](#why-workday-to-azure-ad-user-provisioning-app-is-not-supported-if-we-have-deployed-azure-ad-connect)
  * [Como sugerir melhorias ou solicitar novos recursos relacionados à integração do Workday e do Azure AD?](#how-do-i-suggest-improvements-or-request-new-features-related-to-workday-and-azure-ad-integration)

* **Perguntas sobre o Agente de Provisionamento**
  * [O que é a versão de disponibilidade geral (GA) do Agente de Provisionamento?](#what-is-the-ga-version-of-the-provisioning-agent)
  * [Como saber a versão do meu Agente de Provisionamento?](#how-do-i-know-the-version-of-my-provisioning-agent)
  * [A Microsoft envia por push automaticamente as atualizações do Agente de Provisionamento?](#does-microsoft-automatically-push-provisioning-agent-updates)
  * [Posso instalar o Agente de Provisionamento no mesmo servidor que executa o AAD Connect?](#can-i-install-the-provisioning-agent-on-the-same-server-running-aad-connect)
  * [Como fazer para configurar o Agente de Provisionamento para usar um servidor proxy para comunicação HTTP de saída?](#how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication)
  * [Como garantir que o Agente de Provisionamento é capaz de se comunicar com o locatário do Azure AD e que nenhum firewall está bloqueando as portas exigidas pelo agente?](#how-do-i-ensure-that-the-provisioning-agent-is-able-to-communicate-with-the-azure-ad-tenant-and-no-firewalls-are-blocking-ports-required-by-the-agent)
  * [Como cancelar o registro do domínio associado ao meu Agente de Provisionamento?](#how-do-i-de-register-the-domain-associated-with-my-provisioning-agent)
  * [Como desinstalar o Agente de Provisionamento?](#how-do-i-uninstall-the-provisioning-agent)
  
* **Perguntas sobre configuração e mapeamento de atributos do Workday para o AD**
  * [Como fazer backup ou exportar uma cópia funcional do meu esquema e mapeamento de atributos de provisionamento do Workday?](#how-do-i-back-up-or-export-a-working-copy-of-my-workday-provisioning-attribute-mapping-and-schema)
  * [Eu tenho atributos personalizados no Workday e no Active Directory. Como fazer para configurar a solução para trabalhar com os meus atributos personalizados?](#i-have-custom-attributes-in-workday-and-active-directory-how-do-i-configure-the-solution-to-work-with-my-custom-attributes)
  * [Posso configurar a foto do usuário do Workday para o Active Directory?](#can-i-provision-users-photo-from-workday-to-active-directory)
  * [Como faço para sincronizar os números de celular do Workday com base no consentimento do usuário para uso público?](#how-do-i-sync-mobile-numbers-from-workday-based-on-user-consent-for-public-usage)
  * [Como formatar nomes de exibição no AD com base em atributos de país/departamento/cidade do usuário e tratar as variações regionais?](#how-do-i-format-display-names-in-ad-based-on-the-users-departmentcountrycity-attributes-and-handle-regional-variances)
  * [Como usar SelectUniqueValue para gerar valores exclusivos para o atributo samAccountName?](#how-can-i-use-selectuniquevalue-to-generate-unique-values-for-samaccountname-attribute)
  * [Como remover caracteres com diacríticos e convertê-los em letras normais do alfabeto em português?](#how-do-i-remove-characters-with-diacritics-and-convert-them-into-normal-english-alphabets)

### <a name="solution-capability-questions"></a>Perguntas do recurso de solução

#### <a name="when-processing-a-new-hire-from-workday-how-does-the-solution-set-the-password-for-the-new-user-account-in-active-directory"></a>Ao processar uma nova contratação do Workday, como a solução define a senha para a nova conta de usuário no Active Directory?

Quando o agente de provisionamento local obtém uma solicitação para criar uma nova conta do AD, ele automaticamente gera uma senha aleatória complexa, projetada para atender aos requisitos de complexidade de senha definidos pelo servidor do AD e define isso no objeto do usuário. Essa senha não é registrada em nenhum lugar.

#### <a name="does-the-solution-support-sending-email-notifications-after-provisioning-operations-complete"></a>A solução dá suporte a envio de notificações por email após a conclusão de operações de provisionamento?

Não, enviar notificações por email após a conclusão de operações de provisionamento não tem suporte na versão atual.

#### <a name="how-do-i-manage-delivery-of-passwords-for-new-hires-and-securely-provide-a-mechanism-to-reset-their-password"></a>Como gerenciar a entrega de senhas para novas contratações e fornecer um mecanismo seguro para redefinir sua senha?

Uma das últimas etapas para o provisionamento de uma nova conta do AD é o fornecimento da senha temporária atribuída à conta do AD do usuário. Muitas empresas ainda usam a abordagem tradicional de fornecer a senha temporária para o gerente do usuário, que, em seguida, passa a senha para a contratação nova/temporária. Esse processo tem uma falha de segurança inerente e há uma opção disponível para implementar uma abordagem melhor usando os recursos do Azure AD.

Como parte do processo de contratação, as equipes de RH geralmente executam uma verificação do histórico e verificam o número do celular da nova contratação. Com a integração do Provisionamento de Usuário do Workday para AD, você pode aproveitar esse fato e distribuir um recurso de redefinição de senha self-service para o usuário no Dia 1. Isso é feito propagando o atributo “Número de celular” da nova contratação do Workday para o AD e, em seguida, do AD para o Azure AD usando o AAD Connect. Depois que o “Número de celular” estiver presente no Azure AD, você pode habilitar a [Redefinição de senha self-service (SSPR)](../authentication/howto-sspr-authenticationdata.md) para a conta do usuário, de forma que no Dia 1, uma nova contratação pode usar o número do celular registrado e verificado para autenticação.

#### <a name="does-the-solution-cache-workday-user-profiles-in-the-azure-ad-cloud-or-at-the-provisioning-agent-layer"></a>A solução armazena em cache os perfis de usuário do Workday na nuvem do Azure AD ou na camada de agente de provisionamento?

Não, a solução não mantém um cache de perfis de usuário. O serviço de provisionamento do Azure AD simplesmente atua como um processador de dados, lendo dados do Workday e gravando Active Directory ou Azure AD de destino. Consulte a seção [Gerenciamento de dados pessoais](#managing-personal-data) para obter detalhes relacionados à retenção de dados e privacidade do usuário.

#### <a name="does-the-solution-support-assigning-on-premises-ad-groups-to-the-user"></a>A atribuição dá suporte a atribuição de grupos locais do AD para o usuário?

Não há suporte para essa funcionalidade no momento. A solução alternativa recomendada é implantar um script do PowerShell que consulta o ponto de extremidade de API do Graph do Azure AD para dados de log de auditoria e usá-lo para disparar cenários, como atribuição de grupo. Este script do PowerShell pode ser anexado a um agendador de tarefas e pode ser implantado na mesma caixa que executa o agente de provisionamento.  

#### <a name="which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles"></a>Quais APIs do Workday a solução utiliza para consultar e atualizar perfis de trabalhadores do Workday?

Atualmente, a solução usa as seguintes APIs do Workday:

* Get_Workers (v21.1) para buscar informações de trabalho
* Maintain_Contact_Information (v26.1) para o recurso de write-back de email de trabalho

#### <a name="can-i-configure-my-workday-hcm-tenant-with-two-azure-ad-tenants"></a>Posso configurar meu locatário do HCM do Workday com dois locatários do Azure AD?

Sim, há suporte para essa configuração. Aqui estão as etapas de alto nível para configurar esse cenário:

* Implantar o agente de provisionamento 1 e registrá-lo no locatário 1 do Azure AD.
* Implantar o agente de provisionamento 2 e registrá-lo no locatário 2 do Azure AD.
* Com base nos “Domínios filhos” que cada Agente de Provisionamento irá gerenciar, configure cada agente com o(s) domínio(s). Um agente pode lidar com vários domínios.
* No portal do Azure, configure o aplicativo de Provisionamento de Usuário do Workday para o AD em cada locatário e configure-o com os respectivos domínios.

#### <a name="why-workday-to-azure-ad-user-provisioning-app-is-not-supported-if-we-have-deployed-azure-ad-connect"></a>Por que o aplicativo de provisionamento do usuário do “Workday para Azure AD” não tem suporte se implantamos o Azure AD Connect?

Quando o Azure AD é usado no modo híbrido (em que ele contém uma mistura de nuvem + usuários locais), é importante ter uma definição clara de “origem de autoridade”. Normalmente, os cenários híbridos exigem a implantação do Azure AD Connect. Quando o Azure AD Connect é implantado, o AD local é a origem de autoridade. Incluir o conector do Workday para o Azure AD pode levar a uma situação em que os valores de atributo do Workday potencialmente poderia substituir os valores definidos pelo Azure AD Connect. Portanto, o uso do aplicativo de provisionamento “Workday para Azure AD” não tem suporte quando o Azure AD Connect está habilitado. Em tais situações, é recomendável usar o aplicativo de provisionamento de “Usuário do Workday para AD" provisionamento para obter os usuários no AD local e, em seguida, sincronizá-los ao Azure AD usando o Azure AD Connect.

#### <a name="how-do-i-suggest-improvements-or-request-new-features-related-to-workday-and-azure-ad-integration"></a>Como sugerir melhorias ou solicitar novos recursos relacionados à integração do Workday e do Azure AD?

Seus comentários são muito importantes, pois eles nos ajudam a definir um direcionamento para as futuras versões e aprimoramentos. Estamos abertos a todos os comentários e encorajamos você a enviar sua ideia ou sugestão de melhoria na [fórum de comentários do Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory). Para comentários específicos relacionados à integração do Workday, selecione a categoria *Aplicativos SaaS* e pesquise usando as palavras-chave *Workday* para encontrar os comentários existentes relacionados ao Workday.

![Aplicativos SaaS do UserVoice](media/workday-inbound-tutorial/uservoice_saas_apps.png)

![Workday do UserVoice](media/workday-inbound-tutorial/uservoice_workday_feedback.png)

Ao sugerir uma nova ideia, verifique para ver se alguém já sugeriu um recurso semelhante. Nesse caso, você pode votar na solicitação de recurso ou aprimoramento. Você também pode deixar um comentário sobre o seu caso de uso específico para mostrar seu apoio à ideia e demonstrar como o recurso será importante para você também.

### <a name="provisioning-agent-questions"></a>Perguntas sobre o Agente de Provisionamento

#### <a name="what-is-the-ga-version-of-the-provisioning-agent"></a>O que é a versão de disponibilidade geral (GA) do Agente de Provisionamento?

* Aa versão de disponibilidade geral (GA) do Agente de Provisionamento é 1.1.30 e acima.
* Se sua versão do agente é menor que 1.1.30, você está executando a versão prévia pública e ela será automaticamente atualizada para a versão de GA se o servidor que hospeda o agente tiver o tempo de execução de .NET 4.7.1.
  * Você pode [verificar a versão do .NET](https://docs.microsoft.com/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) instalado em seu servidor. Se o servidor não estiver executando o .NET 4.7.1, você pode [baixar e instalar o .NET 4.7.1](https://support.microsoft.com/help/4033342/the-net-framework-4-7-1-offline-installer-for-windows). O agente de provisionamento será atualizado automaticamente para a versão de GA após a instalação do .NET 4.7.1.

#### <a name="how-do-i-know-the-version-of-my-provisioning-agent"></a>Como saber a versão do meu Agente de Provisionamento?

* Faça logon no servidor Windows em que o Agente de Provisionamento está instalado.
* Vá até o menu **Painel de controle** -> **Desinstalar ou alterar um programa**
* Procure a versão correspondente à entrada **Agente de provisionamento do Microsoft Azure AD Connect**

  ![Portal do Azure](./media/workday-inbound-tutorial/pa_version.png)

#### <a name="does-microsoft-automatically-push-provisioning-agent-updates"></a>A Microsoft envia por push automaticamente as atualizações do Agente de Provisionamento?

Sim, a Microsoft atualiza automaticamente o agente de provisionamento. Você pode desabilitar as atualizações automáticas interrompendo o serviço **Atualizador do agente do Microsoft Azure AD Connect** do Windows.

#### <a name="can-i-install-the-provisioning-agent-on-the-same-server-running-aad-connect"></a>Posso instalar o Agente de Provisionamento no mesmo servidor que executa o AAD Connect?

Sim, você pode instalar o Agente de Provisionamento no mesmo servidor que executa o AAD Connect.

#### <a name="at-the-time-of-configuration-the-provisioning-agent-prompts-for-azure-ad-admin-credentials-does-the-agent-store-the-credentials-locally-on-the-server"></a>No momento da configuração o Agente de Provisionamento solicita as credenciais de administrador do Azure AD. O Agente armazena as credenciais localmente no servidor?

Durante a configuração o Agente de Provisionamento solicita as credenciais de administrador do Azure AD somente para conectar o seu locatário do Azure AD. Ele não armazena as credenciais localmente no servidor. No entanto, ele retém as credenciais usadas para conectar-se ao *domínio local do Active Directory* em um cofre de senha local do Windows.

#### <a name="how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication"></a>Como fazer para configurar o Agente de Provisionamento para usar um servidor proxy para comunicação HTTP de saída?

O Agente de Provisionamento dá suporte ao uso do proxy de saída. Você pode configurá-lo editando o arquivo de configuração do agente **C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config**. Adicione as seguintes linhas a ele, no final do arquivo pouco antes da marca `</configuration>` de fechamento.
Substitua as variáveis [proxy-server] e [proxy-port] pelo nome do servidor proxy e pelos valores da porta.

```xml
    <system.net>
          <defaultProxy enabled="true" useDefaultCredentials="true">
             <proxy
                usesystemdefault="true"
                proxyaddress="http://[proxy-server]:[proxy-port]"
                bypassonlocal="true"
             />
         </defaultProxy>
    </system.net>
```

#### <a name="how-do-i-ensure-that-the-provisioning-agent-is-able-to-communicate-with-the-azure-ad-tenant-and-no-firewalls-are-blocking-ports-required-by-the-agent"></a>Como garantir que o Agente de Provisionamento é capaz de se comunicar com o locatário do Azure AD e que nenhum firewall está bloqueando as portas exigidas pelo agente?

Você também pode verificar se você tem todas as portas abertas necessárias, abrindo o [ferramenta de teste de portas do conector](https://aadap-portcheck.connectorporttest.msappproxy.net/) da sua rede local. Mais marcas de seleção verde significa maior resiliência.

Para certificar-se de que a ferramenta oferece os resultados certos, certifique-se de:

* Abra a ferramenta em um navegador do servidor em que você instalou o Agente de Provisionamento.
* Certifique-se de que quaisquer proxies ou firewalls aplicáveis a seu Agente de Provisionamento também sejam aplicados a esta página. Isso pode ser feito no Internet Explorer, vá para **Configurações -> Opções da Internet -> conexões -> configurações da LAN**. Nessa página, você vê o campo “Usar um Servidor Proxy para sua LAN”. Marque essa caixa e coloque o endereço do proxy no campo “Endereço”.

#### <a name="can-one-provisioning-agent-be-configured-to-provision-multiple-ad-domains"></a>Um Agente de Provisionamento pode ser configurado para provisionar vários domínios do AD?

Sim, um Agente de Provisionamento pode ser configurado para lidar com vários domínios do AD, desde que o agente tenha metas claras para os respectivos controladores de domínio. A Microsoft recomenda configurar um grupo de 3 agentes de provisionamento servindo o mesmo conjunto de domínios do AD para garantir alta disponibilidade e fornecer suporte para failover.

#### <a name="how-do-i-de-register-the-domain-associated-with-my-provisioning-agent"></a>Como cancelar o registro do domínio associado ao meu Agente de Provisionamento?

* No portal do Azure, obtenha a *ID do locatário* do seu locatário do Azure AD.
* Faça logon no servidor Windows em que o Agente de Provisionamento está em execução.
* Abra o PowerShell como administrador do Windows.
* Altere o diretório que contém os scripts de registro e execute os comandos a seguir substituindo o parâmetro de \[ID do locatário\] pelo valor da ID do locatário.

  ```powershell
  cd “C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\RegistrationPowershell\Modules\PSModulesFolder”
  Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\RegistrationPowershell\Modules\PSModulesFolder\AppProxyPSModule.psd1"
  Get-PublishedResources -TenantId "[tenant ID]"
  ```

* Na lista de agentes que aparecer, copie o valor do campo “id” do recurso cujo *resourceName* seja igual ao nome de domínio do AD.
* Cole a id neste comando e execute-o no Powershell.

  ```powershell
  Remove-PublishedResource -ResourceId "[resource ID]" -TenantId "[tenant ID]"
  ```

* Execute novamente o assistente de configuração do Agente.
* Quaisquer outros agentes que foram atribuídos anteriormente a esse domínio precisarão ser reconfigurados.

#### <a name="how-do-i-uninstall-the-provisioning-agent"></a>Como desinstalar o Agente de Provisionamento?

* Faça logon no servidor Windows em que o Agente de Provisionamento está instalado.
* Vá até o menu **Painel de controle** -> **Desinstalar ou alterar um programa**
* Desinstale os programas a seguir:
  * Agente de Provisionamento do Microsoft Azure AD Connect
  * Atualizador do Agente do Microsoft Azure AD Connect
  * Pacote do Agente de Provisionamento do Microsoft Azure AD Connect

### <a name="workday-to-ad-attribute-mapping-and-configuration-questions"></a>Perguntas sobre configuração e mapeamento de atributos do Workday para o AD

#### <a name="how-do-i-back-up-or-export-a-working-copy-of-my-workday-provisioning-attribute-mapping-and-schema"></a>Como fazer backup ou exportar uma cópia funcional do meu esquema e mapeamento de atributos de provisionamento do Workday?

Você pode usar a API do Microsoft Graph para exportar sua configuração de Provisionamento de Usuário do Workday. Consulte as etapas na seção [Exportar e importar a configuração de mapeamento de atributos de Provisionamento de Usuário do Workday](#exporting-and-importing-your-configuration) para obter detalhes.

#### <a name="i-have-custom-attributes-in-workday-and-active-directory-how-do-i-configure-the-solution-to-work-with-my-custom-attributes"></a>Eu tenho atributos personalizados no Workday e no Active Directory. Como fazer para configurar a solução para trabalhar com os meus atributos personalizados?

A solução dá suporte a atributos personalizados do Workday e Active Directory. Para adicionar seus atributos personalizados ao esquema de mapeamento, abra a folha **Mapeamento de atributos** e role para baixo para expandir a seção **Mostrar opções avançadas**. 

![Editar a Lista de Atributos](./media/workday-inbound-tutorial/wd_edit_attr_list.png)

Para adicionar seus atributos personalizados do Workday, selecione a opção *Editar a lista de atributos para Workday* e para adicionar seus atributos personalizados do AD, selecione a opção *Editar lista de atributos no Active Directory local*.

Consulte também:

* [Personalizando a lista de atributos de usuário do Workday](#customizing-the-list-of-workday-user-attributes)

#### <a name="how-do-i-configure-the-solution-to-only-update-attributes-in-ad-based-on-workday-changes-and-not-create-any-new-ad-accounts"></a>Como fazer para configurar a solução para atualizar somente os atributos no AD com base nas alterações do Workday e não criar novas contas do AD?

Essa configuração pode ser feita definindo as **Ações do objeto de destino** na folha **Mapeamentos de atributos** conforme mostrado abaixo:

![Atualizar ação](./media/workday-inbound-tutorial/wd_target_update_only.png)

Marque a caixa de seleção “Atualizar” apenas para operações de atualização para o fluxo do Workday para o AD. 

#### <a name="can-i-provision-users-photo-from-workday-to-active-directory"></a>Posso configurar a foto do usuário do Workday para o Active Directory?

A solução atualmente não dá suporte a atributos binários de configuração, como *thumbnailPhoto* e *jpegPhoto* no Active Directory.

#### <a name="how-do-i-sync-mobile-numbers-from-workday-based-on-user-consent-for-public-usage"></a>Como faço para sincronizar os números de celular do Workday com base no consentimento do usuário para uso público?

* Acesse a folha “Provisionamento” do Aplicativo de Provisionamento do Workday.
* Clique em Mapeamentos de atributos 
* Em **Mapeamentos**, selecione **Sincronizar trabalhadores do Workday com o Active Directory local** (ou **Sincronizar trabalhadores do Workday com o Azure AD**).
* Na página de Mapeamentos de Atributos, role para baixo e marque a caixa “Mostrar opções avançadas”.  Clique em **Editar lista de atributos do Workday**
* Na folha que é aberta, localize o atributo “Celular” e clique na linha para que você possa editar a ![RGPD de celular](./media/workday-inbound-tutorial/mobile_gdpr.png) da **expressão da API**

* Substitua a **expressão da API** pela seguinte expressão nova, que recupera o número de celular de trabalho somente se o “Sinalizador de uso público” estiver definido como “True” no Workday.

    ```
     wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Contact_Data/wd:Phone_Data[translate(string(wd:Phone_Device_Type_Reference/@wd:Descriptor),'abcdefghijklmnopqrstuvwxyz','ABCDEFGHIJKLMNOPQRSTUVWXYZ')='MOBILE' and translate(string(wd:Usage_Data/wd:Type_Data/wd:Type_Reference/@wd:Descriptor),'abcdefghijklmnopqrstuvwxyz','ABCDEFGHIJKLMNOPQRSTUVWXYZ')='WORK' and string(wd:Usage_Data/@wd:Public)='1']/@wd:Formatted_Phone
    ```

* Salve a Lista de Atributos.
* Salve o Mapeamento de Atributos.
* Limpe o estado atual e reinicie a sincronização completa.

#### <a name="how-do-i-format-display-names-in-ad-based-on-the-users-departmentcountrycity-attributes-and-handle-regional-variances"></a>Como formatar nomes de exibição no AD com base em atributos de país/departamento/cidade do usuário e tratar as variações regionais?

É um requisito comum configurar o atributo *displayName* no AD para que ele também forneça informações sobre o departamento e o país do usuário. Por exemplo, se John Smith trabalha no Departamento de Marketing nos Estados Unidos, você poderá querer que o seu *displayName* apareça como *Smith, John (Marketing-EUA)*.

Aqui está como você pode lidar com tais requisitos para construir *CN* ou *displayName* para incluir atributos como empresa, unidade de negócio, cidade ou país.

* Cada atributo do Workday é recuperado usando uma expressão de API XPATH subjacente, que pode ser configurada em **Mapeamento de Atributos -> Seção Avançada -> Editar lista de atributos para Workday**. Aqui está a expressão de API padrão do XPATH para os atributos *PreferredFirstName*, *PreferredLastName*, *empresa* e *SupervisoryOrganization* do Workday.

     [!div class="mx-tdCol2BreakAll"]
     | Atributo do Workday | Expressão XPATH da API |
     | ----------------- | -------------------- |
     | PreferredFirstName | wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Name_Data/wd:Preferred_Name_Data/wd:Name_Detail_Data/wd:First_Name/text() |
     | PreferredLastName | wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Name_Data/wd:Preferred_Name_Data/wd:Name_Detail_Data/wd:Last_Name/text() |
     | Empresa | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data[wd:Organization_Data/wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Company']/wd:Organization_Reference/@wd:Descriptor |
     | SupervisoryOrganization | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data/wd:Organization_Data[wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Supervisory']/wd:Organization_Name/text() |
  
   Confirme com sua equipe do Workday se a expressão de API acima é válida para a sua configuração de locatário do Workday. Se necessário, você pode editar conforme descrito na seção [Personalizar a lista de atributos de usuário do Workday](#customizing-the-list-of-workday-user-attributes).

* Da mesma forma as informações de país presentes no Workday são recuperadas usando o XPATH a seguir: *wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference*

     Há 5 atributos relacionados ao país que estão disponíveis na seção de lista de atributos do Workday.

     | Atributo do Workday | Expressão XPATH da API |
     | ----------------- | -------------------- |
     | CountryReference | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Alpha-3_Code']/text() |
     | CountryReferenceFriendly | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/@wd:Descriptor |
     | CountryReferenceNumeric | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Numeric-3_Code']/text() |
     | CountryReferenceTwoLetter | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Alpha-2_Code']/text() |
     | CountryRegionReference | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Region_Reference/@wd:Descriptor |

  Confirme com sua equipe do Workday se as expressões de API acima são válidas para a sua configuração de locatário do Workday. Se necessário, você pode editar conforme descrito na seção [Personalizar a lista de atributos de usuário do Workday](#customizing-the-list-of-workday-user-attributes).

* Para criar a expressão de mapeamento de atributos correta, identifique qual atributo do Workday representa “com autoridade” o primeiro nome, sobrenome, país e departamento do usuário. Digamos que os atributos são *PreferredFirstName*, *PreferredLastName*, *CountryReferenceTwoLetter* e *SupervisoryOrganization* respectivamente. Você pode usar isso para criar uma expressão para o atributo *displayName* conforme a seguir para obter um nome de exibição, como *Smith, John (Marketing-EUA)*.

    ```
     Append(Join(", ",[PreferredLastName],[PreferredFirstName]), Join(""," (",[SupervisoryOrganization],"-",[CountryReferenceTwoLetter],")"))
    ```
    Após conseguir a expressão correta, edite a tabela de Mapeamentos de Atributos e modifique o mapeamento do atributo *displayName*, conforme mostrado abaixo:   ![Mapeamento de DisplayName](./media/workday-inbound-tutorial/wd_displayname_map.png)

* Estendendo o exemplo acima, vamos dizer que você gostaria de converter nomes de cidades provenientes do Workday em valores de abreviação e, em seguida, usá-los para criar nomes de exibição, como *Smith, John (Nova York)* ou *Ninguém, Zé (NYC)*, em seguida, esse resultado pode ser obtido usando uma expressão Alternar com o atributo *Município* do Workday como a variável determinante.

     ```
    Switch
    (
      [Municipality],
      Join(", ", [PreferredLastName], [PreferredFirstName]),  
           "Chicago", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(CHI)"),
           "New York", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(NYC)"),
           "Phoenix", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(PHX)")
    )
     ```
    Consulte também:
  * [Sintaxe da função Switch](../manage-apps/functions-for-customizing-application-data.md#switch)
  * [Sintaxe da função Join](../manage-apps/functions-for-customizing-application-data.md#join)
  * [Sintaxe da função Append](../manage-apps/functions-for-customizing-application-data.md#append)

#### <a name="how-can-i-use-selectuniquevalue-to-generate-unique-values-for-samaccountname-attribute"></a>Como usar SelectUniqueValue para gerar valores exclusivos para o atributo samAccountName?

Digamos que você deseja gerar valores exclusivos para o atributo *samAccountName* usando uma combinação de atributos *FirstName* e *LastName* do Workday. A seguir está uma expressão com a qual você pode começar:

```
SelectUniqueValue(
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,1), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,2), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,3), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
)
```

Como funciona a expressão acima: Se o usuário for John Smith, ela primeiro tenta gerar JSmith, se já existir JSmith, em seguida, ela gera JoSmith, se também já existir, ela gera JohSmith. A expressão também garante que o valor gerado atende à restrição de comprimento e à restrição de caracteres especiais associados a *samAccountName*.

Consulte também:

* [Sintaxe da função Mid](../manage-apps/functions-for-customizing-application-data.md#mid)
* [Sintaxe da função Replace](../manage-apps/functions-for-customizing-application-data.md#replace)
* [Sintaxe da função SelectUniqueValue](../manage-apps/functions-for-customizing-application-data.md#selectuniquevalue)

#### <a name="how-do-i-remove-characters-with-diacritics-and-convert-them-into-normal-english-alphabets"></a>Como remover caracteres com diacríticos e convertê-los em letras normais do alfabeto em português?

Use a função [NormalizeDiacritics](../manage-apps/functions-for-customizing-application-data.md#normalizediacritics) para remover caracteres especiais no nome e sobrenome do usuário, ao construir o valor do endereço de email ou CN para o usuário.

## <a name="troubleshooting-tips"></a>Dicas de solução de problemas

Esta seção fornece diretrizes específicas sobre como solucionar problemas de provisionamento com a integração do Workday usando os logs de auditoria do Azure AD e os logs do Visualizador de eventos do Windows Server. Ela complementa as etapas de solução de problemas genéricas e os conceitos mostrados no [Tutorial: Relatórios sobre o provisionamento automático de conta de usuário](../manage-apps/check-status-user-account-provisioning.md)

Esta seção cobre os seguintes aspectos de solução de problemas:

* [Configurar o Visualizador de Eventos do Windows para solucionar problemas do agente](#setting-up-windows-event-viewer-for-agent-troubleshooting)
* [Configurar os logs de auditoria do portal do Azure para solucionar problemas de serviço](#setting-up-azure-portal-audit-logs-for-service-troubleshooting)
* [Entender logs para operações de criação da conta de usuário do AD](#understanding-logs-for-ad-user-account-create-operations)
* [Entender logs para operações de atualização do gerente](#understanding-logs-for-manager-update-operations)
* [Resolver erros comumente encontrados](#resolving-commonly-encountered-errors)

### <a name="setting-up-windows-event-viewer-for-agent-troubleshooting"></a>Configurar o Visualizador de Eventos do Windows para solucionar problemas do agente

* Faça logon no Windows Server em que o Agente de Provisionamento está implantado
* Abra o aplicativo da área de trabalho **Visualizador de Eventos do Windows Server**.
* Selecione **Logs do Windows > Aplicativo**.
* Use a opção **Filtrar log atual...** para exibir todos os eventos registrados na fonte **AAD. Connect.ProvisioningAgent** e excluir eventos com a ID de evento “5”, especificando o filtro “-5”, conforme mostrado abaixo.

  ![Visualizador de Eventos do Windows](media/workday-inbound-tutorial/wd_event_viewer_01.png))

* Clique em **OK** e classifique a exibição dos resultados por colunas de **Data e Hora**.

### <a name="setting-up-azure-portal-audit-logs-for-service-troubleshooting"></a>Configurar os logs de auditoria do portal do Azure para solucionar problemas de serviço

* Inicie o [Portal do Azure](https://portal.azure.com) e navegue até a seção **Logs de auditoria** do aplicativo de provisionamento do Workday.
* Use o botão **Colunas** na página de Logs de auditoria para exibir apenas as colunas a seguir no modo de exibição (Data, Atividade, Status, Motivo do Status). Essa configuração garante que você se concentre apenas nos dados que são relevantes para a solução de problemas.

  ![Colunas do log de auditoria](media/workday-inbound-tutorial/wd_audit_logs_00.png)

* Use os parâmetros de consulta **Destino** e **Intervalo de datas** para filtrar a exibição. 
  * Defina o parâmetro de consulta **Destino** para o “ID de Trabalho” ou “ID do Funcionário” do objeto de trabalho do Workday.
  * Defina o **Intervalo de datas** como um período de tempo apropriado ao longo do qual você deseja investigar erros ou problemas com o provisionamento.

  ![Filtros de log de auditoria](media/workday-inbound-tutorial/wd_audit_logs_01.png)

### <a name="understanding-logs-for-ad-user-account-create-operations"></a>Entender logs para operações de criação da conta de usuário do AD

Quando uma nova contratação no Workday é detectada (digamos com ID de Funcionário *21023*), o serviço de provisionamento do Azure AD tenta criar uma nova conta de usuário do AD para o trabalhador e, durante esse processo, cria 4 registros de log de auditoria conforme descrito abaixo:

  [![Log de auditoria criar ops](media/workday-inbound-tutorial/wd_audit_logs_02.png)](media/workday-inbound-tutorial/wd_audit_logs_02.png#lightbox)

Quando você clica em qualquer um dos registros de log de auditoria, a página **Detalhes da atividade** abre. Aqui está o que a página **Detalhes da atividade** exibe para cada tipo de registro de log.

* Registro de **Importação do Workday**: Este registro de log exibe as informações do trabalhador obtidas no Workday. Utilize as informações na seção *Detalhes adicionais* do registro de log para solucionar problemas com a busca de dados no Workday. Um registro de exemplo é mostrado abaixo, junto com os ponteiros para interpretar cada campo.

  ```JSON
  ErrorCode : None  // Use the error code captured here to troubleshoot Workday issues
  EventName : EntryImportAdd // For full sync, value is "EntryImportAdd" and for delta sync, value is "EntryImport"
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID (usually the Worker ID or Employee ID field)
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the record
  ```

* Registro de **Importação do AD**: Este registro de log exibe informações da conta buscadas no AD. Como não há nenhuma conta de AD durante a criação inicial do usuário o *Motivo do status de atividade* indicará que nenhuma conta com o valor do atributo de ID de Correspondência foi encontrada no Active Directory. Utilize as informações na seção *Detalhes adicionais* do registro de log para solucionar problemas com a busca de dados no Workday. Um registro de exemplo é mostrado abaixo, junto com os ponteiros para interpretar cada campo.

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot Workday issues
  EventName : EntryImportObjectNotFound // Implies that object was not found in AD
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  ```

  Para localizar registros de log do Agente de Provisionamento correspondentes a essa operação de importação do AD, abra os logs do Visualizador de Eventos do Windows e use a opção **Localizar...** do menu para localizar entradas de log que contém o valor do atributo de ID de correspondência/Propriedade Joining (neste caso, *21023*).

  ![Localizar](media/workday-inbound-tutorial/wd_event_viewer_02.png)

  Procure a entrada com a *ID de Evento = 9*, que fornecerá a você o filtro de pesquisa LDAP usado pelo agente para recuperar a conta do AD. Você pode verificar se esse é o filtro de pesquisa correto para recuperar entradas de usuário exclusivas.

  ![Pesquisa LDAP](media/workday-inbound-tutorial/wd_event_viewer_03.png)

  O registro que vem imediatamente a seguir com *ID de Evento = 2* captura o resultado da operação de pesquisa e se ela retornou algum resultado.

  ![Resultados de LDAP](media/workday-inbound-tutorial/wd_event_viewer_04.png)

* Registro de **ação de regra de sincronização**: Este registro de log exibe os resultados das regras de mapeamento de atributo e dos filtros de escopo configurados junto com a ação de provisionamento que serão usados para processar o evento de entrada do Workday. Utilize as informações na seção *Detalhes adicionais* do registro de log para solucionar problemas com a ação de sincronização. Um registro de exemplo é mostrado abaixo, junto com os ponteiros para interpretar cada campo.

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot sync issues
  EventName : EntrySynchronizationAdd // Implies that the object will be added
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the profile in Workday
  ```

  Se houver problemas com as expressões de mapeamento de atributos ou os dados de entrada do Workday apresentarem problemas (por exemplo: valor vazio ou nulo para os atributos necessários), você observará uma falha nesta fase com o código de erro fornecendo os detalhes da falha.

* Registro de **Exportação do AD**: Este registro de log exibe o resultado da operação de criação de conta do AD junto com os valores de atributo que foram definidas no processo. Utilize as informações na seção *Detalhes adicionais* do registro de log para solucionar problemas com a ação de criação de conta. Um registro de exemplo é mostrado abaixo, junto com os ponteiros para interpretar cada campo. Na seção “Detalhes adicionais”, “EventName” é definido como “EntryExportAdd”, “JoiningProperty” é definido como o valor do atributo ID de Correspondência, o “SourceAnchor” está definido para o WorkdayID (WID) associado ao registro e “TargetAnchor” é definido como o valor do atributo “ObjectGuid” do AD do usuário recém-criado. 

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot AD account creation issues
  EventName : EntryExportAdd // Implies that object will be created
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the profile in Workday
  TargetAnchor : 83f0156c-3222-407e-939c-56677831d525 // set to the value of the AD "objectGuid" attribute of the new user
  ```

  Para localizar registros de log do Agente de Provisionamento correspondentes a essa operação de exportação do AD, abra os logs do Visualizador de Eventos do Windows e use a opção **Localizar...** do menu para localizar entradas de log que contém o valor do atributo de ID de correspondência/Propriedade Joining (neste caso, *21023*).  

  Procure um registro de HTTP POST correspondente ao carimbo de data/hora da operação de exportação com *ID de Evento = 2*. Esse registro contém os valores de atributo enviados pelo serviço de provisionamento para o agente de provisionamento.

  [![Adicionar SCIM](media/workday-inbound-tutorial/wd_event_viewer_05.png)](media/workday-inbound-tutorial/wd_event_viewer_05.png#lightbox)

  Imediatamente após o evento acima, deve haver outro evento que captura a resposta da operação de criação de conta do AD. Esse evento retorna o novo objectGuid criado no AD e é definido como o atributo TargetAnchor no serviço de provisionamento.

  [![Adicionar SCIM](media/workday-inbound-tutorial/wd_event_viewer_06.png)](media/workday-inbound-tutorial/wd_event_viewer_06.png#lightbox)

### <a name="understanding-logs-for-manager-update-operations"></a>Entender logs para operações de atualização do gerente

O atributo gerente é um atributo de referência no AD. O serviço de provisionamento não define o atributo gerente como parte da operação de criação de usuário. Em vez disso, o atributo gerente é definido como parte de uma operação *atualizar* depois que a conta do AD é criada para o usuário. Expandindo o exemplo acima, digamos que uma nova contratação com ID de Funcionário “21451” é ativada no Workday e o gerente da nova contratação (*21023*) já tem uma conta do AD. Nesse cenário, pesquisar os Logs de auditoria para o usuário 21451 mostra 5 entradas.

  [![Gerenciador de atualização](media/workday-inbound-tutorial/wd_audit_logs_03.png)](media/workday-inbound-tutorial/wd_audit_logs_03.png#lightbox)

Os 4 primeiros registros são aqueles que exploramos como parte da operação de criação do usuário. O quinto registro é a exportação associada à atualização do atributo de gerente. O registro de log exibe o resultado da operação de atualização do gerente de conta do AD, que é executada usando o atributo *objectGuid* do gerente.

  ```JSON
  // Modified Properties
  Name : manager
  New Value : "83f0156c-3222-407e-939c-56677831d525" // objectGuid of the user 21023

  // Additional Details
  ErrorCode : None // Use the error code captured here to troubleshoot AD account creation issues
  EventName : EntryExportUpdate // Implies that object will be created
  JoiningProperty : 21451 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : 9603bf594b9901693f307815bf21870a // WorkdayID of the user
  TargetAnchor : 43b668e7-1d73-401c-a00a-fed14d31a1a8 // objectGuid of the user 21451

  ```

### <a name="resolving-commonly-encountered-errors"></a>Resolver erros comumente encontrados

Esta seção aborda os erros comuns com o provisionamento de usuário do Workday e como resolvê-los. Os erros são agrupados conforme indicado a seguir:

* [Erros do agente de provisionamento](#provisioning-agent-errors)
* [Erros de conectividade](#connectivity-errors)
* [Erros de criação de conta de usuário do AD](#ad-user-account-creation-errors)
* [Erros de atualização de conta de usuário do AD](#ad-user-account-update-errors)

#### <a name="provisioning-agent-errors"></a>Erros do agente de provisionamento

|#|Cenário de erro |Causas prováveis|Resolução recomendada|
|--|---|---|---|
|1.| Erro ao instalar o agente de provisionamento com a mensagem de erro:  *Falha ao iniciar o serviço “Agente de Provisionamento do Microsoft Azure AD Connect” (AADConnectProvisioningAgent). Verifique se você tem privilégios suficientes para iniciar o sistema.* | Esse erro normalmente aparece se você está tentando instalar o agente de provisionamento em um controlador de domínio e a política de grupo impede que o serviço seja iniciado.  Ele também é visto se você tiver uma versão anterior do agente em execução e não o tiver desinstalado antes de iniciar uma nova instalação.| Instale o agente de provisionamento em um servidor que não seja um controlador de domínio. Certifique-se de que as versões anteriores do agente são desinstaladas antes de instalar o novo agente.|
|2.| O serviço do Windows “Agente de Provisionamento do Microsoft Azure AD Connect” está no estado *Iniciando* e não muda para o estado *Em execução*. | Como parte da instalação, o assistente de agente cria uma conta local (**NT Service\\AADConnectProvisioningAgent**) no servidor e essa é a conta de **Logon** usada para iniciar o serviço. Se uma política de segurança em seu servidor Windows impede as contas locais de executarem os serviços, você encontrará esse erro. | Abra o *console de Serviços*. Clique com botão direito no serviço do Windows “Agente de Provisionamento do Microsoft Azure AD Connect” e, na guia Logon, especifique a conta de administrador de domínio para executar o serviço. Reinicie o serviço. |
|3.| Ao configurar o agente de provisionamento com seu domínio do AD na etapa *Conectar o Active Directory*, o assistente leva muito tempo tentando carregar o esquema do AD e eventualmente expira. | Esse erro normalmente aparece se o assistente não puder entrar em contato com o servidor de controlador de domínio do AD devido a problemas de firewall. | Na tela do assistente *Conectar o Active Directory*, além de fornecer as credenciais para o seu domínio do AD, há uma opção chamada *Selecionar prioridade do controlador de domínio*. Use esta opção para selecionar um controlador de domínio que esteja no mesmo site que o servidor do agente e certifique-se de que não há nenhuma regra de firewall bloqueando a comunicação. |

#### <a name="connectivity-errors"></a>Erros de conectividade

Se o serviço de provisionamento não conseguir se conectar ao Workday ou ao Active Directory, isso pode fazer com que o provisionamento entre em um estado de quarentena. Use a tabela abaixo para solucionar problemas de conectividade.

|#|Cenário de erro |Causas prováveis|Resolução recomendada|
|--|---|---|---|
|1.| Ao clicar em **Testar conexão** você receberá a mensagem de erro: *Ocorreu um erro ao conectar-se ao Active Directory. Verifique se o Agente de Provisionamento local está sendo executado e está configurado com o domínio correto do Active Directory.* | Esse erro geralmente mostra se o agente de provisionamento não está em execução ou se há um firewall bloqueando a comunicação entre o Azure AD e o agente de provisionamento. Você também poderá ver esse erro se o domínio não estiver configurado no assistente do agente. | Abra o console *Serviços* console no servidor do Windows para confirmar se o agente está em execução. Abra o assistente do agente de provisionamento e confirme se o domínio correto está registrado com o agente.  |
|2.| O trabalho de provisionamento entrará em um estado de quarentena nos finais de semana (Sex - Sáb) e receberemos uma notificação por email que há um erro com a sincronização. | Uma das causas comuns desse erro é o tempo de inatividade planejado do Workday. Se você estiver usando um locatário de implementação do Workday, observe que o Workday tem tempo de inatividade programado para seus locatários de implementação nos finais de semana (geralmente da noite da sexta-feira até a manhã de sábado) e, durante esse período, os aplicativos de provisionamento do Workday podem entrar em um estado de quarentena, pois eles não conseguem conectar-se ao Workday. Eles voltam ao estado normal quando o locatário de implementação do Workday estiver online novamente. Em casos raros, você também poderá ver esse erro, se a senha do usuário do sistema de integração foi alterada devido à atualização do locatário ou se a conta estiver no estado bloqueado ou expirado. | Verifique com seu parceiro de integração ou administrador do Workday para ver quando o agendamento do tempo de inatividade do Workday para ignorar mensagens de alerta durante o período de tempo de inatividade e confirmar a disponibilidade quando a instância do Workday estiver online novamente.  |


#### <a name="ad-user-account-creation-errors"></a>Erros de criação de conta de usuário do AD

|#|Cenário de erro |Causas prováveis|Resolução recomendada|
|--|---|---|---|
|1.| Exportar falhas em operações no log de auditoria com a mensagem *Erro: OperationsError-SvcErr: Erro de operação. Nenhuma referência superior foi configurada para o serviço de diretório. Portanto, o serviço de diretório não pode emitir referências a objetos fora desta floresta.* | Esse erro geralmente mostra se o *contêiner do Active Directory* UO não está definido corretamente ou se há problemas com o Mapeamento da Expressão usado para *parentDistinguishedName*. | Verifique o parâmetro do *Contêiner do Active Directory* UO para erros de digitação. Se você estiver usando *parentDistinguishedName* no mapeamento de atributos certifique-se de que ele sempre é avaliado como um contêiner conhecido dentro do domínio do AD. Verifique o evento *Exportar* nos logs de auditoria para ver o valor gerado. |
|2.| Exportar falhas em operações no log de auditoria com o código de erro: *SystemForCrossDomainIdentityManagementBadResponse* e a mensagem *Erro: ConstraintViolation-AtrErr: O valor na solicitação é inválido. Um valor para o atributo não estava no intervalo de valores aceitável. Detalhes de \nError: CONSTRAINT_ATT_TYPE - company*. | Embora esse erro seja específico para o atributo *empresa*, você poderá ver esse erro também para outros atributos como *CN*, por exemplo. Este erro aparece devido à restrição de esquema do AD imposta. Por padrão, os atributos *empresa* e *CN* no AD têm um limite máximo de 64 caracteres. Se o valor proveniente do Workday tiver mais que 64 caracteres, então você verá essa mensagem de erro. | Verifique o evento *Exportar* nos logs de auditoria para ver o valor do atributo relatado na mensagem de erro. Considere a possibilidade de truncar o valor proveniente do Workday usando a função [Mid](../manage-apps/functions-for-customizing-application-data.md#mid) ou alterar os mapeamentos para um atributo do AD que não tenha essas mesmas restrições de tamanho.  |

#### <a name="ad-user-account-update-errors"></a>Erros de atualização de conta de usuário do AD

Durante o processo de atualização de conta de usuário do AD, o serviço de provisionamento lê as informações do Workday e do AD, executa as regras de mapeamento de atributos e determina se é necessário realizar alguma alteração. Da mesma forma, um evento de atualização é acionado. Se ocorrer uma falha em qualquer uma dessas etapas, ela é registrada nos logs de auditoria. Use a tabela abaixo para solucionar problemas comuns com erros de atualização.

|#|Cenário de erro |Causas prováveis|Resolução recomendada|
|--|---|---|---|
|1.| Falhas de ação da regra de sincronização no log de auditoria com a mensagem *EventName = EntrySynchronizationError e ErrorCode = EndpointUnavailable*. | Esse erro aparecerá se o serviço de provisionamento não consegue recuperar os dados de perfil do usuário do Active Directory devido a um erro de processamento encontrado pelo agente de provisionamento local. | Verifique os logs do Visualizador de Eventos do Agente de Provisionamento para eventos de erro que indicam problemas com a operação de leitura (Filtrar por ID de Evento 2). |
|2.| O atributo de gerente no AD não será atualizado para determinados usuários no AD. | A causa mais provável desse erro é se você estiver usando regras de escopo e o gerente do usuário não é parte do escopo. Você também pode encontrar esse problema se o atributo de ID de correspondência do gerente (por exemplo, EmployeeID) não for encontrado no domínio do AD de destino ou não estiver definido com o valor correto. | Revise o filtro de escopo e adicione o usuário gerente no escopo. Verifique o perfil de gerente do AD para certificar-se de que há um valor para o atributo de ID de correspondência. |

## <a name="managing-your-configuration"></a>Gerenciar sua configuração

Esta seção descreve como você pode aproveitar, personalizar e gerenciar ainda mais sua configuração de provisionamento de usuário no Workday. Ela abrange os seguintes tópicos:

* [Personalizando a lista de atributos de usuário do Workday](#customizing-the-list-of-workday-user-attributes)  
* [Exportar e importar sua configuração](#exporting-and-importing-your-configuration)

### <a name="customizing-the-list-of-workday-user-attributes"></a>Personalizando a lista de atributos de usuário do Workday

O provisionamento de aplicativos do Workday para o Active Directory e o Azure AD incluem uma lista padrão de atributos de usuário do Workday da qual você pode selecionar. No entanto, essas listas não são abrangentes. O Workday é compatível com centenas de possibilidades de atributos de usuário, que podem ser padrão ou exclusivos para seu locatário do Workday.

O serviço de provisionamento do Azure AD é compatível com possibilidade de personalizar sua lista ou seu atributo do Workday a fim de incluir todos os atributos expostos na operação [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) da API de Recursos Humanos.

Para fazer essa alteração, você deve usar o [Workday Studio](https://community.workday.com/studio-download) para extrair as expressões XPath que representam os atributos que você deseja usar e, em seguida, adicioná-los à sua configuração de provisionamento usando o editor de atributo avançado no Portal do Azure.

**Para recuperar uma expressão XPath para um atributo de usuário do Workday:**

1. Baixe e instale o [Workday Studio](https://community.workday.com/studio-download). Você precisará de uma conta da comunidade do Workday para acessar o instalador.

2. Baixe o arquivo WSDL Human_Resources do Workday desta URL: https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Human_Resources.wsdl

3. Inicie o Workday Studio.

4. Na barra de comandos, selecione a opção **Workday > Testar o serviço Web no Testador**.

5. Selecione **Externo** e selecione o arquivo WSDL Human_Resources que você baixou na etapa 2.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio1.png)

6. Defina o campo **local** como `https://IMPL-CC.workday.com/ccx/service/TENANT/Human_Resources`, mas substituindo "IMPL-CC" pelo tipo de instância real e "LOCATÁRIO" pelo seu nome do locatário real.

7. Defina **Operação** como **Get_Workers**

8.  Clique no pequeno link **configurar** abaixo dos painéis de Solicitação/Resposta para definir suas credenciais do Workday. Marque a opção **Autenticação** e, em seguida, insira o nome de usuário e senha da sua conta do sistema de integração do Workday. Certifique-se de que o nome de usuário como nome de formato\@do locatário e, em seguida, deixe o **WS-Security UsernameToken** opção selecionada.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio2.png)

9. Selecione **OK**.

10. No painel **Solicitação**, cole o XML abaixo e defina **Employee_ID** como a ID de funcionário de um usuário real em seu locatário do Workday. Selecione um usuário que tenha o atributo que você deseja extrair preenchido.

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <env:Envelope xmlns:env="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="https://www.w3.org/2001/XMLSchema">
      <env:Body>
        <wd:Get_Workers_Request xmlns:wd="urn:com.workday/bsvc" wd:version="v21.1">
          <wd:Request_References wd:Skip_Non_Existing_Instances="true">
            <wd:Worker_Reference>
              <wd:ID wd:type="Employee_ID">21008</wd:ID>
            </wd:Worker_Reference>
          </wd:Request_References>
          <wd:Response_Group>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Personal_Information>true</wd:Include_Personal_Information>
            <wd:Include_Employment_Information>true</wd:Include_Employment_Information>
            <wd:Include_Management_Chain_Data>true</wd:Include_Management_Chain_Data>
            <wd:Include_Organizations>true</wd:Include_Organizations>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Transaction_Log_Data>true</wd:Include_Transaction_Log_Data>
            <wd:Include_Photo>true</wd:Include_Photo>
            <wd:Include_User_Account>true</wd:Include_User_Account>
          <wd:Include_Roles>true</wd:Include_Roles>
          </wd:Response_Group>
        </wd:Get_Workers_Request>
      </env:Body>
    </env:Envelope>
    ```

11. Clique em **Enviar Solicitação** (seta verde) para executar o comando. Se for bem-sucedido, a resposta deverá aparecer no painel **Resposta**. Verifique a resposta para garantir que ela tenha os dados da ID de usuário que você inseriu e não um erro.

12. Se for bem-sucedido, copie o XML do painel **Resposta** e salve-o como um arquivo XML.

13. Na barra de comandos do Workday Studio, selecione **File> Open File ...** e abra o arquivo XML que você salvou. Essa ação abrirá o arquivo no editor XML do Workday Studio.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio3.png)

14. Na árvore de arquivo, navegue por **/env: Envelope > env: Body > wd:Get_Workers_Response > wd:Response_Data > wd: Trabalhador** para localizar os dados do seu usuário.

15. Em **wd: Trabalhador**, localize o atributo que você quer adicionar e selecione-o.

16. Copie a expressão XPath para o atributo que você selecionou no campo **Caminho de Documento**.

17. Remova o prefixo **/env:Envelope/env:Body/wd:Get_Workers_Response/wd:Response_Data/** da expressão copiada.

18. Se o último item na expressão copiada for um nó (exemplo: "/wd: Birth_Date"), então acrescente **/text()** no final da expressão. Isso não é necessário se o último item for um atributo (exemplo: "/@wd: type").

19. O resultado deve ser algo como `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`. Esse valor é o que você copiará no Portal do Azure.

**Para adicionar o atributo de usuário personalizado do Workday à sua configuração de provisionamento:**

1. Inicie o [Portal do Azure](https://portal.azure.com) e navegue até a seção Provisionamento do aplicativo de provisionamento do Workday, conforme descrito anteriormente neste tutorial.

2. Defina o **Status de Provisionamento** como **Desativado** e selecione **Salvar**. Essa etapa ajudará a garantir que as alterações entrem em vigor somente quando você estiver pronto.

3. Em **Mapeamentos**, selecione **Sincronizar trabalhadores do Workday com o Active Directory local** (ou **Sincronizar trabalhadores do Workday com o Azure AD**).

4. Role até a parte inferior da próxima tela e selecione **Mostrar opções avançadas**.

5. Selecione **Editar lista de atributos do Workday**.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio_aad1.png)

6. Role até a parte inferior da lista de atributos, até o local em que estão os campos de entrada.

7. Em **Nome**, insira um nome de exibição para o atributo.

8. Em **Tipo**, selecione o tipo que corresponda adequadamente ao seu atributo (**Cadeia de Caracteres** é mais comum).

9. Em **Expressão API**, insira a expressão XPath que você copiou do Workday Studio. Exemplo: `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`

10. Selecione **Adicionar Atributo**.

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio_aad2.png)

11. Selecione **Salvar** acima e, em seguida, **Sim**, na caixa de diálogo. Feche a tela Mapeamento de Atributo, se ainda estiver aberta.

12. Na guia **Provisionamento** principal, selecione **Sincronizar trabalhos do Workday com o Active Directory local** (ou **Sincronizar trabalhos com o Azure AD**) novamente.

13. Selecione **Adicionar novo mapeamento**.

14. Agora, o novo atributo deve aparecer na lista **Atributo de origem**.

15. Adicione um mapeamento para o novo atributo conforme desejado.

16. Quando terminar, lembre-se de definir novamente o **Status de Provisionamento** como **Ativado** e salvar.

### <a name="exporting-and-importing-your-configuration"></a>Exportar e importar sua configuração

Esta seção descreve como usar a API do Microsoft Graph e o Explorador do Graph para exportar seus mapeamentos de atributos e o esquema de provisionamento do Workday para um arquivo JSON e importá-los novamente para o Azure AD.

#### <a name="step-1-retrieve-your-workday-provisioning-app-service-principal-id-object-id"></a>Etapa 1: Recuperar sua ID de Entidade de Serviço de Aplicativo de provisionamento do Workday (ID de Objeto)

1. Inicie o [Portal do Azure](https://portal.azure.com) e navegue até a seção Propriedades do aplicativo de provisionamento do Workday.
1. Na seção Propriedades do seu aplicativo de provisionamento, copie o valor GUID associado ao campo *ID de Objeto*. Esse valor também é chamado de **ServicePrincipalId** do seu aplicativo e ele será usado em operações do Graph Explorer.

   ![ID da Entidade de Serviço de Aplicativo do Workday](./media/workday-inbound-tutorial/wd_export_01.png)

#### <a name="step-2-sign-into-microsoft-graph-explorer"></a>Etapa 2: Entrar no Microsoft Graph Explorer

1. Abrir o [Microsoft Graph Explorer](https://developer.microsoft.com/graph/graph-explorer)
1. Clique no botão “Entrar com a Microsoft” e entre usando as credenciais de administrador do aplicativo ou de administrador global do Azure AD.

    ![Entrar no Graph](./media/workday-inbound-tutorial/wd_export_02.png)

1. Após entrar com êxito, você verá os detalhes da conta de usuário no painel esquerdo.

#### <a name="step-3-retrieve-the-provisioning-job-id-of-the-workday-provisioning-app"></a>Etapa 3: Recuperar a ID de trabalho de provisionamento do aplicativo de provisionamento do Workday

No Microsoft Graph Explorer, execute a seguinte consulta GET, substituindo [servicePrincipalId] pelo **ServicePrincipalId** extraído da [Etapa 1](#step-1-retrieve-your-workday-provisioning-app-service-principal-id-object-id).

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs
```

Você obterá uma resposta, conforme mostrado abaixo. Copie o “atributo de id” presente na resposta. Esse valor é o **ProvisioningJobId** e será usado para recuperar os metadados de esquema subjacentes.

   [![Id do trabalho de provisionamento](./media/workday-inbound-tutorial/wd_export_03.png)](./media/workday-inbound-tutorial/wd_export_03.png#lightbox)

#### <a name="step-4-download-the-provisioning-schema"></a>Etapa 4: Baixar o esquema de provisionamento

No Microsoft Graph Explorer, execute a seguinte consulta GET, substituindo [servicePrincipalId] e [ProvisioningJobId] pelo ServicePrincipalId e o ProvisioningJobId recuperados nas etapas anteriores.

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

Copie o objeto JSON da resposta e salve-o em um arquivo para criar um backup do esquema.

#### <a name="step-5-import-the-provisioning-schema"></a>Etapa 5: Importar o esquema de provisionamento

> [!CAUTION]
> Execute esta etapa somente se você precisar modificar o esquema de configuração que não pode ser alterado usando o portal do Azure ou se precisar restaurar a configuração de um arquivo de backup anterior com um esquema válido e funcional.

No Microsoft Graph Explorer, configure a seguinte consulta PUT, substituindo [servicePrincipalId] e [ProvisioningJobId] pelo ServicePrincipalId e o ProvisioningJobId recuperados nas etapas anteriores.

```http
    PUT https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

Na guia “Corpo da solicitação”, copie o conteúdo do arquivo de esquema JSON.

   [![Corpo da solicitação](./media/workday-inbound-tutorial/wd_export_04.png)](./media/workday-inbound-tutorial/wd_export_04.png#lightbox)

Na guia “Cabeçalhos de solicitação”, adicione o atributo de cabeçalho de Content-Type com o valor “application/json”

   [![Cabeçalhos de solicitação](./media/workday-inbound-tutorial/wd_export_05.png)](./media/workday-inbound-tutorial/wd_export_05.png#lightbox)

Clique no botão “Executar consulta” para importar o novo esquema.

## <a name="managing-personal-data"></a>Gerenciando dados pessoais

A solução de provisionamento do Workday para o Active Directory exige que um agente de sincronização seja instalado em um servidor local do Windows, e esse agente cria logs no log de eventos do Windows podendo conter dados pessoais dependendo dos seus mapeamentos de atributos do Workday para o AD. Para cumprir as obrigações de privacidade do usuário, você pode garantir que nenhum dado seja retido nos logs de eventos por mais de 48 horas, configurando uma tarefa agendada do Windows para limpar o log de eventos.

O serviço de provisionamento do Azure AD se encaixa na categoria **processador de dados** da classificação do RGPD. Como um pipeline de processador de dados, o serviço fornece serviços de processamento de dados para parceiros-chave e consumidores finais. O serviço de provisionamento do Azure AD não gera dados de usuário e não possui controle independente com relação aos dados pessoais coletados e como são utilizados. A recuperação de dados, agregação, análise e relatórios no Azure AD são baseados em dados empresariais existentes.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-hybrid-note.md)]

Em relação à retenção de dados, o serviço de provisionamento do Azure AD não gera relatórios, executa análise ou fornece informações por mais de 30 dias. Portanto, o serviço de provisionamento do Azure AD não armazena, processa ou retém quaisquer dados por mais de 30 dias. Este design está em conformidade com os regulamentos do GDPR, com os regulamentos de conformidade de privacidade da Microsoft e com as políticas de retenção de dados do Azure Active Directory.

## <a name="next-steps"></a>Próximas etapas

* [Saiba como fazer revisão de logs e obter relatórios sobre atividade de provisionamento](../manage-apps/check-status-user-account-provisioning.md)
* [Saiba como configurar o logon único entre o Workday e o Azure Active Directory](workday-tutorial.md)
* [Saiba como integrar outros aplicativos SaaS com o Azure Active Directory](tutorial-list.md)
* [Saiba como usar APIs do Microsoft Graph para gerenciar configurações de provisionamento](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview)
