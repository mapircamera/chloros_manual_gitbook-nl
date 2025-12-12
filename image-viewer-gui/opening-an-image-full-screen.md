# Een afbeelding op volledig scherm openen

De Chloros Image Viewer biedt een speciale interface op volledig scherm voor het bekijken, analyseren en bewerken van uw multispectrale afbeeldingen. Of u nu originele afbeeldingen of bewerkte uitvoer bekijkt, de Image Viewer biedt krachtige tools voor inspectie en analyse.

## Toegang tot de Image Viewer

### Vanuit de bestandsbrowser

De meest gebruikelijke manier om een afbeelding te openen in de Image Viewer:

1. Zorg ervoor dat u zich in het tabblad **Bestandsbrowser** bevindt <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik op een **afbeeldingsminiatuur** in het afbeeldingenraster
3. De afbeelding wordt geopend in het **hoofdvoorvertoningsgebied** (midden van het scherm)
4. De afbeelding is nu geladen en klaar om op volledig scherm te worden bekeken

### Het tabblad Image Viewer openen

Zodra een afbeelding in het voorvertoningsgebied is geladen:

1. Klik op het pictogram **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> in de linkerzijbalk
2. Het tabblad Afbeeldingsviewer wordt geopend en de geselecteerde afbeelding wordt op volledig scherm weergegeven
3. Geavanceerde weergave- en analysetools worden beschikbaar in de linkerzijbalk

***

## Overzicht van de interface van de afbeeldingsviewer

### Hoofdweergavegebied

Het grootste deel van het scherm toont uw afbeelding:

* **Volledige resolutie**: afbeeldingen worden weergegeven op de oorspronkelijke resolutie
* **Zoombaar**: gebruik de bedieningselementen of het muiswiel om in te zoomen
* **Pannen**: klik en sleep om te bewegen wanneer u inzoomt
* **Beeldverhouding behouden**: afbeeldingen worden proportioneel geschaald

***

## Weergaveopties

### Basisnavigatie door afbeeldingen

#### Door afbeeldingen bladeren

Navigeer door uw afbeeldingsset met behulp van sneltoetsen of knoppen:

* **Volgende afbeelding**: klik op de knop → of druk op de toets **→** (pijl naar rechts)
* **Vorige afbeelding**: klik op de knop ← of druk op de toets **←** (pijl naar links)
* **Naar een specifieke afbeelding springen**: ga terug naar de bestandsbrowser en klik op de gewenste miniatuur

#### Zoomknoppen

Pas de vergroting aan om afbeeldingsdetails te bekijken:

**Inzoomen:**

* Klik op de knop **+** (plus)
* Druk op de toets **+** of **=**
* Scroll met het muiswiel **omhoog**

**Uitzoomen:**

* Klik op de knop **−** (min)
* Druk op de toets **−** (min)
* Scroll met het muiswiel **omlaag**

**Aanpassen aan scherm:**

* Klik op de knop **↔** (Aanpassen)
* Druk op de toets **0** (Nul)
* Dubbelklik op de afbeelding

#### Pannen bij zoomen

Wanneer u verder inzoomt dan de schermgrootte:

1. Beweeg de muiscursor over de afbeelding
2. Klik en **houd de linkermuisknop ingedrukt**
3. **Sleep** om de afbeelding te verplaatsen
4. Laat los om het pannen te stoppen

**Alternatief**: gebruik de pijltjestoetsen om in kleine stapjes te pannen

***

## Pixelwaarde inspecteren

### Pixelwaarden bij de cursor bekijken

Wanneer u de muiscursor over de afbeelding beweegt, worden de pixelwaarden in realtime weergegeven:

**Locatie van de waardeweergave:**

* **Drijvend getal en rode lijn in de index LUT-gradiëntlegenda aan de rechterkant**
* **Bij verder inzoomen, drijvende waarde nabij de cursor en gemarkeerde pixel**
* Toont waarden voor pixel **onder de cursor of gemarkeerd**
* Wordt bijgewerkt terwijl u de muis beweegt

***

## Beeldtypes die u kunt bekijken

### Originele beelden (voorafgaand aan verwerking)

**RAW + JPG-afbeeldingen van camera:**

* Geeft RAW-gegevens weer zoals in het voorbeeld
* Toont originele, ongewijzigde waarden
* Handig om de beeldkwaliteit te controleren vóór verwerking

### Gekalibreerde reflectieafbeeldingen

**Na verwerking:**

* Vignette gecorrigeerd
* Reflectie gekalibreerd
* Multi-band TIFF (Red, Green, NIR, enz.)
* Wetenschappelijke gegevens klaar voor analyse

