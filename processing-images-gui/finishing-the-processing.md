# De verwerking afronden

Zodra Chloros de verwerking heeft voltooid, is het tijd om je resultaten te bekijken, de uitvoerkwaliteit te controleren en je verwerkte afbeeldingen klaar te maken voor gebruik in je workflow. Op deze pagina word je door de laatste stappen en de volgende acties geleid.

## Indicatie dat de verwerking is voltooid

Wanneer de verwerking succesvol is afgerond, zie je verschillende indicatoren:

* ✅ **Voortgangsbalk**: bereikt 100% voltooiing
* ✅ **Debuglog**: toont de laatste `[RUN-SUMMARY]`-regels met tellingen (afbeeldingen, cameragroepen, doelen, gekalibreerde afbeeldingen, geschreven bestanden)
* ✅ **Startknop**: Wordt weer actief (klaar voor de volgende verwerkingsrun)
* ✅ **Uitvoerbestanden**: Alle verwerkte afbeeldingen zijn opgeslagen in de uitvoerstructuur van het project (hieronder)

{% hint style="warning" %}
**Een verwerkingsrun waarbij geen beelden worden opgeslagen, is mislukt.** Als u beeldproducten hebt aangevraagd en de run geen beelden heeft geschreven, rapporteert Chloros dit als een mislukking — de `[RUN-SUMMARY]`-vermeldingen in de lognaam geven de waarschijnlijke oorzaak aan (niets geïmporteerd, geen doel gedetecteerd, of elk aangevraagd product overgeslagen omdat het niet van toepassing was). Het CLI-equivalent sluit af met een waarde anders dan nul. Een opzettelijke run met alleen metadata (alle exportproducten uitgeschakeld, geen indexen) is nog steeds geslaagd. Zie [de CLI-referentie](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Je verwerkte beelden vinden

### De uitvoermap openen

1. Klik op het **Hoofdmenu**-pictogram <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(linksboven)
2. Selecteer **"Projectmap openen"**

3. Uw bestandsverkenner opent de projectmap
4. Zoek uw project op naam

### De uitvoerstructuur

Producten worden **in de projectmap opgeslagen, gegroepeerd per camera en vervolgens per bestandsformaat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Cameramap**: `LATT-<sensor>-<lens>-F<filter>` voor LATTICE (overeenkomend met de EXIF-gegevens van de opname: `Model`), `<model>_<filter>` voor Survey3 (bijv. `Survey3N_RGN`). Twee camera’s die dezelfde sensor en hetzelfde filter delen, maar een andere lens hebben, krijgen aparte boomstructuren — vignettering, beeldveld en vervorming verschillen.
* **Formaatmap**: volgt je instelling voor het exportformaat — `tiff16`, `tiff8`, `png8`, `jpg8`, of `tiff32` voor TIFF (32-bits, procent). Radiance is altijd float32 en komt altijd terecht onder `tiff32`.
* **Productmappen**:
  * `Reflectance_Calibrated_Images/` — gekalibreerde reflectantie
  * `Debayered_Images/` — lineair gedebayerd (LATTICE): donkere offset verwijderd en gedemosaiced, zonder flat field of unmix, in de oorspronkelijke oriëntatie van de sensor
  * `Preview_Images/` — weergavevoorbeeld (LATTICE)
  * `Radiance_Images/` — spectrale stralingsintensiteit in float32, W/m²/sr/nm (LATTICE multispectraal)
  * `Vignette_Corrected_Images/` **of** `Sensor_Response_Images/` — de ongekalibreerde fallback voor frames zonder reflectantiereferentie; per run bestaat precies één van de twee, gekozen door de instelling voor vignettcorrectie
  * `<INDEX>_Index_Images/` — één map per geselecteerde index (bijv. `NDVI_Index_Images`)
* **`clamp_masks/`** (nieuw in 1.3.0, alleen voor LATTICE multispectrale M3C-camera’s): verschijnt in een formaatmap, naast de productmappen, wanneer de bandontmenging negatieve waarden in een stralings- of reflectantieproduct op nul afsnijdt. Deze map bevat één masker per pixel per betrokken product, bijv. `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Maskers bevinden zich nooit in een productmap, zodat een productmap ongewijzigd aan fotogrammetriesoftware kan worden doorgegeven. Zie [Clamp-maskers](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Elk geëxporteerd product behoudt de naam van het BRON-bestand.**Een radiance-export van `capture_..._raw.tif` heet nog steeds `capture_..._raw.tif` — het staat alleen in `tiff32/Radiance_Images/`.**De map identificeert het product, niet de bestandsnaam**, dus als je zoekt naar `*radiance*.tif` vind je niets; zoek in plaats daarvan op de mapnaam.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Hoeveel bestanden zouden er moeten zijn?

Tel niet volgens een formule — het aantal uitvoerbestanden hangt af van welke producten zijn ingeschakeld en welke van toepassing zijn op elke camera (bijv. RGB-camera’s krijgen geen radiance/reflectance). Het definitieve aantal staat in het logbestand: de laatste regel `[RUN-SUMMARY]` geeft precies aan hoeveel bestanden zijn geschreven, en in de toelichtingsregels wordt uitgelegd wat er is overgeslagen.

***

## Bewerkte afbeeldingen bekijken

### Snel voorbeeld bekijken in Verkenner

**Ingebouwde voorbeeldweergave van Windows:**

1. Navigeer naar een productmap (bijv. `tiff16/Reflectance_Calibrated_Images/`)
2. Selecteer een afbeeldingsbestand
3. Het voorbeeld verschijnt in het voorbeeldvenster van Windows Verkenner
4. Gebruik de pijltjestoetsen om door de afbeeldingen te bladeren

### Voorbeeld bekijken in externe afbeeldingsviewers

**Aanbevolen viewers:**

* **QGIS** - Gratis GIS-software (het meest geschikt voor georefereerde multispectrale analyse)
* **IrfanView** – Snelle, lichtgewicht afbeeldingsviewer (ondersteunt TIFF)
* **Adobe Photoshop** – Professionele bewerking (ondersteunt TIFF)
* **GIMP** – Gratis alternatief voor Photoshop
* **Windows-foto’s** – Basisweergave (ondersteunt mogelijk geen 16-bits TIFF)

### Voorbeeldweergave in de Chloros-afbeeldingsviewer

Gebruik de ingebouwde afbeeldingsviewer van Chloros voor geavanceerde visualisatie:

1. Klik op een miniatuurafbeelding in de bestandsbrowser
2. De afbeelding wordt geopend in het hoofdvenster voor voorbeeldweergave
3. Klik op het tabblad **Afbeeldingsviewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
in de linkerzijbalk
4. Gebruik [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) voor interactieve analyse

Zie [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) voor gedetailleerde instructies.

***

## Reflectantie-pixelwaarden uitlezen (GIS / Pix4D / Scripts)

Reflectantie wordt opgeslagen als een geheel getal (DN), en **de DN die overeenkomt met ρ = 1,0 is afhankelijk van de broncamera**:

| Bron          | ρ = 1,0 is | Hoe te bepalen                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (speling tot ρ 2,0) | XMP-tag `Chloros:PixelScale=32768` is in het bestand opgenomen |
| Survey3          | **65535** (afgekapt bij ρ 1,0)     | Geen `Chloros:PixelScale`-tag — die afwezigheid is het signaal (Survey3-producten bevatten wel andere `Chloros:*`-verwerkingstags) |

**Lees de `Chloros:PixelScale`-tag en deel erdoor** in plaats van zomaar uit te gaan van een vaste waarde van 65535 — het delen van de LATTICE-reflectantie door 65535 halveert stilzwijgend elke waarde. Een reflectantiebestand van vóór versie 1.3.0 dat als 8-bit is opgeslagen, is afgekapt, niet opnieuw geschaald, en heeft geen schaal-tag: exporteer het opnieuw als 16-bit of 32-bit in plaats van het te delen. Zie [Uitvoerbeeldformaten](../output-image-formats.md) voor het volledige verhaal.

***

## Metadata die worden meegenomen in de exporten

Elk product behoudt het **GPS-blok**en de**EXIF-sub-IFD** van de bronopname, dus een
export bevat `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` en
`CameraSerialNumber`, evenals de georeferentie.

{% hint style="warning" %}
**Als een orthomozaïek op een absurde schaal wordt weergegeven, controleer dan eerst `FocalLength`.**
Pix4D berekent de grondresolutie op basis van de brandpuntsafstand plus de hoogte. Zonder de tag
valt het terug op een volkomen verkeerde schaal — bij een gemeten vlucht met 49 opnames werd een sinaasappelboomgaard van 411 m × 160 m
sinaasappelboomgaard gereconstrueerd als 47,8 km × 13 km, wat een ortho van 455 megapixels opleverde die grotendeels uit
lege ruimte bestond. Traag samenvoegen en een onverwacht groot bestand zijn symptomen hiervan, geen afzonderlijke
problemen.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Niet *elke* tag wordt gekopieerd. De structurele tags van IFD0 worden opzettelijk achterwege gelaten (het kopiëren
ervan beschadigt de LATTICE-uitvoer), en `ExifImageWidth` / `ExifImageHeight` worden uitgesloten
omdat ze de oorspronkelijke opname beschrijven — een export waarvan het formaat is aangepast, zou anders
afmetingen claimen die in tegenspraak zijn met het eigen raster.

Chloros schrijft ook zijn eigen `Chloros:*` XMP-tags waarin wordt beschreven wat er met elk product is gedaan
(bijvoorbeeld `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Vanaf versie 1.3.0 bevatten Survey3-producten deze tags in elke verwerkingsmodus
verwerkingsmodus; indexproducten beschrijven de index zelf (`Chloros:ProcessingLevel=index`, de
formule en of deze is berekend op basis van reflectantie of straling) in plaats van de
tags van de bron-raw te herhalen, en producten in mappen die dieper liggen dan 260 tekens behouden ook hun metagegevens. Zie [Uitvoerbeeldformaten](../output-image-formats.md#index-products).

***

## Het debuglogboek bekijken

### Controleren op waarschuwingen of fouten

1. Open het tabblad **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Blader door de berichten
3. Zoek naar gele waarschuwingen of rode fouten
4. Lees de `[RUN-SUMMARY]`-regels en eventuele hints
5. Neem contact op met MAPIR-ondersteuning voor hulp

### Het logboek opslaan

Om een verslag van de verwerking bij te houden of om het naar de MAPIR-ondersteuning te sturen:

1. Klik op de knop **"Kopiëren"**of**"Downloaden"**

2. Sla het op als tekstbestand in de projectmap
3. Voeg het toe aan de projectdocumentatie
4. Stuur het naar de MAPIR-ondersteuning als er problemen zijn opgetreden

Om een probleem rechtstreeks naar MAPIR te sturen, gebruik je het bugpictogram (**Een bug melden**) bovenaan het tabblad Debuglog — zie [Een bug melden vanuit het tabblad Log](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Veelvoorkomende uitvoerproblemen en oplossingen

### Probleem: Ontbrekende uitvoerbestanden

**Mogelijke oorzaken:**

* Het product is niet van toepassing op die camera (bijv. radiance/reflectance voor RGB-camera’s — dit wordt in het logboek vermeld)
* Er ontbrak een vereiste referentie (bijv. reflectantie zonder doel en zonder `.daq` neerwaartse straling)
* Het selectievakje voor het exporteren van het product was uitgeschakeld in de projectinstellingen
* Er was onvoldoende schijfruimte tijdens het exporteren

**Oplossingen:**

1. Controleer de `[RUN-SUMMARY]`-hints en `[EXPORT-CHECK]`-regels in het foutlogboek — deze geven uitleg over het overslaan van afzonderlijke camera's
2. Controleer de selectievakjes voor het exporteren van het product in [Projectinstellingen](adjusting-project-settings.md)
3. Controleer of er voldoende schijfruimte was
4. Voer de bewerking opnieuw uit nadat de oorzaak is verholpen

### Probleem: Donkere of lichte randen (vignettering nog steeds zichtbaar)

**Mogelijke oorzaken:**

* Vignetteringscorrectie uitgeschakeld
* Camera/lens staat niet in de Chloros-profieldatabase
* Extreme vignettering die de correctiemogelijkheden te boven gaat

**Oplossingen:**

1. Controleer of vignetteringscorrectie is ingeschakeld in de projectinstellingen
2. Controleer of het cameramodel correct is gedetecteerd
3. Neem contact op met MAPIR-ondersteuning als de vignettering aanhoudt

### Probleem: onjuiste kleuren of waarden

**Mogelijke oorzaken:**

* Geen kalibratiedoelen gedetecteerd
* Verkeerd kalibratiedoelmodel geselecteerd
* Reflectiekalibratie uitgeschakeld
* Doelafbeeldingen van slechte kwaliteit

**Oplossingen:**

1. Controleer of reflectiekalibratie is ingeschakeld
2. Controleer de meldingen „Doel gevonden“ in het foutopsporingslogboek
3. Controleer de kwaliteit van de doelafbeeldingen
4. Voer de verwerking opnieuw uit met de juiste doelen gemarkeerd

### Probleem: NDVI-waarden lijken onjuist

**Verwachte NDVI-bereiken:**

* **Water, rotsen, grond**: -0,1 tot 0,2
* **Schaarse/ongezonde vegetatie**: 0,2 tot 0,4
* **Matige vegetatie**: 0,4 tot 0,6
* **Gezonde, dichte vegetatie**: 0,6 tot 0,9**Als de waarden buiten deze bereiken vallen:**

1. Controleer of de reflectiekalibratie is toegepast
2. Controleer of het logbestand van de lichtsensor is opgenomen
3. Controleer of de kalibratiedoelen zijn gedetecteerd
4. Zorg ervoor dat het juiste cameramodel is gedetecteerd
5. Controleer het tijdstip en de omstandigheden van de opname van de doelbeelden
6. Als u zelf indices berekent op basis van reflectiebestanden, controleer dan of u hebt gedeeld door de waarde `Chloros:PixelScale` van het bestand (zie hierboven)

***

## Uw verwerkte beelden gebruiken

### Voor fotogrammetrie / het maken van orthomozaïeken

**Aanbevolen workflow:**

1.**Importeer gekalibreerde reflectantiebeelden** in fotogrammetriesoftware:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behoud EXIF-metadata**: zorg ervoor dat GPS-gegevens behouden blijven voor geotagging
3. **Gekalibreerde workflows**: gebruik reflectantiebeelden voor wetenschappelijke nauwkeurigheid — LATTICE-reflectantiebeelden bevatten de XMP-kalibratietags die Pix4D leest
4. **Verwerk indexmozaïeken**: maak NDVI-orthomozaïeken op basis van afzonderlijke indexbeelden
5. **GeoTIFF met georeferentie exporteren**: Voor gebruik in GIS-toepassingen

### Voor GIS-analyse

**Aanbevolen workflow:**

1.**Importeer in QGIS, ArcGIS of vergelijkbare software**

2.**Gebruik 16-bits TIFF** reflectiebeelden voor multibandanalyse (deel door de waarde `Chloros:PixelScale` in het bestand)
3. **Gebruik indexbeelden** (NDVI, NDRE) als kant-en-klare vegetatielagen
4. **Rastercalculator**: combineer banden voor aangepaste analyses
5. **Exporteren**: maak classificatiekaarten, veranderingdetectie en kaarten van de vegetatiegezondheid

### Voor directe analyse / rapportage

**Aanbevolen werkwijze:**

1.**Gebruik indexafbeeldingen met LUT-kleuren** voor visuele rapporten
2. **Statistieken extraheren**: gemiddelde NDVI-waarde per veld/perceel
3. **Tijdreeksen**: vergelijk indices over meerdere sessies heen
4. **Rapporten genereren**: voeg kaarten, statistieken en visualisaties toe***

## Archivering en back-up

### Aanbevolen back-upstrategie

**Wat moet je opslaan:**

* ✅ **Originele RAW/JPG-afbeeldingen of LATTICE-raw-opnames** – Archiveer op een aparte schijf/in de cloud; raw is de bron van de verwerkingspijplijn en al het andere kan hieruit opnieuw worden gegenereerd
* ✅ **`.daq` / `.csv`-bestanden van de lichtsensor** – Nodig om later de reflectie opnieuw te berekenen
* ✅ **Verwerkte resultaten** – Bewaar gekalibreerde beelden en indices
* ✅ **Projectmap** (`project.json` en bijbehorende bestanden) - Bevat alle instellingen voor herverwerking indien nodig
* ✅ **Debuglog** - Documenteert verwerkingsdetails
* ✅ **Kalibratiedoelafbeeldingen** - Voor verificatie en herverwerking

**Aanbevelingen voor opslag:**

* **Onmiddellijke back-up**: Externe harde schijf
* **Langetermijnarchief**: Cloudopslag (Google Drive, Dropbox, enz.)
* **Kritieke gegevens**: Bewaar 2-3 kopieën op verschillende locaties***

## Volgende verwerkingsruns

### Projectinstellingen hergebruiken

Als je in de toekomst vergelijkbare datasets gaat verwerken:

1. **Sla de projectsjabloon op** (als je dat nog niet hebt gedaan)
2. **Maak een nieuw project aan** met behulp van de opgeslagen sjabloon
3. **Importeer nieuwe afbeeldingen**

4.**Verwerk**met identieke instellingen voor consistentie

### Batchverwerking van meerdere sessies

Voor meerdere sessies/datasets:**Optie 1: GUI – Meerdere projecten**

* Maak voor elke sessie een apart project aan
* Gebruik consistente sjablooninstellingen
* Verwerk ze één voor één

**Optie 2: Chloros CLI (alleen Chloros+)**

* Automatiseer batchverwerking
* Verwerk meerdere mappen met scripts
* Zie [CLI-documentatie](../CLI.md) en de [CLI-referentie](../reference/cli-reference.md)

**Optie 3: Python SDK (alleen Chloros+)**

* Programmatische besturing
* Integratie met analysepijplijnen
* Zie [API-documentatie](../api-python-sdk.md) en de [SDK-referentie](../reference/sdk-reference.md)

***

## Problemen met nabewerking oplossen

### Opnieuw verwerken met andere instellingen

Als de resultaten niet naar wens zijn:

1. Bewaar de originele afbeeldingen (nooit verwijderen)
2. Open hetzelfde project in Chloros
3. Pas de instellingen aan in het paneel ‘Projectinstellingen’
4. Verwerk opnieuw — de uitvoer komt in dezelfde productmappen terecht, dus bestanden met dezelfde naam uit de vorige run worden vervangen

### Een deel van de afbeeldingen verwerken

Om alleen specifieke afbeeldingen opnieuw te verwerken:

1. Maak een nieuw project aan
2. Importeer alleen de afbeeldingen die opnieuw moeten worden verwerkt
3. Gebruik hetzelfde instellingensjabloon
4. Verwerk de kleinere dataset

### Hulp vragen

Als je problemen ondervindt:

* 📧 **E-mail**: info@mapir.camera (voeg het foutlogboek toe)
* 🌐 **Ondersteuning**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Veelgestelde vragen**: [Veelgestelde vragen](../faq.md)
* 📖 **Documentatie**: [Chloros-handleiding](../)

***

## Samenvatting: Volledige workflow

Je hebt nu de volledige Chloros-verwerkingsworkflow doorlopen:

1. ✅ **Project aangemaakt** - Zie [Projecten](../projects.md)
2. ✅ **Bestanden toegevoegd** - Zie [Bestanden toevoegen](adding-files-to-a-project.md)
3. ✅ **Instellingen aangepast** - Zie [Projectinstellingen aanpassen](adjusting-project-settings.md)
4. ✅ **Doelen gemarkeerd** - Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
5. ✅ **Verwerking gestart** - Zie [De verwerking starten](starting-the-processing.md)
6. ✅ **Voortgang gecontroleerd** - Zie [De verwerking controleren](monitoring-the-processing.md)
7. ✅ **Resultaten bekeken** - Deze pagina**Uw gekalibreerde, reflectiecorrigeerde multispectrale beelden zijn klaar voor analyse!**

***

## Aanvullende bronnen

### Geavanceerde functies

* [**Beeldviewer**](../image-viewer-gui/opening-an-image-full-screen.md) - Interactieve visualisatie en analyse
* [**Index/LUT-sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Testen van aangepaste indexen
* [**Formules voor multispectrale indexen**](../project-settings/multispectral-index-formulas.md) - Volledige indexreferentie

### Automatisering en integratie

* [**CLI-documentatie**](../CLI.md) - Batchverwerking via de opdrachtregel
* [**Python SDK**](../api-python-sdk.md) - Programmatische automatisering
* [**Functies van Chloros+**](../#chloros) - Geavanceerde verwerkingsmogelijkheden

### Ondersteuning & Leren

* [**Veelgestelde vragen**](../faq.md) - Antwoorden op veelgestelde vragen
* [**Kalibratiedoelen**](../calibration-targets.md) - Inzicht in reflectiekalibratie
* [**Ondersteunde camera's**](../supported-cameras.md) - Compatibele hardware
