---
title: Arquivo de inclusão
description: Arquivo de inclusão
services: storage
author: wmgries
ms.service: storage
ms.topic: include
ms.date: 07/18/2018
ms.author: wgries
ms.custom: include file
ms.openlocfilehash: 19c570be71583a2933a39780ba948edeb9f68498
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2019
ms.locfileid: "62129086"
---
| Resource | Destino | Limite rígido |
|----------|--------------|------------|
| Serviços de Sincronização de Armazenamento por região | 15 serviços de sincronização de armazenamento | Sim |
| Grupos de sincronização por Serviço de Sincronização de Armazenamento | 100 grupos de sincronização | Sim |
| Servidores registrados por Serviço de Sincronização de Armazenamento | 99 servidores | Sim |
| Pontos de extremidade de nuvem por grupo de sincronização | 1 ponto de extremidade de nuvem | Sim |
| Pontos de extremidade do servidor por grupo de sincronização | 50 pontos de extremidade de servidor | Não  |
| Pontos de extremidade de servidor por servidor | 30 pontos de extremidade de servidor | Sim |
| Tamanho do ponto de extremidade | 4 TiB | Não  |
| Objetos do sistema de arquivos (diretórios e arquivos) por grupo de sincronização | 25 milhões de objetos | Não  |
| Número máximo de objetos do sistema de arquivos (diretórios e arquivos) em um diretório | 1 milhão de objetos | Sim |
| Tamanho máximo do descritor de segurança (diretórios e arquivos) do objeto | 64 KiB | Sim |
| Tamanho do arquivo | 100 GiB | Não  |
| Tamanho mínimo do arquivo para que um arquivo seja colocado em camadas | 64 KiB | Sim |
| Sessões de sincronização simultâneas | Agente V4 e posterior: O limite varia com base nos recursos do sistema disponíveis. <BR> Agente V3: Duas sessões de sincronização do Active Directory por processador ou um máximo de oito sessões de sincronização do Active Directory por servidor. | Sim
