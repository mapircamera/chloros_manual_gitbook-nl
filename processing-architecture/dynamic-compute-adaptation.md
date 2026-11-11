# Dynamische aanpassing van rekenkracht

Chloros 1.3.0 maakt gebruik van hardwaredetectie en automatische selectie van verwerkingsstrategieën. De verwerkingsengine past zich aan uw hardware aan — van een Jetson Orin Nano tot een werkstation met meerdere GPU’s — zonder dat er handmatige configuratie nodig is.

***

## Hoe het werkt

Wanneer Chloros opstart, maakt het een profiel van uw systeem:

1. **Detecteert het besturingssysteem** — Windows of Linux
2. **Identificeert CPU-kernen en totaal RAM-geheugen**

3.**Detecteert de aanwezigheid van een GPU** — NVIDIA CUDA-ondersteuning, VRAM, model
4. **Identificeert het Jetson-model** (indien van toepassing) — via `/proc/device-tree/model`
5. **Controleert thermische sensoren** (Jetson) — voor temperatuurbewuste verwerking
6. **Selecteert de rekenstrategie** — op basis van alle gedetecteerde hardware
7. **Configureert het aantal workers, het pijplijntype en de geheugentoewijzing** automatisch

Het gedetecteerde profiel wordt voor de sessie in het geheugen en op schijf in de cache opgeslagen, zodat latere uitvoeringen sneller starten:

| Platform | Profiel in cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (houdt rekening met `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Verwijder dat bestand om een nieuwe detectie te forceren — handig na het toevoegen van een GPU of meer RAM. Chloros voert ook automatisch een nieuwe detectie uit wanneer de cache is geschreven door een incompatibele oudere versie.

***

## Rekenstrategieën

Chloros selecteert een van de drie rekenstrategieën op basis van uw hardware:

| Strategie | Geselecteerd wanneer | Werkers | Uitvoerder | Pijplijn |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-GPU met**12 GB+ VRAM**(op Jetson unified memory, vereist ook 12 GB+ totaal gedeeld RAM) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimaal 2 —**beperkt tot 2 op Jetson** | Werkprocessen (afzonderlijke CUDA-contexten) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA-GPU met**2-12 GB VRAM**| 3 (I/O-overlap; GPU-toegang geserialiseerd door een semafoor).**1 (sequentieel) op Jetsons met minder dan 12 GB RAM** | Werkprocessen (afzonderlijke CUDA-contexten); sequentieel binnen het proces op Jetsons met weinig RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Geen CUDA-GPU, of minder dan 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Praktijkvoorbeelden van de `GPU_PARALLEL`-formule voor het aantal workers: 12 GB VRAM → 2 workers, 16 GB → 3 workers, 20 GB+ → 4 workers. Elke worker krijgt 3,5 GB toegewezen, de gemeten behoefte van een Texture Aware-taak, binnen 90% van het resterende VRAM na een reserve van 0,5 GB.

{% hint style="info" %}
**De GPU-strategieën vereisen een PyTorch-versie die gebruik kan maken van CUDA.** Een machine waarop PyTorch geen bruikbaar CUDA-apparaat heeft, krijgt `CPU_PARALLEL`, ongeacht de GPU. Het Jetson-pakket 1.3.0 maakt gebruik van de CUDA-build van PyTorch uit JetPack 6, dus een Jetson komt in aanmerking zodra de CUDA- en cuDNN-bibliotheken van JetPack zijn geïnstalleerd. Een Jetson met minder dan 12 GB RAM verwerkt altijd één afbeelding tegelijk, in het proces (geregistreerd als `GPU_SINGLE`).
{% endhint %}

GPU-strategieën draaien een pool van werkprocessen, elk een afzonderlijk OS-proces met een eigen CUDA-context, die Chloros start door zijn eigen geïnstalleerde backend-programma opnieuw te starten (niet via Python’s `multiprocessing`, waarvan het opstarten van werkprocessen niet werkt binnen de verpakte applicatie). De CPU-strategie maakt gebruik van een threadpool (Python’s `ThreadPoolExecutor`). Chloros maakt geen gebruik van gedistribueerde frameworks van derden (zoals Ray).

### Soorten pijplijnen

* **`fused_gpu`** — Volledig GPU-verwerkingspad. Debayer-, correctie- en indexbewerkingen worden in één samengevoegde doorloop op de GPU uitgevoerd. Hoogste doorvoersnelheid, vereist de meeste VRAM.
* **`tiled_gpu`** — Geheugenefficiënt GPU-pad. Verwerkt afbeeldingen in tegels zodat ze binnen het beperkte GPU-geheugen passen. Lagere doorvoer, maar werkt op apparaten met beperkt geheugen.
* **`cpu_fallback`** — Verwerking uitsluitend op de CPU met behulp van multithreaded parallellisme. Wordt gebruikt wanneer er geen NVIDIA-GPU beschikbaar is, en als laatste redmiddel wanneer beide GPU-verwerkingspaden falen.

De fallback-keten tijdens runtime is altijd `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Handmatige overschrijving van de strategie

Stel de omgevingsvariabele `CHLOROS_STRATEGY` in om een specifieke strategie af te dwingen — een noodoplossing voor experts voor het geval de automatische detectie iets kiest dat niet geschikt is voor uw situatie (bijvoorbeeld om de GPU vrij te houden voor ander werk):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Bij het vergelijken van de variabele wordt geen onderscheid gemaakt tussen hoofdletters en kleine letters; alles wat niet een van de drie namen is, wordt genegeerd en de automatische detectie verloopt normaal. Bij een overschrijving kiest Chloros nog steeds het aantal workers voor je:

| Overschrijving | Gebruikt aantal workers |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Een overschrijving van `GPU_SINGLE` of `GPU_PARALLEL` op een machine zonder bruikbaar CUDA-apparaat wordt geweigerd met een `[STRATEGY]`-regel in het logboek, en de run maakt gebruik van `CPU_PARALLEL`.

Geef er de voorkeur aan om dit per opdracht in te stellen in plaats van permanent, zodat normale runs zich automatisch blijven aanpassen.

***

## Platformspecifiek gedrag

| Platform | Strategie | Werkers | Pijplijn | Opmerkingen |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequentieel) | Geheugenefficiënte modus, één afbeelding per keer |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequentieel) | Gedeeld RAM van minder dan 12 GB dwingt tot sequentiële verwerking |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (gelijktijdig) | Aanbevolen edge-apparaat — Jetson-beperking tot 2 workers |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (gelijktijdig) | Maximale edge-prestaties (ook beperkt tot 2 workers door Jetson) |
| **Desktop met 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 workers overlappen I/O terwijl een semafoor de GPU-toegang serialiseert |
| **Desktop met 12 GB+ GPU** | `GPU_PARALLEL` | 2-4 | `fused_gpu` (gelijktijdig) | Optimale desktopprestaties: 12 GB → 2 workers, 16 GB → 3, 20 GB+ → 4 |
| **Systeem met alleen CPU** | `CPU_PARALLEL` | fysieke kernen − 1 (min. 2) | `cpu_fallback` | Geen GPU vereist, maakt gebruik van een threadpool |

