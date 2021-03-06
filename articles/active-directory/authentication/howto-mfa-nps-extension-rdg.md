---
title: Integração do Gateway de área de trabalho remota com a extensão do NPS do MFA do Azure - Active Directory do Azure
description: Integrar sua infraestrutura de Gateway de Área de Trabalho Remota ao Azure MFA usando a extensão Servidor de Políticas de Rede para o Microsoft Azure
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 12/03/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 638703e4d67cbd004f0bd616ba31475f507dfd8a
ms.sourcegitcommit: 8a681ba0aaba07965a2adba84a8407282b5762b2
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/29/2019
ms.locfileid: "64873421"
---
# <a name="integrate-your-remote-desktop-gateway-infrastructure-using-the-network-policy-server-nps-extension-and-azure-ad"></a>Integrar a infraestrutura do seu Gateway de Área de Trabalho Remota usando a extensão do Servidor de Políticas de Rede (NPS) e o Azure AD

Este artigo fornece os detalhes para integração da infraestrutura do seu Gateway de Área de Trabalho Remota com a Autenticação Multifator do Azure (MFA) usando a extensão do Servidor de Políticas de Rede (NPS) para o Microsoft Azure.

A extensão do NPS (Servidor de Políticas de Rede) para o Azure permite que os clientes protejam a autenticação do cliente do serviço RADIUS usando a [MFA (Autenticação Multifator) do Azure](multi-factor-authentication.md) baseada em nuvem. Essa solução fornece uma verificação em duas etapas para adicionar uma segunda camada de segurança para logons de usuário e transações.

Este artigo fornece instruções passo a passo para integrar a infraestrutura do NPS com a Autenticação Multifator do Azure usando a extensão do NPS do Azure. Isso permite a verificação dos usuários que tentarem entrar em um Gateway de Área de Trabalho Remota.

> [!NOTE]
> Esse artigo não deve ser usado com implantações do Servidor MFA e só deve ser usado com as implantações do Azure MFA (baseado em nuvem).

A Política de Rede e Serviços de Acesso (NPS) permite que as organizações façam o seguinte:

* Defina locais centrais para o gerenciamento e controle de solicitações de rede especificando quem pode se conectar, as horas do dia durante as quais as conexões são permitidas, a duração das conexões e o nível de segurança que os clientes devem usar para se conectar e assim por diante. Em vez de especificar essas políticas em cada servidor VPN ou Gateway de Área de Trabalho Remota (RD), essas políticas podem ser especificadas uma vez em um local central. O protocolo RADIUS fornece a Autenticação, Autorização e Contabilização (AAA) centralizadas.
* Estabelecer e impor políticas de integridade do cliente de Proteção de Acesso à Rede (NAP) que determinam se os dispositivos têm acesso irrestrito ou restrito aos recursos de rede.
* Fornece um meio para impor a autenticação e autorização para acesso aos comutadores Ethernet e pontos de acesso sem fio compatíveis com 802.1x.

Normalmente, as organizações usam o NPS (RADIUS) para simplificar e centralizar o gerenciamento de políticas VPN. No entanto, muitas organizações também usam o NPS para simplificar e centralizar o gerenciamento das Políticas de Autorização de Conexão de Área de Trabalho da Área de Trabalho Remota (RD CAPs).

