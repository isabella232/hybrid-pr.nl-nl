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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Een Kubernetes-cluster met hoge Beschik baarheid implementeren op Azure Stack hub

In dit artikel wordt uitgelegd hoe u een Maxi maal beschik bare Kubernetes-cluster omgeving kunt bouwen, die op meerdere Azure Stack hub-instanties wordt geïmplementeerd op verschillende fysieke locaties.

In deze hand leiding voor implementaties van oplossingen leert u het volgende:

> [!div class="checklist"]
> - De AKS-engine downloaden en voorbereiden
> - Verbinding maken met de AKS engine helper-VM
> - Een Kubernetes-cluster implementeren
> - Verbinding maken met het Kubernetes-cluster
> - Azure-pijp lijnen verbinden met het Kubernetes-cluster
> - Bewaking configureren
> - Toepassing implementeren
> - Toepassing automatisch schalen
> - Traffic Manager configureren
> - Kubernetes bijwerken
> - Kubernetes schalen

> [!Tip]  
> ![Hybride pijlers](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

## <a name="prerequisites"></a>Vereisten

Voordat u aan de slag gaat met deze implementatie handleiding, moet u het volgende doen:

- Bekijk het artikel over het [Kubernetes cluster patroon met hoge Beschik baarheid](pattern-highly-available-kubernetes.md) .
- Bekijk de inhoud van de [opslag plaats github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), die aanvullende assets bevat waarnaar in dit artikel wordt verwezen.
- Beschikken over een account dat toegang heeft tot de [Azure stack hub-gebruikers Portal](/azure-stack/user/azure-stack-use-portal), met ten minste de [machtiging Inzender](/azure-stack/user/azure-stack-manage-permissions).

## <a name="download-and-prepare-aks-engine"></a>De AKS-engine downloaden en voorbereiden

De AKS-engine is een binair bestand dat kan worden gebruikt vanaf elke Windows-of Linux-host die de Azure Stack hub Azure Resource Manager eind punten kan bereiken. In deze hand leiding wordt beschreven hoe u een nieuwe virtuele Linux-machine (of Windows) implementeert op Azure Stack hub. Deze wordt later gebruikt wanneer de AKS-engine de Kubernetes-clusters implementeert.

> [!NOTE]
> U kunt ook een bestaande virtuele machine met Windows of Linux gebruiken om een Kubernetes-cluster op Azure Stack hub te implementeren met behulp van de AKS-engine.

Het stapsgewijze proces en de vereisten voor de AKS-engine worden hier beschreven:

* [De AKS-Engine installeren op Linux in azure stack hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (of met behulp van [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

De AKS-engine is een hulp programma voor het implementeren en uitvoeren van (niet-beheerde) Kubernetes-clusters (in Azure en Azure Stack hub).

De details en verschillen van de AKS-engine op Azure Stack hub worden hier beschreven:

* [Wat is de AKS-engine op Azure Stack hub?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [AKS-engine op Azure stack hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (op github)

In de voorbeeld omgeving wordt terraform gebruikt voor het automatiseren van de implementatie van de AKS-engine-VM. U kunt de [Details en code vinden in de GitHub opslag plaats](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

Het resultaat van deze stap is een nieuwe resource groep op Azure Stack hub die de AKS engine helper-VM en gerelateerde resources bevat:

![VM-resources van AKS-engine in Azure Stack hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Als u de AKS-engine in een niet-verbonden lucht-gapped-omgeving moet implementeren, controleert u de [losgekoppelde Azure stack hub-instanties](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) voor meer informatie.

In de volgende stap gebruiken we de zojuist geïmplementeerde VM van de AKS-Engine om een Kubernetes-cluster te implementeren.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Verbinding maken met de AKS engine helper-VM

Eerst moet u verbinding maken met de eerder gemaakte AKS engine helper-VM.

De virtuele machine moet een openbaar IP-adres hebben en moet toegankelijk zijn via SSH (poort 22/TCP).

![Overzichts pagina voor de VM van AKS-engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> U kunt een hulp programma gebruiken van uw keuze als MobaXterm, puTTy of Power shell in Windows 10 om verbinding te maken met een virtuele Linux-machine met behulp van SSH.

```console
ssh <username>@<ipaddress>
```

Nadat u verbinding hebt gemaakt, voert u de opdracht uit `aks-engine` . Ga naar [ondersteunde AKS-Engine versies](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) voor meer informatie over de AKS-engine en Kubernetes-versies.

![AKS-engine-opdracht regel voorbeeld](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Een Kubernetes-cluster implementeren

De AKS engine helper-VM zelf heeft nog geen Kubernetes-cluster gemaakt op onze Azure Stack hub. Het maken van het cluster is de eerste actie die moet worden uitgevoerd in de AKS engine helper-VM.

Het stapsgewijze proces wordt hier beschreven:

* [Een Kubernetes-cluster implementeren met de AKS-engine op Azure Stack hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

Het eind resultaat van de `aks-engine deploy` opdracht en de voor bereidingen in de vorige stappen is een volledig aanbevolen Kubernetes-cluster dat is geïmplementeerd in de Tenant ruimte van het eerste Azure stack hub-exemplaar. Het cluster zelf bestaat uit Azure IaaS-onderdelen, zoals Vm's, load balancers, VNets, disks, enzovoort.

![IaaS-onderdelen van cluster Azure Stack hub-Portal](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure load balancer (K8s API-eind punt)
2) Worker-knoop punten (agent groep)
3) Hoofd knooppunten

Het cluster is nu actief en in de volgende stap wordt er verbinding mee gemaakt.

## <a name="connect-to-the-kubernetes-cluster"></a>Verbinding maken met het Kubernetes-cluster

U kunt nu verbinding maken met het eerder gemaakte Kubernetes-cluster via SSH (met behulp van de SSH-sleutel die is opgegeven als onderdeel van de implementatie) of via `kubectl` (aanbevolen). Het opdracht regel programma Kubernetes `kubectl` is [hier](https://kubernetes.io/docs/tasks/tools/install-kubectl/)beschikbaar voor Windows, Linux en macOS. Het is al vooraf geïnstalleerd en geconfigureerd op de hoofd knooppunten van het cluster.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Kubectl uitvoeren op hoofd knooppunt](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Het is niet raadzaam om het hoofd knooppunt te gebruiken als een JumpBox voor beheer taken. De `kubectl` configuratie wordt opgeslagen in `.kube/config` op de hoofd knooppunten en op de virtuele machine met de AKS-engine. U kunt de configuratie kopiëren naar een beheer computer met verbinding met het Kubernetes-cluster en de `kubectl` opdracht hier gebruiken. Het `.kube/config` bestand wordt ook later gebruikt voor het configureren van een service verbinding in azure-pijp lijnen.

> [!IMPORTANT]
> Bewaar deze bestanden als veilig omdat ze de referenties voor uw Kubernetes-cluster bevatten. Een aanvaller met toegang tot het bestand beschikt over voldoende informatie om beheerders toegang te krijgen. Alle acties die met het eerste bestand worden uitgevoerd `.kube/config` , worden uitgevoerd met behulp van een cluster-beheerders account.

U kunt nu verschillende opdrachten gebruiken `kubectl` om de status van uw cluster te controleren.

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
> Kubernetes heeft een eigen _ *op rollen gebaseerd Access Control (RBAC)** model waarmee u verfijnde roldefinities en functie bindingen kunt maken. Dit is de voorkeurs manier om de toegang tot het cluster te beheren in plaats van de machtigingen voor cluster beheer af te leveren.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Azure-pijp lijnen verbinden met Kubernetes-clusters

Als u Azure-pijp lijnen wilt verbinden met het zojuist geïmplementeerde Kubernetes-cluster, hebt u het bestand uitvoeren config ( `.kube/config` ) nodig zoals in de vorige stap wordt uitgelegd.

* Maak verbinding met een van de hoofd knooppunten van uw Kubernetes-cluster.
* Kopieer de inhoud van het `.kube/config` bestand.
* Ga naar Azure DevOps > project instellingen > service verbindingen om een nieuwe ' Kubernetes-service verbinding te maken (gebruik KubeConfig als verificatie methode)

> [!IMPORTANT]
> Azure-pijp lijnen (of de bijbehorende bouw agenten) moeten toegang hebben tot de Kubernetes-API. Als er een Internet verbinding van Azure-pijp lijnen naar de Azure Stack hub Kubernetes clusetr, moet u een zelf-hostende Azure pipelines build-agent implementeren.

Wanneer u zelf-hostende agents voor Azure-pijp lijnen implementeert, kunt u implementeren op Azure Stack hub of op een computer met een netwerk verbinding met alle vereiste beheer eindpunten. Bekijk de details hier:

* [Azure pipelines-agents](/azure/devops/pipelines/agents/agents) in [Windows](/azure/devops/pipelines/agents/v2-windows) of [Linux](/azure/devops/pipelines/agents/v2-linux)

De sectie [overwegingen bij het implementeren van patronen (CI/cd)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) bevat een beslissings stroom waarmee u kunt begrijpen of u door micro soft gehoste agents of zelf-hostende agents wilt gebruiken:

[![zelf gehoste agents in besluit stroom](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

In deze voorbeeld oplossing bevat de topologie een zelf-hostende build-agent op elk Azure Stack hub-exemplaar. De agent kan toegang krijgen tot de Azure Stack hub-beheer eindpunten en de Kubernetes-cluster-API-eind punten.

[![alleen uitgaand verkeer](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Dit ontwerp voldoet aan een gemeen schappelijke wettelijke vereiste, die alleen uitgaande verbindingen van de toepassings oplossing moet hebben.

## <a name="configure-monitoring"></a>Bewaking configureren

U kunt [Azure monitor](/azure/azure-monitor/) voor containers gebruiken om de containers in de oplossing te bewaken. Dit wijst Azure Monitor naar het door de AKS-engine geïmplementeerde Kubernetes-cluster op Azure Stack hub.

Er zijn twee manieren om Azure Monitor in te scha kelen in uw cluster. Op beide manieren moet u een Log Analytics-werk ruimte instellen in Azure.

* [Methode 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) maakt gebruik van een helm-grafiek
* [Methode twee](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) als onderdeel van de cluster specificatie van de AKS-engine

In de voor beeld-topologie wordt ' methode één ' gebruikt, waardoor automatisering van het proces en updates eenvoudiger kunnen worden geïnstalleerd.

Voor de volgende stap hebt u een Azure LogAnalytics-werk ruimte (ID en sleutel), `Helm` (versie 3) en `kubectl` op uw computer nodig.

Helm is een Kubernetes-pakket beheer dat beschikbaar is als binaire waarde die wordt uitgevoerd op macOS, Windows en Linux. Dit kan hier worden gedownload: [helm.sh](https://helm.sh/docs/intro/quickstart/) helm is afhankelijk van het Kubernetes-configuratie bestand dat voor de opdracht wordt gebruikt `kubectl` .

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Met deze opdracht wordt de Azure Monitor-agent geïnstalleerd op uw Kubernetes-cluster:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Met de OMS-agent (Operations Management Suite) in uw Kubernetes-cluster worden bewakings gegevens naar uw Azure Log Analytics-werk ruimte verzonden (met uitgaande HTTPS). U kunt Azure Monitor nu gebruiken om dieper inzicht te krijgen in uw Kubernetes-clusters op Azure Stack hub. Dit ontwerp is een krachtige manier om de kracht van analyses te demonstreren die automatisch kan worden geïmplementeerd met de clusters van uw toepassing.

[![Azure Stack hub-clusters in azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Details van Azure Monitor cluster](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Als Azure Monitor geen Azure Stack hub-gegevens weergeeft, moet u ervoor zorgen dat u zorgvuldig de instructies voor [het toevoegen van AzureMonitor-Containers oplossing aan een Azure Loganalytics-werk ruimte](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) hebt gevolgd.

## <a name="deploy-the-application"></a>De toepassing implementeren

Voordat u de voorbeeld toepassing installeert, is er nog een stap voor het configureren van de op nginx gebaseerde ingangs controller op het Kubernetes-cluster. De ingangs controller wordt gebruikt als laag 7 load balancer het verkeer routeren in het cluster op basis van de host, het pad of het protocol. Nginx-inkomend is beschikbaar als een helm-grafiek. Raadpleeg de [helm Chart github-opslag plaats](https://github.com/helm/charts/tree/master/stable/nginx-ingress)voor gedetailleerde instructies.

Onze voorbeeld toepassing is ook verpakt als een helm-grafiek, zoals de [Azure Monitoring Agent](#configure-monitoring) in de vorige stap. Zo is het eenvoudig om de toepassing te implementeren op het Kubernetes-cluster. U kunt de [helm-grafiek bestanden vinden in de Companion github opslag plaats](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)

De voorbeeld toepassing is een toepassing met drie lagen, geïmplementeerd op een Kubernetes-cluster op elk van de twee Azure Stack hub-instanties. De toepassing maakt gebruik van een MongoDB-data base. Meer informatie over hoe u de gegevens kunt ophalen die worden gerepliceerd tussen meerdere exemplaren van de patroon [gegevens en opslag overwegingen](pattern-highly-available-kubernetes.md#data-and-storage-considerations).

Nadat u het helm-diagram voor de toepassing hebt geïmplementeerd, ziet u dat alle drie de lagen van uw toepassing worden weer gegeven als implementaties en stateful sets (voor de data base) met één Pod:

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

Aan de hand van de Services vindt u de nginx ingangs controller en het bijbehorende open bare IP-adres:

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

Het adres ' External IP ' is ons eind punt van de toepassing. Het is hoe gebruikers verbinding maken om de toepassing te openen en worden ook gebruikt als het eind punt voor onze volgende stap [Traffic Manager configureren](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>De toepassing automatisch schalen
U kunt eventueel de [horizontale pod autoscaleer](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) configureren om omhoog of omlaag te schalen op basis van bepaalde metrische gegevens, zoals CPU-gebruik. Met de volgende opdracht wordt een horizontale pod voor automatisch schalen gemaakt waarmee 1 tot 10 replica's van de Peuling wordt beheerd door de classificatie-webimplementatie. HPA verhoogt en verlaagt het aantal replica's (via de implementatie) om een gemiddeld CPU-gebruik voor alle aantallen van 80% te behouden.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
U kunt de huidige status van automatisch schalen controleren door het volgende uit te voeren:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Traffic Manager configureren

Als u verkeer tussen twee (of meer) implementaties van de toepassing wilt distribueren, gebruiken we [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview). Azure Traffic Manager is een op DNS gebaseerd verkeer load balancer in Azure.

> [!NOTE]
> Traffic Manager DNS gebruikt om client aanvragen door te sturen naar het meest geschikte service-eind punt, op basis van een routerings methode voor verkeer en de status van de eind punten.

In plaats van Azure Traffic Manager kunt u ook andere wereld wijde oplossingen voor taak verdeling gebruiken die lokaal worden gehost. In het voorbeeld scenario gebruiken we Azure Traffic Manager voor het distribueren van verkeer tussen twee exemplaren van onze toepassing. Ze kunnen op dezelfde of verschillende locaties op Azure Stack hub-instanties worden uitgevoerd:

![on-premises Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

In azure configureren we Traffic Manager om te verwijzen naar de twee verschillende exemplaren van onze toepassing:

[![TM-eindpunt profiel](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Zoals u ziet, verwijzen de twee eind punten naar de twee exemplaren van de geïmplementeerde toepassing uit de [vorige sectie](#deploy-the-application).

Op dit punt:
- De Kubernetes-infra structuur is gemaakt, met inbegrip van een ingangs controller.
- Clusters zijn geïmplementeerd in twee Azure Stack hub-exemplaren.
- Bewaking is geconfigureerd.
- Er wordt door Azure Traffic Manager taak verdeling van verkeer via de twee Azure Stack hub-exemplaren.
- Op basis van deze infra structuur is de voor beeld-toepassing met drie lagen op een geautomatiseerde manier geïmplementeerd met behulp van helm-grafieken. 

De oplossing moet nu toegankelijk zijn voor gebruikers.

Er zijn ook een aantal operationele aandachtspunten na de implementatie die in de volgende twee secties worden besproken.

## <a name="upgrade-kubernetes"></a>Kubernetes bijwerken

Houd rekening met de volgende onderwerpen bij het upgraden van het Kubernetes-cluster:

- Het bijwerken van een Kubernetes-cluster is een complexe bewerking van de dag 2 die kan worden uitgevoerd met behulp van de AKS-engine. Zie [een Kubernetes-cluster upgraden op Azure stack hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)voor meer informatie.
- Met de AKS-engine kunt u clusters upgraden naar nieuwere versies van Kubernetes en Base OS-installatie kopieën. Zie [stappen voor het upgraden naar een nieuwere versie van Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)voor meer informatie. 
- U kunt ook alleen de onderdrukte knoop punten upgraden naar nieuwere versies van installatie kopieën van het basis besturingssysteem. Zie [stappen voor het uitvoeren van een upgrade van de installatie kopie van het besturings systeem](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)voor meer informatie.

Nieuwere installatie kopieën van het basis besturingssysteem bevatten beveiligings-en kernel-updates. Het is de verantwoordelijkheid van de cluster operator voor het bewaken van de beschik baarheid van nieuwere Kubernetes-versies en installatie kopieën van het besturings systeem. De operator moet deze upgrades plannen en uitvoeren met behulp van de AKS-engine. De basis installatie kopieën van het besturings systeem moeten worden gedownload van de Azure Stack hub Marketplace door de operator Azure Stack hub.

## <a name="scale-kubernetes"></a>Kubernetes schalen

Schaal is een andere dag 2 bewerking die kan worden georganiseerd met behulp van de AKS-engine.

De schaal opdracht gebruikt uw cluster configuratie bestand (apimodel.jsop) in de uitvoermap, als invoer voor een nieuwe Azure Resource Manager-implementatie. De AKS-engine voert de schaal bewerking uit op basis van een specifieke agent groep. Wanneer de schaal bewerking is voltooid, werkt de AKS-engine de cluster definitie bij op hetzelfde apimodel.jsvoor het bestand. De cluster definitie weerspiegelt het nieuwe aantal knoop punten om de bijgewerkte, huidige cluster configuratie weer te geven.

- [Een Kubernetes-cluster op Azure Stack hub schalen](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Volgende stappen

- Meer informatie over [overwegingen voor het ontwerpen van hybride apps](overview-app-design-considerations.md)
- Bekijk en suggereer verbeteringen in [de code voor dit voor beeld op github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).