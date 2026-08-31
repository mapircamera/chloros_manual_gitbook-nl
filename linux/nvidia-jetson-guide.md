# NVIDIA Jetson-handleiding

Chloros op NVIDIA Jetson maakt multispectrale beeldverwerking aan de rand mogelijk — in het veld, op UAV’s en in afgelegen installaties. Chloros 1.2.0 detecteert uw Jetson-model bij het opstarten en optimaliseert de verwerkingsstrategie voor de gevonden hardware. **Handmatige afstemming is niet nodig.**

***

## Ondersteunde Jetson-modellen

| Model                | RAM            | Verwerkingsstrategie                                     | Aanbevolen gebruik                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB gedeeld | `GPU_PARALLEL` (2 workers)                              | Maximale prestaties, grote datasets                      |
| **Jetson Orin NX**   | 8–16 GB gedeeld | `GPU_PARALLEL` (2 workers, 16 GB) / `GPU_SINGLE` (8 GB) | Eerste aanbeveling voor inzet in de lucht en in het veld |
| **Jetson Orin Nano** | 8 GB gedeeld     | `GPU_SINGLE` (1 worker, sequentieel)                     | Edge-computing op instapniveau                                 |

{% hint style="info" %}
Het arm64-pakket Linux vereist **JetPack 6**, dat beschikbaar is voor de Jetson Orin-serie. Oudere modellen (Nano, TX2, Xavier NX) kunnen JetPack 6 niet uitvoeren en worden niet ondersteund door het huidige pakket.
{% endhint %}

***

## Vereisten

* **JetPack 6.x** (nieuwste versie aanbevolen)
* **NVIDIA CUDA** (meegeleverd met JetPack)
* **Betaald Chloros+-abonnement** — Copper-niveau of hoger (vereist voor alle toegang tot CLI/SDK; wordt serverzijde afgedwongen)

## Installatie

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Zie [Linux Installatie](linux-installation.md) voor algemene informatie over de installatie van Linux, bestandslocaties en probleemoplossing.

{% hint style="info" %}
**Plaats de uitpakmap op snelle opslag.** De gecompileerde binaire bestanden pakken zichzelf bij elke start uit naar een tijdelijke map — wat vanaf een SD-kaart erg traag verloopt. Chloros gebruikt automatisch `/mnt/ssd/tmp` als dit bestaat; stel anders `TMPDIR` in op een pad op je NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamische rekenkrachtanpassing op Jetson

### Hoe het werkt

Bij het opstarten maakt Chloros een profiel van je systeem:

1. **Detecteert het Jetson-model** via `/proc/device-tree/model`
2. **Leest het beschikbare gedeelde GPU-/CPU-geheugen** (Jetson maakt gebruik van unified memory)
3. **Kiest een verwerkingsstrategie** (`GPU_PARALLEL`, `GPU_SINGLE` of `CPU_PARALLEL`)
4. **Stelt het aantal workers, het pijplijntype en de geheugentoewijzing** automatisch in

De keuze wordt bepaald door het **totale gedeelde RAM**, niet door de modelnaam:

* **Minder dan 12 GB totaal RAM**(alle Jetsons met 8 GB): `GPU_SINGLE` met**1 worker — doelbewuste sequentiële verwerking**. Er is te weinig geheugen voor gelijktijdige workers, dus worden afbeeldingen één voor één verwerkt. Op Jetsons met**8 GB of minder** slaat Thread 3 de workerpool volledig over en voert de verwerking per afbeelding in-process uit.
* **12 GB of meer**(Orin NX 16 GB, AGX Orin): het unified memory voldoet aan de vereisten voor `GPU_PARALLEL`, maar het aantal workers is**op Jetson beperkt tot 2** — de GPU, het RAM-geheugen van de workerprocessen en hun CUDA-contexten per worker putten allemaal uit dezelfde gedeelde pool, waardoor bij meer workers het risico op fouten door onvoldoende geheugen toeneemt.

