# De verwerking volgen

Zodra de verwerking is gestart, biedt Chloros verschillende manieren om de voortgang te volgen, te controleren op problemen en inzicht te krijgen in wat er met uw dataset gebeurt. Op deze pagina wordt uitgelegd hoe u uw verwerking kunt volgen en hoe u de informatie van Chloros kunt interpreteren.

## Overzicht voortgangsbalk

De voortgangsbalk in de bovenste koptekst toont de realtime verwerkingsstatus en het voltooiingspercentage.

### Voortgangsbalk in de gratis modus

Voor gebruikers zonder Chloros+-licentie:

**Voortgangsweergave in 2 fasen:**

1.**Doel detecteren** - Kalibratiedoelen in afbeeldingen vinden
2. **Verwerking** - Correcties toepassen en exporteren**De voortgangsbalk toont:**

* Algemeen voltooiingspercentage (0-100%)
* Naam van de huidige fase
* Eenvoudige horizontale balkweergave

### Voortgangsbalk van Chloros+

Voor gebruikers met een Chloros+-licentie:

**Voortgangsweergave in 4 fasen:**

1.**Detecteren** - Kalibratiedoelen vinden
2. **Analyseren** - Beelden onderzoeken en pijplijn voorbereiden
3. **Kalibreren** - Vignetterings- en reflectiecorrecties toepassen
4. **Exporteren** - Verwerkte bestanden opslaan**Interactieve functies:*** **Beweeg de muis over** de voortgangsbalk om het uitgebreide 4-fasenpaneel te zien
* **Klik op** de voortgangsbalk om het uitgebreide paneel te bevriezen/vast te zetten
* **Klik nogmaals** om het paneel te ontdooien en automatisch te verbergen wanneer de muis wordt weggehaald
* Elke fase toont de individuele voortgang (0-100%)

***

## Inzicht in elke verwerkingsfase

{% hint style="info" %}
**Pijplijnarchitectuur**: Deze 4 GUI-fasen komen overeen met de [4-thread verwerkingspijplijn](../processing-architecture/processing-pipeline.md). Op systemen met GPU-versnelling profiteert Thread 3 (Kalibreren) van [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md), wat de verwerking voor uw specifieke hardware optimaliseert.
{% endhint %}

### Fase 1: Detecteren (doeldetectie)

**Wat gebeurt er:**

* Chloros scant afbeeldingen die zijn gemarkeerd met het selectievakje Doel
* Computervisie-algoritmen identificeren de 4 kalibratiepanelen
* Reflectiewaarden worden uit elk paneel gehaald
* Doeltijdstempels worden geregistreerd voor een juiste kalibratieplanning

**Duur:**

* Met gemarkeerde doelen: 10-60 seconden
* Zonder gemarkeerde doelen: 5-30+ minuten (scant alle afbeeldingen)

**Voortgangsindicator:**

* Detecteren: 0% → 100%
* Aantal gescande afbeeldingen
* Aantal gevonden doelen

**Waarop letten:**

* Moet snel voltooid zijn als doelen correct gemarkeerd zijn
* Als het te lang duurt, zijn de doelen mogelijk niet gemarkeerd
* Controleer het debuglogboek op &quot;Doel gevonden&quot;-berichten

### Fase 2: Analyseren

**Wat gebeurt er:**

* Lezen van EXIF-metadata van afbeeldingen (tijdstempels, belichtingsinstellingen)
* Bepalen van de kalibratiestrategie op basis van de tijdstempels van de doelen
* Organiseren van de wachtrij voor beeldverwerking
* Voorbereiden van parallelle verwerkingsprocessen (alleen Chloros+)

**Duur:** 5-30 seconden**Voortgangsindicator:**

* Analyseren: 0% → 100%
* Snelle fase, meestal snel voltooid

**Waar u op moet letten:**

* Moet gestaag vorderen zonder pauzes
* Waarschuwingen over ontbrekende metadata verschijnen in het debuglogboek

### Fase 3: Kalibreren

**Wat gebeurt er:*** **Debayering**: Omzetten van RAW-Bayerpatroon naar 3 kanalen
* **Vignetteringscorrectie**: Verduistering aan de randen van de lens verwijderen
* **Reflectiekalibratie**: Normaliseren met streefwaarden
* **Indexberekening**: Multispectrale indices berekenen
* Elk beeld verwerken via de volledige pijplijn

**Duur:** Het grootste deel van de totale verwerkingstijd (60-80%)**Voortgangsindicator:**

* Kalibreren: 0% → 100%
* Huidige afbeelding wordt verwerkt
* Afbeeldingen voltooid / Totaal aantal afbeeldingen

