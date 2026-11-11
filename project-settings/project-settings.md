# Projectinstellingen

Via de zijbalk ‘Projectinstellingen’ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

in Chloros

kun je alle aspecten van beeldverwerking, detectie van kalibratiedoelen, berekeningen van multispectrale indexen en exportopties voor je project configureren. Deze instellingen worden samen met je project opgeslagen en kunnen als sjablonen worden opgeslagen voor hergebruik in meerdere projecten.

## Projectinstellingen openen

Om de projectinstellingen te openen:

1. Open een project in Chloros


2. Klik op het tabblad **Projectinstellingen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

in de linkerzijbalk
3. In het instellingenpaneel worden alle beschikbare configuratieopties weergegeven, geordend per categorie

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Instellingen die afhankelijk zijn van andere instellingen, zijn grijs weergegeven.** Wanneer een bovenliggende schakelaar een instelling onmogelijk maakt (bijvoorbeeld: als je *Reflectanciekalibratie / witbalans* uitschakelt, is *Reflectantie exporteren* niet meer mogelijk), wordt de afhankelijke instelling uitgeschakeld en wordt in de tooltip aangegeven welke schakelaar moet worden gewijzigd.
{% endhint %}

***

## Weergave

### Resolutie van afbeeldingsminiaturen

* **Type**: keuzelijst
* **Opties**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Standaard**: Standaard (512 px)
* **Beschrijving**: Resolutie (langste zijde, in pixels) waarmee de miniatuurafbeeldingen in het raster worden weergegeven. Hogere waarden zien er scherper uit bij inzoomen, maar laden langzamer en gebruiken meer geheugen. Volledige resolutie komt overeen met de oorspronkelijke afbeeldingsgrootte.
* **Opmerking**: Alleen voor weergave — dit heeft nooit invloed op de verwerking of geëxporteerde bestanden.***

## Doeldetectie

Deze instellingen bepalen hoe Chloros

-kalibratiedoelen in je afbeeldingen detecteert en verwerkt. Beide zijn alleen actief wanneer **Reflectantiekalibratie / witbalans** is ingeschakeld (anders zijn ze grijs weergegeven, omdat doeldetectie dan volledig wordt overgeslagen).

### Minimaal kalibratiesamplegebied (px)

* **Type**: Getal
* **Bereik**: 0 tot 10.000 pixels
* **Standaard**: 25 pixels
* **Beschrijving**: Stelt het minimale gebied (in pixels) in dat nodig is om een gedetecteerd gebied als een geldig kalibratiedoel te beschouwen. Kleinere waarden detecteren kleinere doelen, maar kunnen het aantal valse positieven verhogen. Grotere waarden vereisen grotere, duidelijkere doelgebieden voor detectie.
* **Wanneer aanpassen**:
  * Verhoog deze waarde als je valse detecties krijgt bij kleine beeldartefacten
  * Verlaag deze waarde als je kalibratiedoelen klein lijken in je afbeeldingen en niet worden gedetecteerd

### Minimale doelclustering (0-100)

* **Type**: Getal
* **Bereik**: 0 tot 100
* **Standaard**: 60
* **Beschrijving**: Bepaalt de clusteringdrempel voor het groeperen van gebieden met vergelijkbare kleuren bij het detecteren van kalibratiedoelen. Bij hogere waarden moeten meer vergelijkbare kleuren worden gegroepeerd, wat resulteert in een conservatievere doelherkenning. Lagere waarden staan meer kleurvariatie toe binnen een doelgroep.
* **Wanneer aanpassen**:
  * Verhoog deze waarde als kalibratiedoelen in meerdere detecties worden opgesplitst
  * Verlaag deze waarde als kalibratiedoelen met kleurvariatie niet volledig worden gedetecteerd

***

## Verwerking

Deze instellingen bepalen hoe Chloros

uw afbeeldingen verwerkt en kalibreert.

### Vignetteringscorrectie

* **Type**: Selectievakje
* **Standaard**: Ingeschakeld (aangevinkt)
* **Beschrijving**: Past vignetteringscorrectie toe om lensverduistering aan de randen van afbeeldingen te compenseren. Vignettering is een veelvoorkomend optisch verschijnsel waarbij de hoeken en randen van een afbeelding donkerder lijken dan het midden als gevolg van de eigenschappen van de lens.
* **Neveneffect**: Met deze schakelaar wordt ook geselecteerd welk *niet-gekalibreerd fallback-product* een run oplevert (zie hieronder).

### Reflectantiekalibratie / witbalans

* **Type**: Selectievakje
* **Standaard**: Ingeschakeld (aangevinkt)
* **Beschrijving**: Schakelt reflectiekalibratie in — op basis van gedetecteerde kalibratiedoelen binnen het beeld en/of downwelling-gegevens van de DAQ-lichtsensor, afhankelijk van de camera en wat beschikbaar is. Dit normaliseert de reflectiewaarden in uw dataset en zorgt voor consistente metingen, ongeacht de lichtomstandigheden.
* **Wanneer uitgeschakeld**: Doeldetectie wordt volledig overgeslagen en**er kan door geen enkele camera een reflectantieproduct worden gegenereerd** — zowel bij Survey3

