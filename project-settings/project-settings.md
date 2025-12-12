# Projectinstellingen

Met de projectinstellingen <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> zijbalk in Chloros kunt u alle aspecten van beeldverwerking, kalibratiedoeldetectie, multispectrale indexberekeningen en exportopties voor uw project configureren. Deze instellingen worden samen met uw project opgeslagen en kunnen als sjablonen worden opgeslagen voor hergebruik in meerdere projecten.

## Projectinstellingen openen

Om de projectinstellingen te openen:

1. Open een project in Chloros
2. Klik op het tabblad **Projectinstellingen**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in de linkerzijbalk.
3. Het instellingenpaneel toont alle beschikbare configuratieopties, geordend per categorie.

***

## Doeldetectie

Deze instellingen bepalen hoe Chloros kalibratiedoelen in uw afbeeldingen detecteert en verwerkt.

### Minimaal kalibratiemonstergebied (px)

* **Type**: Getal
* **Bereik**: 0 tot 10.000 pixels
* **Standaard**: 25 pixels
* **Beschrijving**: Stelt het minimale gebied (in pixels) in dat vereist is om een gedetecteerd gebied als een geldig kalibratiedoelmonster te beschouwen. Kleinere waarden detecteren kleinere doelen, maar kunnen het aantal valse positieven verhogen. Grotere waarden vereisen grotere, duidelijkere doelgebieden voor detectie.
* **Wanneer aanpassen**:
  * Verhoog deze waarde als u valse detecties krijgt op kleine beeldartefacten.
  * Verlaag deze waarde als uw kalibratiedoelen klein lijken in uw afbeeldingen en niet worden gedetecteerd.

### Minimale doelclustering (0-100)

* **Type**: Getal
* **Bereik**: 0 tot 100
* **Standaard**: 60
* **Beschrijving**: Regelt de clusteringdrempel voor het groeperen van gebieden met vergelijkbare kleuren bij het detecteren van kalibratiedoelen. Hogere waarden vereisen dat meer vergelijkbare kleuren worden gegroepeerd, wat resulteert in een meer conservatieve detectie van doelen. Lagere waarden staan meer kleurvariatie toe binnen een doelgroep.
* **Wanneer aanpassen**:
  * Verhoog deze waarde als kalibratiedoelen worden opgesplitst in meerdere detecties.
  * Verlaag deze waarde als kalibratiedoelen met kleurvariatie niet volledig worden gedetecteerd.

***

## Verwerking

Deze instellingen bepalen hoe Chloros uw afbeeldingen verwerkt en kalibreert.

### Vignetteringscorrectie

* **Type**: Selectievakje
* **Standaard**: Ingeschakeld (aangevinkt)
* **Beschrijving**: Past vignetteringscorrectie toe om lensverduistering aan de randen van afbeeldingen te compenseren. Vignettering is een veelvoorkomend optisch fenomeen waarbij de hoeken en randen van een afbeelding donkerder lijken dan het midden vanwege de eigenschappen van de lens.
* **Wanneer uitschakelen**: Schakel deze optie alleen uit als uw camera/lenscombinatie al vignetteringscorrectie heeft toegepast, of als u vignettering handmatig wilt corrigeren tijdens de nabewerking.

### Reflectiekalibratie / witbalans

* **Type**: Selectievakje
* **Standaard**: Ingeschakeld (aangevinkt)
* **Beschrijving**: Schakelt automatische reflectiekalibratie in met behulp van gedetecteerde kalibratiedoelen in uw afbeeldingen. Dit normaliseert de reflectiewaarden in uw dataset en zorgt voor consistente metingen, ongeacht de lichtomstandigheden.
* **Wanneer uitschakelen**: Schakel alleen uit als u onbewerkte, niet-gekalibreerde afbeeldingen wilt verwerken of als u een andere kalibratieworkflow gebruikt.

### Debayer-methode

* **Type**: Keuzelijst
* **Opties**:
  * Hoge kwaliteit (sneller) - Momenteel de enige beschikbare optie
* **Standaard**: Hoge kwaliteit (sneller)
* **Beschrijving**: Selecteert het demosaicing-algoritme dat wordt gebruikt om ruwe Bayer-patroonsensorgegevens om te zetten in full-colour afbeeldingen. De methode &quot;Hoge kwaliteit (sneller)&quot; biedt een optimale balans tussen verwerkingssnelheid en beeldkwaliteit.
* **Opmerking**: In toekomstige versies van Chloros kunnen extra debayer-methoden worden toegevoegd.

