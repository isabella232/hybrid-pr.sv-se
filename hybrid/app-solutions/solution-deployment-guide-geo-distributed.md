---
title: Dirigera trafik med en geo-distribuerad app med Azure och Azure Stack hubb
description: Lär dig hur du dirigerar trafik till vissa slut punkter med en geo-distribuerad app-lösning med hjälp av Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 9fa2c351d2c13d85fe1adb17a35e165de96ea2a2
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895439"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="97165-103">Dirigera trafik med en geo-distribuerad app med Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="97165-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="97165-104">Lär dig hur du dirigerar trafik till vissa slut punkter baserat på olika mått med hjälp av mönstret för geo-distribuerade appar.</span><span class="sxs-lookup"><span data-stu-id="97165-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="97165-105">Genom att skapa en Traffic Manager profil med geografisk Routning och slut punkts konfiguration ser du till att information dirigeras till slut punkter baserat på regionala krav, företags-och internationella regler och dina data behov.</span><span class="sxs-lookup"><span data-stu-id="97165-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="97165-106">I den här lösningen skapar du en exempel miljö för att:</span><span class="sxs-lookup"><span data-stu-id="97165-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="97165-107">Skapa en geo-distribuerad app.</span><span class="sxs-lookup"><span data-stu-id="97165-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="97165-108">Använd Traffic Manager för att rikta din app.</span><span class="sxs-lookup"><span data-stu-id="97165-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="97165-109">Använd mönstret för geo-distribuerade appar</span><span class="sxs-lookup"><span data-stu-id="97165-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="97165-110">Med det geo-distribuerade mönstret omfattar din app regioner.</span><span class="sxs-lookup"><span data-stu-id="97165-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="97165-111">Du kan använda det offentliga molnet som standard, men vissa av användarna kan kräva att deras data finns kvar i sin region.</span><span class="sxs-lookup"><span data-stu-id="97165-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="97165-112">Du kan dirigera användare till det mest lämpliga molnet baserat på deras krav.</span><span class="sxs-lookup"><span data-stu-id="97165-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="97165-113">Problem och överväganden</span><span class="sxs-lookup"><span data-stu-id="97165-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="97165-114">Skalbarhetsöverväganden</span><span class="sxs-lookup"><span data-stu-id="97165-114">Scalability considerations</span></span>

