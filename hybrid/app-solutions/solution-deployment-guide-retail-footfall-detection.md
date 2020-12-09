---
title: Distribuera AI-baserad lösning för identifiering av Footfall i Azure och Azure Stack hubb
description: Lär dig hur du distribuerar en AI-baserad lösning för identifiering av Footfall för att analysera besökares trafik i butiker med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2177b32474dea695967e197acbd4bc1e18422d7b
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901498"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="7ec51-103">Distribuera en AI-baserad lösning för identifiering av Footfall med Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="7ec51-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="7ec51-104">Den här artikeln beskriver hur du distribuerar en AI-baserad lösning som genererar insikter från verkliga världs åtgärder med hjälp av Azure, Azure Stack hubb och Custom Vision AI dev kit.</span><span class="sxs-lookup"><span data-stu-id="7ec51-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="7ec51-105">I den här lösningen får du lära dig att:</span><span class="sxs-lookup"><span data-stu-id="7ec51-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="7ec51-106">Distribuera inbyggda Cloud-programpaket (CNAB) i gränsen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="7ec51-107">Distribuera en app som omfattar moln gränser.</span><span class="sxs-lookup"><span data-stu-id="7ec51-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="7ec51-108">Använd Custom Vision AI dev kit för att göra en härledning på gränsen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="7ec51-109">![Diagram över hybrid pelare](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="7ec51-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="7ec51-110">Microsoft Azure Stack Hub är ett tillägg till Azure.</span><span class="sxs-lookup"><span data-stu-id="7ec51-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="7ec51-111">Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.</span><span class="sxs-lookup"><span data-stu-id="7ec51-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="7ec51-112">Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program.</span><span class="sxs-lookup"><span data-stu-id="7ec51-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="7ec51-113">Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.</span><span class="sxs-lookup"><span data-stu-id="7ec51-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="7ec51-114">Krav</span><span class="sxs-lookup"><span data-stu-id="7ec51-114">Prerequisites</span></span>

