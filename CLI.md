# CLI : Opdrachtregel

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

De **Chloros CLI** biedt krachtige opdrachtregeltoegang tot de Chloros-beeldverwerkingsengine, waardoor automatisering, scripting en headless-werking voor uw beeldverwerkingsworkflows mogelijk wordt.

### Belangrijkste kenmerken

* 🚀 **Automatisering** - Script batchverwerking van meerdere datasets
* 🔗 **Integratie** - Integreer in bestaande workflows en pijplijnen
* 💻 **Headless-werking** - Draait zonder GUI
* 🌍 **Meertalig** - Ondersteuning voor 38 talen
* ⚡ **Parallelle verwerking** - Schaalbaar naar uw CPU (tot 16 parallelle workers)

### Vereisten

| Vereiste          | Details                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Besturingssysteem** | Windows 10/11 (64-bit)                                              |
| **Licentie**          | Chloros+ ([betaald abonnement vereist](https://cloud.mapir.camera/pricing)) |
| **Geheugen**           | Minimaal 8 GB RAM (16 GB aanbevolen)                                  |
| **Internet**         | Vereist voor licentieactivering                                     |
| **Schijfruimte**       | Varieert per projectgrootte                                              |

{% hint style=&quot;warning&quot; %}
**Licentievereiste**: Voor CLI is een betaald Chloros+-abonnement vereist. Standaard (gratis) abonnementen hebben geen toegang tot CLI. Ga naar [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) om te upgraden.
{% endhint %}

## Snel aan de slag

### Installatie

De CLI wordt automatisch meegeleverd met het Chloros-installatieprogramma:

1. Download en voer **Chloros Installer.exe** uit.
2. Voltooi de installatiewizard.
3. CLI geïnstalleerd in: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Het installatieprogramma voegt automatisch `chloros-cli` toe aan uw systeem PATH. Start uw terminal opnieuw op na de installatie.
{% endhint %}

### Eerste installatie

Activeer uw Chloros+-licentie voordat u CLI gebruikt:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Basisgebruik

Verwerk een map met standaardinstellingen:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Commando-referentie

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

**Voorbeeld:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opties voor verwerkingsopdrachten

| Optie                | Type    | Standaard        | Beschrijving                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Pad    | _Vereist_     | Map met RAW/JPG multispectrale afbeeldingen                                         |
| `-o, --output`        | Pad    | Hetzelfde als invoer  | Uitvoermap voor verwerkte afbeeldingen                                                     |
| `-n, --project-name`  | Tekenreeks  | Automatisch gegenereerd | Aangepaste projectnaam                                                                    |
| `--vignette`          | Vlag    | Ingeschakeld        | Vignettecorrectie inschakelen                                                             |
| `--no-vignette`       | Vlag    | -              | Vignettecorrectie uitschakelen                                                            |
| `--reflectance`       | Vlag    | Ingeschakeld        | Reflectiekalibratie inschakelen                                                         |
| `--no-reflectance`    | Vlag    | -              | Reflectiekalibratie uitschakelen                                                        |
| `--ppk`               | Vlag    | Uitgeschakeld       | PPK-correcties toepassen op basis van .daq-lichtsensorgegevens                                      |
| `--format`            | Keuze  | TIFF (16-bits)  | Uitvoerformaat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Geheel getal | Auto           | Minimale doelgrootte in pixels voor detectie van kalibratiepaneel                          |
| `--target-clustering` | Geheel getal | Auto           | Drempelwaarde voor doelclustering (0-100)                                                    |
| `--exposure-pin-1`    | String  | Geen           | Belichting vergrendelen voor cameramodel (pin 1)                                                 |
| `--exposure-pin-2`    | String  | Geen           | Belichting vergrendelen voor cameramodel (pin 2)                                                 |
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

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Speciale tekens**: Gebruik enkele aanhalingstekens rond wachtwoorden die tekens zoals `$`, `!` of spaties bevatten.
{% endhint %}

**Uitvoer:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Inloggegevens wissen

Wis opgeslagen inloggegevens en log uit bij uw account.

**Syntaxis:**

```bash
chloros-cli logout
```

**Voorbeeld:**

```powershell
chloros-cli logout
```

**Uitvoer:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Licentiestatus controleren

Geef de huidige licentie- en authenticatiestatus weer.

**Syntaxis:**

```bash
chloros-cli status
```

**Voorbeeld:**

```powershell
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

Controleer de exportvoortgang van thread 4 tijdens of na de verwerking.

**Syntaxis:**

```bash
chloros-cli export-status
```

**Voorbeeld:**

```powershell
chloros-cli export-status
```

**Gebruiksscenario:** Roep deze opdracht aan terwijl de verwerking wordt uitgevoerd om de voortgang van de export te controleren.

***

### `language` - Taal van de interface beheren

Bekijk of wijzig de CLI-interfacetaal.

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

```powershell
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

| Code    | Taal              | Native naam      |
| ------- | --------------------- | ---------------- |
| `en`    | Engels               | Engels          |
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
| `cs`    | Tsjechisch                 | Čeština          |
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

{% hint style=&quot;success&quot; %}
**Automatische persistentie**: Uw taalvoorkeur wordt opgeslagen in `~/.chloros/cli_language.json` en blijft behouden tijdens alle sessies.
{% endhint %}

***

### `set-project-folder` - Standaardprojectmap instellen

Wijzig de locatie van de standaardprojectmap (gedeeld met GUI).

**Syntaxis:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Voorbeeld:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Projectmap weergeven

Geef de huidige standaardlocatie van de projectmap weer.

**Syntaxis:**

```bash
chloros-cli get-project-folder
```

**Voorbeeld:**

```powershell
chloros-cli get-project-folder
```

**Uitvoer:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Terugzetten naar standaard

Zet de projectmap terug naar de standaardlocatie.

**Syntaxis:**

```bash
chloros-cli reset-project-folder
```

***

## Algemene opties

Deze opties zijn van toepassing op alle opdrachten:

| Optie          | Type    | Standaard       | Beschrijving                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Pad    | Automatisch gedetecteerd | Pad naar uitvoerbaar bestand backend                       |
| `--port`        | Geheel getal | 5000          | Backend API poortnummer                          |
| `--restart`     | Vlag    | -             | Backend geforceerd opnieuw opstarten (bestaande processen beëindigen) |
| `--version`     | Vlag    | -             | Versie-informatie weergeven en afsluiten                |
| `--help`        | Vlag    | -             | Help-informatie weergeven en afsluiten                   |

**Voorbeeld met algemene opties:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Handleiding voor verwerkingsinstellingen

### Parallelle verwerking

Chloros+ CLI **schaalt automatisch** parallelle verwerking om aan te sluiten bij de mogelijkheden van uw computer:

**Hoe het werkt:**

* Detecteert uw CPU-kernen en RAM
* Wijst werknemers toe: **2× CPU-kernen** (maakt gebruik van hyperthreading)
* **Maximaal: 16 parallelle werknemers** (voor stabiliteit)

**Systeemniveaus:**

| Systeemtype   | CPU        | RAM      | Werknemers  | Prestaties     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-end**  | 16+ kernen  | 32+ GB   | Tot 16 | Maximale snelheid   |
| **Middenklasse** | 8-15 cores | 16-31 GB | 8-16     | Uitstekende snelheid |
| **Low-end**   | 4-7 cores  | 8-15 GB  | 4-8      | Goede snelheid      |

{% hint style=&quot;success&quot; %}
**Automatische optimalisatie**: De CLI detecteert automatisch uw systeemspecificaties en configureert optimale parallelle verwerking. Handmatige configuratie is niet nodig!
{% endhint %}

### Debayer-methoden

De CLI gebruikt **Hoge kwaliteit (sneller)** als standaard en aanbevolen debayer-algoritme:

| Methode                      | Kwaliteit | Snelheid | Beschrijving                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Hoge kwaliteit (sneller)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Randbewust algoritme (standaard, aanbevolen) |

### Vignettecorrectie

**Wat het doet:** Corrigeert lichtverlies aan de randen van het beeld (donkere hoeken die vaak voorkomen in camerabeelden).

* **Standaard ingeschakeld** - De meeste gebruikers kunnen dit beter ingeschakeld laten.
* Gebruik `--no-vignette` om uit te schakelen.

{% hint style=&quot;success&quot; %}
**Aanbeveling**: Schakel vignettecorrectie altijd in om een gelijkmatige helderheid over het hele beeld te garanderen.
{% endhint %}

### Reflectiekalibratie

Converteert ruwe sensorwaarden naar gestandaardiseerde reflectiepercentages met behulp van kalibratiepanelen.

* **Standaard ingeschakeld** - Essentieel voor vegetatieanalyse.
* Vereist kalibratiedoelpanelen in afbeeldingen.
* Gebruik `--no-reflectance` om uit te schakelen.

{% hint style=&quot;info&quot; %}
**Vereisten**: Zorg ervoor dat kalibratiepanelen goed belicht en zichtbaar zijn in uw afbeeldingen voor een nauwkeurige reflectieconversie.
{% endhint %}

### PPK-correcties

**Wat het doet:** Past post-processed kinematische correcties toe met behulp van DAQ-A-SD-loggegevens voor verbeterde GPS-nauwkeurigheid.

* **Standaard uitgeschakeld**
* Gebruik `--ppk` om in te schakelen
* Vereist .daq-bestanden in de projectmap van MAPIR DAQ-A-SD-lichtsensor.

### Uitvoerformaten

<table><thead><tr><th width="197">Formaat</th><th width="130.20001220703125">Bitdiepte</th><th width="116.5999755859375">Bestandsgrootte</th><th>Meest geschikt voor</th></tr></thead><tbody><tr><td><strong>TIFF (16-bits)</strong> ⭐</td><td>16-bits geheel getal</td><td>Groot</td><td>GIS-analyse, fotogrammetrie (aanbevolen)</td></tr><tr><td><strong>TIFF (32-bits, procent)</strong></td><td>32-bits float</td><td>Zeer groot</td><td>Wetenschappelijke analyse, onderzoek</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bits geheel getal</td><td>Gemiddeld</td><td>Visuele inspectie, delen via internet</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bits geheel getal</td><td>Klein</td><td>Snelle preview, gecomprimeerde uitvoer</td></tr></tbody></table>***

## Automatisering en scripting

### PowerShell-batchverwerking

Meerdere datasets automatisch verwerken:

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

### Windows-batchscript

Eenvoudige lus voor batchverwerking:

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

### Python Automatiseringsscript

Geavanceerde automatisering met foutafhandeling:

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

1. **Invoer**: map met RAW/JPG-afbeeldingsparen
2. **Detectie**: CLI scant automatisch naar ondersteunde afbeeldingsbestanden
3. **Verwerking**: Parallelle modus schaalt naar uw CPU-kernen (Chloros+)
4. **Uitvoer**: Maakt submappen voor cameramodellen met verwerkte afbeeldingen

### Voorbeeld van uitvoerstructuur

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

### Geschatte verwerkingstijd

Typische verwerkingstijden voor 100 afbeeldingen (elk 12 MP):

| Modus              | Tijd      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Parallelle modus** | 5-10 min  | i7/Ryzen 7, 16 GB RAM, SSD (maximaal 16 workers) |
| **Parallelle modus** | 10-15 min | i5/Ryzen 5, 8 GB RAM, HDD (maximaal 8 workers)   |

{% hint style=&quot;info&quot; %}
**Prestatietip**: De verwerkingstijd varieert afhankelijk van het aantal afbeeldingen, de resolutie en de specificaties van de computer.
{% endhint %}

***

## Probleemoplossing

### CLI niet gevonden

**Fout:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Oplossingen:**

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
   * Bewerk de variabele PATH
   * Voeg toe: `C:\Program Files\Chloros\resources\cli`
   * Start de terminal opnieuw op

***

### Backend kan niet worden gestart

**Fout:**

```
Backend failed to start within 30 seconds
```

**Oplossingen:**

1. Controleer of de backend al actief is (sluit deze eerst)
2. Controleer of Windows Firewall niet blokkeert
3. Probeer een andere poort:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Forceer het opnieuw opstarten van de backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Licentie-/authenticatieproblemen

**Fout:**

```
Chloros+ license required for CLI access
```

**Oplossingen:**

1. Controleer of u een actief Chloros+-abonnement hebt.
2. Log in met uw inloggegevens:

```powershell
chloros-cli login user@example.com 'password'
```

3. Controleer de licentiestatus:

```powershell
chloros-cli status
```

4. Neem contact op met de ondersteuning: info@mapir.camera

***

### Geen afbeeldingen gevonden

**Fout:**

```
No images found in the specified folder
```

**Oplossingen:**

1. Controleer of de map ondersteunde indelingen bevat (.RAW, .TIF, .JPG)
2. Controleer of het pad naar de map correct is (gebruik aanhalingstekens voor paden met spaties)
3. Zorg ervoor dat u leesrechten hebt voor de map.
4. Controleer of de bestandsextensies correct zijn.

***

### Verwerking loopt vast of hangt op

**Oplossingen:**

1. Controleer de beschikbare schijfruimte (zorg ervoor dat er voldoende ruimte is voor de uitvoer).
2. Sluit andere toepassingen om geheugen vrij te maken.
3. Verminder het aantal afbeeldingen (verwerk in batches).

***

### Poort al in gebruik

**Fout:**

```
Port 5000 is already in use
```

**Oplossing:**

Geef een andere poort op:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Veelgestelde vragen

### V: Heb ik een licentie nodig voor de CLI?

**A:** Ja! Voor de CLI is een betaalde **Chloros+-licentie** vereist.

* ❌ Standaard (gratis) abonnement: CLI uitgeschakeld
* ✅ Chloros+ (betaalde) abonnementen: CLI volledig ingeschakeld

Abonneren op: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### V: Kan ik de CLI gebruiken op een server zonder GUI?

**A:** Ja! De CLI werkt volledig headless. Vereisten:

* Windows Server 2016 of hoger
* Visual C++ Redistributable geïnstalleerd
* Voldoende RAM (minimaal 8 GB, 16 GB aanbevolen)
* Eenmalige activering van GUI-licentie op elke machine

***

### V: Waar worden verwerkte afbeeldingen opgeslagen?

**A:** Standaard worden verwerkte afbeeldingen opgeslagen in **dezelfde map als de invoer** in submappen van het cameramodel (bijv. `Survey3N_RGN/`).

Gebruik de optie `-o` om een andere uitvoermap op te geven:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### V: Kan ik meerdere mappen tegelijk verwerken?

**A:** Niet rechtstreeks in één opdracht, maar u kunt scripts gebruiken om mappen achtereenvolgens te verwerken. Zie het gedeelte [Automatisering en scripting](CLI.md#automation--scripting).

***

### V: Hoe sla ik CLI-uitvoer op in een logbestand?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### V: Wat gebeurt er als ik tijdens de verwerking op Ctrl+C druk?

**A:** De CLI zal:

1. De verwerking netjes stoppen
2. De backend afsluiten
3. Afsluiten met code 130

Gedeeltelijk verwerkte afbeeldingen kunnen in de uitvoermap achterblijven.

***

### V: Kan ik de verwerking van CLI automatiseren?

**A:** Absoluut! De CLI is ontworpen voor automatisering. Zie [Automatisering en scripting](CLI.md#automation--scripting) voor voorbeelden van PowerShell, Batch en Python.

***

### V: Hoe controleer ik de CLI-versie?

**A:**

```powershell
chloros-cli --version
```

**Uitvoer:**

```
Chloros CLI 1.0.2
```

***

## Help krijgen

### Help bij de opdrachtregel

Bekijk helpinformatie rechtstreeks in CLI:

```powershell
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
* **Prijzen**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Volledige voorbeelden

### Voorbeeld 1: Basisverwerking

Verwerking met standaardinstellingen (vignettering, reflectie):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Voorbeeld 2: Wetenschappelijke output van hoge kwaliteit

32-bits float TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Voorbeeld 3: Snelle previewverwerking

8-bits PNG zonder kalibratie voor snelle beoordeling:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Voorbeeld 4: PPK-gecorrigeerde verwerking

PPK-correcties toepassen met reflectie:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Voorbeeld 5: Aangepaste uitvoerlocatie

Verwerken naar een andere schijf met een specifiek formaat:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Voorbeeld 6: authenticatieworkflow

Volledige authenticatiestroom voltooien:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Voorbeeld 7: gebruik van meerdere talen

Interfacetaal wijzigen:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
