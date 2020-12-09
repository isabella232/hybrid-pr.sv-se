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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="6b6f2-103">Distribuera ett Kubernetes-kluster med hög tillgänglighet på Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="6b6f2-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="6b6f2-104">Den här artikeln visar hur du skapar en Kubernetes kluster miljö med hög tillgänglighet, som distribueras på flera Azure Stack Hubbs instanser, på olika fysiska platser.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="6b6f2-105">I den här distributions hand boken för lösningen lär du dig att:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6b6f2-106">Ladda ned och Förbered AKS-motorn</span><span class="sxs-lookup"><span data-stu-id="6b6f2-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="6b6f2-107">Ansluta till den virtuella datorn med AKS motor hjälp</span><span class="sxs-lookup"><span data-stu-id="6b6f2-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="6b6f2-108">Distribuera ett Kubernetes-kluster</span><span class="sxs-lookup"><span data-stu-id="6b6f2-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="6b6f2-109">Ansluta till Kubernetes-klustret</span><span class="sxs-lookup"><span data-stu-id="6b6f2-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="6b6f2-110">Anslut Azure-pipeliner till Kubernetes-kluster</span><span class="sxs-lookup"><span data-stu-id="6b6f2-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="6b6f2-111">Konfigurera övervakning</span><span class="sxs-lookup"><span data-stu-id="6b6f2-111">Configure monitoring</span></span>
> - <span data-ttu-id="6b6f2-112">Distribuera programmet</span><span class="sxs-lookup"><span data-stu-id="6b6f2-112">Deploy application</span></span>
> - <span data-ttu-id="6b6f2-113">Autoskala program</span><span class="sxs-lookup"><span data-stu-id="6b6f2-113">Autoscale application</span></span>
> - <span data-ttu-id="6b6f2-114">Konfigurera Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="6b6f2-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="6b6f2-115">Uppgradera Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6b6f2-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="6b6f2-116">Skala Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6b6f2-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="6b6f2-117">![Hybrid pelare](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="6b6f2-118">Microsoft Azure Stack Hub är ett tillägg till Azure.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="6b6f2-119">Azure Stack Hub ger flexibilitet och innovation av molnbaserad data behandling till din lokala miljö, vilket möjliggör det enda hybrid molnet som gör det möjligt att bygga och distribuera hybrid program var som helst.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="6b6f2-120">Artikeln [hybrid app design överväganden](overview-app-design-considerations.md) granskar pelare för program kvalitet (placering, skalbarhet, tillgänglighet, återhämtning, hanterbarhet och säkerhet) för att utforma, distribuera och driva hybrid program.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="6b6f2-121">Design överväganden hjälper till att optimera hybrid utformning och minimera utmaningar i produktions miljöer.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6b6f2-122">Krav</span><span class="sxs-lookup"><span data-stu-id="6b6f2-122">Prerequisites</span></span>

