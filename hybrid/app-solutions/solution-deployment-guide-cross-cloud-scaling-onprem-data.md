---
title: Distribuera en hybrid app med lokala data som skalar över molnet
description: Lär dig hur du distribuerar en app som använder lokala data och skalar över molnet med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911859"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="1ae08-103">Distribuera en hybrid app med lokala data som skalar över molnet</span><span class="sxs-lookup"><span data-stu-id="1ae08-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="1ae08-104">Den här lösnings guiden visar hur du distribuerar en hybrid app som omfattar både Azure och Azure Stack hubb och använder en enda lokal data källa.</span><span class="sxs-lookup"><span data-stu-id="1ae08-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="1ae08-105">Genom att använda en hybrid moln lösning kan du kombinera kompatibiliteten för ett privat moln med skalbarheten för det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="1ae08-106">Utvecklarna kan också dra nytta av Microsoft Developer-eko systemet och tillämpa deras kunskaper i molnet och i lokala miljöer.</span><span class="sxs-lookup"><span data-stu-id="1ae08-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="1ae08-107">Översikt och antaganden</span><span class="sxs-lookup"><span data-stu-id="1ae08-107">Overview and assumptions</span></span>

<span data-ttu-id="1ae08-108">Följ den här självstudien för att skapa ett arbets flöde som gör det möjligt för utvecklare att distribuera en identisk webbapp till ett offentligt moln och ett privat moln.</span><span class="sxs-lookup"><span data-stu-id="1ae08-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="1ae08-109">Den här appen kan komma åt ett icke-Internet-dirigerbart nätverk som finns på det privata molnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="1ae08-110">Dessa webbappar övervakas och när det finns en insamling i trafik, ändrar ett program DNS-posterna för att dirigera trafik till det offentliga molnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="1ae08-111">När trafiken sjunker till nivån före insamling dirigeras trafiken tillbaka till det privata molnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="1ae08-112">Den här självstudien omfattar följande uppgifter:</span><span class="sxs-lookup"><span data-stu-id="1ae08-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="1ae08-113">Distribuera en hybrid-ansluten SQL Server databas server.</span><span class="sxs-lookup"><span data-stu-id="1ae08-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="1ae08-114">Anslut en webbapp i Global Azure till ett hybrid nätverk.</span><span class="sxs-lookup"><span data-stu-id="1ae08-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="1ae08-115">Konfigurera DNS för skalning över molnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="1ae08-116">Konfigurera SSL-certifikat för skalning över molnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="1ae08-117">Konfigurera och distribuera webbappen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="1ae08-118">Skapa en Traffic Manager profil och konfigurera den för skalning över molnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="1ae08-119">Konfigurera Application Insights övervakning och aviseringar för ökad trafik.</span><span class="sxs-lookup"><span data-stu-id="1ae08-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="1ae08-120">Konfigurera automatisk trafik växling mellan Global Azure och Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="1ae08-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="1ae08-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="1ae08-122">Microsoft Azure Stack Hub är ett tillägg till Azure.</span><span class="sxs-lookup"><span data-stu-id="1ae08-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="1ae08-123">Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.</span><span class="sxs-lookup"><span data-stu-id="1ae08-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="1ae08-124">Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program.</span><span class="sxs-lookup"><span data-stu-id="1ae08-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="1ae08-125">Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.</span><span class="sxs-lookup"><span data-stu-id="1ae08-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="1ae08-126">Antaganden</span><span class="sxs-lookup"><span data-stu-id="1ae08-126">Assumptions</span></span>

<span data-ttu-id="1ae08-127">Den här självstudien förutsätter att du har grundläggande kunskaper om Global Azure och Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="1ae08-128">Om du vill veta mer innan du startar självstudien kan du läsa följande artiklar:</span><span class="sxs-lookup"><span data-stu-id="1ae08-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="1ae08-129">Introduktion till Azure</span><span class="sxs-lookup"><span data-stu-id="1ae08-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="1ae08-130">Viktiga begrepp för Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="1ae08-131">Den här kursen förutsätter också att du har en Azure-prenumeration.</span><span class="sxs-lookup"><span data-stu-id="1ae08-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="1ae08-132">Om du inte har någon prenumeration kan du [skapa ett kostnads fritt konto](https://azure.microsoft.com/free/) innan du börjar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="1ae08-133">Krav</span><span class="sxs-lookup"><span data-stu-id="1ae08-133">Prerequisites</span></span>

