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
| **Internet**         | Vereist voor \[optionele] activering van de Chloros+-licentie | Vereist voor \[optionele] activering van de Chloros+-licentie |

#### Linux amd64 (x86\_64)

| Vereisten       | Minimaal                    | Aanbevolen               |
| ----------------- | -------------------------- | ------------------------- |
| **Distributie**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processor**     | x86\_64 (Intel/AMD)        | Intel Core i7 of beter   |
| **Geheugen (RAM)**  | 8 GB                        | 16 GB of meer              |
| **Grafische kaart** | Geen (verwerking via CPU)      | NVIDIA GPU met 4 GB+ VRAM |
| **Opslagruimte**       | 2 GB vrije ruimte             | SSD met 10 GB+ vrije ruimte       |
| **Python**        | Python 3.7+ (voor SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Vereisten      | Minimaal                      | Aanbevolen                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson met JetPack 6 | Jetson Orin NX 16 GB of AGX Orin |
| **Geheugen (RAM)** | 8 GB (gedeeld tussen GPU en CPU)         | 16 GB+ gedeeld                    |
| **Opslagruimte**      | 2 GB vrije ruimte               | NVMe SSD met 10 GB+ vrije ruimte        |
| **Python**       | Python 3.7+ (voor SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-versnelling**: Chloros+-gebruikers met NVIDIA-GPU’s kunnen CUDA-versnelling gebruiken voor aanzienlijk snellere verwerking. Dit werkt met NVIDIA-desktop-GPU’s op Windows en Linux amd64, en op NVIDIA Jetson met JetPack 6. Chloros+-gebruikers profiteren ook van multithreaded verwerking voor maximale snelheid.
{% endhint %}

***

## Chloros
downloaden

### Nieuwste stabiele release: versie 1.3.0 (11 november 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Chloros voor Windows downloaden (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Chloros voor Linux amd64 downloaden (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Download Chloros voor Linux arm64 / Jetson (.deb)</a>


#### Windows-installatieprogramma (GUI + CLI + Backend)

* **Bestandstype**: .exe (Windows-installatieprogramma)

**Installatiestappen:**

1. Download het bovenstaande bestand .exe
2. Dubbelklik op het installatieprogramma om de installatie te starten
3. Volg de aanwijzingen van de installatiewizard
4. Kies de installatiemap (standaard: `C:\Program Files\Chloros\`)
5. Laat op de pagina **Extra taken selecteren**beide opties onder**Systeeminstellingen** aangevinkt (aanbevolen — zie hieronder)
6. Voltooi de installatie en start Chloros of Chloros CLI
7. Meld je aan met je [MAPIR Cloud Chloros+-account](https://cloud.mapir.camera/pricing) (of ga verder met de gratis versie)

**Opties voor systeeminstellingen** (standaard beide aangevinkt):

| Optie, zoals vermeld in het installatieprogramma | Wat het doet | Afzien van stille installatie |
| --- | --- | --- |
| **Windows-lange paden inschakelen** (maakt het mogelijk dat projectmappen dieper dan 260 tekens liggen; aanbevolen) | Stelt de Windows-instelling `LongPathsEnabled`, die voor het hele systeem geldt, in op 1. Windows wordt standaard geleverd met deze instelling uitgeschakeld, waardoor geen enkel bestandspad langer mag zijn dan 260 tekens; Chloros-producten bevinden zich tot ongeveer 130 tekens onder de projectmap, dus als lange paden zijn uitgeschakeld, wordt een dieper gelegen projectmap geweigerd. Wordt van kracht voor programma’s die na de installatie worden gestart. De instelling blijft ook na het verwijderen van het programma actief, omdat andere programma’s er mogelijk op vertrouwen. | `/MERGETASKS="!longpaths"` |
| **De Chloros-backend uitsluiten van scans door Microsoft Defender** (veel snellere camerabeelden; aanbevolen) | Voegt een uitsluiting van het Microsoft Defender-proces toe voor `chloros-backend.exe` in de installatiemap, zodat de realtime bescherming niet elk bestand scant dat de backend schrijft, waar het ook wordt opgeslagen. Bij hoge opnamesnelheden kost het scannen ongeveer 30% van de opnamesnelheid (77,7 versus 52,7 opnames per seconde bij een continue opname met 9 camera’s). Er wordt geen enkel ander programma uitgesloten en de uitsluiting wordt bij het verwijderen ongedaan gemaakt. Wanneer Defender ontbreekt of door beleid is geblokkeerd, mislukt deze stap stilzwijgend en gaat de installatie door. | `/MERGETASKS="!defenderexclusion"` |

Bij een stille installatie (`/SILENT` of `/VERYSILENT`) worden beide opties toegepast, tenzij u zich afmeldt met de weergegeven waarde `/MERGETASKS`.

{% hint style="success" %}
Het installatieprogramma voegt de map CLI (`C:\Program Files\Chloros\cli\`) één keer toe aan uw systeem `PATH` — bij herinstallatie of upgrade wordt er geen dubbele vermelding meer toegevoegd en worden duplicaten verwijderd die door eerdere installatieprogramma’s zijn toegevoegd. Open na de installatie een nieuwe terminal om `chloros-cli` te gebruiken. Het installatieprogramma voegt ook twee Windows-firewallregels toe voor de backend, in elk netwerkprofiel: **Chloros Backend (UDP-In)**, voor camerastreams, PTP, mDNS en de DAQ-E-datastream, en**Chloros Backend (TCP-In)**, beperkt tot het lokale subnet, voor een DAQ-E-firmware-update, waarbij de sensor verbinding maakt met deze computer om het beeld op te halen. Beide worden bij het verwijderen verwijderd.
{% endhint %}

#### Linux amd64 (.deb-pakket — CLI + backend)

* **Bestandstype**: .deb (Debian/Ubuntu-pakket)
* **Architectuur**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb-pakket — CLI + Backend)

* **Bestandstype**: .deb (JetPack 6)
* **Architectuur**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Gebruik de bestandsnaam die je daadwerkelijk hebt gedownload en behoud de `./`, zodat `apt` dat bestand installeert in plaats van naar een pakket op naam te zoeken. Installeer met `apt install`, niet met `dpkg -i`: apt installeert ook de vereisten van de Python SDK, die het pakket aanbeveelt in plaats van vereist, en `dpkg -i` slaat deze over. Sinds versie 1.3.0 worden de backend en de CLI uitgepakt geïnstalleerd onder `/usr/lib/chloros/`, zodat ze starten zonder zichzelf eerst uit te pakken.

Het Jetson-pakket maakt gebruik van de CUDA-build van PyTorch uit JetPack 6, dus de verwerking, inclusief de ingebouwde Texture Aware-debayer, draait op de GPU van de Jetson. Het maakt gebruik van de CUDA- en cuDNN-bibliotheken die JetPack 6 installeert (`sudo apt install nvidia-jetpack` op een image waar deze ontbreken).

Zie [Linux-installatie](linux/linux-installation.md) voor gedetailleerde installatie-instructies en [NVIDIA Jetson-handleiding](linux/nvidia-jetson-guide.md) voor Jetson-specifieke richtlijnen.

#### Python SDK (alle platforms)

Elk installatieprogramma bevat een bijbehorend `chloros_sdk`-wheel, zodat de SDK-versie altijd overeenkomt met de geïnstalleerde GUI/CLI/backend. Op Windows installeert het installatieprogramma dit automatisch in de Python van je systeem. Op Linux installeert de `.deb` de SDK voor de systeem-`python3` zelf (in `/usr/lib/python3/dist-packages`), zodat `import chloros_sdk` direct na de installatie werkt, zonder pip en zonder netwerk. Alleen een virtualenv, die de systeempakketten niet ziet, heeft het meegeleverde wheel nodig dat daarin geïnstalleerd moet worden:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Voor hosts die alleen pip gebruiken (waar geen Chloros-pakket is geïnstalleerd; de SDK stuurt dan een backend op een andere machine aan), is de SDK ook beschikbaar op PyPI:

```bash
pip install chloros-sdk
```

Zie [API: Python SDK](api-python-sdk.md) en de [SDK-referentie](reference/sdk-reference.md) voor documentatie.

{% hint style="info" %}
**Linux-gebruikers**: Het pakket `.deb` installeert de CLI, de backend en de Python-SDK. Er is geen GUI voor Linux — alle interactie verloopt via de CLI of de SDK.
{% endhint %}

***

## Aanvullende bronnen

### Python SDK


Voor ontwikkelaars en automatiseringsworkflows wordt de Chloros Python SDK bij elke installatieprogramma meegeleverd: het Windows-installatieprogramma installeert het in uw Python, en het Linux-pakket `.deb` installeert het voor het systeem `python3`. Op een machine zonder Chloros-pakket (een machine die een backend op een andere machine aanstuurt), installeert u deze via PyPI:

```bash
pip install chloros-sdk
```

**Documentatie**: [API: Python SDK](api-python-sdk.md)

**Vereisten**: een Chloros-backend om aan te sturen (de Windows-installatieprogramma of het Linux-pakket `.deb` installeert er een), en een Chloros+-login op een betaald abonnement

***

## Wat zit erbij

### Windows-installatieprogramma

* ✅ **Chloros GUI** - Volledig uitgeruste grafische interface
* ✅ **Chloros CLI** - Opdrachtregelinterface (vereist een Chloros+ licentie)
* ✅ **Chloros-backend** - Verwerkingsengine
* ✅ **Python SDK** - geïnstalleerd in uw Python indien aanwezig (vereist een Chloros+ licentie)
* ✅ **Cameraprofielen** - Vooraf geconfigureerde MAPIR-camerasjablonen

### Linux.deb-pakket

* ✅ **Chloros CLI** - Opdrachtregelinterface (vereist een Chloros+ licentie)
* ✅ **Chloros Backend** - Verwerkingsengine
* ✅ **Python SDK** - geïnstalleerd voor het systeem `python3` (vereist een Chloros+ licentie)
* ✅ **Cameraprofielen** - Vooraf geconfigureerde MAPIR-camerasjablonen
* ❌ Geen GUI — Linux is headless, alleen CLI/SDK


### Python-SDK (PyPI, voor machines zonder Chloros-pakket)

* ✅ **Chloros-SDK** - Python-API (vereist een Chloros+-licentie)

***

## Upgrade naar Chloros+

Ontgrendel geavanceerde functies met een Chloros+-abonnement:

* 🚀 **Multi-threaded verwerking** - Verwerk beelden parallel
* ⚡ **GPU (CUDA)-versnelling** - Maak gebruik van de kracht van NVIDIA GPU’s
* 💻 **Toegang via de CLI** - Automatiseer met command-line tools
* 🐍 **Python SDK** - Programmatische API-toegang
* 📱 **Meerdere apparaten** - Gebruik op 2-10+ apparaten (afhankelijk van het abonnement)
* **🐻 Geavanceerde textuurbewuste debayer-methode** - een hoogwaardige, randbewuste debayer gecombineerd met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayer-ruis verwijdert.
* 🧮 **Aangepaste formules** - Maak aangepaste multispectrale indices

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Bekijk de abonnementen en prijzen vChloros+</a></p>***

## Hulp bij de installatie

### Probleemoplossing

**Installatie mislukt met de volgende foutmelding:**

* Zorg ervoor dat je beheerdersrechten hebt
* Schakel antivirussoftware tijdelijk uit
* Controleer of je voldoet aan de minimale systeemvereisten

**De applicatie start niet (Windows):**

* Controleer of Windows 10/11 (64-bit) is geïnstalleerd
* Werk de grafische stuurprogramma’s bij
* Controleer de Windows-gebeurtenisviewer voor foutdetails
* Neem contact op met de ondersteuning met de foutlogboeken

**De CLI-interface start niet (Linux):**

* Controleer of het pakket `.deb` correct is geïnstalleerd: `dpkg -l | grep chloros`
* Controleer de machtigingen: `sudo chmod +x /usr/bin/chloros-cli`
* Voer een diagnose uit: `chloros-cli selftest`
* Controleer of er bibliotheken ontbreken: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (vanaf versie 1.3.0 is `/usr/lib/chloros/chloros-backend` een klein opstartscript; het programma zelf bevindt zich in `/usr/lib/chloros/backend/`)

**Problemen met licentieactivering:**

* Zorg ervoor dat de internetverbinding actief is
* Controleer de inloggegevens op [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Controleer of de firewall Chloros niet blokkeert* Zie [Chloros+ Inloggen](chloros+-login.md) voor gedetailleerde instructies

### Ondersteuning krijgen

Hulp nodig bij de installatie of configuratie?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentatie**: [Aan de slag](./)
* ❓ **Veelgestelde vragen**: [Veelgestelde vragen](faq.md)***

## Software-updates

Chloros controleert op updates, geeft aan wanneer er een nieuwe versie beschikbaar is en verwijst naar deze downloadpagina — je voert de update uit door het nieuwe, ondertekende installatieprogramma te starten. Je instellingen, projecten en inloggegevens blijven bij updates behouden. Op Linux en Jetson controleert `chloros-cli update` of er een nieuwere versie is en biedt het aan om de bijbehorende `.deb` te downloaden en te installeren (dit commando is alleen beschikbaar op Linux).

***

## Wijzigingslogboek**Versie 1.3.0 (nieuwste, 11 november 2026)**— zie**Wat is er nieuw in Chloros

1.3.0** op de pagina [Aan de slag](./) voor de volledige lijst.**Versie 1.2.0**— zie**Wat is er nieuw in Chloros 1.2.0** op de pagina [Aan de slag](./).

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

**Nieuwe functies*** **Schakelaar voor afbeelding/metadata**: Schakelaar toegevoegd in de bestandsbrowser om de metadata van de geselecteerde afbeelding in een tabel te bekijken in plaats van in het afbeeldingsraster
* **Zoomschuifregelaar voor afbeeldingsraster**: Nieuwe UI-schuifregelaar om de grootte van de miniaturen aan te passen (ondersteunt ook CTRL + muiswiel)
* **Knoppen voor het exporteren van het afbeeldingsraster**: Knoppen in de bovenste rij om miniaturen om te schakelen van JPG naar bewerkte exportformaten (Targets, Reflectance, Index, LUT)
* **Tabblad Kaart**: Nieuwe interactieve 2D-kaart met GPS-locatiemarkeringen voor afbeeldingen
  * Ondersteunt Google Maps en ESRI-kaarttegels (selecteert automatisch de beste tegeldienst op basis van de beschikbaarheid per zoomniveau)
  * Voorbeeld van miniatuur bij muisaanwijzer op kaartmarkeringen

**Bugfixes*** Verbeterde ondersteuning voor het installeren van Chloros op computers met een andere taal dan Engels

</details>

<details>

<summary>Versie 1.0.3</summary>

**Releasedatum: 20 december 2025**

**Nieuwe functies*** Eerste release

**Verbeteringen*** Eerste release

**Bugfixes*** Eerste release

**Bekende problemen*** Eerste release

</details>

***

## Licentieovereenkomst**Eigen software** - Copyright (c) 2026 MAPIR Inc.

Ongeautoriseerd gebruik, verspreiding of wijziging is verboden.

**Gratis versie**: Beschikbaar voor persoonlijk en commercieel gebruik met beperkte functionaliteit**Chloros+**: Licentie op basis van een abonnement voor geavanceerde functies en commerciële implementaties
