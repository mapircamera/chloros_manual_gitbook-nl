# Opname-instellingen en -modi

Het maken van opnames op het tabblad ‘Camera’s’ gebeurt via één rode knop **Alles vastleggen**en één paneel**Opname-instellingen**, waarin wordt bepaald wat die knop teweegbrengt: welke camera’s meedoen, welke exportformaten elke camera opslaat en of de sluiter één keer, continu of met een bepaald interval afgaat. Op deze pagina wordt het hele proces beschreven: de configuratie, de opname zelf, waar de bestanden op de schijf worden opgeslagen en hoe ze later kunnen worden verwerkt tot gekalibreerde producten. De bedieningselementen voor de camera’s en de array zelf bevinden zich op [Camera-instellingen](camera-settings.md).

{% hint style="info" %}
**Voor opnames moet er een project geopend zijn.** &#x27;Alles vastleggen&#x27; en het tandwiel voor opname-instellingen zijn uitgeschakeld totdat er een project is geopend („Maak of open een project om opnames op te slaan“). Elke opname wordt opgeslagen in de projectmap in `captures/`.
{% endhint %}

## Het paneel ‘Opname-instellingen’

Open dit paneel met het **tandwiel naast ‘Alles vastleggen’**in de cameralijst in de zijbalk, of met de knop**‘Opname-instellingen openen…’** onderaan elk instellingenpaneel per camera. De koptekst luidt ‘Opname-instellingen’ en er staat een ←-knop om terug te gaan.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Uw selecties hier — opgenomen camera’s, selectievakjes per type en de opnamemodus — worden **per project** opgeslagen en hersteld wanneer u het project opnieuw opent.

### Opnamemodi

Drie modusknoppen bovenaan het venster:

| Modus | Wat het doet | Subinstellingen (standaard) |
| --- | --- | --- |
| **Enkel** *(standaard)* | Eén opname met alle geselecteerde camera’s. | — |
| **Continu**| Aaneengesloten opnames totdat een stopvoorwaarde wordt bereikt. | Stoppen bij**Aantal opnames** (standaard 1) *of* **Opnameduur** (standaard 10 s; eenheden: seconden / minuten / uren / dagen). |
| **Interval**(time-lapse) | Opnames in bursts op basis van een timer. |**Opnames / interval**(standaard 1) ·**Elke**N eenheden (standaard 5 s) ·**Gedurende** N eenheden (standaard 1 m). |

