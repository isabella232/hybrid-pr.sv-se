---
title: DevOps-mönstret i Azure Stack Hub
description: Lär dig mer om DevOps-mönstret så att du kan garantera konsekvens mellan distributioner i Azure och Azure Stack hubben.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 306cc9604a8e919724f9f76b7e5122d534d2d1ae
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911958"
---
# <a name="devops-pattern"></a>DevOps-mönster

Kod från en enda plats och distribuera till flera mål i utvecklings-, test-och produktions miljöer som kan finnas i ditt lokala data Center, privata moln eller i det offentliga molnet.

## <a name="context-and-problem"></a>Kontext och problem

Kontinuitet, säkerhet och tillförlitlighet för program distribution är avgörande för organisationer och viktiga för utvecklings team.

Appar kräver ofta omstrukturerad kod för att köras i varje mål miljö. Det innebär att en app inte är helt portabel. Det måste uppdateras, testas och verifieras när det flyttas genom varje miljö. Till exempel måste kod som skrivits i en utvecklings miljö skrivas om för att fungera i en test miljö och skrivas om när den slutligen hamnar i en produktions miljö. Dessutom är den här koden särskilt knuten till värden. Detta ökar kostnaden och komplexiteten med att underhålla din app. Varje version av appen är kopplad till varje miljö. Ökad komplexitet och duplicering ökar risken för säkerhet och kod kvalitet. Dessutom kan inte koden distribueras på ett enkelt sätt när du tar bort återställnings misslyckade värdar eller distribuerar ytterligare värdar för att hantera ökningar i efter frågan.

## <a name="solution"></a>Lösning

Med DevOps-mönstret kan du bygga, testa och distribuera en app som körs på flera moln. Det här mönstret enhets praxis för kontinuerlig integrering och kontinuerlig leverans. Med kontinuerlig integrering skapas och testas kod varje gång en grupp medlem genomför en ändring i versions kontrollen. Kontinuerlig leverans automatiserar varje steg från en version till en produktions miljö. Tillsammans skapar de här processerna en versions process som stöder distribution i olika miljöer. Med det här mönstret kan du utkast till din kod och sedan distribuera samma kod till en lokal miljö, olika privata moln och offentliga moln. Skillnader i miljön kräver en ändring i en konfigurations fil i stället för ändringar i koden.

