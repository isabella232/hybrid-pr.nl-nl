---
title: Een app implementeren waarmee cross-Cloud in Azure en Azure Stack hub wordt geschaald
description: Meer informatie over het implementeren van een app waarmee u meerdere clouds kunt schalen in Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910636"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Een app implementeren waarmee u meerdere clouds schaalt met behulp van Azure en Azure Stack hub

Meer informatie over het maken van een cross-Cloud oplossing voor een hand matig geactiveerd proces voor het overschakelen van een door Azure Stack hub gehoste web-app naar een door Azure gehoste web-app met automatisch schalen via Traffic Manager. Dit proces zorgt voor flexibel en schaalbaar Cloud hulpprogramma wanneer het wordt geladen.

Met dit patroon is het mogelijk dat uw Tenant niet gereed is om uw app uit te voeren in de open bare Cloud. Het is echter mogelijk niet economisch haalbaar voor het bedrijf om de capaciteit te behouden die vereist is in hun on-premises omgeving voor het afhandelen van pieken in de vraag naar de app. Uw Tenant kan gebruikmaken van de elasticiteit van de open bare Cloud met hun lokale oplossing.

In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:

> [!div class="checklist"]
> - Een web-app met meerdere knoop punten maken.
> - Het proces voor doorlopende implementatie (CD) configureren en beheren.
> - Publiceer de web-app naar Azure Stack hub.
> - Maak een release.
> - Meer informatie over het bewaken en bijhouden van uw implementaties.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt gebruiken waarmee u overal hybride apps bouwt en implementeert.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

## <a name="prerequisites"></a>Vereisten

