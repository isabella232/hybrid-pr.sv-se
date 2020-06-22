---
title: Mönster för skalning mellan moln (lokala data) i Azure Stack hubb
description: Lär dig hur du skapar en skalbar Cross-Cloud-app som använder lokal data i Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: edbb608fbf8e5288f29572bfe4cca98ffb3cb8fc
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911961"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Mönster för skalning mellan moln (lokala data)

Lär dig hur du skapar en hybrid app som omfattar Azure och Azure Stack Hub. Det här mönstret visar också hur du använder en enda lokal data källa för efterlevnad.

## <a name="context-and-problem"></a>Kontext och problem

Många organisationer samlar in och lagrar enorma mängder känslig kund information. De är ofta förhindrade att lagra känsliga data i det offentliga molnet på grund av företags föreskrifter eller myndighets principer. Dessa organisationer vill också dra nytta av skalbarheten för det offentliga molnet. Det offentliga molnet kan hantera säsongs toppar i trafik, så att kunderna kan betala för exakt den maskin vara de behöver, när de behöver det.

## <a name="solution"></a>Lösning

Lösningen drar nytta av kompatibiliteten för det privata molnet och kombinerar dem med skalbarheten för det offentliga molnet. Hybrid molnet Azure och Azure Stack Hub ger en konsekvent upplevelse för utvecklare. Denna konsekvens gör att de kan använda sina kunskaper både i offentliga moln och i lokala miljöer.

Med lösnings distributions guiden kan du distribuera en identisk webbapp till ett offentligt och privat moln. Du kan också komma åt ett icke-Internet-dirigerbart nätverk som finns på det privata molnet. Webbapparna övervakas för belastning. Vid en betydande ökning av trafiken manipulerar ett program DNS-poster för att dirigera trafik till det offentliga molnet. När trafiken inte längre är viktig uppdateras DNS-posterna för att dirigera trafiken tillbaka till det privata molnet.

