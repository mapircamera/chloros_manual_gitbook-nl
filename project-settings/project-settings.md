# Projectinstellingen

Via de zijbalk ‘Projectinstellingen’<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

in Chloros kunt u alle aspecten van beeldverwerking, detectie van kalibratiedoelpunten, berekeningen van multispectrale indexen en exportopties voor uw project configureren. Deze instellingen worden samen met uw project opgeslagen en kunnen als sjablonen worden opgeslagen voor hergebruik in meerdere projecten.

## Projectinstellingen openen

Om de projectinstellingen te openen:

1. Open een project in Chloros
2. Klik in de linkerzijbalk op het tabblad **Projectinstellingen**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">


3. Het instellingenpaneel toont alle beschikbare configuratieopties, ingedeeld per categorie

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Instellingen die afhankelijk zijn van andere instellingen, zijn grijs weergegeven.** Wanneer een bovenliggende schakelaar een instelling onmogelijk maakt (bijvoorbeeld: als je *Reflectanciekalibratie / witbalans* uitschakelt, is *Reflectantie exporteren* niet meer mogelijk), wordt de afhankelijke schakelaar uitgeschakeld en wordt in de tooltip vermeld welke schakelaar moet worden gewijzigd.
{% endhint %}

***

## Weergave

### Resolutie van afbeeldingsminiaturen

* **Type**: keuzelijst
* **Opties**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Standaard**: Standaard (512 px)
* **Beschrijving**: Resolutie (langste zijde, in pixels) waarmee de miniaturen in het afbeeldingsraster worden weergegeven. Hogere waarden zien er scherper uit bij inzoomen, maar laden langzamer en gebruiken meer geheugen. Volledige resolutie komt overeen met de oorspronkelijke afbeeldingsgrootte.
* **Opmerking**: Alleen ter weergave — dit heeft nooit invloed op de verwerking of geëxporteerde bestanden.***

## Doelherkenning

Deze instellingen bepalen hoe Chloros kalibratiedoelen in uw afbeeldingen detecteert en verwerkt. Beide zijn alleen actief wanneer **Reflectiekalibratie / witbalans** is ingeschakeld (anders zijn ze grijs weergegeven, omdat doelherkenning dan volledig wordt overgeslagen).

### Minimaal kalibratiesamplegebied (px)

* **Type**: Getal
* **Bereik**: 0 tot 10.000 pixels
* **Standaard**: 25 pixels
* **Beschrijving**: Stelt het minimale gebied (in pixels) in dat nodig is om een gedetecteerd gebied als een geldig kalibratiedoel te beschouwen. Kleinere waarden detecteren kleinere doelen, maar kunnen het aantal valse positieven verhogen. Grotere waarden vereisen grotere, duidelijkere doelgebieden voor detectie.
* **Wanneer aanpassen**:
  * Verhoog deze waarde als u valse detecties krijgt bij kleine beeldartefacten
  * Verlaag deze waarde als uw kalibratiedoelen klein lijken in uw beelden en niet worden gedetecteerd

### Minimale doelclustering (0-100)

* **Type**: Getal
* **Bereik**: 0 tot 100
* **Standaard**: 60
* **Beschrijving**: Bepaalt de clusteringsdrempel voor het groeperen van gebieden met vergelijkbare kleuren bij het detecteren van kalibratiedoelen. Bij hogere waarden moeten meer vergelijkbare kleuren worden gegroepeerd, wat resulteert in een conservatievere detectie van doelen. Lagere waarden staan meer kleurvariatie toe binnen een doelgroep.
* **Wanneer aanpassen**:
  * Verhoog deze waarde als kalibratiedoelen in meerdere detecties worden opgesplitst
  * Verlaag deze waarde als kalibratiedoelen met kleurvariatie niet volledig worden gedetecteerd

***

## Verwerking

Deze instellingen bepalen hoe Chloros uw beelden verwerkt en kalibreert.

### Vignetteringscorrectie

* **Type**: Selectievakje
* **Standaard**: Ingeschakeld (aangevinkt)
* **Beschrijving**: Past vignetteringscorrectie toe om de lensverduistering aan de randen van beelden te compenseren. Vignettering is een veelvoorkomend optisch verschijnsel waarbij de hoeken en randen van een beeld donkerder lijken dan het midden als gevolg van de eigenschappen van de lens.
* **Neveneffect**: Deze schakelaar selecteert ook welk *niet-gekalibreerd fallback-product* een run schrijft (zie hieronder).

