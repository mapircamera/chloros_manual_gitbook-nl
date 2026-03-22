---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Downloaden

Download de nieuwste versie van Chloros om aan de slag te gaan met multispectrale beeldverwerking.

### Systeemvereisten

#### Windows

| Vereiste          | Minimaal                                              | Aanbevolen                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Besturingssysteem** | Windows 10 (64-bits)                                  | Windows 11 (64-bits)                                  |
| **Processor**        | Intel Core i5 of gelijkwaardig                          | Intel Core i7 of beter                              |
| **Geheugen (RAM)**     | 8 GB                                                  | 16 GB of meer                                         |
| **Grafische kaart**    | Compatibel met DirectX 11                                | NVIDIA GPU met 4 GB+ VRAM                            |
| **Opslagruimte**          | 6 GB vrije ruimte                                       | SSD met 10 GB+ vrije ruimte                            |
| **Beeldscherm**          | 1920x1080                                            | 2560x1440 of hoger                                  |
| **Internet**         | Vereist voor \[optioneel] Chloros+ licentieactivering | Vereist voor \[optioneel] Chloros+ licentieactivering |

#### Linux amd64 (x86\_64)

| Vereiste       | Minimaal                    | Aanbevolen               |
| ----------------- | -------------------------- | ------------------------- |
| **Distributie**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Processor**     | x86\_64 (Intel/AMD)        | Intel Core i7 of beter   |
| **Geheugen (RAM)**  | 8 GB                        | 16 GB of meer              |
| **Grafische kaart** | Geen (CPU-verwerking)      | NVIDIA GPU met 4 GB+ VRAM |
| **Opslagruimte**       | 2 GB vrije ruimte             | SSD met 10 GB+ vrije ruimte       |
| **Python**        | Python 3.7+ (voor SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Vereiste      | Minimaal                      | Aanbevolen                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson met JetPack 6 | Jetson Orin NX 16 GB of AGX Orin |
| **Geheugen (RAM)** | 8 GB (gedeeld GPU/CPU)         | 16 GB+ gedeeld                    |
| **Opslagruimte**      | 2 GB vrije ruimte               | NVMe SSD met 10 GB+ vrije ruimte        |
| **Python**       | Python 3.7+ (voor SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-versnelling**: Chloros+-gebruikers met NVIDIA GPU&#x27;s kunnen CUDA-versnelling gebruiken voor aanzienlijk snellere verwerking. Dit werkt zowel op Windows (desktop-GPU&#x27;s) als op Linux (desktop-GPU&#x27;s en NVIDIA Jetson). Chloros+-gebruikers profiteren ook van multithreaded verwerking voor maximale snelheid.
{% endhint %}

***

## Download Chloros

### Laatste stabiele release (23 maart 2026): Versie 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Download Chloros voor Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Download Chloros voor Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Download Chloros voor Linux arm64 / Jetson (.deb)</a>

#### Windows-installatieprogramma (GUI + CLI + Backend)

* **Bestandstype**: .exe (Windows-installatieprogramma)**Installatiestappen:**

1. Download het bovenstaande .exe-bestand
2. Dubbelklik op het installatieprogramma om de installatie te starten
3. Volg de aanwijzingen van de installatiewizard
4. Kies de installatiemap (standaard: `C:\Program Files\[USER]\Chloros\`)
5. Voltooi de installatie en start Chloros of Chloros CLI
6. Meld u aan met uw [MAPIR Cloud Chloros+ account](https://cloud.mapir.camera/pricing) (of ga verder met de gratis versie)

{% hint style="success" %}
Het installatieprogramma voegt automatisch `chloros-cli` toe aan de PATH van uw systeem voor toegang via de opdrachtregel.
{% endhint %}

#### Linux amd64 (.deb-pakket — CLI + Backend)

* **Bestandstype**: .deb (Debian/Ubuntu-pakket)
* **Architectuur**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb-pakket — CLI + Backend)

* **Bestandstype**: .deb (JetPack 6)
* **Architectuur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Zie [Linux Installatie](linux/linux-installation.md) voor gedetailleerde installatie-instructies en [NVIDIA Jetson-handleiding](linux/nvidia-jetson-guide.md) voor Jetson-specifieke richtlijnen.

#### Python SDK (Alle platforms)

```bash
pip install chloros-sdk
```

Zie [API : Python SDK](api-python-sdk.md) voor documentatie.

{% hint style="info" %}
**Linux-gebruikers**: Het `.deb`-pakket installeert de CLI en de backend. De Python SDK wordt apart geïnstalleerd via pip. Er is geen GUI voor Linux — alle interactie verloopt via CLI of SDK.
{% endhint %}

***

## Aanvullende bronnen

### Python SDK

Voor ontwikkelaars en automatiseringsworkflows installeert u de Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentatie**: [API: Python SDK](api-python-sdk.md)**Vereisten**: Chloros moet geïnstalleerd zijn (Windows-installatieprogramma of Linux `.deb`-pakket), Chloros+ licentie-login vereist***

## Wat zit erbij

### Windows-installatieprogramma

* ✅ **Chloros GUI** - Volledig uitgeruste grafische interface
* ✅ **Chloros CLI** - Opdrachtregelinterface (vereist Chloros+ licentie)
* ✅ **Chloros Backend** - Verwerkingsengine
* ✅ **Cameraprofielen** - Vooraf geconfigureerde MAPIR-camerasjablonen

### Linux .deb-pakket

* ✅ **Chloros CLI** - Opdrachtregelinterface (vereist Chloros+ licentie)
* ✅ **Chloros Backend** - Verwerkingsengine
* ✅ **Cameraprofielen** - Vooraf geconfigureerde MAPIR-camerasjablonen
* ❌ Geen GUI — Linux is alleen headless CLI/SDK

### Python SDK (pip, alle platforms)

* ✅ **Chloros SDK** - Python API (vereist Chloros+ licentie)***

## Upgrade naar Chloros+

Ontgrendel geavanceerde functies met een Chloros+-abonnement:

* 🚀 **Multi-threaded verwerking** - Verwerk afbeeldingen parallel
* ⚡ **GPU (CUDA) versnelling** - Maak gebruik van de kracht van NVIDIA GPU&#x27;s
* 💻 **CLI-toegang** - Automatiseer met opdrachtregelprogramma&#x27;s
* 🐍 **Python SDK** - Programmatische toegang tot API
* 📱 **Meerdere apparaten** - Gebruik op 2-10+ apparaten (afhankelijk van het abonnement)
* **🐻 Geavanceerde textuurbewuste debayer-methode** - een hoogwaardige randbewuste debayer gecombineerd met een AI/ML-ruisonderdrukkingsmodel dat bijna alle debayer-ruis verwijdert.
* 🧮 **Aangepaste formules** - Maak aangepaste multispectrale indices

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Bekijk Chloros+ abonnementen en prijzen</a></p>***

## Hulp bij installatie

### Probleemoplossing

**Installatie mislukt met foutmelding:**

* Zorg ervoor dat u beheerdersrechten hebt
* Schakel antivirussoftware tijdelijk uit
* Controleer of u voldoet aan de minimale systeemvereisten

**Toepassing start niet (Windows):**

* Controleer of Windows 10/11 (64-bits) is geïnstalleerd
* Werk grafische stuurprogramma&#x27;s bij
* Controleer Windows Event Viewer voor foutdetails
* Neem contact op met de ondersteuning met foutlogboeken

**CLI start niet (Linux):**

* Controleer of het `.deb`-pakket correct is geïnstalleerd: `dpkg -l | grep chloros`
* Controleer de machtigingen: `sudo chmod +x /usr/bin/chloros-cli`
* Voer diagnostiek uit: `chloros-cli selftest`
* Controleer op ontbrekende bibliotheken: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemen met licentieactivering:**

* Zorg ervoor dat de internetverbinding actief is
* Controleer de inloggegevens op [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Controleer of de firewall Chloros niet blokkeert
* Zie [Chloros+ Inloggen](chloros+-login.md) voor gedetailleerde instructies

### Ondersteuning krijgen

Hulp nodig bij de installatie of configuratie?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentatie**: [Aan de slag](./)
* ❓ **FAQ**: [Veelgestelde vragen](faq.md)***

## Wijzigingslogboek

<details>

<summary>Versie 1.1.0 (nieuwste)</summary>

**Releasedatum: maart 2026**

**Nieuwe functies*** **Linux-ondersteuning** — Native CLI en SDK voor Linux amd64 (x86\_64) en arm64 (NVIDIA Jetson JetPack 6). Installeer via `.deb`-pakketten.
* **NVIDIA Jetson-ondersteuning** — Geoptimaliseerde verwerking voor Jetson Nano-, Orin Nano-, Orin NX- en AGX Orin-edge-apparaten.
* **Dynamische rekenaanpassing** — Automatische hardwaredetectie en optimalisatie van de verwerkingsstrategie. Chloros past zich aan uw hardware aan, van een Jetson Nano tot een werkstation met meerdere GPU&#x27;s.
* **4-thread verwerkingspijplijn** — Gelijktijdige detectie-, kalibratie-, verwerkings- en exportthreads met dynamische GPU-geheugentoewijzing.
* **Nieuwe CLI-commando&#x27;s** — `selftest` (systeemdiagnostiek) en `update` (Linux-updatebeheer).
* **Nieuwe CLI-procesvlaggen** — `--debayer` (standaard/textuurbewust), `--indices` (indices specificeren), `--target` (zoek eerst in de doelsubmap voor snellere detectie).
* **Nieuwe GUI-menu-items** — Bestanden toevoegen, Map toevoegen en Verwerking starten/stoppen zijn nu toegankelijk via het dropdown-menu van het hoofdmenu.**Verbeteringen**

* Automatische detectie van cross-platform backend (Windows- en Linux-paden)
* Verbeterde SDK `get_status()` met voortgangsregistratie per thread
* Nieuwe SDK-uitzonderingen: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Thermisch beheer en adaptieve throttling voor NVIDIA Jetson
* Automatisch geheugenbeheer met OOM-fallback naar getilde GPU-verwerking

</details>

<details>

<summary>Versie 1.0.5</summary>

**Releasedatum: 10 februari 2026**

**Nieuwe functies*** **Texture Aware Debayer-methode \[Alleen Chloros+] -** Texture Aware maakt gebruik van een hoogwaardige, randbewuste debayer in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayering-ruis verwijdert.
* **Ondersteuning voor T4P-kalibratiedoelen*** **Snellere Chloros+ GPU-verwerking, beter geheugenbeheer**

**Bugfixes*** Volledig nieuwe frontend (GUI), zou nu op alle Windows-computers moeten werken.

</details>

<details>

<summary>Versie 1.0.4</summary>

**Releasedatum: 5 januari 2026**

**Nieuwe functies*** **Schakelaar voor afbeelding/metadata**: Schakelaar toegevoegd in de bestandsbrowser om de metadata van de geselecteerde afbeelding in een tabel te bekijken in plaats van in het afbeeldingsraster
* **Zoomschuifregelaar voor afbeeldingsraster**: Nieuwe UI-schuifregelaar om de grootte van de miniaturen aan te passen (ondersteunt ook CTRL + muiswiel)
* **Knoppen voor het exporteren van het afbeeldingsraster**: Knoppen in de bovenste rij om miniaturen te wisselen van JPG naar verwerkte exports (Targets, Reflectance, Index, LUT)
* **Tabblad Kaart**: Nieuwe interactieve 2D-kaart met GPS-locatiemarkeringen voor afbeeldingen
  * Ondersteunt Google Maps en ESRI-kaarttegels (selecteert automatisch de beste tegeldienst op basis van beschikbaarheid per zoomniveau)
  * Voorbeeld van miniatuur bij muisaanwijzer op kaartmarkeringen

**Bugfixes*** Verbeterde ondersteuning voor het installeren van Chloros op niet-Engelstalige computers

</details>

<details>

<summary>Versie 1.0.3</summary>

**Releasedatum: 20 december 2025**

**Nieuwe functies*** Eerste lancering

**Verbeteringen*** Eerste lancering

**Bugfixes*** Eerste lancering

**Bekende problemen*** Eerste lancering

</details>***

## Licentieovereenkomst**Eigendomssoftware** - Copyright (c) 2026 MAPIR Inc.

Ongeautoriseerd gebruik, verspreiding of wijziging is verboden.

**Gratis versie**: Beschikbaar voor persoonlijk en commercieel gebruik met beperkte functionaliteit**Chloros+**: Licentie op basis van een abonnement voor geavanceerde functies en commerciële implementaties
