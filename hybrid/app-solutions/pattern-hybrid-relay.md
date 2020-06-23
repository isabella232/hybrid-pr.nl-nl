---
title: Hybride relay-patroon in Azure en Azure Stack hub
description: Gebruik het hybride relay-patroon in Azure en Azure Stack hub om verbinding te maken met Edge-bronnen die worden beveiligd door firewalls.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910433"
---
# <a name="hybrid-relay-pattern"></a>Hybride relay-patroon

Meer informatie over hoe u verbinding maakt met Edge-bronnen of apparaten die door firewalls worden beveiligd met het hybride relay-patroon en Azure Relay.

## <a name="context-and-problem"></a>Context en probleem

Edge-apparaten bevinden zich vaak achter een bedrijfs firewall of een NAT-apparaat. Hoewel ze veilig zijn, kunnen ze mogelijk niet communiceren met de open bare Cloud of edge-apparaten op andere bedrijfs netwerken. Het kan nodig zijn om op een veilige manier bepaalde poorten en functionaliteit beschikbaar te stellen aan gebruikers in de open bare Cloud.

## <a name="solution"></a>Oplossing

Het hybride relay-patroon gebruikt Azure Relay om een websockets-tunnel te maken tussen twee eind punten die niet rechtstreeks kunnen communiceren. Apparaten die niet on-premises zijn maar verbinding moeten maken met een on-premises eind punt, maken verbinding met een eind punt in de open bare Cloud. Met dit eind punt wordt het verkeer omgeleid naar vooraf gedefinieerde routes via een beveiligd kanaal. Een eind punt in de on-premises omgeving ontvangt het verkeer en stuurt het naar de juiste bestemming.

![architectuur van hybride relay-patroon oplossing](media/pattern-hybrid-relay/solution-architecture.png)

Hoe het hybride relay-patroon werkt:

1. Een apparaat maakt verbinding met de virtuele machine (VM) in azure, op een vooraf gedefinieerde poort.
2. Verkeer wordt doorgestuurd naar de Azure Relay in Azure.
3. De VM op Azure Stack hub, die al een langdurige verbinding met de Azure Relay heeft gemaakt, ontvangt het verkeer en stuurt het door naar de bestemming.
4. De on-premises service of het eind punt verwerkt de aanvraag.

## <a name="components"></a>Onderdelen

Deze oplossing maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| Azure | Azure VM | Een Azure-VM biedt een openbaar toegankelijk eind punt voor de on-premises resource. |
| | Azure Relay | Een [Azure relay](/azure/azure-relay/) biedt de infra structuur voor het onderhouden van de tunnel en de verbinding tussen de virtuele machine van Azure en Azure stack hub.|
| Azure Stack hub | Compute | Een Azure Stack hub-VM levert de server zijde van de hybride relay-tunnel. |
| | Storage | Het AKS-engine cluster dat is geïmplementeerd in Azure Stack hub biedt een schaal bare, robuuste engine voor het uitvoeren van de Face-API-container.|

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten wanneer u bepaalt hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

Met dit patroon kunnen alleen 1:1 poort toewijzingen op de client en de server worden toegewezen. Als poort 80 bijvoorbeeld is getunneld voor één service op het Azure-eind punt, kan deze niet worden gebruikt voor een andere service. Poort toewijzingen moeten dienovereenkomstig worden gepland. De Azure Relay en Vm's moeten op de juiste manier worden geschaald om verkeer te verwerken.

### <a name="availability"></a>Beschikbaarheid

Deze tunnels en verbindingen zijn niet overbodig. Om hoge Beschik baarheid te garanderen, wilt u mogelijk fout controle code implementeren. Een andere optie is om een groep met Azure Relay verbonden Vm's achter een load balancer te hebben.

### <a name="manageability"></a>Beheerbaarheid

Deze oplossing kan veel apparaten en locaties omvatten, wat een onhandigheid kan oplevert. Met de IoT-services van Azure kunnen nieuwe locaties en apparaten automatisch online worden gebracht en up-to-date blijven.

### <a name="security"></a>Beveiliging

Met dit patroon zoals weer gegeven kunt u onbelemmerde toegang geven tot een poort op een intern apparaat vanaf de rand. Overweeg een verificatie mechanisme toe te voegen aan de service op het interne apparaat of vóór het hybride relay-eind punt.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:

- Dit patroon maakt gebruik van Azure Relay. Zie de [Azure relay-documentatie](/azure/azure-relay/)voor meer informatie.
- Zie [overwegingen voor het ontwerpen van hybride toepassingen](overview-app-design-considerations.md) voor meer informatie over best practices en krijg antwoorden op eventuele aanvullende vragen.
- Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [implementatie handleiding voor hybride relay-oplossingen](https://aka.ms/hybridrelaydeployment). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.