### Reflectanciekalibratie / witbalans

* **Type**: Selectievakje
* **Standaard**: Ingeschakeld (aangevinkt)
* **Beschrijving**: Schakelt reflectanciekalibratie in — op basis van gedetecteerde kalibratiedoelen binnen het beeld en/of downwelling-gegevens van de DAQ-lichtsensor, afhankelijk van de camera en wat beschikbaar is. Dit normaliseert de reflectiewaarden in uw dataset en zorgt voor consistente metingen, ongeacht de lichtomstandigheden.
* **Wanneer uitgeschakeld**: Doeldetectie wordt volledig overgeslagen en**er kan door geen enkele camera een reflectieproduct worden geproduceerd** — zowel bij Survey3-doelgestuurde als bij LATTICE DAQ-gestuurde systemen. De afhankelijke instellingen (*Reflectantie exporteren*, *Minimaal herkalibratie-interval* en de drempelwaarden voor doelherkenning) zijn grijs weergegeven.

### Niet-gekalibreerde fallback-producten: Sensorrespons exporteren / Vignettering gecorrigeerd exporteren

* **Type**: Twee selectievakjes
* **Standaardinstellingen**: Beide ingeschakeld (aangevinkt)
* **Beschrijving**: Wanneer een frame niet op reflectie kan worden gekalibreerd (er is geen kalibratiedoel gevonden of de reflectiekalibratie is uitgeschakeld), wordt het in plaats daarvan opgeslagen als een *niet-gekalibreerd fallback-product*. **Per run bestaat er precies één van de twee fallback-producten, voor elk cameramodel**, gekozen door de schakelaar *Vignettecorrectie*:
  * Vignetcorrectie **aan**→ `Vignette_Corrected_Images/` (bepaald door**Vignetcorrectie exporteren**)
  * Vignetcorrectie **uit**→ `Sensor_Response_Images/` (bepaald door**Sensorrespons exporteren**)
* De fallback die niet actief is, wordt grijs weergegeven. Als je het vinkje bij de actieve optie verwijdert, wordt dat bestand helemaal niet opgeslagen.

### LATTICE-exportproducten

Bij projecten met LATTICE-opnames wordt elk geïmporteerd LATTICE-frame in één verwerkingsronde uitgesplitst naar elk ingeschakeld **en toepasselijk**product. Vier selectievakjes regelen de uitsplitsing (standaard allemaal**ingeschakeld**):

| Instelling | Uitvoermap | Wat er wordt geëxporteerd |
| --- | --- | --- |
| **Exporteren met debayering** | `Debayered_Images/` | Het lineaire beeld zonder bayering. Geldt voor RGB en multispectrale camera’s. |
| **Voorbeeld exporteren** | `Preview_Images/` | Het weergavevoorbeeld. RGB = witbalans (DAQ-lichtbron indien beschikbaar, anders grijstinten) + gamma; multispectraal = valse-kleurenuitrekking. |
| **Stralingsintensiteit exporteren** | `Radiance_Images/` | Float32 spectrale stralingsintensiteit in W/m²/sr/nm. Alleen multispectraal (M3C/M3M) — niet van toepassing op RGB-masters. Wordt altijd geschreven als 32-bits TIFF, ongeacht de instelling *Gekalibreerd beeldformaat*. |
| **Exportreflectantie**| `Reflectance_Calibrated_Images/` | Uint16-reflectantie, geschaald zodat**32768 = reflectantie 1,0** (gemarkeerd als XMP `Chloros:PixelScale`). Alleen multispectraal, geschreven wanneer een bijpassend `.daq`-downwelling-record (of een QA-goedgekeurd doel binnen het frame) het frame bestrijkt. |

* RGB-hoofdcamera’s zenden debayered + preview uit; straling/reflectantie worden hiervoor overgeslagen omdat ze niet van toepassing zijn.
* De bitdiepte van debayered/preview volgt de instelling *Gekalibreerd beeldformaat*; straling is altijd float32.
* Survey3-verwerking wordt niet beïnvloed door deze vier schakelaars.

Dezelfde vier schakelaars bestaan in headless-vorm als `chloros-cli process --debayered / --preview / --radiance / --reflectance` en als de SDK&#x27;met overeenkomende parameters. Ze hebben de oude vlag `--radiometric-output` vervangen, die niet langer bestaat.

