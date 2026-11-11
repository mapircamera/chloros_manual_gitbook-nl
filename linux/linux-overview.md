# Overzicht van Linux

Chloros 1.3.0 biedt native Linux-ondersteuning voor de **CLI**en**Python SDK** — headless verwerking van multispectrale beelden, plus live aansturing van LATTICE-camera’s en DAQ-lichtsensoren — op Linux-werkstations, servers en NVIDIA Jetson-edge-apparaten.

{% hint style="info" %}
**Geen desktop-GUI op Linux.**De Chloros Desktop-GUI is uitsluitend beschikbaar voor Windows. Linux-gebruikers werken met Chloros via de [CLI](../CLI.md) en de [Python SDK](../api-python-sdk.md). De `.deb` voegt wel een**Chloros CLI**-item toe aan je applicatiemenu — het opent simpelweg een terminalemulator waarop `chloros-cli` draait.
{% endhint %}

***

## Platformondersteuningsmatrix

| Functie | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop-GUI** | Ja | n.v.t. | Nee | Nee |
| **CLI** (`chloros-cli`) | Ja | Ja | Ja | Ja |
| **Python SDK** (`chloros-sdk`) | Ja | Ja | Ja | Ja |
| **Pijplijn voor beeldverwerking** | Ja | Ja | Ja | Ja |
| **LATTICE-camerabesturing (live)** | Ja (tabblad Camera’s) | Ja (`chloros-cli lattice`, SDK) | Ja | Ja |
| **DAQ-lichtsensoren (live)** | Ja (tabblad ‘Lichtsensoren’) | Ja (`chloros-cli daq pool-*`, SDK) | Ja | Ja |
| **PTP-tijdsynchronisatie (host is grandmaster)** | Ja | Ja (`chloros-cli time-sync`) | Ja | Ja |
| **GPU-versnelling (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Texture Aware debayer** (vereist een CUDA-GPU) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamic Compute Adaptation** | Ja | Ja | Ja | Ja |
| **Backend als systeemservice** (`chloros-backend@<user>.service`) | Nee | Nee | Ja (opt-in) | Ja (opt-in) |
| **In-place updater** (`chloros-cli update`) | Nee (het installatieprogramma uitvoeren) | Nee (het installatieprogramma uitvoeren) | Ja | Ja |***

## Ondersteunde architecturen

| Architectuur | Beschrijving | Pakket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standaard desktop-/serverprocessors (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-processors — NVIDIA Jetson Orin-serie (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Ondersteunde Linux-distributies

* **Ubuntu 22.04 LTS of nieuwer** (amd64)
* **Debian 12 of nieuwer** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin-platforms)***

## Wat Linux-gebruikers krijgen

* **Chloros CLI** — de volledige opdrachtregelinterface voor batchverwerking, automatisering en scripting
* **Chloros Python SDK** — programmatische Python-interface voor onderzoekspijplijnen en aangepaste tools, geïnstalleerd via het `.deb` voor het systeem `python3` (een wheel met bijpassende versie voor virtualenvs zit in `/usr/lib/chloros/sdk/`; PyPI is voor machines zonder een Chloros-pakket)
* **LATTICE-camerabesturing** — LATTICE-camera’s en gesynchroniseerde multi-camera-opstellingen detecteren, aansluiten, configureren en beelden vastleggen via `chloros-cli lattice` en de SDK; `.deb` bevat de Arena SDK-runtime die de camera’s nodig hebben
* **DAQ-lichtsensorbesturing** — sluit DAQ-U/M/E-sensoren aan, stream gekalibreerde spectra en neem `.daq`-bestanden op via `chloros-cli daq pool-*` en de SDK
* **PTP-tijdsynchronisatie** — de Chloros-backend draait de PTP-grandmaster waaraan LATTICE-camera’s en DAQ-E-sensoren zijn gekoppeld; controleer deze met `chloros-cli time-sync` en houd deze headless draaiend met de `chloros-backend@<user>.service` systemd-unit (zie [Linux-installatie](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projectautomatisering** — voer opgeslagen projecten headless uit met `chloros-cli project` en de `open_project` van de SDK
* **GPU-versnelling** — CUDA-versnelde verwerking op NVIDIA-GPU’s (desktop-GPU’s op amd64 en de GPU van de Jetson met JetPack 6)
* **Dynamische rekenaanpassing** — automatische hardware-detectie en selectie van verwerkingsstrategieën, met de `CHLOROS_STRATEGY`-override als noodoplossing voor experts
* **Alle verwerkingsfuncties** — dezelfde pijplijn als in Windows: kalibratie, vignetteringscorrectie, vegetatie-indexen en alle exportformaten
* **Chloros+ functies** — multithreaded (pipelined) verwerking, Texture Aware debayer (op een CUDA-GPU) en aangepaste indices, met een betaald Chloros+ abonnement

## Wat Linux-gebruikers niet krijgen

* **Desktop-GUI** — geen grafische interface; alle interactie verloopt via de CLI of de Python-SDK
* **Beeldviewer** — geen interactieve beeldviewer, rasterweergave of kaartmarkeringen
* **Visueel projectbeheer** — projecten worden aangemaakt en aangestuurd via CLI-opdrachten en SDK-aanroepen (de hardware zelf — camera’s, sensoren, opname — blijft volledig bedienbaar via de terminal)***

## Licentievereisten

Toegang tot de CLI en SDK vereist een **betaald Chloros+-abonnement — Copper of hoger**(Copper, Bronze, Silver, Gold). Het gratis**Iron**-abonnement biedt geen toegang tot de CLI/SDK. Deze beperking wordt afgedwongen door de backend, niet alleen door de CLI:

| Situatie | Reactie van de backend |
| --- | --- |
| Niet ingelogd | `401` met `error_code: AUTH_REQUIRED` |
| Ingelogd op het gratis Iron-abonnement | `403` met `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` werkt op elk tier — het is de enige route die is vrijgesteld van de gate — dus de reden voor een weigering is altijd zichtbaar.

***

## Aan de slag op Linux

1. **Chloros installeren** — zie [Linux-installatie](linux-installation.md) voor de installatie van `.deb`
2. **Controleer** — `chloros-cli --version` geeft `Chloros CLI 1.3.0` weer; `chloros-cli selftest` voert de 7-stappen-diagnose uit
3. **Controleer de Python SDK** — de `.deb` installeert deze: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (in een virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Meld je aan** — `chloros-cli login your@email.com 'your-password'` (eenmaal per machine; bij pakketupgrades blijft de aanmelding behouden, bij het verwijderen van het pakket wordt deze gewist)
5. **Verwerk je eerste dataset** — `chloros-cli process ~/datasets/flight001`

Raadpleeg voor NVIDIA Jetson de speciale [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) voor platformspecifieke installatie, thermisch gedrag en implementatie in de praktijk.

***

## Volgende stappen

* [Installatie op Linux](linux-installation.md) — gedetailleerde installatie, bestandslocaties en probleemoplossing voor amd64 en arm64
* [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) — Jetson-specifieke configuratie, geheugen- en thermisch gedrag, implementatie in de praktijk
* [CLI: opdrachtregel](../CLI.md) — de CLI-handleiding
* [API: Python SDK](../api-python-sdk.md) — de SDK-handleiding
* [CLI-referentie](../reference/cli-reference.md) en [SDK-referentie](../reference/sdk-reference.md) — uitgebreide lijsten met commando’s en API’s voor versie 1.3.0
* [Dynamische rekenkracht-aanpassing](../processing-architecture/dynamic-compute-adaptation.md) — hoe Chloroszich aanpast aan je hardware

{% hint style="info" %}
**Deze handleiding programmatisch lezen.** Elke pagina wordt ook als onbewerkte Markdown aangeboden via een eigen URL plus `.md` (bijvoorbeeld `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), en een index van de volledige handleiding is gepubliceerd op [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
