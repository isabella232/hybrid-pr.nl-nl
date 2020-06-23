---
title: Een SQL Server 2016-beschikbaarheids groep implementeren op Azure en Azure Stack hub
description: Meer informatie over het implementeren van een SQL Server 2016-beschikbaarheids groep naar Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ff6d5b9667e63a6b8d232b6dd93db2d8b12fd46d
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910390"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="afc78-103">Een SQL Server 2016-beschikbaarheids groep implementeren op Azure en Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="afc78-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="afc78-104">In dit artikel wordt stapsgewijs een geautomatiseerde implementatie van een basis met hoge Beschik baarheid (HA) SQL Server 2016 Enter prise-cluster met een asynchrone site voor nood herstel (DR) in twee Azure Stack hub-omgevingen.</span><span class="sxs-lookup"><span data-stu-id="afc78-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="afc78-105">Zie AlwaysOn [Availability groups: een oplossing voor hoge Beschik baarheid en herstel na nood gevallen](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)voor meer informatie over SQL Server 2016 en hoge Beschik baarheid.</span><span class="sxs-lookup"><span data-stu-id="afc78-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="afc78-106">In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:</span><span class="sxs-lookup"><span data-stu-id="afc78-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="afc78-107">Een implementatie op twee Azure Stack hubs te organiseren.</span><span class="sxs-lookup"><span data-stu-id="afc78-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="afc78-108">Gebruik docker om afhankelijkheids problemen met Azure API-profielen te minimaliseren.</span><span class="sxs-lookup"><span data-stu-id="afc78-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="afc78-109">Implementeer een Basic-cluster met hoge Beschik baarheid SQL Server 2016 voor bedrijven met een nood herstel site.</span><span class="sxs-lookup"><span data-stu-id="afc78-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="afc78-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="afc78-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="afc78-111">Microsoft Azure Stack hub is een uitbrei ding van Azure.</span><span class="sxs-lookup"><span data-stu-id="afc78-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="afc78-112">Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt gebruiken waarmee u overal hybride apps bouwt en implementeert.</span><span class="sxs-lookup"><span data-stu-id="afc78-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="afc78-113">In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps.</span><span class="sxs-lookup"><span data-stu-id="afc78-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="afc78-114">De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.</span><span class="sxs-lookup"><span data-stu-id="afc78-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="afc78-115">Architectuur voor SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="afc78-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="afc78-117">Vereisten voor SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="afc78-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="afc78-118">Twee aangesloten Azure Stack hub geïntegreerde systemen (Azure Stack hub).</span><span class="sxs-lookup"><span data-stu-id="afc78-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="afc78-119">Deze implementatie werkt niet op de Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="afc78-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="afc78-120">Zie [Azure stack Overview](https://azure.microsoft.com/overview/azure-stack/)voor meer informatie over Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="afc78-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="afc78-121">Een Tenant abonnement op elke Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="afc78-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="afc78-122">**Noteer de abonnements-ID en het Azure Resource Manager-eind punt voor elke Azure Stack hub.**</span><span class="sxs-lookup"><span data-stu-id="afc78-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="afc78-123">Een service-principal voor Azure Active Directory (Azure AD) die machtigingen heeft voor het Tenant abonnement op elke Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="afc78-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="afc78-124">Mogelijk moet u twee service-principals maken als de Azure Stack hubs worden geïmplementeerd op basis van verschillende Azure AD-tenants.</span><span class="sxs-lookup"><span data-stu-id="afc78-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="afc78-125">Zie [service-principals maken om apps toegang te geven tot Azure stack hub-resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals)voor meer informatie over het maken van een service-principal voor Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="afc78-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="afc78-126">**Noteer de toepassings-ID van elke service-principal, het client geheim en de Tenant naam (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="afc78-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="afc78-127">SQL Server 2016 Enter prise syndicated op de Marketplace van elke Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="afc78-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="afc78-128">Zie [Marketplace-items downloaden naar Azure stack hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item)voor meer informatie over Marketplace-syndicatie.</span><span class="sxs-lookup"><span data-stu-id="afc78-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="afc78-129">**Zorg ervoor dat uw organisatie beschikt over de juiste SQL-licenties.**</span><span class="sxs-lookup"><span data-stu-id="afc78-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="afc78-130">[Docker voor Windows](https://docs.docker.com/docker-for-windows/) geïnstalleerd op uw lokale computer.</span><span class="sxs-lookup"><span data-stu-id="afc78-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="afc78-131">De docker-installatie kopie ophalen</span><span class="sxs-lookup"><span data-stu-id="afc78-131">Get the Docker image</span></span>

<span data-ttu-id="afc78-132">Docker-installatie kopieën voor elke implementatie elimineren afhankelijkheids problemen tussen de verschillende versies van Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="afc78-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="afc78-133">Zorg ervoor dat docker voor Windows gebruikmaakt van Windows-containers.</span><span class="sxs-lookup"><span data-stu-id="afc78-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="afc78-134">Voer het volgende script uit in een opdracht prompt met verhoogde bevoegdheid om de docker-container met de implementatie scripts op te halen.</span><span class="sxs-lookup"><span data-stu-id="afc78-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="afc78-135">De beschikbaarheids groep implementeren</span><span class="sxs-lookup"><span data-stu-id="afc78-135">Deploy the availability group</span></span>

1. <span data-ttu-id="afc78-136">Zodra de container installatie kopie is opgehaald, start u de installatie kopie.</span><span class="sxs-lookup"><span data-stu-id="afc78-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="afc78-137">Zodra de container is gestart, krijgt u een Power shell-terminal met verhoogde bevoegdheden in de container.</span><span class="sxs-lookup"><span data-stu-id="afc78-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="afc78-138">Wijzig de mappen om het implementatie script op te halen.</span><span class="sxs-lookup"><span data-stu-id="afc78-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="afc78-139">Voer de implementatie uit.</span><span class="sxs-lookup"><span data-stu-id="afc78-139">Run the deployment.</span></span> <span data-ttu-id="afc78-140">Geef indien nodig referenties en resource namen op.</span><span class="sxs-lookup"><span data-stu-id="afc78-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="afc78-141">HA verwijst naar de Azure Stack hub waar het HA-cluster wordt geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="afc78-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="afc78-142">DR verwijst naar de Azure Stack hub waar het DR-cluster wordt geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="afc78-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="afc78-143">Typ `Y` om toe te staan dat de NuGet-provider wordt geïnstalleerd, waardoor het API-profiel "2018-03-01-Hybrid" modules moet worden geïnstalleerd.</span><span class="sxs-lookup"><span data-stu-id="afc78-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="afc78-144">Wacht tot de implementatie van de resource is voltooid.</span><span class="sxs-lookup"><span data-stu-id="afc78-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="afc78-145">Zodra de implementatie van de DR-resource is voltooid, sluit u de container.</span><span class="sxs-lookup"><span data-stu-id="afc78-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="afc78-146">Inspecteer de implementatie door de resources te bekijken in de portal van elke Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="afc78-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="afc78-147">Maak verbinding met een van de SQL-exemplaren in de HA-omgeving en controleer de beschikbaarheids groep via SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="afc78-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="afc78-149">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="afc78-149">Next steps</span></span>

- <span data-ttu-id="afc78-150">Gebruik SQL Server Management Studio hand matig failover over het cluster.</span><span class="sxs-lookup"><span data-stu-id="afc78-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="afc78-151">Zie [een geforceerde hand matige failover uitvoeren van een always on-beschikbaarheids groep (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="afc78-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="afc78-152">Meer informatie over hybride Cloud-apps.</span><span class="sxs-lookup"><span data-stu-id="afc78-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="afc78-153">Zie [hybride cloud oplossingen.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="afc78-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="afc78-154">Gebruik uw eigen gegevens of wijzig de code in dit voor beeld op [github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="afc78-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
