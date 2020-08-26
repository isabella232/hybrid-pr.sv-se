---
title: Distribuera en app som skalar över molnet i Azure och Azure Stack hubb
description: Lär dig hur du distribuerar en app som skalar över molnet i Azure och Azure Stack hubben.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886823"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="81e1f-103">Distribuera en app som skalar över molnet med Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="81e1f-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="81e1f-104">Lär dig hur du skapar en lösning för flera moln för att tillhandahålla en manuellt utlöst process för att växla från en Azure Stack hubben webb program till en Azure-värdbaserad webbapp med automatisk skalning via Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="81e1f-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="81e1f-105">Den här processen säkerställer flexibelt och skalbart moln verktyg vid belastning.</span><span class="sxs-lookup"><span data-stu-id="81e1f-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="81e1f-106">Med det här mönstret kanske inte klienten är redo att köra din app i det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="81e1f-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="81e1f-107">Det kanske inte är ekonomiskt genomförbart för verksamheten att bibehålla kapaciteten som krävs i den lokala miljön för att hantera toppar i efter frågan på appen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="81e1f-108">Din klient organisation kan utnyttja det offentliga molnets elastiskhet med sin lokala lösning.</span><span class="sxs-lookup"><span data-stu-id="81e1f-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="81e1f-109">I den här lösningen skapar du en exempel miljö för att:</span><span class="sxs-lookup"><span data-stu-id="81e1f-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="81e1f-110">Skapa en webbapp med flera noder.</span><span class="sxs-lookup"><span data-stu-id="81e1f-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="81e1f-111">Konfigurera och hantera processen för kontinuerlig distribution (CD).</span><span class="sxs-lookup"><span data-stu-id="81e1f-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="81e1f-112">Publicera webbappen till Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81e1f-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="81e1f-113">Skapa en version.</span><span class="sxs-lookup"><span data-stu-id="81e1f-113">Create a release.</span></span>
> - <span data-ttu-id="81e1f-114">Lär dig att övervaka och spåra dina distributioner.</span><span class="sxs-lookup"><span data-stu-id="81e1f-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="81e1f-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="81e1f-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="81e1f-116">Microsoft Azure Stack Hub är ett tillägg till Azure.</span><span class="sxs-lookup"><span data-stu-id="81e1f-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="81e1f-117">Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling i din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.</span><span class="sxs-lookup"><span data-stu-id="81e1f-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="81e1f-118">Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program.</span><span class="sxs-lookup"><span data-stu-id="81e1f-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="81e1f-119">Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.</span><span class="sxs-lookup"><span data-stu-id="81e1f-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="81e1f-120">Förutsättningar</span><span class="sxs-lookup"><span data-stu-id="81e1f-120">Prerequisites</span></span>

