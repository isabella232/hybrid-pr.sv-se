---
title: Mönster för geo-distribuerat program i Azure Stack Hub
description: Lär dig mer om den geo-distribuerade appens mönster för den intelligenta gränsen med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911856"
---
# <a name="geo-distributed-app-pattern"></a>Mönster för geo-distribuerat program

Lär dig att tillhandahålla app-slutpunkter i flera regioner och dirigera användar trafik baserat på plats-och efterlevnads behov.

## <a name="context-and-problem"></a>Kontext och problem

Organisationer med breda geografiska områden strävar efter säker och korrekt distribution och möjliggör åtkomst till data samtidigt som de säkerställer nödvändiga nivåer av säkerhet, efterlevnad och prestanda per användare, plats och enhet över gränserna.

## <a name="solution"></a>Lösning

Routnings mönstret för geografisk trafik i Azure Stack Hub, eller geo-distribuerade appar, gör att trafiken dirigeras till specifika slut punkter baserat på olika mått. Genom att skapa en Traffic Manager med geografisk och slut punkts konfiguration dirigeras trafiken till slut punkter baserat på regionala krav, företags-och internationella regler och data behov.

![Geo-distribuerat mönster](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Komponenter

### <a name="outside-the-cloud"></a>Utanför molnet

#### <a name="traffic-manager"></a>Traffic Manager

I diagrammet finns Traffic Manager utanför det offentliga molnet, men det måste kunna koordinera trafik i både det lokala data centret och det offentliga molnet. Saldobeloppet dirigerar trafik till geografiska platser.

#### <a name="domain-name-system-dns"></a>DNS (Domain Name System)

Domain Name System eller DNS ansvarar för översättning (eller matchning) av en webbplats eller ett tjänst namn till dess IP-adress.

### <a name="public-cloud"></a>Offentligt moln

#### <a name="cloud-endpoint"></a>Moln slut punkt

Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till resurs slut punkten för det offentliga molnet.  

### <a name="local-clouds"></a>Lokala moln

#### <a name="local-endpoint"></a>Lokal slut punkt

Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till resurs slut punkten för det offentliga molnet.

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera mönstret:

### <a name="scalability"></a>Skalbarhet

Mönstret hanterar geografisk trafikroutning i stället för att skalas för att möta ökningen av trafiken. Du kan dock kombinera det här mönstret med andra Azure-lösningar och lokala lösningar. Det här mönstret kan till exempel användas med mönstret för skalning mellan moln.

### <a name="availability"></a>Tillgänglighet

Se till att lokalt distribuerade appar är konfigurerade för hög tillgänglighet via lokal maskin varu konfiguration och program varu distribution.

### <a name="manageability"></a>Hanterbarhet

Mönstret säkerställer sömlös hantering och välbekanta gränssnitt mellan miljöer.

## <a name="when-to-use-this-pattern"></a>När du ska använda det här mönstret

- Min organisation har internationella grenar som kräver anpassade regionala säkerhets-och distributions principer.
- Var och en av organisationens kontor hämtar personal-, affärs-och anläggnings data, vilket kräver rapporterings aktivitet per lokal lagstiftning och tidszon.
- De storskaliga kraven kan uppfyllas genom horisontellt skala ut appar, med flera distributioner av appar som görs inom en region och mellan regioner för att hantera extrema belastnings krav.
- Apparna måste ha hög tillgänglighet och svara på klient begär Anden även i drifts avbrott i en region.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- I [översikten över Azure-Traffic Manager](/azure/traffic-manager/traffic-manager-overview) kan du läsa mer om hur den här DNS-baserade trafikbelastnings utjämningen fungerar.
- I [design överväganden för Hybrid appar](overview-app-design-considerations.md) kan du läsa mer om metod tips och få svar på eventuella ytterligare frågor.
- Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för geo-distribuerad app-lösning](solution-deployment-guide-geo-distributed.md). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter. Du får lära dig hur du dirigerar trafik till vissa slut punkter baserat på olika mått med hjälp av mönstret geo-distribuerat program. Genom att skapa en Traffic Manager profil med geografisk Routning och slut punkts konfiguration ser du till att information dirigeras till slut punkter baserat på regionala krav, företags-och internationella regler och dina data behov.
