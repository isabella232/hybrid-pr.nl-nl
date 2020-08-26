---
title: Een app implementeren waarmee cross-Cloud in Azure en Azure Stack hub wordt geschaald
description: Meer informatie over het implementeren van een app waarmee u meerdere clouds kunt schalen in Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886812"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="7e49c-103">Een app implementeren waarmee u meerdere clouds schaalt met behulp van Azure en Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="7e49c-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="7e49c-104">Meer informatie over het maken van een cross-Cloud oplossing voor een hand matig geactiveerd proces voor het overschakelen van een door Azure Stack hub gehoste web-app naar een door Azure gehoste web-app met automatisch schalen via Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="7e49c-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="7e49c-105">Dit proces zorgt voor flexibel en schaalbaar Cloud hulpprogramma wanneer het wordt geladen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="7e49c-106">Met dit patroon is het mogelijk dat uw Tenant niet gereed is om uw app uit te voeren in de open bare Cloud.</span><span class="sxs-lookup"><span data-stu-id="7e49c-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="7e49c-107">Het is echter mogelijk niet economisch haalbaar voor het bedrijf om de capaciteit te behouden die vereist is in hun on-premises omgeving voor het afhandelen van pieken in de vraag naar de app.</span><span class="sxs-lookup"><span data-stu-id="7e49c-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="7e49c-108">Uw Tenant kan gebruikmaken van de elasticiteit van de open bare Cloud met hun lokale oplossing.</span><span class="sxs-lookup"><span data-stu-id="7e49c-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="7e49c-109">In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:</span><span class="sxs-lookup"><span data-stu-id="7e49c-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="7e49c-110">Een web-app met meerdere knoop punten maken.</span><span class="sxs-lookup"><span data-stu-id="7e49c-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="7e49c-111">Het proces voor doorlopende implementatie (CD) configureren en beheren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="7e49c-112">Publiceer de web-app naar Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="7e49c-113">Maak een release.</span><span class="sxs-lookup"><span data-stu-id="7e49c-113">Create a release.</span></span>
> - <span data-ttu-id="7e49c-114">Meer informatie over het bewaken en bijhouden van uw implementaties.</span><span class="sxs-lookup"><span data-stu-id="7e49c-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="7e49c-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="7e49c-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="7e49c-116">Microsoft Azure Stack hub is een uitbrei ding van Azure.</span><span class="sxs-lookup"><span data-stu-id="7e49c-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="7e49c-117">Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt gebruiken waarmee u overal hybride apps bouwt en implementeert.</span><span class="sxs-lookup"><span data-stu-id="7e49c-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="7e49c-118">In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps.</span><span class="sxs-lookup"><span data-stu-id="7e49c-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="7e49c-119">De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.</span><span class="sxs-lookup"><span data-stu-id="7e49c-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="7e49c-120">Vereisten</span><span class="sxs-lookup"><span data-stu-id="7e49c-120">Prerequisites</span></span>

- <span data-ttu-id="7e49c-121">Azure-abonnement.</span><span class="sxs-lookup"><span data-stu-id="7e49c-121">Azure subscription.</span></span> <span data-ttu-id="7e49c-122">Maak, indien nodig, een [gratis account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) voordat u begint.</span><span class="sxs-lookup"><span data-stu-id="7e49c-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="7e49c-123">Een Azure Stack hub geïntegreerd systeem of implementatie van Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="7e49c-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="7e49c-124">Zie [install the ASDK](/azure-stack/asdk/asdk-install.md)(Engelstalig) voor instructies over het installeren van Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="7e49c-125">Voor een ASDK-automatiserings script na de implementatie gaat u naar: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="7e49c-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="7e49c-126">Het volt ooien van deze installatie kan enkele uren in beslag nemen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="7e49c-127">Implementeer [app service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services op Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="7e49c-128">[Maak plannen/aanbiedingen](/azure-stack/operator/service-plan-offer-subscription-overview.md) binnen de Azure stack hub-omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="7e49c-129">Een [Tenant abonnement maken](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) binnen de Azure stack hub-omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="7e49c-130">Een web-app maken binnen het Tenant abonnement.</span><span class="sxs-lookup"><span data-stu-id="7e49c-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="7e49c-131">Noteer de URL van de nieuwe web-app voor later gebruik.</span><span class="sxs-lookup"><span data-stu-id="7e49c-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="7e49c-132">Implementeer Azure pipelines virtual machine (VM) binnen het Tenant abonnement.</span><span class="sxs-lookup"><span data-stu-id="7e49c-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="7e49c-133">VM van Windows Server 2016 met .NET 3,5 is vereist.</span><span class="sxs-lookup"><span data-stu-id="7e49c-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="7e49c-134">Deze VM wordt gebouwd in het Tenant abonnement op Azure Stack hub als de Private build agent.</span><span class="sxs-lookup"><span data-stu-id="7e49c-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="7e49c-135">[Windows Server 2016 met SQL 2017 VM-installatie kopie](/azure-stack/operator/azure-stack-add-vm-image.md) is beschikbaar op de Azure stack hub Marketplace.</span><span class="sxs-lookup"><span data-stu-id="7e49c-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="7e49c-136">Als deze installatie kopie niet beschikbaar is, werkt u met een Azure Stack hub-operator om te controleren of deze is toegevoegd aan de omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="7e49c-137">Problemen en overwegingen</span><span class="sxs-lookup"><span data-stu-id="7e49c-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="7e49c-138">Schaalbaarheid</span><span class="sxs-lookup"><span data-stu-id="7e49c-138">Scalability</span></span>

