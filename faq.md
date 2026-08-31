---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Veelgestelde vragen

<details>

<summary>Kan ik met Chloros beelden verwerken van camera&#x27;s die niet van het merk MAPIR zijn?</summary>

Nee, Chloros ondersteunt alleen de verwerking van beelden van MAPIR-camera&#x27;s — de Survey3- en LATTICE-series. Raadpleeg de lijst met [ondersteunde cameramodellen](supported-cameras.md) voor meer informatie. We bieden wel verwerking van andere camera&#x27;s aan op MAPIR Cloud; zie de volledige lijst [hier](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Ondersteunt Chloros LATTICE-camera&#x27;s?</summary>

Ja. Chloros 1.2.0 ondersteunt LATTICE M3C- en M3M-cameramodules van begin tot eind: **live bediening**— detecteren, verbinden, voorbeelden bekijken en opnames maken via het tabblad ‘Camera’s’ in de gebruikersinterface, `chloros-cli lattice` of de Python SDK, inclusief gesynchroniseerde multi-camera-opstellingen met PTP-tijdsynchronisatie — en**volledige radiometrische verwerking** van opnames (raw → debayered → straling → reflectantie → index). Zie [Ondersteunde camera’s](supported-cameras.md) en de [LATTICE-handleiding](lattice/README.md).

</details>

<details>

<summary>Kan ik mijn beelden kalibreren voor reflectantie zonder een kalibratiedoel?</summary>

**Survey3:** Nee. Zonder een opname van het kalibratiedoel die rond dezelfde tijd is gemaakt als de opnames zonder doel, kun je de pixelwaarden van de opname niet koppelen aan een bekend reflectantiepercentage. Als u ook het logboek van een MAPIR-lichtsensor niet meeneemt, wordt het omgevingslichtspectrum niet gemeten en zullen de reflectieresultaten niet nauwkeurig zijn.**LATTICE:** Ja. De reflectie kan worden gerelateerd aan de neerwaartse stralingsintensiteit gemeten door een DAQ-lichtsensor in plaats van een paneel (ρ = π·L/E). Wanneer er *wel* een QA-goedgekeurd doel in het beeld aanwezig is, wordt dit standaard de absolute referentie (`--reflectance-source auto`). Eén uitzondering: „F988-reflectie wordt gekalibreerd met behulp van een reflectiepanel in de scène: de band ligt buiten het gekalibreerde bereik van de DAQ-lichtsensor, dus Chloros past uw meest recente panelopname toe en houdt deze vast tussen panelwaarnemingen.” Zie [Kalibratiedoelen](calibration-targets.md).

</details>

<details>

<summary>Heb ik een DAQ-lichtsensor nodig?</summary>

Niet voor stralingsintensiteit: LATTICE-stralingsintensiteitsexporten zijn afkomstig van de radiometrische fabriekskalibratie van elke camera en vereisen noch een DAQ-sensor, noch een doel. Voor **reflectantie**heb je een referentie voor het omgevingslicht nodig — ofwel de neerwaartse meting van een DAQ-lichtsensor, ofwel een kalibratiedoel in beeld. Met een DAQ-sensor kunt u gekalibreerde reflectie waarden genereren**zonder dat u panelen in de opname hoeft te plaatsen**. Opgenomen `.daq`-bestanden worden automatisch aan uw beelden gekoppeld op basis van de tijdstempel. Zie [Kalibratiedoelen](calibration-targets.md) en de [CLI-referentie](reference/cli-reference.md).

</details>

<details>

<summary>Kan ik Chloros gebruiken met een AI-assistent (Claude, ChatGPT, enz.)?</summary>

Ja — deze handleiding en de CLI/SDK zijn daar speciaal voor ontworpen:

