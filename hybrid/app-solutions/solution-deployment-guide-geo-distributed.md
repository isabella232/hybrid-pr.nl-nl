---
title: Direct verkeer met een geografisch gedistribueerde app met behulp van Azure en Azure Stack hub
description: Meer informatie over het omleiden van verkeer naar specifieke eind punten met een geografisch gedistribueerde app-oplossing met behulp van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 9fa2c351d2c13d85fe1adb17a35e165de96ea2a2
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895428"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="3b0ad-103">Direct verkeer met een geografisch gedistribueerde app met behulp van Azure en Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="3b0ad-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3b0ad-104">Meer informatie over het omleiden van verkeer naar specifieke eind punten op basis van verschillende metrische gegevens met behulp van het patroon geo-gedistribueerde apps.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="3b0ad-105">Het maken van een Traffic Manager profiel met op geografische wijze gebaseerde route ring en eindpunt configuratie zorgt ervoor dat informatie wordt doorgestuurd naar eind punten op basis van regionale vereisten, zakelijke en internationale regelgeving en uw gegevens behoeften.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="3b0ad-106">In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3b0ad-107">Een geografisch gedistribueerde app maken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="3b0ad-108">Gebruik Traffic Manager om uw app te richten.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="3b0ad-109">Het patroon voor geo-gedistribueerde apps gebruiken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="3b0ad-110">Met het geo-gedistribueerde patroon omvat uw app regio's.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="3b0ad-111">U kunt de standaard instelling voor de open bare Cloud, maar sommige gebruikers kunnen vereisen dat hun gegevens in hun regio blijven.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="3b0ad-112">U kunt gebruikers naar de meest geschikte Cloud sturen op basis van hun vereisten.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="3b0ad-113">Problemen en overwegingen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="3b0ad-114">Schaalbaarheidsoverwegingen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-114">Scalability considerations</span></span>

