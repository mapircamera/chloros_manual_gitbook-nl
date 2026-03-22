# Installatie van Linux

Chloros wordt voor Linux geleverd in de vorm van `.deb`-pakketten die de CLI en de backend installeren. De Python SDK wordt apart geïnstalleerd via pip.

***

## Linux amd64 (x86_64)

### Systeemvereisten

| Vereiste | Minimaal | Aanbevolen |
| --- | --- | --- |
| **Distributie** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Processor** | x86_64 (Intel/AMD) | Intel Core i7 of beter |
| **Geheugen (RAM)** | 8 GB | 16 GB of meer |
| **Grafische kaart** | Geen (CPU-verwerking) | NVIDIA GPU met 4 GB+ VRAM |
| **Opslagruimte** | 2 GB vrije ruimte | SSD met 10 GB+ vrije ruimte |
| **Python** | Python 3.7+ (voor SDK) | Python 3.10+ |

### Installatie

Download het `.deb`-pakket en installeer:

```bash
sudo dpkg -i chloros-amd64.deb
```

Controleer de installatie:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Systeemvereisten

| Vereiste | Minimaal | Aanbevolen |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson met JetPack 6 | Jetson Orin NX 16 GB of AGX Orin |
| **JetPack** | JetPack 6.x | Nieuwste JetPack 6 |
| **Geheugen (RAM)** | 8 GB (gedeeld GPU/CPU) | 16 GB+ gedeeld |
| **Opslag** | 2 GB vrije ruimte | NVMe SSD met 10 GB+ vrije ruimte |
| **Python** | Python 3.7+ (voor SDK) | Python 3.10+ |

### Installatie

Download het JetPack 6 `.deb`-pakket en installeer:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Controleer de installatie:

```bash
chloros-cli --version
```

Raadpleeg de [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) voor gedetailleerde informatie over de configuratie van Jetson, inclusief thermisch beheer en implementatie in het veld.

***

## Python SDK Installatie (Alle Linux)

De Python SDK wordt apart geïnstalleerd via pip en werkt op zowel amd64 als arm64:

```bash
pip install chloros-sdk
```

Om optionele ondersteuning voor voortgangsstreaming toe te voegen:

```bash
pip install chloros-sdk[progress]
```

Controleer de SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Het `.deb`-pakket installeert de Chloros, CLI en backend. Het Python SDK is een apart pip-pakket dat via een lokale HTTP API met de backend communiceert.
{% endhint %}

***

## Configuratiemappen

Chloros op Linux volgt de [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Doel | Linux Pad | Windows Equivalent |
| --- | --- | --- |
| **Configuratie** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Gegevens / Projecten** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Inloggegevens** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Locaties van uitvoerbare backend-bestanden

Het `.deb`-pakket installeert de backend op een standaardlocatie. De CLI en SDK detecteren het backend-pad automatisch:

| Installatiemethode | Backend-pad |
| --- | --- |
| `.deb`-pakket | `/usr/lib/chloros/chloros-backend` |
| Handmatig / aangepast | `/opt/mapir/chloros/backend/chloros-backend` |

U kunt het backend-pad overschrijven met de vlag `--backend-exe` CLI of de constructorparameter `backend_exe` SDK.

***

## Eerste installatie

### 1. Activeer uw licentie

Een Chloros+ licentie is vereist voor toegang tot CLI en SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Controleer de status van uw licentie

```bash
chloros-cli status
```

### 3. Verwerk uw eerste dataset

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Voer systeemdiagnostiek uit

Controleer of uw systeem correct is geconfigureerd:

```bash
chloros-cli selftest
```

Dit voert 7 diagnostische controles uit, waaronder versie, opstarten van de backend, API connectiviteit en beschikbaarheid van CUDA/GPU.

***

## Voorbeelden van Bash-scripts

### Meerdere datasets verwerken

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Verwerken met aangepaste instellingen

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Geautomatiseerde verwerking met Cron

Voeg dit toe aan uw crontab (`crontab -e`) om nieuwe datasets automatisch te verwerken:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Voorbeeld

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Probleemoplossing

### CLI niet gevonden na installatie

Als `chloros-cli` niet wordt gevonden na het installeren van het `.deb`-pakket:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Toegang geweigerd

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Backend kan niet worden gestart

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA niet gedetecteerd

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Ontbrekende gedeelde bibliotheken

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Chloros bijwerken op Linux

Gebruik de ingebouwde updatecommando om te controleren op updates en deze te installeren:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Volgende stappen

* [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) — Jetson-specifieke optimalisatie en implementatie
* [CLI : Opdrachtregel](../CLI.md) — Volledige CLI-opdrachtreferentie
* [API : Python SDK](../api-python-sdk.md) — Volledige SDK-referentie
* [Dynamische rekenaanpassing](../processing-architecture/dynamic-compute-adaptation.md) — Hoe Chloros zich aanpast aan uw hardware
