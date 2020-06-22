---
title: Mönster för skalning mellan moln i Azure Stack hubb
description: Lär dig hur du skapar en skalbar Cross-Cloud-App på Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911964"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="d9599-103">Mönster för skalning mellan moln</span><span class="sxs-lookup"><span data-stu-id="d9599-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="d9599-104">Lägg automatiskt till resurser i en befintlig app för att få en ökad belastning.</span><span class="sxs-lookup"><span data-stu-id="d9599-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="d9599-105">Kontext och problem</span><span class="sxs-lookup"><span data-stu-id="d9599-105">Context and problem</span></span>

<span data-ttu-id="d9599-106">Din app kan inte öka kapaciteten för att möta oväntade ökningar i efter frågan.</span><span class="sxs-lookup"><span data-stu-id="d9599-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="d9599-107">Detta saknade skalbarhets resultat i användare som inte når appen under de högsta användnings tiderna.</span><span class="sxs-lookup"><span data-stu-id="d9599-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="d9599-108">Appen kan betjäna ett fast antal användare.</span><span class="sxs-lookup"><span data-stu-id="d9599-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="d9599-109">Globala företag kräver säkra, pålitliga och tillgängliga molnbaserade appar.</span><span class="sxs-lookup"><span data-stu-id="d9599-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="d9599-110">Mötet ökar efter frågan och använder rätt infrastruktur för att stödja att behovet är kritiskt.</span><span class="sxs-lookup"><span data-stu-id="d9599-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="d9599-111">Affärs problem för att balansera kostnader och underhåll med affärs data säkerhet, lagring och tillgänglighet i real tid.</span><span class="sxs-lookup"><span data-stu-id="d9599-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="d9599-112">Du kanske inte kan köra din app i det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="d9599-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="d9599-113">Det kanske inte är ekonomiskt genomförbart för verksamheten att bibehålla kapaciteten som krävs i den lokala miljön för att hantera toppar i efter frågan på appen.</span><span class="sxs-lookup"><span data-stu-id="d9599-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="d9599-114">Med det här mönstret kan du använda det offentliga molnets elastiskhet med din lokala lösning.</span><span class="sxs-lookup"><span data-stu-id="d9599-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="d9599-115">Lösning</span><span class="sxs-lookup"><span data-stu-id="d9599-115">Solution</span></span>

<span data-ttu-id="d9599-116">Mönstret för skalning mellan moln utökar en app som finns i ett lokalt moln med offentliga moln resurser.</span><span class="sxs-lookup"><span data-stu-id="d9599-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="d9599-117">Mönstret utlöses av en ökning eller minskning i efter frågan, och lägger till eller tar bort resurser i molnet.</span><span class="sxs-lookup"><span data-stu-id="d9599-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="d9599-118">Dessa resurser ger redundans, snabb tillgänglighet och geo-kompatibel routning.</span><span class="sxs-lookup"><span data-stu-id="d9599-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Mönster för skalning mellan moln](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="d9599-120">Det här mönstret gäller endast för tillstånds lösa komponenter i din app.</span><span class="sxs-lookup"><span data-stu-id="d9599-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="d9599-121">Komponenter</span><span class="sxs-lookup"><span data-stu-id="d9599-121">Components</span></span>

<span data-ttu-id="d9599-122">Mönstret för skalning mellan moln består av följande komponenter.</span><span class="sxs-lookup"><span data-stu-id="d9599-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="d9599-123">Utanför molnet</span><span class="sxs-lookup"><span data-stu-id="d9599-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="d9599-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="d9599-124">Traffic Manager</span></span>

