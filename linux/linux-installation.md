# Installatie van Linux

Chloros wordt voor Linux gedistribueerd als `.deb`-pakketten die de CLI en de backend-server installeren. De Python SDK is een afzonderlijk pip-pakket (ook gebundeld in de `.deb` als een wheel met dezelfde versie).

De bestandsnamen van de pakketten bevatten de versie en de architectuur: `chloros_1.2.0_amd64.deb` voor x86_64 en `chloros_1.2.0_arm64_jp6.deb` voor JetPack 6 Jetson-builds. Vervang in de onderstaande commando’s het bestand door het bestand dat u daadwerkelijk hebt gedownload.

***

## Linux amd64 (x86_64)

### Systeemvereisten

| Vereiste | Minimaal | Aanbevolen |
| --- | --- | --- |
| **Distributie** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processor** | x86_64 (Intel/AMD) | Intel Core i7 of beter |
| **Geheugen (RAM)** | 8 GB | 16 GB of meer |
| **Grafische kaart** | Geen (verwerking via CPU) | NVIDIA GPU met 4 GB+ VRAM (12 GB+ ontgrendelt `GPU_PARALLEL`, 7 GB+ houdt Texture Aware uitgeschakeld voor het single-image-pad) |
| **Opslag** | 2 GB vrije ruimte | SSD met 10 GB+ vrije ruimte |
| **Python** | Python 3.7+ (voor de SDK) | Python 3.10+ |

> **Ubuntu 20.04 en Debian 11 worden niet ondersteund.** De lijst met afhankelijkheden van de `.deb` is
> afgeleid van de bibliotheken waaraan de Chloros-backend daadwerkelijk koppelt, en dat omvat
> `libc6 (>= 2.34)`. Zowel Focal als Bullseye worden geleverd met glibc 2.31, dus `apt` weigert de
> installatie direct, in plaats van deze later tijdens de uitvoering te laten mislukken.

### Installatie

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` lost de afhankelijkheden niet op. Als er ontbrekende pakketten worden gemeld, voltooit `sudo apt-get install -f` (of `sudo apt --fix-broken install`) de installatie — dit is het normale verloop, geen fout.
{% endhint %}

Controleer de installatie:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Systeemvereisten

| Vereiste | Minimaal | Aanbevolen |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson met JetPack 6 | Jetson Orin NX 16 GB of AGX Orin |
| **JetPack** | JetPack 6.x | Nieuwste JetPack 6 |
| **Geheugen (RAM)** | 8 GB (gedeeld tussen GPU en CPU) | 16 GB+ gedeeld (12 GB+ is de drempel voor parallelle GPU-workers) |
| **Opslag** | 2 GB vrije ruimte | NVMe SSD met 10 GB+ vrije ruimte |
| **Python** | Python 3.7+ (voor de SDK) | Python 3.10+ |

### Installatie

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Dezelfde opzet als de amd64 `.deb`, met een CUDA-build die is afgestemd op Jetson Orin / Orin NX / Orin Nano. Zie de [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md).

***

## Python SDK Installatie (alle Linux)

De SDK is een pure-Python HTTP-client voor de backend, dus hetzelfde pakket werkt op zowel amd64 als arm64. Twee bronnen:**Van PyPI** — de gepubliceerde stabiele release:

```bash
pip install chloros-sdk
```

**Vanuit het meegeleverde wheel** — past gegarandeerd bij de CLI/backend die je zojuist hebt geïnstalleerd (gebruik dit wanneer je `.deb` nieuwer is dan PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668-distributies** (Ubuntu 23.10+, Debian 12+) staan geen systeembrede pip-installaties toe. Gebruik `pip install --user …`, een virtuele omgeving of `sudo pip install --break-system-packages …`. Het pakketinstallatieprogramma installeert SDK nooit automatisch in je systeem Python — die keuze is aan jou.
{% endhint %}

Optionele extra&#x27;s:

| Extra | Commando | Voegt toe |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` voor live voortgangsstreaming |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` voor BLE (DAQ-M)-transport |

Controleer de SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
De `.deb` installeert de Chloros, CLI en de backend. De Python en SDK communiceren met die backend via een lokaal HTTP en API (`http://127.0.0.1:5000`) en start deze automatisch op wanneer dat nodig is. Gebruik altijd het letterlijke IPv4-adres in plaats van `localhost` — `localhost` kan worden omgezet naar `::1` en kost ongeveer twee seconden per verzoek.
{% endhint %}

***

## Eerste installatie

### 1. Aanmelden

