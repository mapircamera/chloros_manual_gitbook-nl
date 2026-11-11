# Chloros CLI-referentie

**Versie:**

1.3.0**Gemaakt:**29 juli 2026 19:19 ·**Herzien:** 6 november 2026**Doelgroep:** Geoptimaliseerd voor gebruik door LLM’s; leesbaar voor mensen.**Reikwijdte:** Alle voor gebruikers bestemde subcommando’s van `chloros-cli`, inclusief opties en voorbeelden die gekopieerd en geplakt kunnen worden.

Dit document is de volledige referentie voor het `chloros-cli`-opdrachtregelprogramma dat bij MAPIR Chloros wordt geleverd. Het is bewust zo uitgebreid opgesteld dat een LLM (of mens) elke ondersteunde workflow kan samenstellen op basis van de onderstaande lijsten, zonder de broncode te hoeven bekijken.

Als u alleen de belangrijkste punten nodig hebt, ga dan naar:
- [Snelstart in vijf minuten](#five-minute-quickstart)
- [Workflow voor eerste verbinding met LATTICE-camera](#lattice-camera-first-connect-workflow)
- [Workflow voor eerste verbinding met DAQ-sensor](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Opnamemodi, recorders en offline herverwerking](#capture-modes-recorders--offline-reprocess)

> **Chloros-besturing via een AI-assistent?** Claude Desktop, Claude Code, Cursor en andere MCP-clients kunnen Processing, LATTICE-camera’s en -arrays, en DAQ-lichtsensoren aansturen via `chloros-mcp`, de MCP-server die wordt geïnstalleerd met de Python SDK (en met het Linux .deb-pakket, in `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Configuratie, opties en hulpmiddelen staan in de SDK-referentie onder [MCP-server (AI-assistenten)](sdk-reference.md#mcp-server-ai-assistants) van de SDK-referentie. Een assistent met een shell kan ook elke opdracht op deze pagina direct uitvoeren.

> **Gewijzigd in 1.3.0** (wat een bestaand 1.2.0-script of een bestaande 1.2.0-workflow opmerkt).
>
> **Wijzigingen waarmee een 1.2.0-script mogelijk rekening moet houden:**
>
> - **`--preset` is van toepassing.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` en `align-stream` zijn in versie 1.2 verwijderd.0 en draaide op de eigen instellingen van de camera. Het stelt nu het formaat, de belichting, de versterking en de beeldsnelheid van de voorinstelling in: `--preset high_speed` neemt 8-bit op bij 5 ms, 6 dB, 60 fps. `lattice cam-connect --preset` behoudt ook het formaat van de preset (in versie 1.2.0 werd de camera na het toepassen van de preset ingesteld op 12-bit). Zie [Algemene opties](#common-options-shared-by-most-subcommands).
> - **Een Bayer-pixelformaat op een M3M (mono) camera is de bitdiepte ervan.** De voorinstellingen noemen Bayer-formaten, die een monosensor niet kan vastleggen: in 1.2.0 weigerde de camera het formaat en behield het formaat dat hij al had, dus `--preset high_speed` was daar geen 8-bit. Het is nu het Mono-formaat met dezelfde diepte: `high_speed` en `--pixel-format BayerRG8` leveren `Mono8`, `high_quality`, `triggered` en `BayerRG12` leveren `Mono12` op. In een gemengde M3C + M3M-array neemt elke camera het gedeelde formaat binnen zijn eigen familie aan, en `network-analysis` / `analyze-array --pixel-format` projecteren elke camera op die diepte. Hetzelfde geldt andersom: een M3C waarvoor een Mono-formaat is aangevraagd, neemt het Bayer-formaat van die diepte aan, dus een array waarvan de master een M3M is, opent zijn M3C-camera’s niet langer in Mono-modus. Zie [Mono (M3M) single-band-camera’s](#mono-m3m-single-band-cameras).
> - **`lattice array-capture`-exports zijn standaard uitgelijnd, net als die van de app.** De `debayered`-, `display`, `radiance`, `reflectance` en indexbestanden worden aangepast aan het beeld van de hoofdcamera en bijgesneden tot de overlap van de leden (kleiner dan de sensor vanwege de offsets van de camera’s, `XMP-mapir:AlignmentJSON` `"applied": true`), dus een CLI-opname bestaat uit dezelfde bestanden als de opname van dezelfde scène door de Chloros-app; `raw` blijft in de oorspronkelijke indeling. Voorheen werden bij een opname zonder `--aligned` alle bestanden in de oorspronkelijke grootte opgeslagen, zonder vervorming. `--no-aligned` behoudt dat. Dezelfde standaardinstelling geldt voor `--smart`, `--continuous` en `--interval`, en voor de SDK en `chloros-mcp`. Zie [`array-capture`-modi](#array-capture-modes).
> - **`lattice array-capture --compression` is standaard ingesteld op `none`.** Bestanden worden ongecomprimeerd geschreven (ongeveer 6,3 MB per frame op volledige resolutie) voor de hoogste aanhoudende snelheid; versie 1.2.0 schreef `deflate` (ongeveer 4,1 MB). Geef `--compression deflate` op voor kleinere bestanden. Beide zijn verliesvrij.
> - **PNG en JPEG accepteren alleen de `display` / `preview` en indexproducten.** Een opname met `--format png` of `jpg` met een niveau dat een 8-bits bestand niet kan bevatten, wordt geweigerd voordat er iets wordt opgenomen (HTTP-status 400, afsluiten `1`), net als de standaard `--processing all` met `--format png`/`jpg`. Zie [Capture-exportniveaus](#capture-export-levels-the-all-default).
> - **`lattice dls` heeft `--mac` nodig.**Er is geen standaard-MAC meer (dat was het adres van één eenheid); `daq pool-discover --only ble` zoekt dat van een sensor.**`lattice dls --camera SERIAL`** is nieuw.
> - **Bij meer fouten wordt `1` afgesloten.** `process` wanneer de exportgegevens van een run niet konden worden geverifieerd of er een afbeelding verloren is gegaan, hoewel de andere producten wel zijn weggeschreven; een opnamecommando met een willekeurig bestand dat niet kon worden weggeschreven (de andere bestanden worden nog steeds opgeslagen); `lattice continuous` en `multi-stream` bij een volle schijf; `array-burst` / `array-record` wanneer de recorder een fout meldt; `array-build-video` wanneer een aangevraagd product werd overgeslagen of niet volgens de aanvraag werd opgebouwd; `project burst` / `stream` wanneer een opname halverwege mislukt; `project sensor log` wanneer de opname geen waarde bevat; `lattice exposure --set`, `gain --set` en `configure --set` met een waarde die de camera weigert. `lattice network-analysis --json` wordt afgesloten net als het overzicht (`1` voor `needs_force_slip`). Omgekeerd: `daq pool-disconnect` van een sensor die niet langer in de pool zit, sluit `0` afX. Zie [Exitcodes](#exit-codes).
> - **Een trage `lattice array-connect` mislukt nooit als de array is aangesloten.** Het wacht maximaal 600 s (180 s in 1.2.0; `project connect` 90 s), waarbij de stap van de backend gaandeweg wordt weergegeven, en een verbinding die tegen die tijd nog niet tot stand is gekomen, wordt door de backend ongedaan gemaakt — elke camera wordt teruggezet zoals deze werd aangetroffen — dus een `Error:` (afsluiten `1`) betekent dat er geen array is aangesloten. In 1.2.0 kon een verbinding direct nadat de camera’s waren ingeschakeld, de wachttijd overschrijden en de array verbinden nadat het commando was mislukt. Ctrl+C annuleert nu de verbinding. Zie [Persistente verbindingen](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` behoudt de volledige sensor.** Zonder `--width` / `--height` / `--binning` wordt elke camera ingesteld op de volledige sensor en daalt de framesnelheid tot het niveau dat deze computer en dit netwerk kunnen leveren; de verbindingsuitvoer geeft snellere opties weer (binning of een kleinere ROI) met de verwachte snelheden. Versie 1.2.0 verkleinde of binnede de frames om de snelheid op peil te houden. Zie [`array-connect`-opties](#array-connect-options).
> - **Een traag netwerk weigert een array niet langer.** Elke camera wordt aangepast aan zijn aandeel in de bandbreedte en de framesnelheid daalt mee. Wanneer de camera’s niet zo langzaam kunnen worden aangepast, draait de array in seriële uitstoot (`slip-emit-and-capture`): elk frame komt binnen, maar de camera’s worden niet tegelijkertijd belicht. `--force-tier sim-capture-sim-emit` dringt aan op gelijktijdige belichting. Zie [Array fps &amp; burst-model](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` geven weer welke connect-instelling van toepassing is**: zonder `--pixel-format` worden de formaten ingesteld door connect (Mono12 / BayerRG12, voorheen BayerRG8) en de keuze voor [packed 12-bit](#packed-12-bit-transport---packed). Hetzelfde commando kan daarom een andere framesnelheid aangeven dan in 1.2.0.
> - **Relatieve paden verwijzen naar de locatie waar je het commando uitvoert.** `-o` en de andere paden die de backend opent (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), het `output` van een receptstap, en de standaard `output/` van `lattice index --live` worden omgezet naar de map van waaruit je `chloros-cli` uitvoert. In 1.2.0 loste de backend ze op in zijn eigen installatiemap, wat bij een geïnstalleerde build mislukte met de foutmelding „Toegang geweigerd“. Zie [Conventies](#conventions).
> - **`lattice index` draait in elke vorm op de backend**, net als de andere `lattice`-opdrachten, dus start eerst de backend. De offline-index (`--input`) wordt berekend door de backend, die de TIFF-bestanden leest en de resultaten ernaast of op `-o` opslaat, en `--list-presets` / `--list-gradients` geven de tabellen van de backend weer. Op Windows werken deze nu: voorheen beantwoordde de Windows-CLI elke vorm van `lattice index`, inclusief `--live`, met `Error: lattice_sdk not available`. Op Linux draaide de offline-index vroeger binnen de CLI zonder backend en zonder aanmelding; nu zijn beide nodig, net als bij elke andere CLI-opdracht. Zie [Index / Vegetatiewiskunde](#index--vegetation-maths).
> - **Nieuwe bestanden naast de producten.** Elk radiance-, reflectance- en indexproduct waarin de band-unmix een pixel heeft geklemd, krijgt een masker in een map met de naam `clamp_masks/` naast de map van het product, en er wordt een regel met de naam `clamp mask:` afgedrukt. Zie [Unmix-clamp-masker](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Camera-oriëntatie.** Bij een opname met één camera worden `raw` en `debayered` opgeslagen zoals de sensor ze heeft gelezen en wordt de spiegeling/rotatie van de camera vastgelegd in XMP; `process` past dit toe op de producten die het maakt. Bestanden in de array blijven in hun oorspronkelijke formaat, tenzij je de nieuwe `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Zie [Cameraoriëntatie](#hoe-een-opnamemap-eruit-ziet).
> - **Een exportniveau-naam die niet &#x27;één&#x27; is, wordt geweigerd.** De lijsten `lattice capture --processing` en alle `--levels`-lijsten bevatten `raw`, `debayered`, `radiance`, `reflectance` of `display` (of een alias zoals `preview`; `--processing` en ook `all`). Elke andere naam wordt geweigerd voordat de camera wordt aangeraakt (HTTP 400 `UNKNOWN_LEVEL`, afsluiten `1`); 1.2.0 heeft hiervoor `raw` opgeslagen en is afgesloten met `0`.
> - **`daq pool-set-cap` heeft `--cap-id` nodig.** Zonder dit wordt het commando afgesloten met `2`; 1.2.0 heeft de cap-correctie verwijderd, waardoor een onbedekte sensor overblijft. `--cap-id none` vraagt om de kale sensor.
> - **`lattice resolution --binning-mode` is verdwenen.** Binning is altijd ‘Average’; de vlag wordt nog steeds geaccepteerd, er wordt een waarschuwing weergegeven en er gebeurt niets. Zie [Sensorafstemming](#sensor-tuning).
> - **`process --debayer texture-aware` wordt geweigerd voor LATTICE-opnames.** Voor een map met LATTICE-opnames sluit `process` na het importeren `1` af, waarbij het aantal opnames wordt vermeld, voordat er iets wordt verwerkt. LATTICE-opnames krijgen altijd de standaarddemosaïekbehandeling, dus versie 1.2.0 accepteerde de optie maar negeerde deze; de Chloros-app biedt deze optie er ook niet voor aan. Zie [`chloros-cli process`](#chloros-cli-process).
> - **Camera&#x27;s worden teruggezet naar de toestand waarin ze werden aangetroffen — tenzij `--keep-settings`.** `lattice array-disconnect`, elke tijdelijke opdracht (`multi-capture` zonder array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) en `lattice continuous` zetten elke camera terug zoals ze die aantroffen: de bedrading van de sluiter en de synchronisatielijn, de PTP-instellingen en alle camera-instellingen die tijdens de sessie zijn gewijzigd (belichting en versterking, het AE-doel, limieten en demping, witbalans, zwartniveau, pixelformaat, binning en ROI, spiegelingen, beeldsnelheid). `--keep-settings` slaat daarentegen de instellingen van elke camerainstellingen op in het eigen geheugen van de camera, zodat deze daarmee opstart; `--clear-settings` reset dat geheugen naar de fabrieksinstellingen. In beide gevallen blijven de camera’s vrij draaien. Een project stelt zijn camera’s in vanuit zijn eigen opgeslagen instellingen wanneer het ze aansluit. Zie [Camerageheugen](#camera-memory----keep-settings----clear-settings).
> - **Elke modus slaat een project op dezelfde manier op.** Elke wijziging aan een camera, een array of een lichtsensor die onder een project is aangesloten — vanuit de app, de CLI, de SDK of de MCP-server — wordt opgeslagen in het `cameras.json` / `sensors.json`, en het openen van het project in welke modus dan ook stelt de apparaten in op basis daarvan. Voorheen sloeg alleen de app het project op, en bij de volgende opslag werd alles overschreven. Zie [Eén projectrecord, elke modus](#one-project-record-every-mode).
> - **Bestanden die zijn gemaakt met meerdere camera’s vermelden elke camera afzonderlijk.** Een multiband-stack (`align-apply`, `align-stream`, `index --live --save-multiband`), een live-index samengesteld uit de banden van verschillende camera’s, en een afbeelding uit een array-opname met de naam `composite/` bevatten elk het serienummer, het model en het kalibratiepakket van de broncamera als lijsten (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), en EXIF `SerialNumber` geeft de serienummers weer, gescheiden door komma’s. Vroeger bevatten ze helemaal geen camera-informatie, of alleen het serienummer van de eerste camera. Zie [Bestanden gemaakt met meerdere camera’s](#files-made-from-several-cameras).
> - **Alleen een `raw`-bestand bevat de kalibratietags van de camera.** De MicaSense-compatibele set (`BlackLevel`, `RadiometricCalibration`, het per kanaal geldende `Band{R,G,B}RadiometricCalibration`, de `MicaSenseVignetting*`-sets) en de `CalibrationJSON`-payload beschrijven de ruwe waarden van de sensor; deze worden dus uitsluitend in het `raw`-opnamebestand opgenomen. Alle andere bestanden — de `debayered`-, `display`-, `radiance`-, `reflectance` en indexbestanden, `lattice continuous` en `multi-stream`-frames, en elk `process`-product — bevat de identiteit van de camera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) en, bij een opname, de eigen gegevens van het frame (`XMP-mapir:FrameJSON`: belichting, versterking, temperatuur, pixelformaat, opnametijd). In versie 1.2.0 bevatten ze ook de raw-set, die door een tool (Pix4D, Metashape) toepaste op pixels die geen raw-waarden zijn. `process` vindt de kalibratie van zo’n bestand aan de hand van het serienummer en `CalBundleSha256`.
>
> **Overige wijzigingen:**
>
> - **De automatische beeldsnelheid van een array wordt afgestemd op wat de array levert.** Een snelheid die meer dan 1 % van zijn triggers verliest, wordt verlaagd – dit wordt beoordeeld op basis van meerdere langdurige metingen in plaats van één – en een snelheid waarnaar eerder is teruggeschakeld, wordt af en toe opnieuw geprobeerd. Een snelheid die je zelf instelt, blijft ongewijzigd.
> - **De automatische belichting is stabieler.** Deze blijft constant bij gelijkblijvend licht, stabiliseert zich binnen enkele seconden na een verandering, negeert kleine hoge lichten en herstelt de werking van een camera die niet meer reageert in fel zonlicht. Zie [Hoe automatische belichting werkt](#how-auto-exposure-behaves).
> - **`process` telt alleen je afbeeldingen en opnames.** Als je het opnieuw uitvoert op een map die het `camera_cal/`-bestand van een project bevat, worden die kalibratiebestanden niet meer meegeteld („96 foto’s gevonden” bij een set van 90 foto’s), en een project dat ze al had opgenomen, verwijdert ze.
> - **`lattice multi-test` gaat uit van de inschakelingstoestand van de camera’s.** Het schakelt de PTP van de camera’s in voor de run, wacht tot hun klokken zijn vergrendeld en gestabiliseerd vóór de eerste trigger — elke camera is 120 s vergrendeld, ongeveer 2 minuten vanaf het inschakelen (`--ptp-settle SECONDS` stelt dit in), aangezien een klok die net is gesynchroniseerd gedurende ongeveer 2 minuten een afwijking van maximaal ~200 us vertoont — en zet de PTP van elke camera terug zoals deze werd aangetroffen. Er wordt niet opnieuw gewacht op een camera die nog steeds gesynchroniseerd is van een eerdere test. Vroeger liet het de PTP uitgeschakeld op een netwerk met één switch waar geen DAQ-E reageerde, en las het de vrijlopende klok van elke camera als een PTP-klok die niet zou synchroniseren (`PTP is not locked`, bij elke run). Zie [Multi-camera (tijdelijke sessies)](#multi-camera-transient-sessions).
> - **Een `.daq`-opname geeft aan of de kap is gekozen of verondersteld.** DAQ-U, DAQ-M en DAQ-E worden zonder kap geleverd, en Chloros kan niet zien of er een is aangebracht, dus een sensor die zonder benoemde kap is aangesloten, krijgt standaard `sunshine_cosine`, een *aanname*. Elke Chloros-opname (schema 1.30) slaat nu `cap_id_source` op naast `cap_id`: `operator` wanneer je een kap een naam hebt gegeven (inclusief `none`), `auto_default` wanneer het de standaardnaam was. Bij het verwerken van een opname waarvan de kap werd verondersteld, terwijl de DAQ-kap van het project op ‘Auto’ is ingesteld, geeft een `[daq-dls] WARNING` weer waarin het bestand wordt genoemd: een DAQ-U zonder correctie leest onder zonlicht ongeveer 30x te hoog. **Geef de door u aangepaste cap een naam** wanneer u verbinding maakt, of stel de DAQ-cap van het project in, en de waarschuwing verdwijnt.
> - **Geen pip-consolevensters meer op Windows.** Op een pc waarop Python was geïnstalleerd, voerden de backend en elk van de bijbehorende image workers bij het opstarten één keer `pip` uit, en elk opende een consolevenster. Dat doen ze nu niet meer.
> - **Live `.daq`-opnames overleven een stroomstoring.**Een opname wordt bij elke commit achter de sensor weggeschreven met een logboek en een volledige schijfsynchronisatie, zodat een trage schijf het bestand vertraagt, maar nooit de meetwaarden. Een opname die halverwege een commit is gestopt (stroomuitval, afsluiten) kan `<name>.daq-journal` ernaast achterlaten:**kopieer de twee samen.** Chloros zet het journaal terug bij de eerste keer dat het bestand wordt gelezen, zodat de opname opent met alles wat vóór de stop was vastgelegd (ongeveer de laatste 3 s gaan verloren); in 1.2.0 kon een stroomstoring op het verkeerde moment het hele bestand onleesbaar maken („database disk image is malformed“).
> - **Een opname op een bezette schijf zorgt er niet langer voor dat de sensor vastloopt.** `daq pool-record` zet metingen in de wachtrij en schrijft ze op een eigen thread; een meting die niet kon worden geschreven, wordt meegeteld (`daq pool-list` toont `N write error(s)` met de laatste fout) in plaats van de sensor te laten vastlopen (één schrijfbewerking duurde 8,4 s naast vier camera’s die TIFF’s opsloegen).
> - **`lattice multi-capture` zonder array verliest niet langer de opgeslagen frames wanneer een trigger time-out bereikt.** Een trigger van een camera die in time-out gaat (`SC_ERR_TIMEOUT -1011`, waargenomen bij vier camera’s op één 1 GbE-switch) beëindigt de run bij die framegroep: de reeds opgeslagen groepen worden weergegeven met `Saved:`, gevolgd door de foutmelding, exit `1`. In versie 1.2.0 was dit een HTTP-500-fout zonder lijst, hoewel de bestanden wel op de schijf stonden.
> - **Door software getriggerde camera’s laten niet langer om de andere trigger vallen.** Een camera in de softwaretriggermodus zet nu een trigger die tijdens het uitlezen binnenkomt in de wachtrij (of Chloros wacht tot deze gereed is), zoals camera’s met kabelsynchronisatie dat al deden.
> - **`daq pool-discover` benoemt een DAQ-E-S `DAQ-E-S`** in de kolom `MODEL` (en die van `--json` in de kolom `model`). Voor elke Ethernet-sensor stond er `DAQ-E`; een sensor die niet binnen ongeveer 2 s op de identiteitscontrole reageert, wordt nog steeds vermeld als `DAQ-E`.
> - **Verbeterde LATTICE-indexproducten** (NDVI en andere indices) vanaf `process`, in elke verwerkingsmodus.
> - **OCN-camera’s: verbeterde stralings-, reflectie- en indexproducten bij zonlicht.** Na de bandontmenging wordt een vlootcorrectie toegepast; RGN-camera’s blijven ongewijzigd, en OCN-waarden wijken af van die in versie 1.2.0. Dit geldt ook voor een OCN-opname die met een eerdere versie is gemaakt, wanneer deze wordt verwerkt op een computer waarop de kalibratie staat waarmee de opname is gemaakt (Chloros behoudt de kalibratie van een camera zodra deze is aangesloten).
> - **`lattice continuous` registreert het ruwe beeld.** Een `tiff`-run schreef gedemosaïceerde 3-vlakframes geschreven, waarmee `process` niet kan beginnen. De frames zijn nu het `raw`-bestand dat een array-opname schrijft: één mozaïekvlak met de eigen waarden van de sensor, ongecomprimeerd, met het kalibratiepakket. Een script dat drie vlakken uit deze bestanden leest, leest er één.
>
> - **Elke opname slaat de DAQ-waarde op die erbij is gebruikt, net zoals de GUI dat doet.** `lattice capture`, `lattice continuous` (één `.daq` per frame), `array-capture` en `multi-capture` met een aangesloten array (en de SDK) schrijven de `.daq` en de bijbehorende `.match.json` wanneer een DAQ wordt gekoppeld of in een pool wordt opgenomen, inclusief een opname die uitsluitend uit ruwe gegevens bestaat; in 1.2.0 werd bij het opslaan van een ‘raw’ of ‘debayered’ opname slechts één `--force-daq` opgeslagen. `--no-daq` schakelt dit uit, `--daq-tolerance-ms MS` stelt de overeenkomsttolerantie voor die opname in, en het nieuwe `daq pool-bind` koppelt een camera of een array aan een DAQ vanuit de CLI. Een opname die met `.daq` wordt opgeslagen, wordt in `process` alleen gekoppeld aan de camera’s die erin worden genoemd. Een opname wacht nu op de volgende meting van de DAQ, zodat deze de metingen aan weerszijden van de belichting kan samenvoegen. Zie [De DAQ-meting die een opname opslaat](#the-daq-reading-a-capture-saves) en [Een DAQ-meting ver van de belichting](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** voegt de index toe aan het project voor de banden van elke camera, inclusief een nieuw project. In 1.2.0 werd een naam die nog niet in het project stond, genegeerd met `Unknown index name(s) ignored`.
> - **`process` naast een live-array:**het herstart niet langer een backend die camera’s of DAQ’s bevat, en wacht maximaal 15 s op een run die ten einde loopt.**Ctrl+C** stopt de run in de backend, niet alleen in de CLI. Zie [`process` Opmerkingen](#notes).
> - **Een projectmap die te diep ligt voor de limiet van 260 tekens voor paden in Windows** wordt direct geweigerd met de langste map die wel past, in plaats van een run die niets schrijft. Het Windows-installatieprogramma biedt nu de optie „Windows-lange paden inschakelen” en een uitsluiting voor Microsoft Defender voor de backend; beide zijn standaard aangevinkt. Zie [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** voert de eigen continue vastlegging van de backend uit: elke 2 seconden een voortgangsbalk, een overzicht in plaats van een lijst per bestand, en een afsluitcode anders dan nul bij elk bestand dat niet is geschreven of bij een volle schijf. Zie [`array-capture`-modi](#array-capture-modes).
> - **De eerste `array-capture` na `array-connect`** wacht maximaal 20 s op de eerste framegroep van de array in plaats van te mislukken met HTTP-statuscode 503 &quot;Array still warming up&quot;.
> - **Belichting van een array-element binnen een ROI:** een kleine ROI beperkt de belichting van een array-element niet langer tot de vrije loopperiode van de sensor (4,9 ms bij 768 × 304). Zie [Belichting op een array-element](#exposure-on-an-array-member).
> - **Automatische belichting verlaagt de automatische framesnelheid van een array niet langer.** Een via kabel gesynchroniseerde array draait op de snelheid die het netwerk en de uitleescapaciteit van de camera’s toestaan; automatische belichting vult de periode die deze snelheid biedt, en verhoogt vervolgens de versterking. Een handmatige belichting die langer duurt dan de periode (`lattice exposure --set`, `configure --set`) vertraagt de trigger om deze aan te passen in plaats van te worden geweigerd, en de snelheid keert terug wanneer het element weer op automatisch staat. Zie [Belichting op een array-lid](#exposure-on-an-array-member).
> - **Arrays leveren minder onvolledige framegroepen** op, en er treden minder 503-fouten op („Geen nieuwe gesynchroniseerde framegroep”): de trigger-input van elke slavecamera is nu ontdaan van glitches.
> - **Gecomprimeerd 12-bits transport** (`--packed`, standaard `auto`) verhoogt de beeldsnelheden van arrays met volledige sensor met ongeveer een derde. `array-connect --packet-size` wordt nu toegepast (versie 1.2.0 accepteerde het maar negeerde het). Zie [Gecomprimeerd 12-bits transport](#packed-12-bit-transport---packed).
> - **Een array waarvan de camera’s na een stroomonderbreking van de switch vanzelf weer opstarten.** Na een stroomonderbreking van de switch opent de array van `array-connect`de camera’s opnieuw wanneer ze reageren en begint opnieuw te streamen zodra PTP weer is ingeschakeld; in 1.2.0 stopte het definitief en was er een nieuwe `array-connect` nodig. `array-status` rapporteert `ptp_enabled: False` (en `ptp_off:`) terwijl een camera PTP als uitgeschakeld detecteert, terwijl het voorheen de instelling rapporteerde waarmee de array was verbonden. Zie [Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` neemt op zonder dat de GUI is geopend.** Als de gecombineerde weergave van de GUI is gesloten, geeft de backend de weergave zelf weer: deNDVI of de index die je doorgeeft met de nieuwe `--expr`. Een opname die geen frame schrijft, mislukt nu (`no-frames: …`, afsluiten `1`). Voorheen werd er alleen opgenomen zolang de gecombineerde weergave van de GUI open was, en een opname die niets schreef, sloot `0` nog steeds af. Zie [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Een camera die Chloros al in bezit heeft, wordt niet nogmaals geopend.** Een commando dat deze zelf zou openen, geeft in plaats daarvan een HTTP-status 409 `CAMERA_HELD` terug, waarbij wordt vermeld wie de camera in bezit heeft en welk commando deze vrijgeeft. Dat geldt ook voor een camera die op dit moment door een ander commando wordt gebruikt (een `multi-test` of `multi-capture` die zijn camera’s opent, een `lattice capture`): het tweede commando wordt geweigerd totdat het eerste is voltooid. Het opnieuw openen ervan zou de stream van de houder kunnen stoppen of de backend kunnen laten crashen. Zie [Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Tijdelijke multi-camera-opdrachten leveren elke camera op of mislukken.** `lattice multi-capture` zonder aangesloten array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` en `index --live` verlaten `1` met vermelding van een camera die geen frame heeft geleverd. De streams slaan een framegroep over waarin een camera ontbreekt en stoppen na drie opeenvolgende; `multi-test` drukt een rij af per trigger, `(no frame)` waar een camera het gemist heeft; `align-stream` sluit ook X af bij een taakfout. Elke run zet vervolgens elke camera terug zoals deze werd aangetroffen: de bedrading van de trigger en de synchronisatielijn, de PTP (`PtpEnable`) en alle camera-instellingen die tijdens de run zijn gewijzigd — belichting en versterking, AE-doel, limieten en demping, witbalans, zwartniveau, pixelformaat, binning en ROI, spiegelingen, framesnelheid — tenzij `--keep-settings` / `--clear-settings` (zie [Camerageheugen](#camera-memory----keep-settings----clear-settings)). Voorheen bewaarden deze commando’s alle frames die binnenkwamen en verlieten ze `0` (`align-stream` gaf een taakfout weer als `Note:`), en lieten elke camera achter in `TriggerMode` `On`. Ze stelden ook de belichting van elke camera in op basis van de scène vóór het opslaan van een beeld (hooguit een paar seconden), op de doelwaarde waarmee een aangesloten array elke camera start: 40 multispectraal, 60 RGB (`--brightness N` stelt dit in), en houdt deze instelling aan voor de hele run; hetzelfde geldt voor een enkele `lattice capture` van een camera die Chloros niet vasthoudt. Voorheen werd het eerste frame van elke camera opgenomen met de eigen belichtingslimiet van de camera, ongeveer 100 ms, ongeacht de lichtomstandigheden, en werd het vaak afgekapt. Zie [Multi-camera (tijdelijke sessies)](#multi-camera-transient-sessions).
> - **Tijdelijke multi-camera-opdrachten delen het netwerk op dezelfde manier als `array-connect` dat doet, en openen alleen `--serials`.** Elke camera wordt afgestemd op zijn aandeel in de hostverbinding, zodat vier camera’s op een 1 Gbps-verbinding draaien; voorheen vroegen de vier samen om 125 MB/s, allemaal via 1 GbE, en ging er bij elk frame pakketten verloren. Een array die het netwerk niet aankan, wordt geweigerd voordat er camera&#x27;s gaan streamen. `--serials` kiest nu de camera&#x27;s, waarbij de eerste de trigger activeert; voorheen opende elke opdracht alle camera&#x27;s op het netwerk, ongeacht wat `--serials` aangaf. Een storing die het gevolg is van GVSP-pakketverlies geeft dit nu aan in plaats van de M8-synchronisatiekabel aan te wijzen. Zie [Multi-camera (tijdelijke sessies)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` laat de camera’s vrij draaien, zoals aangetroffen.** `TriggerMode` `Off`, en al het andere zoals de connectie het heeft aangetroffen — de instellingen voor de trigger en de synchronisatielijn (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP en de camera-instellingen die tijdens de sessie zijn gewijzigd — zodat een enkele-cameracommando op een voormalig lid haalt alle frames in één keer op, en de master blijft niet langer op `TriggerSource` `Software` staan. `--keep-settings` slaat de instellingen van de sessie op in het geheugen van elke camera, `--clear-settings` reset dat geheugen (zie [Camerageheugen](#camera-memory----keep-settings----clear-settings)). `lattice continuous` stelt ook een camera die het in de ‘armed’-stand aantreft in op ‘free-run’ voor de run (en zet deze daarna weer terug), en een run zonder frame verlaat `1`.
> - **`lattice configure --get` / `lattice exposure`** geven een `note:` weer met dewerkelijke belichting of versterking weer, terwijl de automatische modus actief is. Een geweigerde `lattice exposure --set` / `configure --set` geeft het bereik aan dat de camera accepteert en zet de automatische modus weer aan in plaats van de automatische belichting uit te schakelen. Zie [Sensorafstemming](#sensor-tuning).
> - **Nieuwe camerabedieningen:**`lattice highlights` (helderwitte oppervlakken in de RGB-weergave naar beneden halen) en `lattice exposure --brightness-auto`.**`lattice multi-detect`** detecteert de bekabelde camera’s in één scanronde en stelt de master standaard in op de RGB-camera.
> - **Nieuwe DAQ-opdrachten:** `daq pool-bind`; `pool-enable-ptp`; en voor de eigen instellingen van een DAQ-E: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` en `pool-update-firmware`. Zie [DAQ-E-apparaatinstellingen](#daq-e-device-settings).
> - **Een DAQ-E die geen gegevens meer levert, wordt door de backend opnieuw verbonden.** Wanneer de stream van een DAQ-E in de pool stilvalt (bijvoorbeeld door het uit- en weer inschakelen van de switch die hem van stroom voorziet, een verbroken cameralink), maakt de backend opnieuw verbinding met de eerdere instellingen en probeert het met steeds langere tussenpozen opnieuw zolang het apparaat niet beschikbaar is. Een opname daarop wordt voortgezet in hetzelfde `.daq`-bestand, waarbij de onderbreking in het bestand wordt gemarkeerd, en `pool-record --stop` geeft een overzicht van elke onderbreking. In versie 1.2.0 bleef de sensor stil tot `pool-disconnect` + `pool-connect`, waardoor ook de opname werd beëindigd. Zie [Een DAQ-E die stopt met leveren](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** geeft aan of de pakketten van de PTP-grandmaster het netwerk bereiken (`health`); als dat niet het geval is, wordt er een `WARNING:` op stderr weergegeven. Zie [`time-sync`](#chloros-cli-time-sync).
> - **Linux .deb:** dit vergroot de socket-ontvangstbuffers van de kernel die camerastreams nodig hebben (bij de standaardinstelling van de kernel gaan er frames en framesnelheid verloren), voegt een backend-service toe die als uw gebruiker draait, en `lattice network` controleert de buffers. De CLI-interface stopt nooit een backend die als service draait, en bij een upgrade worden de ingeschakelde backend-services opnieuw gestart. Zie [De backend als service uitvoeren](#running-the-backend-as-a-service-headless) en [Linux-hosts — socket-ontvangstbuffers](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` geeft aan wanneer uitlijning niet mogelijk was:**elke reden waarom een opname niet-uitgelijnd werd opgeslagen (geen uitlijningsprofiel, leden met verschillende resoluties, een array die niet via de kabel is gesynchroniseerd, een frame waarop het profiel niet kan worden toegewezen) wordt weergegeven als een `Alignment:`-regel.**`chloros-cli daq help`** is `daq --help` (het is afgesloten met `2`), en op Linux vermeldt de helptekst de directe hardware-opdrachten `daq` die in de build zijn opgenomen.
> - **Jetson:** het pakket is `chloros_1.3.0_arm64_jp6.deb`, voor JetPack 6, en de GPU-verwerking ervan draait op de Orin met JetPack’s CUDA PyTorch. Zie [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Conventies

- Alle commando’s hebben het voorvoegsel `chloros-cli`. Op Windows is het binaire bestand `chloros-cli.exe`; op Linux/Jetson is het `chloros-cli`.
- Optionele argumenten worden weergegeven als `--flag`. Verplichte positionele argumenten worden zonder haakjes weergegeven.
- Wanneer er een standaardwaarde is opgegeven, wordt die waarde gebruikt als de vlag wordt weggelaten.
- De CLI is een ‘thin’ HTTP-client via de Chloros-backend (de server op `127.0.0.1:5000` die ook door de desktop-app wordt gebruikt). **Welke commando’s starten het:**
  - De kernopdrachten — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` en `selftest` — maken gebruik van een actieve backend, of starten er een wanneer er geen actief is, en stoppen deze weer wanneer ze klaar zijn.
  - **De commando’s `lattice`, `project` en `daq` starten er nooit een.** Ze hebben een backend nodig die al draait: die van de Windows-desktopapp, de Linux [backend-service](#running-the-backend-as-a-service-headless), of een die je zelf start (zie [De backend starten](#start-the-backend)). Zonder backend geven ze `Error: Chloros backend not reachable at http://127.0.0.1:5000` weer, samen met het commando waarmee deze wordt gestart, en worden ze afgesloten met `1`.
- `CHLOROS_BACKEND_URL=<url>` verwijst de **`lattice`**,**`project`**en**`daq pool-*`** naar een andere backend (`lattice network-analysis` verwijst in plaats daarvan naar `--backend-url`). De kernopdrachten koppelen `http://127.0.0.1:<port>` opzettelijk vast en negeren het (de IPv4-letterlijke waarde voorkomt de ~2 s per verzoek-vertraging van Windows bij `localhost`→`::1`). Zie [Omgevingsvariabelen](#environment-variables).
- **Relatieve paden worden omgezet ten opzichte van de map van waaruit je `chloros-cli` uitvoert.** De backend is het proces dat de uitvoer- en invoermappen opent, dus voordat een pad wordt verzonden, maakt de CLI het absoluut: `-o` / `--output` (inclusief de standaardinstelling, `output`), `--input`, `--burst-dir`, `--profile`, `--save` en `--mask`, een `output` (standaard `output`) van een [recept](#recipe-dsl)-stap, en de standaardmap `output` van `lattice index --live`. `-o captures/` schrijft daarom naar `<current folder>/captures/`, en de afgedrukte paden zijn absoluut. `~` wordt uitgewerkt. Bij een externe backend (`CHLOROS_BACKEND_URL` die naar een andere machine verwijst) worden paden verzonden zoals ze zijn ingevoerd en worden ze op die machine omgezet.
- Een Chloros+-account is vereist voor alle SDK/CLI aanroepen (voer `chloros-cli login` één keer per machine uit; wordt in de cache opgeslagen in `~/.chloros/`).
- In de voorbeelden worden Linux-paden gebruikt; vervang op Windows `/home/user/...` door `C:/Users/.../...`. De seriële poort van een DAQ-U&#x27;seriële poort heeft de vorm `COM3` op Windows en `/dev/ttyUSB0` / `/dev/ttyACM0` op Linux.
- Synopsissen schrijven wederzijds exclusieve vlaggen als `{--a | --b}` en optionele vlaggen als `[...]`.

---

## Synopsis op het hoogste niveau

```
chloros-cli [global options] COMMAND [command options]
```

### Algemene opties

| Vlag | Beschrijving |
| --- | --- |
| `--backend-exe PATH` | Het automatisch gedetecteerde backend-uitvoerbare bestand overschrijven. |
| `--port N` | HTTP-poort van de backend (standaard: `5000`). Wordt alleen gebruikt door de kernopdrachten (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` en `daq` gebruiken `CHLOROS_BACKEND_URL` (standaard `http://127.0.0.1:5000`). |
| `-v, --verbose` | Gedetailleerde uitvoer inschakelen. |
| `--restart` | De backend geforceerd opnieuw opstarten (beëindigt alle actieve `backend_server.py`-processen). Een backend die door een systemd-service (Linux .deb) wordt nooit gestopt: het commando geeft in plaats daarvan `sudo systemctl restart <unit>` weer en wordt afgesloten met `1`. |
| `--version` | Versie weergeven (`Chloros CLI 1.3.0`). |
| `--help` | Help op het hoogste niveau weergeven. |

### Overzicht van commando’s

| Commando | Doel |
| --- | --- |
| [`process`](#chloros-cli-process) | Verwerk een map met Survey3- of LATTICE-opnames van begin tot eind. |
| [`login`](#chloros-cli-login) | Deze machine authenticeren met een Chloros+-account. |
| [`logout`](#chloros-cli-logout) | Opgeslagen inloggegevens wissen. |
| [`status`](#chloros-cli-status) | De huidige licentie- en authenticatiestatus weergeven. |
| [`export-status`](#chloros-cli-export-status) | Live voortgang van Thread-4-export tijdens een `process`-run. |
| [`language`](#chloros-cli-language) | De weergavetaal van de CLI instellen of weergeven (38 ondersteund). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standaard projectmap (gedeeld met de GUI). |
| [`update`](#chloros-cli-update) | Controleren op en installeren van Chloros-updates (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systeemdiagnostiek + rooktests. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-status / -besturing. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-camerabesturing en -opname (meer dan 45 subcommando&#x27;s). |
| [`daq`](#chloros-cli-daq) | DAQ-spectraalsensorbesturing (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Een opgeslagen Chloros-project (camera&#x27;s + DAQ&#x27;s) openen en uitvoeren. |

---

## Installatie

`chloros-cli` wordt meegeleverd in het Chloros-desktopinstallatieprogramma op elk ondersteund platform — er is geen aparte CLI-download. Door het platformpakket te installeren, wordt `chloros-cli` toegevoegd aan uw `PATH`, naast de desktop-app en het backend-programma dat deze aanstuurt.

Nieuwste downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Het installatieprogramma bevat ook handige start-scripts (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) die een gebruiksklare CLI-shell openen.

### Windows (.exe)

1. Download het Windows-installatieprogramma vanaf de downloadpagina.
2. Voer `Chloros Installer 1.3.0.exe` uit en volg de wizard. Het standaardinstallatiepad is `C:\Program Files\Chloros\` (de CLI-omgeving wordt geïnstalleerd in `C:\Program Files\Chloros\cli\`, dat door het installatieprogramma aan de PATH wordt toegevoegd).
3. Open een nieuwe terminal (`cmd.exe`, PowerShell of Windows-terminal) zodat de bijgewerkte `PATH` wordt gedetecteerd.

```powershell
chloros-cli --version
```

Het installatieprogramma voegt automatisch `chloros-cli.exe` toe aan uw systeem `PATH` en bundelt de LATTICE-camerastuurprogramma’s. De backend is `C:\Program Files\Chloros\backend\chloros-backend.exe` (de map `backend` in je installatiemap), die niet op `PATH` staat.

**Systeeminstellingen.** De pagina *Select Additional Tasks* van het installatieprogramma bevat twee opties onder **Systeeminstellingen**, die beide**standaard zijn aangevinkt**:

| Optie (zoals geformuleerd in het installatieprogramma) | Wat het doet | Afzien van stille installatie |
| --- | --- | --- |
| **Windows-lange paden inschakelen** (zorgt ervoor dat projectmappen dieper dan 260 tekens kunnen liggen; aanbevolen) | Stelt de systeembrede instelling `LongPathsEnabled` van Windows in op 1. Windows wordt geleverd met deze instelling uitgeschakeld, waardoor geen enkel pad langer mag zijn dan 260 tekens; Chloros-producten bevinden zich tot ongeveer 130 tekens onder de projectmap, dus een dieper gelegen projectmap wordt geweigerd (zie [Probleemoplossing](#troubleshooting-pointers)). Wordt van kracht voor programma’s die na de installatie worden gestart. Blijft ingeschakeld bij het verwijderen, omdat andere programma’s er mogelijk op vertrouwen. | `/MERGETASKS="!longpaths"` |
| **Sluit de Chloros-backend uit van Microsoft Defender-scans**(veel snellere camerabeelden; aanbevolen) | Voegt een Defender**proces**-uitsluiting toe voor `chloros-backend.exe` in de installatiemap, zodat realtime bescherming niet elk bestand scant dat de backend schrijft, waar het ook wordt geschreven. Er wordt geen ander programma uitgesloten; de uitsluiting wordt bij het verwijderen ongedaan gemaakt. Als Defender ontbreekt of door beleid is geblokkeerd, mislukt de stap zonder melding en gaat de installatie door. | `/MERGETASKS="!defenderexclusion"` |**Waarom de Defender-optie belangrijk is:** bij hoge opnamesnelheden kost het realtime scannen van de opnamebestanden ongeveer 30 % van de opnamesnelheid (gemeten bij een continue opname met 9 camera’s: 77,7 opnames/s uitgesloten, 52,7 gescand). `lattice array-capture --continuous` geeft een melding weer wanneer het vaststelt dat Defender één of meer CPU-kernen gebruikt tijdens het opnemen; voer het installatieprogramma opnieuw uit met de optie aangevinkt.

Bij een stille installatie (`/SILENT` of `/VERYSILENT`) worden beide toegepast, tenzij dit wordt uitgeschakeld, bijvoorbeeld met `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Windows-firewall.** Het installatieprogramma voegt altijd twee inkomende regels toe voor `chloros-backend.exe`, in elk netwerkprofiel, en het verwijderprogramma verwijdert deze: `Chloros Backend (UDP-In)`, voor camerastreams, PTP, mDNS en de DAQ-E-datastroom, en `Chloros Backend (TCP-In)`, beperkt tot het lokale subnet, voor een DAQ-E-firmware-update, waarbij de sensor opnieuw verbinding maakt met deze computer om het beeld op te halen (zie [DAQ-E-apparaatinstellingen](#daq-e-device-settings)). De HTTP-API van de backend blijft gekoppeld aan `127.0.0.1`.

### Linux amd64 (.deb)

Voor Ubuntu 22.04 LTS of nieuwer / op Debian gebaseerde x86_64-werkstations.

> **Ubuntu 20.04 wordt niet ondersteund.** De lijst met afhankelijkheden van het pakket is afgeleid van
> de bibliotheken waaraan de backend daadwerkelijk is gekoppeld, waaronder `libc6 (>= 2.35)`;
> Focal wordt geleverd met glibc 2.31. `apt` weigert de installatie in plaats van deze te laten mislukken tijdens
> de uitvoering. De lijst noemt ook `libgl1`, dat door de meegeleverde OpenCV wordt geladen en dat in een minimale
> serverimage mogelijk ontbreekt; `apt install` voegt het toe.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Het .deb-pakket installeert:
- `chloros-cli` tot en met `/usr/bin/chloros-cli`, en `chloros-backend` (start de backend) tot `/usr/bin/chloros-backend`
- De gecompileerde backend en CLI, uitgepakt, in `/usr/lib/chloros/backend/` en `/usr/lib/chloros/cli/`
- De LATTICE-camerastuurprogramma’s
- Denoiser-modellen, kalibratiebundels en configuratie voor het updatekanaal
- De Python-SDK voor het systeem `python3` (`import chloros_sdk`). `apt install` brengt ook zijn eigen vereisten mee, namelijk `python3-numpy` en `python3-requests` (aanbevolen, maar niet verplicht: de bovenstaande programma’s zijn op zichzelf staand; `dpkg -i` slaat ze over, en de installatie-uitvoer geeft aan wat er ontbreekt). In een virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, dat de socket-ontvangstbuffers van de kernel voor camerastreams tot 32 MB vergroot en deze in één keer toepast (een waarde die al hoger is ingesteld, blijft ongewijzigd). Als je het pakket verwijdert, wordt het bestand verwijderd. Zie [Linux-hosts — socket-ontvangstbuffers](#linux-hosts--socket-receive-buffers).
- Twee optionele systemd-services voor de backend, die beide niet zijn ingeschakeld (volgende paragraaf)

#### De backend als service uitvoeren (headless)

Zonder een service draait de backend (en daarmee de PTP-grandmaster) alleen zolang iets deze heeft gestart: `chloros-backend` in een terminal, een SDK-script of een core-commando `chloros-cli` gedurende de looptijd ervan. Voor een altijd actieve backend op een headless-machine schakel je de service **als jouw gebruiker** in:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` voert de backend uit als die gebruiker. De opnames die het schrijft, zijn van die gebruiker; het gedetailleerde logboek staat in hun `~/.cache/chloros/logs`, en hij gebruikt hun `chloros-cli login` (`~/.chloros`), dus log eerst één keer in als die gebruiker. PTP heeft geen root-rechten nodig. Net als elke geïnstalleerde backend luistert hij alleen op `127.0.0.1:5000`.

`chloros-backend.service` draait de backend als **root**. Het is nog steeds zo geïnstalleerd dat een machine waarop het is ingeschakeld, blijft draaien, maar alles wat een root-backend schrijft, is eigendom van root, dus een gebruiker kan zijn eigen opnames niet verwijderen. Om over te schakelen naar je eigen gebruiker:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Het gedetailleerde logboek van de root-service staat in `/var/log/chloros` en is leesbaar voor leden van de groep `adm` (de service stelt `CHLOROS_LOG_DIR` in; zie [Omgevingsvariabelen](#environment-variables)). Bij het installeren van het pakket wordt het commando `enable` weergegeven, waarbij je gebruikersnaam is ingevuld, evenals de schakelaar wanneer de root-service is ingeschakeld.

**Schakel één backend-eenheid in.** Er draait slechts één backend tegelijk. Een tweede ingeschakelde unit (`chloros-backend.service` naast `chloros-backend@<user>.service`, of twee `@`-units van gebruikers) weigert te starten en blijft staan als `failed` met exitstatus 98; systemd start deze niet opnieuw op. De status van `systemctl status` geeft aan waarom: `Another instance is already running. Exiting.`, vervolgens wat de backend blokkeert (poort 5000, of het vergrendelingsbestand en de gebruiker aan wie het toebehoort), en vervolgens `systemd will not restart <unit> (exit status 98)`. Na een herstart kan elk van beide units de actieve zijn. Om te wisselen, schakel je de unit die je niet gebruikt uit, zoals hierboven beschreven.**De CLI stopt nooit de backend van een service.** `chloros-cli` gebruikt deze zoals hij is, net als de GUI en de SDK: `process` draait erop en laadt zijn eigen project in plaats van het laatste. Waar de CLI anders de backend opnieuw zou starten (`--restart`, of een `process` waarvan de import niets heeft gevonden), noemt het de unit, geeft `sudo systemctl restart <unit>` weer en sluit `1` af, omdat een backend die op die manier wordt gestopt, uitgeschakeld zou blijven. `GET /api/status` benoemt de unit als `service` (`null` voor een backend waarop geen service draait).**Upgrade en verwijdering.** Bij het upgraden van het pakket worden alle backend-services gestopt en vervolgens worden alle ingeschakelde services opnieuw gestart, waarbij de `@<user>`-eenheden als eerste worden gestart (een service die handmatig is gestart maar nooit is ingeschakeld, blijft gestopt, net als na een herstart). Als er meer dan één is ingeschakeld, draait er slechts één en blijven de andere zoals hierboven beschreven in de foutstatus; de upgrade geeft `Chloros: N backend units are enabled: …` en `Keep one enabled: sudo systemctl disable --now <unit>` weer. Verwijdering stopt en schakelt ze uit. Als je het volledig verwijdert, wordt ook `/var/log/chloros` verwijderd.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Dezelfde opzet als het amd64 .deb-bestand, inclusief ontvangstbuffers en [backend-services](#running-the-backend-as-a-service-headless), voor Jetson Orin / Orin NX / Orin Nano op **JetPack 6** (L4T R36). De PyTorch is de CUDA-build van JetPack 6 voor Orin, dus GPU-verwerking, inclusief de textuurgevoelige debayer, draait op de GPU van de Jetson. Het pakket bevat CUDA zelf niet: het maakt gebruik van de CUDA- en cuDNN-bibliotheken die JetPack 6 installeert (`sudo apt install nvidia-jetpack` op een image waar deze ontbreken). Als er geen CUDA-GPU beschikbaar is, vindt de verwerking plaats op de CPU en maakt de textuurbewuste verwerking gebruik van de standaard debayer (zie [`process`](#chloros-cli-process)).

### Eén keer authenticeren per machine

Elk platform vereist een eenmalige Chloros+-aanmelding voordat SDK- en CLI-aanroepen werken:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Inloggegevens worden opgeslagen in `~/.chloros/user_session.json`.

### De backend starten

De commando’s `lattice`, `project` en `daq` communiceren met een backend die al actief is; ze starten er nooit een (zie [Conventies](#conventions)). Zorg dat er een draait terwijl je ze gebruikt:

| Platform | Hoe |
| --- | --- |
| Windows | Open de Chloros-desktopapp (de backend daarvan bedient ook de CLI), of voer `"C:\Program Files\Chloros\backend\chloros-backend.exe"` uit in een eigen terminal (de map `backend` in je installatiemap) en laat deze openstaan. |
| Linux / Jetson | Voer `chloros-backend` uit in een aparte terminal, of laat er altijd één draaien met de [backend-service](#running-the-backend-as-a-service-headless). |

Er draait slechts één backend tegelijk; elke client (de GUI, de CLI, SDK-scripts) deelt deze. De kerncommando’s (`process`, `login`, `status`, …) maken er ook gebruik van en starten er een wanneer er geen actief is.

**Een backend blijft draaien tussen commando’s door**, zodat de volgende backend niet opnieuw hoeft op te starten. Het stopt wanneer je uitlogt (`chloros-cli logout`, tenzij de Chloros-app openstaat of er een run of een offline build bezig is), wanneer je `chloros-cli backend stop` uitvoert, of vanzelf na 30 minuten als er niets gebruik van maakt: geen commando, script of app, geen run of offline build, geen aangesloten camera of lichtsensor. `backend stop` weigert terwijl er een run of een offline build bezig is, en geeft de naam ervan; `--force` stopt het nu en die run of build gaat verloren (een burst wordt bewaard, en `array-build-video` bouwt deze opnieuw op). Stel `CHLOROS_BACKEND_IDLE_MIN` in voordat de backend start om de 30 minuten te wijzigen (`0`: nooit); de [backend-service](#running-the-backend-as-a-service-headless) stopt nooit vanzelf. `chloros-cli backend status` laat zien of er een draait, wat deze gebruikt en of er een offline build draait. `process` opent het project in de actieve backend, tenzij de Chloros-app al geopend is met een project dat heeft geopend: in dat geval weigert het, en moet je eerst vanuit de app verdergaan of dat project sluiten. Een project dat door een eerdere `process` of een SDK-script open is gelaten, wordt gewoon vervangen.

### Controleer de installatie

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` sluit `0` pas af als alle zeven controles zijn geslaagd. Voor de laatste is een CUDA-GPU nodig (een NVIDIA-GPU, of die van een Jetson Orin) en de denoiser-modellen nodig; op een computer zonder GPU wordt daarom `Failed: CUDA + Denoiser` weergegeven en wordt het programma afgesloten bij `1`; alles behalve de uitsluitend GPU-gebaseerde, textuurbewuste debayer werkt daar nog steeds.

> **Chloros+-abonnement vereist.**De CLI vereist een actief Chloros+-abonnement.**Copper**is het instapniveau van Chloros+ — elk betaald Chloros+-niveau biedt toegang tot de CLI en SDK; alleen het gratis**Iron**-niveau biedt dat niet. (Abonnements-ID-toewijzing: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Upgrade bij [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Deze limiet wordt afgedwongen door de backend, niet alleen door de CLI: een verzoek met SDK- of CLI-vlag zonder betaald abonnement wordt afgewezen met `403 PLAN_UPGRADE_REQUIRED`, ongeacht of het afkomstig is van `chloros-cli`, de Python-SDK of een zelfgeschreven HTTP-client. Een uitgelogde aanroeper krijgt in plaats daarvan de foutcode `401 AUTH_REQUIRED`. Toegang werkt offline gedurende de respijtperiode van het abonnement (30 dagen bij maandabonnementen, tot de vervaldatum bij jaarabonnementen) en stopt wanneer die verstrijkt; `chloros-cli status` blijft werken zodat de reden zichtbaar is (dit is de enige SDK-/CLI-route die is vrijgesteld van de tier-gate — `GET /api/license-status`).

---

## Snelstart in vijf minuten

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (stap 2) start een backend voor zijn eigen uitvoering wanneer er nog geen actief is en stopt deze wanneer deze klaar is, dus start er een voor stap 4 en verder. Zie [De backend starten](#start-the-backend).

---

## `chloros-cli process`

Verwerk een map met afbeeldingen via de volledige Chloros-pijplijn (doeldetectie → kalibratie → vignet → reflectantie → indexexport).

### Overzicht

```
chloros-cli process INPUT [OPTIONS]
```

### Positieargumenten

| Argument | Beschrijving |
| --- | --- |
| `INPUT` | Pad naar de invoermap die `.raw + .jpg`- (Survey3), `.tif/.tiff`- (LATTICE) of `.dng`-bestanden bevat. |

### Algemene opties

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `-o, --output PATH` | een nieuwe map met de naam `-n` in de standaardprojectmap | De projectmap. Het project wordt hiernaar vernoemd, en een map die al een `project.json` bevat, krijgt een `_1` / `_2`-broertje in plaats van overschreven te worden. De standaardprojectmap is degene die `set-project-folder` (of de GUI) heeft gekozen, anders `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (tijdstempel) | Projectnaam, alleen gebruikt **zonder** `-o` (met `-o` krijgt het project de naam van de map). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` gebruikt een Chloros+ neurale debayer voor Survey3-beelden; langzamer maar van hogere kwaliteit. Dit is uitsluitend voor Survey3: LATTICE-opnames maken altijd gebruik van de standaard demosaic, dus voor een map met LATTICE-opnames weigert `process` `texture-aware` na het importeren, met een foutmelding waarin wordt aangegeven hoeveel er zijn, en sluit `1` afsluit voordat er iets wordt verwerkt (de Chloros-app biedt hiervoor ook geen ‘Texture Aware’ aan). Er is een CUDA-GPU nodig (een NVIDIA-GPU op Windows / Linux x64, de GPU van de Orin op een Jetson): zonder deze gebruikt de run `standard`, en de backend registreert één regel met de melding waarom (weergegeven met `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Vignetteringscorrectie. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflectanciekalibratie (gebruikt paneeldoel indien gevonden, NIST-kalibratie per serienummer voor LATTICE). Voor LATTICE multispectraal fungeert dit tevens als de reflectantie **product** schakelaar — zie [Exportschakelaars per product](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | uit | Pas PPK GNSS-correcties toe uit sidecar-bestanden. |
| `--exposure-pin-1 CAMERA_MODEL` | geen | Het cameramodel dat is aangesloten op belichtingspin 1, voor PPK-tijdsynchronisatie. |
| `--exposure-pin-2 CAMERA_MODEL` | geen | Het cameramodel dat is aangesloten op belichtingspin 2, voor PPK-tijdsynchronisatie. |
| `--recal-interval SECONDS` | `0` | Minimaal herkalibratie-interval, in seconden (het *minimale herkalibratie-interval* van het project). |
| `--timezone-offset HOURS` | `0` | De tijdzone-afwijking van de lichtsensor ten opzichte van UTC, in uren (de *Tijdzone-afwijking van de lichtsensor* van het project). |
| `--format FORMAT` | `TIFF (16-bit)` | Een van `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros verhoogt nooit de bitdiepte van een afbeelding: bij volledig 8-bits afbeeldingen wordt een formaat met grotere bitdiepte opgeslagen als `TIFF (8-bit)` en de run vermeldt dit één keer. In een project waarin 8-bits en 12/16-bits opnames worden gecombineerd, wordt elke opname geëxporteerd met zijn eigen bitdiepte of een lagere. Radiance is altijd 32-bits float. |
| `--indices NAME [NAME ...]` | geen | Vegetatie-indexen op basis van vooringestelde naam (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), hoofdlettergevoelig. Elk wordt aan het project toegevoegd voor de banden van de betreffende camera, inclusief een nieuw project. Een naam die geen voorinstelling is, of die de banden van de camera niet kunnen vormen (`NDRE` vereist RE en NIR, die geen enkele camera afzonderlijk levert), wordt in een waarschuwing vermeld en krijgt geen indexafbeelding. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Dwing het startpunt van de pijplijn af voor LATTICE TIFF’s (dit heeft geen invloed op Survey3.raw). Ook de nooduitgang waarmee een opname zonder **raw**-bestand toch kan worden verwerkt — zie [Hoe een opnamemap eruitziet](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | aan | Geef het lineaire debayered-product weer (`Debayered_Images`). Zie [Exportschakelaars per product](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | aan | Geeft het weergavevoorbeeld weer (`Preview_Images`): RGB = witbalans (DAQ-lichtbron indien beschikbaar, anders grijze wereld) + gamma; multispec = valsekleuruitrekking. |
| `--radiance / --no-radiance` | aan | Geef float32-straling weer (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referentie voor het LATTICE-reflectantieproduct: `auto` = het QA-goedgekeurde in-frame-doel is de absolute referentie (een doel met een QA-vlag concurreert met de DAQ op basis van de gemeten score), DAQ-downwelling (ρ = π·L/E) als terugvaloptie; `target` = strikt (geen DAQ-vervanging); `daq` = DAQ-bepalend. Zie [Exportschakelaars per product](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | geen | Directory met per eenheid **gemeten** doelspectra voor reflectantie (`<serial>.csv`); valt bij een misser terug op de nominale T3/T4P-spectra. |
| `--array-alignment / --no-array-alignment` | aan | LATTICE-arrays: pas de module-naar-module-uitlijning, die in de `Chloros:Alignment*` XMP van elke opname is vastgelegd, toe op elk verwerkt product (debayered / preview / radiance / reflectance / index). Geen actie voor beelden zonder de tags. |
| `--array-alignment-crop / --no-array-alignment-crop` | bijsnijden | Snijd uitgelijnde exportbestanden bij tot het gemeenschappelijke overlappingsgebied van de array, zodat alle modules één voetafdruk delen; `--no-…` behoudt het volledige sensorcanvas (zwarte vulling buiten de bron). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Herbemonstering voor de uitlijningsvervorming. `nearest` behoudt de exacte bron-DN&#x27;s (geen vermenging van radiometrische waarden tussen pixels). |

### Opties voor doelherkenning

| Vlag | Beschrijving |
| --- | --- |
| `--min-target-size PIXELS` | Minimale grootte van het kalibratiedoel in pixels voor de detector. |
| `--target-clustering 0-100` | Percentage doelclustering (standaard `60`). |
| `--target / --targets` | Zoek alleen naar kalibratiedoelen in een `target`- of `targets`-submap van de invoer (sneller). |

### Voorbeelden

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Exportschakelaars per product (LATTICE multispectraal)

De LATTICE-verwerking splitst zich **in één doorloop**op in alle toepasselijke producten. De vier schakelaars per type — `--debayered`, `--preview`, `--radiance`, `--reflectance` — staan allemaal**standaard AAN**; gebruik het formulier `--no-<type>` om er één uit te schakelen. RGB-mastercamera’s geven uitsluitend gedebayerde beelden + preview weer (geen straling/reflectie per band), dus `--radiance`/`--reflectance` hebben voor hen geen effect. De schakelaars worden genegeerd voor Survey3 `.raw` (dat het standaardreflectantie-/doelpad volgt). *(De oude `--radiometric-output {reflectance,radiance,sensor-response}`-vlag is **verwijderd** en vervangen door deze schakelaars; er is geen `sensor-response`-niveau meer.)*

| Product | Uitvoer | DAQ-downwelling nodig? |
| --- | --- | --- |
| `--debayered` | Lineaire demosaic (`Debayered_Images`): hetzelfde bestand dat een opname opslaat op het `debayered`-niveau — de donkere offset (DSNU) van het kalibratiepakket wordt van de ruwe gegevens afgetrokken, gevolgd door de randgevoelige demosaic van OpenCV; geen lineariteits-LUT, flat field of unmix. | Nee. |
| `--preview` | Voorbeeldweergave (`Preview_Images`): RGB = witbalans + gamma; multispec = valsekleuruitrekking. | Nee. |
| `--radiance` | float32 W/m²/sr/nm afkomstig uit de volledige radiometrische keten (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflectie ρ (`32768` = 1,0), Pix4D-ready. | **Ja**, tenzij het wordt verankerd door een QA-goedgekeurd doel binnen het beeld (zie hieronder). |

`--reflectance-source` kiest de reflectantiereferentie:**`auto`**(standaard) maakt van een QA-goedgekeurd doel binnen het beeld de**absolute referentie**— de aan het doelpunt verankerde empirische lijnketens worden onderling vergeleken op buiten beschouwing gelaten panelen en de gemeten winnaar wordt toegepast, band voor band: een band die de DAQ niet kan kalibreren, of waarvan de DAQ-hellingsaanpassing mislukt, neemt de eigen lijn van het doelpunt (`ReflectanceElBandMethods` geeft aan welke) — een doel dat de kwaliteitscontrole doorstaat met een vlag (een weggelaten paneel, een defecte band) concurreert ook met de DAQ-scheidslijn op basis van die gemeten score, en een frame waaraan het verliest, wordt gemarkeerd met `ReflectanceChain=daq` — terugvallen op de DAQ-downwelling-grenswaarde (ρ = π·L/E) wanneer er geen doel aanwezig is of de kwaliteitscontrole mislukt;**`target`**is strikt (geen DAQ-vervanging);**`daq`**kiest voor het DAQ-bepalende gedrag. De doelgeometrie (ArUco / vaste ROI / strip) is afkomstig uit de doelconfiguratie van het project; `--target-reflectance-dir DIR` bevat per eenheid**gemeten** scans per eenheid vast (`<serial>.csv`), opgezocht aan de hand van het serienummer/de QR-code van de doeleenheid, met de nominale T3/T4P-spectra als back-up.

Het DAQ-reflectantiepad bepaalt automatisch de **tijdstempel-gematchte neerwaartse straling**op basis van een geregistreerde**`.daq`**(DAQ-U/M/E)**of een DAQ-M-eigen `.csv`**die naast de beelden is aangetroffen. Als er geen kalibratiebundel per camera of DAQ lokaal in de cache is opgeslagen, haalt de pijplijn**deze automatisch op uit AWS** bij het eerste gebruik (vereist eenmalig internet; opgeslagen onder `~/.chloros/`).**Van wie is deze meting?**Een `.daq` die bij een opname wordt opgeslagen, is de meting van de camera(&#x27;s) die erin wordt beschreven — genoemd in een `<file>.daq.cameras.json`-sidecar ernaast (`lattice capture`, `multi-capture`, `array-capture` en de GUI; een onbewerkte burst noemt de meting van elk lid in zijn manifest) — en `process` koppelt deze**alleen aan beelden van die camera’s**. Een op zichzelf staande camera zonder eigen meting maakt daarom geen gebruik meer van de DAQ van een andere camera; koppel er een DAQ aan, of maak er een opname mee. De eigen opname van een DAQ (`daq pool-record`, of een opname die in de GUI is gestart), of elke `.daq` zonder die sidecar (inclusief oudere opnames), dient nog steeds voor elke camera.

#### Reflectantiepixels uitlezen (Pix4D / Metashape / je eigen scripts)

Reflectantie wordt opgeslagen als een geheel getal DN, en **de DN die ρ = 1,0 betekent, is afhankelijk van de broncamera**:

| Bron | ρ = 1,0 is | Hoe te herkennen |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (speling tot ρ 2,0) | XMP `Chloros:PixelScale=32768` staat in het bestand vermeld. |
| Survey3 | `65535` (afgekapt bij ρ 1,0) | Geen `Chloros:*` XMP-tags — die afwezigheid *is* het signaal. |

**Lees `Chloros:PixelScale` en deel erdoor** in plaats van uit te gaan van een constante. De tag is gedefinieerd in het uint16-domein, dus hij blijft `32768` in alle uitvoerformaten die een schaalverandering ondergaan — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` en `TIFF (32-bit, Percent)` zijn allemaal zelfbeschrijvend (normaliseer het opgeslagen dtype eerst terug naar uint16: ×257 vanaf 8-bit, ×65535 vanaf float).

> **Bestanden uit oudere builds bevatten mogelijk geen schaal.** Een 8-bits reflectantie-TIFF-bestand wordt opnieuw geschaald en krijgt net als de andere de naam `Chloros:PixelScale`. Builds vóór 1.3.0 *knipten* deze in plaats daarvan af op 0..255, zodat elke waarde boven ρ≈0,008 werd afgevlakt tot 255, en ze lieten zowel de `Chloros:PixelScale`- als de `MicaSense:RadiometricCalibration`-tuple weg. **Als de tag ontbreekt in een LATTICE-reflectantiebestand, ga dan niet uit van een schaal — verwerk het opnieuw** in plaats van pixels te delen die nooit deelbaar waren.

#### Indexpixels lezen (de NDVI- en andere vegetatie-indexen)

**Een raster met gehele getallen is een contrastuitrekking per afbeelding, geen −1…1-kaart.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` en `JPG (8-bit)` indexproducten — inclusief het 8-bits TIFF-bestand waarin een 8-bits opname wordt opgeslagen — slaan

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

waarbij `IndexMin` / `IndexMax` **de laagste en hoogste eindige indexwaarden van dat frame** zijn. DN 0 is niet −1, en de volledige schaal is niet +1.

De Chloros-stempels worden op beide eindpunten aangebracht, zodat de toewijzing omkeerbaar blijft:

| Tag | Betekenis |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — de onderstaande conventie |
| `Chloros:IndexMin` | de indexwaarde die is opgeslagen als DN 0 |
| `Chloros:IndexMax` | de indexwaarde die is opgeslagen als volledige schaal |

Lees ze en keer ze om, waarbij je de opgeslagen DN eerst normaliseert naar 0…1 op basis van de maximale waarde van het opgeslagen dtype, zodat één uitdrukking elk geheel getalformaat dekt:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Lezen vereist geen `-config pix4d.config` — exiftool leest deze XMP-eigenschappen zelfstandig. De configuratie is alleen nodig om ze te *schrijven*.

> **`IndexMin == IndexMax`** betekent dat elke pixel van dat frame dezelfde indexwaarde had. Het raster is op halve schaal gevuld en er is niets om door te delen — neem het eindpunt.

> **Geen `Chloros:Index*`-tags betekent dat er geen toewijzing is opgegeven.** Een LUT-export is een ingekleurde weergave waarvan de pixels verloopkleuren zijn die zijn gekoppeld aan je LUT-drempelvenster, en een frame zonder pixel met een eindige index heeft geen bereik om op te geven. Ga in geen van beide gevallen uit van een toewijzing — dezelfde regel geldt als bij een ontbrekende `PixelScale`.**Omdat de uitrekking per afbeelding geldt, zijn twee frames van één vlucht niet vergelijkbaar als DN**— elk frame heeft zijn eigen eindpunten, dus dezelfde NDVI komt in verschillende afbeeldingen op verschillende DN-waarden terecht. Voor kwantitatief werk over een vlucht heen (zonale statistieken, vaste drempels, tijdreeksen), exporteer**`TIFF (32-bit, Percent)`**: indexrasters worden daar opgeslagen als float32**indexwaarden zelf**, zonder dat herschaling of tags nodig zijn. Ondanks de naam van het formaat zijn indexproducten niet ×100 — die schaalverdeling geldt alleen voor reflectantie met het gegevenstype integer.

#### Unmix-clampmasker (op welke pixels een band is geklemd)

De 3×3 spectrale unmix kan een band negatief maken — wanneer het licht die band nauwelijks bevat, of wanneer de scène buiten het bereik valt dat door de unmix-matrix van de camera wordt beschreven. Dat is geen echte waarde, dus de export zet deze vast op 0 — en de liveweergave toont dezelfde 0. Een band kan op enkele pixels of op bijna alle pixels worden geklemd; het masker registreert beide. `Chloros:Band{R,G,B}UnmixClampFraction` geeft aan op welk deel van een frame dit is gebeurd; het clampmasker geeft aan **welke pixels**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— in een `clamp_masks/`-map**naast** de map van het product, nooit erin (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; een export van `process`: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — voor elke straling, reflectantie- en indexproduct waarin ten minste één pixel is geklemd: zowel opnames als `process`-exporten (en de TIFF-bestanden per frame van `array-build-video --save-tiffs`). Enkelbandige `uint8`, op het eigen pixelraster van het product (inclusief oriëntatie, ROI, binning en array-uitlijning). Elk masker vermeldt de cameratags van het product (`Make` / `Model` / serienummer), de opnametijd en de lenstags, en geeft een overzicht van de banden waarvoor de bits staan (`XMP-Camera:BandName`, bit 0 eerst); de bits `ImageDescription` en `Software` duiden het aan als een clamp-masker, en het bevat geen kalibratie.
- **Bits**: bit 0 = de band in het R-kanaal van het product (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Een ingestelde bit betekent dat de waarde van die pixel in die band is begrensd op 0 — beschouw deze, en elke index die hierop is gebaseerd, als onbetrouwbaar.
- **`Chloros:UnmixClampMask`** wordt op elk product aangebracht waarop de unmix is uitgevoerd: het pad van het masker ten opzichte van de eigen map van het product (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, altijd gescheiden door `/` — voeg dit samen met de map van de afbeelding om het masker te openen), of `none` wanneer er geen pixel is begrensd. Geen tag betekent dat het product nooit de unmix-bewerking heeft ondergaan (RGB- of monochrome camera’s, raw/debayered niveaus).
- `lattice capture` / `array-capture` geven het pad van het masker weer onder het opgeslagen bestand (`clamp mask: …`), en in de lijst met opnameresultaten wordt het vermeld als de `clamp_mask` van het frame.
- `lattice index --live --save-multiband` schrijft één masker per broncamera voor de multiband-stack — zie [`lattice index` — multiband-stackmaskers](#lattice-index--multiband-stack-masks). Een video heeft geen masker: `array-build-video` rapporteert in plaats daarvan de clamp-fracties van elk frame (zie [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Omdat geen enkele productmap, op welke diepte dan ook, een masker bevat, kan een productmap (`radiance/`, `reflectance/`, de map `*_Images/` van een `process`-export) in zijn geheel worden doorgegeven aan Pix4D, Metashape of andere fotogrammetriesoftware — inclusief submappen — en deze software verwerkt alleen afbeeldingen. Geef de productmap door, niet de map erboven (die bevat ook `clamp_masks/` en alle andere exporttypes). De eigen importfunctie van Chloros slaat de maskers over. Een masker lezen: `(mask & 1) != 0` betekent „het R-kanaal is hier begrensd“, `& 2` G, `& 4` B.

#### EXIF-gegevens worden meegenomen in de export

`process` kopieert het **GPS-blok en de ExifIFD** van de bronopname naar elk product, dus een
export bevat `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` en
`CameraSerialNumber` naast de georeferentie.

**`FocalLength` is niet optioneel voor fotogrammetrie.** Pix4D berekent de grondresolutie (GSD) op basis van
brandpuntsafstand plus hoogte; als de tag ontbreekt, valt het programma terug op een volkomen verkeerde schaal. Bij een
vlucht boven een sinaasappelboomgaard met 49 opnames veranderde de ontbrekende tag een terrein van 411 m × 160 m in een gereconstrueerd
47,8 km × 13 km — een ortho van 455 MP die grotendeels uit ‘nodata’ bestond, wat vervolgens werd geïnterpreteerd als een tegelingprobleem en
een BigTIFF-probleem voordat iemand de GSD had gecontroleerd. Als je ortho op een onwaarschijnlijke
schaal uitkomt, voer dan eerst `exiftool -FocalLength` uit op het geëxporteerde product.

De kopie is bewust **niet** `-all:all`: de structurele tags van IFD0&#x27;s structurele tags verstoren de LATTICE-uitvoer wanneer
ze worden gekopieerd, en `ExifImageWidth` / `ExifImageHeight` zijn uitgesloten omdat ze de
*bron*-opname beschrijven — een export waarvan de afmetingen ooit zijn aangepast, zou anders afmetingen bevatten
die in tegenspraak zijn met het eigen raster. XMP wordt rechtstreeks geschreven in plaats van gekopieerd, omdat ExifTool
XMP-tags van dezelfde aanroep verwijdert wanneer het XMP-blok wordt gekopieerd (waardoor de MAPIR-
kalibratietags zouden verdwijnen).

### Waar de uitvoer terechtkomt

De outputbestanden worden **in de projectmap opgeslagen, gegroepeerd per camera en vervolgens per bestandsformaat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

De cameramap is `LATT-<sensor>-<lens>-F<filter>` voor LATTICE (overeenkomend met de EXIF-tag van de opname
`Model`) en `<model>_<filter>` voor Survey3 — twee camera’s die een sensor en filter delen maar verschillen
in lens, behouden afzonderlijke mappen, omdat vignettering, gezichtsveld en vervorming verschillen. De map
map volgt `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, of `tiff32` voor
`TIFF (32-bit, Percent)`.

> **Elk geëxporteerd product behoudt de naam van het BRON-bestand.** Een radiance-export van
> `capture_…_raw.tif` heet nog steeds `capture_…_raw.tif` — het staat alleen in
> `tiff32/Radiance_Images/`. **De map identificeert het product, niet de bestandsnaam**, dus als je zoekt
> op `*radiance*.tif` vind je niets; zoek in plaats daarvan op de map.

### Opnames van de lichtsensor — gekalibreerd `.daq` + `.csv`

`process` verwerkt ook de `.daq`-opnames in je invoermap, en het heeft **geen**
beelden nodig om dit te doen: een DAQ-U / DAQ-M / DAQ-E die op zichzelf wordt gevlogen, vormt een volledige
opname, en een map die alleen `.daq`-bestanden bevat, is een geldige invoer.

Een DAQ kan **zonder** kalibratie worden opgenomen — dat is wat de openbare
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) recorders
(`record_daq.py`) standaard doen: ze schrijven ruwe sensortellingen weg en voorzien het bestand van een tijdstempel, zodat
Chloros de fabriekskalibratie van die sensor **via de seriële poort** (eerst uit de lokale cache,
daarna uit de MAPIR-cloud) en past deze toe. `process` schrijft het resultaat terug:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

De `.csv` bevat één rij per meting: UTC-tijdstempel, integratietijd, totaal vermogen,
fotopische/scotopische lux, PPFD (en de verdeling in blauw/groen/rood), piekgolflengte, gevolgd door het
volledige spectrum op het eigen golflengteraster van de sensor. Het bestand `.daq` wordt opnieuw geïmporteerd zonder
een tweede keer te worden gekalibreerd.

`<name>` is de bestandsnaam van de opname zelf. Twee opnames in verschillende mappen kunnen
één set metenwaarden per opname delen (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), opgeslagen in
dezelfde milliseconde door twee camera’s, en krijgen toch elk hun eigen resultaten: de eerste behoudt
`<name>`, de volgende voegt de mappen toe die hem onderscheiden **na** de naam
(`<name>__<capture>_daq_calibrated.daq`), zodat de datum en tijd van de opname nog steeds voorop staan.

Bij succes rapporteert de run `Light-sensor products written: N (calibrated .daq + .csv)`.
De tekst tussen haakjes beschrijft wat er daadwerkelijk is geschreven, dus luidt deze
`(RAW COUNTS — this sensor has no calibration bundle)` voor een sensor zonder bundel en
`(N calibrated, M raw counts)` voor een map die beide bevat. De eigen
`[DAQ-EXPORT]`- en `[RUN-SUMMARY]`-koppen van de backend ontlenen hun bewoording op dezelfde manier — geen van
de drie kan een ruwe export gekalibreerd noemen.

Een DAQ-U / DAQ-M / DAQ-E-opname waarvan de kalibratiebundel niet kan worden opgehaald — je bent
offline, of die sensor heeft geen kalibratie in het bestand — wordt **overgeslagen met een reden** op een
`[DAQ-EXPORT]`-regel, en wordt nooit weggeschreven als een „gekalibreerd“ bestand met ruwe tellingen.
Maak verbinding met het internet en voer de run opnieuw uit. De reden is degene die de lezer daadwerkelijk
voor dat bestand heeft vastgesteld (onleesbaar schema, geen bundel, een schrijffout), en het
runoverzicht vermeldt **afzonderlijke** redenen — twintig overgeslagen bestanden vanwege één oorzaak worden weergegeven als één
oorzaak, niet als twintig herhalingen daarvan. De CLI noemt aan het einde van
de run elke overgeslagen opname, met de bijbehorende reden:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Een map met opnames waarvan **geen enkele** wordt geëxporteerd, mislukt: zie
[Een run die geen afbeeldingen schrijft, mislukt](#a-run-that-writes-no-images-fails).

Een opname waarvan de naam eindigt op `_calibrated` of `_raw` (`field_raw.daq`) wordt **niet
geïmporteerd**: dat zijn de namen die `process` aan zijn eigen producten toekent, en een product dat
als opname wordt teruggelezen, zou dubbel worden geteld. De CLI geeft dit aan en noemt het bestand —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— in plaats van een lege map te melden. Hernoem een dergelijke opname voordat u deze verwerkt.

#### DAQ-A-opnames worden geëxporteerd als ruwe tellingen

De **DAQ-A** -serie dateert van vóór het systeem met bundels per serienummer en heeft geen kalibratiebundel
om op te halen — in plaats daarvan wordt deze in het veld gekalibreerd aan de hand van een reflectiedoel,
waardoor er nooit een nodig was. Door deze opnames te weigeren, konden de gebruikers hun
cijfers helemaal niet meer uitlezen, dus worden ze onder een **andere naam** geëxporteerd:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Een andere bestandsnaam in plaats van een vlag in het bestand, omdat de claim intact moet blijven
wanneer deze als kale naam per e-mail wordt doorgestuurd. De `.csv`-header vermeldt
`raw spectral sensor counts (NOT irradiance)` en waarschuwt dat de waarden vergelijkbaar zijn
**binnen** het bestand — wat precies is waarvoor ze bij kalibratie op basis van een doel worden gebruikt — en
niet tussen sensoren onderling. De vermogensafhankelijke fotometrische kolommen (totaal vermogen, fotopische en
scotopische lux, PPFD) worden als **NULL** geschreven in plaats van geïntegreerd op basis van tellingen, en het
overzicht van de run vermeldt `RAW COUNTS`, dus „geëxporteerd&quot; in een logboek kan niet worden geïnterpreteerd als bestralingssterkte.

Oudere **v1.01 / v1.02**-opnames (een DAQ-A-SD schrijft deze) bevatten geen tijdstip per meting,
alleen het tijdstip waarop het bestand is geschreven. De image↔downwelling-matcher weigert ze nog steeds — het koppelen van een
frame aan een schrijftijd zou onzichtbare fouten opleveren — maar de exporter leest ze wel, en
het CSV-bestand geeft `clock=daq_created_on` weer, zodat het product aangeeft op welke klok het staat.

### Opmerkingen

- `process` detecteert automatisch of je map Survey3, LATTICE of gemengd is.
- De voortgang wordt gestreamd via Server-Sent Events; de CLI toont live voortgang per thread (Detecteren, Analyseren, Verwerken, Exporteren).
- Voor Linux/Jetson controleert de CLI de swapruimte en kan een waarschuwing geven voordat grote mappen worden verwerkt. De textuurgevoelige debayer past ook automatisch een GPU-frequentiebeperking toe op Jetsons met laag stroomverbruik (Nano, Orin Nano).
- Bij succes rapporteert de run hoeveel beeldproducten er zijn opgeslagen (`Image products written: N`, met `(plus M clamp masks)` wanneer de unmix een pixel heeft afgekapt).
- **`process` deelt op veilige wijze een actieve backend.** Het herstart een reeds actieve backend automatisch alleen wanneer die backend inactief is met een eerder project geladen en geen camera’s, arrays of DAQ’s bevat. Een backend die een van deze elementen bevat (de GUI die een array streamt, een SDK-script, een DAQ-opname) wordt gebruikt zoals hij is en nooit opnieuw opgestart, zodat `process` naast een live array kan draaien. Als de backend meldt dat er een run gaande is, wacht `process` wacht maximaal 15 s tot deze is voltooid (het duurt ongeveer een seconde om een zojuist voltooide run te wissen), waarna het de uitvoering weigert met foutcode `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. De globale vlag `--restart` dwingt nog steeds een nieuwe backend af en beëindigt alle camera-, array- of DAQ-sessies daarop. Een backend die als Linux-systemd-service draait, wordt nooit opnieuw opgestart: `process` maakt er gebruik van, en `--restart` geeft in plaats daarvan `sudo systemctl restart <unit>` weer (zie [De backend als service uitvoeren](#running-the-backend-as-a-service-headless)). Dat geldt ook voor de eigen backend van de Chloros-app: als de app geopend is, gebruikt `process` deze en laadt het zijn project in plaats van dat van de app (een backend die in plaats daarvan zou worden gestart, zou de geopende app de toegang ontzeggen).
- **Een lopende verwerking behoudt zijn project.** Terwijl een verwerkingsrun aan de gang is — gestart door de app, `process`, een SDK-script of `chloros-mcp` — weigert de backend om bestanden aan te maken, te openen, te importeren, toe te voegen of te verwijderen, te verplaatsen, verwijderen of de verwerkingsinstellingen van een project te wijzigen, met HTTP-statuscode 409 `PROCESSING_IN_PROGRESS` en `A processing run is in progress on project '<name>', so ... now is refused`; een `process` die dit ontvangt, sluit `1`. Weergave-instellingen (miniatuurgrootte) worden nog steeds opgeslagen. De verwerkingsmodus en de instellingen voor de doelreflectie van het project worden op dezelfde manier geweigerd (een run behoudt de modus waarin deze is gestart). Een opname die tijdens de run in het project wordt opgeslagen (via de app, de CLI, de SDK of `chloros-mcp`) wordt zoals gewoonlijk opgeslagen, maar wordt pas aan de bestandslijst van het project toegevoegd wanneer de run eindigt; dit wordt aangegeven in het antwoord (`project_add`), net als bij een opname van de lichtsensor die in het project wordt gestopt. Voorheen kon een tweede client tijdens de run overschakelen naar het geopende project, waarna de lopende run naar het andere project werd geëxporteerd.
- **Ctrl+C stopt de run in de backend, niet alleen in de CLI.** De CLI vraagt de backend om de verwerking te stoppen, geeft `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` weer en wacht maximaal 120 s op `Processing stopped.`, zodat `chloros-cli process … && next` geen `next` op een backend die nog bezig is met de run. Een tweede Ctrl+C beëindigt het wachten. `process` sluit vervolgens `130` af, zoals elk onderbroken commando doet.

#### Een run die geen afbeeldingen schrijft, mislukt

Als je om producten hebt gevraagd en de run **geen** heeft geschreven — alleen `project.json` en
`calibration_data.json` — beschouwt `process` dat als een fout: het geeft
`Processing finished but wrote no image products.` en **stopt met een waarde anders dan nul**, zodat een script dit
kan detecteren. Het bericht vermeldt de projectmap en de gebruikelijke oorzaken:

- de invoermap werd niet herkend als een opname (controleer de lay-out en `--input-level`), of
- elk aangevraagd product werd overgeslagen omdat het niet van toepassing was op die camera’s (bijv. het vragen om
  stralingsintensiteit/reflectie van camera’s die alleen RGB ondersteunen).

Voer het opnieuw uit met `--verbose` en controleer het backend-logboek op regels met `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
die per camera uitgeslagen items verklaren die anders niet in de uitvoer van de CLI terechtkomen.

Een opzettelijke run met alleen metadata — alle producten uitgeschakeld en geen `--indices` — is nog steeds een
**succes**, omdat een lege afbeeldingsuitvoer daar het juiste resultaat is.

#### Een run waarvan de exportbestanden niet konden worden geverifieerd, mislukt

Na een run controleert `process` de tags van de geëxporteerde bestanden met ExifTool. Als ExifTool niet kon worden uitgevoerd,
wordt `Exportverificatie MISLUKT: ExifTool kon niet worden uitgevoerd, dus de tags van N van de M geëxporteerde bestanden
niet gecontroleerd (zie de [EXIFTOOL]-regels in het backend-logboek)` and `De resultaten zijn opgeslagen en blijven
op hun plaats; voer de run opnieuw uit om ze te verifiëren.`, does not print `Verwerking voltooid!`, en **stopt met een waarde anders dan nul**. De
resultaten blijven behouden.

#### Welke opnames een run omvat

Opnames die zijn voltooid voordat `process` wordt uitgevoerd, worden meegenomen. Opnames die tijdens de run zijn opgeslagen, worden niet meegenomen; `process`
geeft een waarschuwing over hoeveel er zijn weggelaten, en als je het opnieuw uitvoert, worden deze meegenomen.

#### Een run waarbij een afbeelding verloren is gegaan, mislukt

Als een beeld tijdens een run is mislukt, geeft `process` de melding `N image(s) failed and were not exported` weer met de
meest voorkomende oorzaak, geeft het geen melding `Processing complete!` weer en **stopt met een waarde anders dan nul**. De overige resultaten worden
opgeslagen en blijven op hun plaats staan, en in het backend-logboek staan alle mislukkingen vermeld.

Hetzelfde geldt voor een **run met alleen lichtsensoren**: een map met `.daq`-opnames bevat per definitie geen beeldmateriaal om te exporteren
, en de run wordt beoordeeld op basis van de gekalibreerde `.daq` / `.csv` die in plaats daarvan zijn geschreven.
Als er **niets** is geschreven — elke opname is overgeslagen — wordt
`Processing finished but wrote no light-sensor products.` weergegeven, waarbij elke opname wordt benoemd met de
reden waarom deze is overgeslagen, en **wordt de uitvoer niet-nul**.

---

## `chloros-cli login`

Verifieer deze machine met een Chloros+-cloudaccount. Inloggegevens worden veilig opgeslagen in `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Voorbeelden

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (door delen van het wachtwoord te verwijderen of te dupliceren). Bij een 401-fout probeert de CLI automatisch opnieuw met `$$`,-aangevoegd, en vervolgens met een ontdubbelde helft van het wachtwoord; als een nieuwe poging slaagt, logt het je in en geeft het de juiste syntaxis met enkele aanhalingstekens weer die je de volgende keer moet gebruiken.

> **Gebruik zonder gebruikersinterface/via script: geen geslaagde sessie betekent een inlogprompt.** Een core-commando (`process`, `status`, `export-status`, `time-sync`, …) die zonder een in de cache opgeslagen sessie wordt uitgevoerd, vraagt om `Email:` / `Password:` via stdin voordat het verdergaat. Als stdin gesloten is (cron, een systemd-unit), mislukt het onmiddellijk met foutcode `login cancelled`; als stdin een open terminal of pipe is, wacht het op invoer. Voer `chloros-cli login EMAIL PASSWORD` één keer per machine uit voordat u headless-taken inplant.

---

## `chloros-cli logout`

Wist de in de cache opgeslagen sessie en dwingt bij de volgende aanroep een nieuwe aanmelding af.

Uitloggen terwijl een verwerkingsrun bezig is, stopt deze niet: de run wordt voltooid in de verwerkingsmodus waarin deze is gestart, en de volgende run maakt gebruik van standaardverwerking. `logout` geeft dit aan (`The processing run in progress finishes in <mode> mode; the next run uses standard`), en dat doet de app ook wanneer je uitlogt.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Toont het huidige licentieniveau (Iron/Copper/Bronze/Silver/Gold), de geauthenticeerde gebruiker en het aantal apparaatkoppelingen.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Controleer de actuele voortgang van de Thread-4-export. Kan veilig worden aangeroepen **tijdens** een `process`-uitvoering vanuit een andere shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Stel de weergavetaal van de CLI in (38 talen worden ondersteund, waaronder CJK, RTL en Indic). Schakelt soepel over naar Engels op verouderde consoles die het script niet kunnen weergeven.

```
chloros-cli language [LANG_CODE] [--list]
```

### Voorbeelden

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Commando&#x27;s voor de projectmap

Deze beheren de standaardlocatie van de projectmap (gedeeld met de GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` geeft de map weer waar nieuwe projecten naartoe gaan. Als de geconfigureerde map niet meer bestaat (deze is verwijderd, hernoemd of bevindt zich op een schijf die niet is aangesloten), valt de CLI, net als de GUI, terug op de standaardmap: de standaardmap wordt weergegeven op stdout, er wordt een waarschuwing met de naam van de ontbrekende map naar stderr gestuurd en de exitcode is `0`.

---

## `chloros-cli update`

Alleen Linux/Jetson. Controleert `version_url` ten opzichte van `/etc/chloros/update.conf` en, wanneer er een nieuwere versie bestaat, de bijbehorende `.deb` gedownload en direct geïnstalleerd met `sudo dpkg -i` (geen bevestigingsvraag; bij `sudo` kan om uw wachtwoord worden gevraagd). Op een Jetson wordt de update alleen aangeboden aan een JetPack 6-systeem. Op Windows doet het commando niets en meldt het `up to date`: update met een nieuw installatieprogramma.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Op Linux/Jetson voert de CLI ook **bij elke opstart een automatische updatecontrole uit** (zonder blokkering, vertraagt het commando nooit): het leest `/etc/chloros/update.conf`, slaat het resultaat 1 uur op in de cache in `~/.chloros/update_cache.json` en geeft `Update available: vX.Y.Z / Run: chloros-cli update` weer wanneer er een nieuwere versie beschikbaar is. Wordt bij fouten en op Windows stil overgeslagen; `CHLOROS_DISABLE_UPDATE_CHECK=1` schakelt deze functie uit. Vanwege de cache van 1 uur kan een release die binnen dat uur is gepubliceerd nog steeds als up-to-date worden weergegeven.

---

## `chloros-cli selftest`

Voert een 7-stappen-smoketest uit: versie, beschikbaarheid van poorten, opstarten van backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), aanwezigheid van denoiser-model, gereedheid van CUDA+denoiser. Het maakt gebruik van een actieve backend, of start er een op voor de test en stopt deze daarna. Het geeft `N/7 checks passed` en de mislukte controles weer, en sluit `0` pas af wanneer alle zeven zijn geslaagd; de CUDA+ruisonderdrukker-controle mislukt op een computer zonder CUDA-GPU (zie [De installatie controleren](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmaster-status en -besturing. De Chloros-host draait de PTP-grandmaster; LATTICE-camera’s en DAQ-E-units werken hieraan als slave voor apparaatoverschrijdende tijdstempels (ongeveer 1 ms tussen apparaten). PTP synchroniseert **tijdstempels**: de camera’s van een array maken gelijktijdig opnames omdat de M8-hardwaretrigger ze activeert (minder dan 100 µs tussen modules), niet vanwege PTP. `chloros-cli time-sync` zonder subcommando voert `status` uit.

| Subcommando | Beschrijving |
| --- | --- |
| `status` | Toont de status van de grandmaster, BMCA-prioriteiten, klokidentiteit en of de pakketten het netwerk bereiken: `health` is `OK` (`STOPPED` wanneer de grandmaster niet actief is), of `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` met een `health_detail`, en vervolgens wordt een regel `WARNING:` naar stderr afgedrukt (stdout blijft JSON). `sync_sent` / `announce_sent` tellen alleen de berichten die zijn verzonden; `*_send_errors` en `last_send_error` tellen de berichten die niet zijn verzonden. Nadat een cameralink is verbroken en weer tot stand is gekomen, bouwt de grandmaster zijn sockets zelf weer op (`rebinds`, `last_rebind`). |
| `peers` | Lijst met slaves die via Delay_Req worden waargenomen (camera’s + DAQ-E-sensoren). |
| `cameras` | PTP-status per camera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Het grandmaster-proces opnieuw starten. |
| `set-priority --priority1 N --priority2 N` | BMCA-prioriteiten overschrijven. |

De grandmaster maakt zich bekend via de netwerkkaarten van de camera’s (elke adapter met een `169.254.x.x`-adres inbegrepen). Als er geen actief is, valt hij alleen terug op de adapter van de standaardroute wanneer die bekabeld is, nooit op Wi-Fi of een virtuele adapter (Tailscale, WSL), aangezien er geen camera of DAQ-E zich daarachter bevindt. Als er niets te melden is, draait het zonder sockets (`health` `NO_SOCKETS`), en de volgende camera of DAQ-verbinding bouwt het opnieuw op via de netwerkkaart die op dat moment actief is; `time-sync restart` doet dit in één keer.

De grandmaster draait in een eigen proces, naast de backend (`own_process` / `process_pid` in `time-sync status`), met hoge prioriteit op Windows (`process_priority`), zodat een drukke backend (een verwerkingsrun, een export, meerdere opdrachten tegelijk) de tijdstempels niet kan vertragen; voorheen kon één dergelijke vertraging de klok van alle camera’s tegelijk verschuiven. Dat proces eindigt bij de backend; als het niet kan worden gestart, draait de grandmaster zoals voorheen binnen de backend (`own_process` false, en dat staat ook in het backend-logboek).

### Voorbeelden

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-camerabesturing. Elk subcommando wordt via de Chloros-backend gerouteerd; de backend beheert de cam-pool, zodat volgende CLI-aanroepen dezelfde geopende handle hergebruiken.

### Algemene opties (geldend voor de meeste subcommando’s)

| Vlag | Beschrijving |
| --- | --- |
| `-d, --device N` | Camera-index (standaard: 0). |
| `-s, --serial SN` | Specifiek serienummer; overschrijft `--device`. |
| `--serials SN1,SN2,…` | Door komma’s gescheiden serienummers voor gebruik met meerdere camera’s. De [tijdelijke multi-camera-opdrachten](#multi-camera-transient-sessions) openen alleen deze camera’s, waarbij de eerste de trigger activeert. |
| `--all` | Toepassen op elke gedetecteerde camera. |
| `--exposure US` | Belichtingstijd in microseconden. |
| `--gain DB` | Versterking in dB. |
| `--pixel-format FMT` | bijv. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` op een M3M. De naam bepaalt de bitdiepte binnen de eigen familie van de camera: op een M3M bepaalt een Bayer-naam het Mono-formaat van de bitdiepte (`BayerRG8` stelt `Mono8` in), op een M3C bepaalt een Mono-naam de Bayer-naam (`Mono12` stelt `BayerRG12` in). |
| `--width N` / `--height N` | Afmetingen van de afbeelding. |
| `--preset {default,high_quality,high_speed,triggered}` | Een voorinstelling toepassen: het pixelformaat, de belichting, de versterking en de framesnelheid (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` geeft ze weer). Op een M3M is het formaat het Mono-formaat met dezelfde diepte: `high_speed` legt `Mono8` vast, `high_quality` en `triggered` `Mono12`. Een vlag die erbij wordt gegeven (`--exposure`, `--gain`, `--pixel-format`) heeft voorrang op de voorinstelling, en een instelling die niet in de voorinstelling wordt genoemd, behoudt de eigen instelling van de camera. Alle vrije-run, behalve `triggered`, dat de camera activeert voor een hardware-flank op lijn 2 — als er niets is dat die lijn aanstuurt, zal de camera eeuwig wachten in plaats van opnames te maken. |
| `-o, --output DIR` | Uitvoermap (standaard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-pakketgrootte. `auto` voert ICMP+GVSP-probes uit; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-camera: workflow voor eerste verbinding

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Overzicht van subcommando’s

#### Detectie en informatie

| Subcommando | Doel |
| --- | --- |
| `lattice info` | Lijst met aangesloten camera’s (fabrikant, model, serienummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Het hostsysteem analyseren voor een optimale cameraconfiguratie. `--no-discover` slaat de cameradetectie over (sneller, analyse uitsluitend via de netwerkkaart). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | NIC-instellingen controleren/corrigeren (op Linux ook de socket-ontvangstbuffers van de kernel — zie [Linux-hosts](#linux-hosts--socket-receive-buffers)); bandbreedte/FPS schatten. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Backend met stabiel schema voor netwerk-mogelijkheid + aanbeveling voor array (levert `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` zijn standaard ingesteld op de volledige sensor, `2048` × `1536`. Het stopt bij `0` voor `ok`, `auto_capped_fps` en `auto_shrunk`, en `1` voor `needs_force_slip` en `error`, met of zonder `--json`. Het bereikt de backend bij `--backend-url` (standaard `http://127.0.0.1:5000`), niet bij `CHLOROS_BACKEND_URL`. `auto_capped_fps` behoudt de gevraagde resolutie maar beperkt de doel-fps — lees `recommended.recommended_target_fps` en geef dit door als het connect-doel; beschouw dit als succes, niet als een fout. `auto_shrunk` verschijnt alleen wanneer het gevraagde frame bij geen enkele framesnelheid kan worden geleverd; connect verkleint nooit zelfstandig frames. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Wat-als-analyse zonder camera’s te openen (`--width` / `--height` standaard `2048` × `1536`). Zowel dit als `network-analysis` projecteren wat connect zal streamen: zonder `--pixel-format`, de formaten per camera die connect toepast (Mono12 / BayerRG12), en met `--packed auto` (standaard) de [gecomprimeerde 12-bits](#packed-12-bit-transport---packed) die Connect op deze computer zou nemen — dus de vermelde fps is de snelheid waarmee de array draait. Beide geven de packing-keuze weer. **`--n-active` is het totale aantal camera’s op de kabel, niet alleen dat van deze array**— verhoog deze waarde wanneer losstaande camera’s gelijktijdig streamen, of wanneer de bandbreedte wordt berekend op basis van een vraag die het aantal camera’s te laag inschat (standaard: `len(--models)`). Geeft altijd de samengevoegde regels `Wire budget:` (gevraagde MB/s versus botsingsveilige bovengrens) en `Max cameras:` weer, en markeert `** OVER-SUBSCRIBED**` wanneer de array de bandbreedte overbelast — zie [Array fps &amp; burst-model](#array-fps--burst-model). |
| `lattice gpu` | GPU-status weergeven. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Camerafirmware controleren of bijwerken. De lokale `.fwa`-selectie is vastgezet: het bestand in `firmware/<MODEL_PREFIX>/` dat overeenkomt met de `MIN_FIRMWARE_VERSION` van de build wordt geflashed indien aanwezig (alleen de hoogste versie als back-up), dus een nieuwere leveranciersimage die op de schijf is opgeslagen, blijft inactief totdat die vastgezette selectie wordt gewijzigd — opzettelijk nieuwere releases bereiken apparaten via het ondertekende AWS-manifest, wat de voorkeur heeft wanneer er een nieuwere versie beschikbaar is. |
| `lattice presets [--apply NAME]` | Cameravoorinstellingen weergeven of toepassen. `--apply` stelt vast wat de voorinstellingen omvatten (pixelformaat, belichting, versterking, framesnelheid, trigger) en geeft die instellingen weer zoals de camera ze leest na: op een M3M: het Mono-formaat van de diepte van de voorinstelling; de framesnelheid waarmee deze draait, die lager kan zijn dan de voorinstelling vraagt (`high_speed` vraagt 60 fps; een full-frame camera op een 1 Gbps-verbinding draait op ongeveer 35). Alle andere instellingen die het programma heeft geschreven, worden teruggezet zoals ze zijn aangetroffen. `--apply default` zet de camera terug naar de fabrieksinstellingen: de fabrieksinstellingen met vrijlopende automatische belichting op Line2, de limiet voor de verbindingsdoorvoer en defectcorrectie uitgeschakeld, en de gekalibreerde witbalans van een RGB-camera behouden (eenheid op alle andere). Er wordt niets opgeslagen in het geheugen van de camera, dus bij de volgende keer opstarten begint deze vanuit het eigen geheugen zoals voorheen. Als `--preset` of `default` bij het vastleggen of verbinden wordt gebruikt, is er geen voorinstelling. |
| `lattice status` | Live camerastatus. |

#### Opname

| Subcommando | Doel |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Enkel beeld. **Slaat standaard elk exporttype op**(`--processing all`); zie [Opname-exportniveaus](#capture-export-levels-the-all-default). `--processing` accepteert `all`, `raw`, `debayered`, `radiance`, `reflectance` of `display` (`preview` wordt geaccepteerd voor `display`); elke andere naam wordt geweigerd voordat de camera wordt aangeraakt (verlaat `1`). `--levels` slaat een expliciete subset op (heeft voorrang op `--processing`). Een camera die Chloros niet in bezit heeft, wordt geopend voor de opname, en de belichting ervan wordt eerst op basis van de scène ingesteld (maximaal een paar seconden), zoals bij de [tijdelijke commando’s](#multi-camera-transient-sessions); wanneer de opname eindigt, hoe deze ook eindigt, wordt die camera teruggezet zoals hij werd aangetroffen (formaat, belichting, kleurpijplijn, elke instelling die door de opname is gewijzigd — de lijst onder [Multi-Camera (Transient Sessions)](#multi-camera-transient-sessions)). De DAQ-uitlezing van de gebruikte opname wordt opgeslagen als een `.daq`-sidecar, inclusief een ‘raw-only’-opname; `--no-daq` / `--daq-tolerance-ms` passen deze aan ([De DAQ die een opname leest, slaat deze op](#the-daq-reading-a-capture-saves)). `--force-daq` wordt nog steeds geaccepteerd. `--jpeg-quality` = JPEG-kwaliteit 1–100 (standaard 95).**Bestanden die in het geopende project worden opgeslagen, worden daaraan toegevoegd.** Net als in de app worden bij een opname waarvan de bestanden in de map van het in Chloros geopende project terechtkomen, deze toegevoegd aan de bestandslijst van dat project; bestanden die elders worden opgeslagen, blijven ongemoeid. Het commando geeft één regel weer waarin dit wordt vermeld (`Project: ...`), of, terwijl een verwerkingsrun bezig is, dat ze worden tegengehouden en worden toegevoegd wanneer de run is voltooid. Hetzelfde geldt voor `multi-capture`, `array-capture` (`--smart` ook) en `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Stroom naar schijf tot Ctrl+C. De frames van een `tiff`-runzijn de frames van een `tiff`-run het `raw`-bestand, een array dat door de camera wordt vastgelegd: de mozaïek van de sensor, één vlak met zijn eigen waarden, ongecomprimeerd, met het kalibratiepakket (een frame van 2048 x 1536 met 12 bits is 6,0 MiB plus het pakket, ongeacht de scène), dus `process` begint daarmee. `png` en `jpg` bevatten 8 bits, dus die reeksen zijn gedemosaïceerde frames voor weergave, waar `process` niet mee begint. Bij een DAQ-gebonden (`daq pool-bind`) of in een pool, krijgt elk frame zijn eigen `.daq` en `.match.json` in de map `daq/` van de reeks, aangezien `lattice capture` er één opslaat; `--no-daq` schakelt dat uit, en bij het stoppen wordt weergegeven hoeveel het er zijn. Elk frame wordt één keer opgeslagen, met de volledige metagegevens; bij Ctrl+C worden de reeds vastgelegde frames geschreven voordat de camera wordt vrijgegeven. Stopt zichzelf wanneer de schijf vol raakt, drukt `Failed:` / `Disk full:` en sluit af met `1`; elk frame dat niet kon worden opgeslagen, leidt ook tot het afsluiten met `1`. Een camera die is ingesteld op een hardware-trigger (`TriggerMode` `On`) wordt voor de opname ingesteld op vrijloop. Wanneer de opname eindigt, wordt de camera teruggezet in de toestand waarin hij werd aangetroffen — de triggerinstellingen en alle instellingen die tijdens de opname zijn gewijzigd (belichting, versterking, formaat, …) — tenzij `--keep-settings`, dat de instellingen van de opname-sessie opslaat in het eigen geheugen van de camera (vrije modus), of `--clear-settings`, dat dat geheugen terugzet naar de fabrieksinstellingen. Een run die geen enkel beeld vastlegt, is mislukt: als er gedurende 10 s geen beeld is, stopt de run vanzelf, drukt `No frames:` af met de reden en beëindigt de run met `1`. Frames die de schrijvers niet konden bijhouden, worden geteld als verloren, en bij het stoppen wordt `Dropped:` afgedrukt met de reden (de framesnelheid van de camera, die van de schrijvers en de wachtrij); bij een lagere framesnelheid of een 8-bits pixelformaat worden ze allemaal bewaard. Het commando meldt zich elke 2 s bij de backend; als het afbreekt zonder de opname te stoppen (`kill -9`, onvoldoende geheugen), stopt de backend de opname 30 s na de laatste melding en zet de camera terug (`stop_reason` `client_gone`, en een regel in het backend-logboek). `multi-stream` en `align-stream` doen hetzelfde. Een start waarbij het openen van de camera langer duurt dan de wachttijd van het commando (60 s; 180 s voor `multi-stream` / `align-stream`) — een camera die nog aan het opstarten is — wordt door de backend ongedaan gemaakt: de camera wordt teruggezet en gesloten en er blijft geen opname lopen (504 HTTP-fout `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Live MJPEG-voorvertoning via de browser: maakt verbinding met de camera, geeft de stream-URL weer en opent deze in je browser. `--ae-damping` stelt de automatische belichtingsdemping in (0,4–100). |

##### Livestreams (MJPEG; geen RTSP)

Chloros levert live video als **MJPEG via HTTP**(`multipart/x-mixed-replace`, grenswaarde `frame`) vanuit de backend.**RTSP wordt niet ondersteund.** Open een URL-adres in een browser, of lees deze in met een willekeurige MJPEG-client (OpenCV’s `cv2.VideoCapture(url)`, ffmpeg, VLC). De camera moet eerst in de backend worden aangesloten (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` of via de GUI).

| URL (`http://127.0.0.1:5000` + …) | Stream |
| --- | --- |
| `/api/camera/stream/<serial>` | Het weergavevoorbeeld, tot ongeveer 30 fps. Voor een array-element, stellen `?max_h=` (standaard `720`) en `?jpeg_q=` (standaard `80`) de hoogte en de JPEG-kwaliteit in. Dit is de URL die `lattice viewer` opent. |
| `/api/camera/<serial>/stream-annotated` | Hetzelfde voorbeeld met de overlays van de GUI erin getekend: `zebra=1`, `crosshair=1`, `grid=3x3` (of `9x9`), `histogram=1`, `peaking=1`, `spot=1`; zonder vraag worden de opgeslagen overlay-instellingen van het project voor die camera gebruikt. |
| `/api/camera/<serial>/stream-reflectance` | 8-bits reflectantie per band: ρ = pixel ÷ het deel’`X-Chloros-Pixel-Scale`-header (255 tenzij `refl_max` is geactiveerd). `fps` (standaard `10`), `jpeg_q` (standaard `95`), `max_h` (standaard `0` = native), `band=<name>`, `align=1`. JPEG, dus met verlies. Wordt geweigerd met HTTP-statuscode 409 wanneer de camera geen reflectie heeft (een RGB-camera, geen kalibratiepakket, geen DAQ-meting). |

- **Frames van verschillende camera&#x27;s op elkaar afstemmen.**Elk onderdeel dat een**array-lid** verstuurt, op alle drie de streams, bevat de headers `X-Chloros-Group-Id` (de synced-group-teller van de array), `X-Chloros-Frame-Id` en `X-Chloros-Timestamp-Ns`. Delen van verschillende camera’s met dezelfde `X-Chloros-Group-Id` zijn afkomstig van dezelfde trigger. Een standalone camera heeft geen groep en verstuurt geen groeps-ID. Browsers negeren headers per onderdeel; een client die de multipart-stream zelf parseert, leest deze wel.
- **Gepauzeerd terwijl de array-weergave open is.** `/api/camera/stream/<serial>` van een array-lid verzendt een statisch „paused“-frame terwijl de gecombineerde livestream van de array open is (bijvoorbeeld de array-weergave van de GUI), en gaat verder wanneer deze wordt gesloten.

#### Sensorinstellingen

| Subcommando | Doel |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Lees/schrijf een willekeurig GenICam-knooppunt. `--set ExposureTime=…` (`Gain=…`) zonder de schakelaars `ExposureAuto` (`GainAuto`) die de automatische modus eerst uitschakelen, zoals `lattice exposure --set` dat doet; een waarde die de camera weigert, leidt ertoe dat `Failed to set <node>: …` op stderr wordt afgedrukt en het programma wordt afgesloten met `1`. `--get` en de standaard gegroepeerde weergave geven een `note:` weer onder `ExposureTime` / `Gain` terwijl die automatische modus actief is — zie [Belichting en versterking aflezen in de automatische modus](#reading-exposure-and-versterking-in-automatische-modus). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Belichting &amp; AE. Een `--set`-belichting of `--brightness`-doelwaarde wordt als de jouwe aangehouden (de automatische belichtingsregeling van Chloros laat deze met rust); de versterking blijft zoals hij was, dus in de automatische modus blijft het systeem zich aanpassen aan het doel totdat je `lattice gain` instelt. `--auto` geeft de belichting weer terug, `--brightness-auto` geeft de doelwaarde weer terug aan de automatische doelbepaling vChloros. Zonder vlaggen leest het de belichtingsknooppunten, met dezelfde `note:` als `configure --get`. Bij een waarde die de camera weigert, drukt `--set` `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` af op stderr — de automatische modus waarvan het schrijven was uitgeschakeld, wordt hersteld in plaats van uitgeschakeld te blijven — er wordt geen `Exposure: … us (auto OFF)`-regel weergegeven en het programma wordt afgesloten met `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Versterking en automatische versterking. Bij een `--set`-waarde weigert de camera `Failed to set Gain: …` op stderr af te drukken, geen `Gain: … dB (auto OFF)`-regel, en wordt `1` afgesloten. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Sensor-ROI en binning. Binning is altijd **Gemiddeld** — zie hieronder. |
| `lattice format [--set FMT] [--list]` | Pixelformaat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardware-/softwaretrigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (geen vlaggen = eenmalige witbalans) | Witbalansbewerkingen. Alleen RGB-/Bayer-camera’s; een no-op (overgeslagen) op mono M3M. De one-shot behoudt de witbalans die hij berekent (`BalanceWhiteAuto` en de versterkingen); alle andere instellingen die hij open schrijft, worden teruggezet zoals ze werden aangetroffen, en alles wanneer de one-shot niet kan worden uitgevoerd. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB-kleurverwerkingspijplijn. `natural` (standaard) is de goedkope live-afwerking; `enhanced` voegt defringe + vibrance + CLAHE lokaal contrast toe voor de volledige hub-parity-look tegen ~2× de verwerkingskosten per frame, dus een lagere **live** framerate — opgeslagen opnames krijgen hoe dan ook altijd de volledige verwerking. Alleen RGB-/Bayer-camera’s; wordt overgeslagen op mono M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Verzadiging/contrast in live-preview (camera’s met RGB-filter); `100` is neutraal, `--reset` stelt beide in op 100. Opgeslagen opnames worden hierdoor niet beïnvloed. Wordt overgeslagen op mono M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (geen vlaggen = lezen) | **Hooglichten** weergeven (camera&#x27;s met RGB-filter): de helderste, witste pixels verzwakken — bijv. een helder wit oppervlak dat wordt weergegeven als een vlakke witte vlek. `0` = uit, `-100` = sterkst. Het livevoorbeeld en opnamebeelden op weergaveniveau volgen deze instelling; straling/reflectie doen dat nooit, en het kan geen sensorgegevens herstellen die zijn afgekapt (verlaag daarvoor de belichting). Overgeslagen op mono M3M. |
| `lattice filter [--list]` | Geef een overzicht van de filtermodellen die Chloros kent (`RGN-IMX265`, `OCN`, `NGB`, …). Het filter van een camera wordt afgelezen uit de modelstring en kan niet worden ingesteld: `--set NAME` zoekt alleen een model op en sluit vervolgens `1` af met `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Controleert stroom- en thermische knooppunten; schakelt de energiebesparende stand in of uit. `--sleep` laat de camera in de slaapstand staan (de limiet voor de linkdoorvoer is ingeschakeld (`DeviceLinkThroughputLimitMode` `On`) op het minimum, de energiebesparingsmodus is ingeschakeld indien de camera daarover beschikt) waarbij alle andere instellingen worden teruggezet zoals ze aanvankelijk waren; een mislukte slaapstand wordt in zijn geheel teruggezet. Bij het ontwaken wordt de verbinding teruggezet zoals deze bij het in slaap gaan was (de limiet, de modus en de pakketvertraging `GevSCPD`, die de camera op 0 houdt zolang de limiet is ingeschakeld; dit wordt bijgehouden op deze computer, zodat een wekactie na het herstarten van de backend deze ook aantreft), of, voor een camera die deze computer niet in de slaapstand heeft gezet maar die eruitziet alsof hij slaapt (de limiet is ingeschakeld op het minimum), met de limiet uitgeschakeld en een `GevSCPD` op 0 gelaten bij 80 (de fabrieksinstellingen); een limiet die boven de ondergrens is ingesteld, blijft ongewijzigd. De probe en het ontwaken zetten terug wat hun open-bewerking heeft geschreven. |

> **Binning is altijd ‘Average’.** `lattice resolution --binning N` stelt ‘Average’-binning in en biedt
> geen moduskeuze. Bij ‘Sum’-binning wordt elke DN vermenigvuldigd met N², wat ongeveer twee stops aan
> headroom in de hoge lichten per binningstap kost; Chloros deelt die factor weer uit vóór de
> radiometrische keten, maar pixels die door ‘Sum’ tegen het plafond van de sensor zijn gedreven, worden afgekapt en kunnen niet worden
> hersteld. Het levert niets op wat ‘Gemiddeld’ niet doet — een index is schaalinvariant en de stralingssterkte-
> scalairen zijn gekalibreerd op de ‘Gemiddeld’-schaal. Als een camera al is ingesteld op ‘Som’ (zo
> achtergelaten door een andere tool, of door een fabrieksreset), detecteert Chloros dit via een node-readback en
> compenseert dit; door `lattice resolution --binning N` uit te voeren wordt de instelling teruggezet naar ‘Average’. De 1.2.0
> `--binning-mode`-vlag wordt nog steeds geaccepteerd, zodat oudere scripts blijven draaien, maar er wordt alleen een waarschuwing weergegeven:
> binning blijft op ‘Gemiddeld’ staan.

> **Het wijzigen van binning, pixelformaat en trigger op een camera heeft geen effect.** Op een camera blijft de
> backend openstaan en streamen (`cam-connect`, de GUI), `lattice resolution --binning`
> treedt niet in werking (er wordt gemeld dat de binning nog steeds van kracht is, `Binning: 1x1`), en
> `lattice format --set` en `lattice trigger --mode` worden door de camera geweigerd
> (`SC_ERR_ERROR -1001`). Geef deze eerst vrij (`lattice cam-disconnect -s <SN>`), of stel
> ze in bij het verbinden (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Belichting en versterking aflezen in de automatische modus

Terwijl de `ExposureAuto` (`GainAuto`) van een camera actief is, behoudt het `ExposureTime` (`Gain`) de **laatst geschreven waarde** bij, niet wat de camera op dit moment doet. Voor een camera waarvan de backend open blijft (`cam-connect`, `array-connect`, de GUI), geven `lattice configure --get`, de standaardweergave `lattice configure` en `lattice exposure` (zonder vlaggen) daarom de knooppuntwaarde weer en, daaronder, een opmerking met het nieuwste beeld van de camera:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Lees de framewaarde uit de opmerking; de knooppuntwaarde is alleen de laatste schrijfbewerking. Via HTTP houdt hetzelfde `POST /api/camera/<serial>/read-nodes`-antwoord `nodes` ongewijzigd en voegt het `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, dat van het nieuwste frame) en `notes` (de bovenstaande zin) toe. Beide zijn alleen aanwezig zolang de automatische modus niet `Off` is en er een frame bekend is; een camera die alleen voor het lezen is geopend, heeft geen frame en krijgt geen van beide. `lattice gain` (geen vlaggen) drukt alleen het knooppunt af — gebruik `lattice configure --get Gain` voor de opmerking.

##### Belichting op een array-element

Terwijl een array streamt, draait elk element met **`AcquisitionFrameRateEnable`**, met een framesnelheid die iets hoger ligt dan de triggersnelheid van de array: de periode is de triggerperiode minus 15 % (minimaal 3 ms), of de maximale snelheid van de camera zelf wanneer de trigger daar al dichtbij ligt. `configure --get` op een element leest daarom bij die instelling `AcquisitionFrameRateEnable` als `True` en `AcquisitionFrameRate`; de activering wordt weer uitgeschakeld wanneer de array stopt.**Schrijf deze twee knooppunten niet op een array-lid** — de array houdt ze synchroon met de trigger en herschrijft ze.

Die instelling is ook de maximale belichtingstijd van het lid: ongeveer 85 ms bij een trigger van 10 fps (periode van 100 ms), bij **elke ROI of binning**. Vóór versie 1.3.0 beperkte een kleine ROI de belichtingstijd van een array-element daarentegen tot de free-run-periode van de sensor (4,9 ms bij 768 × 304), hoe langzaam de array ook triggerde.**Een handmatige `ExposureTime` die langer is dan de limiet, vertraagt de trigger om hieraan te voldoen** wanneer deze wordt geschreven met `lattice exposure --set` of `configure --set` (of via de GUI). De array opent eerst de limiet van dat element voor de waarde en laat de lagere snelheid een paar triggerperioden draaien, waardoor het schrijven enkele seconden langer kan duren; de trigger wordt vervolgens niet sneller geactiveerd dan de belichting toestaat. Bij een array met automatische snelheid keert de snelheid die eerder werd gebruikt terug zodra elk element waarvan de handmatige belichting de snelheid vertraagde, weer op automatische belichting staat. Een waarde die de camera zelf niet kan vastleggen, wordt nog steeds geweigerd, waarbij het toegestane bereik in de foutmelding wordt vermeld, en de automatische belichting van het lid wordt hersteld als deze alleen voor die schrijfbewerking was uitgeschakeld. Een recept met de belichting `apply` vertraagt de trigger niet: deze wordt op de bovengrens gehouden. Chloros verkort nooit een handmatige belichting om deze aan te passen aan een snellere trigger; een vastgezette snelheid (`array-connect --target-fps F`) die sneller is dan een belichting toestaat, laat dat lid lang genoeg op het maximum staan, en dat lid neemt geen triggers die sneller zijn dan de belichting toestaat.

#### Kalibratie &amp; Sensoren

| Subcommando | Doel |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibreren aan de hand van een reflectiedoel. Er wordt gezocht in maximaal `--attempts` frames (standaard `30`); een onvolledig frame (GVSP-pakketverlies) kost één poging, niet de hele run. Het mislukt bij `no target detected in N attempts`, of bij `no complete frame in N attempts: <last error>` wanneer er geen volledig frame is aangekomen. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Het spectrum en de bandmagnitudes van een DAQ-M, uitgelezen via Bluetooth (BLE) via de backend. `--mac` is het eigen BLE-MAC-adres van de sensor en is vereist bij elk van de drie acties: er is geen standaardwaarde, aangezien elk apparaat zijn eigen adres heeft (`daq pool-discover --only ble` geeft ze weer); zonder dit nummer wordt het commando beëindigd met `1`. Zonder actie wordt de helptekst weergegeven. `--camera SERIAL` integreert de weergegeven bandmagnitudes via de eigen gemeten spectrale respons van die camera in plaats van de fleet-curve van het filtermodel — RGB-camera’s, waarbij het kalibratiepakket van de camera is geladen (aangesloten) of in de cache van deze computer is opgeslagen. In de koptekst staat wiens curve is gebruikt: `Band magnitudes (camera <SERIAL>):`, of `Band magnitudes (model <filter>):` wanneer de camera er geen heeft (geen RGB-camera, of geen pakket geladen of in de cache opgeslagen) en in plaats daarvan de standaardcurve is gebruikt. Via HTTP neemt `POST /api/dls/cli-read` `camera_serial` en retourneert `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Pas vignetcorrectie toe op bestaande afbeeldingen, elk met het vlakveld van de eigen lens (uit het EXIF-model). `--lens-model` is van toepassing op beelden waarvan het model geen camera vermeldt; een beeld waarvan het model hiermee in tegenspraak is, wordt overgeslagen. |

#### Meerdere camera’s (tijdelijke sessies)

Een tijdelijke opdracht opent de camera’s voor één run en sluit ze wanneer deze is voltooid: `multi-capture` zonder gekoppelde permanente array, `multi-stream` en `multi-test` hier, en `align-calibrate`, `align-apply`, `align-stream` en `index --live` hieronder. Voor al deze commando’s gelden de volgende regels:

- **`--serials SN1,SN2,…` kiest de camera’s.** Alleen die worden geopend, en de eerste activeert de trigger (voor `align-calibrate` en `--reference` indien opgegeven). Een seriële camera die niet op het netwerk is aangesloten, zorgt ervoor dat de run mislukt voordat er een camera wordt geopend. Zonder `--serials` wordt elke camera op het netwerk geopend. Een camera die Chloros al in bezit heeft (via de GUI, `cam-connect`, `array-connect`) weigert de run, ongeacht of `--serials` deze noemt of niet: deze deelt het netwerk waarvoor de run is ingesteld. (`index --live` zonder `--serials` opent in plaats daarvan de ingeschakelde camera’s van zijn profiel: zie [Index / Vegetation Maths](#index--vegetation-maths).)
- **De camera&#x27;s delen het netwerk op dezelfde manier als `array-connect` dat doet.** Elke camera wordt afgestemd op zijn aandeel in wat de hostverbinding vervoert (een vertraging tussen pakketten, niet de doorvoersnelheidslimiet van de camera, waarvan de ondergrens van 31,25 MB/s al wordt overschreden door vier camera’s op 1 GbE), dus draaien vier camera’s op een 1 Gbps-verbinding. Wanneer de camera’s niet allemaal tegelijk kunnen verzenden op het langzaamste tempo dat ze halen, worden ze één voor één geactiveerd, elk met de verbinding voor zichzelf, en de opdracht geeft `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together` weer. Wanneer zelfs maar één camera per keer niet past, wordt de uitvoering geweigerd voordat er ook maar één camera streamt: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP-status 409, afsluiten `1`.
- **Elke camera moet gegevens leveren, anders mislukt de run.** Een camera die geen frame levert, is een fout: het commando geeft `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …` weer, waarbij `<why>` de ophaalfout van die camera is, gevolgd door de oorzaak ervan, en afsluit met `1`. Een frame dat is aangekomen met GVSP-pakketverlies (`IncompleteBufferError: GVSP packet loss …`) betekent dat het netwerk de gecombineerde bandbreedte van de camera’s niet aankan (`Every missing frame arrived with GVSP packet loss: …`): controleer de verbindingssnelheid van de host en van de uplink van de switch (`lattice network`), gebruik jumboframes, of open minder camera’s met `--serials`. Een camera die niets heeft geleverd, ontvangt meestal geen trigger van de master: controleer de M8-synchronisatiekabel en voer vervolgens `lattice multi-detect` uit. Wat elke opdracht doet bij een framegroep waarin een camera ontbreekt, staat in de betreffende rij.
- **Elke camera wordt teruggezet zoals deze is aangetroffen.** Wanneer de run eindigt, krijgt elke camera terug wat deze had toen de run deze opende: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, en `LineMode` / `LineSource` op `Line2` en `Line3` (en de deglitch van de trigger-ingang van een array levert de lijn van een slave op, `LineFilterSelector` / `LineFilterWidth`); zijn `PtpEnable` (en de `PtpSlaveOnly` / `PtpDomainNumber` die daarmee zijn ingesteld); `ExposureAuto` / `GainAuto` (en `ExposureTime` / `Gain`, waar het op een handmatige waarde stond); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (en elke `BalanceRatio` waarbij de witbalans handmatig was ingesteld); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (en `Gamma` waarbij gamma was ingeschakeld), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; de pakketgrootte (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); de pacing, `DeviceLinkThroughputLimitMode` (en `DeviceLinkThroughputLimit` waarbij de modus `On` was), depakketvertraging `GevSCPD` en de startverspringing `GevSCFTD` die een array aan elke camera geeft; de chunkgegevens die elke stream inschakelt (`ChunkModeActive`, elk `ChunkEnable` onder zijn `ChunkSelector`); `AcquisitionFrameRateEnable` (en `AcquisitionFrameRate` waar de activering aan stond — als deze uit staat, is de snelheid alleen-lezen en wordt het maximum gelezen dat de timing toestaat, dus het is geen instelling). Alleen wat verschilt, wordt geschreven: de camera is eerst gestopt, de pacing en de framesnelheid vóór een belichting die hun frameperiode nodig heeft (een camera beperkt `ExposureTime` tot één frameperiode). Met `--keep-settings` worden de trigger- en synchronisatielijn-bedrading teruggezet, draait de camera vrij en wordt de rest opgeslagen in het eigen geheugen van de camera zoals de run het heeft ingesteld; `--clear-settings` zet daar de fabrieksinstellingen.
- **De belichting wordt ingesteld op basis van het beeld vóór het eerste frame wordt vastgelegd.** De run neemt gesynchroniseerde beeldgroepen op en verwijdert deze terwijl de belichting en versterking van elke camera op basis van die beelden worden ingesteld. Het doel is een gemiddelde helderheid op het doel van de camera waarbij niet meer dan 1 % van het beeld wordt afgekapt; een scène met felle hooglichten blijft iets onder het doel. Dit kost doorgaans 2-6 beeldgroepen en maximaal 5 s.
  - Elke camera richt zich op het doel waar een aangesloten array mee begint, via zijn filter: TargetBrightness **40**op een multispectrale camera (RGN, OCN, NGB) en**60** op een RGB-camera. Dat is de helft van de belichtingsdoelwaarde die 80 oplevert (driekwart bij RGB), waardoor een helder reflectiepaneel minder snel wordt afgekapt. `--brightness N` (0-255) stelt in plaats daarvan de doelwaarde voor elke camera in (`multi-test` heeft geen `--brightness`: de automatische belichting is uitgeschakeld). De logbestanden van de backend vermelden de belichtingsdoelwaarde van elke camera: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - De belichting komt op de eerste plaats, tot aan de limiet van de automatische belichting van de camera zelf. De versterking dekt alleen wat er overblijft.
  - De vastgestelde belichting en versterking worden gedurende de gehele opname behouden, dus elk frame van een `multi-stream` wordt met dezelfde belichting opgenomen. Een opname die veranderende lichtomstandigheden moet volgen, hoort thuis in een gekoppelde reeks (`array-connect`), waarvan de automatische belichting zich voortdurend aanpast.
  - `--exposure` en `--gain` worden exact behouden zoals opgegeven. Als beide zijn opgegeven, wordt geen enkel beeld verwijderd.
  - Wanneer de reeks is voltooid, wordt de automatische belichting van elke camera weer ingeschakeld.
  - Het logboek van de backend rapporteert het resultaat: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …`, of `exposure NOT solved after …` met de reden en `(not settled)` voor elke camera waarvoor dit niet het geval was. De beelden worden in dat geval nog steeds vastgelegd.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` wijzigt de limiet van 5 s; `0` schakelt deze uit (zie [Omgevingsvariabelen](#environment-variables)).

**Zolang Chloros een camera vasthoudt** (de GUI, `cam-connect` of een aangesloten array), worden `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` en `index --live` weigeren te draaien, omdat ze elke camera op het netwerk openen en het tweede openen van een vastgehouden camera de backend kan laten crashen. (`index --live` weigert wanneer een camera die het zou openen vastgehouden wordt: de camera’s van `--serials` of van het profiel, en elke camera voor een profiel dat er geen noemt.) De backend geeft een HTTP-antwoord 409 `CAMERA_HELD`, waarbij elke bezette camera en de vrijgavecommando’s worden genoemd (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), waarna de opdracht wordt beëindigd met `1`. Als er een array is aangesloten, slaat `array-capture --continuous` deze op in plaats van `multi-stream`, en `array-capture` (standaard uitgelijnd) slaat een uitgelijnde set op in plaats van `align-apply`; `multi-capture` en `align-calibrate` gebruiken een gekoppelde array die hun camera’s bevat in plaats van ze te openen.

| Subcommando | Doel |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Geeft een lijst weer van alle camera’s met synchronisatiefuncties; met `--serials` alleen die camera’s, de eerste als `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Eén gesynchroniseerd frame van elke camera. Slaat **standaard alle exporttypes op**wanneer een permanente array is aangesloten, en de DAQ-lezing die is gebruikt ([De DAQ-lezing die een opname opslaat](#the-daq-reading-a-capture-saves)); de tijdelijke fallback zonder array wordt**alleen gedebayerd** (voer eerst `array-connect` uit voor de rest). `--brightness` is alleen van toepassing op de fallback: een aangesloten array behoudt zijn eigen automatische belichting. De fallback slaat geen `.daq` op (`--no-daq` / `--daq-tolerance-ms` hebben daar geen effect), en `multi-capture` accepteert geen `--force-daq`. In de fallback mislukt de opname als een camera geen frame levert (`Error: multi-capture failed: no frame from …`, afsluiten `1`) en wordt er geen bestand van die framegroep opgeslagen. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Gesynchroniseerde frames streamen (tijdelijk), `--fps` standaard `3`, totdat Ctrl+C of `--count` groepen. Een framegroep waarvoor een camera niets heeft geleverd, wordt niet geschreven, en drie achter elkaar beëindigen de stream. Als er een groep is overgeslagen, eindigt de run door `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)` af te drukken, plus `Error: the stream stopped: …` met de naam van de camera toen deze de stream beëindigde, en sluit het af met `1`X. Stopt zichzelf wanneer de schijf vol raakt, geeft `Failed:` / `Disk full:` weer en sluit af met `1`; elk bestand dat niet kon worden geschreven, sluit ook `1` af. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO-synchronisatietimingtest (tijdelijk), `--count` standaard `10`-triggers. Geeft weer hoe het netwerk wordt gedeeld (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, en de regel `Serial emit:` wanneer de camera’s na elkaar worden geactiveerd, waarbij de tijdstempels dan de onderlinge vertraging weergeven), een `Sync group: master SN:<serial> -> SN:<serial>, …`-regel per synchronisatiegroep (`(no slaves: not synced)` voor een camera zonder synchronisatiekabel, die op zichzelf wordt geactiveerd), vervolgens één rij per trigger: `[  0] SN:<serial>=<timestamp> \| …`, waarbij `SN:<serial>=(no frame)` staat voor een camera die bij die trigger niets heeft geleverd. Vervolgens `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` en een `SN:<serial>: mean <±x> us, jitter <y> us`-regel per camera, elke camera ten opzichte van de master binnen één klokcyclus: een trigger waarvan de tijdstempels de apparaatklokken van de camera’s en PTP (PTP uitgeschakeld of nog aan het synchroniseren) wordt weggelaten, waarbij een `Note:` ze telt. Elk gemist frame zorgt ervoor dat de test mislukt (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, afsluiten `1`); drie onvolledige triggers op rij beëindigen de test voortijdig. De test mislukt ook (exit `1`) wanneer bij minder dan twee triggers alle camera’s op één klok staan (`… the clocks cannot be compared`), of wanneer de klokken nooit overeenkomen: PTP-klokken die meer dan 1 ms (plus eventuele vertraging op het seriële niveau) afwijken van de master (`… PTP is not locked`), of apparaatklokken waarvan de afwijking tussen triggers groter is dan dat. Het start de PTP-grandmaster als deze niet draait, schakelt de PTP van elke camera in (zoals `array-connect` doet, ongeacht hoe het netwerk eruitziet), en wacht vóór de eerste trigger tot de PTP-klok van elke camera is vergrendeld op de grandmaster en stabiel is: `PtpStatus` Slave op één grandmaster, binnen 100 us daarvan en stabiel gedurende 3 s, en elke camera op die manier vergrendeld, zonder onderbreking, gedurende 120 s — ongeveer 2 minuten vanaf het inschakelen-in. Een klok die net is gesynchroniseerd, wijkt tot ~200 us af gedurende ongeveer 2 minuten nadat deze ‘Slave’ aangeeft, en de eigen `PtpOffsetFromMaster` van de camera geeft dit niet weer; na 120 s komen de camera’s binnen ~10 us van een gestabiliseerde klok. Een camera is nog steeds vergrendeld sinds een eerdere test waarbij PTP aan bleef staan, zoals deze backend het zag, telt vanaf die synchronisatie, dus bij een test direct na een andere hoeft er niet opnieuw te worden gewacht; een camera die door de test wordt ingeschakeld, of een camera die met PTP ingeschakeld is aangetroffen en waarvan deze backend nog niet heeft gezien dat hij gesynchroniseerd is (elders ingeschakeld, of vóór een herstart van de backend), wacht zijn 120 s af, en in een reeks van beide camera’s wachten de net gesynchroniseerde camera’s nog steeds op hun beurt. Een camera die de Slave-modus verlaat, de 100 us overschrijdt of tijdens het wachten drie metingen achter elkaar (1,5 s) een andere grandmaster volgt, begint opnieuw te tellen; één of twee van dergelijke metingen worden geregistreerd (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) en de telling gaat door; een grandmaster-ID dat niet kan worden gelezen, wordt als onbekend beschouwd, niet als een wijziging. Als de telling van alle camera’s tegelijkertijd (binnen een seconde) opnieuw begint, komt dat door de tijd van de grandmaster, niet door die van de camera’s: het logboek van de backend vermeldt `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` vermeldt dit in `grandmaster_steps`, en een wachttijd die vervolgens afloopt, begint zijn foutmelding met `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. De limiet wordt hiervoor niet verlengd. `--ptp-settle SECONDS` stelt de afwikkelingstijd voor de run in (0-600; `0` voor geen afwikkelingswachttijd). De wachttijd is begrensd op de stabilisatietijd plus 90 s (standaard 210 s, minimaal 90 s), en het commando wacht 240 s plus de stabilisatietijd op de backend (standaard 360 s). Het geeft `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)` weer, en het logboek van de backend bevat elke 5 s een regel met de status en offset van elke camera, hoe lang deze al vergrendeld is en hoe lang dit nog duurt: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Elke camera wordt ingeschakeld met PTP uitgeschakeld; vanaf dat moment synchroniseren de klokken binnen ongeveer 10 s. Daarna wordt elke camera teruggezet in de toestand waarin de test hem aantrof — de PTP, de automatische belichting en de versterking worden tijdens de test op ‘Uit’ gehouden, en de rest (tenzij `--keep-settings`) — dus een camera die met PTP aan is achtergebleven (`lattice configure -s <serial> --set PtpEnable=True`, of een testrun met `--keep-settings`) blijft gesynchroniseerd, en zodra deze backend heeft vastgesteld dat deze gedurende de stabilisatietijd gesynchroniseerd is, wacht een latere test hier niet meer op. Als de klokken tijdens het wachten niet tot rust komen, mislukt de test voordat deze wordt geactiveerd (`PTP: NOT locked …`, afsluiten `1`) en geeft aan wat er moet gebeuren: een camera die nooit vergrendeld is (`PtpStatus Listening`) betekent dat de grandmaster deze niet bereikt (`time-sync status`); een camera die steeds uitviel, wordt benoemd met de tijd dat deze vergrendeld was (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); klokken zijn nog steeds aan het convergeren: controleer `time-sync status`, en om ze vóór de test vergrendeld te krijgen, schakel je PTP in en laat je het ingeschakeld (`lattice configure -s <serial> --set PtpEnable=True` voor elke camera, of voer de test uit met `--keep-settings`; de test zet de gevonden instellingen terug, dus het blijft ingeschakeld), voer de test vervolgens uit. |
| `lattice multi-detect [--line LINE] [--json]` | Zoek de camera’s op de M8-synchronisatielijn en een standaardmaster: de RGB-camera (model eindigend op `-FRGB`) als deze is aangesloten, anders de eerste aangesloten camera op serienummer. De M8-kabels komen samen op één knooppunt, dus **elke aangesloten camera kan de master zijn** — de afgedrukte `MASTER` is een standaardinstelling, geen feit over de bedrading. Elke camera stuurt om de beurt de lijn aan en de andere camera’s worden gecontroleerd op de puls; een camera die al in de aangesloten groep is gevonden, wordt niet opnieuw getest, dus een volledig bekabelde opstelling heeft één ronde nodig. Zonder `--line` wordt `Line2` getest, daarna `Line3`. Drukt `MASTER: SN:…` af, één `SLAVE: SN:…` per andere bekabelde camera en `(probed on <line>)`; `--json` voegt het volledige resultaat toe (`master`, `slaves`, `no_cable` — camera’s die de puls niet heeft bereikt — `line`, en per camera `results`). Wanneer dit proces is voltooid, krijgt elke camera de oorspronkelijke instellingen voor de trigger en de synchronisatielijn terug (dezelfde knooppunten als hierboven). |

#### Uitlijning

| Subcommando | Doel |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus detector-/matcher-knoppen `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-regelaars `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinatie van meerdere frames `[--averaging mean\|median\|inlier_weighted]`, geometrische beperkingen `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, ruimtelijke beperking `[--roi X0,Y0,X1,Y1] [--mask PATH]` en per-slave-overschrijvingen `[--per-cam-override SN:KEY=VALUE]` (herhaalbaar) | Bereken uitlijningsprofiel op basis van live camera&#x27;s. Standaardinstellingen: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (in de map van waaruit je het uitvoert), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` heeft zijn eigen standaardinstellingen; zie [de bijbehorende tabel](#project-align-calibrate-options).) `--prefilter` gebruikt standaard `gradient` (randkaart; komt overeen met de GUI/array-aligner — randen blijven behouden over spectrale banden heen). `--matcher flann` levert pas resultaat op bij meer dan ~5000 kenmerken; `--averaging median` is robuust tegen één slechte opname, `inlier_weighted` weegt op basis van het aantal overeenkomsten; `--lock-scale` projecteert naar de dichtstbijzijnde rotatie (geen schaal), `--lock-axis` zet één translatiecomponent op nul; `--mask` is van toepassing op elke camera (gebruik `--per-cam-override` voor instellingen per camera, bijv. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` weigert een kalibratie op te slaan waarvan de RMS van de herprojectie de drempel overschrijdt. `--method manual` haalt zijn punten uit `--points PATH`: een JSON-object `{"<serial>": [[x, y], ...], ...}` dat de pixelcoördinaten van dezelfde kenmerken voor elke camera opgeeft, inclusief die van de referentiecamera (minstens 2 per camera); elk van de `--frames`-profielen wordt op basis van die punten berekend. Wanneer de camera’s in een gekoppelde array zijn opgenomen, maakt de kalibratie gebruik van de live-frames van die array, en wordt het profiel tevens de uitlijning van de array voor de opnames. Anders wordt een [tijdelijke array](#multi-camera-transient-sessions) geopend: een camera die in geen van de `--frames`-frames een beeld levert, faalt bij de kalibratie (verlaat `1` en geef het een naam), en er wordt geen profiel opgeslagen. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Leg één uitgelijnd multiband-frame vast. `--bit-depth` stemt standaard af op de camera; `--no-crop` behoudt het volledige beeld (opgevuld met zwart); `--interpolation` (standaard `linear`) en `--border-mode`/`--border-value` (standaard `constant`/0) regelen de CPU-warp — het GPU-pad is hoe dan ook bilineair. Opent een [transiënte array](#multi-camera-transient-sessions): een camera die geen frame levert, faalt bij de opname (verlaat `1`, met vermelding van de naam), en er wordt niets geschreven. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Stream-uitgelijnde multiband-frames (dezelfde warp-regelaars als `align-apply`), `--fps` standaard `3`. Opent een [transiënte array](#multi-camera-transient-sessions): een framegroep waarvoor een camera niets heeft geleverd, wordt niet opgeslagen, en drie achter elkaar beëindigen de stream. Als er een groep is overgeslagen, wordt de run beëindigd door `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` en `Error: …` af te drukken, en wordt afgesloten met `1`. Een stream die door een andere fout wordt beëindigd, geeft ook `Error: …` weer en wordt afgesloten met `1`. |
| `lattice align-info --profile PATH [--json]` | Profieldetails weergeven. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Volgorde van lagen wijzigen. |

#### Index / Vegetatiewiskunde

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` koppelt de symbolen van een voorinstelling aan banden** (NDVI: `red`, `nir`), elk één `--channel SYMBOL=BAND`.

- **`--live`: alleen voor een symbool waarop de stack geen antwoord kan geven.** Net als bij `--input` krijgt een symbool van een preset dat je weglaat de ene band van de stack (de banden van de ingeschakelde camera&#x27;s van het profiel) waarvan de naam ermee begint. Dit wordt gecontroleerd voordat een camera wordt geopend: een symbool waarop geen enkele band of meerdere banden reageren, wordt dan geweigerd, waarbij de betreffende banden worden genoemd, en `1` wordt verlaten. Een aangepaste `--formula` heeft voor elk symbool een `--channel` nodig.
- **`--live`-producten geven aan van wie ze zijn.** `index.tif` en `index_LUT.png` bevatten de gegevens van de camera in de stapel (merk, model, serienummer), de opnametijd (de belichtingstijd van de referentiecamera, UTC) en de lens, en beide noemen de index als hun enige band (`XMP-Camera:BandName` `NDVI`; de kleuren-PNG-bestanden noemen de index, niet de kleurkanalen), net zoals een `--input`-product dat doet. Een index van banden van meerdere camera’s noemt in plaats daarvan elk van deze banden, en de bijbehorende `Model` is `LATTICE` — zie [Bestanden gemaakt van meerdere camera’s](#files-made-from-several-cameras).
- **`--input`: alleen voor een symbool waarop het bestand geen antwoord kan geven.** Een symbool dat je weglaat, neemt de ene band van het bestand waarvan de naam daarmee begint: `red` neemt `Red_660` of `Red_660_RGN` aan, `nir` neemt `NIR_850` aan. De namen zijn afkomstig uit het bestand zelf: de eigen bandenlijst van de stack, of, bij een Chloros-radiance-/reflectance-opname of een `process`-product, de bijbehorende `Chloros:Band{R,G,B}Name`- en `XMP-Camera:BandName`-tags. `lattice index --input <reflectance capture> --preset NDVI` heeft dus geen `--channel`.
- **Twee banden die bij één symbool horen, worden niet geraden** (een stack met 9 camera’s heeft twee NIR-banden): noem die dan. Een symbool waaraan niets beantwoordt, wordt geweigerd met `channel_map missing entries`, verlaat `1`.**Bandnamen in een stapel** geven het filter van de camera aan: de banden van een M3C-camera zijn `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` op een RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` op een OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` op een RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` op een NGB — en deenkelband is `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` geeft de banden van een profiel weer.

Volledige vlaggenreeks: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (herhaalbaar), `--capture-level raw|debayered|radiance|reflectance|unknown` (het in het bron-TIFF-bestand vastgelegde opnameniveau overschrijven; standaard: afgelezen uit TIFF-metadata, met als terugvaloptie de -waarde uit een Chloros-bestand `Chloros:ProcessingLevel`, dus een reflectantiemeting rapporteert `reflectance`), `--output PATH` / `-o` (offline: een pad; met een afbeeldingsextensie worden de bestanden benoemd op basis van de stam — `-o ndvi_FRGN.png` schrijft `ndvi_FRGN.tif` en `ndvi_FRGN_LUT.png` — en zonder extensie is het een voorvoegsel, `-o ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`; standaard naast de invoer, zoals `<input name>_<preset>`; met `--live` en zonder `-o`, een map `output/` onder de map van waaruit je het commando uitvoert), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Bij `--live` zijn de uitlijningsknoppen ook van toepassing: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (het doel van de automatische belichting van de camera&#x27;s, zie de regels voor tijdelijke sessies), `--serials SN1,SN2,…`. `--live` opent een [tijdelijke array](#multi-camera-transient-sessions): een camera die geen beeld levert, zorgt ervoor dat de run mislukt (verlaat `1` en benoem deze), en er wordt geen stack of index geschreven. Het opent `--serials` (de eerste activeert de trigger), anders de camera’s van het profiel waarvan de `layer_order`-vermelding is ingeschakeld, met de referentiecamera als eerste — de referentiecamera zelfs als de banden zijn uitgeschakeld, aangezien de andere camera’s hierop zijn afgestemd daarop en activeert de trigger. Een camera waarvan de banden zijn uitgeschakeld, wordt nooit gekalibreerd, dus een RGB-camera die in het profiel is uitgeschakeld, faalt niet langer bij een reflectantie-index (`no-absolute-radiance-scale`). Een profiel dat geen camera&#x27;s noemt, opent elke camera op het netwerk, net als voorheen.

Elk formulier draait op de backend, dus deze moet actief zijn: start deze vanuit de Chloros-GUI, of met het commando dat de CLI weergeeft wanneer deze de backend niet kan bereiken. Offline leest de backend `--input` en schrijft de float-TIFF en de ingekleurde PNG zelf, dus een relatief `--input` of `-o` verwijst naar de map van waaruit je het commando uitvoert (zie [Conventies](#conventions)). Beide offline producten bevatten de cameragegevens van de invoer, net zoals bij een `process`-indexexport: merk / model / serienummer, GPS, de lensgegevens, het opnametijdstip (dat van de invoer zelf; wanneer de EXIF-gegevens hierover niets vermelden, het opnamemoment dat is geregistreerd of de tijdstempel in de bestandsnaam, in UTC) en de MAPIR-kalibratie-identiteit — nooit de raw-DN-kalibratie, die niet van toepassing is op een index — plus `Chloros:ProcessingLevel=index` en de indexuitdrukking; de float TIFF-naam verwijst naar één band, de preset. `--list-presets` en `--list-gradients` geven de voorinstellingstabel en de verloopstabel van de backend weer. Een formule die de backend afwijst, of een `--channel`-band die de TIFF niet heeft, verlaat `1` met de reden van de backend.

> **`--channel`-symbolen zijn hoofdlettergevoelig.** De symboolzijde moet exact overeenkomen met de kanaalnamen van de voorinstelling (voorinstellingen gebruiken kleine letters, bijv. NDVI = `red`,`nir` — controleer `--list-presets`), en de bandzijde moet overeenkomen met een bandnaam in de uitgelijnde stack (of een bandindex op basis van 0 zijn in de offline-modus). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` werkt; `--channel RED=660` mislukt met een `channel_map missing entries` foutmelding.

#### `lattice index` — maskers voor multibandstapels

Bij `--capture-level radiance` of `reflectance` kan de 3×3-unmix van elke camera een band op 0 afklemmen (zie [Unmix-clampmasker](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Een multiband-stack bevat banden van verschillende camera’s, en elke camera beperkt zijn eigen band, dus `--live --save-multiband` schrijft **één masker per broncamera** naast de stack:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Elk masker is een enkelband-`uint8` op het pixelraster van de **stack** (via dezelfde uitlijningsvervorming en uitsnede), met de eigen bits van die camera: bit 0 = de band van het R-kanaal, bit 1 = G, bit 2 = B. Een stapelband afkomstig uit kanaal *c* van een camera wordt begrensd waar het bit *c* van die camera (R = 0, G = 1, B = 2) is ingesteld.
- De ingebedde herkomst van elke band benoemt het masker van de camera (`unmix_clamp_mask`, ten opzichte van de map van de stack, of `none`) en het te lezen bit (`unmix_clamp_mask_bit`).
- Een camera waarvan de unmix niets heeft geklemd, krijgt geen bestand; een mono- of RGB-camera voert geen unmix uit en krijgt er ook geen. De CLI geeft elk masker onder de stack weer (`clamp mask SN…: …`); het antwoord vermeldt ze als `clamp_masks`.
- De index-TIFF-/PNG-bestanden van dezelfde run worden berekend op basis van dezelfde stack, op hetzelfde raster, dus de maskers beschrijven deze ook. `--capture-level raw` voert geen unmix uit en schrijft geen masker.
- De stack en de index staan rechtstreeks in `--output`, dus de maskers komen in `--output`’s eigen `clamp_masks/` terecht — nooit boven de map die je hebt genoemd. Die map bevat verschillende soorten bestanden (stack, index, ingekleurde render): selecteer de relevante bestanden hieruit en geef niet de hele map door aan fotogrammetriesoftware.

#### Permanente verbindingen (Smart-Prep, GUI-equivalent Flow)

Deze commando’s houden camera’s open in de backend-pool, ook tussen verschillende CLI-aanroepen door.

| Subcommando | Doel |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Voeg één camera toe aan de pool (één camera, geen array). De [algemene opties](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` en `--packet-size` (`standard` = 1500, `jumbo` = 9000, of bytes; `auto`-probes) worden toegepast bij het aansluiten (een `--exposure` of `--gain` schakelt de automatische modus uit). Het keert terug zodra de stream van de camera is gestart (daar wordt ongeveer 15 s op gewacht, daarna 10 s op de eerste volledige buffer) en geeft `stream: streaming` of `stream: no_frames -- connected, no frames yet: …` weer wanneer de stream niets heeft geleverd — voor een camera die jumbo-pakketten gebruikt, met het advies om opnieuw verbinding te maken met `--packet-size standard`. Terwijl één camera verbinding maakt (bij een eerste verbinding wordt de kalibratie gedownload, ongeveer 70 s), wacht een ander commando dat een camera nodig heeft maximaal 5 s op deze camera, en mislukt vervolgens met HTTP-status 503 `CAMERA_POOL_BUSY`, waarbij de camera die verbinding maakt wordt genoemd; voer het commando even later opnieuw uit. `lattice status` en de cameralijst wachten nooit: een camera die bezig is met verbinden, wordt weergegeven als ‘verbinden’, met de bijbehorende fase. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Vrijgeven. De camera wordt teruggezet zoals deze bij het verbinden werd aangetroffen (triggerbedrading, PTP, instellingen); `--keep-settings` slaat de instellingen in plaats daarvan op in het eigen geheugen, `--clear-settings` reset dat geheugen naar de fabrieksinstellingen (zie [Camerageheugen](#camera-memory----keep-settings----clear-settings)). Een camera die deel uitmaakt van een array maakt de hele array vrij. |
| `lattice cam-list` | Geeft een overzicht van de camera&#x27;s in de pool. |
| **`lattice array-connect`**|**Maak verbinding met een permanente gesynchroniseerde array (HET aanbevolen startpunt).** Voert de volledige GUI-smart-prep-workflow uit. Ongeveer een minuut voor 9 camera’s; direct nadat de camera’s zijn ingeschakeld, kan het enkele minuten duren. Terwijl het wacht, wordt elke stap weergegeven die de backend bereikt, zoals `    <s> s: <step>` (bijvoorbeeld `    41 s: open SN:214000535 (4 of 9)`), en het logboek van de backend bevat één tijdregistratie per stap (`[array-connect <id>] <step>: <s> s …`) en een laatste regel met de traagste stappen. Het wacht maximaal 600 s. Een verbinding die dan nog niet tot stand is gebracht, wordt door de backend ongedaan gemaakt: elke camera die was geopend, wordt teruggezet in de toestand waarin deze werd aangetroffen en gesloten; het commando geeft `Error:` weer met de stap die was bereikt en sluit af met `1`, en er is geen array verbonden (HTTP-status 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C annuleert de verbinding op dezelfde manier (`Interrupted: …`); als de array al verbinding had gemaakt, vermeldt de regel dit en noemt het de `array-disconnect` die deze vrijgeeft. Een commando dat wordt afgebroken terwijl het wacht (`kill -9`) zorgt ervoor dat de verbinding wordt ongedaan gemaakt wanneer de backend 30 seconden lang niets van het commando heeft gehoord. Een array die actief is maar nog geen gesynchroniseerde framegroep heeft, wordt nog steeds als verbonden gerapporteerd, met zijn stream als `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Een array vrijgeven. Elke camera blijft vrij draaien (`TriggerMode` `Off`), waarbij al het andere blijft zoals de verbinding het heeft aangetroffen — de instellingen voor de trigger en de synchronisatielijn (de master’`TriggerSource` terug naar de oorspronkelijke toestand, de synchronisatielijn terug naar een ingang), de PTP-instellingen en de camera-instellingen die tijdens de sessie zijn gewijzigd — zodat een commando voor één camera op een lid onmiddellijk werkt. `--keep-settings` slaat de instellingen van elke camerainstellingen zoals de sessie ze heeft achtergelaten in het eigen geheugen van de camera; `--clear-settings` zet dat geheugen terug naar de fabrieksinstellingen. Zie [Camerageheugen](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Lijst met aangesloten arrays. |
| `lattice array-status [--array-id ID]` | Live fps, PTP, laatste fout. `ptp_enabled` is alleen `True` wanneer elk lid PTP als ingeschakeld aangeeft; `ptp_off:` noemt een lid dat dit niet doet. De `state:`-regel leest `restarting (applying settings)` tijdens een herstart van de instellingen, `recovering (…)` terwijl een lid dat de stroomvoorziening verloor opnieuw wordt geopend (zie de opmerking hieronder), `stalled (…)` terwijl de grab-loop draait maar geen framegroep publiceert, en `stopped (…)` zodra de grab-loop het heeft opgegeven (sluit de array opnieuw aan). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Eén gesynchroniseerde opname van de live-array — Enkel / Continu / Interval / Snelste. **Standaard ingesteld op `all`**(één bestand per toepasselijk exporttype per camera),**uitgelijnd** zoals de opname van de app (`--no-aligned` voor native bestanden). Overgeslagen camera’s (bijv. RGB, uitgesloten van radiance/reflectance) worden gerapporteerd met `Skipped: SN:<serial> (<reason>)`; de DAQ-waarde die de gebruikte opname uitleest, wordt ernaast opgeslagen, inclusief een opname die uitsluitend uit ruwe gegevens bestaat, en gerapporteerd met `DAQ: <path>` ([De DAQ-waarde die een opname uitleest, wordt opgeslagen](#de-daq-die-een-opname-leest-slaat-op)). Zie [Opnamemodi, recorders en offline herverwerking](#capture-modes-recorders--offline-herverwerking). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Neem de live gecombineerde-indexweergave op als video/GIF (monitoringkwaliteit; zonder geopende GUI-weergave geeft de backend de NDVI van de array weer, of `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Raw-Bayer-burst met hoge fps (analysekwaliteit; offline opnieuw verwerken). Net als `array-record` en `array-capture --continuous` meldt deze zich bij elke poll bij de backend: wordt beëindigd zonder te stoppen (`kill -9`), de backend stopt het 30 s na de laatste check-in (`stop_reason` `client_gone`; het manifest van de burst wordt afgerond en de opbouw van de gecombineerde index wordt gestart, zoals bij een stop gebeurt) en de array blijft verbonden. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Een opgeslagen onbewerkte burst opnieuw verwerken tot gekalibreerde video(&#x27;s). |

> **Een camera die de stroomvoorziening verliest of waarvan de verbinding wegvalt, komt vanzelf weer terug.** Wanneer een lid niet meer reageert (een schakelaar die wordt uit- en weer ingeschakeld, start elke camera die hij van stroom voorziet opnieuw op), wacht de array op het lid, opent de verbinding opnieuw zodra het weer reageert, en geeft het terug wat `array-connect` heeft ingesteld: PTP, de trigger- en synchronisatielijn, netwerkpacing, pixelformaat en framegrootte, en automatische belichting. Vervolgens wordt er weer gestreamd zonder nieuwe `array-connect`. Ondertussen toont `array-status` `state: recovering (...)` en geeft `last_error` aan op welke camera wordt gewacht; daarna wordt `re-opened:` weergegeven. Een camera die opnieuw is opgestart, komt terug met PTP uitgeschakeld, dus totdat deze opnieuw wordt geopend, is `ptp_enabled` gelijk aan `False` en noemt `ptp_off:` de camera bij naam.

> **Een vastgehouden camera wordt nooit een tweede keer geopend.** Een tweede keer openen stopt de stream van de houder en herschrijft de instellingen ervan, en kan de backend laten crashen. Dus bij een camera die door de pool wordt vastgehouden (de GUI of `cam-connect`), een array-element, of een camera die door een achtergrondtaak wordt vastgehouden (een actieve `lattice continuous`, `multi-stream` of `align-stream`, van zichzelf of van een andere client), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, een-shot `white-balance` en `presets --apply` weigeren met HTTP-status 409 `CAMERA_HELD`, waarbij de houder en het vrijgavecommando worden vermeld (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, of voor een taak het ID en wie deze heeft gestart), en afsluiten `1`. Hetzelfde geldt voor een camera die op dit moment door een ander commando wordt gebruikt (een `multi-test` of `multi-capture` die zijn camera’s opent, nog een eenmalige actie), voor elk commando hier: de foutmelding geeft aan dat je moet wachten tot dat commando is voltooid. `lattice power` zonder `--sleep` doorzoekt de eigen handle van de houder, indien aanwezig (de pool, een array, de geopende camera van een taak), en wordt geweigerd voor een camera die door een andere opdracht wordt gebruikt. Elk `CAMERA_HELD`-antwoord bevat de code in zowel `code` als `error_code`. Zie [Multi-Camera (Transient Sessions)](#multi-camera-transient-sessions).

##### `array-connect`-opties

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--serials SN1,SN2,…` | alle LATTICE-camera’s automatisch detecteren (minimaal 2 vereist) | De eerste in de reeks is de MASTER. Indien weggelaten, filtert de detectie op LATTICE-modellen (M3C / M3M) en worden deze allemaal aangesloten. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synchronisatielijn. |
| `--target-fps F` | auto | Vuurfrequentie van de MASTER-trigger. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | De tier-kiezer overschrijven. |
| `--wire-ceiling-mbps MB_PER_S` | automatisch gedetecteerd | **Het aanhoudende bandbreedtebudget van de host, in MB/s — het getal waaraan de toewijzing van de gehele array is gekoppeld.** Verlaag deze waarde wanneer de array GVSP-beschadigde frames rapporteert: de automatische waarde wordt afgeleid van de door de NIC gerapporteerde verbindingssnelheid, die USB-adapters, smalle PCIe-lanes en drukke gedeelde fabrics overschat. Wordt opgeslagen in het array-capture-blok van het project, zodat het bij het opnieuw openen of bij een herverbinding via de CLI of SDK wordt hersteld. Zie [Arraystatus](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | geen (volledige sensor) | Hardware-binning (altijd Gemiddeld). |
| `--width N` / `--height N` / `--pixel-format FMT` | volledige sensor; formaat per camera | Een expliciete framegrootte of pixelformaat voor elke camera, in plaats van de keuze tussen de volledige sensor en het formaat per camera (Mono12 / BayerRG12). |
| `--no-recommend` | uit | De netwerkanalyse overslaan (elke camera behoudt dan zijn huidige formaat in plaats van te worden ingesteld op de volledige sensor). |
| `--no-ptp` | uit | PTP uitschakelen (tijdstempels tussen camera&#x27;s zijn dan **niet** vergelijkbaar). |
| `--packed {auto,on,off}` | `auto` | 12-bits frames gecomprimeerd streamen en op deze computer decomprimeren — meer frames per seconde bij volledige sensor, identieke gegevens. `auto` comprimeert alleen wanneer het nuttig is en deze computer het aankan; `off` verpakt nooit. Zie [Verpakt 12-bits transport](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | GVSP-pakketgrootte voor elke camera in de array. `auto` laat elke camera het netwerk verkennen; `jumbo` = 9000, `standard` = 1500, of een grootte in bytes beperkt dit. Vóór versie 1.3.0 accepteerde `array-connect` deze vlag en negeerde deze. |

> **Standaard volledige sensor.** Zonder `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` stelt elke camera in op de volledige sensor — geen
> binning, geen ROI-uitsnede — en laat de beeldsnelheid dalen tot wat deze computer en
> het netwerk aankunnen, in plaats van de beelden te verkleinen om de snelheid hoog te houden. De
> regel met netwerkanalyse toont de verwachte snelheid, en de volgende regel geeft een overzicht van
> snellere opties (binning of een kleinere ROI) met hun verwachte snelheden; sla
> `--binning` of `--width`/`--height` om er een te selecteren. Connect voert nooit zelfstandig binning of uitsnijdingen uit
> .

##### Gepakt 12-bits transport (`--packed`)

Bij volledige sensorbemanning wordt de framesnelheid van een array bepaald door hoe snel elke camera
zijn frames kan verzenden. 12-bits formaten (`BayerRG12`, `Mono12`) worden normaal gesproken verzonden als 16-bits
woorden; **gecomprimeerde** overdracht verzendt dezelfde 12 bits in 1,5 bytes per pixel en
pakt ze op deze computer weer uit, waardoor het verzenden van elk frame 25% minder tijd kost —
ongeveer **33% meer frames per seconde bij volledige sensor** (gemeten: 10,7 → 14,0 fps
op een array met 9 camera&#x27;s). **De opgeslagen gegevens zijn identiek**: dezelfde waarden, dezelfde
16-bits bestanden, dezelfde metagegevens.

- `auto` (standaard) pakt alleen in als dat voordelen oplevert **en** deze computer het
  uitpakken aankan. Bij het verbinden meet de backend zijn eigen uitpakkingssnelheid, het aantal CPU-kernen,
  de huidige CPU-belasting en het netwerkbudget, zodat een desktop met voldoende vrije kernen
  bij volledige sensorgrootte comprimeert, terwijl een drukke of energiezuinige computer ongecomprimeerd streamt. Kleine ROI’s en gebundelde frames leveren
  weinig.
- `off` comprimeert nooit — gebruik dit om compressie uit te schakelen tijdens het diagnosticeren, of op een
  computer die al zijn CPU-capaciteit voor ander werk moet reserveren.
- `on` comprimeert altijd 12-bits formaten.

De connect-uitvoer toont de beslissing en de reden daarvoor (`gecomprimeerd 12-bits: op (auto:
verwacht 11,0 -&gt; 14,4 fps, …)`); `/api/camera/array/status` rapporteert
`packed_transport` en `packed_decision`. Om een standaardinstelling voor de hele machine in te stellen
in plaats van de vlag elke keer door te geven, stel je `CHLOROS_ARRAY_PACKED12=off` (of
`on`) in de omgeving van de backend — een expliciete `--packed on|off` heeft nog steeds voorrang.
8-bits en 16-bits formaten worden nooit gecomprimeerd.

### Eén projectrecord, elke modus

Een projectmap bevat de instellingen van elk apparaat: `cameras.json` — die van elke camera (`cameras[].settings`, voor een enkele camera en voor elk array-lid) en de instellingen die voor het hele array gelden (`arrays[].capture`: formaat, ROI, binning, triggerfrequentie, wire budget) — en `sensors.json`, die van elke lichtsensor (integratietijd, framegemiddelde, limiet). De app, de CLI, de SDK en de MCP-server lezen en schrijven allemaal dezelfde bestanden, op dezelfde manier:

- **Een apparaat dat onder een project is aangesloten, wordt daarin opgeslagen en registreert elke wijziging daarin.** &quot;Onder een project&quot; betekent dat er een project geopend is in de backend (die van de app), of — vanuit de SDK — via een `ChlorosProject`, dat zijn eigen map specificeert. Een camera, array of lichtsensor die het project nog niet bevat, wordt aan de bestanden toegevoegd wanneer deze verbinding maakt (`lattice cam-connect`, `array-connect`, `daq pool-connect`, de app, de SDK, `chloros-mcp`), samen met de bijbehorende instellingen. Een wijziging vanuit een willekeurige modus (`lattice exposure`, `gain`, `configure --set`, `resolution`, de instellingen van een arrayinstellingen en triggerfrequentie, de limiet van een lichtsensor, de bedieningselementen van de app) wordt even later opgeslagen in de bestanden van dat project. Een belichting, versterking of AE-doelwaarde die op een camera is ingesteld, wordt opgeslagen als ‘vastgehouden’, zodat deze bij het opnieuw openen weer wordt gehandhaafd.
- **Bij het openen van een project worden de apparaten daaruit ingesteld**, in elke modus: een camera, array of lichtsensor die is aangesloten zonder eigen instellingen, krijgt de opgeslagen instellingen van het project (`lattice cam-connect` / `array-connect` met een geopend project, `chloros-cli project connect`, `open_project().connect_all()`, het opnieuw openen van de app). Een array die onder een nieuwe id wordt aangesloten, werkt de array bij die het project met dezelfde camera’s heeft opgeslagen; een array met andere camera’s wordt ernaast toegevoegd.
- **De eigen opslag van de app overschrijft nooit de live-instellingen van een apparaat** met een oudere kopie daarvan, en behoudt per lid opgeslagen overschrijvingen van een CLI- of SDK-aanroeper.
- **Het pixelformaat dat u instelt, is het formaat dat een project behoudt.** Een formaat dat is ingesteld in Array-instellingen, via `--pixel-format` of via `connect_array(pixel_format=)`, inclusief 8-bit, wordt opgeslagen en bij het opnieuw verbinden exact overgenomen. Als er niets is ingesteld, maakt elke camera verbinding met de standaard 12-bits instelling (`BayerRG12`, `Mono12` op een monosensor). Een formaatnaam verwijst naar de bitdiepte; de familie is die van de sensor, dus een kleurencamera wordt nooit ingesteld op een Mono-formaat of een monochrome camera op een Bayer-formaat, en de regel `[PixelFormat <serial>] ... written` geeft dit aan wanneer een formaat is toegewezen. De exportbestanden van een 8-bits opname worden geëxporteerd als 8-bits, inclusief reflectie: op schaal gebracht en voorzien van de aanduiding `PixelScale`.
- Een apparaat dat niet aan een project is gekoppeld, neemt niets op.
- **Een opname volgt een project dat erna is aangemaakt; bij het sluiten blijft het gekoppeld aan het project.**Wanneer een project wordt aangemaakt — een SDK-project `create_project()`, `chloros-mcp` of het**Nieuw project**— of gedupliceerd (de**Dupliceren**-functie van de app, waarmee de kopie wordt geopend) terwijl een opname nog steeds naar het eerder geopende project schrijft, gaat de opname door en komt wat er wordt geschreven terecht in het nieuwe project, op dezelfde plaats daarin (hieronder). Het openen van een bestaand project — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, de functie**Project openen** van de app — zorgt ervoor dat een opname nog steeds doorloopt in het project waarin deze wordt opgenomen: de opname wordt daar voltooid en de bestanden worden toegevoegd aan de bestandslijst van dat project (de app stopt elke opname voordat een project wordt geopend).
  - `array-record` en `array-burst`: de video, of de burst-map met alle frames, wordt in zijn geheel daarheen verplaatst wanneer de opname stopt (het is `--duration`, Ctrl+C). Het `Burst dir` dat wordt weergegeven, is het nieuwe bestand, en `--build` en `array-build-video` werken hierop voort. Als de schijf de verplaatsing weigert, blijft de opname staan waar deze is geschreven; hetlogboek vermeldt de reden, net als het HTTP-antwoord bij het stoppen (`relocation_error`). De ruwe frames van een gestopte burst en `.daq` worden toegevoegd aan de bestandslijst van het project waarin ze terechtkomen; een video (een `array-record`, de samengestelde video van een burst) wordt niet toegevoegd aan de bestandslijst van een project.
  - `array-capture --continuous`: elke opname na de wijziging wordt opgeslagen in het nieuwe project; de opnames die eerder zijn opgeslagen, blijven waar ze zijn.
  - `daq pool-record` en `project sensor log` (en de opname via de eigen lichtsensor van de app): het bestand `.daq` wordt bij het stoppen van de opname verplaatst naar demap `light_sensor` van het nieuwe project wanneer de opname stopt, en wordt toegevoegd aan de bestandslijst van dat project.

  Een opname in een map buiten het geopende project (een `-o` ergens anders) blijft waar het is, en `lattice continuous` en `multi-stream` behouden de map die ze toegewezen kregen.
- **In de app worden de opnames na het aanmaken van een nieuw project daarin opgeslagen.** Bij het aanmaken of dupliceren van een project blijven alle opnamesequenties en opnames doorlopen, en elke opname die daarna wordt gemaakt (Alles vastleggen, Beginnen met vastleggen) wordt opgeslagen in het nieuwe project en toegevoegd aan de bestandslijst daarvan. Het openen van een bestaand project in de app stopt eerst alle opnames.
- **Een project wordt niet verplaatst terwijl het aan het opnemen is.**Het verplaatsen van het geopende project naar een andere werkdirectory (via**Instellingen → Werkdirectory** in de app; geen enkel CLI-commando verplaatst een project, een HTTP-client die naar `/api/move-project` post doet dat wel) wordt geweigerd met HTTP 409 `PROJECT_RECORDING` terwijl een array-opname of burst, een continue array-opname, een `lattice continuous` / `multi-stream`-taak of een opname via de lichtsensor ernaar schrijft: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` De app weigert op dezelfde manier, voordat er om een map wordt gevraagd, terwijl de eigen functies ‘Alles vastleggen’, ‘Beginnen met vastleggen’ of een opname via de lichtsensor actief zijn.

Dit staat los van de instellingen die de camera’s behouden bij het verbreken van de verbinding (zie [Camerageheugen](#camera-memory----keep-settings----clear-settings)): een project stelt zijn camera’s zelf in bij het openen.

### Camerageheugen — `--keep-settings` / `--clear-settings`

Wanneer een array is aangesloten, nemen de camera’s de instellingen van de array over, en kun je deze vervolgens wijzigen. Wanneer Chloros-camera’s loskoppelt — `array-disconnect`, het einde van een tijdelijke opdracht (`multi-capture` zonder array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), het stoppen van `lattice continuous`, het einde van een `lattice capture` van een camera die het niet in bezit heeft — het zet elk onderdeel terug zoals het werd aangetroffen vóór de verbinding: de bedrading van de trigger en de synchronisatielijn, PTP- en camera-instellingen (de volledige lijst staat onder [Multi-Camera (Transient Sessions)](#multi-camera-transient-sessions)).

Om de camera’s buiten een project te gebruiken, schrijven twee vlaggen bij elk van deze commando’s naar het **eigen geheugen** van de camera — de instellingen waarmee deze opstart, ongeacht de software:

- **`--keep-settings`** slaat de instellingen van elke camera op zoals ze bij het afsluiten van de sessie in het geheugen stonden (UserSet2, de opstartset). De camera’s blijven vrij draaien (`TriggerMode` `Off`), met de synchronisatielijn-bedrading zoals deze is aangetroffen. Een camera zonder UserSet2 slaat de instellingen op in UserSet1; deze computer onthoudt dit, zodat de zelfherstellende functie van Chloros’ array — waardoor een UserSet1 die is afgeweken van de basislijn van Chloros wordt gereset — laat deze met rust.
- **`--clear-settings`** reset het geheugen van elke camera naar de fabrieksinstellingen: de basislijn van Chloros, die door het zelfherstel van de array wordt vastgelegd — fabrieksinstellingen met `TriggerMode` `Off`, `TriggerSource` `Line2`, automatische belichting, de gekalibreerde witbalans van een RGB-camera blijft behouden — in UserSet1, dat de opstartset vormt.
- De twee kunnen niet worden gecombineerd. Elke camera krijgt een regel: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` of `SN:…: camera memory reset to the factory settings.`; het antwoord `camera_memory` is per camera hetzelfde.

De app biedt deze vlaggen niet aan.

**Elke uitgang zet de camera’s terug.**Dezelfde terugzetting zoals gevonden wordt uitgevoerd op elk uitgangspad, niet alleen bij de bovenstaande commando’s: `cam-disconnect` (en de camera**X**van de app); de**X** van de app op een array of op de rij van een willekeurig array-element (een array kan niet verder draaien zonder een element, dus wordt de hele array vrijgegeven); het sluiten van de app (deze geeft de backend maximaal 20 s de tijd om elke camera, array en het actieve `lattice continuous` terug te zetten voordat deze wordt gestopt; een commando dat dan nog actief is — een tijdelijk commando dat zijn camera’s opent of wacht op hun PTP-klokken, een eenmalig `lattice capture`, `power`, `white-balance` of `presets --apply` — stopt bij het volgende veilige punt, zet eerst zijn camera’s terug, gedurende maximaal 8 s, en beantwoordt HTTP-status 503 met `SHUTTING_DOWN`; een nog actieve connect-array — `lattice array-connect`, de connect van de app — stopt vóór de volgende camera en sluit de geopende camera’s weer af, gedurende maximaal 10 s, en geeft het antwoord HTTP 503 `SHUTTING_DOWN`; en een verzoek dat binnenkomt tijdens het afsluiten opent niets: HTTP 503 `SHUTTING_DOWN`); en een CLI-commando dat wordt afgebroken of waarvan de terminal sluit (SIGTERM / SIGHUP stoppen de taak — en de camera’s weer sluit — voordat het afsluit (`143` / `129`, zoals Ctrl+C). Een `lattice continuous`, `multi-stream` of `align-stream` die direct wordt beëindigd (SIGKILL), waardoor niets kan worden gestopt, wordt door de backend 30 s na de laatste check-in. Een backend die crasht of geforceerd wordt beëindigd, kan niets terugzetten; door de stroom uit en weer in te schakelen keert elke camera terug naar zijn geheugen, en de volgende Chloros-verbinding koppelt een camera die hij als geactiveerd aantreft opnieuw aan.**Een project stelt zijn camera’s zelf in.** Wanneer een array verbinding maakt terwijl er een project openstaat, krijgt elk lid de instellingen die het project ervoor heeft opgeslagen — een belichting, versterking of AE-doelwaarde die erop is vastgehouden (vastgehouden als de jouwe, zodat automatische belichting deze met rust laat) en de witbalansmodus (correctie door Chloros; de eigen witbalans van de camera blijft uitgeschakeld) — wat het geheugen van de camera ook bevat.

### Smart-AE / Smart-Capture

LATTICE-arrays voeren continu AE uit op de achtergrond zodra ze zijn aangesloten, maar een pas ingestelde scène heeft even tijd nodig om te convergeren. `array-capture --smart` is de **kant-en-klare oplossing**: het wacht tot de AE zich voor elke camera in de array heeft gestabiliseerd en activeert dan de opname. Gebruik dit wanneer je tijdens een sessie van scène wisselt.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Het stabilisatiebeleid is standaard conservatief: time-out van 5 s, stabiliteitsvenster van 1,5 s, tolerantie voor belichtingsspreiding van ±5 %. Pas dit aan via de SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) als je een ander gedrag van de automatisering nodig hebt.

#### Hoe automatische belichting werkt

- **Eerst belichting, daarna versterking.** Als de versterking op ‘Auto’ staat, blijft deze op 0 dB totdat de belichting de langst toegestane waarde heeft bereikt.
- **Bij een handmatige belichting blijft de versterking op ‘Auto’ staan** totdat je de versterking wijzigt of op ‘handmatig’ zet. Terug op ‘Auto’ is de limiet van de versterking weer zoals deze eerder was.
- **Kleine hoge lichten maken de scène niet donkerder.** Een spiegeling tot 1 % van het beeld wordt genegeerd, en bij panelen van een bekend kalibratiedoel tot 0,5 % van een paneel.
- **De DAQ-belichtingslimiet is alleen van toepassing zolang een kalibratiedoel in beeld is.** Dit voorkomt dat een wit paneel overbelicht raakt; als er geen paneel zichtbaar is, stelt de camera de belichting in op de scène.
- **Constant licht blijft stabiel, en veranderd licht stabiliseert zich binnen enkele seconden.** Bij constant licht blijven de belichting en de limiet ervan ongewijzigd, en na een grote verandering in het licht stabiliseert het beeld zich binnen enkele seconden. Een clip beperkt de belichting niet langer permanent, en bij een verandering van het ROI-gebied wordt de belichting opnieuw bepaald.
- **Het helderheidsdoel blijft binnen bereik.** Het stijgt niet zolang de camera zich op de belichtingslimiet bevindt en de versterking niet kan toenemen, zodat terugkerend licht geen overbelichting in het beeld veroorzaakt.
- **Een camera waarvan de automatische belichting niet meer reageert, blijft op het niveau staan dat past bij het licht waarin hij zich bevindt.** In fel zonlicht kan een camera terugvallen naar zijn kortste belichtingstijd en daar blijven met een donker beeld, terwijl alle instellingen normaal blijven. Chloros houdt het beeld in de gaten, houdt die camera op het niveau dat hij voor dat licht had ingesteld, en laat hem los wanneer het licht verandert. Een donkere of afgedekte lichtsensor heeft hier geen invloed op.

### Opname-exportniveaus (de standaardinstelling van `all`)

Vanaf deze release, `lattice capture`, `lattice multi-capture` en `lattice array-capture` **standaard ingesteld op `--processing all`** — één opgeslagen bestand per exporttype dat van toepassing is op elke camera, in overeenstemming met het „Capture All“-gedrag van de GUI. De niveaus zijn:

| Niveau | Uitvoer | Van toepassing op |
| --- | --- | --- |
| `raw` | Eénkanaals Bayer (monochrome camera’s: de enkele band) rechtstreeks vanaf de sensor. | Alle camera’s. |
| `debayered` | 3-kanaals kleur, opgeslagen in **RGB**-kanaalvolgorde (een TIFF met RGB-fotometrische interpretatie; PNG/JPEG worden eveneens als RGB geopend): de donkere offset (DSNU) van het kalibratiepakket wordt van de raw-gegevens afgetrokken, gevolgd door OpenCV’s rand-bewuste demosaic van OpenCV — geen lineariteits-LUT, flat field of unmix (mono-camera’s: 1-kanaals grijswaarden). De in het geheugen gedebayerde frames die de Python-SDK retourneert (`grab()`, `frame_stream()`) zijn**BGR**, de volgorde van OpenCV; alleen het opgeslagen bestand is RGB. | Alle camera’s. |
| `radiance` | float32 W/m²/sr/nm via de volledige radiometrische keten. | Alleen multispectraal (M3C/M3M) — **overgeslagen voor RGB-filtercamera’s**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-ready. | Alleen multispectraal, en **alleen wanneer een DAQ is gekoppeld + de camera is gekalibreerd**; anders overgeslagen. |
| `display` | Volledige GUI-voorbeeldketen (CCM + WB + gamma volgens het profiel van de camera). Opgeslagen in een `preview/`-map. `lattice capture` accepteert hiervoor ook de naam `preview`; `array-capture` / `multi-capture` accepteren alleen `display`. | Alle camera&#x27;s. |

Geef één niveau op om alleen dat ene niveau op te slaan (`--processing debayered`). Wanneer je om `all` vraagt, worden niveaus die niet van toepassing zijn op een bepaalde nok overgeslagen (en gerapporteerd), maar er treedt geen fout op — een niet-aangesloten of niet-gekalibreerde camera krijgt nog steeds `raw` / `debayered` / `preview`.

#### De DAQ-meting die een opname opslaat

Telkens wanneer een DAQ aan de camera of array is gekoppeld, of in een pool is opgenomen, wordt de downwelling-meting die voor een opname is gebruikt, weggeschreven naar een **`.daq`**-bestand in de submap `daq/` van de map ‘captures’, met de `.match.json` (hoe ver de meting verwijderd was van de belichting) en de `.cameras.json` (van wie de meting is), en gerapporteerd op een `DAQ:`-regel. Dat geldt voor elk niveau, inclusief een onbewerkte of gedebayerde opname, zodat elke opname later opnieuw kan worden verwerkt tot radiance, reflectance of een index. Het is hetzelfde bestand, met dezelfde naam en sidecars, of de opname nu afkomstig is van de GUI, de CLI of de SDK. Zonder enige DAQ-meting is er geen `.daq`, en een `multi-capture` zonder gekoppelde array (de tijdelijke terugvaloptie) slaat niets op.

| Vlag | Effect |
| --- | --- |
| `--no-daq` | Sla de meting niet op bij een opname waarvan de niveaus deze niet gebruiken (`raw`, `debayered`). Een `radiance`-, `reflectance`- of `display`-niveau schrijft deze waarde nog steeds, omdat bij het opnieuw importeren de meting nodig is waarmee het product is gemaakt. |
| `--daq-tolerance-ms MS` | Hoe ver de meting van de belichting mag afwijken en nog steeds binnen de tolerantie valt (standaard 500). Alleen het oordeel verandert (de `.match.json`&#x27;s `in_tolerance` en `tolerance_ms`, de reflectantie&#x27;s `DlsInTolerance`), nooit de gebruikte meetwaarde. Zie [Een DAQ-meetwaarde ver van de belichting](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | De oudere manier om de `.daq` op te vragen bij een opname die uitsluitend uit RAW-gegevens bestaat. Dit is nu de standaardinstelling, en `lattice capture` en `array-capture` accepteren de vlag nog steeds (`multi-capture` niet). `--no-daq` heeft er voorrang op. |

Welke DAQ: degene die aan de camera of array is gekoppeld (de selectie van de lichtsensor in de GUI, een opgeslagen koppeling van een project, of `chloros-cli daq pool-bind`), of anders de meest recente die de backend heeft. Een raw-burst (`array-burst`) behoudt zoals voorheen zijn eigen `.daq` per afzonderlijke meting in de burst-map.

**PNG- en JPEG-bestanden bevatten 8 bits per kanaal, dus ze gebruiken alleen de `display` / `preview` en indexproducten.** `--format png` of `jpg` met een niveau dat ze niet kunnen bevatten, worden geweigerd voordat er iets wordt vastgelegd, waarbij het niveau en TIFF als het formaat dat dit wel kan bevatten, worden vermeld:

- `radiance` (float32) en `reflectance` (uint16, `32768` = 1,0) — altijd, dus `--processing all` (de standaardinstelling) met `--format png`/`jpg` wordt ook geweigerd: gebruik `--format tiff` of `--levels display`.
- `raw` en `debayered` van een camera die meer dan 8 bits streamt (de 12-bits M3C / Mono12 M3M-frames) — de CLI kan de bitdiepte van de camera niet herkennen, dus de backend weigert deze (HTTP-status 400). Het enkele-frame `lattice capture --processing debayered` haalt een 8-bits frame op voor PNG/JPEG en wordt niet geweigerd, evenmin als een `multi-capture` zonder persistente array (8-bits debayered).

Elke opnamemethode reageert op dezelfde manier (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, en de HTTP-routes daarachter.

### Hoe een opnamemap eruitziet

Elk exporttype wordt in een **eigen submap**onder `-o` opgeslagen, zodat er bij een opname met meerdere niveaus nooit verschillende typen door elkaar komen. De schets toont de**standaard `all`-opname van één camera door `lattice capture`** — meerdere niveaus per camera, dus elk bestand heeft een `_<level>`-achtervoegsel:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Hoe de bestandsnamen variëren:

- **Cameraveld — serienummer, gevolgd door filter:** elk opnamebestand noemt de camera die de opname heeft gemaakt als `SN<serial>-<filter>`, waarbij `<filter>` het filtergedeelte van het cameramodel is: `SN261902142-F450` voor een `M3M-L87-F450`, `SN213800234-FRGN` voor een `M3C-L87-FRGN`, `SN214701292-FRGB` voor een RGB-camera. Het filter voegt het serienummer samen met `-`, niet met `_`, dus een naam heeft dezelfde door `_` gescheiden velden als voorheen (voorvoegsel, datum, tijd, milliseconden, camera, niveau). Een camera waarvan het model niet kan worden gelezen (een gewiste DeviceUserID) krijgt alleen de naam `SN<serial>`. Opnames uit eerdere versies hebben geen filter in de naam; Chloros leest beide.
- **Voorvoegsel:** `lattice capture` noemt bestanden `capture_…`; `array-capture`, `multi-capture` met een gekoppelde reeks, en `project capture` / `burst` / `stream` van een reeks noemt ze `sync_<ts>_SN<serial>-<filter>…`.
- **Niveau-achtervoegsel alleen bij meerdere niveaus.**Een camera die**meerdere**niveaus in één opname opslaat (`all` of `--levels a,b`) voegt `_<level>` toe. Een camera die**één** niveau, behoudt de gewone naam, zonder achtervoegsel — `--processing reflectance` schrijft `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — zodat scripts die zijn geschreven voor opnames met één niveau blijven werken. Het niveau wordt nog steeds aangegeven door de map.
- **Clamp-maskers volgen de naam van het product:** `<product name>_clampmask.tif`. Een reflectiemeting met één niveau schrijft `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; een `all`-meting schrijft `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **De extensie** volgt op `--format`: de standaard `tiff` levert `.tiff` op, `--format tif` levert `.tif` (clamp-maskers zijn altijd `.tif`).
- **Eén uitzondering:** `lattice capture --processing debayered` (of `--levels debayered`) neemt een snel enkel-framepad en schrijft `capture_<ts>_SN<serial>-<filter>.tiff` rechtstreeks naar `-o` — geen submap.
- **Streams en bursts** bevatten hetzelfde veld: `lattice continuous` schrijft `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` en een onbewerkte burst `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` is de tijdstempel van de opname en `<serial>` het serienummer van de camera, dus één gesynchroniseerde groep deelt een
tijdstempel over alle camera’s heen. **Let op de enige asymmetrie:** het `display`-niveau wordt opgeslagen in een map
met de naam `preview/`, terwijl de bestanden met meerdere niveaus `_display` in de naam — de map en het achtervoegsel verschillen
alleen voor dat niveau. Onbekende niveaus vallen terug op een map met hun eigen naam, en als de submap
niet kan worden aangemaakt, wordt het bestand naar de output-root geschreven in plaats van verloren te gaan.

**Een opnamemap opnieuw verwerken:**richt `chloros-cli process` op de**captures-root**
(`output/`). `process` importeert normaal gesproken alleen de map die je opgeeft, maar wanneer die map geen
afbeeldingen bevat en wel submappen heeft, gaat het automatisch de submappen in — zodat de submappen op elk niveau en de
`daq/`-submap in één keer worden opgepikt. Elk niveau van een opname wordt geïmporteerd als één afbeelding, waarbij
de andere niveaus beschikbaar zijn als modi, in plaats van als één afbeelding per niveau.

Het werkt ook om een **submap op niveau** direct te benoemen (bijv. `output/raw/`). Hierdoor blijft `daq/`
achter, dus kopieer of wijs de DAQ-uitlezing ernaast aan wanneer jeeen radiometrisch
product afleidt uit `raw/` — anders kan de tijdstempelovereenkomst nergens aan worden gekoppeld.

**De verwerking begint altijd bij `raw`.** Binnen elke opname is het ruwe beeld de bron voor de verwerkingspijplijn;
`debayered`, `radiance`, `reflectance` en `preview` worden weergegeven als weergavemodi, maar worden nooit
teruggevoerd door de pijplijn. Bij het opnieuw verwerken van een afgeleid product zouden vignettering, CCM en
stralingsberekeningen die al in de pixels zijn ingebakken opnieuw worden toegepast, dus Chloross weigert dit in plaats van
een dubbele verwerking uit te voeren. Twee gevolgen die het vermelden waard zijn:

- De `index/`- en `composite/`-renders worden **nooit** verwerkt. Het zijn uitvoerbestanden, geen opnames —
  een NDVI-LUT-weergave heeft geen zinvolle interpretatie van de straling.
- Een map met opnames die is geëxporteerd **zonder** `raw` (bijv. `array-capture --processing reflectance`) heeft
  geen geldige pijplijnbron. Die opnames worden normaal geïmporteerd en weergegeven, maar `process` slaat
  ze over en geeft dit aan:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Als je echt een afgeleid product moet doorsturen — een hub-sessie die is vastgelegd met
  `demosaic` ingeschakeld, of een legacy-map — dwingt `--input-level {raw,debayered,processed}` het
  ingangspunt af en negeert de overslaan-instelling. Die vlag is de bewuste nooduitgang; `auto` (de standaardinstelling)
  verwerkt nooit een opname zonder raw-gegevens.

**Kalibratietags: alleen bij `raw`.** Het `raw`-bestand van een opname bevat het kalibratiepakket van de camera: de `CalibrationJSON`-payload en de MicaSense-compatibele tags (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), die de raw-waarden van de sensor beschrijven. Alle andere bestanden op elk niveau (`debayered`, `display`, `radiance`, `reflectance`, de weergave `index/`) en elk `lattice continuous` / `multi-stream`-frame bevat de identiteit van de camera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) en het eigen record van het frame, `XMP-mapir:FrameJSON` (belichting, versterking, temperatuur, pixelformaat, opnametijd, DAQ-match), aangezien elk `process`-product deze identificatie draagt. `process` zoekt de kalibratie van zo’n bestand op basis van het serienummer en `CalBundleSha256` in de kalibratiecache van Chloros, waarbij het pakket één keer wordt gedownload als de computer het nog niet heeft.

#### Bestanden gemaakt van meerdere camera&#x27;s

Een bestand dat is samengesteld uit de banden van meer dan één camera — een multibandstack (`align-apply`, `align-stream`, `index --live --save-multiband`), een live-index samengesteld uit meerdere camera&#x27;s (`index.tif`, `index_LUT.png`), een `composite/`-afbeelding van een array-opname — noemt elke camera die erin is verwerkt:

- **Lijsten per camera, in bandvolgorde** (de compositie: achtergrondlaag, daarna voorgrond; een indexlaag met de camera’s waarvan de banden afkomstig zijn): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (het kalibratiepakket waar de banden van elke camera doorheen zijn gegaan, anders — een `align-apply` / `align-stream`-stack, waarvan de banden geen kalibratiepakket hebben ondergaan — het pakket dat de camera Chloros in de cache heeft opgeslagen, datgene waarnaar de opnamebestanden zijn vernoemd), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — parallelle lijsten, waarbij item *n* van elke lijst dezelfde camera beschrijft; `-` wanneer een camera geen waarde heeft (geen kalibratiepakket, een onbekende lens). Elke camera is **één** item: wanneer het bijbehorende pakket meerdere waarden voor een veld bevat — bijvoorbeeld de halogeenlampcertificaten van een pakket — worden deze samengevoegd met `;` (`F-1479-1;F-1480-1;F-1481-1`), nooit met een komma, zodat een lijst één item per camera weergeeft, ongeacht hoe deze wordt afgedrukt.
- **EXIF `SerialNumber`** bevat dezelfde serienummers, gescheiden door komma’s (`214000535,214000551`).
- Het bevat **geen** identiteit voor een enkele camera (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): die benoemen één camera en de bijbehorende set, en `process` zou de kalibratie van het bestand daaruit afleiden. Een index berekend op basis van de banden van meerdere camera’s (`index.tif`, `index_LUT.png`) bevat EXIF-gegevens voor `Model` `LATTICE` (de opstelling), en niet het `LATT-…`-model van één enkele camera.

Een bestand met de banden van **één** camera — een per-band TIFF-bestand (`--per-band`), een `align-apply --per-camera`-frame, een stack of live-index van één enkele camera — bevat de eigen identiteitslabels van die camera, net als elk ander opnamebestand. De stapel, de bestanden per band en per camera van `align-apply`, en elke stapel van `align-stream` en de bijbehorende bestanden per band bevatten de opnametijd van hun groep (de belichtingstijd van de referentiecamera, UTC), en het `--per-camera`-frame van een monocamera bestaat uit één vlak, net als de stapel bij `--bit-depth`.**Camera-oriëntatie (spiegelen / roteren).** De oriëntatie-instelling van een camera (de Mirror H /
Mirror V / rotatie) bepaalt wat er bij een opname wordt opgeslagen:

- **`raw` en `debayered` worden precies zo opgeslagen als de sensor ze heeft uitgelezen**, ongeacht de instelling. Een
  gedraaid of gespiegeld Bayer-mozaïek zou een ander kleurenfilterpatroon opleveren, en de kalibratiekaarten
  (dark offset, flat field) zijn gebaseerd op het eigen raster van de sensor.
- **`preview`/`display`, `radiance`, `reflectance` en de `index/`-weergave zijn georiënteerd** —
  één keer gedraaid, na alle kalibratieberekeningen, zodat ze worden geopend zoals de camera was gericht. Hun
  `clamp_masks/…_clampmask.tif`-bestanden bevinden zich op hetzelfde raster.
- **Elk opnamebestand registreert de instelling** in XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, met de klok mee, na
  de spiegelingen) en `ChlorosOrientationApplied` — `false` op onbewerkte en gedebayerde (native pixels),
  `true` op de georiënteerde producten. `ChlorosProcessing` geeft een overzicht van de bewerkingen die op de pixels van dat bestand zijn toegepast
  (`none` wanneer native). Dat omvat elk frame dat `lattice continuous` streamt (de
  eigen instelling van de camera, niet toegepast) en elk frame van `lattice multi-stream` (de identiteit: de
  bijbehorende frames zijn native, net als bij een array-opname). `lattice align-stream` / `align-apply`
  stacks registreren de instelling van de hoofdcamera, niet toegepast: hun banden worden vervormd naar het frame van de hoofdcamera
  maar niet gedraaid.
- **`process` past de geregistreerde oriëntatie toe** bij het verwerken van een raw-bestand: eerst donkere offset, vlak
  veld, demosaïc en ontmenging in sensorcoördinaten, vervolgens de uitlijningsvervorming van de array, en
  oriëntatie als laatste — zodat de preview-, radiance-, reflectance- en indexproducten precies zo uitkomen als
  die van de live-opname, en `ChlorosOrientationApplied=true` registreren. De niet-gekalibreerde fallback-
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) zijn op dezelfde manier georiënteerd, na
  het flat field. Het debayered product blijft in de oorspronkelijke staat.
- **De Chloros-beeldviewer toont een raw- of debayered-bestand dat is gedraaid volgens de opnamereeks** (alleen weergave —
  het bestand wordt niet gewijzigd), zodat het eruitziet als de bijbehorende producten; de miniatuur, de weergave op volledige grootte, de
  cursoruitlezing en de Index/LUT-sandbox gebruiken allemaal dezelfde draaiing. Een bestand zonder record, en
  een product (dat al is georiënteerd), wordt weergegeven zoals het is opgeslagen. Een sandbox-export registreert de oriëntatie die de
  pixels bevatten (`ChlorosOrientationApplied=true`; de identiteit voor een laag zonder opname).
- **Opnames zonder record worden als native verwerkt** (ongewijzigd gedrag). Vóór 1.3.0 draaide een
  opname met één camera waarbij een spiegeling of rotatie was ingesteld, de raw- en debayered-bestanden zelf om;
  Chloros kan niet vaststellen of zo’n reedsgedraaide raw-bestand niet van een native bestand en doet geen gissingen, dus
  verwerkt het dit zoals het is opgeslagen. De kalibratiekaarten bevinden zich op het eigen raster van de sensor, dus gebruik voor een dergelijke
  opname er een die is gemaakt met versie 1.3.0 wanneer de kalibratie van belang is.
- **Arrays draaien standaard geen enkel bestand in de reeks** (de uitlijningsvervorming en de `Alignment*` XMP bevinden zich in
  de eigen, niet-gedraaide frames van de camera’s) en registreren de identiteit. `array-capture --orient` draait de
  producten van elk lid, na de uitlijningsvervorming: volgens de oriëntatie van de **master** op het moment dat ze
  in het frame ervan werden vervormd (zodat de leden geco-registreerd blijven), of anders volgens die van hun eigen camera. Elk
  bestand registreert de oriëntatie van het frame waarin de eigen pixels zich bevinden — een raw-bestand, dat van de eigen camera —
  en een raw-bestand dat de uitlijningstransformatie bevat, registreert ook `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: die van de master, voor een product dat
  in het frame van de master. `process` draait elk product met de waarde die bij zijn pixels hoort:
  de uitgelijnde waarde wanneer het de raw vervormt (`--array-alignment`, de standaardinstelling — de producten van de live uitgelijnde
  opname), de eigen uitlijning van de camera wanneer dat niet gebeurt (`--no-array-alignment` — de live
  niet-uitgelijnde opname). Dezelfde optie bestaat voor `array-burst`, `project capture` / `burst` /
  `stream`, recepten en de SDK.

### Overgeslagen opnames in arrays met gemengde filters

Wanneer je RGB- en multispectrale camera’s in één array combineert, slaat `array-capture --processing radiance` (of `reflectance`) de multispectrale frames op en **slaat** de RGB-camera&#x27;s — de straling per Bayer-pixel is niet relevant voor een breedbandsensor. De CLI geeft elk opgeslagen bestand (met het exportniveau), elke geschreven `.daq` en elke overslaan expliciet weer, dus het aantal bestanden is geen verrassing:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokens voor de reden van het overslaan volgen het patroon `<level>-not-applicable-to-rgb-cam`. Een camera waarvan het kalibratiepakket niet is geladen, slaat `radiance` / `reflectance` over met `<level>-needs-calibration (connect the camera so its cal pack loads)`, en bij geen bruikbare DAQ-reflectiewaarde worden `reflectance-skipped-no-dls` overgeslagen. Reflectie kan ook worden overgeslagen met `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, en bij `dls-uncalibrated-band-<nm>` wanneer de band grotendeels buiten het radiometrisch gekalibreerde bereik van de DAQ-lichtsensor ligt (~374–974 nm) — van de leverbare SKU’s is alleen F988 beschikbaar, waarvan het ondersteunde traject de reflectantie-paneel-workflow is.

Gebruik `--processing debayered` (of `display`) om elke camera mee te nemen, ongeacht het filtertype, of de standaardinstelling `all` om in één keer alle toepasselijke niveaus per camera te verkrijgen.

### Een DAQ-meting ver van de belichting

Elke `.daq` die bij een opname wordt opgeslagen, is de DAQ-meting die bij die belichting hoort, en de `.match.json` ernaast registreert hoe ver de twee uit elkaar lagen (`gap_ms`) en of dat binnen de tolerantie valt (`in_tolerance`, 500 ms tenzij de opname-instelling `--daq-tolerance-ms`). Een waarde buiten deze tolerantie wordt nog steeds gebruikt, zodat er geen frame verloren gaat, en deze wordt gelabeld: het XMP-bestand van het reflectantiebestand bevat `DlsInTolerance=False` en `DlsMatchGapS`. `lattice capture`, `multi-capture` en `array-capture` geven deze ook weer onder de betreffende `.daq`, waarbij de DAQ&#x27;stroomfout van de DAQ wanneer men het uitlegt:

**Hoe een meting wordt gekoppeld aan een belichting.** Een opname wacht op de eerste meting van de DAQ na de belichting, zodat de metingen aan weerszijden ervan kunnen worden samengevoegd (`method` `interpolated`). Een DAQ-E verstuurt bij daglicht elke 0,4 s een meting en binnenshuis elke 1-2 s, dus de wachttijd komt neer op ongeveer anderhalf van die intervallen voor de opdracht, niet voor de belichtingstijd. Er wordt niet gewacht op een DAQ die is gestopt met verzenden. Een DAQ-E-meting is het gemiddelde licht over het interval sinds de vorige meting, dus deze wordt in het midden van dat interval geplaatst: `gap_ms` wordt gemeten vanaf de belichtingstijd tot dat midden, en `bracket_stamps_ns` geeft de eigen tijdstempels van de metingen weer. `array-capture --continuous` wacht zonder de opname te vertragen: de ruwe bestanden worden direct weggeschreven en hun `.daq` zodra de meting binnenkomt. Met `--interval` wacht elke doorloop alsof het een enkele opname betreft.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Wanneer er geen reflectantiebestand is geschreven op basis van die meting, eindigt de eerste regel in plaats daarvan met `saved anyway; its .match.json says in_tolerance: false`.

De opname eindigt nog steeds met `0`: elk bestand is geschreven en geeft aan wat het bevat. Een DAQ waarvan de stream is vastgelopen, blijft de laatste meting doorgeven; controleer deze dus met `chloros-cli daq pool-list` voordat u vertrouwt op de reflectie van die opname. De backend maakt opnieuw verbinding met een gepoolde DAQ-E die is gestopt met het leveren van gegevens (zie [Een DAQ-E die stopt met leveren](#a-daq-e-that-stops-delivering)). Een meting buiten de tolerantie drukt alleen de `DAQ:`-regel af, tenzij het een bewaarde meting betreft (zie hieronder).

**Een meting die bewaard is gebleven nadat de bijbehorende DAQ wegviel.** Wanneer geen enkele DAQ een meting voor een belichting kan leveren (de gekoppelde DAQ is niet beschikbaar, of geen enkele DAQ uit de pool heeft er een, bijvoorbeeld na `daq pool-disconnect`), hergebruikt de opname de laatste meting die Chloros had, en de `.match.json` beschrijft die meting in relatie tot deze blootstelling: `method`, `retained`, `gap_ms` van deze belichting naar de monsters waarvan de meting afkomstig was, `in_tolerance` van die tussenruimte, en `retained_from_epoch_ns`, de belichting waaraan de meting aanvankelijk werd gekoppeld. Een opname die lang nadat de DAQ was uitgevallen is gemaakt, drukt daarom de bovenstaande regels af, en daaronder daaronder nog één, die zelfs wordt afgedrukt terwijl de meting nog binnen de tolerantie valt:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

De naam `.daq` verwijst nog steeds naar de DAQ die de meting heeft uitgevoerd, met de kalibratie en de cap waaronder die meting is uitgevoerd, zodat bij een herimport bekend is welke DAQ, kalibratie en cap deze hebben geproduceerd.

### Bestanden die niet kunnen worden opgeslagen

Een camera waarvan het bestand niet kan worden opgeslagen — een volle schijf, een fout met de toegangsrechten — verhindert niet dat de bestanden van de andere camera’s worden opgeslagen. Elke opnamecommando (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; en de `capture`, `burst` en `stream` van een recept) geeft elk bestand dat niet is opgeslagen weer op stderr, en **sluit vervolgens `1` af**, ook al is de rest opgeslagen:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Elk bestand dat bij een opname hoort, wordt verwerkt, inclusief het enigeframe zonder debayering (`lattice capture --levels debayered`) inbegrepen. Een `.daq`-lezing waarvan het `.match.json`-herkomstbestand niet kon worden geschreven, wordt weergegeven als `[match]` (het `.daq`-bestand zelf is wel opgeslagen).

Op een volle schijf wordt een herhaalde opname (`array-capture --continuous` / `--interval`, `project burst` / `stream`, een recept `burst` / `stream`) **stopt bij die doorloop** in plaats van de rest van de run niets naar dezelfde schijf te schrijven. Elke andere fout wordt bij elke doorloop gemeld waarin deze zich voordoet, en de run gaat door tot het einde, waarna `1` wordt afgesloten. `array-capture --continuous` (de continue vastlegging van de backend) geeft geen bestanden weer, ook niet de mislukte: deze worden meegeteld in de voortgangs- en samenvattingsregels (`N failed`, `N file(s) failed to write.`) en sluit af met `1` — zie [Continue modus](#continuous-mode---continuous).

De recorders — `array-burst` en `array-record` — melden hun eigen fouten. `array-burst` stopt zichzelf wanneer de schijf vol raakt, of net daarvoor. Beide gaan door ondanks eventuele andere mislukte schrijfpogingen. Wanneer een van beide een fout meldt, wordt deze na de tellers weergegeven, en het commando **`1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` in plaats van `Disk full:` betekent dat de burst werd gestopt voordat de schijf. De frames die vóór de stop zijn opgenomen, blijven behouden. `array-burst --build` slaat de build over wanneer de ruimte opraakt, omdat de video’s naar dezelfde schijf zouden gaan. Maak eerst ruimte vrij en voer vervolgens `array-build-video --burst-dir …` uit. Na elke andere fout wordt de build nog steeds uitgevoerd.

Een `array-record` waarbij helemaal geen frames zijn geschreven, is ook een mislukte opname (`Recorder error: no-frames: the combined view published no frame to record …`, afsluiten van `1`) — zie [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Opnamemodi, recorders en offline herverwerking

Deze werken allemaal op een **persistent array** (voer eerst `array-connect` uit). Ze weerspiegelen het opnamepaneel van de GUI.

### `array-capture`-modi

`array-capture` is één enkele opdracht met vier sluitermodi plus een reeks exportschakelaars:

| Modus | Vlag | Gedrag |
| --- | --- | --- |
| **Enkel** *(standaard)* | (geen) | Eén gesynchroniseerde opnamegroep, daarna afsluiten. |
| **Continu** | `--continuous` | Aaneengesloten opnames totdat `Ctrl+C`, `--count N` opnames, `--duration S` of een volledige schijf. Voert de eigen continue opname van de backend uit (de modus Continu van de GUI) — zie [Modus Continu](#continuous-mode---continuous) hieronder. |
| **Interval** | `--interval S` | Eén doorloop om de `S` seconden (gemeten vanaf het begin van elke doorloop), dezelfde grenzen. |
| **Snelste** | `--fastest` | Alleen ruwe-alleen + de toegewezen DAQ-waarde + de samengestelde index; slaat de berekeningen voor straling/reflectie/weergave over, zodat het frame snel wordt weergegeven. Impliceert `--processing raw --force-daq`. Verwerk de opgeslagen `.daq` later opnieuw tot gekalibreerde producten. |

Exportopties (te combineren met elke modus; alle delen hetzelfde GUI/SDK-eindpunt):

| Vlag | Effect |
| --- | --- |
| `--array-id ID` | De array waaruit moet worden vastgelegd (weglaten wanneer er slechts één is aangesloten). |
| `-o, --output DIR` | Uitvoermap (standaard `output`, in de map van waaruit u de opdracht uitvoert). |
| `--format {tiff,tif,png,jpg,jpeg}` | Bestandsformaat (standaard `tiff`). PNG/JPEG gebruiken alleen `display` en de indexweergave; zie [Opname-exportniveaus](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Enkelvoudig exportniveau (`raw`, `debayered`, `radiance`, `reflectance`, `display`), of `all` (standaard). |
| `--levels L1,L2,…` | Expliciete subset van exporttypen (bijv. `raw,radiance,reflectance`); **heeft voorrang op `--processing`**. |
| `--aligned` / `--no-aligned` | **Standaard: uitgelijnd, net zoals de vastlegging van dezelfde array door de Chloros-app**(de instelling**Uitgelijnd** per array, ingeschakeld tenzij je deze uitschakelt). De niet-raw-export van elk lid (`debayered`, `display`, `radiance`, `reflectance`, de indexweergave) wordt vervormd naar het beeld van de hoofdcamera en bijgesneden tot de gemeenschappelijke overlap van de leden, zodat de bestanden pixel-geregistreerd zijn en dezelfde afmetingen, bijsnijding en `Alignment*`-tags hebben als die van de app — kleiner dan de sensor (of de ROI) met de offsets van de leden; bij een ROI wordt de transformatie opnieuw gebaseerd op het venster. `XMP-mapir:AlignmentJSON` verwijst naar `"applied": true` en levert `crop_roi` op. Raw blijft onvervormd op de oorspronkelijke grootte, maar bevat de transformatie in de metagegevens (`"applied": false`). `--no-aligned` slaat elk bestand op in de oorspronkelijke grootte, onvervormd. `--aligned` is de standaardinstelling. Of de vervorming nu wordt toegepast of niet, elk bestand registreert de `Alignment*`-transformatie van zijn camera zodra de reeks een uitlijningsprofiel heeft — behalve een camera die niet door het profiel wordt uitgelijnd (weggelaten of door een mislukte oplossing gedwongen tot identiteit), en de referentiecamera wanneer geen enkele andere camera is uitgelijnd: deze bevatten geen `Alignment*`-tags. Zonder profiel voor de reeks wordt de opname niet-uitgelijnd opgeslagen, en de CLI geeft de reden weer op een `Alignment:`-regel, net zoals bij elk ander geval waarin de vervormingscorrectie niet kon worden toegepast (elementen met verschillende resoluties, een array die niet via de kabel is gesynchroniseerd, een frame waarop het profiel niet kan worden toegepast). De automatische uitlijning van de array maakt het profiel zodra de belichting stabiel is; `align-calibrate` maakt er op verzoek een aan. |
| `--orient` | Maak de **producten**van elk onderdeel — `display`, `radiance`, `reflectance`, de `index/`-render en de `composite/` — aan de camera-oriëntatie, na de uitlijningsvervorming: een product dat in het frame van de master (de standaardinstelling) is vervormd door de**master&#x27;s**oriëntatie, zodat de onderdelen op elkaar blijven aansluiten; een niet-uitgelijnd exemplaar (`--no-aligned`, of een opname waarop de warp niet kon worden toegepast) volgens de oriëntatie van de eigen camera.**Standaard: uit — elk onderdeelbestand blijft in de oorspronkelijke indeling**, zoals altijd het geval is bij array-opnames. `raw` en `debayered` blijven in beide gevallen in hun oorspronkelijke formaat. Elk bestand registreert de oriëntatie van de producten die het bevat (zie [Camera-oriëntatie](#hoe-een-opnamemap-eruitziet)); de `Alignment*` XMP blijft de transformatie in het oorspronkelijke frame, toegepast vóór de oriëntatie, dus `process` (vervorming, daarna oriëntatie) het live-product reproduceert. |
| `--index` / `--no-index` | De per camera ingestelde vegetatie-index-overlay opslaan / overslaan, indien deze is geconfigureerd. Standaard: weergeven. |
| `--no-daq` | Sla de `.daq`-sidecar over wanneer geen enkel gekozen niveau de DAQ gebruikt (een uitsluitend raw-opname). Standaard wordt de meting opgeslagen, zodat de frames offline opnieuw kunnen worden verwerkt tot reflectantie/index. `--force-daq` wordt nog steeds geaccepteerd (de standaardinstelling). |
| `--daq-tolerance-ms MS` | De DAQ-matchtolerantie voor deze opnames (standaard 500 ms). |
| `--smart` | Wacht tot de AE op alle camera&#x27;s stabiel is voordat er wordt getriggerd (zie [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-pixelcompressie. `none` (standaard sinds 1.3.0; 1.2.0 schreef `deflate`) = ongecomprimeerd — de pixels van de sensor precies zoals ze worden uitgelezen, ~6,3 MB per frame op volledige resolutie, en de hoogste aanhoudende snelheid; `deflate` = verliesloze zlib L1 + horizontale voorspeller, ~4,1 MB per frame, langzamer. Beide zijn verliesloos en worden bij het importeren identiek weergegeven. |

> **Eenmalig geschreven TIFF-bestand + het model met constante snelheid.**Elk bestand wordt**één keer**geschreven: pixels, het XMP-pakket, IFD0 Make/Model/CameraSerialNumber en de EXIF-tags `ExposureTime` / `SerialNumber` in één doorloop, zonder dat er daarna iets wordt herschreven. `--continuous`X legt opeenvolgende gesynchroniseerde groepen vast — elke opname pakt de groep na de vorige, inclusief een groep die werd gepubliceerd terwijl die opname nog aan het opslaan was — zodat het op de snelheid van de array draait. DEFLATE houdt de Python-GIL vast, dus gecomprimeerde schrijfbewerkingen worden**niet**parallel uitgevoerd over de schrijfthreads per camera; voor een aanhoudende opname in volledige resolutie op sensorsnelheid zijn ongecomprimeerde schrijfbewerkingen nodig (de standaardinstelling)**en** een schijf van NVMe-klasse (~600 MB/s continu voor een array van 9 camera’s bij ~10 fps). Dezelfde instelling is beschikbaar als `compression` op `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Continue modus (`--continuous`)

`array-capture --continuous` (zonder `--interval` of `--smart`) draagt de uitvoering over aan de **eigen continue opname van de backend** — de sessie waarin de modus Continu van de GUI draait. De backend slaat elke gesynchroniseerde groep zelf op, met twee opnames tegelijk en geen verzoek van de CLI tussen groepen, zodat deze gelijke tred houdt met een snelle array waarbij een verzoek per opname achterblijft (gemeten op 73–77 opnames/s op een array met 9 camera&#x27;s bij een ROI van 512 pixels). Elke hierboven genoemde exportschakelaar is van toepassing op elke opname.

- **Bounds:** `--count N` stopt na N opnames (één opname = één gesynchroniseerde groep: de bestanden van elk lid); `--duration S` na S seconden. Zonder een van beide loopt het programma door tot `Ctrl+C`, een volle schijf, een fout of het verdwijnen van de array. `Ctrl+C` beëindigt de sessie van de backend, en geeft vervolgens het overzicht weer.
- **Voortgang** elke 2 s, met `, N failed` en `, N groups skipped` toegevoegd wanneer deze niet nul zijn (een overgeslagen groep is een groep die door de array is gepubliceerd maar waarvoor geen opname is gemaakt):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Geen lijst per bestand.** De run eindigt met één samenvattingsregel — vastgelegde bestanden, snelheid, bestanden en de uitvoermap — gevolgd door `N file(s) failed to write.` / `N group(s) skipped.` wanneer de waarde niet nul is:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Stopt** bij een volle schijf met `Continuous capture stopped: the drive is full.`, en bij een fout of het verdwijnen van de array met `Continuous capture stopped: <reason>.` (beide op stderr).
- **Uitgangssstatus:** `0` wanneer de opnames zijn opgeslagen en elk bestand is geschreven, inclusief `Ctrl+C`; `1` bij elk bestand dat niet kon worden geschreven, een volle schijf, een fout, het verdwijnen van de array of helemaal geen opname.
- **Microsoft Defender:** wanneer Defender een CPU-kern of meer gebruikt voor het scannen van de bestanden die de opname schrijft, wordt tijdens de uitvoering één keer het volgende op stderr weergegeven: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Zie de [Systeeminstellingen](#windows-exe) van het installatieprogramma.
- Eén doorlopende opname per array tegelijk; een tweede wordt geweigerd met `a continuous capture is already running for this array`. Via HTTP is de sessie `POST /api/camera/array/continuous/start` (de `/api/camera/array/capture`-body plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` en `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, met of zonder `--continuous`) en**`--continuous --smart`** behouden de lus per verzoek: één opnameverzoek per doorloop — op elk gepland tijdstip voor Interval, nadat AE tot rust is gekomen voor `--smart` — waarbij de bestanden van elke doorloop worden weergegeven zoals in de Single-modus, met een `Done — N capture pass(es).`-regel aan het einde en `--count N` dat het aantal doorlopen telt.

### `array-record` — gecombineerde index video/GIF (monitoringkwaliteit)

Neemt de **live gecombineerde-indexweergave** van de array op in een `.avi` (en optioneel een `.gif`). Als de gecombineerde weergave van de GUI geopend is, neemt het die weergave op zoals deze wordt weergegeven. Als er geen geopend is, genereert de backend de weergave zelf: de index die u doorgeeft met `--expr`, of standaard de NDVI van de array, zodra de array is uitgelijnd. De standaard-NDVI gebruikt de rode en NIR-banden die het dichtst bij 660 nm en 850 nm liggen, net als de liveweergave. Een GUI-weergave die tijdens de opname wordt geopend, wordt opgenomen zolang deze open blijft. Het programma controleert de voortgang elke 2 s en stopt bij `--duration`, `Ctrl+C`, of wanneer de recorder zichzelf beëindigt.

Een opname waarbij geen enkel beeld wordt geschreven, mislukt: de recorder meldt `no-frames: …` (de matrix is mogelijk nog bezig met uitlijnen of het instellen van de belichting) en het commando **beëindigt bij `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--array-id ID` | alleen array | Doelarray (weglaten als er slechts één is aangesloten). |
| `-o, --output DIR` | `output` | Uitvoermap. Een relatief pad bevindt zich in de map van waaruit het commando uitvoert. |
| `--fps F` | `10` | Framesnelheid bij opname. |
| `--duration S` | tot Ctrl+C | Automatisch stoppen na `S` seconden. |
| `--gif` | uit | Schrijf ook een geanimeerde GIF. |
| `--gif-only` | uit | Alleen een GIF opslaan (geen `.avi`). |
| `--expr EXPR` | de NDVI van de array | Indexuitdrukking om op te nemen wanneer er geen GUI-weergave is geopend, in de bandnamen van de array (bijv. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; zie [bandnamen](#index--vegetation-maths)). Een korte kleurnaam (`NIR`, `Red`) werkt alleen als één band van de array deze heeft. Een geopende GUI-weergave wordt in plaats daarvan vastgelegd zoals deze wordt weergegeven. |

### `array-burst` — raw-Bayer-burst met hoge fps (geschikt voor analyse)

Leest rechtstreeks uit de gesynchroniseerde groepsbuffer van de opnamelus — **geen kalibratieketen, geen exiftool, geen liveweergave nodig** — zodat het op de volledige opnamesnelheid van de camera draait. Schrijft raw-frames + een manifest per frame + één `.daq` per afzonderlijke DLS-meting onder `<output>/bursts/<base>/`. Offline opnieuw verwerken (volgende commando), of `--build` doorgeven om dit onmiddellijk bij het stoppen te doen.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--array-id ID` | alleen array | Doelarray. |
| `-o, --output DIR` | `output` | Uitvoermap (burst komt terecht in `<DIR>/bursts/<base>/`). |
| `--duration S` | tot Ctrl+C | Automatisch stoppen na `S` seconden. |
| `--max-frames N` | onbeperkt | Automatisch-stop na `N` onbewerkte frames. |
| `--orient` | uit | De producten die een build van deze burst maakt, worden gedraaid volgens de camera-oriëntatie — producten per camera volgens die van hun eigen camera, de gecombineerde index door die van de master — zoals `array-capture --orient` doet. De onbewerkte frames blijven in hun oorspronkelijke formaat; de `manifest.json` van de burst registreert de oriëntatie en de optie van elk onderdeel. |
| `--build` | uit | Na het stoppen de burst onmiddellijk opnieuw verwerken (hetzelfde als `array-build-video`). |
| `--products …` | `combined:index` | Met `--build`: welke video(&#x27;s) moeten worden samengesteld (zie hieronder). |
| `--fps F` | `10` | Bij `--build`: fps van de uitvoervideo. |
| `--save-tiffs` | uit | Met `--build`: sla ook per frame gekalibreerde TIFF-bestanden op (elk met zijn [unmix-clamp-masker](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on) wanneer een pixel is geklemd; het bouwresultaat vermeldt ze als `clamp_masks`). |
| `--gif` | uit | Met `--build`: schrijf ook geanimeerde GIF(s). |

### `array-build-video` — een opgeslagen burst offline opnieuw verwerken

Elk onbewerkt frame wordt qua tijd gekoppeld aan de dichtstbijzijnde opgeslagen `.daq`-meting en door dezelfde **radiance-/reflectance-/indexketen als de importpijplijn** geleid, waarbij één of meer video’s worden gerenderd.

`--products` is een door komma’s gescheiden lijst van `kind:level`-items, waarbij `kind` ∈ `per_cam` | `combined` en `level` ∈ `radiance` | `reflectance` | `index`. Een losstaande `level` (zonder `kind:`) wordt standaard ingesteld op `per_cam`. De standaardinstelling is `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--burst-dir DIR` | (verplicht) | Pad naar de burst-map (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-lijst, zoals hierboven. |
| `--fps F` | `10` | FPS van de uitvoervideo. |
| `--save-tiffs` | uit | Sla naast de video(&#x27;s) ook per frame gekalibreerde TIFF-bestanden op. |
| `--gif` | uit | Schrijf ook geanimeerde GIF’s. |

**Een video bevat geen clamp-masker** (één beeldstroom, 8-bit). In plaats daarvan bevat het bouwresultaat `clamp_fractions`: per videoproduct, het aandeel in pixels van elk frame dat de unmix binnen de band van het betreffende R/G/B-kanaal vasthoudt — de `Chloros:Band{R,G,B}UnmixClampFraction` waarmee het stralings-/reflectieproduct van dat frame zou worden gemarkeerd (voor de gecombineerde index, van elke deelnemende camera). De run drukt het slechtste frame per product af op een `Clamp:`-regel. `--save-tiffs` voegt de maskers per pixel voor de TIFF’s toe, in `clamp_masks/<level>/` naast de `<level>/`-map van de TIFF-bestanden.**Een voortijdig afgebroken build laat geen onvolledige video achter.** Elke video wordt opgeslagen als `<name>.partial.avi` en krijgt pas zijn definitieve naam als hij voltooid is; een oudere video met die naam blijft dus bestaan totdat zijn vervanging compleet is. Voer de build opnieuw uit om een achtergebleven `.partial.avi` te vervangen.

De run geeft elke video weer die is opgeslagen, gevolgd door elk `Skipped:`-product (een aangevraagde camera/niveau — of de gecombineerde index — die geen video heeft opgeleverd, met de reden — bijv. geen kalibratiebundel, geen DLS-meting voor reflectantie, of geen rood/NIR-bandpaar voor de gecombineerde index) en eventuele `Note:`. Een RGB-camera waarvoor radiance of reflectantie is aangevraagd, krijgt in plaats daarvan de weergavevideo, met witbalans wanneer de kalibratiebundel van de camera beschikbaar is; een camera die zonder dit wordt weergegeven, wordt genoemd in een `Note:`. Het **levert een waarde anders dan nul op** wanneer een aangevraagd product werd overgeslagen of niet volgens de aanvraag werd gegenereerd (een geconfigureerde indexuitdrukking wordt niet offline geëvalueerd); de video’s die wel werden gegenereerd, worden nog steeds geschreven en weergegeven.

> **Kies de juiste recorder.** `array-record` is van *monitoringkwaliteit* — het neemt de weergegeven live-composiet op: de gecombineerde weergave van de GUI, of, als er geen GUI openstaat, de eigen NDVI-weergave (of `--expr`) van de backend. `array-burst` → `array-build-video` is *geschikt voor analyse* — het slaat ruwe sensorgegevens op met volledige snelheid op en reconstrueert daarna gekalibreerde radiance-/reflectance-/indexvideo’s, zonder dat er een liveweergave nodig is.

### Mono (M3M) enkelbandcamera’s

De **M3M**-serie is de mono-variant van de Bayer**M3C**: één smalband-interferentiefilter per camera (`M3M-<lens>-F<wavelength>`, bijv. `M3M-L87-F685`), waardoor de sensor een**enkele grijswaardenband** levert zonder Bayer-mozaïek. Er hoeft niets gedemosaïceerd te worden, er is geen kanaaloverschrijding die ontmengd moet worden en er hoeft geen witbalans ingesteld te worden — de hele kleurenverwerkingspijplijn voor RGB-weergave is simpelweg niet van toepassing.

Wat dat betekent op de CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detecteren een monochrome camera en**slaan deze over met een bericht van één regel** in plaats van zinloze instellingen door te geven. Ze werken nog steeds normaal bij een RGB-/Bayer M3C-camera in dezelfde sessie.
- **Een Bayer-pixelformaat duidt de diepte aan.** Een M3M biedt alleen mono-formaten, dus een Bayer-formaat van `--pixel-format` of `--preset` stelt het mono-formaat met dezelfde diepte in: `BayerRG8` en `--preset high_speed` leveren `Mono8` op; `BayerRG12`, `high_quality` en `triggered` leveren `Mono12`; een samengepakte naam blijft samengepakt (`BayerRG12p` levert `Mono12p` op). De bestanden en `lattice format` hebben de naam Mono-formaat. Een array die M3C- en M3M-camera’s combineert, neemt één formaat op : Bayer op elke M3C, de Mono-diepte op elke M3M. Een naam is de bitdiepte ervan: het omgekeerde geldt ook, dus een M3C wordt nooit in een Mono-formaat geplaatst (`--pixel-format Mono12` is `BayerRG12` op een M3C, ongeacht welke camera de master van de reeks is), en `lattice format --set` wordt op dezelfde manier toegewezen en drukt het ingestelde formaat af.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** werken nog steeds — straling en reflectie zijn *per-band* radiometrische kaarten en zijn perfect gedefinieerd voor één band. Mono-frames bevatten een **identiteits**-sensorresponsmatrix (geen 3×3-ontmenging), dus het vlak doorloopt de kalibratieberekeningen ongewijzigd.
- **Een enkele monochrome camera kan geen vegetatie-index produceren.**NDVI, NDRE en dergelijke hebben ten minste twee banden nodig (bijv. Red + NIR). Om een index te verkrijgen met monochrome hardware, richt je**meerdere** M3M-camera’s op verschillende golflengten, voeg je ze samen tot één multiband-stack en bereken je *daaruit* de index:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` opent de camera’s zelf (een [tijdelijke sessie](#multi-camera-transient-sessions)), dus sluit eerst niets aan: `--serials`, anders worden de ingeschakelde camera’s van het profiel (zie [Index / Vegetatiewiskunde](#index--vegetation-maths)). Zolang Chloros een camera vasthoudt (de GUI, `cam-connect`, `array-connect`) wordt dit geweigerd met HTTP-statuscode 409 `CAMERA_HELD` en wordt het programma afgesloten `1`; geef ze vrij met `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel`-symbolen moeten **precies** overeenkomen met de kanaalnamen van de preset (hoofdlettergevoelig; NDVI’s zijn in kleine letters `red`,`nir` — zie `--list-presets`), en de bandnaam verwijst naar een band in de uitgelijnde stack (in de offline-modus worden ook bandindexen op basis van 0 geaccepteerd, bijv. `--channel red=0 --channel nir=1`).

Het onderscheidende kenmerk in de gehele stack is het token `M3M` in de modelstring (dit komt nooit voor in een `M3C`-string), dat in de GUI/SDK wordt weergegeven als `is_mono`.

---

## Configuratie en afstemming van de host-NIC (LATTICE-arrays)

LATTICE-camera’s streamen GVSP via de Ethernet-adapter van de host, dus bij arrays met meerdere camera’s zijn de **driver**en**grootte van de ontvangstring** net zo belangrijk als de verbindingssnelheid. Verkeerde instellingen worden weergegeven als een `FRAMES WILL DROP` / `Reduce ROI to enable`-melding in het paneel ‘Array-instellingen’ (en in `lattice network-analysis` / de `analyze_array_network()` van de SDK), zelfs als de camera’s zelf in orde zijn.

### USB 10GbE-adapters — Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

De onderstaande instellingen zijn eigenschappen van het Windows-stuurprogramma. Onder Linux krijgt een USB-adapter ook een limiet (400 MB/s op x86_64, 200 MB/s op NVIDIA Jetson) (zie [Array fps &amp; burst-model](#array-fps--burst-model)), en de socket-ontvangstbuffers van de kernel zijn van belang voor elke adapter — zie [Linux-hosts](#linux-hosts--socket-receive-buffers).

| Item | Vereiste waarde | Waarom dit belangrijk is |
| --- | --- | --- |
| **Stuurprogrammaversie**|**≥ v10.67 (jan. 2026)**, INF `rtump64x64sta.inf` | Het verouderde**2016**-stuurprogramma (v10.65, `rtump64x64.inf`) verwerkt het uitschakelen en bugchecks verkeerd, met**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**bij het afsluiten/opnieuw opstarten/in de slaapstand gaan. Het proces loopt vast (~5 min. time-out), de gebruiker schakelt het systeem geforceerd uit, en herhaaldelijk onjuist afsluiten**beschadigt de WMI-repository**(PowerShell/tools beginnen fouten te vertonen met `Invalid class`) en**blokkeert de USB-stack** bij de volgende opstart (NIC kan niet worden ingeschakeld; USB-drives worden niet meer herkend). Voer een update uit via realtek.com (of de leverancier van de dongle) voordat je vertrouwt op schone herstarts. |
| **Ontvangstbuffers**— trefwoord `ReceiveBufferLen` |**256**(maximum van het stuurprogramma) | De RX-ring van de netwerkkaart. De standaardinstelling van het stuurprogramma van**32**laat slechts ~0,26 MB bruikbare ringruimte over — veel te klein voor een burst met meerdere camera’s — waardoor het array-paneel `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` meldt en verbindingen blokkeert. Bij**256**is de ring groot (**~13,5 MB gemeten op de 10GbE-host in het lab**), wat de RX-pijplijn echte speling geeft voor GVSP-bursts met meerdere camera’s. (Of een bepaalde configuratie daadwerkelijk *verbinding maakt*, wordt bepaald door twee controles — de **drain-aware**toelatingscontrole en de**aggregate over-subscription**-controle — niet een ruwe vergelijking tussen burst en ring; zie [Array fps &amp; burst-model](#array-fps--burst-model).) |
| **Ontvangst-URB’s**— trefwoord `PendingReceives` |**64** (max.) | USB-verzoekblokken in behandeling; verhoog samen met de ontvangstbuffers voor het opvangen van bursts. |
| **Jumbo Frame** — trefwoord `*JumboPacket` | **9014** | Nodig voor GVSP-pakketten van 9000 bytes (6× minder pakketten/frame dan bij 1500). |

> ⚠️ **Een update van het NIC-stuurprogramma ZET deze geavanceerde eigenschappen TERUG naar de standaardinstellingen.**Na het bijwerken of vervangen van het adapterstuurprogramma moet u**opnieuw de volgende instellingen toepassen** `ReceiveBufferLen=256` en `PendingReceives=64` opnieuw, anders zal het array-paneel weer blokkeren, ook al is er „niets veranderd aan de hardware“. Dit is de belangrijkste oorzaak waarom een eerder goed werkende installatie plotseling geen verbinding meer wil maken.

Pas toe vanuit een **PowerShell met beheerdersrechten** (vervang door de naam van je adapter, bijv. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` is van toepassing op USB 10GbE-adapters.** Het detecteert nu het type adapter en stelt het juiste ‘receive-ring’-trefwoord in: `*ReceiveBuffers`→2048 voor PCIe-NIC’s (Intel I219, enz.), of `ReceiveBufferLen`→256 + `PendingReceives`→64 voor de Realtek **USB** 10GbE-controller (die `*ReceiveBuffers` niet beschikbaar stelt). De waarden worden begrensd tot de door elk stuurprogramma gerapporteerde maximumwaarde (`NumericParameterMaxValue`), zodat er nooit een waarde buiten het bereik wordt geschreven. Voer het uit vanuit een **terminal met beheerdersrechten**; net als bij elke op het register gebaseerde aanpassing wordt de wijziging pas van kracht na het opnieuw opstarten van de adapter of het systeem. De handmatige `Set-NetAdapterAdvancedProperty`-opdrachten hierboven blijven een prima alternatief — ze worden direct toegepast (de adapter opnieuw koppelen) zonder dat een herstart nodig is.

### Linux
-hosts — ontvangstbuffers voor sockets

Op Linux
, inclusief NVIDIA Jetson, zijn de ontvangstbuffers voor sockets van de kernel net zo belangrijk als de ontvangstring van de netwerkkaart. Bij de standaardinstelling van de kernel (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 bytes) zorgt een reeks GVSP-pakketten van een camera ervoor dat de stream-socket overloopt voordat Chloros
deze kan lezen. De pakketten worden in de kernel weggegooid, terwijl de NIC zelf geen verlies meldt, en een array verliest framegroepen en verlaagt de framesnelheid. Gemeten op een array van 4 camera’s achter een USB 10 GbE-adapter: door beide waarden te verhogen naar 32 MB steeg de framesnelheid van de array na het verbinden van 4,96 naar 7,52 fps, en de continue onbewerkte opname van 4,7 naar 7,6 opnames/s, zonder dat er pakketten in de kernel verloren gingen.

- **Het .deb-pakket stelt deze waarden in.** Bij het installeren van het pakket wordt `/etc/sysctl.d/60-chloros-gige.conf` (beide op `33554432`) geschreven en onmiddellijk toegepast. Een waarde die al hoger is ingesteld, wordt ongewijzigd gelaten.
- **`lattice network` controleert deze waarden.** Wanneer een van beide lager is, wordt `socket receive buffers too small for GigE Vision (…)` vermeld onder *Geconstateerde problemen*, en bevat het logboek van de backend dezelfde waarschuwing bij de eerste verbinding van een camera of array.
- **Om ze handmatig te verhogen** (tot de volgende herstart; het bestand van het pakket past ze bij elke opstart toe):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` verhoogt en handhaaft deze waarden ook, maar alleen wanneer de backend als root draait. Een backend die als uw gebruiker draait (een die is gestart met `chloros-backend`, of [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) kan de kernelinstellingen niet wijzigen en geeft in plaats daarvan de regel `sudo sysctl` weer.

**De automatische adapterafstemming heeft alleen betrekking op aangesloten adapters.** Bij de eerste cameraverbinding stemt een backend die als root draait de bekabelde adapters af waarvan de ontvangstring klein is. Het stemt nu alleen adapters af die een verbinding hebben, dus een losgekoppelde ingebouwde poort behoudt zijn MTU en de ontvangstbuffers blijven ongewijzigd. `lattice network` vermeldt de MTU van een losgekoppelde adapter niet langerde MTU van een niet-aangesloten adapter niet langer als een probleem. `lattice network`, `--fix`, `lattice probe` en de adapterafstemming houden alleen rekening met bekabelde Ethernet-poorten, niet de CAN-bus van een Jetson (`can0`) of de verbindingen in USB-apparaatmodus (`usb0`, `usb1`) die het aan een host-pc biedt; een USB-ethernetadapter voor camera’s telt nog steeds mee.

### Netwerkbasisprincipes (alle LATTICE-verbindingen)

- **Adressering:** link-local `169.254.0.0/16` (GigE Vision LLA). De host neemt een statisch adres `169.254.x.x/16`; camera’s en DAQ-E wijzen zichzelf een adres toe binnen hetzelfde bereik. Geen DHCP/gateway vereist.
- **Pakketgrootte:**bij voorkeur jumbo (9000), maar laat de auto-probe dit bepalen — deze negeert de ICMP-limiet van 1500 bytes van de camera via een GVSP-probe, zodat de pakketgrootte op jumbo uitkomt waar de kabel dit daadwerkelijk ondersteunt.**Elke camera wordt één keer per backend-sessie gemeten, en nogmaals wanneer de verbinding verandert (een andere adapter, een andere verbindingssnelheid of een verbinding die is verbroken, zoals bij het wisselen van een switch); bij het opnieuw verbinden wordt het resultaat hergebruikt. Na een wijziging achter een switch die in de lucht is gebleven, moet je de backend opnieuw opstarten om opnieuw te meten.**Een eerste verbinding kan bij deze controle af en toe enkele minuten duren, en het logboek vermeldt dit wanneer dat het geval is. Gebruik alleen de pin met `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` als je het beter weet dan de probe, en geef de voorkeur aan per commando boven permanent: een pin slaat de probe over, dus als het pad daadwerkelijk geen 9000 kan verwerken, loopt**elke** opname vast met `SC_ERR_TIMEOUT -1011` (zie [Omgevingsvariabelen](#environment-variables)).
- **De RX-ring schaalt mee met `ReceiveBufferLen`:**bij de standaardwaarde `32` is de bruikbare ring ~0,26 MB (te klein voor elke burst met meerdere camera’s); bij de maximale waarde `256` is deze groot (~13,5 MB gemeten op de 10GbE-host in het lab), wat echte speling biedt. Of een configuratie wordt toegelaten, wordt vervolgens bepaald door de ‘drain-aware’ toelatingscontrole**en** de hieronder beschreven controle op totale overinschrijving — niet door een ruwe vergelijking tussen burst en ring.

### Array-fps &amp; burst-model

Hoe het paneel ‘Array-instellingen’ (en `lattice analyze-array` / de `analyze_array_network` van de SDK
) te lezen:

- **De burst wordt per camera opgeteld in het werkelijke pixelformaat van elke camera.**Mono**M3M**-camera’s streamen**Mono12 (2 B/px)**, of met `--pixel-format` het Mono-formaat van de bijbehorende diepte (`BayerRG8` is `Mono8`, 1 B/px);**M3C**Bayer-camera’s streamen 8- of 12-bits (M3C-firmware zendt stilzwijgend BayerRG12 uit, zelfs wanneer BayerRG8 wordt aangevraagd). Een frame met volledige resolutie van 4 camera’s is dus**~12,6 MB als alle 8-bit zijn, maar ~25 MB met drie 12-bit mono-camera’s**. De projectie bepaalt het formaat van elke camera op basis van het model (identiteitscache), zodat de burst overeenkomt met wat er daadwerkelijk over de kabel wordt verzonden — en niet uitgaat van een standaard aanname van BayerRG8.
- **Een USB-ethernetadapter heeft een maximum van 200 MB/s, ongeacht wat er op het typeplaatje staat.** De efficiëntietabel die een verbindingssnelheid omzet in een continu cijfer is afgeleid van PCIe; een USB-NIC geeft zijn *Ethernet*-verbindingssnelheid aan, maar wordt beperkt door de USB-bus en het bijbehorende stuurprogramma. Een USB 10GbE-dongle haalde vroeger ~1063 MB/s &quot;duurzame&quot; snelheid — een getal dat nooit daadwerkelijk werd getest — en de daaruit voortvloeiende beperking beschadigde 6–18 % van de frames, terwijl er nog steeds een gezonde doel-fps werd gerapporteerd. Via USB aangesloten NIC’s zijn nu absoluut begrensd: **200 MB/s** op Windows
en op NVIDIA Jetson, **400 MB/s** op Linux
x86_64 (op Linux
krijgt een adapter op een USB 2.0-poort **30 MB/s** toegewezen, en volgens de specificaties moet deze op een USB 3-poort worden aangesloten) (de limiet wordt bepaald door de USB-bus en het stuurprogramma van de host, dus deze schaalbaarheid hangt niet samen met de specificaties op het typeplaatje; een USB 1 GbE-adapter haalt ~80 MB/s en ondervindt hier geen hinder van). Op Linux
wordt de adapter geïdentificeerd via sysfs (met terugval op `ethtool -i`&#x27;s `bus-info`). Elk cijfer is gemeten: op Windows
vertoonde dezelfde opstelling met 4 camera’s 6–10 % beschadigde frames bij snelheden boven 200 MB/s; op Linux
x86_64 werkte dezelfde adapter foutloos bij elke maximale snelheid tot 480 MB/s (273 MB/s geleverd, de eigen limiet van de camera’s). Op de NVIDIA Jetson werkte dezelfde adapter alleen foutloos bij 200 MB/s en verloor hij daarboven met tussenpozen frames. `wire_ceiling_source` vermeldt dit in woorden in het capaciteitsrapport, en `nic_is_usb` markeert het. Overschrijf dit in beide gevallen met `--wire-ceiling-mbps`.
- **Toelating is ‘drain-aware’, niet ‘whole-burst-vs-ring’.** Een gelijktijdige burst hoeft alleen te passen binnen de *transiënte backlog* = `max(0, Σ per-cam arrival − host drain) × emit_window`, niet de gehele burst. Op een fabric met een snelle host en trage camera’s (een **PCIe**10G-host + 4× 1 GbE-cams: aankomst ≈ 320 MB/s, afvoer ≈ 1063 MB/s) leegt de host sneller dan de cams vullen, backlog ≈ 0, dus**laat**full-res sim-emit het toe, ook al overschrijdt de burst van 25 MB de ring van 13,5 MB. Plaats dezelfde vier camera’s achter een**USB** 10GbE-adapter op Windows
en de afvoer is 200 MB/s, niet 1063 — de binnenkomende gegevens lopen voorop, en het verlies komt tot uiting in beschadigde frames in plaats van een lagere framesnelheid. Op een 1 GbE-host zorgt de DLThr-ondergrens van 31,25 MB/s van de camera’s ervoor dat de instroom de afvoer overstijgt → het systeem **blokkeert** dit correct (voor *deze* klasse van blokkades: verlaag de ROI of gebruik binning ≥ 2). Toelating is een van de **twee** connect-poorten — de andere is de hieronder beschreven controle op totale overinschrijving.
- **De verwachte fps is de cyclus van de traagste camera.**Bij gelijktijdige verzending verstuurt elke camera zijn frame op hetzelfde moment, dus de array doorloopt een cyclus met de langste van de sensoruitslezing en de verzendtijd van het traagste lid (frame-bytes ÷ de verzendsnelheid: de**toegangsverbinding** van de camera, 1 GbE ≈ 80 MB/s, of het aandeel in de maximale bandbreedte van de host wanneer die kleiner is). Elke belichting overlapt de uitlezing van het vorige frame, dus de belichting maakt geen deel uit van de cyclus: automatische belichting vult de periode die de snelheid bepaalt, verhoogt vervolgens de versterking, en een donkere scène verlaagt de snelheid niet langer (in 1.2.0 was dat wel het geval). Alleen een handmatige belichting die langer duurt dan de periode vertraagt de trigger (zie [Belichting op een array-lid](#exposure-on-an-array-member)). Het toevoegen van camera’s vertraagt de array niet totdat hun draadbreedte afneemt. Bij seriële emissie (`slip-emit-and-capture`) worden de emissietijden van de camera’s daarentegen bij elkaar opgeteld, en wordt de belichtingstijd van elk element meegenomen in de spreiding. Gemeten bij volledige sensor, 12-bit, op een array van 9 camera’s met een PCIe 10 GbE-host: 10,7 fps, of 14,0 fps met [gecomprimeerd transport](#packed-12-bit-transport---packed).
- **Na een ononderbroken opname geeft de projectie weer wat er is vastgelegd.** Een opname van 30 s of langer met een automatische snelheid registreert de snelheid waarop het systeem bij die arraygrootte op deze computer is uitgekomen; de korte pogingen van de snelheidsregelaar om een hogere snelheid te halen, worden buiten beschouwing gelaten. Vanaf dat moment vermelden de regel van `array-connect` en het paneel ‘Array-instellingen’ die snelheid wanneer deze lager is dan die van het model, met vermelding van de oorzaak: `clock-bound` (de array draaide op de snelheid waarop deze zich had gestabiliseerd, terwijl de grab-lus wachtte op zijn triggerklok), `loop-bound` (de grab-lus had geen tijd meer), `capture-bound` (de opname liep achter en sloeg groepen over) of `measured` (er wordt niet aangegeven welke). Het cijfer van het model staat in `wire_fps_max`. De array stijgt nog steeds tot de capaciteit van de verbinding.
- **Een array werkt op elk netwerk — de framesnelheid bepaalt de limiet.**Elke camera krijgt een gelijk aandeel van de**botsingsveilige bandbreedtelimiet**(`sustained × sim_emit_factor`), afgestemd op GevSCPD, en ze worden allemaal nog steeds op dezelfde trigger belicht. Op een trage verbinding is het aandeel klein en daalt de framesnelheid mee: 8 12-bits camera’s met volledige resolutie op 1 GbE zonder jumboframes krijgen elk 52 / 8 = 6,5 MB/s, ongeveer**1 fps**(ongeveer 1,3 fps gecomprimeerd), terwijl een 10G-host ze op 11-14 laat draaien. Als de camera’s niet zo langzaam kunnen worden gesynchroniseerd — lees het GevSCPD-bereik van elke camera om dit te achterhalen — schakelt de array over naar**seriële**verzending (`slip-emit-and-capture`): de camera’s gebruiken de verbinding om de beurt op volledige linksnelheid, waarbij ze de ene na de andere worden geactiveerd, zodat elk frame nog steeds aankomt maar de leden**niet gelijktijdig worden blootgesteld**. `analyze-array` en de connect-uitvoer geven de modus aan; zet `--force-tier sim-capture-sim-emit` op ‘refuse’ om de terugval te weigeren. De framegrootte speelt bij deze verdeling geen rol, dus**binning en een kleinere ROI verhogen de framesnelheid, maar veranderen niets aan de vraag of een array past**. Alleen een verbinding die de array zelfs niet serieel kan verwerken, is**overbelast**— bijvoorbeeld veel meer camera’s dan één verbinding aankan, of veel andere camera’s die erop streamen — en daar is het symptoom GVSP-pakketverlies, niet een lagere snelheid: `analyze-array` zet de haalbare fps-waarden op nul en geeft `** OVER-SUBSCRIBED**` weer; de verbinding gaat door met een luide waarschuwing, en een HTTP
-verbinding met `pin_resolution: true` (strikte modus) **wordt geweigerd**. Oplossingen zijn: minder camera’s, end-to-end jumboframes of een snellere netwerkkaart. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` verlaagt de weigering naar een luide waarschuwing voor testdoeleinden — zie [Omgevingsvariabelen](#environment-variables).

### Status van de array — welk subsysteem verliest frames

De `GET /api/camera/array/<array_id>/capability` van een aangesloten array bevat een live
`health`-blok, dat opnieuw wordt geëvalueerd in een voortschrijdend **10-seconden**-venster. Het splitst het frameverlies
op in de twee oorzaken die tegengestelde oplossingen vereisen, in plaats van één „onvolledig”
percentage te rapporteren dat geen van beide noemt:

| Veld | Wat het betekent | Welk subsysteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriële poort) | Het frame **kwam aan en was structureel defect**— GVSP-pakketverlies. |**Netwerk**: bandbreedte, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Het frame **is helemaal niet aangekomen**— de camera heeft niet geactiveerd, of er is niets vanuit de camera verzonden. |**Trigger / synchronisatie**: M8-kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Het slechtste percentage per camera voor elk van beide oorzaken. | — |
| `per_cam_rate_pct` | Gecombineerd percentage onvolledige gegevens per camera (beide oorzaken samen). | — |
| `stable_for_seconds` | Hoe lang elke camera onder de 0,01 % is gebleven. | — |

Boven de 5 % registreert de backend een `[array-health <id>] WARN`-regel waarin de splitsing wordt benoemd — bij de
eerste overschrijding, bij een verandering van ernstniveau, eenmaal per minuut zolang de toestand aanhoudt, en eenmaal wanneer
de toestand weer normaal is. De beschadigde helft schrijft `[gvsp-corrupt <SN>]` naar het backend-logbestand bij de
eerste treffer per camera en reden, en vervolgens een samenvatting om de 60 s; het logpaneel van de app toont in plaats daarvan de
`[array-health <id>]`-regel weer. Elke evaluatie komt nog steeds in het backend-logbestand terecht;
de tellers lopen bij elke buffer verder, ongeacht wat er wordt afgedrukt.

Hetzelfde record rapporteert het getal waaraan de gehele toewijzing is gekoppeld:

| Veld | Wat het betekent |
| --- | --- |
| `wire_ceiling_mbps` | Het geldende, aanhoudende bandbreedtebudget van de host, MB/s. |
| `wire_ceiling_source` | Waar dat getal vandaan komt, in woorden — bijv. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (op Linux
x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) of `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` wanneer `--wire-ceiling-mbps` (of het veld **Wire Budget** in de GUI) dit instelt. |
| `nic_is_usb` | `true` voor een USB-ethernetadapter — zie de USB-limiet hierboven. |

**Interpretatie:** een `gvsp_corrupt_rate_pct`-waarde die niet nul is, terwijl `never_arrived_rate_pct` op 0 staat
betekent dat de trigger en de kabelsynchronisatie perfect zijn en dat 100 % van het verlies zich op het netwerkpad bevindt
— verlaag `--wire-ceiling-mbps` en maak opnieuw verbinding. Het omgekeerde patroon wijst in plaats daarvan op de
synchronisatiekabel of de triggerlijn.

> **`--target-fps` is niet de bepalende factor voor beschadigde frames.** De GevSCPD-pacing wordt
> eenmalig vastgelegd bij het verbinden, dus het verlagen van de triggerfrequentie verandert de duty cycle en niet de
> burst-snelheid bij gelijktijdige uitzending. Een gemeten verlaging van de vraag met een factor 5 leverde geen verbetering op;
> het verlagen van de draadlimiet van 240 naar 200 MB/s bracht dezelfde opstelling van 10,4 %
> corrupt naar 0,00 %.

> **Een actieve array kan zichzelf niet opnieuw plannen op LATTICE (M3C / M3M) firmware.** Het
> kan dit niet zelf verhelpen; verbreek de verbinding en maak opnieuw verbinding, zodat de connect-time-picker
> opnieuw kan plannen met de nieuwe limiet.

### Symptoom → oplossing

| Symptoom (Array-instellingen / verbinden / `analyze_array_network`) | Oorzaak | Oplossing |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` gereset naar 32 (meestal na een stuurprogramma-update) | Stel `ReceiveBufferLen`→256 in, `PendingReceives`→64; open het paneel opnieuw (start de backend opnieuw op als deze de oude ringgrootte in de cache heeft opgeslagen) |
| Opstarten/afsluiten loopt vast; later `Invalid class` WMI-fouten, NIC kan niet worden ingeschakeld, USB-stations ontbreken | Oude Realtek USB 10GbE-driver uit 2016 → BSOD `0x9F` → geforceerd uitschakelen | Werk de adapterdriver bij naar ≥ v10.67 (2026), pas vervolgens de bovenstaande instellingen voor de ontvangstring opnieuw toe |
| De array maakt verbinding op volledige sensor, maar met een lage framesnelheid | „Auto“ behoudt de volledige sensor en laat de snelheid dalen tot wat de kabel aankan | Kies een van de snellere opties uit de lijst met verbindingsuitgangen (`--binning 2` / `4`, of een lagere `--width`/`--height`), schakel jumboframes in of upgrade de verbinding |
| Array rapporteert een gezonde beoogde fps maar levert slechts een fractie daarvan; `health.gvsp_corrupt_rate_pct` niet-nul, `never_arrived_rate_pct` 0 | Het afgeleide bandbreedtebudget van de host is hoger dan wat daadwerkelijk wordt ondersteund (typisch bij een USB-ethernetadapter, een smalle PCIe-lane of een gedeelde fabric) | Maak opnieuw verbinding met een lagere `--wire-ceiling-mbps` en controleer het gezondheidsblok opnieuw. **Niet** `--target-fps` — GevSCPD-pacing is vastgelegd bij het verbinden |
| Linux
: een array verliest framegroepen en verlaagt de framesnelheid, terwijl de NIC geen verlies meldt; `lattice network` vermeldt `socket receive buffers too small` | De ontvangstbuffers van de kernel staan op hun standaardwaarde, waardoor camerapakketten in de kernel worden verloren | Verhoog `net.core.rmem_max` / `rmem_default` naar 32 MB — zie [Linux
-hosts](#linux-hosts--socket-receive-buffers). Het .deb-pakket doet dit tijdens de installatie |
| Camera’s ontbreken in gepubliceerde groepen; `health.never_arrived_rate_pct` is niet nul, `gvsp_corrupt_rate_pct` 0 | Trigger-/synchronisatiepad — de camera’s gaan niet af, geen netwerkprobleem | Controleer de M8-synchronisatiekabel en `--line`; controleer of elk lid is ingeschakeld (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` overschreden in `analyze-array`, of verbindingsweigering in strikte modus (`pin_resolution`; `array over-subscribes the wire`) | De kabel kan de array niet aan bij welke pacing dan ook die de camera&#x27;s bereiken, gelijktijdig of om beurten — veel meer camera&#x27;s dan de kabel kan delen, of andere camera&#x27;s die erop streamen | Minder camera&#x27;s, jumboframes van begin tot eind, of een snellere NIC. **ROI/binning helpt NIET** (het delen is onafhankelijk van de framegrootte). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` overschrijft op de testbank (accepteert het pakketverlies) |
| De array maakt verbinding via `slip-emit-and-capture` (serieel) op een traag netwerk | De camera’s kunnen niet langzaam genoeg worden aangestuurd om de kabel gelijktijdig te delen, dus werken ze om de beurt; frames worden niet op hetzelfde moment vastgelegd | Jumboframes of een snellere NIC herstellen de gelijktijdige opname; `--force-tier sim-capture-sim-emit` staat hierop (en verliest in plaats daarvan pakketten) |

---

## `chloros-cli daq`

Commando’s voor spectrale sensoren. De `pool-*`-opdrachten zijn ‘thin’ HTTP
-clients die de sensor aansturen via de persistente pool van de backend, dus hebben ze een actieve backend nodig (zie [Start de backend](#start-the-backend)). De backend beheert het transport, zodat de CLI
- en SDK
-scripts één actieve handle delen in plaats van te strijden om de seriële poort. `pool-discover` is de enige manier om de BLE-MAC van een DAQ-M te achterhalen.

> **`chloros-cli daq --help`** (of `-h`, of `daq help`) geeft een overzicht van de `pool-*`-subcommando’s (op Linux
samen met de directe hardwarecommando’s die bij de build zijn inbegrepen), en `chloros-cli daq <subcommand> --help` (of `daq help <subcommand>`) geeft de opties voor elk daarvan weer. Deze omvatten het maken van een verbinding, stream, het opnemen van gekalibreerde `.daq`-bestanden, het wisselen van cap-profielen en het koppelen van een DAQ aan een camera, en voor een DAQ-E de netwerkinstellingen, PTP, een herstart en firmware-updates. De DAQ kan ook vanuit Python
worden aangestuurd met `chloros_sdk.connect_daq_sensor()`, dat hetzelfde gepoolde pad gebruikt.
>
> **Gebruik de `pool-*`-opdrachten op elk platform.** Ze gedragen zich hetzelfde op Windows
, Linux
en Jetson. Op Windows
zijn zijn `daq discover` en `daq list` aliassen van `pool-discover`, en elk ander `daq`-commando dat niet `pool-*`, wordt afgesloten met `1` en `DAQ direct-hardware commands require the full daq package`. Op Linux
en Jetson zijn `daq discover`, `daq list` en de andere niet-`pool-*` `daq`-opdrachten de directe-hardwarecommando’s van de DAQ-tool: ze openen de sensor in het eigen proces van de CLI
, buiten de pool van de backend (dus niet op een sensor die door de backend, de GUI of een script wordt beheerd), en `daq discover` daar worden `--only` en `--timeout` geaccepteerd, maar niet `--json`. Deze referentie documenteert uitsluitend de `pool-*`-commando’s.

### Werkstroom voor de eerste verbinding met een DAQ-sensor

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referentie voor `pool-*`

| Subcommando | Doel |
| --- | --- |
| `daq pool-connect` (smart-detect) | Een sensor openen in de backend-pool. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U op een specifieke seriële poort. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC automatisch gescand. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M via BLE op een bekend MAC-adres (impliceert `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet met een bekende hostnaam of IP-adres (impliceert `--eth`). |
| `daq pool-connect --eth` | DAQ-E via Ethernet, host automatisch gedetecteerd (mDNS + ARP-fallback; werkt met een lege ARP-cache op Windows
en Linux
). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Integratievenster / AE-status afstemmen. |
| `daq pool-connect --no-stream` | Verbinding maken, maar nog niet beginnen met streamen (hervatten met `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-correctieprofiel. Standaard in de backend is `sunshine_cosine`; `none` is een kale sensor. Een [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) negeert deze (vaste diffusor). **De sensoren worden zonder kap geleverd, dus geef de door jou gemonteerde kap een naam:** de standaardwaarde wordt geregistreerd als verondersteld (`cap_id_source=auto_default`) en de verwerking geeft hierover een waarschuwing, aangezien een onbeklede DAQ-U bij de zonneschijncorrectie een waarde aangeeft die ~30x te hoog is. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Scan bij elke transport een op zoek naar sensoren waarmee je verbinding zou kunnen maken, zonder daadwerkelijk verbinding te maken. **Zo vind je het BLE-MAC-adres van een DAQ-M.** `--timeout` geldt per transport (standaard 5 s, 1-20). Sensoren die al in de pool openstaan, worden niet weergegeven — een aangesloten DAQ-M stopt met adverteren — dus gebruik `pool-list` daarvoor. `usb` geeft alleen USB-seriële apparaten weer: de eigen seriële poorten van een computer (`COM1`, `/dev/ttyS*` in Linux
) kunnen geen sensor zijn en worden buiten beschouwing gelaten. |
| `daq pool-list` | Toon elke sensor in de backend-pool. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Koppel een camera of een array aan een DAQ, zoals de lichtsensorselectie in de GUI dat doet: de opnames gebruiken, vergelijken en opslaan de meetwaarden van die DAQ. `SENSOR` is een sensor-ID uit `pool-list` of het adres waarop de DAQ bereikbaar is. `--clear` (geen `SENSOR`) verbreekt de koppeling, en de opnames vallen dan terug op de meest recente DAQ. De koppeling blijft bestaan totdat de backend opnieuw wordt opgestart, of totdat die camera, array of DAQ wordt losgekoppeld: een loskoppeling (hier vanuit de SDK
of in de app) verbreekt elke koppeling met de hardware die wordt losgekoppeld, ongeacht wie deze heeft ingesteld, en de koppelingen van de leden van een array met de array zelf. De camera moet zijn aangesloten, in het geopende project staan, of eerder op deze computer zijn geopend, en de array moet zijn aangesloten of in het geopende project staan; elk ander serienummer of ID wordt op basis van de naam geweigerd en er wordt niets gekoppeld. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Vrijgeven — ongeacht wie de sensor heeft geopend; hiermee wordt dus ook een sensor vrijgegeven die door een gecrasht script in de pool is achtergelaten (eventuele opnames daarop worden eerst voltooid). Een sensor die zich niet in de pool bevindt, heeft niets om vrij te geven: er wordt `ID: already released (not in the pool)` weergegeven en het programma wordt afgesloten met `0`; een vrijgave die door de backend is geweigerd of niet is bevestigd, wordt afgesloten met `1`. SDK
: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De meest recente N spectrumframes (standaard 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Streaming hervatten / pauzeren. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Een .daq-opname starten / stoppen. `-o` is standaard ingesteld op `~/Documents/DAQ Live View` op de computer van de backend (een relatief pad onder de map van waaruit je het commando uitvoert); `--duration` (`-d`, standaard `0` = tot `--stop`). `--confirm-after S` geeft een waarschuwing wanneer er S seconden na de start nog geen meting is binnengekomen (standaard 20; 0 schakelt dit uit); met `--duration` `--heartbeat S` geeft elke S seconden het aantal metingen weer en waarschuwt wanneer het aantal niet meer toeneemt (standaard 30; 0 schakelt dit uit). Een opname op een DAQ-E die stilvalt, wordt in hetzelfde bestand voortgezet zodra de backend de verbinding heeft hersteld, en `--stop` geeft de hiaten weer ([Een DAQ-E die stopt met leveren](#a-daq-e-that-stops-delivering)). Ctrl+C beëindigt een `--duration`-opname voortijdig en meldt dit; zonder `--duration` blijft de opname na Ctrl+C doorlopen (afsluiten `130`) tot `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Wissel het cap-correctieprofiel tijdens de uitvoering. `--cap-id` is vereist; `--cap-id none` verwijdert de cap-correctie (een sensor zonder kap). Een DAQ-E-S behoudt de instelling van `as_recorded`, ongeacht wat er is ingesteld. |
| `daq pool-enable-ptp --sensor-id ID` | Schakel PTP weer in voor een DAQ-E waarvan PTP is uitgeschakeld. De instelling wordt op het apparaat opgeslagen, zodat een apparaat dat is uitgeschakeld, uitgeschakeld blijft — de tijdstempels worden niet via PTP gesynchroniseerd — totdat dit commando wordt uitgevoerd. `pool-list` markeert zo’n apparaat (`!! PTP is switched OFF`) en geeft deze opdracht ervoor weer. Chloros
zet een apparaat nooit vanzelf weer aan, omdat ‘uit’ opzettelijk kan zijn. |
| `daq pool-info --sensor-id ID [--json]` | De eigen instellingen van een DAQ-E: de firmware (en of deze Chloros
-release een nieuwere versie bevat), het adres en hoe dit is verkregen (DHCP / link-local / statisch), de opgeslagen netwerkinstelling totdat de sensor deze uitvoert (een instelling die is opgeslagen met `--no-reboot` inbegrepen), de stream bij het opstarten en PTP. Bij een sensor waarvan de stream deze computer niet bereikt, leest het de sensor bij een nieuwe verbinding en meldt dit. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Stel een DAQ-E in op DHCP (`--auto`) of een statisch adres (`--mask` standaard `255.255.255.0`). De sensor start opnieuw op om de instelling toe te passen en maakt opnieuw verbinding met de instellingen die hij had, waarbij hij maximaal `--timeout` seconden (standaard 45) hierop wacht; `--no-reboot` slaat het alleen op. Zie [DAQ-E-apparaatinstellingen](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Of een DAQ-E bij het opstarten begint met streamen zonder dat er software is aangesloten (opgeslagen op de sensor). Chloros
start de stream in beide gevallen zelf. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Een DAQ-E opnieuw opstarten en opnieuw verbinden met de vorige instellingen (wacht maximaal `--timeout`, standaard 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Flash de DAQ-E-firmware die bij deze Chloros
-release is meegeleverd naar een sensor waarop een oudere versie draait, en sluit deze vervolgens opnieuw aan (ongeveer een minuut; schakel de stroom niet uit; `--timeout` standaard 45 s voordat hij weer actief is). Een sensor die al up-to-date is, wordt met rust gelaten. |

### Een DAQ-E die geen gegevens meer verstuurt

Een DAQ-E in de pool die gedurende 20 seconden geen spectrum verstuurt, wordt door de backend opnieuw aangesloten. Dit gebeurt door de switch die de sensor van stroom voorziet uit en weer in te schakelen: de sensor start hiermee opnieuw op en verstuurt geen gegevens meer totdat hij daartoe opdracht krijgt (tenzij `pool-set-auto-stream --on`).

- **Eerst wordt gecontroleerd of het om dezelfde sensor gaat.** Er wordt op het oude adres van de sensor, zonder iets te wijzigen, gevraagd naar de naam, het MAC-adres en het serienummer van de sensor. Hij vraagt ook onder de `daq-e-<id>.local`-naam van de sensor, maar vertrouwt dat antwoord alleen als het afkomstig is van de eigen netwerkadapter (MAC) van de sensor. Er wordt geen verbinding hersteld met of geschreven naar iets anders dat antwoordt.
- **Het maakt opnieuw verbinding met de instellingen van de sessie:** integratietijd, framegemiddelde, automatische belichting en cap-profiel zoals ze waren, niet de standaardinstellingen van de sensor bij het opstarten. Een sensor die PTP ingeschakeld had en terugkwam met PTP uitgeschakeld, wordt weer ingeschakeld. Vervolgens start het de stream opnieuw op, evenals de ontvanger van de computer daarvoor.
- **Er wordt met steeds langere tussenpozen opnieuw geprobeerd:** onmiddellijk, vervolgens na 5, 10, 20 en 40 seconden, en daarna elke 60 seconden zolang de sensor niet bereikbaar is. Bij elke poging wordt de sensor alleen gevraagd wie hij is. Een sensor die wel antwoordt maar na drie herverbindingspogingen nog steeds niets verstuurt, wordt met rust gelaten en gerapporteerd (`auto-reconnect GAVE UP`). `pool-stream --start` probeert het opnieuw.
- **`pool-list` geeft de huidige status weer.** Terwijl de sensor niets van zich laat horen, geeft de regel `STREAM FAULT` aan wat de herverbindingspoging doet. De regel verdwijnt bij het eerste spectrum, waarna de sensor `reconnected automatically N time(s)` weergeeft.
- **Een opname gaat verder in hetzelfde `.daq`.** Het bestand markeert elke onderbreking met twee regels: `event_type` 5 bij het laatste spectrum vóór de onderbreking en 6 bij het eerste erna, elk met alleen een tijdstempel. `pool-record --duration` drukt `!! STREAM LOST …` af terwijl de sensor weg is en `stream resumed after …` wanneer deze terug is. `pool-record --stop` somt elke onderbreking op en geeft een waarschuwing, net zoals bij een leeg bestand (een opname die geen meetwaarde bevat, stopt met `1`).
- **Een opname behoudt één kalibratie.** Als de sensor terugkomt met een andere kalibratie terwijl een opname loopt, behoudt de rest van de opname de kalibratie waarmee deze begon, en de stopwaarschuwing vermeldt dit. `pool-disconnect` + `pool-connect` neemt de nieuwe over.
- **Uw opdrachten hebben voorrang.** Een sensor die u met `pool-disconnect` uitschakelt of met `pool-stream --stop` pauzeert, wordt nooit opnieuw aangesloten, zelfs niet wanneer er al een herverbinding plaatsvindt.
- **`pool-info` op een stille sensor** leest de sensor bij een nieuwe verbinding en voegt `!! stream NOT reaching this computer` toe. Een sensor die niet reageert, wordt onmiddellijk gemeld. `pool-list` toont geen PTP-status meer voor een sensor die niet meer reageert.
- DAQ-U en DAQ-M worden niet behandeld. Een DAQ-M maakt zelf opnieuw verbinding via Bluetooth, en een DAQ-U die van de USB is losgekoppeld, heeft opnieuw `pool-connect` nodig.

### DAQ-E-S — vaste diffusor en kantelsensor

De **DAQ-E-S** is een DAQ-E met twee verschillen die van belang zijn voor de software. Al het andere — detectie, verbinding maken, PTP, opname, de onderstaande apparaatinstellingen — werkt precies hetzelfde als bij een DAQ-E.**De diffusor is vast gemonteerd, dus er is geen correctie via de cap nodig.** De diffusor in Sunshine-stijl is ingebouwd en zat al op het apparaat toen de fabriekskalibratie werd gemeten, zodat die kalibratie al de werkelijke neerwaartse stralingsintensiteit weergeeft. Chloros
stelt de cap in op `as_recorded` (geen profiel), ongeacht wat er wordt gevraagd: een speciaal voor dit apparaat benoemde cap (`sunshine_cosine`, `fov_*`, `none`) wordt genegeerd met een melding, en de opnames slaan `cap_id_source=model` op, waardoor de waarschuwing voor een veronderstelde limiet hiervoor nooit wordt geactiveerd. Pas ook niet handmatig een limiet toe op de gegevens: `sunshine_cosine` bovenaan zou ongeveer 11 keer te hoog uitkomen.

**Het apparaat is voorzien van een 3-assige versnellingsmeter.** De cosinusrespons van de zonnestraalverspreider is alleen geldig wanneer deze horizontaal staat; een gekantelde sensor rapporteert dus een verkeerde stralingsintensiteit — en de reflectie neemt deze fout over — zonder dat er andere symptomen optreden. Elke DAQ-E-S-meting bevat daarom de stand van het apparaat:

| Veld | Betekenis |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Hoeken in graden. `None` wanneer de firmware de hoek als ongedefinieerd markeert (vrije val, een dode as). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Versnelling in milli-g. Als `mag_mg` ver afwijkt van ~1000, betekent dit dat het apparaat in beweging was en dat de hoeken voor dat monster geen betekenis hebben. |
| `cal_applied` | Ingeschakeld: de assen en hoeken liggen in het referentiekader van de diffuser (de kalibratie van de montage van het apparaat is toegepast). Uitgeschakeld: ruw referentiekader van de printplaat, dat een kanteling tot ~2,3° kan weergeven die niet reëel is. |
| `sample_age_ms` | Leeftijd van de versnellingsmetermeting op het moment dat het spectrum werd verzonden. |
| `temp_raw` | Ongekalibreerde **trend** van de chip-temperatuur — alleen verschillen, nooit een thermometerwaarde. |

`daq pool-latest --json` geeft deze weer onder de `imu`-sleutel van elk frame (`null` op een DAQ-U, DAQ-M of gewone DAQ-E). Een `.daq` die is geregistreerd vanaf een DAQ-E-S slaat dezelfde waarden op in de `imu_*`-kolommen (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) naast elk spectrum, zodat een lezer een wolk kan onderscheiden van een omgevallen statief. Bij alle andere modellen zijn die kolommen leeg.

De houdingsstream wordt apart gerapporteerd: `daq pool-list` geeft `!! ATTITUDE STREAM FAULT (spectra unaffected)` weer terwijl het apparaat uitgeschakeld is, en hoeveel fouten het daarna heeft hersteld. Een fout daar leidt nooit tot een gat in de spectrale opname. `daq pool-discover` vermeldt het apparaat als `DAQ-E-S` (als `DAQ-E` als het niet binnen ongeveer 2 s op de identiteitscontrole reageert).

### DAQ-E-apparaatinstellingen

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` zonder `--gateway` slaat geen gateway op** (een eerder ingestelde gateway wordt verwijderd), wat geschikt is voor een cameranetwerk zonder router.
- **Er moet een statisch adres aanwezig zijn op het netwerk van deze computer.** Chloros
weigert een adres waarvan het subnet het adres van deze computer op het netwerk van de sensor niet bevat: na de herstart zou niets de sensor kunnen bereiken om dit terug te zetten. Geef `--force` alleen door als deze computer een nieuw adres krijgt of naar het nieuwe subnet wordt gerouteerd.
- **Een herstart wordt geweigerd terwijl de sensor aan het opnemen is.** Stop eerst de opname (`pool-record --stop`). Een opname die vlak voor een herstart is gestart, wordt daardoor beëindigd, en `pool-set-network`, `pool-reboot` en `pool-update-firmware` eindigen met een `Warning:`-regel waarin het bestandsnaam van de opname wordt vermeld (ook als de sensor daarna niet terugkomt).
- **Na een herstart verbindt Chloros
alleen de sensor zelf opnieuw.** Het wacht op de plek waar de sensor zich bevond — en na `--auto` ook onder de naam `daq-e-<id>.local` — op een antwoord met de naam van de sensor, hostnaam en model, via de netwerkadapter (MAC) die het had, en maakt opnieuw verbinding op het adres dat heeft geantwoord. Een antwoord onder die naam wordt alleen geaccepteerd met dat MAC-adres, dus vanaf een computer die de sensor via een router bereikt, of vanuit WSL2 in de standaard NAT-netwerkconfiguratie, wordt een sensor die door DHCP naar een nieuw adres is verplaatst niet opnieuw verbonden: het commando geeft dit aan en geeft het commando `pool-connect --eth-host <its address> …` weer, waarmee de sensor wordt teruggehaald met de oorspronkelijke instellingen voor belichting en sluitertijd.
- **`pool-update-firmware` weigert een sensor waarvan de firmwareversie niet kan worden gelezen.** Een sensor die er langer over doet dan de wachttijd om terug te komen na de flits, is niet defect: zodra hij antwoordt, wordt hij opnieuw verbonden met de instellingen van de nieuwe firmware ingevuld, en het commando waarschuwt dat hij traag was.
- **Een firmware-update vereist een inkomende TCP-verbinding.** De sensor maakt opnieuw verbinding met deze computer om de opname op te halen. Chloros
luistert op de adapter die naar de sensor leidt (op elke adapter, alleen wanneer het niet kan bepalen welke) en bedient alleen het eigen adres van de sensor; elke andere verbinding wordt gesloten zonder te worden bediend. Op Windows
laat de firewallregel `Chloros Backend (TCP-In)` van het installatieprogramma deze verbinding toe (zie [Windows
(.exe)](#windows-exe)); een firewall die dit blokkeert, zorgt ervoor dat de update mislukt met foutcode `… device never connected back on TCP <port> …`.
- **Gebruik onder WSL2 „mirrored networking“** (`networkingMode=mirrored` in `%UserProfile%\.wslconfig`). In de standaard NAT-modus is het adres van deze computer op het netwerk van de sensor het adres van WSL zelf, dus elk statisch adres wordt geweigerd (behalve `--force`), de livestream bereikt WSL niet en een firmware-update kan niet worden voltooid, omdat de sensor weer verbinding maakt met de computer om de image te ontvangen.
- **De sensor-ID en hostnaam veranderen niet.** Wat het adres ook is, een DAQ-E blijft `daq-e-<id>` en `daq-e-<id>.local`, dus `--eth-host daq-e-<id>.local` blijft hem vinden.
- **De GUI heeft dezelfde instellingen**: tabblad „Lichtsensoren“, de instellingen van de sensor (tandwiel), „Netwerk“. De GUI behoudt zijn eigen verbinding met een sensor, dus als je een sensor wijzigt die de GUI heeft geopend vanuit de GUI: een herstart via de CLI
beëindigt de sessie van de GUI met die sensor.
- De SDK
-equivalenten zijn `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` en `update_firmware()`.

---

## `chloros-cli project`

Een opgeslagen Chloros
-project openen, er verbinding mee maken en aansturen (een map met `cameras.json` + `sensors.json` + `project.json`). Alles verloopt via de backend, zodat de GUI en de CLI
dezelfde hardwarestatus weergeven.

### Overzicht van subcommando’s

| Subcommando | Doel |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Het apparaatmanifest van het project (camera&#x27;s, arrays, sensoren) afdrukken, uit de bijbehorende bestanden lezen en de bronbestanden controleren (zie [Een verplaatst project](#a-moved-project---relink)). Het project wordt niet in de backend geopend: het project dat daar geopend is, en de bijbehorende apparaten en opnames, blijven ongewijzigd. Sluit `1` af wanneer geen van de bronbestanden van het project kan worden gevonden. |
| `project devices PATH [--reconnect]` | Verkenning weergeven of opnieuw uitvoeren. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Elke opgeslagen camera / array / sensor verbinden. Elke array wordt verbonden zoals bij `lattice array-connect`: tot 600 s, waarbij de stappen worden afgedrukt naar stderr (`[array <id>]    <s> s: <step>`; stdout is het JSON
-rapport), en een array die binnen die tijd niet is aangesloten, wordt ongedaan gemaakt — de camera&#x27;s worden teruggezet naar de toestand waarin ze werden aangetroffen — en gerapporteerd onder `errors`; de verbinding blijft nooit bestaan. `--packed` stelt voor elke array het gecomprimeerde 12-bits transport in (standaard `auto`; een project slaat geen instellingen per array op; zie [Gecomprimeerd 12-bits transport](#packed-12-bit-transport---packed)). Elke lichtsensor wordt in de DAQ-pool van de backend geopend op het opgeslagen adres met de opgeslagen integratietijd, framegemiddelde, automatische belichting en cap: een sensor waarvoor geen cap is gekozen, krijgt `sunshine_cosine`; een sensor die ‘bare’ is opgeslagen (`none`) blijft ‘bare’; en een DAQ-E-S neemt nooit een cap aan. Het DAQ-paneel van de GUI maakt geen deel uit van de pool: sluit een sensor daar eerst. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Enkele opname van een benoemde camera of array (`--prefix` standaard `capture`). `--orient` (arrays): de array slaat de ruwe gegevens op, die in de oorspronkelijke indeling blijven — dit geeft aan dat de producten die `process` hieruit maakt, moeten worden georiënteerd, zoals `array-capture --orient`. Standaard: native. `project burst` / `stream` en de `capture` / `burst` / `stream` (`orient: true`) hebben dezelfde optie. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N-frame-burst van een genoemde camera of array (`-n/--count` standaard 5; `-i/--interval` seconden tussen frames, standaard 0). Array-bursts verwijderen dubbele, herhaalde gesynchroniseerde groepen (verouderingscontrole), zodat een array die niet de volledige cyclus doorloopt geen N kopieën van één frame kan teruggeven; geeft de resultaten per iteratie weer. Een opname die halverwege mislukt, stopt de burst en sluit `1` af, waarbij wordt aangegeven hoeveel er daarvoor zijn opgeslagen. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Opname naar schijf in een lus: `--fps F` neemt een seconde (standaard 3), één frame per opname van een camera, één gesynchroniseerde framegroep uit een array (met dezelfde verouderingswatchdog als `project burst`), totdat `-n N` (standaard `0` = tot Ctrl+C, waarbij wordt weergegeven hoeveel er zijn opgeslagen en `0` wordt afgesloten, of `1` wanneer er niets is opgeslagen of een bestand is mislukt; een Ctrl+C voordat `-n N` de stream vastlegt, sluit `130` af). Een opname die halverwege mislukt, stopt de stream en sluit `1` af, waarbij wordt aangegeven hoeveel frames ervoor zijn opgeslagen. `--poll-interval` is verouderd en wordt genegeerd: het wordt nog steeds geaccepteerd zodat oudere scripts blijven draaien, met een opmerking op stderr. |
| `project sensor read PATH NAME [--json]` | Laatste spectrumframe. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Neem een .daq-bestand op voor `--seconds` (standaard 10). Een opname die geen meetwaarden bevat, wordt gemeld op stderr en  wordt afgesloten. |
| `project run PATH RECIPE.yaml` | Voer een YAML-/JSON
-opname-recept uit. `--dry-run` valideert zonder uit te voeren. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Bereken uitlijning voor een array — zie [de vlaggentabel hieronder](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Het huidige uitlijningsprofiel afdrukken. |
| `project align clear PATH NAME` | Het profiel uit de cache verwijderen. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | De transformatie van één slave verschuiven (standaard `0`, `0`, `0`, `1.0`: geen wijziging). |
| `project align export PATH NAME --to FILE` | Sla het profiel op als JSON
. |
| `project align import PATH NAME --from FILE [--no-validate]` | Laad een opgeslagen profiel. |

#### `project align calibrate` Opties

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Uitlijningsmethode. **Deze spellingen verschillen van `lattice align-calibrate`**, dat de verkorte vormen `orb` / `akaze` / `phase`; de twee commando’s zijn bij deze vlag niet onderling uitwisselbaar. |
| `--model {translation, rigid, affine, homography}` | `affine` | Model transformeren om te laten passen. |
| `--frames N` | `1` | Gesynchroniseerde framesnapshots naar gemiddelde. |
| `--reference SN` | de master | Serienummer van de referentiecamera; alle andere leden worden hiernaar vervormd. |
| `--max-features N` | `5000` | Maximumaantal ORB-kenmerken. |
| `--ratio-threshold F` | `0.75` | Lowe&#x27;s-verhoudingstest. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-drempelwaarde voor inliers. |
| `--min-matches N` | `15` | **Kwaliteitsdrempel** — wijs de oplossing af als er minder dan dit aantal inlier-overeenkomsten zijn. |
| `--max-reproj-err-px F` | `4.0` | **Kwaliteitsdrempel** — wijs de oplossing af boven deze RMS-herprojectiefout. |
| `--checkerboard RxC` | — | Bordgeometrie voor `--method checkerboard`, bijv. `9x6`. |
| `--name PROFILE` | leeg | Profielnaam ingebed in de opgeslagen JSON
. **Niet de array-naam** — dat is de positionele `NAME`. |

De twee kwaliteitscontroles zijn de reden waarom een kalibratie weliswaar succesvol kan zijn bij het oplossen, maar toch
wordt geweigerd om op te slaan: een profiel dat aan een van beide niet voldoet, zou stilzwijgend elke
latere opname verkeerd registreren, dus wordt het geweigerd in plaats van opgeslagen.

### Een verplaatst project (`--relink`)

De `project.json` van een project benoemt elk bronbestand — elke afbeelding en elke opname van de lichtsensor
— aan de hand van het volledige pad. Verplaats de projectmap, of de hele map met projecten,
naar een andere schijf, en die paden verwijzen nog steeds naar de oude locatie.

Bij het openen van een project wordt zelfstandig hersteld wat mogelijk is: er wordt gezocht naar een bestand dat zich in de projectmap
bevond in de map waar het project zich nu bevindt, en elk pad dat daar wordt aangetroffen
wordt opnieuw gekoppeld en opgeslagen (`project.json` wordt eerst geback-upt als
`project.json.before-relink-<date>-<time>`). Dit gebeurt bij elke keer openen – van de app,
`chloros-cli project open`, `chloros-cli process` in een bestaand project, de SDK
’s
en `chloros-mcp`’s. Een pad wordt alleen aangepast naar een bestand dat daar aanwezig is.

Bestanden die zich buiten de projectmap bevonden – een hub-sessie, een import via ‘Map toevoegen’ –
kunnen op die manier niet worden gevonden. Geef de map waarin ze zich nu bevinden een naam:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` neemt de map waar de bestanden naartoe zijn gegaan, of een willekeurige map daarboven: de map van elk ontbrekend
bestand wordt daaronder gezocht op basis van de achterliggende mapnamen
(`Hub Projects/<session>/raw/<serial>`, vervolgens `<session>/raw/<serial>`, ...), en de
map worden de bestanden gehaald uit de map waarin de meeste ervan zijn gevonden. `project open` geeft weer
wat opnieuw is gekoppeld, hoeveel bronbestanden zijn gevonden en in welke mappen de rest zich
bevond. Er ontbreken er nog enkele: een waarschuwing, afsluiten van `0` – het project wordt met de rest geopend. Geen
gevonden: een foutmelding met de namen van die mappen en `--relink`, afsluiten van `1`; de backend
weigert een dergelijk project te openen voor `process` of `project devices --reconnect` (HTTP
-code 409 `PROJECT_SOURCES_MISSING`).

### Voorbeelden

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Recept-DSL

`project run RECIPE.yaml` accepteert een YAML- of JSON-bestand waarin een reeks acties wordt beschreven:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Ondersteunde acties: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` neemt de sleutels `cameras.json`, `settings`, `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (de oudere namen `exposure_time`, `exposure_auto`, `gain`, `gain_auto` worden als aliassen geaccepteerd). Een op deze manier ingestelde belichting, versterking of doelwaarde wordt als de jouwe beschouwd: de automatische belichtingsregeling van Chloros (inclusief de door de DAQ gemeten belichting) laat deze ongemoeid. Een handmatige belichting laat de versterking ongewijzigd: in de automatische modus blijft de camera de versterking (over het volledige bereik) aanpassen aan de doelhelderheid, totdat je `gain_db` of `auto_gain: "Off"` instelt. Een latere `auto_exposure: "Continuous"` geeft de belichting weer terug aan de automatische regeling, en `target_brightness_auto: true` geeft een vastgehouden doelwaarde weer terug aan de automatische doelwaarderegeling vChloros. De actie `burst` vereist `name` (verplicht), `count` (standaard 5), `interval` (seconden, standaard 0), `output`, `format`, en `settings` (dezelfde vorm van instellingen per camera als `apply`); array-bursts gebruiken dezelfde ‘fresh-synced-group’-watchdog als `project burst`.

Een `capture`, `burst` of `stream`-stap is de standaardinstelling van `output` `output`. Een relatieve `output` — de standaardwaarde is inbegrepen — wordt omgezet naar de map van waaruit je `chloros-cli project run` uitvoert, niet naar de map van het receptbestand of de projectmap (in 1.2.0 kwam het terecht in de installatiemap van de backend).

Voer het uit:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Elke uitvoering wordt eerst gecontroleerd aan de hand van het project.**Elke actie moet precies een van de bovenstaande werkwoorden als sleutel hebben — `- capture: {name: FrontLeft}`, niet `- action: capture` of `- type: capture` — bij een `name` heeft het project `cameras.json` / `sensors.json`, een camera of array voor `apply` / `capture` / `burst` / `stream` en een lichtsensor voor `sensor`; een `stream` vereist dat `count` &gt; 0 is en een `wait` een duur. Een recept met een dergelijke fout voert**geen** actie uit (`[recipe] error: …`, `[recipe] not run: …`, afsluiten `1`), zelfs niet met `stop_on_error: false`. Toetsen die een stap niet gebruikt, zijn een `[recipe] warning:`, net als elke array of lichtsensor die het recept aanstuurt en die nu niet is aangesloten. Een recept sluit nooit apparaten aan: voer eerst `chloros-cli project connect PATH` uit. `--dry-run` drukt het recept af, vervolgens dezelfde fouten en waarschuwingen, en `[recipe] valid: …` / `NOT valid: …`, voert niets uit en sluit `1` af wanneer het recept een fout bevat.

---

## Omgevingsvariabelen

| Variabele | Effect |
| --- | --- |
| `CHLOROS_BACKEND_URL` | De backend-URL overschrijven (standaard `http://127.0.0.1:5000`) — **wordt alleen ondersteund door de commandofamilies `lattice`, `project` en `daq pool-*`** (`lattice network-analysis` gebruikt in plaats daarvan `--backend-url`). De kerncommando’s (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) koppelen aan `http://127.0.0.1:<port>` en deze variabele negeren (de IPv4-letterlijke waarde omzeilt de Windows-penaliteit van `localhost`→`::1` van ~2 s per verzoek), zodat ze altijd de lokale machine als doel hebben. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Ingesteld in de omgeving van de backend. `1` verzwakt de weigering bij overinschrijving — een array-verbinding met `pin_resolution`, of een [tijdelijke multi-camera-opdracht](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — naar een luide waarschuwing, waarbij pakketverlies bij GVSP wordt geaccepteerd. Alleen voor testdoeleinden — zie [Array fps &amp; burst-model](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Instellen in de omgeving van de backend: `on` of `off` stelt de standaardinstelling in op [gecomprimeerd 12-bits transport](#packed-12-bit-transport---packed) voor elke array (standaard `auto`). Een expliciete `--packed` heeft nog steeds voorrang. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (of `true` / `yes`) schakelt de [automatische updatecontrole](#chloros-cli-update) van de Linux-CLI bij het opstarten uit. |
| `CHLOROS_CLI_MODE` | Ingesteld door de CLI zelf; geeft de backend de opdracht om parallelle verwerking in te schakelen. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` slaat de GVSP-fallback-probe over (alleen ICMP-resultaten). **Dit schakelt jumbo uit, het zorgt er niet alleen voor dat het logboek stiller wordt** — de camera beantwoordt DF-pings slechts tot 1500 op elk pad, dus deze test is het enige dat jumbo kan detecteren. Bespaart ~1 s per camera per verbinding; kost ~1,45× de maximale bandbreedte als het netwerk jumbo *zou* kunnen verwerken. De SDK geeft een waarschuwing wanneer je dit instelt. |
| `CHLOROS_GVSP_SESSION_REUSE` | Instellen in de omgeving van de backend: `0` meet de pakketgrootte bij elke verbinding in plaats van één keer per camera per backend-sessie. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | GVSP-pakketgrootte vastzetten op N bytes; het peilen wordt volledig overgeslagen. Geef de voorkeur aan per commando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) boven een permanente instelling: een vastgelegde grootte past zich niet meer aan het onderliggende netwerk aan, en het vastleggen van 9000 op een pad dat geen jumbo-pakketten kan verwerken, zorgt ervoor dat **elke** opname een time-out krijgt met `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | In de omgeving van de backend instellen: het maximale aantal seconden dat een [tijdelijke multi-camera-opdracht](#multi-camera-transient-sessions), of een `lattice capture` van een camera die niets vasthoudt, besteedt aan het instellen van de belichting van elke camera op basis van de scène voordat deze een frame vasthoudt (standaard `5`). `0` schakelt dit uit: de automatische belichting van elke camera wordt dan vanaf het eerste frame uitgevoerd, beginnend bij de belichtingslimiet van die camera. |
| `CHLOROS_LOG_DIR` | In de omgeving van de backend instellen: de map waar het gedetailleerde logboek naartoe gaat. Standaard `%LOCALAPPDATA%\Chloros\logs` op Windows, `~/.cache/chloros/logs` op Linux (`$XDG_CACHE_HOME/chloros/logs` wanneer dat is ingesteld). De root-service van het .deb-pakket stelt `/var/log/chloros` in, leesbaar voor de groep `adm` — zie [De backend als service uitvoeren](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Locatie voor tijdelijke bestanden. De CLI gebruikt automatisch `/mnt/ssd/tmp` indien aanwezig. |

---

## Uitgangscodes

| Code | Betekenis |
| --- | --- |
| `0` | Succes. |
| `1` | Algemene fout (de meeste fouten bij subcommando’s). Een opnamecommando beëindigt ook met `1` wanneer het schrijven van een bestand dat moest worden opgeslagen is mislukt, ook al zijn de andere bestanden wel opgeslagen, en met `array-burst` / `array-record` stopt `1` wanneer de recorder een fout meldt — zie [Bestanden die niet kunnen worden geschreven](#files-that-fail-to-write). `process` beëindigt ook `1` wanneer de exportgegevens van een run niet konden worden geverifieerd, of wanneer een afbeelding verloren is gegaan, hoewel de andere producten wel zijn opgeslagen. |
| `2` | Argumentfout (een onbekende of ontbrekende vlag; ook `time-sync set-priority` zonder prioriteit). |
| `130` | Onderbroken door Ctrl+C: elk commando dat hierdoor wordt afgebroken — `lattice`, `project`, `daq` en `process` (`process` nadat de backend de uitvoering heeft gestopt, zie [`process` Opmerkingen](#notes)). Een commando dat normaal gesproken met Ctrl+C wordt gestopt — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` en `project stream` met `-n 0` — wordt afgesloten op basis van wat het daadwerkelijk heeft gedaan: `0` wanneer alles was geschreven, `1` wanneer er iets mislukte. |
| `143` / `129` | Beëindigd door SIGTERM / SIGHUP (128 + het signaal), nadat de opruimactie van het commando was uitgevoerd: een lopende taak werd gestopt en de bijbehorende camera&#x27;s werden teruggezet, een verwerkingsrun werd gestopt in de backend. Een `lattice continuous` of andere opname die normaal gesproken met Ctrl+C wordt gestopt, sluit ook `143` af, niet `0`. |

`lattice network-analysis` beëindigt `1` bij de uitspraken `needs_force_slip` en `error`, en `selftest` sluit `1` af, tenzij alle zeven controles zijn geslaagd.

---

## Tips voor het oplossen van problemen

- **&quot;Inloggen vereist&quot;** → Voer `chloros-cli login EMAIL PASSWORD` één keer uit op deze computer.
- **&quot;De projectmap is te diep voor deze computer&quot;** → Lange Windows-paden zijn op deze machine uitgeschakeld (de standaardinstelling van Windows), dus geen enkel pad mag langer zijn dan 260 tekens, en producten staan tot ongeveer 130 tekens onder de projectmap. Er is niets verwerkt. Het bericht geeft de langste projectmap aan die past: gebruik een kortere (`-o` of `chloros-cli set-project-folder`), of schakel lange paden in Windows in en start Chloros opnieuw op. Het Windows-installatieprogramma biedt dit aan als „Windows-lange paden inschakelen“, standaard aangevinkt; gebruik anders Groepsbeleid: *Computerconfiguratie &gt; Beheersjablonen &gt; Systeem &gt; Bestandssysteem &gt; Win32-lange paden inschakelen*.
- **&quot;backend onbereikbaar&quot;** → Start de Chloros-desktopapp (Windows), of start de backend zelf: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` op Windows (de map `backend` in je installatiemap; deze staat niet in `PATH`), `chloros-backend` op Linux. Of controleer `CHLOROS_BACKEND_URL` als het om een externe server gaat.
- **`lattice`-opdrachten mislukken met de foutmelding „LATTICE-camerastuurprogramma’s niet gevonden”** → De backend kon de LATTICE-cameraruntime waarmee deze is geïnstalleerd niet laden. Elke `lattice`-opdracht wordt op de backend uitgevoerd; de CLI-omgeving zelf bevat geen camerastuurprogramma’s. Installeer Chloros opnieuw: het Windows-installatieprogramma en het .deb-pakket installeren de runtime naast de backend.
- **Array connect / Array Settings geeft &quot;FRAMES WILL DROP&quot; of &quot;Reduce ROI to enable&quot; weer** → De ontvangstring van de host-NIC is te klein (wordt vaak teruggezet naar 32 na een update van het NIC-stuurprogramma). Zie [Host-NIC-configuratie en -afstemming](#host-nic-setup--tuning-lattice-arrays) — stel `ReceiveBufferLen=256` en `PendingReceives=64` in.
- **De machine loopt vast bij het opnieuw opstarten/afsluiten, vervolgens verschijnt WMI-fout `Invalid class` / NIC kan niet worden ingeschakeld / USB-stations ontbreken** → Verouderd stuurprogramma voor de USB 10GbE-adapter veroorzaakt fout `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Werk het stuurprogramma van de adapter bij — zie [Host-NIC instellen en afstemmen](#host-nic-setup--tuning-lattice-arrays).
- **Linux: opnames zijn eigendom van root, dus je kunt ze niet verwijderen, en het logboek van de backend staat niet in je `~/.cache`** → De backend draait als root via `chloros-backend.service`. Schakel over naar `chloros-backend@<user>.service`, waar de backend als u wordt uitgevoerd — zie [De backend als service uitvoeren](#running-the-backend-as-a-service-headless). Tot die tijd staat het logboek van de root-service in `/var/log/chloros` (leesbaar voor de groep `adm`).
- **Linux: een backend-eenheid is `failed` met de melding „Er draait al een andere instantie“ (uitgangscode 98)** → Een andere backend bezet de poort al: er kan slechts één tegelijk draaien. Als er twee backend-eenheden zijn ingeschakeld, behoud er dan één: `sudo systemctl disable --now <unit>` voor degene die je niet gebruikt — zie [De backend als service uitvoeren](#running-the-backend-as-a-service-headless). Als alleen deze unit is ingeschakeld, wordt de poort bezet door een backend die buiten systemd is gestart (door `chloros-backend`, `chloros-cli` of een SDK-script): stop die backend en voer vervolgens `sudo systemctl start <unit>` uit.
- **Jetson-swapwaarschuwing** → Voeg swap op basis van een bestand toe; de CLI geeft de exacte `fallocate` / `swapon`-opdrachten weer.
- **&quot;backend niet bereikbaar&quot; bij een `lattice`-, `project`- of `daq`-commando** → er draait geen backend: die commando’s starten er nooit een. Voer `chloros-cli backend status` uit en raadpleeg [De backend starten](#start-the-backend).
- **&quot;`daq net ...` is niet beschikbaar in Chloros&quot;** (Linux en Jetson) → Sommige instellingen van de DAQ-E zelf zijn gereserveerd voor de MAPIR-service. Netwerkinstellingen, herstarts en firmware-updates zijn `pool-set-network`, `pool-reboot` en `pool-update-firmware`; neem voor de rest contact op met de MAPIR-ondersteuning.
- **&quot;Voor directe hardwareopdrachten van de DAQ is het volledige daq-pakket vereist&quot;** (Windows) → Gebruik `pool-*`, waarmee dezelfde sensor via de backend wordt aangestuurd, of `chloros_sdk.connect_daq_sensor()` vanuit Python.

---

## Zie ook

- [Python SDK-referentie](sdk-reference.md) — programmatisch equivalent van elke CLI-opdracht.
- [DAQ-lichtsensoren](https://mapir.gitbook.io/chloros/daq-light-sensors) in de Chloros-handleiding — de sensoren, hun limieten en gekalibreerd bereik, opname, DAQ-E-netwerken.
- [LATTICE-camera’s](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) in de Chloros-handleiding — aansluiten, camera-instellingen, opnamemodi, arrays.
- Online documentatie: `https://mapir.gitbook.io/chloros/cli`
