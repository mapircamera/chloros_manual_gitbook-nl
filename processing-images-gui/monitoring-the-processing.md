# De verwerking volgen

Zodra de verwerking is gestart, biedt Chloros verschillende manieren om de voortgang te volgen, te controleren op problemen en inzicht te krijgen in wat er met je dataset gebeurt. Op deze pagina wordt uitgelegd hoe je de verwerking kunt volgen en hoe je de informatie die Chloros verstrekt kunt interpreteren.

## Overzicht voortgangsbalk

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

1.**Detecteren** – Kalibratiedoelen zoeken
2. **Analyseren** – Beelden onderzoeken en de pijplijn voorbereiden
3. **Kalibreren** - Vignetterings- en reflectancecorrecties toepassen
4. **Exporteren** - Verwerkte bestanden opslaan

**Interactieve functies:**

* **Beweeg de muis over** de voortgangsbalk om het uitgebreide paneel met 4 fasen te zien
* **Klik op** de voortgangsbalk om het uitgebreide paneel te bevriezen/vast te zetten
* **Klik nogmaals** om het paneel weer vrij te geven; het wordt automatisch verborgen wanneer de muis eraf gaat
* Elke fase toont de individuele voortgang (0-100%)

{% hint style="info" %}
**Pariteit met CLI**: tijdens een `chloros-cli process`-run rapporteren dezelfde vier threads ‘Detecting’, ‘Analyzing’, ‘Processing’ en ‘Exporting’, en toont `chloros-cli export-status` de live voortgang van Thread-4-export vanaf een andere terminal. Zie de [CLI-referentie](../reference/cli-reference.md).
{% endhint %}

***

## Inzicht in elke verwerkingsfase

{% hint style="info" %}
**Pijplijnarchitectuur**: Deze 4 GUI-fasen komen overeen met de [4-thread verwerkingspijplijn](../processing-architecture/processing-pipeline.md). Op systemen met GPU-versnelling profiteert thread 3 (Kalibreren) van [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md), waarmee de verwerking wordt geoptimaliseerd voor uw specifieke hardware.
{% endhint %}

### Fase 1: Detecteren (Doelherkenning)

**Wat gebeurt er:**

* Chloros scant de afbeeldingen die je hebt aangevinkt met het selectievakje ‘Doel’ (alle afbeeldingen als er geen zijn aangevinkt)
* Computervisie-algoritmen identificeren de kalibratiepanelen
* Reflectiewaarden worden uit elk paneel gehaald
* Tijdstempels van de doelen worden geregistreerd voor een correcte planning van de kalibratie

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
* Controleer het foutopsporingslogboek op „Doel gevonden“-berichten

### Fase 2: Analyseren

**Wat gebeurt er:**

* EXIF-metadata van afbeeldingen lezen (tijdstempels, belichtingsinstellingen)
* Kalibratiestrategie bepalen op basis van tijdstempels van doelen en beschikbare DAQ-downwelling-gegevens
* Wachtlijst voor beeldverwerking ordenen
* Werkers voor parallelle verwerking voorbereiden (alleen Chloros+)

**Duur:** 5-30 seconden**Voortgangsindicator:**

* Analyseren: 0% → 100%
* Snelle fase, is meestal snel voltooid

**Waarop letten:**

* De voortgang moet gestaag verlopen zonder onderbrekingen
* Waarschuwingen over ontbrekende metagegevens verschijnen in het debuglogboek

### Fase 3: Kalibreren

**Wat gebeurt er:**

* **Debayering**: Het RAW-Bayerpatroon omzetten naar 3 kanalen (overgeslagen voor LATTICE-mono-modules, met een opmerking)
* **Vignetteringscorrectie**: Verduistering aan de randen van de lens verwijderen
* **Reflectiekalibratie**: Normaliseren met streefwaarden en/of DAQ-downwelling
* **Indexberekening**: Multispectrale indices berekenen
* Elk beeld verwerken via de volledige pijplijn

