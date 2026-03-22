# Projectinstellingen aanpassen

Voordat u uw beelden gaat verwerken, is het belangrijk om uw projectinstellingen af te stemmen op uw workflow. Het paneel Projectinstellingen <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> biedt uitgebreide controle over kalibratie, verwerkingsopties, multispectrale indices en exportformaten.

## Projectinstellingen openen

1. Open uw project in Chloros
2. Klik op het **Projectinstellingen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in de linkerzijbalk
3. Het paneel Projectinstellingen toont alle configuratieopties

{% hint style="info" %}
**Instellingen worden automatisch opgeslagen** met uw project. Wanneer u een project opnieuw opent, worden alle instellingen hersteld.
{% endhint %}

***

## Snelle configuratie voor veelvoorkomende workflows

### Standaardinstellingen (aanbevolen voor de meeste gebruikers)

Voor typische MAPIR Survey3-cameraworkflows werken de standaardinstellingen goed:

* ✅ **Vignetcorrectie**: Ingeschakeld
* ✅ **Reflectiekalibratie**: Ingeschakeld (vereist afbeeldingen van MAPIR-doelen)
* ✅ **Debayer-methode**: Standaard (Snel, gemiddelde kwaliteit)
* ✅ **Exportformaat**: TIFF (16-bits)

Importeer gewoon uw afbeeldingen en begin met verwerken met deze standaardinstellingen.

***

## Overzicht projectinstellingen

Het paneel Projectinstellingen is onderverdeeld in verschillende categorieën. Hieronder vindt u een samenvatting van elke sectie. Zie [Projectinstellingen](../project-settings/project-settings.md) voor de volledige documentatie.

### Doelherkenning

Bepaalt hoe Chloros kalibratiedoelen in uw afbeeldingen identificeert.

**Belangrijkste instellingen:*** **Minimaal kalibratiesamplegebied**: Drempelwaarde voor doelherkenning (standaard: 25 pixels)
* **Minimale doelclustering**: Drempelwaarde voor het groeperen van doelgebieden (standaard: 60)**Wanneer aanpassen:**

* Vergroot het samplegebied als er valse detecties optreden
* Verklein het gebied als er geen doelen worden gedetecteerd
* Pas de clustering aan als doelen in meerdere detecties worden opgesplitst

### Verwerking

Belangrijkste opties voor beeldverwerking en kalibratie.

**Belangrijkste instellingen:*** **Vignetcorrectie**: Compenseert lensverduistering aan de randen ✅ Aanbevolen
* **Reflectiekalibratie**: Normaliseert waarden met behulp van kalibratiedoelen ✅ Aanbevolen
* **Debayer-methode**: Algoritme voor het converteren van RAW naar 3-kanaals multispectraal
* **Minimaal herkalibratie-interval**: Tijd tussen het gebruik van kalibratiedoelen (0 = alles gebruiken)**Geavanceerde instellingen:*** **Tijdzone-offset lichtsensor**: Voor PPK-tijdsynchronisatie (standaard: 0)
* **PPK-correcties toepassen**: Gebruikt GPS/belichtingspin-gegevens uit .daq-bestanden
* **Belichtingspin 1/2**: Wijst camera&#x27;s toe aan belichtingspinnen voor opstellingen met twee camera&#x27;s

### Debayer-methode

We bieden momenteel 2 debayering-methoden aan in Chloros:

#### Standaard (Snel, gemiddelde kwaliteit)

Standaard debayer verwerkt snel, maar vertoont debayering-kleurruis, wat resulteert in minder nauwkeurige en ruisigere beelden.

#### Texture Aware (Langzaam, Hoogste kwaliteit) \[Alleen Chloros+]

Texture Aware maakt gebruik van een hoogwaardige randbewuste debayer in combinatie met een AI/ML-ruisonderdrukkingsmodel dat bijna alle debayering-ruis verwijdert. Het Texture Aware-model heeft GPU-geheugen (VRAM) nodig om te draaien. We raden aan dit te gebruiken wanneer u &gt;4 GB VRAM beschikbaar hebt voor snellere verwerking.

### Index (Multispectrale indices)

Configureer welke vegetatie-indices moeten worden berekend en geëxporteerd.

**Hoe indices toevoegen:**

1. Klik op de knop**&quot;Index toevoegen&quot;**

2. Selecteer een index uit het dropdownmenu (NDVI, NDRE, GNDVI, enz.)
3. Configureer de visualisatie-instellingen (LUT-kleuren, waardebereiken)
4. Voeg indien nodig meerdere indices toe

**Populaire indices:*** **NDVI**: Algemene gezondheid van de vegetatie (meest gebruikelijk)
* **NDRE**: Vroege stressdetectie in combinatie met RedEdge
* **GNDVI**: Gevoelig voor chlorofylconcentratie
* **OSAVI**: Werkt goed bij zichtbare bodem
* **EVI**: Gebieden met hoge bladoppervlakte-index (LAI)**Aangepaste formules (alleen Chloros+):**

* Maak aangepaste multispectrale indexformules
* Gebruik bandberekeningen met alle beeldkanalen
* Sla aangepaste formules op voor hergebruik

Zie [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md) voor alle beschikbare indices en formules.

### Exporteren

Bepaalt het bestandsformaat en de kwaliteit van de uitvoer.

**Beschikbare formaten:*** **TIFF (16-bits)**: Aanbevolen voor GIS en wetenschappelijke analyse (bereik 0-65.535)
* **TIFF (32-bits, procent)**: Reflectiewaarden met drijvende komma (bereik 0,0-1,0)
* **PNG (8-bits)**: Verliesloze compressie voor visualisatie (bereik 0-255)
* **JPG (8-bits)**: Kleinste bestanden, compressie met verlies (bereik 0-255)***