<span data-ttu-id="6b6f2-123">Innan du börjar med den här distributions guiden kontrollerar du att:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="6b6f2-124">Läs artikeln om [Kubernetes kluster mönster för hög tillgänglighet](pattern-highly-available-kubernetes.md) .</span><span class="sxs-lookup"><span data-stu-id="6b6f2-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="6b6f2-125">Granska innehållet i [Companion GitHub-lagringsplatsen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), som innehåller ytterligare till gångar som refereras i den här artikeln.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="6b6f2-126">Ha ett konto som har åtkomst till [användar portalen för Azure Stack Hub](/azure-stack/user/azure-stack-use-portal), med minst [deltagar behörighet](/azure-stack/user/azure-stack-manage-permissions).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="6b6f2-127">Ladda ned och Förbered AKS-motorn</span><span class="sxs-lookup"><span data-stu-id="6b6f2-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="6b6f2-128">AKS-motorn är en binärfil som kan användas från valfri Windows-eller Linux-värd som kan komma åt Azure Stack Hub Azure Resource Manager-slutpunkter.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="6b6f2-129">Den här guiden beskriver hur du distribuerar en ny Linux-(eller Windows) virtuell dator på Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="6b6f2-130">Den kommer att användas senare när AKS-motorn distribuerar Kubernetes-klustren.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="6b6f2-131">Du kan också använda en befintlig virtuell Windows-eller Linux-dator för att distribuera ett Kubernetes-kluster på Azure Stack hubb med AKS-motorn.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="6b6f2-132">Steg för steg-processen och kraven för AKS-motorn beskrivs här:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="6b6f2-133">[Installera AKS-motorn på Linux i Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (eller med [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="6b6f2-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="6b6f2-134">AKS-motorn är ett hjälp verktyg för att distribuera och använda (ohanterade) Kubernetes kluster (i Azure och Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="6b6f2-135">Information och skillnader i AKS-motorn på Azure Stack Hub beskrivs här:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="6b6f2-136">Vad är AKS-motorn på Azure Stack Hub?</span><span class="sxs-lookup"><span data-stu-id="6b6f2-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="6b6f2-137">[AKS-motor på Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (på GitHub)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="6b6f2-138">Exempel miljön kommer att använda terraform för att automatisera distributionen av den virtuella AKS-motorn.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="6b6f2-139">Du hittar [information och kod i Companion GitHub-lagrings platsen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="6b6f2-140">Resultatet av det här steget är en ny resurs grupp på Azure Stack hubb som innehåller den virtuella datorn med AKS-motorns hjälp och relaterade resurser:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![AKS-motorns VM-resurser i Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="6b6f2-142">Om du måste distribuera AKS-motorn i en frånkopplad Air-gapped-miljö granskar du [frånkopplade Azure Stack Hubbs instanser](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) för mer information.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="6b6f2-143">I nästa steg ska vi använda den nyligen distribuerade virtuella AKS-motorn för att distribuera ett Kubernetes-kluster.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="6b6f2-144">Ansluta till den virtuella datorn med AKS motor hjälp</span><span class="sxs-lookup"><span data-stu-id="6b6f2-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="6b6f2-145">Först måste du ansluta till den tidigare skapade virtuella datorn för AKS-motorn.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="6b6f2-146">Den virtuella datorn ska ha en offentlig IP-adress och bör vara tillgänglig via SSH (port 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Översikts sida för virtuell AKS-motor](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="6b6f2-148">Du kan använda ett verktyg som du väljer som MobaXterm, SparaTillFil eller PowerShell i Windows 10 för att ansluta till en virtuell Linux-dator med SSH.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="6b6f2-149">När du har anslutit kör du kommandot `aks-engine` .</span><span class="sxs-lookup"><span data-stu-id="6b6f2-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="6b6f2-150">Gå till [AKS-motor versioner som stöds](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) och lär dig mer om AKS-motorn och Kubernetes-versioner.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![AKS-kommando rads exempel](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="6b6f2-152">Distribuera ett Kubernetes-kluster</span><span class="sxs-lookup"><span data-stu-id="6b6f2-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="6b6f2-153">AKS-motorns hjälp verktyg för virtuella datorer har inte skapat något Kubernetes-kluster på vår Azure Stack hubb, men.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="6b6f2-154">Att skapa klustret är den första åtgärden som ska vidtas i AKS-motorns hjälp programs virtuella dator.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="6b6f2-155">Steg för steg-processen beskrivs här:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="6b6f2-156">Distribuera ett Kubernetes-kluster med AKS-motorn på Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="6b6f2-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="6b6f2-157">Slut resultatet för `aks-engine deploy` kommandot och förberedelserna i föregående steg är ett fullständigt aktuellt Kubernetes-kluster som distribueras till klient området för den första Azure Stack Hub-instansen.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="6b6f2-158">Själva klustret består av Azure IaaS-komponenter, t. ex. virtuella datorer, belastningsutjämnare, virtuella nätverk, diskar och så vidare.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Kluster IaaS-komponenter Azure Stack hubb Portal](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="6b6f2-160">Azure Load Balancer (K8s API-slutpunkt)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="6b6f2-161">Arbetsnoder (agent-pool)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="6b6f2-162">Huvudnoder</span><span class="sxs-lookup"><span data-stu-id="6b6f2-162">Master Nodes</span></span>

<span data-ttu-id="6b6f2-163">Klustret är nu igång och i nästa steg ska vi ansluta till den.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="6b6f2-164">Ansluta till Kubernetes-klustret</span><span class="sxs-lookup"><span data-stu-id="6b6f2-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="6b6f2-165">Nu kan du ansluta till det tidigare skapade Kubernetes-klustret, antingen via SSH (med SSH-nyckeln som anges som en del av distributionen) eller via `kubectl` (rekommenderas).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="6b6f2-166">Kommando rads verktyget Kubernetes `kubectl` är tillgängligt för Windows, Linux och MacOS [här](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="6b6f2-167">Den är redan förinstallerad och konfigurerad på huvudnoderna i klustret.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Kör kubectl på huvud nod](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="6b6f2-169">Vi rekommenderar inte att du använder huvudnoden som ett hopp för administrativa uppgifter.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="6b6f2-170">`kubectl`Konfigurationen lagras `.kube/config` på huvud noderna och på den virtuella AKS-motorn.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="6b6f2-171">Du kan kopiera konfigurationen till en administratörs dator med anslutning till Kubernetes-klustret och använda `kubectl` kommandot där.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="6b6f2-172">`.kube/config`Filen används också senare för att konfigurera en tjänst anslutning i Azure-pipelines.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6b6f2-173">Se till att dessa filer är säkra eftersom de innehåller autentiseringsuppgifterna för ditt Kubernetes-kluster.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="6b6f2-174">En angripare med åtkomst till filen har tillräckligt med information för att få administratörs åtkomst till den.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="6b6f2-175">Alla åtgärder som görs med den inledande `.kube/config` filen görs med ett kluster administratörs konto.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="6b6f2-176">Nu kan du prova olika kommandon med `kubectl` för att kontrol lera status för klustret.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

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
> <span data-ttu-id="6b6f2-177">Kubernetes har en egen _ \*rollbaserad Access Control (RBAC) *-* modell som gör att du kan skapa detaljerade roll definitioner och roll bindningar.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="6b6f2-178">Detta är det bättre sättet att kontrol lera åtkomsten till klustret i stället för att bli av klustrets administratörs behörighet.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="6b6f2-179">Ansluta Azure-pipeliner till Kubernetes-kluster</span><span class="sxs-lookup"><span data-stu-id="6b6f2-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="6b6f2-180">För att ansluta Azure-pipeliner till det nydistribuerade Kubernetes-klustret behöver vi dess Kube config ( `.kube/config` )-fil enligt beskrivningen i föregående steg.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="6b6f2-181">Anslut till en av huvudnoderna i ditt Kubernetes-kluster.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="6b6f2-182">Kopiera innehållet i `.kube/config` filen.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="6b6f2-183">Gå till Azure DevOps > projekt inställningar > tjänst anslutningar för att skapa en ny "Kubernetes"-tjänst anslutning (Använd KubeConfig som autentiseringsmetod)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6b6f2-184">Azure-pipeliner (eller dess build-agenter) måste ha åtkomst till Kubernetes-API: et.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="6b6f2-185">Om det finns en Internet anslutning från Azure pipelines till Azure Stack Hub Kubernetes clusetr, måste du distribuera en lokal version av Azure pipelines för Azure.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="6b6f2-186">När du distribuerar egna värdbaserade agenter för Azure-pipeliner kan du distribuera antingen på Azure Stack hubb eller på en dator med nätverks anslutning till alla nödvändiga hanterings slut punkter.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="6b6f2-187">Se informationen här:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-187">See the details here:</span></span>

* <span data-ttu-id="6b6f2-188">[Azure pipeline-agenter](/azure/devops/pipelines/agents/agents) i [Windows](/azure/devops/pipelines/agents/v2-windows) eller [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="6b6f2-189">Avsnittet om [överväganden för mönster distribution (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) innehåller ett besluts flöde som hjälper dig att förstå huruvida du ska använda Microsoft-värdbaserade agenter eller egna värdbaserade agenter:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="6b6f2-190">[![besluts flöde, egna värdbaserade agenter](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="6b6f2-191">I den här exempel lösningen innehåller topologin en egen värd versions agent för varje Azure Stack Hub-instans.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="6b6f2-192">Agenten har åtkomst till Azure Stack Hub Management-slutpunkter och Kubernetes för kluster-API: er.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="6b6f2-193">[![endast utgående trafik](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="6b6f2-194">Den här designen uppfyller ett gemensamt föreskrivande krav, som endast ska ha utgående anslutningar från program lösningen.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="6b6f2-195">Konfigurera övervakning</span><span class="sxs-lookup"><span data-stu-id="6b6f2-195">Configure monitoring</span></span>

<span data-ttu-id="6b6f2-196">Du kan använda [Azure Monitor](/azure/azure-monitor/) för behållare för att övervaka behållarna i lösningen.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="6b6f2-197">Detta leder Azure Monitor till det AKS-distribuerade Kubernetes-klustret på Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="6b6f2-198">Det finns två sätt att aktivera Azure Monitor i klustret.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="6b6f2-199">Båda sätten kräver att du konfigurerar en Log Analytics arbets yta i Azure.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="6b6f2-200">[Metod ett](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) använder ett Helm-diagram</span><span class="sxs-lookup"><span data-stu-id="6b6f2-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="6b6f2-201">[Metod två](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) som en del av AKS-motorns kluster specifikation</span><span class="sxs-lookup"><span data-stu-id="6b6f2-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="6b6f2-202">I prov sto pol Ogin används "metod ett", vilket gör att processen och uppdateringar kan installeras enklare.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="6b6f2-203">I nästa steg behöver du en Azure LogAnalytics-arbetsyta (ID och nyckel), `Helm` (version 3) och `kubectl` på din dator.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="6b6f2-204">Helm är en Kubernetes-paket hanterare som är tillgänglig som en binärfil som körs på macOS, Windows och Linux.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="6b6f2-205">Den kan hämtas här: [Helm.sh](https://helm.sh/docs/intro/quickstart/) Helm förlitar sig på Kubernetes-konfigurationsfilen som används för `kubectl` kommandot.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="6b6f2-206">Med det här kommandot installeras Azure Monitor-agenten på ditt Kubernetes-kluster:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="6b6f2-207">Hanterings agenten för Operations Management Suite (OMS) på ditt Kubernetes-kluster kommer att skicka övervaknings data till din Azure Log Analytics-arbetsyta (med utgående HTTPS).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="6b6f2-208">Nu kan du använda Azure Monitor för att få djupare insikter om dina Kubernetes-kluster på Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="6b6f2-209">Den här designen är ett kraftfullt sätt att demonstrera kraften i analyser som kan distribueras automatiskt med ditt programs kluster.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="6b6f2-210">[![Azure Stack hubb kluster i Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="6b6f2-211">[![Azure Monitor kluster information](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6b6f2-212">Om Azure Monitor inte visar några Azure Stack Hub-data kontrollerar du att du har följt anvisningarna för [hur du lägger till AzureMonitor-Containers lösning i en Azure Loganalytics-arbetsyta](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) noggrant.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="6b6f2-213">Distribuera programmet</span><span class="sxs-lookup"><span data-stu-id="6b6f2-213">Deploy the application</span></span>

<span data-ttu-id="6b6f2-214">Innan du installerar vårt exempel program finns det ett annat steg att konfigurera nginx-based ingångs styrenheten på vårt Kubernetes-kluster.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="6b6f2-215">Ingångs styrenheten används som en Layer 7-belastningsutjämnare för att dirigera trafik i vårt kluster baserat på värd, sökväg eller protokoll.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="6b6f2-216">Nginx – ingress är tillgängligt som ett Helm-diagram.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="6b6f2-217">Detaljerade anvisningar finns i [Helm-diagrammets GitHub-lagringsplats](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="6b6f2-218">Vårt exempel program paketeras också som ett Helm-diagram, som [Azure Monitoring Agent](#configure-monitoring) i föregående steg.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="6b6f2-219">Därför är det enkelt att distribuera programmet till vårt Kubernetes-kluster.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="6b6f2-220">Du kan hitta [Helm i Companion GitHub-lagrings platsen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="6b6f2-221">Exempel programmet är ett program på tre nivåer som distribueras till ett Kubernetes-kluster på var och en av två Azure Stack Hub-instanser.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="6b6f2-222">Programmet använder en MongoDB-databas.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="6b6f2-223">Du kan lära dig mer om hur du hämtar data som replikeras över flera instanser i mönster [data och lagrings överväganden](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="6b6f2-224">När du har distribuerat Helm-diagrammet för programmet visas alla tre nivåer av ditt program som visas som distributioner och tillstånds känsliga uppsättningar (för databasen) med en enda pod:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

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

<span data-ttu-id="6b6f2-225">På sidan tjänster hittar du den nginx-baserade ingångs styrenheten och dess offentliga IP-adress:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

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

<span data-ttu-id="6b6f2-226">Adressen "extern IP" är vår "program slut punkt".</span><span class="sxs-lookup"><span data-stu-id="6b6f2-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="6b6f2-227">Det är hur användarna kommer att ansluta till att öppna programmet och kommer även att användas som slut punkt för nästa steg [konfigurera Traffic Manager](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="6b6f2-228">Skala programmet Autoskala</span><span class="sxs-lookup"><span data-stu-id="6b6f2-228">Autoscale the application</span></span>
<span data-ttu-id="6b6f2-229">Du kan också konfigurera den [vågräta Pod autoskalning](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) för att skala upp eller ned baserat på vissa mått som processor belastning.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="6b6f2-230">Följande kommando skapar en vågrät Pod-autoskalning som hanterar 1 till 10 repliker av poddar som kontrol leras av klassificeringen-webb distribution.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="6b6f2-231">HPA kommer att öka och minska antalet repliker (via distributionen) för att bibehålla en genomsnittlig CPU-belastning för alla poddar på 80%.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="6b6f2-232">Du kan kontrol lera den aktuella statusen för autoskalning genom att köra:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="6b6f2-233">Konfigurera Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="6b6f2-233">Configure Traffic Manager</span></span>

<span data-ttu-id="6b6f2-234">För att distribuera trafik mellan två (eller flera) distributioner av programmet använder vi [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="6b6f2-235">Azure Traffic Manager är en DNS-baserad trafikbelastnings utjämning i Azure.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="6b6f2-236">Traffic Manager använder DNS för att dirigera klient begär anden till den lämpligaste tjänst slut punkten, baserat på en Traffic-routningsmetod och tillståndet för slut punkterna.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="6b6f2-237">I stället för att använda Azure Traffic Manager kan du också använda andra globala lösningar för belastnings utjämning som finns lokalt.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="6b6f2-238">I exempel scenariot använder vi Azure Traffic Manager för att distribuera trafik mellan två instanser av vårt program.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="6b6f2-239">De kan köras på Azure Stack Hub-instanser på samma eller olika platser:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![lokal Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="6b6f2-241">I Azure konfigurerar vi Traffic Manager så att de pekar på de två olika instanserna av programmet:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="6b6f2-242">[![TM-slutpunkt-profil](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="6b6f2-243">Som du kan se pekar de två slut punkterna på de två instanserna av det distribuerade programmet från [föregående avsnitt](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="6b6f2-244">I det här läget:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-244">At this point:</span></span>
- <span data-ttu-id="6b6f2-245">Kubernetes-infrastrukturen har skapats, inklusive en ingångs kontroll.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="6b6f2-246">Kluster har distribuerats över två Azure Stack Hubbs instanser.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="6b6f2-247">Övervakning har kon figurer ATS.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="6b6f2-248">Azure Traffic Manager belastningsutjämna trafik mellan de två Azure Stack Hub-instanserna.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="6b6f2-249">På den här infrastrukturen har exemplet på tre skikts program distribuerats på ett automatiserat sätt med hjälp av Helm-diagram.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="6b6f2-250">Lösningen bör nu vara tillgänglig och tillgänglig för användarna!</span><span class="sxs-lookup"><span data-stu-id="6b6f2-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="6b6f2-251">Det finns också några operativa överväganden efter distribution som beskrivs i följande två avsnitt.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="6b6f2-252">Uppgradera Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6b6f2-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="6b6f2-253">Överväg följande avsnitt när du uppgraderar Kubernetes-klustret:</span><span class="sxs-lookup"><span data-stu-id="6b6f2-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="6b6f2-254">Att uppgradera ett Kubernetes-kluster är en komplex dag 2-åtgärd som kan utföras med AKS-motorn.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="6b6f2-255">Mer information finns i [uppgradera ett Kubernetes-kluster på Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="6b6f2-256">Med AKS-motorn kan du uppgradera kluster till nyare Kubernetes och bas operativ system avbildnings versioner.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="6b6f2-257">Mer information finns i [steg för att uppgradera till en nyare Kubernetes-version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="6b6f2-258">Du kan också uppgradera de underplacerade noderna till nyare bas operativ system avbildnings versioner.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="6b6f2-259">Mer information finns i [steg för att endast uppgradera OS-avbildningen](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="6b6f2-260">Nyare grundläggande OS-avbildningar innehåller säkerhets-och kernel-uppdateringar.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="6b6f2-261">Det är kluster operatörens ansvar att övervaka tillgängligheten för nyare Kubernetes-versioner och OS-avbildningar.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="6b6f2-262">Operatören bör planera och köra dessa uppgraderingar med AKS-motorn.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="6b6f2-263">De grundläggande OS-avbildningarna måste laddas ned från Azure Stack Hub Marketplace av Azure Stack Hub-operatorn.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="6b6f2-264">Skala Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6b6f2-264">Scale Kubernetes</span></span>

<span data-ttu-id="6b6f2-265">Skala är en annan dag 2-åtgärd som kan dirigeras med hjälp av AKS-motorn.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="6b6f2-266">Kommandot Scale återanvänder kluster konfigurations filen (apimodel.jspå) i utdatakatalogen som indata för en ny Azure Resource Manager distribution.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="6b6f2-267">AKS-motorn kör skalnings åtgärden mot en angiven agent.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="6b6f2-268">När skalnings åtgärden är klar uppdaterar AKS-motorn kluster definitionen i samma apimodel.jsi filen.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="6b6f2-269">Kluster definitionen visar antalet nya noder för att återspegla den uppdaterade, aktuella kluster konfigurationen.</span><span class="sxs-lookup"><span data-stu-id="6b6f2-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="6b6f2-270">Skala ett Kubernetes-kluster på Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="6b6f2-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="6b6f2-271">Nästa steg</span><span class="sxs-lookup"><span data-stu-id="6b6f2-271">Next steps</span></span>

- <span data-ttu-id="6b6f2-272">Läs mer om [design överväganden för Hybrid appar](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="6b6f2-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="6b6f2-273">Granska och föreslå förbättringar av [koden för det här exemplet på GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="6b6f2-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>