---
metaLinks: {}
---

# Aan de slag

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros is een softwareapplicatie van [MAPIR](https://www.mapir.camera) voor het verwerken van multispectrale beelden, het live aansturen van MAPIR-hardware en het vastleggen van sensorgegevens. Chloros 1.3.0 ondersteunt de volledige MAPIR-productfamilie:

* **Survey3-camera’s** — verwerk RAW+JPG-opnames tot gekalibreerde reflectantie- en vegetatie-indexkaarten. Zie [Ondersteunde camera’s](supported-cameras.md).
* **LATTICE-camera’s** — sluit de GigE-multispectrale cameramodules live aan, afzonderlijk of als gesynchroniseerde opstellingen met meerdere camera’s: bekijk een voorbeeld, leg beelden vast en verwerk deze tot gekalibreerde stralings- en reflectieproducten. Zie de [LATTICE-sectie](lattice/README.md).
* **DAQ-lichtsensoren** — DAQ-U (USB), DAQ-M (Bluetooth) en DAQ-E en DAQ-E-S (Ethernet) spectrale sensoren: live gekalibreerde spectra, `.daq`-opnames en neerwaartse verlichting voor reflectieverwerking. Zie de [DAQ-sectie](daq/README.md).

{% hint style="success" %}
**Nieuw in Chloros 1.3.0**: AI-assistenten kunnen Chloros aansturen via `chloros-mcp`, en de CLI, de SDK en `chloros-mcp` bieden dezelfde functies. De LATTICE-indexproducten zijn verbeterd, volledige sensorarrays werken sneller en op elk netwerk, en de backend blijft tussen commando’s door draaien. Bekijk de lijst met nieuwe functies hieronder en [download](download.md) de installatieprogramma’s.
{% endhint %}

{% hint style="info" %}
**Gebruik je Chloros met een AI-assistent?** Deze handleiding is daar speciaal voor gemaakt. Laat je assistent het volgende opzoeken:

* `https://mapir.gitbook.io/chloros/llms.txt` — machine-leesbare index van elke pagina.
* Elke pagina als ruwe Markdown — voeg `.md` toe aan de URL (bijv. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* De [CLI-referentie](reference/cli-reference.md) en [SDK-referentie](reference/sdk-reference.md) — volledige referentiepagina’s met exacte waarden, geschreven voor gebruik door LLM’s.

Voorbeeldprompt: *"Lees https://mapir.gitbook.io/chloros/reference/cli-reference.md, en schrijf vervolgens een script dat inlogt en de map ~/flights/flight_001 verwerkt tot GeoTIFFs met reflectantie en NDVI."*

Of laat het Chloros direct uitvoeren: `chloros-mcp`, een MCP-server die met de SDK is geïnstalleerd, werkt samen met Claude Desktop, Claude Code, Cursor en andere MCP-clients.

Volledige handleiding: [Chloros gebruiken met AI-assistenten](ai-assistants.md).
{% endhint %}

***

## Wat is er nieuw in Chloros 1.3.0

* **AI-assistenten kunnen Chloros aansturen.** `chloros-mcp`, een MCP-server die met de Python-SDK is geïnstalleerd, stelt Claude Desktop, Claude Code, Cursor en andere MCP-clients in staat om beeldmateriaal te verwerken en LATTICE-camera’s, arrays en DAQ-lichtsensoren te bedienen. Zie [Chloros gebruiken met AI-assistenten](ai-assistants.md).
* **De CLI, de SDK en `chloros-mcp` bieden dezelfde functies.** Alleen het inloggen, het instellen van DAQ-E en de directe DAQ-servers blijven om veiligheidsredenen bij de CLI. Sommige 1.2.0-scripts moeten worden aangepast: zie **Gewijzigd in 1.3.0** bovenaan de [CLI-referentie](reference/cli-reference.md) en de [SDK-referentie](reference/sdk-reference.md).
* **Verbeterde LATTICE-indexproducten.** NDVI en de andere indexproducten die uit LATTICE-opnames worden geëxporteerd, zijn in elke verwerkingsmodus verbeterd.
* **Windows-installatieopties.**Onder**Systeeminstellingen**biedt het installatieprogramma de opties**Lange Windows-paden inschakelen**en**De Chloros-backend uitsluiten van Microsoft Defender-scans**, die beide standaard zijn aangevinkt. De eerste optie zorgt ervoor dat projectmappen dieper dan 260 tekens kunnen liggen; de tweede voorkomt dat realtime-scans de cameravastlegging vertragen (ongeveer 30% van de vastleggingssnelheid bij hoge snelheden). Het installatieprogramma voegt nu ook de map CLI slechts één keer toe aan `PATH` en verwijdert de dubbele vermeldingen die eerdere installatieprogramma’s hadden achtergelaten. Zie [Download](download.md).
* **Diepe projectmappen worden direct geweigerd.** Als lange paden in Windows zijn uitgeschakeld (de standaardinstelling van Windows), wordt een projectmap die te diep is voor de padlimiet van 260 tekens geweigerd voordat er iets wordt uitgevoerd, waarbij de langste projectmap die wel past wordt getoond en wordt uitgelegd hoe lange paden kunnen worden ingeschakeld. Voorheen leverde een dergelijke uitvoering geen resultaten op. Als lange paden zijn ingeschakeld, behouden producten in zeer diepe mappen nu hun metagegevens (GPS- en XMP-tags).
* **Maskers afkappen.** Wanneer het ontmengen van banden van een LATTICE-radiance- of reflectantieproduct een negatieve bandwaarde oplevert, wordt die waarde afgekapt tot nul, en registreert een per-pixelmasker in een `clamp_masks/`-map naast de productmappen welke pixels en banden hierdoor zijn beïnvloed. Het product benoemt zijn masker in de `Chloros:UnmixClampMask`-tag. Liveweergaven worden nu precies op dezelfde manier afgekapt als bij de export. Zie [Uitvoerbeeldformaten](output-image-formats.md).
* **Producten beschrijven zichzelf.** Indexproducten hebben hun eigen verwerkingstags (`Chloros:ProcessingLevel=index`, de indexuitdrukking en of ze zijn berekend op basis van straling of reflectie) in plaats van die van de bron-raw-bestanden. Survey3-producten bevatten in elke verwerkingsmodus Chloros-verwerkingstags. LATTICE-raw- en debayered-bestanden behouden de oorspronkelijke oriëntatie van de sensor en slaan hun oriëntatie op in XMP; de Image Viewer draait ze alleen om voor weergave.
* **Linux-pakketten worden uitgepakt geleverd.** De backend en de CLI worden geïnstalleerd als mappen onder `/usr/lib/chloros/`, zodat ze niet langer bij elke start opnieuw worden uitgepakt. Op x86_64 doet de backend er niet langer ongeveer 100 seconden over om op te starten, en is de `.deb` ongeveer half zo groot als voorheen. De meegeleverde ExifTool werkt nu op een schone Ubuntu-installatie. De `.deb` installeert nu ook de Python-SDK voor het systeem `python3` (installeer deze met `sudo apt install ./chloros_1.3.0_<arch>.deb`), vergroot de socket-ontvangstbuffers van de kernel die camerastreams nodig hebben, en voegt een backend-service toe die onder uw gebruikersaccount draait. Het Jetson-pakket, `chloros_1.3.0_arm64_jp6.deb`, is gebouwd voor JetPack 6 en verwerkt op de GPU van de Jetson met de CUDA-build van PyTorch van JetPack. Op Linux bereikt elk geëxporteerd beeld nu één keer de schijf (eerst geschreven en getagd in het geheugen, daarna naar de juiste locatie gekopieerd): op een Jetson, waar de schijf het tempo bepaalt, duurt een LATTICE-run met 288 beelden 14 minuten in plaats van 24. Zie [Linux-installatie](linux/linux-installation.md).
* **LATTICE OCN-camera’s meten nauwkeuriger in zonlicht.** Straling, reflectie en indices krijgen na het ontmengen van de banden een snelle correctie, zodat een grijs doel in de zon een waarde oplevert die dicht bij een neutrale NDVI ligt. RGN-camera’s blijven ongewijzigd.
* **Bugrapporten vanuit de app.** Via een bugpictogram bovenaan het tabblad Debug Log worden een beschrijving, maximaal 10 beelden, de logbestanden en systeemgegevens naar MAPIR verzonden. Zie [GUI: Navigatie](navigation.md).
* **LATTICE-arrays.**Arrays behouden standaard de volledige sensor, en gepakt 12-bits transport (`--packed`, standaard `auto`) verhoogt de framesnelheden bij volledige sensor met ongeveer een derde; een array werkt op elk netwerk met de framesnelheid die het netwerk toelaat (een trage verbinding verlaagt de framesnelheid in plaats van de array te weigeren of de frames te verkleinen); een array waarvan de camera’s opnieuw zijn opgestart, herstelt zichzelf, waarbij PTP weer wordt ingeschakeld; automatische belichting verlaagt die snelheid niet langer bij weinig licht (het vult de periode op en verhoogt vervolgens de versterking); Continu opnemen houdt gelijke tred met snelle arrays; een**Highlights**-regelaar voor RGB-camera’s; en de SDK’s `ArraySession` krijgt `grab()`, `frame_stream()` en `continuous()`. Zie de [LATTICE-sectie](lattice/README.md).
* **Stabielere automatische belichting.** De belichting blijft constant bij gelijkblijvend licht en stabiliseert zich binnen enkele seconden na een verandering; kleine hoge lichten maken het beeld niet langer donkerder; bij handmatige belichting blijft de versterking op ‘Auto’ staan; en een camera die in fel zonlicht niet meer reageert, wordt op het niveau gehouden dat past bij het licht, gemeten door de lichtsensor of, indien er geen is aangesloten, door de eigen beelden van de camera. Zie [Camera-instellingen](lattice/camera-settings.md).
* **Lichtsensoren.** **DAQ-E-S-ondersteuning:** de vaste diffuser vereist geen correctie voor de kap, en elke meting bevat de kantel-, rol- en pitchgegevens van het apparaat. Een opname geeft aan of de kap is gekozen of verondersteld, en de verwerking geeft een waarschuwing bij een veronderstelde kap: DAQ-U, DAQ-M en DAQ-E worden zonder kap geleverd, dus geef de gemonteerde kap een naam. Live `.daq`-opnames blijven behouden bij een stroomstoring (bewaar een `.daq-journal`-bestand samen met de bijbehorende `.daq`), en een DAQ-E die stopt met verzenden, wordt opnieuw aangesloten met de onderbreking gemarkeerd in de opname. Een project dat alleen opnames van lichtsensoren bevat, geeft deze weer en verwerkt ze vanuit de app en de CLI, en een opname gemaakt met de `chloros_scripts` wordt weergegeven als de DAQ die deze heeft gemaakt, met een limiet die je kunt instellen. Elke opname slaat de DAQ-waarde op die is gebruikt. Zie de [DAQ-sectie](daq/README.md).
* **Verwerking.**LATTICE-frames beginnen met verwerken tijdens de detectie van doelen. Doel- en DAQ-referenties worden per band bepaald. Bij een verplaatst project worden de bestanden opnieuw gekoppeld. De exportbestanden van een 8-bits opname zijn 8-bits, in een nieuw**TIFF-formaat (8-bits)**. Een Survey3-run zonder kalibratiedoel wordt opgeslagen als het fallback-product (`Vignette_Corrected_Images` of `Sensor_Response_Images`), niet als reflectie. De DLS-tags per band van een reflectiebeeld zijn altijd die van het eigen frame. ‘Texture Aware’ wordt geweigerd voor LATTICE-opnames, die altijd de standaard debayer krijgen.
* **CLI & SDK.** Relatieve uitvoerpaden komen nu terecht in de map van waaruit je het commando of script uitvoert (een geïnstalleerde backend probeerde ze vroeger op te lossen in zijn eigen installatiemap, wat mislukte). De backend blijft actief tussen opdrachten en scripts en stopt zichzelf na 30 minuten inactiviteit; `chloros-cli backend status` en `backend stop` regelen dit. Opnamecommando’s geven de namen op van alle bestanden die ze niet konden schrijven en worden afgesloten met code 1; herhaalde opnames stoppen wanneer de schijf vol is. Een succesvolle uitvoering van `process` rapporteert `Image products written: N (plus M clamp masks)`. Op Windows zoeken de `ChlorosLocal` / `process_folder` van de SDK de geïnstalleerde backend (en gebruiken er een die al actief is), en `read_image_audit_tags` werkt vanuit een standaard `pip install`.

***

## Wat is er nieuw in Chloros 1.2.0

* **Live camerabesturing — nieuw tabblad ‘Camera’s’.** Sluit LATTICE-camera’s één voor één aan of als gesynchroniseerde multi-camera-opstellingen (PTP-tijdsynchronisatie, door hardware geactiveerde opname), met live-preview-overlays, histogrammen per band, slimme automatische belichting, een live-indexcalculator en in-app-firmware-updates voor camera’s.
* **Lichtsensoren — nieuw tabblad ‘Lichtsensoren’.** Sluit DAQ-U- (USB), DAQ-M- (Bluetooth) en DAQ-E- (Ethernet) sensoren aan; bekijk live gekalibreerde spectra (W/m²/nm), sla `.daq`-bestanden op in uw project, kies profielen voor cap-correctie en werk de DAQ-E-firmware bij via het netwerk.
* **Opnamemodi en recorders.** Enkelvoudige / continue / intervalopname plus een ‘Fastest Capture’-modus voor uitsluitend RAW-gegevens; per project selectie van welke camera’s en exporttypes ‘Capture All’ oplevert; array-recorders voor indexvideo van monitoringkwaliteit en RAW-bursts van analysekwaliteit met offline videobouw.
* **LATTICE-verwerkingspijplijn.** Importeer LATTICE-opnamemappen en splits elk onbewerkt frame op in debayered, preview, float32-radiance (W/m²/sr/nm) en reflectantieproducten met schakelaars per product. Reflectantie kan afkomstig zijn van een kalibratiedoel in het frame of van DAQ-downwelling; array-uitlijning wordt toegepast op exportbestanden; ontbrekende fabriekskalibratie wordt automatisch gedownload op basis van het serienummer van de camera.
* **Projecten onthouden hardware.** Aangesloten camera’s en lichtsensoren worden samen met het project opgeslagen (`cameras.json` / `sensors.json`) en maken opnieuw verbinding met hun opgeslagen instellingen wanneer u het project opnieuw opent. Zie [GUI: Projecten](projects.md).
* **Verbeteringen aan de afbeeldingsviewer.** Weergave van cursorpixel/index met correcte reflectieschaling per bestand, laaghistogrammen, een GSD-binning-schuifregelaar, rastermodi ‘Per Trigger’ / ‘Per Camera’, LATTICE-productweergaven en het exporteren van index/LUT-sandbox-bestanden naar schijf.
* **CLI en SDK aanzienlijk uitgebreid.** Nieuwe `lattice`-, `daq pool-*`-, `project`- en `time-sync`-opdrachtfamilies; nieuwe `process`-opties (`--input-level`, schakelaars per product, `--reflectance-source`, vlaggen voor array-uitlijning); SDK-handles voor slimme verbinding (`connect_camera` / `connect_array` / `connect_daq_sensor`) die de backend automatisch starten; `open_project()`-automatisering; het SDK-wiel is gebundeld met de installatieprogramma’s en gepubliceerd op PyPI als `chloros-sdk`.
* **Eerlijke foutmeldingen.** Een `chloros-cli process`-run die producten heeft aangevraagd maar er geen heeft geschreven, mislukt nu duidelijk en stopt met een uitkomst anders dan nul; succesvolle runs rapporteren hoeveel beeldproducten ze hebben geschreven.
* **Nieuwe uitvoerindeling.** Producten worden opgeslagen in `<project>/<camera>/<format>/<Product>_Images/`-mappen en behouden de bronbestandsnaam — de map, niet een bestandsextensie, identificeert het product. Zie [Uitvoerbeeldformaten](output-image-formats.md).
* **Meer invoeropties, abonnementen en talen.** Ondersteuning voor `.dng`-invoer; alle 38 interfacetalen volledig beschikbaar; hardwarebeperkingen per abonnement met gratis (zonder inloggen) gebruik van maximaal 4 camera’s en 2 lichtsensoren.
* **Betrouwbaarheid.** ‘Stop Processing’ wordt netjes afgesloten met een eerlijk uitvoeringsoverzicht, projecten met meerdere camera’s exporteren elke camera, en upgrades via het installatieprogramma zorgen er niet langer voor dat je wordt uitgelogd.***

Chloros is beschikbaar in 3 applicatieomgevingen:

## Chloros: Desktop-GUI-toepassing

Zelfstandig apart venster met alle functies, inclusief de tabbladen ‘Live camera’s’ en ‘Lichtsensoren’. _Alleen voor Windows._

## [Chloros CLI: Opdrachtregelinterface](CLI.md)

Batchverwerking via de opdrachtregel plus live `lattice`-, `daq pool-*`-, `project`- en `time-sync`-opdrachten. Perfect voor automatisering, scripting en headless-bediening. Beschikbaar op **Windows, Linux amd64 en Linux arm64 (NVIDIA Jetson)**. _Voor toegang tot de CLI is een betaald Chloros+-abonnement vereist._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmeerbare Python-interface voor automatisering en aangepaste workflows: volledige pijplijnverwerking, live camera-/array-sessies, DAQ-sensorsessies en automatisering van opgeslagen projecten. Geïnstalleerd met het desktop-/CLI-pakket en ook gepubliceerd als `pip install chloros-sdk`. _Voor toegang tot de API is een betaald Chloros+-abonnement vereist._

***

## Ondersteunde platforms

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nee | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nee | Ja | Ja |

Zie het gedeelte [Linux & Edge Computing](linux/linux-overview.md) voor installatie-instructies voor Linux.

***

## Aan de slag in drie stappen

1. **Installeren** — download en voer het installatieprogramma voor uw platform uit. Zie [Download](download.md).
2. **Inloggen (optioneel voor de GUI)** — de GUI verwerkt afbeeldingen gratis zonder account. Met een [Chloros+-login](chloros+-login.md) krijgt u toegang tot parallelle verwerking, GPU-versnelling, hogere apparaatlimieten en toegang tot de CLI en SDK.
3. **Maak je eerste project aan** — open Chloros, maak een [Nieuw project](projects.md) aan, [voeg je afbeeldingen toe](processing-images-gui/adding-files-to-a-project.md) en [begin met verwerken](processing-images-gui/starting-the-processing.md). Als je in plaats daarvan live hardware wilt aansturen, open dan het tabblad ‘Camera’s’ of ‘Lichtsensoren’ — zie [GUI: Navigatie](navigation.md).

***

## Chloros+

Hoewel Chloros voor de meeste taken gratis te gebruiken is, kan het zijn dat je meer wilt. Dan kan een betaalde licentie voor Chloros+ je voordelen bieden. Met een Chloros+-licentie kun je nieuwe functies ontgrendelen, zoals:

* **Multi-threaded verwerking**: versnel de beeldverwerking voor grotere projecten aanzienlijk door beelden gelijktijdig via de pijplijn te verwerken.
* **GPU (CUDA)-versnelling**: profiteer van de huidige opties voor meer GPU-geheugen om de beeldverwerkingspijplijn nog verder te versnellen. Voor de beste resultaten raden we 4 GB of meer VRAM aan.
* **Chloros+**[**CLI**](CLI.md) **Toegang**: voer Chloros+ uit vanaf de opdrachtregel om het te automatiseren en te integreren in je eigen software. Beschikbaar op elk betaald abonnement; wordt server-side afgedwongen.
* **Chloros+**[**API**](api-python-sdk.md) **Toegang:** voer Chloros+ uit vanuit Python voor programmatische besturing, waardoor naadloze integratie met uw onderzoekspijplijnen, gegevensanalyse-workflows en aangepaste applicaties mogelijk wordt. Beschikbaar op elk betaald abonnement; wordt server-side afgedwongen.
* **Hogere hardware-limieten**: sluit meer camera’s en lichtsensoren tegelijk aan. Zonder inloggen sluit de GUI maximaal 4 camera’s en 2 DAQ-lichtsensoren aan; betaalde abonnementen verhogen beide limieten:

| Abonnement | Camera’s | DAQ-lichtsensoren |
| --- | --- | --- |
| Iron (gratis, geen aanmelding) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Gebruik op meerdere apparaten**: met elke Chloros+-licentie kunnen 2 of meer apparaten worden geregistreerd. Gebruik je MAPIR Cloud-account om geregistreerde apparaten te beheren. Voeg ondersteuning voor meer apparaten toe door je Chloros+-licentie te upgraden.
* **Geavanceerde textuurbewuste debayer-methode:** een hoogwaardige, randbewuste debayer gecombineerd met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayer-ruis verwijdert.
* **Aangepaste multispectrale indexformules:** voer aangepaste multispectrale indices in de Chloros-rastercalculators in, zowel voor verwerking als voor de sandbox voor het bekijken van beelden.
* **Linux & Edge Computing:** draai Chloros op Linux x86_64- en ARM64-platforms, waaronder NVIDIA Jetson, voor veld- en edge-verwerking. Zie [Linux-overzicht](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Prijzen en aanmelding voor Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
