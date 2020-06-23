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
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="258e4-103">machine learning model trainen op het rand patroon</span><span class="sxs-lookup"><span data-stu-id="258e4-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="258e4-104">Genereer modellen voor draag bare machine learning (ML) van gegevens die alleen on-premises bestaan.</span><span class="sxs-lookup"><span data-stu-id="258e4-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="258e4-105">Context en probleem</span><span class="sxs-lookup"><span data-stu-id="258e4-105">Context and problem</span></span>

<span data-ttu-id="258e4-106">Veel organisaties willen inzichten van hun on-premises of verouderde gegevens ontgrendelen met behulp van hulpprogram ma's die hun gegevens wetenschappers begrijpen.</span><span class="sxs-lookup"><span data-stu-id="258e4-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="258e4-107">[Azure machine learning](/azure/machine-learning/) biedt Cloud-systeem eigen hulp middelen voor het trainen, afstemmen en implementeren van ml en diepe leer modellen.</span><span class="sxs-lookup"><span data-stu-id="258e4-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="258e4-108">Sommige gegevens zijn echter te groot voor verzen ding naar de Cloud of kunnen niet naar de cloud worden verzonden om wettelijke redenen.</span><span class="sxs-lookup"><span data-stu-id="258e4-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="258e4-109">Met dit patroon kunnen gegevens wetenschappers Azure Machine Learning gebruiken om modellen te trainen met behulp van on-premises gegevens en reken kracht.</span><span class="sxs-lookup"><span data-stu-id="258e4-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="258e4-110">Oplossing</span><span class="sxs-lookup"><span data-stu-id="258e4-110">Solution</span></span>

<span data-ttu-id="258e4-111">De training aan het rand patroon maakt gebruik van een virtuele machine (VM) die wordt uitgevoerd op Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="258e4-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="258e4-112">De virtuele machine is geregistreerd als een reken doel in azure ML, zodat deze toegang heeft tot gegevens die on-premises beschikbaar zijn.</span><span class="sxs-lookup"><span data-stu-id="258e4-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="258e4-113">In dit geval worden de gegevens opgeslagen in de Blob-opslag van Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="258e4-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="258e4-114">Zodra het model is getraind, wordt het geregistreerd bij Azure ML, in containers en toegevoegd aan een Azure Container Registry voor implementatie.</span><span class="sxs-lookup"><span data-stu-id="258e4-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="258e4-115">Voor deze herhaling van het patroon moet de VM van de Azure Stack hub-training bereikbaar zijn via het open bare Internet.</span><span class="sxs-lookup"><span data-stu-id="258e4-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="258e4-116">[![Model van Train ML aan de rand architectuur](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="258e4-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="258e4-117">Het patroon werkt als volgt:</span><span class="sxs-lookup"><span data-stu-id="258e4-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="258e4-118">De Azure Stack hub-VM wordt geïmplementeerd en geregistreerd als een reken doel met Azure ML.</span><span class="sxs-lookup"><span data-stu-id="258e4-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="258e4-119">Er wordt een experiment gemaakt in azure ML dat gebruikmaakt van de Azure Stack hub-VM als reken doel.</span><span class="sxs-lookup"><span data-stu-id="258e4-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="258e4-120">Zodra het model is getraind, wordt het geregistreerd en in containers opgenomen.</span><span class="sxs-lookup"><span data-stu-id="258e4-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="258e4-121">Het model kan nu worden geïmplementeerd op locaties die on-premises of in de Cloud zijn.</span><span class="sxs-lookup"><span data-stu-id="258e4-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="258e4-122">Onderdelen</span><span class="sxs-lookup"><span data-stu-id="258e4-122">Components</span></span>

<span data-ttu-id="258e4-123">Deze oplossing maakt gebruik van de volgende onderdelen:</span><span class="sxs-lookup"><span data-stu-id="258e4-123">This solution uses the following components:</span></span>

