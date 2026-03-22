# Een afbeelding op volledig scherm openen

De Chloros Image Viewer biedt een speciale interface op volledig scherm voor het bekijken, analyseren en bewerken van uw multispectrale afbeeldingen. Of u nu originele afbeeldingen of bewerkte resultaten bekijkt, de Image Viewer biedt krachtige tools voor inspectie en analyse.

## De Image Viewer openen

### Vanuit de bestandsbrowser

De meest gebruikelijke manier om een afbeelding te openen in de Image Viewer:

1. Zorg ervoor dat u zich op het tabblad **Bestandsbrowser** bevindt <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik op een willekeurige **afbeeldingsminiatuur** in het afbeeldingsraster
3. De afbeelding wordt geopend in het **hoofdvoorvertoningsgebied** (midden van het scherm)
4. De afbeelding is nu geladen en klaar om op volledig scherm te bekijken

### Het tabblad Image Viewer openen

Zodra een afbeelding in het voorvertoningsgebied is geladen:

1. Klik op het **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> in de linkerzijbalk
2. Het tabblad Afbeeldingsviewer wordt geopend en toont de geselecteerde afbeelding op volledig scherm
3. Geavanceerde weergave- en analysehulpmiddelen worden beschikbaar in de linkerzijbalk

***

## Overzicht van de interface van de Afbeeldingsviewer

### Hoofdweergavegebied

Het grootste deel van het scherm toont uw afbeelding:

* **Volledige resolutie**: Afbeeldingen worden weergegeven in de oorspronkelijke resolutie
* **Zoombaar**: gebruik de bedieningselementen of het muiswiel om in te zoomen
* **Verschuifbaar**: klik en sleep om te verplaatsen wanneer ingezoomd
* **Beeldverhouding behouden**: afbeeldingen worden proportioneel geschaald***

## Weergaveopties

### Basisnavigatie door afbeeldingen

#### Blader door afbeeldingen

Navigeer door uw afbeeldingsset met behulp van sneltoetsen of knoppen:

* **Volgende afbeelding**: Klik op de →-knop of druk op de**→**-toets (pijl naar rechts)
* **Vorige afbeelding**: Klik op de ←-knop of druk op de**←**-toets (pijl naar links)
* **Ga naar een specifieke afbeelding**: Ga terug naar de bestandsbrowser en klik op de gewenste miniatuur

#### Zoomknoppen

Pas de vergroting aan om afbeeldingsdetails te bekijken:

**Inzoomen:*** Klik op de **+** (plus) knop
* Druk op de **+**of**=** toets
* Scroll met het muiswiel **omhoog**

**Uitzoomen:*** Klik op de **−** (min) knop
* Druk op de **−** (min) toets
* Scroll met het muiswiel **omlaag**

#### Pannen bij ingezoomd

Wanneer u verder hebt ingezoomd dan de schermgrootte:

1. Beweeg de muiscursor over de afbeelding
2. Klik en **houd de linkermuisknop ingedrukt**

3.**Sleep** om de afbeelding te verplaatsen
4. Laat los om het pannen te stoppen

**Alternatief**: Gebruik de pijltjestoetsen om in kleine stapjes te pannen***

## Pixelwaarden bekijken

### Pixelwaarden bij de cursor bekijken

Terwijl u de muiscursor over de afbeelding beweegt, worden de pixelwaarden in realtime weergegeven:**Locatie van de waardeweergave:*** **Zwevend getal en rode lijn in de legenda van de LUT-gradiënt aan de rechterkant*** **Bij verder inzoomen, zwevende waarde bij de cursor en gemarkeerde pixel*** Toont waarden voor de pixel **onder de cursor of gemarkeerd*** Wordt bijgewerkt terwijl u de muis beweegt

***

## Afbeeldingstypen die u kunt bekijken

### JPG

**JPG-afbeeldingen van de camera:**

* Geeft JPG-gegevens weer zoals in het voorbeeld
* Toont originele, ongecorrigeerde waarden
* Handig voor het controleren van de beeldkwaliteit vóór verwerking

### RAW (Origineel)

### RAW (Reflectantie)

**Na verwerking:**

* Vignettering gecorrigeerd
* Reflectantie gekalibreerd
* Multi-band TIFF (Red, Green, NIR, enz.)
* Wetenschappelijke gegevens klaar voor analyse

