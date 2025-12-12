# De verwerking starten

Nadat u uw afbeeldingen hebt geïmporteerd, uw kalibratiedoelen hebt gemarkeerd en uw projectinstellingen hebt geconfigureerd, bent u klaar om te beginnen met de verwerking. Op deze pagina wordt uitgelegd hoe u de Chloros-verwerkingspijplijn start.

## Checklist voorafgaand aan de verwerking

Controleer voordat u op de knop Start klikt of alles gereed is:

* [ ] **Bestanden geïmporteerd** - Alle afbeeldingen worden weergegeven in de bestandsbrowser
* [ ] **Doelafbeeldingen gemarkeerd** - Doelkolom gecontroleerd voor kalibratieafbeeldingen
* [ ] **Cameramodellen gedetecteerd** - De kolom Cameramodel toont de juiste camera&#x27;s
* [ ] **Instellingen geconfigureerd** - Projectinstellingen gecontroleerd en aangepast
* [ ] **Indexen geselecteerd** - Gewenste multispectrale indexen toegevoegd (indien nodig)
* [ ] **Exportformaat gekozen** - Uitvoerformaat dat geschikt is voor uw workflow

{% hint style=&quot;info&quot; %}
**Tip**: Klik door een paar afbeeldingen in de bestandsbrowser om te controleren of ze correct zijn geladen voordat u ze verwerkt.
{% endhint %}

***

## De verwerking starten

### Zoek de startknop

De knop Start/Afspelen bevindt zich in de bovenste balk van Chloros:

* Positie: midden bovenaan het venster
* Pictogram: **knop Afspelen/Start** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: De knop is ingeschakeld (helder) wanneer deze klaar is voor verwerking

### Klik om te starten

1. Klik op de **knop Afspelen/Starten** in de bovenste koptekst
2. De verwerking begint onmiddellijk
3. De knop wordt tijdens de verwerking uitgeschakeld (grijs)
4. De voortgangsbalk wordt bijgewerkt en toont de verwerkingsstatus

{% hint style=&quot;success&quot; %}
**Verwerking gestart**: Zodra u hierop klikt, voert Chloros automatisch alle verwerkingsstappen uit: doelherkenning, debayering, kalibratie, indexberekening en export.
{% endhint %}

***

## Verwerkingsmodi begrijpen

Chloros werkt in twee verschillende verwerkingsmodi, afhankelijk van uw licentie:

### Gratis modus (sequentiële verwerking)

**Beschikbaar voor alle gebruikers**

**Hoe het werkt:**

* Verwerkt afbeeldingen één voor één, sequentieel
* Single-threaded werking
* Lager geheugengebruik

**Voortgangsbalk toont 2 fasen:**

1. **Doeldetectie** - Scannen naar kalibratiedoelen
2. **Verwerking** - Kalibratie toepassen en afbeeldingen exporteren

**Verwerkingstijd:**

* Veel langzamer dan Chloros+ parallelle modus
* Geschikt voor kleine tot middelgrote datasets (&lt; 200 afbeeldingen)

### Chloros+ modus (parallelle verwerking)

**Vereist Chloros+ licentie**

**Hoe het werkt:**

* Verwerkt meerdere afbeeldingen tegelijkertijd
* Multi-threaded werking (maximaal 16 parallelle workers)
* Maakt gebruik van meerdere CPU-kernen
* Optionele GPU (CUDA)-versnelling met NVIDIA grafische kaarten

**Voortgangsbalk toont 4 fasen:**

1. **Detecteren** - Kalibratiedoelen zoeken
2. **Analyseren** - Beeldmetadata onderzoeken en pijplijn voorbereiden
3. **Kalibreren** - Correcties en kalibraties toepassen
4. **Exporteren** - Verwerkte afbeeldingen en indexen opslaan

**Interactie met voortgangsbalk:**

* **Beweeg de muis** over de balk om een gedetailleerd dropdown-paneel met 4 fasen te zien
* **Klik** op de voortgangsbalk om het dropdown-paneel op zijn plaats te vergrendelen
* **Klik nogmaals** om het paneel te ontgrendelen en te verbergen

**Verwerkingstijd:**

* Aanzienlijk sneller dan de vrije modus
* Schaalbaar met het aantal CPU-kernen
* GPU-versnelling verbetert de snelheid nog verder

{% hint style=&quot;info&quot; %}
**Chloros+ Snelheid**: Parallelle verwerking kan 5-10 keer sneller zijn dan de sequentiële modus voor grote datasets. Een project met 500 afbeeldingen dat in de gratis modus 2 uur duurt, kan met Chloros+ in 15-20 minuten worden voltooid.
{% endhint %}

***

## Wat gebeurt er tijdens de verwerking?

### Fase 1: Doeldetectie

**Wat Chloros doet:**

* Scant gemarkeerde doelafbeeldingen (of alle afbeeldingen als er geen zijn gemarkeerd)
* Identificeert de 4 kalibratiepanelen in elk doel
* Haalt reflectiewaarden uit doelpanelen
* Registreert tijdstempels van doelen voor kalibratieplanning