| <span data-ttu-id="258e4-124">Laag</span><span class="sxs-lookup"><span data-stu-id="258e4-124">Layer</span></span> | <span data-ttu-id="258e4-125">Onderdeel</span><span class="sxs-lookup"><span data-stu-id="258e4-125">Component</span></span> | <span data-ttu-id="258e4-126">Beschrijving</span><span class="sxs-lookup"><span data-stu-id="258e4-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="258e4-127">Azure</span><span class="sxs-lookup"><span data-stu-id="258e4-127">Azure</span></span> | <span data-ttu-id="258e4-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="258e4-128">Azure Machine Learning</span></span> | <span data-ttu-id="258e4-129">[Azure machine learning](/azure/machine-learning/) organiseert de training van het ml-model.</span><span class="sxs-lookup"><span data-stu-id="258e4-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="258e4-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="258e4-130">Azure Container Registry</span></span> | <span data-ttu-id="258e4-131">Azure ML verpakt het model in een container en slaat het op in een [Azure container Registry](/azure/container-registry/) voor implementatie.</span><span class="sxs-lookup"><span data-stu-id="258e4-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="258e4-132">Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="258e4-132">Azure Stack Hub</span></span> | <span data-ttu-id="258e4-133">App Service</span><span class="sxs-lookup"><span data-stu-id="258e4-133">App Service</span></span> | <span data-ttu-id="258e4-134">[Azure stack hub met app service](/azure-stack/operator/azure-stack-app-service-overview) biedt de basis voor de onderdelen aan de rand.</span><span class="sxs-lookup"><span data-stu-id="258e4-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="258e4-135">Compute</span><span class="sxs-lookup"><span data-stu-id="258e4-135">Compute</span></span> | <span data-ttu-id="258e4-136">Een Azure Stack hub-VM waarop Ubuntu wordt uitgevoerd met docker, wordt gebruikt om het ML-model te trainen.</span><span class="sxs-lookup"><span data-stu-id="258e4-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="258e4-137">Storage</span><span class="sxs-lookup"><span data-stu-id="258e4-137">Storage</span></span> | <span data-ttu-id="258e4-138">Privé gegevens kunnen worden gehost in Azure Stack hub-Blob-opslag.</span><span class="sxs-lookup"><span data-stu-id="258e4-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="258e4-139">Problemen en overwegingen</span><span class="sxs-lookup"><span data-stu-id="258e4-139">Issues and considerations</span></span>

<span data-ttu-id="258e4-140">Houd rekening met de volgende punten wanneer u bepaalt hoe u deze oplossing implementeert:</span><span class="sxs-lookup"><span data-stu-id="258e4-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="258e4-141">Schaalbaarheid</span><span class="sxs-lookup"><span data-stu-id="258e4-141">Scalability</span></span>

<span data-ttu-id="258e4-142">Als u wilt dat deze oplossing kan worden geschaald, moet u een VM met een juiste grootte op Azure Stack hub maken voor training.</span><span class="sxs-lookup"><span data-stu-id="258e4-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="258e4-143">Beschikbaarheid</span><span class="sxs-lookup"><span data-stu-id="258e4-143">Availability</span></span>

<span data-ttu-id="258e4-144">Zorg ervoor dat de trainings scripts en Azure Stack hub-VM toegang hebben tot de on-premises gegevens die worden gebruikt voor de training.</span><span class="sxs-lookup"><span data-stu-id="258e4-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="258e4-145">Beheerbaarheid</span><span class="sxs-lookup"><span data-stu-id="258e4-145">Manageability</span></span>

<span data-ttu-id="258e4-146">Zorg ervoor dat modellen en experimenten op de juiste wijze zijn geregistreerd, geversied en gelabeld om Verwar ring te voor komen tijdens de implementatie van een model.</span><span class="sxs-lookup"><span data-stu-id="258e4-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="258e4-147">Beveiliging</span><span class="sxs-lookup"><span data-stu-id="258e4-147">Security</span></span>

<span data-ttu-id="258e4-148">Met dit patroon kan Azure ML toegang tot gevoelige gegevens on-premises krijgen.</span><span class="sxs-lookup"><span data-stu-id="258e4-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="258e4-149">Zorg ervoor dat het account dat wordt gebruikt voor SSH in Azure Stack hub-VM een sterk wacht woord-en trainings script heeft dat geen gegevens in de Cloud bewaard of uploadt.</span><span class="sxs-lookup"><span data-stu-id="258e4-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="258e4-150">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="258e4-150">Next steps</span></span>

<span data-ttu-id="258e4-151">Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:</span><span class="sxs-lookup"><span data-stu-id="258e4-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="258e4-152">Raadpleeg de [documentatie van Azure machine learning](/azure/machine-learning) voor een overzicht van ml en verwante onderwerpen.</span><span class="sxs-lookup"><span data-stu-id="258e4-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="258e4-153">Zie [Azure container Registry](/azure/container-registry/) voor meer informatie over het bouwen, opslaan en beheren van installatie kopieën voor container implementaties.</span><span class="sxs-lookup"><span data-stu-id="258e4-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="258e4-154">Raadpleeg [app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-overview) voor meer informatie over de resource provider en hoe u deze implementeert.</span><span class="sxs-lookup"><span data-stu-id="258e4-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="258e4-155">Zie [overwegingen voor het ontwerpen van hybride toepassingen](overview-app-design-considerations.md) voor meer informatie over best practices en om eventuele aanvullende vragen te ontvangen.</span><span class="sxs-lookup"><span data-stu-id="258e4-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="258e4-156">Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.</span><span class="sxs-lookup"><span data-stu-id="258e4-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="258e4-157">Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met het [Train ml-model in de implementatie handleiding voor Edge](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="258e4-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="258e4-158">De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.</span><span class="sxs-lookup"><span data-stu-id="258e4-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