**Verwerkingsgedrag:*** **Vrije modus**: Verwerkt één afbeelding tegelijk, sequentieel
* **Chloros+ modus**: Verwerkt tot 16 afbeeldingen tegelijk
* **GPU-versnelling**: Versnelt deze fase aanzienlijk**Waar u op moet letten:**

* Gestage voortgang door het aantal afbeeldingen
* Controleer het debuglogboek voor voltooiingsberichten per afbeelding
* Waarschuwingen over beeldkwaliteit of kalibratieproblemen

### Fase 4: Exporteren

**Wat gebeurt er:**

* Gekalibreerde beelden in het geselecteerde formaat naar de schijf schrijven
* Multispectrale indexbeelden met LUT-kleuren exporteren
* Submappen voor cameramodellen aanmaken
* Originele bestandsnamen behouden met de juiste achtervoegsels

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

Het debuglog biedt gedetailleerde informatie over de voortgang van de verwerking en eventuele problemen die zich voordoen.

### Toegang tot het debuglog

1. Klik op het **Debuglog** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> in de linkerzijbalk
2. Het logvenster wordt geopend en toont realtime verwerkingsberichten
3. Het venster scrolt automatisch om de nieuwste berichten weer te geven

### Logberichten begrijpen

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

**Actie:** Bekijk de waarschuwingen na de verwerking, maar onderbreek deze niet

#### Foutberichten (Red)

Kritieke problemen die ervoor kunnen zorgen dat de verwerking mislukt:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Actie:** Stop de verwerking, los de fout op en start opnieuw

### Veelvoorkomende logberichten

| Bericht                          | Betekenis                                | Vereiste actie                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Doel gedetecteerd in \[bestandsnaam]&quot; | Kalibratiedoel succesvol gevonden  | Geen - normaal                                         |
| &quot;Beeld X van Y wordt verwerkt&quot;        | Update van huidige voortgang                | Geen - normaal                                         |
| &quot;Geen doelen gevonden&quot;               | Geen kalibratiedoelen gedetecteerd        | Markeer doelafbeeldingen of schakel reflectiekalibratie uit |
| &quot;Onvoldoende schijfruimte&quot;        | Onvoldoende opslagruimte voor uitvoer          | Maak schijfruimte vrij                                    |
| &quot;Beschadigd bestand overslaan&quot;        | Afbeeldingsbestand is beschadigd                  | Kopieer bestand opnieuw vanaf SD-kaart                             |
| &quot;PPK-gegevens toegepast&quot;               | GPS-correcties uit .daq-bestand toegepast | Geen - normaal                                         |

### Loggegevens kopiëren

Om het logboek te kopiëren voor probleemoplossing of ondersteuning:

1. Open het paneel Debug Log
2. Klik op de knop **&quot;Log kopiëren&quot;** (of klik met de rechtermuisknop → Alles selecteren)
3. Plak in een tekstbestand of e-mail
4. Stuur indien nodig naar MAPIR-ondersteuning

***

## Systeembronnenbewaking

### CPU-gebruik

**Free-modus:**

* 1 CPU-kern op ~100%
* Andere kernen inactief of beschikbaar
* Systeem blijft responsief

**Chloros+ Parallel-modus:**

* Meerdere kernen op 80-100% (maximaal 16 kernen)
* Hoog totaal CPU-gebruik
* Systeem kan minder responsief aanvoelen

**Om te controleren:**

* Windows Taakbeheer (Ctrl+Shift+Esc)
* Tabblad Prestaties → Sectie CPU
* Zoek naar &quot;Chloros&quot; of &quot;chloros-backend&quot; processen

### Geheugen (RAM) gebruik

**Typisch gebruik:**

* Kleine projecten (&lt; 100 afbeeldingen): 2-4 GB
* Middelgrote projecten (100-500 afbeeldingen): 4-8 GB
* Grote projecten (500+ afbeeldingen): 8-16 GB
* De parallelle modus van Chloros+ gebruikt meer RAM

**Als het geheugen bijna op is:**

* Verwerk kleinere batches
* Sluit andere applicaties
* Upgrade het RAM-geheugen als je regelmatig grote datasets verwerkt

### GPU-gebruik (Chloros+ met CUDA)

Wanneer GPU-versnelling is ingeschakeld:

* De NVIDIA GPU vertoont een hoge bezettingsgraad (60-90%)
* Het VRAM-gebruik neemt toe (vereist 4 GB+ VRAM)
* De kalibratiefase verloopt aanzienlijk sneller

**Om te controleren:**

* NVIDIA-pictogram in het systeemvak
* Taakbeheer → Prestaties → GPU
* GPU-Z of een vergelijkbare monitoringtool

### Schijf-I/O

**Wat u kunt verwachten:**

