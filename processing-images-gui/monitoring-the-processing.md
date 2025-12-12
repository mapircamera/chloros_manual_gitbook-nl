# De verwerking monitoren

Zodra de verwerking is gestart, biedt Chloros verschillende manieren om de voortgang te monitoren, te controleren op problemen en inzicht te krijgen in wat er met uw dataset gebeurt. Op deze pagina wordt uitgelegd hoe u uw verwerking kunt volgen en de informatie die Chloros biedt, kunt interpreteren.

## Overzicht voortgangsbalk

De voortgangsbalk in de bovenste koptekst toont de realtime verwerkingsstatus en het voltooiingspercentage.

### Voortgangsbalk in de gratis modus

Voor gebruikers zonder Chloros+-licentie:

**Voortgangsbalk in twee fasen:**

1. **Doeldetectie** - Kalibratiedoelen zoeken in afbeeldingen
2. **Verwerking** - Correcties toepassen en exporteren

**Voortgangsbalk toont:**

* Algemeen voltooiingspercentage (0-100%)
* Naam van de huidige fase
* Eenvoudige horizontale balkvisualisatie

### Chloros+ voortgangsbalk

Voor gebruikers met een Chloros+ licentie:

**Voortgangsbalk met 4 fasen:**

1. **Detecteren** - Kalibratiedoelen zoeken
2. **Analyseren** - Beelden onderzoeken en pijplijn voorbereiden
3. **Kalibreren** - Vignettering en reflectiecorrecties toepassen
4. **Exporteren** - Verwerkte bestanden opslaan

**Interactieve functies:**

* **Beweeg de muis over** de voortgangsbalk om het uitgebreide paneel met 4 fasen te zien
* **Klik op** de voortgangsbalk om het uitgebreide paneel vast te zetten/vast te pinnen
* **Klik nogmaals** om het paneel los te maken en automatisch te verbergen wanneer u de muis weghaalt
* Elke fase toont de individuele voortgang (0-100%)

***

## Uitleg van elke verwerkingsfase

### Fase 1: Detecteren (doeldetectie)

**Wat gebeurt er:**

* Chloros scant afbeeldingen die zijn gemarkeerd met het selectievakje Doel
* Computervisie-algoritmen identificeren de 4 kalibratiepanelen
* Reflectiewaarden worden uit elk paneel gehaald
* Tijdstempels van doelen worden geregistreerd voor een juiste kalibratieschema

**Duur:**

* Met gemarkeerde doelen: 10-60 seconden
* Zonder gemarkeerde doelen: 5-30+ minuten (scant alle afbeeldingen)

**Voortgangsindicator:**

* Detecteren: 0% → 100%
* Aantal gescande afbeeldingen
* Aantal gevonden doelen

**Waar u op moet letten:**

* Moet snel voltooid zijn als de doelen correct zijn gemarkeerd
* Als het te lang duurt, zijn de doelen mogelijk niet gemarkeerd
* Controleer het foutopsporingslogboek op berichten met &#x27;Doel gevonden&#x27;

### Fase 2: Analyseren

**Wat gebeurt er:**

* EXIF-metadata van afbeeldingen lezen (tijdstempels, belichtingsinstellingen)
* Kalibratiestrategie bepalen op basis van tijdstempels van doelen
* Wachtrij voor beeldverwerking organiseren
* Werkers voor parallelle verwerking voorbereiden (alleen Chloros+)

**Duur:** 5-30 seconden

**Voortgangsindicator:**

* Analyseren: 0% → 100%
* Snelle fase, meestal snel voltooid

**Waar u op moet letten:**

* Moet gestaag vorderen zonder pauzes
* Waarschuwingen over ontbrekende metadata verschijnen in het foutopsporingslogboek

### Fase 3: Kalibreren

**Wat gebeurt er:**

* **Debayering**: RAW-Bayer-patroon omzetten naar 3 kanalen
* **Vignettecorrectie**: verwijderen van donkere randen rond de lens
* **Reflectiekalibratie**: normaliseren met streefwaarden
* **Indexberekening**: berekenen van multispectrale indices
* Verwerken van elke afbeelding via de volledige pijplijn

**Duur:** het grootste deel van de totale verwerkingstijd (60-80%)

**Voortgangsindicator:**

* Kalibreren: 0% → 100%
* Huidige afbeelding wordt verwerkt
* Afbeeldingen voltooid / Totaal aantal afbeeldingen

**Verwerkingsgedrag:**

* **Vrije modus**: Verwerkt één afbeelding tegelijk achter elkaar
* **Chloros+ modus**: Verwerkt maximaal 16 afbeeldingen tegelijk
* **GPU-versnelling**: versnelt deze fase aanzienlijk

**Waar u op moet letten:**

