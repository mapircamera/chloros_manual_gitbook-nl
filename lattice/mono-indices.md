# Monochrome camera’s en vegetatie-indexen

## Eén camera = één band

Een **M3M**-camera is de monochrome variant van de Bayer**M3C**: een monochrome IMX265-sensor achter één enkel smalband-interferentiefilter. De modelnaam geeft de band aan — `M3M-<lens>-F<wavelength>`, bijvoorbeeld `M3M-L87-F685` (in Chloros weergegeven als `LATT-M3M-L87-F685`). De sensor levert een **enkele grijswaardenband** zonder Bayer-mozaïek: er hoeft niets te worden gedemosaïkeerd, er is geen kruisoverdracht tussen kanalen die moet worden gescheiden en er hoeft geen witbalans te worden ingesteld.

Gevolgen die je moet weten voordat je een monosysteem plant:

* **Straling en reflectie zijn volledig gedefinieerd per band.**Het zijn radiometrische kaarten per band, dus één M3M-camera produceert gekalibreerde float32-straling (W/m²/sr/nm) en uint16-reflectantie (`32768` = ρ 1,0), precies zoals een M3C-band dat doet. Mono-frames bevatten een**identieke** sensorresponsmatrix — er is geen 3×3-unmix nodig en deze wordt ook niet toegepast.
* **Een enkele monocamera kan geen vegetatie-index produceren.** NDVI, NDRE en dergelijke indices vereisen ten minste twee banden. Om indices te berekenen met monohardware combineer je meerdere M3M-camera’s — zie hieronder.
* M3M-camera’s streamen **Mono12** (12-bit: 2 bytes/pixel via de kabel, of 1,5 bytes/pixel wanneer een array gebruikmaakt van [gecomprimeerd 12-bit transport](arrays.md#packed-12-bit-transport)), wat van belang is voor [het budgetteren van de array-bandbreedte](arrays.md#bandwidth-how-many-cameras-fit).

## Wat Chloros overslaat bij mono — en hoe het je dat laat weten

Fasen van de kleurenpijplijn zijn simpelweg niet van toepassing op een sensor met één band. Chloros **slaat ze over met een bericht van één regel** in plaats van een foutmelding te geven, en voert ze nog steeds normaal uit voor elke M3C (Bayer)-camera in dezelfde sessie:

| Fase | Gedrag bij mono (M3M) | Gedrag bij M3C |
| --- | --- | --- |
| Demosaic / debayer | Overgeslagen — het `debayered`-exportniveau is een grijswaardenafbeelding met 1 kanaal. | 3-kanaals demosaic. |
| Witbalans (`lattice white-balance`) | Overgeslagen met een bericht van één regel. | Wordt normaal uitgevoerd. |
| Kleurprofiel (`lattice color-profile`) | Overgeslagen met een bericht van één regel. | Wordt normaal uitgevoerd. |
| Verzadiging/contrast (`lattice color`) | Overgeslagen met een bericht van één regel. | Werkt normaal. |
| Spectrale crosstalk-ontmenging | Identiteit (geen 3×3-matrix). | 3×3-matrix per camera toegepast. |
| Stralingsintensiteit / reflectie | **Wordt uitgevoerd** — per band, volledig gekalibreerd. | Wordt per band uitgevoerd. |

De GUI past dezelfde gating toe: voor een monocamera verbergt het instellingenvenster per camera de rijen die alleen betrekking hebben op RGB (witbalans, gamma, kleurprofiel, verzadiging, contrast, kanaalsplitsingen), en is het live-histogram vergrendeld op één enkele **MONO**-trace. De onderscheidende factor in de hele stack is het `M3M`-token in de modelstring, dat in de GUI/SDK wordt weergegeven als `is_mono`.

## Indexen vereisen ≥ 2 banden: uitlijnen → stapelen → indexeren

De workflow voor mono-indexen bestaat altijd uit dezelfde drie stappen:

1. **Uitlijnen** — richt meerdere M3M-camera’s op verschillende golflengten (bijv. een F650 „Red “ en een F850 „NIR “), verbind ze als een [multi-camera-array](arrays.md) en laat Chloros de co-registratie-warp tussen de camera’s berekenen.
2. **Stapelen** — de uitgelijnde frames worden één multibandbeeld (elke camera draagt één benoemde band bij).
3. **Index** — evalueer een indexformule over de banden van de stack, waarbij deze optioneel via een LUT kan worden weergegeven.

In de GUI is deze hele keten de weergavemodus **Combined Cameras**: de live compositie is al uitgelijnd en de Index Calculator van de array (hieronder) definieert de formule die wordt weergegeven. Opgenomen exportbestanden kunnen met de opnameoptie**Aligned** naar dezelfde uitlijning worden verplaatst.

## De Index Calculator

De Index Calculator stelt de indexuitdrukking samen die wordt gebruikt door de liveweergave en de indexexport per camera. Het is één gedeeld venster, dat vanaf twee plaatsen in de zijbalk van het tabblad ‘Cameras’ kan worden geopend:

* **Per camera**— Live Preview →**Index**-tandwiel (alleen RGN-/OCN-/NGB-Bayer-camera’s; een enkele monocamera heeft geen indexregeling omdat één band geen index kan vormen).
* **Per array**— array-instellingen → Live Preview →**Index**-tandwiel. Dit is het mono-traject: de bandenlijst omvat**alle bijbehorende camera’s**, dus een monopaar draagt hier zijn twee banden bij.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

De bedieningselementen, van boven naar beneden:

* **Bandchips** („Banden — klik om toe te voegen aan expressie”) — één knop per beschikbare band, gelabeld met kleurnaam + golflengte in nm (dubbele kleurnamen worden onderscheiden, bijvoorbeeld als „Kleur 850”). Door te klikken wordt het bandtoken op de cursorpositie ingevoegd. Banden van camera’s die geen straling per band kunnen produceren (RGB/FRGB) worden eruit gefilterd.
* **Operator- en functieknoppen** — `+ - * / ( ) ^ ,` plus `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Tekstveld voor uitdrukkingen** — vrij in te voeren formule; de plaatshouder toont de klassieke NDVI-vorm `(NIR - Red) / (NIR + Red)`. Een alleen-lezen, in tokens opgedeeld voorbeeld erboven geeft bandchips, getallen en vlaggen weer als onbekende tokens.
* **Geldigheidsbanner**— grijs „Leeg — er wordt geen index toegepast“; groen „Geldige uitdrukking“; rood met de specifieke parseerfout (onbekende band, dubbelzinnige band door meerdere camera’s, ontbrekende haakjes, …); of oranje wanneer de uitdrukking geldig is maar**constant** (bijv. `X/X`, of een NDVI-noemer die is ingevoerd als `−` in plaats van `+`) — een constante brengt het gehele beeld in één kleur weer.
* Er verschijnt een aparte oranje waarschuwing als de toegepaste uitdrukking in orde is, maar het **live-frame uniform is** (vlakke of verzadigde scène) — de histogram-ineenstorting wordt automatisch gedetecteerd.
* **LUT toepassen**(standaard aan; uit = grijsschaaluitrekking),**Niveau**2/3/5/7-stop (standaard 7-stop) en**Min / Max**-invoervelden aan weerszijden van de verloopbalk. Min is standaard ingesteld op**0,2**— dit zoomt de kleurenramp in op het voor vegetatie relevante bereik, terwijl waarden daaronder als grijstinten worden doorgegeven; stel Min in op −1 voor het volledige indexbereik (de**Reset**-knop herstelt −1…+1). Max is standaard ingesteld op 1.
* **Live-histogram** van de indexverdeling — balken op basis van de vierkantswortel, amberkleurige p2/p98-percentiellijnen, een witte mediaanlijn en uitlezen van waarden buiten het bereik („◀ N% < lo“ / „hi < N% ▶") die boven 1 % amberkleurig worden als teken dat het Min/Max-venster moet worden verbreed.
* **Toepassen** past de uitdrukking toe op de livestream; LUT-aanpassingen worden direct toegepast zonder op Toepassen te drukken. Als er een project is geopend, worden de uitdrukking en LUT-instellingen samen met het project opgeslagen en hersteld wanneer je het opnieuw opent; zonder project blijven ze alleen gedurende de sessie behouden.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Het CLI-pad

Dezelfde keten van uitlijnen → stapelen → indexeren, van begin tot eind programmeerbaar:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` gebruikt een gekoppelde array die de camera’s bevat, of opent deze zelf voor één uitvoering. `index --live` opent de camera’s altijd zelf (een [tijdelijke array](../reference/cli-reference.md#multi-camera-transient-sessions)) en heeft geen `--serials`: het opent elke camera op het netwerk. Geef dus elke camera vrij die Chloros vasthoudt voordat — de GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): het een tweede keer openen van een vastgehouden camera kan de stream van de houder stoppen of de backend laten crashen. Een camera die geen frame levert, zorgt ervoor dat de run mislukt, en er wordt geen stack of index geschreven.

`--channel` koppelt de symbolen van een preset aan de bandnamen van de stack. Twee regels helpen je een mislukte run te voorkomen:

* **Symbolen zijn hoofdlettergevoelig** en moeten exact overeenkomen met de kanaalnamen van de preset — presets gebruiken kleine letters (die van NDVI zijn `red`,`nir`; zie `--list-presets`). `--channel red=Red_660` werkt; `--channel RED=660` mislukt met een `channel_map missing entries`-fout.
* De bandzijde moet een band in de uitgelijnde stapel benoemen (`lattice align-info --profile align.json` geeft een overzicht). De offline-modus accepteert ook bandindexen die beginnen bij 0, bijvoorbeeld `--channel red=0 --channel nir=1`.

`lattice index` werkt ook offline met een opgeslagen, uitgelijnde TIFF met meerdere banden. Elk formulier draait op de backend, dus deze moet actief zijn en aangemeld zijn op een betaald Chloros+-abonnement; een relatief `--input` of `--output` verwijst naar de map van waaruit je de opdracht uitvoert:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indexvoorinstellingen

`lattice index --preset` (en de [Index/LUT-sandbox](../image-viewer-gui/index-lut-sandbox.md) op het tabblad Afbeelding, die dezelfde engine gebruikt) bevat deze **22 voorinstellingen**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Voer `chloros-cli lattice index --list-presets` uit voor de formule en kanaalsymbolen van elke voorinstelling, en `--list-gradients` voor de beschikbare kleurverloop. Aangepaste formules gebruiken `--formula EXPR` met dezelfde syntaxis als de Index Calculator. Let op: deze lijst met voorinstellingen is specifiek voor de LATTICE-indexengine — de vervolgkeuzelijst ‘Verwerking’ in de projectinstellingen voor geïmporteerde beelden bevat een andere lijst (zie [Multispectrale indexformules](../project-settings/multispectral-index-formulas.md)).

De volledige set vlaggen (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, uitlijnings- en vervormingsregelaars voor `--live`, en meer) is gedocumenteerd in de [CLI-referentie § Index / Vegetatie-wiskunde](../reference/cli-reference.md#index--vegetation-maths); SDK-equivalenten staan in de [SDK-referentie](../reference/sdk-reference.md).

## Indexproducten vastleggen vanuit een mono-array

Met een aangesloten array en een toegepaste indexuitdrukking slaat `array-capture` (of de GUI-optie **Capture All**) slaat de exportniveaus per camera *en* de indexweergave op — `--index`/`--no-index` schakelt dit in via de CLI en legt standaard alle toepasselijke niveaus vast. De bijdrage van een monocamera aan elke opnamegroep bestaat uit de ene band op raw-/debayered- (grijswaarden)-/radiance-/reflectance-niveaus, plus de gedeelde gecombineerde-indexcompositie wanneer de array in de gecombineerde modus draait. Zie [Multi-cameramatrices § Opname](arrays.md#capturing-monitoring-vs-analysis).
