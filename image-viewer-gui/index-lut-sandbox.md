# Index/LUT-sandbox

De Index/LUT-sandbox is de interactieve werkruimte in de zijbalk van de Chloros Image Viewer. Je kiest een formule, koppelt de kanalen van je camera eraan, geeft het een kleur met een kleurverloop en stelt het waardebereik in — en het beeld wordt live bijgewerkt terwijl je bezig bent. Sinds versie 1.2.0 kun je ook **wat je hebt gemaakt opslaan**, voor één afbeelding of voor het hele project, zonder het opnieuw te verwerken.

## Waarvoor dient de Sandbox?

| Index/LUT Sandbox (interactief)        | Projectverwerking (batch)       |
| -------------------------------------- | -------------------------------- |
| Eén afbeelding per keer, directe feedback  | De volledige dataset in één keer     |
| Experimenteel en iteratief             | Vooraf geconfigureerde instellingen          |
| Rendert live; slaat alleen op wanneer je dat vraagt  | Schrijft altijd productbestanden      |
| Perfect om de juiste instellingen te vinden | Het beste als de instellingen definitief zijn |

{% hint style="success" %}
**De gebruikelijke workflow**: pas de instellingen aan in de Sandbox totdat de visualisatie eruitziet zoals je wilt, en exporteer vervolgens rechtstreeks vanuit de Sandbox, of kopieer dezelfde index- en LUT-instellingen naar [Projectinstellingen](../project-settings/project-settings.md), zodat deze bij de volgende verwerkingsrun in elke afbeelding worden verwerkt.
{% endhint %}

***

## De Sandbox openen

1. Klik op een afbeelding in het raster — deze wordt op volledig scherm geopend in het tabblad **Afbeeldingsviewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Klik op het pictogram **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> om de linkerzijbalk uit te schuiven als deze nog niet geopend is
3. Kies een multibandlaag uit de laagkeuzelijst rechtsboven — **RAW (Reflectance)** is de gebruikelijke keuze, omdat indexwaarden die zijn berekend op basis van gekalibreerde reflectantie, tussen afbeeldingen onderling vergelijkbaar zijn

De zijbalk toont, van boven naar beneden:

* de afbeeldingsnaam en het cameramodel
* de knop **Afbeelding(en) exporteren/opslaan**— verschijnt zodra**Index**of**LUT** is aangevinkt
* de selectievakjes **Index**en**LUT**
* het configuratiepaneel voor de index
* het paneel **Cursorwaarden** met de uitlezing, het histogram en de GSD-regelaar

{% hint style="warning" %}
**Niet beschikbaar voor monochrome camera’s.** Bij een LATTICE M3M-beeld met één band zijn beide selectievakjes uitgeschakeld, met de tooltip _&quot;Niet beschikbaar voor monochrome (M3M) sensoren&quot;_ — een multiband-index is niet gedefinieerd op één band. Om indexen te berekenen op basis van M3M-camera&#x27;s, combineert u twee of meer beelden tot een uitgelijnde multiband-stack en gebruikt u de LATTICE-indexengine.
{% endhint %}

***

## Een index toepassen

