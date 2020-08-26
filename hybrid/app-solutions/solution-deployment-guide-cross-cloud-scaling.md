---
title: Distribuera en app som skalar över molnet i Azure och Azure Stack hubb
description: Lär dig hur du distribuerar en app som skalar över molnet i Azure och Azure Stack hubben.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886823"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Distribuera en app som skalar över molnet med Azure och Azure Stack hubb

Lär dig hur du skapar en lösning för flera moln för att tillhandahålla en manuellt utlöst process för att växla från en Azure Stack hubben webb program till en Azure-värdbaserad webbapp med automatisk skalning via Traffic Manager. Den här processen säkerställer flexibelt och skalbart moln verktyg vid belastning.

Med det här mönstret kanske inte klienten är redo att köra din app i det offentliga molnet. Det kanske inte är ekonomiskt genomförbart för verksamheten att bibehålla kapaciteten som krävs i den lokala miljön för att hantera toppar i efter frågan på appen. Din klient organisation kan utnyttja det offentliga molnets elastiskhet med sin lokala lösning.

I den här lösningen skapar du en exempel miljö för att:

> [!div class="checklist"]
> - Skapa en webbapp med flera noder.
> - Konfigurera och hantera processen för kontinuerlig distribution (CD).
> - Publicera webbappen till Azure Stack Hub.
> - Skapa en version.
> - Lär dig att övervaka och spåra dina distributioner.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling i din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

## <a name="prerequisites"></a>Förutsättningar

- En Azure-prenumeration. Om det behövs kan du skapa ett [kostnads fritt konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) innan du börjar.
- Ett Azure Stack hubb integrerat system eller distribution av Azure Stack Development Kit (ASDK).
  - Anvisningar om hur du installerar Azure Stack Hub finns i [Installera ASDK](/azure-stack/asdk/asdk-install.md).
  - För ett Automation-skript för ASDK efter distribution går du till: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - Den här installationen kan ta några timmar att slutföra.
