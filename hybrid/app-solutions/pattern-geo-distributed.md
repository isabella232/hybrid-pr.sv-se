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
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="be959-103">Mönster för geo-distribuerat program</span><span class="sxs-lookup"><span data-stu-id="be959-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="be959-104">Lär dig att tillhandahålla app-slutpunkter i flera regioner och dirigera användar trafik baserat på plats-och efterlevnads behov.</span><span class="sxs-lookup"><span data-stu-id="be959-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="be959-105">Kontext och problem</span><span class="sxs-lookup"><span data-stu-id="be959-105">Context and problem</span></span>

<span data-ttu-id="be959-106">Organisationer med breda geografiska områden strävar efter säker och korrekt distribution och möjliggör åtkomst till data samtidigt som de säkerställer nödvändiga nivåer av säkerhet, efterlevnad och prestanda per användare, plats och enhet över gränserna.</span><span class="sxs-lookup"><span data-stu-id="be959-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="be959-107">Lösning</span><span class="sxs-lookup"><span data-stu-id="be959-107">Solution</span></span>

<span data-ttu-id="be959-108">Routnings mönstret för geografisk trafik i Azure Stack Hub, eller geo-distribuerade appar, gör att trafiken dirigeras till specifika slut punkter baserat på olika mått.</span><span class="sxs-lookup"><span data-stu-id="be959-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="be959-109">Genom att skapa en Traffic Manager med geografisk och slut punkts konfiguration dirigeras trafiken till slut punkter baserat på regionala krav, företags-och internationella regler och data behov.</span><span class="sxs-lookup"><span data-stu-id="be959-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Geo-distribuerat mönster](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="be959-111">Komponenter</span><span class="sxs-lookup"><span data-stu-id="be959-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="be959-112">Utanför molnet</span><span class="sxs-lookup"><span data-stu-id="be959-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="be959-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="be959-113">Traffic Manager</span></span>

<span data-ttu-id="be959-114">I diagrammet finns Traffic Manager utanför det offentliga molnet, men det måste kunna koordinera trafik i både det lokala data centret och det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="be959-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="be959-115">Saldobeloppet dirigerar trafik till geografiska platser.</span><span class="sxs-lookup"><span data-stu-id="be959-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="be959-116">DNS (Domain Name System)</span><span class="sxs-lookup"><span data-stu-id="be959-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="be959-117">Domain Name System eller DNS ansvarar för översättning (eller matchning) av en webbplats eller ett tjänst namn till dess IP-adress.</span><span class="sxs-lookup"><span data-stu-id="be959-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="be959-118">Offentligt moln</span><span class="sxs-lookup"><span data-stu-id="be959-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="be959-119">Moln slut punkt</span><span class="sxs-lookup"><span data-stu-id="be959-119">Cloud Endpoint</span></span>

<span data-ttu-id="be959-120">Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till resurs slut punkten för det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="be959-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="be959-121">Lokala moln</span><span class="sxs-lookup"><span data-stu-id="be959-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="be959-122">Lokal slut punkt</span><span class="sxs-lookup"><span data-stu-id="be959-122">Local endpoint</span></span>

<span data-ttu-id="be959-123">Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till resurs slut punkten för det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="be959-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="be959-124">Problem och överväganden</span><span class="sxs-lookup"><span data-stu-id="be959-124">Issues and considerations</span></span>

<span data-ttu-id="be959-125">Tänk på följande när du bestämmer hur du ska implementera mönstret:</span><span class="sxs-lookup"><span data-stu-id="be959-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="be959-126">Skalbarhet</span><span class="sxs-lookup"><span data-stu-id="be959-126">Scalability</span></span>

<span data-ttu-id="be959-127">Mönstret hanterar geografisk trafikroutning i stället för att skalas för att möta ökningen av trafiken.</span><span class="sxs-lookup"><span data-stu-id="be959-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="be959-128">Du kan dock kombinera det här mönstret med andra Azure-lösningar och lokala lösningar.</span><span class="sxs-lookup"><span data-stu-id="be959-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="be959-129">Det här mönstret kan till exempel användas med mönstret för skalning mellan moln.</span><span class="sxs-lookup"><span data-stu-id="be959-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="be959-130">Tillgänglighet</span><span class="sxs-lookup"><span data-stu-id="be959-130">Availability</span></span>

<span data-ttu-id="be959-131">Se till att lokalt distribuerade appar är konfigurerade för hög tillgänglighet via lokal maskin varu konfiguration och program varu distribution.</span><span class="sxs-lookup"><span data-stu-id="be959-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="be959-132">Hanterbarhet</span><span class="sxs-lookup"><span data-stu-id="be959-132">Manageability</span></span>

<span data-ttu-id="be959-133">Mönstret säkerställer sömlös hantering och välbekanta gränssnitt mellan miljöer.</span><span class="sxs-lookup"><span data-stu-id="be959-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="be959-134">När du ska använda det här mönstret</span><span class="sxs-lookup"><span data-stu-id="be959-134">When to use this pattern</span></span>

- <span data-ttu-id="be959-135">Min organisation har internationella grenar som kräver anpassade regionala säkerhets-och distributions principer.</span><span class="sxs-lookup"><span data-stu-id="be959-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="be959-136">Var och en av organisationens kontor hämtar personal-, affärs-och anläggnings data, vilket kräver rapporterings aktivitet per lokal lagstiftning och tidszon.</span><span class="sxs-lookup"><span data-stu-id="be959-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="be959-137">De storskaliga kraven kan uppfyllas genom horisontellt skala ut appar, med flera distributioner av appar som görs inom en region och mellan regioner för att hantera extrema belastnings krav.</span><span class="sxs-lookup"><span data-stu-id="be959-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="be959-138">Apparna måste ha hög tillgänglighet och svara på klient begär Anden även i drifts avbrott i en region.</span><span class="sxs-lookup"><span data-stu-id="be959-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="be959-139">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="be959-139">Next steps</span></span>

<span data-ttu-id="be959-140">Mer information om ämnen som introduceras i den här artikeln:</span><span class="sxs-lookup"><span data-stu-id="be959-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="be959-141">I [översikten över Azure-Traffic Manager](/azure/traffic-manager/traffic-manager-overview) kan du läsa mer om hur den här DNS-baserade trafikbelastnings utjämningen fungerar.</span><span class="sxs-lookup"><span data-stu-id="be959-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="be959-142">I [design överväganden för Hybrid appar](overview-app-design-considerations.md) kan du läsa mer om metod tips och få svar på eventuella ytterligare frågor.</span><span class="sxs-lookup"><span data-stu-id="be959-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="be959-143">Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.</span><span class="sxs-lookup"><span data-stu-id="be959-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="be959-144">När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för geo-distribuerad app-lösning](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="be959-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="be959-145">Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.</span><span class="sxs-lookup"><span data-stu-id="be959-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="be959-146">Du får lära dig hur du dirigerar trafik till vissa slut punkter baserat på olika mått med hjälp av mönstret geo-distribuerat program.</span><span class="sxs-lookup"><span data-stu-id="be959-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="be959-147">Genom att skapa en Traffic Manager profil med geografisk Routning och slut punkts konfiguration ser du till att information dirigeras till slut punkter baserat på regionala krav, företags-och internationella regler och dina data behov.</span><span class="sxs-lookup"><span data-stu-id="be959-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
