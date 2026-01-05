# Index/LUT Sandbox

De Index/LUT Sandbox is een interactieve werkruimte binnen de Chloros Image Viewer waarmee u in realtime kunt experimenteren met multispectrale indexberekeningen en kleurvisualisaties. Met deze krachtige tool kunt u verschillende indices testen, waardebereiken verfijnen en publicatieklare visualisaties maken zonder uw volledige dataset opnieuw te verwerken.

## Wat is de Index/LUT Sandbox?

### Doel

De Sandbox biedt:

* **Realtime indexberekening** - Pas direct elke vegetatie-index toe
* **Interactieve LUT-aanpassing** - Verfijn kleurgradiënten en -bereiken
* **Workflowoptimalisatie** - Bepaal de beste instellingen vóór batchverwerking

### Sandbox versus projectverwerking

**Index/LUT Sandbox (interactief):**

* Eén afbeelding tegelijk
* Directe feedback
* Experimenteel en iteratief
* Geen permanente wijzigingen aan bestanden
* Perfect voor verkennen en testen

**Projectverwerking (batch):**

* Hele dataset in één keer
* Vooraf geconfigureerde instellingen
* Permanente uitvoerbestanden
* Tijdrovend
* Het beste wanneer de instellingen definitief zijn

{% hint style=&quot;success&quot; %}
**Beste workflow**: Gebruik de Sandbox om te experimenteren en de optimale index- en LUT-instellingen te vinden, en pas die instellingen vervolgens toe tijdens de projectverwerking voor uw volledige dataset.
{% endhint %}

***

## Werken met de Index/LUT Sandbox

### Vooraf berekende indexen begrijpen

In Chloros kunnen indexen worden toegepast tijdens projectverwerking. Om te bepalen welke index- en LUT-instellingen u wilt toepassen op exports, kunt u het beste de afbeeldingsviewer-sandbox gebruiken.

Met de sandbox kunt u:

