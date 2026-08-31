# Bestanden aan een project toevoegen

Zodra je een project hebt aangemaakt of geopend in Chloros, is de volgende stap het toevoegen van je multispectrale beelden om met de verwerking te beginnen. Via het tabblad ‘Bestandsbrowser’ <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> kun je eenvoudig beelden importeren en je dataset beheren.

## De bestandsbrowser openen

1. Open of maak een project aan in Chloros
2. Klik op het pictogram **Bestandsbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> in de linkerzijbalk
3. Het paneel Bestandsbrowser toont de bestandslijst van uw project

{% hint style="info" %}
**Ondersteunde bestandstypen**:

* **Survey3W / Survey3N**: RAW+JPG-paren en JPG-afbeeldingen (RAW+JPG aanbevolen)
* **LATTICE**: `.tif` / `.tiff`-opnames — vastgelegd door de Chloros-camerabesturing of door een LATTICE-hub
* **Lichtsensorgegevens**: `.daq`-opnames (DAQ-U/M/E) en DAQ-M `.csv`-downwelling-logs — geïmporteerd samen met beeldmateriaal om de reflectiekalibratie te sturen
{% endhint %}

***

## Beelden toevoegen aan uw project

Er zijn twee belangrijke manieren om beelden aan uw project toe te voegen:

### Methode 1: Bestanden toevoegen

Gebruik deze optie om afzonderlijke beeldbestanden of een kleine selectie van bestanden te importeren.

1. Klik op de knop **&quot;Bestanden toevoegen&quot;** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> bovenaan het paneel Bestandsbrowser
2. Navigeer naar de map met uw afbeeldingen
3. Selecteer één of meer afbeeldingsbestanden (houd **Ctrl** ingedrukt om meerdere bestanden te selecteren)
4. Klik op **&quot;Openen&quot;** om de geselecteerde bestanden te importeren

### Methode 2: Map toevoegen

Gebruik deze optie om alle afbeeldingen uit een map in één keer te importeren. U kunt **meerdere mappen** in één dialoogvenster selecteren.

1. Klik op de **&quot;Map toevoegen&quot;**-<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">-knop bovenaan het paneel Bestandsbrowser
2. Navigeer naar de map(pen) met de afbeeldingen van uw opnamesessie en selecteer deze
3. Klik op **&quot;Map selecteren&quot;** om alle ondersteunde afbeeldingen te importeren

{% hint style="info" %}
**Bestanden die niet kunnen worden geladen, worden gemeld.** Als een map bestanden bevat die Chloros herkent maar niet kan laden, krijg je een waarschuwing — afbeeldingen verdwijnen niet stilletjes uit het raster.
{% endhint %}

***

## LATTICE-opnamemappen importeren

