# Camera-instellingen

Het tabblad **Camera&#x27;s**is het live-bedieningspaneel van Chloros voor LATTICE-camera&#x27;s: een hoofdweergavegebied waarin elke aangesloten camera als een live-tegel wordt weergegeven, en een zijbalk waarmee je tussen drie pagina&#x27;s kunt schakelen — de**cameralijst**, een**instellingenpaneel**(instellingen per camera, array of opname — één tegelijk), en de**Indexcalculator**. Op deze pagina worden alle bedieningselementen in de cameralijst, het instellingenvenster per camera en het instellingenvenster voor de array beschreven. Opnamemodi, de selectie van het exporttype en de ‘Capture All’-workflow staan op de bijbehorende pagina [Opname-instellingen &amp; -modi](capture.md).

Het tabblad ‘Camera’s’ verschijnt in de zijbalk zodra de Chloros-backend gereed is. Alle onderstaande bedieningselementen communiceren met de lokale backend via `127.0.0.1:5000`; wijzigingen worden onmiddellijk toegepast op de live camera, tenzij anders vermeld.

## Cameratypen die op deze pagina worden gebruikt

De bedieningselementen worden al dan niet weergegeven, afhankelijk van het geselecteerde cameratype. In de handleiding worden doorlopend de volgende termen gebruikt:

| Term | Betekenis | Filterkanalen |
| --- | --- | --- |
| **RGB-camera** | LATTICE M3C met het FRGB-filter (model bevat `-FRGB`) | Red / Green / Blue |
| **Bayer multispectraal** | LATTICE M3C met FRGN, FOCN of FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — één smalbandfilter, één gekalibreerde band | Enkele band |
| **Array-lid** | Een camera die is aangesloten als onderdeel van een gesynchroniseerde array (gecombineerde of afzonderlijke weergave) | Per filter |

RGB-camera’s ondergaan fotometrische verwerking (witbalans, kleurprofielen, gamma); multispectrale en monochrome camera’s doorlopen de radiometrische keten en slaan de fotometrische instellingen over. Array-leden geven instellingen op streamniveau (pixelformaat, resolutie, binning, trigger, framesnelheid) door aan de array — die rijen worden alleen-lezen in het paneel per camera en worden in plaats daarvan verplaatst naar het paneel met array-instellingen.

## Het hoofdvenster

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

voor de feed Als er geen camera’s zijn aangesloten, toont het feedvenster een **&quot;Sluit een camera aan om te beginnen&quot;**-startscherm met twee knoppen:**Camera aansluiten**(groen, opent het dialoogvenster voor het aansluiten van één camera) en**Array aansluiten** (blauw, opent het dialoogvenster voor het aansluiten van een array). De verbindingsdialoogvensters zelf worden beschreven in [Camera’s aansluiten](connecting.md); array-concepten (synchronisatie, niveaus, bandbreedte) in [Multi-camera-arrays](arrays.md). Wanneer u een opgeslagen project opent dat camera’s bevat, toont de splash-scherm in plaats daarvan een draaiend pictogram met de tekst „N opgeslagen camera’s opnieuw openen…“, terwijl Chloros de streams van de laatste sessie herstelt.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Bovenste balk

| Bedieningselement | Wat het doet |
| --- | --- |
| **Schakelaar weergavemodus**| Schakelt tussen**rasterweergave**(alle tegels als cellen) en**lijstweergave** (arrays over de volledige breedte bovenaan, ÉÉN actieve camera daaronder). Tooltips: &quot;Overschakelen naar rasterweergave&quot; / &quot;Overschakelen naar lijstweergave&quot;. |
| **Rastervergrendeling**(hangslot) | Standaard**vergrendeld** — tegels staan vast. Ontgrendel om tegels naar een willekeurige positie te slepen (tussenruimtes blijven behouden). Het raster wordt automatisch opnieuw vergrendeld telkens wanneer een nieuwe camera verbinding maakt. Tooltips: &quot;Raster ontgrendelen (tegels verslepen inschakelen)&quot; / &quot;Raster vergrendelen (tegels op hun plaats vastzetten)&quot;. |
| **Feed Zoom**-schuifregelaar | Tegelgrootte, van 60 px tot de volledige breedte van de container. Tegels behouden een beeldverhouding van 4:3. Bij een tegelbreedte van minder dan 200 px worden de overlays met de naam en fps verborgen om de tegel overzichtelijk te houden. |

### Feed-tegels

