# Chloros Python SDK Referentie

**Versie:**

1.2.0**Aangemaakt:**29 juli 2026 19:19 ·**Herzien:** 30-08-2026**Pakket:** `chloros-sdk` (PyPI)**Doelgroep:** Geoptimaliseerd voor gebruik door LLM’s; leesbaar voor mensen.**Toepassingsgebied:** Alle openbare klassen, functies en hulpprogramma’s die door `import chloros_sdk` worden aangeboden, met voorbeelden die je kunt kopiëren en plakken, onder meer op het gebied van beeldverwerking, besturing van één camera, gesynchroniseerde arrays, DAQ-sensoren en projectautomatisering.

Als je alleen de hoogtepunten nodig hebt, ga dan naar:
- [Installatie &amp; Snelstart](#installatie)
- [Smart-Connect voor LATTICE-arrays](#smart-connect-voor-lattice-camera’s)
- [DAQ-sensorsessies](#daq-sensorsessies)
- [Projectautomatisering](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Architectuur in 60 seconden

SDK is een thin-Pythonlaag bovenop de backend Chloros (dezelfde Flask-server die de desktop-GUI en CLI gebruiken). Voor automatisering importeer je `chloros_sdk` en roep je methoden op hoog niveau aan; achter de schermen wordt elke aanroep omgezet in een HTTP-verzoek naar de lokale backend op poort 5000 — `http://127.0.0.1:5000/api/...` (bewust niet `localhost`, dat eerst wordt omgezet naar `::1` op Windows en ongeveer 2 s per verzoek kost bij een backend die alleen IPv4 ondersteunt). De backend beheert de hardwarepool — camera’s, DAQ-sensoren, uitlijningsprofielen, framebuffers — zodat SDK-scripts naast de GUI kunnen draaien zonder te strijden om seriële poorten of NIC-bandbreedte.

Er zijn drie interfaces die je zult gebruiken:

1. **`ChlorosLocal` + vrije functies** (`process_folder`, `process_lattice_capture`) — Pijplijn voor beeldverwerking. Verwerk een volledige map via kalibratie / debayer / indexexport vanuit één Python-aanroep.
2. **Smart-connect-handles** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Open een permanente backend-sessie voor live hardware. Dezelfde „smart-prep“-workflow als in de GUI: netwerkprobe, automatische tier-selectie, PTP, AE-seeding, GPIO-triggerconfiguratie.
3. **`ChlorosProject` / `open_project`** — Laad een opgeslagen project (map met `cameras.json` + `sensors.json` + `project.json`), verbind alles in één keer en voer opnames uit met benoemde handles.

Oppervlakken 1 en 2 **starten automatisch een lokale backend** als er nog geen luistert (hetzelfde meegeleverde binaire bestand dat de GUI/CLI opstart) — dus een kaal script werkt vanuit een nieuwe shell zonder dat je eerst een backend hoeft te starten. Geef `auto_start_backend=False` door om dit uit te schakelen (bijvoorbeeld wanneer je naar een backend op afstand verwijst, die nooit wordt gestart). Zie [Automatisch starten van backend](#backend-auto-start). Surface 3 gedraagt zich anders: `open_project()` accepteert geen `auto_start_backend`-parameter, en `connect_all()` start nooit een backend — het probeert `http://127.0.0.1:5000` één keer en, als er niets reageert, schakelt het stilzwijgend over naar directe (backend-vrije) `lattice_sdk`-apparaatbesturing. Alleen `proj.process()` en `stream(..., overlays=True)` bouwen op een &#x27;lazy&#x27; manier een `ChlorosLocal()` op (dat auto-start).

Alle drie zijn ze beveiligd met authenticatie: voer `chloros-cli login` één keer uit op de machine, of log in via de desktop-GUI. Oproepen aan SDK zonder geldige sessie genereren een `ChlorosAuthenticationError`.

Vereisten:
- Python 3.7+ (zoals aangegeven door het pakket; ontwikkeld/getest op 3.10)
- Chloros Desktop lokaal geïnstalleerd (het backend-binaire bestand wordt meegeleverd in het installatieprogramma)
- Actieve Chloros+ login. Het minimum voor SDK / CLI is het **Copper**-niveau of hoger (Copper / Bronze / Silver / Gold); het gratis**Iron**-niveau biedt geen toegang tot SDK / CLI. Dit wordt**server-side** afgedwongen: elk verzoek met de vlaggen SDK / CLI moet zowel een actieve sessie als een betaald abonnement bevatten, anders retourneert de backend `403` met `error_code: PLAN_UPGRADE_REQUIRED` (weergegeven als `ChlorosLicenseError` door `ChlorosLocal`, en als `ChlorosConnectError` door de `connect_*`-helpers). Een uitgelogde aanroeper krijgt in plaats daarvan `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — de twee verschillen van elkaar omdat het opnieuw uitvoeren van `chloros-cli login` de eerste fout verhelpt, maar de tweede niet kan verhelpen.
- Offline gebruik wordt ondersteund binnen de respijtperiode van het abonnement: het abonnementsniveau wordt gelezen uit de cache voor servervalidatie (5 min) of de cache met gesigneerde, aan de machine gebonden licenties (30 dagen voor maandabonnementen, tot het verstrijken van het jaarabonnement). Wanneer die respijtperiode verstrijkt, wordt het abonnement omgezet naar het gratis abonnement en wordt de toegang via SDK / CLI stopgezet totdat de machine één keer verbinding kan maken met de server. `chloros-cli status` (`GET /api/license-status`) blijft bereikbaar op het gratis niveau, zodat de reden zichtbaar is — het is de enige route via SDK / CLI die is vrijgesteld van de niveaubeperking.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS of nieuwer**, of Jetson (JetPack 6). Ubuntu 20.04 wordt**niet** ondersteund: de afhankelijkheden van de `.deb` zijn afgeleid van de bibliotheken waaraan de backend is gekoppeld, waaronder `libc6 (>= 2.34)`, en Focal levert glibc 2.31 mee.

---

## Installatie

De Python SDK is een dunne Python-laag bovenop de Chloros-backend. Voor alles wat verder gaat dan een paar DAQ-only-workflows, moet je het **desktop-pakket Chloros lokaal installeren** (Windows installer of Linux `.deb`) — dat levert het backend-binaire bestand, de Arena SDK-runtime voor LATTICE-camera’s en de kalibratiebundels.

Nieuwste downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Stap 1 — Installeer het Chloros-platformpakket

#### Windows (.exe)

1. Download `Chloros-Setup-x.y.z.exe` vanaf de downloadpagina.
2. Voer het installatieprogramma uit en volg de wizard. Het standaardinstallatiepad is `C:\Program Files\MAPIR\Chloros\`.
3. Start Chloros minstens één keer en log in met je Chloros+-account.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Stap 2 — Installeer de Python SDK

**Het installatieprogramma Chloros wordt geleverd met een bijpassend SDK-wheel.** Elk installatieprogramma Windows en elk .deb-bestand Linux plaatst een `chloros_sdk-X.Y.Z-py3-none-any.whl` op de schijf dat exact overeenkomt met de GUI- / CLI / backend-versie. Je hoeft PyPI niet in de gaten te houden om synchroon te blijven.

#### Windows

Het installatieprogramma voert `pip install` automatisch uit op het meegeleverde wheel-bestand met behulp van je systeemPython (bij voorkeur de `py.exe`-launcher, maar valt terug op `python -m pip`). U hoeft niets te doen — `import chloros_sdk` werkt in uw Python-omgeving na een succesvolle installatie. Als er geen Python op de machine staat, slaat het installatieprogramma deze stap stilzwijgend over en blijven de GUI en CLI gewoon werken.

#### Linux (.deb)

Het .deb-pakket plaatst het wheel-bestand onder de naam `/usr/lib/chloros/sdk/`. De `postinst` geeft de exacte opdracht weer — PEP 668-distributies weigeren standaard globale pip-schrijfbewerkingen, dus we installeren niet automatisch:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Voor Jetson-implementaties in een air-gapped omgeving verloopt dit volledig offline — het wheel staat al op de schijf.

#### Openbare PyPI

Voor hosts die uitsluitend pip gebruiken (geen Chloros-desktoppakket geïnstalleerd; workflows met remote-backend of uitsluitend DAQ):

```bash
pip install chloros-sdk
```

PyPI wordt bijgewerkt bij installatiepakketten van de release-versie, zodat het gepubliceerde wheel overeenkomt met de nieuwste stabiele release. Dev-builds (bijv. `1.1.4.dev1`) worden alleen meegeleverd via het gebundelde installatie-wheel.

#### Controleer

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **EenChloros-abonnement is vereist.** Voor alle SDK-aanroepen is een actieve Chloros-login vereist. Voer `chloros-cli login user@example.com 'YourPassword'` één keer per machine uit; de inloggegevens worden opgeslagen in `~/.chloros/`.

### Heb ik het desktop-pakket nodig?

Het pip-pakket alleen is voor de meeste workflows **niet** voldoende. Dit is wat elk SDK-oppervlak nodig heeft:

| SDK-oppervlak | Is het Desktop-pakket nodig? | Waarom |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja** | Start het backend-binaire bestand automatisch op bij `/usr/lib/chloros/chloros-backend` (Linux) of `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokaal)**/ Nee**(op afstand) | Pure HTTP-clients via de backend. Lokale backend → desktop-pakket vereist. Externe backend → `backend_url=`**via een tunnel** (zie Externe-backendmodus — meegeleverde backends binden alleen aan loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Bestuurt opgeslagen projecten via de backend. |
| Directe LATTICE-klassen (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Vereist de native runtime van de Arena-SDK, die in het desktop-pakket is meegeleverd. `CAMERA_AVAILABLE` is anders bij het importeren `False`. |
| Directe DAQ-klassen (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nee** | Pure Python via pyserial/bleak/zeroconf. Een omgeving met alleen pip kan DAQ’s van begin tot eind aansturen. |

### Remote-Backend-modus (host met alleen pip, via tunnel)

> **De meegeleverde backend is niet bereikbaar via het LAN.** De productieversie
> builds binden zich uitsluitend aan loopback (beide loopback-families) en weigeren categorisch de
> enige niet-loopback-modus (`CHLOROS_CLOUD_MODE`), dus
> `backend_url="http://<lan-ip>:5000"` **niet kan werken met een geïnstalleerde
> Chloros** — dat patroon werkte alleen met een source/dev
> backend. Om een backend op een andere machine aan te sturen, moet je de loopback-
> poort zelf doorsturen en de SDK naar de tunnel laten verwijzen:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Headless / CI / robotica-hosts kunnen één machine met de volledige desktopinstallatie als de „Chloros-server“ gebruiken en `pip install chloros-sdk` overal elders — maar het transport daartussen is de hierboven beschreven, door de gebruiker ingestelde tunnel, niet een directe LAN-URLatie.

> **Bekende beperking — `ChlorosLocal` ondersteunt niet uitsluitend pip.** `ChlorosLocal(backend_url=BACKEND)` lost momenteel een lokaal backend-binair bestand op in zijn constructor *voordat* de URL wordt doorzocht, en genereert de foutmelding `ChlorosBackendError` („Chloros-backend niet gevonden…“) wanneer er geen desktop-pakket is geïnstalleerd — zelfs als er een bereikbare externe backend is. Alleen de hierboven genoemde smart-connect-interface (`connect_camera` / `connect_array` / `connect_daq_sensor`, plus `analyze_array_network` en de `list_*` / `discover_*`-helpers) werkt vanaf een pip-only-host.

### DAQ-alleen-workflow (pip-only-host)

Als je alleen DAQ-sensoren nodig hebt en geen gebruik maakt van LATTICE-camera&#x27;s of beeldverwerking, is het pip-pakket op zichzelf staand:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Geen backend, geen .deb, geen Chloros+ login vereist voor directe hardware-DAQ-werkzaamheden.

---

## Snelstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Topniveau-API-index

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Beeldverwerking — `ChlorosLocal`

De hoofdklasse van de pipeline. Start de backend bij het eerste gebruik, maakt projecten aan en configureert deze, houdt de voortgang bij en retourneert samenvattingen na afloop van de run.

### Constructor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Methoden

| Methode | Beschrijving |
| --- | --- |
| `create_project(project_name, camera=None)` | Een nieuw project aanmaken (optioneel met een camerasjabloon zoals `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importeert RAW/TIF/JPG/DNG-afbeeldingen **en `.daq`-opnames van de lichtsensor**. Levert `count` (afbeeldingen) en `scan_count` (opnames) op. Geeft alleen een waarschuwing als de map geen van beide bevat. |
| `export_light_sensor(daq=True, csv=True)` | Schrijf gekalibreerde `.daq` + `.csv` voor elke lichtsensoropname in het project, in `<project>/Light Sensor/`. Zie [Lichtsensoropnames](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Stel de verwerkingsregelaars in. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Voer de pijplijn uit. Geeft `{"status": "complete", "async": False}` terug, plus een `summary`-sleutel wanneer de backend deze verstrekt — zie [Samenvatting en tips na uitvoering](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Controleer de status van de backend. |
| `logout()` | Wis de in de cache opgeslagen inloggegevens. |
| `shutdown_backend()` | Beëindig de backend (indien gestart met `SDK`). |
| `discover_cameras()` | LATTICE-camera’s detecteren **via de backend van deze instantie** (`/api/camera/discover`). Geeft een lijst met woordenboeken terug (`serial`, `model`, `ip`, …) — dezelfde structuur als die welke de GUI/CLIt zien. Lege lijst als er geen zijn gevonden of als de backend onbereikbaar is. |
| `camera_capture(output_dir, format="tiff", **settings)` | Leg één enkel frame vast**via de backend**(automatischgestart door deze handle), zodat het dezelfde voorbereiding krijgt als de GUI/CLI (standaard 12-bits, hergebruik van pool, ingebedde cal-metadata). Bepaal het doel met `serial=` of `device_index=`; geef `exposure`/`gain`/`pixel_format`/`preset` door als `**settings`. Geeft het verouderde metadata-woordenboek terug (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Levert samengestelde voorbeeldframes met overlay van een gepoolde camera — dunne MJPEG-client via de `/api/camera/<serial>/stream-annotated`-route van de backend (zebra / raster / dradenkruis / histogram / peaking / spot die aan de serverzijde worden getekend). `decode=True` levert BGR-arrays op; `False` levert onbewerkte JPEG-bytes op. Ook per project bereikbaar als `ChlorosProject.stream(overlays=True)`. |

Gebruik als contextmanager voor gegarandeerde opschoning:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Opnames van lichtsensoren — gekalibreerd `.daq` + `.csv`

Een DAQ-U / DAQ-M / DAQ-E kan worden opgenomen **zonder** de bijbehorende kalibratiebundel. Dat is
wat de openbare [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
recorders (`record_daq.py`) standaard doen: ze schrijven ruwe sensortellingen weg en voorzien het
bestand van een tijdstempel, zodat Chloros de fabriekskalibratie van die sensor **op serienummer** — eerst uit de lokale cache
en vervolgens uit de MAPIR-cloud — en past deze toe bij het importeren.

Chloros schrijft het resultaat terug als twee producten per opname, onder
`<project>/Light Sensor/`:

| Product | Wat het is |
| --- | --- |
| `<name>_calibrated.daq` | Het opnieuw te verwerken archief — hetzelfde schema als een live-opname, maar nu met vermelding van de bundel die het heeft geproduceerd. Bij het opnieuw importeren wordt het **niet** nogmaals gekalibreerd. |
| `<name>_calibrated.csv` | Spectrale bestralingssterkte in W/m²/nm op het eigen golflengteraster van de sensor, één rij per meting, plus fotometrische kolommen (totaal vermogen, fotopische/scotopische lux, PPFD en de uitsplitsing daarvan in blauw/groen/rood, piekgolflengte). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Alleen sensoren zonder bundel (DAQ-A).** Ruwe spectrale sensortellingen — *geen* bestralingssterkte. Zie hieronder. |

`process()` voert deze export uit als een van zijn fasen. Hiervoor zijn **geen** beelden nodig:
een zelfstandig ingezet lichtsensor vormt een volwaardige workflow, en een dergelijk project bevat per definitie
geen beelden.

**DAQ-A-opnames worden geëxporteerd als ruwe tellingen.** De DAQ-A-serie dateert van vóór het
serienummergebonden bundelsysteem en heeft geen bundel om op te halen — het wordt in het veld gekalibreerd aan de hand van een
reflectiedoel, waardoor er nooit een nodig was. Die opnames worden geëxporteerd
onder de stam `_raw` in plaats van `_calibrated`: een andere bestandsnaam in plaats van een vlag
in het bestand, omdat de claim het versturen per e-mail als naakte naam moet overleven. De
`.csv`-header vermeldt `raw spectral sensor counts (NOT irradiance)` en waarschuwt dat de
waarden **binnen** het bestand vergelijkbaar zijn — precies waarvoor kalibratie op basis van doelen
ze gebruikt — en niet tussen sensoren onderling. De vermogensafhankelijke fotometrische kolommen (totaal vermogen,
fotopische/scotopische lux, PPFD) geven **NULL** weer in plaats van geïntegreerde waarden op basis van tellingen.

Een DAQ-U / DAQ-M / DAQ-E waarvan de bundel simpelweg niet kon worden opgehaald, wordt nog steeds **overgeslagen**,
er wordt geen ruwe data geschreven: in dat geval bestaat de bundel wel en is „opnieuw verbinden en opnieuw verwerken“ een reëel advies.

Oudere **v1.01 / v1.02**-opnames (een DAQ-A-SD schrijft deze) bevatten geen tijdstip per meting,
alleen de schrijftijd van het bestand. De image↔downwelling-matcher weigert ze nog steeds — het matchen van een
frame met een schrijftijd zou onzichtbare fouten opleveren — maar de exporteur leest ze wel, en de
CSV geeft `clock=daq_created_on` weer, zodat het product aangeeft op welke klok het draait.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Een opname waarvan de kalibratiebundel niet kan worden opgehaald (offline, of een sensor zonder
kalibratie in het bestand) wordt gerapporteerd onder `skipped` **met de reden**. Het wordt nooit
uitgeschreven als een „gekalibreerd” bestand met ruwe tellingen — maak verbinding met het internet en
voer het opnieuw uit, waarna de export wordt voltooid.

### Voortgangscallbacks

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Samenvatting en tips na afloop

Na voltooiing haalt `process()` `GET /api/processing-summary` op en voegt de inhoud toe als `result["summary"]`. Het ophalen gebeurt op basis van &#x27;best effort&#x27; en blokkeert nooit een succesvolle terugkeer — als het overzicht niet beschikbaar is, valt `process()` terug naar de standaardvorm `{"status": "complete", "async": False}`. Elk item in `summary["hints"]` — volledige zinnen met de voorgestelde oplossing, bijvoorbeeld waarom een run geen uitvoer opleverde — wordt ook opnieuw verzonden als een Python `UserWarning`, zodat runs zonder uitvoer zichzelf diagnosticeren, zelfs als je het woordenboek nooit bekijkt:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` is de machinaal leesbare helft:

| Sleutel | Wat het telt |
| --- | --- |
| `models` | Cameragroepen in de run. |
| `images_in_groups` | Bronafbeeldingen verspreid over die groepen. |
| `targets_found` | Gedetecteerde reflectiedoelpunten. |
| `images_calibrated` | Beelden die tijdens de run zijn gekalibreerd. |
| `exported_files` | **Beeldproductbestanden die de run heeft geschreven.** |
| `daq_recordings_exported` / `daq_recordings_skipped` |-sensorregistraties, bewust apart geteld — ze zijn afkomstig uit een andere fase en bestaan ook voor runs zonder beeldmateriaal; als we ze zouden meerekenen, zou een run die alleen DAQ-gegevens bevatten eruitzien alsof er beelden zijn geëxporteerd. |

Daarnaast: `summary["output_dirs"]` (elke map waarnaar is geschreven),
`summary["light_sensor_export"]`, `summary["stopped"]` (waar wanneer de gebruiker de
run heeft onderbroken, zodat gedeeltelijke tellingen niet worden geïnterpreteerd als een voltooide run die te weinig heeft geproduceerd), en
`summary["groups"]` (de uitsplitsing per groep).

`exported_files` wordt door de pijplijn **tijdens het schrijven** vastgelegd, en niet achteraf uit de
beeldobjecten van het project gescand. De parallelle en GPU-strategieën bouwen hun eigen beeldobjecten
(in werkersubprocessen voor de GPU-paden), dus de oude scan rapporteerde
`0 file(s) written` voor elke dergelijke run en gaf vervolgens de ‘zero-exports’-hint af — bij runs
waarbij alles had gewerkt. Als je een script schrijft op basis van dit getal, geeft een succesvolle parallelle run nu
een telling anders dan nul weer.

Overslaan van lichtsensoren rapporteert de reden die de lezer daadwerkelijk voor elk bestand heeft vastgesteld — een
onleesbaar schema, een ontbrekende bundel, een schrijffout — **gededupliceerd**, zodat twintig bestanden
die om één oorzaak zijn overgeslagen, worden als één oorzaak weergegeven in plaats van twintig herhalingen ervan.

> **`process()` wordt niet gegenereerd wanneer een run geen afbeeldingen oplevert.** Dit is het enige punt waarop de SDK en
> de CLI opzettelijk van elkaar verschillen: `chloros-cli process` behandelt „producten werden aangevraagd, maar er werden geen
> geschreven“ als een fout en stopt met een waarde anders dan nul, terwijl de SDK normaal terugkeert en de
> toestand rapporteert via `summary` / hints. Als je pijplijn bij een lege run moet stoppen, controleer dit dan
> zelf — bekijk `summary` (of tel de bestanden in de projectmap) in plaats van te vertrouwen op
> het uitblijven van een uitzondering. De gebruikelijke oorzaken zijn een invoermap die niet werd herkend als een
> opname en producten die werden overgeslagen omdat ze niet van toepassing waren voor de aanwezige camera&#x27;s (bijv. straling van RGB-only
> camera&#x27;s).

### Handige functies

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Ondersteunde waarden

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrische uitvoer (LATTICE multispectrale pijplijn)

Het LATTICE multispectrale (M3C/M3M) exportniveau van de `process`-pijplijn — `reflectance` (standaard), `radiance`, `sensor-response` of `all` (elke toepasselijke modus per afbeelding) — komt overeen met de verwerkingsinstelling **&quot;Radiometrische uitvoer&quot;** van het project. `configure()` heeft hiervoor een speciaal trefwoord:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

De geavanceerde uitweg — het schrijven van de `"Radiometric output"`-sleutel van het project via `custom_settings` — werkt nog steeds, maar onthoud dat dit het gehele instellingenblok vervangt (zie de waarschuwing hieronder):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (de standaardinstelling) deelt de straling van de camera door de **aan de tijdstempel gekoppelde DAQ-neerwaartse straling**, die automatisch wordt afgeleid uit een opgenomen `.daq` (DAQ-U/M/E)**of een DAQ-M-specifieke `.csv`**die bij de beelden is aangetroffen; eventuele kalibratiebundels per camera of DAQ die lokaal ontbreken, worden bij het eerste gebruik**automatisch opgehaald uit AWS**. De CLI stelt dit beschikbaar als schakelaars per producttype op `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **vervangt** het volledige blok met berekende instellingen (het omzeilt de andere trefwoorden en validatie van `configure()`, zoals bedoeld). Neem bij gebruik alle `Project Settings`-sleutels op die voor jou van belang zijn, zoals in het bovenstaande voorbeeld.

---

## Smart-Connect voor LATTICE-camera&#x27;s

Permanente backend-sessies voor live-hardware. Dezelfde eindpunten die de GUI gebruikt, zodat het gedrag identiek is op SDK / CLI / GUI.

### Eén camera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()`-handtekening

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession`-methoden

| Methode | Beschrijving |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICam-knooppunten lezen; retourneert `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Knooppunten schrijven op basis van vriendelijke naam (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Leg een **enkel** beeld vast. Geeft een lijst met één element terug, bestaande uit woordenboeken met beeldmetadata. (Burst/multi-frame-opname is verwijderd — roep `capture()` in een lus aan als je een reeks nodig hebt.) |
| `disconnect()` | Vrijgeven uit de pool. Geen actie als we gekoppeld zijn aan een reeds geopende sessie. |

`capture()`-exportinstellingen (hetzelfde model als de array + GUI):

- `processing` / `levels` — `processing="all"` slaat elk toepasselijk exporttype op; `levels=["raw","radiance"]` slaat alleen die op (overschrijft `processing`). Laat beide weg voor de standaardinstelling van de backend.
- `force_daq=True` — sla de toegewezen DAQ/DLS-meting op als een `.daq`-sidecar, zelfs bij een opname die uitsluitend uit ruwe gegevens bestaat, zodat het frame later opnieuw kan worden verwerkt tot reflectantie/index. Geen actie als er geen DAQ is gekoppeld.

### Gesynchroniseerde array — `ArraySession` (Smart-Prep)

`connect_array` is **het aanbevolen startpunt** voor opstellingen met meerdere camera’s. Het voert achter de schermen de volledige GUI-smart-prep-workflow uit:

1. **Netwerkanalyse** (`/api/camera/array/recommend`) — zoekt de grootste framegrootte die past binnen de sim-emit-laag zonder frames te verliezen.
2. **Automatische laagselectie** — `sim-capture-sim-emit` als de verbinding dit aankan; anders `sim-capture-ftd-stagger` of `slip-emit-and-capture`.
3. **Automatisch verkleinen**— verkleint stilzwijgend de framegrootte / verhoogt de binning wanneer de verbinding de gevraagde resolutie niet aankan.**Dit vangnet dekt geen totale overbelasting**: te veel camera’s voor de verbinding kan niet worden opgelost door frames te verkleinen — zie [Overbelasting](#over-subscription-the-per-cam-floor).
4. **PTP ingeschakeld**als standaardinstelling — tijdstempels van verschillende camera’s worden afgestemd op één gedeelde klok tot op**~1 ms**nauwkeurig. Gelijktijdige belichting wordt geregeld door de M8-hardwaretrigger (**&lt; 100 µs** tussen modules), niet via PTP: PTP synchroniseert *tijdstempels*, niet de belichtingen.
5. **Automatische keuze van pixelformaat per camera** — RGB-camera’s → `BayerRG8`, multispec → `BayerRG12`.
6. **AE-initialisatie** — legt de huidige AE-status van elke camera vast, zodat de verbinding de belichting niet halverwege resetteert.
7. **GPIO-triggerconfiguratie** — `connect_array` schakelt elke camera in (`TriggerMode=On`, `TriggerSource=Line2`) zodat de puls van de master de slaves via de M8-kabel aanstuurt. Dit is een stap die alleen bij een array van toepassing is: een enkele camera die met `LatticeCamera` wordt geopend, werkt in plaats daarvan in de vrije modus.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()`-handtekening

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier`-waarden:
- `"sim-capture-sim-emit"` — werkelijk gelijktijdig (alle cams worden op dezelfde klokflank geactiveerd).
- `"sim-capture-ftd-stagger"` — flexibele verspringing in het tijdsdomein (cams zenden uit op lichtjes verschoven tijdstippen, zodat pakketten serieel over de kabel lopen).
- `"slip-emit-and-capture"` — sequentiële opname per cam (geen temporele synchronisatie; enige optie wanneer geen enkele framegrootte past bij sim).

`wire_ceiling_mbps` overschrijft het **duurzame bandbreedtebudget van de host** in MB/s — het enige
getal waaraan de toewijzing van de gehele array is gekoppeld. Laat het op `None` staan om de automatisch gedetecteerde
waarde te gebruiken. Verlaag deze waarde wanneer de array GVSP-beschadigde frames meldt: de automatisch bepaalde waarde wordt afgeleid
van de door de NIC gerapporteerde linksnelheid, die USB-adapters, smalle PCIe-lanes en
drukke gedeelde fabrics overschat — en deze overschatting komt tot uiting in beschadigde frames in plaats van een
zichtbaar trage verbinding. De waarde wordt opgeslagen in het array-captureblok van het project, dus bij het
opnieuw openen of later via `connect_array` wordt deze net als elke andere array-instelling hersteld.
Zie [Array Health](#array-health--which-subsystem-is-losing-frames).

#### Over-Subscription (de ondergrens per camera)

Sim-emit-pacing wijst elke camera een aandeel toe van het botsingsveilige bandbreedtebudget, met een ondergrens van **8 MB/s per camera**(`per_cam_floor_bps`). Zodra `N × floor` het botsingsveilige plafond overschrijdt,**overbelast de array de verbinding**— de storingsmodus is GVSP-pakketverlies, niet een lagere framesnelheid — en er bestaat geen oplossing op basis van framegrootte:**binning en ROI verlagen het aantal bytes per frame, niet het aantal gepace-de bytes per seconde**dat de geaggregeerde controle vergelijkt. Praktische limieten voor volledige resolutie op een 1 GbE-host:**6 camera’s bij 1500 MTU, 9 met jumboframes** (`max_cams_collision_safe` in het analyseverslag geeft de limiet voor uw verbinding weer). Oplossingen: minder camera’s, end-to-end jumboframes of een snellere NIC.

- De reacties `analyze_array_network()` en `/api/camera/array/connect` bevatten `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` en `per_cam_floor_bps`. Wanneer `oversubscribed` waar is, zet de projectie **de fps-velden op nul** (`achievable_fps_max` / `fps_bright` / `fps_dark`) in plaats van een misleidende, trage maar werkende snelheid te rapporteren.
- `POST /api/camera/array/connect` accepteert een `pin_resolution`-bodyparameter (**alleen HTTP — geen SDK-kwarg**; `connect_array` maakt deze niet beschikbaar). Door vastzetting (pinning) wordt het veiligheidsnet van de ‘walk-down’ bij binning verwijderd, dus een overbelaste verbinding waarbij `pin_resolution` is ingesteld, wordt**hard-geweigerd** met een foutmelding waarin alle mogelijke oplossingen worden genoemd. Zonder pinning gaat de verbinding door met de walk-down, maar wordt er gewaarschuwd dat het verkleinen het totaal niet kan opheffen.
- Uitweg voor bench-work: stel `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` in de omgeving van de backend in om de weigering te verlagen naar een luide waarschuwing — je maakt toch verbinding en accepteert het pakketverlies.

#### Array-status — welk subsysteem verliest frames

`GET /api/camera/array/<array_id>/capability` bevat een actief `health`-blok op een
aangesloten array, opnieuw geëvalueerd in een voortschrijdend **10-seconden** venster. Het splitst frameverlies
op in de twee oorzaken die tegengestelde oplossingen vereisen, in plaats van één „onvolledig”-percentage dat
geen van beide benoemt:

| Veld | Wat het betekent | Welk subsysteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriële poort) | Het frame **kwam aan en was structureel defect**— GVSP-pakketverlies. |**Netwerk**: bandbreedte, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Het frame **is helemaal niet aangekomen**— de camera is niet geactiveerd, of er is niets uitgezonden. |**Trigger / synchronisatie**: M8-kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Laagste verzendfrequentie per camera. | — |
| `per_cam_rate_pct` | Gecombineerd percentage onvolledige opnames per camera (beide oorzaken samen). | — |
| `stable_for_seconds` | Hoe lang elke camera onder de 0,01 % is gebleven. | — |

Naast `health` vermeldt hetzelfde record het aantal dat de gehele toewijzing in beslag neemt:

| Veld | Wat het betekent |
| --- | --- |
| `wire_ceiling_mbps` | Het geldende, aanhoudende bandbreedtebudget van de host, MB/s. |
| `wire_ceiling_source` | Waar dat getal vandaan komt, in woorden — bijv. `USB-capped 200 MB/s (was theoretical 1062; …)` of `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` wanneer `wire_ceiling_mbps=` dit heeft ingesteld. |
| `nic_is_usb` | `true` voor een USB-ethernetadapter. |

Er is geen SDK-wrapper voor dit eindpunt — lees het rechtstreeks:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Uitlezing:** een `gvsp_corrupt_rate_pct`-waarde anders dan nul met `never_arrived_rate_pct` op 0 betekent
dat de trigger en de kabelsynchronisatie perfect zijn en dat 100 % van het verlies zit in het netwerkpad — verlaag
`wire_ceiling_mbps` en maak opnieuw verbinding. Het omgekeerde patroon wijst in plaats daarvan op de synchronisatiekabel of de
triggerlijn.

> **`target_fps` is niet de bepalende factor voor beschadigde frames.** De GevSCPD-pacing wordt eenmalig ingesteld bij
> het verbinden, dus het verlagen van de triggerfrequentie verandert de duty cycle en niet de
> burst-snelheid bij gelijktijdige uitzending. Een gemeten verlaging van de vraag met een factor 5 leverde geen verbetering op, terwijl
> het verlagen van het kabelplafond van 240 naar 200 MB/s bij dezelfde opstelling het percentage beschadigde frames terugbracht van 10,4 % naar
> 0,00 %.

> **Automatische inkrimping halverwege de stream is niet beschikbaar op TRI032S-firmware.** Een actieve array kan
> dit niet zelf verhelpen; verbreek de verbinding en maak opnieuw verbinding, zodat de connect-time-picker opnieuw plant op basis van
> de nieuwe limiet.

Een **USB-ethernetadapter wordt door de probe beperkt tot 200 MB/s**, ongeacht de
specificaties op het typeplaatje: de efficiëntietabel die een verbindingssnelheid omzet in een continu cijfer is
afgeleid van PCIe, en een USB-NIC geeft zijn ethernetverbindingssnelheid aan terwijl deze wordt beperkt door de
USB-bus en het bijbehorende stuurprogramma. De limiet is absoluut, geen fractie — een USB 1 GbE-adapter
haalt ~80 MB/s en ondervindt hier geen hinder van.

#### `ArraySession`-methoden

| Methode | Beschrijving |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Eén gesynchroniseerde opnamegroep. Geeft een `CaptureResult` terug (lijst met frame-dictionaries + `.skipped`). Exportinstellingen hieronder. |
| `capture(..., smart=True)` | **Slimme opname** — wacht tot AE op alle camera’s is gestabiliseerd en start dan de opname. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Snelste opname: alleen raw + de toegewezen DAQ-waarde (+ de vrije gecombineerde index). Komt overeen met de &quot;Fastest Capture&quot;. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enkelvoudig / Continu / Interval in één begrensde lus. Geeft `list[CaptureResult]` terug.**Vereist `count` en/of `duration_s`** zodat het wordt beëindigd (de „SDK“ heeft geen Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Start het opnemen van de live gecombineerde-indexweergave naar video/GIF → `RecorderHandle`. Eén samengestelde recorder per array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Start een raw-Bayer-burst met hoge fps → `RecorderHandle`. Offline opnieuw verwerken met `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Verwerk een opgeslagen raw-burst offline tot gekalibreerde video(&#x27;s). Blokkeert totdat dit voltooid is (`wait=True`) en retourneert `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Controleer de status van een offline bouwtaak: `{running, result, error, burst_dir}`. |
| `disconnect()` | Geef de hele array vrij. |

`capture()` exportcontroles (hetzelfde eindpunt dat de GUI/CLI gebruiken):

- `processing` / `levels` — `processing="all"` (of `levels=["raw","radiance",…]`) slaat elk toepasselijk exporttype per cam op; een enkele `processing`-waarde slaat alleen dat niveau op.
- `aligned=True` — pas de niet-ruwe export van elk element aan aan het [uitlijningsprofiel](#array-alignment) (geco-registreerd); onbewerkte gegevens blijven onvervormd maar dragen de transformatie in de metagegevens. Valt terug op niet-uitgelijnd (met een waarschuwing die wordt weergegeven in de `alignment` van het resultaat) als de reeks geen profiel heeft.
- `render_index=False` — sla de vegetatie-index-overlay per camera over; standaard wordt deze weergegeven waar deze is geconfigureerd.
- `force_daq=True` — sla de toegewezen DAQ/DLS-waarde op als een `.daq` sidecar, zelfs als geen van de gekozen niveaus dit nodig heeft.

**TIFF-compressie (alleen de HTTP-knop):**`ArraySession.capture()` verzendt geen `compression`-sleutel, dus de standaardinstelling van de backend is van toepassing — `POST /api/camera/array/capture` leest een `compression`-bodyparam, `"deflate"` standaard (verliesloze zlib L1 + horizontale voorspeller, ~4,1 MB per frame in volledige resolutie). `"none"` schrijft ongecomprimeerd (~6,3 MB/frame) met een**~5× snellere schrijfsnelheid** — beide zijn verliesvrij en worden bij het importeren identiek gelezen. De `SDK` biedt hiervoor geen kwarg; de uitweg is ``chloros-cli lattice array-capture --compression none`` of onbewerkte `HTTP`. DEFLATE houdt ook de GIL van de Python vast, dus gecomprimeerde schrijfbewerkingen kunnen niet parallel worden uitgevoerd over de schrijf-threads per camera — voor een continue opname met 8 camera’s op volledige resolutie bij sensorsnelheid is `compression: "none"` nodig. Details: [CLI Referentie → array-capture](cli-reference.md).**Overschrijvingen per lid bij export (alleen HTTP):**hetzelfde eindpunt accepteert ook `exclude_serials` (lijst — verwijder leden uit de opgeslagen set; de array wordt nog steeds geactiveerd als één gesynchroniseerde groep en uitgesloten leden worden geretourneerd in `excluded`), `serial_levels` (`{serial: [level tokens]}` overschrijvingen per cam-niveau) en `serial_index` (`{serial: bool}`-overrides per camera voor index-overlay). Dit zijn GUI-pariteitsparameters en**nog geen kwargs vSDK**; leden die ontbreken in de kaarten vallen terug op de array-brede `levels` / `render_index`.

##### Overgeslagen cams inspecteren — `CaptureResult.skipped`

`ArraySession.capture()` retourneert een `CaptureResult`, wat een `list`-subklasse is: doorloop deze, indexeer deze, `len()` — elk bestaand patroon blijft werken. Nieuwe code kan het `.skipped`-attribuut inspecteren om te zien welke cams werden uitgesloten en waarom. Het meest voorkomende geval is RGB cams in een gemengde filterarray wanneer je om `processing="radiance"` of `"reflectance"` vraagt — straling per Bayer-pixel is zinloos voor een breedband-sensor, dus de backend slaat die cams over in plaats van onzin te produceren.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Reden-tokens volgen het patroon `<level>-not-applicable-to-rgb-cam` (één vermelding per overgeslagen niveau, elk met `level`). De reflectiespecifieke overslagen zijn `reflectance-skipped-no-fresh-dls` (geen nieuwe downwelling-meting beschikbaar), `reflectance-skipped-bound-daq-unavailable (…)` (de gekoppelde DAQ kon niet bereikt worden), en `dls-uncalibrated-band-<nm>` — de band ligt grotendeels buiten hetradiometrisch gekalibreerd bereik (~374–974 nm), dus de op de DAQ gebaseerde absolute reflectie-scheiding wordt geweigerd en het frame schakelt luid en duidelijk over naar sensorrespons. Van de leverbare SKU’s veroorzaakt alleen F988 dit; het ondersteunde traject van die camera is de workflow met het reflectantiepaneel.

`processing`-niveaus:

| Niveau | Uitvoer |
| --- | --- |
| `"raw"` | Eénkanaals Bayer (monochrome camera’s: de enkele band) rechtstreeks vanuit de sensor. |
| `"debayered"` *(standaard SDK)* | 3-kanaals BGR via bilineaire demosaïcering (mono-camera’s: 1-kanaals grijsschaal). |
| `"radiance"` | float32 W/m²/sr/nm via de volledige radiometrische keten. Alleen multispectraal — RGB-camera’s worden overgeslagen. |
| `"reflectance"` | uint16 0..32768 (Pix4D-ready); vereist een live DAQ-koppeling voor absolute referentie. Alleen multispectraal. |
| `"display"` | Volledige keten die overeenkomt met het GUI-voorbeeld (CCM + WB + gamma volgens het profiel van de camera). |
| `"all"` | **Eén bestand per toepasselijk niveau** voor elke camera (overeenkomend met de GUI &quot;Capture All&quot; / CLI standaardinstelling). Het geretourneerde `CaptureResult` bevat vervolgens één frame-dict per `(cam, level)`, met het niveau in elke dict; niet-toepasselijke niveaus verschijnen in `.skipped`. De DAQ-meting die voor elk reflectantieframe wordt gebruikt, wordt opgeslagen als een `.daq`-sidecar. |

> **Opmerking — de standaardinstelling wijkt af van de CLI.** `ArraySession.capture()` is standaard ingesteld op `processing="debayered"`; het commando `chloros-cli lattice array-capture` is standaard ingesteld op `processing="all"`. Geef `processing="all"` expliciet door vanuit de SDK om de CLI /GUI-opslag op meerdere niveaus te weerspiegelen.

### Opnamemodi en recorders

Het array-oppervlak weerspiegelt het opnamepaneel van de GUI: de opnamemodi Enkel / Continu / Interval / Snelste sluitertijd, plus twee recorders (live-composietvideo en onbewerkte burst → offline herverwerking).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**is de Continu/Interval-lus van de SDK. Omdat er geen `Ctrl+C` is om deze vanuit een script te onderbreken,**moet** je `count` en/of `duration_s` doorgeven (deze stopt wanneer een van beide wordt bereikt). `interval_s` wordt gemeten vanaf het begin van elke doorloop (in overeenstemming met de GUI). De overige kwargs worden rechtstreeks doorgegeven aan `capture()`.
- **`record`** is *monitoring-kwaliteit*: het legt de live gecombineerde indexcomposiet vast zoals deze wordt weergegeven, dus de gecombineerde stream moet open zijn om frames te kunnen ontvangen. Eén composietrecorder per array (geeft een foutmelding als er al een actief is).
- **`burst` → `build_video`** is *analyse-kwaliteit*: `burst` schrijft ruwe frames + een manifest per frame + één `.daq` per afzonderlijke DLS-uitlezing onder `<output>/bursts/<base>/` op de volledige snelheid (geen keten, geen exiftool, geen liveweergave). `build_video` stemt elk frame qua tijd af op het dichtstbijzijnde `.daq` en voert de radiance/reflectance/index-keten van de importpijplijn opnieuw. `products` is een lijst van `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (standaard: de gecombineerde index). `burst().stop()` start ook automatisch een &#x27;best-effort&#x27;-opbouw van de gecombineerde index, die als `build_job` in het stopresultaat wordt geretourneerd.

#### `RecorderHandle`

Geretourneerd door `ArraySession.record()` en `ArraySession.burst()`. Gebruik dit als een contextmanager om automatisch te stoppen bij het verlaten van de scope, of bestuur het handmatig.

| Lid | Beschrijving |
| --- | --- |
| `job_id` | Backend-taak-id (str). |
| `kind` | `"composite"` (van `record`) of `"raw"` (van `burst`). |
| `start_stats` | Het dict dat wordt geretourneerd door de `start`-aanroep. |
| `result` | `None` tijdens het draaien; het definitieve stop-resultaat-dict zodra gestopt. |
| `stats(timeout=10.0)` | Live taakstatistieken (geschreven frames, gerealiseerde fps, verstreken tijd). |
| `stop(timeout=60.0)` | De recorder stoppen; retourneert en slaat het eindresultaat op in de cache. Idempotent (een tweede aanroep retourneert het in de cache opgeslagen resultaat). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Aansluiten op een reeds verbonden array — `attach_array`

Als de array al actief is (de GUI heeft deze geopend, of een eerdere SDK-sessie heeft `connect_array` aangeroepen), gebruik dan `attach_array` om een handle te verkrijgen in plaats van opnieuw verbinding te maken. `connect_array` geeft in die situatie altijd de foutmelding „Camera  is<sn> al in array<id>&quot; in die situatie, omdat het versturen van een POST-verzoek met `/array/connect` voor een lid in de pool niet idempotent is; `attach_array` leest `/api/camera/array/list` en vergelijkt deze met array_id of serials.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Patroon: SDK-scripts die samen-tenant met de desktop-GUI moeten eerst `attach_array` proberen en terugvallen op `connect_array` als er nog geen array in de pool zit.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Belangrijk — het afsluiten van de contextmanager verbreekt WEL de verbinding.**`ArraySession.disconnect()` verstuurt altijd een POST naar `/array/disconnect`; er is geen ‘attached-not-owned’-controle zoals bij `CameraSession` / `DAQSensorSession`. Als je de array deelt met de GUI en de array niet wilt afbreken bij het verlaten van de scope,**gebruik dan niet het `with`-blok** — bewaar de handle in een normale variabele en sla de expliciete `disconnect()` over:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Hulp bij netwerkanalyse

Handig voordat je de array opent — geeft aan of je voorgestelde instellingen zullen passen:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` is een van `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (anders `error`). `auto_capped_fps` betekent dat de gevraagde resolutie alleen bij een begrensde triggerfrequentie past in de RX-ring — behoud de resolutie en geef `target_fps=result["recommended"]["recommended_target_fps"]` door aan `connect_array` door (zie [Voorbeeld 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Hoe de projectie te interpreteren** (zelfde model als het GUI-paneel ‘Array-instellingen’):

- **Burst (`frame_bytes_total`) wordt per camera opgeteld in het werkelijke pixelformaat van elke camera.**Mono**M3M**-camera’s streamen Mono12 (2 B/px), ongeacht de `pixel_format`-waarde die je doorgeeft, dus een frame met volledige resolutie van 4 camera’s is**~25 MB** bij drie mono-camera’s, niet de ~12,6 MB die een aanname van volledig 8-bit zou opleveren. De backend bepaalt het formaat van elke camera op basis van het model.
- **Admittance (`burst_fits_nic_ring`) is *drain-bewust**, niet ‘hele burst versus ring’: sim-emit past wanneer de host de RX-ring sneller leegtrekt dan de camera’s deze vullen. Een 10G-host + 1 GbE-camera’s**laten** volledige resolutie toe, zelfs wanneer de burst de ring overschrijdt; een 1 GbE-host blokkeert (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` is een conservatieve bovengrens voor seriële ophaling** — `max(readout+emit, N×emit)` waarbij de uitstoot per camera wordt begrensd tot de 1 GbE-camerakoppeling, onafhankelijk van de belichting. Bijv. ~2,8 fps voor een 4-camera-array met volledige resolutie en 12-bits (komt overeen met de gemeten ~2,7–3,0 van de runtime). Volledig model: [CLI Referentie → Array fps &amp; burst-model](cli-reference.md#array-fps--burst-model).
- **Over-subscription (`oversubscribed: true`) betekent dat N × de ondergrens per camera de botsingsveilige bovengrens overschrijdt** — de fps-velden (`achievable_fps_max` / `fps_bright` / `fps_dark`) geven 0 weer, en automatisch verkleinen/binning kan dit niet verhelpen (deze verlagen het aantal bytes per frame, niet het aantal bytes per seconde). Oplossingen zijn minder camera’s, jumboframes of een snellere netwerkkaart; `max_cams_collision_safe` rapporteert de bovengrens (6 camera’s met volledige resolutie op 1 GbE bij 1500 MTU, 9 met jumboframes). Het antwoord bevat ook `aggregate_demand_bps`, `collision_safe_ceiling_bps` en `per_cam_floor_bps` (8 MB/s) bevat. Zie [Over-Subscription](#over-subscription-the-per-cam-floor).

### Detectie en weergave

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE-arrays voeren continu AE uit op de achtergrond zodra ze zijn aangesloten, maar het duurt even voordat een nieuw gerichte scène is geconvergeerd. **Smart-Capture** biedt hier een handige oplossing: het peilt de belichting van elke camera, wacht tot het array stabiel is binnen een venster en activeert vervolgens de opname. Dit komt overeen met de GUI: de „smart“-opnameknop in de desktop-app roept hetzelfde backend-eindpunt aan.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Bij het instellen via `ChlorosProject` (volgende paragraaf) krijg je meer instelmogelijkheden:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Het smart-AE-beleid is standaard conservatief. Verkrap `exposure_tolerance_pct` voor veeleisend radiometrisch werk; verruim het voor snel veranderende scènes waarbij je gewoon ‘goed genoeg’ wilt.

---

## DAQ-sensorsessies

Permanente backend-pool voor spectrale sensoren (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Weerspiegelt het camera-oppervlak: smart-detect, hergebruik van de pool, idempotente koppeling.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioriteit: Ethernet → BLE → USB. Geef een willekeurige expliciete hint door om het transport vast te leggen.

### Vastgelegd transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession`-methoden

| Methode | Beschrijving |
| --- | --- |
| `status(timeout=10.0)` | Overzicht van pool-items (streaming/opnamestatus, golflengtebereik, kalibratie-SHA, integratietijd, frame_avg, AE-status). |
| `latest(n=1, timeout=10.0)` | Geeft maximaal N meest recente spectrumframes terug. |
| `stream_start()` / `stream_stop()` | Streaming hervatten / pauzeren (handle blijft open). |
| `record_start(output_dir=None, device_name=None)` | Start het opnemen van een .daq-bestand. Geeft het bestandspad terug. Wordt geweigerd voor DAQ-U/M zonder AWS-kalibratiebundel (DAQ-E is hiervan vrijgesteld). |
| `record_stop()` | Stop de opname. Geeft `{path, rows}` terug. |
| `disconnect()` | Vrijgeven uit de pool. Geen actie voor gekoppelde, maar niet-eigen handles. |

> **Cap-correctieprofielen (`cap_id`) zijn geen ‘SDK’-knop.** `connect_daq_sensor()` / `DAQSensorSession` stellen geen `cap_id`-parameter of `set_cap`-methode. Selecteer een cap-correctieprofiel voor de vloot via de CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) of de `/api/daq`-routes HTTP van de backend (`/api/daq/connect` en `/api/daq/<id>/cap-id` accepteren `cap_id`).

### Detectie — een adres vinden om verbinding mee te maken

`discover_daq_sensors()` scant USB / BLE / ETH op sensoren die je *zou kunnen* openen. Het is de DAQ-tegenhanger van `discover_lattice_cameras()`, en de enige manier om een **BLE-MAC-adres van een DAQ-M** te verkrijgen — een DAQ-E heeft een hostnaam en een DAQ-U een COM-poort, maar een MAC-adres staat noch op het apparaat vermeld, noch wordt het door het besturingssysteem weergegeven.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Veld | Beschrijving |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-poort / BLE MAC / hostnaam — doorgeven aan `connect_daq_sensor` als `port=` / `mac=` / `eth_host=`. |
| `display` | Voor mensen leesbaar label. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, of `None` voor een poort die de scan niet kan identificeren (USB-seriële adapters zijn zonder een probe niet te onderscheiden, dus onbekende poorten worden weergegeven in plaats van verborgen). |
| `extra` | Details per transport (BLE-advertised-naam, USB-fabrikant, DAQ-E ip/fw/…). Lege waarden worden weggelaten. |

| Parameter | Standaard | Beschrijving |
| --- | --- | --- |
| `transports` | alle drie | Volgorde (of CSV-string) die de scan beperkt. Het loont de moeite deze door te geven als je weet wat je wilt — BLE is het traagste onderdeel. |
| `scan_timeout` | 5 | Scanvenster per transport in seconden; de backend beperkt dit tot 1–20. |
| `timeout` | 60,0 | Maximum voor de ‘HTTP’ voor de gehele aanroep (zoals elders in de SDK). |
| `auto_start_backend` | `True` | Start een lokale backend als er geen actief is. Wordt nooit gestart voor een externe `backend_url`. |

> **Sensoren die al openstaan in de pool verschijnen niet.** Een aangesloten BLE-randapparaat stopt met adverteren en een open COM-poort kan niet worden gescand, dus de discovery-lijst toont wat *beschikbaar is om verbinding mee te maken*. Een leeg resultaat direct nadat je iets hebt aangesloten is te verwachten — gebruik `list_daq_sensors()` voor wat je al in bezit hebt. Transporten waarvan de scan niet kan worden uitgevoerd (geen bleak / zeroconf geïnstalleerd) worden overgeslagen in plaats van een foutmelding te geven, zodat een machine zonder Bluetooth nog steeds de USB- en ETH-resultaten ontvangt.

### Lijst

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Gedeeld gebruik met GUI / CLI

Als de GUI al een sensor heeft geopend, retourneert het aanroepen van `connect_daq_sensor(port="COM3")` vanuit Python een handle met de markering `already_connected=True`. De `disconnect()` is dan een no-op, zodat je SDK-script de sensor niet onder de GUI vandaan haalt wanneer de scope wordt afgesloten.

### Direct-Hardware-klassen (Geen backend)

`daq_sdk` wordt opnieuw geëxporteerd door `chloros_sdk`, zodat je sensoren ook end-to-end in het proces kunt aansturen zonder de backend:

> **Beschikbaarheid:**`daq_sdk` wordt meegeleverd met de desktopinstallatie van Chloros,**niet** bij het PyPI-pakket — `pip install chloros-sdk` biedt je `lattice_sdk`, maar laat `chloros_sdk.DAQ_AVAILABLE == False` achterwege. Controleer deze vlag voordat je deze klassen gebruikt; op een hostschijf met alleen pip moet je de sensor in plaats daarvan via [`connect_daq_sensor()`](#daq-sensor-sessions) aansturen, waarvoor geen lokale transportbibliotheken nodig zijn.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Geef de voorkeur aan het smart-connect-pad (`connect_daq_sensor`) wanneer je gedeeld eigendom met de GUI wilt; gebruik de directe klassen voor headless-scripts die exclusief eigenaar zijn van de sensor.

---

## Projectautomatisering — `ChlorosProject`

Een opgeslagen Chloros-project is een map die `cameras.json` + `sensors.json` + `project.json` bevat. `open_project` laadt het manifest, en `connect_all` brengt elk opgeslagen apparaat online met de opgeslagen instellingen — dezelfde hardwarestatus die de GUI zou produceren.

### Minimaal voorbeeld

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Of als contextmanager:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-methoden

| Methode | Beschrijving |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Elk opgeslagen apparaat detecteren en verbinden. Geeft een verbindingsrapport per klasse terug. Maakt gebruik van een actieve backend wanneer er een luistert op `127.0.0.1:5000`; anders valt het stilzwijgend terug op directe (backend-vrije) `lattice_sdk`-apparaatbesturing — er wordt nooit een backend gestart. |
| `disconnect_all()` | Alles afbreken. |
| `capture_all(output_dir=".")` | Eén frame van elke camera + array + spectrum van elke sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Generator die BGR `numpy`-frames produceert van een genoemde camera (of array). `overlays=False` is een directe `lattice_sdk`-opnamelus (arrays leveren `{serial: frame}`-dicts op). `overlays=True` leidt via `ChlorosLocal.camera_stream()` → de `/api/camera/<serial>/stream-annotated` MJPEG-feed van de backend, waarbij het opgeslagen `ui.overlay`-blok van de camera wordt doorgegeven als queryparameters. Vereist backend-modus en een **standalone camera**: een camera in directe modus genereert `RuntimeError` (de backend kan geen camera ophalen die eigendom is van dit proces) en een array genereert `NotImplementedError` (overlays samengevoegd per camera — stream een lid op naam). Eénmalig equivalent: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Voer uitlijning uit op elke momenteel aangesloten array. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Voer de kalibratie-/indexpijplijn uit op de afbeeldingen van het project (omvat `ChlorosLocal.process`; deze vier zijn de **enige** toegestane kwargs — `indices=` enz. veroorzaken een uitzondering `TypeError`; stel indices in via `ChlorosLocal.configure()`). Construeert op een ‘lazy’ manier een `ChlorosLocal()`, die automatisch een backend start. |

Attributen:
- `proj.cameras` — `Dict[str, CameraHandle]` op naam én serienummer.
- `proj.arrays` — `Dict[str, ArrayHandle]` met naam EN array_id als sleutel.
- `proj.sensors` — `Dict[str, SensorHandle]` met naam EN slot_id als sleutel.
- `proj.config` — `project.json["config"]`-woordenboek.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Verwerkingsniveaus.** `capture()`, `grab()`, en `frame_stream()` gebruiken allemaal hetzelfde `processing`-token,
en de keten is cumulatief — elk niveau voert alles uit dat erboven staat:

| Niveau | Uitvoer | Opmerkingen |
| --- | --- | --- |
| `raw` | 1-kanaals Bayer, sensor-native | Geen demosaïek. Overlays zijn op dit niveau niet beschikbaar. |
| `debayered` | 3-kanaals BGR (**standaard**) | Bilineaire demosaicing. Het enige niveau dat werkt zonder backend-modus. |
| `radiance` | float32, W/m²/sr/nm | Volledige radiometrische keten: demosaïcering + 3×3 unmix (multispec) + DSNU + flat-field + NIST-schaal, waarbij belichting × versterking zijn weggefilterd zodat de waarden absoluut zijn. |
| `reflectance` | uint16, 32768 = 1,0 | Straling gedeeld door neerwaartse stralingsintensiteit (ρ = π·L/E). Vereist een DLS/DAQ-meting — zie de opmerking hieronder. |
| `display` | 8-bit sRGB-achtig | GUI-equivalente weergave: CCM + witbalans + gamma via hetactieve kleurprofiel. |

Alles behalve `debayered` vereist de backend-modus; een camera in directe modus genereert
`NotImplementedError`. `reflectance` vereist een bruikbare downwelling-meting — het einde van het frame haalt
de gebundelde DAQ automatisch in het DLS-slot van de camera, maar zonder gekoppelde DAQ weigert de keten de
reflectantie-uitgang en vermeldt de degradatie eerlijk in de geretourneerde metadata in plaats van stilzwijgend
een inferieur product terug te geven.

> **Reflectantie DN-schaal — codeer deze niet hard.** LATTICE-reflectantie gebruikt `32768` = ρ 1,0 en vermeldt
> XMP `Chloros:PixelScale=32768`; Survey3-reflectantie gebruikt `65535` = ρ 1,0 en bevat geen
> `Chloros:*`-tags. Lees de tag en deel erdoor. Deze is gedefinieerd in het uint16-domein, dus blijft het
> `32768` voor elk formaat dat wordt geschaald (16-bits TIFF, 8-bits PNG /JPG, 32-bits procent) — normaliseer
> het opgeslagen dtype eerst terug naar uint16 (×257 vanaf 8-bits, ×65535 vanaf float). De enige uitzondering:
> een 8-bits-bronopname die is opgeslagen als 8-bit TIFF wordt *afgekapt*, niet opnieuw geschaald, dus er is geen schaal die
> deze beschrijft — Chloros laat in dat geval `PixelScale` en de MicaSense-tuple volledig weg. Behandel een ontbrekende
> tag in een LATTICE-reflectantiebestand als &quot;geen geldige schaal&quot;, niet als een standaardwaarde.

> **EXIF wordt meegenomen naar de export.** `process()` kopieert het GPS-blok van de bronopname
> **en de bijbehorende ExifIFD** over naar elk product, waardoor exportbestanden `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` en `CameraSerialNumber`, evenals de
> georeferentie. Pix4D berekent de grondmonstersafstand op basis van `FocalLength` — zonder dit
> krijgt de reconstructie een volkomen verkeerde schaal (in een gemeten geval werd een locatie van 411 m
> omgezet in een locatie van 47,8 km). Het bestand is bewust niet `-all:all`: de structurele tags van IFD0 verstoren
> de LATTICE-uitvoer, en `ExifImageWidth`/`Height` zijn uitgesloten omdat ze de bronopname
> beschrijven in plaats van het geëxporteerde raster.

Subvlaggen voor de opnamefase (van toepassing op de radiometrische niveaus — `radiance`, `reflectance`, `display`):

| Vlag | Standaard | Betekenis |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST-radiometrische schaal. |
| `apply_white_balance` | `True` | WB LUT. DLS-bewust wanneer een DAQ aan de camera is gekoppeld. |
| `apply_index` | `False` | Evaluatie van vegetatie-index. |
| `index_expression` | `None` | Formule overschrijven. Niet-leeg → automatisch-schakelt index in. |
| `annotated` | `False` | GUI-decoraties (zebra/raster/peaking) overlappen. Niet beschikbaar voor `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Het retourtype is `CapturePathMap`, niet `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` is `Dict[str, Union[str, List[str]]]`: een enkelvoudig
> `processing` wijst aan elke seriële een pad toe, terwijl een meerlaagse (`"all"`, of een
> expliciete `levels`-lijst) het de **geordende lijst** van elk product dat voor die
> camera is opgeslagen. Een live samengestelde compositie, mocht er een worden gestreamd, komt binnen onder de extra
> `"combined"`-sleutel in plaats van onder een serienummer. Code die uitgaat van `str` loopt vast op de
> lijstvorm zonder dat een typecontrole bezwaar maakt — de annotatie vermeldde `Dict[str, str]`
> nog een tijdje nadat de lijstvorm werd geïntroduceerd, en daarom bestaat de alias. Normaliseer
> wanneer je de platte vorm wilt:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Array-uitlijning

`ArrayHandle` maakt het volledige uitlijningsoppervlak zichtbaar. Profielen zijn standaard alleen sessiegebonden — roep `export_alignment()` expliciet aan om ze permanent op te slaan.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Uitlijning bij het verbinden

`connect_all(align=...)` kan elke array automatisch uitlijnen bij het verbinden:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Valt terug op `project.json["config"]["auto_align_on_connect"]` indien niet gespecificeerd.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Directe hardware (zonder backend)

Als je volledig onafhankelijk wilt zijn van de backend (CI, headless robots, embedded), importeer dan `lattice_sdk` en `daq_sdk` rechtstreeks — beide worden opnieuw geëxporteerd door `chloros_sdk`. Let op bij `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` zit in het PyPI-pakket (maar vereist de aanwezigheid van de Arena-SDK-runtime), terwijl `daq_sdk` alleen bij de desktopinstallatie wordt meegeleverd.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Voorinstellingen en de trigger

Drie van de vier voorinstellingen zijn **free-run**: de camera belicht continu en
`capture()` retourneert het volgende frame. `triggered` is de uitzondering — deze zet de
camera in de stand-by voor een hardware-flank op lijn 2, dus er wordt niets vastgelegd totdat er een binnenkomt.

| Voorinstelling | Trigger | Gebruik deze wanneer |
| --- | --- | --- |
| `default` | free-run | algemeen gebruik |
| `high_speed` | free-run | 8-bit, maximaal 60 fps, korte belichtingstijd |
| `high_quality` | free-run | 12-bit, geen fps-limiet — de gebruikelijke keuze voor foto’s |
| `triggered` | **geactiveerd, lijn 2** | de camera is aangesloten op een M8-synchronisatiekabel en wordt door iets anders geactiveerd |

Als je `triggered` (of zelf `trigger_mode="On"` instelt) zonder dat er iets
Lijn 2 aanstuurt, zal elke `capture()` een time-out geven — terecht, aangezien je
de camera hebt gevraagd om te wachten. De SDK legt uit wat er gebeurt als dit gebeurt; zie
[SC_ERR_TIMEOUT tijdens opname](#direct-hardware-backend-free).

> **Opmerking — &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011`-berichten bij het verbinden zijn geen fouten.**&gt; Bij het verbinden probeert de SDK te onderhandelen over**jumboframes** (GVSP-pakketten van 9000 bytes) voor een hogere doorvoersnelheid. Op een directe point-naar-punt-NIC-verbinding (bijv. een link-local `169.254.x.x`-adres) kan het netwerk doorgaans geen jumboframes verwerken, waardoor deze probe een time-out krijgt en regels zoals de volgende registreert:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Dit is de **beoogde terugvalregeling**: de SDK schakelt automatisch terug naar standaardpakketten van 1500 bytes en de camera maakt de verbinding gewoon af (de daaropvolgende `[chunk-enable …]`-regels maken deel uit van de normale verbindingsprocedure). Het vastleggen werkt nog steeds.
>
> Je kunt deze test overslaan, maar **het is niet alleen bedoeld om de logboekuitvoer te onderdrukken — het schakelt jumboframes uit.** De camera beantwoordt ‘Don’t-Fragment’-pings slechts tot 1500 bytes, hoe goed je netwerk ook is, dus de pingtest alleen kan jumbo nooit opsporen; alleen deze probe kan dat. Schakel het uit en de camera blijft voor altijd standaardpakketten van 1500 bytes verzenden, op elk netwerk:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Alleen de moeite waard op een netwerk waarvan je *weet* dat het geen jumbo-pakketten kan verwerken, waar het ongeveer een seconde verbindingstijd per camera bespaart. Aangezien het om een echte afweging gaat en niet om een cosmetische aanpassing, vermeldt de `SDK` dit nu ook wanneer je het gebruikt:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Laat het met rust, tenzij je een reden hebt.** Als deze ingeschakeld blijft, wordt bij elke verbinding opnieuw gemeten welk netwerk je daadwerkelijk hebt: sluit aan op een switch die jumbo-pakketten ondersteunt en de volgende verbinding pikt jumbo vanzelf op, zonder dat je iets hoeft te configureren en zonder herstart.
>
> Als je de jumbo-doorvoer *wilt*, schakel dan jumbo end-to-end in (NIC MTU 9000 + een switch die deze doorlaat), of vergrendel het met `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` als je weet dat de verbinding dit ondersteunt — geef echter de voorkeur aan een `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` per opdracht boven een permanente instelling, aangezien een vergrendelde grootte de peiling overslaat en niet meer aanpast aan het netwerk ervoor. **Elk** apparaat in het pad moet jumbo-pakketten doorlaten — inclusief eventuele PoE-splitters of -injectors, wat de gebruikelijke reden is waarom een anderszins voor jumbo geschikte opstelling deze niet kan verwerken.

> **`SC_ERR_TIMEOUT -1011` tijdens `capture()` / `grab*()` is een ander probleem — dat is een echte fout.**&gt; De bovenstaande opmerking heeft alleen betrekking op `-1011` die door de**connect-time probe**is geregistreerd. Dezelfde foutmelding die bij een**capture** wordt gegenereerd, betekent dat de camera wel degelijk is aangesloten, maar geen beelden verstuurt:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> De doorslaggevende aanwijzing is een camera waarvan het *besturings*kanaal in orde is — detectie werkt, instellingen en `[chunk-enable …]`-schrijfbewerkingen slagen allemaal — terwijl *elk* frame een time-out krijgt.
>
> **De gebruikelijke oorzaak is dat de camera is ingesteld op een hardware-trigger.** Bij `trigger_mode="On"` en `trigger_source="Line2"` zendt de camera helemaal niets uit totdat er een elektrische flank op de M8-synchronisatiekabel aankomt. Als er geen kabel is die die lijn aanstuurt, duurt elke opname oneindig lang. De camera is niet defect en het netwerk werkt prima — hij doet precies wat hem is opgedragen.
>
> `CameraSettings()` en de `default` / `high_speed` / `high_quality` zijn ingesteld op free-run, en een opname die in de time-out raakt terwijl het systeem is geactiveerd, geeft een uitleg in plaats van alleen ‘`-1011`’ weer te geven. `PRESETS["triggered"]` activeert Line2, zoals bedoeld.
>
> Om een camera te dwingen in de free-run-modus te werken:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Als er met `trigger_mode="Off"` nog steeds een time-out optreedt, levert de camera echt geen gegevens — stuur ons het logbestand en `ip link show`.

#### Kleurprofielen (live preview van RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` kiest het kleurprofiel voor de **live preview** op RGB-camera’s (multispec-camera’s negeren deze instelling):

| Profiel | Betekenis |
| --- | --- |
| `raw` | De radiometrische keten volledig omzeilen. |
| `linear` | DSNU + flat + WB, geen CCM, geen gamma. |
| `natural` | Lineair + gemeten CCM + sRGB-gamma, met alleen de eenvoudige afwerking (chroma-afvlakking + desaturatie van hoge lichten) — de realistische standaardinstelling. |
| `enhanced` | `natural` plus de volledige hub-parity-afwerking (defringe, vibrance, CLAHE lokaal contrast). Rijkere weergave tegen ongeveer **het dubbele van de afwerkingskosten per frame**, dus een lagere LIVE-framesnelheid. |
| `custom_temp` | `natural` maar witbalans vastgezet op `custom_cct_k` Kelvin (DLS genegeerd; begrensd op 2000–10.000 K aan backend-zijde). |

Het profiel is een **alleen-voor-live-preview**-regelaar voor snelheid en uiterlijk: opgeslagen opnames krijgen altijd de volledige, rijke afwerking, ongeacht het geselecteerde profiel, dus het kiezen van `natural` om frametijd terug te winnen, verlaagt de kwaliteit van wat op de schijf terechtkomt niet. Een onbekend profiel verhoogt `ValueError`; wanneer een chloros-backend bereikbaar is, wordt de wijziging ook daarheen gepost, zodat het volgende voorbeeldframe dit weergeeft (gebruikers van direct-SDKs zonder backend krijgen de instellingswijziging nog steeds te zien).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M)-camera’s en `Calibration`

Een mono **M3M**-camera (`M3M-<lens>-F<wavelength>`) is enkelbandig: één grijswaardenvlak, geen Bayer-mozaïek, geen 3×3 spectrale-crosstalkmatrix. `Calibration` herkent dit en stelt een `is_mono`-vlag beschikbaar. Reflectantie is nog steeds van toepassing als eenband-radiometrische kaart (de ontmenging is de identiteitsmatrix), maar multibandberekeningen op één enkele camera leveren zinvolle resultaten op in plaats van onzin:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Om een vegetatie-index op te stellen op basis van monochrome hardware, combineer je meerdere M3M-camera&#x27;s bij verschillende golflengten tot een uitgelijnde multiband-stack-bandstapel (zie [Array-uitlijning](#array-alignment)) en bereken de index over die stapel in plaats van op één camera.

DAQ-directe modus:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` geaccepteerde sleutels**— precies `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; afgeschaft ten gunste van `cap_id`), `filter_model` (DAQ-M) en `cap_id` (alle DAQ-types; `None`/`""`/`"none"` = kale sensor, geen afdekkingcorrectie). Onbekende sleutels worden**stilletjes genegeerd** — bijv. `{"integration_time": 64}` doet niets (het moet `integration_time_ms` zijn). Geeft `{"applied": [...], "errors": {...}}` terug en genereert nooit een foutmelding.

`chloros_sdk` exporteert alleen het hierboven gebruikte kernoppervlak opnieuw. De volledige openbare `daq_sdk`-APIatie (22 namen) voegt het volgende toe — importeer deze rechtstreeks vanuit `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Uitzonderingen

Vang de basisklasse op om „alles wChlorost misging“ af te handelen:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` en `ChlorosConfigurationError` worden op het hoogste niveau geëxporteerd, samen met de rest; ze kunnen ook worden geïmporteerd vanuit `chloros_sdk.exceptions`, zoals weergegeven.

Hiërarchie:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## End-to-End-voorbeelden

### 1. Een map verwerken met een aangepaste voortgangsbalk

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live LATTICE-array → Reflectantie + DAQ-referentie

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projectgestuurde opnamecampagne

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Frame-stream met meerdere camera’s → NumPy-pijplijn

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Headless opnamescript rechtstreeks op de hardware (zonder backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Capaciteitstest vóór het aansluiten van een 4-camera-array

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Equivalent van opname-recept (pure Python)

De recept-DSL van de CLI heeft een direct Python-equivalent:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatische start van de backend

De smart-connect-toegangspunten — `connect_camera`, `connect_array`, `connect_daq_sensor` en `discover_lattice_cameras` — zijn thin-HTTP-clients die ervan uitgaan dat er een backend luistert op `127.0.0.1:5000` (de standaardURL van de Smart-Connect-interface). Wanneer de GUI of CLI al actief is, is er al een. Vanuit een kaal script is dat misschien niet het geval — dus starten deze functies **het meegeleverde backend-binaire bestand automatisch** (zonder venster, net zoals `ChlorosLocal` dat doet) voordat ze voor het eerst worden aangeroepen, en wachten vervolgens tot `backend_startup_timeout` totdat het opstart.

Regels:

- **Er wordt altijd alleen een lokale URL gestart.** Een `backend_url` die verwijst naar `localhost` / `127.0.0.1` / `[::1]` komt in aanmerking; elke andere host wordt beschouwd als de machine van iemand andersen wordt nooit gestart.
- **De backend blijft draaien voor hergebruik** (net als bij de CLI) — er vindt geen impliciete afsluiting plaats wanneer je script wordt afgesloten. Als je het script opnieuw uitvoert, wordt de actieve backend hergebruikt.
- **Schakel dit uit met `auto_start_backend=False`** bij elk van deze aanroepen (bijvoorbeeld wanneer je naar een externe backend verwijst, of wanneer je de levenscyclus van de backend zelf beheert).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Als het meegeleverde binaire bestand niet kan worden gevonden of gestart, genereert de daaropvolgende aanroep HTTP een bruikbare, **platformspecifieke** `ChlorosConnectError` in plaats van een gewone trace voor een geweigerde verbinding — op Windows verwijst deze je naar de desktop-app of een `chloros-cli`-opdracht; op Linux (geen GUI) wordt u verwezen naar een `chloros-cli`-commando of de `.deb`.

---

## Omgeving &amp; Headers

De SDK markeert elke backend-HTTP-aanroep met `X-Chloros-Client: sdk`. De backend past de licentieregels van SDK / CLI toe (inloggen **en** een betaald Chloros+-abonnement vereist) in plaats van het gratis GUI-traject. Dit wordt automatisch ingesteld op het moment van importeren — je hoeft niets te doen.

`http://localhost` en `http://127.0.0.1` worden gedetecteerd als de lokale backend. Oproepen naar andere hosts (bijvoorbeeld je eigen analyseservice) blijven ongewijzigd.

Overschrijf de backend-URL door `backend_url=` (of `api_url=` op `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Een niet-loopback `backend_url` bereikt alleen een source/dev-backend — meegeleverde backends binden alleen loopback; zie Remote-Backend Mode voor het tunnelpatroon.)

---

## Versiebeheer en compatibiliteit

- De versie SDK wordt weergegeven als `chloros_sdk.__version__`.
- De SDK koppelt het gedrag aan de meegeleverde backend-versie. Het combineren van een oudere SDK met een nieuwere backend werkt doorgaans (voorwaarts compatibele eindpunten), maar het combineren van een nieuwere SDK met een oudere backend kan leiden tot `404`-fouten op nieuwe eindpunten — upgrade de desktop-app zodat deze overeenkomt.
- De smart-connect-interface (`connect_camera` / `connect_array` / `connect_daq_sensor`) en het eindpunt voor netwerkanalyse retourneren stabiele JSON-schema’s; nieuwe velden zijn aanvullend.

---

## Tips voor het oplossen van problemen

- **`ChlorosAuthenticationError: Login required`** → Voer `chloros-cli login EMAIL PASSWORD` één keer uit op deze machine, of meld je aan via de desktop-app Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → De smart-connect-aanroepen starten automatisch een lokale backend, dus deze foutmelding verschijnt alleen wanneer het meegeleverde binaire bestand niet kan worden gevonden of gestart (bijvoorbeeld op een host met alleen pip en zonder desktop-pakket). De foutmelding is platformafhankelijk: op Windows open je de desktop-app of voer je een willekeurige `chloros-cli`-opdracht uit; op Linux voer je een `chloros-cli`-opdracht uit (er is geen GUI) of installeer je de `.deb`. Voor een externe backend geef je `backend_url=` (en `auto_start_backend=False`) doorgeven.
- **`CAMERA_AVAILABLE == False`** bij het importeren → `lattice_sdk` kon niet worden geladen (meestal zijn de runtime-DLL’s van de Arena-SDK-backend niet geïnstalleerd). Het niet-camera-oppervlak werkt nog steeds.
- **Array connect levert een resolutie lager dan native op**→ De smart-prep van de backend verkleint automatisch de framegrootte zodat deze in de wire past. Gebruik `analyze_array_network()` om te zien waarom, en upgrade vervolgens de link, accepteer de verkleining of geef `force_tier="slip-emit-and-capture"` door voor sequentiële opname. Het veiligheidsnet voor het verkleinen dekt**geen** geaggregeerde over-subscription (`oversubscribed: true`, fps-velden 0): te veel camera’s voor de verbinding kunnen niet worden opgelost door binning/ROI — verminder het aantal camera’s, schakel jumboframes in of stap over op een snellere NIC (zie [Over-Subscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` meldt dat de NIC RX-ring erg klein is (~0,26 MB) / verbindingspoorten met „FRAMES WILL DROP”** → De ontvangstring van de host-NIC staat op de standaardwaarde (wordt vaak teruggezet naar 32 na een update van het NIC-stuurprogramma). Stel op een Realtek USB 10GbE-adapter `ReceiveBufferLen=256` en `PendingReceives=64` (met verhoogde rechten) in en start vervolgens de back-end opnieuw opzodat de ring opnieuw wordt ingelezen. Volledige procedure: [CLI Referentie → Host-NIC instellen en afstemmen](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host loopt vast bij herstarten/afsluiten, later WMI-fouten `Invalid class` / NIC kan niet worden ingeschakeld** → Verouderd USB 10GbE-stuurprogramma dat `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`) veroorzaakt. Werk het adapterstuurprogramma bij naar een actuele versie (≥ 2026) en pas de instellingen voor de ontvangstring opnieuw toe. Zie [CLI Referentie → Host-NIC instellen en afstemmen](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflectie geweigerd** → Voor reflectie op absolute schaal moet een actieve DAQ aan de camera (of array) worden gekoppeld. Koppel deze via de GUI of gebruik `processing="radiance"` (W/m²/sr/nm), waarvoor geen gekoppelde sensor nodig is.
- **`smart=True`-opname duurt langer dan verwacht** → AE-convergentie hangt af van de dynamiek van de scène; verruim `exposure_tolerance_pct` of verkort `stability_window_s` als je een snellere (minder stabiele) trigger wilt.

---

## Zie ook

- [CLI Referentie](cli-reference.md) — elk subcommando van CLI komt overeen met een SDK-aanroep.
- [DAQ-sensorgids](../daq/README.md) — sensorspecifieke bedradings-, kalibratie- en registratieregels.
- Online documentatie: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
