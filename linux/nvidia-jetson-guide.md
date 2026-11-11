# NVIDIA Jetson-handleiding

Chloros
op NVIDIA Jetson maakt multispectrale beeldverwerking aan de rand mogelijk — in het veld, op UAV’s en in afgelegen installaties. Chloros
1.3.0 detecteert je Jetson-model bij het opstarten en optimaliseert de verwerkingsstrategie voor de gevonden hardware. **Handmatige afstemming is niet nodig.**
{% hint style="info" %}
**Het Jetson-pakket 1.3.0 voert de verwerking uit op de GPU van de Jetson.** `chloros_1.3.0_arm64_jp6.deb` is gebouwd voor JetPack 6 en maakt gebruik van de CUDA-build van PyTorch uit JetPack 6, waardoor de verwerking – inclusief de ‘Texture Aware’-debayer – op de GPU van de Orin draait. Het pakket bevat zelf geen CUDA: het maakt gebruik van de CUDA- en cuDNN-bibliotheken die JetPack 6 installeert (`sudo apt install nvidia-jetpack` op een image waar deze ontbreken). Wanneer er geen CUDA-GPU beschikbaar is, vindt de verwerking plaats op de CPU en maakt Texture Aware gebruik van de standaard debayer.
{% endhint %}

***

## Ondersteunde Jetson-modellen

| Model                | RAM            | Verwerkingsstrategie                                     | Aanbevolen gebruik                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB gedeeld | `GPU_PARALLEL` (2 workers)                              | Maximale prestaties, grote datasets                      |
| **Jetson Orin NX**   | 8–16 GB gedeeld  | `GPU_PARALLEL` (2 workers, 16 GB) / `GPU_SINGLE` (8 GB)   | Eerste aanbeveling voor implementatie in het veld              |
| **Jetson Orin Nano** | 8 GB gedeeld     | `GPU_SINGLE` (1 worker, sequentieel)                     | Edge-computing op instapniveau                                 |

{% hint style="info" %}
Het Linux
-arm64-pakket vereist **JetPack 6**, dat beschikbaar is op de Jetson Orin-serie. Oudere modellen (Nano, TX2, Xavier NX) kunnen JetPack 6 niet uitvoeren en worden niet ondersteund door het huidige pakket.
{% endhint %}

***

## Vereisten

* **JetPack 6.x** (nieuwste versie aanbevolen)
* **NVIDIA CUDA en cuDNN** uit JetPack 6 (`sudo apt install nvidia-jetpack` als deze ontbreken in je image)
* **Betaald Chloros
+-abonnement** — Copper-niveau of hoger (vereist voor alle CLI
-/SDK
-toegang; wordt server-side afgedwongen)

