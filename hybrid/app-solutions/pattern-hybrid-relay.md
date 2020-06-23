---
title: Hybride relay-patroon in Azure en Azure Stack hub
description: Gebruik het hybride relay-patroon in Azure en Azure Stack hub om verbinding te maken met Edge-bronnen die worden beveiligd door firewalls.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910433"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="704e9-103">Hybride relay-patroon</span><span class="sxs-lookup"><span data-stu-id="704e9-103">Hybrid relay pattern</span></span>

<span data-ttu-id="704e9-104">Meer informatie over hoe u verbinding maakt met Edge-bronnen of apparaten die door firewalls worden beveiligd met het hybride relay-patroon en Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="704e9-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="704e9-105">Context en probleem</span><span class="sxs-lookup"><span data-stu-id="704e9-105">Context and problem</span></span>

<span data-ttu-id="704e9-106">Edge-apparaten bevinden zich vaak achter een bedrijfs firewall of een NAT-apparaat.</span><span class="sxs-lookup"><span data-stu-id="704e9-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="704e9-107">Hoewel ze veilig zijn, kunnen ze mogelijk niet communiceren met de open bare Cloud of edge-apparaten op andere bedrijfs netwerken.</span><span class="sxs-lookup"><span data-stu-id="704e9-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="704e9-108">Het kan nodig zijn om op een veilige manier bepaalde poorten en functionaliteit beschikbaar te stellen aan gebruikers in de open bare Cloud.</span><span class="sxs-lookup"><span data-stu-id="704e9-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="704e9-109">Oplossing</span><span class="sxs-lookup"><span data-stu-id="704e9-109">Solution</span></span>

