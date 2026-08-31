# Een afbeelding op volledig scherm openen

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Een afbeelding geopend op volledig scherm, met de laagselector rechtsboven</p></figcaption></figure>

De Chloros Image Viewer is de interface op volledig scherm voor het bekijken, inspecteren en meten van uw afbeeldingen. Hier kunt u **echte pixelwaarden** aflezen — DN per kanaal, reflectiepercentage of stralingsintensiteit in W/m²/sr/nm — in plaats van het uitgerekte voorbeeld dat op het scherm wordt weergegeven.

## De afbeeldingsviewer openen

### Vanuit de bestandsbrowser

1. Open het tabblad **Bestandsbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik op een willekeurige **miniatuur** in het [afbeeldingsraster](image-grid.md)
3. De afbeelding wordt op volledig scherm geopend in het tabblad **Afbeeldingsviewer**

De afbeelding wordt geopend in het product dat in het raster werd weergegeven. Als het raster is ingesteld op `RAW (Reflectance)`, is dat de laag waarin je terechtkomt.

### De zijbalk van de afbeeldingsviewer openen

Klik op het **Image Viewer**-<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-pictogram in de linkerzijbalk om het analysepaneel uit te schuiven. Dit bevat, van boven naar beneden:

* de afbeeldingsnaam en het cameramodel
* de knop **Afbeelding(en) exporteren/opslaan** (alleen als er een index of LUT actief is)
* de selectievakjes **Index**en**LUT** en het configuratiepaneel voor de index — zie [Index/LUT Sandbox](index-lut-sandbox.md)
* het paneel **Cursorwaarden**: uitlezing per kanaal, laaghistogram en de GSD-regelaar***

## Navigeren en zoomen

### Door afbeeldingen bladeren

* **Volgende afbeelding**: de →-knop of de**→**-toets (pijl naar rechts)
* **Vorige afbeelding**: de ←-knop of de**←**-toets (pijl naar links)
* **Naar een specifieke afbeelding springen**: ga terug naar het raster en klik op de bijbehorende miniatuur

De zoom- en verschuifstand blijven behouden terwijl u tussen afbeeldingen schakelt, zodat u door een reeks kunt bladeren terwijl u op hetzelfde deel van het beeld blijft.

### Zoomen

Zoomen gebeurt met het **muiswiel**, in stappen van 15%, waarbij de cursor als referentiepunt dient — het punt onder de aanwijzer blijft onder de aanwijzer. Het bereik wordt bepaald door de afbeelding en de venstergrootte: je kunt niet verder uitzoomen dan ‘aanpassen aan venster’, en de bovengrens wordt bepaald door de oorspronkelijke resolutie van de afbeelding.

Er zijn geen speciale zoomboetoetsen in de volledig-scherm-viewer. (In het raster: **Ctrl + `+` / `−`** past de grootte van de miniaturen aan — een andere bedieningsfunctie.)

### Pannen bij ingezoomd beeld

Klik met de linkermuisknop op de afbeelding, houd deze ingedrukt en sleep. Het pannen is beperkt, zodat de afbeelding niet buiten het scherm kan worden gesleept.

### Inspectie per pixel bij hoge zoomfactor

Zodra de effectieve vergroting **60×** overschrijdt, tekent Chloros een gemarkeerd kader rond de afzonderlijke weergegeven pixel onder de cursor en een zwevende waarde ernaast.

De „effectieve“ vergroting houdt rekening met de GSD-blokgrootte: bij een blokgrootte van 8 verschijnt de markering al bij een vergroting van 7,5× in plaats van 60×, omdat één weergegeven pixel al 8 × 8 bronpixels beslaat. Als je weer uitzoomt tot onder de drempelwaarde, verdwijnt de markering.

### Sneltoetsen

