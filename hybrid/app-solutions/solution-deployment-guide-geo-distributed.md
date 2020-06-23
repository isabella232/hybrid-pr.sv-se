---
title: Dirigera trafik med en geo-distribuerad app med Azure och Azure Stack hubb
description: Lär dig hur du dirigerar trafik till vissa slut punkter med en geo-distribuerad app-lösning med hjälp av Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912045"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Dirigera trafik med en geo-distribuerad app med Azure och Azure Stack hubb

Lär dig hur du dirigerar trafik till vissa slut punkter baserat på olika mått med hjälp av mönstret för geo-distribuerade appar. Genom att skapa en Traffic Manager profil med geografisk Routning och slut punkts konfiguration ser du till att information dirigeras till slut punkter baserat på regionala krav, företags-och internationella regler och dina data behov.

I den här lösningen skapar du en exempel miljö för att:

> [!div class="checklist"]
> - Skapa en geo-distribuerad app.
> - Använd Traffic Manager för att rikta din app.

## <a name="use-the-geo-distributed-apps-pattern"></a>Använd mönstret för geo-distribuerade appar

Med det geo-distribuerade mönstret omfattar din app regioner. Du kan använda det offentliga molnet som standard, men vissa av användarna kan kräva att deras data finns kvar i sin region. Du kan dirigera användare till det mest lämpliga molnet baserat på deras krav.

### <a name="issues-and-considerations"></a>Problem och överväganden

#### <a name="scalability-considerations"></a>Skalbarhetsöverväganden

Lösningen som du skapar med den här artikeln är inte att hantera skalbarhet. Men om det används i kombination med andra Azure-lösningar och lokala lösningar, kan du hantera skalbarhets krav. Information om hur du skapar en hybrid lösning med automatisk skalning via Traffic Manager finns i [skapa lösningar för skalning av globala moln med Azure](solution-deployment-guide-cross-cloud-scaling.md).

#### <a name="availability-considerations"></a>Överväganden för tillgänglighet

När det gäller skalbarhet är den här lösningen inte direkt tillgänglig. Men Azure och lokala lösningar kan implementeras i den här lösningen för att säkerställa hög tillgänglighet för alla komponenter som ingår.

### <a name="when-to-use-this-pattern"></a>När du ska använda det här mönstret

- Din organisation har internationella grenar som kräver anpassade regionala säkerhets-och distributions principer.

- Var och en av organisationens kontor hämtar personal-, affärs-och anläggnings data, som kräver rapporterings aktivitet enligt lokala förordningar och tids zoner.

- Storskaliga krav uppfylls genom att skala ut appar vågrätt med flera distributioner av appar inom en enda region och mellan regioner för att hantera extrema belastnings krav.

### <a name="planning-the-topology"></a>Planera topologin

Innan du skapar en distribuerad app kan du känna till följande saker:

- **Anpassad domän för appen:** Vad är det anpassade domän namnet som kunder kommer att använda för att få åtkomst till appen? För exempel appen är det anpassade domän namnet www- * \. scalableasedemo.com.*

- **Traffic Manager domän:** Du väljer ett domän namn när du skapar en [Azure Traffic Manager-profil](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles). Det här namnet kombineras med *trafficmanager.net* -suffixet för att registrera en domän post som hanteras av Traffic Manager. För exempel appen är det valda namnet *skalbart-ASE-demo*. Därför är det fullständiga domän namnet som hanteras av Traffic Manager *Scalable-ASE-demo.trafficmanager.net*.

- **Strategi för skalning av appens avtryck:** Bestäm om appens utrymme ska distribueras över flera App Service miljöer i en enda region, flera regioner eller en blandning av båda metoderna. Beslutet bör baseras på förväntningar av var kund trafiken kommer att slutföras och hur väl resten av en Apps stödjande backend-infrastruktur kan skalas. Till exempel, med en tillstånds lös app på 100%, kan en app skalas enorma med en kombination av flera App Service miljöer per Azure-region, multiplicerat med App Service miljöer som distribuerats över flera Azure-regioner. Med 15 globala Azure-regioner som är tillgängliga för att välja bland kan kunder verkligen bygga en världs omfattande storskalig app. För exempel programmet som används här har tre App Service miljöer skapats i en enda Azure-region (södra centrala USA).