* Gelijkmatige voortgang door het aantal afbeeldingen
* Controleer het foutopsporingslogboek voor voltooiingsberichten per afbeelding
* Waarschuwingen over beeldkwaliteit of kalibratieproblemen

### Fase 4: Exporteren

**Wat gebeurt er:**

* Gekalibreerde afbeeldingen naar schijf schrijven in geselecteerd formaat
* Multispectrale indexafbeeldingen exporteren met LUT-kleuren
* Submappen voor cameramodellen aanmaken
* Originele bestandsnamen behouden met de juiste extensies

**Duur:** 10-20% van de totale verwerkingstijd

**Voortgangsindicator:**

* Exporteren: 0% → 100%
* Bestanden worden geschreven
* Exportformaat en bestemming

**Waar u op moet letten:**

* Waarschuwingen over schijfruimte
* Fouten bij het schrijven van bestanden
* Voltooiing van alle geconfigureerde uitvoer

***

## Tabblad Debuglog

Het debuglog biedt gedetailleerde informatie over de voortgang van de verwerking en eventuele problemen die zich hebben voorgedaan.

### Toegang tot het debuglog

1. Klik op het pictogram **Debuglog** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> in de linkerzijbalk.
2. Het logboekvenster wordt geopend met realtime verwerkingsberichten.
3. Er wordt automatisch gescrolld om de nieuwste berichten weer te geven.

### Logboekberichten begrijpen

#### Informatieberichten (wit/grijs)

Normale verwerkingsupdates:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Waarschuwingsberichten (geel)

Niet-kritieke problemen die de verwerking niet stoppen:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Actie:** Bekijk de waarschuwingen na de verwerking, maar onderbreek deze niet.

#### Foutmeldingen (Red)

Kritieke problemen die ervoor kunnen zorgen dat de verwerking mislukt:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Actie:** Stop de verwerking, los de fout op en start opnieuw.

### Algemene logberichten

| Bericht                          | Betekenis                                | Vereiste actie                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Doel gedetecteerd in \[bestandsnaam]&quot; | Kalibratiedoel succesvol gevonden  | Geen - normaal                                         |
| &quot;Beeld X van Y wordt verwerkt&quot;        | Huidige voortgangsupdate                | Geen - normaal                                         |
| &quot;Geen doelen gevonden&quot;               | Geen kalibratiedoelen gedetecteerd        | Markeer doelbeelden of schakel reflectiekalibratie uit |
| &quot;Onvoldoende schijfruimte&quot;        | Onvoldoende opslagruimte voor uitvoer          | Maak schijfruimte vrij                                    |
| &quot;Beschadigd bestand overslaan&quot;        | Beeldbestand is beschadigd                  | Kopieer bestand opnieuw vanaf SD-kaart                             |
| &quot;PPK-gegevens toegepast&quot;               | GPS-correcties uit .daq-bestand toegepast | Geen - normaal                                         |

### Loggegevens kopiëren

Om het logboek te kopiëren voor probleemoplossing of ondersteuning:

1. Open het paneel Debug Log (Foutopsporingslogboek)
2. Klik op de knop **&quot;Copy Log&quot;** (Logboek kopiëren) (of klik met de rechtermuisknop → Select All (Alles selecteren))
3. Plak in een tekstbestand of e-mail
4. Stuur indien nodig naar MAPIR-ondersteuning

***

## Systeembronnen bewaken

### CPU-gebruik

**Vrije modus:**

* 1 CPU-kern op ~100%
* Andere kernen inactief of beschikbaar
* Systeem blijft responsief

**Chloros+ Parallelle modus:**

* Meerdere kernen op 80-100% (maximaal 16 kernen)
* Hoog algemeen CPU-gebruik
* Systeem kan minder responsief aanvoelen

**Om te controleren:**

* Windows Taakbeheer (Ctrl+Shift+Esc)
* Tabblad Prestaties → sectie CPU
* Zoek naar processen &quot;Chloros&quot; of &quot;chloros-backend&quot;

### Geheugengebruik (RAM)

**Normaal gebruik:**

* Kleine projecten (&lt; 100 afbeeldingen): 2-4 GB
* Middelgrote projecten (100-500 afbeeldingen): 4-8 GB
* Grote projecten (500+ afbeeldingen): 8-16 GB
* Chloros+ parallelle modus gebruikt meer RAM

**Als het geheugen bijna vol is:**

* Verwerk kleinere batches
* Sluit andere applicaties
* Upgrade het RAM-geheugen als u regelmatig grote datasets verwerkt

### GPU-gebruik (Chloros+ met CUDA)

Wanneer GPU-versnelling is ingeschakeld:

* NVIDIA GPU vertoont een hoog gebruik (60-90%)
* VRAM-gebruik neemt toe (vereist 4 GB+ VRAM)
* Kalibratiefase is aanzienlijk sneller

**Om te controleren:**

* NVIDIA-pictogram in het systeemvak
* Taakbeheer → Prestaties → GPU
* GPU-Z of vergelijkbare monitoringtool

