---
title: Hybridmönster och lösningsexempel för Azure och Azure Stack Hub
description: En översikt över hybridmönster och lösningsexempel för att lära sig och skapa hybridlösningar i Azure och Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343866"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Hybridlösningsmönster och exempel för Azure och Azure Stack

Microsoft tillhandahåller Azure och Azure Stack produkter och lösningar som ett konsekvent Azure-ekosystem. Stack Microsoft Azure familjen är en utökning av Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Hybridmoln- och hybridappar

Azure Stack ger flexibiliteten hos molnbaserad databehandling till din lokala miljö och gränsen genom att aktivera ett *hybridmoln.* Azure Stack Hub, Azure Stack HCI och Azure Stack Edge Azure från molnet till dina suveräna datacenter, avdelningskontor, fält och mer. Med den här skilda uppsättningen funktioner kan du:

- Återanvänd kod och kör molnbaserade appar konsekvent i Azure och dina lokala miljöer.
- Kör traditionella virtualiserade arbetsbelastningar med valfria anslutningar till Azure-tjänster.
- Överför data till molnet eller behåll dem i ditt suveräna datacenter för att upprätthålla efterlevnad.
- Kör maskinvaruaccelererad maskininlärning, containerbaserade eller virtualiserade arbetsbelastningar på den intelligenta nätverkskanten.

Appar som sträcker sig över moln kallas även *hybridappar.* Du kan skapa hybridmolnappar i Azure och distribuera dem till ditt anslutna eller frånkopplade datacenter var som helst.

Hybridappscenarier varierar avsevärt med de resurser som är tillgängliga för utveckling. De omfattar även överväganden som geografi, säkerhet, Internetåtkomst med mera. Även om lösningsmönstren och exemplen som beskrivs här inte uppfyller alla krav ger de riktlinjer och exempel för att utforska och återanvända när du implementerar hybridlösningar.

## <a name="solution-patterns"></a>Lösningsmönster

Lösningsmönster och generaliserad, repeterbar designvägledning, från verkliga kundscenarier och upplevelser. Ett mönster är abstrakt, vilket gör att det kan tillämpas på olika typer av scenarier eller vertikala branscher. Varje mönster dokumenterar kontexten och problemet och ger en översikt över ett lösningsexempel. Lösningsexempel är avsett som en möjlig implementering av mönstret.

Det finns två typer av mönsterartiklar:

- Enskilt mönster: ger designvägledning för ett enda scenario för generell användning.
- Flera mönster: ger designvägledning där tillämpning av flera mönster används. Det här mönstret krävs ofta för att lösa mer komplexa scenarier eller branschspecifika problem.

## <a name="solution-deployment-guides"></a>Guider för lösningsdistribution

Stegvisa distributionsguider hjälper dig att distribuera ett lösningsexempel. Guiden kan även referera till ett tillhörande kodexempel som lagras i [GitHub-lösningsexempeldatabasen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).

## <a name="next-steps"></a>Nästa steg

- Se Azure Stack [produkt- och lösningsfamiljen för](/azure-stack) att lära dig mer om hela portföljen med produkter och lösningar.
- Utforska avsnitten "Mönster" och "Lösningsdistributionsguider" i toc för att lära dig mer om var och en.
- Läs mer om [designöverväganden för hybridappar](overview-app-design-considerations.md) för att granska grundpelare för programvarukvalitet för att utforma, distribuera och använda hybridappar.
- [Konfigurera en utvecklingsmiljö på Azure Stack](/azure-stack/user/azure-stack-dev-start) distribuera [din första app](/azure-stack/user/azure-stack-dev-start-deploy-app) på Azure Stack.
