# De verwerking starten

Zodra je je afbeeldingen hebt geïmporteerd, je kalibratiedoelen hebt gemarkeerd en je projectinstellingen hebt geconfigureerd, ben je klaar om met de verwerking te beginnen. Deze pagina helpt je bij het opstarten van de Chloros-verwerkingspijplijn.

## Checklist voor de voorbewerking

Controleer, voordat je op de knop ‘Start’ klikt, of alles klaar is:

* [ ] **Bestanden geïmporteerd** - Alle afbeeldingen verschijnen in de bestandsbrowser
* [ ] **Doelafbeeldingen gemarkeerd** - Kolom ‘Doel’ gecontroleerd op kalibratieafbeeldingen (of een `.daq`-opname geïmporteerd voor LATTICE)
* [ ] **Cameramodellen gedetecteerd** – De kolom ‘Cameramodel’ toont de juiste camera’s
* [ ] **Instellingen geconfigureerd** – Projectinstellingen gecontroleerd en aangepast
* [ ] **Indices geselecteerd** – Gewenste multispectrale indices toegevoegd (indien nodig)
* [ ] **Exportformaat gekozen** - Uitvoerformaat dat geschikt is voor uw workflow

{% hint style="info" %}
**Tip**: Klik door een paar afbeeldingen in de bestandsbrowser om te controleren of ze correct zijn geladen voordat u ze verwerkt.
{% endhint %}

***

## De verwerking starten

### Zoek de startknop

De start-/afspeelknop bevindt zich in de bovenste koptekstbalk van Chloros:

* Positie: bovenaan in het midden van het venster
* Pictogram: **Afspeel-/startknop** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Status: de knop is ingeschakeld (licht) wanneer het systeem klaar is voor verwerking

### Klik om te starten

1. Klik op de **Afspeel-/Startknop** in de bovenste koptekstbalk
2. De verwerking begint onmiddellijk
3. Tijdens de verwerking verandert de knop in een **Stop**-knop
4. De voortgangsbalk wordt bijgewerkt en geeft de verwerkingsstatus weer

{% hint style="success" %}
**Verwerking gestart**: Zodra u klikt, voert Chloros automatisch alle verwerkingsstappen uit – doelherkenning, debayering, kalibratie, indexberekening en export. Het programma detecteert automatisch of uw project Survey3, LATTICE of een combinatie daarvan is, en past voor elke camera de juiste verwerkingspijplijn toe.
{% endhint %}

***

## Inzicht in verwerkingsmodi

Chloros werkt in twee verschillende verwerkingsmodi, afhankelijk van uw licentie:

### Gratis modus (sequentiële verwerking)

**Beschikbaar voor alle gebruikers**

**Hoe het werkt:**

* Verwerkt beelden één voor één, sequentieel
* Single-threaded werking
* Lager geheugengebruik

**De voortgangsbalk toont 2 fasen:**

1.**Doeldetectie** – Scannen naar kalibratiedoelen
2. **Verwerking** – Kalibratie toepassen en beelden exporteren**Verwerkingstijd:**

* Veel langzamer dan de parallelle modus van Chloros+
* Geschikt voor kleine tot middelgrote datasets (< 200 afbeeldingen)

### Chloros+-modus (parallelle verwerking)

**Vereist een Chloros+-licentie**

**Hoe het werkt:**

* Verwerkt meerdere afbeeldingen tegelijkertijd met behulp van een [verwerkingspijplijn met 4 threads](../processing-architecture/processing-pipeline.md)
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) selecteert automatisch de optimale strategie voor uw hardware bij het starten van het programma
* GPU (CUDA)-versnelling met NVIDIA-grafische kaarten
* **Het aantal workers past zich aan de hardware aan**: GPU-strategieën draaien**2-4 gelijktijdige workers** (geschaald op basis van VRAM — 3 op een GPU van 2-12 GB, 2 tot 4 op een GPU van 12 GB of meer); Systemen met alleen een CPU draaien één worker per fysieke kern, minus één**De voortgangsbalk toont 4 fasen** (die overeenkomen met de 4 pijplijn-threads):

