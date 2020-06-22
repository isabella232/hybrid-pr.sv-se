---
title: Distribuera en MongoDB-lösning med hög tillgänglighet till Azure och Azure Stack hubb
description: Lär dig hur du distribuerar en MongoDB-lösning med hög tillgänglighet till Azure och Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b34ba7c10ff5f658d645923ae8b6de2fb2607ccb
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912099"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="39cf6-103">Distribuera en MongoDB-lösning med hög tillgänglighet till Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="39cf6-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="39cf6-104">Den här artikeln går igenom en automatiserad distribution av ett MongoDB-kluster med hög tillgänglighet (HA) med en katastrof återställning (DR) på två Azure Stack Hubbs miljöer.</span><span class="sxs-lookup"><span data-stu-id="39cf6-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="39cf6-105">Mer information om MongoDB och hög tillgänglighet finns i [replik uppsättnings medlemmar](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="39cf6-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="39cf6-106">I den här lösningen skapar du en exempel miljö för att:</span><span class="sxs-lookup"><span data-stu-id="39cf6-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="39cf6-107">Dirigera en distribution mellan två Azure Stack hubbar.</span><span class="sxs-lookup"><span data-stu-id="39cf6-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="39cf6-108">Använd Docker för att minimera beroende problem med Azure API-profiler.</span><span class="sxs-lookup"><span data-stu-id="39cf6-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="39cf6-109">Distribuera ett Basic MongoDB-kluster med hög tillgänglighet med en katastrof återställnings plats.</span><span class="sxs-lookup"><span data-stu-id="39cf6-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="39cf6-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="39cf6-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="39cf6-111">Microsoft Azure Stack Hub är ett tillägg till Azure.</span><span class="sxs-lookup"><span data-stu-id="39cf6-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="39cf6-112">Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling i din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.</span><span class="sxs-lookup"><span data-stu-id="39cf6-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="39cf6-113">Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program.</span><span class="sxs-lookup"><span data-stu-id="39cf6-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="39cf6-114">Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.</span><span class="sxs-lookup"><span data-stu-id="39cf6-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="39cf6-115">Arkitektur för MongoDB med Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="39cf6-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![MongoDB-arkitektur med hög tillgänglighet i Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="39cf6-117">Krav för MongoDB med Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="39cf6-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="39cf6-118">Två anslutna Azure Stack Hub-integrerade system (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="39cf6-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="39cf6-119">Den här distributionen fungerar inte på Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="39cf6-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="39cf6-120">Läs mer om Azure Stack Hub i [Vad är Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="39cf6-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="39cf6-121">En klient prenumeration på varje Azure Stack hubb.</span><span class="sxs-lookup"><span data-stu-id="39cf6-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="39cf6-122">**Anteckna varje prenumerations-ID och Azure Resource Manager slut punkten för varje Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="39cf6-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="39cf6-123">Ett Azure Active Directory (Azure AD)-tjänstens huvud namn som har behörighet till klient prenumerationen på varje Azure Stack hubb.</span><span class="sxs-lookup"><span data-stu-id="39cf6-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="39cf6-124">Du kan behöva skapa två huvud namn för tjänsten om Azure Stack hubbar distribueras mot olika Azure AD-klienter.</span><span class="sxs-lookup"><span data-stu-id="39cf6-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="39cf6-125">Information om hur du skapar ett huvud namn för tjänsten för Azure Stack hubb finns i [använda en app-identitet för att få åtkomst till Azure Stack Hub-resurser](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="39cf6-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="39cf6-126">**Anteckna varje tjänst objekts program-ID, klient hemlighet och klient namn (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="39cf6-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="39cf6-127">Ubuntu 16,04 har syndikerats till varje Azure Stack Hubbs marknads plats.</span><span class="sxs-lookup"><span data-stu-id="39cf6-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="39cf6-128">Läs mer om Marketplace-syndikering i [Hämta Marketplace-objekt till Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="39cf6-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="39cf6-129">[Docker för Windows](https://docs.docker.com/docker-for-windows/) installerat på den lokala datorn.</span><span class="sxs-lookup"><span data-stu-id="39cf6-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="39cf6-130">Hämta Docker-avbildningen</span><span class="sxs-lookup"><span data-stu-id="39cf6-130">Get the Docker image</span></span>

<span data-ttu-id="39cf6-131">Docker-avbildningar för varje distribution eliminerar beroende problem mellan olika versioner av Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="39cf6-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="39cf6-132">Se till att Docker för Windows använder Windows-behållare.</span><span class="sxs-lookup"><span data-stu-id="39cf6-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="39cf6-133">Kör följande kommando i en upphöjd kommando tolk för att hämta Docker-behållaren med distributions skripten.</span><span class="sxs-lookup"><span data-stu-id="39cf6-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="39cf6-134">Distribuera klustren</span><span class="sxs-lookup"><span data-stu-id="39cf6-134">Deploy the clusters</span></span>

1. <span data-ttu-id="39cf6-135">Starta avbildningen när behållar avbildningen har tagits emot.</span><span class="sxs-lookup"><span data-stu-id="39cf6-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="39cf6-136">När behållaren har startats får du en upphöjd PowerShell-Terminal i behållaren.</span><span class="sxs-lookup"><span data-stu-id="39cf6-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="39cf6-137">Ändra kataloger för att komma till distributions skriptet.</span><span class="sxs-lookup"><span data-stu-id="39cf6-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="39cf6-138">Kör distributionen.</span><span class="sxs-lookup"><span data-stu-id="39cf6-138">Run the deployment.</span></span> <span data-ttu-id="39cf6-139">Ange autentiseringsuppgifter och resurs namn där det behövs.</span><span class="sxs-lookup"><span data-stu-id="39cf6-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="39cf6-140">HA refererar till Azure Stack hubben där HA-klustret ska distribueras.</span><span class="sxs-lookup"><span data-stu-id="39cf6-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="39cf6-141">DR refererar till Azure Stack hubben där DR-klustret ska distribueras.</span><span class="sxs-lookup"><span data-stu-id="39cf6-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="39cf6-142">Skriv `Y` för att tillåta att NuGet-providern installeras, vilket kommer att starta API-profilen "2018-03-01-hybrid"-moduler som ska installeras.</span><span class="sxs-lookup"><span data-stu-id="39cf6-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="39cf6-143">Resurserna för att distribueras först.</span><span class="sxs-lookup"><span data-stu-id="39cf6-143">The HA resources will deploy first.</span></span> <span data-ttu-id="39cf6-144">Övervaka distributionen och vänta tills den är klar.</span><span class="sxs-lookup"><span data-stu-id="39cf6-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="39cf6-145">När du har fått meddelandet om att HA-distributionen är färdig kan du kontrol lera att de resurser som distribueras har distribuerats med HA Azure Stack Hub-portalen.</span><span class="sxs-lookup"><span data-stu-id="39cf6-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="39cf6-146">Fortsätt med distributionen av DR-resurser och bestäm om du vill aktivera en hopp ruta i DR Azure Stack Hub för att interagera med klustret.</span><span class="sxs-lookup"><span data-stu-id="39cf6-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="39cf6-147">Vänta tills återställnings resurs distributionen är klar.</span><span class="sxs-lookup"><span data-stu-id="39cf6-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="39cf6-148">När återställningen av en resurs distribution har slutförts avslutar du behållaren.</span><span class="sxs-lookup"><span data-stu-id="39cf6-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="39cf6-149">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="39cf6-149">Next steps</span></span>

- <span data-ttu-id="39cf6-150">Om du har aktiverat den virtuella hopp rutan i DR Azure Stack Hub kan du ansluta via SSH och interagera med MongoDB-klustret genom att installera Mongo CLI.</span><span class="sxs-lookup"><span data-stu-id="39cf6-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="39cf6-151">Mer information om hur du interagerar med MongoDB finns i [Mongo-gränssnittet](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="39cf6-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="39cf6-152">Mer information om hybrid molnappar finns i [hybrid moln lösningar.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="39cf6-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="39cf6-153">Ändra koden till det här exemplet på [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="39cf6-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
