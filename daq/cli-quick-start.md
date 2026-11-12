# Snelstartgids voor de CLI (pool-*)

De meegeleverde `chloros-cli`-drivers sturen DAQ-sensoren aan via de **`daq pool-*`**-opdrachtfamilie — ‘thin’ HTTP-clients die de sensor aansturen via de permanente sensorpool van de Chloros-backend. De backend beheert het transport, zodat de GUI, de CLI en de SDK-scripts allemaal één actieve handle delen in plaats van te strijden om de poort. Alles wat een klant nodig heeft, is bereikbaar via `pool-*`: verbinding maken, streamen, gekalibreerde `.daq`-bestanden opnemen en cap-profielen wisselen.

`pool-*` is de ondersteunde DAQ-interface in uitgebrachte builds. `chloros-cli daq --help` geeft een overzicht van de `pool-*`-subcommando’s, en `daq discover` / `daq list` voeren dezelfde scan uit als `pool-discover`. Wanneer een DAQ-subcommando voor directe hardware niet beschikbaar is, wordt het programma afgesloten met een expliciete foutmelding waarin het ontbrekende pakket wordt genoemd en waarin wordt verwezen naar `pool-*` — niets mislukt in stilte. (`pip install chloros-sdk` biedt evenmin de directe-hardware-opdrachten.)

***

## Vereisten