### Schijf-I/O

**Wat u kunt verwachten:**

* Hoge schijfleesactiviteit tijdens de analysefase
* Hoge schrijfactiviteit tijdens de exportfase
* SSD aanzienlijk sneller dan HDD

**Prestatietip:**

* Gebruik indien mogelijk SSD voor projectmap
* Vermijd netwerkschijven voor grote datasets
* Zorg ervoor dat de schijf niet bijna vol is (beïnvloedt de schrijfsnelheid)

***

## Problemen detecteren tijdens verwerking

### Waarschuwingssignalen

**Voortgang stagneert (geen verandering gedurende 5+ minuten):**

* Controleer het foutopsporingslogboek op fouten
* Controleer de beschikbare schijfruimte
* Controleer Taakbeheer om te zien of Chloros actief is

**Er verschijnen regelmatig foutmeldingen:**

* Stop de verwerking en controleer de fouten
* Veelvoorkomende oorzaken: schijfruimte, beschadigde bestanden, geheugenproblemen
* Zie het gedeelte Probleemoplossing hieronder

**Systeem reageert niet meer:**

* Chloros+ parallelle modus gebruikt te veel bronnen
* Overweeg om het aantal gelijktijdige taken te verminderen of de hardware te upgraden
* De vrije modus is minder intensief voor de bronnen

### Wanneer moet u de verwerking stoppen?

Stop de verwerking als u het volgende ziet:

* ❌ Fouten &quot;Schijf vol&quot; of &quot;Kan bestand niet schrijven&quot;
* ❌ Herhaalde fouten met beschadigde afbeeldingsbestanden
* ❌ Systeem volledig vastgelopen (reageert niet)
* ❌ Verkeerde instellingen geconfigureerd
* ❌ Verkeerde afbeeldingen geïmporteerd

**Hoe stoppen:**

1. Klik op de **knop Stoppen/Annuleren** (vervangt de knop Start)
2. De verwerking wordt gestopt, de voortgang gaat verloren
3. Los de problemen op en start opnieuw vanaf het begin

***

## Problemen oplossen tijdens de verwerking

### De verwerking verloopt erg traag

**Mogelijke oorzaken:**

* Niet-gemarkeerde doelafbeeldingen (alle afbeeldingen worden gescand)
* HDD in plaats van SSD-opslag
* Onvoldoende systeembronnen
* Veel indexen geconfigureerd
* Toegang tot netwerkschijf

**Oplossingen:**

1. Als u net bent begonnen en u zich in de detectiefase bevindt: annuleer, markeer doelen, start opnieuw
2. Voor de toekomst: gebruik SSD, verminder het aantal indexen, upgrade de hardware
3. Overweeg CLI voor batchverwerking van grote datasets

### Waarschuwingen voor &quot;schijfruimte&quot;

**Oplossingen:**

1. Maak onmiddellijk schijfruimte vrij
2. Verplaats het project naar een schijf met meer ruimte
3. Verminder het aantal te exporteren indexen.
4. Gebruik het JPG-formaat in plaats van TIFF (kleinere bestanden).

### Frequente meldingen over &quot;beschadigde bestanden&quot;

**Oplossingen:**

1. Kopieer de afbeeldingen opnieuw vanaf de SD-kaart om de integriteit te waarborgen.
2. Test de SD-kaart op fouten.
3. Verwijder beschadigde bestanden uit het project.
4. Ga door met het verwerken van de resterende afbeeldingen.

### Oververhitting/throttling van het systeem

**Oplossingen:**

1. Zorg voor voldoende ventilatie.
2. Verwijder stof uit de ventilatieopeningen van de computer.
3. Verminder de verwerkingsbelasting (gebruik de modus Free in plaats van Chloros+).
4. Voer de verwerking uit tijdens koelere momenten van de dag.

***

## Meldingen dat de verwerking is voltooid

Wanneer de verwerking is voltooid:

* De voortgangsbalk bereikt 100%
* Het bericht **&quot;Verwerking voltooid&quot;** verschijnt in het foutopsporingslogboek
* De startknop wordt weer ingeschakeld
* Alle uitvoerbestanden staan in de submap van het cameramodel

***

## Volgende stappen

Zodra de verwerking is voltooid:

1. **Bekijk de resultaten** - Zie [De verwerking voltooien](finishing-the-processing.md)
2. **Controleer de uitvoermap** - Controleer of alle bestanden correct zijn geëxporteerd
3. **Bekijk het foutopsporingslogboek** - Controleer op waarschuwingen of fouten
4. **Bekijk een voorbeeld van de verwerkte afbeeldingen** - Gebruik Image Viewer of externe software

Zie [De verwerking voltooien](finishing-the-processing.md) voor informatie over het bekijken en gebruiken van uw verwerkte resultaten.
