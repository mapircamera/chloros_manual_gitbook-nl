# Projectinstellingen aanpassen

Voordat u uw beelden gaat verwerken, is het belangrijk om uw projectinstellingen zo te configureren dat ze aansluiten bij de eisen van uw workflow. Het paneel ‘Projectinstellingen’ (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) biedt uitgebreide controle over kalibratie, verwerkingsopties, multispectrale indices en exportformaten.

## Projectinstellingen openen

1. Open uw project in Chloros
2. Klik op het pictogram **Projectinstellingen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in de linkerzijbalk
3. Het paneel Projectinstellingen toont alle configuratieopties

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Het paneel **Projectinstellingen** — Weergave, doelherkenning en verwerking</p></figcaption></figure>{% hint style="info" %}**Instellingen worden automatisch opgeslagen** samen met je project. Wanneer je een project opnieuw opent, worden alle instellingen hersteld.
{% endhint %}

***

## Snelle installatie voor veelvoorkomende workflows

### Standaardinstellingen (aanbevolen voor de meeste gebruikers)

De standaardinstellingen werken goed voor typische Survey3- en LATTICE-workflows:

* ✅ **Vignetteringscorrectie**: Ingeschakeld
* ✅ **Reflectiekalibratie / witbalans**: Ingeschakeld (maakt gebruik van MAPIR-doelen en/of DAQ-lichtsensorgegevens)
* ✅ **Debayer-methode**: Standaard (Snel, gemiddelde kwaliteit)
* ✅ **Exportformaat**: TIFF (16-bit)
* ✅ **Alle exportproducten**: Ingeschakeld (LATTICE slaat automatisch de fan-out op als debayered, preview, radiance en reflectance)

Importeer gewoon uw afbeeldingen en begin met verwerken met deze standaardinstellingen.

***

## Overzicht van projectinstellingen

Het paneel **Projectinstellingen**is onderverdeeld in de onderstaande secties. Twee extra secties —**DAQ-lichtsensor**en**Array-uitlijning** — verschijnen automatisch wanneer uw project de relevante bestanden bevat. Zie [Projectinstellingen](../project-settings/project-settings.md) voor de volledige documentatie.

### Weergave

* **Resolutie miniatuurafbeeldingen**: Resolutie van de miniatuurafbeeldingen in het afbeeldingsraster. Opties:**Standaard (512 px)**,**1024 px**,**2048 px**,**Volledige resolutie**. Alleen voor weergave — heeft nooit invloed op de verwerking. Hogere waarden zien er scherper uit wanneer ingezoomd, maar laden langzamer.

### Doeldetectie

Bepaalt hoe Chloros kalibratiedoelen in uw afbeeldingen identificeert.

**Belangrijke instellingen:*** **Minimaal kalibratiesamplegebied (px)**: Drempelwaarde voor doeldetectie (standaard:**25**, bereik 0–10.000)
* **Minimale doelclustering (0–100)**: Drempelwaarde voor gelijkenis bij het groeperen van doelgebieden (standaard:**60**)**Wanneer aanpassen:**

* Vergroot het steekproefgebied als er valse detecties optreden
* Verklein het als doelen niet worden gedetecteerd
* Pas de clustering aan als doelen in meerdere detecties worden opgesplitst

{% hint style="info" %}
Deze instellingen zijn grijs weergegeven wanneer **Reflectanciekalibratie / witbalans** is uitgeschakeld — als deze is uitgeschakeld, wordt doelherkenning helemaal niet uitgevoerd.
{% endhint %}

### Verwerking

Belangrijkste opties voor beeldverwerking en kalibratie.

