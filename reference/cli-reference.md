# Chloros Referentie voor CLI

**Versie:**

1.2.0**Gemaakt:**29 juli 2026 19:19 ·**Herzien:** 30 augustus 2026**Doelgroep:** Geoptimaliseerd voor gebruik door grote taalmodellen (LLM); leesbaar voor mensen.**Toepassingsgebied:** Alle voor gebruikers bestemde subcommando’s van `chloros-cli`, inclusief opties en voorbeelden die kunnen worden gekopieerd en geplakt.

Dit document is de volledige handleiding voor het `chloros-cli`-opdrachtregelprogramma dat wordt meegeleverd met MAPIR Chloros. Het is bewust zo uitgebreid opgesteld dat een LLM (of een mens) elke ondersteunde workflow kan samenstellen op basis van de onderstaande lijsten, zonder de broncode te hoeven bekijken.

Als je alleen de belangrijkste punten nodig hebt, ga dan naar:
- [Snelstart in vijf minuten](#five-minute-quickstart)
- [LATTICE Camera First-Connect-workflow](#lattice-camera-first-connect-workflow)
- [DAQ-sensor-eerste-verbinding-workflow](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Opnamemodi, recorders en offline herverwerking](#capture-modes-recorders--offline-reprocess)

---

## Conventies

- Alle commando’s beginnen met het voorvoegsel `chloros-cli`. Op Windows is het binaire bestand `chloros-cli.exe`; op Linux /Jetson is het `chloros-cli`.
- Optionele argumenten worden weergegeven als `--flag`. Verplichte positionele argumenten worden zonder haakjes weergegeven.
- Wanneer een standaardwaarde is opgegeven, wordt die waarde gebruikt als de vlag wordt weggelaten.
- De CLI is een thin-HTTP-client bovenop de Chloros-backend (Flask-server op `127.0.0.1:5000`). De backend wordt door de meeste commando’s automatisch gestart. `CHLOROS_BACKEND_URL=<url>` verwijst naar **`lattice`**,**`project`**, en**`daq pool-*`**-opdrachtfamilies naar een backend op afstand — de kernopdrachten (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixeren opzettelijk `http://127.0.0.1:<port>` en negeren (de IPv4-literal voorkomt Windows&#x27; `localhost`→`::1` ~2 s per verzoek). Zie [Omgevingsvariabelen](#environment-variables).
- Een Chloros+ account is vereist voor alle SDK / CLI-aanroepen (voer `chloros-cli login` één keer per machine uit; in de cache opgeslagen in `~/.chloros/`).
- Voorbeelden gebruiken Linux-paden; vervang op Windows `/home/user/...` door `C:/Users/.../...`.

---

## Overzicht op het hoogste niveau

```
chloros-cli [global options] COMMAND [command options]
```

### Algemene opties

| Vlag | Beschrijving |
| --- | --- |
| `--backend-exe PATH` | Het automatisch gedetecteerde backend-uitvoerbare bestand overschrijven. |
| `--port N` | Backend-HTTP-poort (standaard: `5000`). |
| `-v, --verbose` | Uitgebreide uitvoer inschakelen. |
| `--restart` | De backend geforceerd opnieuw opstarten (beëindigt alle actieve `backend_server.py`-processen). |
| `--version` | Versie weergeven (`Chloros CLI 1.2.0`). |
| `--help` | Help op het hoogste niveau weergeven. |

### Overzicht van opdrachten

| Opdracht | Doel |
| --- | --- |
| [`process`](#chloros-cli-process) | Verwerk een map met end-to-end-opnames van Survey3 of LATTICE. |
| [`login`](#chloros-cli-login) | Deze machine authenticeren met een Chloros+-account. |
| [`logout`](#chloros-cli-logout) | Wist de in de cache opgeslagen inloggegevens. |
| [`status`](#chloros-cli-status) | Toon de huidige licentie-/authenticatiestatus. |
| [`export-status`](#chloros-cli-export-status) | Live voortgang van Thread-4-export tijdens een `process`-run. |
| [`language`](#chloros-cli-language) | De weergavetaal vCLI instellen of weergeven (38 talen ondersteund). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standaard projectmap (gedeeld met de GUI). |
| [`update`](#chloros-cli-update) | Controleren op en installeren van CLI-updates (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systeemdiagnostiek + rooktests. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-status / -besturing. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-camerabesturing en -opname (meer dan 45 subcommando’s). |
| [`daq`](#chloros-cli-daq) | DAQ-spectraalsensorbesturing (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Een opgeslagen Chloros-project (camera’s + DAQ’s) openen en uitvoeren. |

---

## Installatie

`chloros-cli` wordt meegeleverd in het Chloros desktop-installatieprogramma op elk ondersteund platform — er is geen aparte CLI download beschikbaar. Door het platformpakket te installeren, wordt `chloros-cli` toegevoegd aan je `PATH`, naast de desktop-app en het backend-binaire bestand die het aanstuurt.

Nieuwste downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Het installatieprogramma bevat ook handige startprogramma&#x27;s (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) die een kant-en-klare CLI-shell openen; deze worden behandeld in de [CLI-gebruikershandleiding](../CLI.md) en worden hier niet herhaald.

### Windows (.exe)

1. Download het installatieprogramma Windows vanaf de downloadpagina.
2. Voer `Chloros-Setup-x.y.z.exe` uit en volg de wizard. Het standaardinstallatiepad is `C:\Program Files\Chloros\` (de map CLI komt in `C:\Program Files\Chloros\cli\`, dat het installatieprogramma toevoegt aan PATH).
3. Open een nieuwe terminal (`cmd.exe`, PowerShell of Windows Terminal) zodat de bijgewerkte `PATH` wordt herkend.

```powershell
chloros-cli --version
```

Het installatieprogramma voegt `chloros-cli.exe` automatisch toe aan je systeem `PATH` en bundelt de Arena-SDK-runtime die nodig is voor LATTICE-camera’s.

### Linux amd64 (.deb)

Voor Ubuntu 22.04 LTS of nieuwer / op Debian gebaseerde x86_64-werkstations.

> **Ubuntu 20.04 wordt niet ondersteund.** De lijst met afhankelijkheden van het pakket is afgeleid van
> waar de backend daadwerkelijk naar linkt, en daaronder valt ook `libc6 (>= 2.34)`;
> Focal levert glibc 2.31 mee. `apt` weigert de installatie in plaats van deze te laten mislukken tijdens
> de uitvoering.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Het .deb-pakket installeert:
- `chloros-cli` tot `/usr/bin/chloros-cli`
- De gecompileerde backend naar `/usr/lib/chloros/chloros-backend`
- De Arena-SDK-runtime (voor LATTICE-camera’s)
- Denoiser-modellen, kalibratiebundels en configuratie voor het updatekanaal

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Dezelfde indeling als het amd64 .deb-bestand, met een CUDA-build die is afgestemd op Jetson Orin / Orin NX / Orin Nano.

### Eén keer authenticeren per machine

Elk platform vereist een eenmalige aanmelding via Chloros+ voordat de SDK / CLI-aanroepen werken:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Inloggegevens worden opgeslagen in `~/.chloros/user_session.json`.

### Controleer de installatie

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Een Chloros+ abonnement is vereist.**Voor de CLI is een actief Chloros+ abonnement nodig.**Copper**is het instapChloros+ niveau — elk betaald Chloros+ niveau biedt toegang tot CLI / SDK; alleen het gratis**Iron**-niveau niet. (Plan-id-overzicht: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Upgrade via [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Deze limiet wordt afgedwongen door de backend, niet alleen door de CLI: een verzoek met de vlaggen SDK / CLI zonder betaald abonnement wordt afgewezen met `403 PLAN_UPGRADE_REQUIRED`, of het nu afkomstig is van `chloros-cli`, de Python SDK, of een handmatig-gerollde HTTP-client. Een uitgelogde aanroeper krijgt in plaats daarvan de foutmelding `401 AUTH_REQUIRED`. Toegang werkt offline gedurende deresp. de respijtperiode (maandelijks 30 dagen, bij jaarabonnementen tot de vervaldatum) en stopt wanneer die verstrijkt; `chloros-cli status` blijft werken zodat de reden zichtbaar is (het is de route SDK / CLI die is vrijgesteld van de tier-gate — `GET /api/license-status`).

---

## Snelstart in vijf minuten

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Verwerk een map met afbeeldingen via de volledige &#x27;Chloros&#x27;-pijplijn (doeldetectie → kalibratie → vignet → reflectantie → indexexport).

### Overzicht

```
chloros-cli process INPUT [OPTIONS]
```

### Positieargumenten

| Argument | Beschrijving |
| --- | --- |
| `INPUT` | Pad naar de invoermap met `.raw + .jpg`- (Survey3), `.tif/.tiff`- (LATTICE) of `.dng`-bestanden. |

### Algemene opties

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `-o, --output PATH` | een nieuwe map met tijdstempel onder uw standaardprojectpad (`~/Chloros Projects` tenzij anders geconfigureerd) | Projectmap om aan te maken of te hergebruiken. Als de map al een `project.json` bevat, wordt er een `_1`/`_2`-broermap aangemaakt in plaats van deze te overschrijven. |
| `-n, --project-name NAME` | auto (tijdstempel) | Projectnaam. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` maakt gebruik van een Chloros+ neurale debayer; langzamer maar van hogere kwaliteit. |
| `--vignette / --no-vignette` | `--vignette` | Vignetteringscorrectie. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflectiekalibratie (gebruikt paneeldoel indien gevonden, NIST-kalibratie per serienummer voor LATTICE). Voor LATTICE multispectraal fungeert dit tevens als de schakelaar voor het **product** van de reflectantie — zie [Exportschakelaars per product](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | uit | Pas PPK GNSS-correcties toe uit sidecar-bestanden. |
| `--exposure-pin-1 MODEL` | uit | Zet de „pin-1&quot;-model van een dual-camera-opstelling vastzetten. |
| `--exposure-pin-2 MODEL` | uit | Het „pin-2”-model vastzetten. |
| `--recal-interval SECONDS` | 0 | Dwing het opnieuw uitvoeren van kalibratieberekeningen om de N seconden tijdens de opnametijd. |
| `--timezone-offset HOURS` | lokaal | Overschrijf de tijdzone-offset die in de uitvoermetadata is ingebakken. |
| `--format FORMAT` | `TIFF (16-bit)` | Een van `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | geen | Vegetatie-indexen (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Dwing het startpunt van de pijplijn af voor LATTICE TIFF’s (Survey3 .raw blijft ongewijzigd). Ook de nooduitgang waarmee een opname zonder **raw** toch kan worden verwerkt — zie [Hoe een opnamemap eruitziet](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | on | Het lineaire debayered-product uitvoeren (`Debayered_Images`). Zie [Exportschakelaars per product](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | aan | Geef het weergavevoorbeeld weer (`Preview_Images`): RGB = witbalans (DAQ-lichtbron indien beschikbaar, anders grijze wereld) + gamma; multispec = valsekleuruitrekking. |
| `--radiance / --no-radiance` | aan | Geef float32-straling weer (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referentie voor het LATTICE-reflectantieproduct: `auto` = QA-passing in-frame-doel is de absolute referentie, DAQ-downwelling (ρ = π·L/E) als terugvaloptie; `target` = strikt (geen DAQ-vervanging); `daq` = DAQ-autoritatief. Zie [Exportschakelaars per product](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | geen | Map met per eenheid **gemeten** doelsreflectiescans per eenheid (`<serial>.csv`); valt terug op de nominale T3/T4P-spectra bij een ontbrekend resultaat. |
| `--array-alignment / --no-array-alignment` | ingeschakeld | LATTICE-arrays: pas de module-naar-module-uitlijning toe die is vastgelegd in het `Chloros:Alignment*` XMP-bestand van elke opname op elk verwerkt product (debayered / preview / radiance / reflectance / index). Geen actie voor beelden zonder de tags. |
| `--array-alignment-crop / --no-array-alignment-crop` | bijsnijden | Bijsnijd uitgelijnde exporten tot het gemeenschappelijke overlappingsgebied van de array, zodat alle modules één voetafdruk delen; `--no-…` behoudt het volledige sensorcanvas (zwart opgevuld buiten de bron). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Herbemonstering voor de uitlijningsvervorming. `nearest` behoudt de exacte DN-waarden van de bron (geen vermenging van radiometrische waarden tussen pixels). |

### Opties voor doelwitdetectie

| Vlag | Beschrijving |
| --- | --- |
| `--min-target-size PIXELS` | Minimale paneel-doelgrootte (px) voor detector. |
| `--target-clustering 0-100` | Clustergevoeligheid. |
| `--target / --targets` | Behandel de invoermap als alleen-doelpaneel (detectie van overzichtsscans overslaan). |

### Voorbeelden

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Exportschakelaars per product (LATTICE multispectraal)

LATTICE-verwerking splitst zich **in één doorloop**op in alle toepasselijke producten. De vier schakelaars per type — `--debayered`, `--preview`, `--radiance`, `--reflectance` — staan allemaal**standaard AAN**; gebruik het formulier `--no-<type>` om er één uit te schakelen. RGB-mastercams geven uitsluitend debayered + preview weer (geen straling/reflectie), dus `--radiance`/`--reflectance` hebben voor hen geen effect. De schakelaars worden genegeerd voor Survey3 `.raw` (die het standaardreflectie-/doelpad volgt). *(De oude vlag `--radiometric-output {reflectance,radiance,sensor-response}` is **verwijderd** en vervangen door deze schakelaars; er is geen `sensor-response`-niveau meer.)*

| Product | Uitvoer | DAQ-downwelling nodig? |
| --- | --- | --- |
| `--debayered` | Lineaire demosaïc (`Debayered_Images`). | Nee. |
| `--preview` | Voorbeeldweergave (`Preview_Images`): RGB = WB + gamma; multispec = false-kleuruitrekking. | Nr. |
| `--radiance` | float32 W/m²/sr/nm uit de volledige radiometrische keten (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflectie ρ (`32768` = 1,0), Pix4D-ready. | **Ja**, tenzij het wordt verankerd door een QA-goedgekeurd doel binnen het beeld (zie hieronder). |

`--reflectance-source` kiest de reflectantiereferentie:**`auto`**(standaard) maakt van een QA-goedgekeurd doel binnen het beeld de**absolute referentie**— de aan het doel gekoppelde empirische lijnketens worden gekruisscoreerd op vastgehouden-out-panelen en wordt de gemeten winnaar toegepast — waarbij wordt teruggevallen op de DAQ-downwelling-grens (ρ = π·L/E) wanneer er geen doel aanwezig is of de kwaliteitscontrole mislukt;**`target`**is strikt (geen DAQ-vervanging);**`daq`**kiest voor het DAQ-bepalende gedrag. De doelgeometrie (ArUco / vaste ROI / strook) is afkomstig uit de doelconfiguratie van het project; `--target-reflectance-dir DIR` bewaart per eenheid**gemeten** scans (`<serial>.csv`) per eenheid, opgezocht aan de hand van het serienummer/de QR-code van de doeleenheid, met de nominale T3/T4P-spectra als back-up.

Het DAQ-reflectantiepad bepaalt automatisch de **tijdstempel-gematchte neerwaartse straling**op basis van een geregistreerde**`.daq`**(DAQ-U/M/E)**of een native DAQ-M `.csv`**die naast de beelden is aangetroffen. Als een kalibratiebundel per camera of per DAQ niet lokaal in de cache is opgeslagen, haalt de pijplijn deze**automatisch op uit AWS** bij het eerste gebruik (vereist eenmalig internet; opgeslagen onder `~/.chloros/`).

#### Reflectantiepixels lezen (Pix4D / Metashape / je eigen scripts)

Reflectantie wordt opgeslagen als een geheel getal (DN), en **de DN die ρ = 1,0 aangeeft, is afhankelijk van de broncamera**:

| Bron | ρ = 1,0 is | Hoe te herkennen |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (speling tot ρ 2,0) | XMP `Chloros:PixelScale=32768` is in het bestand gestempeld. |
| Survey3 | `65535` (afgekapt bij ρ 1,0) | Geen `Chloros:*` XMP-tags — die afwezigheid *is* het signaal. |

**Lees `Chloros:PixelScale` en deel erdoor** in plaats van uit te gaan van een constante. De tag is gedefinieerd in het uint16-domein, dus hij blijft `32768` in alle uitvoerformaten die herschalen — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` en `TIFF (32-bit, Percent)` zijn allemaal zelfbeschrijvend (normaliseer het opgeslagen dtype eerst terug naar uint16: ×257 vanaf 8-bit, ×65535 vanaf float).

> **Eén geval heeft per ontwerp geen schaal.** Wanneer een 8-bit-bronopname (BayerRG8) wordt weggeschreven als 8-bit-TIFF, *knipt* de pijplijn de waarden af naar 0..255 in plaats van ze te herschalen, zodat elke waarde boven ρ≈0,008 wordt afgevlakt tot 255 en wordt het bestand niet geschaald. Chloros laat daar opzettelijk zowel de tupel `Chloros:PixelScale` als de tupel `MicaSense:RadiometricCalibration` weg en legt uit waarom. **Als de tag ontbreekt in een LATTICE-reflectantiebestand, ga dan niet uit van een schaal — exporteer opnieuw in 16-bit of 32-bit** in plaats van pixels te delen die nooit deelbaar waren.

#### EXIF-gegevens worden meegenomen bij de export

`process` kopieert het **GPS-blok en de ExifIFD** van de bronopname naar elk product, zodat een
export de volgende gegevens bevat: `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` en
`CameraSerialNumber` naast de georeferentie.

**`FocalLength` is niet optioneel voor fotogrammetrie.** Pix4D berekent de grondresolutie (GSD) op basis van
brandpuntsafstand plus hoogte; als de tag ontbreekt, wordt er teruggevallen op een volstrekt verkeerde schaal. Bij een
vlucht boven een sinaasappelboomgaard met 49 opnames veranderde de ontbrekende tag een terrein van 411 m × 160 m in een gereconstrueerd
terrein van 47,8 km × 13 km — een ortho van 455 MP die grotendeels uit ‘nodata’ bestond, wat vervolgens werd geïnterpreteerd als een tegelingprobleem en
een BigTIFF-probleem voordat iemand de GSD had gecontroleerd. Als je ortho op een onwaarschijnlijke
schaal, voer dan eerst `exiftool -FocalLength` uit op het geëxporteerde product.

De kopie is bewust **niet** `-all:all`: de structurele tags van IFD0 verstoren de LATTICE-uitvoer wanneer
worden gekopieerd, en `ExifImageWidth` / `ExifImageHeight` zijn uitgesloten omdat ze de
*bron*-opname — een export waarvan het formaat ooit is aangepast, zou anders afmetingen bevatten
die in tegenspraak zijn met het eigen raster. XMP wordt direct geschreven in plaats van gekopieerd, omdat ExifTool
XMP-tags van dezelfde aanroep verwijdert wanneer het XMP-blok wordt gekopieerd (waardoor de MAPIR
kalibratietags zouden verdwijnen).

### Waar de uitvoer terechtkomt

De bestanden worden **in de projectmap opgeslagen, gegroepeerd per camera en vervolgens per bestandsformaat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

De cameramap is `LATT-<sensor>-<lens>-F<filter>` voor LATTICE (overeenkomend met de EXIF-tag van de opname
`Model`) en `<model>_<filter>` voor Survey3 — twee camera’s die een sensor en filter delen, maar
in lens; houd daarom aparte mappen aan, omdat vignettering, beeldhoek en vervorming verschillen. De map met
formaat volgt op `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` of `tiff32` voor
`TIFF (32-bit, Percent)`.

> **Elk geëxporteerd product behoudt de naam van het BRON-bestand.** Een radiance-export van
> `capture_…_raw.tif` heet nog steeds `capture_…_raw.tif` — het staat alleen in
> `tiff32/Radiance_Images/`. **De map identificeert het product, niet de bestandsnaam**, dus bij het zoeken met glob-symbolen
> naar `*radiance*.tif` wordt niets gevonden; zoek in plaats daarvan op de mapnaam.

### Opnames van de lichtsensor — gekalibreerd `.daq` + `.csv`

`process` verwerkt ook de `.daq`-opnames in je invoermap, en het heeft **geen**
beeldmateriaal nodig om dit te doen: een DAQ-U / DAQ-M / DAQ-E die op zichzelf wordt gevlogen, is een volledige
opname, en een map die alleen `.daq`-bestanden bevat, is een geldige invoer.

Een DAQ kan **zonder** kalibratie worden opgenomen — dat is wat de openbare
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) recorders
(`record_daq.py`) standaard doen: ze schrijven ruwe sensortellingen op en voorzien het bestand van een tijdstempel, zodat
Chloros de fabriekskalibratie van die sensor **via het seriële poortje** ophaalt (eerst uit de lokale cache,
vervolgens uit de MAPIR-cloud) en deze toepast. `process` schrijft het resultaat terug:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

De `.csv` bevat één rij per meting: UTC-tijdstempel, integratietijd, totaal vermogen,
fotopische/scotopische lux, PPFD (en de blauw/groen/rood-verdeling), piekgolflengte, gevolgd door het
volledige spectrum op het eigen golflengteraster van de sensor. De `.daq` wordt opnieuw geïmporteerd zonder
een tweede keer te worden gekalibreerd.

Bij succes rapporteert de run `Light-sensor products written: N (calibrated .daq + .csv)`.
De tekst tussen haakjes beschrijft wat er daadwerkelijk is geschreven, dus het luidt
`(RAW COUNTS — this sensor has no calibration bundle)` voor een bundelsensor zonder bundel en
`(N calibrated, M raw counts)` voor een map die beide bevat. De eigen
`[DAQ-EXPORT]`- en `[RUN-SUMMARY]`-kopteksten van de backend ontlenen hun bewoording op dezelfde manier — geen van
de drie mag een ruwe export ‘gekalibreerd’ noemen.

Een DAQ-U / DAQ-M / DAQ-E-opname waarvan de kalibratiebundel niet kan worden opgehaald — je bent
offline, of die sensor heeft geen kalibratie in het bestand — wordt **met een reden overgeslagen** op een
`[DAQ-EXPORT]`-regel, en wordt nooit weggeschreven als een „gekalibreerd“ bestand met ruwe tellingen.
Maak verbinding met het internet en voer de bewerking opnieuw uit. De reden is degene die de lezer daadwerkelijk
voor dat bestand heeft vastgesteld (onleesbaar schema, geen bundel, een schrijffout), en de
samenvatting vermeldt **afzonderlijke** redenen — twintig bestanden die om één oorzaak zijn overgeslagen, worden weergegeven als één
oorzaak, niet als twintig herhalingen daarvan.

#### DAQ-A-opnames exporteren als ruwe tellingen

De **DAQ-A**-serie dateert van vóór het systeem met bundels per serienummer en heeft geen kalibratiebundel
om op te halen — in plaats daarvan wordt het in het veld gekalibreerd tegen een reflectiedoel, en dat is
de reden waarom het er nooit een nodig had. Door die opnames te weigeren, konden ze hun
cijfers helemaal niet meer verkrijgen, dus exporteren ze onder een **andere naam**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Een andere bestandsnaam in plaats van een vlag in het bestand, omdat de claim intact moet blijven
wanneer deze als kale naam via e-mail wordt doorgestuurd. De `.csv`-header vermeldt
`raw spectral sensor counts (NOT irradiance)` en waarschuwt dat de waarden vergelijkbaar zijn
**binnen** het bestand — wat precies is waarvoor ze bij doelgerichte kalibratie worden gebruikt — en
niet tussen sensoren onderling. De vermogensafhankelijke fotometrische kolommen (totaal vermogen, fotopische en
scotopische lux, PPFD) worden geschreven als **NULL** in plaats van geïntegreerd uit tellingen, en het
samenvattingsrapport vermeldt `RAW COUNTS`, dus wat in een logbestand als „geëxporteerd“ staat, kan niet als bestralingssterkte worden geïnterpreteerd.

Oudere **v1.01 / v1.02**-opnames (een DAQ-A-SD schrijft deze) bevatten geen tijdstip per meting,
alleen het tijdstip waarop het bestand is geschreven. De image↔downwelling-matcher weigert ze nog steeds — het matchen van een
frame met een schrijftijdstip zou onzichtbare fouten opleveren — maar de exporter leest ze, en
de CSV geeft `clock=daq_created_on` weer, zodat het product aangeeft op welke klok het draait.

### Opmerkingen

- `process` detecteert automatisch of je map Survey3, LATTICE of gemengd is.
- De voortgang wordt via Server-Sent Events doorgegeven; de CLI toont live voortgang per thread (Detecteren, Analyseren, Verwerken, Exporteren).
- Voor Linux /Jetson controleert de CLI de swapruimte en kan een waarschuwing geven voordat grote mappen worden verwerkt. De textuurbewuste debayer past ook automatisch een GPU-frequentiebeperking toe op Jetsons met laag stroomverbruik (Nano, Orin Nano).
- Bij succes rapporteert de run hoeveel beeldproducten er zijn geschreven (`Image products written: N`).

#### Een run die geen beelden schrijft, mislukt

Als u om producten hebt gevraagd en de run **geen** — alleen `project.json` en
`calibration_data.json` — dan beschouwt `process` dit als een fout: het geeft
`Processing finished but wrote no image products.` weer en **sluit af zonder-nul**, zodat een script dit kan
detecteren. Het bericht vermeldt de projectmap en de gebruikelijke oorzaken:

- de invoermap werd niet herkend als een opname (controleer de indeling en `--input-level`), of
- elk aangevraagd product werd overgeslagen omdat het niet van toepassing was op die camera’s (bijvoorbeeld het vragen om
  stralingskracht/reflectie van camera’s die alleen RGB ondersteunen).

Voer-voer de bewerking opnieuw uit met `--verbose` en controleer het backend-logboek op regels met `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
die per camera uitgeslagen producten verklaren dieanders de uitvoer van de CLI niet zouden bereiken.

Een opzettelijke run met alleen metadata — alle producten uitgeschakeld en geen `--indices` — is nog steeds een
**succes**, omdat een lege beelduitvoer daar het juiste resultaat is.

Dat geldt ook voor een **run met alleen de lichtsensor**: een map met `.daq`-opnames bevat per definitie geen beelden om te exporteren
, en de run wordt beoordeeld op basis van de gekalibreerde `.daq` / `.csv` die in plaats daarvan is geschreven.

---

## `chloros-cli login`

Verifieer deze machine met een Chloros+ cloudaccount. Inloggegevens worden veilig opgeslagen in `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Voorbeelden

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (door het te verwijderen uit, of delen ervan te dupliceren, het wachtwoord). Bij een 401-fout probeert de CLI automatisch opnieuw, eerst met `$$` eraan toegevoegd, daarna met een ontdubbelde helft van het wachtwoord; als een nieuwe poging slaagt, logt het je in en geeft het de juiste syntaxis met enkele aanhalingstekens weer voor de volgende keer.

> **Gebruik zonder grafische interface/via script: geen geslaagde sessie betekent een interactieve prompt, geen snelle foutmelding.** Elk commando dat een backend start (`process`, `status`, `export-status`, `time-sync`, …) wordt uitgevoerd zonder een in de cache opgeslagen licentie/sessie, komt terecht in een interactieve `Email:` / `Password:`-prompt op stdin voordat het verdergaat. Een onbeheerde taak zonder sessie in de cache zal daarom blijven hangen in afwachting van invoer — voer `chloros-cli login EMAIL PASSWORD` één keer per machine uit voordat u headless-taken inplant.

---

## `chloros-cli logout`

Wist de in de cache opgeslagen sessie en dwingt een nieuwe aanmelding af bij de volgende aanroep.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Geeft het huidige licentieniveau (Iron/Copper/Bronze/Silver/Gold), de geauthenticeerde gebruiker en het aantal apparaatkoppelingen weer.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Controleert de voortgang van de live Thread-4-export. Kan veilig worden aangeroepen **tijdens** een `process`-uitvoering vanuit een andere shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Stel de weergavetaal van de CLI in (38 talen worden ondersteund, waaronder CJK, RTL en Indic). Schakelt soepel over naar Engels op verouderde consoles die het script niet kunnen weergeven.

```
chloros-cli language [LANG_CODE] [--list]
```

### Voorbeelden

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Opdrachten voor de projectmap

Deze beheer de standaardlocatie van de projectmap (gedeeld met de GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Alleen voor Jetson. Controleert `version_url` uit `/etc/chloros/update.conf` en biedt aan om de bijbehorende `.deb` te downloaden en te installerenX.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Op Linux /Jetson voert de CLI ook **bij elke opstart een automatische updatecontrole uit** (zonder blokkering, het commando loopt nooit vertraging op): het leest `/etc/chloros/update.conf`, slaat het resultaat 1 uur op in de cache in `~/.chloros/update_cache.json` en geeft `Update available: vX.Y.Z / Run: chloros-cli update` weer wanneer er een nieuwere versie beschikbaar is. Bijwordt momenteel overgeslagen bij elke fout en bij Windows.

---

## `chloros-cli selftest`

Voert een 7-staps rooktest uit: versie, beschikbaarheid van poorten, opstarten van backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), aanwezigheid van denoiser-model, gereedheid van CUDA+denoiser.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmaster-status en -besturing. De Chloros-host draait de PTP-grandmaster; LATTICE-cams en DAQ-E-units werken hieraan gekoppeld voor apparaatoverschrijdende tijdstempels.

| Subcommando | Beschrijving |
| --- | --- |
| `status` | Toon de status van de grandmaster, BMCA-prioriteiten en klokidentiteit. |
| `peers` | Geef een overzicht van slaves die via Delay_Req worden gedetecteerd (camera’s + DAQ-E-sensoren). |
| `cameras` | Per-camera PTP-status (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Het grandmaster-proces opnieuw starten. |
| `set-priority --priority1 N --priority2 N` | BMCA-prioriteiten overschrijven. |

### Voorbeelden

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-camerabesturing. Elk subcommando wordt via de backend Chloros gerouteerd; de backend beheert de cam-pool, zodat volgende CLI-aanroepen dezelfde open handle hergebruiken.

### Algemene opties (gemeenschappelijk voor de meeste subcommando’s)

| Vlag | Beschrijving |
| --- | --- |
| `-d, --device N` | Camera-index (standaard: 0). |
| `-s, --serial SN` | Specifiek serienummer; overschrijft `--device`. |
| `--serials SN1,SN2,…` | Door komma&#x27;s gescheiden serienummers voor gebruik met meerdere camera&#x27;s. |
| `--all` | Toepassen op elke gedetecteerde camera. |
| `--exposure US` | Belichtingstijd in microseconden. |
| `--gain DB` | Versterking in dB. |
| `--pixel-format FMT` | b.g. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Beeldafmetingen. |
| `--preset {default,high_quality,high_speed,triggered}` | Een instellingenpreset toepassen. Alle instellingen werken in vrije modus, behalve `triggered`, waarmee de camera wordt geactiveerd bij een hardware-flank op lijn 2 — als er niets is dat die lijn aanstuurt, zal de camera oneindig lang wachten in plaats van een opname te maken. |
| `-o, --output DIR` | Uitvoermap (standaard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-pakketgrootte. `auto` voert ICMP+GVSP-probes uit; `jumbo` = 9000; `standard` = 1500. |

### LATTICE Camera-workflow voor eerste verbinding

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referentie subcommando&#x27;s

#### Detectie en informatie

| Subcommando | Doel |
| --- | --- |
| `lattice info` | Lijst met aangesloten camera’s (fabrikant, model, serienummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Het hostsysteem analyseren voor een optimale cameraconfiguratie. `--no-discover` slaat de cameradetectie over (sneller, analyse uitsluitend via de netwerkkaart). |
| `lattice network [--fix] [--estimate] [--cameras N]` | NIC-instellingen controleren/aanpassen; bandbreedte/FPS schatten. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Stabiele-schema backend netwerkcapaciteit + aanbeveling voor array (levert `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` behoudt de gevraagde resolutie maar beperkt de doel-fps — lees `recommended.recommended_target_fps` en geef dit door als het verbindingsdoel; beschouw dit als een succes, niet als een fout. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Wat-als-analyse zonder camera’s te openen. **`--n-active` is het totale aantal camera’s op de verbinding, niet alleen die van deze array**— verhoog deze waarde wanneer standalone camera’s gelijktijdig streamen, of wanneer het verbindingsbudget wordt berekend op basis van een vraag die het aantal camera’s te laag inschat (standaard: `len(--models)`). Geeft altijd de geaggregeerde `Wire budget:` (vereiste MB/s versus botsingsveilige limiet) en `Max cameras:`-regels, en markeert `** OVER-SUBSCRIBED**` wanneer de array de verbinding overbelast — zie [Array fps &amp; burst-model](#array-fps--burst-model). |
| `lattice gpu` | GPU-status weergeven. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Controleer of werk de camerafirmware bij. Lokale `.fwa`-selectie is vastgezet: het bestand in `firmware/<MODEL_PREFIX>/` dat overeenkomt met de `MIN_FIRMWARE_VERSION` van de build wordt geflashed indien aanwezig (alleen de hoogste versie als fallback), dus een nieuwere leveranciersimage die op de schijf is opgeslagen, is inactief totdat die vastzetting wordt opgeheven — opzettelijk nieuwere releases bereiken apparaten via het ondertekende AWS-manifest, wat de voorkeur heeft wanneer er een nieuwere versie is. |
| `lattice presets [--apply NAME]` | Cameravoorinstellingen weergeven of toepassen. |
| `lattice status` | Live camerastatus. |

#### Opname

| Subcommando | Doel |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Enkel beeld. **Slaat standaard elk exporttype op** (`--processing all`); zie [Opname-exportniveaus](#capture-export-levels-the-all-default). `--levels` slaat een expliciete subset op (heeft voorrang op `--processing`); `--force-daq` schrijft de toegewezen DAQ-meting als een `.daq`-sidecar, zelfs bij een opname die uitsluitend uit onbewerkte gegevens bestaat. `--jpeg-quality` = JPEG kwaliteit 1–100 (standaard 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Streamen naar schijf tot Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Live MJPEG-voorvertoning via de browser. `--ae-damping` stelt de demping van de automatische belichting in (0,4–100). |

#### Sensorinstellingen

| Subcommando | Doel |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Elk GenICam-knooppunt lezen/schrijven. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Belichting &amp; AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Versterking &amp; automatische versterking. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Sensor-ROI &amp; binning. |
| `lattice format [--set FMT] [--list]` | Pixelformaat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardware-/softwaretrigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (geen vlaggen = one-shot witbalans) | Witbalansbewerkingen. Alleen voor RGB/Bayer-camera&#x27;s; geen bewerking (overgeslagen) op mono M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB-kleurverwerkingspijplijn. `natural` (standaard) is de goedkope live-afwerking; `enhanced` voegt defringe + vibrance + CLAHE lokaal contrast toe voor de volledige hub-parity-look tegen ~2× de afwerkingskosten per frame, dus een lagere **live** framerate — opgeslagen opnames krijgen hoe dan ook altijd de volledige afwerking. RGB /BAlleen voorayer-camera&#x27;s; wordt overgeslagen bij mono M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Weergaveverzadiging/contrast (camera&#x27;s met RGB-filter). Wordt overgeslagen bij mono M3M. |
| `lattice filter [--set NAME] [--list]` | Stel hetfiltermodel instellen (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Vermogen/thermische knooppunten van de sensor; schakel laagvermogen-stand-by in of uit. |

#### Kalibratie &amp; Sensoren

| Subcommando | Doel |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibreren aan de hand van een reflectiedoel. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Ingebouwde-lichtsensorcommando’s. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Vignetcorrectie toepassen op bestaande beelden. |

#### Meerdere camera’s (transiënte sessies)

| Subcommando | Doel |
| --- | --- |
| `lattice multi-info` | Geef een overzicht van alle camera&#x27;s met synchronisatierollen. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Eén gesynchroniseerd beeld van elke camera. Slaat **standaard alle exporttypes op**wanneer een permanente array is aangesloten; de fallback voor de tijdelijke-array-fallback is**alleen gedebayerd** (voer eerst `array-connect` uit voor de rest). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Gesynchroniseerde frames streamen (transient). |
| `lattice multi-test [--count N]` | GPIO-synchronisatietiming test. |
| `lattice multi-detect [--line LINE] [--json]` | Automatische detectie van GPIO-master/slave-bedrading. |

#### Uitlijning

| Subcommando | Doel |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus detector-/matcher-regelaars `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-regelaars `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinatie van meerdere frames `[--averaging mean\|median\|inlier_weighted]`, geometrische beperkingen `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, ruimtelijke beperking `[--roi X0,Y0,X1,Y1] [--mask PATH]` en per-slave-overschrijvingen `[--per-cam-override SN:KEY=VALUE]` (herhaalbaar) | Bereken uitlijningsprofiel op basis van live camera&#x27;s. `--prefilter` is standaard ingesteld op `gradient` (randkaart; komt overeen met de GUI/array-uitlijner — randen blijven behouden over spectrale banden heen). `--matcher flann` levert pas resultaat op bij meer dan ~5000 kenmerken; `--averaging median` is robuust tegen één slechte opname; `inlier_weighted` weegt op basis van het aantal overeenkomsten; `--lock-scale` projecteert naar de dichtstbijzijnde rotatie (geen schaal); `--lock-axis` zet één translatiecomponent op nul; `--mask` is van toepassing op elke camera (gebruik `--per-cam-override` voor instellingen per camera, bijv. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` weigert een kalibratie op te slaan waarvan de RMS-waarde van de herprojectie de drempel overschrijdt. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Leg één uitgelijnd multiband-frame vast. `--bit-depth` past zich standaard aan de camera aan; `--no-crop` behoudt het volledige beeld (opgevuld met zwart); `--interpolation` (standaard `linear`) en `--border-mode`/`--border-value` (standaard `constant`/0) regelen de CPU-warp — het GPU-pad is in elk geval bilineair. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Stream-uitgelijnde multi-band frames (dezelfde warp-regelaars als `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Details van het profiel weergeven. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Volgorde van lagen wijzigen. |

#### Index / Vegetatiewiskunde

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Volledige vlaggenreeks: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (herhaalbaar), `--capture-level raw|debayered|radiance|reflectance|unknown` (het opnamelevel dat is vastgelegd in de bronTIFF overschrijven; standaard: gelezen uit TIFF metadata), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Bij `--live` zijn ook de uitlijningsknoppen van toepassing: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Bij de symbolen van `--channel` wordt onderscheid gemaakt tussen hoofdletters en kleine letters.** De symboolzijde moet exact overeenkomen met de kanaalnamen van de presets (presets gebruiken kleine letters, bijv. NDVI = `red`,`nir` — zie `--list-presets`), en de bandzijde moet overeenkomen met een bandnaam in de uitgelijnde stapel (of een bandindex vanaf 0 zijn in de offline-modus). `--channel red=Red_660 --channel nir=NIR_850` werkt; `--channel RED=660` mislukt met een `channel_map missing entries`-fout.

#### Permanente verbindingen (Smart-Prep, GUI-equivalente workflow)

Deze commando’s houden camera’s open in de backend-pool bij verschillende aanroepen vCLI.

| Subcommando | Doel |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Eén camera aan de pool toevoegen (één camera, geen array). |
| `lattice cam-disconnect [--serial SN] [--all]` | Vrijgeven. |
| `lattice cam-list` | Geef een overzicht van de camera&#x27;s in de pool. |
| **`lattice array-connect`**|**Sluit een permanente gesynchroniseerde array aan (HET aanbevolen startpunt).** Voert de volledige GUI smart-prep-workflow uit. |
| `lattice array-disconnect [--array-id ID] [--all]` | Een array vrijgeven. |
| `lattice array-list` | Aangesloten arrays weergeven. |
| `lattice array-status [--array-id ID]` | Live fps, PTP, laatste fout. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Eén gesynchroniseerde opname van de live-array — Enkel / Continu / Interval / Snelste. **Standaard ingesteld op `all`** (één bestand per toepasselijk exporttype per camera). Overgeslagen camera&#x27;s (bijv. RGB uitgesloten van stralingskracht/reflectie) worden gerapporteerd met `Skipped: SN:<serial> (<reason>)`; de DAQ-waarde die voor reflectie wordt gebruikt, wordt naast de reflectie opgeslagen en gerapporteerd met `DAQ: <path>`. Zie [Opnamemodi, recorders en offline herverwerking](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Neem het live-beeld van de gecombineerde index op als video/GIF (geschikt voor monitoring; de gecombineerde stream moet geopend zijn). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Raw-Bayer-burst met hoge fps (geschikt voor analyse; offline herverwerken). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Een opgeslagen onbewerkte burst herverwerken tot gekalibreerde video(&#x27;s). |

##### `array-connect` Opties

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--serials SN1,SN2,…` | alle LATTICE-camera’s automatisch detecteren (minimaal 2 nodig) | De eerste in de reeks is de MASTER. Indien weggelaten, filtert de detectie op LATTICE (`TRI032*`) en verbindt ze allemaal. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synchronisatielijn. |
| `--target-fps F` | auto | Vuurfrequentie van de MASTER-trigger. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | De tier-kiezer overschrijven. |
| `--wire-ceiling-mbps MB_PER_S` | automatisch gedetecteerd | **Het aanhoudende bandbreedtebudget van de host, in MB/s — het getal waaraan de toewijzing van de gehele array is gekoppeld.** Verlaag dit wanneer de array GVSP-corrupt frames rapporteert: de automatische waarde is afgeleid van de door de NIC aangegeven linksnelheid, die USB-adapters, smalle PCIe-lanes en drukke gedeelde fabrics te hoog inschat. Wordt opgeslagen in het array-capture-blok van het project, zodat het bij het heropenen / CLI / SDK opnieuw verbinden wordt hersteld. Zie [Arraystatus](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Hardware-binning. |
| `--no-recommend` | uit | Sla de netwerkanalyse over. |
| `--no-ptp` | uit | PTP uitschakelen (tijdstempels tussen camera’s zijn dan **niet** vergelijkbaar). |

### Smart-AE / Smart-Capture

LATTICE-arrays voeren continu AE uit op de achtergrond zodra ze zijn aangesloten, maar het duurt even voordat een nieuw ingestelde scène is geconvergeerd. `array-capture --smart` is de **handige oplossing**: het wacht tot de AE op alle camera’s in de array is gestabiliseerd en start dan de opname. Gebruik dit wanneer je tijdens een sessie van scène wisselt.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Het stabilisatiebeleid is standaard conservatief: time-out van 5 s, stabiliteitsvenster van 1,5 s, tolerantie voor belichtingsspreiding van ±5 %. Pas dit aan via de SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) als je een ander gedrag van de automatisering nodig hebt.

### Exportniveaus voor opnames (de standaardinstelling van `all`)

Vanaf deze release zijn `lattice capture`, `lattice multi-capture` en `lattice array-capture` **standaard ingesteld op `--processing all`** — één opgeslagen bestand per exporttype dat voor elke camera geldt, in overeenstemming met het „Alles vastleggen“-gedrag van de GUI. De niveaus zijn:

| Niveau | Uitvoer | Geldt voor |
| --- | --- | --- |
| `raw` | Eénkanaals Bayer (monochrome camera’s: de enkele band) rechtstreeks vanaf de sensor. | Alle camera’s. |
| `debayered` | 3-kanaals BGR-demosaïek (mono-camera’s: 1-kanaals grijswaarden). | Alle camera’s. |
| `radiance` | float32 W/m²/sr/nm via de volledige radiometrische keten. | Alleen multispectraal (M3C/M3M) — **overgeslagen voor camera’s met een ‘RGB’-filter**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-ready. | Alleen multispectraal, en **alleen wanneer een DAQ is gekoppeld + de camera is gekalibreerd**; anders overgeslagen. |
| `preview` / `display` | Volledige GUI-voorbeeldketen (CCM + WB + gamma volgens het profiel van de camera). `lattice capture` noemt dit `preview`; `array-capture`/`multi-capture` gebruiken `display`. | Alle camera&#x27;s. |

Geef één enkel niveau door om alleen dat ene niveau op te slaan (`--processing debayered`). Wanneer je om `all` vraagt, worden niveaus die niet van toepassing zijn op een bepaalde cam overgeslagen (en gerapporteerd), zonder dat er een foutmelding wordt gegeven — een niet-aangesloten of niet-gekalibreerde camera krijgt nog steeds `raw` / `debayered` / `preview`.

Voor elk reflectantiekader wordt de daadwerkelijk gebruikte DAQ-meting van de neerwaartse straling geschreven naar een **`.daq`**-sidecar naast de beelden (zodat de opname later opnieuw kan worden verwerkt) en gerapporteerd op een `DAQ:`-regel.

### Hoe een opnamemap eruitziet

Elk exporttype komt in een **eigen submap** onder `-o` terecht, zodat bij een opname met meerdere niveaus de typen nooit door elkaar raken:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` is de tijdstempel van de opname en `<serial>` het serienummer van de camera, dus één gesynchroniseerde groep deelt een
tijdstempel over alle camera’s heen. **Let op de enige asymmetrie:** het `display`-niveau wordt opgeslagen in een map
met de naam `preview/`, terwijl de bestanden zelf `_display` in de naam behouden — map en achtervoegsel verschillen
alleen dat niveau. Onbekende niveaus vallen terug op een map met hun eigen naam, en als de submap
niet kan worden aangemaakt, wordt het bestand naar de output-root geschreven in plaats van verloren te gaan.

**Een ‘captures’-map opnieuw verwerken:**wijs `chloros-cli process` naar de**‘captures’-hoofdmap**
(`output/`). `process` importeert normaal gesproken alleen de map die je opgeeft, maar wanneer die map geen
afbeeldingen bevat en wel submappen heeft, gaat het automatisch dieper in de structuur — zodat de submappen op het hoofdniveau en de
hoofdmap `.daq` in één keer worden opgepikt. Elk niveau van een opname wordt geïmporteerd als één afbeelding, waarbij
de andere niveaus beschikbaar zijn als modi, in plaats van als één afbeelding per niveau.

Het direct benoemen van een **submap op een bepaald niveau** (bijv. `output/raw/`) werkt ook. Hierdoor blijft de hoofdmap
`.daq` achter, dus kopieer of wijs de DAQ-uitlezing ernaast aan wanneer je een radiometrisch
product opnieuw afleidt uit `raw/` — anders kan de tijdstempelvergelijking nergens op worden gebaseerd.

**De verwerking begint altijd bij `raw`.** Binnen elke opname is het ruwe frame de bron van de pijplijn;
`debayered`, `radiance`, `reflectance` en `preview` verschijnen als weergavemodi, maar worden nooit
teruggevoerd door de pijplijn. Het opnieuw verwerken van een afgeleid product zou vignettering, CCM en
stralingsberekeningen die al in de pixels zijn ingebakken opnieuw toepassen, dus Chloros weigert dit in plaats van
dubbel-verwerking. Twee gevolgen die de moeite waard zijn om te weten:

- De `index/`- en `composite/`-renders worden **nooit** verwerkt. Het zijn uitvoerbestanden, geen opnames —
  een NDVI LUT-render heeft geen zinvolle radiance-interpretatie.
- Een map met opnames die is geëxporteerd **zonder** `raw` (bijv. `array-capture --processing reflectance`) heeft
  geen geldige pijplijnbron. Die opnames worden normaal geïmporteerd en weergegeven, maar `process` slaat
  ze over en geeft dit aan:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Als je echt een afgeleid product moet doorlaten — een hub-sessie die is vastgelegd met
  `demosaic` ingeschakeld, of een legacy-map — dwingt `--input-level {raw,debayered,processed}` het
  instappunt af en overschrijft de overslaan-instelling. Die vlag is de bewuste nooduitgang; `auto` (de standaardinstelling)
  verwerkt nooit een opname zonder raw-gegevens.

### Overgeslagen opnames in arrays met gemengde filters

Wanneer je ‘RGB’ en multispectrale camera’s in één array combineert, slaat `array-capture --processing radiance` (of `reflectance`) de multispectrale frames op en **slaat** de RGB-camera’s over — de straling per Bayer-pixel is niet relevant voor een breedbandsensor. Het bestand CLI geeft elk opgeslagen bestand (met het exportniveau), elke geschreven `.daq` en elke overslaan expliciet weer, zodat het aantal bestanden geen verrassing is:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokens voor de reden van het overslaan volgen het patroon `<level>-not-applicable-to-rgb-cam`. Reflectie kan ook worden overgeslagen met `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, en met `dls-uncalibrated-band-<nm>` wanneer de band grotendeels buiten het radiometrisch gekalibreerde bereik van de DAQ-lichtsensor ligt (~374–974 nm) — van de leverbare SKU’s geldt dit alleen voor F988, dat de workflow met het reflectantiepaneel ondersteunt.

Gebruik `--processing debayered` (of `display`) om elke camera op te nemen, ongeacht het filtertype, of de standaardinstelling `all` om in één keer alle toepasselijke niveaus per camera te verkrijgen.

---

## Opnamemodi, recorders en offline herverwerking

Deze werken allemaal op een **persistent array** (voer eerst `array-connect` uit). Ze weerspiegelen het opnamepaneel van de GUI.

### `array-capture`-modi

`array-capture` is een enkele opdracht met vier sluitermodi plus een reeks exportschakelaars:

| Modus | Vlag | Gedrag |
| --- | --- | --- |
| **Enkel** *(standaard)* | (geen) | Eén gesynchroniseerde opnamegroep, daarna afsluiten. |
| **Continu** | `--continuous` | Aaneengesloten doorlopen tot `Ctrl+C`, `--count N`, of `--duration S`. |
| **Interval** | `--interval S` | Eén doorgang om de `S` seconden (gemeten vanaf het begin van elke doorgang), dezelfde grenzen. |
| **Snelste** | `--fastest` | Alleen ruwe gegevens + de toegewezen DAQ-waarde + de samengestelde index; slaat de berekeningen voor stralingskracht/reflectie/weergaveberekeningen, zodat het frame snel wordt weergegeven. Impliceert `--processing raw --force-daq`. Verwerk de opgeslagen `.daq` later opnieuw tot gekalibreerde producten. |

Exportschakelaars (te combineren met elke modus; alle delen de GUI/SDK eindpunt):

| Vlag | Effect |
| --- | --- |
| `--processing LEVEL` | Enkel exportniveau, of `all` (standaard). |
| `--levels L1,L2,…` | Expliciete subset van exporttypes (bijv. `raw,radiance,reflectance`); **heeft voorrang op `--processing`**. |
| `--aligned` / `--no-aligned` | Pas de niet-ruwe export van elk element aan aan het [uitlijningsprofiel](#uitlijning) van de array (geco-registreerd). Raw blijft ongewarped, maar draagt de transformatie mee in de metadata. Valt terug op niet-uitgelijnd (met een waarschuwing) als de array geen profiel heeft. |
| `--index` / `--no-index` | De vegetatie-index-overlay per camera opslaan / overslaan, indien deze is geconfigureerd. Standaard: weergeven. |
| `--force-daq` | Sla de toegewezen DAQ/DLS-meting op als een `.daq`-sidecar, zelfs als geen enkel gekozen niveau dit nodig heeft (bijv. een opname met alleen onbewerkte gegevens), zodat de frames offline opnieuw kunnen worden verwerkt tot reflectantie/index. |
| `--smart` | Wacht tot de AE voor alle camera&#x27;s stabiel is voordat je de opname start (zie [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF pixelcompressie. `deflate` (standaard) = verliesvrije zlib L1 + horizontale voorspeller, ~4,1 MB per frame in volledige resolutie; `none` = ongecomprimeerd, ~5× sneller schrijven bij ~6,3 MB per frame — gebruik dit voor maximale continue schrijfsnelheid wanneer de schijfruimte dit toelaat. Beide zijn verliesvrij en worden bij het importeren identiek weergegeven. |

> **Eenmalige schrijfTIFFn + het model voor continue schrijfsnelheid.**Opnames worden in**één**TIFF-bestandsdoorgang geschreven, met pixels + XMP + IFD0 Merk/Model (gemeten op Mono12 met volledige resolutie: 36 ms gecomprimeerd / 6,5 ms ongecomprimeerd, versus ~148 ms voor de oude methode van eerst schrijven en daarna herschrijven met ExifTool); het enige resterende werk voor ExifTool (bijwerken van de EXIF-sub-IFD) wordt uitgevoerd door een asynchrone achtergrondworker, en een frame is voltooid en klaar om te worden geïmporteerd, zelfs als deze nooit wordt uitgevoerd. Merk op dat DEFLATE-compressie de GIL van de Python vasthoudt, dus gecomprimeerde schrijfbewerkingen worden**niet**parallel uitgevoerd over de schrijfthreads per camera — een aanhoudende opname met 8 camera&#x27;s op volledige resolutie op sensorsnelheid (~10,4 fps) vereist `--compression none`**en** een schijf van NVMe-klasse (~500 MB/s aan aanhoudende schrijfbewerkingen). Dezelfde instelling is beschikbaar als `compression` op `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — gecombineerde-index video/GIF (geschikt voor monitoring)

Neemt alles op wat de **live gecombineerde-indexweergave** toont naar een `.avi` (en optioneel een `.gif`). Omdat het de live composietstream aftapt, moet de gecombineerde stream openstaan (bijv. de array wordt in de GUI in een voorbeeld weergegeven) om frames te kunnen vastleggen. Het controleert de voortgang elke 2 s en stopt bij `--duration`, `Ctrl+C` of wanneer de recorder zichzelf beëindigt.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--array-id ID` | alleen array | Doelarray (weglaten als er slechts één is aangesloten). |
| `-o, --output DIR` | `output` | Uitvoermap (backend-lokaal). |
| `--fps F` | `10` | Framesnelheid bij opname. |
| `--duration S` | tot Ctrl+C | Automatisch stoppen na `S` seconden. |
| `--gif` | uit | Schrijf ook een geanimeerde GIF. |
| `--gif-only` | uit | Schrijf alleen een GIF (geen `.avi`). |

### `array-burst` — raw-Bayer-burst met hoge fps (geschikt voor analyse)

Leest rechtstreeks uit de gesynchroniseerde groepsbuffer van de opnamelus — **geen kalibratieketen, geen exiftool, geen liveweergave nodig** — zodat het op de volledige opnamesnelheid van de camera draaitvolledige opnamesnelheid van de camera. Schrijft raw-frames + een manifest per frame + één `.daq` per afzonderlijke DLS-meting onder `<output>/bursts/<base>/`. Offline opnieuw verwerken (volgende commando), of `--build`X door om dit onmiddellijk bij het stoppen te doen.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--array-id ID` | alleen array | Doelarray. |
| `-o, --output DIR` | `output` | Uitvoermap (burst komt terecht in `<DIR>/bursts/<base>/`). |
| `--duration S` | tot Ctrl+C | Automatisch stoppen na `S` seconden. |
| `--max-frames N` | onbeperkt | Automatisch stoppen na `N` onbewerkte frames. |
| `--build` | uit | Na het stoppen de burst onmiddellijk opnieuw verwerken (hetzelfde als `array-build-video`). |
| `--products …` | `combined:index` | Met `--build`: welke video(&#x27;s) moeten worden samengesteld (zie hieronder). |
| `--fps F` | `10` | Met `--build`: fps van de uitvoervideo. |
| `--save-tiffs` | uit | Met `--build`: sla ook per frame gekalibreerde TIFF-bestanden op. |
| `--gif` | uit | Met `--build`: schrijf ook geanimeerde GIF(s). |

### `array-build-video` — een opgeslagen burst offline opnieuw verwerken

Koppelt elk RAW-frame qua tijd aan de dichtstbijzijnde opgeslagen `.daq`-meting en voert het door **dezelfde straling/reflectie/indexketen als de importpijplijn**, waarbij één of meer video’s worden gerenderd.

`--products` is een door komma’s gescheiden lijst van `kind:level`-items, waarbij `kind` ∈ `per_cam` | `combined` en `level` ∈ `radiance` | `reflectance` | `index`. Een losstaande `level` (zonder `kind:`) wordt standaard ingesteld op `per_cam`. De standaardinstelling is `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--burst-dir DIR` | (verplicht) | Pad naar de burst-map (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-lijst, zoals hierboven. |
| `--fps F` | `10` | Frames per seconde (fps) van de uitvoervideo. |
| `--save-tiffs` | uit | Sla ook per frame gekalibreerde TIFF-bestanden op naast de video(&#x27;s). |
| `--gif` | uit | Schrijf ook geanimeerde GIF(&#x27;s). |

> **Kies de juiste recorder.** `array-record` is *geschikt voor monitoring* — deze legt het live-composietbeeld vast zoals-weergave vastlegt en de stream geopend moet hebben. `array-burst` → `array-build-video` is *geschikt voor analyse* — het slaat ruwe sensorgegevens op volle snelheid op en reconstrueert daarna gekalibreerde stralings-/reflectantie-/indexvideo’s, zonder dat een liveweergave nodig is.

### Mono (M3M) enkelbandcamera’s

De **M3M**-serie is de monoversie van de Bayer**M3C**: één smalband-interferentiefilter per camera (`M3M-<lens>-F<wavelength>`, bijv.bijv. `M3M-L87-F685`), waardoor de sensor een**enkele grijswaardenband** levert zonder Bayer-mozaïek. Er hoeft niets te worden gedemosaïceerd, er is geen kanaaloverschrijding die moet worden gescheiden en er hoeft geen witbalans te worden ingesteld — de hele kleurverwerkingspijplijn van RGB is simpelweg niet van toepassing.

Wat dat betekent op de CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detecteren een monochrome camera en**slaan deze over met een bericht van één-regel** in plaats van zinloze instellingen door te voeren. Ze werken nog steeds normaal met een RGB /Bayer M3C-camera in dezelfde sessie.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** werken nog steeds — straling en reflectie zijn *per-band* radiometrische kaarten en zijn perfect gedefinieerd voor één band. Mono-frames bevatten een **identiteits**-sensorresponsmatrix (geen 3×3-unmix), dus het vlak doorloopt de kalibratieberekeningen ongewijzigd.
- **Een enkele monocamera kan geen vegetatie-index produceren.**NDVI / NDRE / enz. hebben ten minste twee banden nodig (bijv. Red + NIR). Om een index te verkrijgen met monocamera&#x27;s, richt je**meerdere** M3M-camera&#x27;s op verschillende golflengten, voeg je ze samen tot één multiband-stack en bereken je *die*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` symbolen moeten **precies** overeenkomen met de kanaalnamen van de preset (hoofdlettergevoelig; NDVI&#x27;s zijn kleine letters `red`,`nir` — zie `--list-presets`), en de bandzijde benoemt een band in de uitgelijnde stapel (de offline-modus accepteert ook bandindexen vanaf 0, b.b.v. `--channel red=0 --channel nir=1`).

Het onderscheidende kenmerk in de hele stapel is het `M3M`-token in de modelstring (dit komt nooit voor in een `M3C`-string), weergegeven in de GUI/SDK weergegeven als `is_mono`.

---

## Configuratie en afstemming van de host-NIC (LATTICE-arrays)

LATTICE-camera’s streamen GVSP via de Ethernet-adapter van de host, dus voor arrays met meerdere camera’s zijn het **stuurprogramma**en de**grootte van de ontvangstring** van de adapter net zo belangrijk als de verbindingssnelheid. Verkeerde instellingen worden weergegeven als een `FRAMES WILL DROP` / `Reduce ROI to enable`-poort in het paneel „Array Settings“ (en in `lattice network-analysis` / de `analyze_array_network()` van de „SDK“), zelfs als de camera’s zelf in orde zijn.

### USB 10GbE-adapters — Realtek RTL8157 (“Realtek USB 10GbE Family Controller”)

| Item | Vereiste waarde | Waarom dit belangrijk is |
| --- | --- | --- |
| **Stuurprogrammaversie**|**≥ v10.67 (jan. 2026)**, INF `rtump64x64sta.inf` | Het verouderde**2016**-stuurprogramma (v10.65, `rtump64x64.inf`) verwerkt uitschakelen en bugchecks met**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**bij het afsluiten, opnieuw opstarten of in de slaapstand gaan. De overgang loopt vast (~5 minuten time-out), de gebruiker schakelt het apparaat geforceerd uit, en herhaalde onjuiste uitschakelingen**beschadigen de WMI-repository**(PowerShell/tools beginnen fouten te vertonen met `Invalid class`) en**zetten de USB-stack klem** bij de volgende opstart (netwerkkaart kan niet worden ingeschakeld; USB-drives worden niet meer herkend). Voer een update uit via realtek.com (of de leverancier van de dongle) voordat u vertrouwt op een schone herstart. |
| **Ontvangstbuffers**— trefwoord `ReceiveBufferLen` |**256**(maximum driver) | De RX-ring van de NIC. De standaardinstelling van de driver van**32**laat slechts ~0,26 MB bruikbare ringruimte over — veel te klein voor een burst met meerdere camera’s — dus het arraypaneel meldt `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` en blokkeert verbindingen. Bij**256**is de ring groot (**~13,5 MB gemeten op de 10GbE-host in het lab**), wat de RX-pijplijn echte speling geeft voor GVSP-bursts met meerdere camera’s. (Of een bepaalde configuratie daadwerkelijk *verbinding maakt*, wordt bepaald door twee controles — de **drain-bewuste**toelatingscontrole en de**aggregate over-subscription**-controle — niet een ruwe vergelijking tussen burst en ring; zie [Array fps &amp; burst-model](#array-fps--burst-model).) |
| **Ontvangst-URB’s**— trefwoord `PendingReceives` |**64** (max.) | USB-verzoekblokken in behandeling; worden samen met ontvangstbuffers vergroot voor burst-absorptie. |
| **Jumbo Frame** — trefwoord `*JumboPacket` | **9014** | Nodig voor GVSP-pakketten van 9000 bytes (6× minder pakketten/frame dan bij 1500). |

> ⚠️ **Een update van het NIC-stuurprogramma ZET deze geavanceerde eigenschappen TERUG naar de standaardinstellingen.**Pas na het bijwerken of vervangen van het adapterstuurprogramma**opnieuw toe** `ReceiveBufferLen=256` en `PendingReceives=64` opnieuw toe, anders zal het array-paneel weer blokkeren, ook al is er „niets veranderd aan de hardware“. Dit is de belangrijkste oorzaak waarom een systeem dat eerder werkte plotseling geen verbinding meer wil maken.

Pas de instellingen toe vanuit een **PowerShell met beheerdersrechten** (vervang de naam van je adapter, bijv. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` is van toepassing op USB 10GbE-adapters.** Het detecteert nu het type adapter en stelt het juiste ‘receive-ring’-trefwoord in: `*ReceiveBuffers`→2048 voor PCIe-netwerkkaarten (Intel I219, enz.), of `ReceiveBufferLen`→256 + `PendingReceives`→64 voor de Realtek **USB** 10GbE-controller (die `*ReceiveBuffers` niet blootstelt). De doelwaarden worden begrensd tot de door elk stuurprogramma gerapporteerde maximumwaarde (`NumericParameterMaxValue`), zodat er nooit een waarde buiten het bereik wordt geschreven. Voer het uit vanuit een **bevoegd** terminalvenster; net als bij elke op het register gebaseerde aanpassing, wordt de wijziging pas van kracht na het herstarten van de adapter of het opnieuw opstarten van de computer. De handmatige `Set-NetAdapterAdvancedProperty`-opdrachten hierboven blijven een prima alternatief — ze worden direct toegepast (de adapter opnieuw koppelen) zonder herstart.

### Netwerkbasisprincipes (alle LATTICE-verbindingen)

- **Adressering:** link-local `169.254.0.0/16` (GigE Vision LLA). De host krijgt een statisch adres `169.254.x.x/16`; camera’s en DAQ-E wijzen zichzelf een adres toe binnen hetzelfde bereik. Geen DHCP/gateway vereist.
- **Pakketgrootte:**geef de voorkeur aan jumbo (9000), maar laat de auto-probe deze bepalen — deze voert bij elke verbinding een nieuwe meting uit en kijkt via een GVSP-probe al voorbij de ICMP-limiet van 1500 bytes van de camera, dus komt hij op jumbo uit waar de kabel dit daadwerkelijk aankan. Stel alleen vast op `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` als je het beter weet dan de probe, en geef de voorkeur aan per opdracht boven permanent: een pin slaat de probe over, dus als het pad daadwerkelijk geen 9000 kan verwerken, loopt**elke** capture vast met `SC_ERR_TIMEOUT -1011` (zie [Omgevingsvariabelen](#environment-variables)).
- **De RX-ring schaalt mee met `ReceiveBufferLen`:**bij de standaardwaarde `32` is de bruikbare ring ~0,26 MB (te klein voor een burst met meerdere camera’s); bij de maximale waarde `256` is deze groot (~13,5 MB gemeten op de 10GbE-host in het lab), wat echte speling biedt. Of een configuratie wordt toegelaten, wordt vervolgens bepaald door de ‘drain-aware’ toelatingscontrole**en** de hieronder beschreven controle op totale overinschrijving — niet door een ruwe vergelijking tussen burst en ring.

### Array-fps &amp; burst-model

Hoe het paneel ‘Array-instellingen’ (en `lattice analyze-array` / de `analyze_array_network` van de SDK) te lezen:

- **De burst wordt opgeteld per-camera bij het werkelijke pixelformaat van elke camera.**Mono**M3M**-camera’s streamen**Mono12 (2 B/px)**;**M3C**Bayer-camera’s streamen 8- of 12-bits (TRI032S zendt stilzwijgend BayerRG12 uit, zelfs wanneer BayerRG8 wordt aangevraagd). Een frame met volledige resolutie van 4 camera’s is dus**~12,6 MB als alle camera’s 8-bit zijn, maar ~25 MB met drie 12-bit monocamera’s**. De projectie bepaalt het formaat van elke camera op basis van het model (identiteitscache), zodat de burst overeenkomt met wat er daadwerkelijk via de kabel wordt verzonden — en niet uitgaat van een standaard aanname van BayerRG8.
- **Een USB-ethernetadapter heeft een maximum van 200 MB/s, ongeacht wat er op het typeplaatje staat.** De efficiëntietabel die een verbindingssnelheid omzet in een continu cijfer is afgeleid van PCIe; een USB-NIC geeft zijn *ethernet* verbindingssnelheid, maar wordt beperkt door de USB-bus en de bijbehorende driver. Een USB 10GbE-dongle haalde vroeger ~1063 MB/s „duurzaam“ — een getal dat nooit is getest — en de daaruit voortvloeiende onregelmatige snelheid beschadigde 6–18 % van de frames, terwijl er nog steeds een gezonde doel-fps werd gerapporteerd. USB-aangesloten NIC’szijn nu absoluut begrensd op **200 MB/s** (de limiet is de bus, dus het schaalbaar is niet met de nominale waarde; een USB 1 GbE-adapter haalt ~80 MB/s en ondervindt hier geen hinder van). `wire_ceiling_source` in het capability-record vermeldt dit in woorden, en `nic_is_usb` markeert het. Overschrijf beide met `--wire-ceiling-mbps`.
- **Admittance houdt rekening met de drain, niet met het geheelburst-versus-ring.** Een gelijktijdige burst hoeft alleen te passen binnen de *tijdelijke achterstand* = `max(0, Σ per-cam arrival − host drain) × emit_window`, niet de volledige burst. Op een fabric met een snelle host en trage cams (een **PCIe**10G-host + 4× 1 GbE-cams: aankomst ≈ 320 MB/s, afvoer ≈ 1063 MB/s) leegt de host sneller dan de camera’s vullen, backlog ≈ 0, dus full-res sim-emit**laat toe**ook al overschrijdt de burst van 25 MB de ring van 13,5 MB. Zet dezelfde vier camera’s achter een**USB**10GbE-adapter en de uitstroom is 200 MB/s, niet 1063 — de aanvoer loopt voor op de afvoer, en het verlies manifesteert zich als beschadigde frames in plaats van als een lagere framesnelheid. Op een 1 GbE-host zorgt de DLThr-ondergrens van 31,25 MB/s van de camera’s ervoor dat de aanvoer de afvoer overstijgt → het**blokkeert** terecht (voor *deze* klasse van blokkeringen: verklein de ROI of gebruik binning ≥ 2). Toelating is een van de **twee** connect-poorten — de andere is de hieronder beschreven controle op totale overinschrijving.
- **De geprojecteerde fps is een conservatieve bovengrens voor seriële opvraging.**De ophaallus van de host haalt momenteel de buffer van elke camera**serieel**op (~één verzendvenster per camera), dus de cyclus wordt begrensd door `max(readout+emit, N × emit)`, waarbij de verzending per camera wordt begrensd tot de**toegangsverbinding**van de camera (1 GbE ≈ 80 MB/s), niet aan de uplink van de host. Voor een array van 4 camera&#x27;s met volledige resolutie en 12-bits is dat**~2,8 fps**, wat overeenkomt met de gemeten ~2,7–3,0 fps. Dit is bewust**onafhankelijk van de belichtingstijd**, dus in donkere scènes kan de werkelijke waarde iets onder het plafond dalen naarmate de belichtingstijd toeneemt. Het seriële ophalen is de echte fps-begrenzer; door dit te parallelliseren zou het plafond stijgen in de richting van de uitstootsnelheid per camera.
- **Geaggregeerde overinschrijving is een harde verbindingblokkering.**De toewijzing van bandbreedte per camera heeft een ondergrens van**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), dus zodra de ondergrens wordt bereikt, kan de totale vraag (`per_cam × N`) de**collision-veilige limiet voor de kabel**(`sustained × sim_emit_factor`) overschrijden. Praktische limieten bij volledige resolutie op 1 GbE:**6 camera’s bij 1500 MTU, 9 met jumbo**. Deze limiet is uitsluitend afhankelijk van de kabel en de ondergrens — hij is**onafhankelijk van de framegrootte**, dus**binning en een kleinere ROI helpen NIET** (ze verlagen het aantal bytes per *frame*, niet het door GevSCPD geregelde aantal bytes per *seconde*); de enige oplossingen zijn minder camera’s, end-to-end jumboframes of een snellere netwerkkaart. Het symptoom zou pakketverlies bij GVSP zijn, niet een geleidelijke fps-vermindering, dus `analyze-array` zet de haalbare fps-waarden op nul en geeft `**OVER-SUBSCRIBED**`, en `array-connect` met een vastgelegde resolutie **weigert verbinding te maken** (de walk-down voert anders frame-binning uit, wat deze klasse van blokkades evenmin oplost). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` degradeert de weigering tot een luide waarschuwing voor benchwerk — zie [Omgevingsvariabelen](#environment-variables).

### Status van de array — welk subsysteem verliest frames

De `GET /api/camera/array/<array_id>/capability` van een aangesloten array bevat een live
`health`-blok, dat opnieuw wordt geëvalueerd in een voortschrijdend venster van **10 seconden**. Het splitst het frameverlies
op in de twee oorzaken die tegengestelde oplossingen vereisen, in plaats van één &quot;onvolledige&quot;
percentage te rapporteren dat geen van beide noemt:

| Veld | Wat het betekent | Welk subsysteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriële poort) | Het frame **kwam aan en was structureel defect**— GVSP-pakketverlies. |**Netwerk**: bandbreedte, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per seriële poort) | Het frame **is helemaal niet aangekomen**— de camera heeft niet geactiveerd, of er is niets vanuit verzonden. |**Trigger / synchronisatie**: M8-kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | De slechtste overdrachtssnelheid van de camera voor elk. | — |
| `per_cam_rate_pct` | Gecombineerd percentage onvolledige opnames per camera (beide oorzaken samen). | — |
| `stable_for_seconds` | Hoe lang elke camera onder de 0,01 % is gebleven. | — |

Boven de 5 % registreert de backend een `[array-health <id>] WARN`-regel waarin de splitsing wordt vermeld — bij de
eerste overschrijding, bij een wijziging van de ernst-bandwijziging, één keer per minuut zolang deze aanhoudt, en één keer wanneer
deze verdwijnt. De beschadigde helft drukt `[gvsp-corrupt <SN>]` af bij de eerste treffer per camera en
reden, gevolgd door een samenvatting elke 60 s. Elke evaluatie komt nog steeds terecht in het backend-logbestand;
de tellers lopen bij elke buffer verder, ongeacht wat er wordt afgedrukt.

Hetzelfde record vermeldt het nummer waaraan de gehele toewijzing is gekoppeld:

| Veld | Wat het betekent |
| --- | --- |
| `wire_ceiling_mbps` | Het geldende, aanhoudende bandbreedtebudget van de host, MB/s. |
| `wire_ceiling_source` | Waar dat getal vandaan komt, in woorden — bijv. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` of `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` wanneer `--wire-ceiling-mbps` (of het veld **Wire Budget** in de GUI) dit instelt. |
| `nic_is_usb` | `true` voor een USB-ethernetadapter — zie de limiet van 200 MB/s hierboven. |

**Interpretatie:** een `gvsp_corrupt_rate_pct`-waarde ongelijk aan nul terwijl `never_arrived_rate_pct` op 0 staat,
betekent dat de trigger en de kabelsynchronisatie perfect zijn en dat 100 % van het verlies zich op het netwerkpad
bevindt — verlaag `--wire-ceiling-mbps` en maak opnieuw verbinding. Het omgekeerde patroon wijst in plaats daarvan op de
synchronisatiekabel of de triggerlijn.

> **`--target-fps` is niet de regelaar voor beschadigde frames.** De GevSCPD-pacing wordt
> eenmalig bij het verbinden, dus het verlagen van de triggerfrequentie verandert de duty cycle en niet de
> burst-snelheid bij gelijktijdige uitzending. Een gemeten 5×-beperking van de vraag leverde geen verbetering op;
> het verlagen van de maximale bandbreedte van 240 naar 200 MB/s bracht hetzelfde systeem van 10,4 %
> beschadigde frames naar 0,00 %.

> **Automatische inkrimping halverwege de stream is niet beschikbaar op TRI032S-firmware.** Een actieve array
> kan dit niet zelf verhelpen; verbreek de verbinding en maak opnieuw verbinding, zodat de connect-time-picker
> opnieuw kan plannen met de nieuwe limiet.

### Symptoom → oplossing

| Symptoom (Array-instellingen / verbinden / `analyze_array_network`) | Oorzaak | Oplossing |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` gereset naar 32 (meestal na een stuurprogramma-update) | Stel `ReceiveBufferLen`→256, `PendingReceives`→64; open het paneel opnieuw (start de backend opnieuw op als deze de oude ringgrootte in de cache heeft opgeslagen) |
| Opstarten/afsluiten loopt vast; later `Invalid class` WMI-fouten, netwerkkaart kan niet worden ingeschakeld, USB-drives ontbreken | Oude Realtek USB 10GbE-driver uit 2016 → BSOD `0x9F` → gedwongen uitschakelingen | Werk de adapterdriver bij naar ≥ v10.67 (2026), pas vervolgens de bovenstaande instellingen voor de ontvangstring opnieuw toe |
| Verbinding lukt, maar levert een lagere resolutie op dan de native resolutie | Smart-prep heeft het frame automatisch verkleind om het aan te passen aan de verbinding | Upgrade de verbinding / accepteer de verkleining / `--force-tier slip-emit-and-capture` |
| Array rapporteert een gezonde doel-fps maar levert slechts een fractie daarvan; `health.gvsp_corrupt_rate_pct` niet-nul, `never_arrived_rate_pct` 0 | Het afgeleide kabelbudget van de host overschat wat deze daadwerkelijk aankan (typisch bij een USB-ethernetadapter, een smalle PCIe-lane of een gedeelde fabric) | Maak opnieuw verbinding met een lagere `--wire-ceiling-mbps` en controleer het statusblok opnieuw. **Niet** `--target-fps` — GevSCPD-pacing is vastgelegd bij het verbinden |
| Camera’s ontbreken in gepubliceerde groepen; `health.never_arrived_rate_pct` is niet nul, `gvsp_corrupt_rate_pct` is 0 | Trigger-/synchronisatiepad — de camera’s worden niet geactiveerd, dit is geen netwerkprobleem | Controleer de M8-synchronisatiekabel en `--line`; controleer of elk lid is ingeschakeld (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` overschreden in `analyze-array`, of verbindingsweigering bijvaste resolutie (`array over-subscribes the wire`) | De totale vraag per camera (minimaal 8 MB/s × N camera’s) overschrijdt de botsingsveilige bandbreedtelimiet — 6 camera’s op volledige resolutie via 1 GbE bij 1500 MTU, 9 met jumbo | Minder camera’s, jumboframes van begin tot eind, of een snellere netwerkkaart. **ROI/binning helpt NIET** (de limiet is onafhankelijk van de framegrootte). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` overschrijft de instellingen op de testbank (accepteert het pakketverlies) |

---

## `chloros-cli daq`

Commando’s voor spectrale sensoren. Twee klassen:
- **`pool-*`**— thin-HTTP-clients die de sensor aansturen via de persistente pool van de backend.**Dit is het ondersteunde pad, en het enige dat aanwezig is in de meegeleverde CLI.** De backend beheert het transport, zodat de GUI, de CLI en de SDK scripts allemaal één actieve handle delen in plaats van te strijden om de seriële poort.
- **Al het andere**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — directe hardware-toegang, hieronder beschreven ter volledigheid. Hiervoor is het `daq`-Python-pakket nodig, dat**in geen enkel geleverd artefact is opgenomen**: de gecompileerde CLI sluit het uit (`scripts/Build-CLI.ps1` stelt `--nofollow-import-to=daq` in, en de transporten `pyserial` / `bleak` / `zeroconf` bevatten het wel), en het PyPI-SDK-pakket bevat het evenmin. Ze werken alleen vanuit een source-checkout, dus beschouw ze als een MAPIR -intern ontwikkelingspad in plaats van iets om naar te grijpen.
- **`discover` / `list`** bevinden zich ergens tussenin: het zijn directe hardwarecommando’s vanuit een broncode-checkout, maar bij een uitgebrachte build vallen ze terug op `pool-discover` en voert de backend de scan uit. Scannen werkt dus overal — wat belangrijk is omdat het de enige manier is om de BLE-MAC van een DAQ-M te achterhalen.

> **`chloros-cli daq --help`** (en `-h` / `help`) geeft een overzicht van de `pool-*`-subcommando’s — de helptekst wordt bewust doorgestuurd naar de pool client, zodat deze de commando’s weergeeft die daadwerkelijk worden uitgevoerd. Als je een subcommando voor directe hardware aanroept op een geleverde build, wordt het programma afgesloten met een expliciete foutmelding waarin het ontbrekende pakket wordt genoemd en je wordt teruggeleid naar `pool-*`; niets mislukt in stilte. (`discover` / `list` vormen de uitzondering — deze worden omgeleid naar `pool-discover` en werken gewoon.)
>
> **Alles wat een klant nodig heeft, is bereikbaar via `pool-*`** — verbinding maken, stream, gekalibreerde `.daq`-bestanden opnemen en cap-profielen wisselen. De DAQ kan ook worden aangestuurd vanaf Python met `chloros_sdk.connect_daq_sensor()`, dat hetzelfde gepoolde pad gebruikt.

### Werkstroom voor de eerste verbinding met de DAQ-sensor

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-referentie

| Subcommando | Doel |
| --- | --- |
| `daq pool-connect` (smart-detect) | Een sensor openen in de backend-pool. |
| `daq pool-connect --port PORT` | DAQ-U op een specifieke seriële poort. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC automatisch gescand. |
| `daq pool-connect --mac MAC` | DAQ-M via BLE op een bekend MAC-adres (impliceert `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet op een bekende host. |
| `daq pool-connect --eth` | DAQ-E via Ethernet, host automatisch gedetecteerd (mDNS + ARP-fallback; werkt vanuit een lege ARP-cache op Windows en Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Integratievenster / AE-status afstemmen. |
| `daq pool-connect --no-stream` | Verbinding maken maar nog niet beginnen met streamen (hervatten met `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-correctieprofiel. Standaard is dit in de backend `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Scan elk transport op sensoren waarmee verbinding kan worden gemaakt, zonder verbinding te maken. **Zo vind je het BLE-MAC-adres van een DAQ-M.** `daq discover` / `daq list` worden in de geleverde builds automatisch hierheen gerouteerd. Sensoren die al in de pool openstaan, worden niet weergegeven — een verbonden DAQ-M stopt met adverteren — dus gebruik daarvoor `pool-list`. |
| `daq pool-list` | Toon elke sensor in de backend-pool. |
| `daq pool-disconnect --sensor-id ID [--all]` | Vrijgeven. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Meest recente N spectrumframes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Hervatten / streaming pauzeren. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Een .daq-opname starten / stoppen. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Het cap-correctieprofiel tijdens runtime wisselen. |

### Directe hardware-subcommando’s (alleen bij het uitchecken van de broncode — niet in de geleverde builds)

> Vermeld ter volledigheid. Hiervoor is het `daq`-Python-pakket vereist, plus `pyserial` / `bleak` / `zeroconf`, die geen van alle meegeleverd worden in de gecompileerde CLI of de PyPI SDK — ze werken alleen vanuit een MAPIR source checkout. **Als u een vrijgegeven Chloros-build gebruikt, gebruik dan in plaats daarvan de bovenstaande `pool-*`-commando’s**; deze omvatten connect, stream, record en cap-selectie.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Open, maak verbinding met en bestuur een opgeslagen Chloros-project (een map met `cameras.json` + `sensors.json` + `project.json`). Alles verloopt via de backend, zodat de GUI en CLI dezelfde hardwarestatus weergeven.

### Overzicht van subcommando’s

| Subcommando | Doel |
| --- | --- |
| `project open PATH` | Het apparaatmanifest van het project weergeven (camera’s, arrays, sensoren). |
| `project devices PATH [--reconnect]` | Vermeld of herhaal de detectie. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Maak verbinding met elke opgeslagen camera / array / sensor. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Eén opname maken van een opgegeven camera of array. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Een reeks van N beelden maken van een opgegeven camera of array (`-n/--count` standaard 5; `-i/--interval` seconden tussen frames, standaard 0). Reeksen verwijderen dubbele, herhaalde gesynchroniseerde groepen (verouderingscontrole), zodat een reeks met gedeeltelijke cycli niet N kopieën van één frame kan teruggeven; geeft resultaten per iteratie weer. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Streamnaar schijf via een backend-taak. `--poll-interval` = aantal seconden tussen `/stats`-polls (standaard 2,0). |
| `project sensor read PATH NAME [--json]` | Laatste spectrumframe. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | .daq-bestand opnemen. |
| `project run PATH RECIPE.yaml` | Een YAML/JSON-opname-recept uitvoeren. `--dry-run` valideert zonder uit te voeren. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Bereken uitlijning voor een array — zie [de vlaggentabel hieronder](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Huidig uitlijningsprofiel weergeven. |
| `project align clear PATH NAME` | Het profiel uit de cache verwijderen. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | De transformatie van één slave verschuiven. |
| `project align export PATH NAME --to FILE` | Het profiel opslaan in JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Een opgeslagen profiel laden. |

#### `project align calibrate`-opties

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Uitlijningsmethode. **Deze spellingen verschillen van `lattice align-calibrate`**, die de verkorte vormen `orb` / `akaze` / `phase` gebruikt; de twee opdrachten zijn bij deze vlag niet onderling uitwisselbaar. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformeer model aanpassen. |
| `--frames N` | `1` | Gesynchroniseerde framesnapshots naar gemiddelde. |
| `--reference SN` | de master | Referentiecamera-serienummer; elk ander lid wordtdaaraan gekoppeld. |
| `--max-features N` | `5000` | Maximumaantal ORB-kenmerken. |
| `--ratio-threshold F` | `0.75` | Lowe&#x27;s-verhoudingstest. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-drempelwaarde voor inliers. |
| `--min-matches N` | `15` | **Kwaliteitscontrole** — de oplossing wordt afgewezen als er minder dan dit aantal inlier-overeenkomsten is. |
| `--max-reproj-err-px F` | `4.0` | **Kwaliteitsdrempel** — de oplossing wordt afgewezen als de RMS-fout bij herprojectie hoger is dan deze waarde. |
| `--checkerboard RxC` | — | Bordgeometrie voor `--method checkerboard`, bijv. `9x6`. |
| `--name PROFILE` | leeg | Profielnaam ingebed in de opgeslagen JSON. **Niet de arraynaam** — dat is de positionele `NAME`. |

De twee kwaliteitscontroles zijn de reden waarom een kalibratie weliswaar succesvol kan zijn bij het oplossen, maar toch
het opslaan kan weigeren: een profiel dat aan een van beide niet voldoet, zou stilzwijgend elke
latere opname verkeerd registreren, dus wordt het geweigerd in plaats van opgeslagen.

### Voorbeelden

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Recept-DSL

`project run RECIPE.yaml` accepteert een YAML- of JSON-bestand waarin een reeks acties wordt beschreven:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Ondersteunde acties: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. De actie `burst` vereist `name` (verplicht), `count` (standaard 5), `interval` (seconden, standaard 0), `output`, `format`, en `settings` (dezelfde vorm van instellingen per camera als `apply`); array-bursts gebruiken dezelfde ‘fresh-synced-group’-watchdog als `project burst`.

Voer het uit:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Omgevingsvariabelen

| Variabele | Effect |
| --- | --- |
| `CHLOROS_BACKEND_URL` | De backend-URL overschrijven (standaard `http://127.0.0.1:5000`) — **wordt alleen ondersteund door de commandofamilies `lattice`, `project` en `daq pool-*`.** De kernopdrachten (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) pin `http://127.0.0.1:<port>` en negeer deze variabele (de IPv4-letterwaarde omzeilt de Windows `localhost`→`::1` ~2 s-per-verzoek-penaliteit), zodat ze altijd de lokale machine als doel hebben. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` verlaagt de capaciteit van de array bij-weigering van een verbinding bij een abonnement (totale vraag per camera &gt; botsingsveilige bandbreedtelimiet met `pin_resolution`) naar een luide waarschuwing en doorgaan, waarbij GVSP-pakketverlies wordt geaccepteerd. Alleen voor gebruik op de testbank — zie [Array fps &amp; burst-model](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Ingesteld door de CLI zelf; geeft de backend de opdracht om parallelle verwerking in te schakelen. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` slaat de GVSP-fallback-probe over (alleen ICMP-resultaten). **Dit schakelt jumbo uit, het zorgt er niet alleen voor dat het logboek stiller wordt** — de camera beantwoordt DF-pings slechts tot 1500 op elk pad, dus deze probe is het enige dat jumbo kan detecteren. Bespaart ~1 s per camera per verbinding; kost ~1,45× de maximale bandbreedte als het netwerk *wel* jumbo-pakketten had kunnen verwerken. De SDK geeft een waarschuwing wanneer je dit instelt. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Leg de GVSP-pakketgrootte vast op N bytes; slaat het peilen volledig over. Geef de voorkeur aan instelling per commando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) boven een permanente instelling: een vaste grootte past zich niet meer aan het netwerk ervoor aan, en het vastzetten van 9000 op een pad dat geen jumbo-pakketten kan verwerken, zorgt ervoor dat **elke** capture-time-out met `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | De Nuitka-map voor onefile-extractie overschrijven. De CLI gebruikt automatisch `/mnt/ssd/tmp` indien aanwezig. |

---

## Uitgangscodes

| Code | Betekenis |
| --- | --- |
| `0` | Geslaagd. |
| `1` | Algemene fout (de meeste fouten bij subcommando&#x27;s). |
| `2` | Fout in argument. |
| `130` | Onderbroken door Ctrl+C. |

---

## Tips voor het oplossen van problemen

- **&quot;Inloggen vereist&quot;** → Voer `chloros-cli login EMAIL PASSWORD` één keer uit op deze machine.
- **&quot;backend onbereikbaar&quot;** → Start de desktop-app Chloros, of voer het backend-programma rechtstreeks uit (`chloros-backend`), of controleer `CHLOROS_BACKEND_URL` als het om een externe verbinding gaat.
- **`lattice`-opdrachten mislukken met de melding &quot;LATTICE-camerastuurprogramma&#x27;s niet gevonden&quot;** → De Arena-SDK-runtime is niet geïnstalleerd; de CLI wordt geleverd met `win32api` gebundeld op Windows, maar de C-runtime maakt deel uit van het GUI-installatieprogramma.
- **Array connect / Array-instellingen geven &quot;FRAMES WILL DROP&quot; of &quot;Reduce ROI to enable&quot; weer** → De ontvangstring van de host-NIC is te klein (wordt vaak teruggezet naar 32 na een update van het NIC-stuurprogramma). Zie [Host-NIC-configuratie en -afstemming](#host-nic-setup--tuning-lattice-arrays) — stel `ReceiveBufferLen=256` en `PendingReceives=64` in.
- **Machine loopt vast bij herstart/uitschakelen, vervolgens WMI `Invalid class` / NIC kan niet worden ingeschakeld / USB-drives ontbreken** → Verouderd stuurprogramma voor de USB 10GbE-adapter veroorzaakt `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Werk het stuurprogramma van de adapter bij — zie [Host NIC-configuratie en -afstemming](#host-nic-setup--tuning-lattice-arrays).
- **Jetson-swapwaarschuwing** → Voeg een op bestanden gebaseerde swap toe; de CLI geeft de exacte `fallocate` / `swapon`-opdrachten weer.
- **DAQ-directe commando’s ontbreken** → Verwacht: de meegeleverde `chloros-cli` sluit het `daq`-pakket opzettelijk uit, dus alleen `pool-*` is aanwezig (de PyPI-SDK bevat het ook niet). Gebruik `pool-*`, dat dezelfde sensor via de backend aanstuurt, of `chloros_sdk.connect_daq_sensor()` van Python.

---

## Zie ook

- [Python SDK-referentie](sdk-reference.md) — programmatisch equivalent van elke CLI-opdracht.
- [DAQ-sensorgids](../daq/README.md) — sensorspecifieke bedrading + kalibratie.
- Online documentatie: `https://mapir.gitbook.io/chloros/cli`
