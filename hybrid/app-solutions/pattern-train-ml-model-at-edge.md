---
title: machine learning model trainen op het rand patroon
description: Meer informatie over het machine learning model training aan de rand van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910414"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>machine learning model trainen op het rand patroon

Genereer modellen voor draag bare machine learning (ML) van gegevens die alleen on-premises bestaan.

## <a name="context-and-problem"></a>Context en probleem

Veel organisaties willen inzichten van hun on-premises of verouderde gegevens ontgrendelen met behulp van hulpprogram ma's die hun gegevens wetenschappers begrijpen. [Azure machine learning](/azure/machine-learning/) biedt Cloud-systeem eigen hulp middelen voor het trainen, afstemmen en implementeren van ml en diepe leer modellen.  

Sommige gegevens zijn echter te groot voor verzen ding naar de Cloud of kunnen niet naar de cloud worden verzonden om wettelijke redenen. Met dit patroon kunnen gegevens wetenschappers Azure Machine Learning gebruiken om modellen te trainen met behulp van on-premises gegevens en reken kracht.

## <a name="solution"></a>Oplossing

De training aan het rand patroon maakt gebruik van een virtuele machine (VM) die wordt uitgevoerd op Azure Stack hub. De virtuele machine is geregistreerd als een reken doel in azure ML, zodat deze toegang heeft tot gegevens die on-premises beschikbaar zijn. In dit geval worden de gegevens opgeslagen in de Blob-opslag van Azure Stack hub.

Zodra het model is getraind, wordt het geregistreerd bij Azure ML, in containers en toegevoegd aan een Azure Container Registry voor implementatie. Voor deze herhaling van het patroon moet de VM van de Azure Stack hub-training bereikbaar zijn via het open bare Internet.

[![Model van Train ML aan de rand architectuur](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

Het patroon werkt als volgt:

1. De Azure Stack hub-VM wordt geïmplementeerd en geregistreerd als een reken doel met Azure ML.
2. Er wordt een experiment gemaakt in azure ML dat gebruikmaakt van de Azure Stack hub-VM als reken doel.
3. Zodra het model is getraind, wordt het geregistreerd en in containers opgenomen.
4. Het model kan nu worden geïmplementeerd op locaties die on-premises of in de Cloud zijn.

## <a name="components"></a>Onderdelen

Deze oplossing maakt gebruik van de volgende onderdelen:

| Laag | Onderdeel | Beschrijving |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [Azure machine learning](/azure/machine-learning/) organiseert de training van het ml-model. |
| | Azure Container Registry | Azure ML verpakt het model in een container en slaat het op in een [Azure container Registry](/azure/container-registry/) voor implementatie.|
| Azure Stack hub | App Service | [Azure stack hub met app service](/azure-stack/operator/azure-stack-app-service-overview) biedt de basis voor de onderdelen aan de rand. |
| | Compute | Een Azure Stack hub-VM waarop Ubuntu wordt uitgevoerd met docker, wordt gebruikt om het ML-model te trainen. |
| | Storage | Privé gegevens kunnen worden gehost in Azure Stack hub-Blob-opslag. |

## <a name="issues-and-considerations"></a>Problemen en overwegingen

Houd rekening met de volgende punten wanneer u bepaalt hoe u deze oplossing implementeert:

### <a name="scalability"></a>Schaalbaarheid

Als u wilt dat deze oplossing kan worden geschaald, moet u een VM met een juiste grootte op Azure Stack hub maken voor training.

### <a name="availability"></a>Beschikbaarheid

Zorg ervoor dat de trainings scripts en Azure Stack hub-VM toegang hebben tot de on-premises gegevens die worden gebruikt voor de training.

### <a name="manageability"></a>Beheerbaarheid

Zorg ervoor dat modellen en experimenten op de juiste wijze zijn geregistreerd, geversied en gelabeld om Verwar ring te voor komen tijdens de implementatie van een model.

### <a name="security"></a>Beveiliging

Met dit patroon kan Azure ML toegang tot gevoelige gegevens on-premises krijgen. Zorg ervoor dat het account dat wordt gebruikt voor SSH in Azure Stack hub-VM een sterk wacht woord-en trainings script heeft dat geen gegevens in de Cloud bewaard of uploadt.

## <a name="next-steps"></a>Volgende stappen

Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:

- Raadpleeg de [documentatie van Azure machine learning](/azure/machine-learning) voor een overzicht van ml en verwante onderwerpen.
- Zie [Azure container Registry](/azure/container-registry/) voor meer informatie over het bouwen, opslaan en beheren van installatie kopieën voor container implementaties.
- Raadpleeg [app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-overview) voor meer informatie over de resource provider en hoe u deze implementeert.
- Zie [overwegingen voor het ontwerpen van hybride toepassingen](overview-app-design-considerations.md) voor meer informatie over best practices en om eventuele aanvullende vragen te ontvangen.
- Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.

Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met het [Train ml-model in de implementatie handleiding voor Edge](https://aka.ms/edgetrainingdeploy). De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.
