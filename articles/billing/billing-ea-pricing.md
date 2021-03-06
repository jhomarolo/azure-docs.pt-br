---
title: Exibir e baixar os preços do Azure da sua organização | Microsoft Docs
description: Saiba como exibir e baixar o preço ou estimar os custos com o preço da sua organização.
services: ''
documentationcenter: ''
author: adpick
manager: jureid
editor: ''
tags: billing
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/22/2019
ms.author: banders
ms.custom: seodec18
ms.openlocfilehash: d4c64433089f14d845cea8be9adf0ef13675cfd1
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "62127626"
---
# <a name="view-and-download-your-organizations-azure-pricing"></a>Exibir e baixar os preços do Azure da sua organização

Os clientes do Azure com um Enterprise Agreement (EA) ou [contrato de cliente do Microsoft](#check-your-access-to-a-microsoft-customer-agreement) pode exibir e baixar seus preços no portal do Azure. Se você tiver um contrato de cliente da Microsoft, consulte a exibição e download de preço para o contrato de cliente da Microsoft.

## <a name="view-and-download-ea-pricing"></a>Exibir e fazer o download do preço de EA

Dependendo das políticas definidas para sua organização pelo Enterprise Admin, somente determinadas funções administrativas fornecem acesso às informações de preços do EA da organização. Para obter mais informações, consulte [Entendendo as funções administrativas do Azure Enterprise Agreement no Azure](billing-understand-ea-roles.md).

1. Como um administrador de empresa, entrar para o [portal do Azure](https://portal.azure.com/).
1. Pesquise **Gerenciamento de Custos do Azure + Cobrança**.

   ![Captura de tela que mostra a pesquisa do portal do Azure](./media/billing-ea-pricing/portal-cm-billing-search.png)

1. Na conta de faturamento, selecione **Usage + charges**.

   ![Captura de tela que mostra o uso e as cobranças em Faturamento](./media/billing-ea-pricing/ea-pricing-usage-charges-nav.png)

1. Selecione ![Captura de tela que mostra a pesquisa do portal do Azure](./media/billing-ea-pricing/download-icon.png) **Faça o Download** para o mês.
1. Sob **tabela de preços**, selecione **fazer o download do csv**.

   ![Captura de tela que mostra o botão csv de download da planilha de preços](./media/billing-ea-pricing/download-ea-price-sheet.png)

## <a name="view-and-download-pricing-for-your-microsoft-customer-agreement"></a>Exibir e baixar os preços para o contrato de cliente da Microsoft

Você deve ser o proprietário do perfil de cobrança, colaborador, leitor ou Gerenciador de faturas para exibir e baixar o preço. Para saber mais sobre as funções de cobrança para contratos de clientes da Microsoft, consulte [perfil de funções e tarefas de cobrança](billing-understand-mca-roles.md#billing-profile-roles-and-tasks).

### <a name="download-price-sheets-for-the-current-billing-period"></a>Baixe planilhas de preço para o período de cobrança atual

1. Entre no [Portal do Azure](https://portal.azure.com).
1. Pesquise **Gerenciamento de Custos do Azure + Cobrança**.
1. Selecione um perfil de cobrança. Dependendo do seu acesso, você precisa selecionar uma conta de cobrança pela primeira vez.
1. No **visão geral** folha, localize os links de download sob os encargos month-to-date.
1. Selecione **tabela de preços do Azure**.
![Captura de tela que mostra o download da visão geral](./media/billing-ea-pricing/open-pricing.png)

### <a name="download-price-sheets-for-billed-charges"></a>Baixe planilhas de preço para os encargos cobrados

1. Entre no [Portal do Azure](https://portal.azure.com).
1. Pesquise **Gerenciamento de Custos do Azure + Cobrança**.
1. Selecione um perfil de cobrança. Dependendo do seu acesso, você precisa selecionar uma conta de cobrança pela primeira vez.
1. Selecione **Faturas**.
1. Na grade de nota fiscal, localize a linha da fatura correspondente à tabela de preços que você deseja baixar.
1. Clique nas reticências (`...`) no final da linha.
![Captura de tela que mostra o botão de reticências selecionado](./media/billing-ea-pricing/billingprofile-invoicegrid.png)

1. Se você quiser ver os preços para os serviços na fatura selecionada, selecione **tabela de preços de nota fiscal**.
1. Se você quiser ver os preços de todos os serviços do Azure para determinado período de cobrança, selecione **tabela de preços do Azure**.

![Captura de tela que mostra o menu de contexto com folhas de preços](./media/billing-ea-pricing/contextmenu-pricesheet.png)

## <a name="estimate-costs-with-the-azure-pricing-calculator"></a>Estimar os custos com a Calculadora de preços do Azure

Você também pode usar o preço da sua organização para estimar os custos com a Calculadora de preços do Azure.

1. Vá para a [Calculadora de preços do Azure](https://azure.microsoft.com/pricing/calculator).
1. No canto superior direito, selecione **entrar**.
1. Em **Programas e Oferta** > **Programa de Licenciamento**, selecione **Enterprise Agreement (EA)**.
1. Em **Programas e Oferta** > **Contrato selecionado**, selecione **Nenhum selecionado**.

    ![Captura de tela que mostra o botão csv de download da planilha de preços](./media/billing-ea-pricing/ea-pricing-calculator-estimate.png)

1. Escolha a organização.
1. Escolha **Aplicar**.
1. Pesquise e adicione produtos à sua estimativa.
1. Os preços estimados apresentados são baseados no preço da organização selecionada.

## <a name="check-your-access-to-a-microsoft-customer-agreement"></a>Verifique o acesso a um contrato de cliente da Microsoft
[!INCLUDE [billing-check-mca](../../includes/billing-check-mca.md)]

## <a name="next-steps"></a>Próximas etapas

Se você for um cliente EA, consulte:

- [Gerenciar o acesso às suas informações de cobrança EA do Azure](billing-manage-access.md)
- [Exibir e baixar sua fatura do Microsoft Azure](billing-download-azure-invoice.md)
- [Exibir e baixar seus encargos e o uso do Microsoft Azure](billing-download-azure-daily-usage.md)
- [Entenda sua fatura dos clientes Enterprise Agreement](billing-understand-your-bill-ea.md)

Se você tiver um contrato de cliente da Microsoft, consulte:

- [Compreender os termos na sua folha de preço para um contrato de cliente da Microsoft](billing-mca-understand-pricesheet.md)
- [Exibir e baixar sua fatura do Microsoft Azure](billing-download-azure-invoice.md)
- [Exibir e baixar seus encargos e o uso do Microsoft Azure](billing-download-azure-daily-usage.md)
- [Exibir e baixar documentos de imposto para o perfil de cobrança](billing-mca-download-tax-document.md)
- [Entender os encargos na fatura do seu perfil de cobrança](billing-mca-understand-your-bill.md)
