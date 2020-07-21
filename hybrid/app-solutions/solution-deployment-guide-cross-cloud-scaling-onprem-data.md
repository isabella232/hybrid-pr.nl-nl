---
title: Implementeer hybride app met on-premises gegevens die meerdere clouds schalen
description: Meer informatie over het implementeren van een app die gebruikmaakt van on-premises gegevens en het schalen van meerdere clouds met behulp van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6de35cb55c4c35a2a9927f9ffc2516ccb00cd89f
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477317"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="76cfb-103">Implementeer hybride app met on-premises gegevens die meerdere clouds schalen</span><span class="sxs-lookup"><span data-stu-id="76cfb-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="76cfb-104">In deze oplossings handleiding ziet u hoe u een hybride app implementeert die zowel Azure als Azure Stack hub omvat en één on-premises gegevens bron gebruikt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="76cfb-105">Met een hybride Cloud oplossing kunt u de nalevings voordelen van een privécloud combi neren met de schaal baarheid van de open bare Cloud.</span><span class="sxs-lookup"><span data-stu-id="76cfb-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="76cfb-106">Uw ontwikkel aars kunnen ook profiteren van het micro soft Developer ecosysteem en hun vaardig heden Toep assen op de Cloud-en on-premises omgevingen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="76cfb-107">Overzicht en hypo Thesen</span><span class="sxs-lookup"><span data-stu-id="76cfb-107">Overview and assumptions</span></span>

<span data-ttu-id="76cfb-108">Volg deze zelf studie om een werk stroom in te stellen waarmee ontwikkel aars een identieke Web-app kunnen implementeren in een open bare Cloud en een privécloud.</span><span class="sxs-lookup"><span data-stu-id="76cfb-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="76cfb-109">Deze app kan toegang krijgen tot een niet-Internet routeerbaar netwerk dat wordt gehost op de privécloud.</span><span class="sxs-lookup"><span data-stu-id="76cfb-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="76cfb-110">Deze web-apps worden bewaakt en wanneer er een piek in het verkeer is, worden de DNS-records gewijzigd om verkeer om te leiden naar de open bare Cloud.</span><span class="sxs-lookup"><span data-stu-id="76cfb-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="76cfb-111">Wanneer het verkeer naar het niveau voor de Prikker daalt, wordt het verkeer teruggestuurd naar de privécloud.</span><span class="sxs-lookup"><span data-stu-id="76cfb-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="76cfb-112">Deze zelfstudie bestaat uit de volgende taken:</span><span class="sxs-lookup"><span data-stu-id="76cfb-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="76cfb-113">Implementeer een SQL Server-database server met hybride verbinding.</span><span class="sxs-lookup"><span data-stu-id="76cfb-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="76cfb-114">Een web-app in een globaal Azure verbinden met een hybride netwerk.</span><span class="sxs-lookup"><span data-stu-id="76cfb-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="76cfb-115">Configureer DNS voor het schalen van meerdere clouds.</span><span class="sxs-lookup"><span data-stu-id="76cfb-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="76cfb-116">Configureer SSL-certificaten voor het schalen van meerdere clouds.</span><span class="sxs-lookup"><span data-stu-id="76cfb-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="76cfb-117">De web-app configureren en implementeren.</span><span class="sxs-lookup"><span data-stu-id="76cfb-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="76cfb-118">Maak een Traffic Manager profiel en configureer dit voor cross-Cloud schaling.</span><span class="sxs-lookup"><span data-stu-id="76cfb-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="76cfb-119">Application Insights bewaking en waarschuwingen instellen voor verhoogd verkeer.</span><span class="sxs-lookup"><span data-stu-id="76cfb-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="76cfb-120">Configureer automatisch verkeer overschakelen tussen de wereld wijde Azure-en Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="76cfb-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="76cfb-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="76cfb-122">Microsoft Azure Stack hub is een uitbrei ding van Azure.</span><span class="sxs-lookup"><span data-stu-id="76cfb-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="76cfb-123">Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.</span><span class="sxs-lookup"><span data-stu-id="76cfb-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="76cfb-124">In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps.</span><span class="sxs-lookup"><span data-stu-id="76cfb-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="76cfb-125">De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.</span><span class="sxs-lookup"><span data-stu-id="76cfb-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="76cfb-126">Aannames</span><span class="sxs-lookup"><span data-stu-id="76cfb-126">Assumptions</span></span>

<span data-ttu-id="76cfb-127">In deze zelf studie wordt ervan uitgegaan dat u een basis kennis hebt van wereld wijd Azure en Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="76cfb-128">Als u meer informatie wilt over het starten van de zelf studie, raadpleegt u de volgende artikelen:</span><span class="sxs-lookup"><span data-stu-id="76cfb-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="76cfb-129">Inleiding tot Azure</span><span class="sxs-lookup"><span data-stu-id="76cfb-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="76cfb-130">Hoofd concepten van Azure Stack-hub</span><span class="sxs-lookup"><span data-stu-id="76cfb-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="76cfb-131">In deze zelf studie wordt ervan uitgegaan dat u een Azure-abonnement hebt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="76cfb-132">Als u nog geen abonnement hebt, [maakt u een gratis account](https://azure.microsoft.com/free/) voordat u begint.</span><span class="sxs-lookup"><span data-stu-id="76cfb-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="76cfb-133">Vereisten</span><span class="sxs-lookup"><span data-stu-id="76cfb-133">Prerequisites</span></span>

