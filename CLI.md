# CLI : Opdrachtregel

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>De **Chloros CLI** biedt krachtige opdrachtregeltoegang tot de Chloros-beeldverwerkingsengine, waardoor automatisering, scripting en headless-werking voor uw beeldverwerkingsworkflows mogelijk worden.

### Belangrijkste kenmerken

* 🚀 **Automatisering** - Batchverwerking van meerdere datasets via scripts
* 🔗 **Integratie** - Integreer in bestaande workflows en pijplijnen
* 💻 **Headless-werking** - Draai zonder GUI
* 🌍 **Meertalig** - Ondersteuning voor 38 talen
* ⚡ **Parallelle verwerking** - [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) optimaliseert automatisch voor uw hardware

### Vereisten

| Vereiste          | Details                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Besturingssysteem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licentie**          | Chloros+ ([betaald abonnement vereist](https://cloud.mapir.camera/pricing)) |
| **Geheugen**           | Minimaal 8 GB RAM (16 GB aanbevolen)                                  |
| **Internet**         | Vereist voor licentieactivering                                     |
| **Schijfruimte**       | Varieert afhankelijk van de projectgrootte                                              |

{% hint style="warning" %}
**Licentievereiste**: Voor CLI is een betaald Chloros+-abonnement vereist. Standaard (gratis) abonnementen hebben geen toegang tot CLI. Ga naar [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) om te upgraden.
{% endhint %}

## Snel aan de slag

### Installatie

#### Windows

De CLI wordt automatisch meegeleverd met het Chloros-installatieprogramma:

1. Download en voer **Chloros Installer.exe** uit
2. Voltooi de installatiewizard
3. CLI geïnstalleerd in: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Het installatieprogramma voegt automatisch `chloros-cli` toe aan de PATH van uw systeem. Start uw terminal opnieuw op na de installatie.
{% endhint %}

#### Linux

Installeer het `.deb`-pakket voor uw architectuur:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Raadpleeg [Linux Installatie](linux/linux-installation.md) voor gedetailleerde informatie over de installatie van Linux.

### Eerste installatie

Activeer uw Chloros+ licentie voordat u de CLI gebruikt:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Basisgebruik

Verwerk een map met standaardinstellingen:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Commando-overzicht

### Algemene syntaxis

```
chloros-cli [global-options] <command> [command-options]
```

***

## Commando&#x27;s

### `process` - Afbeeldingen verwerken

Verwerk afbeeldingen in een map met kalibratie.

**Syntaxis:**

```bash
chloros-cli process <input-folder> [options]
```

**Voorbeelden:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Opties voor het verwerkingscommando

| Optie                | Type    | Standaard        | Beschrijving                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Pad    | _Vereist_     | Map met RAW/JPG multispectrale afbeeldingen                                         |
| `-o, --output`        | Pad    | Hetzelfde als invoer  | Uitvoermap voor verwerkte afbeeldingen                                                     |
| `-n, --project-name`  | Tekst  | Automatisch gegenereerd | Aangepaste projectnaam                                                                    |
| `--vignette`          | Vlag    | Ingeschakeld        | Vignetcorrectie inschakelen                                                             |
| `--no-vignette`       | Vlag    | -              | Vignetcorrectie uitschakelen                                                            |
| `--reflectance`       | Vlag    | Ingeschakeld        | Reflectiekalibratie inschakelen                                                         |
| `--no-reflectance`    | Vlag    | -              | Reflectiekalibratie uitschakelen                                                        |
| `--ppk`               | Vlag    | Uitgeschakeld       | Pas PPK-correcties toe op basis van .daq-lichtsensorgegevens                                      |
| `--format`            | Keuze  | TIFF (16-bits)  | Uitvoerformaat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Geheel getal | Auto           | Minimale doelgrootte in pixels voor detectie van kalibratiepaneel                          |
| `--target-clustering` | Geheel getal | Auto           | Drempelwaarde voor clustering van doelen (0-100)                                                    |
| `--debayer`           | Keuze  | `standard`     | Debayer-methode: `standard` of `texture-aware` (alleen Chloros+)                          |
| `--target`, `--targets` | Vlag  | Uitgeschakeld       | Zoek alleen naar kalibratiedoelen in een submap &quot;target&quot; of &quot;targets&quot; (versnelt de verwerking) |
| `--indices`           | Lijst    | Geen           | Te berekenen vegetatie-indexen (bijv. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Tekst  | Geen           | Belichting vergrendelen voor cameramodel (Pin 1)                                                 |
| `--exposure-pin-2`    | Tekst  | Geen           | Belichting vergrendelen voor cameramodel (Pin 2)                                                 |
| `--recal-interval`    | Geheel getal | Auto           | Herkalibratie-interval in seconden                                                      |
| `--timezone-offset`   | Geheel getal | 0              | Tijdzone-offset in uren                                                               |

***

### `login` - Account verifiëren

Log in met uw Chloros+ inloggegevens om CLI-verwerking in te schakelen.

**Syntaxis:**

```bash
chloros-cli login <email> <password>
```

**Voorbeeld:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Speciale tekens**: Gebruik enkele aanhalingstekens rond wachtwoorden die tekens zoals `$`, `!` of spaties bevatten.
{% endhint %}

**Uitvoer:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Inloggegevens wissen

Wis opgeslagen inloggegevens en log uit bij uw account.

**Syntaxis:**

```bash
chloros-cli logout
```

**Voorbeeld:**

```bash
chloros-cli logout
```

**Uitvoer:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK-gebruikers**: De Python SDK biedt ook een programmatische `logout()`-methode voor het wissen van inloggegevens binnen Python-scripts. Raadpleeg de [Python SDK documentatie](api-python-sdk.md#logout) voor meer informatie.
{% endhint %}

***

### `status` - Licentiestatus controleren

Geef de huidige licentie- en authenticatiestatus weer.

**Syntaxis:**

```bash
chloros-cli status
```

**Voorbeeld:**

```bash
chloros-cli status
```

**Uitvoer:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Exportvoortgang controleren

Controleer de exportvoortgang van Thread 4 tijdens of na de verwerking.

**Syntaxis:**

```bash
chloros-cli export-status
```

**Voorbeeld:**

```bash
chloros-cli export-status
```

**Toepassing:** Roep deze opdracht aan terwijl de verwerking loopt om de voortgang van de export te controleren.***

### `language` - Beheer van de interfacetaal

Bekijk of wijzig de interfacetaal van CLI.

**Syntaxis:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Voorbeelden:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Ondersteunde talen (38 in totaal)

| Code    | Taal              | Oorspronkelijke naam      |
| ------- | --------------------- | ---------------- |
| `en`    | Engels               | English          |
| `es`    | Spaans               | Español          |
| `pt`    | Portugees            | Português        |
| `fr`    | Frans                | Français         |
| `de`    | Duits                | Deutsch          |
| `it`    | Italiaans               | Italiano         |
| `ja`    | Japans              | 日本語              |
| `ko`    | Koreaans                | 한국어              |
| `zh`    | Chinees (vereenvoudigd)  | 简体中文             |
| `zh-TW` | Chinees (traditioneel) | 繁體中文             |
| `ru`    | Russisch               | Русский          |
| `nl`    | Nederlands                 | Nederlands       |
| `ar`    | Arabisch                | العربية          |
| `pl`    | Pools                | Polski           |
| `tr`    | Turks               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesisch            | Bahasa Indonesia |
| `vi`    | Vietnamees            | Tiếng Việt       |
| `th`    | Thais                  | ไทย              |
| `sv`    | Zweeds               | Svenska          |
| `da`    | Deens                | Dansk            |
| `no`    | Noors             | Norsk            |
| `fi`    | Fins               | Suomi            |
| `el`    | Grieks                 | Ελληνικά         |
| `cs`    | Tsjechisch                | Čeština          |
| `hu`    | Hongaars             | Magyar           |
| `ro`    | Roemeens              | Română           |
| `uk`    | Oekraïens             | Українська       |
| `pt-BR` | Braziliaans Portugees  | Português Brasileiro |
| `zh-HK` | Kantonees             | 粵語             |
| `ms`    | Maleis                 | Bahasa Melayu    |
| `sk`    | Slowaaks                | Slovenčina       |
| `bg`    | Bulgaars             | Български        |
| `hr`    | Kroatisch              | Hrvatski         |
| `lt`    | Litouws            | Lietuvių         |
| `lv`    | Lets               | Latviešu         |
| `et`    | Ests              | Eesti            |
| `sl`    | Sloveens             | Slovenščina      |

{% hint style="success" %}
**Automatische persistentie**: Uw taalvoorkeur wordt opgeslagen in `~/.chloros/cli_language.json` en blijft behouden tijdens alle sessies.
{% endhint %}

***

### `set-project-folder` - Standaardprojectmap instellen

Wijzig de locatie van de standaardprojectmap (gedeeld met GUI op Windows).

**Syntaxis:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Voorbeelden:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Projectmap weergeven

Geef de huidige locatie van de standaardprojectmap weer.

**Syntaxis:**

```bash
chloros-cli get-project-folder
```

**Voorbeeld:**

```bash
chloros-cli get-project-folder
```

**Uitvoer:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Terugzetten naar standaard

De projectmap terugzetten naar de standaardlocatie.

**Syntaxis:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Systeemdiagnostiek uitvoeren

Voer 7 diagnostische controles uit om uw systeemconfiguratie te verifiëren.

**Syntaxis:**

```bash
chloros-cli selftest
```

**Uitgevoerde diagnostiek:**

1. Versiecontrole
2. Beschikbaarheid van poort (5000)
3. Opstarten van backend
4. API-connectiviteitstest
5. Systeeminformatie en GPU-detectie
6. Verificatie van denoiser-modellen
7. Controle op beschikbaarheid van CUDA

{% hint style="info" %}
**Handig voor probleemoplossing**: Voer `selftest` uit na de installatie om te controleren of uw systeem correct is geconfigureerd, met name op Linux/Jetson waar de GPU- en CUDA-instellingen mogelijk moeten worden geverifieerd.
{% endhint %}

***

### `update` - Controleren op updates (alleen Linux)

Controleer op en installeer CLI-updates op Linux-systemen.

**Syntaxis:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Optie    | Beschrijving                        |
| --------- | ---------------------------------- |
| `--check` | Alleen controleren op updates, niet installeren |

{% hint style="info" %}
Deze opdracht is alleen beschikbaar op Linux. Op Windows worden updates geleverd via het installatieprogramma.
{% endhint %}

***

## Algemene opties

Deze opties zijn van toepassing op alle commando&#x27;s:

| Optie            | Type    | Standaard       | Beschrijving                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Pad    | Automatisch gedetecteerd | Pad naar uitvoerbaar backend-bestand                       |
| `--port`          | Geheel getal | 5000          | Poortnummer van backend API                          |
| `--restart`       | Vlag    | -             | Backend geforceerd opnieuw opstarten (beëindigt bestaande processen) |
| `--version`       | Vlag    | -             | Versie-informatie weergeven en afsluiten                |
| `--help`          | Vlag    | -             | Helpinformatie weergeven en afsluiten                   |

{% hint style="info" %}
**Automatische detectie van backend**: Het pad `--backend-exe` wordt automatisch gedetecteerd per platform:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (handmatig)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Voorbeeld met algemene opties:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Handleiding voor verwerkingsinstellingen

### Parallelle verwerking en dynamische rekenaanpassing

Chloros 1.1.0 bevat [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) — de verwerkingsengine **detecteert automatisch uw hardware** en selecteert de optimale strategie:

| Platform | Strategie | Werkers | Pijplijn | Opmerkingen |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Geheugenefficiënt, geserialiseerd |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Gelijktijdige GPU-verwerking |
| **Desktop met 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Goede desktopprestaties |
| **Desktop met 12 GB+ GPU** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optimale desktopprestaties |
| **Systeem met alleen CPU** | `CPU_PARALLEL` | cores - 1 | `cpu_fallback` | Geen GPU vereist |

{% hint style="success" %}
**Geen handmatige configuratie nodig!** Chloros detecteert automatisch uw CPU, GPU, RAM en (op Jetson) thermische sensoren, en configureert vervolgens automatisch de optimale verwerkingspijplijn.
{% endhint %}

### Debayer-methoden

| Methode | CLI-vlag | Kwaliteit | Snelheid | Licentie |
| --- | --- | --- | --- | --- |
| **Standaard (Snel, Gemiddelde kwaliteit)** | `--debayer standard` | Goed | Snel | Gratis / Chloros+ |
| **Textuurbewust (Langzaam, Hoogste kwaliteit)** | `--debayer texture-aware` | Hoogste | Langzaam | Alleen Chloros+ |

De standaard debayer-methode is **Standaard**. De**Textuurbewuste** methode maakt gebruik van een AI/ML-ruisonderdrukkingsmodel voor de hoogste uitvoerkwaliteit, maar vereist een Chloros+ licentie en een NVIDIA GPU.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Vignettecorrectie

**Wat het doet:** Corrigeert lichtverval aan de randen van het beeld (donkere hoeken die vaak voorkomen in camerabeelden).

* **Standaard ingeschakeld** - De meeste gebruikers kunnen dit beter ingeschakeld laten
* Gebruik `--no-vignette` om uit te schakelen

{% hint style="success" %}
**Aanbeveling**: Schakel vignetcorrectie altijd in om een gelijkmatige helderheid over het hele beeld te garanderen.
{% endhint %}

### Reflectiekalibratie

Zet ruwe sensorwaarden om in gestandaardiseerde reflectiepercentages met behulp van kalibratiepanelen.

* **Standaard ingeschakeld** - Essentieel voor vegetatieanalyse
* Vereist kalibratiedoelpanelen in beelden
* Gebruik `--no-reflectance` om uit te schakelen

{% hint style="info" %}
**Vereisten**: Zorg ervoor dat kalibratiepanelen goed belicht en zichtbaar zijn in uw beelden voor een nauwkeurige reflectanceconversie.
{% endhint %}

### PPK-correcties

**Wat het doet:** Past Post-Processed Kinematic-correcties toe met behulp van DAQ-A-SD-loggegevens voor verbeterde GPS-nauwkeurigheid.

* **Standaard uitgeschakeld**
* Gebruik `--ppk` om in te schakelen
* Vereist .daq-bestanden in de projectmap van de MAPIR DAQ-A-SD-lichtsensor.

### Uitvoerformaten

<table><thead><tr><th width="197">Formaat</th><th width="130.20001220703125">Bitdiepte</th><th width="116.5999755859375">Bestandsgrootte</th><th>Meest geschikt voor</th></tr></thead><tbody><tr><td><strong>TIFF (16-bits)</strong> ⭐</td><td>16-bits geheel getal</td><td>Groot</td><td>GIS-analyse, fotogrammetrie (aanbevolen)</td></tr><tr><td><strong>TIFF (32-bit, Percent)</strong></td><td>32-bits drijvende komma</td><td>Zeer groot</td><td>Wetenschappelijke analyse, onderzoek</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bits geheel getal</td><td>Middel</td><td>Visuele inspectie, delen via het web</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bits geheel getal</td><td>Klein</td><td>Snel voorbeeld, gecomprimeerde uitvoer</td></tr></tbody></table>***

## Automatisering &amp; scripting

### PowerShell-batchverwerking (Windows)

Verwerk automatisch meerdere datasetsmappen op Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows Batchscript (Windows)

Eenvoudige lus voor batchverwerking op Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash-batchverwerking (Linux)

Verwerk meerdere datasetsmappen op Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python-automatiseringsscript (platformonafhankelijk)

Geavanceerde automatisering met foutafhandeling (werkt op Windows en Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Verwerkingsworkflow

### Standaardworkflow

1. **Invoer**: Map met RAW/JPG-beeldparen
2. **Detectie**: CLI scant automatisch naar ondersteunde beeldbestanden
3. **Verwerking**: Parallelle modus schaalt mee met uw CPU-kernen (Chloros+)
4. **Uitvoer**: Maakt submappen per cameramodel aan met verwerkte afbeeldingen

### Voorbeeld van de uitvoerstructuur

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Geschatte verwerkingstijden

Typische verwerkingstijden voor 100 afbeeldingen (elk 12 MP):

| Platform | Modus | Geschatte tijd | Opmerkingen |
| --- | --- | --- | --- |
| **Desktop 12 GB+ GPU** | `GPU_PARALLEL` | 5-10 min | Snelste optie |
| **Desktop 8 GB GPU** | `GPU_SINGLE` | 10-15 min | Goede prestaties |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15-25 min | Edge computing |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30-60 min | Beperkt geheugen |
| **Alleen CPU** | `CPU_PARALLEL` | 20-40 min | Geen GPU vereist |

{% hint style="info" %}
**Prestatietip**: De verwerkingstijd varieert afhankelijk van het aantal afbeeldingen, de resolutie, de debayer-methode en de hardware. Texture Aware debayer duurt aanzienlijk langer dan Standard. Zie [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) voor meer informatie.
{% endhint %}

***

## Probleemoplossing

### CLI niet gevonden

**Windows-fout:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Oplossingen:**

1. Controleer de installatielocatie:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Gebruik het volledige pad als het niet in PATH staat:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Voeg handmatig toe aan PATH:
   * Open Systeemeigenschappen → Omgevingsvariabelen
   * Bewerk de PATH-variabele
   * Voeg toe: `C:\Program Files\Chloros\resources\cli`
   * Start de terminal opnieuw op

**Linux Fout:**

```
chloros-cli: command not found
```

**Linux Oplossingen:**

1. Controleer de installatie:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Herlaad uw shell:

```bash
source ~/.bashrc
```

3. Controleer de machtigingen:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend kan niet worden gestart**Fout:**

```

Backend failed to start within 30 seconds
```

**Oplossingen:**

1. Controleer of de backend al actief is (sluit deze eerst)
2. Controleer of de firewall niet blokkeert (Windows) of controleer de beschikbaarheid van de poort (Linux: `lsof -i :5000`)
3. Probeer een andere poort:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Forceer een herstart van de backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Controleer op Linux of het uitvoerbare bestand van de backend bestaat:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Licentie-/authenticatieproblemen**Fout:**

```

Chloros+ license required for CLI access
```

**Oplossingen:**

1. Controleer of u een actief Chloros+-abonnement hebt
2. Log in met uw inloggegevens:

```bash
chloros-cli login user@example.com 'password'
```

3. Controleer de licentiestatus:

```bash
chloros-cli status
```

4. Neem contact op met de ondersteuning: info@mapir.camera

***

### Geen afbeeldingen gevonden**Fout:**

```

No images found in the specified folder
```

**Oplossingen:**

1. Controleer of de map ondersteunde formaten bevat (.RAW, .TIF, .JPG)
2. Controleer of het pad naar de map correct is (gebruik aanhalingstekens voor paden met spaties)
3. Zorg ervoor dat u leesrechten voor de map hebt
4. Controleer of de bestandsextensies correct zijn

***

### Verwerking loopt vast of hangt**Oplossingen:**

1. Controleer de beschikbare schijfruimte (zorg dat er voldoende is voor de uitvoer)
2. Sluit andere toepassingen om geheugen vrij te maken
3. Verminder het aantal afbeeldingen (verwerk in batches)

***

### Poort al in gebruik**Fout:**

```

Port 5000 is already in use
```

**Oplossingen:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Veelgestelde vragen

### V: Heb ik een licentie nodig voor de CLI?

**A:**Ja! Voor de CLI is een betaalde**Chloros+ licentie** vereist.

* ❌ Standaard (gratis) abonnement: CLI uitgeschakeld
* ✅ Chloros+ (betaalde) abonnementen: CLI volledig ingeschakeld

Abonneren via: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### V: Kan ik de CLI gebruiken op een server zonder GUI?**A:** Ja! De CLI draait volledig headless. Dit is het belangrijkste gebruiksscenario op Linux.**Windows-server:**
* Windows-server 2016 of later
* Visual C++ Redistributable geïnstalleerd

**Linux-server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) of JetPack 6 (arm64)
* Installeren via het `.deb`-pakket

**Beide platforms:**
* Minimaal 8 GB RAM (16 GB aanbevolen)
* Eenmalige licentieactivering: `chloros-cli login user@example.com 'password'`

***

### V: Waar worden bewerkte afbeeldingen opgeslagen?**A:**Standaard worden bewerkte afbeeldingen opgeslagen in**dezelfde map als de invoer**, in submappen met het cameramodel (bijv. `Survey3N_RGN/`).

Gebruik de optie `-o` om een andere uitvoermap op te geven:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### V: Kan ik meerdere mappen tegelijk verwerken?**A:** Niet direct in één opdracht, maar u kunt scripts gebruiken om mappen achtereenvolgens te verwerken. Zie het gedeelte [Automatisering en scripting](CLI.md#automation--scripting).***

### V: Hoe sla ik de uitvoer van CLI op in een logbestand?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### V: Wat gebeurt er als ik tijdens de verwerking op Ctrl+C druk?**A:** De CLI zal:

1. De verwerking op een nette manier stoppen
2. De backend afsluiten
3. Afsluiten met code 130

Gedeeltelijk verwerkte afbeeldingen kunnen in de uitvoermap achterblijven.

***

### V: Kan ik de verwerking van CLI automatiseren?**A:** Absoluut! De CLI is ontworpen voor automatisering. Zie [Automatisering &amp; scripting](CLI.md#automation--scripting) voor PowerShell (Windows), Batch (Windows), Bash (Linux) en Python (platformonafhankelijk).***

### V: Hoe controleer ik de CLI-versie?**A:**

```bash
chloros-cli --version
```

**Uitvoer:**

```

Chloros CLI 1.1.0
```

***

## Hulp krijgen

### Help via de opdrachtregel

Bekijk helpinformatie rechtstreeks in de CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Ondersteuningskanalen

* **E-mail**: info@mapir.camera
* **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Prijzen**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Volledige voorbeelden

### Voorbeeld 1: Basisverwerking

Verwerking met standaardinstellingen (vignettering, reflectie):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Voorbeeld 2: Wetenschappelijke uitvoer van hoge kwaliteit

32-bits float TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Voorbeeld 3: Snelle verwerking van voorbeelden

8-bits PNG zonder kalibratie voor snelle beoordeling:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Voorbeeld 4: PPK-gecorrigeerde verwerking

Pas PPK-correcties toe met reflectantie:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Voorbeeld 5: Aangepaste uitvoerlocatie

Verwerk naar een andere locatie met een specifiek formaat:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Voorbeeld 6: Authenticatiewerkstroom

Volledige authenticatiestroom (hetzelfde op alle platforms):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Voorbeeld 7: Meertalig gebruik

Interfacetaal wijzigen (hetzelfde op alle platforms):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