-doelgestuurde als bij LATTICE DAQ-gestuurde opnames. De afhankelijke instellingen (*Reflectantie exporteren*, *Minimaal herkalibratie-interval* en de drempelwaarden voor doeldetectie) zijn grijs weergegeven.

### Niet-gekalibreerde fallback-producten: Sensorrespons exporteren / Vignettering gecorrigeerd exporteren

* **Type**: Twee selectievakjes
* **Standaardinstellingen**: Beide ingeschakeld (aangevinkt)
* **Beschrijving**: Wanneer een frame niet op reflectantie kan worden gekalibreerd (er is geen kalibratiedoel gevonden of de reflectantiekalibratie is uitgeschakeld), wordt het in plaats daarvan opgeslagen als een *niet-gekalibreerd fallback-product*. **Per run bestaat er precies één van de twee fallback-producten, voor elk cameramodel**, gekozen door de schakelaar *Vignetcorrectie*:
  * Vignetcorrectie **aan**→ `Vignette_Corrected_Images/` (bepaald door**Exporteren met vignetcorrectie**)
  * Vignetcorrectie **uit**→ `Sensor_Response_Images/` (bepaald door**Exporteren met sensorrespons**)
* De fallback die niet actief is, wordt grijs weergegeven. Als je het vinkje bij de actieve optie verwijdert, wordt dat bestand helemaal niet meer opgeslagen.

### LATTICE-exportproducten

Bij projecten met LATTICE-opnames wordt elk geïmporteerd LATTICE-frame in één verwerkingsronde uitgesplitst naar alle ingeschakelde **en toepasselijke**producten. Vier selectievakjes regelen de verspreiding (standaard allemaal**ingeschakeld**):

| Instelling | Uitvoermap | Wat er wordt geëxporteerd |
| --- | --- | --- |
| **Export debayered** | `Debayered_Images/` | Het lineair gedebayerde beeld: de donkere offset van het kalibratiepakket is verwijderd, gevolgd door een randbewuste demosaic, zonder lineariteitscorrectie, flat field of unmix — hetzelfde bestand dat een opname opslaat op het debayered-niveau. Behoudt de oorspronkelijke oriëntatie van de sensor (opgeslagen in XMP). Geldt voor RGB

- en multispectrale camera’s. |
| **Voorbeeld exporteren** | `Preview_Images/` | Het weergavevoorbeeld. RGB

= witbalans (DAQ-lichtbron indien beschikbaar, anders ‘gray-world’) + gamma; multispectraal = valsekleurenuitrekking. |
| **Stralingsintensiteit exporteren** | `Radiance_Images/` | Float32 spectrale stralingsintensiteit in W/m²/sr/nm. Alleen multispectraal (M3C/M3M) — niet van toepassing op RGB

-masters. Wordt altijd opgeslagen als 32-bits TIFF

, ongeacht de instelling *Gekalibreerd beeldformaat*. |
| **Reflectantie exporteren**| `Reflectance_Calibrated_Images/` | Uint16-reflectantie, geschaald zodat**32768 = reflectantie 1,0** (gemarkeerd als XMP `Chloros:PixelScale`). Alleen multispectraal, opgeslagen wanneer een bijbehorend `.daq`-downwelling-record (of een QA-goedgekeurd doel binnen het beeld) het beeld bestrijkt. |

* RGB

-mastercamera’s zenden debayered + preview uit; radiance/reflectance worden hiervoor overgeslagen omdat ze niet van toepassing zijn.
* De bitdiepte van debayered/preview volgt de instelling *Gekalibreerd beeldformaat*; radiance is altijd float32.
* De verwerking van Survey3

