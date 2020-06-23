---
title: Hybride Cloud connectiviteit configureren in Azure en Azure Stack hub
description: Meer informatie over het configureren van hybride Cloud connectiviteit met Azure en Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0e1a0fc4fb4110fdb406d4b4b2e72abb8f5412c9
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910313"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Hybride Cloud connectiviteit configureren met Azure en Azure Stack hub

U kunt toegang krijgen tot resources met beveiliging in wereld wijd Azure en Azure Stack hub met behulp van het hybride verbindings patroon.

In deze oplossing bouwt u een voorbeeld omgeving in voor het volgende:

> [!div class="checklist"]
> - Bewaar gegevens on-premises om te voldoen aan de vereisten voor privacy of regelgeving, maar blijf toegang tot de wereld wijde Azure-resources.
> - Onderhoud van een verouderd systeem tijdens het gebruik van implementaties van apps in de Cloud en bronnen in wereld wijd Azure.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub is een uitbrei ding van Azure. Azure Stack hub biedt de flexibiliteit en innovatie van Cloud Computing naar uw on-premises omgeving, waardoor u de enige hybride Cloud kunt maken en implementeren.  
> 
> In het artikel [hybride overwegingen](overview-app-design-considerations.md) voor het ontwerpen van een app worden de pijlers van de software kwaliteit (plaatsing, schaal baarheid, Beschik baarheid, tolerantie, beheersbaarheid en beveiliging) beoordeeld voor het ontwerpen, implementeren en beheren van hybride apps. De ontwerp overwegingen helpen bij het optimaliseren van het ontwerp van hybride apps, zodat de uitdagingen in productie omgevingen worden geminimaliseerd.

## <a name="prerequisites"></a>Vereisten

Er zijn enkele onderdelen vereist voor het bouwen van een hybride connectiviteits implementatie. Sommige van deze onderdelen nemen enige tijd in beslag.

### <a name="azure"></a>Azure