{% hint style="warning" %}
**Als je elk van toepassing zijnd product uitschakelt, mislukt de run.** Vanaf versie 1.2.0 meldt een verwerkingsrun die om producten werd gevraagd maar geen beeldproducten heeft geschreven een fout, en de CLI wordt afgesloten met een waarde anders dan nul, in plaats van een stilzwijgend succes te melden. Het logboek vermeldt het product dat niet kon worden geschreven en de reden daarvoor. Een opzettelijk uitsluitend op metagegevens gerichte run (niets aangevraagd) is nog steeds een succes.
{% endhint %}

### Reflectantiebron (projectinstelling, ingesteld via CLI/SDK)

Het project slaat ook op welke **reflectantierferentie** het LATTICE-reflectantieproduct gebruikt. Er is geen specifieke instelling in het instellingenpaneel; de waarde wordt opgeslagen in de projectconfiguratie als `Processing → "Target reflectance source"` en wordt ingesteld met `chloros-cli process --reflectance-source {auto,target,daq}` of de SDK’s `reflectance_source`-parameter:

* **`auto`** (standaard): een QA-goedgekeurd kalibratiedoel binnen het beeld wordt de absolute referentie; bij afwezigheid van een doel of bij mislukking van de QA wordt teruggevallen op de DAQ-downwelling-divident (ρ = πL/E) wanneer er geen doel aanwezig is of de kwaliteitscontrole mislukt.
* **`target`**: strikt doelgestuurde reflectantie — geen DAQ-vervanging.
* **`daq`**: reflectantie op basis van DAQ-gegevens; doelen binnen het beeld worden niet als referentie gebruikt.

De opgeslagen waarde wordt gevergeleken zonder onderscheid tussen hoofdletters en kleine letters, en enkele spellingsvarianten worden als aliassen geaccepteerd: `target`, `target_image`, `empirical` en `empirical_line` betekenen allemaal **doel**; `daq`, `dls`, `light_sensor` en `sensor` betekenen allemaal**daq**. Al het andere — inclusief een ontbrekende toets — wordt omgezet naar**auto**.**Gemeten** doelscans per eenheid worden opgezocht aan de hand van het serienummer/QR-code van de doeleenheid, als `<serial>.csv`, op drie plaatsen: de map die is opgegeven met `--target-reflectance-dir` (opgeslagen als `Processing → "Target reflectance dir"`), de eigen map `target_reflectance/` van het project en het pad in de omgevingsvariabele `CHLOROS_TARGET_REFLECTANCE_DIR`. Als er geen gemeten scan voor die eenheid bestaat, wordt in plaats daarvan de nominale gepubliceerde curve voor het doelmodel gebruikt.

### Debayer-methode

* **Type**: Keuzelijst
* **Opties**:
  * Standaard (Snel, gemiddelde kwaliteit)
  * Textuurbewust (Langzaam, hoogste kwaliteit) \[Chloros+]
* **Standaard**: Standaard (Snel, gemiddelde kwaliteit)
* **Beschrijving**: Selecteert het demosaicing-algoritme dat wordt gebruikt om ruwe Bayer-patroonsensorgegevens om te zetten in full-colour beelden. De methode „Standaard (Snel, gemiddelde kwaliteit)“ biedt een optimale balans tussen verwerkingssnelheid en beeldkwaliteit. De methode „Textuurbewust (Langzaam, hoogste kwaliteit)“ \[Chloros+] maakt gebruik van een hoogwaardige, randbewuste debayering in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayeringruis verwijdert. Het Textuurbewuste model vereist GPU-geheugen (VRAM) nodig om te draaien. We raden aan dit te gebruiken wanneer je &gt;4 GB VRAM beschikbaar hebt voor een snellere verwerking.
* **Als de rij überhaupt een keuzelijst is**: de keuzelijst met twee opties verschijnt alleen wanneer**beide**voorwaarden gelden — je bent aangemeld met een in aanmerking komend Chloros+-abonnement,**en** het project geen LATTICE-opnames bevat. Anders wordt de rij weergegeven als platte tekst met de tekst `Standard (Fast, Medium Quality)`, zonder keuzemogelijkheid.
* **Opmerking over LATTICE**: Er is geen door LATTICE getraind ‘Texture Aware’-model, en de verwerkingspijplijn dwingt de standaard demosaic af voor LATTICE-frames, ongeacht de opgeslagen waarde. Als je een LATTICE-map toevoegt aan een project waarin ‘Texture Aware’ al was geselecteerd, schrijft Chloros de instelling terug naar ‘Standaard’ in plaats van een verouderde waarde achter te laten in `project.json`.

