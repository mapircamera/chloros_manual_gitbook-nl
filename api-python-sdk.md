# API: Python SDK

{% hint style="info" %}
**Op zoek naar de volledige API?** Deze pagina is een praktische handleiding. Alle openbare klassen, methoden, exacte handtekeningen en kopieer-en-plakbare voorbeelden staan in de [SDK-referentie](reference/sdk-reference.md), die is geoptimaliseerd voor AI-assistenten.**Werk je met een AI-assistent?** Plak deze URL in de chat, zodat deze de volledige, actuele Chloros 1.3.0 API van URL bevat:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Elke pagina van deze handleiding is beschikbaar als onbewerkte markdown onder de kleine letters van de slug + `.md`, en de volledige handleiding is geïndexeerd op `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

De **Chloros Python SDK** (`chloros-sdk` op PyPI) stuurt alles aan wat de desktop-app vanuit Python kan doen: batchverwerking van afbeeldingen, live besturing van LATTICE-camera’s en -arrays, DAQ-sessies met lichtsensoren en automatisering van opgeslagen projecten. Het is een dunne laag bovenop dezelfde lokale backend die de GUI en CLI gebruiken (HTTP op `127.0.0.1:5000`), dus het gedrag is identiek op alle drie de interfaces.

## Installeren

De SDK wordt meegeleverd met het Chloros-desktop pakket, dat ook de verwerkingsbackend en hardware-runtimes bevat, en altijd qua versie overeenkomt.

**Stap 1 — Installeer Chloros.** Windows: voer het desktopinstallatieprogramma (standaardpad `C:\Program Files\Chloros\`) uit vanaf de [Download](download.md)-pagina. Linux: installeer het `.deb`-pakket met `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Linux-installatie](linux/linux-installation.md)).**Stap 2 — Controleer de SDK** (Python 3.7+). Meestal hoeft er niets te worden geïnstalleerd:

* **Windows:** het installatieprogramma installeert het meegeleverde SDK-wheel in de Python van je systeem (als er geen Python is geïnstalleerd, wordt deze stap overgeslagen).
* **Linux:** het `.deb` installeert de SDK voor het systeem `python3` in `/usr/lib/python3/dist-packages`. De vereisten, `python3-numpy` en `python3-requests`, worden door het pakket aanbevolen: `apt install` bevat ze, `dpkg -i` niet.
* **Een virtualenv** heeft geen toegang tot de pakketten van het systeem; installeer het meegeleverde wheel daarin: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` op Linux, of `chloros_sdk-*.whl` in de map `sdk` van de Windows-installatiemap (standaard `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Op een machine zonder Chloros-pakket, waarvan de SDK een backend op een andere machine aanstuurt (zie [Wat een pip-only-installatie oplevert](#what-a-pip-only-install-gets)), installeer je het via PyPI: `pip install chloros-sdk`. PyPI wordt bij elke release-build bijgewerkt, zodat het overeenkomt met de nieuwste stabiele release.

**Stap 3 — Log één keer per machine in:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Inloggegevens worden opgeslagen in `~/.chloros/` (beide platforms). Op Windows kun je op dezelfde manier inloggen via het tabblad ‘Gebruiker’ <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> van de desktop-app. Voor de SDK is een betaald Chloros+-abonnement vereist — zie [Licentievereisten](#license-requirement) hieronder.

| Vereiste | Details |
| --- | --- |
| **Chloros geïnstalleerd** | Windows: desktop-installatieprogramma; Linux: `.deb`-pakket (bevat het backend-uitvoeringsbestand en de SDK) |
| **Python** | 3.7 of hoger (ontwikkeld/getest op 3.10) |
| **Besturingssysteem** | Windows 10/11 64-bit, Ubuntu 22.04 LTS of nieuwer, of NVIDIA Jetson (JetPack 6) |
| **Licentie** | Actieve Chloros+-login, elk betaald abonnement (Copper of hoger) |

## Klaar in 60 seconden

Met één aanroep maak je een project aan, importeer je een map, configureer je de verwerking en voer je de pijplijn uit — waarbij de backend automatisch wordt gestart als deze nog niet draait:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Gebruik op Linux Linux-paden: `/home/user/drone_images/flight001`. De SDK werkt op beide platforms op identieke wijze.)

Een LATTICE-opnamemap verwerken? Gebruik dan de LATTICE-vriendelijke wrapper — deze past de juiste standaardinstellingen toe (geen detectie van het doelpaneel, standaard debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — volledige controle over de pijplijn

Gebruik voor alles wat meer is dan een éénregelige opdracht `ChlorosLocal`. Dit maakt gebruik van een backend die al draait (die van de desktop-app), of start de geïnstalleerde backend bij het eerste gebruik (`auto_start_backend=True`; op Windows vanuit de map `backend` van de Chloros-installatiemap, standaard `C:\Program Files\Chloros\backend\`), maakt en configureert projecten, houdt de voortgang bij en geeft na afloop een samenvatting weer.

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
Gebruik de standaard `http://127.0.0.1:5000` in plaats van deze te vervangen door `localhost` — onder Windows wordt `localhost` eerst omgezet naar `::1` en kost dit ongeveer 2 seconden per verzoek bij de backend die alleen IPv4 ondersteunt.
{% endhint %}

Gebruik het als een contextmanager. Het verlaten van het blok beëindigt het gebruik van de backend door deze instantie, niet de backend zelf: deze blijft draaien voor het volgende script en stopt vanzelf na 30 minuten inactiviteit. `shutdown_backend()` stopt een instantie die is gestart.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` accepteert de volgende trefwoorden: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` en `custom_settings`. De belangrijkste waarden:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

De LATTICE-specifieke instellingen (`input_level`, `radiometric_output`, de `array_alignment*`-familie) worden gedocumenteerd met hun volledige waardetabellen in de [SDK-referentie](reference/sdk-reference.md#supported-values).

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

### Het overzicht na afloop van de run lezen — en lege runs opsporen

Na voltooiing voegt `process()` het verwerkingsoverzicht van de backend toe als `result["summary"]`. Elk item in `summary["hints"]` is een volledige zin waarin opmerkelijke zaken worden uitgelegd — bijvoorbeeld waarom een run geen output heeft opgeleverd — en elke hint wordt ook opnieuw uitgezonden als een Python-`UserWarning`, zodat lege runs zichzelf diagnosticeren, zelfs als je het dict nooit inspecteert:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` wordt niet geactiveerd wanneer een run geen afbeeldingen oplevert.** Dit is het enige punt waarop de SDK en de CLI opzettelijk van elkaar verschillen: `chloros-cli process` beschouwt „producten werden aangevraagd, maar er werden geen geschreven“ als een fout en stopt met een waarde anders dan nul, terwijl de SDK normaal terugkeert en de toestand rapporteert via `summary` / hints. Als uw pijplijn bij een lege run moet stoppen, controleer dit dan zelf — inspecteer `summary` (of tel de bestanden in de projectmap) in plaats van te vertrouwen op een uitzondering.
{% endhint %}

## Smart Connect — live hardware

Drie helpers openen permanente sessies in de hardwarepool van de backend — dezelfde pool die de CLI, andere scripts en de cameratabbladen van de GUI gebruiken, zodat SDK-scripts naast deze kunnen bestaan zonder te strijden om seriële poorten of netwerkbandbreedte. Alle drie starten ze automatisch een lokale backend als er geen actief is. Het DAQ-paneel van de GUI maakt geen deel uit van de pool: sluit een lichtsensor in de GUI voordat je deze vanuit een script opent.

### Enkele LATTICE-camera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relatieve paden verwijzen naar de locatie waar je script wordt uitgevoerd (gewijzigd in 1.3.0).** Met een lokale backend is een relatief `output_dir` (inclusief de standaard `"output"`) — en de andere paden die de backend opent (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — worden omgezet naar de werkdirectory van uw script, zodat `cam.capture("output/")` naar `<current folder>/output/` schrijft. In versie 1.2.0 loste een geïnstalleerde backend deze op in zijn eigen installatiemap en gaf een foutmelding „Toegang geweigerd“. Bij een backend op afstand worden paden verzonden zoals ze zijn ingevoerd.
{% endhint %}

### Gesynchroniseerde array — `connect_array`

`connect_array` is het aanbevolen startpunt voor opstellingen met meerdere camera's. Het voert dezelfde smart-prep-stroom uit als de GUI: netwerkanalyse, automatische selectie van synchronisatieniveau, PTP-tijdsynchronisatie, selectie van pixelformaat per camera, AE-seeding en het activeren van de GPIO-trigger. De **eerste seriële poort is de master** (deze activeert de hardware-triggerpuls); de overige zijn slaves.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Voeg `smart=True` toe aan elke array-opname om te wachten tot de automatische belichting voor alle camera’s is gestabiliseerd voordat je de trigger activeert.

Nieuw in versie 1.3.0: een `ArraySession` levert je ook direct gesynchroniseerde frames, zonder bestanden te schrijven:

* `arr.grab()` retourneert één frame per camera, allemaal afkomstig van één trigger, als numpy-arrays (`{serial: array}`); `.group_id` is de gedeelde trigger en `.errors` benoemt elke camera die buiten beschouwing is gelaten.
* `arr.frame_stream(fps=5.0)` levert één dergelijke set per tick op, voor liveverwerking in je eigen pijplijn.
* `arr.continuous("output/")` voert de eigen continue opname van de backend uit (de modus ‘Continuous’ van de GUI) en retourneert een `ContinuousHandle` met `status()`, `wait()` en `stop()` — gebruik dit in plaats van een opnamelus wanneer elke trigger met een hoge frequentie moet worden opgeslagen.

Zie de [SDK-referentie](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep) voor opnamemodi (Enkel / Continu / Interval / Snelste), recorders, burst-to-video en array-uitlijning.

### DAQ-lichtsensor — `connect_daq_sensor`

Zonder argumenten detecteert `connect_daq_sensor()` automatisch het transportprotocol (prioriteit: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Elk frame bevat de 135-punts `spectrum` (W/m²/nm na kalibratie), een `is_saturated`-vlag en CIE `x`, `y`, `z`. Om een specifieke sensor of transportmethode vast te leggen, geef je één expliciete hint door:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Let op: profielen voor cap-correctie (`cap_id`) zijn **geen** SDK-regelaar — selecteer deze in plaats daarvan via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Opgeslagen projecten — `open_project`

Een opgeslagen Chloros-project behoudt de gekoppelde hardware (`cameras.json` + `sensors.json` naast `project.json`), en `chloros_sdk.open_project(path)` kan zijn camera’s, arrays en lichtsensoren in één keer opnieuw verbinden en opnames aansturen op basis van de apparaatnaam. Lichtsensoren maken opnieuw verbinding via een actieve Chloros-backend, met hun opgeslagen instellingen en opnamecapaciteit. Zie [Projectautomatisering](reference/sdk-reference.md#project-automation--chlorosproject) in de referentie.

## Wat een installatie met alleen pip oplevert

Controleer de beschikbaarheidsvlaggen op moduleniveau voordat je hardwareoppervlakken gebruikt:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Op een host met **alleen** `pip install chloros-sdk` en zonder Chloros-desktop pakket:

* `ChlorosLocal`, `process_folder` en `process_lattice_capture` werken **niet** — hiervoor is het backend-binaire bestand nodig dat in het desktop-installatieprogramma wordt meegeleverd.
* De smart-connect-helpers (`connect_camera`, `connect_array`, `connect_daq_sensor`) zijn pure HTTP-clients, dus ze werken met een backend op een andere machine — maar de meegeleverde backends zijn alleen voor loopback, dus je moet de poort zelf doorsturen (bijv. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) en `backend_url="http://127.0.0.1:5000"` samen met `auto_start_backend=False` doorgeven. Zie [Remote-Backend-modus](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* De LATTICE-klassen voor directe hardware (`LatticeCamera`, `CameraPool`, …) kunnen worden geïmporteerd, maar hebben de Arena SDK-runtime uit het desktop-pakket nodig — zonder deze is `CAMERA_AVAILABLE` gelijk aan `False`.
* DAQ-sensoren worden uitsluitend aangestuurd via een (getunnelde) backend — `connect_daq_sensor()`, of de `proj.sensors` van een project; de backend opent de sensor op zijn eigen machine.

## AI-assistenten — `chloros-mcp`

De SDK bevat `chloros-mcp`, een MCP-server waarmee Claude Desktop, Claude Code, Cursor en andere MCP-clients Chloros kunnen aansturen: beelden verwerken en LATTICE-camera’s, arrays en DAQ-lichtsensoren bedienen. De tools ervan omvatten alles wat deze SDK doet. Het wordt samen met de SDK geïnstalleerd, dus voor Claude Code is de installatie slechts één regel:

```bash
claude mcp add chloros -- chloros-mcp
```

Andere clients: [Chloros gebruiken met AI-assistenten](ai-assistants.md). Opties en alle tools: [MCP-server (AI-assistenten)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Licentievereisten

Voor toegang tot de SDK is een actieve Chloros+-login vereist op een betaald abonnement — **Copper of hoger**(Copper / Bronze / Silver / Gold); het gratis Iron-abonnement biedt geen toegang tot de SDK of CLI. De handhaving vindt**aan de serverzijde** plaats: elk SDK-verzoek moet zowel een actieve sessie als een betaald abonnement bevatten, anders retourneert de backend `403` / `PLAN_UPGRADE_REQUIRED` (aangeroepen als `ChlorosLicenseError` door `ChlorosLocal`, en als `ChlorosConnectError` door de `connect_*`-helpers). Een uitgelogde aanroeper krijgt in plaats daarvan `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — het opnieuw uitvoeren van `chloros-cli login` lost het eerste geval op, maar niet het tweede.

Offline gebruik werkt binnen de respijtperiode van het abonnement: het abonnementstype wordt gelezen uit de cache voor servervalidatie (5 minuten) of de cache voor ondertekende, aan de machine gebonden licenties (30 dagen voor maandabonnementen; tot het verstrijken van het jaarabonnement). Wanneer de respijtperiode verstrijkt, wordt het abonnement omgezet naar de gratis versie en wordt de SDK-toegang stopgezet totdat de machine één keer verbinding maakt met de server. `chloros-cli status` blijft bereikbaar op het gratis niveau, zodat de reden altijd zichtbaar is. Zie [Chloros+ Login](chloros+-login.md).

## Uitzonderingen

Vang de basisklasse op om „alles wat er mis is gegaan met Chloros“ af te handelen:

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
* [Chloros gebruiken met AI-assistenten](ai-assistants.md) — `chloros-mcp`-installatie- en promptvoorschriften.
* [Download](download.md) — installatieprogramma’s voor Windows en Linux.