- **Namngivnings konvention för App Service miljöer:** Varje App Service miljö kräver ett unikt namn. Utöver en eller två App Service miljöer är det bra att ha en namngivnings konvention som hjälper dig att identifiera varje App Service miljö. För den exempel app som används här användes en enkel namngivnings konvention. Namnen på de tre App Services miljöerna är *fe1ase*, *fe2ase*och *fe3ase*.

- **Namngivnings konvention för apparna:** Eftersom flera instanser av appen kommer att distribueras krävs ett namn för varje instans av den distribuerade appen. Med App Service-miljön för Power Apps kan samma app-namn användas i flera miljöer. Eftersom varje App Service miljö har ett unikt domänsuffix kan utvecklare välja att återanvända exakt samma app-namn i varje miljö. En utvecklare kan till exempel ha appar som heter enligt följande: *MyApp.foo1.p.azurewebsites.net*, *MyApp.foo2.p.azurewebsites.net*, *MyApp.foo3.p.azurewebsites.net*och så vidare. För den app som används här har varje App-instans ett unikt namn. De instans namn som används för appar är *webfrontend1*, *webfrontend2*och *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

## <a name="part-1-create-a-geo-distributed-app"></a>Del 1: skapa en geo-distribuerad app

I den här delen skapar du en webbapp.

> [!div class="checklist"]
> - Skapa webbappar och publicera.
> - Lägg till kod i Azure databaser.
> - Peka appen Bygg till flera moln mål.
> - Hantera och konfigurera CD-processen.

### <a name="prerequisites"></a>Förutsättningar

En Azure-prenumeration och Azure Stack Hub-installation krävs.

### <a name="geo-distributed-app-steps"></a>Steg för geo-distribuerad app

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Skaffa en anpassad domän och konfigurera DNS

