# De verwerking volgen

Zodra de verwerking is gestart, biedt Chloros verschillende manieren om de voortgang te volgen, te controleren op problemen en inzicht te krijgen in wat er met uw dataset gebeurt. Op deze pagina wordt uitgelegd hoe u uw verwerking kunt volgen en hoe u de informatie die Chloros verstrekt, kunt interpreteren.

## Overzicht van de voortgangsbalk

De voortgangsbalk in de bovenste koptekst toont de realtime verwerkingsstatus en het voltooiingspercentage. De voortgang wordt live vanuit de backend gestreamd via Server-Sent Events (SSE), zodat de balk weergeeft wat de pijplijn op dat moment daadwerkelijk doet.

### Voortgangsbalk in de gratis modus

Voor gebruikers zonder Chloros+-licentie:

**Voortgangsweergave in 2 fasen:**

1.**Doeldetectie** – Kalibratiedoelen in afbeeldingen zoeken
2. **Verwerking** – Correcties toepassen en exporteren**De voortgangsbalk toont:**

* Algemeen voltooiingspercentage (0-100%)
* Naam van de huidige fase
* Eenvoudige horizontale balkweergave

### Chloros+ voortgangsbalk

Voor gebruikers met een Chloros+ licentie:

**Voortgangsweergave in 4 fasen:**

1.**Detecteren** - Kalibratiedoelen vinden
2. **Analyseren** - Beelden onderzoeken en verwerkingspijplijn voorbereiden
3. **Kalibreren** - Vignetterings- en reflectancecorrecties toepassen
4. **Exporteren** - Verwerkte bestanden opslaan**Interactieve functies:*** **Beweeg de muis over** de voortgangsbalk om het uitgevouwen paneel met 4 fasen te zien
* **Klik op** de voortgangsbalk om het uitgevouwen paneel te vergrendelen/vast te zetten
* **Klik nogmaals** om de vergrendeling op te heffen; het paneel wordt automatisch verborgen wanneer de muis eraf gaat
* Elke fase toont de individuele voortgang (0-100%)

{% hint style="info" %}
**CLI-pariteit**: tijdens een `chloros-cli process`-run rapporteren dezelfde vier threads ‘Detecting’, ‘Analyzing’, Processing, Exporting, en `chloros-cli export-status` toont de live voortgang van de export van Thread-4 vanaf een andere terminal. Zie de [CLI-referentie](../reference/cli-reference.md).
{% endhint %}

***

## Inzicht in elke verwerkingsfase

{% hint style="info" %}
**Pijplijnarchitectuur**: Deze 4 GUI-fasen komen overeen met de [4-thread-verwerkingspijplijn](../processing-architecture/processing-pipeline.md). Op systemen met GPU-versnelling profiteert thread 3 (Kalibreren) van [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md), waarmee de verwerking wordt geoptimaliseerd voor uw specifieke hardware.
{% endhint %}

### Fase 1: Detecteren (doeldetectie)

**Wat gebeurt er:**

* Chloros scant de afbeeldingen die u hebt aangevinkt met het selectievakje ‘Doel’ (alle afbeeldingen, alleen als er geen zijn aangevinkt)
* Computervisie-algoritmen identificeren de kalibratiepanelen
* Reflectiewaarden worden uit elk paneel gehaald
* Tijdstempels van doelen worden geregistreerd voor een correcte planning van de kalibratie

**Duur:**

* Met gemarkeerde doelen: 10-60 seconden
* Zonder gemarkeerde doelen: 5-30+ minuten (scant alle afbeeldingen)

**Voortgangsindicator:**

* Detecteren: 0% → 100%
* Aantal gescande afbeeldingen (telt alleen de afbeeldingen die daadwerkelijk worden gescand)
* Aantal gevonden doelen

**Waarop letten:**

* Moet snel voltooid zijn als de doelen correct zijn gemarkeerd
* Als het te lang duurt, zijn de doelen mogelijk niet gemarkeerd
* Controleer het foutopsporingslogboek op „Doel gevonden“-meldingen

### Fase 2: Analyseren

**Wat gebeurt er:**

* EXIF-metadata van afbeeldingen uitlezen (tijdstempels, belichtingsinstellingen)
* Kalibratiestrategie bepalen op basis van tijdstempels van doelen en beschikbare DAQ-downwelling-gegevens
* Wachtlijst voor beeldverwerking ordenen
* Werkers voor parallelle verwerking voorbereiden (alleen Chloros+)

**Duur:** 5-30 seconden**Voortgangsindicator:**