### Minimaal herkalibratie-interval

* **Type**: Getal
* **Bereik**: 0 tot 3600 seconden
* **Standaard**: 0 seconden
* **Beschrijving**: Stelt het minimale tijdsinterval (in seconden) in tussen het gebruik van kalibratiedoelen. Wanneer deze waarde is ingesteld op 0, gebruikt Chloros elk gedetecteerd kalibratiedoel. Wanneer deze waarde is ingesteld op een hogere waarde, gebruikt Chloros alleen kalibratiedoelen die ten minste dit aantal seconden van elkaar verwijderd zijn, waardoor de verwerkingstijd voor datasets met frequente kalibratiedoelopnames wordt verkort.
* **Wanneer aanpassen**:
  * Stel in op 0 voor maximale kalibratienauwkeurigheid wanneer de lichtomstandigheden variëren.
  * Verhoog (bijvoorbeeld tot 60-300 seconden) voor snellere verwerking wanneer de belichting consistent is en u veel kalibratiedoelafbeeldingen hebt.

### Tijdzone-offset lichtsensor

* **Type**: Getal
* **Bereik**: -12 tot +12 uur
* **Standaard**: 0 uur
* **Beschrijving**: Specificeert de tijdzone-offset (in uren ten opzichte van UTC) voor tijdstempels van lichtsensorgegevens. Dit wordt gebruikt bij het verwerken van PPK-gegevensbestanden (Post-Processed Kinematic) om te zorgen voor een correcte tijdsynchronisatie tussen beeldopnames en GPS-gegevens.
* **Wanneer aanpassen**: Stel dit in op uw lokale tijdzone-offset als uw PPK-gegevens lokale tijd gebruiken in plaats van UTC. Bijvoorbeeld:
  * Pacific Time: -8 of -7 (afhankelijk van zomertijd)
  * Eastern Time: -5 of -4 (afhankelijk van zomertijd)
  * Central European Time: +1 of +2 (afhankelijk van zomertijd)

### PPK-correcties toepassen

* **Type**: Selectievakje
* **Standaard**: Uitgeschakeld (niet aangevinkt)
* **Beschrijving**: Maakt het gebruik mogelijk van Post-Processed Kinematic (PPK)-correcties van MAPIR DAQ-recorders met een GPS (GNSS). Wanneer deze optie is ingeschakeld, gebruikt Chloros alle .daq-logbestanden met belichtingspin-gegevens in uw projectmap en past het nauwkeurige geolocatiecorrecties toe op uw afbeeldingen.
* **Vereiste**: er moet een .daq-logbestand met belichtingspin-gegevens aanwezig zijn in uw projectmap
* **Wanneer inschakelen**: het wordt aanbevolen om PPK-correctie altijd in te schakelen als u belichtingsfeedbackgegevens in uw .daq-logbestand hebt.

### Belichtingspin 1

* **Type**: Keuzelijst
* **Zichtbaarheid**: Alleen zichtbaar wanneer &quot;PPK-correcties toepassen&quot; is ingeschakeld EN belichtingsgegevens beschikbaar zijn voor pin 1
* **Opties**:
  * Cameramodelnamen die in het project zijn gedetecteerd
  * &quot;Niet gebruiken&quot; - Deze belichtingspin negeren
* **Standaard**: Automatisch geselecteerd op basis van projectconfiguratie
* **Beschrijving**: Wijst een specifieke camera toe aan belichtingspin 1 voor PPK-tijdsynchronisatie. De belichtingspin registreert het exacte tijdstip waarop de camerasluiter wordt geactiveerd, wat cruciaal is voor nauwkeurige PPK-geolocatie.
* **Gedrag bij automatische selectie**:
  * Eén camera + één pin: selecteert automatisch de camera
  * Eén camera + twee pinnen: pin 1 wordt automatisch toegewezen aan de camera
  * Meerdere camera&#x27;s: handmatige selectie vereist

### Belichtingspin 2

* **Type**: selectie via vervolgkeuzelijst
* **Zichtbaarheid**: alleen zichtbaar wanneer &quot;PPK-correcties toepassen&quot; is ingeschakeld EN belichtingsgegevens beschikbaar zijn voor pin 2
* **Opties**:
  * Cameramodelnamen die in het project zijn gedetecteerd
  * &quot;Niet gebruiken&quot; - deze belichtingspin negeren
