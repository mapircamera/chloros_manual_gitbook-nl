# Kaartmarkeringen

Het tabblad Kaart geeft je afbeeldingen weer op een interactieve 2D-kaart op basis van hun GPS-coördinaten. Het biedt je een geografisch overzicht van een opnamesessie en is de snelste manier – direct na het importeren – om afbeeldingen te verwijderen die je niet wilt bewerken.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Het tabblad Kaart openen

1. Open of maak een project aan in Chloros
2. Importeer foto&#x27;s die GPS-metadata bevatten
3. Klik op het tabblad **Kaart** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> in de linkerzijbalk
4. Op de kaart wordt bij de GPS-locatie van elke foto een markering weergegeven

{% hint style="info" %}
**GPS vereist**: alleen afbeeldingen met GPS-coördinaten in hun EXIF-metadata verschijnen op de kaart. Een afbeelding zonder coördinaten blijft wel in het project aanwezig en wordt nog steeds normaal verwerkt — er staat alleen geen markering bij.
{% endhint %}

***

## Afbeeldingen aanpassen via het tabblad &#x27;Kaart&#x27;

Het tabblad **Kaart**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> heeft dezelfde knoppen voor het toevoegen <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> en verwijderen <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> van bestanden als het tabblad [**Bestandsbrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Het toont dezelfde lijst met projectbestanden, met geografische kolommen:

| Kolom        | Inhoud                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Naam**      | De bestandsnaam zoals deze uit de camera kwam                             |
| **Breedtegraad** | Decimale graden, zes decimalen                                |
| **Lengtegraad** | Decimale graden, zes decimalen                                |
| **Hoogte**  | Meters, één decimaal — `-` wanneer de afbeelding geen hoogte bevat |

{% hint style="info" %}
Klik op een kolomkop om op die kolom te sorteren; klik nogmaals om de volgorde om te keren.
{% endhint %}

{% hint style="warning" %}
**Hoogte is de hoogte boven zeeniveau, niet de hoogte boven de grond.** De waarde is afkomstig van de EXIF-tag `GPSAltitude` van de afbeelding, die is gekoppeld aan het gemiddelde zeeniveau. Het is niet de vlieghoogte boven het terrein, en Chloros zal hieruit geen grondresolutie afleiden — boven een veld op 300 m boven zeeniveau registreert een drone op 100 m AGL hier ongeveer 400 m. Gebruik de kolom om uitschieters op te sporen en een consistente vlieghoogte te controleren, niet als een AGL-meting.
{% endhint %}

***

## Afbeeldingsmarkeringen

Elke afbeelding met GPS-gegevens krijgt een markering op de bijbehorende coördinaten.

### Weergave van markeringen

* Markeringen bevinden zich op de exacte coördinaten die voor elke opname zijn geregistreerd
* Markeringen die dicht bij elkaar liggen, kunnen visueel overlappen wanneer je uitzoomt — zoom in om ze van elkaar te scheiden
* Geselecteerde en gemarkeerde markeringen worden boven de rest weergegeven

### Voorbeeld bij aanwijzen

* **Beweeg de muis** over een marker om een miniatuur van die afbeelding met de bestandsnaam te laten verschijnen
* **Klik**op een markering om de afbeelding te selecteren en het pop-upvenster**vast te zetten** — het blijft staan totdat u ergens anders klikt. Zolang een pop-upvenster is vastgezet, verdwijnt het niet wanneer u met de muis over andere markeringen beweegt
* Dit is de snelste manier om één bepaald beeld in een grote sessie te vinden zonder de kaart te verlaten

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Het tabblad ‘Kaart’ geeft alle afbeeldingen met geotags in het project weer</p></figcaption></figure>### Superzoom

{% hint style="success" %}
**SUPERZOOM**: wanneer je de maximale zoomfactor bereikt waarvoor de tegelprovider beeldmateriaal heeft, worden de tegels bij verder inzoomen nog groter in plaats van dat het inzoomen stopt, zodat je markeringen die bijna op elkaar liggen van elkaar kunt onderscheiden.
{% endhint %}

* Superzoom werkt alleen wanneer je **op** de maximale zoomfactor van de aanbieder voor die locatie zit en de tegels volledig zijn geladen. Daaronder werkt het zoomen normaal
* Het bereik is **1× tot 32×** bovenop de eigen maximale zoomfactor van de aanbieder
* Een indicator in de hoek geeft de huidige superzoom weer als percentage, en een **×**-knop ernaast brengt je met één klik terug naar de normale zoomstand
* Uitzoomen gaat altijd door naar de kaart zelf, zodat je nooit vast komt te zitten in superzoom
* Zoomen en pannen tijdens superzoom geeft de resulterende verschuiving door aan de kaart, zodat het gebied buiten het midden waarnaar je bent verplaatst, doorgaat met het opvragen van tegels in plaats van leeg te worden
* Markeringen worden weergegeven als vectorelementen in plaats van gerasterd, zodat ze op elk superzoomniveau scherp blijven

***

## Aanbieders van kaarttegels

{% hint style="success" %}
**Automatische selectie**: Chloros kiest de tegelservice die het beste zoomniveau biedt voor de locatie van je afbeeldingen. Je kunt op elk moment handmatig wisselen.
{% endhint %}

| Aanbieder        | Opmerkingen                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Brede wereldwijde dekking; ondersteunt alle vier de tegelsoorten                                                                                                            |
| **Esri ArcGIS**| Vaak luchtfoto&#x27;s met een hogere resolutie in bepaalde regio&#x27;s. De tegelsoort**Terrain** wordt niet aangeboden voor Esri en de bijbehorende knop is uitgeschakeld zolang Esri is geselecteerd |***

## Soorten kaarttegels

Kies het type kaartlaag met de knoppen (van links naar rechts):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Type                 | Toont                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terrein**          | Hoogte-schaduw met kaartdetails (wegen, labels). Alleen Google       |
| **Kaart**              | Standaard straatkaarttegels — de optie met de laagste bandbreedte              |
| **Satelliet**        | Gedetailleerde satellietbeelden, geen labels — de optie met de hoogste bandbreedte |
| **Hybride** (standaard) | Satellietbeelden met daaroverheen getekende wegen en labels                |

Het tabblad Kaart wordt geopend op **Hybride**. Uw keuze wordt doorgevoerd bij het wisselen van aanbieder, voor zover de aanbieder dit ondersteunt.***

## Navigatie op de kaart

* **Zoomen**: het scrollwieltje van de muis of de zoomknoppen op de kaart
* **Pannen**: klikken en slepen
* **Volledig scherm**: met de knop voor volledig scherm wordt de kaart over het hele venster weergegeven***

## Toepassingen

### Vliegpad beoordelen

* Bekijk in één oogopslag het dekkingsgebied van een dronesessie
* Spoor hiaten op waar een passage is overgeslagen
* Controleer of de vlucht het geplande patroon heeft gevolgd

### Beoordeling van grondmetingen

* Bekijk hoe de opnames vanaf de grond zijn verdeeld
* Lokaliseer kalibratiedoelkaders ten opzichte van het meetgebied
* Bepaal waar aanvullende opnames nodig zijn

### Kwaliteitscontrole

* Vind beelden die op een onverwachte plek zijn vastgelegd en verwijder deze vóór de verwerking
* Sorteer op hoogte om een beeld te herkennen dat op de verkeerde hoogte is vastgelegd, of een beeld waarbij de GPS-positie onnauwkeurig was
* Vergelijk de locaties van de beelden met de veldnotities

***

## Probleemoplossing

### Er verschijnen geen markeringen

**Mogelijke oorzaken**

* De foto’s bevatten geen GPS-metadata
* GPS was tijdens het maken van de opname uitgeschakeld op de camera
* EXIF-gegevens zijn door andere software verwijderd vóór het importeren

**Wat te doen**: controleer of GPS op de camera is ingeschakeld en importeer de originele bestanden opnieuw. Je kunt controleren of een specifiek bestand coördinaten bevat door het op te zoeken in de bestandstabel op het tabblad Kaart — een foto zonder coördinaten heeft daar geen rij.

### Markeringen staan op de verkeerde plaats

**Mogelijke oorzaken**: een slechte satellietpositie op het moment van opname, of GPS-afwijking tijdens de sessie.**Wat te doen**: dit is een probleem dat zich tijdens de opname heeft voorgedaan en dat Chloros achteraf niet kan corrigeren. Gebruik voor nauwkeurig werk een PPK/RTK-GPS-workflow — zie de instelling**PPK-correcties toepassen** in [Projectinstellingen](../project-settings/project-settings.md).

### De kaart is leeg of het laden van tegels stopt

De tegelaanbieders zijn online diensten. Als er geen tegels meer binnenkomen, controleer dan de netwerkverbinding van het apparaat en probeer vervolgens van aanbieder te wisselen. Als je sterk had ingezoomd, druk dan op de resetknop **×** om terug te keren naar een normaal zoomniveau en laat de kaart opnieuw tegels opvragen.***

## Gerelateerde pagina’s

* [**Afbeeldingenraster**](image-grid.md) — dezelfde afbeeldingsset als miniaturen
* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) — één afbeelding in detail bekijken
* [**Bestanden aan een project toevoegen**](../processing-images-gui/adding-files-to-a-project.md) — de knoppen voor het toevoegen en verwijderen van bestanden die dit tabblad deelt
