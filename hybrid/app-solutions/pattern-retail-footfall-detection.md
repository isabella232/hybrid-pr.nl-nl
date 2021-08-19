---
title: Detectiepatroon voor voetval met behulp van Azure en Azure Stack Hub
description: Meer informatie over het gebruik van Azure en Azure Stack Hub om een op AI gebaseerde oplossing voor voetvaldetectie te implementeren voor het analyseren van het verkeer van winkels.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 79fb39d418bed53ef6a78980fcd9188bdf6e57ae
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281275"
---
# <a name="footfall-detection-pattern"></a>Detectiepatroon voetval

Dit patroon biedt een overzicht voor het implementeren van een op AI gebaseerde oplossing voor voetvaldetectie voor het analyseren van bezoekersverkeer in winkels. De oplossing genereert inzichten op basis van acties uit de echte wereld, met behulp van Azure, Azure Stack Hub en de Custom Vision AI Dev Kit.

## <a name="context-and-problem"></a>Context en probleem

Contoso Stores wil graag inzicht krijgen in hoe klanten hun huidige producten ontvangen met betrekking tot de winkelindeling. Ze kunnen geen personeel in elke sectie plaatsen en het is inefficiënt om een team analisten de camerabeelden van een hele winkel te laten bekijken. Bovendien hebben geen van hun winkels voldoende bandbreedte om video van al hun camera's naar de cloud te streamen voor analyse.

Contoso wil een niet-ingrijpende, privacyvriendelijke manier vinden om de demografische gegevens, deloyaliteit en reacties van hun klanten te bepalen voor het opslaan van beeldschermen en producten.

## <a name="solution"></a>Oplossing

Dit patroon voor retailanalyse maakt gebruik van een gelaagde benadering voor de deferencing aan de rand. Met behulp van Custom Vision AI Dev Kit worden alleen afbeeldingen met menselijke gezichten verzonden voor analyse naar een privé-Azure Stack Hub die Azure Cognitive Services. Geanonimiseerde, geaggregeerde gegevens worden naar Azure verzonden voor aggregatie in alle winkels en visualisaties in Power BI. Door de rand en openbare cloud te combineren, kan Contoso profiteren van moderne AI-technologie en tegelijkertijd voldoen aan hun bedrijfsbeleid en de privacy van hun klanten respecteren.

[![Oplossing voor detectie van voetvalpatronen](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Hier volgt een samenvatting van de manier waarop de oplossing werkt:

1. De Custom Vision AI Dev Kit krijgt een configuratie van IoT Hub, waarmee de IoT Edge Runtime en een ML geïnstalleerd.
2. Als het model een persoon ziet, wordt er een foto gemaakt en geüpload naar Azure Stack Hub blobopslag.
3. De blobservice activeert een Azure-functie op Azure Stack Hub.
4. De Azure-functie roept een container aan met de Face-API om demografische en emotiegegevens op te halen uit de afbeelding.
5. De gegevens worden geanonimiseerd en verzonden naar Azure Event Hubs cluster.
6. Het Event Hubs cluster pusht de gegevens naar Stream Analytics.
7. Stream Analytics aggregeert de gegevens en pusht deze naar Power BI.

## <a name="components"></a>Onderdelen

Deze oplossing maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| In-store hardware | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Biedt in-store filtering met behulp van een lokaal ML model dat alleen afbeeldingen van personen vast legt voor analyse. Veilig ingericht en bijgewerkt via IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs biedt een schaalbaar platform voor het opnemen van geanonimiseerde gegevens die netjes zijn geïntegreerd met Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Een Azure Stream Analytics geaggregeerde gegevens worden geaggregeerd en in 15 seconden voor visualisatie in groepen opgenomen. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI biedt een gebruiksvriendelijke dashboardinterface voor het weergeven van de uitvoer van Azure Stream Analytics. |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | De App Service resourceprovider (RP) biedt een basis voor edge-onderdelen, waaronder hosting- en beheerfuncties voor web-apps/API's en Functions. |
| | Azure Kubernetes Service [enginecluster (AKS)](https://github.com/Azure/aks-engine) | De AKS RP AKS-Engine cluster geïmplementeerd in Azure Stack Hub biedt een schaalbare, robuuste engine voor het uitvoeren van de Face-API-container. |
| | Azure Cognitive Services [Face-API-containers](/azure/cognitive-services/face/face-how-to-install-containers)| De Azure Cognitive Services RP met Face-API-containers biedt demografische, emotie- en unieke bezoekersdetectie op het particuliere netwerk van Contoso. |
| | Blob Storage | Afbeeldingen die zijn vastgelegd vanuit de AI Dev Kit, worden geüpload naar Azure Stack Hub blobopslag van uw computer. |
| | Azure Functions | Een Azure-functie die wordt uitgevoerd Azure Stack Hub ontvangt invoer uit blobopslag en beheert de interacties met de Face-API. Er worden geanonimiseerde gegevens naar een Event Hubs cluster in Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten bij het bepalen hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

Als u wilt dat deze oplossing kan worden geschaald op meerdere camera's en locaties, moet u ervoor zorgen dat alle onderdelen de toegenomen belasting kunnen verwerken. Mogelijk moet u acties uitvoeren zoals:

- Verhoog het aantal Stream Analytics streaming-eenheden.
- Schaal de Face-API-implementatie uit.
- Verhoog Event Hubs clusterdoorvoer.
- In extreme gevallen kan het nodig zijn Azure Functions naar een virtuele machine te migreren.

### <a name="availability"></a>Beschikbaarheid

Omdat deze oplossing gelaagd is, is het belangrijk om na te denken over het omgaan met netwerk- of stroomstoringen. Afhankelijk van de bedrijfsbehoeften wilt u mogelijk een mechanisme implementeren om afbeeldingen lokaal in de cache op te nemen en vervolgens door te Azure Stack Hub wanneer de verbinding wordt teruggestuurd. Als de locatie groot genoeg is, is het implementeren van een Data Box Edge met de Face-API-container op die locatie mogelijk een betere optie.

### <a name="manageability"></a>Beheerbaarheid

Deze oplossing kan veel apparaten en locaties overspannen, wat onprakbaar kan worden. [De IoT-services van Azure](/azure/iot-fundamentals/) kunnen worden gebruikt om automatisch nieuwe locaties en apparaten online te brengen en ze up-to-date te houden.

### <a name="security"></a>Beveiliging

Deze oplossing legt klantafbeeldingen vast, waardoor beveiliging een belangrijke overweging is. Zorg ervoor dat alle opslagaccounts zijn beveiligd met het juiste toegangsbeleid en regelmatig sleutels roteren. Zorg ervoor dat opslagaccounts en Event Hubs bewaarbeleid hebben dat voldoet aan de privacyregels van het bedrijf en de overheid. Zorg er ook voor dat u de toegangsniveaus van de gebruiker in een laag op een laag oplaagt. Lagen zorgen ervoor dat gebruikers alleen toegang hebben tot de gegevens die ze nodig hebben voor hun rol.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel zijn geïntroduceerd:

- Zie het [patroon Gelaagde gegevens,](https://aka.ms/tiereddatadeploy)dat wordt gebruikt door het detectiepatroon voetval.
- Zie de [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) voor meer informatie over het gebruik van Custom Vision. 

Wanneer u klaar bent om het oplossingsvoorbeeld te testen, gaat u verder met de [implementatiehandleiding voor voetvaldetectie.](/azure/architecture/hybrid/deployments/solution-deployment-guide-retail-footfall-detection) De implementatiehandleiding bevat stapsgewijs instructies voor het implementeren en testen van de onderdelen.