Je kunt de automatische keuze overschrijven met de omgevingsvariabele `CHLOROS_STRATEGY` — zie [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Gedrag per model

| Jetson-model                | Strategie       | Werkers | Uitvoering                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sequentiële lus binnen het proces (`tiled_gpu` bij geheugendruk) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | Sequentiële in-process-lus                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Gelijktijdige werkprocessen, `fused_gpu`-pad  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Gelijktijdige werkprocessen, `fused_gpu`-pad  |

Het belangrijkste verschil tussen de platforms is **het geheugen**. Een Jetson met 8 GB moet bij hoge belasting afbeeldingen één voor één verwerken met behulp van een geheugenefficiënte &#x27;tiled&#x27;-aanpak, terwijl een Orin met 16 GB of meer twee afbeeldingen tegelijk via de GPU kan verwerken met behulp van de &#x27;fused pipeline&#x27; met hogere doorvoercapaciteit.

### GPU-budget per model

Elk Jetson-model heeft ook een hardwareprofiel dat bepaalt hoeveel van de gedeelde pool de verwerking mag beanspriken, en dat de batchgroottes schaalt:

| Model | Maximale GPU-budget | Vermenigvuldigingsfactor batchgrootte | Gereserveerd voor systeem/weergave |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Het gedetecteerde RAM-geheugen past het profiel aan: bij een Jetson die **16 GB of meer** rapporteert, wordt de batchvermenigvuldigingsfactor verhoogd met ×1,2. De basisbatchgrootte vóór vermenigvuldigingsfactoren is 8 afbeeldingen.

Zie [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) voor de volledige referentie over rekenkrachtanpassing.

***

## GPU-frequentiebeperking voor Texture Aware op Nano en Orin Nano

De Texture Aware-debayer voert GPU-neurale-netwerk-inferentie uit, wat bij volledige GPU-kloksnelheid **waarschuwingen voor overstroom**kan activeren op Jetson-modellen met laag vermogen (klasse 10-15 W). Voordat de Texture Aware-verwerking op een**Jetson Nano of Orin Nano**wordt gestart, controleert Chloros de maximale frequentie van de GPU en beperkt deze tot**510 MHz** (510000000) als deze op dat moment hoger is:

* Als het commando CLI de sysfs-node voor de GPU-frequentie kan schrijven, wordt de limiet **automatisch toegepast** en wordt er een bevestiging weergegeven.
* Zo niet (root-rechten vereist), dan geeft CLI het exacte `sudo`-commando weer om de beperking handmatig toe te passen, wacht even zodat je het kunt lezen en gaat dan verder — de verwerking loopt nog steeds, maar er kunnen waarschuwingen voor overstroom verschijnen.

Om de limiet zelf toe te passen vóór de verwerking:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modellen met een hoger vermogen (Orin NX 25W, AGX Orin 60W) draaien op volledige GPU-snelheid; er wordt geen limiet toegepast. De standaard debayer activeert de limiet nooit op welk model dan ook.

{% hint style="info" %}
**Texture Aware op Jetson verwerkt altijd één afbeelding tegelijk.** Elke worker zou zijn eigen CUDA-context (~1 GB) nodig hebben, plus een eigen kopie van het denoiser-model, wat het unified memory niet aankan — daarom is het Texture Aware-pad op Jetson vastgezet aan één enkele worker, waarbij de toegang tot de GPU wordt geserialiseerd. Verwacht dat Texture Aware op elke Jetson aanzienlijk langzamer zal zijn dan Standard.
{% endhint %}

***

## Thermisch beheer

Jetson-apparaten hebben beperkte thermische speelruimte, vooral bij gebruik in gesloten ruimtes of in vliegtuigen. Chloros bewaakt de SoC-temperatuur en past de batchgroottes automatisch aan:

| Temperatuur         | Actie                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normale werking — volledige verwerkingssnelheid          |
| **70 °C** (Waarschuwing) | Batchgrootte wordt geleidelijk verkleind (100% → 50% tussen 70 °C en 80 °C) |
| **80 °C** (Kritiek) | Agressieve beperking (50% → 0% tussen 80 °C en 90 °C) |
| **90 °C** (Uitschakeling) | GPU-verwerking volledig stoppen — afkoeling vereist |

{% hint style="warning" %}
**Zorg voor voldoende ventilatie en warmteafvoer** bij langdurige verwerking, met name in gesloten veldbehuizingen of luchtvaartsystemen. Thermische beperking vermindert de verwerkingssnelheid om de hardware te beschermen.
{% endhint %}

***

## Geheugenbeheer

Jetson-apparaten maken gebruik van **unified memory** — de GPU en CPU delen hetzelfde fysieke RAM. Het gerapporteerde VRAM (bijv. ~15,3 GB op een Orin NX 16 GB) is geen speciaal GPU-geheugen; het is hetzelfde RAM dat het besturingssysteem en alle andere processen gebruiken.

### Waarschuwing en aanbevelingen inzake swap

Voordat de verwerking op Jetson begint, telt de CLI de RAW-afbeeldingen in uw invoermap (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-voorbeelden worden niet meegeteld), schat het maximale geheugen dat de run nodig heeft, en **geeft een waarschuwing vóór het starten** als het RAM + swap-geheugen waarschijnlijk onvoldoende is. De waarschuwing heeft als kop `LOW MEMORY WARNING - Jetson Detected`, geeft het aantal afbeeldingen weer, het RAM-geheugen, de huidige swapruimte en de geschatte piek, en geeft vervolgens de exacte `fallocate` / `chmod` / `mkswap` / `swapon`-opdrachten die zijn afgestemd op de omvang van je project (nooit kleiner dan 8 GB). Het programma pauzeert een paar seconden zodat het bericht niet verloren gaat in de scrollback, waarna de verwerking wordt voortgezet.**Geheugenschattingen die door de waarschuwing worden gebruikt:**

| Debayer-modus | Basis | Per afbeelding |
| --- | --- | --- |
| Standaard | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (model + Python-runtime) | ~15 MB |

De waarschuwing wordt geactiveerd wanneer de geschatte piek het RAM-geheugen plus de swapruimte minus een veiligheidsmarge van 1 GB overschrijdt, en er wordt alleen rekening gehouden met **door bestanden ondersteunde** swapruimte — een configuratie die uitsluitend op zram draait, zal nog steeds worden gemarkeerd.

Om handmatig swapruimte toe te voegen (bijvoorbeeld: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

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
```### OOM (Out of Memory) -afhandeling

Tijdens de verwerking controleert Chloros het GPU-geheugen en schakelt het op een soepele manier over naar een lagere prestatieniveau in plaats van te crashen:

1. Wanneer het GPU-geheugengebruik **85%** overschrijdt, worden de batchgroottes preventief verkleind
2. Als er toch een ‘out-of-memory’-gebeurtenis optreedt, wordt de batchgrootte **gehalveerd**, en bij elke opeenvolgende OOM nogmaals gehalveerd; elke daaropvolgende succesvolle batch draait die straf één stap terug
3. Bij aanhoudende druk schakelt de pijplijn over van `fused_gpu` naar het geheugenefficiënte `tiled_gpu`-pad, en als laatste redmiddel naar CPU-verwerking

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
* SD-kaarten zijn te traag voor verwerking — gebruik ze uitsluitend als opstartmedium
* Houd rekening met 2-3 keer de grootte van je ruwe beeldgegevens voor de verwerkte output

### Headless-werking via SSH

Chloros CLI is ideaal voor headless Jetson-implementaties:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Altijd actieve backend voor LATTICE / DAQ-E-tijdsynchronisatie

Als uw Jetson LATTICE-camera’s of DAQ-E-lichtsensoren headless aanstuurt, schakel dan de backend-systemd-service in zodat de PTP-grandmaster continu draait (de service is geïnstalleerd maar standaard niet ingeschakeld):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Zie [Linux Installatie](linux-installation.md#always-on-ptp-for-headless-hosts) voor details, waaronder hoe het pakket ervoor zorgt dat PTP-poorten 319/320 zonder root-rechten kunnen worden toegewezen.

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

`chloros-cli process` geeft een niet-nul-waarde terug wanneer een run die producten heeft aangevraagd geen afbeeldingen schrijft, zodat de foutstatus van systemd zinvol is voor monitoring.

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
    --indices NDVI NDRE GNDVI
```

### Python en SDK op Jetson

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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Aanbevolen Jetson-systemen voor gebruik in het veld

Overweeg voor inzet in het veld en vanuit de lucht deze Jetson Orin NX 16 GB-carrierboardopties:

* **In de lucht/drone**: Systemen met trillingsbestendigheid (MIL-STD), lichtgewicht (minder dan 300 g), passieve koeling
* **Robuust veldgebruik**: IP67/IP69K waterdichte behuizingen met PoE GigE-cameraconnectiviteit
* **Minimaal/budget**: Ontwikkelaarskits met aanvullende behuizingen

Neem contact op met [MAPIR Support](https://www.mapir.camera/community/contact) voor specifieke hardwareaanbevelingen voor uw toepassingsscenario.

***

## Volgende stappen

* [Linux-installatie](linux-installation.md) — Algemene details over de installatie van Linux
* [Dynamische rekenkrachtadaptatie](../processing-architecture/dynamic-compute-adaptation.md) — Volledige referentie voor rekenstrategieën
* [Verwerkingspijplijn](../processing-architecture/processing-pipeline.md) — Inzicht in de 4-thread-pijplijn
* [CLI : Opdrachtregel](../CLI.md) — De CLI-handleiding
* [API : Python SDK](../api-python-sdk.md) — De SDK-handleiding
* [CLI-referentie](../reference/cli-reference.md) en [SDK-referentie](../reference/sdk-reference.md) — Uitgebreide lijst met commando’s/API voor versie 1.2.0
