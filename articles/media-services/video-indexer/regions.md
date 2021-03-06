---
title: Regiões nas quais o Video Indexer está disponível – Azure
titlesuffix: Azure Media Services
description: Este artigo aborda as regiões do Azure nas quais o Video Indexer está disponível.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.topic: article
ms.date: 02/10/2019
ms.author: anzaman
ms.openlocfilehash: 8c1d51c2bb6062f1360c831549e523a5821ff14b
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60532184"
---
# <a name="azure-regions-in-which-video-indexer-exists"></a>Regiões do Azure nas quais existe um Video Indexer

As APIs do Video Indexer contêm um parâmetro **location** que deve ser definido com a região do Azure para a qual a chamada deve ser encaminhada. Ela precisa ser uma [região do Azure na qual o Video Indexer está disponível](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services&regions=all).

## <a name="locations"></a>Locais

O parâmetro **location** precisa receber o nome do código da região do Azure como seu valor. Se estiver usando o Video Indexer no modo de versão prévia, você deverá colocar *"trial"* como o valor. Caso contrário, para obter o nome do código da região do Azure na qual a conta está localizada e para a qual a chamada deverá ser encaminhada, execute a seguinte linha na [CLI do Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest):

```bash
az account list-locations
```

Depois de executar a linha mostrada acima, você obterá uma lista de todas as regiões do Azure. Navegue para a região do Azure que tenha o *displayName* que você está procurando e use seu valor *name* para o parâmetro **location**.

Por exemplo, para a região Oeste dos EUA 2 (exibida abaixo) do Azure, você usará "westus2" para o parâmetro **location**.

```json
   {
      "displayName": "West US 2",
      "id": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/locations/westus2",
      "latitude": "47.233",
      "longitude": "-119.852",
      "name": "westus2",
      "subscriptionId": null
    }
```

## <a name="next-steps"></a>Próximas etapas

- [Personalizar o modelo de Linguagem usando APIs](customize-language-model-with-api.md)
- [Personalizar o modelo de Marcas usando APIs](customize-brands-model-with-api.md)
- [Personalizar o modelo de Pessoa usando APIs](customize-person-model-with-api.md)
