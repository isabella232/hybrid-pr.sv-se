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
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Distribuera en SQL Server 2016-tillgänglighets grupp till Azure och Azure Stack Hub

I den här artikeln får du stegvisa anvisningar genom en automatiserad distribution av ett grundläggande (HA) SQL Server 2016 Enterprise-kluster med en asynkron haveri beredskap (DR) i två Azure Stack Hub-miljöer. Mer information om SQL Server 2016 och hög tillgänglighet finns i [Always on-tillgänglighets grupper: en lösning för hög tillgänglighet och katastrof återställning](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).

I den här lösningen skapar du en exempel miljö för att:

> [!div class="checklist"]
> - Dirigera en distribution mellan två Azure Stack hubbar.
> - Använd Docker för att minimera beroende problem med Azure API-profiler.
> - Distribuera ett grundläggande SQL Server 2016 Enterprise-kluster med en katastrof återställnings plats.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling i din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

## <a name="architecture-for-sql-server-2016"></a>Arkitektur för SQL Server 2016

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Krav för SQL Server 2016

- Två anslutna Azure Stack Hub-integrerade system (Azure Stack Hub). Den här distributionen fungerar inte på Azure Stack Development Kit (ASDK). Mer information om Azure Stack Hub finns i [Azure Stack översikt](https://azure.microsoft.com/overview/azure-stack/).
- En klient prenumeration på varje Azure Stack hubb.
  - **Anteckna varje prenumerations-ID och Azure Resource Manager slut punkten för varje Azure Stack Hub.**
- Ett Azure Active Directory (Azure AD)-tjänstens huvud namn som har behörighet till klient prenumerationen på varje Azure Stack hubb. Du kan behöva skapa två huvud namn för tjänsten om Azure Stack hubbar distribueras mot olika Azure AD-klienter. Information om hur du skapar ett huvud namn för tjänsten för Azure Stack hubb finns i [skapa tjänstens huvud namn för att ge appar åtkomst till Azure Stack Hub-resurser](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).
  - **Anteckna varje tjänst objekts program-ID, klient hemlighet och klient namn (xxxxx.onmicrosoft.com).**
- SQL Server 2016 Enterprise syndikerat till varje Azure Stack Hubbs marknads plats. Läs mer om Marketplace-syndikering i [Hämta Marketplace-objekt till Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).
    **Kontrol lera att din organisation har rätt SQL-licenser.**
- [Docker för Windows](https://docs.docker.com/docker-for-windows/) installerat på den lokala datorn.

## <a name="get-the-docker-image"></a>Hämta Docker-avbildningen

Docker-avbildningar för varje distribution eliminerar beroende problem mellan olika versioner av Azure PowerShell.

1. Se till att Docker för Windows använder Windows-behållare.
2. Kör följande skript i en upphöjd kommando tolk för att hämta Docker-behållaren med distributions skripten.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Distribuera tillgänglighets gruppen

1. Starta avbildningen när behållar avbildningen har tagits emot.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. När behållaren har startats får du en upphöjd PowerShell-Terminal i behållaren. Ändra kataloger för att komma till distributions skriptet.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Kör distributionen. Ange autentiseringsuppgifter och resurs namn där det behövs. HA refererar till Azure Stack hubben där HA-klustret ska distribueras. DR refererar till Azure Stack hubben där DR-klustret ska distribueras.

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

4. Skriv `Y` för att tillåta att NuGet-providern installeras, vilket kommer att starta API-profilen "2018-03-01-hybrid"-moduler som ska installeras.

5. Vänta tills resurs distributionen har slutförts.

6. När du har slutfört återställningen av DR-resursen avslutar du behållaren.

      ```powershell
      exit
      ```

7. Granska distributionen genom att Visa resurserna i varje Azure Stack Hubbs Portal. Anslut till en av SQL-instanserna i HA-miljön och granska tillgänglighets gruppen via SQL Server Management Studio (SSMS).

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Nästa steg

- Använd SQL Server Management Studio för att redundansväxla klustret manuellt. Se [utföra en framtvingad manuell redundansväxling av en tillgänglighets grupp som alltid är tillgänglig (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- Läs mer om hybrid molnappar. Se [hybrid moln lösningar.](https://aka.ms/azsdevtutorials)
- Använd dina egna data eller ändra koden till det här exemplet på [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
