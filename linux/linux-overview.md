# Linux Overzicht

Chloros 1.1.0 biedt native Linux-ondersteuning voor de **CLI**en**Python SDK**, waardoor headless multispectrale beeldverwerking mogelijk wordt op Linux-werkstations, servers en NVIDIA Jetson edge-apparaten.

{% hint style="info" %}
**Geen GUI op Linux.** De Chloros Desktop GUI is alleen beschikbaar op Windows. Linux-gebruikers communiceren met Chloros via de [CLI](../CLI.md) en [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Platformondersteuningsmatrix

| Functie | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop-GUI** | Ja | N.v.t. | Nee | Nee |
| **CLI** | Ja | Ja | Ja | Ja |
| **Python SDK** | Ja | Ja | Ja | Ja |
| **GPU-versnelling (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Textuurgevoelige debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamische rekenaanpassing** | Ja | Ja | Ja | Ja |***

## Ondersteunde architecturen

| Architectuur | Beschrijving | Installatiemethode |
| --- | --- | --- |
| **amd64 (x86_64)** | Standaard desktop-/serverprocessors (Intel, AMD) | `.deb`-pakket |
| **arm64 (aarch64)** | ARM-gebaseerde processors, voornamelijk NVIDIA Jetson | `.deb`-pakket (JetPack 6) |

## Ondersteunde Linux-distributies

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson-platforms)***

## Wat Linux-gebruikers krijgen

* **Chloros CLI** — Volledige opdrachtregelinterface voor batchverwerking, automatisering en scripting
* **Chloros Python SDK** — Programmatische Python-interface (`pip install chloros-sdk`) voor integratie in onderzoekspijplijnen en aangepaste tools
* **GPU-versnelling** — CUDA-versnelde verwerking op NVIDIA GPU&#x27;s (desktop en Jetson)
* **Dynamische rekenaanpassing** — Automatische hardware-detectie en optimalisatie van de verwerkingsstrategie
* **Alle verwerkingsfuncties** — Dezelfde multispectrale verwerkingspijplijn als Windows (kalibratie, vignetteringscorrectie, vegetatie-indexen, alle exportformaten)
* **Chloros+-functies** — Multithreaded verwerking, Texture Aware debayer, aangepaste indices (met Chloros+-licentie)

## Wat Linux-gebruikers niet krijgen

* **Desktop-GUI** — Geen grafische interface; alle interactie verloopt via CLI of Python SDK
* **Afbeeldingsviewer** — Geen interactieve afbeeldingsviewer, rasterweergave of kaartmarkeringen
* **Visueel projectbeheer** — Projecten worden beheerd via CLI-opdrachten en SDK-aanroepen***

## Aan de slag met Linux

1. **Installeer Chloros** — Zie [Linux Installatie](linux-installation.md) voor de installatie van het `.deb`-pakket
2. **Installeer de Python SDK** (optioneel) — `pip install chloros-sdk`
3. **Activeer uw licentie** — `chloros-cli login your@email.com 'password'`
4. **Verwerk uw eerste dataset** — `chloros-cli process ~/datasets/flight001`

Gebruikers van NVIDIA Jetson kunnen de speciale [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) raadplegen voor platformspecifieke installatie en optimalisatie.

***

## Volgende stappen

* [Linux Installatie](linux-installation.md) — Gedetailleerde installatie-instructies voor amd64 en arm64
* [NVIDIA Jetson-handleiding](nvidia-jetson-guide.md) — Jetson-specifieke configuratie, thermisch beheer en implementatie in de praktijk
* [CLI : Opdrachtregel](../CLI.md) — Volledige CLI-referentie
* [API : Python SDK](../api-python-sdk.md) — Volledige SDK-referentie
* [Dynamische rekenaanpassing](../processing-architecture/dynamic-compute-adaptation.md) — Hoe Chloros zich aanpast aan uw hardware
