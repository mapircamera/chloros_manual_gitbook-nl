# API : Python SDK

De **Chloros Python SDK** biedt programmatische toegang tot de Chloros beeldverwerkingsengine, waardoor automatisering, aangepaste workflows en naadloze integratie met uw Python-toepassingen en onderzoekspijplijnen mogelijk worden.

### Belangrijkste kenmerken

* 🐍 **Native Python** - Schone, Pythonic API voor beeldverwerking
* 🔧 **Volledige API-toegang** - Volledige controle over Chloros-verwerking
* 🚀 **Automatisering** - Bouw aangepaste batchverwerkingsworkflows
* 🔗 **Integratie** - Chloros integreren in bestaande Python-toepassingen
* 📊 **Klaar voor onderzoek** - Perfect voor wetenschappelijke analysepijplijnen
* ⚡ **Parallelle verwerking** - Schaalbaar naar uw CPU-kernen (Chloros+)

### Vereisten

| Vereiste          | Details                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Moet lokaal zijn geïnstalleerd                                           |
| **Licentie**          | Chloros+ ([betaald abonnement vereist](https://cloud.mapir.camera/pricing)) |
| **Besturingssysteem** | Windows 10/11 (64-bits)                                              |
| **Python**           | Python 3.7 of hoger                                                |
| **Geheugen**           | minimaal 8 GB RAM (16 GB aanbevolen)                                  |
| **Internet**         | vereist voor licentieactivering                                     |

{% hint style=&quot;warning&quot; %}
**Licentievereiste**: Voor Python SDK is een betaald Chloros+-abonnement vereist voor toegang tot API. Standaard (gratis) abonnementen hebben geen toegang tot API/SDK. Ga naar [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) om te upgraden.
{% endhint %}

## Snel aan de slag

### Installatie

Installeer via pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Eerste installatie**: Voordat u de SDK gebruikt, moet u uw Chloros+-licentie activeren door Chloros, Chloros (browser) of Chloros CLI te openen en in te loggen met uw inloggegevens. Dit hoeft slechts eenmaal te worden gedaan.
{% endhint %}

### Basisgebruik

Verwerk een map met slechts een paar regels:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Volledige controle

Voor geavanceerde workflows:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Installatiehandleiding

### Vereisten

Voordat u SDK installeert, moet u ervoor zorgen dat u beschikt over:

1. **Chloros Desktop** geïnstalleerd ([download](download.md))
2. **Python 3.7+** geïnstalleerd ([python.org](https://www.python.org))
3. **Actieve Chloros+-licentie** ([upgrade](https://cloud.mapir.camera/pricing))

### Installeren via pip

**Standaardinstallatie:**

```bash
pip install chloros-sdk
```

**Met ondersteuning voor voortgangsbewaking:**

```bash
pip install chloros-sdk[progress]
```

**Ontwikkelingsinstallatie:**

```bash
pip install chloros-sdk[dev]
```

### Installatie controleren

Controleer of SDK correct is geïnstalleerd:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Eerste installatie

### Licentie activeren

De SDK gebruikt dezelfde licentie als Chloros, Chloros (browser) en Chloros CLI. Activeer eenmaal via de GUI of CLI:

1. Open **Chloros of Chloros (browser)** en log in op het tabblad Gebruiker <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Of open **CLI**.
2. Voer uw Chloros+-inloggegevens in en log in
3. De licentie wordt lokaal in de cache opgeslagen (blijft behouden na herstarten)

{% hint style=&quot;success&quot; %}
**Eenmalige installatie**: Nadat u zich hebt aangemeld via de GUI of CLI, gebruikt SDK automatisch de opgeslagen licentie. Er is geen aanvullende authenticatie nodig!
{% endhint %}

### Verbinding testen

Controleer of de SDK verbinding kan maken met Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API Referentie

### ChlorosLocal-klasse

Hoofdklasse voor lokale Chloros-beeldverwerking.

#### Constructor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parameters:**

| Parameter                 | Type | Standaard                   | Beschrijving                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL van lokale Chloros-backend          |
| `auto_start_backend`      | bool | `True`                    | Start backend automatisch indien nodig |
| `backend_exe`             | str  | `None` (automatische detectie)      | Pad naar uitvoerbaar backend-bestand            |
| `timeout`                 | int  | `30`                      | Time-out voor verzoek in seconden            |
| `backend_startup_timeout` | int  | `60`                      | Time-out voor opstarten backend (seconden) |

**Voorbeelden:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Methoden

#### `create_project(project_name, camera=None)`

Maak een nieuw Chloros-project aan.

**Parameters:**

| Parameter      | Type | Vereist | Beschrijving                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ja      | Naam voor het project                                     |
| `camera`       | str  | Nee       | Camerasjabloon (bijv. &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Retourneert:** `dict` - Reactie op het aanmaken van een project

**Voorbeeld:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Afbeeldingen importeren vanuit een map.

**Parameters:**

| Parameter     | Type     | Vereist | Beschrijving                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Pad | Ja      | Pad naar map met afbeeldingen         |
| `recursive`   | bool     | Nee       | Submappen doorzoeken (standaard: False) |

**Retourneert:** `dict` - Importeren resultaten met aantal bestanden

**Voorbeeld:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Configureer verwerkingsinstellingen.

**Parameters:**

| Parameter                 | Type | Standaard                 | Beschrijving                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Hoge kwaliteit (sneller)&quot; | Debayer-methode                  |
| `vignette_correction`     | bool | `True`                  | Vignettecorrectie inschakelen      |
| `reflectance_calibration` | bool | `True`                  | Reflectiekalibratie inschakelen  |
| `indices`                 | lijst | `None`                  | Te berekenen vegetatie-indexen |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Uitvoerformaat                   |
| `ppk`                     | bool | `False`                 | PPK-correcties inschakelen          |
| `custom_settings`         | dict | `None`                  | Geavanceerde aangepaste instellingen        |

**Exportformaten:**

* `"TIFF (16-bit)"` - Aanbevolen voor GIS/fotogrammetrie
* `"TIFF (32-bit, Percent)"` - Wetenschappelijke analyse
* `"PNG (8-bit)"` - Visuele inspectie
* `"JPG (8-bit)"` - Gecomprimeerde uitvoer

**Beschikbare indexen:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2, en meer.

**Voorbeeld:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Verwerk de projectafbeeldingen.

**Parameters:**

| Parameter           | Type     | Standaard      | Beschrijving                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Verwerkingsmodus: &quot;parallel&quot; of &quot;serieel&quot;   |
| `wait`              | bool     | `True`       | Wachten op voltooiing                       |
| `progress_callback` | callable | `None`       | Callback-functie voor voortgang (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Polling-interval voor voortgang (seconden)   |

**Retourneert:** `dict` - Verwerkingsresultaten

{% hint style=&quot;warning&quot; %}
**Parallelle modus**: vereist Chloros+ licentie. Schaalbaar naar uw CPU-kernen (maximaal 16 workers).
{% endhint %}

**Voorbeeld:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Haal de huidige projectconfiguratie op.

**Retourneert:** `dict` - Huidige projectconfiguratie

**Voorbeeld:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Haal backend-statusinformatie op.

**Retourneert:** `dict` - Backend-status

**Voorbeeld:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Sluit de backend af (indien gestart door SDK).

**Voorbeeld:**

```python
chloros.shutdown_backend()
```

***

### Handige functies

#### `process_folder(folder_path, **options)`

Handige functie van één regel om een map te verwerken.

**Parameters:**

| Parameter                 | Type     | Standaard         | Beschrijving                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Pad | Vereist        | Pad naar map met afbeeldingen     |
| `project_name`            | str      | Automatisch gegenereerd  | Projectnaam                   |
| `camera`                  | str      | `None`          | Camerasjabloon                |
| `indices`                 | lijst     | `["NDVI"]`      | Indices om te berekenen           |
| `vignette_correction`     | bool     | `True`          | Vignettecorrectie inschakelen     |
| `reflectance_calibration` | bool     | `True`          | Reflectiekalibratie inschakelen |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Uitvoerformaat                  |
| `mode`                    | str      | `"parallel"`    | Verwerkingsmodus                |
| `progress_callback`       | callable | `None`          | Voortgangs-callback              |

**Retourneert:** `dict` - Verwerkingsresultaten

**Voorbeeld:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Ondersteuning voor contextmanager

De SDK ondersteunt contextmanagers voor automatische opschoning:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Volledige voorbeelden

### Voorbeeld 1: Basisverwerking

Een map verwerken met standaardinstellingen:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Voorbeeld 2: aangepaste workflow

Volledige controle over de verwerkingspijplijn:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Voorbeeld 3: batchverwerking van meerdere mappen

Verwerk meerdere vluchtgegevenssets:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Voorbeeld 4: integratie van onderzoekspijplijn

Chloros integreren met gegevensanalyse:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Voorbeeld 5: Aangepaste voortgangsbewaking

Geavanceerde voortgangsbewaking met logboekregistratie:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Voorbeeld 6: Foutverwerking

Robuuste foutafhandeling voor productiegebruik:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Voorbeeld 7: opdrachtregelprogramma

Bouw een aangepast CLI-programma met SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Gebruik:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Uitzonderingsafhandeling

De SDK biedt specifieke uitzonderingsklassen voor verschillende fouttypes:

### Uitzonderingshiërarchie

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Voorbeelden van uitzonderingen

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Geavanceerde onderwerpen

### Aangepaste backendconfiguratie

Gebruik een aangepaste backendlocatie of -configuratie:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Niet-blokkerende verwerking

Start de verwerking en ga door met andere taken:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Geheugenbeheer

Verwerk grote datasets in batches:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Probleemoplossing

### Backend start niet

**Probleem:** SDK kan de backend niet starten.

**Oplossingen:**

1. Controleer of Chloros Desktop is geïnstalleerd:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Controleer of Windows Firewall niet blokkeert
3. Probeer het handmatige backend-pad:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licentie niet gedetecteerd

**Probleem:** SDK waarschuwt voor ontbrekende licentie

**Oplossingen:**

1. Open Chloros, Chloros (browser) of Chloros CLI en log in.
2. Controleer of de licentie in de cache is opgeslagen:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Neem contact op met de ondersteuning: info@mapir.camera

***

### Importfouten

**Probleem:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Oplossingen:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Time-out bij verwerking

**Probleem:** Time-out bij verwerking

**Oplossingen:**

1. Verleng de time-out:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Verwerk kleinere batches
3. Controleer de beschikbare schijfruimte
4. Controleer de systeembronnen

***

### Poort al in gebruik

**Probleem:** Backend-poort 5000 bezet

**Oplossingen:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Of zoek en sluit het conflicterende proces:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Tips voor betere prestaties

### Verwerkingssnelheid optimaliseren

1. **Gebruik parallelle modus** (vereist Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Verminder de uitvoerresolutie** (indien acceptabel)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Schakel onnodige indexen uit**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Verwerk op SSD** (niet op HDD)

***

### Geheugenoptimalisatie

Voor grote datasets:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Achtergrondverwerking

Maak Python vrij voor andere taken:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integratievoorbeelden

### Django-integratie

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Veelgestelde vragen

### V: Is er een internetverbinding nodig voor SDK?

**A:** Alleen voor de eerste activering van de licentie. Nadat u bent ingelogd via Chloros, Chloros (browser) of Chloros CLI, wordt de licentie lokaal opgeslagen en werkt deze 30 dagen offline.

***

### V: Kan ik SDK gebruiken op een server zonder GUI?

**A:** Ja! Vereisten:

* Windows Server 2016 of hoger
* Chloros geïnstalleerd (eenmalig)
* Licentie geactiveerd op een willekeurige machine (in cache opgeslagen licentie gekopieerd naar server)

***

### V: Wat is het verschil tussen Desktop, CLI en SDK?

| Functie         | Desktop GUI | CLI Opdrachtregel | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interface**   | Point-click | Command          | Python API  |
| **Meest geschikt voor**    | Visueel werk | Scripting        | Integratie |
| **Automatisering**  | Beperkt     | Goed             | Uitstekend   |
| **Flexibiliteit** | Basis       | Goed             | Maximaal     |
| **Licentie**     | Chloros+    | Chloros+         | Chloros+    |

***

### V: Kan ik apps distribueren die zijn gebouwd met de SDK?

**A:** SDK-code kan worden geïntegreerd in uw applicaties, maar:

* Eindgebruikers moeten Chloros geïnstalleerd hebben
* Eindgebruikers moeten actieve Chloros+-licenties hebben
* Voor commerciële distributie is een OEM-licentie vereist.

Neem contact op met info@mapir.camera voor vragen over OEM.

***

### V: Hoe werk ik de SDK bij?

```bash
pip install --upgrade chloros-sdk
```

***

### V: Waar worden bewerkte afbeeldingen opgeslagen?

Standaard in het projectpad:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### V: Kan ik afbeeldingen verwerken vanuit Python-scripts die volgens een schema worden uitgevoerd?

**A:** Ja! Gebruik Windows Taakplanner met Python-scripts:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Plan via Taakplanner om dagelijks uit te voeren.

***

### V: Ondersteunt SDK async/await?

**A:** De huidige versie is synchroon. Gebruik voor asynchroon gedrag `wait=False` of voer het uit in een aparte thread:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Hulp krijgen

### Documentatie

* **API-referentie**: deze pagina

### Ondersteuningskanalen

* **E-mail**: info@mapir.camera
* **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Prijzen**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Voorbeeldcode

Alle hier vermelde voorbeelden zijn getest en klaar voor productie. Kopieer ze en pas ze aan voor uw gebruikssituatie.

***

## Licentie

**Eigendomsrechtelijke software** - Copyright (c) 2025 MAPIR Inc.

SDK vereist een actief Chloros+-abonnement. Ongeautoriseerd gebruik, distributie of wijziging is verboden.
