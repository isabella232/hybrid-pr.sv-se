---
title: Hybrid relä mönster i Azure och Azure Stack hubb
description: Använd hybrid relä mönstret i Azure och Azure Stack Hub för att ansluta till kant resurser som skyddas av brand väggar.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911970"
---
# <a name="hybrid-relay-pattern"></a>Hybrid relä mönster

Lär dig hur du ansluter till Edge-resurser eller enheter som skyddas av brand väggar med hjälp av hybrid relä mönstret och Azure Relay.

## <a name="context-and-problem"></a>Kontext och problem

Gräns enheter är ofta bakom en företags brand vägg eller NAT-enhet. Även om de är säkra kan de inte kommunicera med det offentliga molnet eller gräns enheterna i andra företags nätverk. Det kan vara nödvändigt att exponera vissa portar och funktioner för användare i det offentliga molnet på ett säkert sätt.

## <a name="solution"></a>Lösning

Hybrid relä mönstret använder Azure Relay för att upprätta en WebSockets-tunnel mellan två slut punkter som inte kan kommunicera direkt. Enheter som inte är lokala men som måste anslutas till en lokal slut punkt ansluter till en slut punkt i det offentliga molnet. Den här slut punkten omdirigerar trafiken på fördefinierade vägar över en säker kanal. En slut punkt i den lokala miljön tar emot trafiken och dirigerar den till rätt mål.

![lösnings arkitektur för Hybrid relä mönster](media/pattern-hybrid-relay/solution-architecture.png)

Så här fungerar hybrid relä mönstret:

1. En enhet ansluter till den virtuella datorn (VM) i Azure på en fördefinierad port.
2. Trafiken vidarebefordras till Azure Relay i Azure.
3. Den virtuella datorn på Azure Stack Hub, som redan har upprättat en lång livs längd anslutning till Azure Relay, tar emot trafiken och vidarebefordrar den till målet.
4. Den lokala tjänsten eller slut punkten bearbetar begäran.

## <a name="components"></a>Komponenter

Den här lösningen använder följande komponenter:

| Lager | Komponent | Description |
|----------|-----------|-------------|
| Azure | Azure VM | En virtuell Azure-dator tillhandahåller en offentligt tillgänglig slut punkt för den lokala resursen. |
| | Azure Relay | En [Azure Relay](/azure/azure-relay/) tillhandahåller infrastrukturen för att underhålla tunneln och anslutningen mellan den virtuella Azure-datorn och Azure Stack Hub-VM.|
| Azure Stack hubb | Compute | En Azure Stack Hub-VM innehåller hybrid relä tunnelns Server sida. |
| | Storage | AKS-motorns kluster som distribueras till Azure Stack Hub ger en skalbar, elastisk motor för att köra Ansikts-API containern.|

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

Det här mönstret tillåter endast 1:1 port mappningar på klienten och servern. Om port 80 till exempel är tunnlad för en tjänst på Azure-slutpunkten kan den inte användas för en annan tjänst. Port mappningar bör därför planeras. Den Azure Relay och de virtuella datorerna bör skalas på rätt sätt för att hantera trafik.

### <a name="availability"></a>Tillgänglighet

Dessa tunnlar och anslutningar är inte redundanta. För att säkerställa hög tillgänglighet kanske du vill implementera fel söknings kod. Ett annat alternativ är att ha en pool med Azure Relay anslutna virtuella datorer bakom en belastningsutjämnare.

### <a name="manageability"></a>Hanterbarhet

Den här lösningen kan omfatta många enheter och platser, vilket kan ge svårhanterligt. Azures IoT-tjänster kan automatiskt placera nya platser och enheter online och hålla dem uppdaterade.

### <a name="security"></a>Säkerhet

Det här mönstret som visas ger oinskränkt åtkomst till en port på en intern enhet från kanten. Överväg att lägga till en autentiseringsmekanism till tjänsten på den interna enheten, eller framför hybrid relä slut punkten.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- Det här mönstret använder Azure Relay. Mer information finns i Azure Relay- [dokumentationen](/azure/azure-relay/).
- Se [design överväganden för hybrid program](overview-app-design-considerations.md) för att lära dig mer om bästa praxis och få svar på eventuella ytterligare frågor.
- Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för Hybrid relä lösning](https://aka.ms/hybridrelaydeployment). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.