**Belangrijkste instellingen:*** **Vignetteringscorrectie**: Compenseert lensverduistering aan de randen ✅ Aanbevolen
* **Reflectankalibratie / witbalans**: Kalibreert beelden met behulp van gedetecteerde doelen (Survey3) en/of DAQ-lichtsensorgegevens (LATTICE) ✅ Aanbevolen
* **Debayer-methode**: Algoritme voor het omzetten van RAW naar 3-kanaals multispectraal
* **Minimaal herkalibratie-interval**: Minimale tijd in seconden tussen het gebruik van kalibratiedoelen (standaard:**0** = alles gebruiken, bereik 0–3600)**Niet-gekalibreerde alternatieve producten:**Wanneer een frame niet op reflectie kan worden gekalibreerd (geen doel beschikbaar of kalibratie uitgeschakeld), wordt het geëxporteerd als een van twee alternatieve producten —**er bestaat precies één van het paar per run**, gekozen door de schakelaar voor vignetcorrectie:

* **Sensorrespons exporteren**: schrijft `Sensor_Response_Images` — wordt gebruikt wanneer vignetcorrectie**uitgeschakeld** is
* **Vignettecorrectie exporteren**: schrijft `Vignette_Corrected_Images` — wordt gebruikt wanneer vignettecorrectie**aan** staat

Het selectievakje dat niet van toepassing is, is grijs weergegeven. Als u het vinkje bij het actieve selectievakje verwijdert, wordt dat bestand helemaal niet meer geschreven.

**LATTICE-exportproducten** (weergegeven voor elk project; deze zijn van toepassing op LATTICE-opnames):

* **Exporteren met debayering**: de lineair gedebayerde afbeelding (`Debayered_Images`). Geldt voor RGB en multispectrale modules.
* **Voorbeeld exporteren**: het weergavevoorbeeld (`Preview_Images`). RGB = witbalans (DAQ-lichtbron indien beschikbaar, anders grijswereld) + gamma; multispectraal = valsekleuruitrekking.
* **Stralingsintensiteit exporteren**: float32 spectrale stralingsintensiteit (`Radiance_Images`, W/m²/sr/nm). Alleen multispectrale modules — niet van toepassing op RGB-masters.
* ****Reflectantie exporteren**: uint16-reflectantie (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) wanneer een `.daq`-meting van neerwaartse straling of een doel binnen het beeld het beeld bestrijkt. Alleen multispectrale modules.

Alle vier zijn **standaard ingeschakeld**— één geïmporteerd LATTICE-ruwframe wordt in één verwerkingsronde verspreid over elk ingeschakeld en toepasselijk product. Het selectievakje**Reflectantie exporteren** is grijs weergegeven wanneer reflectantiecalibratie is uitgeschakeld. Instellingen die onmogelijk zijn vanwege de bovenliggende schakelaar, zijn altijd grijs weergegeven met een tooltip waarin de te wijzigen schakelaar wordt vermeld.**Geavanceerde instellingen:*** **Tijdzone-offset lichtsensor**: Aantal uren ten opzichte van UTC voor het afstemmen van de tijd van de lichtsensor (standaard: 0, bereik −12 tot +12)
* **PPK-correcties toepassen**: Gebruikt GPS-/belichtingspin-gegevens uit `.daq`-bestanden (standaard: uit)
* **Belichtingspin 1/2**: Wijst camera’s toe aan belichtingspinnen voor opstellingen met twee camera’s

{% hint style="info" %}
**Het LATTICE-invoerniveau is automatisch.** LATTICE-opnames bevatten hun verwerkingsniveau in de XMP-metadata, en de verwerking komt altijd bij het onbewerkte beeld in de pijplijn terecht — er hoeft niets in de GUI te worden geconfigureerd. (De vlag CLI en `--input-level` bestaan als een instelling voor gevorderde gebruikers ter overschrijving bij opnames waarbij metadata verloren is gegaan; zie de [CLI-referentie](../reference/cli-reference.md).)
{% endhint %}

### Debayer-methode

We bieden momenteel 2 debayering-methoden aan in Chloros:

#### Standaard (Snel, gemiddelde kwaliteit)

De standaard debayer verwerkt snel, maar vertoont kleurruis door de debayering, wat resulteert in minder nauwkeurige en ruisrijkere beelden.

#### Texture Aware (traag, hoogste kwaliteit) \[Alleen Chloros+]

