---
title: Exemplo de implantação de script da CLI do Azure
description: Criar um cluster Linux seguro do Service Fabric no Azure usando a CLI do Azure.
services: service-fabric
documentationcenter: ''
author: aljo-microsoft
manager: chackdan
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 01/18/2018
ms.author: aljo
ms.custom: mvc
ms.openlocfilehash: 3a50d6672833ab1b4986e38ca1ddf24a8c33c562
ms.sourcegitcommit: c6dc9abb30c75629ef88b833655c2d1e78609b89
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/29/2019
ms.locfileid: "58664447"
---
# <a name="create-a-secure-service-fabric-linux-cluster-in-azure"></a>Criar um cluster Linux seguro do Service Fabric no Azure

Esse comando cria um certificado autoassinado, adiciona-o a um cofre de chaves e baixa o certificado localmente.  O novo certificado é usado para proteger o cluster quando ele for implantado.  Você também pode usar um certificado já existente em vez de criar um novo.  De qualquer forma, o nome da referência do certificado deve corresponder ao domínio usado para acessar o cluster do Service Fabric. Essa correspondência é necessária para fornecer um SSL para os pontos de extremidade de gerenciamento de HTTPS e o Service Fabric Explorer do cluster. Você não pode obter um certificado SSL de uma AC para o domínio `.cloudapp.azure.com` . Você deve obter um nome de domínio personalizado para seu cluster. Quando você solicitar um certificado de uma autoridade de certificação, o nome de assunto do certificado deve corresponder ao nome de domínio personalizado usado para seu cluster.

Se necessário, instale a [CLI do Azure](/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="sample-script"></a>Script de exemplo

[!code-sh[main](../../../cli_scripts/service-fabric/create-cluster/create-cluster.sh "Deploy an application to a cluster")]

## <a name="clean-up-deployment"></a>Limpar a implantação

Após a execução do exemplo de script, o comando a seguir pode ser usado para remover o grupo de recursos, o cluster e todos os recursos relacionados.

```azurecli
ResourceGroupName = "aztestclustergroup"
az group delete --name $ResourceGroupName
```

## <a name="script-explanation"></a>Explicação sobre o script

Este script usa os seguintes comandos. Cada comando da tabela é vinculado à documentação específica do comando.

| Comando | Observações |
|---|---|
| [az sf cluster create](https://docs.microsoft.com/cli/azure/sf/cluster?view=azure-cli-latest) | Cria um novo cluster do Service Fabric.  |

## <a name="next-steps"></a>Próximas etapas

Mais exemplos da CLI do Service Fabric para o Azure Service Fabric podem ser encontrados em [Exemplos da CLI do Service Fabric](../samples-cli.md).