As organizações também podem integrar o NPS com o Azure MFA para aumentar a segurança e fornecer um alto nível de conformidade. Isso ajuda a garantir que os usuários estabeleçam a verificação em duas etapas para entrar no Gateway de Área de Trabalho Remota. Para conceder acesso aos usuários, é necessário que eles forneçam a combinação de nome de usuário e senha com as informações que o usuário tem em seu controle. Essas informações devem ser confiáveis e não facilmente duplicadas, como um número de telefone celular, o número de telefone fixo, o aplicativo em um dispositivo móvel e assim por diante. RDG atualmente dá suporte a chamada telefônica e notificações por push de métodos do aplicativo Microsoft authenticator para 2FA. Para obter mais informações sobre os métodos de autenticação com suporte, confira a seção [Determinar quais métodos de autenticação que os usuários podem usar](howto-mfa-nps-extension.md#determine-which-authentication-methods-your-users-can-use).

Antes da disponibilidade da extensão do NPS do Azure, os clientes que desejavam implementar a verificação em duas etapas para ambientes integrados de NPS e o Azure MFA tinham que configurar e manter um servidor de MFA separado no ambiente local, conforme documentado no [Gateway de Área de Trabalho Remota e Servidor de Autenticação Multifator do Azure usando RADIUS](howto-mfaserver-nps-rdg.md).

A disponibilidade da extensão do NPS para o Azure agora dá às organizações a opção de implantar uma solução MFA local ou uma solução MFA baseada em nuvem para autenticação de cliente RADIUS segura.

## <a name="authentication-flow"></a>Fluxo de autenticação

Para os usuários terem acesso aos recursos de rede por meio de um Gateway de Área de Trabalho Remota, eles devem atender às condições especificadas na Política de Autorização de Conexão de Área de Trabalho Remota (RD CAP) e uma Política de Autorização de Recursos de Área de Trabalho Remota (RD RAP). As RD CAPs especificam quem possui autorização para conectar-se a Gateways de Área de Trabalho Remota. As RD RAPs especificam os recursos de rede, como áreas de trabalho remotas ou aplicativos remotos, aos quais o usuário tem permissão para se conectar através do Gateway de Área de Trabalho Remota.

Um Gateway de Área de Trabalho Remota pode ser configurado para usar um repositório política central para RD CAPs. As RD RAPs não podem usar uma política central, pois elas são processadas no Gateway de Área de Trabalho Remota. Um exemplo de um Gateway de Área de Trabalho Remota configurada para usar um repositório de política central para RD CAPs é um cliente RADIUS para outro servidor NPS que serve como o repositório de políticas central.

Quando a extensão do NPS para o Azure é integrada com o NPS e o Gateway de Área de Trabalho Remota, um fluxo de autenticação correto funciona da seguinte maneira:

1. O servidor de Gateway de Área de Trabalho Remota recebe uma solicitação de autenticação de um usuário de área de trabalho remota para se conectar a um recurso, como uma sessão de Área de Trabalho Remota. O servidor de Gateway de Área de Trabalho Remota age como um cliente RADIUS e converte a solicitação para uma mensagem de solicitação de acesso RADIUS e envia a mensagem para o servidor RADIUS (NPS) onde a extensão do NPS está instalada.
1. A combinação de nome de usuário e senha é verificada no Active Directory e o usuário é autenticado.
1. Se todas as condições conforme especificadas na Solicitação de Conexão de NPS e as Políticas de Rede forem atendidas (por exemplo, hora do dia ou restrições de associação a um grupo), a extensão NPS dispara uma solicitação de autenticação secundária com o Azure MFA.
1. O Azure MFA comunica-se com o Azure AD, recupera os detalhes do usuário e executa a autenticação secundária usando métodos com suporte.
1. Após o desafio de MFA ter sido concluído com sucesso, o Azure MFA comunica o resultado para a extensão do NPS.
1. O servidor NPS, no qual a extensão está instalada, envia uma mensagem Access-Accept do RADIUS para a política RD CAP para o servidor de Gateway de Área de Trabalho Remota.
1. O usuário tem acesso ao recurso de rede solicitado por meio do Gateway de Área de Trabalho Remota.

## <a name="prerequisites"></a>Pré-requisitos

Esta seção fornece detalhes sobre os pré-requisitos necessários para integrar o Azure MFA com o Gateway de Área de Trabalho Remota. Antes de iniciar, você deverá ter os seguintes pré-requisitos já preparados.  

* Infraestrutura de Serviços de Área de Trabalho Remota (RDS)
* Licença do Azure MFA
* Software do Windows Server
* Função de Serviços de Acesso (NPS) e Política de Rede
* Azure Active Directory sincronizado com o Active Directory local
* ID do GUID do Azure Active Directory

### <a name="remote-desktop-services-rds-infrastructure"></a>Infraestrutura de Serviços de Área de Trabalho Remota (RDS)

Você deve ter uma infraestrutura de Serviços de Área de Trabalho Remota (RDS) em vigor. Se você não fizer isso, em seguida, você pode criar rapidamente essa infraestrutura no Azure usando o seguinte modelo de início rápido: [Criar implantação de coleção de sessão de área de trabalho remota](https://github.com/Azure/azure-quickstart-templates/tree/ad20c78b36d8e1246f96bb0e7a8741db481f957f/rds-deployment).

Se você quiser criar manualmente uma infraestrutura de RDS local rapidamente para fins de teste, siga as etapas para implantar uma.
**Saiba mais**: [Implantar RDS com início rápido do Azure](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-in-azure) e [implantação de infraestrutura de RDS básica](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-deploy-infrastructure).

### <a name="azure-mfa-license"></a>Licença do Azure MFA

É necessário uma licença para o Azure MFA, que está disponível por meio do Azure AD Premium ou outros pacotes que o incluem. Licenças baseadas em consumo para a MFA do Azure como licenças por usuário ou por autenticação não são compatíveis com a extensão do NPS. Para obter mais informações, consulte [Como obter Autenticação Multifator do Azure](concept-mfa-licensing.md). Para fins de teste, você pode usar uma assinatura de avaliação.

### <a name="windows-server-software"></a>Software do Windows Server

A extensão NPS requer o Windows Server 2008 R2 SP1 ou superior com o serviço de função NPS instalado. Todas as etapas nesta seção foram realizadas usando o Windows Server 2016.

### <a name="network-policy-and-access-services-nps-role"></a>Função de Serviços de Acesso (NPS) e Política de Rede

O serviço de função NPS fornece o servidor RADIUS e a funcionalidade do cliente, bem como o serviço de integridade de Política de Acesso de Rede. Essa função deve ser instalada em pelo menos dois computadores em sua infraestrutura: O Gateway de área de trabalho remota e outro servidor membro ou controlador de domínio. Por padrão, a função já está presente no computador configurado como o Gateway de Área de Trabalho Remota.  Você deve também instalar a função NPS em pelo menos um outro computador, como um controlador de domínio ou servidor membro.

Para obter informações sobre como instalar a função do NPS do serviço Windows Server 2012 ou anterior, consulte [Instalar um Servidor de Política de Integridade de NAP](https://technet.microsoft.com/library/dd296890.aspx). Para obter uma descrição de melhores práticas recomendadas para NPS, incluindo a recomendação para instalar o NPS em um controlador de domínio, consulte [Práticas recomendadas para NPS](https://technet.microsoft.com/library/cc771746).

### <a name="azure-active-directory-synched-with-on-premises-active-directory"></a>Azure Active Directory sincronizado com o Active Directory local

Para usar a extensão do NPS, os usuários locais devem ser sincronizados com o Azure AD e habilitados para MFA. Esta seção pressupõe que os usuários locais são sincronizados com o Azure AD usando o AD Connect. Para obter informações sobre o Azure AD Connect, consulte [Integrar seus diretórios locais com o Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

### <a name="azure-active-directory-guid-id"></a>ID do GUID do Azure Active Directory

Para instalar a extensão NPS, você precisa saber o GUID do Azure AD. As instruções para localizar o GUID do Azure AD são fornecidas abaixo.

## <a name="configure-multi-factor-authentication"></a>Configurar Autenticação Multifator

Esta seção fornece instruções para integração do Azure MFA com o Gateway de Área de Trabalho Remota. Como administrador, você deve configurar o serviço Azure MFA antes que os usuários possam registrar seus dispositivos multifatores ou aplicativos.

Siga as etapas em [Guia de introdução à Autenticação Multifator do Azure na nuvem](howto-mfa-getstarted.md) para habilitar a MFA para os usuários do Azure AD.

### <a name="configure-accounts-for-two-step-verification"></a>Configurar contas para verificação em duas etapas

Depois que uma conta tiver sido habilitada para MFA, não será possível entrar nos recursos controlados pela política de MFA até que você tenha configurado com êxito um dispositivo confiável a ser usado para o segundo fator de autenticação e autenticar-se usando a verificação em duas etapas.

Siga as etapas em [O que a Autenticação Multifator do Azure significa para mim?](../user-help/multi-factor-authentication-end-user.md) para compreender e configurar corretamente os dispositivos para MFA com sua conta de usuário.

## <a name="install-and-configure-nps-extension"></a>Instalar e configurar a extensão do NPS

Esta seção fornece instruções para configurar a infraestrutura do RDS para usar o Azure MFA para autenticação de cliente com o Gateway de Área de Trabalho Remota.

### <a name="acquire-azure-active-directory-guid-id"></a>Obter a ID do GUID do Azure Active Directory

Como parte da configuração da extensão do NPS, você precisa fornecer credenciais de administrador e a ID do Azure AD para o seu locatário do Azure AD. As próximas etapas mostram como obter o ID do locatário.

1. Entre no [Portal do Azure](https://portal.azure.com) como administrador global do locatário do Azure.
1. Na barra de navegação esquerda, selecione o ícone **Azure Active Directory**.
1. Selecione **Propriedades**.
1. Na folha Propriedades, ao lado de ID do diretório, clique no ícone **Copiar**, conforme mostrado abaixo, para copiar a ID para a área de transferência.

   ![Obtendo a ID de diretório do portal do Azure](./media/howto-mfa-nps-extension-rdg/image1.png)

### <a name="install-the-nps-extension"></a>Instalar a extensão NPS

Instale a extensão NPS em um servidor que tem a função de Serviços de Acesso (NPS) e Política de Rede instalada. Isso funciona como o servidor RADIUS no seu projeto.

> [!Important]
> Certifique-se de que você não instalou a extensão NPS no servidor de Gateway de Área de Trabalho Remota.
>

1. Instalar a [extensão NPS](https://aka.ms/npsmfa).
1. Copie o arquivo de instalação executável (NpsExtnForAzureMfaInstaller.exe) para o servidor NPS.
1. No servidor NPS, clique duas vezes em **NpsExtnForAzureMfaInstaller.exe**. Se solicitado, clique em **Executar**.
1. Na extensão NPS para a caixa de diálogo Configuração do MFA do Azure, examine os termos de licença de software, marque **Eu concordo com os termos e condições de licença** e clique em **Instalar**.
1. Na Extensão NPS para a caixa de diálogo Configurar o MFA do Azure, clique em **Fechar**.

### <a name="configure-certificates-for-use-with-the-nps-extension-using-a-powershell-script"></a>Configurar certificados para uso com a extensão NPS usando um script do PowerShell

Em seguida, você precisa configurar certificados para uso pela extensão NPS para garantia e comunicações seguras. Os componentes NPS incluem um script do Windows PowerShell que configura um certificado autoassinado para uso com o NPS.

O script executa as ações a seguir:

* Cria um certificado autoassinado
* Associa a chave pública do certificado à entidade de serviço no Azure AD
* Armazena o certificado no repositório do computador local
* Concede acesso à chave privada do certificado ao usuário de rede
* Reinicia o serviço do Servidor de Políticas de Rede

Se você quiser usar seus próprios certificados, será necessário associar a chave pública do seu certificado à entidade de serviço no Azure AD e assim por diante.

Para usar o script, forneça a extensão com suas credenciais de administrador do Azure AD e a ID de locatário do Azure AD que você copiou anteriormente. Execute o script em cada servidor NPS onde você instalou a extensão NPS. Faremos o seguinte:

1. Abra um prompt do Windows PowerShell administrativo.
1. No prompt do PowerShell, digite `cd ‘c:\Program Files\Microsoft\AzureMfa\Config’` e clique em **ENTER**.
1. Digite`.\AzureMfaNpsExtnConfigSetup.ps1`, e aperte **ENTER**. O script verifica se o módulo do Azure Active Directory PowerShell está instalado. Se não estiver instalado, o script instala o módulo para você.

   ![Running AzureMfaNpsExtnConfigSetup.ps1 in Azure AD PowerShell](./media/howto-mfa-nps-extension-rdg/image4.png)
  
1. Depois que o script verifica a instalação do módulo do PowerShell, ele exibe a caixa de diálogo de módulo do Azure Active Directory PowerShell. Na caixa de diálogo, insira suas credenciais de administrador do Azure AD e a senha e clique em **Entrar**.

   ![Autenticar no Azure AD no PowerShell](./media/howto-mfa-nps-extension-rdg/image5.png)

1. Quando solicitado, cole a ID de diretório que você copiou anteriormente na área de transferência e pressione **ENTER**.

   ![Inserindo o ID de diretório no PowerShell](./media/howto-mfa-nps-extension-rdg/image6.png)

1. O script cria um certificado autoassinado e executa outras alterações de configuração. A saída deve ser como a imagem abaixo.

   ![Saída do PowerShell que mostra um certificado autoassinado](./media/howto-mfa-nps-extension-rdg/image7.png)

## <a name="configure-nps-components-on-remote-desktop-gateway"></a>Configurar componentes NPS no Gateway de Área de Trabalho Remota

Nesta seção, você pode configurar as diretivas de autorização de conexão de Gateway de Área de Trabalho Remota e outras configurações de RADIUS.

O fluxo de autenticação requer que as mensagens RADIUS sejam trocadas entre o Gateway de Área de Trabalho Remota e o servidor NPS onde o servidor NPS está instalado. Isso significa que você deve configurar as configurações do cliente RADIUS no Gateway de Área de Trabalho Remota e no servidor NPS onde a extensão NPS está instalada.

### <a name="configure-remote-desktop-gateway-connection-authorization-policies-to-use-central-store"></a>Configurar políticas de autorização de conexão de Gateway de Área de Trabalho Remota para usar o repositório central

As políticas de autorização de conexão de Área de Trabalho Remota (RD CAPs) especificam os requisitos para se conectar ao servidor de Gateway de Área de Trabalho Remota. As RD CAPs podem ser armazenadas localmente (padrão) ou podem ser armazenadas em um repositório de RD CAP central que está executando o NPS. Para configurar a integração do Azure MFA com RDS, você precisa especificar o uso de um repositório central.

1. No servidor de Gateway de Área de Trabalho Remota, abra **Gerenciador do Servidor**.
1. No menu, clique em **Ferramentas**, aponte para **Serviços de Área de Trabalho Remota** e, em seguida, clique em **Gerenciador de Gateway de Área de Trabalho Remota**.
1. No Gerenciador de Gateway de Área de Trabalho Remota, clique com o botão direito em **\[Nome do servidor\] (Local)** e clique em **Propriedades**.
1. Na caixa de diálogo Propriedades, selecione a guia do **Repositório de RD CAP**.
1. Acesse a guia Repositório RD CAP, selecione **Servidor central executando NPS**. 
1. No campo **Insira um nome ou endereço IP do servidor que executa o NPS**, digite o endereço IP ou o nome do servidor onde você instalou a extensão NPS.

   ![Insira o nome ou endereço IP do servidor NPS](./media/howto-mfa-nps-extension-rdg/image10.png)
  
1. Clique em **Adicionar**.
1. Na caixa de diálogo **Segredo compartilhado**, digite um segredo compartilhado e, em seguida, clique em **OK**. Certifique-se de registrar esse segredo compartilhado e armazene o registro de forma segura.

   >[!NOTE]
   >O segredo compartilhado é usado para estabelecer confiança entre os clientes e os servidores RADIUS. Crie um segredo longo e complexo.
   >

   ![Criar um segredo compartilhado para estabelecer a relação de confiança](./media/howto-mfa-nps-extension-rdg/image11.png)

1. Clique em **OK** para fechar a caixa de diálogo.

### <a name="configure-radius-timeout-value-on-remote-desktop-gateway-nps"></a>Configurar o valor de tempo limite do RADIUS no NPS de Gateway de Área de Trabalho Remota

Para garantir que haja tempo para validar as credenciais do usuário, executar a verificação em duas etapas, receber respostas e responder a mensagens RADIUS, é necessário ajustar o valor de tempo limite do RADIUS.

1. No servidor de Gateway de Área de Trabalho Remota, abra Gerenciador do Servidor. No menu, clique em **Ferramentas** e, em seguida, clique em **Servidor de Políticas de Rede**.
1. No console **NPS (Local)**, expanda **Clientes e Servidores RADIUS** e selecione **Servidor RADIUS remoto**.

   ![Console de gerenciamento de servidor de diretivas de rede mostrando o servidor RADIUS remoto](./media/howto-mfa-nps-extension-rdg/image12.png)

1. No painel de detalhes, clique duas vezes em **GRUPO DE SERVIDORES GATEWAY TS**.

   >[!NOTE]
   >Este grupo de servidores RADIUS foi criado quando você configurou o servidor central para políticas NPS. O Gateway de Área de Trabalho Remota encaminha mensagens RADIUS para este servidor ou grupo de servidores, se houver mais de um no grupo.
   >

1. Na caixa de diálogo **Propriedades de GRUPO DE SERVIDORES GATEWAY TS**, selecione o endereço IP ou nome do servidor NPS configurado para armazenar RD CAPs e, em seguida, clique em **Editar**.

   ![Selecione o IP ou nome do servidor NPS configurado anteriormente](./media/howto-mfa-nps-extension-rdg/image13.png)

1. Na caixa de diálogo **Editar Servidor RADIUS**, selecione a guia **Balanceamento de Carga**.
1. Na guia **Balanceamento de Carga** no campo **Número de segundos sem resposta antes que uma solicitação seja descartada**, altere o valor padrão de 3 para um valor entre 30 e 60 segundos.
1. No campo **Número de segundos entre as solicitações quando o servidor é identificado como indisponível**, altere o valor padrão de 30 segundos para um valor que seja igual ou maior que o valor especificado na etapa anterior.

   ![Editar configurações de tempo limite do servidor Radius no guia de balanceamento de carga](./media/howto-mfa-nps-extension-rdg/image14.png)

1. Clique em **OK** duas vezes para fechar as caixas de diálogo.

### <a name="verify-connection-request-policies"></a>Verifique as Políticas de Solicitação de Conexão

Por padrão, quando você configurar o Gateway de Área de Trabalho Remota para usar um repositório central de política para políticas de autorização de conexão, o Gateway de Área de Trabalho Remota está configurado para encaminhar solicitações de CAP para o servidor NPS. O servidor NPS com a extensão Azure MFA instalada, processa a solicitação de acesso RADIUS. As etapas a seguir mostram como verificar a política de solicitação de conexão padrão.

1. No Gateway de Área de Trabalho Remota, no console do NPS (Local), expanda **Políticas** e selecione **Políticas de Solicitação de Conexão**.
1. Clique duas vezes em **política de autorização de GATEWAY TS**.
1. Na caixa de diálogo **Propriedades da POLÍTICA DE AUTORIZAÇÃO DE GATEWAY TS**, clique na guia **Configurações**.
1. Na guia **Configurações**, em Encaminhamento de Solicitação de Conexão, clique em **Autenticação**. O cliente RADIUS está configurado para encaminhar solicitações para autenticação.

   ![Definir configurações de autenticação, especificando o grupo de servidores](./media/howto-mfa-nps-extension-rdg/image15.png)

1. Clique em **Cancelar**.

## <a name="configure-nps-on-the-server-where-the-nps-extension-is-installed"></a>Configurar o NPS no servidor onde a extensão NPS está instalada

O servidor NPS onde a extensão NPS está instalada deve ser capaz de trocar mensagens RADIUS com o servidor NPS no Gateway de Área de Trabalho Remota. Para habilitar a troca de mensagens, você precisa configurar os componentes do NPS no servidor onde o serviço de extensão NPS está instalado.

### <a name="register-server-in-active-directory"></a>Registrar o Servidor no Active Directory

Para funcionar corretamente nesse cenário, o servidor NPS deve ser registrado no Active Directory.

1. No servidor NPS, abra o **Gerenciador do Servidor**.
1. No Gerenciador do Servidor, clique em **Ferramentas**e, em seguida, clique em **Servidor de Políticas de Rede**.
1. No console do Servidor de Políticas de Rede, clique com o botão direito em **NPS (Local)** e, em seguida, clique em **Registrar servidor no Active Directory**.
1. Clique em **OK** duas vezes.

   ![Registrar o servidor NPS no Active Directory](./media/howto-mfa-nps-extension-rdg/image16.png)

1. Deixe o console aberto para o próximo procedimento.

### <a name="create-and-configure-radius-client"></a>Criar e configurar o cliente RADIUS

O Gateway de Área de Trabalho Remota deve ser configurado como um cliente RADIUS para o servidor NPS.

1. No servidor NPS onde a extensão NPS estiver instalada, no console **NPS (Local)**, clique com botão direito em **Clientes RADIUS** e clique em **Novo**.

   ![Criar um novo cliente RADIUS no console do NPS](./media/howto-mfa-nps-extension-rdg/image17.png)

1. Na caixa de diálogo **Novo cliente RADIUS**, forneça um nome amigável, como _Gateway_ e o endereço IP ou nome DNS do servidor de Gateway de Área de Trabalho Remota.
1. Nos campos **Segredo compartilhado** e **Confirmar segredo compartilhado**, insira o mesmo segredo usado anteriormente.

   ![Configurar um nome amigável e o endereço IP ou DNS](./media/howto-mfa-nps-extension-rdg/image18.png)

1. Clique em **OK** para fechar a caixa de diálogo do Novo cliente RADIUS.

### <a name="configure-network-policy"></a>Configurar Política de Rede

Lembre-se de que o servidor NPS com a extensão do MFA do Azure é o repositório de política central designado para a CAP (Política de Autorização de Conexão). Portanto, você precisa implementar uma CAP no servidor NPS para autorizar solicitações de conexão válidas.  

1. No Servidor NPS, abra o console NPS (Local), expanda **Políticas** e clique em **Políticas de Rede**.
1. Clique com o botão direito em **Conexões com outros servidores de acesso** e clique em **Duplicar Política**.

   ![Duplicar a conexão a outra política de servidores de acesso](./media/howto-mfa-nps-extension-rdg/image19.png)

1. Clique com o botão direito em **Cópia de Conexões para outros servidores de acesso** e clique em **Propriedades**.
1. Na caixa de diálogo **Cópia de conexões com outros servidores de acesso**, em **Nome da política**, insira um nome adequado, como _RDG_CAP_. Marque **Política habilitada** e selecione **Conceder acesso**. Opcionalmente, em **Tipo de servidor de acesso à rede**, selecione **Gateway de Área de Trabalho Remota** ou você pode deixá-lo como **Não especificado**.

   ![Nome da política, habilitar e conceder acesso](./media/howto-mfa-nps-extension-rdg/image21.png)

1. Clique na guia **Restrições** e marque **Permitem que os clientes se conectem sem negociar um método de autenticação**.

   ![Modificar métodos de autenticação para permitir que os clientes se conectem](./media/howto-mfa-nps-extension-rdg/image22.png)

1. Opcionalmente, clique na guia **Condições** e adicione as condições que devem ser atendidas para que a conexão seja autorizada, por exemplo, associação em um grupo específico do Windows.

   ![Opcionalmente, especifique as condições da conexão](./media/howto-mfa-nps-extension-rdg/image23.png)

1. Clique em **OK**. Quando solicitado para exibir o tópico da Ajuda correspondente, clique em **Não**.
1. Certifique-se de que a nova política está no topo da lista, que a política está habilitada, e que ela concede acesso.

   ![Mover sua política para o topo da lista](./media/howto-mfa-nps-extension-rdg/image24.png)

## <a name="verify-configuration"></a>Verificar a configuração

Para verificar a configuração, será necessário entrar no Gateway de Área de Trabalho Remota com um cliente RDP adequado. Certifique-se de usar uma conta que é permitida por suas Políticas de Autorização de Conexão e está habilitada para o Azure MFA.

Você pode usar a página **Acesso via Web à Área de Trabalho Remota**, conforme mostrado abaixo.

![Teste no acesso via Web da área de trabalho remota](./media/howto-mfa-nps-extension-rdg/image25.png)

Ao inserir corretamente as suas credenciais para autenticação primária, a caixa de diálogo Conexão de Área de Trabalho Remota mostra o status de Iniciando conexão remota, conforme mostrado abaixo. 

Se você autenticar com sucesso o método de autenticação secundário que você configurou anteriormente no Azure MFA, você está conectado ao recurso. No entanto, se a autenticação secundária não for bem-sucedida, seu acesso aos recursos será negado. 

![Conexão de área de trabalho remoto iniciando uma conexão remota](./media/howto-mfa-nps-extension-rdg/image26.png)

No exemplo a seguir, o aplicativo do Autenticador em um Windows Phone é usado para fornecer a autenticação secundária.

![Exemplo Windows Phone autenticador verificação mostrando de aplicativo](./media/howto-mfa-nps-extension-rdg/image27.png)

Depois de autenticado com sucesso usando o método de autenticação secundária, você é conectado ao Gateway de Área de Trabalho Remota normalmente. No entanto, como é necessário usar um método de autenticação secundária usando um aplicativo móvel em um dispositivo confiável, o processo de entrada é mais seguro do que seria caso contrário.

### <a name="view-event-viewer-logs-for-successful-logon-events"></a>Exibir logs do Visualizador de Eventos para eventos de logon com sucesso

Para exibir os eventos de logon com sucesso nos logs do Visualizador de Eventos do Windows, você pode emitir o seguinte comando do Windows PowerShell para consultar os logs de Serviços de Terminal do Windows e a Segurança do Windows.

Para consultar eventos de entrada bem-sucedida nos logs operacionais do Gateway _(Event Viewer\Applications and Services Logs\Microsoft\Windows\TerminalServices-Gateway\Operational)_, use os seguintes comandos do PowerShell:

* `Get-WinEvent -Logname Microsoft-Windows-TerminalServices-Gateway/Operational | where {$_.ID -eq '300'} | FL`
* Este comando exibe os eventos do Windows que mostram que o usuário atende aos requisitos da política de autorização de recursos (RD RAPS) e foi concedido acesso.

![Exibir eventos usando o PowerShell](./media/howto-mfa-nps-extension-rdg/image28.png)

* `Get-WinEvent -Logname Microsoft-Windows-TerminalServices-Gateway/Operational | where {$_.ID -eq '200'} | FL`
* Este comando exibe os eventos que aparecem quando o usuário atende aos requisitos de política de autorização de conexão.

![exibindo a política de autorização de conexão usando o PowerShell](./media/howto-mfa-nps-extension-rdg/image29.png)

Você também pode exibir esse log e o filtro nas IDs de evento, 300 e 200. Para consultar eventos de logon com sucesso nos logs de Visualizador de eventos de segurança, use o seguinte comando:

* `Get-WinEvent -Logname Security | where {$_.ID -eq '6272'} | FL`
* Este comando pode ser executado no NPS central ou no Servidor de Gateway de Área de Trabalho Remota.

![Eventos de logon com êxito de exemplo](./media/howto-mfa-nps-extension-rdg/image30.png)

Você também pode exibir o log de segurança ou o modo de exibição personalizado de Serviços de Acesso e Política de Rede, conforme mostrado abaixo:

![Visualizador de eventos de serviços de acesso e política de rede](./media/howto-mfa-nps-extension-rdg/image31.png)

No servidor onde você instalou a extensão NPS para o Azure MFA, você pode encontrar os logs de aplicativo do Visualizador de Eventos específicas para a extensão em _Application and Services Logs\Microsoft\AzureMfa_.

![Logs de eventos do aplicativo de visualizador AuthZ](./media/howto-mfa-nps-extension-rdg/image32.png)

## <a name="troubleshoot-guide"></a>Guia de Solução de Problemas

Se a configuração não estiver funcionando conforme o esperado, o primeiro lugar para iniciar a solução de problemas é verificar se o usuário está configurado para usar o Azure MFA. Peça ao usuário para conectar-se ao [Portal do Azure](https://portal.azure.com). Se os usuários são solicitados para verificação secundária e são autenticados com sucesso, você pode eliminar uma configuração incorreta do Azure MFA.

Se o Azure MFA está funcionando para o(s) usuário(s), você deve analisar os logs de eventos relevantes. Isso inclui os Eventos de Segurança, Gateway operacional e logs do Azure MFA que foram discutidos na seção anterior.

Abaixo está um exemplo de saída do log de segurança mostrando um evento de logon com falha (Evento ID 6273).

![Exemplo de um evento de logon com falha](./media/howto-mfa-nps-extension-rdg/image33.png)

Abaixo está um evento relacionado dos logs do AzureMFA:

![Exemplo de log do Azure MFA no Visualizador de eventos](./media/howto-mfa-nps-extension-rdg/image34.png)

Para executar opções de solução de problemas avançadas, consulte os arquivos de log de formato do banco de dados NPS onde o serviço NPS está instalado. Esses arquivos de log são criados na pasta _%SystemRoot%\System32\Logs_ como arquivos de texto separado por vírgula.

Para obter uma descrição desses arquivos de log, consulte [Interpretar arquivos de log de formato de banco de dados de NPS](https://technet.microsoft.com/library/cc771748.aspx). As entradas nesses arquivos de log podem ser difícil de interpretar sem importá-los para uma planilha ou um banco de dados. Você pode encontrar vários analisadores de IAS on-line para ajudá-lo a interpretar os arquivos de log.

A imagem abaixo mostra a saída de um desses [aplicativos shareware](https://www.deepsoftware.com/iasviewer) que pode ser baixado.

![Exemplo de analisador do IAS do aplicativo Shareware](./media/howto-mfa-nps-extension-rdg/image35.png)

E, por último, para mais opções de solução de problemas, você pode usar um analisador de protocolo, como o [Analisador de Mensagens da Microsoft](https://technet.microsoft.com/library/jj649776.aspx).

A imagem abaixo do Analisador de Mensagens da Microsoft mostra o tráfego de rede filtrado no protocolo RADIUS que contém o nome de usuário **CONTOSO\AliceC**.

![Microsoft Message Analyzer, mostrando o tráfego filtrado](./media/howto-mfa-nps-extension-rdg/image36.png)

## <a name="next-steps"></a>Próximas etapas

[Como obter a Autenticação Multifator do Azure](concept-mfa-licensing.md)

[Gateway de Área de Trabalho Remota e Servidor de Autenticação Multifator do Azure usando RADIUS](howto-mfaserver-nps-rdg.md)

[Integrar seus diretórios locais no Azure Active Directory](../hybrid/whatis-hybrid-identity.md)
