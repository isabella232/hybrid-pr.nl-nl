---
title: Patroon voor schalen in de cloud (on-premises gegevens) in Azure Stack Hub
description: Meer informatie over het bouwen van een schaalbare cloud-app die gebruikmaakt van on-Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281241"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Patroon voor schalen in de cloud (on-premises gegevens)

Meer informatie over het bouwen van een hybride app die Azure en Azure Stack Hub. Dit patroon laat ook zien hoe u één on-premises gegevensbron gebruikt voor naleving.

## <a name="context-and-problem"></a>Context en probleem

Veel organisaties verzamelen en bewaren enorme hoeveelheden gevoelige klantgegevens. Vaak wordt voorkomen dat gevoelige gegevens in de openbare cloud worden opgeslagen vanwege bedrijfsvoorschriften of overheidsbeleid. Deze organisaties willen ook profiteren van de schaalbaarheid van de openbare cloud. De openbare cloud kan seizoensgebonden pieken in het verkeer verwerken, zodat klanten precies betalen voor de hardware die ze nodig hebben, wanneer ze deze nodig hebben.

## <a name="solution"></a>Oplossing

De oplossing profiteert van de nalevingsvoordelen van de privécloud, door deze te combineren met de schaalbaarheid van de openbare cloud. De Azure- en Azure Stack Hub hybride cloud bieden een consistente ervaring voor ontwikkelaars. Met deze consistentie kunnen ze hun vaardigheden toepassen op zowel openbare cloud- als on-premises omgevingen.

Met de implementatiehandleiding voor de oplossing kunt u een identieke web-app implementeren in een openbare en privécloud. U hebt ook toegang tot een routeerbaar netwerk dat niet via internet wordt gehost in de privécloud. De web-apps worden gecontroleerd op belasting. Na een aanzienlijke toename van het verkeer bewerkt een programma DNS-records om verkeer om te leiden naar de openbare cloud. Wanneer het verkeer niet langer belangrijk is, worden de DNS-records bijgewerkt om verkeer terug te leiden naar de privécloud.

[![Schalen in de cloud met on-prem gegevenspatroon](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Onderdelen

Deze oplossing maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/) kunt u web-apps, RESTful API-apps en Azure Functions. Alles in de programmeertaal van uw keuze, zonder infrastructuur te beheren. |
| | Azure Virtual Network| [Azure Virtual Network (VNet)](/azure/virtual-network/virtual-networks-overview) is de fundamentele bouwsteen voor particuliere netwerken in Azure. Met VNet kunnen meerdere Azure-resourcetypen, zoals virtuele machines (VM), veilig communiceren met elkaar, internet en on-premises netwerken. In de oplossing wordt ook het gebruik van extra netwerkonderdelen gedemonstreerd:<br>- app- en gatewaysubnetten.<br>- een lokale on-premises netwerkgateway.<br>- een virtuele netwerkgateway die fungeert als een site-naar-site-VPN-gatewayverbinding.<br>- een openbaar IP-adres.<br>- een punt-naar-site-VPN-verbinding.<br>- Azure DNS voor het hosten van DNS-domeinen en het bieden van naamresolutie. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) is een dns-gebaseerde load balancer. Hiermee kunt u de distributie van gebruikersverkeer voor service-eindpunten in verschillende datacenters bepalen. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) is een extensible Application Performance Management-service voor webontwikkelaars die apps op meerdere platforms bouwen en beheren.|
| | Azure Functions | [Azure Functions](/azure/azure-functions/) kunt u uw code uitvoeren in een serverloze omgeving zonder dat u eerst een VM moet maken of een web-app moet publiceren. |
| | Automatische schaalaanpassing in Azure | [Automatisch schalen](/azure/azure-monitor/platform/autoscale-overview) is een ingebouwde functie van Cloud Services, VM's en web-apps. Met de functie kunnen apps het beste presteren wanneer de vraag verandert. Apps passen zich aan voor pieken in het verkeer, en stellen u op de hoogte wanneer metrische gegevens naar behoefte worden gewijzigd en geschaald. |
| Azure Stack Hub | IaaS Compute | Azure Stack Hub kunt u hetzelfde app-model, de selfserviceportal en API's van Azure gebruiken. Azure Stack Hub IaaS biedt een breed scala aan opensource-technologieën voor consistente hybride cloudimplementaties. In het voorbeeld van de oplossing wordt Windows server-VM gebruikt om SQL Server te maken.|
| | Azure App Service | Net als de Azure-web-app gebruikt de oplossing Azure App Service [op Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) web-app te hosten. |
| | Netwerken | De Azure Stack Hub Virtual Network werkt precies zoals de Azure Virtual Network. Er worden veel van dezelfde netwerkonderdelen gebruikt, waaronder aangepaste hostnamen.
| Azure DevOps Services | Aanmelden | Stel snel continue integratie in voor bouwen, testen en implementeren. Zie Registreren en aanmelden bij [Azure DevOps voor meer informatie.](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops) |
| | Azure Pipelines | Gebruik [Azure Pipelines voor](/azure/devops/pipelines/agents/agents?view=azure-devops) continue integratie/continue levering. Met Azure Pipelines kunt u gehoste build- en releaseagents en -definities beheren. |
| | Codeopslagplaats | Maak gebruik van meerdere code-opslagplaatsen om uw ontwikkelingspijplijn te stroomlijnen. Gebruik bestaande code-opslagplaatsen in GitHub, Bitbucket, Dropbox, OneDrive en Azure-opslagplaatsen. |

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten bij het bepalen hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

