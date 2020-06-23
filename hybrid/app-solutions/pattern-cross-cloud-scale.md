---
title: Patroon voor meerdere Cloud schalen in Azure Stack hub
description: Meer informatie over het bouwen van een schaal bare, cross-Cloud-app in Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910427"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="c5215-103">Patroon voor meerdere Cloud schalen</span><span class="sxs-lookup"><span data-stu-id="c5215-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="c5215-104">Voeg automatisch resources toe aan een bestaande app om een grotere belasting te kunnen bieden.</span><span class="sxs-lookup"><span data-stu-id="c5215-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c5215-105">Context en probleem</span><span class="sxs-lookup"><span data-stu-id="c5215-105">Context and problem</span></span>

<span data-ttu-id="c5215-106">Uw app kan geen capaciteit verg Roten om te voldoen aan onverwachte toename van de vraag.</span><span class="sxs-lookup"><span data-stu-id="c5215-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="c5215-107">Dit heeft geen gevolgen voor de schaal baarheid van gebruikers die de app niet bereiken tijdens piek gebruik.</span><span class="sxs-lookup"><span data-stu-id="c5215-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="c5215-108">De app kan een vast aantal gebruikers onderhouden.</span><span class="sxs-lookup"><span data-stu-id="c5215-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="c5215-109">Wereld wijde ondernemingen vereisen veilige, betrouw bare en beschik bare Cloud-apps.</span><span class="sxs-lookup"><span data-stu-id="c5215-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="c5215-110">Verg root de vraag en maakt gebruik van de juiste infra structuur ter ondersteuning van de vraag.</span><span class="sxs-lookup"><span data-stu-id="c5215-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="c5215-111">Bedrijven kunnen moeite besparen met kosten en onderhoud met de beveiliging van Bedrijfs gegevens, opslag en real-time Beschik baarheid.</span><span class="sxs-lookup"><span data-stu-id="c5215-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="c5215-112">Mogelijk kunt u uw app niet uitvoeren in de open bare Cloud.</span><span class="sxs-lookup"><span data-stu-id="c5215-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="c5215-113">Het is echter mogelijk niet economisch haalbaar voor het bedrijf om de capaciteit te behouden die vereist is in hun on-premises omgeving voor het afhandelen van pieken in de vraag naar de app.</span><span class="sxs-lookup"><span data-stu-id="c5215-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="c5215-114">Met dit patroon kunt u gebruikmaken van de elasticiteit van de open bare Cloud met uw on-premises oplossing.</span><span class="sxs-lookup"><span data-stu-id="c5215-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="c5215-115">Oplossing</span><span class="sxs-lookup"><span data-stu-id="c5215-115">Solution</span></span>

<span data-ttu-id="c5215-116">Met het patroon voor meerdere Cloud schalen wordt een app uitgebreid die zich in een lokale Cloud bevindt met open bare cloud resources.</span><span class="sxs-lookup"><span data-stu-id="c5215-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="c5215-117">Het patroon wordt geactiveerd door een verhoging of afname in de vraag en voegt respectievelijk resources toe aan of verwijderd uit de Cloud.</span><span class="sxs-lookup"><span data-stu-id="c5215-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="c5215-118">Deze bronnen bieden redundantie, snelle Beschik baarheid en geo-compatibele route ring.</span><span class="sxs-lookup"><span data-stu-id="c5215-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Patroon voor meerdere Cloud schalen](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="c5215-120">Dit patroon is alleen van toepassing op stateless onderdelen van uw app.</span><span class="sxs-lookup"><span data-stu-id="c5215-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="c5215-121">Onderdelen</span><span class="sxs-lookup"><span data-stu-id="c5215-121">Components</span></span>

<span data-ttu-id="c5215-122">Het patroon voor meerdere Cloud schalen bestaat uit de volgende onderdelen.</span><span class="sxs-lookup"><span data-stu-id="c5215-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="c5215-123">Buiten de Cloud</span><span class="sxs-lookup"><span data-stu-id="c5215-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="c5215-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="c5215-124">Traffic Manager</span></span>

