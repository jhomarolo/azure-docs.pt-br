---
title: Os links na página não funcionam para um aplicativo de Proxy de Aplicativo | Microsoft Docs
description: Como solucionar problemas com links desfeitos em aplicativos de Proxy de Aplicativo integrados com o Azure AD
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/10/2018
ms.author: celested
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: d860ac77653a97855248c45e43d28ea4c694979e
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60442243"
---
# <a name="links-on-the-page-dont-work-for-an-application-proxy-application"></a>Os links na página não funcionam para um aplicativo de Proxy de Aplicativo

Este artigo ajuda-o a solucionar problemas porque links no seu aplicativo de Proxy de Aplicativo do Azure Active Directory não funcionam corretamente.

## <a name="overview"></a>Visão geral 
Após publicar um aplicativo de Proxy de Aplicativo, os únicos links que funcionam por padrão no aplicativo são links para destinos contidos na URL raiz publicada. Os links nos aplicativos não estão funcionando, a URL interna para o aplicativo provavelmente não inclui todos os destinos de links dentro do aplicativo.

**Por que isso acontece?** Ao clicar em um link em um aplicativo, o Proxy de aplicativo tenta resolver a URL como uma URL interna dentro do mesmo aplicativo ou como uma URL disponível externamente. Se o link aponta para uma URL interna que não está dentro do mesmo aplicativo, ele não pertence a nenhum um desses buckets e resulta em um erro não encontrado.

## <a name="ways-you-can-resolve-broken-links"></a>Maneiras de resolver links desfeitos

Há três maneiras de resolver esse problema. As escolhas abaixo estão listadas em crescente complexidade.

1.  Certifique-se de se que a URL interna é uma raiz que contém todos os links relevantes para o aplicativo. Isso permite que todos os links sejam resolvidos como conteúdo publicado no mesmo aplicativo.

    Se você alterar a URL interna, mas não quiser alterar a página de aterrissagem para usuários, altere a URL da home page para a URL interna publicada anteriormente. Isso pode ser feito indo para "Azure Active Directory" -&gt; Registros de aplicativo -&gt; selecione o aplicativo -&gt; Propriedades. Neste guia de propriedades, você verá o campo "URL da Home Page", que pode ser ajustado para a página de aterrissagem desejada.

2.  Se seus aplicativos usam nomes de domínio totalmente qualificados (FQDNs), use [domínios personalizados](application-proxy-configure-custom-domain.md) para publicar seus aplicativos. Esse recurso permite que a mesma URL seja usada internamente e externamente.

    Essa opção garante que os links em seu aplicativo sejam acessíveis externamente através do Proxy de Aplicativo, pois os links dentro do aplicativo para URLs internas também são reconhecidos externamente. Todos os links ainda precisam pertencer a um aplicativo publicado. No entanto, com essa opção os links não precisam pertencer ao mesmo aplicativo e podem pertencer a vários aplicativos.

3.  Se nenhuma dessas opções for viável, há várias opções para habilitar a conversão de link embutido. Essas opções incluem o uso do Intune Managed Browser, da extensão My Apps ou o uso da configuração de conversão de link em seu aplicativo. Para saber mais sobre cada uma dessas opções e como habilitá-las, confira [Redirecionar links codificados para aplicativos publicados com o Proxy de Aplicativo do Azure AD](application-proxy-configure-hard-coded-link-translation.md).

## <a name="next-steps"></a>Próximos passos
[Trabalhar com servidores proxy locais existentes](application-proxy-configure-connectors-with-proxy-servers.md)

