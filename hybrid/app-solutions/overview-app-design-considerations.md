---
title: Overwegingen voor het ontwerpen van hybride apps in Azure en Azure Stack hub
description: Meer informatie over ontwerp overwegingen bij het bouwen van een hybride app voor de intelligente Cloud en intelligente rand, waaronder plaatsing, schaal baarheid, Beschik baarheid en tolerantie.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4fd52f76baad8059e130adfc01cdd0152b40a510
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910420"
---
# <a name="hybrid-app-design-considerations"></a>Overwegingen voor het ontwerpen van hybride apps

Microsoft Azure is de enige consistente hybride Cloud. Zo kunt u uw ontwikkelings investeringen hergebruiken en apps inschakelen die wereld wijd Azure, de soevereine Azure-Clouds en Azure Stack, een uitbrei ding van Azure in uw Data Center. Apps die Clouds omvatten, worden ook *hybrid apps*genoemd.

In de [*hand leiding Azure-toepassing architectuur*](https://docs.microsoft.com/azure/architecture/guide) wordt een gestructureerde benadering beschreven voor het ontwerpen van apps die schaalbaar, robuust en Maxi maal beschikbaar zijn. De overwegingen die in de [*Azure-toepassing architectuur handleiding*](https://docs.microsoft.com/azure/architecture/guide) zijn beschreven, zijn gelijk van toepassing op apps die zijn ontworpen voor één Cloud en voor apps die Clouds omvatten.

In dit artikel worden de [*pijlers van de software kwaliteit*](https://docs.microsoft.com/azure/architecture/guide/pillars) beschreven die worden besproken in de [*Azure-toepassing*](https://docs.microsoft.com/azure/architecture/guide/) [ *Architecture-hand leiding*,](https://docs.microsoft.com/azure/architecture/guide/) die specifiek gericht is op het ontwerpen van hybride apps. Daarnaast voegen we een *placement* -pijler toe als hybride apps niet exclusief voor één Cloud of een on-premises Data Center.

Hybride scenario's variëren aanzienlijk met de resources die beschikbaar zijn voor ontwikkeling en beschik bare overwegingen zoals geografie, beveiliging, Internet toegang en andere overwegingen. Hoewel deze hand leiding uw specifieke overwegingen niet kan inventariseren, kan deze een aantal belang rijke richt lijnen en aanbevolen procedures bieden die u kunt volgen. Het ontwerpen, configureren, implementeren en onderhouden van een hybride app-architectuur omvat veel ontwerp overwegingen die mogelijk niet bekend zijn bij u.

Dit document is gericht op het samen stellen van de mogelijke vragen die zich kunnen voordoen bij het implementeren van hybride apps en het bieden van overwegingen (deze pijlers) en de aanbevolen procedures om ermee te werken. Door deze vragen tijdens de ontwerp fase te adresseren, voor komt u de problemen die ze kunnen veroorzaken in de productie.

Dit zijn in feite vragen die u moet overwegen voordat u een hybride app gaat maken. Om aan de slag te gaan, moet u het volgende doen:

- De app-onderdelen identificeren en evalueren.
- Evalueer app-onderdelen op de pijlers.

## <a name="evaluate-the-app-components"></a>De app-onderdelen evalueren

Elk onderdeel van een app heeft een eigen specifieke rol binnen de grotere app en moet worden geëvalueerd met alle ontwerp overwegingen. De vereisten en functies van elk onderdeel moeten worden toegewezen aan deze overwegingen om te helpen bij het bepalen van de app-architectuur.

Deel uw app af in de onderdelen door de architectuur van uw app te beoordelen en te bepalen wat deze bevat. Onderdelen kunnen ook andere apps bevatten die door uw app worden uitgevoerd. Wanneer u de onderdelen identificeert, evalueert u uw beoogde hybride bewerkingen op basis van hun kenmerken door de volgende vragen te stellen:

- Wat is het doel van het onderdeel?
- Wat zijn de onderlinge afhankelijkheden tussen de onderdelen?

Een app kan bijvoorbeeld een front-end en een back-end als twee onderdelen hebben gedefinieerd. In een hybride scenario bevindt de front-end zich in één Cloud en de back-end in de andere. De app biedt communicatie kanalen tussen de front-end en de gebruiker, en ook tussen de front-end en de back-end.

Een app-onderdeel wordt gedefinieerd door veel formulieren en scenario's. De belangrijkste taak is het identificeren ervan en hun Cloud-of on-premises locatie.

De algemene app-onderdelen die in uw inventaris moeten worden opgenomen, worden weer gegeven in tabel 1.

### <a name="table-1-common-app-components"></a>Tabel 1. Algemene app-onderdelen

| **Onderdeel** | **Richt lijnen voor hybride apps** |
| ---- | ---- |
| Clientverbindingen | Uw app (op elk apparaat) kan op verschillende manieren toegang krijgen tot gebruikers, vanaf een enkel invoer punt, met inbegrip van de volgende manieren:<br>-Een client-server model waarvoor de gebruiker een-client moet hebben geïnstalleerd om met de app te kunnen werken. Een op een server gebaseerde app die wordt geopend vanuit een browser.<br>-Client verbindingen kunnen meldingen bevatten wanneer de verbinding wordt verbroken of waarschuwingen wanneer er kosten voor roaming kunnen worden toegepast. |
| Verificatie  | Verificatie kan vereist zijn voor een gebruiker die verbinding maakt met de app of van het ene onderdeel dat verbinding maakt met het andere. |
| API's  | U kunt ontwikkel aars programmatisch toegang geven tot uw app met API-sets en klassen bibliotheken en een verbindings interface bieden op basis van Internet standaarden. U kunt ook Api's gebruiken om een app af te breken in onafhankelijk werkende logische eenheden. |
| Services  | U kunt een bondig service gebruiken om de functies voor een app te bieden. Een service kan de engine zijn waarop de app wordt uitgevoerd. |
| Wachtrijen | U kunt wacht rijen gebruiken om de status van de levens cycli en statussen van de onderdelen van uw app te organiseren. Deze wacht rijen kunnen berichten, meldingen en buffer mogelijkheden bieden voor het abonneren van partijen. |
| Gegevensopslag | Een app kan stateless of stateful zijn. Stateful apps hebben gegevens opslag nodig waaraan kan worden voldaan door talloze indelingen en volumes. |
| Gegevens in de cache  | Een onderdeel voor het opslaan van gegevens in uw ontwerp kan strategische problemen met de latentie aanpakken en een rol spelen in het activeren van Cloud bursting. |
| Gegevensopname | Gegevens kunnen op verschillende manieren naar een app worden verzonden, variërend van door de gebruiker ingediende waarden in een webformulier tot continu gegevens stromen met een hoog volume. |
| Gegevensverwerking | Uw gegevens verwerkings taken (zoals rapporten, analyses, batch uitvoer en gegevens transformatie) kunnen worden verwerkt op de bron of worden opgedeeld in een afzonderlijk onderdeel met behulp van een kopie van de gegevens. |

## <a name="assess-app-components-for-pillars"></a>App-onderdelen voor pijlers beoordelen

Voor elk onderdeel evalueren de kenmerken voor elke pijler. Wanneer u elk onderdeel evalueert met alle pijlers, kunnen vragen die u mogelijk niet hebt gezien, bekend zijn bij u die van invloed zijn op het ontwerp van de hybride app. Met deze overwegingen zou u een waarde kunnen toevoegen in de optimalisatie van uw app. Tabel 2 bevat een beschrijving van elke pijler die betrekking heeft op hybride apps.

### <a name="table-2-pillars"></a>Tabel 2. Pijlers

| **Pijler** | **Beschrijving** |
| ----------- | --------------------------------------------------------- |
| Plaatsing  | De strategische positionering van onderdelen in hybride apps. |
| Schaalbaarheid  | De mogelijkheid van een systeem om toegenomen belasting af te handelen. |
| Beschikbaarheid  | Het deel van de tijd dat een hybride app functioneel is en werkt. |
| Flexibiliteit | De mogelijkheid om een hybride app te herstellen. |
| Beheerbaarheid | Operationele processen die ervoor zorgen dat een systeem blijft werken. |
| Beveiliging | Bescherming van hybride apps en gegevens tegen bedreigingen. |

## <a name="placement"></a>Plaatsing

Een hybride app heeft een plaatsings overweging, zoals voor het Data Center.

Plaatsing is de belang rijke taak van het plaatsen van onderdelen, zodat deze een hybride app kunnen best preserviceren. Per definitie, locaties hybride apps, zoals van on-premises naar de Cloud en tussen verschillende Clouds. U kunt op twee manieren onderdelen van de app op Clouds plaatsen:

- **Verticaal hybride apps**  
    App-onderdelen worden verdeeld over verschillende locaties. Elk afzonderlijk onderdeel kan meerdere instanties bevatten die zich alleen op één locatie bevinden.

- **Horizontale hybride apps**  
    App-onderdelen worden verdeeld over verschillende locaties. Elk afzonderlijk onderdeel kan meerdere exemplaren hebben die meerdere locaties bespannen.

    Sommige onderdelen zijn op de hoogte van hun locatie, terwijl anderen geen kennis hebben van hun locatie en plaatsing. Deze virtuousness kan worden bereikt met een abstractie laag. Deze laag, met een modern app-Framework zoals micro Services, kan bepalen hoe de app wordt verwerkt door de plaatsing van app-onderdelen die worden uitgevoerd op knoop punten in verschillende Clouds.

### <a name="placement-checklist"></a>Controle lijst plaatsing

**Controleer de vereiste locaties.** Zorg ervoor dat de app of een van de bijbehorende onderdelen vereist zijn voor het uitvoeren van of het vereisen van certificering voor een specifieke Cloud. Dit kan betrekking hebben op de soevereiniteit-vereisten van uw bedrijf of door de wet worden gedicteerd. Bepaal ook of een on-premises bewerking is vereist voor een bepaalde locatie of land instelling.

**Connectiviteits afhankelijkheden vaststellen.** De vereiste locaties en andere factoren kunnen de connectiviteits afhankelijkheden van uw onderdelen bepalen. Bepaal bij het plaatsen van de onderdelen de optimale connectiviteit en beveiliging voor communicatie. Dit zijn onder andere [ *VPN-*,](https://docs.microsoft.com/azure/vpn-gateway/) [ *ExpressRoute*](https://docs.microsoft.com/azure/expressroute/) -en [ *hybride verbindingen*.](https://docs.microsoft.com/azure/app-service/app-service-hybrid-connections)

**Bepaal de platform mogelijkheden.** Voor elk app-onderdeel raadpleegt u of de vereiste resource provider voor het app-onderdeel beschikbaar is in de Cloud en of de band breedte kan voldoen aan de verwachte vereisten voor door Voer en latentie.

**Transport baarheid plannen.** Gebruik moderne app-frameworks, zoals containers of micro Services, voor het plannen van het verplaatsen van bewerkingen en om service afhankelijkheden te voor komen.

**Bepaal de vereisten voor data soevereiniteit.** Hybride apps zijn geschikt voor de isolatie van gegevens, zoals op een lokaal Data Center. Bekijk de plaatsing van uw resources om het succes van deze vereiste te optimaliseren.

**Een latentie plannen.** Tussen Cloud bewerkingen kunnen fysieke afstanden tussen de onderdelen van de app worden geïntroduceerd. Controleer de vereisten voor een wille keurige latentie.

**Verkeers stromen beheren.** Verwerk het piek gebruik en de juiste en beveiligde communicatie voor persoonlijke gegevens gegevens wanneer deze worden geopend door de front-end in een open bare Cloud.

## <a name="scalability"></a>Schaalbaarheid

Schaal baarheid is de mogelijkheid van een systeem om een grotere belasting van een app te verwerken. deze kan in de loop van de tijd variëren, omdat andere factoren en krachten de omvang van de doel groep beïnvloeden, naast de grootte en het bereik van de app.

Zie [*schaal baarheid*](https://docs.microsoft.com/azure/architecture/guide/pillars#scalability) in de vijf pijlers van de architectuur uitmuntendheid voor de belangrijkste bespreking van deze pijler.

Met een horizontale schaal benadering voor hybride apps kunt u meer exemplaren toevoegen om aan de vraag te voldoen en ze vervolgens uitschakelen tijdens stilte tijd.

In hybride scenario's moet u voor het uitschalen van afzonderlijke onderdelen extra aandacht schenken wanneer onderdelen over verschillende Clouds worden verspreid. Als u een deel van de app wilt schalen, kan de schaal van een andere worden aangepast. Als bijvoorbeeld het aantal client verbindingen toeneemt, maar de webservices van de app op de juiste wijze niet zijn geschaald, kan de belasting van de data base de app verzadigen.

Sommige app-onderdelen kunnen lineair uitschalen, terwijl andere afhankelijkheden voor schalen hebben en mogelijk beperkt zijn tot de mate waarin ze kunnen worden geschaald. Een VPN-tunnel met hybride connectiviteit voor de locaties van app-onderdelen heeft bijvoorbeeld een limiet voor de band breedte en latentie waarmee deze kan worden geschaald. Hoe worden onderdelen van de app geschaald om ervoor te zorgen dat aan deze vereisten wordt voldaan?

### <a name="scalability-checklist"></a>Controlelijst voor schaalbaarheid

**Drempel waarden voor schalen vaststellen.** Voor het afhandelen van de verschillende afhankelijkheden in uw app, bepaalt u de mate waarin app-onderdelen in verschillende Clouds onafhankelijk van elkaar kunnen worden geschaald, terwijl ze toch voldoen aan de vereisten voor het uitvoeren van de app. Hybride apps moeten vaak bepaalde gebieden in de app schalen om een functie te kunnen afhandelen en die van invloed zijn op de rest van de app. Als u bijvoorbeeld een aantal front-end-exemplaren overschrijdt, moet u mogelijk de back-end schalen.

**Schaal schema's definiëren.** De meeste apps hebben drukke Peri Oden, dus u moet hun piek tijden samen voegen in schema's om de optimale schaal aanpassing te coördineren.

**Een gecentraliseerd bewakings systeem gebruiken.** De mogelijkheden voor het controleren van het platform kunnen automatisch worden geschaald, maar hybride apps hebben een gecentraliseerd bewakings systeem nodig waarmee de systeem status en-belasting worden geaggregeerd. Een gecentraliseerd bewakings systeem kan een resource op één locatie schalen en schalen, afhankelijk van de bron op een andere locatie. Daarnaast kan een systeem voor centrale bewaking bijhouden welke Clouds resources automatisch schalen en welke Clouds dat niet doen.

**Maak gebruik van mogelijkheden voor automatisch schalen (indien beschikbaar).** Als de mogelijkheden voor automatisch schalen deel uitmaken van uw architectuur, implementeert u automatisch schalen door drempel waarden in te stellen waarmee wordt gedefinieerd wanneer een app-onderdeel omhoog, omlaag of omlaag moet worden geschaald. Een voor beeld van automatisch schalen is het maken van een client verbinding in één Cloud om verhoogde capaciteit af te handelen, maar zorgt ervoor dat andere afhankelijkheden van de app worden gespreid over verschillende Clouds, om ook te worden geschaald. De mogelijkheden voor automatisch schalen van deze afhankelijke onderdelen moeten worden vastgesteld.

Als automatisch schalen niet beschikbaar is, kunt u overwegen om scripts en andere bronnen te implementeren voor hand matige schaling, die wordt geactiveerd door drempel waarden in het gecentraliseerde bewakings systeem.

**Bepaal de verwachte belasting per locatie.** Hybride apps die client aanvragen verwerken, kunnen voornamelijk op één locatie vertrouwen. Wanneer het laden van client aanvragen een drempel waarde overschrijdt, kunnen er extra resources worden toegevoegd op een andere locatie om de belasting van inkomende aanvragen te verdelen. Zorg ervoor dat de client verbindingen de toegenomen belastingen kunnen verwerken en om ook geautomatiseerde procedures te bepalen voor de client verbindingen om de belasting te verwerken.

## <a name="availability"></a>Beschikbaarheid

Beschik baarheid is het tijdstip waarop een systeem functioneel is en werkt. Beschik baarheid wordt gemeten als percentage van de uptime. App-fouten, infrastructuur problemen en systeem belasting kunnen alle Beschik baarheid verminderen.

Zie [*Beschik baarheid*](/azure/architecture/framework/) in de vijf pijlers van de architectuur uitmuntendheid voor de belangrijkste bespreking van deze pijler.

### <a name="availability-checklist"></a>Controlelijst voor beschikbaarheid

**Redundantie voor connectiviteit bieden.** Hybride apps vereisen connectiviteit tussen de Clouds waarin de app is verspreid. U hebt de keuze uit technologieën voor hybride connectiviteit, dus naast uw primaire technologie, kunt u ook een andere technologie gebruiken om redundantie te bieden met geautomatiseerde failover-mogelijkheden als de primaire technologie mislukt.

**Fout domeinen classificeren.** Voor fout tolerante apps zijn meerdere fout domeinen vereist. Fout domeinen helpen bij het isoleren van het probleem, bijvoorbeeld als er een storing in één harde schijf optreedt op het moment dat een top-of-rack-switch uitvalt, of als het volledige Data Center niet beschikbaar is. In een hybride app kan een locatie worden geclassificeerd als een fout domein. Meer informatie over de beschik baarheid is het meer wat u nodig hebt om te evalueren hoe één fout domein moet worden geclassificeerd.

**De upgrade domeinen worden geclassificeerd.** Upgrade domeinen worden gebruikt om ervoor te zorgen dat exemplaren van app-onderdelen beschikbaar zijn, terwijl andere exemplaren van hetzelfde onderdeel worden onderhouden met updates of functie-upgrades. Net als bij fout domeinen kunnen upgrade domeinen worden geclassificeerd op basis van hun plaatsing op verschillende locaties. U moet bepalen of een app-onderdeel een upgrade op een locatie kan uitvoeren voordat het op een andere locatie wordt geüpgraded, of dat er andere domein configuraties vereist zijn. Eén locatie kan zichzelf meerdere upgrade domeinen hebben.

**Volg instanties en beschik baarheid.** Maxi maal beschik bare app-onderdelen kunnen beschikbaar zijn via taak verdeling en synchrone gegevens replicatie. U moet bepalen hoeveel instanties offline kunnen zijn voordat de service wordt onderbroken.

**Implementeer zelf herstel.** In het geval van een probleem veroorzaakt een onderbreking van de app-beschik baarheid, een detectie door een bewakings systeem kan zelf herstel activiteiten initiëren naar de app, zoals het verwerken van het mislukte exemplaar en het opnieuw implementeren ervan. Dit is waarschijnlijk een centrale bewakings oplossing, geïntegreerd met een hybride, continue integratie en continue levering (CI/CD)-pijp lijn. De app is geïntegreerd met een bewakings systeem om problemen te identificeren waarvoor een app-onderdeel opnieuw moet worden geïmplementeerd. Het bewakings systeem kan ook Hybrid CI/CD activeren om het app-onderdeel opnieuw te implementeren en mogelijk ook andere afhankelijke onderdelen op dezelfde of andere locaties.

**Service Level Agreements (Sla's) onderhouden.** Beschik baarheid is van cruciaal belang voor alle overeenkomsten voor het onderhouden van de connectiviteit met de services en apps die u met uw klanten hebt. Elke locatie waar uw hybride app afhankelijk is, heeft mogelijk een eigen SLA. Deze verschillende Sla's kunnen van invloed zijn op de algemene SLA van uw hybride app.

## <a name="resiliency"></a>Flexibiliteit

Tolerantie is de mogelijkheid voor een hybride app en een systeem om fouten op te lossen en verder te werken. Het doel van tolerantie is het retour neren van de app naar een volledig functionerende status nadat een fout is opgetreden. Flexibiliteits strategieën zijn oplossingen zoals back-ups, replicatie en nood herstel.

Zie [*tolerantie*](https://docs.microsoft.com/azure/architecture/guide/pillars#resiliency) in de vijf pijlers van de architectuur uitmuntendheid voor de belangrijkste bespreking van deze pijler.

### <a name="resiliency-checklist"></a>Controlelijst voor tolerantie

**Bewaak afhankelijkheden voor nood herstel.** Herstel na nood gevallen in één Cloud vereist mogelijk wijzigingen in app-onderdelen in een andere cloud. Als een of meer onderdelen van de ene Cloud een failover naar een andere locatie hebben, binnen dezelfde Cloud of naar een andere Cloud, moeten de afhankelijke onderdelen op de hoogte worden gesteld van deze wijzigingen. Dit geldt ook voor de connectiviteits afhankelijkheden. Voor tolerantie is een volledig getest app-herstel plan voor elke Cloud vereist.

**Herstel stroom instellen.** Een effectief herstel stroom ontwerp heeft evaluatie van de app-onderdelen voor de mogelijkheid om buffers toe te passen, nieuwe pogingen, mislukte gegevens overdracht opnieuw proberen en, indien nodig, terugvallen op een andere service of werk stroom. U moet bepalen welk back-upmechanisme moet worden gebruikt, wat de herstel procedure betreft en hoe vaak het wordt getest. U moet ook de frequentie voor zowel incrementele als volledige back-ups bepalen.

**Gedeeltelijke herstel bewerkingen testen.** Een gedeeltelijk herstel voor een deel van de app kan een herverzekering bieden voor gebruikers die niet niet beschikbaar zijn. Dit deel van het plan moet ervoor zorgen dat een gedeeltelijke terugzet bewerking geen neven effecten heeft, zoals een service voor back-up en herstel die communiceert met de app om deze op de juiste manier af te sluiten voordat de back-up wordt gemaakt.

**Bepaal de nood herstel-en toewijzings verantwoordelijkheid.** In een herstel plan moet worden beschreven wie, en welke functies, acties voor back-up en herstel kunnen initiëren naast waarvan een back-up kan worden gemaakt en hersteld.

**Vergelijkt de drempel waarden voor zelf herstel met nood herstel.** Bepaal de zelf herstel mogelijkheden van een app voor automatische herstel initiatie en de tijd die nodig is om de zelf herstel van een app te kunnen beschouwen als een storing of slagen. Bepaal de drempel waarden voor elke Cloud.

**Controleer de beschik baarheid van tolerantie functies.** Bepaal de beschik baarheid van tolerantie functies en mogelijkheden voor elke locatie. Als een locatie niet de vereiste mogelijkheden biedt, kunt u deze locatie integreren in een gecentraliseerde service die de tolerantie functies biedt.

**Uitval tijd vaststellen.** Bepaal de verwachte downtime als gevolg van onderhoud voor de app als geheel en als app-onderdelen.

**Procedures voor het oplossen van problemen met documenten.** Definieer probleemoplossings procedures voor het opnieuw implementeren van resources en app-onderdelen.

## <a name="manageability"></a>Beheerbaarheid

De overwegingen voor het beheren van uw hybride apps zijn van cruciaal belang bij het ontwerpen van uw architectuur. Een goed beheerde hybride app biedt een infra structuur als code waarmee de integratie van consistente app-code in een gemeen schappelijke ontwikkelings pijplijn mogelijk wordt. Door het implementeren van consistente systeem-en individuele tests van wijzigingen in de infra structuur, kunt u ervoor zorgen dat een geïntegreerde implementatie als de wijzigingen de tests door geven, zodat ze in de bron code kunnen worden samengevoegd.

Zie [*DevOps*](/azure/architecture/framework/#devops) in de vijf pijlers van de architectuur uitmuntendheid voor de belangrijkste bespreking van deze pijler.

### <a name="manageability-checklist"></a>Controle lijst voor beheer baarheid

**Bewaking implementeren.** Gebruik een gecentraliseerd bewakings systeem van app-onderdelen die over verschillende Clouds verspreid zijn om een geaggregeerde weer gave van hun status en prestaties te bieden. Dit systeem omvat het bewaken van de app-onderdelen en de mogelijkheden van het gerelateerde platform.

Bepaal welke onderdelen van de app moeten worden bewaakt.

**Beleid voor coördinaten.** Elke locatie waar een hybride app zich bevindt, kan een eigen beleid hebben dat betrekking heeft op toegestane resource typen, naam conventies, tags en andere criteria.

**Rollen definiëren en gebruiken.** Als database beheerder moet u de vereiste machtigingen voor de verschillende personen bepalen (zoals een eigenaar van een app, een database beheerder en een eind gebruiker) die toegang nodig hebben tot de resources van de app. Deze machtigingen moeten worden geconfigureerd voor de resources en binnen de app. Met een op rollen gebaseerd toegangs beheer (RBAC)-systeem kunt u deze machtigingen instellen voor de app-resources. Deze toegangs rechten zijn lastig wanneer alle resources in één Cloud worden geïmplementeerd, maar nog meer aandacht vereisen wanneer de resources over verschillende Clouds worden verspreid. Machtigingen voor resources die zijn ingesteld in de ene Cloud, gelden niet voor resources die in een andere cloud zijn ingesteld.

**Gebruik CI/CD-pijp lijnen.** Een continue integratie-en continue ontwikkeling (CI/CD)-pijp lijn kan een consistent proces bieden voor het ontwerpen en implementeren van apps die verspreid over Clouds zijn, en om kwaliteits garanties te bieden voor de infra structuur en de app. Met deze pijp lijn kunnen de infra structuur en app worden getest in één Cloud en geïmplementeerd in een andere cloud. Met de pijp lijn kunt u zelfs bepaalde onderdelen van uw hybride app implementeren in één Cloud en andere onderdelen naar een andere Cloud, in feite de basis vormen voor de implementatie van hybride apps. Een CI/CD-systeem is van cruciaal belang voor het afhandelen van de onderdelen van de app met afhankelijkheden voor elkaar tijdens de installatie, zoals de web-app die een connection string voor de data base nodig heeft.

**De levens cyclus beheren.** Omdat resources van een hybride app locaties kunnen omvatten, moeten de beheer mogelijkheden voor de levens cyclus van elke enkele locatie worden geaggregeerd in één beheer eenheid voor levens cyclus. Denk na over hoe ze worden gemaakt, bijgewerkt en verwijderd.

**Bekijk strategieën voor het oplossen van problemen.** Voor het oplossen van problemen met een hybride app zijn meer app-onderdelen vereist dan voor de app die wordt uitgevoerd in een enkele Cloud. Naast de connectiviteit tussen de Clouds, wordt de app uitgevoerd op twee platformen in plaats van één. Een belang rijke taak bij het oplossen van hybride apps is het onderzoeken van de geaggregeerde status en prestaties bewaking van de app-onderdelen.

## <a name="security"></a>Beveiliging

Beveiliging is een van de belangrijkste aandachtspunten voor elke Cloud-app en wordt nog eens kritiek voor hybride Cloud-apps.

Zie [*beveiliging*](https://docs.microsoft.com/azure/architecture/guide/pillars#security) in de vijf pijlers van de architectuur uitmuntendheid voor de belangrijkste bespreking van deze pijler.

### <a name="security-checklist"></a>Controlelijst voor beveiliging

**Ga ervan uit dat de schending.** Als er sprake is van een deel van de app, moet u ervoor zorgen dat er oplossingen zijn om de verspreiding van de schending zo klein mogelijk te houden, niet alleen op dezelfde locatie, maar ook op verschillende locaties.

**De toegestane netwerk toegang bewaken.** Bepaal het netwerk toegangs beleid voor de app, zoals alleen toegang tot de app vanuit een specifiek subnet en sta alleen de minimum poorten en protocollen toe tussen de onderdelen die nodig zijn om de app correct te laten functioneren.

**Krachtige verificatie gebruiken.** Een robuust verificatie schema is essentieel voor de beveiliging van uw app. Overweeg het gebruik van een federatieve id-provider die mogelijkheden voor eenmalige aanmelding biedt en gebruikmaakt van een of meer van de volgende schema's: gebruikers naam en wacht woord aanmelden, open bare en persoonlijke sleutels, twee ledige of meervoudige verificatie en vertrouwde beveiligings groepen. Bepaal de juiste bronnen voor het opslaan van gevoelige gegevens en andere geheimen voor app-verificatie, naast de certificaat typen en de vereisten.

**Versleuteling gebruiken.** Identificeer welke gebieden van de app versleuteling gebruiken, zoals voor gegevens opslag of client communicatie en toegang.

**Gebruik beveiligde kanalen.** Een beveiligd kanaal in de Clouds is essentieel voor het leveren van beveiligings-en verificatie controles, realtime-beveiliging, quarantaine en andere services in verschillende Clouds.

**Rollen definiëren en gebruiken.** Implementeer rollen voor resource configuraties en toegang met één identiteit in verschillende Clouds. Bepaal de vereisten voor op rollen gebaseerde toegangs beheer (RBAC) voor de app en de bijbehorende platform bronnen.

**Controleer uw systeem.** Systeem bewaking kan gegevens van zowel de app-onderdelen als de gerelateerde Cloud platform bewerkingen vastleggen en samen voegen.

## <a name="summary"></a>Samenvatting

In dit artikel vindt u een controle lijst met items die belang rijk zijn bij het ontwerpen en ontwerpen van uw hybride apps. Door deze pijlers te bekijken voordat u uw app implementeert, voor komt u dat u deze vragen in productie uitval kunt uitvoeren en kunt u uw ontwerp mogelijk opnieuw bezoeken.

Het lijkt altijd een tijdrovende taak, maar u kunt uw rendement op uw investering gemakkelijk terugkrijgen als u uw App ontwerpt op basis van deze pijlers.

## <a name="next-steps"></a>Volgende stappen

Zie de volgende bronnen voor meer informatie:

- [Hybride Cloud](https://azure.microsoft.com/overview/hybrid-cloud/)
- [Hybride Cloud-apps](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [Azure Resource Manager-sjablonen voor consistentie van de cloud ontwikkelen](https://aka.ms/consistency)