- Azure-abonnement. Maak, indien nodig, een [gratis account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) voordat u begint.
- Een Azure Stack hub geïntegreerd systeem of implementatie van Azure Stack Development Kit (ASDK).
  - Zie [install the ASDK](/azure-stack/asdk/asdk-install.md)(Engelstalig) voor instructies over het installeren van Azure stack hub.
  - Voor een ASDK-automatiserings script na de implementatie gaat u naar:[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - Het volt ooien van deze installatie kan enkele uren in beslag nemen.
- Implementeer [app service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services op Azure stack hub.
- [Maak plannen/aanbiedingen](/azure-stack/operator/service-plan-offer-subscription-overview.md) binnen de Azure stack hub-omgeving.
- Een [Tenant abonnement maken](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) binnen de Azure stack hub-omgeving.
- Een web-app maken binnen het Tenant abonnement. Noteer de URL van de nieuwe web-app voor later gebruik.
- Implementeer Azure pipelines virtual machine (VM) binnen het Tenant abonnement.
- VM van Windows Server 2016 met .NET 3,5 is vereist. Deze VM wordt gebouwd in het Tenant abonnement op Azure Stack hub als de Private build agent.
- [Windows Server 2016 met SQL 2017 VM-installatie kopie](/azure-stack/operator/azure-stack-add-vm-image.md) is beschikbaar op de Azure stack hub Marketplace. Als deze installatie kopie niet beschikbaar is, werkt u met een Azure Stack hub-operator om te controleren of deze is toegevoegd aan de omgeving.

## <a name="issues-and-considerations"></a>Problemen en overwegingen

### <a name="scalability"></a>Schaalbaarheid

Het belangrijkste onderdeel van cross-Cloud schaling is de mogelijkheid om direct en op aanvraag schalen te leveren tussen de open bare en on-premises Cloud infrastructuur, zodat u een consistente en betrouw bare service kunt bieden.

### <a name="availability"></a>Beschikbaarheid

Zorg ervoor dat lokaal geïmplementeerde apps zijn geconfigureerd voor hoge Beschik baarheid via on-premises hardwareconfiguratie en software-implementatie.

### <a name="manageability"></a>Beheerbaarheid

De oplossing voor meerdere clouds zorgt voor naadloos beheer en de vertrouwde interface tussen omgevingen. Power shell wordt aanbevolen voor beheer op meerdere platforms.

## <a name="cross-cloud-scaling"></a>Cross-Cloud schalen

### <a name="get-a-custom-domain-and-configure-dns"></a>Een aangepast domein ophalen en DNS configureren

Werk het DNS-zone bestand voor het domein bij. Azure AD controleert het eigendom van de aangepaste domein naam. Gebruik [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) voor Azure/Office 365/externe DNS-records in azure, of Voeg de DNS-vermelding toe aan [een ander DNS-REGI ster](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Een aangepast domein registreren bij een openbaar registratie service.
2. Meld u aan bij de domeinnaamregistrar voor het domein. Een goedgekeurde beheerder kan verplicht zijn om DNS-updates uit te voeren.
3. Werk het DNS-zone bestand voor het domein bij door de DNS-vermelding toe te voegen die door Azure AD wordt geleverd. (De DNS-vermelding heeft geen invloed op het gedrag van e-mail routering of webhosting.)

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Een standaard web-app met meerdere knoop punten maken in Azure Stack hub

Stel hybride continue integratie en continue implementatie (CI/CD) in voor het implementeren van web-apps naar Azure en Azure Stack hub en voor het autopushen van wijzigingen in beide Clouds.

> [!Note]  
> Azure Stack hub met de juiste installatie kopieën die zijn syndicated om te worden uitgevoerd (Windows Server en SQL) en App Service implementatie is vereist. Raadpleeg de App Service documentatie [vereisten voor het implementeren van app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)voor meer informatie.

### <a name="add-code-to-azure-repos"></a>Code toevoegen aan Azure opslag plaatsen

Azure-opslagplaatsen

1. Meld u aan bij Azure opslag plaatsen met een account met rechten voor het maken van projecten op Azure opslag plaatsen.

    Hybride CI/CD kan worden toegepast op de code van de app en de infra structuur. Gebruik [Azure Resource Manager sjablonen](https://azure.microsoft.com/resources/templates/) voor zowel privé als gehoste Cloud ontwikkeling.

    ![Verbinding maken met een project in azure opslag plaatsen](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Kloon de opslag plaats** door de standaard web-app te maken en te openen.

    ![Opslag plaats in azure-web-app klonen](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Zelf-opgenomen web-app-implementatie maken voor App Services in beide Clouds

1. Bewerk het bestand **webapplication. csproj** . Selecteren `Runtimeidentifier` en toevoegen `win10-x64` . (Zie [zelf-opgenomen implementatie](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentatie.)

    ![Web-app-project bestand bewerken](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Check de code in op Azure opslag plaatsen met behulp van team Explorer.

3. Controleer of de app-code is ingecheckt in azure opslag plaatsen.

## <a name="create-the-build-definition"></a>De build-definitie maken

1. Meld u aan bij Azure-pijp lijnen om te bevestigen dat u build-definities wilt maken.

2. Add **-r win10-x64-** code. Deze toevoeging is nood zakelijk om een zelfstandige implementatie met .NET core te activeren.

    ![Code toevoegen aan de web-app](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Voer de build uit. In het [zelf opgenomen implementatie](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) proces worden artefacten gepubliceerd die worden uitgevoerd op Azure en Azure stack hub.

## <a name="use-an-azure-hosted-agent"></a>Een gehoste agent van Azure gebruiken

Het gebruik van een gehoste build-agent in azure-pijp lijnen is een handige optie om web-apps te bouwen en te implementeren. Onderhoud en upgrades worden automatisch uitgevoerd door Microsoft Azure, waardoor een continue en ononderbroken ontwikkelings cyclus mogelijk is.

### <a name="manage-and-configure-the-cd-process"></a>Het CD-proces beheren en configureren

Azure-pijp lijnen en Azure DevOps services bieden een zeer Configureer bare en beheersbaarere pijp lijn voor meerdere omgevingen, zoals ontwikkelings-, staging-, QA-en productie omgevingen; met inbegrip van het vereisen van goed keuringen in specifieke fasen.

## <a name="create-release-definition"></a>Release definitie maken

1. Selecteer de knop met het **plus teken** om een nieuwe release toe te voegen op het tabblad **releases** in de sectie **Build and release** van Azure DevOps Services.

    ![Release-definitie maken](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Pas de Azure App Service-implementatie sjabloon toe.

   ![Azure App Service implementatie sjabloon Toep assen](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. Voeg onder **artefact toevoegen**het artefact toe voor de Azure Cloud build-app.

   ![Artefact toevoegen aan Azure Cloud build](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. Selecteer onder pijplijn tabblad de **fase, taak** koppeling van de omgeving en stel de waarden van de Azure-cloud omgeving in.

   ![Waarden van de Azure-cloud omgeving instellen](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Stel de **omgevings naam** in en selecteer het **Azure-abonnement** voor het Azure-Cloud eindpunt.

      ![Azure-abonnement voor Azure-Cloud eindpunt selecteren](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. Stel onder **app service name**de vereiste Azure app service-naam in.

      ![Azure app service-naam instellen](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Voer ' gehoste VS2017 ' in de wachtrij van de **agent** in voor de gehoste Azure cloud-omgeving.

      ![Agent wachtrij instellen voor gehoste Azure-cloud omgeving](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. Selecteer in het menu implementeren Azure App Service het geldige **pakket of** de juiste map voor de omgeving. Selecteer **OK** om **de maplocatie te**selecteren.
  
      ![Pakket of map voor Azure App Service omgeving selecteren](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Pakket of map voor Azure App Service omgeving selecteren](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Sla alle wijzigingen op en ga terug naar de **release pijplijn**.

    ![Wijzigingen in de release pijplijn opslaan](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Voeg een nieuw artefact toe om de build voor de Azure Stack hub-app te selecteren.

    ![Nieuw artefact voor Azure Stack hub-app toevoegen](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Voeg nog een omgeving toe door de implementatie van Azure App Service toe te passen.

    ![Omgeving toevoegen aan Azure App Service-implementatie](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Geef de nieuwe omgeving de naam Azure Stack.

    ![Naam omgeving in Azure App Service-implementatie](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Zoek de Azure Stack omgeving onder **taak** tabblad.

    ![Azure Stack omgeving](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Selecteer het abonnement voor het Azure Stack-eind punt.

    ![Het abonnement voor het Azure Stack-eind punt selecteren](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Stel de naam van de Azure Stack web-app in als de naam van de app service.
    ![Azure Stack naam van de web-app instellen](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Selecteer de Azure Stack agent.

    ![De Azure Stack-agent selecteren](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. Selecteer onder de sectie implementatie Azure App Service het geldige **pakket of** de juiste map voor de omgeving. Selecteer **OK** om de maplocatie te selecteren.

    ![Selecteer een map voor Azure App Service implementatie](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selecteer een map voor Azure App Service implementatie](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. Onder variabele tabblad voegt u een variabele `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` met de naam in, stelt u de waarde in op **True**en bereik Azure stack.

    ![Variabele toevoegen aan Azure-app-implementatie](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Selecteer het trigger pictogram **continue** implementatie in beide artefacten en schakel de trigger **voor het implementeren** van de implementatie in.

    ![Trigger voor continue implementatie selecteren](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Selecteer het pictogram voor waarden **voorafgaand** aan de implementatie in de Azure stack omgeving en stel de trigger in op **na release.**

    ![Voor waarden voor installatie vooraf selecteren](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Sla alle wijzigingen op.

> [!Note]  
> Sommige instellingen voor de taken zijn mogelijk automatisch gedefinieerd als [omgevings variabelen](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) bij het maken van een release definitie op basis van een sjabloon. Deze instellingen kunnen niet worden gewijzigd in de taak instellingen. in plaats daarvan moet het bovenliggende omgevings item worden geselecteerd om deze instellingen te bewerken.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Publiceren naar Azure Stack hub via Visual Studio

Door eind punten te maken, kunt u met een Azure DevOps Services-build Azure-service-Apps implementeren op Azure Stack hub. Azure-pijp lijnen maken verbinding met de build-agent, die verbinding maakt met Azure Stack hub.

1. Meld u aan bij Azure DevOps Services en ga naar de pagina app-instellingen.

2. Selecteer bij **instellingen**de optie **beveiliging**.

3. Selecteer in **VSTS-groepen**de optie **endpoint Crea tors**.

4. Selecteer **toevoegen**op het tabblad **leden** .

5. In **gebruikers en groepen toevoegen**voert u een gebruikers naam in en selecteert u deze gebruiker in de lijst met gebruikers.

6. Selecteer **Save changes**.

7. Selecteer in de lijst **VSTS groepen** de optie **endpoint Administrators**.

8. Selecteer **toevoegen**op het tabblad **leden** .

9. In **gebruikers en groepen toevoegen**voert u een gebruikers naam in en selecteert u deze gebruiker in de lijst met gebruikers.

10. Selecteer **Save changes**.

Nu de gegevens van het eind punt bestaan, zijn de Azure-pijp lijnen naar Azure Stack hub-verbinding klaar voor gebruik. De build agent in Azure Stack hub haalt instructies op uit Azure pipelines en vervolgens geeft de agent informatie over het eind punt voor communicatie met Azure Stack hub.

## <a name="develop-the-app-build"></a>De app-build ontwikkelen

> [!Note]  
> Azure Stack hub met de juiste installatie kopieën die zijn syndicated om te worden uitgevoerd (Windows Server en SQL) en App Service implementatie is vereist. Zie [vereisten voor het implementeren van app service op Azure stack hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)voor meer informatie.

Gebruik [Azure Resource Manager sjablonen](https://azure.microsoft.com/resources/templates/) zoals web-app-code van Azure opslag plaatsen om te implementeren op beide Clouds.

### <a name="add-code-to-an-azure-repos-project"></a>Code toevoegen aan een Azure opslag plaatsen-project

1. Meld u aan bij Azure opslag plaatsen met een account met rechten voor het maken van projecten op Azure Stack hub.

2. **Kloon de opslag plaats** door de standaard web-app te maken en te openen.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Zelf-opgenomen web-app-implementatie maken voor App Services in beide Clouds

1. Bewerk het bestand **webapplication. csproj** : Selecteer `Runtimeidentifier` en vervolgens toevoegen `win10-x64` . Zie voor meer informatie de documentatie over de [zelfstandige implementatie](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .

2. Gebruik team Explorer om de code in azure opslag plaatsen te controleren.

3. Controleer of de app-code is ingecheckt in azure opslag plaatsen.

### <a name="create-the-build-definition"></a>De build-definitie maken

1. Meld u aan bij Azure-pijp lijnen met een account dat een build-definitie kan maken.

2. Ga naar de pagina **Build Web Application** voor het project.

3. In **argumenten**, add **-r win10-x64-** code. Deze toevoeging is vereist voor het activeren van een zelfstandige implementatie met .NET core.

4. Voer de build uit. In het [zelf opgenomen implementatie constructie](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) proces worden artefacten gepubliceerd die kunnen worden uitgevoerd op Azure en Azure stack hub.

#### <a name="use-an-azure-hosted-build-agent"></a>Een door Azure gehoste build-agent gebruiken

Het gebruik van een gehoste build-agent in azure-pijp lijnen is een handige optie om web-apps te bouwen en te implementeren. Onderhoud en upgrades worden automatisch uitgevoerd door Microsoft Azure, waardoor een continue en ononderbroken ontwikkelings cyclus mogelijk is.

### <a name="configure-the-continuous-deployment-cd-process"></a>Het proces van de doorlopende implementatie (CD) configureren

Azure-pijp lijnen en Azure DevOps-services bieden een zeer Configureer bare en beheersbare pijp lijn voor de release van meerdere omgevingen, zoals ontwikkeling, fase ring, kwaliteits borging (QA) en productie. Dit proces kan bestaan uit het vereisen van goed keuringen in specifieke fasen van de levens cyclus van de app.

#### <a name="create-release-definition"></a>Release definitie maken

Het maken van een release definitie is de laatste stap in het app-bouw proces. Deze release definitie wordt gebruikt voor het maken van een release en het implementeren van een build.

1. Meld u aan bij Azure-pijp lijnen en ga naar **Build en release** voor het project.

2. Op het tabblad **releases** selecteert u **[+]** en kiest u **release definitie maken**.

3. Kies bij **een sjabloon selecteren de**optie **Azure app service-implementatie**en selecteer vervolgens **Toep assen**.

4. Selecteer op **artefact toevoegen**van de **bron (build-definitie)** de Azure Cloud build-app.

5. Op het tabblad **pijp lijn** selecteert u de koppeling **1 fase**, **1 taak** om **omgevings taken weer te geven**.

6. Op het tabblad **taken** voert u Azure in als de naam van de **omgeving** en selecteert u in de lijst met **Azure-abonnementen** de Cloud handelaren-Web EP.

7. Voer de naam in van de **Azure app service**, die zich `northwindtraders` in de volgende scherm opname bevindt.

8. Voor de agent fase selecteert u **gehoste VS2017** in de lijst **agent wachtrij** .

9. In **implementatie Azure app service**selecteert u het geldige **pakket of** de juiste map voor de omgeving.

10. Selecteer in **bestand of map selecteren**de optie **OK** naar **locatie**.

11. Sla alle wijzigingen op en ga terug naar de **pijp lijn**.

12. Selecteer op het tabblad **pijp lijn** de optie **artefact toevoegen**en kies het **NorthwindCloud handelaren-vat** in de lijst **bron (build Definition)** .

13. Voeg een andere omgeving toe aan **een sjabloon selecteren**. Kies **Azure app service-implementatie** en selecteer vervolgens **Toep assen**.

14. Voer `Azure Stack Hub` in als de **naam**van de omgeving.

15. Zoek en selecteer op het tabblad **taken** Azure stack hub.

16. Selecteer in de lijst met **Azure-abonnementen** **AzureStack handelaren-vaartuigen EP** voor het eind punt van de Azure stack hub.

17. Voer de naam van de Azure Stack hub-web-app in als de naam van de **app service**.

18. Onder **selectie van agent**kiest u **AzureStack-b Douglas-FIR** in de lijst **agent wachtrij** .

19. Voor **implementatie Azure app service**selecteert u het geldige **pakket of** de juiste map voor de omgeving. Selecteer in **bestand of map selecteren**de optie **OK** voor de **locatie**van de map.

20. Zoek op het tabblad **variabele** de variabele met de naam `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` . Stel de waarde van de variabele in op **waar**en stel het bereik in op **Azure stack hub**.

21. Op het tabblad **pijp lijn** selecteert u het **trigger pictogram continue implementatie** voor de NorthwindCloud handelaren-webartefacten en stelt u de **trigger voor continue implementatie** in op **ingeschakeld**. Doe hetzelfde voor het **NorthwindCloud-scheeps-** artefact.

22. Selecteer voor de Azure Stack hub-omgeving het pictogram **voor waarden voorafgaand aan implementatie** de trigger instellen op **na release**.

23. Sla alle wijzigingen op.

> [!Note]  
> Sommige instellingen voor release taken worden automatisch gedefinieerd als [omgevings variabelen](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) bij het maken van een release definitie op basis van een sjabloon. Deze instellingen kunnen niet worden gewijzigd in de taak instellingen, maar kunnen worden gewijzigd in de bovenliggende omgevings items.

## <a name="create-a-release"></a>Een release maken

1. Open de lijst **release** op het tabblad **pipeline** en selecteer **release maken**.

2. Voer een beschrijving in voor de release, Controleer of de juiste artefacten zijn geselecteerd en selecteer vervolgens **maken**. Na enkele ogen blikken wordt een banner weer gegeven dat aangeeft dat de nieuwe release is gemaakt en de release naam wordt weer gegeven als een koppeling. Selecteer de koppeling om de pagina release overzicht weer te geven.

3. De pagina release overzicht bevat details over de release. In de volgende scherm opname voor ' release-2 ' toont de sectie **omgevingen** de **Implementatie status** voor Azure als ' wordt uitgevoerd ', en de status voor Azure stack hub is geslaagd. Wanneer de implementatie status voor de Azure-omgeving wordt gewijzigd in ' geslaagd ', wordt een banner weer gegeven dat aangeeft dat de release gereed is voor goed keuring. Wanneer een implementatie in behandeling is of mislukt, wordt een pictogram met een blauw **(i)-** informatie weer gegeven. Beweeg de muis aanwijzer over het pictogram om een pop-upvenster te zien met de reden voor de vertraging of de fout.

4. In andere weer gaven, zoals de lijst met releases, wordt ook een pictogram weer gegeven dat aangeeft dat goed keuring in behandeling is. In het pop-upvenster voor dit pictogram ziet u de naam van de omgeving en meer informatie over de implementatie. Het is eenvoudig voor een beheerder de algehele voortgang van releases te zien en te zien welke releases wachten op goed keuring.

## <a name="monitor-and-track-deployments"></a>Implementaties controleren en bijhouden

1. Selecteer op de pagina overzicht van **Release 2** de optie **Logboeken**. Tijdens een implementatie toont deze pagina het Live logboek van de agent. In het linkerdeel venster ziet u de status van elke bewerking in de implementatie voor elke omgeving.

2. Selecteer het persoons pictogram in de kolom **actie** voor een goed keuring vóór de implementatie of na de implementatie om te zien wie de implementatie heeft goedgekeurd (of geweigerd) en welk bericht ze hebben geleverd.

3. Nadat de implementatie is voltooid, wordt het hele logboek bestand weer gegeven in het rechterdeel venster. Selecteer een wille keurige **stap** in het linkerdeel venster om het logboek bestand voor één stap weer te geven, zoals de **opdracht initialiseren**. De mogelijkheid om afzonderlijke logboeken te bekijken maakt het gemakkelijker om delen van de algehele implementatie te traceren en fouten op te sporen. **Sla** het logboek bestand op voor een stap of **down load alle logboeken als zip**.

4. Open het tabblad **samen vatting** om algemene informatie over de release weer te geven. Deze weer gave bevat details over de build, de omgevingen waarin deze is geïmplementeerd, de implementatie status en andere informatie over de release.

5. Selecteer een omgevings koppeling (**Azure** of **Azure stack hub**) om informatie over bestaande en in behandeling zijnde implementaties te bekijken in een specifieke omgeving. Gebruik deze weer gaven als snelle manier om te controleren of dezelfde build in beide omgevingen is geïmplementeerd.

6. Open de **geïmplementeerde productie-app** in een browser. Open bijvoorbeeld de URL voor de website van Azure-app Services `https://[your-app-name\].azurewebsites.net` .

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>Integratie van Azure en Azure Stack hub biedt een schaal bare oplossing voor meerdere clouds

Een flexibele en robuuste multi-Cloud service biedt gegevens beveiliging, back-up en redundantie, consistente en snelle Beschik baarheid, schaal bare opslag en distributie en geo-compatibele route ring. Dit hand matig geactiveerde proces zorgt voor betrouw bare en efficiënte belasting wisseling tussen gehoste web-apps en onmiddellijke Beschik baarheid van cruciale gegevens.

## <a name="next-steps"></a>Volgende stappen

- Zie [Cloud ontwerp patronen](https://docs.microsoft.com/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.
