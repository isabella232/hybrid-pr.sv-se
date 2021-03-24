---
title: Konfigurera hybrid moln anslutning i Azure och Azure Stack hubb
description: Lär dig hur du konfigurerar hybrid moln anslutning med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4480f51b03082f2a0cbb7f2f213e05b7bf488646
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895388"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Konfigurera hybrid moln anslutning med Azure och Azure Stack hubb

Du kan komma åt resurser med säkerhet i Global Azure och Azure Stack hubb med hjälp av hybrid anslutnings mönstret.

I den här lösningen skapar du en exempel miljö för att:

> [!div class="checklist"]
> - Behåll data lokalt för att uppfylla sekretess-och reglerings krav, men ha till gång till globala Azure-resurser.
> - Underhålla ett äldre system när du använder Cloud-skalad app-distribution och resurser i globala Azure.

> [!Tip]  
> ![Diagram över hybrid pelare](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

## <a name="prerequisites"></a>Förutsättningar

Några komponenter krävs för att bygga en distribution av hybrid anslutningar. Några av de här komponenterna tar tid att förbereda, så planera därför.

### <a name="azure"></a>Azure

- Om du inte har någon Azure-prenumeration kan du skapa ett [kostnadsfritt konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) innan du börjar.
- Skapa en [webbapp](/aspnet/core/tutorials/publish-to-azure-webapp-using-vs) i Azure. Anteckna webbappens webb adress eftersom du behöver den i lösningen.

### <a name="azure-stack-hub"></a>Azure Stack Hub

En Azure OEM/maskin varu partner kan distribuera en produktions Azure Stack hubb och alla användare kan distribuera en Azure Stack Development Kit (ASDK).

- Använd din Azure Stack hubb för produktion eller distribuera ASDK.
   >[!Note]
   >Det kan ta upp till 7 timmar att distribuera ASDK, så du bör planera detta.

- Distribuera [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS-tjänster till Azure Stack Hub.
- [Skapa planer och erbjudanden](/azure-stack/operator/service-plan-offer-subscription-overview) i Azure Stack Hub-miljön.
- [Skapa klient prenumeration](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) i Azure Stack Hub-miljön.

### <a name="azure-stack-hub-components"></a>Azure Stack Hub-komponenter

En Azure Stack Hub-operatör måste distribuera App Service, skapa planer och erbjudanden, skapa en klient prenumeration och lägga till Windows Server 2016-avbildningen. Om du redan har dessa komponenter kontrollerar du att de uppfyller kraven innan du startar den här lösningen.

I det här lösnings exemplet förutsätts att du har några grundläggande kunskaper om Azure och Azure Stack Hub. Läs följande artiklar om du vill veta mer innan du påbörjar lösningen:

- [Introduktion till Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Viktiga begrepp för Azure Stack hubb](/azure-stack/operator/azure-stack-overview)

### <a name="before-you-begin"></a>Innan du börjar

Kontrol lera att du uppfyller följande kriterier innan du börjar konfigurera hybrid moln anslutning:

- Du behöver en extern offentlig IPv4-adress för VPN-enheten. Den här IP-adressen kan inte hittas bakom NAT (Network Address Translation).
- Alla resurser distribueras i samma region/plats.

#### <a name="solution-example-values"></a>Exempel värden för lösning

I exemplen i den här lösningen används följande värden. Du kan använda dessa värden för att skapa en test miljö eller referera till dem för att få en bättre förståelse för exemplen. Mer information om inställningar för VPN gateway finns i [om VPN Gateway inställningar](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).

Specifikationer för anslutning:

- **VPN-typ**: Route-based
- **Anslutnings typ**: plats-till-plats (IPSec)
- **Gateway-typ**: VPN
- **Namn på Azure-anslutning**: Azure-Gateway-AzureStack-S2SGateway (portalen fylls i det här värdet)
- **Azure Stack hubb anslutningens namn**: AzureStack-Gateway – Azure-S2SGateway (portalen fylls i det här värdet)
- **Delad nyckel**: alla kompatibla med VPN-maskinvara, med matchande värden på båda sidor om anslutning
- **Prenumeration**: valfri prioriterad prenumeration
- **Resurs grupp**: Test-Infra

IP-adresser för nätverk och undernät:

| Azure/Azure Stack Hub-anslutning | Name | Undernät | IP-adress |
|---|---|---|---|
| Azure vNet | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Azure Stack hubb vNet | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Azure Virtual Network Gateway | Azure-Gateway |  |  |
| Azure Stack hubb Virtual Network Gateway | AzureStack-Gateway |  |  |
| Azure Public IP | Azure-GatewayPublicIP |  | Bestäms vid skapande |
| Azure Stack hubb offentlig IP | AzureStack-GatewayPublicIP |  | Bestäms vid skapande |
| Azure lokal nätverksgateway | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Azure Stack Hub offentlig IP-värde |
| Azure Stack hubb lokal nätverksgateway | Azure-S2SGateway<br>10.100.102.0/23 |  | Offentligt IP-värde för Azure |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Skapa ett virtuellt nätverk i Global Azure och Azure Stack hubb

Använd följande steg för att skapa ett virtuellt nätverk med hjälp av portalen. Du kan använda dessa [exempel värden](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) om du använder den här artikeln som en lösning. Om du använder den här artikeln för att konfigurera en produktions miljö ersätter du exempel inställningarna med dina egna värden.

> [!IMPORTANT]
> Du måste se till att det inte finns några överlappande IP-adresser i Azure eller Azure Stack hubb för vNet-adress utrymmen.

Så här skapar du ett vNet i Azure:

1. Använd webbläsaren för att ansluta till [Azure Portal](https://portal.azure.com/) och logga in med ditt Azure-konto.
2. Välj **Skapa en resurs**. I fältet **Sök på Marketplace anger du** ' Virtual Network '. Välj **virtuellt nätverk** från resultaten.
3. I listan **Välj en distributions modell** väljer du **Resource Manager** och väljer sedan **skapa**.
4. Konfigurera VNet-inställningarna på **Skapa virtuellt nätverk**. De obligatoriska fält namnen föregås av en röd asterisk.  När du anger ett giltigt värde ändras asterisken till en grön bock markering.

Så här skapar du ett vNet i Azure Stack Hub:

1. Upprepa stegen ovan (1-4) med hjälp av Azure Stack Hub- **klient portalen**.

## <a name="add-a-gateway-subnet"></a>Lägga till ett gatewayundernät

Innan du ansluter det virtuella nätverket till en gateway måste du skapa Gateway-undernätet för det virtuella nätverk som du vill ansluta till. Gateway-tjänsterna använder de IP-adresser som du anger i Gateway-undernätet.

I [Azure Portal](https://portal.azure.com/)navigerar du till det virtuella Resource Manager-nätverket där du vill skapa en virtuell nätverksgateway.

1. Välj det virtuella nätverket för att öppna sidan **virtuellt nätverk** .
2. I **Inställningar** väljer du **undernät**.
3. På sidan **undernät** väljer du **+ Gateway-undernät** för att öppna sidan **Lägg till undernät** .

    ![Lägg till gateway-undernät](media/solution-deployment-guide-connectivity/image4.png)

4. **Namnet** på under nätet fylls i automatiskt med värdet ' GatewaySubnet '. Det här värdet krävs för att Azure ska kunna identifiera under nätet som gateway-undernätet.
5. Ändra **adress intervalls** värden som har angetts för att matcha dina konfigurations krav och välj sedan **OK**.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Skapa en Virtual Network gateway i Azure och Azure Stack

Använd följande steg för att skapa en virtuell nätverksgateway i Azure.

1. På vänster sida av Portal sidan väljer du **+** och anger "virtuell nätverksgateway" i Sök fältet.
2. I **resultat** väljer du **virtuell nätverksgateway**.
3. I **virtuell nätverksgateway** väljer du **skapa** för att öppna sidan **Skapa virtuell nätverksgateway** .
4. På **Skapa virtuell nätverksgateway** anger du värdena för din nätverksgateway med hjälp av våra **exempel värden för självstudie**. Inkludera följande ytterligare värden:

   - **SKU**: Basic
   - **Virtual Network**: Välj det virtuella nätverk som du skapade tidigare. Gateway-undernätet som du skapade väljs automatiskt.
   - **Första IP-konfiguration**: den offentliga IP-adressen för din gateway.
     - Välj **skapa Gateway IP-konfiguration**, som tar dig till sidan **Välj offentlig IP-adress** .
     - Välj **+ Skapa ny** för att öppna sidan **skapa offentlig IP-adress** .
     - Ange ett **namn** för din offentliga IP-adress. Låt SKU: n vara **Basic** och välj sedan **OK** för att spara ändringarna.

       > [!Note]
       > VPN Gateway stöder för närvarande endast dynamisk offentlig IP-adressallokering. Detta betyder dock inte att IP-adressen ändras efter att den har tilldelats till din VPN-gateway. Den enda gången den offentliga IP-adressen ändras är när gatewayen tas bort och återskapas. Storleks ändring, återställning eller annat internt underhåll/uppgraderingar till din VPN-gateway ändrar inte IP-adressen.

5. Verifiera dina Gateway-inställningar.
6. Välj **skapa** för att skapa VPN-gatewayen. Gateway-inställningarna verifieras och panelen "distribuerar virtuell nätverksgateway" visas på din instrument panel.

   >[!Note]
   >Det kan ta upp till 45 minuter att skapa en gateway. Det är möjligt att du behöver uppdatera din portalsida för att se statusen som slutförd.

    När gatewayen har skapats kan du se IP-adressen som tilldelats den genom att titta på det virtuella nätverket i portalen. Gatewayen visas som en ansluten enhet. Om du vill se mer information om gatewayen väljer du enheten.

7. Upprepa föregående steg (1-5) i Azure Stack Hub-distributionen.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Skapa den lokala Nätverksgatewayen i Azure och Azure Stack hubb

Den lokala nätverksgatewayen avser vanligtvis din lokala plats. Du ger platsen ett namn som Azure eller Azure Stack Hub kan referera till, och sedan ange:

- IP-adressen för den lokala VPN-enhet som du skapar en anslutning för.
- IP-adressprefix som kommer att dirigeras via VPN-gatewayen till VPN-enheten. Adressprefixen du anger är de prefix som finns på det lokala nätverket.

  >[!Note]
  >Om ditt lokala nätverk ändras eller om du behöver ändra den offentliga IP-adressen för VPN-enheten kan du uppdatera värdena senare.

1. I portalen väljer du **+ skapa en resurs**.
2. I rutan Sök anger du **lokal nätverksgateway** och väljer sedan **RETUR** för att söka. En lista med resultat visas.
3. Välj **lokal** nätverksgateway och välj sedan **skapa** för att öppna sidan **skapa lokal** nätverksgateway.
4. På **skapa lokal** nätverksgateway anger du värden för din lokala nätverksgateway med hjälp av våra **exempel värden för självstudier**. Inkludera följande ytterligare värden:

    - **IP-adress**: den offentliga IP-adressen för VPN-enheten som du vill att Azure eller Azure Stack Hub ska ansluta till. Ange en giltig offentlig IP-adress som inte ligger bakom en NAT så att Azure kan komma åt adressen. Om du inte har IP-adressen just nu kan du använda ett värde från exemplet som plats hållare. Du måste gå tillbaka och ersätta plats hållaren med den offentliga IP-adressen för VPN-enheten. Azure kan inte ansluta till enheten förrän du anger en giltig adress.
    - **Adress utrymme**: adress intervallet för det nätverk som representerar det här lokala nätverket. Du kan lägga till flera adressintervall. Kontrol lera att de intervall som du anger inte överlappar intervallen för andra nätverk som du vill ansluta till. Azure vidarebefordrar det adressintervall som du anger till den lokala VPN-enhetens IP-adress. Använd dina egna värden om du vill ansluta till din lokala plats, inte ett exempel värde.
    - **Konfigurera BGP-inställningar**: Använd endast när du konfigurerar BGP. Annars väljer du inte det här alternativet.
    - **Prenumeration**: kontrol lera att rätt prenumeration visas.
    - **Resurs grupp**: Välj den resurs grupp som du vill använda. Du kan antingen skapa en ny resurs grupp eller välja en som du redan har skapat.
    - **Plats**: Välj den plats som det här objektet kommer att skapas i. Du kanske vill välja samma plats som ditt VNet finns i, men du behöver inte göra det.
5. När du är klar med att ange de värden som krävs väljer du **skapa** för att skapa den lokala Nätverksgatewayen.
6. Upprepa de här stegen (1-5) i Azure Stack Hub-distributionen.

## <a name="configure-your-connection"></a>Konfigurera din anslutning

Plats-till-plats-anslutningar till ett lokalt nätverk kräver en VPN-enhet. VPN-enheten som du konfigurerar kallas anslutning. Om du vill konfigurera din anslutning behöver du:

- En delad nyckel. Den här nyckeln är samma delade nyckel som du anger när du skapar en plats-till-plats-VPN-anslutning. I vårt exempel använder vi en enkel delad nyckel. Vi rekommenderar att du skapar och använder en mer komplex nyckel.
- Den offentliga IP-adressen för din virtuella nätverksgateway. Du kan visa den offentliga IP-adressen genom att använda Azure Portal, PowerShell eller CLI. Om du vill hitta den offentliga IP-adressen för din VPN-gateway med hjälp av Azure Portal går du till virtuella nätverksgateway och väljer sedan namnet på din gateway.

Använd följande steg för att skapa en VPN-anslutning från plats till plats mellan din virtuella nätverksgateway och din lokala VPN-enhet.

1. I Azure Portal väljer du **+ skapa en resurs**.
2. Sök efter **anslutningar**.
3. I **resultat** väljer du **anslutningar**.
4. Välj **skapa** på **anslutning**.
5. Konfigurera följande inställningar på **Skapa anslutning**:

    - **Anslutnings typ**: Välj plats-till-plats (IPSec).
    - **Resurs grupp**: Välj din test resurs grupp.
    - **Virtual Network Gateway**: Välj den virtuella nätverksgateway som du skapade.
    - **Lokal** nätverksgateway: Välj den lokala nätverksgateway som du skapade.
    - **Anslutnings namn**: det här namnet fylls i automatiskt med värdena från de två gatewayerna.
    - **Delad nyckel**: det här värdet måste matcha det värde som du använder för din lokala VPN-enhet. I exempel exemplet använder "vi abc123", men du bör använda något mer komplicerat. Det viktiga är att det här värdet *måste* vara samma värde som du anger när du konfigurerar VPN-enheten.
    - Värdena för **prenumeration**, **resurs grupp** och **plats** är fasta.

6. Välj **OK** för att skapa anslutningen.

Du kan se anslutningen på sidan **anslutningar** för den virtuella Nätverksgatewayen. Status kommer att gå från *okänd* till att *ansluta* och sedan *lyckas*.

## <a name="next-steps"></a>Nästa steg

- Mer information om moln mönster i Azure finns i [design mönster för molnet](/azure/architecture/patterns).