<span data-ttu-id="704e9-110">Het hybride relay-patroon gebruikt Azure Relay om een websockets-tunnel te maken tussen twee eind punten die niet rechtstreeks kunnen communiceren.</span><span class="sxs-lookup"><span data-stu-id="704e9-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="704e9-111">Apparaten die niet on-premises zijn maar verbinding moeten maken met een on-premises eind punt, maken verbinding met een eind punt in de open bare Cloud.</span><span class="sxs-lookup"><span data-stu-id="704e9-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="704e9-112">Met dit eind punt wordt het verkeer omgeleid naar vooraf gedefinieerde routes via een beveiligd kanaal.</span><span class="sxs-lookup"><span data-stu-id="704e9-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="704e9-113">Een eind punt in de on-premises omgeving ontvangt het verkeer en stuurt het naar de juiste bestemming.</span><span class="sxs-lookup"><span data-stu-id="704e9-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![architectuur van hybride relay-patroon oplossing](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="704e9-115">Hoe het hybride relay-patroon werkt:</span><span class="sxs-lookup"><span data-stu-id="704e9-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="704e9-116">Een apparaat maakt verbinding met de virtuele machine (VM) in azure, op een vooraf gedefinieerde poort.</span><span class="sxs-lookup"><span data-stu-id="704e9-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="704e9-117">Verkeer wordt doorgestuurd naar de Azure Relay in Azure.</span><span class="sxs-lookup"><span data-stu-id="704e9-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="704e9-118">De VM op Azure Stack hub, die al een langdurige verbinding met de Azure Relay heeft gemaakt, ontvangt het verkeer en stuurt het door naar de bestemming.</span><span class="sxs-lookup"><span data-stu-id="704e9-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="704e9-119">De on-premises service of het eind punt verwerkt de aanvraag.</span><span class="sxs-lookup"><span data-stu-id="704e9-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="704e9-120">Onderdelen</span><span class="sxs-lookup"><span data-stu-id="704e9-120">Components</span></span>

<span data-ttu-id="704e9-121">Deze oplossing maakt gebruik van de volgende onderdelen:</span><span class="sxs-lookup"><span data-stu-id="704e9-121">This solution uses the following components:</span></span>

| <span data-ttu-id="704e9-122">Laag</span><span class="sxs-lookup"><span data-stu-id="704e9-122">Layer</span></span> | <span data-ttu-id="704e9-123">Onderdeel</span><span class="sxs-lookup"><span data-stu-id="704e9-123">Component</span></span> | <span data-ttu-id="704e9-124">Beschrijving</span><span class="sxs-lookup"><span data-stu-id="704e9-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="704e9-125">Azure</span><span class="sxs-lookup"><span data-stu-id="704e9-125">Azure</span></span> | <span data-ttu-id="704e9-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="704e9-126">Azure VM</span></span> | <span data-ttu-id="704e9-127">Een Azure-VM biedt een openbaar toegankelijk eind punt voor de on-premises resource.</span><span class="sxs-lookup"><span data-stu-id="704e9-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="704e9-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="704e9-128">Azure Relay</span></span> | <span data-ttu-id="704e9-129">Een [Azure relay](/azure/azure-relay/) biedt de infra structuur voor het onderhouden van de tunnel en de verbinding tussen de virtuele machine van Azure en Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="704e9-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="704e9-130">Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="704e9-130">Azure Stack Hub</span></span> | <span data-ttu-id="704e9-131">Compute</span><span class="sxs-lookup"><span data-stu-id="704e9-131">Compute</span></span> | <span data-ttu-id="704e9-132">Een Azure Stack hub-VM levert de server zijde van de hybride relay-tunnel.</span><span class="sxs-lookup"><span data-stu-id="704e9-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="704e9-133">Storage</span><span class="sxs-lookup"><span data-stu-id="704e9-133">Storage</span></span> | <span data-ttu-id="704e9-134">Het AKS-engine cluster dat is geïmplementeerd in Azure Stack hub biedt een schaal bare, robuuste engine voor het uitvoeren van de Face-API-container.</span><span class="sxs-lookup"><span data-stu-id="704e9-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="704e9-135">Problemen en overwegingen</span><span class="sxs-lookup"><span data-stu-id="704e9-135">Issues and considerations</span></span>

<span data-ttu-id="704e9-136">Houd rekening met de volgende punten wanneer u bepaalt hoe u deze oplossing implementeert:</span><span class="sxs-lookup"><span data-stu-id="704e9-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="704e9-137">Schaalbaarheid</span><span class="sxs-lookup"><span data-stu-id="704e9-137">Scalability</span></span>

<span data-ttu-id="704e9-138">Met dit patroon kunnen alleen 1:1 poort toewijzingen op de client en de server worden toegewezen.</span><span class="sxs-lookup"><span data-stu-id="704e9-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="704e9-139">Als poort 80 bijvoorbeeld is getunneld voor één service op het Azure-eind punt, kan deze niet worden gebruikt voor een andere service.</span><span class="sxs-lookup"><span data-stu-id="704e9-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="704e9-140">Poort toewijzingen moeten dienovereenkomstig worden gepland.</span><span class="sxs-lookup"><span data-stu-id="704e9-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="704e9-141">De Azure Relay en Vm's moeten op de juiste manier worden geschaald om verkeer te verwerken.</span><span class="sxs-lookup"><span data-stu-id="704e9-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="704e9-142">Beschikbaarheid</span><span class="sxs-lookup"><span data-stu-id="704e9-142">Availability</span></span>

<span data-ttu-id="704e9-143">Deze tunnels en verbindingen zijn niet overbodig.</span><span class="sxs-lookup"><span data-stu-id="704e9-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="704e9-144">Om hoge Beschik baarheid te garanderen, wilt u mogelijk fout controle code implementeren.</span><span class="sxs-lookup"><span data-stu-id="704e9-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="704e9-145">Een andere optie is om een groep met Azure Relay verbonden Vm's achter een load balancer te hebben.</span><span class="sxs-lookup"><span data-stu-id="704e9-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="704e9-146">Beheerbaarheid</span><span class="sxs-lookup"><span data-stu-id="704e9-146">Manageability</span></span>

<span data-ttu-id="704e9-147">Deze oplossing kan veel apparaten en locaties omvatten, wat een onhandigheid kan oplevert.</span><span class="sxs-lookup"><span data-stu-id="704e9-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="704e9-148">Met de IoT-services van Azure kunnen nieuwe locaties en apparaten automatisch online worden gebracht en up-to-date blijven.</span><span class="sxs-lookup"><span data-stu-id="704e9-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="704e9-149">Beveiliging</span><span class="sxs-lookup"><span data-stu-id="704e9-149">Security</span></span>

<span data-ttu-id="704e9-150">Met dit patroon zoals weer gegeven kunt u onbelemmerde toegang geven tot een poort op een intern apparaat vanaf de rand.</span><span class="sxs-lookup"><span data-stu-id="704e9-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="704e9-151">Overweeg een verificatie mechanisme toe te voegen aan de service op het interne apparaat of vóór het hybride relay-eind punt.</span><span class="sxs-lookup"><span data-stu-id="704e9-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="704e9-152">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="704e9-152">Next steps</span></span>

<span data-ttu-id="704e9-153">Voor meer informatie over de onderwerpen die in dit artikel worden geïntroduceerd:</span><span class="sxs-lookup"><span data-stu-id="704e9-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="704e9-154">Dit patroon maakt gebruik van Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="704e9-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="704e9-155">Zie de [Azure relay-documentatie](/azure/azure-relay/)voor meer informatie.</span><span class="sxs-lookup"><span data-stu-id="704e9-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="704e9-156">Zie [overwegingen voor het ontwerpen van hybride toepassingen](overview-app-design-considerations.md) voor meer informatie over best practices en krijg antwoorden op eventuele aanvullende vragen.</span><span class="sxs-lookup"><span data-stu-id="704e9-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="704e9-157">Bekijk de [Azure stack-familie van producten en oplossingen](/azure-stack) voor meer informatie over de volledige Port Folio van producten en oplossingen.</span><span class="sxs-lookup"><span data-stu-id="704e9-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="704e9-158">Wanneer u klaar bent om het voor beeld van de oplossing te testen, gaat u verder met de [implementatie handleiding voor hybride relay-oplossingen](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="704e9-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="704e9-159">De implementatie handleiding bevat stapsgewijze instructies voor het implementeren en testen van de onderdelen.</span><span class="sxs-lookup"><span data-stu-id="704e9-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>