# Afbeeldingslagen

Met het vervolgkeuzemenu ‘Afbeeldingslagen’ in de Chloros-afbeeldingsviewer kunt u snel schakelen tussen verschillende versies van dezelfde afbeelding – van de originele opnames tot bewerkte reflectie-uitvoer en berekende indexafbeeldingen.

## Wat zijn afbeeldingslagen?

In Chloros verwijzen **lagen** naar de verschillende afbeeldingsuitvoer die beschikbaar is voor één bronafbeelding. Wanneer u afbeeldingen verwerkt, maakt Chloros meerdere versies aan:

* **Originele afbeeldingen** (JPG- en RAW-bestanden van uw camera)
* **Reflectantie gekalibreerde** uitvoer (als reflectantiecalibratie was ingeschakeld)
* **Doelafbeeldingen** (als de afbeelding kalibratiedoelen bevat)
* **Indexafbeeldingen** (NDVI, NDRE, GNDVI, enz. als indexen waren geconfigureerd)

Met de **Layer Selector-dropdown** rechtsboven in de Image Viewer kunt u direct tussen deze versies schakelen zonder de viewer te verlaten.***

## Beschikbare laagtypen

### JPG

* De originele JPG-voorbeeldafbeelding van uw camera
* Altijd beschikbaar voor alle afbeeldingen
* Onbewerkt, zoals vastgelegd door de camera
* Het snelst te laden en weer te geven

**Wanneer te bekijken:**

* Snel voorbeeld van de originele opname
* Controleren van de compositie en kadrering van de afbeelding
* Controleren van de opnamekwaliteit vóór verwerking

### RAW (Origineel)

* De originele RAW-sensorgegevens van uw camera
* Debayered zonder dat er nabewerking is toegepast
* Hogere bitdiepte dan JPG (meestal 12-bits of 14-bits sensorgegevens)

**Wanneer te bekijken:**

* De kwaliteit van de originele sensorgegevens inspecteren
* Controleren op sensorproblemen of artefacten
* Resultaten voor en na verwerking vergelijken

### RAW (Doel)

* Verschijnt alleen bij afbeeldingen waarvan is vastgesteld dat ze kalibratiedoelen bevatten
* Toont de originele RAW-afbeelding met gedetecteerd doel
* Wordt gebruikt om te verifiëren of de doelherkenning succesvol was

**Wanneer te bekijken:**

* Bevestigen dat kalibratiedoelen correct zijn gedetecteerd
* De beeldkwaliteit van het doel controleren
* Problemen met kalibratie oplossen

{% hint style="info" %}
**Doellaag**: Deze laag verschijnt alleen in de vervolgkeuzelijst voor afbeeldingen die kalibratiedoelen bevatten. Gewone opnameafbeeldingen hebben deze optie niet.
{% endhint %}

### RAW (Reflectantie)

* De gekalibreerde reflectantie-uitvoerbeeld
* Vignettering gecorrigeerd (indien ingeschakeld tijdens verwerking)
* Reflectantie gekalibreerd met behulp van doelgegevens (indien ingeschakeld)
* Multiband TIFF met alle camerakanaal
* Pixelwaarden vertegenwoordigen procentuele reflectantie (bij gebruik van de procentuele modus)
* Klaar om te bewerken met de [Index/LUT Sandbox](index-lut-sandbox.md)

**Wanneer te bekijken:**

* Het inspecteren van gekalibreerde resultaten
* Het controleren van de kalibratiekwaliteit
* Het controleren van pixelwaarden op wetenschappelijke nauwkeurigheid
* Het vergelijken met het origineel om kalibratie-effecten te zien

{% hint style="success" %}
**Aanbevolen**: Gebruik de RAW (Reflectie)-laag bij het controleren van pixelwaarden voor wetenschappelijke metingen en analyses.
{% endhint %}

### RAW (NDVI Index)... en soortgelijke

* Berekende vegetatie-indexafbeelding (NDVI in dit voorbeeld)
* De indexnaam verandert afhankelijk van welke index tijdens de verwerking is geconfigureerd
* Voorbeelden: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index), enz.
* Grijswaardenafbeelding met één band die de resultaten van de indexberekening weergeeft
* Er verschijnt één laag voor elke index die is geconfigureerd in de projectinstellingen