LATTICE-opnames worden opgeslagen met **één submap per exportniveau** — bijvoorbeeld `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — met het bijbehorende `.daq`-downwelling-bestand in de hoofdmap:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Stel de map in op de hoofdmap van de opnames** (`output/` hierboven). Wanneer de geselecteerde map zelf geen afbeeldingen bevat maar wel submappen heeft, gaat Chloros daar automatisch in — de submappen op dat niveau en de hoofdmap `.daq` worden allemaal in één keer opgehaald.**Hoe opnames worden geïmporteerd:*** Elke opname wordt geïmporteerd als een **enkele afbeelding**, gegroepeerd per opname (niet één item per niveau). De andere niveaus van dezelfde opname verschijnen als weergavemodi van die ene afbeelding.
* **De verwerking begint altijd bij het onbewerkte frame.** De andere niveaus zijn weliswaar zichtbaar, maar alleen `raw` wordt door de verwerkingspijplijn geleid — het opnieuw verwerken van een reeds verwerkt product zou leiden tot dubbele correcties, dus Chloros wordt afgewezen. Een opnieuw geïmporteerde export kan nooit de raw-plaats van een opname innemen.
* Een opnamemap die **zonder** raw-bestanden is opgeslagen, wordt normaal weergegeven, maar de verwerking slaat deze over en vermeldt dit in het logboek. (De vlag CLI `--input-level` kan in dit geval een startpunt afdwingen — zie [de CLI-referentie](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**LATTICE-hubsessies** worden op dezelfde manier geïmporteerd: kies bij ‘Map toevoegen’ de sessiemap die je vanuit de hub hebt gekopieerd (deze bevat `raw/` plus `previews/`), samen met eventuele DAQ-M `.csv` downwelling-logbestanden. Als de kalibratie van de camera of DAQ nog niet in de cache van je computer is opgeslagen, haalt Chloros deze bij het importeren automatisch op aan de hand van het serienummer (hiervoor is eenmalig internetverbinding nodig).***

## De tabel in de bestandsbrowser begrijpen

Zodra de beelden zijn geïmporteerd, verschijnen ze in een tabel met de volgende kolommen:

### Bestandsnaam

* Oorspronkelijke bestandsnaam van de camera
* Behoudt de naamgevingsconventie van de camera (bijv. IMG\_0001.RAW of capture\_20260816\_101500\_SN213800234\_raw.tif)

### Tijdstempel

* Datum en tijdstip waarop de afbeelding is vastgelegd
* Ontleend aan de EXIF-metadata van de afbeelding
* Wordt gebruikt voor het afstemmen van lichtsensoren, PPK-synchronisatie en het plannen van kalibratiedoelen

### Cameramodel

* Automatisch gedetecteerde camera- en filterconfiguratie
* Voorbeelden van Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE-voorbeelden: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Wordt gebruikt om de juiste verwerkingsprofielen toe te passen

### Doelkolom (selectievakje)

* Vink dit vakje aan voor afbeeldingen die kalibratiedoelen bevatten
* Wanneer ten minste één afbeelding is aangevinkt, worden **alleen de aangevinkte afbeeldingen gescand** op doelen
* Zie [Doelafbeeldingen kiezen](choosing-target-images.md) voor meer informatie

### Metagegevens van afbeeldingen bekijken

Als u op de schakelknop in de rechterbovenhoek boven de tabel klikt, worden de metagegevens van de geselecteerde afbeelding weergegeven in het afbeeldingsrastergebied.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Bestanden van lichtsensoren in uw project

* De bestanden `.daq` en `.csv` verschijnen in de lijst van de bestandsbrowser, maar zijn geen aanklikbare afbeeldingen — ze leveren de neerwaartse stralingsintensiteit voor reflectiekalibratie.
* Elk geïmporteerd `.daq`/`.csv`-bestand wordt weergegeven onder **Projectinstellingen → DAQ-lichtsensor**, waar u de geldende correctie voor de diffusorkap per bestand kunt bekijken. Zie [Projectinstellingen aanpassen](adjusting-project-settings.md).
* Opnames die u maakt op het tabblad **Lichtsensoren** worden automatisch toegevoegd aan het geopende project — handmatig importeren is niet nodig.***

## Bestanden beheren in uw project

### Bestanden verwijderen

Om ongewenste afbeeldingen uit uw project te verwijderen:

1. Selecteer een of meer afbeeldingen in de tabel van de bestandsbrowser
2. Klik op de knop **&quot;Geselecteerde verwijderen&quot;** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Bevestig het verwijderen (bestanden worden niet van de schijf gewist, maar alleen uit het project verwijderd)

### Sorteren en filteren

* **Sorteren op kolom**: klik op een kolomkop om afbeeldingen te sorteren
* **Sorteren op tijdstempel**: handig voor het ordenen van chronologische opnamesequenties
* **Filter op cameramodel**: groepeer afbeeldingen op cameratype als u meerdere camera’s gebruikt***

## Afbeeldingsvoorbeeld

### Volledige afbeelding bekijken

Klik op een willekeurige miniatuurafbeelding in de bestandsbrowser om deze in het hoofdvoorbeeldvenster weer te geven:

1. De afbeelding verschijnt in het centrale voorbeeldvenster
2. Gebruik de zoomknoppen om de details van de afbeelding te bekijken
3. Navigeer tussen afbeeldingen met de pijltjestoetsen

### Snelle navigatie

* **Vorige afbeelding**: Klik op de pijl naar links of druk op de ←-toets
* **Volgende afbeelding**: Klik op de pijl naar rechts of druk op de →-toets
* **In-/uitzoomen**: Gebruik het muiswiel of de zoomknoppen
* **Pannen**: Klik en sleep over de afbeelding wanneer je hebt ingezoomd***

## Omgaan met dubbele bestanden

Chloros detecteert automatisch dubbele bestanden en negeert deze:

* Bestanden met identieke bestandsnamen worden overgeslagen
* Voorkomt onbedoelde dubbele verwerking
* Er wordt een waarschuwingsbericht weergegeven wanneer dubbele bestanden worden gedetecteerd

{% hint style="warning" %}
**Belangrijk**: Hernoem of wijzig uw originele afbeeldingsbestanden niet vóór het importeren. Chloros is voor een correcte verwerking afhankelijk van de originele bestandsnamen en metagegevens.
{% endhint %}

***

## Gemengde cameradatasets

Als je project afbeeldingen bevat van meerdere MAPIR-camera’s:

1. Chloros detecteert automatisch elk cameramodel — Survey3, LATTICE of een combinatie daarvan
2. Elk cameratype wordt verwerkt met het bijbehorende kalibratieprofiel
3. De bestandsbrowser geeft het cameramodel weer in de kolom ‘Cameramodel’
4. Elke camera krijgt bij verwerking een eigen structuur van uitvoermappen

**Voorbeeldscenario’s**: Survey3W RGN + Survey3N OCN opstelling met twee camera’s, of een LATTICE-array met een RGB-master en meerdere smalbandmodules***

## Aanbevolen werkwijzen

### Organiseer vóór het importeren

* Bewaar kalibratiedoelafbeeldingen in dezelfde map als de opnameafbeeldingen
* Bewaar de `.daq` / `.csv`-lichtsensorbestanden van elke opnamesessie bij de beelden van die sessie
* Behoud de oorspronkelijke mappenstructuur van je camera/SD-kaart/hub
* Meng geen datasets van verschillende sessies in één project

### Bestandsnaamgeving

* Behoud de oorspronkelijke camerabestandsnamen (IMG\_0001.RAW, capture\_..., enz.)
* Hernoem bestanden niet vóór het importeren
* Oorspronkelijke namen bevatten belangrijke metagegevens

### Afbeeldingen van kalibratiedoelen

* Voeg altijd 1-2 afbeeldingen van kalibratiedoelen per sessie toe (Survey3; voor LATTICE kan een DAQ-opname als vervanging dienen — zie [Kalibratiedoelen kiezen](choosing-target-images.md))
* Maak opnamedoelen voor en na de opnamesessie
* Plaats de doelen onder dezelfde lichtomstandigheden als het opnamegebied
* Markeer doelafbeeldingen met het selectievakje ‘Doel’

***

## Veelvoorkomende problemen en oplossingen

### Afbeeldingen verschijnen niet na het importeren

**Mogelijke oorzaken:**

* Bestandsformaat wordt niet ondersteund (zie de lijst met ondersteunde typen bovenaan deze pagina)
* Afbeeldingen zijn afkomstig van camera’s die niet tot de serie MAPIR behoren (zie [Ondersteunde camera’s](../supported-cameras.md))
* Beschadigd bestand of onvolledige overdracht vanaf de SD-kaart

**Oplossing**: Controleer of het bestandsformaat en het cameramodel compatibel zijn, en bekijk de waarschuwing bij het laden van bestanden om te zien welke bestanden precies zijn mislukt

### Cameramodel niet gedetecteerd

**Mogelijke oorzaken:**

* Gewijzigde EXIF-metadata
* Afbeeldingen bewerkt in externe software
* Onvolledige bestandsoverdracht

**Oplossing**: Importeer de originele, ongewijzigde bestanden opnieuw vanuit de camera/SD-kaart

### Ontbrekende tijdstempels

**Mogelijke oorzaken:**

* De klok van de camera is niet correct ingesteld
* EXIF-gegevens zijn verwijderd door externe software

**Oplossing**: Controleer of de tijdinstellingen van de camera correct waren tijdens het maken van de opnamen

### Bij het opnieuw openen van het project worden ontbrekende bestanden gemeld

Als bronbestanden zijn verplaatst of verwijderd sinds het project voor het laatst werd geopend, geeft Chloros aan **welke** bestanden ontbreken, in plaats van een leeg raster te openen. Herstel de bestanden op hun oorspronkelijke paden, of verwijder de ontbrekende vermeldingen en importeer ze opnieuw.***

## Volgende stappen

Zodra je bestanden zijn geïmporteerd:

1. **Controleer de bestandslijst** - Zorg ervoor dat alle afbeeldingen correct zijn geladen
2. **Controleer de cameramodellen** - Controleer of de camera&#x27;s correct zijn gedetecteerd
3. **Markeer doelafbeeldingen** - Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
4. **Pas instellingen aan** - Configureer verwerkingsopties in [Projectinstellingen](adjusting-project-settings.md)
5. **Start de verwerking** - Zie [De verwerking starten](starting-the-processing.md)

Raadpleeg [Projectinstellingen aanpassen](adjusting-project-settings.md) voor gedetailleerde informatie over de projectconfiguratie.