{% hint style="info" %}
**Jetson unified memory**: Jetson-apparaten delen GPU- en CPU-geheugen. Een Jetson Orin NX met 16 GB rapporteert ~15,3 GB VRAM, maar dat is hetzelfde fysieke RAM dat wordt gebruikt door het besturingssysteem en de CPU-processen. Daarom komen Jetsons met 16 GB of meer in aanmerking voor `GPU_PARALLEL`, net als een desktop-GPU van 12 GB of meer, maar zijn ze beperkt tot 2 workers — de GPU, de worker-processen en hun CUDA-contexten per worker putten allemaal uit dezelfde gedeelde pool.
{% endhint %}

### GPU-budget op basis van VRAM (discrete GPU’s)

Op x86_64-hosts met een discrete NVIDIA-GPU bepaalt het gedetecteerde VRAM ook hoeveel van de verwerkingscapaciteit de kaart mag claimen en hoe groot batches mogen worden:

| Gedetecteerd VRAM | Maximum GPU-budget | Vermenigvuldigingsfactor batchgrootte |
| --- | --- | --- |
| **8 GB+** | 90% | ×2,0 |
| **6-8 GB** | 85% | ×1,75 |
| **3,5–6 GB** | 80% | ×1,5 |
| **2–3,5 GB** | 75% | ×1,25 |
| **Minder dan 2 GB** | 70% | ×1,0 |

