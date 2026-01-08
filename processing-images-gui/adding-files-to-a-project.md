# Bestanden toevoegen aan een project

Nadat u een project hebt gemaakt of geopend in Chloros, is de volgende stap het toevoegen van uw multispectrale beelden om te beginnen met de verwerking. Met het tabblad Bestandsbrowser<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> maakt het eenvoudig om beelden te importeren en uw dataset te beheren.

## Toegang tot de bestandsbrowser

1. Open of maak een project in Chloros
2. Klik op het pictogram **Bestandsbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> in de linkerzijbalk
3. Het paneel Bestandsbrowser geeft de bestandslijst van uw project weer

{% hint style="info" %}
**Ondersteunde bestandstypen**: Chloros ondersteunt RAW+JPG- en JPG-afbeeldingsbestanden van MAPIR Survey3W en Survey3N camera&#x27;s. Alleen RAW+JPG wordt aanbevolen.
{% endhint %}

***

## Afbeeldingen toevoegen aan uw project

Er zijn twee manieren om afbeeldingen toe te voegen aan uw project:

### Methode 1: Bestanden toevoegen

Gebruik deze optie om afzonderlijke afbeeldingsbestanden of een kleine selectie van bestanden te importeren.

1. Klik op de knop **&quot;Bestanden toevoegen&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> bovenaan het paneel Bestandsbrowser
2. Navigeer naar de map met uw afbeeldingen
3. Selecteer een of meer afbeeldingsbestanden (houd **Ctrl** ingedrukt om meerdere bestanden te selecteren)
4. Klik op **&quot;Openen&quot;** om de geselecteerde bestanden te importeren

### Methode 2: Map toevoegen

Gebruik deze optie om alle afbeeldingen uit een map in één keer te importeren.

1. Klik op de knop **&quot;Map toevoegen&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> bovenaan het paneel Bestandsbrowser.
2. Navigeer naar de map met de afbeeldingen van uw opnamesessie en selecteer deze.
3. Klik op **&quot;Map selecteren&quot;** om alle ondersteunde afbeeldingen uit die map te importeren.

***

## De tabel van de bestandsbrowser begrijpen

Zodra de afbeeldingen zijn geïmporteerd, worden ze weergegeven in een tabel met de volgende kolommen:

### Bestandsnaam

* Originele bestandsnaam van de camera
* Handhaaft de naamgevingsconventie van de camera (bijv. IMG\_0001.RAW)

### Tijdstempel

* Datum en tijd waarop de afbeelding is gemaakt
* Ontleend aan de EXIF-metadata van de afbeelding
* Gebruikt voor PPK-synchronisatie en detectie van kalibratiedoelen

### Cameramodel

* Automatisch gedetecteerde camera- en filterconfiguratie
* Voorbeelden: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Gebruikt om de juiste verwerkingsprofielen toe te passen

### Doelkolom (selectievakje)

* Vink dit vakje aan voor afbeeldingen die kalibratiedoelen bevatten
* Versnelt de doelherkenning tijdens de verwerking aanzienlijk
* Zie [Doelafbeeldingen kiezen](choosing-target-images.md) voor meer informatie

### Afbeeldingsmetadata bekijken

Als u op de schakelknop rechtsboven de tabel klikt, worden de metagegevens van de geselecteerde afbeelding weergegeven in het afbeeldingsrastergebied.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Bestanden in uw project beheren

### Bestanden verwijderen

Om ongewenste afbeeldingen uit uw project te verwijderen:

1. Selecteer een of meer afbeeldingen in de tabel Bestandsbrowser
2. Klik op de knop **&quot;Geselecteerde verwijderen&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Bevestig het verwijderen (bestanden worden niet van de schijf verwijderd, maar alleen uit het project verwijderd).

### Sorteren en filteren

* **Sorteren op kolom**: klik op een kolomkop om afbeeldingen te sorteren.
* **Sorteren op tijdstempel**: handig voor het ordenen van chronologische opnamereeksen.
* **Filter op cameramodel**: groepeer afbeeldingen op cameratype als u meerdere camera&#x27;s gebruikt.

***

## Afbeeldingsvoorbeeld

### Volledige afbeelding bekijken

Klik op een afbeeldingsminiatuur in de bestandsbrowser om deze weer te geven in het hoofdvoorbeeldgebied:

1. De afbeelding verschijnt in het voorbeeldpaneel in het midden
2. Gebruik de zoomknoppen om de afbeelding in detail te bekijken
3. Navigeer tussen afbeeldingen met de pijltjestoetsen

