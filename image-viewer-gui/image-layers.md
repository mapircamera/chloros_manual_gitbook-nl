# Beeldlagen

Met de vervolgkeuzelijst Beeldlagen in de Chloros-beeldviewer kunt u snel schakelen tussen verschillende versies van hetzelfde beeld, van de originele opnames tot bewerkte reflectie-uitvoer en berekende indexbeelden.

## Wat zijn beeldlagen?

In Chloros verwijzen **lagen** naar de verschillende afbeeldingsuitvoer die beschikbaar is voor één bronafbeelding. Wanneer u afbeeldingen verwerkt, maakt Chloros meerdere versies:

* **Originele afbeeldingen** (JPG- en RAW-bestanden van uw camera)
* **Reflectantie-gekalibreerde** uitvoer (als reflectantiekalibratie was ingeschakeld)
* **Doelafbeeldingen** (als de afbeelding kalibratiedoelen bevat)
* **Indexafbeeldingen** (NDVI, NDRE, GNDVI, enz. als indexen waren geconfigureerd)

Met de **Layer Selector-dropdown** rechtsboven in de Image Viewer kunt u direct tussen deze versies schakelen zonder de viewer te verlaten.

***

## Beschikbare laagtypen

### JPG

* De originele JPG-previewafbeelding van uw camera
* Altijd beschikbaar voor alle afbeeldingen
* Onbewerkt, zoals vastgelegd door de camera
* Snelst te laden en weer te geven

**Wanneer bekijken:**

* Snel voorbeeld van originele opname
* Beeldcompositie en kadrering controleren
* Opnamekwaliteit controleren vóór verwerking

### RAW (origineel)

* De originele RAW-sensorgegevens van uw camera
* Debayered zonder nabewerking
* Hogere bitdiepte dan JPG (meestal 12-bits of 14-bits sensorgegevens)

**Wanneer bekijken:**

* De kwaliteit van de originele sensorgegevens inspecteren
* Controleren op sensorproblemen of artefacten
* De resultaten vóór en na verwerking vergelijken

### RAW (doel)

* Wordt alleen weergegeven voor afbeeldingen waarvan is vastgesteld dat ze kalibratiedoelen bevatten
* Toont de originele RAW-afbeelding met gedetecteerd doel
* Wordt gebruikt om te controleren of de doeldetectie succesvol was

**Wanneer bekijken:**

* Om te controleren of kalibratiedoelen correct zijn gedetecteerd
* Om de beeldkwaliteit van het doel te controleren
* Om kalibratieproblemen op te lossen

{% hint style=&quot;info&quot; %}
**Doellaag**: deze laag verschijnt alleen in de vervolgkeuzelijst voor afbeeldingen die kalibratiedoelen bevatten. Normale opnamebeelden hebben deze optie niet.
{% endhint %}

### RAW (reflectie)

* Het gekalibreerde reflectie-uitvoerbeeld
* Vignette gecorrigeerd (indien ingeschakeld tijdens verwerking)
* Reflectie gekalibreerd met behulp van doeldata (indien ingeschakeld)
* Multiband TIFF met alle camerakanalen
* Pixelwaarden vertegenwoordigen procentuele reflectie (bij gebruik van de procentmodus)
* Klaar om te bewerken met de [Index/LUT Sandbox](index-lut-sandbox.md)

**Wanneer bekijken:**

* Gekalibreerde resultaten inspecteren
* Kalibratiekwaliteit controleren
* Pixelwaarden controleren op wetenschappelijke nauwkeurigheid
* Vergelijken met origineel om kalibratie-effecten te zien

{% hint style=&quot;success&quot; %}
**Aanbevolen**: Gebruik de RAW-laag (reflectie) bij het controleren van pixelwaarden voor wetenschappelijke metingen en analyses.
{% endhint %}

### RAW (NDVI Index)... en vergelijkbaar

