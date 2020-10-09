---
title: Distribuera en MongoDB-lösning med hög tillgänglighet till Azure och Azure Stack hubb
description: Lär dig hur du distribuerar en MongoDB-lösning med hög tillgänglighet till Azure och Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852515"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Distribuera en MongoDB-lösning med hög tillgänglighet till Azure och Azure Stack hubb

Den här artikeln går igenom en automatiserad distribution av ett MongoDB-kluster med hög tillgänglighet (HA) med en katastrof återställning (DR) på två Azure Stack Hubbs miljöer. Mer information om MongoDB och hög tillgänglighet finns i [replik uppsättnings medlemmar](https://docs.mongodb.com/manual/core/replica-set-members/).

I den här lösningen skapar du en exempel miljö för att:

> [!div class="checklist"]
> - Dirigera en distribution mellan två Azure Stack hubbar.
> - Använd Docker för att minimera beroende problem med Azure API-profiler.
> - Distribuera ett Basic MongoDB-kluster med hög tillgänglighet med en katastrof återställnings plats.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling i din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Arkitektur för MongoDB med Azure Stack Hub

![MongoDB-arkitektur med hög tillgänglighet i Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Krav för MongoDB med Azure Stack Hub

- Två anslutna Azure Stack Hub-integrerade system (Azure Stack Hub). Den här distributionen fungerar inte på Azure Stack Development Kit (ASDK). Läs mer om Azure Stack Hub i [Vad är Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)
  - En klient prenumeration på varje Azure Stack hubb. 
  - **Anteckna varje prenumerations-ID och Azure Resource Manager slut punkten för varje Azure Stack Hub.**
- Ett Azure Active Directory (Azure AD)-tjänstens huvud namn som har behörighet till klient prenumerationen på varje Azure Stack hubb. Du kan behöva skapa två huvud namn för tjänsten om Azure Stack hubbar distribueras mot olika Azure AD-klienter. Information om hur du skapar ett huvud namn för tjänsten för Azure Stack hubb finns i [använda en app-identitet för att få åtkomst till Azure Stack Hub-resurser](/azure-stack/user/azure-stack-create-service-principals).
  - **Anteckna varje tjänst objekts program-ID, klient hemlighet och klient namn (xxxxx.onmicrosoft.com).**
- Ubuntu 16,04 har syndikerats till varje Azure Stack Hubbs marknads plats. Läs mer om Marketplace-syndikering i [Hämta Marketplace-objekt till Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- [Docker för Windows](https://docs.docker.com/docker-for-windows/) installerat på den lokala datorn.

## <a name="get-the-docker-image"></a>Hämta Docker-avbildningen

Docker-avbildningar för varje distribution eliminerar beroende problem mellan olika versioner av Azure PowerShell.

1. Se till att Docker för Windows använder Windows-behållare.
2. Kör följande kommando i en upphöjd kommando tolk för att hämta Docker-behållaren med distributions skripten.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Distribuera klustren

1. Starta avbildningen när behållar avbildningen har tagits emot.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. När behållaren har startats får du en upphöjd PowerShell-Terminal i behållaren. Ändra kataloger för att komma till distributions skriptet.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Kör distributionen. Ange autentiseringsuppgifter och resurs namn där det behövs. HA refererar till Azure Stack hubben där HA-klustret ska distribueras. DR refererar till Azure Stack hubben där DR-klustret ska distribueras.

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

4. Skriv `Y` för att tillåta att NuGet-providern installeras, vilket kommer att starta API-profilen "2018-03-01-hybrid"-moduler som ska installeras.

5. Resurserna för att distribueras först. Övervaka distributionen och vänta tills den är klar. När du har fått meddelandet om att HA-distributionen är färdig kan du kontrol lera att de resurser som distribueras har distribuerats med HA Azure Stack Hub-portalen.

6. Fortsätt med distributionen av DR-resurser och bestäm om du vill aktivera en hopp ruta i DR Azure Stack Hub för att interagera med klustret.

7. Vänta tills återställnings resurs distributionen är klar.

8. När återställningen av en resurs distribution har slutförts avslutar du behållaren.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Nästa steg

- Om du har aktiverat den virtuella hopp rutan i DR Azure Stack Hub kan du ansluta via SSH och interagera med MongoDB-klustret genom att installera Mongo CLI. Mer information om hur du interagerar med MongoDB finns i [Mongo-gränssnittet](https://docs.mongodb.com/manual/mongo/).
- Mer information om hybrid molnappar finns i [hybrid moln lösningar.](/azure-stack/user/)
- Ändra koden till det här exemplet på [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).