### Indexafbeeldingen

**NDVI, NDRE, GNDVI, enz. (\_NDVI.tif-bestanden):**

* Enkelvoudige grijswaardenafbeeldingen
* Pixelwaarden vertegenwoordigen indexberekeningsresultaten
* Bereik doorgaans -1 tot +1 voor genormaliseerde indices
* Kleuren-LUT&#x27;s kunnen worden toegepast voor visualisatie

***

## Index- en LUT-toepassing

Multispectrale indices en kleuren-LUT&#x27;s toepassen:

1. Zoek **Index/LUT Sandbox** in de **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> zijbalk
2. Selecteer vegetatie-index (NDVI, NDRE, enz.)
3. Selecteer multispectrale formule of maak uw eigen aangepaste formule (alleen Chloros+)
4. Pas kleur-LUT-gradiënt toe voor visualisatie
5. Pas waardebereiken en drempels aan

Zie [Index/LUT Sandbox](index-lut-sandbox.md) voor gedetailleerde instructies.

***

## Sneltoetsen

### Navigatie

* **→** (pijl naar rechts): volgende afbeelding
* **←** (pijl naar links): vorige afbeelding
* **Home**: eerste afbeelding in de lijst
* **End**: Laatste afbeelding in de lijst

### Zoomen

* **+** of **=**: Inzoomen
* **−**: Uitzoomen
* **0** (nul): Aanpassen aan scherm
* **Muiswiel**: In-/uitzoomen

### Weergavebediening

* **P**: Pixelprocentmodus in-/uitschakelen
* **L**: Lagenpaneel in-/uitschakelen
* **Esc**: Volledig scherm sluiten of terugkeren naar bestandsbrowser

### Overig

* **Ctrl+S**: Huidige afbeelding opslaan
* **F**: Volledig scherm (indien beschikbaar)

***

### Indexberekeningen controleren

Controleer of de indexen correct zijn berekend:

1. Open NDVI of een andere indexafbeelding
2. Controleer de vegetatiegebieden:
   * **NDVI**: Moet 0,4-0,9 weergeven voor gezonde planten
   * **NDRE**: hogere waarden voor krachtige groei
   * **GNDVI**: vergelijkbaar met NDVI, maar chlorofylgevoelig
3. Controleer niet-vegetatie:
   * **Bodem**: Bijna 0 of licht negatief
   * **Water**: Negatieve waarden (-0,5 tot 0)

***

## Problemen met weergeven oplossen

### Afbeelding kan niet worden geopend

**Mogelijke oorzaken:**

* Bestand beschadigd tijdens verwerking
* Niet-ondersteund bestandsformaat
* Onvoldoende geheugen voor grote afbeelding

**Oplossingen:**

1. Probeer het bestand te openen in een externe viewer om de integriteit van het bestand te controleren
2. Controleer of het bestandsformaat overeenkomt met het verwachte type
3. Sluit andere toepassingen om geheugen vrij te maken
4. Probeer een kleiner/ander beeld

### Zwart-wit beeldweergave

**Mogelijke oorzaken:**

* Waardebereik buiten de weergavemogelijkheden
* 32-bits float-beeld met ongebruikelijke waarden
* Fout bij indexberekening

**Oplossingen:**

1. Controleer de pixelwaarden. Als deze allemaal erg laag of erg hoog zijn, pas dan het weergavebereik aan.
2. Probeer het bestand te openen in QGIS of een soortgelijk programma met automatische bereikaanpassing.
3. Controleer het foutopsporingslogboek van de verwerking op fouten.

### Pixelwaarden lijken onjuist

**Mogelijke oorzaken:**

* Verkeerde afbeelding weergeven (origineel versus verwerkt)
* Kalibratie is niet correct toegepast
* Lichtsensorgegevens zijn niet opgenomen in de invoer
* Percentagemodus is onjuist ingesteld

**Oplossingen:**

1. Controleer of u de verwerkte uitvoer bekijkt (controleer de bestandsextensie)
2. Controleer de status van de knop voor de percentagemodus
3. Vergelijk met bekende goede afbeeldingen uit dezelfde dataset

***

## Volgende stappen

Nu u afbeeldingen op volledig scherm kunt bekijken:

* [**Afbeeldingslagen**](image-layers.md) - Meer informatie over multibandvisualisatie
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Pas aangepaste indexen en kleurovergangen toe
* [**Multispectrale indexformules**](../project-settings/multispectral-index-formulas.md) - Begrijp de beschikbare indexen

Voor de verwerkingsworkflow, zie:

* [**Afbeeldingen verwerken (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Volledige verwerkingsgids
