# Verwerkingspijplijn

Chloros 1.3.0 maakt gebruik van een verwerkingspijplijn met 4 threads die werkt als een gefaseerde lopende band. Elke thread behandelt een afzonderlijke fase van de workflow, zodat er tegelijkertijd meerdere afbeeldingen in verschillende fasen in behandeling kunnen zijn.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architectuur van de verwerkingspijplijn

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Elke afbeelding doorloopt alle vier de threads in volgorde. Met de multithread-verwerking vChloros+ bezetten meerdere afbeeldingen tegelijkertijd verschillende threads — terwijl thread 3 één afbeelding verwerkt, kan thread 1 de volgende detecteren, thread 2 een andere kalibreren en thread 4 een voltooide afbeelding naar de schijf schrijven.

De voortgang wordt per thread gerapporteerd en gestreamd via Server-Sent Events (de backend publiceert deze op `/api/events`). In de live voortgangsweergave van de CLI worden de vier fasen aangeduid als **Detecteren, Analyseren, Verwerken, Exporteren**.

***

## Details van de threads

### Thread 1: Detectie

**Doel**: Afbeeldingen laden en kalibratiedoelen detecteren.

* Leest afbeeldingsbestanden van de schijf — Survey3-paren `.raw`+`.jpg`, LATTICE `.tif`/`.tiff`-opnames en `.dng`
* Haalt EXIF-metadata op (GPS, cameramodel, tijdstempels, belichting)
* Detecteert kalibratiedoelen: met ArUco gemarkeerde doelgeometrieën voor LATTICE-opnames, en de klassieke paneeldetector voor Survey3-kalibratiedoel-foto’s
* Uitvoer: beeldgegevens + metadata + resultaten van doeldetectie

Voornamelijk een I/O- en CPU-gebonden thread.

### Thread 2: Kalibratie

**Doel**: Kalibratieparameters berekenen op basis van gedetecteerde doelen.

* Berekent reflectiekalibratiecoëfficiënten op basis van doelafbeeldingen
* Berekent vignetteringscorrectieparameters
* Bepaalt kalibratiecurves per band
* Uitvoer: kalibratieparameters voor elke afbeelding

Een CPU-gebonden rekenthread. Bij een Survey3- of gemengde run wacht Thread 3 hierop wanneer reflectiekalibratie is ingeschakeld, zodat de coëfficiënten gereed zijn voordat er beelden worden verwerkt. Bij een run met uitsluitend LATTICE draait Thread 3 tijdens de detectie: elk frame start zodra het doelanker definitief is. Een frame neemt het dichtstbijzijnde doelbeeld in de tijd aan beide kanten, dus het wacht tot elk frame van de camera dat in de tijd dichterbij ligt dan het huidige dichtstbijzijnde doelbeeld, is gescand. De resultaten zijn hetzelfde, alleen eerder beschikbaar.

### Thread 3: Verwerking (GPU)

**Doel**: Correcties toepassen en vegetatie-indexen berekenen.**Dit is de meest rekenintensieve thread.*** **Debayering**: zet RAW-Bayer-gegevens om in meerkanaalsbeelden
  * Standaard (snel, gemiddelde kwaliteit) — standaard, `--debayer standard`
  * Textuurbewust (langzaam, hoogste kwaliteit) — alleen Chloros+, `--debayer texture-aware`, maakt gebruik van een AI/ML-ruisonderdrukkingsmodel
  * LATTICE mono (M3M)-opnames zijn enkelbandig: de demosaïc- en witbalansstappen worden hiervoor overgeslagen (met een logbericht van één regel), terwijl alle M3C/Bayer-beelden in dezelfde run deze stappen wel ondergaan
* **Vignetteringscorrectie**: past lensvignetteringscorrectie toe over het gehele beeld
* **Reflectanciekalibratie**: past kalibratiecoëfficiënten toe om om te zetten naar reflectantiewaarden
* **Indexberekening**: berekent vegetatie-indexen (NDVI, NDRE, GNDVI, …)
* Uitvoer: verwerkte beeldgegevens, klaar voor export

Deze thread profiteert het meest van GPU-versnelling, en het is de thread die door [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) wordt afgestemd.

### Thread 4: Exporteren

**Doel**: Verwerkte beelden naar schijf schrijven.

