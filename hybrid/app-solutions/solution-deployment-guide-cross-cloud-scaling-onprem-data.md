---
title: Implementeer hybride app met on-premises gegevens die meerdere clouds schalen
description: Meer informatie over het implementeren van een app die gebruikmaakt van on-premises gegevens en het schalen van meerdere clouds met behulp van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ecc42a94e2c59531b2a2e933772b0d8ce8c58609
ms.sourcegitcommit: 0d5b5336bdb969588d0b92e04393e74b8f682c3b
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/22/2020
ms.locfileid: "92353475"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Implementeer hybride app met on-premises gegevens die meerdere clouds schalen

In deze oplossings handleiding ziet u hoe u een hybride app implementeert die zowel Azure als Azure Stack hub omvat en één on-premises gegevens bron gebruikt.

Met een hybride Cloud oplossing kunt u de nalevings voordelen van een privécloud combi neren met de schaal baarheid van de open bare Cloud. Uw ontwikkel aars kunnen ook profiteren van het micro soft Developer ecosysteem en hun vaardig heden Toep assen op de Cloud-en on-premises omgevingen.

## <a name="overview-and-assumptions"></a>Overzicht en hypo Thesen

Volg deze zelf studie om een werk stroom in te stellen waarmee ontwikkel aars een identieke Web-app kunnen implementeren in een open bare Cloud en een privécloud. Deze app kan toegang krijgen tot een niet-Internet routeerbaar netwerk dat wordt gehost op de privécloud. Deze web-apps worden bewaakt en wanneer er een piek in het verkeer is, worden de DNS-records gewijzigd om verkeer om te leiden naar de open bare Cloud. Wanneer het verkeer naar het niveau voor de Prikker daalt, wordt het verkeer teruggestuurd naar de privécloud.

Deze zelfstudie bestaat uit de volgende taken:

> [!div class="checklist"]
> - Implementeer een SQL Server-database server met hybride verbinding.
> - Een web-app in een globaal Azure verbinden met een hybride netwerk.
> - Configureer DNS voor het schalen van meerdere clouds.
> - Configureer SSL-certificaten voor het schalen van meerdere clouds.
> - De web-app configureren en implementeren.
> - Maak een Traffic Manager profiel en configureer dit voor cross-Cloud schaling.
> - Application Insights bewaking en waarschuwingen instellen voor verhoogd verkeer.
> - Configureer automatisch verkeer overschakelen tussen de wereld wijde Azure-en Azure Stack hub.

