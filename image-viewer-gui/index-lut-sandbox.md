# Index/LUT-Sandbox

De Index/LUT-Sandbox is een interactieve werkruimte binnen de Chloros Image Viewer waarmee u in realtime kunt experimenteren met multispectrale indexberekeningen en kleurvisualisaties. Met deze krachtige tool kunt u verschillende indices testen, waardebereiken verfijnen en publicatieklare visualisaties maken zonder uw volledige dataset opnieuw te verwerken.

## Wat is de Index/LUT Sandbox?

### Doel

De Sandbox biedt:

* **Realtime indexberekening** - Pas direct elke vegetatie-index toe
* **Interactieve LUT-aanpassing** - Verfijn kleurgradiënten en -bereiken
* **Workflowoptimalisatie** - Bepaal de beste instellingen vóór batchverwerking

### Sandbox versus projectverwerking

**Index/LUT Sandbox (interactief):**

* Eén afbeelding per keer
* Directe feedback
* Experimenteel en iteratief
* Geen permanente wijzigingen aan bestanden
* Perfect voor verkenning en testen

**Projectverwerking (batch):**

* Volledige dataset in één keer
* Vooraf geconfigureerde instellingen
* Permanente uitvoerbestanden
* Tijdrovend
* Het beste wanneer de instellingen definitief zijn

{% hint style="success" %}
**Beste workflow**: Gebruik de Sandbox om te experimenteren en de optimale index- en LUT-instellingen te vinden, en pas die instellingen vervolgens toe tijdens de projectverwerking voor uw volledige dataset.
{% endhint %}

***

## Werken met de Index/LUT-Sandbox

### Vooraf berekende indices begrijpen

In Chloros kunnen indices worden toegepast tijdens de projectverwerking. Om te bepalen welke index- en LUT-instellingen u wilt toepassen op exporten, kunt u het beste de afbeeldingsviewer-sandbox gebruiken.

Met de sandbox kunt u:

