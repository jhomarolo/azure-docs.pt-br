---
title: Configurar as declarações de grupo para aplicativos com o Active Directory do Azure | Microsoft Docs
description: Informações sobre como configurar as declarações de grupo para uso com o Azure AD.
services: active-directory
documentationcenter: ''
ms.reviewer: paulgarn
manager: daveba
ms.component: hybrid
ms.service: active-directory
ms.workload: identity
ms.topic: article
ms.date: 02/27/2019
ms.author: billmath
author: billmath
ms.openlocfilehash: 622a3ce0f80bd09bd09fa7ff097f68155318142d
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60351158"
---
# <a name="configure-group-claims-for-applications-with-azure-active-directory-public-preview"></a>Configurar as declarações de grupo para aplicativos com o Azure Active Directory (visualização pública)

O Azure Active Directory pode fornecer uma informações de associação de grupo de usuários em tokens para uso em aplicativos.  Há suporte para dois padrões principais:

- Grupos identificados por seu identificador de objeto (OID) (geralmente disponível) do Active Directory do Azure
- Grupos identificados por SAMAccountName ou GroupSID de AD (Active Directory) sincronizadas grupos e usuários (visualização pública)

> [!Note]
> Suporte ao uso de locais e nomes de identificadores de segurança (SIDs) foi projetado para permitir a movimentação de aplicativos existentes do AD FS.    Grupos gerenciados no Azure AD não contêm os atributos necessários para emitir essas declarações.

## <a name="group-claims-for-applications-migrating-from-ad-fs-and-other-idps"></a>Declarações de grupo para aplicativos Migrando do AD FS e outros IDPs

Muitos aplicativos que são configurados para se autenticar com o AD FS se baseiam nas informações de associação de grupo na forma de atributos de grupo do AD do Windows.   Esses atributos são o SAMAccountName, que pode ser o nome qualificado pelo domínio do grupo ou o SID de grupo do Windows.  Quando o aplicativo é federado com o AD FS, o AD FS usa a função TokenGroups para recuperar as associações de grupo para o usuário.

Para corresponder o token receberia um aplicativo do AD FS, podem ser emitidas a declarações de função e grupo que contém o domínio qualificado SAMAccountName, em vez do objectID do grupo do Active Directory do Azure.

Os formatos com suporte para declarações de grupo são:

- **GroupObjectId de diretório Active Directory do Azure** (disponível para todos os grupos)
- **SAMAccountName** (disponível para grupos sincronizados do Active Directory)
- **NetbiosDomain\samAccountName** (disponível para grupos sincronizados do Active Directory)
- **DNSDomainName\samAccountName** (disponível para grupos sincronizados do Active Directory)

> [!NOTE]
> Atributos SAMAccountName e OnPremisesGroupSID só estão disponíveis em objetos de grupo sincronizados do Active Directory.   Eles não estão disponíveis em grupos criados no Azure Active Directory ou Office 365.   Aplicativos que dependem de atributos de grupo local obtém-los sincronizados apenas para grupos.

## <a name="options-for-applications-to-consume-group-information"></a>Opções para consumo de informações do grupo de aplicativos

Uma maneira para aplicativos para obter informações de grupo é chamar o ponto de extremidade de grupos do gráfico para recuperar a associação de grupo para o usuário autenticado. Essa chamada garante que todos os grupos de que um usuário é membro do estão disponíveis mesmo quando há um grande número de grupos envolvidos e o aplicativo precisa para enumerar todos os grupos que o usuário é membro do.  Enumeração de grupo, em seguida, é independente das limitações de tamanho de token.

No entanto, se um aplicativo existente já espera consumir as informações de grupo por meio de declarações no token, que ele recebe, do Active Directory do Azure pode ser configurado com um número de opções de declarações diferentes para atender às necessidades do aplicativo.  Considere as seguintes opções:

- Ao usar a associação de grupo para uma finalidade de autorização do aplicativo (se a associação de grupo é obtida de token ou de gráfico), é preferível usar o ObjectID do grupo, que é imutável e exclusivo no Azure Active Directory e estão disponíveis para todos os grupos .
- Se usando o SAMAccountName de grupo para autorização, use nomes de domínio qualificado;  tem menos chance de situações decorrentes havia conflito de nomes. SAMAccountName por conta própria pode ser exclusivo dentro de um domínio do Active Directory, mas se mais de um domínio do Active Directory estiver sincronizado com um locatário do Azure Active Directory há a possibilidade de mais de um grupo tenham o mesmo nome.
- Considere o uso de [funções de aplicativo](../../active-directory/develop/howto-add-app-roles-in-azure-ad-apps.md) para fornecer uma camada de indireção entre a associação de grupo e o aplicativo.   Em seguida, o aplicativo toma decisões de autorização interno com base em clams de função no token.
- Se o aplicativo é configurado para obter atributos de grupo que são sincronizados do Active Directory e um grupo não contém os atributos que ele não será incluído nas declarações.
- Declarações de grupo em tokens incluem grupos aninhados.   Se um usuário é membro do GroupB e GroupB é um membro de GroupA, as declarações de grupo para o usuário conterá GroupA e GroupB. Para organizações com uso intenso de grupos aninhados e os usuários com um grande número de associações de grupo, o número de grupos listados no token pode aumentar o tamanho de token.   O Azure Active Directory limita o número de grupos, ele emite um token de 150 para declarações SAML e 200 para JWT.