### Minimaal herkalibratie-interval

* **Type**: Getal
* **Bereik**: 0 tot 3.600 seconden
* **Standaard**: 0 seconden
* **Beschrijving**: Stelt het minimale tijdsinterval (in seconden) in tussen het gebruik van kalibratiedoelen. Wanneer ingesteld op 0, zal Chloros elk gedetecteerd kalibratiedoel gebruiken. Wanneer ingesteld op een hogere waarde, zal Chloros alleen kalibratiedoelen gebruiken die ten minste dit aantal seconden van elkaar verwijderd zijn, waardoor de verwerkingstijd wordt verkort voor datasets met frequente registraties van kalibratiedoelen.
* **Wanneer aanpassen**:
  * Stel in op 0 voor maximale kalibratienauwkeurigheid wanneer de lichtomstandigheden variëren
  * Verhoog de waarde (bijv. naar 60-300 seconden) voor snellere verwerking wanneer het licht constant is en er regelmatig beelden van kalibratiedoelen worden vastgelegd

### Tijdzone-afwijking lichtsensor

* **Type**: Getal
* **Bereik**: -12 tot +12 uur
* **Standaard**: 0 uur
* **Beschrijving**: Specificeert de tijdzone-offset (in uren ten opzichte van UTC) voor tijdstempels van lichtsensorgegevens, die wordt gebruikt bij het koppelen van lichtsensorlogs aan de opnametijden van beelden. Nieuwere `.daq`-opnames bevatten hun eigen tijdzone-herkomst, dus dit is vooral nodig voor oudere logs die in lokale tijd zijn opgenomen.

### PPK-correcties toepassen

* **Type**: Selectievakje
* **Standaard**: Uitgeschakeld (niet aangevinkt)
* **Beschrijving**: Schakelt het gebruik van Post-Processed Kinematic (PPK)-correcties in van MAPIR DAQ-recorders die zijn uitgerust met een GPS (GNSS). Wanneer deze optie is ingeschakeld, zal Chloros alle .daq-logbestanden met belichtingspin-gegevens in uw projectmap gebruiken en nauwkeurige geolocatiegecorrigeringen toepassen op uw beelden.
* **Vereiste**: Er moet een .daq-logbestand met belichtingspin-vermeldingen aanwezig zijn in uw projectmap
* **Wanneer inschakelen**: Het wordt aanbevolen om PPK-correctie altijd in te schakelen als u belichtingsfeedbackgegevens in uw .daq-logbestand hebt.

### Belichtingspin 1

* **Type**: Keuzelijst
* **Zichtbaarheid**: Alleen zichtbaar wanneer „PPK-correcties toepassen“ is ingeschakeld EN er belichtingsgegevens beschikbaar zijn voor pin 1
* **Opties**:
  * Namen van cameramodellen die in het project zijn gedetecteerd
  * „Niet gebruiken“ – Deze belichtingspin negeren
* **Standaard**: Wordt automatisch geselecteerd op basis van de projectconfiguratie
* **Beschrijving**: Wijst een specifieke camera toe aan belichtingspin 1 voor PPK-tijdsynchronisatie. De belichtingspin registreert het exacte tijdstip waarop de camerasluiter wordt geactiveerd, wat cruciaal is voor nauwkeurige PPK-geolocatie.
* **Gedrag bij automatische selectie**:
  * Eén camera + één pin: selecteert automatisch de camera
  * Eén camera + twee pinnen: pin 1 wordt automatisch aan de camera toegewezen
  * Meerdere camera’s: handmatige selectie vereist

### Belichtingspin 2

* **Type**: Keuzelijst
* **Zichtbaarheid**: Alleen zichtbaar wanneer „PPK-correcties toepassen“ is ingeschakeld EN er belichtingsgegevens beschikbaar zijn voor pin 2
* **Opties**:
  * Namen van cameramodellen die in het project zijn gedetecteerd
  * „Niet gebruiken“ – Deze belichtingspin negeren
* **Standaard**: Wordt automatisch geselecteerd op basis van de projectconfiguratie
* **Beschrijving**: Wijst een specifieke camera toe aan belichtingspin 2 voor PPK-tijdsynchronisatie bij gebruik van een opstelling met twee camera’s.
* **Gedrag bij automatische selectie**:
  * Eén camera + één pin: Pin 2 wordt automatisch ingesteld op „Niet gebruiken“
  * Eén camera + twee pinnen: Pin 2 wordt automatisch ingesteld op „Niet gebruiken“
  * Meerdere camera’s: Handmatige selectie vereist