<span data-ttu-id="7e49c-139">Het belangrijkste onderdeel van cross-Cloud schaling is de mogelijkheid om direct en op aanvraag schalen te leveren tussen de open bare en on-premises Cloud infrastructuur, zodat u een consistente en betrouw bare service kunt bieden.</span><span class="sxs-lookup"><span data-stu-id="7e49c-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="7e49c-140">Beschikbaarheid</span><span class="sxs-lookup"><span data-stu-id="7e49c-140">Availability</span></span>

<span data-ttu-id="7e49c-141">Zorg ervoor dat lokaal geïmplementeerde apps zijn geconfigureerd voor hoge Beschik baarheid via on-premises hardwareconfiguratie en software-implementatie.</span><span class="sxs-lookup"><span data-stu-id="7e49c-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="7e49c-142">Beheerbaarheid</span><span class="sxs-lookup"><span data-stu-id="7e49c-142">Manageability</span></span>

<span data-ttu-id="7e49c-143">De oplossing voor meerdere clouds zorgt voor naadloos beheer en de vertrouwde interface tussen omgevingen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="7e49c-144">Power shell wordt aanbevolen voor beheer op meerdere platforms.</span><span class="sxs-lookup"><span data-stu-id="7e49c-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="7e49c-145">Schalen in de cloud</span><span class="sxs-lookup"><span data-stu-id="7e49c-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="7e49c-146">Een aangepast domein ophalen en DNS configureren</span><span class="sxs-lookup"><span data-stu-id="7e49c-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="7e49c-147">Werk het DNS-zone bestand voor het domein bij.</span><span class="sxs-lookup"><span data-stu-id="7e49c-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="7e49c-148">Azure AD controleert het eigendom van de aangepaste domein naam.</span><span class="sxs-lookup"><span data-stu-id="7e49c-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="7e49c-149">Gebruik [Azure DNS](/azure/dns/dns-getstarted-portal) voor azure/Microsoft 365/externe DNS-records in azure, of Voeg de DNS-vermelding toe aan [een andere DNS-registratie](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="7e49c-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="7e49c-150">Een aangepast domein registreren bij een openbaar registratie service.</span><span class="sxs-lookup"><span data-stu-id="7e49c-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="7e49c-151">Meld u aan bij de domeinnaamregistrar voor het domein.</span><span class="sxs-lookup"><span data-stu-id="7e49c-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="7e49c-152">Een goedgekeurde beheerder kan verplicht zijn om DNS-updates uit te voeren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="7e49c-153">Werk het DNS-zone bestand voor het domein bij door de DNS-vermelding toe te voegen die door Azure AD wordt geleverd.</span><span class="sxs-lookup"><span data-stu-id="7e49c-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="7e49c-154">(De DNS-vermelding heeft geen invloed op het gedrag van e-mail routering of webhosting.)</span><span class="sxs-lookup"><span data-stu-id="7e49c-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="7e49c-155">Een standaard web-app met meerdere knoop punten maken in Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="7e49c-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="7e49c-156">Stel hybride continue integratie en continue implementatie (CI/CD) in voor het implementeren van web-apps naar Azure en Azure Stack hub en voor het autopushen van wijzigingen in beide Clouds.</span><span class="sxs-lookup"><span data-stu-id="7e49c-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="7e49c-157">Azure Stack hub met de juiste installatie kopieën die zijn syndicated om te worden uitgevoerd (Windows Server en SQL) en App Service implementatie is vereist.</span><span class="sxs-lookup"><span data-stu-id="7e49c-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="7e49c-158">Raadpleeg de App Service documentatie [vereisten voor het implementeren van app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="7e49c-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="7e49c-159">Code toevoegen aan Azure opslag plaatsen</span><span class="sxs-lookup"><span data-stu-id="7e49c-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="7e49c-160">Azure-opslagplaatsen</span><span class="sxs-lookup"><span data-stu-id="7e49c-160">Azure Repos</span></span>