**Mogelijke indexnamen:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI-index)
* RAW (EVI-index)
* RAW (SAVI-index)
* En nog veel meer... (zie [Formules voor multispectrale indexen](../project-settings/multispectral-index-formulas.md))

**Wanneer te bekijken:**

* Het onderzoeken van de resultaten van indexberekeningen
* Het controleren van indexwaardebereiken
* Het identificeren van gebieden van belang
* Het verifiëren van indexbeelden voordat deze worden gebruikt in GIS of analyse

***

## De laagselector gebruiken

### De vervolgkeuzelijst openen

1. Open een afbeelding in volledig scherm (klik op een willekeurige miniatuur in de afbeeldingsviewer)
2. Zoek de **laag-dropdown** in de rechterbovenhoek van de viewer
3. De dropdown toont de momenteel geselecteerde laag (bijv. &quot;JPG&quot;)
4. Klik op de dropdown om alle beschikbare lagen te zien

### Van laag wisselen

1. Klik op de laag-dropdown om de lijst te openen
2. Alle beschikbare lagen voor de huidige afbeelding worden getoond
3. Klik op een willekeurige laagnaam om naar die versie te wisselen
4. De afbeelding wordt onmiddellijk bijgewerkt om de geselecteerde laag weer te geven

**Snel wisselen:**

* Het dropdown-menu onthoudt uw laatste selectie
* Wanneer u naar de volgende afbeelding navigeert, probeert Chloros hetzelfde laagtype weer te geven
* Als die laag niet bestaat op de volgende afbeelding, wordt standaard JPG weergegeven

### Beschikbaarheid van lagen

Niet alle lagen zijn beschikbaar voor elke afbeelding:

**Altijd beschikbaar:*** ✅ JPG (elke afbeelding heeft een JPG-voorbeeld)

**Voorwaardelijk beschikbaar:**

* ⚠️ RAW (Origineel) - Alleen als de afbeelding is vastgelegd in de RAW- of RAW+JPG-modus
* ⚠️ RAW (Doel) - Alleen als de afbeelding gedetecteerde kalibratiedoelen bevat
* ⚠️ RAW (Reflectantie) - Alleen na verwerking met reflectantiekalibratie ingeschakeld
* ⚠️ RAW (\[Index] Index) - Alleen na verwerking met geconfigureerde indices

***

## Laagpersistentie

### Navigeren tussen afbeeldingen

Wanneer u naar een andere afbeelding navigeert (met de pijltjestoetsen of door op miniaturen te klikken):**De laagvoorkeur blijft behouden:**

* Als u &quot;RAW (Reflectantie)&quot; bekijkt, toont de volgende afbeelding &quot;RAW (Reflectantie)&quot; (indien beschikbaar)
* Als u &quot;RAW (NDVI Index)&quot; bekijkt, toont de volgende afbeelding &quot;RAW (NDVI Index)&quot; (indien beschikbaar)
* Als dezelfde laag niet bestaat, wordt standaard JPG weergegeven

**Voorbeeldworkflow:**

1. Open afbeelding 1, schakel over naar RAW (NDVI Index)
2. Druk op → om afbeelding 2 te bekijken
3. Afbeelding 2 toont automatisch de RAW (NDVI Index) laag
4. Ga verder met navigeren - alle afbeeldingen tonen de NDVI-laag
5. Zeer efficiënt voor het bekijken van indexresultaten over vele afbeeldingen

***

## Veelvoorkomende workflows

### Workflow 1: Voor/na-vergelijking

**Doel**: Vergelijk de originele afbeelding met de gekalibreerde afbeelding

1. Open de bewerkte afbeelding in de afbeeldingsviewer
2. Selecteer **RAW (Origineel)** in de vervolgkeuzelijst
3. Let op de vignettering en de niet-gekalibreerde waarden
4. Schakel over naar **RAW (Reflectantie)** in de vervolgkeuzelijst
5. Vergelijk - vignettering verwijderd, waarden gekalibreerd