**Duur:** 1-30 seconden (met gemarkeerde doelen), 5-30+ minuten (ongemarkeerd)

### Fase 2: Debayering (RAW-conversie)

**Wat Chloros doet:**

* Converteert RAW-Bayer-patroongegevens naar volledige RGB-afbeeldingen
* Past een hoogwaardig demosaicing-algoritme toe
* Behoudt maximale beeldkwaliteit en details

**Duur:** varieert afhankelijk van het aantal afbeeldingen en de CPU-snelheid

### Fase 3: Kalibratie

**Wat Chloros doet:**

* **Vignettecorrectie**: verwijdert lensverduistering aan de randen
* **Reflectiekalibratie**: normaliseert met behulp van doelreflectiewaarden
* Past correcties toe op alle banden/kanalen
* Gebruikt het juiste kalibratiedoel voor elke afbeelding op basis van de tijdstempel

**Duur:** Het grootste deel van de verwerkingstijd

### Fase 4: Indexberekening

**Wat Chloros doet:**

* Berekent geconfigureerde multispectrale indices (NDVI, NDRE, enz.)
* Past bandwiskunde toe op gekalibreerde beelden
* Genereert indexbeelden voor elke geselecteerde index

**Duur:** Enkele seconden per beeld

### Fase 5: Exporteren

**Wat Chloros doet:**

* Slaat gekalibreerde beelden op in het geselecteerde formaat
* Exporteert indexbeelden met geconfigureerde LUT-kleuren
* Schrijft bestanden naar submappen van cameramodellen
* Behoudt originele bestandsnamen met achtervoegsels

**Duur:** Varieert per exportformaat en bestandsgrootte

***

## Verwerkingsgedrag

### Automatische verwerkingspijplijn

Eenmaal gestart, wordt de hele pijplijn automatisch uitgevoerd:

* Geen gebruikersinteractie nodig
* Alle geconfigureerde stappen worden achtereenvolgens uitgevoerd
* Voortgang wordt in realtime weergegeven

### Computergebruik tijdens verwerking

**Vrije modus:**

* Relatief laag CPU-gebruik (single-threaded)
* Computer blijft responsief voor andere taken
* Veilig om Chloros te minimaliseren en in andere toepassingen te werken

**Chloros+ Parallelle modus:**

* Hoog CPU-gebruik (multi-threaded, tot 16 cores)
* Met GPU-versnelling: hoog GPU-gebruik
* Computer reageert mogelijk minder snel tijdens verwerking
* Start geen andere CPU-intensieve taken

{% hint style=&quot;warning&quot; %}
**Prestatietip**: Sluit andere toepassingen en laat Chloros alle systeembronnen gebruiken voor de beste prestaties van Chloros+.
{% endhint %}

### Verwerking kan niet worden gepauzeerd

**Belangrijke beperkingen:**

* Eenmaal gestart, kan de verwerking niet worden gepauzeerd.
* U kunt de verwerking annuleren, maar de voortgang gaat verloren.
* Gedeeltelijke resultaten worden niet opgeslagen.
* Bij annulering moet u opnieuw beginnen.

**Planningstip:** Overweeg bij zeer grote projecten om in batches te verwerken of CLI te gebruiken voor een betere controle.

***

## Uw verwerking controleren

Terwijl de verwerking wordt uitgevoerd, kunt u:

* **De voortgangsbalk bekijken** - Bekijk het totale voltooiingspercentage
* **De huidige fase bekijken** - Detecteren, analyseren, kalibreren of exporteren
* **Het tabblad Logboek controleren** - Bekijk gedetailleerde verwerkingsberichten en waarschuwingen
* **Een voorbeeld van voltooide afbeeldingen bekijken** - Sommige exportbestanden kunnen tijdens de verwerking verschijnen

Zie [De verwerking controleren] voor gedetailleerde informatie over het controleren.(monitoring-the-processing.md).

***

## De verwerking annuleren

Als u de verwerking moet stoppen:

### Hoe annuleren

1. Zoek de **knop Stoppen/Annuleren** (vervangt de knop Starten tijdens de verwerking)
2. Klik op de knop Stoppen
3. De verwerking wordt onmiddellijk gestopt
4. Gedeeltelijke resultaten worden verwijderd

### Wanneer annuleren

**Geldige redenen om te annuleren:**

* U realiseert zich dat er onjuiste instellingen zijn gebruikt
* U bent vergeten doelafbeeldingen te markeren
* Er zijn verkeerde afbeeldingen geïmporteerd
* Het systeem werkt te traag of reageert niet

**Na het annuleren:**

* Controleer en los eventuele problemen op
* Pas de instellingen indien nodig aan
* Start de verwerking opnieuw vanaf het begin
* Sluit Chloros volledig af en start opnieuw op voor de beste ervaring

{% hint style=&quot;warning&quot; %}
**Geen gedeeltelijke resultaten**: bij annuleren wordt alle voortgang verwijderd. Chloros slaat gedeeltelijk verwerkte afbeeldingen niet op.
{% endhint %}

