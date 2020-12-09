---
title: Kubernetes-patroon met hoge Beschik baarheid met behulp van Azure en Azure Stack hub
description: Meer informatie over hoe een Kubernetes-cluster oplossing hoge Beschik baarheid biedt met behulp van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 454cc0a0531882b7a8ec050a461420ce13eebcfe
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911791"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Kubernetes-cluster patroon met hoge Beschik baarheid

In dit artikel wordt beschreven hoe u een Maxi maal beschik bare Kubernetes-infra structuur met behulp van de Azure Kubernetes service (AKS)-engine op Azure Stack hub architectt en gebruikt. Dit scenario is gebruikelijk voor organisaties met kritieke werk belastingen in zeer beperkte en gereglementeerde omgevingen. Organisaties in domeinen zoals financiën, verdediging en overheid.

## <a name="context-and-problem"></a>Context en probleem

Veel organisaties ontwikkelen Cloud-systeem eigen oplossingen die gebruikmaken van geavanceerde services en technologieën, zoals Kubernetes. Azure biedt data centers in de meeste regio's van de wereld, soms is er sprake van Edge use-cases en scenario's waarin bedrijfs kritieke toepassingen op een specifieke locatie moeten worden uitgevoerd. Overwegingen zijn onder andere:

- Locatie gevoeligheid
- Latentie tussen de toepassing en on-premises systemen
- Bandbreedte behoud
- Connectiviteit
- Wettelijke of wettelijke vereisten

Azure, in combi natie met Azure Stack hub, behandelt de meeste van deze problemen. Hieronder wordt een groot aantal opties, beslissingen en overwegingen gegeven voor een succes volle implementatie van Kubernetes die wordt uitgevoerd op Azure Stack hub.

## <a name="solution"></a>Oplossing

In dit voor beeld wordt ervan uitgegaan dat we een strikte set beperkingen moeten afhandelen. De toepassing moet on-premises worden uitgevoerd en alle persoonlijke gegevens mogen geen open bare Cloud Services bereiken. Bewaking en andere niet-PII-gegevens kunnen naar Azure worden verzonden en daar worden verwerkt. Externe services, zoals een openbaar Container Registry of anderen, kunnen worden geopend, maar kunnen worden gefilterd via een firewall of proxy server.

De voorbeeld toepassing die hier wordt weer gegeven (gebaseerd op [Azure Kubernetes service workshop](/learn/modules/aks-workshop/)) is ontworpen voor gebruik waar mogelijk Kubernetes-systeem eigen oplossingen. Met dit ontwerp voor komt u dat de leverancier wordt vergrendeld, in plaats van platform-systeem eigen services te gebruiken. Als voor beeld gebruikt de toepassing een zelf-hostende MongoDB data base-back-end in plaats van een PaaS-service of externe database service.

[![Toepassings patroon hybride](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

In het voor gaande diagram ziet u de toepassings architectuur van de voorbeeld toepassing die wordt uitgevoerd op Kubernetes op Azure Stack hub. De app bestaat uit verschillende onderdelen, waaronder:

 1) Een op AKS engine gebaseerd Kubernetes-cluster op Azure Stack hub.
 2) [CERT-Manager](https://www.jetstack.io/cert-manager/), waarmee een reeks hulpprogram ma's voor certificaat beheer wordt geboden in Kubernetes, die wordt gebruikt om automatisch certificaten aan te vragen van de code ring.
 3) Een Kubernetes-naam ruimte die de toepassings onderdelen bevat voor de front-end (beoordelingen-web), API (classificaties-API) en data base (ratings-MongoDb).
 4) De ingangs controller die HTTP/HTTPS-verkeer naar eind punten in het Kubernetes-cluster routeert.

De voorbeeld toepassing wordt gebruikt om de toepassings architectuur te illustreren. Alle onderdelen zijn voor beelden. De architectuur bevat slechts één implementatie van een toepassing. Om hoge Beschik baarheid (HA) te kunnen uitvoeren, worden de implementatie ten minste twee keer op twee verschillende Azure Stack hub-instanties uitgevoerd. ze kunnen worden uitgevoerd op dezelfde locatie of op twee (of meer) verschillende sites:

