---
title: Simulação de dispositivo com monitoramento remoto do IoT - Azure | Microsoft Docs
description: Este guia de instruções mostra como usar o simulador de dispositivo com o acelerador de solução de monitoramento remoto.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 03/08/2019
ms.topic: conceptual
ms.openlocfilehash: 81efd9dc5d33ed23574b1cb66f26ccb444d5a3ff
ms.sourcegitcommit: 5839af386c5a2ad46aaaeb90a13065ef94e61e74
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/19/2019
ms.locfileid: "58180527"
---
# <a name="create-and-test-a-new-simulated-device"></a>Criar e testar um novo dispositivo simulado

O acelerador de solução de Monitoramento Remoto permite que você defina seus próprios dispositivos simulados. Este artigo mostra como definir um novo dispositivo de lâmpada simulado e, em seguida, testá-lo localmente. O acelerador de solução inclui dispositivos simulados, como resfriadores e caminhões. No entanto, você pode definir seus próprios dispositivos simulados para testar as soluções de IoT antes de implantar dispositivos reais.

> [!NOTE]
> Este artigo descreve como usar dispositivos simulados hospedados no serviço de simulação do dispositivo. Se você quer criar um dispositivo real, confira [Conectar seu dispositivo ao acelerador de solução de Monitoramento Remoto](iot-accelerators-connecting-devices.md).

Este guia de instruções mostra como personalizar o microsserviço de simulação do dispositivo. Esse microsserviço faz parte do acelerador de solução de Monitoramento Remoto. Para mostrar os recursos de simulação do dispositivo, este guia de instruções usa dois cenários no aplicativo de IoT da Contoso:

[!INCLUDE [iot-solution-accelerators-create-device](../../includes/iot-solution-accelerators-create-device.md)]

## <a name="next-steps"></a>Próximas etapas

Este guia mostrou como criar um tipo de dispositivo simulado personalizado e testá-lo executando o microsserviço de simulação do dispositivo localmente.

A próxima etapa recomendada é saber como implantar os tipos de dispositivos simulados personalizados no [Acelerador de solução de Monitoramento Remoto](iot-accelerators-remote-monitoring-deploy-simulated-device.md).