| Toets                             | Waar       | Actie                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Volledig scherm | Volgende afbeelding                          |
| **←**                           | Volledig scherm | Vorige afbeelding                      |
| **Ctrl + R**                    | Volledig scherm | De index/LUT-sandbox resetten         |
| **Ctrl + `+`**/**Ctrl + `=`** | Raster        | Grotere miniaturen (4 px per druk op de toets)  |
| **Ctrl + `−`**                  | Raster        | Kleinere miniaturen (4 px per druk op de toets) |***

## Cursorwaarden

Beweeg de cursor over de afbeelding en het paneel **Cursorwaarden** geeft de waarde weer van elk kanaal eronder.

{% hint style="success" %}
**Dit zijn de werkelijke getallen van het bestand.** Het canvas op het scherm is een uitgerekt 8-bits voorbeeld en kan deze waarden niet weergeven; daarom neemt Chloros een steekproef uit het daadwerkelijke productbestand voor de weergave. Dat is de reden waarom een 12-bits raw-frame waarden boven 255 weergeeft en waarom een float32-radiance-laag fysieke eenheden weergeeft.
{% endhint %}

### Wat de kolommen betekenen

Het paneel past zich aan de laag aan die u bekijkt:

| Laag die u bekijkt              | Weergegeven kolommen    | Opmerkingen                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Reflectie                        | **DN**en**%** | Het percentage wordt berekend op basis van de eigen schaal van dat bestand — zie hieronder                                      |
| Straling                           | **W/m²/sr/nm**   | Fysieke waarden als drijvende-kommagetallen; geen DN-kolom, omdat een DN hier geen betekenis heeft                           |
| Ruw / Debayered / voorbeeld / JPG    | **DN**           | Digitale gehele getallen                                                                         |
| 32-bits export van reflectiepercentages | alleen **%**       | De opgeslagen float is geen DN, dus als deze naar een geheel getal zou worden afgerond, zou er een zinloze waarde zoals `0` of `1` worden weergegeven |

Elke rij is gelabeld met de kanaalnaam van het filter van je camera — `Red / Green / NIR` voor RGN, `Orange / Cyan / NIR` voor OCN, `NIR / Green / Blue` voor NGB, `Red / Green / Blue` voor RGB, en de naam van de enkele band voor RE-, NIR- en mono-M3M-camera’s. Elk label is voorzien van een gekleurde stip die overeenkomt met de kanaalcirkels die in de indexformule-editor worden gebruikt.

Opgeslagen **index- en LUT**-afbeeldingen vormen een speciaal geval: ze bevatten kleurkaartcomponenten in plaats van spectrale banden, dus hun rijen zijn gelabeld als `Red / Green / Blue` (of `Index` voor een indexbestand met één kanaal) in plaats van met de filternamen van de camera.

Wanneer een index actief is in de sandbox, verschijnt er onder de kanalen een extra rij met de **indexwaarde** op de cursor, samen met de naam van de index en een witte stip die overeenkomt met de markering op het histogram.

### Het reflectentiepercentage maakt gebruik van de eigen schaal van elk bestand

{% hint style="warning" %}
**Ga er niet vanuit dat 65535 = 100% is.** Chloros slaat reflectie op verschillende schalen op, afhankelijk van welke camera het heeft geproduceerd, en de viewer bepaalt per bestand de juiste schaal.
{% endhint %}

| Bron                  | DN die overeenkomt met reflectie 1,0 | Hoe het wordt geïdentificeerd                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP-tag `Chloros:PixelScale=32768` wordt in elke LATTICE-reflectantie-export geschreven. Dankzij de 2× headroom kan het bestand ρ boven 1,0 bevatten zonder clipping |
| **Survey3**|**65535**                      | Geen Chloros XMP-schaal-tag — de Survey3-kalibratie schrijft ρ × dtype-max en clipt bij 1,0                                                               |

De viewer, de index/LUT-sandbox en de indexexport berekenen de schaal allemaal via dezelfde implementatie, dus een waarde die je bij de cursor afleest, is dezelfde waarde die de indexberekening heeft gebruikt.

Twee gevolgen die het vermelden waard zijn:

* Een **32-bits procent**TIFF slaat DN/65535 op als een float, en een**8-bits** PNG/JPG-export slaat DN × 255/65535 op — de viewer zet beide weer terug voordat er een percentage wordt weergegeven.
* Eén geval kan niet worden hersteld: een **8-bits TIFF-export van een opname met een 8-bits bron** wordt begrensd tot 0–255 in plaats van opnieuw geschaald, en bevat opzettelijk geen schaaltag. Voor die bestanden geeft het paneel alleen de DN weer, zonder procentkolom. Dit is het eerlijke antwoord, geen bug.***

## Het laaghistogram

Onder de cursorrijen bevindt zich een live histogram van de laag die u bekijkt, in **256 bins**. Standaard wordt één gecombineerde curve getekend, gewogen volgens `(R + 2G + B) / 4` — dezelfde meetruimte die de LATTICE-camera-histogrammen gebruiken. Als u**RGB** in, wordt deze vervangen door krommen per kanaal in de kanaalkleuren, die additief worden gemengd zodat overlappingen leesbaar blijven. Mono-lagen geven altijd de enkele kromme weer.

De horizontale as is in de eigen eenheid van de laag:

| Laag       | As-eenheid  | Maximum as                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Reflectantie | procent    | 125% — de headroom van het product staat ρ boven 1,0 toe           |
| Stralingskracht    | W/m²/sr/nm | De eigen piek van het frame, afgerond naar twee significante cijfers |
| 8-bits gegevens  | DN         | 255                                                        |
| 12-bits gegevens | DN         | 4095                                                       |
| 16-bits gegevens | DN         | 65535                                                      |

Wanneer de as in DN staat en op een van deze drie bovengrenzen uitkomt, weet Chloros ook de bitdiepte van wat u bekijkt.

Boven het histogram bevinden zich drie knoppen:

| Knop     | Standaard | Effect                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | Aan      | Tekent markeringslijnen op het histogram op de exacte waarden die in de bovenstaande rijen worden weergegeven, zodat je kunt zien waar de pixel onder je cursor zich bevindt in de verdeling van het beeld. In de modus RGB is er één markering per kanaal in een eigen kleur; anders is er één witte markering bij de gecombineerde waarde |
| **INDEX**| Aan      | Wordt alleen weergegeven terwijl een index actief is. Schakelt het histogram om van de bronbanden naar de**indexwaardeverdeling**, waarbij de twee clippedrempels worden weergegeven als oranje stippellijnen en de indexwaarde van de cursor als een witte lijn                                                          |
| **RGB**| Uit     | Schakelt over van de gecombineerde curve naar curven per kanaal. Op een monosensor geeft deze knop**MONO** weer en is hij uitgeschakeld — er is slechts één kanaal om weer te geven                                                                                                                                  |

Het histogram wordt berekend op basis van de **blokken die je ziet**, niet op basis van de bronpixels erachter: als je de GSD-blokgrootte wijzigt, wordt de verdeling opnieuw berekend, zodat het histogram, de cursormarkering en het weergegeven beeld altijd met elkaar overeenkomen.***

## GSD-blokgrootte

Onderaan het paneel bevindt zich de **GSD (px)**-regelaar: een invoerveld, een schuifbalk van**1 tot 256**en een**RESET**-knop.

Hiermee wordt de _weergegeven_ afbeelding grover gemaakt door een N × N-blok bronpixels te middelen tot één weergegeven pixel. `1` is de oorspronkelijke resolutie.

* Dit heeft invloed op **de weergave op volledig scherm, de rasterminiaturen, de cursorweergave en beide histogrammen** — alles wat de afbeelding weergeeft, gebruikt dezelfde basisresolutie.
* Dit geldt **alleen voor de weergave**. De verwerking en export blijven ongewijzigd. Er is één bewuste uitzondering: een export via de [Index/LUT Sandbox](index-lut-sandbox.md) slaat op wat je op dat moment ziet, dus deze behoudt de huidige blokgrootte, en het exportpaneel waarschuwt je wanneer de blokgrootte groter is dan 1.
* De waarde wordt **per project** opgeslagen als `viewer_display.gsd_bin` in `project.json`, zodat deze behouden blijft bij het sluiten en opnieuw openen.
* De cursorweergave geeft de blokwaarde weer, niet de bronpixel, wanneer de blokgrootte groter is dan 1 — de weergegeven waarde is het gemiddelde van het blok onder je cursor.

{% hint style="info" %}
**Waarom „blokgrootte“ en niet centimeters per pixel?** Voor een waarde in cm/px is een hoogte boven de grond nodig. De EXIF-gegevens van een enkel beeld bevatten de GPS-hoogte boven gemiddeld zeeniveau, niet boven het terrein waarop de camera was gericht; daarom geeft Chloros geen afstand tot de grond weer die het niet op betrouwbare wijze kan afleiden. De blokgrootte in bronpixels is dezelfde uitwijkoplossing die de MAPIR-cloudtools gebruiken wanneer de grondmonstersafstand onbekend is.
{% endhint %}

***

## Beeldtypen die je kunt bekijken

Het laagmenu rechtsboven in de viewer toont alle versies van het huidige beeld. Welke items worden weergegeven, hangt af van de camera en van wat er is verwerkt — zie [Beeldlagen](image-layers.md) voor de volledige lijst en hoe het keuzemenu werkt.

### Survey3

* **JPG** — het eigen voorbeeldbestand van de camera
* **RAW (Origineel)** — het bronbestand `.RAW`, gedebayerd voor weergave, zonder correcties
* **RAW (Doel)** — een frame waarvan is vastgesteld dat het een kalibratiedoel bevat
* **RAW (Reflectantie)** — het gekalibreerde reflectantieproduct (65535 = ρ 1,0)
* **Vignettering gecorrigeerd**/**Sensorrespons** — het niet-gekalibreerde fallback-product
* **Witgebalanceerd** — het witgebalanceerde product
* **RAW (`<INDEX>`-index)**en**`<INDEX>` LUT** — berekende indexafbeeldingen

