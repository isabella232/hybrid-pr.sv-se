---
title: Out of stock detection using Azure and Azure Stack Edge
description: Lär dig hur du använder Azure och Azure Stack Edge för att implementera out-of-stock-identifiering.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343883"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Identifiering av out-of-lager vid kantmönstret

Det här mönstret illustrerar hur du fastställer om hyllor har produkter utanför lager med hjälp av Azure Stack Edge eller Azure IoT Edge enhet och nätverkskameror.

## <a name="context-and-problem"></a>Kontext och problem

Fysiska butiker förlorar försäljning eftersom när kunderna letar efter ett objekt finns den inte på en hyllplan. Objektet kan dock ha funnits i butikens backend-lagring och inte fyllas på igen. Butiker vill använda personalen mer effektivt och meddelas automatiskt när objekt behöver fyllas på.

## <a name="solution"></a>Lösning

I lösningsexempel används en gränsenhet, till exempel en Azure Stack Edge i varje butik, som effektivt bearbetar data från kameror i butiken. Med den här optimerade designen kan butiker endast skicka relevanta händelser och bilder till molnet. Designen sparar bandbredd, lagringsutrymme och säkerställer kundens integritet. När bildrutor läses från varje kamera bearbetar en ML-modell bilden och returnerar alla områden utanför lagerområdet. Bilden och utanför lagerområden visas i en lokal webbapp. Dessa data kan skickas till en Time Series Insight-miljö för att visa insikter i Power BI.

![Lösningsarkitektur med slut på lager vid gränsen](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Så här fungerar lösningen:

1. Bilder tas från en nätverkskamera via HTTP eller RTSP.
2. Bildens storlek ändras och skickas till inferensdrivrutinen, som kommunicerar med ML-modellen för att avgöra om det finns några avbildningar som inte finns i lager.
3. ML-modellen returnerar alla områden utanför lagerområdet.
4. Inferensdrivrutinen laddar upp den råa bilden till en blob (om sådan anges) och skickar resultatet från modellen till Azure IoT Hub och en begränsningsruta på enheten.
5. Begränsningsreprocessorn lägger till avgränsande rutor i bilden och cachelagrar bildsökvägen i en minnes minnesbaserad databas.
6. Webbappen frågar efter bilder och visar dem i den ordning som tas emot.
7. Meddelanden från IoT Hub sammanställs i Time Series Insights.
8. Power BI visar en interaktiv rapport över slut på lagerartiklar över tid med data från Time Series Insights.


## <a name="components"></a>Komponenter

Den här lösningen använder följande komponenter:

| Skikt | Komponent | Beskrivning |
|----------|-----------|-------------|
| Lokal maskinvara | Nätverkskamera | En nätverkskamera krävs, med antingen en HTTP- eller RTSP-feed för att tillhandahålla bilder för slutsatsledning. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) hanterar enhetsetablering och meddelanden för gränsenheterna. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) lagrar meddelanden från IoT Hub för visualisering. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) tillhandahåller affärsfokuserade rapporter om händelser som inte är i lager. Power BI ett lätt att använda instrumentpanelsgränssnitt för att visa utdata från Azure Stream Analytics. |
| Azure Stack Edge eller<br>Azure IoT Edge enhet | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) orkestrering av körningen för de lokala containrarna och hanterar enhetshantering och uppdateringar.|
| | Brainwave för Azure-projekt | På en Azure Stack Edge enhet använder [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) Field-Programmable Gate Arrays (FPGA) för att påskynda ML-inferens.|

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

De flesta maskininlärningsmodeller kan bara köras på ett visst antal bildrutor per sekund, beroende på den maskinvara som tillhandahålls. Fastställ den optimala samplingsfrekvensen från dina kamera(er) för att säkerställa att ML-pipelinen inte backar upp. Olika typer av maskinvara hanterar olika antal kameror och bildfrekvenser.

### <a name="availability"></a>Tillgänglighet

Det är viktigt att tänka på vad som kan hända om gränsenheterna förlorar anslutningen. Fundera över vilka data som kan gå förlorade från Time Series Insights och Power BI instrumentpanel. Den tillhandahållna exempellösningen är inte utformad för att vara hög tillgänglig.

### <a name="manageability"></a>Hanterbarhet

Den här lösningen kan sträcka sig över många enheter och platser, vilket kan bli oskadlig. Azures IoT-tjänster kan automatiskt ta nya platser och enheter online och hålla dem uppdaterade. Lämpliga procedurer för datastyrning måste också följas.

### <a name="security"></a>Säkerhet

Det här mönstret hanterar potentiellt känsliga data. Kontrollera att nycklarna roteras regelbundet och att behörigheterna för Azure Storage konto och lokala resurser har angetts korrekt.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:
- Flera IoT-relaterade tjänster används i det här [mönstret, inklusive Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/)och [Azure Time Series Insights](/azure/time-series-insights/).
- Mer information om Microsoft Project [](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) Brainwave finns i bloggmeddelandet och kolla in [videon Azure Accelerated Machine Learning with Project Brainwave (Azure Accelerated Machine Learning with Project Brainwave).](https://www.youtube.com/watch?v=DJfMobMjCX0)
- Se [Designöverväganden för hybridapp](overview-app-design-considerations.md) för att lära dig mer om metodtips och få svar på eventuella ytterligare frågor.
- I Azure Stack [med produkter och lösningar kan du](/azure-stack) läsa mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösningsexempel fortsätter du med distributionsguiden [för Edge ML-härledningslösningen.](https://aka.ms/edgeinferencingdeploy) Distributionsguiden innehåller stegvisa instruktioner för att distribuera och testa dess komponenter.
