---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formules voor multispectrale indexen

De onderstaande indexformules maken gebruik van een combinatie van de gemiddelde transmissiebereiken van het Survey3-filter:

<table><thead><tr><th align="center">Survey3 Filterkleur</th><th width="196.199951171875" align="center">Survey3 Filternamen</th><th width="159.800048828125" align="center">Transmissiebereik (FWHM)</th><th align="center">Gemiddelde transmissie</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Wanneer deze formules worden gebruikt, kan de naam eindigen op &quot;\_1&quot; of &quot;\_2&quot;, wat aangeeft welk NIR-filter, namelijk NIR1 of NIR2, is gebruikt.

Voor LATTICE M3C-camera’s (Bayer drievoudige banddoorlaat) gebruikt dezelfde indexengine de M3C-filterbanden:

| M3C-filter | Band 1 (midden/FWHM) | Band 2 (midden/FWHM) | Band 3 (midden/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M-camera&#x27;s zijn enkelbandig (één smalbandfilter per camera), dus er worden geen multibandindexen berekend voor een afzonderlijk M3M-beeld. Om indexen te berekenen met M3M, combineer je twee of meer camera&#x27;s tot een uitgelijnde multiband-stack en gebruik je de LATTICE-indexengine (`chloros-cli lattice index`, of de live Index Calculator in de GUI).

***

## Waar elke indexnaam werkt

Chloros heeft **drie** indexoppervlakken, en hun vooraf ingestelde lijsten zijn niet identiek. Gebruik dit gedeelte om te controleren of een naam werkt op de plek waar u deze wilt gebruiken.

| Waar je bent | Welke lijst is van toepassing | Aantal |
| --- | --- | --- |
| Projectinstellingen → Index → Index toevoegen (GUI) | Oppervlak 1 | 27 |
| Afbeeldingsviewer [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Oppervlak 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Oppervlak 2 | 22 |
| SDK `process_folder(indices=[...])` | Oppervlak 2 | 22 |
| `chloros-cli lattice index --preset` | Oppervlak 3 | 22 (een andere 22) |
| Tabblad Camera&#x27;s - Live-indexcalculator | Surface 3 | 22 (een andere 22) |

Surfaces 1 en 2 verwerken **één beeld tegelijk van één camera**, met behulp van de symboolslots `x`/`y`/`z`(/`a`) die aan de filterkanalen van die camera zijn gekoppeld. Oppervlak 3 werkt met een**uitgelijnde multiband-stack** — meerdere LATTICE-camera’s die in één kubus zijn gecoördineerd — en verwijst naar kanalen met kleine letters.

### 1. GUI-projectinstellingen / Dropdownmenu ‘Sandbox’ in de afbeeldingsviewer — 27 formules

De dropdownlijst geeft ze in deze volgorde weer (dit is de volgorde waarin ze zijn ingevoegd, niet alfabetisch):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

In de GUI sleep je de filterkanalen van je camera naar de bandvakken van de formule, zodat elke formule kan worden gebruikt met elke bandtoewijzing die je camera ondersteunt. Aangepaste formules die je hebt opgeslagen, worden onderaan deze lijst toegevoegd.

De **vijf formules die alleen in de GUI beschikbaar zijn** formules — die niet worden geaccepteerd door de lijst CLI/SDK `--indices` — zijn als volgt geïmplementeerd:

| Alleen via de GUI instelbare voorinstelling | Formule (zoals geïmplementeerd) | Slots |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (vier slots) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

De beoogde koppeling voor elk van deze wordt verderop op deze pagina in een aparte sectie weergegeven (bijvoorbeeld: GARI verwacht x=Green, y=NIR, z=Blue, a=Red). GARI is de enige formule in Chloros die een vierde slot gebruikt.

### 2. CLI / SDK `--indices` naamuittrekking — 22 voorinstellingen

De optie `chloros-cli process --indices` (en de parameter SDK `indices`) accepteert de volgende voorinstellingsnamen:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Onbekende indexnamen worden zonder melding overgeslagen.** Een naam die niet in deze lijst voorkomt (inclusief de vijf formules die alleen in de GUI beschikbaar zijn: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` en elke aangepaste formule die u in de GUI hebt opgeslagen) wordt genegeerd met alleen een logboekmelding — de uitvoering gaat door zonder die index en de uitvoering zelf wordt nog steeds als succesvol gerapporteerd. De melding wordt weergegeven als:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Bij het vergelijken van namen wordt geen onderscheid gemaakt tussen hoofdletters en kleine letters en worden witruimtes verwijderd; dus `ndvi`, `NDVI` en ` NDVI ` zijn dezelfde voorinstelling. Een voorinstelling wordt ook overgeslagen als deze een band vereist die het filter van je camera niet biedt.
{% endhint %}

De exacte formules zoals geïmplementeerd (de symbolen `x`/`y`/`z` zijn bandslots; de standaardtoewijzing wordt per voorinstelling weergegeven):

| Voorinstelling | Formule (zoals geïmplementeerd) | Standaardfilter | Slots (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Hoe een presetnaam wordt omgezet in bandposities

Wanneer je een naakte naam zoals `NDVI` doorgeeft, moet Chloros bepalen welk kanaal van welk bestand elk symbool leest. Het maakt gebruik van deze tabel, die een filtercode koppelt aan de array-positie van elk kanaal:

| Filtercode | Kanaal → array-index |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` wordt geaccepteerd als alias voor Orange, ook 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Het **standaardfilter** van de voorinstelling (de kolom „Standaardfilter“ in de kolom hierboven) wordt gebruikt wanneer het project afbeeldingen bevat met dat filter. Als dat niet het geval is, doorzoekt Chloros de filters die daadwerkelijk in het project aanwezig zijn in de volgorde `RGN, OCN, NGB, RGB, RE, NIR` en kiest het de eerste die elk kanaal kan leveren dat de voorinstelling nodig heeft. Als geen enkel filter dat kan, wordt de voorinstelling voor die run genegeerd. Dit is de reden waarom `NDVI`, aangevraagd op een dataset die uitsluitend uit OCN bestaat, nog steeds een zinvol resultaat oplevert — het koppelt zich aan de posities Orange en NIR van OCN.

LATTICE M3C-modelstrings bevatten het filter met het voorvoegsel `F` (`LATT-M3C-L41-FRGN`), maar het voorvoegsel wordt weggelaten wanneer de filtercode uit de afbeelding wordt gelezen, zodat een FRGN-camera de rij `RGN` erboven kan verwerken en geen speciale behandeling nodig heeft.

### 3. LATTICE-indexengine (`lattice index --preset`, live Index Calculator) — 22 voorinstellingen

De LATTICE-engine werkt op uitgelijnde multiband-stacks (live arrays of geëxporteerde multiband-TIFF’s) en gebruikt kanaalnamen in kleine letters (`red`, `green`, `blue`, `red_edge`, `nir`). De lijst met voorinstellingen verschilt van die van de twee hierboven:

| Voorinstelling | Formule | Kanalen |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | rood, nir |
| GNDVI | `(nir - green) / (nir + green)` | groen, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | blauw, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | rood\_rand, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | blauw, groen, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | rood, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | rood, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | rood, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | blauw, rood, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | rood, NIR |
| CVI | `(nir / green) - (red / green)` | rood, groen, nir |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | rood, nir |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | rood, NIR |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | rood, groen, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | rood, groen, blauw |
| NGRDI | `(green - red) / (green + red)` | rood, groen |
| VARI | `(green - red) / (green + red - blue)` | rood, groen, blauw |
| TGI | `green - 0.39*red - 0.61*blue` | rood, groen, blauw |
| EXG | `2*green - red - blue` | rood, groen, blauw |
| CIRE | `(nir / red_edge) - 1` | rood\_rand, NIR |
| CIGREEN | `(nir / green) - 1` | groen, NIR |
| NDWI | `(green - nir) / (green + nir)` | groen, nir |

Voer `chloros-cli lattice index --list-presets` uit om deze tabel vanuit uw geïnstalleerde build af te drukken, en `--list-gradients` voor de beschikbare kleurverlopen. Kanaalsymbolen zijn hoofdlettergevoelig en moeten overeenkomen met de kleine letters in de namen van de voorinstellingen (bijv. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Zoals geïmplementeerd in de GUI en de presetlijst CLI/SDK, is CVI de ‘ratio-of-ratios’-formule:

$$
CVI = {(z / y) \over (x / y)}
$$

met de standaard RGB-kanaaltoewijzing x=Red, y=Green, z=Blue. In de GUI kun je elk kanaal van je camera naar de x/y/z-slots slepen. Let op: de voorinstelling `CVI` van de LATTICE-indexengine gebruikt een andere formule, namelijk `(NIR / Green) - (Red / Green)` — raadpleeg de tabellen hierboven voor het oppervlak dat je gebruikt.

***

## ENDVI - Enhanced Normalized Difference Vegetation Index

Deze index maakt naast NIR en groen ook gebruik van het blauwe kanaal en is populair bij NGB-gefilterde camera’s waarbij de blauwe band de rode vervangt.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

De implementatie is de symboolformule `((x+y)-(2*z))/((x+y)+(2*z))` — wijs de NIR- en Green-kanalen van je camera toe aan de x/y-slots en Blue aan z (voor een NGB-camera: x=NIR, y=Green, z=Blue).

***

## EVI - Verbeterde vegetatie-index

Deze index is oorspronkelijk ontwikkeld voor gebruik met MODIS-gegevens als verbetering ten opzichte van NDVI door het vegetatiesignaal te optimaliseren in gebieden met een hoge bladoppervlakte-index (LAI). De index is vooral nuttig in gebieden met hoge LAI-regio&#x27;s waar NDVI verzadigd kan raken. De index maakt gebruik van het blauwe reflectiebereik om achtergrondsignalen van de bodem te corrigeren en atmosferische invloeden, waaronder verstrooiing door aërosolen, te verminderen.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-waarden moeten voor vegetatiepixels tussen 0 en 1 liggen. Heldere elementen zoals wolken en witte gebouwen, evenals donkere elementen zoals water, kunnen leiden tot afwijkende pixelwaarden in een EVI-beeld. Voordat u een EVI-beeld maakt, moet u wolken en heldere elementen uit de reflectantieafbeelding maskeren en, indien gewenst, de pixelwaarden beperken tot de waarden 0 tot 1.

_Referentie: Huete, A., et al. „Overzicht van de radiometrische en biofysische prestaties van de MODIS-vegetatie-indexen.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Bosbedekkingsindex 1

_Alleen GUI — niet beschikbaar als CLI/SDK `--indices`-voorinstelling._

Deze index onderscheidt boskronen van andere soorten vegetatie met behulp van multispectrale reflectiebeelden die een ‘red edge’-band bevatten.

$$
FCI1 = Red * RedEdge
$$

Beboste gebieden zullen lagere FCI1-waarden hebben vanwege de lagere reflectie van bomen en de aanwezigheid van schaduwen binnen het bladerdak.

_Referentie: Becker, Sarah J., Craig S.T. Daughtry en Andrew L. Russ. &quot;Robuuste bosbedekkingsindexen voor multispectrale beelden.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Bosbedekkingsindex 2

_Alleen via de GUI — niet beschikbaar als voorinstelling CLI/SDK `--indices`._

Deze index onderscheidt boskronen van andere soorten vegetatie met behulp van multispectrale reflectiebeelden die geen ‘red edge’-band bevatten.

$$
FCI2 = Red * NIR
$$

Beboste gebieden zullen lagere FCI2-waarden hebben vanwege de lagere reflectie van bomen en de aanwezigheid van schaduwen binnen het bladerdak.

_Referentie: Becker, Sarah J., Craig S.T. Daughtry en Andrew L. Russ. &quot;Robuuste bosbedekkingsindexen voor multispectrale beelden.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Index voor wereldwijde milieumonitoring

_Alleen GUI — niet beschikbaar als voorinstelling CLI/SDK `--indices`._

Deze niet-lineaire vegetatie-index wordt gebruikt voor wereldwijde milieumonitoring op basis van satellietbeelden en probeert atmosferische effecten te corrigeren. Hij is vergelijkbaar met NDVI, maar is minder gevoelig voor atmosferische effecten. Hij wordt beïnvloed door kale grond; daarom wordt het gebruik ervan niet aanbevolen in gebieden met schaarse of matig dichte vegetatie.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Waarbij:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referentie: Pinty, B., en M. Verstraete. GEMI: een niet-lineaire index voor het monitoren van wereldwijde vegetatie via satellieten. Vegetation 101 (1992): 15-20._

***

## GARI - Green Atmosferisch bestendige index

_Alleen via de GUI — niet beschikbaar als voorinstelling voor CLI/SDK/`--indices`._

Deze index is gevoeliger voor een breed bereik aan chlorofylconcentraties en minder gevoelig voor atmosferische effecten dan NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

De gammaconstante is een wegingsfunctie die afhankelijk is van de aërosolomstandigheden in de atmosfeer. ENVI gebruikt een waarde van 1,7, wat de aanbevolen waarde is van Gitelson, Kaufman en Merzylak (1996, pagina 296).

_Referentie: Gitelson, A., Y. Kaufman en M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Chlorofylindex

Deze index wordt gebruikt om het chlorofylgehalte in bladeren van een breed scala aan plantensoorten te schatten.

$$
GCI = {NIR \over Green} - 1
$$

Het gebruik van brede NIR- en groene golflengten zorgt voor een betere voorspelling van het chlorofylgehalte en biedt tegelijkertijd meer gevoeligheid en een hogere signaal-ruisverhouding.

_Referentie: Gitelson, A., Y. Gritz en M. Merzlyak. „Relaties tussen het chlorofylgehalte in bladeren en spectrale reflectie, en algoritmen voor niet-destructieve chlorofylmeting in bladeren van hogere planten.“ Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Bladindex

Deze index is oorspronkelijk ontworpen voor gebruik met een digitale RGB-camera om de tarwe-bedekking te meten, waarbij de rode, groene en blauwe digitale getallen (DN’s) variëren van 0 tot 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-waarden variëren van -1 tot +1. Negatieve waarden staan voor bodem en niet-levende elementen, terwijl positieve waarden staan voor groene bladeren en stengels.

_Referentie: Louhaichi, M., M. Borman en D. Johnson. „Ruimtelijk gelokaliseerd platform en luchtfotografie voor het documenteren van de effecten van begrazing op tarwe.” Geocarto International 16, nr. 1 (2001): 65-70._

***

## GNDVI - Green Genormaliseerde Vegetatie-index

Deze index is vergelijkbaar met NDVI, behalve dat hij het groene spectrum van 540 tot 570 nm meet in plaats van het rode spectrum. Deze index is gevoeliger voor de chlorofylconcentratie dan NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Bron: Gitelson, A., en M. Merzlyak. &quot;Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Geoptimaliseerde, voor de bodem gecorrigeerde vegetatie-index

Deze index is oorspronkelijk ontworpen met behulp van kleur-infraroodfotografie om de stikstofbehoefte van maïs te voorspellen. Hij is vergelijkbaar met OSAVI, maar vervangt de groene band door de rode.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referentie: Sripada, R., et al. „Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.” Proefschrift, North Carolina State University, 2005._

***

## Verhouding tussen GRVI en Green: Vegetatie-index

Deze index is gevoelig voor fotosynthetische snelheden in boskronen, aangezien de groene en rode reflecties sterk worden beïnvloed door veranderingen in bladpigmenten.

$$
GRVI = {NIR \over Green }
$$

_Referentie: Sripada, R., et al. „Luchtfotografie in kleur en infrarood voor het bepalen van de stikstofbehoefte bij maïs in het vroege seizoen.” Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Bodemgecorrigeerde vegetatie-index

Deze index is oorspronkelijk ontworpen met behulp van kleur-infraroodfotografie om de stikstofbehoefte van maïs te voorspellen. Hij is vergelijkbaar met SAVI, maar vervangt de groene band door de rode.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referentie: Sripada, R., et al. „Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.” Doctoraatsverhandeling, North Carolina State University, 2005._

***

## LAI - Bladoppervlakte-index

Deze index wordt gebruikt om de bladbedekking te schatten en de groei en opbrengst van gewassen te voorspellen. ENVI berekent de groene LAI aan de hand van de volgende empirische formule van Boegh et al. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Waarbij EVI gelijk is aan:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hoge LAI-waarden liggen doorgaans tussen ongeveer 0 en 3,5. Wanneer de scène echter wolken en andere heldere elementen bevat die verzadigde pixels veroorzaken, kunnen de LAI-waarden hoger zijn dan 3,5. Idealiter moet u wolken en heldere elementen uit uw scène wegmaskeren voordat u een LAI-afbeelding maakt.

_Referentie: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde en A. Thomsen. „Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.” Remote Sensing of Environment 81, nr. 2-3 (2002): 179-193._

***

## LCI - Chlorofylindex van bladeren

_Alleen via de GUI — niet beschikbaar als voorinstelling voor CLI/SDK `--indices`._

Deze index wordt gebruikt om het chlorofylgehalte in hogere planten te schatten en is gevoelig voor variaties in reflectie die worden veroorzaakt door chlorofylabsorptie.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referentie: Datt, B. &quot;Remote Sensing of Water Content in Eucalyptus Leaves.&quot; Journal of Plant Physiology 154, nr. 1 (1999): 30-36._

***

## MNLI - Modified Non-Linear Index

Deze index is een verbetering van de Non-Linear Index (NLI), waarbij de Soil Adjusted Vegetation Index (SAVI) is geïntegreerd om rekening te houden met de bodemachtergrond. ENVI gebruikt een aanpassingsfactor voor de bladerdakachtergrond (_L_) met een waarde van 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referentie: Yang, Z., P. Willis en R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 - Modified Soil Adjusted Vegetation Index 2

Deze index is een eenvoudigere versie van de door Qi et al. (1994) voorgestelde MSAVI-index, die een verbetering vormt ten opzichte van de Soil Adjusted Vegetation Index (SAVI). Het vermindert bodemruis en vergroot het dynamisch bereik van het vegetatiesignaal. MSAVI2 is gebaseerd op een inductieve methode die geen constante _L_-waarde gebruikt (zoals bij SAVI) om gezonde vegetatie te benadrukken.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referentie: Qi, J., A. Chehbouni, A. Huete, Y. Kerr en S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - Modified Simple Ratio

Deze index is een aanpassing van de eenvoudige NIR/Red-verhouding, bedoeld om de relatie met biofysische parameters te lineariseren, en is bij hogere vegetatiedichtheden gevoeliger dan NDVI.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referentie: Chen, J. „Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.“ Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE- Genormaliseerd verschil RedEdge

Deze index is vergelijkbaar met NDVI, maar vergelijkt het contrast tussen NIR en RedEdge in plaats in plaats van met Red, waardoor vegetatiestress vaak eerder wordt gedetecteerd.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Genormaliseerde vegetatie-index

Deze index is een maatstaf voor gezonde, groene vegetatie. De combinatie van de genormaliseerde verschilformule en het gebruik van de gebieden met de hoogste absorptie en reflectie van chlorofyl zorgt ervoor dat de index onder uiteenlopende omstandigheden robuust is. De index kan echter verzadigd raken bij dichte begroeiing, wanneer de waarde van LAI hoog wordt.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

De waarde van deze index varieert van -1 tot 1. Het gebruikelijke bereik voor groene vegetatie ligt tussen 0,2 en 0,8.

_Bron: Rouse, J., R. Haas, J. Schell en D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI - Niet-lineaire index

Deze index gaat ervan uit dat de relatie tussen veel vegetatie-indexen en biofysische oppervlakteparameters niet-lineair is. Hij lineariseert relaties met oppervlakteparameters die doorgaans niet-lineair zijn.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referentie: Goel, N., en W. Qin. „Invloeden van de bladerdakarchitectuur op de relaties tussen diverse vegetatie-indexen en LAI en Fpar: een computersimulatie.“ Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Geoptimaliseerde bodemaangepaste vegetatie-index

Deze index is gebaseerd op de bodemaangepaste vegetatie-index (SAVI). Er wordt een standaardwaarde van 0,16 gebruikt voor de correctiefactor voor de bladerdakachtergrond. Rondeaux (1996) stelde vast dat deze waarde bij een lage vegetatiebedekking een grotere bodemvariatie oplevert dan SAVI, terwijl de gevoeligheid voor een vegetatiebedekking van meer dan 50% toeneemt. Deze index kan het best worden gebruikt in gebieden met relatief schaarse vegetatie, waar de bodem door het bladerdak heen zichtbaar is.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referentie: Rondeaux, G., M. Steven en F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Renormalized Difference Vegetation Index

Deze index maakt gebruik van het verschil tussen nabij-infrarode en rode golflengten, in combinatie met de NDVI, om gezonde vegetatie te benadrukken. Hij is ongevoelig voor de effecten van de bodem en de stand van de zon ten opzichte van de waarnemer.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referentie: Roujean, J., en F. Breon. „Schatting van de door vegetatie geabsorbeerde PAR op basis van bidirectionele reflectiemetingen.” Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Bodemgecorrigeerde vegetatie-index

Deze index is vergelijkbaar met NDVI, maar onderdrukt de effecten van bodempixels. Er wordt gebruikgemaakt van een aanpassingsfactor voor de bladerdakachtergrond, _L_, die een functie is van de vegetatiedichtheid en waarvoor vaak voorafgaande kennis van de hoeveelheid vegetatie vereist is. Huete (1988) stelt een optimale waarde van _L_=0,5 voor om rekening te houden met variaties in de bodemachtergrond van de eerste orde. Deze index kan het best worden gebruikt in gebieden met relatief schaarse vegetatie waar de bodem door het bladerdak heen zichtbaar is.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referentie: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Getransformeerde verschilvegetatie-index

Deze index is nuttig voor het monitoren van de vegetatiebedekking in stedelijke omgevingen. Hij raakt niet verzadigd zoals NDVI en SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referentie: Bannari, A., H. Asalhi en P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” in Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, deel 5 (2002)._

***

## VARI - Index voor zichtbare, atmosferisch resistente vegetatie

Deze index is gebaseerd op de ARVI en wordt gebruikt om het aandeel vegetatie in een beeld te schatten met een lage gevoeligheid voor atmosferische effecten.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referentie: Gitelson, A., et al. &quot;Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Vegetatie-index met breed dynamisch bereik

Deze index is vergelijkbaar met NDVI, maar maakt gebruik van een wegingscoëfficiënt (_a_) om het verschil tussen de bijdragen van de signalen in het nabij--infrarood- en rode signalen aan de NDVI. De WDRVI is bijzonder effectief in beelden met een matige tot hoge vegetatiedichtheid wanneer de NDVI groter is dan 0.6 overschrijdt. De NDVI heeft de neiging af te vlakken wanneer de vegetatiefractie en de bladoppervlakte-index (LAI) toenemen, terwijl de WDRVI gevoeliger is voor een breder bereik aan vegetatiefracties en voor veranderingen in LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

De wegingscoëfficiënt (_a_) kan variëren van 0,1 tot 0,2. Een waarde van 0,2 wordt aanbevolen door Henebry, Viña en Gitelson (2004).

_Referenties_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, nr. 2 (2004): 165-173._

_Henebry, G., A. Viña en A. Gitelson. „The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.” Gap Analysis Bulletin 12: 50-56._
