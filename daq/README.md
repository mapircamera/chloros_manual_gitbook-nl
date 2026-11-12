# DAQ-lichtsensoren

> **Op zoek naar de hardware?**De sensoren zelf — modellen, bevestiging, afdekkappen, poorten, voeding en de SCANNER-app — worden beschreven in de**[DAQ-gebruikershandleiding](https://mapir.gitbook.io/daq)**. Dit hoofdstuk behandelt het gebruik ervan vanuit Chloros. De**DAQ**-lichtsensoren van

MAPIR meten omgevingslicht als radiometrisch gekalibreerde spectra. In Chloros vervullen ze twee functies:

* **Een op zichzelf staand spectraal instrument** — live spectrumgrafieken, colorimetrische gegevens en `.daq`-opnames, allemaal via het [tabblad Lichtsensoren](gui.md), de [CLI](cli-quick-start.md) of de Python SDK.
* **Een bron van neerwaartse straling voor reflectantie** — tijdens de verwerking interpoleert Chloros je `.daq`-metingen naar het belichtingstijdstip van elke opname en gebruikt het gemeten neerwaartse licht om de straling van de camera om te zetten in reflectantie (`--reflectance-source daq`); voor gekalibreerde banden is geen paneel ter plaatse nodig.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Drie modellen, één gegevensformaat

| Model | Overdracht | Detectie |
| --- | --- | --- |
| **DAQ-U** | USB (serieel) | scan via seriële poort |
| **DAQ-M** | Bluetooth Low Energy | BLE-scan op naam |
| **DAQ-E** | Ethernet (IPv4, PoE-voeding) | mDNS `_daq-e._tcp` (hostnaam `daq-e-<id>.local`) |

**DAQ-E-S (nieuw in 1.3.0).** Een DAQ-E met een vaste diffusor en een kantelsensor. Chloros vermeldt dit apparaat als `DAQ-E-S` en het werkt precies zoals een DAQ-E, behalve dat er geen cap-correctie wordt toegepast en elke meting de kantel-, rol- en pitchgegevens van het apparaat bevat. Zie [DAQ-E-S: vaste diffusor en kanteling](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Alle drie gebruiken hetzelfde communicatieprotocol en leveren identieke gegevens:

* Een **spectrum met 135 meetpunten van 340 tot 1010 nm in stappen van 5 nm**, plus CIE XYZ-tristimuluswaarden, in elk frame.
* **Radiometrisch gekalibreerde spectrale stralingsintensiteit in W/m²/nm** — de fabriekskalibratieset van elk apparaat (plus het bijbehorende actieve cap-correctieprofiel) wordt toegepast voordat de gegevens bij u terechtkomen.
* Hetzelfde **`.daq`-opnameformaat** (een SQLite-bestand). De verdere verwerking is identiek, ongeacht via welk transportprotocol het bestand is gegenereerd.

De transportstacks (USB-serieel, BLE, mDNS/zeroconf) zijn gebundeld in de Chloros-backend — er hoeft niets te worden geïnstalleerd om via de GUI of de `pool-*`-opdrachten van de CLI met een van de drie modellen te communiceren.

***

## Gekalibreerd bereik: 340–1010 nm gerapporteerd, ~374–974 nm gekalibreerd

De sensor rapporteert het volledige raster van 340–1010 nm, maar de NIST-traceerbare radiometrische versterking beslaat ongeveer **374–974 nm**. Chloroseert de absolute-reflectantieverdeling voor elke cameraband waarvan minder dan de helft van het spectrale gewicht binnen dat gekalibreerde bereik ligt; de overgeslagen band wordt gerapporteerd met de overslaagreden `dls-uncalibrated-band-<nm>`.

Van de geleverde LATTICE-filter-SKU’s is alleen **F988** hierdoor beïnvloed:

De reflectantie van F988 wordt gekalibreerd met behulp van een reflectantiepaneel ter plaatse: de band ligt buiten het gekalibreerde bereik van de DAQ-lichtsensor, dus Chloros past uw meest recente paneelopname toe en behoudt deze tussen de paneelmetingen door.

Als een F988-opname wordt verwerkt terwijl alleen DAQ-gegevens beschikbaar zijn, wijst Chloros de op DAQ gebaseerde reflectantie voor die band af met overslaagreden `dls-uncalibrated-band-988` — de [reflectantiepaneel-workflow](../calibration-targets.md) is de ondersteunde methode voor F988.

***

## Sensor-ID’s

Elke DAQ rapporteert een stabiele sensor-ID. De vorm ervan verschilt per model:

| Model | Vorm ID | Voorbeeld |
| --- | --- | --- |
| DAQ-U | 5-octet met koppeltekens | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 octetten met koppeltekens | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

De sensor-ID is:

* ingebed in elk `.daq`-bestand dat het registreert,
* de sleutel die Chloros gebruikt om de fabriekskalibratiebundel van dat apparaat op te halen,
* de waarde die je doorgeeft aan `--sensor-id` in de CLI-opdrachten voor `pool-*`, en
* voor de DAQ-E ook de mDNS-hostnaam (`daq-e-def330.local`) — de waarde die `--eth-host` accepteert.

***

## Fabriekskalibratie en de cloud

Elke DAQ-eenheid wordt afzonderlijk in de fabriek gekalibreerd met een naar NIST herleidbare radiometrische keten, en Chloros laadt voor elke eenheid het bijbehorende kalibratiebundel op basis van de sensor-ID. Het kalibratierapport per eenheid (PDF) kan worden gedownload via de instellingen van de sensor op het [tabblad Lichtsensoren](gui.md).

{% hint style="warning" %}
**Voor DAQ-U en DAQ-M is cloudtoegang vereist voor kalibratie.**Geen van beide modellen slaat iets op het apparaat zelf op: hun fabriekskalibratiebundels staan in de cloud vMAPIR en worden opgehaald op basis van de sensor-ID (en vervolgens lokaal in de cache opgeslagen). Chloros heeft een internetverbinding nodig om gekalibreerde W/m²/nm-gegevens van een DAQ-U of DAQ-M te kunnen leveren.**DAQ-E vormt hierop een uitzondering** — deze heeft zijn kalibratie op het apparaat zelf opgeslagen.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Waar opnames terechtkomen

| Oppervlak | Standaard `.daq`-bestemming |
| --- | --- |
| GUI — tabblad Lichtsensoren | `<project folder>/light_sensor/` (voltooide opnames worden automatisch aan het project toegevoegd) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` op de machine waarop de backend draait (een relatief `-o`-pad wordt omgezet naar de map waarin je de opdracht hebt uitgevoerd wanneer de backend op dezelfde computer draait) |

Elke `.daq`-bestandsnaam bevat de sensor-ID en een tijdstempel.

***

## In dit hoofdstuk

* [**Het tabblad DAQ in Chloros**](gui.md) — de volledige GUI-handleiding: het aansluiten van elk model, instellingen per sensor, spectrumgrafieken, live colorimetrische gegevens, reflectantie met twee sensoren en opname.
* [**Snelstart CLI (pool-\*)**](cli-quick-start.md) — DAQ-sensoren aansturen vanuit `chloros-cli daq pool-*`, het ondersteunde commandoregelpad.
* [**Cap-profielen & gekalibreerd bereik**](caps-and-range.md) — welke caps er per model bestaan, hoe je ze kunt definiëren en het gekalibreerde spectrale bereik in detail.
* [**Opname & het .daq-formaat**](recording.md) — het `.daq` SQLite-formaat en opnameworkflows.
* [**DAQ-E-netwerken en tijdsynchronisatie**](ethernet-ptp.md) — DAQ-E-transportmodi en PTP-tijdsynchronisatie.
* [**Reflectiewerkprocessen**](reflectance.md) — het gebruik van DAQ-downwelling-gegevens om reflectie te berekenen.
* Zie voor volledige documentatie op vlaggenniveau de [CLI-referentie](../reference/cli-reference.md) (sectie `chloros-cli daq`) en de [SDK-referentie](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), beide geschreven om direct door AI-assistenten te kunnen worden gebruikt.