* Analyseren: 0% → 100%
* Snelle fase, is meestal snel voltooid

**Waar u op moet letten:**

* De voortgang moet gestaag verlopen zonder onderbrekingen
* Waarschuwingen over ontbrekende metagegevens verschijnen in het foutopsporingslogboek

### Fase 3: Kalibreren

**Wat gebeurt er:*** **Debayering**: Het RAW-Bayerpatroon omzetten naar 3 kanalen (overgeslagen voor LATTICE-mono-modules, met een opmerking)
* **Vignetteringscorrectie**: Verduistering aan de randen van de lens verwijderen
* **Reflectiekalibratie**: Normaliseren met streefwaarden en/of DAQ-downwelling
* **Indexberekening**: Multispectrale indices berekenen
* Elk beeld verwerken via de volledige pijplijn

**Duur:** Het grootste deel van de totale verwerkingstijd (60-80%)**Voortgangsindicator:**

* Kalibreren: 0% → 100%
* Huidige afbeelding wordt verwerkt
* Voltooide afbeeldingen / Totaal aantal afbeeldingen

**Verwerkingsgedrag:*** **Vrije modus**: Verwerkt één afbeelding per keer, achtereenvolgens
* **Chloros+-modus**: Laat een aan de hardware aangepaste werkpool draaien — 1-4 gelijktijdige workers op GPU-systemen (afhankelijk van VRAM), één worker per fysieke kern (minus één) op systemen met alleen een CPU. Zie [Dynamische rekenaanpassing](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-versnelling**: Versnelt deze fase aanzienlijk**Waar je op moet letten:**

* Gelijkmatige voortgang door het aantal afbeeldingen heen
* Controleer het debuglogboek op voltooiingsberichten per afbeelding
* Waarschuwingen over beeldkwaliteit of kalibratieproblemen

### Fase 4: Exporteren

**Wat gebeurt er:**

* Verwerkte beelden worden na voltooiing in het geselecteerde formaat naar de schijf geschreven
* **LATTICE**: elk frame wordt uitgesplitst naar elk ingeschakeld product (debayered / preview / radiance / reflectance)
* Multispectrale indexbeelden worden geëxporteerd met LUT-kleuren
* De uitvoerstructuur `<project>/<camera>/<format>/<Product>_Images/` wordt aangemaakt — geëxporteerde bestanden behouden de bronbestandsnaam; de map geeft het product aan

**Duur:** 10-20% van de totale verwerkingstijd**Voortgangsindicator:**

* Exporteren: 0% → 100%
* Bestanden worden geschreven
* Exportformaat en bestemming

**Waar u op moet letten:**

* Waarschuwingen over schijfruimte
* Fouten bij het schrijven van bestanden
* Voltooiing van alle geconfigureerde uitvoer

***

## Tabblad Debuglog

Het debuglogboek biedt gedetailleerde informatie over de voortgang van de verwerking en eventuele problemen die zich voordoen. Opstartberichten van de backend worden ook in de logconsole weergegeven, zodat het logboek het volledige verhaal vertelt, zelfs als u het pas later opent.

### Toegang tot het debuglogboek

1. Klik op het **Debug Log**-<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

-pictogram in de linkerzijbalk
2. Het logvenster wordt geopend en toont realtime verwerkingsberichten
3. Er wordt automatisch gescrolld om de nieuwste berichten weer te geven

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Logberichten begrijpen

Chloros-logregels worden voorafgegaan door tags tussen haakjes die het subsysteem aanduiden — bijvoorbeeld `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Het belangrijkste om te weten is het **overzicht van de run**, dat aan het einde van elke run wordt weergegeven (inclusief gestopte runs):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Er volgen extra `[RUN-SUMMARY]`-hintregels wanneer iets uitleg behoeft — bijvoorbeeld een run die niets heeft opgeleverd, of een camera waarvan het aangevraagde product is overgeslagen omdat het niet van toepassing was. `[EXPORT-CHECK]`-regels geven uitleg per-camera-overslaan (bijv. waarom een RGB-camera geen stralingsproduct kreeg).

De algemene ernstniveaus van berichten (onderstaande voorbeelden zijn ter illustratie, niet letterlijk):

#### Informatiemeldingen (wit/grijs)

Normale verwerkingsupdates: verwerking gestart, doelen gedetecteerd (met aantal panelen), voortgang van de kalibratie per afbeelding, bestanden geëxporteerd, verwerking voltooid.

#### Waarschuwingsmeldingen (geel)

Niet-kritieke problemen die de verwerking niet stoppen — bijv. ontbrekende GPS-gegevens in een frame, een groot tijdsverschil tussen doelbeelden, of een laag contrast in een kalibratiepaneel.

**Actie:** Bekijk de waarschuwingen na de verwerking, maar onderbreek deze niet

#### Foutmeldingen (Red)

Kritieke problemen die ervoor kunnen zorgen dat de verwerking mislukt — bijv. schijf vol, een beschadigd beeldbestand of geen doelen gedetecteerd terwijl reflectiekalibratie was aangevraagd.

**Actie:** Stop de verwerking, los de fout op en start opnieuw

### Veelvoorkomende logboeksituaties

| Situatie                             | Betekenis                                       | Vereiste actie                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Doel gedetecteerd in \[bestandsnaam]    | Kalibratiedoel succesvol gevonden         | Geen — normaal                                         |
| Voortgangsbalken per afbeelding            | Huidige voortgang bijgewerkt                       | Geen — normaal                                         |
| Geen doelen gevonden                      | Geen kalibratiedoelen gedetecteerd               | Doelafbeeldingen markeren of reflectiekalibratie uitschakelen |
| Onvoldoende schijfruimte               | Onvoldoende opslagruimte voor uitvoer                 | Schijfruimte vrijmaken                                    |
| Beschadigd bestand wordt overgeslagen               | Afbeeldingsbestand is beschadigd                         | Bestand opnieuw kopiëren vanaf SD-kaart                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Een opname zonder raw-frame kan niet worden verwerkt | Neem opnieuw op met raw, of gebruik CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | De run heeft geen beeldproducten opgeleverd — gerapporteerd als een fout met hints | Lees de hintregels; controleer wat is overgeslagen en waarom |

### Loggegevens kopiëren

Om het logboek te kopiëren voor probleemoplossing of ondersteuning:

1. Open het paneel Debuglog
2. Klik op de knop **&quot;Log kopiëren&quot;** (of klik met de rechtermuisknop → Alles selecteren)
3. Plak de inhoud in een tekstbestand of e-mail
4. Stuur het indien nodig naar de ondersteuning van MAPIR

***

## Systeembronnenbewaking

### CPU-gebruik

**Free-modus:**

* 1 CPU-kern op ~100%
* Andere kernen inactief of beschikbaar
* Systeem blijft responsief

**Chloros+ Parallel-modus:**

* Meerdere kernen met hoge bezetting — hoeveel hangt af van de strategie die is gekozen door [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Het systeem kan minder responsief aanvoelen

**Om te controleren:**

* Windows Taakbeheer (Ctrl+Shift+Esc)
* Tabblad Prestaties → Sectie CPU
* Zoek naar processen met de naam &quot;Chloros&quot; of &quot;chloros-backend&quot;

### Geheugengebruik (RAM)

**Typisch gebruik:**

* Kleine projecten (&lt; 100 afbeeldingen): 2-4 GB
* Middelgrote projecten (100-500 afbeeldingen): 4-8 GB
* Grote projecten (500+ afbeeldingen): 8-16 GB
* De parallelle modus van Chloros+ verbruikt meer RAM

**Als het geheugen bijna op is:**

* Verwerk kleinere batches
* Sluit andere applicaties
* Breid het RAM-geheugen uit als je regelmatig grote datasets verwerkt

### GPU-gebruik (Chloros+ met CUDA)

Wanneer GPU-versnelling is ingeschakeld:

* De NVIDIA-GPU vertoont een hoge bezettingsgraad (60–90%)
* Het VRAM-gebruik neemt toe (vereist 4 GB+ VRAM; 7 GB+ voor gelijktijdige Texture Aware-debayering)
* De kalibratiefase verloopt aanzienlijk sneller

**Om te controleren:**

* NVIDIA-pictogram in het systeemvak
* Taakbeheer → Prestaties → GPU
* GPU-Z of een vergelijkbare monitoringtool

### Schijf-I/O

**Wat u kunt verwachten:**

* Hoge schijfleesactiviteit tijdens de analysefase
* Hoge schijfschrijfactiviteit tijdens de exportfase
* SSD is aanzienlijk sneller dan HDD

**Prestatietip:**

* Gebruik indien mogelijk een SSD voor de projectmap
* Vermijd netwerkschijven voor grote datasets
* Zorg ervoor dat de schijf niet bijna vol is (beïnvloedt de schrijfsnelheid)

***

## Problemen opsporen tijdens de verwerking

### Waarschuwingssignalen

**Voortgang stagneert (geen verandering gedurende 5+ minuten):**

* Controleer het foutlogboek op fouten
* Controleer of er voldoende schijfruimte beschikbaar is
* Controleer in Taakbeheer of Chloros actief is

**Er verschijnen regelmatig foutmeldingen:**

* Stop de verwerking en bekijk de fouten
* Veelvoorkomende oorzaken: schijfruimte, beschadigde bestanden, geheugenproblemen
* Zie het gedeelte ‘Problemen oplossen’ hieronder

**Het systeem reageert niet meer:**

* Chloros+ in parallelle modus gebruikt te veel systeembronnen
* Overweeg het aantal gelijktijdige taken te verminderen of de hardware te upgraden
* De vrije modus is minder systeembronintensief

### Wanneer moet je de verwerking stoppen?

Stop de verwerking als je het volgende ziet:

* ❌ Fouten zoals „Schijf vol“ of „Kan bestand niet schrijven“
* ❌ Herhaalde fouten door beschadigde beeldbestanden
* ❌ Systeem volledig vastgelopen (reageert niet)
* ❌ Je realiseert je dat er verkeerde instellingen zijn geconfigureerd
* ❌ Verkeerde beelden geïmporteerd

**Hoe te stoppen:**

1. Klik op de**Stop-knop** (vervangt de Start-knop) — één keer is voldoende
2. De balk geeft &quot;Stoppen...&quot; weer terwijl het huidige beeld wordt afgerond; daarna eindigt de bewerking in een gestopte toestand
3. Reeds geëxporteerde producten blijven op de schijf staan; het logboek geeft een gedetailleerd overzicht (`[RUN-SUMMARY]`) van wat er is voltooid
4. Los de problemen op en start opnieuw — de bewerking begint dan vanaf het begin

***

## Problemen oplossen tijdens de verwerking

### De verwerking verloopt erg traag

**Mogelijke oorzaken:**

* Niet-gemarkeerde doelafbeeldingen (alle afbeeldingen worden gescand)
* HDD in plaats van SSD-opslag
* Onvoldoende systeembronnen
* Veel geconfigureerde indexen
* Toegang tot netwerkstation

**Oplossingen:**

1. Als het proces net is gestart en zich in de detectiefase bevindt: stop, markeer doelen, start opnieuw op
2. Voor de toekomst: gebruik een SSD, verminder het aantal indexen, upgrade de hardware
3. Overweeg CLI voor batchverwerking van grote datasets

### Waarschuwingen over „schijfruimte“

**Oplossingen:**

1. Maak onmiddellijk schijfruimte vrij
2. Verplaats het project naar een schijf met meer ruimte
3. Verminder het aantal te exporteren indexen
4. Schakel LATTICE-exportproducten uit die je niet nodig hebt (Projectinstellingen → Verwerking)
5. Gebruik het JPG-formaat in plaats van TIFF (kleinere bestanden)

### Frequente meldingen over &quot;beschadigde bestanden&quot;

**Oplossingen:**

1. Kopieer de afbeeldingen opnieuw vanaf de SD-kaart om de integriteit te waarborgen
2. Controleer de SD-kaart op fouten
3. Verwijder beschadigde bestanden uit het project
4. Ga door met het verwerken van de resterende afbeeldingen

### Oververhitting / Throttling van het systeem

**Oplossingen:**

1. Zorg voor voldoende ventilatie
2. Verwijder stof uit de ventilatieopeningen van de computer
3. Verminder de verwerkingsbelasting (gebruik de Free-modus in plaats van Chloros+)
4. Voer de verwerking uit op koelere momenten van de dag

***

## Melding dat de verwerking is voltooid

Wanneer de verwerking is voltooid:

* De voortgangsbalk bereikt 100%
* De regels `[RUN-SUMMARY]` verschijnen in het foutlogboek met de definitieve tellingen
* De startknop wordt weer actief
* Alle uitvoerbestanden bevinden zich in de uitvoerstructuur per camera van het project: `<project>/<camera>/<format>/<Product>_Images/`

***

## Volgende stappen

Zodra de verwerking is voltooid:

1. **Bekijk de resultaten** - Zie [De verwerking afronden](finishing-the-processing.md)
2. **Controleer de uitvoermap** - Controleer of alle bestanden correct zijn geëxporteerd
3. **Controleer het foutlogboek** – Controleer op eventuele waarschuwingen of fouten
4. **Bekijk een voorbeeld van de verwerkte afbeeldingen** – Gebruik de afbeeldingsviewer of externe software

Zie [De verwerking afronden](finishing-the-processing.md) voor informatie over het controleren en gebruiken van je verwerkte resultaten.
