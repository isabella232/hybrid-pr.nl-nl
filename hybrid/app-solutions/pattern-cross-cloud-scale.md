---
title: Patroon voor schalen in de cloud in Azure Stack Hub
description: Meer informatie over het bouwen van een schaalbare cloud-app in Azure en Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 90e0c177b5eaee4d223b4613e0b2ddf385fa799c
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281258"
---
# <a name="cross-cloud-scaling-pattern"></a>Patroon voor schalen in de cloud

Automatisch resources toevoegen aan een bestaande app om een toename in de belasting aan te kunnen.

## <a name="context-and-problem"></a>Context en probleem

Uw app kan de capaciteit niet verhogen om te voldoen aan onverwachte toename van de vraag. Door dit gebrek aan schaalbaarheid kunnen gebruikers de app tijdens piekuren niet bereiken. De app kan een vast aantal gebruikers leveren.

Wereldwijde ondernemingen hebben veilige, betrouwbare en beschikbare cloud-apps nodig. Het voldoen aan de toenemende vraag en het gebruik van de juiste infrastructuur om die vraag te ondersteunen, is essentieel. Bedrijven hebben moeite om kosten en onderhoud in balans te brengen met beveiliging, opslag en realtime beschikbaarheid van bedrijfsgegevens.

Mogelijk kunt u uw app niet uitvoeren in de openbare cloud. Het is echter misschien niet haalbaar dat het bedrijf de vereiste capaciteit in hun on-premises omgeving behoudt om pieken in de vraag naar de app af te handelen. Met dit patroon kunt u de elasticiteit van de openbare cloud gebruiken met uw on-premises oplossing.

## <a name="solution"></a>Oplossing

Het patroon voor schalen tussen de cloud breidt een app uit die zich in een lokale cloud met openbare cloudbronnen bevindt. Het patroon wordt geactiveerd door een toename of afname in de vraag en voegt respectievelijk resources toe aan of verwijdert ze in de cloud. Deze resources bieden redundantie, snelle beschikbaarheid en geo-compatibele routering.

![Patroon voor schalen in de cloud](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Dit patroon is alleen van toepassing op staatloze onderdelen van uw app.

## <a name="components"></a>Onderdelen

Het schaalpatroon voor meerdere cloudomgevingen bestaat uit de volgende onderdelen.

### <a name="outside-the-cloud"></a>Buiten de cloud

#### <a name="traffic-manager"></a>Traffic Manager

In het diagram bevindt dit zich buiten de groep van de openbare cloud, maar moet het verkeer in zowel het lokale datacenter als de openbare cloud kunnen coördineren. De balancer biedt hoge beschikbaarheid voor apps door eindpunten te bewaken en indien nodig failoverherdistributie te bieden.

#### <a name="domain-name-system-dns"></a>Domain Name System (DNS)

De Domain Name System DNS is verantwoordelijk voor het omzetten (of omzetten) van een website of servicenaam naar het IP-adres.

### <a name="cloud"></a>Cloud

#### <a name="hosted-build-server"></a>Gehoste buildserver

Een omgeving voor het hosten van uw build-pipeline.

#### <a name="app-resources"></a>App-resources

De app-resources moeten kunnen worden in- en uitschalen, zoals virtuele-machineschaalsets en containers.

#### <a name="custom-domain-name"></a>Aangepaste domeinnaam

Gebruik een aangepaste domeinnaam voor glob-routeringsaanvragen.

#### <a name="public-ip-addresses"></a>Openbare IP-adressen

Openbare IP-adressen worden gebruikt om het binnenkomende verkeer via Traffic Manager naar het resources-eindpunt van de openbare cloud-app te laten gaan.  

### <a name="local-cloud"></a>Lokale cloud

#### <a name="hosted-build-server"></a>Gehoste buildserver

Een omgeving voor het hosten van uw build-pipeline.

#### <a name="app-resources"></a>App-resources

De app-resources moeten kunnen worden in- en uitschalen, zoals virtuele-machineschaalsets en containers.

#### <a name="custom-domain-name"></a>Aangepaste domeinnaam

Gebruik een aangepaste domeinnaam voor glob-routeringsaanvragen.

#### <a name="public-ip-addresses"></a>Openbare IP-adressen

Openbare IP-adressen worden gebruikt om het binnenkomende verkeer via Traffic Manager naar het resources-eindpunt van de openbare cloud-app te laten gaan.

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Beschouw de volgende punten als u besluit hoe u dit patroon wilt implementeren:

### <a name="scalability"></a>Schaalbaarheid

Het belangrijkste onderdeel van schalen in de cloud is de mogelijkheid om schaalbaarheid op aanvraag te bieden. Schalen moet plaatsvinden tussen openbare en lokale cloudinfrastructuur en een consistente, betrouwbare service bieden op aanvraag.

### <a name="availability"></a>Beschikbaarheid

Zorg ervoor dat lokaal geïmplementeerde apps zijn geconfigureerd voor hoge beschikbaarheid via on-premises hardwareconfiguratie en software-implementatie.

### <a name="manageability"></a>Beheerbaarheid

Het patroon voor cloudoverschrijdende zorgt voor naadloos beheer en een vertrouwde interface tussen omgevingen.

## <a name="when-to-use-this-pattern"></a>Wanneer dit patroon gebruiken

U gebruikt dit patroon voor het volgende:

- Wanneer u de capaciteit van uw app wilt verhogen met onverwachte vraag of periodieke vraag.
- Wanneer u niet wilt investeren in resources die alleen tijdens pieken worden gebruikt. Betaal voor wat u gebruikt.

Dit patroon wordt niet aanbevolen wanneer:

- Uw oplossing vereist dat gebruikers verbinding maken via internet.
- Uw bedrijf heeft lokale voorschriften die vereisen dat de oorspronkelijke verbinding afkomstig is van een on-site-oproep.
- Uw netwerk ervaart regelmatig knelpunten die de prestaties van het schalen zouden beperken.
- Uw omgeving is niet verbonden met internet en kan de openbare cloud niet bereiken.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over onderwerpen die in dit artikel zijn geïntroduceerd:

- Zie het [Azure Traffic Manager overzicht](/azure/traffic-manager/traffic-manager-overview) voor meer informatie over hoe dit DNS-verkeer load balancer werkt.
- Zie [Ontwerpoverwegingen voor hybride toepassingen](overview-app-design-considerations.md) voor meer informatie over best practices en voor antwoorden op aanvullende vragen.
- Zie de [Azure Stack producten en oplossingen](/azure-stack) voor meer informatie over het hele portfolio met producten en oplossingen.

Wanneer u klaar bent om het voorbeeld van de oplossing te testen, gaat u verder met de implementatiehandleiding voor de oplossing voor [cloudschalen.](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling) De implementatiehandleiding bevat stapsgewijs instructies voor het implementeren en testen van de onderdelen. U leert hoe u een cloudoplossing kunt maken om een handmatig geactiveerd proces te bieden voor het overschakelen van een Azure Stack Hub gehoste web-app naar een door Azure gehoste web-app. U leert ook hoe u automatisch schalen kunt gebruiken via Traffic Manager, zodat u flexibel en schaalbaar cloudprogramma kunt gebruiken bij belasting.