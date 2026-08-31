# Opname-instellingen en -modi

Het maken van opnames op het tabblad ‘Camera’s’ verloopt via één rode knop **Alles vastleggen**en één paneel**Opname-instellingen**, waarin wordt bepaald wat die knop teweegbrengt: welke camera’s meedoen, welke exportformaten elke camera opslaat en of de sluiter één keer, continu of met een bepaald interval afgaat. Op deze pagina wordt het gehele proces beschreven: de configuratie, het opnamen zelf, waar de bestanden op de schijf worden opgeslagen en hoe ze later kunnen worden verwerkt tot gekalibreerde producten. De bedieningselementen voor de camera’s en de opstelling zelf zijn te vinden onder [Camera-instellingen](camera-settings.md).

{% hint style="info" %}
**Voor opnames moet er een project geopend zijn.** De knop Alles vastleggen en het tandwielpictogram voor Opname-instellingen zijn uitgeschakeld totdat er een project is geopend („Maak of open een project om opnames op te slaan”). Elke opname wordt opgeslagen in de projectmap in `captures/`.
{% endhint %}

## Het venster ‘Opname-instellingen’

Open dit venster met het **tandwiel naast ‘Alles vastleggen’**in de cameralijst in de zijbalk, of met de knop**‘Opname-instellingen openen…’** onderaan elk instellingenvenster per camera. De koptekst luidt „Opname-instellingen“ met een ←-terugknop.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Je selecties hier — opgenomen camera’s, selectievakjes per type en de opnamemodus — worden **per project** opgeslagen en hersteld wanneer je het project opnieuw opent.

### Opnamemodi

Drie modusknoppen bovenaan het venster:

| Modus | Wat het doet | Subinstellingen (standaardinstellingen) |
| --- | --- | --- |
| **Enkel** *(standaard)* | Eén opname met alle geselecteerde camera’s. | — |
| **Continu**| Opeenvolgende opnames totdat een stopvoorwaarde wordt bereikt. | Stoppen bij**Aantal opnames** (standaard 1) *of* **Opnameduur** (standaard 10 s; eenheden: seconden / minuten / uren / dagen). |
| **Interval**(timelapse) | Opnamereeksen volgens een timer. |**Opnames / interval**(standaard 1) ·**Elke**N eenheden (standaard 5 s) ·**Gedurende** N eenheden (standaard 1 m). |

In de modus Continu of Interval verandert de knop „Alles vastleggen“ tijdens het uitvoeren in een **Stop (N)**-knop, waarbij de opnames worden geteld naarmate ze binnenkomen.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Camera’s en exporttypes kiezen

De helptekst van het venster vat het samen: kies welke camera&#x27;s en exporttypes „Alles vastleggen“ oplevert — alles staat standaard aan en de keuzes worden bij dit project opgeslagen.

* De knoppen **Alles selecteren / Niets selecteren** schakelen in één keer de selectievakjes van elke camera in of uit.
* **Schakelaars voor bulkexporttypes**(twee rijen knoppen):**All Raw / All Debayered / All Preview / All Radiance / All Reflectance / All Index**. Elke schakelaar heeft drie statussen: groen ✓ = ingeschakeld voor elke camera die dit ondersteunt, oranje – = ingeschakeld voor sommige, grijs = geen. Een schakelaar is uitgeschakeld wanneer geen enkele aangesloten camera dat type ondersteunt. Ze worden allemaal grijs weergegeven wanneer „Fastest Capture“ is ingeschakeld.
* **Rijen per camera**: een selectievakje om op te nemen, plus een uitklapbare (▸/▾) lijst met de voor die camera beschikbare exporttypes met afzonderlijke selectievakjes. De rij toont een teller zoals &quot;4/6&quot;.

### Exporttypes en welke camera’s deze ondersteunen

Er zijn zes exporttypes: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Alleen de toepasselijke types verschijnen in de rij van elke camera:

| Exporttype | Inhoud | RGB (FRGB) | Bayer multispectraal (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mozaïek (mono: de enkele band) rechtstreeks van de sensor | ✓ | ✓ | ✓ |
| **Debayered** | Lineaire demosaïek (mono: 1-kanaals grijswaarden) | ✓ | ✓ | ✓ |
| **Voorbeeld** | Volledige weergaveketen (witbalans + gamma volgens het profiel van de camera; multispectraal: valsekleurversterking) | ✓ | ✓ | ✓ |
| **Stralingsintensiteit** | float32 W/m²/sr/nm via de volledige radiometrische keten | — (niet beschikbaar) | ✓ | ✓ |
| **Reflectantie** | uint16 ρ (32768 = 1,0) | — (niet beschikbaar) | ✓ — wordt alleen weergegeven als de camera een DAQ-lichtsensor heeft (eigen of overgenomen van de array) | hetzelfde als multispectraal |
| **Index** | Weergave vegetatie-index (LUT) | — | ✓ — vereist een ingeschakelde, niet-lege indexuitdrukking op de camera, en wordt niet aangeboden aan leden van een gecombineerde array (de array beschikt over één gedeelde index) | — (een index vereist ≥2 banden; zie [Mono-camera’s &amp; vegetatie-indexen](mono-indices.md)) |

Stralingsintensiteit en reflectie worden nooit aangeboden voor RGB-camera’s — stralingsintensiteit per Bayer-pixel is niet zinvol voor een breedbandige fotometrische sensor.

### Snelste opname

De schakelaar **⚡ Snelste opname — alleen raw**(oranje wanneer ingeschakeld) overschrijft alle exportselecties naar**alleen raw** — plus een gratis samengesteld beeld met gecombineerde index voor arrays — zodat het beeld zo snel mogelijk wordt opgeslagen: de berekeningen voor stralingsintensiteit/reflectie/weergave worden volledig overgeslagen op het moment van opname.

{% hint style="info" %}
**Er wordt nog steeds een `.daq` opgeslagen.** Wanneer een lichtsensor is toegewezen, schrijft „Fastest Capture“ nog steeds de DAQ-waarde voor neerwaartse straling naast de RAW-frames — zodat stralingsintensiteit, reflectie en indexproducten later allemaal kunnen worden gegenereerd door herverwerking (zie [Vastleggingen herverwerken](#re-processing-captures-into-calibrated-products)). Fastest Capture heeft ook geen invloed op de selecties in de selectievakjes: schakel het uit en ze komen weer terug.
{% endhint %}

### Instellingen per array

Elke aangesloten array krijgt zijn eigen groepkaart in het venster:

* **Selectievakje ‘Include’** (drie standen voor alle leden) en de naam van de array met de weergavemodus: „(combined | separate)“.
* Selectievakje **Uitgelijnd**(standaard**aan**): past de export van leden aan het uitlijningsprofiel van de array aan, zodat de exporten pixel-geregistreerd zijn tussen camera’s. Raw-bestanden blijven onvervormd, maar bevatten de transformatie in hun metadata. (Het profiel zelf wordt berekend in het [venster met array-instellingen](camera-settings.md#alignment-co-registration-combined-only).)
* De rijen met cameraleden zijn genest binnen de kaart.

De arraykaart bevat ook twee recorders. Zie ze als **monitoring versus analyse**:

| Recorder | Niveau | Wat het opneemt |
| --- | --- | --- |
| **● Indexvideo opnemen / ■ Opname stoppen** *(alleen bij gecombineerde arrays)* | **Monitoring** | De live samengestelde gecombineerde index als video met 10 fps — 8-bit, previewresolutie, LUT ingebakken. Vereist een geopend project en een live streamingweergave. Toont frames + verstreken tijd tijdens het opnemen. |
| **⦿ Raw-burst opnemen / ■ Raw-burst stoppen** *(elke array)* | **Analyse**| Raw Bayer-frames met de live-opnamesnelheid (zonder verwerking) plus een manifest per frame en `.daq`-metingen, opgeslagen in `captures/bursts/`. Na een burst verschijnt een**Video samenstellen**-knop: deze verwerkt de burst offline opnieuw tot gekalibreerde video — gecombineerde index en/of per camera straling / reflectie / index — plus optionele TIFFs. Het samenstellen van de gecombineerde index start automatisch wanneer je de burst stopt. |##

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

De ‘Capture All’-workflow

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Druk op **Capture All** in de cameralijst in de zijbalk:

1. Elke opgenomen, zichtbare, niet-gepauzeerde camera neemt op met de geselecteerde exportformaten. **Arrays worden als één gesynchroniseerde trigger geactiveerd** (één enkele gesynchroniseerde groep voor alle leden — zie [Multi-Camera Arrays](arrays.md)); losstaande camera’s nemen afzonderlijk op.
2. Verborgen (oog) of gepauzeerde camera’s worden overgeslagen. Een array wordt pas volledig geblokkeerd wanneer *alle* leden ervan verborgen of gepauzeerd zijn.
3. Wanneer er een lichtsensor is toegewezen, wordt de bijbehorende DAQ-waarde voor neerwaartse straling samen met de beelden opgeslagen als een `.daq`-bestand — zelfs bij opnames die uitsluitend uit ruwe gegevens bestaan — zodat er later altijd radiometrische producten kunnen worden afgeleid.
4. De knop toont de voortgang in realtime — „Beeld vastleggen… voltooid/totaal“ — en wordt in de modus Continu/Interval **Stop (N)**. Elk vastleggingsitem heeft een time-out van 300 s.
5. Wanneer de passage is voltooid, verschijnt er een resultaatmelding met **&quot;N bestanden opgeslagen&quot;**of**&quot;N opgeslagen, F mislukt&quot;**, plus &quot;(S verborgen/gepauzeerd/overgeslagen)&quot; wanneer camera’s zijn overgeslagen.

## Waar opnames worden opgeslagen

Opnames worden opgeslagen onder het geopende project in `<project>/captures/`. Elk exporttype komt in een **eigen submap** terecht, zodat bij een opname met meerdere niveaus de typen nooit door elkaar raken:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` is de tijdstempel van de opname en `<serial>` het serienummer van de camera. Op zichzelf staande opnames krijgen de naam `capture_<ts>_SN<serial>_<level>`; reeksopnames van één gesynchroniseerde trigger krijgen de naam `sync_<ts>_SN<serial>_<level>` en **delen één tijdstempel voor alle camera’s in de groep** (het niveaueindtoevoegsel wordt weggelaten wanneer een camera slechts één niveau opslaat).
* **Een afwijking om rekening mee te houden:** het weergaveniveau wordt opgeslagen in een map met de naam `preview/`, terwijl de bestanden `_display` in de naam behouden — de map en het achtervoegsel verschillen alleen voor dat niveau.
* Onbekende niveaus worden standaard opgeslagen in een map met hun eigen naam; als er geen submap kan worden aangemaakt, wordt het bestand naar de hoofdmap van de opnames geschreven in plaats van verloren te gaan.
* Capture-TIFF’s worden standaard verliesvrij gecomprimeerd (DEFLATE) en bevatten hun volledige kalibratie- en verwerkingsmetadata **in de XMP van het bestand** — opnames zijn zelfbeschrijvend en hebben geen andere sidecar-bestanden dan het bestand met de naam `.daq`.

Dit is dezelfde indeling die `chloros-cli lattice capture` / `array-capture` naar hun `-o`-map schrijven — gedocumenteerd in de [CLI-referentie § Hoe een opnamemap eruitziet](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Opnames opnieuw verwerken tot gekalibreerde producten

De vastgelegde ruwe frames plus het opgeslagen `.daq` zijn alles wat de verwerkingspijplijn nodig heeft — daarom is ‘Fastest Capture’ veilig voor echt werk.

* **GUI**: voeg de map met opnames toe aan een project ([Bestanden toevoegen aan een project](../processing-images-gui/adding-files-to-a-project.md)) en verwerk deze zoals gewoonlijk.
* **CLI**: wijs `process` naar de**hoofdmap van de opnames**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importeert normaal gesproken alleen de map die je opgeeft, maar wanneer die map geen afbeeldingen bevat en wel submappen heeft, wordt deze automatisch doorlopen — zodat de submappen op niveau en de **`.daq`**-bestanden in de hoofdmap in één keer worden opgepikt. Elke opname wordt geïmporteerd als een**enkele afbeelding**, waarbij de andere niveaus als weergavemodi zijn gekoppeld, en niet als één afbeelding per niveau.

Het rechtstreeks benoemen van een submap op een bepaald niveau (bijv. `…/captures/raw/`) werkt ook, maar dan blijven de hoofdmappen `.daq` achter — kopieer deze erbij wanneer je een radiometrisch product opnieuw afleidt uit `raw/`, anders kan de tijdstempelvergelijking niets vinden om mee te vergelijken.

{% hint style="warning" %}
**De verwerking begint altijd bij `raw`.**Binnen elke opname is het ruwe beeld de bron voor de verwerkingspijplijn; `debayered`, `radiance`, `reflectance` en `preview` worden als weergavemodi gegenereerd, maar worden nooit teruggevoerd door de pijplijn — het opnieuw verwerken van een afgeleid product zou de vignettering, kleur en stralingsberekeningen die al in de pixels zijn ingebakken opnieuw toepassen, dus Chloros wordt afgewezen in plaats van dubbel verwerkt. De weergaven `index/` en `composite/` worden helemaal niet verwerkt (het zijn uitvoerbestanden, geen opnames). Een map met opnames die**zonder** raw-importen is opgeslagen, wordt normaal weergegeven, maar `process` slaat deze over en geeft dit aan; `--input-level {raw,debayered,processed}` is de bewuste nooduitgang die een startpunt afdwingt. Zie de [CLI-referentie](../reference/cli-reference.md#what-a-captures-folder-looks-like) voor de exacte oversla-meldingen.
{% endhint %}

Nog twee gedragingen die de moeite waard zijn om te weten bij het scripten van herverwerking:

* Een `chloros-cli process`-run die producten heeft aangevraagd maar **geen beeldproducten**heeft geschreven, mislukt met een duidelijke foutmelding en stopt met een niet-nulwaarde** — je krijgt nooit een stille, lege run. Succesvolle runs rapporteren hun productaantallen. (Een opzettelijke run met alleen metadata telt nog steeds als succes.)
* Opnieuw geïmporteerde, verwerkte exporten nemen nooit de ‘raw’-slot van een opname in beslag — de originele ‘raw’ blijft altijd de bron van de pijplijn.

## CLI-equivalenten

Alles op deze pagina kan zonder grafische interface worden uitgevoerd. De opnamemodi van de GUI komen rechtstreeks overeen met `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enkel | `chloros-cli lattice array-capture` |
| Continu | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Snelste opname | `array-capture --fastest` |
| Uitgelijnd selectievakje | `--aligned / --no-aligned` |
| Selectievakjes voor exporttype | `--processing LEVEL` of `--levels L1,L2,…` (standaard `all`) |
| Video-index opnemen | `chloros-cli lattice array-record` |
| Raw-burst opnemen / Video samenstellen | `chloros-cli lattice array-burst` / `array-build-video` |

Volledige vlagentabellen, de optie voor vastgelegde opname met smart-AE (`--smart`) en het model met constante snelheid zijn te vinden in de [CLI Referentie § Opnamemodi, recorders en offline herverwerking](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
