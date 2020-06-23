---
title: Geen voorraad detectie met Azure en Azure Stack Edge
description: Meer informatie over hoe u Azure en Azure Stack Edge-Services kunt gebruiken om de detectie van de voor Raad uit te voeren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910451"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Geen voorraad detectie op het rand patroon

Dit patroon illustreert hoe u kunt bepalen of rekken met een Azure Stack rand of Azure IoT Edge apparaat en netwerk camera's geen voorraad items bevatten.

## <a name="context-and-problem"></a>Context en probleem

De verkoop van fysieke winkels gaat verloren omdat wanneer klanten een item zoeken, het niet aanwezig is op het schap. Het item kan echter aanwezig zijn in de back-up van de Store en niet opnieuw worden opgeslagen. Winkels willen hun mede werkers efficiënter gebruiken en automatisch op de hoogte worden gesteld wanneer items opnieuw moeten worden opgeslagen.

## <a name="solution"></a>Oplossing

In het voor beeld van de oplossing wordt een edge-apparaat gebruikt, zoals een Azure Stack rand in elke Store, waarmee gegevens efficiënt worden verwerkt van camera's in de Store. Met dit geoptimaliseerde ontwerp kunnen winkels alleen relevante gebeurtenissen en installatie kopieën verzenden naar de Cloud. Het ontwerp bespaart band breedte, opslag ruimte en zorgt voor de privacy van klanten. Als frames van elke camera worden gelezen, wordt de installatie kopie door een ML-model verwerkt en worden er geen voorraad gebieden weer gegeven. De afbeeldings-en buiten-voorraad gebieden worden weer gegeven op een lokale web-app. Deze gegevens kunnen naar een time series Insight-omgeving worden verzonden om inzicht in Power BI weer te geven.

![Niet-voorradige oplossings architectuur](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Hier volgt een beschrijving van de werking van de oplossing:

1. Installatie kopieën worden vastgelegd van een netwerk camera over HTTP of RTSP.
2. De grootte van de installatie kopie wordt gewijzigd en verzonden naar het stuur programma voor ingrijpen, dat communiceert met het ML-model om te bepalen of er niet-voorraad afbeeldingen zijn.
3. Het model ML retourneert een van de voorraad gebieden.
4. Het stuur programma voor delooiing uploadt de onbewerkte installatie kopie naar een BLOB (indien opgegeven) en verstuurt de resultaten van het model naar Azure IoT Hub en een selectie kader processor op het apparaat.
5. De selectie kader processor voegt selectie kaders toe aan de afbeelding en slaat het pad naar de afbeelding op in een in-memory database.
6. De web-app zoekt naar installatie kopieën en geeft deze weer in de ontvangen volg orde.
7. Berichten van IoT Hub worden geaggregeerd in Time Series Insights.
8. In Power BI wordt een interactief rapport weer gegeven van items die gedurende een bepaalde periode in de loop van de tijd vallen met de gegevens van Time Series Insights.


## <a name="components"></a>Onderdelen

Deze oplossing maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| On-premises hardware | Netwerk camera | Er is een netwerk camera vereist, met een HTTP-of RTSP-feed om de installatie kopieën op te geven. |
| Azure | Azure IoT Hub | [Azure IOT hub](/azure/iot-hub/) verwerkt apparaat-inrichting en-berichten voor de apparaten van de rand. |
|  | Azure Time Series Insights | [Azure time series Insights](/azure/time-series-insights/) worden de berichten van IOT hub opgeslagen voor visualisatie. |
|  | Power BI | [Micro soft power bi](https://powerbi.microsoft.com/) voorziet in zakelijke rapporten over gebeurtenissen die zich niet in de voor Raad bevinden. Power BI biedt een gebruiks vriendelijke dash board-interface voor het weer geven van de uitvoer van Azure Stream Analytics. |
| Azure Stack rand of<br>Azure IoT Edge apparaat | Azure IoT Edge | [Azure IOT Edge](/azure/iot-edge/) organiseert de runtime voor on-premises containers en beheert het beheer en de updates van apparaten.|
| | Azure project brainwave | Op een Azure Stack edge-apparaat gebruikt [project brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) veld-Programmeer bare poort matrices (fpga's) om ml te versnellen.|

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten wanneer u bepaalt hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

De meeste machine learning modellen kunnen alleen worden uitgevoerd op een bepaald aantal frames per seconde, afhankelijk van de beschik bare hardware. Bepaal de optimale sampling frequentie van uw camera (s) om ervoor te zorgen dat er geen back-up wordt gemaakt van de ML-pijp lijn. Verschillende typen hardware verwerken verschillende aantallen camera's en frame snelheden.

### <a name="availability"></a>Beschikbaarheid

Het is belang rijk om te bepalen wat er kan gebeuren als de verbinding met het edge-apparaat is verbroken. Houd er rekening mee dat de gegevens van het Time Series Insights en Power BI dash board verloren kunnen gaan. De voorziene voorbeeld oplossing is niet ontworpen om Maxi maal beschikbaar te zijn.

### <a name="manageability"></a>Beheerbaarheid

Deze oplossing kan veel apparaten en locaties omvatten, wat een onhandigheid kan oplevert. Met de IoT-services van Azure kunnen nieuwe locaties en apparaten automatisch online worden gebracht en up-to-date blijven. De juiste procedures voor gegevens beheer moeten ook worden gevolgd.

### <a name="security"></a>Beveiliging

Met dit patroon worden mogelijk gevoelige gegevens verwerkt. Zorg ervoor dat sleutels regel matig worden geroteerd en dat de machtigingen voor het Azure Storage account en lokale shares correct zijn ingesteld.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:
- Er worden meerdere IoT-gerelateerde services gebruikt in dit patroon, waaronder [Azure IOT Edge](/azure/iot-edge/), [Azure IOT hub](/azure/iot-hub/)en [Azure time series Insights](/azure/time-series-insights/).
- Voor meer informatie over micro soft project brainwave raadpleegt u [de aankondiging van de blog](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) en uitchecken [van de Azure-versnelde machine learning met project brainwave-video](https://www.youtube.com/watch?v=DJfMobMjCX0).
- Bekijk de overwegingen voor het [ontwerpen van hybride apps](overview-app-design-considerations.md) voor meer informatie over aanbevolen procedures en voor het verkrijgen van antwoorden op eventuele aanvullende vragen.
- Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [gelaagde gegevens voor de implementatie handleiding voor analyse oplossingen](https://aka.ms/edgeinferencingdeploy). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.
