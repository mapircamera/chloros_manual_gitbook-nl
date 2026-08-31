# Afbeeldingenraster

Nadat je afbeeldingen in een project hebt geïmporteerd, zie je ze in het hoofdvenster in een raster gerangschikt. In dit raster kies je **welke versie van elke afbeelding je bekijkt** — met de knoppen erboven schakel je alle miniaturen in één keer om tussen de bronbestanden en de bewerkte versies.

## Grootte van miniaturen

Gebruik de zoomschuifbalk rechtsboven om de grootte van de afbeeldingsminiaturen aan te passen. De schuifbalk loopt van **64 px tot 1200 px**.

* Met **Ctrl + muiswiel** kun je de miniaturen ook schalen.
* **Ctrl + `+`**/**Ctrl + `=`**en**Ctrl + `−`** verhogen of verlagen de grootte met 4 px per druk op de toets. Het toetsenbordpad stopt bij 64 px aan de onderkant en, aan de bovenkant, bij de grootte waarbij precies twee miniaturen per rij in het huidige venster passen.
* De grootte die je kiest, wordt samen met het project opgeslagen (`UI → Grid thumbnail size` in `project.json`, standaard `160`), zodat deze bij het opnieuw openen van het project wordt hersteld.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>De *resolutie* van miniaturen is een aparte instelling, los van de *grootte* van miniaturen: zie **Weergave → Resolutie miniatuurafbeeldingen** in [Projectinstellingen](../project-settings/project-settings.md) (standaard 512 px aan de lange zijde). De grootte bepaalt hoe groot de tegel wordt weergegeven; de resolutie bepaalt hoeveel detail er wordt opgehaald om deze te vullen.***

## De rasterwerkbalk

De rij knoppen boven het raster bestaat uit maximaal drie groepen, van links naar rechts:

1. **Per trigger / Per camera** — groeperingsmodus. Wordt alleen weergegeven bij projecten met LATTICE-opnames.
2. **Camerafilterknoppen** — één per LATTICE-camera. Wordt alleen weergegeven in de modus Per camera.
3. **Knoppen voor export-/weergavemodus** — welk product elke miniatuur weergeeft.

Wanneer het venster te smal is voor alle knoppen, worden de groepen van rechts naar links samengevouwen tot dropdownmenu’s die verschijnen bij het aanwijzen: eerst worden de export-/weergaveknoppen ingeklapt, daarna de cameraknoppen. De ingeklapte groep laat één enkele knop achter met de momenteel actieve keuze, en als je er met de muis overheen gaat, schuift de volledige set naar beneden. **Per Trigger / Per Camera klapt nooit in.**<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->***

## Knoppen voor exporteren en bekijken

Deze knoppen schakelen tussen de verschillende afbeeldingstypen in het raster met miniaturen. **Er verschijnt een knop zodra het product waarnaar de knop verwijst, bestaat** — wat voor de bronbestanden betekent: direct bij het importeren, niet pas na de verwerking.Chloros

scant de producten van het project opnieuw terwijl een run bezig is, dus knoppen verschijnen tijdens de verwerking zodra elk product op de schijf wordt opgeslagen.

### De basisknop

De meest linkse exportknop is gelabeld voor **wat je daadwerkelijk hebt geïmporteerd**:

| Wat je hebt geïmporteerd | Knoplabel |
| --- | --- |
|Survey3

RAW+JPG | `JPG` |
| LATTICE-opnames met een weergavevoorbeeld naast het RAW-frame | `PNG` of `TIFF`, afhankelijk van welke voorbeelden er zijn |
| LATTICE-opnames waarbij het basisbestand **het**RAW-beeld**is** | *geen knop* — `RAW (Original)` toont dat bestand al |

In een gemengd project volgt het label de extensie die door de meeste afbeeldingen wordt gebruikt.

### Productknoppen

| Knop | Toont | Wanneer deze verschijnt |
| --- | --- | --- |
| **Doelen** | Afbeeldingen met een gedetecteerd kalibratiedoel | Na een run waarbij doelen zijn gedetecteerd |
| **Reflectantie** | Gekalibreerde reflectantiebeelden | Alleen bijSurvey3

-projecten — LATTICE-projecten gebruiken in plaats daarvan `RAW (Reflectance)`, dus het raster toont nooit twee reflectantieknoppen |
| **Witgebalanceerd** | Het witgebalanceerde resultaat (camera’s vanRGB

) | Na verwerking |
| **Vignetcorrectie** | De niet-gekalibreerde fallback met vignetcorrectie | Na een run waarbij reflectantiekalibratie niet kon worden toegepast en *Vignetcorrectie* was ingeschakeld |
| **Sensorrespons** | De niet-gekalibreerde fallback voor sensorrespons | Hetzelfde, maar met *Vignetcorrectie* uitgeschakeld |
| **`RAW (<INDEX> Index)`** | Eén knop per berekende index | Na een run met geconfigureerde indexen |
| **`<INDEX> LUT`** | Eén knop per index met kleurtoewijzing | Na een run waarbij een LUT was geconfigureerd |
| **`<Index> <Index\|LUT> <NNN>`** | Eén knop per [Index/LUT Sandbox](index-lut-sandbox.md) exportrun | Op het moment dat een sandbox-export is voltooid |

### Knoppen op LATTICE-niveau

Projecten die LATTICE-opnames bevatten, voegen deze toe, gelabeld met de naam van het niveau in plaats van een productnaam:

| Knop | Niveau |
| --- | --- |
| **RAW (Origineel)** | Het bron-raw-frame, zoals geïmporteerd |
| **RAW (Straling)** | Float32 spectrale straling, W/m²/sr/nm |
| **RAW (Reflectie)** | uint16 reflectie, 32768 = ρ 1,0 |

`RAW (Original)` is direct na het importeren beschikbaar — er is geen verwerking nodig. Wanneer een LATTICE-import helemaal geen basisknop heeft (het basisbestand van elke opname is het raw-frame), verschuift het raster automatisch naar de eerste beschikbare niveauknop, zodat de markering in de werkbalk overeenkomt met wat u ziet.

Twee niveaus-Chloros

-exports krijgen **geen eigen rasterknop**:

* **Debayered** — de `RAW (Original)`-weergave wordt al debayered weergegeven, dus een tweede knop op een visueel identieke afbeelding zou overbodig zijn. Het `RAW (Debayered)`-product wordt nog steeds naar de schijf geschreven en is nog steeds selecteerbaar via de vervolgkeuzelijst voor lagen op volledig scherm.
* **Voorbeeld** — opRGB

-camera’s wordt het voorbeeld geregistreerd als de `White Balanced`-laag, die wel een knop heeft. Op multispectrale camera&#x27;s wordt het geregistreerd als `RAW (Preview)` en is het bereikbaar via het dropdown-menu voor lagen op volledig scherm.

{% hint style="info" %}
Deze niveauknoppen worden alleen weergegeven voor projecten die daadwerkelijk LATTICE-frames bevatten.Survey3

-projecten registreren een aantal van dezelfde interne laag-namen, en de knoppen worden daarvoor weggefilterd, zodat eenSurvey3

-raster zijn vertrouwde `JPG / Targets / Reflectance`-set behoudt.
{% endhint %}

Als je op een rasterminiatuur klikt, wordt de [Afbeeldingsviewer](opening-an-image-full-screen.md) op volledig scherm geopend voor **hetzelfde product dat in het raster wordt weergegeven** — als het raster is ingesteld op `Targets`, opent de miniatuur de geëxporteerde doelafbeelding.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->


***

## Een LATTICE-project groeperen: per trigger versus per camera

Array-opnames leveren meerdere beelden op van hetzelfde moment, gemaakt door verschillende cameramodules. Groepering bepaalt hoe het raster deze afbeeldingen stapelt. Beide modi geven inklapbare kopbalken over de volledige breedte weer; **elke groep begint uitgevouwen**, enChloros
onthoudt welke je hebt ingeklapt. De inklapstatus wordt per modus afzonderlijk bijgehouden, dus het inklappen van een groep in ‘Per camera’ heeft geen invloed op de instelling ‘Per trigger’.

### Per Camera (standaard)

Eén groep per cameramodule. De koptekst toont het cameramodel en het serienummer (`LATT-M3M — <serial>`) en het aantal foto’s. De tegels binnen een groep worden chronologisch gerangschikt op basis van het opnamemoment.

In deze modus krijgt de werkbalk ook één **camerafilterknop per camera**, met het label `MODEL (SERIAL)`. Alle camera’s zijn aanvankelijk geselecteerd; als je op een knop klikt, wordt die camera gedeselecteerd en wordt de bijbehorende groep uit het raster verwijderd. Dit is de snelste manier om één band over een hele vlucht te bekijken.

### Per trigger

Eén groep per opnamegebeurtenis — de reeks frames die alle modules bij dezelfde trigger hebben vastgelegd. De koptekst toont het tijdstip van opname, het aantal camera’s dat hieraan heeft bijgedragen en een icoontje per cameramodel in de groep. Tegels binnen een groep zijn gesorteerd op serienummer van de camera, zodat dezelfde band voor elke trigger in dezelfde kolom staat.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Niet-LATTICE-beelden in een gemengd project worden niet gegroepeerd — ze worden weergegeven als gewone tegels na de groepen.

***

## Miniatuurafbeeldingen in het raster volgen de GSD-blokgrootte

Als je een **GSD (px)**-blokgrootte hebt ingesteld in de zijbalk van het tabblad ‘Afbeelding’, worden de rasterminiaturen in diezelfde grondresolutie weergegeven — niet alleen in de weergave op volledig scherm. Een blokgrootte van 8 betekent dat elke weergegeven pixel het gemiddelde is van een blok van 8 × 8 bronpixels, overal in de app waar de afbeelding wordt getoond.

Omdat een tegel om te beginnen slechts een paar honderd pixels breed is, maken grove blokgroottes al veel eerder geen zichtbaar verschil meer in het raster dan in de weergave op volledig scherm: een kader van 4000 px dat in een tegel van 160 px wordt getekend, komt al neer op ongeveer 25 bronpixels per weergegeven pixel. Zie [Een afbeelding op volledig scherm openen](opening-an-image-full-screen.md#gsd-block-size) voor het bedieningselement zelf.

***

## Gerelateerde pagina’s

* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) — de weergave op volledig scherm, cursorwaarden en histogram
* [**Afbeeldingslagen**](image-layers.md) — de lagen-vervolgkeuzelijst in de volledig-scherm-viewer
* [**Index/LUT-Sandbox**](index-lut-sandbox.md) — indexvisualisaties samenstellen en exporteren
* [**Projectinstellingen**](../project-settings/project-settings.md) — de exportschakelaars die bepalen welke producten überhaupt beschikbaar zijn