1. Vink het vakje **Index** bovenaan de zijbalk aan
2. Kies het filter van je camera uit de vervolgkeuzelijst aan de linkerkant (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Kies een indexformule uit de rechter keuzelijst — 27 ingebouwde formules, plus eventuele aangepaste formules die je hebt opgeslagen
4. De formule wordt hieronder als wiskundige uitdrukking weergegeven, met een lege cirkel bij elke bandpositie. **Sleep een gekleurde kanaalcirkel naar een vak** om deze te koppelen
5. Zodra elk vak dat de formule gebruikt is gekoppeld, wordt de afbeelding bijgewerkt en worden de indexwaarden weergegeven
6. Beweeg de cursor over de afbeelding om waarden af te lezen; het paneel **Cursorwaarden** voegt een indexrij toe met de waarde onder de cursor

Dubbelklik op een gekoppeld vakje om het te wissen. Een onvolledige formule is een normale toestand tijdens het slepen, geen fout — de afbeelding wordt simpelweg niet bijgewerkt totdat de formule compleet is.

De kanaalcirkels zijn kleurgecodeerd: rood = Red, groen = Green, blauw = Blue, oranje = Orange, cyaan = Cyan, paars = NIR, magenta = RE. Dezelfde kleuren worden gebruikt voor de kanaalstippen en histogramcurves in het paneel ‘Cursorwaarden’.

### Voorbeeld van NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Zie [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md) voor de volledige formule-referentie — alle drie de voorinstellingslijsten en welke namen waar werken.

### Met ‘Index’ aangevinkt maar zonder LUT

De afbeelding wordt weergegeven in **grijstinten**, uitgerekt tussen de twee drempelwaarden. Dit is opzettelijk: de indexafbeelding bestaat uit scalaire gegevens, en grijstinten geven deze het meest getrouw weer. Voeg een LUT toe als je kleur wilt.***

## Werken met LUT’s (Look-Up Tables)

Een **Look-Up Table** koppelt indexwaarden aan kleuren: invoer NDVI 0,65, uitvoer een bepaalde groene kleur. Het verandert de gegevens niet — het verandert de manier waarop je ze interpreteert.

### Een LUT toevoegen

1. Klik op de <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> **&quot;+ LUT toevoegen&quot;**-knop onder de formule
2. Kies een kleurverloop
3. Stel het minimum en maximum voor clipping in
4. Kies een clippingmodus
5. Vink het vakje **LUT** in de zijbalk aan om deze weer te geven

Het LUT-selectievakje blijft uitgeschakeld totdat er daadwerkelijk een LUT is geconfigureerd in de index.

### Een kleurverloop kiezen

Beweeg de muis over de **verloopbalk**om de lijst met voorinstellingen te openen — Chloros bevat**zeven** voorinstellingen voor kleurverlopen:

| # | Verloop                            | Vorm                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Geel → Green (**standaard**)  | Divergerend — komt overeen met de gebruikelijke intuïtie over vegetatie: groen = gezond |
| 2 | Paars → Geel → Green             | Divergerend, met een duidelijk laagste punt                                  |
| 3 | Bruin → Wit → Blue                | Divergerend rond een licht middenpunt                                   |
| 4 | Zwart → Paars → Roze → Lichtgeel | Opeenvolgend, van donker naar licht                                           |
| 5 | Red → Geel → Blue                 | Afwijkend rond een licht middenpunt                                   |
| 6 | Paars → Blue → Green → Geel      | Opeenvolgend, van donker naar licht                                           |
| 7 | Orange → Wit → Paars             | Divergerend rond een licht middenpunt                                   |

Een **divergerend**kleurverloop plaatst een neutrale kleur in het midden van je venster, wat goed werkt wanneer het middelpunt een bepaalde betekenis heeft (een drempelwaarde, een basisdatum). Een**sequentieel** kleurverloop verloopt monotoon van donker naar licht, wat goed werkt voor een hoeveelheid die alleen ‘meer’ en ‘minder’ kent.

Elke voorinstelling heeft zeven kleurstops. Klik op een voorinstelling en de afbeelding wordt onmiddellijk bijgewerkt (als het vakje LUT is aangevinkt).

### De kleurstops bewerken

Onder de verloopbalk bevindt zich een rij kleurstalen, één per stop:

* **Een kleur wijzigen**: klik op een kleurstaal om de kleurenkiezer te openen (kleurenwiel, RGB/HSV-schuifregelaars of een hex-code zoals `#FF0000`)
* **Een stop toevoegen**: klik op de**+**-knop aan het einde van de rij — er wordt een witte stop toegevoegd
* **Een stop verwijderen**:**dubbelklik** op het staal
* **Een bewerkt verloop bewaren**: klik op het opslagpictogram naast de verloopbalk om je bewerkte verloop toe te voegen aan de lijst met voorinstellingen, zodat je het later opnieuw kunt selecteren

Het verloop dat je voor een index hebt geconfigureerd, wordt samen met die index opgeslagen in de projectinstellingen, zodat het behouden blijft wanneer je het project sluit en opnieuw opent.

**Minder stopplaatsen**zorgen voor duidelijke zones die als een classificatie worden geïnterpreteerd;**meer stopplaatsen** zorgen voor vloeiende, bijna fotografische overgangen. Drie tot vijf stopplaatsen zijn geschikt voor presentatiedia’s en classificatiekaarten; zes tot tien zijn geschikt voor algemene analyse; vijftien of meer zijn geschikt voor gedetailleerde inspectie- en publicatiefiguren.

### Het waardebereik instellen

De drempelregelaar is een **schuifregelaar met twee handvatten**die loopt van −1 tot +1, met aan elk uiteinde een bewerkbaar tekstvak voor exacte waarden en een**AUTO**-knop.

* Sleep een van de schuifregelaars, of typ een getal in het bijbehorende vakje en druk op Enter
* **AUTO**stelt het bereik in op het**2e en 98e percentiel** van de geldige indexwaarden van de afbeelding — een goed uitgangspunt dat uitschieters negeert. Chloros rondt het resultaat adaptief af, tot 4 decimalen voor een zeer smal bereik, 3 voor een smal bereik en 2 in alle andere gevallen
* Elke handmatige aanpassing heeft voorrang op AUTO totdat u opnieuw op AUTO drukt

Voorbeeld NDVI-vensters:

| Doel                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Alles weergeven                         | −1,0 | 1,0 |
| Alleen vegetatie, bodem en water uitsluiten | 0,2  | 0,9 |
| Alleen gezonde vegetatie                 | 0,5  | 0,9 |
| Stress benadrukken                        | 0,2  | 0,5 |

Door het venster te verkleinen, wordt het contrast binnen het gebied dat je wilt bekijken verhoogd en wordt al het andere buiten het bereik geduwd — waar de **Clipping Mode** bepaalt wat ermee gebeurt.***

## Clipping-modi

Wanneer de indexwaarde van een pixel buiten het min/max-venster valt, bepaalt de Clipping Mode hoe deze wordt weergegeven.

| Label in het dropdown-menu                  | Opgeslagen waarde      | Pixels buiten het bereik worden weergegeven als                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum &amp; Maximum** (standaard) | `clip`            | De dichtstbijzijnde eindkleur van het verloop — waarden onder het minimum krijgen de eerste kleur, waarden boven het maximum krijgen de laatste |
| **Transparante achtergrond**      | `transparent`     | Volledig transparant (echte alfa)                                                                                                  |
| **Indexachtergrond**| `indexColor`      | Grijswaarden, uitgerekt over het**volledige** indexbereik van de afbeelding, zodat structuren buiten dit bereik nog steeds in grijs zichtbaar zijn                |
| **Originele achtergrond**         | `backgroundColor` | De onderliggende afbeelding zelf, zodat de kleuroverlay bovenop de echte scène ligt                                                |

| Modus                       | Het meest geschikt voor                               | Uiterlijk                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum &amp; Maximum**      | Volledige gegevensweergave, wetenschappelijke analyse | Elke pixel gekleurd                      |
| **Transparante achtergrond** | GIS-overlays, een waardeband isoleren   | Kleur binnen het venster, niets daarbuiten |
| **Indexachtergrond**       | Nadruk met behoud van gegevenscontext    | Kleur binnen, grijs buiten               |
| **Originele achtergrond**    | Rapporten en presentaties              | Kleur binnen, foto buiten         |

{% hint style="info" %}
**Pixels zonder gegevens zijn altijd transparant, in elke modus.** Een pixel waarvan de index niet eindig is (een deling door 0) of precies −1,0 of +1,0 is (verzadigingssentinels, waarbij de ene band nul aangeeft terwijl de andere dat niet doet) wordt behandeld als ‘geen gegevens’ in plaats van als een extreme waarde. Hierdoor blijven overbelichte delen en donkere schaduwen buiten je kleurschaal, in plaats van dat ze worden weergegeven als de meest extreme waarde in het beeld. Dezelfde regel bepaalt welke pixels worden gebruikt voor de AUTO-drempels en het indexhistogram, zodat deze drie met elkaar overeenkomen.
{% endhint %}

De transparantie blijft behouden wanneer het bestand wordt geëxporteerd als PNG. Dit kan niet worden weergegeven in JPG.

***

## Waarden aflezen tijdens het afstemmen

Het paneel **Cursorwaarden** onder het configuratiepaneel is het meetinstrument voor de Sandbox:

* Beweeg de cursor over de afbeelding en lees de bronwaarden per kanaal af, plus de indexwaarde in de betreffende rij
* Schakel de knop **INDEX** boven het histogram in om de verdeling van de indexwaarden in het frame te zien, waarbij je twee clipdrempels als oranje stippellijnen worden weergegeven en de waarde van de cursor als een witte lijn — dit is de snelste manier om een venster te kiezen dat daadwerkelijk je gegevens bevat
* Schakel **CURSOR** in om markeringslijnen te zien bij de waarden onder de aanwijzer
* Zoom in tot meer dan 60× (minder als er een GSD-blokgrootte is ingesteld) om afzonderlijke weergegeven pixels met een zwevende waarde te markeren

Een praktische werkwijze:

1. Noteer de waarden boven gezonde vegetatie, gestresste vegetatie, kale grond en water
2. Kijk waar die clusters zich op het indexhistogram bevinden
3. Stel min/max in om het cluster dat voor jou van belang is te omkaderen
4. Kies een uitsnijdmodus — _Original Background_ houdt de scène eromheen zichtbaar

***

## Exporteren vanuit de Sandbox

Alles hierboven is een live voorbeeld totdat je het opslaat. De knop **Afbeelding(en) exporteren/opslaan** bovenaan de zijbalk opent een venster dat over de zijbalk schuift (in plaats van de afbeelding te bedekken, zodat je nog steeds kunt zien waarover je een beslissing neemt).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opties

| Optie                          | Effect                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Toepassen op huidige afbeelding**      | Slaat precies de getoonde afbeelding op, met deze instellingen                                                                                                |
| **Toepassen op alle projectafbeeldingen** | Voert dezelfde configuratie opnieuw uit op elke afbeelding in het project. Afbeeldingen zonder de banden die deze index nodig heeft, worden overgeslagen en niet als fouten behandeld |
| **Index/LUT-gradiëntbalk**      | Schrijft ook per export een afzonderlijke legenda-afbeelding, met het waardenbereik aangegeven                                                                     |
| **Indexhistogram**             | Schrijft ook per export een afzonderlijke histogramafbeelding, met de minimum- en maximumwaarden van de gegevens en de clippingdrempels                                               |

Als de **GSD-blokgrootte** op het tabblad ‘Afbeelding’ hoger is dan 1, wordt dit in het venster aangegeven voordat u de bewerking bevestigt: bij het exporteren wordt opgeslagen wat u ziet, inclusief blokgemiddelden. Stel de GSD-regelaar eerst terug op 1 als u de volledige resolutie wilt behouden.

### Waar de bestanden naartoe gaan

Elke klik op **Exporteren**wijst een**nieuwe, nooit eerder gebruikte map** toe:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Voorbeelden: `Sandbox_Exports/NDVI_LUT_001/`, en vervolgens `Sandbox_Exports/NDVI_LUT_002/` voor de volgende run. De nummering wordt bepaald door te scannen wat er al op de schijf staat, zodat deze behouden blijft bij herstarts en wanneer je mappen handmatig verwijdert. Er wordt nooit iets overschreven — het hele doel van de Sandbox is om de ene poging te vergelijken met de vorige.

In de map, per afbeelding:

| Bestand                                                   | Inhoud                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | De gerenderde afbeelding, pixel voor pixel zoals de viewer deze weergeeft |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Het bijbehorende bestand met de kleurverloopbalk, indien aangevraagd                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Het bijbehorende bestand met het indexhistogram, indien aangevraagd                  |

De twee bijlagen worden altijd in **volledige resolutie** opgeslagen, zelfs wanneer het hoofdbeeld blokgewijs is gemiddeld: een blokgrootte is gelijk aan de schermresolutie, en beide bijlagen geven de werkelijke indexwaarden per pixel weer. Ze geven ook meer informatie weer dan de versies op het scherm — beide vermelden zowel het stretch-venster _als_ de werkelijke minimum- en maximumwaarden van de gegevens, zodat een opgeslagen legenda maanden later nog steeds leesbaar is zonder dat het project geopend is.

### Voortgang en resultaten

Het exporteren van een heel project duurt enkele minuten, dus de uitvoer rapporteert via een live voortgangskanaal in plaats van het systeem te blokkeren:

* Een voortgangsbalk toont `current / total` en het bestand dat wordt geschreven
* Wanneer het proces is voltooid, geeft het venster weer hoeveel afbeeldingen zijn geëxporteerd, hoeveel zijn overgeslagen en het pad naar de uitvoermap
* Overgeslagen afbeeldingen worden vermeld met de reden (maximaal vijf worden weergegeven, daarna een regel „+N meer“). De gebruikelijke reden is een laag die niet over de kanalen beschikt die deze index nodig heeft
* Als **geen** enkele afbeelding in het project de index kan gebruiken, meldt de bewerking een fout in plaats van een lege map achter te laten

Er kan slechts één sandbox-export tegelijk worden uitgevoerd. Het starten van een tweede bewerking terwijl er al een bezig is, wordt geweigerd met een duidelijke melding, in plaats van dat twee bewerkingen om hetzelfde projectbestand gaan strijden.

### Het raster neemt de run over

Elke voltooide run verschijnt als een eigen knop in de [afbeeldingsraster](image-grid.md) werkbalk, met het label `<IndexName> <Index|LUT> <NNN>`. Zo vergelijk je uitvoeringen: exporteer twee keer met verschillende gradiënten of drempelwaarden en schakel vervolgens tussen de twee knoppen in het raster.

***

## Aangepaste indexformules (Chloros+)

{% hint style="info" %}
**Waar kun je ze aanmaken**: in de zijbalk van de Sandbox, of in**Projectinstellingen** vóór de verwerking. Beide schrijven naar dezelfde lijst op projectniveau.
{% endhint %}

1. Open de rekenmachine voor aangepaste formules via het dropdownmenu voor indexformules (hiervoor moet je zijn ingelogd met een geldig Chloros+-abonnement)
2. Schrijf de formule met behulp van de **band-slot-symbolen** `x`, `y`, `z`, `a`, `b`, `c` — dit zijn geen bandnamen
3. Beschikbare operators: `+`, `-`, `*`, `/`, `^` en `()` voor groepering
4. Beschikbare functies: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Geef het een naam en sla het op — het verschijnt onderaan de formule-keuzelijst en je koppelt de slots door kanaalcirkels te verslepen, precies zoals bij een ingebouwde preset

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Aangepaste formules zijn alleen beschikbaar via de GUI.** De optie CLI/SDK `--indices` breidt de 22 ingebouwde presetnamen uit en slaat al het andere stilzwijgend over, inclusief je aangepaste formules. Om een aangepaste formule in batches te verwerken, configureert u deze in de projectinstellingen en voert u de verwerking uit, of gebruikt u de exportoptie ‘Toepassen op alle projectafbeeldingen’ in de Sandbox.
{% endhint %}

***

## Probleemoplossing

### &quot;Deze laag heeft niet de kanalen die deze index nodig heeft&quot;

De formule leest een kanaalpositie die de huidige laag niet heeft — bijvoorbeeld een index met drie slots op een bestand met één of twee kanalen. Schakel over naar een multibandlaag (reflectantie of debayered), of kies een index die past bij het filter van je camera.

### &quot;Kon de backend voor beeldverwerking niet bereiken&quot;

De backend reageert niet. Controleer het tabblad Log; als de backend opnieuw wordt opgestart, herstelt de Sandbox zichzelf zodra deze weer beschikbaar is.

### Het beeld veranderde niet toen ik een cirkel versleepte

De formule is nog niet compleet. Een onvolledige formule wordt behandeld als een normale toestand tijdens het verslepen — er wordt niets weergegeven en er wordt geen fout gemeld. Vul elk veld in dat de formule gebruikt.

### De hele afbeelding heeft één kleur

Je clipvenster ligt waarschijnlijk ver buiten de gegevens. Druk op **AUTO**om het uit te lijnen op het 2e/98e percentiel, of schakel het**INDEX**-histogram in om te zien waar de gegevens zich daadwerkelijk bevinden.

### De geëxporteerde kleuren komen niet overeen met wat ik zag

Dat zou wel moeten — het exportpad is een bewuste weerspiegeling van het live-voorbeeld, inclusief de alfa in clipping-modus, en het blokgemiddelde wordt _na_ de inkleuring toegepast, precies zoals de viewer dat doet. Als ze verschillen, controleer dan of de GSD-blokgrootte niet is veranderd tussen het bekijken en het exporteren.

***

## Volgende stappen

* [**Beeldlagen**](image-layers.md) — op welke laag een index moet worden uitgevoerd, en wat de waarden betekenen
* [**Een afbeelding op volledig scherm openen**](opening-an-image-full-screen.md) — de cursorweergave, het histogram en de GSD-regeling in detail
* [**Formules voor multispectrale indexen**](../project-settings/multispectral-index-formulas.md) — alle voorinstellingen, op elk oppervlak
* [**Projectinstellingen**](../project-settings/project-settings.md) — de ingestelde parameters opslaan voor een verwerkingsrun