1. **Detecteren** (Thread 1) – Kalibratiedoelen zoeken
2. **Analyseren** (thread 2) – Beeldmetadata onderzoeken en kalibratie berekenen
3. **Kalibreren** (thread 3) – Debayering, vignetteringscorrectie, kalibratie, indexberekening
4. **Exporteren** (thread 4) – Bewerkte afbeeldingen en indexen opslaan**Interactie met de voortgangsbalk:*** **Beweeg de muis** over de balk om het gedetailleerde dropdown-paneel met de 4 fasen te zien
* **Klik** op de voortgangsbalk om het dropdown-paneel vast te zetten
* **Klik nogmaals** om het paneel weer vrij te geven en te verbergen**Verwerkingstijd:**

* Aanzienlijk sneller dan de gratis modus
* GPU-versnelling verhoogt de snelheid nog verder

{% hint style="info" %}
**Chloros+ Snelheid**: Parallelle verwerking kan bij grote datasets 5-10 keer sneller zijn dan de sequentiële modus. Een project met 500 afbeeldingen dat in de gratis modus 2 uur duurt, kan met Chloros+ in 15-20 minuten worden voltooid.
{% endhint %}

***

## Wat gebeurt er tijdens de verwerking?

### Fase 1: Doeldetectie

**Wat Chloros doet:**

* Scant de afbeeldingen die u in de kolom ‘Doel’ hebt aangevinkt (alle afbeeldingen als er geen zijn aangevinkt)
* Identificeert de kalibratiepanelen in elk doel
* Haalt reflectiewaarden uit de doelpanelen
* Registreert tijdstempels van de doelen voor het plannen van kalibraties

**Duur:** 1-30 seconden (met gemarkeerde doelen), 5-30+ minuten (ongemarkeerd)

### Fase 2: Debayering (RAW-conversie)

**Wat Chloros doet:**

* Converteert RAW-gegevens in Bayer-patroon naar volledige 3-kanaals beelden (LATTICE mono-modules blijven enkelbandig — debayering wordt hiervoor overgeslagen met een opmerking in het logboek)
* Past het geselecteerde demosaicing-algoritme toe
* Behoudt maximale beeldkwaliteit en detail

**Duur:** Varieert afhankelijk van het aantal beelden en de snelheid van de CPU/GPU

### Fase 3: Kalibratie

**Wat Chloros doet:*** **Vignetteringscorrectie**: Verwijdert lensverduistering aan de randen
* **Reflectanciekalibratie**: Normaliseert aan de hand van doelreflectantiewaarden en/of DAQ-downwellinggegevens
* Past correcties toe op alle banden/kanalen
* Gebruikt voor elke afbeelding de juiste kalibratierferentie op basis van de tijdstempel

**Duur:** Het grootste deel van de verwerkingstijd

### Fase 4: Indexberekening

**Wat Chloros doet:**

* Berekent geconfigureerde multispectrale indices (NDVI, NDRE, enz.)
* Past bandberekeningen toe op gekalibreerde beelden
* Genereert indexbeelden voor elke geselecteerde index

**Duur:** Enkele seconden per beeld

### Fase 5: Exporteren

**Wat Chloros doet:**

* Slaat verwerkte beelden op in het geselecteerde formaat
* **LATTICE fan-out**: elk onbewerkt LATTICE-frame wordt in één keer geëxporteerd als elk ingeschakeld product — debayered, preview, radiance (altijd float32), reflectance
* Schrijft bestanden naar de projectuitvoerstructuur: `<project>/<camera>/<format>/<Product>_Images/`
* **Behoudt de bronbestandsnaam** — de map identificeert het product, er wordt geen achtervoegsel toegevoegd**Duur:** Varieert afhankelijk van het exportformaat en de bestandsgrootte***

## Verwerkingsgedrag

### Automatische verwerkingspijplijn