## Installatie

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` installeert de Python
-SDK
voor het systeem `python3`, dat bij deze build hoort. Alleen in een virtualenv moet het meegeleverde wheel worden geïnstalleerd: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (volledig offline). Zie [Linux
-installatie](linux-installation.md) voor algemene details over de Linux
-installatie, bestandslocaties en het oplossen van problemen.

{% hint style="info" %}
**Geen uitpakken meer bij het opstarten.** Vanaf versie 1.3.0 worden de backend en de CLI
&#x27; ongedicht geïnstalleerd onder `/usr/lib/chloros/`, zodat ze zichzelf niet langer bij elke start naar een tijdelijke map uitpakken. Tijdelijke bestanden worden nog steeds automatisch naar `/mnt/ssd/tmp` verplaatst als die map bestaat; anders kunt u `TMPDIR` instellen op een pad op uw NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamische rekenkracht-aanpassing op Jetson

### Hoe het werkt

Bij het opstarten stelt Chloros
een profiel van je systeem op:

1. **Detecteert het Jetson-model** via `/proc/device-tree/model`
2. **Leest het beschikbare gedeelde GPU-/CPU-geheugen** (Jetson maakt gebruik van unified memory)
3. **Vraagt PyTorch of het CUDA kan gebruiken** — als CUDA en cuDNN van JetPack 6 zijn geïnstalleerd, is dit mogelijk
4. **Kiest automatisch een verwerkingsstrategie** (`GPU_PARALLEL`, `GPU_SINGLE` of `CPU_PARALLEL`) en het aantal workers

De keuze wordt bepaald door het **totale gedeelde RAM-geheugen**, niet door de modelnaam:

* **Bij minder dan 12 GB totaal RAM-geheugen**(alle Jetsons met 8 GB):**1 worker — bewuste sequentiële verwerking**. Er is te weinig geheugen voor gelijktijdige workers, dus worden afbeeldingen één voor één verwerkt. Op Jetsons met**8 GB of minder** slaat Thread 3 de werkpool volledig over en voert het de bewerkingen per afbeelding in-process uit. In het backend-logboek wordt deze strategie `GPU_SINGLE` genoemd.
* **12 GB of meer**(Orin NX 16 GB, AGX Orin): het unified memory voldoet aan de vereisten voor `GPU_PARALLEL`, maar het aantal workers is**op Jetson beperkt tot 2** — de GPU, het RAM-geheugen van de workerprocessen en de CUDA-contexten per worker putten allemaal uit dezelfde gedeelde pool, waardoor bij meer workers het risico op fouten door onvoldoende geheugen toeneemt.
* **Geen bruikbare CUDA** (CUDA-bibliotheken van JetPack ontbreken): `CPU_PARALLEL` bij 12 GB of meer, hieronder één image tegelijk.

Je kunt de automatische keuze overschrijven met de omgevingsvariabele `CHLOROS_STRATEGY` — zie [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Gedrag per model

| Jetson-model                | Strategie       | Werkers | Uitvoering                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sequentiële in-process-lus (`tiled_gpu` bij geheugendruk) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sequentiële lus binnen het proces                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Gelijktijdige werkprocessen, `fused_gpu`-pad  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Gelijktijdige werkprocessen, `fused_gpu`-pad  |

Het belangrijkste verschil tussen de platforms is **het geheugen**. Een Jetson met 8 GB moet bij hoge belasting afbeeldingen één voor één verwerken met behulp van een geheugenefficiënte &#x27;tiled&#x27;-aanpak, terwijl een Orin met 16 GB of meer twee afbeeldingen tegelijk via de GPU kan verwerken met behulp van de &#x27;fused pipeline&#x27; met hogere doorvoercapaciteit.

### GPU-budget per model

Elk Jetson-model heeft ook een hardwareprofiel dat bepaalt hoeveel van de gedeelde pool de verwerking mag gebruiken, en dat de batchgroottes schaalt:

| Model | Maximum GPU-budget | Vermenigvuldigingsfactor batchgrootte | Gereserveerd voor systeem/scherm |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Het gedetecteerde RAM-geheugen past het profiel aan: bij een Jetson die **16 GB of meer** rapporteert, wordt de batchvermenigvuldigingsfactor verhoogd met ×1,2. De basisbatchgrootte vóór vermenigvuldigingsfactoren is 8 afbeeldingen.

Zie [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) voor de volledige referentie over rekenkrachtadaptatie.

***

## GPU-frequentiebeperking voor Texture Aware op Nano en Orin Nano

De Texture Aware-debayer voert GPU-neurale-netwerkinferentie uit, wat bij volledige GPU-kloksnelheid **waarschuwingen voor overstroom**kan activeren op Jetson-modellen met laag vermogen (klasse 10-15 W). Wanneer je `--debayer texture-aware` doorgeeft op een**Jetson Nano of Orin Nano**, controleert de CLI
de maximale frequentie van de GPU en beperkt deze tot **510 MHz** (510000000) als deze momenteel hoger is:

* Als de CLI
het sysfs-knooppunt voor de GPU-frequentie kan schrijven, wordt de limiet **automatisch toegepast** en wordt er een bevestiging weergegeven.
* Zo niet (root-rechten vereist), geeft de CLI
het exacte `sudo`-commando weer om de limiet handmatig toe te passen, wacht even zodat je het kunt lezen en gaat dan verder — de verwerking loopt nog steeds, maar er kunnen waarschuwingen voor overstroom verschijnen.

Om de limiet zelf toe te passen vóór de verwerking:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modellen met een hoger vermogen (Orin NX 25W, AGX Orin 60W) draaien op volledige GPU-snelheid; er wordt geen limiet toegepast. De Standard-debayer activeert de limiet nooit op welk model dan ook.

{% hint style="info" %}
**Texture Aware op Jetson verwerkt altijd één afbeelding tegelijk.** Elke worker zou zijn eigen CUDA-context (~1 GB) nodig hebben, plus een eigen kopie van het denoiser-model, wat het unified memory niet aankan — daarom is het Texture Aware-pad op Jetson vastgezet aan één enkele worker, waarbij de toegang tot de GPU wordt geserialiseerd. Verwacht dat Texture Aware op elke Jetson aanzienlijk langzamer zal zijn dan Standard. Texture Aware is van toepassing op Survey3
-afbeeldingen: LATTICE-opnames krijgen altijd de Standard-debayer. Zonder een bruikbare CUDA-GPU wordt Standard gebruikt, en de backend logt één regel `[DEBAYER] … needs a CUDA GPU …` waarin wordt uitgelegd waarom (weergegeven met `chloros-cli --verbose`).
{% endhint %}

***

## Warmtebeheer

Jetson-apparaten hebben een beperkte thermische marge, vooral bij gebruik in gesloten ruimtes of in de lucht. Chloros
houdt de SoC-temperatuur in de gaten en past de batchgroottes automatisch aan:

| Temperatuur         | Actie                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normale werking — volledige verwerkingssnelheid          |
| **70 °C** (Waarschuwing) | Batchgrootte wordt geleidelijk verkleind (100% → 50% tussen 70 °C en 80 °C) |
| **80 °C** (Kritiek) | Agressieve beperking (50% → 0% tussen 80 °C en 90 °C) |
| **90 °C** (Uitschakeling) | GPU-verwerking volledig stoppen — afkoelen vereist |

{% hint style="warning" %}
**Zorg voor voldoende ventilatie en warmteafvoer** bij langdurige verwerking, met name in gesloten veldbehuizingen of luchtvaartuigsystemen. Thermische beperking vermindert de verwerkingsdoorvoer om de hardware te beschermen.
{% endhint %}

***

## Geheugenbeheer

Jetson-apparaten maken gebruik van **unified memory** — de GPU en CPU delen hetzelfde fysieke RAM-geheugen. Het gerapporteerde VRAM (bijv. ~15,3 GB op een Orin NX 16 GB) is geen speciaal GPU-geheugen; het is hetzelfde RAM-geheugen dat het besturingssysteem en alle andere processen gebruiken.

### Waarschuwing en aanbevelingen met betrekking tot swap

Voordat de verwerking op Jetson begint, telt de CLI
het aantal RAW-afbeeldingen in uw invoermap (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-voorbeelden worden niet meegeteld), schat het maximale geheugen dat de run nodig heeft, en **geeft een waarschuwing voordat het start** als RAM + swap waarschijnlijk onvoldoende is. De waarschuwing heeft als kop `LOW MEMORY WARNING - Jetson Detected`, geeft het aantal afbeeldingen, het RAM-geheugen, de huidige swapruimte en de geschatte piek, en geeft vervolgens de exacte `fallocate` / `chmod` / `mkswap` / `swapon`-opdrachten weer, afgestemd op uw project (nooit kleiner dan 8 GB). Het programma pauzeert een paar seconden zodat het bericht niet verloren gaat in de scrollback, waarna de verwerking wordt voortgezet.**Geheugenschattingen gebruikt door de waarschuwing:**

| Debayer-modus | Basis | Per afbeelding |
| --- | --- | --- |
| Standaard | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (model + Python-runtime) | ~15 MB |

De waarschuwing wordt geactiveerd wanneer de geschatte piek het RAM-geheugen plus de swapruimte minus een veiligheidsmarge van 1 GB overschrijdt, en er wordt alleen rekening gehouden met **op bestanden gebaseerde** swapruimte — een configuratie die uitsluitend op zram draait, zal nog steeds worden gemarkeerd.

Om handmatig swap toe te voegen (voorbeeld: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### OOM (Out of Memory) afhandeling

Tijdens de verwerking houdt Chloros
het GPU-geheugen in de gaten en schakelt het op een soepele manier over naar een lagere prestatieniveau in plaats van te crashen:

1. Wanneer het GPU-geheugengebruik **85%** overschrijdt, worden de batchgroottes preventief verkleind
2. Als er toch een out-of-memory-gebeurtenis optreedt, wordt de batchgrootte **gehalveerd**, en bij elke opeenvolgende OOM opnieuw gehalveerd; elke daaropvolgende succesvolle batch draait die beperking één stap terug
3. Bij aanhoudende druk schakelt de pijplijn over van `fused_gpu` naar het geheugenefficiënte `tiled_gpu`-pad, en als laatste redmiddel naar CPU-verwerking

Omdat de GPU het RAM-geheugen van de Jetson deelt, wordt een run nog steeds beperkt door het RAM-geheugen plus de swapruimte — de bovenstaande swapwaarschuwing is de controle waar je op moet letten.

***

## Implementatie in de praktijk

### Overwegingen met betrekking tot stroomverbruik

| Jetson-model     | Typisch stroomverbruik | Opmerkingen                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 W              | DC-barrel-aansluiting          |
| Jetson Orin NX   | 10-25 W             | DC-barrel-aansluiting          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD of barrel-aansluiting |

Plan uw stroomverbruik voor langdurige verwerking — het piekvermogen wordt verbruikt tijdens de GPU-intensieve Thread 3 (verwerking).

### Aanbevelingen voor opslag

* **NVMe SSD** wordt sterk aanbevolen voor arm64-implementaties
* SD-kaarten zijn te traag voor verwerking — gebruik ze alleen als opstartmedium
* Houd rekening met 2-3x de grootte van je ruwe beeldgegevens voor de verwerkte output

### Headless-bediening via SSH
De Chloros-CLI
van

Chloros
is ideaal voor headless Jetson-implementaties:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Altijd actieve backend voor LATTICE / DAQ-E-tijdsynchronisatie

Als je Jetson LATTICE-camera’s of DAQ-E-lichtsensoren headless aanstuurt, schakel dan de backend-systemd-service **als jouw gebruiker** in, zodat de PTP-grandmaster continu draait en SDK
-scripts een actieve backend vinden (de unit is geïnstalleerd maar standaard niet ingeschakeld):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

De backend draait dan onder uw gebruikersaccount: de opnames die hij schrijft zijn van u, en hij gebruikt uw `chloros-cli login`. Zie [Linux
-installatie](linux-installation.md#always-on-ptp-for-headless-hosts) voor details, waaronder de root-`chloros-backend.service` en hoe het pakket ervoor zorgt dat PTP-poorten 319/320 zonder root-rechten kunnen worden gebonden.

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

`chloros-cli process` geeft een waarde anders dan nul terug wanneer een run die producten heeft aangevraagd geen afbeeldingen schrijft, zodat de foutstatus van systemd zinvol is voor monitoring.

Combineer dit met een systemd-timer voor geplande verwerking:

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

### Basisverwerking op Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### Python
-SDK
op Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Aanbevolen Jetson-systemen voor gebruik in het veld

Overweeg voor inzet in het veld en in de lucht deze Jetson Orin NX 16 GB-carrierboardopties:

* **In de lucht/drone**: Systemen met trillingsbestendigheid (MIL-STD), lichtgewicht (minder dan 300 g), passieve koeling
* **Robuust veldgebruik**: IP67/IP69K waterdichte behuizingen met PoE GigE-cameraconnectiviteit
* **Minimaal/budget**: Ontwikkelaarskits met aanvullende behuizingen

Neem contact op met [MAPIR
-ondersteuning](https://www.mapir.camera/community/contact) voor specifieke hardwareaanbevelingen voor uw toepassingsscenario.

***

## Volgende stappen

* [Linux
-installatie](linux-installation.md) — Algemene informatie over de installatie van Linux

* [Dynamische rekenkrachtadaptatie](../processing-architecture/dynamic-compute-adaptation.md) — Volledige referentie voor rekenkrachtstrategieën
* [Verwerkingspijplijn](../processing-architecture/processing-pipeline.md) — Inzicht in de 4-thread-pijplijn
* [CLI: opdrachtregel](../CLI.md) — De CLI
-handleiding
* [API
: Python
SDK
](../api-python-sdk.md) — De SDK
-handleiding
* [Referentie CLI
](../reference/cli-reference.md) en [Referentie SDK
](../reference/sdk-reference.md) — Uitgebreide lijsten met commando’s en API
’s voor versie 1.3.0
