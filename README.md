---
metaLinks: {}
---

# Aan de slag

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros is een softwareapplicatie van [MAPIR](https://www.mapir.camera) voor het verwerken van multispectrale beelden, het live aansturen van MAPIR-hardware en het vastleggen van sensorgegevens. Chloros 1.2.0 ondersteunt de volledige MAPIR-productfamilie:

* **Survey3-camera’s** — verwerken RAW+JPG-opnames tot gekalibreerde reflectantie- en vegetatie-indexkaarten. Zie [Ondersteunde camera’s](supported-cameras.md).
* **LATTICE-camera’s** — sluit de GigE-multispectrale cameramodules live aan, afzonderlijk of als gesynchroniseerde multi-camera-opstellingen: bekijk een voorbeeld, leg beelden vast en verwerk deze tot gekalibreerde stralings- en reflectieproducten. Zie de [LATTICE-sectie](lattice/README.md).
* **DAQ-lichtsensoren** — DAQ-U (USB), DAQ-M (Bluetooth) en DAQ-E (Ethernet) spectrale sensoren: live gekalibreerde spectra, `.daq`-opnames en neerwaartse verlichting voor reflectieverwerking. Zie de [DAQ-sectie](daq/README.md).

{% hint style="success" %}
**Nieuw in Chloros 1.2.0**: live besturing van LATTICE-camera&#x27;s en -arrays, integratie van DAQ-lichtsensoren, opnamemodi en recorders, een volledige radiometrische verwerkingspijplijn voor LATTICE, projectautomatisering vanuit de CLI/SDK, en nog veel meer. Bekijk de lijst met nieuwe functies hieronder en [download](download.md) voor het wijzigingslogboek.
{% endhint %}

{% hint style="info" %}
**Gebruik je Chloros met een AI-assistent?** Deze handleiding is daar speciaal voor gemaakt. Laat je assistent het volgende opzoeken:

* `https://mapir.gitbook.io/chloros/llms.txt` — machine-leesbare index van elke pagina.
* Elke pagina als onbewerkte Markdown — voeg `.md` toe aan de URL (bijv. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* De [CLI-referentie](reference/cli-reference.md) en [SDK-referentie](reference/sdk-reference.md) — volledige referentiepagina’s met exacte waarden, geschreven voor gebruik door LLM’s.

Voorbeeldprompt: *&quot;Lees https://mapir.gitbook.io/chloros/reference/cli-reference.md, en schrijf vervolgens een script dat inlogt en de map ~/flights/flight_001 verwerkt tot GeoTIFFs met reflectantie + NDVI.&quot;*

Volledige handleiding: [Chloros gebruiken met AI-assistenten](ai-assistants.md).
{% endhint %}

***

## Wat is er nieuw in Chloros 1.2.0

* **Live camerabesturing — nieuw tabblad ‘Camera’s’.** Sluit LATTICE-camera’s één voor één aan of als gesynchroniseerde opstellingen met meerdere camera’s (PTP-tijdsynchronisatie, door hardware geactiveerde opname), met live-preview-overlays, histogrammen per band, slimme automatische belichting, een live-indexcalculator en firmware-updates voor camera’s via de app.
* **Lichtsensoren — nieuw tabblad ‘Lichtsensoren’.** Sluit DAQ-U (USB), DAQ-M (Bluetooth) en DAQ-E (Ethernet) sensoren aan; bekijk live gekalibreerde spectra (W/m²/nm), sla `.daq`-bestanden op in uw project, kies profielen voor cap-correctie en werk de DAQ-E-firmware bij via het netwerk.
* **Opnamemodi en recorders.** Enkelvoudige / Continue / Intervalopname plus een ‘Fastest Capture’-modus uitsluitend voor raw-gegevens; per project selectie van welke camera’s en exporttypes ‘Capture All’ oplevert; array-recorders voor indexvideo van monitoringkwaliteit en raw-bursts van analysekwaliteit met offline videobouw.
* **LATTICE-verwerkingspijplijn.** Importeer LATTICE-opnamemappen en splits elk onbewerkt frame op in debayered, preview, float32-radiance (W/m²/sr/nm) en reflectantieproducten met schakelaars per product. Reflectantie kan afkomstig zijn van een kalibratiedoel binnen het frame of van DAQ-downwelling; array-uitlijning wordt toegepast op exportbestanden; ontbrekende fabriekskalibratie wordt automatisch gedownload op basis van het serienummer van de camera.
* **Projecten onthouden hardware.** Aangesloten camera’s en lichtsensoren worden samen met het project opgeslagen (`cameras.json` / `sensors.json`) en maken opnieuw verbinding met hun opgeslagen instellingen wanneer u het project heropent. Zie [GUI: Projecten](projects.md).
* **Verbeteringen aan de beeldviewer.** Weergave van cursorpixel/index met correcte reflectieschaling per bestand, laaghistogrammen, een GSD-binning-schuifregelaar, rastermodi ‘Per Trigger’ en ‘Per Camera’, LATTICE-productweergaven en het exporteren van index/LUT-sandbox-bestanden naar schijf.
* **CLI &amp; SDK, aanzienlijk uitgebreid.** Nieuwe `lattice`-, `daq pool-*`-, `project`- en `time-sync`-opdrachtfamilies; nieuwe `process`-opties (`--input-level`, schakelaars per product, `--reflectance-source`, vlaggen voor array-uitlijning); SDK smart-connect-handles (`connect_camera` / `connect_array` / `connect_daq_sensor`) die de backend automatisch starten; `open_project()`-automatisering; het SDK-wiel is gebundeld met de installatieprogramma’s en gepubliceerd op PyPI als `chloros-sdk`.
* **Eerlijke foutmeldingen.** Een `chloros-cli process`-run die producten heeft aangevraagd maar geen enkel product heeft geschreven, mislukt nu met een duidelijke foutmelding en stopt met een uitkomst anders dan nul; succesvolle runs rapporteren hoeveel beeldproducten ze hebben geschreven.
* **Nieuwe uitvoerindeling.** Producten worden opgeslagen in `<project>/<camera>/<format>/<Product>_Images/`-mappen en behouden de bronbestandsnaam — de map, niet een bestandsextensie, identificeert het product. Zie [Uitvoerbeeldformaten](output-image-formats.md).
* **Meer invoeropties, abonnementen en talen.** Ondersteuning voor `.dng`-invoer; alle 38 interfacetalen zijn volledig beschikbaar; hardwarebeperkingen per abonnement met gratis (zonder inloggen) gebruik van maximaal 4 camera’s en 2 lichtsensoren.
* **Betrouwbaarheid.** ‘Stop Processing’ wordt netjes afgesloten met een eerlijk uitvoeringsoverzicht, bij projecten met meerdere camera’s worden alle camera’s geëxporteerd en bij upgrades van het installatieprogramma word je niet langer uitgelogd.***

Chloros is beschikbaar in 3 applicatieomgevingen:

## Chloros: Desktop-GUI-applicatie

Zelfstandig apart venster met alle functies, inclusief de tabbladen ‘Live camera’s’ en ‘Lichtsensoren’. _Alleen voor Windows._

## [Chloros CLI: Opdrachtregelinterface](CLI.md)

Batchverwerking via de opdrachtregel plus live-opdrachten `lattice`, `daq pool-*`, `project` en `time-sync`. Perfect voor automatisering, scripting en headless-werking. Beschikbaar op **Windows, Linux amd64 en Linux arm64 (NVIDIA Jetson)**. _Toegang tot CLI vereist een betaald Chloros+-abonnement._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmatische Python-interface voor automatisering en aangepaste workflows: volledige pijplijnverwerking, live camera-/array-sessies, DAQ-sensorsessies en automatisering van opgeslagen projecten. Geïnstalleerd met het desktop-/CLI-pakket en ook gepubliceerd als `pip install chloros-sdk`. _Voor toegang tot de API is een betaald Chloros+-abonnement vereist._

***

## Ondersteunde platforms

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nee | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nee | Ja | Ja |

Raadpleeg het gedeelte [Linux &amp; Edge Computing](linux/linux-overview.md) voor installatie-instructies voor Linux.

***

## Aan de slag in drie stappen

1. **Installeren** — download en voer het installatieprogramma voor uw platform uit. Zie [Download](download.md).
2. **Inloggen (optioneel voor de GUI)** — de GUI verwerkt afbeeldingen gratis zonder account. Met een [Chloros+-login](chloros+-login.md) krijgt u toegang tot parallelle verwerking, GPU-versnelling, hogere apparaatlimieten en toegang tot CLI/SDK.
3. **Maak je eerste project aan** — open Chloros, maak een [Nieuw project](projects.md), [voeg je afbeeldingen toe](processing-images-gui/adding-files-to-a-project.md) en [begin met verwerken](processing-images-gui/starting-the-processing.md). Als je in plaats daarvan live hardware wilt aansturen, open dan het tabblad ‘Camera’s’ of ‘Lichtsensoren’ — zie [GUI: Navigatie](navigation.md).***

## Chloros+

Hoewel Chloros voor de meeste taken gratis te gebruiken is, kan het zijn dat je meer mogelijkheden wilt. Dan kan een betaalde licentie voor Chloros+ je van pas komen. Met een Chloros+-licentie kun je nieuwe functies ontgrendelen, zoals:

* **Multi-threaded verwerking**: versnel de beeldverwerking voor grotere projecten aanzienlijk door beelden gelijktijdig via de pijplijn te verwerken.
* **GPU (CUDA)-versnelling**: profiteer van de huidige opties voor meer GPU-geheugen om de beeldverwerkingspijplijn nog verder te versnellen. Voor het beste resultaat raden we 4 GB of meer VRAM aan.
* **Chloros+**[**CLI**](CLI.md)**Toegang**: voer Chloros+ uit vanaf de opdrachtregel om het te automatiseren en te integreren in je eigen software. Beschikbaar op elk betaald abonnement; wordt serverzijde afgedwongen.
* **Chloros+**[**API**](api-python-sdk.md)**Toegang:** voer Chloros+ uit vanuit Python voor programmatische besturing, waardoor naadloze integratie met uw onderzoekspijplijnen, workflows voor gegevensanalyse en aangepaste applicaties mogelijk wordt. Beschikbaar op elk betaald abonnement; wordt server-side afgedwongen.
* **Hogere hardware-limieten**: sluit meer camera’s en lichtsensoren tegelijk aan. Zonder inloggen sluit de GUI maximaal 4 camera’s en 2 DAQ-lichtsensoren aan; betaalde abonnementen verhogen beide limieten:

| Abonnement | Camera’s | DAQ-lichtsensoren |
| --- | --- | --- |
| Iron (gratis, geen aanmelding) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Gebruik op meerdere apparaten**: met elke Chloros+-licentie kunnen 2 of meer apparaten worden geregistreerd. Gebruik je MAPIR Cloud-account om geregistreerde apparaten te beheren. Voeg ondersteuning voor meer apparaten toe door uw Chloros+-licentie te upgraden.
* **Geavanceerde textuurbewuste debayer-methode:** een hoogwaardige, randbewuste debayer gecombineerd met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayer-ruis verwijdert.
* **Aangepaste multispectrale indexformules:** voer aangepaste multispectrale indices in de Chloros-rastercalculators in, zowel voor verwerking als voor de sandbox voor het bekijken van afbeeldingen.
* **Linux &amp; edge computing:** draai Chloros op Linux x86_64- en ARM64-platforms, waaronder NVIDIA Jetson, voor veld- en edge-verwerking. Zie [Linux Overzicht](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Prijzen en aanmelding</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