<span data-ttu-id="1ae08-134">Innan du startar den här lösningen ser du till att du uppfyller följande krav:</span><span class="sxs-lookup"><span data-stu-id="1ae08-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="1ae08-135">En Azure Stack Development Kit (ASDK) eller en prenumeration på ett integrerat system för Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="1ae08-136">Om du vill distribuera ASDK följer du anvisningarna i [distribuera ASDK med installations programmet](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="1ae08-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="1ae08-137">Installationen av Azure Stack Hub måste ha följande installerat:</span><span class="sxs-lookup"><span data-stu-id="1ae08-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="1ae08-138">Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="1ae08-138">The Azure App Service.</span></span> <span data-ttu-id="1ae08-139">Arbeta med din Azure Stack Hub-operatör för att distribuera och konfigurera Azure App Service i din miljö.</span><span class="sxs-lookup"><span data-stu-id="1ae08-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="1ae08-140">Den här självstudien kräver att App Service har minst en (1) tillgänglig dedikerad arbets roll.</span><span class="sxs-lookup"><span data-stu-id="1ae08-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="1ae08-141">En Windows Server 2016-avbildning.</span><span class="sxs-lookup"><span data-stu-id="1ae08-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="1ae08-142">En Windows Server 2016 med en Microsoft SQL Server avbildning.</span><span class="sxs-lookup"><span data-stu-id="1ae08-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="1ae08-143">Lämpliga planer och erbjudanden.</span><span class="sxs-lookup"><span data-stu-id="1ae08-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="1ae08-144">Ett domän namn för din webbapp.</span><span class="sxs-lookup"><span data-stu-id="1ae08-144">A domain name for your web app.</span></span> <span data-ttu-id="1ae08-145">Om du inte har ett domän namn kan du köpa ett från en domän leverantör som GoDaddy, BlueHost och InMotion.</span><span class="sxs-lookup"><span data-stu-id="1ae08-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="1ae08-146">Ett SSL-certifikat för din domän från en betrodd certifikat utfärdare som LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="1ae08-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="1ae08-147">En webbapp som kommunicerar med en SQL Server-databas och stöder Application Insights.</span><span class="sxs-lookup"><span data-stu-id="1ae08-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="1ae08-148">Du kan hämta exempel appen [dotnetcore-SQLDB-självstudie](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) från GitHub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="1ae08-149">Ett hybrid nätverk mellan ett virtuellt Azure-nätverk och Azure Stack hubb virtuellt nätverk.</span><span class="sxs-lookup"><span data-stu-id="1ae08-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="1ae08-150">Detaljerade anvisningar finns i [Konfigurera hybrid moln anslutning med Azure och Azure Stack hubb](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="1ae08-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="1ae08-151">En pipeline för kontinuerlig integrering/en kontinuerlig distribution (CI/CD) med en privat build-agent på Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="1ae08-152">Detaljerade anvisningar finns i [Konfigurera hybrid moln identitet med Azure och Azure Stack Hub-appar](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="1ae08-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="1ae08-153">Distribuera en hybrid-ansluten SQL Server databas server</span><span class="sxs-lookup"><span data-stu-id="1ae08-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="1ae08-154">Logga in på användar portalen för Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="1ae08-155">Välj **Marketplace**på **instrument panelen**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="1ae08-157">Välj **Compute**i **Marketplace**och välj sedan **mer**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="1ae08-158">Under **mer**väljer du den **kostnads fria SQL Server licensen: SQL Server 2017 Developer på Windows Server** -avbildning.</span><span class="sxs-lookup"><span data-stu-id="1ae08-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Välj en avbildning av en virtuell dator i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="1ae08-160">På **licens för gratis SQL Server: SQL Server 2017-utvecklare på Windows Server**, Välj **skapa**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="1ae08-161">I **grundläggande > du konfigurera grundläggande inställningar**, ange ett **namn** för den virtuella datorn (VM), ett **användar namn** för SQL Server SA och ett **lösen ord** för säkerhets associationen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="1ae08-162">I list rutan **prenumeration** väljer du den prenumeration som du distribuerar till.</span><span class="sxs-lookup"><span data-stu-id="1ae08-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="1ae08-163">För **resurs grupp**använder du **Välj befintlig** och sätter den virtuella datorn i samma resurs grupp som din Azure Stack hubb-webbapp.</span><span class="sxs-lookup"><span data-stu-id="1ae08-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Konfigurera grundläggande inställningar för virtuell dator i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="1ae08-165">Välj en storlek för den virtuella datorn under **storlek**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="1ae08-166">I den här självstudien rekommenderar vi A2_Standard eller en DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="1ae08-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="1ae08-167">Under **inställningar > konfigurera valfria funktioner**, konfigurerar du följande inställningar:</span><span class="sxs-lookup"><span data-stu-id="1ae08-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="1ae08-168">**Lagrings konto**: skapa ett nytt konto om du behöver ett.</span><span class="sxs-lookup"><span data-stu-id="1ae08-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="1ae08-169">**Virtuellt nätverk**:</span><span class="sxs-lookup"><span data-stu-id="1ae08-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="1ae08-170">Kontrol lera att din SQL Server VM har distribuerats på samma virtuella nätverk som VPN-gatewayerna.</span><span class="sxs-lookup"><span data-stu-id="1ae08-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="1ae08-171">**Offentlig IP-adress**: Använd standardinställningarna.</span><span class="sxs-lookup"><span data-stu-id="1ae08-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="1ae08-172">**Nätverks säkerhets grupp**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="1ae08-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="1ae08-173">Skapa en ny NSG.</span><span class="sxs-lookup"><span data-stu-id="1ae08-173">Create a new NSG.</span></span>
   - <span data-ttu-id="1ae08-174">**Tillägg och övervakning**: Behåll standardinställningarna.</span><span class="sxs-lookup"><span data-stu-id="1ae08-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="1ae08-175">**Lagrings konto för diagnostik**: skapa ett nytt konto om du behöver ett.</span><span class="sxs-lookup"><span data-stu-id="1ae08-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="1ae08-176">Välj **OK** för att spara konfigurationen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-176">Select **OK** to save your configuration.</span></span>

     ![Konfigurera valfria VM-funktioner i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="1ae08-178">Under **SQL Server inställningar**konfigurerar du följande inställningar:</span><span class="sxs-lookup"><span data-stu-id="1ae08-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="1ae08-179">Välj **offentlig (Internet)** för **SQL-anslutning**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="1ae08-180">Behåll standardvärdet **1433**för **port**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="1ae08-181">Välj **Aktivera**för **SQL-autentisering**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="1ae08-182">När du aktiverar SQL-autentisering ska den automatiskt fyllas i med "SQLAdmin"-informationen som du konfigurerade i **grunderna**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="1ae08-183">Behåll standardvärdena för resten av inställningarna.</span><span class="sxs-lookup"><span data-stu-id="1ae08-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="1ae08-184">Välj **OK**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-184">Select **OK**.</span></span>

     ![Konfigurera SQL Server inställningar i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="1ae08-186">Vid **Sammanfattning**granskar du VM-konfigurationen och väljer sedan **OK** för att starta distributionen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Konfigurations Sammanfattning i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="1ae08-188">Det tar lite tid att skapa den nya virtuella datorn.</span><span class="sxs-lookup"><span data-stu-id="1ae08-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="1ae08-189">Du kan visa STATUSEN för dina virtuella datorer på **virtuella datorer**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Status för virtuella datorer i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="1ae08-191">Skapa webb program i Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1ae08-192">Azure App Service fören klar att köra och hantera en webbapp.</span><span class="sxs-lookup"><span data-stu-id="1ae08-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="1ae08-193">Eftersom Azure Stack Hub är konsekvent med Azure kan App Service köras i båda miljöerna.</span><span class="sxs-lookup"><span data-stu-id="1ae08-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="1ae08-194">Du använder App Service som värd för din app.</span><span class="sxs-lookup"><span data-stu-id="1ae08-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="1ae08-195">Skapa webb program</span><span class="sxs-lookup"><span data-stu-id="1ae08-195">Create web apps</span></span>

1. <span data-ttu-id="1ae08-196">Skapa en webbapp i Azure genom att följa anvisningarna i [hantera ett App Service plan i Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="1ae08-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="1ae08-197">Se till att du sätter webbappen i samma prenumeration och resurs grupp som ditt hybrid nätverk.</span><span class="sxs-lookup"><span data-stu-id="1ae08-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="1ae08-198">Upprepa föregående steg (1) i Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="1ae08-199">Lägg till väg för Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="1ae08-200">App Service på Azure Stack Hub måste dirigeras från det offentliga Internet för att användarna ska kunna komma åt din app.</span><span class="sxs-lookup"><span data-stu-id="1ae08-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="1ae08-201">Om din Azure Stack hubb är tillgänglig från Internet noterar du den offentliga IP-adressen eller URL: en för den Azure Stack Hub-webbappen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="1ae08-202">Om du använder en ASDK kan du [Konfigurera en statisk NAT-mappning](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) för att exponera App Service utanför den virtuella miljön.</span><span class="sxs-lookup"><span data-stu-id="1ae08-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="1ae08-203">Ansluta en webbapp i Azure till ett hybrid nätverk</span><span class="sxs-lookup"><span data-stu-id="1ae08-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="1ae08-204">För att tillhandahålla anslutning mellan webb klient delen i Azure och SQL Server databasen i Azure Stack Hub måste webbappen vara ansluten till hybrid nätverket mellan Azure och Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="1ae08-205">Om du vill aktivera anslutning måste du:</span><span class="sxs-lookup"><span data-stu-id="1ae08-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="1ae08-206">Konfigurera punkt-till-plats-anslutning.</span><span class="sxs-lookup"><span data-stu-id="1ae08-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="1ae08-207">Konfigurera webbappen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-207">Configure the web app.</span></span>
- <span data-ttu-id="1ae08-208">Ändra den lokala Nätverksgatewayen i Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="1ae08-209">Konfigurera det virtuella Azure-nätverket för punkt-till-plats-anslutning</span><span class="sxs-lookup"><span data-stu-id="1ae08-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="1ae08-210">Den virtuella Nätverksgatewayen på Azure-sidan av hybrid nätverket måste tillåta punkt-till-plats-anslutningar för att kunna integreras med Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="1ae08-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="1ae08-211">Gå till sidan virtuell nätverksgateway i Azure.</span><span class="sxs-lookup"><span data-stu-id="1ae08-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="1ae08-212">Under **Inställningar**väljer **du punkt-till-plats-konfiguration**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Alternativet punkt-till-plats i Azure Virtual Network Gateway](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="1ae08-214">Välj **Konfigurera nu** för att konfigurera punkt-till-plats.</span><span class="sxs-lookup"><span data-stu-id="1ae08-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Starta punkt-till-plats-konfiguration i Azure virtuell nätverksgateway](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="1ae08-216">På sidan **punkt-till-plats** -konfiguration anger du det privata IP-adressintervall som du vill använda i **adresspoolen**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="1ae08-217">Kontrol lera att det intervall som du anger inte överlappar något av adress intervallen som redan används av undernät i de globala Azure-eller Azure Stack Hub-komponenterna i hybrid nätverket.</span><span class="sxs-lookup"><span data-stu-id="1ae08-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="1ae08-218">Avmarkera **IKEV2 VPN**under **tunnel typ**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="1ae08-219">Välj **Spara** för att slutföra konfigurationen av punkt-till-plats.</span><span class="sxs-lookup"><span data-stu-id="1ae08-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Punkt-till-plats-inställningar i Azure virtuell nätverksgateway](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="1ae08-221">Integrera Azure App Service-appen med hybrid nätverket</span><span class="sxs-lookup"><span data-stu-id="1ae08-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="1ae08-222">Om du vill ansluta appen till Azure VNet följer du anvisningarna i [Gateway krävs VNet-integrering](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="1ae08-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="1ae08-223">Gå till **Inställningar** för App Service plan som är värd för webbappen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="1ae08-224">I **Inställningar**väljer du **nätverk**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-224">In **Settings**, select **Networking**.</span></span>

    ![Konfigurera nätverk för App Service plan](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="1ae08-226">I **VNet-integration**väljer **du klicka här för att hantera**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Hantera VNET-integrering för App Service plan](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="1ae08-228">Välj det virtuella nätverk som du vill konfigurera.</span><span class="sxs-lookup"><span data-stu-id="1ae08-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="1ae08-229">Under **IP-adresser dirigeras till VNet**, anger du IP-adressintervallet för det virtuella Azure-nätverket, Azure Stack hubb-VNet och punkt-till-plats-adress utrymmen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="1ae08-230">Välj **Spara** för att validera och spara inställningarna.</span><span class="sxs-lookup"><span data-stu-id="1ae08-230">Select **Save** to validate and save these settings.</span></span>

    ![IP-adressintervall som ska vidarebefordras i Virtual Network-integrering](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="1ae08-232">Mer information om hur App Service integreras med Azure virtuella nätverk finns i [integrera din app med en Azure-Virtual Network](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="1ae08-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="1ae08-233">Konfigurera det virtuella nätverket för Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="1ae08-234">Den lokala Nätverksgatewayen i Azure Stack hubbens virtuella nätverk måste konfigureras för att dirigera trafik från App Service punkt-till-plats-adress intervall.</span><span class="sxs-lookup"><span data-stu-id="1ae08-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="1ae08-235">I Azure Stack hubb går du till **lokal nätverksgateway**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="1ae08-236">Under **Inställningar** väljer du **Konfiguration**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-236">Under **Settings**, select **Configuration**.</span></span>

    ![Konfigurations alternativ för gateway i Azure Stack hubb lokal nätverksgateway](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="1ae08-238">I **adress utrymme**anger du punkt-till-plats-adressintervallet för den virtuella Nätverksgatewayen i Azure.</span><span class="sxs-lookup"><span data-stu-id="1ae08-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Adress utrymme för punkt-till-plats i Azure Stack hubb lokal nätverksgateway](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="1ae08-240">Välj **Spara** för att validera och spara konfigurationen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="1ae08-241">Konfigurera DNS för skalning över molnet</span><span class="sxs-lookup"><span data-stu-id="1ae08-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="1ae08-242">Genom att konfigurera DNS för appar över molnet korrekt kan användare komma åt de globala Azure-och Azure Stack Hub-instanserna av din webbapp.</span><span class="sxs-lookup"><span data-stu-id="1ae08-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="1ae08-243">DNS-konfigurationen för den här självstudien låter också Azure Traffic Manager dirigera trafik när belastningen ökar eller minskar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="1ae08-244">I den här självstudien används Azure DNS för att hantera DNS eftersom App Service domäner inte fungerar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="1ae08-245">Skapa under domäner</span><span class="sxs-lookup"><span data-stu-id="1ae08-245">Create subdomains</span></span>

<span data-ttu-id="1ae08-246">Eftersom Traffic Manager är beroende av DNS-CNAME, krävs en under domän för att dirigera trafik till slut punkter korrekt.</span><span class="sxs-lookup"><span data-stu-id="1ae08-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="1ae08-247">Mer information om DNS-poster och domän mappning finns i [Mappa domäner med Traffic Manager](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="1ae08-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="1ae08-248">För Azure-slutpunkten skapar du en under domän som användare kan använda för att få åtkomst till din webbapp.</span><span class="sxs-lookup"><span data-stu-id="1ae08-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="1ae08-249">I den här självstudien kan använda **app.Northwind.com**, men du bör anpassa det här värdet baserat på din egen domän.</span><span class="sxs-lookup"><span data-stu-id="1ae08-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="1ae08-250">Du måste också skapa en under domän med en A-post för Azure Stack Hub-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="1ae08-251">Du kan använda **azurestack.Northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="1ae08-252">Konfigurera en anpassad domän i Azure</span><span class="sxs-lookup"><span data-stu-id="1ae08-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="1ae08-253">Lägg till **app.Northwind.com** -värdnamnet i Azure-webbappen genom att [Mappa en CNAME till Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="1ae08-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="1ae08-254">Konfigurera anpassade domäner i Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="1ae08-255">Lägg till **azurestack.Northwind.com** -värdnamnet i Azure Stack Hub-webbappen genom [att mappa en A-post till Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="1ae08-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="1ae08-256">Använd IP-adressen Internet-dirigerbart för App Service-appen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="1ae08-257">Lägg till **app.Northwind.com** -värdnamnet i Azure Stack Hub-webbappen genom att [Mappa en CNAME till Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="1ae08-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="1ae08-258">Använd det värdnamn som du konfigurerade i föregående steg (1) som mål för CNAME.</span><span class="sxs-lookup"><span data-stu-id="1ae08-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="1ae08-259">Konfigurera SSL-certifikat för skalning över molnet</span><span class="sxs-lookup"><span data-stu-id="1ae08-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="1ae08-260">Det är viktigt att se till att känsliga data som samlas in av din webbapp är säkra vid överföring till och när de lagras i SQL-databasen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="1ae08-261">Du konfigurerar dina Azure-och Azure Stack Hub-webbappar så att de använder SSL-certifikat för all inkommande trafik.</span><span class="sxs-lookup"><span data-stu-id="1ae08-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="1ae08-262">Lägg till SSL till Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1ae08-263">Så här lägger du till SSL i Azure:</span><span class="sxs-lookup"><span data-stu-id="1ae08-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="1ae08-264">Kontrol lera att SSL-certifikatet som du får är giltigt för den under domän som du skapade.</span><span class="sxs-lookup"><span data-stu-id="1ae08-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="1ae08-265">(Det är OK att använda certifikat med jokertecken.)</span><span class="sxs-lookup"><span data-stu-id="1ae08-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="1ae08-266">I Azure följer du anvisningarna i avsnittet **förbereda din webbapp** och **binder ditt SSL-certifikat** i avsnittet [BIND ett befintligt anpassat ssl-certifikat till Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) -artikeln.</span><span class="sxs-lookup"><span data-stu-id="1ae08-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="1ae08-267">Välj **SNI-baserad SSL** som **SSL-typ**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="1ae08-268">Omdirigera all trafik till HTTPS-porten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="1ae08-269">Följ instruktionerna i avsnittet om att **använda https** i avsnittet [BIND ett befintligt anpassat SSL-certifikat till Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="1ae08-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="1ae08-270">Så här lägger du till SSL till Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="1ae08-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="1ae08-271">Upprepa steg 1-3 som du använde för Azure.</span><span class="sxs-lookup"><span data-stu-id="1ae08-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="1ae08-272">Konfigurera och distribuera webbappen</span><span class="sxs-lookup"><span data-stu-id="1ae08-272">Configure and deploy the web app</span></span>

<span data-ttu-id="1ae08-273">Du konfigurerar appens kod för att rapportera telemetri till rätt Application Insights-instans och konfigurera webbapparna med rätt anslutnings strängar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="1ae08-274">Mer information om Application Insights finns i [Vad är Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="1ae08-274">To learn more about Application Insights, see [What is Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="1ae08-275">Lägg till Application Insights</span><span class="sxs-lookup"><span data-stu-id="1ae08-275">Add Application Insights</span></span>

1. <span data-ttu-id="1ae08-276">Öppna din webbapp i Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="1ae08-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="1ae08-277">[Lägg till Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) i projektet för att överföra telemetri som Application Insights använder för att skapa aviseringar när webb trafiken ökar eller minskar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-277">[Add Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="1ae08-278">Konfigurera dynamiska anslutnings strängar</span><span class="sxs-lookup"><span data-stu-id="1ae08-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="1ae08-279">Varje instans av webbappen kommer att använda en annan metod för att ansluta till SQL-databasen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="1ae08-280">Appen i Azure använder den privata IP-adressen för SQL Server VM och appen i Azure Stack Hub använder den offentliga IP-adressen för SQL Server VM.</span><span class="sxs-lookup"><span data-stu-id="1ae08-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="1ae08-281">På ett integrerat Azure Stack Hub-system får den offentliga IP-adressen inte vara Internet-dirigerbart.</span><span class="sxs-lookup"><span data-stu-id="1ae08-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="1ae08-282">På en ASDK dirigeras inte den offentliga IP-adressen utanför ASDK.</span><span class="sxs-lookup"><span data-stu-id="1ae08-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="1ae08-283">Du kan använda App Service miljövariabler för att skicka en annan anslutnings sträng till varje instans av appen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="1ae08-284">Öppna appen i Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="1ae08-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="1ae08-285">Öppna Startup.cs och hitta följande kodblock:</span><span class="sxs-lookup"><span data-stu-id="1ae08-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="1ae08-286">Ersätt det tidigare kod blocket med följande kod, som använder en anslutnings sträng som definierats i *appsettings.jspå* filen:</span><span class="sxs-lookup"><span data-stu-id="1ae08-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="1ae08-287">Konfigurera inställningar för App Service app</span><span class="sxs-lookup"><span data-stu-id="1ae08-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="1ae08-288">Skapa anslutnings strängar för Azure och Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="1ae08-289">Strängarna måste vara desamma, förutom de IP-adresser som används.</span><span class="sxs-lookup"><span data-stu-id="1ae08-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="1ae08-290">I Azure och Azure Stack hubb lägger du till rätt anslutnings sträng [som en app-inställning](https://docs.microsoft.com/azure/app-service/web-sites-configure) i webbappen med `SQLCONNSTR\_` som ett prefix i namnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](https://docs.microsoft.com/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="1ae08-291">**Spara** inställningarna för webbappen och starta om appen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="1ae08-292">Aktivera automatisk skalning i Global Azure</span><span class="sxs-lookup"><span data-stu-id="1ae08-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="1ae08-293">När du skapar en webbapp i en App Services miljö börjar den med en instans.</span><span class="sxs-lookup"><span data-stu-id="1ae08-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="1ae08-294">Du kan automatiskt skala ut för att lägga till instanser för att tillhandahålla fler beräknings resurser för din app.</span><span class="sxs-lookup"><span data-stu-id="1ae08-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="1ae08-295">På samma sätt kan du automatiskt skala in och minska antalet instanser som appen behöver.</span><span class="sxs-lookup"><span data-stu-id="1ae08-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="1ae08-296">Du måste ha en App Service plan för att kunna konfigurera skala ut och skala in.</span><span class="sxs-lookup"><span data-stu-id="1ae08-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="1ae08-297">Om du inte har någon plan skapar du ett innan du påbörjar nästa steg.</span><span class="sxs-lookup"><span data-stu-id="1ae08-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="1ae08-298">Aktivera automatisk utskalning</span><span class="sxs-lookup"><span data-stu-id="1ae08-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="1ae08-299">I Azure, letar du upp App Service plan för de platser som du vill skala ut och väljer sedan **skala ut (App Service plan)**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Skala ut Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="1ae08-301">Välj **Aktivera autoskalning**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-301">Select **Enable autoscale**.</span></span>

    ![Aktivera autoskalning i Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="1ae08-303">Ange ett namn för den **automatiska skalnings inställningen**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="1ae08-304">För **standard** regeln för automatisk skalning väljer du **skala baserat på ett mått**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="1ae08-305">Ange att **instans gränserna** ska vara **minst: 1**, **Max: 10**och **standard: 1**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Konfigurera autoskalning i Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="1ae08-307">Välj **+ Lägg till en regel**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="1ae08-308">I **mått källa**väljer du **aktuell resurs**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="1ae08-309">Använd följande kriterier och åtgärder för regeln.</span><span class="sxs-lookup"><span data-stu-id="1ae08-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="1ae08-310">Kriterie</span><span class="sxs-lookup"><span data-stu-id="1ae08-310">Criteria</span></span>

1. <span data-ttu-id="1ae08-311">Under **tids agg regering väljer du** **Average**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="1ae08-312">Under **Metric Name**väljer du **processor procent**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="1ae08-313">Under **operatör**väljer du **större än**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="1ae08-314">Ange **tröskelvärdet** till **50**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="1ae08-315">Ange **varaktigheten** till **10**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="1ae08-316">Action</span><span class="sxs-lookup"><span data-stu-id="1ae08-316">Action</span></span>

1. <span data-ttu-id="1ae08-317">Under **åtgärd**väljer du **öka antalet efter**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="1ae08-318">Ange **antalet instanser** till **2**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="1ae08-319">Ange **nedkylning** till **5**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="1ae08-320">Välj **Lägg till**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-320">Select **Add**.</span></span>

5. <span data-ttu-id="1ae08-321">Välj **+ Lägg till en regel**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="1ae08-322">I **mått källa**väljer du **aktuell resurs.**</span><span class="sxs-lookup"><span data-stu-id="1ae08-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="1ae08-323">Den aktuella resursen kommer att innehålla App Service plan namn/GUID och list rutorna **resurs typ** och **resurs** är inte tillgängliga.</span><span class="sxs-lookup"><span data-stu-id="1ae08-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="1ae08-324">Aktivera automatisk skalning i</span><span class="sxs-lookup"><span data-stu-id="1ae08-324">Enable automatic scale in</span></span>

<span data-ttu-id="1ae08-325">När trafiken minskar kan Azure-webbappen automatiskt minska antalet aktiva instanser för att minska kostnaderna.</span><span class="sxs-lookup"><span data-stu-id="1ae08-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="1ae08-326">Den här åtgärden är mindre aggressiv än att skala ut och minimera påverkan på användare i appar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="1ae08-327">Gå till **standard** villkoret för skala ut och välj sedan **+ Lägg till en regel**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="1ae08-328">Använd följande kriterier och åtgärder för regeln.</span><span class="sxs-lookup"><span data-stu-id="1ae08-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="1ae08-329">Kriterie</span><span class="sxs-lookup"><span data-stu-id="1ae08-329">Criteria</span></span>

1. <span data-ttu-id="1ae08-330">Under **tids agg regering väljer du** **Average**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="1ae08-331">Under **Metric Name**väljer du **processor procent**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="1ae08-332">Under **operator**väljer du **mindre än**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="1ae08-333">Ange **tröskelvärdet** till **30**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="1ae08-334">Ange **varaktigheten** till **10**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="1ae08-335">Action</span><span class="sxs-lookup"><span data-stu-id="1ae08-335">Action</span></span>

1. <span data-ttu-id="1ae08-336">Under **åtgärd**väljer du **minska antalet efter**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="1ae08-337">Ange **antalet instanser** till **1**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="1ae08-338">Ange **nedkylning** till **5**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="1ae08-339">Välj **Lägg till**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="1ae08-340">Skapa en Traffic Manager profil och konfigurera skalning mellan moln</span><span class="sxs-lookup"><span data-stu-id="1ae08-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="1ae08-341">Skapa en Traffic Manager-profil i Azure och konfigurera sedan slut punkter för att aktivera skalning över molnet.</span><span class="sxs-lookup"><span data-stu-id="1ae08-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="1ae08-342">Skapa Traffic Manager profil</span><span class="sxs-lookup"><span data-stu-id="1ae08-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="1ae08-343">Välj **Skapa en resurs**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="1ae08-344">Välj **Nätverk**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-344">Select **Networking**.</span></span>
3. <span data-ttu-id="1ae08-345">Välj **Traffic Manager profil** och konfigurera följande inställningar:</span><span class="sxs-lookup"><span data-stu-id="1ae08-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="1ae08-346">I **namn**anger du ett namn för din profil.</span><span class="sxs-lookup"><span data-stu-id="1ae08-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="1ae08-347">Det här namnet **måste** vara unikt i trafficmanager.net-zonen och används för att skapa ett nytt DNS-namn (till exempel northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="1ae08-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="1ae08-348">För **routningsmetod**väljer du **viktat**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="1ae08-349">För **prenumeration**väljer du den prenumeration som du vill skapa profilen i.</span><span class="sxs-lookup"><span data-stu-id="1ae08-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="1ae08-350">I **resurs grupp**skapar du en ny resurs grupp för den här profilen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="1ae08-351">I **Resursgruppsplats** väljer du plats för resursgruppen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="1ae08-352">Den här inställningen refererar till platsen för resurs gruppen och påverkar inte den Traffic Manager profilen som distribueras globalt.</span><span class="sxs-lookup"><span data-stu-id="1ae08-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="1ae08-353">Välj **Skapa**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-353">Select **Create**.</span></span>

    ![Skapa Traffic Manager profil](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="1ae08-355">När den globala distributionen av Traffic Managers profilen är klar visas den i listan över resurser för resurs gruppen som du skapade den under.</span><span class="sxs-lookup"><span data-stu-id="1ae08-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="1ae08-356">Lägga till Traffic Manager-slutpunkter</span><span class="sxs-lookup"><span data-stu-id="1ae08-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="1ae08-357">Sök efter den Traffic Manager profil som du har skapat.</span><span class="sxs-lookup"><span data-stu-id="1ae08-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="1ae08-358">Om du har navigerat till resurs gruppen för profilen väljer du profilen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="1ae08-359">I **Traffic Manager profil**väljer du **slut punkter**under **Inställningar**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="1ae08-360">Välj **Lägg till**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-360">Select **Add**.</span></span>

4. <span data-ttu-id="1ae08-361">I **Lägg till slut punkt**använder du följande inställningar för Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="1ae08-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="1ae08-362">I **typ**väljer du **extern slut punkt**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="1ae08-363">Ange ett **namn** för slut punkten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="1ae08-364">För **fullständigt kvalificerat domän namn (FQDN) eller IP-** adress anger du den externa URL: en för din Azure Stack Hub-webbapp.</span><span class="sxs-lookup"><span data-stu-id="1ae08-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="1ae08-365">Behåll standardvärdet **1**för **vikt**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="1ae08-366">Den här vikten resulterar i all trafik som kommer till den här slut punkten om den är felfri.</span><span class="sxs-lookup"><span data-stu-id="1ae08-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="1ae08-367">Lämna **Lägg till som inaktiverat** avmarkerat.</span><span class="sxs-lookup"><span data-stu-id="1ae08-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="1ae08-368">Välj **OK** för att spara Azure Stack Hub-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="1ae08-369">Du konfigurerar Azure-slutpunkten härnäst.</span><span class="sxs-lookup"><span data-stu-id="1ae08-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="1ae08-370">På **Traffic Manager profil**väljer du **slut punkter**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="1ae08-371">Välj **+ Lägg till**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-371">Select **+Add**.</span></span>
3. <span data-ttu-id="1ae08-372">Använd följande inställningar för Azure på **Lägg till slut punkt**:</span><span class="sxs-lookup"><span data-stu-id="1ae08-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="1ae08-373">I **typ**väljer du **Azure-slutpunkt**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="1ae08-374">Ange ett **namn** för slut punkten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="1ae08-375">För **mål resurs typ**väljer du **App Service**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="1ae08-376">För **mål resurs**väljer du **Välj en app service** om du vill visa en lista över Web Apps i samma prenumeration.</span><span class="sxs-lookup"><span data-stu-id="1ae08-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="1ae08-377">I **Resurs** väljer du den apptjänst som du vill lägga till som den första slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="1ae08-378">I **vikt**väljer du **2**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="1ae08-379">Den här inställningen resulterar i all trafik som går till den här slut punkten om den primära slut punkten är ohälsosam eller om du har en regel/avisering som omdirigerar trafik när den utlöses.</span><span class="sxs-lookup"><span data-stu-id="1ae08-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="1ae08-380">Lämna **Lägg till som inaktiverat** avmarkerat.</span><span class="sxs-lookup"><span data-stu-id="1ae08-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="1ae08-381">Välj **OK** för att spara Azure-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="1ae08-382">När båda slut punkterna har kon figurer ATS visas de i **Traffic Manager profil** när du väljer **slut punkter**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="1ae08-383">Exemplet i följande skärm bild visar två slut punkter, med status-och konfigurations information för var och en.</span><span class="sxs-lookup"><span data-stu-id="1ae08-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Slut punkter i Traffic Managers profil](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="1ae08-385">Konfigurera Application Insights övervakning och avisering</span><span class="sxs-lookup"><span data-stu-id="1ae08-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="1ae08-386">Med Azure Application Insights kan du övervaka din app och skicka aviseringar baserat på de villkor som du konfigurerar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="1ae08-387">Några exempel är: appen är inte tillgänglig, har fel eller visar prestanda problem.</span><span class="sxs-lookup"><span data-stu-id="1ae08-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="1ae08-388">Du ska använda Application Insights mått för att skapa aviseringar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="1ae08-389">När dessa aviseringar utlöses växlar webbappens instans automatiskt från Azure Stack hubb till Azure för att skala ut, och sedan tillbaka till Azure Stack Hub för att skala in.</span><span class="sxs-lookup"><span data-stu-id="1ae08-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="1ae08-390">Skapa en avisering utifrån mått</span><span class="sxs-lookup"><span data-stu-id="1ae08-390">Create an alert from metrics</span></span>

<span data-ttu-id="1ae08-391">Gå till resurs gruppen för den här självstudien och välj sedan Application Insights-instansen som ska öppnas **Application Insights**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="1ae08-393">Du använder den här vyn för att skapa en skalbar avisering och en avisering om skalnings avisering.</span><span class="sxs-lookup"><span data-stu-id="1ae08-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="1ae08-394">Skapa en skalbar avisering</span><span class="sxs-lookup"><span data-stu-id="1ae08-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="1ae08-395">Under **Konfigurera**väljer du **aviseringar (klassisk)**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="1ae08-396">Välj **Lägg till mått varning (klassisk)**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="1ae08-397">Konfigurera följande inställningar i **Lägg till regel**:</span><span class="sxs-lookup"><span data-stu-id="1ae08-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="1ae08-398">I **namn**anger du **burst i Azure-molnet**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="1ae08-399">En **Beskrivning** är valfri.</span><span class="sxs-lookup"><span data-stu-id="1ae08-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="1ae08-400">Under **käll**  >  **avisering på**väljer du **mått**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="1ae08-401">Under **kriterier**väljer du din prenumeration, resurs grupp för din Traffic Manager profil och namnet på den Traffic Manager profilen för resursen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="1ae08-402">För **mått**väljer du **begär ande frekvens**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="1ae08-403">För **villkor**väljer du **större än**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="1ae08-404">För **tröskel**anger du **2**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="1ae08-405">För **period**väljer **du de senaste 5 minuterna**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="1ae08-406">Under **meddela via**:</span><span class="sxs-lookup"><span data-stu-id="1ae08-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="1ae08-407">Markera kryss rutan för **e-postägare, deltagare och läsare**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="1ae08-408">Ange din e-postadress för **ytterligare administratörs-e-post (er)**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="1ae08-409">I meny raden väljer du **Spara**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="1ae08-410">Skapa en skalnings avisering</span><span class="sxs-lookup"><span data-stu-id="1ae08-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="1ae08-411">Under **Konfigurera**väljer du **aviseringar (klassisk)**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="1ae08-412">Välj **Lägg till mått varning (klassisk)**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="1ae08-413">Konfigurera följande inställningar i **Lägg till regel**:</span><span class="sxs-lookup"><span data-stu-id="1ae08-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="1ae08-414">I **namn**anger du **skala tillbaka till Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="1ae08-415">En **Beskrivning** är valfri.</span><span class="sxs-lookup"><span data-stu-id="1ae08-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="1ae08-416">Under **käll**  >  **avisering på**väljer du **mått**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="1ae08-417">Under **kriterier**väljer du din prenumeration, resurs grupp för din Traffic Manager profil och namnet på den Traffic Manager profilen för resursen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="1ae08-418">För **mått**väljer du **begär ande frekvens**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="1ae08-419">För **villkor**väljer du **mindre än**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="1ae08-420">För **tröskel**anger du **2**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="1ae08-421">För **period**väljer **du de senaste 5 minuterna**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="1ae08-422">Under **meddela via**:</span><span class="sxs-lookup"><span data-stu-id="1ae08-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="1ae08-423">Markera kryss rutan för **e-postägare, deltagare och läsare**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="1ae08-424">Ange din e-postadress för **ytterligare administratörs-e-post (er)**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="1ae08-425">I meny raden väljer du **Spara**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="1ae08-426">Följande skärm bild visar aviseringarna för att skala ut och skala in.</span><span class="sxs-lookup"><span data-stu-id="1ae08-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights aviseringar (klassisk)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="1ae08-428">Omdirigera trafik mellan Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1ae08-429">Du kan konfigurera manuell eller automatisk växling av din webb program trafik mellan Azure och Azure Stack hubben.</span><span class="sxs-lookup"><span data-stu-id="1ae08-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="1ae08-430">Konfigurera manuell växling mellan Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1ae08-431">När webbplatsen når tröskelvärdena som du konfigurerar får du en avisering.</span><span class="sxs-lookup"><span data-stu-id="1ae08-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="1ae08-432">Använd följande steg för att manuellt omdirigera trafik till Azure.</span><span class="sxs-lookup"><span data-stu-id="1ae08-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="1ae08-433">I Azure Portal väljer du din Traffic Manager profil.</span><span class="sxs-lookup"><span data-stu-id="1ae08-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Traffic Manager slut punkter i Azure Portal](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="1ae08-435">Välj **slut punkter**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="1ae08-436">Välj **Azure-slutpunkten**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="1ae08-437">Under **status**väljer du **aktive rad**och väljer sedan **Spara**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Aktivera Azure-slutpunkten i Azure Portal](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="1ae08-439">I **slut punkter** för Traffic Manager profilen väljer du **extern slut punkt**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="1ae08-440">Under **status**väljer du **inaktive rad**och väljer sedan **Spara**.</span><span class="sxs-lookup"><span data-stu-id="1ae08-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Inaktivera Azure Stack Hub-slutpunkten i Azure Portal](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="1ae08-442">När slut punkterna har kon figurer ATS går appens trafik till din Azure Scale-Out-webbapp i stället för Azure Stack Hub-webbappen.</span><span class="sxs-lookup"><span data-stu-id="1ae08-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Slut punkterna har ändrats i Azure Web App-trafik](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="1ae08-444">Ändra tillbaka flödet till Azure Stack Hub genom att följa de föregående stegen för att:</span><span class="sxs-lookup"><span data-stu-id="1ae08-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="1ae08-445">Aktivera Azure Stack Hub-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="1ae08-446">Inaktivera Azure-slutpunkten.</span><span class="sxs-lookup"><span data-stu-id="1ae08-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="1ae08-447">Konfigurera automatisk växling mellan Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="1ae08-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1ae08-448">Du kan också använda Application Insights övervakning om din app körs i en [Server](https://azure.microsoft.com/overview/serverless-computing/) lös miljö som tillhandahålls av Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="1ae08-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="1ae08-449">I det här scenariot kan du konfigurera Application Insights att använda en webhook som anropar en Function-app.</span><span class="sxs-lookup"><span data-stu-id="1ae08-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="1ae08-450">Den här appen aktiverar eller inaktiverar automatiskt en slut punkt som svar på en avisering.</span><span class="sxs-lookup"><span data-stu-id="1ae08-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="1ae08-451">Använd följande steg som en guide för att konfigurera automatisk trafik växling.</span><span class="sxs-lookup"><span data-stu-id="1ae08-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="1ae08-452">Skapa en Azure Function-app.</span><span class="sxs-lookup"><span data-stu-id="1ae08-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="1ae08-453">Skapa en HTTP-utlöst funktion.</span><span class="sxs-lookup"><span data-stu-id="1ae08-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="1ae08-454">Importera Azure SDK: er för Resource Manager, Web Apps och Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="1ae08-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="1ae08-455">Utveckla kod till:</span><span class="sxs-lookup"><span data-stu-id="1ae08-455">Develop code to:</span></span>

   - <span data-ttu-id="1ae08-456">Autentisera till din Azure-prenumeration.</span><span class="sxs-lookup"><span data-stu-id="1ae08-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="1ae08-457">Använd en parameter som växlar Traffic Manager slut punkter för att dirigera trafik till Azure eller Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1ae08-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="1ae08-458">Spara koden och Lägg till funktions appens URL med lämpliga parametrar i **webhook** -avsnittet i Application Insights varnings regel inställningar.</span><span class="sxs-lookup"><span data-stu-id="1ae08-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="1ae08-459">Trafiken omdirigeras automatiskt när en Application Insights-avisering utlöses.</span><span class="sxs-lookup"><span data-stu-id="1ae08-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1ae08-460">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="1ae08-460">Next steps</span></span>

- <span data-ttu-id="1ae08-461">Mer information om moln mönster i Azure finns i [design mönster för molnet](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="1ae08-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