wordt niet beïnvloed door deze vier schakelaars.
* Stralings- en reflectantieproducten van multispectrale M3C-camera’s kunnen ook een per-pixel **clamp-masker** per pixel op in een `clamp_masks/`-map naast de productmappen, waarbij wordt vastgelegd waar de bandontmenging een negatieve waarde tot nul heeft geknipt. Zie [Uitvoerbeeldformaten](../output-image-formats.md#clamp-masks).

Dezelfde vier schakelaars bestaan ook in headless-modus als `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Ze hebben de oude CLI

-vlag `--radiometric-output` vervangen, die niet langer bestaat.

{% hint style="warning" %}
**Als je alle toepasselijke producten uitschakelt, mislukt de run.** Vanaf versie 1.2.0 meldt een verwerkingsrun waarvoor producten werden aangevraagd maar die geen beeldproducten heeft geschreven, een fout en sluit de CLI

af met een waarde anders dan nul, in plaats van stilzwijgend succes te melden. Het logboek vermeldt welk product niet kon worden geschreven en waarom. Een opzettelijke run met alleen metadata (niets aangevraagd) is nog steeds een succes.
{% endhint %}

### Reflectantiebron (projectinstelling, ingesteld via de CLI

)

Het project slaat ook op welke **reflectantierferentie** het LATTICE-reflectantieproduct gebruikt. Er is geen specifieke instelling in het instellingenpaneel; de waarde wordt in de projectconfiguratie opgeslagen als `Processing → "Target reflectance source"` en wordt ingesteld met `chloros-cli process --reflectance-source {auto,target,daq}` (`configure()` in de SDK

heeft hiervoor geen trefwoord):

* **`auto`** (standaard): een in-frame kalibratiedoel dat de kwaliteitscontrole (QA) doorstaat, wordt de absolute referentie; als er geen doel aanwezig is of de QA mislukt, wordt teruggevallen op de DAQ-downwelling-dividend (ρ = πL/E). Een doel dat de kwaliteitscontrole (QA) met een vlag doorstaat (een paneel is gevallen of een band is defect) krijgt niet automatisch de voorkeur: de ketens ‘doel’, ‘doel+DAQ’ en ‘alleen DAQ’ worden getest in volgorde van hun gemeten score op de buiten beschouwing gelaten panelen.
* **`target`**: strikt doelgestuurde reflectantie — geen DAQ-vervanging.
* **`daq`**: DAQ-bepaalde reflectantie; in-frame-doelen worden niet als referentie gebruikt.

De opgeslagen waarde wordt zonder onderscheid tussen hoofdletters en kleine letters vergeleken en enkele spellingsvarianten worden als aliassen geaccepteerd: `target`, `target_image`, `empirical` en `empirical_line` betekenen allemaal **doel**; `daq`, `dls`, `light_sensor` en `sensor` betekenen allemaal**daq**. Al het andere — inclusief een ontbrekende sleutel — wordt omgezet naar**auto**.**Gemeten** doelscans per eenheid worden opgezocht aan de hand van het serienummer/QR-code van de doeleenheid, zoals `<serial>.csv`, op drie plaatsen: de map die is opgegeven met `--target-reflectance-dir` (opgeslagen als `Processing → "Target reflectance dir"`), de project-eigen map `target_reflectance/`, en het pad in de omgevingsvariabele `CHLOROS_TARGET_REFLECTANCE_DIR`. Als er geen gemeten scan voor die eenheid bestaat, wordt in plaats daarvan de nominale gepubliceerde curve voor het doelmodel gebruikt.

### Debayer-methode

* **Type**: Keuzelijst
* **Opties**:
  * Standaard (snel, gemiddelde kwaliteit)
  * Textuurbewust (traag, hoogste kwaliteit) \[Chloros

+]
* **Standaard**: Standaard (snel, gemiddelde kwaliteit)
* **Beschrijving**: Selecteert het demosaicing-algoritme dat wordt gebruikt om ruwe Bayer-patroonsensorgegevens om te zetten in full-colour beelden. De methode „Standaard (snel, gemiddelde kwaliteit)“ biedt een optimale balans tussen verwerkingssnelheid en beeldkwaliteit. De methode „Textuurbewust (traag, hoogste kwaliteit)“ \[Chloros

+] maakt gebruik van een hoogwaardige, randgevoelige debayering in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayeringruis verwijdert. Het Textuurgevoelig-model vereist GPU-geheugen (VRAM) om te kunnen draaien. We raden aan dit te gebruiken wanneer je &gt;4 GB VRAM beschikbaar hebt voor een snellere verwerking.
* **Als de rij überhaupt een keuzelijst is**: de keuzelijst met twee opties verschijnt alleen wanneer**beide** voorwaarden zijn vervuld — je bent aangemeld met een in aanmerking komend Chloros

+-abonnement, **en** het project bevat geen LATTICE-opnames. Anders wordt de rij weergegeven als platte tekst met de tekst `Standard (Fast, Medium Quality)`, zonder keuzemogelijkheden.
* **Opmerking over LATTICE**: Er bestaat geen door LATTICE getraind ‘Texture Aware’-model, en de pijplijn dwingt de standaard-demosaic af voor LATTICE-frames, ongeacht de opgeslagen waarde. Als je een LATTICE-map toevoegt aan een project waarin ‘Texture Aware’ al was geselecteerd, zet Chloros

de instelling terug naar ‘Standaard’ in plaats van een verouderde waarde in `project.json`.

### Minimaal herkalibratie-interval

* **Type**: Getal
* **Bereik**: 0 tot 3.600 seconden
* **Standaard**: 0 seconden
* **Beschrijving**: Stelt het minimale tijdsinterval (in seconden) in tussen het gebruik van kalibratiedoelen. Bij instelling op 0 gebruikt Chloros

elk gedetecteerd kalibratiedoel. Bij instelling op een hogere waarde gebruikt Chloros

alleen kalibratiedoelen die ten minste dit aantal seconden van elkaar verwijderd zijn, waardoor de verwerkingstijd wordt verkort voor datasets met frequente opnames van kalibratiedoelen.
* **Wanneer aanpassen**:
  * Stel in op 0 voor maximale kalibratienauwkeurigheid wanneer de lichtomstandigheden variëren
  * Verhoog de waarde (bijv. naar 60–300 seconden) voor snellere verwerking wanneer het licht constant is en u veelvuldig beelden van kalibratiedoelen hebt

### Tijdzone-offset lichtsensor

* **Type**: Getal
* **Bereik**: -12 tot +12 uur
* **Standaard**: 0 uur
* **Beschrijving**: Geeft de tijdzone-offset (in uren ten opzichte van UTC) aan voor tijdstempels van lichtsensorgegevens, die wordt gebruikt bij het koppelen van lichtsensorlogboeken aan de opnametijden van beelden. Nieuwere `.daq`-opnames bevatten hun eigen tijdzone-herkomst, dus dit is voornamelijk nodig voor oudere logboeken die in lokale tijd zijn opgenomen.

### PPK-correcties toepassen

* **Type**: Selectievakje
* **Standaard**: Uitgeschakeld (niet aangevinkt)
* **Beschrijving**: Schakelt het gebruik van Post-Processed Kinematic (PPK)-correcties in van MAPIR

DAQ-recorders die zijn uitgerust met een GPS (GNSS). Wanneer deze optie is ingeschakeld, gebruikt Chloros

alle .daq-logbestanden met belichtingspinnengegevens in uw projectmap en past het nauwkeurige geolocatiegecorrecties toe op uw beelden.
* **Vereiste**: Er moet een .daq-logbestand met belichtingspin-gegevens aanwezig zijn in uw projectmap
* **Wanneer inschakelen**: Het wordt aanbevolen om PPK-correctie altijd in te schakelen als u belichtingsfeedbackgegevens in uw .daq-logbestand hebt.

### Belichtingspin 1

* **Type**: Keuzelijst
* **Zichtbaarheid**: Alleen zichtbaar wanneer &quot;PPK-correcties toepassen&quot; is ingeschakeld EN er belichtingsgegevens beschikbaar zijn voor pin 1
* **Opties**:
  * Namen van cameramodellen die in het project zijn gedetecteerd
  * &quot;Niet gebruiken&quot; – Deze belichtingspin negeren
* **Standaard**: Automatisch geselecteerd op basis van de projectconfiguratie
* **Beschrijving**: Wijst een specifieke camera toe aan belichtingspin 1 voor PPK-tijdsynchronisatie. De belichtingspin registreert het exacte tijdstip waarop de camerasluiter wordt geactiveerd, wat cruciaal is voor nauwkeurige PPK-geolocatie.
* **Gedrag bij automatische selectie**:
  * Eén camera + één pin: selecteert automatisch de camera
  * Eén camera + twee pinnen: pin 1 wordt automatisch toegewezen aan de camera
  * Meerdere camera’s: handmatige selectie vereist

### Belichtingspin 2

* **Type**: Keuze uit een vervolgkeuzelijst
* **Zichtbaarheid**: Alleen zichtbaar wanneer „PPK-correcties toepassen“ is ingeschakeld EN er belichtingsgegevens beschikbaar zijn voor pin 2
* **Opties**:
  * In het project gedetecteerde cameramodelnamen
  * „Niet gebruiken“ – Deze belichtingspin negeren
* **Standaard**: Automatisch geselecteerd op basis van de projectconfiguratie
* **Beschrijving**: Wijst een specifieke camera toe aan belichtingspin 2 voor PPK-tijdsynchronisatie bij gebruik van een opstelling met twee camera’s.
* **Gedrag bij automatische selectie**:
  * Eén camera + één pin: Pin 2 wordt automatisch ingesteld op „Niet gebruiken“
  * Eén camera + twee pinnen: Pin 2 wordt automatisch ingesteld op „Niet gebruiken“
  * Meerdere camera’s: Handmatige selectie vereist
* **Opmerking**: Dezelfde camera kan niet tegelijkertijd aan zowel pin 1 als pin 2 worden toegewezen.***

## DAQ-lichtsensor

Dit gedeelte verschijnt in Projectinstellingen en geeft een overzicht van alle DAQ-downwelling-bestanden in het project — `.daq`-opnames en DAQ-M `.csv`-downwellinglogs. Opnames die zijn gemaakt op het tabblad &#x27;Lichtsensoren&#x27; worden automatisch toegevoegd aan het geopende project.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Elke rij toont het bestand, het sensormodel en de diffusorkapcorrectie die daadwerkelijk van kracht is voor dat bestand. Boven de rijen bevindt zich één projectbrede instelling:

### Kapoverschrijving (alle bestanden)

* **Type**: keuzelijst
* **Opties**: `Auto` plus de diffusorkapcorrectieprofielen die geldig zijn voor de sensortypen die in het project aanwezig zijn
* **Standaard**: Auto
* **Opgeslagen als**: `Processing → "DAQ cap id"` (standaard `auto`)
* **Beschrijving**: `Auto` gebruikt de geregistreerde kap van elk bestand (de Sunshine-kap wordt verondersteld wanneer er niets is geregistreerd, maar DAQ-U, DAQ-M en DAQ-E worden geleverd zonder kap en Chloros

kan niet zien wat er is gemonteerd, dus geeft de verwerking een waarschuwing bij een veronderstelde cap: kies de cap die was gemonteerd). Het kiezen van een specifieke cap overschrijft **elk** downwelling-bestand in het project, behalve die van een DAQ-E-S, waarvan de diffuser deel uitmaakt van de kalibratie; een project met alleen DAQ-E-S’en toont geen keuzelijst: ruwe opnames worden hiermee gecorrigeerd, en opnames die al een kap bevatten, worden opnieuw gerefereerd (de opgenomen correctie wordt ongedaan gemaakt en de geselecteerde correctie wordt toegepast).
* **Belangrijk**: De geselecteerde kap moet overeenkomen met de kap die fysiek tijdens de opname was gemonteerd. Noch de sensor, noch de software kan de fysieke kap detecteren — een verkeerd gekoppelde kap-ID leidt tot een onjuiste correctie van de spectra.

Er is bewust gekozen voor **één** projectbrede instelling in plaats van dropdown-menu’s per bestand: de instelling is van toepassing op elke neerwaartse bron in het project.***

## Uitlijning van arrays

Dit gedeelte verschijnt **alleen** wanneer ten minste één afbeelding in het project de module-naar-module-uitlijningstransformatie bevat die LATTICE-arrays bij het vastleggen aanbrengen (XMP `Chloros:Alignment*`-tags). Hier wordt weergegeven hoeveel afbeeldingen uitlijningstags bevatten, welke camera als referentie dient (`REF`-badge) en een tabel met het aantal afbeeldingen per camera.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Array-uitlijning toepassen

* **Type**: Selectievakje
* **Standaard**: Ingeschakeld (aangevinkt)
* **Opgeslagen als**: `Processing → "Array alignment"`
* **Beschrijving**: Vervormt elk verwerkt product (debayered / preview / radiance / reflectance / index) naar de gedeelde referentiegeometrie van de array met behulp van de transformatie die bij het vastleggen is vastgelegd. Uitgeschakeld = exporteren in de oorspronkelijke geometrie per sensor.

### Bijsnijden tot gemeenschappelijke overlap

* **Type**: Selectievakje (alleen actief als *Array-uitlijning toepassen* is ingeschakeld)
* **Standaard**: Ingeschakeld (aangevinkt)
* **Opgeslagen als**: `Processing → "Array alignment crop"`
* **Beschrijving**: Bijsnijden van uitgelijnde exporten tot het gebied dat alle cameramodules delen, zodat elke band dezelfde voetafdruk heeft. Uit behoudt het volledige sensorcanvas (zwarte vulling buiten de bron).

### Herbemonstering

* **Type**: Keuzelijst (alleen actief wanneer *Array-uitlijning toepassen* is ingeschakeld)
* **Opties**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Standaard**: Bilineair
* **Opgeslagen als**: `Processing → "Array alignment interpolation"`
* **Beschrijving**: Interpolatie die wordt gebruikt door de uitlijningsvervorming. *Nearest* behoudt de exacte bronwaarden (geen vermenging tussen pixels) voor strikte radiometrische analyse; *Bilineair* is het beste voor kaarten en visueel gebruik.

Dezelfde drie opties bestaan ook zonder voorvoegsel als `chloros-cli process --array-alignment`, `--array-alignment-crop` en `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Met deze instellingen kunt u multispectrale indices configureren voor analyse en visualisatie.

### Index toevoegen

* **Type**: Speciaal configuratiepaneel voor indexen
* **Beschrijving**: Opent een interactief paneel waarin u multispectrale vegetatie-indexen (NDVI

, NDRE

, EVI

, enz.) kunt selecteren en configureren om tijdens de beeldverwerking te berekenen. U kunt meerdere indexen toevoegen, elk met hun eigen visualisatie-instellingen.
* **Beschikbare indices**: De vervolgkeuzelijst in de GUI bevat**27** vooraf gedefinieerde multispectrale indexformules (zie [Multispectrale indexformules](multispectral-index-formulas.md) voor de volledige lijst, inclusief welke namen ook worden geaccepteerd door de CLI

/SDK

`--indices`-optie).
* **Functies**:
  * Kies uit vooraf gedefinieerde indexformules
  * Sleep de filterkanalen van je camera naar de bandvakken van de formule
  * Configureer kleurverloop voor visualisatie (LUT - Look-Up Tables)
  * Stel drempelwaarden en clippingmodi in
  * Maak aangepaste indexformules
* **Opmerking**: Er worden geen indexen berekend voor LATTICE M3M-monochromecamera’s met één band — multibandindexen zijn ongedefinieerd bij één band. Dit heeft geen invloed op de Survey3

- en LATTICE M3C-camera’s.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Elke index die u toevoegt, geeft de formule weer als wiskundige uitdrukking, met een gekleurde cirkel per bandvak: rood = Red

, groen = Green

, blauw = Blue

, oranje = Orange

, cyaan = Cyan

, paars = NIR

, magenta = RE. Sleep een cirkel vanuit de rij boven de formule naar een slot om deze te koppelen; dubbelklik op een gekoppeld slot om de koppeling te verwijderen. De index wordt slechts één keer berekend als elk slot dat de formule gebruikt een kanaal bevat.

### Aangepaste formules (Chloros

+-functie)

