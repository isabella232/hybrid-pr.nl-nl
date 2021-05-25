---
title: Niet-voorraaddetectie met Behulp van Azure en Azure Stack Edge
description: Meer informatie over het gebruik van Azure en Azure Stack Edge om buiten voorraaddetectie te implementeren.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343872"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Niet-voorraaddetectie aan de randpatroon

Dit patroon laat zien hoe u kunt bepalen of schappen geen voorraad meer hebben met behulp van een Azure Stack Edge of Azure IoT Edge apparaat- en netwerkcamera's.

## <a name="context-and-problem"></a>Context en probleem

Fysieke winkels verliezen verkoop, omdat wanneer klanten naar een artikel zoeken, het niet op de schappen aanwezig is. Het item kan zich echter achter in de winkel hebben bewaard en niet opnieuw zijn aangevuld. Winkels willen hun personeel efficiënter gebruiken en automatisch een melding ontvangen wanneer items moeten worden aangevuld.

## <a name="solution"></a>Oplossing

In het voorbeeld van de oplossing wordt een edge-apparaat gebruikt, zoals een Azure Stack Edge in elk winkel, waarmee gegevens van camera's in de winkel efficiënt worden verwerkt. Met dit geoptimaliseerde ontwerp kunnen winkels alleen relevante gebeurtenissen en afbeeldingen naar de cloud verzenden. Het ontwerp bespaart bandbreedte, opslagruimte en garandeert de privacy van klanten. Wanneer frames van elke camera worden gelezen, verwerkt een ML-model de afbeelding en retourneert het eventuele uit voorraadgebieden. De afbeelding en buiten voorraadgebieden worden weergegeven in een lokale web-app. Deze gegevens kunnen worden verzonden naar een Time Series Insight-omgeving om inzichten in uw Power BI.

![Niet op voorraad bij edge-oplossingsarchitectuur](media/pattern-out-of-stock-at-edge/solution-architecture.png)

De oplossing werkt als volgende:

1. Afbeeldingen worden vastgelegd vanaf een netwerkcamera via HTTP of RTSP.
2. De afbeelding wordt van het ene naar het deferentie-stuurprogramma verzonden, dat communiceert met het ML-model om te bepalen of er geen afbeeldingen op voorraad zijn.
3. Het ML-model retourneert alle uit voorraadgebieden.
4. Met het deferencingst stuurprogramma wordt de onbewerkte afbeelding geüpload naar een blob (indien opgegeven) en worden de resultaten van het model naar Azure IoT Hub en een begrensvakprocessor op het apparaat verzendt.
5. Met de begrenzendvakprocessor worden begrenzen aan de afbeelding toegevoegd en wordt het pad naar de afbeelding in een in-memory database.
6. De web-app vraagt naar afbeeldingen en geeft deze weer in de ontvangen volgorde.
7. Berichten van IoT Hub worden geaggregeerd in Time Series Insights.
8. Power BI geeft een interactief rapport weer van niet-voorraaditems gedurende een periode met de gegevens uit Time Series Insights.


## <a name="components"></a>Onderdelen

Deze oplossing maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| On-premises hardware | Netwerkcamera | Een netwerkcamera is vereist, met een HTTP- of RTSP-feed om de afbeeldingen voor de deferentie op te geven. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) verwerkt het inrichten en verzenden van berichten voor de edge-apparaten. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) worden de berichten uit de IoT Hub op voor visualisatie. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) bedrijfsgerichte rapporten over niet-aandelengebeurtenissen. Power BI biedt een gebruiksvriendelijke dashboardinterface voor het weergeven van de uitvoer van Azure Stream Analytics. |
| Azure Stack Edge of<br>Azure IoT Edge apparaat | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) de runtime voor de on-premises containers en verwerkt apparaatbeheer en updates.|
| | Azure-project brainwave | Op een Azure Stack Edge gebruikt [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) Field-Programmable Gate Arrays (FPGA's) om ML-deferencing te versnellen.|

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten bij het bepalen hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

De machine learning modellen kunnen alleen worden uitgevoerd met een bepaald aantal frames per seconde, afhankelijk van de opgegeven hardware. Bepaal de optimale steekproeffrequentie van uw camera(s) om ervoor te zorgen dat er geen back-up wordt van de ML-pijplijn. Verschillende typen hardware verwerken verschillende aantallen camera's en framesnelheden.

### <a name="availability"></a>Beschikbaarheid

Het is belangrijk om te overwegen wat er kan gebeuren als de connectiviteit van het edge-apparaat wordt verliest. Bedenk welke gegevens mogelijk verloren gaan in het Time Series Insights en Power BI dashboard. De opgegeven voorbeeldoplossing is niet ontworpen om zeer beschikbaar te zijn.

### <a name="manageability"></a>Beheerbaarheid

Deze oplossing kan veel apparaten en locaties overspannen, wat onprakkig kan worden. De IoT-services van Azure kunnen automatisch nieuwe locaties en apparaten online brengen en up-to-date houden. De juiste procedures voor gegevensbeheer moeten ook worden gevolgd.

### <a name="security"></a>Beveiliging

Met dit patroon worden mogelijk gevoelige gegevens verwerkt. Zorg ervoor dat sleutels regelmatig worden geroteerd en dat de machtigingen voor het Azure Storage account en lokale shares correct zijn ingesteld.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over onderwerpen die in dit artikel zijn geïntroduceerd:
- In dit patroon worden meerdere IoT-gerelateerde services gebruikt, Azure IoT Edge [,](/azure/iot-edge/) [Azure IoT Hub](/azure/iot-hub/)en [Azure Time Series Insights.](/azure/time-series-insights/)
- Zie voor meer informatie over [](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) Microsoft Project Brainwave de blogaankondiging en bekijk de Video over [versnelde Azure Machine Learning met Project Brainwave.](https://www.youtube.com/watch?v=DJfMobMjCX0)
- Zie [Ontwerpoverwegingen voor hybride apps](overview-app-design-considerations.md) voor meer informatie over best practices en voor antwoorden op aanvullende vragen.
- Zie de [Azure Stack producten en oplossingen](/azure-stack) voor meer informatie over het hele portfolio met producten en oplossingen.

Wanneer u klaar bent om het oplossingsvoorbeeld te testen, gaat u verder met de implementatiehandleiding voor de [Edge ML-deferencingoplossing.](https://aka.ms/edgeinferencingdeploy) De implementatiehandleiding bevat stapsgewijs instructies voor het implementeren en testen van de onderdelen.