### LATTICE

LATTICE-opnames maken gebruik van hetzelfde dropdown-menu, met de namen van de niveaus in de pijplijn:

| Laag                 | Wat deze bevat                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Origineel)**    | Het bron-RAW-frame zoals vastgelegd                                     |
| **RAW (Debayered)**   | Het lineaire, debayered beeld                                           |
| **RAW (Voorbeeld)**     | Het weergavevoorbeeld — valsekleurenuitrekking voor multispectrale camera’s |
| **Witbalans**    | Het weergavevoorbeeld voor RGB-mastercamera’s (witbalans + gamma)   |
| **RAW (Straling)**    | Float32 spectrale straling in W/m²/sr/nm                              |
| **RAW (Reflectantie)** | uint16 reflectantie, 32768 = ρ 1,0                                    |

Straling en reflectantie zijn uitsluitend multispectraal: een RGB-mastercamera heeft geen radiometrie per band, dus die lagen worden er niet voor gegenereerd.

***

## Index en LUT-toepassing

Pas multispectrale indices en kleuren-Look-Up Tables toe vanuit de zijbalk:

1. Open de **Image Viewer**-<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-zijbalk
2. Vink **Index** aan
3. Kies het filter van je camera en een indexformule, en sleep vervolgens de kanaalcirkels naar de vakjes van de formule
4. Voeg een LUT toe en kies een verloop, drempelwaarden en een clippingmodus
5. Bekijk de waarden bij de cursor en sla het resultaat op met **Export/Save Image(s)**Zie [Index/LUT Sandbox](index-lut-sandbox.md) voor de volledige handleiding.***

