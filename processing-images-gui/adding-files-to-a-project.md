# Bestanden aan een project toevoegen

Zodra je een project hebt aangemaakt of geopend in Chloros, is de volgende stap het toevoegen van je multispectrale beelden om met de verwerking te beginnen. Via het tabblad Bestandsbrowser<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> maakt het eenvoudig om beelden te importeren en uw dataset te beheren.

## Toegang tot de bestandsbrowser

1. Open of maak een project aan in Chloros
2. Klik op het **Bestandsbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> in de linkerzijbalk
3. Het paneel Bestandsbrowser toont de bestandslijst van uw project

{% hint style="info" %}
**Ondersteunde bestandstypen**: Chloros ondersteunt RAW+JPG- en JPG-afbeeldingsbestanden van MAPIR, Survey3W en Survey3N-camera&#x27;s. Alleen RAW+JPG wordt aanbevolen.
{% endhint %}

***

## Afbeeldingen toevoegen aan uw project

Er zijn twee manieren om afbeeldingen aan uw project toe te voegen:

### Methode 1: Bestanden toevoegen

Gebruik deze optie om afzonderlijke afbeeldingsbestanden of een kleine selectie van bestanden te importeren.

1. Klik op de knop **&quot;Bestanden toevoegen&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> bovenaan het bestandsbrowservenster
2. Navigeer naar de map met uw afbeeldingen
3. Selecteer een of meer afbeeldingsbestanden (houd **Ctrl** ingedrukt om meerdere bestanden te selecteren)
4. Klik op **&quot;Openen&quot;** om de geselecteerde bestanden te importeren

### Methode 2: Map toevoegen

Gebruik deze optie om alle afbeeldingen uit een map in één keer te importeren.

1. Klik op de knop **&quot;Map toevoegen&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> bovenaan het paneel Bestandsbrowser
2. Navigeer naar de map met de afbeeldingen van uw opnamesessie en selecteer deze
3. Klik op **&quot;Map selecteren&quot;** om alle ondersteunde afbeeldingen uit die map te importeren***

## De tabel in de Bestandsbrowser begrijpen

Zodra afbeeldingen zijn geïmporteerd, verschijnen ze in een tabel met de volgende kolommen:

### Bestandsnaam

* Originele bestandsnaam van de camera
* Behoudt de naamgevingsconventie van de camera (bijv. IMG\_0001.RAW)

### Tijdstempel

* Datum en tijd waarop de afbeelding is vastgelegd
* Ontleend aan de EXIF-metadata van de afbeelding
* Gebruikt voor PPK-synchronisatie en detectie van kalibratiedoelen

### Cameramodel

* Automatisch gedetecteerde camera- en filterconfiguratie
* Voorbeelden: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Wordt gebruikt om de juiste verwerkingsprofielen toe te passen

### Doelkolom (selectievakje)

* Vink dit vakje aan voor afbeeldingen die kalibratiedoelen bevatten
* Versnelt de doelherkenning aanzienlijk tijdens de verwerking
* Zie [Doelafbeeldingen kiezen](choosing-target-images.md) voor meer informatie

### Afbeeldingsmetadata bekijken

Als u op de schakelknop in de rechterbovenhoek boven de tabel klikt, worden de metadata van de geselecteerde afbeelding weergegeven in het afbeeldingsrastergebied.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Bestanden in uw project beheren

### Bestanden verwijderen

Om ongewenste afbeeldingen uit uw project te verwijderen:

1. Selecteer een of meer afbeeldingen in de tabel Bestandsbrowser
2. Klik op de knop **&quot;Geselecteerde verwijderen&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> knop
3. Bevestig het verwijderen (bestanden worden niet van de schijf verwijderd, maar alleen uit het project)

### Sorteren en filteren

* **Sorteren op kolom**: Klik op een kolomkop om afbeeldingen te sorteren
* **Sorteren op tijdstempel**: Handig voor het ordenen van chronologische opnamesequenties
* **Filter op cameramodel**: Groepeer afbeeldingen op cameratype als u meerdere camera&#x27;s gebruikt***

## Afbeeldingsvoorbeeld

### Volledige afbeelding bekijken

Klik op een willekeurige miniatuurafbeelding in de bestandsbrowser om deze in het hoofdvoorbeeldgebied weer te geven:

1. De afbeelding verschijnt in het centrale voorbeeldvenster
2. Gebruik de zoomknoppen om de details van de afbeelding te bekijken
3. Navigeer tussen afbeeldingen met de pijltjestoetsen

