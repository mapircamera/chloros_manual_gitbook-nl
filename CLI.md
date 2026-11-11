# CLI: Opdrachtregel

> **Volledige handleiding:**[CLI-handleiding](reference/cli-reference.md) beschrijft**elke optie van elk subcommando** en is geoptimaliseerd voor AI-assistenten — plak de URL in je assistent en vraag om een werkend commando: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tip voor AI-tools:** elke pagina van deze handleiding is beschikbaar als onbewerkte Markdown door `.md` toe te voegen aan de URL (bijv. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), en `https://mapir.gitbook.io/chloros/llms.txt` indexeert de volledige handleiding voor gebruik door LLM’s.

> **Wil je dat je assistent Chloros zelf uitvoert?** `chloros-mcp`, een MCP-server die samen met de Python-SDK is geïnstalleerd, werkt met Claude Desktop, Claude Code, Cursor en andere MCP-clients: zie [Chloros gebruiken met AI-assistenten](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Wat de CLI is

`chloros-cli` is de opdrachtregel-frontend voor dezelfde verwerkingsengine die de Chloros-desktopapp gebruikt. Het is een ‘thin’ HTTP-client die verbinding maakt met de Chloros-backend (een lokale server op `127.0.0.1:5000`) — de meeste commando’s starten de backend automatisch, dus één enkele `chloros-cli process …`-aanroep is alles wat een script nodig heeft. De backend blijft vervolgens draaien voor de volgende opdracht: de backend stopt bij `chloros-cli logout` (tenzij de app geopend is of er een uitvoering of build bezig is), bij `chloros-cli backend stop`, of vanzelf na 30 minuten zonder client, uitvoering of aangesloten apparaat. `chloros-cli backend status` toont dit aan.

Het draait op **Windows 10/11 (x64)**en**Linux (x86_64, en NVIDIA Jetson arm64 op JetPack 6)**, in elke terminal, zonder dat er een GUI nodig is. Controleer je installatie met:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

De opdrachtgroepen, in één oogopslag:

* **Verwerking & account** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 talen — zie [Ondersteunde talen](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (alleen Linux/Jetson)
* **Live hardware** — `lattice` (LATTICE-camerabesturing, meer dan 45 subcommando’s), `daq pool-*` (DAQ-lichtsensoren), `time-sync` (PTP)
* **Automatisering** — `project` (een opgeslagen Chloros-project zonder grafische interface uitvoeren, inclusief YAML-opname-recepten)

Algemene opties die de moeite waard zijn om te kennen: `--port N` (backend-poort, standaard `5000`), `-v/--verbose`, `--restart` (de backend geforceerd opnieuw opstarten), `--backend-exe PATH`. Zie de [CLI-referentie](reference/cli-reference.md) voor de volledige lijst.

***

## Installatie

De CLI **wordt meegeleverd in het Chloros-installatieprogramma** op elk platform — er is geen aparte CLI-download. Download het installatieprogramma via de [Download](download.md)-pagina.

### Windows


Het installatieprogramma plaatst de CLI op:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

en voegt die map eenmalig toe aan je systeem `PATH` (bij een upgrade worden ook dubbele vermeldingen verwijderd die door eerdere installatieprogramma’s zijn achtergelaten) — **open een nieuwe terminal**na de installatie, zodat de bijgewerkte `PATH` wordt herkend. Het installatieprogramma plaatst ook opstart-scripts (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) in de installatiemap, plus een**Chloros CLI** snelkoppeling in het Startmenu, die elk een terminal openen met `chloros-cli`, klaar voor gebruik.

### Linux


Installeer `.deb` voor uw architectuur met `apt` (niet `dpkg -i`, dat de aanbevolen vereisten van de Python SDK overslaat):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Hiermee wordt `chloros-cli` tot en met `/usr/bin/chloros-cli` (al aanwezig in `PATH`), de backend en de CLI-programma’s die zijn uitgepakt in `/usr/lib/chloros/backend/` en `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` is een launcher die de backend start), de Arena SDK-runtime die de backend nodig heeft voor LATTICE-camera’s, en de Python-SDK voor het systeem `python3`. Zie [Linux-installatie](linux/linux-installation.md) voor meer informatie.

### Controleren

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Inloggen en licenties

Toegang tot de CLI (en de Python SDK) vereist een **betaald Chloros+-abonnement**— elk betaald abonnement biedt deze toegang; het gratis abonnement niet. De beperking wordt**server-side** afgedwongen door de backend, niet door het CLI-programma: een aanroep terwijl je bent uitgelogd wordt geweigerd met `401 AUTH_REQUIRED`, en een aanroep vanuit het gratis abonnement met `403 PLAN_UPGRADE_REQUIRED`, ongeacht of deze afkomstig is van `chloros-cli`, de SDK of een zelfgeschreven HTTP-client. Upgrade via [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Log **één keer per machine** in:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Wachtwoorden met speciale tekens**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` wordt door de shell vervormd (de CLI detecteert dit bij een 401 en probeert het automatisch opnieuw, maar met enkele aanhalingstekens wordt het probleem volledig vermeden).
{% endhint %}

De sessie wordt opgeslagen in de cache in `~/.chloros/user_session.json` en blijft offline werken gedurende de respijtperiode van het abonnement (30 dagen voor maandabonnementen, tot de vervaldatum voor jaarabonnementen). `chloros-cli status` werkt zelfs zonder betaald abonnement, dus de reden voor een weigering is altijd zichtbaar.

{% hint style="danger" %}
**Headless-taken inplannen? Log eerst in.**Een commando dat een backend start (`process`, `status`, `export-status`, …) die wordt uitgevoerd**zonder gecachete sessie**, mislukt niet direct — het komt terecht in een interactieve `Email:` / `Password:`-prompt op stdin. Een onbemande cron-taak of CI-stap zal daarom**vastlopen in afwachting van invoer**. Voer `chloros-cli login EMAIL 'PASSWORD'` één keer uit op de machine voordat je iets inplant.
{% endhint %}

***

## Je eerste verwerkingsrun

Wijs `process` naar een map met opnames — het detecteert automatisch Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` of een combinatie daarvan:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

De voortgang wordt live weergegeven per pijplijn-thread (Detecteren, Analyseren, Verwerken, Exporteren), en een succesvolle run wordt afgesloten met een rapportage van het aantal afbeeldingsproducten dat is geschreven: `Image products written: N`, gevolgd door `(plus M clamp masks)` wanneer tijdens de run ook [clamp-maskers](output-image-formats.md#clamp-masks) zijn opgeslagen (deze worden niet als producten geteld).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Als er al een Chloros-backend actief is (bijvoorbeeld die van de desktop-app), maakt `process` daar gebruik van en opent het project daarin. Het weigert zolang de app openstaat op een project dat door de app is geopend: voer het proces uit vanuit de app, of sluit dat project. Als een vorige run nog bezig is, wacht het maximaal 15 seconden voordat het de aanvraag weigert. `--restart` dwingt een nieuwe backend af. Op Linux wordt een backend die als systemd-service draait nooit opnieuw opgestart: `process` gebruikt deze zoals hij is, en waar het anders de backend opnieuw zou opstarten (`--restart`, of een import die niets heeft gevonden), geeft het `sudo systemctl restart <unit>` weer en sluit het af met `1`.

### Waar de uitvoer terechtkomt

`process` schrijft naar een **projectmap**, niet naar je invoermap:

* Zonder `-o`: het project wordt aangemaakt in je standaardprojectmap (gedeeld met de GUI; beheer deze met `get-project-folder` / `set-project-folder`; als er geen is ingesteld, `Documents/MAPIR/Chloros Projects` in je gebruikersmap), met de naam `-n/--project-name` of een tijdstempel (`YYYYMMDD_HHMMSS`) indien weggelaten.
* Met `-o PATH`: die map **is** de projectmap. Als deze al een `project.json` bevat, wordt er een `_1`/`_2`… met dit achtervoegsel aangemaakt in plaats van het bestand te overschrijven. Een relatief `PATH` wordt bepaald ten opzichte van de map van waaruit je `chloros-cli` uitvoert.

{% hint style="info" %}
**Relatieve paden (gewijzigd in 1.3.0).** Elk pad dat de backend opent — `-o`/`--output` bij elk commando (inclusief het standaard `output` van opnamecommando’s), `--input`, `--burst-dir`, `--profile`, `--save` en `--mask` — wordt omgezet naar de map van waaruit je `chloros-cli` uitvoert, dus `-o captures/` schrijft naar `<current folder>/captures/`. In versie 1.2.0 loste de backend deze op in zijn eigen installatiemap, wat mislukte met de foutmelding „Toegang geweigerd“. Bij een backend op afstand (`CHLOROS_BACKEND_URL`) worden paden verzonden zoals ze zijn ingevoerd en opgelost op die machine.
{% endhint %}

{% hint style="warning" %}
**Windows: houd de projectmap kort, tenzij lange paden zijn ingeschakeld.** Als lange paden in Windows zijn uitgeschakeld (de standaardinstelling van Windows), mag geen enkel bestandspad langer zijn dan 260 tekens, en staan producten tot ongeveer 130 tekens onder de projectmap. `process` controleert dit voordat het start: een projectmap die te diep is, wordt geweigerd met **"De projectmap is te diep voor deze computer"**, de langste projectmap die past, en hoe je lange paden kunt inschakelen — er wordt niets verwerkt. Gebruik een kortere `-o` (of `set-project-folder`), of schakel lange paden in Windows in (de optie **Enable Windows long paths** van het installatieprogramma, standaard aangevinkt) en start Chloros opnieuw op.
{% endhint %}

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

De cameramap is `LATT-<sensor>-<lens>-F<filter>` voor LATTICE (overeenkomend met de EXIF-gegevens van de opname `Model`) en `<model>_<filter>` (bijv. `Survey3N_RGN`) voor Survey3. De map voor bestandsformaten volgt op `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` of `tiff32` voor `TIFF (32-bit, Percent)`.

Bij een multispectrale M3C-camera kan naast de productmappen ook een map met de naam `clamp_masks/` in een formaatmap voorkomen (bijvoorbeeld `tiff16/clamp_masks/Reflectance_Calibrated_Images/` en `tiff32/clamp_masks/Radiance_Images/`). Deze map bevat de per-pixelmaskers van stralings- en reflectieproducten waarin de bandunmix een negatieve waarde op nul heeft vastgezet — zie [Clamp-maskers](output-image-formats.md#clamp-masks). Een productmap bevat nooit een masker.

{% hint style="info" %}
**Elk geëxporteerd product behoudt de naam van het BRON-bestand.**Een stralings-export van `capture_..._raw.tif` heet nog steeds `capture_..._raw.tif` — het staat alleen in `tiff32/Radiance_Images/`.**De map identificeert het product, niet de bestandsnaam**, dus gebruik een glob-patroon voor de map, niet voor het achtervoegsel `*radiance*`.
{% endhint %}

### De opties die je daadwerkelijk zult gebruiken

| Vlag | Standaard | Wat het doet |
| --- | --- | --- |
| `-o, --output PATH` | standaard projectmap | Locatie van de projectmap (zie hierboven). |
| `-n, --project-name NAME` | tijdstempel | Projectnaam. |
| `--format FMT` | `TIFF (16-bit)` | Een van `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros voert geen opschaling uit: bij volledig 8-bits afbeeldingen wordt een dieper formaat opgeslagen als `TIFF (8-bit)`, en dit wordt één keer vermeld. |
| `--indices NAME [NAME ...]` | geen | Te exporteren vegetatie-indexen (zie [Vegetatie-indexen](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neurale debayer, langzamer, hoogste kwaliteit (Chloros+, vereist een CUDA-GPU: zonder deze wordt `standard` gebruikt; niet toegestaan voor LATTICE-opnames, die altijd `standard` krijgen). |
| `--vignette / --no-vignette` | aan | Vignettecorrectie. |
| `--reflectance / --no-reflectance` | aan | Reflectantiekalibratie; voor LATTICE is dit ook de schakelaar voor het reflectantieproduct. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Het startpunt van de verwerkingspijplijn voor LATTICE-TIFF’s afdwingen. |

Voor al het overige — afstemming van doeldetectie, PPK, belichtingspinnen, vlaggen voor array-uitlijning — zie de [`process`-sectie van de CLI-referentie](reference/cli-reference.md).

***

## Kiezen wat er moet worden geëxporteerd (LATTICE-producten)

De LATTICE-verwerking wordt **in één doorloop**over**elk toepasselijk product**verdeeld. Vier schakelaars per product staan**standaard allemaal op AAN**; gebruik het formulier `--no-` om er één uit te schakelen:

| Schakelaar | Product |
| --- | --- |
| `--debayered` | Lineaire demosaïc → `Debayered_Images/`: de donkere offset van het kalibratiepakket verwijderd, gevolgd door een randbewuste demosaic, zonder lineariteitscorrectie, flat field of unmix — hetzelfde bestand dat een opname opslaat op het `debayered`-niveau. Behoudt de oorspronkelijke oriëntatie van de sensor (opgeslagen in XMP). |
| `--preview` | Voorbeeldweergave (witbalans + gamma; valsekleuruitrekking voor multispectraal) → `Preview_Images/` |
| `--radiance` | float32 straling, W/m²/sr/nm → `Radiance_Images/` (altijd `tiff32/`) |
| `--reflectance` | uint16 reflectie, Pix4D-ready → `Reflectance_Calibrated_Images/` |

RGB-mastercamera’s geven uitsluitend debayered + preview weer — straling/reflectantie per band is niet relevant voor een breedbandsensor, dus die schakelaars hebben voor hen geen effect. Survey3 `.raw` negeert de schakelaars en volgt het standaardpad voor reflectantie/doel.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (standaard `auto`) kiest de reflectantiereferentie: `auto` maakt een QA-goedgekeurd [kalibratiedoel](calibration-targets.md) binnen het beeld de absolute referentie en valt terug op de DAQ-lichtsensor-downwelling-verdeling (ρ = π·L/E) wanneer er geen doel aanwezig is (een doel dat de kwaliteitscontrole doorstaat met een vlag, concurreert met de DAQ op basis van de gemeten score); `target` is strikt (geen DAQ-vervanging); `daq` is DAQ-autoritatief. Per eenheid gemeten doel-scans kunnen worden aangeleverd met `--target-reflectance-dir`.

{% hint style="info" %}
**Reflectiepixels uitlezen:**de DN die ρ = 1,0 betekent, is**per bron** — LATTICE-bestanden stempelen `Chloros:PixelScale=32768` in XMP; Survey3-bestanden gebruiken 65535 en bevatten geen `Chloros:PixelScale`-tag (ze bevatten wel andere `Chloros:*`-verwerkingstags). Lees de tag en deel erdoor in plaats van uit te gaan van een constante. Details en het ene opzettelijke randgeval zonder schaal staan in de [CLI-referentie](reference/cli-reference.md).
{% endhint %}

**De verwerking begint altijd bij `raw`.** Afgeleide producten (debayered/radiance/reflectance-exports) worden nooit teruggevoerd door de pijplijn — het opnieuw importeren en verwerken ervan zou de kalibratieberekeningen dubbel toepassen, dus Chloros slaat ze over en geeft dit aan. `--input-level` is de bewuste nooduitgang voor het geval je echt een startpunt moet forceren.

***

## Wanneer een run mislukt

Vanaf versie 1.2.0 geeft `process` een duidelijke foutmelding in plaats van „succesvol“ te zijn zonder dat er iets te zien is:

* Een run die **producten heeft aangevraagd maar er geen heeft geschreven**— alleen `project.json` en `calibration_data.json` — geeft `Processing finished but wrote no image products.` weer en**stopt met een waarde anders dan nul**, zodat scripts dit kunnen detecteren. De gebruikelijke oorzaken: de invoermap werd niet herkend als een opname (controleer de indeling en `--input-level`), of elk aangevraagd product was niet van toepassing op die camera’s (bijv. het vragen om radiance/reflectance van camera’s die alleen RGB ondersteunen).
* Een **opzettelijke run met alleen metadata** (alle producten uitgeschakeld, geen `--indices`) is nog steeds geslaagd — een lege afbeeldingsuitvoer is in dit geval het juiste resultaat.
* Voer de run opnieuw uit met `--verbose` en controleer het backend-logboek op regels met `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, die uitleg geven over het overslaan van afzonderlijke camera’s.
* Op Windows, waar lange paden zijn uitgeschakeld, wordt een projectmap die te diep is voor de limiet van 260 tekens **voordat** de verwerking begint geweigerd, waarbij de langste projectmap die wel past wordt gebruikt (zie [Waar de uitvoer terechtkomt](#where-the-outputs-land)).
* Opnamecommando’s (`lattice capture`, `lattice array-capture` en de andere opnamefamilies) noemen elk bestand dat ze niet konden schrijven en beëindigen met `1`; herhaalde opnames stoppen wanneer de schijf vol is.

Uitvoercodes: `0` succes · `1` algemene fout · `2` argumentfout · `130` onderbroken door Ctrl+C.

***

## Vegetatie-indexen

Voer `--indices` uit met een of meer vooraf ingestelde namen; elke index komt in zijn eigen `<INDEX>_Index_Images/`-map terecht:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

De 22 vooraf ingestelde namen die `process --indices` accepteert:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Een naam die nog niet in het project voorkomt, wordt eraan toegevoegd. Een naam die geen voorinstelling is, of die de banden van de camera's niet kunnen weergeven, wordt overgeslagen met een waarschuwing waarin de naam wordt vermeld. Als de CLI-interface de huidige indexlijst van het project niet kan lezen, laat deze de lijst ongewijzigd en geeft een waarschuwing in plaats van deze te vervangen.

{% hint style="warning" %}
**Er zijn drie indexlijsten — haal ze niet door elkaar.**De vervolgkeuzelijst „Projectinstellingen“ van de GUI bevat 27 formules (voegt `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — deze vijf zijn uitsluitend voor de GUI en zijn**niet** geldig voor `--indices`). Het live/offline-commando `lattice index --preset` maakt gebruik van een eigen, afzonderlijke lijst met 22 voorinstellingen. Formules en bandberekeningen zijn gedocumenteerd in [Multispectrale indexformules](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-lichtsensoren: een korte kennismaking

De `daq pool-*`-familie stuurt MAPIR DAQ-spectrale sensoren (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) aan via de permanente pool van de backend — de GUI, CLI en SDK delen allemaal één live-handle. **`pool-*` is het ondersteunde DAQ-pad in de meegeleverde CLI**; andere `daq`-subcommando’s waarnaar mogelijk wordt verwezen, zijn een uitsluitend voor MAPIR-interne bron bestemde interface en worden afgesloten met een expliciete foutmelding die u naar `pool-*` verwijst.

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

`pool-record` zonder `--duration` loopt door tot `pool-record --stop`; de standaarduitvoermap is `~/Documents/DAQ Live View/` **op de machine van de backend**. Het cap-correctieprofiel wordt gekozen op het moment van verbinding (`--cap-id`, standaard voor de backend `sunshine_cosine`) en kan live worden gewisseld met `pool-set-cap` — cap-profielen en het gekalibreerde bereik van de sensor worden behandeld in de DAQ-hoofdstukken van deze handleiding.

{% hint style="warning" %}
**DAQ-E op een host met meerdere netwerkkaarten:** de eerste automatische detectie van `pool-connect --eth` na het opstarten kan mislukken, zelfs als de sensor in orde is. `--eth-host <ip-or-hostname>` is de betrouwbare variant — gebruik deze wanneer de detectie geen resultaten oplevert.
{% endhint %}

***

## LATTICE-camera’s, PTP en projectautomatisering

De `lattice`-familie (meer dan 45 subcommando’s) dekt het volledige werk met LATTICE-camera’s: detectie, afzonderlijke opnames, permanente gesynchroniseerde arrays met de ‘smart-prep connect’-workflow van de GUI, live-voorbeeld in de browser, uitlijning, indexberekeningen en host-NIC-diagnostiek. Een voorproefje:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Daarnaast: `chloros-cli time-sync` rapporteert over de PTP-grandmaster die de Chloros-host draait (LATTICE-camera’s en DAQ-E-sensoren werken hier als slaves mee voor apparaatoverschrijdende tijdstempels), en `chloros-cli project` opent een opgeslagen Chloros-project en stuurt de bijbehorende camera’s, arrays en sensoren aan zonder beeldscherm — inclusief gescripte YAML-opnameprocedures.

Deze drie families (`lattice`, `project`, `daq pool-*`) zijn ook de enige die `CHLOROS_BACKEND_URL` ondersteunen voor het aansturen van een **externe** backend; de kerncommando’s zijn altijd gericht op de lokale machine.

Volledige stapsgewijze handleidingen zijn te vinden in de LATTICE-hoofdstukken van deze handleiding; elke vlag staat in de [CLI-referentie](reference/cli-reference.md).

***

## Probleemoplossing: Top 5

| Symptoom | Oplossing |
| --- | --- |
| `Login required`, of een geplande taak loopt vast bij een `Email:`-prompt | Voer `chloros-cli login EMAIL 'PASSWORD'` één keer uit op deze machine — commando’s zonder een in de cache opgeslagen sessie worden dan interactief uitgevoerd in plaats van direct te mislukken. |
| `backend unreachable` | Start de Chloros-desktopapp, of start de backend zelf: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` op Windows (de map `backend` in je installatiemap; deze staat niet op `PATH`), `chloros-backend` op Linux. `chloros-cli backend status` geeft aan of er een actief is; `lattice`/`project`/`daq pool-*` starten er nooit een. Als u ze naar een backend op afstand laat verwijzen, controleer dan `CHLOROS_BACKEND_URL`. |
| Array-verbinding geblokkeerd: `FRAMES WILL DROP` / `Reduce ROI to enable` | Ontvangstring van de host-NIC teruggezet naar standaardinstellingen — de belangrijkste oorzaak waarom een eerder werkende installatie geen verbinding meer wil maken, meestal na een update van het NIC-stuurprogramma. Voer `chloros-cli lattice network --fix` uit vanaf een **terminal met verhoogde rechten** (of stel `ReceiveBufferLen=256`, `PendingReceives=64` in); zie *Host NIC Setup & Tuning* in de referentie. |
| Het `daq`-subcommando wordt afgesloten: „vereist het volledige DAQ-pakket…“ | Gebruik de `daq pool-*`-opdrachten, die dezelfde sensor via de backend aansturen: verbinden, streamen, opnemen, cap-selectie, en voor een DAQ-E de netwerkinstellingen, een herstart en firmware-updates (of `chloros_sdk.connect_daq_sensor()` vanuit Python). |
| Jetson geeft een swap-waarschuwing weer bij grote mappen | Voeg file-backed swap toe — de CLI geeft de exacte `fallocate`/`swapon`-opdrachten weer die moeten worden uitgevoerd. |

***

## Hulp krijgen

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Elke vlag, elk subcommando:** [CLI-referentie](reference/cli-reference.md)
* **Python-equivalent:** [Python SDK](api-python-sdk.md) en de [SDK Reference](reference/sdk-reference.md)
* **Ondersteuning:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
