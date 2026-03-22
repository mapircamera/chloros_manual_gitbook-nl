# De verwerking starten

Zodra u uw afbeeldingen hebt geïmporteerd, uw kalibratiedoelen hebt gemarkeerd en uw projectinstellingen hebt geconfigureerd, bent u klaar om met de verwerking te beginnen. Op deze pagina wordt uitgelegd hoe u de Chloros-verwerkingspijplijn start.

## Checklist voor de voorbewerking

Controleer voordat u op de Start-knop klikt of alles klaar is:

* [ ] **Bestanden geïmporteerd** - Alle afbeeldingen verschijnen in de bestandsbrowser
* [ ] **Doelafbeeldingen gemarkeerd** - Doelkolom gecontroleerd op kalibratieafbeeldingen
* [ ] **Cameramodellen gedetecteerd** - Kolom Cameramodel toont de juiste camera&#x27;s
* [ ] **Instellingen geconfigureerd** - Projectinstellingen gecontroleerd en aangepast
* [ ] **Indexen geselecteerd** - Gewenste multispectrale indexen toegevoegd (indien nodig)
* [ ] **Exportformaat gekozen** - Uitvoerformaat dat geschikt is voor uw workflow

{% hint style="info" %}
**Tip**: Klik door een paar afbeeldingen in de bestandsbrowser om te controleren of ze correct zijn geladen voordat u de verwerking start.
{% endhint %}

***

## De verwerking starten

### Zoek de startknop

De start-/afspeelknop bevindt zich in de bovenste balk van Chloros:

* Positie: bovenaan in het midden van het venster
* Pictogram: **Afspeel-/startknop** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Status: De knop is ingeschakeld (licht) wanneer deze klaar is voor verwerking

### Klik om te starten

1. Klik op de **Afspeel-/Startknop** in de bovenste balk
2. De verwerking begint onmiddellijk
3. De knop wordt uitgeschakeld (grijs) tijdens de verwerking
4. De voortgangsbalk wordt bijgewerkt en toont de verwerkingsstatus

{% hint style="success" %}
**Verwerking gestart**: Zodra u klikt, voert Chloros automatisch alle verwerkingsstappen uit: doelherkenning, debayering, kalibratie, indexberekening en export.
{% endhint %}

***

## Inzicht in verwerkingsmodi

Chloros werkt in twee verschillende verwerkingsmodi, afhankelijk van uw licentie:

### Gratis modus (sequentiële verwerking)

**Beschikbaar voor alle gebruikers**

**Hoe het werkt:**

* Verwerkt afbeeldingen één voor één, sequentieel
* Single-threaded werking
* Lager geheugengebruik

**Voortgangsbalk toont 2 fasen:**

1.**Doel detecteren** - Scannen naar kalibratiedoelen
2. **Verwerking** - Kalibratie toepassen en afbeeldingen exporteren**Verwerkingstijd:**

* Veel langzamer dan de parallelle modus van Chloros+
* Geschikt voor kleine tot middelgrote datasets (&lt; 200 afbeeldingen)

### Chloros+-modus (parallelle verwerking)

**Vereist Chloros+-licentie**

**Hoe het werkt:**

* Verwerkt meerdere afbeeldingen tegelijkertijd met behulp van een [4-thread verwerkingspijplijn](../processing-architecture/processing-pipeline.md)
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) selecteert automatisch de optimale strategie voor uw hardware
* GPU (CUDA)-versnelling met NVIDIA grafische kaarten (desktop en Jetson)
* Schaalbaar van een Jetson Nano (1 worker) tot een desktop met 12 GB+ GPU (3-4 workers)

**De voortgangsbalk toont 4 fasen** (overeenkomend met de 4 pijplijn-threads):

1. **Detecteren** (Thread 1) - Kalibratiedoelen vinden
2. **Analyseren** (Thread 2) - Beeldmetadata onderzoeken en kalibratie berekenen
3. **Kalibreren** (Thread 3) - GPU-debayering, vignetteringscorrectie, indexberekening
4. **Exporteren** (thread 4) - Bewerkte afbeeldingen en indexen opslaan**Interactie met de voortgangsbalk:*** **Beweeg de muis** over de balk om het gedetailleerde dropdown-paneel met 4 fasen te zien
* **Klik** op de voortgangsbalk om het dropdown-paneel vast te zetten
* **Klik nogmaals** om het paneel weer vrij te geven en te verbergen**Verwerkingstijd:**

* Aanzienlijk sneller dan de gratis modus
* Schaalbaar met het aantal CPU-kernen
* GPU-versnelling verbetert de snelheid nog verder

{% hint style="info" %}
**Chloros+ Snelheid**: Parallelle verwerking kan 5-10 keer sneller zijn dan de sequentiële modus voor grote datasets. Een project met 500 afbeeldingen dat in de gratis modus 2 uur duurt, kan met Chloros+ in 15-20 minuten worden voltooid.
{% endhint %}

***

## Wat gebeurt er tijdens de verwerking

### Fase 1: Doeldetectie

**Wat Chloros doet:**