> Pré-requisitos para o uso de atributos de grupo sincronizados do Active Directory:   Os grupos devem ser sincronizados do Active Directory usando o Azure AD Connect.

Há duas etapas para configurar o Azure Active Directory para a emissão de nomes de grupo para grupos do Active Directory.

1. **Sincronizar os nomes de grupo do Active Directory** antes do Azure Active Directory podem emitir os nomes de grupo ou no grupo local no grupo ou função de declarações de SID, os atributos necessários precisam ser sincronizados do Active Directory.  Você deve estar executando o Azure AD Connect versão 1.2.70 ou posterior.   Antes da versão 1.2.70 do Azure AD Connect sincronizará os objetos de grupo th do Active Directory, mas não inclui os atributos de nome de grupo necessário por padrão.  Você deve atualizar para a versão atual.

2. **Configurar o registro do aplicativo no Azure Active Directory para incluir declarações de grupo nos tokens** declarações de grupo podem ser configurado na seção de aplicativos empresariais do portal para um aplicativo de galeria ou inexistente na Galeria do SSO do SAML, ou usando o manifesto do aplicativo na seção de registros de aplicativo.  Para configurar as declarações de grupo no aplicativo manifesto consulte "Configurando o registro de aplicativo do Active Directory do Azure para os atributos de grupo" abaixo.

## <a name="configure-group-claims-for-saml-applications-using-sso-configuration"></a>Configurar as declarações de grupo para aplicativos SAML usando a configuração de SSO

Para configurar as declarações de grupo para um aplicativo de galeria ou inexistente na Galeria de SAML, abrir aplicativos empresariais, clique no aplicativo na lista e selecione a configuração de logon único.

Selecione o ícone Editar ao lado de "Grupos retornadas no token"

![declarações de interface do usuário](media/how-to-connect-fed-group-claims/group-claims-ui-1.png)

Use os botões de opção para selecionar a quais grupos devem ser incluídos no token

![declarações de interface do usuário](media/how-to-connect-fed-group-claims/group-claims-ui-2.png)

| Seleção | DESCRIÇÃO |
|----------|-------------|
| **Todos os grupos** | Emite a grupos de segurança e distribuição de lista.   Ele também faz com que funções de diretório que o usuário está atribuído a ser emitida em uma declaração 'wids' e quaisquer funções de aplicativo que o usuário está atribuído a ser emitido na declaração de funções. |
| **Grupos de segurança** | Emite a grupos de segurança que o usuário é membro na declaração grupos |
| **Lista de distribuição** | Emite a listas de distribuição, de que o usuário é um membro |
| **Função de diretório** | Se o usuário for atribuído a funções de diretório são emitidos como 'wids' declaração (declaração não será emitida grupos) |

Por exemplo, para emitir todos os grupos de segurança que o usuário é um membro de, selecione os grupos de segurança

![declarações de interface do usuário](media/how-to-connect-fed-group-claims/group-claims-ui-3.png)

### <a name="advanced-options"></a>Opções Avançadas

A maneira como as declarações de grupo são emitidas pode ser modificado pelas configurações em Opções avançadas

Personalize o nome da declaração de grupo:  Se selecionado, um tipo de declaração diferentes pode ser especificado para declarações de grupo.   Insira o tipo de declaração no campo de nome e o namespace opcional para a declaração no campo namespace.

![declarações de interface do usuário](media/how-to-connect-fed-group-claims/group-claims-ui-4.png)

Para emitir grupos usando o Active Directory atributos em vez do Azure AD objectIDs marque a caixa 'Retornar grupos como nomes, em vez de IDs' em selecionar o formato na lista suspensa.  Isso substitui a ID de objeto nas declarações com valores de cadeia de caracteres que contém os nomes de grupo.   Somente os grupos sincronizados do Active Directory serão incluídos nas declarações.

![declarações de interface do usuário](media/how-to-connect-fed-group-claims/group-claims-ui-5.png)

