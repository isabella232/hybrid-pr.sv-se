---
title: Skalningsmönster mellan moln i Azure Stack Hub
description: Lär dig hur du skapar en skalbar app mellan moln i Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 90e0c177b5eaee4d223b4613e0b2ddf385fa799c
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281269"
---
# <a name="cross-cloud-scaling-pattern"></a>Skalningsmönster mellan moln

Lägg automatiskt till resurser i en befintlig app för att hantera en ökad belastning.

## <a name="context-and-problem"></a>Kontext och problem

Din app kan inte öka kapaciteten för att möta oväntade ökningar i efterfrågan. Den här bristen på skalbarhet leder till att användarna inte når appen under perioder med hög belastning. Appen kan använda ett fast antal användare.

Globala företag behöver säkra, tillförlitliga och tillgängliga molnbaserade appar. Det är mycket viktigt att möta ökad efterfrågan och att använda rätt infrastruktur för att stödja efterfrågan. Företag har svårt att balansera kostnader och underhåll med säkerhet, lagring och tillgänglighet i realtid.

Du kanske inte kan köra appen i det offentliga molnet. Det kanske dock inte är ekonomiskt gångbart för företaget att upprätthålla den kapacitet som krävs i den lokala miljön för att hantera toppar i efterfrågan på appen. Med det här mönstret kan du använda elasticiteten i det offentliga molnet med din lokala lösning.

## <a name="solution"></a>Lösning

Skalningsmönstret mellan moln utökar en app som finns i ett lokalt moln med offentliga molnresurser. Mönstret utlöses av en ökning eller minskning av efterfrågan och lägger till eller tar bort resurser i molnet. Dessa resurser ger redundans, snabb tillgänglighet och geo-kompatibel routning.

![Skalningsmönster mellan moln](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Det här mönstret gäller endast tillståndslösa komponenter i din app.

## <a name="components"></a>Komponenter

Skalningsmönstret mellan moln består av följande komponenter.

### <a name="outside-the-cloud"></a>Utanför molnet

#### <a name="traffic-manager"></a>Traffic Manager

I diagrammet finns detta utanför den offentliga molngruppen, men den skulle behöva kunna samordna trafik i både det lokala datacentret och det offentliga molnet. Balanseraren ger hög tillgänglighet för appen genom att övervaka slutpunkter och tillhandahålla omdistribution av redundans vid behov.

#### <a name="domain-name-system-dns"></a>DNS (Domain Name System)

Den Domain Name System, eller DNS, ansvarar för att översätta (eller matcha) ett webbplats- eller tjänstnamn till dess IP-adress.

### <a name="cloud"></a>Moln

#### <a name="hosted-build-server"></a>Värd- och byggserver

En miljö som är värd för din bygg-pipeline.

#### <a name="app-resources"></a>Appresurser

Appresurserna måste kunna skala in och ut, till exempel vm-skalningsuppsättningar och containrar.

#### <a name="custom-domain-name"></a>Anpassat domännamn

Använd ett anpassat domännamn för att dirigera begäranden glob.

#### <a name="public-ip-addresses"></a>Offentliga IP-adresser

Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till slutpunkten för appens offentliga molnresurser.  

### <a name="local-cloud"></a>Lokalt moln

#### <a name="hosted-build-server"></a>Värd- och byggserver

En miljö som är värd för din bygg-pipeline.

#### <a name="app-resources"></a>Appresurser

Appresurserna behöver kunna skala in och ut, till exempel VM-skalningsuppsättningar och containrar.

#### <a name="custom-domain-name"></a>Anpassat domännamn

Använd ett anpassat domännamn för att dirigera begäranden glob.

#### <a name="public-ip-addresses"></a>Offentliga IP-adresser

Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till slutpunkten för appens offentliga molnresurser.

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera mönstret:

### <a name="scalability"></a>Skalbarhet

Den viktigaste komponenten vid skalning mellan moln är möjligheten att leverera skalning på begäran. Skalning måste ske mellan offentlig och lokal molninfrastruktur och tillhandahålla en konsekvent och tillförlitlig tjänst per efterfrågan.

### <a name="availability"></a>Tillgänglighet

Se till att lokalt distribuerade appar konfigureras för hög tillgänglighet via lokal maskinvarukonfiguration och programvarudistribution.

### <a name="manageability"></a>Hanterbarhet

Mönstret mellan moln säkerställer sömlös hantering och ett bekant gränssnitt mellan miljöer.

## <a name="when-to-use-this-pattern"></a>När du ska använda det här mönstret

Använd det här mönstret:

- När du behöver öka appkapaciteten med oväntade krav eller periodiska behov.
- När du inte vill investera i resurser som endast kommer att användas under toppar. Betala för det du använder.

Det här mönstret rekommenderas inte när:

- Din lösning kräver att användare ansluter via Internet.
- Ditt företag har lokala föreskrifter som kräver att den ursprungliga anslutningen kommer från ett samtal på plats.
- Nätverket upplever vanliga flaskhalsar som begränsar skalningens prestanda.
- Din miljö är frånkopplad från Internet och kan inte nå det offentliga molnet.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- I [översikten Azure Traffic Manager mer](/azure/traffic-manager/traffic-manager-overview) information om hur den här DNS-baserade trafiklastbalanseraren fungerar.
- I [Designöverväganden för hybridprogram](overview-app-design-considerations.md) kan du läsa mer om metodtips och få svar på eventuella ytterligare frågor.
- I Azure Stack [med produkter och lösningar kan du](/azure-stack) läsa mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösningsexempel fortsätter du med distributionsguiden för [skalningslösningen mellan moln.](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling) Distributionsguiden innehåller stegvisa instruktioner för att distribuera och testa dess komponenter. Du lär dig hur du skapar en lösning mellan moln för att tillhandahålla en manuellt utlöst process för att växla från en Azure Stack Hub värdbaserade webbapp till en Azure-värdbaserade webbapp. Du får också lära dig hur du använder automatisk skalning via Traffic Manager, vilket säkerställer ett flexibelt och skalbart molnverktyg under belastning.