### Werkstroom 2: Indexbeoordeling

**Doel**: Snel de NDVI-resultaten in de dataset bekijken

1. Open de eerste bewerkte afbeelding
2. Selecteer **RAW (NDVI Index)** in het dropdown-menu
3. Gebruik de pijltoetsen om naar de volgende afbeelding te gaan
4. De NDVI-laag blijft automatisch staan
5. Ga door alle afbeeldingen heen en controleer de NDVI-patronen
6. Schakel over naar **RAW (NDRE Index)** om te vergelijken

### Workflow 3: Doelverificatie

**Doel**: Controleer of alle doelafbeeldingen correct zijn gedetecteerd

1. Navigeer naar een doelafbeelding
2. Selecteer **RAW (Target)** in de vervolgkeuzelijst
3. Controleer of de kalibratiedoelen duidelijk zichtbaar zijn en gedetecteerd worden
4. Navigeer naar de volgende doelafbeelding
5. Herhaal de verificatie voor alle doelen

### Workflow 4: Controle van pixelwaarden

**Doel**: Controleer de reflectiewaarden op wetenschappelijke nauwkeurigheid

1. Open de bewerkte afbeelding
2. Selecteer de laag **RAW (Reflectance)**

3. Schakel de modus**Pixel Percent** in (knop in de werkbalk rechtsboven)
4. Beweeg de cursor over vegetatiegebieden
5. Controleer of de pixelwaarden binnen de verwachte bereiken liggen (30-70% voor NIR, 5-15% voor Red)
6. Controleer bodem- en watergebieden op juiste waarden

***

## Inzicht in pixelwaarden per laag

Verschillende lagen tonen verschillende pixelwaarde-bereiken:

### JPG-laag

* **Bereik**: 0-255 (8-bit)
* **Betekenis**: Weergavewaarden, gammagecorrigeerd
* **Gebruik**: Alleen visuele inspectie, niet voor wetenschappelijke metingen

### RAW (Origineel)

* **Bereik**: 0-65535 (16-bit)
* **Betekenis**: Ruwe digitale sensorgegevens
* **Gebruik**: Controle van sensorprestaties, niet gekalibreerd

### RAW (Reflectantie)

* **Bereik**: 0-65.535 (16-bits TIFF) of 0,0-1,0 (32-bits Percent)
* **Betekenis**: Gekalibreerd reflectiepercentage
* **Gebruik**: Wetenschappelijke metingen en analyse**Voor 16-bits TIFF:**Deel door 65.535 om het reflectiepercentage te verkrijgen**Voor 32-bits Percent:** Waarden geven direct het percentage weer (0,5 = 50% reflectie)

### RAW (indexbeelden)

* **Bereik**: Varieert per index (meestal -1,0 tot +1,0 voor genormaliseerde indices)
* **Betekenis**: Resultaat van de indexberekening
* **Voorbeelden**:
  * NDVI: -1 tot +1 (vegetatie meestal 0,4 tot 0,9)
  * NDRE: -1 tot +1 (stressdetectie)
  * EVI: 0 tot 1 (verbeterde vegetatie)

***

## Tips en best practices

### Efficiënt wisselen tussen lagen

* **Toetsenbord sneltoetsen**: Hoewel er geen sneltoetsen zijn voor lagen, werken de navigatiepijltjes (←/→) voor alle lagen
* **Consistente workflows**: Kies één laag (bijv. NDVI) en bekijk de volledige dataset voordat u naar een andere laag overschakelt
* **Snelle vergelijkingen**: Schakel tussen Original en Reflectance om de verwerkingskwaliteit te controleren

### Overwegingen met betrekking tot prestaties

* **JPG laadt het snelst**: Gebruik dit voor snelle navigatie door veel afbeeldingen
* **RAW-lagen laden langzamer**: Hogere resolutie en bitdiepte
* **Indexlagen**: Vergelijkbare snelheid als Reflectance-lagen
* **De eerste keer laden is het langzaamst**: Daaropvolgende weergaven van dezelfde laag worden in de cache opgeslagen en zijn sneller

