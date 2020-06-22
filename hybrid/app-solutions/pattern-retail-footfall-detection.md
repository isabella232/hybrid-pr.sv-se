---
title: Footfall identifierings mönster med Azure och Azure Stack hubb
description: Lär dig hur du använder Azure och Azure Stack Hub för att implementera en AI-baserad lösning för identifiering av Footfall för analys av detalj handels lagrings trafik.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912003"
---
# <a name="footfall-detection-pattern"></a>Mönster för Footfall-identifiering

Det här mönstret ger en översikt över hur du implementerar en AI-baserad lösning för identifiering av Footfall för att analysera besökares trafik i butiker. Lösningen genererar insikter från verkliga åtgärder med hjälp av Azure, Azure Stack Hub och Custom Vision AI dev kit.

## <a name="context-and-problem"></a>Kontext och problem

Contoso-butiker vill få insikter om hur kunderna får sina aktuella produkter i förhållande till Store-layout. De kan inte placera personal i varje avsnitt och det är ineffektivt att ha ett team med analytiker granska en hel butiks kamera tagningar. Dessutom har ingen av sina butiker tillräckligt med bandbredd för att strömma video från alla kameror till molnet för analys.

Contoso vill kunna hitta ett diskret, sekretess sätt för att fastställa deras kunders demografiska uppgifter, lojalitet och reaktioner på att lagra skärmar och produkter.

## <a name="solution"></a>Lösning

Det här mönstret för detalj handels analys använder en metod med flera nivåer för inferencing i kanten. Genom att använda Custom Vision AI dev kit skickas bara bilder med mänskliga ansikten för analys till ett privat Azure Stack hubb som kör Azure Cognitive Services. Anonymiserats skickas sammanställda data till Azure för agg regering över alla butiker och visualiseringar i Power BI. Genom att kombinera Edge och det offentliga molnet kan Contoso dra nytta av modern AI-teknik samtidigt som de fortfarande är kompatibla med företagets principer och respektera sina kunders integritet.

[![Lösning för Footfall identifierings mönster](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Här är en sammanfattning av hur lösningen fungerar:

1. Custom Vision AI dev kit hämtar en konfiguration från IoT Hub, som installerar IoT Edge Runtime och en ML-modell.
2. Om modellen ser en person, tar den en bild och laddar upp den till Azure Stack hubb Blob Storage.
3. Blob-tjänsten utlöser en Azure-funktion på Azure Stack Hub.
4. Azure-funktionen anropar en behållare med Ansikts-API för att få demografisk och känslo data från avbildningen.
5. Data är anonymiserats och skickas till ett Azure Event Hubs-kluster.
6. Event Hubs klustret pushar bort data till Stream Analytics.
7. Stream Analytics samlar in data och skickar dem till Power BI.

## <a name="components"></a>Komponenter

Den här lösningen använder följande komponenter:

| Lager | Komponent | Description |
|----------|-----------|-------------|
| Maskin vara i butiken | [Custom Vision AI dev kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Tillhandahåller filtrering i butiken med hjälp av en lokal ML-modell som bara fångar bilder av personer för analys. Säkert etablerade och uppdaterade via IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs ger en skalbar plattform för att mata in anonymiserats-data som kan integreras snyggt med Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Ett Azure Stream Analytics jobb sammanställer anonymiserats-data och grupperar dem till 15-sekunders fönster för visualisering. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI tillhandahåller ett lättanvänt instrument panels gränssnitt för visning av utdata från Azure Stream Analytics. |
| Azure Stack hubb | [App Service](/azure-stack/operator/azure-stack-app-service-overview.md) | App Service Resource Provider (RP) tillhandahåller en bas för Edge-komponenter, inklusive värd-och hanterings funktioner för webbappar/API: er och funktioner. |
| | Azure Kubernetes service [(AKS)-motor](https://github.com/Azure/aks-engine) kluster | AKS RP med AKS-Engine-kluster som distribueras till Azure Stack Hub ger en skalbar, elastisk motor för att köra Ansikts-API containern. |
| | [Ansikts-API behållare](/azure/cognitive-services/face/face-how-to-install-containers) för Azure Cognitive Services| Azure Cognitive Services RP med Ansikts-API-behållare tillhandahåller demografisk, känslo och unik identifiering av besökare i Contosos privata nätverk. |
| | Blob Storage | Bilder som har hämtats från AI dev kit överförs till Azure Stack hubbens Blob Storage. |
| | Azure Functions | En Azure-funktion som körs på Azure Stack hub tar emot indata från Blob Storage och hanterar interaktioner med Ansikts-API. Den avger anonymiserats-data till ett Event Hubs-kluster som finns i Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

För att den här lösningen ska kunna skalas över flera kameror och platser måste du kontrol lera att alla komponenter kan hantera den ökade belastningen. Du kan behöva vidta åtgärder som:

- Öka antalet Stream Analytics strömnings enheter.
- Skala ut Ansikts-API-distributionen.
- Öka Event Hubs-klustrets data flöde.
- För extrema fall kan det vara nödvändigt att migrera från Azure Functions till en virtuell dator.

### <a name="availability"></a>Tillgänglighet

Eftersom den här lösningen är i nivå av, är det viktigt att tänka på hur du hanterar nätverk eller strömavbrott. Beroende på affärs behov kanske du vill implementera en mekanism för att cachelagra avbildningar lokalt och sedan vidarebefordra till Azure Stack hubb när anslutningen returnerar. Om platsen är tillräckligt stor kan det vara ett bättre alternativ att distribuera en Data Box Edge med Ansikts-API container till den platsen.

### <a name="manageability"></a>Hanterbarhet

Den här lösningen kan omfatta många enheter och platser, vilket kan ge svårhanterligt. [Azures IoT-tjänster](/azure/iot-fundamentals/) kan användas för att automatiskt ta nya platser och enheter online och hålla dem uppdaterade.

### <a name="security"></a>Säkerhet

Den här lösningen fångar kund avbildningar, vilket gör säkerheten till en avgörande faktor. Kontrol lera att alla lagrings konton skyddas med rätt åtkomst principer och rotera nycklar regelbundet. Se till att lagrings konton och Event Hubs har bevarande principer som uppfyller sekretess bestämmelser för företag och myndigheter. Se också till att nivå nivåer för användar åtkomst. Med skiktning ser du till att användarna bara har åtkomst till de data de behöver för sin roll.

## <a name="next-steps"></a>Nästa steg

Mer information om de ämnen som introduceras i den här artikeln:

- Se det [skiktade data mönstret](https://aka.ms/tiereddatadeploy), som utnyttjas av Footfall identifierings mönster.
- Läs mer om hur du använder anpassad vision i [Custom vision AI dev kit](https://azure.github.io/Vision-AI-DevKit-Pages/) . 

När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för Footfall-identifiering](solution-deployment-guide-retail-footfall-detection.md). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.
