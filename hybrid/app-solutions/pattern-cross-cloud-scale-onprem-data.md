---
title: Patroon voor cross-Cloud schaling (on-premises gegevens) in Azure Stack hub
description: Meer informatie over het bouwen van een schaal bare cross-Cloud-app die gebruikmaakt van on-premises gegevens in Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: edbb608fbf8e5288f29572bfe4cca98ffb3cb8fc
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910426"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Patroon voor cross-Cloud schaling (on-premises gegevens)

Meer informatie over het bouwen van een hybride app die Azure en Azure Stack hub omspant. In dit patroon ziet u ook hoe u één on-premises gegevens bron kunt gebruiken om te voldoen aan het beleid.

## <a name="context-and-problem"></a>Context en probleem

Veel organisaties verzamelen en slaan enorme hoeveel heden gevoelige klant gegevens op. Het is vaak niet mogelijk om gevoelige gegevens in de open bare Cloud op te slaan vanwege bedrijfs voorschriften of overheids beleid. Deze organisaties willen ook profiteren van de schaal baarheid van de open bare Cloud. De open bare Cloud kan seizoen pieken in het verkeer afhandelen, zodat klanten precies kunnen betalen voor de hardware die ze nodig hebben.

## <a name="solution"></a>Oplossing

De oplossing maakt gebruik van de nalevings voordelen van de privécloud en combineert deze met de schaal baarheid van de open bare Cloud. De hybride Cloud Azure en Azure Stack hub bieden een consistente ervaring voor ontwikkel aars. Met deze consistentie kunnen ze hun vaardig heden Toep assen op open bare Cloud-en on-premises omgevingen.

Met de implementatie handleiding voor oplossingen kunt u een identieke Web-app implementeren in een open bare en een privécloud. U kunt ook toegang krijgen tot een niet-Internet routeerbaar netwerk dat wordt gehost op de privécloud. De web-apps worden gecontroleerd op belasting. Wanneer het verkeer aanzienlijk toeneemt, worden de DNS-records door een programma bewerkt om verkeer naar de open bare Cloud om te leiden. Wanneer het verkeer niet meer significant is, worden de DNS-records bijgewerkt om het verkeer terug te sturen naar de privécloud.

[![Meerdere Cloud schalen met on-premises gegevens patroon](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Onderdelen

Deze oplossing maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| Azure | Azure App Service | Met [Azure app service](/azure/app-service/) kunt u web-apps, rest API apps en Azure functions bouwen en hosten. Alle in de programmeer taal van uw keuze, zonder dat u de infra structuur hoeft te beheren. |
| | Azure Virtual Network| [Azure Virtual Network (VNet)](/azure/virtual-network/virtual-networks-overview) is de fundamentele bouw stenen voor particuliere netwerken in Azure. VNet maakt het mogelijk meerdere Azure-resource typen, zoals virtuele machines (VM), veilig te communiceren met elkaar, Internet en on-premises netwerken. De oplossing demonstreert ook het gebruik van extra netwerk onderdelen:<br>-subnetten voor apps en gateways.<br>-een lokale on-premises netwerk gateway.<br>-een virtuele netwerk gateway die fungeert als een site-naar-site-VPN-gateway verbinding.<br>-een openbaar IP-adres.<br>-een punt-naar-site-VPN-verbinding.<br>-Azure DNS voor het hosten van DNS-domeinen en het leveren van naam omzetting. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) is een op DNS gebaseerd verkeer Load Balancer. Hiermee kunt u de distributie van gebruikers verkeer voor service-eind punten in verschillende data centers beheren. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) is een uitbreid bare service voor het beheer van toepassings prestaties voor webontwikkelaars die apps op meerdere platforms bouwen en beheren.|
| | Azure Functions | Met [Azure functions](/azure/azure-functions/) kunt u uw code in een serverloze omgeving uitvoeren zonder dat u eerst een virtuele machine hoeft te maken of een web-app hoeft te publiceren. |
| | Automatische schaalaanpassing in Azure | [Automatisch schalen](/azure/azure-monitor/platform/autoscale-overview) is een ingebouwde functie van Cloud Services, vm's en web-apps. Met de functie kunnen apps de beste prestaties uitvoeren wanneer de vraag verandert. Apps worden aangepast aan pieken in het verkeer, waarna u op de hoogte wordt gebracht wanneer metrische gegevens naar behoefte worden gewijzigd en geschaald. |
| Azure Stack hub | IaaS compute | Met Azure Stack hub kunt u hetzelfde app-model, Self-Service Portal en Api's gebruiken die worden ingeschakeld door Azure. Azure Stack hub IaaS biedt een breed scala aan open-source technologieën voor consistente hybride Cloud implementaties. In het voor beeld van de oplossing wordt een Windows Server-VM gebruikt om bijvoorbeeld te SQL Server.|
| | Azure App Service | Net als de Azure-web-app gebruikt de oplossing [Azure app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-overview) om de web-app te hosten. |
| | Netwerken | De Azure Stack hub Virtual Network werkt precies hetzelfde als de Azure Virtual Network. Het maakt gebruik van veel van dezelfde netwerk onderdelen, waaronder aangepaste hostnamen.
| Azure DevOps Services | Aanmelden | Stel snel continue integratie in voor bouwen, testen en implementatie. Zie [Aanmelden bij Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops)voor meer informatie. |
| | Azure-pijplijnen | Gebruik [Azure-pijp lijnen](/azure/devops/pipelines/agents/agents?view=azure-devops) voor continue integratie/continue levering. Met Azure-pijp lijnen kunt u gehoste build-en release-agents en definities beheren. |
| | Codeopslagplaats | Maak gebruik van meerdere code opslagplaatsen om uw ontwikkel pijplijn te stroom lijnen. Gebruik bestaande code opslagplaatsen in GitHub, bitbucket, Dropbox, OneDrive en Azure opslag plaatsen. |

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten wanneer u bepaalt hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

