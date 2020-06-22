---
title: Distribuera AI-baserad lösning för identifiering av Footfall i Azure och Azure Stack hubb
description: Lär dig hur du distribuerar en AI-baserad lösning för identifiering av Footfall för att analysera besökares trafik i butiker med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911862"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Distribuera en AI-baserad lösning för identifiering av Footfall med Azure och Azure Stack hubb

Den här artikeln beskriver hur du distribuerar en AI-baserad lösning som genererar insikter från verkliga världs åtgärder med hjälp av Azure, Azure Stack hubb och Custom Vision AI dev kit.

I den här lösningen får du lära dig att:

> [!div class="checklist"]
> - Distribuera inbyggda Cloud-programpaket (CNAB) i gränsen. 
> - Distribuera en app som omfattar moln gränser.
> - Använd Custom Vision AI dev kit för att göra en härledning på gränsen.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

## <a name="prerequisites"></a>Krav

Innan du börjar med den här distributions guiden kontrollerar du att:

- Läs avsnittet om [identifierings mönstret för Footfall](pattern-retail-footfall-detection.md) .
- Få användar åtkomst till en Azure Stack Development Kit (ASDK) eller en integrerad system instans för Azure Stack Hub med:
  - [Azure App Service på Azure Stack Hub-resurs-providern](/azure-stack/operator/azure-stack-app-service-overview.md) installerad. Du måste ha operatörs åtkomst till Azure Stack Hub-instansen eller arbeta med administratören för att installera.
  - En prenumeration på ett erbjudande som ger App Service och lagrings kvot. Du behöver operatörs åtkomst för att skapa ett erbjudande.
