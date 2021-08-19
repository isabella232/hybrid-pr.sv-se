---
title: Mönster för geo-distribuerade appar i Azure Stack Hub
description: Lär dig mer om det geo-distribuerade appmönstret för den intelligenta nätverkskanten med hjälp av Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281235"
---
# <a name="geo-distributed-app-pattern"></a>Mönster för geo-distribuerad app

Lär dig hur du tillhandahåller appslutpunkter i flera regioner och dirigerar användartrafik baserat på plats och efterlevnadsbehov.

## <a name="context-and-problem"></a>Kontext och problem

Organisationer med omfattande geografiska områden strävar efter att distribuera och ge åtkomst till data på ett säkert och korrekt sätt samtidigt som de säkerställer nödvändiga nivåer av säkerhet, efterlevnad och prestanda per användare, plats och enhet över gränser.

## <a name="solution"></a>Lösning

Med Azure Stack Hub trafikdirigeringsmönster eller geo-distribuerade appar kan trafiken dirigeras till specifika slutpunkter baserat på olika mått. Att skapa Traffic Manager med geografisk routning och slutpunktskonfiguration dirigerar trafik till slutpunkter baserat på regionala krav, företags- och internationella regelverk och databehov.

![Geo-distribuerat mönster](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Komponenter

### <a name="outside-the-cloud"></a>Utanför molnet

#### <a name="traffic-manager"></a>Traffic Manager

I diagrammet finns Traffic Manager utanför det offentliga molnet, men det måste kunna samordna trafik i både det lokala datacentret och det offentliga molnet. Balanseraren dirigerar trafik till geografiska platser.

#### <a name="domain-name-system-dns"></a>DNS (Domain Name System)

Den Domain Name System, eller DNS, ansvarar för att översätta (eller matcha) ett webbplats- eller tjänstnamn till dess IP-adress.

### <a name="public-cloud"></a>Offentligt moln

#### <a name="cloud-endpoint"></a>Molnslutpunkt

Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till slutpunkten för appens offentliga molnresurser.  

### <a name="local-clouds"></a>Lokala moln

#### <a name="local-endpoint"></a>Lokal slutpunkt

Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till slutpunkten för appens offentliga molnresurser.

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera mönstret:

### <a name="scalability"></a>Skalbarhet

Mönstret hanterar geografisk trafikroutning i stället för skalning för att möta trafikökningar. Du kan dock kombinera det här mönstret med andra Azure-lösningar och lokala lösningar. Det här mönstret kan till exempel användas med skalningsmönstret mellan moln.

### <a name="availability"></a>Tillgänglighet

Se till att lokalt distribuerade appar konfigureras för hög tillgänglighet via lokal maskinvarukonfiguration och programvarudistribution.

### <a name="manageability"></a>Hanterbarhet

Mönstret säkerställer sömlös hantering och ett bekant gränssnitt mellan miljöer.

## <a name="when-to-use-this-pattern"></a>När du ska använda det här mönstret

- Min organisation har internationella grenar som kräver anpassade regionala säkerhets- och distributionsprinciper.
- Var och en av min organisations kontor hämtar personal-, affärs- och anläggningsdata, vilket kräver rapporteringsaktivitet enligt lokala föreskrifter och tidszoner.
- Storskaliga krav kan uppfyllas genom horisontell utskalning av appar, där flera appdistributioner görs i en enda region och mellan regioner för att hantera extrema belastningskrav.
- Apparna måste ha hög tillgänglighet och svara på klientbegäranden även vid avbrott i en region.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- I [översikten Azure Traffic Manager mer](/azure/traffic-manager/traffic-manager-overview) information om hur den här DNS-baserade trafiklastbalanseraren fungerar.
- Se [Designöverväganden för hybridapp](overview-app-design-considerations.md) för att lära dig mer om metodtips och få svar på eventuella ytterligare frågor.
- I Azure Stack [med produkter och lösningar kan du](/azure-stack) läsa mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösningsexempel fortsätter du med [distributionsguiden för den geo-distribuerade applösningen.](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed) Distributionsguiden innehåller stegvisa instruktioner för att distribuera och testa dess komponenter. Du lär dig hur du dirigerar trafik till specifika slutpunkter baserat på olika mått med hjälp av det geo-distribuerade appmönstret. Genom att Traffic Manager profil med geografisk routning och slutpunktskonfiguration säkerställer du att information dirigeras till slutpunkter baserat på regionala krav, företags- och internationella regelverk och dina databehov.