Azure en Azure Stack Hub zijn uniek geschikt voor de behoeften van de huidige wereldwijde gedistribueerde onderneming.

#### <a name="hybrid-cloud-without-the-hassle"></a>Hybride cloud zonder problemen

Microsoft biedt een uitstekende integratie van on-premises assets met Azure Stack Hub en Azure in één geïntegreerde oplossing. Deze integratie elimineert het beheer van oplossingen met meerdere punten en een combinatie van cloudproviders. Met schalen in de cloud is de kracht van Azure slechts een paar klikken verwijderd. U kunt uw Azure Stack Hub verbinden met Azure met behulp van cloud-bursting en uw gegevens en apps zijn beschikbaar in Azure wanneer dat nodig is.

- U hoeft geen secundaire DR-site te bouwen en te onderhouden.
- Bespaar tijd en geld door tapeback-up te elimineren en maximaal 99 jaar aan back-upgegevens in Azure op te slaan.
- Migreert eenvoudig de werkbelastingen van Hyper-V, Physical (in preview) en VMware (in preview) naar Azure om gebruik te maken van de economie en elasticiteit van de cloud.
- Voer rekenintensieve rapporten of analyses uit op een gerepliceerde kopie van uw on-premises asset in Azure zonder dat dit invloed heeft op productieworkloads.
- Burst in de cloud en voer on-premises workloads uit in Azure, met grotere rekensjablonen wanneer dat nodig is. Hybride biedt u de kracht die u nodig hebt wanneer u deze nodig hebt.
- Maak ontwikkelomgevingen met meerdere lagen in Azure met een paar muisklikken. Repliceer zelfs live productiegegevens naar uw ontwikkel-/testomgeving om deze bijna in realtime te synchroniseren.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Economie van cloudoverschrijdende schaalbaarheid met Azure Stack Hub

Het belangrijkste voordeel van cloud-bursting is voordelige besparingen. U betaalt alleen voor de extra resources wanneer er vraag is naar deze resources. Geen uitgaven meer aan onnodige extra capaciteit of het voorspellen van pieken en schommelingen in de vraag.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Hoge vraag naar de cloud verminderen

Schalen in de cloud kan worden gebruikt om verwerkingsbelastingen te verwerken. De belasting wordt gedistribueerd door basis-apps te verplaatsen naar de openbare cloud, door lokale resources vrij te maken voor bedrijfskritieke apps. Een app kan worden toegepast op de privécloud en vervolgens alleen naar de openbare cloud bursten wanneer dat nodig is om aan de eisen te voldoen.

### <a name="availability"></a>Beschikbaarheid

Wereldwijde implementatie heeft zijn eigen uitdagingen, zoals variabele connectiviteit en verschillende overheidsvoorschriften per regio. Ontwikkelaars kunnen slechts één app ontwikkelen en deze vervolgens implementeren voor verschillende redenen met verschillende vereisten. Implementeer uw app in de openbare Cloud van Azure en implementeer vervolgens extra exemplaren of onderdelen lokaal. U kunt verkeer tussen alle exemplaren beheren met behulp van Azure.

### <a name="manageability"></a>Beheerbaarheid

#### <a name="a-single-consistent-development-approach"></a>Eén consistente ontwikkelingsbenadering

Met Azure Azure Stack Hub u een consistente set ontwikkelhulpprogramma's in de hele organisatie gebruiken. Deze consistentie maakt het eenvoudiger om een praktijk van continue integratie en continue ontwikkeling (CI/CD) te implementeren. Veel apps en services die zijn geïmplementeerd in Azure of Azure Stack Hub zijn uitwisselbaar en kunnen naadloos op beide locaties worden uitgevoerd.

Een hybride CI/CD-pijplijn kan u helpen:

- Start een nieuwe build op basis van code-commits naar uw codeopslagplaats.
- Implementeer uw nieuwe code automatisch in Azure voor gebruikersacceptatietests.
- Zodra uw code is getest, implementeert u automatisch Azure Stack Hub.

