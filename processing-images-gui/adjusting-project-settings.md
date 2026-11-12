# Projectinstellingen aanpassen

Voordat u uw afbeeldingen gaat bewerken, is het belangrijk om uw projectinstellingen zo te configureren dat ze aansluiten bij uw workflow. Het paneel Projectinstellingen <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> biedt uitgebreide controle over kalibratie, bewerkingsopties, multispectrale indices en exportformaten.

## Projectinstellingen openen

1. Open uw project in Chloros
2. Klik op het pictogram **Projectinstellingen** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> in de linkerzijbalk
3. Het paneel Projectinstellingen toont alle configuratieopties

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Het paneel **Projectinstellingen** — Weergave, doelherkenning en verwerking</p></figcaption></figure>

{% hint style="info" %}
**Instellingen worden automatisch opgeslagen** samen met je project. Wanneer je een project opnieuw opent, worden alle instellingen hersteld.
{% endhint %}

***

## Snelle configuratie voor veelvoorkomende workflows

### Standaardinstellingen (aanbevolen voor de meeste gebruikers)

De standaardinstellingen werken goed voor typische Survey3- en LATTICE-workflows:

* ✅ **Vignetteringscorrectie**: Ingeschakeld
* ✅ **Reflectankalibratie / witbalans**: Ingeschakeld (maakt gebruik van MAPIR-doelen en/of DAQ-lichtsensorgegevens)
* ✅ **Debayer-methode**: Standaard (Snel, gemiddelde kwaliteit)
* ✅ **Exportformaat**: TIFF (16-bit) (TIFF (8-bit) voor een volledig 8-bit-project)
* ✅ **Alle exportproducten**: Ingeschakeld (LATTICE legt automatisch de fan-out vast in debayered, preview, radiance en reflectance)

Importeer gewoon je afbeeldingen en begin met verwerken met deze standaardinstellingen.

***

## Overzicht van projectinstellingen

Het paneel Projectinstellingen is onderverdeeld in de onderstaande secties. Twee extra secties — **DAQ-lichtsensor**en**Array-uitlijning** — verschijnen automatisch wanneer je project de relevante bestanden bevat. Zie [Projectinstellingen](../project-settings/project-settings.md) voor de volledige documentatie.

### Weergave

* **Resolutie miniatuurafbeeldingen**: Resolutie van de miniatuurafbeeldingen in het afbeeldingsraster. Opties:**Standaard (512 px)**,**1024 px**,**2048 px**,**Volledige resolutie**. Alleen voor weergave — heeft nooit invloed op de verwerking. Hogere waarden zien er scherper uit wanneer ingezoomd, maar laden langzamer.

### Doeldetectie

Bepaalt hoe Chloros-kalibratiedoelen in uw afbeeldingen identificeert.

**Belangrijkste instellingen:**

* **Minimaal kalibratiemonstergebied (px)**: Drempelwaarde voor doeldetectie (standaard:**25**, bereik 0–10000)
* **Minimale doelclustering (0–100)**: Drempelwaarde voor de gelijkenis bij het groeperen van doelgebieden (standaard:**60**)**Wanneer aanpassen:**

* Vergroot het steekproefgebied als er valse detecties optreden
* Verklein het als doelen niet worden gedetecteerd
* Pas de clustering aan als doelen in meerdere detecties worden opgesplitst

{% hint style="info" %}
Deze instellingen zijn grijs weergegeven wanneer **Reflectanciekalibratie / witbalans** is uitgeschakeld — als deze is uitgeschakeld, wordt doeldetectie helemaal niet uitgevoerd.
{% endhint %}

### Verwerking

Belangrijkste opties voor beeldverwerking en kalibratie.

**Belangrijkste instellingen:**

* **Vignetteringscorrectie**: Compenseert lensverduistering aan de randen ✅ Aanbevolen
* **Reflectankalibratie / witbalans**: Kalibreert beelden met behulp van gedetecteerde doelen (Survey3) en/of DAQ-lichtsensorgegevens (LATTICE) ✅ Aanbevolen
* **Debayer-methode**: Algoritme voor het omzetten van RAW naar 3-kanaals multispectraal
* **Minimaal herkalibratie-interval**: Minimale tijd in seconden tussen het gebruik van kalibratiedoelen (standaard:**0** = alles gebruiken, bereik 0–3600)**Niet-gekalibreerde alternatieve producten:**Wanneer een frame niet op reflectie kan worden gekalibreerd (geen doel beschikbaar of kalibratie uitgeschakeld), wordt het geëxporteerd als een van twee alternatieve producten —**er bestaat precies één van het paar per run**, gekozen door de schakelaar voor vignettcorrectie:

