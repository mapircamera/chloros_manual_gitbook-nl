# Kaartmarkeringen

Het tabblad Kaart geeft uw afbeeldingen weer op een interactieve 2D-kaart op basis van hun GPS-coördinaten. Dit biedt een geografisch overzicht van uw opnamesessie en helpt u de ruimtelijke dekking te visualiseren. Het is ook handig wanneer u uw afbeeldingen voor het eerst importeert, om snel afbeeldingen te verwijderen die u niet hoeft te verwerken.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Toegang tot het tabblad Kaart

1. Open of maak een project in Chloros.
2. Importeer afbeeldingen die GPS-metadata bevatten.
3. Klik op het tabblad **Kaart** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> in de linkerzijbalk.
4. De kaart geeft markeringen weer op de GPS-locatie van elke afbeelding.

{% hint style="info" %}
**GPS vereist**: alleen afbeeldingen met ingebedde GPS-coördinaten in hun EXIF-metadata worden op de kaart weergegeven. Zorg ervoor dat GPS is ingeschakeld op uw camera tijdens het maken van de opname.
{% endhint %}

***

## Afbeeldingen aanpassen vanuit het tabblad Kaart

Het tabblad **Kaart** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> heeft dezelfde functies voor toevoegen  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  en verwijder  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  bestandsfuncties als het tabblad [**Bestandsbrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Het toont ook dezelfde lijst met projectbestanden, maar met andere kolomkoppen:

### Bestandsnaam

* Originele bestandsnaam van de camera
* Behoudt de naamgevingsconventie van de camera (bijv. IMG\_0001.RAW)

### Breedtegraad

* De breedtegraad van de afbeelding

### Lengtegraad

* De lengtegraad van de afbeelding

### Hoogte

* De hoogte van de afbeelding

{% hint style="info" %}
Als u op de kolomkoppen van de tabel klikt, worden de rijgegevens gesorteerd
{% endhint %}

***

## Afbeeldingsmarkeringen

Elke afbeelding met GPS-gegevens wordt weergegeven door een markering op de kaart:

### Weergave van markeringen

* Markeringen geven de exacte GPS-coördinaten aan waar elke afbeelding is gemaakt.
* Geclusterde markeringen kunnen worden gegroepeerd wanneer u uitzoomt.
* Zoom in om de locaties van individuele afbeeldingen te zien.

{% hint style="success" %}
SUPERZOOM: Wanneer u het maximale zoomniveau van de kaarttegelprovider bereikt, wordt de tegel bij verder inzoomen vergroot, zodat u markeringen kunt zien die dicht bij elkaar liggen.
{% endhint %}

### Voorbeeld bij aanwijzen

* **Beweeg uw muis** over een markering om een miniatuurvoorbeeld van die afbeelding te zien.
* Zo kunt u snel visuele identificatie uitvoeren zonder de kaartweergave te verlaten.
* Handig voor het lokaliseren van specifieke afbeeldingen binnen een grote opnamesessie.

***

## Kaarttegelaanbieders

{% hint style="success" %}
**Automatische selectie**: Chloros kiest automatisch de tegeldienst die het beste zoomniveau biedt voor uw huidige kaartlocatie. U kunt desgewenst handmatig schakelen tussen providers.
{% endhint %}

Het tabblad Kaart ondersteunt twee tegelleveranciers voor de achtergrondkaartbeelden:

### Google Maps

* Standaard satelliet- en kaartbeelden van Google
* Het beste voor algemene wereldwijde dekking

### ESRI

* Satelliet- en luchtfoto&#x27;s van ESRI ArcGIS
* Biedt vaak beelden met een hogere resolutie in bepaalde regio&#x27;s

***

## Soorten kaarttegels

U kunt het type kaartlaag kiezen (van links naar rechts):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Terrein

Toont hoogteprofielen en kaarttegels met details (wegen, enz.)

### Kaart

Toont standaard (lagere bandbreedte) kaarttegels met details (wegen, enz.)

### Satelliet

Toont gedetailleerde (hogere bandbreedte) satellietkaarttegels

### Hybride

Toont satellietkaarttegels met toegevoegde details (wegen, enz.)

***

## Kaartnavigatie

### Zoomknoppen

* **In-/uitzoomen**: gebruik het scrollwiel van de muis of de zoomknoppen
* **Volledig scherm**: kaart op volledig scherm weergeven

### Panknoppen

* **Pannen**: klik en sleep om over de kaart te bewegen

***

## Gebruiksscenario&#x27;s

### Visualisatie van vliegroutes

* Bekijk het dekkingsgebied van drone-opnamesessies
* Identificeer hiaten in de beelddekking
* Controleer de uitvoering van vliegroutes

### Beoordeling van grondonderzoek

* Bekijk de ruimtelijke verdeling van opnames vanaf de grond
* Lokaliseer kalibratiedoelbeelden ten opzichte van het onderzoeksgebied
* Plan extra opnamelocaties

### Kwaliteitscontrole

* Identificeer snel beelden die op onverwachte locaties zijn vastgelegd.
* Controleer de GPS-nauwkeurigheid in de dataset.
* Vergelijk beeldlocaties met veldnotities.

***

## Probleemoplossing

### Er worden geen markeringen weergegeven

**Mogelijke oorzaken:**

* Beelden bevatten geen GPS-metadata.
* GPS was tijdens het vastleggen uitgeschakeld op de camera.
* EXIF-gegevens zijn verwijderd door externe software.

**Oplossing**: Controleer of GPS is ingeschakeld op uw camera en importeer de originele bestanden opnieuw.

### Markeringen op verkeerde locatie

**Mogelijke oorzaken:**

* De GPS van de camera had een slechte satellietfix.
* GPS-afwijking tijdens het vastleggen.

**Oplossing**: Dit is meestal een probleem met de vastleggingstijd; overweeg het gebruik van PPK/RTK GPS voor precisietoepassingen.