* **Opmerking**: Dezelfde camera kan niet tegelijkertijd aan zowel Pin 1 als Pin 2 worden toegewezen.***

## DAQ-lichtsensor

Dit gedeelte verschijnt in de projectinstellingen en geeft een overzicht van alle DAQ-downwelling-bestanden in het project — `.daq`-opnames en DAQ-M `.csv`-downwelling-logs. Opnames die zijn gemaakt op het tabblad ‘Lichtsensoren’ worden automatisch toegevoegd aan het geopende project.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Elke rij toont het bestand, het sensormodel en de correctie voor de diffusorkap die daadwerkelijk van kracht is voor dat bestand. Boven de rijen bevindt zich één projectbrede instelling:

### Kap-override (alle bestanden)

* **Type**: keuzelijst
* **Opties**: `Auto` plus de cap-correctieprofielen die geldig zijn voor de in het project aanwezige sensortypen
* **Standaard**: Auto
* **Opgeslagen als**: `Processing → "DAQ cap id"` (standaard `auto`)
* **Beschrijving**: `Auto` gebruikt de geregistreerde kap van elk bestand (de ‘Sunshine’-cap wordt verondersteld wanneer er niets is geregistreerd — alle MAPIR DAQ’s worden geleverd met de ‘Sunshine’-corrector). Het selecteren van een specifieke cap overschrijft**elk** downwelling-bestand in het project: ruwe opnames worden hiermee gecorrigeerd, en opnames die al een cap bevatten, worden opnieuw gerefereerd (de opgeslagen correctie wordt ongedaan gemaakt en de geselecteerde correctie wordt toegepast).
* **Belangrijk**: De geselecteerde kap moet overeenkomen met de kap die fysiek tijdens de opname was aangebracht. Noch de sensor, noch de software kan de fysieke kap detecteren — een verkeerd overeenkomend cap-ID leidt tot een onjuiste correctie van de spectra.

Er is bewust gekozen voor **één** projectbrede instelling in plaats van dropdownmenu’s per bestand: de instelling geldt voor elke neerwaartse bron in het project.***

## Uitlijning van arrays

Dit gedeelte verschijnt **alleen** wanneer ten minste één afbeelding in het project de module-naar-module uitlijningstransformatie bevat die LATTICE-arrays bij het vastleggen aanbrengen (XMP `Chloros:Alignment*`-tags). Hier wordt weergegeven hoeveel afbeeldingen uitlijningstags bevatten, welke camera de referentie is (`REF`-badge) en een tabel met het aantal afbeeldingen per camera.

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
* **Beschrijving**: Interpolatie die wordt gebruikt door de uitlijningsvervorming. *Nearest* behoudt de exacte bronwaarden (geen vermenging tussen pixels) voor strikte radiometrische analyse; ‘Bilineair’ is het meest geschikt voor mapping en visueel gebruik.

