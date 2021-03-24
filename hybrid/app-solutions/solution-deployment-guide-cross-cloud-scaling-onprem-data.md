---
title: Distribuera en hybrid app med lokala data som skalar över molnet
description: Lär dig hur du distribuerar en app som använder lokala data och skalar över molnet med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0989859fd68847932d3e69defee59740a2bffd44
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895405"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Distribuera en hybrid app med lokala data som skalar över molnet

Den här lösnings guiden visar hur du distribuerar en hybrid app som omfattar både Azure och Azure Stack hubb och använder en enda lokal data källa.

Genom att använda en hybrid moln lösning kan du kombinera kompatibiliteten för ett privat moln med skalbarheten för det offentliga molnet. Utvecklarna kan också dra nytta av Microsoft Developer-eko systemet och tillämpa deras kunskaper i molnet och i lokala miljöer.

## <a name="overview-and-assumptions"></a>Översikt och antaganden

Följ den här självstudien för att skapa ett arbets flöde som gör det möjligt för utvecklare att distribuera en identisk webbapp till ett offentligt moln och ett privat moln. Den här appen kan komma åt ett icke-Internet-dirigerbart nätverk som finns på det privata molnet. Dessa webbappar övervakas och när det finns en insamling i trafik, ändrar ett program DNS-posterna för att dirigera trafik till det offentliga molnet. När trafiken sjunker till nivån före insamling dirigeras trafiken tillbaka till det privata molnet.

Den här självstudien omfattar följande uppgifter:

> [!div class="checklist"]
> - Distribuera en hybrid-ansluten SQL Server databas server.
> - Anslut en webbapp i Global Azure till ett hybrid nätverk.
> - Konfigurera DNS för skalning över molnet.
> - Konfigurera SSL-certifikat för skalning över molnet.
> - Konfigurera och distribuera webbappen.
> - Skapa en Traffic Manager profil och konfigurera den för skalning över molnet.
> - Konfigurera Application Insights övervakning och aviseringar för ökad trafik.
> - Konfigurera automatisk trafik växling mellan Global Azure och Azure Stack Hub.

