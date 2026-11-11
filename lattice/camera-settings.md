# Camera-instellingen

Het tabblad **Camera&#x27;s** is het live-bedieningspaneel vChloros

voor LATTICE-camera&#x27;s: een hoofdweergavegebied waarin elke aangesloten camera als een live-tegel wordt weergegeven, en een zijbalk waarmee je tussen drie pagina&#x27;s kunt schakelen — de **cameralijst**, een**instellingenvenster**(instellingen per camera, array of opname — één tegelijk) en de**Indexcalculator**. Op deze pagina worden alle bedieningselementen in de cameralijst, het instellingenvenster per camera en het instellingenvenster voor de array beschreven. Opnamemodi, selectie van het exporttype en de „Capture All“-workflow staan op de bijbehorende pagina [Opname-instellingen &amp; -modi](capture.md).

Het tabblad „Camera’s“ verschijnt in de zijbalk zodra de Chloros

-backend gereed is. Alle onderstaande bedieningselementen communiceren met de lokale backend via `127.0.0.1:5000`; wijzigingen worden onmiddellijk toegepast op de livecamera, tenzij anders vermeld.

## Cameratypen die op deze pagina worden gebruikt

Bedieningselementen worden al dan niet weergegeven, afhankelijk van het geselecteerde cameratype. In de handleiding worden doorlopend de volgende termen gebruikt:

| Term | Betekenis | Filterkanalen |
| --- | --- | --- |
| **RGB

-camera** | LATTICE M3C met het FRGB-filter (model bevat `-FRGB`) | Red

/ Green

/ Blue

|
| **Bayer multispectraal** | LATTICE M3C met FRGN, FOCN of FNGB | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — één smalbandfilter, één gekalibreerde band | Enkele band |
| **Array-lid** | Een camera die is aangesloten als onderdeel van een gesynchroniseerde array (gecombineerde of afzonderlijke weergave) | Per filter |

Bij RGB

-camera’s vindt fotometrische verwerking plaats (witbalans, kleurprofielen, gamma); bij multispectrale en mono-camera’s wordt de radiometrische keten toegepast en worden de fotometrische instellingen overgeslagen. Array-leden passen instellingen op streamniveau (pixelformaat, resolutie, binning, trigger, framesnelheid) door aan de array — die rijen worden alleen-lezen in het paneel per camera en worden verplaatst naar het paneel met array-instellingen.

## Het hoofdgebied voor de feed

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Als er geen camera’s zijn aangesloten, toont het feed-gebied een **&quot;Sluit een camera aan om te beginnen&quot;**-startscherm met twee knoppen:**Camera aansluiten**(groen, opent het verbindingsvenster voor één camera) en**Array verbinden** (blauw, opent het verbindingsvenster voor een array). De verbindingsvensters zelf worden beschreven in [Camera’s aansluiten](connecting.md); array-concepten (synchronisatie, lagen, bandbreedte) in [Multi-camera-arrays](arrays.md). Wanneer je een opgeslagen project opent dat camera’s bevat, toont het welkomstscherm in plaats daarvan een draaiend pictogram met de tekst &quot;N opgeslagen camera’s opnieuw openen…&quot; terwijl Chloros

de streams van de laatste sessie herstelt.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Bovenste balk

| Bedieningselement | Wat het doet |
| --- | --- |
| **Weergavemodus wisselen**| Schakelt tussen**rasterweergave**(alle tegels als cellen) en**lijstweergave**(rijen over de volledige breedte: elke gecombineerde array als één samengestelde rij, elk onderdeel van een**afzonderlijke** array als een eigen rij met een eigen feed en kalibratiedoel-tabel, en daaronder ÉÉN op zichzelf staande camera — de actieve). Tooltips: „Overschakelen naar rasterweergave“ / „Overschakelen naar lijstweergave“. |
| **Rastervergrendeling**(hangslot) | Standaard**vergrendeld** — tegels staan vast op hun plaats. Ontgrendel om tegels naar een willekeurige positie te slepen (tussenruimtes blijven behouden). Het raster wordt automatisch opnieuw vergrendeld telkens wanneer een nieuwe camera verbinding maakt. Tooltips: „Raster ontgrendelen (tegels verslepen inschakelen)“ / „Raster vergrendelen (tegels op hun plaats vastzetten)“. |
| **Feed-zoom**-schuifregelaar | Tegelgrootte, van 60 px tot de volledige breedte van de container. Cellen behouden een beeldverhouding van 4:3. Bij een celbreedte van minder dan 200 px worden de overlays met de naam en fps verborgen om de tegel overzichtelijk te houden. |

### Feed-tegels

