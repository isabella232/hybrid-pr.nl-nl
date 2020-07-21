---
title: Het DevOps-patroon in Azure Stack hub
description: Meer informatie over het DevOps-patroon, zodat u consistentie kunt garanderen tussen implementaties in Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: e26056a9507a7467473b009725d4f210d9d59ec8
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477232"
---
# <a name="devops-pattern"></a>DevOps patroon

Code vanaf één locatie en implementeren in meerdere doelen in ontwikkel-, test-en productie omgevingen die zich in uw lokale Data Center, persoonlijke Clouds of open bare cloud bevinden.

## <a name="context-and-problem"></a>Context en probleem

Continuïteit van de toepassings implementatie, beveiliging en betrouw baarheid zijn essentieel voor organisaties en essentieel voor ontwikkel teams.

Apps vereisen vaak herstructured code om in elke doel omgeving te worden uitgevoerd. Dit betekent dat een app niet volledig draagbaar is. Het moet worden bijgewerkt, getest en gevalideerd terwijl het door elke omgeving heen wordt verplaatst. Code die is geschreven in een ontwikkel omgeving moet vervolgens worden herschreven om te werken in een test omgeving en opnieuw te worden geschreven wanneer het uiteindelijk in een productie omgeving gaat. Daarnaast is deze code specifiek gebonden aan de host. Dit verhoogt de kosten en complexiteit van het onderhouden van uw app. Elke versie van de app is gekoppeld aan elke omgeving. Dankzij de verbeterde complexiteit en duplicatie wordt het risico van beveiliging en code kwaliteit verhoogd. Daarnaast kan de code niet direct opnieuw worden geïmplementeerd wanneer u herstel hosts herstelt of extra hosts implementeert om toename in de vraag te verwerken.

## <a name="solution"></a>Oplossing

Met het DevOps-patroon kunt u een app bouwen, testen en implementeren die wordt uitgevoerd op meerdere clouds. Dit patroon is de praktijk eenheid voor continue integratie en continue levering. Met doorlopende integratie wordt code gebouwd en getest wanneer een teamlid een wijziging doorvoert in versie beheer. Continue levering automatiseert elke stap van een build naar een productie omgeving. Samen maken deze processen een release proces dat ondersteuning biedt voor implementatie in diverse omgevingen. Met dit patroon kunt u uw code ontwerpen en vervolgens dezelfde code implementeren in een lokale omgeving, verschillende privécloud en open bare Clouds. Verschillen in de omgeving vereisen een wijziging in een configuratie bestand in plaats van wijzigingen in de code.

