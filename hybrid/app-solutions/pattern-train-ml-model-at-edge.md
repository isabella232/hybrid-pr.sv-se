---
title: Träna maskin inlärnings modell i gräns mönstret
description: Lär dig hur du gör Machine Learning-modellens utbildning på gränsen med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911949"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>Träna maskin inlärnings modell i gräns mönstret

Generera modeller för bärbara Machine Learning (ML) från data som bara finns lokalt.

## <a name="context-and-problem"></a>Kontext och problem

Många organisationer skulle vilja låsa upp insikter från sina lokala eller äldre data med hjälp av verktyg som deras data experter förstår. [Azure Machine Learning](/azure/machine-learning/) tillhandahåller Cloud-inbyggt verktyg för att träna, finjustera och distribuera ml-och djup inlärnings modeller.  

Men vissa data är för stora för att skickas till molnet eller kan inte skickas till molnet av reglerings skäl. Med det här mönstret kan data forskare använda Azure Machine Learning för att träna modeller med lokala data och data bearbetning.

## <a name="solution"></a>Lösning

Träningen vid gräns mönstret använder en virtuell dator (VM) som körs på Azure Stack Hub. Den virtuella datorn är registrerad som ett beräknings mål i Azure ML, så att den får åtkomst till data som bara finns lokalt. I det här fallet lagras data i Azure Stack hubbens Blob Storage.

När modellen har tränats registreras den med Azure ML, containerd och läggs till i en Azure Container Registry för distribution. För den här iterationen av mönstret måste den virtuella datorn för Azure Stack Hub-utbildning vara tillgänglig via det offentliga Internet.

[![Träna ML-modell på gräns arkitekturen](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

Så här fungerar mönstret:

1. Den virtuella datorn Azure Stack Hub distribueras och registreras som ett beräknings mål med Azure ML.
2. Ett experiment skapas i Azure ML som använder den virtuella datorn Azure Stack Hub som ett beräknings mål.
3. När modellen har tränats registreras den och containern.
4. Modellen kan nu distribueras till platser som antingen är lokala eller i molnet.

## <a name="components"></a>Komponenter

Den här lösningen använder följande komponenter:

| Lager | Komponent | Description |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [Azure Machine Learning](/azure/machine-learning/) dirigerar utbildningen för ml-modellen. |
| | Azure Container Registry | Azure ML paketerar modellen i en behållare och lagrar den i en [Azure Container Registry](/azure/container-registry/) för distribution.|
| Azure Stack hubb | App Service | [Azure Stack hubb med App Service](/azure-stack/operator/azure-stack-app-service-overview) tillhandahåller basen för komponenterna på gränsen. |
| | Compute | En Azure Stack Hub-dator som kör Ubuntu med Docker används för att träna ML-modellen. |
| | Storage | Privata data kan finnas i Azure Stack hubb Blob Storage. |

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

Om du vill aktivera den här lösningen för skalning måste du skapa en lämplig virtuell dator på Azure Stack hubb för utbildning.

### <a name="availability"></a>Tillgänglighet

Se till att utbildnings skripten och Azure Stack Hub VM har åtkomst till lokala data som används för utbildning.

### <a name="manageability"></a>Hanterbarhet

Se till att modeller och experiment är korrekt registrerade, versioner och taggade för att undvika förvirring vid modell distribution.

### <a name="security"></a>Säkerhet

Det här mönstret ger Azure ML åtkomst till möjliga känsliga data lokalt. Se till att det konto som används för SSH i Azure Stack Hub-VM har ett starkt lösen ord och utbildnings skript inte bevarar eller överför data till molnet.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- I [Azure Machine Learning-dokumentationen](/azure/machine-learning) finns en översikt över ml och närliggande ämnen.
- Se [Azure Container Registry](/azure/container-registry/) för att lära dig att skapa, lagra och hantera avbildningar för behållar distributioner.
- Läs mer om resurs leverantören och hur du distribuerar med hjälp av [app service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) .
- Se [design överväganden för hybrid program](overview-app-design-considerations.md) för att lära dig mer om metod tips och hur du får svar på ytterligare frågor.
- Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösnings exemplet fortsätter du med [träna ml-modellen i gräns distributions guiden](https://aka.ms/edgetrainingdeploy). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.
