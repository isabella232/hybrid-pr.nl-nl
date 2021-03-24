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
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="6cf91-103">Een op AI gebaseerde Footfall-detectie oplossing implementeren met behulp van Azure en Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="6cf91-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="6cf91-104">In dit artikel wordt beschreven hoe u een op AI gebaseerde oplossing implementeert die inzichten genereert op basis van Real-World-acties met behulp van Azure, Azure Stack hub en de Custom Vision AI dev kit.</span><span class="sxs-lookup"><span data-stu-id="6cf91-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="6cf91-105">In deze oplossing leert u het volgende:</span><span class="sxs-lookup"><span data-stu-id="6cf91-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6cf91-106">Implementeer native Cloud Application bundels (CNAB) aan de rand.</span><span class="sxs-lookup"><span data-stu-id="6cf91-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="6cf91-107">Implementeer een app die de Cloud grenzen omspant.</span><span class="sxs-lookup"><span data-stu-id="6cf91-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="6cf91-108">Gebruik de Custom Vision AI dev kit voor de rand.</span><span class="sxs-lookup"><span data-stu-id="6cf91-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="6cf91-109">![Diagram hybride pijlers](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="6cf91-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="6cf91-110">Microsoft Azure Stack hub is een uitbrei ding van Azure.</span><span class="sxs-lookup"><span data-stu-id="6cf91-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="6cf91-111">Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.</span><span class="sxs-lookup"><span data-stu-id="6cf91-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="6cf91-112">In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps.</span><span class="sxs-lookup"><span data-stu-id="6cf91-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="6cf91-113">De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.</span><span class="sxs-lookup"><span data-stu-id="6cf91-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6cf91-114">Vereisten</span><span class="sxs-lookup"><span data-stu-id="6cf91-114">Prerequisites</span></span>