- Distribuera [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS-tjänster till Azure Stack Hub.
- [Skapa planer/erbjudanden](/azure-stack/operator/service-plan-offer-subscription-overview.md) i Azure Stack Hub-miljön.
- [Skapa klient prenumeration](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) i Azure Stack Hub-miljön.
- Skapa en webbapp i klient prenumerationen. Anteckna den nya webb program-URL: en för senare användning.
- Distribuera virtuella datorer i Azure pipelines (VM) i klient prenumerationen.
- Windows Server 2016 VM med .NET 3,5 krävs. Den här virtuella datorn kommer att skapas i klient prenumerationen på Azure Stack Hub som den privata build-agenten.
- [Windows Server 2016 med SQL 2017 VM-avbildning](/azure-stack/operator/azure-stack-add-vm-image.md) finns på Azure Stack Hub Marketplace. Om avbildningen inte är tillgänglig arbetar du med en Azure Stack nav-operator för att se till att den läggs till i miljön.

## <a name="issues-and-considerations"></a>Problem och överväganden

### <a name="scalability"></a>Skalbarhet

Nyckel komponenten för skalning över moln är möjligheten att leverera omedelbar skalning på begäran mellan offentliga och lokala moln infrastrukturer, vilket ger konsekvent och tillförlitlig tjänst.

### <a name="availability"></a>Tillgänglighet

Se till att lokalt distribuerade appar är konfigurerade för hög tillgänglighet via lokal maskin varu konfiguration och program varu distribution.

### <a name="manageability"></a>Hanterbarhet

Lösningen över molnet säkerställer sömlös hantering och välbekant gränssnitt mellan miljöer. PowerShell rekommenderas för plattforms oberoende hantering.

## <a name="cross-cloud-scaling"></a>Skalning mellan moln

### <a name="get-a-custom-domain-and-configure-dns"></a>Hämta en anpassad domän och konfigurera DNS

Uppdatera DNS-zonfilen för domänen. Azure AD kommer att verifiera ägarskapet för det anpassade domän namnet. Använd [Azure DNS](/azure/dns/dns-getstarted-portal) för azure/Microsoft 365/external DNS-poster i Azure eller Lägg till DNS-posten på [en annan DNS-registrator](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).

1. Registrera en anpassad domän med en offentlig registrator.
2. Logga in hos domännamnsregistratorn för domänen. En godkänd administratör kan krävas för att göra DNS-uppdateringar.
3. Uppdatera DNS-zonfilen för domänen genom att lägga till DNS-posten från Azure AD. (DNS-posten påverkar inte e-postroutningen eller webb värd beteenden.)

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Skapa en standard-webbapp med flera noder i Azure Stack hubb

Konfigurera hybrid kontinuerlig integrering och kontinuerlig distribution (CI/CD) för att distribuera webbappar till Azure och Azure Stack hubb och för att skicka ändringar till båda molnen.

> [!Note]  
> Azure Stack hubben med rätt bilder som ska köras (Windows Server och SQL) och App Service distribution krävs. Mer information finns i krav för App Service dokumentation [för att distribuera app service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

### <a name="add-code-to-azure-repos"></a>Lägg till kod i Azure databaser

Azure-lagringsplatser

1. Logga in på Azure-databaser med ett konto som har behörighet att skapa projekt på Azure databaser.

    Hybrid CI/CD kan gälla både för appens kod och infrastruktur kod. Använd [Azure Resource Manager mallar](https://azure.microsoft.com/resources/templates/) för både privat och värdbaserad moln utveckling.

    ![Ansluta till ett projekt i Azure databaser](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Klona lagrings platsen** genom att skapa och öppna standard webb programmet.

    ![Klona lagrings platsen i Azure-webbappen](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Skapa en självständig distribution av webbappar för App Services i båda molnen

1. Redigera filen **WebApplication. CSPROJ** . Välj `Runtimeidentifier` och Lägg till `win10-x64` . (Mer information finns i dokumentationen för den [självständiga distributionen](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)

    ![Redigera projekt fil för webbapp](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Checka in koden till Azure databaser med hjälp av team Explorer.

3. Bekräfta att appens kod har marker ATS i Azure-databaser.

## <a name="create-the-build-definition"></a>Skapa build-definitionen

1. Logga in på Azure-pipelines för att bekräfta möjligheten att skapa Bygg definitioner.

2. Add **-r Win10-x64-** kod. Detta tillägg är nödvändigt för att utlösa en fristående distribution med .NET Core.

    ![Lägg till kod i webbappen](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Kör versionen. Den [fristående distributions](/dotnet/core/deploying/deploy-with-vs#simpleSelf) processen publicerar artefakter som körs på Azure och Azure Stack hubben.

## <a name="use-an-azure-hosted-agent"></a>Använd en Azure-värdbaserad agent

Det är ett bekvämt alternativ att bygga och distribuera webbappar med hjälp av en värdbaserad Bygg agent i Azure-pipeliner. Underhåll och uppgraderingar görs automatiskt av Microsoft Azure, vilket möjliggör en kontinuerlig och oavbruten utvecklings cykel.

### <a name="manage-and-configure-the-cd-process"></a>Hantera och konfigurera CD-processen

Azure-pipeliner och Azure DevOps Services erbjuder en mycket konfigurerbar och hanterbar pipeline för versioner till flera miljöer som utveckling, mellanlagring, frågor och produktions miljöer. inklusive krav på godkännanden i vissa steg.

## <a name="create-release-definition"></a>Skapa versions definition

1. Välj **plus** -knappen för att lägga till en ny version under fliken **utgåvor** i avsnittet **build och release** i Azure DevOps Services.

    ![Skapa en versionsdefinition](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Använd mallen för Azure App Service distribution.

   ![Använd mall för Azure App Service distribution](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. Lägg till artefakten för Azure Cloud build-appen under **Lägg till artefakt**.

   ![Lägg till artefakt i Azure Cloud build](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. Under fliken pipelines väljer du **fas, aktivitets** länk för miljön och anger värden för Azure Cloud-miljön.

   ![Ange värden för Azure Cloud-miljön](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Ange **miljö namnet** och välj Azure- **prenumeration** för Azure Cloud-slutpunkten.

      ![Välj Azure-prenumeration för Azures moln slut punkt](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. Under **App Service Name**anger du det obligatoriska namnet för Azure App Service.

      ![Ange namn på Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Ange "Hosted VS2017" under **agent kön** för Azure-molnets värd miljö.

      ![Ställ in agent kön för Azure-molnets värd miljö](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. I menyn Distribuera Azure App Service väljer du ett giltigt **paket eller** en giltig mapp för miljön. Välj **OK** för **mappens plats**.
  
      ![Välj paket eller mapp för Azure App Services miljö](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Välj paket eller mapp för Azure App Services miljö](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Spara alla ändringar och gå tillbaka till **versions pipelinen**.

    ![Spara ändringar i versions pipeline](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Lägg till en ny artefakt som väljer build för Azure Stack Hub-appen.

    ![Lägg till ny artefakt för Azure Stack Hub-appen](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Lägg till en miljö genom att använda Azure App Service distribution.

    ![Lägga till miljöer i Azure App Service distribution](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Namnge den nya miljön "Azure Stack".

    ![Namn miljö i Azure App Service distribution](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Hitta Azure Stacks miljön under fliken **aktivitet** .

    ![Azure Stack miljö](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Välj prenumerationen för Azure Stack slut punkten.

    ![Välj prenumerationen för Azure Stack slut punkten](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Ange Azure Stack webbappens namn som App Service-namn.
    ![Ange Azure Stack webb program namn](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Välj den Azure Stack agenten.

    ![Välj Azure Stack agent](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. Under avsnittet Distribuera Azure App Service väljer du ett giltigt **paket eller** en giltig mapp för miljön. Välj **OK** för mappens plats.

    ![Välj mapp för Azure App Service distribution](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Välj mapp för Azure App Service distribution](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. Under fliken variabel lägger du till en variabel med namnet `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , anger värdet **True**och scopet till Azure Stack.

    ![Lägg till variabel i Azure App distribution](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Välj ikonen för **kontinuerlig** distribution av utlösare i båda artefakterna och aktivera utlösaren **återuppta** distribution.

    ![Välj kontinuerlig distributions utlösare](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Välj ikonen för **för distributions** villkor i Azure Stacks miljön och Ställ in utlösaren på **efter version.**

    ![Välj för distributions villkor](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Spara alla ändringar.

> [!Note]  
> Vissa inställningar för aktiviteterna kan ha definierats automatiskt som [miljövariabler](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) när du skapar en versions definition från en mall. De här inställningarna kan inte ändras i aktivitets inställningarna. i stället måste den överordnade miljö posten väljas för att redigera de här inställningarna.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Publicera till Azure Stack hubb via Visual Studio

Genom att skapa slut punkter kan en Azure DevOps Services-version Distribuera Azure-tjänsteappar till Azure Stack hubben. Azure-pipeliner ansluter till build-agenten, som ansluter till Azure Stack Hub.

1. Logga in på Azure DevOps Services och gå till appens inställnings sida.

2. I **Inställningar**väljer du **säkerhet**.

3. I **VSTS-grupper**väljer du **slut punkts skapare**.

4. På fliken **medlemmar** väljer du **Lägg till**.

5. I **Lägg till användare och grupper anger du**ett användar namn och väljer användaren i listan över användare.

6. Välj **Spara ändringar**.

7. I listan **VSTS-grupper** väljer du **slut punkts administratörer**.

8. På fliken **medlemmar** väljer du **Lägg till**.

9. I **Lägg till användare och grupper anger du**ett användar namn och väljer användaren i listan över användare.

10. Välj **Spara ändringar**.

Nu när slut punkts informationen finns är Azure-pipelinen till Azure Stack Hub-anslutning redo att användas. Build-agenten i Azure Stack Hub hämtar instruktioner från Azure-pipelines och agenten förmedlar slut punkts informationen för kommunikation med Azure Stack Hub.

## <a name="develop-the-app-build"></a>Utveckla appens build

> [!Note]  
> Azure Stack hubben med rätt bilder som ska köras (Windows Server och SQL) och App Service distribution krävs. Mer information finns i [krav för distribution av app service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

Använd [Azure Resource Manager mallar](https://azure.microsoft.com/resources/templates/) som Web App Code från Azure databaser för att distribuera till båda molnen.

### <a name="add-code-to-an-azure-repos-project"></a>Lägga till kod i ett Azure databaser-projekt

1. Logga in på Azure-databaser med ett konto som har skapande rättigheter för projekt på Azure Stack Hub.

2. **Klona lagrings platsen** genom att skapa och öppna standard webb programmet.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Skapa en självständig distribution av webbappar för App Services i båda molnen

1. Redigera filen **WebApplication. CSPROJ** : Välj `Runtimeidentifier` och Lägg sedan till `win10-x64` . Mer information finns i dokumentationen för [självständiga distributioner](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .

2. Använd Team Explorer för att kontrol lera koden i Azure databaser.

3. Bekräfta att appens kod har marker ATS i Azure-databaser.

### <a name="create-the-build-definition"></a>Skapa build-definitionen

1. Logga in på Azure-pipelines med ett konto som kan skapa en build-definition.

2. Gå till sidan för att **bygga webb program** för projektet.

3. I **argument**, Add **-r Win10-x64** Code. Detta tillägg krävs för att utlösa en fristående distribution med .NET Core.

4. Kör versionen. Den [fristående distributions](/dotnet/core/deploying/deploy-with-vs#simpleSelf) processen publicerar artefakter som kan köras på Azure och Azure Stack hubben.

#### <a name="use-an-azure-hosted-build-agent"></a>Använd en Azure Hosted build-agent

Det är ett bekvämt alternativ att bygga och distribuera webbappar med hjälp av en värdbaserad Bygg agent i Azure-pipeliner. Underhåll och uppgraderingar görs automatiskt av Microsoft Azure, vilket möjliggör en kontinuerlig och oavbruten utvecklings cykel.

### <a name="configure-the-continuous-deployment-cd-process"></a>Konfigurera processen för kontinuerlig distribution (CD)

Azure-pipeliner och Azure DevOps Services erbjuder en mycket konfigurerbar och hanterbar pipeline för versioner till flera miljöer som utveckling, mellanlagring, kvalitets säkring (frågor och svar) och produktion. Den här processen kan omfatta krav på godkännanden i vissa faser i appens livs cykel.

#### <a name="create-release-definition"></a>Skapa versions definition

Att skapa en versions definition är det sista steget i bygg processen för appar. Denna versions definition används för att skapa en version och distribuera en version.

1. Logga in på Azure-pipelines och gå till **version och lansering** för projektet.

2. På fliken **utgåvor** väljer du **[+]** och väljer sedan **skapa versions definition**.

3. Välj **Azure App service distribution**på **Välj en mall**och välj sedan **Använd**.

4. Välj Azure Cloud build-appen från **källan (build definition)** på **Lägg till artefakt**.

5. På fliken **pipelines** väljer du länken **1 fas**, **1 aktivitet** , för att **Visa miljö aktiviteter**.

6. På fliken **uppgifter** anger du Azure som **miljö namn** och väljer AzureCloud Traders – Web EP från listan Azure- **prenumeration** .

7. Ange **namnet på Azure App Service**, som finns `northwindtraders` i nästa skärmdump.

8. För agent fasen väljer du **VÄRDBASERAD VS2017** i listan **agent kö** .

9. I **distribuera Azure App Service**väljer du ett giltigt **paket eller** en giltig mapp för miljön.

10. I **Välj fil eller mapp**väljer du **OK** till **plats**.

11. Spara alla ändringar och gå tillbaka till **pipelinen**.

12. På fliken **pipelines** väljer du **Lägg till artefakt**och väljer **NorthwindCloud Traders-fartyget** från käll listan **(build definition)** .

13. Lägg till en annan miljö på **Välj en mall**. Välj **Azure App service distribution** och välj sedan **Använd**.

14. Ange `Azure Stack Hub` som **miljö namn**.

15. På fliken **aktiviteter** , leta upp och välj Azure Stack Hub.

16. I listan **Azure-prenumeration** väljer du **AzureStack Traders – fartyg EP** för Azure Stack Hub-slutpunkten.

17. Ange Azure Stack Hub-webbappens namn som **App Service-** namn.

18. Under **agent val**väljer du **AzureStack-b Douglas FIR** från listan **agent kö** .

19. För **distribuera Azure App Service**väljer du ett giltigt **paket eller** en giltig mapp för miljön. På sidan **Välj fil eller mapp**väljer du **OK** för mappens **plats**.

20. På fliken **variabel** letar du upp variabeln med namnet `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` . Ange värdet **True**för variabeln och ange dess omfång till **Azure Stack Hub**.

21. På fliken **pipelines** väljer du ikonen för **kontinuerlig distribution av utlösare** för NorthwindCloud Traders – webb artefakt och ställer in den **kontinuerliga distributions utlösaren** på **aktive rad**. Gör samma sak för **NorthwindCloud Traders-fartygets** artefakt.

22. För Azure Stack Hub-miljö väljer du ikonen för **för distributions villkor** ange utlösaren till **efter versionen**.

23. Spara alla ändringar.

> [!Note]  
> Vissa inställningar för versions aktiviteter definieras automatiskt som [miljövariabler](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) när du skapar en versions definition från en mall. De här inställningarna kan inte ändras i aktivitets inställningarna men kan ändras i de överordnade miljö objekten.

## <a name="create-a-release"></a>Skapa en version

1. På fliken **pipeline** öppnar du listan **version** och väljer **Skapa version**.

2. Ange en beskrivning av versionen, kontrol lera att rätt artefakter har valts och välj sedan **skapa**. Efter en liten stund visas en banderoll som anger att den nya versionen skapades och att versions namnet visas som en länk. Välj länken för att se sammanfattnings sidan för utgåvor.

3. Sidan versions Sammanfattning visar information om versionen. I följande skärm bild för "Release-2" visar avsnittet **miljöer** **distributions statusen** för Azure som "pågår" och statusen för Azure Stack Hub är "lyckades". När distributions statusen för Azure-miljön ändras till "lyckades" visas en banderoll som anger att versionen är klar för godkännande. När en distribution väntar eller har misslyckats visas en blå **(i)** informations ikon. Hovra över ikonen för att se ett popup-fönster som innehåller orsaken till fördröjningen eller haveriet.

4. Andra vyer, till exempel listan över versioner, visar också en ikon som indikerar att godkännande väntar. Popup-fönstret för den här ikonen visar miljö namnet och mer information om distributionen. Det är enkelt för en administratör att se det övergripande förloppet för versioner och se vilka versioner som väntar på godkännande.

## <a name="monitor-and-track-deployments"></a>Övervaka och spåra distributioner

1. På sammanfattnings sidan för **Release 2** väljer du **loggar**. Under en distribution visar den här sidan Live-loggen från agenten. Den vänstra rutan visar status för varje åtgärd i distributionen för varje miljö.

2. Välj person ikonen i kolumnen **åtgärd** för ett godkännande före distribution eller efter distribution för att se vem som har godkänt (eller avvisat) distributionen och meddelandet de tillhandahöll.

3. När distributionen är klar visas hela logg filen i den högra rutan. Välj något av **stegen** i det vänstra fönstret för att se logg filen för ett enda steg, t. ex. **initiera jobb**. Möjligheten att se enskilda loggar gör det lättare att spåra och felsöka delar av den övergripande distributionen. **Spara** logg filen för ett steg eller **Ladda ned alla loggar som zip**.

4. Öppna fliken **Sammanfattning** om du vill se allmän information om versionen. Den här vyn visar information om bygget, de miljöer som den distribuerades till, distributions status och annan information om versionen.

5. Välj en miljö länk (**Azure** eller **Azure Stack Hub**) om du vill se information om befintliga och väntande distributioner till en speciell miljö. Använd dessa vyer som ett snabbt sätt att kontrol lera att samma version har distribuerats i båda miljöerna.

6. Öppna den **distribuerade produktions programmet** i en webbläsare. För webbplatsen för Azure App tjänster öppnar du till exempel URL: en `https://[your-app-name\].azurewebsites.net` .

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>Integrering av Azure och Azure Stack Hub ger en skalbar lösning för kors molnet

En flexibel och robust tjänst för flera moln ger data säkerhet, säkerhets kopiering och redundans, konsekvent och snabb tillgänglighet, skalbar lagring och distribution och geo-kompatibel routning. Den manuellt utlösta processen garanterar en tillförlitlig och effektiv belastnings växling mellan värdbaserade webbappar och omedelbar tillgänglighet för viktiga data.

## <a name="next-steps"></a>Nästa steg

- Mer information om moln mönster i Azure finns i [design mönster för molnet](/azure/architecture/patterns).
