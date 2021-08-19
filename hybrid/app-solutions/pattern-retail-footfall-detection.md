---
title: Mönster för fotavkänning med Azure och Azure Stack Hub
description: Lär dig hur du använder Azure Azure Stack Hub för att implementera en AI-baserad lösning för fotavkänning för analys av butikstrafik.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 79fb39d418bed53ef6a78980fcd9188bdf6e57ae
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281286"
---
# <a name="footfall-detection-pattern"></a>Mönster för fotfallsidentifiering

Det här mönstret ger en översikt för implementering av en AI-baserad lösning för fotfallsidentifiering för analys av besökartrafik i detaljhandeln. Lösningen genererar insikter från verkliga åtgärder med hjälp av Azure, Azure Stack Hub och Custom Vision AI Dev Kit.

## <a name="context-and-problem"></a>Kontext och problem

Contoso-butiker vill få insikter om hur kunder tar emot sina aktuella produkter i förhållande till butikslayouten. De kan inte placera personalen i varje avsnitt och det är ineffektivt att få ett team med analytiker att granska en hel butiks kamerasekvenser. Dessutom har ingen av deras butiker tillräckligt med bandbredd för att strömma video från alla sina kameror till molnet för analys.

Contoso vill hitta ett diskret, sekretessvänligt sätt att fastställa kundernas demografi, lojalitet och reaktioner på butiksvisningar och produkter.

## <a name="solution"></a>Lösning

Det här detaljhandelsanalysmönstret använder en nivåindelad metod för att dra slutsatser vid gränsen. Med hjälp Custom Vision AI Dev Kit skickas endast bilder med ansikten för analys till en privat Azure Stack Hub som kör Azure Cognitive Services. Anonymiserade, aggregerade data skickas till Azure för aggregering över alla butiker och visualiseringar i Power BI. Genom att kombinera gränsmolnet och det offentliga molnet kan Contoso dra nytta av modern AI-teknik samtidigt som de efterlever sina företagsprinciper och respekterar sina kunders integritet.

[![Lösning för mönster för sidfallsidentifiering](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Här är en sammanfattning av hur lösningen fungerar:

1. Den Custom Vision AI Dev Kit hämtar en konfiguration från IoT Hub, som installerar IoT Edge Runtime och en ML-modell.
2. Om modellen ser en person tar den en bild och laddar upp den till Azure Stack Hub bloblagring.
3. Blob-tjänsten utlöser en Azure-funktion Azure Stack Hub.
4. Azure-funktionen anropar en container med ansikts-API:et för att hämta demografiska data och känslodata från bilden.
5. Data maskeras och skickas till ett Azure Event Hubs kluster.
6. Klustret Event Hubs data till Stream Analytics.
7. Stream Analytics aggregerar data och push-erar dem till Power BI.

## <a name="components"></a>Komponenter

Den här lösningen använder följande komponenter:

| Skikt | Komponent | Beskrivning |
|----------|-----------|-------------|
| Maskinvara i butik | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Tillhandahåller filtrering i butik med hjälp av en lokal ML som endast samlar in bilder av personer för analys. Etableras och uppdateras säkert via IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs en skalbar plattform för att mata in anonymiserade data som integreras smidigt med Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Ett Azure Stream Analytics aggregerar anonymiserade data och grupperar dem i 15-sekundersfönster för visualisering. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI ett lätt att använda instrumentpanelsgränssnitt för att visa utdata från Azure Stream Analytics. |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | Den App Service resursprovidern (RP) tillhandahåller en bas för gränskomponenter, inklusive värd- och hanteringsfunktioner för webbappar/API:er och Functions. |
| | Azure Kubernetes Service [(AKS)-motorkluster](https://github.com/Azure/aks-engine) | AKS RP med ett AKS-Engine som distribueras till Azure Stack Hub ger en skalbar, motståndskraftig motor för att köra ansikts-API-containern. |
| | Azure Cognitive Services [ansikts-API-containrar](/azure/cognitive-services/face/face-how-to-install-containers)| Den Azure Cognitive Services RP med Ansikts-API-containrar ger demografi, känslor och unik identifiering av besökare i Contosos privata nätverk. |
| | Blob Storage | Avbildningar som har tagits från AI Dev Kit laddas upp Azure Stack Hub till bloblagringen. |
| | Azure Functions | En Azure-funktion som körs Azure Stack Hub tar emot indata från Blob Storage och hanterar interaktionerna med ansikts-API:et. Den skickar anonymiserade data till ett Event Hubs kluster som finns i Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

För att den här lösningen ska kunna skalas över flera kameror och platser måste du se till att alla komponenter kan hantera den ökade belastningen. Du kan behöva vidta åtgärder som:

- Öka antalet strömningsenheter Stream Analytics enheter.
- Skala ut ansikts-API-distributionen.
- Öka Event Hubs klustergenomflödet.
- I extrema fall kan det vara nödvändigt att Azure Functions från en virtuell dator till en virtuell dator.

### <a name="availability"></a>Tillgänglighet

Eftersom den här lösningen är nivåindelad är det viktigt att tänka på hur du hanterar nätverks- eller strömavbrott. Beroende på företagets behov kanske du vill implementera en mekanism för att cachelagra bilder lokalt och sedan vidarebefordra till Azure Stack Hub när anslutningen returneras. Om platsen är tillräckligt stor kan distribution av en Data Box-enhet Edge med ansikts-API-containern till den platsen vara ett bättre alternativ.

### <a name="manageability"></a>Hanterbarhet

Den här lösningen kan sträcka sig över många enheter och platser, vilket kan bli oskadlig. [Azures IoT-tjänster](/azure/iot-fundamentals/) kan användas för att automatiskt ta nya platser och enheter online och hålla dem uppdaterade.

### <a name="security"></a>Säkerhet

Den här lösningen samlar in kundbilder, vilket gör säkerheten ytterst viktig. Se till att alla lagringskonton skyddas med rätt åtkomstprinciper och rotera nycklar regelbundet. Se till att lagringskonton Event Hubs har kvarhållningsprinciper som uppfyller företagets och myndigheters sekretessregler. Se även till att nivåindelad användaråtkomstnivå. Nivåindelad lagring säkerställer att användarna bara har åtkomst till de data som de behöver för sin roll.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnena som introduceras i den här artikeln:

- Se mönstret [för nivåindelade](https://aka.ms/tiereddatadeploy)data , som används av mönstret för fotfallsidentifiering.
- I Custom Vision [AI Dev Kit kan du](https://azure.github.io/Vision-AI-DevKit-Pages/) läsa mer om hur du använder Custom Vision. 

När du är redo att testa lösningsexempel fortsätter du med [distributionsguiden för Footfall-identifiering.](/azure/architecture/hybrid/deployments/solution-deployment-guide-retail-footfall-detection) Distributionsguiden innehåller stegvisa instruktioner för att distribuera och testa dess komponenter.