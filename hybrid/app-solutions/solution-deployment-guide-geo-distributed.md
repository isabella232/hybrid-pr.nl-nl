---
title: Direct verkeer met een geografisch gedistribueerde app met behulp van Azure en Azure Stack hub
description: Meer informatie over het omleiden van verkeer naar specifieke eind punten met een geografisch gedistribueerde app-oplossing met behulp van Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910523"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Direct verkeer met een geografisch gedistribueerde app met behulp van Azure en Azure Stack hub

Meer informatie over het omleiden van verkeer naar specifieke eind punten op basis van verschillende metrische gegevens met behulp van het patroon geo-gedistribueerde apps. Het maken van een Traffic Manager profiel met op geografische wijze gebaseerde route ring en eindpunt configuratie zorgt ervoor dat informatie wordt doorgestuurd naar eind punten op basis van regionale vereisten, zakelijke en internationale regelgeving en uw gegevens behoeften.

In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:

> [!div class="checklist"]
> - Een geografisch gedistribueerde app maken.
> - Gebruik Traffic Manager om uw app te richten.

## <a name="use-the-geo-distributed-apps-pattern"></a>Het patroon voor geo-gedistribueerde apps gebruiken

Met het geo-gedistribueerde patroon omvat uw app regio's. U kunt de standaard instelling voor de open bare Cloud, maar sommige gebruikers kunnen vereisen dat hun gegevens in hun regio blijven. U kunt gebruikers naar de meest geschikte Cloud sturen op basis van hun vereisten.

### <a name="issues-and-considerations"></a>Problemen en overwegingen

#### <a name="scalability-considerations"></a>Schaalbaarheidsoverwegingen

De oplossing die u met dit artikel bouwt, is niet geschikt voor schaal baarheid. Als u echter gebruikt in combi natie met andere Azure-en on-premises oplossingen, kunt u aan de schaalbaarheids vereisten voldoen. Zie voor meer informatie over het maken van een hybride oplossing met automatisch schalen via Traffic Manager [oplossingen voor cross-Cloud schalen met Azure](solution-deployment-guide-cross-cloud-scaling.md).

#### <a name="availability-considerations"></a>Beschikbaarheidsoverwegingen

Net als bij schaal baarheids overwegingen wordt deze oplossing niet rechtstreeks op de hoogte van de beschik baarheid. Azure-en on-premises oplossingen kunnen echter in deze oplossing worden geïmplementeerd om hoge Beschik baarheid voor alle betrokken onderdelen te garanderen.

### <a name="when-to-use-this-pattern"></a>Wanneer dit patroon gebruiken

- Uw organisatie heeft internationale vertakkingen waarvoor aangepaste regionale beveiligings-en distributie beleidsregels zijn vereist.

- Alle kant oren van uw organisatie halen werk nemer-, bedrijfs-en faciliteit gegevens, waarvoor rapportage activiteiten per lokale regelgeving en tijd zones vereist zijn.

- Aan hoge schaal vereisten wordt voldaan door apps horizon taal te schalen met meerdere app-implementaties binnen één regio en in verschillende regio's om extreem belasting vereisten af te handelen.

### <a name="planning-the-topology"></a>De topologie plannen

Voordat u een gedistribueerde app-footprint maakt, is het handig om de volgende zaken te kennen:

- **Aangepast domein voor de app:** Wat is de aangepaste domein naam die klanten gebruiken om toegang te krijgen tot de app? Voor de voor beeld-app is de aangepaste domein naam *www- \. scalableasedemo.com.*

- **Traffic Manager domein:** Er wordt een domein naam gekozen bij het maken van een [Azure Traffic Manager-profiel](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles). Deze naam wordt gecombineerd met het *trafficmanager.net* -achtervoegsel voor het registreren van een domein vermelding die wordt beheerd door Traffic Manager. Voor de voor beeld-app is de gekozen naam *schaalbaar-ASE-demo*. Als gevolg hiervan is de volledige domein naam die wordt beheerd door Traffic Manager, *Scalable-ASE-demo.trafficmanager.net*.