Elke camera geeft een samengestelde live-tegel weer; een camera kan bovendien drie grijswaarden-tegels met **splitsing per kanaal** weergeven (zie [Kanaalsplitsingen](#display-overlays-drawn-over-the-live-feed)), en arrays geven een gecombineerde tegel weer. De actieve tegel is voorzien van een selectiering in de kleur van de camera (of array).

Als je met de muis over een tegel beweegt, verschijnt er een **X**-sluitknop:

* Als je een **samengestelde** tegel sluit terwijl de kanaalsplitsingen nog zichtbaar zijn, wordt alleen de samengestelde tegel verborgen.
* Als je de **laatste zichtbare tegel van een op zichzelf staande camera** sluit, wordt de verbinding met die camera verbroken.
* **Splitsingstegels van een gecombineerde array verbreken nooit** de verbinding met de camera — ze worden alleen verborgen.

Als het raster ontgrendeld is, kun je elke tegel naar een willekeurig vak slepen; de indeling wordt samen met het project opgeslagen.

## Zijbalk — cameralijst

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Op de eerste pagina van de zijbalk staan alle aangesloten camera’s en arrays vermeld:

* **Camera verbinden**(groen) /**Array verbinden** (blauw, toont „Detecteren...” tijdens het scannen). Beide zijn uitgeschakeld zolang een verbindingsdialoogvenster open is.
* **Alles vastleggen** (rood) — legt elke vermelde camera vast met de exporttypes die zijn gekozen in de vastleginstellingen. Vereist een geopend project. Volledig gedocumenteerd in [Opname-instellingen &amp; modi](capture.md).
* **Tandwiel voor opname-instellingen** (naast Alles vastleggen) — opent het [venster Opname-instellingen](capture.md#the-capture-settings-pane). Uitgeschakeld zonder project of tijdens het vastleggen.

### Camerarijen

Elke camerarij toont een kleurgecodeerde rand (de aangepaste kleur van de camera), een &quot;CAM&quot;-label — met een blauwe **M**(master) of groene**S** (slave) als rolletter voor array-leden — en de weergavenaam. De standaardnaam is `LATTICE-MODEL (serial)`; hernoem deze via het instellingenvenster per camera. Rijknoppen:

| Knop | Effect |
| --- | --- |
| **Oog**| Zichtbaarheid in- of uitschakelen. Verborgen camera’s verdwijnen uit het raster en worden**uitgesloten van ‘Capture All’**. |
| **Tandwiel** | Open het instellingenvenster per camera (volgende sectie). |
| **Pauzeren / Afspelen**| Bevries het livevoorbeeld**alleen aan de weergavezijde** — de opname door de backend blijft doorgaan. Gepauzeerde camera’s kunnen niet opnemen. |
| **X** | Verbinding verbreken. De gebruikersinterface wordt onmiddellijk bijgewerkt (in het beste geval); het verbreken van de verbinding door de backend zelf kan 10–30 s duren. |

### Rijen in de array

Een rij in de array toont een „ARRAY“-badge in de kleur van de array, de naam van de array (aanpasbaar in de array-instellingen) en een **DAQ · aan/uit**-knop —**aan** wanneer de lichtsensor op array-niveau is ingesteld *of* een van de leden een sensor per camera heeft; de tooltip geeft precies aan welke sensor wat doorgeeft. De camera’s van de leden worden daaronder met inspringing weergegeven in hun eigen rijen. Knoppen in array-rijen: **oog**(verbergt/toont ALLE leden tegelijk),**tandwiel**(venster met array-instellingen),**X**(ontkoppel de hele array).

De status van de lichtsensor (DLS) die in array-rijen en het venster met array-instellingen wordt gebruikt, kent vier toestanden:**uit**,**in afwachting**(nog geen spectrum),**actief**(er is binnen de laatste 3 s een spectrum binnengekomen) en**verouderd** — geen nieuw spectrum in 3 s, maar de laatste meting wordt *nog steeds gebruikt* (DAQ-metingen vervallen nooit op het vastleggingspad).

Totdat je de volgorde wijzigt, toont de lijst eerst arrays en daarna losstaande camera’s, waarbij camera’s (en de leden van elke array) in **cameravolgorde** staan: M3C (Bayer) vóór M3M (mono); onder M3C: FRGB, FRGN, FOCN en vervolgens FNGB; M3M op filtergolflengte, van laag naar hoog. De standaardindeling van het raster volgt dezelfde volgorde. Je kunt losstaande camera’s en hele arraygroepen in de zijbalk langs elkaar slepen om de lijst te herschikken; arrayleden kunnen niet afzonderlijk worden versleept.

## Instellingenvenster per camera

Open dit venster met het **tandwieltje** op een camerarij. Het venster schuift over de cameralijst heen.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Koptekst**: het**kleurstaal**van de camera (klik om een native kleurenkiezer te openen — hiermee stel je de kleur van de zijbalkrand en de selectiering van de tegels in), de**naam**met een potloodknop**Hernoemen**(als je een lege naam opslaat, wordt de standaardnaam `MODEL (serial)` hersteld) en**×** om te sluiten.

### Live histogram

Bovenaan het paneel staat een live luma-histogram dat wordt berekend op basis van het JPEG

-voorbeeld met een frequentie van ~8 Hz. Het gemiddelde is Bayer-gewogen — (R+2G+B)/4 — om overeen te komen met de eigen AE-meting van de camera.

* **OOrange

n gestreepte lijn**= het AE-doel.**Sleep deze horizontaal om het doel te verplaatsen** — bij het loslaten wordt één commando verzonden, en door te slepen wordt de AE-doelmodus omgeschakeld naar Handmatig.
* **Green

doorlopende lijn** = de werkelijke gemiddelde luma (wat de AE momenteel levert).
* **RGB

-knop** (rechtsboven): schakelt tussen overlay-histogrammen per band, gekleurd volgens het filter van de camera (bijv. bij FRGN: grijs NIR

, groen, rood). Bij monochrome (M3M) camera’s staat op de knop „MONO“ en is deze uitgeschakeld — bij monochrome camera’s wordt altijd het luma-histogram van één band weergegeven.
* De labels op de X-as volgen de bitdiepte van de sensor van het huidige pixelformaat: 0..255, 0..1023, 0..4095 of 0..65535.

### Rijen

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

met camera-informatie

| Rij | Gedrag |
| --- | --- |
| **Model** | Alleen-lezen (bijv. `LATT-M3C-L87-FRGN`). |
| **Radiometrische kalibratie** | Green**&quot;Actief&quot;**-badge met een bijschrift dat het kalibratieniveau, de hash, de kalibratiedatum en de bandenlijst weergeeft, geladen vanuit het kalibratiepakket van de camera (zie [Fabrieksradiometrische kalibratie](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Verborgen voor RGB

-camera’s** — deze beschikken over een fotometrische witbalanskaling, niet over stralingsintensiteit per band. |
| **Kalibratierapport**|**Download**-knop — opent het NIST-kalibratiecertificaat per serienummer van de camera als PDF in de viewer van je besturingssysteem. Als het certificaat nog niet in de cache is opgeslagen, toont Chloros

in plaats daarvan een hint. |
| **Serienummer** | Alleen-lezen. |
| **Firmware**| Toont de huidige versie en zoekt vervolgens de beschikbare versie voor dit model op (in de cache per model — een reeks van N camera’s controleert de server één keer). Status: „Bezig met controleren…“ →**„Bijwerken naar X“**-knop → „Bezig met flashen…“ → „Bijgewerkt van A naar B“ / „Mislukt: …“ / „Overgeslagen: …“ / groen**„Up-to-date“**. Tooltip van de updateknop: „Fabrieksinstellingen herstellen + flashen + UserSet1 herprogrammeren. ~2–3 minuten; niet loskoppelen.“ |
| **IP** | Alleen-lezen. |
| **Temperatuur** | Alleen-lezen, wordt elke 3 s vernieuwd. Wordt oranje bij ≥65 °C en rood met een ⚠ bij ≥75 °C. |
| **Kalibratiedoel** selectievakje | Schakelt ArUco-reflectiedetectie in met een NDVI

-validatietabel per paneel onder de livefeed (lijstweergave) — zie [De kalibratiedoel-tabel](#the-calibration-target-table). Alleen tijdens de sessie — wordt altijd uitgeschakeld geopend. |
| **Lichtsensor**-vervolgkeuzelijst | Koppelt een DAQ-lichtsensor (DAQ-E/M/U, uit de lijst op het tabblad ‘Lichtsensoren’) aan deze camera voor verlichtingscorrectie van neerwaarts licht (DLS) en voorspellende automatische belichting. &quot;&quot;Geen&quot; verwijdert de koppeling. Als er geen sensoren zijn aangesloten, toont de keuzelijst &quot;(geen sensoren aangesloten — open het tabblad DAQ)&quot;. De koppeling wordt samen met het project opgeslagen. |

#### De tabel ‘Kalibratiedoel’

De tabel is **uitsluitend bedoeld voor validatie** („alleen validatie — wijzigt de kalibratie niet”). De koptekst vermeldt het serienummer van de camera, het filter, het doelnummer en de markergrootte in pixels, en de tabel bevat één rij per paneel:

* **Paneel**, gevolgd door één kolom per band met de waarde van het paneel — een Bayer-camera toont haar drie banden op naam en golflengte (bijv. FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); een **mono (M3M)-camera toont zijn enkele band** (bijv. `F750`) en heeft geen NDVI

-kolommen.
* **Sat** — het percentage pixels van het paneel dat verzadigd is (amber vanaf 1 %, rood vanaf 5 %).
* **NDVI

Meas**,**NDVI

Theo** (de referentie-reflectantie-NDVI

van het paneel) en **Δ**— alleen voor multispectrale Bayer-camera’s. Een klein teken achter „NDVI

Meas“ geeft aan waarop de gemeten NDVI

is gebaseerd:**ρ**(reflectantie — er is een lichtsensorwaarde beschikbaar),**L**(stralingswaarde — geen lichtsensorwaarde) of**DN**(ruwe camerawaarden). Alleen een**ρ**-NDVI

is vergelijkbaar met NDVI

Theo: voor L en DN is de Δ-kolom grijs weergegeven en wordt dit in een opmerking aangegeven (&quot;Geen lichtsensorwaarde: NDVI

is afgeleid van stralingsintensiteit, dus Δ is geen kalibratiecontrole&quot; / &quot;NDVI

is afgeleid van ruwe camerawaarden, dus Δ is geen kalibratiecontrole&quot;). Bij ρ is Δ groen onder 0,02, oranje onder 0,05 en rood in alle andere gevallen.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Belichting &amp; Versterking

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alle numerieke invoervelden hier maken gebruik van draaiknoppen die versnellen bij lang vasthouden: tikken = ±1, langer dan 1,5 s vasthouden = ±10, langer dan 3 s vasthouden = ±100. De waarde wordt naar de camera verzonden zodra je loslaat.

| Instelling | Bereik / keuzes | Standaard | Van toepassing op | Wat het doet |
| --- | --- | --- | --- | --- |
| **Belichting (us)**| De minimale/maximale livewaarden van de camera | Auto | Alles | Belichtingstijd in microseconden, met een**Auto/Handmatig**schakelaar. Auto = continue automatische belichting door de camera. Als je een handmatige belichting instelt, blijft**Versterking** op Auto staan (als dat al zo was), zodat je het volledige versterkingsbereik van de camera kunt gebruiken, totdat je de versterking aanpast of op Handmatig zet. |
| **Versterking (dB)**| De live min./max. van de camera (bijv. tot 48 dB) | Handmatig (uit) | Alle | Analoge/digitale versterking met een eigen**Auto/Handmatig**-schakelaar. |
| **AE-doelhelderheid**| 0–255 | 80, modus**Auto**| Alle (aanpasbaar wanneer AE of automatische versterking is ingeschakeld) | De helderheid waarop de AE zich richt. In**Auto**(de standaardinstelling) kiest een op het histogram gebaseerde backend-controller zelf de doelwaarde, waarbij de belichting op 60–75 % van het maximum van de sensor wordt gehouden. Door een waarde in te voeren of de oranje lijn van het histogram te verslepen, schakelt u over naar**Handmatig**. |
| **AE-afvlakking** | 0,5–40, stap 0,1 | 8,0 | Alles | AE-demping. Tooltip: &quot;Lager = AE reageert sneller (kan bij hoge fps pulseren). Hoger = vloeiender / langzamer.” Waarden die ver onder de standaardinstelling liggen, kunnen ervoor zorgen dat de AE pulseert en de streaming bij hoge framesnelheden destabiliseert; 8,0 is de stabiele standaardinstelling. |
| **AE-regio van belang**| Selectievakje inschakelen +**Richten**-knop | Uit | Alle | Indien ingeschakeld, meet AE alleen het groen gestreepte gebied in plaats van het hele beeld.**Richten** activeert ‘klikken om te plaatsen’ op de livefeed: een klik centreert een gebied op 30 % van het beeld; klikken en slepen tekent een aangepaste rechthoek (minimaal 5 % × 5 %). *Aim* wordt automatisch uitgeschakeld na één plaatsing. Het gebied wordt teruggezet naar de oorspronkelijke cameracoördinaten volgens de door jou ingestelde rotatie/spiegeling, en wordt samen met het project opgeslagen. |
| **AE-afstemsnelheid** | 0,1–5, stap 0,1 | 1,0 | Alleen voor array-leden | Hoe snel het automatische AE-doel veranderingen in de helderheid van de scène volgt; bij 1,0× wordt elke 2,5 s opnieuw gecontroleerd. |
| **Highlightbescherming** | Streng (1 %) / Normaal (5 %) / Soepel (15 %) | Streng | Camera’s die de instelling weergeven | Hoeveel van het beeld mag naar wit clippen voordat de AE het beeld donkerder maakt. |

{% hint style="info" %}
**Lichtvereiste voor Bayer-multispectrale camera’s (RGN

/ OCN

/ NGB

):** de scène moet in alle drie de kanalen voldoende licht hebben, anders werkt de kalibratie niet correct — één enkele sensorbelichting bestrijkt alle drie de spectra. Gebruik een DAQ-lichtsensor om je licht te meten, of schakel over naar volledig mono (M3M) zodat elke band zijn eigen belichting krijgt. Als een opname hieraan niet voldoet, detecteert Chloros

dit en waarschuwt het je (de ‘unmix-clamp’-melding). Live geeft een kanaalsplitsingstegel **&quot;Bandgegevens met lage betrouwbaarheid · N%&quot;** weer wanneer de unmix die band op 25 % of meer van de pixels heeft geklemd.
{% endhint %}

{% hint style="info" %}
**Hoe automatische belichting belichtingstijd en versterking verdeelt (1.3.0).** Als de versterking op &#x27;Auto&#x27; staat, houdt Chloros

de versterking op 0 dB totdat de belichting de langst toegestane waarde heeft bereikt (de bovenste AE-limiet, de triggerperiode van een array of het DAQ-belichtingsplafond); pas dan stijgt de versterking. In een array heeft de triggertijd voorrang: automatische belichting vult de periode die de triggertijd aangeeft, en verhoogt vervolgens de versterking, zodat een donkerder wordende scène de automatische triggerfrequentie niet langer verlaagt (in 1.2.0 was dat wel het geval). Een **handmatige**belichting op een array-lid die langer duurt dan de triggerperiode, vertraagt de trigger van de array om hieraan aan te passen; de automatische frequentie keert terug zodra elk element dat deze vertraagde weer op automatische belichting staat. Wanneer zowel**Belichting**als**AE-doelhelderheid** op ‘Auto’ staan en een DAQ-lichtsensor een meting levert, wordt de belichting van elke camera vastgehouden zodra de automatische belichting zich heeft gestabiliseerd, zolang het gemeten licht binnen ongeveer ±25 % blijft; een grotere, aanhoudende lichtverandering, een herstart van de instellingen (bijvoorbeeld een ROI-wijziging) of een door u aangebrachte wijziging in de belichting activeert deze opnieuw.
{% endhint %}

{% hint style="info" %}
**Hoe automatische belichting werkt (1.3.0).**

* Bij handmatige belichting blijft de versterking op ‘Auto’ staan totdat je de versterking wijzigt.
* Kleine hoge lichten maken de scène niet donkerder: een spiegeling tot 1 % van het beeld wordt genegeerd, en in de panelen van een bekend kalibratiedoel tot 0,5 % van een paneel.
* De DAQ-belichtingslimiet geldt alleen zolang een kalibratiedoel in beeld is; als er geen paneel te zien is, belicht de camera de scène.
* De belichting blijft stabiel bij constant licht en stabiliseert zich binnen enkele seconden na een grote verandering. Een clip beperkt de belichting niet langer definitief, en bij een verandering van het ROI wordt de belichting opnieuw bepaald.
* Het helderheidsdoel overschrijdt het bereik van de camera nietbereik, zodat terugkerend licht geen overbelichting in het beeld veroorzaakt.
* Een camera waarvan de automatische belichting niet meer reageert in fel zonlicht, wordt op het niveau gehouden dat past bij het licht waarin hij zich bevindt, gemeten door de lichtsensor of, indien er geen is aangesloten, door de eigen beelden van de camera, en losgelaten wanneer het licht verandert. Een donkere of afgedekte lichtsensor verstoort dit niet.
{% endhint %}

### Pixelformaat en resolutie

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Array-leden** tonen de alleen-lezen rijen „Huidige“ (formaat + BxH) en „Binning“ met de opmerking „Ingesteld in array-instellingen“ — een herstart van de stream op één lid zou de synchronisatie verstoren, dus deze worden beheerd in het [paneel met array-instellingen](#array-settings-pane).**Standalone camera’s** krijgen:

| Instelling | Keuzes | Wat het doet |
| --- | --- | --- |
| **Pixelformaat** | 8-bit / 10-bit / 12-bit / 16-bit | Bitdiepte van de sensor; de formaatfamilie is die van de sensor. De standaardinstelling is 12-bit, en een project slaat op wat je instelt, inclusief 8-bit, en maakt opnieuw verbinding met die instelling. |
| **Resolutie** | Volledig / Half / Kwart | Relatief ten opzichte van de huidige binning: Volledig = 2048/N × 1536/N voor N×N-binning. |
| **Binning** | 1x1 (geen) / 2x2 / 4x4 | Hardware N×N-binning — grotere waarden verlagen de resolutie, maar verbeteren de SNR en de framesnelheid. Als je dit wijzigt, wordt de stream opnieuw gestart en worden alle ROI’s gereset naar het nieuwe volledige gezichtsveld. |
| **Huidig** | alleen-lezen | De daadwerkelijke WxH en (x, y) offset die van kracht zijn. |

### Livevoorbeeld

Alles in dit gedeelte is **alleen aan de weergavezijde**— het verandert wat je in de livefeed ziet, terwijl opgeslagen opnames lineair en ongewijzigd blijven — met één uitzondering:**Vignette** is radiometrisch en heeft ook invloed op exportbestanden (zie hieronder).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Instelling | Bereik / keuzes | Standaard | Van toepassing op | Wat het doet |
| --- | --- | --- | --- | --- |
| **Weergaveresolutie** | 360p (snelst) / 480p / 720p / 1080p / Native sensorresolutie (traagst) | 720p | Alles | De hoogte waarop de backend de radiometrische previewketen uitvoert. Een lagere instelling levert een hogere framesnelheid op zonder het gezichtsveld te wijzigen. |
| **Index**| Selectievakje ‘Inschakelen’ + tandwiel | Uit | Alleen Bayer-multispectraal,**niet** voor gecombineerde arrays | Live preview van de vegetatie-index. Het tandwiel opent het gedeelde [Indexcalculator](#index-calculator-pane) dat vooraf is geladen met de natuurlijke filterbanden van de camera (bijv. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). De aangepaste uitdrukking plus LUT (aan/uit, standaardniveau 7-stop, standaardmin. 0,2, standaardmax. 1) wordt bij elk voorbeeldframe berekend. Leden van de gecombineerde array verbergen deze rij — de array heeft één gedeelde index. |
| **Witbalans** | Uit / Eenmalig / Continu + een knop voor opnieuw vastleggen | Continu | Alleen RGB

| Live witbalans. De vernieuwingsknop registreert de witbalans opnieuw op basis van het huidige DLS-spectrum (uitgeschakeld wanneer de modus op Uit staat). |
| **Gamma** | Aan / Uit | Aan | Alleen RGB

| Weergave van gamma (γ = 2,2 LUT) in het live-voorbeeld. Opgeslagen opnames blijven lineair. |
| **Ruisonderdrukking** | Selectievakje + sterkte 0–100 | Uit / 50 | Alle (per camera, zelfs binnen arrays) | Bilateraal filter in het live-voorbeeld. Hoger = vloeiender maar zachtere details. |
| **Scherpte** | Selectievakje + sterkte 0–100 | Uit / 30 | Alles | Onscherp masker in het livevoorbeeld, als laatste toegepast. Kan ruis versterken. Alleen in het voorbeeld. |
| **Vignettering**| Selectievakje + sterkte 0–100 | Uit / 0 | Alles | Handmatige correctie van resterende vignettering (maakt hoeken lichter), bovenop de Smart Vignette-schatting van de reeks.**Radiometrisch — beïnvloedt zowel de liveweergave als de export**, in tegenstelling tot Ruisonderdrukking/Scherpte. |
| **Kleurprofiel** | Raw / Lineair / Natuurlijk / Verbeterd / Aangepaste temperatuur | Natuurlijk | Alleen RGB

| Zie hieronder. |
| **Kleurtemperatuur** | 2000–10000 K, stap 100 | 5500 K | Alleen RGB

, aangepast temperatuurprofiel | Koppelt de witbalans aan een vaste gecorreleerde kleurtemperatuur (DLS-invoer wordt genegeerd). De laatst gekozen Kelvin-waarde wordt onthouden bij het wisselen van profiel. |
| **Hooglichten** | −100 tot 0, stap 1 | 0 (uit) | Alleen RGB

| Nieuw in 1.3.0. Maakt de helderste, witste delen van de afbeelding donkerder (0 = ongewijzigd, −100 = sterkst). Geldt voor het live-voorbeeld **en voor opnames op schermniveau (Voorbeeld)**, die de instelling vastleggen (`Chloros:DisplayHighlights`); exporten van straling en reflectie worden nooit beïnvloed. Het kan geen sensorgegevens herstellen die zijn afgekapt — verlaag daarvoor de belichting. De zebra-overlay volgt het donkerder gemaakte beeld. Uitgeschakeld bij het Raw-kleurprofiel (&quot;Wordt niet toegepast bij het Raw-kleurprofiel.&quot;). |
| **Verzadiging** | 0–200 (100 = neutraal) | 100 | Alleen RGB

| HSV-verzadiging in de live preview. |
| **Contrast** | 0–200 (100 = neutraal) | 100 | Alleen RGB

| Lineair contrast rond middengrijs in het livevoorbeeld. |
| **Spiegelen H / Spiegelen V** | Selectievakjes | Uit | Alles | Het voorbeeld horizontaal / verticaal spiegelen. |
| **Rotatie**| 0° / 90° / 180° / 270° | 0° | Alles | Het voorbeeld roteren. De oriëntatie wordt toegepast aan het einde van de backend-voorbeeldketting.**Opgeslagen opnames (1.3.0):**de producten Preview, Radiance, Reflectance en Index van een standalone camera worden gedraaid door Spiegeling/Rotatie, terwijl**Raw en Debayered behouden de oorspronkelijke oriëntatie van de camera**; elk opnamebestand slaat de instelling op in zijn XMP, zodat deze bij latere verwerking van een raw-bestand op dezelfde manier wordt toegepast.**Bestanden in een array behouden hun oorspronkelijke oriëntatie** (de instelling wordt opgeslagen, maar niet toegepast), en samengestelde weergaven van de array negeren deze. |**Semantiek van kleurprofielen** (RGB

-camera’s):

* **Raw** — de verwerkingsketen wordt volledig omzeild.
* **Lineair** — donker-signaal + flat-field + witbalans; geen kleurenmatrix, geen gamma.
* **Natuurlijk** *(standaard)* — lineair plus de gemeten kleurcorrectiematrix en een aan de scène aangepaste tooncurve.
* **Verbeterd**— Natuurlijk plus levendigheid en CLAHE lokaal contrast. De extra kosten gelden**alleen voor het live-voorbeeld** — opgeslagen opnames krijgen altijd de volledige afwerking, ongeacht het profiel.
* **Aangepaste temperatuur** — Natuurlijk met witbalans vastgezet op de door u gekozen Kelvin-waarde.

{% hint style="warning" %}
Voor ‘Natuurlijk’, ‘Verbeterd’ en ‘Aangepaste temperatuur’ toont het venster een opmerking over de toon: beelden worden lichter gemaakt op basis van hun eigen scène, waardoor opgeslagen *weergave*-afbeeldingen niet frame-voor-frame vergelijkbaar zijn. **Exporteer straling of reflectie voor metingen.**
{% endhint %}

### Weergave-overlays (over de livefeed heen getekend)

Deze zijn alleen voor de frontend — ze worden over de video heen getekend en raken de stream of opnames nooit.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlay | Bedieningselementen | Standaard | Wat het doet |
| --- | --- | --- | --- |
| **Zebra** | Selectievakje + drempelwaarde 200–255 | Uit / 250 | Magenta diagonale strepen op afgeknipte pixels. |
| **Kruisdraad** | Selectievakje | Uit | Markering in het midden van het frame. |
| **Raster** | Uit / 3 × 3 / 9 × 9 | Uit | Compositieraster. |
| **Histogram** | Selectievakje + breedte 0,10–0,90 van het beeld | Uit / 0,25 | Een histogramstrook in de feed. |
| **Focus Peak** | Selectievakje + drempelwaarde 20–200 + kleurstaal | Uit / 80 / `#ff5722` | Sobel-randmarkering voor scherpstellen. |
| **Kanaalsplitsingen** | &quot;Splitsingen weergeven (Red

/ Green

/ NIR

)&quot; / Knop &quot;Splitsingen verbergen&quot; | Verborgen | Voegt naast het samengestelde beeld drie onafhankelijke grijswaardetegels per kanaal toe (de knoptekst volgt de filterkanalen van de camera). Elke gesplitste tegel kan worden versleept en heeft dezelfde randkleur als de camera. Niet beschikbaar op monochrome camera&#x27;s. Wordt samen met het project opgeslagen. |

### Spotmeter

* Vinkvakje **Klik om te meten**: klik op het livebeeld om één pixel te meten (een dradenkruis markeert deze), of klik en sleep over een gebied voor een pixelgemiddelde.**Wissen**verwijdert de meting en het dradenkruis. Sluit de**Aim**-modus van AE-ROI uit.
* ****Weergeven**-keuzelijst:**Raw (bitdiepte)**— native digitale waarden met de bitdiepte van de sensor (bijv. 12-bit → 0..4095) — of**Weergave (8-bit)**(standaard). Wanneer een live-index actief is, toont**Display** in plaats daarvan de berekende indexwaarde (bijv. NDVI

).
* Het uitleespaneel geeft pixelcoördinaten, framegrootte, pixelformaat, bitdiepte en een kanaaltabel (Chan / Waarde / %) weer met bandlabels en golflengten; Bayer-groene paren worden gemiddeld; regio-monsters tonen „N px avg“.

De status van de spotmeter geldt alleen voor de huidige sessie.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Voorspellende automatische belichting (DLS-gestuurd)

Dit gedeelte verschijnt alleen wanneer **ten minste één DAQ-lichtsensor is aangesloten** — de solver heeft een live neerwaarts spectrum nodig om dit te sturen.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Instelling | Bereik | Standaard | Wat het doet |
| --- | --- | --- | --- |
| **Inschakelen** | Selectievakje | Aan (standalone camera’s) | Een gesloten-vorm-oplosser gebruikt het DLS-spectrum plus de scalairen uit het kalibratiepakket van de camera om de helderste band dicht bij verzadiging te brengen, terwijl de donkerste band boven de SNR-drempel blijft — één opname per berekening, geen stabilisatielus. Ontworpen voor tijdopnames op zonne-energie waarbij elke opname correct belicht moet zijn. De backend schakelt stilzwijgend over naar reactieve AE wanneer de DLS-meting verouderd of ontbrekend is of het kalibratiepakket niet is geladen. |
| **Afvlakking (α)** | 0,05–1,0, stap 0,05 | 0,3 | Afvlakking van opeenvolgende voorspellende oplossingen (lager = vloeiender). |
| **Scène-reflectie**| Knop**ρ opnieuw kalibreren** | — | Herberekent de scène-reflectiefactor die de solver gebruikt. |

{% hint style="info" %}
**Array Connect schakelt voorspellende AE standaard uit** — voor arrays zorgen de slimme AE van Chloros

en de automatische belichting aan de camerazijde voor de belichting (met verzadigingsbeveiliging), en de schatting van de reflectie van één scène door de voorspellende AE-reflectieschatting van de voorspellende AE is niet veilig bij gemengde scènes. Je kunt deze functie hier per camera opnieuw inschakelen als je specifiek DLS-gestuurde radiometrische belichting wilt.
{% endhint %}

**DAQ-gestuurde belichtingslimiet en AE met vastgelegde invalshoek.** Ongeacht het bovenstaande selectievakje: wanneer een DAQ-lichtsensor is toegewezen aan een RGB

-camera, berekent Chloros

— op basis van de gemeten absolute neerwaartse stralingsintensiteit — de maximale belichting × versterking waarbij een oppervlak met 100 % reflectie onder clipping blijft, en past dit toe als een **maximum**voor de automatische belichting. Sinds versie 1.3.0 geldt het maximum alleen zolang een kalibratiedoel in het gezichtsveld van die camera is (waargenomen in het beeldkader binnen de laatste 10 s); als er geen doel in beeld is, wordt in plaats daarvan de gewone automatische belichting van de camera, met zijn eigen bescherming tegen overbelichting, toegepast, zodat de rest van de scène niet donkerder wordt gehouden dan nodig is. Terwijl de bovengrens actief is, werkt de camera**incident-pinned**: hij werkt in open-loop op de belichting op basis van de invallende lichtmeting met een versterking van 0 dB — de belichting volgt het gemeten licht, niet de inhoud van de scène. Omdat de bovengrens de belichting alleen kan verkorten, kan deze zelf geen clipping veroorzaken. Sinds versie 1.3.0 wordt de bovengrens berekend met de eigen belichtingsconstante van de camera uit het kalibratiepakket (een gemiddelde waarde voor het hele park alleen wanneer het pakket er geen bevat). De bovengrens wordt automatisch uitgeschakeld — en de normale automatische belichting van de scène wordt hervat — wanneer er geen kalibratiedoel in beeld is, de DAQ-meting ontbreekt, verouderd (&gt;30 s), of donker is, of als ≥15 % van het beeld clipt bij de vastgezette belichting (wat betekent dat de sensor en de camera verschillende verlichting waarnemen). Er is geen GUI-schakelaar; dit is standaardgedrag wanneer een RGB

-camera een DAQ-koppeling heeft.

### Acquisitie &amp; Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array-leden tonen bovendien de alleen-lezen rijen **Rol**(Master in blauw / Slave in groen),**Synchronisatielijn**en**Peers**.

| Besturing | Keuzes | Standaard | Opmerkingen |
| --- | --- | --- | --- |
| **Triggermodus** | Uit / Aan | Aan | Uitgeschakeld voor array-leden (de array regelt het triggeren). |
| **Triggerbron** | Software / Lijn 0 (M8) / Lijn 1 / Lijn 2 | Lijn 0 | Verborgen wanneer de triggermodus is uitgeschakeld; uitgeschakeld voor array-leden. Lijn 0 is de opto-geïsoleerde externe triggeringang van de M8. |
| **Beeldsnelheid**| Auto / Handmatig + waarde | Auto |**Auto**: de begrenzing van de beeldsnelheid van de camera is uitgeschakeld — de belichting bepaalt het aantal fps, en het venster toont de actuele live-snelheid.**Handmatig**: u stelt het maximum aantal fps in met een schuifregelaar (van 1 tot het door de bandbreedte beperkte maximum), waarbij de huidige werkelijke snelheid als uitgangspunt dient. Array-leden zien een alleen-lezen &quot;N fps (live)&quot; met &quot;Ingesteld in array-instellingen&quot;. |

### Netwerk / Transport

| Rij | Gedrag |
| --- | --- |
| **Pakketgrootte**| 1500 (Standaard) / 9000 (Jumbo) — standaard**Jumbo**. |
| **Doorvoersnelheid** | Alleen-lezen limiet voor de doorvoersnelheid van de verbinding in MB/s. De backend verdeelt dit bij elke verbinding of verbreking opnieuw over alle aangesloten camera’s. |
| **Bufferbeheer** | Alleen-lezen modus voor bufferbeheer. |

### Opname

Het venster eindigt met een knop **&quot;Open opname-instellingen…&quot;** die naar het [venster Opname-instellingen](capture.md#the-capture-settings-pane) gaat (uitgeschakeld totdat er een project is geopend — &quot;Maak of open een project om opnames op te slaan&quot;). Als de camera verborgen of gepauzeerd is, herinnert een tip je eraan om deze weer zichtbaar te maken of te hervatten voordat je gaat opnemen.

## Paneel met array-instellingen

Open dit venster via het **tandwiel**in een ARRAY-rij. Koptekst: array-naam met een potloodje om de naam te wijzigen en**×** om te sluiten. Onderstaande secties gemarkeerd met *alleen gecombineerd* verschijnen alleen voor arrays die zijn aangesloten in de gecombineerde weergavemodus.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synchronisatie

Alleen-lezen rijen **Master**,**Slaves**en**Sync Line**.

### Omgevingslichtsensor

Wordt weergegeven voor zowel gecombineerde als afzonderlijke arrays:

* **Lichtsensor**-vervolgkeuzelijst — koppelt één DAQ aan de gehele array. De selectie wordt onmiddellijk doorgevoerd, wordt doorgegeven naar de eigen lichtsensor-keuzelijst van elke camera (je kunt dit per camera nog steeds overschrijven) en begint met het doorsturen van spectra naar de array.
* Live **Status**-regel: Uit · &quot;Wachten op eerste spectrum…&quot; · &quot;Actief — alle camera’s in de reeks zijn verlichtingsgecorrigeerd&quot; · &quot;Geen nieuw spectrum in de afgelopen 3 s — nog steeds de laatste meting in gebruik (geen time-out voor verouderde gegevens)…&quot;.
* Opmerking in het venster: &quot;Radiometrische correctie voor de hele array. Instellingen per camera hebben voorrang hierop.&quot;

### Kalibratiedoel

Een eigen groep direct na de omgevingslichtsensor, voor zowel gecombineerde als afzonderlijke arrays:

* Selectievakje **Inschakelen** — &quot;MAPIR

ArUco-doel detecteren en NDVI

-LUT ten opzichte van paneelreflectie valideren&quot;; stuurde de doeloverlay en validatietabellen van de gecombineerde tegel aan. Bij een **afzonderlijke** array schakelt het de eigen [kalibratiedoel-tabel](#the-calibration-target-table) in (één per camerarij in de lijstweergave).

### Opname — uniforme sensorinstellingen

Wordt weergegeven voor zowel gecombineerde als afzonderlijke arrays (de groepskop luidt „Opname (uniform voor alle camera’s)”). Deze instellingen gelden uniform voor elk lid (wijzigingen per lid zouden de synchronisatie verstoren). Wijzigingen worden opgeslagen en samen toegepast met **Toepassen**— behalve de schakelaar**ROI aan / ROI uit**, die onmiddellijk van kracht wordt.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Instelling | Keuzes / bereik | Wat het doet |
| --- | --- | --- |
| **Pixelformaat** | 8-bit / 10-bit / 12-bit / 16-bit | Uniforme bitdiepte voor alle leden; elke camera neemt het formaat van die diepte over voor zijn eigen sensor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardware-binning — behoudt het volledige gezichtsveld en verhoogt tegelijkertijd de signaal-ruisverhouding (SNR) en de beeldsnelheid. Als je dit wijzigt, worden de ROI-velden gereset naar het nieuwe volledige gezichtsveld. |
| **Resolutie**-voorinstelling | Volledig / Half / Kwart | Relatief ten opzichte van binning; vult de ROI-velden met een gecentreerde uitsnede. |
| **ROI-uitsnede (px)**|**ROI aan / ROI uit**-schakelaar,**Selecteren**-knop, W / H / X / Y-velden, volledige sensorkkaart | Sensoruitsnede — zie [De ROI-editor](#the-roi-editor). Breedte/hoogte worden uitgelijnd op veelvouden van 16 (minimaal 64); verschuivingen worden uitgelijnd op veelvouden van 4. Een „max WxH“-hint toont de bovengrens en**Reset** zet de velden terug naar het volledige gezichtsveld. Tijdens het bewerken volgt het oranje uitsnijkader elke veldwijziging op de kaart en op de array-tegel. |
| **Triggerfrequentie**| Schakelaar Auto / Handmatig + fps 0,5–10, stap 0,5 |**Auto**(standaard): de backend leidt de triggerfrequentie af uit resolutie en bandbreedte — de invoer is uitgeschakeld en toont de afgeleide waarde.**Handmatig**: vergrendelt je waarde bij Toepassen. Een vergrendelde frequentie die een nieuwe framegrootte niet aankan, wordt vastgehouden op de hoogste frequentie die die grootte toelaat, en keert terug zodra de framegrootte dit weer toelaat. |

Opmerking in het venster: „Wijzigingen in formaat/resolutie zorgen ervoor dat alle camera’s kortstondig opnieuw opstarten. De triggerfrequentie wordt live toegepast.“ De knoppen **Toepassen / Annuleren** bevinden zich onderaan het venster.

#### De ROI-editor

Nieuw in 1.3.0. De ROI-uitsnijdingsrij en de kaart eronder werken samen:

* **ROI aan / ROI uit**— schakelt de sensoruitsnijding**onmiddellijk**in of uit (de camera’s worden kort opnieuw opgestart; de array-tegel toont „Instellingen toepassen…“).**Uit**leest de volledige sensor en**onthoudt je uitsnede**— de velden blijven deze weergeven;**Aan** past deze opnieuw toe. Als je de ROI inschakelt zonder dat er een uitsnede is ingesteld, wordt een gecentreerde uitsnede op halve grootte toegepast („Er was geen uitsnede ingesteld, dus is een gecentreerde uitsnede op halve grootte toegepast. Pas deze aan met de velden.”).
* **Selecteren**— tekent het uitsnijdgebied op de kaart: sleep eroverheen of klik op twee tegenoverliggende hoeken (Esc annuleert). De rechthoek wordt vastgelegd in de velden W/H/X/Y, uitgelijnd volgens de bovenstaande stappen; er wordt niets toegepast totdat je op**Toepassen** drukt, dus je kunt het eerst nauwkeurig afstemmen met de pijltjes in de velden. Eén rechthoek per keer dat je op Selecteren drukt.
* **De kaart**toont het uitsnedegebied over de gehele sensor, zodat je het weer kunt verbreden: „Volledige sensor, live“ terwijl de camera’s de volledige sensor uitlezen, of &quot;Full-frame snapshot vanaf {tijdstip}. De camera’s lezen alleen het uitsnedegebied.&quot; terwijl ze zijn bijgesneden. Als de array tot nu toe alleen bijgesneden beelden heeft gestreamd, geeft de kaart weer: &quot;Nog geen full-frame weergave: deze array heeft alleen bijgesneden beelden gestreamd.&quot; met een knop**Volledig beeld weergeven**, waarmee de ROI wordt uitgeschakeld. Het label `W×H @ X,Y` van het uitsnedegebied bevindt zich buiten het kader (erboven, of eronder wanneer het kader de bovenkant raakt), op de kaart en op de live-tegel.
* **Toepassen**bevestigt de instellingen: een uitsnede schakelt de ROI in; het volledige beeld (na**Reset**) laat deze uitgeschakeld, waarbij de vorige uitsnede nog steeds wordt onthouden; een ROI die is uitgeschakeld en waarvan de velden nog steeds de onthouden uitsnede tonen, blijft uitgeschakeld wanneer je een andere wijziging toepast (formaat, binning, frequentie).
* **Bij een uitgelijnde array wordt het uitsnijdvenster van elk lid geplaatst op basis van de uitlijning**, zodat elke camera hetzelfde deel van de scène in beeld brengt. Een array die is bijgesneden voordat deze ooit werd uitgelijnd, wordt eerst uitgelijnd op de volledige sensor (de uitsnede wordt opgeheven, de uitlijning wordt opgelost en de uitsnede wordt hersteld met vensters per camera).
* Na een wijziging in de uitsnede wordt de automatische belichting opnieuw afgestemd op de nieuwe kadrering. Binnen een uitsnede kan de belichting van een lid nog steeds tot aan de triggerperiode doorlopen.
* De aan/uit-status van de ROI en de opgeslagen uitsnede worden samen met het project opgeslagen.

### Uitlijning (co-registratie)

Weergegeven voor **zowel gecombineerde als afzonderlijke arrays** (in versie 1.2.0 werd dit alleen voor gecombineerde arrays getoond); de groepskop luidt „Uitlijning”.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Residu**-pilletje: „RMS x,xx px“ — groen bij minder dan 1 px, oranje bij minder dan 3 px, rood in alle andere gevallen of als een camera is uitgevallen; „geen profiel“ vóór de eerste berekening.
* Samenvattingsregel: „✓ Alle camera’s uitgelijnd (N)” / „⚠ p/N camera’s uitgelijnd — <serial (filter)="">mislukt” / „Bijsnijden actief — Opnieuw kalibreren om uit te lijnen (gebruikt volledige sensor)” / „Wachten tot de belichting is gestabiliseerd…“ / „Instellingen toepassen…“ (de camera’s worden opnieuw opgestart) / „Array levert geen frames meer — sluit de array opnieuw aan“.
* Tabel per camera: camera (laatste 4 cijfers van serienummer + filter), herprojectiefout in px met aantal overeenkomsten („ref” voor de master), en de genormaliseerde kruiscorrelatiescore van de overlapping ten opzichte van de drempelwaarde van 0,35.
* **Uitlijning opnieuw kalibreren** knop (tekst „Uitlijning kalibreren“ vóór het eerste profiel) — voert de co-registratie opnieuw uit op nieuwe beelden.
* **&quot;Camera&#x27;s automatisch belichten voor uitlijning&quot;** selectievakje (standaard aangevinkt) — maakt donkere of vlakke camera&#x27;s tijdelijk helderder (eerst belichting, daarna versterking) zodat ze textuur hebben om te matchen, en herstelt vervolgens de automatische belichting.

Chloros lijnt elke array automatisch uit na het verbinden, in beide weergavemodi, zodra de automatische belichting van de leden is gestabiliseerd (het blijft dit tot 5 minuten lang proberen); het gecombineerde voorbeeld wordt ook uitgelijnd bij het openen. Een afzonderlijke array heeft geen samengestelde tegel, dus deze sectie controleert de uitlijningsstatus en voert zelf ‘Recalibrate’ uit. Kalibreer opnieuw als de scherpstelling of de diepte van de scène is gewijzigd. Uitlijning is **per definitie alleen voor de huidige sessie** — deze wordt nooit opgeslagen in een profiel, omdat deze afhankelijk is van de afstand tot de scène op dat moment. Opnames kunnen nog steeds worden geëxporteerd met pixelregistratie (zie [Uitlijngestuurde exporten](capture.md#per-array-controls)).

### Slimme vignettering

* Selectievakje **Correctie inschakelen**— past de per camera geschatte vignettering toe op de radiometrische keten (live**en** exporten).
* ****Kalibreren vanuit huidige weergave**— richt de array eerst op een uniform doel (vlak scherm, muur of lucht); elke camera wordt afzonderlijk geflatteerd en de status geeft &quot;n/N cams · −x,x %&quot; vlakheidswinst weer. Met**Wissen** wordt de schatting verwijderd.
* Status: „Gekalibreerd”, „Niet gekalibreerd” of „Gedeeltelijk gekalibreerd — opnieuw uitvoeren” wanneer alleen de schattingen van sommige camera’s nog van toepassing zijn bij hun huidige kadrering (bijvoorbeeld nadat een „Opnieuw kalibreren” het uitsnijdvenster van een camera heeft verplaatst) — voer „Kalibreren” opnieuw uit.
* Fijnafstemming per camera met de **Vignette**-schuifregelaar per camera in [Live Preview](#live-preview).

### Live Preview

* Vinkvakje **Histogram**+ schuifregelaar voor breedte (0,10–0,90 van het beeld) — de histogramstrook in de livefeed op de tegels van alle leden tegelijk (inclusief tegels met gesplitste kanalen), de per camera [Histogramoverlay](#display-overlays-drawn-over-the-live-feed) die voor de hele array is ingesteld. Deze wordt als ingeschakeld weergegeven wanneer elk lid deze al heeft. Op een**afzonderlijke** array is dit de enige rij in de groep.
* *Alleen gecombineerd:* **Index**: selectievakje inschakelen + tandwiel — opent de gedeelde [Indexcalculator](#index-calculator-pane) met banden die zijn getekend op basis van**alle** camera’s van de leden. Een voorbeeldregel voor de uitdrukking eronder toont de huidige uitdrukking („Geen uitdrukking ingesteld — open de calculator om er een te maken“), die elke seconde wordt vernieuwd.
* *Alleen gecombineerd:* **Renderresolutie**-dropdown (dezelfde voorinstellingen als per camera, standaard 720p): de hoogte van de liveweergavestream**en** de opgeslagen grootte van de samengestelde export. Let op in het venster: &quot;Voorbeeld + opgeslagen samengestelde grootte. Beelden per camera worden altijd in volledige resolutie geëxporteerd.&quot;

### Weergavelaagjes *(alleen gecombineerd)** Selectievakje **Inschakelen** (standaard uitgeschakeld — de hoofdcamera wordt rechtstreeks weergegeven; ingeschakeld = gelaagde compositie).
* **Keuzemenu’s**Voorgrond**/**Achtergrond**: elke deelnemende camera (op naam) of**Index**. Wanneer Voorgrond op Index staat, tonen pixels buiten de LUT-min/max de Achtergrondlaag.

### Gesplitste weergave *(alleen gecombineerd)*

**&quot;Deelnemende camera’s weergeven&quot;**— een knop**Lidcamera’s splitsen / verbergen** die de eigen livebeelden van elk lid als afzonderlijke rastertegels naast de compositie toevoegt. De tegels lezen de bestaande framebuffer van de array (geen extra camera-aansluiting). Alleen in rasterweergave; per array samen met het project opgeslagen.

### Mogelijkheden

Een alleen-lezen paneel dat elke 5 s wordt vernieuwd:

* **Niveau-label**: &quot;Gelijktijdige opname&quot; (groen) · &quot;Gelijktijdige opname (FTD-versprongen uitsturing)&quot; (groen) · &quot;Versprongen opname (100 ms drift)&quot; (oranje) · &quot;Configuratie te groot&quot; (rood).
* **Frame-status**: &quot;x,xx % onvolledig&quot; — groen bij minder dan 1 %, oranje bij minder dan 5 %, rood bij 5 % of meer.
* ****Linklijn**: „NIC {mbps} Mbps – sustained {MB/s} MB/s“.

Dit is het live bandbreedtebudget van de array. Zie [Multi-Camera Arrays](arrays.md) en de [CLI-referentie](../reference/cli-reference.md) voor het onderliggende fps- en netwerkmodel — en wat u moet wijzigen wanneer het niveau oranje of rood wordt.

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Paneel ‘Indexcalculator’

De derde pagina in de zijbalk, die wordt gedeeld door het ‘Index’-tandwiel per camera en het ‘Index’-tandwiel voor de gecombineerde array (één tegelijk — de kop luidt ‘Indexcalculator — <camera name="">’ of ‘Indexcalculator — <array name="">’). Hier wordt de bandlijst weergegeven (de natuurlijke filterbanden van de camera-natuurlijke banden, of alle banden van de leden van de array), de huidige uitdrukking en de LUT-configuratie (aan/uit, niveau — standaard 7 stops, min — standaard 0,2, max — standaard 1), plus een live indexhistogram. Met **Toepassen** wordt de uitdrukking vastgelegd; LUT-wijzigingen worden live toegepast op het voorbeeld.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Instellingen per camera versus door de array beheerde instellingen

Beknopte overzicht van wat waar staat wanneer een camera deel uitmaakt van een array:

| Door de array beheerd (alleen-lezen in het cameravenster) | Nog steeds per camera binnen een array |
| --- | --- |
| Pixelformaat, resolutie, binning | Automatische belichting (belichting, versterking, doelwaarde, afvlakking, ROI) |
| Triggerstand/bron, beeldsnelheid | Ruisonderdrukking, scherpte, vignettering |
| | Oriëntatie (spiegelen/rotatie), weergave-overlays (array-instellingen → Live Preview stelt het histogram voor alle leden tegelijk in), spotmeter |
| | Index (arrays met afzonderlijke weergave), koppeling lichtsensor |

Overkoepelend gedrag:

* **Gecombineerde versus afzonderlijke weergave** wordt gekozen bij het koppelen van de array: gecombineerd = één uitgelijnde samengestelde tegel (leden sturen alleen beelden via Split View); afzonderlijk = elk lid geeft zijn eigen gesynchroniseerde tegel weer. Een camera toont nooit zowel een afzonderlijke feed als een array-tegel.
* ****Automatisch opnieuw verbinden**: bij het openen van een opgeslagen project worden de camera’s en arrays hersteld en worden alle opgeslagen instellingen opnieuw toegepast op de backend voordat de streams worden hervat.
* **Capture-gating**: verborgen of gepauzeerde camera’s worden uitgesloten van ‘Capture All’; een array wordt pas volledig geblokkeerd wanneer ALLE leden verborgen of gepauzeerd zijn. Zie [Opname-instellingen en -modi](capture.md).

## Hoe instellingen worden bewaard

De status van het cameratabblad wordt **samen met het project** opgeslagen, niet in de browser:

* Bij elke reactieve wijziging wordt een momentopname van de camera’s en arrays gemaakt in het `cameras.json`-bestand van het project (met een debounce van 500 ms). Dit omvat cameranamen en -kleuren, belichting/versterking/AE-instellingen, pixelformaat/resolutie/binning, triggerfrequentie, voorvertoningsinstellingen (renderresolutie, ruisonderdrukking, scherpte, vignettering, kleurprofiel, hooglichten, verzadiging/contrast), oriëntatie, overlays, kanaalsplitsingen, indexconfiguratie, voorspellende AE-instellingen, AE-ROI, array-namen, weergavemodus, instellingen voor array-opname (inclusief de ROI-uitsnede, of de ROI is ingeschakeld en de opgeslagen uitsnede wanneer deze is uitgeschakeld), het Wire Budget van de array en het rasterblok (feedzoom, weergavemodus, rastervergrendeling, handmatige tegelvolgorde, verborgen camera’s, gesloten tegels, actieve camera).
* De koppelingen van de lichtsensoren worden opgeslagen in het `sensors.json`-bestand van het project.
* Bij het opnieuw openen van het project wordt de hardware opnieuw gekoppeld en worden alle instellingen opnieuw toegepast.
* **Geen project geopend = alleen sessie**: zonder project blijft er niets behouden wanneer Chloros wordt afgesloten.
* Alleen sessie, ongeacht het project: pauzetoestand, spotmeter-metingen, het selectievakje ‘Kalibratiedoel’ per camera (wordt standaard altijd uitgeschakeld geopend) en het uitlijningsprofiel van de array (wordt per sessie opnieuw berekend, zoals bedoeld).
* Eén uitzondering: de exportselecties voor **Opname-instellingen** en de opnamemodus blijven per project bewaard in de lokale app-opslag in plaats van in `cameras.json` — zie [Opname-instellingen &amp; -modi](capture.md).</array></camera></serial>