* Berekende vegetatie-indexafbeelding (NDVI in dit voorbeeld)
* De indexnaam verandert op basis van welke index tijdens de verwerking is geconfigureerd
* Voorbeelden: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index), enz.
* Enkelvoudige grijswaardenafbeelding met de resultaten van de indexberekening
* Er verschijnt één laag voor elke index die is geconfigureerd in Projectinstellingen

**Mogelijke indexnamen:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI-index)
* RAW (EVI-index)
* RAW (SAVI-index)
* En nog veel meer... (zie [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md))

**Wanneer bekijken:**

* Bij het onderzoeken van indexberekeningsresultaten
* Bij het controleren van indexwaardebereiken
* Bij het identificeren van interessante gebieden
* Bij het verifiëren van indexafbeeldingen voordat deze worden gebruikt in GIS of analyses

***

## De laagselector gebruiken

### De vervolgkeuzelijst openen

1. Open een afbeelding in de modus Volledig scherm (klik op een willekeurige miniatuur in de afbeeldingsviewer)
2. Zoek de **laagdropdown** in de rechterbovenhoek van de viewer
3. De dropdown toont de momenteel geselecteerde laag (bijv. &quot;JPG&quot;)
4. Klik op de dropdown om alle beschikbare lagen te zien

### Van laag wisselen

1. Klik op de laagdropdown om de lijst te openen
2. Alle beschikbare lagen voor de huidige afbeelding worden weergegeven
3. Klik op een laagnaam om naar die versie over te schakelen.
4. De afbeelding wordt onmiddellijk bijgewerkt om de geselecteerde laag weer te geven.

**Snel overschakelen:**

* De vervolgkeuzelijst onthoudt uw laatste selectie.
* Wanneer u naar de volgende afbeelding navigeert, probeert Chloros hetzelfde laagtype weer te geven.
* Als die laag niet op de volgende afbeelding bestaat, wordt standaard JPG weergegeven.

### Beschikbaarheid van lagen

Niet alle lagen zijn beschikbaar voor elke afbeelding:

**Altijd beschikbaar:**

* ✅ JPG (elke afbeelding heeft een JPG-voorbeeld)

**Voorwaardelijk beschikbaar:**

* ⚠️ RAW (origineel) - Alleen als de afbeelding is vastgelegd in RAW- of RAW+JPG-modus
* ⚠️ RAW (doel) - Alleen als de afbeelding gedetecteerde kalibratiedoelen bevat
* ⚠️ RAW (reflectie) - Alleen na verwerking met reflectiekalibratie ingeschakeld
* ⚠️ RAW (\[Index] Index) - Alleen na verwerking met geconfigureerde indexen

***

## Persistentie van lagen

### Navigeren tussen afbeeldingen

Wanneer u naar een andere afbeelding navigeert (met de pijltjestoetsen of door op miniaturen te klikken):

**De laagvoorkeur blijft behouden:**

* Als u &quot;RAW (reflectie)&quot; bekijkt, wordt de volgende afbeelding weergegeven als &quot;RAW (reflectie)&quot; (indien beschikbaar)
* Als u &quot;RAW (NDVI Index)&quot; bekijkt, wordt de volgende afbeelding weergegeven als &quot;RAW (NDVI Index)&quot; (indien beschikbaar)
* Als dezelfde laag niet bestaat, wordt standaard JPG weergegeven.

**Voorbeeld van een workflow:**

1. Open afbeelding 1, schakel over naar RAW (NDVI Index).
2. Druk op → om afbeelding 2 te bekijken.
3. Afbeelding 2 geeft automatisch de laag RAW (NDVI Index) weer.
4. Ga verder met navigeren - alle afbeeldingen tonen de NDVI-laag
5. Zeer efficiënt voor het bekijken van indexresultaten voor veel afbeeldingen

***

## Veelvoorkomende workflows

### Workflow 1: Vergelijking voor/na

**Doel**: Vergelijk de originele afbeelding met de gekalibreerde afbeelding

1. Open de verwerkte afbeelding in Image Viewer
2. Selecteer **RAW (origineel)** in de vervolgkeuzelijst
3. Let op de vignettering en niet-gekalibreerde waarden
4. Schakel over naar **RAW (reflectie)** in de vervolgkeuzelijst
5. Vergelijk - vignettering verwijderd, waarden gekalibreerd