Texture Aware maakt gebruik van een hoogwaardige, randgevoelige debayermethode in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayeringruis verwijdert. Het model heeft GPU-geheugen (VRAM) nodig om te draaien: met **7 GB of meer VRAM** kan het meerdere beelden tegelijk verwerken; bij minder dan 7 GB wordt één beeld per keer verwerkt (aanzienlijk langzamer). Zie [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-opnames maken altijd gebruik van de standaard-demosaic.** Er is geen door LATTICE getraind Texture Aware-model, dus de optie wordt niet aangeboden voor LATTICE-beelden — Survey3-beelden in hetzelfde project kunnen er nog steeds gebruik van maken.
{% endhint %}

### Index (multispectrale indices)

Configureer welke vegetatie-indices moeten worden berekend en geëxporteerd. De vervolgkeuzelijst in de GUI biedt **27 vooraf gedefinieerde indexformules**.**Indices toevoegen:**

1. Klik op de knop**&quot;Index toevoegen&quot;**

2. Selecteer een index uit het dropdown-menu (NDVI, NDRE, GNDVI, enz.)
3. Configureer de visualisatie-instellingen (LUT-kleuren, waardebereiken)
4. Voeg indien nodig meerdere indexen toe

**Populaire indexen:*** **NDVI**: Algemene gezondheid van de vegetatie (meest gangbaar)
* **NDRE**: Vroegtijdige stressdetectie in combinatie met RedEdge
* **GNDVI**: Gevoelig voor chlorofylconcentratie
* **OSAVI**: Werkt goed bij zichtbare bodem
* **EVI**: Gebieden met een hoge bladoppervlakte-index (LAI)**Aangepaste formules:**

* Maak aangepaste multispectrale indexformules met bandberekeningen over alle beeldkanalen
* Sla aangepaste formules op voor hergebruik
* Aangepaste formules zijn een Chloros+-functie; beschikbaarheid is afhankelijk van uw abonnementsniveau

Zie [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md).

### Exporteren

Bepaalt het bestandsformaat van de uitvoer.

**Beschikbare formaten**(instelling:**Gekalibreerd beeldformaat**, standaard**TIFF (16-bit)**):

* **TIFF (16-bit)**: Aanbevolen voor GIS en wetenschappelijke analyse
* **TIFF (32-bit, procent)**: Drijvende-kommagetallen
* **PNG (8-bit)**: Verliesloze compressie voor visualisatie
* **JPG (8-bit)**: Kleinste bestanden, compressie met verlies

De uitvoerbestanden worden opgeslagen in de projectmap, gegroepeerd per camera en formaat: `<project>/<camera>/<format>/<Product>_Images/`. Radiance wordt **altijd** als float32 opgeslagen in de map `tiff32`, ongeacht deze instelling. Geëxporteerde bestanden behouden de bronbestandsnaam — de map identificeert het product. Zie [De verwerking afronden](finishing-the-processing.md) voor de volledige uitvoerstructuur.

{% hint style="warning" %}
**Reflectiewaarden lezen**: de DN waarbij ρ = 1,0 is, hangt af van de broncamera — LATTICE gebruikt 32768 (aangeduid als XMP `Chloros:PixelScale`), Survey3 gebruikt 65535. Lees de tag in plaats van uit te gaan van een constante. Zie [Uitvoerbeeldformaten](../output-image-formats.md).
{% endhint %}

### DAQ-lichtsensor

In dit gedeelte worden alle DAQ-downwelling-bestanden (`.daq` / `.csv`) in je project weergegeven, één rij per bestand, met het sensormodel, de bestandsnaam en de **cap**-correctie van de diffuser die voor dat bestand van kracht is.

* **Cap-overschrijving (alle bestanden)**: één dropdownmenu dat voor het hele project geldt.**Auto** (standaard) gebruikt de geregistreerde cap van elk bestand — waar niets is geregistreerd, wordt aangenomen dat er zonneschijn was, aangezien alle MAPIR-DAQ’s worden geleverd met de zonneschijncorrector. Het selecteren van een cap overschrijft alle bestanden: ruwe opnames worden hiermee gecorrigeerd en opnames die al een cap bevatten, worden opnieuw gerefereerd (de geregistreerde correctie wordt ongedaan gemaakt en de geselecteerde cap wordt toegepast).
* Rijen geven een waarschuwing wanneer een geregistreerde limiet de veronderstelde standaard van de hub was in plaats van door de operator bevestigd, en wanneer de geselecteerde limiet geen profiel heeft voor dat apparaatmodel (de overschrijving wordt voor dat bestand geweigerd).

DAQ-opnames die zijn gemaakt in het tabblad ‘Lichtsensoren’ worden automatisch toegevoegd aan het geopende project, en geïmporteerde `.daq` / `.csv`-bestanden verschijnen hier zodra ze zijn toegevoegd.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Onderste projectinstellingen — Index, exportformaat, het DAQ-gedeelte ‘Lichtsensoren’ en de bedieningselementen voor projectsjablonen en -mappen</p></figcaption></figure>### Array-uitlijning

Dit gedeelte verschijnt **alleen** wanneer ten minste één afbeelding in het project de module-naar-module-uitlijningtransformatie bevat die LATTICE-arrays bij het vastleggen aanbrengen (`Chloros:Alignment*` XMP). Hier wordt weergegeven hoeveel afbeeldingen tags bevatten en welke camera als referentie dient, met de volgende bedieningselementen:

* **Array-uitlijning toepassen** (standaard: aan): pas op elk verwerkt product (debayered / preview / radiance / reflectance / index) een vervorming toe naar de gedeelde referentiegeometrie van de array. Uit = exporteren in de oorspronkelijke sensorgeometrie.
* **Bijsnijden tot gemeenschappelijke overlap** (standaard: aan): snijd uitgelijnde exporten bij tot het gebied dat alle modules delen, zodat elke band dezelfde voetafdruk heeft. Uit behoudt het volledige sensorbeeld (zwarte vulling buiten de bron).
* **Resampling**:**Bilineair (vloeiend, standaard)**,**Dichtstbijzijnde (exacte waarden behouden)**— geen vermenging tussen pixels, voor strikte radiometrische analyse — of**Kubisch (scherpst)**.***

## Instellingen opslaan en laden

### Projectsjabloon opslaan

Maak herbruikbare sjablonen voor consistente workflows:

1. Configureer alle gewenste instellingen in het paneel ‘Projectinstellingen’
2. Scrol naar de sectie **&quot;Projectsjabloon opslaan&quot;** onderaan
3. Voer een beschrijvende sjabloonnaam in (bijv. „Survey3N\_RGN\_Agriculture”)
4. Klik op het opslagpictogram

**Voordelen:**

* Pas identieke instellingen toe op meerdere projecten
* Deel configuraties met teamleden
* Zorg voor consistentie bij herhaalde enquêtes

### Sjabloon laden bij een nieuw project

Bij het aanmaken van een nieuw project:

1. Selecteer **&quot;Nieuw project&quot;** in het hoofdmenu
2. Kies een projectsjabloon in de optionele sjabloonkiezer
3. Alle instellingen uit het sjabloon worden automatisch toegepast

### Werkmap

De instelling **&quot;Werkmap&quot;** bepaalt waar nieuwe projecten standaard worden aangemaakt:

* **Standaardlocatie**: `C:\Users\[Username]\Chloros Projects`
* **Locatie wijzigen**: klik op het bewerkingspictogram en selecteer een nieuwe map
* **Gedeeld met CLI**: `chloros-cli` gebruikt dezelfde standaardinstelling voor de projectmap
* **Wanneer wijzigen**:
  * Netwerkschijf voor teamsamenwerking
  * Een ander station met meer opslagruimte
  * Georganiseerde mappenstructuur per jaar/klant

***

## PPK-configuratie (Post-Processed Kinematic)

Bij gebruik van MAPIR DAQ-recorders met GPS voor nauwkeurige geolocatie:

### Vereisten

* MAPIR DAQ met GPS (GNSS)-module
* .daq-logbestand met vermeldingen van belichtingspinnen
* Camera aangesloten op de belichtingspinnen van de DAQ tijdens de opnamesessie

### Configuratiestappen

1. Plaats het .daq-logbestand in uw projectmap
2. Schakel in Projectinstellingen het selectievakje **&quot;PPK-correcties toepassen&quot;** in
3. Stel indien nodig **&quot;Tijdzone-offset lichtsensor&quot;** in (standaard: 0 voor UTC)
4. Wijs camera&#x27;s toe aan belichtingspinnen:
   * **Eén camera**: wordt automatisch toegewezen aan pin 1
   * **Twee camera&#x27;s**: wijs elke camera handmatig toe aan de juiste pin**Toewijzing van belichtingspinnen:*** **Belichtingspin 1**: Selecteer het cameramodel uit de vervolgkeuzelijst
* **Belichtingspin 2**: Selecteer de tweede camera of „Niet gebruiken“
* Dezelfde camera kan niet aan beide pinnen worden toegewezen

{% hint style="warning" %}
**Belangrijk**: Belichtingspinnen moeten correct aan de bijbehorende camera’s worden toegewezen. Een onjuiste toewijzing leidt tot onjuiste geolocatiegegevens.
{% endhint %}

***

## Geavanceerde scenario’s

### Projecten met meerdere camera’s

Bij het verwerken van beelden van meerdere MAPIR-camera’s in één project:

1. Chloros detecteert automatisch elk cameramodel (zowel Survey3 als LATTICE)
2. Elke camera krijgt de juiste verwerkingsprofielen toegewezen en beschikt over een eigen structuur van uitvoermappen
3. PPK: Wijs elke Survey3-camera handmatig toe aan de juiste belichtingspin
4. Alle camera’s gebruiken hetzelfde exportformaat en dezelfde indexen

**Voorbeelden**: Survey3W RGN + Survey3N OCN opstelling met twee camera&#x27;s, of een LATTICE-opstelling waarin een RGB-master wordt gecombineerd met smalbandmodules

### Time-lapse- of onderzoeken met meerdere meetdata

Voor herhaalde onderzoeken van hetzelfde gebied in de loop van de tijd:

1. Maak een sjabloon aan met uw standaardinstellingen
2. Gebruik bij elke sessie een consistente opstelling van kalibratiedoelen
3. Verwerk elke datum als een afzonderlijk project
4. Gebruik identieke instellingen voor vergelijkbare resultaten
5. Exporteer in hetzelfde formaat voor temporele analyse

### Grote datasets

Voor projecten met veel beelden (500+):

* Overweeg om het project op te splitsen in kleinere projecten op basis van datum of gebied
* Gebruik Chloros+ parallelle verwerking voor snellere resultaten
* Overweeg CLI of API voor batchautomatisering
* Pas het minimale herkalibratie-interval aan om de detectietijd van doelen te verkorten

***

## Je instellingen controleren

Controleer deze belangrijke instellingen voordat je met de verwerking begint:

* [ ] Cameramodel correct gedetecteerd in de bestandsbrowser
* [ ] Vignetteringscorrectie ingeschakeld
* [ ] Reflectiekalibratie ingeschakeld
* [ ] Voor Survey3: ten minste één kalibratiedoelafbeelding geïmporteerd en gecontroleerd; voor LATTICE: een doel en/of een `.daq`-downwelling-opname aanwezig
* [ ] Gewenste multispectrale indices toegevoegd
* [ ] Exportformaat geschikt voor je workflow
* [ ] PPK-instellingen geconfigureerd (indien je .daq-bestanden met belichtingsgebeurtenissen gebruikt)

***

## Volgende stappen

Zodra je instellingen zijn geconfigureerd:

1. **Markeer kalibratiedoelafbeeldingen** - Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
2. **Start de verwerking** - Zie [De verwerking starten](starting-the-processing.md)
3. **Volg de voortgang** - Zie [De verwerking volgen](monitoring-the-processing.md)

Raadpleeg de referentiedocumentatie [Projectinstellingen](../project-settings/project-settings.md) voor volledige details over alle beschikbare instellingen.
