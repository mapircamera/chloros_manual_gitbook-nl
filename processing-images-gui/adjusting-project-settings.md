# Projectinstellingen aanpassen

Voordat u uw afbeeldingen verwerkt, is het belangrijk om uw projectinstellingen te configureren zodat deze aansluiten bij uw workflowvereisten. Het paneel Projectinstellingen <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> bieden uitgebreide controle over kalibratie, verwerkingsopties, multispectrale indices en exportformaten.

## Projectinstellingen openen

1. Open uw project in Chloros
2. Klik op het pictogram **Projectinstellingen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in de linkerzijbalk
3. Het paneel Projectinstellingen toont alle configuratieopties

{% hint style=&quot;info&quot; %}
**De instellingen worden automatisch opgeslagen** met uw project. Wanneer u een project opnieuw opent, worden alle instellingen hersteld.
{% endhint %}

***

## Snelle installatie voor veelvoorkomende workflows

### Standaardinstellingen (aanbevolen voor de meeste gebruikers)

Voor typische MAPIR Survey3 cameraworkflows werken de standaardinstellingen goed:

* ✅ **Vignettecorrectie**: ingeschakeld
* ✅ **Reflectiekalibratie**: ingeschakeld (vereist afbeeldingen van MAPIR-doelen)
* ✅ **Debayer-methode**: Hoge kwaliteit (sneller)
* ✅ **Exportformaat**: TIFF (16-bits)

Importeer gewoon uw afbeeldingen en begin met de verwerking met deze standaardinstellingen.

***

## Overzicht van projectinstellingen

Het paneel Projectinstellingen is onderverdeeld in verschillende categorieën. Hieronder vindt u een overzicht van elke sectie. Raadpleeg [Projectinstellingen](../project-settings/project-settings.md) voor de volledige documentatie.

### Doeldetectie

Bepaalt hoe Chloros kalibratiedoelen in uw afbeeldingen identificeert.

**Belangrijkste instellingen:**

* **Minimale kalibratiemonsteroppervlakte**: Drempelwaarde voor doelherkenning (standaard: 25 pixels)
* **Minimale doelclustering**: Drempelwaarde voor het groeperen van doelgebieden (standaard: 60)

**Wanneer aanpassen:**

* Vergroot het monsteroppervlak als er valse detecties worden gedaan.
* Verklein het monsteroppervlak als er geen doelen worden gedetecteerd.
* Pas de clustering aan als doelen worden opgesplitst in meerdere detecties.

### Verwerking

Belangrijkste opties voor beeldverwerking en kalibratie.

**Belangrijkste instellingen:**

* **Vignettecorrectie**: Compenseert voor lensverduistering aan de randen ✅ Aanbevolen
* **Reflectiekalibratie**: Normaliseert waarden met behulp van kalibratiedoelen ✅ Aanbevolen
* **Debayer-methode**: Algoritme voor het converteren van RAW naar 3-kanaals multispectraal
* **Minimaal herkalibratie-interval**: tijd tussen het gebruik van kalibratiedoelen (0 = alles gebruiken)

**Geavanceerde instellingen:**

* **Tijdzone-offset lichtsensor**: voor PPK-tijdsynchronisatie (standaard: 0)
* **PPK-correcties toepassen**: gebruikt GPS-/belichtingspindata uit .daq-bestanden
* **Belichtingspin 1/2**: wijst camera&#x27;s toe aan belichtingspinnen voor opstellingen met twee camera&#x27;s

### Index (multispectrale indices)

Configureer welke vegetatie-indices moeten worden berekend en geëxporteerd.

**Indices toevoegen:**

1. Klik op de knop **&quot;Index toevoegen&quot;**
2. Selecteer een index in het vervolgkeuzemenu (NDVI, NDRE, GNDVI, enz.)
3. Configureer de visualisatie-instellingen (LUT-kleuren, waardebereiken)
4. Voeg indien nodig meerdere indices toe

**Populaire indices:**

* **NDVI**: Algemene gezondheid van de vegetatie (meest gebruikelijk)
* **NDRE**: Vroege stressdetectie met RedEdge
* **GNDVI**: Gevoelig voor chlorofylconcentratie
* **OSAVI**: Werkt goed met zichtbare grond
* **EVI**: Regio&#x27;s met hoge bladoppervlakte-index (LAI)

**Aangepaste formules (alleen Chloros+):**

* Maak aangepaste multispectrale indexformules
* Gebruik bandwiskunde met alle beeldkanalen
* Sla aangepaste formules op voor hergebruik

Zie [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md) voor alle beschikbare indices en formules.

### Exporteren

Regelt het uitvoerbestandsformaat en de kwaliteit.

**Beschikbare formaten:**

* **TIFF (16-bits)**: aanbevolen voor GIS en wetenschappelijke analyse (bereik 0-65.535)
* **TIFF (32-bits, procent)**: reflectiewaarden met drijvende komma (bereik 0,0-1,0)
* **PNG (8-bit)**: Verliesvrije compressie voor visualisatie (bereik 0-255)
* **JPG (8-bit)**: Kleinste bestanden, compressie met verlies (bereik 0-255)

***

## Instellingen opslaan en laden

### Projectsjabloon opslaan

Maak herbruikbare sjablonen voor consistente workflows:

1. Configureer alle gewenste instellingen in het paneel Projectinstellingen.
2. Scrol naar het gedeelte **&quot;Projectsjabloon opslaan&quot;** onderaan.
3. Voer een beschrijvende sjabloonnaam in (bijv. &quot;Survey3N\_RGN\_Agriculture&quot;).
4. Klik op het pictogram Opslaan.

**Voordelen:**

* Pas identieke instellingen toe op meerdere projecten.
* Deel configuraties met teamleden.
* Zorg voor consistentie bij herhaalde enquêtes.

### Sjabloon laden in nieuw project

Bij het maken van een nieuw project:

1. Selecteer **&quot;Nieuw project&quot;** in het hoofdmenu.
2. Kies de optie **&quot;Laden vanuit sjabloon&quot;**.
3. Selecteer uw opgeslagen sjabloon.
4. Alle instellingen worden automatisch toegepast.

### Werkmap

De instelling **&quot;Projectmap opslaan&quot;** geeft aan waar nieuwe projecten standaard worden aangemaakt:

* **Standaardlocatie**: `C:\Users\[Username]\Chloros Projects`
* **Locatie wijzigen**: klik op het pictogram Bewerken en selecteer een nieuwe map
* **Wanneer wijzigen**:
  * Netwerkschijf voor teamsamenwerking
  * Andere schijf met meer opslagruimte
  * Georganiseerde mappenstructuur op jaar/klant

***

## PPK (Post-Processed Kinematic) instellen

Als u MAPIR DAQ-recorders met GPS gebruikt voor nauwkeurige geolocatie:

### Vereisten

* MAPIR DAQ met GPS (GNSS)-module
* .daq-logbestand met blootstellingspin-invoer
* Camera aangesloten op DAQ-belichtingspinnen tijdens opnamesessie

### Configuratiestappen

1. Plaats het .daq-logbestand in uw projectmap.
2. Schakel in Projectinstellingen het selectievakje **&quot;PPK-correcties toepassen&quot;** in.
3. Stel indien nodig **&quot;Tijdzone-offset lichtsensor&quot;** in (standaard: 0 voor UTC).
4. Wijs camera&#x27;s toe aan belichtingspinnen:
   * **Enkele camera**: Wordt automatisch toegewezen aan pin 1
   * **Twee camera&#x27;s**: Wijs elke camera handmatig toe aan de juiste pin

**Toewijzing belichtingspinnen:**

* **Belichtingspin 1**: Selecteer het cameramodel in de vervolgkeuzelijst
* **Belichtingspin 2**: Selecteer de tweede camera of &quot;Niet gebruiken&quot;
* Dezelfde camera kan niet aan beide pinnen worden toegewezen

{% hint style=&quot;warning&quot; %}
**Belangrijk**: Belichtingspinnen moeten correct worden toegewezen aan hun respectievelijke camera&#x27;s. Een onjuiste toewijzing leidt tot onjuiste geolocatiegegevens.
{% endhint %}

***

## Geavanceerde scenario&#x27;s

### Projecten met meerdere camera&#x27;s

Bij het verwerken van beelden van meerdere MAPIR-camera&#x27;s in één project:

1. Chloros detecteert automatisch elk cameramodel
2. Elke camera krijgt het juiste verwerkingsprofiel
3. PPK: wijs elke camera handmatig toe aan de juiste belichtingspin
4. Alle camera&#x27;s gebruiken hetzelfde exportformaat en dezelfde indexen

**Voorbeeld**: Survey3W RGN + Survey3N OCN dubbele camera-opstelling

### Time-lapse- of multi-date-onderzoeken

Voor herhaalde onderzoeken van hetzelfde gebied in de loop van de tijd:

1. Maak een sjabloon met uw standaardinstellingen
2. Gebruik elke sessie een consistente kalibratiedoelopstelling
3. Verwerk elke datum als een afzonderlijk project
4. Gebruik identieke instellingen voor vergelijkbare resultaten
5. Exporteer in hetzelfde formaat voor temporele analyse

### Grote datasets

Voor projecten met veel afbeeldingen (500+):

* Overweeg om het project op te splitsen in kleinere projecten op basis van datum of gebied.
* Gebruik Chloros+ parallelle verwerking voor snellere resultaten.
* Overweeg CLI of API voor batchautomatisering.
* Pas het minimale herkalibratie-interval aan om de detectietijd van het doel te verkorten.

***

## Uw instellingen controleren

Controleer voordat u begint met verwerken de volgende belangrijke instellingen:

* [ ] Cameramodel correct gedetecteerd in bestandsbrowser
* [ ] Vignettecorrectie ingeschakeld
* [ ] Reflectiekalibratie ingeschakeld
* [ ] Ten minste één kalibratiedoelafbeelding geïmporteerd
* [ ] Gewenste multispectrale indices toegevoegd
* [ ] Exportformaat geschikt voor uw workflow
* [ ] PPK-instellingen geconfigureerd (bij gebruik van .daq met belichtingsgebeurtenissen)

***

## Volgende stappen

Zodra uw instellingen zijn geconfigureerd:

1. **Kalibratiedoelafbeeldingen markeren** - Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
2. **Verwerking starten** - Zie [De verwerking starten](starting-the-processing.md)
3. **Voortgang controleren** - Zie [De verwerking controleren](monitoring-the-processing.md)

Raadpleeg de referentiedocumentatie [Projectinstellingen](../project-settings/project-settings.md) voor volledige details over alle beschikbare instellingen.