### Workflow 2: Index bekijken

**Doel**: snel NDVI-resultaten in de dataset bekijken

1. Open de eerste bewerkte afbeelding
2. Selecteer **RAW (NDVI Index)** in de vervolgkeuzelijst
3. Gebruik de → pijltjestoets om naar de volgende afbeelding te gaan
4. De NDVI-laag blijft automatisch bestaan
5. Ga door met alle afbeeldingen en controleer de NDVI-patronen
6. Schakel over naar **RAW (NDRE Index)** om te vergelijken

### Workflow 3: Doelverificatie

**Doel**: Controleer of alle doelafbeeldingen correct zijn gedetecteerd

1. Navigeer naar een doelafbeelding
2. Selecteer **RAW (Target)** in de vervolgkeuzelijst
3. Controleer of de kalibratiedoelen duidelijk zichtbaar zijn en worden gedetecteerd
4. Navigeer naar de volgende doelafbeelding
5. Herhaal de verificatie voor alle doelen

### Workflow 4: Pixelwaarde-inspectie

**Doel**: Controleer de reflectiewaarden op wetenschappelijke nauwkeurigheid

1. Open de verwerkte afbeelding
2. Selecteer de laag **RAW (Reflectance)**
3. Schakel de modus **Pixel Percent** in (knop in de werkbalk rechtsboven)
4. Beweeg de cursor over vegetatiegebieden
5. Controleer of de pixelwaarden binnen de verwachte bereiken liggen (30-70% voor NIR, 5-15% voor Red)
6. Controleer of de waarden voor bodem- en watergebieden correct zijn

***

## Pixelwaarden per laag begrijpen

Verschillende lagen tonen verschillende pixelwaardebereiken:

### JPG-laag

* **Bereik**: 0-255 (8-bit)
* **Betekenis**: weergavewaarden, gammacorrectie toegepast
* **Gebruik**: alleen visuele inspectie, niet voor wetenschappelijke metingen

### RAW (origineel)

* **Bereik**: 0-65535 (16-bit)
* **Betekenis**: Ruwe digitale sensorgegevens
* **Gebruik**: Controle van sensorprestaties, niet gekalibreerd

### RAW (reflectie)

* **Bereik**: 0-65.535 (16-bits TIFF) of 0,0-1,0 (32-bits procent)
* **Betekenis**: Gekalibreerde procentuele reflectie
* **Gebruik**: Wetenschappelijke metingen en analyse

**Voor 16-bits TIFF:** Deel door 65.535 om de procentuele reflectie te verkrijgen **Voor 32-bits procent:** Waarden geven direct het percentage weer (0,5 = 50% reflectie)

### RAW (indexafbeeldingen)

* **Bereik**: varieert per index (doorgaans -1,0 tot +1,0 voor genormaliseerde indices)
* **Betekenis**: resultaat van indexberekening
* **Voorbeelden**:
  * NDVI: -1 tot +1 (vegetatie doorgaans 0,4 tot 0,9)
  * NDRE: -1 tot +1 (stressdetectie)
  * EVI: 0 tot 1 (verbeterde vegetatie)

***

## Tips en best practices

### Efficiënt wisselen tussen lagen

* **Kennis van sneltoetsen**: hoewel er geen sneltoetsen voor lagen zijn, werken de navigatiepijltjes (←/→) voor alle lagen.
* **Consistente workflows**: kies één laag (bijv. NDVI) en bekijk de volledige dataset voordat u naar een andere laag overschakelt.
* **Snelle vergelijkingen**: schakel tussen Origineel en Reflectie om de verwerkingskwaliteit te controleren.

### Overwegingen met betrekking tot prestaties

* **JPG laadt het snelst**: gebruik dit voor snelle navigatie door veel afbeeldingen.
* **RAW-lagen laden langzamer**: hogere resolutie en bitdiepte.
* **Indexlagen**: vergelijkbare snelheid als reflectielagen.
* **De eerste keer laden duurt het langst**: volgende weergaven van dezelfde laag worden in de cache opgeslagen en zijn sneller.

