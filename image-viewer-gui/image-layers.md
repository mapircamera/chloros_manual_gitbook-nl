# Afbeeldingslagen

Met het **laagmenu** rechtsboven in de afbeeldingsviewer kun je schakelen tussen alle versies van de afbeelding die je bekijkt — van de bronopname via elk verwerkt product tot de berekende indexafbeeldingen — zonder de viewer te verlaten.

## Wat zijn afbeeldingslagen?

Een „laag“ in Chloros

is één **productbestand**dat aan één bronafbeelding is gekoppeld. Bij het importeren krijgt u de bronbestanden; tijdens de verwerking wordt voor elk product dat de run heeft gegenereerd een laag toegevoegd. Geëxporteerde bestanden behouden de bestandsnaam van het bronbestand — het is de**map** die het product identificeert, en de laagnaam is het label dat Chloros

aan die map toekent.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## De lagenlijst

### Altijd aanwezig

| Laag | Wat het is |
| --- | --- |
| **JPG**(of**PNG

**/**TIFF

**) | Het basisbestand dat bij de opname is binnengekomen. Survey3

importeert een `.JPG` naast elke `.RAW`; LATTICE-opnames leveren een PNG

- of TIFF

-weergavevoorbeeld op. Gelabeld naar wat daadwerkelijk is geïmporteerd |
| **RAW (Origineel)** | Het bron-RAW-frame, gedebayered voor weergave zonder toegepaste correcties. Beschikbaar vanaf het moment van importeren — het hoeft niet te worden verwerkt |

Een LATTICE-opname waarvan het basisbestand **het**RAW-frame**is**, heeft geen aparte basisvermelding: `RAW (Original)` dekt dit al.

### Survey3

-verwerkingsproducten

| Laag | Geschreven naar | Bestaat wanneer |
| --- | --- | --- |
| **RAW (Target)** | — | Het frame werd geïdentificeerd als een frame dat een kalibratiedoel bevat |
| **RAW (Reflectance)** | `Reflectance_Calibrated_Images/` | De reflectantiekalibratie is succesvol uitgevoerd op dit frame |
| **Vignettering gecorrigeerd**| `Vignette_Corrected_Images/` | Het frame kon niet worden gekalibreerd op basis van reflectie**en** *vignetteringscorrectie* was ingeschakeld |
| **Sensorrespons**| `Sensor_Response_Images/` | Het beeld kon niet worden gekalibreerd op reflectie**en** *vignetteringscorrectie* was uitgeschakeld |
| **Witbalans** | `White_Balanced_Images/` | Er is een product met witbalans opgeslagen |

{% hint style="info" %}
**Vignetcorrectie en sensorrespons zijn alternatieven, nooit beide tegelijk.** Per run bestaat er precies één niet-gekalibreerd fallback-product voor elk cameramodel, en de schakelaar *Vignetcorrectie* bepaalt welk product dat is. Zie [Projectinstellingen](../project-settings/project-settings.md).
{% endhint %}

### LATTICE-niveaus

LATTICE legt deze in één verwerkingsronde vast. Welke er bestaan, hangt af van de exportschakelaars per product in de projectinstellingen en van wat van toepassing is op de camera.

| Laag | Geschreven naar | Van toepassing op |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB

en multispectraal |
| **RAW (Preview)** | `Preview_Images/` | Multispectraal (valse-kleurenuitrekking) |
| **Witbalans** | `Preview_Images/` | RGB

-hoofdcamera’s — het RGB

-voorbeeld wordt onder deze naam geregistreerd, zodat het overeenkomt met de gelijknamige Survey3

-laag |
| **RAW (straling)** | `Radiance_Images/` | Alleen multispectraal |
| **RAW (reflectantie)** | `Reflectance_Calibrated_Images/` | Alleen multispectraal, en alleen wanneer een bijbehorend `.daq`-downwelling-record of een QA-goedgekeurd in-frame-doel het beeld bestrijkt |

RGB

-mastercamera’s hebben geen radiometrie per band, dus straling en reflectie worden voor deze camera’s overgeslagen als **niet van toepassing** — dit wordt in het logboek vermeld in plaats van dat er stilzwijgend een fout wordt gerapporteerd.

{% hint style="info" %}
**Raw- en debayered-lagen komen overeen met de producten (nieuw in 1.3.0).**LATTICE raw- en debayered-bestanden worden opgeslagen in de oorspronkelijke oriëntatie van de sensor en registreren in XMP de oriëntatie die bij hen hoort; de andere producten worden opgeslagen met die oriëntatie al toegepast. De viewer en het beeldraster schakelen**RAW (Origineel)**en**RAW (Debayered)** op basis van die vermelding in voor weergave, zodat het beeld niet langer wordt omgedraaid wanneer u schakelt tussen een productlaag en een raw-laag. Dit betreft alleen de weergave — de bestanden worden nooit herschreven. Bestanden zonder oriëntatiegegevens (vastgelegd vóór versie 1.3.0 of geschreven door andere software) worden weergegeven zoals ze zijn opgeslagen. Zie [Uitvoerbeeldformaten](../output-image-formats.md#orientation).
{% endhint %}

Clamp-maskers (`*_clampmask.tif` in een `clamp_masks/`-map) zijn geen lagen: Chloros

slaat ze over bij het importeren. Zie [Uitvoerbeeldformaten](../output-image-formats.md#clamp-masks).

### Index-, LUT- en sandbox-lagen

| Laagpatroon | Voorbeeld | Waar het vandaan komt |
| --- | --- | --- |
| **RAW (`<INDEX>`-index)** | `RAW (NDVI Index)` | Eén per index die is geconfigureerd in de projectinstellingen, berekend tijdens de verwerking |
| **`<INDEX>` LUT** | `NDVI LUT` | De kleurgemapte versie van een index |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Eén per [Index/LUT Sandbox](index-lut-sandbox.md) exportrun |

Als dezelfde indexnaam meer dan eens met verschillende instellingen is geconfigureerd, krijgen de tweede en volgende een nummer in de naam (`RAW (NDVI2 Index)`), zodat de lagen van elkaar te onderscheiden blijven.

***

## De laagselector gebruiken

1. Open een afbeelding op volledig scherm door op een miniatuur in het raster te klikken
2. Klik op het **laagmenu** rechtsboven in de viewer
3. Kies een laag — de afbeelding wordt onmiddellijk bijgewerkt

In het keuzemenu staan **JPG, RAW (Origineel), RAW (Doel), RAW (Reflectantie)** als eerste, in die volgorde, en daarachter wordt de rest weergegeven in de volgorde waarin de producten zijn geregistreerd.

### Laagvoorkeur tijdens het navigeren

Als je op **←**/**→** drukt, ga je naar de volgende afbeelding en wordt geprobeerd je op dezelfde laag te houden:

1. **Eerst exacte overeenkomst** — als de volgende afbeelding een laag met dezelfde naam heeft, krijg je die te zien. Dit zorgt ervoor dat je op `RAW (NDVI Index)` blijft terwijl je door een hele set bladert
2. **Vervolgens een overeenkomst op type** — een indexlaag zoekt naar elke indexlaag, een LUT naar elke LUT, reflectantie naar reflectantie, doel naar doel, origineel naar origineel, basis naar basis
3. **Vervolgens, alleen voor exportlagen** — de naam blijft behouden, zelfs als de lagenlijst nog niet is bijgewerkt, omdat het bestand al op de schijf staat. Hierdoor kun je producten bekijken terwijl ze nog worden geschreven tijdens een run
4. **Anders** — de eerste beschikbare laag, wat normaal gesproken de basisafbeelding is

De sidecar-bestanden `.daq` en `.csv` in het project worden overgeslagen bij navigatie met de pijltjestoetsen, zodat je bij het doorlopen van afbeeldingen nooit op een opname van de lichtsensor terechtkomt.

Zoomen en pannen worden ook over de afbeeldingen heen overgedragen, waardoor het eenvoudig is om dezelfde veldpositie voor en na te vergelijken.

***

## Pixelwaarden per laag begrijpen

Het [paneel Cursorwaarden](opening-an-image-full-screen.md#cursor-values) geeft de werkelijke waarde per kanaal onder je cursor weer, in de eenheid waarin die laag is opgeslagen. De kolommen veranderen per laag:

| Laag | Weergegeven eenheid | Opmerkingen |
| --- | --- | --- |
| Basis (JPG / PNG

/ TIFF

-voorbeeld) | DN, 0–255 | Weergavewaarden, gammagecorrigeerd op RGB

. Alleen voor visuele inspectie |
| RAW (Origineel) | DN | Digitale RAW-sensorgegevens. De histogram-as geeft de diepte aan: 255 (8-bits), 4095 (12-bits) of 65535 (16-bits) |
| RAW (Debayered) | DN | Lineair, geen weergave-uitrekking |
| RAW (Voorbeeld) / Witbalans | DN | Weergaveproduct — uitgerekt of gammagecorrigeerd. Niet bedoeld voor metingen |
| RAW (Stralingsintensiteit) | **W/m²/sr/nm** | Float32 fysieke stralingsintensiteit. Geen DN-kolom |
| RAW (Reflectantie) | DN **en %** | Percentage berekend op basis van de eigen schaal van dat bestand — zie hieronder |
| Index / LUT / sandbox-exporten | Indexwaarde, of RGB

-componenten | Een indexbestand met één kanaal geeft de indexwaarde weer; een LUT-bestand met kleurtoewijzing geeft de componenten Red

/Green

/Blue

weer |

### Reflectantie: de schaal is per bestand

{% hint style="warning" %}
**&quot;Delen door 65.535&quot; is alleen correct voor Survey3

.** De reflectantie van LATTICE wordt op een andere schaal opgeslagen, en het door elkaar gebruiken van de twee delers is de meest voorkomende manier om reflectantiewaarden te krijgen die precies de helft zijn van wat ze zouden moeten zijn.
{% endhint %}

| Bron | DN die gelijk is aan reflectantie 1,0 | Geïdentificeerd door |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | De XMP-tag `Chloros:PixelScale=32768` die in elke LATTICE-reflectantie-export is opgenomen. De 2× speling betekent dat ρ boven 1,0 weergegeven kan worden in plaats van afgekapt te worden |
| **Survey3**| **65535** | Geen Chloros

XMP-schaaltag — Survey3

-kalibratie schrijft ρ × dtype-max en knipt af bij 1,0 |

Voor GIS en scripting: lees `Chloros:PixelScale` uit het bestand en deel erdoor. Als de tag ontbreekt, heeft het bestand de Survey3

-schaal (65535). De viewer, de index/LUT-sandbox en de indexexport bepalen de schaal allemaal op dezelfde manier, dus het getal dat je bij de cursor ziet, is het getal dat de indexberekening heeft gebruikt.

Formaatspecifieke opslag bovenop die schaal:

* **TIFF

(32-bit, Percent)** slaat DN / 65535 op als een float
* **PNG

(8-bit)**en**JPG (8-bit)** slaan DN × 255 / 65535 op
* Een **8-bits DN-product van een 8-bits opname**, of een reflectantiebestand dat vóór versie 1.3.0 is geschreven, wordt begrensd tot 0–255 en bevat geen schaaltag. Het paneel geeft voor die bestanden alleen de DN weer, zonder kolom met percentages

### Bereiken van indexwaarden

| Indexfamilie | Typisch bereik | Waarde |
| --- | --- | --- |
| Genormaliseerd verschil (NDVI

, GNDVI

, NDRE

, ENDVI…) | −1 tot +1 | Gezonde vegetatie doorgaans 0,4–0,9; kale grond rond 0; water negatief |
| Bodemaangepast (SAVI

, OSAVI

, MSAVI2…) | ruwweg −1 tot +1,5 | Vergelijkbare waarde als NDVI

, maar met onderdrukte bodemachtergrond |
| Verhouding (GRVI

, GCI

, MSR, CIRE…) | onbeperkt boven | Verhoudingen stijgen onbeperkt naarmate de noemerband naar nul gaat |
| EVI

/ LAI

| 0 tot ~1, 0 tot ~3,5 | Wolken en andere verzadigde pixels zorgen ervoor dat beide waarden buiten het bereik vallen — maskeer deze eerst |

Zie [Formules voor multispectrale indexen](../project-settings/multispectral-index-formulas.md) voor de exacte formule achter elke voorinstelling.

***

## Veelvoorkomende workflows

### Vergelijking voor en na

1. Selecteer **RAW (Origineel)** en let op de vignettering en de niet-gekalibreerde waarden
2. Schakel over naar **RAW (Reflectance)**

3. Vergelijk — vignettering verwijderd, waarden gekalibreerd. Zoom en pan blijven behouden, zodat je naar hetzelfde gebied kijkt

### Bekijk één index over een hele reeks

1. Open de eerste bewerkte afbeelding en selecteer de indexlaag
2. Druk herhaaldelijk op **→** — de indexlaag volgt je van afbeelding naar afbeelding
3. Houd het histogram in de zijbalk in de gaten terwijl je doorloopt: een frame waarvan de verdeling plotseling verandert, is het waard om nader te bekijken

### Controleer kalibratiedoelen

1. Selecteer **RAW (Target)** op een doelbeeld
2. Controleer of het doel duidelijk zichtbaar is en wordt gedetecteerd
3. Ga naar het volgende doelbeeld — de doellaag volgt mee

### Controleer de nauwkeurigheid van de reflectiewaarden

1. Selecteer **RAW (Reflectance)**

2. Lees de kolom**%** in het paneel Cursorwaarden — deze is al correct geschaald voor dat bestand
3. Controleer of de waarden kloppen met bekende materialen in het beeld: gezonde vegetatie heeft een hoge NIR

-waarde en een lage rode waarde; een kalibratiedoel zou een waarde moeten aangeven die dicht bij de gepubliceerde reflectiewaarde ligt

***

## Problemen oplossen

### Een laag die ik verwachtte, staat niet in de vervolgkeuzelijst

**Mogelijke oorzaken**

* De afbeelding is nooit verwerkt — alleen de basislaag en `RAW (Original)` zijn aanwezig
* De exportschakelaar van het product is uitgeschakeld in de projectinstellingen
* Het product is niet van toepassing op die camera (stralings- en reflectanciegegevens op een RGB

-master; elke index op een single-band M3M-monochromecamera)
* De reflectantiekalibratie had geen gegevens om mee te werken — geen `.daq`-dekking in neerwaartse richting en geen QA-goedgekeurd doel binnen het beeld — dus viel het beeld terug op ‘Vignette Corrected’ of ‘Sensor Response’

**Wat te doen**

1. Controleer het logboek van de run: Chloros

geeft aan wanneer een aangevraagd exportproduct onmogelijk was en waarom
2. Controleer de exportschakelaars per product in [Projectinstellingen](../project-settings/project-settings.md)
3. Controleer of de productmap aanwezig is in de uitvoerstructuur van het project
4. Voer de bewerking opnieuw uit met het product ingeschakeld

### De lagenlijst lijkt verouderd

Tijdens een lopende run scant Chloros

de productmappen van het project opnieuw en herstelt ontbrekende laagregistraties op basis van wat er daadwerkelijk op de schijf staat. Een laag waarvan de export is voltooid, verschijnt dus normaal gesproken vanzelf in een poll. Als je wegklikt van de afbeelding en weer terugkeert, wordt een nieuwe weergave geforceerd.

### De reflectiewaarden lijken de helft te zijn van wat ze zouden moeten zijn

U deelt vrijwel zeker een LATTICE-bestand door 65535. Gebruik `Chloros:PixelScale` (32768), of bekijk de **%**-kolom, waarin dit al is toegepast.

### Indexlaag bestaat, maar de afbeelding is leeg

De index heeft banden nodig die je laag niet heeft — bijvoorbeeld een index die een derde kanaal leest, toegepast op een bestand met één of twee kanalen. Schakel over naar een multibandlaag (reflectantie of debayered), of kies een index die past bij het filter van de camera.

***

## Volgende stappen

* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) — cursorweergave, histogram en GSD-regeling
* [**Index/LUT-sandbox**](index-lut-sandbox.md) — interactieve indexvisualisatie en export
* [**Multispectrale indexformules**](../project-settings/multispectral-index-formulas.md) — de indexreferentie
* [**De verwerking afronden**](../processing-images-gui/finishing-the-processing.md) — de structuur van de uitvoermap waarnaar deze lagen verwijzen
