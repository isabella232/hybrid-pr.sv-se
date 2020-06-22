---
title: Distribuera en SQL Server 2016-tillgänglighets grupp till Azure och Azure Stack Hub
description: Lär dig hur du distribuerar en tillgänglighets grupp för SQL Server 2016 till Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ff6d5b9667e63a6b8d232b6dd93db2d8b12fd46d
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911925"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="b634e-103">Distribuera en SQL Server 2016-tillgänglighets grupp till Azure och Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="b634e-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="b634e-104">I den här artikeln får du stegvisa anvisningar genom en automatiserad distribution av ett grundläggande (HA) SQL Server 2016 Enterprise-kluster med en asynkron haveri beredskap (DR) i två Azure Stack Hub-miljöer.</span><span class="sxs-lookup"><span data-stu-id="b634e-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="b634e-105">Mer information om SQL Server 2016 och hög tillgänglighet finns i [Always on-tillgänglighets grupper: en lösning för hög tillgänglighet och katastrof återställning](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="b634e-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="b634e-106">I den här lösningen skapar du en exempel miljö för att:</span><span class="sxs-lookup"><span data-stu-id="b634e-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="b634e-107">Dirigera en distribution mellan två Azure Stack hubbar.</span><span class="sxs-lookup"><span data-stu-id="b634e-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="b634e-108">Använd Docker för att minimera beroende problem med Azure API-profiler.</span><span class="sxs-lookup"><span data-stu-id="b634e-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="b634e-109">Distribuera ett grundläggande SQL Server 2016 Enterprise-kluster med en katastrof återställnings plats.</span><span class="sxs-lookup"><span data-stu-id="b634e-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="b634e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="b634e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="b634e-111">Microsoft Azure Stack Hub är ett tillägg till Azure.</span><span class="sxs-lookup"><span data-stu-id="b634e-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="b634e-112">Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling i din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.</span><span class="sxs-lookup"><span data-stu-id="b634e-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="b634e-113">Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program.</span><span class="sxs-lookup"><span data-stu-id="b634e-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="b634e-114">Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.</span><span class="sxs-lookup"><span data-stu-id="b634e-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="b634e-115">Arkitektur för SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="b634e-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="b634e-117">Krav för SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="b634e-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="b634e-118">Två anslutna Azure Stack Hub-integrerade system (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="b634e-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="b634e-119">Den här distributionen fungerar inte på Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="b634e-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="b634e-120">Mer information om Azure Stack Hub finns i [Azure Stack översikt](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="b634e-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="b634e-121">En klient prenumeration på varje Azure Stack hubb.</span><span class="sxs-lookup"><span data-stu-id="b634e-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="b634e-122">**Anteckna varje prenumerations-ID och Azure Resource Manager slut punkten för varje Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="b634e-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="b634e-123">Ett Azure Active Directory (Azure AD)-tjänstens huvud namn som har behörighet till klient prenumerationen på varje Azure Stack hubb.</span><span class="sxs-lookup"><span data-stu-id="b634e-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="b634e-124">Du kan behöva skapa två huvud namn för tjänsten om Azure Stack hubbar distribueras mot olika Azure AD-klienter.</span><span class="sxs-lookup"><span data-stu-id="b634e-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="b634e-125">Information om hur du skapar ett huvud namn för tjänsten för Azure Stack hubb finns i [skapa tjänstens huvud namn för att ge appar åtkomst till Azure Stack Hub-resurser](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="b634e-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="b634e-126">**Anteckna varje tjänst objekts program-ID, klient hemlighet och klient namn (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="b634e-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="b634e-127">SQL Server 2016 Enterprise syndikerat till varje Azure Stack Hubbs marknads plats.</span><span class="sxs-lookup"><span data-stu-id="b634e-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="b634e-128">Läs mer om Marketplace-syndikering i [Hämta Marketplace-objekt till Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="b634e-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="b634e-129">**Kontrol lera att din organisation har rätt SQL-licenser.**</span><span class="sxs-lookup"><span data-stu-id="b634e-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="b634e-130">[Docker för Windows](https://docs.docker.com/docker-for-windows/) installerat på den lokala datorn.</span><span class="sxs-lookup"><span data-stu-id="b634e-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="b634e-131">Hämta Docker-avbildningen</span><span class="sxs-lookup"><span data-stu-id="b634e-131">Get the Docker image</span></span>

<span data-ttu-id="b634e-132">Docker-avbildningar för varje distribution eliminerar beroende problem mellan olika versioner av Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="b634e-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="b634e-133">Se till att Docker för Windows använder Windows-behållare.</span><span class="sxs-lookup"><span data-stu-id="b634e-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="b634e-134">Kör följande skript i en upphöjd kommando tolk för att hämta Docker-behållaren med distributions skripten.</span><span class="sxs-lookup"><span data-stu-id="b634e-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="b634e-135">Distribuera tillgänglighets gruppen</span><span class="sxs-lookup"><span data-stu-id="b634e-135">Deploy the availability group</span></span>

1. <span data-ttu-id="b634e-136">Starta avbildningen när behållar avbildningen har tagits emot.</span><span class="sxs-lookup"><span data-stu-id="b634e-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="b634e-137">När behållaren har startats får du en upphöjd PowerShell-Terminal i behållaren.</span><span class="sxs-lookup"><span data-stu-id="b634e-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="b634e-138">Ändra kataloger för att komma till distributions skriptet.</span><span class="sxs-lookup"><span data-stu-id="b634e-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="b634e-139">Kör distributionen.</span><span class="sxs-lookup"><span data-stu-id="b634e-139">Run the deployment.</span></span> <span data-ttu-id="b634e-140">Ange autentiseringsuppgifter och resurs namn där det behövs.</span><span class="sxs-lookup"><span data-stu-id="b634e-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="b634e-141">HA refererar till Azure Stack hubben där HA-klustret ska distribueras.</span><span class="sxs-lookup"><span data-stu-id="b634e-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="b634e-142">DR refererar till Azure Stack hubben där DR-klustret ska distribueras.</span><span class="sxs-lookup"><span data-stu-id="b634e-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="b634e-143">Skriv `Y` för att tillåta att NuGet-providern installeras, vilket kommer att starta API-profilen "2018-03-01-hybrid"-moduler som ska installeras.</span><span class="sxs-lookup"><span data-stu-id="b634e-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="b634e-144">Vänta tills resurs distributionen har slutförts.</span><span class="sxs-lookup"><span data-stu-id="b634e-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="b634e-145">När du har slutfört återställningen av DR-resursen avslutar du behållaren.</span><span class="sxs-lookup"><span data-stu-id="b634e-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="b634e-146">Granska distributionen genom att Visa resurserna i varje Azure Stack Hubbs Portal.</span><span class="sxs-lookup"><span data-stu-id="b634e-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="b634e-147">Anslut till en av SQL-instanserna i HA-miljön och granska tillgänglighets gruppen via SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="b634e-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="b634e-149">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="b634e-149">Next steps</span></span>

- <span data-ttu-id="b634e-150">Använd SQL Server Management Studio för att redundansväxla klustret manuellt.</span><span class="sxs-lookup"><span data-stu-id="b634e-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="b634e-151">Se [utföra en framtvingad manuell redundansväxling av en tillgänglighets grupp som alltid är tillgänglig (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="b634e-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="b634e-152">Läs mer om hybrid molnappar.</span><span class="sxs-lookup"><span data-stu-id="b634e-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="b634e-153">Se [hybrid moln lösningar.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="b634e-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="b634e-154">Använd dina egna data eller ändra koden till det här exemplet på [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="b634e-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
