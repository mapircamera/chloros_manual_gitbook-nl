---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Downloaden

Download de nieuwste versie van Chloros om aan de slag te gaan met multispectrale beeldverwerking.

### Systeemvereisten

| Vereiste          | Minimaal                                              | Aanbevolen                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Besturingssysteem** | Windows 10 (64-bits)                                  | Windows 11 (64-bits)                                  |
| **Processor**        | Intel Core i5 of gelijkwaardig                          | Intel Core i7 of beter                              |
| **Geheugen (RAM)**     | 8 GB                                                  | 16 GB of meer                                         |
| **Grafische kaart**    | Compatibel met DirectX 11                                | NVIDIA GPU met 4 GB+ VRAM                            |
| **Opslagruimte**          | 6 GB vrije ruimte                                       | SSD met 10 GB+ vrije ruimte                            |
| **Beeldscherm**          | 1920x1080                                            | 2560x1440 of hoger                                  |
| **Internet**         | Vereist voor \[optioneel] Chloros+ licentieactivering | Vereist voor \[optioneel] Chloros+ licentieactivering |

{% hint style="info" %}
**GPU-versnelling**: Chloros+-gebruikers met NVIDIA GPU&#x27;s kunnen CUDA-versnelling gebruiken voor aanzienlijk snellere verwerking. Chloros+-gebruikers profiteren ook van multi-threaded verwerking voor maximale snelheid.
{% endhint %}

***

## Download Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Download Chloros hier</a>

### Laatste stabiele release

**Chloros-installatieprogramma voor Windows*** **Versie**: 1.0.5
* **Releasedatum**: 10 februari 2026
* **Bestandsgrootte (download)**: 1,6 GB
* **Bestandsgrootte (geïnstalleerd)**: 5,7 GB
* **Bestandstype**: .exe (Windows-installatieprogramma)

#### **Installatiestappen:**

1. Download het bestand `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Dubbelklik op het installatieprogramma om de installatie te starten
3. Volg de instructies van de installatiewizard
4. Kies de installatiemap (standaard: `C:\Program Files\[USER]\Chloros\`)
5. Voltooi de installatie en start Chloros of Chloros CLI
6. Meld u aan met uw [MAPIR Cloud Chloros+ account](https://cloud.mapir.camera/pricing) (of ga verder met de gratis versie)

{% hint style="success" %}
Het installatieprogramma voegt automatisch `chloros-cli` toe aan uw systeem PATH voor toegang via de opdrachtregel.
{% endhint %}

***

## Aanvullende bronnen

### Python SDK

Voor ontwikkelaars en automatiseringsworkflows installeert u Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentatie**: [API: Python SDK](api-python-sdk.md)**Vereisten**: Chloros Desktop moet geïnstalleerd zijn, Chloros+ licentie-login vereist.***

## Wat zit erbij?

De Chloros-installatie omvat:

* ✅ **Chloros** - Volledig uitgeruste grafische interface (GUI)
* ✅ **Chloros CLI** - Opdrachtregelinterface (vereist Chloros+-licentie)
* ✅ **Chloros SDK** - Python API (vereist Chloros+ licentie)
* ✅ **Cameraprofielen** - Vooraf geconfigureerde MAPIR camerasjablonen***

## Upgrade naar Chloros+

Ontgrendel geavanceerde functies met een Chloros+ abonnement:

* 🚀 **Multi-threaded verwerking** - Verwerk beelden parallel
* ⚡ **GPU (CUDA) versnelling** - Maak gebruik van de kracht van NVIDIA GPU
* 💻 **CLI-toegang** - Automatiseer met command-line tools
* 🐍 **Python SDK** - Programmatische API-toegang
* 📱 **Meerdere apparaten** - Gebruik op 2-10+ apparaten (afhankelijk van het abonnement)
* **🐻 Geavanceerde textuurgevoelige debayer-methode** - een hoogwaardige randgevoelige debayer in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayeringruis verwijdert. 
* 🧮 **Aangepaste formules** - Maak aangepaste multispectrale indices

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Bekijk Chloros+ Abonnementen en prijzen</a></p>***

## Installatiehulp

### Probleemoplossing

**Installatie mislukt met foutmelding:**

* Zorg ervoor dat u beheerdersrechten hebt
* Schakel antivirussoftware tijdelijk uit
* Controleer of u voldoet aan de minimale systeemvereisten

**Toepassing start niet:**

* Controleer of Windows 10/11 (64-bit) is geïnstalleerd
* Werk grafische stuurprogramma&#x27;s bij
* Controleer Windows Event Viewer voor foutdetails
* Neem contact op met ondersteuning met foutlogboeken

**Problemen met licentieactivering:**

* Zorg ervoor dat de internetverbinding actief is
* Controleer de inloggegevens op [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Controleer of de firewall Chloros niet blokkeert
* Zie [Chloros+ Login](chloros+-login.md) voor gedetailleerde instructies

### Ondersteuning krijgen

Hulp nodig bij de installatie of configuratie?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentatie**: [Aan de slag](./)
* ❓ **FAQ**: [Veelgestelde vragen](faq.md)***

## Wijzigingslogboek

<details>

<summary>Versie 1.0.5</summary>

#### **Releasedatum**: 10 februari 2026**Nieuwe functies*** **Texture Aware Debayer-methode \[Chloros+ Alleen] -** Texture Aware maakt gebruik van een hoogwaardige edge-aware debayer in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayeringruis verwijdert.
* **Ondersteuning voor T4P-kalibratiedoelen*** **Snellere Chloros+ GPU-verwerking, beter geheugenbeheer**

**Bugfixes*** Volledig nieuwe frontend (GUI), zou nu op alle Windows-computers moeten werken.

</details>

<details>

<summary>Versie 1.0.4</summary>

#### **Releasedatum**: 5 januari 2026**Nieuwe functies*** **Afbeelding/metadata-schakelaar**: schakelaar toegevoegd in bestandsbrowser om de metadata van de geselecteerde afbeelding in een tabel te bekijken in plaats van in het afbeeldingsraster
* **Zoomschuifregelaar voor afbeeldingsraster**: nieuwe UI-schuifregelaar om de grootte van miniaturen aan te passen (ondersteunt ook CTRL + muiswiel)
* **Exportknoppen afbeeldingsraster**: knoppen in de bovenste rij om miniaturen te wijzigen van JPG naar verwerkte exports (doelen, reflectie, index, LUT)
* **Tabblad Kaart**: Nieuwe interactieve 2D-kaart met GPS-locatiemarkeringen voor afbeeldingen.
  * Ondersteunt Google Maps en ESRI-kaarttegels (selecteert automatisch de beste tegelservice op basis van de beschikbaarheid van het zoomniveau).
  * Miniatuurvoorvertoning bij muisaanwijzer op kaartmarkeringen.

**Bugfixes*** Verbeterde ondersteuning voor het installeren van Chloros op niet-Engelstalige computers.

</details>

<details>

<summary>Versie 1.0.3</summary>

#### **Releasedatum**: 20 december 2025**Nieuwe functies*** Eerste lancering

**Verbeteringen*** Eerste lancering

**Bugfixes*** Eerste lancering

**Bekende problemen*** Eerste lancering

</details>***

## Licentieovereenkomst**Eigendomsrechtelijke software** - Copyright (c) 2026 MAPIR Inc.

Ongeautoriseerd gebruik, distributie of wijziging is verboden.

**Gratis versie**: beschikbaar voor persoonlijk en commercieel gebruik met beperkte functionaliteit.**Chloros+**: licentie op basis van een abonnement voor geavanceerde functies en commerciële toepassingen.