Alguns aplicativos exigem que as informações de associação do grupo apareçam na declaração de 'função'. Opcionalmente, você pode emitir os grupos do usuário como funções, marcando a caixa de emissão de declarações de uma função de grupos.

![declarações de interface do usuário](media/how-to-connect-fed-group-claims/group-claims-ui-6.png)

> [!NOTE]
> Se a opção de emissão de dados de grupo como funções for usada, somente os grupos aparecerão na declaração de função.  Quaisquer funções de aplicativo que o usuário está atribuído não será exibida na declaração de função.

## <a name="configure-the-azure-ad-application-registration-for-group-attributes"></a>Configurar o registro de aplicativo do Azure AD para atributos de grupo

Declarações de grupo também podem ser configuradas na [declarações opcionais](../../active-directory/develop/active-directory-optional-claims.md) seção o [manifesto do aplicativo](../../active-directory/develop/reference-app-manifest.md).

1. No portal -> Azure Active Directory -> aplicativo registros -> selecione aplicativo -> manifesto

2. Habilitar declarações de associação de grupo, alterando o groupMembershipClaim

   Os valores válidos são:

   - "Tudo"
   - "SecurityGroup"
   - "DistributionList"
   - "DirectoryRole"

   Por exemplo: 

   ```json
   "groupMembershipClaims": "SecurityGroup"
   ```

   Por padrão, de que será emitido ObjectIDs de grupo no grupo valor de declaração.  Para modificar o valor da declaração para conter atributos de grupo local ou para alterar o tipo de declaração para função, use a configuração de OptionalClaims da seguinte maneira:

3. Conjunto de declarações opcionais de configuração de nome de grupo.

   Se você quiser grupos no token para conter os atributos de grupo do AD na seção de declarações opcionais especificam qual declaração opcional de tipo de token deve ser aplicada ao local, o nome da declaração opcional solicitado e outras propriedades desejadas.  Vários tipos de token podem ser listados:

   - Token para o token de ID de OIDC
   - accessToken para o token de acesso OAuth/OIDC
   - Saml2Token para tokens SAML.

   > [!NOTE]
   > O tipo de Saml2Token se aplica a SAML1.1 e SAML 2.0, tokens de formato

   Para cada tipo de token relevante, modifique a declaração de grupos para usar a seção OptionalClaims no manifesto. O esquema de OptionalClaims é da seguinte maneira:

   ```json
   {
   "name": "groups",
   "source": null,
   "essential": false,
   "additionalProperties": []
   }
   ```

   | Esquema de declarações opcionais | Value |
   |----------|-------------|
   | **name:** | Deve ser "grupos" |
   | **Fonte:** | Não usado. Omitir ou especifique null |
   | **essential:** | Não usado. Omitir ou especifique false |
   | **additionalProperties:** | Lista de propriedades adicionais.  As opções válidas são "sam_account_name", "dns_domain_and_sam_account_name", "netbios_domain_and_sam_account_name", "emit_as_roles" |

   Em additionalProperties apenas um dos "sam_account_name", "dns_domain_and_sam_account_name", "netbios_domain_and_sam_account_name" são necessárias.  Se mais de um estiver presente, o primeiro é usado e quaisquer outros ignorados.

   Alguns aplicativos exigem informações de grupo sobre o usuário na declaração de função.  Para alterar o tipo de declaração de um grupo de declaração para uma declaração de função, adicione "emit_as_roles" às propriedades adicionais.  Os valores do grupo serão emitidos na declaração de função.

   > [!NOTE]
   > Se "emit_as_roles" for usado quaisquer funções de aplicativo configurado que o usuário é atribuído será não aparecem na declaração de função

### <a name="examples"></a>Exemplos

Emissão de grupos como nomes de grupo em tokens de acesso OAuth no formato dnsDomainName\SAMAccountName

```json
"optionalClaims": {
    "accessToken": [{
        "name": "groups",
        "additionalProperties": ["dns_domain_and_sam_account_name"]
    }]
}
 ```

Emitir nomes de grupo a ser retornado em formato netbiosDomain\samAccountName como as funções de declaração SAML e Tokens de ID de OIDC:

```json
"optionalClaims": {
    "saml2Token": [{
        "name": "groups",
        "additionalProperties": ["netbios_name_and_sam_account_name", "emit_as_roles"]
    }],

    "idToken": [{
        "name": "groups",
        "additionalProperties": ["netbios_name_and_sam_account_name", "emit_as_roles"]
    }]
 }
 ```

## <a name="next-steps"></a>Próximas etapas

[O que é identidade híbrida?](whatis-hybrid-identity.md)