# NVIDIA Jetson-handleiding

Chloros op NVIDIA Jetson maakt multispectrale beeldverwerking aan de rand mogelijk — in het veld, op UAV’s en in afgelegen installaties. Chloros detecteert automatisch uw Jetson-model en optimaliseert de verwerkingsstrategie voor uw hardware.

***

## Ondersteunde Jetson-modellen

| Model                | RAM            | Verwerkingsstrategie                                   | Aanbevolen gebruik                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB gedeeld | `GPU_PARALLEL` (4 workers)                            | Maximale prestaties, grote datasets                      |
| **Jetson Orin NX**   | 8-16 GB gedeeld  | `GPU_PARALLEL` (3 workers, 16 GB) / `GPU_SINGLE` (8 GB) | Primaire aanbeveling voor inzet in de lucht en in het veld |
| **Jetson Orin Nano** | 8 GB gedeeld     | `GPU_SINGLE` (1 worker)                               | Edge computing op instapniveau                                 |
| **Jetson Nano**      | 4-8 GB gedeeld   | `GPU_SINGLE` (1 worker)                               | Instapmodel, beperkt geheugen                          |

{% hint style="info" %}
**Oudere Jetson-modellen** (TX2, TX1, Xavier NX) worden mogelijk niet ondersteund. De prestaties variëren afhankelijk van het beschikbare GPU-geheugen en de CUDA-mogelijkheden.
{% endhint %}

***

## Vereisten

* **JetPack 6.x** (nieuwste versie aanbevolen)
* **NVIDIA CUDA** (meegeleverd met JetPack)
* **Chloros+ licentie** (vereist voor toegang tot CLI/SDK)

## Installatie

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Zie [Linux Installatie](linux-installation.md) voor algemene informatie over de installatie van Linux.

***

## Dynamische rekenaanpassing op Jetson

Chloros detecteert automatisch uw Jetson-model en selecteert de optimale verwerkingsstrategie. **Handmatige afstemming is niet nodig.**

### Hoe het werkt

Bij het opstarten maakt Chloros een profiel van uw systeem:

1. **Detecteert het Jetson-model** via `/proc/device-tree/model`
2. **Leest beschikbaar GPU-/gedeeld geheugen**

3.**Selecteert een verwerkingsstrategie** (`GPU_PARALLEL`, `GPU_SINGLE` of `CPU_PARALLEL`)
4. **Stelt het aantal workers, het pijplijntype en de geheugentoewijzing** automatisch in

### Gedrag per model

| Jetson-model                | Strategie       | Werkers | Pijplijn                       | Gelijktijdigheid |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (geheugenefficiënt) | Gesequentialiseerd  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Geserialiseerd  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Geserialiseerd  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (volledig GPU-pad)    | Gelijktijdig  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Gelijktijdig  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** is de ideale keuze voor edge-implementatie — het maakt gebruik van de `GPU_PARALLEL`-strategie met 3 gelijktijdige workers, waardoor echte parallelle GPU-verwerking wordt geboden in een compacte vormfactor.
{% endhint %}

Het belangrijkste verschil tussen de platforms is **het geheugen**. Een Jetson Nano met 8 GB gedeeld geheugen moet afbeeldingen één voor één verwerken met behulp van een geheugenefficiënte tiled-aanpak, terwijl een Orin NX met 16 GB 3 afbeeldingen tegelijkertijd via de GPU kan verwerken met behulp van de fused pipeline met hogere doorvoer.

Zie [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) voor de volledige referentie over compute-aanpassing.

***

## Thermisch beheer

Jetson-apparaten hebben beperkte thermische ruimte, vooral bij inbouw- of luchtvaarttoepassingen. Chloros bevat automatische thermische monitoring en throttling:

| Temperatuur         | Actie                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normale werking — volledige verwerkingssnelheid          |
| **70 °C** (Waarschuwing) | Batchgrootte automatisch verkleinen                   |
| **80 °C** (Kritiek) | Agressieve beperking — lagere gelijktijdigheid         |
| **90 °C** (Uitschakeling) | GPU-verwerking volledig stoppen — afkoeling vereist |

