---
title: Mönster för skalning mellan moln (lokala data) i Azure Stack Hub
description: Lär dig hur du skapar en skalbar app över flera moln som använder lokala data i Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281252"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Mönster för skalning mellan moln (lokala data)

Lär dig hur du skapar en hybridapp som sträcker sig över Azure och Azure Stack Hub. Det här mönstret visar också hur du använder en enda lokal datakälla för kompatibilitet.

## <a name="context-and-problem"></a>Kontext och problem

Många organisationer samlar in och lagrar enorma mängder känsliga kunddata. Ofta hindras de från att lagra känsliga data i det offentliga molnet på grund av företagsregler eller myndighetspolicyer. Dessa organisationer vill också dra nytta av skalbarheten i det offentliga molnet. Det offentliga molnet kan hantera säsongstoppar i trafiken, så att kunderna kan betala för exakt den maskinvara de behöver, när de behöver den.

## <a name="solution"></a>Lösning

Lösningen drar nytta av efterlevnadsfördelarna med det privata molnet och kombinerar dem med skalbarheten i det offentliga molnet. Azure och Azure Stack Hub hybridmoln ger en konsekvent upplevelse för utvecklare. Med den här konsekvensen kan de tillämpa sina kunskaper på både offentliga och lokala miljöer.

Med distributionsguiden för lösningen kan du distribuera en identisk webbapp till ett offentligt och privat moln. Du kan också komma åt ett dirigerbart nätverk som inte är internet som finns i det privata molnet. Webbapparna övervakas med avseende på belastning. Vid en betydande ökning av trafiken manipulerar ett program DNS-poster för att omdirigera trafik till det offentliga molnet. När trafiken inte längre är betydande uppdateras DNS-posterna för att dirigera trafiken tillbaka till det privata molnet.

[![Skalning mellan moln med datamönster på plats](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Komponenter

Den här lösningen använder följande komponenter:

| Skikt | Komponent | Beskrivning |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/) kan du skapa och vara värd för webbappar, RESTful-API-appar och Azure Functions. Allt i val av programmeringsspråk, utan att hantera infrastrukturen. |
| | Azure Virtual Network| [Azure Virtual Network (VNet) är](/azure/virtual-network/virtual-networks-overview) den grundläggande byggstenen för privata nätverk i Azure. VNet gör att flera Azure-resurstyper, till exempel virtuella datorer (VM), på ett säkert sätt kan kommunicera med varandra, Internet och lokala nätverk. Lösningen visar också användningen av ytterligare nätverkskomponenter:<br>– app- och gateway-undernät.<br>– en lokal lokal nätverksgateway.<br>– en virtuell nätverksgateway som fungerar som en vpn-gatewayanslutning från plats till plats.<br>– en offentlig IP-adress.<br>– en punkt-till-plats-VPN-anslutning.<br>– Azure DNS för att vara värd för DNS-domäner och tillhandahålla namnmatchning. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) är en lastbalanserare för DNS-baserad trafik. Du kan styra distributionen av användartrafik för tjänstslutpunkter i olika datacenter. |
| | Azure Application Insights | [Program Insights är](/azure/azure-monitor/app/app-insights-overview) en utökningsbar tjänst för hantering av programprestanda för webbutvecklare som skapar och hanterar appar på flera plattformar.|
| | Azure Functions | [Azure Functions](/azure/azure-functions/) kan du köra din kod i en serverlös miljö utan att först behöva skapa en virtuell dator eller publicera en webbapp. |
| | Autoskalning i Azure | [Autoskalning](/azure/azure-monitor/platform/autoscale-overview) är en inbyggd funktion i Cloud Services, virtuella datorer och webbappar. Med funktionen kan appar utföra sitt bästa när efterfrågan ändras. Apparna justeras efter trafiktoppar, vilket meddelar dig när måtten ändras och skalas efter behov. |
| Azure Stack Hub | IaaS-beräkning | Azure Stack Hub kan du använda samma appmodell, självbetjäningsportalen och API:er som aktiveras av Azure. Azure Stack Hub IaaS möjliggör en mängd olika tekniker med öppen källkod för konsekventa hybridmolndistributioner. I lösningsexempel används till Windows server-VM SQL Server till exempel.|
| | Azure App Service | Precis som Azure-webbappen använder lösningen Azure App Service [på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) som värd för webbappen. |
| | Nätverk | Den Azure Stack Hub Virtual Network fungerar precis som Azure-Virtual Network. Många av nätverkskomponenterna används, inklusive anpassade värdnamn.
| Azure DevOps Services | Registrera dig | Konfigurera snabbt kontinuerlig integrering för att skapa, testa och distribuera. Mer information finns i [Registrera dig och logga in på Azure DevOps.](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops) |
| | Azure-pipelines | Använd [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) för kontinuerlig integrering/kontinuerlig leverans. Med Azure Pipelines kan du hantera värdhanterade byggar- och lanseringsagenter och definitioner. |
| | Kodlagringsplats | Utnyttja flera koddatabaser för att effektivisera din utvecklingspipeline. Använd befintliga kodlagringsplatsen i GitHub, Bitbucket, Dropbox, OneDrive och Azure Repos. |

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