* **Sensorrespons exporteren**: schrijft `Sensor_Response_Images` — wordt gebruikt wanneer vignetcorrectie**uitgeschakeld** is
* **Vignettecorrectie exporteren**: schrijft `Vignette_Corrected_Images` — wordt gebruikt wanneer vignettecorrectie**aan** staat

Het selectievakje dat niet van toepassing is, is grijs weergegeven. Als u het vinkje bij het actieve selectievakje verwijdert, wordt dat bestand helemaal niet meer geschreven.

**LATTICE-exportproducten** (weergegeven voor elk project; deze zijn van toepassing op LATTICE-opnames):

* **Exporteer debayered**: het lineaire, gedebayerde beeld (`Debayered_Images`): de donkere offset van het kalibratiepakket is verwijderd, waarna demosaicing is toegepast, zonder flat field of unmix — hetzelfde bestand dat een opname opslaat op het gedebayerde niveau — in de oorspronkelijke oriëntatie van de sensor. Geldt voor RGB- en multispectrale modules.
* **Exporteren als voorbeeld**: het weergavevoorbeeld (`Preview_Images`). RGB = witbalans (DAQ-lichtbron indien beschikbaar, anders grijs-wereld) + gamma; multispectraal = valsekleurenuitrekking.
* **Stralingsintensiteit exporteren**: spectrale stralingsintensiteit in float32 (`Radiance_Images`, W/m²/sr/nm). Alleen multispectrale modules — niet van toepassing op RGB-masters.
* **Reflectantie exporteren**: uint16-reflectantie (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) wanneer een `.daq`-meting van neerwaartse straling of een doel binnen het beeld het beeld bestrijkt. Alleen multispectrale modules.

Alle vier zijn **standaard ingeschakeld**— één geïmporteerd LATTICE-ruwframe wordt in één verwerkingsronde uitgespreid naar elk ingeschakeld en toepasselijk product. Het selectievakje**Reflectantie exporteren** is grijs weergegeven wanneer Reflectantiekalibratie is uitgeschakeld. Instellingen die onmogelijk zijn vanwege de bovenliggende schakelaar, zijn altijd grijs weergegeven met een tooltip waarin de te wijzigen schakelaar wordt vermeld.

**Geavanceerde instellingen:**

* **Tijdzone-offset lichtsensor**: Aantal uren ten opzichte van UTC voor het afstemmen van de tijd van de lichtsensor (standaard: 0, bereik −12 tot +12)
* **PPK-correcties toepassen**: maakt gebruik van GPS-/belichtingspin-gegevens uit `.daq`-bestanden (standaard: uit)
* **Belichtingspin 1/2**: wijst camera’s toe aan belichtingspinnen voor opstellingen met twee camera’s

{% hint style="info" %}
**Het LATTICE-invoerniveau is automatisch.** LATTICE-opnames bevatten hun verwerkingsniveau in XMP-metadata, en de verwerking komt altijd bij het onbewerkte frame in de pijplijn terecht — er hoeft niets te worden geconfigureerd in de GUI. (De CLI-vlag `--input-level` is bedoeld als override voor gevorderde gebruikers bij opnames waarbij metadata verloren is gegaan; zie de [CLI-referentie](../reference/cli-reference.md).)
{% endhint %}

### Debayer-methode

We bieden momenteel 2 debayering-methoden aan in Chloros:

#### Standaard (Snel, gemiddelde kwaliteit)

De standaard debayer verwerkt snel, maar vertoont kleurruis door de debayering, wat resulteert in minder nauwkeurige en ruisrijkere beelden.

#### Textuurgevoelig (traag, hoogste kwaliteit) \[Alleen Chloros+]