* **Nieuwe indexen en kleurverloop (LUT&#x27;s) toepassen** om de gegevens te visualiseren
* **Visualisatie-instellingen** interactief aanpassen
* **Bekijken** van reeds berekende indexafbeeldingen
* **Inspecteren** van pixelwaarden op alle zoomniveaus

### De Sandbox openen

De Index/LUT-Sandbox is toegankelijk via het tabblad in de **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Klik op een afbeelding in het afbeeldingsraster van de bestandsbrowser; deze wordt geopend in het tabblad **Image Viewer**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 2. Klik op**het tabblad Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> om de linker pop-out zijbalk te openen als deze nog niet open is

### Een afbeelding selecteren om een Index/LUT op toe te passen

Om met een index te werken in de Image Viewer <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Open een afbeelding** uit het hoofdraster door erop te klikken
2. Het tabblad **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tabblad wordt dan geopend
3. Klik op de **Layer dropdown** (rechtsboven in de viewer)
4. Selecteer de laag uit de dropdown:
   * RAW (Reflectance)

### Een index toepassen op een afbeelding

Zodra de afbeelding op volledig scherm staat en de **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> is geopend:

1. Vink het vakje Index bovenaan de zijbalk aan
2. Kies het filter van uw camera uit het linker dropdown-menu
3. Kies de gewenste indexformule uit het rechter dropdown-menu
4. Sleep de kleurcirkels van het filterkanaal naar de locaties in de indexformule hieronder
5. Zodra de formule geldig is, wordt de afbeelding bijgewerkt en worden de indexwaarden weergegeven
6. Beweeg uw muiscursor om de waarden op de locatie van de cursor te zien
7. Zoom in om individuele pixels en de bijbehorende waarden te zien

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

Een **Look-Up Table (LUT)** zet numerieke indexwaarden om in kleuren voor visualisatie:

* **Invoer**: Indexpixelwaarde (bijv. NDVI 0,65)
* **Uitvoer**: RGB kleur (bijv. felgroen)
* **Doel**: Patronen gemakkelijker zichtbaar en interpreteerbaar maken**Grijswaarden- versus kleuren-LUT:**

* Grijswaarden: Wetenschappelijk en neutraal, toont ruwe gegevens
* Kleuren-LUT: Intuïtief en indrukwekkend, benadrukt patronen en verschillen

{% hint style="success" %}
**Visualisatiekracht**: Door een kleuren-LUT toe te passen op een grijswaarden-indexafbeelding wordt het aanzienlijk eenvoudiger om patronen, afwijkingen en interessante gebieden in één oogopslag te herkennen.
{% endhint %}

### Een LUT toepassen op een indexafbeelding

Zodra u een indexafbeelding hebt die

1. Klik op de <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> &quot;+LUT toevoegen&quot;-knop
2. Selecteer het kleurverloop
3. Pas de minimale en maximale eindpunten van de clipping aan
4. Pas de clippingmodus aan
5. Vink het vakje Index aan in de zijbalk van de **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> om de LUT toe te passen

### Een kleurverloop kiezen

**Een verloop selecteren:**

1. Zoek in het LUT-paneel de**gekleurde verloopbalk**

2. Beweeg uw muis eroverheen om de beschikbare verloopvoorinstellingen te bekijken
3. Selecteer het gewenste verloop
4. De afbeelding **wordt onmiddellijk bijgewerkt** met nieuwe kleuren wanneer het vakje Index is aangevinkt

{% hint style="success" %}
**Aanbevolen werkwijze**: Voor vegetatie-indexen zoals NDVI is het verloop Red-Geel-Green het meest intuïtief, omdat het overeenkomt met natuurlijke kleurassociaties (groen = gezond, geel = matig, rood = gestrest).
{% endhint %}

### Kleurklassen aanpassen

De **Classes-regelaar**bepaalt hoeveel afzonderlijke kleurstappen er in uw kleurverloop verschijnen:**Opties voor het aantal klassen:*** **2-5 klassen**: Zeer brede categorieën, duidelijke zones
* **6-10 klassen**: Evenwichtig, goed voor classificatie
* **11-20 klassen**: Vloeiende kleurverlopen, continu uiterlijk
* **20+ klassen**: Bijna continu, maximale vloeiendheid**Aanpassen:**

1. Zoek in het LUT-paneel de**kleurstalen onder de verloopbalk**

2. Pas het aantal klassen aan door er met de +-knop klassen toe te voegen
3. Verwijder klassen door te dubbelklikken op een kleurstaal
4. Het verloop wordt **in realtime** bijgewerkt op de afbeelding**Effect op de visualisatie:*** **Minder klassen** (3-5): Creëert duidelijke zones, vereenvoudigde classificatie, gemakkelijker te onderscheiden categorieën
* **Gemiddeld aantal klassen** (6-10): Evenwichtige aanpak, geschikt voor de meeste toepassingen
* **Meer klassen** (15-20): Vloeiende overgangen, gedetailleerde variatie, fotografisch uiterlijk**Wanneer te gebruiken:*** **Weinig klassen (3-5)**: Presentatiedia&#x27;s, classificatiekaarten, eenvoudige rapporten
* **Gemiddeld aantal klassen (6-10)**: Algemene analyse, evenwichtige details, standaardrapporten
* **Veel klassen (15-20)**: Wetenschappelijke analyse, gedetailleerde inspectie, uitvoer van publicatiekwaliteit

### Waardebereiken verfijnen

De **waarderange-regelaars**bepalen welke indexwaarden worden toegewezen aan welke kleuren in uw kleurverloop:**Regelaars voor het bereik in het LUT-paneel:*** **Minimale waarde**: Ondergrens van de kleurschaal
* **Maximale waarde**: Bovengrens van de kleurschaal
* **Tussenliggende waarden**: Worden automatisch verdeeld tussen min en max (op basis van het aantal klassen)

#### Min./max. waarden aanpassen

**Om waardebereiken aan te passen:**

1. Zoek in het LUT-paneel de invoervelden**Min. waarde**en**Max. waarde**

2. Klik op het veld**Min. waarde**

3. Typ de gewenste minimumwaarde (bijv. `0.2`)
4. Druk op **Enter** of klik buiten het veld
5. Herhaal dit voor het veld **Max. waarde** (bijv. `0.9`)
6. De visualisatie **wordt onmiddellijk bijgewerkt**{% hint style="info" %}**Automatische schaalverdeling**: Wanneer u voor het eerst een LUT toepast, stelt Chloros automatisch de min/max in op het werkelijke gegevensbereik in de afbeelding. U kunt dit bereik vervolgens verkleinen om u te concentreren op specifieke waardebereiken die van belang zijn.
{% endhint %}

**Voorbeeld van bereikaanpassingen in NDVI:*** **Volledig bereik**: `-1.0` tot `1.0` (toon alle mogelijke waarden)
* **Gericht op vegetatie**: `0.2` tot `0.9` (sluit kale grond en water uit)
* **Alleen gezonde vegetatie**: `0.5` tot `0.9` (markeer alleen krachtige planten)
* **Stressdetectie**: `0.2` tot `0.5` (benadruk probleemgebieden)
* **Aangepast bereik**: Pas aan op basis van uw waargenomen pixelwaarden**Waarom bereiken aanpassen?*** **Verhoog het contrast** in uw interessegebied
* **Sluit irrelevante waarden uit** (bijv. watermassa&#x27;s, kale grond)
* **Standaardiseer de visualisatie** over meerdere afbeeldingen of datums
* **Benadruk subtiele verschillen** binnen een smal waardebereik

### Waarden buiten het bereik uitsnijden

Wanneer pixelwaarden buiten het door u gedefinieerde min/max-bereik vallen, kunt u met behulp van **uitsnijdmodi** bepalen hoe deze worden weergegeven.

#### **Beschikbare opties voor clippingmodi:**

#### 1. Minimum en Maximum

* Pixels **onder het minimum**→ weergeven met de**eerste kleur** in het kleurverloop (bijv. rood)
* Pixels **boven het maximum**→ weergeven met de**laatste kleur** in het kleurverloop (bijv. groen)
* **Toepassing**: benadruk uitersten, toon het volledige gegevensbereik met verzadigde kleuren aan de grenzen
* **Voorbeeld**: NDVI-waarden onder 0,2 worden allemaal rood weergegeven, waarden boven 0,9 worden allemaal groen weergegeven

#### 2. Transparante achtergrond

* Pixels **buiten het bereik**worden**volledig transparant*** Alleen pixels **binnen het bereik** vertonen een kleurverloop
* **Toepassing**: GIS-overlay, specifieke waardebereiken isoleren, alleen gebieden van belang markeren
* **Voorbeeld**: Toon alleen NDVI 0,4-0,7 in kleur, al het andere transparant

{% hint style="warning" %}
**Beperking van transparantie**: Transparante pixels worden in de viewer weergegeven als de achtergrondkleur. Bij export tijdens verwerking blijft de transparantie behouden in het PNG-formaat, maar niet in JPG.
{% endhint %}

#### 3. Indexachtergrond

* Pixels **buiten het bereik**worden weergegeven in**grijstinten** (met ruwe indexwaarden)
* Pixels **binnen het bereik**tonen een**kleurenverloop*** **Toepassing**: Subtiele markering, behoud van context terwijl aandachtsgebieden worden benadrukt
* **Voorbeeld**: Markeer gestresste vegetatie met kleur (NDVI 0,3-0,5) terwijl gezonde gebieden in grijs worden weergegeven

#### 4. Originele achtergrond

* Pixels **buiten het bereik**geven het**originele multispectrale beeld** weer
* Pixels **binnen het bereik**tonen een**kleurenverloop*** **Toepassing**: Meest intuïtief - combineert de natuurlijke beeldcontext met een analytische kleuroverlay
* **Voorbeeld**: Bekijk het daadwerkelijke uiterlijk van het veld/gewas met daaroverheen kleurgecodeerde stressgebieden

### De juiste clippingmodus kiezen

| Clippingmodus              | Meest geschikt voor                                   | Visualisatiestijl          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum en maximum**    | Volledige gegevensweergave, wetenschappelijke analyse     | Alle pixels gekleurd           |
| **Transparante achtergrond** | GIS-overlays, specifieke bereiken isoleren    | Kleur binnen bereik, leeg daarbuiten |
| **Indexachtergrond**       | Subtiele nadruk, behoud van gegevenscontext  | Kleur op bereik, grijs daarbuiten  |
| **Originele achtergrond**    | Rapporten, presentaties, intuïtieve analyse | Kleur op bereik, foto daarbuiten |

### Aangepaste LUT-kleuren maken

Voor volledige controle over uw visualisatie kunt u **aangepaste kleurverlopen** maken door individuele kleurstops te bewerken.**Een aangepast verloop maken:**

1. Zoek in het LUT-paneel de**verloopvoorbeeldbalk**

2. Zoek de**kleurstaalvierkanten** onder het verloop
3. **Klik op een kleurstop** om deze te selecteren
4. Er wordt een **kleurenkiezer** geopend
5. Kies een nieuwe kleur met behulp van:
   * **Kleurenwiel**: Visuele kleurselectie
   * **RGB/HSV-schuifregelaars**: Nauwkeurige kleurcontrole
   * **Hex-code invoeren**: Exacte kleurspecificatie (bijv. `#FF0000` voor rood)
6. Klik buiten de kleurenkiezer **om de nieuwe kleur toe te passen**

7. Het verloop**wordt onmiddellijk bijgewerkt** op de afbeelding**Kleurstops toevoegen of verwijderen:*** **Een stop toevoegen**: Klik op het +-pictogram om een nieuw kleurstaal aan het einde toe te voegen
* **Een stop verwijderen**: Dubbelklik op het kleurvakje om het kleurstaal te verwijderen**Aanpassingsstrategieën:*** **Verloop omkeren**: Draai de kleurvolgorde om om de betekenis om te keren (bijv. groen = laag, rood = hoog)
* **Merkkleuren**: Stem het kleurenpalet van uw organisatie af op rapporten
* **Geschikt voor kleurenblinden**: Gebruik combinaties van oranje-blauw of paars-geel
* **Afdrukoptimalisatie**: Kies kleuren die zowel bij afdrukken in kleur als in grijstinten goed werken
* **Meerdere drempels**: Gebruik verschillende kleuren bij specifieke drempelwaarden voor classificatie

{% hint style="info" %}
**Aangepaste kleurverlopen opslaan**: Aangepaste kleurverlopen kunnen worden opgeslagen en hergebruikt. Klik op het opslagpictogram in het LUT-paneel om uw aangepaste kleurenschema&#x27;s te bewaren voor toekomstig gebruik.
{% endhint %}

***

## Interactieve workflow

### Real-time updates

Alle LUT-aanpassingen in de sandbox werken **onmiddellijk en interactief** door in de afbeelding:

* **Van laag wisselen** → Afbeelding verandert onmiddellijk
* **Verloop selecteren** → Kleuren worden direct bijgewerkt
* **Waardebereik aanpassen** → Contrast verandert in realtime
* **Klassen wijzigen** → De vloeiendheid van het verloop wordt onmiddellijk bijgewerkt
* **Clipping aanpassen** → Weergave van de achtergrond verandert onmiddellijk
* **Kleuren bewerken** → Aangepast verloop wordt onmiddellijk toegepast**Geen &quot;Toepassen&quot;-knop nodig** - alle wijzigingen zijn live en interactief!

{% hint style="success" %}
**Live feedback**: Dankzij de onmiddellijke visuele feedback kunt u snel experimenteren met verschillende instellingen totdat u de optimale visualisatie voor uw analysebehoeften hebt gevonden.
{% endhint %}

### Iteratieve verfijningsworkflow

**Typische LUT-optimalisatieworkflow:**

1.**Selecteer indexlaag** (bijv. RAW (Reflectantie))
2. **Pas index toe** - Kies camerafilter en indexformule, sleep gekleurde cirkels naar de juiste locatie in de indexformule
3. **Pas LUT-verloop toe** - Begin met de voorinstelling Red-Yellow-Green
4. **Controleer pixelwaarden** - Beweeg de cursor rond, let op de waardebereiken
5. **Pas min/max aan** - Beperk het bereik om je te concentreren op vegetatie (bijv. 0,2 tot 0,9)
6. **Kies clipping** - Probeer &quot;Original Background&quot; voor context
7. **Verfijn kleuren** - Pas het verloop indien nodig aan voor specifieke nadruk
8. **Rond instellingen af**- Documenteer instellingen en kopieer naar Projectinstellingen voor exportverwerking

### Controle van pixelwaarden

Inzicht in de werkelijke pixelwaarden is cruciaal voor het instellen van effectieve LUT-bereiken:**Hoe waarden te controleren:**

1. Pixelwaarden worden weergegeven wanneer voor de afbeelding het vakje Index of zowel Index als LUT**is aangevinkt**.
2. **Beweeg uw cursor** over verschillende delen van de afbeelding
3. **Bekijk de pixelwaarden** die in de legenda worden weergegeven terwijl u de muisaanwijzer erover beweegt
4. Zoom in om individuele pixels te zien die zijn gemarkeerd met een zwevende waarde
5. **Noteer** de waardebereiken voor verschillende kenmerken:
   * **Gezonde vegetatie**: bijv. NDVI 0,55-0,85
   * **Gestresste vegetatie**: bijv. NDVI 0,30-0,50
   * **Kale grond**: bijv. NDVI 0,05-0,25
   * **Water** (indien aanwezig): bijv. NDVI -0,05 tot 0,10**Pixelwaarden gebruiken om LUT-bereiken in te stellen:**Pas na inspectie van de pixelwaarden uw LUT-min/max dienovereenkomstig aan:**Voorbeeldscenario:*** **Waarneming**: Bodemwaarden = 0,05-0,25, Gestresst = 0,25-0,50, Gezond = 0,50-0,85
* **Doel**: Alleen de gezondheid van de planten visualiseren (bodem uitsluiten)
* **LUT-instellingen**: Min = `0.25`, Max = `0.85`
* **Clipping**: &quot;Original Background&quot; om de bodem in natuurlijke kleur te zien
* **Resultaat**: Kleurverloop is alleen van toepassing op vegetatie, bodem wordt weergegeven als origineel beeld

{% hint style="info" %}
**Dynamisch bereik**: Verschillende gewassen, seizoenen en groeifasen hebben verschillende waardebereiken. Controleer altijd de pixelwaarden in uw specifieke dataset voordat u LUT-bereiken instelt.
{% endhint %}

***

## Aangepaste indices (Chloros+)

### Aangepaste indexformules maken

{% hint style="info" %}
**Waar aan te maken**: Aangepaste indices kunnen worden geconfigureerd in**Projectinstellingen** vóór de verwerking, evenals in de zijbalk van de Image Viewer-sandbox.
{% endhint %}

**Een aangepaste index maken:**

1.**Open Projectinstellingen** (vóór de verwerking) of de zijbalk van de Image Viewer-sandbox
2. Ga naar de **vervolgkeuzelijst met indexformules**

3. Zoek naar de optie**&quot;Aangepast&quot;** (u moet zijn ingelogd met een Chloros+-licentie)
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

{% hint style="warning" %}
**Formulevalidatie**: Zorg ervoor dat uw formule gebruikmaakt van banden die beschikbaar zijn in uw camera. RedEdge is bijvoorbeeld alleen beschikbaar op camera&#x27;s met een RedEdge-filter.
{% endhint %}

***

## Volgende stappen

Nu u de Index/LUT Sandbox begrijpt:

* **Toepassen op verwerking**: Gebruik de gevonden instellingen in [Projectinstellingen](../project-settings/project-settings.md)
* **Batchverwerking**: Pas geoptimaliseerde indexen toe op volledige datasets
* **Meer informatie**: Lees [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md)

Gerelateerde documentatie:

* [**Beeldlagen**](image-layers.md) - Laagbeheer en visualisatie
* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) - Basisprincipes van de afbeeldingsviewer
* [**Afbeeldingen verwerken (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Volledige verwerkingsworkflow