### <a name="a-single-consistent-identity-management-solution"></a>Eén consistente oplossing voor identiteitsbeheer

Azure Stack Hub werkt met Azure Active Directory (Azure AD) en Active Directory Federation Services (ADFS). Azure Stack Hub werkt met Azure AD in verbonden scenario's. Voor omgevingen die geen verbinding hebben, kunt u ADFS gebruiken als een niet-verbonden oplossing. Service-principals worden gebruikt om toegang te verlenen tot apps, zodat ze resources kunnen implementeren of configureren via Azure Resource Manager.

### <a name="security"></a>Beveiliging

#### <a name="ensure-compliance-and-data-sovereignty"></a>Naleving en gegevenssoevereiniteit garanderen

Azure Stack Hub kunt u dezelfde service uitvoeren in meerdere landen als wanneer u een openbare cloud gebruikt. Door dezelfde app in datacenters in elk land te implementeren, kan aan de vereisten voor gegevenssoevereiniteit worden voldaan. Deze mogelijkheid zorgt ervoor dat persoonsgegevens binnen de grenzen van elk land worden bewaard.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub - beveiligingsstatus

Er is geen beveiligingsstatus zonder een solide, continu onderhoudsproces. Daarom heeft Microsoft geïnvesteerd in een orchestration-engine die patches en updates naadloos in de hele infrastructuur past.

Dankzij partnerrelaties met Azure Stack Hub OEM-partners breidt Microsoft dezelfde beveiligingsstatus uit naar OEM-specifieke onderdelen, zoals de Hardware Lifecycle Host en de software die erop wordt uitgevoerd. Deze samenwerking zorgt Azure Stack Hub een uniforme, solide beveiligingsstatus heeft voor de hele infrastructuur. Klanten kunnen op hun beurt hun app-workloads bouwen en beveiligen.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Gebruik van service-principals via PowerShell, CLI en Azure Portal

Als u resourcetoegang wilt verlenen tot een script of app, stelt u een identiteit in voor uw app en verifieert u de app met zijn eigen referenties. Deze identiteit wordt ook wel een service-principal genoemd en biedt u het volgende:

- Wijs machtigingen toe aan de app-identiteit die anders zijn dan uw eigen machtigingen en die zijn beperkt tot exact de behoeften van de app.
- Een certificaat voor verificatie gebruiken bij het uitvoeren van een onbewaakt script.

Zie Een app-identiteit gebruiken voor toegang tot resources voor meer informatie over het maken van een service-principal en het gebruik van [een certificaat voor referenties.](/azure-stack/operator/azure-stack-create-service-principals)

## <a name="when-to-use-this-pattern"></a>Wanneer dit patroon gebruiken

- Mijn organisatie gebruikt een DevOps-benadering of heeft er een gepland voor de nabije toekomst.
- Ik wil CI/CD-procedures implementeren in mijn Azure Stack Hub implementatie en de openbare cloud.
- Ik wil de CI/CD-pijplijn consolideren in cloud- en on-premises omgevingen.
- Ik wil apps naadloos kunnen ontwikkelen met behulp van cloud- of on-premises services.
- Ik wil gebruikmaken van consistente vaardigheden voor ontwikkelaars in cloud- en on-premises apps.
- Ik gebruik Azure, maar ik heb ontwikkelaars die in een on-premises Azure Stack Hub cloud werken.
- Mijn on-premises apps ervaren pieken in de vraag tijdens seizoensgebonden, cyclische of onvoorspelbare schommelingen.
- Ik heb on-premises onderdelen en ik wil de cloud gebruiken om deze naadloos te schalen.
- Ik wil schaalbaarheid in de cloud, maar ik wil dat mijn app zo veel mogelijk on-premises wordt uitgevoerd.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over onderwerpen die in dit artikel zijn geïntroduceerd:

- Bekijk [Hoe u apps dynamisch kunt schalen tussen datacenters en de openbare cloud](https://www.youtube.com/watch?v=2lw8zOpJTn0) voor een overzicht van hoe dit patroon wordt gebruikt.
- Zie [Ontwerpoverwegingen voor hybride apps](overview-app-design-considerations.md) voor meer informatie over best practices en voor het beantwoorden van aanvullende vragen die u mogelijk hebt.
- Dit patroon maakt gebruik van Azure Stack productfamilie, waaronder Azure Stack Hub. Zie de [Azure Stack producten en oplossingen voor](/azure-stack) meer informatie over het hele portfolio met producten en oplossingen.

Wanneer u klaar bent om het voorbeeld van de oplossing te testen, gaat u verder met de implementatiehandleiding voor de oplossing voor schalen in de cloud [(on-premises gegevens).](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data) De implementatiehandleiding bevat stapsgewijs instructies voor het implementeren en testen van de onderdelen.