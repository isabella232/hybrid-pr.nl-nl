---
title: Geografisch gedistribueerd app-patroon in Azure Stack hub
description: Meer informatie over het patroon voor geografisch gedistribueerde apps voor de intelligente rand met behulp van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910319"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="aab1e-103">Patroon van geografisch gedistribueerde app</span><span class="sxs-lookup"><span data-stu-id="aab1e-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="aab1e-104">Meer informatie over het bieden van app-eind punten voor meerdere regio's en het routeren van gebruikers verkeer op basis van locatie-en nalevings behoeften.</span><span class="sxs-lookup"><span data-stu-id="aab1e-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="aab1e-105">Context en probleem</span><span class="sxs-lookup"><span data-stu-id="aab1e-105">Context and problem</span></span>

<span data-ttu-id="aab1e-106">Organisaties met een brede geografi streven ernaar om veilig en nauw keurig toegang te krijgen tot gegevens, terwijl er behoefte is aan beveiliging, naleving en prestaties per gebruiker, locatie en apparaat tussen de randen.</span><span class="sxs-lookup"><span data-stu-id="aab1e-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="aab1e-107">Oplossing</span><span class="sxs-lookup"><span data-stu-id="aab1e-107">Solution</span></span>

<span data-ttu-id="aab1e-108">Met het Azure Stack hub-verkeer voor geografische verkeers routering of geografisch gedistribueerde apps kan verkeer worden omgeleid naar specifieke eind punten op basis van verschillende metrische gegevens.</span><span class="sxs-lookup"><span data-stu-id="aab1e-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="aab1e-109">Het maken van een Traffic Manager met op geografische basis gebaseerde route ring en eindpunt configuratie stuurt verkeer naar eind punten op basis van regionale vereisten, zakelijke en internationale regelgeving en gegevens behoeften.</span><span class="sxs-lookup"><span data-stu-id="aab1e-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Geografisch gedistribueerd patroon](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="aab1e-111">Onderdelen</span><span class="sxs-lookup"><span data-stu-id="aab1e-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="aab1e-112">Buiten de Cloud</span><span class="sxs-lookup"><span data-stu-id="aab1e-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="aab1e-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="aab1e-113">Traffic Manager</span></span>

<span data-ttu-id="aab1e-114">In het diagram bevindt Traffic Manager zich buiten de open bare Cloud, maar moet het verkeer in het lokale Data Center en de open bare Cloud kunnen worden gecoördineerd.</span><span class="sxs-lookup"><span data-stu-id="aab1e-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="aab1e-115">De Balancer routeert verkeer naar geografische locaties.</span><span class="sxs-lookup"><span data-stu-id="aab1e-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="aab1e-116">Domain Name System (DNS)</span><span class="sxs-lookup"><span data-stu-id="aab1e-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="aab1e-117">De Domain Name System, of DNS, is verantwoordelijk voor het vertalen van een website-of service naam naar het IP-adres.</span><span class="sxs-lookup"><span data-stu-id="aab1e-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="aab1e-118">Openbare cloud</span><span class="sxs-lookup"><span data-stu-id="aab1e-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="aab1e-119">Cloud-eind punt</span><span class="sxs-lookup"><span data-stu-id="aab1e-119">Cloud Endpoint</span></span>

<span data-ttu-id="aab1e-120">Open bare IP-adressen worden gebruikt voor het routeren van binnenkomend verkeer via Traffic Manager naar het eind punt van de open bare Cloud app-resources.</span><span class="sxs-lookup"><span data-stu-id="aab1e-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="aab1e-121">Lokale Clouds</span><span class="sxs-lookup"><span data-stu-id="aab1e-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="aab1e-122">Lokaal eind punt</span><span class="sxs-lookup"><span data-stu-id="aab1e-122">Local endpoint</span></span>

<span data-ttu-id="aab1e-123">Open bare IP-adressen worden gebruikt voor het routeren van binnenkomend verkeer via Traffic Manager naar het eind punt van de open bare Cloud app-resources.</span><span class="sxs-lookup"><span data-stu-id="aab1e-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="aab1e-124">Problemen en overwegingen</span><span class="sxs-lookup"><span data-stu-id="aab1e-124">Issues and considerations</span></span>

<span data-ttu-id="aab1e-125">Beschouw de volgende punten als u besluit hoe u dit patroon wilt implementeren:</span><span class="sxs-lookup"><span data-stu-id="aab1e-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="aab1e-126">Schaalbaarheid</span><span class="sxs-lookup"><span data-stu-id="aab1e-126">Scalability</span></span>

<span data-ttu-id="aab1e-127">Het patroon handelt geografische verkeers routering af in plaats van schalen om te voldoen aan toename van verkeer.</span><span class="sxs-lookup"><span data-stu-id="aab1e-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="aab1e-128">U kunt dit patroon echter combi neren met andere Azure-en on-premises oplossingen.</span><span class="sxs-lookup"><span data-stu-id="aab1e-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="aab1e-129">Dit patroon kan bijvoorbeeld worden gebruikt in combi natie met het patroon voor meerdere Cloud schalen.</span><span class="sxs-lookup"><span data-stu-id="aab1e-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="aab1e-130">Beschikbaarheid</span><span class="sxs-lookup"><span data-stu-id="aab1e-130">Availability</span></span>

<span data-ttu-id="aab1e-131">Zorg ervoor dat lokaal geïmplementeerde apps zijn geconfigureerd voor hoge Beschik baarheid via on-premises hardwareconfiguratie en software-implementatie.</span><span class="sxs-lookup"><span data-stu-id="aab1e-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="aab1e-132">Beheerbaarheid</span><span class="sxs-lookup"><span data-stu-id="aab1e-132">Manageability</span></span>

<span data-ttu-id="aab1e-133">Het patroon zorgt voor naadloos beheer en de vertrouwde interface tussen omgevingen.</span><span class="sxs-lookup"><span data-stu-id="aab1e-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="aab1e-134">Wanneer dit patroon gebruiken</span><span class="sxs-lookup"><span data-stu-id="aab1e-134">When to use this pattern</span></span>

- <span data-ttu-id="aab1e-135">Mijn organisatie heeft internationale vertakkingen waarvoor aangepaste regionale beveiligings-en distributie beleidsregels zijn vereist.</span><span class="sxs-lookup"><span data-stu-id="aab1e-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="aab1e-136">Elke kant oren van mijn organisatie halen werk nemer-, bedrijfs-en faciliteit gegevens op, waarvoor rapportage activiteiten per lokale regelgeving en tijd zone nodig zijn.</span><span class="sxs-lookup"><span data-stu-id="aab1e-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="aab1e-137">Aan hoge schaal vereisten kan worden voldaan door apps horizon taal te schalen, waarbij meerdere app-implementaties worden uitgevoerd binnen één regio en in verschillende regio's om extreem belasting vereisten af te handelen.</span><span class="sxs-lookup"><span data-stu-id="aab1e-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="aab1e-138">De apps moeten Maxi maal beschikbaar zijn en reageren op aanvragen van clients, zelfs in een storing in één regio.</span><span class="sxs-lookup"><span data-stu-id="aab1e-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="aab1e-139">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="aab1e-139">Next steps</span></span>

<span data-ttu-id="aab1e-140">Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:</span><span class="sxs-lookup"><span data-stu-id="aab1e-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="aab1e-141">Zie het [overzicht van Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) voor meer informatie over hoe dit op DNS gebaseerd verkeer Load Balancer werkt.</span><span class="sxs-lookup"><span data-stu-id="aab1e-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="aab1e-142">Bekijk de overwegingen voor het [ontwerpen van hybride apps](overview-app-design-considerations.md) voor meer informatie over aanbevolen procedures en voor het verkrijgen van antwoorden voor aanvullende vragen.</span><span class="sxs-lookup"><span data-stu-id="aab1e-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="aab1e-143">Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.</span><span class="sxs-lookup"><span data-stu-id="aab1e-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="aab1e-144">Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [implementatie handleiding voor geografisch gedistribueerde app-oplossingen](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="aab1e-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="aab1e-145">De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.</span><span class="sxs-lookup"><span data-stu-id="aab1e-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="aab1e-146">U leert hoe u verkeer naar specifieke eind punten kunt omleiden, op basis van verschillende metrische gegevens met behulp van het geo-gedistribueerde app-patroon.</span><span class="sxs-lookup"><span data-stu-id="aab1e-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="aab1e-147">Het maken van een Traffic Manager profiel met op geografische wijze gebaseerde route ring en eindpunt configuratie zorgt ervoor dat informatie wordt doorgestuurd naar eind punten op basis van regionale vereisten, zakelijke en internationale regelgeving en uw gegevens behoeften.</span><span class="sxs-lookup"><span data-stu-id="aab1e-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