<span data-ttu-id="d9599-125">I diagrammet finns detta utanför den offentliga moln gruppen, men det skulle behöva kunna koordinera trafik i både det lokala data centret och det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="d9599-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="d9599-126">Balancer ger hög tillgänglighet för appen genom att övervaka slut punkter och tillhandahålla omdistribution vid fel vid behov.</span><span class="sxs-lookup"><span data-stu-id="d9599-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="d9599-127">DNS (Domain Name System)</span><span class="sxs-lookup"><span data-stu-id="d9599-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="d9599-128">Domain Name System eller DNS ansvarar för översättning (eller matchning) av en webbplats eller ett tjänst namn till dess IP-adress.</span><span class="sxs-lookup"><span data-stu-id="d9599-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="d9599-129">Molnet</span><span class="sxs-lookup"><span data-stu-id="d9599-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="d9599-130">Värdbaserad Bygg Server</span><span class="sxs-lookup"><span data-stu-id="d9599-130">Hosted build server</span></span>

<span data-ttu-id="d9599-131">En miljö som är värd för din build-pipeline.</span><span class="sxs-lookup"><span data-stu-id="d9599-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="d9599-132">App-resurser</span><span class="sxs-lookup"><span data-stu-id="d9599-132">App resources</span></span>

<span data-ttu-id="d9599-133">App-resurserna måste kunna skalas in och ut, som skalnings uppsättningar och behållare för virtuella datorer.</span><span class="sxs-lookup"><span data-stu-id="d9599-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="d9599-134">Anpassat domän namn</span><span class="sxs-lookup"><span data-stu-id="d9599-134">Custom domain name</span></span>

<span data-ttu-id="d9599-135">Använd ett anpassat domän namn för BLOB.</span><span class="sxs-lookup"><span data-stu-id="d9599-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="d9599-136">Offentliga IP-adresser</span><span class="sxs-lookup"><span data-stu-id="d9599-136">Public IP addresses</span></span>

<span data-ttu-id="d9599-137">Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till resurs slut punkten för det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="d9599-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="d9599-138">Lokalt moln</span><span class="sxs-lookup"><span data-stu-id="d9599-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="d9599-139">Värdbaserad Bygg Server</span><span class="sxs-lookup"><span data-stu-id="d9599-139">Hosted build server</span></span>

<span data-ttu-id="d9599-140">En miljö som är värd för din build-pipeline.</span><span class="sxs-lookup"><span data-stu-id="d9599-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="d9599-141">App-resurser</span><span class="sxs-lookup"><span data-stu-id="d9599-141">App resources</span></span>

<span data-ttu-id="d9599-142">App-resurserna behöver möjlighet att skala in och ut, som skalnings uppsättningar och behållare för virtuella datorer.</span><span class="sxs-lookup"><span data-stu-id="d9599-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="d9599-143">Anpassat domän namn</span><span class="sxs-lookup"><span data-stu-id="d9599-143">Custom domain name</span></span>

<span data-ttu-id="d9599-144">Använd ett anpassat domän namn för BLOB.</span><span class="sxs-lookup"><span data-stu-id="d9599-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="d9599-145">Offentliga IP-adresser</span><span class="sxs-lookup"><span data-stu-id="d9599-145">Public IP addresses</span></span>

<span data-ttu-id="d9599-146">Offentliga IP-adresser används för att dirigera inkommande trafik via Traffic Manager till resurs slut punkten för det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="d9599-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="d9599-147">Problem och överväganden</span><span class="sxs-lookup"><span data-stu-id="d9599-147">Issues and considerations</span></span>

<span data-ttu-id="d9599-148">Tänk på följande när du bestämmer hur du ska implementera mönstret:</span><span class="sxs-lookup"><span data-stu-id="d9599-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="d9599-149">Skalbarhet</span><span class="sxs-lookup"><span data-stu-id="d9599-149">Scalability</span></span>

<span data-ttu-id="d9599-150">Huvud komponenten för skalning över moln är möjligheten att leverera skalning på begäran.</span><span class="sxs-lookup"><span data-stu-id="d9599-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="d9599-151">Skalning måste ske mellan offentliga och lokala moln infrastrukturer och tillhandahålla en enhetlig, tillförlitlig tjänst per behov.</span><span class="sxs-lookup"><span data-stu-id="d9599-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="d9599-152">Tillgänglighet</span><span class="sxs-lookup"><span data-stu-id="d9599-152">Availability</span></span>