* **Type**: Reeks definities van aangepaste formules
* **Beschikbaarheid**: Vereist aanmelding met een in aanmerking komend Chloros

+-abonnement.
* **Beschrijving**: Hiermee kun je aangepaste multispectrale indexformules maken en opslaan met behulp van bandwiskunde. Aangepaste formules worden opgeslagen met je projectinstellingen en kunnen net als ingebouwde indexen worden gebruikt.
* **Zo maakt u een formule aan**:
  1. Open in het configuratiepaneel ‘Index’ de rekenmachine voor aangepaste formules
  2. Schrijf de formule met behulp van de **band-slot-symbolen**, niet met bandnamen
  3. Sla de formule op onder een beschrijvende naam — deze verschijnt dan onderaan de formule-keuzelijst, en je sleept de kanaalcirkels van je camera naar de bijbehorende slots, precies zoals bij een ingebouwde voorinstelling
* **Formulesyntaxis**:
  * Band-slots: `x`, `y`, `z`, `a`, `b`, `c` — zes posities die je door te slepen aan echte kanalen toewijst
  * Operatoren: `+`, `-`, `*`, `/`, `^` en `()` voor groepering
  * Functies: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Waarom symbolen, geen bandnamen**: een formule geschreven als `(y-x)/(y+x)` werkt op elke camera, omdat de drag-and-drop-toewijzing bepaalt of `y` de 850 nm NIR

