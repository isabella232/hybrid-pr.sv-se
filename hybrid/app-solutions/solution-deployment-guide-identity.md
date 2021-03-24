---
title: Konfigurera hybrid moln identitet för Azure och Azure Stack Hub-appar
description: Lär dig hur du konfigurerar hybrid moln identitet för Azure och Azure Stack Hub-appar.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895357"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="a5327-103">Konfigurera hybrid moln identitet för Azure och Azure Stack Hub-appar</span><span class="sxs-lookup"><span data-stu-id="a5327-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="a5327-104">Lär dig hur du konfigurerar en hybrid moln identitet för dina Azure-och Azure Stack Hub-appar.</span><span class="sxs-lookup"><span data-stu-id="a5327-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="a5327-105">Du har två alternativ för att bevilja åtkomst till dina appar i både Global Azure och Azure Stack hubb.</span><span class="sxs-lookup"><span data-stu-id="a5327-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="a5327-106">När Azure Stack Hub har en kontinuerlig anslutning till Internet kan du använda Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="a5327-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="a5327-107">När Azure Stack hubb är frånkopplad från Internet kan du använda Azures katalog federerade tjänster (AD FS).</span><span class="sxs-lookup"><span data-stu-id="a5327-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="a5327-108">Du använder tjänstens huvud namn för att bevilja åtkomst till dina Azure Stack Hub-appar för distribution eller konfiguration med hjälp av Azure Resource Manager i Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a5327-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="a5327-109">I den här lösningen skapar du en exempel miljö för att:</span><span class="sxs-lookup"><span data-stu-id="a5327-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="a5327-110">Upprätta en hybrid identitet i Global Azure och Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="a5327-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="a5327-111">Hämta en token för att få åtkomst till API: et för Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a5327-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="a5327-112">Du måste ha behörighet för Azure Stack Hub-operatör för stegen i den här lösningen.</span><span class="sxs-lookup"><span data-stu-id="a5327-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="a5327-113">![Diagram över hybrid pelare](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="a5327-113">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="a5327-114">Microsoft Azure Stack Hub är ett tillägg till Azure.</span><span class="sxs-lookup"><span data-stu-id="a5327-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="a5327-115">Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling i din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.</span><span class="sxs-lookup"><span data-stu-id="a5327-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="a5327-116">Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program.</span><span class="sxs-lookup"><span data-stu-id="a5327-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="a5327-117">Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.</span><span class="sxs-lookup"><span data-stu-id="a5327-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="a5327-118">Skapa ett huvud namn för tjänsten för Azure AD i portalen</span><span class="sxs-lookup"><span data-stu-id="a5327-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="a5327-119">Om du har distribuerat Azure Stack hubb med Azure AD som identitets lager kan du skapa tjänstens huvud namn precis som du gör med Azure.</span><span class="sxs-lookup"><span data-stu-id="a5327-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="a5327-120">[Använd en app-identitet för att komma åt resurser](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) visar hur du utför stegen via portalen.</span><span class="sxs-lookup"><span data-stu-id="a5327-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="a5327-121">Se till att du har [nödvändiga behörigheter för Azure AD](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) innan du börjar.</span><span class="sxs-lookup"><span data-stu-id="a5327-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="a5327-122">Skapa ett huvud namn för tjänsten för AD FS med PowerShell</span><span class="sxs-lookup"><span data-stu-id="a5327-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="a5327-123">Om du har distribuerat Azure Stack hubb med AD FS kan du använda PowerShell för att skapa ett huvud namn för tjänsten, tilldela en roll för åtkomst och logga in från PowerShell med den identiteten.</span><span class="sxs-lookup"><span data-stu-id="a5327-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="a5327-124">[Använd en app-identitet för att komma åt resurser](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) visar hur du utför de steg som krävs med hjälp av PowerShell.</span><span class="sxs-lookup"><span data-stu-id="a5327-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="a5327-125">Använda API: et för Azure Stack hubb</span><span class="sxs-lookup"><span data-stu-id="a5327-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="a5327-126">API-lösningen för [Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use)  vägleder dig genom processen att hämta en token för att få åtkomst till API: t för Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a5327-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="a5327-127">Ansluta till Azure Stack hubb med PowerShell</span><span class="sxs-lookup"><span data-stu-id="a5327-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="a5327-128">Snabb starten [för att komma igång med PowerShell i Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) vägleder dig genom de steg som krävs för att installera Azure PowerShell och ansluta till din Azure Stack Hub-installation.</span><span class="sxs-lookup"><span data-stu-id="a5327-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="a5327-129">Förutsättningar</span><span class="sxs-lookup"><span data-stu-id="a5327-129">Prerequisites</span></span>

<span data-ttu-id="a5327-130">Du behöver en Azure Stack hubb installation ansluten till Azure AD med en prenumeration som du har åtkomst till.</span><span class="sxs-lookup"><span data-stu-id="a5327-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="a5327-131">Om du inte har en Azure Stack Hub-installation kan du använda dessa instruktioner för att konfigurera en [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="a5327-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="a5327-132">Ansluta till Azure Stack hubb med hjälp av kod</span><span class="sxs-lookup"><span data-stu-id="a5327-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="a5327-133">Om du vill ansluta till Azure Stack hubb med hjälp av kod använder du API: et för Azure Resource Manager slut punkter för att hämta autentiserings-och Graf-slutpunkter för installationen av Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a5327-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="a5327-134">Autentisera sedan med REST-begäranden.</span><span class="sxs-lookup"><span data-stu-id="a5327-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="a5327-135">Du kan hitta ett exempel på ett klient program på [GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="a5327-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="a5327-136">Om inte Azure SDK för ditt språk alternativ stöder Azure API-profiler kanske SDK inte fungerar med Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a5327-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="a5327-137">Mer information om Azure API-profiler finns i artikeln [Hantera API-versioner profiler](/azure-stack/user/azure-stack-version-profiles) .</span><span class="sxs-lookup"><span data-stu-id="a5327-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a5327-138">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="a5327-138">Next steps</span></span>

- <span data-ttu-id="a5327-139">Mer information om hur identiteten hanteras i Azure Stack Hub finns i [identitets arkitektur för Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span><span class="sxs-lookup"><span data-stu-id="a5327-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span></span>
- <span data-ttu-id="a5327-140">Mer information om moln mönster i Azure finns i [design mönster för molnet](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="a5327-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
