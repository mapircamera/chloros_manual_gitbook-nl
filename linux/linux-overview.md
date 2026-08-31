# Linux Overzicht

Chloros 1.2.0 biedt native Linux-ondersteuning voor de **CLI**en**Python SDK** — headless multispectrale beeldverwerking, plus live aansturing van LATTICE-camera’s en DAQ-lichtsensoren — op Linux-werkstations, servers en NVIDIA Jetson-edge-apparaten.

{% hint style="info" %}
**Geen desktop-GUI op Linux.**De Chloros-desktop-GUI is uitsluitend beschikbaar op Windows. Linux-gebruikers communiceren met Chloros via de [CLI](../CLI.md) en [Python SDK](../api-python-sdk.md). De `.deb` voegt inderdaad een**Chloros CLI** toevoeging aan je applicatiemenu — het opent gewoon een terminalemulator waarop `chloros-cli` draait.
{% endhint %}

***

## Platformondersteuningsmatrix

| Functie | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop-GUI** | Ja | n.v.t. | Nee | Nee |
| **CLI** (`chloros-cli`) | Ja | Ja | Ja | Ja |
| **Python SDK** (`chloros-sdk`) | Ja | Ja | Ja | Ja |
| **Beeldverwerkingspijplijn** | Ja | Ja | Ja | Ja |
| **LATTICE-camerabesturing (live)** | Ja (tabblad Camera&#x27;s) | Ja (`chloros-cli lattice`, SDK) | Ja | Ja |
| **DAQ-lichtsensoren (live)** | Ja (tabblad &#x27;Lichtsensoren&#x27;) | Ja (`chloros-cli daq pool-*`, SDK) | Ja | Ja |
| **PTP-tijdsynchronisatie (host is grandmaster)** | Ja | Ja (`chloros-cli time-sync`) | Ja | Ja |
| **GPU-versnelling (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Texture Aware debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamische rekenaanpassing** | Ja | Ja | Ja | Ja |
| **Backend als systeemservice** (`chloros-backend.service`) | Nee | Nee | Ja (opt-in) | Ja (opt-in) |
| **In-place updater** (`chloros-cli update`) | Nee (voer het installatieprogramma uit) | Nee (voer het installatieprogramma uit) | Ja | Ja |***

## Ondersteunde architecturen

| Architectuur | Beschrijving | Pakket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standaard desktop-/serverprocessors (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-processors — NVIDIA Jetson Orin-serie | `chloros_<version>_arm64_jp6.deb` (JetPack 6-build) |

## Ondersteunde Linux-distributies

* **Ubuntu 22.04 LTS of nieuwer** (amd64)
* **Debian 12 of nieuwer** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin-platforms)***

## Wat Linux-gebruikers krijgen

* **Chloros CLI** — de volledige opdrachtregelinterface voor batchverwerking, automatisering en scripting
* **Chloros Python SDK** — programmatische Python-interface voor onderzoekspijplijnen en aangepaste tools (te installeren via PyPI, en ook meegeleverd in de `.deb` als een wheel met bijpassende versie)
* **LATTICE-camerabesturing** — LATTICE-camera’s en gesynchroniseerde multi-camera-opstellingen detecteren, aansluiten, configureren en beelden vastleggen via `chloros-cli lattice` en de SDK; de `.deb` bevat de Arena SDK-runtime die de camera’s nodig hebben
* **DAQ-lichtsensorbesturing** — sluit DAQ-U/M/E-sensoren aan, stream gekalibreerde spectra en neem `.daq`-bestanden op via `chloros-cli daq pool-*` en de SDK
* **PTP-tijdsynchronisatie** — de Chloros-backend draait de PTP-grandmaster waaraan LATTICE-camera’s en DAQ-E-sensoren zijn gekoppeld; controleer deze met `chloros-cli time-sync`, en laat deze headless draaien met de systemd-unit `chloros-backend.service` (zie [Linux-installatie](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projectautomatisering** — voer opgeslagen projecten headless uit met `chloros-cli project` en de `open_project` van de SDK
* **GPU-versnelling** — CUDA-versnelde verwerking op NVIDIA-GPU&#x27;s (desktop en Jetson)
* **Dynamic Compute Adaptation** — automatische hardware-detectie en selectie van verwerkingsstrategieën, met de `CHLOROS_STRATEGY`-override als uitweg voor experts
* **Alle verwerkingsfuncties** — dezelfde pijplijn als Windows: kalibratie, vignetteringscorrectie, vegetatie-indexen en alle exportformaten
* **Chloros+-functies** — multithreaded (pipelined) verwerking, Texture Aware debayer en aangepaste indices, met een betaald Chloros+-abonnement

## Wat Linux-gebruikers niet krijgen

* **Desktop-GUI** — geen grafische interface; alle interactie verloopt via CLI of Python SDK
* **Beeldviewer** — geen interactieve beeldviewer, rasterweergave of kaartmarkeringen
* **Visueel projectbeheer** — projecten worden aangemaakt en aangestuurd via CLI-commando’s en SDK-aanroepen (de hardware zelf — camera’s, sensoren, opname — blijft volledig bedienbaar vanaf de terminal)***

## Licentievereisten

Toegang tot CLI en SDK vereist een **betaald Chloros+-abonnement — Copper of hoger**(Copper, Bronze, Silver, Gold). Het gratis**Iron**-abonnement biedt geen toegang tot CLI/SDK. Deze minimumvereiste wordt afgedwongen door de backend, niet alleen door de CLI:

| Situatie | Reactie van de backend |
| --- | --- |
| Niet ingelogd | `401` met `error_code: AUTH_REQUIRED` |
| Ingelogd op het gratis Iron-niveau | `403` met `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` werkt op elk niveau — het is de enige route die is vrijgesteld van de gate — dus de reden voor een weigering is altijd zichtbaar.

***

## Aan de slag met Linux

1. **Installeer Chloros** — zie [Linux-installatie](linux-installation.md) voor de installatie van `.deb`
2. **Controleer** — `chloros-cli --version` drukt `Chloros CLI 1.2.0` af; `chloros-cli selftest` voert de 7-stappen-diagnose uit
3. **Installeer de Python en SDK** (optioneel) — `pip install chloros-sdk`
4. **Meld u aan** — `chloros-cli login your@email.com 'your-password'` (eenmaal per machine, en opnieuw na elke pakketupgrade)
5. **Verwerk uw eerste dataset** — `chloros-cli process ~/datasets/flight001`

Raadpleeg voor NVIDIA Jetson de speciale [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) voor platformspecifieke installatie, thermisch gedrag en implementatie in de praktijk.

***

## Volgende stappen

* [Linux Installatie](linux-installation.md) — gedetailleerde installatie, bestandslocaties en probleemoplossing voor amd64 en arm64
* [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) — Jetson-specifieke configuratie, geheugen- en thermisch gedrag, implementatie in de praktijk
* [CLI : Opdrachtregel](../CLI.md) — de CLI-handleiding
* [API : Python SDK](../api-python-sdk.md) — de SDK-handleiding
* [CLI-referentie](../reference/cli-reference.md) en [SDK-referentie](../reference/sdk-reference.md) — uitgebreide lijst met commando’s/API voor versie 1.2.0
* [Dynamische rekenkracht-aanpassing](../processing-architecture/dynamic-compute-adaptation.md) — hoe Chloros zich aanpast aan je hardware

{% hint style="info" %}
**Deze handleiding programmatisch lezen.** Elke pagina wordt ook als ruwe Markdown aangeboden op de eigen URL plus `.md` (bijvoorbeeld `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), en er is een index van de gehele handleiding gepubliceerd op [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
