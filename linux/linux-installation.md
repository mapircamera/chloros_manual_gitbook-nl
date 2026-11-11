# Installatie op Linux

Chloros wordt voor Linux gedistribueerd als `.deb`-pakketten die de CLI, de backend-server en de Python-SDK installeren (voor het systeem `python3`, plus een bij de versie passend wheel voor virtualenvs).

De bestandsnamen van de pakketten bevatten de versie en de architectuur: `chloros_1.3.0_amd64.deb` voor x86_64, en `chloros_1.3.0_arm64_jp6.deb` voor arm64 (NVIDIA Jetson, JetPack 6). Vervang in de onderstaande commando’s het bestand dat je daadwerkelijk hebt gedownload.

{% hint style="info" %}
**Nieuw in 1.3.0: de programma’s worden uitgepakt geleverd.** De backend en de CLI worden geïnstalleerd als mappen, `/usr/lib/chloros/backend/` en `/usr/lib/chloros/cli/`, in plaats van als afzonderlijke zelfuitpakkende bestanden. Ze pakken zichzelf niet langer bij elke start uit in een tijdelijke map, waardoor ze veel sneller opstarten (op x86_64 was de backend vroeger bij elke start ongeveer 100 seconden bezig met uitpakken) en er is niet langer enkele GB vrije ruimte nodig in `/tmp`. De amd64-versie `.deb` is ook ongeveer half zo groot als voorheen.
{% endhint %}

***

## Linux amd64 (x86_64)

### Systeemvereisten

| Vereiste | Minimaal | Aanbevolen |
| --- | --- | --- |
| **Distributie** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processor** | x86_64 (Intel/AMD) | Intel Core i7 of beter |
| **Geheugen (RAM)** | 8 GB | 16 GB of meer |
| **Grafische kaart** | Geen (verwerking via CPU) | NVIDIA GPU met 4 GB+ VRAM (12 GB+ ontgrendelt `GPU_PARALLEL`, 7 GB+ houdt Texture Aware uitgeschakeld voor het single-image-pad) |
| **Opslagruimte** | 2 GB vrije ruimte | SSD met 10 GB+ vrije ruimte |
| **Python** | Python 3.7+ (voor de SDK) | Python 3.10+ |

> **Ubuntu 20.04 en Debian 11 worden niet ondersteund.** De lijst met afhankelijkheden van de `.deb` is
> afgeleid van de bibliotheken waaraan de Chloros-backend daadwerkelijk koppelt, en dat omvat een
> `libc6` die nieuwer is dan de glibc 2.31 die in Focal en Bullseye wordt meegeleverd, dus `apt` weigert de
> installatie direct, in plaats van deze later tijdens de uitvoering te laten mislukken.

### Installatie

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Gebruik `apt install ./…`, niet `dpkg -i`.** `apt` lost de afhankelijkheden van het pakket op en installeert ook wat het aanbeveelt: de vereisten van de Python SDK, `python3-numpy` en `python3-requests`. `dpkg -i` installeert geen van beide. De CLI en backend zijn op zichzelf staand en werken zonder deze componenten, maar `import chloros_sdk` heeft ze nodig; de installatie-uitvoer vermeldt welke er ontbreken. Bewaar het bestand `./`, zodat `apt` het bestand installeert in plaats van te zoeken naar een pakket met die naam. Het pakket declareert ook `libgl1`, dat door de meegeleverde OpenCV wordt geladen en mogelijk ontbreekt in een minimale serverimage.
{% endhint %}

Controleer de installatie:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Systeemvereisten

| Vereiste | Minimaal | Aanbevolen |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson met JetPack 6 | Jetson Orin NX 16 GB of AGX Orin |
| **JetPack** | JetPack 6.x | Nieuwste JetPack 6 |
| **Geheugen (RAM)** | 8 GB (gedeeld tussen GPU en CPU) | 16 GB+ gedeeld (12 GB+ is de drempel voor parallelle verwerking) |
| **Opslag** | 2 GB vrije ruimte | NVMe SSD met 10 GB+ vrije ruimte |
| **Python** | Python 3.7+ (voor de SDK) | Python 3.10+ |

