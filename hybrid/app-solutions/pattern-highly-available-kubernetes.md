---
title: Kubernetes-patroon met hoge beschikbaarheid met behulp van Azure en Azure Stack Hub
description: Meer informatie over hoe een Kubernetes-clusteroplossing hoge beschikbaarheid biedt met behulp van Azure en Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281309"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Kubernetes-clusterpatroon met hoge beschikbaarheid

In dit artikel wordt beschreven hoe u een op Kubernetes gebaseerde infrastructuur met hoge mate ontwerpt en gebruikt met behulp van Azure Kubernetes Service(AKS)-engine op Azure Stack Hub. Dit scenario is gebruikelijk voor organisaties met kritieke workloads in zeer beperkte en gereguleerde omgevingen. Organisaties in domeinen zoals financiën, verdediging en overheid.

## <a name="context-and-problem"></a>Context en probleem

Veel organisaties ontwikkelen cloudeigen oplossingen die gebruikmaken van geavanceerde services en technologieën zoals Kubernetes. Hoewel Azure datacenters biedt in de meeste regio's van de wereld, zijn er soms randgebruiksscenario's waarin bedrijfskritieke toepassingen op een specifieke locatie moeten worden uitgevoerd. Overwegingen zijn onder andere:

- Gevoeligheid van locatie
- Latentie tussen de toepassing en on-premises systemen
- Bandbreedte-bandbreedte
- Connectiviteit
- Wettelijke of wettelijke vereisten

Azure lost in combinatie met Azure Stack Hub de meeste van deze problemen op. Hieronder vindt u een uitgebreide reeks opties, beslissingen en overwegingen voor een geslaagde implementatie van Kubernetes op Azure Stack Hub.

## <a name="solution"></a>Oplossing

Bij dit patroon wordt ervan uitgenomen dat we te maken hebben met een strikte set beperkingen. De toepassing moet on-premises worden uitgevoerd en alle persoonlijke gegevens mogen geen openbare cloudservices bereiken. Bewakingsgegevens en andere niet-PII-gegevens kunnen naar Azure worden verzonden en daar worden verwerkt. Externe services, zoals een openbare Container Registry of andere, zijn toegankelijk, maar kunnen worden gefilterd via een firewall of proxyserver.

De voorbeeldtoepassing die hier wordt weergegeven [(op basis van Azure Kubernetes Service Workshop](/learn/modules/aks-workshop/)) is ontworpen om waar mogelijk kubernetes-eigen oplossingen te gebruiken. Dit ontwerp voorkomt dat leveranciers worden vergrendeld in plaats van platformeigen services te gebruiken. De toepassing maakt bijvoorbeeld gebruik van een zelf-hostend MongoDB-databaseback-end in plaats van een PaaS-service of externe databaseservice.

[![Hybride toepassingspatroon](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Het voorgaande diagram illustreert de toepassingsarchitectuur van de voorbeeldtoepassing die wordt uitgevoerd op Kubernetes op Azure Stack Hub. De app bestaat uit verschillende onderdelen, waaronder:

 1) Een kubernetes-cluster op basis van een AKS-engine op Azure Stack Hub.
 2) [certificaatbeheer, dat](https://www.jetstack.io/cert-manager/)een reeks hulpprogramma's biedt voor certificaatbeheer in Kubernetes, die wordt gebruikt om automatisch certificaten aan te vragen bij Let's Encrypt.
 3) Een Kubernetes-naamruimte die de toepassingsonderdelen voor de front-end (ratings-web), API (ratings-api) en database (ratings-mongodb) bevat.
 4) De toegangscontroller die HTTP/HTTPS-verkeer routeeert naar eindpunten binnen het Kubernetes-cluster.

De voorbeeldtoepassing wordt gebruikt om de toepassingsarchitectuur te illustreren. Alle onderdelen zijn voorbeelden. De architectuur bevat slechts één toepassingsimplementatie. Om hoge beschikbaarheid (HA) te bereiken, voeren we de implementatie ten minste twee keer uit op twee verschillende Azure Stack Hub-exemplaren. Ze kunnen worden uitgevoerd op dezelfde locatie of op twee (of meer) verschillende sites:

