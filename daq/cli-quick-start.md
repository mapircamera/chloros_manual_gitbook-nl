# CLI Snelstartgids (pool-*)

De meegeleverde `chloros-cli`-drivers sturen DAQ-sensoren aan via de **`daq pool-*`** commandofamilie — lichte HTTP-clients die de sensor aansturen via de permanente sensorpool van de Chloros-backend. De backend beheert het transport, dus de GUI, de CLI en de SDK-scripts delen allemaal één live handle in plaats van te strijden om de poort. Alles wat een klant nodig heeft, is bereikbaar via `pool-*`: verbinding maken, streamen, gekalibreerde `.daq`-bestanden opnemen en cap-profielen wisselen.

`pool-*` is ook het **enige** DAQ-oppervlak in uitgebrachte builds. `chloros-cli daq --help` geeft een overzicht van de `pool-*`-subcommando’s, en het aanroepen van een direct-hardware DAQ-subcommando in een uitgebrachte build leidt tot een expliciete foutmelding waarin het ontbrekende pakket wordt genoemd en waarin je wordt verwezen naar `pool-*` — niets mislukt in stilte. (De direct-hardware-commando’s werken alleen vanuit een MAPIR-broncode-checkout; `pip install chloros-sdk` biedt ze evenmin aan.)

***

## Vereisten

* **De Chloros-backend moet actief zijn** — de `pool-*`-commando’s zijn HTTP-clients, geen hardwarestuurprogramma’s. Start op Windows de Chloros-desktopapp (deze start de backend). Schakel op een headless Linux/Jetson de service in: `sudo systemctl enable --now chloros-backend.service`.
* **Een Chloros+ (betaalde tier) aanmelding**: voer eerst `chloros-cli login` uit. De handhaving vindt plaats aan de serverzijde — zonder aanmelding mislukken opdrachten met `401 AUTH_REQUIRED`; op het gratis (Iron)-niveau mislukken ze met `403 PLAN_UPGRADE_REQUIRED`.
* De commando’s zijn standaard gericht op `http://127.0.0.1:5000`; de `daq pool-*`-familie houdt rekening met de omgevingsvariabele `CHLOROS_BACKEND_URL` als je backend elders draait.

***

## Een sessie van vijf minuten

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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
| `daq pool-connect` | Smart-detect: zoek elke DAQ op deze machine. |
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
| `--cap-id CAP` | Cap-correctieprofiel; standaardinstelling van de backend is `sunshine_cosine`. Zie [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**`--eth`-waarschuwing bij automatische detectie.** Op een host met meerdere netwerkinterfaces (meer dan één actieve netwerkinterface) kan de *eerste* `pool-connect --eth` na het opstarten leeg zijn, zelfs als de sensor in orde is — de detectieprocedure kan de interface van de sensor missen terwijl de ARP-cache nog niet is opgewarmd. Als `--eth` niets vindt, probeer het dan opnieuw, of sla de detectie helemaal over met `--eth-host <ip-or-hostname>`, wat de betrouwbare methode is op machines met meerdere netwerkverbindingen. De hostnaam van de DAQ-E is `daq-e-<id>.local` (bijv. `daq-e-def330.local`); het gewone IP-adres werkt ook.
{% endhint %}

## `pool-list` — bekijk wat er is aangesloten

Toont elke sensor in de backend-pool, inclusief de `sensor_id` die voor alle andere commando’s nodig is:

| Model | `sensor_id`-indeling | Voorbeeld |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-octet met koppeltekens | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — spectrumframes lezen

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Geeft het meest recente frame terug, of de meest recente `--recent N`-frames; `--json` genereert machinaal leesbare uitvoer voor scripting. Frames zijn radiometrisch gekalibreerde spectrale stralingsintensiteit (W/m²/nm) op het raster van 135 punten, 340–1010 nm, waarbij het kapprofiel van de sensor al is toegepast. Voor kwantitatieve stralingswaarden moet u het gemiddelde nemen van ten minste 15 seconden aan frames — dit is een kenmerk van het instrument, geen defect.

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Uitvoermap, bepaald **op de machine waarop de backend draait**. |
| `--device-name NAME` | — | Label dat bij de opname wordt opgeslagen. |
| `--stop` | — | Een lopende opname stoppen. |

{% hint style="info" %}
De opname vindt plaats in de backend, dus het `.daq`-bestand komt terecht op het bestandssysteem van de **backend-machine** — standaard in `~/Documents/DAQ Live View/` daar, niet noodzakelijkerwijs waar je het CLI-programma hebt uitgevoerd. Bestandsnamen bevatten de sensor-ID en een tijdstempel.
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

Een kap-ID die niet tot de set van de sensor behoort, wordt bij het aansluiten geweigerd met een duidelijke foutmelding. `none` (DAQ-E) betekent dat de kap fysiek is verwijderd — er wordt nog steeds een fabrieksgeometrieprofiel toegepast voor de verzonken glazen diffusor van de DAQ-E, dus het is geen no-op, en een ‘bare’ DAQ-E is een bench-configuratie, geen ondersteunde veldconfiguratie. (Een ‘naakte’ DAQ-U is echt ‘naakt’ en heeft helemaal geen correctieprofiel nodig; de DAQ-M wordt gebruikt met de bijbehorende Sunshine-kap.)

## `pool-disconnect` — sensoren vrijgeven

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Overzicht van commando’s

| Commando | Doel |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Een sensor in de backend-pool openen. |
| `daq pool-list` | Toon alle sensoren in de pool met hun `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De meest recente N gekalibreerde spectrumframes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Streaming hervatten / pauzeren. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Een `.daq`-opname starten / stoppen (backend-zijde). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Het cap-correctieprofiel tijdens runtime wisselen. |
| `daq pool-disconnect --sensor-id ID [--all]` | Eén sensor of alle sensoren vrijgeven. |

***

## Problemen oplossen bij een eerste verbinding met de DAQ-E

1. De DAQ-E heeft geen status-LED — controleer of de stroomtoevoer in orde is via de PoE/link-indicator op de switch of de injectorpoort, en wacht na het inschakelen enkele seconden totdat het apparaat is opgestart en verbinding heeft gemaakt met het netwerk.
2. De backend-machine moet zich in **hetzelfde broadcastdomein** bevinden als de sensor — mDNS gaat niet door routers heen.
3. Accepteer bij de eerste keer opstarten op Windows de Defender-firewallmelding (mDNS UDP 5353, DAQ-E-gegevens UDP 5002, PTP UDP 319/320).
4. Nog steeds geen reactie van `--eth`? Gebruik dan `--eth-host` met de hostnaam van het apparaat (`daq-e-<id>.local`) of het IP-adres — dit is de meest betrouwbare methode, vooral bij hosts met meerdere IP-adressen.

***{% hint style="info" %}**Tip voor AI-assistenten.** Elke pagina van deze handleiding wordt aangeboden als onbewerkte Markdown — voeg `.md` toe aan de kleine letters van de slug van een pagina (deze pagina: URL); de machinaal leesbare index is dan `https://mapir.gitbook.io/chloros/llms.txt`. Voor volledige documentatie op vlaggenniveau van `chloros-cli daq` en alle andere opdrachtenfamilies, haal de [CLI-referentie](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); het Python-pad is `chloros_sdk.connect_daq_sensor()` in de [SDK-referentie](../reference/sdk-reference.md).
{% endhint %}
