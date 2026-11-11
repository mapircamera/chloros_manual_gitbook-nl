---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Veelgestelde vragen

<details>

<summary>Kan ik met Chloros beelden verwerken van camera&#x27;s die niet van het merk MAPIR zijn?</summary>

Nee, Chloros ondersteunt alleen de verwerking van beelden van MAPIR-camera&#x27;s — de Survey3- en LATTICE-series. Raadpleeg de lijst met [ondersteunde cameramodellen](supported-cameras.md) voor meer informatie. We bieden wel verwerking van beelden van andere camera&#x27;s aan via MAPIR Cloud; zie de volledige lijst [hier](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Ondersteunt Chloros LATTICE-camera’s?</summary>

Ja. Chloros 1.3.0 ondersteunt LATTICE M3C- en M3M-cameramodules van begin tot eind: **live bediening**— detecteren, verbinden, bekijken en opnamen maken via het tabblad ‘Camera’s’ in de GUI, `chloros-cli lattice`, of de Python-SDK, inclusief gesynchroniseerde opstellingen met meerdere camera’s met PTP-tijdsynchronisatie — en**volledige radiometrische verwerking** van opnamen (raw → debayered → radiance → reflectance → index). Zie [Ondersteunde camera’s](supported-cameras.md) en de [LATTICE-handleiding](lattice/README.md).

</details>

<details>

<summary>Kan ik mijn beelden kalibreren voor reflectantie zonder een kalibratiedoel?</summary>

**Survey3

:** Nee. Zonder een opname van het kalibratiedoel die rond dezelfde tijd is gemaakt als de opnames zonder doel, kun je de pixelwaarden van de opname niet koppelen aan een bekend reflectantiepercentage. Als je ook geen logbestand van een MAPIR

-lichtsensor toevoegt, wordt het omgevingslichtspectrum niet gemeten en zullen de reflectie-resultaten niet nauwkeurig zijn.

**LATTICE:** Ja. De reflectie kan worden gerelateerd aan de neerwaartse stralingsintensiteit die wordt gemeten door een DAQ-lichtsensor in plaats van een paneel (ρ = π·L/E). Wanneer er *wel* een QA-goedgekeurd doel binnen het beeld aanwezig is, wordt dit standaard de absolute referentie (`--reflectance-source auto`), tenzij het met een vlag wordt goedgekeurd, in welk geval het doel en de DAQ-ketens met elkaar concurreren op basis van de gemeten score. Eén uitzondering: „De reflectie van F988 wordt gekalibreerd met behulp van een reflectiepaneel in de scène: de band ligt buiten het gekalibreerde bereik van de DAQ-lichtsensor, dus Chloros

past je meest recente paneelopname toe en behoudt deze tussen paneelwaarnemingen door.” Zie [Kalibratiedoelen](calibration-targets.md).

</details>

<details>

<summary>Heb ik een DAQ-lichtsensor nodig?</summary>

Niet voor stralingsintensiteit: LATTICE-stralingsintensiteitsexporten zijn afkomstig van de radiometrische fabriekskalibratie van elke camera en vereisen noch een DAQ-sensor, noch een doel. Voor **reflectantie**heb je een referentie voor het omgevingslicht nodig — ofwel een neerwaartse meting van een DAQ-lichtsensor, ofwel een kalibratiedoel in het beeld. Met een DAQ-sensor kunt u gekalibreerde reflectie waarden genereren**zonder panelen in de opname te plaatsen**. Opgenomen `.daq`-bestanden worden automatisch aan uw beelden gekoppeld op basis van de tijdstempel. Zie [Kalibratiedoelen](calibration-targets.md) en de [CLI

-referentie](reference/cli-reference.md).

</details>

<details>

<summary>Kan ik Chloros gebruiken met een AI-assistent (Claude, ChatGPT, enz.)?</summary>

Ja — deze handleiding en de CLI/SDK zijn hiervoor ontworpen:

* De volledige index van de handleiding is beschikbaar op `https://mapir.gitbook.io/chloros/llms.txt`, zodat AI-assistenten elke pagina kunnen vinden.
* De onbewerkte Markdown-code van elke pagina is beschikbaar via de URL van de pagina in kleine letters, met `.md` eraan toegevoegd (bijvoorbeeld `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* De [CLI-referentie](reference/cli-reference.md) en [SDK-referentie](reference/sdk-reference.md) zijn geschreven voor gebruik door LLM’s: exacte vlaggen, standaardinstellingen, afsluitingssemantiek en commando’s die je kunt kopiëren en plakken.

Zie [AI-assistenten](ai-assistants.md) voor informatie over hoe je je assistent op Chloros kunt laten werken.

</details>

<details>

<summary>Waar komen mijn verwerkte uitvoerbestanden terecht?</summary>

De outputbestanden worden opgeslagen in de projectmap, gegroepeerd per camera en vervolgens per bestandsformaat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **cameramap** — `LATT-<sensor>-<lens>-F<filter>` voor LATTICE, `<model>_<filter>` (bijv. `Survey3N_RGN`) voor Survey3
* **formaatmap** — `tiff16`, `tiff8`, `png8`, `jpg8` of `tiff32`
* **productmappen** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (altijd onder `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — Alleen LATTICE multispectraal: maskers per pixel naast de productmappen, zie hieronder**Geëxporteerde bestanden behouden de naam van het bronbestand — de map identificeert het product, niet een bestandsextensie.**Met de CLI wordt de projectmap aangemaakt onder je standaardprojectmap (`Documents/MAPIR/Chloros Projects`, tenzij je dit hebt gewijzigd), tenzij je `-o` opgeeft; een relatief `-o` wordt relatief beschouwd ten opzichte van de map van waaruit u de opdracht uitvoert. Merk op dat een `chloros-cli process`-uitvoering waarbij producten zijn opgevraagd maar geen enkel product is geschreven, `Processing finished but wrote no image products.` weergeeft en**met een waarde anders dan nul afsluit**, zodat scripts dit kunnen detecteren. Zie [Uitvoerbeeldformaten](output-image-formats.md) en de [CLI-referentie](reference/cli-reference.md).

</details>

<details>

<summary>Wat zijn de mapjes `clamp_masks` en de *_clampmask.tif-bestanden?</summary>

Nieuw in 1.3.0. De radiance- en reflectance-producten van een multispectrale LATTICE M3C-camera ondergaan een 3×3-band-unmix, en wanneer dit in een band een negatieve waarde oplevert, knipt Chloros die waarde af tot nul. Een product met ten minste één geknipte pixel krijgt een enkelbandig `uint8`-masker dat aangeeft welke pixels en banden zijn geknipt (bit 0 = het R-kanaal van het product, bit 1 = G, bit 2 = B). Maskers worden opgeslagen in een `clamp_masks/`-map **naast** de productmappen — bijvoorbeeld `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — en de `Chloros:UnmixClampMask` XMP-tag van het product noemt het bijbehorende masker (of vermeldt `none`).

Het zijn geen afbeeldingen van de scène: de eigen importfunctie van Chloros slaat ze over, en omdat ze nooit in een productmap staan, kun je een productmap gewoon zo doorgeven aan Pix4D of Metashape. Zie [Clamp-maskers](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Waarom geeft Chloros de melding „De projectmap is te diep voor deze computer“?</summary>

Dit geldt alleen voor Windows. Bij Windows zijn lange paden standaard uitgeschakeld, waardoor geen enkel bestandspad langer mag zijn dan 260 tekens. Chloros-producten bevinden zich tot ongeveer 130 tekens onder de projectmap, dus Chloros weigert een diepere projectmap vóór de verwerking — in plaats van het programma uit te voeren en niets op te slaan, zoals vóór versie 1.3.0 kon gebeuren. Het bericht geeft de langste projectmap aan die nog past.

Los het op een van de volgende manieren op:

* Gebruik een kortere projectmap (de **Werkmap** in Projectinstellingen, of `-o` / `chloros-cli set-project-folder` voor de CLI).
* Schakel lange paden in Windows in en start Chloros opnieuw op. De optie **Enable Windows long paths** van het Chloros-installatieprogramma (standaard aangevinkt) zorgt hiervoor; voer het installatieprogramma opnieuw uit als je deze optie hebt uitgeschakeld.

Zie [De verwerking starten](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Kan ik mijn afbeeldingen bewerken voordat ik ze in Chloros verwerk?</summary>

Nee. Chloros gaat ervan uit dat de invoergegevens niet zijn gewijzigd. Wijzig de bestandsnamen niet.

</details>

<details>

<summary>Kan ik mijn MAPIR Survey3-camera&#x27;s instellen op automatische belichting en de beelden verwerken in Chloros?</summary>

Nee. Survey3-beelddatasets moeten een vaste/vergrendelde belichting hebben, dus geen automatische sluitertijd of automatische ISO. Alle beelden van hetzelfde cameramodel moeten een identieke sluitertijd en ISO (belichting) hebben.

LATTICE-camera’s hebben deze beperking niet: Chloros regelt de belichting in realtime (Smart AE), en bij elke opname worden de daadwerkelijk gebruikte belichting en versterking geregistreerd, waarmee de radiometrische verwerkingspijplijn rekening houdt.

</details>

<details>

<summary>Kan Chlorosorthomosaïekbeelden verwerken of analyseren?</summary>

Nee. Alleen afzonderlijke MAPIR-camerabeelden worden ondersteund, geen samengevoegde beelden zoals een orthomozaïekkaart.

</details>

<details>

<summary>Hoe kan ik de stap van de doelherkenning in Chloros versnellen?</summary>

Door in de tabel van de bestandsbrowser de doelbeelden in de rechterkolom vooraf te selecteren, geeft u Chloros de opdracht om alleen in die beelden naar kalibratiedoelen te zoeken, wat de verwerking aanzienlijk versnelt.

</details>

<details>

<summary>Als ik mijn beelden naar <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> wil uploaden, moet ik ze dan eerst in Chloros verwerken voordat ik ze upload?</summary>

Als u van plan bent om uw beelden te uploaden naar ons online verwerkingsplatform [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), bewerk de beelden dan niet vóór het uploaden. Cloud voert dezelfde verwerking uit en nog veel meer.

</details>

<details>

<summary>Zal MAPIR ooit functie X ondersteunen? Ik zou heel graag zien dat MAPIR X zou aanbieden.</summary>

We zijn altijd geïnteresseerd in feedback over onze producten. Als je een probleem met onze producten tegenkomt, of een suggestie hebt over hoe we onze producten kunnen verbeteren, neem dan [CONTACT MET ONS OP](https://www.mapir.camera/community/contact) om je ideeën te delen. Het grootste deel van ons onderzoek en ontwikkeling is erop gericht om in te spelen op de belangrijkste behoeften van onze klanten.

</details>

<details>

<summary>Is Chloros beschikbaar voor Linux?</summary>

Ja! Chloros 1.3.0 ondersteunt Linux amd64 (x86_64) en arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakketten. De CLI en de Python SDK worden volledig ondersteund op Linux, inclusief live besturing van LATTICE-camera’s en DAQ-sensoren. Er is geen GUI voor Linux — alle interactie verloopt via de [CLI](CLI.md) of de [Python SDK](api-python-sdk.md). Zie het [Overzicht van Linux](linux/linux-overview.md) voor meer informatie.

</details>

<details>

<summary>Kan ik Chloros op NVIDIA Jetson draaien?</summary>

Ja! Chloros ondersteunt de NVIDIA Jetson **Orin-serie** — Orin Nano, Orin NX en AGX Orin — waarop JetPack 6 draait. Oudere modellen (Jetson Nano, TX2, Xavier NX) kunnen JetPack 6 niet uitvoeren en worden niet ondersteund door het huidige pakket. Chloros detecteert automatisch uw Jetson-model en optimaliseert de verwerkingsstrategie. Het Jetson-pakket maakt gebruik van de CUDA-build van PyTorch uit JetPack 6, zodat de verwerking op de GPU van de Jetson plaatsvindt. Raadpleeg de [NVIDIA Jetson-handleiding](linux/nvidia-jetson-guide.md) voor instructies over installatie en implementatie.

</details>

<details>

<summary>Optimaliseert Chloros zich automatisch voor mijn hardware?</summary>

Ja! Chloros bevat [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) dat automatisch je CPU, GPU, RAM en (op Jetson) thermische sensoren detecteert. Vervolgens kiest het de optimale verwerkingsstrategie — van `GPU_PARALLEL` op GPU’s met veel geheugen tot `GPU_SINGLE` op kleinere GPU’s en `CPU_PARALLEL` op systemen zonder een CUDA-GPU die PyTorch kan gebruiken. Er is geen handmatige configuratie nodig.

</details>

<details>

<summary>Wat is de 4-thread verwerkingspijplijn?</summary>

Chloros
maakt gebruik van een 4-thread pipelined architectuur voor Chloros
+-gebruikers: Thread 1 (Detectie) laadt afbeeldingen en detecteert kalibratiedoelen, Thread 2 (Kalibratie) berekent de reflectiekalibratie, Thread 3 (Verwerking) voert GPU-versnelde debayering en indexberekening uit, en Thread 4 (Exporteren) schrijft uitvoerbestanden. Meerdere afbeeldingen kunnen tegelijkertijd in verschillende threads worden verwerkt voor een maximale doorvoer. Zie [Verwerkingspijplijn](processing-architecture/processing-pipeline.md) voor meer informatie.

</details>

<details>

<summary>Hoe voer ik diagnostiek uit op mijn Chloros-installatie?</summary>

Gebruik het commando `selftest` om een 7-stappen-smoketest uit te voeren: versie, beschikbaarheid van poorten, opstarten van backend, API-connectiviteit (`/api/test`), systeeminformatie (`/api/system-info` — GPU/CUDA/PyTorch), aanwezigheid van het denoiser-model en gereedheid van CUDA + denoiser:

```bash
chloros-cli selftest
```

Dit is vooral handig op Linux-systemen om de GPU- en CUDA-configuratie te controleren. Op een Jetson betekent `CUDA: False` meestal dat de CUDA- en cuDNN-bibliotheken van JetPack 6 ontbreken: installeer deze met `sudo apt install nvidia-jetpack`.

</details>
