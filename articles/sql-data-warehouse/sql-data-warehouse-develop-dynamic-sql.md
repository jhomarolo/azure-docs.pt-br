---
title: Usando exibições do SQL no Azure SQL Data Warehouse | Microsoft Docs
description: Dicas para usar SQL dinâmico no SQL Data Warehouse do Azure para desenvolvimento de soluções.
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: implement
ms.date: 04/17/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: e1d209828313068e61b4acec3c4f92b7d643e1b6
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60402481"
---
# <a name="dynamic-sql-in-sql-data-warehouse"></a>SQL dinâmico no SQL Data Warehouse
Dicas para usar SQL dinâmico no SQL Data Warehouse do Azure para desenvolvimento de soluções.

## <a name="dynamic-sql-example"></a>Exemplo SQL dinâmico

Ao desenvolver o código do aplicativo para o SQL Data Warehouse, talvez seja preciso usar um sql dinâmico para ajudar a fornecer soluções flexíveis, genéricas e modulares. No momento, o SQL Data Warehouse não dá suporte a tipos de dados de blob. O não suporte a tipos de dados blob pode limitar o tamanho de suas cadeias de caracteres desde que os tipos de dados blobs incluam tipos varchar (máx) e nvarchar (máx). Se você usou estes tipos no código do seu aplicativo ao compilar cadeias de caracteres muito grandes, você precisará dividir o código em partes e usar a instrução EXEC em seu lugar.

Um exemplo simples:

```sql
DECLARE @sql_fragment1 VARCHAR(8000)=' SELECT name '
,       @sql_fragment2 VARCHAR(8000)=' FROM sys.system_views '
,       @sql_fragment3 VARCHAR(8000)=' WHERE name like ''%table%''';

EXEC( @sql_fragment1 + @sql_fragment2 + @sql_fragment3);
```

Se a cadeia de caracteres for curta, você poderá usar [sp_executesql](/sql/relational-databases/system-stored-procedures/sp-executesql-transact-sql) normalmente.

> [!NOTE]
> Instruções executadas como SQL dinâmico ainda estarão sujeitas a todas as regras de validação de TSQL.
> 
> 

## <a name="next-steps"></a>Próximas etapas
Para obter mais dicas de desenvolvimento, confira [visão geral de desenvolvimento](sql-data-warehouse-overview-develop.md).

