---
title: Hybrid relä mönster i Azure och Azure Stack hubb
description: Använd hybrid relä mönstret i Azure och Azure Stack Hub för att ansluta till kant resurser som skyddas av brand väggar.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911970"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="c4b0c-103">Hybrid relä mönster</span><span class="sxs-lookup"><span data-stu-id="c4b0c-103">Hybrid relay pattern</span></span>

<span data-ttu-id="c4b0c-104">Lär dig hur du ansluter till Edge-resurser eller enheter som skyddas av brand väggar med hjälp av hybrid relä mönstret och Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c4b0c-105">Kontext och problem</span><span class="sxs-lookup"><span data-stu-id="c4b0c-105">Context and problem</span></span>

<span data-ttu-id="c4b0c-106">Gräns enheter är ofta bakom en företags brand vägg eller NAT-enhet.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="c4b0c-107">Även om de är säkra kan de inte kommunicera med det offentliga molnet eller gräns enheterna i andra företags nätverk.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="c4b0c-108">Det kan vara nödvändigt att exponera vissa portar och funktioner för användare i det offentliga molnet på ett säkert sätt.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="c4b0c-109">Lösning</span><span class="sxs-lookup"><span data-stu-id="c4b0c-109">Solution</span></span>

<span data-ttu-id="c4b0c-110">Hybrid relä mönstret använder Azure Relay för att upprätta en WebSockets-tunnel mellan två slut punkter som inte kan kommunicera direkt.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="c4b0c-111">Enheter som inte är lokala men som måste anslutas till en lokal slut punkt ansluter till en slut punkt i det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="c4b0c-112">Den här slut punkten omdirigerar trafiken på fördefinierade vägar över en säker kanal.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="c4b0c-113">En slut punkt i den lokala miljön tar emot trafiken och dirigerar den till rätt mål.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![lösnings arkitektur för Hybrid relä mönster](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="c4b0c-115">Så här fungerar hybrid relä mönstret:</span><span class="sxs-lookup"><span data-stu-id="c4b0c-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="c4b0c-116">En enhet ansluter till den virtuella datorn (VM) i Azure på en fördefinierad port.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="c4b0c-117">Trafiken vidarebefordras till Azure Relay i Azure.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="c4b0c-118">Den virtuella datorn på Azure Stack Hub, som redan har upprättat en lång livs längd anslutning till Azure Relay, tar emot trafiken och vidarebefordrar den till målet.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="c4b0c-119">Den lokala tjänsten eller slut punkten bearbetar begäran.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="c4b0c-120">Komponenter</span><span class="sxs-lookup"><span data-stu-id="c4b0c-120">Components</span></span>

<span data-ttu-id="c4b0c-121">Den här lösningen använder följande komponenter:</span><span class="sxs-lookup"><span data-stu-id="c4b0c-121">This solution uses the following components:</span></span>

| <span data-ttu-id="c4b0c-122">Lager</span><span class="sxs-lookup"><span data-stu-id="c4b0c-122">Layer</span></span> | <span data-ttu-id="c4b0c-123">Komponent</span><span class="sxs-lookup"><span data-stu-id="c4b0c-123">Component</span></span> | <span data-ttu-id="c4b0c-124">Description</span><span class="sxs-lookup"><span data-stu-id="c4b0c-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="c4b0c-125">Azure</span><span class="sxs-lookup"><span data-stu-id="c4b0c-125">Azure</span></span> | <span data-ttu-id="c4b0c-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="c4b0c-126">Azure VM</span></span> | <span data-ttu-id="c4b0c-127">En virtuell Azure-dator tillhandahåller en offentligt tillgänglig slut punkt för den lokala resursen.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="c4b0c-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="c4b0c-128">Azure Relay</span></span> | <span data-ttu-id="c4b0c-129">En [Azure Relay](/azure/azure-relay/) tillhandahåller infrastrukturen för att underhålla tunneln och anslutningen mellan den virtuella Azure-datorn och Azure Stack Hub-VM.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="c4b0c-130">Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="c4b0c-130">Azure Stack Hub</span></span> | <span data-ttu-id="c4b0c-131">Compute</span><span class="sxs-lookup"><span data-stu-id="c4b0c-131">Compute</span></span> | <span data-ttu-id="c4b0c-132">En Azure Stack Hub-VM innehåller hybrid relä tunnelns Server sida.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="c4b0c-133">Storage</span><span class="sxs-lookup"><span data-stu-id="c4b0c-133">Storage</span></span> | <span data-ttu-id="c4b0c-134">AKS-motorns kluster som distribueras till Azure Stack Hub ger en skalbar, elastisk motor för att köra Ansikts-API containern.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="c4b0c-135">Problem och överväganden</span><span class="sxs-lookup"><span data-stu-id="c4b0c-135">Issues and considerations</span></span>

<span data-ttu-id="c4b0c-136">Tänk på följande när du bestämmer hur du ska implementera den här lösningen:</span><span class="sxs-lookup"><span data-stu-id="c4b0c-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="c4b0c-137">Skalbarhet</span><span class="sxs-lookup"><span data-stu-id="c4b0c-137">Scalability</span></span>

<span data-ttu-id="c4b0c-138">Det här mönstret tillåter endast 1:1 port mappningar på klienten och servern.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="c4b0c-139">Om port 80 till exempel är tunnlad för en tjänst på Azure-slutpunkten kan den inte användas för en annan tjänst.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="c4b0c-140">Port mappningar bör därför planeras.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="c4b0c-141">Den Azure Relay och de virtuella datorerna bör skalas på rätt sätt för att hantera trafik.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="c4b0c-142">Tillgänglighet</span><span class="sxs-lookup"><span data-stu-id="c4b0c-142">Availability</span></span>

<span data-ttu-id="c4b0c-143">Dessa tunnlar och anslutningar är inte redundanta.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="c4b0c-144">För att säkerställa hög tillgänglighet kanske du vill implementera fel söknings kod.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="c4b0c-145">Ett annat alternativ är att ha en pool med Azure Relay anslutna virtuella datorer bakom en belastningsutjämnare.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="c4b0c-146">Hanterbarhet</span><span class="sxs-lookup"><span data-stu-id="c4b0c-146">Manageability</span></span>

<span data-ttu-id="c4b0c-147">Den här lösningen kan omfatta många enheter och platser, vilket kan ge svårhanterligt.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="c4b0c-148">Azures IoT-tjänster kan automatiskt placera nya platser och enheter online och hålla dem uppdaterade.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="c4b0c-149">Säkerhet</span><span class="sxs-lookup"><span data-stu-id="c4b0c-149">Security</span></span>

<span data-ttu-id="c4b0c-150">Det här mönstret som visas ger oinskränkt åtkomst till en port på en intern enhet från kanten.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="c4b0c-151">Överväg att lägga till en autentiseringsmekanism till tjänsten på den interna enheten, eller framför hybrid relä slut punkten.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c4b0c-152">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="c4b0c-152">Next steps</span></span>

<span data-ttu-id="c4b0c-153">Mer information om ämnen som introduceras i den här artikeln:</span><span class="sxs-lookup"><span data-stu-id="c4b0c-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="c4b0c-154">Det här mönstret använder Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="c4b0c-155">Mer information finns i Azure Relay- [dokumentationen](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="c4b0c-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="c4b0c-156">Se [design överväganden för hybrid program](overview-app-design-considerations.md) för att lära dig mer om bästa praxis och få svar på eventuella ytterligare frågor.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="c4b0c-157">Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="c4b0c-158">När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för Hybrid relä lösning](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="c4b0c-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="c4b0c-159">Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.</span><span class="sxs-lookup"><span data-stu-id="c4b0c-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>