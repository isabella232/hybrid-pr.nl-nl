---
title: Op AI gebaseerde Footfall-detectie oplossing implementeren in Azure en Azure Stack hub
description: Meer informatie over het implementeren van een op AI gebaseerde Footfall detectie oplossing voor het analyseren van bezoekers verkeer in winkels met Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: caedbd4758b9ae8c93cf9bb625ed9aac68bfa196
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895359"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Een op AI gebaseerde Footfall-detectie oplossing implementeren met behulp van Azure en Azure Stack hub

In dit artikel wordt beschreven hoe u een op AI gebaseerde oplossing implementeert die inzichten genereert op basis van Real-World-acties met behulp van Azure, Azure Stack hub en de Custom Vision AI dev kit.

In deze oplossing leert u het volgende:

> [!div class="checklist"]
> - Implementeer native Cloud Application bundels (CNAB) aan de rand. 
> - Implementeer een app die de Cloud grenzen omspant.
> - Gebruik de Custom Vision AI dev kit voor de rand.

> [!Tip]  
> ![Diagram hybride pijlers](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

## <a name="prerequisites"></a>Vereisten

Voordat u aan de slag gaat met deze implementatie handleiding, moet u het volgende doen:

- Bekijk het onderwerp [Footfall-detectie patroon](pattern-retail-footfall-detection.md) .
- Gebruikers toegang verkrijgen tot een met Azure Stack Development Kit (ASDK) of Azure Stack hub geïntegreerde systeem instantie, met:
  - De [Azure app service op Azure stack hub-resource provider](/azure-stack/operator/azure-stack-app-service-overview) geïnstalleerd. U hebt operator toegang tot uw Azure Stack hub-exemplaar nodig of u kunt samen werken met de beheerder om deze te installeren.
  - Een abonnement op een aanbieding die App Service en opslag limiet biedt. U hebt operator toegang nodig om een aanbieding te maken.
- Toegang verkrijgen tot een Azure-abonnement.
  - Als u nog geen abonnement op Azure hebt, Meld u dan aan voor een [gratis proef account](https://azure.microsoft.com/free/) voordat u begint.
- Maak twee service-principals in uw Directory:
  - Eén ingesteld voor gebruik met Azure-resources, met toegang tot het Azure-abonnements bereik.
  - Er is een set ingesteld voor gebruik met Azure Stack hub-resources, met toegang op het Azure Stack hub-abonnements bereik.
  - Zie [een app-identiteit gebruiken voor toegang tot resources](/azure-stack/operator/azure-stack-create-service-principals)voor meer informatie over het maken van service-principals en het autoriseren van de toegang. Als u liever Azure CLI wilt gebruiken, raadpleegt u [een Azure-service-principal maken met Azure cli](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).
- Implementeer Azure-Cognitive Services in azure of Azure Stack hub.
  - Lees eerst [meer over Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).
  - Ga vervolgens naar [Azure stack hub en implementeer Azure Cognitive Services](/azure-stack/user/azure-stack-solution-template-cognitive-services) om Cognitive services te implementeren op Azure stack hub. U moet zich eerst aanmelden voor toegang tot de preview-versie.
- Een niet-geconfigureerde Azure Custom Vision AI dev kit klonen of downloaden. Zie de [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/)voor meer informatie.
- Meld u aan voor een Power BI-account.
- Een Azure Cognitive Services Face-API-abonnements sleutel en eind punt-URL. U kunt beide met de gratis proef versie van [Cognitive Services uitproberen](https://azure.microsoft.com/try/cognitive-services/?api=face-api) . Of volg de instructies in [Create a cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).
- Installeer de volgende ontwikkelings bronnen:
  - [Azure CLI 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - / [Importeren](https://porter.sh/). U gebruikt importeren om Cloud-apps te implementeren met behulp van CNAB-bundel manifesten die voor u zijn.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Azure IoT-hulpprogramma's voor Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Python-extensie voor Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>De hybride Cloud-app implementeren

Eerst gebruikt u de opdracht regel importeren om een referentieset te genereren en implementeert u vervolgens de Cloud-app.  

1. Kloon of down load de voorbeeld code van de oplossing van https://github.com/azure-samples/azure-intelligent-edge-patterns . 

1. Met het importeren wordt een set referenties gegenereerd waarmee de implementatie van de app wordt geautomatiseerd. Zorg ervoor dat u het volgende hebt voordat u de opdracht voor het genereren van referenties uitvoert:

    - Een service-principal voor toegang tot Azure-resources, met inbegrip van de Service-Principal-ID, de sleutel en de Tenant-DNS.
    - De abonnements-ID voor uw Azure-abonnement.
    - Een service-principal voor toegang tot Azure Stack hub-resources, met inbegrip van de Service-Principal-ID, de sleutel en de Tenant-DNS.
    - De abonnements-ID voor uw Azure Stack hub-abonnement.
    - De Azure Cognitive Services Face-API sleutel en het resource-eind punt-URL.

1. Voer het proces voor het genereren van referenties importeren uit en volg de prompts:

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Voor het uitvoeren van de functie moet u ook een set para meters moeten worden uitgevoerd. Maak een tekst bestand voor de para meter en voer de volgende naam/waarde-paren in. Vraag uw Azure Stack hub-beheerder als u hulp nodig hebt met een van de vereiste waarden.

   > [!NOTE] 
   > De `resource suffix` waarde wordt gebruikt om ervoor te zorgen dat de resources van de implementatie unieke namen hebben in Azure. Dit moet een unieke teken reeks van letters en cijfers zijn, Maxi maal 8 tekens.

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   Sla het tekst bestand op en noteer het pad.

1. U bent nu klaar om de Hybrid Cloud-app te implementeren met behulp van importeren. Voer de installatie opdracht uit en controleer of er resources zijn geïmplementeerd in Azure en Azure Stack hub:

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. Nadat de implementatie is voltooid, noteert u de volgende waarden:
    - De connection string van de camera.
    - Het opslag account voor installatie kopie connection string.
    - De namen van de resource groep.

## <a name="prepare-the-custom-vision-ai-devkit"></a>De Custom Vision AI DevKit voorbereiden

Stel vervolgens de Custom Vision AI dev kit in, zoals wordt weer gegeven in de DevKit Quick Start van de [Vision AI](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/). U kunt uw camera ook instellen en testen met behulp van de connection string die u in de vorige stap hebt opgegeven.

## <a name="deploy-the-camera-app"></a>De camera-app implementeren

Gebruik de opdracht regel importeren om een referentieset te genereren en implementeer vervolgens de camera-app.

1. Met het importeren wordt een set referenties gegenereerd waarmee de implementatie van de app wordt geautomatiseerd. Zorg ervoor dat u het volgende hebt voordat u de opdracht voor het genereren van referenties uitvoert:

    - Een service-principal voor toegang tot Azure-resources, met inbegrip van de Service-Principal-ID, de sleutel en de Tenant-DNS.
    - De abonnements-ID voor uw Azure-abonnement.
    - Het opslag account voor installatie kopieën connection string die is ingevoerd tijdens de implementatie van de Cloud-app.

1. Voer het proces voor het genereren van referenties importeren uit en volg de prompts:

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Voor het uitvoeren van de functie moet u ook een set para meters moeten worden uitgevoerd. Maak een tekst bestand voor de para meter en voer de volgende tekst in. Vraag uw Azure Stack hub-beheerder als u enkele van de vereiste waarden niet kent.

    > [!NOTE]
    > De `deployment suffix` waarde wordt gebruikt om ervoor te zorgen dat de resources van de implementatie unieke namen hebben in Azure. Dit moet een unieke teken reeks van letters en cijfers zijn, Maxi maal 8 tekens.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    Sla het tekst bestand op en noteer het pad.

4. U bent nu klaar om de camera-app te implementeren met behulp van importeren. Voer de installatie opdracht uit en kijk wanneer de IoT Edge-implementatie is gemaakt.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. Controleer of de implementatie van de camera is voltooid door de camera feed te bekijken op `https://<camera-ip>:3000/` , waarbij `<camara-ip>` het IP-adres van de camera is. Deze stap kan tot 10 minuten duren.

## <a name="configure-azure-stream-analytics"></a>Azure Stream Analytics configureren

Nu de gegevens naar Azure Stream Analytics van de camera stromen, moeten we deze hand matig autoriseren om te communiceren met Power BI.

1. Open vanuit de Azure Portal **alle resources** en de taak *process-Footfall \[ yoursuffix \]* .

2. Selecteer in de sectie **Taaktopologie** van het deelvenster van de Stream Analytics-taak de optie **Uitvoer**.

3. Selecteer de uitvoer Sink **verkeers uitvoer** .

4. Selecteer **autorisatie vernieuwen** en meld u aan bij uw Power bi-account.
  
    ![Autorisatie prompt vernieuwen in Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. Sla de uitvoer instellingen op.

6. Ga naar het deel venster **overzicht** en selecteer **starten** om te beginnen met het verzenden van gegevens naar Power bi.

7. Selecteer **Nu** voor de starttijd van de taakuitvoer en selecteer **Starten**. U kunt de taakstatus bekijken in de meldingsbalk.

## <a name="create-a-power-bi-dashboard"></a>Een Power BI-dash board maken

1. Zodra de taak is voltooid, gaat u naar [Power bi](https://powerbi.com/) en meldt u zich aan met uw werk-of school account. Als de query van de Stream Analytics-taak uitvoer resultaten oplevert, bestaat de gegevensset van de *Footfall* die u hebt gemaakt op het tabblad **gegevens sets** .

2. Selecteer in uw Power BI-werk ruimte **+ maken** om een nieuw dash board met de naam *Footfall analyse* te maken.

3. Selecteer boven in het venster de optie **Tegel toevoegen**. Selecteer **Aangepaste streaminggegevens** en **Volgende**. Kies de **Footfall-gegevensset** onder **uw gegevens sets**. Selecteer **kaart** in de vervolg keuzelijst **type visualisatie** en voeg **leeftijd** toe aan **velden**. Selecteer **Volgende** om een naam voor de tegel in te voeren en selecteer vervolgens **Toepassen** om het bestand te maken.

4. U kunt desgewenst extra velden en kaarten toevoegen.

## <a name="test-your-solution"></a>Uw oplossing testen

Houd er rekening mee dat de gegevens in de kaarten die u hebt gemaakt in Power BI verandert als verschillende mensen vóór de camera lopen. Het kan Maxi maal twintig seconden duren voordat het vastleggen wordt weer gegeven.

## <a name="remove-your-solution"></a>Uw oplossing verwijderen

Als u uw oplossing wilt verwijderen, voert u de volgende opdrachten uit met behulp van de para meters die u hebt gemaakt voor de implementatie:

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>Volgende stappen

- Meer informatie over [overwegingen voor het ontwerpen van hybride apps](overview-app-design-considerations.md)
- Bekijk en suggereer verbeteringen in [de code voor dit voor beeld op github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).
