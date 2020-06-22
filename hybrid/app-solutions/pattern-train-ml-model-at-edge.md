---
title: Träna maskin inlärnings modell i gräns mönstret
description: Lär dig hur du gör Machine Learning-modellens utbildning på gränsen med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911949"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="a20b1-103">Träna maskin inlärnings modell i gräns mönstret</span><span class="sxs-lookup"><span data-stu-id="a20b1-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="a20b1-104">Generera modeller för bärbara Machine Learning (ML) från data som bara finns lokalt.</span><span class="sxs-lookup"><span data-stu-id="a20b1-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="a20b1-105">Kontext och problem</span><span class="sxs-lookup"><span data-stu-id="a20b1-105">Context and problem</span></span>

<span data-ttu-id="a20b1-106">Många organisationer skulle vilja låsa upp insikter från sina lokala eller äldre data med hjälp av verktyg som deras data experter förstår.</span><span class="sxs-lookup"><span data-stu-id="a20b1-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="a20b1-107">[Azure Machine Learning](/azure/machine-learning/) tillhandahåller Cloud-inbyggt verktyg för att träna, finjustera och distribuera ml-och djup inlärnings modeller.</span><span class="sxs-lookup"><span data-stu-id="a20b1-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="a20b1-108">Men vissa data är för stora för att skickas till molnet eller kan inte skickas till molnet av reglerings skäl.</span><span class="sxs-lookup"><span data-stu-id="a20b1-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="a20b1-109">Med det här mönstret kan data forskare använda Azure Machine Learning för att träna modeller med lokala data och data bearbetning.</span><span class="sxs-lookup"><span data-stu-id="a20b1-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="a20b1-110">Lösning</span><span class="sxs-lookup"><span data-stu-id="a20b1-110">Solution</span></span>