### Installatie

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Dezelfde opzet als de amd64 `.deb` (inclusief Python SDK, ontvangstbuffers en backend-services), voor Jetson Orin / Orin NX / Orin Nano op **JetPack 6** (L4T R36). De PyTorch-versie is de CUDA-build van JetPack 6 voor Orin, dus GPU-verwerking, inclusief de Texture Aware-debayer, draait op de GPU van de Jetson. Het pakket bevat CUDA zelf niet: het maakt gebruik van de CUDA- en cuDNN-bibliotheken die JetPack 6 installeert (`sudo apt install nvidia-jetpack` op een image waar deze ontbreken). Als er geen CUDA-GPU beschikbaar is, vindt de verwerking plaats op de CPU en gebruikt `--debayer texture-aware` de Standard-debayer (de backend registreert één `[DEBAYER] … needs a CUDA GPU …`-regel waarin wordt uitgelegd waarom). Zie de [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) voor informatie over het gedrag van Jetson met betrekking tot geheugen, temperatuur en implementatie in het veld.***

## Installatie van de Python-SDK (alle Linux-versies)**`.deb` installeert de SDK voor je.** Beide pakketten (amd64 en Jetson) installeren het voor het systeem `python3`, dus `import chloros_sdk` werkt direct na de installatie, zonder pip en zonder netwerk. De bijbehorende pakketten (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` en de projectladers) worden ondergebracht in `/usr/lib/python3/dist-packages`, dat eigendom is van het pakket: bij een upgrade worden ze vervangen, en bij het verwijderen van het pakket worden ze verwijderd. De versie komt altijd overeen met de CLI en backend die je hebt geïnstalleerd.

