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
| **Opslagruimte**          | 6 GB vrije ruimte                                       | SSD met 10 GB of meer vrije ruimte                            |
| **Beeldscherm**          | 1920x1080                                            | 2560x1440 of hoger                                  |
| **Internet**         | Vereist voor \[optioneel\] Chloros+ licentieactivering | Vereist voor \[optioneel\] Chloros+ licentieactivering |

#### Linux amd64 (x86_64)

| Vereisten       | Minimaal                    | Aanbevolen               |
| ----------------- | -------------------------- | ------------------------- |
| **Distributie**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processor**     | x86_64 (Intel/AMD)        | Intel Core i7 of beter   |
| **Geheugen (RAM)**  | 8 GB                        | 16 GB of meer              |
| **Grafische kaart** | Geen (verwerking via CPU)      | NVIDIA GPU met 4 GB+ VRAM |
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
**GPU-versnelling**: Gebruikers van Chloros+ met NVIDIA-GPU&#x27;s kunnen CUDA-versnelling gebruiken voor aanzienlijk snellere verwerking. Dit werkt zowel op Windows (desktop-GPU’s) als op Linux (desktop-GPU’s en NVIDIA Jetson). Gebruikers van Chloros+ profiteren bovendien van multithreaded verwerking voor maximale snelheid.
{% endhint %}

***

## Download Chloros

### Nieuwste stabiele release: versie 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Download Chloros voor Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Download Chloros voor Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Download Chloros voor Linux arm64 / Jetson (.deb)</a>

#### Windows-installatieprogramma (GUI + CLI + Backend)

* **Bestandstype**: .exe (Windows-installatieprogramma)**Installatiestappen:**

