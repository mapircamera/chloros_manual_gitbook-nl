# Verwerkingspijplijn

Chloros 1.1.0 maakt gebruik van een verwerkingspijplijn met vier threads die functioneert als een gefaseerde lopende band. Elke thread behandelt een afzonderlijke fase van de verwerkingsworkflow, waardoor meerdere afbeeldingen gelijktijdig in verschillende fasen kunnen worden verwerkt.

***

## Pijplijnarchitectuur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Elke afbeelding doorloopt alle vier de threads in volgorde. Met de multithread-verwerking van Chloros+ kunnen meerdere afbeeldingen tegelijkertijd in verschillende threads aanwezig zijn — terwijl Thread 3 één afbeelding verwerkt, kan Thread 1 de volgende detecteren, kan Thread 2 een andere kalibreren en kan Thread 4 een eerder verwerkte afbeelding naar de schijf schrijven.

***

## Details van de threads

### Thread 1: Detectie

**Doel**: Afbeeldingen laden en kalibratiedoelen detecteren.

* Leest afbeeldingsbestanden van de schijf (RAW, JPG)
* Haalt EXIF-metadata op (GPS, cameramodel, tijdstempels, belichting)
* Detecteert ArUco-kalibratiedoelen in gemarkeerde doelafbeeldingen
* Uitvoer: beeldgegevens + metadata + resultaten van doeldetectie

Dit is voornamelijk een I/O- en CPU-gebonden thread.

### Thread 2: Kalibratie

**Doel**: Berekent kalibratieparameters op basis van gedetecteerde doelen.

* Berekent reflectantie-kalibratiecoëfficiënten op basis van doelafbeeldingen
* Berekent vignetteringscorrectieparameters
* Bepaalt kalibratiecurves per band
* Uitvoer: kalibratieparameters voor elke afbeelding

Dit is een CPU-gebonden rekenthread.

### Thread 3: Verwerking (GPU)

**Doel**: Correcties toepassen en vegetatie-indexen berekenen.**Dit is de meest rekenintensieve thread.*** **Debayering**: Converteert RAW-gegevens met Bayer-patroon naar meerkanaalsbeelden
  * Standaard (Snel, gemiddelde kwaliteit) — standaard
  * Textuurbewust (Langzaam, hoogste kwaliteit) — alleen Chloros+, maakt gebruik van AI/ML-ruisonderdrukking
* **Vignetteringscorrectie**: Past lensvignetteringscorrectie toe over het hele beeld
* **Reflectiekalibratie**: past kalibratiecoëfficiënten toe om te converteren naar reflectiewaarden
* **Indexberekening**: berekent vegetatie-indexen (NDVI, NDRE, GNDVI, enz.)
* Uitvoer: verwerkte beeldgegevens klaar voor export

Deze thread profiteert het meest van GPU-versnelling. Het [Dynamic Compute Adaptation](dynamic-compute-adaptation.md)-systeem optimaliseert voornamelijk het gedrag van deze thread.

### Thread 4: Exporteren

**Doel**: Verwerkte afbeeldingen naar schijf schrijven.

* Schrijft uitvoerbestanden in het geselecteerde formaat (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Sluit EXIF-metadata in uitvoerbestanden in (GPS, tijdstempels, verwerkingsparameters)
* Organiseert de uitvoer in submappen per cameramodel
* Uitvoer: definitieve bestanden op schijf

Dit is voornamelijk een I/O-gebonden thread. SSD-opslag verbetert de prestaties van Thread 4 aanzienlijk.

***

## Sequentiële versus pipelined verwerking

### Gratis modus (Sequentiële)

In de gratis versie van Chloros worden afbeeldingen **één voor één** verwerkt, sequentieel door alle vier de fasen:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

De voortgangsbalk in de GUI toont 2 fasen: Doelherkenning en Verwerking.

### Chloros+-modus (Pipelined)

Met een Chloros+-licentie werken alle vier de threads **gelijktijdig** aan verschillende afbeeldingen:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

De voortgangsbalk in de GUI toont 4 fasen: Detecteren, Analyseren, Kalibreren, Exporteren. Beweeg de muis over de voortgangsbalk om de voortgang per thread te zien.

{% hint style="success" %}
**Pipelined-verwerking met Chloros+** kan 3-5 keer sneller zijn dan sequentiële verwerking, afhankelijk van uw hardware en de grootte van de dataset. De snelheidswinst is het grootst op systemen met snelle GPU&#x27;s en SSD&#x27;s.
{% endhint %}

***

## Voortgang export thread 4

In Chloros 1.1.0 heeft de exportthread (thread 4) zijn eigen speciale voortgangsweergave. U kunt de voortgang van de export afzonderlijk volgen:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

De verwerking is voltooid wanneer Thread 4 100% bereikt.

***

## Relatie met dynamische rekenaanpassing

Het [Dynamic Compute Adaptation](dynamic-compute-adaptation.md)-systeem heeft voornamelijk invloed op **Thread 3 (Verwerking)**:

* **`GPU_PARALLEL`**-strategie: Thread 3 verwerkt meerdere afbeeldingen tegelijkertijd via de GPU met behulp van de `fused_gpu`-pijplijn
* **`GPU_SINGLE`**-strategie: Thread 3 verwerkt één afbeelding tegelijk met behulp van de geheugenefficiënte `tiled_gpu`-pijplijn
* **`CPU_PARALLEL`**-strategie: Thread 3 maakt gebruik van CPU-gebaseerde verwerking met multithreaded parallellisme

De GPU-geheugentoewijzing van Thread 3 verandert ook dynamisch naarmate Threads 1 en 2 worden voltooid — zie [Dynamische GPU-geheugentoewijzing](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Volgende stappen

* [Dynamische rekenaanpassing](dynamic-compute-adaptation.md) — Hoe Chloros de optimale strategie voor uw hardware selecteert
* [NVIDIA Jetson-gids](../linux/nvidia-jetson-guide.md) — Platformspecifiek pijplijngedrag op Jetson
* [De verwerking monitoren](../processing-images-gui/monitoring-the-processing.md) — Voortgangsmonitoring via de GUI