* Schrijft uitvoerbestanden in het geselecteerde formaat — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Sluit metagegevens in de uitvoerbestanden in (GPS, tijdstempels, verwerkingsparameters) — inclusief de `Chloros:*`-verwerkingstags, die vanaf versie 1.3.0 in elke verwerkingsmodus worden geschreven, zowel voor Survey3 als voor LATTICE
* Organiseert de uitvoer in de projectmap als `<camera>/<format>/<Product>_Images/` — bijvoorbeeld `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Geëxporteerde bestanden behouden de naam van het bronbestand; de map identificeert het product.**
* Bij LATTICE-opnames kan één bronframe worden opgesplitst in verschillende producten (Debayered, Preview, Radiance, Reflectance, Index), elk in een eigen productmap. Indexproducten worden berekend op basis van het reflectantieproduct dat de run heeft gegenereerd (als dat er niet is, op basis van een ander product dat de run heeft gegenereerd, zoals radiance). Radiance- en reflectance-producten van multispectrale M3C-camera’s kunnen ook een per-pixel clamp-masker krijgen in `<format>/clamp_masks/` (zie [Uitvoerbeeldformaten](../output-image-formats.md#clamp-masks))
* Uitvoer: definitieve bestanden op schijf

Voornamelijk een I/O-gebonden thread — SSD-opslag levert een merkbare verbetering op.

***

## Achter de schermen: uitvoerders

Binnen Thread 3 wordt het werk per afbeelding als volgt geparalleliseerd:

* **GPU-strategieën**(`GPU_SINGLE`, `GPU_PARALLEL`) maken gebruik van een pool van**worker-processen**, elk een afzonderlijk OS-proces met een eigen CUDA-context. Chloros start elke worker door het eigen geïnstalleerde backend-programma opnieuw te starten — niet via Python’s `multiprocessing`, waarvan het opstarten van workers niet werkt binnen de verpakte applicatie
* **`CPU_PARALLEL`** maakt gebruik van een threadpool (de `ThreadPoolExecutor` van Python) — NumPy en OpenCV geven de GIL vrij, dus threads zijn voldoende
* Jetson-apparaten met 8 GB of minder gedeeld RAM slaan de executor volledig over en verwerken in-process, sequentieel
* Texture Aware op een GPU met minder dan 7 GB VRAM draait ook sequentieel — het denoiser-model past niet meer dan één keer

Chloros maakt geen gebruik van gedistribueerde frameworks van derden (zoals Ray). Zie [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) voor informatie over hoe de strategie en het aantal workers worden gekozen.

***

## Sequentiële versus pipelined verwerking

### Free Mode (sequentieel)

In de gratis versie van Chloros worden beelden **één voor één**, sequentieel door alle vier de fasen verwerkt:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

De GUI toont een vereenvoudigde voortgangsbalk in de gratis modus; de sequentiële fasen worden weergegeven als **Target Detection**en vervolgens**Processing**.

### Chloros+-modus (pijplijnverwerking)

Met een Chloros+-licentie werken alle vier de threads **gelijktijdig** aan verschillende afbeeldingen:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

De voortgangsbalk in de GUI toont de vier fasen; beweeg de muis erover om de voortgang per thread te zien. In de CLI worden dezelfde vier fasen live weergegeven als **Detecting, Analyzing, Processing, Exporting**.

{% hint style="info" %}
**Eén label, twee namen.** De CLI noemt fase 3 _Processing_. De voortgangsfeed van de backend in premium-modus — die welke de voortgangsbalk in de GUI weergeeft — noemt dezelfde fase _Calibrating_. Het gaat om dezelfde thread die hetzelfde werk uitvoert (Thread 3: debayer, correcties, indices).
{% endhint %}

{% hint style="success" %}
**Pipeline-verwerking met Chloros+** kan 3 tot 5 keer sneller zijn dan sequentiële verwerking, afhankelijk van je hardware en de grootte van de dataset. De snelheidswinst is het grootst op systemen met snelle GPU’s en SSD’s.
{% endhint %}

***

## Voortgang van thread 4 (exporteren)

De exportthread heeft zijn eigen voortgangsregistratie, die je afzonderlijk kunt opvragen:**CLI :**

```bash
chloros-cli export-status
```

**SDK :**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

De verwerking is voltooid wanneer Thread 4 100% bereikt.

{% hint style="info" %}
**Een run waarbij geen afbeeldingen worden geschreven, is mislukt.**Bij succes rapporteert `chloros-cli process` hoeveel afbeeldingsproducten er zijn geschreven (`Image products written: N`). Als er producten zijn aangevraagd en er**geen**zijn geschreven — alleen `project.json` en `calibration_data.json` — geeft de CLI `Processing finished but wrote no image products.` weer en**sluit af met een waarde anders dan nul**, waarbij de projectmap en de gebruikelijke oorzaken worden vermeld (de invoermap werd niet herkend als een opname — controleer de lay-out en `--input-level` — of elk aangevraagd product was niet van toepassing op die camera’s). Scripts kunnen vertrouwen op de afsluitcode.
{% endhint %}

***

## Relatie tot Dynamic Compute Adaptation

[Dynamic Compute Adaptation](dynamic-compute-adaptation.md) heeft voornamelijk invloed op **Thread 3 (Verwerking)**:

* **`GPU_PARALLEL`**: Thread 3 verwerkt meerdere afbeeldingen tegelijkertijd via de GPU met behulp van de `fused_gpu`-pijplijn
* **`GPU_SINGLE`**: Thread 3 serialiseert de toegang tot de GPU met een semafoor terwijl werkprocessen I/O overlappen, met behulp van `fused_gpu` of de geheugenefficiënte `tiled_gpu`-pijplijn
* **`CPU_PARALLEL`**: Thread 3 maakt gebruik van CPU-gebaseerde verwerking met multithread-parallellisme

De GPU-geheugentoewijzing van thread 3 neemt ook toe naarmate threads 1 en 2 worden voltooid — zie [Dynamische GPU-geheugentoewijzing](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Volgende stappen

* [Dynamische rekenaanpassing](dynamic-compute-adaptation.md) — Hoe Chloros de optimale strategie voor uw hardware selecteert
* [NVIDIA Jetson-handleiding](../linux/nvidia-jetson-guide.md) — Platformspecifiek pijplijngedrag op Jetson
* [De verwerking monitoren](../processing-images-gui/monitoring-the-processing.md) — Voortgangsmonitoring via de GUI
* [Referentie CLI](../reference/cli-reference.md) — `process`, `export-status`, exitcodes en uitvoerindeling