![Infrastructuurarchitectuur](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Services zoals Azure Container Registry, Azure Monitor en andere services worden buiten Azure Stack Hub gehost in Azure of on-premises. Dit hybride ontwerp beschermt de oplossing tegen uitval van één Azure Stack Hub exemplaar.

## <a name="components"></a>Onderdelen

De algehele architectuur bestaat uit de volgende onderdelen:

**Azure Stack Hub** is een uitbreiding van Azure die workloads kan uitvoeren in een on-premises omgeving door Azure-services te bieden in uw datacenter. Ga naar [Azure Stack Hub overzicht](/azure-stack/operator/azure-stack-overview) voor meer informatie.

**Azure Kubernetes Service Engine (AKS Engine)** is de engine achter het beheerde Kubernetes-serviceaanbod, Azure Kubernetes Service (AKS), dat momenteel beschikbaar is in Azure. Voor Azure Stack Hub kunnen we met de AKS-engine volledig uitgeruste, zelf-beheerde Kubernetes-clusters implementeren, schalen en upgraden met behulp van de IaaS-mogelijkheden van Azure Stack Hub. Ga naar [Overzicht van de AKS-engine](https://github.com/Azure/aks-engine) voor meer informatie.

Ga naar [Bekende problemen en beperkingen voor](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) meer informatie over de verschillen tussen de AKS-engine in Azure en de AKS-engine op Azure Stack Hub.

**Azure Virtual Network (VNet)** wordt gebruikt om de netwerkinfrastructuur op elke Azure Stack Hub te leveren voor de Virtual Machines (VM's) die als host voor de Kubernetes-clusterinfrastructuur worden gebruikt.

**Azure Load Balancer** wordt gebruikt voor het Kubernetes API-eindpunt en de Nginx-toegangscontroller. De load balancer routeert extern verkeer (bijvoorbeeld internet) naar knooppunten en VM's die een specifieke service aanbieden.

**Azure Container Registry (ACR)** wordt gebruikt voor het opslaan van persoonlijke Docker-afbeeldingen en Helm-grafieken, die in het cluster worden geïmplementeerd. De AKS-engine kan worden geverifieerd met Container Registry met behulp van een Azure AD-identiteit. Kubernetes vereist geen ACR. U kunt andere containerregisters gebruiken, zoals Docker Hub.

**Azure-repos** is een set hulpprogramma's voor versiebeheer die u kunt gebruiken om uw code te beheren. U kunt ook GitHub of andere git-opslagplaatsen gebruiken. Ga naar [Overzicht van Azure-repos](/azure/devops/repos/get-started/what-is-repos) voor meer informatie.

**Azure Pipelines** maakt deel uit van Azure DevOps Services en voert geautomatiseerde builds, tests en implementaties uit. U kunt ook CI/CD-oplossingen van derden gebruiken, zoals Jenkins. Ga naar [Overzicht van Azure Pipeline](/azure/devops/pipelines/get-started/what-is-azure-pipelines) voor meer informatie.

**Azure Monitor** verzamelt en slaat metrische gegevens en logboeken op, inclusief metrische platformgegevens voor de Azure-services in de oplossing en toepassings-telemetrie. Gebruik deze gegevens om de toepassing te bewaken, waarschuwingen en dashboards in te stellen en een hoofdoorzaakanalyse van fouten uit te voeren. Azure Monitor kan worden geïntegreerd met Kubernetes voor het verzamelen van metrische gegevens van controllers, knooppunten en containers, evenals containerlogboeken en logboeken van hoofdknooppunten. Ga naar [Azure Monitor Overzicht](/azure/azure-monitor/overview) voor meer informatie.

**Azure Traffic Manager** is een op DNS gebaseerde verkeers-load balancer waarmee u verkeer optimaal kunt distribueren naar services in verschillende Azure-regio's of Azure Stack Hub implementaties. Traffic Manager biedt ook hoge beschikbaarheid en reactiesnelheid. De eindpunten van de toepassing moeten van buitenaf toegankelijk zijn. Er zijn ook andere on-premises oplossingen beschikbaar.

**Kubernetes Ingress Controller** toont HTTP(S)-routes naar services in een Kubernetes-cluster. Voor dit doel kan Nginx of een geschikte controller voor binnengressen worden gebruikt.

**Helm** is een pakketbeheerder voor Kubernetes-implementatie en biedt een manier om verschillende Kubernetes-objecten, zoals Implementaties, Services, Geheimen, te bundelen in één 'grafiek'. U kunt een grafiekobject publiceren, implementeren, versiebeheer beheren en bijwerken. Azure Container Registry kunnen worden gebruikt als opslagplaats voor het opslaan van verpakte Helm-grafieken.

## <a name="design-considerations"></a>Overwegingen bij het ontwerpen

Dit patroon volgt enkele overwegingen op hoog niveau die in meer detail worden uitgelegd in de volgende secties van dit artikel:

- De toepassing maakt gebruik van systeemeigen Kubernetes-oplossingen om te voorkomen dat leveranciers worden vergrendeld.
- De toepassing maakt gebruik van een microservicearchitectuur.
- Azure Stack Hub heeft geen inkomende verbindingen nodig, maar maakt uitgaande internetverbinding mogelijk.

Deze aanbevolen procedures zijn ook van toepassing op workloads en scenario's in de praktijk.

## <a name="scalability-considerations"></a>Schaalbaarheidsoverwegingen

Schaalbaarheid is belangrijk om gebruikers consistente, betrouwbare en goed presterende toegang tot de toepassing te bieden.

Het voorbeeldscenario heeft betrekking op schaalbaarheid op meerdere lagen van de toepassingsstack. Hier is een overzicht op hoog niveau van de verschillende lagen:

| Architectuurniveau | Beïnvloedt | Hoe? |
| --- | --- | ---
| Toepassing | Toepassing | Horizontaal schalen op basis van het aantal pods/replica's/Container Instances* |
| Cluster | Kubernetes-cluster | Aantal knooppunten (tussen 1 en 50), VM-SKU-grootten en knooppuntgroepen (AKS-engine op Azure Stack Hub ondersteunt momenteel slechts één knooppuntgroep); de schaalopdracht van de AKS-engine gebruiken (handmatig) |
| Infrastructuur | Azure Stack Hub | Aantal knooppunten, capaciteit en schaaleenheden binnen een Azure Stack Hub implementatie |

\* De horizontale automatische schaal van pods (HPA) van Kubernetes gebruiken; geautomatiseerde schaalbaarheid op basis van metrische gegevens of verticaal schalen door de container-exemplaren (cpu/geheugen) te schalen.

**Azure Stack Hub (infrastructuurniveau)**

De Azure Stack Hub infrastructuur is de basis van deze implementatie, omdat Azure Stack Hub wordt uitgevoerd op fysieke hardware in een datacenter. Wanneer u uw Hub-hardware selecteert, moet u keuzes maken voor CPU, geheugendichtheid, opslagconfiguratie en het aantal servers. Raadpleeg de volgende resources Azure Stack Hub meer informatie over de schaalbaarheid van uw bedrijf:

- [Capaciteitsplanning voor Azure Stack Hub overzicht](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Extra schaaleenheidknooppunten toevoegen in Azure Stack Hub](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes-cluster (clusterniveau)**

Het Kubernetes-cluster zelf bestaat uit en is gebaseerd op Azure (Stack) IaaS-onderdelen, waaronder reken-, opslag- en netwerkbronnen. Kubernetes-oplossingen hebben betrekking op hoofd- en werkknooppunten, die worden geïmplementeerd als VM's in Azure (en Azure Stack Hub).

- [Besturingsvlakknooppunten](/azure/aks/concepts-clusters-workloads#control-plane) (master) bieden de belangrijkste Kubernetes-services en de orchestration van toepassingsworkloads.
- [Werkknooppunten](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (werkknooppunten) voeren de workloads van uw toepassing uit.

Bij het selecteren van VM-grootten voor de eerste implementatie zijn er verschillende overwegingen:  

- **Kosten:** houd bij het plannen van uw werkknooppunten rekening met de totale kosten per VM die u maakt. Als voor uw toepassingsworkloads bijvoorbeeld beperkte resources nodig zijn, moet u plannen om kleinere VM's te implementeren. Azure Stack Hub, zoals Azure, wordt normaal gesproken gefactureerd op basis van verbruik, dus het op de juiste wijze bepalen van de VM's voor Kubernetes-rollen is van cruciaal belang voor het optimaliseren van de verbruikskosten. 

- **Schaalbaarheid:** de schaalbaarheid van het cluster wordt bereikt door het aantal hoofd- en werkknooppunten in en uit te schalen of door extra knooppuntgroepen toe te voegen (momenteel Azure Stack Hub beschikbaar). Het schalen van het cluster kan worden uitgevoerd op basis van prestatiegegevens die worden verzameld met behulp van Container Insights (Azure Monitor + Log Analytics). 

    Als uw toepassing meer (of minder) resources nodig heeft, kunt u uw huidige knooppunten horizontaal uitschalen (tussen 1 en 50 knooppunten). Als u meer dan 50 knooppunten nodig hebt, kunt u een extra cluster maken in een afzonderlijk abonnement. U kunt de werkelijke VM's niet verticaal omhoog schalen naar een andere VM-grootte zonder het cluster opnieuw te moetenployeren.

    Schalen wordt handmatig uitgevoerd met behulp van de AKS Engine-helper-VM die in eerste instantie is gebruikt om het Kubernetes-cluster te implementeren. Zie [Kubernetes-clusters schalen voor meer informatie](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Quota:** houd rekening met [de quota](/azure-stack/operator/azure-stack-quota-types) die u hebt geconfigureerd bij het plannen van een AKS-implementatie op uw Azure Stack Hub. Zorg ervoor dat [voor elk abonnement](/azure-stack/operator/service-plan-offer-subscription-overview) de juiste plannen en quota zijn geconfigureerd. Het abonnement moet rekening houden met de hoeveelheid rekenkracht, opslag en andere services die nodig zijn voor uw clusters wanneer ze worden uitschalen.

- **Toepassingsworkloads:** raadpleeg de concepten Clusters en [workloads](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) in het kubernetes-basisconcepten voor Azure Kubernetes Service document. Dit artikel helpt u bij het bereik van de juiste VM-grootte op basis van de reken- en geheugenbehoeften van uw toepassing.  

**Toepassing (toepassingsniveau)**

Op de toepassingslaag gebruiken we Kubernetes [Horizontal Pod Autoscaler (HPA).](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) HPA kan het aantal replica's (pod/Container Instances) in onze implementatie verhogen of verlagen op basis van verschillende metrische gegevens (zoals CPU-gebruik).

Een andere optie is om container instances verticaal te schalen. Dit kan worden bereikt door de hoeveelheid CPU en het aangevraagde geheugen te wijzigen en beschikbaar te maken voor een specifieke implementatie. Zie [Resources voor containers beheren](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) op kubernetes.io voor meer informatie.

## <a name="networking-and-connectivity-considerations"></a>Aandachtspunten voor netwerken en connectiviteit

Netwerken en connectiviteit zijn ook van invloed op de drie lagen die eerder zijn vermeld voor Kubernetes op Azure Stack Hub. In de volgende tabel ziet u de lagen en de services die ze bevatten:

| Laag | Beïnvloedt | Wat? |
| --- | --- | ---
| Toepassing | Toepassing | Hoe is de toepassing toegankelijk? Wordt deze blootgesteld aan internet? |
| Cluster | Kubernetes-cluster | Kubernetes-API, AKS Engine-VM, containerafbeeldingen (egress) binnenhalen), bewakingsgegevens en telemetrie verzenden (egress) |
| Infrastructuur | Azure Stack Hub | Toegankelijkheid van Azure Stack Hub eindpunten voor beheer, zoals de portal en Azure Resource Manager eindpunten. |

**Toepassing**

Voor de toepassingslaag is de belangrijkste overweging of de toepassing beschikbaar is en toegankelijk is via internet. Vanuit het oogpunt van Kubernetes betekent internettoegankelijkheid dat een implementatie of pod wordt blootstellen met behulp van een Kubernetes-service of een toegangscontroller.

> [!NOTE]
> We raden het gebruik van toegangscontrollers aan om Kubernetes Services beschikbaar te maken, aangezien het aantal openbare FRONT-Azure Stack Hub's beperkt is tot 5. Dit ontwerp beperkt ook het aantal Kubernetes-services (met het type LoadBalancer) tot 5, wat te klein is voor veel implementaties. Ga naar de [documentatie van de AKS-engine](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) voor meer informatie.

Het beschikbaar maken van een toepassing met behulp van een openbaar IP-adres via een Load Balancer of een toegangscontroller betekent niet altijd dat de toepassing nu toegankelijk is via internet. Het is mogelijk dat Azure Stack Hub een openbaar IP-adres hebben dat alleen zichtbaar is op het lokale intranet. Niet alle openbare IP-adressen zijn echt internet-gericht.

In het vorige blok wordt het toegangsverkeer naar de toepassing in overweging nemen. Een ander onderwerp dat moet worden overwogen voor een geslaagde Kubernetes-implementatie is uitgaand/uitgaand verkeer. Hier zijn enkele gebruikssituaties waarvoor het uit te gaan verkeer is vereist:

- Container-afbeeldingen die zijn opgeslagen op DockerHub of Azure Container Registry
- Helm-grafieken ophalen
- Toepassingsgegevens Insights (of andere bewakingsgegevens)

Sommige bedrijfsomgevingen vereisen mogelijk het _gebruik_ van transparante of _niet-transparante_ proxyservers. Deze servers vereisen een specifieke configuratie op verschillende onderdelen van ons cluster. De documentatie van de AKS-engine bevat verschillende informatie over het geschikt maken van netwerk-proxies. Zie AKS Engine [and proxy servers (AKS-engine en proxyservers) voor meer informatie](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Ten laatste moet verkeer tussen verschillende clusters tussen de Azure Stack Hub stromen. De voorbeeldimplementatie bestaat uit afzonderlijke Kubernetes-clusters die worden uitgevoerd op afzonderlijke Azure Stack Hub exemplaren. Het verkeer ertussen, zoals het replicatieverkeer tussen twee databases, is 'extern verkeer'. Extern verkeer moet worden gerouteerd via een site-naar-site-VPN of Azure Stack Hub openbare IP-adressen om Kubernetes op twee Azure Stack Hub verbinden:

![inter- en intraclusterverkeer](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

Het Kubernetes-cluster hoeft niet per se toegankelijk te zijn via internet. Het relevante onderdeel is de Kubernetes-API die wordt gebruikt voor het gebruik van een cluster, bijvoorbeeld met behulp van `kubectl` . Het Kubernetes API-eindpunt moet toegankelijk zijn voor iedereen die het cluster beheert of toepassingen en services op het cluster implementeert. Dit onderwerp wordt uitgebreid besproken vanuit het perspectief van DevOps in de sectie Overwegingen voor implementatie [(CI/CD)](#deployment-cicd-considerations) hieronder.

Op clusterniveau zijn er ook enkele overwegingen met het oog op het verkeer op basis van het verkeer:

- Knooppuntupdates (voor Ubuntu)
- Bewakingsgegevens (verzonden naar Azure LogAnalytics)
- Andere agents waarvoor uitgaand verkeer is vereist (specifiek voor de omgeving van elke deployer)

Voordat u uw Kubernetes-cluster implementeert met behulp van de AKS-engine, moet u het uiteindelijke netwerkontwerp plannen. In plaats van een toegewezen Virtual Network maken, kan het efficiënter zijn om een cluster in een bestaand netwerk te implementeren. U kunt bijvoorbeeld gebruikmaken van een bestaande site-naar-site-VPN-verbinding die al is geconfigureerd in Azure Stack Hub omgeving.

**Infrastructuur**  

Infrastructuur verwijst naar toegang tot de Azure Stack Hub management-eindpunten. Eindpunten zijn de tenant- en beheerportals en de eindpunten Azure Resource Manager beheerder en tenant. Deze eindpunten zijn vereist voor het Azure Stack Hub en de kernservices.

## <a name="data-and-storage-considerations"></a>Overwegingen voor gegevens en opslag

Er worden twee exemplaren van onze toepassing geïmplementeerd, op twee afzonderlijke Kubernetes-clusters, op twee Azure Stack Hub exemplaren. Voor dit ontwerp moeten we nadenken over het repliceren en synchroniseren van gegevens ertussen.

Met Azure beschikken we over de ingebouwde mogelijkheid om opslag te repliceren in meerdere regio's en zones in de cloud. Op dit Azure Stack Hub zijn er geen native manieren om opslag te repliceren tussen twee verschillende Azure Stack Hub-exemplaren. Ze vormen twee onafhankelijke clouds zonder overkoepelende manier om ze als set te beheren. Bij het plannen van tolerantie voor toepassingen die in verschillende Azure Stack Hub, moet u rekening houden met deze onafhankelijkheid in het ontwerp en de implementatie van uw toepassing.

In de meeste gevallen is opslagreplicatie niet nodig voor een flexibele en zeer beschikbare toepassing die op AKS is geïmplementeerd. Maar u moet onafhankelijke opslag per Azure Stack Hub in uw toepassingsontwerp overwegen. Als dit ontwerp een probleem of een wegenblok is voor het implementeren van de oplossing op Azure Stack Hub, zijn er mogelijke oplossingen van Microsoft-partners die opslagbijlagen leveren. Storage bijlagen bieden een opslagreplicatieoplossing voor meerdere Azure Stack Hubs en Azure. Zie partneroplossingen voor [meer informatie.](#partner-solutions)

In onze architectuur werden deze lagen beschouwd als:

**Configuratie**

Configuratie omvat de configuratie van Azure Stack Hub, de AKS-engine en het Kubernetes-cluster zelf. De configuratie moet zoveel mogelijk worden geautomatiseerd en als Infrastructuur als code worden opgeslagen in een op Git gebaseerd versiebeheersysteem zoals Azure DevOps of GitHub. Deze instellingen kunnen niet eenvoudig worden gesynchroniseerd in meerdere implementaties. Daarom raden we u aan om de configuratie van buitenaf op te slaan en toe te passen en de DevOps-pijplijn te gebruiken.

**Toepassing**

De toepassing moet worden opgeslagen in een git-opslagplaats. Wanneer er een nieuwe implementatie, wijzigingen in de toepassing of herstel na noodherstel is, kan deze eenvoudig worden geïmplementeerd met behulp van Azure Pipelines.

**Gegevens**

Gegevens zijn de belangrijkste overweging bij de meeste toepassingsontwerpen. Toepassingsgegevens moeten synchroon blijven tussen de verschillende exemplaren van de toepassing. Voor gegevens is ook een back-up- en noodherstelstrategie nodig als er een storing is.

Het bereiken van dit ontwerp is sterk afhankelijk van technologische keuzen. Hier zijn enkele oplossingsvoorbeelden voor het implementeren van een database op een manier met hoge Azure Stack Hub:

- [Een beschikbaarheidsgroep SQL Server 2016 implementeren in Azure en Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Een MongoDB-oplossing met hoge beschikbaarheid implementeren in Azure en Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Overwegingen bij het werken met gegevens op meerdere locaties is een nog complexere overweging voor een zeer beschikbare en flexibele oplossing. Overweeg het volgende:

- Latentie en netwerkverbinding tussen Azure Stack Hubs.
- Beschikbaarheid van identiteiten voor services en machtigingen. Elk Azure Stack Hub kan worden geïntegreerd met een externe map. Tijdens de implementatie kiest u voor Azure Active Directory (Azure AD) of Active Directory Federation Services (ADFS). Daarom is het mogelijk om één identiteit te gebruiken die kan communiceren met meerdere onafhankelijke Azure Stack Hub exemplaren.

## <a name="business-continuity-and-disaster-recovery"></a>Bedrijfscontinuïteit en herstel na noodgevallen

Bedrijfscontinuïteit en herstel na noodherstel (BCDR) is een belangrijk onderwerp in Azure Stack Hub en Azure. Het belangrijkste verschil is dat in Azure Stack Hub operator het hele BCDR-proces moet beheren. In Azure worden onderdelen van BCDR automatisch beheerd door Microsoft.

BCDR is van invloed op dezelfde gebieden die worden vermeld in de vorige sectie [Overwegingen voor gegevens en opslag:](#data-and-storage-considerations)

- Infrastructuur/configuratie
- Beschikbaarheid van toepassingen
- Toepassingsgegevens

En zoals vermeld in de vorige sectie, zijn deze gebieden de verantwoordelijkheid van de Azure Stack Hub operator en kunnen deze per organisatie verschillen. Plan BCDR op basis van uw beschikbare hulpprogramma's en processen.

**Infrastructuur en configuratie**

In deze sectie worden de fysieke en logische infrastructuur en de configuratie van Azure Stack Hub. Het bevat acties in de beheer- en tenantruimten.

De Azure Stack Hub operator (of beheerder) is verantwoordelijk voor het onderhoud van de Azure Stack Hub exemplaren. Inclusief onderdelen zoals het netwerk, opslag, identiteit en andere onderwerpen die buiten het bereik van dit artikel vallen. Zie de volgende bronnen voor meer informatie over Azure Stack Hub bewerkingen:

- [Gegevens herstellen in Azure Stack Hub met de Infrastructure Backup Service](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Back-up voor Azure Stack Hub inschakelen vanuit de beheerdersportal](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Herstellen van onherstelbaar gegevensverlies](/azure-stack/operator/azure-stack-backup-recover-data)
- [Aanbevolen procedures voor de service Infrastructure Backup](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub is het platform en de fabric waarop Kubernetes-toepassingen worden geïmplementeerd. De eigenaar van de toepassing voor de Kubernetes-toepassing is een gebruiker van Azure Stack Hub, met toegang verleend voor het implementeren van de toepassingsinfrastructuur die nodig is voor de oplossing. Toepassingsinfrastructuur betekent in dit geval het Kubernetes-cluster, geïmplementeerd met behulp van de AKS-engine en de omliggende services. Deze onderdelen worden geïmplementeerd in Azure Stack Hub, beperkt door een Azure Stack Hub aanbieding. Zorg ervoor dat de aanbieding die door de eigenaar van de Kubernetes-toepassing wordt geaccepteerd, voldoende capaciteit heeft (uitgedrukt in Azure Stack Hub quota) om de hele oplossing te implementeren. Zoals aanbevolen in de vorige sectie, moet de implementatie van de toepassing worden geautomatiseerd met behulp van Infrastructure-as-Code en implementatiepijplijnen zoals Azure DevOps Azure Pipelines.

Zie overzicht van Azure Stack Hub, [abonnementen,](/azure-stack/operator/service-plan-offer-subscription-overview) aanbiedingen en abonnementen voor meer Azure Stack Hub Azure Stack Hub over aanbiedingen en quota

Het is belangrijk om de configuratie van de AKS-engine, inclusief de uitvoer, veilig op te slaan. Deze bestanden bevatten vertrouwelijke informatie die wordt gebruikt voor toegang tot het Kubernetes-cluster. Daarom moet het worden beschermd tegen blootstelling aan niet-beheerders.

**Beschikbaarheid van toepassingen**

De toepassing mag niet afhankelijk zijn van back-ups van een geïmplementeerd exemplaar. Standaard moet u de toepassing opnieuw toepassen volgens infrastructure-as-code-patronen. U kunt bijvoorbeeld azure DevOps Azure Pipelines opnieuw implementeren. De BCDR-procedure moet betrekking hebben op het opnieuw toepassen van de toepassing op hetzelfde of een ander Kubernetes-cluster.

**Toepassingsgegevens**

Toepassingsgegevens zijn het essentiële onderdeel om gegevensverlies te minimaliseren. In de vorige sectie zijn technieken beschreven voor het repliceren en synchroniseren van gegevens tussen twee (of meer) exemplaren van de toepassing. Afhankelijk van de database-infrastructuur (MySQL, MongoDB, MSSQL of andere) die wordt gebruikt voor het opslaan van de gegevens, zijn er verschillende beschikbaarheids- en back-uptechnieken beschikbaar waar u uit kunt kiezen.

De aanbevolen manieren om integriteit te bereiken, zijn:
- Een systeemeigen back-upoplossing voor de specifieke database.
- Een back-upoplossing die officieel ondersteuning biedt voor back-up en herstel van het databasetype dat door uw toepassing wordt gebruikt.

> [!IMPORTANT]
> Sla uw back-upgegevens niet op dezelfde Azure Stack Hub instantie waarin uw toepassingsgegevens zich bevinden. Een volledige storing van het Azure Stack Hub exemplaar zou ook uw back-ups in gevaar brengen.

## <a name="availability-considerations"></a>Beschikbaarheidsoverwegingen

Kubernetes op Azure Stack Hub geïmplementeerd via AKS Engine is geen beheerde service. Het is een geautomatiseerde implementatie en configuratie van een Kubernetes-cluster met behulp van Azure Infrastructure-as-a-Service (IaaS). Als zodanig biedt het dezelfde beschikbaarheid als de onderliggende infrastructuur.

Azure Stack Hub infrastructuur is al bestand tegen fouten en biedt mogelijkheden zoals beschikbaarheidssets om onderdelen te distribueren over meerdere fout- en [updatedomeinen.](/azure-stack/user/azure-stack-vm-considerations#high-availability) Maar de onderliggende technologie (failoverclustering) veroorzaakt nog steeds enige downtime voor VM's op een beïnvloede fysieke server, als er een hardwarestoring is.

Het is een goede gewoonte om uw Kubernetes-productiecluster en de workload te implementeren naar twee (of meer) clusters. Deze clusters moeten worden gehost op verschillende locaties of datacenters en gebruikmaken van technologieën zoals Azure Traffic Manager om gebruikers te routeer op basis van de reactietijd van het cluster of op basis van geografie.

![Gebruik Traffic Manager om verkeersstromen te beheren](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Klanten die één Kubernetes-cluster hebben, maken doorgaans verbinding met het SERVICE-IP-adres of de DNS-naam van een bepaalde toepassing. In een implementatie met meerdere clusters moeten klanten verbinding maken met een Traffic Manager DNS-naam die naar de services/ingress op elk Kubernetes-cluster wijst.

![Gebruik Traffic Manager om naar een on-premises cluster te gaan](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Dit patroon is ook een best practice [voor (beheerde) AKS-clusters in Azure.](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment)

Het Kubernetes-cluster zelf, geïmplementeerd via de AKS-engine, moet bestaan uit ten minste drie hoofdknooppunten en twee werkknooppunten.

## <a name="identity-and-security-considerations"></a>Identiteits- en beveiligingsoverwegingen

Identiteit en beveiliging zijn belangrijke onderwerpen. Met name wanneer de oplossing onafhankelijke Azure Stack Hub omvat. Kubernetes en Azure (inclusief Azure Stack Hub) hebben beide afzonderlijke mechanismen voor op rollen gebaseerd toegangsbeheer (RBAC):

- Azure RBAC beheert de toegang tot resources in Azure (en Azure Stack Hub), waaronder de mogelijkheid om nieuwe Azure-resources te maken. Machtigingen kunnen worden toegewezen aan gebruikers, groepen of service-principals. (Een service-principal is een beveiligingsidentiteit die wordt gebruikt door toepassingen.)
- Kubernetes RBAC beheert machtigingen voor de Kubernetes-API. Het maken van pods en het aanbieden van pods zijn bijvoorbeeld acties die aan een gebruiker kunnen worden geautoriseerd (of geweigerd) via RBAC. Als u Kubernetes-machtigingen wilt toewijzen aan gebruikers, maakt u rollen en rolbindingen.

**Azure Stack Hub identiteit en RBAC**

Azure Stack Hub biedt twee opties voor id-provider. De provider die u gebruikt, is afhankelijk van de omgeving en of deze wordt uitgevoerd in een verbonden of niet-verbonden omgeving:

- Azure AD: kan alleen worden gebruikt in een verbonden omgeving.
- ADFS naar een traditioneel Active Directory-forest: kan worden gebruikt in een verbonden of niet-verbonden omgeving.

De id-provider beheert gebruikers en groepen, inclusief verificatie en autorisatie voor toegang tot resources. Toegang kan worden verleend aan Azure Stack Hub resources zoals abonnementen, resourcegroepen en afzonderlijke resources, zoals VM's of load balancers. Als u een consistent toegangsmodel wilt hebben, kunt u overwegen om dezelfde groepen (direct of genest) te gebruiken voor alle Azure Stack Hubs. Hier is een configuratievoorbeeld:

![geneste aad-groepen met azure stack hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

Het voorbeeld bevat een toegewezen groep (met AAD of ADFS) voor een specifiek doel. Als u bijvoorbeeld inzendermachtigingen wilt bieden voor de resourcegroep die onze Kubernetes-clusterinfrastructuur op een specifiek Azure Stack Hub-exemplaar bevat (hier 'Inzender voor Seattle K8s-cluster'). Deze groepen worden vervolgens genest in een algemene groep die de 'subgroepen' voor elke Azure Stack Hub.

Onze voorbeeldgebruiker heeft nu 'Inzender'-machtigingen voor beide resourcesgroepen die de volledige set Kubernetes-infrastructuurbronnen bevatten. De gebruiker heeft toegang tot resources op beide Azure Stack Hub exemplaren, omdat de exemplaren dezelfde id-provider delen.

> [!IMPORTANT]
> Deze machtigingen zijn alleen van Azure Stack Hub en sommige resources die erboven zijn geïmplementeerd. Een gebruiker met dit toegangsniveau kan veel schade aanrichten, maar heeft geen toegang tot de Kubernetes IaaS-VM's en de Kubernetes-API zonder extra toegang tot de Kubernetes-implementatie.

**Kubernetes-identiteit en RBAC**

Een Kubernetes-cluster gebruikt standaard niet dezelfde id-provider als de Azure Stack Hub. De VM's die het Kubernetes-cluster, de hoofdknooppunten en werkknooppunten hosten, gebruiken de SSH-sleutel die is opgegeven tijdens de implementatie van het cluster. Deze SSH-sleutel is vereist om via SSH verbinding te maken met deze knooppunten.

De Kubernetes-API (bijvoorbeeld toegankelijk via ) wordt ook beveiligd door serviceaccounts, waaronder een `kubectl` standaardserviceaccount voor clusterbeheerder. De referenties voor dit serviceaccount worden in eerste instantie opgeslagen in het `.kube/config` bestand op uw Kubernetes-hoofdknooppunten.

**Geheimenbeheer en toepassingsreferenties**

Voor het opslaan van geheimen, zoals verbindingsreeksen of databasereferenties, zijn er verschillende opties, waaronder:

- Azure Key Vault
- Kubernetes Secrets
- Oplossingen van derden, zoals HashiCorp Vault (uitgevoerd op Kubernetes)

Sla geheimen of referenties niet op in niet-gecodeerde tekst in uw configuratiebestanden, toepassingscode of in scripts. En sla ze niet op in een versiebeheersysteem. In plaats daarvan moet de implementatieautomatisering de geheimen naar behoefte ophalen.

## <a name="patch-and-update"></a>Patch en update

Het **patch- en updateproces (PNU)** in Azure Kubernetes Service is gedeeltelijk geautomatiseerd. Kubernetes-versie-upgrades worden handmatig geactiveerd, terwijl beveiligingsupdates automatisch worden toegepast. Deze updates kunnen beveiligingsfixes voor het besturingssysteem of kernelupdates bevatten. AKS start deze Linux-knooppunten niet automatisch opnieuw op om het updateproces te voltooien. 

Het PNU-proces voor een Kubernetes-cluster dat is geïmplementeerd met behulp van de AKS-engine op Azure Stack Hub, is niet-wordend en is de verantwoordelijkheid van de clusteroperator. 

De AKS-engine helpt bij de twee belangrijkste taken:

- [Upgraden naar een nieuwere Kubernetes- en basisversie van de besturingssysteemafbeelding](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Alleen de basis-os-afbeelding upgraden](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Nieuwere basisbesturingssysteemafbeeldingen bevatten de nieuwste beveiligingsfixes voor het besturingssysteem en kernelupdates. 

Het [mechanisme voor upgrade](https://wiki.debian.org/UnattendedUpgrades) zonder toezicht installeert automatisch beveiligingsupdates die worden uitgebracht voordat een nieuwe versie van de basisbesturingssysteemafbeelding beschikbaar is in Azure Stack Hub Marketplace. Upgrade zonder toezicht is standaard ingeschakeld en installeert beveiligingsupdates automatisch, maar de Kubernetes-clusterknooppunten worden niet opnieuw opgestart. Het opnieuw opstarten van de knooppunten kan worden geautomatiseerd met behulp van de open-source [ **K** Ubernetes **RE** boot **D** aemon (maanded))](/azure/aks/node-updates-kured). Er wordt ge kijken naar Linux-knooppunten waarvoor opnieuw moet worden opgestart. Vervolgens wordt het opnieuw plannen van het opnieuw opstarten van pods en het opnieuw opstarten van knooppunten automatisch verwerkt.

## <a name="deployment-cicd-considerations"></a>Overwegingen voor implementatie (CI/CD)

Azure en Azure Stack Hub dezelfde Azure Resource Manager-API's beschikbaar maken. Deze API's worden net als elke andere Azure-cloud (Azure, Azure China 21Vianet, Azure Government) Azure Government). Er kunnen verschillen zijn in API-versies tussen clouds en Azure Stack Hub biedt slechts een subset van services. De beheer-eindpunt-URI is ook verschillend voor elke cloud en elk exemplaar van Azure Stack Hub.

Afgezien van de subtiele verschillen die worden vermeld, Azure Resource Manager REST API's een consistente manier om te communiceren met Zowel Azure als Azure Stack Hub. Dezelfde set hulpprogramma's kan hier worden gebruikt als bij elke andere Azure-cloud. U kunt Azure DevOps, hulpprogramma's zoals Jenkins of PowerShell, gebruiken om services te implementeren en te Azure Stack Hub.

**Overwegingen**

Een van de belangrijkste verschillen op het gebied van Azure Stack Hub implementaties is de vraag over toegankelijkheid van internet. Toegankelijkheid via internet bepaalt of u een door Microsoft gehoste of zelf-hostende buildagent selecteert voor uw CI/CD-taken.

Een zelf-hostende agent kan worden uitgevoerd boven op Azure Stack Hub (als een IaaS-VM) of in een netwerksubnet dat toegang heeft tot Azure Stack Hub. Ga naar [Azure Pipelines-agents](/azure/devops/pipelines/agents/agents) voor meer informatie over de verschillen.

De volgende afbeelding helpt u om te bepalen of u een zelf-hostend of door Microsoft gehoste buildagent nodig hebt:

![Zelf-hostende buildagents Ja of Nee](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Zijn de Azure Stack Hub toegankelijk via internet?
  - Ja: We kunnen Azure Pipelines gebruiken met door Microsoft gehoste agents om verbinding te maken met Azure Stack Hub.
  - Nee: We hebben zelf-hostende agents nodig die verbinding kunnen maken met Azure Stack Hub van uw organisatie.
- Is ons Kubernetes-cluster toegankelijk via internet?
  - Ja: We kunnen Azure Pipelines gebruiken met door Microsoft gehoste agents om rechtstreeks te communiceren met het Kubernetes API-eindpunt.
  - Nee: we hebben zelf-hostende agents nodig die verbinding kunnen maken met het API-eindpunt van het Kubernetes-cluster.

In scenario's waarin Azure Stack Hub en Kubernetes-API toegankelijk zijn via internet, kan de implementatie gebruikmaken van een door Microsoft gehoste agent. Deze implementatie resulteert in een toepassingsarchitectuur als volgt:

[![Overzicht van openbare architectuur](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Als de Azure Resource Manager-eindpunten, Kubernetes-API of beide niet rechtstreeks toegankelijk zijn via internet, kunnen we gebruikmaken van een zelf-hostende buildagent om de pijplijnstappen uit te voeren. Dit ontwerp heeft minder connectiviteit nodig en kan alleen worden geïmplementeerd met on-premises netwerkconnectiviteit met Azure Resource Manager eindpunten en de Kubernetes-API:

[![Overzicht van on-prem-architectuur](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Hoe zit het met niet-verbonden scenario's?** In scenario's waarin Azure Stack Hub kubernetes of beide geen beheerinternetverbinding hebben, is het nog steeds mogelijk om Azure DevOps te gebruiken voor uw implementaties. U kunt een zelf-hostende agentgroep gebruiken (een DevOps-agent die on-premises of op Azure Stack Hub zelf wordt uitgevoerd) of een volledig zelf-hostende Azure DevOps Server on-premises. De zelf-hostende agent heeft alleen uitgaande HTTPS-internetverbinding (TCP/443) nodig.

Het patroon kan een Kubernetes-cluster (geïmplementeerd en gegroepeerd met AKS-engine) op elk Azure Stack Hub gebruiken. Het bevat een toepassing die bestaat uit een front-end, een mid-tier, back-end-services (bijvoorbeeld MongoDB) en een op nginx gebaseerde ingress Controller. In plaats van een database te gebruiken die wordt gehost op het K8s-cluster, kunt u gebruikmaken van externe gegevensopslag. Databaseopties zijn MySQL, SQL Server of een soort database die wordt gehost buiten Azure Stack Hub of in IaaS. Configuraties zoals deze vallen hier niet binnen het bereik.

## <a name="partner-solutions"></a>Partneroplossingen

Er zijn Microsoft Partner-oplossingen waarmee u de mogelijkheden van uw Azure Stack Hub. Deze oplossingen zijn nuttig gevonden bij implementaties van toepassingen die worden uitgevoerd op Kubernetes-clusters.  

## <a name="storage-and-data-solutions"></a>Storage en gegevensoplossingen

Zoals beschreven in [Overwegingen voor](#data-and-storage-considerations)gegevens en opslag, Azure Stack Hub momenteel geen native oplossing voor het repliceren van opslag in meerdere exemplaren. In tegenstelling tot Azure bestaat de mogelijkheid van het repliceren van opslag in meerdere regio's niet. In Azure Stack Hub is elk exemplaar een eigen afzonderlijke cloud. Er zijn echter oplossingen beschikbaar van Microsoft-partners die opslagreplicatie mogelijk maken in Azure Stack Hubs en Azure. 

**SCHAALBAARHEID**

[Schaalbaarheid](https://www.scality.com/) biedt opslag op webschaal die sinds 2009 digitale bedrijven mogelijk heeft gemaakt. De Scality RING, onze software-gedefinieerde opslag, zet x86-basisservers om in een onbeperkte opslaggroep voor elk type gegevens (bestand en object) op petabytes aan gegevens.

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) vereenvoudigt bedrijfsopslag met onbeperkte schaalbare opslag die enorme gegevenssets samen consolideert in één, eenvoudig te beheerde omgeving.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over concepten die in dit artikel zijn geïntroduceerd:

- [Schalen in de cloud en](pattern-cross-cloud-scale.md) geografisch [gedistribueerde app-patronen](pattern-geo-distributed.md) in Azure Stack Hub.
- [Microservicesarchitectuur op Azure Kubernetes Service (AKS)](/azure/architecture/reference-architectures/microservices/aks).

Wanneer u klaar bent om het voorbeeld van de oplossing te testen, gaat u verder met de implementatiehandleiding voor [Kubernetes-clusters](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)met hoge beschikbaarheid. De implementatiehandleiding bevat stapsgewijs instructies voor het implementeren en testen van de onderdelen.