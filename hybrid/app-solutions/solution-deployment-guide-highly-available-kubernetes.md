---
title: Distribuera Kubernetes-kluster med hög tillgänglighet på Azure Stack Hub
description: Lär dig hur du distribuerar en Kubernetes-kluster lösning för hög tillgänglighet med Azure och Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911739"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Distribuera ett Kubernetes-kluster med hög tillgänglighet på Azure Stack Hub

Den här artikeln visar hur du skapar en Kubernetes kluster miljö med hög tillgänglighet, som distribueras på flera Azure Stack Hubbs instanser, på olika fysiska platser.

I den här distributions hand boken för lösningen lär du dig att:

> [!div class="checklist"]
> - Ladda ned och Förbered AKS-motorn
> - Ansluta till den virtuella datorn med AKS motor hjälp
> - Distribuera ett Kubernetes-kluster
> - Ansluta till Kubernetes-klustret
> - Anslut Azure-pipeliner till Kubernetes-kluster
> - Konfigurera övervakning
> - Distribuera programmet
> - Autoskala program
> - Konfigurera Traffic Manager
> - Uppgradera Kubernetes
> - Skala Kubernetes

> [!Tip]  
> ![Hybrid pelare](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub är ett tillägg till Azure. Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.  
> 
> Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program. Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.

## <a name="prerequisites"></a>Krav

Innan du börjar med den här distributions guiden kontrollerar du att:

- Läs artikeln om [Kubernetes kluster mönster för hög tillgänglighet](pattern-highly-available-kubernetes.md) .
- Granska innehållet i [Companion GitHub-lagringsplatsen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), som innehåller ytterligare till gångar som refereras i den här artikeln.
- Ha ett konto som har åtkomst till [användar portalen för Azure Stack Hub](/azure-stack/user/azure-stack-use-portal), med minst [deltagar behörighet](/azure-stack/user/azure-stack-manage-permissions).

## <a name="download-and-prepare-aks-engine"></a>Ladda ned och Förbered AKS-motorn

AKS-motorn är en binärfil som kan användas från valfri Windows-eller Linux-värd som kan komma åt Azure Stack Hub Azure Resource Manager-slutpunkter. Den här guiden beskriver hur du distribuerar en ny Linux-(eller Windows) virtuell dator på Azure Stack Hub. Den kommer att användas senare när AKS-motorn distribuerar Kubernetes-klustren.

> [!NOTE]
> Du kan också använda en befintlig virtuell Windows-eller Linux-dator för att distribuera ett Kubernetes-kluster på Azure Stack hubb med AKS-motorn.

Steg för steg-processen och kraven för AKS-motorn beskrivs här:

* [Installera AKS-motorn på Linux i Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (eller med [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

AKS-motorn är ett hjälp verktyg för att distribuera och använda (ohanterade) Kubernetes kluster (i Azure och Azure Stack Hub).

Information och skillnader i AKS-motorn på Azure Stack Hub beskrivs här:

* [Vad är AKS-motorn på Azure Stack Hub?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [AKS-motor på Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (på GitHub)

Exempel miljön kommer att använda terraform för att automatisera distributionen av den virtuella AKS-motorn. Du hittar [information och kod i Companion GitHub-lagrings platsen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

Resultatet av det här steget är en ny resurs grupp på Azure Stack hubb som innehåller den virtuella datorn med AKS-motorns hjälp och relaterade resurser:

![AKS-motorns VM-resurser i Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Om du måste distribuera AKS-motorn i en frånkopplad Air-gapped-miljö granskar du [frånkopplade Azure Stack Hubbs instanser](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) för mer information.

I nästa steg ska vi använda den nyligen distribuerade virtuella AKS-motorn för att distribuera ett Kubernetes-kluster.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Ansluta till den virtuella datorn med AKS motor hjälp

Först måste du ansluta till den tidigare skapade virtuella datorn för AKS-motorn.

Den virtuella datorn ska ha en offentlig IP-adress och bör vara tillgänglig via SSH (port 22/TCP).

![Översikts sida för virtuell AKS-motor](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Du kan använda ett verktyg som du väljer som MobaXterm, SparaTillFil eller PowerShell i Windows 10 för att ansluta till en virtuell Linux-dator med SSH.

```console
ssh <username>@<ipaddress>
```

När du har anslutit kör du kommandot `aks-engine` . Gå till [AKS-motor versioner som stöds](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) och lär dig mer om AKS-motorn och Kubernetes-versioner.

![AKS-kommando rads exempel](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Distribuera ett Kubernetes-kluster

AKS-motorns hjälp verktyg för virtuella datorer har inte skapat något Kubernetes-kluster på vår Azure Stack hubb, men. Att skapa klustret är den första åtgärden som ska vidtas i AKS-motorns hjälp programs virtuella dator.

Steg för steg-processen beskrivs här:

* [Distribuera ett Kubernetes-kluster med AKS-motorn på Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

Slut resultatet för `aks-engine deploy` kommandot och förberedelserna i föregående steg är ett fullständigt aktuellt Kubernetes-kluster som distribueras till klient området för den första Azure Stack Hub-instansen. Själva klustret består av Azure IaaS-komponenter, t. ex. virtuella datorer, belastningsutjämnare, virtuella nätverk, diskar och så vidare.

![Kluster IaaS-komponenter Azure Stack hubb Portal](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure Load Balancer (K8s API-slutpunkt)
2) Arbetsnoder (agent-pool)
3) Huvudnoder

Klustret är nu igång och i nästa steg ska vi ansluta till den.

## <a name="connect-to-the-kubernetes-cluster"></a>Ansluta till Kubernetes-klustret

Nu kan du ansluta till det tidigare skapade Kubernetes-klustret, antingen via SSH (med SSH-nyckeln som anges som en del av distributionen) eller via `kubectl` (rekommenderas). Kommando rads verktyget Kubernetes `kubectl` är tillgängligt för Windows, Linux och MacOS [här](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Den är redan förinstallerad och konfigurerad på huvudnoderna i klustret.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Kör kubectl på huvud nod](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Vi rekommenderar inte att du använder huvudnoden som ett hopp för administrativa uppgifter. `kubectl`Konfigurationen lagras `.kube/config` på huvud noderna och på den virtuella AKS-motorn. Du kan kopiera konfigurationen till en administratörs dator med anslutning till Kubernetes-klustret och använda `kubectl` kommandot där. `.kube/config`Filen används också senare för att konfigurera en tjänst anslutning i Azure-pipelines.

> [!IMPORTANT]
> Se till att dessa filer är säkra eftersom de innehåller autentiseringsuppgifterna för ditt Kubernetes-kluster. En angripare med åtkomst till filen har tillräckligt med information för att få administratörs åtkomst till den. Alla åtgärder som görs med den inledande `.kube/config` filen görs med ett kluster administratörs konto.

Nu kan du prova olika kommandon med `kubectl` för att kontrol lera status för klustret.

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> Kubernetes har en egen _ *rollbaserad Access Control (RBAC) *-* modell som gör att du kan skapa detaljerade roll definitioner och roll bindningar. Detta är det bättre sättet att kontrol lera åtkomsten till klustret i stället för att bli av klustrets administratörs behörighet.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Ansluta Azure-pipeliner till Kubernetes-kluster

För att ansluta Azure-pipeliner till det nydistribuerade Kubernetes-klustret behöver vi dess Kube config ( `.kube/config` )-fil enligt beskrivningen i föregående steg.

* Anslut till en av huvudnoderna i ditt Kubernetes-kluster.
* Kopiera innehållet i `.kube/config` filen.
* Gå till Azure DevOps > projekt inställningar > tjänst anslutningar för att skapa en ny "Kubernetes"-tjänst anslutning (Använd KubeConfig som autentiseringsmetod)

> [!IMPORTANT]
> Azure-pipeliner (eller dess build-agenter) måste ha åtkomst till Kubernetes-API: et. Om det finns en Internet anslutning från Azure pipelines till Azure Stack Hub Kubernetes clusetr, måste du distribuera en lokal version av Azure pipelines för Azure.

När du distribuerar egna värdbaserade agenter för Azure-pipeliner kan du distribuera antingen på Azure Stack hubb eller på en dator med nätverks anslutning till alla nödvändiga hanterings slut punkter. Se informationen här:

* [Azure pipeline-agenter](/azure/devops/pipelines/agents/agents) i [Windows](/azure/devops/pipelines/agents/v2-windows) eller [Linux](/azure/devops/pipelines/agents/v2-linux)

Avsnittet om [överväganden för mönster distribution (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) innehåller ett besluts flöde som hjälper dig att förstå huruvida du ska använda Microsoft-värdbaserade agenter eller egna värdbaserade agenter:

[![besluts flöde, egna värdbaserade agenter](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

I den här exempel lösningen innehåller topologin en egen värd versions agent för varje Azure Stack Hub-instans. Agenten har åtkomst till Azure Stack Hub Management-slutpunkter och Kubernetes för kluster-API: er.

[![endast utgående trafik](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Den här designen uppfyller ett gemensamt föreskrivande krav, som endast ska ha utgående anslutningar från program lösningen.

## <a name="configure-monitoring"></a>Konfigurera övervakning

Du kan använda [Azure Monitor](/azure/azure-monitor/) för behållare för att övervaka behållarna i lösningen. Detta leder Azure Monitor till det AKS-distribuerade Kubernetes-klustret på Azure Stack Hub.

Det finns två sätt att aktivera Azure Monitor i klustret. Båda sätten kräver att du konfigurerar en Log Analytics arbets yta i Azure.

* [Metod ett](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) använder ett Helm-diagram
* [Metod två](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) som en del av AKS-motorns kluster specifikation

I prov sto pol Ogin används "metod ett", vilket gör att processen och uppdateringar kan installeras enklare.

I nästa steg behöver du en Azure LogAnalytics-arbetsyta (ID och nyckel), `Helm` (version 3) och `kubectl` på din dator.

Helm är en Kubernetes-paket hanterare som är tillgänglig som en binärfil som körs på macOS, Windows och Linux. Den kan hämtas här: [Helm.sh](https://helm.sh/docs/intro/quickstart/) Helm förlitar sig på Kubernetes-konfigurationsfilen som används för `kubectl` kommandot.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Med det här kommandot installeras Azure Monitor-agenten på ditt Kubernetes-kluster:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Hanterings agenten för Operations Management Suite (OMS) på ditt Kubernetes-kluster kommer att skicka övervaknings data till din Azure Log Analytics-arbetsyta (med utgående HTTPS). Nu kan du använda Azure Monitor för att få djupare insikter om dina Kubernetes-kluster på Azure Stack Hub. Den här designen är ett kraftfullt sätt att demonstrera kraften i analyser som kan distribueras automatiskt med ditt programs kluster.

[![Azure Stack hubb kluster i Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Azure Monitor kluster information](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Om Azure Monitor inte visar några Azure Stack Hub-data kontrollerar du att du har följt anvisningarna för [hur du lägger till AzureMonitor-Containers lösning i en Azure Loganalytics-arbetsyta](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) noggrant.

## <a name="deploy-the-application"></a>Distribuera programmet

Innan du installerar vårt exempel program finns det ett annat steg att konfigurera nginx-based ingångs styrenheten på vårt Kubernetes-kluster. Ingångs styrenheten används som en Layer 7-belastningsutjämnare för att dirigera trafik i vårt kluster baserat på värd, sökväg eller protokoll. Nginx – ingress är tillgängligt som ett Helm-diagram. Detaljerade anvisningar finns i [Helm-diagrammets GitHub-lagringsplats](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

Vårt exempel program paketeras också som ett Helm-diagram, som [Azure Monitoring Agent](#configure-monitoring) i föregående steg. Därför är det enkelt att distribuera programmet till vårt Kubernetes-kluster. Du kan hitta [Helm i Companion GitHub-lagrings platsen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)

Exempel programmet är ett program på tre nivåer som distribueras till ett Kubernetes-kluster på var och en av två Azure Stack Hub-instanser. Programmet använder en MongoDB-databas. Du kan lära dig mer om hur du hämtar data som replikeras över flera instanser i mönster [data och lagrings överväganden](pattern-highly-available-kubernetes.md#data-and-storage-considerations).

När du har distribuerat Helm-diagrammet för programmet visas alla tre nivåer av ditt program som visas som distributioner och tillstånds känsliga uppsättningar (för databasen) med en enda pod:

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

På sidan tjänster hittar du den nginx-baserade ingångs styrenheten och dess offentliga IP-adress:

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

Adressen "extern IP" är vår "program slut punkt". Det är hur användarna kommer att ansluta till att öppna programmet och kommer även att användas som slut punkt för nästa steg [konfigurera Traffic Manager](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Skala programmet Autoskala
Du kan också konfigurera den [vågräta Pod autoskalning](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) för att skala upp eller ned baserat på vissa mått som processor belastning. Följande kommando skapar en vågrät Pod-autoskalning som hanterar 1 till 10 repliker av poddar som kontrol leras av klassificeringen-webb distribution. HPA kommer att öka och minska antalet repliker (via distributionen) för att bibehålla en genomsnittlig CPU-belastning för alla poddar på 80%.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Du kan kontrol lera den aktuella statusen för autoskalning genom att köra:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Konfigurera Traffic Manager

För att distribuera trafik mellan två (eller flera) distributioner av programmet använder vi [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview). Azure Traffic Manager är en DNS-baserad trafikbelastnings utjämning i Azure.

> [!NOTE]
> Traffic Manager använder DNS för att dirigera klient begär anden till den lämpligaste tjänst slut punkten, baserat på en Traffic-routningsmetod och tillståndet för slut punkterna.

I stället för att använda Azure Traffic Manager kan du också använda andra globala lösningar för belastnings utjämning som finns lokalt. I exempel scenariot använder vi Azure Traffic Manager för att distribuera trafik mellan två instanser av vårt program. De kan köras på Azure Stack Hub-instanser på samma eller olika platser:

![lokal Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

I Azure konfigurerar vi Traffic Manager så att de pekar på de två olika instanserna av programmet:

[![TM-slutpunkt-profil](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Som du kan se pekar de två slut punkterna på de två instanserna av det distribuerade programmet från [föregående avsnitt](#deploy-the-application).

I det här läget:
- Kubernetes-infrastrukturen har skapats, inklusive en ingångs kontroll.
- Kluster har distribuerats över två Azure Stack Hubbs instanser.
- Övervakning har kon figurer ATS.
- Azure Traffic Manager belastningsutjämna trafik mellan de två Azure Stack Hub-instanserna.
- På den här infrastrukturen har exemplet på tre skikts program distribuerats på ett automatiserat sätt med hjälp av Helm-diagram. 

Lösningen bör nu vara tillgänglig och tillgänglig för användarna!

Det finns också några operativa överväganden efter distribution som beskrivs i följande två avsnitt.

## <a name="upgrade-kubernetes"></a>Uppgradera Kubernetes

Överväg följande avsnitt när du uppgraderar Kubernetes-klustret:

- Att uppgradera ett Kubernetes-kluster är en komplex dag 2-åtgärd som kan utföras med AKS-motorn. Mer information finns i [uppgradera ett Kubernetes-kluster på Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- Med AKS-motorn kan du uppgradera kluster till nyare Kubernetes och bas operativ system avbildnings versioner. Mer information finns i [steg för att uppgradera till en nyare Kubernetes-version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- Du kan också uppgradera de underplacerade noderna till nyare bas operativ system avbildnings versioner. Mer information finns i [steg för att endast uppgradera OS-avbildningen](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Nyare grundläggande OS-avbildningar innehåller säkerhets-och kernel-uppdateringar. Det är kluster operatörens ansvar att övervaka tillgängligheten för nyare Kubernetes-versioner och OS-avbildningar. Operatören bör planera och köra dessa uppgraderingar med AKS-motorn. De grundläggande OS-avbildningarna måste laddas ned från Azure Stack Hub Marketplace av Azure Stack Hub-operatorn.

## <a name="scale-kubernetes"></a>Skala Kubernetes

Skala är en annan dag 2-åtgärd som kan dirigeras med hjälp av AKS-motorn.

Kommandot Scale återanvänder kluster konfigurations filen (apimodel.jspå) i utdatakatalogen som indata för en ny Azure Resource Manager distribution. AKS-motorn kör skalnings åtgärden mot en angiven agent. När skalnings åtgärden är klar uppdaterar AKS-motorn kluster definitionen i samma apimodel.jsi filen. Kluster definitionen visar antalet nya noder för att återspegla den uppdaterade, aktuella kluster konfigurationen.

- [Skala ett Kubernetes-kluster på Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Nästa steg

- Läs mer om [design överväganden för Hybrid appar](overview-app-design-considerations.md)
- Granska och föreslå förbättringar av [koden för det här exemplet på GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).