* Scant gemarkeerde doelafbeeldingen (of alle afbeeldingen als er geen zijn gemarkeerd)
* Identificeert de 4 kalibratiepanelen in elk doel
* Haalt reflectiewaarden uit de doelpanelen
* Registreert tijdstempels van doelen voor kalibratieplanning

**Duur:** 1-30 seconden (met gemarkeerde doelen), 5-30+ minuten (ongemarkeerd)

### Fase 2: Debayering (RAW-conversie)

**Wat Chloros doet:**

* Converteert RAW-Bayer-patroongegevens naar volledige RGB-afbeeldingen
* Past een hoogwaardig demosaicing-algoritme toe
* Behoudt maximale beeldkwaliteit en detail

**Duur:** Varieert afhankelijk van het aantal afbeeldingen en de CPU-snelheid

### Fase 3: Kalibratie

**Wat Chloros doet:*** **Vignetteringscorrectie**: Verwijdert lensverduistering aan de randen
* **Reflectiekalibratie**: Normaliseert met behulp van doelreflectiewaarden
* Past correcties toe op alle banden/kanalen
* Gebruikt voor elke afbeelding het juiste kalibratiedoel op basis van de tijdstempel

**Duur:** Het grootste deel van de verwerkingstijd

### Fase 4: Indexberekening

**Wat Chloros doet:**

* Berekent geconfigureerde multispectrale indices (NDVI, NDRE, enz.)
* Past bandberekeningen toe op gekalibreerde beelden
* Genereert indexbeelden voor elke geselecteerde index

**Duur:** Enkele seconden per beeld

### Fase 5: Exporteren

**Wat Chloros doet:**

* Slaat gekalibreerde beelden op in het geselecteerde formaat
* Exporteert indexbeelden met geconfigureerde LUT-kleuren
* Schrijft bestanden naar submappen van cameramodellen
* Behoudt originele bestandsnamen met achtervoegsels

**Duur:** Varieert afhankelijk van exportformaat en bestandsgrootte***

## Verwerkingsgedrag

### Automatische verwerkingspijplijn

Eenmaal gestart, verloopt de gehele pijplijn automatisch:

* Geen gebruikersinteractie nodig
* Alle geconfigureerde stappen worden achtereenvolgens uitgevoerd
* Voortgangsupdates worden in realtime weergegeven

### Computergebruik tijdens verwerking

**Vrije modus:**

* Relatief laag CPU-gebruik (single-threaded)
* Computer blijft responsief voor andere taken
* Veilig om Chloros te minimaliseren en in andere applicaties te werken

**Chloros+ Parallelle modus:**

* Hoog CPU-gebruik (multi-threaded, tot 16 cores)
* Met GPU-versnelling: Hoog GPU-gebruik
* Computer reageert mogelijk minder snel tijdens de verwerking
* Vermijd het starten van andere CPU-intensieve taken

{% hint style="warning" %}
**Prestatietip**: Sluit voor de beste Chloros+ prestaties andere applicaties en laat Chloros de volledige systeembronnen gebruiken.
{% endhint %}

### Verwerking kan niet worden gepauzeerd

**Belangrijke beperkingen:**

* Eenmaal gestart, kan de verwerking niet worden gepauzeerd
* U kunt de verwerking annuleren, maar de voortgang gaat verloren
* Gedeeltelijke resultaten worden niet opgeslagen
* Moet opnieuw worden gestart vanaf het begin indien geannuleerd

**Planningsadvies:** Overweeg bij zeer grote projecten om in batches te verwerken of CLI te gebruiken voor betere controle.***

## Uw verwerking monitoren

Terwijl de verwerking loopt, kunt u:

* **De voortgangsbalk bekijken** - Bekijk het totale voltooiingspercentage
* **De huidige fase bekijken** - Detecteren, Analyseren, Kalibreren of Exporteren
* **Het tabblad Logboek controleren** - Bekijk gedetailleerde verwerkingsberichten en waarschuwingen
* **Een voorbeeld van voltooide afbeeldingen bekijken** - Sommige exportbestanden kunnen tijdens de verwerking verschijnen

Zie [De verwerking controleren](monitoring-the-processing.md) voor gedetailleerde informatie over het controleren.

***

## De verwerking annuleren

Als u de verwerking moet stoppen:

### Hoe te annuleren

1. Zoek de **knop Stoppen/Annuleren** (vervangt de knop Start tijdens de verwerking)
2. Klik op de knop Stoppen
3. De verwerking wordt onmiddellijk gestopt
4. Gedeeltelijke resultaten worden verwijderd

### Wanneer annuleren

**Geldige redenen om te annuleren:**

* U realiseert zich dat er onjuiste instellingen zijn gebruikt
* U bent vergeten de doelafbeeldingen te markeren
* Er zijn verkeerde afbeeldingen geïmporteerd
* Het systeem werkt te traag of reageert niet

**Na het annuleren:**

* Controleer en los eventuele problemen op
* Pas de instellingen indien nodig aan
* Start de verwerking opnieuw vanaf het begin
* Sluit Chloros volledig af en start opnieuw op voor de beste ervaring