1. <span data-ttu-id="7e49c-161">Meld u aan bij Azure opslag plaatsen met een account met rechten voor het maken van projecten op Azure opslag plaatsen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="7e49c-162">Hybride CI/CD kan worden toegepast op de code van de app en de infra structuur.</span><span class="sxs-lookup"><span data-stu-id="7e49c-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="7e49c-163">Gebruik [Azure Resource Manager sjablonen](https://azure.microsoft.com/resources/templates/) voor zowel privé als gehoste Cloud ontwikkeling.</span><span class="sxs-lookup"><span data-stu-id="7e49c-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Verbinding maken met een project in azure opslag plaatsen](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="7e49c-165">**Kloon de opslag plaats** door de standaard web-app te maken en te openen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Opslag plaats in azure-web-app klonen](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="7e49c-167">Zelf-opgenomen web-app-implementatie maken voor App Services in beide Clouds</span><span class="sxs-lookup"><span data-stu-id="7e49c-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="7e49c-168">Bewerk het bestand **webapplication. csproj** .</span><span class="sxs-lookup"><span data-stu-id="7e49c-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="7e49c-169">Selecteren `Runtimeidentifier` en toevoegen `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="7e49c-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="7e49c-170">(Zie [zelf-opgenomen implementatie](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentatie.)</span><span class="sxs-lookup"><span data-stu-id="7e49c-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Web-app-project bestand bewerken](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="7e49c-172">Check de code in op Azure opslag plaatsen met behulp van team Explorer.</span><span class="sxs-lookup"><span data-stu-id="7e49c-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="7e49c-173">Controleer of de app-code is ingecheckt in azure opslag plaatsen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="7e49c-174">De build-definitie maken</span><span class="sxs-lookup"><span data-stu-id="7e49c-174">Create the build definition</span></span>

1. <span data-ttu-id="7e49c-175">Meld u aan bij Azure-pijp lijnen om te bevestigen dat u build-definities wilt maken.</span><span class="sxs-lookup"><span data-stu-id="7e49c-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="7e49c-176">Add **-r win10-x64-** code.</span><span class="sxs-lookup"><span data-stu-id="7e49c-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="7e49c-177">Deze toevoeging is nood zakelijk om een zelfstandige implementatie met .NET core te activeren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Code toevoegen aan de web-app](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="7e49c-179">Voer de build uit.</span><span class="sxs-lookup"><span data-stu-id="7e49c-179">Run the build.</span></span> <span data-ttu-id="7e49c-180">In het [zelf opgenomen implementatie](/dotnet/core/deploying/deploy-with-vs#simpleSelf) proces worden artefacten gepubliceerd die worden uitgevoerd op Azure en Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="7e49c-181">Een gehoste agent van Azure gebruiken</span><span class="sxs-lookup"><span data-stu-id="7e49c-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="7e49c-182">Het gebruik van een gehoste build-agent in azure-pijp lijnen is een handige optie om web-apps te bouwen en te implementeren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="7e49c-183">Onderhoud en upgrades worden automatisch uitgevoerd door Microsoft Azure, waardoor een continue en ononderbroken ontwikkelings cyclus mogelijk is.</span><span class="sxs-lookup"><span data-stu-id="7e49c-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="7e49c-184">Het CD-proces beheren en configureren</span><span class="sxs-lookup"><span data-stu-id="7e49c-184">Manage and configure the CD process</span></span>

<span data-ttu-id="7e49c-185">Azure-pijp lijnen en Azure DevOps services bieden een zeer Configureer bare en beheersbaarere pijp lijn voor meerdere omgevingen, zoals ontwikkelings-, staging-, QA-en productie omgevingen; met inbegrip van het vereisen van goed keuringen in specifieke fasen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="7e49c-186">Release definitie maken</span><span class="sxs-lookup"><span data-stu-id="7e49c-186">Create release definition</span></span>

1. <span data-ttu-id="7e49c-187">Selecteer de knop met het **plus teken** om een nieuwe release toe te voegen op het tabblad **releases** in de sectie **Build and release** van Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="7e49c-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Release-definitie maken](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="7e49c-189">Pas de Azure App Service-implementatie sjabloon toe.</span><span class="sxs-lookup"><span data-stu-id="7e49c-189">Apply the Azure App Service Deployment template.</span></span>

   ![Azure App Service implementatie sjabloon Toep assen](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="7e49c-191">Voeg onder **artefact toevoegen**het artefact toe voor de Azure Cloud build-app.</span><span class="sxs-lookup"><span data-stu-id="7e49c-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Artefact toevoegen aan Azure Cloud build](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="7e49c-193">Selecteer onder pijplijn tabblad de **fase, taak** koppeling van de omgeving en stel de waarden van de Azure-cloud omgeving in.</span><span class="sxs-lookup"><span data-stu-id="7e49c-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Waarden van de Azure-cloud omgeving instellen](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="7e49c-195">Stel de **omgevings naam** in en selecteer het **Azure-abonnement** voor het Azure-Cloud eindpunt.</span><span class="sxs-lookup"><span data-stu-id="7e49c-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure-abonnement voor Azure-Cloud eindpunt selecteren](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="7e49c-197">Stel onder **app service name**de vereiste Azure app service-naam in.</span><span class="sxs-lookup"><span data-stu-id="7e49c-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure app service-naam instellen](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="7e49c-199">Voer ' gehoste VS2017 ' in de wachtrij van de **agent** in voor de gehoste Azure cloud-omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Agent wachtrij instellen voor gehoste Azure-cloud omgeving](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="7e49c-201">Selecteer in het menu implementeren Azure App Service het geldige **pakket of** de juiste map voor de omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="7e49c-202">Selecteer **OK** om **de maplocatie te**selecteren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-202">Select **OK** to **folder location**.</span></span>
  
      ![Pakket of map voor Azure App Service omgeving selecteren](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Pakket of map voor Azure App Service omgeving selecteren](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="7e49c-205">Sla alle wijzigingen op en ga terug naar de **release pijplijn**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Wijzigingen in de release pijplijn opslaan](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="7e49c-207">Voeg een nieuw artefact toe om de build voor de Azure Stack hub-app te selecteren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Nieuw artefact voor Azure Stack hub-app toevoegen](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="7e49c-209">Voeg nog een omgeving toe door de implementatie van Azure App Service toe te passen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Omgeving toevoegen aan Azure App Service-implementatie](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="7e49c-211">Geef de nieuwe omgeving de naam Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="7e49c-211">Name the new environment "Azure Stack".</span></span>

    ![Naam omgeving in Azure App Service-implementatie](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="7e49c-213">Zoek de Azure Stack omgeving onder **taak** tabblad.</span><span class="sxs-lookup"><span data-stu-id="7e49c-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack omgeving](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="7e49c-215">Selecteer het abonnement voor het Azure Stack-eind punt.</span><span class="sxs-lookup"><span data-stu-id="7e49c-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Het abonnement voor het Azure Stack-eind punt selecteren](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="7e49c-217">Stel de naam van de Azure Stack web-app in als de naam van de app service.</span><span class="sxs-lookup"><span data-stu-id="7e49c-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="7e49c-218">![Azure Stack naam van de web-app instellen](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="7e49c-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="7e49c-219">Selecteer de Azure Stack agent.</span><span class="sxs-lookup"><span data-stu-id="7e49c-219">Select the Azure Stack agent.</span></span>

    ![De Azure Stack-agent selecteren](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="7e49c-221">Selecteer onder de sectie implementatie Azure App Service het geldige **pakket of** de juiste map voor de omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="7e49c-222">Selecteer **OK** om de maplocatie te selecteren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-222">Select **OK** to folder location.</span></span>

    ![Selecteer een map voor Azure App Service implementatie](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selecteer een map voor Azure App Service implementatie](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="7e49c-225">Onder variabele tabblad voegt u een variabele `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` met de naam in, stelt u de waarde in op **True**en bereik Azure stack.</span><span class="sxs-lookup"><span data-stu-id="7e49c-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Variabele toevoegen aan Azure-app-implementatie](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="7e49c-227">Selecteer het trigger pictogram **continue** implementatie in beide artefacten en schakel de trigger **voor het implementeren** van de implementatie in.</span><span class="sxs-lookup"><span data-stu-id="7e49c-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Trigger voor continue implementatie selecteren](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="7e49c-229">Selecteer het pictogram voor waarden **voorafgaand** aan de implementatie in de Azure stack omgeving en stel de trigger in op **na release.**</span><span class="sxs-lookup"><span data-stu-id="7e49c-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Voor waarden voor installatie vooraf selecteren](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="7e49c-231">Sla alle wijzigingen op.</span><span class="sxs-lookup"><span data-stu-id="7e49c-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="7e49c-232">Sommige instellingen voor de taken zijn mogelijk automatisch gedefinieerd als [omgevings variabelen](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) bij het maken van een release definitie op basis van een sjabloon.</span><span class="sxs-lookup"><span data-stu-id="7e49c-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="7e49c-233">Deze instellingen kunnen niet worden gewijzigd in de taak instellingen. in plaats daarvan moet het bovenliggende omgevings item worden geselecteerd om deze instellingen te bewerken.</span><span class="sxs-lookup"><span data-stu-id="7e49c-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="7e49c-234">Publiceren naar Azure Stack hub via Visual Studio</span><span class="sxs-lookup"><span data-stu-id="7e49c-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="7e49c-235">Door eind punten te maken, kunt u met een Azure DevOps Services-build Azure-service-Apps implementeren op Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="7e49c-236">Azure-pijp lijnen maken verbinding met de build-agent, die verbinding maakt met Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="7e49c-237">Meld u aan bij Azure DevOps Services en ga naar de pagina app-instellingen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="7e49c-238">Selecteer bij **instellingen**de optie **beveiliging**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="7e49c-239">Selecteer in **VSTS-groepen**de optie **endpoint Crea tors**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="7e49c-240">Selecteer **toevoegen**op het tabblad **leden** .</span><span class="sxs-lookup"><span data-stu-id="7e49c-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="7e49c-241">In **gebruikers en groepen toevoegen**voert u een gebruikers naam in en selecteert u deze gebruiker in de lijst met gebruikers.</span><span class="sxs-lookup"><span data-stu-id="7e49c-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="7e49c-242">Selecteer **Save changes**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="7e49c-243">Selecteer in de lijst **VSTS groepen** de optie **endpoint Administrators**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="7e49c-244">Selecteer **toevoegen**op het tabblad **leden** .</span><span class="sxs-lookup"><span data-stu-id="7e49c-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="7e49c-245">In **gebruikers en groepen toevoegen**voert u een gebruikers naam in en selecteert u deze gebruiker in de lijst met gebruikers.</span><span class="sxs-lookup"><span data-stu-id="7e49c-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="7e49c-246">Selecteer **Save changes**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-246">Select **Save changes**.</span></span>

<span data-ttu-id="7e49c-247">Nu de gegevens van het eind punt bestaan, zijn de Azure-pijp lijnen naar Azure Stack hub-verbinding klaar voor gebruik.</span><span class="sxs-lookup"><span data-stu-id="7e49c-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="7e49c-248">De build agent in Azure Stack hub haalt instructies op uit Azure pipelines en vervolgens geeft de agent informatie over het eind punt voor communicatie met Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="7e49c-249">De app-build ontwikkelen</span><span class="sxs-lookup"><span data-stu-id="7e49c-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="7e49c-250">Azure Stack hub met de juiste installatie kopieën die zijn syndicated om te worden uitgevoerd (Windows Server en SQL) en App Service implementatie is vereist.</span><span class="sxs-lookup"><span data-stu-id="7e49c-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="7e49c-251">Zie [vereisten voor het implementeren van app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="7e49c-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="7e49c-252">Gebruik [Azure Resource Manager sjablonen](https://azure.microsoft.com/resources/templates/) zoals web-app-code van Azure opslag plaatsen om te implementeren op beide Clouds.</span><span class="sxs-lookup"><span data-stu-id="7e49c-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="7e49c-253">Code toevoegen aan een Azure opslag plaatsen-project</span><span class="sxs-lookup"><span data-stu-id="7e49c-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="7e49c-254">Meld u aan bij Azure opslag plaatsen met een account met rechten voor het maken van projecten op Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="7e49c-255">**Kloon de opslag plaats** door de standaard web-app te maken en te openen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="7e49c-256">Zelf-opgenomen web-app-implementatie maken voor App Services in beide Clouds</span><span class="sxs-lookup"><span data-stu-id="7e49c-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="7e49c-257">Bewerk het bestand **webapplication. csproj** : Selecteer `Runtimeidentifier` en vervolgens toevoegen `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="7e49c-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="7e49c-258">Zie voor meer informatie de documentatie over de [zelfstandige implementatie](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .</span><span class="sxs-lookup"><span data-stu-id="7e49c-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="7e49c-259">Gebruik team Explorer om de code in azure opslag plaatsen te controleren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="7e49c-260">Controleer of de app-code is ingecheckt in azure opslag plaatsen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="7e49c-261">De build-definitie maken</span><span class="sxs-lookup"><span data-stu-id="7e49c-261">Create the build definition</span></span>

1. <span data-ttu-id="7e49c-262">Meld u aan bij Azure-pijp lijnen met een account dat een build-definitie kan maken.</span><span class="sxs-lookup"><span data-stu-id="7e49c-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="7e49c-263">Ga naar de pagina **Build Web Application** voor het project.</span><span class="sxs-lookup"><span data-stu-id="7e49c-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="7e49c-264">In **argumenten**, add **-r win10-x64-** code.</span><span class="sxs-lookup"><span data-stu-id="7e49c-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="7e49c-265">Deze toevoeging is vereist voor het activeren van een zelfstandige implementatie met .NET core.</span><span class="sxs-lookup"><span data-stu-id="7e49c-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="7e49c-266">Voer de build uit.</span><span class="sxs-lookup"><span data-stu-id="7e49c-266">Run the build.</span></span> <span data-ttu-id="7e49c-267">In het [zelf opgenomen implementatie constructie](/dotnet/core/deploying/deploy-with-vs#simpleSelf) proces worden artefacten gepubliceerd die kunnen worden uitgevoerd op Azure en Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="7e49c-268">Een door Azure gehoste build-agent gebruiken</span><span class="sxs-lookup"><span data-stu-id="7e49c-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="7e49c-269">Het gebruik van een gehoste build-agent in azure-pijp lijnen is een handige optie om web-apps te bouwen en te implementeren.</span><span class="sxs-lookup"><span data-stu-id="7e49c-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="7e49c-270">Onderhoud en upgrades worden automatisch uitgevoerd door Microsoft Azure, waardoor een continue en ononderbroken ontwikkelings cyclus mogelijk is.</span><span class="sxs-lookup"><span data-stu-id="7e49c-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="7e49c-271">Het proces van de doorlopende implementatie (CD) configureren</span><span class="sxs-lookup"><span data-stu-id="7e49c-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="7e49c-272">Azure-pijp lijnen en Azure DevOps-services bieden een zeer Configureer bare en beheersbare pijp lijn voor de release van meerdere omgevingen, zoals ontwikkeling, fase ring, kwaliteits borging (QA) en productie.</span><span class="sxs-lookup"><span data-stu-id="7e49c-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="7e49c-273">Dit proces kan bestaan uit het vereisen van goed keuringen in specifieke fasen van de levens cyclus van de app.</span><span class="sxs-lookup"><span data-stu-id="7e49c-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="7e49c-274">Release definitie maken</span><span class="sxs-lookup"><span data-stu-id="7e49c-274">Create release definition</span></span>

<span data-ttu-id="7e49c-275">Het maken van een release definitie is de laatste stap in het app-bouw proces.</span><span class="sxs-lookup"><span data-stu-id="7e49c-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="7e49c-276">Deze release definitie wordt gebruikt voor het maken van een release en het implementeren van een build.</span><span class="sxs-lookup"><span data-stu-id="7e49c-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="7e49c-277">Meld u aan bij Azure-pijp lijnen en ga naar **Build en release** voor het project.</span><span class="sxs-lookup"><span data-stu-id="7e49c-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="7e49c-278">Op het tabblad **releases** selecteert u **[+]** en kiest u **release definitie maken**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="7e49c-279">Kies bij **een sjabloon selecteren de**optie **Azure app service-implementatie**en selecteer vervolgens **Toep assen**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="7e49c-280">Selecteer op **artefact toevoegen**van de **bron (build-definitie)** de Azure Cloud build-app.</span><span class="sxs-lookup"><span data-stu-id="7e49c-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="7e49c-281">Op het tabblad **pijp lijn** selecteert u de koppeling **1 fase**, **1 taak** om **omgevings taken weer te geven**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="7e49c-282">Op het tabblad **taken** voert u Azure in als de naam van de **omgeving** en selecteert u in de lijst met **Azure-abonnementen** de Cloud handelaren-Web EP.</span><span class="sxs-lookup"><span data-stu-id="7e49c-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="7e49c-283">Voer de naam in van de **Azure app service**, die zich `northwindtraders` in de volgende scherm opname bevindt.</span><span class="sxs-lookup"><span data-stu-id="7e49c-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="7e49c-284">Voor de agent fase selecteert u **gehoste VS2017** in de lijst **agent wachtrij** .</span><span class="sxs-lookup"><span data-stu-id="7e49c-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="7e49c-285">In **implementatie Azure app service**selecteert u het geldige **pakket of** de juiste map voor de omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="7e49c-286">Selecteer in **bestand of map selecteren**de optie **OK** naar **locatie**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="7e49c-287">Sla alle wijzigingen op en ga terug naar de **pijp lijn**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="7e49c-288">Selecteer op het tabblad **pijp lijn** de optie **artefact toevoegen**en kies het **NorthwindCloud handelaren-vat** in de lijst **bron (build Definition)** .</span><span class="sxs-lookup"><span data-stu-id="7e49c-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="7e49c-289">Voeg een andere omgeving toe aan **een sjabloon selecteren**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="7e49c-290">Kies **Azure app service-implementatie** en selecteer vervolgens **Toep assen**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="7e49c-291">Voer `Azure Stack Hub` in als de **naam**van de omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="7e49c-292">Zoek en selecteer op het tabblad **taken** Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="7e49c-293">Selecteer in de lijst met **Azure-abonnementen** **AzureStack handelaren-vaartuigen EP** voor het eind punt van de Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="7e49c-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="7e49c-294">Voer de naam van de Azure Stack hub-web-app in als de naam van de **app service**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="7e49c-295">Onder **selectie van agent**kiest u **AzureStack-b Douglas-FIR** in de lijst **agent wachtrij** .</span><span class="sxs-lookup"><span data-stu-id="7e49c-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="7e49c-296">Voor **implementatie Azure app service**selecteert u het geldige **pakket of** de juiste map voor de omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="7e49c-297">Selecteer in **bestand of map selecteren**de optie **OK** voor de **locatie**van de map.</span><span class="sxs-lookup"><span data-stu-id="7e49c-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="7e49c-298">Zoek op het tabblad **variabele** de variabele met de naam `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="7e49c-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="7e49c-299">Stel de waarde van de variabele in op **waar**en stel het bereik in op **Azure stack hub**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="7e49c-300">Op het tabblad **pijp lijn** selecteert u het **trigger pictogram continue implementatie** voor de NorthwindCloud handelaren-webartefacten en stelt u de **trigger voor continue implementatie** in op **ingeschakeld**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="7e49c-301">Doe hetzelfde voor het **NorthwindCloud-scheeps-** artefact.</span><span class="sxs-lookup"><span data-stu-id="7e49c-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="7e49c-302">Selecteer voor de Azure Stack hub-omgeving het pictogram **voor waarden voorafgaand aan implementatie** de trigger instellen op **na release**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="7e49c-303">Sla alle wijzigingen op.</span><span class="sxs-lookup"><span data-stu-id="7e49c-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="7e49c-304">Sommige instellingen voor release taken worden automatisch gedefinieerd als [omgevings variabelen](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) bij het maken van een release definitie op basis van een sjabloon.</span><span class="sxs-lookup"><span data-stu-id="7e49c-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="7e49c-305">Deze instellingen kunnen niet worden gewijzigd in de taak instellingen, maar kunnen worden gewijzigd in de bovenliggende omgevings items.</span><span class="sxs-lookup"><span data-stu-id="7e49c-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="7e49c-306">Een release maken</span><span class="sxs-lookup"><span data-stu-id="7e49c-306">Create a release</span></span>

1. <span data-ttu-id="7e49c-307">Open de lijst **release** op het tabblad **pipeline** en selecteer **release maken**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="7e49c-308">Voer een beschrijving in voor de release, Controleer of de juiste artefacten zijn geselecteerd en selecteer vervolgens **maken**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="7e49c-309">Na enkele ogen blikken wordt een banner weer gegeven dat aangeeft dat de nieuwe release is gemaakt en de release naam wordt weer gegeven als een koppeling.</span><span class="sxs-lookup"><span data-stu-id="7e49c-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="7e49c-310">Selecteer de koppeling om de pagina release overzicht weer te geven.</span><span class="sxs-lookup"><span data-stu-id="7e49c-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="7e49c-311">De pagina release overzicht bevat details over de release.</span><span class="sxs-lookup"><span data-stu-id="7e49c-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="7e49c-312">In de volgende scherm opname voor ' release-2 ' toont de sectie **omgevingen** de **Implementatie status** voor Azure als ' wordt uitgevoerd ', en de status voor Azure stack hub is geslaagd.</span><span class="sxs-lookup"><span data-stu-id="7e49c-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="7e49c-313">Wanneer de implementatie status voor de Azure-omgeving wordt gewijzigd in ' geslaagd ', wordt een banner weer gegeven dat aangeeft dat de release gereed is voor goed keuring.</span><span class="sxs-lookup"><span data-stu-id="7e49c-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="7e49c-314">Wanneer een implementatie in behandeling is of mislukt, wordt een pictogram met een blauw **(i)-** informatie weer gegeven.</span><span class="sxs-lookup"><span data-stu-id="7e49c-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="7e49c-315">Beweeg de muis aanwijzer over het pictogram om een pop-upvenster te zien met de reden voor de vertraging of de fout.</span><span class="sxs-lookup"><span data-stu-id="7e49c-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="7e49c-316">In andere weer gaven, zoals de lijst met releases, wordt ook een pictogram weer gegeven dat aangeeft dat goed keuring in behandeling is.</span><span class="sxs-lookup"><span data-stu-id="7e49c-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="7e49c-317">In het pop-upvenster voor dit pictogram ziet u de naam van de omgeving en meer informatie over de implementatie.</span><span class="sxs-lookup"><span data-stu-id="7e49c-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="7e49c-318">Het is eenvoudig voor een beheerder de algehele voortgang van releases te zien en te zien welke releases wachten op goed keuring.</span><span class="sxs-lookup"><span data-stu-id="7e49c-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="7e49c-319">Implementaties controleren en bijhouden</span><span class="sxs-lookup"><span data-stu-id="7e49c-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="7e49c-320">Selecteer op de pagina overzicht van **Release 2** de optie **Logboeken**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="7e49c-321">Tijdens een implementatie toont deze pagina het Live logboek van de agent.</span><span class="sxs-lookup"><span data-stu-id="7e49c-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="7e49c-322">In het linkerdeel venster ziet u de status van elke bewerking in de implementatie voor elke omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="7e49c-323">Selecteer het persoons pictogram in de kolom **actie** voor een goed keuring vóór de implementatie of na de implementatie om te zien wie de implementatie heeft goedgekeurd (of geweigerd) en welk bericht ze hebben geleverd.</span><span class="sxs-lookup"><span data-stu-id="7e49c-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="7e49c-324">Nadat de implementatie is voltooid, wordt het hele logboek bestand weer gegeven in het rechterdeel venster.</span><span class="sxs-lookup"><span data-stu-id="7e49c-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="7e49c-325">Selecteer een wille keurige **stap** in het linkerdeel venster om het logboek bestand voor één stap weer te geven, zoals de **opdracht initialiseren**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="7e49c-326">De mogelijkheid om afzonderlijke logboeken te bekijken maakt het gemakkelijker om delen van de algehele implementatie te traceren en fouten op te sporen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="7e49c-327">**Sla** het logboek bestand op voor een stap of **down load alle logboeken als zip**.</span><span class="sxs-lookup"><span data-stu-id="7e49c-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="7e49c-328">Open het tabblad **samen vatting** om algemene informatie over de release weer te geven.</span><span class="sxs-lookup"><span data-stu-id="7e49c-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="7e49c-329">Deze weer gave bevat details over de build, de omgevingen waarin deze is geïmplementeerd, de implementatie status en andere informatie over de release.</span><span class="sxs-lookup"><span data-stu-id="7e49c-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="7e49c-330">Selecteer een omgevings koppeling (**Azure** of **Azure stack hub**) om informatie over bestaande en in behandeling zijnde implementaties te bekijken in een specifieke omgeving.</span><span class="sxs-lookup"><span data-stu-id="7e49c-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="7e49c-331">Gebruik deze weer gaven als snelle manier om te controleren of dezelfde build in beide omgevingen is geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="7e49c-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="7e49c-332">Open de **geïmplementeerde productie-app** in een browser.</span><span class="sxs-lookup"><span data-stu-id="7e49c-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="7e49c-333">Open bijvoorbeeld de URL voor de website van Azure-app Services `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="7e49c-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="7e49c-334">Integratie van Azure en Azure Stack hub biedt een schaal bare oplossing voor meerdere clouds</span><span class="sxs-lookup"><span data-stu-id="7e49c-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="7e49c-335">Een flexibele en robuuste multi-Cloud service biedt gegevens beveiliging, back-up en redundantie, consistente en snelle Beschik baarheid, schaal bare opslag en distributie en geo-compatibele route ring.</span><span class="sxs-lookup"><span data-stu-id="7e49c-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="7e49c-336">Dit hand matig geactiveerde proces zorgt voor betrouw bare en efficiënte belasting wisseling tussen gehoste web-apps en onmiddellijke Beschik baarheid van cruciale gegevens.</span><span class="sxs-lookup"><span data-stu-id="7e49c-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="7e49c-337">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="7e49c-337">Next steps</span></span>

- <span data-ttu-id="7e49c-338">Zie [Cloud ontwerp patronen](/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.</span><span class="sxs-lookup"><span data-stu-id="7e49c-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