### Snelle navigatie

* **Vorige afbeelding**: Klik op de pijl naar links of druk op de ←-toets
* **Volgende afbeelding**: Klik op de pijl naar rechts of druk op de →-toets
* **In-/uitzoomen**: Gebruik het muiswiel of de zoomknoppen
* **Pannen**: Klik en sleep op de afbeelding wanneer u hebt ingezoomd***

## Omgaan met dubbele bestanden

Chloros detecteert automatisch dubbele bestanden en negeert deze:

* Bestanden met identieke bestandsnamen worden overgeslagen
* Voorkomt onbedoelde dubbele verwerking
* Er wordt een waarschuwingsbericht weergegeven wanneer duplicaten worden gedetecteerd

{% hint style="warning" %}
**Belangrijk**: Hernoem of wijzig uw originele afbeeldingsbestanden niet voordat u ze importeert. Chloros is afhankelijk van de originele bestandsnamen en metagegevens voor een correcte verwerking.
{% endhint %}

***

## Gemengde cameradatasets

Als uw project afbeeldingen bevat van meerdere MAPIR-camera&#x27;s:

1. Chloros detecteert automatisch elk cameramodel
2. Elk cameratype wordt verwerkt met het juiste kalibratieprofiel
3. De bestandsbrowser geeft het cameramodel weer in de kolom &#x27;Cameramodel&#x27;
4. Bij de verwerking worden de juiste instellingen voor elk cameratype toegepast

**Voorbeeldscenario**: Survey3W RGN + Survey3N OCN opstelling met twee camera&#x27;s***

## Best practices

### Organiseer vóór het importeren

* Bewaar kalibratiedoelafbeeldingen in dezelfde map als de survey-afbeeldingen
* Behoud de oorspronkelijke mapstructuur van uw camera/SD-kaart
* Meng geen datasets van verschillende sessies in één project

### Bestandsnaamgeving

* Behoud de oorspronkelijke camerabestandsnamen (IMG\_0001.RAW, enz.)
* Hernoem bestanden niet vóór het importeren
* Originele namen bevatten belangrijke metadata

### Kalibratiedoelafbeeldingen

* Neem altijd 1-2 kalibratiedoelafbeeldingen per sessie op
* Leg doelen vast voor en na de opnamesessie
* Plaats doelen in dezelfde lichtomstandigheden als het opnamegebied
* Markeer doelafbeeldingen met het selectievakje &#x27;Doel&#x27; om de verwerking te versnellen

***

## Veelvoorkomende problemen en oplossingen

### Afbeeldingen verschijnen niet na het importeren

**Mogelijke oorzaken:**

* Bestandsformaat wordt niet ondersteund (alleen RAW+JPG en JPG van MAPIR-camera&#x27;s)
* Afbeeldingen zijn afkomstig van niet-MAPIR-camera&#x27;s (zie [Ondersteunde camera&#x27;s](../supported-cameras.md))
* Beschadigd bestand of onvolledige overdracht vanaf SD-kaart

**Oplossing**: Controleer of het bestandsformaat en het cameramodel compatibel zijn

### Cameramodel niet gedetecteerd

**Mogelijke oorzaken:**

* Gewijzigde EXIF-metadata
* Afbeeldingen bewerkt in externe software
* Onvolledige bestandsoverdracht

**Oplossing**: Importeer opnieuw de originele, ongewijzigde bestanden vanaf de camera/SD-kaart

### Ontbrekende tijdstempels

**Mogelijke oorzaken:**

* Klok van de camera niet correct ingesteld
* EXIF-gegevens verwijderd door externe software

**Oplossing**: Controleer of de tijdinstellingen van de camera correct waren tijdens het vastleggen***

## Volgende stappen

Zodra uw bestanden zijn geïmporteerd:

1. **Controleer de bestandslijst** - Zorg ervoor dat alle afbeeldingen correct zijn geladen
2. **Controleer de cameramodellen** - Controleer of de camera&#x27;s correct zijn gedetecteerd
3. **Markeer de doelafbeeldingen** - Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
4. **Pas de instellingen aan** - Configureer de verwerkingsopties in [Projectinstellingen](adjusting-project-settings.md)
5. **Start de verwerking** - Zie [De verwerking starten](starting-the-processing.md)

Zie [Projectinstellingen aanpassen](adjusting-project-settings.md) voor gedetailleerde informatie over projectconfiguratie.