* **Standaard**: automatisch geselecteerd op basis van projectconfiguratie
* **Beschrijving**: wijst een specifieke camera toe aan belichtingspin 2 voor PPK-tijdsynchronisatie bij gebruik van een opstelling met twee camera&#x27;s.
* **Gedrag bij automatische selectie**:
  * Eén camera + één pin: pin 2 wordt automatisch ingesteld op &quot;Niet gebruiken&quot;
  * Enkele camera + twee pinnen: Pin 2 wordt automatisch ingesteld op &quot;Niet gebruiken&quot;
  * Meerdere camera&#x27;s: Handmatige selectie vereist
* **Opmerking**: Dezelfde camera kan niet tegelijkertijd aan zowel Pin 1 als Pin 2 worden toegewezen.

***

## Index

Met deze instellingen kunt u multispectrale indices configureren voor analyse en visualisatie.

### Index toevoegen

* **Type**: Speciaal configuratiescherm voor indexen
* **Beschrijving**: Opent een interactief scherm waarin u multispectrale vegetatie-indexen (NDVI, NDRE, EVI, enz.) kunt selecteren en configureren om tijdens de beeldverwerking te berekenen. U kunt meerdere indices toevoegen, elk met hun eigen visualisatie-instellingen.
* **Beschikbare indices**: Het systeem bevat meer dan 30 vooraf gedefinieerde multispectrale indices, waaronder:
  * NDVI (genormaliseerde vegetatie-index)
  * NDRE (genormaliseerde verschilindex RedEdge)
  * EVI (verbeterde vegetatie-index)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * En nog veel meer (zie [Multispectrale indexformules](multispectral-index-formulas.md) voor de volledige lijst)
* **Functies**:
  * Kies uit vooraf gedefinieerde indexformules
  * Configureer visualisatiekleurengradiënten (LUT - Look-Up Tables)
  * Stel drempelwaarden in voor analyse
  * Maak aangepaste indexformules

### Aangepaste formules (Chloros+ functie)

* **Type**: Reeks aangepaste formuledfinities
* **Beschrijving**: Hiermee kunt u aangepaste multispectrale indexformules maken en opslaan met behulp van bandwiskunde. Aangepaste formules worden opgeslagen met uw projectinstellingen en kunnen net als ingebouwde indexen worden gebruikt.
* **Hoe te maken**:
  1. Zoek in het configuratiescherm Index naar de optie voor aangepaste formules.
  2. Definieer uw formule met behulp van bandidentificaties (bijv. NIR, Red, Green, Blue).
  3. Sla de formule op met een beschrijvende naam.
* **Formulesyntaxis**: Standaard wiskundige bewerkingen worden ondersteund, waaronder:
  * Rekenkundig: `+`, `-`, `*`, `/`
  * Haakjes voor de volgorde van bewerkingen
  * Bandreferenties: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Exporteren

Deze instellingen bepalen het formaat en de kwaliteit van geëxporteerde bewerkte afbeeldingen.

### Gekalibreerd afbeeldingsformaat

* **Type**: Keuzelijst
* **Opties**:
  * **TIFF (16-bits)** - Ongecomprimeerd 16-bits TIFF-formaat
  * **TIFF (32-bits, procent)** - 32-bits drijvende-kommagetal TIFF met reflectiewaarden als percentages
  * **PNG (8-bits)** - Gecomprimeerd 8-bits PNG-formaat
  * **JPG (8-bits)** - Gecomprimeerd 8-bits JPEG-formaat
* **Standaard**: TIFF (16-bits)
* **Beschrijving**: Selecteert het bestandsformaat voor het opslaan van verwerkte en gekalibreerde afbeeldingen.
* **Aanbevolen formaten**:
  * **TIFF (16-bits)**: Aanbevolen voor wetenschappelijke analyse en professionele workflows. Behoudt maximale datakwaliteit zonder compressieartefacten. Het meest geschikt voor multispectrale analyse en verdere verwerking in GIS-software.
  * **TIFF (32-bits, procent)**: Het meest geschikt voor workflows die reflectiewaarden als percentages (0-100%) vereisen. Biedt maximale precisie voor radiometrische metingen.
  * **PNG (8-bit)**: Geschikt voor weergave op het web en algemene visualisatie. Kleinere bestandsgroottes met compressie zonder verlies, maar met een kleiner dynamisch bereik.
  * **JPG (8-bit)**: Kleinste bestandsgroottes, alleen geschikt voor voorbeelden en weergave op het web. Maakt gebruik van compressie met verlies, wat niet geschikt is voor wetenschappelijke analyse.

***

## Projectsjabloon opslaan