> [!Tip]  
> ![Diagram över hybrid pelare](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

### <a name="assumptions"></a>Antaganden

Den här självstudien förutsätter att du har grundläggande kunskaper om Global Azure och Azure Stack Hub. Om du vill veta mer innan du startar självstudien kan du läsa följande artiklar:

- [Introduktion till Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Viktiga begrepp för Azure Stack hubb](/azure-stack/operator/azure-stack-overview)

Den här kursen förutsätter också att du har en Azure-prenumeration. Om du inte har någon prenumeration kan du [skapa ett kostnads fritt konto](https://azure.microsoft.com/free/) innan du börjar.

## <a name="prerequisites"></a>Förutsättningar

Innan du startar den här lösningen ser du till att du uppfyller följande krav:

- En Azure Stack Development Kit (ASDK) eller en prenumeration på ett integrerat system för Azure Stack Hub. Om du vill distribuera ASDK följer du anvisningarna i [distribuera ASDK med installations programmet](/azure-stack/asdk/asdk-install).
- Installationen av Azure Stack Hub måste ha följande installerat:
  - Azure App Service. Arbeta med din Azure Stack Hub-operatör för att distribuera och konfigurera Azure App Service i din miljö. Den här självstudien kräver att App Service har minst en (1) tillgänglig dedikerad arbets roll.
  - En Windows Server 2016-avbildning.
  - En Windows Server 2016 med en Microsoft SQL Server avbildning.
  - Lämpliga planer och erbjudanden.
  - Ett domän namn för din webbapp. Om du inte har ett domän namn kan du köpa ett från en domän leverantör som GoDaddy, BlueHost och InMotion.
- Ett SSL-certifikat för din domän från en betrodd certifikat utfärdare som LetsEncrypt.
- En webbapp som kommunicerar med en SQL Server-databas och stöder Application Insights. Du kan hämta exempel appen [dotnetcore-SQLDB-självstudie](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) från GitHub.
- Ett hybrid nätverk mellan ett virtuellt Azure-nätverk och Azure Stack hubb virtuellt nätverk. Detaljerade anvisningar finns i [Konfigurera hybrid moln anslutning med Azure och Azure Stack hubb](solution-deployment-guide-connectivity.md).

- En pipeline för kontinuerlig integrering/en kontinuerlig distribution (CI/CD) med en privat build-agent på Azure Stack Hub. Detaljerade anvisningar finns i [Konfigurera hybrid moln identitet med Azure och Azure Stack Hub-appar](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Distribuera en hybrid-ansluten SQL Server databas server

1. Logga in på användar portalen för Azure Stack Hub.

2. Välj **Marketplace** på **instrument panelen**.

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. Välj **Compute** i **Marketplace** och välj sedan **mer**. Under **mer** väljer du den **kostnads fria SQL Server licensen: SQL Server 2017 Developer på Windows Server** -avbildning.

    ![Välj en avbildning av en virtuell dator i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image2.png)

4. På **licens för gratis SQL Server: SQL Server 2017-utvecklare på Windows Server**, Välj **skapa**.

5. I **grundläggande > du konfigurera grundläggande inställningar**, ange ett **namn** för den virtuella datorn (VM), ett **användar namn** för SQL Server SA och ett **lösen ord** för säkerhets associationen.  I list rutan **prenumeration** väljer du den prenumeration som du distribuerar till. För **resurs grupp** använder du **Välj befintlig** och sätter den virtuella datorn i samma resurs grupp som din Azure Stack hubb-webbapp.

    ![Konfigurera grundläggande inställningar för virtuell dator i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image3.png)

6. Välj en storlek för den virtuella datorn under **storlek**. I den här självstudien rekommenderar vi A2_Standard eller en DS2_V2_Standard.

7. Under **inställningar > konfigurera valfria funktioner**, konfigurerar du följande inställningar:

   - **Lagrings konto**: skapa ett nytt konto om du behöver ett.
   - **Virtuellt nätverk**:

     > [!Important]  
     > Kontrol lera att din SQL Server VM har distribuerats på samma virtuella nätverk som VPN-gatewayerna.

   - **Offentlig IP-adress**: Använd standardinställningarna.
   - **Nätverks säkerhets grupp**: (NSG). Skapa en ny NSG.
   - **Tillägg och övervakning**: Behåll standardinställningarna.
   - **Lagrings konto för diagnostik**: skapa ett nytt konto om du behöver ett.
   - Välj **OK** för att spara konfigurationen.

     ![Konfigurera valfria VM-funktioner i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image4.png)

8. Under **SQL Server inställningar** konfigurerar du följande inställningar:

   - Välj **offentlig (Internet)** för **SQL-anslutning**.
   - Behåll standardvärdet **1433** för **port**.
   - Välj **Aktivera** för **SQL-autentisering**.

     > [!Note]  
     > När du aktiverar SQL-autentisering ska den automatiskt fyllas i med "SQLAdmin"-informationen som du konfigurerade i **grunderna**.

   - Behåll standardvärdena för resten av inställningarna. Välj **OK**.

     ![Konfigurera SQL Server inställningar i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image5.png)

9. Vid **Sammanfattning** granskar du VM-konfigurationen och väljer sedan **OK** för att starta distributionen.

    ![Konfigurations Sammanfattning i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image6.png)

10. Det tar lite tid att skapa den nya virtuella datorn. Du kan visa STATUSEN för dina virtuella datorer på **virtuella datorer**.

    ![Status för virtuella datorer i Azure Stack hubb användar portalen](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Skapa webb program i Azure och Azure Stack hubb

Azure App Service fören klar att köra och hantera en webbapp. Eftersom Azure Stack Hub är konsekvent med Azure kan App Service köras i båda miljöerna. Du använder App Service som värd för din app.

### <a name="create-web-apps"></a>Skapa webb program

1. Skapa en webbapp i Azure genom att följa anvisningarna i [hantera ett App Service plan i Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan). Se till att du sätter webbappen i samma prenumeration och resurs grupp som ditt hybrid nätverk.

2. Upprepa föregående steg (1) i Azure Stack Hub.

### <a name="add-route-for-azure-stack-hub"></a>Lägg till väg för Azure Stack hubb

App Service på Azure Stack Hub måste dirigeras från det offentliga Internet för att användarna ska kunna komma åt din app. Om din Azure Stack hubb är tillgänglig från Internet noterar du den offentliga IP-adressen eller URL: en för den Azure Stack Hub-webbappen.

Om du använder en ASDK kan du [Konfigurera en statisk NAT-mappning](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) för att exponera App Service utanför den virtuella miljön.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Ansluta en webbapp i Azure till ett hybrid nätverk

För att tillhandahålla anslutning mellan webb klient delen i Azure och SQL Server databasen i Azure Stack Hub måste webbappen vara ansluten till hybrid nätverket mellan Azure och Azure Stack Hub. Om du vill aktivera anslutning måste du:

- Konfigurera punkt-till-plats-anslutning.
- Konfigurera webbappen.
- Ändra den lokala Nätverksgatewayen i Azure Stack Hub.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Konfigurera det virtuella Azure-nätverket för punkt-till-plats-anslutning

Den virtuella Nätverksgatewayen på Azure-sidan av hybrid nätverket måste tillåta punkt-till-plats-anslutningar för att kunna integreras med Azure App Service.

1. Gå till sidan för virtuell nätverksgateway i Azure Portal. Under **Inställningar** väljer **du punkt-till-plats-konfiguration**.

    ![Alternativet punkt-till-plats i Azure Virtual Network Gateway](media/solution-deployment-guide-hybrid/image8.png)

2. Välj **Konfigurera nu** för att konfigurera punkt-till-plats.

    ![Starta punkt-till-plats-konfiguration i Azure virtuell nätverksgateway](media/solution-deployment-guide-hybrid/image9.png)

3. På sidan **punkt-till-plats** -konfiguration anger du det privata IP-adressintervall som du vill använda i **adresspoolen**.

   > [!Note]  
   > Kontrol lera att det intervall som du anger inte överlappar något av adress intervallen som redan används av undernät i de globala Azure-eller Azure Stack Hub-komponenterna i hybrid nätverket.

   Avmarkera **IKEV2 VPN** under **tunnel typ**. Välj **Spara** för att slutföra konfigurationen av punkt-till-plats.

   ![Punkt-till-plats-inställningar i Azure virtuell nätverksgateway](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Integrera Azure App Service-appen med hybrid nätverket

1. Om du vill ansluta appen till Azure VNet följer du anvisningarna i [Gateway krävs VNet-integrering](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).

2. Gå till **Inställningar** för App Service plan som är värd för webbappen. I **Inställningar** väljer du **nätverk**.

    ![Konfigurera nätverk för App Service plan](media/solution-deployment-guide-hybrid/image11.png)

3. I **VNet-integration** väljer **du klicka här för att hantera**.

    ![Hantera VNET-integrering för App Service plan](media/solution-deployment-guide-hybrid/image12.png)

4. Välj det virtuella nätverk som du vill konfigurera. Under **IP-adresser dirigeras till VNet**, anger du IP-adressintervallet för det virtuella Azure-nätverket, Azure Stack hubb-VNet och punkt-till-plats-adress utrymmen. Välj **Spara** för att validera och spara inställningarna.

    ![IP-adressintervall som ska vidarebefordras i Virtual Network-integrering](media/solution-deployment-guide-hybrid/image13.png)

Mer information om hur App Service integreras med Azure virtuella nätverk finns i [integrera din app med en Azure-Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Konfigurera det virtuella nätverket för Azure Stack hubb

Den lokala Nätverksgatewayen i Azure Stack hubbens virtuella nätverk måste konfigureras för att dirigera trafik från App Service punkt-till-plats-adress intervall.

1. I Azure Stack Hub-portalen går du till **lokal nätverksgateway**. Under **Inställningar** väljer du **Konfiguration**.

    ![Konfigurations alternativ för gateway i Azure Stack hubb lokal nätverksgateway](media/solution-deployment-guide-hybrid/image14.png)

2. I **adress utrymme** anger du punkt-till-plats-adressintervallet för den virtuella Nätverksgatewayen i Azure.

    ![Adress utrymme för punkt-till-plats i Azure Stack hubb lokal nätverksgateway](media/solution-deployment-guide-hybrid/image15.png)

3. Välj **Spara** för att validera och spara konfigurationen.

## <a name="configure-dns-for-cross-cloud-scaling"></a>Konfigurera DNS för skalning över molnet

Genom att konfigurera DNS för appar över molnet korrekt kan användare komma åt de globala Azure-och Azure Stack Hub-instanserna av din webbapp. DNS-konfigurationen för den här självstudien låter också Azure Traffic Manager dirigera trafik när belastningen ökar eller minskar.

I den här självstudien används Azure DNS för att hantera DNS eftersom App Service domäner inte fungerar.

### <a name="create-subdomains"></a>Skapa under domäner

Eftersom Traffic Manager är beroende av DNS-CNAME, krävs en under domän för att dirigera trafik till slut punkter korrekt. Mer information om DNS-poster och domän mappning finns i [Mappa domäner med Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).

För Azure-slutpunkten skapar du en under domän som användare kan använda för att få åtkomst till din webbapp. I den här självstudien kan använda **app.Northwind.com**, men du bör anpassa det här värdet baserat på din egen domän.

Du måste också skapa en under domän med en A-post för Azure Stack Hub-slutpunkten. Du kan använda **azurestack.Northwind.com**.

### <a name="configure-a-custom-domain-in-azure"></a>Konfigurera en anpassad domän i Azure

1. Lägg till **app.Northwind.com** -värdnamnet i Azure-webbappen genom att [Mappa en CNAME till Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Konfigurera anpassade domäner i Azure Stack hubb

1. Lägg till **azurestack.Northwind.com** -värdnamnet i Azure Stack Hub-webbappen genom [att mappa en A-post till Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Använd IP-adressen Internet-dirigerbart för App Service-appen.

2. Lägg till **app.Northwind.com** -värdnamnet i Azure Stack Hub-webbappen genom att [Mappa en CNAME till Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Använd det värdnamn som du konfigurerade i föregående steg (1) som mål för CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Konfigurera SSL-certifikat för skalning över molnet

Det är viktigt att se till att känsliga data som samlas in av din webbapp är säkra vid överföring till och när de lagras i SQL-databasen.

Du konfigurerar dina Azure-och Azure Stack Hub-webbappar så att de använder SSL-certifikat för all inkommande trafik.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Lägg till SSL till Azure och Azure Stack hubb

Så här lägger du till SSL i Azure:

1. Kontrol lera att SSL-certifikatet som du får är giltigt för den under domän som du skapade. (Det är OK att använda certifikat med jokertecken.)

2. Följ anvisningarna i avsnittet **förbereda din webbapp** och **BIND SSL-certifikat** i avsnittet [BIND ett befintligt anpassat ssl-certifikat till Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) artikeln i Azure Portal. Välj **SNI-baserad SSL** som **SSL-typ**.

3. Omdirigera all trafik till HTTPS-porten. Följ instruktionerna i avsnittet om att   **använda https** i avsnittet [BIND ett befintligt anpassat SSL-certifikat till Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) .

Så här lägger du till SSL till Azure Stack Hub:

1. Upprepa steg 1-3 som du använde för Azure med hjälp av Azure Stack Hub-portalen.

## <a name="configure-and-deploy-the-web-app"></a>Konfigurera och distribuera webbappen

Du konfigurerar appens kod för att rapportera telemetri till rätt Application Insights-instans och konfigurera webbapparna med rätt anslutnings strängar. Mer information om Application Insights finns i [Vad är Application Insights?](/azure/application-insights/app-insights-overview)

### <a name="add-application-insights"></a>Lägg till Application Insights

1. Öppna din webbapp i Microsoft Visual Studio.

2. [Lägg till Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) i projektet för att överföra telemetri som Application Insights använder för att skapa aviseringar när webb trafiken ökar eller minskar.

### <a name="configure-dynamic-connection-strings"></a>Konfigurera dynamiska anslutnings strängar

Varje instans av webbappen kommer att använda en annan metod för att ansluta till SQL-databasen. Appen i Azure använder den privata IP-adressen för SQL Server VM och appen i Azure Stack Hub använder den offentliga IP-adressen för SQL Server VM.

> [!Note]  
> På ett integrerat Azure Stack Hub-system får den offentliga IP-adressen inte vara Internet-dirigerbart. På en ASDK dirigeras inte den offentliga IP-adressen utanför ASDK.

Du kan använda App Service miljövariabler för att skicka en annan anslutnings sträng till varje instans av appen.

1. Öppna appen i Visual Studio.

2. Öppna Start. CS och hitta följande kodblock:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Ersätt det tidigare kod blocket med följande kod, som använder en anslutnings sträng som definierats i *appsettings.jspå* filen:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>Konfigurera inställningar för App Service app

1. Skapa anslutnings strängar för Azure och Azure Stack Hub. Strängarna måste vara desamma, förutom de IP-adresser som används.

2. I Azure och Azure Stack hubb lägger du till rätt anslutnings sträng [som en app-inställning](/azure/app-service/web-sites-configure) i webbappen med `SQLCONNSTR\_` som ett prefix i namnet.

3. **Spara** inställningarna för webbappen och starta om appen.

## <a name="enable-automatic-scaling-in-global-azure"></a>Aktivera automatisk skalning i Global Azure

När du skapar en webbapp i en App Services miljö börjar den med en instans. Du kan automatiskt skala ut för att lägga till instanser för att tillhandahålla fler beräknings resurser för din app. På samma sätt kan du automatiskt skala in och minska antalet instanser som appen behöver.

> [!Note]  
> Du måste ha en App Service plan för att kunna konfigurera skala ut och skala in. Om du inte har någon plan skapar du ett innan du påbörjar nästa steg.

### <a name="enable-automatic-scale-out"></a>Aktivera automatisk utskalning

1. I Azure Portal letar du reda på App Service plan för de platser som du vill skala ut och väljer sedan **skala ut (App Service plan)**.

    ![Skala ut Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. Välj **Aktivera autoskalning**.

    ![Aktivera autoskalning i Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. Ange ett namn för den **automatiska skalnings inställningen**. För **standard** regeln för automatisk skalning väljer du **skala baserat på ett mått**. Ange att **instans gränserna** ska vara **minst: 1**, **Max: 10** och **standard: 1**.

    ![Konfigurera autoskalning i Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. Välj **+ Lägg till en regel**.

5. I **mått källa** väljer du **aktuell resurs**. Använd följande kriterier och åtgärder för regeln.

#### <a name="criteria"></a>Kriterie

1. Under **tids agg regering väljer du** **Average**.

2. Under **Metric Name** väljer du **processor procent**.

3. Under **operatör** väljer du **större än**.

   - Ange **tröskelvärdet** till **50**.
   - Ange **varaktigheten** till **10**.

#### <a name="action"></a>Action

1. Under **åtgärd** väljer du **öka antalet efter**.

2. Ange **antalet instanser** till **2**.

3. Ange **nedkylning** till **5**.

4. Välj **Lägg till**.

5. Välj **+ Lägg till en regel**.

6. I **mått källa** väljer du **aktuell resurs.**

   > [!Note]  
   > Den aktuella resursen kommer att innehålla App Service plan namn/GUID och list rutorna **resurs typ** och **resurs** är inte tillgängliga.

### <a name="enable-automatic-scale-in"></a>Aktivera automatisk skalning i

När trafiken minskar kan Azure-webbappen automatiskt minska antalet aktiva instanser för att minska kostnaderna. Den här åtgärden är mindre aggressiv än att skala ut och minimera påverkan på användare i appar.

1. Gå till **standard** villkoret för skala ut och välj sedan **+ Lägg till en regel**. Använd följande kriterier och åtgärder för regeln.

#### <a name="criteria"></a>Kriterie

1. Under **tids agg regering väljer du** **Average**.

2. Under **Metric Name** väljer du **processor procent**.

3. Under **operator** väljer du **mindre än**.

   - Ange **tröskelvärdet** till **30**.
   - Ange **varaktigheten** till **10**.

#### <a name="action"></a>Action

1. Under **åtgärd** väljer du **minska antalet efter**.

   - Ange **antalet instanser** till **1**.
   - Ange **nedkylning** till **5**.

2. Välj **Lägg till**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Skapa en Traffic Manager profil och konfigurera skalning mellan moln

Skapa en Traffic Manager profil med hjälp av Azure Portal och konfigurera sedan slut punkter för att aktivera skalning över molnet.

### <a name="create-traffic-manager-profile"></a>Skapa Traffic Manager profil

1. Välj **Skapa en resurs**.
2. Välj **Nätverk**.
3. Välj **Traffic Manager profil** och konfigurera följande inställningar:

   - I **namn** anger du ett namn för din profil. Det här namnet **måste** vara unikt i trafficmanager.net-zonen och används för att skapa ett nytt DNS-namn (till exempel northwindstore.trafficmanager.net).
   - För **routningsmetod** väljer du **viktat**.
   - För **prenumeration** väljer du den prenumeration som du vill skapa profilen i.
   - I **resurs grupp** skapar du en ny resurs grupp för den här profilen.
   - I **Resursgruppsplats** väljer du plats för resursgruppen. Den här inställningen refererar till platsen för resurs gruppen och påverkar inte den Traffic Manager profilen som distribueras globalt.

4. Välj **Skapa**.

    ![Skapa Traffic Manager profil](media/solution-deployment-guide-hybrid/image19.png)

   När den globala distributionen av Traffic Managers profilen är klar visas den i listan över resurser för resurs gruppen som du skapade den under.

### <a name="add-traffic-manager-endpoints"></a>Lägga till Traffic Manager-slutpunkter

1. Sök efter den Traffic Manager profil som du har skapat. Om du har navigerat till resurs gruppen för profilen väljer du profilen.

2. I **Traffic Manager profil** väljer du **slut punkter** under **Inställningar**.

3. Välj **Lägg till**.

4. I **Lägg till slut punkt** använder du följande inställningar för Azure Stack Hub:

   - I **typ** väljer du **extern slut punkt**.
   - Ange ett **namn** för slut punkten.
   - För **fullständigt kvalificerat domän namn (FQDN) eller IP-** adress anger du den externa URL: en för din Azure Stack Hub-webbapp.
   - Behåll standardvärdet **1** för **vikt**. Den här vikten resulterar i all trafik som kommer till den här slut punkten om den är felfri.
   - Lämna **Lägg till som inaktiverat** avmarkerat.

5. Välj **OK** för att spara Azure Stack Hub-slutpunkten.

Du konfigurerar Azure-slutpunkten härnäst.

1. På **Traffic Manager profil** väljer du **slut punkter**.
2. Välj **+Lägg till**.
3. Använd följande inställningar för Azure på **Lägg till slut punkt**:

   - I **typ** väljer du **Azure-slutpunkt**.
   - Ange ett **namn** för slut punkten.
   - För **mål resurs typ** väljer du **App Service**.
   - För **mål resurs** väljer du **Välj en app service** om du vill visa en lista över Web Apps i samma prenumeration.
   - I **Resurs** väljer du den apptjänst som du vill lägga till som den första slutpunkten.
   - I **vikt** väljer du **2**. Den här inställningen resulterar i all trafik som går till den här slut punkten om den primära slut punkten är ohälsosam eller om du har en regel/avisering som omdirigerar trafik när den utlöses.
   - Lämna **Lägg till som inaktiverat** avmarkerat.

4. Välj **OK** för att spara Azure-slutpunkten.

När båda slut punkterna har kon figurer ATS visas de i **Traffic Manager profil** när du väljer **slut punkter**. Exemplet i följande skärm bild visar två slut punkter, med status-och konfigurations information för var och en.

![Slut punkter i Traffic Managers profil](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a>Konfigurera Application Insights övervakning och aviseringar i Azure

Med Azure Application Insights kan du övervaka din app och skicka aviseringar baserat på de villkor som du konfigurerar. Några exempel är: appen är inte tillgänglig, har fel eller visar prestanda problem.

Du ska använda Azure Application Insights-mått för att skapa aviseringar. När dessa aviseringar utlöses växlar webbappens instans automatiskt från Azure Stack hubb till Azure för att skala ut, och sedan tillbaka till Azure Stack Hub för att skala in.

### <a name="create-an-alert-from-metrics"></a>Skapa en avisering utifrån mått

I Azure Portal går du till resurs gruppen för den här självstudien och väljer Application Insights-instansen för att öppna **Application Insights**.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Du använder den här vyn för att skapa en skalbar avisering och en avisering om skalnings avisering.

### <a name="create-the-scale-out-alert"></a>Skapa en skalbar avisering

1. Under **Konfigurera** väljer du **aviseringar (klassisk)**.
2. Välj **Lägg till mått varning (klassisk)**.
3. Konfigurera följande inställningar i **Lägg till regel**:

   - I **namn** anger du **burst i Azure-molnet**.
   - En **Beskrivning** är valfri.
   - Under **käll**  >  **avisering på** väljer du **mått**.
   - Under **kriterier** väljer du din prenumeration, resurs grupp för din Traffic Manager profil och namnet på den Traffic Manager profilen för resursen.

4. För **mått** väljer du **begär ande frekvens**.
5. För **villkor** väljer du **större än**.
6. För **tröskel** anger du **2**.
7. För **period** väljer **du de senaste 5 minuterna**.
8. Under **meddela via**:
   - Markera kryss rutan för **e-postägare, deltagare och läsare**.
   - Ange din e-postadress för **ytterligare administratörs-e-post (er)**.

9. I meny raden väljer du **Spara**.

### <a name="create-the-scale-in-alert"></a>Skapa en skalnings avisering

1. Under **Konfigurera** väljer du **aviseringar (klassisk)**.
2. Välj **Lägg till mått varning (klassisk)**.
3. Konfigurera följande inställningar i **Lägg till regel**:

   - I **namn** anger du **skala tillbaka till Azure Stack Hub**.
   - En **Beskrivning** är valfri.
   - Under **käll**  >  **avisering på** väljer du **mått**.
   - Under **kriterier** väljer du din prenumeration, resurs grupp för din Traffic Manager profil och namnet på den Traffic Manager profilen för resursen.

4. För **mått** väljer du **begär ande frekvens**.
5. För **villkor** väljer du **mindre än**.
6. För **tröskel** anger du **2**.
7. För **period** väljer **du de senaste 5 minuterna**.
8. Under **meddela via**:
   - Markera kryss rutan för **e-postägare, deltagare och läsare**.
   - Ange din e-postadress för **ytterligare administratörs-e-post (er)**.

9. I meny raden väljer du **Spara**.

Följande skärm bild visar aviseringarna för att skala ut och skala in.

   ![Application Insights aviseringar (klassisk)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Omdirigera trafik mellan Azure och Azure Stack hubb

Du kan konfigurera manuell eller automatisk växling av din webb program trafik mellan Azure och Azure Stack hubben.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Konfigurera manuell växling mellan Azure och Azure Stack hubb

När webbplatsen når tröskelvärdena som du konfigurerar får du en avisering. Använd följande steg för att manuellt omdirigera trafik till Azure.

1. I Azure Portal väljer du din Traffic Manager profil.

    ![Traffic Manager slut punkter i Azure Portal](media/solution-deployment-guide-hybrid/image20.png)

2. Välj **slut punkter**.
3. Välj **Azure-slutpunkten**.
4. Under **status** väljer du **aktive rad** och väljer sedan **Spara**.

    ![Aktivera Azure-slutpunkten i Azure Portal](media/solution-deployment-guide-hybrid/image23.png)

5. I **slut punkter** för Traffic Manager profilen väljer du **extern slut punkt**.
6. Under **status** väljer du **inaktive rad** och väljer sedan **Spara**.

    ![Inaktivera Azure Stack Hub-slutpunkten i Azure Portal](media/solution-deployment-guide-hybrid/image24.png)

När slut punkterna har kon figurer ATS går appens trafik till din Azure Scale-Out-webbapp i stället för Azure Stack Hub-webbappen.

 ![Slut punkterna har ändrats i Azure Web App-trafik](media/solution-deployment-guide-hybrid/image25.png)

Ändra tillbaka flödet till Azure Stack Hub genom att följa de föregående stegen för att:

- Aktivera Azure Stack Hub-slutpunkten.
- Inaktivera Azure-slutpunkten.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Konfigurera automatisk växling mellan Azure och Azure Stack hubb

Du kan också använda Application Insights övervakning om din app körs i en [Server](https://azure.microsoft.com/overview/serverless-computing/) lös miljö som tillhandahålls av Azure Functions.

I det här scenariot kan du konfigurera Application Insights att använda en webhook som anropar en Function-app. Den här appen aktiverar eller inaktiverar automatiskt en slut punkt som svar på en avisering.

Använd följande steg som en guide för att konfigurera automatisk trafik växling.

1. Skapa en Azure Function-app.
2. Skapa en HTTP-utlöst funktion.
3. Importera Azure SDK: er för Resource Manager, Web Apps och Traffic Manager.
4. Utveckla kod till:

   - Autentisera till din Azure-prenumeration.
   - Använd en parameter som växlar Traffic Manager slut punkter för att dirigera trafik till Azure eller Azure Stack Hub.

5. Spara koden och Lägg till funktions appens URL med lämpliga parametrar i **webhook** -avsnittet i Application Insights varnings regel inställningar.
6. Trafiken omdirigeras automatiskt när en Application Insights-avisering utlöses.

## <a name="next-steps"></a>Nästa steg

- Mer information om moln mönster i Azure finns i [design mönster för molnet](/azure/architecture/patterns).
