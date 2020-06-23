---
title: Gelaagde gegevens voor het analyse patroon met Azure en Azure Stack hub
description: Leer hoe u Azure en Azure Stack hub kunt gebruiken om een gelaagde gegevens oplossing in de hybride Cloud te implementeren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910476"
---
# <a name="tiered-data-for-analytics-pattern"></a>Patroon voor gelaagde gegevens voor analyse

Dit patroon illustreert hoe u Azure Stack hub en Azure kunt gebruiken om gegevens in meerdere on-premises en Cloud locaties te stage, te analyseren, te verwerken, op te schonen en op te slaan.

## <a name="context-and-problem"></a>Context en probleem

Een van de problemen die zich voordoen bij bedrijfs organisaties in de moderne technologie, is voor de beveiliging van gegevens opslag, verwerking en analyse. Overwegingen zijn onder andere:

- gegevens inhoud
- location
- beveiligings-en privacy-vereisten
- toegangs machtigingen
- onderhoud
- opslag magazijn

Azure is in combi natie met Azure Stack hub gericht op gegevens en biedt voordelige oplossingen. Deze oplossing wordt het beste uitgedrukt in een gedistribueerde productie-of logistiek bedrijf.

De oplossing is gebaseerd op het volgende scenario:

- Een grote productie organisatie met meerdere branches.
- Gegevens opslag,-verwerking en-distributie tussen wereld wijde externe locaties en de centrale hoofd vestiging zijn vereist.
- Activiteiten van werk nemers en machines, faciliteit informatie en zakelijke rapporten die veilig moeten blijven. De gegevens moeten op de juiste wijze worden gedistribueerd en voldoen aan het regionale nalevings beleid en de industriële voor Schriften.

## <a name="solution"></a>Oplossing

Het gebruik van on-premises en open bare Cloud omgevingen voldoet aan de behoeften van bedrijven met meerdere faciliteiten. Azure Stack hub biedt een snelle, veilige en flexibele oplossing voor het verzamelen, verwerken, opslaan en distribueren van lokale en externe gegevens. Dit patroon is vooral nuttig wanneer de beveiliging, vertrouwelijkheid, bedrijfs beleid en wettelijke vereisten kunnen verschillen tussen locaties en gebruikers.

![Gelaagde gegevens patroon voor analyse van oplossingen](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Onderdelen

Dit patroon maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| Azure | Storage | Een [Azure Storage](/azure/storage/) account voorziet in een steriel data verbruik-eind punt. Azure Storage is Microsoft's cloudoplossing bedoeld voor scenario's voor gegevensopslag. Azure Storage biedt een zeer schaal bare object opslag voor gegevens objecten en een File System-Service voor de Cloud. Het biedt ook een berichten Archief voor betrouw bare berichten en een NoSQL-archief. |
| Azure Stack hub | Storage | Een [Azure stack hub Storage](/azure-stack/user/azure-stack-storage-overview) -account wordt gebruikt voor meerdere services:<br><br>- **Blob-opslag** voor onbewerkte gegevens opslag. Blob-opslag kan elk type tekst of binaire gegevens bevatten, zoals een document, media bestand of app-installatie programma. Elke blob is ingedeeld onder een container. Containers bieden een handige manier om beveiligings beleid toe te wijzen aan groepen objecten. Een opslag account kan een wille keurig aantal containers bevatten en een container kan een wille keurig aantal blobs bevatten, tot de capaciteits limiet van 500 TB van het opslag account.<br>- **Blob-opslag** voor gegevens archief. Er zijn voor delen van goedkope opslag voor het koelen van gegevens archivering. Voor beelden van leuke gegevens zijn back-ups, media-inhoud, weten schappelijke gegevens, naleving en archiverings gegevens. Over het algemeen worden alle gegevens die zelden worden gebruikt, beschouwd als koud opslag. Gegevens lagen op basis van kenmerken, zoals de frequentie van toegang en de Bewaar periode. Klant gegevens zijn niet regel matig toegankelijk, maar vereisen vergelijk bare latentie en prestaties voor dynamische gegevens.<br>- **Wachtrij opslag** voor verwerkte gegevens opslag. Queue Storage biedt Cloud berichten tussen app-onderdelen. Bij het ontwerpen van apps voor schaal worden app-onderdelen vaak losgekoppeld zodat ze onafhankelijk kunnen worden geschaald. Queue Storage biedt asynchrone berichten voor communicatie tussen app-onderdelen, ongeacht of ze worden uitgevoerd in de Cloud, op het bureau blad, op een on-premises server of op een mobiel apparaat. Queue Storage biedt daarnaast ondersteuning voor het beheren van asynchrone taken en het samenstellen van proceswerkstromen. |
| | Azure Functions | De [Azure functions](/azure/azure-functions/) -service wordt verzorgd door de Azure app service van de resource provider [Azure stack hub](/azure-stack/operator/azure-stack-app-service-overview) . Met Azure Functions kunt u uw code in een eenvoudige, serverloze omgeving uitvoeren als reactie op verschillende gebeurtenissen. Azure Functions schaal om aan de vraag te voldoen zonder een virtuele machine te maken of een web-app te publiceren, met behulp van de programmeer taal van uw keuze. Functies worden gebruikt door de oplossing voor:<br><br>- **Gegevens innemen**<br>- **Gegevens sterilisatie.** Hand matig geactiveerde functies kunnen geplande gegevens verwerking uitvoeren, opschonen en archiveren. Voor beelden zijn de klant lijst reiniging en maandelijkse rapport verwerking.|

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten wanneer u bepaalt hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

Azure Functions-en opslag oplossingen worden geschaald om te voldoen aan de gegevens volumes en de verwerkings vereisten. Zie [Azure Storage scaling-documentatie](/azure/storage/common/storage-scalability-targets)voor informatie over de schaal baarheid van Azure.

### <a name="availability"></a>Beschikbaarheid

Opslag is de primaire Beschik baarheid voor dit patroon. Verbinding via snelle koppelingen is vereist voor het verwerken en distribueren van grote hoeveel heden gegevens.

### <a name="manageability"></a>Beheerbaarheid

Beheer baarheid van deze oplossing is afhankelijk van de hulpprogram ma's voor ontwerpen in gebruik en betrokkenheid van broncode beheer.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:

- Raadpleeg de documentatie over [Azure Storage](/azure/storage/) en [Azure functions](/azure/azure-functions/) . Dit patroon maakt intensief gebruik van Azure Storage accounts en Azure Functions op zowel Azure als Azure Stack hub.
- Zie [overwegingen voor het ontwerpen van hybride toepassingen](overview-app-design-considerations.md) voor meer informatie over aanbevolen procedures en voor antwoorden op aanvullende vragen.
- Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [gelaagde gegevens voor de implementatie handleiding voor analyse oplossingen](https://aka.ms/tiereddatadeploy). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.