## Probleemoplossing

### De afbeelding kan niet worden geopend

**Mogelijke oorzaken**: het bestand is na het importeren verplaatst of verwijderd; het product is nooit opgeslagen; onvoldoende geheugen voor een zeer grote afbeelding.**Wat te doen**:

1. Controleer of het bestand van de laag nog steeds in de uitvoerstructuur van het project staat
2. Open het bestand in een externe viewer om te controleren of het intact is
3. Sluit andere toepassingen om geheugen vrij te maken

### De afbeelding is zwart, wit of heeft bizarre kleuren

**Mogelijke oorzaken**: de beelduitrekking heeft niets om mee te werken (een vrijwel constant frame); een float32-laag met ongebruikelijke waarden; een index die geen geldige gegevens heeft opgeleverd.**Wat te doen**:

1. Lees de cursorwaarden af — als elk kanaal op of rond nul ligt, zit het probleem in de gegevens, niet in de weergave
2. Controleer het histogram: een enkele piek aan één uiteinde geeft aan dat het frame is afgekapt of leeg is
3. Controleer het verwerkingslogboek voor de run die de laag heeft geproduceerd

### De waarden lijken onjuist

**Mogelijke oorzaken**: je bevindt je op een andere laag dan je denkt; je vergelijkt een percentage met een ruwe DN; je vergelijkt een LATTICE-bestand met een Survey3-bestand met dezelfde deler.**Wat te doen**:

1. Controleer de geselecteerde laag in de vervolgkeuzelijst — de eenheden in het paneel volgen die van de laag
2. Gebruik voor reflectie de **%**-kolom in plaats van de DN zelf te delen; als je toch moet delen, gebruik dan de `Chloros:PixelScale` van dat bestand (32768 voor LATTICE, indien afwezig betekent dit 65535 voor Survey3)
3. Stel de GSD-blokgrootte weer in op 1 — boven 1 lees je een blokgemiddelde af, geen pixel
4. Controleer of de reflectanciekalibratie daadwerkelijk is uitgevoerd voor dat frame; een niet-gekalibreerd fallback-product (Sensor Response / Vignette Corrected) is geen reflectantie

***

## Volgende stappen

* [**Beeldlagen**](image-layers.md) — elke laagnaam, indien aanwezig, en wat de waarden ervan betekenen
* [**Index/LUT-Sandbox**](index-lut-sandbox.md) — indexvisualisaties samenstellen, afstemmen en exporteren
* [**Kaartmarkeringen**](map-markers.md) — dezelfde beeldset op een kaart
* [**Multispectrale indexformules**](../project-settings/multispectral-index-formulas.md) — de indexreferentie

Zie [Beelden verwerken (GUI)](../processing-images-gui/adding-files-to-a-project.md) voor de verwerkingsworkflow.