Uppdatera DNS-zonfilen för domänen. Azure AD kan sedan verifiera ägarskapet för det anpassade domän namnet. Använd [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) för Azure/Office 365/externa DNS-poster i Azure eller Lägg till DNS-posten på [en annan DNS-registrator](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Registrera en anpassad domän med en offentlig registrator.

2. Logga in hos domännamnsregistratorn för domänen. En godkänd administratör kan krävas för att göra DNS-uppdateringar.

3. Uppdatera DNS-zonfilen för domänen genom att lägga till DNS-posten från Azure AD. DNS-posten ändrar inte beteenden, till exempel e-postroutning eller webb värd.

### <a name="create-web-apps-and-publish"></a>Skapa webbappar och publicera

Konfigurera hybrid kontinuerlig integrering/kontinuerlig leverans (CI/CD) för att distribuera webbappen till Azure och Azure Stack hubb och skicka automatiskt ändringar till båda molnen.

> [!Note]  
> Azure Stack hubben med rätt bilder som ska köras (Windows Server och SQL) och App Service distribution krävs. Mer information finns i [krav för distribution av app service på Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

#### <a name="add-code-to-azure-repos"></a>Lägg till kod i Azure databaser

1. Logga in i Visual Studio med ett **konto som har projekt skapande rättigheter** på Azure-databaser.

    CI/CD kan gälla både för appens kod och infrastruktur kod. Använd [Azure Resource Manager mallar](https://azure.microsoft.com/resources/templates/) för både privat och värdbaserad moln utveckling.

    ![Ansluta till ett projekt i Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. **Klona lagrings platsen** genom att skapa och öppna standard webb programmet.

    ![Klona lagrings platser i Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Skapa webb program distribution i båda molnen

1. Redigera filen **WebApplication. CSPROJ** : Välj `Runtimeidentifier` och Lägg till `win10-x64` . (Mer information finns i dokumentationen för den [självständiga distributionen](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)

    ![Redigera webb program projekt filen i Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. **Checka in koden till Azure databaser** med hjälp av team Explorer.

3. Bekräfta att **program koden** har checkats in i Azure-databaser.

### <a name="create-the-build-definition"></a>Skapa build-definitionen

1. **Logga in på Azure-pipelines** för att bekräfta möjligheten att skapa Bygg definitioner.

2. Lägg till `-r win10-x64` kod. Detta tillägg är nödvändigt för att utlösa en fristående distribution med .NET Core.

    ![Lägga till kod i build-definitionen i Azure-pipeline](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Kör versionen**. Den [fristående distributions](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) processen publicerar artefakter som kan köras på Azure och Azure Stack hubben.

#### <a name="using-an-azure-hosted-agent"></a>Använda en Azure-värdbaserad agent

Det är ett bekvämt alternativ att bygga och distribuera webbappar med hjälp av en värdbaserad agent i Azure-pipeliner. Underhåll och uppgraderingar utförs automatiskt av Microsoft Azure, vilket möjliggör oavbruten utveckling, testning och distribution.

### <a name="manage-and-configure-the-cd-process"></a>Hantera och konfigurera CD-processen

Azure DevOps Services erbjuder en mycket konfigurerbar och hanterbar pipeline för versioner till flera miljöer, till exempel utvecklings-, mellanlagrings-, frågor och produktions miljöer. inklusive krav på godkännanden i vissa steg.

## <a name="create-release-definition"></a>Skapa versions definition

1. Välj **plus** -knappen för att lägga till en ny version under fliken **utgåvor** i avsnittet **build och release** i Azure DevOps Services.

    ![Skapa en versions definition i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. Använd mallen för Azure App Service distribution.

   ![Använd mallen för Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. Lägg till artefakten för Azure Cloud build-appen under **Lägg till artefakt**.

   ![Lägg till artefakt i Azure Cloud build i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. Under fliken pipelines väljer du **fas, aktivitets** länk för miljön och anger värden för Azure Cloud-miljön.

   ![Ange Azure-molnets miljö värden i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. Ange **miljö namnet** och välj Azure- **prenumeration** för Azure Cloud-slutpunkten.

      ![Välj Azure-prenumeration för Azure Cloud-slutpunkt i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. Under **App Service Name**anger du det obligatoriska namnet för Azure App Service.

      ![Ange namn på Azure App Service i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. Ange "Hosted VS2017" under **agent kön** för Azure-molnets värd miljö.

      ![Ställ in agent kön för Azure-molnets värd miljö i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. I menyn Distribuera Azure App Service väljer du ett giltigt **paket eller** en giltig mapp för miljön. Välj **OK** för **mappens plats**.
  
      ![Välj paket eller mapp för Azure App Services miljö i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Välj paket eller mapp för Azure App Services miljö i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. Spara alla ändringar och gå tillbaka till **versions pipelinen**.

    ![Spara ändringar i versions pipeline i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. Lägg till en ny artefakt som väljer build för Azure Stack Hub-appen.

    ![Lägg till ny artefakt för Azure Stack Hub-app i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. Lägg till en miljö genom att använda Azure App Service distribution.

    ![Lägga till miljöer i Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. Namnge den nya miljön Azure Stack Hub.

    ![Namn miljö i Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. Hitta Azure Stack Hub-miljön under fliken **aktivitet** .

    ![Azure Stack Hub-miljö i Azure DevOps Services i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. Välj prenumerationen för Azure Stack Hub-slutpunkten.

    ![Välj prenumerationen för Azure Stack Hub-slutpunkten i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. Ange namnet på den Azure Stack Hub-webbappen som App Service-namn.

    ![Ange Azure Stack Hub-webbappens namn i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. Välj Azure Stack Hub-agenten.

    ![Välj Azure Stack Hub-agenten i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. Under avsnittet Distribuera Azure App Service väljer du ett giltigt **paket eller** en giltig mapp för miljön. Välj **OK** för mappens plats.

    ![Välj mapp för Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Välj mapp för Azure App Service distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. Under fliken variabel lägger du till en variabel med namnet `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , anger värdet **True**och scope till Azure Stack Hub.

    ![Lägg till variabel till Azure App distribution i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. Välj ikonen för **kontinuerlig** distribution av utlösare i båda artefakterna och aktivera utlösaren **återuppta** distribution.

    ![Välj kontinuerlig distributions utlösare i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. Välj ikonen för **för distributions** villkor i Azure Stack Hub-miljön och Ställ in utlösaren på **efter version.**

    ![Välj för distributions villkor i Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. Spara alla ändringar.

> [!Note]  
> Vissa inställningar för aktiviteterna kan ha definierats automatiskt som [miljövariabler](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) när du skapar en versions definition från en mall. De här inställningarna kan inte ändras i aktivitets inställningarna. i stället måste den överordnade miljö posten väljas för att redigera de här inställningarna.

## <a name="part-2-update-web-app-options"></a>Del 2: uppdatera webb programs alternativ

[Azure App Service](https://docs.microsoft.com/azure/app-service/overview) ger en mycket skalbar och automatisk korrigering av webb värd tjänst.

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Mappa ett befintligt anpassat DNS-namn till Azure Web Apps.
> - Använd en **CNAME-post** och en **a-post** för att mappa ett anpassat DNS-namn till App Service.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Mappa ett befintligt anpassat DNS-namn till Azure Web Apps

> [!Note]  
> Använd en CNAME för alla anpassade DNS-namn förutom en rot domän (till exempel northwind.com).

Om du vill migrera en live-webbplats och dess DNS-domännamn till App Service kan du läsa [Migrera ett aktivt DNS-namn till Azure App Service](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).

### <a name="prerequisites"></a>Förutsättningar

För att slutföra den här lösningen:

- [Skapa en app service app](https://docs.microsoft.com/azure/app-service/)eller Använd en app som skapats för en annan lösning.

- Köp ett domän namn och se till att du har åtkomst till DNS-registret för domän leverantören.

Uppdatera DNS-zonfilen för domänen. Azure AD kommer att verifiera ägarskapet för det anpassade domän namnet. Använd [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) för Azure/Office 365/externa DNS-poster i Azure eller Lägg till DNS-posten på [en annan DNS-registrator](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

- Registrera en anpassad domän med en offentlig registrator.

- Logga in hos domännamnsregistratorn för domänen. (En godkänd administratör kan krävas för att göra DNS-uppdateringar.)

- Uppdatera DNS-zonfilen för domänen genom att lägga till DNS-posten från Azure AD.

Om du till exempel vill lägga till DNS-poster för northwindcloud.com och www \. -northwindcloud.com konfigurerar du DNS-inställningarna för rot domänen northwindcloud.com.

> [!Note]  
> Du kan köpa ett domän namn med hjälp av [Azure Portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain). För att kunna mappa ett anpassat DNS-namn till en webbapp måste webbappens [App Service-plan](https://azure.microsoft.com/pricing/details/app-service/) vara en betalplan (**Delad**, **Basic**, **Standard** eller **Premium**).

### <a name="create-and-map-cname-and-a-records"></a>Skapa och mappa CNAME-och A-poster

#### <a name="access-dns-records-with-domain-provider"></a>Använda DNS-poster med domänleverantör

> [!Note]  
>  Använd Azure DNS för att konfigurera ett anpassat DNS-namn för Azure Web Apps. Mer information finns i [Använda Azure DNS för att skapa inställningar för anpassad domän för en Azure-tjänst](https://docs.microsoft.com/azure/dns/dns-custom-domain).

1. Logga in på webbplatsen för huvud-providern.

2. Sök upp sidan för hantering av DNS-poster. Varje domän leverantör har sitt eget gränssnitt för DNS-poster. Leta efter områden på webbplatsen med namnet **Domännamn**, **DNS**, eller **Namnserverhantering**.

Sidan DNS-poster kan visas i **Mina domäner**. Hitta länken som heter **zonfilen**, **DNS-poster**eller **Avancerad konfiguration**.

Skärmbilden nedan är ett exempel på en sida med DNS-poster:

![Exempelsida för DNS-poster](media/solution-deployment-guide-geo-distributed/image28.png)

1. I domän namn registrator väljer du **Lägg till eller skapa** för att skapa en post. Vissa providrar har olika länkar för att lägga till olika posttyper. Läs leverantörens dokumentation.

2. Lägg till en CNAME-post för att mappa en under domän till appens standard värd namn.

   För www \. northwindcloud.com-domänens exempel lägger du till en CNAME-post som mappar namnet till `<app_name>.azurewebsites.net` .

När du har lagt till CNAME ser sidan DNS-poster ut som i följande exempel:

![Portalnavigering till Azure-app](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Aktivera CNAME-postmappning i Azure

1. Logga in på Azure Portal på en ny flik.

2. Gå till App Services.

3. Välj webbapp.

4. Välj **Anpassade domäner** i det vänstra navigeringsfönstret på appsidan i Azure Portal.

5. Välj **+** ikonen bredvid **Lägg till värdnamn**.

6. Skriv det fullständigt kvalificerade domän namnet, t `www.northwindcloud.com` . ex..

7. Välj **Verifiera**.

8. Om det här alternativet anges lägger du till ytterligare poster av andra typer ( `A` eller `TXT` ) till domän namn registrerar DNS-poster. Azure kommer att tillhandahålla värden och typer för dessa poster:

   a.  En **A**-post för att mappa till appens IP-adress.

   b.  En **TXT**-post att mappa till appens standardvärdnamn `<app_name>.azurewebsites.net`. App Service använder bara den här posten vid konfigurations tiden för att verifiera den anpassade domän ägarskapet. Efter verifieringen tar du bort TXT-posten.

9. Slutför den här uppgiften på fliken domän registrator och verifiera igen tills knappen **Lägg till värdnamn** är aktive rad.

10. Se till att **post typen hostname** är inställd på **CNAME** (www.example.com eller någon under domän).

11. Välj **Lägg till värddatornamn**.

12. Skriv det fullständigt kvalificerade domän namnet, t `northwindcloud.com` . ex..

13. Välj **Verifiera**. **Lägg till** aktive rad.

14. Se till att **post typen hostname** är inställd på **en post** (example.com).

15. **Lägg till värdnamn**.

    Det kan ta lite tid innan de nya värdarna visas på sidan **anpassade domäner** för appen. Försök att uppdatera webbläsaren så att informationen uppdateras.
  
    ![Anpassade domäner](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Om ett fel uppstår visas ett meddelande om verifierings fel längst ned på sidan. ![Domän verifierings fel](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  Ovanstående steg kan upprepas för att mappa en domän med jokertecken ( \* . northwindcloud.com). Detta gör att du kan lägga till ytterligare under domäner till den här app service utan att behöva skapa en separat CNAME-post för var och en. Följ anvisningarna i registratorn för att konfigurera den här inställningen.

#### <a name="test-in-a-browser"></a>Testa i en webbläsare

Bläddra till det eller de DNS-namn som kon figurer ATS tidigare (till exempel `northwindcloud.com` eller `www.northwindcloud.com` ).

## <a name="part-3-bind-a-custom-ssl-cert"></a>Del 3: binda ett anpassat SSL-certifikat

I den här delen kommer vi att:

> [!div class="checklist"]
> - Bind det anpassade SSL-certifikatet till App Service.
> - Använd HTTPS för appen.
> - Automatisera SSL-certifikat bindning med skript.

> [!Note]  
> Om det behövs kan du skaffa ett kund-SSL-certifikat i Azure Portal och binda det till webbappen. Mer information finns i [själv studie kursen om App Service certifikat](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).

### <a name="prerequisites"></a>Förutsättningar

För att slutföra den här lösningen:

- [Skapa en App Service-app.](https://docs.microsoft.com/azure/app-service/)
- [Mappa ett anpassat DNS-namn till din webbapp.](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- Hämta ett SSL-certifikat från en betrodd certifikat utfärdare och Använd nyckeln för att signera begäran.

### <a name="requirements-for-your-ssl-certificate"></a>Krav för ditt SSL-certifikat

Om du vill använda ett certifikat i App Service måste certifikatet uppfylla alla följande krav:

- Signerat av en betrodd certifikat utfärdare.

- Exporteras som en lösenordsskyddad PFX-fil.

- Innehåller en privat nyckel som är minst 2048 bitar lång.

- Innehåller alla mellanliggande certifikat i certifikat kedjan.

> [!Note]  
> **Elliptic Curve Cryptography (ECC)-certifikat** fungerar med App Service, men ingår inte i den här hand boken. Kontakta en certifikat utfärdare för att få hjälp med att skapa ECC-certifikat.

#### <a name="prepare-the-web-app"></a>Förbered webbappen

För att binda ett anpassat SSL-certifikat till webbappen måste [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) finnas på nivån **Basic**, **standard**eller **Premium** .

#### <a name="sign-in-to-azure"></a>Logga in på Azure

1. Öppna [Azure Portal](https://portal.azure.com/) och gå till webbappen.

2. Välj **app Services**på menyn till vänster och välj sedan namnet på webb programmet.

![Välj webbapp i Azure Portal](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Kontrollera prisnivån

1. I det vänstra navigerings fönstret på sidan webb program bläddrar du till avsnittet **Inställningar** och väljer **skala upp (App Service plan)**.

    ![Skala upp-menyn i en webbapp](media/solution-deployment-guide-geo-distributed/image34.png)

1. Se till att webbappen inte finns på nivån **kostnads fri** eller **delad** . Webbappens aktuella nivå markeras i en mörkblå ruta.

    ![Kontrol lera pris nivån i webb programmet](media/solution-deployment-guide-geo-distributed/image35.png)

Anpassad SSL stöds inte på nivån **kostnads fri** eller **delad** . Följ stegen i nästa avsnitt eller sidan **Välj pris nivå** och hoppa över för att [Ladda upp och binda ditt SSL-certifikat](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Skala upp App Service-planen

1. Välj en av nivåerna **Basic**, **Standard** eller **Premium**.

2. Välj **Välj**.

![Välj pris nivå för din webbapp](media/solution-deployment-guide-geo-distributed/image36.png)

Skalnings åtgärden har slutförts när meddelandet visas.

![Uppskalningsmeddelande](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Bind ditt SSL-certifikat och slå samman mellanliggande certifikat

Slå samman flera certifikat i kedjan.

1. **Öppna varje certifikat** som du har fått i en text redigerare.

2. Skapa en fil för det sammanfogade certifikatet med namnet *mergedcertificate. CRT*. I redigeringsprogrammet kopierar du innehållet i varje certifikat till den här filen. Ordningen på dina certifikat ska följa ordningen i certifikatkedjan, först med ditt certifikat och sist med rotcertifikatet. Det ser ut som i följande exempel:

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>Exportera certifikat till PFX

Exportera det sammanslagna SSL-certifikatet med den privata nyckel som genereras av certifikatet.

En privat nyckel fil skapas via OpenSSL. Om du vill exportera certifikatet till PFX kör du följande kommando och ersätter plats hållarna `<private-key-file>` och `<merged-certificate-file>` med sökvägen till den privata nyckeln och den sammanslagna certifikat filen:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

När du uppmanas till det anger du ett export lösen ord för att ladda upp SSL-certifikatet till App Service senare.

När IIS eller **Certreq.exe** används för att generera en certifikatbegäran installerar du certifikatet på en lokal dator och [exporterar certifikatet till PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

#### <a name="upload-the-ssl-certificate"></a>Ladda upp SSL-certifikatet

1. Välj **SSL-inställningar** i den vänstra navigeringen i webbappen.

2. Välj **överför certifikat**.

3. I **PFX-certifikatfil**väljer du PFX-fil.

4. I **certifikat lösen ord**skriver du det lösen ord som du skapade när du exporterade PFX-filen.

5. Välj **Överför**.

    ![Överför SSL-certifikat](media/solution-deployment-guide-geo-distributed/image38.png)

När App Service har laddat upp certifikatet visas det på sidan **SSL-inställningar** .

![SSL-inställningar](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>Binda SSL-certifikatet

1. I avsnittet **SSL-bindningar** väljer du **Lägg till bindning**.

    > [!Note]  
    >  Om certifikatet har laddats upp, men inte visas i domän namn i list rutan **hostname** , försöker du uppdatera webb sidan.

2. På sidan **Lägg till SSL-bindning** använder du List rutan för att välja det domän namn som ska skyddas och det certifikat som ska användas.

3. I **SSL-typ** väljer du om du vill använda [**Servernamnindikator (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) eller IP-baserad SSL.

    - **SNI-baserad SSL**: flera SNI-baserade SSL-bindningar kan läggas till. Med det här alternativet kan flera SSL-certifikat skydda flera domäner på samma IP-adress. De flesta moderna webbläsare (inklusive Internet Explorer, Chrome, Firefox och Opera) stöder SNI (mer information om webbläsare som stöds finns i [Servernamnindikator](https://wikipedia.org/wiki/Server_Name_Indication)).

    - **IP-baserad SSL**: det går bara att lägga till en IP-baserad SSL-bindning. Med det här alternativet tillåts endast ett SSL-certifikat för att skydda en dedikerad offentlig IP-adress. Skydda flera domäner genom att skydda dem med samma SSL-certifikat. IP-baserad SSL är det traditionella alternativet för SSL-bindning.

4. Välj **Lägg till bindning**.

    ![Lägg till SSL-bindning](media/solution-deployment-guide-geo-distributed/image40.png)

När App Service har laddat upp certifikatet visas det i avsnitten **SSL-bindningar** .

![SSL-bindningar har överförts](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>Mappa om A-posten för IP SSL

Om IP-baserad SSL inte används i webbappen går du vidare till [testa https för din anpassade domän](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

Som standard använder webbapp en delad offentlig IP-adress. När certifikatet är kopplat till IP-baserad SSL skapar App Service en ny och dedikerad IP-adress för webbappen.

När en A-post mappas till webbappen måste domän registret uppdateras med den dedikerade IP-adressen.

Sidan **anpassad domän** uppdateras med den nya, dedikerade IP-adressen. Kopiera den här [IP-adressen](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)och mappa sedan om A- [posten](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) till den nya IP-adressen.

#### <a name="test-https"></a>Testa HTTPS

I olika webbläsare går du till `https://<your.custom.domain>` för att se till att webbappen hanteras.

![Bläddra till webb program](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Om certifikat verifierings fel inträffar kan ett självsignerat certifikat vara orsaken, eller så kan mellanliggande certifikat ha lämnats kvar vid export till PFX-filen.

#### <a name="enforce-https"></a>Använda HTTPS

Som standard har vem som helst åtkomst till webbappen med HTTP. Alla HTTP-förfrågningar till HTTPS-porten kan omdirigeras.

På sidan webb program väljer du **SL-inställningar**. I **Endast HTTPS** väljer du **På**.

![Använda HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

När åtgärden har slutförts går du till någon av de HTTP-URL: er som pekar på appen. Ett exempel:

- https://<app_name>. azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>Använda TLS 1.1/1.2

Appen tillåter [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0 som standard, vilket inte längre anses vara säkert av bransch standarder (t. ex. [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)). Följ dessa steg om du vill göra en högre TLS-version obligatorisk:

1. På sidan webbapp i det vänstra navigerings fönstret väljer du SSL- **Inställningar**.

2. I **TLS-version**väljer du den lägsta TLS-versionen.

    ![Kräv TLS 1.1 eller 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Skapa en Traffic Manager-profil

1. Välj **skapa en resurs**  >  **nätverk**  >  **Traffic Manager profil**  >  **skapa**.

2. I **Skapa Traffic Manager-profil** gör du följande:

    1. I **namn**anger du ett namn för profilen. Det här namnet måste vara unikt i Traffic manager.net-zonen och resulterar i DNS-namnet trafficmanager.net, som används för att få åtkomst till den Traffic Manager profilen.

    2. I **routningsmetod**väljer du **metoden geografisk routning**.

    3. I **prenumeration**väljer du den prenumeration som du vill skapa profilen under.

    4. I **Resursgrupp** skapar du en ny resursgrupp att placera profilen under.

    5. I **Resursgruppsplats** väljer du plats för resursgruppen. Den här inställningen refererar till platsen för resurs gruppen och har ingen inverkan på den Traffic Manager profilen som distribueras globalt.

    6. Välj **Skapa**.

    7. När den globala distributionen av Traffic Managers profilen är klar visas den i respektive resurs grupp som en av resurserna.

        ![Resurs grupper i skapa Traffic Manager profil](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Lägga till Traffic Manager-slutpunkter

1. I Portal Sök fältet söker du efter **Traffic Manager profil** namn som skapades i föregående avsnitt och väljer Traffic Manager-profilen i de resultat som visas.

2. I **Traffic Manager profil**i avsnittet **Inställningar** väljer du **slut punkter**.

3. Välj **Lägg till**.

4. Lägger till Azure Stack Hub-slutpunkten.

5. I **typ**väljer du **extern slut punkt**.

6. Ange ett **namn** för den här slut punkten, helst namnet på Azure Stack hubben.

7. För fullständigt kvalificerat domän namn (**FQDN**) använder du den externa URL: en för webbappen för Azure Stack Hub.

8. Under geo-mappning väljer du en region/kontinent där resursen finns. Till exempel **Europa.**

9. Under List rutan land/region som visas väljer du det land som gäller för den här slut punkten. Till exempel **Tyskland**.

10. Behåll **Lägg till som inaktiverad** som avmarkerat.

11. Välj **OK**.

12. Lägger till Azure-slutpunkt:

    1. I **typ**väljer du **Azure-slutpunkt**.

    2. Ange ett **namn** för slut punkten.

    3. För **mål resurs typ**väljer du **App Service**.

    4. För **mål resurs**väljer du **Välj en app service** för att visa listan över Web Apps under samma prenumeration. I **resurs**väljer du App Service som används som första slut punkt.

13. Under geo-mappning väljer du en region/kontinent där resursen finns. Till exempel **Nordamerika/Central Amerika/Karibien.**

14. Under List rutan land/region som visas lämnar du den här punkten tom för att välja hela den regionala grupperingen.

15. Behåll **Lägg till som inaktiverad** som avmarkerat.

16. Välj **OK**.

    > [!Note]  
    >  Skapa minst en slut punkt med en geografisk omfattning av alla (värld) som ska fungera som standard slut punkt för resursen.

17. När båda slut punkterna har lagts till visas de i **Traffic Manager profil** tillsammans med deras övervaknings status som **online**.

    ![Slut punkts status för Traffic Manager profil](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>Globalt företag är beroende av Azures funktioner för geo-distribution

Genom att dirigera data trafik via Azure Traffic Manager och geografibaserade slut punkter kan globala företag följa regionala bestämmelser och hålla data kompatibla och säkra, vilket är viktigt för att det ska bli både lokala och fjärranslutna affärs platser.

## <a name="next-steps"></a>Nästa steg

- Mer information om moln mönster i Azure finns i [design mönster för molnet](https://docs.microsoft.com/azure/architecture/patterns).