- **Strategie voor het schalen van de app-footprint:** Bepaal of het gebruik van de app wordt gedistribueerd over meerdere App Service omgevingen in één regio, meerdere regio's of een combi natie van beide benaderingen. De beslissing moet worden gebaseerd op de verwachtingen van waar klant verkeer van oorsprong is en hoe goed de rest van de ondersteunende back-end-infra structuur van een app kan worden geschaald. Een app kan bijvoorbeeld met een staatloze app van 100% worden geschaald met behulp van een combi natie van meerdere App Service omgevingen per Azure-regio, vermenigvuldigd met App Service omgevingen die zijn geïmplementeerd in meerdere Azure-regio's. Met 15 + wereld wijde Azure-regio's die beschikbaar zijn voor keuze, kunnen klanten echt een hele wereld voor Hyper-Scale-apps bouwen. Voor de voor beeld-app die hier wordt gebruikt, zijn er drie App Service omgevingen gemaakt in één Azure-regio (Zuid-Centraal).

- **Naam Conventie voor de app service omgevingen:** Elke App Service omgeving moet een unieke naam hebben. Naast een of twee App Service omgevingen is het handig om een naam Conventie te hebben om elke App Service omgeving te identificeren. Voor de voor beeld-app die hier wordt gebruikt, is een eenvoudige naam conventie gebruikt. De namen van de drie App Service omgevingen zijn *fe1ase*, *fe2ase*en *fe3ase*.

- **Naam Conventie voor de apps:** Omdat er meerdere exemplaren van de app worden geïmplementeerd, is een naam vereist voor elk exemplaar van de geïmplementeerde app. Met App Service Environment voor Power apps kan dezelfde app-naam worden gebruikt in meerdere omgevingen. Omdat elke App Service omgeving een uniek domein achtervoegsel heeft, kunnen ontwikkel aars ervoor kiezen om de exacte dezelfde app-naam in elke omgeving te gebruiken. Een ontwikkelaar kan bijvoorbeeld apps als volgt hebben met de naam: *MyApp.foo1.p.azurewebsites.net*, *MyApp.foo2.p.azurewebsites.net*, *MyApp.foo3.p.azurewebsites.net*, enzovoort. Voor de app die hier wordt gebruikt, heeft elk exemplaar van de app een unieke naam. De namen van de app-exemplaren die worden gebruikt, zijn *webfrontend1*, *webfrontend2*en *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

## <a name="part-1-create-a-geo-distributed-app"></a>Deel 1: een geografisch gedistribueerde app maken

In dit gedeelte maakt u een web-app.

> [!div class="checklist"]
> - Web-apps maken en publiceren.
> - Voeg code toe aan Azure opslag plaatsen.
> - Stel in dat de app op meerdere Cloud doelen is gebouwd.
> - Het CD-proces beheren en configureren.

### <a name="prerequisites"></a>Vereisten

Een Azure-abonnement en een installatie van Azure Stack hub zijn vereist.

### <a name="geo-distributed-app-steps"></a>Stappen voor geografisch gedistribueerde apps

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Een aangepast domein verkrijgen en DNS configureren

