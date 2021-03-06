---
title: Entender o desconto de reservas para bancos de dados SQL do Azure | Microsoft Docs
description: Saiba como um desconto de reserva é aplicado à execução de bancos de dados SQL do Azure.
documentationcenter: ''
author: yashesvi
manager: yashar
editor: ''
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/13/2019
ms.author: banders
ms.openlocfilehash: 4b4c6b390e9b3a0cf764f998523fe3c1cdc66026
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60370280"
---
# <a name="how-a-reservation-discount-is-applied-to-azure-sql-databases"></a>Como um desconto de reserva é aplicado a bancos de dados SQL do Azure

Depois de comprar uma capacidade reservada do Banco de Dados SQL do Azure, o desconto de reserva é aplicado automaticamente aos Bancos de Dados SQL que correspondem aos atributos e à quantidade da reserva. Uma reserva abrange os custos de computação de banco de dados SQL. Você é cobrado por software, armazenamento e rede às taxas normais. Você pode cobrir os custos de licenciamento para bancos de dados SQL com [benefício híbrido do Azure](https://azure.microsoft.com/pricing/hybrid-benefit/).

Para instâncias de máquina Virtual reservadas, consulte [entender instâncias reservadas do Azure VM desconto](billing-understand-vm-reservation-charges.md).

## <a name="how-reservation-discount-is-applied"></a>Como o desconto de reserva é aplicado

Um desconto de reserva é "*uso-it-ou-perder-it*". Dessa forma, se você não tiver recursos correspondentes para qualquer hora, em seguida, você perderá uma quantidade de reserva para essa hora. Você não pode transportar encaminhar horas reservadas não utilizadas.

Quando você desligar um recurso, o desconto de reserva se aplica automaticamente a outro recurso correspondente no escopo especificado. Se nenhum recurso de correspondência é encontrado no escopo especificado, são as horas reservadas *perdido*.

## <a name="discount-applied-to-sql-databases"></a>Desconto aplicado a bancos de dados SQL

 O desconto de capacidade reservada do Banco de Dados SQL é aplicado à execução de Bancos de Dados SQL a cada hora. A reserva que você compra é correspondida ao uso de computação emitido pelos Bancos de Dados SQL em execução. Para Bancos de Dados SQL que não são executados a hora inteira, a reserva é aplicada automaticamente a outros Bancos de Dados SQL que correspondem aos atributos de reserva. O desconto pode ser aplicado a bancos de dados SQL que estão sendo executados simultaneamente. Se você não tiver bancos de dados SQL que sejam executados durante a hora inteira que corresponda aos atributos de reserva, você não obterá o benefício total do desconto de reserva para essa hora.

Os exemplos a seguir mostram como o desconto de capacidade reservada do Banco de Dados SQL se aplica, dependendo do número de núcleos que você comprou e quando está sendo executado.

- Cenário 1: Você compra um Banco de Dados SQL com capacidade reservada para um Banco de Dados SQL de 8 núcleos. Você executa um banco de dados SQL de 16 núcleos que corresponde ao restante dos atributos da reserva. Você é cobrado pelo preço pré-pago de 8 núcleos de uso de computação do Banco de Dados SQL. Você recebe o desconto de reserva para uma hora de uso de computação do Banco de Dados SQL principal de 8.

Para o restante desses exemplos, suponha que a capacidade de banco de dados do SQL reservadas que comprar é para um banco de dados SQL de 16 núcleos e o restante dos atributos de reserva corresponder os bancos de dados SQL em execução.

- Cenário 2: você executa dois Bancos de Dados SQL com 8 núcleos cada por uma hora. O desconto de reserva de 16 núcleos é aplicado para computação de uso para as duas de 8 núcleos de bancos de dados SQL.
- Cenário 3: você executa um Banco de Dados SQL de 16 núcleos das 13h às 13h30. Você pode executar outro núcleo 16 banco de dados SQL de 1:30 às 2 horas. Ambos são cobertas pelo desconto de reserva.
- Cenário 4: você executa um Banco de Dados SQL de 16 núcleos das 13h às 13h45. Você pode executar outro núcleo 16 banco de dados SQL de 1:30 às 2 horas. Você será cobrado o preço pago conforme o uso para a sobreposição de 15 minutos. O desconto de reserva se aplica ao uso de computação para o restante do tempo.

Para entender e visualizar o aplicativo de suas reservas do Azure nos relatórios de uso de faturamento, consulte [Entender o uso de reserva do Azure](billing-understand-reserved-instance-usage-ea.md).

## <a name="need-help-contact-us"></a>Precisa de ajuda? Fale conosco

Se você tiver dúvidas ou precisar de Ajuda, [criar uma solicitação de suporte](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre as Reservas do Azure, consulte os seguintes artigos:

- [O que são Reservas do Azure?](billing-save-compute-costs-reservations.md)
- [Pré-pagamento para máquinas virtuais com instâncias de VMs reservadas do Azure](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Pagar antecipadamente por recursos de computação de banco de dados SQL com capacidade reservada do Azure SQL Database](../sql-database/sql-database-reserved-capacity.md)
- [Gerenciar Reservas do Azure](billing-manage-reserved-vm-instance.md)
- [Entender o uso de reserva para a sua assinatura paga conforme o uso](billing-understand-reserved-instance-usage.md)
- [Entenda o uso de reservas para o seu registro Enterprise](billing-understand-reserved-instance-usage-ea.md)
- [Entender o uso de reserva para assinaturas de CSP](/partner-center/azure-reservations)
