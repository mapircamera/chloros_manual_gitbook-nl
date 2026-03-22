# Dynamische aanpassing aan de rekenkracht

Chloros 1.1.0 introduceert intelligente hardware-detectie en automatische selectie van verwerkingsstrategieën. De verwerkingsengine past zich aan uw hardware aan — van een Jetson Nano tot een werkstation met meerdere GPU’s — zonder dat er handmatige configuratie nodig is.

***

## Hoe het werkt

Wanneer Chloros start, maakt het automatisch een profiel van uw systeem:

1. **Detecteert het besturingssysteem** — Windows of Linux
2. **Identificeert CPU-kernen en totaal RAM**

3.**Detecteert aanwezigheid van GPU** — NVIDIA CUDA-capaciteit, VRAM, model
4. **Identificeert Jetson-model** (indien van toepassing) — via `/proc/device-tree/model`
5. **Controleert thermische sensoren** (Jetson) — voor temperatuurbewuste verwerking
6. **Selecteert de optimale rekenstrategie** — op basis van alle gedetecteerde hardware
7. **Configureert het aantal workers, het type pipeline en de geheugentoewijzing** automatisch

Het resultaat wordt in de cache opgeslagen, zodat volgende runs sneller starten. Als de hardware verandert (bijv. een GPU wordt toegevoegd), voert Chloros bij de volgende start een nieuwe profilering uit.

***

## Rekenstrategieën

Chloros selecteert een van de drie rekenstrategieën op basis van uw hardware:

| Strategie | GPU vereist | Werkers | Pipeline | Meest geschikt voor |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Ja (12 GB+ VRAM of 16 GB+ gedeeld) | 3-4 | `fused_gpu` | Desktop-GPU&#x27;s met 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Ja (&lt; 12 GB VRAM) | 1-3 | `tiled_gpu` | Instap-GPU&#x27;s, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nee | cores - 1 | `cpu_fallback` | Systemen zonder NVIDIA GPU |

### Pipelinetypen

* **`fused_gpu`** — Volledig GPU-verwerkingspad. Alle debayer-, correctie- en indexbewerkingen worden in één samengevoegde doorloop op de GPU uitgevoerd. Hoogste doorvoer, maar vereist meer VRAM.
* **`tiled_gpu`** — Geheugenefficiënt GPU-pad. Verwerkt afbeeldingen in tegels zodat ze binnen het beperkte GPU-geheugen passen. Lagere doorvoer, maar werkt op apparaten met beperkt geheugen.
* **`cpu_fallback`** — Verwerking uitsluitend op de CPU met behulp van multithreaded parallellisme. Wordt gebruikt wanneer er geen NVIDIA GPU beschikbaar is.***

## Platformspecifiek gedrag

| Platform | Strategie | Werkers | Pipeline | Opmerkingen |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (geserialiseerd) | Geheugenefficiënte modus, verwerkt één afbeelding tegelijk |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (gelijktijdig) | Aanbevolen edge-apparaat — echte parallelle GPU-verwerking |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (gelijktijdig) | Maximale edge-prestaties |
| **Desktop met 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Goede desktopprestaties met geheugenefficiënte tegels |
| **Desktop met 12 GB+ GPU** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optimale desktopprestaties |
| **Systeem met alleen CPU** | `CPU_PARALLEL` | cores - 1 | `cpu_fallback` | Geen GPU vereist, maakt gebruik van ThreadPool |

{% hint style="info" %}
**Jetson unified memory**: Jetson-apparaten delen GPU- en CPU-geheugen. Een Jetson Orin NX 16 GB rapporteert ~15,3 GB VRAM, maar dit is hetzelfde fysieke RAM dat wordt gebruikt door het besturingssysteem en de CPU-processen. Chloros houdt hier rekening mee bij het instellen van drempels voor geheugentoewijzing.
{% endhint %}

***

## Dynamische GPU-geheugentoewijzing

Chloros maakt gebruik van een [verwerkingspijplijn met 4 threads](processing-pipeline.md):