Met deze functie kunt u uw huidige projectinstellingen opslaan als een herbruikbaar sjabloon.

* **Type**: Tekstinvoer + knop Opslaan
* **Beschrijving**: Voer een beschrijvende naam in voor uw instellingssjabloon en klik op het pictogram Opslaan. Het sjabloon slaat al uw huidige projectinstellingen op (doeldetectie, verwerkingsopties, indices en exportformaat) zodat u deze gemakkelijk kunt hergebruiken in toekomstige projecten.
* **Gebruiksscenario&#x27;s**:
  * Maak sjablonen voor verschillende camerasystemen (RGB, multispectraal, NIR)
  * Sla standaardconfiguraties op voor specifieke gewastypen of analyseworkflows
  * Deel consistente instellingen binnen een team
* **Gebruiksaanwijzing**:
  1. Configureer alle gewenste projectinstellingen
  2. Voer een sjabloonnaam in (bijv. &quot;RedEdge Survey3 NDVI Standaard&quot;)
  3. Klik op het pictogram Opslaan
  4. De sjabloon kan nu worden geladen bij het aanmaken van nieuwe projecten

***

## Projectmap opslaan

Deze instelling bepaalt waar nieuwe projecten standaard worden opgeslagen.

* **Type**: Weergave van de maproute + knop Bewerken
* **Standaard**: `C:\Users\[Username]\Chloros Projects`
* **Beschrijving**: Toont de huidige standaardmap waar nieuwe Chloros-projecten worden aangemaakt. Klik op het pictogram Bewerken om een andere map te selecteren.
* **Wanneer wijzigen**:
  * Stel in op een netwerkstation voor teamsamenwerking.
  * Wijzig naar een station met meer opslagruimte voor grote datasets.
  * Organiseer projecten op jaar, klant of projecttype in verschillende mappen.
* **Opmerking**: het wijzigen van deze instelling heeft alleen invloed op NIEUWE projecten. Bestaande projecten blijven op hun oorspronkelijke locatie staan.

***

## Instellingen behouden

Alle projectinstellingen worden automatisch opgeslagen met uw projectbestand (`.mapir`-projectformaat). Wanneer u een project opnieuw opent, worden alle instellingen precies zo hersteld als u ze hebt achtergelaten.

### Hiërarchie van instellingen

Instellingen worden in de volgende volgorde toegepast:

1. **Systeemstandaarden** - Ingebouwde standaardinstellingen gedefinieerd door Chloros
2. **Sjablooninstellingen** - Als u een sjabloon laadt bij het maken van een project
3. **Opgeslagen projectinstellingen** - Instellingen die zijn opgeslagen met het projectbestand
4. **Handmatige aanpassingen** - Alle wijzigingen die u tijdens de huidige sessie aanbrengt

### Instellingen en beeldverwerking

De meeste wijzigingen in de instellingen (vooral in de categorieën Verwerking en Exporteren) leiden tot een herverwerking van de beelden om de nieuwe instellingen weer te geven. Sommige instellingen zijn echter &quot;alleen voor export&quot; en vereisen geen onmiddellijke herverwerking:

* Projectsjabloon opslaan
* Werkdirectory
* Gekalibreerd afbeeldingsformaat (van toepassing bij exporteren)

***

## Best practices

1. **Begin met de standaardinstellingen**: De standaardinstellingen werken goed voor de meeste MAPIR-camerasystemen en typische workflows.
2. **Sjablonen maken**: Nadat u de instellingen voor een specifieke workflow of camera hebt geoptimaliseerd, slaat u deze op als sjabloon om consistentie tussen projecten te garanderen.
3. **Test voordat u de volledige verwerking uitvoert**: wanneer u met nieuwe instellingen experimenteert, test u deze eerst op een kleine subset van afbeeldingen voordat u uw volledige dataset verwerkt.
4. **Documenteer uw instellingen**: gebruik beschrijvende sjabloonnamen die het camerasysteem, het verwerkingstype en het beoogde gebruik aangeven (bijvoorbeeld &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Selectie van exportformaat**: Kies uw exportformaat op basis van uw eindgebruik:
   * Wetenschappelijke analyse → TIFF (16-bits of 32-bits)
   * GIS-verwerking → TIFF (16-bits)
   * Snelle visualisatie → PNG (8-bit)
   * Delen via internet → JPG (8-bit)

***

Voor meer informatie over multispectrale indices in Chloros, zie de pagina [Multispectrale indexformules](multispectral-index-formulas.md).