![DevOps-mönster](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Med en enhetlig uppsättning utvecklingsverktyg i lokala, privata moln och i offentliga moln miljöer kan du implementera en metod för kontinuerlig integrering och kontinuerlig leverans. Appar och tjänster som distribueras med DevOps-mönstret är utbytbara och kan köras på någon av dessa platser, vilket drar nytta av funktioner och funktioner i lokala och offentliga moln.

Genom att använda en DevOps release-pipeline kan du:

- Starta en ny version baserat på kod skrivningar till en enda lagrings plats.
- Distribuera automatiskt den nyskapade koden till det offentliga molnet för testning av användar godkännande.
- Distribuera automatiskt till ett privat moln när din kod har klarat testet.

## <a name="issues-and-considerations"></a>Problem och överväganden

DevOps-mönstret är avsett att garantera konsekvens mellan distributioner oavsett mål miljö. Funktionerna varierar dock mellan moln miljöer och lokala miljöer. Tänk på följande:

- Finns det funktioner, slut punkter, tjänster och andra resurser i distributionen som är tillgängliga på mål distributions platserna?
- Lagras konfigurations artefakter på platser som är tillgängliga i molnet?
- Kommer distributions parametrar att fungera i alla mål miljöer?
- Är resurs specifika egenskaper tillgängliga i alla mål moln?

Mer information finns i [utveckla Azure Resource Manager mallar för moln konsekvens](https://docs.microsoft.com/azure/azure-resource-manager/templates-cloud-consistency).

Tänk också på följande när du bestämmer hur du ska implementera det här mönstret:

### <a name="scalability"></a>Skalbarhet

Distributions Automation system är nyckel kontroll punkten i DevOps-mönstren. Implementeringar kan variera. Valet av rätt server storlek beror på storleken på den förväntade arbets belastningen. VM kostar mer att skala än behållare. Men om du vill använda containrar för skalning måste byggprocessen köras med containrar.

### <a name="availability"></a>Tillgänglighet

Tillgängligheten i DevPattern innebär att kunna återställa all statusinformation som är kopplad till ditt arbets flöde, till exempel test resultat, kod beroenden eller andra artefakter. Överväg två vanliga mått vid utvärdering av tillgänglighetskraven:

- Mål för återställnings tid (RTO) anger hur länge du kan gå utan system.

- Återställnings punkt mål (återställnings punkt mål) anger hur mycket data du kan få att förlora om ett avbrott i tjänsten påverkar systemet.

I praktiken, RTO och återställnings punkt innebär redundans och säkerhets kopiering. I det globala Azure-molnet är tillgängligheten inte en fråga om maskin varu återställning – det är en del av Azure, men du kan i stället se till att du behåller tillstånd för dina DevOps-system. På Azure Stack hubb kan maskin varu återställning vara ett övervägande.

En annan viktig hänsyn när du utformar systemet som används för distributions automatisering är åtkomst kontroll och korrekt hantering av de rättigheter som krävs för att distribuera tjänster till moln miljöer. Vilka rättigheter behövs för att skapa, ta bort eller ändra distributioner? Till exempel krävs det vanligt vis en uppsättning rättigheter för att skapa en resurs grupp i Azure och en annan för att distribuera tjänster i resurs gruppen.

### <a name="manageability"></a>Hanterbarhet

Utformningen av alla system som baseras på DevOps-mönstret måste överväga automatisering, loggning och aviseringar för varje tjänst i portföljen. Använd delade tjänster, ett program team eller både och och spåra även säkerhets principer och styrning.

Distribuera produktions miljöer och utvecklings-och test miljöer i separata resurs grupper i Azure eller Azure Stack Hub. Sedan kan du övervaka varje Miljös resurser och summera fakturerings kostnaderna per resurs grupp. Du kan också ta bort resurser som en uppsättning, vilket är användbart för test distributioner.

## <a name="when-to-use-this-pattern"></a>När du ska använda det här mönstret

Använd det här mönstret om:

- Du kan utveckla kod i en miljö som uppfyller behoven hos dina utvecklare och distribuera till en miljö som är speciell för din lösning, där det kan vara svårt att utveckla ny kod.
- Du kan använda kod och verktyg som dina utvecklare vill, så länge de kan följa den kontinuerliga integreringen och den kontinuerliga leverans processen i DevOps-mönstret.

Det här mönstret rekommenderas inte:

- Om du inte kan automatisera infrastruktur, etablering av resurser, konfiguration, identitet och säkerhets uppgifter.
- Om team inte har åtkomst till hybrid moln resurser för att implementera en metod för kontinuerlig integrering/kontinuerlig utveckling (CI/CD).

## <a name="next-steps"></a>Nästa steg

Mer information om ämnen som introduceras i den här artikeln:

- I [dokumentationen för Azure DevOps](/azure/devops) finns mer information om Azure DevOps och relaterade verktyg, inklusive Azure databaser och Azure-pipelines.
- Se [Azure Stacks familj med produkter och lösningar](/azure-stack) för att lära dig mer om hela portföljen med produkter och lösningar.

När du är redo att testa lösnings exemplet fortsätter du med [distributions guiden för DevOps hybrid CI/CD-lösning](https://aka.ms/hybriddevopsdeploy). Distributions guiden innehåller steg-för-steg-instruktioner för att distribuera och testa dess komponenter. Du lär dig hur du distribuerar en app till Azure och Azure Stack hubb med en pipeline för Hybrid kontinuerlig integrering/kontinuerlig leverans (CI/CD).
