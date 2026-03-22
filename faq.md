---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Veelgestelde vragen

<details>

<summary>Kan ik met Chloros beelden verwerken van camera&#x27;s die niet van het merk MAPIR zijn?</summary>

Nee, Chloros ondersteunt alleen de verwerking van beelden van MAPIR-camera&#x27;s. Raadpleeg de lijst met [ondersteunde cameramodellen](supported-cameras.md) voor meer informatie. We bieden wel verwerking van andere camera&#x27;s aan op MAPIR Cloud, zie de volledige lijst [hier](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Kan ik mijn beelden kalibreren voor reflectie zonder een kalibratiedoel?</summary>

Nee. Zonder een beeld van het kalibratiedoel dat is vastgelegd op het moment dat de niet-doelbeelden worden vastgelegd, kunt u de pixelwaarden van het beeld niet relateren aan een bekend reflectiepercentage. Als u ook het logboek van een MAPIR-lichtsensor niet opneemt, wordt het omgevingslichtspectrum niet gemeten en zullen de reflectieresultaten niet nauwkeurig zijn.

</details>

<details>

<summary>Kan ik mijn afbeeldingen bewerken voordat ik ze verwerk in Chloros?</summary>

Nee. Chloros gaat ervan uit dat de invoergegevens niet zijn gewijzigd. Wijzig de bestandsnamen niet.

</details>

<details>

<summary>Kan ik mijn MAPIR en Survey3-camera&#x27;s instellen op automatische belichting en de beelden verwerken in Chloros?</summary>

Nee. Survey3-beelddatasets moeten een vaste/vergrendelde belichting hebben, dus geen automatische sluitertijd of automatische ISO. Alle beelden van hetzelfde cameramodel moeten een identieke sluitertijd en ISO (belichting) hebben.

</details>

<details>

<summary>Kan Chloros orthomosaïekbeelden verwerken of analyseren?</summary>

Nee. Alleen afzonderlijke MAPIR-camerabeelden worden ondersteund, geen samengevoegde beelden zoals een orthomosaïekkaart.

</details>

<details>

<summary>Hoe kan ik de doelherkenningsstap van Chloros versnellen?</summary>

Door in de tabel van de bestandsbrowser de doelafbeeldingen in de rechterkolom vooraf te selecteren, geeft u Chloros de opdracht om alleen in die afbeeldingen naar kalibratiedoelen te zoeken, wat de verwerking aanzienlijk versnelt.

</details>

<details>

<summary>Als ik mijn beelden ga uploaden naar <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> moet ik ze dan eerst verwerken in Chloros voordat ik ze upload?</summary>

Als u van plan bent om te uploaden naar ons online verwerkingsplatform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), bewerk de afbeeldingen dan niet voorafgaand aan het uploaden. Cloud voert dezelfde verwerking uit en nog veel meer.

</details>

<details>

<summary>Zal MAPIR ooit functie X ondersteunen? Ik zou heel graag willen dat MAPIR X aanbood.</summary>

We zijn altijd geïnteresseerd in feedback over onze producten. Als u een probleem met onze producten constateert, of een suggestie heeft over hoe we onze producten kunnen verbeteren, neem dan [CONTACT MET ONS OP](https://www.mapir.camera/community/contact) om uw mening te delen. Het grootste deel van onze R&amp;D is gericht op het luisteren naar de belangrijkste behoeften van onze klanten.

</details>

<details>

<summary>Is Chloros beschikbaar voor Linux?</summary>

Ja! Chloros 1.1.0 ondersteunt Linux amd64 (x86_64) en arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakketten. De CLI en Python SDK worden volledig ondersteund op Linux. Er is geen GUI voor Linux — alle interactie verloopt via de [CLI](CLI.md) of [Python SDK](api-python-sdk.md). Zie [Linux Overzicht](linux/linux-overview.md) voor details.

</details>

<details>

<summary>Kan ik Chloros op NVIDIA Jetson uitvoeren?</summary>

Ja! Chloros 1.1.0 ondersteunt NVIDIA Jetson-platforms, waaronder Jetson Nano, Orin Nano, Orin NX en AGX Orin met JetPack 6. Chloros detecteert automatisch uw Jetson-model en optimaliseert de verwerkingsstrategie. Raadpleeg de [NVIDIA Jetson-handleiding](linux/nvidia-jetson-guide.md) voor instructies over installatie en implementatie.

</details>

<details>

<summary>Wordt Chloros automatisch geoptimaliseerd voor mijn hardware?</summary>

Ja! Chloros 1.1.0 bevat [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) dat uw CPU, GPU, RAM en (op Jetson) thermische sensoren automatisch detecteert. Vervolgens selecteert het de optimale verwerkingsstrategie — van `GPU_PARALLEL` op systemen met veel geheugen tot `GPU_SINGLE` op apparaten met beperkte capaciteit en `CPU_PARALLEL` op systemen zonder NVIDIA GPU. Er is geen handmatige configuratie nodig.

</details>

<details>

<summary>Wat is de 4-thread verwerkingspijplijn?</summary>

Chloros 1.1.0 maakt gebruik van een 4-thread pipelined architectuur voor Chloros+ gebruikers: Thread 1 (Detectie) laadt afbeeldingen en detecteert kalibratiedoelen, Thread 2 (Kalibratie) berekent de reflectiekalibratie, Thread 3 (Verwerking) voert GPU-versnelde debayering en indexberekening uit, en Thread 4 (Exporteren) schrijft uitvoerbestanden. Meerdere afbeeldingen kunnen tegelijkertijd in verschillende threads worden verwerkt voor een maximale doorvoer. Zie [Verwerkingspijplijn](processing-architecture/processing-pipeline.md) voor meer informatie.

</details>

<details>

<summary>Hoe voer ik diagnostiek uit op mijn Chloros-installatie?</summary>

Gebruik het commando `selftest` om 7 systeemdiagnoses uit te voeren, waaronder versiecontrole, poortbeschikbaarheid, opstarten van de backend, API-connectiviteit, systeeminformatie, denoiser-modellen en CUDA-beschikbaarheid:

```bash
chloros-cli selftest
```

Dit is vooral handig op Linux/Jetson-systemen om de GPU- en CUDA-configuratie te controleren.

</details>