Textuurgevoelig maakt gebruik van een hoogwaardige, randgevoelige debayer in combinatie met een AI/ML-ruisonderdrukkingsmodel dat vrijwel alle debayeringruis verwijdert. Het model heeft GPU-geheugen (VRAM) nodig om te draaien: met **7 GB of meer VRAM** kan het meerdere afbeeldingen tegelijk verwerken; bij minder dan 7 GB verwerkt het één afbeelding per keer (aanzienlijk langzamer). Zie [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-opnames maken altijd gebruik van de standaard-demosaic.** Er is geen door LATTICE getraind ‘textuurgevoelig’ model, dus de optie wordt niet aangeboden voor LATTICE-beelden — Survey3-beelden in hetzelfde project kunnen er nog steeds gebruik van maken.
{% endhint %}

### Index (multispectrale indices)

Configureer welke vegetatie-indices moeten worden berekend en geëxporteerd. De vervolgkeuzelijst in de GUI biedt **27 vooraf gedefinieerde indexformules**.**Indices toevoegen:**

1. Klik op de knop**"Index toevoegen"**

2. Selecteer een index uit het dropdown-menu (NDVI, NDRE, GNDVI, enz.)
3. Configureer de visualisatie-instellingen (LUT-kleuren, waardebereiken)
4. Voeg indien nodig meerdere indexen toe

**Populaire indexen:**

* **NDVI**: Algemene gezondheid van de vegetatie (meest gebruikelijk)
* **NDRE**: Vroegtijdige stressdetectie met RedEdge
* **GNDVI**: Gevoelig voor chlorofylconcentratie
* **OSAVI**: Werkt goed bij zichtbare bodem
* **EVI**: Gebieden met een hoge bladoppervlakte-index (LAI)

**Aangepaste formules:**

* Maak aangepaste multispectrale indexformules met bandberekeningen over alle beeldkanalen
* Sla aangepaste formules op voor hergebruik
* Aangepaste formules zijn een functie van Chloros+; beschikbaarheid is afhankelijk van uw abonnementsniveau

Zie [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md) voor alle beschikbare indexen en formules — inclusief welke namen alleen in de GUI beschikbaar zijn en welke ook in de CLI/SDK werken.

### Exporteren

Bepaalt het bestandsformaat van de uitvoer.

**Beschikbare formaten**(instelling:**Gekalibreerd beeldformaat**, standaard**TIFF (16-bit)**):

* **TIFF (16-bit)**: Aanbevolen voor GIS en wetenschappelijke analyse
* **TIFF (8-bit)**: 8-bit TIFF
* **TIFF (32-bit, Percent)**: Drijvende-kommagetallen
* **PNG (8-bit)**: Verliesloze compressie voor visualisatie
* **JPG (8-bit)**: Kleinste bestanden, compressie met verlies

De lijst toont welke bestandsformaten de afbeeldingen van het project kunnen bevatten: Chloros voert geen opschaling van de bitdiepte uit. Voor alle 8-bit afbeeldingen zijn alleen TIFF (8-bit), PNG (8-bit) en JPG (8-bit) beschikbaar, waarbij TIFF (8-bit) de standaardinstelling is. In een gemengd project wordt elke opname geëxporteerd met de eigen diepte of een lagere. Radiance blijft 32-bit float.

De uitvoerbestanden worden opgeslagen in de projectmap, gegroepeerd per camera en formaat: `<project>/<camera>/<format>/<Product>_Images/`. Radiance wordt **altijd** als float32 opgeslagen in de map `tiff32`, ongeacht deze instelling. Geëxporteerde bestanden behouden de bronbestandsnaam — de map identificeert het product. LATTICE-multispectrale exporten kunnen ook per-pixel-clampmaskers opslaan in een `clamp_masks/`-map naast de productmappen. Zie [De verwerking afronden](finishing-the-processing.md) voor de volledige uitvoerstructuur.

{% hint style="warning" %}
**Reflectiewaarden aflezen**: de DN-waarde die ρ = 1,0 aangeeft, is afhankelijk van de broncamera — LATTICE gebruikt 32768 (aangeduid als XMP `Chloros:PixelScale`), Survey3 gebruikt 65535. Lees de tag in plaats van uit te gaan van een constante waarde. Zie [Uitvoerbeeldformaten](../output-image-formats.md).
{% endhint %}

### DAQ-lichtsensor

In dit gedeelte worden alle DAQ-downwelling-bestanden (`.daq` / `.csv`) in je project weergegeven, één rij per bestand, met het sensormodel, de bestandsnaam en de **cap** die voor dat bestand van kracht is.

* **Cap-overschrijving (alle bestanden)**: één projectbrede vervolgkeuzelijst.**Auto** (standaard) gebruikt de geregistreerde cap van elk bestand — er wordt uitgegaan van zonneschijn als er niets is geregistreerd, maar DAQ-U, DAQ-M en DAQ-E worden geleverd zonder cap en Chloros kan niet zien wat er is ingesteld, dus geeft de verwerking een waarschuwing bij een veronderstelde cap: kies de cap die is ingesteld. Het selecteren van een cap overschrijft elk bestand behalve die van een DAQ-E-S, waarvan de diffuser deel uitmaakt van de kalibratie: ruwe opnames worden hiermee gecorrigeerd en opnames die al een cap bevatten, worden opnieuw gerefereerd (de geregistreerde correctie wordt ongedaan gemaakt en de geselecteerde cap wordt toegepast).
* Rijen geven een waarschuwing wanneer een geregistreerde cap de veronderstelde standaard van de hub was in plaats van door de gebruiker bevestigd, en wanneer de geselecteerde cap geen profiel heeft voor dat apparaatmodel (de overschrijving wordt voor dat bestand geweigerd).

DAQ-opnames die zijn gemaakt in het tabblad ‘Lichtsensoren’ worden automatisch toegevoegd aan het geopende project, en geïmporteerde `.daq` / `.csv`-bestanden verschijnen hier zodra ze zijn toegevoegd.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Onderste projectinstellingen — Index, exportformaat, het DAQ-gedeelte ‘Lichtsensoren’ en de bedieningselementen voor projectsjablonen/mappen</p></figcaption></figure>### Array-uitlijning

Dit gedeelte verschijnt **alleen** wanneer ten minste één afbeelding in het project de module-naar-module-uitlijningstransformatie bevat die LATTICE-arrays bij het vastleggen aanbrengen (`Chloros:Alignment*` XMP). Hier wordt weergegeven hoeveel beelden tags bevatten en welke camera als referentie dient, met de volgende bedieningselementen:

* **Array-uitlijning toepassen** (standaard: aan): pas op elk verwerkt product (debayered / preview / radiance / reflectance / index) een vervorming toe naar de gedeelde referentiegeometrie van de array. Uit = exporteren in de oorspronkelijke sensorgeometrie.
* **Bijsnijden tot gemeenschappelijke overlap** (standaard: aan): snijd uitgelijnde exporten bij tot het gebied dat alle modules delen, zodat elke band dezelfde voetafdruk heeft. Uit behoudt het volledige sensorbeeld (zwarte vulling buiten de bron).
* **Hersampling**:**Bilineair (vloeiend, standaard)**,**Dichtstbijzijnde (exacte waarden behouden)**— geen vermenging tussen pixels, voor strikte radiometrische analyse — of**Kubisch (scherpst)**.

***

## Instellingen opslaan en laden

### Projectsjabloon opslaan

Maak herbruikbare sjablonen voor consistente workflows:

1. Configureer alle gewenste instellingen in het paneel **Projectinstellingen**

2. Scrol naar de sectie**"Projectsjabloon opslaan"** onderaan
3. Voer een beschrijvende sjabloonnaam in (bijv. „Survey3N\_RGN\_Agriculture”)
4. Klik op het opslagpictogram

**Voordelen:**

* Pas identieke instellingen toe op meerdere projecten
* Deel configuraties met teamleden
* Behoud consistentie bij herhaalde metingen

### Sjabloon laden bij een nieuw project

Bij het aanmaken van een nieuw project:

1. Selecteer **"Nieuw project"** in het hoofdmenu
2. Kies een projectsjabloon in de optionele sjabloonkiezer
3. Alle instellingen uit het sjabloon worden automatisch toegepast

### Werkmap

De instelling **"Werkmap"** bepaalt waar nieuwe projecten standaard worden aangemaakt:

* **Standaardlocatie**: `Documents\MAPIR\Chloros Projects` in uw Windows-map ‘Documenten’ (meestal `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; deze volgt de map ‘Documenten’ als OneDrive of een roamingprofiel deze verplaatst)
* **Locatie wijzigen**: klik op het bewerkingspictogram en selecteer een nieuwe map
* **Gedeeld met de CLI**: `chloros-cli` gebruikt dezelfde standaardinstelling voor de projectmap
* **Houd het kort in Windows**: tenzij lange paden in Windows zijn ingeschakeld, wordt een projectmap met een naam die langer is dan ongeveer 130 tekens geweigerd vóór de verwerking — zie [De verwerking starten](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Wanneer wijzigen**:
  * Netwerkschijf voor teamsamenwerking
  * Andere schijf met meer opslagruimte
  * Georganiseerde mappenstructuur per jaar/klant

***

## PPK (Post-Processed Kinematic) instellen

Bij gebruik van MAPIR DAQ-recorders met GPS voor nauwkeurige geolocatie:

### Vereisten

* MAPIR DAQ met GPS (GNSS)-module
* .daq-logbestand met vermeldingen van belichtingspinnen
* Camera aangesloten op de belichtingspinnen van de DAQ tijdens de opnamesessie

### Configuratiestappen

1. Plaats het .daq-logbestand in je projectmap
2. Schakel in de projectinstellingen het selectievakje **"PPK-correcties toepassen"** in
3. Stel indien nodig **"Tijdzone-offset lichtsensor"** in (standaard: 0 voor UTC)
4. Wijs camera’s toe aan belichtingspinnen:
   * **Eén camera**: wordt automatisch toegewezen aan pin 1
   * **Twee camera’s**: wijs elke camera handmatig toe aan de juiste pin

**Toewijzing van belichtingspinnen:**

* **Belichtingspin 1**: Selecteer het cameramodel uit de vervolgkeuzelijst
* **Belichtingspin 2**: Selecteer de tweede camera of “Niet gebruiken”
* Dezelfde camera kan niet aan beide pinnen worden toegewezen

{% hint style="warning" %}
**Belangrijk**: Belichtingspinnen moeten correct aan de bijbehorende camera’s worden toegewezen. Een onjuiste toewijzing leidt tot verkeerde geolocatiegegevens.
{% endhint %}

***

## Geavanceerde scenario’s

### Projecten met meerdere camera’s

Bij het verwerken van beelden van meerdere MAPIR-camera’s in één project:

1. Chloros detecteert automatisch elk cameramodel (zowel Survey3 als LATTICE)
2. Elke camera krijgt de juiste verwerkingsprofielen en elke camera krijgt zijn eigen mapenstructuur voor de uitvoer
3. PPK: Wijs elke Survey3-camera handmatig toe aan de juiste belichtingspin
4. Alle camera’s gebruiken hetzelfde exportformaat en dezelfde indexen

**Voorbeelden**: een dual-camera-opstelling met Survey3W RGN + Survey3N OCN, of een LATTICE-array waarin een RGB-master wordt gecombineerd met smalbandmodules

### Time-lapse- of metingen op meerdere data

Voor herhaalde metingen van hetzelfde gebied in de loop van de tijd:

1. Maak een sjabloon aan met uw standaardinstellingen
2. Gebruik bij elke sessie een consistente opstelling van kalibratiedoelen
3. Verwerk elke datum als een afzonderlijk project
4. Gebruik identieke instellingen voor vergelijkbare resultaten
5. Exporteer in hetzelfde formaat voor temporele analyse

### Grote datasets

Voor projecten met veel beelden (500+):

* Overweeg om de dataset op te splitsen in kleinere projecten per datum of gebied
* Gebruik de parallelle verwerking van Chloros+ voor snellere resultaten
* Overweeg CLI of API voor batchautomatisering
* Pas het minimale herkalibratie-interval aan om de detectietijd van doelen te verkorten

***

## Uw instellingen controleren

Controleer deze belangrijke instellingen voordat u begint met verwerken:

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

1. **Markeer kalibratiedoelafbeeldingen** – Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
2. **Start de verwerking** – Zie [De verwerking starten](starting-the-processing.md)
3. **Volg de voortgang** - Zie [De verwerking volgen](monitoring-the-processing.md)

Raadpleeg de referentiedocumentatie [Projectinstellingen](../project-settings/project-settings.md) voor volledige details over alle beschikbare instellingen.
