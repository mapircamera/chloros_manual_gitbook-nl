# API : Python SDK

{% hint style="info" %}
**Op zoek naar de volledige API?** Deze pagina is een praktische handleiding. Elke openbare klasse, methode, exacte handtekening en kopieer-en-plakbaar voorbeeld staat in de [SDK-referentie](reference/sdk-reference.md), die is geoptimaliseerd voor AI-assistenten.**Werk je met een AI-assistent?** Plak deze URL in de chat, zodat deze de volledige, actuele Chloros 1.2.0 API bevat:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Elke pagina van deze handleiding is beschikbaar als onbewerkte markdown onder de kleine letters van de slug + `.md`, en de volledige handleiding is geïndexeerd op `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

De **Chloros Python SDK** (`chloros-sdk` op PyPI) stuurt alles aan wat de desktop-app kan doen vanaf Python: batchverwerking van afbeeldingen, live besturing van LATTICE-camera’s en -arrays, DAQ-sessies met lichtsensoren en automatisering van opgeslagen projecten. Het is een dunne laag bovenop dezelfde lokale backend die de GUI en CLI gebruiken (HTTP op `127.0.0.1:5000`), dus het gedrag is identiek op alle drie de platforms.

## Installeren

De installatie bestaat uit twee stappen: eerst het Chloros-desktop pakket (dit biedt de verwerkingsbackend en hardwareruntimes), daarna het Python-pakket.

**Stap 1 — Installeer Chloros.** Windows: voer het desktopinstallatieprogramma (standaardpad `C:\Program Files\MAPIR\Chloros\`) uit vanaf de [Download](download.md)-pagina. Linux: installeer het `.deb`-pakket ([Linux-installatie](linux/linux-installation.md)).**Stap 2 — Installeer SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Je hebt pip misschien niet eens nodig: elke installatieprogramma bevat een bijpassend SDK-wheel. Het Windows-installatieprogramma installeert dit automatisch in je systeem Python; de Linux `.deb` plaatst het in `/usr/lib/chloros/sdk/` en geeft de exacte `pip install --user`-opdracht weer. PyPI wordt bij release-builds bijgewerkt, dus `pip install chloros-sdk` komt overeen met de nieuwste stabiele release.

**Stap 3 — Log één keer per machine in:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Inloggegevens worden opgeslagen in `~/.chloros/` (beide platforms). Op Windows kun je op dezelfde manier inloggen via het tabblad Gebruikers<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">n van de desktop-app. Voor SDK is een betaald Chloros+-abonnement vereist — zie [Licentievereisten](#license-requirement) hieronder.

| Vereiste | Details |
| --- | --- |
| **Chloros geïnstalleerd** | Windows: desktop-installatieprogramma; Linux: `.deb`-pakket (bevat het backend-binaire bestand) |
| **Python** | 3.7 of hoger (ontwikkeld/getest op 3.10) |
| **Besturingssysteem** | Windows 10/11 64-bits, Ubuntu 22.04 LTS of nieuwer, of NVIDIA Jetson (JetPack 6) |
| **Licentie** | Actieve Chloros+-login, elk betaald abonnement (Copper of hoger) |

## De overwinning in 60 seconden

Met één aanroep wordt een project aangemaakt, een map geïmporteerd, de verwerking geconfigureerd en de pijplijn uitgevoerd — waarbij de backend automatisch wordt gestart als deze nog niet draait:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Gebruik op Linux de Linux-paden: `/home/user/drone_images/flight001`. De SDK werkt op beide platforms op identieke wijze.)

Een LATTICE-opnamemap verwerken? Gebruik de LATTICE-vriendelijke wrapper — deze past de juiste standaardinstellingen toe (geen detectie van paneeldoelen, standaard debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — volledige controle over de pijplijn

Gebruik `ChlorosLocal` voor alles wat meer is dan een éénregelig commando. Deze start de backend bij het eerste gebruik (`auto_start_backend=True`), maakt en configureert projecten, houdt de voortgang in de gaten en geeft een samenvatting na afloop.

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

{% hint style="info" %}
Gebruik de standaardinstelling `http://127.0.0.1:5000` in plaats van deze te vervangen door `localhost` — bij Windows wordt `localhost` eerst omgezet naar `::1` en kost dit ongeveer 2 seconden per verzoek bij de backend die alleen IPv4 ondersteunt.
{% endhint %}

