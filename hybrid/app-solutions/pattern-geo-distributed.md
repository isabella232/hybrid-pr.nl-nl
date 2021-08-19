---
title: Geo-gedistribueerd app-patroon in Azure Stack Hub
description: Meer informatie over het patroon van geografisch gedistribueerde apps voor de intelligente rand met behulp van Azure en Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281224"
---
# <a name="geo-distributed-app-pattern"></a>Patroon geografisch gedistribueerde apps

Leer hoe u app-eindpunten in meerdere regio's kunt bieden en gebruikersverkeer kunt doorseen op basis van locatie- en nalevingsbehoeften.

## <a name="context-and-problem"></a>Context en probleem

Organisaties met uitgebreide geografische gebieden streven ernaar veilig en nauwkeurig toegang tot gegevens te distribueren en in te stellen terwijl de vereiste niveaus van beveiliging, naleving en prestaties per gebruiker, locatie en apparaat over de grenzen heen worden gewaarborgd.

## <a name="solution"></a>Oplossing

Met Azure Stack Hub geografische verkeersrouteringspatroon, of geografisch gedistribueerde apps, kan verkeer worden omgeleid naar specifieke eindpunten op basis van verschillende metrische gegevens. Het maken Traffic Manager met geografische routering en eindpuntconfiguratie routeert verkeer naar eindpunten op basis van regionale vereisten, bedrijfs- en internationale regelgeving en gegevensbehoeften.

![Geografisch gedistribueerd patroon](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Onderdelen

### <a name="outside-the-cloud"></a>Buiten de cloud

#### <a name="traffic-manager"></a>Traffic Manager

In het diagram Traffic Manager zich buiten de openbare cloud, maar het moet verkeer in zowel het lokale datacenter als de openbare cloud kunnen coördineren. De balancer routeer verkeer naar geografische locaties.

#### <a name="domain-name-system-dns"></a>Domain Name System (DNS)

De Domain Name System dns is verantwoordelijk voor het omzetten (of omzetten) van een website of servicenaam naar het IP-adres.

### <a name="public-cloud"></a>Openbare cloud

#### <a name="cloud-endpoint"></a>Cloud-eindpunt

Openbare IP-adressen worden gebruikt om het binnenkomende verkeer via Traffic Manager naar het eindpunt van de openbare cloud-app-resources te routeeren.  

### <a name="local-clouds"></a>Lokale clouds

#### <a name="local-endpoint"></a>Lokaal eindpunt

Openbare IP-adressen worden gebruikt om het binnenkomende verkeer via Traffic Manager naar het eindpunt van de openbare cloud-app-resources te routeeren.

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Beschouw de volgende punten als u besluit hoe u dit patroon wilt implementeren:

### <a name="scalability"></a>Schaalbaarheid

Het patroon verwerkt geografische verkeersroutering in plaats van schalen om te voldoen aan de toename van het verkeer. U kunt dit patroon echter combineren met andere Azure- en on-premises oplossingen. Dit patroon kan bijvoorbeeld worden gebruikt met het patroon voor schalen in de cloud.

### <a name="availability"></a>Beschikbaarheid

Zorg ervoor dat lokaal geïmplementeerde apps zijn geconfigureerd voor hoge beschikbaarheid via on-premises hardwareconfiguratie en software-implementatie.

### <a name="manageability"></a>Beheerbaarheid

Het patroon zorgt voor naadloos beheer en een vertrouwde interface tussen omgevingen.

## <a name="when-to-use-this-pattern"></a>Wanneer dit patroon gebruiken

- Mijn organisatie heeft internationale vertakkingen waarvoor aangepast regionaal beveiligings- en distributiebeleid is vereist.
- Elk kantoor van mijn organisatie haalt werknemers-, bedrijfs- en faciliteitsgegevens op, waarvoor rapportageactiviteit per lokale regelgeving en tijdzone is vereist.
- Aan grootschalige vereisten kan worden voldaan door horizontaal apps uit te schalen, met meerdere app-implementaties binnen één regio en tussen regio's om te voldoen aan extreme belastingsvereisten.
- De apps moeten zeer beschikbaar zijn en responsief zijn op clientaanvragen, zelfs bij uitval in één regio.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over onderwerpen die in dit artikel zijn geïntroduceerd:

- Zie het [Azure Traffic Manager overzicht](/azure/traffic-manager/traffic-manager-overview) voor meer informatie over de manier waarop dit DNS-verkeer load balancer werkt.
- Zie [Overwegingen voor het ontwerp van hybride apps](overview-app-design-considerations.md) voor meer informatie over best practices en voor antwoorden op aanvullende vragen.
- Zie de [Azure Stack producten en oplossingen voor](/azure-stack) meer informatie over het hele portfolio met producten en oplossingen.

Wanneer u klaar bent om het voorbeeld van de oplossing te testen, gaat u verder met de implementatiehandleiding voor de [geografisch gedistribueerde app-oplossing](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed). De implementatiehandleiding bevat stapsgewijs instructies voor het implementeren en testen van de onderdelen. U leert hoe u verkeer naar specifieke eindpunten kunt leiden op basis van verschillende metrische gegevens met behulp van het geografisch gedistribueerde app-patroon. Het maken Traffic Manager profiel met geografische routering en eindpuntconfiguratie zorgt ervoor dat informatie wordt gerouteerd naar eindpunten op basis van regionale vereisten, bedrijfs- en internationale regelgeving en uw gegevensbehoeften.