Azure och Azure Stack Hub är unikt lämpade för att stödja behoven hos dagens globalt distribuerade verksamhet.

#### <a name="hybrid-cloud-without-the-hassle"></a>Hybridmoln utan problem

Microsoft erbjuder en ovald integrering av lokala tillgångar med Azure Stack Hub och Azure i en enhetlig lösning. Den här integreringen eliminerar problemet med att hantera flera punktlösningar och en blandning av molnleverantörer. Med skalning mellan moln är kraften i Azure bara några klick bort. Anslut bara Azure Stack Hub azure med burst-data i molnet så blir dina data och appar tillgängliga i Azure när det behövs.

- Eliminera behovet av att skapa och underhålla en sekundär DR-plats.
- Spara tid och pengar genom att eliminera säkerhetskopiering på band och lagra upp till 99 års säkerhetskopieringsdata i Azure.
- Migrera enkelt arbetsbelastningarna Hyper-V, Fysisk (i förhandsversion) och VMware (i förhandsversion) till Azure för att dra nytta av ekonomin och elasticiteten i molnet.
- Kör beräkningsintensiva rapporter eller analyser på en replikerad kopia av din lokala tillgång i Azure utan att påverka produktionsarbetsbelastningar.
- Burst-in i molnet och kör lokala arbetsbelastningar i Azure, med större beräkningsmallar när det behövs. Hybrid ger dig den kraft du behöver när du behöver den.
- Skapa utvecklingsmiljöer på flera nivåer i Azure med några klick – du kan till och med replikera produktionsdata i realtid till din utvecklings-/testmiljö för att hålla den synkroniserad nästan i realtid.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Ekonomi med skalning mellan moln med Azure Stack Hub

Den största fördelen med burst-data i molnet är ekonomiska besparingar. Du betalar bara för de ytterligare resurserna när det finns ett behov av dessa resurser. Inga fler utgifter för onödig extra kapacitet eller försök att förutsäga toppar och variationer i efterfrågan.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Minska hög belastning på efterfrågan i molnet

Skalning mellan moln kan användas för bearbetning av bearbetning. Belastningen distribueras genom att grundläggande appar flyttas till det offentliga molnet, vilket frigör lokala resurser för affärskritiska appar. En app kan tillämpas på det privata molnet och sedan burst-kopplas till det offentliga molnet endast när det behövs för att uppfylla kraven.

### <a name="availability"></a>Tillgänglighet

Global distribution har sina egna utmaningar, till exempel variabel anslutning och olika myndighetsregler per region. Utvecklare kan utveckla bara en app och sedan distribuera den över olika orsaker med olika krav. Distribuera din app till det offentliga Azure-molnet och distribuera sedan ytterligare instanser eller komponenter lokalt. Du kan hantera trafik mellan alla instanser med hjälp av Azure.

### <a name="manageability"></a>Hanterbarhet

#### <a name="a-single-consistent-development-approach"></a>En enda konsekvent utvecklingsmetod

Med Azure Azure Stack Hub kan du använda en konsekvent uppsättning utvecklingsverktyg i hela organisationen. Den här konsekvensen gör det enklare att implementera en metod för kontinuerlig integrering och kontinuerlig utveckling (CI/CD). Många appar och tjänster som distribueras i Azure eller Azure Stack Hub är utbytbara och kan köras sömlöst på någon av platserna.

En CI/CD-hybridpipeline kan hjälpa dig att:

- Initiera en ny version baserat på kod commits till din koddatabas.
- Distribuera automatiskt den nya koden till Azure för testning av användargodkännande.
- När koden har testats distribuerar du automatiskt till Azure Stack Hub.

### <a name="a-single-consistent-identity-management-solution"></a>En enda, konsekvent identitetshanteringslösning

Azure Stack Hub fungerar med både Azure Active Directory (Azure AD) och Active Directory Federation Services (AD FS) (ADFS). Azure Stack Hub fungerar med Azure AD i anslutna scenarier. För miljöer som inte har någon anslutning kan du använda ADFS som en frånkopplad lösning. Tjänstens huvudnamn används för att bevilja åtkomst till appar, så att de kan distribuera eller konfigurera resurser via Azure Resource Manager.

### <a name="security"></a>Säkerhet

#### <a name="ensure-compliance-and-data-sovereignty"></a>Säkerställa efterlevnad och datasuveränitet

Azure Stack Hub kan du köra samma tjänst i flera länder som om du använder ett offentligt moln. Genom att distribuera samma app i datacenter i varje land kan krav på datasuveränitet uppfyllas. Den här funktionen säkerställer att personliga data förvaras inom varje lands gränser.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub – säkerhetsstatus

Det finns ingen säkerhetshållning utan en gedigen, kontinuerlig serviceprocess. Därför investerade Microsoft i en orkestreringsmotor som smidigt tillämpar korrigeringar och uppdateringar i hela infrastrukturen.

Tack vare samarbeten med Azure Stack Hub OEM-partner utökar Microsoft samma säkerhetsstatus till OEM-specifika komponenter, till exempel värden för maskinvarulivscykel och programvara som körs ovanpå den. Det här Azure Stack Hub säkerställer att Azure Stack Hub har en enhetlig och stabil säkerhetshållning i hela infrastrukturen. Kunder kan i sin tur skapa och skydda sina apparbetsbelastningar.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Användning av tjänstens huvudnamn via PowerShell, CLI och Azure Portal

Om du vill ge resursåtkomst till ett skript eller en app ställer du in en identitet för din app och autentiserar appen med sina egna autentiseringsuppgifter. Den här identiteten kallas tjänstens huvudnamn och gör att du kan:

- Tilldela behörigheter till den appidentitet som skiljer sig från dina egna behörigheter och som är begränsade till exakt appens behov.
- Använda ett certifikat för autentisering när du kör oövervakade skript.

Mer information om hur du skapar tjänstens huvudnamn och använder ett certifikat för autentiseringsuppgifter finns i [Använda en appidentitet för att få åtkomst till resurser.](/azure-stack/operator/azure-stack-create-service-principals)

## <a name="when-to-use-this-pattern"></a>När du ska använda det här mönstret

- Min organisation använder en DevOps-metod eller har en plan för den närmaste framtiden.
- Jag vill implementera CI/CD-metoder i min Azure Stack Hub implementering och det offentliga molnet.
- Jag vill konsolidera CI/CD-pipelinen i molnet och lokala miljöer.
- Jag vill kunna utveckla appar sömlöst med hjälp av molntjänster eller lokala tjänster.
- Jag vill utnyttja konsekventa utvecklarkunskaper i molnappar och lokala appar.
- Jag använder Azure men jag har utvecklare som arbetar i ett lokalt Azure Stack Hub moln.
- Mina lokala appar upplever toppar i efterfrågan under säsongsbaserade, cykliska eller oförutsägbara variationer.
- Jag har lokala komponenter och jag vill använda molnet för att skala dem sömlöst.
- Jag vill ha molnskalbarhet men jag vill att min app ska köras lokalt så mycket som möjligt.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- Titta [på hur du dynamiskt skalar appar mellan datacenter och offentliga](https://www.youtube.com/watch?v=2lw8zOpJTn0) moln för en översikt över hur det här mönstret används.
- Se [Designöverväganden för hybridapp](overview-app-design-considerations.md) för att lära dig mer om metodtips och för att besvara ytterligare frågor som du kan ha.
- Det här mönstret använder Azure Stack produktfamilj, inklusive Azure Stack Hub. Se Azure Stack [produkt- och lösningsfamilj för](/azure-stack) att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösningsexempel fortsätter du med distributionsguiden för skalningslösningen [mellan moln (lokala data).](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data) Distributionsguiden innehåller stegvisa instruktioner för att distribuera och testa dess komponenter.