* Hoge schijfleesactiviteit tijdens de analysefase
* Hoge schrijfactiviteit op de schijf tijdens de exportfase
* SSD is aanzienlijk sneller dan HDD

**Prestatietip:**

* Gebruik indien mogelijk een SSD voor de projectmap
* Vermijd netwerkschijven voor grote datasets
* Zorg ervoor dat de schijf niet bijna vol is (beïnvloedt de schrijfsnelheid)

***

## Problemen detecteren tijdens de verwerking

### Waarschuwingssignalen

**Voortgang stagneert (geen verandering gedurende 5+ minuten):**

* Controleer het foutlogboek op fouten
* Controleer de beschikbare schijfruimte
* Controleer Taakbeheer om te zien of Chloros actief is

**Foutmeldingen verschijnen regelmatig:**

* Stop de verwerking en bekijk de fouten
* Veelvoorkomende oorzaken: schijfruimte, beschadigde bestanden, geheugenproblemen
* Zie het gedeelte Probleemoplossing hieronder

**Systeem reageert niet meer:**

* Chloros+ parallelle modus gebruikt te veel bronnen
* Overweeg het aantal gelijktijdige taken te verminderen of de hardware te upgraden
* De vrije modus is minder bronintensief

### Wanneer de verwerking stoppen

Stop de verwerking als u het volgende ziet:

* ❌ Fouten &quot;Schijf vol&quot; of &quot;Kan bestand niet schrijven&quot;
* ❌ Herhaalde fouten met betrekking tot beschadigde afbeeldingsbestanden
* ❌ Systeem volledig vastgelopen (reageert niet)
* ❌ U realiseert zich dat er verkeerde instellingen zijn geconfigureerd
* ❌ Verkeerde afbeeldingen geïmporteerd

**Hoe te stoppen:**

1. Klik op de**Stop/Annuleren-knop** (vervangt de Start-knop)
2. De verwerking wordt gestopt, de voortgang gaat verloren
3. Los de problemen op en begin opnieuw vanaf het begin

***

## Probleemoplossing tijdens de verwerking

### De verwerking verloopt erg traag

**Mogelijke oorzaken:**

* Niet-gemarkeerde doelafbeeldingen (alle afbeeldingen worden gescand)
* HDD in plaats van SSD-opslag
* Onvoldoende systeembronnen
* Veel indexen geconfigureerd
* Toegang tot netwerkschijf

**Oplossingen:**

1. Als het net is gestart en zich in de detectiefase bevindt: Annuleren, doelen markeren, opnieuw starten
2. Voor de toekomst: Gebruik SSD, verminder het aantal indexen, upgrade de hardware
3. Overweeg CLI voor batchverwerking van grote datasets

### Waarschuwingen over &quot;schijfruimte&quot;

**Oplossingen:**

1. Maak onmiddellijk schijfruimte vrij
2. Verplaats het project naar een schijf met meer ruimte
3. Verminder het aantal te exporteren indexen
4. Gebruik JPG-formaat in plaats van TIFF (kleinere bestanden)

### Frequente &quot;Beschadigd bestand&quot;-meldingen

**Oplossingen:**

1. Kopieer afbeeldingen opnieuw vanaf de SD-kaart om de integriteit te waarborgen
2. Test de SD-kaart op fouten
3. Verwijder beschadigde bestanden uit het project
4. Ga door met het verwerken van de resterende afbeeldingen

### Oververhitting / Throttling van het systeem

**Oplossingen:**

1. Zorg voor voldoende ventilatie
2. Verwijder stof uit de ventilatieopeningen van de computer
3. Verminder de verwerkingsbelasting (gebruik de Free-modus in plaats van Chloros+)
4. Verwerk tijdens koelere momenten van de dag

***

## Melding dat de verwerking is voltooid

Wanneer de verwerking is voltooid:

* De voortgangsbalk bereikt 100%
* Het bericht **&quot;Verwerking voltooid&quot;** verschijnt in het foutlogboek
* De startknop wordt weer actief
* Alle uitvoerbestanden bevinden zich in de submap van het cameramodel

***

## Volgende stappen

Zodra de verwerking is voltooid:

1. **Bekijk de resultaten** - Zie [De verwerking afronden](finishing-the-processing.md)
2. **Controleer de uitvoermap** - Controleer of alle bestanden correct zijn geëxporteerd
3. **Bekijk het foutlogboek** - Controleer op waarschuwingen of fouten
4. **Bekijk een voorbeeld van de verwerkte afbeeldingen** - Gebruik de afbeeldingsviewer of externe software

Voor informatie over het bekijken en gebruiken van uw verwerkte resultaten, zie [De verwerking afronden](finishing-the-processing.md).