### Snelle navigatie

* **Vorige afbeelding**: klik op de pijl naar links of druk op de toets ←
* **Volgende afbeelding**: klik op de pijl naar rechts of druk op de toets →
* **In-/uitzoomen**: gebruik het muiswiel of de zoomknoppen
* **Pannen**: klik en sleep op de afbeelding wanneer u hebt ingezoomd

***

## Omgaan met dubbele bestanden

Chloros detecteert automatisch dubbele bestanden en negeert deze:

* Bestanden met identieke bestandsnamen worden overgeslagen
* Voorkomt onbedoelde dubbele verwerking
* Er wordt een waarschuwingsbericht weergegeven wanneer duplicaten worden gedetecteerd

{% hint style="warning" %}
**Belangrijk**: Hernoem of wijzig uw originele afbeeldingsbestanden niet voordat u ze importeert. Chloros is afhankelijk van originele bestandsnamen en metagegevens voor een correcte verwerking.
{% endhint %}

***

## Gemengde cameragegevenssets

Als uw project afbeeldingen van meerdere MAPIR-camera&#x27;s bevat:

1. Chloros detecteert automatisch elk cameramodel
2. Elk cameratype wordt verwerkt met het juiste kalibratieprofiel
3. De bestandsbrowser geeft het cameramodel weer in de kolom Cameramodel
4. De verwerking past de juiste instellingen toe voor elk cameratype

**Voorbeeldscenario**: Survey3W RGN + Survey3N OCN dubbele camera-opstelling

***

## Best practices

### Organiseren vóór het importeren

* Bewaar kalibratiedoelafbeeldingen in dezelfde map als de onderzoeksafbeeldingen
* Behoud de oorspronkelijke mappenstructuur van uw camera/SD-kaart
* Meng geen datasets van verschillende sessies in één project

### Bestandsnaamgeving

* Behoud de oorspronkelijke bestandsnamen van de camera (IMG\_0001.RAW, enz.)
* Hernoem bestanden niet vóór het importeren
* Originele namen bevatten belangrijke metagegevens.

### Kalibratiedoelafbeeldingen

* Voeg altijd 1-2 kalibratiedoelafbeeldingen per sessie toe.
* Leg doelen vast vóór en na de opnamesessie.
* Plaats doelen in dezelfde lichtomstandigheden als het opnamegebied.
* Markeer doelafbeeldingen met het selectievakje Doel om de verwerking te versnellen.

***

## Veelvoorkomende problemen en oplossingen

### Afbeeldingen worden niet weergegeven na het importeren

**Mogelijke oorzaken:**

* Bestandsformaat wordt niet ondersteund (alleen RAW+JPG en JPG van MAPIR-camera&#x27;s)
* Afbeeldingen zijn afkomstig van niet-MAPIR-camera&#x27;s (zie [Ondersteunde camera&#x27;s](../supported-cameras.md))
* Bestand beschadigd of onvolledige overdracht vanaf SD-kaart

**Oplossing**: Controleer de compatibiliteit van het bestandsformaat en het cameramodel.

### Cameramodel niet gedetecteerd

**Mogelijke oorzaken:**

* Gewijzigde EXIF-metadata
* Afbeeldingen bewerkt in externe software
* Onvolledige bestandsoverdracht

**Oplossing**: Importeer de originele, ongewijzigde bestanden opnieuw vanaf de camera/SD-kaart.

### Ontbrekende tijdstempels

**Mogelijke oorzaken:**

* Cameraklok niet correct ingesteld
* EXIF-gegevens verwijderd door externe software

**Oplossing**: Controleer of de tijdinstellingen van de camera correct waren tijdens het vastleggen

***

## Volgende stappen

Zodra uw bestanden zijn geïmporteerd:

1. **Controleer de bestandslijst** - Zorg ervoor dat alle afbeeldingen correct zijn geladen
2. **Controleer de cameramodellen** - Controleer of de camera correct is gedetecteerd
3. **Markeer doelafbeeldingen** - Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
4. **Pas de instellingen aan** - Configureer de verwerkingsopties in [Projectinstellingen](adjusting-project-settings.md)
5. **Start de verwerking** - Zie [De verwerking starten](starting-the-processing.md)

Voor gedetailleerde informatie over projectconfiguratie, zie [Projectinstellingen aanpassen](adjusting-project-settings.md).
