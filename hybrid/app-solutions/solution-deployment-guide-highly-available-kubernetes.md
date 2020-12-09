---
title: Maxi maal beschik bare Kubernetes-cluster implementeren op Azure Stack hub
description: Meer informatie over het implementeren van een Kubernetes-cluster oplossing voor hoge Beschik baarheid met behulp van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911735"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="6ad17-103">Een Kubernetes-cluster met hoge Beschik baarheid implementeren op Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="6ad17-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="6ad17-104">In dit artikel wordt uitgelegd hoe u een Maxi maal beschik bare Kubernetes-cluster omgeving kunt bouwen, die op meerdere Azure Stack hub-instanties wordt geïmplementeerd op verschillende fysieke locaties.</span><span class="sxs-lookup"><span data-stu-id="6ad17-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="6ad17-105">In deze hand leiding voor implementaties van oplossingen leert u het volgende:</span><span class="sxs-lookup"><span data-stu-id="6ad17-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6ad17-106">De AKS-engine downloaden en voorbereiden</span><span class="sxs-lookup"><span data-stu-id="6ad17-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="6ad17-107">Verbinding maken met de AKS engine helper-VM</span><span class="sxs-lookup"><span data-stu-id="6ad17-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="6ad17-108">Een Kubernetes-cluster implementeren</span><span class="sxs-lookup"><span data-stu-id="6ad17-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="6ad17-109">Verbinding maken met het Kubernetes-cluster</span><span class="sxs-lookup"><span data-stu-id="6ad17-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="6ad17-110">Azure-pijp lijnen verbinden met het Kubernetes-cluster</span><span class="sxs-lookup"><span data-stu-id="6ad17-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="6ad17-111">Bewaking configureren</span><span class="sxs-lookup"><span data-stu-id="6ad17-111">Configure monitoring</span></span>
> - <span data-ttu-id="6ad17-112">Toepassing implementeren</span><span class="sxs-lookup"><span data-stu-id="6ad17-112">Deploy application</span></span>
> - <span data-ttu-id="6ad17-113">Toepassing automatisch schalen</span><span class="sxs-lookup"><span data-stu-id="6ad17-113">Autoscale application</span></span>
> - <span data-ttu-id="6ad17-114">Traffic Manager configureren</span><span class="sxs-lookup"><span data-stu-id="6ad17-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="6ad17-115">Kubernetes bijwerken</span><span class="sxs-lookup"><span data-stu-id="6ad17-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="6ad17-116">Kubernetes schalen</span><span class="sxs-lookup"><span data-stu-id="6ad17-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="6ad17-117">![Hybride pijlers](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="6ad17-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="6ad17-118">Microsoft Azure Stack hub is een uitbrei ding van Azure.</span><span class="sxs-lookup"><span data-stu-id="6ad17-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="6ad17-119">Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.</span><span class="sxs-lookup"><span data-stu-id="6ad17-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="6ad17-120">In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps.</span><span class="sxs-lookup"><span data-stu-id="6ad17-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="6ad17-121">De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.</span><span class="sxs-lookup"><span data-stu-id="6ad17-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6ad17-122">Vereisten</span><span class="sxs-lookup"><span data-stu-id="6ad17-122">Prerequisites</span></span>