[![Skalning mellan moln med lokal data mönster](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Komponenter

Den här lösningen använder följande komponenter:

| Lager | Komponent | Description |
|----------|-----------|-------------|
| Azure | Azure App Service | Med [Azure App Service](/azure/app-service/) kan du bygga och vara värd för webbappar, RESTful API-appar och Azure Functions. Alla i det programmeringsspråk som du väljer utan att hantera infrastrukturen. |
| | Azure Virtual Network| [Azure Virtual Network (VNet)](/azure/virtual-network/virtual-networks-overview) är det grundläggande Bygg blocket för privata nätverk i Azure. VNet möjliggör flera typer av Azure-resurser, till exempel virtuella datorer (VM), för att kommunicera på ett säkert sätt med varandra, Internet och lokala nätverk. Lösningen visar också användningen av ytterligare nätverks komponenter:<br>– app-och gateway-undernät.<br>– en lokal nätverksgateway.<br>– en virtuell nätverksgateway som fungerar som en plats-till-plats-VPN gateway-anslutning.<br>– en offentlig IP-adress.<br>– en punkt-till-plats-VPN-anslutning.<br>-Azure DNS för att vara värd för DNS-domäner och tillhandahålla namn matchning. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) är en DNS-baserad trafikbelastnings utjämning. Det gör att du kan styra distributionen av användar trafik för tjänst slut punkter i olika data Center. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) är en utöknings bar hanterings tjänst för program prestanda för webbutvecklare som skapar och hanterar appar på flera plattformar.|
| | Azure Functions | Med [Azure Functions](/azure/azure-functions/) kan du köra koden i en miljö utan server utan att först behöva skapa en virtuell dator eller publicera en webbapp. |
| | Autoskalning i Azure | [Autoskalning](/azure/azure-monitor/platform/autoscale-overview) är en inbyggd funktion i Cloud Services, virtuella datorer och webbappar. Funktionen gör att appar kan utföra sitt bästa när efter frågan ändras. Apparna kommer att justeras för trafik toppar och meddela dig när måtten ändras och skalas efter behov. |
| Azure Stack hubb | IaaS-beräkning | Med Azure Stack Hub kan du använda samma app-modell, självbetjänings Portal och API: er som aktive ras av Azure. Azure Stack Hub IaaS ger en mängd tekniker med öppen källkod för konsekventa hybrid moln distributioner. I lösnings exemplet används till exempel en virtuell Windows Server-dator för SQL Server.|
| | Azure App Service | Precis som Azure-webbappen använder lösningen [Azure App Service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) för att vara värd för webbappen. |
| | Nätverk | Azure Stack Hub-Virtual Network fungerar precis som Azure-Virtual Network. Den använder många av samma nätverks komponenter, inklusive anpassade värdnamn.
| Azure DevOps Services | Registrera dig | Konfigurera snabbt kontinuerlig integrering för build, test och distribution. Mer information finns i [Registrera dig och logga in på Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Azure Pipelines | Använd [Azure-pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) för kontinuerlig integrering/kontinuerlig leverans. Med Azure-pipeline kan du hantera värdbaserade bygg-och lanserings agenter och definitioner. |
| | Kodlagringsplats | Utnyttja flera kod databaser för att effektivisera din utvecklings pipeline. Använd befintliga kod databaser i GitHub, BitBucket, Dropbox, OneDrive och Azure databaser. |

## <a name="issues-and-considerations"></a>Problem och överväganden

Tänk på följande när du bestämmer hur du ska implementera den här lösningen:

### <a name="scalability"></a>Skalbarhet

Azure och Azure Stack Hub är unikt lämpade för att stödja behoven hos dagens globalt distribuerade företag.

#### <a name="hybrid-cloud-without-the-hassle"></a>Hybrid moln utan besväret

Microsoft erbjuder en oöverträffad-integrering av lokala till gångar med Azure Stack Hub och Azure i en enhetlig lösning. Den här integrationen eliminerar besväret med att hantera flera punkt lösningar och en blandning av moln leverantörer. Med skalning över molnet är kraften i Azure bara några klick bort. Anslut bara Azure Stack hubben till Azure med moln överföring och dina data och appar är tillgängliga i Azure när det behövs.

- Eliminera behovet av att bygga och underhålla en sekundär DR-webbplats.
- Spara tid och pengar genom att eliminera band säkerhets kopiering och hus upp till 99 års säkerhets kopierings data i Azure.
- Du kan enkelt migrera att köra Hyper-V, fysisk (i för hands version) och VMware (i för hands version) arbets belastningar i Azure för att dra nytta av den ekonomiska och elastiska molnets flexibilitet.
- Kör beräknings intensiva rapporter eller analyser på en replikerad kopia av din lokala till gång i Azure utan att påverka produktions arbets belastningar.
- Överför till molnet och kör lokala arbets belastningar i Azure, med större Compute-mallar när det behövs. Hybrid ger dig den kraft du behöver, när du behöver den.
- Skapa utvecklings miljöer på flera nivåer i Azure med några få klickningar – du kan även replikera Live-produktions data till din utvecklings-/test miljö för att hålla dem i nära real tids synkronisering.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Ekonomi för skalning mellan moln med Azure Stack hubb

Den främsta fördelen med moln överföring är ekonomiskt besparingar. Du betalar bara för de ytterligare resurserna när det finns ett behov av dessa resurser. Inga fler utgifter för onödig extra kapacitet eller försök att förutsäga toppar och fluktuationer i efter frågan.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Minska belastningen på hög belastning till molnet

Skalning över molnet kan användas för att begränsa bearbetningen av belastningar. Belastningen distribueras genom att flytta grundläggande appar till det offentliga molnet, vilket frigör lokala resurser för verksamhets kritiska appar. En app kan tillämpas på det privata molnet och sedan överföra enbart burst till det offentliga molnet när det är nödvändigt för att uppfylla kraven.

### <a name="availability"></a>Tillgänglighet

Global distribution har sina egna utmaningar, till exempel varierande anslutningar och olika myndighets bestämmelser efter region. Utvecklare kan utveckla en enda app och sedan distribuera den på olika orsaker med olika krav. Distribuera din app till det offentliga Azure-molnet och distribuera sedan ytterligare instanser eller komponenter lokalt. Du kan hantera trafik mellan alla instanser med Azure.

### <a name="manageability"></a>Hanterbarhet

#### <a name="a-single-consistent-development-approach"></a>En enda, konsekvent utvecklings metod

Med Azure och Azure Stack hubb kan du använda en konsekvent uppsättning utvecklingsverktyg i hela organisationen. Den här konsekvensen gör det enklare att implementera en metod för kontinuerlig integrering och kontinuerlig utveckling (CI/CD). Många appar och tjänster som distribueras i Azure eller Azure Stack hubb är utbytbara och kan köras på någon av platserna sömlöst.

En hybrid CI/CD-pipeline kan hjälpa dig att:

- Starta en ny version utifrån kod skrivningar till din kod lagrings plats.
- Distribuera automatiskt den nyskapade koden till Azure för att testa användar godkännande.
- När din kod har klarat testet distribuerar du den automatiskt till Azure Stack Hub.

### <a name="a-single-consistent-identity-management-solution"></a>En enda, konsekvent identitets hanterings lösning

Azure Stack Hub fungerar med både Azure Active Directory (Azure AD) och Active Directory Federation Services (AD FS) (ADFS). Azure Stack Hub fungerar med Azure AD i anslutna scenarier. För miljöer som inte har någon anslutning kan du använda ADFS som en frånkopplad lösning. Tjänstens huvud namn används för att bevilja åtkomst till appar, så att de kan distribuera eller konfigurera resurser via Azure Resource Manager.

### <a name="security"></a>Säkerhet

#### <a name="ensure-compliance-and-data-sovereignty"></a>Säkerställa efterlevnad och data suveränitet

Med Azure Stack Hub kan du köra samma tjänst i flera länder som om du använder ett offentligt moln. Genom att distribuera samma app i Data Center i varje land kan data suveränitets krav uppfyllas. Den här funktionen säkerställer att person uppgifter hålls inom varje lands gränser.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack hubb – säkerhet position

Det finns ingen säkerhets position utan en solid, kontinuerlig underhålls process. Därför har Microsoft investerat i en Orchestration-motor som tillämpar korrigeringar och uppdateringar sömlöst i hela infrastrukturen.

Tack vare partnerskap med Azure Stack Hub OEM-partners, utökar Microsoft samma säkerhets position till OEM-komponenter, t. ex. maskin varans livs cykel värd och den program vara som körs ovanpå den. Det här partnerskapet garanterar Azure Stack Hub har en enhetlig och robust säkerhets position över hela infrastrukturen. Kunder kan i sin tur bygga och skydda sina app-arbetsbelastningar.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Användning av tjänstens huvud namn via PowerShell, CLI och Azure Portal

Om du vill ge resurs åtkomst till ett skript eller en app konfigurerar du en identitet för din app och autentiserar appen med sina egna autentiseringsuppgifter. Den här identiteten kallas för tjänstens huvud namn och gör att du kan:

- Tilldela behörigheter till appens identitet som skiljer sig från dina egna behörigheter och som är begränsade till exakt appens behov.
- Använda ett certifikat för autentisering när du kör oövervakade skript.

Mer information om hur du skapar tjänstens huvud namn och använder ett certifikat för autentiseringsuppgifter finns i [använda en app-identitet för att få åtkomst till resurser](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>När du ska använda det här mönstret

- Min organisation använder en DevOps metod eller har en planerad för nära framtid.
- Jag vill implementera CI/CD-praxis över mitt Azure Stack Hub-implementering och det offentliga molnet.
- Jag vill konsolidera CI/CD-pipeline i molnet och i lokala miljöer.
- Jag vill kunna utveckla appar sömlöst med moln tjänster eller lokala tjänster.
- Jag vill utnyttja konsekventa kunskaper om utvecklare i molnet och lokala appar.
- Jag använder Azure men jag har utvecklare som arbetar i ett lokalt Azure Stack Hub-moln.
- Mina lokala appar upplevs på begäran under säsongs-, cykliska eller oförutsägbara fluktuationer.
- Jag har lokala komponenter och jag vill använda molnet för att skala dem sömlöst.
- Jag vill ha en skalbarhet i molnet, men jag vill att min app ska köras lokalt så mycket som möjligt.

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- Titta på [dynamiskt Skala appar mellan data Center och offentliga moln](https://www.youtube.com/watch?v=2lw8zOpJTn0) för en översikt över hur det här mönstret används.
- I [design överväganden för Hybrid appar](overview-app-design-considerations.md) kan du läsa mer om metod tips och få svar på fler frågor som du kan ha.
- Det här mönstret använder Azure Stack produkt familj, inklusive Azure Stack Hub. Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för globala moln lösningar (lokala data)](solution-deployment-guide-cross-cloud-scaling-onprem-data.md). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.