<span data-ttu-id="a20b1-111">Träningen vid gräns mönstret använder en virtuell dator (VM) som körs på Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a20b1-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="a20b1-112">Den virtuella datorn är registrerad som ett beräknings mål i Azure ML, så att den får åtkomst till data som bara finns lokalt.</span><span class="sxs-lookup"><span data-stu-id="a20b1-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="a20b1-113">I det här fallet lagras data i Azure Stack hubbens Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="a20b1-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="a20b1-114">När modellen har tränats registreras den med Azure ML, containerd och läggs till i en Azure Container Registry för distribution.</span><span class="sxs-lookup"><span data-stu-id="a20b1-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="a20b1-115">För den här iterationen av mönstret måste den virtuella datorn för Azure Stack Hub-utbildning vara tillgänglig via det offentliga Internet.</span><span class="sxs-lookup"><span data-stu-id="a20b1-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="a20b1-116">[![Träna ML-modell på gräns arkitekturen](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="a20b1-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="a20b1-117">Så här fungerar mönstret:</span><span class="sxs-lookup"><span data-stu-id="a20b1-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="a20b1-118">Den virtuella datorn Azure Stack Hub distribueras och registreras som ett beräknings mål med Azure ML.</span><span class="sxs-lookup"><span data-stu-id="a20b1-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="a20b1-119">Ett experiment skapas i Azure ML som använder den virtuella datorn Azure Stack Hub som ett beräknings mål.</span><span class="sxs-lookup"><span data-stu-id="a20b1-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="a20b1-120">När modellen har tränats registreras den och containern.</span><span class="sxs-lookup"><span data-stu-id="a20b1-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="a20b1-121">Modellen kan nu distribueras till platser som antingen är lokala eller i molnet.</span><span class="sxs-lookup"><span data-stu-id="a20b1-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="a20b1-122">Komponenter</span><span class="sxs-lookup"><span data-stu-id="a20b1-122">Components</span></span>

<span data-ttu-id="a20b1-123">Den här lösningen använder följande komponenter:</span><span class="sxs-lookup"><span data-stu-id="a20b1-123">This solution uses the following components:</span></span>

| <span data-ttu-id="a20b1-124">Lager</span><span class="sxs-lookup"><span data-stu-id="a20b1-124">Layer</span></span> | <span data-ttu-id="a20b1-125">Komponent</span><span class="sxs-lookup"><span data-stu-id="a20b1-125">Component</span></span> | <span data-ttu-id="a20b1-126">Description</span><span class="sxs-lookup"><span data-stu-id="a20b1-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="a20b1-127">Azure</span><span class="sxs-lookup"><span data-stu-id="a20b1-127">Azure</span></span> | <span data-ttu-id="a20b1-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="a20b1-128">Azure Machine Learning</span></span> | <span data-ttu-id="a20b1-129">[Azure Machine Learning](/azure/machine-learning/) dirigerar utbildningen för ml-modellen.</span><span class="sxs-lookup"><span data-stu-id="a20b1-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="a20b1-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="a20b1-130">Azure Container Registry</span></span> | <span data-ttu-id="a20b1-131">Azure ML paketerar modellen i en behållare och lagrar den i en [Azure Container Registry](/azure/container-registry/) för distribution.</span><span class="sxs-lookup"><span data-stu-id="a20b1-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="a20b1-132">Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="a20b1-132">Azure Stack Hub</span></span> | <span data-ttu-id="a20b1-133">App Service</span><span class="sxs-lookup"><span data-stu-id="a20b1-133">App Service</span></span> | <span data-ttu-id="a20b1-134">[Azure Stack hubb med App Service](/azure-stack/operator/azure-stack-app-service-overview) tillhandahåller basen för komponenterna på gränsen.</span><span class="sxs-lookup"><span data-stu-id="a20b1-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="a20b1-135">Compute</span><span class="sxs-lookup"><span data-stu-id="a20b1-135">Compute</span></span> | <span data-ttu-id="a20b1-136">En Azure Stack Hub-dator som kör Ubuntu med Docker används för att träna ML-modellen.</span><span class="sxs-lookup"><span data-stu-id="a20b1-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="a20b1-137">Storage</span><span class="sxs-lookup"><span data-stu-id="a20b1-137">Storage</span></span> | <span data-ttu-id="a20b1-138">Privata data kan finnas i Azure Stack hubb Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="a20b1-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="a20b1-139">Problem och överväganden</span><span class="sxs-lookup"><span data-stu-id="a20b1-139">Issues and considerations</span></span>

<span data-ttu-id="a20b1-140">Tänk på följande när du bestämmer hur du ska implementera den här lösningen:</span><span class="sxs-lookup"><span data-stu-id="a20b1-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="a20b1-141">Skalbarhet</span><span class="sxs-lookup"><span data-stu-id="a20b1-141">Scalability</span></span>

<span data-ttu-id="a20b1-142">Om du vill aktivera den här lösningen för skalning måste du skapa en lämplig virtuell dator på Azure Stack hubb för utbildning.</span><span class="sxs-lookup"><span data-stu-id="a20b1-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="a20b1-143">Tillgänglighet</span><span class="sxs-lookup"><span data-stu-id="a20b1-143">Availability</span></span>

<span data-ttu-id="a20b1-144">Se till att utbildnings skripten och Azure Stack Hub VM har åtkomst till lokala data som används för utbildning.</span><span class="sxs-lookup"><span data-stu-id="a20b1-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="a20b1-145">Hanterbarhet</span><span class="sxs-lookup"><span data-stu-id="a20b1-145">Manageability</span></span>

<span data-ttu-id="a20b1-146">Se till att modeller och experiment är korrekt registrerade, versioner och taggade för att undvika förvirring vid modell distribution.</span><span class="sxs-lookup"><span data-stu-id="a20b1-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="a20b1-147">Säkerhet</span><span class="sxs-lookup"><span data-stu-id="a20b1-147">Security</span></span>

<span data-ttu-id="a20b1-148">Det här mönstret ger Azure ML åtkomst till möjliga känsliga data lokalt.</span><span class="sxs-lookup"><span data-stu-id="a20b1-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="a20b1-149">Se till att det konto som används för SSH i Azure Stack Hub-VM har ett starkt lösen ord och utbildnings skript inte bevarar eller överför data till molnet.</span><span class="sxs-lookup"><span data-stu-id="a20b1-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a20b1-150">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="a20b1-150">Next steps</span></span>

<span data-ttu-id="a20b1-151">Mer information om ämnen som introduceras i den här artikeln:</span><span class="sxs-lookup"><span data-stu-id="a20b1-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="a20b1-152">I [Azure Machine Learning-dokumentationen](/azure/machine-learning) finns en översikt över ml och närliggande ämnen.</span><span class="sxs-lookup"><span data-stu-id="a20b1-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="a20b1-153">Se [Azure Container Registry](/azure/container-registry/) för att lära dig att skapa, lagra och hantera avbildningar för behållar distributioner.</span><span class="sxs-lookup"><span data-stu-id="a20b1-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="a20b1-154">Läs mer om resurs leverantören och hur du distribuerar med hjälp av [app service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) .</span><span class="sxs-lookup"><span data-stu-id="a20b1-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="a20b1-155">Se [design överväganden för hybrid program](overview-app-design-considerations.md) för att lära dig mer om metod tips och hur du får svar på ytterligare frågor.</span><span class="sxs-lookup"><span data-stu-id="a20b1-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="a20b1-156">Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.</span><span class="sxs-lookup"><span data-stu-id="a20b1-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="a20b1-157">När du är redo att testa lösnings exemplet fortsätter du med [träna ml-modellen i gräns distributions guiden](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="a20b1-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="a20b1-158">Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter.</span><span class="sxs-lookup"><span data-stu-id="a20b1-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
