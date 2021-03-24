---
title: Hybrid mönster och lösnings exempel för Azure och Azure Stack Hub
description: En översikt över hybrid mönster och lösnings exempel för att lära och skapa hybrid lösningar på Azure och Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4f86e5ae4b8b9bd7693617b07419b67dfcf05dc1
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895320"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a>Hybrid mönster och lösnings exempel för Azure och Azure Stack

Microsoft tillhandahåller Azure och Azure Stack produkter och lösningar som ett enhetligt Azure-eko system. Microsoft Azure Stacks familjen är en utökning av Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Hybrid molnet och hybrid appar

Azure Stack ger dig flexibilitet för molnbaserad data behandling till din lokala miljö och gränsen genom att aktivera ett *hybrid moln*. Azure Stack Hub, Azure Stack HCI och Azure Stack Edge utöka Azure från molnet till våra data Center, avdelnings kontor, fält och annat. Med den här olika uppsättningen funktioner kan du:

- Återanvänd kod och kör molnbaserade appar konsekvent i Azure och i dina lokala miljöer.
- Kör traditionella virtualiserade arbets belastningar med valfria anslutningar till Azure-tjänster.
- Överför data till molnet eller behåll dem i det suveräna data centret för att upprätthålla efterlevnaden.
- Kör maskin vara som kan accelereras med maskin vara, containerbaserade eller virtualiserade arbets belastningar, allt på den intelligenta gränsen.

Appar som sträcker sig över moln kallas även *hybrid appar*. Du kan bygga hybrid molnappar i Azure och distribuera dem till din anslutna eller frånkopplade data Center var du än befinner dig.

Scenarier med hybrid program varierar kraftigt med de resurser som är tillgängliga för utveckling. De omfattar också överväganden som geografi, säkerhet, Internet åtkomst och andra. Även om mönstren och lösningarna som beskrivs här inte uppfyller alla krav, ger de rikt linjer och exempel för att utforska och återanvända samtidigt som de implementerar hybrid lösningar.

## <a name="design-patterns"></a>Designmönster

Design mönster utslagning generaliserad design vägledning, från verkliga världs kund scenarier och upplevelser. Ett mönster är abstrakt, vilket gör det möjligt att använda dem på olika typer av scenarier eller vertikala branscher. Varje mönster dokumenterar kontexten och problemet och ger en översikt över ett lösnings exempel. Lösnings exemplet är avsett som en möjlig implementering av mönstret.

Det finns två typer av mönster artiklar:

- Enstaka mönster: ger design vägledning för ett enda allmänt syfte.
- Multi-Pattern: ger design rikt linjer för hur du använder flera mönster. Det här mönstret krävs ofta för att lösa mer komplexa scenarier eller branschspecifika problem.

## <a name="solution-deployment-guides"></a>Guider för lösningsdistribution

Steg för steg-distributions guider hjälper till att distribuera ett lösnings exempel. Guiden kan också referera till ett kod exempel för assistenten som lagras i [exempel lagrings platsen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)för GitHub-lösningar.

## <a name="next-steps"></a>Nästa steg

- Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.
- Utforska avsnitten "mönster" och "lösnings distributions guider" i innehålls förteckningen om du vill veta mer om dem.
- Läs om [design överväganden för Hybrid appar](overview-app-design-considerations.md) om du vill läsa mer om program varu kvalitet för att utforma, distribuera och driva hybrid program.
- [Konfigurera en utvecklings miljö på Azure Stack](/azure-stack/user/azure-stack-dev-start) och [distribuera din första app](/azure-stack/user/azure-stack-dev-start-deploy-app) på Azure Stack.