Elke camera geeft een samengestelde live-tegel weer; een camera kan bovendien drie grijswaarden-tegels met **kanaalsplitsing** weergeven (zie [Kanaalsplitsingen](#display-overlays-drawn-over-the-live-feed)), en arrays geven een gecombineerde tegel weer. De actieve tegel is voorzien van een selectiering in de kleur van de camera (of array).

Als je met de muis over een tegel beweegt, verschijnt er een **X**-sluitknop:

* Als je een **samengestelde** tegel sluit terwijl de kanaalsplitsingen nog zichtbaar zijn, wordt alleen de samengestelde tegel verborgen.
* Als je de **laatst zichtbare tegel van een op zichzelf staande camera** sluit, wordt de verbinding met die camera verbroken.
* **Gesplitste tegels van een gecombineerde array verbreken nooit de verbinding** met de camera — ze worden alleen verborgen.

Als het raster ontgrendeld is, kun je elke tegel naar elk vak slepen; de indeling wordt samen met het project opgeslagen.

## Zijbalk — cameralijst

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Op de eerste pagina van de zijbalk staan alle aangesloten camera’s en arrays vermeld:

* **Camera verbinden**(groen) /**Array verbinden** (blauw, toont „Detecteren...“ tijdens het scannen). Beide zijn uitgeschakeld zolang een verbindingsdialoogvenster open is.
* **Alles vastleggen** (rood) — legt alle weergegeven camera’s vast met de exporttypes die zijn gekozen in de vastleginstellingen. Hiervoor moet er een project geopend zijn. Volledig gedocumenteerd in [Vastleginstellingen &amp; modi](capture.md).
* **Tandwiel voor vastleginstellingen** (naast Alles vastleggen) — opent het [venster Opname-instellingen](capture.md#the-capture-settings-pane). Uitgeschakeld zonder project of tijdens het vastleggen.

### Camerarijen

Elke camerarij toont een kleurgecodeerde rand (de aangepaste kleur van de camera), een ‘CAM’-label — met een blauwe **M**(master) of groene**S** (slave) als rolletter voor array-leden — en de weergavenaam. De standaardnaam is `LATTICE-MODEL (serial)`; hernoem deze via het instellingenvenster per camera. Rijknoppen:

| Knop | Effect |
| --- | --- |
| **Oog**| Zichtbaarheid in- of uitschakelen. Verborgen camera’s verdwijnen uit het raster en worden**uitgesloten van ‘Alles vastleggen’**. |
| **Tandwiel** | Het instellingenvenster per camera openen (volgende sectie). |
| **Pauze / Afspelen**| Bevries het livevoorbeeld**alleen aan de weergavezijde** — de opname op de server blijft doorgaan. Gepauzeerde camera’s kunnen niet opnemen. |
| **X** | Verbinding verbreken. De gebruikersinterface wordt onmiddellijk bijgewerkt (optimistisch); het verbreken van de verbinding op de server kan 10–30 s duren. |

### Rijen in de array

Een rij in de array toont een „ARRAY“-badge in de kleur van de array, de naam van de array (aanpasbaar in de array-instellingen) en een **DAQ · aan/uit**-knop —**aan** wanneer de lichtsensor op array-niveau is ingesteld *of* wanneer een lid een camera-specifieke sensor heeft; in de tooltip staat precies welke sensor wat doorgeeft. De camera’s van de leden worden daaronder met inspringing weergegeven in hun eigen rijen. Knoppen in de array-rij: **oog**(verbergt/toont ALLE leden tegelijk),**tandwiel**(venster met array-instellingen),**X**(de hele array loskoppelen).

De status van de lichtsensor (DLS) die in de array-rijen en het venster met array-instellingen wordt gebruikt, kent vier toestanden:**uit**,**in afwachting**(nog geen spectrum),**actief**(er is binnen de laatste 3 s een spectrum binnengekomen), en**verouderd** — geen nieuw spectrum in de afgelopen 3 seconden, maar de laatste meting wordt *nog steeds gebruikt* (DAQ-metingen vervallen nooit in het vastleggingspad).

Je kunt losstaande camera’s en hele arraygroepen in de zijbalk langs elkaar slepen om de lijst te herschikken; arrayleden kunnen niet afzonderlijk worden versleept.

## Instellingenvenster per camera

Open dit venster met het **tandwieltje** in een camerarij. Het venster schuift over de cameralijst.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Koptekst**: het**kleurstaal**van de camera (klik om een native kleurenkiezer te openen — hiermee stel je de kleur van de zijbalkrand en de selectiering van de tegels in), de**naam**met een potloodknop**Hernoemen**(als je een lege naam opslaat, wordt de standaardnaam `MODEL (serial)` hersteld) en**×** om te sluiten.

### Live histogram

Bovenaan het paneel staat een live luma-histogram dat wordt berekend op basis van het JPEG-voorbeeld met een frequentie van ~8 Hz. Het gemiddelde is Bayer-gewogen — (R+2G+B)/4 — om overeen te komen met de eigen AE-meting van de camera.

* **Orange stippellijn**= het AE-doel.**Sleep deze horizontaal om het doel aan te passen** — bij het loslaten wordt één commando verzonden, en tijdens het slepen wordt de AE-doelmodus omgeschakeld naar Handmatig.
* **Green ononderbroken lijn** = de werkelijke gemiddelde luma (wat de AE momenteel levert).
* **RGB-knop** (rechtsboven): schakelt tussen bandgewijze overlay-histogrammen, gekleurd volgens het filter van de camera (bijv. bij FRGN: grijs NIR, groen, rood). Bij monochrome (M3M) camera’s staat er „MONO“ op de knop en is deze uitgeschakeld — bij monochrome camera’s wordt altijd het luma-histogram van één band weergegeven.
* De labels op de X-as volgen de bitdiepte van de sensor van het huidige pixelformaat: 0..255, 0..1023, 0..4095 of 0..65535.

### Rijen

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

met camera-informatie | Rij | Gedrag |
| --- | --- |
| **Model** | Alleen-lezen (bijv. `LATT-M3C-L87-FRGN`). |
| **Radiometrische kalibratie**| Green**&quot;Actief&quot;**-badge met een bijschrift dat het kalibratieniveau, de hash, de kalibratiedatum en de bandenlijst weergeeft, geladen vanuit het kalibratiepakket van de camera (zie [Fabrieksradiometrische kalibratie](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Verborgen voor RGB-camera&#x27;s** — deze beschikken over een fotometrische witbalanskalibratie, niet over stralingskalibratie per band. |
| **Kalibratierapport**|**Download**-knop — opent het NIST-kalibratiecertificaat per serienummer van de camera als PDF in de viewer van uw besturingssysteem. Als het certificaat nog niet in de cache is opgeslagen, geeft Chloros in plaats daarvan een hint weer. |
| **Serienummer** | Alleen-lezen. |
| **Firmware**| Toont de huidige versie en zoekt vervolgens de beschikbare versie voor dit model op (in de cache per model — een reeks van N camera’s controleert de server één keer). Status: &quot;Bezig met controleren…&quot; →**&quot;Update naar X&quot;**-knop → &quot;Bezig met flashen…&quot; → &quot;Bijgewerkt van A naar B&quot; / &quot;Mislukt: …&quot; / &quot;Overgeslagen: …&quot; / groene**&quot;Up-to-date&quot;**. De tooltip bij de updateknop: &quot;Fabrieksinstellingen herstellen + flashen + UserSet1 herprogrammeren. ~2–3 minuten; niet loskoppelen.&quot; |
| **IP** | Alleen-lezen. |
| **Temperatuur** | Alleen-lezen, wordt elke 3 s vernieuwd. Wordt oranje bij ≥65 °C en rood met een ⚠ bij ≥75 °C. |
| **Kalibratiedoel** selectievakje | Schakelt ArUco-reflectiedetectie in met een validatietabel per paneel (NDVI) onder de liveweergave (lijstweergave). Alleen voor deze sessie — is altijd uitgeschakeld. |
| Keuzelijst **Lichtsensor** | Koppelt een DAQ-lichtsensor (DAQ-E/M/U, uit de lijst op het tabblad Lichtsensoren) aan deze camera voor verlichtingscorrectie bij neerwaarts invallend licht (DLS) en voorspellende automatische belichting. &quot;Geen&quot; verwijdert de koppeling. Als er geen sensoren zijn aangesloten, toont de keuzelijst &quot;(geen sensoren aangesloten — open het DAQ-tabblad)&quot;. De koppeling wordt samen met het project opgeslagen. |

### Belichting &amp; Versterking

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alle numerieke invoervelden hier maken gebruik van draaiknoppen die versnellen bij lang indrukken: tikken = ±1, langer dan 1,5 s ingedrukt houden = ±10, langer dan 3 s ingedrukt houden = ±100. De waarde wordt naar de camera verzonden wanneer u loslaat.

| Regelaar | Bereik / keuzes | Standaard | Van toepassing op | Wat het doet |
| --- | --- | --- | --- | --- |
| **Belichting (us)**| De live min./max. van de camera | Auto | Alle | Belichtingstijd in microseconden, met een**Auto/Handmatig**-schakelaar. Auto = continue automatische belichting door de camera. |
| **Versterking (dB)**| De live min./max. van de camera (bijv. tot 48 dB) | Handmatig (uit) | Alle | Analoge/digitale versterking met een eigen**Auto/Handmatig**-schakelaar. |
| **AE-doelhelderheid**| 0–255 | 80, modus**Auto**| Alle (bewerkbaar wanneer AE of automatische versterking is ingeschakeld) | De helderheid waarop de AE zich richt. In**Auto**(de standaardinstelling) kiest een op het histogram gebaseerde backend-controller zelf de doelwaarde, waarbij de belichting op 60–75 % van het maximum van de sensor wordt gehouden. Door een waarde in te voeren of de oranje lijn van het histogram te verslepen, schakelt u over naar**Handmatig**. |
| **AE-afvlakking** | 0,5–40, stap 0,1 | 8,0 | Alles | AE-demping. Tooltip: „Lager = AE reageert sneller (kan bij hoge fps pulseren). Hoger = vloeiender / langzamer.“ Waarden die ver onder de standaardinstelling liggen, kunnen ervoor zorgen dat de AE schommelt en de stream bij hoge framesnelheden destabiliseert; 8,0 is de stabiele standaardinstelling. |
| **AE-regio van belang**| Selectievakje ‘Inschakelen’ +**Richten**-knop | Uit | Alles | Indien ingeschakeld, meet de AE alleen het groen gestreepte gebied in plaats van het hele beeld.**Richten** activeert ‘klikken om te plaatsen’ op de livefeed: een klik centreert een gebied op 30 % van het beeld; klikken en slepen tekent een aangepaste rechthoek (minimaal 5 % × 5 %). Aim wordt automatisch uitgeschakeld na één plaatsing. Het gebied wordt teruggeprojecteerd naar de oorspronkelijke cameracoördinaten volgens de door jou ingestelde rotatie/spiegeling, en wordt samen met het project opgeslagen. |
| **AE-afstemsnelheid** | 0,1–5, stap 0,1 | 1,0 | Alleen voor leden van de array | Hoe snel het automatische AE-doel veranderingen in de helderheid van de scène volgt; bij 1,0× wordt elke 2,5 s opnieuw gecontroleerd. |
| **Highlightbescherming** | Streng (1 %) / Normaal (5 %) / Soepel (15 %) | Streng | Camera’s die deze instelling ondersteunen | Hoeveel van het beeld mag naar wit clippen voordat de AE het beeld donkerder maakt. |

{% hint style="info" %}
**Lichtvereiste voor multispectrale Bayer-camera’s (RGN / OCN / NGB):** de scène moet in alle drie de kanalen voldoende licht hebben, anders werkt de kalibratie niet correct — één enkele sensorbelichting bestrijkt alle drie de spectra. Gebruik een DAQ-lichtsensor om je licht te meten, of ga volledig mono (M3M) werken, zodat elke band zijn eigen belichting krijgt. Als een opname hier niet aan voldoet, detecteert Chloros dit en waarschuwt het je (de ‘unmix-clamp’-melding).
{% endhint %}

### Pixelformaat en

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

resolutie**Array-leden** tonen de alleen-lezen rijen &#x27;Huidige&#x27; (formaat + WxH) en &#x27;Binning&#x27; met de opmerking &#x27;Ingesteld in array-instellingen&#x27; — een herstart van de stream op één lid zou de synchronisatie verstoren, dus deze worden beheerd in het [paneel met array-instellingen](#array-settings-pane).**Standalone camera’s** krijgen:

| Instelling | Keuzes | Wat het doet |
| --- | --- | --- |
| **Pixelformaat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Pixelformaat van de sensor (bitdiepte). |
| **Resolutie** | Volledig / Half / Kwart | Relatief ten opzichte van de huidige binning: Volledig = 2048/N × 1536/N voor N×N-binning. |
| **Binning** | 1x1 (geen) / 2x2 / 4x4 | Hardware N×N-binning — hogere waarden verlagen de resolutie, maar verbeteren de signaal-ruisverhouding (SNR) en de beeldsnelheid. Als je dit wijzigt, wordt de stream opnieuw gestart en worden alle ROI’s gereset naar het nieuwe volledige gezichtsveld. |
| **Huidig** | alleen-lezen | De daadwerkelijke WxH en (x, y) offset die van kracht zijn. |

### Livevoorbeeld

Alles in dit gedeelte is **alleen aan de weergavezijde**— het verandert wat je in de livefeed ziet, terwijl opgeslagen opnames lineair en ongewijzigd blijven — met één uitzondering:**Vignette** is radiometrisch en heeft ook invloed op exportbestanden (zie hieronder).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Instelling | Bereik / keuzes | Standaard | Van toepassing op | Wat het doet |
| --- | --- | --- | --- | --- |
| **Weergaveresolutie** | 360p (snelst) / 480p / 720p / 1080p / Native sensorresolutie (traagst) | 720p | Alles | De hoogte waarop de backend de radiometrische previewketen uitvoert. Een lagere instelling levert een hogere framesnelheid op zonder het gezichtsveld te wijzigen. |
| **Index**| Selectievakje inschakelen + tandwiel | Uit | Alleen Bayer-multispectraal,**geen** gecombineerde-array-leden | Live voorbeeld van de vegetatie-index. Het tandwiel opent het gedeelde [Indexcalculator](#index-calculator-pane) dat vooraf is geladen met de natuurlijke filterbanden van de camera (bijv. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). De aangepaste uitdrukking plus LUT (aan/uit, standaardniveau 3, standaardminimum 0,2, max standaard 1) wordt bij elk voorbeeldframe berekend. Leden van een gecombineerde array verbergen deze rij — de array heeft één gedeelde index. |
| **Witbalans** | Uit / Eenmalig / Continu + een knop voor opnieuw vastleggen | Continu | Alleen RGB | Live witbalans. Met de vernieuwingsknop wordt de witbalans opnieuw vastgelegd op basis van het huidige DLS-spectrum (uitgeschakeld wanneer de modus op Uit staat). |
| **Gamma** | Aan / Uit | Aan | Alleen RGB | Gamma (γ = 2,2 LUT) weergeven in het live-voorbeeld. Opgeslagen opnames blijven lineair. |
| **Ruisonderdrukking** | Selectievakje + sterkte 0–100 | Uit / 50 | Alle (per camera, zelfs binnen arrays) | Bilateraal filter in het livevoorbeeld. Hoger = vloeiender maar zachtere details. |
| **Scherpte** | Selectievakje + sterkte 0–100 | Uit / 30 | Alle | Onscherp masker in het livevoorbeeld, als laatste toegepast. Kan ruis versterken. Alleen in het voorbeeld. |
| **Vignettering**| Selectievakje + sterkte 0–100 | Uit / 0 | Alles | Handmatige correctie van resterende vignettering (maakt hoeken lichter), bovenop de Smart Vignette-schatting van de reeks.**Radiometrisch — beïnvloedt zowel de liveweergave als de export**, in tegenstelling tot Ruisonderdrukking/Scherpte. |
| **Kleurprofiel** | Raw / Lineair / Natuurlijk / Verbeterd / Aangepaste temperatuur | Natuurlijk | Alleen RGB | Zie hieronder. |
| **Kleurtemperatuur** | 2000–10000 K, stap 100 | 5500 K | Alleen RGB, aangepast temperatuurprofiel | Zet de witbalans vast op een vaste gecorreleerde kleurtemperatuur (DLS-invoer wordt genegeerd). De laatst gekozen Kelvin-waarde wordt onthouden bij het wisselen van profiel. |
| **Verzadiging** | 0–200 (100 = neutraal) | 100 | Alleen RGB | HSV-verzadiging in het livevoorbeeld. |
| **Contrast** | 0–200 (100 = neutraal) | 100 | Alleen RGB | Lineair contrast rond middengrijs in het livevoorbeeld. |
| **Spiegelen H / Spiegelen V** | Selectievakjes | Uit | Alle | Het voorbeeld horizontaal / verticaal spiegelen. |
| **Rotatie**| 0° / 90° / 180° / 270° | 0° | Alle | Het voorbeeld roteren. De oriëntatie wordt toegepast aan het einde van de backend-voorbeeldketting —**opgeslagen opnames behouden de oorspronkelijke oriëntatie van de camera**, en samengestelde weergaven negeren deze. |**Semantiek van kleurprofielen** (RGB-camera’s):

* **Raw** — de verwerkingsketen volledig omzeilen.
* **Lineair** — donker-signaal + flat-field + witbalans; geen kleurmatrix, geen gamma.
* **Natuurlijk** *(standaard)* — lineair plus de gemeten kleurcorrectiematrix en een aan de scène aangepaste tooncurve.
* **Verbeterd**— Natuurlijk plus levendigheid en lokaal CLAHE-contrast. De extra kosten gelden**alleen voor de live-preview** — opgeslagen opnames krijgen altijd de volledige afwerking, ongeacht het profiel.
* **Aangepaste temperatuur** — Natuurlijk met witbalans vastgezet op de door jou gekozen Kelvin-waarde.

{% hint style="warning" %}
Voor Natuurlijk, Verbeterd en Aangepaste temperatuur toont het venster een opmerking over de toon: frames worden lichter gemaakt op basis van hun eigen scène, dus opgeslagen *weergave*-afbeeldingen zijn niet frame-voor-frame vergelijkbaar. **Exporteer straling of reflectie voor metingen.**
{% endhint %}

### Weergave-overlays (over de livefeed getekend)

Deze zijn alleen in de frontend beschikbaar — ze worden over de video heen getekend en hebben geen invloed op de stream of opnames.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlay | Bedieningselementen | Standaard | Wat het doet |
| --- | --- | --- | --- |
| **Zebra** | Selectievakje + drempelwaarde 200–255 | Uit / 250 | Magenta diagonale strepen op afgeknipte pixels. |
| **Kruisdraad** | Selectievakje | Uit | Markering in het midden van het frame. |
| **Raster** | Uit / 3 × 3 / 9 × 9 | Uit | Compositieraster. |
| **Histogram** | Selectievakje + breedte 0,10–0,90 van het beeld | Uit / 0,25 | Een histogramstrook tijdens het bekijken. |
| **Focus Peak** | Selectievakje + drempelwaarde 20–200 + kleurstaal | Uit / 80 / `#ff5722` | Sobel-randmarkering voor scherpstellen. |
| **Kanaalsplitsingen** | &quot;Splitsingen weergeven (Red / Green / NIR)&quot; / &quot;Splitsingen verbergen&quot;-knop | Verborgen | Voegt drie onafhankelijke grijswaardetegels per kanaal toe naast de samengestelde (het label van de knop volgt de filterkanalen van de camera). Elk gesplitst vakje kan worden versleept en heeft dezelfde randkleur als de camera. Niet beschikbaar op monokameras. Wordt samen met het project opgeslagen. |

### Spotmeter

* Selectievakje **Klik om te meten**: klik op het livebeeld om één pixel te meten (dit wordt gemarkeerd met een dradenkruis), of klik en sleep over een gebied voor een pixelgemiddelde. Met**Wissen**worden de meting en het dradenkruis verwijderd. Sluit de**Aim**-modus van AE-ROI uit.
* **Show**-dropdown:**Raw (bitdiepte)**— native digitale waarden met de bitdiepte van de sensor (bijv. 12-bit → 0,,4095) — of**Display (8-bit)** (standaard). Wanneer een live-index actief is, toont Display in plaats daarvan de berekende indexwaarde (bijv. NDVI).
* Het uitleespaneel toont pixelcoördinaten, framegrootte, pixelformaat, bitdiepte en een kanaaltabel (Chan / Waarde / %) met bandlabels en golflengten; Bayer-groene paren worden gemiddeld; region-monsters tonen „N px avg“.

De status van de spotmeter geldt alleen voor de huidige sessie.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Voorspellende automatische belichting (DLS-gestuurd)

Dit gedeelte verschijnt alleen wanneer **ten minste één DAQ-lichtsensor is aangesloten** — de solver heeft een live neerwaarts gericht spectrum nodig om het te sturen.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Instelling | Bereik | Standaard | Wat het doet |
| --- | --- | --- | --- |
| **Inschakelen** | Selectievakje | Aan (standalone camera’s) | Een solver met gesloten formule gebruikt het DLS-spectrum plus de scalairen uit het kalibratiepakket van de camera om de helderste band dicht bij verzadiging te brengen, terwijl de donkerste band boven de SNR-drempel blijft — één belichtingsschrijfbewerking per oplossing, geen stabilisatielus. Ontworpen voor tijdopnames op zonne-energie waarbij elke opname correct belicht moet zijn. De backend schakelt stilzwijgend over naar reactieve AE wanneer de DLS-meting verouderd of ontbrekend is of het kalibratiepakket niet is geladen. |
| **Afvlakking (α)** | 0,05–1,0, stap 0,05 | 0,3 | Afvlakking van opeenvolgende voorspellende oplossingen (lager = vloeiender). |
| **Scène-reflectantie**|**ρ**-knop | — | Herberekent de scène-reflectiefactor die de solver gebruikt. |

{% hint style="info" %}
**Array connect schakelt voorspellende AE standaard uit** — voor arrays zorgen Chloros&#x27; &#x27;Smart AE&#x27; plus automatische belichting aan de camerakant zorgen voor de belichting (met verzadigingsbeveiliging) en de enkele schatting van de scène-reflectie door voorspellende AE is niet veilig bij gemengde scènes. U kunt deze functie hier per camera opnieuw inschakelen als u specifiek DLS-gestuurde radiometrische belichting wilt.
{% endhint %}

**Door DAQ gestuurde belichtingslimiet en aan invalshoek gekoppelde AE.**Ongeacht het bovenstaande selectievakje: wanneer een DAQ-lichtsensor is toegewezen aan een RGB-camera, berekent Chloros — op basis van de gemeten absolute neerwaartse stralingsintensiteit — de maximale belichting × versterking waarbij een oppervlak met 100 % reflectie onder clipping blijft, en past dit toe als een**maximum**voor de automatische belichting. Zolang de bovengrens actief is, staat de camera in**incident-pinned**-modus: hij werkt in open-loop op de gemeten invallende belichting met een versterking van 0 dB — de belichting volgt het gemeten licht, niet de inhoud van de scène. Omdat de bovengrens de belichting alleen kan verkorten, kan deze zelf geen clipping veroorzaken. De bovengrens wordt automatisch uitgeschakeld — en de normale automatische belichting van de scène wordt hervat — wanneer de DAQ-waarde ontbreekt, verouderd is (&gt;30 s), of donker is, of als ≥15 % van het beeld clipt bij de vastgezette belichting (wat betekent dat de sensor en de camera verschillende verlichting waarnemen). Er is geen schakelaar in de GUI; dit is standaardgedrag wanneer een RGB-camera een DAQ-koppeling heeft.

### Leden van de Acquisition &amp; Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array tonen bovendien de alleen-lezen **Rol**(Master in blauw / Slave in groen),**Synchronisatielijn**, en**Peers**-rijen.

| Besturing | Keuzes | Standaard | Opmerkingen |
| --- | --- | --- | --- |
| **Triggermodus** | Uit / Aan | Aan | Uitgeschakeld voor array-leden (de array regelt het triggeren). |
| **Triggerbron** | Software / Line0 (M8) / Line1 / Line2 | Line0 | Verborgen wanneer Triggermodus op Uit staat; uitgeschakeld voor array-leden. Lijn 0 is de opto-geïsoleerde externe trigger-ingang van de M8. |
| **Beeldsnelheid**| Auto / Handmatig + waarde | Auto |**Auto**: de limiet voor de beeldsnelheid van de camera is uitgeschakeld — de belichting bepaalt het aantal fps, en het venster toont de actuele live-snelheid.**Handmatig**: u beperkt het aantal fps met een schuifregelaar (van 1 tot het door de bandbreedte beperkte maximum), uitgaande van de huidige werkelijke snelheid. Array-leden zien een alleen-lezen „N fps (live)“ met „Ingesteld in array-instellingen“. |

### Netwerk / Transport

| Rij | Gedrag |
| --- | --- |
| **Pakketgrootte**| 1500 (Standaard) / 9000 (Jumbo) — standaard**Jumbo**. |
| **Doorvoersnelheid** | Alleen-lezen limiet voor de doorvoersnelheid van de verbinding in MB/s. De backend verdeelt dit bij elke verbinding of verbreking opnieuw over alle aangesloten camera’s. |
| **Bufferbeheer** | Alleen-lezen modus voor bufferbeheer. |

### Opname

Het paneel eindigt met een knop **&quot;Opname-instellingen openen…&quot;** die naar het [paneel Opname-instellingen](capture.md#the-capture-settings-pane) gaat (uitgeschakeld totdat er een project is geopend — &quot;Maak of open een project om opnames op te slaan&quot;). Als de camera verborgen of gepauzeerd is, herinnert een tip je eraan om deze weer zichtbaar te maken of te hervatten voordat je een opname maakt.

## Paneel Array-instellingen

Open dit met het **tandwiel**in een ARRAY-rij. Koptekst: array-naam met een potloodje om de naam te wijzigen en**×** om te sluiten. Secties hieronder gemarkeerd met *alleen gecombineerd* verschijnen alleen voor arrays die zijn aangesloten in de gecombineerde weergavemodus.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synchronisatie

Alleen-lezen rijen **Master**,**Slaves**en**Synchronisatielijn**.

### Omgevingslichtsensor

Wordt weergegeven voor zowel gecombineerde als afzonderlijke arrays:

* Selectievakje **Kalibratiedoel** — &quot;MAPIR ArUco-doel detecteren en NDVI valideren ten opzichte van de LUT voor paneelreflectie&quot;; bepaalt de doeloverlay en validatietabel van de gecombineerde tegel.
* ****Lichtsensor**-vervolgkeuzelijst — koppelt één DAQ aan de gehele array. De selectie wordt onmiddellijk doorgevoerd, wordt doorgegeven aan de eigen**Lichtsensor**-vervolgkeuzelijst van elke camera in de array (je kunt dit per camera nog steeds overschrijven) en begint spectra door te sturen naar de array.
* Live **Status**-regel: Uit · &quot;Wachten op eerste spectrum…&quot; · &quot;Actief — alle camera’s in de array zijn verlichtingsgecorrigeerd&quot; · &quot;Geen nieuw spectrum in de afgelopen 3 s — nog steeds de laatste meting in gebruik (geen time-out voor verouderde gegevens)…&quot;.
* Opmerking in het venster: &quot;Radiometrische correctie voor de hele array. Instellingen per camera hebben voorrang hierop.&quot;

### Opname — uniforme sensorinstellingen *(alleen gecombineerd)*

Deze instellingen gelden uniform voor elk lid (wijzigingen per lid zouden de synchronisatie verstoren). Wijzigingen worden in de wachtrij geplaatst en gezamenlijk toegepast.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Instelling | Keuzes / bereik | Wat het doet |
| --- | --- | --- |
| **Pixelformaat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Uniform sensorformaat voor alle leden. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardware-binning — behoudt het volledige gezichtsveld terwijl de SNR en de framesnelheid worden verhoogd. Als je dit wijzigt, worden de ROI-velden gereset naar het nieuwe volledige gezichtsveld. |
| **Resolutie**-voorinstelling | Volledig / Half / Kwart | Relatief ten opzichte van binning; vult de ROI-velden met een gecentreerde uitsnede. |
| **ROI-uitsnede (px)**| W / H / X / Y numerieke velden | Sensoruitsnede. Breedte/hoogte worden afgestemd op veelvouden van 16 (minimaal 64); offsets worden afgestemd op veelvouden van 4. Een “max WxH”-hint geeft de bovengrens aan en**Reset** keert terug naar het volledige gezichtsveld. Tijdens het bewerken wordt een oranje live-voorbeeldkader van de uitsnede op de array-tegel getekend (inclusief een schematische weergave van de volledige sensor wanneer de uitsnede naar buiten wordt uitgebreid). |
| **Triggerfrequentie**| Schakelaar Auto / Handmatig + fps 0,5–10, stapgrootte 0,5 |**Auto**(standaard): de backend leidt de triggerfrequentie af uit de resolutie en bandbreedte — de invoer is uitgeschakeld en toont de afgeleide waarde.**Handmatig**: legt uw waarde vast bij Toepassen. |

Opmerking in het venster: „Wijzigingen in formaat/resolutie zorgen ervoor dat alle camera’s kortstondig opnieuw opstarten. De triggerfrequentie wordt direct toegepast.“ De knoppen **Toepassen / Annuleren** bevinden zich onderaan het venster.

### Uitlijning (co-registratie) *(alleen gecombineerd)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Resterende** pill: „RMS x,xx px“ — groen onder 1 px, oranje onder 3 px, rood in alle andere gevallen of als een camera heeft gefaald; „geen profiel“ vóór de eerste berekening.
* Samenvattingsregel: &quot;✓ Alle camera’s uitgelijnd (N)&quot; / &quot;⚠ p/N camera’s uitgelijnd —  <serial (filter)="">mislukt&quot; / &quot;Bijsnijden actief — Opnieuw kalibreren om uit te lijnen (gebruikt volledige sensor)&quot; / &quot;Wachten tot de belichting stabiel is…&quot;.
* Tabel per camera: camera (laatste 4 cijfers van serienummer + filter), herprojectiefout in px met aantal overeenkomsten („ref” voor de master), en de genormaliseerde kruiscorrelatiescore van de overlapping ten opzichte van de drempelwaarde van 0,35.
* **Knop**Uitlijning opnieuw kalibreren** (tekst „Uitlijning kalibreren“ vóór het eerste profiel) — voert de co-registratie opnieuw uit op nieuwe frames.
* **Selectievakje &quot;Camera&#x27;s automatisch belichten voor uitlijning&quot;** (standaard aangevinkt) — maakt donkere of vlakke camera&#x27;s tijdelijk helderder (eerst belichting, dan versterking) zodat ze textuur hebben om te matchen, en herstelt vervolgens de automatische belichting.

Het gecombineerde voorbeeld wordt automatisch uitgelijnd bij het openen; kalibreer opnieuw als de scherpstelling of de diepte van de scène is veranderd. Uitlijning is **per definitie alleen voor de huidige sessie** — deze wordt nooit opgeslagen in een profiel, omdat deze afhankelijk is van de afstand tot de scène op dat moment. Opnames kunnen nog steeds worden geëxporteerd met pixelregistratie (zie [Uitgelijnde exporten](capture.md#per-array-controls)).

### Slimme vignettering

* Selectievakje **Correctie inschakelen**— past de per camera geschatte vignettering toe op de radiometrische keten (live**en** bij export).
* **Kalibreren vanuit huidige weergave**— richt de array eerst op een uniform doel (vlak scherm, muur of lucht); elke camera wordt afzonderlijk geflatteerd en de status geeft „n/N camera’s · −x,x %“ vlakheidswinst weer. Met**Wissen** wordt de schatting verwijderd.
* Stel per camera nauwkeurig af met de **Vignettering**-schuifregelaar per camera in [Live preview](#live-preview).

### Live Preview *(alleen gecombineerd)** **Index**: vink het selectievakje aan + tandwiel — opent de gedeelde [Indexcalculator](#index-calculator-pane) met banden die zijn getekend op basis van**alle** aangesloten camera’s. Een voorbeeldregel voor de uitdrukking eronder toont de huidige uitdrukking („Geen uitdrukking ingesteld — open de calculator om er een te maken“), die elke seconde wordt vernieuwd.
* **Renderresolutie**-dropdown (dezelfde voorinstellingen als per camera, standaard 720p): de hoogte van de liveweergavestream**en** de opgeslagen exportgrootte van de compositie. Let op in het paneel: „Voorbeeld + opgeslagen composietgrootte. Beelden per camera worden altijd in volledige resolutie geëxporteerd.”

### Weergavelaag *(alleen gecombineerd)** Selectievakje **Inschakelen** (standaard uitgeschakeld — de hoofdcamera wordt rechtstreeks weergegeven; ingeschakeld = gelaagde compositie).
* **Voorgrond**/**Achtergrond**-keuzelijsten: elke lidcamera (op naam) of**Index**. Wanneer Voorgrond op Index staat, tonen pixels buiten de LUT-min/max de Achtergrondlaag.

### Gesplitste weergave *(alleen gecombineerd)*

**&quot;Lidcamera’s weergeven&quot;**— een knop**Lidcamera’s splitsen / verbergen** waarmee de eigen livebeelden van elk lid als afzonderlijke rastertegels naast de compositie worden toegevoegd. De tegels lezen de bestaande framebuffer van de array (geen extra camera-aansluiting). Alleen in rasterweergave; per array samen met het project opgeslagen.

### Mogelijkheden

Een alleen-lezen paneel dat elke 5 s wordt vernieuwd:

* **Niveau-label**: &quot;Gelijktijdige opname&quot; (groen) · &quot;Gelijktijdige opname (FTD-versprongen uitsturing)&quot; (groen) · &quot;Versprongen opname (100 ms drift)&quot; (oranje) · &quot;Configuratie te groot&quot; (rood).
* **Frame-status**: &quot;x,xx % onvolledig&quot; — groen bij minder dan 1 %, oranje bij minder dan 5 %, rood bij 5 % of meer.
* **Verbindingslijn**: &quot;NIC {mbps} Mbps - continu {MB/s} MB/s&quot;.

Dit is het actuele bandbreedtebudget van de array. Zie [Multi-Camera Arrays](arrays.md) en de [CLI-referentie](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Paneel ‘Indexcalculator’

De derde pagina in de zijbalk, die wordt gedeeld door de Index-instelling per camera en de Index-instelling voor de gecombineerde array (één tegelijk — de kop luidt „Indexcalculator — <camera name="">“ of „Indexcalculator —<array name="">

“). Hier worden de bandenlijst (de natuurlijke filterbanden van de camera, of alle banden van de leden van de array), de huidige uitdrukking en de LUT-configuratie (aan/uit, niveau — standaard 3, min — standaard 0,2, max — standaard 1), plus een live indexhistogram. Met **Toepassen** wordt de uitdrukking vastgelegd; LUT-wijzigingen worden live toegepast op het voorbeeld.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Instellingen per camera versus door de array beheerde instellingen

Beknopt overzicht van wat zich waar bevindt wanneer een camera deel uitmaakt van een array:

| Door de array beheerd (alleen-lezen in het cameravenster) | Nog steeds per camera binnen een array |
| --- | --- |
| Pixelformaat, resolutie, binning | Automatische belichting (belichting, versterking, doelwaarde, afvlakking, ROI) |
| Triggermodus/bron, framesnelheid | Ruisonderdrukking, scherpte, vignettering |
| | Oriëntatie (spiegelen/rotatie), weergave-overlays, spotmeter |
| | Index (arrays met afzonderlijke weergave), koppeling lichtsensor |

Overkoepelend gedrag:

* **Gecombineerde versus afzonderlijke weergave** wordt gekozen bij het koppelen van de array: gecombineerd = één uitgelijnde samengestelde tegel (leden sturen alleen beelden via Split View); afzonderlijk = elk lid geeft zijn eigen gesynchroniseerde tegel weer. Een camera toont nooit zowel een afzonderlijke feed als een array-tegel.
* **Automatisch opnieuw verbinden**: bij het openen van een opgeslagen project worden de camera’s en arrays hersteld en worden alle opgeslagen instellingen opnieuw toegepast op de backend voordat de streams worden hervat.
* **Opnamegating**: verborgen of gepauzeerde camera’s worden uitgesloten van ‘Alles opnemen’; een array wordt pas volledig geblokkeerd wanneer ALLE leden verborgen of gepauzeerd zijn. Zie [Opname-instellingen &amp; -modi](capture.md).

## Hoe instellingen worden bewaard

De status van het cameratabblad wordt **samen met het project** opgeslagen, niet in de browser:

* Bij elke reactieve wijziging wordt een momentopname van de camera&#x27;s en arrays opgeslagen in het `cameras.json`-bestand van het project (met een debounce van 500 ms). Dit omvat cameranamen en -kleuren, instellingen voor belichting/versterking/AE, pixelformaat/resolutie/binning, triggerfrequentie, voorbelichtingsinstellingen (renderresolutie, ruisonderdrukking, scherpte, vignettering, kleurprofiel, verzadiging/contrast), oriëntatie, overlays, kanaalsplitsingen, indexconfiguratie, instellingen voor voorspellende AE, AE-ROI, array-namen, weergavemodus, opname-instellingen voor arrays (inclusief ROI-uitsnijdpositie) en het rasterblok (feed-zoom, weergavemodus, rastervergrendeling, handmatige tegelvolgorde, verborgen camera’s, gesloten tegels, actieve camera).
* Koppelingen van lichtsensoren worden opgeslagen in het `sensors.json`-bestand van het project.
* Bij het opnieuw openen van het project wordt de hardware opnieuw gekoppeld en worden al deze instellingen opnieuw toegepast.
* **Geen project geopend = alleen sessie**: zonder project blijft er niets behouden bij het sluiten van Chloros.
* Alleen sessie, ongeacht het project: pauzetoestand, spotmeter-metingen, het selectievakje ‘Kalibratiedoel’ per camera (wordt standaard altijd uitgeschakeld) en het uitlijningsprofiel van de array (wordt per sessie opnieuw berekend, zoals bedoeld).
* Eén uitzondering: de exportselecties voor **Opname-instellingen** en de opnamemodus blijven per project bewaard in de lokale app-opslag in plaats van in `cameras.json` — zie [Opname-instellingen en -modi](capture.md).</array></camera></serial>
