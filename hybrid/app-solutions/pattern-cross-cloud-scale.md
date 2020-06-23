---
title: Patroon voor meerdere Cloud schalen in Azure Stack hub
description: Meer informatie over het bouwen van een schaal bare, cross-Cloud-app in Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910427"
---
# <a name="cross-cloud-scaling-pattern"></a>Patroon voor meerdere Cloud schalen

Voeg automatisch resources toe aan een bestaande app om een grotere belasting te kunnen bieden.

## <a name="context-and-problem"></a>Context en probleem

Uw app kan geen capaciteit verg Roten om te voldoen aan onverwachte toename van de vraag. Dit heeft geen gevolgen voor de schaal baarheid van gebruikers die de app niet bereiken tijdens piek gebruik. De app kan een vast aantal gebruikers onderhouden.

Wereld wijde ondernemingen vereisen veilige, betrouw bare en beschik bare Cloud-apps. Verg root de vraag en maakt gebruik van de juiste infra structuur ter ondersteuning van de vraag. Bedrijven kunnen moeite besparen met kosten en onderhoud met de beveiliging van Bedrijfs gegevens, opslag en real-time Beschik baarheid.

Mogelijk kunt u uw app niet uitvoeren in de open bare Cloud. Het is echter mogelijk niet economisch haalbaar voor het bedrijf om de capaciteit te behouden die vereist is in hun on-premises omgeving voor het afhandelen van pieken in de vraag naar de app. Met dit patroon kunt u gebruikmaken van de elasticiteit van de open bare Cloud met uw on-premises oplossing.

## <a name="solution"></a>Oplossing

Met het patroon voor meerdere Cloud schalen wordt een app uitgebreid die zich in een lokale Cloud bevindt met open bare cloud resources. Het patroon wordt geactiveerd door een verhoging of afname in de vraag en voegt respectievelijk resources toe aan of verwijderd uit de Cloud. Deze bronnen bieden redundantie, snelle Beschik baarheid en geo-compatibele route ring.

![Patroon voor meerdere Cloud schalen](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Dit patroon is alleen van toepassing op stateless onderdelen van uw app.

## <a name="components"></a>Onderdelen

Het patroon voor meerdere Cloud schalen bestaat uit de volgende onderdelen.

### <a name="outside-the-cloud"></a>Buiten de Cloud

#### <a name="traffic-manager"></a>Traffic Manager

Dit bevindt zich buiten de open bare Cloud groep in het diagram, maar zou wel het verkeer in het lokale Data Center en de open bare Cloud moeten kunnen coördineren. De Balancer levert een hoge Beschik baarheid voor de app door eind punten te bewaken en zo nodig failover te distribueren.

#### <a name="domain-name-system-dns"></a>Domain Name System (DNS)

De Domain Name System, of DNS, is verantwoordelijk voor het vertalen van een website-of service naam naar het IP-adres.

### <a name="cloud"></a>Cloud

#### <a name="hosted-build-server"></a>Gehoste build server

Een omgeving voor het hosten van uw build-pijp lijn.

#### <a name="app-resources"></a>App-resources

De app-resources moeten kunnen worden geschaald en uitgeschaald, zoals virtuele-machine schaal sets en containers.

#### <a name="custom-domain-name"></a>Aangepaste domein naam

Gebruik een aangepaste domein naam voor routerings aanvragen Globs.

#### <a name="public-ip-addresses"></a>Openbare IP-adressen

Open bare IP-adressen worden gebruikt voor het routeren van binnenkomend verkeer via Traffic Manager naar het eind punt van de open bare Cloud app-resources.  

### <a name="local-cloud"></a>Lokale Cloud

#### <a name="hosted-build-server"></a>Gehoste build server

Een omgeving voor het hosten van uw build-pijp lijn.

#### <a name="app-resources"></a>App-resources

De app-resources moeten kunnen worden geschaald en uitgeschaald, zoals virtuele-machine schaal sets en containers.

#### <a name="custom-domain-name"></a>Aangepaste domein naam

Gebruik een aangepaste domein naam voor routerings aanvragen Globs.

#### <a name="public-ip-addresses"></a>Openbare IP-adressen

Open bare IP-adressen worden gebruikt voor het routeren van binnenkomend verkeer via Traffic Manager naar het eind punt van de open bare Cloud app-resources.

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Beschouw de volgende punten als u besluit hoe u dit patroon wilt implementeren:

### <a name="scalability"></a>Schaalbaarheid

Het belangrijkste onderdeel van cross-Cloud schalen is de mogelijkheid om op aanvraag schalen te leveren. Er moet worden geschaald tussen de open bare en lokale Cloud infrastructuur en een consistente, betrouw bare service per vraag bieden.

### <a name="availability"></a>Beschikbaarheid

Zorg ervoor dat lokaal geïmplementeerde apps zijn geconfigureerd voor hoge Beschik baarheid via on-premises hardwareconfiguratie en software-implementatie.

### <a name="manageability"></a>Beheerbaarheid

Het cross-Cloud-patroon zorgt voor naadloos beheer en de vertrouwde interface tussen omgevingen.

## <a name="when-to-use-this-pattern"></a>Wanneer dit patroon gebruiken

U gebruikt dit patroon voor het volgende:

- Wanneer u uw app-capaciteit moet verhogen met onverwachte vraag of periodieke vraag naar vraag.
- Wanneer u niet wilt investeren in resources die alleen tijdens pieken worden gebruikt. Betaal voor wat u gebruikt.

Dit patroon wordt niet aanbevolen wanneer:

- Voor uw oplossing moeten gebruikers verbinding maken via internet.
- Uw bedrijf heeft lokale voor Schriften die vereisen dat de oorspronkelijke verbinding afkomstig is van een on-site oproep.
- Uw netwerk heeft regel matig knel punten die de prestaties van de schaal baarheid zouden beperken.
- Uw omgeving is niet verbonden met internet en de open bare Cloud kan niet worden bereikt.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:

- Zie het [overzicht van Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) voor meer informatie over hoe dit op DNS gebaseerd verkeer Load Balancer werkt.
- Zie [overwegingen voor het ontwerpen van hybride toepassingen](overview-app-design-considerations.md) voor meer informatie over aanbevolen procedures en voor het verkrijgen van antwoorden voor eventuele aanvullende vragen.
- Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de implementatie handleiding voor het implementeren van de [Cross-Cloud-schaal oplossing](solution-deployment-guide-cross-cloud-scaling.md). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen. U leert hoe u een cross-Cloud oplossing kunt maken om een hand matig geactiveerd proces te bieden voor het overschakelen van een door Azure Stack hub gehoste web-app naar een door Azure gehoste web-app. U leert ook hoe u automatisch schalen kunt gebruiken via Traffic Manager, waardoor flexibele en schaal bare Cloud hulpprogramma's worden gebruikt wanneer de belasting wordt geladen.