Eenmaal gestart, verloopt de gehele pijplijn automatisch:

* Geen gebruikersinteractie nodig
* Alle geconfigureerde stappen worden achtereenvolgens uitgevoerd
* Voortgangsupdates worden in realtime weergegeven
* Geëxporteerde bestanden worden naar de schijf geschreven zodra ze voltooid zijn — je kunt voltooide uitvoerbestanden openen terwijl de verwerking doorgaat

### Computergebruik tijdens de verwerking

**Vrije modus:**

* Relatief laag CPU-gebruik (single-threaded)
* De computer blijft responsief voor andere taken
* Het is veilig om Chloros te minimaliseren en in andere applicaties te werken

**Chloros+ parallelle modus:**

* Hoog CPU-gebruik binnen de workerpool van de strategie
* Met GPU-versnelling: hoog GPU-gebruik
* De computer reageert mogelijk minder snel tijdens de verwerking
* Start geen andere CPU-intensieve taken

{% hint style="warning" %}
**Prestatietip**: Sluit voor de beste prestaties van Chloros+ andere applicaties af en laat Chloros de volledige systeembronnen gebruiken.
{% endhint %}

### De verwerking kan niet worden gepauzeerd (maar het stoppen verloopt netjes)

* Eenmaal gestart, kan de verwerking niet worden gepauzeerd en later hervat
* Als je op **Stop** klikt, wordt de run bij de eerste klik netjes gestopt
* Producten die al vóór het stoppen zijn geëxporteerd, blijven op de schijf staan
* Een gestopte run rapporteert eerlijk wat er is voltooid (zie de `[RUN-SUMMARY]`-regels in het logboek)
* Bij een nieuwe run begint de pijplijn helemaal opnieuw

**Planningsadvies:** Overweeg bij zeer grote projecten om de verwerking in batches uit te voeren of de CLI te gebruiken voor meer controle.***

## Uw verwerking volgen

Terwijl de verwerking loopt, kunt u:

* **De voortgangsbalk bekijken** – Bekijk het totale voltooiingspercentage
* **De huidige fase bekijken** – Detecteren, Analyseren, Kalibreren of Exporteren
* **Het tabblad ‘Log’ controleren** – Bekijk gedetailleerde verwerkingsberichten en waarschuwingen
* **Een voorbeeld van voltooide afbeeldingen bekijken** – Geëxporteerde bestanden verschijnen tijdens de verwerking op de schijf

Zie [De verwerking controleren](monitoring-the-processing.md) voor gedetailleerde informatie over het controleren van de verwerking.

***

## De verwerking stoppen

Als u de verwerking moet stoppen:

### Hoe u de verwerking stopt

1. Zoek de **Stop-knop**(vervangt de knop**Start** tijdens de verwerking)
2. Klik er één keer op — de balk geeft **"Stoppen..."** weer terwijl de lopende afbeelding wordt voltooid
3. De verwerking eindigt in een definitieve gestopte toestand en het logboek geeft een eerlijk overzicht (`[RUN-SUMMARY]`) weer van wat is voltooid

### Wanneer stoppen

**Geldige redenen om te stoppen:**

* Je realiseert je dat er onjuiste instellingen zijn gebruikt
* Je bent vergeten doelafbeeldingen te markeren
* Er zijn verkeerde afbeeldingen geïmporteerd
* Het systeem werkt te traag of reageert niet

**Na het stoppen:**

* Producten die vóór het stoppen zijn geëxporteerd, blijven op de schijf staan
* Controleer en los eventuele problemen op, pas de instellingen indien nodig aan
* Start de verwerking opnieuw — de run begint vanaf het begin

***

## Geschatte verwerkingstijd

De werkelijke verwerkingstijd varieert sterk, afhankelijk van:

* Aantal afbeeldingen
* Afbeeldingsresolutie
* RAW- versus JPG-invoerformaat
* Verwerkingsmodus (Free versus Chloros+)
* CPU-snelheid en aantal cores
* Beschikbaarheid van GPU (alleen Chloros+)
* Aantal te berekenen indexen
* Aantal ingeschakelde exportproducten (LATTICE)