<span data-ttu-id="97165-115">Lösningen som du skapar med den här artikeln är inte att hantera skalbarhet.</span><span class="sxs-lookup"><span data-stu-id="97165-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="97165-116">Men om det används i kombination med andra Azure-lösningar och lokala lösningar, kan du hantera skalbarhets krav.</span><span class="sxs-lookup"><span data-stu-id="97165-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="97165-117">Information om hur du skapar en hybrid lösning med automatisk skalning via Traffic Manager finns i [skapa lösningar för skalning av globala moln med Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="97165-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="97165-118">Överväganden för tillgänglighet</span><span class="sxs-lookup"><span data-stu-id="97165-118">Availability considerations</span></span>

<span data-ttu-id="97165-119">När det gäller skalbarhet är den här lösningen inte direkt tillgänglig.</span><span class="sxs-lookup"><span data-stu-id="97165-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="97165-120">Men Azure och lokala lösningar kan implementeras i den här lösningen för att säkerställa hög tillgänglighet för alla komponenter som ingår.</span><span class="sxs-lookup"><span data-stu-id="97165-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="97165-121">När du ska använda det här mönstret</span><span class="sxs-lookup"><span data-stu-id="97165-121">When to use this pattern</span></span>

- <span data-ttu-id="97165-122">Din organisation har internationella grenar som kräver anpassade regionala säkerhets-och distributions principer.</span><span class="sxs-lookup"><span data-stu-id="97165-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="97165-123">Var och en av organisationens kontor hämtar personal-, affärs-och anläggnings data, som kräver rapporterings aktivitet enligt lokala förordningar och tids zoner.</span><span class="sxs-lookup"><span data-stu-id="97165-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="97165-124">Storskaliga krav uppfylls genom att skala ut appar vågrätt med flera distributioner av appar inom en enda region och mellan regioner för att hantera extrema belastnings krav.</span><span class="sxs-lookup"><span data-stu-id="97165-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="97165-125">Planera topologin</span><span class="sxs-lookup"><span data-stu-id="97165-125">Planning the topology</span></span>

<span data-ttu-id="97165-126">Innan du skapar en distribuerad app kan du känna till följande saker:</span><span class="sxs-lookup"><span data-stu-id="97165-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="97165-127">**Anpassad domän för appen:** Vad är det anpassade domän namnet som kunder kommer att använda för att få åtkomst till appen?</span><span class="sxs-lookup"><span data-stu-id="97165-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="97165-128">För exempel appen är det anpassade domän namnet www- *\. scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="97165-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="97165-129">**Traffic Manager domän:** Du väljer ett domän namn när du skapar en [Azure Traffic Manager-profil](/azure/traffic-manager/traffic-manager-manage-profiles).</span><span class="sxs-lookup"><span data-stu-id="97165-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="97165-130">Det här namnet kombineras med *trafficmanager.net* -suffixet för att registrera en domän post som hanteras av Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="97165-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="97165-131">För exempel appen är det valda namnet *skalbart-ASE-demo*.</span><span class="sxs-lookup"><span data-stu-id="97165-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="97165-132">Därför är det fullständiga domän namnet som hanteras av Traffic Manager *Scalable-ASE-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="97165-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="97165-133">**Strategi för skalning av appens avtryck:** Bestäm om appens utrymme ska distribueras över flera App Service miljöer i en enda region, flera regioner eller en blandning av båda metoderna.</span><span class="sxs-lookup"><span data-stu-id="97165-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="97165-134">Beslutet bör baseras på förväntningar av var kund trafiken kommer att slutföras och hur väl resten av en Apps stödjande backend-infrastruktur kan skalas.</span><span class="sxs-lookup"><span data-stu-id="97165-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="97165-135">Till exempel, med en tillstånds lös app på 100%, kan en app skalas enorma med en kombination av flera App Service miljöer per Azure-region, multiplicerat med App Service miljöer som distribuerats över flera Azure-regioner.</span><span class="sxs-lookup"><span data-stu-id="97165-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="97165-136">Med 15 globala Azure-regioner som är tillgängliga för att välja bland kan kunder verkligen bygga en världs omfattande storskalig app.</span><span class="sxs-lookup"><span data-stu-id="97165-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="97165-137">För exempel programmet som används här har tre App Service miljöer skapats i en enda Azure-region (södra centrala USA).</span><span class="sxs-lookup"><span data-stu-id="97165-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="97165-138">**Namngivnings konvention för App Service miljöer:** Varje App Service miljö kräver ett unikt namn.</span><span class="sxs-lookup"><span data-stu-id="97165-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="97165-139">Utöver en eller två App Service miljöer är det bra att ha en namngivnings konvention som hjälper dig att identifiera varje App Service miljö.</span><span class="sxs-lookup"><span data-stu-id="97165-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="97165-140">För den exempel app som används här användes en enkel namngivnings konvention.</span><span class="sxs-lookup"><span data-stu-id="97165-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="97165-141">Namnen på de tre App Services miljöerna är *fe1ase*, *fe2ase* och *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="97165-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="97165-142">**Namngivnings konvention för apparna:** Eftersom flera instanser av appen kommer att distribueras krävs ett namn för varje instans av den distribuerade appen.</span><span class="sxs-lookup"><span data-stu-id="97165-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="97165-143">Med App Service-miljön för Power Apps kan samma app-namn användas i flera miljöer.</span><span class="sxs-lookup"><span data-stu-id="97165-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="97165-144">Eftersom varje App Service miljö har ett unikt domänsuffix kan utvecklare välja att återanvända exakt samma app-namn i varje miljö.</span><span class="sxs-lookup"><span data-stu-id="97165-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="97165-145">En utvecklare kan till exempel ha appar som heter enligt följande: *MyApp.foo1.p.azurewebsites.net*, *MyApp.foo2.p.azurewebsites.net*, *MyApp.foo3.p.azurewebsites.net* och så vidare.</span><span class="sxs-lookup"><span data-stu-id="97165-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="97165-146">För den app som används här har varje App-instans ett unikt namn.</span><span class="sxs-lookup"><span data-stu-id="97165-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="97165-147">De instans namn som används för appar är *webfrontend1*, *webfrontend2* och *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="97165-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="97165-148">![Diagram över hybrid pelare](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="97165-148">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="97165-149">Microsoft Azure Stack Hub är ett tillägg till Azure.</span><span class="sxs-lookup"><span data-stu-id="97165-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="97165-150">Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.</span><span class="sxs-lookup"><span data-stu-id="97165-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="97165-151">Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program.</span><span class="sxs-lookup"><span data-stu-id="97165-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="97165-152">Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.</span><span class="sxs-lookup"><span data-stu-id="97165-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="97165-153">Del 1: skapa en geo-distribuerad app</span><span class="sxs-lookup"><span data-stu-id="97165-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="97165-154">I den här delen skapar du en webbapp.</span><span class="sxs-lookup"><span data-stu-id="97165-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="97165-155">Skapa webbappar och publicera.</span><span class="sxs-lookup"><span data-stu-id="97165-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="97165-156">Lägg till kod i Azure databaser.</span><span class="sxs-lookup"><span data-stu-id="97165-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="97165-157">Peka appen Bygg till flera moln mål.</span><span class="sxs-lookup"><span data-stu-id="97165-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="97165-158">Hantera och konfigurera CD-processen.</span><span class="sxs-lookup"><span data-stu-id="97165-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="97165-159">Förutsättningar</span><span class="sxs-lookup"><span data-stu-id="97165-159">Prerequisites</span></span>

<span data-ttu-id="97165-160">En Azure-prenumeration och Azure Stack Hub-installation krävs.</span><span class="sxs-lookup"><span data-stu-id="97165-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="97165-161">Steg för geo-distribuerad app</span><span class="sxs-lookup"><span data-stu-id="97165-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="97165-162">Skaffa en anpassad domän och konfigurera DNS</span><span class="sxs-lookup"><span data-stu-id="97165-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="97165-163">Uppdatera DNS-zonfilen för domänen.</span><span class="sxs-lookup"><span data-stu-id="97165-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="97165-164">Azure AD kan sedan verifiera ägarskapet för det anpassade domän namnet.</span><span class="sxs-lookup"><span data-stu-id="97165-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="97165-165">Använd [Azure DNS](/azure/dns/dns-getstarted-portal) för azure/Microsoft 365/external DNS-poster i Azure eller Lägg till DNS-posten på [en annan DNS-registrator](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="97165-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="97165-166">Registrera en anpassad domän med en offentlig registrator.</span><span class="sxs-lookup"><span data-stu-id="97165-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="97165-167">Logga in hos domännamnsregistratorn för domänen.</span><span class="sxs-lookup"><span data-stu-id="97165-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="97165-168">En godkänd administratör kan krävas för att göra DNS-uppdateringar.</span><span class="sxs-lookup"><span data-stu-id="97165-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="97165-169">Uppdatera DNS-zonfilen för domänen genom att lägga till DNS-posten från Azure AD.</span><span class="sxs-lookup"><span data-stu-id="97165-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="97165-170">DNS-posten ändrar inte beteenden, till exempel e-postroutning eller webb värd.</span><span class="sxs-lookup"><span data-stu-id="97165-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="97165-171">Skapa webbappar och publicera</span><span class="sxs-lookup"><span data-stu-id="97165-171">Create web apps and publish</span></span>

<span data-ttu-id="97165-172">Konfigurera hybrid kontinuerlig integrering/kontinuerlig leverans (CI/CD) för att distribuera webbappen till Azure och Azure Stack hubb och skicka automatiskt ändringar till båda molnen.</span><span class="sxs-lookup"><span data-stu-id="97165-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="97165-173">Azure Stack hubben med rätt bilder som ska köras (Windows Server och SQL) och App Service distribution krävs.</span><span class="sxs-lookup"><span data-stu-id="97165-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="97165-174">Mer information finns i [krav för distribution av app service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="97165-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="97165-175">Lägg till kod i Azure databaser</span><span class="sxs-lookup"><span data-stu-id="97165-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="97165-176">Logga in i Visual Studio med ett **konto som har projekt skapande rättigheter** på Azure-databaser.</span><span class="sxs-lookup"><span data-stu-id="97165-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="97165-177">CI/CD kan gälla både för appens kod och infrastruktur kod.</span><span class="sxs-lookup"><span data-stu-id="97165-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="97165-178">Använd [Azure Resource Manager mallar](https://azure.microsoft.com/resources/templates/) för både privat och värdbaserad moln utveckling.</span><span class="sxs-lookup"><span data-stu-id="97165-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Ansluta till ett projekt i Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="97165-180">**Klona lagrings platsen** genom att skapa och öppna standard webb programmet.</span><span class="sxs-lookup"><span data-stu-id="97165-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Klona lagrings platser i Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="97165-182">Skapa webb program distribution i båda molnen</span><span class="sxs-lookup"><span data-stu-id="97165-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="97165-183">Redigera filen **WebApplication. CSPROJ** : Välj `Runtimeidentifier` och Lägg till `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="97165-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="97165-184">(Mer information finns i dokumentationen för den [självständiga distributionen](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)</span><span class="sxs-lookup"><span data-stu-id="97165-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Redigera webb program projekt filen i Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="97165-186">**Checka in koden till Azure databaser** med hjälp av team Explorer.</span><span class="sxs-lookup"><span data-stu-id="97165-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="97165-187">Bekräfta att **program koden** har checkats in i Azure-databaser.</span><span class="sxs-lookup"><span data-stu-id="97165-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="97165-188">Skapa build-definitionen</span><span class="sxs-lookup"><span data-stu-id="97165-188">Create the build definition</span></span>

1. <span data-ttu-id="97165-189">**Logga in på Azure-pipelines** för att bekräfta möjligheten att skapa Bygg definitioner.</span><span class="sxs-lookup"><span data-stu-id="97165-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="97165-190">Lägg till `-r win10-x64` kod.</span><span class="sxs-lookup"><span data-stu-id="97165-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="97165-191">Detta tillägg är nödvändigt för att utlösa en fristående distribution med .NET Core.</span><span class="sxs-lookup"><span data-stu-id="97165-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Lägga till kod i build-definitionen i Azure-pipeline](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="97165-193">**Kör versionen**.</span><span class="sxs-lookup"><span data-stu-id="97165-193">**Run the build**.</span></span> <span data-ttu-id="97165-194">Den [fristående distributions](/dotnet/core/deploying/deploy-with-vs#simpleSelf) processen publicerar artefakter som kan köras på Azure och Azure Stack hubben.</span><span class="sxs-lookup"><span data-stu-id="97165-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="97165-195">Använda en Azure-värdbaserad agent</span><span class="sxs-lookup"><span data-stu-id="97165-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="97165-196">Det är ett bekvämt alternativ att bygga och distribuera webbappar med hjälp av en värdbaserad agent i Azure-pipeliner.</span><span class="sxs-lookup"><span data-stu-id="97165-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="97165-197">Underhåll och uppgraderingar utförs automatiskt av Microsoft Azure, vilket möjliggör oavbruten utveckling, testning och distribution.</span><span class="sxs-lookup"><span data-stu-id="97165-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="97165-198">Hantera och konfigurera CD-processen</span><span class="sxs-lookup"><span data-stu-id="97165-198">Manage and configure the CD process</span></span>

<span data-ttu-id="97165-199">Azure DevOps Services erbjuder en mycket konfigurerbar och hanterbar pipeline för versioner till flera miljöer, till exempel utvecklings-, mellanlagrings-, frågor och produktions miljöer. inklusive krav på godkännanden i vissa steg.</span><span class="sxs-lookup"><span data-stu-id="97165-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="97165-200">Skapa versions definition</span><span class="sxs-lookup"><span data-stu-id="97165-200">Create release definition</span></span>

1. <span data-ttu-id="97165-201">Välj **plus** -knappen för att lägga till en ny version under fliken **utgåvor** i avsnittet **build och release** i Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="97165-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Skapa en versions definition i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="97165-203">Använd mallen för Azure App Service distribution.</span><span class="sxs-lookup"><span data-stu-id="97165-203">Apply the Azure App Service Deployment template.</span></span>

   ![Använd mallen för Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="97165-205">Lägg till artefakten för Azure Cloud build-appen under **Lägg till artefakt**.</span><span class="sxs-lookup"><span data-stu-id="97165-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Lägg till artefakt i Azure Cloud build i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="97165-207">Under fliken pipelines väljer du **fas, aktivitets** länk för miljön och anger värden för Azure Cloud-miljön.</span><span class="sxs-lookup"><span data-stu-id="97165-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Ange Azure-molnets miljö värden i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="97165-209">Ange **miljö namnet** och välj Azure- **prenumeration** för Azure Cloud-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="97165-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Välj Azure-prenumeration för Azure Cloud-slutpunkt i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="97165-211">Under **App Service Name** anger du det obligatoriska namnet för Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="97165-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Ange namn på Azure App Service i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="97165-213">Ange "Hosted VS2017" under **agent kön** för Azure-molnets värd miljö.</span><span class="sxs-lookup"><span data-stu-id="97165-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Ställ in agent kön för Azure-molnets värd miljö i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="97165-215">I menyn Distribuera Azure App Service väljer du ett giltigt **paket eller** en giltig mapp för miljön.</span><span class="sxs-lookup"><span data-stu-id="97165-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="97165-216">Välj **OK** för **mappens plats**.</span><span class="sxs-lookup"><span data-stu-id="97165-216">Select **OK** to **folder location**.</span></span>
  
      ![Välj paket eller mapp för Azure App Services miljö i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Dialog rutan Välj mapp 1](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="97165-219">Spara alla ändringar och gå tillbaka till **versions pipelinen**.</span><span class="sxs-lookup"><span data-stu-id="97165-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Spara ändringar i versions pipeline i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="97165-221">Lägg till en ny artefakt som väljer build för Azure Stack Hub-appen.</span><span class="sxs-lookup"><span data-stu-id="97165-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Lägg till ny artefakt för Azure Stack Hub-app i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="97165-223">Lägg till en miljö genom att använda Azure App Service distribution.</span><span class="sxs-lookup"><span data-stu-id="97165-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Lägga till miljöer i Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="97165-225">Namnge den nya miljön Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="97165-225">Name the new environment Azure Stack Hub.</span></span>

    ![Namn miljö i Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="97165-227">Hitta Azure Stack Hub-miljön under fliken **aktivitet** .</span><span class="sxs-lookup"><span data-stu-id="97165-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure Stack Hub-miljö i Azure DevOps Services i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="97165-229">Välj prenumerationen för Azure Stack Hub-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="97165-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Välj prenumerationen för Azure Stack Hub-slutpunkten i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="97165-231">Ange namnet på den Azure Stack Hub-webbappen som App Service-namn.</span><span class="sxs-lookup"><span data-stu-id="97165-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Ange Azure Stack Hub-webbappens namn i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="97165-233">Välj Azure Stack Hub-agenten.</span><span class="sxs-lookup"><span data-stu-id="97165-233">Select the Azure Stack Hub agent.</span></span>

    ![Välj Azure Stack Hub-agenten i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="97165-235">Under avsnittet Distribuera Azure App Service väljer du ett giltigt **paket eller** en giltig mapp för miljön.</span><span class="sxs-lookup"><span data-stu-id="97165-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="97165-236">Välj **OK** för mappens plats.</span><span class="sxs-lookup"><span data-stu-id="97165-236">Select **OK** to folder location.</span></span>

    ![Välj mapp för Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Dialog rutan Välj mapp 2](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="97165-239">Under fliken variabel lägger du till en variabel med namnet `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , anger värdet **True** och scope till Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="97165-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Lägg till variabel till Azure App distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="97165-241">Välj ikonen för **kontinuerlig** distribution av utlösare i båda artefakterna och aktivera utlösaren **återuppta** distribution.</span><span class="sxs-lookup"><span data-stu-id="97165-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Välj kontinuerlig distributions utlösare i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="97165-243">Välj ikonen för **för distributions** villkor i Azure Stack Hub-miljön och Ställ in utlösaren på **efter version.**</span><span class="sxs-lookup"><span data-stu-id="97165-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Välj för distributions villkor i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="97165-245">Spara alla ändringar.</span><span class="sxs-lookup"><span data-stu-id="97165-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="97165-246">Vissa inställningar för aktiviteterna kan ha definierats automatiskt som [miljövariabler](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) när du skapar en versions definition från en mall.</span><span class="sxs-lookup"><span data-stu-id="97165-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="97165-247">De här inställningarna kan inte ändras i aktivitets inställningarna. i stället måste den överordnade miljö posten väljas för att redigera de här inställningarna.</span><span class="sxs-lookup"><span data-stu-id="97165-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="97165-248">Del 2: uppdatera webb programs alternativ</span><span class="sxs-lookup"><span data-stu-id="97165-248">Part 2: Update web app options</span></span>

<span data-ttu-id="97165-249">[Azure App Service](/azure/app-service/overview) ger en mycket skalbar och automatisk korrigering av webb värd tjänst.</span><span class="sxs-lookup"><span data-stu-id="97165-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="97165-251">Mappa ett befintligt anpassat DNS-namn till Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="97165-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="97165-252">Använd en **CNAME-post** och en **a-post** för att mappa ett anpassat DNS-namn till App Service.</span><span class="sxs-lookup"><span data-stu-id="97165-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="97165-253">Mappa ett befintligt anpassat DNS-namn till Azure Web Apps</span><span class="sxs-lookup"><span data-stu-id="97165-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="97165-254">Använd en CNAME för alla anpassade DNS-namn förutom en rot domän (till exempel northwind.com).</span><span class="sxs-lookup"><span data-stu-id="97165-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="97165-255">Om du vill migrera en live-webbplats och dess DNS-domännamn till App Service kan du läsa [Migrera ett aktivt DNS-namn till Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="97165-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="97165-256">Förutsättningar</span><span class="sxs-lookup"><span data-stu-id="97165-256">Prerequisites</span></span>

<span data-ttu-id="97165-257">För att slutföra den här lösningen:</span><span class="sxs-lookup"><span data-stu-id="97165-257">To complete this solution:</span></span>

- <span data-ttu-id="97165-258">[Skapa en app service app](/azure/app-service/)eller Använd en app som skapats för en annan lösning.</span><span class="sxs-lookup"><span data-stu-id="97165-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="97165-259">Köp ett domän namn och se till att du har åtkomst till DNS-registret för domän leverantören.</span><span class="sxs-lookup"><span data-stu-id="97165-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="97165-260">Uppdatera DNS-zonfilen för domänen.</span><span class="sxs-lookup"><span data-stu-id="97165-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="97165-261">Azure AD kommer att verifiera ägarskapet för det anpassade domän namnet.</span><span class="sxs-lookup"><span data-stu-id="97165-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="97165-262">Använd [Azure DNS](/azure/dns/dns-getstarted-portal) för azure/Microsoft 365/external DNS-poster i Azure eller Lägg till DNS-posten på [en annan DNS-registrator](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="97165-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="97165-263">Registrera en anpassad domän med en offentlig registrator.</span><span class="sxs-lookup"><span data-stu-id="97165-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="97165-264">Logga in hos domännamnsregistratorn för domänen.</span><span class="sxs-lookup"><span data-stu-id="97165-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="97165-265">(En godkänd administratör kan krävas för att göra DNS-uppdateringar.)</span><span class="sxs-lookup"><span data-stu-id="97165-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="97165-266">Uppdatera DNS-zonfilen för domänen genom att lägga till DNS-posten från Azure AD.</span><span class="sxs-lookup"><span data-stu-id="97165-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="97165-267">Om du till exempel vill lägga till DNS-poster för northwindcloud.com och www \. -northwindcloud.com konfigurerar du DNS-inställningarna för rot domänen northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="97165-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="97165-268">Du kan köpa ett domän namn med hjälp av [Azure Portal](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="97165-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="97165-269">För att kunna mappa ett anpassat DNS-namn till en webbapp måste webbappens [App Service-plan](https://azure.microsoft.com/pricing/details/app-service/) vara en betalplan (**Delad**, **Basic**, **Standard** eller **Premium**).</span><span class="sxs-lookup"><span data-stu-id="97165-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="97165-270">Skapa och mappa CNAME-och A-poster</span><span class="sxs-lookup"><span data-stu-id="97165-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="97165-271">Använda DNS-poster med domänleverantör</span><span class="sxs-lookup"><span data-stu-id="97165-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="97165-272">Använd Azure DNS för att konfigurera ett anpassat DNS-namn för Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="97165-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="97165-273">Mer information finns i [Använda Azure DNS för att skapa inställningar för anpassad domän för en Azure-tjänst](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="97165-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="97165-274">Logga in på webbplatsen för huvud-providern.</span><span class="sxs-lookup"><span data-stu-id="97165-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="97165-275">Sök upp sidan för hantering av DNS-poster.</span><span class="sxs-lookup"><span data-stu-id="97165-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="97165-276">Varje domän leverantör har sitt eget gränssnitt för DNS-poster.</span><span class="sxs-lookup"><span data-stu-id="97165-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="97165-277">Leta efter områden på webbplatsen med namnet **Domännamn**, **DNS**, eller **Namnserverhantering**.</span><span class="sxs-lookup"><span data-stu-id="97165-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="97165-278">Sidan DNS-poster kan visas i **Mina domäner**.</span><span class="sxs-lookup"><span data-stu-id="97165-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="97165-279">Hitta länken som heter **zonfilen**, **DNS-poster** eller **Avancerad konfiguration**.</span><span class="sxs-lookup"><span data-stu-id="97165-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="97165-280">Skärmbilden nedan är ett exempel på en sida med DNS-poster:</span><span class="sxs-lookup"><span data-stu-id="97165-280">The following screenshot is an example of a DNS records page:</span></span>

![Exempelsida för DNS-poster](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="97165-282">I domän namn registrator väljer du **Lägg till eller skapa** för att skapa en post.</span><span class="sxs-lookup"><span data-stu-id="97165-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="97165-283">Vissa providrar har olika länkar för att lägga till olika posttyper.</span><span class="sxs-lookup"><span data-stu-id="97165-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="97165-284">Läs leverantörens dokumentation.</span><span class="sxs-lookup"><span data-stu-id="97165-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="97165-285">Lägg till en CNAME-post för att mappa en under domän till appens standard värd namn.</span><span class="sxs-lookup"><span data-stu-id="97165-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="97165-286">För www \. northwindcloud.com-domänens exempel lägger du till en CNAME-post som mappar namnet till `<app_name>.azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="97165-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="97165-287">När du har lagt till CNAME ser sidan DNS-poster ut som i följande exempel:</span><span class="sxs-lookup"><span data-stu-id="97165-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Portalnavigering till Azure-app](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="97165-289">Aktivera CNAME-postmappning i Azure</span><span class="sxs-lookup"><span data-stu-id="97165-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="97165-290">Logga in på Azure Portal på en ny flik.</span><span class="sxs-lookup"><span data-stu-id="97165-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="97165-291">Gå till App Services.</span><span class="sxs-lookup"><span data-stu-id="97165-291">Go to App Services.</span></span>

3. <span data-ttu-id="97165-292">Välj webbapp.</span><span class="sxs-lookup"><span data-stu-id="97165-292">Select web app.</span></span>

4. <span data-ttu-id="97165-293">Välj **Anpassade domäner** i det vänstra navigeringsfönstret på appsidan i Azure Portal.</span><span class="sxs-lookup"><span data-stu-id="97165-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="97165-294">Välj **+** ikonen bredvid **Lägg till värdnamn**.</span><span class="sxs-lookup"><span data-stu-id="97165-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="97165-295">Skriv det fullständigt kvalificerade domän namnet, t `www.northwindcloud.com` . ex..</span><span class="sxs-lookup"><span data-stu-id="97165-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="97165-296">Välj **Verifiera**.</span><span class="sxs-lookup"><span data-stu-id="97165-296">Select **Validate**.</span></span>

8. <span data-ttu-id="97165-297">Om det här alternativet anges lägger du till ytterligare poster av andra typer ( `A` eller `TXT` ) till domän namn registrerar DNS-poster.</span><span class="sxs-lookup"><span data-stu-id="97165-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="97165-298">Azure kommer att tillhandahålla värden och typer för dessa poster:</span><span class="sxs-lookup"><span data-stu-id="97165-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="97165-299">a.</span><span class="sxs-lookup"><span data-stu-id="97165-299">a.</span></span>  <span data-ttu-id="97165-300">En **A**-post för att mappa till appens IP-adress.</span><span class="sxs-lookup"><span data-stu-id="97165-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="97165-301">b.</span><span class="sxs-lookup"><span data-stu-id="97165-301">b.</span></span>  <span data-ttu-id="97165-302">En **TXT**-post att mappa till appens standardvärdnamn `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="97165-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="97165-303">App Service använder bara den här posten vid konfigurations tiden för att verifiera den anpassade domän ägarskapet.</span><span class="sxs-lookup"><span data-stu-id="97165-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="97165-304">Efter verifieringen tar du bort TXT-posten.</span><span class="sxs-lookup"><span data-stu-id="97165-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="97165-305">Slutför den här uppgiften på fliken domän registrator och verifiera igen tills knappen **Lägg till värdnamn** är aktive rad.</span><span class="sxs-lookup"><span data-stu-id="97165-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="97165-306">Se till att **post typen hostname** är inställd på **CNAME** (www.example.com eller någon under domän).</span><span class="sxs-lookup"><span data-stu-id="97165-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="97165-307">Välj **Lägg till värddatornamn**.</span><span class="sxs-lookup"><span data-stu-id="97165-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="97165-308">Skriv det fullständigt kvalificerade domän namnet, t `northwindcloud.com` . ex..</span><span class="sxs-lookup"><span data-stu-id="97165-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="97165-309">Välj **Verifiera**.</span><span class="sxs-lookup"><span data-stu-id="97165-309">Select **Validate**.</span></span> <span data-ttu-id="97165-310">**Lägg till** aktive rad.</span><span class="sxs-lookup"><span data-stu-id="97165-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="97165-311">Se till att **post typen hostname** är inställd på **en post** (example.com).</span><span class="sxs-lookup"><span data-stu-id="97165-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="97165-312">**Lägg till värdnamn**.</span><span class="sxs-lookup"><span data-stu-id="97165-312">**Add hostname**.</span></span>

    <span data-ttu-id="97165-313">Det kan ta lite tid innan de nya värdarna visas på sidan **anpassade domäner** för appen.</span><span class="sxs-lookup"><span data-stu-id="97165-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="97165-314">Försök att uppdatera webbläsaren så att informationen uppdateras.</span><span class="sxs-lookup"><span data-stu-id="97165-314">Try refreshing the browser to update the data.</span></span>
  
    ![Anpassade domäner](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="97165-316">Om ett fel uppstår visas ett meddelande om verifierings fel längst ned på sidan.</span><span class="sxs-lookup"><span data-stu-id="97165-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Domän verifierings fel](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="97165-318">Ovanstående steg kan upprepas för att mappa en domän med jokertecken ( \* . northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="97165-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="97165-319">Detta gör att du kan lägga till ytterligare under domäner till den här app service utan att behöva skapa en separat CNAME-post för var och en.</span><span class="sxs-lookup"><span data-stu-id="97165-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="97165-320">Följ anvisningarna i registratorn för att konfigurera den här inställningen.</span><span class="sxs-lookup"><span data-stu-id="97165-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="97165-321">Testa i en webbläsare</span><span class="sxs-lookup"><span data-stu-id="97165-321">Test in a browser</span></span>

<span data-ttu-id="97165-322">Bläddra till det eller de DNS-namn som kon figurer ATS tidigare (till exempel `northwindcloud.com` eller `www.northwindcloud.com` ).</span><span class="sxs-lookup"><span data-stu-id="97165-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="97165-323">Del 3: binda ett anpassat SSL-certifikat</span><span class="sxs-lookup"><span data-stu-id="97165-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="97165-324">I den här delen kommer vi att:</span><span class="sxs-lookup"><span data-stu-id="97165-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="97165-325">Bind det anpassade SSL-certifikatet till App Service.</span><span class="sxs-lookup"><span data-stu-id="97165-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="97165-326">Använd HTTPS för appen.</span><span class="sxs-lookup"><span data-stu-id="97165-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="97165-327">Automatisera SSL-certifikat bindning med skript.</span><span class="sxs-lookup"><span data-stu-id="97165-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="97165-328">Om det behövs kan du skaffa ett kund-SSL-certifikat i Azure Portal och binda det till webbappen.</span><span class="sxs-lookup"><span data-stu-id="97165-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="97165-329">Mer information finns i [själv studie kursen om App Service certifikat](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="97165-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="97165-330">Förutsättningar</span><span class="sxs-lookup"><span data-stu-id="97165-330">Prerequisites</span></span>

<span data-ttu-id="97165-331">För att slutföra den här lösningen:</span><span class="sxs-lookup"><span data-stu-id="97165-331">To complete this  solution:</span></span>

- [<span data-ttu-id="97165-332">Skapa en App Service-app.</span><span class="sxs-lookup"><span data-stu-id="97165-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="97165-333">Mappa ett anpassat DNS-namn till din webbapp.</span><span class="sxs-lookup"><span data-stu-id="97165-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="97165-334">Hämta ett SSL-certifikat från en betrodd certifikat utfärdare och Använd nyckeln för att signera begäran.</span><span class="sxs-lookup"><span data-stu-id="97165-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="97165-335">Krav för ditt SSL-certifikat</span><span class="sxs-lookup"><span data-stu-id="97165-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="97165-336">Om du vill använda ett certifikat i App Service måste certifikatet uppfylla alla följande krav:</span><span class="sxs-lookup"><span data-stu-id="97165-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="97165-337">Signerat av en betrodd certifikat utfärdare.</span><span class="sxs-lookup"><span data-stu-id="97165-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="97165-338">Exporteras som en lösenordsskyddad PFX-fil.</span><span class="sxs-lookup"><span data-stu-id="97165-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="97165-339">Innehåller en privat nyckel som är minst 2048 bitar lång.</span><span class="sxs-lookup"><span data-stu-id="97165-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="97165-340">Innehåller alla mellanliggande certifikat i certifikat kedjan.</span><span class="sxs-lookup"><span data-stu-id="97165-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="97165-341">**Elliptic Curve Cryptography (ECC)-certifikat** fungerar med App Service, men ingår inte i den här hand boken.</span><span class="sxs-lookup"><span data-stu-id="97165-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="97165-342">Kontakta en certifikat utfärdare för att få hjälp med att skapa ECC-certifikat.</span><span class="sxs-lookup"><span data-stu-id="97165-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="97165-343">Förbered webbappen</span><span class="sxs-lookup"><span data-stu-id="97165-343">Prepare the web app</span></span>

<span data-ttu-id="97165-344">För att binda ett anpassat SSL-certifikat till webbappen måste [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) finnas på nivån **Basic**, **standard** eller **Premium** .</span><span class="sxs-lookup"><span data-stu-id="97165-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="97165-345">Logga in på Azure</span><span class="sxs-lookup"><span data-stu-id="97165-345">Sign in to Azure</span></span>

1. <span data-ttu-id="97165-346">Öppna [Azure Portal](https://portal.azure.com/) och gå till webbappen.</span><span class="sxs-lookup"><span data-stu-id="97165-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="97165-347">Välj **app Services** på menyn till vänster och välj sedan namnet på webb programmet.</span><span class="sxs-lookup"><span data-stu-id="97165-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Välj webbapp i Azure Portal](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="97165-349">Kontrollera prisnivån</span><span class="sxs-lookup"><span data-stu-id="97165-349">Check the pricing tier</span></span>

1. <span data-ttu-id="97165-350">I det vänstra navigerings fönstret på sidan webb program bläddrar du till avsnittet **Inställningar** och väljer **skala upp (App Service plan)**.</span><span class="sxs-lookup"><span data-stu-id="97165-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Skala upp-menyn i en webbapp](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="97165-352">Se till att webbappen inte finns på nivån **kostnads fri** eller **delad** .</span><span class="sxs-lookup"><span data-stu-id="97165-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="97165-353">Webbappens aktuella nivå markeras i en mörkblå ruta.</span><span class="sxs-lookup"><span data-stu-id="97165-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Kontrol lera pris nivån i webb programmet](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="97165-355">Anpassad SSL stöds inte på nivån **kostnads fri** eller **delad** .</span><span class="sxs-lookup"><span data-stu-id="97165-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="97165-356">Följ stegen i nästa avsnitt eller sidan **Välj pris nivå** och hoppa över för att [Ladda upp och binda ditt SSL-certifikat](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="97165-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="97165-357">Skala upp App Service-planen</span><span class="sxs-lookup"><span data-stu-id="97165-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="97165-358">Välj en av nivåerna **Basic**, **Standard** eller **Premium**.</span><span class="sxs-lookup"><span data-stu-id="97165-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="97165-359">Välj **Välj**.</span><span class="sxs-lookup"><span data-stu-id="97165-359">Select **Select**.</span></span>

![Välj pris nivå för din webbapp](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="97165-361">Skalnings åtgärden har slutförts när meddelandet visas.</span><span class="sxs-lookup"><span data-stu-id="97165-361">The scale operation is complete when notification is displayed.</span></span>

![Uppskalningsmeddelande](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="97165-363">Bind ditt SSL-certifikat och slå samman mellanliggande certifikat</span><span class="sxs-lookup"><span data-stu-id="97165-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="97165-364">Slå samman flera certifikat i kedjan.</span><span class="sxs-lookup"><span data-stu-id="97165-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="97165-365">**Öppna varje certifikat** som du har fått i en text redigerare.</span><span class="sxs-lookup"><span data-stu-id="97165-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="97165-366">Skapa en fil för det sammanfogade certifikatet med namnet *mergedcertificate. CRT*.</span><span class="sxs-lookup"><span data-stu-id="97165-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="97165-367">I redigeringsprogrammet kopierar du innehållet i varje certifikat till den här filen.</span><span class="sxs-lookup"><span data-stu-id="97165-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="97165-368">Ordningen på dina certifikat ska följa ordningen i certifikatkedjan, först med ditt certifikat och sist med rotcertifikatet.</span><span class="sxs-lookup"><span data-stu-id="97165-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="97165-369">Det ser ut som i följande exempel:</span><span class="sxs-lookup"><span data-stu-id="97165-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="97165-370">Exportera certifikat till PFX</span><span class="sxs-lookup"><span data-stu-id="97165-370">Export certificate to PFX</span></span>

<span data-ttu-id="97165-371">Exportera det sammanslagna SSL-certifikatet med den privata nyckel som genereras av certifikatet.</span><span class="sxs-lookup"><span data-stu-id="97165-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="97165-372">En privat nyckel fil skapas via OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="97165-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="97165-373">Om du vill exportera certifikatet till PFX kör du följande kommando och ersätter plats hållarna `<private-key-file>` och `<merged-certificate-file>` med sökvägen till den privata nyckeln och den sammanslagna certifikat filen:</span><span class="sxs-lookup"><span data-stu-id="97165-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="97165-374">När du uppmanas till det anger du ett export lösen ord för att ladda upp SSL-certifikatet till App Service senare.</span><span class="sxs-lookup"><span data-stu-id="97165-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="97165-375">När IIS eller **Certreq.exe** används för att generera en certifikatbegäran installerar du certifikatet på en lokal dator och [exporterar certifikatet till PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="97165-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="97165-376">Ladda upp SSL-certifikatet</span><span class="sxs-lookup"><span data-stu-id="97165-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="97165-377">Välj **SSL-inställningar** i den vänstra navigeringen i webbappen.</span><span class="sxs-lookup"><span data-stu-id="97165-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="97165-378">Välj **överför certifikat**.</span><span class="sxs-lookup"><span data-stu-id="97165-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="97165-379">I **PFX-certifikatfil** väljer du PFX-fil.</span><span class="sxs-lookup"><span data-stu-id="97165-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="97165-380">I **certifikat lösen ord** skriver du det lösen ord som du skapade när du exporterade PFX-filen.</span><span class="sxs-lookup"><span data-stu-id="97165-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="97165-381">Välj **Överför**.</span><span class="sxs-lookup"><span data-stu-id="97165-381">Select **Upload**.</span></span>

    ![Överför SSL-certifikat](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="97165-383">När App Service har laddat upp certifikatet visas det på sidan **SSL-inställningar** .</span><span class="sxs-lookup"><span data-stu-id="97165-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![SSL-inställningar](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="97165-385">Binda SSL-certifikatet</span><span class="sxs-lookup"><span data-stu-id="97165-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="97165-386">I avsnittet **SSL-bindningar** väljer du **Lägg till bindning**.</span><span class="sxs-lookup"><span data-stu-id="97165-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="97165-387">Om certifikatet har laddats upp, men inte visas i domän namn i list rutan **hostname** , försöker du uppdatera webb sidan.</span><span class="sxs-lookup"><span data-stu-id="97165-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="97165-388">På sidan **Lägg till SSL-bindning** använder du List rutan för att välja det domän namn som ska skyddas och det certifikat som ska användas.</span><span class="sxs-lookup"><span data-stu-id="97165-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="97165-389">I **SSL-typ** väljer du om du vill använda [**Servernamnindikator (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) eller IP-baserad SSL.</span><span class="sxs-lookup"><span data-stu-id="97165-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="97165-390">**SNI-baserad SSL**: flera SNI-baserade SSL-bindningar kan läggas till.</span><span class="sxs-lookup"><span data-stu-id="97165-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="97165-391">Med det här alternativet kan flera SSL-certifikat skydda flera domäner på samma IP-adress.</span><span class="sxs-lookup"><span data-stu-id="97165-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="97165-392">De flesta moderna webbläsare (inklusive Internet Explorer, Chrome, Firefox och Opera) stöder SNI (mer information om webbläsare som stöds finns i [Servernamnindikator](https://wikipedia.org/wiki/Server_Name_Indication)).</span><span class="sxs-lookup"><span data-stu-id="97165-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="97165-393">**IP-baserad SSL**: det går bara att lägga till en IP-baserad SSL-bindning.</span><span class="sxs-lookup"><span data-stu-id="97165-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="97165-394">Med det här alternativet tillåts endast ett SSL-certifikat för att skydda en dedikerad offentlig IP-adress.</span><span class="sxs-lookup"><span data-stu-id="97165-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="97165-395">Skydda flera domäner genom att skydda dem med samma SSL-certifikat.</span><span class="sxs-lookup"><span data-stu-id="97165-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="97165-396">IP-baserad SSL är det traditionella alternativet för SSL-bindning.</span><span class="sxs-lookup"><span data-stu-id="97165-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="97165-397">Välj **Lägg till bindning**.</span><span class="sxs-lookup"><span data-stu-id="97165-397">Select **Add Binding**.</span></span>

    ![Lägg till SSL-bindning](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="97165-399">När App Service har laddat upp certifikatet visas det i avsnitten **SSL-bindningar** .</span><span class="sxs-lookup"><span data-stu-id="97165-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![SSL-bindningar har överförts](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="97165-401">Mappa om A-posten för IP SSL</span><span class="sxs-lookup"><span data-stu-id="97165-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="97165-402">Om IP-baserad SSL inte används i webbappen går du vidare till [testa https för din anpassade domän](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="97165-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="97165-403">Som standard använder webbapp en delad offentlig IP-adress.</span><span class="sxs-lookup"><span data-stu-id="97165-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="97165-404">När certifikatet är kopplat till IP-baserad SSL skapar App Service en ny och dedikerad IP-adress för webbappen.</span><span class="sxs-lookup"><span data-stu-id="97165-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="97165-405">När en A-post mappas till webbappen måste domän registret uppdateras med den dedikerade IP-adressen.</span><span class="sxs-lookup"><span data-stu-id="97165-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="97165-406">Sidan **anpassad domän** uppdateras med den nya, dedikerade IP-adressen.</span><span class="sxs-lookup"><span data-stu-id="97165-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="97165-407">Kopiera den här [IP-adressen](/azure/app-service/app-service-web-tutorial-custom-domain)och mappa sedan om A- [posten](/azure/app-service/app-service-web-tutorial-custom-domain) till den nya IP-adressen.</span><span class="sxs-lookup"><span data-stu-id="97165-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="97165-408">Testa HTTPS</span><span class="sxs-lookup"><span data-stu-id="97165-408">Test HTTPS</span></span>

<span data-ttu-id="97165-409">I olika webbläsare går du till `https://<your.custom.domain>` för att se till att webbappen hanteras.</span><span class="sxs-lookup"><span data-stu-id="97165-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Bläddra till webb program](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="97165-411">Om certifikat verifierings fel inträffar kan ett självsignerat certifikat vara orsaken, eller så kan mellanliggande certifikat ha lämnats kvar vid export till PFX-filen.</span><span class="sxs-lookup"><span data-stu-id="97165-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="97165-412">Använda HTTPS</span><span class="sxs-lookup"><span data-stu-id="97165-412">Enforce HTTPS</span></span>

<span data-ttu-id="97165-413">Som standard har vem som helst åtkomst till webbappen med HTTP.</span><span class="sxs-lookup"><span data-stu-id="97165-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="97165-414">Alla HTTP-förfrågningar till HTTPS-porten kan omdirigeras.</span><span class="sxs-lookup"><span data-stu-id="97165-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="97165-415">På sidan webb program väljer du **SL-inställningar**.</span><span class="sxs-lookup"><span data-stu-id="97165-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="97165-416">I **Endast HTTPS** väljer du **På**.</span><span class="sxs-lookup"><span data-stu-id="97165-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Använda HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="97165-418">När åtgärden har slutförts går du till någon av de HTTP-URL: er som pekar på appen.</span><span class="sxs-lookup"><span data-stu-id="97165-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="97165-419">Exempel:</span><span class="sxs-lookup"><span data-stu-id="97165-419">For example:</span></span>

- <span data-ttu-id="97165-420"> https://<app_name>. azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="97165-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="97165-421">Använda TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="97165-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="97165-422">Appen tillåter [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0 som standard, vilket inte längre anses vara säkert av bransch standarder (t. ex. [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span><span class="sxs-lookup"><span data-stu-id="97165-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="97165-423">Följ dessa steg om du vill göra en högre TLS-version obligatorisk:</span><span class="sxs-lookup"><span data-stu-id="97165-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="97165-424">På sidan webbapp i det vänstra navigerings fönstret väljer du SSL- **Inställningar**.</span><span class="sxs-lookup"><span data-stu-id="97165-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="97165-425">I **TLS-version** väljer du den lägsta TLS-versionen.</span><span class="sxs-lookup"><span data-stu-id="97165-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Kräv TLS 1.1 eller 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="97165-427">Skapa en Traffic Manager-profil</span><span class="sxs-lookup"><span data-stu-id="97165-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="97165-428">Välj **skapa en resurs**  >  **nätverk**  >  **Traffic Manager profil**  >  **skapa**.</span><span class="sxs-lookup"><span data-stu-id="97165-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="97165-429">I **Skapa Traffic Manager-profil** gör du följande:</span><span class="sxs-lookup"><span data-stu-id="97165-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="97165-430">I **namn** anger du ett namn för profilen.</span><span class="sxs-lookup"><span data-stu-id="97165-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="97165-431">Det här namnet måste vara unikt i Traffic manager.net-zonen och resulterar i DNS-namnet trafficmanager.net, som används för att få åtkomst till den Traffic Manager profilen.</span><span class="sxs-lookup"><span data-stu-id="97165-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="97165-432">I **routningsmetod** väljer du **metoden geografisk routning**.</span><span class="sxs-lookup"><span data-stu-id="97165-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="97165-433">I **prenumeration** väljer du den prenumeration som du vill skapa profilen under.</span><span class="sxs-lookup"><span data-stu-id="97165-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="97165-434">I **Resursgrupp** skapar du en ny resursgrupp att placera profilen under.</span><span class="sxs-lookup"><span data-stu-id="97165-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="97165-435">I **Resursgruppsplats** väljer du plats för resursgruppen.</span><span class="sxs-lookup"><span data-stu-id="97165-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="97165-436">Den här inställningen refererar till platsen för resurs gruppen och har ingen inverkan på den Traffic Manager profilen som distribueras globalt.</span><span class="sxs-lookup"><span data-stu-id="97165-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="97165-437">Välj **Skapa**.</span><span class="sxs-lookup"><span data-stu-id="97165-437">Select **Create**.</span></span>

    7. <span data-ttu-id="97165-438">När den globala distributionen av Traffic Managers profilen är klar visas den i respektive resurs grupp som en av resurserna.</span><span class="sxs-lookup"><span data-stu-id="97165-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Resurs grupper i skapa Traffic Manager profil](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="97165-440">Lägga till Traffic Manager-slutpunkter</span><span class="sxs-lookup"><span data-stu-id="97165-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="97165-441">I Portal Sök fältet söker du efter **Traffic Manager profil** namn som skapades i föregående avsnitt och väljer Traffic Manager-profilen i de resultat som visas.</span><span class="sxs-lookup"><span data-stu-id="97165-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="97165-442">I **Traffic Manager profil** i avsnittet **Inställningar** väljer du **slut punkter**.</span><span class="sxs-lookup"><span data-stu-id="97165-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="97165-443">Välj **Lägg till**.</span><span class="sxs-lookup"><span data-stu-id="97165-443">Select **Add**.</span></span>

4. <span data-ttu-id="97165-444">Lägger till Azure Stack Hub-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="97165-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="97165-445">I **typ** väljer du **extern slut punkt**.</span><span class="sxs-lookup"><span data-stu-id="97165-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="97165-446">Ange ett **namn** för den här slut punkten, helst namnet på Azure Stack hubben.</span><span class="sxs-lookup"><span data-stu-id="97165-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="97165-447">För fullständigt kvalificerat domän namn (**FQDN**) använder du den externa URL: en för webbappen för Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="97165-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="97165-448">Under geo-mappning väljer du en region/kontinent där resursen finns.</span><span class="sxs-lookup"><span data-stu-id="97165-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="97165-449">Till exempel **Europa.**</span><span class="sxs-lookup"><span data-stu-id="97165-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="97165-450">Under List rutan land/region som visas väljer du det land som gäller för den här slut punkten.</span><span class="sxs-lookup"><span data-stu-id="97165-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="97165-451">Till exempel **Tyskland**.</span><span class="sxs-lookup"><span data-stu-id="97165-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="97165-452">Behåll **Lägg till som inaktiverad** som avmarkerat.</span><span class="sxs-lookup"><span data-stu-id="97165-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="97165-453">Välj **OK**.</span><span class="sxs-lookup"><span data-stu-id="97165-453">Select **OK**.</span></span>

12. <span data-ttu-id="97165-454">Lägger till Azure-slutpunkt:</span><span class="sxs-lookup"><span data-stu-id="97165-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="97165-455">I **typ** väljer du **Azure-slutpunkt**.</span><span class="sxs-lookup"><span data-stu-id="97165-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="97165-456">Ange ett **namn** för slut punkten.</span><span class="sxs-lookup"><span data-stu-id="97165-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="97165-457">För **mål resurs typ** väljer du **App Service**.</span><span class="sxs-lookup"><span data-stu-id="97165-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="97165-458">För **mål resurs** väljer du **Välj en app service** för att visa listan över Web Apps under samma prenumeration.</span><span class="sxs-lookup"><span data-stu-id="97165-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="97165-459">I **resurs** väljer du App Service som används som första slut punkt.</span><span class="sxs-lookup"><span data-stu-id="97165-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="97165-460">Under geo-mappning väljer du en region/kontinent där resursen finns.</span><span class="sxs-lookup"><span data-stu-id="97165-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="97165-461">Till exempel **Nordamerika/Central Amerika/Karibien.**</span><span class="sxs-lookup"><span data-stu-id="97165-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="97165-462">Under List rutan land/region som visas lämnar du den här punkten tom för att välja hela den regionala grupperingen.</span><span class="sxs-lookup"><span data-stu-id="97165-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="97165-463">Behåll **Lägg till som inaktiverad** som avmarkerat.</span><span class="sxs-lookup"><span data-stu-id="97165-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="97165-464">Välj **OK**.</span><span class="sxs-lookup"><span data-stu-id="97165-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="97165-465">Skapa minst en slut punkt med en geografisk omfattning av alla (värld) som ska fungera som standard slut punkt för resursen.</span><span class="sxs-lookup"><span data-stu-id="97165-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="97165-466">När båda slut punkterna har lagts till visas de i **Traffic Manager profil** tillsammans med deras övervaknings status som **online**.</span><span class="sxs-lookup"><span data-stu-id="97165-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Slut punkts status för Traffic Manager profil](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="97165-468">Globalt företag är beroende av Azures funktioner för geo-distribution</span><span class="sxs-lookup"><span data-stu-id="97165-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="97165-469">Genom att dirigera data trafik via Azure Traffic Manager och geografibaserade slut punkter kan globala företag följa regionala bestämmelser och hålla data kompatibla och säkra, vilket är viktigt för att det ska bli både lokala och fjärranslutna affärs platser.</span><span class="sxs-lookup"><span data-stu-id="97165-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="97165-470">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="97165-470">Next steps</span></span>

- <span data-ttu-id="97165-471">Mer information om moln mönster i Azure finns i [design mönster för molnet](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="97165-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
