---
title: Nivå data för analys mönster med Azure och Azure Stack hubb
description: Lär dig hur du använder Azure och Azure Stack Hub för att implementera en data lösning med flera nivåer i hybrid molnet.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912012"
---
# <a name="tiered-data-for-analytics-pattern"></a>Nivå data för analys mönster

Det här mönstret illustrerar hur du använder Azure Stack hubb och Azure för att mellanlagra, analysera, bearbeta, sanera och lagra data i flera lokala och molnbaserade platser.

## <a name="context-and-problem"></a>Kontext och problem

Ett av de problem som är riktade till företags organisationer i modern teknik landskap är att säkra data lagring, bearbetning och analys. Några överväganden är:

- data innehåll
- location
- krav på säkerhet och sekretess
- åtkomst behörigheter
- underhållskostnaderna
- lagrings lager hantering

Azure, i kombination med Azure Stack Hub, hanterar data problem och erbjuder lösningar med låg kostnad. Den här lösningen är bäst uttryckt via ett distribuerat tillverknings-eller logistik företag.

Lösningen baseras på följande scenario:

- En stor tillverknings organisation för flera grenar.
- Snabb och säker data lagring, bearbetning och distribution mellan globala fjärranslutna platser och det centrala huvud kontoret krävs.
- Aktivitet för anställda och maskiner, lokal information och affärs rapporterings data som måste vara skyddade. Data måste distribueras på lämpligt sätt och uppfylla regionala regler för efterlevnad och bransch bestämmelser.

## <a name="solution"></a>Lösning

Användning av både lokala och offentliga moln miljöer uppfyller kraven för företag med flera anläggningar. Azure Stack Hub erbjuder en snabb, säker och flexibel lösning för insamling, bearbetning, lagring och distribution av lokala och fjärranslutna data. Det här mönstret är särskilt användbart om säkerhet, sekretess, företags policy och myndighets krav kan skilja sig mellan platser och användare.

![Mönster för data skikt för analys av lösnings arkitektur](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Komponenter

Det här mönstret använder följande komponenter:

| Lager | Komponent | Description |
|----------|-----------|-------------|
| Azure | Storage | Ett [Azure Storage](/azure/storage/) -konto tillhandahåller en steril data förbruknings slut punkt. Azure Storage är Microsofts molntjänstlagringslösning för moderna datalagringsscenarier. Azure Storage erbjuder en massivt skalbar objekt lagring för data objekt och en fil system tjänst för molnet. Det innehåller också ett meddelande arkiv för Reliable Messaging och ett NoSQL-lager. |
| Azure Stack hubb | Storage | Ett [Azure Stack hubb lagrings](/azure-stack/user/azure-stack-storage-overview) konto används för flera tjänster:<br><br>- **Blob Storage** för rå data lagring. Blob Storage kan innehålla vilken typ av text eller binära data som helst, till exempel ett dokument, en mediefil eller ett installations program. Varje Blob organiseras under en behållare. Behållare är ett användbart sätt att tilldela säkerhets principer till grupper av objekt. Ett lagrings konto kan innehålla valfritt antal behållare, och en behållare kan innehålla valfritt antal blobbar, upp till kapacitets gränsen på 500 TB för lagrings kontot.<br>- **Blob Storage** för data arkiv. Det finns fördelar med låg kostnads lagring för arkivering av data. Exempel på häftiga data är säkerhets kopior, medie innehåll, vetenskapliga data, efterlevnad och Arkiv data. I allmänhet betraktas alla data som används sällan som cool Storage. Skikt data baserat på attribut som frekvens för åtkomst och kvarhållningsperiod. Kund information används sällan, men kräver liknande svars tid och prestanda för data.<br>- **Köa lagring** för bearbetning av data lagring. Queue Storage tillhandahåller moln meddelanden mellan app-komponenter. I utformningen av appar för skalning är app-komponenterna ofta fristående så att de kan skalas oberoende av varandra. Queue Storage levererar asynkrona meddelanden för kommunikation mellan program komponenter, oavsett om de körs i molnet, på Skriv bordet, på en lokal server eller på en mobil enhet. Queue Storage har också stöd för hantering av asynkrona åtgärder och utveckling av processarbetsflöden. |
| | Azure Functions | Tjänsten [Azure Functions](/azure/azure-functions/) tillhandahålls av [Azure App Service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) -resurs-providern. Med Azure Functions kan du köra din kod i en enkel, Server lös miljö som svar på en rad olika händelser. Azure Functions skala för att möta efter frågan utan att behöva skapa en virtuell dator eller publicera en webbapp med valfritt programmeringsspråk. Funktionerna används av lösningen för:<br><br>- **Data insugning**<br>- **Data sterilisering.** Manuellt utlösta funktioner kan utföra schemalagd data bearbetning, rensa och arkivera. Exempel kan vara skrubbning av listor över kund listor och månatlig rapport bearbetning.|

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

Azure Functions-och lagrings lösningar skalas för att uppfylla data volym-och bearbetnings krav. Information och mål för Azure-skalbarhet finns [Azure Storage skalbarhets dokumentation](/azure/storage/common/storage-scalability-targets).

### <a name="availability"></a>Tillgänglighet

Storage är det primära tillgänglighets övervägandet för det här mönstret. Anslutning via snabb länkar krävs för bearbetning och distribution av stora data volymer.

### <a name="manageability"></a>Hanterbarhet

Hantering av den här lösningen är beroende av redigerings verktyg som används och engagemang för käll kontroll.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- Se dokumentationen för [Azure Storage](/azure/storage/) och [Azure Functions](/azure/azure-functions/) . Det här mönstret gör att Azure Storage konton och Azure Functions används på både Azure-och Azure Stack-hubben.
- Se [design överväganden för hybrid program](overview-app-design-considerations.md) för att lära dig mer om bästa praxis och få svar på fler frågor.
- Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösnings exemplet kan du fortsätta med [distributions guiden för analys av lösningar](https://aka.ms/tiereddatadeploy). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.