* De volledige index van de handleiding wordt aangeboden op `https://mapir.gitbook.io/chloros/llms.txt`, zodat AI-assistenten elke pagina kunnen vinden.
* De onbewerkte Markdown-code van elke pagina is beschikbaar op de bijbehorende pagina met kleine letters (URL), waaraan `.md` is toegevoegd (bijvoorbeeld `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* De [CLI-referentie](reference/cli-reference.md) en [SDK-referentie](reference/sdk-reference.md) zijn geschreven voor gebruik door LLM&#x27;s: exacte vlaggen, standaardinstellingen, afsluitingssemantiek en commando&#x27;s die je kunt kopiëren en plakken.

Zie [AI-assistenten](ai-assistants.md) voor informatie over hoe je je assistent kunt laten verwijzen naar Chloros.

</details>

<details>

<summary>Waar worden mijn verwerkte uitvoerbestanden opgeslagen?</summary>

De resultaten worden opgeslagen in de projectmap, gegroepeerd per camera en vervolgens per bestandsformaat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **cameramap** — `LATT-<sensor>-<lens>-F<filter>` voor LATTICE, `<model>_<filter>` (bijv. `Survey3N_RGN`) voor Survey3
* **formaatmap** — `tiff16`, `tiff8`, `png8`, `jpg8` of `tiff32`
* **productmappen** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (altijd onder `tiff32`), `<INDEX>_Index_Images/`**Geëxporteerde bestanden behouden de naam van het bronbestand — de map identificeert het product, niet een bestandsextensie.**Met CLI wordt de projectmap naast de invoermap aangemaakt, tenzij u `-o` opgeeft. Let op: een `chloros-cli process`-run waarbij producten zijn aangevraagd maar geen enkel product is geschreven, geeft `Processing finished but wrote no image products.` weer en**stopt met een waarde anders dan nul**, zodat scripts dit kunnen detecteren. Zie [Uitvoerbeeldformaten](output-image-formats.md) en de [CLI-referentie](reference/cli-reference.md).

</details>

<details>

<summary>Kan ik mijn afbeeldingen bewerken voordat ik ze verwerk in Chloros?</summary>

Nee. Chloros gaat ervan uit dat de invoergegevens niet zijn gewijzigd. Wijzig de bestandsnamen niet.

</details>

<details>

<summary>Kan ik mijn MAPIR en Survey3-camera’s instellen op automatische belichting en de afbeeldingen verwerken in Chloros?</summary>

Nee. Beelddatasets van Survey3 moeten een vaste/vergrendelde belichting hebben, dus geen automatische sluitertijd of automatische ISO. Alle beelden van hetzelfde cameramodel moeten een identieke sluitertijd en ISO (belichting) hebben.

LATTICE-camera’s hebben deze beperking niet: Chloros regelt de belichting in realtime (Smart AE), en bij elke opname worden de daadwerkelijk gebruikte belichting en versterking vastgelegd, waarmee de radiometrische verwerkingspijplijn rekening houdt.

</details>

<details>

<summary>Kan de Chloros orthomosaïekbeelden verwerken of analyseren?</summary>

Nee. Alleen afzonderlijke MAPIR-camerabeelden worden ondersteund, geen samengevoegde beelden zoals een orthomozaïekkaart.

</details>

<details>

<summary>Hoe kan ik de stap voor het detecteren van doelen in Chloros versnellen?</summary>

Door in de tabel van de bestandsbrowser de doelbeelden in de rechterkolom vooraf te selecteren, geeft u Chloros de opdracht om alleen in die beelden naar kalibratiedoelen te zoeken, waardoor de verwerking aanzienlijk wordt versneld.

</details>

<details>

<summary>Als ik mijn beelden wil uploaden naar <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">de MAPIR Cloud,</a> moet ik ze dan eerst verwerken in Chloros voordat ik ze upload?</summary>

Als u van plan bent om naar ons online verwerkingsplatform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) te uploaden, bewerk de afbeeldingen dan niet vóór het uploaden. Cloud voert dezelfde verwerking uit en nog veel meer.

</details>

<details>

<summary>Zal MAPIR ooit functie X ondersteunen? Ik zou heel graag zien dat MAPIR X zou aanbieden.</summary>

We zijn altijd geïnteresseerd in feedback over onze producten. Als je een probleem met onze producten tegenkomt, of een suggestie hebt over hoe we onze producten kunnen verbeteren, neem dan [CONTACT MET ONS OP](https://www.mapir.camera/community/contact) om je mening te delen. Het grootste deel van ons onderzoek en ontwikkeling (R&amp;D) is gebaseerd op het luisteren naar de belangrijkste behoeften van onze klanten.

</details>

<details>

<summary>Is Chloros beschikbaar voor Linux?</summary>

Ja! Chloros 1.2.0 ondersteunt Linux amd64 (x86_64) en arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakketten. De CLI en Python SDK worden volledig ondersteund op Linux, inclusief livebesturing van LATTICE-camera’s en DAQ-sensoren. Er is geen GUI voor Linux — alle interactie verloopt via de [CLI](CLI.md) of [Python SDK](api-python-sdk.md). Zie [Linux Overzicht](linux/linux-overview.md) voor meer informatie.

</details>

<details>

<summary>Kan ik Chloros op NVIDIA Jetson gebruiken?</summary>

Ja! Chloros ondersteunt NVIDIA Jetson-platforms, waaronder Jetson Nano, Orin Nano, Orin NX en AGX Orin met JetPack 6. Chloros detecteert automatisch je Jetson-model en optimaliseert de verwerkingsstrategie. Raadpleeg de [NVIDIA Jetson-handleiding](linux/nvidia-jetson-guide.md) voor instructies over installatie en implementatie.

</details>

<details>

<summary>Wordt Chloros automatisch geoptimaliseerd voor mijn hardware?</summary>

Ja! Chloros bevat [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) dat automatisch je CPU, GPU, RAM en (op Jetson) thermische sensoren detecteert. Vervolgens kiest het de optimale verwerkingsstrategie — van `GPU_PARALLEL` op systemen met veel geheugen tot `GPU_SINGLE` op apparaten met beperkte middelen tot `CPU_PARALLEL` op systemen zonder NVIDIA GPU. Er is geen handmatige configuratie nodig.

</details>

<details>

<summary>Wat is de 4-thread verwerkingspijplijn?</summary>

Chloros maakt gebruik van een 4-thread-pijplijnarchitectuur voor gebruikers van Chloros+: Thread 1 (Detectie) laadt afbeeldingen en detecteert kalibratiedoelen, Thread 2 (Kalibratie) berekent de reflectiekalibratie, Thread 3 (Verwerking) voert GPU-versnelde debayering en indexberekening uit, en Thread 4 (Exporteren) schrijft uitvoerbestanden. Meerdere afbeeldingen kunnen tegelijkertijd in verschillende threads worden verwerkt voor een maximale doorvoersnelheid. Zie [Verwerkingspijplijn](processing-architecture/processing-pipeline.md) voor meer informatie.

</details>

<details>

<summary>Hoe voer ik diagnostische tests uit op mijn Chloros-installatie?</summary>

Gebruik het commando `selftest` om een 7-stappen-smoketest uit te voeren: versie, beschikbaarheid van poorten, opstarten van de backend, API-connectiviteit (`/api/test`), systeeminformatie (`/api/system-info` — GPU/CUDA/PyTorch), aanwezigheid van het denoiser-model en gereedheid van CUDA + denoiser:

```bash
chloros-cli selftest
```

Dit is met name handig op Linux/Jetson-systemen om de GPU- en CUDA-configuratie te controleren.

</details>
