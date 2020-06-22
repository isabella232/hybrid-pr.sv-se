---
title: Mönster för skalning mellan moln i Azure Stack hubb
description: Lär dig hur du skapar en skalbar Cross-Cloud-App på Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911964"
---
# <a name="cross-cloud-scaling-pattern"></a>Mönster för skalning mellan moln

Lägg automatiskt till resurser i en befintlig app för att få en ökad belastning.

## <a name="context-and-problem"></a>Kontext och problem

Din app kan inte öka kapaciteten för att möta oväntade ökningar i efter frågan. Detta saknade skalbarhets resultat i användare som inte når appen under de högsta användnings tiderna. Appen kan betjäna ett fast antal användare.

Globala företag kräver säkra, pålitliga och tillgängliga molnbaserade appar. Mötet ökar efter frågan och använder rätt infrastruktur för att stödja att behovet är kritiskt. Affärs problem för att balansera kostnader och underhåll med affärs data säkerhet, lagring och tillgänglighet i real tid.

Du kanske inte kan köra din app i det offentliga molnet. Det kanske inte är ekonomiskt genomförbart för verksamheten att bibehålla kapaciteten som krävs i den lokala miljön för att hantera toppar i efter frågan på appen. Med det här mönstret kan du använda det offentliga molnets elastiskhet med din lokala lösning.

## <a name="solution"></a>Lösning

Mönstret för skalning mellan moln utökar en app som finns i ett lokalt moln med offentliga moln resurser. Mönstret utlöses av en ökning eller minskning i efter frågan, och lägger till eller tar bort resurser i molnet. Dessa resurser ger redundans, snabb tillgänglighet och geo-kompatibel routning.

![Mönster för skalning mellan moln](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Det här mönstret gäller endast för tillstånds lösa komponenter i din app.

## <a name="components"></a>Komponenter

Mönstret för skalning mellan moln består av följande komponenter.

### <a name="outside-the-cloud"></a>Utanför molnet

#### <a name="traffic-manager"></a>Traffic Manager

I diagrammet finns detta utanför den offentliga moln gruppen, men det skulle behöva kunna koordinera trafik i både det lokala data centret och det offentliga molnet. Balancer ger hög tillgänglighet för appen genom att övervaka slut punkter och tillhandahålla omdistribution vid fel vid behov.

#### <a name="domain-name-system-dns"></a>DNS (Domain Name System)

Domain Name System eller DNS ansvarar för översättning (eller matchning) av en webbplats eller ett tjänst namn till dess IP-adress.

### <a name="cloud"></a>Molnet

#### <a name="hosted-build-server"></a>Värdbaserad Bygg Server

En miljö som är värd för din build-pipeline.

#### <a name="app-resources"></a>App-resurser

App-resurserna måste kunna skalas in och ut, som skalnings uppsättningar och behållare för virtuella datorer.

#### <a name="custom-domain-name"></a>Anpassat domän namn

Använd ett anpassat domän namn för BLOB.

#### <a name="public-ip-addresses"></a>Offentliga IP-adresser

Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till resurs slut punkten för det offentliga molnet.  

### <a name="local-cloud"></a>Lokalt moln

#### <a name="hosted-build-server"></a>Värdbaserad Bygg Server

En miljö som är värd för din build-pipeline.

#### <a name="app-resources"></a>App-resurser

App-resurserna behöver möjlighet att skala in och ut, som skalnings uppsättningar och behållare för virtuella datorer.

#### <a name="custom-domain-name"></a>Anpassat domän namn

Använd ett anpassat domän namn för BLOB.

#### <a name="public-ip-addresses"></a>Offentliga IP-adresser

Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till resurs slut punkten för det offentliga molnet.

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera mönstret:

### <a name="scalability"></a>Skalbarhet

Huvud komponenten för skalning över moln är möjligheten att leverera skalning på begäran. Skalning måste ske mellan offentliga och lokala moln infrastrukturer och tillhandahålla en enhetlig, tillförlitlig tjänst per behov.

### <a name="availability"></a>Tillgänglighet

Se till att lokalt distribuerade appar är konfigurerade för hög tillgänglighet via lokal maskin varu konfiguration och program varu distribution.

### <a name="manageability"></a>Hanterbarhet

Mönstret mellan molnet säkerställer sömlös hantering och välbekanta gränssnitt mellan miljöer.

## <a name="when-to-use-this-pattern"></a>När du ska använda det här mönstret

Använd det här mönstret:

- När du behöver öka din app-kapacitet med oväntade krav eller periodiska krav på begäran.
- När du inte vill investera i resurser som endast kommer att användas under toppar. Betala för det du använder.

Det här mönstret rekommenderas inte när:

- Din lösning kräver att användare ansluter via Internet.
- Ditt företag har lokala bestämmelser som kräver att den ursprungliga anslutningen kommer från ett samtal på plats.
- Nätverket upplever vanliga Flask halsar som begränsar skalningens prestanda.
- Din miljö är frånkopplad från Internet och kan inte komma åt det offentliga molnet.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- I [översikten över Azure-Traffic Manager](/azure/traffic-manager/traffic-manager-overview) kan du läsa mer om hur den här DNS-baserade trafikbelastnings utjämningen fungerar.
- Se [design överväganden för hybrid program](overview-app-design-considerations.md) för att lära dig mer om bästa praxis och få svar på eventuella ytterligare frågor.
- Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för Cross-Cloud skalnings lösning](solution-deployment-guide-cross-cloud-scaling.md). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter. Du får lära dig hur du skapar en lösning för flera moln för att tillhandahålla en manuellt utlöst process för växling från en Azure Stack hubben webbapp till en Azure-värdbaserad webbapp. Du lär dig också hur du använder autoskalning via Traffic Manager, vilket garanterar flexibelt och skalbart moln verktyg vid belastning.