<span data-ttu-id="c5215-125">Dit bevindt zich buiten de open bare Cloud groep in het diagram, maar zou wel het verkeer in het lokale Data Center en de open bare Cloud moeten kunnen coördineren.</span><span class="sxs-lookup"><span data-stu-id="c5215-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="c5215-126">De Balancer levert een hoge Beschik baarheid voor de app door eind punten te bewaken en zo nodig failover te distribueren.</span><span class="sxs-lookup"><span data-stu-id="c5215-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="c5215-127">Domain Name System (DNS)</span><span class="sxs-lookup"><span data-stu-id="c5215-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="c5215-128">De Domain Name System, of DNS, is verantwoordelijk voor het vertalen van een website-of service naam naar het IP-adres.</span><span class="sxs-lookup"><span data-stu-id="c5215-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="c5215-129">Cloud</span><span class="sxs-lookup"><span data-stu-id="c5215-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="c5215-130">Gehoste build server</span><span class="sxs-lookup"><span data-stu-id="c5215-130">Hosted build server</span></span>

<span data-ttu-id="c5215-131">Een omgeving voor het hosten van uw build-pijp lijn.</span><span class="sxs-lookup"><span data-stu-id="c5215-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="c5215-132">App-resources</span><span class="sxs-lookup"><span data-stu-id="c5215-132">App resources</span></span>

<span data-ttu-id="c5215-133">De app-resources moeten kunnen worden geschaald en uitgeschaald, zoals virtuele-machine schaal sets en containers.</span><span class="sxs-lookup"><span data-stu-id="c5215-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="c5215-134">Aangepaste domein naam</span><span class="sxs-lookup"><span data-stu-id="c5215-134">Custom domain name</span></span>

<span data-ttu-id="c5215-135">Gebruik een aangepaste domein naam voor routerings aanvragen Globs.</span><span class="sxs-lookup"><span data-stu-id="c5215-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="c5215-136">Openbare IP-adressen</span><span class="sxs-lookup"><span data-stu-id="c5215-136">Public IP addresses</span></span>

<span data-ttu-id="c5215-137">Open bare IP-adressen worden gebruikt voor het routeren van binnenkomend verkeer via Traffic Manager naar het eind punt van de open bare Cloud app-resources.</span><span class="sxs-lookup"><span data-stu-id="c5215-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="c5215-138">Lokale Cloud</span><span class="sxs-lookup"><span data-stu-id="c5215-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="c5215-139">Gehoste build server</span><span class="sxs-lookup"><span data-stu-id="c5215-139">Hosted build server</span></span>

<span data-ttu-id="c5215-140">Een omgeving voor het hosten van uw build-pijp lijn.</span><span class="sxs-lookup"><span data-stu-id="c5215-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="c5215-141">App-resources</span><span class="sxs-lookup"><span data-stu-id="c5215-141">App resources</span></span>

<span data-ttu-id="c5215-142">De app-resources moeten kunnen worden geschaald en uitgeschaald, zoals virtuele-machine schaal sets en containers.</span><span class="sxs-lookup"><span data-stu-id="c5215-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="c5215-143">Aangepaste domein naam</span><span class="sxs-lookup"><span data-stu-id="c5215-143">Custom domain name</span></span>

<span data-ttu-id="c5215-144">Gebruik een aangepaste domein naam voor routerings aanvragen Globs.</span><span class="sxs-lookup"><span data-stu-id="c5215-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="c5215-145">Openbare IP-adressen</span><span class="sxs-lookup"><span data-stu-id="c5215-145">Public IP addresses</span></span>

<span data-ttu-id="c5215-146">Open bare IP-adressen worden gebruikt voor het routeren van binnenkomend verkeer via Traffic Manager naar het eind punt van de open bare Cloud app-resources.</span><span class="sxs-lookup"><span data-stu-id="c5215-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c5215-147">Problemen en overwegingen</span><span class="sxs-lookup"><span data-stu-id="c5215-147">Issues and considerations</span></span>

<span data-ttu-id="c5215-148">Beschouw de volgende punten als u besluit hoe u dit patroon wilt implementeren:</span><span class="sxs-lookup"><span data-stu-id="c5215-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="c5215-149">Schaalbaarheid</span><span class="sxs-lookup"><span data-stu-id="c5215-149">Scalability</span></span>

<span data-ttu-id="c5215-150">Het belangrijkste onderdeel van cross-Cloud schalen is de mogelijkheid om op aanvraag schalen te leveren.</span><span class="sxs-lookup"><span data-stu-id="c5215-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="c5215-151">Er moet worden geschaald tussen de open bare en lokale Cloud infrastructuur en een consistente, betrouw bare service per vraag bieden.</span><span class="sxs-lookup"><span data-stu-id="c5215-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="c5215-152">Beschikbaarheid</span><span class="sxs-lookup"><span data-stu-id="c5215-152">Availability</span></span>