<span data-ttu-id="d9599-153">Se till att lokalt distribuerade appar är konfigurerade för hög tillgänglighet via lokal maskin varu konfiguration och program varu distribution.</span><span class="sxs-lookup"><span data-stu-id="d9599-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="d9599-154">Hanterbarhet</span><span class="sxs-lookup"><span data-stu-id="d9599-154">Manageability</span></span>

<span data-ttu-id="d9599-155">Mönstret mellan molnet säkerställer sömlös hantering och välbekanta gränssnitt mellan miljöer.</span><span class="sxs-lookup"><span data-stu-id="d9599-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="d9599-156">När du ska använda det här mönstret</span><span class="sxs-lookup"><span data-stu-id="d9599-156">When to use this pattern</span></span>

<span data-ttu-id="d9599-157">Använd det här mönstret:</span><span class="sxs-lookup"><span data-stu-id="d9599-157">Use this pattern:</span></span>

- <span data-ttu-id="d9599-158">När du behöver öka din app-kapacitet med oväntade krav eller periodiska krav på begäran.</span><span class="sxs-lookup"><span data-stu-id="d9599-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="d9599-159">När du inte vill investera i resurser som endast kommer att användas under toppar.</span><span class="sxs-lookup"><span data-stu-id="d9599-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="d9599-160">Betala för det du använder.</span><span class="sxs-lookup"><span data-stu-id="d9599-160">Pay for what you use.</span></span>

<span data-ttu-id="d9599-161">Det här mönstret rekommenderas inte när:</span><span class="sxs-lookup"><span data-stu-id="d9599-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="d9599-162">Din lösning kräver att användare ansluter via Internet.</span><span class="sxs-lookup"><span data-stu-id="d9599-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="d9599-163">Ditt företag har lokala bestämmelser som kräver att den ursprungliga anslutningen kommer från ett samtal på plats.</span><span class="sxs-lookup"><span data-stu-id="d9599-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="d9599-164">Nätverket upplever vanliga Flask halsar som begränsar skalningens prestanda.</span><span class="sxs-lookup"><span data-stu-id="d9599-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="d9599-165">Din miljö är frånkopplad från Internet och kan inte komma åt det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="d9599-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d9599-166">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="d9599-166">Next steps</span></span>

<span data-ttu-id="d9599-167">Mer information om ämnen som introduceras i den här artikeln:</span><span class="sxs-lookup"><span data-stu-id="d9599-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="d9599-168">I [översikten över Azure-Traffic Manager](/azure/traffic-manager/traffic-manager-overview) kan du läsa mer om hur den här DNS-baserade trafikbelastnings utjämningen fungerar.</span><span class="sxs-lookup"><span data-stu-id="d9599-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="d9599-169">Se [design överväganden för hybrid program](overview-app-design-considerations.md) för att lära dig mer om bästa praxis och få svar på eventuella ytterligare frågor.</span><span class="sxs-lookup"><span data-stu-id="d9599-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="d9599-170">Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.</span><span class="sxs-lookup"><span data-stu-id="d9599-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="d9599-171">När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för Cross-Cloud skalnings lösning](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="d9599-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="d9599-172">Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.</span><span class="sxs-lookup"><span data-stu-id="d9599-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="d9599-173">Du får lära dig hur du skapar en lösning för flera moln för att tillhandahålla en manuellt utlöst process för växling från en Azure Stack hubben webbapp till en Azure-värdbaserad webbapp.</span><span class="sxs-lookup"><span data-stu-id="d9599-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="d9599-174">Du lär dig också hur du använder autoskalning via Traffic Manager, vilket garanterar flexibelt och skalbart moln verktyg vid belastning.</span><span class="sxs-lookup"><span data-stu-id="d9599-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