### Ruwe schattingen (Chloros+, afbeeldingen van 12 MP, moderne CPU)

| Aantal afbeeldingen | Gratis modus | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 afbeeldingen   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 afbeeldingen  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 afbeeldingen  | 1-1,5 uur | 20-30 min      | 10-15 min      |
| 500 afbeeldingen  | 2-3 uur   | 45-60 min      | 20-30 min      |
| 1000 afbeeldingen | 4-6 uur   | 1,5-2 uur      | 40-60 min      |

{% hint style="info" %}
**Eerste keer**: De eerste verwerking kan langer duren omdat Chloros caches en profielen opbouwt. De verwerking van vergelijkbare datasets zal daarna sneller verlopen.
{% endhint %}

***

## Veelvoorkomende problemen bij het opstarten

### Startknop uitgeschakeld (grijs weergegeven)

**Mogelijke oorzaken:**

* Er zijn geen afbeeldingen geïmporteerd
* De backend is nog niet volledig opgestart
* De vorige verwerking is nog bezig
* Het project is nog niet volledig geladen

**Oplossingen:**

1. Wacht tot de backend volledig is geïnitialiseerd (controleer het pictogram in het hoofdmenu)
2. Controleer of de afbeeldingen zijn geïmporteerd in de bestandsbrowser
3. Start Chloros opnieuw op als de knop nog steeds uitgeschakeld is
4. Controleer het foutlogboek op foutmeldingen

### Verwerking start maar mislukt onmiddellijk

**Mogelijke oorzaken:**

* Geen geldige afbeeldingen in het project
* Beschadigde afbeeldingsbestanden
* Onvoldoende schijfruimte
* Onvoldoende geheugen (RAM)

**Oplossingen:**

1. Controleer het foutlogboek <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> op foutmeldingen
2. Controleer of er voldoende schijfruimte beschikbaar is
3. Probeer een kleinere subset afbeeldingen te verwerken
4. Controleer of de afbeeldingen niet beschadigd zijn

### De run is voltooid, maar er zijn geen afbeeldingen geschreven

Een run waarbij afbeeldingsproducten zijn aangevraagd maar geen enkele is geschreven, wordt behandeld als een **mislukking, niet als een succes** — Chloros meldt dit duidelijk:

* Het GUI-logboek geeft de meldingen `[RUN-SUMMARY]` weer, die de waarschijnlijke oorzaak aangeven — geen afbeeldingen geïmporteerd, geen doel gedetecteerd, of elk aangevraagd product overgeslagen omdat het niet van toepassing is (bijv. het aanvragen van straling/reflectie van camera’s die alleen RGB ondersteunen)
* Het CLI-equivalent (`chloros-cli process`) geeft `Processing finished but wrote no image products.` weer en **stopt met een waarde anders dan nul**, zodat scripts dit kunnen detecteren
* Een opzettelijke run met alleen metadata (alle exportproducten uitgeschakeld, geen indexen) telt nog steeds als succes