Voor toegang tot CLI en SDK is een betaald Chloros+-abonnement (**Copper** of hoger), wat aan de serverzijde wordt afgedwongen: een uitgelogde aanroeper krijgt `401 AUTH_REQUIRED` en een aanroeper met een gratis abonnement (Iron) krijgt `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Inloggegevens worden in de cache opgeslagen in `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Je moet na elke installatie of upgrade opnieuw inloggen.** Het script `prerm` van het pakket wist opzettelijk `~/.chloros/user_session.json` en de in de cache opgeslagen licentie voor elke gebruiker op de machine, zodat een nieuwe build de licentie altijd opnieuw valideert in plaats van te vertrouwen op een verouderde cache.
{% endhint %}

### 2. Controleer de status van je licentie

```bash
chloros-cli status
```

`chloros-cli status` werkt op elk niveau (inclusief gratis), zodat u altijd kunt zien waarom toegang wel of niet beschikbaar is.

### 3. Voer systeemdiagnostiek uit

```bash
chloros-cli selftest
```

Er worden zeven controles achtereenvolgens uitgevoerd en het commando retourneert een waarde anders dan nul als een van deze controles mislukt:

| # | Controle | Wat het aantoont |
| --- | --- | --- |
| 1 | **Versie** | CLI rapporteert zijn versie (`v1.2.0`). |
| 2 | **Poort beschikbaar** | Poort 5000 is vrij, *of* wordt al beantwoord door een goed functionerende Chloros-backend (dat geldt als geslaagd). |
| 3 | **Opstarten van backend** | Het backend-programma wordt gestart. |
| 4 | **API-test (`/api/test`)** | De backend beantwoordt `status: ok`. |
| 5 | **Systeeminformatie** | Geeft `GPU: <name>, CUDA: <bool>, PyTorch: <version>` weer vanuit `/api/system-info`. |
| 6 | **Denoiser-modellen** | Vindt `*.pth.enc`-modellen (op Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + ruisonderdrukker**| Texture Aware is daadwerkelijk bruikbaar — vereist CUDA**en** ten minste één modelbestand. |

De run eindigt met `N/7 checks passed`, waarbij eventuele fouten bij naam worden vermeld.

### 4. Verwerk je eerste dataset

```bash
chloros-cli process ~/datasets/flight001
```

***

## Bestanden en mappen

### Per gebruiker

Chloros bewaart zijn inloggegevens en de configuratie van CLI in één platformonafhankelijke map, **`~/.chloros/`** (op Windows, `%USERPROFILE%\.chloros\`). Twee Linux-specifieke caches volgen daarentegen de XDG-conventies — deze houden rekening met `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` wanneer deze zijn ingesteld.

| Pad | Doel |
| --- | --- |
| `~/.chloros/user_session.json` | Cache voor inlogsessies, aangemaakt door `chloros-cli login` (wordt bij elke installatie/upgrade van een pakket gewist) |
| `~/.chloros/working_directory.txt` | Overschrijving van de standaardprojectmap (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI taalvoorkeur (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Taalinstelling gedeeld met de Windows GUI — een `language` heeft hier voorrang op `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache van één uur voor de opstartcontrole op updates van Linux/Jetson |
| `~/.chloros/backend.log` | Backend-logboek wanneer de backend werd gestart door de CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | In de cache opgeslagen LATTICE-kalibratiepakketten per camera, gesorteerd op serienummer en bundel-hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Optionele gebruikersaanpassingen voor DAQ-capaciteitscorrectieprofielen |
| `~/.config/chloros/system_config.json` | In de cache opgeslagen hardwareprofiel van Dynamic Compute Adaptation — verwijder dit om een nieuwe hardwaredetectie te forceren |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serverlogs, één bestand per start |
| `~/Chloros Projects/` | Standaard projectmap wanneer er geen overschrijving is ingesteld |

### Systeembreed

| Pad | Doel |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-script — stelt `LD_LIBRARY_PATH` in voor de meegeleverde native bibliotheken en voert vervolgens het daadwerkelijke binaire bestand uit |
| `/usr/bin/chloros-backend` | Wrapper-script — hetzelfde, plus `CHLOROS_PRODUCTION=1` zodat de authenticatiepoort van de backend zichzelf nooit stilzwijgend kan uitschakelen |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | De gecompileerde binaire bestanden |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK-runtime vereist voor LATTICE-camera’s |
| `/usr/lib/chloros/models/*.pth.enc` | Versleutelde denoiser-modellen die worden gebruikt door de Texture Aware-debayer |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK-wheel die precies bij deze build past |
| `/usr/lib/chloros/exiftool` | Meegeleverde exiftool (alleen via een symlink gekoppeld aan `/usr/local/bin/exiftool` als er geen systeem-exiftool bestaat) |
| `/etc/chloros/update.conf` | Configuratie van het updatekanaal gelezen door `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Stelt `net.ipv4.ip_unprivileged_port_start = 319` in zodat de backend de PTP-poorten kan binden zonder root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Wijst de dynamische lader naar `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Verleent de aangemelde gebruiker toegang tot de DAQ-U USB-seriële brug (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Opt-in voor een altijd actieve backend-service (geïnstalleerd, **niet ingeschakeld**) |
| `/usr/share/applications/chloros-cli.desktop` | &quot;Chloros CLI&quot; item in het applicatiemenu dat een terminal opent |

## Locatie van het uitvoerbare backend-bestand

CLI en SDK detecteren de backend automatisch:

| Component | Pad |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Overschrijf het backend-pad met de vlag `--backend-exe` CLI of de constructorparameter `backend_exe` SDK, en de poort met `--port` (standaard `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` verwijst naar de **`lattice`**,**`project`**en**`daq pool-*`** op een backend op afstand. De kernopdrachten (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) negeren dit opzettelijk en richten zich altijd op `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-camera’s en DAQ-lichtsensoren op Linux

De live-hardware-opdrachtfamilies werken allemaal op Linux (amd64 en Jetson):

* **`chloros-cli lattice`** — detecteren, verbinden, configureren en opnames maken van LATTICE-camera&#x27;s en gesynchroniseerde arrays. De `.deb` bundelt de Arena SDK-runtime die hiervoor nodig is en registreert deze bij de dynamische lader.
* **`chloros-cli daq pool-*`** — sluit DAQ-U/M/E-lichtsensoren aan via de backend-pool, gekalibreerde spectra streamen en `.daq`-bestanden opnemen. De gecompileerde CLI bevat uitsluitend de `pool-*`-familie: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — voer een opgeslagen project (met bijbehorende camera’s, sensoren en verwerkingsinstellingen) headless uit.
* **`chloros-cli time-sync`** — de PTP-grandmaster inspecteren waarop de Chloros-backend draait voor LATTICE-camera’s en DAQ-E-sensoren.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` is vereist voor `pool-latest`, `pool-stream`, `pool-record` en `pool-set-cap`; `pool-list` toont de ID&#x27;s die momenteel in de pool aanwezig zijn.

{% hint style="info" %}
**Geef de voorkeur aan `--eth-host` voor de eerste DAQ-E-verbinding op een machine met meerdere netwerkinterfaces.** Automatische detectie doorzoekt mDNS en kan de interface van de sensor missen vanwege een lege ARP-cache, waardoor de eerste `pool-connect --eth` na het opstarten kan mislukken, zelfs als de sensor volledig in orde is. Door het IP-adres of de hostnaam van de sensor door te geven, wordt de detectie volledig overgeslagen.
{% endhint %}

**DAQ-U seriële machtigingen** worden afgehandeld door de geïnstalleerde udev-regel (`uaccess` + groep `dialout`). Als een sensor die al was aangesloten ontoegankelijk blijft, laad dan de regels opnieuw of sluit de sensor opnieuw aan:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Raadpleeg de [CLI-referentie](../CLI.md) voor de volledige reeks commando’s.

### Altijd actieve PTP voor headless hosts

Bij de eerste installatie wordt de systemd-unit `chloros-backend.service` gegenereerd, maar deze is **niet ingeschakeld**. Schakel deze in op een headless Jetson of server waarop de PTP-tijdsynchronisatie continu moet blijven draaien voor DAQ-E-sensoren en LATTICE-camera’s:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Zonder deze unit draait PTP alleen terwijl de backend Chloros actief is — dat wil zeggen, tijdens een actieve CLI/SDK-sessie.

Het apparaat koppelt de backend aan `127.0.0.1:5000` (omgevingsinstellingen `CHLOROS_HOST` / `CHLOROS_PORT` binnen het apparaat; overschrijven met `sudo systemctl edit chloros-backend.service`) en start deze bij een storing na 5 seconden opnieuw op.

**Hoe PTP aan zijn poorten komt.** PTP gebruikt UDP 319/320, beide onder de normale drempel van 1024 bevoorrechte poorten. Het `postinst` van het pakket schrijft `/etc/sysctl.d/60-chloros-ptp.conf` met `net.ipv4.ip_unprivileged_port_start = 319`, waardoor de backend deze kan binden terwijl het als uw gebruiker draait. Als extra veiligheidsmaatregel wordt ook `setcap cap_net_bind_service,cap_net_raw=+ep` toegepast op het backend-binaire bestand — daarom is `libcap2-bin` een gedeclareerde afhankelijkheid van het pakket.***

## Voorbeelden van Bash-scripts

{% hint style="info" %}
**Scriptvriendelijke exitcodes.**`chloros-cli process` beëindigt `0` bij succes en**met een waarde anders dan nul bij een fout — inclusief een uitvoering waarbij beeldproducten werden aangevraagd maar er geen werden geschreven** (het geeft `Processing finished but wrote no image products.` weer en vermeldt de projectmap en de gebruikelijke oorzaken). Succesvolle uitvoeringen rapporteren hoeveel beeldproducten zijn opgeslagen (`Image products written: N`). Uitvoercodes: `0` succes, `1` mislukking, `2` argumentfout, `130` onderbroken.
{% endhint %}

### Meerdere datasets verwerken

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Verwerken met aangepaste instellingen

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Er zijn precies vier geldige `--format`-waarden, en deze bevatten spaties — zet ze altijd tussen aanhalingstekens:

| `--format`-waarde | Uitvoermap |
| --- | --- |
| `TIFF (16-bit)` *(standaard)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accepteert `standard` (standaard) of `texture-aware` (Chloros+).

### Geautomatiseerde verwerking met Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Voorbeeld

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Problemen oplossen

### CLI niet gevonden na installatie

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Toegang geweigerd

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### „setcap failed“ tijdens installatie

`.deb` past `cap_net_bind_service` toe op `/usr/lib/chloros/chloros-backend`, zodat het de PTP-poorten 319/320 kan binden zonder root-rechten. Als `libcap2-bin` ontbrak tijdens de installatie, wordt de aanroep overgeslagen. Installeer het en installeer het pakket opnieuw:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP start niet / kan poort 319 niet toewijzen

Controleer of de drempel voor poorten zonder bevoegdheden is verlaagd en pas deze opnieuw toe voor de huidige opstart als dat nog niet is gebeurd:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Controleer vervolgens de grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### &quot;LATTICE-camerastuurprogramma&#x27;s niet gevonden&quot;

De Arena SDK-runtime wordt niet gevonden. Controleer of de loaderconfiguratie die het pakket schrijft, aanwezig is en is vernieuwd:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Backend kon niet worden gestart

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

De backend-logbestanden voor de mislukte start staan in `~/.cache/chloros/logs/`.

### CUDA niet gedetecteerd

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` rapporteert hetzelfde in één regel: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Ontbrekende gedeelde bibliotheken

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Trage opstarttijd op systemen met SD-kaart

De gecompileerde binaire bestanden worden bij elke start uitgepakt naar een tijdelijke map. Als `/mnt/ssd/tmp` bestaat, gebruikt Chloros dit automatisch; anders stel je `TMPDIR` in op een snel bestandssysteem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Chloros bijwerken op Linux

Het commando `update` is uitsluitend beschikbaar op Linux/Jetson. Het controleert de versie die is gepubliceerd in het updatekanaal dat is geconfigureerd bij `/etc/chloros/update.conf` en biedt aan om de bijbehorende `.deb` te downloaden en te installeren:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Op Linux/Jetson voert de CLI bij elke opstart ook een niet-blokkerende updatecontrole uit (het resultaat wordt één uur in de cache opgeslagen in `~/.chloros/update_cache.json`) en geeft `Update available: vX.Y.Z` weer als er een nieuwere versie beschikbaar is. Je instellingen en projecten blijven bij een update behouden; je moet daarna wel opnieuw inloggen.

## Verwijderen

```bash
sudo apt remove chloros
```

Het verwijderen stopt `chloros-backend.service`, herstelt de standaard ondergrens voor niet-bevoegde poorten (1024), verwijdert de symlink naar de meegeleverde exiftool en de configuratie van de Arena-loader, en wist de in de cache opgeslagen inloggegevens. Uw projecten en `~/.chloros/`-gegevensbestanden blijven intact.

***

## Volgende stappen

* [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) — Jetson-specifieke optimalisatie en implementatie
* [CLI : Opdrachtregel](../CLI.md) — de CLI-handleiding
* [API : Python SDK](../api-python-sdk.md) — de SDK-handleiding
* [CLI-referentie](../reference/cli-reference.md) en [SDK-referentie](../reference/sdk-reference.md) — uitgebreide lijst met opdrachten/API voor versie 1.2.0
* [Dynamische rekenaanpassing](../processing-architecture/dynamic-compute-adaptation.md) — hoe Chloros zich aanpast aan uw hardware