<span data-ttu-id="7ec51-115">Innan du börjar med den här distributions guiden kontrollerar du att:</span><span class="sxs-lookup"><span data-stu-id="7ec51-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="7ec51-116">Läs avsnittet om [identifierings mönstret för Footfall](pattern-retail-footfall-detection.md) .</span><span class="sxs-lookup"><span data-stu-id="7ec51-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="7ec51-117">Få användar åtkomst till en Azure Stack Development Kit (ASDK) eller en integrerad system instans för Azure Stack Hub med:</span><span class="sxs-lookup"><span data-stu-id="7ec51-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="7ec51-118">[Azure App Service på Azure Stack Hub-resurs-providern](/azure-stack/operator/azure-stack-app-service-overview.md) installerad.</span><span class="sxs-lookup"><span data-stu-id="7ec51-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="7ec51-119">Du måste ha operatörs åtkomst till Azure Stack Hub-instansen eller arbeta med administratören för att installera.</span><span class="sxs-lookup"><span data-stu-id="7ec51-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="7ec51-120">En prenumeration på ett erbjudande som ger App Service och lagrings kvot.</span><span class="sxs-lookup"><span data-stu-id="7ec51-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="7ec51-121">Du behöver operatörs åtkomst för att skapa ett erbjudande.</span><span class="sxs-lookup"><span data-stu-id="7ec51-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="7ec51-122">Få åtkomst till en Azure-prenumeration.</span><span class="sxs-lookup"><span data-stu-id="7ec51-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="7ec51-123">Om du inte har en Azure-prenumeration kan du registrera dig för ett [kostnads fritt utvärderings konto](https://azure.microsoft.com/free/) innan du börjar.</span><span class="sxs-lookup"><span data-stu-id="7ec51-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="7ec51-124">Skapa två tjänst huvud namn i din katalog:</span><span class="sxs-lookup"><span data-stu-id="7ec51-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="7ec51-125">En som är konfigurerad för användning med Azure-resurser, med åtkomst i Azure-prenumerationens omfattning.</span><span class="sxs-lookup"><span data-stu-id="7ec51-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="7ec51-126">En konfiguration som ska användas med Azure Stack hubb resurser, med åtkomst till prenumerations omfånget Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7ec51-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="7ec51-127">Mer information om hur du skapar tjänstens huvud namn och hur du auktoriserar åtkomst finns i [använda en app-identitet för att få åtkomst till resurser](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="7ec51-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="7ec51-128">Om du föredrar att använda Azure CLI kan du läsa [skapa ett Azure-tjänstens huvud namn med Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span><span class="sxs-lookup"><span data-stu-id="7ec51-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="7ec51-129">Distribuera Azure Cognitive Services i Azure eller Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7ec51-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="7ec51-130">Börja med att [läsa mer om Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="7ec51-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="7ec51-131">Gå sedan till [Distribuera Azure Cognitive Services till Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) för att distribuera Cognitive Services på Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7ec51-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="7ec51-132">Du måste först registrera dig för att få åtkomst till för hands versionen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="7ec51-133">Klona eller hämta ett Azure Custom Vision AI dev-paket som inte har kon figurer ATS.</span><span class="sxs-lookup"><span data-stu-id="7ec51-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="7ec51-134">Mer information finns i [AI-DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="7ec51-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="7ec51-135">Registrera dig för ett Power BI-konto.</span><span class="sxs-lookup"><span data-stu-id="7ec51-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="7ec51-136">En Azure Cognitive Services Ansikts-API prenumerations nyckel och slut punkts-URL.</span><span class="sxs-lookup"><span data-stu-id="7ec51-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="7ec51-137">Du kan få båda med [försöket Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) kostnads fri utvärdering.</span><span class="sxs-lookup"><span data-stu-id="7ec51-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="7ec51-138">Eller följ instruktionerna i [skapa ett Cognitive Services konto](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="7ec51-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="7ec51-139">Installera följande utvecklings resurser:</span><span class="sxs-lookup"><span data-stu-id="7ec51-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="7ec51-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="7ec51-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="7ec51-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="7ec51-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="7ec51-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="7ec51-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="7ec51-143">Du kan använda Porter för att distribuera molnappar med CNAB-paket manifest som du har fått.</span><span class="sxs-lookup"><span data-stu-id="7ec51-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="7ec51-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="7ec51-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="7ec51-145">Azure IoT-verktyg för Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="7ec51-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="7ec51-146">Python-tillägg för Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="7ec51-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="7ec51-147">Python</span><span class="sxs-lookup"><span data-stu-id="7ec51-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="7ec51-148">Distribuera Hybrid Cloud-appen</span><span class="sxs-lookup"><span data-stu-id="7ec51-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="7ec51-149">Först använder du Porter CLI för att generera en Credential-uppsättning och sedan distribuera Cloud-appen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="7ec51-150">Klona eller hämta lösnings exempel koden från https://github.com/azure-samples/azure-intelligent-edge-patterns .</span><span class="sxs-lookup"><span data-stu-id="7ec51-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="7ec51-151">Porter genererar en uppsättning autentiseringsuppgifter som automatiserar distributionen av appen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="7ec51-152">Innan du kör kommandot för att skapa autentiseringsuppgifter måste du ha följande tillgängligt:</span><span class="sxs-lookup"><span data-stu-id="7ec51-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="7ec51-153">Ett tjänst huvud namn för åtkomst till Azure-resurser, inklusive tjänstens huvud namns-ID, nyckel och klient-DNS.</span><span class="sxs-lookup"><span data-stu-id="7ec51-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="7ec51-154">Prenumerations-ID för din Azure-prenumeration.</span><span class="sxs-lookup"><span data-stu-id="7ec51-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="7ec51-155">Ett tjänst huvud namn för att få åtkomst till Azure Stack Hub-resurser, inklusive tjänstens huvud namn ID, nyckel och klient-DNS.</span><span class="sxs-lookup"><span data-stu-id="7ec51-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="7ec51-156">Prenumerations-ID för Azure Stack Hub-prenumerationen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="7ec51-157">Din Azure Cognitive Services Ansikts-API nyckel-och resurs slut punkts-URL.</span><span class="sxs-lookup"><span data-stu-id="7ec51-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="7ec51-158">Kör processen för generering av Porter-autentiseringsuppgifter och följ anvisningarna:</span><span class="sxs-lookup"><span data-stu-id="7ec51-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="7ec51-159">Porter kräver också att en uppsättning parametrar körs.</span><span class="sxs-lookup"><span data-stu-id="7ec51-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="7ec51-160">Skapa en parameter text fil och ange följande namn/värde-par.</span><span class="sxs-lookup"><span data-stu-id="7ec51-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="7ec51-161">Be Azure Stack Hub-administratören om du behöver hjälp med något av de värden som krävs.</span><span class="sxs-lookup"><span data-stu-id="7ec51-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="7ec51-162">`resource suffix`Värdet används för att säkerställa att distributionens resurser har unika namn i Azure.</span><span class="sxs-lookup"><span data-stu-id="7ec51-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="7ec51-163">Det måste vara en unik sträng med bokstäver och siffror, inte längre än 8 tecken.</span><span class="sxs-lookup"><span data-stu-id="7ec51-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="7ec51-164">Spara text filen och anteckna sökvägen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="7ec51-165">Nu är du redo att distribuera Hybrid Cloud-appen med Porter.</span><span class="sxs-lookup"><span data-stu-id="7ec51-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="7ec51-166">Kör installations kommandot och se när resurser distribueras till Azure och Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="7ec51-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="7ec51-167">När distributionen är klar noterar du följande värden:</span><span class="sxs-lookup"><span data-stu-id="7ec51-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="7ec51-168">Kamerans anslutnings sträng.</span><span class="sxs-lookup"><span data-stu-id="7ec51-168">The camera's connection string.</span></span>
    - <span data-ttu-id="7ec51-169">Anslutnings strängen för avbildnings lagrings kontot.</span><span class="sxs-lookup"><span data-stu-id="7ec51-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="7ec51-170">Resurs gruppens namn.</span><span class="sxs-lookup"><span data-stu-id="7ec51-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="7ec51-171">Förbereda Custom Vision AI-DevKit</span><span class="sxs-lookup"><span data-stu-id="7ec51-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="7ec51-172">Konfigurera sedan Custom Vision AI dev kit som visas i den [vision AI-DevKit snabb start](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="7ec51-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="7ec51-173">Du kan också konfigurera och testa kameran med hjälp av anslutnings strängen som anges i föregående steg.</span><span class="sxs-lookup"><span data-stu-id="7ec51-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="7ec51-174">Distribuera appen kamera</span><span class="sxs-lookup"><span data-stu-id="7ec51-174">Deploy the camera app</span></span>

<span data-ttu-id="7ec51-175">Använd Porter CLI för att generera en uppsättning autentiseringsuppgifter och distribuera sedan appen kamera.</span><span class="sxs-lookup"><span data-stu-id="7ec51-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="7ec51-176">Porter genererar en uppsättning autentiseringsuppgifter som automatiserar distributionen av appen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="7ec51-177">Innan du kör kommandot för att skapa autentiseringsuppgifter måste du ha följande tillgängligt:</span><span class="sxs-lookup"><span data-stu-id="7ec51-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="7ec51-178">Ett tjänst huvud namn för åtkomst till Azure-resurser, inklusive tjänstens huvud namns-ID, nyckel och klient-DNS.</span><span class="sxs-lookup"><span data-stu-id="7ec51-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="7ec51-179">Prenumerations-ID för din Azure-prenumeration.</span><span class="sxs-lookup"><span data-stu-id="7ec51-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="7ec51-180">Anslutnings strängen för avbildnings lagrings kontot som angavs när du distribuerade Cloud App.</span><span class="sxs-lookup"><span data-stu-id="7ec51-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="7ec51-181">Kör processen för generering av Porter-autentiseringsuppgifter och följ anvisningarna:</span><span class="sxs-lookup"><span data-stu-id="7ec51-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="7ec51-182">Porter kräver också att en uppsättning parametrar körs.</span><span class="sxs-lookup"><span data-stu-id="7ec51-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="7ec51-183">Skapa en parameter text fil och ange följande text.</span><span class="sxs-lookup"><span data-stu-id="7ec51-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="7ec51-184">Be Azure Stack Hub-administratören om du inte vet några av de nödvändiga värdena.</span><span class="sxs-lookup"><span data-stu-id="7ec51-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="7ec51-185">`deployment suffix`Värdet används för att säkerställa att distributionens resurser har unika namn i Azure.</span><span class="sxs-lookup"><span data-stu-id="7ec51-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="7ec51-186">Det måste vara en unik sträng med bokstäver och siffror, inte längre än 8 tecken.</span><span class="sxs-lookup"><span data-stu-id="7ec51-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="7ec51-187">Spara text filen och anteckna sökvägen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="7ec51-188">Nu är du redo att distribuera Camera-appen med Porter.</span><span class="sxs-lookup"><span data-stu-id="7ec51-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="7ec51-189">Kör installations kommandot och se när IoT Edge distributionen har skapats.</span><span class="sxs-lookup"><span data-stu-id="7ec51-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="7ec51-190">Kontrol lera att kamerans distribution är slutförd genom att visa kamerans feed på `https://<camera-ip>:3000/` , där `<camara-ip>` är kamerans IP-adress.</span><span class="sxs-lookup"><span data-stu-id="7ec51-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="7ec51-191">Det här steget kan ta upp till 10 minuter.</span><span class="sxs-lookup"><span data-stu-id="7ec51-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="7ec51-192">Konfigurera Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="7ec51-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="7ec51-193">Nu när data flödar till Azure Stream Analytics från kameran måste vi manuellt godkänna det för att kommunicera med Power BI.</span><span class="sxs-lookup"><span data-stu-id="7ec51-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="7ec51-194">Från Azure Portal öppnar du **alla resurser** och *\[ yoursuffix \] -jobbet för process Footfall* .</span><span class="sxs-lookup"><span data-stu-id="7ec51-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="7ec51-195">I avsnittet **Jobbtopologi** i Stream Analytics-jobbfönstret väljer du alternativet **Utdata**.</span><span class="sxs-lookup"><span data-stu-id="7ec51-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="7ec51-196">Välj Sink för utgående **trafik** .</span><span class="sxs-lookup"><span data-stu-id="7ec51-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="7ec51-197">Välj **förnya auktorisering** och logga in på ditt Power BI-konto.</span><span class="sxs-lookup"><span data-stu-id="7ec51-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Förnya behörighets frågan i Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="7ec51-199">Spara inställningarna för utdata.</span><span class="sxs-lookup"><span data-stu-id="7ec51-199">Save the output settings.</span></span>

6. <span data-ttu-id="7ec51-200">Gå till **översikts** fönstret och välj **Starta** för att börja skicka data till Power BI.</span><span class="sxs-lookup"><span data-stu-id="7ec51-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="7ec51-201">Välj **Nu** som starttid för jobbutdata och välj **Start**.</span><span class="sxs-lookup"><span data-stu-id="7ec51-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="7ec51-202">Du kan se dess status i meddelandefältet.</span><span class="sxs-lookup"><span data-stu-id="7ec51-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="7ec51-203">Skapa en instrument panel för Power BI</span><span class="sxs-lookup"><span data-stu-id="7ec51-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="7ec51-204">När jobbet har slutförts går du till [Power BI](https://powerbi.com/) och loggar in med ditt arbets-eller skol konto.</span><span class="sxs-lookup"><span data-stu-id="7ec51-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="7ec51-205">Om Stream Analytics jobbets fråga resulterar i resultat, finns den data uppsättning för *Footfall-dataset* som du skapade under fliken **data uppsättningar** .</span><span class="sxs-lookup"><span data-stu-id="7ec51-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="7ec51-206">Från arbets ytan Power BI väljer du **+ skapa** för att skapa en ny instrument panel med namnet *Footfall analys.*</span><span class="sxs-lookup"><span data-stu-id="7ec51-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="7ec51-207">Välj **Lägg till panel** högst upp i fönstret.</span><span class="sxs-lookup"><span data-stu-id="7ec51-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="7ec51-208">Välj sedan **Anpassade strömmande data** och **Nästa**.</span><span class="sxs-lookup"><span data-stu-id="7ec51-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="7ec51-209">Välj **Footfall-dataset** under **dina data uppsättningar**.</span><span class="sxs-lookup"><span data-stu-id="7ec51-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="7ec51-210">Välj **kort** i list rutan **typ av visualisering** och Lägg till **ålder** i **fält**.</span><span class="sxs-lookup"><span data-stu-id="7ec51-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="7ec51-211">Välj **Nästa** för att ange ett namn på panelen och välj sedan **Applicera** för att skapa panelen.</span><span class="sxs-lookup"><span data-stu-id="7ec51-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="7ec51-212">Du kan lägga till ytterligare fält och kort som du vill.</span><span class="sxs-lookup"><span data-stu-id="7ec51-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="7ec51-213">Testa din lösning</span><span class="sxs-lookup"><span data-stu-id="7ec51-213">Test Your Solution</span></span>

<span data-ttu-id="7ec51-214">Observera hur data i korten som du skapade i Power BI ändras när olika personer går framför kameran.</span><span class="sxs-lookup"><span data-stu-id="7ec51-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="7ec51-215">Det kan ta upp till 20 sekunder innan Inferences har registrerats.</span><span class="sxs-lookup"><span data-stu-id="7ec51-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="7ec51-216">Ta bort din lösning</span><span class="sxs-lookup"><span data-stu-id="7ec51-216">Remove Your Solution</span></span>

<span data-ttu-id="7ec51-217">Om du vill ta bort lösningen kör du följande kommandon med hjälp av Porter med samma parameter-filer som du skapade för distribution:</span><span class="sxs-lookup"><span data-stu-id="7ec51-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="7ec51-218">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="7ec51-218">Next steps</span></span>

- <span data-ttu-id="7ec51-219">Läs mer om [design överväganden för Hybrid appar](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="7ec51-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="7ec51-220">Granska och föreslå förbättringar av [koden för det här exemplet på GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="7ec51-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