van een RGN

-filter is of de 808 nm NIR

van een OCN

-filter. De ingebouwde voorinstellingen worden op dezelfde manier opgeslagen — zie [Multispectrale indexformules](multispectral-index-formulas.md) voor de exacte symboolvorm van alle 27.
* **Waar ze werken**: aangepaste formules worden samen met de projectinstellingen opgeslagen en kunnen zowel in de [Index/LUT-sandbox](../image-viewer-gui/index-lut-sandbox.md) als tijdens de verwerking worden gebruikt. Ze worden**niet** geaccepteerd door de CLI

/SDK

-naamlijst `--indices`, die alleen de 22 ingebouwde voorinstelnamen uitbreidt.

***

## Exporteren

Deze instellingen bepalen het formaat en de kwaliteit van geëxporteerde, bewerkte afbeeldingen.

### Gekalibreerd afbeeldingsformaat

* **Type**: Keuzelijst
* **Opties**:
  * **TIFF

(16-bit)** - Ongecomprimeerd 16-bits TIFF

-formaat
  * **TIFF

(8-bit)** - Ongecomprimeerd 8-bit TIFF

-formaat
  * **TIFF

(32-bit, Percent)** - 32-bit TIFF

met drijvende komma en reflectiewaarden in percentages
  * **PNG