<span data-ttu-id="6ad17-123">Voordat u aan de slag gaat met deze implementatie handleiding, moet u het volgende doen:</span><span class="sxs-lookup"><span data-stu-id="6ad17-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="6ad17-124">Bekijk het artikel over het [Kubernetes cluster patroon met hoge Beschik baarheid](pattern-highly-available-kubernetes.md) .</span><span class="sxs-lookup"><span data-stu-id="6ad17-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="6ad17-125">Bekijk de inhoud van de [opslag plaats github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), die aanvullende assets bevat waarnaar in dit artikel wordt verwezen.</span><span class="sxs-lookup"><span data-stu-id="6ad17-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="6ad17-126">Beschikken over een account dat toegang heeft tot de [Azure stack hub-gebruikers Portal](/azure-stack/user/azure-stack-use-portal), met ten minste de [machtiging Inzender](/azure-stack/user/azure-stack-manage-permissions).</span><span class="sxs-lookup"><span data-stu-id="6ad17-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="6ad17-127">De AKS-engine downloaden en voorbereiden</span><span class="sxs-lookup"><span data-stu-id="6ad17-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="6ad17-128">De AKS-engine is een binair bestand dat kan worden gebruikt vanaf elke Windows-of Linux-host die de Azure Stack hub Azure Resource Manager eind punten kan bereiken.</span><span class="sxs-lookup"><span data-stu-id="6ad17-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="6ad17-129">In deze hand leiding wordt beschreven hoe u een nieuwe virtuele Linux-machine (of Windows) implementeert op Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6ad17-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="6ad17-130">Deze wordt later gebruikt wanneer de AKS-engine de Kubernetes-clusters implementeert.</span><span class="sxs-lookup"><span data-stu-id="6ad17-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="6ad17-131">U kunt ook een bestaande virtuele machine met Windows of Linux gebruiken om een Kubernetes-cluster op Azure Stack hub te implementeren met behulp van de AKS-engine.</span><span class="sxs-lookup"><span data-stu-id="6ad17-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="6ad17-132">Het stapsgewijze proces en de vereisten voor de AKS-engine worden hier beschreven:</span><span class="sxs-lookup"><span data-stu-id="6ad17-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="6ad17-133">[De AKS-Engine installeren op Linux in azure stack hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (of met behulp van [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="6ad17-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="6ad17-134">De AKS-engine is een hulp programma voor het implementeren en uitvoeren van (niet-beheerde) Kubernetes-clusters (in Azure en Azure Stack hub).</span><span class="sxs-lookup"><span data-stu-id="6ad17-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="6ad17-135">De details en verschillen van de AKS-engine op Azure Stack hub worden hier beschreven:</span><span class="sxs-lookup"><span data-stu-id="6ad17-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="6ad17-136">Wat is de AKS-engine op Azure Stack hub?</span><span class="sxs-lookup"><span data-stu-id="6ad17-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="6ad17-137">[AKS-engine op Azure stack hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (op github)</span><span class="sxs-lookup"><span data-stu-id="6ad17-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="6ad17-138">In de voorbeeld omgeving wordt terraform gebruikt voor het automatiseren van de implementatie van de AKS-engine-VM.</span><span class="sxs-lookup"><span data-stu-id="6ad17-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="6ad17-139">U kunt de [Details en code vinden in de GitHub opslag plaats](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="6ad17-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="6ad17-140">Het resultaat van deze stap is een nieuwe resource groep op Azure Stack hub die de AKS engine helper-VM en gerelateerde resources bevat:</span><span class="sxs-lookup"><span data-stu-id="6ad17-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![VM-resources van AKS-engine in Azure Stack hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="6ad17-142">Als u de AKS-engine in een niet-verbonden lucht-gapped-omgeving moet implementeren, controleert u de [losgekoppelde Azure stack hub-instanties](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="6ad17-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="6ad17-143">In de volgende stap gebruiken we de zojuist geïmplementeerde VM van de AKS-Engine om een Kubernetes-cluster te implementeren.</span><span class="sxs-lookup"><span data-stu-id="6ad17-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="6ad17-144">Verbinding maken met de AKS engine helper-VM</span><span class="sxs-lookup"><span data-stu-id="6ad17-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="6ad17-145">Eerst moet u verbinding maken met de eerder gemaakte AKS engine helper-VM.</span><span class="sxs-lookup"><span data-stu-id="6ad17-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="6ad17-146">De virtuele machine moet een openbaar IP-adres hebben en moet toegankelijk zijn via SSH (poort 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="6ad17-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Overzichts pagina voor de VM van AKS-engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="6ad17-148">U kunt een hulp programma gebruiken van uw keuze als MobaXterm, puTTy of Power shell in Windows 10 om verbinding te maken met een virtuele Linux-machine met behulp van SSH.</span><span class="sxs-lookup"><span data-stu-id="6ad17-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="6ad17-149">Nadat u verbinding hebt gemaakt, voert u de opdracht uit `aks-engine` .</span><span class="sxs-lookup"><span data-stu-id="6ad17-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="6ad17-150">Ga naar [ondersteunde AKS-Engine versies](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) voor meer informatie over de AKS-engine en Kubernetes-versies.</span><span class="sxs-lookup"><span data-stu-id="6ad17-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![AKS-engine-opdracht regel voorbeeld](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="6ad17-152">Een Kubernetes-cluster implementeren</span><span class="sxs-lookup"><span data-stu-id="6ad17-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="6ad17-153">De AKS engine helper-VM zelf heeft nog geen Kubernetes-cluster gemaakt op onze Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6ad17-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="6ad17-154">Het maken van het cluster is de eerste actie die moet worden uitgevoerd in de AKS engine helper-VM.</span><span class="sxs-lookup"><span data-stu-id="6ad17-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="6ad17-155">Het stapsgewijze proces wordt hier beschreven:</span><span class="sxs-lookup"><span data-stu-id="6ad17-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="6ad17-156">Een Kubernetes-cluster implementeren met de AKS-engine op Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="6ad17-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="6ad17-157">Het eind resultaat van de `aks-engine deploy` opdracht en de voor bereidingen in de vorige stappen is een volledig aanbevolen Kubernetes-cluster dat is geïmplementeerd in de Tenant ruimte van het eerste Azure stack hub-exemplaar.</span><span class="sxs-lookup"><span data-stu-id="6ad17-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="6ad17-158">Het cluster zelf bestaat uit Azure IaaS-onderdelen, zoals Vm's, load balancers, VNets, disks, enzovoort.</span><span class="sxs-lookup"><span data-stu-id="6ad17-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![IaaS-onderdelen van cluster Azure Stack hub-Portal](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="6ad17-160">Azure load balancer (K8s API-eind punt)</span><span class="sxs-lookup"><span data-stu-id="6ad17-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="6ad17-161">Worker-knoop punten (agent groep)</span><span class="sxs-lookup"><span data-stu-id="6ad17-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="6ad17-162">Hoofd knooppunten</span><span class="sxs-lookup"><span data-stu-id="6ad17-162">Master Nodes</span></span>

