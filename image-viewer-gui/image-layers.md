# Afbeeldingslagen

Met het **laagmenu** rechtsboven in de afbeeldingsviewer kun je schakelen tussen alle versies van de afbeelding die je bekijkt — van de bronopname via elk bewerkt product tot de berekende indexafbeeldingen — zonder de viewer te verlaten.

## Wat zijn afbeeldingslagen?

Een „laag“ in Chloros is één **productbestand**dat aan één bronafbeelding is gekoppeld. Bij het importeren krijgt u de bronbestanden; tijdens de verwerking wordt er voor elk product dat tijdens de run is gegenereerd een laag toegevoegd. Geëxporteerde bestanden behouden de bestandsnaam van de bron — het is de**map** die het product identificeert, en de laagnaam is het label dat Chloros aan die map toekent.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## De lagenlijst

### Altijd aanwezig

| Laag | Wat het is |
| --- | --- |
| **JPG**(of**PNG**/**TIFF**) | Het basisbestand dat bij de opname is binnengekomen. Survey3 importeert een `.JPG` naast elke `.RAW`; LATTICE-opnames leveren een PNG of TIFF weergavevoorbeeld mee. Gelabeld voor wat daadwerkelijk is geïmporteerd |
| **RAW (Origineel)** | Het bron-raw-frame, gedebayerd voor weergave zonder toegepaste correcties. Beschikbaar vanaf het moment van import — het hoeft niet te worden verwerkt |

Een LATTICE-opname waarvan het basisbestand **het**RAW-frame**is**, heeft geen aparte basisvermelding: `RAW (Original)` dekt dit al.

### Survey3-verwerkingsproducten

| Laag | Geschreven naar | Bestaat wanneer |
| --- | --- | --- |
| **RAW (Doel)** | — | Het frame werd geïdentificeerd als een frame dat een kalibratiedoel bevat |
| **RAW (Reflectantie)** | `Reflectance_Calibrated_Images/` | De reflectantiekalibratie is succesvol uitgevoerd op dit frame |
| **Vignettering gecorrigeerd**| `Vignette_Corrected_Images/` | Het frame kon niet worden gekalibreerd op basis van reflectie**en** *vignetteringscorrectie* was ingeschakeld |
| **Sensorrespons**| `Sensor_Response_Images/` | Het frame kon niet op reflectie worden gekalibreerd**en** *vignetteringscorrectie* was uitgeschakeld |
| **Witbalans** | `White_Balanced_Images/` | Er is een product met witbalans opgeslagen |

{% hint style="info" %}
**Vignetcorrectie en sensorrespons zijn alternatieven, nooit beide tegelijk.** Per run bestaat er precies één niet-gekalibreerd fallback-product voor elk cameramodel, en de schakelaar *Vignetcorrectie* bepaalt welk product dat is. Zie [Projectinstellingen](../project-settings/project-settings.md).
{% endhint %}

### LATTICE-niveaus

LATTICE legt de fan-out in één verwerkingsstap vast in deze niveaus. Welke er beschikbaar zijn, hangt af van de exportschakelaars per product in de projectinstellingen en van wat van toepassing is op de camera.

| Laag | Geschreven naar | Van toepassing op |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB en multispectraal |
| **RAW (Voorbeeld)** | `Preview_Images/` | Multispectraal (valse kleuren-uitrekking) |
| **Witbalans** | `Preview_Images/` | RGB-mastercamera’s — het RGB-voorbeeld is onder deze naam geregistreerd, zodat het aansluit bij de gelijknamige Survey3-laag |
| **RAW (straling)** | `Radiance_Images/` | Alleen multispectraal |
| **RAW (reflectantie)** | `Reflectance_Calibrated_Images/` | Alleen multispectraal, en alleen wanneer een bijpassend `.daq`-downwelling-record of een in-frame-doel dat de kwaliteitscontrole heeft doorstaan het frame bestrijkt |

RGB-mastercamera&#x27;s hebben geen radiometrie per band, dus straling en reflectie worden voor deze camera&#x27;s overgeslagen als **niet van toepassing** — dit wordt in het logboek vermeld in plaats van dat er stilzwijgend een fout wordt gerapporteerd.

### Index-, LUT- en sandbox-lagen

| Laagpatroon | Voorbeeld | Waar het vandaan komt |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | Eén per index die is geconfigureerd in de projectinstellingen, berekend tijdens de verwerking |
| **`<INDEX>` LUT** | `NDVI LUT` | De kleurgemapte versie van een index |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Eén per exportrun van [Index/LUT Sandbox](index-lut-sandbox.md) |

Als dezelfde indexnaam meer dan eens met verschillende instellingen is geconfigureerd, krijgen de tweede en volgende een nummer in de naam (`RAW (NDVI2 Index)`), zodat de lagen van elkaar te onderscheiden blijven.

***

## De laagselector gebruiken

1. Open een afbeelding op volledig scherm door op een miniatuur in het raster te klikken
2. Klik op de **laagkeuzelijst** rechtsboven in de viewer
3. Kies een laag — de afbeelding wordt onmiddellijk bijgewerkt

In het keuzemenu staan **JPG, RAW (Origineel), RAW (Doel), RAW (Reflectantie)** als eerste, in die volgorde, en daarachter wordt de rest weergegeven in de volgorde waarin de producten zijn geregistreerd.

### Laagvoorkeur bij het navigeren

Als je op **←**/**→** drukt, ga je naar de volgende afbeelding en wordt geprobeerd je op dezelfde laag te houden:

1. **Eerst exacte overeenkomst** — als de volgende afbeelding een laag met dezelfde naam heeft, krijg je die te zien. Hierdoor blijf je op `RAW (NDVI Index)` terwijl je door een hele set bladert
2. **Vervolgens een overeenkomst op type** — een indexlaag zoekt naar elke indexlaag, een LUT naar elke LUT, reflectantie naar reflectantie, doel naar doel, origineel naar origineel, basis naar basis
3. **Vervolgens, alleen voor exportlagen** — de naam blijft behouden, zelfs als de lagenlijst nog niet is bijgewerkt, omdat het bestand al op de schijf staat. Hierdoor kun je producten bekijken terwijl ze nog worden geschreven tijdens een run
4. **Anders** — de eerste beschikbare laag, wat normaal gesproken de basisafbeelding is

De sidecar-bestanden `.daq` en `.csv` in het project worden overgeslagen bij navigatie met de pijltjestoetsen, zodat je bij het doorlopen van afbeeldingen nooit bij een opname van de lichtsensor terechtkomt.

Zoomen en pannen worden ook over de afbeeldingen heen overgedragen, waardoor het eenvoudig is om dezelfde veldpositie voor en na te vergelijken.

***

## Pixelwaarden per laag begrijpen

Het [paneel Cursorwaarden](opening-an-image-full-screen.md#cursor-values) geeft de werkelijke waarde per kanaal onder je cursor weer, in de eenheid waarin die laag is opgeslagen. De kolommen veranderen per laag:

| Laag | Weergegeven eenheid | Opmerkingen |
| --- | --- | --- |
| Basis (JPG / PNG / TIFF-voorbeeld) | DN, 0–255 | Weergavewaarden, gammagecorrigeerd op RGB. Alleen visuele inspectie |
| RAW (Origineel) | DN | Ruwe digitale sensorgegevens. De histogram-as geeft de diepte aan: 255 (8-bit), 4095 (12-bit) of 65535 (16-bit) |
| RAW (Debayered) | DN | Lineair, geen weergave-uitrekking |
| RAW (Voorbeeld) / Witbalans | DN | Weergaveproduct — uitgerekt of gamma-gecorrigeerd. Niet bedoeld voor metingen |
| RAW (Stralingsintensiteit) | **W/m²/sr/nm** | Float32 fysieke stralingsintensiteit. Geen DN-kolom |
| RAW (reflectie) | DN **en %** | Percentage berekend op basis van de eigen schaal van dat bestand — zie hieronder |
| Index / LUT / sandbox-exporten | Indexwaarde, of RGB-componenten | Een indexbestand met één kanaal rapporteert de indexwaarde; een LUT-bestand met kleuraanpassing rapporteert Red/Green/Blue-componenten |

### Reflectie: de schaal geldt per bestand

{% hint style="warning" %}
**&quot;Delen door 65.535&quot; is alleen correct voor Survey3.** De reflectie van LATTICE wordt op een andere schaal opgeslagen, en het door elkaar gebruiken van de twee delers is de meest voorkomende manier om reflectiewaarden te krijgen die precies de helft zijn van wat ze zouden moeten zijn.
{% endhint %}

| Bron | DN die gelijk is aan reflectie 1,0 | Geïdentificeerd door |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | De XMP-tag `Chloros:PixelScale=32768` die in elke LATTICE-reflectantie-export is opgenomen. De 2× headroom betekent dat ρ boven 1,0 weergegeven kan worden in plaats van afgekapt te worden |
| **Survey3**|**65535** | Geen Chloros XMP-schaaltag — de kalibratie schrijft ρ × dtype-max en klipt bij 1,0 |

Voor GIS en scripting: lees `Chloros:PixelScale` uit het bestand en deel erdoor. Als de tag ontbreekt, is het bestand Survey3-schaal (65535). De viewer, de index/LUT-sandbox en de indexexport berekenen de schaal allemaal op dezelfde manier, dus het getal dat je bij de cursor ziet, is het getal dat de indexberekening heeft gebruikt.

Formaatspecifieke opslag bovenop die schaal:

* **TIFF (32-bit, procent)** slaat DN / 65535 op als een float
* **PNG (8-bit)**en**JPG (8-bit)** slaan DN × 255 / 65535 op
* Een **8-bits TIFF-export van een opname met een 8-bits bron** wordt begrensd tot 0–255 in plaats van op schaal gebracht, en bevat bewust geen schaaltag. Het paneel geeft voor die bestanden alleen de DN weer, zonder procentkolom

### Indexwaardebereiken

| Indexfamilie | Typisch bereik | Waarde |
| --- | --- | --- |
| Genormaliseerd verschil (NDVI, GNDVI, NDRE, ENDVI…) | −1 tot +1 | Gezonde vegetatie doorgaans 0,4–0,9; kale grond rond 0; water negatief |
| Bodemgecorrigeerd (SAVI, OSAVI, MSAVI2…) | ruwweg −1 tot +1,5 | Vergelijkbare waarde als NDVI, waarbij de bodemachtergrond is onderdrukt |
| Verhouding (GRVI, GCI, MSR, CIRE…) | onbeperkt boven | Verhoudingen stijgen onbeperkt naarmate de noemerband naar nul gaat |
| EVI / LAI | 0 tot ~1, 0 tot ~3,5 | Wolken en andere verzadigde pixels zorgen ervoor dat beide waarden buiten het bereik vallen — maskeer deze eerst |

Zie [Formules voor multispectrale indexen](../project-settings/multispectral-index-formulas.md) voor de exacte formule achter elke voorinstelling.

***

## Veelgebruikte workflows

### Vergelijking voor en na

1. Selecteer **RAW (Origineel)** en let op de vignettering en de niet-gekalibreerde waarden
2. Schakel over naar **RAW (Reflectantie)**

3. Vergelijk — vignettering verwijderd, waarden gekalibreerd. Zoom en pan blijven behouden, zodat je naar hetzelfde gebied kijkt

### Eén index over een hele reeks bekijken

1. Open de eerste bewerkte afbeelding en selecteer de indexlaag
2. Druk herhaaldelijk op **→** — de indexlaag volgt je van afbeelding naar afbeelding
3. Houd het histogram in de zijbalk in de gaten terwijl je doorloopt: een frame waarvan de verdeling een sprong vertoont, is het waard om nader te bekijken

### Kalibratiedoelen controleren

1. Selecteer **RAW (Target)** op een doelbeeld
2. Controleer of het doel duidelijk zichtbaar is en wordt gedetecteerd
3. Ga naar het volgende doelbeeld — de doellaag volgt mee

### Controleer de nauwkeurigheid van de reflectiewaarden

1. Selecteer **RAW (Reflectance)**

2. Lees de kolom**%** in het paneel ‘Cursorwaarden’ — deze is al correct geschaald voor dat bestand
3. Controleer of de waarden kloppen met bekende materialen in het beeld: gezonde vegetatie heeft een hoge waarde voor NIR en een lage waarde voor rood; een kalibratiedoel moet een waarde vertonen die dicht bij de gepubliceerde reflectiewaarde ligt

***

## Probleemoplossing

### Een laag die ik verwachtte, staat niet in de vervolgkeuzelijst

**Mogelijke oorzaken**

* De afbeelding is nooit verwerkt — alleen de basislaag en `RAW (Original)` bestaan
* De exportschakelaar van het product is uitgeschakeld in de projectinstellingen
* Het product is niet van toepassing op die camera (straling en reflectie op een RGB-master; elke index op een enkelbandige M3M-monochromecamera)
* De reflectankalibratie had geen gegevens om mee te werken — geen `.daq`-dekking in neerwaartse richting en geen in-frame doel dat de kwaliteitscontrole heeft doorstaan — dus viel het frame terug op ‘Vignette Corrected’ of ‘Sensor Response’

**Wat te doen**

1. Controleer het logboek van de run: Chloros geeft aan wanneer een aangevraagd exportproduct onmogelijk was en waarom
2. Controleer de exportschakelaars per product in [Projectinstellingen](../project-settings/project-settings.md)
3. Controleer of de productmap bestaat in de uitvoerstructuur van het project
4. Voer de bewerking opnieuw uit met het product ingeschakeld

### De lagenlijst lijkt verouderd

Chloros scant de productmappen van het project opnieuw terwijl een run bezig is en herstelt ontbrekende laagregistraties op basis van wat er daadwerkelijk op de schijf staat, dus een laag waarvan de export is voltooid, verschijnt vanzelf in een poll. Als je even wegklikt van de afbeelding en weer teruggaat, wordt een nieuwe weergave geforceerd.

### De reflectiewaarden lijken de helft te zijn van wat ze zouden moeten zijn

U deelt vrijwel zeker een LATTICE-bestand door 65535. Gebruik `Chloros:PixelScale` (32768), of bekijk de **%**-kolom, waarin dit al is toegepast.

### Indexlaag bestaat maar de afbeelding is leeg

De index heeft banden nodig die je laag niet heeft — bijvoorbeeld een index die een derde kanaal leest, toegepast op een bestand met één of twee kanalen. Schakel over naar een multibandlaag (reflectantie of debayered), of kies een index die past bij het filter van de camera.

***

## Volgende stappen

* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) — cursoraflezing, histogram en GSD-regeling
* [**Index/LUT-sandbox**](index-lut-sandbox.md) — interactieve indexvisualisatie en export
* [**Formules voor multispectrale indexen**](../project-settings/multispectral-index-formulas.md) — de indexreferentie
* [**De verwerking afronden**](../processing-images-gui/finishing-the-processing.md) — de structuur van de uitvoermap waarnaar deze lagen verwijzen
