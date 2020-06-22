---
title: Design överväganden för Hybrid appar i Azure och Azure Stack hubb
description: Lär dig mer om design överväganden när du skapar en hybrid app för det intelligenta molnet och intelligent gräns, inklusive placering, skalbarhet, tillgänglighet och återhämtning.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4fd52f76baad8059e130adfc01cdd0152b40a510
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911955"
---
# <a name="hybrid-app-design-considerations"></a>Design överväganden för Hybrid appar

Microsoft Azure är det enda konsekventa hybrid molnet. Det gör att du kan återanvända dina utvecklings investeringar och göra det möjligt för appar som kan sträcka sig över globala Azure, de suveräna Azure-molnen och Azure Stack, som är en förlängning av Azure i ditt data Center. Appar som sträcker sig över moln kallas även *hybrid appar*.

I [*guiden Azure Application arkitektur*](https://docs.microsoft.com/azure/architecture/guide) beskrivs en strukturerad metod för att utforma appar som är skalbara, elastiska och hög tillgängliga. De överväganden som beskrivs i [*Azure Application arkitektur guiden*](https://docs.microsoft.com/azure/architecture/guide) gäller även för appar som är utformade för ett enda moln och för appar som omfattar moln.

Den här artikeln innehåller en utökning av de [*pelare för program varu kvaliteten*](https://docs.microsoft.com/azure/architecture/guide/pillars) som diskuteras i [*Azure Application*](https://docs.microsoft.com/azure/architecture/guide/) [ *arkitektur guide*,](https://docs.microsoft.com/azure/architecture/guide/) som fokuserar på att utforma hybrid appar. Dessutom lägger vi till en *placerings* pelare eftersom hybrid appar inte är exklusiva till ett moln eller ett lokalt Data Center.

Hybrid scenarier varierar kraftigt med de resurser som är tillgängliga för utveckling och omfattar bland annat geografi, säkerhet, Internet åtkomst och andra överväganden. Även om den här hand boken inte kan räkna upp dina speciella överväganden kan du ange några viktiga rikt linjer och metod tips som du kan följa. Att designa, konfigurera, distribuera och underhålla en hybrid program arkitektur omfattar många design överväganden som kanske inte är kända för dig.

Det här dokumentet syftar till att samla in möjliga frågor som kan uppstå när du implementerar hybrid program och tillhandahåller överväganden (dessa pelare) och bästa metoder för att arbeta med dem. Genom att åtgärda dessa frågor under Design fasen kan du undvika de problem som kan uppstå i produktionen.

De är i stort sett frågor som du behöver tänka på innan du skapar en hybrid app. För att komma igång måste du göra följande:

- Identifiera och utvärdera app-komponenterna.
- Utvärdera program komponenter mot pelaren.

## <a name="evaluate-the-app-components"></a>Utvärdera app-komponenterna

Varje komponent i en app har sin egen specifika roll i den större appen och bör granskas med alla design överväganden. Varje komponents krav och funktioner bör mappas till dessa överväganden som hjälper dig att fastställa appens arkitektur.

Skapa din app i dess komponenter genom att studera appens arkitektur och fastställa vad den består av. Komponenter kan också innehålla andra appar som din app interagerar med. När du identifierar komponenterna kan du utvärdera dina avsedda hybrid åtgärder enligt deras egenskaper genom att ställa följande frågor:

- Vad är syftet med komponenten?
- Vilka är sambanden mellan komponenterna?

En app kan till exempel ha en klient del och en server del definierad som två komponenter. I ett hybrid scenario finns klient delen i ett moln och Server delen är i det andra. Appen tillhandahåller kommunikations kanaler mellan klient delen och användaren, och även mellan klient delen och Server delen.

En app-komponent definieras av många formulär och scenarier. Den viktigaste uppgiften identifierar dem och deras moln eller lokala plats.

De vanliga app-komponenter som ska tas med i inventeringen visas i tabell 1.

### <a name="table-1-common-app-components"></a>Tabell 1. Vanliga app-komponenter

| **Komponent** | **Vägledning för hybrid program** |
| ---- | ---- |
| Klientanslutningar | Din app (på valfri enhet) kan komma åt användare på olika sätt, från en enda start punkt, inklusive följande sätt:<br>-En klient-server-modell som kräver att användaren har en-klient installerad för att fungera med appen. En serverbaserad app som nås från en webbläsare.<br>– Klient anslutningar kan innehålla meddelanden när anslutningen är bruten eller aviseringar när nätverks växlings avgifter kan tillkomma. |
| Autentisering  | Autentisering kan krävas för en användare som ansluter till appen, eller från en komponent som ansluter till en annan. |
| API:er  | Du kan ge utvecklare program mässig åtkomst till din app med API-uppsättningar och klass bibliotek och tillhandahålla ett anslutnings gränssnitt baserat på Internet standarder. Du kan också använda API: er för att dela upp en app i oberoende operativ logiska enheter. |
| Tjänster  | Du kan använda kortfattad-tjänster för att tillhandahålla funktioner för en app. En tjänst kan vara den motor som appen körs på. |
| Köer | Du kan använda köer för att organisera status för livs cykel och tillstånd för appens komponenter. Dessa köer kan tillhandahålla funktioner för meddelande hantering, meddelanden och buffring för att prenumerera på parter. |
| Datalagring | En app kan vara tillstånds lös eller tillstånds känslig. Tillstånds känsliga appar behöver data lagring som kan uppfyllas av många olika format och volymer. |
| Cachelagring av data  | En komponent för cachelagring av data i din design kan hantera svars tids fördröjningar och spela upp en roll när de utlöser molnet. |
| Datainhämtning | Data kan skickas till en app på många sätt, från användare som har skickat in ett webb formulär till kontinuerligt hög volym data flöde. |
| Databearbetning | Dina data bearbetnings uppgifter (till exempel rapporter, analyser, batch-exporter och data omvandling) kan antingen bearbetas vid källan eller avlastas på en separat komponent med hjälp av en kopia av data. |

## <a name="assess-app-components-for-pillars"></a>Utvärdera app-komponenter för pelare

Utvärdera egenskaperna för varje pelare för varje komponent. När du utvärderar varje komponent med alla pelare visas frågor som du kanske inte har övervägt som kan bli kända för dig som påverkar hybrid appens design. Att tänka på detta kan leda till värde när du optimerar din app. Tabell 2 innehåller en beskrivning av varje pelare som relaterar till hybrid program.

### <a name="table-2-pillars"></a>Tabell 2. Pelare

| **Grundpelare** | **Beskrivning** |
| ----------- | --------------------------------------------------------- |
| Placering  | Den strategiska placeringen av komponenter i hybrid program. |
| Skalbarhet  | Systemets förmåga att hantera ökad belastning. |
| Tillgänglighet  | Den andel av tiden som en hybrid app fungerar. |
| Återhämtning | Möjligheten för en hybrid app att återställa. |
| Hanterbarhet | Driftsprocesser som håller ett system igång och i produktion. |
| Säkerhet | Skydda hybrid program och data från hot. |

## <a name="placement"></a>Placering

En hybrid app har en placering som är till för data centret.

Placering är den viktigaste uppgiften att placera komponenter så att de bäst kan betjäna en hybrid app. Efter definition, omfattar hybrid appar platser, till exempel från lokala platser till molnet och mellan olika moln. Du kan placera komponenter i appen på moln på två sätt:

- **Vertikala hybrid appar**  
    App-komponenter distribueras på olika platser. Varje enskild komponent kan bara ha flera instanser på en enda plats.

- **Horisontella hybrid appar**  
    App-komponenter distribueras på olika platser. Varje enskild komponent kan ha flera instanser som sträcker sig över flera platser.

    Vissa komponenter kan vara medvetna om deras plats medan andra inte har någon kunskap om deras plats och placering. Den här virtuousness kan uppnås med ett abstraktions lager. Det här lagret, med ett moderna program ramverk som mikrotjänster, kan definiera hur appen ska servas genom placeringen av app-komponenter som körs på noder i molnet.

### <a name="placement-checklist"></a>Placerings check lista

**Verifiera nödvändiga platser.** Kontrol lera att appen eller någon av dess komponenter krävs för att kunna användas i, eller Kräv certifiering för, ett särskilt moln. Detta kan innefatta suveränitets krav från företaget eller dikteras enligt lag. Ta också reda på om det krävs några lokala åtgärder för en viss plats eller nationella inställningar.

**Fastställa anslutnings beroenden.** Obligatoriska platser och andra faktorer kan diktera anslutnings beroenden mellan dina komponenter. När du placerar komponenterna bestämmer du optimal anslutning och säkerhet för kommunikation mellan dem. Alternativen är [ *VPN*,](https://docs.microsoft.com/azure/vpn-gateway/) [ *ExpressRoute*](https://docs.microsoft.com/azure/expressroute/) och [ *hybridanslutningar*.](https://docs.microsoft.com/azure/app-service/app-service-hybrid-connections)

**Utvärdera plattforms funktioner.** För varje app-komponent, se om den obligatoriska resurs leverantören för app-komponenten är tillgänglig i molnet och om bandbredden kan hantera det förväntade data flödet och latens krav.

**Planera för portabilitet.** Använd moderna app-ramverk, t. ex. behållare eller mikrotjänster, för att planera för flytt av åtgärder och för att förhindra tjänst beroenden.

**Fastställ krav för data suveränitet.** Hybrid appar är avsedd för att kunna isolera data isolering, till exempel på ett lokalt Data Center. Granska placeringen av dina resurser för att optimera framgången för det här kravet.

**Planera för svars tid.** Mellan moln åtgärder kan introducera fysiskt avstånd mellan app-komponenterna. Ta reda på kraven för att tillgodose eventuella svars tider.

**Styr trafik flöden.** Hantera högsta användning och lämplig och säker kommunikation för personliga identifierbara informations data när de används av klient delen i ett offentligt moln.

## <a name="scalability"></a>Skalbarhet

Skalbarhet är möjligheten för ett system att hantera ökad belastning på en app, vilket kan variera över tid som andra faktorer och tvingar till att påverka mål gruppens storlek, förutom appens storlek och omfattning.

För kärn diskussionen om denna pelare, se [*skalbarhet*](https://docs.microsoft.com/azure/architecture/guide/pillars#scalability) i de fem grundarna av arkitektur expert.

Med en horisontell skalnings metod för Hybrid appar kan du lägga till fler instanser för att möta efter frågan och sedan inaktivera dem under tysta perioder.

I hybrid scenarier kräver skalning av enskilda komponenter ytterligare överväganden när komponenterna sprids över moln. Skalning av en del av appen kan kräva skalning av en annan. Om antalet klient anslutningar exempelvis ökar, men appens webb tjänster inte skalas ut korrekt, kan belastningen på databasen fylla appen.

Vissa app-komponenter kan skalas linjärt, medan andra har skalnings beroenden och kan begränsas till den omfattning som de kan skala. Till exempel har en VPN-tunnel som tillhandahåller hybrid anslutning för platserna för program komponenter en gräns för bandbredden och fördröjningen som kan skalas till. Hur är komponenter i appen skalade för att säkerställa att dessa krav uppfylls?

### <a name="scalability-checklist"></a>Checklista för skalbarhet

**Fastställa tröskelvärden för skalning.** Om du vill hantera olika beroenden i din app bestämmer du i vilken utsträckning program komponenter i olika moln kan skalas oberoende av varandra, samtidigt som de fortfarande uppfyller kraven för att köra appen. Hybrid appar behöver ofta skala vissa områden i appen för att hantera en funktion när den interagerar och påverkar resten av appen. Om du till exempel överskrider ett antal klient dels instanser kan det krävas skalning av Server delen.

**Definiera skalnings scheman.** De flesta appar har upptagna perioder, så du måste sätta samman de högsta tiderna i scheman för att samordna optimal skalning.

**Använd ett centraliserat övervaknings system.** Plattforms övervaknings funktioner kan tillhandahålla automatisk skalning, men hybrid appar behöver ett centraliserat övervaknings system som aggregerar system hälsan och belastningen. Ett centraliserat övervaknings system kan initiera skalning av en resurs på en plats och skala beroende på resurs på en annan plats. Dessutom kan ett centralt övervaknings system spåra vilka moln som autoskalar resurser och vilka moln som inte är det.

**Utnyttja funktioner för automatisk skalning (som tillgängliga).** Om funktionerna för automatisk skalning är en del av arkitekturen implementerar du autoskalning genom att ange tröskelvärden som definierar när en app-komponent behöver skalas upp, ut, ned eller in. Ett exempel på automatisk skalning är en klient anslutning som är autoskalad i ett moln för att hantera ökad kapacitet, men som gör att andra beroenden i appen sprids över olika moln, så att de också skalas. Funktionerna för automatisk skalning av dessa beroende komponenter måste vara specifika.

Om autoskalning inte är tillgängligt kan du överväga att implementera skript och andra resurser för att hantera manuell skalning, som utlöses av tröskelvärden i det centraliserade övervaknings systemet.

**Bestäm förväntad belastning efter plats.** Hybrid appar som hanterar klient begär Anden kanske huvudsakligen förlitar sig på en enda plats. När belastningen på klient begär Anden överskrider ett tröskelvärde kan ytterligare resurser läggas till på en annan plats för att distribuera belastningen på inkommande begär Anden. Se till att klient anslutningarna kan hantera de ökade belastningarna och även fastställa eventuella automatiserade procedurer för klient anslutningarna för att hantera belastningen.

## <a name="availability"></a>Tillgänglighet

Tillgänglighet är den tid som ett system fungerar och fungerar. Tillgänglighet mäts i procent av drift tiden. App-fel, infrastruktur problem och system belastning kan minska tillgängligheten.

För kärn diskussionen om denna pelare, se [*tillgänglighet*](/azure/architecture/framework/) i de fem grundarna i arkitektur expert.

### <a name="availability-checklist"></a>Checklista för tillgänglighet

**Tillhandahålla redundans för anslutning.** Hybrid appar kräver anslutning mellan de moln som appen sprider sig över. Du har möjlighet att välja tekniker för Hybrid anslutning, så förutom ditt primära teknik alternativ bör du använda en annan teknik för att tillhandahålla redundans med funktioner för automatisk redundans om den primära tekniken inte fungerar.

**Klassificera fel domäner.** Feltoleranta appar kräver flera fel domäner. Fel domäner hjälper till att isolera fel punkten, t. ex. om en enskild hård disk kraschar lokalt, om en top-of-rack-växel slutar fungera, eller om det fullständiga data centret inte är tillgängligt. I en hybrid app kan en plats klassificeras som en fel domän. Med fler tillgänglighets krav, desto mer behöver du utvärdera hur en enskild feldomän ska klassificeras.

**Klassificera uppgraderings domäner.** Uppgraderings domäner används för att säkerställa att instanser av app-komponenter är tillgängliga, medan andra instanser av samma komponent betjänas med uppdateringar eller funktions uppgraderingar. Som med fel domäner kan uppgraderings domäner klassificeras efter deras placering på olika platser. Du måste bestämma om en app-komponent ska kunna uppgraderas på en plats innan den uppgraderas på en annan plats, eller om andra domänautentiseringsuppgifter krävs. En enskild plats kan ha flera uppgraderings domäner.

**Spåra instanser och tillgänglighet.** Program komponenter med hög tillgänglighet kan vara tillgängliga via belastnings utjämning och synkron datareplikering. Du måste bestämma hur många instanser som kan vara offline innan tjänsten avbryts.

**Implementera själv återställning.** I händelse av ett problem som orsakar ett avbrott i appens tillgänglighet kan en identifiering av ett övervaknings system initiera Självåterställande aktiviteter till appen, till exempel tömma den felande instansen och distribuera den igen. Förmodligen kräver detta en central övervaknings lösning som är integrerad med en hybrid kontinuerlig integrering och en pipeline för kontinuerlig leverans (CI/CD). Appen är integrerad med ett övervaknings system för att identifiera problem som kan kräva omdistribution av en app-komponent. Övervaknings systemet kan även utlösa hybrid CI/CD för att omdistribuera app-komponenten och eventuellt andra beroende komponenter på samma eller andra platser.

**Underhåll service nivå avtal (service avtal).** Tillgänglighet är viktigt för alla avtal för att upprätthålla anslutningarna till de tjänster och appar som du har med kunderna. Varje plats som din hybrid app är beroende av kan ha ett eget service avtal. Dessa olika service avtal kan påverka det övergripande service avtalet för din hybrid app.

## <a name="resiliency"></a>Återhämtning

Återhämtning är möjligheten för en hybrid app och ett system att återställa efter fel och fortsätta att fungera. Målet med återhämtning är att returnera appen till ett fullständigt fungerande tillstånd när ett fel har inträffat. Återhämtnings strategier omfattar lösningar som säkerhets kopiering, replikering och haveri beredskap.

För kärn diskussionen om den här pelaren, se [*återhämtning*](https://docs.microsoft.com/azure/architecture/guide/pillars#resiliency) i de fem grundarna av arkitektur expert.

### <a name="resiliency-checklist"></a>Checklista för elasticitet

**Återställ katastrofer för haveri beredskap.** Haveri beredskap i ett moln kan kräva ändringar i app-komponenter i ett annat moln. Om en eller flera komponenter från ett moln har redundansväxlats till en annan plats, antingen inom samma moln eller till ett annat moln, måste de beroende komponenterna vara medvetna om dessa ändringar. Detta inkluderar även anslutnings beroenden. Återhämtning kräver en fullständigt testad program återställnings plan för varje moln.

**Etablera återställnings flöde.** En effektiv design av återställnings flödet har utvärderat app-komponenter för att kunna hantera buffertar, nya försök, försöka överföra misslyckad data överföring igen och, om det behövs, återgå till en annan tjänst eller ett annat arbets flöde. Du måste bestämma vilken mekanism för säkerhets kopiering som ska användas, vad dess återställnings procedur omfattar och hur ofta den testas. Du bör också fastställa frekvensen för både stegvisa och fullständiga säkerhets kopieringar.

**Testa delvis återställningar.** En delvis återställning för en del av appen kan ge åter betalning till användare som inte är tillgängliga. Den här delen av planen bör se till att en partiell återställning inte har några sido effekter, till exempel en säkerhets kopierings-och återställnings tjänst som interagerar med appen, så att den stängs av korrekt innan säkerhets kopieringen görs.

**Bestäm haveri beredskap för instigators och tilldela ansvar.** En återställnings plan bör beskriva vem och vilka roller som kan initiera säkerhets kopierings-och återställnings åtgärder utöver vad som kan säkerhets kopie ras och återställas.

**Jämför tröskelvärden för automatisk återställning med haveri beredskap.** Fastställ en Apps själv återställnings funktion för automatisk återställnings initiering och den tid som krävs för att en Apps själv återställning ska anses vara en misslyckad eller lyckad. Avgör tröskelvärdena för varje moln.

**Verifiera tillgänglighet för återhämtnings funktioner.** Fastställ tillgänglighet för återhämtnings funktioner och-funktioner för varje plats. Om en plats inte tillhandahåller de funktioner som krävs bör du överväga att integrera platsen i en centraliserad tjänst som tillhandahåller återhämtnings funktionerna.

**Fastställ stillestånds tid.** Fastställ förväntad stillestånds tid på grund av underhåll av appen som helhet och som app-komponenter.

**Dokumentera fel söknings procedurer.** Definiera fel söknings procedurer för omdistribution av resurser och app-komponenter.

## <a name="manageability"></a>Hanterbarhet

Överväganden för hur du hanterar dina hybrid appar är viktiga för att designa din arkitektur. En väl hanterad hybrid app tillhandahåller en infrastruktur som kod som möjliggör integrering av konsekvent app-kod i en gemensam utvecklings pipeline. Genom att implementera konsekventa systemomfattande och enskilda tester av ändringar i infrastrukturen, kan du säkerställa en integrerad distribution om ändringarna klarar testerna, så att de kan slås samman till käll koden.

För kärn diskussionen om denna pelare, se [*DevOps*](/azure/architecture/framework/#devops) i de fem grundarna i arkitektur expert.

### <a name="manageability-checklist"></a>Check lista för hantering

**Implementera övervakning.** Använd ett centraliserat övervaknings system av app-komponenter som sprids över moln för att tillhandahålla en sammanställd vy över deras hälsa och prestanda. Det här systemet omfattar övervakning av både app-komponenter och relaterade plattforms funktioner.

Bestäm vilka delar av appen som kräver övervakning.

**Koordinera principer.** Varje plats som en hybrid app omfattar kan ha en egen princip som täcker tillåtna resurs typer, namngivnings konventioner, taggar och andra kriterier.

**Definiera och använda roller.** Som databas administratör måste du bestämma vilka behörigheter som krävs för olika personer (t. ex. en app-ägare, en databas administratör och en slutanvändare) som behöver åtkomst till program resurser. Dessa behörigheter måste konfigureras på resurserna och i appen. Med ett system med rollbaserad åtkomst kontroll (RBAC) kan du ange dessa behörigheter för app-resurserna. Dessa behörigheter är utmanande när alla resurser distribueras i ett enda moln, men kräver ännu mer uppmärksamhet när resurserna sprids över moln. Behörigheter för resurser som anges i ett moln gäller inte för resurser som angetts i ett annat moln.

**Använd CI/CD-pipeliner.** En pipeline för kontinuerlig integrering och kontinuerlig utveckling (CI/CD) kan ge en konsekvent process för att skapa och distribuera appar som sträcker sig över flera moln, och för att ge kvalitets säkring för sin infrastruktur och app. Den här pipelinen gör att infrastrukturen och appen kan testas i ett moln och distribueras i ett annat moln. Pipelinen gör att du kan distribuera vissa komponenter i din hybrid app till ett moln och andra komponenter till ett annat moln, vilket i grunden utgör grunden för distribution av hybrid program. Ett CI/CD-system är viktigt för att hantera program komponenterna för beroenden för varandra under installationen, till exempel att webbappen behöver en anslutnings sträng till-databasen.

**Hantera livs cykeln.** Eftersom resurser i en hybrid app kan sträcka sig över platser måste varje enskild platss hanterings kapacitet för livs cykel sammanställas till en enda livs cykel hanterings enhet. Överväg hur de skapas, uppdateras och tas bort.

**Undersök fel söknings strategier.** Fel sökning av en hybrid app omfattar fler app-komponenter än samma app som körs i ett enda moln. Förutom anslutningen mellan molnen körs appen på två plattformar i stället för en. En viktig uppgift i fel sökning av hybrid program är att undersöka den sammanställda hälso tillstånds-och prestanda övervakningen av app-komponenterna.

## <a name="security"></a>Säkerhet

Säkerhet är ett av de viktigaste övervägandena för alla molnappar och det blir ännu mer viktigt för Hybrid molnappar.

För kärn diskussionen om den här pelaren, se [*säkerhet*](https://docs.microsoft.com/azure/architecture/guide/pillars#security) i de fem grundarna av arkitektur expert.

### <a name="security-checklist"></a>Säkerhetskontrollista

**Anta överträdelse.** Om en del av appen komprometteras, kontrollerar du att det finns lösningar på plats för att minimera spridningen av överträdelsen, inte bara inom samma plats, utan även mellan platser.

**Övervaka tillåten nätverks åtkomst.** Fastställ principerna för nätverks åtkomst för appen, till exempel endast att komma åt appen från ett särskilt undernät och endast tillåta lägsta portar och protokoll mellan de komponenter som krävs för att appen ska fungera korrekt.

**Använd robust autentisering.** Ett robust autentiseringsschema är avgörande för appens säkerhet. Överväg att använda en federerad identitets leverantör som tillhandahåller funktioner för enkel inloggning och använder ett eller flera av följande scheman: användar namn och lösen ord inloggning, offentliga och privata nycklar, två faktorer eller Multi-Factor Authentication och betrodda säkerhets grupper. Fastställ lämpliga resurser för att lagra känsliga data och andra hemligheter för app-autentisering förutom certifikat typer och deras krav.

**Använd kryptering.** Identifiera vilka områden i appen som använder kryptering, till exempel för data lagring eller klient kommunikation och åtkomst.

**Använd säkra kanaler.** En säker kanal över molnen är viktig för att tillhandahålla säkerhets-och verifierings kontroller, real tids skydd, karantän och andra tjänster i molnet.

**Definiera och använda roller.** Implementera roller för resurs konfiguration och åtkomst med enstaka identitet över moln. Fastställ vilka RBAC-krav (rollbaserad åtkomst kontroll) som krävs för appen och dess plattforms resurser.

**Granska systemet.** System övervakning kan logga och samla in data från både app-komponenter och relaterade moln plattforms åtgärder.

## <a name="summary"></a>Sammanfattning

Den här artikeln innehåller en check lista över objekt som är viktiga att tänka på när du redigerar och utformar dina hybrid program. Genom att granska dessa pelare innan du distribuerar din app kan du inte köra dessa frågor i produktions avbrott och eventuellt kräva att du går tillbaka till din design.

Det kan verka som en tids krävande uppgift i förväg, men du får enkelt avkastning på investeringen om du utformar appen baserat på dessa pelare.

## <a name="next-steps"></a>Nästa steg

Mer information finns i följande resurser:

- [Hybridmoln](https://azure.microsoft.com/overview/hybrid-cloud/)
- [Hybrid molnappar](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [Utveckla Azure Resource Manager-mallar för molnkonsekvens](https://aka.ms/consistency)