Dezelfde drie opties bestaan zonder voorvoegsel als `chloros-cli process --array-alignment`, `--array-alignment-crop` en `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Met deze instellingen kunt u multispectrale indexen configureren voor analyse en visualisatie.

### Index toevoegen

* **Type**: Speciaal configuratiepaneel voor indexen
* **Beschrijving**: Opent een interactief paneel waarin u multispectrale vegetatie-indexen (NDVI, NDRE, EVI, enz.) die tijdens de beeldverwerking moeten worden berekend. U kunt meerdere indexen toevoegen, elk met hun eigen visualisatie-instellingen.
* **Beschikbare indexen**: De vervolgkeuzelijst in de GUI bevat**27** vooraf gedefinieerde multispectrale indexformules (zie [Multispectrale indexformules](multispectral-index-formulas.md) voor de volledige lijst, inclusief welke namen ook worden geaccepteerd door de CLI/SDK `--indices`-optie) worden geaccepteerd.
* **Functies**:
  * Kies uit vooraf gedefinieerde indexformules
  * Sleep de filterkanalen van je camera naar de bandvakken van de formule
  * Configureer kleurverloop voor visualisatie (LUT - Look-Up Tables)
  * Stel drempelwaarden en clippingmodi in
  * Maak aangepaste indexformules
* **Opmerking**: Er worden geen indexen berekend voor LATTICE M3M-monochromecamera’s met één band — multibandindexen zijn ongedefinieerd bij één band. Survey3 en LATTICE M3C worden hierdoor niet beïnvloed.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Elke index die u toevoegt, geeft de formule weer als wiskundige uitdrukking, met een gekleurde cirkel per bandvak: rood = Red, groen = Green, blauw = Blue, oranje = Orange, cyaan = Cyan, paars = NIR, magenta = RE. Sleep een cirkel vanuit de rij boven de formule naar een slot om deze te koppelen; dubbelklik op een gekoppeld slot om de koppeling te verwijderen. De index wordt slechts één keer berekend zodra elk slot dat de formule gebruikt een kanaal heeft.

### Aangepaste formules (Chloros+ functie)

* **Type**: Reeks definities van aangepaste formules
* **Beschikbaarheid**: Vereist aanmelding met een in aanmerking komend Chloros+-abonnement.
* **Beschrijving**: Hiermee kun je aangepaste multispectrale indexformules maken en opslaan met behulp van bandberekeningen. Aangepaste formules worden samen met je projectinstellingen opgeslagen en kunnen net als ingebouwde indexen worden gebruikt.
* **Zo maak je ze aan**:
  1. Open in het configuratiepaneel ‘Index’ de rekenmachine voor aangepaste formules
  2. Schrijf de formule met behulp van de **bandslotsymbolen**, niet de bandnamen
  3. Sla de formule op met een beschrijvende naam — deze verschijnt dan onderaan de formule-dropdown, en u sleept de kanaalcirkels van uw camera naar de slots, precies zoals bij een ingebouwdeingebouwde voorinstelling
* **Syntaxis van de formule**:
  * Bandvakken: `x`, `y`, `z`, `a`, `b`, `c` — zes posities die je aan echte kanalen toewijst door ze te verslepen
  * Operatoren: `+`, `-`, `*`, `/`, `^` en `()` voor groepering
  * Functies: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Waarom symbolen in plaats van bandnamen**: een formule die is geschreven als `(y-x)/(y+x)` werkt op elke camera, omdat de-en-sleep-toewijzing bepaalt of `y` de 850 nm NIR van een RGN-filter is of de 808 nm NIR van een OCN-filter. De ingebouwde voorinstellingen worden op dezelfde manier opgeslagen — zie [Multispectrale indexformules](multispectral-index-formulas.md) voor de exacte symboolvorm van alle 27.
* **Waar ze werken**: aangepaste formules worden samen met de projectinstellingen opgeslagen en kunnen zowel in de [Index/LUT-sandbox](../image-viewer-gui/index-lut-sandbox.md) als tijdens de verwerking worden gebruikt. Ze worden**niet** geaccepteerd door de CLI/SDK `--indices`-naamlijst, die alleen de 22 ingebouwde voorinstelnamen uitbreidt.***

## Exporteren

Deze instellingen bepalen het formaat en de kwaliteit van geëxporteerde, bewerkte afbeeldingen.

### Gekalibreerd afbeeldingsformaat

* **Type**: Keuzelijst
* **Opties**:
  * **TIFF (16-bits)** - Ongecomprimeerd 16-bits TIFF-formaat
  * **TIFF (32-bit, Percent)** - 32-bits drijvende-kommapunt TIFF met reflectiewaarden als percentages
  * **PNG (8-bits)** - Gecomprimeerd 8-bits PNG-formaat
  * **JPG (8-bits)** - Gecomprimeerd 8-bits JPEG-formaat
* **Standaard**: TIFF (16-bits)
* **Beschrijving**: Selecteert het bestandsformaat voor het opslaan van bewerkte en gekalibreerde afbeeldingen. De geëxporteerde bestanden worden opgeslagen in een submap per formaat binnen de map van elke camera (`tiff16`, `tiff32`, `png8`, `jpg8`), met één `<Product>_Images/`-map per product. Geëxporteerde bestanden behouden de bronbestandsnaam — de map, niet een bestandsextensie, identificeert het product.
* **Aanbevelingen voor het formaat**:
  * **TIFF (16-bits)**: Aanbevolen voor wetenschappelijke analyse en professionele workflows. Behoudt maximale gegevenskwaliteit zonder compressieartefacten. Het meest geschikt voor multispectrale analyse en verdere verwerking in GIS-software.
  * **TIFF (32-bit, procent)**: Het meest geschikt voor workflows waarbij reflectiewaarden als percentages (0-100%) vereist zijn. Biedt maximale precisie voor radiometrische metingen.
  * **PNG (8-bit)**: Geschikt voor weergave op het web en algemene visualisatie. Kleinere bestandsgroottes met verliesloze compressie, maar een beperkt dynamisch bereik.
  * **JPG (8-bit)**: Kleinste bestandsgroottes, alleen geschikt voor voorbeelden en weergave op het web. Maakt gebruik van compressie met verlies, wat niet geschikt is voor wetenschappelijke analyse.
* **Opmerking**: LATTICE-radiance wordt altijd geëxporteerd als 32-bits float TIFF, ongeacht deze instelling.***

## Project-sjabloon opslaan

Met deze functie kunt u uw huidige projectinstellingen opslaan als een herbruikbaar sjabloon.

* **Type**: Tekstinvoer + knop ‘Opslaan’
* **Beschrijving**: Voer een beschrijvende naam in voor uw instellingensjabloon en klik op het opslagpictogram. Het sjabloon slaat al uw huidige projectinstellingen op (doeldetectie, verwerkingsopties, indexen en exportformaat) op, zodat u deze in toekomstige projecten gemakkelijk opnieuw kunt gebruiken. Sjablonen worden opgeslagen in de map `Project Templates/` binnen de opslagmap van uw project, en kunnen ook worden geselecteerd of geëxporteerd vanuit het hoofdmenu (*Sjabloon selecteren* / *Sjabloon opslaan* / *Sjabloon exporteren*).
* **Toepassingen**:
  * Maak sjablonen voor verschillende camerasystemen (RGB, multispectraal, NIR)
  * Sla standaardconfiguraties op voor specifieke gewassoorten of analyseworkflows
  * Deel consistente instellingen binnen een team
* **Gebruiksaanwijzing**:
  1. Configureer alle gewenste projectinstellingen
  2. Voer een sjabloonnaam in (bijv. „RedEdge Survey3 NDVI Standaard”)
  3. Klik op het opslagpictogram
  4. De sjabloon kan nu worden geladen bij het aanmaken van nieuwe projecten

***

## Projectmap opslaan

Deze instelling bepaalt waar nieuwe projecten standaard worden opgeslagen.

* **Type**: Weergave van het map pad + knop ‘Bewerken’
* **Standaard (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Standaard (Linux)**: `~/Chloros Projects`
* **Beschrijving**: Toont de huidige standaardmap waarin nieuwe Chloros-projecten worden aangemaakt. Klik op het bewerkingspictogram om een andere map te selecteren. De overschrijving wordt opgeslagen als een enkele regel tekst in `~/.chloros/working_directory.txt` — in Windows, namelijk `C:\Users\<Username>\.chloros\working_directory.txt`. Als dat bestand ontbreekt of een pad aangeeft dat niet meer bestaat, valt Chloros terug op de bovenstaande standaardinstelling. CLI leest en schrijft hetzelfde bestand, dus `chloros-cli` en de GUI zijn het altijd eens over de locatie van projecten.
* **Projectsjablonen** bevinden zich in een submap `Project Templates/` van deze map.
* **Wanneer wijzigen**:
  * Stel een netwerkschijf in voor teamsamenwerking
  * Wijzig naar een schijf met meer opslagruimte voor grote datasets
  * Organiseer projecten per jaar, klant of projecttype in verschillende mappen
* **Opmerking**: Het wijzigen van deze instelling heeft alleen invloed op NIEUWE projecten. Bestaande projecten blijven op hun oorspronkelijke locaties staan.***

## Instellingen behouden

Een Chloros-project is een **map**. Alle projectinstellingen worden opgeslagen in `project.json` daarbinnen; aangesloten hardware wordt samen daarmee onthouden in `cameras.json` en `sensors.json`, zodat bij het opnieuw openen van een project ook de camera’s en lichtsensoren weer worden aangesloten. Wanneer je een project opnieuw opent, worden alle instellingen precies zo hersteld als u ze had achtergelaten. Opgeslagen projecten kunnen ook zonder monitor worden bediend met `chloros-cli project` of de `open_project` van de SDK.

### Hiërarchie van instellingen

Instellingen worden in de volgende volgorde toegepast:

1. **Systeemstandaarden** - Ingebouwde standaardinstellingen gedefinieerd door Chloros
2. **Sjablooninstellingen** - Als u een sjabloon laadt bij het aanmaken van een project
3. **Opgeslagen projectinstellingen** - Instellingen die samen met het projectbestand zijn opgeslagen
4. **Handmatige aanpassingen** - Alle wijzigingen die u tijdens de huidige sessie aanbrengt

### Instellingen en beeldverwerking

Verwerkingsinstellingen worden ingelezen wanneer een verwerkingsrun start. Het wijzigen van een instelling heeft geen terugwerkende kracht op producten die al op de schijf staan — voer de verwerking opnieuw uit om de nieuwe instellingen toe te passen. Een aantal instellingen heeft helemaal geen invloed op de verwerking:

* Resolutie miniatuurafbeeldingen (alleen voor weergave)
* Project-sjabloon opslaan
* Projectmap opslaan

***

## Overzicht van configuratiesleutels

Voor automatisering (CLI `--config`, SDK `configure` of het direct lezen van `project.json`), zijn dit de exacte sleutels onder `Project Settings`:

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
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

De `Array alignment`-sleutels worden geschreven zodra het gedeelte ‘Array Alignment’ voor het eerst wordt weergegeven of wanneer ze door een automatiseringsaanroep worden ingesteld. Als ze ontbreken, gebruikt de pijplijn dezelfde waarden als hierboven weergegeven (`true`, `true`, bilineair), dus een project.json zonder deze sleutels gedraagt zich identiek aan een project met deze sleutels.

### Sleutels opgeslagen in `project.json` zonder bedieningselement in het instellingenpaneel

Deze bevinden zich in dezelfde `Project Settings`-boomstructuur en worden door Processing gelezen, maar je zult er geen widget voor vinden in de zijbalk:

| Sleutelpad | Type | Standaard | Ingesteld door |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Overschrijft de manier waarop LATTICE-invoer-TIFF’s worden geïnterpreteerd; `auto` leidt uit de `Chloros:ProcessingLevel` XMP-tag van elk bestand plus het aantal kanalen af te leiden. Wordt genegeerd voor Survey3 `.raw`-opnames. Dit is bewust geen GUI-instelling — ‘auto’ is in alle normale gevallen correct. |
| `Processing → Target reflectance dir` | padstring | `""` | `chloros-cli process --target-reflectance-dir`, of het projectdoel API |
| `Processing → Target reflectance config` | woordenboek op serienummer van de camera | `{}` | Een doel binnen het beeld registreren (modus `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | padstring | `""` | SDK `process_folder(daq_log_path=…)`. Verwijst naar een `.daq`-opname of een map met dergelijke opnames |
| `Target Detection → Minimum calibration target squares` | getal | `4` | Oude standaardinstelling; geen besturing en geen CLI-vlag |
| `UI → Grid thumbnail size` | getal | `160` | De eigen miniatuurzoomschuifregelaar van het afbeeldingsraster |