![Infrastructuur architectuur](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Services als Azure Container Registry, Azure Monitor en anderen, worden buiten Azure Stack hub gehost in azure of on-premises. Dit hybride ontwerp beveiligt de oplossing tegen storingen van één Azure Stack hub-exemplaar.

## <a name="components"></a>Onderdelen

De algemene architectuur bestaat uit de volgende onderdelen:

**Azure stack hub** is een uitbrei ding van Azure waarmee werk belastingen kunnen worden uitgevoerd in een on-premises omgeving door Azure-Services in uw Data Center te bieden. Ga naar [Azure stack hub-overzicht](/azure-stack/operator/azure-stack-overview) voor meer informatie.

De **Azure Kubernetes service Engine (AKS-Engine)** is de engine achter de beheerde Kubernetes service-aanbieding, Azure Kubernetes service (AKS), die momenteel beschikbaar is in Azure. Met de AKS-engine van Azure Stack hub kunt u volledig functionele, zelf-beheerde Kubernetes-clusters implementeren, schalen en upgraden met behulp van de IaaS-mogelijkheden van Azure Stack hub. Ga naar [overzicht](https://github.com/Azure/aks-engine) van de AKS-engine voor meer informatie.

Ga naar [bekende problemen en beperkingen](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) voor meer informatie over de verschillen tussen de AKS-engine op Azure en de AKS-engine op Azure stack hub.

**Azure Virtual Network (VNet)** wordt gebruikt om de netwerk infrastructuur op elke Azure stack hub te bieden voor de virtual machines (vm's) die de Kubernetes-cluster infrastructuur hosten.

**Azure Load Balancer** wordt gebruikt voor het Kubernetes API-eind punt en de nginx ingangs controller. De load balancer stuurt externe verkeer (bijvoorbeeld Internet) naar knoop punten en Vm's die een specifieke service bieden.

**Azure container Registry (ACR)** wordt gebruikt voor het opslaan van persoonlijke docker-installatie kopieën en helm-grafieken, die worden geïmplementeerd in het cluster. De AKS-engine kan worden geverifieerd met de Container Registry met behulp van een Azure AD-identiteit. Kubernetes vereist geen ACR. U kunt andere container registers gebruiken, zoals docker hub.

**Azure opslag plaatsen** is een set hulpprogram ma's voor versie beheer die u kunt gebruiken om uw code te beheren. U kunt ook GitHub of andere opslag plaatsen op basis van Git gebruiken. Ga naar [Azure opslag plaatsen Overview](/azure/devops/repos/get-started/what-is-repos) voor meer informatie.

**Azure-pipelines** maakt deel uit van Azure DevOps Services en voert automatische builds, tests en implementaties uit. U kunt ook gebruikmaken van CI/CD-oplossingen van derden, zoals Jenkins. Ga naar [overzicht van Azure-pijp lijn](/azure/devops/pipelines/get-started/what-is-azure-pipelines) voor meer informatie.

**Azure monitor** worden metrische gegevens en logboeken verzameld en opgeslagen, met inbegrip van de metrische gegevens van het platform voor de Azure-Services in de oplossing en de telemetrie van de toepassing. Gebruik deze gegevens om de toepassing te controleren, waarschuwingen en dash boards in te stellen en de belangrijkste oorzaak analyse van fouten uit te voeren. Azure Monitor integreert met Kubernetes voor het verzamelen van metrische gegevens van controllers, knoop punten en containers, evenals container logboeken en hoofd knooppunt Logboeken. Ga naar [Azure Monitor overzicht](/azure/azure-monitor/overview) voor meer informatie.

**Azure Traffic Manager** is een op DNS gebaseerd verkeer Load Balancer waarmee u verkeer optimaal kunt distribueren naar Services in verschillende Azure-regio's of implementaties van Azure stack hub. Traffic Manager biedt ook hoge Beschik baarheid en reactie snelheid. De eind punten van de toepassing moeten toegankelijk zijn vanaf buiten. Er zijn ook andere on-premises oplossingen beschikbaar.

Met de **Kubernetes ingress-controller** worden http (S)-routes naar Services in een Kubernetes-cluster getoond. Nginx of een wille keurige ingangs controller kan voor dit doel worden gebruikt.

**Helm** is een pakket beheerder voor Kubernetes-implementatie, waarmee u verschillende Kubernetes-objecten zoals implementaties, services en geheimen kunt bundelen in één grafiek. U kunt het publiceren, implementeren, beheren van versie beheer en het bijwerken van een grafiek object. Azure Container Registry kunnen worden gebruikt als opslag plaats voor het opslaan van verpakte helm-grafieken.

## <a name="design-considerations"></a>Overwegingen bij het ontwerpen

In de volgende secties van dit artikel wordt dit patroon gevolgd door enkele belang rijke overwegingen voor het hoge niveau:

- De toepassing maakt gebruik van Kubernetes-systeem eigen oplossingen om de vergren deling van een leverancier te voor komen.
- De toepassing maakt gebruik van een micro service architectuur.
- Azure Stack hub heeft geen inkomende toegang nodig, maar maakt een uitgaande Internet verbinding mogelijk.

Deze aanbevolen procedures zijn ook van toepassing op Real-World-workloads en scenario's.

## <a name="scalability-considerations"></a>Schaalbaarheidsoverwegingen

Schaal baarheid is belang rijk om gebruikers consistente, betrouw bare en goed presterende toegang te bieden tot de toepassing.

Het voorbeeld scenario heeft betrekking op schaal baarheid op meerdere lagen van de toepassings stack. Dit is een overzicht op hoog niveau van de verschillende lagen:

| Architectuur niveau | Alleen | Hoe? |
| --- | --- | ---
| Toepassing | Toepassing | Horizon taal schalen op basis van het aantal peulen/Replica's/Container Instances * |
| Cluster | Kubernetes-cluster | Aantal knoop punten (tussen 1 en 50), VM-SKU-grootten en knooppunt Pools (AKS-engine op Azure Stack hub ondersteunt momenteel slechts één knooppunt groep); de schaal opdracht van de AKS-engine gebruiken (hand matig) |
| Infrastructuur | Azure Stack Hub | Aantal knoop punten, capaciteit en schaal eenheden binnen een Azure Stack hub-implementatie |

\* Kubernetes ' Horizontal pod autoscaleer (HPA) gebruiken; geautomatiseerd schalen op basis van metrische gegevens of verticaal schalen door het formaat van de container instanties te verg Roten (CPU/geheugen).

**Azure Stack hub (infrastructuur niveau)**

De infra structuur van Azure Stack hub is de basis van deze implementatie, omdat Azure Stack hub op fysieke hardware in een Data Center wordt uitgevoerd. Wanneer u uw hub-hardware selecteert, moet u keuzen maken voor CPU, geheugen densiteit, opslag configuratie en aantal servers. Raadpleeg de volgende bronnen voor meer informatie over de schaal baarheid van Azure Stack hub:

- [Overzicht van capaciteits planning voor Azure Stack hub](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Extra knoop punten voor schaal eenheden toevoegen in Azure Stack hub](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes-cluster (cluster niveau)**

Het Kubernetes-cluster zelf bestaat uit en is gebaseerd op Azure (stack) IaaS-onderdelen, waaronder reken-, opslag-en netwerk bronnen. Kubernetes-oplossingen hebben betrekking op Master-en worker-knoop punten, die worden geïmplementeerd als Vm's in azure (en Azure Stack hub).

- [Control vlak nodes](/azure/aks/concepts-clusters-workloads#control-plane) (Master) bieden de kern Kubernetes Services en de indeling van werk belastingen van toepassingen.
- [Werk knooppunten](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (werk nemers) voeren de werk belastingen van uw toepassingen uit.

Wanneer u VM-grootten voor de eerste implementatie selecteert, zijn er verschillende overwegingen:  

- **Kosten** : bij het plannen van uw worker-knoop punten houdt u rekening met de totale kosten per VM. Als de werk belasting van uw toepassing bijvoorbeeld beperkte bronnen vereist, moet u plannen om kleinere virtuele machines te implementeren. Azure Stack hub, zoals Azure, wordt normaal gesp roken gefactureerd op basis van verbruik. Daarom is het van groot belang dat de virtuele machines voor Kubernetes-rollen cruciaal zijn voor het optimaliseren van verbruiks kosten. 

- **Schaal baarheid: schaal** baarheid van het cluster wordt bereikt door het aantal hoofd-en werk knooppunten te schalen of door extra knooppunt groepen toe te voegen (die momenteel niet beschikbaar zijn op Azure stack hub). Het schalen van het cluster kan worden uitgevoerd op basis van prestatie gegevens die zijn verzameld met behulp van container Insights (Azure Monitor + Log Analytics). 

    Als uw toepassing meer (of minder) resources nodig heeft, kunt u de huidige knoop punten horizon taal (of in) schalen (tussen 1 en 50 knoop punten). Als u meer dan 50 knoop punten nodig hebt, kunt u een extra cluster maken in een afzonderlijk abonnement. U kunt de werkelijke Vm's niet verticaal schalen naar een andere VM-grootte zonder het cluster opnieuw te implementeren.

    U kunt de schaal hand matig schalen met behulp van de AKS engine helper-VM die in eerste instantie is gebruikt voor het implementeren van het Kubernetes-cluster. Zie [Kubernetes-clusters schalen](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md) voor meer informatie

- **Quota's** : Houd rekening met de [quota's](/azure-stack/operator/azure-stack-quota-types) die u hebt geconfigureerd bij het plannen van een AKS-implementatie op uw Azure stack hub. Zorg ervoor dat voor elk [abonnement](/azure-stack/operator/service-plan-offer-subscription-overview) de juiste plannen en de quota's zijn geconfigureerd. Het abonnement moet voldoen aan de berekenings-, opslag-en andere services die nodig zijn voor uw clusters wanneer deze worden uitgeschaald.

- **Werk belastingen voor toepassingen** : Raadpleeg de [concepten voor clusters en workloads](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) in het Kubernetes core-concepten voor Azure Kubernetes service-document. Dit artikel helpt u bij het bereiken van de juiste VM-grootte op basis van de berekenings-en geheugen behoeften van uw toepassing.  

**Toepassing (toepassings niveau)**

Op de toepassingslaag gebruiken we Kubernetes [horizontal pod autoscaleer (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). HPA kan het aantal Replica's (pod/Container Instances) in onze implementatie verhogen of verlagen op basis van verschillende metrische gegevens (zoals CPU-gebruik).

Een andere optie is om container instanties verticaal te schalen. Dit kan worden bereikt door de hoeveelheid CPU en geheugen die is aangevraagd en beschikbaar te wijzigen voor een specifieke implementatie. Zie [resources beheren voor containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) op kubernetes.io voor meer informatie.

## <a name="networking-and-connectivity-considerations"></a>Overwegingen voor netwerk-en connectiviteit

Netwerken en connectiviteit zijn ook van invloed op de drie lagen die eerder zijn vermeld voor Kubernetes op Azure Stack hub. De volgende tabel bevat de lagen en de services die ze bevatten:

| Laag | Alleen | Wat? |
| --- | --- | ---
| Toepassing | Toepassing | Hoe is de toepassing toegankelijk? Wordt het blootgesteld aan Internet? |
| Cluster | Kubernetes-cluster | Kubernetes-API, AKS engine-VM, container installatie kopieën ophalen (uitgaand), controle gegevens en telemetrie verzenden (uitgaand) |
| Infrastructuur | Azure Stack Hub | Toegankelijkheid van de Azure Stack hub-beheer eindpunten, zoals de portal en Azure Resource Manager-eind punten. |

**Toepassing**

Voor de toepassingslaag is het belang rijk om te bepalen of de toepassing beschikbaar is en toegankelijk is via internet. Vanuit een Kubernetes-perspectief houdt Internet toegankelijkheid bij het weer geven van een implementatie of pod met behulp van een Kubernetes-service of een ingangs controller.

> [!NOTE]
> Het is raadzaam om inkomende controllers te gebruiken om Kubernetes services beschikbaar te maken, omdat het aantal open bare frontend-Ip's op Azure Stack hub beperkt is tot 5. Dit ontwerp beperkt ook het aantal Kubernetes-Services (met het type LoadBalancer) tot 5, wat te klein is voor veel implementaties. Ga naar de [documentatie](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) van de AKS-engine voor meer informatie.

Het beschikbaar maken van een toepassing via een openbaar IP-adres via een Load Balancer of een ingangs controller betekent niet nessecarily dat de toepassing nu toegankelijk is via internet. Het is mogelijk dat Azure Stack hub een openbaar IP-adres heeft dat alleen zichtbaar is op het lokale intranet. niet alle open bare Ip's zijn echt op Internet gericht.

In het vorige blok is het binnenkomend verkeer naar de toepassing beschouwd. Een ander onderwerp dat moet worden overwogen voor een geslaagde implementatie van Kubernetes is uitgaand/uitkomend verkeer. Hier volgen enkele gebruiks voorbeelden waarvoor uitgaand verkeer is vereist:

- Container installatie kopieën ophalen die zijn opgeslagen op DockerHub of Azure Container Registry
- Helm-grafieken ophalen
- Application Insights gegevens (of andere bewakings gegevens) verzenden

Voor sommige bedrijfs omgevingen is het gebruik van _transparante_ of _niet-transparante_ proxy servers vereist. Voor deze servers is specifieke configuratie van de verschillende onderdelen van het cluster vereist. De documentatie van de AKS-engine bevat verschillende details over het beheer van netwerkproxys. Zie [AKS-engine en proxy servers](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md) voor meer informatie.

Ten slotte moet verkeer van meerdere clusters stromen tussen Azure Stack hub-instanties. De voorbeeld implementatie bestaat uit afzonderlijke Kubernetes-clusters die worden uitgevoerd op afzonderlijke instanties van Azure Stack hub. Verkeer ertussen, zoals het replicatie verkeer tussen twee data bases, is externe verkeer. Extern verkeer moet worden gerouteerd via een site-naar-site VPN-of Azure Stack hub open bare IP-adressen om verbinding te maken met Kubernetes op twee Azure Stack hub-instanties:

![verkeer tussen en binnen cluster](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

Het Kubernetes-cluster hoeft niet noodzakelijkerwijs via internet toegankelijk te zijn. Het relevante deel is de Kubernetes-API die wordt gebruikt voor het uitvoeren van een cluster, bijvoorbeeld met behulp van `kubectl` . Het Kubernetes-API-eind punt moet toegankelijk zijn voor iedereen die het cluster exploiteert of toepassingen en services boven op de toepassing implementeert. In dit onderwerp vindt u meer informatie over een DevOps-perspectief in de sectie [overwegingen voor implementatie (CI/cd)](#deployment-cicd-considerations) hieronder.

Op cluster niveau zijn er ook enkele overwegingen rond uitgaand verkeer:

- Knooppunt updates (voor Ubuntu)
- Bewakings gegevens (verzonden naar Azure LogAnalytics)
- Andere agents die uitgaand verkeer vereisen (specifiek voor elke omgeving van de implementatie)

Plan het uiteindelijke netwerk ontwerp voordat u uw Kubernetes-cluster implementeert met behulp van de AKS-engine. In plaats van een toegewezen Virtual Network te maken, kan het efficiënter zijn om een cluster in een bestaand netwerk te implementeren. U kunt bijvoorbeeld gebruikmaken van een bestaande site-naar-site-VPN-verbinding die al is geconfigureerd in uw Azure Stack hub-omgeving.

**Infrastructuur**  

Infra structuur verwijst naar toegang tot de Azure Stack hub-beheer eindpunten. Eind punten zijn onder andere de Tenant-en beheer portals en de Azure Resource Manager-beheer-en Tenant-eind punten. Deze eind punten zijn vereist om Azure Stack hub en de bijbehorende kern services te kunnen uitvoeren.

## <a name="data-and-storage-considerations"></a>Overwegingen voor gegevens en opslag

Er worden twee exemplaren van onze toepassing geïmplementeerd op twee afzonderlijke Kubernetes-clusters, in twee Azure Stack hub-instanties. Aan de hand van dit ontwerp moeten we overwegen hoe u gegevens kunt repliceren en synchroniseren.

Met Azure hebben we de ingebouwde mogelijkheid om opslag te repliceren over meerdere regio's en zones in de Cloud. Momenteel met Azure Stack hub zijn er geen systeem eigen manieren om opslag te repliceren tussen twee verschillende Azure Stack hub-instanties: ze vormen twee onafhankelijke Clouds zonder overkoepelende manier om ze te beheren als een set. Het plannen van de flexibiliteit van toepassingen die worden uitgevoerd op Azure Stack hub dwingt u uit om deze onafhankelijkheid in het ontwerp en de implementatie van uw toepassing te overwegen.

In de meeste gevallen is opslag replicatie niet nodig voor een robuuste en Maxi maal beschik bare toepassing die op AKS wordt geïmplementeerd. U moet echter wel onafhankelijke opslag per Azure Stack hub-exemplaar in het ontwerp van uw toepassing overwegen. Als dit ontwerp een bezorgdheid-of wegblok is voor het implementeren van de oplossing op Azure Stack hub, zijn er mogelijke oplossingen van micro soft-partners die opslag bijlagen bieden. Opslag bijlagen bieden een oplossing voor opslag replicatie op meerdere Azure Stack hubs en Azure. Raadpleeg de [partner oplossingen](#partner-solutions)voor meer informatie.

In onze architectuur werden deze lagen in acht genomen:

**Configuratie**

De configuratie bevat de configuratie van Azure Stack hub, de AKS-engine en het Kubernetes-cluster zelf. De configuratie moet zo veel mogelijk worden geautomatiseerd en als infrastructuur code worden opgeslagen in een versie beheersysteem op basis van Git, zoals Azure DevOps of GitHub. Deze instellingen kunnen niet eenvoudig worden gesynchroniseerd tussen meerdere implementaties. Daarom is het raadzaam om configuratie van buiten te bewaren en toe te passen en DevOps-pijp lijn te gebruiken.

**Toepassing**

De toepassing moet worden opgeslagen in een opslag plaats op basis van Git. Wanneer er een nieuwe implementatie is, wijzigingen in de toepassing of herstel na nood gevallen, kan deze eenvoudig worden geïmplementeerd met behulp van Azure-pijp lijnen.

**Gegevens**

Gegevens zijn de belangrijkste aandachtspunten voor de meeste toepassings ontwerpen. Toepassings gegevens moeten gesynchroniseerd blijven tussen de verschillende exemplaren van de toepassing. Gegevens hebben ook een strategie voor back-ups en herstel na nood gevallen nodig als er sprake is van een storing.

Het bereiken van dit ontwerp is sterk afhankelijk van de technologie opties. Hier volgen enkele voor beelden van oplossingen voor het implementeren van een Data Base op een Maxi maal beschik bare manier op Azure Stack hub:

- [Een SQL Server 2016-beschikbaarheids groep implementeren op Azure en Azure Stack hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Een Maxi maal beschik bare MongoDB-oplossing implementeren op Azure en Azure Stack hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Overwegingen bij het werken met gegevens op meerdere locaties is een nog complexere overweging voor een Maxi maal beschik bare en robuuste oplossing. Overweeg het volgende:

- Latentie en netwerk verbinding tussen Azure Stack hubs.
- Beschik baarheid van identiteiten voor services en machtigingen. Elk Azure Stack hub-exemplaar kan worden geïntegreerd met een externe map. Tijdens de implementatie kunt u Azure Active Directory (Azure AD) of Active Directory Federation Services (ADFS) gebruiken. Het is dus mogelijk om één identiteit te gebruiken die kan communiceren met meerdere onafhankelijke instanties van Azure Stack hub.

## <a name="business-continuity-and-disaster-recovery"></a>Bedrijfscontinuïteit en herstel na noodgevallen

Bedrijfs continuïteit en herstel na nood gevallen (BCDR) is een belang rijk onderwerp in zowel Azure Stack hub als Azure. Het belangrijkste verschil is dat de operator in Azure Stack hub het hele BCDR-proces moet beheren. In Azure worden delen van BCDR automatisch beheerd door micro soft.

BCDR is van invloed op dezelfde gebieden als beschreven in de vorige sectie [gegevens en opslag overwegingen](#data-and-storage-considerations):

- Infra structuur/configuratie
- Toepassings beschikbaarheid
- Toepassingsgegevens

En zoals vermeld in de vorige sectie, zijn deze gebieden de verantwoordelijkheid van de operator Azure Stack hub en kunnen verschillen tussen organisaties. Plan BCDR volgens uw beschik bare Program ma's en processen.

**Infra structuur en configuratie**

In deze sectie worden de fysieke en logische infra structuur en de configuratie van Azure Stack hub beschreven. Het behandelt acties in de beheerder en de Tenant ruimten.

De operator voor de Azure Stack hub (of beheerder) is verantwoordelijk voor het onderhoud van de instanties van de Azure Stack hub. Inclusief onderdelen zoals het netwerk, de opslag, de identiteit en andere onderwerpen die buiten het bereik van dit artikel vallen. Raadpleeg de volgende bronnen voor meer informatie over de details van Azure Stack hub-bewerkingen:

- [Gegevens in Azure Stack hub herstellen met de Infrastructure Backup-Service](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Back-up voor Azure Stack hub inschakelen vanuit de beheer Portal](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Herstellen van onherstelbaar gegevensverlies](/azure-stack/operator/azure-stack-backup-recover-data)
- [Aanbevolen procedures voor de service Infrastructure Backup](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack hub is het platform en de infra structuur waarop Kubernetes toepassingen worden geïmplementeerd. De eigenaar van de toepassing voor de Kubernetes-toepassing is een gebruiker van Azure Stack hub, waarvoor toegang is verleend om de toepassings infrastructuur te implementeren die nodig is voor de oplossing. Toepassings infrastructuur in dit geval betekent het Kubernetes-cluster, geïmplementeerd met behulp van de AKS-engine en de omringende Services. Deze onderdelen worden geïmplementeerd in Azure Stack hub, beperkt door een Azure Stack hub-aanbieding. Zorg ervoor dat de aanbieding die door de eigenaar van de Kubernetes-toepassing is geaccepteerd, voldoende capaciteit heeft (uitgedrukt in Azure Stack hub-quota's) om de volledige oplossing te implementeren. Zoals aanbevolen in de vorige sectie, moet de toepassings implementatie worden geautomatiseerd met behulp van infrastructuur codes en implementatie pijplijnen zoals Azure DevOps Azure-pijp lijnen.

Zie [Azure stack hub Services, plannen, aanbiedingen en abonnementen Overview](/azure-stack/operator/service-plan-offer-subscription-overview) voor meer informatie over Azure stack hub en quota's.

Het is belang rijk om de configuratie van de AKS-engine veilig op te slaan en op te slaan, inclusief de uitvoer. Deze bestanden bevatten vertrouwelijke informatie die wordt gebruikt voor toegang tot het Kubernetes-cluster. Daarom moet het worden beschermd tegen niet-beheerders.

**Toepassings beschikbaarheid**

De toepassing hoeft niet te vertrouwen op back-ups van een geïmplementeerd exemplaar. Als standaard procedure implementeert u de toepassing volledig na de infra structuur-as-code-patronen. Implementeer bijvoorbeeld opnieuw met Azure DevOps Azure-pijp lijnen. De BCDR-procedure moet de toepassing opnieuw implementeren naar dezelfde of een andere Kubernetes-cluster.

**Toepassings gegevens**

Toepassings gegevens vormen een belang rijk onderdeel om gegevens verlies te minimaliseren. In de vorige sectie zijn technieken voor het repliceren en synchroniseren van gegevens tussen twee (of meer) exemplaren van de toepassing beschreven. Afhankelijk van de database infrastructuur (MySQL, MongoDB, MSSQL of anderen) die wordt gebruikt om de gegevens op te slaan, zijn er verschillende Beschik baarheid van de data base en back-uptechnieken beschikbaar om uit te kiezen.

De aanbevolen manieren om integriteit te waarborgen zijn:
- Een systeem eigen back-upoplossing voor de specifieke data base.
- Een back-upoplossing die officieel ondersteuning biedt voor back-up en herstel van het database type dat door uw toepassing wordt gebruikt.

> [!IMPORTANT]
> Sla uw back-upgegevens niet op hetzelfde Azure Stack hub-exemplaar op waar de toepassings gegevens zich bevinden. Een volledige storing van het Azure Stack hub-exemplaar zou ook een inbreuk op uw back-ups veroorzaken.

## <a name="availability-considerations"></a>Beschikbaarheidsoverwegingen

Kubernetes op Azure Stack hub die via de AKS-engine is geïmplementeerd, is geen beheerde service. Het is een geautomatiseerde implementatie en configuratie van een Kubernetes-cluster met behulp van Azure Infrastructure-as-a-Service (IaaS). Dit biedt dezelfde Beschik baarheid als de onderliggende infra structuur.

Azure Stack hub-infra structuur is al flexibel voor fouten en biedt mogelijkheden als beschikbaarheids sets voor het distribueren van onderdelen over meerdere [fout-en update domeinen](/azure-stack/user/azure-stack-vm-considerations#high-availability). Maar de onderliggende technologie (failover clustering) heeft nog steeds enige downtime voor Vm's op een betrokken fysieke server, als er sprake is van een hardwarestoring.

Het is een goed idee om uw productie Kubernetes-cluster en de werk belasting op twee (of meer) clusters te implementeren. Deze clusters moeten worden gehost op verschillende locaties of data centers en technologieën zoals Azure Traffic Manager gebruiken om gebruikers te routeren op basis van de reactie tijd van een cluster of op basis van geografie.

![De verkeers stromen met behulp van Traffic Manager beheren](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Klanten met één Kubernetes-cluster maken doorgaans verbinding met het service-IP-adres of de DNS-naam van een bepaalde toepassing. In een implementatie met meerdere clusters moeten klanten verbinding maken met een Traffic Manager DNS-naam die verwijst naar de services/binnenkomend op elk Kubernetes-cluster.

![Traffic Manager gebruiken om naar een on-premises cluster te routeren](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Dit patroon is ook een [Best Practice voor (beheerde) AKS-clusters in azure](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

Het Kubernetes-cluster zelf, geïmplementeerd via de AKS-engine, moet bestaan uit ten minste drie hoofd knooppunten en twee worker-knoop punten.

## <a name="identity-and-security-considerations"></a>Identiteits-en beveiligings overwegingen

Identiteit en beveiliging zijn belang rijke onderwerpen. Met name wanneer de oplossing onafhankelijk Azure Stack hub-instanties omvat. Kubernetes en Azure (inclusief Azure Stack hub) hebben beide verschillende mechanismen voor op rollen gebaseerd toegangs beheer (RBAC):

- Azure RBAC beheert de toegang tot resources in azure (en Azure Stack hub), met inbegrip van de mogelijkheid om nieuwe Azure-resources te maken. Machtigingen kunnen worden toegewezen aan gebruikers, groepen of service-principals. (Een Service-Principal is een beveiligings identiteit die door toepassingen wordt gebruikt.)
- Kubernetes RBAC beheert machtigingen voor de Kubernetes-API. Een voor beeld: het maken van een Peul en het weer geven van peulen zijn acties die kunnen worden toegestaan (of geweigerd) aan een gebruiker via RBAC. Als u Kubernetes-machtigingen aan gebruikers wilt toewijzen, maakt u rollen en rollen bindingen.

**Azure Stack hub-identiteit en RBAC**

Azure Stack hub biedt twee opties voor de ID-provider. De provider die u gebruikt, is afhankelijk van de omgeving en of deze wordt uitgevoerd in een verbonden of niet-verbonden omgeving:

- Azure AD: kan alleen worden gebruikt in een verbonden omgeving.
- ADFS naar een traditioneel Active Directory forest: kan worden gebruikt in een verbonden of niet-verbonden omgeving.

De ID-provider beheert gebruikers en groepen, met inbegrip van verificatie en autorisatie voor toegang tot resources. Toegang kan worden verleend aan Azure Stack hub-resources, zoals abonnementen, resource groepen en individuele resources, zoals Vm's of load balancers. Als u een consistent toegangs model wilt hebben, kunt u overwegen om dezelfde groepen (direct of genest) te gebruiken voor alle Azure Stack hubs. Hier volgt een voor beeld van een configuratie:

![geneste Aad-groepen met Azure stack-hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

Het voor beeld bevat een specifieke groep (met AAD of ADFS) voor een specifiek doel. Als u bijvoorbeeld Inzender machtigingen wilt verlenen voor de resource groep met de Kubernetes-cluster infrastructuur op een specifiek Azure Stack hub-exemplaar (hier ' Seattle K8s cluster contributor '). Deze groepen worden vervolgens genest in een algemene groep die de subgroepen bevat voor elke Azure Stack hub.

Onze voorbeeld gebruiker heeft nu machtigingen voor ' Inzender ' voor beide resource groepen die de volledige set Kubernetes-infrastructuur resources bevatten. De gebruiker heeft toegang tot resources op beide Azure Stack hub-instanties, omdat de instanties dezelfde id-provider delen.

> [!IMPORTANT]
> Deze machtigingen zijn alleen van invloed op Azure Stack hub en enkele van de resources die hierop zijn geïmplementeerd. Een gebruiker met dit toegangs niveau kan een hoop beschadiging doen, maar heeft geen toegang tot de Kubernetes IaaS Vm's en de Kubernetes-API zonder extra toegang tot de Kubernetes-implementatie.

**Kubernetes identiteit en RBAC**

Voor een Kubernetes-cluster wordt standaard niet dezelfde id-provider gebruikt als voor de Azure Stack hub. De virtuele machines die het Kubernetes-cluster, de Master-en worker-knoop punten hosten, gebruiken de SSH-sleutel die is opgegeven tijdens de implementatie van het cluster. Deze SSH-sleutel is vereist om via SSH verbinding te maken met deze knoop punten.

De Kubernetes-API (bijvoorbeeld toegankelijk via `kubectl` ) wordt ook beveiligd door service accounts, met inbegrip van een standaard-Cluster Administrator-Service account. De referenties voor dit service account worden in eerste instantie opgeslagen in het `.kube/config` bestand op uw Kubernetes-hoofd knooppunten.

**Beheer-en toepassings referenties voor geheimen**

Om geheimen zoals verbindings reeksen of database referenties op te slaan, zijn er verschillende opties, waaronder:

- Azure Key Vault
- Kubernetes Secrets
- oplossingen van derden, zoals HashiCorp-kluis (uitgevoerd op Kubernetes)

Sla geheimen of referenties niet op als tekst zonder opmaak in uw configuratie bestanden, toepassings code of in scripts. En sla ze niet op in een versie beheersysteem. In plaats daarvan moeten de geheimen zo nodig worden opgehaald door de implementatie automatisering.

## <a name="patch-and-update"></a>Patch en update

Het proces **patch en update (PNU)** in de Azure Kubernetes-service is gedeeltelijk geautomatiseerd. Kubernetes-versie-upgrades worden hand matig geactiveerd, terwijl beveiligings updates automatisch worden toegepast. Deze updates kunnen bestaan uit beveiligings correcties voor het besturings systeem of kernel-updates. AKS start deze Linux-knoop punten niet automatisch opnieuw op om het update proces te volt ooien. 

Het PNU-proces voor een Kubernetes-cluster dat is geïmplementeerd met behulp van AKS-engine op Azure Stack hub, is onbeheerd en is de verantwoordelijkheid van de cluster operator. 

De AKS-engine helpt bij de twee belangrijkste taken:

- [Upgrade naar een nieuwere versie van de installatie kopie voor Kubernetes en Base OS](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Alleen de basis installatie kopie van het besturings systeem bijwerken](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Nieuwere basis installatie kopieën van het besturings systeem bevatten de nieuwste beveiligingsfixes en kernel-updates. 

Met het mechanisme voor [upgrade zonder toezicht](https://wiki.debian.org/UnattendedUpgrades) worden automatisch beveiligings updates geïnstalleerd die worden uitgebracht voordat een nieuwe versie van de installatie kopie van het basis besturingssysteem beschikbaar is in de Azure stack hub Marketplace. De upgrade voor een installatie zonder toezicht is standaard ingeschakeld en de beveiligings updates worden automatisch geïnstalleerd, maar de Kubernetes-cluster knooppunten worden niet opnieuw opgestart. Het opnieuw opstarten van de knoop punten kan worden geautomatiseerd met behulp van de open-source [ **K** Ubernetes **opnieuw** opstarten **D** Aemon (kured))](/azure/aks/node-updates-kured). Kured controleert voor Linux-knoop punten die opnieuw moeten worden opgestart en verwerkt vervolgens automatisch het opnieuw plannen van het uitvoeren van het proces en het opnieuw opstarten van het knoop punt.

## <a name="deployment-cicd-considerations"></a>Overwegingen voor implementatie (CI/CD)

Azure en Azure Stack hub bieden dezelfde Azure Resource Manager REST-Api's. Deze Api's worden geadresseerd, zoals elke andere Azure-Cloud (Azure, Azure-China 21Vianet, Azure Government). Er zijn mogelijk verschillen in API-versies tussen Clouds en Azure Stack hub biedt slechts een subset van services. De URI van het beheer eindpunt verschilt ook voor elke Cloud en voor elk exemplaar van Azure Stack hub.

Azure Resource Manager REST Api's bieden een consistente manier om te communiceren met zowel Azure als Azure Stack hub. U kunt hier dezelfde set hulpprogram ma's gebruiken als voor andere Azure-Clouds. U kunt Azure DevOps, hulpprogram ma's zoals Jenkins of Power shell, gebruiken om services te implementeren en te organiseren voor Azure Stack hub.

**Overwegingen**

Een van de belangrijkste verschillen wanneer het gaat om Azure Stack hub-implementaties is de vraag van Internet toegankelijkheid. Internet toegankelijkheid bepaalt of een door micro soft of een zelf-hostende build-agent moet worden geselecteerd voor uw CI/CD-taken.

Een zelf-hostende agent kan worden uitgevoerd op Azure Stack hub (als een IaaS-VM) of in een subnet van een netwerk dat toegang heeft tot Azure Stack hub. Ga naar [Azure pipelines-agents](/azure/devops/pipelines/agents/agents) voor meer informatie over de verschillen.

De volgende afbeelding helpt u om te bepalen of u een zelf-hostend of een door micro soft gehoste build-agent nodig hebt:

![Zelf-hostende build-agents ja of Nee](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Zijn de Azure Stack hub-beheer eindpunten toegankelijk via Internet?
  - Ja: we kunnen Azure-pijp lijnen gebruiken met door micro soft gehoste agents om verbinding te maken met Azure Stack hub.
  - Nee: er zijn zelf-hostende agents nodig die verbinding kunnen maken met de beheer eindpunten van Azure Stack hub.
- Is het Kubernetes-cluster toegankelijk via Internet?
  - Ja: we kunnen Azure-pijp lijnen gebruiken met door micro soft gehoste agents om rechtstreeks te communiceren met het Kubernetes API-eind punt.
  - Nee: er zijn zelf-hostende agents nodig die verbinding kunnen maken met het Kubernetes-cluster-API-eind punt.

In scenario's waarin de Azure Stack hub-beheer-eind punten en Kubernetes-API toegankelijk zijn via internet, kan de implementatie gebruikmaken van een door micro soft gehoste agent. Deze implementatie resulteert in een toepassings architectuur als volgt:

[![Overzicht van open bare architectuur](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Als de Azure Resource Manager-eind punten, Kubernetes-API of beide niet rechtstreeks toegankelijk zijn via internet, kunnen we gebruikmaken van een zelf-hostende build-agent om de pijplijn stappen uit te voeren. Dit ontwerp heeft minder connectiviteit nodig en kan worden geïmplementeerd met alleen on-premises netwerk connectiviteit met Azure Resource Manager-eind punten en de Kubernetes-API:

[![Overzicht van on-premises architectuur](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Hoe zit het met niet-verbonden scenario's?** In scenario's waarin een Azure Stack hub of Kubernetes of beide niet over Internet gerichte beheer eindpunten beschikken, is het nog steeds mogelijk om Azure DevOps te gebruiken voor uw implementaties. U kunt een zelf-hostende agent groep gebruiken (een DevOps-agent die on-premises of op Azure Stack hub zelf wordt uitgevoerd) of een volledig zelf-hostend Azure DevOps Server on-premises. De zelf-hostende agent heeft alleen uitgaande HTTPS (TCP/443) Internet verbinding nodig.

Het patroon kan gebruikmaken van een Kubernetes-cluster (geïmplementeerd en gedistribueerd met AKS-Engine) op elk Azure Stack hub-exemplaar. Het bevat een toepassing die bestaat uit een front-end, een mid-tier, backend-services (bijvoorbeeld MongoDB) en een op nginx gebaseerde ingangs controller. In plaats van een Data Base die wordt gehost op het K8s-cluster, kunt u gebruikmaken van "externe gegevens opslag". Database opties zijn onder andere MySQL, SQL Server, of een Data Base die buiten Azure Stack hub of in IaaS wordt gehost. Configuraties zoals dit zijn niet in het bereik.

## <a name="partner-solutions"></a>Partneroplossingen

Er zijn micro soft-partner oplossingen die de mogelijkheden van Azure Stack hub kunnen uitbreiden. Deze oplossingen zijn handig voor implementaties van toepassingen die worden uitgevoerd op Kubernetes-clusters.  

## <a name="storage-and-data-solutions"></a>Opslag-en gegevens oplossingen

Zoals beschreven in [overwegingen voor gegevens en opslag](#data-and-storage-considerations), heeft Azure stack hub momenteel geen systeem eigen oplossing om opslag over meerdere exemplaren te repliceren. In tegens telling tot Azure bestaat de mogelijkheid om opslag te repliceren over meerdere regio's niet. In Azure Stack hub is elk exemplaar een eigen afzonderlijke Cloud. Er zijn echter oplossingen beschikbaar van micro soft-partners die opslag replicatie mogelijk maken via Azure Stack hubs en Azure. 

**Schaal baarheid**

Met [schaal](https://www.scality.com/) levert de opslag op webschaal die is ingeschakeld voor digitale bedrijven sinds 2009. De schaal RING, onze door de software gedefinieerde opslag, maakt het mogelijk dat x86-servers met een grote beslag worden omgezet in een onbeperkte opslag groep voor elk type gegevens – bestand en object – op PETA byte schaal.

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) vereenvoudigt bedrijfs opslag met onbeperkte schaal bare opslag waarmee enorme gegevens sets worden samengevoegd tot één eenvoudig beheerde omgeving.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de concepten die in dit artikel worden geïntroduceerd:

- [Meerdere Cloud schalen](pattern-cross-cloud-scale.md) en patronen van [geografisch gedistribueerde apps](pattern-geo-distributed.md) in azure stack hub.
- [Architectuur van micro Services op Azure Kubernetes service (AKS)](/azure/architecture/reference-architectures/microservices/aks).

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u door met de [Kubernetes voor maximale Beschik baarheid van cluster implementatie](solution-deployment-guide-highly-available-kubernetes.md). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.