* **Nieuwe index- en kleurgradiënten (LUT&#x27;s) toepassen** om de gegevens te visualiseren
* **Visualisatie-instellingen interactief aanpassen*** **Bekijken** van reeds berekende indexafbeeldingen
* **Inspecteren** van pixelwaarden op alle zoomniveaus

### De sandbox openen

De Index/LUT-sandbox is toegankelijk via het tabblad **Afbeeldingsviewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> zijbalk:

1. Klik op een afbeelding in het afbeeldingenraster van de bestandsbrowser. Deze wordt geopend in het tabblad **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Klik op **het tabblad Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> om de linker pop-out zijbalk te openen als deze nog niet open is

### Een afbeelding selecteren om een index/LUT op toe te passen

Om met een index in de afbeeldingsviewer te werken <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandbox:

1. **Open een afbeelding** uit het hoofdraster door erop te klikken
2. Het tabblad **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tabblad wordt dan geopend
3. Klik op de **Layer dropdown** (rechtsboven in de viewer)
4. Selecteer de laag uit de dropdown:
   * RAW (Reflectance)

### Een index toepassen op een afbeelding

Zodra de afbeelding op volledig scherm wordt weergegeven en de **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tabblad zijbalk is geopend:

1. Vink het vakje Index bovenaan de zijbalk aan
2. Kies het filter van uw camera uit de linker dropdown
3. Kies de gewenste indexformule uit de rechter dropdown
4. Sleep de kleurcirkels van het filterkanaal naar de locaties in de indexformule hieronder
5. Zodra de formule geldig is, wordt de afbeelding bijgewerkt en worden de indexwaarden weergegeven
6. Beweeg uw muiscursor om de waarden op de locatie van de cursor te zien
7. Zoom in om individuele pixels en de bijbehorende waarden te zien.

Elke index heeft een specifiek waardebereik en een specifieke betekenis:

#### NDVI Voorbeeld

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Zie [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md) voor volledige documentatie over indexformules.

***

## Werken met LUT&#x27;s (Look-Up Tables)

### Wat is een LUT?

Een **Look-Up Table (LUT)** koppelt numerieke indexwaarden aan kleuren voor visualisatie:

* **Invoer**: Indexpixelwaarde (bijv. NDVI 0,65)
* **Uitvoer**: RGB-kleur (bijv. felgroen)
* **Doel**: patronen gemakkelijker te zien en te interpreteren maken**Grijstinten versus kleuren-LUT:**

* Grijstinten: wetenschappelijk en neutraal, toont ruwe gegevens
* Kleuren-LUT: intuïtief en indrukwekkend, benadrukt patronen en verschillen

{% hint style=&quot;success&quot; %}
**Visualisatiekracht**: Door een kleuren-LUT toe te passen op een grijswaardenindexafbeelding worden patronen, afwijkingen en interessante gebieden in één oogopslag veel gemakkelijker te herkennen.
{% endhint %}

### Een LUT toepassen op een indexafbeelding

Zodra u een indexafbeelding hebt die

1. Klik op de <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> knop &quot;+LUT toevoegen&quot;
2. Selecteer het kleurovergang
3. Pas de minimale/maximale eindpunten van de clipping aan
4. Pas de clippingmodus aan
5. Vink het vakje Index aan in de **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> om de LUT toe te passen.

### Een kleurovergang kiezen

**Een overgang selecteren:**

1. Zoek in het LUT-paneel de**gekleurde verloopbalk**.
2. Beweeg uw muis eroverheen om de beschikbare verloopvoorinstellingen te bekijken.
3. Selecteer het gewenste verloop.
4. De afbeelding wordt **onmiddellijk bijgewerkt** met nieuwe kleuren wanneer het vakje Index is aangevinkt.

{% hint style=&quot;success&quot; %}
**Best practice**: voor vegetatie-indexen zoals NDVI is het verloop Red-Yellow-Green het meest intuïtief, omdat het overeenkomt met natuurlijke kleurassociaties (groen = gezond, geel = matig, rood = gestrest).
{% endhint %}

### Kleurklassen aanpassen

Met de **klassenregelaar**bepaalt u hoeveel afzonderlijke kleurmogelijkheden er in uw gradiënt verschijnen:**Opties voor het aantal klassen:*** **2-5 klassen**: zeer brede categorieën, duidelijke zones
* **6-10 klassen**: Evenwichtig, goed voor classificatie
* **11-20 klassen**: Vloeiende kleurverlopen, continu uiterlijk
* **20+ klassen**: Bijna continu, maximale vloeiendheid**Aanpassen:**

1. Zoek in het LUT-paneel de**kleurstalen onder de kleurverloopbalk**

2. Pas het aantal klassen aan door ze toe te voegen met de + knop
3. Verwijder het aantal klassen door te dubbelklikken op een kleurstaal
4. Het verloop wordt **in realtime** bijgewerkt op de afbeelding**Effect op visualisatie:*** **Minder klassen** (3-5): creëert duidelijke zones, vereenvoudigde classificatie, gemakkelijker te onderscheiden categorieën
* **Gemiddeld aantal klassen** (6-10): evenwichtige aanpak, geschikt voor de meeste toepassingen
* **Meer klassen** (15-20): vloeiende overgangen, gedetailleerde variatie, fotografisch uiterlijk**Wanneer te gebruiken:*** **Weinig klassen (3-5)**: presentatiedia&#x27;s, classificatiekaarten, eenvoudige rapporten
* **Gemiddelde klassen (6-10)**: algemene analyse, evenwichtige details, standaardrapporten
* **Veel klassen (15-20)**: wetenschappelijke analyse, gedetailleerde inspectie, output van publicatiekwaliteit

### Waardebereiken verfijnen

Met de **waardebereikregelaars**bepaalt u welke indexwaarden aan welke kleuren in uw kleurverloop worden toegewezen:**Bereikregelaars in het LUT-paneel:*** **Minimale waarde**: Ondergrens van de kleurenschaal
* **Maximale waarde**: Bovengrens van de kleurenschaal
* **Tussenliggende waarden**: Automatisch verdeeld tussen min en max (op basis van het aantal klassen)

#### Min/max-waarden aanpassen

**Om waardebereiken aan te passen:**

1. Zoek in het LUT-paneel de invoervelden**Min. waarde**en**Max. waarde**

2. Klik op het veld**Minimale waarde**

3. Typ de gewenste minimale waarde (bijv. `0.2`)
4. Druk op **Enter** of klik buiten het veld
5. Herhaal dit voor het veld **Maximale waarde** (bijv. `0.9`)
6. De visualisatie **wordt onmiddellijk bijgewerkt**{% hint style=&quot;info&quot; %}**Automatische schaalaanpassing**: Wanneer u voor het eerst een LUT toepast, stelt Chloros automatisch de min/max in op het werkelijke gegevensbereik in de afbeelding. U kunt dit bereik vervolgens verkleinen om u te concentreren op specifieke waardebereiken die voor u interessant zijn.
{% endhint %}

**Voorbeeld NDVI bereikaanpassingen:*** **Volledig bereik**: `-1.0` tot `1.0` (toon alle mogelijke waarden)
* **Gericht op vegetatie**: `0.2` tot `0.9` (kaal land en water uitsluiten)
* **Alleen gezonde vegetatie**: `0.5` tot `0.9` (alleen krachtige planten markeren)
* **Stressdetectie**: `0.2` tot `0.5` (benadruk probleemgebieden)
* **Aangepast bereik**: pas aan op basis van uw waargenomen pixelwaarden**Waarom bereiken aanpassen?*** **Verhoog het contrast** in uw interessegebied
* **Sluit irrelevante waarden uit** (bijv. waterlichamen, kale grond)
* **Standaardiseer de visualisatie** voor meerdere afbeeldingen of datums
* **Benadruk subtiele verschillen** binnen een smal waardebereik

### Waarden buiten het bereik knippen

Wanneer pixelwaarden buiten het door u gedefinieerde minimum-/maximumbereik vallen, kunt u met behulp van **knippermodi** bepalen hoe ze worden weergegeven.

#### **Beschikbare opties voor knippermodi:**

#### 1. Minimum en maximum

* Pixels **onder het minimum**→ weergeven met de**eerste kleur** in het kleurverloop (bijv. rood)
* Pixels **boven het maximum**→ weergeven met de**laatste kleur** in het verloop (bijv. groen)
* **Toepassing**: benadruk extremen, toon het volledige gegevensbereik met verzadigde kleuren bij de grenzen
* **Voorbeeld**: NDVI-waarden onder 0,2 worden allemaal rood weergegeven, waarden boven 0,9 worden allemaal groen weergegeven

#### 2. Transparante achtergrond

* Pixels **buiten het bereik**worden**volledig transparant*** Alleen pixels **binnen het bereik** tonen kleurovergang
* **Toepassing**: GIS-overlay, specifieke waardebereiken isoleren, alleen interessante gebieden markeren
* **Voorbeeld**: Alleen NDVI 0,4-0,7 in kleur weergeven, al het andere transparant

{% hint style=&quot;warning&quot; %}
**Beperking transparantie**: Transparante pixels worden in de viewer weergegeven als de achtergrondkleur. Bij export tijdens verwerking blijft de transparantie behouden in PNG-formaat, maar niet in JPG.
{% endhint %}

#### 3. Indexachtergrond

* Pixels **buiten het bereik**worden weergegeven in**grijstinten** (met de ruwe indexwaarden)
* Pixels **binnen het bereik**tonen**kleurverloop*** **Toepassing**: subtiele accentuering, behoud van context terwijl interessante gebieden worden benadrukt
* **Voorbeeld**: kleuraccentuering van gestresste vegetatie (NDVI 0,3-0,5) terwijl gezonde gebieden in grijs worden weergegeven

#### 4. Originele achtergrond

* Pixels **buiten het bereik**geven het**originele multispectrale beeld** weer
* Pixels **binnen het bereik**geven een**kleurverloop** weer
* **Toepassing**: meest intuïtief - combineert natuurlijke beeldcontext met analytische kleuroverlay
* **Voorbeeld**: bekijk het daadwerkelijke uiterlijk van het veld/gewas met kleurgecodeerde stressgebieden eroverheen

### De juiste knippermodus kiezen

| Knippermodus              | Meest geschikt voor                                   | Visualisatiestijl          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum en maximum**    | Volledige gegevensweergave, wetenschappelijke analyse     | Alle pixels gekleurd           |
| **Transparante achtergrond** | GIS-overlays, specifieke bereiken isoleren    | Kleur op bereik, leeg daarbuiten |
| **Indexachtergrond**       | Subtiele nadruk, gegevenscontext behouden  | Kleur op bereik, grijs daarbuiten  |
| **Originele achtergrond**    | Rapporten, presentaties, intuïtieve analyse | Kleur op bereik, foto daarbuiten |

### Aangepaste LUT-kleuren maken

Voor volledige controle over uw visualisatie kunt u **aangepaste kleurovergangen** maken door individuele kleurovergangen te bewerken.**Een aangepaste kleurovergang maken:**

1. Zoek in het LUT-paneel de**kleurovergang-voorbeeldbalk**

2. Zoek de**kleurstalenvierkantjes** onder het verloop
3. **Klik op een kleurovergang** om deze te selecteren
4. Er wordt een **kleurenkiezer** geopend
5. Kies een nieuwe kleur met behulp van:
   * **Kleurwiel**: visuele kleurselectie
   * **RGB/HSV-schuifregelaars**: nauwkeurige kleurregeling
   * **Hex-code invoeren**: exacte kleurspecificatie (bijv. `#FF0000` voor rood)
6. Klik buiten de kleurenkiezer **om de nieuwe kleur toe te passen**.
7. Het verloop **wordt onmiddellijk bijgewerkt** op de afbeelding.**Kleurstops toevoegen of verwijderen:*** **Een stop toevoegen**: Klik op het +-pictogram om een nieuw kleurstaal aan het einde toe te voegen
* **Een stop verwijderen**: Dubbelklik op het kleurvierkantje om het kleurstaal te verwijderen**Aanpassingsstrategieën:*** **Verloop omkeren**: Draai de kleurvolgorde om om de betekenis om te keren (bijv. groen = laag, rood = hoog)
* **Merk kleuren**: Stem het kleurenpalet van uw organisatie af op rapporten
* **Geschikt voor kleurenblinden**: gebruik combinaties van oranje-blauw of paars-geel
* **Optimalisatie voor afdrukken**: kies kleuren die zowel bij afdrukken in kleur als in grijstinten goed tot hun recht komen
* **Meerdere drempels**: gebruik verschillende kleuren bij specifieke waardedrempels voor classificatie

{% hint style=&quot;info&quot; %}
**Aangepaste kleurverlopen opslaan**: Aangepaste kleurverlopen kunnen worden opgeslagen en opnieuw worden gebruikt. Klik op het opslagpictogram in het LUT-paneel om uw aangepaste kleurenschema&#x27;s op te slaan voor toekomstig gebruik.
{% endhint %}

***

## Interactieve workflow

### Real-time updates

Alle LUT-aanpassingen in de sandbox werken het beeld **onmiddellijk en interactief** bij:

* **Laag wisselen** → Beeld verandert onmiddellijk
* **Verloop selecteren** → Kleuren worden onmiddellijk bijgewerkt
* **Waardebereik aanpassen** → Contrast verandert in realtime
* **Klassen wijzigen** → Vloeiendheid van het verloop wordt onmiddellijk bijgewerkt
* **Knippen wijzigen** → Achtergrondweergave verandert onmiddellijk
* **Kleuren bewerken** → Aangepast verloop wordt onmiddellijk toegepast**Geen &quot;Toepassen&quot;-knop nodig** - alle wijzigingen zijn live en interactief!

{% hint style=&quot;success&quot; %}
**Live feedback**: dankzij de directe visuele feedback kunt u snel experimenteren met verschillende instellingen totdat u de optimale visualisatie voor uw analysebehoeften hebt gevonden.
{% endhint %}

### Iteratieve verfijningsworkflow

**Typische LUT-optimalisatieworkflow:**

1.**Selecteer indexlaag** (bijv. RAW (reflectie))
2. **Index toepassen** - Kies camerafilter en indexformule, sleep gekleurde cirkels naar de juiste locatie in de indexformule
3. **LUT-gradiënt toepassen** - Begin met Red-Yellow-Green-preset
4. **Pixelwaarden inspecteren** - Beweeg de cursor, noteer waardebereiken
5. **Min/max aanpassen** - Beperk het bereik om u te concentreren op vegetatie (bijv. 0,2 tot 0,9)
6. **Knippen kiezen** - Probeer &quot;Originele achtergrond&quot; voor context
7. **Kleuren verfijnen** - Pas het verloop indien nodig aan voor specifieke nadruk
8. **Instellingen voltooien**- Documenteer instellingen en kopieer ze naar Projectinstellingen voor exportverwerking

### Pixelwaarde-inspectie

Inzicht in de werkelijke pixelwaarden is cruciaal voor het instellen van effectieve LUT-bereiken:**Hoe waarden inspecteren:**

1. Pixelwaarden worden weergegeven wanneer het afbeelding de Index of zowel de Index als LUT**vakjes aangevinkt** heeft.
2. **Beweeg uw cursor** over verschillende delen van de afbeelding
3. **Bekijk de pixelwaarden** die in de legenda worden weergegeven wanneer u met de muis over de afbeelding beweegt
4. Zoom in om individuele pixels te zien die zijn gemarkeerd met een zwevende waarde
5. **Noteer** de waardebereiken voor verschillende kenmerken:
   * **Gezonde vegetatie**: bijv. NDVI 0,55-0,85
   * **Gestresste vegetatie**: bijv. NDVI 0,30-0,50
   * **Kale grond**: bijv. NDVI 0,05-0,25
   * **Water** (indien aanwezig): bijv. NDVI -0,05 tot 0,10**Pixelwaarden gebruiken om LUT-bereiken in te stellen:**Pas na controle van de pixelwaarden uw LUT min/max dienovereenkomstig aan:**Voorbeeldscenario:*** **Waarneming**: Bodemwaarden = 0,05-0,25, Gestrest = 0,25-0,50, Gezond = 0,50-0,85
* **Doel**: Alleen de gezondheid van planten visualiseren (bodem uitsluiten)
* **LUT-instellingen**: Min = `0.25`, Max = `0.85`
* **Knippen**: &quot;Originele achtergrond&quot; om de bodem in natuurlijke kleuren te zien
* **Resultaat**: Kleurverloop is alleen van toepassing op vegetatie, bodem wordt weergegeven als originele afbeelding

{% hint style=&quot;info&quot; %}
**Dynamisch bereik**: Verschillende gewassen, seizoenen en groeifasen hebben verschillende waardebereiken. Controleer altijd de pixelwaarden in uw specifieke dataset voordat u LUT-bereiken instelt.
{% endhint %}

***

## Aangepaste indexen (Chloros+)

### Aangepaste indexformules maken

{% hint style=&quot;info&quot; %}
**Waar aanmaken**: Aangepaste indexen kunnen worden geconfigureerd in**Projectinstellingen** vóór de verwerking, evenals in de zijbalk van de Image Viewer-sandbox.
{% endhint %}

**Een aangepaste index maken:**

1.**Open Projectinstellingen** (vóór verwerking) of de zijbalk van de Image Viewer-sandbox.
2. Ga naar de **dropdown-lijst met indexformules**.
3. Zoek de optie **&quot;Aangepast&quot;** (u moet zijn ingelogd met een Chloros+-licentie).
4. **Definieer uw formule** met behulp van bandvariabelen:
   * Bandnamen: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, enz.
   * Operatoren: `+`, `-`, `*`, `/`, `^` (exponent)
   * Functies: `sqrt()`, `abs()`, enz. (indien ondersteund)
   * Haakjes: `()` voor de volgorde van bewerkingen
5. **Geef uw index een naam** (bijv. &quot;MyIndex&quot; of &quot;CustomNDVI&quot;)
6. **Sla de configuratie op**

**Voorbeelden van aangepaste formules:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Formulevalidatie**: zorg ervoor dat uw formule gebruikmaakt van banden die beschikbaar zijn in uw camera. RedEdge is bijvoorbeeld alleen beschikbaar op camera&#x27;s met een RedEdge-filter.
{% endhint %}

***

## Volgende stappen

Nu u de Index/LUT Sandbox begrijpt:

* **Toepassen op verwerking**: gebruik de ontdekte instellingen in [Projectinstellingen](../project-settings/project-settings.md)
* **Batchverwerking**: pas geoptimaliseerde indexen toe op volledige datasets
* **Meer informatie**: lees [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md)

Gerelateerde documentatie:

* [**Beeldlagen**](image-layers.md) - Laagbeheer en visualisatie
* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) - Basisprincipes van de afbeeldingsviewer
* [**Afbeeldingen verwerken (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Volledige verwerkingsworkflow
