# CLI : Opdrachtregel

> **Volledige referentie:**[CLI Reference](reference/cli-reference.md) documenteert**elke vlag van elke subopdracht** en is geoptimaliseerd voor AI-assistenten — plak de URL in uw assistent en vraag om een werkende opdracht: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tip voor AI-tools:** elke pagina van deze handleiding is beschikbaar als onbewerkte Markdown door `.md` toe te voegen aan de URL (bijv. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), en `https://mapir.gitbook.io/chloros/llms.txt` indexeert de volledige handleiding voor gebruik door LLM’s.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Wat deCLI
is

`chloros-cli` is de command-line front-end voor dezelfde verwerkingsengine die de desktop-appChloros
gebruikt. Het is een thin-HTTP
-client bovenop deChloros
-backend (een lokale server op `127.0.0.1:5000`) — de meeste commando’s starten de backend automatisch, dus een enkel `chloros-cli process …`-oproep is alles wat een script nodig heeft.

Het draait op **Windows
10/11 (x64)**en**Linux
(x86_64, en NVIDIA Jetson arm64 op JetPack 6)**, in elke terminal, zonder dat een GUI nodig is. Controleer je installatie met:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

De commandofamilies in één oogopslag:

* **Verwerking &amp; account** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 talen — zie [Ondersteunde talen](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (alleenLinux
/Jetson)
* **Live hardware** — `lattice` (LATTICE-camerabesturing, meer dan 45 subcommando’s), `daq pool-*` (DAQ-lichtsensoren), `time-sync` (PTP)
* **Automatisering** — `project` (een opgeslagenChloros
-project zonder beeldscherm uitvoeren, inclusief YAML-opnamevoorschriften)

Algemene opties die de moeite waard zijn om te kennen: `--port N` (backend-poort, standaard `5000`), `-v/--verbose`, `--restart` (de backend geforceerd opnieuw opstarten), `--backend-exe PATH`. Zie de [CLI
-referentie](reference/cli-reference.md) voor de volledige lijst.

***

## Installatie

CLI
**wordt meegeleverd in hetChloros
-installatieprogramma** op elk platform — er is geen aparteCLI
-download. Download het installatieprogramma via de [Download](download.md)-pagina.

###Windows


Het installatieprogramma plaatst deCLI
in:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

en voegt die map toe aan je systeem `PATH` — **open een nieuwe terminal**na de installatie, zodat de bijgewerkte `PATH` wordt opgepikt. Het installatieprogramma plaatst ook opstart-scripts (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) in de installatiemap, plus een**Chloros
CLI
** snelkoppeling in het Startmenu, die elk een terminal openen met `chloros-cli` klaar voor gebruik.

###Linux


Installeer de `.deb` voor uw architectuur:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Hiermee wordt `chloros-cli` tot en met `/usr/bin/chloros-cli` (reeds op `PATH`) en de backend naar `/usr/lib/chloros/chloros-backend`, samen met de Arena-SDK
-runtime die nodig is voor LATTICE-camera’s. Zie [Linux
Installatie](linux/linux-installation.md) voor meer informatie.

### Controleren

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Inloggen en licenties

CLI
(enPython
SDK
) vereist een **betaaldChloros
+ abonnement**— elk betaald abonnement biedt deze toegang; het gratis abonnement niet. De beperking wordt**server-side** afgedwongen door de backend, niet door hetCLI
-programma: een aanroep terwijl je bent uitgelogd wordt afgewezen met `401 AUTH_REQUIRED`, en een aanvraag terwijl je bent ingelogd op het gratis abonnement met de foutcode `403 PLAN_UPGRADE_REQUIRED`, ongeacht of deze afkomstig is van `chloros-cli`, deSDK
of een zelfgeschrevenHTTP
-client. Upgrade via [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Log **één keer per machine** in:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Wachtwoorden met speciale tekens**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` wordt door de shell vervormd; deCLI
detecteert dit bij een 401-fout en probeert het automatisch opnieuw, maar met enkele aanhalingstekens wordt het probleem volledig vermeden).
{% endhint %}

De sessie wordt opgeslagen in de cache in `~/.chloros/user_session.json` en blijft offline werken gedurende de respijtperiode van het abonnement (30 dagen voor maandabonnementen, tot de vervaldatum voor jaarabonnementen). `chloros-cli status` werkt zelfs zonder betaald abonnement, dus de reden voor een weigering is altijd zichtbaar.

{% hint style="danger" %}
**Headless taken plannen? Log eerst in.**Een commando dat een backend start (`process`, `status`, `export-status`, …) die wordt uitgevoerd**zonder een in de cache opgeslagen sessie**, mislukt niet direct — het komt terecht in een interactieve `Email:` / `Password:`-prompt op stdin. Een onbemande cron-taak of CI-stap zal daarom**vastlopen in afwachting van invoer**. Voer `chloros-cli login EMAIL 'PASSWORD'` één keer uit op de machine voordat je iets inplant.
{% endhint %}

***

## Je eerste verwerkingsrun

Richt `process` op een map met opnames — het detecteert automatischSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` of een combinatie daarvan:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

De voortgang wordt per pijplijn-thread live weergegeven (Detecteren, Analyseren, Verwerken, Exporteren), en een succesvolle run wordt afgesloten met een rapportage van het aantal afbeeldingsproducten dat is opgeslagen (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Waar de uitvoer terechtkomt

`process` schrijft naar een **projectmap**, niet naar uw invoermap:

* Zonder `-o`: het project wordt aangemaakt in je standaardprojectmap (gedeeld met de GUI; beheer deze met `get-project-folder` / `set-project-folder`, fallback `~/Chloros Projects`), met de naam `-n/--project-name` of een tijdstempel (`YYYYMMDD_HHMMSS`) indien weggelaten.
* Met `-o PATH`: die map **is** de projectmap. Als deze map al een `project.json` bevat, wordt er een map met het achtervoegsel `_1`/`_2`… aangemaakt in plaats van de bestaande te overschrijven.

Binnen het project worden producten gegroepeerd **op camera en vervolgens op bestandsformaat**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

De cameramap is `LATT-<sensor>-<lens>-F<filter>` voor LATTICE (overeenkomend met de EXIF-gegevens van de opname `Model`) en `<model>_<filter>` (bijv. `Survey3N_RGN`) voorSurvey3
. De map voor het bestandsformaat volgt op `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` of `tiff32` voor `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Elk geëxporteerd product behoudt de naam van het BRON-bestand.**Een Radiance-export van `capture_..._raw.tif` heet nog steeds `capture_..._raw.tif` — het staat alleen in de map `tiff32/Radiance_Images/`.**De map identificeert het product, niet de bestandsnaam**, dus gebruik een glob-patroon voor de map, niet voor het achtervoegsel `*radiance*`.
{% endhint %}

### De opties die je daadwerkelijk zult gebruiken

| Vlag | Standaard | Wat het doet |
| --- | --- | --- |
| `-o, --output PATH` | standaard projectmap | Locatie van de projectmap (zie hierboven). |
| `-n, --project-name NAME` | tijdstempel | Projectnaam. |
| `--format FMT` | `TIFF (16-bit)` | Een van `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | geen | Te exporteren vegetatie-indexen (zie [Vegetatie-indexen](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neurale debayer, langzamer, hoogste kwaliteit (Chloros
+, NVIDIA GPU). |
| `--vignette / --no-vignette` | aan | Vignetcorrectie. |
| `--reflectance / --no-reflectance` | aan | Reflectantiecalibratie; voor LATTICE is dit ook de schakelaar voor het reflectantieproduct. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Het startpunt van de pijplijn voor LATTICE-TIFF’s afdwingen. |

Voor al het overige — afstemming van doeldetectie, PPK, belichtingspinnen, vlaggen voor array-uitlijning — zie de [sectie `process` van deCLI
-referentie](reference/cli-reference.md).

***

## Kiezen wat je wilt exporteren (LATTICE-producten)

De LATTICE-verwerking wordt **in één doorloop**over alle toepasselijke producten verdeeld. Vier schakelaars per product staan**standaard allemaal op AAN**; gebruik het formulier `--no-` om er één uit te schakelen:

| Schakelaar | Product |
| --- | --- |
| `--debayered` | Lineaire demosaïc → `Debayered_Images/` |
| `--preview` | Voorbeeld weergeven (witbalans + gamma; valsekleuruitrekking voor multispectraal) → `Preview_Images/` |
| `--radiance` | float32 straling, W/m²/sr/nm → `Radiance_Images/` (altijd `tiff32/`) |
| `--reflectance` | uint16 reflectantie, Pix4D-ready → `Reflectance_Calibrated_Images/` |

RGB
Mastercamera’s zenden uitsluitend debayered + preview uit — radiance/reflectance per band is niet relevant voor een breedbandsensor, dus die schakelaars hebben voor hen geen effect.Survey3
`.raw` negeert de schakelaars en volgt het standaardreflectance/target-pad.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (standaard `auto`) kiest de reflectantiereferentie: `auto` maakt een QA-goedgekeurd [kalibratiedoel](calibration-targets.md) binnen het beeld als absolute referentie en valt terug op de DAQ-lichtsensor-downwelling-verdeling (ρ = π·L/E) wanneer er geen doel aanwezig is; `target` is strikt (geen DAQ-vervanging); `daq` is DAQ-autoritatief. Per-eenheid gemeten doelscans kunnen worden geleverd met `--target-reflectance-dir`.

{% hint style="info" %}
**Reflectiepixels uitlezen:**de DN-waarde ρ = 1,0 is**per bron** — LATTICE-bestanden stempelen `Chloros:PixelScale=32768` in XMP;Survey3
-bestanden gebruiken 65535 (en bevatten geen `Chloros:*`-tags). Lees de tag en deel erdoor in plaats van uit te gaan van een constante. Details en het ene opzettelijke randgeval zonder schaal staan in de [CLI
-referentie](reference/cli-reference.md).
{% endhint %}

**De verwerking begint altijd bij `raw`.** Afgeleide producten (exportbestanden van debayering, straling of reflectie) worden nooit teruggevoerd door de pijplijn — het opnieuw importeren en verwerken ervan zou de kalibratieberekeningen dubbel toepassen, dusChloros
slaat ze over en geeft dit aan. `--input-level` is de bewuste nooduitgang voor het geval je echt een startpunt moet forceren.

***

## Wanneer een run mislukt

Vanaf versie 1.2.0 meldt `process` luidkeels een fout in plaats van „succesvol“ te zijn zonder iets te tonen:

* Een run die **producten heeft aangevraagd maar er geen heeft geschreven**— alleen `project.json` en `calibration_data.json` — geeft `Processing finished but wrote no image products.` weer en**stopt met een waarde anders dan nul**, zodat scripts dit kunnen detecteren. De gebruikelijke oorzaken: de invoermap werd niet herkend als een opname (controleer de indeling en `--input-level`), of elk aangevraagd product was niet van toepassing op die camera’s (bijv. het vragen om radiance/reflectance van camera’s die alleenRGB
ondersteunen).
* Een **opzettelijke run met alleen metadata** (alle producten uitgeschakeld, geen `--indices`) is nog steeds geslaagd — een lege afbeeldingsuitvoer is daar het juiste resultaat.
* Voer de run opnieuw uit met `--verbose` en controleer het backend-logboek op regels met `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, die uitleg geven over het overslaan van afzonderlijke camera&#x27;s.

Uitvoercodes: `0` succes · `1` algemene fout · `2` argumentfout · `130` onderbroken door Ctrl+C.

***

## Vegetatie-indexen

Voer `--indices` uit met een of meer vooraf ingestelde namen; elke index komt in zijn eigen `<INDEX>_Index_Images/`-map terecht:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

De 22 vooraf ingestelde namen die `process --indices` accepteert:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Er zijn drie indexlijsten — haal ze niet door elkaar.**De vervolgkeuzelijst „Projectinstellingen“ van de GUI bevat 27 formules (voegt `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — deze vijf zijn uitsluitend voor de GUI en zijn**niet** geldig voor `--indices`). Het live/offline-commando `lattice index --preset` maakt gebruik van een eigen, afzonderlijke lijst met 22 voorinstellingen. Formules en bandberekeningen zijn gedocumenteerd in [Multispectrale indexformules](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-lichtsensoren: een korte kennismaking

De `daq pool-*`-familie stuurtMAPIR
DAQ-spectrale sensoren (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) aan via de permanente pool van de backend — de GUI,CLI
enSDK
delen allemaal één live-handle. **`pool-*` is het ondersteunde DAQ-pad in de meegeleverdeCLI
**; andere `daq`-subcommando’s waarnaar mogelijk wordt verwezen, zijn eenMAPIR
-intern oppervlak dat uitsluitend als bron dient en afsluit met een expliciete foutmelding die u doorverwijst naar `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` zonder `--duration` draait tot `pool-record --stop`; de standaard uitvoermap is `~/Documents/DAQ Live View/` **op de machine van de backend**. Het cap-correctieprofiel wordt gekozen op het moment van verbinding (`--cap-id`, backend-standaard `sunshine_cosine`) en kan live worden gewisseld met `pool-set-cap` — cap-profielen en het gekalibreerde bereik van de sensor worden behandeld in de DAQ-hoofdstukken van deze handleiding.

{% hint style="warning" %}
**DAQ-E op een host met meerdere netwerkkaarten:** de eerste automatische detectie van `pool-connect --eth` na het opstarten kan mislukken, zelfs als de sensor in orde is. `--eth-host <ip-or-hostname>` is de betrouwbare variant — gebruik deze wanneer de detectie geen resultaten oplevert.
{% endhint %}

***

## LATTICE-camera’s, PTP &amp; projectautomatisering

De `lattice`-familie (meer dan 45 subcommando’s) dekt het volledige werk met LATTICE-camera’s: detectie, afzonderlijke opnames, permanente gesynchroniseerde arrays met de ‘smart-prep connect’-workflow van de GUI, live browser-preview, uitlijning, indexberekeningen en host-NIC-diagnostiek. Een voorproefje:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Daarnaast: `chloros-cli time-sync` rapporteert over de PTP-grandmaster die op deChloros
-host draait (LATTICE-camera’s en DAQ-E-sensoren werken hier als slaves mee voor apparaatoverschrijdende tijdstempels), en `chloros-cli project` opent een opgeslagenChloros
-project en stuurt de bijbehorende camera’s, arrays en sensoren headless aan — inclusief gescripte YAML-opnameprocedures.

Deze drie families (`lattice`, `project`, `daq pool-*`) zijn ook de enige die `CHLOROS_BACKEND_URL` ondersteunen voor het aansturen van een **externe** backend; de kerncommando’s zijn altijd gericht op de lokale machine.

Volledige stap-voor-stap-handleidingen zijn te vinden in de LATTICE-hoofdstukken van deze handleiding; elke vlag staat in de [CLI
-referentie](reference/cli-reference.md).

***

## Probleemoplossing: Top 5

| Symptoom | Oplossing |
| --- | --- |
| `Login required`, of een geplande taak loopt vast bij een `Email:`-prompt | Voer `chloros-cli login EMAIL 'PASSWORD'` één keer uit op deze machine — commando’s zonder een in de cache opgeslagen sessie worden dan interactief uitgevoerd in plaats van direct te mislukken. |
| `backend unreachable` | Start de desktop-appChloros
, of voer het backend-binaire bestand rechtstreeks uit (`chloros-backend`). Als u `lattice`/`project`/`daq pool-*` op een backend op afstand richt, controleer dan `CHLOROS_BACKEND_URL`. |
| Array-verbinding geblokkeerd: `FRAMES WILL DROP` / `Reduce ROI to enable` | De ontvangstring van de host-NIC is teruggezet naar de standaardinstellingen — de belangrijkste oorzaak waarom een eerder werkende rig geen verbinding meer wil maken, meestal na een update van het NIC-stuurprogramma. Voer `chloros-cli lattice network --fix` uit vanaf een **terminal met verhoogde rechten** (of stel `ReceiveBufferLen=256`, `PendingReceives=64` in); zie het hoofdstuk *Host NIC Setup &amp; Tuning* in de handleiding. |
| Het `daq`-subcommando stopt met de melding: &quot;vereist het volledige DAQ-pakket…&quot; | Te verwachten bij geleverde builds — de gecompileerdeCLI
bevat alleen de `daq pool-*`-familie, die het tot stand brengen van verbindingen, streamen, opnemen en cap-selectie omvat. Gebruik `pool-*` (of `chloros_sdk.connect_daq_sensor()` vanPython
). |
| Jetson geeft een swap-waarschuwing weer bij grote mappen | Voeg file-backed swap toe — deCLI
geeft de exacte `fallocate`/`swapon`-opdrachten weer die moeten worden uitgevoerd. |

***

## Hulp krijgen

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Elke vlag, elk subcommando:** [CLI
Referentie](reference/cli-reference.md)
* **Equivalent inPython
:** [Python
SDK
](api-python-sdk.md) en de [SDK
Referentie](reference/sdk-reference.md)
* **Ondersteuning:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