- Als u nog geen abonnement op Azure hebt, maak dan een [gratis account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) aan voordat u begint.
- Een [Web-app](https://docs.microsoft.com/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?view=vsts&tabs=vsts) maken in Azure. Noteer de URL van de web-app, omdat u deze nodig hebt in de oplossing.

### <a name="azure-stack-hub"></a>Azure Stack hub

Een Azure OEM/Hardware-partner kan een productie Azure Stack hub implementeren en alle gebruikers kunnen een Azure Stack Development Kit (ASDK) implementeren.

- Gebruik uw productie Azure Stack hub of implementeer de ASDK.
   >[!Note]
   >Het implementeren van de ASDK kan tot 7 uur duren. Dit kan daarom worden gepland.

- Implementeer [app service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services op Azure stack hub.
- [Maak plannen en aanbiedingen](/azure-stack/operator/service-plan-offer-subscription-overview.md) in de Azure stack hub-omgeving.
- Een [Tenant abonnement maken](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) binnen de Azure stack hub-omgeving.

### <a name="azure-stack-hub-components"></a>Azure Stack hub-onderdelen

Een Azure Stack hub-operator moet de App Service implementeren, plannen en aanbiedingen maken, een Tenant abonnement maken en de installatie kopie van Windows Server 2016 toevoegen. Als u deze onderdelen al hebt, moet u ervoor zorgen dat ze voldoen aan de vereisten voordat u deze oplossing start.

In dit voor beeld van een oplossing wordt ervan uitgegaan dat u een eenvoudige kennis van Azure en Azure Stack hub hebt. Lees de volgende artikelen voor meer informatie voordat u de oplossing start:

- [Inleiding tot Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Hoofd concepten van Azure Stack-hub](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a>Voordat u begint

Controleer of u aan de volgende criteria voldoet voordat u begint met het configureren van hybride Cloud connectiviteit:

- U hebt een extern gericht openbaar IPv4-adres nodig voor uw VPN-apparaat. Dit IP-adres kan zich niet achter een NAT (netwerkadresomzetting) bevinden.
- Alle resources worden in dezelfde regio/locatie geïmplementeerd.

#### <a name="solution-example-values"></a>Voorbeeld waarden voor oplossingen

Voor de voor beelden in deze oplossing worden de volgende waarden gebruikt. U kunt deze waarden gebruiken om een test omgeving te maken of om een beter inzicht te krijgen in de voor beelden. Zie [over VPN gateway-instellingen](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings)voor meer informatie over instellingen voor VPN-gateway.

Verbindings specificaties:

- **VPN-type**: op route gebaseerd
- **Verbindings type**: site-naar-site (IPSec)
- **Gateway type**: VPN
- **Azure-verbindings naam**: Azure-gateway-AzureStack-S2SGateway (de portal zal deze waarde automatisch invullen)
- **Azure stack hub-verbindings naam**: AzureStack-gateway-Azure-S2SGateway (de portal zal deze waarde automatisch invullen)
- **Gedeelde sleutel**: alle compatibele en VPN-hardware, met overeenkomende waarden aan beide zijden van de verbinding
- **Abonnement**: elk gewenst abonnement
- **Resource groep**: test-infra structuur

IP-adressen van netwerk en subnet:

| Azure/Azure Stack hub-verbinding | Naam | Subnet | IP-adres |
|---|---|---|---|
| Azure vNet | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Azure Stack hub vNet | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Azure Virtual Network-gateway | Azure-gateway |  |  |
| Azure Stack hub Virtual Network gateway | AzureStack-gateway |  |  |
| Open bare IP van Azure | Azure-GatewayPublicIP |  | Bepaald bij maken |
| Open bare IP van Azure Stack hub | AzureStack-GatewayPublicIP |  | Bepaald bij maken |
| Lokale Azure-netwerk gateway | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Open bare IP-waarde van Azure Stack hub |
| Lokale netwerk gateway van Azure Stack hub | Azure-S2SGateway<br>10.100.102.0/23 |  | Open bare IP-waarde van Azure |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Een virtueel netwerk maken in de wereld wijde Azure-en Azure Stack hub

Gebruik de volgende stappen om een virtueel netwerk te maken met behulp van de portal. U kunt deze [voorbeeld waarden](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) gebruiken als u dit artikel alleen als een oplossing gebruikt. Als u dit artikel gebruikt voor het configureren van een productie omgeving, vervangt u de voorbeeld instellingen door uw eigen waarden.

> [!IMPORTANT]
> U moet ervoor zorgen dat er geen overlap ping is voor IP-adressen in azure-of Azure Stack hub vNet-adres ruimten.

Een vNet maken in Azure:

1. Gebruik uw browser om verbinding te maken met de [Azure Portal](https://portal.azure.com/) en u aan te melden met uw Azure-account.
2. Selecteer **Een resource maken**. In het veld **Marketplace doorzoeken** voert u ' virtueel netwerk ' in. Selecteer **virtueel netwerk** in de resultaten.
3. Selecteer in de lijst **Selecteer een implementatie model** **Resource Manager**en selecteer vervolgens **maken**.
4. Configureer de VNet-instellingen op het **virtuele netwerk maken**. De vereiste veld namen worden voorafgegaan door een rood sterretje.  Wanneer u een geldige waarde opgeeft, wordt het sterretje gewijzigd in een groen vinkje.

Een vNet maken in Azure Stack hub:

1. Herhaal de bovenstaande stappen (1-4) met behulp van de Azure Stack hub- **Tenant Portal**.

## <a name="add-a-gateway-subnet"></a>Een gatewaysubnet toevoegen

Voordat u het virtuele netwerk verbindt met een gateway, moet u het gateway-subnet maken voor het virtuele netwerk waarmee u verbinding wilt maken. De Gateway Services gebruiken de IP-adressen die u opgeeft in het gateway-subnet.

Navigeer in het [Azure Portal](https://portal.azure.com/)naar het virtuele netwerk van Resource Manager waar u een virtuele netwerk gateway wilt maken.

1. Selecteer het vNet om de pagina **Virtual Network** te openen.
2. Selecteer in **instellingen**de optie **subnetten**.
3. Selecteer op de pagina **subnetten** **+ Gateway-subnet** om de pagina **subnet toevoegen** te openen.

    ![Gateway-subnet toevoegen](media/solution-deployment-guide-connectivity/image4.png)

4. De **naam** voor het subnet wordt automatisch gevuld met de waarde ' GatewaySubnet '. Deze waarde is vereist voor Azure om het subnet als het gateway-subnet te herkennen.
5. Wijzig de **adres bereik** waarden die zijn opgegeven om te voldoen aan uw configuratie vereisten en selecteer vervolgens **OK**.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Een Virtual Network-gateway maken in Azure en Azure Stack

Gebruik de volgende stappen voor het maken van een virtuele netwerk gateway in Azure.

1. Klik aan de linkerkant van de portal pagina **+** en voer ' virtuele netwerk gateway ' in het zoek veld in.
2. Selecteer in **resultaten** **virtuele netwerk gateway**.
3. Selecteer in **virtuele netwerk gateway** **maken** om de pagina **virtuele netwerk gateway maken** te openen.
4. Geef bij **virtuele netwerk gateway maken**de waarden op voor uw netwerk gateway met behulp van de **voorbeeld waarden van de zelf studie**. Neem de volgende aanvullende waarden op:

   - **SKU**: basis
   - **Virtual Network**: Selecteer het virtuele netwerk dat u eerder hebt gemaakt. Het gateway-subnet dat u hebt gemaakt, wordt automatisch geselecteerd.
   - **Eerste IP-configuratie**: het open bare IP-adres van uw gateway.
     - Selecteer **IP-configuratie voor de gateway maken**. Hiermee gaat u naar de pagina **openbaar IP-adres kiezen** .
     - Selecteer **+ nieuwe maken** om de pagina **openbaar IP-adres maken** te openen.
     - Voer een **naam** in voor uw open bare IP-adres. Verlaat de SKU als **basis**en selecteer **OK** om uw wijzigingen op te slaan.

       > [!Note]
       > Momenteel ondersteunt VPN Gateway alleen dynamische toewijzing van open bare IP-adressen. Dit betekent echter niet dat het IP-adres verandert nadat het aan uw VPN-gateway is toegewezen. De enige keer dat het open bare IP-adres wordt gewijzigd wanneer de gateway wordt verwijderd en opnieuw wordt gemaakt. Het wijzigen van het formaat, het opnieuw instellen of andere interne onderhouds-en upgrade-instellingen voor uw VPN-gateway veranderen het IP-adres niet.

5. Controleer de gateway-instellingen.
6. Selecteer **maken** om de VPN-gateway te maken. De gateway-instellingen worden gevalideerd en de tegel ' de gateway van het virtuele netwerk implementeren ' wordt weer gegeven op uw dash board.

   >[!Note]
   >Het aanmaken van een gateway kan tot 45 minuten duren. U moet mogelijk uw portal-pagina vernieuwen om de voltooide status te kunnen zien.

    Nadat de gateway is gemaakt, ziet u het IP-adres dat hieraan is toegewezen door te kijken naar het virtuele netwerk in de portal. De gateway wordt weergegeven als verbonden apparaat. Selecteer het apparaat voor meer informatie over de gateway.

7. Herhaal de vorige stappen (1-5) op uw Azure Stack hub-implementatie.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>De lokale netwerk gateway maken in Azure en Azure Stack hub

De lokale netwerkgateway verwijst doorgaans naar uw on-premises locatie. U geeft de site een naam die door Azure of Azure Stack hub kan worden verwezen, en vervolgens geeft u het volgende op:

- Het IP-adres van het on-premises VPN-apparaat waarvoor u een verbinding maakt.
- De IP-adres voorvoegsels die via de VPN-gateway worden doorgestuurd naar het VPN-apparaat. De adresvoorvoegsels die u opgeeft, zijn de voorvoegsels die zich in uw on-premises netwerk bevinden.

  >[!Note]
  >Als uw on-premises netwerk wordt gewijzigd of als u het open bare IP-adres voor het VPN-apparaat wilt wijzigen, kunt u deze waarden later bijwerken.

1. Selecteer in de portal **+ een resource maken**.
2. Voer in het zoekvak **lokale netwerk gateway**in en selecteer vervolgens **Enter** om te zoeken. Er wordt een lijst met resultaten weer gegeven.
3. Selecteer **lokale netwerk gateway**en selecteer vervolgens **maken** om de pagina **lokale netwerk gateway maken** te openen.
4. In **lokale netwerk gateway maken**geeft u de waarden voor uw lokale netwerk gateway op met behulp van de **voorbeeld waarden van de zelf studie**. Neem de volgende aanvullende waarden op:

    - **IP-adres**: het open bare IP-adres van het VPN-apparaat waarvan u Azure of Azure stack hub verbinding wilt laten maken. Geef een geldig openbaar IP-adres op dat zich niet achter een NAT bevindt, zodat Azure het adres kan bereiken. Als u het IP-adres momenteel niet hebt, kunt u een waarde uit het voor beeld gebruiken als tijdelijke aanduiding. U moet terugkeren en de tijdelijke aanduiding vervangen door het open bare IP-adres van uw VPN-apparaat. Azure kan pas verbinding maken met het apparaat als u een geldig adres opgeeft.
    - **Adres ruimte**: het adres bereik voor het netwerk dat door dit lokale netwerk wordt vertegenwoordigd. U kunt meerdere adresruimtebereiken toevoegen. Zorg ervoor dat de door u opgegeven bereiken elkaar niet overlappen met bereiken van andere netwerken waarmee u verbinding wilt maken. Azure stuurt het adresbereik dat u opgeeft, door naar het IP-adres van het on-premises VPN-apparaat. Gebruik uw eigen waarden als u verbinding wilt maken met uw on-premises site, geen voorbeeld waarde.
    - **BGP-instellingen configureren**: alleen gebruiken bij het configureren van BGP. Als dat niet het geval is, selecteert u deze optie niet.
    - **Abonnement**: Controleer of het juiste abonnement wordt weer gegeven.
    - **Resource groep**: Selecteer de resource groep die u wilt gebruiken. U kunt een nieuwe resource groep maken of er een selecteren die u al hebt gemaakt.
    - **Locatie**: Selecteer de locatie waarin dit object wordt gemaakt. Mogelijk wilt u dezelfde locatie selecteren als uw VNet, maar u hoeft dit niet te doen.
5. Wanneer u klaar bent met het opgeven van de vereiste waarden, selecteert u **maken** om de lokale netwerk gateway te maken.
6. Herhaal deze stappen (1-5) voor uw implementatie van Azure Stack hub.

## <a name="configure-your-connection"></a>Uw verbinding configureren

Voor site-naar-site-verbindingen met een on-premises netwerk is een VPN-apparaat vereist. Het VPN-apparaat dat u configureert, wordt een verbinding genoemd. Als u uw verbinding wilt configureren, hebt u het volgende nodig:

- Een gedeelde sleutel. Deze sleutel is dezelfde gedeelde sleutel die u opgeeft wanneer u uw site-naar-site-VPN-verbinding maakt. In onze voorbeelden gebruiken we een eenvoudige gedeelde sleutel. We raden u aan een complexere sleutel te genereren.
- Het open bare IP-adres van de gateway van uw virtuele netwerk. U kunt het openbare IP-adres weergeven met behulp van Azure Portal, PowerShell of de CLI. Als u het open bare IP-adres van uw VPN-gateway wilt zoeken met behulp van de Azure Portal, gaat u naar de gateways van het virtuele netwerk en selecteert u de naam van uw gateway.

Gebruik de volgende stappen om een site-naar-site-VPN-verbinding te maken tussen de gateway van uw virtuele netwerk en uw on-premises VPN-apparaat.

1. Selecteer in het Azure Portal **+ een resource maken**.
2. Zoeken naar **verbindingen**.
3. Selecteer in **resultaten** **verbindingen**.
4. Selecteer bij **verbinding**de optie **maken**.
5. Configureer bij **verbinding maken**de volgende instellingen:

    - **Verbindings type**: Selecteer site-naar-site (IPSec).
    - **Resource groep**: Selecteer de resource groep testen.
    - **Virtual Network gateway**: Selecteer de virtuele netwerk gateway die u hebt gemaakt.
    - **Lokale netwerk gateway**: Selecteer de lokale netwerk gateway die u hebt gemaakt.
    - **Verbindings naam**: deze naam wordt automatisch ingevuld met behulp van de waarden van de twee gateways.
    - **Gedeelde sleutel**: deze waarde moet overeenkomen met de waarde die u gebruikt voor uw lokale on-PREMISES VPN-apparaat. In het voor beeld van de zelf studie wordt gebruikgemaakt van ' abc123 ', maar u moet een complexere waarde gebruiken. Het belangrijkste is dat deze waarde *moet* overeenkomen met de waarde die u opgeeft bij het configureren van uw VPN-apparaat.
    - De waarden voor het **abonnement**, de **resource groep**en de **locatie** zijn vast.

6. Selecteer **OK** om uw verbinding te maken.

U kunt de verbinding zien op de pagina **verbindingen** van de gateway van het virtuele netwerk. De status gaat van *onbekend* naar *verbinding maken*en vervolgens naar *geslaagd*.

## <a name="next-steps"></a>Volgende stappen

- Zie [Cloud ontwerp patronen](https://docs.microsoft.com/azure/architecture/patterns)voor meer informatie over Azure Cloud-patronen.