- <span data-ttu-id="81e1f-121">En Azure-prenumeration.</span><span class="sxs-lookup"><span data-stu-id="81e1f-121">Azure subscription.</span></span> <span data-ttu-id="81e1f-122">Om det behövs kan du skapa ett [kostnads fritt konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) innan du börjar.</span><span class="sxs-lookup"><span data-stu-id="81e1f-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="81e1f-123">Ett Azure Stack hubb integrerat system eller distribution av Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="81e1f-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="81e1f-124">Anvisningar om hur du installerar Azure Stack Hub finns i [Installera ASDK](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="81e1f-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="81e1f-125">För ett Automation-skript för ASDK efter distribution går du till: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="81e1f-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="81e1f-126">Den här installationen kan ta några timmar att slutföra.</span><span class="sxs-lookup"><span data-stu-id="81e1f-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="81e1f-127">Distribuera [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS-tjänster till Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81e1f-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="81e1f-128">[Skapa planer/erbjudanden](/azure-stack/operator/service-plan-offer-subscription-overview.md) i Azure Stack Hub-miljön.</span><span class="sxs-lookup"><span data-stu-id="81e1f-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="81e1f-129">[Skapa klient prenumeration](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) i Azure Stack Hub-miljön.</span><span class="sxs-lookup"><span data-stu-id="81e1f-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="81e1f-130">Skapa en webbapp i klient prenumerationen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="81e1f-131">Anteckna den nya webb program-URL: en för senare användning.</span><span class="sxs-lookup"><span data-stu-id="81e1f-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="81e1f-132">Distribuera virtuella datorer i Azure pipelines (VM) i klient prenumerationen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="81e1f-133">Windows Server 2016 VM med .NET 3,5 krävs.</span><span class="sxs-lookup"><span data-stu-id="81e1f-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="81e1f-134">Den här virtuella datorn kommer att skapas i klient prenumerationen på Azure Stack Hub som den privata build-agenten.</span><span class="sxs-lookup"><span data-stu-id="81e1f-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="81e1f-135">[Windows Server 2016 med SQL 2017 VM-avbildning](/azure-stack/operator/azure-stack-add-vm-image.md) finns på Azure Stack Hub Marketplace.</span><span class="sxs-lookup"><span data-stu-id="81e1f-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="81e1f-136">Om avbildningen inte är tillgänglig arbetar du med en Azure Stack nav-operator för att se till att den läggs till i miljön.</span><span class="sxs-lookup"><span data-stu-id="81e1f-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="81e1f-137">Problem och överväganden</span><span class="sxs-lookup"><span data-stu-id="81e1f-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="81e1f-138">Skalbarhet</span><span class="sxs-lookup"><span data-stu-id="81e1f-138">Scalability</span></span>

<span data-ttu-id="81e1f-139">Nyckel komponenten för skalning över moln är möjligheten att leverera omedelbar skalning på begäran mellan offentliga och lokala moln infrastrukturer, vilket ger konsekvent och tillförlitlig tjänst.</span><span class="sxs-lookup"><span data-stu-id="81e1f-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="81e1f-140">Tillgänglighet</span><span class="sxs-lookup"><span data-stu-id="81e1f-140">Availability</span></span>

<span data-ttu-id="81e1f-141">Se till att lokalt distribuerade appar är konfigurerade för hög tillgänglighet via lokal maskin varu konfiguration och program varu distribution.</span><span class="sxs-lookup"><span data-stu-id="81e1f-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="81e1f-142">Hanterbarhet</span><span class="sxs-lookup"><span data-stu-id="81e1f-142">Manageability</span></span>

<span data-ttu-id="81e1f-143">Lösningen över molnet säkerställer sömlös hantering och välbekant gränssnitt mellan miljöer.</span><span class="sxs-lookup"><span data-stu-id="81e1f-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="81e1f-144">PowerShell rekommenderas för plattforms oberoende hantering.</span><span class="sxs-lookup"><span data-stu-id="81e1f-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="81e1f-145">Skalning mellan moln</span><span class="sxs-lookup"><span data-stu-id="81e1f-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="81e1f-146">Hämta en anpassad domän och konfigurera DNS</span><span class="sxs-lookup"><span data-stu-id="81e1f-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="81e1f-147">Uppdatera DNS-zonfilen för domänen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="81e1f-148">Azure AD kommer att verifiera ägarskapet för det anpassade domän namnet.</span><span class="sxs-lookup"><span data-stu-id="81e1f-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="81e1f-149">Använd [Azure DNS](/azure/dns/dns-getstarted-portal) för azure/Microsoft 365/external DNS-poster i Azure eller Lägg till DNS-posten på [en annan DNS-registrator](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="81e1f-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="81e1f-150">Registrera en anpassad domän med en offentlig registrator.</span><span class="sxs-lookup"><span data-stu-id="81e1f-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="81e1f-151">Logga in hos domännamnsregistratorn för domänen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="81e1f-152">En godkänd administratör kan krävas för att göra DNS-uppdateringar.</span><span class="sxs-lookup"><span data-stu-id="81e1f-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="81e1f-153">Uppdatera DNS-zonfilen för domänen genom att lägga till DNS-posten från Azure AD.</span><span class="sxs-lookup"><span data-stu-id="81e1f-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="81e1f-154">(DNS-posten påverkar inte e-postroutningen eller webb värd beteenden.)</span><span class="sxs-lookup"><span data-stu-id="81e1f-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="81e1f-155">Skapa en standard-webbapp med flera noder i Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="81e1f-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="81e1f-156">Konfigurera hybrid kontinuerlig integrering och kontinuerlig distribution (CI/CD) för att distribuera webbappar till Azure och Azure Stack hubb och för att skicka ändringar till båda molnen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="81e1f-157">Azure Stack hubben med rätt bilder som ska köras (Windows Server och SQL) och App Service distribution krävs.</span><span class="sxs-lookup"><span data-stu-id="81e1f-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="81e1f-158">Mer information finns i krav för App Service dokumentation [för att distribuera app service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="81e1f-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="81e1f-159">Lägg till kod i Azure databaser</span><span class="sxs-lookup"><span data-stu-id="81e1f-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="81e1f-160">Azure-lagringsplatser</span><span class="sxs-lookup"><span data-stu-id="81e1f-160">Azure Repos</span></span>

1. <span data-ttu-id="81e1f-161">Logga in på Azure-databaser med ett konto som har behörighet att skapa projekt på Azure databaser.</span><span class="sxs-lookup"><span data-stu-id="81e1f-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="81e1f-162">Hybrid CI/CD kan gälla både för appens kod och infrastruktur kod.</span><span class="sxs-lookup"><span data-stu-id="81e1f-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="81e1f-163">Använd [Azure Resource Manager mallar](https://azure.microsoft.com/resources/templates/) för både privat och värdbaserad moln utveckling.</span><span class="sxs-lookup"><span data-stu-id="81e1f-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Ansluta till ett projekt i Azure databaser](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="81e1f-165">**Klona lagrings platsen** genom att skapa och öppna standard webb programmet.</span><span class="sxs-lookup"><span data-stu-id="81e1f-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Klona lagrings platsen i Azure-webbappen](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="81e1f-167">Skapa en självständig distribution av webbappar för App Services i båda molnen</span><span class="sxs-lookup"><span data-stu-id="81e1f-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="81e1f-168">Redigera filen **WebApplication. CSPROJ** .</span><span class="sxs-lookup"><span data-stu-id="81e1f-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="81e1f-169">Välj `Runtimeidentifier` och Lägg till `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="81e1f-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="81e1f-170">(Mer information finns i dokumentationen för den [självständiga distributionen](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)</span><span class="sxs-lookup"><span data-stu-id="81e1f-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Redigera projekt fil för webbapp](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="81e1f-172">Checka in koden till Azure databaser med hjälp av team Explorer.</span><span class="sxs-lookup"><span data-stu-id="81e1f-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="81e1f-173">Bekräfta att appens kod har marker ATS i Azure-databaser.</span><span class="sxs-lookup"><span data-stu-id="81e1f-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="81e1f-174">Skapa build-definitionen</span><span class="sxs-lookup"><span data-stu-id="81e1f-174">Create the build definition</span></span>

1. <span data-ttu-id="81e1f-175">Logga in på Azure-pipelines för att bekräfta möjligheten att skapa Bygg definitioner.</span><span class="sxs-lookup"><span data-stu-id="81e1f-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="81e1f-176">Add **-r Win10-x64-** kod.</span><span class="sxs-lookup"><span data-stu-id="81e1f-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="81e1f-177">Detta tillägg är nödvändigt för att utlösa en fristående distribution med .NET Core.</span><span class="sxs-lookup"><span data-stu-id="81e1f-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Lägg till kod i webbappen](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="81e1f-179">Kör versionen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-179">Run the build.</span></span> <span data-ttu-id="81e1f-180">Den [fristående distributions](/dotnet/core/deploying/deploy-with-vs#simpleSelf) processen publicerar artefakter som körs på Azure och Azure Stack hubben.</span><span class="sxs-lookup"><span data-stu-id="81e1f-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="81e1f-181">Använd en Azure-värdbaserad agent</span><span class="sxs-lookup"><span data-stu-id="81e1f-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="81e1f-182">Det är ett bekvämt alternativ att bygga och distribuera webbappar med hjälp av en värdbaserad Bygg agent i Azure-pipeliner.</span><span class="sxs-lookup"><span data-stu-id="81e1f-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="81e1f-183">Underhåll och uppgraderingar görs automatiskt av Microsoft Azure, vilket möjliggör en kontinuerlig och oavbruten utvecklings cykel.</span><span class="sxs-lookup"><span data-stu-id="81e1f-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="81e1f-184">Hantera och konfigurera CD-processen</span><span class="sxs-lookup"><span data-stu-id="81e1f-184">Manage and configure the CD process</span></span>

<span data-ttu-id="81e1f-185">Azure-pipeliner och Azure DevOps Services erbjuder en mycket konfigurerbar och hanterbar pipeline för versioner till flera miljöer som utveckling, mellanlagring, frågor och produktions miljöer. inklusive krav på godkännanden i vissa steg.</span><span class="sxs-lookup"><span data-stu-id="81e1f-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="81e1f-186">Skapa versions definition</span><span class="sxs-lookup"><span data-stu-id="81e1f-186">Create release definition</span></span>

1. <span data-ttu-id="81e1f-187">Välj **plus** -knappen för att lägga till en ny version under fliken **utgåvor** i avsnittet **build och release** i Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="81e1f-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Skapa en versionsdefinition](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="81e1f-189">Använd mallen för Azure App Service distribution.</span><span class="sxs-lookup"><span data-stu-id="81e1f-189">Apply the Azure App Service Deployment template.</span></span>

   ![Använd mall för Azure App Service distribution](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="81e1f-191">Lägg till artefakten för Azure Cloud build-appen under **Lägg till artefakt**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Lägg till artefakt i Azure Cloud build](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="81e1f-193">Under fliken pipelines väljer du **fas, aktivitets** länk för miljön och anger värden för Azure Cloud-miljön.</span><span class="sxs-lookup"><span data-stu-id="81e1f-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Ange värden för Azure Cloud-miljön](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="81e1f-195">Ange **miljö namnet** och välj Azure- **prenumeration** för Azure Cloud-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="81e1f-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Välj Azure-prenumeration för Azures moln slut punkt](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="81e1f-197">Under **App Service Name**anger du det obligatoriska namnet för Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="81e1f-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Ange namn på Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="81e1f-199">Ange "Hosted VS2017" under **agent kön** för Azure-molnets värd miljö.</span><span class="sxs-lookup"><span data-stu-id="81e1f-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Ställ in agent kön för Azure-molnets värd miljö](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="81e1f-201">I menyn Distribuera Azure App Service väljer du ett giltigt **paket eller** en giltig mapp för miljön.</span><span class="sxs-lookup"><span data-stu-id="81e1f-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="81e1f-202">Välj **OK** för **mappens plats**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-202">Select **OK** to **folder location**.</span></span>
  
      ![Välj paket eller mapp för Azure App Services miljö](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Välj paket eller mapp för Azure App Services miljö](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="81e1f-205">Spara alla ändringar och gå tillbaka till **versions pipelinen**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Spara ändringar i versions pipeline](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="81e1f-207">Lägg till en ny artefakt som väljer build för Azure Stack Hub-appen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Lägg till ny artefakt för Azure Stack Hub-appen](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="81e1f-209">Lägg till en miljö genom att använda Azure App Service distribution.</span><span class="sxs-lookup"><span data-stu-id="81e1f-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Lägga till miljöer i Azure App Service distribution](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="81e1f-211">Namnge den nya miljön "Azure Stack".</span><span class="sxs-lookup"><span data-stu-id="81e1f-211">Name the new environment "Azure Stack".</span></span>

    ![Namn miljö i Azure App Service distribution](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="81e1f-213">Hitta Azure Stacks miljön under fliken **aktivitet** .</span><span class="sxs-lookup"><span data-stu-id="81e1f-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack miljö](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="81e1f-215">Välj prenumerationen för Azure Stack slut punkten.</span><span class="sxs-lookup"><span data-stu-id="81e1f-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Välj prenumerationen för Azure Stack slut punkten](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="81e1f-217">Ange Azure Stack webbappens namn som App Service-namn.</span><span class="sxs-lookup"><span data-stu-id="81e1f-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="81e1f-218">![Ange Azure Stack webb program namn](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="81e1f-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="81e1f-219">Välj den Azure Stack agenten.</span><span class="sxs-lookup"><span data-stu-id="81e1f-219">Select the Azure Stack agent.</span></span>

    ![Välj Azure Stack agent](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="81e1f-221">Under avsnittet Distribuera Azure App Service väljer du ett giltigt **paket eller** en giltig mapp för miljön.</span><span class="sxs-lookup"><span data-stu-id="81e1f-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="81e1f-222">Välj **OK** för mappens plats.</span><span class="sxs-lookup"><span data-stu-id="81e1f-222">Select **OK** to folder location.</span></span>

    ![Välj mapp för Azure App Service distribution](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Välj mapp för Azure App Service distribution](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="81e1f-225">Under fliken variabel lägger du till en variabel med namnet `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , anger värdet **True**och scopet till Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="81e1f-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Lägg till variabel i Azure App distribution](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="81e1f-227">Välj ikonen för **kontinuerlig** distribution av utlösare i båda artefakterna och aktivera utlösaren **återuppta** distribution.</span><span class="sxs-lookup"><span data-stu-id="81e1f-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Välj kontinuerlig distributions utlösare](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="81e1f-229">Välj ikonen för **för distributions** villkor i Azure Stacks miljön och Ställ in utlösaren på **efter version.**</span><span class="sxs-lookup"><span data-stu-id="81e1f-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Välj för distributions villkor](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="81e1f-231">Spara alla ändringar.</span><span class="sxs-lookup"><span data-stu-id="81e1f-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="81e1f-232">Vissa inställningar för aktiviteterna kan ha definierats automatiskt som [miljövariabler](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) när du skapar en versions definition från en mall.</span><span class="sxs-lookup"><span data-stu-id="81e1f-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="81e1f-233">De här inställningarna kan inte ändras i aktivitets inställningarna. i stället måste den överordnade miljö posten väljas för att redigera de här inställningarna.</span><span class="sxs-lookup"><span data-stu-id="81e1f-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="81e1f-234">Publicera till Azure Stack hubb via Visual Studio</span><span class="sxs-lookup"><span data-stu-id="81e1f-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="81e1f-235">Genom att skapa slut punkter kan en Azure DevOps Services-version Distribuera Azure-tjänsteappar till Azure Stack hubben.</span><span class="sxs-lookup"><span data-stu-id="81e1f-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="81e1f-236">Azure-pipeliner ansluter till build-agenten, som ansluter till Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81e1f-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="81e1f-237">Logga in på Azure DevOps Services och gå till appens inställnings sida.</span><span class="sxs-lookup"><span data-stu-id="81e1f-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="81e1f-238">I **Inställningar**väljer du **säkerhet**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="81e1f-239">I **VSTS-grupper**väljer du **slut punkts skapare**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="81e1f-240">På fliken **medlemmar** väljer du **Lägg till**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="81e1f-241">I **Lägg till användare och grupper anger du**ett användar namn och väljer användaren i listan över användare.</span><span class="sxs-lookup"><span data-stu-id="81e1f-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="81e1f-242">Välj **Spara ändringar**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="81e1f-243">I listan **VSTS-grupper** väljer du **slut punkts administratörer**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="81e1f-244">På fliken **medlemmar** väljer du **Lägg till**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="81e1f-245">I **Lägg till användare och grupper anger du**ett användar namn och väljer användaren i listan över användare.</span><span class="sxs-lookup"><span data-stu-id="81e1f-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="81e1f-246">Välj **Spara ändringar**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-246">Select **Save changes**.</span></span>

<span data-ttu-id="81e1f-247">Nu när slut punkts informationen finns är Azure-pipelinen till Azure Stack Hub-anslutning redo att användas.</span><span class="sxs-lookup"><span data-stu-id="81e1f-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="81e1f-248">Build-agenten i Azure Stack Hub hämtar instruktioner från Azure-pipelines och agenten förmedlar slut punkts informationen för kommunikation med Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81e1f-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="81e1f-249">Utveckla appens build</span><span class="sxs-lookup"><span data-stu-id="81e1f-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="81e1f-250">Azure Stack hubben med rätt bilder som ska köras (Windows Server och SQL) och App Service distribution krävs.</span><span class="sxs-lookup"><span data-stu-id="81e1f-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="81e1f-251">Mer information finns i [krav för distribution av app service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="81e1f-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="81e1f-252">Använd [Azure Resource Manager mallar](https://azure.microsoft.com/resources/templates/) som Web App Code från Azure databaser för att distribuera till båda molnen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="81e1f-253">Lägga till kod i ett Azure databaser-projekt</span><span class="sxs-lookup"><span data-stu-id="81e1f-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="81e1f-254">Logga in på Azure-databaser med ett konto som har skapande rättigheter för projekt på Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81e1f-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="81e1f-255">**Klona lagrings platsen** genom att skapa och öppna standard webb programmet.</span><span class="sxs-lookup"><span data-stu-id="81e1f-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="81e1f-256">Skapa en självständig distribution av webbappar för App Services i båda molnen</span><span class="sxs-lookup"><span data-stu-id="81e1f-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="81e1f-257">Redigera filen **WebApplication. CSPROJ** : Välj `Runtimeidentifier` och Lägg sedan till `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="81e1f-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="81e1f-258">Mer information finns i dokumentationen för [självständiga distributioner](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .</span><span class="sxs-lookup"><span data-stu-id="81e1f-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="81e1f-259">Använd Team Explorer för att kontrol lera koden i Azure databaser.</span><span class="sxs-lookup"><span data-stu-id="81e1f-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="81e1f-260">Bekräfta att appens kod har marker ATS i Azure-databaser.</span><span class="sxs-lookup"><span data-stu-id="81e1f-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="81e1f-261">Skapa build-definitionen</span><span class="sxs-lookup"><span data-stu-id="81e1f-261">Create the build definition</span></span>

1. <span data-ttu-id="81e1f-262">Logga in på Azure-pipelines med ett konto som kan skapa en build-definition.</span><span class="sxs-lookup"><span data-stu-id="81e1f-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="81e1f-263">Gå till sidan för att **bygga webb program** för projektet.</span><span class="sxs-lookup"><span data-stu-id="81e1f-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="81e1f-264">I **argument**, Add **-r Win10-x64** Code.</span><span class="sxs-lookup"><span data-stu-id="81e1f-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="81e1f-265">Detta tillägg krävs för att utlösa en fristående distribution med .NET Core.</span><span class="sxs-lookup"><span data-stu-id="81e1f-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="81e1f-266">Kör versionen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-266">Run the build.</span></span> <span data-ttu-id="81e1f-267">Den [fristående distributions](/dotnet/core/deploying/deploy-with-vs#simpleSelf) processen publicerar artefakter som kan köras på Azure och Azure Stack hubben.</span><span class="sxs-lookup"><span data-stu-id="81e1f-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="81e1f-268">Använd en Azure Hosted build-agent</span><span class="sxs-lookup"><span data-stu-id="81e1f-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="81e1f-269">Det är ett bekvämt alternativ att bygga och distribuera webbappar med hjälp av en värdbaserad Bygg agent i Azure-pipeliner.</span><span class="sxs-lookup"><span data-stu-id="81e1f-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="81e1f-270">Underhåll och uppgraderingar görs automatiskt av Microsoft Azure, vilket möjliggör en kontinuerlig och oavbruten utvecklings cykel.</span><span class="sxs-lookup"><span data-stu-id="81e1f-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="81e1f-271">Konfigurera processen för kontinuerlig distribution (CD)</span><span class="sxs-lookup"><span data-stu-id="81e1f-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="81e1f-272">Azure-pipeliner och Azure DevOps Services erbjuder en mycket konfigurerbar och hanterbar pipeline för versioner till flera miljöer som utveckling, mellanlagring, kvalitets säkring (frågor och svar) och produktion.</span><span class="sxs-lookup"><span data-stu-id="81e1f-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="81e1f-273">Den här processen kan omfatta krav på godkännanden i vissa faser i appens livs cykel.</span><span class="sxs-lookup"><span data-stu-id="81e1f-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="81e1f-274">Skapa versions definition</span><span class="sxs-lookup"><span data-stu-id="81e1f-274">Create release definition</span></span>

<span data-ttu-id="81e1f-275">Att skapa en versions definition är det sista steget i bygg processen för appar.</span><span class="sxs-lookup"><span data-stu-id="81e1f-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="81e1f-276">Denna versions definition används för att skapa en version och distribuera en version.</span><span class="sxs-lookup"><span data-stu-id="81e1f-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="81e1f-277">Logga in på Azure-pipelines och gå till **version och lansering** för projektet.</span><span class="sxs-lookup"><span data-stu-id="81e1f-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="81e1f-278">På fliken **utgåvor** väljer du **[+]** och väljer sedan **skapa versions definition**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="81e1f-279">Välj **Azure App service distribution**på **Välj en mall**och välj sedan **Använd**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="81e1f-280">Välj Azure Cloud build-appen från **källan (build definition)** på **Lägg till artefakt**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="81e1f-281">På fliken **pipelines** väljer du länken **1 fas**, **1 aktivitet** , för att **Visa miljö aktiviteter**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="81e1f-282">På fliken **uppgifter** anger du Azure som **miljö namn** och väljer AzureCloud Traders – Web EP från listan Azure- **prenumeration** .</span><span class="sxs-lookup"><span data-stu-id="81e1f-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="81e1f-283">Ange **namnet på Azure App Service**, som finns `northwindtraders` i nästa skärmdump.</span><span class="sxs-lookup"><span data-stu-id="81e1f-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="81e1f-284">För agent fasen väljer du **VÄRDBASERAD VS2017** i listan **agent kö** .</span><span class="sxs-lookup"><span data-stu-id="81e1f-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="81e1f-285">I **distribuera Azure App Service**väljer du ett giltigt **paket eller** en giltig mapp för miljön.</span><span class="sxs-lookup"><span data-stu-id="81e1f-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="81e1f-286">I **Välj fil eller mapp**väljer du **OK** till **plats**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="81e1f-287">Spara alla ändringar och gå tillbaka till **pipelinen**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="81e1f-288">På fliken **pipelines** väljer du **Lägg till artefakt**och väljer **NorthwindCloud Traders-fartyget** från käll listan **(build definition)** .</span><span class="sxs-lookup"><span data-stu-id="81e1f-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="81e1f-289">Lägg till en annan miljö på **Välj en mall**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="81e1f-290">Välj **Azure App service distribution** och välj sedan **Använd**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="81e1f-291">Ange `Azure Stack Hub` som **miljö namn**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="81e1f-292">På fliken **aktiviteter** , leta upp och välj Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81e1f-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="81e1f-293">I listan **Azure-prenumeration** väljer du **AzureStack Traders – fartyg EP** för Azure Stack Hub-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="81e1f-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="81e1f-294">Ange Azure Stack Hub-webbappens namn som **App Service-** namn.</span><span class="sxs-lookup"><span data-stu-id="81e1f-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="81e1f-295">Under **agent val**väljer du **AzureStack-b Douglas FIR** från listan **agent kö** .</span><span class="sxs-lookup"><span data-stu-id="81e1f-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="81e1f-296">För **distribuera Azure App Service**väljer du ett giltigt **paket eller** en giltig mapp för miljön.</span><span class="sxs-lookup"><span data-stu-id="81e1f-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="81e1f-297">På sidan **Välj fil eller mapp**väljer du **OK** för mappens **plats**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="81e1f-298">På fliken **variabel** letar du upp variabeln med namnet `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="81e1f-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="81e1f-299">Ange värdet **True**för variabeln och ange dess omfång till **Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="81e1f-300">På fliken **pipelines** väljer du ikonen för **kontinuerlig distribution av utlösare** för NorthwindCloud Traders – webb artefakt och ställer in den **kontinuerliga distributions utlösaren** på **aktive rad**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="81e1f-301">Gör samma sak för **NorthwindCloud Traders-fartygets** artefakt.</span><span class="sxs-lookup"><span data-stu-id="81e1f-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="81e1f-302">För Azure Stack Hub-miljö väljer du ikonen för **för distributions villkor** ange utlösaren till **efter versionen**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="81e1f-303">Spara alla ändringar.</span><span class="sxs-lookup"><span data-stu-id="81e1f-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="81e1f-304">Vissa inställningar för versions aktiviteter definieras automatiskt som [miljövariabler](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) när du skapar en versions definition från en mall.</span><span class="sxs-lookup"><span data-stu-id="81e1f-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="81e1f-305">De här inställningarna kan inte ändras i aktivitets inställningarna men kan ändras i de överordnade miljö objekten.</span><span class="sxs-lookup"><span data-stu-id="81e1f-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="81e1f-306">Skapa en version</span><span class="sxs-lookup"><span data-stu-id="81e1f-306">Create a release</span></span>

1. <span data-ttu-id="81e1f-307">På fliken **pipeline** öppnar du listan **version** och väljer **Skapa version**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="81e1f-308">Ange en beskrivning av versionen, kontrol lera att rätt artefakter har valts och välj sedan **skapa**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="81e1f-309">Efter en liten stund visas en banderoll som anger att den nya versionen skapades och att versions namnet visas som en länk.</span><span class="sxs-lookup"><span data-stu-id="81e1f-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="81e1f-310">Välj länken för att se sammanfattnings sidan för utgåvor.</span><span class="sxs-lookup"><span data-stu-id="81e1f-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="81e1f-311">Sidan versions Sammanfattning visar information om versionen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="81e1f-312">I följande skärm bild för "Release-2" visar avsnittet **miljöer** **distributions statusen** för Azure som "pågår" och statusen för Azure Stack Hub är "lyckades".</span><span class="sxs-lookup"><span data-stu-id="81e1f-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="81e1f-313">När distributions statusen för Azure-miljön ändras till "lyckades" visas en banderoll som anger att versionen är klar för godkännande.</span><span class="sxs-lookup"><span data-stu-id="81e1f-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="81e1f-314">När en distribution väntar eller har misslyckats visas en blå **(i)** informations ikon.</span><span class="sxs-lookup"><span data-stu-id="81e1f-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="81e1f-315">Hovra över ikonen för att se ett popup-fönster som innehåller orsaken till fördröjningen eller haveriet.</span><span class="sxs-lookup"><span data-stu-id="81e1f-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="81e1f-316">Andra vyer, till exempel listan över versioner, visar också en ikon som indikerar att godkännande väntar.</span><span class="sxs-lookup"><span data-stu-id="81e1f-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="81e1f-317">Popup-fönstret för den här ikonen visar miljö namnet och mer information om distributionen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="81e1f-318">Det är enkelt för en administratör att se det övergripande förloppet för versioner och se vilka versioner som väntar på godkännande.</span><span class="sxs-lookup"><span data-stu-id="81e1f-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="81e1f-319">Övervaka och spåra distributioner</span><span class="sxs-lookup"><span data-stu-id="81e1f-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="81e1f-320">På sammanfattnings sidan för **Release 2** väljer du **loggar**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="81e1f-321">Under en distribution visar den här sidan Live-loggen från agenten.</span><span class="sxs-lookup"><span data-stu-id="81e1f-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="81e1f-322">Den vänstra rutan visar status för varje åtgärd i distributionen för varje miljö.</span><span class="sxs-lookup"><span data-stu-id="81e1f-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="81e1f-323">Välj person ikonen i kolumnen **åtgärd** för ett godkännande före distribution eller efter distribution för att se vem som har godkänt (eller avvisat) distributionen och meddelandet de tillhandahöll.</span><span class="sxs-lookup"><span data-stu-id="81e1f-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="81e1f-324">När distributionen är klar visas hela logg filen i den högra rutan.</span><span class="sxs-lookup"><span data-stu-id="81e1f-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="81e1f-325">Välj något av **stegen** i det vänstra fönstret för att se logg filen för ett enda steg, t. ex. **initiera jobb**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="81e1f-326">Möjligheten att se enskilda loggar gör det lättare att spåra och felsöka delar av den övergripande distributionen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="81e1f-327">**Spara** logg filen för ett steg eller **Ladda ned alla loggar som zip**.</span><span class="sxs-lookup"><span data-stu-id="81e1f-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="81e1f-328">Öppna fliken **Sammanfattning** om du vill se allmän information om versionen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="81e1f-329">Den här vyn visar information om bygget, de miljöer som den distribuerades till, distributions status och annan information om versionen.</span><span class="sxs-lookup"><span data-stu-id="81e1f-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="81e1f-330">Välj en miljö länk (**Azure** eller **Azure Stack Hub**) om du vill se information om befintliga och väntande distributioner till en speciell miljö.</span><span class="sxs-lookup"><span data-stu-id="81e1f-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="81e1f-331">Använd dessa vyer som ett snabbt sätt att kontrol lera att samma version har distribuerats i båda miljöerna.</span><span class="sxs-lookup"><span data-stu-id="81e1f-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="81e1f-332">Öppna den **distribuerade produktions programmet** i en webbläsare.</span><span class="sxs-lookup"><span data-stu-id="81e1f-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="81e1f-333">För webbplatsen för Azure App tjänster öppnar du till exempel URL: en `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="81e1f-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="81e1f-334">Integrering av Azure och Azure Stack Hub ger en skalbar lösning för kors molnet</span><span class="sxs-lookup"><span data-stu-id="81e1f-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="81e1f-335">En flexibel och robust tjänst för flera moln ger data säkerhet, säkerhets kopiering och redundans, konsekvent och snabb tillgänglighet, skalbar lagring och distribution och geo-kompatibel routning.</span><span class="sxs-lookup"><span data-stu-id="81e1f-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="81e1f-336">Den manuellt utlösta processen garanterar en tillförlitlig och effektiv belastnings växling mellan värdbaserade webbappar och omedelbar tillgänglighet för viktiga data.</span><span class="sxs-lookup"><span data-stu-id="81e1f-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="81e1f-337">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="81e1f-337">Next steps</span></span>

- <span data-ttu-id="81e1f-338">Mer information om moln mönster i Azure finns i [design mönster för molnet](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="81e1f-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
