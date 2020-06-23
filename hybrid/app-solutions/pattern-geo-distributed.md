---
title: Geografisch gedistribueerd app-patroon in Azure Stack hub
description: Meer informatie over het patroon voor geografisch gedistribueerde apps voor de intelligente rand met behulp van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910319"
---
# <a name="geo-distributed-app-pattern"></a>Patroon van geografisch gedistribueerde app

Meer informatie over het bieden van app-eind punten voor meerdere regio's en het routeren van gebruikers verkeer op basis van locatie-en nalevings behoeften.

## <a name="context-and-problem"></a>Context en probleem

Organisaties met een brede geografi streven ernaar om veilig en nauw keurig toegang te krijgen tot gegevens, terwijl er behoefte is aan beveiliging, naleving en prestaties per gebruiker, locatie en apparaat tussen de randen.

## <a name="solution"></a>Oplossing

Met het Azure Stack hub-verkeer voor geografische verkeers routering of geografisch gedistribueerde apps kan verkeer worden omgeleid naar specifieke eind punten op basis van verschillende metrische gegevens. Het maken van een Traffic Manager met op geografische basis gebaseerde route ring en eindpunt configuratie stuurt verkeer naar eind punten op basis van regionale vereisten, zakelijke en internationale regelgeving en gegevens behoeften.

![Geografisch gedistribueerd patroon](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Onderdelen

### <a name="outside-the-cloud"></a>Buiten de Cloud

#### <a name="traffic-manager"></a>Traffic Manager

In het diagram bevindt Traffic Manager zich buiten de open bare Cloud, maar moet het verkeer in het lokale Data Center en de open bare Cloud kunnen worden gecoördineerd. De Balancer routeert verkeer naar geografische locaties.

#### <a name="domain-name-system-dns"></a>Domain Name System (DNS)

De Domain Name System, of DNS, is verantwoordelijk voor het vertalen van een website-of service naam naar het IP-adres.

### <a name="public-cloud"></a>Openbare cloud

#### <a name="cloud-endpoint"></a>Cloud-eind punt

Open bare IP-adressen worden gebruikt voor het routeren van binnenkomend verkeer via Traffic Manager naar het eind punt van de open bare Cloud app-resources.  

### <a name="local-clouds"></a>Lokale Clouds

#### <a name="local-endpoint"></a>Lokaal eind punt

Open bare IP-adressen worden gebruikt voor het routeren van binnenkomend verkeer via Traffic Manager naar het eind punt van de open bare Cloud app-resources.

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Beschouw de volgende punten als u besluit hoe u dit patroon wilt implementeren:

### <a name="scalability"></a>Schaalbaarheid

Het patroon handelt geografische verkeers routering af in plaats van schalen om te voldoen aan toename van verkeer. U kunt dit patroon echter combi neren met andere Azure-en on-premises oplossingen. Dit patroon kan bijvoorbeeld worden gebruikt in combi natie met het patroon voor meerdere Cloud schalen.

### <a name="availability"></a>Beschikbaarheid

Zorg ervoor dat lokaal geïmplementeerde apps zijn geconfigureerd voor hoge Beschik baarheid via on-premises hardwareconfiguratie en software-implementatie.

### <a name="manageability"></a>Beheerbaarheid

Het patroon zorgt voor naadloos beheer en de vertrouwde interface tussen omgevingen.

## <a name="when-to-use-this-pattern"></a>Wanneer dit patroon gebruiken

- Mijn organisatie heeft internationale vertakkingen waarvoor aangepaste regionale beveiligings-en distributie beleidsregels zijn vereist.
- Elke kant oren van mijn organisatie halen werk nemer-, bedrijfs-en faciliteit gegevens op, waarvoor rapportage activiteiten per lokale regelgeving en tijd zone nodig zijn.
- Aan hoge schaal vereisten kan worden voldaan door apps horizon taal te schalen, waarbij meerdere app-implementaties worden uitgevoerd binnen één regio en in verschillende regio's om extreem belasting vereisten af te handelen.
- De apps moeten Maxi maal beschikbaar zijn en reageren op aanvragen van clients, zelfs in een storing in één regio.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:

- Zie het [overzicht van Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) voor meer informatie over hoe dit op DNS gebaseerd verkeer Load Balancer werkt.
- Bekijk de overwegingen voor het [ontwerpen van hybride apps](overview-app-design-considerations.md) voor meer informatie over aanbevolen procedures en voor het verkrijgen van antwoorden voor aanvullende vragen.
- Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [implementatie handleiding voor geografisch gedistribueerde app-oplossingen](solution-deployment-guide-geo-distributed.md). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen. U leert hoe u verkeer naar specifieke eind punten kunt omleiden, op basis van verschillende metrische gegevens met behulp van het geo-gedistribueerde app-patroon. Het maken van een Traffic Manager profiel met op geografische wijze gebaseerde route ring en eindpunt configuratie zorgt ervoor dat informatie wordt doorgestuurd naar eind punten op basis van regionale vereisten, zakelijke en internationale regelgeving en uw gegevens behoeften.