* **Thread 1** (Detectie) — Afbeelding laden, EXIF-parsing, doelwitdetectie
* **Thread 2** (Kalibratie) — Berekening van reflectiekalibratie
* **Thread 3** (Verwerking) — GPU-debayer, vignetteringscorrectie, indexberekening
* **Thread 4** (Exporteren) — Bestanden schrijven, metadata insluiten

Naarmate eerdere pijplijn-threads hun werk voltooien (bijv. alle afbeeldingen zijn gedetecteerd), wordt hun GPU-geheugentoewijzing vrijgegeven en **herverdeeld over de resterende actieve threads**. Dit betekent dat Thread 3 (de GPU-intensieve fase) geleidelijk meer geheugen krijgt naarmate de pijplijn vordert, waardoor de doorvoer voor het meest rekenintensieve werk wordt verbeterd.

### Toewijzingsfasen

| Fase | Actieve threads | GPU-geheugenverdeling |
| --- | --- | --- |
| **Vroeg** | 1, 2, 3, 4 | Verdeeld over alle threads |
| **Midden-vroeg** | 2, 3, 4 | Geheugen van thread 1 herverdeeld |
| **Midden-laat** | 3, 4 | Geheugen van threads 1+2 gaat naar 3+4 |
| **Laat** | 3 of 4 | Maximaal geheugen voor resterende thread |***

## Texture Aware-verwerking

De Texture Aware-debayer-methode (alleen Chloros+) gebruikt aanzienlijk meer GPU-geheugen dan de standaardmethode vanwege het AI/ML-ruisonderdrukkingsmodel:

* Systemen met **&lt; 7 GB VRAM** worden gedwongen tot een synchrone verwerkingslus voor de Texture Aware-modus (één afbeelding per keer)
* Systemen met **7 GB+ VRAM** kunnen Texture Aware gelijktijdig verwerken, hoewel met een lager aantal workers in vergelijking met Standard***

## Thermisch beheer (Jetson)

Jetson-apparaten hebben thermische beperkingen, vooral bij gebruik in gesloten ruimtes of in de lucht. Chloros bewaakt de GPU- en CPU-temperaturen en past de verwerking automatisch aan:

| Temperatuur | Reactie |
| --- | --- |
| **&lt; 70 °C** | Normale werking — volledige snelheid |
| **70 °C** (Waarschuwing) | Batchgrootte verkleinen |
| **80 °C** (Kritiek) | Agressieve beperking — lagere gelijktijdigheid en aantal workers |
| **90 °C** (Uitschakeling) | GPU-verwerking volledig stoppen |

Voor temperatuurbewaking wordt `tegrastats` gebruikt op Jetson-platforms. Op desktopsystemen met voldoende koeling wordt thermische throttling zelden geactiveerd.

***

## Omgaan met geheugendruk

Chloros bewaakt de geheugendruk van het systeem tijdens de verwerking:

* **Geheugendrempel**: 85% bezetting activeert conservatief gedrag
* **OOM-reductie**: Als er een out-of-memory-gebeurtenis optreedt, wordt de toewijzing met 25% verminderd (0,75x vermenigvuldigingsfactor)
* **Pipeline-fallback**: Bij ernstige geheugendruk schakelt de pijplijn automatisch over van `fused_gpu` naar `tiled_gpu`
* **Aanbevelingen voor swapruimte**: Op Jetson waarschuwt Chloros u als de swapruimte onvoldoende is voor de grootte van uw dataset***

## Bewaking van rekenaanpassing

### Statusuitvoer van CLI

Wanneer de verwerking start, geeft CLI het gedetecteerde hardwareprofiel weer:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Systeemdiagnostiek

Voer `chloros-cli selftest` uit om een volledig hardwareprofiel te bekijken en de rekencapaciteiten te verifiëren:

```bash
chloros-cli selftest
```

Hiermee worden de beschikbaarheid van CUDA, het GPU-geheugen, denoiser-modellen en de backend-connectiviteit gecontroleerd.

***

## Volgende stappen

* [Verwerkingspijplijn](processing-pipeline.md) — Inzicht in de 4-thread pijplijnarchitectuur
* [NVIDIA Jetson-handleiding](../linux/nvidia-jetson-guide.md) — Jetson-specifieke implementatie en optimalisatie
* [CLI : Opdrachtregel](../CLI.md) — Volledige CLI-referentie