<span data-ttu-id="3b0ad-115">De oplossing die u met dit artikel bouwt, is niet geschikt voor schaal baarheid.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="3b0ad-116">Als u echter gebruikt in combi natie met andere Azure-en on-premises oplossingen, kunt u aan de schaalbaarheids vereisten voldoen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="3b0ad-117">Zie voor meer informatie over het maken van een hybride oplossing met automatisch schalen via Traffic Manager [oplossingen voor cross-Cloud schalen met Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="3b0ad-118">Beschikbaarheidsoverwegingen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-118">Availability considerations</span></span>

<span data-ttu-id="3b0ad-119">Net als bij schaal baarheids overwegingen wordt deze oplossing niet rechtstreeks op de hoogte van de beschik baarheid.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="3b0ad-120">Azure-en on-premises oplossingen kunnen echter in deze oplossing worden geïmplementeerd om hoge Beschik baarheid voor alle betrokken onderdelen te garanderen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="3b0ad-121">Wanneer dit patroon gebruiken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-121">When to use this pattern</span></span>

- <span data-ttu-id="3b0ad-122">Uw organisatie heeft internationale vertakkingen waarvoor aangepaste regionale beveiligings-en distributie beleidsregels zijn vereist.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="3b0ad-123">Alle kant oren van uw organisatie halen werk nemer-, bedrijfs-en faciliteit gegevens, waarvoor rapportage activiteiten per lokale regelgeving en tijd zones vereist zijn.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="3b0ad-124">Aan hoge schaal vereisten wordt voldaan door apps horizon taal te schalen met meerdere app-implementaties binnen één regio en in verschillende regio's om extreem belasting vereisten af te handelen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="3b0ad-125">De topologie plannen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-125">Planning the topology</span></span>

<span data-ttu-id="3b0ad-126">Voordat u een gedistribueerde app-footprint maakt, is het handig om de volgende zaken te kennen:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="3b0ad-127">**Aangepast domein voor de app:** Wat is de aangepaste domein naam die klanten gebruiken om toegang te krijgen tot de app?</span><span class="sxs-lookup"><span data-stu-id="3b0ad-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="3b0ad-128">Voor de voor beeld-app is de aangepaste domein naam *www- \. scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="3b0ad-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="3b0ad-129">**Traffic Manager domein:** Er wordt een domein naam gekozen bij het maken van een [Azure Traffic Manager-profiel](/azure/traffic-manager/traffic-manager-manage-profiles).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="3b0ad-130">Deze naam wordt gecombineerd met het *trafficmanager.net* -achtervoegsel voor het registreren van een domein vermelding die wordt beheerd door Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="3b0ad-131">Voor de voor beeld-app is de gekozen naam *schaalbaar-ASE-demo*.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="3b0ad-132">Als gevolg hiervan is de volledige domein naam die wordt beheerd door Traffic Manager, *Scalable-ASE-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="3b0ad-133">**Strategie voor het schalen van de app-footprint:** Bepaal of het gebruik van de app wordt gedistribueerd over meerdere App Service omgevingen in één regio, meerdere regio's of een combi natie van beide benaderingen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="3b0ad-134">De beslissing moet worden gebaseerd op de verwachtingen van waar klant verkeer van oorsprong is en hoe goed de rest van de ondersteunende back-end-infra structuur van een app kan worden geschaald.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="3b0ad-135">Een app kan bijvoorbeeld met een staatloze app van 100% worden geschaald met behulp van een combi natie van meerdere App Service omgevingen per Azure-regio, vermenigvuldigd met App Service omgevingen die zijn geïmplementeerd in meerdere Azure-regio's.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="3b0ad-136">Met 15 + wereld wijde Azure-regio's die beschikbaar zijn voor keuze, kunnen klanten echt een hele wereld voor Hyper-Scale-apps bouwen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="3b0ad-137">Voor de voor beeld-app die hier wordt gebruikt, zijn er drie App Service omgevingen gemaakt in één Azure-regio (Zuid-Centraal).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="3b0ad-138">**Naam Conventie voor de app service omgevingen:** Elke App Service omgeving moet een unieke naam hebben.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="3b0ad-139">Naast een of twee App Service omgevingen is het handig om een naam Conventie te hebben om elke App Service omgeving te identificeren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="3b0ad-140">Voor de voor beeld-app die hier wordt gebruikt, is een eenvoudige naam conventie gebruikt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="3b0ad-141">De namen van de drie App Service omgevingen zijn *fe1ase*, *fe2ase* en *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="3b0ad-142">**Naam Conventie voor de apps:** Omdat er meerdere exemplaren van de app worden geïmplementeerd, is een naam vereist voor elk exemplaar van de geïmplementeerde app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="3b0ad-143">Met App Service Environment voor Power apps kan dezelfde app-naam worden gebruikt in meerdere omgevingen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="3b0ad-144">Omdat elke App Service omgeving een uniek domein achtervoegsel heeft, kunnen ontwikkel aars ervoor kiezen om de exacte dezelfde app-naam in elke omgeving te gebruiken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="3b0ad-145">Een ontwikkelaar kan bijvoorbeeld apps als volgt hebben met de naam: *MyApp.foo1.p.azurewebsites.net*, *MyApp.foo2.p.azurewebsites.net*, *MyApp.foo3.p.azurewebsites.net*, enzovoort.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="3b0ad-146">Voor de app die hier wordt gebruikt, heeft elk exemplaar van de app een unieke naam.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="3b0ad-147">De namen van de app-exemplaren die worden gebruikt, zijn *webfrontend1*, *webfrontend2* en *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="3b0ad-148">![Diagram hybride pijlers](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3b0ad-148">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3b0ad-149">Microsoft Azure Stack hub is een uitbrei ding van Azure.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3b0ad-150">Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3b0ad-151">In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3b0ad-152">De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="3b0ad-153">Deel 1: een geografisch gedistribueerde app maken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="3b0ad-154">In dit gedeelte maakt u een web-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3b0ad-155">Web-apps maken en publiceren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="3b0ad-156">Voeg code toe aan Azure opslag plaatsen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="3b0ad-157">Stel in dat de app op meerdere Cloud doelen is gebouwd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="3b0ad-158">Het CD-proces beheren en configureren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="3b0ad-159">Vereisten</span><span class="sxs-lookup"><span data-stu-id="3b0ad-159">Prerequisites</span></span>

<span data-ttu-id="3b0ad-160">Een Azure-abonnement en een installatie van Azure Stack hub zijn vereist.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="3b0ad-161">Stappen voor geografisch gedistribueerde apps</span><span class="sxs-lookup"><span data-stu-id="3b0ad-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="3b0ad-162">Een aangepast domein verkrijgen en DNS configureren</span><span class="sxs-lookup"><span data-stu-id="3b0ad-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="3b0ad-163">Werk het DNS-zone bestand voor het domein bij.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="3b0ad-164">Azure AD kan vervolgens het eigendom van de aangepaste domein naam verifiëren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="3b0ad-165">Gebruik [Azure DNS](/azure/dns/dns-getstarted-portal) voor azure/Microsoft 365/externe DNS-records in azure, of Voeg de DNS-vermelding toe aan [een andere DNS-registratie](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="3b0ad-166">Een aangepast domein registreren bij een openbaar registratie service.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="3b0ad-167">Meld u aan bij de domeinnaamregistrar voor het domein.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="3b0ad-168">Een goedgekeurde beheerder kan verplicht zijn de DNS-updates uit te voeren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="3b0ad-169">Werk het DNS-zone bestand voor het domein bij door de DNS-vermelding toe te voegen die door Azure AD wordt geleverd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="3b0ad-170">Het wijzigen van de DNS-vermelding heeft geen invloed op het gedrag zoals mail routering of webhosting.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="3b0ad-171">Web-apps maken en publiceren</span><span class="sxs-lookup"><span data-stu-id="3b0ad-171">Create web apps and publish</span></span>

<span data-ttu-id="3b0ad-172">Stel hybride continue integratie/continue levering (CI/CD) in om de web-app te implementeren in Azure en Azure Stack hub, en automatisch push wijzigingen door te voeren naar beide Clouds.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="3b0ad-173">Azure Stack hub met de juiste installatie kopieën die zijn syndicated om te worden uitgevoerd (Windows Server en SQL) en App Service implementatie is vereist.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="3b0ad-174">Zie [vereisten voor het implementeren van app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="3b0ad-175">Code toevoegen aan Azure opslag plaatsen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="3b0ad-176">Meld u aan bij Visual Studio met een **account met rechten** voor het maken van projecten op Azure opslag plaatsen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="3b0ad-177">CI/CD kan worden toegepast op de code van de app en de infra structuur.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="3b0ad-178">Gebruik [Azure Resource Manager sjablonen](https://azure.microsoft.com/resources/templates/) voor zowel privé als gehoste Cloud ontwikkeling.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Verbinding maken met een project in Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="3b0ad-180">**Kloon de opslag plaats** door de standaard web-app te maken en te openen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Opslag plaats klonen in Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="3b0ad-182">Implementatie van web-app in beide clouds maken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="3b0ad-183">Bewerk het bestand **webapplication. csproj** : selecteren `Runtimeidentifier` en toevoegen `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="3b0ad-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="3b0ad-184">(Zie [zelf-opgenomen implementatie](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentatie.)</span><span class="sxs-lookup"><span data-stu-id="3b0ad-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Web-app-project bestand bewerken in Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="3b0ad-186">**Check de code in op Azure opslag plaatsen** met behulp van team Explorer.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="3b0ad-187">Controleer of de **toepassings code** is ingecheckt in azure opslag plaatsen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="3b0ad-188">De build-definitie maken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-188">Create the build definition</span></span>

1. <span data-ttu-id="3b0ad-189">**Meld u aan bij Azure-pijp lijnen** om te bevestigen dat u de bouw definities wilt maken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="3b0ad-190">Voeg `-r win10-x64` code toe.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="3b0ad-191">Deze toevoeging is nood zakelijk om een zelfstandige implementatie met .NET core te activeren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Code toevoegen aan de build-definitie in azure-pijp lijnen](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="3b0ad-193">**Voer de build uit**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-193">**Run the build**.</span></span> <span data-ttu-id="3b0ad-194">In het [zelf opgenomen implementatie constructie](/dotnet/core/deploying/deploy-with-vs#simpleSelf) proces worden artefacten gepubliceerd die kunnen worden uitgevoerd op Azure en Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="3b0ad-195">Een door Azure gehoste agent gebruiken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="3b0ad-196">Het gebruik van een gehoste agent in azure-pijp lijnen is een handige optie om web-apps te bouwen en te implementeren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="3b0ad-197">Onderhoud en upgrades worden automatisch uitgevoerd door Microsoft Azure, waardoor het mogelijk is om de ontwikkeling, het testen en de implementatie te onderbreken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="3b0ad-198">Het CD-proces beheren en configureren</span><span class="sxs-lookup"><span data-stu-id="3b0ad-198">Manage and configure the CD process</span></span>

<span data-ttu-id="3b0ad-199">Azure DevOps Services voorziet in een zeer Configureer bare en beheersbare pijp lijn voor releases in meerdere omgevingen, zoals ontwikkelings-, staging-, QA-en productie omgevingen; met inbegrip van het vereisen van goed keuringen in specifieke fasen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="3b0ad-200">Release definitie maken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-200">Create release definition</span></span>

1. <span data-ttu-id="3b0ad-201">Selecteer de knop met het **plus teken** om een nieuwe release toe te voegen op het tabblad **releases** in de sectie **Build and release** van Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Een release definitie maken in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="3b0ad-203">Pas de Azure App Service-implementatie sjabloon toe.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-203">Apply the Azure App Service Deployment template.</span></span>

   ![Azure App Service implementatie sjabloon Toep assen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="3b0ad-205">Voeg onder **artefact toevoegen** het artefact toe voor de Azure Cloud build-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Artefact toevoegen aan Azure Cloud build in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="3b0ad-207">Selecteer onder pijplijn tabblad de **fase, taak** koppeling van de omgeving en stel de waarden van de Azure-cloud omgeving in.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Waarden van de Azure-cloud omgeving instellen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="3b0ad-209">Stel de **omgevings naam** in en selecteer het **Azure-abonnement** voor het Azure-Cloud eindpunt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selecteer Azure-abonnement voor Azure Cloud-eind punt in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="3b0ad-211">Stel onder **app service name** de vereiste Azure app service-naam in.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure app service-naam instellen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="3b0ad-213">Voer ' gehoste VS2017 ' in de wachtrij van de **agent** in voor de gehoste Azure cloud-omgeving.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Agent wachtrij instellen voor gehoste Azure-cloud omgeving in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="3b0ad-215">Selecteer in het menu implementeren Azure App Service het geldige **pakket of** de juiste map voor de omgeving.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="3b0ad-216">Selecteer **OK** om **de maplocatie te** selecteren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-216">Select **OK** to **folder location**.</span></span>
  
      ![Selecteer pakket of map voor Azure App Service omgeving in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Dialoog venster voor het kiezen van mappen 1](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="3b0ad-219">Sla alle wijzigingen op en ga terug naar de **release pijplijn**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Wijzigingen in de release pijplijn in azure DevOps services opslaan](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="3b0ad-221">Voeg een nieuw artefact toe om de build voor de Azure Stack hub-app te selecteren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Nieuwe artefacten voor Azure Stack hub-app toevoegen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="3b0ad-223">Voeg nog een omgeving toe door de implementatie van Azure App Service toe te passen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Omgeving toevoegen aan Azure App Service-implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="3b0ad-225">Geef de nieuwe omgeving een naam Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-225">Name the new environment Azure Stack Hub.</span></span>

    ![Naam omgeving in Azure App Service-implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="3b0ad-227">Zoek de Azure Stack hub-omgeving onder **taak** tabblad.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure Stack hub-omgeving in azure DevOps Services in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="3b0ad-229">Selecteer het abonnement voor het eind punt van de Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Selecteer het abonnement voor het Azure Stack hub-eind punt in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="3b0ad-231">Stel de naam van de Azure Stack hub-web-app in als de naam van de app service.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Azure Stack hub-web-app-naam instellen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="3b0ad-233">Selecteer de Azure Stack hub-agent.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-233">Select the Azure Stack Hub agent.</span></span>

    ![Selecteer de Azure Stack hub-agent in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="3b0ad-235">Selecteer onder de sectie implementatie Azure App Service het geldige **pakket of** de juiste map voor de omgeving.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="3b0ad-236">Selecteer **OK** om de maplocatie te selecteren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-236">Select **OK** to folder location.</span></span>

    ![Selecteer een map voor Azure App Service implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Dialoog venster voor het kiezen van mappen 2](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="3b0ad-239">Onder op het tabblad variabele voegt u een variabele met de naam in `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , stelt u de waarde in op **True** en bereik Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Een variabele toevoegen aan Azure-app-implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="3b0ad-241">Selecteer het trigger pictogram **continue** implementatie in beide artefacten en schakel de trigger **voor het implementeren** van de implementatie in.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Trigger voor continue implementatie selecteren in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="3b0ad-243">Selecteer het pictogram voor waarden **voorafgaand** aan de implementatie in de Azure stack hub-omgeving en stel de trigger in op **na release.**</span><span class="sxs-lookup"><span data-stu-id="3b0ad-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Voor waarden voor voor bereiding selecteren in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="3b0ad-245">Sla alle wijzigingen op.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="3b0ad-246">Sommige instellingen voor de taken zijn mogelijk automatisch gedefinieerd als [omgevings variabelen](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) bij het maken van een release definitie op basis van een sjabloon.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="3b0ad-247">Deze instellingen kunnen niet worden gewijzigd in de taak instellingen. in plaats daarvan moet het bovenliggende omgevings item worden geselecteerd om deze instellingen te bewerken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="3b0ad-248">Deel 2: opties voor Web-Apps bijwerken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-248">Part 2: Update web app options</span></span>

<span data-ttu-id="3b0ad-249">[Azure App Service](/azure/app-service/overview) biedt een uiterst schaalbare webhostingservice met self-patchfunctie.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="3b0ad-251">Een bestaande aangepaste DNS-naam toewijzen aan Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="3b0ad-252">Gebruik een **CNAME-record** en een **a-record** om een aangepaste DNS-naam toe te wijzen aan app service.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="3b0ad-253">Een bestaande aangepaste DNS-naam toewijzen aan Azure Web Apps</span><span class="sxs-lookup"><span data-stu-id="3b0ad-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="3b0ad-254">Gebruik een CNAME voor alle aangepaste DNS-namen, met uitzonde ring van een hoofd domein (bijvoorbeeld northwind.com).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="3b0ad-255">Zie voor het migreren van een live site en de DNS-domeinnaam naar App Service, [Een actieve DNS-naam migreren naar Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="3b0ad-256">Vereisten</span><span class="sxs-lookup"><span data-stu-id="3b0ad-256">Prerequisites</span></span>

<span data-ttu-id="3b0ad-257">Voor het volt ooien van deze oplossing:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-257">To complete this solution:</span></span>

- <span data-ttu-id="3b0ad-258">[Maak een app service-app](/azure/app-service/)of gebruik een app die is gemaakt voor een andere oplossing.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="3b0ad-259">Koop een domein naam en zorg ervoor dat u toegang tot het DNS-REGI ster voor de domein provider hebt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="3b0ad-260">Werk het DNS-zone bestand voor het domein bij.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="3b0ad-261">Azure AD controleert het eigendom van de aangepaste domein naam.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="3b0ad-262">Gebruik [Azure DNS](/azure/dns/dns-getstarted-portal) voor azure/Microsoft 365/externe DNS-records in azure, of Voeg de DNS-vermelding toe aan [een andere DNS-registratie](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="3b0ad-263">Een aangepast domein registreren bij een openbaar registratie service.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="3b0ad-264">Meld u aan bij de domeinnaamregistrar voor het domein.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="3b0ad-265">(Een goedgekeurde beheerder kan verplicht zijn om DNS-updates te maken.)</span><span class="sxs-lookup"><span data-stu-id="3b0ad-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="3b0ad-266">Werk het DNS-zone bestand voor het domein bij door de DNS-vermelding toe te voegen die door Azure AD wordt geleverd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="3b0ad-267">Als u bijvoorbeeld DNS-vermeldingen wilt toevoegen voor northwindcloud.com en www \. northwindcloud.com, configureert u DNS-instellingen voor het hoofd domein northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="3b0ad-268">U kunt een domein naam aanschaffen met behulp van de [Azure Portal](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="3b0ad-269">Om een aangepaste DNS-naam toe te wijzen aan een web-app, moet het [App Service-plan](https://azure.microsoft.com/pricing/details/app-service/) van de web-app een betaalde categorie zijn (**Shared**, **Basic**, **Standard** of **Premium**).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="3b0ad-270">CNAME-en A-records maken en toewijzen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="3b0ad-271">Toegang tot DNS-records via domeinprovider</span><span class="sxs-lookup"><span data-stu-id="3b0ad-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="3b0ad-272">Gebruik Azure DNS om een aangepaste DNS-naam te configureren voor Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="3b0ad-273">Zie [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain) (Azure DNS gebruiken om aangepaste domeininstellingen te verstrekken voor een Azure-service) voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="3b0ad-274">Meld u aan bij de website van de hoofd provider.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="3b0ad-275">Ga naar de pagina voor het beheren van DNS-records.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="3b0ad-276">Elke domein provider heeft zijn eigen DNS-record interface.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="3b0ad-277">Doorgaans heeft het sitegedeelte waar u moet zijn, een naam als **Domain Name**, **DNS** of **Name Server Management**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="3b0ad-278">De pagina DNS-records kan worden weer gegeven in **mijn domeinen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="3b0ad-279">Zoek de koppeling met de naam **zone bestand**, **DNS-records** of **Geavanceerde configuratie**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="3b0ad-280">In de schermafbeelding hieronder wordt een voorbeeld van een pagina met DNS-records weergegeven:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-280">The following screenshot is an example of a DNS records page:</span></span>

![Voorbeeld van een pagina met DNS-records](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="3b0ad-282">Selecteer in domein naam registratie **toevoegen of maken** om een record te maken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="3b0ad-283">Sommige providers hebben afzonderlijke links voor verschillende typen records.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="3b0ad-284">Raadpleeg de documentatie van de provider.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="3b0ad-285">Voeg een CNAME-record toe om een subdomein toe te wijzen aan de standaard-hostnaam van de app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="3b0ad-286">Voeg voor het \. voor beeld van het www northwindcloud.com-domein een CNAME-record toe die de naam toewijst aan `<app_name>.azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="3b0ad-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="3b0ad-287">Nadat u de CNAME hebt toegevoegd, ziet de pagina met DNS-records eruit als in het volgende voor beeld:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Navigatie naar Azure-app in de portal](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="3b0ad-289">De toewijzing van het CNAME-record in Azure inschakelen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="3b0ad-290">Meld u op een nieuw tabblad aan bij de Azure Portal.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="3b0ad-291">Ga naar App Services.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-291">Go to App Services.</span></span>

3. <span data-ttu-id="3b0ad-292">Selecteer Web-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-292">Select web app.</span></span>

4. <span data-ttu-id="3b0ad-293">Selecteer in het linkernavigatievenster van de app-pagina in de Azure portal **Aangepaste domeinen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="3b0ad-294">Selecteer het **+** pictogram naast **hostnaam toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="3b0ad-295">Typ de Fully Qualified Domain Name, zoals `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="3b0ad-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="3b0ad-296">Selecteer **Valideren**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-296">Select **Validate**.</span></span>

8. <span data-ttu-id="3b0ad-297">Indien aangegeven, voegt u aanvullende records van andere typen ( `A` of `TXT` ) toe aan de domein naam registratie-DNS-records.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="3b0ad-298">De waarden en typen van deze records worden door Azure verstrekt:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="3b0ad-299">a.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-299">a.</span></span>  <span data-ttu-id="3b0ad-300">Een **A**-record toewijzen aan het IP-adres van de app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="3b0ad-301">b.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-301">b.</span></span>  <span data-ttu-id="3b0ad-302">Een **TXT**-record toewijzen aan de standaardhostnaam `<app_name>.azurewebsites.net` van de app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="3b0ad-303">App Service gebruikt deze record alleen op configuratie tijd om het eigendom van het aangepaste domein te verifiëren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="3b0ad-304">Na de verificatie verwijdert u de TXT-record.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="3b0ad-305">Voltooi deze taak op het tabblad domein registratie en pas de validatie opnieuw uit tot de knop **hostnaam toevoegen** wordt geactiveerd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="3b0ad-306">Zorg ervoor dat het **hostnaam-record type** is ingesteld op **CNAME** (www.example.com of een wille keurig subdomein).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="3b0ad-307">Selecteer **Hostnaam toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="3b0ad-308">Typ de Fully Qualified Domain Name, zoals `northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="3b0ad-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="3b0ad-309">Selecteer **Valideren**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-309">Select **Validate**.</span></span> <span data-ttu-id="3b0ad-310">De **toevoegen** is geactiveerd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="3b0ad-311">Zorg ervoor dat het **hostnaam-record type** is ingesteld op **een record** (example.com).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="3b0ad-312">**Hostnaam toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-312">**Add hostname**.</span></span>

    <span data-ttu-id="3b0ad-313">Het kan enige tijd duren voordat de nieuwe hostnamen worden weer gegeven op de pagina **aangepaste domeinen** van de app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="3b0ad-314">Vernieuw de browser voor om de gegevens bij te werken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-314">Try refreshing the browser to update the data.</span></span>
  
    ![Aangepaste domeinen](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="3b0ad-316">Als er een fout optreedt, wordt aan de onderkant van de pagina een melding over een verificatie fout weer gegeven.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Domein verificatie fout](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="3b0ad-318">De bovenstaande stappen kunnen worden herhaald om een Joker teken domein ( \* . northwindcloud.com) toe te wijzen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="3b0ad-319">Hiermee kunnen extra subdomeinen worden toegevoegd aan deze app service zonder dat hiervoor een afzonderlijke CNAME-record hoeft te worden gemaakt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="3b0ad-320">Volg de registratie-instructies voor het configureren van deze instelling.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="3b0ad-321">Testen in een browser</span><span class="sxs-lookup"><span data-stu-id="3b0ad-321">Test in a browser</span></span>

<span data-ttu-id="3b0ad-322">Blader naar de DNS-naam (s) die u eerder hebt geconfigureerd (bijvoorbeeld `northwindcloud.com` of `www.northwindcloud.com` ).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="3b0ad-323">Deel 3: een aangepast SSL-certificaat binden</span><span class="sxs-lookup"><span data-stu-id="3b0ad-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="3b0ad-324">In dit gedeelte gaan we het volgende doen:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3b0ad-325">Koppel het aangepaste SSL-certificaat aan App Service.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="3b0ad-326">HTTPS afdwingen voor de app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="3b0ad-327">SSL-certificaat binding met scripts automatiseren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="3b0ad-328">Als dat nodig is, kunt u een SSL-certificaat van de klant verkrijgen in de Azure Portal en het binden aan de web-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="3b0ad-329">Zie de [zelf studie over app service-certificaten](/azure/app-service/web-sites-purchase-ssl-web-site)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="3b0ad-330">Vereisten</span><span class="sxs-lookup"><span data-stu-id="3b0ad-330">Prerequisites</span></span>

<span data-ttu-id="3b0ad-331">Voor het volt ooien van deze oplossing:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-331">To complete this  solution:</span></span>

- [<span data-ttu-id="3b0ad-332">Een App Service-app maken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="3b0ad-333">Wijs een aangepaste DNS-naam toe aan uw web-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="3b0ad-334">Haal een SSL-certificaat van een vertrouwde certificerings instantie op en gebruik de sleutel om de aanvraag te ondertekenen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="3b0ad-335">Vereisten voor uw SSL-certificaat</span><span class="sxs-lookup"><span data-stu-id="3b0ad-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="3b0ad-336">Als u een certificaat in App Service wilt gebruiken, moet het certificaat aan de volgende vereisten voldoen:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="3b0ad-337">Ondertekend door een vertrouwde certificerings instantie.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="3b0ad-338">Geëxporteerd als een PFX-bestand dat met een wacht woord is beveiligd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="3b0ad-339">Bevat een persoonlijke sleutel van ten minste 2048 bits lang.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="3b0ad-340">Bevat alle tussenliggende certificaten in de certificaat keten.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="3b0ad-341">De **ECC-algoritmen** voor het uitvoeren van een elliptische curve werken met app service, maar zijn niet opgenomen in deze hand leiding.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="3b0ad-342">Raadpleeg een certificerings instantie voor hulp bij het maken van ECC-certificaten.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="3b0ad-343">De web-app voorbereiden</span><span class="sxs-lookup"><span data-stu-id="3b0ad-343">Prepare the web app</span></span>

<span data-ttu-id="3b0ad-344">Als u een aangepast SSL-certificaat aan de Web-App wilt koppelen, moet het [app service plan](https://azure.microsoft.com/pricing/details/app-service/) zich in de laag **Basic**, **Standard** of **Premium** bestaan.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="3b0ad-345">Aanmelden bij Azure</span><span class="sxs-lookup"><span data-stu-id="3b0ad-345">Sign in to Azure</span></span>

1. <span data-ttu-id="3b0ad-346">Open de [Azure Portal](https://portal.azure.com/) en ga naar de web-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="3b0ad-347">Selecteer in het menu links **app Services** en selecteer vervolgens de naam van de web-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Web-app in Azure Portal selecteren](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="3b0ad-349">Controleer de prijscategorie</span><span class="sxs-lookup"><span data-stu-id="3b0ad-349">Check the pricing tier</span></span>

1. <span data-ttu-id="3b0ad-350">Ga in de linkernavigatiebalk van de pagina Web-app naar het gedeelte **instellingen** en selecteer **omhoog schalen (app service plan)**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menu omhoog schalen in web-app](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="3b0ad-352">Zorg ervoor dat de web-app zich niet in de laag **gratis** of **gedeeld** bevindt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="3b0ad-353">De huidige laag van de web-app is gemarkeerd in een donker blauw vak.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![De prijs categorie in de web-app controleren](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="3b0ad-355">Aangepaste SSL wordt niet ondersteund in de laag **gratis** of **gedeeld** .</span><span class="sxs-lookup"><span data-stu-id="3b0ad-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="3b0ad-356">Volg de stappen in de volgende sectie of de pagina **uw prijs categorie kiezen** om [uw SSL-certificaat te uploaden en te binden](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="3b0ad-357">Uw App Service-plan omhoog schalen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="3b0ad-358">Selecteer de prijscategorie **Basic**, **Standard** of **Premium**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="3b0ad-359">Kies **Selecteren**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-359">Select **Select**.</span></span>

![Prijs categorie kiezen voor uw web-app](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="3b0ad-361">De schaal bewerking is voltooid wanneer de melding wordt weer gegeven.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-361">The scale operation is complete when notification is displayed.</span></span>

![Melding voor omhoog schalen](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="3b0ad-363">Uw SSL-certificaat binden en tussenliggende certificaten samen voegen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="3b0ad-364">Meerdere certificaten in de keten samen voegen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="3b0ad-365">**Open elk certificaat** dat u hebt ontvangen in een tekst editor.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="3b0ad-366">Maak een bestand voor het samengevoegde certificaat met de naam *mergedcertificate. CRT*.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="3b0ad-367">Kopieer de inhoud van elk certificaat in dit bestand in een teksteditor.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="3b0ad-368">De volgorde van uw certificaten moet de volgorde in de certificaatketen volgen, beginnend met uw certificaat en eindigend met het hoofdcertificaat.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="3b0ad-369">Het lijkt op het volgende voorbeeld:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="3b0ad-370">Certificaat naar PFX exporteren</span><span class="sxs-lookup"><span data-stu-id="3b0ad-370">Export certificate to PFX</span></span>

<span data-ttu-id="3b0ad-371">Het samengevoegde SSL-certificaat exporteren met de persoonlijke sleutel die door het certificaat is gegenereerd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="3b0ad-372">Er wordt een bestand met een persoonlijke sleutel gemaakt via OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="3b0ad-373">Als u het certificaat naar PFX wilt exporteren, voert u de volgende opdracht uit en vervangt u de tijdelijke aanduidingen `<private-key-file>` en het `<merged-certificate-file>` pad naar de persoonlijke sleutel en het samengevoegde certificaat bestand:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="3b0ad-374">Wanneer u hierom wordt gevraagd, definieert u een export wachtwoord voor het uploaden van uw SSL-certificaat naar App Service later.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="3b0ad-375">Wanneer IIS of **Certreq.exe** wordt gebruikt om de certificaat aanvraag te genereren, installeert u het certificaat op een lokale computer en [exporteert u het certificaat naar pfx](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="3b0ad-376">Het SSL-certificaat uploaden</span><span class="sxs-lookup"><span data-stu-id="3b0ad-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="3b0ad-377">Selecteer **SSL-instellingen** in het linkernavigatievenster van de web-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="3b0ad-378">Selecteer **certificaat uploaden**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="3b0ad-379">Selecteer in **PFX-certificaat bestand** de optie pfx-bestand.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="3b0ad-380">In **certificaat wachtwoord** typt u het wacht woord dat u hebt gemaakt bij het exporteren van het pfx-bestand.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="3b0ad-381">Selecteer **Uploaden**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-381">Select **Upload**.</span></span>

    ![SSL-certificaat uploaden](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="3b0ad-383">Als App Service klaar bent met het uploaden van het certificaat, wordt het weer gegeven op de pagina **SSL-instellingen** .</span><span class="sxs-lookup"><span data-stu-id="3b0ad-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![SSL-instellingen](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="3b0ad-385">Uw SSL-certificaat binden</span><span class="sxs-lookup"><span data-stu-id="3b0ad-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="3b0ad-386">Selecteer in de sectie **SSL-bindingen** de optie **binding toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="3b0ad-387">Als het certificaat is geüpload, maar niet wordt weer gegeven in domein namen in de vervolg keuzelijst **hostname** , probeert u de browser pagina te vernieuwen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="3b0ad-388">Op de pagina **SSL-binding toevoegen** gebruikt u de vervolg keuzelijst om de domein naam te selecteren die u wilt beveiligen en het certificaat dat u wilt gebruiken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="3b0ad-389">Selecteer in **SSL-type** of u [**Servernaamindicatie (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) wilt gebruiken of op IP gebaseerde SSL.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="3b0ad-390">**Op SNI gebaseerd SSL**: er kunnen meerdere SSL-bindingen op basis van SNI worden toegevoegd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="3b0ad-391">Met deze optie kunnen meerdere SSL-certificaten verschillende domeinen beveiligen op hetzelfde IP-adres.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="3b0ad-392">De meeste moderne browsers (waaronder Internet Explorer, Chrome, Firefox en Opera) ondersteunen SNI. Ga voor uitgebreidere informatie over browserondersteuning naar [Servernaamindicatie](https://wikipedia.org/wiki/Server_Name_Indication).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="3b0ad-393">**SSL op basis van IP**: er kan slechts één SSL-binding op basis van IP worden toegevoegd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="3b0ad-394">Met deze optie kan slechts één SSL-certificaat een specifiek openbaar IP-adres beveiligen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="3b0ad-395">Als u meerdere domeinen wilt beveiligen, moet u ze allemaal beveiligen met hetzelfde SSL-certificaat.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="3b0ad-396">SSL is de traditionele optie voor SSL-binding.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="3b0ad-397">Selecteer **binding toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-397">Select **Add Binding**.</span></span>

    ![SSL-binding toevoegen](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="3b0ad-399">Als App Service klaar bent met het uploaden van het certificaat, wordt het weer gegeven in de secties **SSL-bindingen** .</span><span class="sxs-lookup"><span data-stu-id="3b0ad-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Het uploaden van SSL-bindingen is voltooid](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="3b0ad-401">De A-record opnieuw toewijzen voor IP SSL</span><span class="sxs-lookup"><span data-stu-id="3b0ad-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="3b0ad-402">Als op IP gebaseerde SSL niet wordt gebruikt in de web-app, gaat u door naar [https testen voor uw aangepaste domein](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="3b0ad-403">De web-app maakt standaard gebruik van een gedeeld openbaar IP-adres.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="3b0ad-404">Wanneer het certificaat is gebonden aan SSL op basis van IP, App Service maakt een nieuw en toegewijd IP-adres voor de web-app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="3b0ad-405">Wanneer een A-record wordt toegewezen aan de web-app, moet het domein register worden bijgewerkt met het specifieke IP-adres.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="3b0ad-406">De pagina **aangepast domein** wordt bijgewerkt met het nieuwe, toegewezen IP-adres.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="3b0ad-407">Kopieer dit [IP-adres](/azure/app-service/app-service-web-tutorial-custom-domain)en wijs vervolgens de [A-record](/azure/app-service/app-service-web-tutorial-custom-domain) opnieuw toe aan dit nieuwe IP-adres.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="3b0ad-408">HTTPS testen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-408">Test HTTPS</span></span>

<span data-ttu-id="3b0ad-409">Ga in verschillende browsers naar `https://<your.custom.domain>` om ervoor te zorgen dat de web-app wordt aangeboden.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![naar web-app bladeren](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="3b0ad-411">Als er fouten in de certificaat validatie optreden, kan een zelfondertekend certificaat de oorzaak zijn of kunnen tussenliggende certificaten zijn uitgeschakeld bij het exporteren naar het PFX-bestand.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="3b0ad-412">HTTPS afdwingen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-412">Enforce HTTPS</span></span>

<span data-ttu-id="3b0ad-413">Standaard heeft iedereen toegang tot de web-app met behulp van HTTP.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="3b0ad-414">Alle HTTP-aanvragen voor de HTTPS-poort kunnen worden omgeleid.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="3b0ad-415">Selecteer op de pagina Web-App de optie **SL-instellingen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="3b0ad-416">Klik op **Alleen HTTPS** en selecteer **Aan**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-416">Then, in **HTTPS Only**, select **On**.</span></span>

![HTTPS afdwingen](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="3b0ad-418">Wanneer de bewerking is voltooid, gaat u naar een van de HTTP-Url's die verwijzen naar de app.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="3b0ad-419">Bijvoorbeeld:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-419">For example:</span></span>

- <span data-ttu-id="3b0ad-420"> https://<app_name>. azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="3b0ad-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="3b0ad-421">TLS 1.1/1.2 afdwingen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="3b0ad-422">De app staat standaard [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0 toe, die niet meer wordt beschouwd als veilig door industriële normen (zoals [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span><span class="sxs-lookup"><span data-stu-id="3b0ad-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="3b0ad-423">Als u hogere TLS-versies wilt afdwingen, volgt u deze stappen:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="3b0ad-424">Selecteer op de pagina Web-app in het linkernavigatievenster de optie **SSL-instellingen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="3b0ad-425">Selecteer in **TLS-versie** de minimale TLS-versie.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![TLS 1.1 of 1.2 afdwingen](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="3b0ad-427">Een Traffic Manager-profiel maken</span><span class="sxs-lookup"><span data-stu-id="3b0ad-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="3b0ad-428">Selecteer **een resource maken**  >  **netwerk**  >  **Traffic Manager profiel**  >  **maken**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="3b0ad-429">Vul het volgende in bij **Traffic Manager-profiel maken**:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="3b0ad-430">Geef bij **naam** een naam op voor het profiel.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="3b0ad-431">Deze naam moet uniek zijn binnen de zone verkeer manager.net en resulteert in de DNS-naam trafficmanager.net, die wordt gebruikt voor toegang tot het profiel van de Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="3b0ad-432">Selecteer bij **routerings methode** de **geografische routerings methode**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="3b0ad-433">Selecteer bij **abonnement** het abonnement waaronder u dit profiel wilt maken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="3b0ad-434">In **Resourcegroep** maakt u een nieuwe resourcegroep om dit profiel voor te maken.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="3b0ad-435">In **Locatie van de resourcegroep** selecteert u de locatie van de resourcegroep.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="3b0ad-436">Deze instelling verwijst naar de locatie van de resource groep en heeft geen invloed op het Traffic Manager profiel wereld wijd geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="3b0ad-437">Selecteer **Maken**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-437">Select **Create**.</span></span>

    7. <span data-ttu-id="3b0ad-438">Wanneer de globale implementatie van het Traffic Manager profiel is voltooid, wordt het weer gegeven in de betreffende resource groep als een van de resources.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Resource groepen in Traffic Manager profiel maken](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="3b0ad-440">Traffic Manager-eindpunten toevoegen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="3b0ad-441">Zoek in de zoek balk van de portal naar de naam van het **Traffic Manager profiel** dat u in de voor gaande sectie hebt gemaakt en selecteer het Traffic Manager-profiel in de weer gegeven resultaten.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="3b0ad-442">Selecteer in **Traffic Manager profiel** in de sectie **instellingen** de optie **eind punten**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="3b0ad-443">Selecteer **Toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-443">Select **Add**.</span></span>

4. <span data-ttu-id="3b0ad-444">Het Azure Stack hub-eind punt toevoegen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="3b0ad-445">Selecteer bij **type** **externe eind punt**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="3b0ad-446">Geef een **naam** op voor dit eind punt, in het ideale geval de naam van de Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="3b0ad-447">Gebruik voor Fully Qualified Domain Name (**FQDN**) de externe URL voor de web-app Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="3b0ad-448">Selecteer onder geo-toewijzing een regio/continent waar de resource zich bevindt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="3b0ad-449">Bijvoorbeeld **Europa.**</span><span class="sxs-lookup"><span data-stu-id="3b0ad-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="3b0ad-450">Selecteer in de vervolg keuzelijst land/regio die wordt weer gegeven het land dat van toepassing is op dit eind punt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="3b0ad-451">Bijvoorbeeld **Duitsland**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="3b0ad-452">Laat **Toevoegen als uitgeschakeld** uit staan.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="3b0ad-453">Selecteer **OK**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-453">Select **OK**.</span></span>

12. <span data-ttu-id="3b0ad-454">De Azure-eindpunt toevoegen:</span><span class="sxs-lookup"><span data-stu-id="3b0ad-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="3b0ad-455">Selecteer voor **type** **Azure-eind punt**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="3b0ad-456">Geef een **naam** op voor het eind punt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="3b0ad-457">Selecteer **app service** bij **doel resource type**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="3b0ad-458">Selecteer bij **doel resource** **een app service kiezen** om de vermelding van de web apps onder hetzelfde abonnement weer te geven.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="3b0ad-459">Kies in **resource** de app service die als eerste eind punt wordt gebruikt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="3b0ad-460">Selecteer onder geo-toewijzing een regio/continent waar de resource zich bevindt.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="3b0ad-461">Bijvoorbeeld **Noord-Amerika/Centraal-Amerika/Caribisch gebied.**</span><span class="sxs-lookup"><span data-stu-id="3b0ad-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="3b0ad-462">In de vervolg keuzelijst land/regio die wordt weer gegeven, laat u deze plaats leeg om alle bovenstaande regionale groepering te selecteren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="3b0ad-463">Laat **Toevoegen als uitgeschakeld** uit staan.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="3b0ad-464">Selecteer **OK**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="3b0ad-465">Maak ten minste één eind punt met een geografisch bereik van alle (wereld) om als het standaard eindpunt voor de resource te fungeren.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="3b0ad-466">Wanneer beide eind punten zijn toegevoegd, worden ze weer gegeven in **Traffic Manager profiel** samen met hun controle status als **online**.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Eind punt status van Traffic Manager profiel](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="3b0ad-468">De wereld wijde onderneming is afhankelijk van de mogelijkheden van de geo-distributie van Azure</span><span class="sxs-lookup"><span data-stu-id="3b0ad-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="3b0ad-469">Door het omleiden van gegevens verkeer via Azure Traffic Manager en geografie-specifieke eind punten kunnen wereld wijde ondernemingen zich houden aan regionale voor schriften en de gegevens conform en veilig houden, wat van belang is voor het succes van lokale en externe bedrijfs locaties.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3b0ad-470">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="3b0ad-470">Next steps</span></span>

- <span data-ttu-id="3b0ad-471">Zie [Cloud ontwerp patronen](/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.</span><span class="sxs-lookup"><span data-stu-id="3b0ad-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
