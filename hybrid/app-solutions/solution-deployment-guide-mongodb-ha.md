---
title: Een Maxi maal beschik bare MongoDB-oplossing implementeren op Azure en Azure Stack hub
description: Meer informatie over het implementeren van een Maxi maal beschik bare MongoDB-oplossing voor Azure en Azure Stack hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b34ba7c10ff5f658d645923ae8b6de2fb2607ccb
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910595"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="a7030-103">Een Maxi maal beschik bare MongoDB-oplossing implementeren op Azure en Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="a7030-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="a7030-104">In dit artikel wordt stapsgewijs geautomatiseerd met een eenvoudige implementatie van een MongoDB-cluster met hoge Beschik baarheid met een nood herstel site (DR) voor twee Azure Stack hub-omgevingen.</span><span class="sxs-lookup"><span data-stu-id="a7030-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="a7030-105">Zie [leden van replica sets](https://docs.mongodb.com/manual/core/replica-set-members/)voor meer informatie over MongoDb en hoge Beschik baarheid.</span><span class="sxs-lookup"><span data-stu-id="a7030-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="a7030-106">In deze oplossing maakt u een voorbeeld omgeving voor het volgende:</span><span class="sxs-lookup"><span data-stu-id="a7030-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="a7030-107">Een implementatie op twee Azure Stack hubs te organiseren.</span><span class="sxs-lookup"><span data-stu-id="a7030-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="a7030-108">Gebruik docker om afhankelijkheids problemen met Azure API-profielen te minimaliseren.</span><span class="sxs-lookup"><span data-stu-id="a7030-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="a7030-109">Implementeer een basis MongoDB-cluster met hoge Beschik baarheid met een nood herstel site.</span><span class="sxs-lookup"><span data-stu-id="a7030-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="a7030-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="a7030-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="a7030-111">Microsoft Azure Stack hub is een uitbrei ding van Azure.</span><span class="sxs-lookup"><span data-stu-id="a7030-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="a7030-112">Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt gebruiken waarmee u overal hybride apps bouwt en implementeert.</span><span class="sxs-lookup"><span data-stu-id="a7030-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="a7030-113">In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps.</span><span class="sxs-lookup"><span data-stu-id="a7030-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="a7030-114">De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.</span><span class="sxs-lookup"><span data-stu-id="a7030-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="a7030-115">Architectuur voor MongoDB met Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="a7030-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![hoge beschik bare MongoDB-architectuur in Azure Stack hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="a7030-117">Vereisten voor MongoDB met Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="a7030-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="a7030-118">Twee aangesloten Azure Stack hub geïntegreerde systemen (Azure Stack hub).</span><span class="sxs-lookup"><span data-stu-id="a7030-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="a7030-119">Deze implementatie werkt niet op de Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="a7030-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="a7030-120">Zie [Wat is Azure stack hub?](https://azure.microsoft.com/products/azure-stack/hub/) voor meer informatie over Azure stack hub?</span><span class="sxs-lookup"><span data-stu-id="a7030-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="a7030-121">Een Tenant abonnement op elke Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="a7030-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="a7030-122">**Noteer de abonnements-ID en het Azure Resource Manager-eind punt voor elke Azure Stack hub.**</span><span class="sxs-lookup"><span data-stu-id="a7030-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="a7030-123">Een service-principal voor Azure Active Directory (Azure AD) die machtigingen heeft voor het Tenant abonnement op elke Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="a7030-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="a7030-124">Mogelijk moet u twee service-principals maken als de Azure Stack hubs worden geïmplementeerd op basis van verschillende Azure AD-tenants.</span><span class="sxs-lookup"><span data-stu-id="a7030-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="a7030-125">Zie [een app-identiteit gebruiken om toegang te krijgen tot Azure stack hub-resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals)voor meer informatie over het maken van een service-principal voor Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="a7030-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="a7030-126">**Noteer de toepassings-ID van elke service-principal, het client geheim en de Tenant naam (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="a7030-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="a7030-127">Ubuntu 16,04 is gepubliceerd naar de Marketplace van elke Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="a7030-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="a7030-128">Zie [Marketplace-items downloaden naar Azure stack hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item)voor meer informatie over Marketplace-syndicatie.</span><span class="sxs-lookup"><span data-stu-id="a7030-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="a7030-129">[Docker voor Windows](https://docs.docker.com/docker-for-windows/) geïnstalleerd op uw lokale computer.</span><span class="sxs-lookup"><span data-stu-id="a7030-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="a7030-130">De docker-installatie kopie ophalen</span><span class="sxs-lookup"><span data-stu-id="a7030-130">Get the Docker image</span></span>

<span data-ttu-id="a7030-131">Docker-installatie kopieën voor elke implementatie elimineren afhankelijkheids problemen tussen de verschillende versies van Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="a7030-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="a7030-132">Zorg ervoor dat docker voor Windows gebruikmaakt van Windows-containers.</span><span class="sxs-lookup"><span data-stu-id="a7030-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="a7030-133">Voer de volgende opdracht uit in een opdracht prompt met verhoogde bevoegdheid om de docker-container met de implementatie scripts op te halen.</span><span class="sxs-lookup"><span data-stu-id="a7030-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="a7030-134">De clusters implementeren</span><span class="sxs-lookup"><span data-stu-id="a7030-134">Deploy the clusters</span></span>

1. <span data-ttu-id="a7030-135">Zodra de container installatie kopie is opgehaald, start u de installatie kopie.</span><span class="sxs-lookup"><span data-stu-id="a7030-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="a7030-136">Zodra de container is gestart, krijgt u een Power shell-terminal met verhoogde bevoegdheden in de container.</span><span class="sxs-lookup"><span data-stu-id="a7030-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="a7030-137">Wijzig de mappen om het implementatie script op te halen.</span><span class="sxs-lookup"><span data-stu-id="a7030-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="a7030-138">Voer de implementatie uit.</span><span class="sxs-lookup"><span data-stu-id="a7030-138">Run the deployment.</span></span> <span data-ttu-id="a7030-139">Geef indien nodig referenties en resource namen op.</span><span class="sxs-lookup"><span data-stu-id="a7030-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="a7030-140">HA verwijst naar de Azure Stack hub waar het HA-cluster wordt geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="a7030-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="a7030-141">DR verwijst naar de Azure Stack hub waar het DR-cluster wordt geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="a7030-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="a7030-142">Typ `Y` om toe te staan dat de NuGet-provider wordt geïnstalleerd, waardoor het API-profiel "2018-03-01-Hybrid" modules moet worden geïnstalleerd.</span><span class="sxs-lookup"><span data-stu-id="a7030-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="a7030-143">De HA-bronnen worden eerst geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="a7030-143">The HA resources will deploy first.</span></span> <span data-ttu-id="a7030-144">Controleer de implementatie en wacht totdat deze is voltooid.</span><span class="sxs-lookup"><span data-stu-id="a7030-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="a7030-145">Zodra het bericht wordt weer gegeven dat de HA-implementatie is voltooid, kunt u de Azure Stack-hub van de HA controleren om de resources te zien die zijn geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="a7030-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="a7030-146">Ga verder met de implementatie van DR-resources en beslis of u een Jump box op de DR Azure Stack hub wilt inschakelen om met het cluster te communiceren.</span><span class="sxs-lookup"><span data-stu-id="a7030-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="a7030-147">Wacht tot de implementatie van de DR-resource is voltooid.</span><span class="sxs-lookup"><span data-stu-id="a7030-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="a7030-148">Zodra de implementatie van de DR-resource is voltooid, sluit u de container.</span><span class="sxs-lookup"><span data-stu-id="a7030-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="a7030-149">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="a7030-149">Next steps</span></span>

- <span data-ttu-id="a7030-150">Als u de VM met het Jump box op de DR Azure Stack hub hebt ingeschakeld, kunt u verbinding maken via SSH en met het MongoDB-cluster communiceren door de Mongo CLI te installeren.</span><span class="sxs-lookup"><span data-stu-id="a7030-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="a7030-151">Zie [de Mongo-shell](https://docs.mongodb.com/manual/mongo/)voor meer informatie over interactie met MongoDb.</span><span class="sxs-lookup"><span data-stu-id="a7030-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="a7030-152">Zie [hybride cloud oplossingen](https://aka.ms/azsdevtutorials) voor meer informatie over hybride Cloud-apps.</span><span class="sxs-lookup"><span data-stu-id="a7030-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="a7030-153">Wijzig de code in dit voor beeld op [github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="a7030-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