- Få åtkomst till en Azure-prenumeration.
  - Om du inte har en Azure-prenumeration kan du registrera dig för ett [kostnads fritt utvärderings konto](https://azure.microsoft.com/free/) innan du börjar.
- Skapa två tjänst huvud namn i din katalog:
  - En som är konfigurerad för användning med Azure-resurser, med åtkomst i Azure-prenumerationens omfattning.
  - En konfiguration som ska användas med Azure Stack hubb resurser, med åtkomst till prenumerations omfånget Azure Stack Hub.
  - Mer information om hur du skapar tjänstens huvud namn och hur du auktoriserar åtkomst finns i [använda en app-identitet för att få åtkomst till resurser](/azure-stack/operator/azure-stack-create-service-principals.md). Om du föredrar att använda Azure CLI kan du läsa [skapa ett Azure-tjänstens huvud namn med Azure CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).
- Distribuera Azure Cognitive Services i Azure eller Azure Stack Hub.
  - Börja med att [läsa mer om Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).
  - Gå sedan till [Distribuera Azure Cognitive Services till Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) för att distribuera Cognitive Services på Azure Stack Hub. Du måste först registrera dig för att få åtkomst till för hands versionen.
- Klona eller hämta ett Azure Custom Vision AI dev-paket som inte har kon figurer ATS. Mer information finns i [AI-DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).
- Registrera dig för ett Power BI-konto.
- En Azure Cognitive Services Ansikts-API prenumerations nyckel och slut punkts-URL. Du kan få båda med [försöket Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) kostnads fri utvärdering. Eller följ instruktionerna i [skapa ett Cognitive Services konto](/azure/cognitive-services/cognitive-services-apis-create-account).
- Installera följande utvecklings resurser:
  - [Azure CLI 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/). Du kan använda Porter för att distribuera molnappar med CNAB-paket manifest som du har fått.
  - [Visual Studio-koden](https://code.visualstudio.com/)
  - [Azure IoT-verktyg för Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Python-tillägg för Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>Distribuera Hybrid Cloud-appen

Först använder du Porter CLI för att generera en Credential-uppsättning och sedan distribuera Cloud-appen.  

1. Klona eller hämta lösnings exempel koden från https://github.com/azure-samples/azure-intelligent-edge-patterns . 

1. Porter genererar en uppsättning autentiseringsuppgifter som automatiserar distributionen av appen. Innan du kör kommandot för att skapa autentiseringsuppgifter måste du ha följande tillgängligt:

    - Ett tjänst huvud namn för åtkomst till Azure-resurser, inklusive tjänstens huvud namns-ID, nyckel och klient-DNS.
    - Prenumerations-ID för din Azure-prenumeration.
    - Ett tjänst huvud namn för att få åtkomst till Azure Stack Hub-resurser, inklusive tjänstens huvud namn ID, nyckel och klient-DNS.
    - Prenumerations-ID för Azure Stack Hub-prenumerationen.
    - Din Azure Cognitive Services Ansikts-API nyckel-och resurs slut punkts-URL.

1. Kör processen för generering av Porter-autentiseringsuppgifter och följ anvisningarna:

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Porter kräver också att en uppsättning parametrar körs. Skapa en parameter text fil och ange följande namn/värde-par. Be Azure Stack Hub-administratören om du behöver hjälp med något av de värden som krävs.

   > [!NOTE] 
   > `resource suffix`Värdet används för att säkerställa att distributionens resurser har unika namn i Azure. Det måste vara en unik sträng med bokstäver och siffror, inte längre än 8 tecken.

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
   Spara text filen och anteckna sökvägen.

1. Nu är du redo att distribuera Hybrid Cloud-appen med Porter. Kör installations kommandot och se när resurser distribueras till Azure och Azure Stack Hub:

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. När distributionen är klar noterar du följande värden:
    - Kamerans anslutnings sträng.
    - Anslutnings strängen för avbildnings lagrings kontot.
    - Resurs gruppens namn.

## <a name="prepare-the-custom-vision-ai-devkit"></a>Förbereda Custom Vision AI-DevKit

Konfigurera sedan Custom Vision AI dev kit som visas i den [vision AI-DevKit snabb start](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/). Du kan också konfigurera och testa kameran med hjälp av anslutnings strängen som anges i föregående steg.

## <a name="deploy-the-camera-app"></a>Distribuera appen kamera

Använd Porter CLI för att generera en uppsättning autentiseringsuppgifter och distribuera sedan appen kamera.

1. Porter genererar en uppsättning autentiseringsuppgifter som automatiserar distributionen av appen. Innan du kör kommandot för att skapa autentiseringsuppgifter måste du ha följande tillgängligt:

    - Ett tjänst huvud namn för åtkomst till Azure-resurser, inklusive tjänstens huvud namns-ID, nyckel och klient-DNS.
    - Prenumerations-ID för din Azure-prenumeration.
    - Anslutnings strängen för avbildnings lagrings kontot som angavs när du distribuerade Cloud App.

1. Kör processen för generering av Porter-autentiseringsuppgifter och följ anvisningarna:

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Porter kräver också att en uppsättning parametrar körs. Skapa en parameter text fil och ange följande text. Be Azure Stack Hub-administratören om du inte vet några av de nödvändiga värdena.

    > [!NOTE]
    > `deployment suffix`Värdet används för att säkerställa att distributionens resurser har unika namn i Azure. Det måste vara en unik sträng med bokstäver och siffror, inte längre än 8 tecken.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    Spara text filen och anteckna sökvägen.

4. Nu är du redo att distribuera Camera-appen med Porter. Kör installations kommandot och se när IoT Edge distributionen har skapats.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. Kontrol lera att kamerans distribution är slutförd genom att visa kamerans feed på `https://<camera-ip>:3000/` , där `<camara-ip>` är kamerans IP-adress. Det här steget kan ta upp till 10 minuter.

## <a name="configure-azure-stream-analytics"></a>Konfigurera Azure Stream Analytics

Nu när data flödar till Azure Stream Analytics från kameran måste vi manuellt godkänna det för att kommunicera med Power BI.

1. Från Azure Portal öppnar du **alla resurser**och * \[ yoursuffix \] -jobbet för process Footfall* .

2. I avsnittet **Jobbtopologi** i Stream Analytics-jobbfönstret väljer du alternativet **Utdata**.

3. Välj Sink för utgående **trafik** .

4. Välj **förnya auktorisering** och logga in på ditt Power BI-konto.
  
    ![Förnya behörighets frågan i Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. Spara inställningarna för utdata.

6. Gå till **översikts** fönstret och välj **Starta** för att börja skicka data till Power BI.

7. Välj **Nu** som starttid för jobbutdata och välj **Start**. Du kan se dess status i meddelandefältet.

## <a name="create-a-power-bi-dashboard"></a>Skapa en instrument panel för Power BI

1. När jobbet har slutförts går du till [Power BI](https://powerbi.com/) och loggar in med ditt arbets-eller skol konto. Om Stream Analytics jobbets fråga resulterar i resultat, finns den data uppsättning för *Footfall-dataset* som du skapade under fliken **data uppsättningar** .

2. Från arbets ytan Power BI väljer du **+ skapa** för att skapa en ny instrument panel med namnet *Footfall analys.*

3. Välj **Lägg till panel** högst upp i fönstret. Välj sedan **Anpassade strömmande data** och **Nästa**. Välj **Footfall-dataset** under **dina data uppsättningar**. Välj **kort** i list rutan **typ av visualisering** och Lägg till **ålder** i **fält**. Välj **Nästa** för att ange ett namn på panelen och välj sedan **Applicera** för att skapa panelen.

4. Du kan lägga till ytterligare fält och kort som du vill.

## <a name="test-your-solution"></a>Testa din lösning

Observera hur data i korten som du skapade i Power BI ändras när olika personer går framför kameran. Det kan ta upp till 20 sekunder innan Inferences har registrerats.

## <a name="remove-your-solution"></a>Ta bort din lösning

Om du vill ta bort lösningen kör du följande kommandon med hjälp av Porter med samma parameter-filer som du skapade för distribution:

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>Nästa steg

- Läs mer om [design överväganden för Hybrid appar]. (overview-app-design-considerations.md)
- Granska och föreslå förbättringar av [koden för det här exemplet på GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).
