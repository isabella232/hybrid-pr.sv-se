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
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Konfigurera hybrid moln identitet för Azure och Azure Stack Hub-appar

Lär dig hur du konfigurerar en hybrid moln identitet för dina Azure-och Azure Stack Hub-appar.

Du har två alternativ för att bevilja åtkomst till dina appar i både Global Azure och Azure Stack hubb.

 * När Azure Stack Hub har en kontinuerlig anslutning till Internet kan du använda Azure Active Directory (Azure AD).
 * När Azure Stack hubb är frånkopplad från Internet kan du använda Azures katalog federerade tjänster (AD FS).

Du använder tjänstens huvud namn för att bevilja åtkomst till dina Azure Stack Hub-appar för distribution eller konfiguration med hjälp av Azure Resource Manager i Azure Stack Hub.

I den här lösningen skapar du en exempel miljö för att:

> [!div class="checklist"]
> - Upprätta en hybrid identitet i Global Azure och Azure Stack hubb
> - Hämta en token för att få åtkomst till API: et för Azure Stack Hub.

Du måste ha behörighet för Azure Stack Hub-operatör för stegen i den här lösningen.

> [!Tip]  
> ![Diagram över hybrid pelare](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling i din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Skapa ett huvud namn för tjänsten för Azure AD i portalen

Om du har distribuerat Azure Stack hubb med Azure AD som identitets lager kan du skapa tjänstens huvud namn precis som du gör med Azure. [Använd en app-identitet för att komma åt resurser](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) visar hur du utför stegen via portalen. Se till att du har [nödvändiga behörigheter för Azure AD](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) innan du börjar.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Skapa ett huvud namn för tjänsten för AD FS med PowerShell

Om du har distribuerat Azure Stack hubb med AD FS kan du använda PowerShell för att skapa ett huvud namn för tjänsten, tilldela en roll för åtkomst och logga in från PowerShell med den identiteten. [Använd en app-identitet för att komma åt resurser](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) visar hur du utför de steg som krävs med hjälp av PowerShell.

## <a name="using-the-azure-stack-hub-api"></a>Använda API: et för Azure Stack hubb

API-lösningen för [Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use)  vägleder dig genom processen att hämta en token för att få åtkomst till API: t för Azure Stack Hub.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Ansluta till Azure Stack hubb med PowerShell

Snabb starten [för att komma igång med PowerShell i Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) vägleder dig genom de steg som krävs för att installera Azure PowerShell och ansluta till din Azure Stack Hub-installation.

### <a name="prerequisites"></a>Förutsättningar

Du behöver en Azure Stack hubb installation ansluten till Azure AD med en prenumeration som du har åtkomst till. Om du inte har en Azure Stack Hub-installation kan du använda dessa instruktioner för att konfigurera en [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Ansluta till Azure Stack hubb med hjälp av kod

Om du vill ansluta till Azure Stack hubb med hjälp av kod använder du API: et för Azure Resource Manager slut punkter för att hämta autentiserings-och Graf-slutpunkter för installationen av Azure Stack Hub. Autentisera sedan med REST-begäranden. Du kan hitta ett exempel på ett klient program på [GitHub](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>Om inte Azure SDK för ditt språk alternativ stöder Azure API-profiler kanske SDK inte fungerar med Azure Stack Hub. Mer information om Azure API-profiler finns i artikeln [Hantera API-versioner profiler](/azure-stack/user/azure-stack-version-profiles) .

## <a name="next-steps"></a>Nästa steg

- Mer information om hur identiteten hanteras i Azure Stack Hub finns i [identitets arkitektur för Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).
- Mer information om moln mönster i Azure finns i [design mönster för molnet](/azure/architecture/patterns).