***

## Geschatte verwerkingstijd

De werkelijke verwerkingstijd varieert sterk, afhankelijk van:

* Aantal afbeeldingen
* Afbeeldingsresolutie
* RAW- versus JPG-invoerformaat
* Verwerkingsmodus (Free versus Chloros+)
* CPU-snelheid en aantal cores
* Beschikbaarheid van GPU (alleen Chloros+)
* Aantal te berekenen indices
* Complexiteit van het exportformaat

### Ruwe schattingen (Chloros+, 12 MP-afbeeldingen, moderne CPU)

| Aantal afbeeldingen | Gratis modus | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 afbeeldingen   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 afbeeldingen  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 afbeeldingen  | 1-1,5 uur | 20-30 min      | 10-15 min      |
| 500 afbeeldingen  | 2-3 uur   | 45-60 min      | 20-30 min      |
| 1000 afbeeldingen | 4-6 uur   | 1,5-2 uur      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Eerste keer**: De eerste verwerking kan langer duren omdat Chloros caches en profielen aanmaakt. De volgende verwerking van vergelijkbare datasets zal sneller gaan.
{% endhint %}

***

## Veelvoorkomende problemen bij het opstarten

### Startknop uitgeschakeld (grijs weergegeven)

**Mogelijke oorzaken:**

* Er zijn geen afbeeldingen geïmporteerd
* De backend is niet volledig opgestart
* Eerdere verwerking is nog bezig
* Het project is niet volledig geladen

**Oplossingen:**

1. Wacht tot de backend volledig is geïnitialiseerd (controleer het pictogram in het hoofdmenu)
2. Controleer of de afbeeldingen zijn geïmporteerd in de bestandsbrowser
3. Start Chloros opnieuw op als de knop nog steeds uitgeschakeld is
4. Controleer het foutlogboek op foutmeldingen

### Verwerking start en mislukt vervolgens onmiddellijk

**Mogelijke oorzaken:**

* Geen geldige afbeeldingen in project
* Beschadigde afbeeldingsbestanden
* Onvoldoende schijfruimte
* Onvoldoende geheugen (RAM)

**Oplossingen:**

1. Controleer het foutopsporingslogboek <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> op foutmeldingen
2. Controleer de beschikbare schijfruimte
3. Probeer een kleinere subset van afbeeldingen te verwerken
4. Controleer of de afbeeldingen niet beschadigd zijn

### Waarschuwing &quot;Geen doelen gedetecteerd&quot;

**Mogelijke oorzaken:**

* Vergeten doelafbeeldingen te markeren
* Doelafbeeldingen bevatten geen zichtbare doelen
* Instellingen voor doel detectie te streng

**Oplossingen:**

1. Bekijk [Doelafbeeldingen kiezen](choosing-target-images.md)
2. Markeer de juiste afbeeldingen in de kolom Doel
3. Controleer of de doelen zichtbaar zijn in de gemarkeerde afbeeldingen
4. Pas indien nodig de instellingen voor doelherkenning aan

***

## Tips voor een succesvolle verwerking

### Voordat u begint

1. **Test eerst met een kleine subset** - Verwerk 10-20 afbeeldingen om de instellingen te controleren
2. **Controleer de beschikbare schijfruimte** - Zorg ervoor dat er 2-3 keer de grootte van de dataset vrij is
3. **Sluit onnodige toepassingen** - Maak systeembronnen vrij
4. **Controleer de doelafbeeldingen** - Bekijk een voorbeeld van de gemarkeerde doelen om de kwaliteit te controleren
5. **Sla het project op** - Het project wordt automatisch opgeslagen, maar het is een goede gewoonte om het handmatig op te slaan.

### Tijdens de verwerking

1. **Voorkom dat het systeem in slaapstand gaat** - Schakel de energiebesparende modi uit.
2. **Houd Chloros op de voorgrond** - Of zorg ervoor dat het ten minste zichtbaar is in de taakbalk.
3. **Controleer af en toe de voortgang** - Controleer op waarschuwingen of fouten.
4. **Laad geen andere zware applicaties** - Vooral niet in de parallelle modus van Chloros+

### Chloros+ GPU-versnelling

Als u NVIDIA GPU-versnelling gebruikt:

1. Werk de NVIDIA-stuurprogramma&#x27;s bij naar de nieuwste versie
2. Zorg ervoor dat de GPU 4 GB+ VRAM heeft
3. Sluit GPU-intensieve applicaties (games, videobewerking)
4. Controleer de GPU-temperatuur (zorg voor voldoende koeling)

***

## Volgende stappen

Zodra de verwerking is gestart:

1. **Controleer de voortgang** - Zie [De verwerking controleren](monitoring-the-processing.md)
2. **Wacht tot de verwerking is voltooid** - De verwerking verloopt automatisch
3. **Bekijk de resultaten** - Zie [De verwerking voltooien](finishing-the-processing.md)

Voor informatie over wat u tijdens de verwerking moet doen, zie [De verwerking controleren](monitoring-the-processing.md).