Discrete GPU’s reserveren slechts 0,5 GB voor het systeem, omdat ze geen systeem-RAM delen. Jetson-profielen reserveren veel meer en hanteren een lagere limiet — zie de [NVIDIA Jetson-gids](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamische toewijzing van GPU-geheugen

Chloros maakt gebruik van een [verwerkingspijplijn met 4 threads](processing-pipeline.md):

* **Thread 1** (Detectie) — Afbeelding laden, EXIF-gegevens parseren, doelwitdetectie
* **Thread 2** (Kalibratie) — Berekening van de reflectiekalibratie
* **Thread 3** (Verwerking) — GPU-debayer, vignetteringscorrectie, indexberekening
* **Thread 4** (Exporteren) — Bestanden schrijven, metadata insluiten

Threads 1, 2 en 4 leggen weinig druk op de GPU; thread 3 is de meest veeleisende. Naarmate eerdere pipeline-threads worden voltooid, wordt hun GPU-budget **herverdeeld over de resterende actieve threads**, waardoor thread 3 in de loop van de verwerking steeds meer geheugen krijgt.

### Toewijzingsfasen

| Fase | Actieve threads | Verdeling GPU-geheugen |
| --- | --- | --- |
| **Vroeg** | 1, 2, 3, 4 | Verdeeld over alle threads, het grootste deel naar Thread 3 |
| **Midden-vroeg** | 2, 3, 4 | Het aandeel van Thread 1 wordt herverdeeld |
| **Midden-laat** | 3, 4 | De aandelen van threads 1 en 2 gaan naar 3 en 4 |
| **Laat** | 3 of 4 | De laatste actieve thread krijgt zijn maximale toewijzing |

Twee regels bepalen de getallen:

* Een thread die de **enige** actieve is, krijgt de maximale toewijzing van zijn profiel.
* Wanneer er meer dan één *zware* GPU-taak actief is, wordt de basistoewijzing van elke zware taak over deze taken verdeeld (nooit onder het geconfigureerde minimum).

De waarde die daadwerkelijk tijdens de uitvoering wordt gebruikt, is de **laagste** van de toewijzing in het platformprofiel en de live-aanbeveling van de GPU-geheugenmonitor, dus een drukbezette kaart krijgt altijd voorrang boven een optimistisch profiel.***

## Texture Aware-verwerking

De Texture Aware-debayer (**alleen Chloros+** — `--debayer texture-aware`) voert een AI/ML-ruisonderdrukkingsmodel uit dat ongeveer 1,75 GB VRAM in FP16 per kopie nodig heeft, dus gebruikt het veel meer GPU-geheugen dan de standaardmethode:

* Systemen met **minder dan 7 GB VRAM**verwerken Textuurbewust in een**synchrone lus, één afbeelding tegelijk** — er is geen ruimte voor meerdere modelkopieën en een werkpool zou alleen maar voor conflicten zorgen
* Systemen met **7 GB of meer VRAM** kunnen Texture Aware gelijktijdig verwerken, zij het met een lager aantal workers in vergelijking met de standaardmethode
* Op **Jetson** wordt Texture Aware altijd aan één enkele worker gekoppeld
* Zonder **bruikbare CUDA-GPU** maakt de run gebruik van de standaard debayer, en logt de backend één regel met de melding `[DEBAYER] … needs a CUDA GPU …` waarin wordt uitgelegd waarom
* Op Jetsons met laag stroomverbruik (Nano, Orin Nano) past de CLI een GPU-frequentiebeperking toe wanneer je Texture Aware aanvraagt — zie de [NVIDIA Jetson-handleiding](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Warmtebeheer (Jetson)

Jetson-apparaten hebben thermische beperkingen, vooral bij gebruik in gesloten ruimtes of in vliegtuigen. Chloros houdt de ingebouwde temperatuursensoren van de Jetson in de gaten en schaalt de batchgroottes automatisch:

| Temperatuur | Reactie |
| --- | --- |
| **< 70 °C** | Normale werking — volle snelheid |
| **70 °C** (Waarschuwing) | Batchgrootte wordt geleidelijk verkleind (100% → 50% tussen 70 °C en 80 °C) |
| **80 °C** (Kritiek) | Agressieve beperking (50% → 0% tussen 80 °C en 90 °C) |
| **90 °C** (Uitschakeling) | GPU-verwerking volledig stoppen |

Op desktopsystemen met voldoende koeling wordt thermische beperking zelden geactiveerd.

***

## Omgaan met geheugendruk

 Chloros houdt het GPU-geheugen tijdens de verwerking continu in de gaten en reageert op drie niveaus.

**Batchgrootte.** Een batch begint bij 8 afbeeldingen vermenigvuldigd met de platformvermenigvuldigingsfactor uit de bovenstaande tabellen. Chloros controleert vervolgens het vrije VRAM, reserveert 20% daarvan voor de eigen overhead van PyTorch en gaat uit van ongeveer 100 MB GPU-geheugen per afbeelding van 12 MP — de batchgrootte is de kleinste van de twee: de limiet afgeleid van het geheugen of de platformbasis. Deze daalt nooit onder 1.

**Pre-emptieve vermindering.**Bij een**VRAM-bezettingsgraad van meer dan 85%** worden de batchgroottes verkleind voordat er iets misgaat.**Beperking van de toewijzing per thread.** Naarmate het actuele gebruik stijgt, wordt het GPU-budget van elke thread teruggeschroefd: ×0,75 bij een gebruik van meer dan 80%, ×0,5 bij meer dan 90%. De drempels van de monitor zijn 70% (conservatief), 85% (normale bedrijfslimiet) en 95% (OOM-risico).**OOM-terugtrekking en herstel.** Als er toch een out-of-memory-gebeurtenis plaatsvindt:

* wordt de batchgrootte **gehalveerd**, en bij elke opeenvolgende OOM-situatie opnieuw gehalveerd — elke daaropvolgende succesvolle batch verlicht die beperking met één stap
* worden de toewijzingen aan actieve threads teruggebracht tot 70% van hun huidige waarde en schakelt de toewijzer over naar zijn conservatieve strategie, die weer wordt versoepeld na een reeks succesvolle toewijzingen
* onder zware druk schakelt de pijplijn terug van `fused_gpu` naar `tiled_gpu`, en als laatste redmiddel naar `cpu_fallback`

**Host-RAM (Jetson).** Vóór de verwerking schat de CLI het maximale hostgeheugen in op basis van het aantal afbeeldingen en de debayer-modus, en geeft een waarschuwing als het RAM-geheugen plus de door bestanden ondersteunde swapruimte waarschijnlijk onvoldoende is. Daarbij worden de exacte commando’s weergegeven om swapruimte toe te voegen — zie de [NVIDIA Jetson-handleiding](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Controle van de rekenaanpassing

### Systeemdiagnostiek

`chloros-cli selftest` is de snelste manier om te controleren wat de rekenlaag waarneemt:

```bash
chloros-cli selftest
```

De 7 controles hebben betrekking op de versie, de beschikbaarheid van poorten, het opstarten van de backend, `/api/test`, systeeminformatie, de aanwezigheid van het denoiser-model en de gereedheid van CUDA + denoiser. Controle 5 geeft de hardwareregel direct weer:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Controle 7 geeft `CUDA: <bool>, Denoiser: <bool>` weer — beide moeten waar zijn om Texture Aware überhaupt te kunnen gebruiken.

### Backend-logs

De strategie en het aantal workers worden bij het begin van elke run binnen de backend gekozen — er is geen CLI-banner die deze weergeeft. Wanneer iets zich onverwacht gedraagt (een GPU-pad dat terugvalt, een OOM, een denoiser die niet wil laden), is het backend-logboek voor die sessie de plek waar dit zichtbaar wordt:

| Platform | Locatie logbestand |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (één bestand per start) |
| **Linux, via CLI gestarte backend** | eveneens `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Live voortgang

Tijdens een run toont de CLI de live voortgang per thread (Detecteren, Analyseren, Verwerken, Exporteren), gestreamd via Server-Sent Events — de praktische indicatie of Thread 3 de bottleneck is. Zie [Verwerkingspijplijn](processing-pipeline.md).

***

## Volgende stappen

* [Verwerkingspijplijn](processing-pipeline.md) — Inzicht in de 4-thread-pijplijnarchitectuur
* [NVIDIA Jetson-handleiding](../linux/nvidia-jetson-guide.md) — Jetson-specifieke implementatie en optimalisatie
* [CLI: opdrachtregel](../CLI.md) — De CLI-handleiding
* [CLI-referentie](../reference/cli-reference.md) — Uitgebreide lijst met opdrachten voor versie 1.3.0