![DevOps patroon](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Met een consistente set ontwikkel tools voor on-premises, privécloud en open bare Cloud omgevingen kunt u een praktijk van continue integratie en continue levering implementeren. Apps en services die zijn geïmplementeerd met behulp van het DevOps-patroon zijn uitwisselbaar en kunnen op elk van deze locaties worden uitgevoerd, waarbij gebruik wordt gemaakt van on-premises en open bare Cloud functies en-mogelijkheden.

Met een DevOps-release pijplijn kunt u het volgende doen:

- Start een nieuwe build op basis van code doorvoeren aan één opslag plaats.
- Implementeer automatisch uw zojuist opgebouwde code naar de open bare Cloud voor acceptatie tests door gebruikers.
- Automatisch implementeren in een privécloud nadat uw code is getest.

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Het DevOps-patroon is bedoeld om consistentie tussen implementaties te garanderen, ongeacht de doel omgeving. De mogelijkheden verschillen echter in de Cloud-en on-premises omgevingen. Houd rekening met de volgende punten:

- Zijn de functies, eind punten, services en andere resources in uw implementatie beschikbaar op de doel implementatie locaties?
- Zijn er configuratie-artefacten opgeslagen op locaties die toegankelijk zijn voor verschillende Clouds?
- Werken de implementatie parameters in alle doel omgevingen?
- Zijn er resource-specifieke eigenschappen beschikbaar in alle doel-Clouds?

Zie [Azure Resource Manager sjablonen voor Cloud consistentie ontwikkelen](/azure/azure-resource-manager/templates-cloud-consistency)voor meer informatie.

Houd ook rekening met de volgende punten wanneer u bepaalt hoe u dit patroon wilt implementeren:

### <a name="scalability"></a>Schaalbaarheid

Implementatie automatiserings systemen zijn het belangrijkste besturings punt in de DevOps-patronen. Implementaties kunnen variëren. De selectie van de juiste server grootte is afhankelijk van de grootte van de verwachte werk belasting. Vm's kosten meer om te schalen dan containers. Uw build-proces moet echter worden uitgevoerd met containers om containers te gebruiken voor schalen.

### <a name="availability"></a>Beschikbaarheid

Beschik baarheid in de context van de DevPattern houdt in dat status gegevens kunnen worden hersteld die zijn gekoppeld aan uw werk stroom, zoals test resultaten, code afhankelijkheden of andere artefacten. Overweeg twee algemene metrische gegevens als u uw beschikbaarheidsvereisten wilt beoordelen:

- Beoogde herstel tijd (RTO) geeft aan hoe lang u zonder systeem kunt gaan.

- Beoogd herstel punt (RPO) geeft aan hoeveel gegevens u kunt verliezen als een onderbreking in de service van invloed is op het systeem.

In de praktijk, RTO en RPO is redundantie en back-up te impliceren. Op de wereld wijde Azure-Cloud is beschik baarheid geen vraag over het herstel van hardware. Dit is een onderdeel van Azure, maar u kunt de status van uw DevOps-systemen behouden. Op Azure Stack hub is het mogelijk dat het herstel van hardware wordt overwogen.

Een andere belang rijke overweging bij het ontwerpen van het systeem dat wordt gebruikt voor implementatie automatisering is toegangs beheer en het juiste beheer van de benodigde rechten voor het implementeren van services in Cloud omgevingen. Welke rechten zijn er nodig om implementaties te maken, te verwijderen of te wijzigen? Zo is bijvoorbeeld één set rechten vereist voor het maken van een resource groep in Azure en een andere voor het implementeren van services in de resource groep.

### <a name="manageability"></a>Beheerbaarheid

Het ontwerp van elk systeem op basis van het DevOps-patroon moet rekening houden met automatisering, logboek registratie en waarschuwingen voor elke service in de Port Folio. Gebruik gedeelde services, een toepassings team of beide, en houd ook het beveiligings beleid en governance bij.

Implementeer productie omgevingen en ontwikkel-en test omgevingen in afzonderlijke resource groepen op Azure of Azure Stack hub. Vervolgens kunt u de resources van elke omgeving bewaken en de facturerings kosten per resource groep totaliseren. U kunt ook resources als een set verwijderen. Dit is handig voor test implementaties.

## <a name="when-to-use-this-pattern"></a>Wanneer dit patroon gebruiken

Gebruik dit patroon als:

- U kunt code ontwikkelen in één omgeving die voldoet aan de behoeften van uw ontwikkel aars en implementeren in een omgeving die specifiek is voor uw oplossing, waar het lastig kan zijn om nieuwe code te ontwikkelen.
- U kunt de code en hulpprogram ma's van uw ontwikkel aars gebruiken, zolang ze de continue integratie en continue levering van het proces in het DevOps-patroon kunnen volgen.

Dit patroon wordt niet aanbevolen voor het volgende:

- Als u de infra structuur niet kunt automatiseren, dient u bronnen, configuratie-, identiteits-en beveiligings taken in te richten.
- Als teams geen toegang hebben tot hybride cloud resources om een continue integratie/doorlopende ontwikkeling (CI/CD)-benadering te implementeren.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:

- Raadpleeg de [documentatie van Azure DevOps](/azure/devops) voor meer informatie over Azure DevOps en gerelateerde hulpprogram ma's, waaronder Azure opslag plaatsen en Azure-pijp lijnen.
- Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [implementatie handleiding voor hybride CI/cd-oplossingen van DevOps](https://aka.ms/hybriddevopsdeploy). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen. U leert hoe u een app implementeert in Azure en Azure Stack hub met behulp van een pipeline met hybride doorlopende integratie/continue levering (CI/CD).
