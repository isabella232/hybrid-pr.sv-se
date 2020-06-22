---
title: Slut på identifiering av lager med Azure och Azure Stack Edge
description: Lär dig hur du använder Azure och Azure Stack Edge-tjänster för att genomföra bristande lager identifiering.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911988"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Slut på lager identifiering i gräns mönstret

Det här mönstret illustrerar hur du avgör om hyllorna har slut på lager objekt med hjälp av en Azure Stack Edge-eller Azure IoT Edge enhets-och nätverks kameror.

## <a name="context-and-problem"></a>Kontext och problem

Fysiska detalj handels butiker förlorar försäljning eftersom när kunderna söker efter ett objekt, finns det inte på hyllan. Objektet kan dock ha varit på bak sidan av butiken och inte bearbetas igen. Butiker vill använda sin personal mer effektivt och få automatiskt meddelanden när objekt behöver återskapas.

## <a name="solution"></a>Lösning

I lösnings exemplet används en gräns enhet, till exempel en Azure Stack Edge i varje butik, som effektivt bearbetar data från kameror i butiken. I den här optimerade designen kan butiker endast skicka relevanta händelser och avbildningar till molnet. Designen sparar bandbredd, lagrings utrymme och säkerställer kund sekretess. När bild rutor läses från varje kamera, bearbetar en ML-modell avbildningen och returnerar eventuella inaktuella lager områden. Bilden och antalet lager områden visas i en lokal webbapp. Dessa data kan skickas till en insikts miljö i Time Series som visar insikter i Power BI.

![Brist på aktie lösnings arkitektur](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Så här fungerar lösningen:

1. Avbildningar samlas in från en nätverks kamera över HTTP eller RTSP.
2. Bilden ändrar storlek och skickas till driv rutins driv rutinen, som kommunicerar med ML-modellen för att avgöra om det finns några lager bilder.
3. ML-modellen returnerar eventuella inaktuella lager områden.
4. Inferencing-drivrutinen laddar upp RAW-avbildningen till en BLOB (om den anges) och skickar resultatet från modellen till Azure IoT Hub och en processor för en markerings ram på enheten.
5. Processorns markerings ram lägger till avgränsnings rutor i avbildningen och cachelagrar avbildnings Sök vägen i en minnes intern databas.
6. Webbappen frågar efter bilder och visar dem i den ordning som har tagits emot.
7. Meddelanden från IoT Hub sammanställs i Time Series Insights.
8. Power BI visar en interaktiv rapport över färdiga lager objekt över tid med data från Time Series Insights.


## <a name="components"></a>Komponenter

Den här lösningen använder följande komponenter:

| Lager | Komponent | Description |
|----------|-----------|-------------|
| Lokal maskin vara | Nätverks kamera | En nätverks kamera krävs, med antingen ett HTTP-eller RTSP-flöde för att tillhandahålla avbildningar för härledning. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) hanterar enhets etablering och meddelanden för gräns enheterna. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) lagrar meddelanden från IoT Hub för visualisering. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) ger affärs fokuserade rapporter om brist på aktie händelser. Power BI tillhandahåller ett lättanvänt instrument panels gränssnitt för visning av utdata från Azure Stream Analytics. |
| Azure Stack Edge eller<br>Azure IoT Edge enhet | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) dirigerar körningen för de lokala behållarna och hanterar enhets hantering och uppdateringar.|
| | Azure Project-Brainwave | På en Azure Stack Edge-enhet använder [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) FPGAs (Field-programmerbara grind mat ris) för att påskynda ml inferencing.|

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

De flesta Machine Learning-modeller kan bara köras med ett visst antal bild rutor per sekund, beroende på vilken maskin vara som tillhandahålls. Fastställ den optimala samplings frekvensen från dina kamera (er) för att säkerställa att den ML pipelinen inte säkerhets kopie ras. Olika typer av maskin vara kommer att hantera olika antal kameror och bild hastigheter.

### <a name="availability"></a>Tillgänglighet

Det är viktigt att tänka på vad som kan hända om gräns enheten förlorar anslutningen. Överväg vilka data som kan gå förlorade från Time Series Insights och Power BI instrument panelen. Exempel lösningen som anges har inte utformats för hög tillgänglighet.

### <a name="manageability"></a>Hanterbarhet

Den här lösningen kan omfatta många enheter och platser, vilket kan ge svårhanterligt. Azures IoT-tjänster kan automatiskt placera nya platser och enheter online och hålla dem uppdaterade. Lämpliga data styrnings procedurer måste också följas.

### <a name="security"></a>Säkerhet

Det här mönstret hanterar potentiellt känsliga data. Se till att nycklarna roteras regelbundet och att behörigheterna för Azure Storage konto och lokala resurser är korrekt inställda.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:
- Flera IoT-relaterade tjänster används i det här mönstret, inklusive [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/)och [Azure Time Series Insights](/azure/time-series-insights/).
- Läs mer om Microsoft Project Brainwave i [blogg meddelandet](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) och kolla in [Azure accelererade Machine Learning med Project Brainwave video](https://www.youtube.com/watch?v=DJfMobMjCX0).
- I [design överväganden för Hybrid appar](overview-app-design-considerations.md) kan du läsa mer om metod tips och få svar på ytterligare frågor.
- Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösnings exemplet kan du fortsätta med [distributions guiden för analys av lösningar](https://aka.ms/edgeinferencingdeploy). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.
