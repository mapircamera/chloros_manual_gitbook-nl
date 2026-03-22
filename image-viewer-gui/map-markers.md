# Kaartmarkeringen

Op het tabblad Kaart worden uw beelden op basis van hun GPS-coördinaten weergegeven op een interactieve 2D-kaart. Dit biedt een geografisch overzicht van uw opnamesessie en helpt u de ruimtelijke dekking in beeld te brengen. Het is ook handig bij het importeren van uw beelden om snel alle beelden te verwijderen die u niet hoeft te verwerken.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Het tabblad Kaart openen

1. Open of maak een project aan in Chloros
2. Importeer afbeeldingen die GPS-metadata bevatten
3. Klik op het tabblad **Kaart** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> in de linkerzijbalk
4. De kaart toont markeringen op de GPS-locatie van elke afbeelding

{% hint style="info" %}
**GPS vereist**: Alleen afbeeldingen met ingebedde GPS-coördinaten in hun EXIF-metadata worden op de kaart weergegeven. Zorg ervoor dat GPS op uw camera is ingeschakeld tijdens het maken van de opname.
{% endhint %}

***

## Afbeeldingen aanpassen via het tabblad Kaart

Het tabblad **Kaart**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> heeft dezelfde knop voor toevoegen  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  en verwijder  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  bestandsknoppen als het [**Bestandsbrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Het toont ook dezelfde lijst met projectbestanden, maar met andere kolomkoppen:

### Bestandsnaam

* Originele bestandsnaam van de camera
* Houdt de naamgevingsconventie van de camera aan (bijv. IMG\_0001.RAW)

### Breedtegraad

* De breedtegraad van de afbeelding

### Lengtegraad

* De lengtegraad van de afbeelding

### Hoogte

* De hoogte van de afbeelding

{% hint style="info" %}
Als u op de kolomkoppen van de tabel klikt, worden de rijgegevens ook gesorteerd
{% endhint %}

***

## Afbeeldingsmarkeringen

Elke afbeelding met GPS-gegevens wordt weergegeven door een markering op de kaart:

### Weergave van markeringen

* Markeringen geven de exacte GPS-coördinaten aan waar elke afbeelding is gemaakt
* Geclusterde markeringen kunnen bij uitzoomen worden gegroepeerd
* Zoom in om individuele afbeeldingslocaties te zien

{% hint style="success" %}
SUPER-ZOOM: Wanneer u het maximale zoomniveau van de kaarttegelprovider bereikt, wordt de tegel bij verder inzoomen vergroot, waardoor u markeringen kunt zien die dicht bij elkaar liggen.
{% endhint %}

### Voorbeeld bij aanwijzen

* **Beweeg uw muis** over een markering om een miniatuurvoorbeeld van die afbeelding te zien
* Dit maakt snelle visuele identificatie mogelijk zonder de kaartweergave te verlaten
* Handig voor het vinden van specifieke afbeeldingen binnen een grote opnamesessie

***

## Kaarttegelproviders

{% hint style="success" %}
**Automatische selectie**: Chloros kiest automatisch de tegeldienst die het beste zoomniveau biedt voor uw huidige kaartlocatie. U kunt desgewenst handmatig tussen aanbieders schakelen.
{% endhint %}

Het tabblad Kaart ondersteunt twee tegelaanbieders voor de achtergrondkaartbeelden:

### Google Maps

* Standaard satelliet- en kaartbeelden van Google
* Het beste voor algemene wereldwijde dekking

### ESRI

* Satelliet- en luchtfoto&#x27;s van ESRI ArcGIS
* Biedt vaak beelden met een hogere resolutie in bepaalde regio&#x27;s

***

## Soorten kaarttegels

U kunt het type kaartlaag kiezen (van links naar rechts):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terrein

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

* **In-/uitzoomen**: Gebruik het scrollwiel van de muis of de zoomknoppen
* **Volledig scherm**: De kaart op volledig scherm weergeven

### Pan-knoppen

* **Pannen**: Klik en sleep om over de kaart te bewegen***

## Toepassingen

### Visualisatie van vliegroutes

* Bekijk het dekkingsgebied van drone-opnamesessies
* Identificeer hiaten in de beelddekking
* Controleer de uitvoering van de vliegroute

### Beoordeling van grondonderzoek

* Bekijk de ruimtelijke verdeling van opnames vanaf de grond
* Lokaliseer kalibratiedoelbeelden ten opzichte van het onderzoeksgebied
* Plan aanvullende opnamelocaties

### Kwaliteitscontrole

* Identificeer snel beelden die op onverwachte locaties zijn vastgelegd
* Controleer de GPS-nauwkeurigheid in de dataset
* Vergelijk beeldlocaties met veldnotities

***

## Probleemoplossing

### Er verschijnen geen markeringen

**Mogelijke oorzaken:**

* Beelden bevatten geen GPS-metadata
* GPS was uitgeschakeld op de camera tijdens het vastleggen
* EXIF-gegevens zijn verwijderd door externe software

**Oplossing**: Controleer of GPS is ingeschakeld op uw camera en importeer de originele bestanden opnieuw

### Markeringen op verkeerde locatie

**Mogelijke oorzaken:**

* De GPS van de camera had een slechte satellietpositie
* GPS-afwijking tijdens het vastleggen

**Oplossing**: Dit is meestal een probleem met het tijdstip van vastleggen; overweeg het gebruik van PPK/RTK GPS voor precisietoepassingen