(8-bit)** - Gecomprimeerd 8-bit PNG

-formaat
  * **JPG (8-bit)** - Gecomprimeerd 8-bit JPEG

-formaat
* **Standaard**: TIFF

(16-bit); bij een volledig 8-bit-project wordt TIFF

(8-bit) weergegeven en worden alleen de 8-bit-formaten aangeboden
* **Beschrijving**: Hiermee selecteert u het bestandsformaat voor het opslaan van bewerkte en gekalibreerde afbeeldingen. De geëxporteerde bestanden worden opgeslagen in een submap per formaat binnen de map van elke camera (`tiff16`, `tiff32`, `png8`, `jpg8`), met één `<Product>_Images/`-map per product. Geëxporteerde bestanden behouden de bronbestandsnaam — de map, niet een bestandsextensie, identificeert het product.
* **Aanbevelingen voor bestandsformaten**:
  * **TIFF

(16-bits)**: Aanbevolen voor wetenschappelijke analyse en professionele workflows. Behoudt maximale gegevenskwaliteit zonder compressieartefacten. Het meest geschikt voor multispectrale analyse en verdere verwerking in GIS-software.
  * **TIFF

(32-bit, Percent)**: Het meest geschikt voor workflows waarbij reflectiewaarden als percentages (0-100%) nodig zijn. Biedt maximale precisie voor radiometrische metingen.
  * **PNG