Azure en Azure Stack hub zijn uniek voor de behoeften van de wereld wijd gedistribueerde bedrijfs activiteiten.

#### <a name="hybrid-cloud-without-the-hassle"></a>Hybride Cloud zonder de gedoe

Micro soft biedt een ongeëvenaarde integratie van on-premises assets met Azure Stack hub en Azure in één geïntegreerde oplossing. Deze integratie elimineert de moeite van het beheren van meerdere punt oplossingen en een combi natie van cloud providers. Met cross-Cloud schalen is de kracht van Azure slechts een paar klikken weg. Verbind uw Azure Stack hub met Azure met Cloud bursting en uw gegevens en apps worden zo nodig in azure beschikbaar.

- Voorkom de nood zaak om een secundaire DR-site te bouwen en te onderhouden.
- Bespaar tijd en geld door tape back-ups en House tot 99 jaar aan back-upgegevens te elimineren in Azure.
- U kunt eenvoudig met de werk belasting van Hyper-V, fysieke (in Preview) en VMware (in Preview) naar Azure migreren om de economie en de elasticiteit van de cloud te benutten.
- Voer computerintensieve rapporten of analyses uit op een gerepliceerde kopie van uw on-premises asset in azure zonder dat dit van invloed is op de productiewerk belastingen.
- Breek in de Cloud en voer on-premises workloads uit in azure, met grotere reken sjablonen wanneer dat nodig is. Hybride biedt u de kracht die u nodig hebt, wanneer u deze nodig hebt.
- Maak met enkele klikken ontwikkel omgevingen met meerdere lagen in Azure. u kunt zelfs live productie gegevens repliceren naar uw ontwikkel-en test omgeving om deze bijna in realtime synchroniseren te houden.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Economie van cross-Cloud schalen met Azure Stack hub

Het belangrijkste voor deel van het gebruik van Cloud-burstisatie is een voordelige besparing. U betaalt alleen voor de extra resources wanneer er een aanvraag voor deze resources is. U hoeft niet meer te best Eden aan overbodige extra capaciteit of om te voors pellen vraag pieken en fluctuaties.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Verminder de belasting van hoge vraag in de Cloud

Het schalen van meerdere clouds kan worden gebruikt om verwerkings last te verg Roten. De belasting wordt gedistribueerd door basis-apps te verplaatsen naar de open bare Cloud, zodat lokale bronnen worden vrijgemaakt voor bedrijfskritische apps. Een app kan worden toegepast op de privécloud en vervolgens alleen worden gebursteerd naar de open bare Cloud als dat nodig is om aan de vraag te voldoen.

### <a name="availability"></a>Beschikbaarheid

Wereld wijde implementatie heeft zijn eigen uitdagingen, zoals de connectiviteit van variabelen en verschillende overheids voorschriften per regio. Ontwikkel aars kunnen slechts één app ontwikkelen en deze vervolgens over verschillende redenen implementeren met verschillende vereisten. Implementeer uw app in de open bare Azure-Cloud en implementeer extra exemplaren of onderdelen lokaal. U kunt verkeer tussen alle exemplaren beheren met Azure.

### <a name="manageability"></a>Beheerbaarheid

#### <a name="a-single-consistent-development-approach"></a>Een enkelvoudige, consistente ontwikkelings benadering

Met Azure en Azure Stack hub kunt u een consistente set ontwikkel tools voor de hele organisatie gebruiken. Deze consistentie maakt het gemakkelijker om een praktijk van continue integratie en doorlopende ontwikkeling (CI/CD) te implementeren. Veel apps en services die zijn geïmplementeerd in azure of Azure Stack hub zijn uitwisselbaar en kunnen op beide locaties naadloos worden uitgevoerd.