Gebruik het als contextmanager voor gegarandeerde opschoning:

```python
import chloros_sdk

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

`configure()` accepteert de volgende trefwoorden: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` en `custom_settings`. De belangrijkste waarden:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

De LATTICE-specifieke regelaars (`input_level`, `radiometric_output`, de `array_alignment*`-familie) worden gedocumenteerd met hun volledige waardetabellen in de [SDK-referentie](reference/sdk-reference.md#supported-values).

### De voortgang volgen

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Het overzicht na afloop lezen — en lege runs opsporen

Na voltooiing voegt `process()` het verwerkingsoverzicht van de backend toe als `result["summary"]`. Elk item in `summary["hints"]` is een volledige zin waarin opmerkelijke zaken worden toegelicht — bijvoorbeeld waarom een run geen uitvoer heeft opgeleverd — en elke hint wordt ook opnieuw verzonden als een Python `UserWarning`, zodat lege runs zichzelf diagnosticeren, zelfs als je het dict nooit inspecteert:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` wordt niet gegenereerd wanneer een run geen afbeeldingen oplevert.** Dit is het enige punt waarop de SDK en de CLI opzettelijk van elkaar verschillen: `chloros-cli process` beschouwt „producten werden aangevraagd, maar er werden geen geschreven” als een fout en stopt met een uitgangssignaal anders dan nul, terwijl de SDK normaal terugkeert en de toestand rapporteert via `summary` / hints. Als uw pijplijn bij een lege run moet stoppen, controleer dit dan zelf — inspecteer `summary` (of tel de bestanden in de projectmap) in plaats van te vertrouwen op een uitzondering.
{% endhint %}

## Smart Connect — live hardware

Drie helpers openen permanente sessies in de hardwarepool van de backend — dezelfde pool die de GUI gebruikt, zodat SDK-scripts naast de desktop-app kunnen bestaan zonder te strijden om seriële poorten of netwerkbandbreedte. Alle drie starten ze automatisch een lokale backend als er geen actief is.

### Enkele LATTICE-camera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Gesynchroniseerde array — `connect_array`

`connect_array` is het aanbevolen startpunt voor opstellingen met meerdere camera’s. Het voert dezelfde ‘smart-prep’-stroom uit als de GUI: netwerkanalyse, automatische selectie van synchronisatieniveau, PTP-tijdsynchronisatie, selectie van pixelformaat per camera, AE-seeding en het activeren van de GPIO-trigger. De **eerste seriële poort is de master** (deze activeert de hardware-triggerpuls); de overige zijn slaves.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Voeg `smart=True` toe aan elke array-opname om te wachten tot de automatische belichting voor alle camera’s is gestabiliseerd voordat de trigger wordt geactiveerd. Zie de [SDK-referentie](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep) voor opnamemodi (Enkel / Continu / Interval / Snelste), recorders, burst-to-video en array-uitlijning.

### DAQ-lichtsensor — `connect_daq_sensor`

Zonder argumenten detecteert `connect_daq_sensor()` automatisch het transportprotocol (prioriteit: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Elk frame bevat de 135-punts `spectrum` (W/m²/nm na kalibratie), een `is_saturated`-vlag en CIE `x`, `y`, `z`. Om een specifieke sensor of transportprotocol vast te leggen — de betrouwbare keuze op hosts met meerdere netwerkinterfaces, waar automatische Ethernet-detectie bij de eerste poging een goed functionerende DAQ-E kan missen — geef je één expliciete hint door:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Let op: profielen voor cap-correctie (`cap_id`) zijn **geen** SDK-regelaar — selecteer ze in plaats daarvan via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Opgeslagen projecten — `open_project`

Een opgeslagen Chloros-project behoudt de aangesloten hardware (`cameras.json` + `sensors.json` naast `project.json`), en `chloros_sdk.open_project(path)` kan alles in één keer opnieuw verbinden en opnames per apparaatnaam uitvoeren. Zie [Projectautomatisering](reference/sdk-reference.md#project-automation--chlorosproject) in de referentie.

## Wat een installatie via pip alleen oplevert

Controleer de beschikbaarheidsvlaggen op moduleniveau voordat je hardwareoppervlakken gebruikt:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Op een host met **alleen** `pip install chloros-sdk` en geen Chloros-desktop pakket:

* `ChlorosLocal`, `process_folder` en `process_lattice_capture` werken **niet** — hiervoor is het backend-binaire bestand nodig dat in het desktop-installatieprogramma is meegeleverd.
* De smart-connect-helpers (`connect_camera`, `connect_array`, `connect_daq_sensor`) zijn pure HTTP-clients, dus ze werken met een backend op een andere machine — maar de meegeleverde backends zijn alleen voor loopback, dus je moet de poort zelf doorsturen (bijv. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) en `backend_url="http://127.0.0.1:5000"` samen met `auto_start_backend=False` doorgeven. Zie [Remote-Backend-modus](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* De LATTICE-klassen voor directe hardware (`LatticeCamera`, `CameraPool`, …) kunnen wel worden geïmporteerd, maar hebben de Arena SDK-runtime uit het desktop-pakket nodig — zonder deze is `CAMERA_AVAILABLE` gelijk aan `False`.
* `daq_sdk` (de directe DAQ-klassen) wordt meegeleverd met de desktopinstallatie, niet met het PyPI-pakket, dus `DAQ_AVAILABLE` is `False` op een host die alleen pip gebruikt — stuur DAQ-sensoren in plaats daarvan aan via `connect_daq_sensor()` tegen een (getunnelde) backend.

## Licentievereiste

Toegang tot SDK vereist een actieve Chloros+-login op een betaald abonnement — **Copper of hoger**(Copper / Bronze / Silver / Gold); het gratis Iron-abonnement biedt geen toegang tot SDK/CLI. De handhaving vindt**server-side** plaats: elk SDK-verzoek moet zowel een actieve sessie als een betaald abonnement bevatten, anders retourneert de backend `403` / `PLAN_UPGRADE_REQUIRED` (gegenereerd als `ChlorosLicenseError` door `ChlorosLocal`, en als `ChlorosConnectError` door de `connect_*`-helpers). Een uitgelogde aanroeper krijgt in plaats daarvan `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — het opnieuw uitvoeren van `chloros-cli login` lost het eerste geval op, maar niet het tweede.

Offline gebruik werkt binnen de respijtperiode van het abonnement: het niveau wordt gelezen uit de cache voor servervalidatie (5 minuten) of de cache met ondertekende, aan de machine gebonden licenties (30 dagen voor maandabonnementen; tot het verstrijken van het jaarabonnement). Wanneer de respijtperiode verstrijkt, wordt het abonnement omgezet naar de gratis versie en wordt de toegang via SDK geblokkeerd totdat de machine eenmaal verbinding heeft gemaakt met de server. `chloros-cli status` blijft bereikbaar op het gratis niveau, zodat de reden altijd zichtbaar is. Zie [Chloros+ Inloggen](chloros+-login.md).

## Uitzonderingen

Vang de basisklasse op om &quot;alles wat er mis is gegaan met Chloros&quot; af te handelen:

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

Alle pijplijnuitzonderingen (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) zijn afgeleid van `ChlorosError`. Een valkuil: `ChlorosConnectError` — wordt alleen gegenereerd door `connect_camera` / `connect_array` / `connect_daq_sensor` — is afgeleid van de gewone `Exception`, **niet** van `ChlorosError`, dus `except ChlorosError` zal deze fout niet opvangen. De volledige hiërarchie staat in de [SDK-referentie](reference/sdk-reference.md#exceptions).

## Zie ook

* [SDK-referentie](reference/sdk-reference.md) — het volledige API-oppervlak, geoptimaliseerd voor AI-assistenten.
* [CLI-referentie](reference/cli-reference.md) — elke CLI-subopdracht komt overeen met een SDK-aanroep.
* [Download](download.md) — installatieprogramma&#x27;s voor Windows en Linux.