(8-bit)**: Geschikt voor weergave op het web en algemene visualisatie. Kleinere bestandsgroottes met verliesloze compressie, maar beperkt dynamisch bereik.
  * **JPG (8-bit)**: Kleinste bestandsgroottes, alleen geschikt voor voorbeelden en weergave op het web. Maakt gebruik van compressie met verlies, wat niet geschikt is voor wetenschappelijke analyse.
* **Opmerking**: LATTICE-radiance wordt altijd geëxporteerd als 32-bit float-TIFF

, ongeacht deze instelling.

***

## Projectsjabloon opslaan

Met deze functie kun je je huidige projectinstellingen opslaan als een herbruikbaar sjabloon.

* **Type**: Tekstinvoer + knop ‘Opslaan’
* **Beschrijving**: Voer een beschrijvende naam in voor je instellingssjabloon en klik op het opslagpictogram. Het sjabloon slaat al je huidige projectinstellingen op (doeldetectie, verwerkingsopties, indexen, en exportformaat) op, zodat u deze in toekomstige projecten gemakkelijk kunt hergebruiken. Sjablonen worden opgeslagen in de map `Project Templates/` binnen de opslagmap van uw project, en kunnen ook worden geselecteerd of geëxporteerd vanuit het hoofdmenu (*Sjabloon selecteren* / *Sjabloon opslaan* / *Sjabloon exporteren*).
* **Toepassingen**:
  * Maak sjablonen voor verschillende camerasystemen (RGB

, multispectraal, NIR

)
  * Sla standaardconfiguraties op voor specifieke gewassoorten of analyseworkflows
  * Deel consistente instellingen binnen een team
* **Gebruiksaanwijzing**:
  1. Configureer alle gewenste projectinstellingen
  2. Voer een sjabloonnaam in (bijv. „RedEdge

Survey3

NDVI

Standaard”)
  3. Klik op het opslagpictogram
  4. Het sjabloon kan nu worden geladen bij het aanmaken van nieuwe projecten

***

## Projectmap opslaan

Deze instelling bepaalt waar nieuwe projecten standaard worden opgeslagen.

* **Type**: Weergave van het map pad + knop ‘Bewerken’
* **Standaard (Windows

)**: `Documents\MAPIR\Chloros Projects` in uw map ‘Documenten’ — meestal `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, of de locatie waar OneDrive of een roamingprofiel de map ‘Documenten’ naartoe heeft verplaatst
* **Standaard (Linux

)**: `~/Documents/MAPIR/Chloros Projects`
* **Beschrijving**: Toont de huidige standaardmap waarin nieuwe Chloros

-projecten worden aangemaakt. Klik op het bewerkingspictogram om een andere map te selecteren. De aangepaste instelling wordt opgeslagen als één regel tekst in `~/.chloros/working_directory.txt` — in Windows

is dat `C:\Users\<Username>\.chloros\working_directory.txt`. Als dat bestand ontbreekt of een pad aangeeft dat niet meer bestaat, valt Chloros

terug op de hierboven genoemde standaardinstelling. De CLI

leest en schrijft hetzelfde bestand, zodat `chloros-cli` en de GUI het altijd eens zijn over de locatie van projecten.
* **Projectsjablonen** bevinden zich in een submap `Project Templates/` van deze map.
* **Wanneer wijzigen**:
  * Stel een netwerkschijf in voor teamsamenwerking
  * Wijzig naar een schijf met meer opslagruimte voor grote datasets
  * Organiseer projecten per jaar, klant of projecttype in verschillende mappen
* **Opmerking**: Het wijzigen van deze instelling heeft alleen invloed op NIEUWE projecten. Bestaande projecten blijven op hun oorspronkelijke locaties staan.
* **Lengte van Windows

-paden**: tenzij lange Windows

-paden zijn ingeschakeld (de optie **Enable Windows

long paths** van het installatieprogramma), wordt een projectmap met een padlengte van meer dan ongeveer 130 tekens vóór de verwerking geweigerd, omdat de resulterende paden de limiet van 260 tekens van Windows

zouden overschrijden. Zie [De verwerking starten](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Instellingen bewaren

Een Chloros

-project is een **map**. Alle projectinstellingen worden daarin opgeslagen in `project.json`; aangesloten hardware wordt samen daarmee onthouden in `cameras.json` en `sensors.json`, zodat bij het opnieuw openen van een project ook de camera’s en lichtsensoren opnieuw worden aangesloten. Wanneer u een project opnieuw opent, worden alle instellingen precies zo hersteld als u ze had achtergelaten. Opgeslagen projecten kunnen ook zonder beeldscherm worden aangestuurd met `chloros-cli project` of `open_project` uit de SDK

.

### Hiërarchie van instellingen

Instellingen worden in de volgende volgorde toegepast:

1. **Systeemstandaarden** - Ingebouwde standaardinstellingen gedefinieerd door Chloros

2. **Sjablooninstellingen** - Als je een sjabloon laadt bij het aanmaken van een project
3. **Opgeslagen projectinstellingen** - Instellingen die samen met het projectbestand zijn opgeslagen
4. **Handmatige aanpassingen** – Alle wijzigingen die je tijdens de huidige sessie aanbrengt

### Instellingen en beeldverwerking

Verwerkingsinstellingen worden ingelezen wanneer een verwerkingsrun start. Het wijzigen van een instelling heeft geen terugwerkende kracht op producten die al op de schijf staan — voer de verwerking opnieuw uit om de nieuwe instellingen toe te passen. Een aantal instellingen heeft helemaal geen invloed op de verwerking:

* Resolutie miniatuurafbeeldingen (alleen voor weergave)
* Projectsjabloon opslaan
* Projectmap opslaan

***

## Overzicht van configuratiesleutels

Voor automatisering (CLI

`--config`, SDK

`configure` of het rechtstreeks inlezen van `project.json`) zijn dit de exacte sleutels onder `Project Settings`:

| Sleutelpad | Type | Standaard |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | getal 0-10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | getal 0-100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standaard |
| `Processing → Minimum recalibration interval` | getal 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | getal -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | cap-profiel-id of `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | lijst met indexconfiguraties | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