**Duur:** Het grootste deel van de totale verwerkingstijd (60-80%)**Voortgangsindicator:**

* Kalibreren: 0% → 100%
* Huidige afbeelding wordt verwerkt
* Voltooide afbeeldingen / Totaal aantal afbeeldingen

**Verwerkingsgedrag:**

* **Vrije modus**: Verwerkt één beeld tegelijk, achtereenvolgens
* **Chloros+-modus**: Draait een hardware-adaptieve werkpool — 1-4 gelijktijdige workers op GPU-systemen (afhankelijk van VRAM), één worker per fysieke kern (minus één) op systemen met alleen een CPU. Zie [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-versnelling**: Versnelt deze fase aanzienlijk**Waarop letten:**

* Gestage voortgang in het aantal afbeeldingen
* Controleer het debuglogboek op voltooiingsberichten per afbeelding
* Waarschuwingen over beeldkwaliteit of kalibratieproblemen

### Fase 4: Exporteren

**Wat gebeurt er:**

* Verwerkte beelden worden na voltooiing in het geselecteerde formaat naar de schijf geschreven
* **LATTICE**: elk frame wordt uitgesplitst naar elk ingeschakeld product (debayered / preview / radiance / reflectance)
* Multispectrale indexafbeeldingen worden geëxporteerd met LUT-kleuren
* De uitvoerstructuur `<project>/<camera>/<format>/<Product>_Images/` wordt aangemaakt — geëxporteerde bestanden behouden de bestandsnaam van het bronbestand; de map geeft het product aan

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

Het debuglog biedt gedetailleerde informatie over de voortgang van de verwerking en eventuele problemen die zich hebben voorgedaan. Opstartberichten van de backend worden ook in de logconsole weergegeven, zodat het log het volledige verhaal vertelt, zelfs als u het pas later opent.

### Het debuglogboek openen

1. Klik op het **Debuglogboek**-pictogram <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

in de linkerzijbalk
2. Het logpaneel wordt geopend en toont realtime verwerkingsberichten
3. Er wordt automatisch gescrolld om de nieuwste berichten weer te geven

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Logberichten begrijpen De logregels van

Chloros worden voorafgegaan door tags tussen haakjes die het subsysteem benoemen — bijvoorbeeld `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Het belangrijkste om te weten is het **run-overzicht**, dat aan het einde van elke run wordt weergegeven (inclusief gestopte runs):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Er volgen extra `[RUN-SUMMARY]`-hintregels wanneer iets uitleg behoeft — bijvoorbeeld een run die niets heeft opgeleverd, of een camera waarvan het gevraagde product werd overgeslagen omdat het niet van toepassing was. `[EXPORT-CHECK]`-regels geven uitleg over het overslaan per camera (bijvoorbeeld waarom een RGB-camera geen stralingsproduct kreeg).

De algemene ernstniveaus van berichten (onderstaande voorbeelden zijn ter illustratie, niet letterlijk):

#### Informatieberichten (wit/grijs)

Normale verwerkingsupdates: verwerking gestart, doelen gedetecteerd (met aantal panelen), voortgang van de kalibratie per afbeelding, bestanden geëxporteerd, verwerking voltooid.

#### Waarschuwingsberichten (geel)

Niet-kritieke problemen die de verwerking niet stoppen — bijv. ontbrekende GPS-gegevens in een frame, een groot tijdsverschil tussen doelbeelden, of een laag contrast in een kalibratiepaneel.

**Actie:** Controleer de waarschuwingen na de verwerking, maar onderbreek deze niet

#### Foutmeldingen (roodRed

)

Kritieke problemen die ervoor kunnen zorgen dat de verwerking mislukt — bijv. schijf vol, een beschadigd beeldbestand of geen doelen gedetecteerd terwijl reflectiekalibratie was aangevraagd.

**Actie:** Stop de verwerking, los de fout op en start opnieuw

### Veelvoorkomende logboek-situaties

| Situatie                             | Betekenis                                       | Vereiste actie                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Doel gedetecteerd in \[bestandsnaam]        | Kalibratiedoel succesvol gevonden         | Geen — normaal                                         |
| Voortgangsbalken per afbeelding              | Huidige voortgangsupdate                       | Geen — normaal                                         |
| Geen doelen gevonden                      | Geen kalibratiedoelen gedetecteerd               | Doelafbeeldingen markeren of reflectiekalibratie uitschakelen |
| Onvoldoende schijfruimte               | Onvoldoende opslagruimte voor uitvoer                 | Schijfruimte vrijmaken                                    |
| Beschadigd bestand wordt overgeslagen               | Beeldbestand is beschadigd                         | Kopieer het bestand opnieuw vanaf de SD-kaart                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Een opname zonder raw-frame kan niet worden verwerkt | Neem opnieuw op met raw, of gebruik de CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | De bewerking leverde geen beeldproducten op — gerapporteerd als een fout met hints | Lees de hintregels; controleer wat is overgeslagen en waarom |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Lange paden in Windows zijn uitgeschakeld en de projectmap is te diep voor de limiet van 260 tekens; er is niets verwerkt | Gebruik een kortere projectmap (het bericht geeft de langste naam aan die past), of schakel lange paden in Windows in en start Chloros opnieuw op — zie [De verwerking starten](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Het clampmasker van een LATTICE-product kon niet worden geschreven; het product zelf is in orde en geeft de foutmelding `Chloros:UnmixClampMask=unavailable` | Controleer de schijfruimte en maprechten, en voer de verwerking vervolgens opnieuw uit als je het masker nodig hebt |

### Loggegevens kopiëren

Om het logboek te kopiëren voor probleemoplossing of ondersteuning:

1. Open het paneel **Debug Log**

2. Klik op de knop**Kopiëren**(kopieert het logboek naar het klembord) of op**Downloaden** om het op te slaan als tekstbestand
3. Plak de inhoud in een tekstbestand of e-mail
4. Stuur het indien nodig naar de MAPIR-ondersteuning

### Een bug melden via het tabblad Logboek

Nieuw in 1.3.0: het bugpictogram (**Een bug melden**) bovenaan het tabblad Debuglogboek stuurt een rapport rechtstreeks naar MAPIR.

1. Klik op het bugpictogram bovenaan het tabblad Debuglogboek
2. Beschrijf het probleem: wat je aan het doen was, wat je verwachtte en wat er in plaats daarvan gebeurde (verplicht)
3. Vink eventueel **Voeg een paar afbeeldingen toe (maximaal 10)**aan en kies afbeeldingen uit het geopende project, of gebruik**Bladeren...** voor een ander afbeeldingsbestand
4. Klik op **Verzenden**

Het rapport bevat je beschrijving, eventuele bijgevoegde afbeeldingen, de Chloros-logbestanden en systeemgegevens (Chloros-versie, besturingssysteem, GPU en projectinstellingen). Je moet zijn ingelogd bij Chloros (tabblad ‘Gebruiker’) om een rapport te kunnen verzenden. Je kunt het venster sluiten terwijl het rapport wordt verzonden; zodra het verzenden is voltooid, wordt er een referentienummer zoals `Bug report sent. Reference: BR-…` in het logboek geschreven — vermeld dit nummer wanneer je contact opneemt met MAPIR. Als het rapport niet kan worden geüpload (bijvoorbeeld vanwege een ontbrekende internetverbinding), slaat Chloros het op je computer op en toont het pad, zodat je het per e-mail kunt versturen naar support@mapir.camera.

***

## Systeembronnenbewaking

### CPU-gebruik

**Vrije modus:**

* 1 CPU-kern op ~100%
* Andere kernen inactief of beschikbaar
* Systeem blijft responsief

**Parallelle modus van Chloros+:**

* Meerdere kernen met hoge bezettingsgraad — hoeveel hangt af van de strategie die is gekozen door [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Het systeem kan minder responsief aanvoelen

**Om te controleren:**

* Windows Taakbeheer (Ctrl+Shift+Esc)
* Tabblad Prestaties → Sectie CPU
* Zoek naar "Chloros"- of "chloros-backend"-processen

### Geheugen (RAM)-gebruik

**Typisch gebruik:**

* Kleine projecten (< 100 afbeeldingen): 2-4 GB
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

**Wat je kunt verwachten:**

* Hoge schijfleessnelheid tijdens de analysefase
* Hoge schrijfactiviteit tijdens de exportfase
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

* De parallelle modus van Chloros verbruikt te veel systeembronnen
* Overweeg het aantal gelijktijdige taken te verminderen of de hardware te upgraden
* De vrije modus is minder systeembronintensief

### Wanneer de verwerking stoppen

Stop de verwerking als je het volgende ziet:

* ❌ Fouten zoals „Schijf vol“ of „Kan bestand niet schrijven“
* ❌ Herhaaldelijke fouten met beschadigde afbeeldingsbestanden
* ❌ Systeem is volledig vastgelopen (reageert niet)
* ❌ Je realiseert je dat er verkeerde instellingen zijn geconfigureerd
* ❌ Verkeerde afbeeldingen geïmporteerd

**Zo stop je de verwerking:**

1. Klik op de**Stop-knop** (vervangt de Start-knop) — één keer is voldoende
2. De balk geeft „Stoppen...“ weer terwijl de lopende afbeelding wordt afgerond; daarna eindigt de verwerking in een gestopte toestand
3. Reeds geëxporteerde producten blijven op de schijf staan; het logboek geeft een eerlijke `[RUN-SUMMARY]`-melding weer van wat is voltooid
4. Los de problemen op en start opnieuw — de bewerking begint vanaf het begin

***

## Probleemoplossing tijdens de verwerking

### De verwerking verloopt erg traag

**Mogelijke oorzaken:**

* Niet-gemarkeerde doelafbeeldingen (alle afbeeldingen worden gescand)
* HDD in plaats van SSD-opslag
* Onvoldoende systeembronnen
* Veel geconfigureerde indexen
* Toegang tot netwerkstation

**Oplossingen:**

1. Als het proces net is gestart en zich in de detectiefase bevindt: stop, markeer doelen, start opnieuw
2. Voor de toekomst: gebruik een SSD, verminder het aantal indexen, upgrade de hardware
3. Overweeg de CLI-interface voor batchverwerking van grote datasets

### Waarschuwingen over „schijfruimte“

**Oplossingen:**

1. Maak onmiddellijk schijfruimte vrij
2. Verplaats het project naar een schijf met meer ruimte
3. Verminder het aantal te exporteren indexen
4. Schakel LATTICE-exportproducten uit die je niet nodig hebt (Projectinstellingen → Verwerking)
5. Gebruik het JPG-formaat in plaats van TIFF (kleinere bestanden)

### Frequente meldingen over "beschadigde bestanden"

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
* De regels `[RUN-SUMMARY]` verschijnen in het foutlogboek met de definitieve aantallen
* De startknop wordt weer actief
* Alle uitvoerbestanden bevinden zich in de uitvoerstructuur per camera van het project: `<project>/<camera>/<format>/<Product>_Images/`

***

## Volgende stappen

Zodra de verwerking is voltooid:

1. **Bekijk de resultaten** - Zie [De verwerking afronden](finishing-the-processing.md)
2. **Controleer de uitvoermap** - Controleer of alle bestanden correct zijn geëxporteerd
3. **Controleer het foutlogboek** – Controleer op eventuele waarschuwingen of fouten
4. **Bekijk een voorbeeld van de verwerkte afbeeldingen** – Gebruik de afbeeldingsviewer of externe software

Zie [De verwerking afronden](finishing-the-processing.md) voor informatie over het bekijken en gebruiken van uw verwerkte resultaten.
