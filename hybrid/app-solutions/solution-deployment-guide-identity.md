---
title: De hybride Cloud-identiteit voor Azure-en Azure Stack hub-apps configureren
description: Meer informatie over het configureren van een hybride Cloud-identiteit voor Azure en Azure Stack hub-apps.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477249"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>De hybride Cloud-identiteit voor Azure-en Azure Stack hub-apps configureren

Meer informatie over het configureren van een hybride Cloud-identiteit voor uw Azure-en Azure Stack hub-apps.

U hebt twee opties voor het verlenen van toegang tot uw apps in zowel de wereld wijde Azure-als Azure Stack-hub.

 * Als Azure Stack hub een continue verbinding met internet heeft, kunt u Azure Active Directory (Azure AD) gebruiken.
 * Als Azure Stack hub wordt losgekoppeld van het Internet, kunt u Azure Directory Federated Services (AD FS) gebruiken.

U kunt Service-principals gebruiken om toegang te verlenen tot uw Azure Stack hub-apps voor implementatie of configuratie met behulp van de Azure Resource Manager in Azure Stack hub.

In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:

> [!div class="checklist"]
> - Een hybride identiteit instellen in de wereld wijde Azure-en Azure Stack hub
> - Een Token ophalen om toegang te krijgen tot de Azure Stack hub-API.

U moet Azure Stack hub-operator machtigingen hebben voor de stappen in deze oplossing.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt gebruiken waarmee u overal hybride apps bouwt en implementeert.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Een service-principal voor Azure AD maken in de portal

Als u Azure Stack hub hebt geïmplementeerd met behulp van Azure AD als identiteits opslag, kunt u service-principals maken op dezelfde manier als voor Azure. [Gebruik een app-identiteit voor toegang tot bronnen voor](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) meer informatie over het uitvoeren van de stappen via de portal. Zorg ervoor dat u over de [vereiste Azure AD-machtigingen](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) beschikt voordat u begint.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Een service-principal maken voor AD FS met behulp van Power shell

Als u Azure Stack hub met AD FS hebt geïmplementeerd, kunt u Power shell gebruiken om een service-principal te maken, een rol voor toegang toe te wijzen en u aan te melden vanuit Power shell met die identiteit. [Gebruik een app-identiteit voor toegang tot bronnen om](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) te zien hoe u de vereiste stappen kunt uitvoeren met behulp van Power shell.

## <a name="using-the-azure-stack-hub-api"></a>De Azure Stack hub-API gebruiken

De [Azure stack hub API](/azure-stack/user/azure-stack-rest-api-use.md) -oplossing begeleidt u stapsgewijs door het proces van het ophalen van een token voor toegang tot de Azure stack hub-API.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Verbinding maken met Azure Stack hub met behulp van Power shell

De Snelstartgids [om aan de slag te gaan met Power shell in azure stack hub](/azure-stack/operator/azure-stack-powershell-install.md) begeleidt u bij de stappen die nodig zijn om Azure PowerShell te installeren en verbinding te maken met de installatie van Azure stack hub.

### <a name="prerequisites"></a>Vereisten

U hebt een Azure Stack hub-installatie die is verbonden met Azure AD, een abonnement waartoe u toegang hebt. Als u geen Azure Stack hub-installatie hebt, kunt u deze instructies gebruiken om een [Azure stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md)in te stellen.

#### <a name="connect-to-azure-stack-hub-using-code"></a>Verbinding maken met Azure Stack hub met behulp van code

Als u verbinding wilt maken met Azure Stack hub met behulp van code, gebruikt u de API voor de Azure Resource Manager-eind punten om de verificatie-en Graph-eind punten voor uw Azure Stack hub-installatie te verkrijgen. Verifieer vervolgens met behulp van REST-aanvragen. U kunt een voor beeld-client toepassing vinden op [github](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>Tenzij de Azure SDK voor uw taal keuze Azure API-profielen ondersteunt, werkt de SDK mogelijk niet met Azure Stack hub. Zie het artikel [API-versie profielen beheren](/azure-stack/user/azure-stack-version-profiles.md) voor meer informatie over Azure API-profielen.

## <a name="next-steps"></a>Volgende stappen

- Zie [identiteits architectuur voor Azure stack hub](/azure-stack/operator/azure-stack-identity-architecture.md)voor meer informatie over hoe identiteiten worden afgehandeld in azure stack hub.
- Zie [Cloud ontwerp patronen](/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.