<span data-ttu-id="c5215-153">Zorg ervoor dat lokaal geïmplementeerde apps zijn geconfigureerd voor hoge Beschik baarheid via on-premises hardwareconfiguratie en software-implementatie.</span><span class="sxs-lookup"><span data-stu-id="c5215-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="c5215-154">Beheerbaarheid</span><span class="sxs-lookup"><span data-stu-id="c5215-154">Manageability</span></span>

<span data-ttu-id="c5215-155">Het cross-Cloud-patroon zorgt voor naadloos beheer en de vertrouwde interface tussen omgevingen.</span><span class="sxs-lookup"><span data-stu-id="c5215-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c5215-156">Wanneer dit patroon gebruiken</span><span class="sxs-lookup"><span data-stu-id="c5215-156">When to use this pattern</span></span>

<span data-ttu-id="c5215-157">U gebruikt dit patroon voor het volgende:</span><span class="sxs-lookup"><span data-stu-id="c5215-157">Use this pattern:</span></span>

- <span data-ttu-id="c5215-158">Wanneer u uw app-capaciteit moet verhogen met onverwachte vraag of periodieke vraag naar vraag.</span><span class="sxs-lookup"><span data-stu-id="c5215-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="c5215-159">Wanneer u niet wilt investeren in resources die alleen tijdens pieken worden gebruikt.</span><span class="sxs-lookup"><span data-stu-id="c5215-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="c5215-160">Betaal voor wat u gebruikt.</span><span class="sxs-lookup"><span data-stu-id="c5215-160">Pay for what you use.</span></span>

<span data-ttu-id="c5215-161">Dit patroon wordt niet aanbevolen wanneer:</span><span class="sxs-lookup"><span data-stu-id="c5215-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="c5215-162">Voor uw oplossing moeten gebruikers verbinding maken via internet.</span><span class="sxs-lookup"><span data-stu-id="c5215-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="c5215-163">Uw bedrijf heeft lokale voor Schriften die vereisen dat de oorspronkelijke verbinding afkomstig is van een on-site oproep.</span><span class="sxs-lookup"><span data-stu-id="c5215-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="c5215-164">Uw netwerk heeft regel matig knel punten die de prestaties van de schaal baarheid zouden beperken.</span><span class="sxs-lookup"><span data-stu-id="c5215-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="c5215-165">Uw omgeving is niet verbonden met internet en de open bare Cloud kan niet worden bereikt.</span><span class="sxs-lookup"><span data-stu-id="c5215-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c5215-166">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="c5215-166">Next steps</span></span>

<span data-ttu-id="c5215-167">Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:</span><span class="sxs-lookup"><span data-stu-id="c5215-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="c5215-168">Zie het [overzicht van Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) voor meer informatie over hoe dit op DNS gebaseerd verkeer Load Balancer werkt.</span><span class="sxs-lookup"><span data-stu-id="c5215-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="c5215-169">Zie [overwegingen voor het ontwerpen van hybride toepassingen](overview-app-design-considerations.md) voor meer informatie over aanbevolen procedures en voor het verkrijgen van antwoorden voor eventuele aanvullende vragen.</span><span class="sxs-lookup"><span data-stu-id="c5215-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="c5215-170">Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.</span><span class="sxs-lookup"><span data-stu-id="c5215-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="c5215-171">Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de implementatie handleiding voor het implementeren van de [Cross-Cloud-schaal oplossing](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="c5215-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="c5215-172">De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.</span><span class="sxs-lookup"><span data-stu-id="c5215-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="c5215-173">U leert hoe u een cross-Cloud oplossing kunt maken om een hand matig geactiveerd proces te bieden voor het overschakelen van een door Azure Stack hub gehoste web-app naar een door Azure gehoste web-app.</span><span class="sxs-lookup"><span data-stu-id="c5215-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="c5215-174">U leert ook hoe u automatisch schalen kunt gebruiken via Traffic Manager, waardoor flexibele en schaal bare Cloud hulpprogramma's worden gebruikt wanneer de belasting wordt geladen.</span><span class="sxs-lookup"><span data-stu-id="c5215-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
