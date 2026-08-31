# DAQ-lichtsensoren

> **Op zoek naar de hardware?**De sensoren zelf — modellen, bevestiging, afdekkappen, poorten, voeding en de SCANNER-app — worden beschreven in de**[DAQ-gebruikershandleiding](https://mapir.gitbook.io/daq)**. In dit hoofdstuk wordt het gebruik ervan behandeld vanaf Chloros.

De **DAQ**-lichtsensoren van MAPIR meten omgevingslicht als radiometrisch gekalibreerde spectra. In Chloros vervullen ze twee functies:

* **Een op zichzelf staand spectraal instrument** — live spectrumgrafieken, colorimetrische gegevens en `.daq`-opnames, allemaal via het [tabblad Lichtsensoren](gui.md), de [CLI](cli-quick-start.md) of de Python SDK.
* **Een neerwaartse stralingsbron voor reflectantie** — tijdens de verwerking interpoleert Chloros uw `.daq`-metingen naar de tijdstempel van elke opnametijdstempel van de opname en gebruikt het gemeten neerwaartse licht om de straling van de camera om te zetten in reflectie (`--reflectance-source daq`); voor gekalibreerde banden is geen paneel in de scène nodig.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Drie modellen, één gegevensformaat

| Model | Transport | Discovery |
| --- | --- | --- |
| **DAQ-U** | USB (serieel) | scan via seriële poort |
| **DAQ-M** | Bluetooth Low Energy | BLE-scan op naam |
| **DAQ-E** | Ethernet (IPv4, PoE-voeding) | mDNS `_daq-e._tcp` (hostnaam `daq-e-<id>.local`) |

Alle drie gebruiken ze hetzelfde netwerkprotocol en leveren identieke gegevens:

* Een **spectrum met 135 meetpunten van 340 tot 1010 nm in stappen van 5 nm**, plus CIE XYZ-tristimuluswaarden, in elk frame.
* **Radiometrisch gekalibreerde spectrale stralingsintensiteit in W/m²/nm** — de fabriekskalibratieset van elk apparaat (plus het bijbehorende actieve cap-correctieprofiel) wordt toegepast voordat de gegevens u bereiken.
* Hetzelfde **`.daq`-opnameformaat** (een SQLite-bestand). De verdere verwerking is identiek, ongeacht via welk transportprotocol het bestand is gegenereerd.

De transportstacks (USB-serieel, BLE, mDNS/zeroconf) zijn gebundeld in de Chloros-backend — er hoeft niets te worden geïnstalleerd om via de GUI of de CLI-opdrachten met een van de drie modellen te communiceren.

***

## Gekalibreerd bereik: 340–1010 nm gerapporteerd, ~374–974 nm gekalibreerd

De sensor rapporteert het volledige raster van 340–1010 nm, maar de NIST-traceerbare radiometrische versterking beslaat ongeveer **374–974 nm**. Chloros weigert de absolute-reflectiedeling voor elke cameraband waarvan minder dan de helft van het spectrale gewicht binnen dat gekalibreerde bereik valt; de overgeslagen band wordt gerapporteerd met overslaagreden `dls-uncalibrated-band-<nm>`.

Van de leverbare LATTICE-filter-SKU’s is alleen **F988** hierdoor getroffen:

De reflectie van F988 wordt gekalibreerd met behulp van een reflectiepanel ter plaatse: de band ligt buiten het gekalibreerde bereik van de DAQ-lichtsensor, dus Chloros past uw meest recente panelmeting toe en behoudt deze tussen de panelmetingen door.

Als een F988-opname wordt verwerkt terwijl alleen DAQ-gegevens beschikbaar zijn, wijst Chloros de op DAQ gebaseerde reflectantie voor die band af met overslaagreden `dls-uncalibrated-band-988` — de [reflectantiepaneel-workflow](../calibration-targets.md) is de ondersteunde werkwijze voor F988.

***

## Sensor-ID’s

Elke DAQ rapporteert een stabiele sensor-ID. De vorm ervan verschilt per model:

| Model | ID-vorm | Voorbeeld |
| --- | --- | --- |
| DAQ-U | 5 octetten met koppeltekens | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 octetten met koppeltekens | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

De sensor-ID is:

* ingebed in elk `.daq`-bestand dat het registreert,
* de sleutel die Chloros gebruikt om de fabriekskalibratiebundel van dat apparaat op te halen,
* de waarde die u doorgeeft aan `--sensor-id` in de CLI `pool-*`-opdrachten, en
* voor de DAQ-E ook de mDNS-hostnaam (`daq-e-def330.local`) — de waarde die `--eth-host` accepteert.

***

## Fabriekskalibratie en de cloud

Elke DAQ-eenheid wordt afzonderlijk in de fabriek gekalibreerd met een naar NIST herleidbare radiometrische keten, en Chloros laadt de kalibratiebundel van elke eenheid op basis van de sensor-ID. Het kalibratierapport per eenheid (PDF) kan worden gedownload via de instellingen van de sensor op het [tabblad Lichtsensoren](gui.md).

{% hint style="warning" %}
**DAQ-U en DAQ-M vereisen cloudtoegang voor kalibratie.**Geen van beide modellen slaat iets op het apparaat zelf op: hun fabriekskalibratiebundels bevinden zich in de cloud van MAPIR en worden opgehaald op basis van de sensor-ID (en vervolgens lokaal in de cache opgeslagen). Chloros heeft een internetverbinding nodig om gekalibreerde W/m²/nm-gegevens van een DAQ-U of DAQ-M te kunnen leveren.**DAQ-E vormt hierop een uitzondering** — dit apparaat heeft zijn kalibratie opgeslagen op het apparaat zelf.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Waar opnames terechtkomen

| Oppervlak | Standaardbestemming voor `.daq` |
| --- | --- |
| GUI — tabblad Lichtsensoren | `<project folder>/light_sensor/` (voltooide opnames worden automatisch aan het project toegevoegd) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` op de machine waarop de backend draait |

Elke `.daq`-bestandsnaam bevat de sensor-ID en een tijdstempel.

***

## In dit hoofdstuk

* [**Het tabblad DAQ in Chloros**](gui.md) — de volledige GUI-handleiding: het aansluiten van elk model, instellingen per sensor, spectrumgrafieken, live colorimetrische gegevens, reflectie met twee sensoren en opname.
* [**CLI Snelstart (pool-\*)**](cli-quick-start.md) — het aansturen van DAQ-sensoren vanuit `chloros-cli daq pool-*`, het ondersteunde commandoregelpad.
* [**Cap-profielen &amp; gekalibreerd bereik**](caps-and-range.md) — welke caps er per model bestaan, hoe je ze kunt specificeren en het gekalibreerde spectrale bereik in detail.
* [**Opname &amp; het .daq-formaat**](recording.md) — het `.daq` SQLite-formaat en opnameworkflows.
* [**DAQ-E-netwerken en tijdsynchronisatie**](ethernet-ptp.md) — DAQ-E-transportmodi en PTP-tijdsynchronisatie.
* [**Reflectiewerkprocessen**](reflectance.md) — gebruik van DAQ-downwelling-gegevens om reflectie te genereren.
* Zie voor volledige documentatie op vlaggenniveau de [CLI-referentie](../reference/cli-reference.md) (sectie `chloros-cli daq`) en de [SDK-referentie](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), beide geschreven om direct door AI-assistenten te kunnen worden gebruikt.
