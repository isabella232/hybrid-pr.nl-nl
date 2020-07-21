---
title: Een Maxi maal beschik bare MongoDB-oplossing implementeren op Azure en Azure Stack hub
description: Meer informatie over het implementeren van een Maxi maal beschik bare MongoDB-oplossing voor Azure en Azure Stack hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: f6064aaa1087a3c0cfc26e09371e81752c777edb
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477266"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Een Maxi maal beschik bare MongoDB-oplossing implementeren op Azure en Azure Stack hub

In dit artikel wordt stapsgewijs geautomatiseerd met een eenvoudige implementatie van een MongoDB-cluster met hoge Beschik baarheid met een nood herstel site (DR) voor twee Azure Stack hub-omgevingen. Zie [leden van replica sets](https://docs.mongodb.com/manual/core/replica-set-members/)voor meer informatie over MongoDb en hoge Beschik baarheid.

In deze oplossing maakt u een voorbeeld omgeving voor het volgende:

> [!div class="checklist"]
> - Een implementatie op twee Azure Stack hubs te organiseren.
> - Gebruik docker om afhankelijkheids problemen met Azure API-profielen te minimaliseren.
> - Implementeer een basis MongoDB-cluster met hoge Beschik baarheid met een nood herstel site.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt gebruiken waarmee u overal hybride apps bouwt en implementeert.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Architectuur voor MongoDB met Azure Stack hub

![hoge beschik bare MongoDB-architectuur in Azure Stack hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Vereisten voor MongoDB met Azure Stack hub

- Twee aangesloten Azure Stack hub geïntegreerde systemen (Azure Stack hub). Deze implementatie werkt niet op de Azure Stack Development Kit (ASDK). Zie [Wat is Azure stack hub?](https://azure.microsoft.com/products/azure-stack/hub/) voor meer informatie over Azure stack hub?
  - Een Tenant abonnement op elke Azure Stack hub. 
  - **Noteer de abonnements-ID en het Azure Resource Manager-eind punt voor elke Azure Stack hub.**
- Een service-principal voor Azure Active Directory (Azure AD) die machtigingen heeft voor het Tenant abonnement op elke Azure Stack hub. Mogelijk moet u twee service-principals maken als de Azure Stack hubs worden geïmplementeerd op basis van verschillende Azure AD-tenants. Zie [een app-identiteit gebruiken om toegang te krijgen tot Azure stack hub-resources](/azure-stack/user/azure-stack-create-service-principals)voor meer informatie over het maken van een service-principal voor Azure stack hub.
  - **Noteer de toepassings-ID van elke service-principal, het client geheim en de Tenant naam (xxxxx.onmicrosoft.com).**
- Ubuntu 16,04 is gepubliceerd naar de Marketplace van elke Azure Stack hub. Zie [Marketplace-items downloaden naar Azure stack hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item)voor meer informatie over Marketplace-syndicatie.
- [Docker voor Windows](https://docs.docker.com/docker-for-windows/) geïnstalleerd op uw lokale computer.

## <a name="get-the-docker-image"></a>De docker-installatie kopie ophalen

Docker-installatie kopieën voor elke implementatie elimineren afhankelijkheids problemen tussen de verschillende versies van Azure PowerShell.

1. Zorg ervoor dat docker voor Windows gebruikmaakt van Windows-containers.
2. Voer de volgende opdracht uit in een opdracht prompt met verhoogde bevoegdheid om de docker-container met de implementatie scripts op te halen.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>De clusters implementeren

1. Zodra de container installatie kopie is opgehaald, start u de installatie kopie.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Zodra de container is gestart, krijgt u een Power shell-terminal met verhoogde bevoegdheden in de container. Wijzig de mappen om het implementatie script op te halen.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Voer de implementatie uit. Geef indien nodig referenties en resource namen op. HA verwijst naar de Azure Stack hub waar het HA-cluster wordt geïmplementeerd. DR verwijst naar de Azure Stack hub waar het DR-cluster wordt geïmplementeerd.

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

4. Typ `Y` om toe te staan dat de NuGet-provider wordt geïnstalleerd, waardoor het API-profiel "2018-03-01-Hybrid" modules moet worden geïnstalleerd.

5. De HA-bronnen worden eerst geïmplementeerd. Controleer de implementatie en wacht totdat deze is voltooid. Zodra het bericht wordt weer gegeven dat de HA-implementatie is voltooid, kunt u de Azure Stack-hub van de HA controleren om de resources te zien die zijn geïmplementeerd.

6. Ga verder met de implementatie van DR-resources en beslis of u een Jump box op de DR Azure Stack hub wilt inschakelen om met het cluster te communiceren.

7. Wacht tot de implementatie van de DR-resource is voltooid.

8. Zodra de implementatie van de DR-resource is voltooid, sluit u de container.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Volgende stappen

- Als u de VM met het Jump box op de DR Azure Stack hub hebt ingeschakeld, kunt u verbinding maken via SSH en met het MongoDB-cluster communiceren door de Mongo CLI te installeren. Zie [de Mongo-shell](https://docs.mongodb.com/manual/mongo/)voor meer informatie over interactie met MongoDb.
- Zie [hybride cloud oplossingen](https://aka.ms/azsdevtutorials) voor meer informatie over hybride Cloud-apps.
- Wijzig de code in dit voor beeld op [github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