De eigen vereisten van de SDK, `python3-numpy` en `python3-requests`, zijn **aanbevolen, niet vereist** door het pakket: `sudo apt install ./chloros_<version>_<arch>.deb` installeert ze, `dpkg -i` niet. De installatie-uitvoer vermeldt de SDK-versie die `python3` importeert, eventuele ontbrekende vereisten, en alle `chloros_sdk`-bestanden elders in het pad van `python3` die de kopie van het pakket overschrijven (zoals een versie die eerder met `pip install --user` is geïnstalleerd).**In een virtualenv**, dat de pakketten van het systeem niet ziet, installeer je het meegeleverde wheel daarin (volledig offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Vanuit PyPI** — alleen voor een machine waarop geen Chloros-pakket is geïnstalleerd, waarvan de SDK een backend op een andere machine aanstuurt (zie [Remote-Backend Mode](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Optionele extra's, voor een pip-installatie (een virtualenv of PyPI):

| Extra | Commando | Voegt toe |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` voor het live streamen van voortgang |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` voor BLE (DAQ-M)-transport |

Controleer de SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
De Python-SDK communiceert met de backend die `.deb` installeert via een lokale HTTP-API (`http://127.0.0.1:5000`). `ChlorosLocal` en de connect-helpers starten deze automatisch op wanneer dat nodig is; de `connect_all()` van een opgeslagen project doet dit nooit, dus voer op een headless-machine de backend uit als een [service](#always-on-ptp-for-headless-hosts). Gebruik altijd het letterlijke IPv4-adres in plaats van `localhost` — `localhost` kan worden omgezet naar `::1` en kost ongeveer twee seconden per verzoek.
{% endhint %}

***

## Eerste installatie

### 1. Inloggen

Voor toegang via de CLI en SDK is een betaald Chloros+-abonnement (**Copper** of hoger) vereist; dit wordt serverzijde afgedwongen: een uitgelogde aanroeper krijgt `401 AUTH_REQUIRED`, en een aanroeper met een gratis abonnement (Iron) krijgt `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Inloggegevens worden opgeslagen in `~/.chloros/user_session.json`.

{% hint style="info" %}
**Bij upgrades blijft je inloggegevens behouden; bij verwijdering worden deze gewist.** Als je een nieuwere versie van `.deb` over een bestaande installatie installeert (ook via `chloros-cli update`), blijven `~/.chloros/user_session.json` en de in de cache opgeslagen licentie behouden, zodat headless CLI-/SDK-taken blijven werken. Als je het pakket (`sudo apt remove chloros`) verwijdert, worden de opgeslagen inloggegevens en de licentiecache voor elke gebruiker op de machine gewist, dus voor een nieuwe installatie is `chloros-cli login` weer nodig.
{% endhint %}

### 2. Controleer je licentiestatus

```bash
chloros-cli status
```

`chloros-cli status` werkt op elk niveau (inclusief gratis), zodat je altijd kunt zien waarom toegang wel of niet beschikbaar is.

### 3. Voer systeemdiagnostiek uit

```bash
chloros-cli selftest
```

Er worden zeven controles achtereenvolgens uitgevoerd, en het commando retourneert een waarde anders dan nul als een van deze controles mislukt:

| # | Controle | Wat het aantoont |
| --- | --- | --- |
| 1 | **Versie** | De CLI geeft de versie weer (`v1.3.0`). |
| 2 | **Poort beschikbaar** | Poort 5000 is vrij, *of* er is al een reactie ontvangen van een goed functionerende Chloros-backend (dat telt als geslaagd). |
| 3 | **Opstarten backend** | Het backend-programma wordt gestart. |
| 4 | **API-test (`/api/test`)** | De backend antwoordt met `status: ok`. |
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

### Per gebruiker bewaart

Chloros zijn inloggegevens en CLI-configuratie in één platformonafhankelijke map, **`~/.chloros/`** (op Windows: `%USERPROFILE%\.chloros\`). Twee Linux-specifieke caches volgen daarentegen de XDG-conventies — deze gebruiken `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` wanneer ze zijn ingesteld.

| Pad | Doel |
| --- | --- |
| `~/.chloros/user_session.json` | Cache voor inlogsessies, aangemaakt door `chloros-cli login` (blijft behouden bij pakketupgrades; wordt gewist wanneer het pakket wordt verwijderd) |
| `~/.chloros/working_directory.txt` | Overschrijving van de standaardprojectmap (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Taalvoorkeur voor de CLI-interface (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Taalinstelling gedeeld met de Windows-GUI — een `language` heeft hier voorrang op `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache van één uur voor de opstart-updatecontrole van Linux/Jetson |
| `~/.chloros/backend.log` | Backend-logboek wanneer de backend via de CLI is gestart |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | In de cache opgeslagen LATTICE-kalibratiepakketten per camera, gesorteerd op serienummer en bundel-hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Optionele gebruikersaanpassingen voor DAQ-capaciteitscorrectieprofielen |
| `~/.config/chloros/system_config.json` | In de cache opgeslagen hardwareprofiel van Dynamic Compute Adaptation — verwijder dit om een nieuwe hardwaredetectie te forceren |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serverlogs, één bestand per start |
| `~/Documents/MAPIR/Chloros Projects/` | Standaard projectmap wanneer er geen overschrijving is ingesteld |

### Systeembreed

| Pad | Doel |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-script — stelt `LD_LIBRARY_PATH` in voor de meegeleverde native bibliotheken (en `TMPDIR` naar `/mnt/ssd/tmp` wanneer die map bestaat), voert vervolgens het daadwerkelijke binaire bestand uit |
| `/usr/bin/chloros-backend` | Wrapper-script — hetzelfde voor de backend, die het markeert als een geïnstalleerde productie-build |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | De gecompileerde CLI en backend, uitgepakt geïnstalleerd (nieuw in 1.3.0); de programma’s zijn `cli/chloros-cli` en `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Starterscripts van twee regels die de bovenstaande echte programma’s starten — bewaard op deze paden omdat de SDK, de CLI en oudere scripts ze hier zoeken |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK-runtime vereist voor LATTICE-camera’s |
| `/usr/lib/chloros/models/*.pth.enc` | Versleutelde denoiser-modellen die worden gebruikt door de Texture Aware-debayer |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (en `lattice_sdk`, `lattice_cam`, `arena_api`, de projectladers) | De Python SDK, geïnstalleerd voor het systeem `python3` en eigendom van het pakket |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK-wheel die exact overeenkomt met deze build, voor een virtualenv |
| `/usr/lib/chloros/exiftool` | Meegeleverde ExifTool-launcher (alleen via een symlink gekoppeld aan `/usr/local/bin/exiftool` als er geen systeem-exiftool bestaat). Vanaf versie 1.3.0 wordt het ExifTool-script geleverd met een eigen Perl-bibliotheek in `/usr/lib/chloros/exiftool-dist/`, zodat het zonder het systeempakket `libimage-exiftool-perl` kan draaien |
| `/etc/chloros/update.conf` | Configuratie van het updatekanaal, gelezen door `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Stelt `net.ipv4.ip_unprivileged_port_start = 319` in zodat de backend de PTP-poorten kan binden zonder root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Verhoogt de socket-ontvangstbuffers van de kernel (`net.core.rmem_max`, `net.core.rmem_default`) voor LATTICE-camerastromen naar 32 MB en past deze toe bij de installatie (een reeds ingestelde hogere waarde blijft ongewijzigd). Bij de standaardinstelling van de kernel worden camerapakketten in de kernel weggegooid en verliest een array frames en framerate; `chloros-cli lattice network` meldt wanneer de buffers bijna leeg zijn. Zie [Linux-hosts — socket-ontvangstbuffers](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Wijst de dynamische lader naar `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Verleent de aangemelde gebruiker toegang tot de DAQ-U USB-seriële brug (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Opt-in ‘always-on’ backend-service die de backend uitvoert als de door u opgegeven gebruiker (geïnstalleerd, **niet ingeschakeld**) — deze moet u inschakelen, zie [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Dezelfde service die de backend als **root**uitvoert (geïnstalleerd,**niet ingeschakeld**), behouden zodat een machine waarop deze is ingeschakeld blijft draaien |
| `/var/log/chloros/` | Het gedetailleerde logboek van de root-service, leesbaar voor de groep `adm` (wordt verwijderd wanneer het pakket volledig wordt verwijderd) |
| `/usr/share/applications/chloros-cli.desktop` | "Chloros CLI"-menu-item dat een terminal opent |

**Firewall (nieuw in 1.3.0).** Als `ufw` of `firewalld` actief is op het moment van installatie, opent het pakket de UDP-poorten `5002` (DAQ-E-gegevens), `5353` (mDNS-detectie), `319` en `320` (PTP), en verwijdert deze regels wanneer het pakket wordt verwijderd. Als er een firewall is geïnstalleerd maar niet actief is, voegt het niets toe en geeft het de poorten weer die moeten worden toegestaan mocht u deze later inschakelen. Het opent geen TCP-poort: een DAQ-E-firmware-update, waarbij de sensor via TCP verbinding maakt met deze computer om de afbeelding op te halen, mislukt met foutcode `… device never connected back on TCP <port> …` wanneer een actieve firewall die verbinding blokkeert.

## Locatie van het uitvoerbare backend-bestand

De CLI en SDK detecteren de backend automatisch:

| Component | Pad |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (een launcher voor `/usr/lib/chloros/backend/chloros-backend`) |

Overschrijf het pad naar de backend met de CLI-vlag `--backend-exe` of de SDK-constructorparameter `backend_exe`, en de poort met `--port` (standaard `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` wijst de **`lattice`**,**`project`**en**`daq pool-*`** op een backend op afstand. De kernopdrachten (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) negeren dit opzettelijk en richten zich altijd op `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-camera’s en DAQ-lichtsensoren op Linux

De commandoreeksen van live-hardware werken allemaal op Linux (amd64 en Jetson):

* **`chloros-cli lattice`** — detecteer, maak verbinding, configureer en leg beelden vast van LATTICE-camera’s en gesynchroniseerde arrays. De `.deb` bundelt de benodigde Arena SDK-runtime en registreert deze bij de dynamische loader.
* **`chloros-cli daq pool-*`** — DAQ-U/M/E-lichtsensoren aansluiten via de backend-pool, gekalibreerde spectra streamen, `.daq`-bestanden opnemen en cap-profielen wisselen, en voor een DAQ-E de netwerkinstellingen wijzigen, het apparaat opnieuw opstarten en de firmware bijwerken. `chloros-cli daq discover` scant naar sensoren die je kunt aansluiten, en `chloros-cli daq --help` geeft een overzicht van de `pool-*`-subcommando’s.
* **`chloros-cli project`** — voer een opgeslagen project (met bijbehorende camera’s, sensoren en verwerkingsinstellingen) headless uit.
* **`chloros-cli time-sync`** — controleer de PTP-grandmaster waarop de Chloros-backend draait voor LATTICE-camera’s en DAQ-E-sensoren.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` is vereist voor `pool-latest`, `pool-stream`, `pool-record` en `pool-set-cap`; `pool-list` toont de ID's die momenteel in de pool aanwezig zijn.

{% hint style="info" %}
**`--eth-host` slaat de detectiefase over.** `pool-connect --eth` vindt een DAQ-E via mDNS, met een ARP-fallback die werkt vanuit een lege ARP-cache; door het IP-adres of de hostnaam van de sensor door te geven met `--eth-host` wordt er rechtstreeks verbinding mee gemaakt.
{% endhint %}

**De seriële machtigingen voor DAQ-U** worden afgehandeld door de geïnstalleerde udev-regel (`uaccess` + groep `dialout`). Als een sensor die al was aangesloten ontoegankelijk blijft, laad dan de regels opnieuw of sluit de sensor opnieuw aan:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Raadpleeg de [CLI-referentie](../CLI.md) voor de volledige set commando’s.

### Altijd actieve PTP voor headless hosts

Het pakket installeert twee systemd-services voor de backend, die beide niet zijn ingeschakeld. Op een headless Jetson of server waarop de PTP-tijdsynchronisatie continu moet blijven draaien voor DAQ-E-sensoren en LATTICE-camera’s, of waar SDK-scripts opgeslagen projecten koppelen (`connect_all()` start nooit een backend), schakel de service dan **als uw gebruiker** in:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` draait de backend als die gebruiker: de opnames die het schrijft zijn van die gebruiker, het gedetailleerde logboek staat in hun `~/.cache/chloros/logs` en het gebruikt hun `chloros-cli login` (`~/.chloros`), dus log eerst één keer in als die gebruiker. Bij het installeren van het pakket wordt deze opdracht weergegeven met uw gebruikersnaam ingevuld.

Zonder een service draait de backend (en daarmee de PTP-grandmaster) alleen zolang iets deze heeft gestart — een `chloros-cli`-opdracht of een SDK-script.

De unit koppelt de backend aan de `127.0.0.1:5000` (omgevingsinstellingen `CHLOROS_HOST` / `CHLOROS_PORT` binnen de unit; overschrijf met `sudo systemctl edit chloros-backend@.service`) en start deze bij een storing na 5 seconden opnieuw op.

**De root-service.**`chloros-backend.service` draait de backend als**root**. Deze is nog steeds geïnstalleerd, zodat een machine waarop deze is ingeschakeld blijft draaien, maar alles wat een root-backend schrijft, is eigendom van root, dus een gebruiker kan zijn eigen opnames niet verwijderen; het logboek bevindt zich in `/var/log/chloros` en is leesbaar voor de groep `adm`. Om over te schakelen naar uw gebruiker:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**De backend van een service wordt gedeeld en wordt nooit door een client gestopt.** `chloros-cli` en SDK-scripts gebruiken de backend zoals ze die aantreffen: `process` draait erop en laadt zijn eigen project in plaats van het laatste. Waar de CLI anders de backend opnieuw zou starten (`--restart`, of een `process` waarvan de import niets vond), noemt het de unit, geeft `sudo systemctl restart <unit>` weer en sluit `1` af, omdat een backend die op die manier wordt gestopt, uitgeschakeld zou blijven. Start deze zelf opnieuw op met `sudo systemctl restart chloros-backend@$USER.service`. De backend van een service stopt nooit uit zichzelf bij inactiviteit, en `chloros-cli backend stop` weigert dit, waarbij het de opdracht `systemctl stop` noemt.**Hoe PTP aan zijn poorten komt.** PTP gebruikt UDP 319/320, beide onder de normale drempel van 1024 voor bevoorrechte poorten. Het `postinst` van het pakket schrijft `/etc/sysctl.d/60-chloros-ptp.conf` met `net.ipv4.ip_unprivileged_port_start = 319`, waardoor de backend deze kan binden terwijl deze als uw gebruiker draait. Vanaf versie 1.3.0 stelt het pakket geen bestandsrechten (`setcap`) in op de backend, en is `libcap2-bin` niet langer een afhankelijkheid: op de uitgepakte backend zou een bestandsrecht het opstarten verhinderen, dus voeg er zelf geen toe.***

## Voorbeelden van Bash-scripts

{% hint style="info" %}
**Scriptvriendelijke exitcodes.**`chloros-cli process` beëindigt `0` bij succes en**met een waarde anders dan nul bij een fout — inclusief een uitvoering waarbij beeldproducten werden aangevraagd maar er geen werden geschreven** (het geeft `Processing finished but wrote no image products.` weer en vermeldt de projectmap en de gebruikelijke oorzaken). Succesvolle uitvoeringen rapporteren hoeveel beeldproducten zijn geschreven (`Image products written: N`, plus `(plus M clamp masks)` wanneer maskers zijn geschreven). Uitgangscodes: `0` succes, `1` mislukking, `2` argumentfout, `130` onderbroken.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Er zijn precies vier geldige `--format`-waarden, en deze bevatten spaties — zet ze altijd tussen aanhalingstekens:

| `--format`-waarde | Uitvoermap |
| --- | --- |
| `TIFF (16-bit)` *(standaard)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accepteert `standard` (standaard) of `texture-aware` (Chloros+).

### Geautomatiseerde verwerking met Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Voorbeeld van de Python-SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Probleemoplossing

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend start niet na het toevoegen van bestandsrechten

Voer `setcap` niet uit op de backend. Vanaf versie 1.3.0 stelt het pakket geen bestandsmogelijkheden in, en de PTP-poorten 319/320 zijn afkomstig van de instelling `sysctl` die wordt beschreven onder [Hoe PTP aan zijn poorten komt](#always-on-ptp-for-headless-hosts). Een bestandscapaciteit zet de uitgepakte backend in een beveiligde uitvoeringsmodus waarin deze zijn eigen bibliotheken niet kan vinden, waardoor hij niet start. Als u er een hebt toegevoegd, verwijder deze dan:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP start niet / kan geen verbinding maken met poort 319

Controleer of de ondergrens voor niet-bevoegde poorten is verlaagd en pas deze opnieuw toe voor de huidige opstart als dat nog niet is gebeurd:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Controleer vervolgens de grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice`-opdrachten mislukken met de foutmelding „LATTICE-camerastuurprogramma’s niet gevonden”

De backend kon de LATTICE-cameraruntime waarmee deze is geïnstalleerd niet laden. Elk `lattice`-commando wordt op de backend uitgevoerd; de CLI zelf bevat geen camerastuurprogramma’s. Installeer Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`) opnieuw: de `.deb` installeert de runtime naast de backend, in `/usr/lib/chloros/arena_runtime`, en registreert deze bij de dynamische lader. Om die registratie te controleren:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

De backend-logbestanden voor de mislukte start staan in `~/.cache/chloros/logs/`. Als poort 5000 bij een backend-service hoort (`systemctl status chloros-backend@$USER.service`), start de service dan opnieuw op in plaats van het proces te beëindigen.

### CUDA niet gedetecteerd

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` rapporteert hetzelfde in één regel: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Ontbrekende gedeelde bibliotheken

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Trage opstarttijd op systemen met SD-kaart

Vanaf versie 1.3.0 worden de backend en de CLI ongecomprimeerd geïnstalleerd, zodat ze zich niet langer bij elke start naar een tijdelijke map uitpakken. Tijdelijke bestanden worden automatisch naar `/mnt/ssd/tmp` verplaatst als die map bestaat; anders kun je ze naar een snel bestandssysteem laten verwijzen:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Chloros bijwerken op Linux

Het commando `update` is alleen beschikbaar op Linux/Jetson. Het controleert de versie die is gepubliceerd in het updatekanaal dat is geconfigureerd in `/etc/chloros/update.conf` en biedt aan om de bijbehorende `.deb` te downloaden en te installeren:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Op Linux/Jetson voert de CLI bij elke opstart ook een niet-blokkerende updatecontrole uit (het resultaat wordt een uur lang in de cache opgeslagen in `~/.chloros/update_cache.json`) en geeft `Update available: vX.Y.Z` weer wanneer er een nieuwere versie beschikbaar is. Uw instellingen, projecten en aanmeldingsgegevens blijven na een update behouden.

Het upgraden van het pakket stopt alle backend-services tijdens de installatie, waardoor een actief script de verbinding met zijn camera’s en sensoren verliest en vervolgens elke ingeschakelde service opnieuw start (een service die handmatig is gestart maar nooit is ingeschakeld, blijft gestopt, net als na een herstart).

## Verwijderen

```bash
sudo apt remove chloros
```

Het verwijderen stopt en schakelt alle backend-services uit, herstelt de standaard ondergrens voor niet-bevoegde poorten (1024), verwijdert het instellingenbestand voor de ontvangstbuffer (de huidige waarden blijven behouden tot de herstart), de symlink naar de meegeleverde exiftool, de Arena-loaderconfiguratie, de firewallregels die het pakket heeft toegevoegd en de Python-SDK die het heeft geïnstalleerd, en wist de in de cache opgeslagen inloggegevens. Uw projecten en `~/.chloros/`-gegevensbestanden blijven intact. `sudo apt purge chloros` verwijdert ook `/var/log/chloros`.

***

## Volgende stappen

* [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) — Jetson-specifieke optimalisatie en implementatie
* [CLI: opdrachtregel](../CLI.md) — de CLI-handleiding
* [API: Python SDK](../api-python-sdk.md) — de SDK-handleiding
* [CLI-referentie](../reference/cli-reference.md) en [SDK-referentie](../reference/sdk-reference.md) — uitgebreide lijsten met commando’s en API’s voor versie 1.3.0
* [Dynamische rekenkrachtanpassing](../processing-architecture/dynamic-compute-adaptation.md) — hoe Chloroszich aanpast aan uw hardware
