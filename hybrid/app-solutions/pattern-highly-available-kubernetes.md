---
title: Kubernetes-mönster med hög tillgänglighet med Azure och Azure Stack Hub
description: Lär dig hur en Kubernetes-klusterlösning ger hög tillgänglighet med hjälp av Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281320"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Mönster för Kubernetes-kluster med hög tillgänglighet

Den här artikeln beskriver hur du skapar och använder en Kubernetes-baserad infrastruktur med hög Azure Kubernetes Service (AKS) på Azure Stack Hub. Det här scenariot är vanligt för organisationer med kritiska arbetsbelastningar i mycket begränsade och reglerade miljöer. Organisationer inom områden som ekonomi, försvar och myndigheter.

## <a name="context-and-problem"></a>Kontext och problem

Många organisationer utvecklar molnbaserade lösningar som utnyttjar de senaste tjänsterna och teknikerna som Kubernetes. Även om Azure tillhandahåller datacenter i de flesta regioner i världen, finns det ibland gränsfall och scenarier där affärskritiska program måste köras på en viss plats. Här är några saker att tänka på:

- Platskänslighet
- Svarstid mellan programmet och lokala system
- Bandbreddsbandbredd
- Anslutning
- Regelkrav eller lagstadgade krav

Azure, i kombination med Azure Stack Hub, hanterar de flesta av dessa problem. En bred uppsättning alternativ, beslut och överväganden för en lyckad implementering av Kubernetes som körs Azure Stack Hub beskrivs nedan.

## <a name="solution"></a>Lösning

Det här mönstret förutsätter att vi måste hantera en strikt uppsättning begränsningar. Programmet måste köras lokalt och alla personliga data får inte nå offentliga molntjänster. Övervakning och andra icke-PII-data kan skickas till Azure och bearbetas där. Externa tjänster som en offentlig Container Registry eller andra kan nås men kan filtreras via en brandvägg eller proxyserver.

Exempelprogrammet som visas här (baserat på [Azure Kubernetes Service Workshop)](/learn/modules/aks-workshop/)är utformat för att använda Kubernetes-interna lösningar när det är möjligt. Den här designen undviker leverantörslåsning i stället för att använda plattformsbaserade tjänster. Programmet använder till exempel en mongoDB-databasdel med egen värd i stället för en PaaS-tjänst eller en extern databastjänst.

[![Hybrid för programmönster](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Föregående diagram illustrerar programarkitekturen för exempelprogrammet som körs på Kubernetes Azure Stack Hub. Appen består av flera komponenter, bland annat:

 1) Ett AKS-motorbaserat Kubernetes-kluster Azure Stack Hub.
 2) [cert-manager](https://www.jetstack.io/cert-manager/), som tillhandahåller en uppsättning verktyg för certifikathantering i Kubernetes som används för att automatiskt begära certifikat från Let's Encrypt.
 3) Ett Kubernetes-namnområde som innehåller programkomponenterna för frontend (ratings-web), api (ratings-api) och databas (ratings-mongodb).
 4) Ingress-kontrollanten som dirigerar HTTP/HTTPS-trafik till slutpunkter i Kubernetes-klustret.

Exempelprogrammet används för att illustrera programarkitekturen. Alla komponenter är exempel. Arkitekturen innehåller bara en enda programdistribution. För att uppnå hög tillgänglighet (HA) kör vi distributionen minst två gånger på två olika Azure Stack Hub-instanser – de kan köras antingen på samma plats eller på två (eller flera) olika platser:

![Infrastrukturarkitektur](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Tjänster som Azure Container Registry, Azure Monitor och andra finns utanför Azure Stack Hub i Azure eller lokalt. Den här hybriddesignen skyddar lösningen mot avbrott i en enda Azure Stack Hub instans.

## <a name="components"></a>Komponenter

Den övergripande arkitekturen består av följande komponenter:

**Azure Stack Hub** är ett tillägg till Azure som kan köra arbetsbelastningar i en lokal miljö genom att tillhandahålla Azure-tjänster i ditt datacenter. Gå till [Azure Stack Hub för mer](/azure-stack/operator/azure-stack-overview) information.

**Azure Kubernetes Service Engine (AKS Engine)** är motorn bakom det hanterade Kubernetes-tjänsterbjudandet, Azure Kubernetes Service (AKS), som är tillgängligt i Azure i dag. Med Azure Stack Hub kan vi till exempel använda AKS Engine för att distribuera, skala och uppgradera fullständigt hanterade Kubernetes-kluster med hjälp av Azure Stack Hub:s IaaS-funktioner. Gå till [Översikt över AKS-motor om](https://github.com/Azure/aks-engine) du vill veta mer.

Gå till [Kända problem och begränsningar om](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) du vill veta mer om skillnaderna mellan AKS-motorn i Azure och AKS-motorn Azure Stack Hub.

**Azure Virtual Network (VNet)** används för att tillhandahålla nätverksinfrastrukturen på varje Azure Stack Hub för de Virtual Machines (VM) som är värdar för Kubernetes-klusterinfrastrukturen.

**Azure Load Balancer** används för Kubernetes API-slutpunkten och Nginx-indatakontrollanten. Lastbalanseraren dirigerar extern (till exempel Internet) trafik till noder och virtuella datorer som erbjuder en specifik tjänst.

**Azure Container Registry (ACR)** används för att lagra privata Docker-avbildningar och Helm-diagram som distribueras till klustret. AKS-motorn kan autentisera med Container Registry med hjälp av en Azure AD-identitet. Kubernetes kräver inte ACR. Du kan använda andra containerregister, till exempel Docker Hub.

**Azure Repos** är en uppsättning versionskontrollverktyg som du kan använda för att hantera din kod. Du kan också GitHub lagringsplatsen eller andra Git-baserade lagringsplatsen. Gå till [Översikt över Azure Repos om](/azure/devops/repos/get-started/what-is-repos) du vill veta mer.

**Azure Pipelines** är en del av Azure DevOps Services och kör automatiserade byggen, tester och distributioner. Du kan också använda CI/CD-lösningar från tredje part, till exempel Jenkins. Gå till [Översikt över Azure Pipeline om](/azure/devops/pipelines/get-started/what-is-azure-pipelines) du vill veta mer.

**Azure Monitor** samlar in och lagrar mått och loggar, inklusive plattformsmått för Azure-tjänsterna i lösningen och programtelemetri. Använd dessa data för att övervaka programmet, konfigurera aviseringar och instrumentpaneler och utföra rotorsaksanalys av fel. Azure Monitor integreras med Kubernetes för att samla in mått från kontrollanter, noder och containrar, samt containerloggar och huvudnodloggar. Gå till [Azure Monitor för mer](/azure/azure-monitor/overview) information.

**Azure Traffic Manager** är en lastbalanserare för DNS-baserad trafik som gör att du kan distribuera trafik optimalt till tjänster i olika Azure-regioner eller Azure Stack Hub distributioner. Traffic Manager ger också hög tillgänglighet och svarstider. Programslutpunkterna måste vara tillgängliga utifrån. Det finns även andra lokala lösningar.

**Kubernetes Ingress Controller** exponerar HTTP(S)-vägar för tjänster i ett Kubernetes-kluster. Nginx eller en lämplig ingress-kontrollant kan användas för detta ändamål.

**Helm** är en pakethanterare för Kubernetes-distribution som ger ett sätt att paketera olika Kubernetes-objekt som Distributioner, Tjänster, Hemligheter, i ett enda "diagram". Du kan publicera, distribuera, styra versionshantering och uppdatera ett diagramobjekt. Azure Container Registry kan användas som en lagringsplats för att lagra paketerade Helm-diagram.

## <a name="design-considerations"></a>Designöverväganden

Det här mönstret följer några viktiga överväganden som förklaras i detalj i nästa avsnitt i den här artikeln:

- Programmet använder Kubernetes-interna lösningar för att undvika leverantörslåsning.
- Programmet använder en mikrotjänstarkitektur.
- Azure Stack Hub behöver inte inkommande men tillåter utgående Internetanslutning.

Dessa rekommenderade metoder gäller även för verkliga arbetsbelastningar och scenarier.

## <a name="scalability-considerations"></a>Skalbarhetsöverväganden

Skalbarhet är viktigt för att ge användarna konsekvent, tillförlitlig och högpresterande åtkomst till programmet.

Exempelscenariot omfattar skalbarhet på flera lager i programstacken. Här är en översikt över de olika lagren:

| Arkitekturnivå | Påverkar | Hur gör jag? |
| --- | --- | ---
| Program | Program | Horisontell skalning baserat på antalet poddar/repliker/Container Instances* |
| Kluster | Kubernetes-kluster | Antal noder (mellan 1 och 50), VM-SKU-storlekar och nodpooler (AKS-motor på Azure Stack Hub stöder för närvarande endast en enda nodpool); använda AKS-motorns skalningskommando (manuellt) |
| Infrastruktur | Azure Stack Hub | Antal noder, kapacitet och skalningsenheter i en Azure Stack Hub distribution |

\* Använda HpA (Horizontal Pod Autoscaler) i Kubernetes automatiserad måttbaserad skalning eller vertikal skalning genom att ändra storlek på containerinstanserna (cpu/minne).

**Azure Stack Hub (infrastrukturnivå)**

Den Azure Stack Hub infrastrukturen är grunden för den här implementeringen, eftersom Azure Stack Hub körs på fysisk maskinvara i ett datacenter. När du väljer hubbmaskinvara måste du välja processor, minnesdensitet, lagringskonfiguration och antal servrar. Mer information Azure Stack Hub om skalbarheten finns i följande resurser:

- [Kapacitetsplanering för Azure Stack Hub översikt](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Lägga till ytterligare skalningsenhetsnoder i Azure Stack Hub](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes-kluster (klusternivå)**

Själva Kubernetes-klustret består av och bygger på Azure (Stack) IaaS-komponenter som beräknings-, lagrings- och nätverksresurser. Kubernetes-lösningar omfattar huvud- och arbetsnoder som distribueras som virtuella datorer i Azure (och Azure Stack Hub).

- [Kontrollplansnoder](/azure/aks/concepts-clusters-workloads#control-plane) (huvudnoder) tillhandahåller grundläggande Kubernetes-tjänster och orkestrering av programarbetsbelastningar.
- [Arbetsnoder](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (arbetsnoder) kör dina programarbetsbelastningar.

När du väljer VM-storlekar för den första distributionen finns det flera saker att tänka på:  

- **Kostnad** – När du planerar arbetsnoderna bör du tänka på den totala kostnaden per virtuell dator som du kommer att medföra. Om dina programarbetsbelastningar till exempel kräver begränsade resurser bör du planera att distribuera virtuella datorer av mindre storlek. Azure Stack Hub, till exempel Azure, debiteras vanligtvis per förbrukning, så det är viktigt att storleksändringen av de virtuella datorerna för Kubernetes-roller är lämplig för att optimera förbrukningskostnaderna. 

- **Skalbarhet** – klustrets skalbarhet uppnås genom att skala in och ut antalet huvudnoder och arbetsnoder, eller genom att lägga till ytterligare nodpooler (inte tillgängligt på Azure Stack Hub idag). Du kan skala klustret baserat på prestandadata som samlas in med hjälp av Container Insights (Azure Monitor + Log Analytics). 

    Om programmet behöver fler (eller färre) resurser kan du skala ut (eller in) dina aktuella noder vågrätt (mellan 1 och 50 noder). Om du behöver fler än 50 noder kan du skapa ytterligare ett kluster i en separat prenumeration. Du kan inte skala upp de faktiska virtuella datorerna lodrätt till en annan VM-storlek utan att omdistribuera klustret.

    Skalning görs manuellt med hjälp av den virtuella AKS Engine-hjälpdatorn som användes för att distribuera Kubernetes-klustret från början. Mer information finns i [Skala Kubernetes-kluster](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Kvoter** – Överväg de [kvoter](/azure-stack/operator/azure-stack-quota-types) som du har konfigurerat när du planerar en AKS-distribution på Azure Stack Hub. Kontrollera att varje [prenumeration](/azure-stack/operator/service-plan-offer-subscription-overview) har rätt planer och kvoter konfigurerade. Prenumerationen måste hantera mängden beräkning, lagring och andra tjänster som behövs för dina kluster när de skalas ut.

- **Programarbetsbelastningar** – Se begreppen [kluster och arbetsbelastningar](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) i kubernetes-kärnbegreppen för Azure Kubernetes Service dokument. Den här artikeln hjälper dig att begränsa storleken på den virtuella datorn baserat på programmets beräknings- och minnesbehov.  

**Program (programnivå)**

På programlagret använder vi Kubernetes [Horizontal Pod Autoscaler (HPA).](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) HPA kan öka eller minska antalet repliker (pod/Container Instances) i vår distribution baserat på olika mått (t.ex. processoranvändning).

Ett annat alternativ är att skala containerinstanser lodrätt. Detta kan åstadkommas genom att ändra mängden processor- och minne som begärs och är tillgängligt för en specifik distribution. Se [Hantera resurser för containrar](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) på kubernetes.io mer information.

## <a name="networking-and-connectivity-considerations"></a>Nätverks- och anslutningsöverväganden

Nätverk och anslutning påverkar också de tre lager som nämnts tidigare för Kubernetes Azure Stack Hub. I följande tabell visas lagren och vilka tjänster de innehåller:

| Skikt | Påverkar | Vad? |
| --- | --- | ---
| Program | Program | Hur är programmet tillgängligt? Kommer den att exponeras mot Internet? |
| Kluster | Kubernetes-kluster | Kubernetes API, AKS Engine VM, Hämta containeravbildningar (utgående), Skicka övervakningsdata och telemetri (utgående) |
| Infrastruktur | Azure Stack Hub | Tillgängligheten för Azure Stack Hub hanteringsslutpunkter som portalen och Azure Resource Manager slutpunkter. |

**Program**

För programlagret är det viktigaste att överväga om programmet är exponerat och tillgängligt från Internet. Från ett Kubernetes-perspektiv innebär Internettillgänglighet att exponera en distribution eller podd med hjälp av en Kubernetes-tjänst eller en ingress-kontrollant.

> [!NOTE]
> Vi rekommenderar att du använder indatastyrenheter för att exponera Kubernetes-tjänster eftersom antalet offentliga IP-adresser i Azure Stack Hub är begränsat till 5. Den här designen begränsar också antalet Kubernetes Services (med typen LoadBalancer) till 5, vilket blir för litet för många distributioner. Mer information finns [i dokumentationen för AKS Engine.](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips)

Att exponera ett program med en offentlig IP-adress via en Load Balancer eller en ingress-kontrollant innebär inte att programmet nu är tillgängligt via Internet. Det är möjligt för Azure Stack Hub att ha en offentlig IP-adress som bara är synlig på det lokala intranätet – alla offentliga IP-adresser är inte internetriktade.

Det föregående blocket tar hänsyn till ingående trafik till programmet. Ett annat ämne som måste övervägas för en lyckad Kubernetes-distribution är utgående/utgående trafik. Här är några användningsfall som kräver utgående trafik:

- Hämta containeravbildningar som lagras på DockerHub eller Azure Container Registry
- Hämta Helm-diagram
- Skicka program Insights data (eller andra övervakningsdata)

Vissa företagsmiljöer kan kräva användning av _transparenta eller_ _icke-transparenta_ proxyservrar. Dessa servrar kräver specifik konfiguration på olika komponenter i vårt kluster. AKS Engine-dokumentationen innehåller olika detaljer om hur du kan hantera nätverk proxy. Mer information finns i [AKS-motorn och proxyservrar](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Slutligen måste trafik mellan kluster flöda mellan Azure Stack Hub instanser. Exempeldistributionen består av enskilda Kubernetes-kluster som körs på Azure Stack Hub instanser. Trafiken mellan dem, till exempel replikeringstrafiken mellan två databaser, är "extern trafik". Extern trafik måste dirigeras via antingen ett VPN för plats till plats eller Azure Stack Hub offentliga IP-adresser för att ansluta Kubernetes på två Azure Stack Hub instanser:

![trafik mellan och inom kluster](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Kluster**  

Kubernetes-klustret behöver inte nödvändigtvis vara tillgängligt via Internet. Den relevanta delen är Kubernetes-API:et som används för att driva ett kluster, till exempel med hjälp av `kubectl` . Kubernetes API-slutpunkten måste vara tillgänglig för alla som driver klustret eller distribuerar program och tjänster ovanpå det. Det här avsnittet beskrivs i detalj ur ett DevOps-perspektiv i avsnittet [Överväganden för distribution (CI/CD)](#deployment-cicd-considerations) nedan.

På klusternivå finns det också några saker att tänka på när det gäller utgående trafik:

- Noduppdateringar (för Ubuntu)
- Övervakningsdata (skickas till Azure LogAnalytics)
- Andra agenter som kräver utgående trafik (specifik för varje distribuerarmiljö)

Innan du distribuerar Kubernetes-klustret med AKS Engine bör du planera för den slutliga nätverksdesignen. I stället för att skapa Virtual Network dedikerade nätverk kan det vara mer effektivt att distribuera ett kluster till ett befintligt nätverk. Du kan till exempel använda en befintlig VPN-anslutning från plats till plats som redan har konfigurerats i din Azure Stack Hub miljö.

**Infrastruktur**  

Infrastruktur syftar på åtkomst till Azure Stack Hub slutpunkter för hantering. Slutpunkter omfattar klient- och administratörsportalerna och slutpunkterna för Azure Resource Manager administratör och klientorganisation. Dessa slutpunkter krävs för att Azure Stack Hub och dess kärntjänster.

## <a name="data-and-storage-considerations"></a>Överväganden för data och lagring

Två instanser av vårt program distribueras på två enskilda Kubernetes-kluster över två Azure Stack Hub instanser. Den här designen kräver att vi överväger hur data ska replikeras och synkroniseras mellan dem.

Med Azure har vi den inbyggda funktionen för att replikera lagring över flera regioner och zoner i molnet. För närvarande Azure Stack Hub det inte finns några interna sätt att replikera lagring över två olika Azure Stack Hub-instanser – de utgör två oberoende moln utan något övergripande sätt att hantera dem som en uppsättning. Genom att planera för återhämtning av program som körs Azure Stack Hub du att tänka på detta oberoende i din programdesign och distribution.

I de flesta fall är lagringsreplikering inte nödvändigt för ett elastiskt program med hög tillgänglig distribution i AKS. Men du bör överväga oberoende lagring per Azure Stack Hub instans i programdesignen. Om den här designen är ett problem eller en väg att distribuera lösningen på Azure Stack Hub finns det möjliga lösningar från Microsoft-partner som tillhandahåller lagringsbilagor. Storage bifogade filer ger en lösning för lagringsreplikering över flera Azure Stack Hubs och Azure. Mer information finns i [Partnerlösningar.](#partner-solutions)

I vår arkitektur har dessa lager övervägts:

**Konfiguration**

Konfigurationen omfattar konfigurationen av Azure Stack Hub, AKS Engine och själva Kubernetes-klustret. Konfigurationen bör automatiseras så mycket som möjligt och lagras som infrastruktur som kod i ett Git-baserat versionskontrollsystem som Azure DevOps eller GitHub. De här inställningarna kan inte enkelt synkroniseras över flera distributioner. Därför rekommenderar vi att du lagrar och tillämpar konfiguration utifrån och använder DevOps-pipeline.

**Program**

Programmet ska lagras på en Git-baserad lagringsplats. När det finns en ny distribution, ändringar i programmet eller haveriberedskap kan den enkelt distribueras med hjälp av Azure Pipelines.

**Data**

Data är den viktigaste faktorn i de flesta programdesigner. Programdata måste vara synkroniserade mellan de olika instanserna av programmet. Data behöver också en strategi för säkerhetskopiering och haveriberedskap vid ett avbrott.

Att uppnå den här designen beror till stor del på teknikval. Här följer några lösningsexempel för att implementera en databas på ett sätt med hög Azure Stack Hub:

- [Distribuera en SQL Server 2016-tillgänglighetsgrupp till Azure och Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Distribuera en MongoDB-lösning med hög tillgång till Azure och Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Att tänka på när du arbetar med data på flera platser är ett ännu mer komplext övervägande för en lösning med hög tillgång och återhämtning. Tänk på att:

- Svarstid och nätverksanslutning mellan Azure Stack Hubs.
- Tillgänglighet för identiteter för tjänster och behörigheter. Varje Azure Stack Hub-instans integreras med en extern katalog. Under distributionen väljer du att använda antingen Azure Active Directory (Azure AD) eller Active Directory Federation Services (AD FS) (ADFS). Därför finns det möjlighet att använda en enda identitet som kan interagera med flera oberoende Azure Stack Hub instanser.

## <a name="business-continuity-and-disaster-recovery"></a>Affärskontinuitet och haveriberedskap

Affärskontinui och haveriberedskap (BCDR) är ett viktigt ämne i både Azure Stack Hub och Azure. Den största skillnaden är att i Azure Stack Hub måste operatorn hantera hela BCDR-processen. I Azure hanteras delar av BCDR automatiskt av Microsoft.

BCDR påverkar samma områden som nämns i föregående avsnitt Data- [och lagringsöverväganden:](#data-and-storage-considerations)

- Infrastruktur/konfiguration
- Programtillgänglighet
- Programdata

Som vi nämnde i föregående avsnitt ansvarar dessa områden för Azure Stack Hub och kan variera mellan olika organisationer. Planera BCDR enligt dina tillgängliga verktyg och processer.

**Infrastruktur och konfiguration**

Det här avsnittet beskriver den fysiska och logiska infrastrukturen och konfigurationen av Azure Stack Hub. Den omfattar åtgärder i administratörs- och klientorganisationsutrymmena.

Den Azure Stack Hub (eller administratören) ansvarar för underhåll av Azure Stack Hub instanser. Inklusive komponenter som nätverk, lagring, identitet och andra ämnen som ligger utanför omfånget för den här artikeln. Mer information om Azure Stack Hub finns i följande resurser:

- [Återställa data i Azure Stack Hub med Infrastructure Backup service](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Aktivera säkerhetskopiering Azure Stack Hub från administratörsportalen](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Återställa från oåterkallelig dataförlust](/azure-stack/operator/azure-stack-backup-recover-data)
- [Metodtips för Infrastructure Backup-tjänsten](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub är den plattform och infrastruktur som Kubernetes-program ska distribueras på. Kubernetes-programmets programägare är en användare av Azure Stack Hub, med åtkomst beviljad för att distribuera den programinfrastruktur som behövs för lösningen. Programinfrastruktur innebär i det här fallet Kubernetes-klustret, distribuerat med AKS Engine och de omgivande tjänsterna. Dessa komponenter distribueras i Azure Stack Hub, begränsade av ett Azure Stack Hub erbjudande. Kontrollera att erbjudandet som accepterats av Kubernetes-programägaren har tillräckligt med kapacitet (uttryckt i Azure Stack Hub kvoter) för att distribuera hela lösningen. Som vi rekommenderar i föregående avsnitt bör programdistributionen automatiseras med hjälp av pipelines för infrastruktur som kod och distribution som Azure DevOps Azure Pipelines.

Mer information om Azure Stack Hub och kvoter finns i [översikten Azure Stack Hub tjänster, planer, erbjudanden och prenumerationer](/azure-stack/operator/service-plan-offer-subscription-overview)

Det är viktigt att spara och lagra AKS-motorns konfiguration på ett säkert sätt, inklusive dess utdata. De här filerna innehåller konfidentiell information som används för åtkomst till Kubernetes-klustret, så de måste skyddas från att exponeras för icke-administratörer.

**Programtillgänglighet**

Programmet ska inte förlita sig på säkerhetskopior av en distribuerad instans. Som standard distribuerar du om programmet helt enligt M2P-mönster (Infrastruktur som kod). Du kan till exempel distribuera om med Azure DevOps Azure Pipelines. BCDR-proceduren bör omfatta omdistribution av programmet till samma eller ett annat Kubernetes-kluster.

**Programdata**

Programdata är den viktigaste delen för att minimera dataförlust. I föregående avsnitt beskrevs metoder för att replikera och synkronisera data mellan två (eller flera) instanser av programmet. Beroende på databasinfrastrukturen (MySQL, MongoDB, MSSQL eller andra) som används för att lagra data, finns det olika tillgängliga tekniker för databastillgänglighet och säkerhetskopiering att välja mellan.

De rekommenderade sätten att uppnå integritet är att använda antingen:
- En inbyggd säkerhetskopieringslösning för den specifika databasen.
- En säkerhetskopieringslösning som officiellt stöder säkerhetskopiering och återställning av den databastyp som används av ditt program.

> [!IMPORTANT]
> Lagra inte dina säkerhetskopierade data på samma Azure Stack Hub instans där dina programdata finns. Ett fullständigt avbrott i den Azure Stack Hub instansen skulle också kompromettera dina säkerhetskopior.

## <a name="availability-considerations"></a>Överväganden för tillgänglighet

Kubernetes på Azure Stack Hub som distribueras via AKS Engine är inte en hanterad tjänst. Det är en automatiserad distribution och konfiguration av ett Kubernetes-kluster med hjälp av Azure IaaS (Infrastruktur som en tjänst). Därför ger den samma tillgänglighet som den underliggande infrastrukturen.

Azure Stack Hub infrastruktur är redan motståndskraftig mot fel och innehåller funktioner som tillgänglighetsuppsättningar för att distribuera komponenter över flera [fel- och uppdateringsdomäner.](/azure-stack/user/azure-stack-vm-considerations#high-availability) Men den underliggande tekniken (redundansklustring) medför fortfarande vissa driftstopp för virtuella datorer på en fysisk server som påverkas, om det uppstår ett maskinvarufel.

Det är en bra idé att distribuera kubernetes-klustret för produktion samt arbetsbelastningen till två (eller flera) kluster. Dessa kluster bör finnas på olika platser eller datacenter och använda tekniker som Azure Traffic Manager för att dirigera användare baserat på klustersvarstid eller geografiskt område.

![Använda Traffic Manager för att styra trafikflöden](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Kunder som har ett enda Kubernetes-kluster ansluter vanligtvis till tjänstens IP- eller DNS-namn för ett visst program. I en distribution med flera kluster bör kunderna ansluta till ett Traffic Manager DNS-namn som pekar på tjänsterna/ingressen på varje Kubernetes-kluster.

![Använda Traffic Manager för att dirigera till ett lokalt kluster](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Det här mönstret är också [bästa praxis för (hanterade) AKS-kluster i Azure](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

Själva Kubernetes-klustret, som distribueras via AKS-motorn, bör bestå av minst tre huvudnoder och två arbetsnoder.

## <a name="identity-and-security-considerations"></a>Identitets- och säkerhetsöverväganden

Identitet och säkerhet är viktiga ämnen. Särskilt när lösningen omfattar oberoende Azure Stack Hub instanser. Kubernetes och Azure (inklusive Azure Stack Hub) har båda olika mekanismer för rollbaserad åtkomstkontroll (RBAC):

- Azure RBAC styr åtkomsten till resurser i Azure (och Azure Stack Hub), inklusive möjligheten att skapa nya Azure-resurser. Behörigheter kan tilldelas till användare, grupper eller tjänstens huvudnamn. (Tjänstens huvudnamn är en säkerhetsidentitet som används av program.)
- Kubernetes RBAC styr behörigheter till Kubernetes-API:et. Att till exempel skapa poddar och lista poddar är åtgärder som kan auktoriseras (eller nekas) till en användare via RBAC. Om du vill tilldela Kubernetes-behörigheter till användare skapar du roller och rollbindningar.

**Azure Stack Hub identitet och RBAC**

Azure Stack Hub finns två alternativ för identitetsprovidern. Vilken provider du använder beror på miljön och om den körs i en ansluten eller frånkopplad miljö:

- Azure AD – kan bara användas i en ansluten miljö.
- ADFS till en traditionell Active Directory-skog – kan användas i både en ansluten eller frånkopplad miljö.

Identitetsprovidern hanterar användare och grupper, inklusive autentisering och auktorisering för åtkomst till resurser. Åtkomst kan beviljas för Azure Stack Hub resurser som prenumerationer, resursgrupper och enskilda resurser som virtuella datorer eller lastbalanserare. Om du vill ha en konsekvent åtkomstmodell bör du överväga att använda samma grupper (direkt eller kapslade) för alla Azure Stack Hubs. Här är ett konfigurationsexempel:

![kapslade aad-grupper med Azure Stack Hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

Exemplet innehåller en dedikerad grupp (med AAD eller ADFS) för ett specifikt syfte. Du kan till exempel ange deltagarbehörigheter för resursgruppen som innehåller vår Kubernetes-klusterinfrastruktur på en specifik Azure Stack Hub instans (här "Seattle K8s Cluster Contributor"). Dessa grupper kapslas sedan in i en övergripande grupp som innehåller "undergrupper" för varje Azure Stack Hub.

Vår exempelanvändare har nu behörigheten Deltagare till båda resursgrupperna som innehåller hela uppsättningen Kubernetes-infrastrukturresurser. Användaren kommer att ha åtkomst till resurser Azure Stack Hub båda instanserna, eftersom instanserna delar samma identitetsprovider.

> [!IMPORTANT]
> Dessa behörigheter påverkar Azure Stack Hub och vissa av de resurser som distribueras ovanpå den. En användare som har den här åtkomstnivån kan göra stor skada, men kan inte komma åt de virtuella Kubernetes IaaS-datorerna eller Kubernetes-API:et utan ytterligare åtkomst till Kubernetes-distributionen.

**Kubernetes-identitet och RBAC**

Ett Kubernetes-kluster använder som standard inte samma identitetsprovider som den Azure Stack Hub. De virtuella datorer som är värdar för Kubernetes-klustret, huvudnoderna och arbetsnoderna använder den SSH-nyckel som anges under distributionen av klustret. Den här SSH-nyckeln krävs för att ansluta till dessa noder med hjälp av SSH.

Kubernetes-API:et (till exempel som nås med hjälp av ) skyddas också av tjänstkonton, inklusive ett `kubectl` standardkonto för tjänsten "klusteradministratör". Autentiseringsuppgifterna för det här tjänstkontot lagras inledningsvis `.kube/config` i filen på dina Kubernetes-huvudnoder.

**Hantering av hemligheter och autentiseringsuppgifter för program**

Om du vill lagra hemligheter som anslutningssträngar eller databasautentiseringsuppgifter finns det flera alternativ, bland annat:

- Azure Key Vault
- Kubernetes-hemligheter
- Tredjepartslösningar som HashiCorp Vault (körs på Kubernetes)

Lagra inte hemligheter eller autentiseringsuppgifter i klartext i konfigurationsfiler, programkod eller i skript. Och lagra dem inte i ett versionskontrollsystem. I stället bör distributionsautomationen hämta hemligheterna efter behov.

## <a name="patch-and-update"></a>Korrigera och uppdatera

**PNU-processen (Patch** and Update) i Azure Kubernetes Service delvis automatiserad. Kubernetes-versionsuppgraderingar utlöses manuellt, medan säkerhetsuppdateringar tillämpas automatiskt. Dessa uppdateringar kan innehålla os-säkerhetskorrigeringar eller kerneluppdateringar. AKS startar inte om dessa Linux-noder automatiskt för att slutföra uppdateringsprocessen. 

PNU-processen för ett Kubernetes-kluster som distribueras med AKS Engine på Azure Stack Hub är ohanterad och är klusteroperatorns ansvar. 

AKS-motorn hjälper till med de två viktigaste uppgifterna:

- [Uppgradera till en nyare Kubernetes- och basversion av OPERATIVSYSTEMavbildningen](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Uppgradera endast basoperativsystemavbildningen](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Nyare basoperativsystemavbildningar innehåller de senaste korrigeringarna av operativsystemssäkerhet och kerneluppdateringar. 

Mekanismen [för obevakad](https://wiki.debian.org/UnattendedUpgrades) uppgradering installerar automatiskt säkerhetsuppdateringar som släpps innan en ny basversion av operativsystemavbildningen är tillgänglig i Azure Stack Hub Marketplace. Obevakad uppgradering är aktiverat som standard och installerar säkerhetsuppdateringar automatiskt, men startar inte om Kubernetes-klusternoderna. Omstart av noderna kan automatiseras med hjälp av den öppna källkoden [ **K** Ubernetes **RE** boot **D** aemon (kurerad))](/azure/aks/node-updates-kured). Den avdelade söker efter Linux-noder som kräver en omstart och hanterar sedan automatiskt omläggningen av poddar som körs och omstarten av noden.

## <a name="deployment-cicd-considerations"></a>Överväganden för distribution (CI/CD)

Azure och Azure Stack Hub exponerar samma Azure Resource Manager REST-API:er. Dessa API:er behandlas precis som andra Azure-moln (Azure, Azure China 21Vianet, Azure Government). Det kan finnas skillnader i API-versioner mellan moln och Azure Stack Hub endast tillhandahåller en delmängd av tjänsterna. Slutpunkts-URI för hantering är också olika för varje moln och varje instans av Azure Stack Hub.

Förutom de små skillnader som nämnts ger Azure Resource Manager REST-API:er ett konsekvent sätt att interagera med både Azure och Azure Stack Hub. Samma uppsättning verktyg kan användas här som med andra Azure-moln. Du kan använda Azure DevOps, verktyg som Jenkins eller PowerShell, för att distribuera och dirigera tjänster till Azure Stack Hub.

**Överväganden**

En av de största skillnaderna när det gäller Azure Stack Hub distributioner är frågan om Internettillgänglighet. Internettillgänglighet avgör om du vill välja en Microsoft-värd eller en egen värdbaserade byggaragent för dina CI/CD-jobb.

En agent med egen värd kan köras ovanpå en Azure Stack Hub (som en virtuell IaaS-dator) eller i ett nätverksundernät som har åtkomst Azure Stack Hub. Gå till [Azure Pipelines-agenter om](/azure/devops/pipelines/agents/agents) du vill veta mer om skillnaderna.

Följande bild hjälper dig att avgöra om du behöver en egen värd eller en Microsoft-värdbaserade byggaragent:

![Build-agenter med egen värd Ja eller Nej](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Är Azure Stack Hub hanteringsslutpunkter tillgängliga via Internet?
  - Ja: Vi kan använda Azure Pipelines med Microsofts värdbaserade agenter för att ansluta till Azure Stack Hub.
  - Nej: Vi behöver egna agenter som kan ansluta till Azure Stack Hub hanteringsslutpunkter.
- Är vårt Kubernetes-kluster tillgängligt via Internet?
  - Ja: Vi kan använda Azure Pipelines med Microsofts värdbaserade agenter för att interagera direkt med Kubernetes API-slutpunkten.
  - Nej: Vi behöver agenter med egen värd som kan ansluta till kubernetes-klustrets API-slutpunkt.

I scenarier där Azure Stack Hub-hanteringsslutpunkter och Kubernetes-API är tillgängliga via Internet kan distributionen använda en Microsoft-värdbaserade agent. Den här distributionen resulterar i en programarkitektur på följande sätt:

[![Översikt över offentlig arkitektur](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Om Azure Resource Manager slutpunkter, Kubernetes API eller båda inte är direkt åtkomliga via Internet kan vi använda en egen byggaragent för att köra pipelinestegen. Den här designen behöver mindre anslutning och kan endast distribueras med lokal nätverksanslutning till Azure Resource Manager-slutpunkter och Kubernetes-API:et:

[![Översikt över den on-prem-arkitekturen](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Vad gäller för frånkopplade scenarier?** I scenarier där antingen Azure Stack Hub, Kubernetes eller båda inte har Internetuppriktade hanteringsslutpunkter, är det fortfarande möjligt att använda Azure DevOps för dina distributioner. Du kan antingen använda en lokal agentpool (som är en DevOps-agent som körs lokalt eller på Azure Stack Hub) eller en helt lokal Azure DevOps Server lokalt. Den egna agenten behöver bara utgående HTTPS-internetanslutning (TCP/443).

Mönstret kan använda ett Kubernetes-kluster (distribuerat och orkestrerat med AKS-motorn) på varje Azure Stack Hub instans. Den innehåller ett program som består av en frontend, en mellannivå, backend-tjänster (till exempel MongoDB) och en nginx-baserad ingress-kontrollant. I stället för att använda en databas som finns i K8s-klustret kan du använda "externa datalager". Databasalternativen omfattar MySQL, SQL Server eller någon typ av databas som finns utanför Azure Stack Hub eller i IaaS. Konfigurationer som den här är inte i omfånget här.

## <a name="partner-solutions"></a>Partnerlösningar

Det finns Microsoft-partnerlösningar som kan utöka funktionerna i Azure Stack Hub. Dessa lösningar har visat sig vara användbara i distributioner av program som körs på Kubernetes-kluster.  

## <a name="storage-and-data-solutions"></a>Storage och datalösningar

Enligt beskrivningen [i Data- och lagringsöverväganden](#data-and-storage-considerations)har Azure Stack Hub för närvarande inte en inbyggd lösning för att replikera lagring över flera instanser. Till skillnad från Azure finns inte möjligheten att replikera lagring över flera regioner. I Azure Stack Hub är varje instans ett eget distinkt moln. Lösningar är dock tillgängliga från Microsoft-partner som möjliggör lagringsreplikering mellan Azure Stack Hubs och Azure. 

**SKALBARHET**

[Scality](https://www.scality.com/) levererar lagring i webbskala som har drivs av digitala företag sedan 2009. Scality RING, vår programvarudefinierade lagring, omvandlar x86-servrar till en obegränsad lagringspool för alla typer av data – filer och objekt – i petabyteskala.

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) förenklar företagslagring med obegränsad och skalbar lagring som konsoliderar enorma datamängder till en enda miljö som är enkel att hantera.

## <a name="next-steps"></a>Nästa steg

Mer information om begrepp som introduceras i den här artikeln:

- [Skalning mellan moln och](pattern-cross-cloud-scale.md) [mönster för geo-distribuerade appar](pattern-geo-distributed.md) i Azure Stack Hub.
- [Arkitektur för mikrotjänster på Azure Kubernetes Service (AKS).](/azure/architecture/reference-architectures/microservices/aks)

När du är redo att testa lösningsexempel fortsätter du med [distributionsguiden för Kubernetes-kluster](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)med hög tillgänglighet. Distributionsguiden innehåller stegvisa instruktioner för att distribuera och testa dess komponenter.