<span data-ttu-id="6ad17-163">Het cluster is nu actief en in de volgende stap wordt er verbinding mee gemaakt.</span><span class="sxs-lookup"><span data-stu-id="6ad17-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="6ad17-164">Verbinding maken met het Kubernetes-cluster</span><span class="sxs-lookup"><span data-stu-id="6ad17-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="6ad17-165">U kunt nu verbinding maken met het eerder gemaakte Kubernetes-cluster via SSH (met behulp van de SSH-sleutel die is opgegeven als onderdeel van de implementatie) of via `kubectl` (aanbevolen).</span><span class="sxs-lookup"><span data-stu-id="6ad17-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="6ad17-166">Het opdracht regel programma Kubernetes `kubectl` is [hier](https://kubernetes.io/docs/tasks/tools/install-kubectl/)beschikbaar voor Windows, Linux en macOS.</span><span class="sxs-lookup"><span data-stu-id="6ad17-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="6ad17-167">Het is al vooraf geïnstalleerd en geconfigureerd op de hoofd knooppunten van het cluster.</span><span class="sxs-lookup"><span data-stu-id="6ad17-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Kubectl uitvoeren op hoofd knooppunt](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="6ad17-169">Het is niet raadzaam om het hoofd knooppunt te gebruiken als een JumpBox voor beheer taken.</span><span class="sxs-lookup"><span data-stu-id="6ad17-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="6ad17-170">De `kubectl` configuratie wordt opgeslagen in `.kube/config` op de hoofd knooppunten en op de virtuele machine met de AKS-engine.</span><span class="sxs-lookup"><span data-stu-id="6ad17-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="6ad17-171">U kunt de configuratie kopiëren naar een beheer computer met verbinding met het Kubernetes-cluster en de `kubectl` opdracht hier gebruiken.</span><span class="sxs-lookup"><span data-stu-id="6ad17-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="6ad17-172">Het `.kube/config` bestand wordt ook later gebruikt voor het configureren van een service verbinding in azure-pijp lijnen.</span><span class="sxs-lookup"><span data-stu-id="6ad17-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6ad17-173">Bewaar deze bestanden als veilig omdat ze de referenties voor uw Kubernetes-cluster bevatten.</span><span class="sxs-lookup"><span data-stu-id="6ad17-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="6ad17-174">Een aanvaller met toegang tot het bestand beschikt over voldoende informatie om beheerders toegang te krijgen.</span><span class="sxs-lookup"><span data-stu-id="6ad17-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="6ad17-175">Alle acties die met het eerste bestand worden uitgevoerd `.kube/config` , worden uitgevoerd met behulp van een cluster-beheerders account.</span><span class="sxs-lookup"><span data-stu-id="6ad17-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="6ad17-176">U kunt nu verschillende opdrachten gebruiken `kubectl` om de status van uw cluster te controleren.</span><span class="sxs-lookup"><span data-stu-id="6ad17-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

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
> <span data-ttu-id="6ad17-177">Kubernetes heeft een eigen _ *op rollen gebaseerd Access Control (RBAC)*\* model waarmee u verfijnde roldefinities en functie bindingen kunt maken.</span><span class="sxs-lookup"><span data-stu-id="6ad17-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="6ad17-178">Dit is de voorkeurs manier om de toegang tot het cluster te beheren in plaats van de machtigingen voor cluster beheer af te leveren.</span><span class="sxs-lookup"><span data-stu-id="6ad17-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="6ad17-179">Azure-pijp lijnen verbinden met Kubernetes-clusters</span><span class="sxs-lookup"><span data-stu-id="6ad17-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="6ad17-180">Als u Azure-pijp lijnen wilt verbinden met het zojuist geïmplementeerde Kubernetes-cluster, hebt u het bestand uitvoeren config ( `.kube/config` ) nodig zoals in de vorige stap wordt uitgelegd.</span><span class="sxs-lookup"><span data-stu-id="6ad17-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="6ad17-181">Maak verbinding met een van de hoofd knooppunten van uw Kubernetes-cluster.</span><span class="sxs-lookup"><span data-stu-id="6ad17-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="6ad17-182">Kopieer de inhoud van het `.kube/config` bestand.</span><span class="sxs-lookup"><span data-stu-id="6ad17-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="6ad17-183">Ga naar Azure DevOps > project instellingen > service verbindingen om een nieuwe ' Kubernetes-service verbinding te maken (gebruik KubeConfig als verificatie methode)</span><span class="sxs-lookup"><span data-stu-id="6ad17-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6ad17-184">Azure-pijp lijnen (of de bijbehorende bouw agenten) moeten toegang hebben tot de Kubernetes-API.</span><span class="sxs-lookup"><span data-stu-id="6ad17-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="6ad17-185">Als er een Internet verbinding van Azure-pijp lijnen naar de Azure Stack hub Kubernetes clusetr, moet u een zelf-hostende Azure pipelines build-agent implementeren.</span><span class="sxs-lookup"><span data-stu-id="6ad17-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="6ad17-186">Wanneer u zelf-hostende agents voor Azure-pijp lijnen implementeert, kunt u implementeren op Azure Stack hub of op een computer met een netwerk verbinding met alle vereiste beheer eindpunten.</span><span class="sxs-lookup"><span data-stu-id="6ad17-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="6ad17-187">Bekijk de details hier:</span><span class="sxs-lookup"><span data-stu-id="6ad17-187">See the details here:</span></span>