In de modus Continu of Interval verandert de knop Alles vastleggen tijdens het uitvoeren in een **Stop (N)**-knop, waarbij de opnames worden geteld naarmate ze binnenkomen. Een volle schijf stopt de opname (zie [De ‘Capture All’-workflow](#the-capture-all-flow)).**Continuous op arrays draait binnen de backend (nieuw in 1.3.0).** Wanneer elke camera in de lijst tot een array behoort (standalone camera’s zijn verborgen of uitgesloten), verstuurt de modus ‘Continuous’ niet langer één verzoek per opname: de Chloros

-backend voert de opnamelus zelf uit, schrijft één gesynchroniseerde groep terwijl deze de volgende selecteert, en de knop peilt het aantal opnames. Als er standalone camera’s in de lijst staan, voert Chloros

opnames pass voor pass uit, zoals voorheen.

{% hint style="warning" %}
**Snelle opname op Windows

: zorg ervoor dat de Chloros

-backend wordt uitgesloten van Microsoft Defender.** Het in realtime scannen van elk bestand dat bij een hoge opnamesnelheid wordt geschreven, kostte in de tests van MAPIR

ongeveer 30 % van de opnamesnelheid (honderden bestanden per seconde bij een klein ROI). Het Windows

-installatieprogramma biedt de optie **&quot;De Chloros

-backend uitsluiten van Microsoft Defender-scans (veel snellere cameracapture; aanbevolen)&quot;**, die standaard is aangevinkt, en verwijdert de uitsluiting bij het verwijderen van het programma. Als u dit vinkje hebt verwijderd, voer dan het installatieprogramma opnieuw uit met het vinkje aangevinkt, of voeg een uitsluiting van het Defender-proces toe voor `chloros-backend.exe` (in de map `backend` van de Chloros

-installatiemap). Wanneer Defender één of meer CPU-kernen gebruikt tijdens de continue opname van een array, noteert de Chloros

-backend dit eenmaal in zijn logboek, met dezelfde twee oplossingen.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Camera’s en exporttypes kiezen

De helptekst in het venster vat het samen: kies welke camera’s en exporttypes “Alles vastleggen” oplevert — alles is standaard ingeschakeld en de keuzes worden bij dit project opgeslagen.

* De knoppen **Alles selecteren / Niets selecteren** schakelen in één keer de selectievakjes van alle camera’s in of uit.
* **Schakelaars voor bulkexporttypes**(twee rijen knoppen):**Alle Raw / Alle Debayered / Alle Preview / Alle Radiance / Alle Reflectance / Alle Index**. Elk heeft een driestandenkleur: groen ✓ = ingeschakeld voor elke camera die dit ondersteunt, oranje – = ingeschakeld voor sommige, grijs = geen. Een schakelaar is uitgeschakeld wanneer geen enkele aangesloten camera dat type ondersteunt. Ze worden allemaal grijs weergegeven wanneer ‘Fastest Capture’ is ingeschakeld.
* **Rijen per camera**: een selectievakje, plus een uitklapbare (▸/▾) lijst met de toepasselijke exporttypes voor die camera, met afzonderlijke selectievakjes. De rij toont een aantal ingeschakelde opties, bijvoorbeeld &quot;4/6&quot;.

### Exporttypes en welke camera’s deze ondersteunen

Er zijn zes exporttypes: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Alleen de toepasselijke types verschijnen in de rij van elke camera:

| Exporttype | Inhoud | RGB

(FRGB) | Bayer multispectraal (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mozaïek (mono: de enkele band) rechtstreeks van de sensor | ✓ | ✓ | ✓ |
| **Debayered** | Lineaire demosaïek: de donkere offset (DSNU) uit het kalibratiepakket verwijderd uit de raw, gevolgd door een randgevoelige demosaïek — geen lineariteitscorrectie, flat field of unmix. Opgeslagen als een 3-kanaals RGB

-TIFF

-bestand (mono: 1-kanaals grijswaarden) | ✓ | ✓ | ✓ |
| **Voorbeeld** | Volledige weergaveketen (witbalans + gamma volgens het profiel van de camera, plus de instelling ‘Highlights’ van de RGB

-camera; multispectraal: valsekleurenuitrekking) | ✓ | ✓ | ✓ |
| **Stralingsintensiteit** | float32 W/m²/sr/nm via de volledige radiometrische keten | — (niet beschikbaar) | ✓ | ✓ |
| **Reflectie** | uint16 ρ (32768 = 1,0) | — (niet beschikbaar) | ✓ — wordt alleen weergegeven als de camera een DAQ-lichtsensor heeft (eigen of overgenomen van de array) | hetzelfde als multispectraal |
| **Index** | Weergave vegetatie-index (LUT) | — | ✓ — vereist een ingeschakelde, niet-lege indexuitdrukking op de camera, en wordt niet aangeboden aan leden van een gecombineerde array (de array bezit één gedeelde index) | — (een index vereist ≥2 banden; zie [Mono-camera’s &amp; vegetatie-indexen](mono-indices.md)) |

Stralingsintensiteit en reflectie worden nooit aangeboden voor RGB

-camera’s — stralingsintensiteit per Bayer-pixel is niet zinvol voor een breedbandige fotometrische sensor.

### Snelste opname

De schakelaar **⚡ Snelste opname — alleen raw**(oranje wanneer ingeschakeld) overschrijft alle exportselecties naar**alleen raw** — plus een gratis samengestelde afbeelding met gecombineerde index voor arrays — zodat het frame zo snel mogelijk wordt opgeslagen: de berekeningen voor straling/reflectie/weergave worden volledig overgeslagen op het moment van opname.

{% hint style="info" %}
**Er wordt nog steeds een `.daq` opgeslagen.** Wanneer een lichtsensor is toegewezen, schrijft ‘Snelste opname’ nog steeds de DAQ-waarde voor neerwaartse straling op (in de map `daq/` van de opnames) — zodat stralings-, reflectie- en indexproducten later allemaal kunnen worden gegenereerd door herverwerking (zie [Opnames opnieuw verwerken](#opnames-opnieuw-verwerken-tot-gekalibreerde-producten)). Fastest Capture heeft ook geen invloed op je selecties via de selectievakjes: schakel de functie uit en ze komen weer terug.
{% endhint %}

### Instellingen per array

Elke aangesloten array krijgt een eigen groepskaart in het venster:

* **Selectievakje ‘Opnemen’** (drie standen voor alle leden) en de naam van de array met de weergavemodus: „(gecombineerd | afzonderlijk)“.
* Selectievakje **Uitgelijnd**(standaard**aan**): past de export van leden aan het uitlijningsprofiel van de array aan, zodat de exporten pixel-geregistreerd zijn tussen camera’s. Raw-bestanden blijven onvervormd, maar bevatten de transformatie in hun metagegevens. (Het profiel zelf wordt berekend in het [paneel met array-instellingen](camera-settings.md#alignment-co-registration).) Bestanden van array-leden worden opgeslagen in de oorspronkelijke oriëntatie van elke camera, waarbij eventuele instellingen voor spiegelen of roteren in het bestand worden vastgelegd in plaats van toegepast.
* Rijen met cameraleden zijn genest binnen de kaart.

De arraykaart bevat ook twee recorders. Beschouw ze als **monitoring versus analyse**:

| Recorder | Niveau | Wat het opneemt |
| --- | --- | --- |
| **● Indexvideo opnemen / ■ Opname stoppen** *(alleen gecombineerde arrays)* | **Monitoring** | De live samengestelde gecombineerde index als video met 10 fps — 8-bit, previewresolutie, LUT ingebakken. Vereist een geopend project en een live streamingweergave. Toont frames + verstreken tijd tijdens het opnemen. |
| **⦿ Raw-burst opnemen / ■ Raw-burst stoppen** *(elke array)* | **Analyse**| Raw Bayer-frames met de live-opnamesnelheid (zonder verwerking) plus een manifest per frame en `.daq`-metingen, opgeslagen in `captures/bursts/`. Na een burst verschijnt een**Video samenstellen**-knop: deze verwerkt de burst offline opnieuw tot gekalibreerde video — gecombineerde index en/of per camera straling / reflectie / index — plus optionele TIFF&#x27;s. Het samenstellen van de gecombineerde index start automatisch wanneer je de burst stopt. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## De ‘Capture All’-workflow

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->



Druk op **Capture All** in de cameralijst in de zijbalk:

1. Elke opgenomen, zichtbare, niet-gepauzeerde camera maakt opnames met de geselecteerde exportformaten. **Arrays worden als één gesynchroniseerde trigger geactiveerd** (één gesynchroniseerde groep voor alle leden — zie [Multi-Camera Arrays](arrays.md)); losstaande camera’s maken afzonderlijk opnames.
2. Verborgen (oog) of gepauzeerde camera’s worden overgeslagen. Een array wordt pas volledig geblokkeerd wanneer *alle* leden ervan verborgen of gepauzeerd zijn.
3. Wanneer er een lichtsensor is toegewezen, wordt de bijbehorende DAQ-meting van de neerwaartse straling opgeslagen als een `.daq`-bestand in de `daq/`-map van de opnames — zelfs bij opnames die uitsluitend uit ruwe gegevens bestaan — zodat er later altijd radiometrische producten kunnen worden afgeleid.
4. De knop toont de voortgang in realtime — „Capturing… done/total“ — en wordt in de modus Continu/Interval **Stop (N)**. Elk opname-item heeft een time-out van 300 s.
5. Wanneer de run is voltooid, meldt een resultaatvenster **&quot;N bestanden opgeslagen&quot;**of**&quot;N opgeslagen, F mislukt&quot;**, plus &quot;(S verborgen/gepauzeerd/overgeslagen)&quot; wanneer camera&#x27;s zijn overgeslagen. Als de schijf vol raakt, stopt de run en wordt**&quot;Gestopt: schijf vol. N opgeslagen, F mislukt&quot;** gemeld — elk bestand dat niet kon worden geschreven, wordt als mislukt geteld.

## Waar opnames worden opgeslagen

Opnames worden opgeslagen onder het geopende project in `<project>/captures/`. Elk exporttype wordt in een **eigen submap** opgeslagen, zodat bij een opname met meerdere niveaus de typen nooit door elkaar komen:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` is de tijdstempel van de opname en `<serial>` het serienummer van de camera. Op zichzelf staande opnames krijgen de naam `capture_<ts>_SN<serial>_<level>`; reeksopnames van één gesynchroniseerde trigger krijgen de naam `sync_<ts>_SN<serial>_<level>` en **delen één tijdstempel voor alle camera’s in de groep** (het niveau-achtervoegsel wordt weggelaten wanneer een camera slechts één niveau opslaat).
* **Een afwijking om te weten:** het weergaveniveau wordt opgeslagen in een map met de naam `preview/`, terwijl de bestanden `_display` in de naam behouden — de map en het achtervoegsel verschillen alleen voor dat niveau.
* Onbekende niveaus worden standaard in een map met hun eigen naam opgeslagen; als er geen submap kan worden aangemaakt, wordt het bestand naar de hoofdmap van de opnames geschreven in plaats van verloren te gaan.
* Opname-TIFF’s worden standaard verliesvrij gecomprimeerd (DEFLATE) en bevatten hun volledige kalibratie- en verwerkingsmetadata **in de XMP van het bestand** — opnames zijn zelfbeschrijvend, zonder andere sidecar-bestanden dan het `.daq`-bestand (en het bijbehorende `.daq.cameras.json`).
* **Clamp-maskers (nieuw in 1.3.0).**Wanneer de 3×3 spectrale ontmenging een band op ten minste één pixel op 0 moest afklemmen, krijgen de radiance-, reflectance- en indexproducten een masker per pixel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = de band in het R-kanaal van het product, bit 1 = G, bit 2 = B); de XMP-tag `Chloros:UnmixClampMask` van het product verwijst hiernaar. De maskers staan**naast** de productmappen, nooit erin, zodat een productmap zoals `reflectance/` ongewijzigd aan fotogrammetriesoftware kan worden doorgegeven. De eigen importfunctie van Chloros

slaat de maskers over.
* **Oriëntatie (1.3.0).** De Preview-, Radiance-, Reflectance- en Index-producten van een standalone camera worden gedraaid door de instelling voor Spiegeling/Rotatie; Raw- en Debayered-bestanden blijven zoals de sensor ze heeft uitgelezen. Alle bestanden in de array blijven in hun oorspronkelijke formaat. Elk bestand slaat de instelling op in zijn XMP, zodat deze bij herverwerking op dezelfde manier wordt toegepast.
* **Van welke camera een `.daq` afkomstig is (1.3.0).** De `.daq` van een opname wordt tijdens de verwerking uitsluitend gekoppeld aan beelden van de camera’s die in de `.daq.cameras.json` worden genoemd. Een stand-alone camera zonder eigen lichtsensor gebruikt daarom niet langer de meetwaarde van een andere camera — koppel er een sensor aan, of registreer er een in het tabblad Lichtsensoren. Een registratie van een lichtsensor (en elke `.daq` zonder dat bestand, inclusief oudere opnames) is nog steeds bruikbaar voor elke camera.

Dit is dezelfde indeling die `chloros-cli lattice capture` / `array-capture` naar hun `-o`-map schrijven — gedocumenteerd in de [CLI

-referentie § Hoe een opnamemap eruitziet](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Opnames opnieuw verwerken tot gekalibreerde producten

De vastgelegde onbewerkte frames plus het opgeslagen `.daq` zijn alles wat de verwerkingspijplijn nodig heeft — daarom is Fastest Capture veilig voor echt werk.

* **GUI**: voeg de map met opnames toe aan een project ([Bestanden toevoegen aan een project](../processing-images-gui/adding-files-to-a-project.md)) en verwerk deze zoals gewoonlijk.
* **CLI**: wijs `process` naar de **hoofdmap met opnames**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importeert normaal gesproken alleen de map die je opgeeft, maar als die map geen afbeeldingen bevat en wel submappen heeft, gaat het automatisch dieper in de structuur — zodat de submappen op dat niveau en de map `daq/` in één keer worden opgepikt. Elke opname wordt geïmporteerd als **één afbeelding**, waarbij de andere niveaus als weergavemodi worden toegevoegd, en niet als één afbeelding per niveau.

Het direct benoemen van een submap van een niveau (bijv. `…/captures/raw/`) werkt ook, maar laat de map `daq/` achter — kopieer deze ernaast wanneer je-een radiometrisch product afleiden uit `raw/`, anders kan de tijdstempelvergelijking niets vinden om mee te vergelijken.

{% hint style="warning" %}
**De verwerking begint altijd bij `raw`.** Binnen elke opname is het ruwe beeld de bron van de pijplijn; `debayered`, `radiance`, `reflectance`, en `preview` worden als weergavemodi meegeleverd, maar worden nooit teruggekoppeld via de pijplijn — het opnieuw verwerken van een afgeleid product zou de vignettering, kleur en stralingsberekeningen die al in de pixels zijn ingebakken opnieuw toepassen, dus Chloros

-weigert dit in plaats van dubbele verwerking. De weergaven `index/` en `composite/` worden helemaal niet verwerkt (het zijn uitvoerbestanden, geen opnames). Een map met opnames die **zonder** raw-importen is opgeslagen, wordt normaal weergegeven, maar `process` slaat deze over en geeft dit aan; `--input-level {raw,debayered,processed}` is de bewuste nooduitgang die een startpunt afdwingt. Zie de [CLI

-referentie](../reference/cli-reference.md#what-a-captures-folder-looks-like) voor de exacte oversla-meldingen.
{% endhint %}

Nog twee gedragingen die de moeite waard zijn om te weten bij het scripten van herverwerking:

* Een `chloros-cli process`-run die producten heeft aangevraagd maar **geen beeldproducten**heeft geschreven, mislukt met een duidelijke foutmelding en stopt met een niet-nulwaarde** — je krijgt nooit een stille, lege run. Succesvolle runs rapporteren het aantal producten. (Een opzettelijke run met alleen metadata telt nog steeds als succes.)
* Opnieuw geïmporteerde, verwerkte exporten nemen nooit de ‘raw’-slot van een opname in beslag — de originele ‘raw’ blijft altijd de bron van de pijplijn.

## CLI

-equivalenten

Alles op deze pagina kan zonder grafische interface worden aangestuurd. De opnamemodi van de GUI komen rechtstreeks overeen met `chloros-cli lattice array-capture`:

| GUI | CLI

|
| --- | --- |
| Enkelvoudig | `chloros-cli lattice array-capture` |
| Continu | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Snelste opname | `array-capture --fastest` |
| Uitgelijnd selectievakje | `--aligned / --no-aligned` |
| Selectievakjes voor exporttype | `--processing LEVEL` of `--levels L1,L2,…` (standaard `all`) |
| Video-index opnemen | `chloros-cli lattice array-record` |
| Ruwe burst opnemen / Video samenstellen | `chloros-cli lattice array-burst` / `array-build-video` |

Volledige vlagentabellen, de optie voor vastgelegde opname met slimme AE (`--smart`) en het model met constante snelheid worden beschreven in de [CLI

-referentie § Opnamemodi, recorders en offline herverwerking](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