### RAW (Index)

**NDVI, NDRE, GNDVI, enz. (\_NDVI.tif-bestanden):**

* Grijswaardenbeelden met één band
* Pixelwaarden vertegenwoordigen de resultaten van de indexberekening
* Bereik doorgaans -1 tot +1 voor genormaliseerde indices
* Er kunnen kleur-LUT&#x27;s worden toegepast voor visualisatie

***

## Toepassing van index en LUT

Pas multispectrale indices en kleur-Look-Up Tables toe:

1. Zoek **Index/LUT Sandbox**in**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> zijbalk
2. Selecteer vegetatie-index (NDVI, NDRE, enz.)
3. Selecteer multispectrale formule of maak uw eigen aangepaste formule (alleen Chloros+)
4. Pas kleur-LUT-verloop toe voor visualisatie
5. Pas waardebereiken en drempels aan

Zie [Index/LUT Sandbox](index-lut-sandbox.md) voor gedetailleerde instructies.

***

## Sneltoetsen

### Navigatie

* **→** (Pijl naar rechts): Volgende afbeelding
* **←** (Pijl naar links): Vorige afbeelding
* **Home**: Eerste afbeelding in de lijst
* **End**: Laatste afbeelding in de lijst

### Zoomen

* **+**of**=**: Inzoomen
* **−**: Uitzoomen
* **Muiswiel**: In-/uitzoomen***

### Indexberekeningen controleren

Controleer of de indices correct zijn berekend:

1. Open NDVI of een andere indexafbeelding
2. Controleer de vegetatiegebieden:
   * **NDVI**: Moet 0,4-0,9 aangeven voor gezonde planten
   * **NDRE**: Hogere waarden voor krachtige groei
   * **GNDVI**: Vergelijkbaar met NDVI, maar gevoelig voor chlorofyl
3. Controleer niet-vegetatie:
   * **Bodem**: Bijna 0 of licht negatief
   * **Water**: Negatieve waarden (-0,5 tot 0)***

## Problemen met het bekijken oplossen

### Afbeelding kan niet worden geopend

**Mogelijke oorzaken:**

* Bestand beschadigd tijdens verwerking
* Niet-ondersteund bestandsformaat
* Onvoldoende geheugen voor grote afbeelding

**Oplossingen:**

1. Probeer het bestand in een externe viewer te openen om de integriteit te controleren
2. Controleer of het bestandsformaat overeenkomt met het verwachte type
3. Sluit andere applicaties om geheugen vrij te maken
4. Probeer een kleinere/andere afbeelding

### Afbeelding wordt zwart of wit weergegeven

**Mogelijke oorzaken:**

* Waardebereik buiten weergavemogelijkheden
* 32-bits float-afbeelding met ongebruikelijke waarden
* Fout bij indexberekening

**Oplossingen:**

1. Controleer de pixelwaarden – als deze allemaal erg laag of erg hoog zijn, pas dan het weergavebereik aan
2. Probeer het bestand te openen in QGIS of een soortgelijk programma met automatische bereikaanpassing
3. Controleer het foutlogboek van de verwerking op fouten

### Pixelwaarden lijken onjuist

**Mogelijke oorzaken:**

* Verkeerde afbeelding wordt bekeken (origineel versus verwerkt)
* Kalibratie is niet correct toegepast
* Gegevens van de lichtsensor zijn niet opgenomen in de invoer
* De procentmodus is onjuist ingeschakeld

**Oplossingen:**

1. Controleer of u de verwerkte uitvoer bekijkt (controleer de extensie van de bestandsnaam)
2. Controleer de status van de knop voor de procentmodus
3. Vergelijk met afbeeldingen uit dezelfde dataset waarvan bekend is dat ze goed zijn

***

## Volgende stappen

Nu u afbeeldingen op volledig scherm kunt bekijken:

* [**Afbeeldingslagen**](image-layers.md) - Meer informatie over multibandvisualisatie
* [**Index/LUT-sandbox**](index-lut-sandbox.md) - Pas aangepaste indices en kleurtoewijzingen toe
* [**Multispectrale indexformules**](../project-settings/multispectral-index-formulas.md) - Begrijp de beschikbare indices

Zie voor de verwerkingsworkflow:

* [**Beelden verwerken (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Volledige verwerkingsgids
