---
title: Transformar dados com o Notebook do Databricks - Azure | Microsoft Docs
description: Saiba como processar ou transformar dados executando um notebook do Databricks.
services: data-factory
documentationcenter: ''
ms.assetid: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 03/15/2018
author: sharonlo101
ms.author: shlo
manager: craigg
ms.openlocfilehash: 8036a8694bb8c8d0db236eba831f13dc2bf47d0a
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60311657"
---
# <a name="transform-data-by-running-a-databricks-notebook"></a>Transformar dados executando um notebook do Databricks

A Atividade de Notebook do Azure Databricks em um [pipeline do Data Factory](concepts-pipelines-activities.md) executa um notebook do Databricks no workspace do Azure Databricks. Este documento se baseia no artigo sobre as  [atividades de transformação de dados](transform-data.md) , que apresenta uma visão geral da transformação de dados e das atividades de transformação permitidas. O Azure Databricks é uma plataforma gerenciada para executar o Apache Spark.

## <a name="databricks-notebook-activity-definition"></a>Definição de atividade do Notebook Databricks

A seguir está a definição JSON de exemplo de uma Atividade de Notebook do Databricks:

```json
{
    "activity": {
        "name": "MyActivity",
        "description": "MyActivity description",
        "type": "DatabricksNotebook",
        "linkedServiceName": {
            "referenceName": "MyDatabricksLinkedservice",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "notebookPath": "/Users/user@example.com/ScalaExampleNotebook",
            "baseParameters": {
                "inputpath": "input/folder1/",
                "outputpath": "output/"
            },
            "libraries": [
                {
                "jar": "dbfs:/docs/library.jar"
                }
            ]
        }
    }
}
```

## <a name="databricks-notebook-activity-properties"></a>Propriedades da atividade de Notebook do Databricks

A tabela a seguir descreve as propriedades JSON usadas na definição de JSON:

|Propriedade|DESCRIÇÃO|Obrigatório|
|---|---|---|
|Nome|Nome da atividade no pipeline.|Sim|
|description|Texto que descreve o que a atividade faz.|Não |
|tipo|Para Atividade de Notebook do Databricks, o tipo da atividade é DatabricksNotebook.|Sim|
|linkedServiceName|Nome do serviço vinculado do Databricks no qual o notebook do Databricks executa. Saiba mais sobre esse serviço vinculado no artigo  [Serviços de computação vinculados](compute-linked-services.md) .|Sim|
|notebookPath|O caminho absoluto do notebook a ser executado no workspace do Databricks. Esse caminho deve começar com uma barra.|Sim|
|baseParameters|Uma matriz de pares chave-valor. Parâmetros básicos podem ser utilizados para cada execução de atividade. Se o notebook utilizar um parâmetro que não for especificado, será usado o valor padrão do notebook. Encontre mais informações sobre os parâmetros em [Notebooks do Databricks](https://docs.databricks.com/api/latest/jobs.html#jobsparampair).|Não |
|bibliotecas|Uma lista de bibliotecas a serem instaladas no cluster, que executará o trabalho. Pode ser uma matriz de \<cadeia de caracteres, object>.|Não |


## <a name="supported-libraries-for-databricks-activities"></a>Bibliotecas com suporte para atividades do Databricks

Na definição da atividade acima do Databricks você especifica esses tipos de biblioteca: *jar*, *egg*, *maven*, *pypi*,  *cran*.

```json
{
    "libraries": [
        {
            "jar": "dbfs:/mnt/libraries/library.jar"
        },
        {
            "egg": "dbfs:/mnt/libraries/library.egg"
        },
        {
            "maven": {
                "coordinates": "org.jsoup:jsoup:1.7.2",
                "exclusions": [ "slf4j:slf4j" ]
            }
        },
        {
            "pypi": {
                "package": "simplejson",
                "repo": "http://my-pypi-mirror.com"
            }
        },
        {
            "cran": {
                "package": "ada",
                "repo": "https://cran.us.r-project.org"
            }
        }
    ]
}

```

Para obter mais detalhes, consulte a [documentação do Databricks](https://docs.azuredatabricks.net/api/latest/libraries.html#managedlibrarieslibrary) para tipos de biblioteca.

## <a name="how-to-upload-a-library-in-databricks"></a>Como carregar uma biblioteca no Databricks

#### <a name="using-databricks-workspace-uihttpsdocsazuredatabricksnetuser-guidelibrarieshtmlcreate-a-library"></a>[Usando o workspace do Databricks da interface do usuário](https://docs.azuredatabricks.net/user-guide/libraries.html#create-a-library)

Para obter o caminho dbfs da biblioteca adicionada usando a interface do usuário, você pode usar a [CLI do Databricks (instalação)](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#install-the-cli). 

Normalmente, as bibliotecas Jar são armazenadas em dbfs:/FileStore/jars ao usar a interface do usuário. Você pode listar todos os por meio da CLI: *databricks fs ls dbfs:/FileStore/jars*.



#### <a name="copy-library-using-databricks-clihttpsdocsazuredatabricksnetuser-guidedev-toolsdatabricks-clihtmlcopy-a-file-to-dbfs"></a>[Copie a biblioteca usando a CLI do Databricks](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#copy-a-file-to-dbfs)

Exemplo: *databricks fs cp SparkPi-assembly-0.1.jar dbfs:/FileStore/jars*