Twee viewer-voorkeuren worden **op het hoogste niveau in `project.json`** opgeslagen, volledig buiten `Project Settings`, omdat het om weergavestatus gaat in plaats van verwerkingsinstellingen:

| Sleutelpad | Type | Standaard | Ingesteld door |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | geheel getal 1–256 | `1` | De GSD (px) — zie [Een afbeelding op volledig scherm openen](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Aanbevolen werkwijzen

1. **Begin met de standaardinstellingen**: De standaardinstellingen werken goed voor de meeste MAPIR-camerasystemen en gangbare workflows.
2. **Maak sjablonen**: Zodra je de instellingen voor een specifieke workflow of camera hebt geoptimaliseerd, sla je deze op als sjabloon om consistentie tussen projecten te waarborgen.
3. **Test voordat je de volledige verwerking start**: Probeer bij het experimenteren met nieuwe instellingen deze eerst uit op een kleine subset van afbeeldingen voordat je je volledige dataset verwerkt.
4. **Leg je instellingen vast**: Gebruik beschrijvende sjabloonnamen die het camerasysteem, het verwerkingstype en het beoogde gebruik aangeven (bijv. „Survey3\_RGB\_NDVI\_Landbouw“).
5. **Keuze van het exportformaat**: Kies je exportformaat op basis van het einddoel:
   * Wetenschappelijke analyse → TIFF (16-bit of 32-bit)
   * GIS-verwerking → TIFF (16-bits)
   * Snelle visualisatie → PNG (8-bits)
   * Delen op het web → JPG (8-bit)

***

Zie de pagina [Formules voor multispectrale indexen](multispectral-index-formulas.md) voor meer informatie over multispectrale indexen in Chloros.
