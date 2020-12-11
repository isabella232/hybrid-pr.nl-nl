---
title: Een SQL Server 2016-beschikbaarheids groep implementeren op Azure en Azure Stack hub
description: Meer informatie over het implementeren van een SQL Server 2016-beschikbaarheids groep naar Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0f857515a44ece7f967ade3dee8f493481709851
ms.sourcegitcommit: c890f2c5e5e5f9f93c921f02dd1a6ca5026d5289
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 12/10/2020
ms.locfileid: "97091799"
---
# <a name="deploy-a-sql-server-2016-availability-group-across-two-azure-stack-hub-environments"></a>Een SQL Server 2016-beschikbaarheids groep implementeren in twee Azure Stack hub-omgevingen

In dit artikel wordt stapsgewijs een geautomatiseerde implementatie van een basis met hoge Beschik baarheid (HA) SQL Server 2016 Enter prise-cluster met een asynchrone site voor nood herstel (DR) in twee Azure Stack hub-omgevingen. Zie AlwaysOn [Availability groups: een oplossing voor hoge Beschik baarheid en herstel na nood gevallen](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)voor meer informatie over SQL Server 2016 en hoge Beschik baarheid.

In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:

> [!div class="checklist"]
> - Een implementatie op twee Azure Stack hubs te organiseren.
> - Gebruik docker om afhankelijkheids problemen met Azure API-profielen te minimaliseren.
> - Implementeer een Basic-cluster met hoge Beschik baarheid SQL Server 2016 voor bedrijven met een nood herstel site.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt gebruiken waarmee u overal hybride apps bouwt en implementeert.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

## <a name="architecture-for-sql-server-2016"></a>Architectuur voor SQL Server 2016

![SQL Server 2016 SQL HA Azure Stack hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Vereisten voor SQL Server 2016

- Twee aangesloten Azure Stack hub geïntegreerde systemen (Azure Stack hub). Deze implementatie werkt niet op de Azure Stack Development Kit (ASDK). Zie [Azure stack Overview](https://azure.microsoft.com/overview/azure-stack/)voor meer informatie over Azure stack hub.
- Een Tenant abonnement op elke Azure Stack hub.
  - **Noteer de abonnements-ID en het Azure Resource Manager-eind punt voor elke Azure Stack hub.**
- Een service-principal voor Azure Active Directory (Azure AD) die machtigingen heeft voor het Tenant abonnement op elke Azure Stack hub. Mogelijk moet u twee service-principals maken als de Azure Stack hubs worden geïmplementeerd op basis van verschillende Azure AD-tenants. Zie [service-principals maken om apps toegang te geven tot Azure stack hub-resources](/azure-stack/user/azure-stack-create-service-principals)voor meer informatie over het maken van een service-principal voor Azure stack hub.
  - **Noteer de toepassings-ID van elke service-principal, het client geheim en de Tenant naam (xxxxx.onmicrosoft.com).**
- SQL Server 2016 Enter prise syndicated op de Marketplace van elke Azure Stack hub. Zie [Marketplace-items downloaden naar Azure stack hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item)voor meer informatie over Marketplace-syndicatie.
    **Zorg ervoor dat uw organisatie beschikt over de juiste SQL-licenties.**
- [Docker voor Windows](https://docs.docker.com/docker-for-windows/) geïnstalleerd op uw lokale computer.

## <a name="get-the-docker-image"></a>De docker-installatie kopie ophalen

Docker-installatie kopieën voor elke implementatie elimineren afhankelijkheids problemen tussen de verschillende versies van Azure PowerShell.

1. Zorg ervoor dat docker voor Windows gebruikmaakt van Windows-containers.
2. Voer het volgende script uit in een opdracht prompt met verhoogde bevoegdheid om de docker-container met de implementatie scripts op te halen.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>De beschikbaarheids groep implementeren

1. Zodra de container installatie kopie is opgehaald, start u de installatie kopie.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. Zodra de container is gestart, krijgt u een Power shell-terminal met verhoogde bevoegdheden in de container. Wijzig de mappen om het implementatie script op te halen.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Voer de implementatie uit. Geef indien nodig referenties en resource namen op. HA verwijst naar de Azure Stack hub waar het HA-cluster wordt geïmplementeerd. DR verwijst naar de Azure Stack hub waar het DR-cluster wordt geïmplementeerd.

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

4. Typ `Y` om toe te staan dat de NuGet-provider wordt geïnstalleerd, waardoor het API-profiel "2018-03-01-Hybrid" modules moet worden geïnstalleerd.

5. Wacht tot de implementatie van de resource is voltooid.

6. Zodra de implementatie van de DR-resource is voltooid, sluit u de container.

      ```powershell
      exit
      ```

7. Inspecteer de implementatie door de resources te bekijken in de portal van elke Azure Stack hub. Maak verbinding met een van de SQL-exemplaren in de HA-omgeving en controleer de beschikbaarheids groep via SQL Server Management Studio (SSMS).

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Volgende stappen

- Gebruik SQL Server Management Studio hand matig failover over het cluster. Zie [een geforceerde hand matige failover uitvoeren van een always on-beschikbaarheids groep (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- Meer informatie over hybride Cloud-apps. Zie [hybride cloud oplossingen.](/azure-stack/user/)
- Gebruik uw eigen gegevens of wijzig de code in dit voor beeld op [github](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