> [!Tip]  
> ![Diagram hybride pijlers](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

### <a name="assumptions"></a>Aannames

In deze zelf studie wordt ervan uitgegaan dat u een basis kennis hebt van wereld wijd Azure en Azure Stack hub. Als u meer informatie wilt over het starten van de zelf studie, raadpleegt u de volgende artikelen:

- [Inleiding tot Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Hoofd concepten van Azure Stack-hub](/azure-stack/operator/azure-stack-overview.md)

In deze zelf studie wordt ervan uitgegaan dat u een Azure-abonnement hebt. Als u nog geen abonnement hebt, [maakt u een gratis account](https://azure.microsoft.com/free/) voordat u begint.

## <a name="prerequisites"></a>Vereisten

Controleer voordat u met deze oplossing begint of u aan de volgende vereisten voldoet:

- Een Azure Stack Development Kit (ASDK) of een abonnement op een met Azure Stack hub geïntegreerd systeem. Als u de ASDK wilt implementeren, volgt u de instructies in [de ASDK implementeren met behulp van het installatie programma](/azure-stack/asdk/asdk-install.md).
- Voor de installatie van de Azure Stack hub moet het volgende zijn geïnstalleerd:
  - Het Azure App Service. Werk samen met uw Azure Stack hub-operator om de Azure App Service te implementeren en te configureren in uw omgeving. In deze zelf studie moet het App Service ten minste één (1) beschik bare toegewezen werk rollen hebben.
  - Een installatie kopie van Windows Server 2016.
  - Een Windows Server 2016 met een Microsoft SQL Server-installatie kopie.
  - De juiste plannen en aanbiedingen.
  - Een domein naam voor uw web-app. Als u geen domein naam hebt, kunt u er een kopen bij een domein provider zoals GoDaddy, bluehost en inbeweging.
- Een SSL-certificaat voor uw domein van een vertrouwde certificerings instantie, zoals LetsEncrypt.
- Een web-app die communiceert met een SQL Server-Data Base en die Application Insights ondersteunt. U kunt de voor beeld-app voor [dotnetcore-sqldb-zelf studie](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) downloaden van github.
- Een hybride netwerk tussen een virtueel Azure-netwerk en Azure Stack hub-netwerk. Zie [Configure Hybrid Cloud connectivity with Azure and Azure stack hub](solution-deployment-guide-connectivity.md)(Engelstalig) voor gedetailleerde instructies.

- Een ' CI/CD-pijp lijn met hybride integratie/continue implementatie ' met een Private build-agent op Azure Stack hub. Zie [Configure Hybrid Cloud identity with Azure and Azure stack hub apps](solution-deployment-guide-identity.md)(Engelstalig) voor gedetailleerde instructies.

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Een SQL Server database server met hybride verbinding implementeren

1. Meld u aan bij de gebruikers portal van de Azure Stack hub.

2. Selecteer **Marketplace**in het **dash board**.

    ![Azure Stack hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. Selecteer **Compute**in **Marketplace**en kies vervolgens **meer**. Selecteer onder **meer**de **gratis licentie voor SQL Server: SQL Server 2017 Developer op Windows Server** -installatie kopie.

    ![Een installatie kopie van een virtuele machine in Azure Stack hub-gebruikers Portal selecteren](media/solution-deployment-guide-hybrid/image2.png)

4. Selecteer in **gratis SQL Server licentie: SQL Server 2017 Developer op Windows Server**de optie **maken**.

5. Geef bij **basis beginselen > basis instellingen configureren**een **naam** op voor de virtuele machine (VM), een **gebruikers naam** voor de SQL Server SA en een **wacht woord** voor de SA.  Selecteer in de vervolg keuzelijst **abonnement** het abonnement dat u implementeert. Gebruik voor **resource groep**de **optie bestaande** en plaats de virtuele machine in dezelfde Resource groep als uw Azure stack hub-web-app.

    ![Basis instellingen configureren voor virtuele machines in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image3.png)

6. Kies onder **grootte**een grootte voor de virtuele machine. Voor deze zelf studie raden wij u aan A2_Standard of een DS2_V2_Standard.

7. Configureer onder **instellingen > optionele functies configureren**de volgende instellingen:

   - **Opslag account**: Maak een nieuw account als u er een hebt.
   - **Virtueel netwerk**:

     > [!Important]  
     > Zorg ervoor dat uw SQL Server-VM wordt geïmplementeerd op hetzelfde virtuele netwerk als de VPN-gateways.

   - **Openbaar IP-adres**: gebruik de standaard instellingen.
   - **Netwerk beveiligings groep**: (NSG). Maak een nieuwe NSG.
   - **Extensies en controle**: behoud de standaard instellingen.
   - **Opslag account voor diagnostische gegevens**: Maak een nieuw account als u er een hebt.
   - Selecteer **OK** om uw configuratie op te slaan.

     ![Optionele VM-functies configureren in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image4.png)

8. Configureer onder **SQL Server instellingen**de volgende instellingen:

   - Selecteer voor **SQL**-connectiviteit **openbaar (Internet)**.
   - Voor **poort**, behoud de standaard waarde van **1433**.
   - Selecteer **inschakelen**voor **SQL-verificatie**.

     > [!Note]  
     > Wanneer u SQL-verificatie inschakelt, moet deze automatisch worden ingevuld met de ' SQLAdmin-informatie die u hebt geconfigureerd in de **basis beginselen**.

   - Behoud de standaard waarden voor de overige instellingen. Selecteer **OK**.

     ![SQL Server-instellingen configureren in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image5.png)

9. Controleer bij **samen vatting**de configuratie van de virtuele machine en selecteer vervolgens **OK** om de implementatie te starten.

    ![Configuratie samenvatting in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image6.png)

10. Het duurt enige tijd om de nieuwe virtuele machine te maken. U kunt de STATUS van uw Vm's bekijken in **virtuele machines**.

    ![Status van virtuele machines in Azure Stack hub-gebruikers Portal](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Web-apps maken in Azure en Azure Stack hub

De Azure App Service vereenvoudigt het uitvoeren en beheren van een web-app. Omdat Azure Stack hub consistent is met Azure, kan de App Service in beide omgevingen worden uitgevoerd. U gebruikt de App Service om uw app te hosten.

### <a name="create-web-apps"></a>Web-apps maken

1. Maak een web-app in azure door de instructies te volgen in [een app service-abonnement beheren in azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan). Zorg ervoor dat u de web-app in hetzelfde abonnement en dezelfde resource groep plaatst als uw hybride netwerk.

2. Herhaal de vorige stap (1) in Azure Stack hub.

### <a name="add-route-for-azure-stack-hub"></a>Route voor Azure Stack hub toevoegen

De App Service op Azure Stack hub moet routeerbaar zijn van het open bare Internet om gebruikers toegang te geven tot uw app. Als uw Azure Stack hub toegankelijk is via internet, noteert u het open bare IP-adres of de URL voor de web-app Azure Stack hub.

Als u een ASDK gebruikt, kunt u [een statische NAT-toewijzing configureren](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) om app service buiten de virtuele omgeving zichtbaar te maken.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Een web-app in azure verbinden met een hybride netwerk

Als u de verbinding tussen de Webfront-end in Azure en de SQL Server-data base in Azure Stack hub wilt maken, moet u de web-app verbinden met het hybride netwerk tussen Azure en Azure Stack hub. Als u connectiviteit wilt inschakelen, moet u het volgende doen:

- Configureer punt-naar-site-connectiviteit.
- De web-app configureren.
- Wijzig de lokale netwerk gateway in Azure Stack hub.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Het virtuele Azure-netwerk configureren voor punt-naar-site-connectiviteit

De virtuele netwerk gateway aan de Azure-kant van het hybride netwerk moet punt-naar-site-verbindingen toestaan om te integreren met Azure App Service.

1. Ga in het Azure Portal naar de pagina virtuele netwerk gateway. Onder **instellingen**selecteert u **punt-naar-site-configuratie**.

    ![Punt-naar-site optie in azure Virtual Network-gateway](media/solution-deployment-guide-hybrid/image8.png)

2. Selecteer **nu configureren** om punt-naar-site te configureren.

    ![Begin punt-naar-site-configuratie in virtuele Azure-netwerk gateway](media/solution-deployment-guide-hybrid/image9.png)

3. Voer op de pagina **punt-naar-site** -configuratie het privé-IP-adres bereik in dat u wilt gebruiken in de **adres groep**.

   > [!Note]  
   > Zorg ervoor dat het bereik dat u opgeeft, niet overlapt met een van de adresbereiken die al worden gebruikt door subnetten in de globale onderdelen van Azure of Azure Stack hub van het hybride netwerk.

   Schakel onder **Tunnel Type**het selectie vakje **IKEv2 VPN**uit. Selecteer **Opslaan** om het configureren van punt-naar-site te volt ooien.

   ![Punt-naar-site-instellingen in virtuele Azure-netwerk gateway](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>De Azure App Service-app integreren met het hybride netwerk

1. Als u de app wilt verbinden met Azure VNet, volgt u de instructies in de [Gateway vereiste VNet-integratie](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).

2. Ga naar **instellingen** voor het app service plan dat als host fungeert voor de web-app. Selecteer in **instellingen**de optie **netwerken**.

    ![Netwerken configureren voor het App Service-abonnement](media/solution-deployment-guide-hybrid/image11.png)

3. Selecteer in **VNET-integratie** **Klik hier om te beheren**.

    ![VNET-integratie voor het App Service plan beheren](media/solution-deployment-guide-hybrid/image12.png)

4. Selecteer het VNET dat u wilt configureren. Voer onder **IP-adressen GEROUTEERD naar VNET**het IP-adres bereik in voor de Azure VNET, het Azure stack hub VNET en de punt-naar-site-adres ruimten. Selecteer **Opslaan** om deze instellingen te valideren en op te slaan.

    ![IP-adresbereiken voor route ring in Virtual Network integratie](media/solution-deployment-guide-hybrid/image13.png)

Zie [uw app integreren met een Azure-Virtual Network](/azure/app-service/web-sites-integrate-with-vnet)voor meer informatie over de manier waarop app service integreert met Azure VNets.

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Het virtuele netwerk van de Azure Stack hub configureren

De lokale netwerk gateway in het virtuele netwerk Azure Stack hub moet worden geconfigureerd om verkeer te routeren vanuit het App Service punt-naar-site-adres bereik.

1. Ga in de Azure Stack hub-Portal naar de **lokale netwerk gateway**. Selecteer onder **Instellingen** de optie **Configuratie**.

    ![De optie gateway configuratie in de lokale netwerk gateway van Azure Stack hub](media/solution-deployment-guide-hybrid/image14.png)

2. Voer in **adres ruimte**het punt-naar-site-adres bereik in voor de gateway van het virtuele netwerk in Azure.

    ![Punt-naar-site-adres ruimte in de lokale netwerk gateway van Azure Stack hub](media/solution-deployment-guide-hybrid/image15.png)

3. Selecteer **Opslaan** om de configuratie te valideren en op te slaan.

## <a name="configure-dns-for-cross-cloud-scaling"></a>DNS configureren voor cross-Cloud schaling

Door DNS correct te configureren voor cross-Cloud-apps, hebben gebruikers toegang tot de algemene Azure-en Azure Stack hub-instanties van uw web-app. De DNS-configuratie voor deze zelf studie zorgt er ook voor dat Azure Traffic Manager verkeer stuurt wanneer de belasting toeneemt of afneemt.

In deze zelf studie wordt gebruikgemaakt van Azure DNS om de DNS te beheren, omdat App Service domeinen niet werken.

### <a name="create-subdomains"></a>Subdomeinen maken

Omdat Traffic Manager afhankelijk is van DNS-CNAME, is er een subdomein nodig om verkeer naar eind punten goed te routeren. Zie voor meer informatie over DNS-records en domein toewijzing [domeinen toewijzen met Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).

Voor het Azure-eind punt maakt u een subdomein waarmee gebruikers toegang kunnen krijgen tot uw web-app. Voor deze zelf studie kunt u **app.Northwind.com**gebruiken, maar u moet deze waarde aanpassen op basis van uw eigen domein.

U moet ook een subdomein met een record maken voor het eind punt van de Azure Stack hub. U kunt **azurestack.Northwind.com**gebruiken.

### <a name="configure-a-custom-domain-in-azure"></a>Een aangepast domein configureren in azure

1. Voeg de **app.Northwind.com** hostname toe aan de Azure-web-app door [een CNAME toe te wijzen aan Azure app service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Aangepaste domeinen configureren in Azure Stack hub

1. Voeg de **azurestack.Northwind.com** -hostnaam toe aan de Web-App van de Azure stack hub door een [A-record aan Azure app service toe te wijzen](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Gebruik het IP-adres van Internet Routeer voor de App Service-app.

2. Voeg de **app.Northwind.com** -hostnaam toe aan de web-app Azure stack hub door [een CNAME toe te wijzen aan Azure app service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Gebruik de hostnaam die u in de vorige stap (1) hebt geconfigureerd als doel voor de CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>SSL-certificaten configureren voor cross-Cloud schalen

Het is belang rijk om ervoor te zorgen dat gevoelige gegevens die door uw web-app worden verzameld, worden beveiligd in door voer naar en wanneer deze worden opgeslagen op de SQL database.

U configureert uw Azure-en Azure Stack hub-web-apps voor het gebruik van SSL-certificaten voor al het binnenkomende verkeer.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>SSL toevoegen aan Azure en Azure Stack hub

SSL toevoegen aan Azure:

1. Controleer of het SSL-certificaat dat u hebt ontvangen, geldig is voor het subdomein dat u hebt gemaakt. (U kunt Joker certificaten gebruiken.)

2. Volg in de Azure Portal de instructies in de **Web-app voorbereiden** en **BIND uw SSL-certificaat** in de secties [een bestaand aangepast SSL-certificaat binden aan Azure web apps](/azure/app-service/app-service-web-tutorial-custom-ssl) . Selecteer **SSL op basis van SNI** als het **SSL-type**.

3. Alle verkeer omleiden naar de HTTPS-poort. Volg de instructies in de sectie   **https afdwingen** van het artikel [een bestaand aangepast SSL-certificaat binden aan Azure web apps](/azure/app-service/app-service-web-tutorial-custom-ssl) .

SSL toevoegen aan Azure Stack hub:

1. Herhaal de stappen 1-3 die u hebt gebruikt voor Azure met behulp van de Azure Stack hub-Portal.

## <a name="configure-and-deploy-the-web-app"></a>De web-app configureren en implementeren

U configureert de app-code om telemetrie te rapporteren aan het juiste Application Insights-exemplaar en de web-apps te configureren met de juiste verbindings reeksen. Zie [Wat is Application Insights?](/azure/application-insights/app-insights-overview) voor meer informatie over Application Insights.

### <a name="add-application-insights"></a>Application Insights toevoegen

1. Open uw web-app in micro soft Visual Studio.

2. [Voeg Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) toe aan uw project voor het verzenden van de telemetrie die Application Insights gebruikt om waarschuwingen te maken wanneer webverkeer toeneemt of afneemt.

### <a name="configure-dynamic-connection-strings"></a>Dynamische verbindings reeksen configureren

Voor elk exemplaar van de web-app wordt een andere methode gebruikt om verbinding te maken met de SQL database. De app in azure maakt gebruik van het privé-IP-adres van de SQL Server virtuele machine en de app in Azure Stack hub gebruikt het open bare IP-adres van de SQL Server VM.

> [!Note]  
> Op een Azure Stack hub geïntegreerd systeem mag het open bare IP-adres niet Internet routeerbaar zijn. Op een ASDK kan het open bare IP-adres niet worden gerouteerd buiten de ASDK.

U kunt App Service omgevings variabelen gebruiken om een andere connection string door te geven aan elk exemplaar van de app.

1. Open de app in Visual Studio.

2. Open Startup.cs en zoek het volgende code blok:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Vervang het vorige code blok door de volgende code, die gebruikmaakt van een connection string gedefinieerd in de *appsettings.jsvoor* het bestand:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>App Service app-instellingen configureren

1. Maak verbindings reeksen voor Azure en Azure Stack hub. De teken reeksen moeten hetzelfde zijn, met uitzonde ring van de IP-adressen die worden gebruikt.

2. Voeg in Azure en Azure Stack hub de juiste connection string toe [als een app-instelling](/azure/app-service/web-sites-configure) in de web-app, met `SQLCONNSTR\_` als voor voegsel in de naam.

3. **Sla** de web-app-instellingen op en start de app opnieuw.

## <a name="enable-automatic-scaling-in-global-azure"></a>Automatisch schalen inschakelen in wereld wijd Azure

Wanneer u uw web-app in een App Service omgeving maakt, begint deze met één exemplaar. U kunt automatisch uitschalen om instanties toe te voegen voor het leveren van meer reken resources voor uw app. Op dezelfde manier kunt u automatisch schalen en het aantal exemplaren beperken dat uw app nodig heeft.

> [!Note]  
> U moet een App Service-abonnement hebben om uitschalen en schalen in te kunnen configureren. Als u nog geen abonnement hebt, maakt u er een voordat u begint met de volgende stappen.

### <a name="enable-automatic-scale-out"></a>Automatisch uitschalen inschakelen

1. Zoek in de Azure Portal het App Service plan voor de sites die u wilt uitschalen en selecteer vervolgens **uitschalen (app service plan)**.

    ![Uitschalen Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. Selecteer **automatisch schalen inschakelen**.

    ![Automatisch schalen inschakelen in Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. Voer een naam in voor de **instellings naam voor automatisch schalen**. Selecteer **schalen op basis van een metriek**voor de **standaard** regel voor automatisch schalen. Stel de **limieten** voor de instanties in op **minimum: 1**, **maximum: 10**en **standaard: 1**.

    ![Automatisch schalen configureren in Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. Selecteer **+ een regel toevoegen**.

5. Selecteer **huidige resource**in **metrische bron**. Gebruik de volgende criteria en acties voor de regel.

#### <a name="criteria"></a>Criteria

1. Onder **tijd aggregatie** selecteert u **gemiddelde**.

2. Selecteer onder **metrische naam**de optie **CPU-percentage**.

3. Onder **operator**selecteert u **groter dan**.

   - Stel de **drempel waarde** in op **50**.
   - Stel de **duur** in op **10**.

#### <a name="action"></a>Bewerking

1. Selecteer onder **bewerking**de optie **aantal verhogen per**.

2. Stel het **aantal exemplaren** in op **2**.

3. Stel de **afkoelen** in op **5**.

4. Selecteer **Toevoegen**.

5. Selecteer de **+ een regel toevoegen**.

6. Selecteer **huidige resource** in **metrische bron**.

   > [!Note]  
   > De huidige resource bevat de naam/GUID van uw App Service plan en het **resource type** en de vervolg keuzelijst **resource** zijn niet beschikbaar.

### <a name="enable-automatic-scale-in"></a>Automatisch schalen inschakelen in

Wanneer het verkeer afneemt, kan de Azure-web-app automatisch het aantal actieve instanties verlagen om de kosten te verlagen. Deze actie is minder agressief dan uitschalen en minimaliseert de impact op app-gebruikers.

1. Ga naar de **standaard** waarde voor uitschalen en selecteer **+ een regel toevoegen**. Gebruik de volgende criteria en acties voor de regel.

#### <a name="criteria"></a>Criteria

1. Onder **tijd aggregatie** selecteert u **gemiddelde**.

2. Selecteer onder **metrische naam**de optie **CPU-percentage**.

3. Selecteer onder **operator**de optie **kleiner dan**.

   - Stel de **drempel waarde** in op **30**.
   - Stel de **duur** in op **10**.

#### <a name="action"></a>Bewerking

1. Selecteer onder **bewerking**de optie **aantal verlagen door**.

   - Stel het **aantal exemplaren** in op **1**.
   - Stel de **afkoelen** in op **5**.

2. Selecteer **Toevoegen**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Een Traffic Manager profiel maken en cross-Cloud schalen configureren

Maak een Traffic Manager profiel met behulp van de Azure Portal en configureer eind punten om cross-Cloud schalen in te scha kelen.

### <a name="create-traffic-manager-profile"></a>Traffic Manager profiel maken

1. Selecteer **Een resource maken**.
2. Selecteer **Netwerken**.
3. Selecteer **Traffic Manager profiel** en configureer de volgende instellingen:

   - Voer bij **naam**een naam in voor uw profiel. Deze naam **moet** uniek zijn in de zone trafficmanager.net en wordt gebruikt voor het maken van een nieuwe DNS-naam (bijvoorbeeld northwindstore.trafficmanager.net).
   - Selecteer voor **routerings methode**de **gewogen**.
   - Selecteer bij **abonnement**het abonnement waarin u dit profiel wilt maken.
   - Maak in **resource groep**een nieuwe resource groep voor dit profiel.
   - In **Locatie van de resourcegroep** selecteert u de locatie van de resourcegroep. Deze instelling verwijst naar de locatie van de resource groep en heeft geen invloed op het Traffic Manager profiel dat wereld wijd wordt geïmplementeerd.

4. Selecteer **Maken**.

    ![Traffic Manager profiel maken](media/solution-deployment-guide-hybrid/image19.png)

   Wanneer de globale implementatie van uw Traffic Manager profiel is voltooid, wordt dit weer gegeven in de lijst met resources voor de resource groep die u hebt gemaakt onder.

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager-eindpunten toevoegen

1. Zoek het Traffic Manager profiel dat u hebt gemaakt. Als u naar de resource groep voor het profiel navigeert, selecteert u het profiel.

2. Selecteer in **Traffic Manager profiel**onder **instellingen**de optie **eind punten**.

3. Selecteer **Toevoegen**.

4. Gebruik in **eind punt toevoegen**de volgende instellingen voor Azure stack hub:

   - Selecteer bij **type** **externe eind punt**.
   - Voer een **naam** in voor het eind punt.
   - Voor **Fully Qualified Domain Name (FQDN) of IP**voert u de externe URL in voor uw Azure stack hub-web-app.
   - Voor **gewicht**, behoud de standaard waarde, **1**. Dit gewicht resulteert in al het verkeer naar dit eind punt als dit in orde is.
   - Schakel **toevoegen als uitgeschakeld** laten uit.

5. Selecteer **OK** om het Azure stack hub-eind punt op te slaan.

U configureert nu het Azure-eind punt.

1. Selecteer **eind punten**op **Traffic Manager profiel**.
2. Selecteer **+Toevoegen**.
3. Gebruik op **eind punt toevoegen**de volgende instellingen voor Azure:

   - Selecteer voor **type** **Azure-eind punt**.
   - Voer een **naam** in voor het eind punt.
   - Selecteer **app service**bij **doel resource type**.
   - Selecteer bij **doel resource**een **app service kiezen** om een lijst van web apps in hetzelfde abonnement weer te geven.
   - Kies in **Resource** de app-service die u als eerste eindpunt wilt toevoegen.
   - Selecteer voor **gewicht** **2**. Deze instelling resulteert in al het verkeer naar dit eind punt als het primaire eind punt een slechte status heeft of als u een regel/waarschuwing hebt die verkeer omleidt wanneer het wordt geactiveerd.
   - Schakel **toevoegen als uitgeschakeld** laten uit.

4. Selecteer **OK** om het Azure-eind punt op te slaan.

Nadat beide eind punten zijn geconfigureerd, worden ze weer gegeven in **Traffic Manager profiel** wanneer u **eind punten**selecteert. In het voor beeld in de volgende scherm opname ziet u twee eind punten, met status-en configuratie gegevens voor elk van deze.

![Eind punten in Traffic Manager profiel](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a>Application Insights bewaking en waarschuwingen in azure instellen

Met Azure-toepassing Insights kunt u uw app bewaken en waarschuwingen verzenden op basis van de voor waarden die u configureert. Enkele voor beelden zijn: de app is niet beschikbaar, er zijn fouten opgetreden of prestatie problemen worden weer gegeven.

U gebruikt Azure-toepassing Insights-metrische gegevens om waarschuwingen te maken. Wanneer deze waarschuwingen worden geactiveerd, wordt het exemplaar van uw web-app automatisch overgeschakeld van Azure Stack hub naar Azure om uit te schalen en vervolgens weer terug naar Azure Stack hub om in te schalen.

### <a name="create-an-alert-from-metrics"></a>Een waarschuwing maken vanuit metrische gegevens

In de Azure Portal gaat u naar de resource groep voor deze zelf studie en selecteert u de Application Insights instantie om **Application Insights**te openen.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

U gebruikt deze weer gave om een uitschaal waarschuwing te maken en een waarschuwing voor een schaal in te stellen.

### <a name="create-the-scale-out-alert"></a>De uitschaal waarschuwing maken

1. Onder **configureren**selecteert u **waarschuwingen (klassiek)**.
2. Selecteer **metrische waarschuwing toevoegen (klassiek)**.
3. Configureer in **regel toevoegen**de volgende instellingen:

   - Voer bij **naam** **burst in azure Cloud in**.
   - Een **Beschrijving** is optioneel.
   - Onder **bron**  >  **waarschuwing op**selecteert u **metrische gegevens**.
   - Onder **criteria**selecteert u uw abonnement, de resource groep voor uw Traffic Manager profiel en de naam van het Traffic Manager profiel voor de resource.

4. Selecteer voor **metrische gegevens** **aanvraag frequentie**.
5. Selecteer **Condition**voor voor waarde **groter dan**.
6. Voer voor **drempel waarde** **2**in.
7. Voor **periode**selecteert u **de afgelopen 5 minuten**.
8. Onder **melding via**:
   - Schakel het selectie vakje in voor **e-mail eigenaren, mede werkers en lezers**.
   - Voer uw e-mail adres in voor **extra beheerders-e-mail (s)**.

9. Selecteer **Opslaan**op de menu balk.

### <a name="create-the-scale-in-alert"></a>De waarschuwing voor de schaal van maken

1. Onder **configureren**selecteert u **waarschuwingen (klassiek)**.
2. Selecteer **metrische waarschuwing toevoegen (klassiek)**.
3. Configureer in **regel toevoegen**de volgende instellingen:

   - Voer bij **naam** **een inschaal in azure stack hub in**.
   - Een **Beschrijving** is optioneel.
   - Onder **bron**  >  **waarschuwing op**selecteert u **metrische gegevens**.
   - Onder **criteria**selecteert u uw abonnement, de resource groep voor uw Traffic Manager profiel en de naam van het Traffic Manager profiel voor de resource.

4. Selecteer voor **metrische gegevens** **aanvraag frequentie**.
5. Selecteer **Condition**voor voor waarde **kleiner dan**.
6. Voer voor **drempel waarde** **2**in.
7. Voor **periode**selecteert u **de afgelopen 5 minuten**.
8. Onder **melding via**:
   - Schakel het selectie vakje in voor **e-mail eigenaren, mede werkers en lezers**.
   - Voer uw e-mail adres in voor **extra beheerders-e-mail (s)**.

9. Selecteer **Opslaan**op de menu balk.

De volgende scherm afbeelding toont de waarschuwingen voor uitschalen en schalen.

   ![Application Insights waarschuwingen (klassiek)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Verkeer tussen Azure en Azure Stack hub omleiden

U kunt hand matig of automatisch overschakelen van uw web-app-verkeer tussen Azure en Azure Stack hub configureren.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Hand matige scha kelen tussen Azure en Azure Stack hub configureren

Wanneer uw website de drempels bereikt die u configureert, ontvangt u een waarschuwing. Gebruik de volgende stappen om het verkeer hand matig om te leiden naar Azure.

1. Selecteer uw Traffic Manager profiel in het Azure Portal.

    ![Traffic Manager eind punten in Azure Portal](media/solution-deployment-guide-hybrid/image20.png)

2. Selecteer **eind punten**.
3. Selecteer het **Azure-eind punt**.
4. Selecteer onder **status**de optie **ingeschakeld**en selecteer vervolgens **Opslaan**.

    ![Azure-eind punt in Azure Portal inschakelen](media/solution-deployment-guide-hybrid/image23.png)

5. Selecteer op **eind punten** voor het profiel van de Traffic Manager **externe eind punt**.
6. Selecteer onder **status**de optie **uitgeschakeld**en selecteer vervolgens **Opslaan**.

    ![Azure Stack hub-eind punt in Azure Portal uitschakelen](media/solution-deployment-guide-hybrid/image24.png)

Nadat de eind punten zijn geconfigureerd, gaat het verkeer van de app naar uw Azure scale-out-web-app in plaats van de web-app Azure Stack hub.

 ![De eind punten zijn gewijzigd in het verkeer van de Azure-web-app](media/solution-deployment-guide-hybrid/image25.png)

Als u de stroom weer wilt terugdraaien naar Azure Stack hub, gebruikt u de vorige stappen om:

- Schakel het Azure Stack hub-eind punt in.
- Het Azure-eind punt uitschakelen.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Automatische switching tussen Azure en Azure Stack hub configureren

U kunt ook Application Insights bewaking gebruiken als uw app wordt uitgevoerd in een [serverloze](https://azure.microsoft.com/overview/serverless-computing/) omgeving van Azure functions.

In dit scenario kunt u Application Insights configureren voor het gebruik van een webhook waarmee een functie-app wordt aangeroepen. Met deze app kunt u een eind punt automatisch in-of uitschakelen als reactie op een waarschuwing.

Gebruik de volgende stappen als richt lijn voor het configureren van automatisch overschakelen van verkeer.

1. Maak een Azure-functie-app.
2. Een door HTTP geactiveerde functie maken.
3. Importeer de Azure-Sdk's voor Resource Manager, Web Apps en Traffic Manager.
4. Code ontwikkelen voor:

   - Verifieer uw Azure-abonnement.
   - Gebruik een para meter waarmee de Traffic Manager-eind punten worden gewisseld om verkeer naar Azure of Azure Stack hub om te leiden.

5. Sla de code op en voeg de URL van de functie-app met de juiste para meters toe aan de sectie **webhook** van de instellingen voor de Application Insights waarschuwings regel.
6. Verkeer wordt automatisch omgeleid wanneer een Application Insights waarschuwing wordt geactiveerd.

## <a name="next-steps"></a>Volgende stappen

- Zie [Cloud ontwerp patronen](/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.
