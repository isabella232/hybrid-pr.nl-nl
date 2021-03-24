---
title: De hybride Cloud-identiteit voor Azure-en Azure Stack hub-apps configureren
description: Meer informatie over het configureren van een hybride Cloud-identiteit voor Azure en Azure Stack hub-apps.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895343"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="6e4bc-103">De hybride Cloud-identiteit voor Azure-en Azure Stack hub-apps configureren</span><span class="sxs-lookup"><span data-stu-id="6e4bc-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="6e4bc-104">Meer informatie over het configureren van een hybride Cloud-identiteit voor uw Azure-en Azure Stack hub-apps.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="6e4bc-105">U hebt twee opties voor het verlenen van toegang tot uw apps in zowel de wereld wijde Azure-als Azure Stack-hub.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="6e4bc-106">Als Azure Stack hub een continue verbinding met internet heeft, kunt u Azure Active Directory (Azure AD) gebruiken.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="6e4bc-107">Als Azure Stack hub wordt losgekoppeld van het Internet, kunt u Azure Directory Federated Services (AD FS) gebruiken.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="6e4bc-108">U kunt Service-principals gebruiken om toegang te verlenen tot uw Azure Stack hub-apps voor implementatie of configuratie met behulp van de Azure Resource Manager in Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="6e4bc-109">In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:</span><span class="sxs-lookup"><span data-stu-id="6e4bc-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6e4bc-110">Een hybride identiteit instellen in de wereld wijde Azure-en Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="6e4bc-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="6e4bc-111">Een Token ophalen om toegang te krijgen tot de Azure Stack hub-API.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="6e4bc-112">U moet Azure Stack hub-operator machtigingen hebben voor de stappen in deze oplossing.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="6e4bc-113">![Diagram hybride pijlers](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="6e4bc-113">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="6e4bc-114">Microsoft Azure Stack hub is een uitbrei ding van Azure.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="6e4bc-115">Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt gebruiken waarmee u overal hybride apps bouwt en implementeert.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="6e4bc-116">In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="6e4bc-117">De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="6e4bc-118">Een service-principal voor Azure AD maken in de portal</span><span class="sxs-lookup"><span data-stu-id="6e4bc-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="6e4bc-119">Als u Azure Stack hub hebt geïmplementeerd met behulp van Azure AD als identiteits opslag, kunt u service-principals maken op dezelfde manier als voor Azure.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="6e4bc-120">[Gebruik een app-identiteit voor toegang tot bronnen voor](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) meer informatie over het uitvoeren van de stappen via de portal.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="6e4bc-121">Zorg ervoor dat u over de [vereiste Azure AD-machtigingen](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) beschikt voordat u begint.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="6e4bc-122">Een service-principal maken voor AD FS met behulp van Power shell</span><span class="sxs-lookup"><span data-stu-id="6e4bc-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="6e4bc-123">Als u Azure Stack hub met AD FS hebt geïmplementeerd, kunt u Power shell gebruiken om een service-principal te maken, een rol voor toegang toe te wijzen en u aan te melden vanuit Power shell met die identiteit.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="6e4bc-124">[Gebruik een app-identiteit voor toegang tot bronnen om](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) te zien hoe u de vereiste stappen kunt uitvoeren met behulp van Power shell.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="6e4bc-125">De Azure Stack hub-API gebruiken</span><span class="sxs-lookup"><span data-stu-id="6e4bc-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="6e4bc-126">De [Azure stack hub API](/azure-stack/user/azure-stack-rest-api-use)  -oplossing begeleidt u stapsgewijs door het proces van het ophalen van een token voor toegang tot de Azure stack hub-API.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="6e4bc-127">Verbinding maken met Azure Stack hub met behulp van Power shell</span><span class="sxs-lookup"><span data-stu-id="6e4bc-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="6e4bc-128">De Snelstartgids [om aan de slag te gaan met Power shell in azure stack hub](/azure-stack/operator/azure-stack-powershell-install) begeleidt u bij de stappen die nodig zijn om Azure PowerShell te installeren en verbinding te maken met de installatie van Azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="6e4bc-129">Vereisten</span><span class="sxs-lookup"><span data-stu-id="6e4bc-129">Prerequisites</span></span>

<span data-ttu-id="6e4bc-130">U hebt een Azure Stack hub-installatie die is verbonden met Azure AD, een abonnement waartoe u toegang hebt.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="6e4bc-131">Als u geen Azure Stack hub-installatie hebt, kunt u deze instructies gebruiken om een [Azure stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install)in te stellen.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="6e4bc-132">Verbinding maken met Azure Stack hub met behulp van code</span><span class="sxs-lookup"><span data-stu-id="6e4bc-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="6e4bc-133">Als u verbinding wilt maken met Azure Stack hub met behulp van code, gebruikt u de API voor de Azure Resource Manager-eind punten om de verificatie-en Graph-eind punten voor uw Azure Stack hub-installatie te verkrijgen.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="6e4bc-134">Verifieer vervolgens met behulp van REST-aanvragen.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="6e4bc-135">U kunt een voor beeld-client toepassing vinden op [github](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="6e4bc-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="6e4bc-136">Tenzij de Azure SDK voor uw taal keuze Azure API-profielen ondersteunt, werkt de SDK mogelijk niet met Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="6e4bc-137">Zie het artikel [API-versie profielen beheren](/azure-stack/user/azure-stack-version-profiles) voor meer informatie over Azure API-profielen.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6e4bc-138">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="6e4bc-138">Next steps</span></span>

- <span data-ttu-id="6e4bc-139">Zie [identiteits architectuur voor Azure stack hub](/azure-stack/operator/azure-stack-identity-architecture)voor meer informatie over hoe identiteiten worden afgehandeld in azure stack hub.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span></span>
- <span data-ttu-id="6e4bc-140">Zie [Cloud ontwerp patronen](/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.</span><span class="sxs-lookup"><span data-stu-id="6e4bc-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