* **De Chloros-backend moet actief zijn** — de `pool-*`-opdrachten zijn HTTP-clients, geen hardwarestuurprogramma’s. Start op Windows de Chloros-desktopapp (deze start de backend). Op een headless Linux/Jetson moet je de service als je gebruiker inschakelen: `sudo systemctl enable --now chloros-backend@$USER.service` (zie [Linux-installatie](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Een Chloros+ (betaalde versie) aanmelding**: voer eerst `chloros-cli login` uit. De handhaving vindt plaats aan de serverzijde — zonder aanmelding mislukken commando’s met `401 AUTH_REQUIRED`; op het gratis (Iron) abonnement mislukken ze met `403 PLAN_UPGRADE_REQUIRED`.
* De commando’s zijn standaard gericht op `http://127.0.0.1:5000`; de `daq pool-*`-familie houdt rekening met de omgevingsvariabele `CHLOROS_BACKEND_URL` als je backend elders draait.

***

## Een sessie van vijf minuten

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — een sensor in de pool openen

| Variant | Betekenis |
| --- | --- |
| `daq pool-connect` | Smart-detect: zoek alle DAQ’s op deze machine. |
| `daq pool-connect --port PORT` | DAQ-U op een specifieke seriële poort (bijv. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC automatisch gescand. |
| `daq pool-connect --mac MAC` | DAQ-M op een bekend BLE-MAC-adres (impliceert `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E met een bekende hostnaam of IP — **de betrouwbare methode**. |
| `daq pool-connect --eth` | DAQ-E met automatische detectie (mDNS, met een ARP-fallback). Zie de opmerking hieronder. |

Afstemmingsvlaggen, allemaal optioneel:

| Vlag | Betekenis |
| --- | --- |
| `--integration-time MS` / `-t MS` | Handmatige integratietijd in milliseconden. |
| `--frame-avg N` / `-f N` | Gemiddeld aantal frames per gerapporteerd spectrum. |
| `--no-ae` | Automatische belichting uitschakelen (AE is standaard ingeschakeld). |
| `--no-stream` | Verbinding maken zonder de stream te starten (later hervatten met `pool-stream --start`). |
| `--cap-id CAP` | Cap-correctieprofiel; de standaardinstelling van de backend is `sunshine_cosine`. Zie [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Als je een DAQ-E aansluit die al in de pool zit onder een ander adres — de hostnaam `daq-e-<id>.local` of het IP-adres — wordt de bestaande poolvermelding teruggegeven in plaats van het apparaat een tweede keer te openen.

{% hint style="warning" %}
**`--eth` waarschuwing bij automatische detectie.** Op een host met meerdere netwerkinterfaces (meer dan één actieve netwerkinterface) kan de *eerste* `pool-connect --eth` na het opstarten leeg zijn, zelfs als de sensor in orde is — de detectieprocedure kan de interface van de sensor missen zolang de ARP-cache nog niet is opgewarmd. Als `--eth` niets vindt, probeer het dan opnieuw, of sla de detectie helemaal over met `--eth-host <ip-or-hostname>`, wat de betrouwbare methode is op machines met meerdere netwerkinterfaces. De hostnaam van de DAQ-E is `daq-e-<id>.local` (bijv. `daq-e-def330.local`); het gewone IP-adres werkt ook.
{% endhint %}

## `pool-discover` — scannen zonder verbinding te maken

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Zoekt naar sensoren waarmee je verbinding zou kunnen maken en toont het adres om verbinding mee te maken, zonder daadwerkelijk verbinding te maken. `--only` neemt een door komma’s gescheiden subset van `usb,ble,eth`; `--timeout SEC` stelt de scantijd per transport in (standaard 5 s, begrensd op 1–20 s). Sensoren die al in de pool zijn geopend, worden niet weergegeven (een aangesloten DAQ-M stopt met adverteren) — gebruik daarvoor `pool-list`. In een geleverde build voeren `chloros-cli daq discover` en `daq list` dezelfde scan uit.

## `pool-list` — bekijk wat er is aangesloten

Toont elke sensor in de backend-pool, inclusief de `sensor_id` die voor alle andere commando’s nodig is:

| Model | `sensor_id`-vorm | Voorbeeld |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-octet met koppeltekens | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Een DAQ-E waarvan PTP is uitgeschakeld (de instelling wordt op het apparaat opgeslagen) wordt in `pool-list` gemarkeerd met `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` en de opdracht waarmee deze weer wordt ingeschakeld — zie [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — spectrumframes lezen

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Geeft het meest recente frame terug, of de meest recente `--recent N`-frames; `--json` genereert machinaal leesbare uitvoer voor scripting. Frames zijn radiometrisch gekalibreerde spectrale stralingsintensiteit (W/m²/nm) op het 135-puntsraster van 340–1010 nm, waarbij het kapprofiel van de sensor al is toegepast. Voor kwantitatieve stralingsintensiteitswaarden moet u frames van ten minste 15 seconden middelen — dit is een eigenschap van het instrument, geen defect.

## `pool-stream` — streaming pauzeren of hervatten

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — een `.daq`-bestand opnemen

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Vlag | Standaard | Betekenis |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Opnameduur in seconden; `0` betekent: blijf draaien totdat je `--stop` invoert. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Uitvoermap. Bij een backend op deze computer wordt een relatief pad omgezet naar de map waarin je het commando hebt uitgevoerd; bij een backend op afstand wordt het pad omgezet **op de machine waarop de backend draait**. |
| `--device-name NAME` | — | Label dat bij de opname wordt opgeslagen. |
| `--stop` | — | Een lopende opname stoppen. |
| `--confirm-after SEC` | `20` | Wacht na het starten maximaal zo lang totdat de eerste rij is geschreven, en geef een luide waarschuwing (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) als er geen binnenkomt. `0` schakelt de controle uit. |
| `--heartbeat SEC` | `30` | Met `--duration` wordt het aantal rijen elke SEC seconden weergegeven en wordt er een waarschuwing gegeven als het aantal niet verder toeneemt. `0` schakelt deze uit. |

{% hint style="info" %}
De opname vindt plaats in de backend, dus het bestand `.daq` komt terecht op het bestandssysteem van de **backend-machine** — standaard in `~/Documents/DAQ Live View/` daar, niet noodzakelijkerwijs waar je de CLI hebt uitgevoerd. Bestandsnamen bevatten de sensor-ID en een tijdstempel.
{% endhint %}

## `pool-set-cap` — de gemonteerde dop aangeven

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

De kap-ID selecteert het in de fabriek gemeten correctieprofiel dat op elk spectrum wordt toegepast, en deze **moet overeenkomen met de kap die fysiek op de sensor is gemonteerd** — noch de sensor, noch de software kan de kap zelf detecteren, en de selectie wordt in elk `.daq`-bestand vastgelegd. De standaardinstelling is overal `sunshine_cosine` (elke DAQ wordt geleverd met de Sunshine cosinus-correctiekap geïnstalleerd, ~12× demping volgens ontwerp — een niet-aangegeven kapwisseling corrigeert spectra verkeerd met ongeveer die factor).

| `--cap-id` | Beschikbaar op |
| --- | --- |
| `sunshine_cosine` (standaard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Alleen DAQ-U |
| `none` | Alleen DAQ-E — zie opmerking |

Een kap-ID die buiten de set van de sensor valt, wordt bij het aansluiten geweigerd met een duidelijke foutmelding. `none` (DAQ-E) betekent dat de kap fysiek is verwijderd — er wordt nog steeds een fabrieksgeometrieprofiel toegepast voor de verzonken glazen diffuser van de DAQ-E, dus het is geen no-op, en een ‘bare’ DAQ-E is een bench-configuratie, geen ondersteunde veldconfiguratie. (Een onbedekte DAQ-U is volledig onbedekt en heeft helemaal geen correctieprofiel nodig; de DAQ-M wordt gebruikt met de bijbehorende Sunshine-kap.)

## `pool-disconnect` — sensoren vrijgeven

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` maakt een sensor vrij, ongeacht wie deze heeft geopend; het maakt dus ook een sensor vrij die door een gecrasht script in de pool is achtergelaten (een opname daarop wordt eerst voltooid). Een sensor die zich niet in de pool bevindt, kan niet worden vrijgegeven: het commando geeft `<sensor-id>: already released (not in the pool)` weer en wordt afgesloten met `0`. Een vrijgave die door de backend is geweigerd of niet is bevestigd, wordt afgesloten met `1`.

## `pool-enable-ptp` — de PTP van een DAQ-E weer inschakelen

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Een DAQ-E bewaart de in- en uitschakelstatus van PTP in zijn eigen geheugen, dus een apparaat waarvan de PTP is uitgeschakeld, blijft uitgeschakeld — en de tijdstempels worden niet via PTP gesynchroniseerd — totdat deze weer wordt ingeschakeld. `pool-list` markeert zo’n apparaat en geeft deze opdracht ervoor weer. Chloros-schakelt een apparaat nooit zelfstandig weer in, omdat ‘uit’ opzettelijk kan zijn. Nieuw in 1.3.0.

## DAQ-E-apparaatinstellingen

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` toont de firmware van de sensor (en of deze Chloros-release een nieuwere versie bevat), het adres en hoe het aan dat adres is gekomen, de opgeslagen netwerkinstelling als deze afwijkt van de huidige, streaming bij het opstarten en PTP. `pool-set-network` start de sensor opnieuw op om de wijziging toe te passen en verbindt hem opnieuw met de instellingen die hij eerder had; `--no-reboot` slaat deze alleen op. Er moet een statisch adres op het netwerk van deze computer aanwezig zijn, anders weigert Chloros het; `--force` negeert dat, maar alleen voor een computer die opnieuw geadresseerd of ernaar gerouteerd zal worden. Herstarten wordt geweigerd terwijl de sensor gegevens registreert. `pool-update-firmware` is het equivalent van **Update to \<version\>** in de GUI voor de opdrachtregel, en de manier om een DAQ-E op Linux bij te werken, waar geen GUI beschikbaar is. Nieuw in 1.3.0.

***

## Overzicht van commando’s

| Commando | Doel |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Een sensor openen in de backend-pool. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Scannen naar sensoren zonder verbinding te maken (zoekt het BLE-MAC-adres van een DAQ-M). |
| `daq pool-list` | Toon elke sensor in de pool met zijn `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De meest recente N gekalibreerde spectrumframes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Streaming hervatten / pauzeren. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Een `.daq`-opname starten / stoppen (aan de backend-zijde). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Het cap-correctieprofiel tijdens de uitvoering wisselen. |
| `daq pool-disconnect --sensor-id ID [--all]` | Eén sensor of alle sensoren vrijgeven. |
| `daq pool-enable-ptp --sensor-id ID` | PTP weer inschakelen voor een DAQ-E waarvan PTP is uitgeschakeld. |
| `daq pool-info --sensor-id ID [--json]` | De firmware, het adres, de opgeslagen netwerkinstellingen, streaming bij het opstarten en PTP van een DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP of een statisch adres; start de sensor opnieuw op en maakt opnieuw verbinding. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Streamen bij het opstarten. |
| `daq pool-reboot --sensor-id ID` | Een DAQ-E opnieuw opstarten en opnieuw verbinden. |
| `daq pool-update-firmware --sensor-id ID` | De DAQ-E-firmware van deze release op een oudere sensor flashen. |

***

## Problemen met de eerste verbinding van een DAQ-E oplossen

1. De DAQ-E heeft geen status-LED — controleer of er stroom is via de PoE/link-indicator op de switch of de injectorpoort, en wacht na het inschakelen enkele seconden totdat het apparaat is opgestart en verbinding heeft gemaakt met het netwerk.
2. De backend-machine moet zich in **hetzelfde broadcastdomein** bevinden als de sensor — mDNS gaat niet door routers heen.
3. Een firewall moet mDNS (UDP 5353), DAQ-E-gegevens (UDP 5002) en PTP (UDP 319/320) doorlaten. Het Windows-installatieprogramma voegt hiervoor een regel toe, en het Linux-programma `.deb` opent deze poorten in een `ufw` / `firewalld` die actief was op het moment van installatie; een firewall van een derde partij, of een die later is ingeschakeld, moet deze toestaan.
4. Nog steeds geen reactie van `--eth`? Gebruik `--eth-host` met de hostnaam (`daq-e-<id>.local`) of het IP-adres van het apparaat, waardoor het detectieproces wordt overgeslagen.

***
{% hint style="info" %}
**Tip voor AI-assistenten.** Elke pagina van deze handleiding wordt als onbewerkte Markdown aangeboden — voeg `.md` toe aan de URL-slug in kleine letters van een pagina (deze pagina: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); de machinaal leesbare index is dan `https://mapir.gitbook.io/chloros/llms.txt`. Voor volledige documentatie op vlaggenniveau van `chloros-cli daq` en alle andere opdrachtenfamilies, raadpleeg de [CLI-referentie](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); het Python-pad is `chloros_sdk.connect_daq_sensor()` in de [SDK-referentie](../reference/sdk-reference.md).
{% endhint %}