<span data-ttu-id="76cfb-134">Controleer voordat u met deze oplossing begint of u aan de volgende vereisten voldoet:</span><span class="sxs-lookup"><span data-stu-id="76cfb-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="76cfb-135">Een Azure Stack Development Kit (ASDK) of een abonnement op een met Azure Stack hub geïntegreerd systeem.</span><span class="sxs-lookup"><span data-stu-id="76cfb-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="76cfb-136">Als u de ASDK wilt implementeren, volgt u de instructies in [de ASDK implementeren met behulp van het installatie programma](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="76cfb-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="76cfb-137">Voor de installatie van de Azure Stack hub moet het volgende zijn geïnstalleerd:</span><span class="sxs-lookup"><span data-stu-id="76cfb-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="76cfb-138">Het Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="76cfb-138">The Azure App Service.</span></span> <span data-ttu-id="76cfb-139">Werk samen met uw Azure Stack hub-operator om de Azure App Service te implementeren en te configureren in uw omgeving.</span><span class="sxs-lookup"><span data-stu-id="76cfb-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="76cfb-140">In deze zelf studie moet het App Service ten minste één (1) beschik bare toegewezen werk rollen hebben.</span><span class="sxs-lookup"><span data-stu-id="76cfb-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="76cfb-141">Een installatie kopie van Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="76cfb-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="76cfb-142">Een Windows Server 2016 met een Microsoft SQL Server-installatie kopie.</span><span class="sxs-lookup"><span data-stu-id="76cfb-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="76cfb-143">De juiste plannen en aanbiedingen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="76cfb-144">Een domein naam voor uw web-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-144">A domain name for your web app.</span></span> <span data-ttu-id="76cfb-145">Als u geen domein naam hebt, kunt u er een kopen bij een domein provider zoals GoDaddy, bluehost en inbeweging.</span><span class="sxs-lookup"><span data-stu-id="76cfb-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="76cfb-146">Een SSL-certificaat voor uw domein van een vertrouwde certificerings instantie, zoals LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="76cfb-147">Een web-app die communiceert met een SQL Server-Data Base en die Application Insights ondersteunt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="76cfb-148">U kunt de voor beeld-app voor [dotnetcore-sqldb-zelf studie](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) downloaden van github.</span><span class="sxs-lookup"><span data-stu-id="76cfb-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="76cfb-149">Een hybride netwerk tussen een virtueel Azure-netwerk en Azure Stack hub-netwerk.</span><span class="sxs-lookup"><span data-stu-id="76cfb-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="76cfb-150">Zie [Configure Hybrid Cloud connectivity with Azure and Azure stack hub](solution-deployment-guide-connectivity.md)(Engelstalig) voor gedetailleerde instructies.</span><span class="sxs-lookup"><span data-stu-id="76cfb-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="76cfb-151">Een ' CI/CD-pijp lijn met hybride integratie/continue implementatie ' met een Private build-agent op Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="76cfb-152">Zie [Configure Hybrid Cloud identity with Azure and Azure stack hub apps](solution-deployment-guide-identity.md)(Engelstalig) voor gedetailleerde instructies.</span><span class="sxs-lookup"><span data-stu-id="76cfb-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="76cfb-153">Een SQL Server database server met hybride verbinding implementeren</span><span class="sxs-lookup"><span data-stu-id="76cfb-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="76cfb-154">Meld u aan bij de gebruikers portal van de Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="76cfb-155">Selecteer **Marketplace**in het **dash board**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="76cfb-157">Selecteer **Compute**in **Marketplace**en kies vervolgens **meer**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="76cfb-158">Selecteer onder **meer**de **gratis licentie voor SQL Server: SQL Server 2017 Developer op Windows Server** -installatie kopie.</span><span class="sxs-lookup"><span data-stu-id="76cfb-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Een installatie kopie van een virtuele machine in Azure Stack hub-gebruikers Portal selecteren](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="76cfb-160">Selecteer in **gratis SQL Server licentie: SQL Server 2017 Developer op Windows Server**de optie **maken**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="76cfb-161">Geef bij **basis beginselen > basis instellingen configureren**een **naam** op voor de virtuele machine (VM), een **gebruikers naam** voor de SQL Server SA en een **wacht woord** voor de SA.</span><span class="sxs-lookup"><span data-stu-id="76cfb-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="76cfb-162">Selecteer in de vervolg keuzelijst **abonnement** het abonnement dat u implementeert.</span><span class="sxs-lookup"><span data-stu-id="76cfb-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="76cfb-163">Gebruik voor **resource groep**de **optie bestaande** en plaats de virtuele machine in dezelfde Resource groep als uw Azure stack hub-web-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Basis instellingen configureren voor virtuele machines in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="76cfb-165">Kies onder **grootte**een grootte voor de virtuele machine.</span><span class="sxs-lookup"><span data-stu-id="76cfb-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="76cfb-166">Voor deze zelf studie raden wij u aan A2_Standard of een DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="76cfb-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="76cfb-167">Configureer onder **instellingen > optionele functies configureren**de volgende instellingen:</span><span class="sxs-lookup"><span data-stu-id="76cfb-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="76cfb-168">**Opslag account**: Maak een nieuw account als u er een hebt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="76cfb-169">**Virtueel netwerk**:</span><span class="sxs-lookup"><span data-stu-id="76cfb-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="76cfb-170">Zorg ervoor dat uw SQL Server-VM wordt geïmplementeerd op hetzelfde virtuele netwerk als de VPN-gateways.</span><span class="sxs-lookup"><span data-stu-id="76cfb-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="76cfb-171">**Openbaar IP-adres**: gebruik de standaard instellingen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="76cfb-172">**Netwerk beveiligings groep**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="76cfb-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="76cfb-173">Maak een nieuwe NSG.</span><span class="sxs-lookup"><span data-stu-id="76cfb-173">Create a new NSG.</span></span>
   - <span data-ttu-id="76cfb-174">**Extensies en controle**: behoud de standaard instellingen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="76cfb-175">**Opslag account voor diagnostische gegevens**: Maak een nieuw account als u er een hebt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="76cfb-176">Selecteer **OK** om uw configuratie op te slaan.</span><span class="sxs-lookup"><span data-stu-id="76cfb-176">Select **OK** to save your configuration.</span></span>

     ![Optionele VM-functies configureren in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="76cfb-178">Configureer onder **SQL Server instellingen**de volgende instellingen:</span><span class="sxs-lookup"><span data-stu-id="76cfb-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="76cfb-179">Selecteer voor **SQL**-connectiviteit **openbaar (Internet)**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="76cfb-180">Voor **poort**, behoud de standaard waarde van **1433**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="76cfb-181">Selecteer **inschakelen**voor **SQL-verificatie**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="76cfb-182">Wanneer u SQL-verificatie inschakelt, moet deze automatisch worden ingevuld met de ' SQLAdmin-informatie die u hebt geconfigureerd in de **basis beginselen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="76cfb-183">Behoud de standaard waarden voor de overige instellingen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="76cfb-184">Selecteer **OK**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-184">Select **OK**.</span></span>

     ![SQL Server-instellingen configureren in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="76cfb-186">Controleer bij **samen vatting**de configuratie van de virtuele machine en selecteer vervolgens **OK** om de implementatie te starten.</span><span class="sxs-lookup"><span data-stu-id="76cfb-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Configuratie samenvatting in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="76cfb-188">Het duurt enige tijd om de nieuwe virtuele machine te maken.</span><span class="sxs-lookup"><span data-stu-id="76cfb-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="76cfb-189">U kunt de STATUS van uw Vm's bekijken in **virtuele machines**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Status van virtuele machines in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="76cfb-191">Web-apps maken in Azure en Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="76cfb-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="76cfb-192">De Azure App Service vereenvoudigt het uitvoeren en beheren van een web-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="76cfb-193">Omdat Azure Stack hub consistent is met Azure, kan de App Service in beide omgevingen worden uitgevoerd.</span><span class="sxs-lookup"><span data-stu-id="76cfb-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="76cfb-194">U gebruikt de App Service om uw app te hosten.</span><span class="sxs-lookup"><span data-stu-id="76cfb-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="76cfb-195">Web-apps maken</span><span class="sxs-lookup"><span data-stu-id="76cfb-195">Create web apps</span></span>

1. <span data-ttu-id="76cfb-196">Maak een web-app in azure door de instructies te volgen in [een app service-abonnement beheren in azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="76cfb-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="76cfb-197">Zorg ervoor dat u de web-app in hetzelfde abonnement en dezelfde resource groep plaatst als uw hybride netwerk.</span><span class="sxs-lookup"><span data-stu-id="76cfb-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="76cfb-198">Herhaal de vorige stap (1) in Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="76cfb-199">Route voor Azure Stack hub toevoegen</span><span class="sxs-lookup"><span data-stu-id="76cfb-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="76cfb-200">De App Service op Azure Stack hub moet routeerbaar zijn van het open bare Internet om gebruikers toegang te geven tot uw app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="76cfb-201">Als uw Azure Stack hub toegankelijk is via internet, noteert u het open bare IP-adres of de URL voor de web-app Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="76cfb-202">Als u een ASDK gebruikt, kunt u [een statische NAT-toewijzing configureren](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) om app service buiten de virtuele omgeving zichtbaar te maken.</span><span class="sxs-lookup"><span data-stu-id="76cfb-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="76cfb-203">Een web-app in azure verbinden met een hybride netwerk</span><span class="sxs-lookup"><span data-stu-id="76cfb-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="76cfb-204">Als u de verbinding tussen de Webfront-end in Azure en de SQL Server-data base in Azure Stack hub wilt maken, moet u de web-app verbinden met het hybride netwerk tussen Azure en Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="76cfb-205">Als u connectiviteit wilt inschakelen, moet u het volgende doen:</span><span class="sxs-lookup"><span data-stu-id="76cfb-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="76cfb-206">Configureer punt-naar-site-connectiviteit.</span><span class="sxs-lookup"><span data-stu-id="76cfb-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="76cfb-207">De web-app configureren.</span><span class="sxs-lookup"><span data-stu-id="76cfb-207">Configure the web app.</span></span>
- <span data-ttu-id="76cfb-208">Wijzig de lokale netwerk gateway in Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="76cfb-209">Het virtuele Azure-netwerk configureren voor punt-naar-site-connectiviteit</span><span class="sxs-lookup"><span data-stu-id="76cfb-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="76cfb-210">De virtuele netwerk gateway aan de Azure-kant van het hybride netwerk moet punt-naar-site-verbindingen toestaan om te integreren met Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="76cfb-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="76cfb-211">Ga in azure naar de pagina virtuele netwerk gateway.</span><span class="sxs-lookup"><span data-stu-id="76cfb-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="76cfb-212">Onder **instellingen**selecteert u **punt-naar-site-configuratie**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Punt-naar-site optie in azure Virtual Network-gateway](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="76cfb-214">Selecteer **nu configureren** om punt-naar-site te configureren.</span><span class="sxs-lookup"><span data-stu-id="76cfb-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Begin punt-naar-site-configuratie in virtuele Azure-netwerk gateway](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="76cfb-216">Voer op de pagina **punt-naar-site** -configuratie het privé-IP-adres bereik in dat u wilt gebruiken in de **adres groep**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="76cfb-217">Zorg ervoor dat het bereik dat u opgeeft, niet overlapt met een van de adresbereiken die al worden gebruikt door subnetten in de globale onderdelen van Azure of Azure Stack hub van het hybride netwerk.</span><span class="sxs-lookup"><span data-stu-id="76cfb-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="76cfb-218">Schakel onder **Tunnel Type**het selectie vakje **IKEv2 VPN**uit.</span><span class="sxs-lookup"><span data-stu-id="76cfb-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="76cfb-219">Selecteer **Opslaan** om het configureren van punt-naar-site te volt ooien.</span><span class="sxs-lookup"><span data-stu-id="76cfb-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Punt-naar-site-instellingen in virtuele Azure-netwerk gateway](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="76cfb-221">De Azure App Service-app integreren met het hybride netwerk</span><span class="sxs-lookup"><span data-stu-id="76cfb-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="76cfb-222">Als u de app wilt verbinden met Azure VNet, volgt u de instructies in de [Gateway vereiste VNet-integratie](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="76cfb-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="76cfb-223">Ga naar **instellingen** voor het app service plan dat als host fungeert voor de web-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="76cfb-224">Selecteer in **instellingen**de optie **netwerken**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-224">In **Settings**, select **Networking**.</span></span>

    ![Netwerken configureren voor het App Service-abonnement](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="76cfb-226">Selecteer in **VNET-integratie** **Klik hier om te beheren**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![VNET-integratie voor het App Service plan beheren](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="76cfb-228">Selecteer het VNET dat u wilt configureren.</span><span class="sxs-lookup"><span data-stu-id="76cfb-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="76cfb-229">Voer onder **IP-adressen GEROUTEERD naar VNET**het IP-adres bereik in voor de Azure VNET, het Azure stack hub VNET en de punt-naar-site-adres ruimten.</span><span class="sxs-lookup"><span data-stu-id="76cfb-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="76cfb-230">Selecteer **Opslaan** om deze instellingen te valideren en op te slaan.</span><span class="sxs-lookup"><span data-stu-id="76cfb-230">Select **Save** to validate and save these settings.</span></span>

    ![IP-adresbereiken voor route ring in Virtual Network integratie](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="76cfb-232">Zie [uw app integreren met een Azure-Virtual Network](/azure/app-service/web-sites-integrate-with-vnet)voor meer informatie over de manier waarop app service integreert met Azure VNets.</span><span class="sxs-lookup"><span data-stu-id="76cfb-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="76cfb-233">Het virtuele netwerk van de Azure Stack hub configureren</span><span class="sxs-lookup"><span data-stu-id="76cfb-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="76cfb-234">De lokale netwerk gateway in het virtuele netwerk Azure Stack hub moet worden geconfigureerd om verkeer te routeren vanuit het App Service punt-naar-site-adres bereik.</span><span class="sxs-lookup"><span data-stu-id="76cfb-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="76cfb-235">Ga in Azure Stack hub naar de **lokale netwerk gateway**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="76cfb-236">Selecteer onder **Instellingen** de optie **Configuratie**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-236">Under **Settings**, select **Configuration**.</span></span>

    ![De optie gateway configuratie in de lokale netwerk gateway van Azure Stack hub](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="76cfb-238">Voer in **adres ruimte**het punt-naar-site-adres bereik in voor de gateway van het virtuele netwerk in Azure.</span><span class="sxs-lookup"><span data-stu-id="76cfb-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Punt-naar-site-adres ruimte in de lokale netwerk gateway van Azure Stack hub](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="76cfb-240">Selecteer **Opslaan** om de configuratie te valideren en op te slaan.</span><span class="sxs-lookup"><span data-stu-id="76cfb-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="76cfb-241">DNS configureren voor cross-Cloud schaling</span><span class="sxs-lookup"><span data-stu-id="76cfb-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="76cfb-242">Door DNS correct te configureren voor cross-Cloud-apps, hebben gebruikers toegang tot de algemene Azure-en Azure Stack hub-instanties van uw web-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="76cfb-243">De DNS-configuratie voor deze zelf studie zorgt er ook voor dat Azure Traffic Manager verkeer stuurt wanneer de belasting toeneemt of afneemt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="76cfb-244">In deze zelf studie wordt gebruikgemaakt van Azure DNS om de DNS te beheren, omdat App Service domeinen niet werken.</span><span class="sxs-lookup"><span data-stu-id="76cfb-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="76cfb-245">Subdomeinen maken</span><span class="sxs-lookup"><span data-stu-id="76cfb-245">Create subdomains</span></span>

<span data-ttu-id="76cfb-246">Omdat Traffic Manager afhankelijk is van DNS-CNAME, is er een subdomein nodig om verkeer naar eind punten goed te routeren.</span><span class="sxs-lookup"><span data-stu-id="76cfb-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="76cfb-247">Zie voor meer informatie over DNS-records en domein toewijzing [domeinen toewijzen met Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="76cfb-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="76cfb-248">Voor het Azure-eind punt maakt u een subdomein waarmee gebruikers toegang kunnen krijgen tot uw web-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="76cfb-249">Voor deze zelf studie kunt u **app.Northwind.com**gebruiken, maar u moet deze waarde aanpassen op basis van uw eigen domein.</span><span class="sxs-lookup"><span data-stu-id="76cfb-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="76cfb-250">U moet ook een subdomein met een record maken voor het eind punt van de Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="76cfb-251">U kunt **azurestack.Northwind.com**gebruiken.</span><span class="sxs-lookup"><span data-stu-id="76cfb-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="76cfb-252">Een aangepast domein configureren in azure</span><span class="sxs-lookup"><span data-stu-id="76cfb-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="76cfb-253">Voeg de **app.Northwind.com** hostname toe aan de Azure-web-app door [een CNAME toe te wijzen aan Azure app service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="76cfb-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="76cfb-254">Aangepaste domeinen configureren in Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="76cfb-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="76cfb-255">Voeg de **azurestack.Northwind.com** -hostnaam toe aan de Web-App van de Azure stack hub door een [A-record aan Azure app service toe te wijzen](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="76cfb-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="76cfb-256">Gebruik het IP-adres van Internet Routeer voor de App Service-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="76cfb-257">Voeg de **app.Northwind.com** -hostnaam toe aan de web-app Azure stack hub door [een CNAME toe te wijzen aan Azure app service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="76cfb-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="76cfb-258">Gebruik de hostnaam die u in de vorige stap (1) hebt geconfigureerd als doel voor de CNAME.</span><span class="sxs-lookup"><span data-stu-id="76cfb-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="76cfb-259">SSL-certificaten configureren voor cross-Cloud schalen</span><span class="sxs-lookup"><span data-stu-id="76cfb-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="76cfb-260">Het is belang rijk om ervoor te zorgen dat gevoelige gegevens die door uw web-app worden verzameld, worden beveiligd in door voer naar en wanneer deze worden opgeslagen op de SQL database.</span><span class="sxs-lookup"><span data-stu-id="76cfb-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="76cfb-261">U configureert uw Azure-en Azure Stack hub-web-apps voor het gebruik van SSL-certificaten voor al het binnenkomende verkeer.</span><span class="sxs-lookup"><span data-stu-id="76cfb-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="76cfb-262">SSL toevoegen aan Azure en Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="76cfb-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="76cfb-263">SSL toevoegen aan Azure:</span><span class="sxs-lookup"><span data-stu-id="76cfb-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="76cfb-264">Controleer of het SSL-certificaat dat u hebt ontvangen, geldig is voor het subdomein dat u hebt gemaakt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="76cfb-265">(U kunt Joker certificaten gebruiken.)</span><span class="sxs-lookup"><span data-stu-id="76cfb-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="76cfb-266">Volg in azure de instructies in de **Web-app voorbereiden** en **BIND uw SSL-certificaat** in de secties [een bestaand aangepast SSL-certificaat binden aan Azure web apps](/azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="76cfb-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="76cfb-267">Selecteer **SSL op basis van SNI** als het **SSL-type**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="76cfb-268">Alle verkeer omleiden naar de HTTPS-poort.</span><span class="sxs-lookup"><span data-stu-id="76cfb-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="76cfb-269">Volg de instructies in de sectie **https afdwingen** van het artikel [een bestaand aangepast SSL-certificaat binden aan Azure web apps](/azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="76cfb-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="76cfb-270">SSL toevoegen aan Azure Stack hub:</span><span class="sxs-lookup"><span data-stu-id="76cfb-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="76cfb-271">Herhaal de stappen 1-3 die u hebt gebruikt voor Azure.</span><span class="sxs-lookup"><span data-stu-id="76cfb-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="76cfb-272">De web-app configureren en implementeren</span><span class="sxs-lookup"><span data-stu-id="76cfb-272">Configure and deploy the web app</span></span>

<span data-ttu-id="76cfb-273">U configureert de app-code om telemetrie te rapporteren aan het juiste Application Insights-exemplaar en de web-apps te configureren met de juiste verbindings reeksen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="76cfb-274">Zie [Wat is Application Insights?](/azure/application-insights/app-insights-overview) voor meer informatie over Application Insights.</span><span class="sxs-lookup"><span data-stu-id="76cfb-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="76cfb-275">Application Insights toevoegen</span><span class="sxs-lookup"><span data-stu-id="76cfb-275">Add Application Insights</span></span>

1. <span data-ttu-id="76cfb-276">Open uw web-app in micro soft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="76cfb-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="76cfb-277">[Voeg Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) toe aan uw project voor het verzenden van de telemetrie die Application Insights gebruikt om waarschuwingen te maken wanneer webverkeer toeneemt of afneemt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="76cfb-278">Dynamische verbindings reeksen configureren</span><span class="sxs-lookup"><span data-stu-id="76cfb-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="76cfb-279">Voor elk exemplaar van de web-app wordt een andere methode gebruikt om verbinding te maken met de SQL database.</span><span class="sxs-lookup"><span data-stu-id="76cfb-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="76cfb-280">De app in azure maakt gebruik van het privé-IP-adres van de SQL Server virtuele machine en de app in Azure Stack hub gebruikt het open bare IP-adres van de SQL Server VM.</span><span class="sxs-lookup"><span data-stu-id="76cfb-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="76cfb-281">Op een Azure Stack hub geïntegreerd systeem mag het open bare IP-adres niet Internet routeerbaar zijn.</span><span class="sxs-lookup"><span data-stu-id="76cfb-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="76cfb-282">Op een ASDK kan het open bare IP-adres niet worden gerouteerd buiten de ASDK.</span><span class="sxs-lookup"><span data-stu-id="76cfb-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="76cfb-283">U kunt App Service omgevings variabelen gebruiken om een andere connection string door te geven aan elk exemplaar van de app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="76cfb-284">Open de app in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="76cfb-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="76cfb-285">Open Startup.cs en zoek het volgende code blok:</span><span class="sxs-lookup"><span data-stu-id="76cfb-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="76cfb-286">Vervang het vorige code blok door de volgende code, die gebruikmaakt van een connection string gedefinieerd in de *appsettings.jsvoor* het bestand:</span><span class="sxs-lookup"><span data-stu-id="76cfb-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="76cfb-287">App Service app-instellingen configureren</span><span class="sxs-lookup"><span data-stu-id="76cfb-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="76cfb-288">Maak verbindings reeksen voor Azure en Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="76cfb-289">De teken reeksen moeten hetzelfde zijn, met uitzonde ring van de IP-adressen die worden gebruikt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="76cfb-290">Voeg in Azure en Azure Stack hub de juiste connection string toe [als een app-instelling](/azure/app-service/web-sites-configure) in de web-app, met `SQLCONNSTR\_` als voor voegsel in de naam.</span><span class="sxs-lookup"><span data-stu-id="76cfb-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="76cfb-291">**Sla** de web-app-instellingen op en start de app opnieuw.</span><span class="sxs-lookup"><span data-stu-id="76cfb-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="76cfb-292">Automatisch schalen inschakelen in wereld wijd Azure</span><span class="sxs-lookup"><span data-stu-id="76cfb-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="76cfb-293">Wanneer u uw web-app in een App Service omgeving maakt, begint deze met één exemplaar.</span><span class="sxs-lookup"><span data-stu-id="76cfb-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="76cfb-294">U kunt automatisch uitschalen om instanties toe te voegen voor het leveren van meer reken resources voor uw app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="76cfb-295">Op dezelfde manier kunt u automatisch schalen en het aantal exemplaren beperken dat uw app nodig heeft.</span><span class="sxs-lookup"><span data-stu-id="76cfb-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="76cfb-296">U moet een App Service-abonnement hebben om uitschalen en schalen in te kunnen configureren.</span><span class="sxs-lookup"><span data-stu-id="76cfb-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="76cfb-297">Als u nog geen abonnement hebt, maakt u er een voordat u begint met de volgende stappen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="76cfb-298">Automatisch uitschalen inschakelen</span><span class="sxs-lookup"><span data-stu-id="76cfb-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="76cfb-299">Ga in azure naar het App Service plan voor de sites die u wilt uitschalen en selecteer vervolgens **uitschalen (app service plan)**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Uitschalen Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="76cfb-301">Selecteer **automatisch schalen inschakelen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-301">Select **Enable autoscale**.</span></span>

    ![Automatisch schalen inschakelen in Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="76cfb-303">Voer een naam in voor de **instellings naam voor automatisch schalen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="76cfb-304">Selecteer **schalen op basis van een metriek**voor de **standaard** regel voor automatisch schalen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="76cfb-305">Stel de **limieten** voor de instanties in op **minimum: 1**, **maximum: 10**en **standaard: 1**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Automatisch schalen configureren in Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="76cfb-307">Selecteer **+ een regel toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="76cfb-308">Selecteer **huidige resource**in **metrische bron**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="76cfb-309">Gebruik de volgende criteria en acties voor de regel.</span><span class="sxs-lookup"><span data-stu-id="76cfb-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="76cfb-310">Criteria</span><span class="sxs-lookup"><span data-stu-id="76cfb-310">Criteria</span></span>

1. <span data-ttu-id="76cfb-311">Onder **tijd aggregatie** selecteert u **gemiddelde**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="76cfb-312">Selecteer onder **metrische naam**de optie **CPU-percentage**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="76cfb-313">Onder **operator**selecteert u **groter dan**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="76cfb-314">Stel de **drempel waarde** in op **50**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="76cfb-315">Stel de **duur** in op **10**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="76cfb-316">Actie</span><span class="sxs-lookup"><span data-stu-id="76cfb-316">Action</span></span>

1. <span data-ttu-id="76cfb-317">Selecteer onder **bewerking**de optie **aantal verhogen per**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="76cfb-318">Stel het **aantal exemplaren** in op **2**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="76cfb-319">Stel de **afkoelen** in op **5**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="76cfb-320">Selecteer **Toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-320">Select **Add**.</span></span>

5. <span data-ttu-id="76cfb-321">Selecteer de **+ een regel toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="76cfb-322">Selecteer **huidige resource** in **metrische bron**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="76cfb-323">De huidige resource bevat de naam/GUID van uw App Service plan en het **resource type** en de vervolg keuzelijst **resource** zijn niet beschikbaar.</span><span class="sxs-lookup"><span data-stu-id="76cfb-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="76cfb-324">Automatisch schalen inschakelen in</span><span class="sxs-lookup"><span data-stu-id="76cfb-324">Enable automatic scale in</span></span>

<span data-ttu-id="76cfb-325">Wanneer het verkeer afneemt, kan de Azure-web-app automatisch het aantal actieve instanties verlagen om de kosten te verlagen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="76cfb-326">Deze actie is minder agressief dan uitschalen en minimaliseert de impact op app-gebruikers.</span><span class="sxs-lookup"><span data-stu-id="76cfb-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="76cfb-327">Ga naar de **standaard** waarde voor uitschalen en selecteer **+ een regel toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="76cfb-328">Gebruik de volgende criteria en acties voor de regel.</span><span class="sxs-lookup"><span data-stu-id="76cfb-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="76cfb-329">Criteria</span><span class="sxs-lookup"><span data-stu-id="76cfb-329">Criteria</span></span>

1. <span data-ttu-id="76cfb-330">Onder **tijd aggregatie** selecteert u **gemiddelde**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="76cfb-331">Selecteer onder **metrische naam**de optie **CPU-percentage**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="76cfb-332">Selecteer onder **operator**de optie **kleiner dan**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="76cfb-333">Stel de **drempel waarde** in op **30**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="76cfb-334">Stel de **duur** in op **10**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="76cfb-335">Actie</span><span class="sxs-lookup"><span data-stu-id="76cfb-335">Action</span></span>

1. <span data-ttu-id="76cfb-336">Selecteer onder **bewerking**de optie **aantal verlagen door**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="76cfb-337">Stel het **aantal exemplaren** in op **1**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="76cfb-338">Stel de **afkoelen** in op **5**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="76cfb-339">Selecteer **Toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="76cfb-340">Een Traffic Manager profiel maken en cross-Cloud schalen configureren</span><span class="sxs-lookup"><span data-stu-id="76cfb-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="76cfb-341">Maak een Traffic Manager profiel in Azure en configureer eind punten om cross-Cloud schalen in te scha kelen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="76cfb-342">Traffic Manager profiel maken</span><span class="sxs-lookup"><span data-stu-id="76cfb-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="76cfb-343">Selecteer **Een resource maken**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="76cfb-344">Selecteer **Netwerken**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-344">Select **Networking**.</span></span>
3. <span data-ttu-id="76cfb-345">Selecteer **Traffic Manager profiel** en configureer de volgende instellingen:</span><span class="sxs-lookup"><span data-stu-id="76cfb-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="76cfb-346">Voer bij **naam**een naam in voor uw profiel.</span><span class="sxs-lookup"><span data-stu-id="76cfb-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="76cfb-347">Deze naam **moet** uniek zijn in de zone trafficmanager.net en wordt gebruikt voor het maken van een nieuwe DNS-naam (bijvoorbeeld northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="76cfb-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="76cfb-348">Selecteer voor **routerings methode**de **gewogen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="76cfb-349">Selecteer bij **abonnement**het abonnement waarin u dit profiel wilt maken.</span><span class="sxs-lookup"><span data-stu-id="76cfb-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="76cfb-350">Maak in **resource groep**een nieuwe resource groep voor dit profiel.</span><span class="sxs-lookup"><span data-stu-id="76cfb-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="76cfb-351">In **Locatie van de resourcegroep** selecteert u de locatie van de resourcegroep.</span><span class="sxs-lookup"><span data-stu-id="76cfb-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="76cfb-352">Deze instelling verwijst naar de locatie van de resource groep en heeft geen invloed op het Traffic Manager profiel dat wereld wijd wordt geïmplementeerd.</span><span class="sxs-lookup"><span data-stu-id="76cfb-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="76cfb-353">Selecteer **Maken**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-353">Select **Create**.</span></span>

    ![Traffic Manager profiel maken](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="76cfb-355">Wanneer de globale implementatie van uw Traffic Manager profiel is voltooid, wordt dit weer gegeven in de lijst met resources voor de resource groep die u hebt gemaakt onder.</span><span class="sxs-lookup"><span data-stu-id="76cfb-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="76cfb-356">Traffic Manager-eindpunten toevoegen</span><span class="sxs-lookup"><span data-stu-id="76cfb-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="76cfb-357">Zoek het Traffic Manager profiel dat u hebt gemaakt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="76cfb-358">Als u naar de resource groep voor het profiel navigeert, selecteert u het profiel.</span><span class="sxs-lookup"><span data-stu-id="76cfb-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="76cfb-359">Selecteer in **Traffic Manager profiel**onder **instellingen**de optie **eind punten**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="76cfb-360">Selecteer **Toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-360">Select **Add**.</span></span>

4. <span data-ttu-id="76cfb-361">Gebruik in **eind punt toevoegen**de volgende instellingen voor Azure stack hub:</span><span class="sxs-lookup"><span data-stu-id="76cfb-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="76cfb-362">Selecteer bij **type** **externe eind punt**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="76cfb-363">Voer een **naam** in voor het eind punt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="76cfb-364">Voor **Fully Qualified Domain Name (FQDN) of IP**voert u de externe URL in voor uw Azure stack hub-web-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="76cfb-365">Voor **gewicht**, behoud de standaard waarde, **1**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="76cfb-366">Dit gewicht resulteert in al het verkeer naar dit eind punt als dit in orde is.</span><span class="sxs-lookup"><span data-stu-id="76cfb-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="76cfb-367">Schakel **toevoegen als uitgeschakeld** laten uit.</span><span class="sxs-lookup"><span data-stu-id="76cfb-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="76cfb-368">Selecteer **OK** om het Azure stack hub-eind punt op te slaan.</span><span class="sxs-lookup"><span data-stu-id="76cfb-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="76cfb-369">U configureert nu het Azure-eind punt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="76cfb-370">Selecteer **eind punten**op **Traffic Manager profiel**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="76cfb-371">Selecteer **+ toevoegen**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-371">Select **+Add**.</span></span>
3. <span data-ttu-id="76cfb-372">Gebruik op **eind punt toevoegen**de volgende instellingen voor Azure:</span><span class="sxs-lookup"><span data-stu-id="76cfb-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="76cfb-373">Selecteer voor **type** **Azure-eind punt**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="76cfb-374">Voer een **naam** in voor het eind punt.</span><span class="sxs-lookup"><span data-stu-id="76cfb-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="76cfb-375">Selecteer **app service**bij **doel resource type**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="76cfb-376">Selecteer bij **doel resource**een **app service kiezen** om een lijst van web apps in hetzelfde abonnement weer te geven.</span><span class="sxs-lookup"><span data-stu-id="76cfb-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="76cfb-377">Kies in **Resource** de app-service die u als eerste eindpunt wilt toevoegen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="76cfb-378">Selecteer voor **gewicht** **2**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="76cfb-379">Deze instelling resulteert in al het verkeer naar dit eind punt als het primaire eind punt een slechte status heeft of als u een regel/waarschuwing hebt die verkeer omleidt wanneer het wordt geactiveerd.</span><span class="sxs-lookup"><span data-stu-id="76cfb-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="76cfb-380">Schakel **toevoegen als uitgeschakeld** laten uit.</span><span class="sxs-lookup"><span data-stu-id="76cfb-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="76cfb-381">Selecteer **OK** om het Azure-eind punt op te slaan.</span><span class="sxs-lookup"><span data-stu-id="76cfb-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="76cfb-382">Nadat beide eind punten zijn geconfigureerd, worden ze weer gegeven in **Traffic Manager profiel** wanneer u **eind punten**selecteert.</span><span class="sxs-lookup"><span data-stu-id="76cfb-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="76cfb-383">In het voor beeld in de volgende scherm opname ziet u twee eind punten, met status-en configuratie gegevens voor elk van deze.</span><span class="sxs-lookup"><span data-stu-id="76cfb-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Eind punten in Traffic Manager profiel](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="76cfb-385">Application Insights bewaking en waarschuwingen instellen</span><span class="sxs-lookup"><span data-stu-id="76cfb-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="76cfb-386">Met Azure-toepassing Insights kunt u uw app bewaken en waarschuwingen verzenden op basis van de voor waarden die u configureert.</span><span class="sxs-lookup"><span data-stu-id="76cfb-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="76cfb-387">Enkele voor beelden zijn: de app is niet beschikbaar, er zijn fouten opgetreden of prestatie problemen worden weer gegeven.</span><span class="sxs-lookup"><span data-stu-id="76cfb-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="76cfb-388">U gebruikt Application Insights metrische gegevens om waarschuwingen te maken.</span><span class="sxs-lookup"><span data-stu-id="76cfb-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="76cfb-389">Wanneer deze waarschuwingen worden geactiveerd, wordt het exemplaar van uw web-app automatisch overgeschakeld van Azure Stack hub naar Azure om uit te schalen en vervolgens weer terug naar Azure Stack hub om in te schalen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="76cfb-390">Een waarschuwing maken vanuit metrische gegevens</span><span class="sxs-lookup"><span data-stu-id="76cfb-390">Create an alert from metrics</span></span>

<span data-ttu-id="76cfb-391">Ga naar de resource groep voor deze zelf studie en selecteer vervolgens het Application Insights exemplaar om **Application Insights**te openen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="76cfb-393">U gebruikt deze weer gave om een uitschaal waarschuwing te maken en een waarschuwing voor een schaal in te stellen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="76cfb-394">De uitschaal waarschuwing maken</span><span class="sxs-lookup"><span data-stu-id="76cfb-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="76cfb-395">Onder **configureren**selecteert u **waarschuwingen (klassiek)**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="76cfb-396">Selecteer **metrische waarschuwing toevoegen (klassiek)**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="76cfb-397">Configureer in **regel toevoegen**de volgende instellingen:</span><span class="sxs-lookup"><span data-stu-id="76cfb-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="76cfb-398">Voer bij **naam** **burst in azure Cloud in**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="76cfb-399">Een **Beschrijving** is optioneel.</span><span class="sxs-lookup"><span data-stu-id="76cfb-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="76cfb-400">Onder **bron**  >  **waarschuwing op**selecteert u **metrische gegevens**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="76cfb-401">Onder **criteria**selecteert u uw abonnement, de resource groep voor uw Traffic Manager profiel en de naam van het Traffic Manager profiel voor de resource.</span><span class="sxs-lookup"><span data-stu-id="76cfb-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="76cfb-402">Selecteer voor **metrische gegevens** **aanvraag frequentie**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="76cfb-403">Selecteer **Condition**voor voor waarde **groter dan**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="76cfb-404">Voer voor **drempel waarde** **2**in.</span><span class="sxs-lookup"><span data-stu-id="76cfb-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="76cfb-405">Voor **periode**selecteert u **de afgelopen 5 minuten**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="76cfb-406">Onder **melding via**:</span><span class="sxs-lookup"><span data-stu-id="76cfb-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="76cfb-407">Schakel het selectie vakje in voor **e-mail eigenaren, mede werkers en lezers**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="76cfb-408">Voer uw e-mail adres in voor **extra beheerders-e-mail (s)**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="76cfb-409">Selecteer **Opslaan**op de menu balk.</span><span class="sxs-lookup"><span data-stu-id="76cfb-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="76cfb-410">De waarschuwing voor de schaal van maken</span><span class="sxs-lookup"><span data-stu-id="76cfb-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="76cfb-411">Onder **configureren**selecteert u **waarschuwingen (klassiek)**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="76cfb-412">Selecteer **metrische waarschuwing toevoegen (klassiek)**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="76cfb-413">Configureer in **regel toevoegen**de volgende instellingen:</span><span class="sxs-lookup"><span data-stu-id="76cfb-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="76cfb-414">Voer bij **naam** **een inschaal in azure stack hub in**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="76cfb-415">Een **Beschrijving** is optioneel.</span><span class="sxs-lookup"><span data-stu-id="76cfb-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="76cfb-416">Onder **bron**  >  **waarschuwing op**selecteert u **metrische gegevens**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="76cfb-417">Onder **criteria**selecteert u uw abonnement, de resource groep voor uw Traffic Manager profiel en de naam van het Traffic Manager profiel voor de resource.</span><span class="sxs-lookup"><span data-stu-id="76cfb-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="76cfb-418">Selecteer voor **metrische gegevens** **aanvraag frequentie**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="76cfb-419">Selecteer **Condition**voor voor waarde **kleiner dan**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="76cfb-420">Voer voor **drempel waarde** **2**in.</span><span class="sxs-lookup"><span data-stu-id="76cfb-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="76cfb-421">Voor **periode**selecteert u **de afgelopen 5 minuten**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="76cfb-422">Onder **melding via**:</span><span class="sxs-lookup"><span data-stu-id="76cfb-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="76cfb-423">Schakel het selectie vakje in voor **e-mail eigenaren, mede werkers en lezers**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="76cfb-424">Voer uw e-mail adres in voor **extra beheerders-e-mail (s)**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="76cfb-425">Selecteer **Opslaan**op de menu balk.</span><span class="sxs-lookup"><span data-stu-id="76cfb-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="76cfb-426">De volgende scherm afbeelding toont de waarschuwingen voor uitschalen en schalen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights waarschuwingen (klassiek)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="76cfb-428">Verkeer tussen Azure en Azure Stack hub omleiden</span><span class="sxs-lookup"><span data-stu-id="76cfb-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="76cfb-429">U kunt hand matig of automatisch overschakelen van uw web-app-verkeer tussen Azure en Azure Stack hub configureren.</span><span class="sxs-lookup"><span data-stu-id="76cfb-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="76cfb-430">Hand matige scha kelen tussen Azure en Azure Stack hub configureren</span><span class="sxs-lookup"><span data-stu-id="76cfb-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="76cfb-431">Wanneer uw website de drempels bereikt die u configureert, ontvangt u een waarschuwing.</span><span class="sxs-lookup"><span data-stu-id="76cfb-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="76cfb-432">Gebruik de volgende stappen om het verkeer hand matig om te leiden naar Azure.</span><span class="sxs-lookup"><span data-stu-id="76cfb-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="76cfb-433">Selecteer uw Traffic Manager profiel in het Azure Portal.</span><span class="sxs-lookup"><span data-stu-id="76cfb-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Traffic Manager eind punten in Azure Portal](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="76cfb-435">Selecteer **eind punten**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="76cfb-436">Selecteer het **Azure-eind punt**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="76cfb-437">Selecteer onder **status**de optie **ingeschakeld**en selecteer vervolgens **Opslaan**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Azure-eind punt in Azure Portal inschakelen](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="76cfb-439">Selecteer op **eind punten** voor het profiel van de Traffic Manager **externe eind punt**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="76cfb-440">Selecteer onder **status**de optie **uitgeschakeld**en selecteer vervolgens **Opslaan**.</span><span class="sxs-lookup"><span data-stu-id="76cfb-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Azure Stack hub-eind punt in Azure Portal uitschakelen](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="76cfb-442">Nadat de eind punten zijn geconfigureerd, gaat het verkeer van de app naar uw Azure scale-out-web-app in plaats van de web-app Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="76cfb-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![De eind punten zijn gewijzigd in het verkeer van de Azure-web-app](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="76cfb-444">Als u de stroom weer wilt terugdraaien naar Azure Stack hub, gebruikt u de vorige stappen om:</span><span class="sxs-lookup"><span data-stu-id="76cfb-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="76cfb-445">Schakel het Azure Stack hub-eind punt in.</span><span class="sxs-lookup"><span data-stu-id="76cfb-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="76cfb-446">Het Azure-eind punt uitschakelen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="76cfb-447">Automatische switching tussen Azure en Azure Stack hub configureren</span><span class="sxs-lookup"><span data-stu-id="76cfb-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="76cfb-448">U kunt ook Application Insights bewaking gebruiken als uw app wordt uitgevoerd in een [serverloze](https://azure.microsoft.com/overview/serverless-computing/) omgeving van Azure functions.</span><span class="sxs-lookup"><span data-stu-id="76cfb-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="76cfb-449">In dit scenario kunt u Application Insights configureren voor het gebruik van een webhook waarmee een functie-app wordt aangeroepen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="76cfb-450">Met deze app kunt u een eind punt automatisch in-of uitschakelen als reactie op een waarschuwing.</span><span class="sxs-lookup"><span data-stu-id="76cfb-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="76cfb-451">Gebruik de volgende stappen als richt lijn voor het configureren van automatisch overschakelen van verkeer.</span><span class="sxs-lookup"><span data-stu-id="76cfb-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="76cfb-452">Maak een Azure-functie-app.</span><span class="sxs-lookup"><span data-stu-id="76cfb-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="76cfb-453">Een door HTTP geactiveerde functie maken.</span><span class="sxs-lookup"><span data-stu-id="76cfb-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="76cfb-454">Importeer de Azure-Sdk's voor Resource Manager, Web Apps en Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="76cfb-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="76cfb-455">Code ontwikkelen voor:</span><span class="sxs-lookup"><span data-stu-id="76cfb-455">Develop code to:</span></span>

   - <span data-ttu-id="76cfb-456">Verifieer uw Azure-abonnement.</span><span class="sxs-lookup"><span data-stu-id="76cfb-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="76cfb-457">Gebruik een para meter waarmee de Traffic Manager-eind punten worden gewisseld om verkeer naar Azure of Azure Stack hub om te leiden.</span><span class="sxs-lookup"><span data-stu-id="76cfb-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="76cfb-458">Sla de code op en voeg de URL van de functie-app met de juiste para meters toe aan de sectie **webhook** van de instellingen voor de Application Insights waarschuwings regel.</span><span class="sxs-lookup"><span data-stu-id="76cfb-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="76cfb-459">Verkeer wordt automatisch omgeleid wanneer een Application Insights waarschuwing wordt geactiveerd.</span><span class="sxs-lookup"><span data-stu-id="76cfb-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="76cfb-460">Volgende stappen</span><span class="sxs-lookup"><span data-stu-id="76cfb-460">Next steps</span></span>

- <span data-ttu-id="76cfb-461">Zie [Cloud ontwerp patronen](/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.</span><span class="sxs-lookup"><span data-stu-id="76cfb-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