<span data-ttu-id="6cf91-115">Voordat u aan de slag gaat met deze implementatie handleiding, moet u het volgende doen:</span><span class="sxs-lookup"><span data-stu-id="6cf91-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="6cf91-116">Bekijk het onderwerp [Footfall-detectie patroon](pattern-retail-footfall-detection.md) .</span><span class="sxs-lookup"><span data-stu-id="6cf91-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="6cf91-117">Gebruikers toegang verkrijgen tot een met Azure Stack Development Kit (ASDK) of Azure Stack hub geïntegreerde systeem instantie, met:</span><span class="sxs-lookup"><span data-stu-id="6cf91-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="6cf91-118">De [Azure app service op Azure stack hub-resource provider](/azure-stack/operator/azure-stack-app-service-overview) geïnstalleerd.</span><span class="sxs-lookup"><span data-stu-id="6cf91-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview) installed.</span></span> <span data-ttu-id="6cf91-119">U hebt operator toegang tot uw Azure Stack hub-exemplaar nodig of u kunt samen werken met de beheerder om deze te installeren.</span><span class="sxs-lookup"><span data-stu-id="6cf91-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="6cf91-120">Een abonnement op een aanbieding die App Service en opslag limiet biedt.</span><span class="sxs-lookup"><span data-stu-id="6cf91-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="6cf91-121">U hebt operator toegang nodig om een aanbieding te maken.</span><span class="sxs-lookup"><span data-stu-id="6cf91-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="6cf91-122">Toegang verkrijgen tot een Azure-abonnement.</span><span class="sxs-lookup"><span data-stu-id="6cf91-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="6cf91-123">Als u nog geen abonnement op Azure hebt, Meld u dan aan voor een [gratis proef account](https://azure.microsoft.com/free/) voordat u begint.</span><span class="sxs-lookup"><span data-stu-id="6cf91-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="6cf91-124">Maak twee service-principals in uw Directory:</span><span class="sxs-lookup"><span data-stu-id="6cf91-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="6cf91-125">Eén ingesteld voor gebruik met Azure-resources, met toegang tot het Azure-abonnements bereik.</span><span class="sxs-lookup"><span data-stu-id="6cf91-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="6cf91-126">Er is een set ingesteld voor gebruik met Azure Stack hub-resources, met toegang op het Azure Stack hub-abonnements bereik.</span><span class="sxs-lookup"><span data-stu-id="6cf91-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="6cf91-127">Zie [een app-identiteit gebruiken voor toegang tot resources](/azure-stack/operator/azure-stack-create-service-principals)voor meer informatie over het maken van service-principals en het autoriseren van de toegang.</span><span class="sxs-lookup"><span data-stu-id="6cf91-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals).</span></span> <span data-ttu-id="6cf91-128">Als u liever Azure CLI wilt gebruiken, raadpleegt u [een Azure-service-principal maken met Azure cli](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span><span class="sxs-lookup"><span data-stu-id="6cf91-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="6cf91-129">Implementeer Azure-Cognitive Services in azure of Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6cf91-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="6cf91-130">Lees eerst [meer over Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="6cf91-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="6cf91-131">Ga vervolgens naar [Azure stack hub en implementeer Azure Cognitive Services](/azure-stack/user/azure-stack-solution-template-cognitive-services) om Cognitive services te implementeren op Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="6cf91-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="6cf91-132">U moet zich eerst aanmelden voor toegang tot de preview-versie.</span><span class="sxs-lookup"><span data-stu-id="6cf91-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="6cf91-133">Een niet-geconfigureerde Azure Custom Vision AI dev kit klonen of downloaden.</span><span class="sxs-lookup"><span data-stu-id="6cf91-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="6cf91-134">Zie de [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="6cf91-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="6cf91-135">Meld u aan voor een Power BI-account.</span><span class="sxs-lookup"><span data-stu-id="6cf91-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="6cf91-136">Een Azure Cognitive Services Face-API-abonnements sleutel en eind punt-URL.</span><span class="sxs-lookup"><span data-stu-id="6cf91-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="6cf91-137">U kunt beide met de gratis proef versie van [Cognitive Services uitproberen](https://azure.microsoft.com/try/cognitive-services/?api=face-api) .</span><span class="sxs-lookup"><span data-stu-id="6cf91-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="6cf91-138">Of volg de instructies in [Create a cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="6cf91-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="6cf91-139">Installeer de volgende ontwikkelings bronnen:</span><span class="sxs-lookup"><span data-stu-id="6cf91-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="6cf91-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="6cf91-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [<span data-ttu-id="6cf91-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="6cf91-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="6cf91-142">/ [Importeren](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="6cf91-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="6cf91-143">U gebruikt importeren om Cloud-apps te implementeren met behulp van CNAB-bundel manifesten die voor u zijn.</span><span class="sxs-lookup"><span data-stu-id="6cf91-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="6cf91-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="6cf91-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="6cf91-145">Azure IoT-hulpprogramma's voor Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="6cf91-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="6cf91-146">Python-extensie voor Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="6cf91-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="6cf91-147">Python</span><span class="sxs-lookup"><span data-stu-id="6cf91-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="6cf91-148">De hybride Cloud-app implementeren</span><span class="sxs-lookup"><span data-stu-id="6cf91-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="6cf91-149">Eerst gebruikt u de opdracht regel importeren om een referentieset te genereren en implementeert u vervolgens de Cloud-app.</span><span class="sxs-lookup"><span data-stu-id="6cf91-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="6cf91-150">Kloon of down load de voorbeeld code van de oplossing van https://github.com/azure-samples/azure-intelligent-edge-patterns .</span><span class="sxs-lookup"><span data-stu-id="6cf91-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="6cf91-151">Met het importeren wordt een set referenties gegenereerd waarmee de implementatie van de app wordt geautomatiseerd.</span><span class="sxs-lookup"><span data-stu-id="6cf91-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="6cf91-152">Zorg ervoor dat u het volgende hebt voordat u de opdracht voor het genereren van referenties uitvoert:</span><span class="sxs-lookup"><span data-stu-id="6cf91-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="6cf91-153">Een service-principal voor toegang tot Azure-resources, met inbegrip van de Service-Principal-ID, de sleutel en de Tenant-DNS.</span><span class="sxs-lookup"><span data-stu-id="6cf91-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="6cf91-154">De abonnements-ID voor uw Azure-abonnement.</span><span class="sxs-lookup"><span data-stu-id="6cf91-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="6cf91-155">Een service-principal voor toegang tot Azure Stack hub-resources, met inbegrip van de Service-Principal-ID, de sleutel en de Tenant-DNS.</span><span class="sxs-lookup"><span data-stu-id="6cf91-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="6cf91-156">De abonnements-ID voor uw Azure Stack hub-abonnement.</span><span class="sxs-lookup"><span data-stu-id="6cf91-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="6cf91-157">De Azure Cognitive Services Face-API sleutel en het resource-eind punt-URL.</span><span class="sxs-lookup"><span data-stu-id="6cf91-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="6cf91-158">Voer het proces voor het genereren van referenties importeren uit en volg de prompts:</span><span class="sxs-lookup"><span data-stu-id="6cf91-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="6cf91-159">Voor het uitvoeren van de functie moet u ook een set para meters moeten worden uitgevoerd.</span><span class="sxs-lookup"><span data-stu-id="6cf91-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="6cf91-160">Maak een tekst bestand voor de para meter en voer de volgende naam/waarde-paren in.</span><span class="sxs-lookup"><span data-stu-id="6cf91-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="6cf91-161">Vraag uw Azure Stack hub-beheerder als u hulp nodig hebt met een van de vereiste waarden.</span><span class="sxs-lookup"><span data-stu-id="6cf91-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="6cf91-162">De `resource suffix` waarde wordt gebruikt om ervoor te zorgen dat de resources van de implementatie unieke namen hebben in Azure.</span><span class="sxs-lookup"><span data-stu-id="6cf91-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="6cf91-163">Dit moet een unieke teken reeks van letters en cijfers zijn, Maxi maal 8 tekens.</span><span class="sxs-lookup"><span data-stu-id="6cf91-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

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
   <span data-ttu-id="6cf91-164">Sla het tekst bestand op en noteer het pad.</span><span class="sxs-lookup"><span data-stu-id="6cf91-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="6cf91-165">U bent nu klaar om de Hybrid Cloud-app te implementeren met behulp van importeren.</span><span class="sxs-lookup"><span data-stu-id="6cf91-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="6cf91-166">Voer de installatie opdracht uit en controleer of er resources zijn geïmplementeerd in Azure en Azure Stack hub:</span><span class="sxs-lookup"><span data-stu-id="6cf91-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="6cf91-167">Nadat de implementatie is voltooid, noteert u de volgende waarden:</span><span class="sxs-lookup"><span data-stu-id="6cf91-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="6cf91-168">De connection string van de camera.</span><span class="sxs-lookup"><span data-stu-id="6cf91-168">The camera's connection string.</span></span>
    - <span data-ttu-id="6cf91-169">Het opslag account voor installatie kopie connection string.</span><span class="sxs-lookup"><span data-stu-id="6cf91-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="6cf91-170">De namen van de resource groep.</span><span class="sxs-lookup"><span data-stu-id="6cf91-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="6cf91-171">De Custom Vision AI DevKit voorbereiden</span><span class="sxs-lookup"><span data-stu-id="6cf91-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="6cf91-172">Stel vervolgens de Custom Vision AI dev kit in, zoals wordt weer gegeven in de DevKit Quick Start van de [Vision AI](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="6cf91-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="6cf91-173">U kunt uw camera ook instellen en testen met behulp van de connection string die u in de vorige stap hebt opgegeven.</span><span class="sxs-lookup"><span data-stu-id="6cf91-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="6cf91-174">De camera-app implementeren</span><span class="sxs-lookup"><span data-stu-id="6cf91-174">Deploy the camera app</span></span>

<span data-ttu-id="6cf91-175">Gebruik de opdracht regel importeren om een referentieset te genereren en implementeer vervolgens de camera-app.</span><span class="sxs-lookup"><span data-stu-id="6cf91-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="6cf91-176">Met het importeren wordt een set referenties gegenereerd waarmee de implementatie van de app wordt geautomatiseerd.</span><span class="sxs-lookup"><span data-stu-id="6cf91-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="6cf91-177">Zorg ervoor dat u het volgende hebt voordat u de opdracht voor het genereren van referenties uitvoert:</span><span class="sxs-lookup"><span data-stu-id="6cf91-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="6cf91-178">Een service-principal voor toegang tot Azure-resources, met inbegrip van de Service-Principal-ID, de sleutel en de Tenant-DNS.</span><span class="sxs-lookup"><span data-stu-id="6cf91-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="6cf91-179">De abonnements-ID voor uw Azure-abonnement.</span><span class="sxs-lookup"><span data-stu-id="6cf91-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="6cf91-180">Het opslag account voor installatie kopieën connection string die is ingevoerd tijdens de implementatie van de Cloud-app.</span><span class="sxs-lookup"><span data-stu-id="6cf91-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="6cf91-181">Voer het proces voor het genereren van referenties importeren uit en volg de prompts:</span><span class="sxs-lookup"><span data-stu-id="6cf91-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="6cf91-182">Voor het uitvoeren van de functie moet u ook een set para meters moeten worden uitgevoerd.</span><span class="sxs-lookup"><span data-stu-id="6cf91-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="6cf91-183">Maak een tekst bestand voor de para meter en voer de volgende tekst in.</span><span class="sxs-lookup"><span data-stu-id="6cf91-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="6cf91-184">Vraag uw Azure Stack hub-beheerder als u enkele van de vereiste waarden niet kent.</span><span class="sxs-lookup"><span data-stu-id="6cf91-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="6cf91-185">De `deployment suffix` waarde wordt gebruikt om ervoor te zorgen dat de resources van de implementatie unieke namen hebben in Azure.</span><span class="sxs-lookup"><span data-stu-id="6cf91-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="6cf91-186">Dit moet een unieke teken reeks van letters en cijfers zijn, Maxi maal 8 tekens.</span><span class="sxs-lookup"><span data-stu-id="6cf91-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="6cf91-187">Sla het tekst bestand op en noteer het pad.</span><span class="sxs-lookup"><span data-stu-id="6cf91-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="6cf91-188">U bent nu klaar om de camera-app te implementeren met behulp van importeren.</span><span class="sxs-lookup"><span data-stu-id="6cf91-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="6cf91-189">Voer de installatie opdracht uit en kijk wanneer de IoT Edge-implementatie is gemaakt.</span><span class="sxs-lookup"><span data-stu-id="6cf91-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="6cf91-190">Controleer of de implementatie van de camera is voltooid door de camera feed te bekijken op `https://<camera-ip>:3000/` , waarbij `<camara-ip>` het IP-adres van de camera is.</span><span class="sxs-lookup"><span data-stu-id="6cf91-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="6cf91-191">Deze stap kan tot 10 minuten duren.</span><span class="sxs-lookup"><span data-stu-id="6cf91-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="6cf91-192">Azure Stream Analytics configureren</span><span class="sxs-lookup"><span data-stu-id="6cf91-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="6cf91-193">Nu de gegevens naar Azure Stream Analytics van de camera stromen, moeten we deze hand matig autoriseren om te communiceren met Power BI.</span><span class="sxs-lookup"><span data-stu-id="6cf91-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="6cf91-194">Open vanuit de Azure Portal **alle resources** en de taak *process-Footfall \[ yoursuffix \]* .</span><span class="sxs-lookup"><span data-stu-id="6cf91-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="6cf91-195">Selecteer in de sectie **Taaktopologie** van het deelvenster van de Stream Analytics-taak de optie **Uitvoer**.</span><span class="sxs-lookup"><span data-stu-id="6cf91-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="6cf91-196">Selecteer de uitvoer Sink **verkeers uitvoer** .</span><span class="sxs-lookup"><span data-stu-id="6cf91-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="6cf91-197">Selecteer **autorisatie vernieuwen** en meld u aan bij uw Power bi-account.</span><span class="sxs-lookup"><span data-stu-id="6cf91-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Autorisatie prompt vernieuwen in Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="6cf91-199">Sla de uitvoer instellingen op.</span><span class="sxs-lookup"><span data-stu-id="6cf91-199">Save the output settings.</span></span>

6. <span data-ttu-id="6cf91-200">Ga naar het deel venster **overzicht** en selecteer **starten** om te beginnen met het verzenden van gegevens naar Power bi.</span><span class="sxs-lookup"><span data-stu-id="6cf91-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="6cf91-201">Selecteer **Nu** voor de starttijd van de taakuitvoer en selecteer **Starten**.</span><span class="sxs-lookup"><span data-stu-id="6cf91-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="6cf91-202">U kunt de taakstatus bekijken in de meldingsbalk.</span><span class="sxs-lookup"><span data-stu-id="6cf91-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="6cf91-203">Een Power BI-dash board maken</span><span class="sxs-lookup"><span data-stu-id="6cf91-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="6cf91-204">Zodra de taak is voltooid, gaat u naar [Power bi](https://powerbi.com/) en meldt u zich aan met uw werk-of school account.</span><span class="sxs-lookup"><span data-stu-id="6cf91-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="6cf91-205">Als de query van de Stream Analytics-taak uitvoer resultaten oplevert, bestaat de gegevensset van de *Footfall* die u hebt gemaakt op het tabblad **gegevens sets** .</span><span class="sxs-lookup"><span data-stu-id="6cf91-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="6cf91-206">Selecteer in uw Power BI-werk ruimte **+ maken** om een nieuw dash board met de naam *Footfall analyse* te maken.</span><span class="sxs-lookup"><span data-stu-id="6cf91-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="6cf91-207">Selecteer boven in het venster de optie **Tegel toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="6cf91-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="6cf91-208">Selecteer **Aangepaste streaminggegevens** en **Volgende**.</span><span class="sxs-lookup"><span data-stu-id="6cf91-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="6cf91-209">Kies de **Footfall-gegevensset** onder **uw gegevens sets**.</span><span class="sxs-lookup"><span data-stu-id="6cf91-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="6cf91-210">Selecteer **kaart** in de vervolg keuzelijst **type visualisatie** en voeg **leeftijd** toe aan **velden**.</span><span class="sxs-lookup"><span data-stu-id="6cf91-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="6cf91-211">Selecteer **Volgende** om een naam voor de tegel in te voeren en selecteer vervolgens **Toepassen** om het bestand te maken.</span><span class="sxs-lookup"><span data-stu-id="6cf91-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="6cf91-212">U kunt desgewenst extra velden en kaarten toevoegen.</span><span class="sxs-lookup"><span data-stu-id="6cf91-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="6cf91-213">Uw oplossing testen</span><span class="sxs-lookup"><span data-stu-id="6cf91-213">Test Your Solution</span></span>

<span data-ttu-id="6cf91-214">Houd er rekening mee dat de gegevens in de kaarten die u hebt gemaakt in Power BI verandert als verschillende mensen vóór de camera lopen.</span><span class="sxs-lookup"><span data-stu-id="6cf91-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="6cf91-215">Het kan Maxi maal twintig seconden duren voordat het vastleggen wordt weer gegeven.</span><span class="sxs-lookup"><span data-stu-id="6cf91-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="6cf91-216">Uw oplossing verwijderen</span><span class="sxs-lookup"><span data-stu-id="6cf91-216">Remove Your Solution</span></span>

<span data-ttu-id="6cf91-217">Als u uw oplossing wilt verwijderen, voert u de volgende opdrachten uit met behulp van de para meters die u hebt gemaakt voor de implementatie:</span><span class="sxs-lookup"><span data-stu-id="6cf91-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="6cf91-218">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="6cf91-218">Next steps</span></span>

- <span data-ttu-id="6cf91-219">Meer informatie over [overwegingen voor het ontwerpen van hybride apps](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="6cf91-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="6cf91-220">Bekijk en suggereer verbeteringen in [de code voor dit voor beeld op github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="6cf91-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
