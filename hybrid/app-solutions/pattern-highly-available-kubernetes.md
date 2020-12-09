---
title: Kubernetes-mönster med hög tillgänglighet med Azure och Azure Stack hubb
description: Lär dig hur en Kubernetes-kluster lösning ger hög tillgänglighet med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 454cc0a0531882b7a8ec050a461420ce13eebcfe
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911795"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Kluster mönster för Kubernetes med hög tillgänglighet

Den här artikeln beskriver hur du skapar och använder en Kubernetes-baserad infrastruktur med hög tillgänglighet med hjälp av Azure Kubernetes service (AKS)-motorn på Azure Stack Hub. Det här scenariot är vanligt för organisationer med kritiska arbets belastningar i miljöer med hög begränsning och regler. Organisationer i domäner som ekonomi, försvar och myndigheter.

## <a name="context-and-problem"></a>Kontext och problem

Många organisationer utvecklar molnbaserade lösningar som utnyttjar de avancerade tjänsterna och teknikerna som Kubernetes. Även om Azure tillhandahåller data Center i de flesta regioner i världen, finns det ibland Edge-fall och scenarier där affärs kritiska program måste köras på en bestämd plats. Några överväganden är:

- Plats känslighet
- Fördröjning mellan program och lokala system
- Bevarande av bandbredd
- Anslutningar
- Regler eller lagstadgade krav

Azure, i kombination med Azure Stack hubb, löser de flesta av dessa problem. En rad olika alternativ, beslut och överväganden för en lyckad implementering av Kubernetes som körs på Azure Stack Hub beskrivs nedan.

## <a name="solution"></a>Lösning

Det här mönstret förutsätter att vi måste hantera en strikt uppsättning begränsningar. Programmet måste köras lokalt och alla personliga uppgifter får inte komma åt offentliga moln tjänster. Övervakning och andra icke-PII-data kan skickas till Azure och bearbetas där. Externa tjänster som en offentlig Container Registry eller andra kan nås, men kan filtreras via en brand vägg eller proxyserver.

Exempel programmet som visas här (baserat på [Azure Kubernetes service workshop](/learn/modules/aks-workshop/)) är utformat för att använda Kubernetes-inbyggda lösningar när det är möjligt. Den här designen gör att du slipper låsa leverantören, i stället för att använda plattforms intern tjänster. Som exempel använder programmet en databas server med egen värd i stället för en PaaS-tjänst eller en extern databas tjänst.

[![Program mönster hybrid](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Föregående diagram illustrerar program arkitekturen i exempel programmet som körs på Kubernetes på Azure Stack Hub. Appen består av flera komponenter, inklusive:

 1) Ett AKS Engine-baserat Kubernetes-kluster på Azure Stack Hub.
 2) [cert Manager](https://www.jetstack.io/cert-manager/), som innehåller en uppsättning verktyg för certifikat hantering i Kubernetes, som används för att automatiskt begära certifikat från att tillåta kryptering.
 3) Ett Kubernetes-namnområde som innehåller program komponenterna för klient delen (klassificering – webb), API (klassificerings-API) och databas (ratings-MongoDB).
 4) Den ingångs kontroll enhet som dirigerar HTTP/HTTPS-trafik till slut punkter inom Kubernetes-klustret.

Exempel programmet används för att illustrera program arkitekturen. Alla komponenter är exempel. Arkitekturen innehåller bara en enda program distribution. För att uppnå hög tillgänglighet (HA) kommer vi att köra distributionen minst två gånger på två olika Azure Stack Hubbs instanser – de kan köras antingen på samma plats eller på två (eller flera) olika platser:

![Infrastruktur arkitektur](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Tjänster som Azure Container Registry, Azure Monitor och andra platser ligger utanför Azure Stack hubben i Azure eller lokalt. Den här hybrid designen skyddar lösningen mot avbrott i en enskild Azure Stack Hub-instans.

## <a name="components"></a>Komponenter

Den övergripande arkitekturen består av följande komponenter:

**Azure Stack Hub** är en förlängning av Azure som kan köra arbets belastningar i en lokal miljö genom att tillhandahålla Azure-tjänster i ditt data Center. Gå till [Översikt över Azure Stack Hub](/azure-stack/operator/azure-stack-overview) om du vill veta mer.

**Azure Kubernetes service Engine (AKS Engine)** är motorn bakom det hanterade Kubernetes Service-erbjudandet, Azure Kubernetes service (AKS), som är tillgängligt i Azure idag. För Azure Stack Hub låter AKS-motorn distribuera, skala och uppgradera fullständigt aktuella, självhanterade Kubernetes-kluster med hjälp av Azure Stack Hub: s IaaS-funktioner. Gå till [AKS Engine-översikten](https://github.com/Azure/aks-engine) om du vill veta mer.

Gå till [kända problem och begränsningar](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) för att lära dig mer om skillnaderna mellan AKS-motorn på Azure och AKS-motorn på Azure Stack Hub.

**Azure Virtual Network (VNet)** används för att tillhandahålla nätverks infrastrukturen på varje Azure Stack hubb för de Virtual Machines (VM) som är värd för Kubernetes-kluster infrastrukturen.

**Azure Load Balancer** används för Kubernetes API-slutpunkten och Nginx ingångs kontroll. Belastningsutjämnaren dirigerar externa (till exempel Internet) trafik till noder och virtuella datorer som erbjuder en speciell tjänst.

**Azure Container Registry (ACR)** används för att lagra privata Docker-avbildningar och Helm-diagram, som distribueras till klustret. AKS-motorn kan autentiseras med Container Registry med hjälp av en Azure AD-identitet. Kubernetes kräver inte ACR. Du kan använda andra behållar register, till exempel Docker Hub.

**Azure databaser** är en uppsättning versions kontroll verktyg som du kan använda för att hantera din kod. Du kan också använda GitHub eller andra git-baserade databaser. Gå till [Översikt över Azure databaser](/azure/devops/repos/get-started/what-is-repos) om du vill veta mer.

**Azure-pipeliner** ingår i Azure DevOps Services och kör automatiserade byggen, tester och distributioner. Du kan också använda tredjeparts CI/CD-lösningar som Jenkins. Gå till [Översikt över Azure pipeline](/azure/devops/pipelines/get-started/what-is-azure-pipelines) för mer information.

**Azure Monitor** samlar in och lagrar mått och loggar, inklusive plattforms mått för Azure-tjänsterna i lösningen och programtelemetri. Använd dessa data för att övervaka programmet, konfigurera aviseringar och instrument paneler och utföra rotor Saks analyser av fel. Azure Monitor integreras med Kubernetes för att samla in mått från kontrollanter, noder och behållare, samt behållar loggar och Master Node-loggar. Gå till [Azure Monitor översikt](/azure/azure-monitor/overview) för mer information.

**Azure Traffic Manager** är en DNS-baserad trafikbelastnings utjämning som gör att du kan distribuera trafik optimalt till tjänster i olika Azure-regioner eller Azure Stack Hub-distributioner. Traffic Manager ger också hög tillgänglighet och svars tider. Program slut punkterna måste vara tillgängliga från utsidan. Det finns även andra lokala lösningar tillgängliga.

**Kubernetes ingress-styrenheten** exponerar http (S) vägar till tjänster i ett Kubernetes-kluster. Nginx eller lämplig ingångs kontroll kan användas för detta ändamål.

**Helm** är en paket hanterare för Kubernetes-distribution, vilket ger ett sätt att paketera olika Kubernetes-objekt, t. ex. distributioner, tjänster, hemligheter, i ett enda "diagram". Du kan publicera, distribuera, kontrol lera versions hantering och uppdatera ett diagram objekt. Azure Container Registry kan användas som lagrings plats för att lagra paketerade Helm-diagram.

## <a name="design-considerations"></a>Designöverväganden

Det här mönstret följer några viktiga överväganden som beskrivs i detalj i nästa avsnitt i den här artikeln:

- Programmet använder Kubernetes-inbyggda lösningar för att undvika att leverantören låser sig.
- Programmet använder en arkitektur för mikrotjänster.
- Azure Stack Hub behöver inte vara inkommande men tillåter utgående Internet anslutning.

Dessa rekommendationer gäller även för verkliga arbets belastningar och scenarier.

## <a name="scalability-considerations"></a>Skalbarhetsöverväganden

Skalbarhet är viktigt för att ge användarna konsekvent, tillförlitlig och väl genomförd åtkomst till programmet.

Exempel scenariot omfattar skalbarhet för flera skikt i program stacken. Här är en översikt över de olika nivåerna:

| Arkitektur nivå | Nätverk | Hur gör jag? |
| --- | --- | ---
| Program | Program | Horisontell skalning baserat på antalet poddar/repliker/Container Instances * |
| Kluster | Kubernetes-kluster | Antalet noder (mellan 1 och 50), VM-SKU-storlek och Node-pooler (AKS-motorn på Azure Stack Hub stöder för närvarande bara en pool med en nod). använda AKS-motorns skalnings kommando (manuell) |
| Infrastruktur | Azure Stack Hub | Antal noder, kapacitet och skalnings enheter inom en Azure Stack hubb distribution |

\* Använda Kubernetes "horisontell Pod autoskalning (HPA); automatiserad Metric-baserad skalning eller vertikal skalning genom att ändra behållar instanserna (CPU/minne).

**Azure Stack hubb (infrastruktur nivå)**

Azure Stack Hub-infrastrukturen är grunden för den här implementeringen eftersom Azure Stack Hub körs på fysisk maskin vara i ett Data Center. När du väljer nav maskin vara måste du välja mellan CPU, minnes täthet, lagrings konfiguration och antal servrar. Om du vill veta mer om skalbarhet för Azure Stack hubb kan du läsa följande resurser:

- [Översikt över kapacitets planering för Azure Stack hubb](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Lägg till ytterligare noder för skalnings enhet i Azure Stack hubb](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes-kluster (kluster nivå)**

Själva Kubernetes-klustret består av och bygger på Azure (stack) IaaS-komponenter, inklusive beräknings-, lagrings-och nätverks resurser. Kubernetes-lösningar omfattar Master-och Worker-noder, som distribueras som virtuella datorer i Azure (och Azure Stack Hub).

- [Control plan-noder](/azure/aks/concepts-clusters-workloads#control-plane) (Master) tillhandahåller kärn Kubernetes-tjänster och dirigering av program arbets belastningar.
- [Worker-noder](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (Worker) kör dina arbets belastningar för program.

När du väljer VM-storlekar för den första distributionen finns det flera saker att tänka på:  

- **Kostnad** – när du planerar dina arbetsnoder bör du tänka på den totala kostnaden per virtuell dator som du kommer att ådra dig. Om dina program arbets belastningar till exempel kräver begränsade resurser, bör du planera att distribuera mindre virtuella datorer. Azure Stack hubb, till exempel Azure, debiteras vanligt vis enligt en konsumtion, så det är därför viktigt att storleks änd ande virtuella datorer för Kubernetes-roller är viktiga för att optimera förbruknings kostnaderna. 

- **Skalbarhet** – skalbarhet för klustret uppnås genom att skala in och ut antalet master-och Worker-noder, eller genom att lägga till ytterligare resurspooler (inte tillgängligt på Azure Stack Hub idag). Skalning av klustret kan göras baserat på prestanda data som samlas in med hjälp av container Insights (Azure Monitor + Log Analytics). 

    Om ditt program behöver fler (eller färre) resurser kan du skala ut (eller i) dina aktuella noder vågrätt (mellan 1 och 50 noder). Om du behöver fler än 50 noder kan du skapa ett ytterligare kluster i en separat prenumeration. Du kan inte skala upp de faktiska virtuella datorerna lodrätt till en annan VM-storlek utan att omdistribuera klustret.

    Skalning görs manuellt med hjälp av den virtuella AKS-motorn som användes för att distribuera Kubernetes-klustret från början. Mer information finns i [skala Kubernetes-kluster](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Kvoter** – Överväg de [kvoter](/azure-stack/operator/azure-stack-quota-types) som du har konfigurerat när du planerar en AKS-distribution på Azure Stack Hub. Se till att varje [prenumeration](/azure-stack/operator/service-plan-offer-subscription-overview) har rätt planer och kvoterna konfigurerade. Prenumerationen måste anpassas efter mängden data bearbetning, lagring och andra tjänster som behövs för att klustren ska kunna skalas ut.

- **Program arbets belastningar** – se [begreppen kluster och arbets belastnings koncept](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) i Kubernetes Core-koncept för Azure Kubernetes service-dokument. I den här artikeln får du hjälp med att omfånget av rätt VM-storlek baserat på programmets beräknings-och minnes behov.  

**Program (program nivå)**

I program skiktet använder vi Kubernetes [horisontell Pod autoskalning (hPa)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). HPA kan öka eller minska antalet repliker (Pod/Container Instances) i vår distribution baserat på olika mått (till exempel processor användning).

Ett annat alternativ är att skala container instanser lodrätt. Detta kan åstadkommas genom att ändra mängden CPU och minne som begärs och är tillgängliga för en bestämd distribution. Mer information finns i [Hantera resurser för behållare](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) på Kubernetes.io.

## <a name="networking-and-connectivity-considerations"></a>Nätverks-och anslutnings överväganden

Nätverk och anslutning påverkar även de tre skikt som tidigare nämnts för Kubernetes på Azure Stack Hub. I följande tabell visas de lager och vilka tjänster de innehåller:

| Skikt | Nätverk | Vad? |
| --- | --- | ---
| Program | Program | Hur är programmet tillgängligt? Kommer den att exponeras för Internet? |
| Kluster | Kubernetes-kluster | Kubernetes-API, AKS Engine VM, Hämta behållar avbildningar (utgående), skicka övervaknings data och telemetri (utgående) |
| Infrastruktur | Azure Stack Hub | Tillgänglighet för Azure Stack Hub Management-slutpunkter som portalen och Azure Resource Manager slut punkter. |

**Program**

För program lagret är den viktigaste överväganden om programmet exponeras och är tillgängligt från Internet. Från ett Kubernetest perspektiv innebär Internet åtkomst att exponera en distributions-eller Pod med hjälp av en Kubernetes-tjänst eller en ingångs kontroll.

> [!NOTE]
> Vi rekommenderar att du använder ingångs kort för att exponera Kubernetes-tjänster som antalet offentliga IP-adresser på Azure Stack Hub är begränsat till 5. Den här designen begränsar också antalet Kubernetes-tjänster (med typen LoadBalancer) till 5, vilket är för litet för många distributioner. Mer information finns i [dokumentationen till AKS-motorn](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) .

Att exponera ett program med hjälp av en offentlig IP-adress via en Load Balancer eller en ingångs kontroll är inte nessecarily att programmet nu är tillgängligt via Internet. Det är möjligt att Azure Stack hubb har en offentlig IP-adress som bara är synlig i det lokala intranätet, men inte alla offentliga IP-adresser är Internet-riktade.

Föregående block avser inkommande trafik till programmet. Ett annat avsnitt som måste beaktas för en lyckad Kubernetes-distribution är utgående/utgående trafik. Här är några användnings fall som kräver utgående trafik:

- Hämta behållar avbildningar som lagras på DockerHub eller Azure Container Registry
- Hämtar Helm-diagram
- Sändning av Application Insights data (eller andra övervaknings data)

Vissa företags miljöer kan kräva användning av _transparenta_ eller _icke-transparenta_ proxyservrar. Dessa servrar kräver en speciell konfiguration på olika komponenter i vårt kluster. Dokumentation om AKS-motorn innehåller olika uppgifter om hur du hanterar nätverks-proxyservrar. Mer information finns i [AKS motor and proxy servers](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Till sist måste trafiken mellan kluster ligga mellan Azure Stack Hub-instanser. Exempel distributionen består av enskilda Kubernetes-kluster som körs på enskilda Azure Stack Hubbs instanser. Trafik mellan dem, till exempel replikeringstrafik mellan två databaser, är "extern trafik". Extern trafik måste dirigeras via antingen en plats-till-plats-VPN-eller Azure Stack hubb offentliga IP-adresser för att ansluta Kubernetes på två Azure Stack Hub-instanser:

![trafik mellan och inom kluster](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Kluster**  

Kubernetes-klustret behöver inte nödvändigt vis vara tillgängligt via Internet. Den relevanta delen är det Kubernetes-API som används för att köra ett kluster, till exempel med hjälp av `kubectl` . Kubernetes API-slutpunkten måste vara tillgänglig för alla som använder klustret eller distribuerar program och tjänster ovanpå dem. Det här avsnittet beskrivs mer detaljerat från en DevOps i avsnittet [distribution (CI/CD)](#deployment-cicd-considerations) nedan.

På kluster nivå finns det också några överväganden kring utgående trafik:

- Uppdatering av nod (för Ubuntu)
- Övervaknings data (skickas till Azure-LogAnalytics)
- Andra agenter som kräver utgående trafik (gäller för varje distributions miljö)

Innan du distribuerar ditt Kubernetes-kluster med AKS-motorn bör du planera för den slutliga nätverks utformningen. I stället för att skapa en dedikerad Virtual Network kan det vara mer effektivt att distribuera ett kluster till ett befintligt nätverk. Du kan till exempel utnyttja en befintlig VPN-anslutning för plats-till-plats som redan har kon figurer ATS i din Azure Stack Hub-miljö.

**Infrastruktur**  

Infrastruktur syftar på åtkomst till Azure Stack Hub Management-slutpunkter. Slut punkter innehåller klient-och administrations portalerna och Azure Resource Manager-administratör och klient slut punkter. De här slut punkterna krävs för att kunna använda Azure Stack Hub och dess kärn tjänster.

## <a name="data-and-storage-considerations"></a>Överväganden för data och lagring

Två instanser av programmet kommer att distribueras på två enskilda Kubernetes-kluster i två Azure Stack Hubbs instanser. Den här designen kräver att vi funderar på hur du ska replikera och synkronisera data mellan dem.

Med Azure har vi den inbyggda funktionen för att replikera lagring över flera regioner och zoner i molnet. För närvarande med Azure Stack Hub finns det inga inbyggda sätt att replikera lagringen över två olika Azure Stack Hub-instanser – de utgör två oberoende moln utan att det finns något överlappande sätt att hantera dem som en uppsättning. Genom att planera för återhämtning av program som körs i Azure Stack Hub kan du överväga detta oberoende i dina program design och distributioner.

I de flesta fall behövs inte Storage-replikering för ett elastiskt och hög tillgängligt program som distribueras på AKS. Men du bör överväga oberoende lagring per Azure Stack Hub-instans i din program design. Om den här designen är ett problem eller ett väg block för att distribuera lösningen på Azure Stack hubb, finns det möjliga lösningar från Microsoft-partner som tillhandahåller bilagor för lagring. Storage-bilagor tillhandahåller en lösning för lagrings replikering över flera Azure Stack hubbar och Azure. Mer information finns i [partner lösningar](#partner-solutions).

I vår arkitektur ansågs dessa lager vara:

**Konfiguration**

Konfigurationen omfattar konfigurationen av Azure Stack hubb, AKS-motor och Kubernetes-klustret. Konfigurationen bör automatiseras så mycket som möjligt och lagras som infrastruktur som kod i ett git-baserat versions kontroll system som Azure DevOps eller GitHub. Dessa inställningar kan inte enkelt synkroniseras över flera distributioner. Vi rekommenderar därför att du lagrar och tillämpar konfiguration från utsidan och använder DevOps pipeline.

**Program**

Programmet ska lagras i en git-baserad lagrings plats. När det finns en ny distribution, ändringar i programmet eller haveri beredskap, kan det enkelt distribueras med hjälp av Azure-pipeliner.

**Data**

Data är den viktigaste övervägandeheten i de flesta program design. Program data måste vara synkroniserade mellan olika instanser av programmet. Data behöver också en strategi för säkerhets kopiering och haveri beredskap om det uppstår ett avbrott.

Att uppnå den här designen beror på teknik val. Här följer några exempel på lösningar för att implementera en databas med hög tillgänglighet på Azure Stack Hub:

- [Distribuera en SQL Server 2016-tillgänglighets grupp till Azure och Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Distribuera en MongoDB-lösning med hög tillgänglighet till Azure och Azure Stack hubb](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Att tänka på när du arbetar med data på flera platser är en ännu mer komplicerad lösning för en hög tillgänglig och flexibel lösning. Tänk på att:

- Svars tid och nätverks anslutning mellan Azure Stack hubbar.
- Tillgänglighet för identiteter för tjänster och behörigheter. Varje Azure Stack Hub-instans integreras med en extern katalog. Under distributionen väljer du antingen Azure Active Directory (Azure AD) eller Active Directory Federation Services (AD FS) (ADFS). Därför är det möjligt att använda en enda identitet som kan samverka med flera oberoende Azure Stack Hubbs instanser.

## <a name="business-continuity-and-disaster-recovery"></a>Affärskontinuitet och haveriberedskap

Verksamhets kontinuitet och haveri beredskap (BCDR) är ett viktigt avsnitt i både Azure Stack hubb och Azure. Den största skillnaden är att operatören måste hantera hela BCDR-processen i Azure Stack Hub. I Azure hanteras delar av BCDR automatiskt av Microsoft.

BCDR påverkar samma områden som anges i föregående avsnitts [data och överväganden för lagring](#data-and-storage-considerations):

- Infrastruktur/konfiguration
- Tillgänglighet för program
- Programdata

Som vi nämnt i föregående avsnitt är dessa områden ansvaret för Azure Stack Hub-operatören och kan variera mellan olika organisationer. Planera BCDR enligt dina tillgängliga verktyg och processer.

**Infrastruktur och konfiguration**

I det här avsnittet beskrivs den fysiska och logiska infrastrukturen och konfigurationen av Azure Stack Hub. Den täcker åtgärder i administratören och klientens utrymmen.

Azure Stack Hubbs operatorn (eller administratören) ansvarar för underhåll av Azure Stack Hub-instanserna. Inklusive komponenter som nätverk, lagring, identitet och andra ämnen som omfattas av den här artikeln. Mer information om de olika Azure Stack Hub-åtgärderna finns i följande resurser:

- [Återställa data i Azure Stack hubb med tjänsten Infrastructure Backup](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Aktivera säkerhets kopiering för Azure Stack hubb från administratörs portalen](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Återställa från oåterkallelig dataförlust](/azure-stack/operator/azure-stack-backup-recover-data)
- [Metodtips för Infrastructure Backup-tjänsten](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub är den plattform och infrastruktur som Kubernetes-program kommer att distribueras till. Program ägaren för Kubernetes-programmet är en användare av Azure Stack Hub, med åtkomst som beviljats för att distribuera den program infrastruktur som krävs för lösningen. Program infrastruktur i det här fallet innebär Kubernetes-klustret, distribuerat med AKS-motorn och de omgivande tjänsterna. Dessa komponenter kommer att distribueras till Azure Stack Hub, vilket är begränsat av ett Azure Stack Hub-erbjudande. Se till att erbjudandet som godkänns av Kubernetes program ägare har tillräckligt med kapacitet (uttryckt i Azure Stack hubb kvoter) för att distribuera hela lösningen. Som rekommenderas i föregående avsnitt bör program distributionen automatiseras med hjälp av infrastruktur som kod och distributions pipeliner som Azure DevOps Azure-pipelines.

Mer information om Azure Stack Hubbs erbjudanden och kvoter finns i [Översikt över Azure Stack hubb, abonnemang, erbjudanden och prenumerationer](/azure-stack/operator/service-plan-offer-subscription-overview)

Det är viktigt att spara och lagra AKS Engine-konfigurationen på ett säkert sätt, inklusive dess utdata. De här filerna innehåller konfidentiell information som används för att komma åt Kubernetes-klustret, så den måste skyddas från att exponeras för icke-administratörer.

**Tillgänglighet för program**

Programmet bör inte förlita sig på säkerhets kopieringar av en distribuerad instans. Som standard praxis kan du distribuera om programmet fullständigt efter infrastruktur-som-kod mönster. Distribuera till exempel med Azure DevOps Azure-pipelines. BCDR-proceduren bör omfatta omdistribution av programmet till samma eller ett annat Kubernetes-kluster.

**Program data**

Program data är den viktigaste delen för att minimera data förlust. I föregående avsnitt beskrevs tekniker för att replikera och synkronisera data mellan två (eller flera) instanser av programmet. Beroende på databas infrastrukturen (MySQL, MongoDB, MSSQL eller andra) som används för att lagra data, finns det olika databas tillgänglighets-och säkerhets kopierings tekniker som du kan välja mellan.

Rekommenderade metoder för att uppnå integritet är att använda antingen:
- En inbyggd lösning för säkerhets kopiering för den specifika databasen.
- En säkerhets kopierings lösning som officiellt stöder säkerhets kopiering och återställning av den databas typ som används av ditt program.

> [!IMPORTANT]
> Lagra inte dina säkerhets kopierings data på samma Azure Stack Hub-instans där dina program data finns. Ett fullständigt avbrott i Azure Stack Hub-instansen skulle också äventyra dina säkerhets kopieringar.

## <a name="availability-considerations"></a>Överväganden för tillgänglighet

Kubernetes på Azure Stack Hub som distribueras via AKS-motorn är inte en hanterad tjänst. Det är en automatiserad distribution och konfiguration av ett Kubernetes-kluster med Azure Infrastructure-as-a-Service (IaaS). Detta ger samma tillgänglighet som den underliggande infrastrukturen.

Azure Stack Hub-infrastrukturen är redan flexibelt för fel och ger funktioner som tillgänglighets uppsättningar för att distribuera komponenter över flera [fel och uppdaterings domäner](/azure-stack/user/azure-stack-vm-considerations#high-availability). Men den underliggande tekniken (redundanskluster) medför fortfarande vissa stillestånds tider för virtuella datorer på en påverkad fysisk server, om ett maskin varu fel uppstår.

Det är en bra idé att distribuera ditt produktions Kubernetes-kluster samt arbets belastningen till två (eller flera) kluster. Dessa kluster bör finnas på olika platser eller data Center, och använder tekniker som Azure Traffic Manager för att dirigera användare baserat på kluster svars tid eller baserat på geografi.

![Använda Traffic Manager för att kontrol lera trafik flöden](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Kunder som har ett enda Kubernetes-kluster ansluter vanligt vis till tjänstens IP-adress eller DNS-namn för ett visst program. I en distribution med flera kluster bör kunderna ansluta till ett Traffic Manager DNS-namn som pekar på tjänsterna/ingressen på varje Kubernetes-kluster.

![Använda Traffic Manager för att dirigera till ett lokalt kluster](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Det här mönstret är också [bästa praxis för (hanterade) AKS-kluster i Azure](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

Kubernetes-klustret, som distribueras via AKS-motorn, bör bestå av minst tre huvudnoder och två arbetsnoder.

## <a name="identity-and-security-considerations"></a>Identitets-och säkerhets aspekter

Identitet och säkerhet är viktiga ämnen. Särskilt när lösningen sträcker sig oberoende Azure Stack Hubbs instanser. Kubernetes och Azure (inklusive Azure Stack Hub) har båda särskilda mekanismer för rollbaserad åtkomst kontroll (RBAC):

- Azure RBAC styr åtkomsten till resurser i Azure (och Azure Stack Hub), inklusive möjligheten att skapa nya Azure-resurser. Behörigheter kan tilldelas till användare, grupper eller tjänstens huvud namn. (Ett huvud namn för tjänsten är en säkerhets identitet som används av program.)
- Kubernetes RBAC styr behörigheter till Kubernetes-API: et. Att till exempel skapa poddar och Visa poddar är åtgärder som kan godkännas (eller nekas) till en användare via RBAC. För att tilldela Kubernetes behörigheter till användare skapar du roller och roll bindningar.

**Azure Stack hubb identitet och RBAC**

Azure Stack Hub erbjuder två alternativ för identitets leverantörer. Vilken provider du använder beror på miljön och om den körs i en ansluten eller frånkopplad miljö:

- Azure AD – kan bara användas i en ansluten miljö.
- ADFS till en traditionell Active Directory skog – kan användas både i en ansluten eller frånkopplad miljö.

Identitets leverantören hanterar användare och grupper, inklusive autentisering och auktorisering för åtkomst till resurser. Åtkomst kan beviljas till Azure Stack hubb resurser som prenumerationer, resurs grupper och enskilda resurser, t. ex. virtuella datorer eller belastningsutjämnare. Om du vill ha en konsekvent åtkomst modell bör du överväga att använda samma grupper (antingen direkt eller kapslat) för alla Azure Stack hubbar. Här är ett konfigurations exempel:

![kapslade AAD-grupper med Azure Stack Hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

Exemplet innehåller en dedikerad grupp (med AAD eller ADFS) för ett specifikt syfte. Om du till exempel vill ge deltagar behörighet för resurs gruppen som innehåller vår Kubernetes-kluster infrastruktur på en speciell Azure Stack Hub-instans (här "Seattle K8s Cluster Contributor"). Dessa grupper kapslas sedan in i en övergripande grupp som innehåller "under grupper" för varje Azure Stack hubb.

Vår exempel användare kommer nu att ha "deltagar"-behörighet för båda resurs grupperna som innehåller hela uppsättningen infrastruktur resurser för Kubernetes. Användaren kommer att ha åtkomst till resurser på båda Azure Stack Hubbs instanser, eftersom instanserna delar samma identitets leverantör.

> [!IMPORTANT]
> Dessa behörigheter påverkar endast Azure Stack hubb och några av de resurser som distribueras ovanpå den. En användare som har den här åtkomst nivån kan göra mycket skadlig, men kan inte komma åt Kubernetes IaaS-VM: er eller Kubernetes-API utan ytterligare åtkomst till Kubernetes-distributionen.

**Kubernetes-identitet och RBAC**

Ett Kubernetes-kluster använder som standard inte samma identitetsprovider som Azure Stack hubben. De virtuella datorerna som är värdar för Kubernetes-klustret, Master-och Worker-noderna använder SSH-nyckeln som anges under distributionen av klustret. Den här SSH-nyckeln krävs för att ansluta till de här noderna med SSH.

Kubernetes-API (som till exempel nås med hjälp av `kubectl` ) skyddas också av tjänst konton, inklusive ett tjänst konto för standard kluster administratör. Autentiseringsuppgifterna för det här tjänst kontot lagras initialt i `.kube/config` filen på dina Kubernetes huvud noder.

**Hemligheter för hantering och programautentiseringsuppgifter**

För att lagra hemligheter som anslutnings strängar eller autentiseringsuppgifter finns det flera alternativ, inklusive:

- Azure Key Vault
- Kubernetes-hemligheter
- lösningar från tredje part som HashiCorp Vault (som körs på Kubernetes)

Lagra inte hemligheter eller autentiseringsuppgifter i klartext i konfigurationsfiler, program koden eller i skript. Och lagra dem inte i ett versions kontroll system. I stället bör distributions automatiseringen Hämta hemligheterna vid behov.

## <a name="patch-and-update"></a>Uppdatering och uppdatering

Processen för **korrigering och uppdatering (PNU)** i Azure Kubernetes-tjänsten är delvis automatiserad. Kubernetes versions uppgraderingar utlöses manuellt, medan säkerhets uppdateringar tillämpas automatiskt. Dessa uppdateringar kan omfatta säkerhets korrigeringar för operativ system eller kernel-uppdateringar. AKS startar inte om dessa Linux-noder automatiskt för att slutföra uppdaterings processen. 

PNU-processen för ett Kubernetes-kluster som distribueras med AKS-motorn på Azure Stack Hub är ohanterad och ansvarar för kluster operatorn. 

AKS-motorn hjälper till med de två viktigaste uppgifterna:

- [Uppgradera till en nyare Kubernetes och bas operativ system avbildnings version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Uppgradera endast bas operativ system avbildningen](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Nyare Base OS-avbildningar innehåller de senaste säkerhets korrigeringarna för operativ systemet och kernel-uppdateringar. 

Den [obevakade uppgraderings](https://wiki.debian.org/UnattendedUpgrades) mekanismen installerar automatiskt säkerhets uppdateringar som släpps innan en ny bas operativ system avbildnings version är tillgänglig på Azure Stack hubb Marketplace. Obevakad uppgradering är aktiverat som standard och installerar säkerhets uppdateringar automatiskt, men startar inte om Kubernetes-klusternoderna. Att starta om noderna kan automatiseras med hjälp av Aemon med öppen källkod [ **K** Ubernetes reboot **D**(kured)) **RE**](/azure/aks/node-updates-kured). Kured söker efter Linux-noder som kräver en omstart och hanterar sedan automatiskt omschemaläggningen av pågående poddar och omstart av en nod.

## <a name="deployment-cicd-considerations"></a>Överväganden för distribution (CI/CD)

Azure och Azure Stack Hub exponerar samma Azure Resource Manager REST-API: er. Dessa API: er behandlas som andra Azure-moln (Azure, Azure Kina 21Vianet, Azure Government). Det kan finnas skillnader i API-versioner mellan moln och Azure Stack Hub tillhandahåller bara en delmängd av tjänsterna. URI: n för hanterings slut punkter är också olika för varje moln och varje instans av Azure Stack hubben.

Förutom de diskreta skillnaderna som anges kan Azure Resource Manager REST-API: er tillhandahålla ett konsekvent sätt att interagera med både Azure och Azure Stack Hub. Samma uppsättning verktyg kan användas här som används med andra Azure-moln. Du kan använda Azure DevOps, verktyg som Jenkins eller PowerShell för att distribuera och dirigera tjänster till Azure Stack Hub.

**Överväganden**

En av de största skillnaderna när det kommer till Azure Stack hubb distributioner är frågan om Internet-tillgänglighet. Internet-hjälpmedel avgör om du vill välja en Microsoft-värdbaserad eller en egen värd versions agent för dina CI/CD-jobb.

En lokal agent kan köras ovanpå Azure Stack Hub (som en IaaS VM) eller i ett undernät för nätverk som har åtkomst till Azure Stack Hub. Gå till [Azure pipelines-agenter](/azure/devops/pipelines/agents/agents) om du vill veta mer om skillnaderna.

Följande bild hjälper dig att bestämma om du behöver en egen värd eller en Microsoft-värdbaserad build-agent:

![Egna värdbaserade Bygg agenter Ja eller Nej](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Är slut punkterna för hantering av Azure Stack Hub tillgängliga via Internet?
  - Ja: vi kan använda Azure-pipelines med Microsofts värdbaserade agenter för att ansluta till Azure Stack Hub.
  - Nej: vi behöver egen värdbaserade agenter som kan ansluta till Azure Stack hubbens hanterings slut punkter.
- Är vårt Kubernetes-kluster tillgängligt via Internet?
  - Ja: vi kan använda Azure-pipelines med Microsofts värdbaserade agenter för att interagera direkt med Kubernetes API-slutpunkten.
  - Nej: vi behöver egna värdbaserade agenter som kan ansluta till Kubernetes-klustrets API-slutpunkt.

I scenarier där slut punkterna för Azure Stack Hub-hantering och Kubernetes-API är tillgängliga via Internet kan distributionen använda en Microsoft-värdbaserad agent. Den här distributionen leder till en program arkitektur på följande sätt:

[![Översikt över offentlig arkitektur](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Om Azure Resource Manager-slutpunkter, Kubernetes-API: n eller båda inte är direkt tillgängliga via Internet, kan vi utnyttja en egen värd versions agent för att köra pipeline-stegen. Den här designen kräver mindre anslutnings barhet och kan distribueras med endast lokal nätverks anslutning till Azure Resource Manager slut punkter och Kubernetes-API: et:

[![Översikt över lokal-arkitekturen](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Vad gäller frånkopplade scenarier?** I scenarier där antingen Azure Stack hubb eller Kubernetes eller båda av dem inte har Internet-riktade hanterings slut punkter, är det fortfarande möjligt att använda Azure DevOps för dina distributioner. Du kan antingen använda en lokal programmediepool (som är en DevOps-agent som körs lokalt eller på själva Azure Stack hubben) eller en helt egen värd Azure DevOps Server lokalt. Den egen värdbaserade agenten behöver bara utgående HTTPS (TCP/443) Internet anslutning.

Mönstret kan använda ett Kubernetes-kluster (distribuerat och dirigerat med AKS-motorn) på varje Azure Stack Hub-instans. Den innehåller ett program som består av en klient del, en mellan nivå, backend-tjänster (till exempel MongoDB) och en nginx-baserad ingångs kontroll. I stället för att använda en databas som finns i K8s-klustret kan du utnyttja "externa data lager". Databas alternativ inkluderar MySQL, SQL Server eller någon typ av databas som finns utanför Azure Stack Hub eller i IaaS. Konfigurationer som detta omfattas inte här.

## <a name="partner-solutions"></a>Partnerlösningar

Det finns Microsofts partner lösningar som kan utöka funktionerna i Azure Stack Hub. Dessa lösningar har hittats användbara vid distribution av program som körs på Kubernetes-kluster.  

## <a name="storage-and-data-solutions"></a>Lagrings-och data lösningar

Som det beskrivs i [data-och lagrings överväganden](#data-and-storage-considerations)har Azure Stack Hub för närvarande ingen inbyggd lösning för att replikera lagring över flera instanser. Till skillnad från Azure finns inte möjligheten att replikera lagring över flera regioner. I Azure Stack Hub, är varje instans ett eget distinkt moln. Lösningar är dock tillgängliga från Microsoft-partner som aktiverar lagrings replikering i Azure Stack hubbar och Azure. 

**SCALITY**

[Scality](https://www.scality.com/) levererar lagring av webb skala som har Powered digital verksamhet sedan 2009. Scality-ringen, vår programdefinierad lagring, vänder sig till x86-servrar i en obegränsad lagringspool för alla typer av data – fil och objekt – vid petabyte skalning.

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) fören klar företags lagringen med obegränsad skalbar lagring som konsoliderar enorma data uppsättningar till en enda, lätt hanterad miljö.

## <a name="next-steps"></a>Nästa steg

Mer information om begrepp som introduceras i den här artikeln:

- Mönster för [skalning mellan moln](pattern-cross-cloud-scale.md) och [geo-distribuerade appar](pattern-geo-distributed.md) i Azure Stack Hub.
- [Mikrotjänsters arkitektur på Azure Kubernetes service (AKS)](/azure/architecture/reference-architectures/microservices/aks).

När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för Kubernetes-kluster med hög tillgänglighet](solution-deployment-guide-highly-available-kubernetes.md). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.