Zie [de CLI-referentie](../reference/cli-reference.md#a-run-that-writes-no-images-fails) voor de volledige semantiek.

### De projectmap is te diep voor deze computer

**Alleen Windows, nieuw in 1.3.0.** Windows wordt geleverd met lange paden uitgeschakeld, waardoor geen enkel bestandspad langer mag zijn dan 260 tekens. Chloros-producten bevinden zich tot ongeveer 130 tekens onder de projectmap, dus een projectmap die dieper ligt dan dat, kan de bijbehorende producten niet ontvangen. In plaats van de run uit te voeren en niets te schrijven, controleert Chloros nu voordat de verwerking begint:

* Als je op Start klikt, wordt er geen run gestart: de voortgangsbalk geeft de reden weer en in het foutlogboek wordt `[PROCESS] refused: The project folder is too deep for this computer…` weergegeven. Het bericht vermeldt de lengte van de projectmap en de langste projectmap die wel past.
* Het aanmaken of openen van een project in een map die zelfs voor de eigen bestanden van het project te diep ligt, wordt eveneens geweigerd. Een berichtvenster legt uit waarom, met vermelding van de langste projectmap die past en hoe je lange paden kunt inschakelen, en er wordt niets aangemaakt of geopend; het foutlogboek registreert dit als `[PROJECT] refused: …`. Het dupliceren van een project onder een langere naam kan eveneens de limiet overschrijden: de kopie wordt gemaakt maar niet geopend, en hetzelfde bericht geeft de reden aan.

**Oplossingen:**

1. Bewaar projecten in een map met een kortere naam — wijzig de**werkmap** in [Projectinstellingen](../project-settings/project-settings.md) en maak het project opnieuw aan, of
2. Schakel lange paden in Windows in en start Chloros opnieuw op: voer het Chloros-installatieprogramma opnieuw uit met het vinkje bij **Enable Windows long paths** (dit is standaard aangevinkt), of gebruik Groepsbeleid: *Computerconfiguratie > Beheersjablonen > Systeem > Bestandssysteem > Win32-lange paden inschakelen*.

### Waarschuwing „Geen doelen gedetecteerd“

**Mogelijke oorzaken:**

* Vergeten doelafbeeldingen te markeren
* Doelafbeeldingen bevatten geen zichtbare doelen
* Instellingen voor doelherkenning zijn te streng

**Oplossingen:**

1. Raadpleeg [Doelafbeeldingen kiezen](choosing-target-images.md)
2. Markeer de juiste afbeeldingen in de kolom 'Doel'
3. Controleer of de doelen zichtbaar zijn in de gemarkeerde afbeeldingen
4. Pas indien nodig de instellingen voor doeldetectie aan

***

## Tips voor een succesvolle verwerking

### Voordat u begint

1. **Test eerst met een kleine subset** - Verwerk 10-20 afbeeldingen om de instellingen te controleren
2. **Controleer de beschikbare schijfruimte** - Zorg voor 2-3 keer de grootte van de dataset aan vrije ruimte (meer als alle LATTICE-producten zijn ingeschakeld)
3. **Sluit overbodige applicaties** – Maak systeembronnen vrij
4. **Controleer de doelafbeeldingen** – Bekijk een voorbeeld van de gemarkeerde doelen om de kwaliteit te waarborgen
5. **Sla het project op** – Het project wordt automatisch opgeslagen, maar het is aan te raden om het handmatig op te slaan

### Tijdens de verwerking

1. **Voorkom dat het systeem in de slaapstand gaat** - Schakel energiebesparingsmodi uit
2. **Houd Chloros op de voorgrond** - Of zorg in ieder geval dat het zichtbaar is in de taakbalk
3. **Controleer af en toe de voortgang** - Controleer op waarschuwingen of fouten
4. **Laad geen andere zware applicaties** - Vooral niet in de parallelle modus van Chloros+

### Chloros+ GPU-versnelling

Als je NVIDIA GPU-versnelling gebruikt:

1. Werk de NVIDIA-stuurprogramma’s bij naar de nieuwste versie
2. Zorg ervoor dat de GPU 4 GB of meer VRAM heeft (7 GB of meer voor gelijktijdige Texture Aware-debayering)
3. Sluit GPU-intensieve toepassingen (games, videobewerking)
4. Houd de GPU-temperatuur in de gaten (zorg voor voldoende koeling)

***

## Volgende stappen

Zodra de verwerking is gestart:

1. **Houd de voortgang in de gaten** - Zie [De verwerking volgen](monitoring-the-processing.md)
2. **Wacht tot het proces is voltooid** - De verwerking verloopt automatisch
3. **Bekijk de resultaten** – Zie [De verwerking afronden](finishing-the-processing.md)

Voor informatie over wat u tijdens de verwerking moet doen, zie [De verwerking volgen](monitoring-the-processing.md).