* <span data-ttu-id="6ad17-188">[Azure pipelines-agents](/azure/devops/pipelines/agents/agents) in [Windows](/azure/devops/pipelines/agents/v2-windows) of [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="6ad17-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="6ad17-189">De sectie [overwegingen bij het implementeren van patronen (CI/cd)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) bevat een beslissings stroom waarmee u kunt begrijpen of u door micro soft gehoste agents of zelf-hostende agents wilt gebruiken:</span><span class="sxs-lookup"><span data-stu-id="6ad17-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="6ad17-190">[![zelf gehoste agents in besluit stroom](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6ad17-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="6ad17-191">In deze voorbeeld oplossing bevat de topologie een zelf-hostende build-agent op elk Azure Stack hub-exemplaar.</span><span class="sxs-lookup"><span data-stu-id="6ad17-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="6ad17-192">De agent kan toegang krijgen tot de Azure Stack hub-beheer eindpunten en de Kubernetes-cluster-API-eind punten.</span><span class="sxs-lookup"><span data-stu-id="6ad17-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="6ad17-193">[![alleen uitgaand verkeer](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6ad17-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="6ad17-194">Dit ontwerp voldoet aan een gemeen schappelijke wettelijke vereiste, die alleen uitgaande verbindingen van de toepassings oplossing moet hebben.</span><span class="sxs-lookup"><span data-stu-id="6ad17-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="6ad17-195">Bewaking configureren</span><span class="sxs-lookup"><span data-stu-id="6ad17-195">Configure monitoring</span></span>

<span data-ttu-id="6ad17-196">U kunt [Azure monitor](/azure/azure-monitor/) voor containers gebruiken om de containers in de oplossing te bewaken.</span><span class="sxs-lookup"><span data-stu-id="6ad17-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="6ad17-197">Dit wijst Azure Monitor naar het door de AKS-engine geïmplementeerde Kubernetes-cluster op Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6ad17-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="6ad17-198">Er zijn twee manieren om Azure Monitor in te scha kelen in uw cluster.</span><span class="sxs-lookup"><span data-stu-id="6ad17-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="6ad17-199">Op beide manieren moet u een Log Analytics-werk ruimte instellen in Azure.</span><span class="sxs-lookup"><span data-stu-id="6ad17-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="6ad17-200">[Methode 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) maakt gebruik van een helm-grafiek</span><span class="sxs-lookup"><span data-stu-id="6ad17-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="6ad17-201">[Methode twee](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) als onderdeel van de cluster specificatie van de AKS-engine</span><span class="sxs-lookup"><span data-stu-id="6ad17-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="6ad17-202">In de voor beeld-topologie wordt ' methode één ' gebruikt, waardoor automatisering van het proces en updates eenvoudiger kunnen worden geïnstalleerd.</span><span class="sxs-lookup"><span data-stu-id="6ad17-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="6ad17-203">Voor de volgende stap hebt u een Azure LogAnalytics-werk ruimte (ID en sleutel), `Helm` (versie 3) en `kubectl` op uw computer nodig.</span><span class="sxs-lookup"><span data-stu-id="6ad17-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="6ad17-204">Helm is een Kubernetes-pakket beheer dat beschikbaar is als binaire waarde die wordt uitgevoerd op macOS, Windows en Linux.</span><span class="sxs-lookup"><span data-stu-id="6ad17-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="6ad17-205">Dit kan hier worden gedownload: [helm.sh](https://helm.sh/docs/intro/quickstart/) helm is afhankelijk van het Kubernetes-configuratie bestand dat voor de opdracht wordt gebruikt `kubectl` .</span><span class="sxs-lookup"><span data-stu-id="6ad17-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="6ad17-206">Met deze opdracht wordt de Azure Monitor-agent geïnstalleerd op uw Kubernetes-cluster:</span><span class="sxs-lookup"><span data-stu-id="6ad17-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="6ad17-207">Met de OMS-agent (Operations Management Suite) in uw Kubernetes-cluster worden bewakings gegevens naar uw Azure Log Analytics-werk ruimte verzonden (met uitgaande HTTPS).</span><span class="sxs-lookup"><span data-stu-id="6ad17-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="6ad17-208">U kunt Azure Monitor nu gebruiken om dieper inzicht te krijgen in uw Kubernetes-clusters op Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6ad17-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="6ad17-209">Dit ontwerp is een krachtige manier om de kracht van analyses te demonstreren die automatisch kan worden geïmplementeerd met de clusters van uw toepassing.</span><span class="sxs-lookup"><span data-stu-id="6ad17-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="6ad17-210">[![Azure Stack hub-clusters in azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6ad17-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="6ad17-211">[![Details van Azure Monitor cluster](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6ad17-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6ad17-212">Als Azure Monitor geen Azure Stack hub-gegevens weergeeft, moet u ervoor zorgen dat u zorgvuldig de instructies voor [het toevoegen van AzureMonitor-Containers oplossing aan een Azure Loganalytics-werk ruimte](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) hebt gevolgd.</span><span class="sxs-lookup"><span data-stu-id="6ad17-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="6ad17-213">De toepassing implementeren</span><span class="sxs-lookup"><span data-stu-id="6ad17-213">Deploy the application</span></span>

<span data-ttu-id="6ad17-214">Voordat u de voorbeeld toepassing installeert, is er nog een stap voor het configureren van de op nginx gebaseerde ingangs controller op het Kubernetes-cluster.</span><span class="sxs-lookup"><span data-stu-id="6ad17-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="6ad17-215">De ingangs controller wordt gebruikt als laag 7 load balancer het verkeer routeren in het cluster op basis van de host, het pad of het protocol.</span><span class="sxs-lookup"><span data-stu-id="6ad17-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="6ad17-216">Nginx-inkomend is beschikbaar als een helm-grafiek.</span><span class="sxs-lookup"><span data-stu-id="6ad17-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="6ad17-217">Raadpleeg de [helm Chart github-opslag plaats](https://github.com/helm/charts/tree/master/stable/nginx-ingress)voor gedetailleerde instructies.</span><span class="sxs-lookup"><span data-stu-id="6ad17-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="6ad17-218">Onze voorbeeld toepassing is ook verpakt als een helm-grafiek, zoals de [Azure Monitoring Agent](#configure-monitoring) in de vorige stap.</span><span class="sxs-lookup"><span data-stu-id="6ad17-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="6ad17-219">Zo is het eenvoudig om de toepassing te implementeren op het Kubernetes-cluster.</span><span class="sxs-lookup"><span data-stu-id="6ad17-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="6ad17-220">U kunt de [helm-grafiek bestanden vinden in de Companion github opslag plaats](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span><span class="sxs-lookup"><span data-stu-id="6ad17-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="6ad17-221">De voorbeeld toepassing is een toepassing met drie lagen, geïmplementeerd op een Kubernetes-cluster op elk van de twee Azure Stack hub-instanties.</span><span class="sxs-lookup"><span data-stu-id="6ad17-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="6ad17-222">De toepassing maakt gebruik van een MongoDB-data base.</span><span class="sxs-lookup"><span data-stu-id="6ad17-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="6ad17-223">Meer informatie over hoe u de gegevens kunt ophalen die worden gerepliceerd tussen meerdere exemplaren van de patroon [gegevens en opslag overwegingen](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span><span class="sxs-lookup"><span data-stu-id="6ad17-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="6ad17-224">Nadat u het helm-diagram voor de toepassing hebt geïmplementeerd, ziet u dat alle drie de lagen van uw toepassing worden weer gegeven als implementaties en stateful sets (voor de data base) met één Pod:</span><span class="sxs-lookup"><span data-stu-id="6ad17-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

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

<span data-ttu-id="6ad17-225">Aan de hand van de Services vindt u de nginx ingangs controller en het bijbehorende open bare IP-adres:</span><span class="sxs-lookup"><span data-stu-id="6ad17-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

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

<span data-ttu-id="6ad17-226">Het adres ' External IP ' is ons eind punt van de toepassing.</span><span class="sxs-lookup"><span data-stu-id="6ad17-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="6ad17-227">Het is hoe gebruikers verbinding maken om de toepassing te openen en worden ook gebruikt als het eind punt voor onze volgende stap [Traffic Manager configureren](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="6ad17-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="6ad17-228">De toepassing automatisch schalen</span><span class="sxs-lookup"><span data-stu-id="6ad17-228">Autoscale the application</span></span>
<span data-ttu-id="6ad17-229">U kunt eventueel de [horizontale pod autoscaleer](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) configureren om omhoog of omlaag te schalen op basis van bepaalde metrische gegevens, zoals CPU-gebruik.</span><span class="sxs-lookup"><span data-stu-id="6ad17-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="6ad17-230">Met de volgende opdracht wordt een horizontale pod voor automatisch schalen gemaakt waarmee 1 tot 10 replica's van de Peuling wordt beheerd door de classificatie-webimplementatie.</span><span class="sxs-lookup"><span data-stu-id="6ad17-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="6ad17-231">HPA verhoogt en verlaagt het aantal replica's (via de implementatie) om een gemiddeld CPU-gebruik voor alle aantallen van 80% te behouden.</span><span class="sxs-lookup"><span data-stu-id="6ad17-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="6ad17-232">U kunt de huidige status van automatisch schalen controleren door het volgende uit te voeren:</span><span class="sxs-lookup"><span data-stu-id="6ad17-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="6ad17-233">Traffic Manager configureren</span><span class="sxs-lookup"><span data-stu-id="6ad17-233">Configure Traffic Manager</span></span>

<span data-ttu-id="6ad17-234">Als u verkeer tussen twee (of meer) implementaties van de toepassing wilt distribueren, gebruiken we [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="6ad17-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="6ad17-235">Azure Traffic Manager is een op DNS gebaseerd verkeer load balancer in Azure.</span><span class="sxs-lookup"><span data-stu-id="6ad17-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="6ad17-236">Traffic Manager DNS gebruikt om client aanvragen door te sturen naar het meest geschikte service-eind punt, op basis van een routerings methode voor verkeer en de status van de eind punten.</span><span class="sxs-lookup"><span data-stu-id="6ad17-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="6ad17-237">In plaats van Azure Traffic Manager kunt u ook andere wereld wijde oplossingen voor taak verdeling gebruiken die lokaal worden gehost.</span><span class="sxs-lookup"><span data-stu-id="6ad17-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="6ad17-238">In het voorbeeld scenario gebruiken we Azure Traffic Manager voor het distribueren van verkeer tussen twee exemplaren van onze toepassing.</span><span class="sxs-lookup"><span data-stu-id="6ad17-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="6ad17-239">Ze kunnen op dezelfde of verschillende locaties op Azure Stack hub-instanties worden uitgevoerd:</span><span class="sxs-lookup"><span data-stu-id="6ad17-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![on-premises Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="6ad17-241">In azure configureren we Traffic Manager om te verwijzen naar de twee verschillende exemplaren van onze toepassing:</span><span class="sxs-lookup"><span data-stu-id="6ad17-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="6ad17-242">[![TM-eindpunt profiel](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6ad17-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="6ad17-243">Zoals u ziet, verwijzen de twee eind punten naar de twee exemplaren van de geïmplementeerde toepassing uit de [vorige sectie](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="6ad17-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="6ad17-244">Op dit punt:</span><span class="sxs-lookup"><span data-stu-id="6ad17-244">At this point:</span></span>
- <span data-ttu-id="6ad17-245">De Kubernetes-infra structuur is gemaakt, met inbegrip van een ingangs controller.</span><span class="sxs-lookup"><span data-stu-id="6ad17-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="6ad17-246">Clusters zijn geïmplementeerd in twee Azure Stack hub-exemplaren.</span><span class="sxs-lookup"><span data-stu-id="6ad17-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="6ad17-247">Bewaking is geconfigureerd.</span><span class="sxs-lookup"><span data-stu-id="6ad17-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="6ad17-248">Er wordt door Azure Traffic Manager taak verdeling van verkeer via de twee Azure Stack hub-exemplaren.</span><span class="sxs-lookup"><span data-stu-id="6ad17-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="6ad17-249">Op basis van deze infra structuur is de voor beeld-toepassing met drie lagen op een geautomatiseerde manier geïmplementeerd met behulp van helm-grafieken.</span><span class="sxs-lookup"><span data-stu-id="6ad17-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="6ad17-250">De oplossing moet nu toegankelijk zijn voor gebruikers.</span><span class="sxs-lookup"><span data-stu-id="6ad17-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="6ad17-251">Er zijn ook een aantal operationele aandachtspunten na de implementatie die in de volgende twee secties worden besproken.</span><span class="sxs-lookup"><span data-stu-id="6ad17-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="6ad17-252">Kubernetes bijwerken</span><span class="sxs-lookup"><span data-stu-id="6ad17-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="6ad17-253">Houd rekening met de volgende onderwerpen bij het upgraden van het Kubernetes-cluster:</span><span class="sxs-lookup"><span data-stu-id="6ad17-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="6ad17-254">Het bijwerken van een Kubernetes-cluster is een complexe bewerking van de dag 2 die kan worden uitgevoerd met behulp van de AKS-engine.</span><span class="sxs-lookup"><span data-stu-id="6ad17-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="6ad17-255">Zie [een Kubernetes-cluster upgraden op Azure stack hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="6ad17-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="6ad17-256">Met de AKS-engine kunt u clusters upgraden naar nieuwere versies van Kubernetes en Base OS-installatie kopieën.</span><span class="sxs-lookup"><span data-stu-id="6ad17-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="6ad17-257">Zie [stappen voor het upgraden naar een nieuwere versie van Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="6ad17-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="6ad17-258">U kunt ook alleen de onderdrukte knoop punten upgraden naar nieuwere versies van installatie kopieën van het basis besturingssysteem.</span><span class="sxs-lookup"><span data-stu-id="6ad17-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="6ad17-259">Zie [stappen voor het uitvoeren van een upgrade van de installatie kopie van het besturings systeem](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="6ad17-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="6ad17-260">Nieuwere installatie kopieën van het basis besturingssysteem bevatten beveiligings-en kernel-updates.</span><span class="sxs-lookup"><span data-stu-id="6ad17-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="6ad17-261">Het is de verantwoordelijkheid van de cluster operator voor het bewaken van de beschik baarheid van nieuwere Kubernetes-versies en installatie kopieën van het besturings systeem.</span><span class="sxs-lookup"><span data-stu-id="6ad17-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="6ad17-262">De operator moet deze upgrades plannen en uitvoeren met behulp van de AKS-engine.</span><span class="sxs-lookup"><span data-stu-id="6ad17-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="6ad17-263">De basis installatie kopieën van het besturings systeem moeten worden gedownload van de Azure Stack hub Marketplace door de operator Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6ad17-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="6ad17-264">Kubernetes schalen</span><span class="sxs-lookup"><span data-stu-id="6ad17-264">Scale Kubernetes</span></span>

<span data-ttu-id="6ad17-265">Schaal is een andere dag 2 bewerking die kan worden georganiseerd met behulp van de AKS-engine.</span><span class="sxs-lookup"><span data-stu-id="6ad17-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="6ad17-266">De schaal opdracht gebruikt uw cluster configuratie bestand (apimodel.jsop) in de uitvoermap, als invoer voor een nieuwe Azure Resource Manager-implementatie.</span><span class="sxs-lookup"><span data-stu-id="6ad17-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="6ad17-267">De AKS-engine voert de schaal bewerking uit op basis van een specifieke agent groep.</span><span class="sxs-lookup"><span data-stu-id="6ad17-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="6ad17-268">Wanneer de schaal bewerking is voltooid, werkt de AKS-engine de cluster definitie bij op hetzelfde apimodel.jsvoor het bestand.</span><span class="sxs-lookup"><span data-stu-id="6ad17-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="6ad17-269">De cluster definitie weerspiegelt het nieuwe aantal knoop punten om de bijgewerkte, huidige cluster configuratie weer te geven.</span><span class="sxs-lookup"><span data-stu-id="6ad17-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="6ad17-270">Een Kubernetes-cluster op Azure Stack hub schalen</span><span class="sxs-lookup"><span data-stu-id="6ad17-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="6ad17-271">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="6ad17-271">Next steps</span></span>

- <span data-ttu-id="6ad17-272">Meer informatie over [overwegingen voor het ontwerpen van hybride apps](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="6ad17-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="6ad17-273">Bekijk en suggereer verbeteringen in [de code voor dit voor beeld op github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="6ad17-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>