De `Array alignment`-sleutels worden geschreven zodra de sectie ‘Array Alignment’ voor het eerst wordt weergegeven of wanneer een automatiseringsaanroep ze instelt. Zolang ze ontbreken, gebruikt de pijplijn dezelfde waarden als hierboven weergegeven (`true`, `true`, bilineair), dus een project.json zonder deze sleutels gedraagt zich identiek aan een project met deze sleutels.

### Sleutels opgeslagen in `project.json` zonder bedieningselementen in het instellingenpaneel

Deze bevinden zich onder dezelfde `Project Settings`-structuur en worden door de verwerking gelezen, maar je zult er geen widget voor vinden in de zijbalk:

| Sleutelpad | Type | Standaard | Ingesteld door |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK

`input_level=`. Overschrijft de manier waarop LATTICE-invoer-TIFF

’s worden geïnterpreteerd; `auto` leidt dit af uit de `Chloros:ProcessingLevel` XMP-tag van elk bestand plus het aantal kanalen. Wordt genegeerd voor Survey3

-opnames. Dit is bewust geen GUI-instelling — ‘auto’ is in alle normale gevallen correct. |
| `Processing → Target reflectance dir` | padstring | `""` | `chloros-cli process --target-reflectance-dir`, of de projectdoel-API

|
| `Processing → Target reflectance config` | woordenboek met cameraserienummer als sleutel | `{}` | Registreren van een doel binnen het beeld (modus `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | padstring | `""` | SDK

`process_folder(daq_log_path=…)`. Verwijst naar een `.daq`-opname of een map met dergelijke opnames |
| `Target Detection → Minimum calibration target squares` | getal | `4` | Oude standaardinstelling; geen besturing en geen CLI

-vlag |
| `UI → Grid thumbnail size` | getal | `160` | De eigen miniatuurzoomschuifregelaar van het afbeeldingsraster |

Twee viewer-voorkeuren worden **op het hoogste niveau in `project.json`** opgeslagen, volledig buiten `Project Settings` om, omdat het om weergavestatus gaat in plaats van verwerkingsinstellingen:

| Sleutelpad | Type | Standaard | Ingesteld door |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | geheel getal 1–256 | `1` | De GSD (px)-regelaar van het afbeeldingstabblad — zie [Een afbeelding op volledig scherm openen](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Aanbevolen werkwijzen

1. **Begin met de standaardinstellingen**: De standaardinstellingen werken goed voor de meeste MAPIR

-camerasystemen en gangbare workflows.
2. **Maak sjablonen**: Zodra je de instellingen voor een specifieke workflow of camera hebt geoptimaliseerd, sla je deze op als sjabloon om consistentie tussen projecten te waarborgen.
3. **Test voordat je de volledige verwerking start**: Probeer bij het experimenteren met nieuwe instellingen deze eerst uit op een kleine subset van afbeeldingen voordat je je volledige dataset verwerkt.
4. **Documenteer je instellingen**: Gebruik beschrijvende sjabloonnamen die het camerasysteem, het verwerkingstype en het beoogde gebruik aangeven (bijv. „Survey3

\_RGB\_NDVI\_Agriculture”).
5. **Keuze van exportformaat**: Kies je exportformaat op basis van het eindgebruik:
   * Wetenschappelijke analyse → TIFF (16-bit of 32-bit)
   * GIS-verwerking → TIFF

(16-bit)
   * Snelle visualisatie → PNG

(8-bit)
   * Delen via het web → JPG (8-bit)

***

Zie de pagina [Formules voor multispectrale indices](multispectral-index-formulas.md) voor meer informatie over multispectrale indices in Chloros

.