### Kwaliteitscontrole

* **Controleer altijd RAW (origineel)**: controleer de kwaliteit van de brongegevens voordat u vertrouwt op de verwerkte uitvoer
* **Vergelijk lagen**: gebruik laagwisseling om te controleren of de verwerking correct heeft gewerkt
* **Controleer indexbereiken**: gebruik de modus Pixel Percent met indexlagen om te controleren of de waarden redelijk zijn

***

## Probleemoplossing

### Laag niet beschikbaar

**Probleem**: De verwachte laag verschijnt niet in de vervolgkeuzelijst

**Mogelijke oorzaken:**

* Afbeelding is niet verwerkt (alleen JPG en RAW (origineel) beschikbaar)
* Reflectiekalibratie was uitgeschakeld tijdens de verwerking
* Specifieke index was niet geconfigureerd in Projectinstellingen
* Afbeelding is een afbeelding met alleen doelen (geen indices gegenereerd voor doelen)

**Oplossingen:**

1. Controleer of de afbeelding is verwerkt (controleer de uitvoermap op verwerkte bestanden)
2. Controleer de projectinstellingen om te bevestigen dat de indexen zijn geconfigureerd
3. Verwerk opnieuw met de gewenste indexen ingeschakeld

### Verkeerde laag weergegeven

**Probleem**: Afbeelding wordt geopend in een onverwachte laag

**Oorzaak**: Laagvoorkeur van vorige afbeelding is overgenomen, maar die laag bestaat niet in de huidige afbeelding

**Oplossing:** Chloros valt automatisch terug op JPG wanneer de voorkeurslaag niet beschikbaar is. Dit is normaal gedrag.

### Kalibratiedoelen niet zichtbaar

**Probleem:** RAW-laag (doel) geeft geen doel detectie weer.

**Mogelijke oorzaken:**

* Doelen zijn niet gedetecteerd tijdens de verwerking
* Afbeelding bevat geen doelen
* Instellingen voor doel detectie zijn te streng

**Oplossingen:**

1. Controleer het debuglogboek op berichten met &quot;Doel gevonden&quot;
2. Controleer of de afbeelding daadwerkelijk zichtbare kalibratiedoelen bevat
3. Pas de instellingen voor doel detectie aan in Projectinstellingen
4. Zie [Doelafbeeldingen kiezen](../processing-images-gui/choosing-target-images.md)

***

## Gerelateerde functies

### Tools voor afbeeldingsviewer

Bij het bekijken van een laag kunt u het volgende gebruiken:

* **Zoomknoppen**: vergroten om details te bekijken
* **Pannen**: klikken en slepen om door de ingezoomde afbeelding te bewegen
* **Pixelwaarde bekijken**: waarden bekijken op de locatie van de cursor
* **Navigatiepijlen**: schakel tussen afbeeldingen terwijl u de laag behoudt
* **Pixelprocentmodus**: schakel tussen DN- en procentweergave

Zie [Een afbeelding op volledig scherm openen](opening-an-image-full-screen.md) voor de volledige documentatie over de afbeeldingsviewer.

### Index/LUT Sandbox

Voor interactieve index-tests en visualisatie:

* **Realtime indexberekening**: test verschillende indexformules
* **LUT-kleurtoewijzing**: pas kleurovergangen toe op grijswaardenindexen
* **Visualisaties exporteren**: sla gekleurde indexafbeeldingen op

Zie [Index/LUT Sandbox](index-lut-sandbox.md) voor meer informatie.

***

## Volgende stappen

Nu u beeldlagen begrijpt:

* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) - Volledige handleiding voor Image Viewer
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Interactieve indexvisualisatie
* [**Multispectrale indexformules**](../project-settings/multispectral-index-formulas.md) - Referentie van beschikbare indices
* [**De verwerking voltooien**](../processing-images-gui/finishing-the-processing.md) - Inzicht in verwerkte outputs