{% hint style="warning" %}
**Zorg voor voldoende ventilatie en warmteafvoer** voor langdurige verwerking, vooral in gesloten behuizingen of systemen in de lucht. Thermische beperking vermindert de verwerkingsdoorvoer om de hardware te beschermen.
{% endhint %}

***

## Geheugenbeheer

Jetson-apparaten maken gebruik van **unified memory** — de GPU en CPU delen hetzelfde fysieke RAM. Dit betekent dat het gerapporteerde VRAM (bijv. 15,3 GB op Orin NX 16 GB) geen speciaal GPU-geheugen is; het wordt gedeeld met het besturingssysteem en andere processen.

### Aanbevelingen voor swapruimte

Voor grote datasets of Texture Aware-debayer-verwerking kan Chloros aanbevelen om swapruimte aan te maken:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Geheugenschattingen per afbeelding:**

* Standaard debayer: ~10 MB per afbeelding
* Texture Aware debayer: ~15 MB per afbeelding

Chloros berekent automatisch het benodigde geheugen op basis van de grootte van uw dataset en waarschuwt u als swap wordt aanbevolen.

### OOM (Out of Memory) Fallback

Als tijdens de verwerking een out-of-memory-situatie wordt gedetecteerd:

1. Verlaagt Chloros automatisch het aantal GPU-workers
2. Schakelt over van de `fused_gpu`-pijplijn naar de `tiled_gpu`-pijplijn (geheugenefficiënter)
3. De verwerking wordt voortgezet met een lagere doorvoer in plaats van dat het systeem crasht

***

## Implementatie in het veld

### Overwegingen met betrekking tot stroomverbruik

| Jetson-model     | Typisch stroomverbruik | Opmerkingen                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C of barrel-aansluiting    |
| Jetson Orin Nano | 7-15 W              | DC-barrel-aansluiting          |
| Jetson Orin NX   | 10-25 W             | DC-barrel-aansluiting          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD of barrel-aansluiting |

Plan uw stroomverbruik voor langdurige verwerking — het piekvermogen wordt verbruikt tijdens de GPU-intensieve Thread 3 (verwerking).

### Aanbevelingen voor opslag

* **NVMe SSD** wordt sterk aanbevolen voor arm64-implementaties
* SD-kaarten zijn te traag voor verwerking — gebruik ze alleen als opstartmedia
* Houd rekening met 2-3x de grootte van uw ruwe beeldgegevens voor de verwerkte output

### Headless-werking via SSH

Chloros CLI is ideaal voor headless Jetson-implementaties:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Geautomatiseerde verwerking met systemd

Maak een systemd-service aan voor geautomatiseerde verwerking:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Koppel deze aan een systemd-timer voor geplande verwerking:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Voorbeeldworkflows

### Basisverwerking met Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK op Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Batchverwerking van meerdere vluchten

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Aanbevolen Jetson-systemen voor gebruik in het veld

Overweeg voor implementaties in het veld en in de lucht deze Jetson Orin NX 16 GB-carrierboardopties:

* **In de lucht/drone**: systemen met trillingsbestendigheid (MIL-STD), lichtgewicht (minder dan 300 g), passieve koeling
* **Robuust veldgebruik**: IP67/IP69K waterdichte behuizingen met PoE GigE-cameraconnectiviteit
* **Minimaal/budget**: Ontwikkelaarskits met aanvullende behuizingen

Neem contact op met [MAPIR Support](https://www.mapir.camera/community/contact) voor specifieke hardwareaanbevelingen voor uw toepassingsscenario.

***

## Volgende stappen

* [Linux Installatie](linux-installation.md) — Algemene details over de installatie van Linux
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — Volledige referentie voor de rekenstrategie
* [Verwerkingspijplijn](../processing-architecture/processing-pipeline.md) — Inzicht in de 4-thread-pijplijn
* [CLI : Opdrachtregel](../CLI.md) — Volledige CLI-referentie
* [API : Python SDK](../api-python-sdk.md) — Volledige SDK-referentie