{% hint style="warning" %}
**Geen gedeeltelijke resultaten**: Annuleren verwijdert alle voortgang. Chloros slaat gedeeltelijk verwerkte afbeeldingen niet op.
{% endhint %}

***

## Geschatte verwerkingstijd

De werkelijke verwerkingstijd varieert sterk op basis van:

* Aantal afbeeldingen
* Afbeeldingsresolutie
* RAW- versus JPG-invoerformaat
* Verwerkingsmodus (Free versus Chloros+)
* CPU-snelheid en aantal cores
* Beschikbaarheid van GPU (alleen Chloros+)
* Aantal te berekenen indexen
* Complexiteit van het exportformaat

### Ruwe schattingen (Chloros+, 12 MP-afbeeldingen, moderne CPU)

| Aantal afbeeldingen | Gratis modus | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 afbeeldingen   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 afbeeldingen  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 afbeeldingen  | 1-1,5 uur | 20-30 min      | 10-15 min      |
| 500 afbeeldingen  | 2-3 uur   | 45-60 min      | 20-30 min      |
| 1000 afbeeldingen | 4-6 uur   | 1,5-2 uur      | 40-60 min      |

{% hint style="info" %}
**Eerste keer**: De eerste verwerking kan langer duren omdat Chloros caches en profielen opbouwt. De verwerking van soortgelijke datasets daarna zal sneller verlopen.
{% endhint %}

***

## Veelvoorkomende problemen bij het opstarten

### Startknop uitgeschakeld (grijs)

**Mogelijke oorzaken:**

* Geen afbeeldingen geïmporteerd
* Backend niet volledig opgestart
* Vorige verwerking loopt nog
* Project niet volledig geladen

**Oplossingen:**

1. Wacht tot de backend volledig is geïnitialiseerd (controleer het pictogram in het hoofdmenu)
2. Controleer of de afbeeldingen zijn geïmporteerd in de bestandsbrowser
3. Start Chloros opnieuw op als de knop uitgeschakeld blijft
4. Controleer het foutlogboek op foutmeldingen

### Verwerking start en mislukt vervolgens onmiddellijk

**Mogelijke oorzaken:**

* Geen geldige afbeeldingen in het project
* Beschadigde afbeeldingsbestanden
* Onvoldoende schijfruimte
* Onvoldoende geheugen (RAM)

**Oplossingen:**

1. Controleer het foutlogboek <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> op foutmeldingen
2. Controleer de beschikbare schijfruimte
3. Probeer een kleinere subset van afbeeldingen te verwerken
4. Controleer of de afbeeldingen niet beschadigd zijn

### Waarschuwing &quot;Geen doelen gedetecteerd&quot;

**Mogelijke oorzaken:**

* Vergeten doelafbeeldingen te markeren
* Doelafbeeldingen bevatten geen zichtbare doelen
* Instellingen voor doelherkenning te streng

**Oplossingen:**

1. Bekijk [Doelafbeeldingen kiezen](choosing-target-images.md)
2. Markeer de juiste afbeeldingen in de kolom Doel
3. Controleer of doelen zichtbaar zijn in gemarkeerde afbeeldingen
4. Pas de instellingen voor doeldetectie indien nodig aan

***

## Tips voor een succesvolle verwerking

### Voordat u begint

1. **Test eerst met een kleine subset** - Verwerk 10-20 afbeeldingen om de instellingen te controleren
2. **Controleer de beschikbare schijfruimte** - Zorg voor 2-3x de grootte van de dataset aan vrije ruimte
3. **Sluit onnodige applicaties** - Maak systeembronnen vrij
4. **Controleer de doelafbeeldingen** - Bekijk een voorbeeld van gemarkeerde doelen om de kwaliteit te controleren
5. **Sla het project op** - Het project wordt automatisch opgeslagen, maar het is een goede gewoonte om het handmatig op te slaan

### Tijdens de verwerking

1. **Voorkom dat het systeem in de slaapstand gaat** - Schakel energiebesparingsmodi uit
2. **Houd Chloros op de voorgrond** - Of zorg er in ieder geval voor dat het zichtbaar is in de taakbalk
3. **Controleer af en toe de voortgang** - Controleer op waarschuwingen of fouten
4. **Laad geen andere zware applicaties** - Vooral niet in de parallelle modus van Chloros+

### Chloros+ GPU-versnelling

Als u NVIDIA GPU-versnelling gebruikt:

1. Werk de NVIDIA-stuurprogramma&#x27;s bij naar de nieuwste versie
2. Zorg ervoor dat de GPU 4 GB of meer VRAM heeft
3. Sluit GPU-intensieve toepassingen (games, videobewerking)
4. Houd de GPU-temperatuur in de gaten (zorg voor voldoende koeling)

***

## Volgende stappen

Zodra de verwerking is gestart:

1. **Houd de voortgang in de gaten** - Zie [De verwerking volgen](monitoring-the-processing.md)
2. **Wacht tot het proces is voltooid** - De verwerking verloopt automatisch
3. **Bekijk de resultaten** - Zie [De verwerking afronden](finishing-the-processing.md)

Voor informatie over wat u tijdens de verwerking moet doen, zie [De verwerking controleren](monitoring-the-processing.md).