## Instellingen opslaan en laden

### Project-sjabloon opslaan

Maak herbruikbare sjablonen voor consistente workflows:

1. Configureer alle gewenste instellingen in het paneel Projectinstellingen
2. Scrol naar de sectie **&quot;Projectsjabloon opslaan&quot;** onderaan
3. Voer een beschrijvende sjabloonnaam in (bijv. &quot;Survey3N\_RGN\_Landbouw&quot;)
4. Klik op het opslagpictogram

**Voordelen:**

* Pas identieke instellingen toe op meerdere projecten
* Deel configuraties met teamleden
* Zorg voor consistentie bij herhaalde enquêtes

### Sjabloon laden bij nieuw project

Bij het aanmaken van een nieuw project:

1. Selecteer **&quot;Nieuw project&quot;** in het hoofdmenu
2. Kies de optie **&quot;Laden vanuit sjabloon&quot;**

3. Selecteer uw opgeslagen sjabloon
4. Alle instellingen worden automatisch toegepast

### Werkmap

De instelling **&quot;Projectmap opslaan&quot;** bepaalt waar nieuwe projecten standaard worden aangemaakt:

* **Standaardlocatie**: `C:\Users\[Username]\Chloros Projects`
* **Locatie wijzigen**: Klik op het bewerkingspictogram en selecteer een nieuwe map
* **Wanneer wijzigen**:
  * Netwerkschijf voor teamsamenwerking
  * Andere schijf met meer opslagruimte
  * Georganiseerde mappenstructuur per jaar/klant

***

## PPK (Post-Processed Kinematic) instellingen

Bij gebruik van MAPIR DAQ-recorders met GPS voor nauwkeurige geolocatie:

### Vereisten

* MAPIR DAQ met GPS (GNSS)-module
* .daq-logbestand met belichtingspinnen
* Camera aangesloten op DAQ-belichtingspinnen tijdens opnamesessie

### Configuratiestappen

1. Plaats het .daq-logbestand in uw projectmap
2. Schakel in Projectinstellingen het selectievakje **&quot;PPK-correcties toepassen&quot;** in
3. Stel indien nodig **&quot;Tijdzone-offset lichtsensor&quot;** in (standaard: 0 voor UTC)
4. Wijs camera&#x27;s toe aan belichtingspinnen:
   * **Enkele camera**: Wordt automatisch toegewezen aan pin 1
   * **Dubbele camera&#x27;s**: Wijs elke camera handmatig toe aan de juiste pin**Toewijzing belichtingspinnen:*** **Belichtingspin 1**: Selecteer het cameramodel uit de vervolgkeuzelijst
* **Belichtingspin 2**: Selecteer de tweede camera of &quot;Niet gebruiken&quot;
* Dezelfde camera kan niet aan beide pinnen worden toegewezen

{% hint style="warning" %}
**Belangrijk**: Belichtingspinnen moeten correct aan hun respectievelijke camera&#x27;s worden toegewezen. Een onjuiste toewijzing leidt tot verkeerde geolocatiegegevens.
{% endhint %}

***

## Geavanceerde scenario&#x27;s

### Projecten met meerdere camera&#x27;s

Bij het verwerken van beelden van meerdere MAPIR-camera&#x27;s in één project:

1. Chloros detecteert automatisch elk cameramodel
2. Elke camera krijgt het juiste verwerkingsprofiel
3. PPK: Wijs elke camera handmatig toe aan de juiste belichtingspin
4. Alle camera&#x27;s gebruiken hetzelfde exportformaat en dezelfde indexen

**Voorbeeld**: Survey3W RGN + Survey3N OCN dual-camera rig

### Time-lapse- of multi-date-metingen

Voor herhaalde metingen van hetzelfde gebied in de loop van de tijd:

1. Maak een sjabloon met uw standaardinstellingen
2. Gebruik bij elke sessie een consistente kalibratiedoelopstelling
3. Verwerk elke datum als een afzonderlijk project
4. Gebruik identieke instellingen voor vergelijkbare resultaten
5. Exporteer in hetzelfde formaat voor temporele analyse

### Grote datasets

Voor projecten met veel afbeeldingen (500+):

* Overweeg om op te splitsen in kleinere projecten per datum of gebied
* Gebruik Chloros+ parallelle verwerking voor snellere resultaten
* Overweeg CLI of API voor batchautomatisering
* Pas het minimale herkalibratie-interval aan om de detectietijd van doelen te verkorten

***

## Uw instellingen controleren

Controleer de volgende belangrijke instellingen voordat u begint met verwerken:

* [ ] Cameramodel correct gedetecteerd in bestandsbrowser
* [ ] Vignetcorrectie ingeschakeld
* [ ] Reflectiekalibratie ingeschakeld
* [ ] Ten minste één kalibratiedoelafbeelding geïmporteerd
* [ ] Gewenste multispectrale indices toegevoegd
* [ ] Exportformaat geschikt voor uw workflow
* [ ] PPK-instellingen geconfigureerd (bij gebruik van .daq met belichtingsgebeurtenissen)

***

## Volgende stappen

Zodra uw instellingen zijn geconfigureerd:

1. **Markeer kalibratiedoelafbeeldingen** - Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
2. **Start de verwerking** - Zie [De verwerking starten](starting-the-processing.md)
3. **Volg de voortgang** - Zie [De verwerking volgen](monitoring-the-processing.md)

Raadpleeg de referentiedocumentatie [Projectinstellingen](../project-settings/project-settings.md) voor volledige details over alle beschikbare instellingen.