Een hybride CI/CD-pijp lijn kan u helpen:

- Start een nieuwe build op basis van code doorvoer voor uw code opslagplaats.
- Implementeer automatisch uw zojuist opgebouwde code naar Azure voor acceptatie tests door gebruikers.
- Zodra uw code is getest, kunt u automatisch implementeren naar Azure Stack hub.

### <a name="a-single-consistent-identity-management-solution"></a>Een enkelvoudige, consistente oplossing voor identiteits beheer

Azure Stack hub werkt met zowel Azure Active Directory (Azure AD) als Active Directory Federation Services (ADFS). Azure Stack hub werkt met Azure AD in verbonden scenario's. Voor omgevingen die geen verbinding hebben, kunt u ADFS gebruiken als een niet-verbonden oplossing. Service-principals worden gebruikt om toegang te verlenen aan apps, zodat ze bronnen kunnen implementeren of configureren via Azure Resource Manager.

### <a name="security"></a>Beveiliging

#### <a name="ensure-compliance-and-data-sovereignty"></a>Naleving garanderen en gegevens soevereiniteit

Met Azure Stack hub kunt u dezelfde service uitvoeren in meerdere landen als bij het gebruik van een open bare Cloud. Wanneer u dezelfde app in data centers in elk land implementeert, kunnen aan data soevereiniteit-vereisten worden voldaan. Deze functionaliteit zorgt ervoor dat persoons gegevens binnen de grenzen van elk land worden bewaard.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack hub-beveiligings postuur

Er is geen beveiligings postuur zonder een ononderbroken, doorlopend onderhouds proces. Daarom heeft micro soft geïnvesteerd in een Orchestration-engine die patches en updates naadloos toepast op de hele infra structuur.

Dankzij partnerschappen met Azure Stack hub OEM-partners, breidt micro soft dezelfde beveiligings postuur uit voor OEM-specifieke onderdelen, zoals de hardware Lifecycle host en de software die erop wordt uitgevoerd. Deze samen werking garandeert dat Azure Stack hub een uniforme, solide beveiligings postuur in de gehele infra structuur heeft. Klanten kunnen hun app-workloads op zijn beurt bouwen en beveiligen.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Het gebruik van service-principals via Power shell, CLI en Azure Portal

Als u toegang tot een script of app wilt verlenen aan de resource, stelt u een identiteit in voor uw app en verifieert u de app met zijn eigen referenties. Deze identiteit wordt een Service-Principal genoemd en biedt de volgende mogelijkheden:

- Wijs machtigingen toe aan de app-identiteit die afwijkt van uw eigen machtigingen en beperkt de behoeften van de app nauw keurig.
- Een certificaat voor verificatie gebruiken bij het uitvoeren van een onbewaakt script.

Zie voor meer informatie over het maken van een Service-Principal en het gebruik van een certificaat voor referenties, [een app-identiteit gebruiken om toegang te krijgen tot resources](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>Wanneer dit patroon gebruiken

- Mijn organisatie gebruikt een DevOps-benadering of heeft één gepland voor de nabije toekomst.
- Ik wil CI/CD-prak tijken implementeren in mijn Azure Stack hub-implementatie en de open bare Cloud.
- Ik wil de CI/CD-pijp lijn in de Cloud-en on-premises omgevingen consolideren.
- Ik wil de mogelijkheid om apps naadloos te ontwikkelen met behulp van Cloud-of on-premises Services.
- Ik wil een consistente ontwikkel vaardigheid gebruiken in zowel Cloud-als on-premises apps.
- Ik gebruik Azure, maar ik heb ontwikkel aars die werken in een on-premises Azure Stack hub-Cloud.
- Mijn on-premises apps hebben pieken in de vraag tijdens seizoen, cyclische of onvoorspelbare schommelingen.
- Ik heb on-premises onderdelen en ik wil de Cloud gebruiken om ze naadloos te schalen.
- Ik wil de schaal baarheid van de Cloud, maar ik wil dat mijn app zo veel mogelijk on-premises wordt uitgevoerd.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:

- Bekijk het [dynamisch schalen van apps tussen data centers en open bare Cloud](https://www.youtube.com/watch?v=2lw8zOpJTn0) voor een overzicht van hoe dit patroon wordt gebruikt.
- Zie [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van apps voor meer informatie over aanbevolen procedures en voor het beantwoorden van aanvullende vragen die u mogelijk hebt.
- Dit patroon maakt gebruik van de Azure Stack product familie, waaronder Azure Stack hub. Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [implementatie handleiding voor de oplossing voor cross-Cloud schaling (on-premises gegevens)](solution-deployment-guide-cross-cloud-scaling-onprem-data.md). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.