### Kwaliteitscontrole

* **Controleer altijd RAW (Origineel)**: Controleer de kwaliteit van de brongegevens voordat u de verwerkte resultaten vertrouwt
* **Vergelijk lagen**: Gebruik het wisselen tussen lagen om te controleren of de verwerking correct is verlopen
* **Controleer indexbereiken**: Gebruik de modus Pixelpercentage met indexlagen om te controleren of de waarden redelijk zijn***

## Probleemoplossing

### Laag niet beschikbaar

**Probleem**: Verwachte laag verschijnt niet in de vervolgkeuzelijst**Mogelijke oorzaken:**

* Afbeelding is niet verwerkt (alleen JPG en RAW (origineel) beschikbaar)
* Reflectantiecalibratie was uitgeschakeld tijdens de verwerking
* Specifieke index was niet geconfigureerd in de projectinstellingen
* Afbeelding is een afbeelding die alleen voor doelen is bedoeld (er worden geen indices gegenereerd voor doelen)

**Oplossingen:**

1. Controleer of de afbeelding is verwerkt (kijk in de uitvoermap of er verwerkte bestanden staan)
2. Controleer de projectinstellingen om te zien of de indices zijn geconfigureerd
3. Verwerk opnieuw met de gewenste indices ingeschakeld

### Verkeerde laag weergegeven

**Probleem**: Afbeelding wordt geopend in een onverwachte laag**Oorzaak**: De laagvoorkeur van de vorige afbeelding is overgenomen, maar die laag bestaat niet in de huidige afbeelding**Oplossing**: Chloros schakelt automatisch over naar JPG wanneer de voorkeurslaag niet beschikbaar is – dit is normaal gedrag

### Kalibratiedoelen niet zichtbaar

**Probleem**: RAW-laag (Target) toont geen doelherkenning**Mogelijke oorzaken:**

* Doelen werden niet gedetecteerd tijdens de verwerking
* Afbeelding bevat eigenlijk geen doelen
* Instellingen voor doelherkenning zijn te streng

**Oplossingen:**

1. Controleer het foutlogboek op berichten met &quot;Target found&quot;
2. Controleer of de afbeelding daadwerkelijk zichtbare kalibratiedoelen bevat
3. Pas de instellingen voor doelherkenning aan in Projectinstellingen
4. Zie [Doelafbeeldingen kiezen](../processing-images-gui/choosing-target-images.md)

***

## Gerelateerde functies

### Hulpmiddelen voor de afbeeldingsviewer

Bij het bekijken van een laag kunt u gebruikmaken van:

* **Zoomknoppen**: Vergroot om details te bekijken
* **Pannen**: Klik en sleep om door de ingezoomde afbeelding te navigeren
* **Pixelwaarde-inspectie**: Bekijk waarden op de cursorlocatie
* **Navigatiepijlen**: Schakel tussen afbeeldingen terwijl de laag behouden blijft
* **Pixelpercentage-modus**: Schakel tussen DN- en procentweergave

Zie [Een afbeelding op volledig scherm openen](opening-an-image-full-screen.md) voor de volledige documentatie van de afbeeldingsviewer.

### Index/LUT-sandbox

Voor interactieve index-tests en visualisatie:

* **Realtime indexberekening**: Test verschillende indexformules
* **LUT-kleurtoewijzing**: Pas kleurverlopen toe op grijswaardenindexen
* **Visualisaties exporteren**: Sla gekleurde indexafbeeldingen op

Zie [Index/LUT-sandbox](index-lut-sandbox.md) voor meer informatie.

***

## Volgende stappen

Nu u beeldlagen begrijpt:

* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) - Volledige handleiding voor de afbeeldingsviewer
* [**Index/LUT-sandbox**](index-lut-sandbox.md) - Interactieve indexvisualisatie
* [**Multispectrale indexformules**](../project-settings/multispectral-index-formulas.md) - Referentie van beschikbare indices
* [**De verwerking afronden**](../processing-images-gui/finishing-the-processing.md) - Inzicht in verwerkte uitvoer