Werk het DNS-zone bestand voor het domein bij. Azure AD kan vervolgens het eigendom van de aangepaste domein naam verifiëren. Gebruik [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) voor Azure/Office 365/externe DNS-records in azure, of Voeg de DNS-vermelding toe aan [een ander DNS-REGI ster](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Een aangepast domein registreren bij een openbaar registratie service.

2. Meld u aan bij de domeinnaamregistrar voor het domein. Een goedgekeurde beheerder kan verplicht zijn de DNS-updates uit te voeren.

3. Werk het DNS-zone bestand voor het domein bij door de DNS-vermelding toe te voegen die door Azure AD wordt geleverd. Het wijzigen van de DNS-vermelding heeft geen invloed op het gedrag zoals mail routering of webhosting.

### <a name="create-web-apps-and-publish"></a>Web-apps maken en publiceren

Stel hybride continue integratie/continue levering (CI/CD) in om de web-app te implementeren in Azure en Azure Stack hub, en automatisch push wijzigingen door te voeren naar beide Clouds.

> [!Note]  
> Azure Stack hub met de juiste installatie kopieën die zijn syndicated om te worden uitgevoerd (Windows Server en SQL) en App Service implementatie is vereist. Zie [vereisten voor het implementeren van app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)voor meer informatie.

#### <a name="add-code-to-azure-repos"></a>Code toevoegen aan Azure opslag plaatsen

1. Meld u aan bij Visual Studio met een **account met rechten** voor het maken van projecten op Azure opslag plaatsen.

    CI/CD kan worden toegepast op de code van de app en de infra structuur. Gebruik [Azure Resource Manager sjablonen](https://azure.microsoft.com/resources/templates/) voor zowel privé als gehoste Cloud ontwikkeling.

    ![Verbinding maken met een project in Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. **Kloon de opslag plaats** door de standaard web-app te maken en te openen.

    ![Opslag plaats klonen in Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Implementatie van web-app in beide clouds maken

1. Bewerk het bestand **webapplication. csproj** : selecteren `Runtimeidentifier` en toevoegen `win10-x64` . (Zie [zelf-opgenomen implementatie](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentatie.)

    ![Web-app-project bestand bewerken in Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. **Check de code in op Azure opslag plaatsen** met behulp van team Explorer.

3. Controleer of de **toepassings code** is ingecheckt in azure opslag plaatsen.

### <a name="create-the-build-definition"></a>De build-definitie maken

1. **Meld u aan bij Azure-pijp lijnen** om te bevestigen dat u de bouw definities wilt maken.

2. Voeg `-r win10-x64` code toe. Deze toevoeging is nood zakelijk om een zelfstandige implementatie met .NET core te activeren.

    ![Code toevoegen aan de build-definitie in azure-pijp lijnen](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Voer de build uit**. In het [zelf opgenomen implementatie constructie](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) proces worden artefacten gepubliceerd die kunnen worden uitgevoerd op Azure en Azure stack hub.

#### <a name="using-an-azure-hosted-agent"></a>Een door Azure gehoste agent gebruiken

Het gebruik van een gehoste agent in azure-pijp lijnen is een handige optie om web-apps te bouwen en te implementeren. Onderhoud en upgrades worden automatisch uitgevoerd door Microsoft Azure, waardoor het mogelijk is om de ontwikkeling, het testen en de implementatie te onderbreken.

### <a name="manage-and-configure-the-cd-process"></a>Het CD-proces beheren en configureren

Azure DevOps Services voorziet in een zeer Configureer bare en beheersbare pijp lijn voor releases in meerdere omgevingen, zoals ontwikkelings-, staging-, QA-en productie omgevingen; met inbegrip van het vereisen van goed keuringen in specifieke fasen.

## <a name="create-release-definition"></a>Release definitie maken

1. Selecteer de knop met het **plus teken** om een nieuwe release toe te voegen op het tabblad **releases** in de sectie **Build and release** van Azure DevOps Services.

    ![Een release definitie maken in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. Pas de Azure App Service-implementatie sjabloon toe.

   ![Azure App Service implementatie sjabloon Toep assen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. Voeg onder **artefact toevoegen**het artefact toe voor de Azure Cloud build-app.

   ![Artefact toevoegen aan Azure Cloud build in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. Selecteer onder pijplijn tabblad de **fase, taak** koppeling van de omgeving en stel de waarden van de Azure-cloud omgeving in.

   ![Waarden van de Azure-cloud omgeving instellen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. Stel de **omgevings naam** in en selecteer het **Azure-abonnement** voor het Azure-Cloud eindpunt.

      ![Selecteer Azure-abonnement voor Azure Cloud-eind punt in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. Stel onder **app service name**de vereiste Azure app service-naam in.

      ![Azure app service-naam instellen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. Voer ' gehoste VS2017 ' in de wachtrij van de **agent** in voor de gehoste Azure cloud-omgeving.

      ![Agent wachtrij instellen voor gehoste Azure-cloud omgeving in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. Selecteer in het menu implementeren Azure App Service het geldige **pakket of** de juiste map voor de omgeving. Selecteer **OK** om **de maplocatie te**selecteren.
  
      ![Selecteer pakket of map voor Azure App Service omgeving in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selecteer pakket of map voor Azure App Service omgeving in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. Sla alle wijzigingen op en ga terug naar de **release pijplijn**.

    ![Wijzigingen in de release pijplijn in azure DevOps services opslaan](media/solution-deployment-guide-geo-distributed/image14.png)

10. Voeg een nieuw artefact toe om de build voor de Azure Stack hub-app te selecteren.

    ![Nieuwe artefacten voor Azure Stack hub-app toevoegen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. Voeg nog een omgeving toe door de implementatie van Azure App Service toe te passen.

    ![Omgeving toevoegen aan Azure App Service-implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. Geef de nieuwe omgeving een naam Azure Stack hub.

    ![Naam omgeving in Azure App Service-implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. Zoek de Azure Stack hub-omgeving onder **taak** tabblad.

    ![Azure Stack hub-omgeving in azure DevOps Services in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. Selecteer het abonnement voor het eind punt van de Azure Stack hub.

    ![Selecteer het abonnement voor het Azure Stack hub-eind punt in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. Stel de naam van de Azure Stack hub-web-app in als de naam van de app service.

    ![Azure Stack hub-web-app-naam instellen in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. Selecteer de Azure Stack hub-agent.

    ![Selecteer de Azure Stack hub-agent in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. Selecteer onder de sectie implementatie Azure App Service het geldige **pakket of** de juiste map voor de omgeving. Selecteer **OK** om de maplocatie te selecteren.

    ![Selecteer een map voor Azure App Service implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selecteer een map voor Azure App Service implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. Onder op het tabblad variabele voegt u een variabele met de naam in `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , stelt u de waarde in op **True**en bereik Azure stack hub.

    ![Een variabele toevoegen aan Azure-app-implementatie in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. Selecteer het trigger pictogram **continue** implementatie in beide artefacten en schakel de trigger **voor het implementeren** van de implementatie in.

    ![Trigger voor continue implementatie selecteren in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. Selecteer het pictogram voor waarden **voorafgaand** aan de implementatie in de Azure stack hub-omgeving en stel de trigger in op **na release.**

    ![Voor waarden voor voor bereiding selecteren in azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. Sla alle wijzigingen op.

> [!Note]  
> Sommige instellingen voor de taken zijn mogelijk automatisch gedefinieerd als [omgevings variabelen](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) bij het maken van een release definitie op basis van een sjabloon. Deze instellingen kunnen niet worden gewijzigd in de taak instellingen. in plaats daarvan moet het bovenliggende omgevings item worden geselecteerd om deze instellingen te bewerken.

## <a name="part-2-update-web-app-options"></a>Deel 2: opties voor Web-Apps bijwerken

[Azure App Service](https://docs.microsoft.com/azure/app-service/overview) biedt een uiterst schaalbare webhostingservice met self-patchfunctie.

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Een bestaande aangepaste DNS-naam toewijzen aan Azure Web Apps.
> - Gebruik een **CNAME-record** en een **a-record** om een aangepaste DNS-naam toe te wijzen aan app service.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Een bestaande aangepaste DNS-naam toewijzen aan Azure Web Apps

> [!Note]  
> Gebruik een CNAME voor alle aangepaste DNS-namen, met uitzonde ring van een hoofd domein (bijvoorbeeld northwind.com).

Zie voor het migreren van een live site en de DNS-domeinnaam naar App Service, [Een actieve DNS-naam migreren naar Azure App Service](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).

### <a name="prerequisites"></a>Vereisten

Voor het volt ooien van deze oplossing:

- [Maak een app service-app](https://docs.microsoft.com/azure/app-service/)of gebruik een app die is gemaakt voor een andere oplossing.

- Koop een domein naam en zorg ervoor dat u toegang tot het DNS-REGI ster voor de domein provider hebt.

Werk het DNS-zone bestand voor het domein bij. Azure AD controleert het eigendom van de aangepaste domein naam. Gebruik [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) voor Azure/Office 365/externe DNS-records in azure, of Voeg de DNS-vermelding toe aan [een ander DNS-REGI ster](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

- Een aangepast domein registreren bij een openbaar registratie service.

- Meld u aan bij de domeinnaamregistrar voor het domein. (Een goedgekeurde beheerder kan verplicht zijn om DNS-updates te maken.)

- Werk het DNS-zone bestand voor het domein bij door de DNS-vermelding toe te voegen die door Azure AD wordt geleverd.

Als u bijvoorbeeld DNS-vermeldingen wilt toevoegen voor northwindcloud.com en www \. northwindcloud.com, configureert u DNS-instellingen voor het hoofd domein northwindcloud.com.

> [!Note]  
> U kunt een domein naam aanschaffen met behulp van de [Azure Portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain). Om een aangepaste DNS-naam toe te wijzen aan een web-app, moet het [App Service-plan](https://azure.microsoft.com/pricing/details/app-service/) van de web-app een betaalde categorie zijn (**Shared**, **Basic**, **Standard** of ** Premium**).

### <a name="create-and-map-cname-and-a-records"></a>CNAME-en A-records maken en toewijzen

#### <a name="access-dns-records-with-domain-provider"></a>Toegang tot DNS-records via domeinprovider

> [!Note]  
>  Gebruik Azure DNS om een aangepaste DNS-naam te configureren voor Azure Web Apps. Zie [Use Azure DNS to provide custom domain settings for an Azure service](https://docs.microsoft.com/azure/dns/dns-custom-domain) (Azure DNS gebruiken om aangepaste domeininstellingen te verstrekken voor een Azure-service) voor meer informatie.

1. Meld u aan bij de website van de hoofd provider.

2. Ga naar de pagina voor het beheren van DNS-records. Elke domein provider heeft zijn eigen DNS-record interface. Doorgaans heeft het sitegedeelte waar u moet zijn, een naam als **Domain Name**, **DNS** of **Name Server Management**.

De pagina DNS-records kan worden weer gegeven in **mijn domeinen**. Zoek de koppeling met de naam **zone bestand**, **DNS-records**of **Geavanceerde configuratie**.

In de schermafbeelding hieronder wordt een voorbeeld van een pagina met DNS-records weergegeven:

![Voorbeeld van een pagina met DNS-records](media/solution-deployment-guide-geo-distributed/image28.png)

1. Selecteer in domein naam registratie **toevoegen of maken** om een record te maken. Sommige providers hebben afzonderlijke links voor verschillende typen records. Raadpleeg de documentatie van de provider.

2. Voeg een CNAME-record toe om een subdomein toe te wijzen aan de standaard-hostnaam van de app.

   Voeg voor het \. voor beeld van het www northwindcloud.com-domein een CNAME-record toe die de naam toewijst aan `<app_name>.azurewebsites.net` .

Nadat u de CNAME hebt toegevoegd, ziet de pagina met DNS-records eruit als in het volgende voor beeld:

![Navigatie naar Azure-app in de portal](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>De toewijzing van het CNAME-record in Azure inschakelen

1. Meld u op een nieuw tabblad aan bij de Azure Portal.

2. Ga naar App Services.

3. Selecteer Web-app.

4. Selecteer in het linkernavigatievenster van de app-pagina in de Azure portal **Aangepaste domeinen**.

5. Selecteer het **+** pictogram naast **hostnaam toevoegen**.

6. Typ de Fully Qualified Domain Name, zoals `www.northwindcloud.com` .

7. Selecteer **Valideren**.

8. Indien aangegeven, voegt u aanvullende records van andere typen ( `A` of `TXT` ) toe aan de domein naam registratie-DNS-records. De waarden en typen van deze records worden door Azure verstrekt:

   a.  Een **A**-record toewijzen aan het IP-adres van de app.

   b.  Een **TXT**-record toewijzen aan de standaardhostnaam `<app_name>.azurewebsites.net` van de app. App Service gebruikt deze record alleen op configuratie tijd om het eigendom van het aangepaste domein te verifiëren. Na de verificatie verwijdert u de TXT-record.

9. Voltooi deze taak op het tabblad domein registratie en pas de validatie opnieuw uit tot de knop **hostnaam toevoegen** wordt geactiveerd.

10. Zorg ervoor dat het **hostnaam-record type** is ingesteld op **CNAME** (www.example.com of een wille keurig subdomein).

11. Selecteer **Hostnaam toevoegen**.

12. Typ de Fully Qualified Domain Name, zoals `northwindcloud.com` .

13. Selecteer **Valideren**. De **toevoegen** is geactiveerd.

14. Zorg ervoor dat het **hostnaam-record type** is ingesteld op **een record** (example.com).

15. **Hostnaam toevoegen**.

    Het kan enige tijd duren voordat de nieuwe hostnamen worden weer gegeven op de pagina **aangepaste domeinen** van de app. Vernieuw de browser voor om de gegevens bij te werken.
  
    ![Aangepaste domeinen](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Als er een fout optreedt, wordt aan de onderkant van de pagina een melding over een verificatie fout weer gegeven. ![Domein verificatie fout](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  De bovenstaande stappen kunnen worden herhaald om een Joker teken domein ( \* . northwindcloud.com) toe te wijzen. Hiermee kunnen extra subdomeinen worden toegevoegd aan deze app service zonder dat hiervoor een afzonderlijke CNAME-record hoeft te worden gemaakt. Volg de registratie-instructies voor het configureren van deze instelling.

#### <a name="test-in-a-browser"></a>Testen in een browser

Blader naar de DNS-naam (s) die u eerder hebt geconfigureerd (bijvoorbeeld `northwindcloud.com` of `www.northwindcloud.com` ).

## <a name="part-3-bind-a-custom-ssl-cert"></a>Deel 3: een aangepast SSL-certificaat binden

In dit gedeelte gaan we het volgende doen:

> [!div class="checklist"]
> - Koppel het aangepaste SSL-certificaat aan App Service.
> - HTTPS afdwingen voor de app.
> - SSL-certificaat binding met scripts automatiseren.

> [!Note]  
> Als dat nodig is, kunt u een SSL-certificaat van de klant verkrijgen in de Azure Portal en het binden aan de web-app. Zie de [zelf studie over app service-certificaten](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site)voor meer informatie.

### <a name="prerequisites"></a>Vereisten

Voor het volt ooien van deze oplossing:

- [Een App Service-app maken.](https://docs.microsoft.com/azure/app-service/)
- [Wijs een aangepaste DNS-naam toe aan uw web-app.](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- Haal een SSL-certificaat van een vertrouwde certificerings instantie op en gebruik de sleutel om de aanvraag te ondertekenen.

### <a name="requirements-for-your-ssl-certificate"></a>Vereisten voor uw SSL-certificaat

Als u een certificaat in App Service wilt gebruiken, moet het certificaat aan de volgende vereisten voldoen:

- Ondertekend door een vertrouwde certificerings instantie.

- Geëxporteerd als een PFX-bestand dat met een wacht woord is beveiligd.

- Bevat een persoonlijke sleutel van ten minste 2048 bits lang.

- Bevat alle tussenliggende certificaten in de certificaat keten.

> [!Note]  
> De **ECC-algoritmen** voor het uitvoeren van een elliptische curve werken met app service, maar zijn niet opgenomen in deze hand leiding. Raadpleeg een certificerings instantie voor hulp bij het maken van ECC-certificaten.

#### <a name="prepare-the-web-app"></a>De web-app voorbereiden

Als u een aangepast SSL-certificaat aan de Web-App wilt koppelen, moet het [app service plan](https://azure.microsoft.com/pricing/details/app-service/) zich in de laag **Basic**, **Standard**of **Premium** bestaan.

#### <a name="sign-in-to-azure"></a>Aanmelden bij Azure

1. Open de [Azure Portal](https://portal.azure.com/) en ga naar de web-app.

2. Selecteer in het menu links **app Services**en selecteer vervolgens de naam van de web-app.

![Web-app in Azure Portal selecteren](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Controleer de prijscategorie

1. Ga in de linkernavigatiebalk van de pagina Web-app naar het gedeelte **instellingen** en selecteer **omhoog schalen (app service plan)**.

    ![Menu omhoog schalen in web-app](media/solution-deployment-guide-geo-distributed/image34.png)

1. Zorg ervoor dat de web-app zich niet in de laag **gratis** of **gedeeld** bevindt. De huidige laag van de web-app is gemarkeerd in een donker blauw vak.

    ![De prijs categorie in de web-app controleren](media/solution-deployment-guide-geo-distributed/image35.png)

Aangepaste SSL wordt niet ondersteund in de laag **gratis** of **gedeeld** . Volg de stappen in de volgende sectie of de pagina **uw prijs categorie kiezen** om [uw SSL-certificaat te uploaden en te binden](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Uw App Service-plan omhoog schalen

1. Selecteer de prijscategorie **Basic**, **Standard** of **Premium**.

2. Selecteer **Selecteren**.

![Prijs categorie kiezen voor uw web-app](media/solution-deployment-guide-geo-distributed/image36.png)

De schaal bewerking is voltooid wanneer de melding wordt weer gegeven.

![Melding voor omhoog schalen](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Uw SSL-certificaat binden en tussenliggende certificaten samen voegen

Meerdere certificaten in de keten samen voegen.

1. **Open elk certificaat** dat u hebt ontvangen in een tekst editor.

2. Maak een bestand voor het samengevoegde certificaat met de naam *mergedcertificate. CRT*. Kopieer de inhoud van elk certificaat in dit bestand in een teksteditor. De volgorde van uw certificaten moet de volgorde in de certificaatketen volgen, beginnend met uw certificaat en eindigend met het hoofdcertificaat. Het lijkt op het volgende voorbeeld:

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>Certificaat naar PFX exporteren

Het samengevoegde SSL-certificaat exporteren met de persoonlijke sleutel die door het certificaat is gegenereerd.

Er wordt een bestand met een persoonlijke sleutel gemaakt via OpenSSL. Als u het certificaat naar PFX wilt exporteren, voert u de volgende opdracht uit en vervangt u de tijdelijke aanduidingen `<private-key-file>` en het `<merged-certificate-file>` pad naar de persoonlijke sleutel en het samengevoegde certificaat bestand:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Wanneer u hierom wordt gevraagd, definieert u een export wachtwoord voor het uploaden van uw SSL-certificaat naar App Service later.

Wanneer IIS of **Certreq.exe** wordt gebruikt om de certificaat aanvraag te genereren, installeert u het certificaat op een lokale computer en [exporteert u het certificaat naar pfx](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

#### <a name="upload-the-ssl-certificate"></a>Het SSL-certificaat uploaden

1. Selecteer **SSL-instellingen** in het linkernavigatievenster van de web-app.

2. Selecteer **certificaat uploaden**.

3. Selecteer in **PFX-certificaat bestand**de optie pfx-bestand.

4. In **certificaat wachtwoord**typt u het wacht woord dat u hebt gemaakt bij het exporteren van het pfx-bestand.

5. Selecteer **Uploaden**.

    ![SSL-certificaat uploaden](media/solution-deployment-guide-geo-distributed/image38.png)

Als App Service klaar bent met het uploaden van het certificaat, wordt het weer gegeven op de pagina **SSL-instellingen** .

![SSL-instellingen](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>Uw SSL-certificaat binden

1. Selecteer in de sectie **SSL-bindingen** de optie **binding toevoegen**.

    > [!Note]  
    >  Als het certificaat is geüpload, maar niet wordt weer gegeven in domein namen in de vervolg keuzelijst **hostname** , probeert u de browser pagina te vernieuwen.

2. Op de pagina **SSL-binding toevoegen** gebruikt u de vervolg keuzelijst om de domein naam te selecteren die u wilt beveiligen en het certificaat dat u wilt gebruiken.

3. Selecteer in **SSL-type** of u [**Servernaamindicatie (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) wilt gebruiken of op IP gebaseerde SSL.

    - **Op SNI gebaseerd SSL**: er kunnen meerdere SSL-bindingen op basis van SNI worden toegevoegd. Met deze optie kunnen meerdere SSL-certificaten verschillende domeinen beveiligen op hetzelfde IP-adres. De meeste moderne browsers (waaronder Internet Explorer, Chrome, Firefox en Opera) ondersteunen SNI. Ga voor uitgebreidere informatie over browserondersteuning naar [Servernaamindicatie](https://wikipedia.org/wiki/Server_Name_Indication).

    - **SSL op basis van IP**: er kan slechts één SSL-binding op basis van IP worden toegevoegd. Met deze optie kan slechts één SSL-certificaat een specifiek openbaar IP-adres beveiligen. Als u meerdere domeinen wilt beveiligen, moet u ze allemaal beveiligen met hetzelfde SSL-certificaat. SSL is de traditionele optie voor SSL-binding.

4. Selecteer **binding toevoegen**.

    ![SSL-binding toevoegen](media/solution-deployment-guide-geo-distributed/image40.png)

Als App Service klaar bent met het uploaden van het certificaat, wordt het weer gegeven in de secties **SSL-bindingen** .

![Het uploaden van SSL-bindingen is voltooid](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>De A-record opnieuw toewijzen voor IP SSL

Als op IP gebaseerde SSL niet wordt gebruikt in de web-app, gaat u door naar [https testen voor uw aangepaste domein](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

De web-app maakt standaard gebruik van een gedeeld openbaar IP-adres. Wanneer het certificaat is gebonden aan SSL op basis van IP, App Service maakt een nieuw en toegewijd IP-adres voor de web-app.

Wanneer een A-record wordt toegewezen aan de web-app, moet het domein register worden bijgewerkt met het specifieke IP-adres.

De pagina **aangepast domein** wordt bijgewerkt met het nieuwe, toegewezen IP-adres. Kopieer dit [IP-adres](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)en wijs vervolgens de [A-record](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) opnieuw toe aan dit nieuwe IP-adres.

#### <a name="test-https"></a>HTTPS testen

Ga in verschillende browsers naar `https://<your.custom.domain>` om ervoor te zorgen dat de web-app wordt aangeboden.

![naar web-app bladeren](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Als er fouten in de certificaat validatie optreden, kan een zelfondertekend certificaat de oorzaak zijn of kunnen tussenliggende certificaten zijn uitgeschakeld bij het exporteren naar het PFX-bestand.

#### <a name="enforce-https"></a>HTTPS afdwingen

Standaard heeft iedereen toegang tot de web-app met behulp van HTTP. Alle HTTP-aanvragen voor de HTTPS-poort kunnen worden omgeleid.

Selecteer op de pagina Web-App de optie **SL-instellingen**. Klik op **Alleen HTTPS** en selecteer **Aan**.

![HTTPS afdwingen](media/solution-deployment-guide-geo-distributed/image43.png)

Wanneer de bewerking is voltooid, gaat u naar een van de HTTP-Url's die verwijzen naar de app. Bijvoorbeeld:

- https://<app_name>. azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>TLS 1.1/1.2 afdwingen

De app staat standaard [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0 toe, die niet meer wordt beschouwd als veilig door industriële normen (zoals [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)). Als u hogere TLS-versies wilt afdwingen, volgt u deze stappen:

1. Selecteer op de pagina Web-app in het linkernavigatievenster de optie **SSL-instellingen**.

2. Selecteer in **TLS-versie**de minimale TLS-versie.

    ![TLS 1.1 of 1.2 afdwingen](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Een Traffic Manager-profiel maken

1. Selecteer **een resource maken**  >  **netwerk**  >  **Traffic Manager profiel**  >  **maken**.

2. Vul het volgende in bij **Traffic Manager-profiel maken**:

    1. Geef bij **naam**een naam op voor het profiel. Deze naam moet uniek zijn binnen de zone verkeer manager.net en resulteert in de DNS-naam trafficmanager.net, die wordt gebruikt voor toegang tot het profiel van de Traffic Manager.

    2. Selecteer bij **routerings methode**de **geografische routerings methode**.

    3. Selecteer bij **abonnement**het abonnement waaronder u dit profiel wilt maken.

    4. In **Resourcegroep** maakt u een nieuwe resourcegroep om dit profiel voor te maken.

    5. In **Locatie van de resourcegroep** selecteert u de locatie van de resourcegroep. Deze instelling verwijst naar de locatie van de resource groep en heeft geen invloed op het Traffic Manager profiel wereld wijd geïmplementeerd.

    6. Selecteer **Maken**.

    7. Wanneer de globale implementatie van het Traffic Manager profiel is voltooid, wordt het weer gegeven in de betreffende resource groep als een van de resources.

        ![Resource groepen in Traffic Manager profiel maken](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager-eindpunten toevoegen

1. Zoek in de zoek balk van de portal naar de naam van het **Traffic Manager profiel** dat u in de voor gaande sectie hebt gemaakt en selecteer het Traffic Manager-profiel in de weer gegeven resultaten.

2. Selecteer in **Traffic Manager profiel**in de sectie **instellingen** de optie **eind punten**.

3. Selecteer **Toevoegen**.

4. Het Azure Stack hub-eind punt toevoegen.

5. Selecteer bij **type** **externe eind punt**.

6. Geef een **naam** op voor dit eind punt, in het ideale geval de naam van de Azure stack hub.

7. Gebruik voor Fully Qualified Domain Name (**FQDN**) de externe URL voor de web-app Azure stack hub.

8. Selecteer onder geo-toewijzing een regio/continent waar de resource zich bevindt. Bijvoorbeeld **Europa.**

9. Selecteer in de vervolg keuzelijst land/regio die wordt weer gegeven het land dat van toepassing is op dit eind punt. Bijvoorbeeld **Duitsland**.

10. Laat **Toevoegen als uitgeschakeld** uit staan.

11. Selecteer **OK**.

12. De Azure-eindpunt toevoegen:

    1. Selecteer voor **type** **Azure-eind punt**.

    2. Geef een **naam** op voor het eind punt.

    3. Selecteer **app service**bij **doel resource type**.

    4. Selecteer bij **doel resource** **een app service kiezen** om de vermelding van de web apps onder hetzelfde abonnement weer te geven. Kies in **resource**de app service die als eerste eind punt wordt gebruikt.

13. Selecteer onder geo-toewijzing een regio/continent waar de resource zich bevindt. Bijvoorbeeld **Noord-Amerika/Centraal-Amerika/Caribisch gebied.**

14. In de vervolg keuzelijst land/regio die wordt weer gegeven, laat u deze plaats leeg om alle bovenstaande regionale groepering te selecteren.

15. Laat **Toevoegen als uitgeschakeld** uit staan.

16. Selecteer **OK**.

    > [!Note]  
    >  Maak ten minste één eind punt met een geografisch bereik van alle (wereld) om als het standaard eindpunt voor de resource te fungeren.

17. Wanneer beide eind punten zijn toegevoegd, worden ze weer gegeven in **Traffic Manager profiel** samen met hun controle status als **online**.

    ![Eind punt status van Traffic Manager profiel](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>De wereld wijde onderneming is afhankelijk van de mogelijkheden van de geo-distributie van Azure

Door het omleiden van gegevens verkeer via Azure Traffic Manager en geografie-specifieke eind punten kunnen wereld wijde ondernemingen zich houden aan regionale voor schriften en de gegevens conform en veilig houden, wat van belang is voor het succes van lokale en externe bedrijfs locaties.

## <a name="next-steps"></a>Volgende stappen

- Zie [Cloud ontwerp patronen](https://docs.microsoft.com/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.
