---
title: Detectie patroon voor Footfall met behulp van Azure en Azure Stack hub
description: Meer informatie over het gebruik van Azure en Azure Stack hub voor het implementeren van een op AI gebaseerde Footfall detectie oplossing voor het analyseren van het verkeer van Retail Store.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910468"
---
# <a name="footfall-detection-pattern"></a>Footfall-detectie patroon

Dit patroon biedt een overzicht van het implementeren van een op AI gebaseerde Footfall detectie oplossing voor het analyseren van bezoekers verkeer in winkels. De oplossing genereert inzichten van praktijk acties, met behulp van Azure, Azure Stack hub en de Custom Vision AI dev kit.

## <a name="context-and-problem"></a>Context en probleem

Contoso-winkels willen graag inzicht krijgen in hoe klanten hun huidige producten ontvangen met betrekking tot de opslag indeling. Ze kunnen geen personeel in elke sectie plaatsen en het is inefficiënt om een team van analisten te laten beoordelen van de camera beelden van de hele winkel. Daarnaast hebben geen van hun winkels voldoende band breedte om video van al hun camera's naar de cloud te streamen voor analyse.

Contoso wil graag een onopvallende, privacy vriendelijke manier vinden om de demografische, loyaliteit en reacties van hun klanten te bepalen voor het opslaan van weer gaven en producten.

## <a name="solution"></a>Oplossing

In dit retail Analytics-patroon wordt gebruikgemaakt van een gelaagde benadering voor het afwijzen van de rand. Door gebruik te maken van de Custom Vision AI dev kit, worden alleen afbeeldingen met menselijke gezichten verzonden voor analyse naar een privé Azure Stack hub die Azure Cognitive Services uitvoert. Geanonimiseerd, geaggregeerde gegevens worden naar Azure verzonden voor aggregatie over alle archieven en visualisaties in Power BI. Door de rand en open bare Cloud te combi neren, kunnen we profiteren van de moderne AI-technologie, terwijl ze ook in overeenstemming zijn met hun bedrijfs beleid en de privacy van hun klanten eerbiedigen.

[![Oplossing voor Footfall-detectie patroon](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Hier volgt een overzicht van de werking van de oplossing:

1. De Custom Vision AI dev kit haalt een configuratie op van IoT Hub, waarmee de IoT Edge runtime en een ML-model worden geïnstalleerd.
2. Als het model een persoon ziet, neemt het een foto mee en uploadt deze naar Azure Stack hub-Blob-opslag.
3. De BLOB-service activeert een Azure-functie op Azure Stack hub.
4. De Azure-functie roept een container met de Face-API op om demografische-en Emotion-gegevens op te halen uit de installatie kopie.
5. De gegevens worden geanonimiseerd en verzonden naar een Azure Event Hubs-cluster.
6. Het Event Hubs cluster duwt de gegevens naar Stream Analytics.
7. Stream Analytics worden de gegevens geaggregeerd en naar Power BI gepusht.

## <a name="components"></a>Onderdelen

Deze oplossing maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| Hardware in de Store | [Custom Vision AI dev kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Biedt in-Store filtering met behulp van een lokaal ML-model dat alleen installatie kopieën van mensen vastlegt voor analyse. Veilig ingericht en bijgewerkt via IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs biedt een schaalbaar platform voor het opnemen van geanonimiseerd-gegevens die netjes worden geïntegreerd met Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Een Azure Stream Analytics taak aggregeert de geanonimiseerd-gegevens en groepeert deze in 15-Second Windows voor visualisatie. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI biedt een gebruiks vriendelijke dash board-interface voor het weer geven van de uitvoer van Azure Stream Analytics. |
| Azure Stack hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview.md) | De App Service Resource provider (RP) biedt een basis voor Edge-onderdelen, waaronder hosting-en beheer functies voor web-apps/Api's en functies. |
| | Azure Kubernetes service [(AKS) engine-](https://github.com/Azure/aks-engine) cluster | De AKS RP met AKS-engine-cluster dat is geïmplementeerd in Azure Stack hub biedt een schaal bare, robuuste engine voor het uitvoeren van de Face-API-container. |
| | Azure Cognitive Services [Face-API-containers](/azure/cognitive-services/face/face-how-to-install-containers)| De Azure Cognitive Services RP met Face-API-containers biedt demografische, Emotion en unieke detectie van bezoekers op het particuliere netwerk van contoso. |
| | Blob Storage | Installatie kopieën die zijn vastgelegd in de AI dev kit, worden geüpload naar de Blob-opslag van Azure Stack hub. |
| | Azure Functions | Een Azure-functie die wordt uitgevoerd op Azure Stack hub ontvangt invoer van Blob Storage en beheert de interacties met de Face-API. Het verzendt geanonimiseerd-gegevens naar een Event Hubs-cluster dat zich in azure bevindt.<br><br>|

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten wanneer u bepaalt hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

Als u wilt dat deze oplossing kan worden geschaald over meerdere camera's en locaties, moet u ervoor zorgen dat alle onderdelen de toegenomen belasting kunnen afhandelen. U moet mogelijk acties uitvoeren zoals:

- Verhoog het aantal Stream Analytics streaming-eenheden.
- Uitschalen van de Face-API-implementatie.
- Verhoog de Event Hubs cluster doorvoer.
- In uitzonderlijke gevallen kan het nodig zijn om van Azure Functions naar een virtuele machine te migreren.

### <a name="availability"></a>Beschikbaarheid

Omdat deze oplossing is gelaagd, is het belang rijk om te zien hoe u kunt omgaan met netwerk-of energie storingen. Afhankelijk van de behoeften van uw bedrijf wilt u mogelijk een mechanisme implementeren om installatie kopieën lokaal in de cache te plaatsen, waarna de Azure Stack-hub wordt doorgestuurd wanneer de verbinding wordt geretourneerd. Als de locatie groot genoeg is, is het implementeren van een Data Box Edge met de Face-API-container naar die locatie mogelijk een betere optie.

### <a name="manageability"></a>Beheerbaarheid

Deze oplossing kan veel apparaten en locaties omvatten, wat een onhandigheid kan oplevert. [De IOT-services van Azure](/azure/iot-fundamentals/) kunnen worden gebruikt om automatisch nieuwe locaties en apparaten online te brengen en deze up-to-date te houden.

### <a name="security"></a>Beveiliging

Met deze oplossing worden de installatie kopieën van klanten vastgelegd, waardoor de beveiliging een grootste overweging wordt. Zorg ervoor dat alle opslag accounts zijn beveiligd met het juiste toegangs beleid en om regel matig sleutels te draaien. Zorg ervoor dat opslag accounts en Event Hubs Bewaar beleid hebben dat voldoet aan de privacyinstellingen van de bedrijfs-en regerings wetgeving. Zorg er ook voor dat u het toegangs niveau van de gebruiker laagt. Met lagen zorgt u ervoor dat gebruikers alleen toegang hebben tot de gegevens die ze nodig hebben voor hun rol.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:

- Zie het [patroon gelaagde gegevens](https://aka.ms/tiereddatadeploy), dat wordt gebruikt door het detectie patroon Footfall.
- Raadpleeg de [Custom Vision AI dev kit](https://azure.github.io/Vision-AI-DevKit-Pages/) voor meer informatie over het gebruik van aangepaste visie. 

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [implementatie handleiding voor Footfall-detectie](solution-deployment-guide-retail-footfall-detection.md). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.