1. Download het bovenstaande .exe-bestand
2. Dubbelklik op het installatieprogramma om de installatie te starten
3. Volg de aanwijzingen van de installatiewizard
4. Kies de installatiemap (standaard: `C:\Program Files\MAPIR\Chloros\`)
5. Voltooi de installatie en start Chloros of Chloros CLI
6. Meld je aan met je [MAPIR Cloud Chloros+-account](https://cloud.mapir.camera/pricing) (of ga verder met de gratis versie)

{% hint style="success" %}
Het installatieprogramma voegt automatisch `chloros-cli` toe aan de PATH van je systeem voor toegang via de opdrachtregel.
{% endhint %}

#### Linux amd64 (.deb-pakket — CLI + Backend)

* **Bestandstype**: .deb (Debian/Ubuntu-pakket)
* **Architectuur**: x86_64 (amd64)

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

#### Python SDK (alle platforms)

Elk installatieprogramma bevat een bijbehorend `chloros_sdk`-wiel, zodat de SDK-versie altijd overeenkomt met de geïnstalleerde GUI/CLI/backend. Op Windows installeert het installatieprogramma het automatisch in uw systeem Python; op Linux plaatst `.deb` het wheel in `/usr/lib/chloros/sdk/` en geeft het installatiecommando weer:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Voor hosts die alleen pip gebruiken (waarop geen Chloros-pakket is geïnstalleerd), is de SDK ook beschikbaar op PyPI:

```bash
pip install chloros-sdk
```

Zie [API : Python SDK](api-python-sdk.md) en de [SDK-referentie](reference/sdk-reference.md) voor documentatie.

{% hint style="info" %}
**Linux-gebruikers**: Het `.deb`-pakket installeert het CLI en de backend. Er is geen GUI voor Linux — alle interactie verloopt via CLI of SDK.
{% endhint %}

***

## Aanvullende bronnen

### Python SDK

Voor ontwikkelaars en automatiseringsworkflows installeert u de Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentatie**: [API: Python SDK](api-python-sdk.md)**Vereisten**: Chloros moet geïnstalleerd zijn (Windows-installatieprogramma of Linux `.deb`-pakket), Chloros+ licentie-inlog vereist***

## Wat zit erbij

### Windows-installatieprogramma

* ✅ **Chloros GUI** - Volledig uitgeruste grafische interface
* ✅ **Chloros CLI** - Opdrachtregelinterface (vereist een Chloros+ licentie)
* ✅ **Chloros Backend** - Verwerkingsengine
* ✅ **Cameraprofielen** - Vooraf geconfigureerde MAPIR-camerasjablonen

### Linux .deb-pakket

* ✅ **Chloros CLI** - Opdrachtregelinterface (vereist Chloros+ licentie)
* ✅ **Chloros Backend** - Verwerkingsengine
* ✅ **Cameraprofielen** - Vooraf geconfigureerde MAPIR-camerasjablonen
* ❌ Geen grafische gebruikersinterface — Linux is uitsluitend een headless CLI/SDK

### Python SDK (pip, alle platforms)

* ✅ **Chloros SDK** - Python API (vereist een Chloros+-licentie)***

## Upgrade naar Chloros+

Ontgrendel geavanceerde functies met een Chloros+-abonnement:

* 🚀 **Multi-threaded verwerking** - Verwerk afbeeldingen parallel
* ⚡ **GPU (CUDA)-versnelling** - Maak gebruik van de kracht van NVIDIA-GPU&#x27;s
* 💻 **Toegang tot CLI** - Automatiseer met opdrachtregelprogramma&#x27;s
* 🐍 **Python SDK** - Programmatische toegang tot API
* 📱 **Meerdere apparaten** - Gebruik op 2-10+ apparaten (afhankelijk van het abonnement)
* **🐻 Geavanceerde textuurbewuste debayermethode** - een hoogwaardige, randbewuste debayer in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayerruis verwijdert.
* 🧮 **Aangepaste formules** - Maak aangepaste multispectrale indices

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Bekijk Chloros+ abonnementen en prijzen</a></p>***

## Hulp bij de installatie

### Probleemoplossing

**Installatie mislukt met foutmelding:**

* Zorg ervoor dat je beheerdersrechten hebt
* Schakel antivirussoftware tijdelijk uit
* Controleer of je voldoet aan de minimale systeemvereisten

**De applicatie start niet (Windows):**

* Controleer of Windows 10/11 (64-bit) is geïnstalleerd
* Werk de grafische stuurprogramma’s bij
* Controleer de Windows Gebeurtenisviewer voor foutdetails
* Neem contact op met de ondersteuning en stuur de foutenlogboeken mee

**CLI start niet (Linux):**

* Controleer of het `.deb`-pakket correct is geïnstalleerd: `dpkg -l | grep chloros`
* Controleer de machtigingen: `sudo chmod +x /usr/bin/chloros-cli`
* Voer een diagnose uit: `chloros-cli selftest`
* Controleer of er bibliotheken ontbreken: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

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
* ❓ **Veelgestelde vragen**: [Veelgestelde vragen](faq.md)***

## Software-updates

Chloros controleert op updates, geeft aan wanneer er een nieuwe versie beschikbaar is en linkt naar deze downloadpagina — u voert de update uit door het nieuwe, ondertekende installatieprogramma uit te voeren. Uw instellingen en projecten blijven behouden na updates. Op Linux en Jetson controleert `chloros-cli update` of er een nieuwere versie is en biedt het aan om de bijbehorende `.deb` te downloaden en te installeren (dit commando is alleen beschikbaar in Linux).

***

## Wijzigingslogboek**Versie 1.2.0 (nieuwste)**— zie**Wat is er nieuw in Chloros 1.2.0** op de pagina [Aan de slag](./) voor de volledige lijst met functies.

<details>

<summary>Versie 1.0.5</summary>

**Releasedatum: 10 februari 2026**

**Nieuwe functies*** **Texture Aware-debayer-methode \[Alleen Chloros+] -** Texture Aware maakt gebruik van een hoogwaardige, randgevoelige debayer in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayer-ruis verwijdert.
* **Ondersteuning voor T4P-kalibratiedoelen*** **Snellere Chloros+ GPU-verwerking, beter geheugenbeheer**

**Bugfixes*** Volledig nieuwe frontend (GUI), zou nu op alle Windows-computers moeten werken.

</details>

<details>

<summary>Versie 1.0.4</summary>

**Releasedatum: 5 januari 2026**

**Nieuwe functies*** **Schakelaar voor afbeeldingen/metadata**: Schakelaar toegevoegd in de bestandsbrowser om de metadata van de geselecteerde afbeelding in een tabel te bekijken in plaats van in het afbeeldingsraster
* **Zoomschuifbalk voor afbeeldingsraster**: Nieuwe UI-schuifbalk om de grootte van de miniaturen aan te passen (ondersteunt ook CTRL + muiswiel)
* **Knoppen voor het exporteren van het afbeeldingsraster**: Knoppen in de bovenste rij om miniaturen om te schakelen van JPG naar bewerkte exportformaten (Targets, Reflectance, Index, LUT)
* **Tabblad Kaart**: Nieuwe interactieve 2D-kaart met GPS-locatiemarkeringen voor afbeeldingen
  * Ondersteunt Google Maps en ESRI-kaarttegels (selecteert automatisch de beste tegeldienst op basis van de beschikbaarheid per zoomniveau)
  * Voorbeeld van miniatuurafbeelding bij kaartmarkeringen wanneer de muis eroverheen beweegt

**Bugfixes*** Verbeterde ondersteuning voor het installeren van Chloros op computers met een andere taal dan Engels

</details>

<details>

<summary>Versie 1.0.3</summary>

**Releasedatum: 20 december 2025**

**Nieuwe functies*** Eerste release

**Verbeteringen*** Eerste release

**Bugfixes*** Eerste release

**Bekende problemen*** Eerste release

</details>***

## Licentieovereenkomst**Eigen software** - Copyright (c) 2026 MAPIR Inc.

Ongeautoriseerd gebruik, verspreiding of wijziging is verboden.

**Gratis versie**: Beschikbaar voor persoonlijk en commercieel gebruik met beperkte functionaliteit**Chloros+**: Licentie op basis van een abonnement voor geavanceerde functies en commerciële implementaties
