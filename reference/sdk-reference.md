# Chloros Python SDK-referentie

**Versie:**

1.3.0**Gemaakt:**29 juli 2026 19:19 ·**Herzien:** 6 november 2026**Pakket:** `chloros-sdk` (PyPI)**Doelgroep:** Geoptimaliseerd voor gebruik door LLM’s; leesbaar voor mensen.**Toepassingsgebied:** Alle openbare klassen, functies en helpers die door `import chloros_sdk` worden aangeboden, met voorbeelden die je kunt kopiëren en plakken, waaronder beeldverwerking, besturing van één camera, gesynchroniseerde arrays, DAQ-sensoren en projectautomatisering, plus de `chloros-mcp`-server voor AI-assistenten.

Als je alleen de hoogtepunten wilt zien, ga dan naar:
- [Installatie & Snelstart](#installatie)
- [Smart-Connect voor LATTICE-arrays](#smart-connect-voor-lattice-camera’s)
- [DAQ-sensorsessies](#daq-sensorsessies)
- [Camerahulpprogramma’s](#camera-hulpprogramma’s)
- [Transient-arrays, uitlijningsprofielen & indices](#transient-arrays-uitlijningsprofielen--indices)
- [Projectmap & recepten](#projectmap--recepten)
- [Projectautomatisering](#project-automatisering--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Live-frames in je eigen pijplijn](#live-frames-in-je-eigen-pijplijn)
- [MCP-server (AI-assistenten)](#mcp-server-ai-assistants)

> **Gewijzigd in 1.3.0** (wat een bestaand 1.2.0-script opmerkt).
>
> **Wijzigingen waarmee een 1.2.0-script mogelijk rekening moet houden:**
>
> - **Een release die de backend niet bevestigt, genereert nu een foutmelding.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` op een array die de sessie heeft geopend, `DAQSensorSession.disconnect()` / `force_disconnect()` op een sensor die de sessie heeft geopend, en `ChlorosProject.disconnect_all()` genereren een `ChlorosConnectError`-fout wanneer de backend de vrijgave weigert of deze niet op tijd bevestigt, omdat het apparaat mogelijk nog steeds bezet is. In 1.2.0 werden ze stil teruggestuurd. Het afsluiten van een `with`-blok genereert nog steeds geen foutmelding.
> - **Een geleend apparaat wordt nooit afgebroken.** `with attach_array(...)` en een expliciete `disconnect()` daarop laten de array actief blijven (in 1.2.0 werd het apparaat bij het verlaten van het blok afgebroken, zelfs wanneer de GUI er eigenaar van was); roep `force_disconnect()` aan als je het wilt beëindigen. Evenzo laat een `connect_daq_sensor()` die de sensor al in de pool aantrof – inclusief de zero-flag smart-detect – deze open staan op `disconnect()` en bij het verlaten van het `with`-blok.
> - **PNG en JPEG accepteren alleen de `display`- en indexproducten.** `capture()` met `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) en een `radiance`- of `reflectance`-niveau, of `processing="all"`, roept `ValueError` aan vóór elk verzoek; een 12-bits `raw` / `debayered` wordt door de backend geweigerd. Zie [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` opent die sensor of genereert** `ChlorosConnectError` met een overzicht van wat het heeft gevonden; het schakelt nooit over naar een andere sensor. Zie [Vastgezet transport](#pinned-transport).
> - **`frame_stream()` genereert een foutmelding in plaats van stil te blijven:** `FrameFetchError` zou bij een weigering in de volgende tick (een reflectiemeting zonder enige DAQ), na 60 s zonder frame en na 60 s zonder nieuwe trigger. Zie [Eén trigger per meting](#one-trigger-per-read--group_id).
> - **Een time-out bij het uitlezen of een verbroken verbinding genereert de foutmelding `ChlorosConnectError`** bij elke smart-connect-aanroep en `analyze_array_network()`, waarbij 1.2.0 de onderliggende time-out of verbindingsfout genereerde.
> - **`connect_array()` wacht standaard 600 s (`timeout`, 180 s in 1.2.0), en een verbinding die wordt opgegeven, wordt ongedaan gemaakt.** De aanroep stuurt zijn `timeout` naar de backend, die elke camera terugzet in de toestand waarin deze werd aangetroffen en geen verbinding tot stand brengt wanneer de verbinding niet op tijd tot stand is gekomen; de aanroep genereert `ChlorosConnectError` (HTTP-status 504 `CLIENT_DEADLINE_PASSED`). In versie 1.2.0 kon een verbinding die direct na het inschakelen van de camera’s tot stand werd gebracht, langer duren dan de aanroep en de array verbinden nadat deze een fout had gegenereerd. `connect_all()` wacht op elke opgeslagen array op dezelfde manier (120 s in 1.2.0). Zie [`connect_array()`-handtekening](#connect_array-signature).
> - **`connect_array()` behoudt de volledige sensor.** Zonder `width` / `height` / `binning` is elke camera ingesteld op de volledige sensor en daalt de beeldsnelheid tot het niveau dat de host en het netwerk leveren; 1.2.0 heeft de frames verkleind of gebundeld om de snelheid op peil te houden. Zie [Synchronized Array](#synchronized-array--arraysession-smart-prep).
> - **Een Bayer-pixelformaat op een M3M (mono) camera is de diepte ervan.** Een M3M biedt alleen mono-formaten, en de voorinstellingen noemen de Bayer-formaten: in 1.2.0 weigerde de camera het formaat en behield zijn eigen formaat, terwijl `settings.pixel_format` het Bayer-formaat benoemde. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, een `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` en een array's gedeelde formaat stellen nu het Mono-formaat met dezelfde diepte in op een M3M: `"BayerRG8"` en `high_speed` leveren `Mono8` op; `"BayerRG12"`, `high_quality` en `triggered` leveren `Mono12` op; `cam.settings.pixel_format` benoemt het. Hetzelfde geldt andersom: een M3C wordt nooit in een Mono-formaat opgeslagen, dus `"Mono12"` — afkomstig van een van deze, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` of het opgeslagen formaat van een array – levert `BayerRG12` op bij een M3C, ongeacht welke camera de master van de array is. Een `CameraSettings` die je doorgeeft, behoudt zijn naam, dus hetzelfde object vraagt nog steeds aan elke camera om die diepte binnen zijn eigen familie. `connect_camera(preset=...)` behoudt het formaat van de preset (1.2.0 stelt de camera in op 12-bit na het toepassen van de preset), en een camera die op een preset wordt geopend, schrijft niet langer wat hij heeft opgenomen naar `PRESETS`. Elk ander formaat dat een camera weigert (een formaat dat ontbreekt in de `PixelFormat`-lijst, zoals `RGB8` op een M3M of `BayerRG16` op firmware die dit niet ondersteunt) krijgt ook geen eigen naam: na een `LatticeCamera`-verbinding, `apply_settings()` of `set_format()`, `cam.settings.pixel_format` en `packed_transport` geven het formaat aan waarin de camera streamt, en één afgedrukte regel geeft het formaat aan dat werd geweigerd.
> - **`analyze_array_network()` geeft aan welke connect van toepassing is:** `pixel_format=None` (voorheen `"BayerRG8"`) betekent de formaten die connect instelt (Mono12 / BayerRG12), en `packed="auto"` de keuze voor gepakt transport, dus het aantal fps dat wordt vermeld kan afwijken van 1.2.0'. Een `pixel_format` dat je doorgeeft, projecteert een M3M op die diepte. Zie [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` retourneert een `CaptureResult`** (nog steeds een lijst met frame-dicts) met `.skipped`, `.failed`, `.daq_files` en `.daq_matches`. Een `ArraySession.capture()`-resultaat bevat ook `.alignment` (of de uitlijningsvervorming is toegepast, en waarom niet) en `.oriented`.
> - **Array-captures worden standaard uitgelijnd, net als die van de app.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` en `ArrayHandle.capture()` / `capture_smart()` passen de `debayered`, `display`, `radiance`, `reflectance` en indexbestanden in het beeld van de hoofdcamera en snijdt deze bij tot de overlap van de leden (kleiner dan de sensor met de offsets van de camera's, `XMP-mapir:AlignmentJSON` `"applied": true`), zodat het dezelfde bestanden zijn als de opname van dezelfde scène door de Chloros-app; `raw` blijft in de oorspronkelijke grootte. Voorheen werden bij een opname zonder `aligned=True` alle bestanden op de oorspronkelijke grootte opgeslagen, zonder vervorming. `aligned=False` behoudt dat. `chloros-mcp`’`capture_array` en `start_capture_run` hanteren dezelfde standaardinstelling. Zie [`ArraySession`-methoden](#arraysession-methods).
> - **`connect_all()` geeft een backend aan die de SDK weigert.** Een actieve backend die is uitgelogd of een abonnement heeft dat lager is dan Copper, wordt gerapporteerd onder `errors` met een eigen reden (HTTP 401 / 403), niet als „er draait geen Chloros-backend“, en elke `connect_all()`-aanroep controleert de backend opnieuw, dus het aanroepen ervan na het inloggen werkt op hetzelfde projectobject.
> - **Een relatief `output_dir` werkt met een lokale backend.** Relatieve paden worden omgezet naar absolute paden ten opzichte van de werkdirectory van je script voordat ze worden verzonden. In 1.2.0 loste de backend ze op in zijn eigen installatiemap, wat mislukte met de foutmelding "Toegang geweigerd".
> - **`numpy` is nu een vereiste** van de SDK (de live-frame-aanroepen retourneren numpy-arrays).
> - **Een naam op exportniveau die niet "one" is, wordt geweigerd.** `processing=` en `levels=` accepteren `raw`, `debayered`, `radiance`, `reflectance` of `display` (of een alias zoals `preview`; `processing=` en ook `"all"`). Elke andere naam wordt geweigerd voordat de camera wordt aangeraakt (HTTP 400 `UNKNOWN_LEVEL`: `capture()` genereert `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); 1.2.0 heeft hiervoor `raw` opgeslagen of geretourneerd.
> - **Texture Aware wordt geweigerd voor LATTICE-opnames.** Bij een project met LATTICE-opnames genereert `configure(debayer="Texture Aware (Slow, Highest Quality)")` een `ChlorosConfigurationError`, en hetzelfde geldt voor `process()` na een `configure()` die hierom vroeg vóór het importeren. Een Texture Aware-waarde die het project eerder had opgeslagen, wordt ingesteld op Standaard, met een `UserWarning`. LATTICE-opnames krijgen altijd de standaarddemosaic, dus 1.2.0 accepteerde de instelling en negeerde deze; de Chloros-app biedt deze optie hiervoor ook niet aan. Zie [Ondersteunde waarden](#supported-values).
> - **Camera’s worden teruggezet naar de toestand waarin ze werden aangetroffen — tenzij `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, het einde van elke tijdelijke functie (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) en het stoppen van `start_camera_continuous()` brengen elke camera terug in de toestand waarin deze werd aangetroffen: de bedrading van de trigger en de synchronisatielijn, de PTP en alle camera-instellingen die tijdens de sessie zijn gewijzigd (belichting en versterking, het AE-doel, limieten en demping, witbalans, zwartniveau, pixelformaat, binning en ROI, spiegelingen, beeldsnelheid). `keep_settings=True` slaat daarentegen de instellingen van elke camera op in het eigen geheugen van de camera, zodat deze daarmee opstart; `clear_settings=True` reset dat geheugen naar de fabrieksinstellingen. In beide gevallen blijven de camera’s vrij draaien. Een project stelt zijn camera’s in op basis van de eigen opgeslagen instellingen wanneer het verbinding met ze maakt. Zie [Camerageheugen](#camera-memory--keep_settings--clear_settings).
> - **Elke modus slaat een project op dezelfde manier op.** Elke wijziging aan een camera, een array of een lichtsensor die onder een project is aangesloten — vanuit de SDK, de app, de CLI of de MCP-server — wordt opgeslagen in de `cameras.json` / `sensors.json`, en als je het project in een willekeurige modus opent, worden de apparaten op basis daarvan ingesteld; een `ChlorosProject` wijst zijn eigen map toe aan de backend. Voorheen sloeg alleen de app het project op. Zie [Eén projectrecord, elke modus](#one-project-record-every-mode).
> - **Bestanden die zijn gemaakt op basis van meerdere camera’s vermelden elke camera afzonderlijk.** Een multiband-stack, een live-index samengesteld uit de banden van meerdere camera’s en een `composite/`-beeld van een array-opname bevatten de serienummers, modellen en kalibratiepakketten van elke broncamera als lijsten (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), en EXIF `SerialNumber` geeft de serienummers weer, gescheiden door komma’s. Vroeger bevatten ze geen cameragegevens, of alleen het serienummer van de eerste camera. Zie [Bestanden gemaakt met meerdere camera’s](#files-made-from-several-cameras).
> - **Alleen een `raw`-bestand bevat de kalibratietags van de camera.** De MicaSense-compatibele set (`BlackLevel`, `RadiometricCalibration`, de per-kanaals `Band{R,G,B}RadiometricCalibration`, de `MicaSenseVignetting*`-sets) en de `CalibrationJSON`-payload beschrijven de ruwe waarden van de sensor, dus deze worden alleen meegeleverd in het `raw`-opnamebestand. Alle andere bestanden — een opname'`debayered`, `display`, `radiance`, `reflectance` en indexbestanden, frames van continu-opname en multi-camera-streamframes, en elk `process()`-productbestand — bevat de identiteit van de camera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) en, bij een opname, de eigen gegevens van het frame (`XMP-mapir:FrameJSON`: belichting, versterking, temperatuur, pixelformaat, opnametijd). In versie 1.2.0 bevatten deze bestanden ook de raw-set, die door een verwerkingstool (Pix4D, Metashape) werd toegepast op pixels die geen raw-waarden waren. Processing bepaalt de kalibratie van een dergelijk bestand aan de hand van het serienummer en `CalBundleSha256`.
>
> **Overige wijzigingen:**
>
> - **De automatische beeldsnelheid van een array wordt afgestemd op wat de array levert.** Een snelheid die meer dan 1 % van zijn triggers verliest, wordt verlaagd – dit wordt beoordeeld op basis van meerdere langdurige metingen in plaats van één – en een snelheid waarnaar eerder is teruggeschakeld, wordt af en toe opnieuw geprobeerd. Een snelheid die je zelf instelt, blijft ongewijzigd.
> - **De automatische belichting is stabieler.** Deze blijft constant bij gelijkblijvend licht, stabiliseert zich binnen enkele seconden na een verandering, negeert kleine overbelichte delen en herstelt de werking van een camera die niet meer reageert in fel zonlicht. Zie [Hoe automatische belichting werkt](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` telt alleen je afbeeldingen en opnames.** In een map die het `camera_cal/`-bestand van een project bevat, worden die kalibratiebestanden niet langer als afbeeldingen en opnames geteld, en een project dat ze al had opgenomen, verwijdert ze.
> - **`configure(radiometric_output=...)` is van toepassing.** Het stelt nu de `Export radiance` / `Export reflectance`-producten van het project in; voorheen hadden de eigen productinstellingen van elk project voorrang, waardoor `radiance`, `sensor-response` en `all` allemaal straling en reflectie exporteerden.
> - **`test_array_sync()` start vanuit de ingeschakelde toestand van de camera’s.** Het schakelt de PTP van de camera’s in voor de run, wacht tot hun klokken zijn gesynchroniseerd en gestabiliseerd vóór de eerste trigger (`ptp_lock`) — elke camera is 120 s gesynchroniseerd, ongeveer 2 minuten vanaf het inschakelen (`ptp_settle_s=` stelt dit in), aangezien een klok die net is gesynchroniseerd gedurende ongeveer 2 minuten een afwijking van maximaal ~200 us vertoont — en zet de PTP van elke camera terug zoals deze werd aangetroffen. Er wordt niet opnieuw gewacht op een camera die nog steeds gesynchroniseerd is van een eerdere test. Vroeger liet het PTP uitgeschakeld op een netwerk met één switch waar geen DAQ-E reageerde, en las het de vrijlopende klok van elke camera als een PTP-klok die niet zou synchroniseren ("PTP is niet vergrendeld", bij elke run).
> - **Een `.daq`-opname geeft aan of de kap is gekozen of verondersteld.** DAQ-U, DAQ-M en DAQ-E worden zonder kap geleverd, en Chloros kan niet zien of er een is aangebracht, dus een sensor die zonder benoemde kap is aangesloten, krijgt standaard `sunshine_cosine`, een *aanname*. Elke Chloros-opname (schema 1.30) slaat nu `cap_id_source` op naast `cap_id`: `operator` wanneer je een kap een naam hebt gegeven (inclusief `none`), `auto_default` wanneer dit de standaardinstelling was. Bij het verwerken van een opname waarvan de cap werd verondersteld, terwijl de DAQ-cap van het project op ‘Auto’ staat, wordt een `[daq-dls] WARNING` afgedrukt waarin het bestand wordt genoemd: een DAQ-U zonder extra instellingen onder de zonnecorrectie geeft een waarde weer die ongeveer 30x te hoog is. **Geef de door jou geïnstalleerde cap een naam** wanneer je aansluit (`connect_daq_sensor(cap_id=...)` of `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, of de keuzelijst voor condensatoren in de GUI), of stel de DAQ-condensator van het project in, en de waarschuwing verdwijnt.
> - **Geen pip-consolevensters meer op Windows.** Op een pc waarop Python is geïnstalleerd, voerden de backend en elk van de bijbehorende image workers bij het opstarten één keer `pip` uit, waarbij elk een consolevenster opende. Dat gebeurt nu niet meer.
> - **Live `.daq`-opnames blijven behouden bij een stroomstoring.**Een opname wordt bij elke commit achter de sensor weggeschreven met een logboek en een volledige schijfsynchronisatie, zodat een trage schijf het bestand vertraagt, maar nooit de meetwaarden. Een opname die halverwege een commit is gestopt (stroomstoring, afsluiten) kan `<name>.daq-journal` ernaast achterlaten:**kopieer de twee samen.** Chloros zet het journaal terug bij de eerste keer dat het bestand wordt gelezen, zodat de opname wordt geopend met alles wat vóór de stop was vastgelegd (ongeveer de laatste 3 s gaan verloren); in 1.2.0 kon een stroomstoring op het verkeerde moment het hele bestand onleesbaar maken („database disk image is malformed“).
> - **`DAQSensorSession.record_start()` op een drukke schijf zorgt er niet langer voor dat de sensor vastloopt**: metingen worden in de wachtrij geplaatst en op een eigen thread geschreven, en een meting die niet kon worden geschreven, wordt meegeteld in `status()`'s `record_errors` (samen met `record_last_error`).
> - **Direct `LatticeCamera`:**een frame zonder chunk-belichting/versterking rapporteert nu de**werkelijke** belichting en versterking van de camera (de fallback las een waarde uit de cache die gedurende een hele automatische belichtingscyclus bevroren kon blijven), en `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` passen dezelfde 5 µs-ingangsdeglitch toe die de kabelslaves van Chloros krijgen (ongefilterd: een slave die frames afvuurt zonder dat er een trigger is verzonden).
> - **Softwaregestuurde camera’s laten niet langer om de andere trigger vallen.** Een camera in de software-triggermodus zet nu een trigger die tijdens het uitlezen binnenkomt in de wachtrij (of Chloros wacht tot deze gereed is), zoals bij kabelsynchrone camera’s al het geval was.
> - **`discover_daq_sensors()` benoemt een DAQ-E-S tot `DAQ-E-S`.** De naam `model` was voor elke Ethernet-sensor `DAQ-E`; een sensor die niet binnen ongeveer 2 s op de identiteitscontrole reageert, wordt nog steeds vermeld, als `DAQ-E`.
> - **Nieuw: `chloros-mcp`, een MCP-server voor AI-assistenten.** Claude Desktop, Claude Code, Cursor en andere MCP-clients kunnen beeldmateriaal verwerken (een project aanmaken, opnames importeren, instellingen kiezen, verwerken, rapporteren wat de run heeft geëxporteerd) en LATTICE-camera’s, arrays en DAQ-lichtsensoren bedienen: verbinden, een voorbeeld bekijken, belichting en kleur instellen, opnamen maken, opnemen. De tools dekken de functionaliteit van deze SDK. Het wordt samen met de SDK geïnstalleerd, en het Linux-.deb-pakket plaatst het in `/usr/bin`. Zie [MCP-server (AI-assistenten)](#mcp-server-ai-assistants).
> - **Nieuw: camera-hulpprogramma’s**, de `chloros-cli lattice`-opdrachten als SDK-aanroepen: kleurprofiel, verzadiging / contrast, hoge lichten, witbalans (eenmalig, auto, vaste verhoudingen), voorinstellingen, filtermodellen, firmwarestatus en -update, stroomvoorziening (peilen, slaapstand, ontwaken), doelkalibratie, netwerkadapters (controleren, schatten, repareren), vignetteringscorrectie op bestanden, een DLS-meting en de hostpeiling. Zie [Camerahulpprogramma’s](#camera-utilities).
> - **Nieuw: de rest van de CLI als SDK-aanroepen**, zodat de CLI, de SDK en `chloros-mcp` dezelfde functies bieden (de CLI behoudt om veiligheidsredenen alleen het inloggen, de provisioning van een DAQ-E en de directe DAQ-servers): camera’s die met één commando worden geopend (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), uitlijningsprofielbestanden (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indexbestanden (`index_catalog()`, `compute_index()`, `capture_live_index()`), de PTP-grandmaster (`time_sync_status()` en aanverwante bestanden), de projectmap (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) en recepten (`run_project_recipe()`, de eigen runner van de CLI). Voor lichtsensoren: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` en `light_sensor_reflectance()` (omgevings- en objectreflectie en indices van twee sensoren); voor een camera: `CameraSession.status()` en `capture_repeated()`; voor een array: `ArraySession.capability()` en `set_member_settings()` (belichting, versterking, ... van een element). Zie [Transiënte arrays, uitlijningsprofielen & indices](#transient-arrays-alignment-profiles--indices), [Projectmap & recepten](#project-folder--recipes) en [DAQ-sensorsessies](#daq-sensor-sessions).
> - **Verbeterde LATTICE-indexproducten** (NDVI en andere indices) uit `process()`, in elke verwerkingsmodus.
> - **LATTICE OCN-camera’s: verbeterde radiance-, reflectance- en indexproducten bij zonlicht.** Na de bandunmix wordt een vlootcorrectie toegepast — live, bij opnames en in `process()`; RGN-camera’s blijven ongewijzigd. Zie [Radiometrische output](#radiometric-output-lattice-multispectral-pipeline).
> - **Elke opname slaat de gebruikte DAQ-waarde op, net zoals de GUI dat doet.** `CameraSession.capture()` en `ArraySession.capture()` / `continuous()` schrijven het `.daq`-bestand en het bijbehorende `.match.json` wanneer een DAQ wordt gekoppeld of in een pool wordt geplaatst, inclusief een opname die uitsluitend uit onbewerkte gegevens bestaat; in 1.2.0 werd een 'raw'- of 'debayered'-opname alleen opgeslagen met `force_daq=True`. `daq=False` schakelt dit uit en `daq_tolerance_ms=` stelt de overeenkomingstolerantie in; beide resultaten geven aan wat er is opgeslagen in `.daq_files` en `.daq_matches` is opgeslagen. Een opname wacht nu op de volgende meting van de DAQ, zodat de metingen aan weerszijden van de belichting kunnen worden samengevoegd. Zie [De DAQ-meting die een opname opslaat](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` bevat live-frames en een continue opname aan de serverzijde.** `grab()` en `frame_stream()` retourneren numpy-frames van één trigger (dezelfde uitlezing als de `ArrayHandle.grab()` / `frame_stream()` van de project-API), en `continuous()` slaat elke gesynchroniseerde groep op in de backend en retourneert een [`ContinuousHandle`](#continuoushandle). Zie [Live frames in je eigen pijplijn](#live-frames-in-your-own-pipeline).
> - **Een array `grab()` is gekoppeld aan één trigger** (`ArrayHandle.grab()` en de nieuwe `ArraySession.grab()`). In 1.2.0 leest 12 van de 20 vier-camera `ArrayHandle.grab()`-uitlezingen verspreid over twee triggers. Elk frame bevat nu `group_id`, en een array `frame_stream()` levert elke trigger één keer op. Zie [Eén trigger per read](#one-trigger-per-read--group_id).
> - **Frames per band en uitgelijnde frames:** `grab()` / `frame_stream()` nemen `band=`, `align=`, `bit_depth=` en `refl_max=` op camera- en array-handles. Zie [Een enkele band exporteren](#exporting-a-single-band--band).
> - **Nieuwe `ArraySession`-methoden en -argumenten:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (samen met `CaptureResult.group_seq` en `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` koppelt opeenvolgende groepen aan elkaar en stopt wanneer de schijf vol is.
> - **`ArraySession.record()` registreert zonder dat de GUI is geopend.** Als de gecombineerde weergave van de GUI is gesloten, geeft de backend de weergave zelf weer: de array’NDVI of de index die je doorgeeft als `expr=`. `RecorderHandle.stop()` genereert een foutmelding voor een mislukte opname, waarbij geen enkel frame is geschreven, en behoudt het stopresultaat op `.result` (het achterlaten van een `with`-blok veroorzaakt geen foutmelding: controleer `rec.result`). Zie [`RecorderHandle`](#recorderhandle).
> - **Nieuwe `DAQSensorSession`-methoden:** `enable_ptp()`, `force_disconnect()`, en voor de eigen instellingen van een DAQ-E: `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` en `update_firmware()`. `record_stop()` retourneert `warning` en `gaps`. Een herstart die een opname heeft afgesloten, noemt deze in zijn antwoord of, wanneer de herstart mislukt, in het `ChlorosConnectError` (`recording_closed`; zie [DAQ-E-apparaatinstellingen](#daq-e-device-settings)). Zie [`DAQSensorSession`-methoden](#daqsensorsession-methods).
> - **Een DAQ-E in de pool die stopt met gegevenslevering, wordt door de backend opnieuw aangesloten**, met de instellingen die hij had, en een opname daarop wordt voortgezet in hetzelfde `.daq` met de onderbreking gemarkeerd in het bestand; `record_stop()` geeft de onderbrekingen weer als `gaps`. In versie 1.2.0 bleef de sensor stil totdat deze werd losgekoppeld en opnieuw werd aangesloten, waardoor de opname werd beëindigd. Zie de CLI-referentie onder [Een DAQ-E die stopt met leveren](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` werkt vanuit een geïnstalleerde SDK.** Het wiel (de installatieprogramma’sen PyPI’s) bevat nu de benodigde projectloaders; voorheen veroorzaakte `open_project()` een fout met de naam `ImportError`, waarbij `project_cameras` / `project_sensors` in elke via pip geïnstalleerde SDK. `connect_all()` verbindt de opgeslagen camera’s, arrays en lichtsensoren via een actieve backend (lichtsensoren via de DAQ-pool), en een opgeslagen apparaat waarnaar het niet kan zoeken, wordt onder `errors` vermeld met de reden, in plaats van zonder meer te worden overgeslagen. Een opgeslagen array die al actief is in de backend (die van de GUI’s, of die van `chloros-cli project connect`) wordt gekoppeld en blijft draaien, terwijl dit voorheen als een fout werd gemeld („Camera  staat<sn> al in array <id>”). Zie [`ChlorosProject`-methoden](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` en `process_lattice_capture()` zoeken Chloros op Windows.** Ze zoeken op de locatie waar het installatieprogramma de backend plaatst (standaard `<install folder>\backend\`, `C:\Program Files\Chloros\backend\`, een aangepaste map uit het eigen record van het installatieprogramma), en maken verbinding met een backend die al draait zonder er überhaupt naar te zoeken. In 1.2.0 gaven ze de foutmelding „Chloros-backend niet gevonden” op een geïnstalleerde machine, tenzij je `backend_exe=` doorgeeft.
> - **Een backend die bij het opstarten vastloopt, wordt onmiddellijk gemeld.** Wanneer `ChlorosLocal()` of een smart-connect-aanroep een backend start die afsluit voordat deze reageert, geeft de aanroep binnen enkele seconden de foutmelding `ChlorosBackendError` (van `ChlorosLocal`) en `ChlorosConnectError` (van de smart--connect-aanroepen, met de exitcode, het commando dat het heeft uitgevoerd en de laatste regels die de backend heeft weergegeven. Versie 1.2.0 wachtte de opstarttime-out van 60 s af en gaf geen reden op. Zie [Backend Auto-Start](#backend-auto-start).
> - **De backend blijft draaien tussen scripts door.** Het verlaten van een `with ChlorosLocal(...)`-blok, `process_folder()` / `process_lattice_capture()` die terugkeren en `chloros-mcp` die afsluit, stoppen een door de SDK gestarte backend niet langer; het volgende script hergebruikt deze. De backend stopt bij `logout()` (tenzij de Chloros-app openstaat of er een uitvoering gaande is), bij `shutdown_backend()`, of vanzelf na 30 minuten als er niets gebruik van maakt (`CHLOROS_BACKEND_IDLE_MIN` in de omgeving van de backend verandert dat; `0`: nooit).
> - **Een backend die door de SDK is gestart, wordt gevraagd af te sluiten, niet beëindigd.** `shutdown_backend()` laat de backend zijn camera’s en lichtsensoren vrijgeven en eventuele opnames afronden; deze wordt pas beëindigd als hij 10 seconden later nog steeds actief is. Versie 1.2.0 beëindigde hem onmiddellijk (op Windows een harde afsluiting), waardoor een DAQ-lichtsensor die op die backend was aangesloten, mogelijk niet meer reageerde. Een backend die de SDK als actief aantrof, wordt nog steeds met rust gelaten.
> - **Op Linux voert auto-start nooit een `backend_server.py` uit vanuit de huidige map.** Als er geen backend was geïnstalleerd, voerde een script dat in een map met een backend werd gestart, deze uit; nu wordt alleen de geïnstalleerde backend gestart (of, voor `ChlorosLocal`, degene die `backend_exe=` noemt).
> - **`configure(indices=...)`**voegt elke preset toe aan het project voor de banden van de betreffende camera, inclusief een nieuw project, en geeft een waarschuwing (`UserWarning`) voor een naam die niet kan worden toegevoegd.**`import_images()`** rapporteert `.daq`-opnames als `scan_count` en geeft geen waarschuwing meer over een LATTICE-opnamemap.
> - **`read_image_audit_tags()` werkt vanuit een pip-installatie.** Het leest zelf de metagegevens van de afbeelding; in 1.2.0 retourneerde het `{}` zonder de interne bestanden van de desktopinstallatie in het Python-pad.
> - **`capture()` direct na `connect_array()`** wacht maximaal 20 seconden op de eerste framegroep van de array in plaats van te mislukken met HTTP-statuscode 503 "Array still warming up".
> - **`status()` meldt een stilstand.** Het nieuwe veld `stalled` is True terwijl de opnamelus draait, maar publiceert niets, en `actual_fps` is dan 0. In 1.2.0 behield `actual_fps` zijn laatste waarde tijdens een stilstand, dus een vastgelopen array toonde `actual_fps > 0` terwijl opnames mislukten met 503 "Geen nieuwe gesynchroniseerde framegroep".
> - **Minder onvolledige framesets en 503-fouten bij het vastleggen:** de trigger-ingang van elke slavecamera is nu ontdaan van glitches.
> - **Gecomprimeerd 12-bits transport** (`connect_array(packed="auto")`) verhoogt de framesnelheden van de volledige sensorarray met ongeveer een derde. Zie [Gecomprimeerd 12-bits transport](#packed-12-bit-transport-packed).
> - **Belichting van de array binnen een ROI:** een kleine ROI beperkt de belichting van een arraylid niet langer tot de free-run-periode van de sensor (4,9 ms bij 768 × 304). Zie [Belichting op arrayleden](#exposure-on-array-members).
> - **Automatische belichting verlaagt de automatische beeldsnelheid van een array niet langer.** Een via kabel gesynchroniseerde array werkt op de snelheid die het netwerk en de uitleescapaciteit van de camera’s toestaan; automatische belichting vult de periode die deze snelheid biedt en verhoogt vervolgens de versterking. Een handmatige belichting die langer duurt dan de periode vertraagt de trigger om deze aan te passen in plaats van te worden geweigerd. Zie [Belichting op array-elementen](#exposure-on-array-members).
> - **`read_nodes()`**voegt `frame_values` en `notes` toe terwijl automatische belichting actief is (zie [`read_nodes()` bij automatische belichting](#read_nodes-under-auto-exposure--frame_values-and-notes)). Wanneer de camera een belichting of versterking weigert, geeft**`set_settings()`** het bereik aan dat wel wordt geaccepteerd en schakelt de automatische modus weer in die voor die waarde was uitgeschakeld; `set_settings(target_brightness_auto=True)` is nieuw.
> - **`orient=`** op `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` en `ArrayHandle.capture()` / `capture_smart()` draait de producten mee met de oriëntatie van de camera. Zie [`ArraySession`-methoden](#arraysession-methods).
> - **`DLS()` heeft de eigen `mac_address` van de sensor nodig.** Er is geen standaard-MAC meer (dit was het adres van één eenheid); `DLS()` zonder MAC genereert een `DLSError`. Zie [DAQ-M-bandmagnitudes](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Het Windows-installatieprogramma** biedt de opties „Windows lange paden inschakelen“ en „De Chloros-backend uitsluiten van Microsoft Defender-scans“, die beide standaard zijn aangevinkt. Zie [Stap 1](#step-1--install-the-chloros-platform-package).
> - **Het Linux-.deb-pakket** installeert de Python-SDK voor het systeem `python3` (voorheen werd alleen het wheel-pakket op de schijf geplaatst), vergroot de socket-ontvangstbuffers van de kernel die nodig zijn voor camerastreams, en voegt een backend-service toe die onder jouw gebruikersaccount draait. Het Jetson-pakket is `chloros_1.3.0_arm64_jp6.deb`, voor JetPack 6, met JetPack’s CUDA PyTorch. Zie [Stap 1](#step-1--install-the-chloros-platform-package) en [Stap 2](#linux-deb).

---

## Architectuur in 60 seconden

De SDK is een dunne Python-laag bovenop de Chloros-backend (dezelfde Flask-server die de desktop-GUI en CLI gebruiken). Voor automatisering importeer je `chloros_sdk` en roep je methoden op hoog niveau aan; achter de schermen wordt elke aanroep een HTTP-verzoek naar de lokale backend op poort 5000 — `http://127.0.0.1:5000/api/...` (bewust niet `localhost`, dat op Windows eerst wordt omgezet naar `::1` en ~2 s per verzoek kost bij een backend die alleen IPv4 ondersteunt). De backend beheert de hardwarepool — camera’s, DAQ-sensoren, uitlijningsprofielen, framebuffers — zodat SDK-scripts naast de GUI kunnen bestaan zonder te hoeven strijden om seriële poorten of NIC-bandbreedte.

Er zijn drie interfaces die je zult gebruiken:

1. **`ChlorosLocal` + vrije functies** (`process_folder`, `process_lattice_capture`) — Beeldverwerkingspijplijn. Verwerk een volledige map via kalibratie / debayer / indexexport met één Python-aanroep.
2. **Smart-connect-handles** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Open een permanente backend-sessie voor live hardware. Dezelfde „smart-prep“-workflow als de GUI: netwerkprobe, automatische tier-selectie, PTP, AE-seeding, GPIO-triggerconfiguratie.
3. **`ChlorosProject` / `open_project`** — Laad een opgeslagen project (map met `cameras.json` + `sensors.json` + `project.json`), sluit de bijbehorende camera’s, arrays en lichtsensoren in één keer aan en stuur opnames aan met benoemde handles.

Vlakken 1 en 2 **starten automatisch een lokale backend** als er nog geen actief is (hetzelfde meegeleverde binaire bestand dat de GUI en CLI starten) — zodat een eenvoudig script werkt vanuit een nieuwe shell zonder dat je eerst een backend hoeft te starten: `ChlorosLocal` (en `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` en `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` en `analyze_array_network()` starten nooit zelfstandig op: ze hebben een draaiende backend nodig. Geef `auto_start_backend=False` door om dit uit te schakelen (bijvoorbeeld wanneer je naar een backend op afstand verwijst, die nooit wordt gestart). Zie [Backend automatisch starten](#backend-auto-start). Surface 3 gedraagt zich anders: `open_project()` accepteert geen `auto_start_backend`-parameter, en `connect_all()` start nooit een backend op — het probeert `http://127.0.0.1:5000` één keer en, als er niets reageert, valt het terug op directe (backend-vrije) `lattice_sdk`-apparaatbesturing. Daarvoor zijn de Arena SDK-runtime en OpenCV (`cv2`) moet in je Python kunnen worden geïmporteerd, wat het SDK-wheel niet installeert; zonder deze worden elke opgeslagen camera en array als een fout gemeld, met vermelding van de reden. De lichtsensoren van een project hebben geen backend-vrij pad: als er geen backend draait, wordt elke opgeslagen sensor als een fout gemeld, met vermelding van de reden (zie [`SensorHandle`](#sensorhandle)). Alleen `proj.process()` en `stream(..., overlays=True)` construeren op een 'lazy' manier een `ChlorosLocal()` (die automatisch start).

Alle drie zijn ze auth-gated: voer `chloros-cli login` één keer uit op de machine, of meld je aan via de desktop-GUI. Wat een aanroep zonder geldige sessie veroorzaakt, hangt af van de interface (zie [Uitzonderingen](#exceptions)): `ChlorosAuthenticationError` vanuit `ChlorosLocal` (en `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` van de smart-connect-aanroepen en hun sessies, `status: "error"` met een `error_code` van `analyze_array_network()`, en een vermelding onder `errors` in het `connect_all()`-rapport.

Vereisten:
- Python 3.7+ (zoals aangegeven door het pakket; ontwikkeld/getest op 3.10)
- Chloros Desktop lokaal geïnstalleerd (het backend-uitvoeringsbestand zit in het installatieprogramma)
- Actieve Chloros+-login op **elk betaald Chloros+-abonnement (Copper of hoger)**; het gratis**Iron**-abonnement biedt geen toegang tot SDK/CLI. Dit wordt**aan de serverkant**afgedwongen-zijde**: elk verzoek met SDK- of CLI-vlag moet zowel een actieve sessie als een betaald abonnement bevatten. Een abonnement lager dan Copper krijgt de foutcode `403` met `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` van `ChlorosLocal`, `ChlorosConnectError` bij smart-connect-aanroepen). Een uitgelogde beller krijgt in plaats daarvan `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` van `ChlorosLocal`, `ChlorosConnectError` van de smart-connect-oproepen) — de twee zijn verschillend omdat het opnieuw uitvoeren van `chloros-cli login` de fout `401` verhelpt, maar de fout `403` niet kan verhelpen; hiervoor is een upgrade van het abonnement nodig.
- Offline gebruik wordt ondersteund binnen de respijtperiode van het abonnement: het abonnementsniveau wordt gelezen uit de cache voor servervalidatie (5 min) of de cache voor ondertekende, aan de machine gebonden licenties (30 dagen voor maandabonnementen, tot het verstrijken van het jaarabonnement). Wanneer die respijtperiode verstrijkt, wordt het abonnement omgezet naar de gratis versie en wordt de toegang tot de SDK en CLI geblokkeerd totdat de machine één keer verbinding kan maken met de server. De foutcode `chloros-cli status` (`GET /api/license-status`) blijft zichtbaar in de gratis versie, zodat de oorzaak duidelijk is.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS of nieuwer**, of Jetson (JetPack 6). Ubuntu 20.04 wordt**niet** ondersteund: de afhankelijkheden van `.deb` zijn afgeleid van de bibliotheken waaraan de backend is gekoppeld, waaronder `libc6 (>= 2.35)`, en Focal wordt geleverd met glibc 2.31.

---

## Installatie

De Python SDK is een dunne Python-laag bovenop de Chloros-backend. Tenzij je een backend op een andere machine aanstuurt (zie [Remote-Backend-modus](#remote-backend-mode-pip-only-host-via-tunnel)), moet je het **Chloros-desktop-pakket lokaal geïnstalleerd hebben** (Windows-installatieprogramma of Linux `.deb`), inclusief DAQ-lichtsensoren. Het pakket bevat het backend-uitvoeringsbestand, de Arena SDK-runtime voor LATTICE-camera’s en de kalibratiebundels.

Nieuwste downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Stap 1 — Installeer het Chloros-platformpakket

#### Windows (.exe)

1. Download `Chloros Installer 1.3.0.exe` vanaf de downloadpagina.
2. Voer het installatieprogramma uit en volg de wizard. Het standaardinstallatiepad is `C:\Program Files\Chloros\`.
3. Start Chloros minstens één keer op en log in met je Chloros+-account.

Het installatieprogramma zorgt ervoor dat de backend (`chloros-backend.exe`, niets anders) door de Windows-firewall in elk netwerkprofiel: UDP-inkomend verkeer, voor camerastreams, PTP, mDNS en de DAQ-E-stream, en TCP-inkomend verkeer uitsluitend vanuit het lokale subnet, voor de verbinding die een DAQ-E tijdens een firmware-update terug naar de computer maakt ([`update_firmware()`](#daq-e-device-settings)). Het verwijderprogramma verwijdert beide regels. De HTTP-API van de backend blijft in beide gevallen op `127.0.0.1` staan.

De groep **Systeeminstellingen** van de wizard bevat twee opties, die beide standaard zijn aangevinkt:

| Optie | Wat het doet | Afzien van stille installatie |
| --- | --- | --- |
| **Windows-lange paden inschakelen** | Hiermee kunnen projectmappen dieper genest zijn dan de padlimiet van 260 tekens in Windows. Als lange paden zijn uitgeschakeld (de standaardinstelling van Windows), wordt een projectmap die te diep is voor de bijbehorende producten geweigerd voordat er iets wordt verwerkt — zie [Probleemoplossing](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **De Chloros-backend uitsluiten van Microsoft Defender-scans** | Voorkomt dat Defender een real-time-tijdsscan van elk bestand dat door een camera-opname wordt geschreven. Bij hoge opnamesnelheden kost de scan ongeveer 30 % van de snelheid: gemeten bij ROI 512 op een opstelling met 9 camera’s, liep de continue raw-opname op 52,7 opnames/s met scannen en 77,7 opnames/s wanneer de backend werd uitgesloten. | `/MERGETASKS="!defenderexclusion"` |

Een continue opname rapporteert de belasting van Defender als `defender_cores` (zie [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Installeer met `apt install ./…`, niet met `dpkg -i`: apt installeert ook de vereisten van de Python-SDK, die het pakket aanbeveelt in plaats van vereist (zie [Stap 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Voor Jetson Orin / Orin NX / Orin Nano op JetPack 6 (L4T R36). De PyTorch-versie is de CUDA-build van JetPack 6, dus GPU-verwerking vindt plaats op de GPU van de Orin; het maakt gebruik van de CUDA- en cuDNN-bibliotheken die JetPack 6 installeert (`sudo apt install nvidia-jetpack` op een image waar deze ontbreken).

#### Linux: ontvangstbuffers en de backend-service

Beide .deb-pakketten vergroten de socket-ontvangstbuffers van de kernel die nodig zijn voor LATTICE-camerastreams tot 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Bij de standaardinstelling van de kernel worden camerapakketten in de kernel weggegooid en verliest een array frames en framerate; `chloros-cli lattice network` meldt wanneer de buffers bijna leeg zijn. Zie de CLI-referentie [Linux-hosts — socket-ontvangstbuffers](cli-reference.md#linux-hosts--socket-receive-buffers).

Op een machine zonder grafische interface moet je de backend als een service **onder jouw gebruikersaccount** uitvoeren, zodat een script er een kan vinden die actief is (`connect_all()` start nooit een backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

De backend draait dan als u: de opnames die hij schrijft zijn van u, en hij gebruikt uw `chloros-cli login`. De root-`chloros-backend.service` is nog steeds geïnstalleerd voor machines waarop deze is ingeschakeld, maar elk bestand dat een root-backend schrijft, is eigendom van root. Zie de CLI-referentie [De backend als service uitvoeren](cli-reference.md#running-the-backend-as-a-service-headless).

Een backend die door de service wordt uitgevoerd, is gedeeld en wordt nooit door een client gestopt: SDK-scripts en `chloros-cli` gebruiken deze zoals ze hem aantreffen (`ChlorosLocal.shutdown_backend()` stopt alleen een backend die `ChlorosLocal` zelf heeft gestart), en `GET /api/status` noemt de unit `service`. Start het opnieuw op met `sudo systemctl restart chloros-backend@$USER.service`. Het upgraden van het pakket stopt de backend-services tijdens de installatie, waardoor een actief script de verbinding met zijn camera’s en sensoren verliest en vervolgens alle ingeschakelde apparaten opnieuw opstart; het verwijderen van het pakket stopt ze en schakelt de ingeschakelde apparaten uit. Er draait slechts één backend tegelijk, dus schakel één unit in: een tweede weigert te starten en blijft `failed` (uitstapstatus 98, niet opnieuw opgestart), en de status `systemctl status` geeft aan waarom. Om van unit te wisselen, schakel je de unit uit die je niet gebruikt.</id></sn>

### Stap 2 — Installeer de Python-SDK

**Het Chloros-installatieprogramma bevat een bijbehorend SDK-wheel.** Elk Windows-installatieprogramma en elk Linux-.deb-pakket plaatst een `chloros_sdk-X.Y.Z-py3-none-any.whl` op de schijf dat exact overeenkomt met de GUI-/CLI-/backend-versie. Je hoeft niet op PyPI te letten om up-to-date te blijven.

#### Windows

Het installatieprogramma voert `pip install` automatisch uit met het meegeleverde wheel, waarbij gebruik wordt gemaakt van de Python-versie van je systeem (bij voorkeur de `py.exe`-launcher, anders wordt teruggevallen op `python -m pip`). U hoeft niets te doen — `import chloros_sdk` werkt in uw Python-omgeving na een succesvolle installatie. Als er geen Python op de computer staat, slaat het installatieprogramma deze stap stilzwijgend over en blijven de GUI en CLI gewoon werken.

#### Linux (.deb)

Beide .deb-pakketten (x86_64 en Jetson) installeren de SDK voor het systeem `python3`, zodat `import chloros_sdk` na de installatie werkt, zonder pip en zonder netwerk. De pakketten (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` en de projectloaders) worden ondergebracht in `/usr/lib/python3/dist-packages`, en het `chloros-mcp`-commando van de SDK (de [MCP-server](#mcp-server-ai-assistants)) in `/usr/bin`, die allemaal eigendom zijn van het pakket: bij een upgrade worden ze vervangen en bij het verwijderen van het pakket worden ze verwijderd.

De eigen vereisten van de SDK, `python3-numpy` en `python3-requests`, zijn **aanbevolen, niet verplicht** (de GUI, CLI en backend zijn op zichzelf staand): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` op een Jetson) installeert ze, `dpkg -i` doet dat niet. De installatie-uitvoer geeft de SDK-versie weer die `python3` importeert, eventuele ontbrekende vereisten, en een `chloros_sdk` elders in het pad van `python3` die de kopie van het pakket overschrijft (zoals een exemplaar dat eerder met `pip install --user` is geïnstalleerd).

In een virtualenv, die de pakketten van het systeem niet ziet, installeer je het meegeleverde wheel daarin:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Voor Jetson-implementaties in een air-gapped omgeving gebeurt dit volledig offline — de SDK en het wheel staan al op de schijf.

#### Openbare PyPI

Voor hosts die alleen pip gebruiken (waar geen Chloros-desktoppakket is geïnstalleerd; de SDK stuurt dan een backend op een andere machine aan, zie [Remote-Backend-modus](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI bevat de nieuwste stabiele release. Zorg ervoor dat de SDK-versie overeenkomt met die van de Chloros-backend die erdoor wordt aangestuurd (zie [Versiebeheer en compatibiliteit](#versioning--compatibility)). De DAQ-M-helper `DLS` heeft ook `bleak` nodig: `pip install "chloros-sdk[camera]"` (of `pip install bleak`).

#### Controleer

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+-abonnement vereist.** Voor alle SDK-aanroepen is een actieve Chloros+-login vereist. Voer `chloros-cli login user@example.com 'YourPassword'` één keer per machine uit; de inloggegevens worden opgeslagen in `~/.chloros/`.

### Heb ik het Desktop-pakket nodig?

Het pip-pakket alleen is voor de meeste workflows **niet** voldoende. Dit is wat elk SDK-oppervlak nodig heeft:

| SDK-oppervlak | Is het Desktop-pakket nodig? | Waarom |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja**(om er een te starten)**/ Nee** (een backend die al bereikbaar is) | Gebruik een actieve backend (die van de desktop-app of een ander gestart script); anders wordt de geïnstalleerde automatisch gestart: `/usr/lib/chloros/chloros-backend` (Linux) of `<install folder>\backend\chloros-backend.exe` (voor Windows; standaard `C:\Program Files\Chloros\backend\`, een aangepaste installatiemap wordt uit het installatiebestand gelezen). Geef `backend_exe=` door om het zelf een naam te geven. |
| `chloros-mcp` ([MCP-server](#mcp-server-ai-assistants)) | **Ja**(om er een te starten)**/ Nee** (er is al een backend bereikbaar) | Gebaseerd op `ChlorosLocal`, dus hetzelfde als de rij hierboven. |
| `read_image_audit_tags` | **Nee** | Leest de eigen metagegevens van de bewerkte afbeelding (alleen Python-standaardbibliotheek). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokaal)**/ Nee**(op afstand) | Pure HTTP-clients via de backend. Lokale backend → desktop-pakket vereist. Backend op afstand → `backend_url=`**via een tunnel** (zie Remote-Backend Mode — meegeleverde backends binden alleen aan loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Opgeslagen projecten via de backend. |
| Directe LATTICE-klassen (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Vereist de Arena SDK-runtime, die in het desktop-pakket is meegeleverd, en OpenCV (`cv2`) in je Python-omgeving, die noch door het desktop-pakket, noch door de SDK wordt geïnstalleerd (`pip install opencv-python`). `CAMERA_AVAILABLE` is anders bij het importeren gelijk aan `False`. `DLS` heeft ook `bleak` nodig. |

### Remote-Backend-modus (alleen pip-host, via tunnel)

> **De geïnstalleerde backend is niet bereikbaar via het LAN.** Deze luistert
> alleen op de loopback (`127.0.0.1` / `::1`), dus `backend_url="http://<lan-ip>:5000"`
> **kan geen verbinding maken met een geïnstalleerde Chloros**. Om een backend op een andere
> machine aan te sturen, moet je de loopback-poort zelf doorsturen en de SDK naar de
> tunnel laten verwijzen:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Headless-/CI-/robotica-hosts kunnen één machine met de volledige desktopinstallatie als „Chloros-server“ gebruiken en `pip install chloros-sdk` op alle andere locaties — maar de verbinding daartussen is de hierboven beschreven, door de gebruiker ingestelde tunnel, niet een directe LAN-URL.

> **`ChlorosLocal` op een host die alleen pip ondersteunt.** `ChlorosLocal(backend_url=BACKEND)` maakt verbinding met een backend die het kan bereiken zonder desktopinstallatie: het zoekt alleen naar een lokaal backend-binair bestand wanneer het er een moet STARTEN (geen reactie en `auto_start_backend=True`). Tot en met versie 1.2.0 zocht het eerst naar het binaire bestand en genereerde het de foutmelding `ChlorosBackendError` („Chloros-backend niet gevonden…“) op een host met alleen pip, zelfs als er een backend bereikbaar was. De mappen die je doorgeeft (`process_folder`, `import_images`) worden op deze machine gecontroleerd en door de backend gelezen; bij een backend op afstand moeten ze dus op beide systemen naar dezelfde map verwijzen, zoals een gedeeld station.

### Werkstroom uitsluitend voor DAQ (host met alleen pip)

`pip install chloros-sdk` kan op zichzelf geen DAQ-sensor aansturen: een DAQ-sensor wordt altijd aangestuurd via een Chloros-backend met `connect_daq_sensor()`:

- **Sensor op deze machine:** installeer hier het Chloros-desktop pakket. `connect_daq_sensor()` start zijn backend als er nog geen actief is.
- **Sensor op een andere machine:** die machine heeft het desktop-pakket nodig; bereik de backend via de tunnel in [Remote-Backend-modus](#remote-backend-mode-pip-only-host-via-tunnel). De backend opent de sensor op zijn eigen machine, dus een DAQ-U is aangesloten op die machine, een DAQ-M bevindt zich binnen het Bluetooth-bereik ervan en een DAQ-E bevindt zich op een netwerk dat de backend kan bereiken.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Hoe dan ook dwingt de backend het Chloros+-abonnement af: log in op de machine waarop de backend draait (`chloros-cli login` of de desktop-app) met een betaald abonnement (Copper of hoger). Een backend waarbij je bent uitgelogd en een abonnement lager dan Copper genereren beide een `ChlorosConnectError` met de reden van de backend (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Snelstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## API-index op het hoogste niveau

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Beeldverwerking — `ChlorosLocal`

De hoofdklasse van de pijplijn. Start de backend bij het aanmaken (als er geen reageert en `auto_start_backend=True`), maakt projecten aan en configureert deze, houdt de voortgang bij en retourneert samenvattingen na afloop.

### Constructor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Methoden

| Methode | Beschrijving |
| --- | --- |
| `create_project(project_name, camera=None)` | Een nieuw project aanmaken (optioneel met een camerasjabloon zoals `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Open een opgeslagen project (de map ervan of de naam ervan in de map ‘projects’) en maak dit tot het geopende project. De paden van een verplaatst project worden opnieuw gericht naar de map waarin het zich nu bevindt; `relink` geeft de naam van de map waarin de overige ontbrekende bestanden zich nu bevinden. Geeft het antwoord van de backend terug met `source_files`; genereert een `ChlorosProjectFilesMissingError`-foutmelding wanneer er niets kan worden gevonden. Zie **Een verplaatst project** hieronder. |
| `import_images(folder_path, recursive=False)` | Importeer RAW/TIF/JPG/DNG-afbeeldingen **en `.daq` lichtsensoropnames**. Geeft `count` (afbeeldingen) en `scan_count` (opnames). Geeft alleen een waarschuwing als de map geen van beide bevat. |
| `export_light_sensor(daq=True, csv=True)` | Schrijf gekalibreerde `.daq` + `.csv` voor elke lichtsensoropname in het project, in `<project>/Light Sensor/`. Zie [Lichtsensoropnames](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Stel de verwerkingsregelaars in. `indices` zijn vooringestelde namen (`NDVI`, `GNDVI`, `OSAVI`, …), die elk aan het project zijn toegevoegd voor de banden van de bijbehorende camera; een naam die geen voorinstelling is, of die de banden niet kunnen vormen (`NDRE` heeft RE en NIR nodig, die geen enkele camera afzonderlijk biedt), krijgt een naam in de vorm `UserWarning` en geen indexafbeelding. `daq_log_path` verwijst naar een opname van een lichtsensor (een `.daq`- of `.csv`-bestand, of een map met dergelijke bestanden) waarmee elk frame wordt vergeleken, zowel in LATTICE als in Survey3, in plaats van de opnames die samen met de beelden zijn geïmporteerd; wordt aangegeven dat een pad ontbreekt of geen opname bevat, en worden de geïmporteerde opnames gebruikt. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Voer de pijplijn uit. Geeft `{"status": "complete", "async": False}` terug, plus een `summary`-sleutel wanneer de backend er een levert — zie [Samenvatting en tips na uitvoering](#post-run-summary--hints) — `export_check_failure` wanneer de exportgegevens van de run niet konden worden geverifieerd, en `image_failure` wanneer er een afbeelding verloren is gegaan. `wait=False` retourneert `{"status": "started", "async": True}` in één keer. |
| `get_config()` / `get_status()` / `status()` | Controleer de status van de backend. |
| `logout()` | Uitloggen: wist de in de cache opgeslagen inloggegevens en maakt de apparaatslot van deze machine vrij (`device_deregistration` in het resultaat). Een lopende verwerkingsrun wordt niet gestopt: deze wordt voltooid in de modus waarin deze is gestart (`run_in_progress: true` en `run_mode` in het resultaat), en de volgende run maakt gebruik van standaardverwerking. |
| `shutdown_backend()` | Stop de backend als deze instantie deze heeft gestart; een backend die al actief is, wordt met rust gelaten. Het achterlaten van een `with`-blok of het ophalen van de instantie door de garbage collector leidt niet tot het stoppen ervan: de backend blijft draaien voor het volgende script en stopt zichzelf na 30 minuten als er niets gebruik van maakt. De backend wordt gevraagd om af te sluiten, dus geeft hij eerst zijn camera’s en lichtsensoren vrij en rondt hij eventuele opnames af; hij wordt pas beëindigd als hij 10 seconden later nog steeds actief is. |
| `discover_cameras()` | LATTICE-camera’s detecteren **via de backend van deze instantie** (`/api/camera/discover`). Geeft een lijst met woordenboeken terug (`serial`, `model`, `ip`, …) — dezelfde structuur als die de GUI en CLI zien. Lege lijst als er geen worden gevonden of als de backend onbereikbaar is. |
| `camera_capture(output_dir, format="tiff", **settings)` | Leg één enkel frame vast**via de backend**(automatisch gestart door deze handle), zodat het dezelfde voorbereiding krijgt als de GUI/CLI (standaard 12-bit, hergebruik van de pool, ingebedde kalibratiemetadata). Bepaal het doel met `serial=` of `device_index=`; geef `exposure`/`gain`/`pixel_format`/`preset` door als `**settings`. Geeft het verouderde metadata-woordenboek terug (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Levert samengestelde preview-frames met overlay van een gepoolde camera — dunne MJPEG-client via de `/api/camera/<serial>/stream-annotated`-route van de backend (zebra / raster / dradenkruis / histogram / peaking / spot getekend aan de serverzijde). `decode=True` levert BGR-arrays op en vereist OpenCV (`cv2`), dat de SDK niet installeert; `False` levert onbewerkte JPEG-bytes op. Ook per project bereikbaar als `ChlorosProject.stream(overlays=True)`. |

**Een lopende run behoudt zijn project.** Terwijl een verwerkingsrun bezig is — die van dit script, van de app, van `chloros-cli process` of van `chloros-mcp` — weigert de backend een project aan te maken, te openen, erin te importeren of de verwerkingsinstellingen ervan te wijzigen (HTTP-status 409 `PROCESSING_IN_PROGRESS`): `create_project()` en `import_images()` genereren een `ChlorosProcessingError`, en `configure()` genereert `ChlorosConfigurationError`, en `process()` met een andere modus dan de huidige genereert `ChlorosProcessingError` (een run behoudt de modus waarin deze is gestart), elk met het bericht van de backend. Een opname die in het project is opgeslagen, wordt vervolgens aan het project toegevoegd wanneer de run eindigt (dat blijkt uit het antwoord `project_add`). Wacht tot de run is voltooid (`get_status()`) en ga dan verder.**Een verplaatst project.** De `project.json` van een project benoemt elk bronbestand (elke afbeelding en elke opname van de lichtsensor) aan de hand van het volledige pad, dus als je de projectmap – of de hele map met projecten – naar een ander station verplaatst, blijven ze de oude locatie vermelden. Elke keer dat je het project opent, wordt automatisch gecorrigeerd wat mogelijk is: er wordt gezocht naar een bestand dat zich in de projectmap bevond in de map waar het project zich nu bevindt, en elk pad dat daar wordt gevonden, wordt aangepast en opgeslagen (`project.json` wordt eerst opgeslagen als `project.json.before-relink-<date>-<time>`). Een pad wordt alleen aangepast naar een bestand dat daadwerkelijk aanwezig is. Bestanden die zich buiten de projectmap bevonden – een hub-sessie, een import via ‘Map toevoegen’ – worden alleen gevonden met `relink=<the folder they are in now>` (of een map daarboven). `open_project()` levert `source_files` op: `total`, `found`, `missing`, `missing_roots` (de mappen waarin de ontbrekende bestanden zich bevonden), `missing_outside_project` en `relink` (wat er opnieuw is gekoppeld, van waar naar waar). Een project waarin enkele bestanden ontbreken, wordt samen met de rest geopend; een project waarin geen ontbrekende bestanden zijn gevonden, wordt niet geopend, en `open_project()` genereert foutmelding `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). De CLI-fouten `project open --relink` en `chloros-mcp`’`open_project` (`relink_folder`) van de CLI doen hetzelfde.

Gebruik als contextmanager voor gegarandeerde opschoning:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Opnames van lichtsensoren — gekalibreerd `.daq` + `.csv`

Een DAQ-U / DAQ-M / DAQ-E kan worden opgenomen **zonder** de bijbehorende kalibratiebundel. Dat is
wat de openbare [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
recorders (`record_daq.py`) standaard doen: ze schrijven ruwe sensortellingen weg en voorzien het
bestand van een tijdstempel, zodat Chloros eerst de fabriekskalibratie van die sensor **op basis van het serienummer** ophaalt — eerst uit de lokale cache
eerst, daarna de MAPIR-cloud — en past deze toe bij het importeren.

Chloros schrijft het resultaat terug als twee producten per opname, onder
`<project>/Light Sensor/`:

| Product | Wat het is |
| --- | --- |
| `<name>_calibrated.daq` | Het opnieuw te verwerken archief — hetzelfde schema als een live-opname, maar nu met vermelding van de bundel die het heeft geproduceerd. Bij het opnieuw importeren wordt het **niet** nogmaals gekalibreerd. |
| `<name>_calibrated.csv` | Spectrale stralingsintensiteit in W/m²/nm op het eigen golflengteraster van de sensor, één rij per meting, plus fotometrische kolommen (totaal vermogen, fotopische/scotopische lux, PPFD en de verdeling daarvan in blauw/groen/rood, piekgolflengte). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Alleen sensoren zonder bundel (DAQ-A).** Ruwe spectrale sensortellingen — *geen* stralingsintensiteit. Zie hieronder. |

`<name>` is de bestandsnaam van de opname zelf. Twee opnames in verschillende mappen kunnen
één — per opname gemeten waarde (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) die in
dezelfde milliseconde door twee camera’s is vastgelegd — en elk krijgt nog steeds zijn eigen producten: de eerste behoudt
`<name>`, de volgende voegt de mappen toe die het onderscheiden **na** de naam
(`<name>__<capture>_daq_calibrated.daq`), zodat de datum en tijd van de opname nog steeds voorop staan.

`process()` voert deze export uit als een van zijn fasen. Hiervoor zijn **geen** beelden nodig:
een op zichzelf gevlogen lichtsensor is een volwaardige workflow, en een dergelijk project bevat per definitie geen
beelden.

**DAQ-A-opnames worden geëxporteerd als ruwe tellingen.** De DAQ-A-serie dateert van vóór het
bundelsysteem per serienummer en heeft geen bundel om op te halen — het wordt in het veld gekalibreerd tegen een
reflectiedoel, en daarom was er nooit een nodig. Die opnames worden geëxporteerd
onder een `_raw`-stam in plaats van `_calibrated`: een andere bestandsnaam in plaats van een vlag
in het bestand, omdat de claim het versturen per e-mail als kale naam moet overleven. De
`.csv`-header vermeldt `raw spectral sensor counts (NOT irradiance)` en waarschuwt dat de
waarden **binnen** het bestand vergelijkbaar zijn — precies waarvoor kalibratie op basis van doelen
— en niet tussen sensoren. De vermogensafhankelijke fotometrische kolommen (totaal vermogen,
fotopische/scotopische lux, PPFD) worden als **NULL** weergegeven in plaats van geïntegreerd op basis van tellingen.

Een DAQ-U / DAQ-M / DAQ-E waarvan de bundel simpelweg niet kon worden opgehaald, wordt nog steeds **overgeslagen**,
en niet in ruwe vorm weggeschreven: in dat geval bestaat de bundel wel en is „opnieuw verbinden en opnieuw verwerken“ een reëel advies.

Oudere **v1.01 / v1.02**-opnames (een DAQ-A-SD schrijft deze) bevatten geen tijdstip per meting,
alleen het schrijftijdstip van het bestand. De image↔downwelling-matcher weigert ze nog steeds — het matchen van een
frame aan een schrijftijd zou onzichtbare fouten opleveren — maar de exporteur leest ze wel, en het
 CSV-bestand geeft `clock=daq_created_on` weer, zodat het product aangeeft op welke klok het staat.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Een opname waarvan de kalibratiebundel niet kan worden opgehaald (offline, of een sensor zonder
kalibratie in het bestand) wordt gerapporteerd onder `skipped` **met de reden**. Het wordt nooit
uitgeschreven als een „gekalibreerd“ bestand met ruwe tellingen — maak verbinding met het internet en
voer de bewerking opnieuw uit, waarna de export wordt voltooid.

### Voortgangscallbacks

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Samenvatting en tips na afloop

Na voltooiing haalt `process()` `GET /api/processing-summary` op en voegt de inhoud toe als `result["summary"]`. Het ophalen gebeurt op basis van ‘best effort’ en blokkeert nooit een succesvolle terugkeer — als het overzicht niet beschikbaar is, valt `process()` terug op de eenvoudige `{"status": "complete", "async": False}`-vorm. Elk item in `summary["hints"]` — volledige zinnen met de voorgestelde oplossing, bijvoorbeeld waarom een run geen uitvoer opleverde — wordt ook opnieuw uitgegeven als een Python-`UserWarning`, zodat runs zonder uitvoer zichzelf diagnosticeren, zelfs als je het dict nooit inspecteert:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` is het machinaal leesbare deel:

| Sleutel | Wat het telt |
| --- | --- |
| `models` | Cameragroepen in de run. |
| `images_in_groups` | Bronbeelden in die groepen. |
| `targets_found` | Gedetecteerde reflectantiedoelen. |
| `images_calibrated` | Afbeeldingen waarvoor de run een reflectantieproduct heeft gegenereerd. |
| `images_failed` | Afbeeldingen die zijn mislukt en niet zijn geëxporteerd. |
| `exported_files` | **Afbeeldingsproductbestanden die de run heeft gegenereerd**: elk product (debayered, preview, radiance, reflectance, index, target renders), in elk formaat. |
| `reflectance_files` | De reflectantieproducten daaronder (`Reflectance_Calibrated_Images`), ongeacht of deze zijn gekalibreerd door een doel of door de stralingsintensiteit van de lichtsensor (DAQ). Een run zonder doel die toch reflectantie heeft opgeslagen, vermeldt dit in een hint; alleen bij een run die niets heeft opgeslagen, wordt gemeld dat de reflectantie-uitvoer ontbreekt. |
| `clamp_masks_written` | De [unmix-clamp-maskers](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) die ernaast staan vermeld — zijn aanvullende gegevens, geen resultaten, en staan dus niet in `exported_files` (de CLI-opdracht telt ze op dezelfde manier apart). |
| `captures_left_out` | Ruwe frames in de map `captures` van het project die deze run niet heeft vastgelegd. Een run wacht bij het starten op wat ervoor is vastgelegd, dus dit is 0 tenzij er tijdens de run een opname is opgeslagen of de wachttijd is verstreken. `summary["captures_left_out_note"]` geeft aan wat ermee moet gebeuren. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Opnames van de lichtsensor, bewust apart geteld — ze zijn afkomstig uit een andere fase en bestaan ook voor runs zonder beeldmateriaal; als ze zouden worden meegeteld, zou een run die alleen uit DAQ-gegevens bestaat eruitzien alsof er beelden zijn geëxporteerd. |

Daarnaast: `summary["output_dirs"]` (elke map waarnaar is geschreven),
`summary["light_sensor_export"]`, `summary["stopped"]` (waar wanneer de gebruiker de
run heeft onderbroken, zodat gedeeltelijke tellingen niet worden geïnterpreteerd als een voltooide run die te weinig heeft opgeleverd), en
`summary["groups"]` (de uitsplitsing per groep).

`exported_files` wordt door de pijplijn **tijdens het schrijven** geregistreerd, en niet achteraf uit de
beeldobjecten van het project gescand. De parallelle en GPU-strategieën bouwen hun eigen beeldobjecten
(in werkersubprocessen voor de GPU-paden), dus de oude scan rapporteerde
`0 file(s) written` voor elke dergelijke run en gaf vervolgens de ‘zero-exports’-hint af — bij runs
waarbij alles had gewerkt. Als je een script schrijft op basis van dit nummer, rapporteert een succesvolle parallelle run nu
een telling die niet nul is.

Overslagen door de lichtsensor rapporteren de reden die de lezer daadwerkelijk voor elk bestand heeft vastgesteld — een
onleesbaar schema, een ontbrekende bundel, een schrijffout — **gededupliceerd**, zodat twintig bestanden
die om één reden zijn overgeslagen, als één reden worden weergegeven in plaats van twintig herhalingen ervan.

> **`process()` wordt niet gegenereerd wanneer een run geen afbeeldingen oplevert.** Dit is een van de twee punten waarop de SDK en
> de CLI opzettelijk van elkaar verschillen: `chloros-cli process` beschouwt „producten werden aangevraagd, maar er werden geen
> geschreven” als een fout en stopt met een waarde anders dan nul, terwijl de SDK normaal terugkeert en de
> toestand via `summary` / hints. Als je pijplijn moet stoppen bij een lege run, controleer dit dan
> zelf — inspecteer `summary` (of tel de bestanden in de projectmap) in plaats van te vertrouwen op
> het ontbreken van een uitzondering. De gebruikelijke oorzaken zijn een invoermap die niet werd herkend als een
> opname en producten die werden overgeslagen omdat ze niet van toepassing waren op de aanwezige camera’s (bijv. straling van camera’s die alleen RGB
> ondersteunen).
>
> `process()` wordt ook niet geactiveerd wanneer de export van een run niet kon worden geverifieerd: er wordt een waarschuwing gegeven en
> `result["export_check_failure"]` wordt ingesteld. De producten worden wel weggeschreven. Om de run hierop te stoppen, moet je die sleutel controleren. Een run waarbij
> een afbeelding ontbreekt, gedraagt zich hetzelfde: er wordt een waarschuwing gegeven en `result["image_failure"]` wordt ingesteld, waardoor `chloros-cli process`
> met een waarde anders dan nul afsluit.

### Handige functies

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Elke run draait binnen zijn eigen `with ChlorosLocal(...)`; een backend die moest worden gestart, blijft draaien wanneer de run terugkeert, voor de volgende aanroep, en stopt zichzelf na 30 minuten als er niets gebruik van maakt. Geen van beide accepteert `auto_start_backend`, `radiometric_output` of `daq_log_path`; gebruik daarvoor `ChlorosLocal` en `configure()` voor die gevallen. `process_lattice_capture()` accepteert geen `camera`, `debayer` of `ppk`: het slaat de paneeldetectie over en gebruikt de standaard debayer.

### Ondersteunde waarden

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrische uitvoer (LATTICE multispectrale pijplijn)

Het LATTICE multispectrale (M3C/M3M) exportniveau van de `process`-pijplijn — `reflectance` (standaard), `radiance`, `sensor-response` of `all` (elke toepasselijke modus per afbeelding) — bepaalt de **"Export radiance"**en**"Export reflectance"****: `radiance` exporteert alleen radiance, `reflectance` alleen reflectance, `sensor-response` geen van beide (alleen de debayered- en preview-producten), `all` beide. De debayered- en preview-producten kunnen op dit niveau niet worden gewijzigd. `configure()` heeft hiervoor een speciaal trefwoord:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

De geavanceerde noodoplossing — het schrijven van de producten via `custom_settings` — werkt nog steeds, maar onthoud dat dit het gehele instellingenblok vervangt (zie de waarschuwing hieronder). Schrijf de `"Export <product>"`-sleutels: de eigen instellingen van een project (`"Export radiance"` / `"Export reflectance"`-instellingen hebben voorrang op de oudere `"Radiometric output"`-sleutel, zodat die sleutel op zichzelf niets verandert.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (de standaardinstelling) deelt de straling van de camera door de **aan de tijdstempel gekoppelde DAQ-neerwaartse straling**, die automatisch wordt afgeleid uit een opgenomen `.daq` (DAQ-U/M/E)**of een DAQ-M-specifieke `.csv`**die bij de beelden is aangetroffen; eventuele lokaal ontbrekende kalibratiebundels per camera of DAQ worden bij het eerste gebruik**automatisch opgehaald uit AWS**. De CLI-interface stelt dit beschikbaar volgenstype productschakelaars op `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Van wie is deze meting?**Een `.daq` die een opname opslaat, is de meting van de camera('s) die deze beschrijft — genoemd in een `<file>.daq.cameras.json`-sidecar ernaast (enkele, gesynchroniseerde en array-opnames; bij een onbewerkte burst wordt de opname van elk onderdeel in het manifest vermeld) — en wordt**alleen gekoppeld aan beelden van die camera’s**. Een op zichzelf staande camera zonder eigen opname maakt daarom geen gebruik meer van de DAQ van een andere camera; koppel er een DAQ aan, of maak er een opname mee. Een eigen opname van een DAQ (`chloros-cli daq pool-record`, of een opname die in de GUI is gestart), of elke `.daq` zonder dat sidecar-bestand (inclusief oudere opnames), is nog steeds bruikbaar voor elke camera.**OCN-camera’s in zonlicht.** Elk OCN-kalibratiepakket wordt samengesteld onder smalbandlicht, met één LED per band, terwijl zonlicht de banden mengt. Chloros 1.3.0 past na het ontmengen van de banden een algemene correctie op basis van zonlicht toe, wat de OCN-producten voor straling, reflectie en index in zonlicht verbetert. Deze correctie wordt toegepast overal waar de ontmenging plaatsvindt — `process()`, opnames, `grab()` / `frame_stream()` en de liveweergave — voor OCN-pakketten die dit ondersteunen; RGN-camera’s blijven ongewijzigd. OCN-beelden die met een eerdere versie zijn vastgelegd, krijgen deze correctie ook wanneer ze worden verwerkt op een computer waarop de kalibratie staat waarmee ze zijn gemaakt (Chloros behoudt de kalibratie van een camera zodra deze is aangesloten); anders worden ze verwerkt zoals voorheen. De OCN-stralings-, reflectie- en indexwaarden verschillen daarom van die van versie 1.2.0.

> `custom_settings` **vervangt** het volledige blok met berekende instellingen (het omzeilt de andere trefwoorden en validatie van `configure()` zoals bedoeld). Wanneer u het gebruikt, neem dan elke `Project Settings`-sleutel op die voor u van belang is, zoals in het bovenstaande voorbeeld.

#### Indexpixels lezen (de NDVI en de andere vegetatie-indexen)

**Een raster met gehele getallen is een contrastuitrekking per afbeelding, geen −1…1-kaart.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` en `JPG (8-bit)` indexproducten — inclusief de 8-bits TIFF-bestanden waarin een 8-bits opname wordt opgeslagen — slaan

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

waarbij `IndexMin` / `IndexMax` **de laagste en hoogste eindige indexwaarden van dat frame** zijn. DN 0 is niet −1, en de volledige schaal is niet +1.

De Chloros-tag markeert beide eindpunten, zodat de afbeelding omkeerbaar blijft:

| Tag | Betekenis |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — de onderstaande conventie |
| `Chloros:IndexMin` | de indexwaarde opgeslagen als DN 0 |
| `Chloros:IndexMax` | de indexwaarde opgeslagen als volledige schaal |

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

Voor het lezen is geen `-config pix4d.config` nodig — exiftool leest deze XMP-eigenschappen zelfstandig. De configuratie is alleen nodig om ze te *schrijven*.

> **`IndexMin == IndexMax`** betekent dat elke pixel van dat frame dezelfde indexwaarde had. Het raster is halverwege de schaal gevuld en er is niets om door te delen — neem het eindpunt.

> **Geen `Chloros:Index*`-tags betekent dat er geen mapping is opgegeven.** Een LUT-export is een ingekleurde weergave waarvan de pixels kleurverlopen zijn die zijn gekoppeld aan je LUT-drempelvenster, en een beeld zonder pixels met een eindige index heeft geen bereik om aan te geven. Ga in geen van beide gevallen uit van een toewijzing — dezelfde regel geldt als bij een ontbrekende `PixelScale`.**Omdat de uitrekking per afbeelding geldt, zijn twee frames van één vlucht niet vergelijkbaar als DN**— elk frame heeft zijn eigen eindpunten, dus dezelfde NDVI komt in verschillende afbeeldingen op verschillende DN-waarden terecht. Voor kwantitatief werk over een vlucht heen (zonale statistieken, vaste drempels, tijdreeksen), exporteer**`TIFF (32-bit, Percent)`**: indexrasters worden daar opgeslagen als float32**indexwaarden zelf**, zonder dat herschaling of tags nodig zijn. Ondanks de naam van het formaat zijn indexproducten niet ×100 — die schaalverdeling geldt alleen voor reflectantie met het gegevenstype integer.

#### Unmix-clampmasker — op welke pixels een band is geklemd

De 3×3 spectrale unmix kan een band negatief maken — wanneer het licht die band nauwelijks bevat, of wanneer de scène buiten het bereik valt dat door de unmix-matrix van de camera wordt beschreven. Dat is geen echte waarde, dus de export zet deze vast op 0 — en de liveweergave toont dezelfde 0. Een band kan op enkele pixels of op bijna alle pixels worden geklemd; het masker registreert beide. `Chloros:Band{R,G,B}UnmixClampFraction` geeft aan op hoeveel van een frame dat is gebeurd; het klemmasker geeft aan **welke pixels**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— in een `clamp_masks/`-map**naast** de map van het product, nooit erin (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — voor elk radiance-, reflectance- en indexproduct waarin ten minste één pixel is geklemd. Enkelbandige `uint8`, op het eigen pixelraster van het product (oriëntatie, ROI, binning en de array-uitlijningsvervorming inbegrepen). Elk masker vermeldt de cameratags van het bijbehorende product (`Make` / `Model` / serienummer), de opnametijd en de lenstags, en geeft een overzicht van de banden waarvoor de bits staan (`XMP-Camera:BandName`, bit 0 eerst); de `ImageDescription` en `Software` markeren het als een clamp-masker, en het bevat geen kalibratie.
- **Bits**: bit 0 = de band in het R-kanaal van het product (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Een ingestelde bit betekent dat de waarde van die pixel in die band is begrensd op 0 — behandel deze, en elke index die hieruit is opgebouwd, als een waarde met een lage betrouwbaarheid.
- **`Chloros:UnmixClampMask`** wordt op elk product aangebracht waarop de unmix is uitgevoerd: het pad van het masker ten opzichte van de eigen map van het product (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, altijd gescheiden door `/`-gescheiden — voeg dit samen met de map van de afbeelding om het masker te openen), of `none` wanneer geen enkele pixel is begrensd. Geen tag betekent dat het product nooit de unmix-bewerking heeft ondergaan (RGB- of monochrome camera’s, raw/debayered niveaus).
- Opnameresultaten bevatten dit als de **`clamp_mask`** van het frame (het pad naar het masker, of `None` wanneer er niets is geklemd) — in de resultaten van `CameraSession.capture()` zijn de `ArraySession.capture()`'de `CaptureResult`-frames en de HTTP-opname-responsen. De offline build vermeldt de maskers als `clamp_masks`.
- De multiband-stack van een live-index (`/api/camera/cli-array-index` met `save_multiband`) krijgt één masker per broncamera, in de eigen `clamp_masks/` van de uitvoermap — zie de CLI-referentie'`lattice index`. Een video heeft geen masker: het resultaat van de offline build rapporteert in plaats daarvan de clamp-fractie van elk frame.

Omdat geen enkele productmap, op welke diepte dan ook, een masker bevat, kan een productmap in zijn geheel — inclusief submappen — aan fotogrammetriesoftware worden doorgegeven, en deze verwerkt alleen afbeeldingen. Geef de productmap door, niet de map erboven. De eigen importfunctie van Chloros slaat de maskers over.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect voor LATTICE-camera’s

Permanente backend-sessies voor live hardware. Dezelfde eindpunten die de GUI gebruikt, dus het gedrag is identiek in SDK, CLI en GUI.

### Enkele camera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

De hier ingestelde belichting, versterking en het doel worden als de jouwe behouden: de automatische belichtingsregeling van Chloros (inclusief de door de DAQ gemeten belichting) laat deze ongemoeid. `cam.set_settings(exposure_auto="Continuous")` (of `gain_auto=`) geeft de belichting (versterking) terug, en `cam.set_settings(target_brightness_auto=True)` geeft de doelwaarde terug aan de automatische doelinstelling van Chloros.

#### `connect_camera()` Handtekening

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Het keert terug zodra de stream is gestart.** De backend wacht, binnen bepaalde grenzen (ongeveer 15 s totdat de stream start, daarna 10 s voor de eerste volledige buffer — niet voor het eerste voorbeeldframe, dat ongeveer 1,5 s later volgt), dus een `status()` direct nadat er `streaming` staat. De `connect_stream` van de sessie is het `stream`-blok van het antwoord: `state` `streaming`, `no_frames` (de stream is gestart en heeft niets geleverd — dat blijkt uit `note` en, voor een camera die met jumbo-pakketten werkt, om opnieuw verbinding te maken met een pakketgrootte van 1500 bytes), `starting` of `start_failed`, met `waited_s`. `None` voor een camera die al was verbonden. Een voorbeeldweergave (`get_camera_preview`, `/api/camera/jpeg/<serial>`) wordt direct na het verzoek aangevraagd; er wordt vervolgens een beperkte tijd (4 s) gewacht op het eerste voorbeeldframe van de camera in plaats van te antwoorden met "Nog geen beeld beschikbaar"; een camera waarvan de stream nog niets heeft geleverd (`no_frames`, of ingesteld op een hardwaretrigger) reageert nog steeds onmiddellijk.
- **De pakketgrootte kun je zelf instellen.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` opent de camera met pakketten van 1500 bytes, zonder dit te testen (`chloros-cli lattice cam-connect --packet-size standard` verzendt hetzelfde).
- **Er kan slechts één verbinding tegelijk in de camerapool**: het openen, het testen van de pakketgrootte, een eerste kalibratiedownload (ongeveer 70 s voor een camera die nieuw is op deze computer), het starten van de stream. Een verzoek dat ondertussen een camera nodig heeft (`read_nodes`, `set_settings`, een opname, een tweede verbinding) wacht hier maximaal 5 s op, en mislukt vervolgens met HTTP-status 503 (`CAMERA_POOL_BUSY`), waarbij de camera wordt genoemd die verbinding maakt (`ChlorosConnectError`); vraag het over even nog eens. De lijsten wachten nooit: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` en `CameraSession.status()` antwoorden onmiddellijk. Een camera die bezig is met verbinden, wordt vermeld onder het antwoord van de pool: `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — en `elapsed_s`), en de waarde van `status()` is `connecting: True`, met de waarde van `phase`; de andere camera's worden live uitgelezen wanneer de pool vrij is, anders wordt de laatste uitlezing gebruikt (`live: False`).

#### `CameraSession`-methoden

| Methode | Beschrijving |
| --- | --- |
| `status(timeout=15.0)` | De live-status van de camera, zoals gelezen door `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` of `stopped`), `device` (model, serienummer, firmware, temperatuur, `is_mono`) en de huidige instellingen. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Leest GenICam-knooppunten; retourneert `{nodes, errors, enums, device, was_in_pool, in_array_id}`, plus `frame_values` en `notes` terwijl de automatische belichting actief is (zie hieronder). |
| `set_settings(**kwargs)` | Schrijf knooppunten op basis van vriendelijke naam (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Een `exposure_time` (`gain`) is een handmatige belichting (versterking): geschreven zonder `exposure_auto` (`gain_auto`) wordt geschreven, wordt de automatische modus eerst uitgeschakeld, zodat de waarde niet wordt geweigerd door een actieve automatische modus. Het resultaat `applied` bevat wat de camera heeft vastgelegd; een schrijfpoging die de camera weigert, staat in `errors` en wordt niet vastgehouden. De fout geeft het bereik aan dat de camera nu accepteert (een `ExposureTime`-maximum volgt de beeldsnelheid van de camera), en een automatische modus die alleen voor de geweigerde waarde was uitgeschakeld, wordt weer ingeschakeld; `restored` geeft dit weer (bijv. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Neem een **enkel** frame vastleggen. Geeft een `CaptureResult` terug: een lijst met metadata-woordenboeken van frames (één per opgeslagen niveau), met `.skipped`, `.failed`, `.daq_files` en `.daq_matches`. (Het vastleggen van burst- of multi-frames is verwijderd — roep `capture()` in een lus aan als je een reeks nodig hebt.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Aaneengesloten (`interval_s=0`) of intervalopnames van deze camera, voor `count`-opnames en/of `duration_s` (ten minste één, zodat het proces wordt beëindigd), net zoals `ArraySession.capture_repeated()` dat doet voor een array. Geeft de lijst met `CaptureResult`’en terug; het stopt na een opname die meldt dat de schijf vol is. `on_capture(i, result)` wordt na elke opname uitgevoerd; andere trefwoorden gaan naar `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Vrijgeven uit de pool; de camera wordt teruggezet zoals deze bij het verbinden werd aangetroffen (triggerbedrading, PTP, instellingen); `keep_settings=True` slaat de instellingen in plaats daarvan op in het eigen geheugen, `clear_settings=True` reset dat geheugen (zie [Camerageheugen](#camera-memory--keep_settings--clear_settings)); retourneert `camera_memory` wanneer om een van beide wordt gevraagd. Een camera die deel uitmaakt van een array maakt de gehele array vrij. Geen actie als we zijn gekoppeld aan een reeds geopende sessie. Geeft een waarde terug zodra de backend het vrijgeven bevestigt, of wanneer er niets te vrijgeven valt (de camera is al verdwenen, of er draait geen backend); een vrijgave waarbij de time-out is verstreken of die door de backend is geweigerd, genereert de foutcode `ChlorosConnectError`, omdat de camera mogelijk nog steeds wordt vastgehouden en de volgende verbinding hiermee zou botsen. Het verlaten van het `with`-blok genereert nooit een foutcode. |

Attributen: `serial`, `device` (het detectierecord van de camera) en `backend_url`.

`capture()`-exportinstellingen (hetzelfde model als de array + GUI):

- `processing` / `levels` — `processing="all"` slaat elk toepasselijk exporttype op; `levels=["raw","radiance"]` slaat alleen die op (heeft voorrang op `processing`). Laat beide weg voor de standaardinstelling van de backend.
- `daq` — de DAQ-waarde van de gebruikte opname wordt standaard opgeslagen als een `.daq`-sidecar, inclusief een ‘raw-only’-opname, zodat het frame later opnieuw kan worden verwerkt tot reflectantie/index. `daq=False` slaat dit over wanneer geen enkel niveau er gebruik van maakt. `force_daq=True` is de oudere spelling van de standaardinstelling. Geen DAQ, geen `.daq`.
- `daq_tolerance_ms` — hoe ver de meting van de belichting mag afwijken en nog steeds binnen de tolerantie valt (standaard 500). Alleen de beoordeling verandert.

#### `read_nodes()` bij automatische belichting — `frame_values` en `notes`

Terwijl `ExposureAuto` (`GainAuto`) actief is, leest het knooppunt `ExposureTime` (`Gain`) de **laatst geschreven waarde** terug, niet wat de camera op dat moment doet. `nodes` behoudt nog steeds die knooppuntwaarde. Wanneer je `ExposureTime` of `Gain` opvraagt bij een aangesloten camera waarvan de automatische modus niet `Off` is, wordt aan het antwoord toegevoegd:

- `frame_values` — de waarde die het nieuwste beeld van de camera bevatte, onder de naam van het knooppunt (`ExposureTime` in µs, `Gain` in dB);
- `notes` — één zin per dergelijk knooppunt waarin beide waarden worden vermeld.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Beide sleutels ontbreken wanneer de automatische modus `Off` is (het knooppunt is dan dewerkelijke waarde) en voor een camera die niet is aangesloten op de backend. Voor een array geeft `ArraySession.status()["member_ae"]` de framewaarden van elk element in één keer weer.

#### DAQ-gemeten belichtingslimiet (HTTP)

Als er een DAQ-lichtsensor is aangesloten, beperkt Chloros de belichting × versterking van een camera op basis van het gemeten daglicht, zodat een wit referentiepaneel niet overbelicht raakt. De limiet geldt alleen zolang een kalibratiedoel in het gezichtsveld van die camera is (te zien in het beeld binnen de laatste 10 s); als er geen doel in beeld is, wordt in plaats daarvan de normale automatische belichting van de camera, met zijn eigen bescherming tegen overbelichting, gebruikt, zodat de rest van een scène niet donkerder wordt gehouden dan nodig is. Er is geen SDK-wrapper; lees of schakel deze per camera in of uit met `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (de POST-body `{"enabled": false}` schakelt deze uit voor die camera). Het antwoord bevat `enabled`, `cap_us` (de live-drempelwaarde, `None` wanneer inactief, ook wanneer er geen kalibratiedoel in beeld is), `k_exp_g` (de constante waarmee de bovengrens wordt berekend) en:

| Veld | Betekenis |
| --- | --- |
| `k_exp_g_source` | Welke constante wordt gebruikt: `camera` — die van deze camera zelf, berekend op basis van het labpunt in het kalibratiepakket ( RGB-camera’s); `fleet` — de vlootconstante, gebruikt wanneer het pakket geen labpunt bevat; `override` — een veldoverschrijving die in de backend is ingesteld. |
| `k_exp_g_fleet` | De vlootconstante, ter vergelijking. |

### Gesynchroniseerde array — `ArraySession` (Smart-Prep)

`connect_array` is **het aanbevolen startpunt** voor opstellingen met meerdere camera’s. Het voert achter de schermen de volledige Smart-Prep-workflow via de grafische gebruikersinterface uit:

1. **Netwerkanalyse**(`/api/camera/array/recommend`) — stelt de**volledige sensor** (geen binning, geen ROI-uitsnede) in en laat de framesnelheid dalen tot wat deze host en het netwerk leveren, geprojecteerd op basis van de pixelformaten en de packing die Connect zal gebruiken. Frames worden nooit verkleind om de snelheid hoog te houden: geef een kleinere `width` / `height` of `binning=2` / `4` door om resolutie in te ruilen voor beeldsnelheid (de`alternatives` geeft een overzicht van deze opties met hun verwachte fps).
2. **Automatische tier-selectie** — `sim-capture-sim-emit` (elke camera belicht bij dezelfde trigger, met een afgestemde belichtingstijd om de bandbreedte te delen, waarbij de framesnelheid daalt naarmate de gedeelde bandbreedte afneemt) op elk netwerk waarop de camera’s zo kunnen worden afgestemd dat ze de bandbreedte delen; `slip-emit-and-capture` (camera’s gebruiken de verbinding om de beurt, niet gelijktijdig) alleen wanneer dat niet mogelijk is.
3. **PTP standaard ingeschakeld**— tijdstempels van verschillende camera’s komen op één gedeelde klok uit tot op**~1 ms**nauwkeurig, de [PTP-grandmaster](#ptp-grandmaster--time_sync_status) van de backend. Gelijktijdige belichting wordt geregeld door de M8-hardwaretrigger (**< 100 µs** tussen modules), niet door PTP: PTP synchroniseert *tijdstempels*, niet belichtingen.
4. **Automatische keuze van pixelformaat per camera** — mono → `Mono12`, kleur → `BayerRG12`.
5. **AE-seeding** — maakt een momentopname van de huidige AE-status van elke camera, zodat de verbinding de belichting niet halverwege reseteert.
6. **GPIO-triggerconfiguratie** — `connect_array` schakelt elke camera in (`TriggerMode=On`, `TriggerSource=Line2`), zodat de puls van de masterpuls de slaves aanstuurt via de M8-kabel. Dit is een stap die alleen voor arrays geldt: een enkele camera die met `LatticeCamera` wordt geopend, werkt in plaats daarvan in de vrije modus. Door de array vrij te geven (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) zet elke camera terug in de vrijloopstand (`TriggerMode=Off`), waarbij al het overige blijft zoals de connectie het heeft aangetroffen: de instellingen voor de trigger en de synchronisatielijn (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; de synchronisatielijn van de master terug naar een ingang), PTP en de camera-instellingen die tijdens de sessie zijn gewijzigd — tenzij `keep_settings=True` / `clear_settings=True` (zie [Camerageheugen](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Attributen: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` en `packed_reason` (de laatste vier zijn `None` op een handle van `attach_array`), `connect_stream` en `backend_url`. `connect_stream` is het `stream`-blok van het verbindingsantwoord, de vorm van de enkele verbinding plus `members` (`{serial: {state, why?}}`): `streaming` zodra de eerste gesynchroniseerde groep van de array is aangekomen (de backend wacht hierop, met een limiet van ongeveer 10 s), of `no_frames` met de laatste ophaalfout van elk lid als zijn `why` en, voor jumbo-pakketten, de 1500-byte-fallback in de `note`. `None` op een `attach_array`-handle.

#### `connect_array()`-handtekening

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` geeft aan hoe lang de aanroep op de verbinding wacht, en wordt naar de
backend verzonden als deadline. Als de verbinding tegen die tijd nog niet tot stand is gebracht, wordt deze ongedaan gemaakt: elke camera die
was geopend, wordt teruggezet in de toestand waarin deze werd aangetroffen en gesloten, er wordt geen array aangesloten en de
aanroep genereert een `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, waarbij de
stap wordt genoemd die de verbinding had bereikt). Een aanroep die een foutmelding genereert, laat dus nooit een array
achter zich. Een array van 9 camera’s maakt binnen ongeveer een minuut verbinding; direct nadat
de camera’s zijn opgestart, kan dit enkele minuten duren. Een array die actief is maar nog geen
gesynchroniseerde framegroep heeft, wordt geretourneerd met `connect_stream["state"]`
`no_frames`. Het logboek van de backend bevat één tijdlijn per verbindingsstap
(`[array-connect <id>] <step>: <s> s …`) en een laatste regel waarin de traagste stap wordt genoemd.

`force_tier`-waarden:
- `"sim-capture-sim-emit"` — echt gelijktijdig (alle camera’s vuren af op dezelfde klokflank).
- `"sim-capture-ftd-stagger"` — flexibele spreiding in het tijdsdomein (cams zenden op licht verschoven tijdstippen uit, zodat pakketten serieel over de kabel lopen).
- `"slip-emit-and-capture"` — sequentiële opname per cam (geen temporele synchronisatie). Wordt alleen automatisch gekozen wanneer de camera's niet langzaam genoeg kunnen worden afgestemd om de kabel gelijktijdig te delen.

`wire_ceiling_mbps` overschrijft het **continue kabelbudget van de host** in MB/s — het enige
getal waaraan de toewijzing van de gehele array is gekoppeld. Laat het op `None` staan om de automatisch gedetecteerde
waarde te gebruiken. Verlaag het wanneer de array GVSP-beschadigde frames meldt: de automatische waarde wordt afgeleid
van de door de NIC gerapporteerde verbindingssnelheid, die USB-adapters, smalle PCIe-lanes en
drukke gedeelde fabrics overschat — en deze overschatting komt tot uiting in beschadigde frames in plaats van in een
zichtbaar trage verbinding. De waarde wordt opgeslagen in het array-opnameblok van het project, dus bij het
opnieuw openen of een latere `connect_array` wordt deze net als elke andere array-instelling hersteld.
Zie [Arraystatus](#array-health--which-subsystem-is-losing-frames).

#### Gecomprimeerd 12-bits transport (`packed=`)

Bij volledige sensorbelasting wordt de framesnelheid van een array bepaald door hoe snel elke camera
zijn frames kan verzenden. 12-bits formaten (`BayerRG12`, `Mono12`) worden normaal gesproken als 16-bits
woorden verzonden; bij ‘packed transport’ worden dezelfde 12 bits in 1,5 bytes per pixel verzonden en
op de host weer uitgepakt, waardoor het verzenden van elk frame 25% minder tijd kost — ongeveer
**33% meer frames per seconde bij volledige sensoren** (gemeten: 10,7 → 14,0 fps op een
opstelling met 9 camera’s). **De opgeslagen gegevens zijn identiek**: dezelfde waarden, dezelfde 16-bits
bestanden, dezelfde metagegevens.

- `packed="auto"` (standaard) comprimeert alleen wanneer dit nuttig is **en** de host het
  ontpakken aankan: bij het verbinden meet de backend zijn eigen ontpakkingssnelheid,
  het aantal CPU’s, de huidige belasting en het netwerkbudget. Een desktop met voldoende cores
  pakt bij volledige sensordata in; een drukke of energiezuinige host (single-boardcomputers,
  embedded controllers) streamt ongepakt. Kleine ROI’s en gebinnede frames hebben hier weinig baat bij.
- `packed="off"` (of `False`) comprimeert nooit — om dit uit te sluiten tijdens het diagnosticeren,
  of op een host die zijn CPU voor ander werk moet reserveren.
- `packed="on"` (of `True`) comprimeert altijd 12-bits formaten.
- Elke andere waarde genereert `ValueError`. 8-bits en 16-bits formaten worden nooit
  gecomprimeerd.

`analyze_array_network(..., packed="auto")` komt tot dezelfde conclusie,
dus de fps is de snelheid waarmee de array zal draaien (het antwoord bevat
`packed_transport` en `packed_reason`); de standaardinstelling van `pixel_format` is wat
connect toepast (Mono12 / BayerRG12) in plaats van 8-bits.

De beslissing en de reden daarvoor staan in de sessie:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` past dezelfde overschrijving toe op elke
array in een project (standaard `None`, wat betekent dat `"auto"`: een project geen
'packed'-instelling per array opslaat).
Voor een machinebrede standaardinstelling stelt u `CHLOROS_ARRAY_PACKED12=off` (of `on`) in de
omgeving van de backend in; een expliciete instelling van `packed="on"` / `"off"` heeft nog steeds voorrang.
`/api/camera/array/status` rapporteert `packed_transport` en `packed_decision`.

#### Verwachte beeldsnelheden

Gemeten beeldsnelheden van de array (hardware-gesynchroniseerd, 12-bits; volledige sensor = 2048 × 1536). Gebruik deze om `status()["actual_fps"]` te controleren; `analyze_array_network()` geeft een schatting van de snelheid voor je eigen host en netwerk.

| Configuratie | Framesnelheid array |
| --- | --- |
| Volledige sensor, PCIe 10 GbE-host-NIC, gecomprimeerd transport (`packed="auto"` op een desktop) | ~14 fps |
| Volledige sensor, PCIe 10 GbE-host-NIC, ongepakt (`packed="off"`) | ~10,5 fps |
| Volledige sensor, 8 camera’s op een USB 10 GbE-adapter | ~3,8 fps |
| Volledige sensor, 1 GbE-hostverbinding | 1,2–1,7 fps |
| 2 × 2 binning (`binning=2`), 10 GbE-host-NIC | ~30–33 fps |

- Op een PCIe 10 GbE-verbinding vormen de **camera's** de beperkende factor bij volledige sensor, niet het netwerk: 4 en 8 camera's draaien met dezelfde snelheid.
- **`target_fps` is de triggersnelheid, en de array draait nooit sneller.** `target_fps=1` laat de array draaien met 1 fps; `None` (de standaardinstelling) laat deze draaien met de hoogste snelheid die de host en het netwerk aankunnen. Een waarde boven die snelheid wordt naar die snelheid verlaagd (zie [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` van `4294` in `status()` is geen linksnelheid.** Dit betekent dat Windows de snelheid van de adapter niet kon aflezen (het stuurprogramma rapporteerde de grootste 32-bits waarde), wat typisch is voor USB-ethernetadapters. De bruikbare snelheid van een USB-adapter op Windows is hoe dan ook beperkt tot 200 MB/s (zie [Array Health](#array-health--which-subsystem-is-losing-frames)); zie voor het afstemmen hiervan de sectie „USB 10GbE-adapters” in de CLI-referentie onder [Host NIC Setup & Tuning](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Ook onder Linux geldt een USB-beperking: 400 MB/s op x86_64, 200 MB/s op NVIDIA Jetson.** De adapter wordt geïdentificeerd via sysfs (met terugval op `ethtool -i`), dus een USB 5 GbE- of 10 GbE-adapter wordt begroot op basis van de limiet in plaats van de op het typeplaatje vermelde 500 / ~1060 MB/s. Het x86_64-cijfer is gemeten op Linux: dezelfde adapter die op Windows frames beschadigt bij snelheden boven 200 MB/s, liet een opstelling met 4 camera’s foutloos draaien bij elke limiet tot 480 MB/s. Op de NVIDIA Jetson werkte dezelfde adapter alleen foutloos bij 200 MB/s en verloor hij daarboven af en toe frames. Als je host meer data verwerkt zonder dat er frames verloren gaan met `gvsp_corrupt`, verhoog deze dan met `wire_ceiling_mbps`. Op Linux is voor een adapter op een USB 2.0-poort 30 MB/s gereserveerd; sluit deze dan aan op een USB 3-poort.

#### Over-Subscription (de ondergrens per camera)

Een array werkt op elk netwerk; de framesnelheid bepaalt de limiet. Sim-emit-pacing wijst elke camera een gelijk aandeel toe van het botsingsveilige netwerkbudget en ze geven allemaal nog steeds beelden af bij dezelfde trigger, dus op een trage verbinding is het aandeel — en de framesnelheid — klein: 8 volledige-res 12-bits camera’s op 1 GbE zonder jumboframes krijgen elk 6,5 MB/s, ongeveer **1 fps**(ongeveer 1,3 fps gepakt). Het aandeel heeft een ondergrens op de laagste snelheid waarmee de camera’s kunnen worden aangestuurd (`per_cam_floor_bps`: bij het verbinden wordt het GevSCPD-bereik van elke camera gelezen, en nooit minder dan 1 MB/s). Als de verbinding een lagere snelheid vereist, valt de verbinding terug op**seriële**verzending (`slip-emit-and-capture`): de camera’s werken om de beurt op de volledige verbindingssnelheid, zodat elk frame aankomt maar de leden**niet gelijktijdig worden blootgesteld**— de responslaag en blokkers (`SERIAL: ...`) geven dit aan; geef `force_tier="sim-capture-sim-emit"` door om de terugval te weigeren. De framegrootte speelt geen rol bij het delen, dus**binning en ROI verhogen de framesnelheid maar veranderen niets aan het feit of een array past**. Alleen een verbinding die de array bij geen enkele synchronisatie, noch gelijktijdig noch om beurten, kan verwerken,**raakt overbelast** — de storingsmodus is GVSP-pakketverlies, niet een lagere framesnelheid. Oplossingen: minder camera’s, end-to-end jumboframes of een snellere netwerkkaart.

- De reacties `analyze_array_network()` en `/api/camera/array/connect` bevatten `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` en `per_cam_floor_bps`. Wanneer `oversubscribed` waar is, zet de projectie **de fps-velden op nul** (`achievable_fps_max` / `fps_bright` / `fps_dark`) in plaats van een misleidende, trage maar functionerende snelheid te rapporteren.
- `POST /api/camera/array/connect` accepteert een `pin_resolution` body-parameter (**alleen HTTP — geen SDK-kwarg**; `connect_array` stelt deze niet beschikbaar). Het is een strikte modus: een overbelaste verbinding waarbij `pin_resolution` is ingesteld, wordt**kategoraal geweigerd** met een foutmelding waarin elke mogelijke oplossing wordt genoemd. Zonder deze instelling gaat de verbinding door met een waarschuwing dat de array pakketten zal verliezen. In beide gevallen behoudt de verbinding de gevraagde framegrootte — er wordt nooit uit eigen beweging gebundeld of bijgesneden.
- Alleen voor testdoeleinden: stel `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` in de omgeving van de backend in om de weigering te verlagen naar een luide waarschuwing — je maakt dan toch verbinding en accepteert het pakketverlies.

#### Array-status — welk subsysteem verliest frames

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) bevat een live `health`-blok op een
aangesloten array, dat opnieuw wordt geëvalueerd op een voortschrijdend **10-seconden** venster. Het splitst frameverlies
op in de twee oorzaken die tegengestelde oplossingen vereisen, in plaats van één „onvolledig“-percentage dat
geen van beide benoemt:

| Veld | Wat het betekent | Welk subsysteem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per serienummer) | Het frame **kwam aan en was structureel defect**— GVSP-pakketverlies. |**Netwerk**: bandbreedte, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Het frame **is helemaal niet aangekomen**— de camera is niet geactiveerd, of er is niets uitgezonden. |**Trigger / sync**: M8-kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Slechtste overdrachtssnelheid per elk. | — |
| `per_cam_rate_pct` | Gecombineerd percentage onvolledige opnames per camera (beide oorzaken samen). | — |
| `stable_for_seconds` | Hoe lang elke camera onder de 0,01 % is gebleven. | — |

Naast `health` vermeldt hetzelfde record het getal waaraan de gehele toewijzing hangt:

| Veld | Wat het betekent |
| --- | --- |
| `wire_ceiling_mbps` | Het geldende, aanhoudende bandbreedtebudget van de host, MB/s. |
| `wire_ceiling_source` | Waar dat getal vandaan komt, in woorden — bijv. `USB-capped 200 MB/s (was theoretical 1062; …)` of `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` wanneer `wire_ceiling_mbps=` dit heeft ingesteld. |
| `nic_is_usb` | `true` voor een USB-ethernetadapter. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interpretatie:** een waarde anders dan nul voor `gvsp_corrupt_rate_pct`, terwijl `never_arrived_rate_pct` op 0 staat, betekent
dat de trigger en de kabelsynchronisatie perfect zijn en dat 100 % van het verlies zich op het netwerkpad bevindt — verlaag
`wire_ceiling_mbps` en maak opnieuw verbinding. Het omgekeerde patroon wijst juist op de synchronisatiekabel of de
triggerlijn.

> **`target_fps` is niet de oorzaak van beschadigde frames.** De GevSCPD-pacing wordt eenmalig vastgelegd bij
> het verbinden, dus het verlagen van de triggerfrequentie verandert de duty cycle en niet de
> burst-snelheid bij gelijktijdige uitzending. Een gemeten verlaging van de vraag met een factor 5 leverde geen verbetering op, terwijl
> het verlagen van de kabelcapaciteit van 240 naar 200 MB/s zorgde ervoor dat hetzelfde systeem van 10,4 % beschadigde frames naar
> 0,00 % ging.

> **Een actieve array kan zichzelf niet opnieuw plannen op LATTICE (M3C / M3M) firmware.** Het kan
> dit niet zelf verhelpen; koppel de verbinding los en maak deze opnieuw aan, zodat de connect-time picker opnieuw plant op basis van
> de nieuwe limiet.

Een **USB-ethernetadapter wordt door de probe beperkt tot 200 MB/s**, ongeacht het
typeaanduiding: de efficiëntietabel die een verbindingssnelheid omzet in een continu haalbare waarde is
afgeleid van PCIe, en een USB-NIC geeft zijn Ethernet-verbindingssnelheid door terwijl deze wordt beperkt door de
USB-bus en de bijbehorende driver. De limiet is absoluut, geen fractie — een USB 1 GbE-adapter
haalt ~80 MB/s en wordt hierdoor niet beïnvloed. Dit geldt voor Windows en Linux (inclusief Jetson);
zie [Verwachte framesnelheden](#expected-frame-rates) voor waar dit is gemeten.

#### `ArraySession`-methoden

| Methode | Beschrijving |
| --- | --- |
| `status(timeout=10.0)` | Live-status van de actieve array: `target_fps` / `target_fps_auto` (de triggerfrequentie en of de frequentiecontroller deze beheert), `target_fps_clamped_reason` (waarom een vastgezette frequentie lager wordt gehouden, anders `None`), `actual_fps` (gemeten; 0 zodra de grab-loop het heeft opgegeven of vastloopt), `recommended_fps`, `streaming` (False zodra de grab-loop heeft opgegeven — controleer dit, niet alleen `last_error`; blijft ‘True’ na een herstart van de instellingen, wat door `restarting` wordt gemarkeerd), `stalled` (True zolang de grab-loop draait maar gedurende ongeveer tien frameperioden, 2–30 s, geen framegroep heeft gepubliceerd; `last_error` geeft dit pas na 60 s aan), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` alleen wanneer de array was verbonden met PTP **en** elk lid heeft voor het laatst `PtpEnable` ingeschakeld gelezen; niet hetzelfde als vergrendeld, wat `ptp_epoch` meet), `ptp_configured` (waar de verbinding om vroeg), `ptp_off_members` / `ptp_unread_members` (leden die voor het laatst `PtpEnable` 'uit' hebben gelezen / niet hebben geantwoord), `member_recovery` (`None`, of een lid dat de stroomvoorziening of zijn verbinding verloor: `state` `waiting` terwijl het niet reageert, `reopening`, vervolgens `recovered`, met `members` en `why` — de array opent het opnieuw wanneer het weer reageert en geeft het zijn PTP-tijd bij het verbinden terug, trigger- en synchronisatielijnbedrading, pacing en camera-instellingen, waarna het weer streamt zonder nieuwe `connect_array`) en `ptp_epoch` (gemeten), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` betekent dat Windows de snelheid van de adapter niet kon lezen — zie [Verwachte framesnelheden](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (of de stralingsketen per band bruikbaar is, en welk onderdeel deze heeft onderbroken), en `band_clamp` — terwijl een liveweergave de matrix weergeeft: het aandeel verlichte pixels dat de unmix op 0 heeft begrensd, per element (`members`: `{serial: [band0, band1, band2]}` in de R-, G-, B-bandvolgorde van de camera) en per band toont de composiet (`composite`: `{band name: share}`), waarbij `warn` het aandeel aangeeft dat de GUI markeert als „Bandgegevens met lage betrouwbaarheid“ (ongeveer één keer per seconde gemeten; `None` wanneer er geen liveweergave actief is), en `member_ae` — per element, de belichting (`exposure_us`) en versterking (`gain_db`) van het laatst gepubliceerde frame, afkomstig uit het frame zelf (`read_nodes` van `ExposureTime` / `Gain` geeft de laatste waarde terug die is geschreven terwijl de automatische belichting actief is, niet wat er op dat moment gebeurt). |
| `set_member_settings(serial, **kwargs)` | Schrijf de instellingen van één camera-lid, zoals `CameraSession.set_settings()` ze opneemt (belichting, versterking, doelhelderheid, ...); het lid blijft in de array. Genereert een `ChlorosConnectError`-fout voor een seriële camera die geen lid is. Zie [Belichting op array-leden](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Waar de smart-prep op is uitgekomen (het opnamelevel, de framegrootte, `pixel_format`), het live `health`-blok en het draadbudget van de host. Zie [Arraystatus](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Wijzig de triggerfrequentie terwijl de array draait, zonder opnieuw verbinding te maken. Een getal legt deze vast (de frequentieregelaar laat deze met rust); `None` zet deze terug op automatisch. Geeft `{requested, target_fps, target_fps_auto, clamped}` — `clamped` is True wanneer de snelheid is verlaagd tot wat de host en het netwerk aankunnen. Zie [Intervalopname voor drone-inspecties](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Eén beeld van elke camera, **allemaal vanaf één trigger**, als numpy-arrays (er worden geen bestanden geschreven). Geeft een `GrabResult` (een `dict`): `{serial: array}`, of `{serial: (array, metadata)}` met `with_metadata=True`. `.group_id` is de trigger die de frames delen; `.errors` koppelt elke weggelaten camera aan de reden (een camera die bij die trigger niet kan worden bediend, wordt weggelaten en nooit door een andere camera vervangen). `processing` is `raw` / `debayered` / `radiance` / `reflectance` / `display`. Hetzelfde geldt voor de project-API [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — één gedeelde implementatie — dus elk trefwoord daar (`band`, `align`, `bit_depth`, `refl_max`) werkt hier. Vereist `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator die één `grab()`-resultaat per tick oplevert (alleen trefwoordargumenten), afgestemd op `fps`, begrensd door `duration` (seconden) en/of `count` (`None` = onbegrensd). **Elke opgeleverde set is een nieuwe trigger:** een tick waarvan de nieuwste trigger de reeks is die hij al heeft opgeleverd, wacht op de volgende trigger in plaats van deze te herhalen, zodat de stream loopt met de lagere van `fps` en de eigen framesnelheid van de array. Genereert `FrameFetchError` in plaats van stil te vallen: onmiddellijk wanneer elke camera om een of andere reden de volgende tick zou herhalen, na 60 s aan ticks zonder frame, en na 60 s zonder nieuwe trigger („geen nieuwe trigger gedurende 60 s”). Dezelfde implementatie als `ArrayHandle.frame_stream()`. Zie [Live-frames in je eigen pijplijn](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Start de eigen continue opname van de backend (de modus „Continu” van de GUI) → [`ContinuousHandle`](#continuoushandle). Slaat **elke** gesynchroniseerde groep op die de array publiceert, zoals `capture()` deze zou opslaan (dezelfde exportinstellingen), totdat deze wordt gestopt of totdat `max_captures` / `duration_s` (`None` = totdat het wordt gestopt). Geeft een `ChlorosConnectError`-foutmelding als er al een proces voor deze array actief is. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Eén gesynchroniseerde opnamegroep. Geeft een `CaptureResult` (lijst met frame-dicts + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Geef een vorig resultaat`group_seq` door als `after_seq` om de volgende gesynchroniseerde groep daarna te krijgen in plaats van op een nieuwe te wachten. Exportinstellingen hieronder. |
| `capture(..., smart=True)` | **Slimme opname** — wacht tot AE op alle camera’s is gestabiliseerd en activeert dan de opname. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Snelste opname: alleen raw + de toegewezen DAQ-waarde (+ de vrije gecombineerde index). Komt overeen met de knop „Snelste opname“ in de GUI. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enkelvoudig / Continu / Interval in één begrensde lus. Geeft `list[CaptureResult]` terug.**Vereist `count` en/of `duration_s`** om te kunnen worden beëindigd (de SDK heeft geen Ctrl+C). Wordt ook vroegtijdig beëindigd, na de doorloop, wanneer een doorloop een volle schijf meldt (een `.failed`-vermelding met `disk_full`); die doorloop is het laatste resultaat. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Begin met het opnemen van de live gecombineerde-indexweergave naar video/GIF → `RecorderHandle`. De video is een Motion-JPEG `.avi` (ffmpeg is niet nodig). Als er geen GUI-weergave is geopend, geeft de backend het weer: `expr` (een indexuitdrukking in de bandnamen van de array, bijv. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), of standaard de NDVI van de array. Eén samengestelde recorder per array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Start een raw-Bayer-burst met hoge fps → `RecorderHandle`. Offline opnieuw verwerken met `build_video()`. `orient=True`: de producten die `build_video()` maakt, worden gedraaid volgens de camera-oriëntatie (producten per camera volgens hun eigen camera, de gecombineerde index volgens die van de master); de raw-frames blijven in hun oorspronkelijke formaat en het burst-manifest registreert de oriëntatie van elk onderdeel. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Verwerk een opgeslagen onbewerkte burst offline tot gekalibreerde video('s). Blokkeert totdat het klaar is (`wait=True`) en retourneert het bouwresultaat: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — zie [het bouwresultaat](#the-build_video-result) hieronder. |
| `build_video_status(job_id, timeout=15.0)` | Controleer de status van een offline bouwtaak: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Geef de hele array vrij — **als deze sessie deze heeft geopend** (`connect_array`): dan is het `force_disconnect()`, en genereert het een foutmelding `ChlorosConnectError` wanneer het vrijgeven niet wordt bevestigd (het `with`-blok genereert nooit een fout bij het afsluiten). Bij een handle van `attach_array` laat het de array actief blijven en geeft het een regel weer op stderr waarin dit wordt vermeld: de array kan behoren tot de GUI of een ander script. Elke camera blijft vrij draaien en blijft verder zoals de connectie deze heeft aangetroffen; `keep_settings=True` slaat de instellingen op in het eigen geheugen van de camera, `clear_settings=True` reset dat geheugen naar de fabrieksinstellingen (zie [Camerageheugen](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Geef de gehele array vrij, ongeacht wie deze heeft geopend. Gebruik dit op een gekoppelde handle wanneer je de array wilt beëindigen (bijvoorbeeld om deze opnieuw te verbinden met andere instellingen). Keert terug zodra de backend de vrijgave bevestigt, of wanneer er niets te vrijgeven valt (de array is al verdwenen, of er draait geen backend); een afbraakprocedure die is verlopen of die door de backend is geweigerd, genereert `ChlorosConnectError`. Met beide vlaggen wordt geretourneerd wat het geheugen van elke camera heeft gekregen (`camera_memory`), anders `None`. |

`capture()` exportcontroles (hetzelfde eindpunt dat de GUI en CLI gebruiken):

- `processing` / `levels` — `processing="all"` (of `levels=["raw","radiance",…]`) slaat elk toepasselijk exporttype per camera op; een enkele `processing`-waarde slaat alleen dat niveau op.
- `aligned` — **standaard ingeschakeld (`None`), aangezien de opname van dezelfde array door de Chloros-app**(de**uitgelijnde** opname-instelling per array). De niet-ruwe export van elk lid wordt door het [uitlijningsprofiel](#array-alignment) en bijgesneden tot de gemeenschappelijke overlap van de leden, zodat de bestanden pixel-geregistreerd zijn en dezelfde afmetingen, uitsnede en `Alignment*`-tags hebben als die van de app (binnen een ROI wordt de transformatie opnieuw gebaseerd op het venster); `XMP-mapir:AlignmentJSON` vermeldt `"applied": true` met zijn `crop_roi`. Raw blijft onvervormd op de oorspronkelijke grootte, maar draagt de transformatie mee in de metagegevens. `aligned=False` slaat elk bestand op in de oorspronkelijke grootte, onvervormd; `aligned=True` is de standaardinstelling. `continuous()`, `capture(smart=True)`, `capture_repeated()` en de `ArrayHandle.capture()` / `capture_smart()` op projectniveau hanteren dezelfde standaardinstelling. Of de vervorming nu wordt toegepast of niet, elk lidbestand registreert de `Alignment*`-transformatie van zijn camera zodra de reeks een uitlijningsprofiel heeft — behalve een camera die niet in het profiel is uitgelijnd (weggelaten of door een mislukte oplossing gedwongen tot identiteit), en de referentiecamera wanneer geen enkele andere camera is uitgelijnd: deze bevatten geen `Alignment*`-tags. Zonder profiel voor de reeks wordt de opname niet uitgelijnd opgeslagen, en het resultaat geeft dit aan: `result.alignment` is `{"requested": True, "applied": False, "warnings": [...]}`, één waarschuwing per reden waarom de warp niet kon worden toegepast (geen profiel, leden met verschillende resoluties, een reeks die niet-gesynchroniseerd, een frame waarop het profiel niet kan worden toegepast). `applied` is alleen `True` wanneer elke export die om de warp vroeg, deze ook heeft gekregen. De automatische uitlijning van de array maakt het profiel zodra de belichting stabiel is; `ArrayHandle.calibrate_alignment()` / de `align-calibrate` van de CLI maken er een op verzoek. `result.oriented` geeft aan of `orient=True` de producten heeft gedraaid.
- `render_index=False` — sla de vegetatie-index-overlay per camera over; standaard wordt deze weergegeven waar deze is geconfigureerd.
- `daq` / `daq_tolerance_ms` — de DAQ-meting wordt standaard opgeslagen als een `.daq`-sidecar, zelfs als geen enkel gekozen niveau dit nodig heeft; `daq=False` slaat dit dan over, en `daq_tolerance_ms` stelt de overeenkomsttolerantie in (standaard 500). `force_daq=True` is de oudere spelling van de standaardwaarde. Zie [De DAQ-meting die een opname opslaat](#de-daq-waarde-die-een-opname-opslaat--daq_files-en-daq_matches).
- `orient=True` — zet de **producten**van elk lid (`display`, `radiance`, `reflectance`, de index per camera en het samengestelde beeld) aan de camera-oriëntatie, na de uitlijningsvervorming: met de oriëntatie van de**master**op het moment dat ze in het frame daarvan werden verplaatst (de standaardinstelling `aligned` — de leden blijven geco-registreerd), anders met die van hun eigen camera.**Standaard: elk onderdeelbestand in de oorspronkelijke oriëntatie**, zoals bij array-opnames altijd het geval is; `raw` en `debayered` blijven in beide gevallen in de oorspronkelijke oriëntatie. Elk bestand registreert de oriëntatie van het frame waarin de pixels zich bevinden (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — een raw-bestand, van de eigen camera), en het XMP-bestand `Alignment*` bevat de transformatie van het native-frame, toegepast vóór de oriëntatie; een RAW-bestand dat die transformatie bevat, registreert ook `ChlorosAligned*` (die van het masterframe), de oriëntatie van een product dat is vervormd naar het frame van het masterframe. De verwerking draait elk product met de oriëntatie die bij de pixels hoort: de uitgelijnde oriëntatie wanneer het RAW-bestand wordt vervormd (Array alignment aan), de oriëntatie van de camera zelf wanneer dat niet doet. `capture_repeated()` en `capture_fastest()` laten deze ongewijzigd door; de `ArrayHandle.capture()` / `capture_smart()` op projectniveau nemen deze ook over.

**TIFF-compressie (alleen HTTP-knop):** `ArraySession.capture()` verstuurt geen `compression`-sleutel, dus geldt de standaardinstelling van de backend — `POST /api/camera/array/capture` leest een `compression`-bodyparameter, `"none"` standaard: ongecomprimeerd, de pixels van de sensor precies zoals ze worden gelezen (~6,3 MB per frame in volledige resolutie). `"deflate"` (verliesvrije zlib L1 + horizontale voorspeller, ~4,1 MB/frame) schrijft kleinere bestanden langzamer — beide zijn verliesvrij en worden bij het importeren identiek gelezen. De SDK biedt hiervoor geen kwarg; de uitweg is `chloros-cli lattice array-capture --compression deflate` of onbewerkte HTTP. DEFLATE houdt de Python GIL vast, dus gecomprimeerd schrijven kan niet parallel worden uitgevoerd over de schrijfthreads per camera — voor aanhoudende opname in volledige resolutie op sensorsnelheid is de ongecomprimeerde standaardinstelling nodig. Details: [CLI-referentie → array-capture](cli-reference.md).**Opeenvolgende groepen (HTTP):** het eindpunt retourneert `group_seq`; een lus die dit bij het volgende verzoek terugstuurt als `after_seq`, ontvangt de volgende gesynchroniseerde groep — inclusief een groep die is gepubliceerd terwijl de vorige opname nog werd opgeslagen — in plaats van te wachten op een groep die na het verzoek is gepubliceerd, en `skipped_groups` rapporteert alles wat de lus heeft gemist. `capture_repeated`, de `--continuous` van de CLI en de continue modus van de GUI doen dit allemaal; intervalpasses blijven op elk gepland moment actueel.**Exportoverschrijvingen per lid (alleen HTTP):**hetzelfde eindpunt accepteert ook `exclude_serials` (lijst — leden uit de opgeslagen set verwijderen; de array wordt nog steeds als één gesynchroniseerde groep geactiveerd en uitgesloten leden worden geretourneerd in `excluded`), `serial_levels` (`{serial: [level tokens]}` overschrijvingen op cam-niveau) en `serial_index` (`{serial: bool}` overschrijvingen van index-overlays per cam). Dit zijn GUI-pariteitsparameters en**nog geen SDK-kwargs**; leden die ontbreken in de kaarten vallen terug op de array-brede `levels` / `render_index`.

##### Overgeslagen cams inspecteren — `CaptureResult.skipped`

`ArraySession.capture()` retourneert een `CaptureResult`, wat een subklasse is van `list`: doorloop deze, indexeer deze, `len()` — elk bestaand patroon blijft werken. Nieuwe code kan het `.skipped`-attribuut inspecteren om te zien welke cams werden uitgesloten en waarom. Het meest voorkomende geval zijn RGB-camera's in een array met gemengde filters wanneer je om `processing="radiance"` of `"reflectance"` vraagt — straling per Bayer-pixel is zinloos voor een breedbandsensor, dus de backend slaat die camera’s over in plaats van onzin te produceren.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Reden-tokens volgen het patroon `<level>-not-applicable-to-rgb-cam` (één vermelding per overgeslagen niveau, elk met `level`). Een camera waarvan het kalibratiepakket niet is geladen, slaat `radiance` / `reflectance` over met `<level>-needs-calibration (connect the camera so its cal pack loads)`, en bij het ontbreken van een bruikbare DAQ-reflectiemeting worden de sprongen overgeslagen met `reflectance-skipped-no-dls`. De andere reflectiespecifieke sprongen zijn `reflectance-skipped-no-fresh-dls` (geen nieuwe downwelling-meting beschikbaar), `reflectance-skipped-bound-daq-unavailable (…)` (de gekoppelde DAQ kon niet worden bereikt), en `dls-uncalibrated-band-<nm>` — de band ligt grotendeels buiten het radiometrisch gekalibreerde bereik van de DAQ-lichtsensor (~374–974 nm), dus de op de DAQ gebaseerde absolute reflectie-scheiding wordt geweigerd en het frame schakelt luidruchtig over naar de sensorrespons. Van de geleverde SKU’s veroorzaakt alleen F988 dit; het ondersteunde pad van die camera is de workflow met het reflectiepanel.

##### Bestanden die niet konden worden geschreven — `CaptureResult.failed`

Een opname waarbij slechts een deel van de bestanden kon worden geschreven, retourneert nog steeds: de lijst bevat elk bestand dat **wel** is geschreven, en `.failed` somt de rest op, één dict per bestand met `serial` (`None` voor de gecombineerde composiet en de sidecar van een array, `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, of `match` — de herkomst van de `.daq`'s `.match.json`, toen de `.daq` werd geschreven maar de bijbehorende sidecar niet) en `error`. Op een volle schijf bevat het item ook `disk_full: True`; elke latere opname op die schijf zou op dezelfde manier mislukken, en daarom stopt `capture_repeated()` daar. Er wordt geen uitzondering gegenereerd voor een gedeeltelijke opname, en er bleef geen halfgeschreven bestand achter op de schijf in `.failed`. `.failed` is leeg bij een volledige opname.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### De DAQ die een opname uitleest, slaat deze op — `.daq_files` en `.daq_matches`

Telkens wanneer een DAQ aan de camera of array is gekoppeld, of in een pool is opgenomen, slaat elke opname de gebruikte neerwaartse meting op: een `.daq` in de submap `daq/` van de opnamemap, samen met de bijbehorende `.match.json` en `.cameras.json` (waarvan de meting afkomstig is). Dit geldt ook voor een onbewerkte of gedebayerde opname, en het is hetzelfde bestand, ongeacht of de opname afkomstig is van de GUI, de CLI of de SDK. `daq=False` slaat dit over voor een opname waarvan de niveaus geen gebruik maken van de DAQ (een `radiance`-, `reflectance`- of `display`-niveau schrijft het nog steeds); `daq_tolerance_ms` verschuift de tolerantiegrens voor die opname en nooit de gebruikte meting. Welke DAQ: degene die aan de camera of array is gekoppeld (de lichtsensorselectie in de GUI, een projectkoppeling of `chloros-cli daq pool-bind`), of anders de meest recente die de backend heeft. De CLI beschrijft de bedieningselementen `--no-daq` en `--daq-tolerance-ms` ([CLI-referentie](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Een opname slaat de gebruikte lichtsensorwaarde op als een `.daq`, met daarnaast een `<file>.daq.match.json` die aangeeft hoe de waarde aan de belichting is gekoppeld: `method`, `gap_ms` (van de belichting naar de dichtstbijzijnde DAQ-waarde) en `in_tolerance` (binnen 500 ms, of de `daq_tolerance_ms` van de opname). De XMP-gegevens van het reflectantiebestand bevatten dezelfde informatie als `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` en `Chloros:DlsInTolerance`.

Een opname wacht op de eerste meting van de DAQ na de belichting, zodat deze de metingen aan weerszijden ervan kan combineren (`method` `interpolated`). Een DAQ-E verstuurt bij daglicht elke 0,4 s een meting en binnenshuis elke 1-2 s, dus `capture()` wordt pas ongeveer anderhalf van die intervallen later teruggestuurd; de belichting zelf loopt geen vertraging op. Er wordt niet gewacht op een DAQ die is gestopt met verzenden. Een DAQ-E-meting is het gemiddelde lichtniveau over het interval sinds de vorige meting, dus deze wordt in het midden van dat interval geplaatst: `gap_ms` wordt gemeten ten opzichte van dat midden, en de `.match.json`'`bracket_stamps_ns` geeft de eigen tijdstempels van de metingen weer. `ArraySession.continuous()` wacht zonder de opname te vertragen. De terugloop van `capture_repeated()`waarbij ze niet wachten, zodat ze hun frequentie behouden en worden gekoppeld aan de dichtstbijzijnde meting vóór de belichting.

Een meting verloopt nooit, dus er gaat geen frame verloren door een te late meting — in plaats daarvan wordt deze gemarkeerd:

- **De stream van de DAQ is vastgelopen** — de nieuwste meting wordt gebruikt, en `gap_ms` geeft aan hoe oud deze is (`in_tolerance: false` meer dan 500 ms).
- **Geen DAQ om uit te lezen**— verbinding verbroken (`disconnect()`, `chloros-cli daq pool-disconnect`), of de gekoppelde DAQ is niet bereikbaar — de laatste meting van de camera wordt hergebruikt en gekoppeld aan**deze** belichting: `method`, `retained`, `gap_ms` van deze belichting, `in_tolerance` wordt opnieuw berekend, en `retained_from_epoch_ns` verwijst naar de belichting waarmee het voor het eerst werd gekoppeld. Een camera die nooit een meting heeft gehad, krijgt nog steeds geen reflectiewaarde, en het resultaat geeft aan waarom (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, of de weigering van de opname).

Bestanden die een opname opslaat in de map van het project dat geopend is in Chloros, worden toegevoegd aan de bestandslijst van dat project, net zoals een opname in de app dat doet (bestanden die elders zijn opgeslagen, blijven ongemoeid): `CameraSession.capture()`, `ArraySession.capture()` (ook `smart=True`) en `capture_cameras_transient()`. Het resultaat staat vermeld in `.project_add` (`capture_cameras_transient()`: het `project_add` van het woordenboek) -- `held`, `files` en `message` van de backend; `held` is `True` terwijl een verwerkingsrun bezig is, en de bestanden worden samengevoegd zodra deze is voltooid. `None` wanneer er geen bestand in het project is terechtgekomen.

`CameraSession.capture()` en `ArraySession.capture()` geven deze terug als de resultaten `.daq_files` en `.daq_matches`, als de HTTP-opname-antwoorden (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) `daq_files` en `daq_matches` bevatten, één per `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` en `stream_fault` — de eigen streamfout van de DAQ, die alleen wordt gecontroleerd bij een meetwaarde buiten de tolerantie. Controleer, voordat u vertrouwt op de reflectie uit een opname, `in_tolerance` in de bijbehorende `.match.json` of XMP, of de [`status()`](#daqsensorsession-methods).

`processing`-niveaus:

| Niveau | Uitvoer |
| --- | --- |
| `"raw"` | Eénkanaals Bayer (mono-camera’s: de enkele band) rechtstreeks van de sensor. |
| `"debayered"` *(standaard in SDK)* | 3-kanaals kleur: de donkere offset (DSNU) van het kalibratiepakket afgetrokken van de raw-gegevens, gevolgd door OpenCV’s randgevoelige demosaic — geen lineariteits-LUT, flat field of unmix (monochrome camera’s: 1-kanaals grijswaarden). Het opgeslagen TIFF-bestand heeft de **RGB**-kanaalvolgorde (fotometrische RGB, zodat elke afbeeldingsviewer de ware kleuren weergeeft); de arrays in het geheugen die `grab()` / `frame_stream()` retourneren, hebben de**BGR**-volgorde van OpenCV volgorde — draai de kanalen om (`frame[..., ::-1]`) om een vergelijking te maken met een opgeslagen bestand. `chloros-cli process` schrijft hetzelfde resultaat als `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm via de volledige radiometrische keten. Alleen multispectraal — RGB-camera’s worden overgeslagen. |
| `"reflectance"` | uint16 0..32768 (Pix4D-ready); vereist een live DAQ-koppeling voor absolute referentie. Alleen multispectraal. |
| `"display"` | Volledige keten die overeenkomt met het GUI-voorbeeld (CCM + WB + gamma volgens het profiel van de camera). |
| `"all"` | **Eén bestand per toepasselijk niveau** voor elke camera (overeenkomend met de GUI-instelling „Capture All“ / CLI-standaard). Het geretourneerde `CaptureResult` bevat vervolgens één frame-dict per `(cam, level)`, met het niveau in elke dict; niet-toepasselijke niveaus verschijnen in `.skipped`. De DAQ die de opname heeft gelezen, wordt opgeslagen als een `.daq`-sidecar. |

> **Opmerking — de standaardinstelling wijkt af van de CLI.** `ArraySession.capture()` is standaard ingesteld op `processing="debayered"`; het commando `chloros-cli lattice array-capture` is standaard ingesteld op `processing="all"`. Geef `processing="all"` expliciet door vanuit de SDK om de opslag op meerdere niveaus van de CLI/GUI te evenaren.

##### Bestandsnamen voor array-opnames

Elk niveau wordt opgeslagen in een eigen submap van `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` voor `display`, `index/`, `composite/`). Elk bestand benoemt de camera als `SN<serial>-<filter>`: het serienummer, gevolgd door het filtergedeelte van het cameramodel — `SN214000533-FOCN` voor een `M3C-L87-FOCN`, `SN261902142-F450` voor een `M3M-L87-F450` — gevolgd door `-`, dus een naam heeft dezelfde door `_` gescheiden velden als voorheen. Een camera waarvan het model niet kan worden gelezen, is alleen `SN<serial>`, en opnames uit eerdere versies hebben geen filter in de naam. De bestandsnaam hangt af van het aantal niveaus waar een lid om werd gevraagd:

| Gevraagd aantal niveaus | Bestandsnaam | Voorbeeld (`reflectance`, standaard `format="tiff"`) |
| --- | --- | --- |
| **Eén niveau** (`processing="reflectance"`, of een `levels` met één item) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — geen niveausuffix, net als in eerdere releases, zodat bestaande scripts blijven werken | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Meerdere niveaus** (`processing="all"`, of `levels` met meer dan één item) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

Het [clamp-masker](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) volgt op de productnaam, `clamp_masks/<product folder>/<product name>_clampmask.tif`: het vastleggen van de reflectie op éénreflectantiemeting op één niveau schrijft `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, terwijl `processing="all"` `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif` schrijft. De regel voor het achtervoegsel wordt per lid bepaald op basis van de niveaus waarnaar werd gevraagd, dus een overgeslagen niveau (bijv. reflectie op een RGB-camera) verandert dit niet. Lees de paden uit de geretourneerde frames in plaats van ze zelf samen te stellen. Een opname met één camera (`CameraSession.capture()`) voegt altijd het niveau toe: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG/JPEG: alleen weergave- en indexproducten

Een PNG- of JPEG-bestand bevat 8 bits per kanaal. Een verzoek om een bestand met een niveau dat het niet kan bevatten, wordt geweigerd voordat er iets wordt vastgelegd, met een bericht waarin het niveau wordt genoemd en TIFF als het formaat dat dit niveau wel ondersteunt:

- **`radiance`**(float32) en**`reflectance`**(uint16, `32768` = 1,0) — en dus ook `processing="all"`, dat deze omvat. De SDK controleert dit aan de hand van de argumenten: `capture()` genereert**`ValueError`** zonder een verzoek (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` van een camera die meer dan 8 bits streamt** (12-bit M3C, Mono12 M3M) — de backend weigert deze, omdat hij de bitdiepte van elke camera kent: HTTP-statuscode 400 met `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, gegenereerd als `ChlorosConnectError` (`CameraSession` / `ArraySession`) of `RuntimeError` (`CameraHandle` / `ArrayHandle`). De opname op één niveau met `debayered` van één camera levert een 8-bits frame op voor PNG/JPEG en wordt niet geweigerd.

Sla radiometrische gegevens op als `format="tiff"` (`ext=".tiff"`); gebruik PNG/JPEG voor `display` en indexweergaven.

#### Hoe automatische belichting werkt

- **Eerst belichting, daarna versterking.** Als de versterking op ‘Auto’ staat, blijft deze op 0 dB totdat de belichting de maximaal toegestane waarde heeft bereikt.
- **Bij handmatige belichting blijft de versterking op ‘Auto’ staan** totdat je de versterking wijzigt of op ‘handmatig’ instelt. Terug op ‘Auto’ is de limiet van de versterking weer zoals deze eerder was.
- **Kleine hoge lichten maken de scène niet donkerder.** Een spiegeling tot 1 % van het beeld wordt genegeerd, en in de panelen van een bekend kalibratiedoel tot 0,5 % van een paneel.
- **De DAQ-belichtingslimiet geldt alleen zolang een kalibratiedoel in beeld is.** Dit voorkomt dat een wit paneel overbelicht raakt; als er geen paneel in beeld is, stelt de camera de belichting in op de scène.
- **Constant licht blijft stabiel, en veranderd licht stabiliseert zich binnen enkele seconden.** Bij constant licht blijven de belichting en de limiet ervan ongewijzigd, en na een grote verandering in het licht stabiliseert het beeld zich binnen enkele seconden. Een overbelichting beperkt de belichting niet langer permanent, en bij een verandering van het ROI-gebied wordt de belichting opnieuw bepaald.
- **Het helderheidsdoel blijft binnen bereik.** Het stijgt niet zolang de camera zich op de belichtingslimiet bevindt en de versterking niet kan toenemen, zodat terugkerend licht geen overbelichting in het beeld veroorzaakt.
- **Een camera waarvan de automatische belichting niet meer reageert, wordt op het niveau gehouden dat bij het huidige licht hoort.** In fel zonlicht kan een camera terugvallen naar de kortste belichtingstijd en daar blijven met een donker beeld, terwijl alle instellingen normaal lijken. Chloros houdt het beeld in de gaten, houdt die camera op het niveau dat voor dat licht gold, en laat het los wanneer het licht verandert. Een donkere of afgedekte lichtsensor heeft hier geen invloed op.

#### Belichting op array-leden

Een LATTICE-camera beperkt `ExposureTime` tot één periode van de beeldsnelheid. Terwijl een array streamt, voert Chloros voor elk lid `AcquisitionFrameRateEnable` uit, met een snelheid die net boven de triggervrequentie ligt, zodat de maximale belichtingstijd van een element iets onder de triggerperiode ligt, **bij elke ROI**:

- De frameperiode van een element is de triggerperiode minus een speling van 15 % van de periode (minimaal 3 ms, maximaal de helft van de periode), en de maximale belichtingstijd is die frameperiode minus ongeveer 0,1 ms. Voorbeeld: een trigger van 10 fps (100 ms) levert een lidfrequentie op van ongeveer 11,8 fps en een blootstellingsplafond van ongeveer 85 ms.
- **Een handmatige `ExposureTime` die langer duurt dan die bovengrens, vertraagt de trigger om hieraan aan te passen.** Chloros opent eerst de bovengrens van dat lid voor de waarde en laat de lagere snelheid een paar triggerperioden draaien (het schrijven kan een paar seconden langer duren), schrijft het vervolgens, en de trigger vuurt niet sneller af dan de belichtingstijd toestaat. Bij een array met automatische snelheid keert de snelheid die eerder werd gebruikt terug zodra elk lid waarvan de handmatige belichting de snelheid vertraagde, weer op automatische belichting staat. Een waarde die de camera zelf niet kan vastleggen, wordt nog steeds geweigerd, waarbij het bereik dat nu wordt geaccepteerd in de foutmelding wordt weergegeven, en de automatische belichting van het lid wordt hersteld als deze alleen voor dat schrijfproces was uitgeschakeld. Vóór versie 1.3.0 beperkte een kleine ROI elk lid tot de eigen vrije loopperiode van de sensor, ongeacht de triggerfrequentie: 4,9 ms bij 768 × 304, zodat geen enkel lid helderder kon worden gemaakt.
- Chloros verkort nooit een handmatige belichting om deze aan te passen aan een snellere trigger. Een automatische frequentie blijft binnen de grenzen van wat de handmatige belichtingen toestaan; een vastgezette frequentie die sneller is dan een belichting toestaat, laat dat lid lang genoeg ruimte voor die frequentie, en het accepteert geen triggers die sneller zijn dan de belichting toestaat. Verlaag de belichting of de vastgezette frequentie wanneer elk lid elke trigger moet verwerken.
- **Teruglezen:** het uitlezen van de knooppunten van een element toont `AcquisitionFrameRateEnable` True en `AcquisitionFrameRate` met de snelheid van het element; de activering wordt weer uitgeschakeld wanneer de array stopt.
- **De array is eigenaar van deze twee knooppunten. Schrijf `AcquisitionFrameRateEnable` of `AcquisitionFrameRate` niet op een array-lid** — stel in plaats daarvan desnelheid van de array in met `target_fps` / `set_target_fps()`.

Schrijf de blootstelling van een lid met `arr.set_member_settings(serial, exposure_time=20000)` (de trefwoorden van `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` doet hetzelfde). Het element blijft in de array. Open een array-element niet met `connect_camera()`: connect weigert een camera die al in een array zit, en de `CameraSession` die het zou retourneren, verwijdert de camera uit de array wanneer de verbinding wordt verbroken. `ArraySession.status()["member_ae"]` rapporteert de belichting van elk elementbelichting en versterking van elk element uit het nieuwste frame.

### Eén projectrecord, voor elke modus

Een projectmap bevat de instellingen van elk apparaat: `cameras.json` — die van elke camera (`cameras[].settings`, voor een enkele camera en voor elk array-lid) en de array-brede instellingen van elke array (`arrays[].capture`: formaat, ROI, binning, triggerfrequentie, draadbudget) — en `sensors.json`, de instellingen van elke lichtsensor (integratietijd, framegemiddelde, limiet). De app, de CLI, de SDK en de MCP-server lezen en schrijven allemaal dezelfde bestanden, op dezelfde manier:

- **Een apparaat dat onder een project is aangesloten, wordt daarin opgeslagen en registreert elke wijziging daarin.** "Onder een project" betekent dat er een project geopend is in de backend (die van de app), of — via een `ChlorosProject` (`open_project()`), dat zijn eigen map aan de backend doorgeeft (`project_path`). Een camera, array of lichtsensor die het project nog niet bevat, wordt bij het verbinden toegevoegd aan de bestanden ervan (`connect_camera()`, `connect_array()`, de lichtsensorverbinding, de CLI, de app, `chloros-mcp`), inclusief de bijbehorende instellingen. Een wijziging vanuit elke modus (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, de array-instellingen, `set_cap()`, de CLI, de bedieningselementen van de app) wordt even later opgeslagen in de bestanden van dat project. Een belichting, versterking of AE-doelwaarde die op een camera is ingesteld, wordt opgeslagen als ‘vastgehouden’, zodat deze bij het opnieuw openen weer wordt behouden.
- **Bij het openen van een project worden de apparaten daaruit ingesteld**, in elke modus: een camera, array of lichtsensor die is aangesloten zonder eigen instellingen, krijgt deopgeslagen instellingen (`lattice cam-connect` / `array-connect` bij een geopend project, `chloros-cli project connect`, `open_project().connect_all()`, het opnieuw openen van de app). Een array die onder een nieuwe id wordt aangesloten, werkt de array bij die het project met dezelfde camera’s heeft opgeslagen; een array met andere camera’s wordt ernaast toegevoegd.
- **De eigen opslag van de app overschrijft nooit de live-instellingen van een apparaat** met een oudere kopie daarvan, en behoudt per lid opgeslagen overschrijvingen van een CLI- of SDK-aanroeper.
- **Het pixelformaat dat je instelt, is het formaat dat een project behoudt.** Een formaat dat is ingesteld in Array-instellingen, via `--pixel-format` of via `connect_array(pixel_format=)`, inclusief 8-bit, wordt opgeslagen en bij het opnieuw verbinden exact overgenomen. Als er niets is ingesteld, maakt elke camera verbinding met de standaard 12-bits instelling (`BayerRG12`, `Mono12` op een monosensor). Een formaatnaam verwijst naar de bitdiepte; de familie is die van de sensor, dus een kleurencamera wordt nooit ingesteld op een Mono-formaat of een monocamera op een Bayer-formaat, en de regel `[PixelFormat <serial>] ... written` geeft aan wanneer een formaat is toegewezen. De exportbestanden van een 8-bits opname zijn 8-bits, inclusief reflectie: op schaal aangepast en voorzien van de `PixelScale`-code.
- Een apparaat dat zonder project is aangesloten, registreert niets.
- **Een opname volgt het project dat erna is aangemaakt; bij het sluiten blijft het bij zijn project.**Wanneer een project wordt aangemaakt — `create_project()`, `chloros-mcp`’s `create_project`, of de**Nieuw project**-functie van de app — of gedupliceerd (de**Dupliceren**-functie van de app, waarmee de kopie wordt geopend) terwijl een opname nog steeds naar het eerder geopende project schrijft, gaat de opname door en komt wat er wordt geschreven terecht in het nieuwe project, op dezelfde plek daarin (hieronder). Het openen van een bestaand project — `ChlorosProject.process()` (waardoor het project op de achtergrond wordt geopend), `chloros-cli process`, de optie**Project openen** van de app — zorgt ervoor dat een opname nog steeds doorloopt in het project waarin deze wordt opgenomen: de opname wordt daar voltooid en de bestanden worden toegevoegd aan de bestandslijst van dat project (de app stopt elke opname voordat een project wordt geopend).
  - `ArraySession.record()` en `ArraySession.burst()`: de video, of de burst-map met elk frame, wordt in zijn geheel daarheen verplaatst wanneer de opname stopt. Het resultaat van het stoppen geeft het nieuwe pad aan (`out_dir`, `video_path`), en daaruit wordt een burst samengesteld. Als het verplaatsen door de schijf wordt geweigerd, blijft de opname staan waar deze is opgeslagen, en wordt dit in het resultaat vermeld (`relocation_error`). De onbewerkte frames van een gestopte burst en `.daq` worden toegevoegd aan de bestandslijst van het project waarin deze terechtkomt; een video wordt niet toegevoegd aan de bestandslijst van een project toegevoegd.
  - `ArraySession.continuous()`: elke opname na de wijziging wordt in het nieuwe project opgeslagen; de opnames die eerder zijn opgeslagen, blijven waar ze zijn.
  - `DAQSensorSession.record_start()` (en de opname van de eigen lichtsensor van de app): de `.daq` wordt naar de map `light_sensor` van het nieuwe project verplaatst wanneer de opname stopt; `record_stop()` retourneert het nieuwe pad en het bestand wordt toegevoegd aan de bestandslijst van dat project.

  Een opname in een map buiten het geopende project blijft waar hij is, en `start_camera_continuous()` / multi-stream-taken behouden de map die ze toegewezen kregen.
- **In de app worden de opnames na het aanmaken van een nieuw project daarin geplaatst.** Het aanmaken of dupliceren van een project zorgt ervoor dat alle lopende opnameprocessen en opnames doorgaan, en elke opname die daarna wordt gemaakt (Alles vastleggen, Begin met vastleggen) wordt opgeslagen in het nieuwe project en toegevoegd aan de bestandslijst daarvan. Het openen van een bestaand project in de app stopt ze eerst allemaal.
- **Een project wordt niet verplaatst terwijl het opneemt.**Het verplaatsen van het geopende project naar een andere werkdirectory (de**Instellingen → Werkdirectory** van de app; geen enkele SDK-aanroep verplaatst een project, een HTTP-client die naar `/api/move-project` verzendt doet dat wel) wordt geweigerd met HTTP 409 `PROJECT_RECORDING` terwijl een array-opname of burst, een continue array-opname, een continue camera-/multistream-taak of een lichtsensoropname ernaar schrijft: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Dit staat los van de instellingen die de camera’s behouden bij het verbreken van de verbinding (zie [Camerageheugen](#camera-memory--keep_settings--clear_settings)): een project stelt zijn camera’s zelf in bij het openen.

### Camerageheugen — `keep_settings` / `clear_settings`

Wanneer een array wordt aangesloten, nemen de camera’s de instellingen van de array over, en kun je deze vervolgens wijzigen. Wanneer Chloros de camera’s loskoppelt — `ArraySession.disconnect()` / `force_disconnect()`, het einde van een tijdelijke functie, het stoppen van `start_camera_continuous()` — worden ze allemaal teruggezet naar de toestand waarin ze zich bevonden vóór de aansluiting: de bedrading van de trigger en de synchronisatielijn, PTP en camera-instellingen — `ExposureAuto` / `GainAuto` (en een handmatige `ExposureTime` / `Gain`), `TargetBrightness` en de AE-limieten en demping, witbalans, zwartniveau, `PixelFormat`, binning en de ROI, de kleurenpijplijn (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP’s `PtpSlaveOnly` / `PtpDomainNumber`, de pakketgrootte (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), de pacing (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), de chunkgegevens (`ChunkModeActive`, elk `ChunkEnable` onder zijn `ChunkSelector`), de deglitch van de trigger-input: een array geeft de synchronisatielijn van een slave (`LineFilterSelector` / `LineFilterWidth`) en de opnamefrequentie (`AcquisitionFrameRate` alleen wanneer de activering was ingeschakeld: wanneer deze is uitgeschakeld, geeft de snelheid de maximale waarde weer die de timing toestaat, niet een instelling). Alleen wat verschilt, wordt geschreven: de camera is eerst gestopt, de timing en beeldsnelheid vóór een belichting die hun beeldperiode nodig heeft.

Om de camera’s buiten een project te gebruiken, schrijven twee argumenten het **eigen geheugen** van de camera — waarmee deze opstart, in elke software (op `disconnect()` / `force_disconnect()`, elke tijdelijke functie, `start_camera_continuous()` en `ChlorosProject.disconnect_all()`; MCP: dezelfde argumenten als bij `disconnect_device` en de tijdelijke tools; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** slaat de instellingen van elke camera op zoals deze bij het afsluiten van de sessie in het geheugen stonden (UserSet2, de opstartset). De camera’s blijven vrij draaien (`TriggerMode` `Off`), de bedrading van de synchronisatielijn blijft zoals deze is aangetroffen. Een camera zonder UserSet2 slaat de instellingen op in UserSet1; deze computer onthoudt dit, dus het zelfherstel van de Chloros-array — dat een UserSet1 reset die is afgeweken van de basislijn van Chloros — laat deze camera met rust.
- **`clear_settings=True`** zet het geheugen van elke camera terug naar de fabrieksinstellingen: de basislijn van Chloros, die door de zelfherstelfunctie van de array wordt vastgelegd — fabrieksinstellingen met `TriggerMode` `Off`, `TriggerSource` `Line2`, automatische belichting, de gekalibreerde witbalans van een RGB-camera behouden — in UserSet1, dat de opstartset vormt.
- Niet beide (`ValueError`; de backend geeft 400 als antwoord). Het antwoord bevat `camera_memory`, per camera `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` of `{"cleared": bool, "errors": [...]}` — `force_disconnect()` geeft het terug; voor een taak wordt het geleverd met `stop()`.

De app biedt deze niet aan.

**Elke uitgang zet de camera’s terug.**Het terugzetten in de oorspronkelijke staat vindt plaats op elk uitgangspad: `CameraSession.disconnect()` (camera**X**van de app), de**X** van de app op een array of een willekeurig array-element (de hele array wordt vrijgegeven), waarbij de app wordt afgesloten (de backend krijgt maximaal 20 s om elke camera, array en actieve continue opname terug te zetten; een tijdelijke functie of een eenmalige aanroep die nog actief is, stopt bij het volgende veilige punt, zet eerst de camera’s terug, gedurende maximaal 8 s, en beantwoordt HTTP 503 `SHUTTING_DOWN`; een nog actieve `connect_array()` stopt vóór de volgende camera en zet de camera’s die hij heeft geopend terug, gedurende maximaal 10 s, en genereert `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); en een die tijdens het afsluiten binnenkomt, opent niets: HTTP 503 `SHUTTING_DOWN`), en een `chloros-cli`-opdracht die wordt beëindigd of waarvan de terminal wordt gesloten (de taak wordt gestopt en de camera’s worden eerst teruggezet). Een backend die crasht of geforceerd wordt beëindigd, kan niets terugzetten; bij het opnieuw opstarten wordt elke camera teruggezet naar het geheugen.**Een project stelt zijn camera’s zelf in.** Wanneer een array verbinding maakt via een project — het geopende project van de app, of een `ChlorosProject`, dat zichzelf aan de backend (`project_path`) bekendmaakt — krijgt elk lid de instellingen die het project ervoor heeft opgeslagen: een belichting, versterking of AE-doelwaarde die erop is vastgezet (vastgezet als de jouwe, zodat automatische belichting het met rust laat) en de witbalansmodus (correctie van Chloros; de eigen witbalans van de camera blijft uitgeschakeld), ongeacht wat er in het geheugen van de camera staat.

### Opnamemodi & Recorders

Het oppervlak van de array weerspiegelt het opnamepaneel van de GUI: de opnamemodi Enkel / Continu / Interval / Snelste sluitertijd, plus twee recorders (live-composite video en raw-burst → offline herverwerking).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**is de Continu/Interval-lus van de SDK. Omdat er geen `Ctrl+C` is om deze vanuit een script te onderbreken, moet je**`count`**en/of**`duration_s`**doorgeven (de lus stopt zodra een van beide wordt bereikt).**`interval_s`** wordt gemeten vanaf het begin van elke doorloop (net als in de GUI). Bij continue doorlopen (`interval_s=0`) worden `group_seq` → `after_seq` aan elkaar gekoppeld, dus het zijn opeenvolgende gesynchroniseerde groepen in de snelheid van de arraysnelheid van de array. De overige kwargs worden rechtstreeks doorgegeven aan `capture()`.
- **`continuous`**is de eigen continue opname van de backend — de modus ‘Continuous’ van de GUI.**Gebruik deze in plaats van `capture_repeated()` wanneer je elke trigger met een hoge framesnelheid op schijf wilt opslaan:** `capture_repeated()` verstuurt één HTTP-verzoek per groep vanuit je script, terwijl `continuous()` twee opnames tegelijk binnen de backend verwerkt zonder heen-en-terugverkeer tussen groepen. Gemeten bij ROI 512 op een opstelling met 9 camera’s: 73–77 opnames/s, tegenover een opstelling die ongeveer 78 publiceert. Gebruik `capture_repeated()` voor intervalopnames (`interval_s`) en voor passages die je één voor één inspecteert. Sluit op Windows de backend uit van Microsoft Defender voor hoge snelheden (zie [Stap 1](#step-1--install-the-chloros-platform-package)).
- **`record`** is *monitoring-grade*: het registreert de weergegeven live samengestelde index. Als de gecombineerde weergave van de GUI geopend is, registreert het die weergave zoals deze wordt weergegeven. Als er geen geopend is, rendert de backend de weergave zelf: `expr` als je er een doorgeeft, anders de NDVI-waarden van de array (de rode en NIR-banden die het dichtst bij 660 nm en 850 nm liggen, zoals de liveweergave ze selecteert), zodra de array is uitgelijnd. Noem de banden in `expr` zoals de array ze benoemt: `<band>_<filter>` voor een M3C-camera (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` voor een M3M (`Mono_850`); een korte kleurnaam (`NIR`, `Red`) werkt alleen als één band van de array deze heeft. Een GUI-weergave die tijdens de opname wordt geopend, wordt opgenomen zolang deze open blijft. Een opname die geen frame schrijft, mislukt met `error` en `"no-frames: …"` (de array kan nog bezig zijn met uitlijnen of het stabiliseren van de belichting), en er wordt een foutmelding gegenereerd met `stop()` (zie [`RecorderHandle`](#recorderhandle)). Eén samengestelde recorder per array (er wordt een foutmelding gegenereerd als er al een actief is).
- **`burst` → `build_video`** is *analysekwaliteit*: `burst` schrijft onbewerkte frames + een manifest per frame + één `.daq` per afzonderlijke DLS-uitlezing onder `<output>/bursts/<base>/` met de volledige snelheid van de opnamelus (geen keten, geen exiftool, geen liveweergave). `build_video` koppelt elk frame qua tijd aan de dichtstbijzijnde `.daq` en voert de radiance/reflectantie/index-keten van de importpijplijn opnieuw uit. `products` is een lijst van `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (standaard: de gecombineerde index). `burst().stop()` start ook automatisch een ‘best-effort’ gecombineerde-index-build, die als `build_job` in het stopresultaat wordt geretourneerd.

##### Het `build_video`-resultaat

| Sleutel | Betekenis |
| --- | --- |
| `success` | `True` alleen wanneer elk aangevraagd product is opgebouwd zoals gevraagd: ten minste één uitvoer, niets in `skipped`, en geen `degraded`. |
| `outputs` | Paden van de video's (en GIF's) die zijn weggeschreven. |
| `errors` | Problemen, als tekst. |
| `skipped` | Eén `{"serial", "level", "reason"}` per aangevraagd product waarvoor **geen** video is geproduceerd — een camera’s (geen kalibratiebundel voor die camera, geen DLS-meting voor reflectie, de verwerkingsketen faalt) of de gecombineerde indexen, die `"kind": "combined"` en `serial` `None` bevatten (geen rood/NIR-bandpaar onder de camera’s, geen groep met stralingswaarde voor beide banden). Elke vermelding leidt tot `success` `False`; de andere video’s worden nog steeds geschreven en weergegeven. |
| `substitutions` | Camera’s weergegeven op een ander niveau dan gevraagd, en waarom — een RGB-camera waarvoor radiance/reflectance is gevraagd, krijgt de weergavevideo, met witbalans wanneer de kalibratiebundel van de camera beschikbaar is. Dit is geen fout. |
| `notes` | Informatieve tekst, geen problemen — bijv. een camera zonder indexuitdrukking per camera heeft geen indexvideo, of de weergavevideo van een RGB-camera is niet witgebalanceerd (geen kalibratiebundel, of een zonder fabriekswitbalans), waarbij de camera wordt genoemd. Geen fout. |
| `degraded` | `True` wanneer een product is geschreven maar niet het gevraagde product is: de offline gecombineerde index is het ingebouwde genormaliseerde verschil tussen rood en NIR, waardoor een geconfigureerde indexuitdrukking niet wordt geëvalueerd. Dit leidt tot `success` `False`. |
| `combined` | Overzicht van de gecombineerde index: de gebruikte banden, het LUT-bereik, de opgebouwde frames. |
| `clamp_masks` | Paden van de [unmix-clamp-maskers](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) van de per-frame gekalibreerde TIFF's (`save_tiffs=True`), in `clamp_masks/<level>/` naast de `<level>/`-map van de TIFF's, één per TIFF waarin een pixel is geklemd. Een video bevat er geen. |
| `clamp_fractions` | Wat een video aangeeft in plaats van een masker: één `{"kind", "serial", "level", "frames"}` per videoproduct (`kind` `"combined"` voor de camera’s die deel uitmaken van de gecombineerde index), `frames` een lijst van `[frame, R, G, B]` — het aandeel van de pixels in dat frame dat de unmix heeft begrensd in de band die het R/G/B-kanaal bevat (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Geretourneerd door `ArraySession.record()` en `ArraySession.burst()`. Gebruik het als een contextmanager om automatisch te stoppen bij het afsluiten van de scope, of bestuur het handmatig.

| Lid | Beschrijving |
| --- | --- |
| `job_id` | Backend-taak-id (str). |
| `kind` | `"composite"` (van `record`) of `"raw"` (van `burst`). |
| `start_stats` | Het dict dat wordt geretourneerd door de `start`-aanroep. |
| `result` | `None` tijdens het draaien; het definitieve stop-resultaatdict zodra gestopt. |
| `stats(timeout=10.0)` | Live taakstatistieken (geschreven frames, gerealiseerde fps, verstreken). |
| `stop(timeout=60.0)` | De recorder stoppen; retourneert en slaat het eindresultaat op in de cache (`.result` blijft `None` als het stopverzoek zelf is mislukt). Voor een `record()`-opname die is mislukt — een opname waarbij geen enkel frame is geschreven (`error` `"no-frames: …"`) — wordt het resultaat opgeslagen in de cache op `.result`, waarna `ChlorosConnectError` wordt gegenereerd. Idempotent (een tweede aanroep retourneert het in de cache opgeslagen resultaat, zonder een fout te genereren). Het verlaten van een `with`-blok stopt de recorder zonder een foutmelding te genereren, dus controleer daarna `rec.result.get("error")`. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Geretourneerd door `ArraySession.continuous()`. Gebruik het als contextmanager — het verlaten van het `with`-blok stopt de opname als deze nog actief is — of stuur het aan met `status()` / `wait()` / `stop()`. Er wordt telkens één doorlopende opname per array uitgevoerd.

| Lid | Beschrijving |
| --- | --- |
| `start_status` | Het statusdict dat door de startaanroep wordt geretourneerd. |
| `result` | `None` tijdens het uitvoeren; het definitieve statusdict zodra de opname is beëindigd (ingesteld door `stop()`, of door een `status()` / `wait()` die het einde heeft waargenomen). |
| `status(timeout=10.0)` | Live statusdict (velden hieronder). |
| `wait(timeout=None, poll_s=0.5)` | Blokkeer totdat de opname vanzelf eindigt — `max_captures` of `duration_s` bereikt, de array leeg, een volle schijf, een fout — en retourneer de eindstatus. Met `timeout` de meest recente status na dat aantal seconden retourneren, zelfs als het proces nog loopt: zie `running`. |
| `stop(timeout=30.0)` | Stop de opname en retourneer de eindstatus. Idempotent (een tweede aanroep retourneert het in de cache opgeslagen resultaat). |

Statusvelden:

| Veld | Betekenis |
| --- | --- |
| `running` | Waar zolang de opname actief is. |
| `captures` | Gesynchroniseerde groepen opgeslagen. |
| `files` | Bestanden geschreven. |
| `failed_files` | Bestanden die niet konden worden geschreven (zoals in [`CaptureResult.failed`](#bestanden-die-niet-konden-worden-geschreven--captureresultfailed)). |
| `failed_captures` | Opnamepogingen die in hun geheel zijn mislukt, inclusief pogingen waarbij de opname opnieuw is geprobeerd (een HTTP-503-fout terwijl de array niet op tijd een nieuwe groep had). |
| `skipped_groups` | Door de array gepubliceerde groepen die niet zijn opgeslagen (de opname liep achter). |
| `captures_per_s` | Gemiddelde opslagpercentage sinds de start. |
| `elapsed_s` / `started_at` | Aantal seconden dat het proces tot nu toe heeft gedraaid; starttijd (Unix-epoch-seconden). |
| `last_group_seq` | De nieuwste gesynchroniseerde groep die is opgeslagen. |
| `last_error` | De laatste fout, of `None`. |
| `stop_reason` | Eenmaal beëindigd: `count` (`max_captures` bereikt), `duration` (`duration_s` bereikt), `stopped` (`stop()` aangeroepen), `array_gone` (de array werd losgekoppeld), `error` (20 mislukkingen op rij, of een verzoek dat door de backend werd afgewezen), `disk_full` (een bestand waarvoor de schijf geen ruimte had). `None` tijdens het draaien. |
| `max_captures` / `duration_s` | De grenzen waarmee het werd gestart (`0` = geen). |
| `defender_cores` | CPU-kernen die Microsoft Defender in de afgelopen 10 seconden heeft gebruikt; `None` totdat er een wordt gemeten, en altijd `None` op een host zonder Defender. |
| `defender_warned` | Waar zodra de capture heeft geregistreerd dat Defender deze vertraagt (bij 1 kern of meer). Sluit de backend uit van Defender-scans — zie [Stap 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Live frames in je eigen pijplijn

Om gesynchroniseerde frames in je eigen code te voeden — machine learning, een aangepaste viewer, je eigen recorder — gebruik je `ArraySession.frame_stream()`. Elke tick levert een `GrabResult` op: `{serial: numpy array}` voor elke camera, **allemaal vanuit één trigger**, met de id van die trigger op `.group_id`. Het is dezelfde implementatie als de `ArrayHandle.frame_stream()` van de project-API, dus alles onder [Eén trigger per leesbewerking](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) en [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) geldt voor beide.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Elke opgeleverde set is een nieuwe trigger.** `frames.group_id` herhaalt zich nooit binnen een stream; wanneer je om meer `fps` vraagt dan de array aankan, wacht de stream op de volgende trigger, zodat deze op de langzamere van de twee wordt uitgevoerd.
- **Ontbrekende camera’s worden benoemd, niet aangevuld.** Een camera die bij die trigger niet kon worden bediend, ontbreekt in het woordenboek en wordt vermeld in `frames.errors`.
- **Het eindigt met een knal.** Bij een weigering wordt `FrameFetchError` bij de volgende tick herhaald, na 60 seconden zonder frame of na 60 seconden zonder nieuwe trigger.**Wat het oplevert.**`frame_stream()` verplaatst volledige numpy-frames via de lokale HTTP-verbinding en haalt een maximale snelheid van bijna**170 MB/s**. Dat is voldoende voor liveverwerking van ruwe frames, of van een gebundelde of ROI-array, maar niet voor elke trigger op volledige resolutie — om**elke** trigger te behouden, sla je deze op schijf op met [`continuous()`](#continuoushandle). Gemeten op een array van vier camera’s (FOCN, FRGN, FRGB, F450) met ‘packed transport’; elke tick was een complete set van één trigger:

| | Volledige sensor (array 14,2–14,4 fps) | 2 × 2 binning (array ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 sets/s | 22,2 sets/s |
| `frame_stream(processing="debayered")` | 2,3 sets/s | 9,3 sets/s |
| `frame_stream(processing="display")` | 0,8 sets/s | 3,0 sets/s |
| MJPEG per camera, 4 streams bij 720p | elk 13,5 fps; elk frame gekoppeld aan een trigger | elk ~25 fps; ~45 % van de triggers aanwezig op alle 4 |

Gebruik `processing="raw"` voor doorvoer en demosaicing aan jouw kant; het `debayered`-niveau verwijdert ook de donkere offset van het kalibratiepakket, wat bij een demosaic van `raw` niet het geval is.

#### MJPEG-streams per camera (geen RTSP)

**RTSP wordt niet ondersteund.** De streaminginterface is MJPEG via HTTP (`multipart/x-mixed-replace`, één JPEG per deel, elk deel met een `Content-Length`), één URL per camera:

| URL | Wat het weergeeft |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bits voorbeeldweergave, tot ~30 fps. Voor een array-lid bepalen `max_h` (standaard 720, 64–2160) en `jpeg_q` (standaard 80) bepalen de grootte en kwaliteit. |
| `GET /api/camera/<serial>/stream-annotated` | Hetzelfde voorbeeld met daarop de GUI-overlays (zebra, raster, dradenkruis, histogram, peaking, spotmeter). `ChlorosLocal.camera_stream()` leest dit voor je. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bits reflectantie per band — zie [Live reflectantiestream](#live-reflectance-stream-mjpeg). |

- **Triggerheaders.** Voor een array-element bevat elk deel van alle drie de streams `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` en `X-Chloros-Timestamp-Ns`. Delen van verschillende camera’s met dezelfde `X-Chloros-Group-Id` vormen dezelfde trigger: verzamel de delen en groepeer ze op basis van die header om een set weer samen te stellen. Een op zichzelf staande camera heeft geen groep: de preview-delen bevatten geen van deze headers (de reflectantieonderdelen hebben een lege `X-Chloros-Group-Id`).
- **De preview-stream laat frames vallen bij hoge belasting.** De stream levert het nieuwste frame en slaat alle frames over die niet op tijd konden worden verzonden; bij hoge array-snelheden bereikt dus niet elke trigger de stream van elke camera (zie de tabel hierboven). Gebruik voor elke trigger `frame_stream()` of `continuous()`.
- **`/api/camera/stream/<serial>` wordt gepauzeerd terwijl de gecombineerde livestream van de array open is** (`/api/camera/array/stream/<array_id>` — de arrayweergave van de GUI opent deze). De stream van elk lid verzendt vervolgens één plaatshouderframe met de tekst „Paused (array stream active)“, zonder `X-Chloros-*`-headers, en verder niets totdat de gecombineerde stream wordt gesloten; daarna wordt de stream vanzelf hervat. `stream-annotated` en `stream-reflectance` pauzeren niet. Sluit de array-weergave van de GUI wanneer een script de preview-streams per camera leest.
- **Gebruik `frame_stream()` voor getallen, niet voor afbeeldingen.**JPEG is verliesgevend en 8-bits; `frame_stream()` levert dezelfde resultaten op zonder codec in het pad.

### Intervalopname voor drone-inspecties

Een inspectievlucht heeft om de paar meter een gesynchroniseerde beeldgroep nodig, niet elk frame dat de array kan produceren. Twee instellingen regelen dit:**`target_fps`**bepaalt hoe vaak de camera’s een frame maken, en**`capture_repeated(interval_s=…)`** bepaalt hoe vaak een frame wordt opgeslagen.**De camera’s blijven streamen tussen het opslaan door.** Bij elk interval wordt de nieuwste gesynchroniseerde groep opgeslagen; de beelden daartussen worden niet opgeslagen. De automatische belichting past zich bij elk beeld aan, dus de belichting is al gestabiliseerd wanneer het opslaan plaatsvindt. Er is geen modus waarin de camera’s tussen opnames stoppen en voor elke opname opnieuw worden geactiveerd: elke opname zou dan moeten wachten tot de stream begint en de belichting zich stabiliseert.**Stel de framesnelheid in op wat het interval vereist.**Als je ‘auto’ laat staan, draait een array op de hoogste snelheid die de host en het netwerk aankunnen. Door een lagere snelheid in te stellen, verminder je de warmteontwikkeling van de camera, de netwerkbelasting en de belasting van de host-CPU, zonder dat dit invloed heeft op wat er wordt opgeslagen. Kies een snelheid die**2–4 frames per interval** oplevert:

- Bij elke opslag wordt gewacht op het volgende beeld nadat dit is aangevraagd, waardoor een opgeslagen groep tot één beeldperiode na het geplande moment terechtkomt. Een hogere snelheid houdt die vertraging klein.
- De automatische belichting wordt één keer per beeld aangepast, dus een zeer lage snelheid vertraagt de snelheid waarmee de belichting een voorbijtrekkende wolk volgt.

**Bereken het interval op basis van het vluchtplan:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Een voetafdruk van bijvoorbeeld 60 m bij 80 % voorwaartse overlapping komt neer op een afstand van 12 m; bij 10 m/s is dat `interval_s=1.2`, en een vastgezet `target_fps=3` levert tussen de 3 en 4 beelden per interval op. Het vastleggen is tijdgebonden: de SDK heeft geen modus die wordt geactiveerd door afstand of GPS, dus houd een constante grondsnelheid aan op meetlijnen.

**Sla de ruwe gegevens op het vliegtuig op en kalibreer later.** `interval_s` wordt gemeten vanaf het begin van elke opslag, dus als een opslag langer duurt dan het interval, begint de volgende te laat en verspringt de afstand. Door de ruwe gegevens samen met de waarde van de lichtsensor op te slaan (`processing="raw"`; deze waarde wordt standaard opgeslagen), blijft elke opslag kort en blijft elk frame achteraf converteerbaar naar reflectie.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**legt de snelheid vanaf het begin vast;**`arr.set_target_fps(fps)`**wijzigt deze tijdens de vlucht zonder opnieuw verbinding te maken, en**`arr.set_target_fps(None)`** zet deze terug op automatisch. Een snelheid die hoger is dan wat de host en het netwerk aankunnen, wordt verlaagd tot dat maximum — controleer `clamped` in het resultaat — en de laagste snelheid is 0,1 fps.
- **Gebruik `burst()` niet om frames te verwijderen.** Het schrijft elk frame dat de array produceert; het volgt wel op een vastgezette `target_fps`, dus het is alleen het juiste hulpmiddel wanneer elk frame gewenst is.
- **`target_fps` herstelt geen beschadigde frames.** Als je deze waarde verlaagt, wordt het aantal geproduceerde frames verminderd, niet de snelheid waarmee elk frame wordt verzonden; voor door GVSP beschadigde frames moet je in plaats daarvan `wire_ceiling_mbps` verlagen (zie [Array Health](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Aansluiten op een reeds verbonden array — `attach_array`

Als de array al actief is (de GUI heeft deze geopend, of een eerdere SDK-sessie heeft `connect_array` aangeroepen), gebruik dan `attach_array` om een handle te verkrijgen in plaats van opnieuw verbinding te maken. `connect_array` geeft in die situatie altijd de foutmelding „Camera  <sn>zit al in de array<id>", omdat het versturen van een POST-verzoek met `/array/connect` voor een lid in de pool niet idempotent is; `attach_array` leest `/api/camera/array/list` en vergelijkt op basis van array_id of serienummers.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Patroon: SDK-scripts die samen met de desktop-GUI worden gebruikt, moeten eerst `attach_array` proberen en terugvallen op `connect_array` als er nog geen array in de pool zit.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Een gekoppelde array wordt geleend, niet in bezit genomen.** Een handle van `attach_array` maakt de array nooit ongedaan: noch het afsluiten van het `with`-blok, noch een expliciete `disconnect()` geeft deze vrij (de expliciete aanroep drukt één regel af naar stderr waarin staat dat de array actief heeft gelaten). Dezelfde regel geldt voor `CameraSession` / `DAQSensorSession`. Het gelijktijdig gebruik met de GUI is dus veilig, met of zonder `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Als je het wel wilt beëindigen — bijvoorbeeld om opnieuw verbinding te maken met een andere `wire_ceiling_mbps` — roep dan `arr.force_disconnect()` aan. Opnieuw verbinding maken na een gewone `disconnect()` op een gekoppelde handle mislukt met de foutmelding "Camera  is<sn> al in array <id>", omdat er niets is vrijgegeven.

### Hulp bij netwerkanalyse

Handig voordat je de array opent — geeft aan of je voorgestelde instellingen zullen passen:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Er wordt nooit een backend gestart: een backend die niet reageert, genereert `ChlorosConnectError`, terwijl een afgewezen verzoek als resultaat wordt teruggestuurd. Een ongeldige `packed` genereert `ValueError`.

`status` is een van de volgende: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (anders `error` — inclusief een afgewezen verzoek: niet ingelogd, `error_code: AUTH_REQUIRED`, of een abonnement onder de SDK-drempel, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` betekent dat de gevraagde resolutie alleen bij een gemaximeerde triggerfrequentie in de RX-ring past — behoud de resolutie en ga verder van `target_fps=result["recommended"]["recommended_target_fps"]` naar `connect_array` (zie [Voorbeeld 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` verschijnt alleen wanneer het gevraagde frame bij geen enkele framesnelheid kan worden geleverd; connect verkleint frames nooit uit zichzelf, dus pas `recommended` zelf toe.

**Hoe de projectie te interpreteren** (hetzelfde model als het GUI-paneel „Array Settings“):

- **Burst (`frame_bytes_total`) wordt per camera opgeteld in het werkelijke pixelformaat van elke camera.**Mono**M3M**-camera’s streamen Mono12 (2 B/px), of met een `pixel_format` het Mono-formaat van de bijbehorende diepte (`"BayerRG8"` is `Mono8`, 1 B/px), dus zonder deze is een frame met volledige resolutie van 4 camera’s**~25 MB** met drie monocamera’s, niet de ~12,6 MB die een aanname van volledig 8-bit oplevert. De backend bepaalt het formaat van elke camera op basis van het model.
- **Admittance (`burst_fits_nic_ring`) houdt rekening met de afvoer**, niet op basis van ‘hele burst versus ring’: sim-emit past wanneer de host de RX-ring sneller leegtrekt dan de camera’s deze vullen. Een 10G-host + 1 GbE-camera’s**laten** volledige resolutie toe, zelfs wanneer de burst de ring overschrijdt; een 1 GbE-host blokkeert (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` is een conservatieve bovengrens voor seriële ophaling** — `max(readout+emit, N×emit)` waarbij de uitstoot per camera wordt begrensd tot de 1 GbE-cameralink, onafhankelijk van de belichting. Bijv. ~2,8 fps voor een 4-camera-array met volledige resolutie en 12-bits (komt overeen met de gemeten ~2,7–3,0 van de runtime). Volledig model: [CLI-referentie → Array-fps- en burst-model](cli-reference.md#array-fps--burst-model).
- **Na een continue opname geeft de projectie weer wat er is vastgelegd.** Een opname van 30 s of langer met een automatische snelheid registreert de snelheid waarop het systeem bij die arraygrootte op deze computer is uitgekomen; de korte pogingen van de snelheidsregelaar om een hogere snelheid te halen, worden buiten beschouwing gelaten. Vanaf dat moment vermeldt de regel in `array-connect`', de analyse van `connect_array()` en het paneel ‘Array-instellingen’ vermelden die snelheid wanneer deze lager is dan die van het model, met de beperking die in `cycle_limiter`: `"clock"` (de array draaide op de snelheid waarop hij tot rust was gekomen, terwijl de opnamelus wachtte op zijn triggerklok), `"loop"` (de opnamelus had geen tijd meer), `"capture"` (de opname raakte achterop en sloeg groepen over) of `"measured"` (er wordt niet aangegeven welke van de twee). Het cijfer van het model staat in `wire_fps_max`. De array stijgt nog steeds naar wat de verbinding aankan.
- **Over-subscription (`oversubscribed: true`) betekent dat de verbinding de array bij geen enkele beeldsnelheid die de camera’s bereiken kan verwerken** — de fps-velden (`achievable_fps_max` / `fps_bright` / `fps_dark`) geven 0 weer, en binning kan dit niet verhelpen (het verlaagt het aantal bytes per frame, niet het aantal bytes per seconde). Een trage kabel alleen veroorzaakt dit niet: in plaats daarvan daalt de framesnelheid, en als de camera’s niet langzaam genoeg kunnen worden aangestuurd, wordt het niveau `slip-emit-and-capture` (serieel). Oplossingen zijn minder camera’s, jumboframes of een snellere netwerkkaart; `max_cams_collision_safe` geeft de maximale capaciteit van je verbinding en camera’s aan. Het antwoord bevat ook `aggregate_demand_bps`, `collision_safe_ceiling_bps` en `per_cam_floor_bps`. Zie [Over-Subscription](#over-subscription-the-per-cam-floor).

### Detectie en weergave

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Beide accepteren alleen trefwoorden: `backend_url=` en `timeout=30.0`, en retourneren een lijst met woordenboeken. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) neemt ook `auto_start_backend=True`; de `list_*`-aanroepen starten nooit een backend.

---

## Smart-AE / Smart-Capture

LATTICE-arrays voeren continu AE uit op de achtergrond zodra ze zijn aangesloten, maar het duurt even voordat een nieuw ingestelde scène is geconvergeerd. **Smart-capture** is de kant-en-klare oplossing: het peilt de belichting van elke camera, wacht tot de array stabiel is over een venster, en activeert vervolgens de opname. Het is gelijkwaardig aan de GUI: de „smart“-opnameknop van de desktop-app roept hetzelfde backend-eindpunt aan.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Wanneer je via `ChlorosProject` werkt (volgende paragraaf), krijg je meer instelmogelijkheden:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Het smart-AE-beleid is standaard conservatief. Maak `exposure_tolerance_pct` strenger voor veeleisend radiometrisch werk; maak het ruimer voor snel veranderende scènes waarbij je gewoon ‘goed genoeg’ wilt.

---

## DAQ-sensorsessies

Permanente backend-pool voor spectrale sensoren (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Spiegelt het camera-oppervlak: smart-detect, hergebruik van de pool, idempotente koppeling.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioriteit: Ethernet → BLE → USB. Geef een willekeurige expliciete hint door om het transport vast te leggen.

### Vastgelegd transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` benoemt één sensor, en alleen die sensor wordt ooit geopend. Deze wordt vergeleken met de reeds geopende sensoren; vervolgens wordt de hostnaam of het IP-adres van een DAQ-E (een DAQ-E die de netwerkscan alleen via het IP-adres ziet, wordt op alleen-lezenbasis gevraagd naar zijn naam, MAC-adres en serienummer, zodat elk van deze apparaten het ook kan vinden), de BLE-MAC van een DAQ-M en de id die een DAQ-U of DAQ-M rapporteert zodra deze is geopend (elk ander apparaat dat ter controle is geopend, wordt weer gesloten). Als het niet wordt gevonden, genereert `connect_daq_sensor` een `ChlorosConnectError` met een overzicht van wat er is gevonden; inclusief een adres (`port=` / `mac=` / `eth_host=`) dat een andere sensor bevat, wordt er ook een foutmelding gegenereerd. Het valt nooit terug op de sensor die zichtbaar is — daar is de aanroep met de zero-flag voor bedoeld.</id></sn></id></sn>

### `connect_daq_sensor()`-handtekening

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Een sensor die al in de pool zit, wordt teruggegeven zoals hij is (geleend): `integration_time`, `frame_avg` en `enable_ae` worden er niet op toegepast. Koppel deze eerst los om hem met uw instellingen te openen. `cap_id` vormt hierop een uitzondering: op een geleende sensor wordt deze ingesteld met `set_cap()`. Indien weggelaten, geldt de standaardinstelling van de backend (zie de opmerking onder de methodentabel).

### `DAQSensorSession`-methoden

| Methode | Beschrijving |
| --- | --- |
| `status(timeout=10.0)` | Overzicht van pool-items (streaming-/opnamestatus, golflengtebereik, kalibratiesha, integratietijd, frame_avg, AE-status). Voor een DAQ-E waarvan de stream is gestopt: `reconnect` (wanneer de automatische herverbinding van de backend actief is: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` terwijl deze streamt), `stream_lost_s`, `reconnects` en `last_reconnect`, en `record_gaps` voor de opname die bezig is; `rx_stream_fault` geeft de status ‘opnieuw verbinden’ aan totdat het eerste spectrum binnenkomt. |
| `latest(*, n=1, timeout=10.0)` | Geef de N meest recente spectrumframes weer (elk `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Streaming hervatten / pauzeren (handle blijft open). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Begin met het opnemen van een .daq-bestand. Geeft het bestandspad terug. Wordt geweigerd voor DAQ-U/M zonder een AWS-kalibratiebundel (DAQ-E is hiervan vrijgesteld). |
| `record_stop()` | Stoppen met opnemen. Geeft `{path, rows, warning, gaps}` terug; `warning` is `None` bij een foutloze opname, anders de diagnose van de backend (bijvoorbeeld een leeg bestand of een gat in de stream), die ook naar stderr wordt afgedrukt. `gaps` geeft een overzicht van elk segment waarin de stream niets leverde: `start_utc`, `end_utc` (`None`: nog steeds verloren bij het stoppen), `duration_s`, `reconnected`, `marked_in_file`. Bij een continue opname is deze leeg. |
| `set_cap(cap_id, timeout=15.0)` | Stel het cap-correctieprofiel in dat vanaf de volgende meting wordt toegepast: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, of `"none"` (of `None`) voor een onbeklede sensor. Geeft hetoverzicht, dat door `cap_id` wordt teruggelezen. Een DAQ-E-S houdt `as_recorded` op de gevraagde waarde (de diffuser is ingebouwd). Geeft `ChlorosConnectError` weer voor een profiel dat niet kan worden geladen. CLI-opdracht: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Alleen DAQ-E: schakel PTP weer in voor een apparaat waarvan PTP is uitgeschakeld (`status()["rx_ptp_enabled"]` is `False`; de instelling wordt op het apparaat opgeslagen, dus het blijft uitgeschakeld totdat het weer wordt ingeschakeld). Geeft de vernieuwde status terug. Genereert `ChlorosConnectError` voor een DAQ-U / DAQ-M, of wanneer het apparaat niet bevestigt. Chloros schakelt een apparaat nooit uit zichzelf weer in, omdat ‘uit’ opzettelijk kan zijn. CLI-code: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Alleen DAQ-E: de eigen instellingen van het apparaat. `firmware` (`running`, `bundled`, `update_available`), `ip` en `ip_mode` (`dhcp` / `linklocal` / `static`: wat er nu draait), `network` (de opgeslagen instelling: `mode`, plus `ip` / `mask` / `gateway` in statische toestand), `network_change_pending` (geldt totdat het apparaat de opgeslagen instelling uitvoert, een instelling waarin `set_network(..., reboot=False)` is opgenomen), `auto_stream`, `ptp`. `firmware["update_available"]` is `None` wanneer deversie niet kon worden gelezen. Terwijl de stream van het apparaat deze computer niet bereikt, wordt deze bij een nieuwe verbinding wel gelezen en voegt het dict `stream_silent` (`True`), `stream_fault` en `reconnect` toe; een apparaat dat niet reageert, veroorzaakt een foutmelding. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E alleen: DHCP (`"auto"`) of een statisch adres. Het apparaat wordt opnieuw opgestart om de instelling toe te passen en opnieuw verbinding te maken; deze sessie blijft werken en `address` volgt het apparaat. `reboot=False` slaat de instelling alleen op. Zie [DAQ-E-apparaatinstellingen](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Alleen DAQ-E: streamen bij het opstarten zonder aangesloten software (opgeslagen op het apparaat). Geeft de bevestigde instelling weer. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Alleen DAQ-E: start het apparaat opnieuw op en sluit het opnieuw aan met de instellingen die het eerder had. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Alleen DAQ-E: flash de firmware die bij deze Chloros-release is meegeleverd naar een apparaat waarop een oudere versie draait, en sluit het vervolgens opnieuw aan (duurt ongeveer een minuut; schakel het apparaat niet uit). Geeft `updated`, `fw_before`, `fw_after`, `bundled` en een `warning` wanneer het apparaat traag reageerde of terugkwam op een andere versie; `updated` is `False` voor een apparaat dat al de actuele versie heeft. Genereert `ChlorosConnectError` voor een eenheid waarvan de versie niet kan worden gelezen. CLI-opdracht: `pool-update-firmware`. |
| `disconnect()` | Vrijgeven uit de pool. Geen-op voor gekoppelde handles die niet in eigendom zijn. Op een sensor die door deze sessie is geopend, is dit `force_disconnect()`, en genereert het een `ChlorosConnectError`-fout wanneer het vrijgeven niet wordt bevestigd (het `with`-blok geeft nooit een foutmelding). |
| `force_disconnect(timeout=30.0)` | De sensor vrijgeven, ongeacht wie deze heeft geopend — bijvoorbeeld een sensor die door een gecrasht script in de pool is achtergelaten, en die een nieuw script alleen bereikt als een gekoppelde handle waarvan `disconnect()` niets doet. Het beëindigen van de sensor van een andere client stopt elke opname die deze aan het maken is. Keert terug zodra de backend het vrijgeven bevestigt, of wanneer er niets te vrijgeven valt (de sensor bevindt zich niet in de pool, of er draait geen backend); een vrijgave die is verlopen of die door de backend is geweigerd, genereert `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Attributen: `sensor_id`, `model`, `transport`, `address` (volgt op een DAQ-E die door `set_network()` opnieuw is geadresseerd), `info` en `backend_url`.

> **Geef de dop een naam.** DAQ-U, DAQ-M en DAQ-E worden zonder condensator geleverd, en Chloros kan niet zien of er een is gemonteerd: een sensor die zonder `cap_id` is aangesloten, krijgt `sunshine_cosine`, een aanname die volgens de opnames is gemaakt (`cap_id_source=auto_default`). Een onbeschermde DAQ-U geeft onder die correctie een waarde weer die ongeveer 30x te hoog is; geef daarom `cap_id="none"` door voor een onbeschermde sensor (of `set_cap("none")`), en het profiel van de kap die je anders hebt gemonteerd.

### DAQ-E-S — vaste diffusor en kantelsensor

De **DAQ-E-S** is een DAQ-E met twee verschillen die van belang zijn voor de software. Al het andere — detectie, verbinding, PTP, opname, de onderstaande apparaatinstellingen — werkt precies zoals bij een DAQ-E.**De diffusor is vast gemonteerd, dus er is geen correctie voor de kap nodig.** De diffuser in Sunshine-stijl is ingebouwd en zat al op het apparaat toen de fabriekskalibratie werd gemeten, zodat die kalibratie al de werkelijke neerwaartse stralingsintensiteit weergeeft. Chloros stelt zijn cap in op `as_recorded` (geen profiel), ongeacht wat wordt gevraagd: een cap die naar het apparaat is vernoemd (`sunshine_cosine`, `fov_*`, `none`) wordt genegeerd met een melding, en de opnames slaan `cap_id_source=model` op, waardoor de waarschuwing voor een veronderstelde cap nooit wordt geactiveerd. Pas ook niet handmatig een limiet toe op de gegevens ervan: `sunshine_cosine` zou dan ongeveer 11 keer te hoge waarden aangeven.

**Het apparaat is uitgerust met een 3-assige versnellingsmeter.** De cosinusrespons van de zonnestraalverspreider is alleen geldig wanneer deze horizontaal staat; een gekantelde sensor rapporteert dus een verkeerde stralingsintensiteit — en de reflectie neemt deze fout over — zonder dat er andere symptomen optreden. Elke DAQ-E-S-meting bevat daarom de stand van het apparaat:

| Veld | Betekenis |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Hoeken in graden. `None` wanneer de firmware de hoek als ongedefinieerd markeert (vrije val, een dode as). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Versnelling in milli-g. `mag_mg` ver van ~1000 betekent dat het apparaat in beweging was en dat de hoeken voor dat monster geen betekenis hebben. |
| `cal_applied` | Ingeschakeld: de assen en hoeken liggen in het referentiekader van de diffuser (de kalibratie van de montage van het apparaat is toegepast). Uitgeschakeld: onbewerkt referentiekader van de printplaat, dat een kanteling tot ~2,3° kan weergeven die niet reëel is. |
| `sample_age_ms` | Leeftijd van de versnellingsmetermeting op het moment dat het spectrum werd verzonden. |
| `temp_raw` | **Trend** van de niet-gekalibreerde chip-temperatuur — alleen verschillen, nooit een thermometerwaarde. |

`DAQSensorSession.latest()` geeft deze weer onder de `imu`-sleutel van elk frame (`None` op een DAQ-U, DAQ-M of gewone DAQ-E). Een `.daq` die is geregistreerd vanaf een DAQ-E-S slaat dezelfde waarden op in de kolommen `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) naast elk spectrum, zodat een lezer een wolk kan onderscheiden van een omgestoten statief. Bij alle andere modellen zijn die kolommen leeg.

De houdingsstream rapporteert over zijn eigen velden in `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), gescheiden van de spectrale velden, zodat een storing in de versnellingsmeter nooit wordt geïnterpreteerd als een gat in de opname. `discover_daq_sensors()` en de `model` van de sessie vermelden `DAQ-E-S`.

### DAQ-E-apparaatinstellingen

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Er moet een statisch adres aanwezig zijn op het netwerk van deze computer.** `set_network` genereert een foutmelding `ChlorosConnectError` voor een apparaat waarvan het subnet het adres van deze computer niet bevatadres op het netwerk van de sensor: na de herstart kan niets de sensor bereiken om de instelling terug te zetten. Geef `force=True` alleen door wanneer deze computer een nieuw adres krijgt of naar het nieuwe subnet wordt gerouteerd.
- **`set_network`, `reboot` en `update_firmware` starten de sensor opnieuw op** en genereren `ChlorosConnectError` terwijl deze gegevens registreert, en wanneer deze niet binnen `comeback_timeout` seconden terugkomt. De sensor-ID en de `daq-e-<id>.local` hostnaam veranderen nooit.
- **Een opname die vlak voor de herstart is gestart, wordt hierdoor afgesloten en van een naam voorzien.** Wanneer de sensor weer bereikbaar is, bevat het geretourneerde dict `recording_closed` (het pad naar het bestand) en een `warning` waarin dit wordt vermeld. Wanneer de sensor niet terugkomt of de firmware-overdracht mislukt, geeft de `ChlorosConnectError` dit in plaats daarvan aan: het bericht eindigt met de waarschuwing, en `e.recording_closed` / `e.warning` worden ingesteld (beide `None` wanneer er geen opname was afgesloten). Start de opname opnieuw zodra de sensor weer is aangesloten.
- **Alleen de sensor zelf wordt opnieuw aangesloten.** Na de herstart moet het apparaat reageren op de plek waar het zich bevond -- en na `set_network("auto")` ook onder zijn `.local`-naam -- met zijn naam, hostnaam en model, via de netwerkadapter (MAC) die het had; `address` wordt het IP-adres dat heeft gereageerd. Een reactie onder de naam telt alleen met dat MAC-adres, dus vanaf een computer die de sensor via een router bereikt, of vanuit het standaard NAT-netwerk van WSL2, wordt een sensor die door DHCP is verplaatst niet opnieuw verbonden (`ChlorosConnectError`; maak opnieuw verbinding via het nieuwe adres).
- **Voor een firmware-update moet de sensor opnieuw verbinding maken.** `update_firmware()` nodigt het apparaat uit, en het apparaat opent een TCP-verbinding terug naar deze computer voor de image, op een poort die het besturingssysteem kiest. De backend luistert op de adapter die het apparaat bereikt (op elke adapter, alleen wanneer niet kan worden vastgesteld welke) en bedient alleen dat apparaat: elke andere verbinding wordt geweigerd (`[DAQ-FW] … refused a firmware connection from <ip>` in de uitvoer van de backend) terwijl deze blijft wachten. Een firewall die de verbinding blokkeert, zorgt ervoor dat de update mislukt met `device never connected back on TCP <port>`. Het Windows-installatieprogramma voegt hiervoor de regel toe (zie [Stap 1](#windows-exe)); op een andere firewall moet inkomend TCP-verkeer naar de backend vanuit het subnet van de sensor worden toegestaan.
- **Gebruik onder WSL2 'mirrored networking'** (`networkingMode=mirrored` in `.wslconfig`): in de standaard NAT-modus wordt elk statisch adres geweigerd (het eigen adres van WSL bevindt zich niet op het netwerk van de sensor), de stream bereikt WSL niet en een firmware-update kan niet worden voltooid.
- De GUI behoudt zijn eigen verbinding met een sensor: een herstart via een script beëindigt de sessie van de GUI met de sensor. De GUI heeft dezelfde instellingen (tabblad Lichtsensoren, de instellingen van de sensor, Netwerk).

### Detectie — een adres vinden om verbinding mee te maken

`discover_daq_sensors()` scant USB / BLE / ETH op sensoren die je *zou kunnen* openen. Het is de DAQ-tegenhanger van `discover_lattice_cameras()`, en de enige manier om het **BLE-MAC-adres van een DAQ-M** te verkrijgen — een DAQ-E heeft een hostnaam en een DAQ-U een COM-poort, maar een MAC-adres staat noch op het apparaat vermeld, noch wordt het door het besturingssysteem weergegeven.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Veld | Beschrijving |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-poort / BLE MAC / hostnaam — doorgeven aan `connect_daq_sensor` als `port=` / `mac=` / `eth_host=`. |
| `display` | Voor mensen leesbaar label. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, of `None`. Een Ethernet-sensor is `DAQ-E-S` of `DAQ-E`, aangezien deze zichzelf rapporteert wanneer daarom wordt gevraagd, alleen-lezen, tijdens de scan (`DAQ-E` wanneer hij niet op tijd antwoordt). Een USB-seriële poort geeft `DAQ-U` weer wanneer de bridge-chip ervan een type is dat door een DAQ-U wordt gebruikt (FTDI, CH340, CP210x, Prolific) — een aanwijzing uit de beschrijving van de poort, geen probe — en `None` voor elk ander USB-apparaat, dat nog steeds in de lijst staat, zodat je het kunt doorgeven als `port=`. Een seriële poort zonder USB-leveranciers-ID en zonder een dergelijke bridge-chip (een UART op het moederbord, zoals `/dev/ttyS*` in Linux, een seriële Bluetooth-verbinding, Intel AMT) kan geen DAQ-U zijn en staat niet in de lijst. |
| `extra` | Details per transport (BLE-advertentienaam; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Lege waarden worden weggelaten. |

| Parameter | Standaard | Beschrijving |
| --- | --- | --- |
| `transports` | alle drie | Reeks (of CSV-string) die de scan beperkt. Het loont de moeite deze door te geven als je weet wat je wilt — BLE is het trage onderdeel. |
| `scan_timeout` | 5 | Scanvenster per transport in seconden; de backend beperkt dit tot 1–20. |
| `timeout` | 60,0 | HTTP-limiet voor de gehele aanroep (zoals elders in de SDK). |
| `auto_start_backend` | `True` | Start een lokale backend als er nog geen actief is. Wordt nooit gestart voor een externe `backend_url`. |

> **Sensoren die al in de pool openstaan, worden niet weergegeven.** Een verbonden BLE-randapparaat stopt met adverteren en een open COM-poort kan niet worden gescand, dus de ontdekkingslijst toont wat *beschikbaar is om verbinding mee te maken*. Een leeg resultaat direct nadat je iets hebt aangesloten, is te verwachten — gebruik `list_daq_sensors()` voor wat je al in bezit hebt. Transporten waarvan de scan niet kan worden uitgevoerd (geen bleak / zeroconf geïnstalleerd) worden overgeslagen in plaats van een foutmelding te geven, zodat een machine zonder Bluetooth nog steeds zijn USB- en ETH-antwoorden krijgt.

### Lijst

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Welke DAQ voedt welke camera — `daq_bindings()`

De reflectie van een opname en het `.daq`-bestand waarin deze wordt opgeslagen, maken gebruik van de DAQ die aan die camera of array is gekoppeld, of anders de meest recente DAQ die de backend heeft. De selectie van de lichtsensor in de GUI en `chloros-cli daq pool-bind` stellen dit in; evenals:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` retourneert `{"cameras": {serial: b}, "arrays": {array_id: b}}`: elke `b` bevat `binding` (de ingestelde waarde), `source` (`project`, uit de `sensors.json` of `runtime` van het project), `known` / `known_as` (of de camera- of array-sleutel een naam heeft die Chloros kent, en hoe: `connected`, `project` — de `cameras.json` — of, voor een camera, `seen`: eerder op deze computer geopend; `known: False` voor een sleutel die niets benoemt) en `resolved` (de poolsensor waaraan het nu is gekoppeld: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Een runtime-koppeling heeft voorrang op de project-s voor de sessie; het projectbestand wordt niet gewijzigd. Het loskoppelen van een camera, een array of een DAQ verbreekt elke koppeling met die hardware, ongeacht wie deze heeft ingesteld (de leden van een array worden gekoppeld aan de array zelf).

`set_daq_bindings()` koppelt alleen een camera die is aangesloten, in het geopende project staat of eerder op deze computer is geopend, en een array die is aangesloten of in het geopende project staat (of een toets die al is gekoppeld). Elke andere toets genereert een `ChlorosConnectError` met de naam (`400`, `code`, `UNKNOWN_BINDING_TARGET`, met de toetsen `unknown`), en **niets** in die aanroep wordt gekoppeld. `None` wist alle sleutels.

### Omgevings- + objectreflectie — `light_sensor_reflectance()`

Twee samengevoegde DAQ’s, één die naar de hemel kijkt (omgeving) en één naar beneden naar het doel (object), geven de reflectie en vegetatie-indexen van het doel weer op basis van hun laatste metingen: het ‘Ambient + Object’-paar in de GUI en `chloros-cli daq reflectance` zonder de sensoren een tweede keer te openen. Beide moeten zijn aangesloten en gegevens doorgeven.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Banden.** De reflectantie van elke band is de band-geïntegreerde bestralingssterkte ten opzichte van die van de omgeving. `daq_band_presets(filter_code=None)` geeft de bandensets weer: één per LATTICE M3C-filter (`RGN`, `OCN`, `NGB`, het eigen bandmodel van de camera, dus een DAQ-NDVI en een camera-NDVI zijn dezelfde grootheid) en `GENERIC` breedband. De `suggested` is de bijbehorende voorinstelling `filter_code`, anders het filter van de eerste aangesloten camera (`suggested_from` geeft aan welke).
- **Geeft terug** `bands` (per band `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (de ingebouwde-ins), `custom` (elk met `value`, of `valid: False` en de `error`), `invalid_bands`, `reflectance_type`, `preset` en de twee meetwaarden `timestamps` (`ts_ns`). Een bereik dat de omgevingssensor niet kon meten (buiten het meetbereik of op het ruisniveau) is ongeldig, met vermelding van de reden, en elke index die hierop is gebaseerd wordt weggelaten; deze wordt nooit berekend ten opzichte van nul.
- **Schijnbaar of relatief.** De schaalafhankelijke indices (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) hebben een absolute verhouding nodig, dus ze worden alleen weergegeven wanneer beide sensoren vermogensgekalibreerd zijn (elk heeft een `calibration_bundle_sha`): `"Apparent Reflectance"`. Anders is het resultaat `"Relative Reflectance"`, met alleen de schaalinvariante indices.
- **Twee verschillende golflengterasters** worden vergeleken op hun overlapping: de punten van de omgevingssensor binnen het bereik van het object, en het spectrum van het object dat daarop is geïnterpoleerd.
- **Twee sensoren, nooit één.** Als één sensor als beide wordt aangeduid (met hetzelfde ID, in elke spelling die de pool accepteert), leidt dit tot `ChlorosConnectError`: de meting van een sensor over zichzelf is 1,0 op elke band, wat niets meet. De backend weigert dit ook (`POST /api/daq/indices` beantwoordt `400`, `code` `SAME_SENSOR`, terwijl de `ambient_sensor_id` en `object_sensor_id` die de SDK verstuurt één samengevoegde sensor zijn, bijvoorbeeld de ID en de poort).

### Gedeeld gebruik met de CLI en andere scripts

De DAQ-sensorpool van de backend wordt gedeeld door elke CLI- en SDK-client op de machine. Als een sensor al in de pool zit — geopend door `chloros-cli daq pool-connect` of door een ander script — levert het aanroepen van `connect_daq_sensor(port="COM3")` vanuit Python een geleende handle op: de `disconnect()` is een no-op, dus je script haalt de sensor niet weg onder de client die deze bij het afsluiten van de scope heeft geopend. Hetzelfde geldt voor `sensor_id=` (getypt met `:` of `-`; beide komen overeen), voor een DAQ-E die bij het openen op zijn IP-adres werd aangeduid terwijl hij via hostnaam werd geopend (of andersom), en voor de smart-detect met de nul-vlag wanneer de backend antwoordt met een sensor die hij al in bezit had — een `with`-blok geeft alleen een sensor vrij die het zelf heeft geopend.

De Chloros-GUI maakt **geen** deel uit van deze pool: het DAQ-paneel ervan onderhoudt zijn eigen verbinding met de sensor, dus `connect_daq_sensor` leent nooit een sensor die de GUI heeft geopend. Gebruik niet tegelijkertijd dezelfde sensor vanuit de GUI en vanuit een script. Een DAQ-U of DAQ-M die de GUI in bezit heeft, kan normaal gesproken gewoon niet worden geopend, maar een DAQ-E accepteert een tweede sessie: de connect-opdracht van het script wijzigt de acquisitie-instellingen van het apparaat onder de GUI, en de `disconnect()` stopt de stream van het apparaat, waardoor de liveweergave van de GUI en eventuele opnames worden beëindigd. Sluit de sensor eerst in de GUI.

### PTP Grandmaster — `time_sync_status()`

De backend draait de PTP-grandmaster waarvan LATTICE-camera’s en DAQ-E-units hun tijdstempels betrekken. Deze start op verzoek: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` en `connect_all()` starten het als het niet draait — inclusief een verbinding die vervolgens mislukt, zoals `connect_all()` die een DAQ-E probeert te benaderen die er niet is — en dat geldt ook voor `test_array_sync()` en elke aanroep die camera’s opent voor één commando (`capture_cameras_transient()`, `start_multi_camera_stream()`, de uitlijningsaanroepen, `capture_live_index()`).

- **Waar het zich meldt.** Op de bekabelde netwerkadapters die naar de camera’s zijn gericht (inclusief een `169.254.x.x` link-local-adres), anders op de adapter van de standaardroute als die bekabeld is. Nooit op wifi of een virtuele adapter (Tailscale, WSL): daar zit geen camera of DAQ-E achter. Als er geen dergelijke adapter actief is, blijft het systeem op niets adverteren (`health` `NO_SOCKETS`) en koppelt het zich aan de adapter zodra de volgende camera of DAQ verbinding maakt.
- **Na een verbroken verbinding.** Wanneer de camera-adapter de verbinding verliest en deze weer terugkrijgt (een kabel die losraakt, een vervangen switch, een USB-adapter die opnieuw wordt geïdentificeerd), bouwt de grandmaster zijn sockets zelf opnieuw op, zonder opnieuw op te starten, en controleert hij elke aangesloten DAQ-E opnieuw, waarbij hij degene die in de knoop was geraakt opnieuw activeert. Hij bouwt de verbindingen ook opnieuw op wanneer alle camera’s en DAQ-E’s die hij bediende gedurende 15 seconden stil blijven.
- **Bereikt hij de kabel?** Lees `time_sync_status()` (`GET /api/time-sync/status`). `state` geeft alleen aan dat de grandmaster-thread draait; `health` geeft aan of de pakketten worden verzonden: `OK` (geen storing waargenomen), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (sommige adapters werken niet) of `PEERS_SILENT` (de camera's en DAQ-E's die het bediende, reageerden niet meer), met een zin in `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` tellen alleen de berichten die zijn verzonden; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` en `last_delay_req_age_s` beschrijven de berichten die niet zijn verzonden, en `rebinds` / `last_rebind` de herbouwde berichten. `sync_sends_slow` telt de Syncs waarvan de verzending langer duurde dan 200 us en `sync_send_max_us` is de langste; de oortijd van elke Follow_Up telt maximaal 200 us van de verzending mee. De grandmaster draait in een eigen proces (`own_process`, `process_pid`), met hoge prioriteit op Windows (`process_priority`), zodat een drukke backend de tijdstempels niet kan vertragen en de klok van elke camera in één keer kan bijstellen; als dat proces niet kan worden gestart, draait het zoals voorheen binnen de backend (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Functie | Wat het doet |
| --- | --- |
| `time_sync_status()` | De status, gezondheid, klokidentiteit, prioriteiten en berichtentellers van de grandmaster (zie hierboven). `chloros-cli time-sync status`. |
| `time_sync_peers()` | De PTP-slaves die hij waarneemt (camera’s, DAQ-E’s): `peers`, elk `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | De `PtpStatus` van elke aangesloten arraycamera, `PtpOffsetFromMaster` en `PtpMeanPathDelay` (een camera waarvan de firmware geen PTP-knooppunten bevat, heeft alleen een serienummer). `time-sync cameras`. |
| `restart_time_sync()` | Stop de grandmaster en start deze opnieuw op; de PTP van elke DAQ-E in de pool wordt daarna opnieuw gecontroleerd (`daq_ptp_rechecked`). Genereert `ChlorosConnectError` wanneer deze niet terugkomt. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | De PTP-prioriteiten (0-255; de laagste wint de verkiezing tot best-master), voor een netwerk met een andere PTP-master. Moet actief zijn. `time-sync set-priority`. |

Elk neemt `backend_url=` en `timeout=` en start een lokale backend die nog niet draait.

---

## Camera-hulpprogramma’s

Wat `chloros-cli lattice` doet naast verbinden en vastleggen, zijn eenvoudige functies in `chloros_sdk` (module `chloros_sdk.camera_utils`). Elk roept de backend-route aan die het CLI-commando gebruikt, met de regels van de CLI, zodat de twee zich op dezelfde manier gedragen. Ze nemen een **serienummer** van de camera als argument, geen sessie, en dezelfde `backend_url=` / `timeout=`-trefwoorden als de smart-connect-aanroepen; een backend die niet draait, wordt gestart, net zoals bij `connect_camera()`. Bij een weigering wordt `ChlorosConnectError` gegenereerd met de reden van de backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Functie | Wat het doet |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Het kleurprofiel van het beeldscherm: `raw`, `linear`, `natural`, `enhanced` of `custom_temp`, en de kleurtemperatuur ervan (begrensd op 2000-10000 K; wordt uitgelezen wanneer het profiel `custom_temp` is). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Weergaveverzadiging en contrast, elk 0-200 % (100 = ongewijzigd); `reset=True` zet beide terug naar 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | De helderste, witste pixels van de schermweergave verlagen: -100..0 (afgekapt naar nul, vervolgens begrensd; 0 = uit). |
| `camera_white_balance_once(serial, settle_s=None)` | Eenmalige witbalans: de camera streamt kort, convergeert en retourneert `red_ratio` / `blue_ratio` (`ratios_read` False wanneer deze niet kan worden gerapporteerd). De camera behoudt de berekende witbalans; alle andere instellingen die door de open-methode zijn geschreven, worden teruggezet zoals ze zijn aangetroffen. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (continu) of `"off"`; of vaste `red` / `blue`-balansverhoudingen, waarbij de automatische modus als eerste wordt uitgeschakeld. Een verhouding die de camera weigert, wordt aangeduid onder `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | De camera-voorinstellingen, en bij het toepassen van een daarvan: de camera behoudt de instellingen die de voorinstelling aangeeft (het antwoord is `settings`, zoals de camera ze daarna uitleest, bijv. de framesnelheid waarmee hij draait; `None` voor een veld dat de voorinstelling niet benoemt), en elke andere instelling die het bestand bevat, wordt teruggezet zoals aangetroffen. `"default"` zet de camera terug naar de fabrieksinstellingen (de fabrieksinstellingen met vrijlopende automatische belichting, de limiet voor de linkdoorvoer en defectcorrectie uitgeschakeld, de gekalibreerde witbalans van een RGB-camera behouden), niet opgeslagen in het geheugen: het antwoord bevat `factory_defaults: True` en elk veld van `settings`. In `camera_presets()` is `default` gelijk aan `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | De filtermodellen en de details van één model (banden, conditienummer van de spectrale respons). Het filtermodel van een camera wordt uit de modelstring gelezen; het wordt niet handmatig ingesteld. |
| `camera_firmware_status(serial=None)` | De firmware van elke cameraheeft de volgende firmware: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` en `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Flash de firmware van deze Chloros-build naar één camera: fabrieksreset, flash, herstart, de Chloros-basisinstellingen worden herschreven; de kalibratie van de camera blijft behouden. Enkele minuten; de camera moet ingeschakeld blijven. `force=True` flasht een bestaande camera opnieuw. |
| `camera_power_probe(serial)` | De stroom- en thermische knooppunten van de camera en de temperatuur ervan. |
| `sleep_camera(serial)` / `wake_camera(serial)` | De energiezuinige stand-bymodus van de camera (de camera blijft in slaapstand totdat hij wordt gewekt: de limiet voor de linkdoorvoer is ingeschakeld (`DeviceLinkThroughputLimitMode` `On`) op het minimum, de energiebesparingsmodus is ingeschakeld indien de camera daarover beschikt), met de temperatuur ervoor en erna; bij het ontwaken wordt de verbinding teruggezet zoals deze was toen de camera in slaapstand ging (de limiet, de modus en de pakketvertraging `GevSCPD`, die de camera op 0 houdt zolang de limiet is ingeschakeld; dit wordt op deze computer bijgehouden, zodat een latere backend het ook terugvindt bij het ontwaken), of, voor een camera die deze computer niet in de slaapstand heeft gezet maar die in slaap lijkt te zijn (de limiet is ingeschakeld op de ondergrens), waarbij de limiet wordt uitgeschakeld en een `GevSCPD` op 0 wordt gelaten bij 80 (de fabrieksinstellingen); een limiet die boven de ondergrens is ingesteld, blijft ongewijzigd. Alleen de stroomstatus wordt behouden; alle andere instellingen die door hun open-schrijfbewerking zijn gewijzigd, worden teruggezet naar de oorspronkelijke waarde. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Leg frames vast totdat een MAPIR-kalibratiedoel in beeld is, en pas de coëfficiënten per kanaal aan op basis van de panelen: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | De Ethernet-adapters van deze computer voor GigE Vision (MTU, linksnelheid, jumboframes) en eventuele `issues`; `estimate=True` voegt de beeldsnelheid en bandbreedte toe die `cameras`-camera's zouden krijgen. |
| `fix_network_settings()` | Pas de GigE Vision-adapterinstellingen toe op decamera-adapter. Dit wijzigt de instellingen van de adapter en werkt alleen als Chloros als beheerder wordt uitgevoerd; anders wordt er een foutmelding weergegeven met het logboek van de gevonden gegevens. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Voer vignetteringscorrectie uit op de afbeeldingen in een map, elk met de flat-field-kaart van de eigen lens: `processed`, `lens_models` (de lens waarvoor elk bestand is gecorrigeerd), `skipped` (met redenen). Zonder `lens_model` wordt de lens van elk bestand afgeleid uit het EXIF-model; een `lens_model` is van toepassing op bestanden waarvan het Model geen camera noemt, en een bestand waarvan het Model hiermee in tegenspraak is, wordt overgeslagen. Alleen bestanden. |
| `read_dls(mac, filter=None, camera_serial=None)` | Eén meting van een DLS (Bluetooth-sensor voor neerwaarts vallend licht): spectrum, stralingsintensiteit per band en bandmagnitudes via de eigen respons van de camera wanneer `camera_serial` wordt opgegeven. `discover_daq_sensors(transports=["ble"])` bepaalt het MAC-adres. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | De hardware van de host voor camera's en de aanbevolen configuratie (`report_text`), en de CPU / GPU / PyTorch-runtime van het systeem. |

- **Sommige hebben de camera voor zichzelf nodig.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` en `update_camera_firmware()` openen de camera. Zolang Chloros deze bezet houdt (verbonden in de app, met `connect_camera()`, als lid van een array, of op dit moment in gebruik door een ander commando, zoals een arraytest of een opname waarbij de camera’s worden geopend), weigert de backend dit met HTTP-statuscode 409 `CAMERA_HELD` (zowel in `code` als in `error_code`), waarbij wordt aangegeven hoe het kan worden vrijgegeven. De kleurinstellingen, highlights, `camera_power_probe()` en de uitleesopdrachten werken op een aangesloten camera.
- **Kleur, highlights en witbalans zijn bedoeld voor RGB-camera’s.** Op een single-band mono (M3M)-camera veranderen de setters niets en retourneren ze `{"skipped": True, "reason": ...}`, net als de CLI. Het zijn weergave-instellingen: de liveweergave en opnames op weergaveniveau, nooit straling of reflectie.

---

## Transiënte arrays, uitlijningsprofielen en indices

Wat `chloros-cli lattice` doet zonder een permanente verbinding, net als de functies in `chloros_sdk` (module `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` en `index`. Elk commando verstuurt het verzoek dat het CLI-commando verstuurt, met de standaardinstellingen van de CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Ze openen de camera’s zelf.** Elk commando opent zijn camera’s, voert zijn taak uit en sluit ze weer, dus het heeft camera’s nodig die Chloros niet in bezit heeft (aangesloten in de app, met `connect_camera()`, of als array-element); de backend weigert met HTTP-statuscode 409 `CAMERA_HELD` en geeft aan hoe je er een kunt vrijgeven. Een tijdelijke opening duurt even (ongeveer 40 s voor een paar camera’s). Voor herhaalde opnames is is `connect_array()` één keer de snelste methode, en het werkt voor elk exporttype, terwijl een tijdelijke opname een debayered beeld opslaat. `calibrate_alignment_profile()` is de uitzondering: bij een aangesloten array van die camera’s lost het dit op vanuit de eigen frames van de array (`used_live_array`).
- **`serials`** zijn de camera’s die moeten worden geopend, waarbij de eerste de trigger activeert; `None` opent elke camera op het netwerk.
- **`settings`** is een dict met overschrijvingen van camera-instellingen, de velden die door de vlaggen van de CLI worden ingesteld: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` en `preset` (een camera-voorinstelling met deze naam). Een opgegeven belichting of versterking schakelt die automatische regeling uit, net zoals de vlaggen dat doen.
- **`gpu`**: `True` dwingt de GPU-warp af, `False` de CPU (`--gpu` / `--no-gpu`); `None` laat de backend kiezen.
- **`keep_settings`**/**`clear_settings`** (elke functie die camera’s opent, standaard `False`): aan het einde wordt elke camera teruggezet zoals deze werd aangetroffen; `keep_settings=True` slaat de instellingen van de run in plaats daarvan op in het eigen geheugen van de camera (vrijlopend), `clear_settings=True` zet dat geheugen terug naar de fabrieksinstellingen — zie [Camerageheugen](#camera-memory--keep_settings--clear_settings). Het antwoord (voor een taak, dat van `stop()`) bevat `camera_memory`.
- **Paden zijn die van de backend**: relatieve paden worden absoluut gemaakt ten opzichte van je werkdirectory wanneer de backend op deze machine draait. De functies voor het uitlijningsprofielbestand (`alignment_profile_info()`, `reorder_alignment_profile()`) lezen het bestand op deze computer, net zoals de CLI.

| Functie | Wat het doet |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Neemt continu beelden op van één camera totdat dit wordt gestopt. Geeft een `CliJob` terug. De frames van een `tiff`-run vormen het `raw`-bestand, een array dat de opnames van die camera bevat (één mozaïekvlak op deeigen waarden, ongecomprimeerd); `png` en `jpg` bevatten 8 bits, dus die runs zijn gedemosaïceerde frames om te bekijken. Een gebonden of gepoolde DAQ schrijft één `.daq` per frame in demap `daq/` (`daq_files` in de statistieken). Elk frame wordt één keer geschreven met de volledige metadata (model, opnametijd, lens, banden, kalibratie op een raw-frame); `stop()` keert terug zodra elk vastgelegd frame is weggeschreven (maximaal 25 s), en pas dan wordt de camera gesloten. Een camera die in de stand-by-modus is gelaten voor een hardware-trigger (`TriggerMode` `On`) wordt voor de opname ingesteld op ‘free-run’ (het `trigger` in het startantwoord geeft aan wat er is aangetroffen). Bij het stoppen wordt de camera teruggezet zoals hij werd aangetroffen — de triggerinstellingen en alle instellingen die tijdens de run zijn gewijzigd — tenzij `keep_settings=True`, dat de instellingen van de run opslaat in het eigen geheugen van de camera (vrije loop), of `clear_settings=True`, waarmee dat geheugen wordt gereset. Een run die geen frames opneemt, beëindigt zichzelf na 10 s met `stop_reason` `no_frames` en de reden in `last_error`, en de bijbehorende `stop()` geeft die reden weer als `error`. Frames die zijn weggevallen omdat de schrijvers het niet konden bijhouden, worden geteld, waarbij `drop_reason` aangeeft waarom. Voor een camera die Chloros vasthoudt, gebruik je `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Synchroniseerde framegroepen streamen bij `fps`, voor `count` groepen of totdat het wordt gestopt. Geeft een `CliJob` terug (de `start_reply` bevat `n_cams` en de wire `pacing`); de uiteindelijke statistieken tellen het aantal opgehaalde, geschreven en weggegooide groepen, de bestanden die de schrijver niet kon schrijven en de groepen waarvoor een camera niets heeft geleverd. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Eén gesynchroniseerde opname, één debayered beeld per camera: `results` (`filepath`, `serial`, `frame_id`) en `pacing`. Er ontstaat een melding dat er een camera ontbreekt, waarbij eerst wordt vermeld hoeveel bestanden zijn opgeslagen. `lattice multi-capture` zonder aangesloten array. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` gesynchroniseerde opnames bij een vaste belichting: `sync_groups`, de `master`, één rij `group_timestamps` per trigger (een camera die er één heeft gemist, ontbreekt in de betreffende rij) en `sync`: de offset van elke andere camera ten opzichte van de master, trigger per trigger (`offsets_us`: per camera `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), op ÉÉN klok (`clock_domain`: `ptp`, of `device` — PTP uitgeschakeld, de eigen uptime-klok van elke camera, dus de offset is het moment waarop deze is opgestart en alleen de jitter meet de synchronisatie). Een trigger waarvan de tijdstempels een combinatie zijn van de apparaatklok en PTP (PTP uitgeschakeld of nog aan het synchroniseren) wordt weggelaten en meegeteld (`groups_mixed_domain`), net als een trigger waarbij een PTP-klok nog steeds meer dan `agree_within_us` afwijkt van de master (`groups_clocks_disagree`); `notes` geven dit aan. `clock_offset_mean_us` / `clock_offset_jitter_us` zijn het grootste gemiddelde en de grootste jitter per camera. `success` is onwaar, samen met `error`, wanneer een trigger werd gemist, wanneer minder dan twee triggers elke camera op één klok hebben, of wanneer de klokken nooit overeenkomen (`sync.problems`); de rijen worden in beide gevallen geretourneerd. Het start de PTP-grandmaster als deze niet draait (`time_sync` geeft aan dat `grandmaster_running` en `grandmaster_started_by_this_test`), schakelt de PTP van elke camera in (zoals `connect_array()` doet), en wacht vóór de eerste trigger tot de PTP-klok van elke camera is gesynchroniseerd met de grandmaster en gestabiliseerd is: `PtpStatus` Slave op één grandmaster, binnen 100 us daarvan en stabiel gedurende 3 s, en elke camera op die manier vergrendeld, zonder onderbreking, gedurende 120 s — ongeveer 2 minuten na het inschakelen. Een klok die net is gesynchroniseerd, wijkt tot ~200 us af gedurende ongeveer 2 minuten nadat deze ‘Slave’ heeft gelezen, en de eigen `PtpOffsetFromMaster` van de camera geeft dit niet weer; na 120 s komen de camera’s binnen ~10 us van een gestabiliseerde klok. Een camera die nog steeds vergrendeld is sinds een eerdere test waarbij PTP aan bleef staan, zoals deze backend het zag, telt vanaf die vergrendeling, zodat een test direct na een andere niet opnieuw hoeft te wachten; een camera die door de test wordt ingeschakeld, of een camera die met PTP ingeschakeld is aangetroffen en waarvan deze backend nog niet heeft gezien dat deze is gesynchroniseerd (elders ingeschakeld, of vóór een herstart van de backend), wacht zijn 120 s af, en in een reeks van beide camera’s wachten de net gesynchroniseerde camera’s nog steeds op hun beurt. Een camera die de Slave-status verlaat, de 100 us overschrijdt of gedurende 3 metingen achter elkaar een andere grandmaster volgt (1,5 s) tijdens het wachten, begint opnieuw te tellen; één of twee van dergelijke metingen worden gelogd (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) en de telling gaat door, en een grandmaster-ID die niet kan worden gelezen, wordt als onbekend beschouwd, niet als een wijziging. Als de telling van alle camera’s tegelijkertijd (binnen een seconde) opnieuw begint, komt dat door de tijd van de grandmaster, niet door die van de camera’s: het logboek van de backend’logboek vermeldt `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` vermeldt het in `grandmaster_steps`, en een wachttijd die vervolgens afloopt, begint zijn foutmelding met `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. De limiet wordt hiervoor niet verlengd. `ptp_settle_s` (0-600; `0` voor geen afwachting) stelt de afwachtingstijd voor de run in; een andere waarde wordt geweigerd (HTTP-status 400). De wachttijd is begrensd op de afwikkelingstijd plus 90 s (standaard 210 s, minimaal 90 s), en de aanroep wacht op de backend `timeout` s: standaard 240 plus de afwikkelingstijd (360 s), net zoals bij `chloros-mcp`'s `test_array_sync`. `ptp_lock` heeft `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (hoe lang elke camera al vergrendeld en stabiel is), `found_settled` (de camera’s die al gedurende de stabilisatietijd vergrendeld waren toen ze werden gevonden) en, per camera, `status`, `offsets_ns`, `slave_s` (hoe lang deze al is vergrendeld; Geen: niet vergrendeld) en `remaining_s` (Geen wanneer de PTP van geen enkele camera zou worden ingeschakeld). Het logboek van de backend bevat elke 5 s een regel met de status van elke camera, hoe lang deze al is vergrendeld en hoe lang dit nog duurt. Elke camera wordt ingeschakeld met PTP uitgeschakeld; vanaf dat moment synchroniseren de klokken binnen ongeveer 10 s. Daarna wordt elke camera teruggezet in de toestand waarin de test hem aantrof — de PTP, de automatische belichting en de versterking worden tijdens de test op ‘Uit’ gehouden (`ExposureAuto` / `GainAuto`, en een handmatige `ExposureTime` / `Gain`), en de rest, tenzij `keep_settings=True` — dus een camera die achterblijft met PTP ingeschakeld (`PtpEnable=True`, of een testrun met `keep_settings=True`) verbleven vergrendeld, en zodra deze backend heeft gezien dat deze gedurende de stabilisatietijd vergrendeld was, wacht een latere test er niet meer op. Als de klokken niet stabiliseren, is `success` onwaar vóór elke trigger, en `error` geeft aan wat er moet gebeuren: een camera die nooit is vergrendeld (`not_slave`) betekent dat de grandmaster deze niet bereikt (`time_sync_status()`); een camera die steeds uitviel, wordt aangeduid met hoe lang deze vergrendeld was (`remaining_s`); klokken die nog steeds convergeren (`far`): controleer `time_sync_status()`, en om ze vóór de test vergrendeld te krijgen, schakel je PTP in en laat je het ingeschakeld (`PtpEnable=True` op elke camera, of `keep_settings=True`; de test zet de gevonden instellingen terug, dus het blijft aan), voer de test vervolgens uit. `lattice multi-test`. |
| `detect_array_master(line=None)` | De camera’s op de synchronisatielijn en een standaardmaster: `master_serial`, `slave_serials`, de `line` die zijn getest. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Wat een reeks van deze cameramodellen (productnamen, `"LATT-M3C-L87-FOCN"`) op het netwerk van deze computer terecht zou komen: `tier`, `achievable_fps_max`, het kabelbudget (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` en `fix_suggestions`. Standaard worden de modellen van de gedetecteerde camera’s weergegeven; er is geen camera geopend. Zie [`analyze_array_network()`](#network-analysis-helper) voor camera's die u hebt, met de instellingen die een verbinding zou gebruiken. `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Stel een uitlijningsprofiel op basis van livebeelden op en sla dit op. `method`: `orb`, `akaze`, `phase`, `checkerboard` (met `checkerboard="9x6"`, de binnenhoeken) of `manual` (met `control_points={serial: [[x, y], ...]}`, ook die van de referentiecamera). `model`: `translation`, `rigid`, `affine` of `homography`. De meer gedetailleerde opties zijn de vlaggen van de CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` is `{serial: {option: value}}`. `rms_threshold_px` weigert een slechtere oplossing op te slaan. Geeft `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (per camera `reprojection_error`, `num_matches`) en `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Eén uitgelijnd multibandbeeld via het profiel: `files` (het multiband-TIFF-bestand met zijn `bands` en grootte, plus één per band met `per_band` en één per camera met `per_camera`) en `vignette_corrected`. `bands` selecteert en `order` rangschikt de banden op naam (een lijst of een reeks met komma’s). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Stream-uitgelijnde multibandbeelden (dezelfde warp-opties als `capture_aligned_multiband()`), voor `count` of totdat het proces wordt gestopt. Geeft een `CliJob` terug; de uiteindelijke statistieken zijn `frames_written` en eventuele `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Een profielbestand dat wordt samengevat als `align-info`, geeft het volgende weer: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (per camera `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (per camera `band_names`, `wavelengths_nm`, `enabled`); `profile` is het volledige bestand. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Wijzig het profielbestand ter plekke: `order` zijn bandnamen (elk verplaatst de laag van zijn camera omhoog, in die volgorde), `enable` / `disable` zijn serienummers. Geeft het nieuwe `layer_order` terug. `lattice align-reorder`. |
| `index_catalog()` | De indexvoorinstellingen (`presets`: elk `name`, `channels`, `formula`) en de kleur `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Een index voor een uitgelijnd multiband-TIFF-bestand: een `preset` (`NDVI`, `NDRE`, ...) of een `formula` over de symbolen `channel_map` die naar banden verwijzen (`{"NIR": "NIR_850", "Red": 0}`, op naam of index). Een voorinstelling’symbool `channel_map` laat de ene band van het bestand weg waarvan de naam daarmee begint (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), benoemd door het bestand zelf (de bandlijst ervan, of de `Chloros:Band{R,G,B}Name` van een Chloros-opname of -product), dus NDVI-gegevens op een Chloros-reflectieopname hebben geen `channel_map` nodig; twee banden die bij één symbool horen, worden afgewezen, niet geraden. `capture_level` geeft aan wat de banden bevatten: `reflectance` of `radiance` maken de waarden kwantitatief; een niet-gekalibreerde stack (die van `align-apply`) levert kwalitatieve waarden op; `"unknown"` leest het uit het bestand, inclusief de `Chloros:ProcessingLevel` van een Chloros-bestand. Geeft `stats` terug (vorm, het 1-99 percentielbereik, het LUT-bereik, de gebruikte `capture_level`) en de `files` die naast de invoer is geschreven (`<input stem>_<preset>.tif`, `..._LUT.png`) of bij `output`, een pad: met een afbeeldingsextensie worden de bestanden benoemd op basis van de stam (`output="D:/out/ndvi_FRGN.png"` schrijft `ndvi_FRGN.tif` en `ndvi_FRGN_LUT.png`), zonder extensie is het een voorvoegsel (`D:/out/ndvi` -> `ndvi.tif`, `ndvi_LUT.png`); lees de paden vanaf `files`. Deze bevatten de cameragegevens van de invoer (merk / model / serienummer, GPS, lens, opnametijd in UTC, de kalibratie-ID maar nooit de raw-DN-kalibratie), `Chloros:ProcessingLevel=index`, en de index als hun enige band (`XMP-Camera:BandName`, zowel het TIFF- als het LUT-PNG-bestand). Alleen bestanden; geen camera. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Opnemen, kalibreren (`reflectance` of `radiance`; `raw` slaat dit over), uitlijnen en een index berekenen in één stap; `save_multiband=True` bewaart de gekalibreerde stapel, die `compute_index()` later kan overnemen. Geeft `stats` terug (met de `serials` waarvan de banden zijn gebruikt) en de `files`. Het opent `serials` (de eerste activeert de trigger), anders de camera’s van het profiel waarvan de `layer_order`-vermelding is ingeschakeld, waarbij de referentiecamera als eerste (wordt geopend zelfs als de banden ervan zijn uitgeschakeld); een camera waarvan de banden zijn uitgeschakeld, wordt nooit gekalibreerd. `channel_map` is alleen nodig voor een symbool waarop de stack niet kan reageren, zoals in `compute_index()`: het symbool van een preset neemt de ene ingeschakelde band waarvan de naam daarmee begint, gecontroleerd voordat een camera wordt geopend (als er geen of meerdere reageren, wordt dit geweigerd, waarbij ze bij naam worden genoemd); een aangepaste `formula` is voor elk symbool nodig. `index.tif` en de LUT bevatten de camera van de stack, de opnametijd (de belichtingstijd van de referentiecamera, UTC) en de lens, en beide benoemen de index als hun enige band (`XMP-Camera:BandName`, bijv. `NDVI`: de kleuren-PNG-bestanden noemen de index, niet de kleurkanalen). `lattice index --live`. |

### `CliJob`

De handle die de drie streams retourneren. De taak draait op de backend, die de camera’s in bezit houdt totdat deze wordt gestopt.

| Lid | Beschrijving |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (frames opgeslagen / geschreven, verwijderd, `fps`, `elapsed`, schrijffouten, `stop_reason`; bij een doorlopende taak ook `frames_grabbed` en `frames_queued` — elk vastgelegd frame wordt geschreven, in de wachtrij geplaatst, verwijderd of mislukt — en `drop_reason`, `writers` en `write_ms`, de gemiddelde tijd die het schrijven van één frame in beslag nam) en `output_dir`. `state` is `running`, `ended` (het is zelf gestopt en is nog niet gestopt), `stopping` (een stop is bezig zijn wachtrij weg te schrijven: live statistieken, en `stopped_by`) of `finished` (zijn eindstatistieken, de `error` en `stopped_by` van de stop). Het record van een gestopte taak wordt 10 minuten bewaard (maximaal de laatste 32), voor elke client: een ander script of `chloros-mcp` leest dezelfde status en eindstatistieken. Een taak die is gestart met `chloros-cli` (`lattice continuous`, `multi-stream`, `align-stream`) wordt door de backend gestopt wanneer de CLI30 seconden lang niet meer incheckt — beëindigd, de machine in slaapstand — met `stop_reason`, `client_gone` en `stopped_by` naar de backend; het lezen van de statistieken vanaf hier houdt het niet in leven. Een taak die vanuit de SDK is gestart, draait door totdat deze wordt gestopt. Bij elke start wordt de `timeout` verzonden als de deadline van de backend (`client_deadline_s`): een camera die daarna wordt geopend (terwijl deze nog aan het opstarten is), wordt teruggezet en gesloten en er wordt geen taak gestart, en de aanroep genereert `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Een camera die door een taak wordt vastgehouden, wordt geweigerd bij elke andere open (`connect_camera()`, een nieuwe start, een tijdelijke opname) met HTTP-status 409 `CAMERA_HELD`, waarbij de taak en de starter worden vermeld. |
| `stop(timeout=30.0)` | Stop de taak en sluit de bijbehorende camera’s: `final_stats`, `output_dir` en `error` wanneer het stoppen niet soepel is verlopen. Een tweede aanroep levert hetzelfde antwoord op, net als een stopopdracht van een andere client: een die wordt verzonden terwijl de eerste nog bezig is met het wegschrijven van zijn wachtrij, wacht hierop (tot 25 s; daarna wordt er een foutmelding gegenereerd en volgen de definitieve statistieken op `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Wacht tot het vanzelf eindigt (wanneer `count` is bereikt, de schijf vol is of er een fout optreedt), en roep vervolgens `stop()` aan om de definitieve tellingen te verzamelen. `timeout` genereert een foutmelding `TimeoutError` terwijl het nog actief is; een taak zonder `count` eindigt pas wanneer deze wordt gestopt. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Het ID en het type (`continuous`, `multi-stream`, `align-stream`), waar het naar schrijft, het antwoord van de startroute en of `stop()` is uitgevoerd. |

Gebruik het als contextmanager om het bij het verlaten van de scope te stoppen.

---

## Projectmap & recepten

**De projectmap** is de map waarin nieuwe projecten worden aangemaakt, gedeeld door de Chloros-app, de CLI en de SDK (module `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Een geconfigureerde map die niet meer bestaat, valt terug op de standaardmap (`Documents/MAPIR/Chloros Projects`), net als de app. Een relatief pad dat aan `set_project_folder()` wordt opgegeven, wordt afgeleid van je werkdirectory wanneer de backend op deze machine draait.

**Een recept** is de YAML-/JSON-actielijst die `chloros-cli project run PATH RECIPE` uitvoert op een opgeslagen project. `run_project_recipe()` voert deze uit met de CLI'eigen runner (geïnstalleerd met de SDK), dus één recept gedraagt zich hetzelfde vanuit beide:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** is een `.yaml` / `.yml` / `.json`-bestand of het recept zelf: een dict met `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`), en optioneel `overrides` en `stop_on_error` (standaard true: de uitvoering stopt bij de eerste actie die mislukt). De acties zijn de [Recipe DSL](cli-reference.md#recipe-dsl) uit de CLI-referentie; een relatief `output` wordt omgezet naar de werkdirectory van je script.
- **Retourwaarden** `ok` (alle acties zijn geslaagd), `actions` (hoeveel), `log` (de uitvoer van de runner; `echo=True` geeft dit ook weer tijdens het uitvoeren) en `validation`. Een receptbestand dat ontbreekt of niet kan worden geparseerd, genereert `ChlorosConnectError`.
- **Gecontroleerd voordat het wordt uitgevoerd.**`validation` is `{"ok", "errors", "warnings", "steps"}`: elke actie moet precies één bekend werkwoord als sleutel hebben (`{"capture": {"name": ...}}`, niet `{"action": "capture"}` of `{"type": "capture"}`) die verwijst naar een camera, array of sensor die het project bezit, van het juiste type. Een recept met een fout voert**geen** actie uit (`ok` false, de fouten in `log`), ongeacht wat `stop_on_error` aangeeft. Ongebruikte sleutels, en elke array of lichtsensor die het recept aanstuurt maar die nu niet is aangesloten, zijn `warnings`. Een recept sluit geen apparaten aan: sluit de eerste van het project aan (`ChlorosProject.connect_all()`, `chloros-cli project connect`, of de MCP-tool `connect_project_devices`). `dry_run=True` voert niets uit en retourneert `{"recipe": ..., "validation": ...}`.
- In een proces wordt slechts één recept tegelijk uitgevoerd. Voor een YAML-recept is PyYAML vereist.

---

## Projectautomatisering — `ChlorosProject`

Een opgeslagen Chloros-project is een map die `cameras.json` + `sensors.json` + `project.json` bevat. `open_project(path, relink=None)` laadt het manifest (`relink`: de map waarin de ontbrekende bronbestanden van een verplaatst project zich nu bevinden, gebruikt wanneer de backend het laadt -- `process()` -- zie **Een verplaatst project** onder `ChlorosLocal`), en `connect_all` brengt elke opgeslagen camera, array en lichtsensor online met de opgeslagen instellingen — dezelfde hardwarestatus die de GUI zou produceren. Lichtsensoren maken verbinding via de [DAQ-sensorpool](#daq-sensor-sessions), dus ze hebben een actieve backend nodig (zie [`SensorHandle`](#sensorhandle)).

### Minimaal voorbeeld

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Of als contextmanager:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-methoden

| Methode | Beschrijving |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Ontdek en maak verbinding met elke opgeslagen camera, array en lichtsensor. Geeft een verbindingsrapport per klasse terug (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, elk met `connected`, `missing` en `errors`; `arrays` bevat ook `alignment`). Met een backend die luistert op `127.0.0.1:5000` worden de opgeslagen camera's en arrays via deze backend gevonden en verbonden. Als er geen backend is, vindt en bestuurt dit proces ze zelf via `lattice_sdk`, waarvoor de Arena SDK-runtime en OpenCV (`cv2`) nodig heeft — het SDK-wheel installeert geen van beide; wanneer dit niet lukt, wordt elke opgeslagen camera en array vermeld onder `errors` met de reden, en blijft `missing` leeg. Een backend die draait maar de SDK weigert (uitgelogd: HTTP-status 401; een plan onder Copper: 403) is geen „geen backend”: elke opgeslagen camera, array en lichtsensor wordt vermeld onder `errors` met de reden van de backend, en er wordt niets geopend in dit proces. Bij elke aanroep wordt de backend opnieuw gecontroleerd, dus `connect_all()` maakt na het inloggen of na het starten vChloros verbinding met hetzelfde projectobject. Een opgeslagen array die al in de backend actief is met dezelfde camera’s (die van de GUI of die van `chloros-cli project connect`) wordt gekoppeld zoals deze is, zonder de opgeslagen instellingen opnieuw toe te passen, en `disconnect_all()` laat deze draaien. Lichtsensoren lopen altijd via de backend (zie [`SensorHandle`](#sensorhandle)). Er wordt nooit een backend gestart. |
| `disconnect_all()` | Alles afbreken. Elk apparaat wordt eerst vrijgegeven; vervolgens, als de backend de vrijgave van een camera, array of lichtsensor niet heeft bevestigd (de afbraak is verlopen of werd geweigerd), wordt elk apparaat aangeduid met een `ChlorosConnectError`-naam. Een apparaat dat al is verdwenen, of als er geen backend draait, is stil. Een array of lichtsensor die een andere client had geopend, blijft draaien. De `with`-exit van het project wordt nooit geactiveerd. |
| `capture_all(output_dir=".")` | Eén frame van elke aangesloten camera + array, en één spectrum (`SensorHandle.read()`) van elke aangesloten lichtsensor, achtereenvolgens vastgelegd (eerst camera’s, dan arrays, dan sensoren), niet in één keer. |
| `stream(camera, overlays=False, fps=10.0)` | Generator die BGR `numpy`-frames oplevert van een met naam aangeduide camera (of array). `overlays=False` leest frames met de handle'`grab()` op het `debayered`-niveau: via de backend wanneer het project via een backend is verbonden, anders via een directe `lattice_sdk`-opnamelus (arrays leveren `{serial: frame}`-woordenboeken op). `overlays=True` routeert via `ChlorosLocal.camera_stream()` → de `/api/camera/<serial>/stream-annotated` MJPEG-feed van de backend, waarbij het opgeslagen `ui.overlay`-blok van de camera als queryparameters wordt doorgegeven. Vereist backend-modus, OpenCV (`cv2`) en een **standalone camera**: een camera in direct-mode genereert foutcode `RuntimeError` (de backend kan geen camera vastpakken die eigendom is van dit proces) en een array genereert foutcode `NotImplementedError` (overlays samengevoegd per camera — stream een element op naam). Equivalent voor eenmalig gebruik: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Voer uitlijning uit op elke momenteel aangesloten array. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Voer de kalibratie-/indexpijplijn uit op de afbeeldingen van het project (omvat `ChlorosLocal.process`; deze vier zijn de **enige** geaccepteerde kwargs — `indices=` enz. veroorzaakt een fout `TypeError`; stel indices in via `ChlorosLocal.configure()`). Bouwt op een 'lazy' manier een `ChlorosLocal()` op, die automatisch een backend start. |

Attributen:
- `proj.cameras` — `Dict[str, CameraHandle]` op naam EN serienummer.
- `proj.arrays` — `Dict[str, ArrayHandle]` met naam EN array_id als sleutel.
- `proj.sensors` — `Dict[str, SensorHandle]` met naam EN slot_id als sleutel.
- `proj.config` — `project.json["config"]` woordenboek.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Verwerkingsniveaus.** `capture()`, `grab()` en `frame_stream()` gebruiken allemaal hetzelfde `processing`
token, en de keten is cumulatief — elk niveau voert alles uit wat erboven staat:

| Niveau | Uitvoer | Opmerkingen |
| --- | --- | --- |
| `raw` | 1-kanaals Bayer, sensor-native | Geen demosaïek. Overlays zijn op dit niveau niet beschikbaar. |
| `debayered` | 3-kanaals kleur (**standaard** voor `CameraHandle` en alle `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` en `stream()` zijn standaard ingesteld op `raw`): BGR in het geheugen (`grab()` / `frame_stream()`), opgeslagen als RGB door `capture()` | In backend-modus, als opname: de donkere offset (DSNU) van het kalibratiepakket wordt van de raw-gegevens afgetrokken, gevolgd door OpenCV’s randbewuste demosaic — geen lineariteits-LUT, flat field of unmix. Het enige niveau dat werkt zonder backend-modus is de gewone bilineaire demosaic van de camerabibliotheek (geen donkere offset). Het opgeslagen TIFF-bestand is fotometrisch RGB, dus keer de kanalen van een frame in het geheugen om (`frame[..., ::-1]`) om het te vergelijken met een bestand. |
| `radiance` | float32, W/m²/sr/nm, **één vlak per band** | Volledige radiometrische keten: demosaic + 3×3 unmix (multispec) + DSNU + flat-field + NIST-schaal, waarbij belichting × versterking zijn weggefilterd zodat de waarden absoluut zijn. |
| `reflectance` | uint16, 32768 = 1,0 (of uint8, 255 = 1,0 — zie `bit_depth`), **één vlak per band** | Straling gedeeld door neerwaartse stralingsintensiteit (ρ = π·L/E). Vereist een DLS/DAQ-meting — zie de opmerking hieronder. |
| `display` | 8-bits sRGB-achtig | Weergave gelijk aan GUI: CCM + witbalans + gamma via het actieve kleurprofiel van de camera. |

Alleen in de directe (backend-vrije) modus is `debayered` beschikbaar: `grab()` op een ander niveau activeert
`NotImplementedError`, `capture()` negeert `processing`, en `frame_stream()` activeert
`RuntimeError` op elk niveau. `reflectance` heeft een bruikbare downwelling-meting nodig — het frame-eindpunt trekt
de samengevoegde DAQ automatisch naar de DLS-sleuf van de camera, maar zonder gekoppelde DAQ weigert de keten de
reflectantie-uitgang en vermeldt de degradatie eerlijk in de geretourneerde metadata in plaats van stilletjes
een minderwaardig product terug te sturen.

> **Reflectantie DN-schaal — codeer deze niet hard.** LATTICE-reflectantie gebruikt `32768` = ρ 1,0 en vermeldt
> XMP `Chloros:PixelScale=32768`; Survey3-reflectantie gebruikt `65535` = ρ 1,0 en bevat geen
> `Chloros:*`-tags. Lees de tag en deel erdoor. Het is gedefinieerd in het uint16-domein, dus het blijft
> `32768` voor elk formaat dat wordt geschaald (16-bits TIFF, 8-bits TIFF, 8-bits PNG/JPG, 32-bits procent)
> — normaliseer het opgeslagen dtype eerst terug naar uint16 (×257 vanuit 8-bit, ×65535 vanuit float). De enige
> uitzondering is een bestand uit een build van vóór 1.3.0: een 8-bit TIFF-bestand werd toen *afgekapt*, niet op schaal gebracht, dus
> er geen schaal voor geldt en het bevat geen `PixelScale`- of MicaSense-tuple. Behandel een ontbrekende tag in een
> LATTICE-reflectantiebestand als „geen geldige schaal”, niet als een standaardwaarde.

**Kalibratietags: alleen op `raw`.** Het `raw`-bestand van een opname bevat het kalibratiepakket van de camera: de `CalibrationJSON`-payload en de MicaSense-compatibele tags (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), die de ruwe waarden van de sensor beschrijven. Elk bestand op een ander niveau (`debayered`, `display`, `radiance`, `reflectance`, de `index/`-weergave) en elk frame van een continue opname of multicamerastream bevat de identiteit van de camera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) en de eigen opnamegegevens van het frame, `XMP-mapir:FrameJSON` (belichting, versterking, temperatuur, pixelformaat, opnametijd, DAQ-match), aangezien elk `process()`-product deze identificatie draagt. De verwerking zoekt de kalibratie van zo’n bestand op basis van het serienummer en `CalBundleSha256` in de kalibratiecache van Chloros, waarbij het pakket één keer wordt gedownload als het op deze computer ontbreekt.

#### Bestanden gemaakt met meerdere camera’s

Een bestand dat is samengesteld uit de banden van meer dan één camera — een multiband-stack (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), een live-index samengesteld uit beelden van meerdere camera's (`index.tif`, `index_LUT.png`), een `composite/`-beeld van een array-opname — noemt elke camera die erin is verwerkt:

- **Per-cameralijsten, in bandvolgorde** (de compositie: achtergrondlaag, daarna voorgrond; een indexlaag, de camera’s waarvan de banden afkomstig zijn): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (het kalibratiepakket waar de banden van elke camera doorheen zijn gegaan, anders — een stapel van `capture_aligned_multiband()` / `start_aligned_stream()`, waarvan de banden door geen enkel pakket gaan — het pakket dat de camera Chloros in de cache heeft opgeslagen, het pakket waarvan de opnamebestanden de naam dragen), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — parallelle lijsten, waarbij item *n* van elke lijst dezelfde camera beschrijft; `-` wanneer een camera geen waarde heeft (geen kalibratiepakket, een onbekende lens). Elke camera is **één** item: wanneer het bijbehorende pakket meerdere waarden voor een veld bevat — de halogeenlampcertificaten van een pakket — deze worden samengevoegd met `;` (`F-1479-1;F-1480-1;F-1481-1`), nooit met een komma, zodat een lijst één item per camera weergeeft, ongeacht hoe deze wordt afgedrukt.
- **EXIF `SerialNumber`** bevat dezelfde serienummers, gescheiden door komma’s (`214000535,214000551`).
- Het bevat **geen** individuele camera-identiteit (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): die benoemen één camera en de bijbehorende set, en de verwerking zou op basis daarvan de kalibratie van het bestand bepalen. Een index die is berekend op basis van de banden van meerdere camera’s (`index.tif`, `index_LUT.png`) bevat EXIF-gegevens `Model` `LATTICE` (de opstelling), niet het `LATT-…`-model van één camera.

Een bestand met de banden van **één** camera — een TIFF per band, een `capture_aligned_multiband(per_camera=True)`-frame, een stack of live-index van één enkele camera — bevat de eigen identiteitstags van die camera, net zoals elk ander opnamebestand dat doet. De stapel, de bestanden per band en per camera van `capture_aligned_multiband()`, en elke stapel van `start_aligned_stream()` en de bijbehorende bestanden per band, bevatten de opnametijd van hun groep (de belichtingstijd van de referentiecamera, UTC), en een `per_camera`-frame van een monocamera bestaat uit één vlak, net als de stapel in `bit_depth`.**Camera-oriëntatie (spiegelen / roteren).** Frames uit `grab()` / `frame_stream()` hebben altijd de
oorspronkelijke oriëntatie van de camera, op elk niveau. Opgeslagen opnames volgen de oriëntatie-instelling
van de camera (de GUI-opties „Mirror H“ / „Mirror V“ / rotatie):

- **`raw`- en `debayered`-bestanden geven precies weer wat de sensor heeft geregistreerd**, ongeacht de instelling — een gedraaid of gespiegeld Bayer-mozaïek
  zou een ander kleurenfilterpatroon opleveren, en de kalibratiekaarten zijn
  op het eigen raster van de sensor.
- **`display`, `radiance`, `reflectance` en indexbestanden zijn georiënteerd** — één keer gedraaid, na alle
  kalibratieberekeningen. Hun clamp-maskers bevinden zich op hetzelfde raster.
- **Elk opnamebestand registreert de instelling** in XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, met de klok mee, na
  de spiegelingen) en `ChlorosOrientationApplied` — `false` op onbewerkte en gedebayerde beelden, `true` op de georiënteerde
  producten. `ChlorosProcessing` geeft een overzicht van de bewerkingen die op de pixels van dat bestand zijn toegepast (`none` wanneer
  het om native gegevens gaat). Dat omvat elk frame van een doorlopende opname (de eigen instelling van de camera,
  niet toegepast) en van een stream met meerdere camera’s (de identiteit: de frames waaruit deze bestaat zijn native, net als bij een
  array-opname).
- **Bij het verwerken van een project wordt de geregistreerde oriëntatie** toegepast op de producten die worden gemaakt op basis van een raw
  (eerst kalibratie in sensorcoördinaten, vervolgens de array-uitlijningsvervorming, ten slotte de oriëntatie),
  zodat ze overeenkomen met de producten van de live-opname — inclusief de ongekalibreerde sensorrespons en de voor vignettering gecorrigeerde
  fallback-producten; het debayered product blijft native.
- **De Chloros-beeldviewer toont een onbewerkt of debayered bestand dat is gedraaid op basis van de opname** — alleen ter weergave,
  het bestand wordt niet gewijzigd; een bestand zonder de opname wordt getoond zoals het is opgeslagen.
- **Opnames zonder de opname worden als native verwerkt.** Vóór versie 1.3.0 draaide een opname met één camera
  zijn raw- en debayered-bestanden zelf om; Chloros kan zo’n raw-bestand niet onderscheiden van een native bestand en
  doet geen gissingen.
- **Array-opnames houden elk afzonderlijk bestand standaard in native formaat** en registreren de identiteit;
  `ArraySession.capture(orient=True)` verwerkt hun resultaten (zie de bedieningselementen voor array-opnames).

#### Uitvoer per band en gemengde arrays

Op de niveaus `radiance` en `reflectance` levert een camera **één vlak per band van die
camera** — drie voor een M3C (de drie ongemengde banden, BGR-kanaalvolgorde), één voor een M3M (het
enige smalbandfilter). De gerepliceerde vlakken van de monocamera worden samengevouwen, dus een **gemengde
M3C/M3M-array levert per serie verschillende arrayvormen op**. Lees de `bands`-sleutel uit de
framemetadata om te achterhalen wat elk vlak is; leid dit niet af uit het aantal kanalen:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Controleer altijd `hdr["processing_level"] == "reflectance"` voordat je de waarden gebruikt. Een demotie
(geen DAQ-meting, geen kalibratiepakket, onbekende bandindeling) wordt gerapporteerd in `processing_level` en
`radiometric_refused` in plaats van een verhoging — de array die je terugkrijgt is nog steeds een geldig frame, alleen
niet het product waar je om hebt gevraagd. Een opgeslagen opname die is gedegradeerd, bevat dezelfde code in de XMP
als `Chloros:RadiometricRefused` (bijv. `no-band-layout`, `no-dls-snapshot`).

#### Eén trigger per uitlezing — `group_id`

**Array-leden worden geleverd vanuit de gesynchroniseerde framegroep van de array, niet vanuit de eigen
opnamebuffer van elke camera.** De camera’s van een hardwaresynchrone array worden gezamenlijk blootgesteld, maar N uitlezingen per camera
worden niet gezamenlijk *bemonsterd* — het uitlezen van het meest recente gebufferde frame van elke camera overschrijdt de triggergrenzen
grenzen, en één enkel weggelaten frame ontkoppelt de set zonder dat iemand het merkt.

Het leveren vanuit de groep is noodzakelijk, maar op zichzelf niet voldoende: elk verzoek maakt nog steeds
een momentopname van *de meest recente* groep op het eigen moment, en de opnamelus kan tussendoor nieuwe frames publiceren.
Gemeten op een live array met 4 camera’s, **overspanden 12 van de 20 parallelle uitlezingen twee groepen** — altijd
precies naast elkaar. Het uitlezen wordt dus **vastgezet**:

1. Elk frame bevat `group_id`, de monotone publicatieteller van de opstelling.
2. `grab()` leest de master (of, als de master niet kan worden bediend, de volgende camera die dat wel kan),
   en koppelt vervolgens de overige camera’s aan de groeps-ID waarop het terechtkwam (`?group_id=N` op het HTTP-
   eindpunt).
3. De backend bewaart recente groepen **minstens een halve seconde**, zodat een groep die al
   is ingehaald door latere publicaties nog steeds wordt bediend wanneer die trigger wordt geactiveerd — lang genoeg voor de roundtrip van de
   reflectantie-export per camera, zelfs bij een volledige sensorarray met 9 camera’s. Een toewijzing die de backend niet
   langer vasthoudt, wordt geweigerd met `409` en `group_aged_out: true` in plaats van stilzwijgend beantwoord
   met een andere trigger — `grab()` leest de set opnieuw in.

Opnieuw gemeten met pinning: **20 van de 20 uitlezingen leverden één trigger op.** `grab()` doet dit voor je;
je hebt alleen `group_id=` nodig wanneer je het HTTP-eindpunt rechtstreeks aanstuurt.

- Een camera die niet kan worden bediend in de vastgezette groep — ontbreekt daarin, of weigert het verzoek
  (een `band` die hij niet heeft, een niveau dat hij niet kan produceren) — wordt **weggelaten in plaats van aangevuld**
  door een andere trigger, en de andere camera’s worden nog steeds geretourneerd.
- Het dict dat `grab()` retourneert, is een `GrabResult` (een gewone `dict`-subklasse;
  `from chloros_sdk import GrabResult`): `.errors` koppelt elk
  weggelaten serienummer aan de uitzondering die bij het lezen is gegenereerd, en `.group_id` is de gedeelde groep. Een lege
  `.errors` betekent dat elke camera deel uitmaakt van deze groep.
- Een standalone camera heeft geen groep, rapporteert `group_id: None` en **wijst** een `group_id=`
  pin af in plaats van deze te negeren.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Dezelfde teller wordt gepubliceerd op **elke MJPEG-stream per camera** van een array-lid — het
voorbeeld (`/api/camera/stream/<serial>`), het geannoteerde voorbeeld (`/stream-annotated`) en de
reflectantiestream (`/stream-reflectance`) — als `X-Chloros-Group-Id` op elk onderdeel, naast
`X-Chloros-Frame-Id` en `X-Chloros-Timestamp-Ns`. Streams zijn continu in plaats van
op een bepaald tijdstip, dus ze worden gekoppeld door onderdelen te verzamelen en te groeperen op basis van die header — er is geen
pin die ingesteld hoeft te worden. Een op zichzelf staande camera heeft geen groep. Zie
[MJPEG-streams per camera](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` valt nooit stil.** Een tick die mislukt om een reden die vanzelf kan worden verholpen,
wordt overgeslagen — een camerastream levert hiervoor niets op, een array-stream levert de camera’s op die wel
hebben gereageerd (mogelijk geen enkele). Die redenen zijn: nog geen frame; de ‘grab’-thread van de camera is een
seconde of twee gepauzeerd (`404 not streaming`, waarop de backend reageert wanneer een andere camera verbinding maakt
of verbreekt, of wanneer een instellingswijziging de pool opnieuw in evenwicht brengt); een herstart van de backend of `5xx`; uitlijning
is nog niet voltooid (`409 … retry shortly`); en een weigering van de reflectiemeting omdat de DAQ nog geen meting heeft
(`no-dls-snapshot`, een verouderde of lege meting). Een weigering bij de volgende tick zou zich alleen maar herhalen — een
onbekende `band`, `bit_depth=8` op stralingsintensiteit, een kalibratiepakket zonder stralingsintensiteitsschaal, een reflectantie-
weigering met **helemaal geen DAQ** (geen enkele gekoppeld aan de camera of de array, geen enkele verbonden met de
backend: de `dls_source` van de weigering is `"none"`), elke andere `4xx` —
veroorzaakt `chloros_sdk.FrameFetchError` (een `RuntimeError`, die `.status` en `.body` bevat) in
één keer. Hetzelfde geldt voor **60 seconden** aan overgeslagen ticks zonder tussenliggend frame, wat de `fps` ook moge zijn — lang
genoeg om het stabiliseren van de uitlijning en een herstart van de backend te doorstaan, kort genoeg zodat een camera die
echt weg is, de stream beëindigt. Die ‘give-up’-fout is ook een `FrameFetchError`, zowel voor een camera als voor
een array, waarbij de uitzondering van de laatste leesbewerking op `.errors` staat (en, voor een camera, als `__cause__`). Voor
een array betekent „geweigerd” dat elke camera heeft geweigerd; de uitzondering `.errors` bevat de
reden van elke camera.

**Een array `frame_stream()` levert elke trigger één keer op** (zowel `ArrayHandle` als `ArraySession`). Een
tick waarvan de nieuwste set de trigger is die hij al heeft opgeleverd, wacht op de volgende trigger in plaats van
dezelfde set opnieuw op te leveren, zodat de stream loopt met de lagere van `fps` en de eigen framerate van de array, en opeenvolgende sets delen nooit een `group_id`. Een array die stopt met publiceren genereert
`FrameFetchError` („geen nieuwe trigger gedurende 60 s“) in plaats van de laatste set te herhalen. Zonder dit zou een
stream die om meer vroeg dan de array publiceerde — of direct na het verbinden startte —
herhalingen uitdelen: in één gemeten run bevatten 20 sets 11 verschillende groeps-ID’s.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Eén enkele band exporteren — `band`

`grab()` / `frame_stream()` gebruik `band="<name>"` om slechts één van de banden van een camera te exporteren als een
eenkanaals `(H, W, 1)`-array — dezelfde vorm als die van een monocamera, dus de uitvoer van één band wordt
op identieke wijze verwerkt, of deze nu afkomstig is van een M3M of van het splitsen van een M3C. De rode band van een FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Toegestane spellingen zijn de volledige lay-outnaam (`Red_660`, `NIR_850`, `Green_550`) of de korte
naam (`red`, `nir`, `green`), waarbij hoofdletters en kleine letters niet van belang zijn. Een onbekende naam wordt afgewezen en de beschikbare
namen worden weergegeven.

> **De selectie gebeurt op naam, niet op kanaalnummer — en dat is bewust zo gekozen.** „Kanaal 1“ is `Green_550` in
> de BGR-kanaalvolgorde van de array, maar *rood* in de lettervolgorde van de filtercodelettervolgorde (`RGN`), en de twee
> komen niet overeen. Als je de verkeerde kiest, krijg je een volkomen aannemelijk frame van de verkeerde band, dat
> verderop in het systeem niet kan worden gedetecteerd. Om die reden is er geen numerieke index.

#### De streams uitlijnen — `align`

`align=True` past het frame aan aan het gemeenschappelijke referentieframe van de array. Elke camera wordt aangepast
via **dezelfde** profielresolutie en uitsnede-ROI, zodat streams per camera die onafhankelijk van elkaar worden geopend**onderling geregistreerd** zijn — je hoeft ze niet samen op te vragen:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Vereisten en gedrag:

- De camera moet zich in een **hardware-gesynchroniseerde** array bevinden (M8-synchronisatiekabel) die beschikt over een uitlijningsprofiel
  — co-registratie gaat ervan uit dat de leden gelijktijdig worden blootgesteld, wat een software- of
  free-run-array niet biedt. Wordt anders geweigerd met `409`.
- **Afgewezen, nooit stilzwijgend niet-uitgelijnd.** Het opnamepad kan niet-uitgelijnde beelden opslaan met een waarschuwing
  omdat een bestand zijn herkomsttags behoudt en later opnieuw kan worden geregistreerd; een livestream heeft geen
  dergelijke tweede kans, dus een ontbrekend profiel is een fout in plaats van een terugvaloptie.
- **Het aanvragen van uitlijning start het proces.** Het aanvragen van `align` op een array zonder profiel start de
  kalibratie en retourneert `409` met waar het op wacht (bijv. *"wachten tot de belichting
  zich stabiliseert op 214000551 (1/3)"*) — probeer het opnieuw en het lukt, doorgaans binnen enkele tientallen seconden.
  Dit is belangrijk bij headless gebruik: de automatische uitlijning bij het verbinden geeft het na 30 s op, en elke poging
  binnen dat tijdsbestek wordt overgeslagen terwijl smart-AE nog aan het stabiliseren is, dus een array waarvan de belichting
  langzaam convergeert, leverde vroeger geen profiel op voor de hele sessie en er was geen manier om
  er een te verkrijgen zonder de gecombineerde weergave op de desktop te openen. `alignment_status` rapporteert ook
  `ae_align_holdouts`, waarbij de camera’s die nog aan het stabiliseren zijn en hun voortgang worden genoemd.
- Geef elke stream van één array **dezelfde `max_h`**, anders komen hun uitsneden op verschillende pixelrasters terecht.
- De uitlijning wordt *vóór* de bitdieptecodering toegepast, zodat de warp waarden met volledige precisie
  interpoleert.

#### `bit_depth` — precisie inruilen voor bandbreedte

`grab()` en `frame_stream()` accepteren `bit_depth=16` (standaard) of `bit_depth=8`:

| `bit_depth` | Reflectancecodering | Kosten |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, speelruimte tot ρ 2,0 | Volledige precisie. |
| `8` | uint8 die `0..refl_max` overspant (standaard ρ 1,0) | De helft van het aantal bytes; ~0,004 ρ kwantisering bij de standaardbovengrens. Waarden boven de bovengrens worden afgekapt — zie hieronder. |

##### De 8-bits bovengrens — `refl_max`

`refl_max` stelt de ρ in die het 8-bits bereik bestrijkt, dus bij `pixel_scale = 255 / refl_max` is ρ nog steeds
`pixel / pixel_scale`. `grab()` en `frame_stream()` gebruiken het als trefwoord voor zowel de camera als
array-handles (`?refl_max=` op het HTTP-eindpunt). Verhoog deze waarde om ruimte te creëren voor spiegelende
hooglichten, in proportioneel grovere stappen:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | stap |
| --- | --- | --- |
| `1.0` (standaard) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automatische belichting kan een 8-bits clip niet corrigeren, en als je het toch probeert, worden de gegevens nog slechter.**
> ρ = π·L/E waarbij belichting en versterking zijn weggefilterd, dus ρ is een eigenschap van de scène, niet van de
> camera-instelling — gemeten op een live-array: het halveren van de belichting verschuifde de gemiddelde ρ met 1,3%. Het verhogen van
> de belichting om het bereik te „passen“, drijft pixels juist in **sensor**verzadiging, wat
> L onderschat en ρ **laag** vertekent zonder dat er een foutmelding wordt gegeven (gemeten: max. ρ 1,203 → 0,985 bij
> 2× belichting, gemiddeld −23%). Kies het bereik met `refl_max`; het voorkomen van
> verzadiging van de sensor is waar AE al voor dient, en dat beschermt de *geldigheid* van ρ in plaats van het bereik ervan.

**Lees `reflectance_clipped_fraction`, niet de vlag.** `reflectance_clipped_at_1_0` slaat aan bij één
enkele pixel — bij een echte scène rapporteerde het `True` terwijl de fractie `0.000001` was, één pixel
op een miljoen. De fractie geeft aan of je de bovengrens moet verhogen.

**`bit_depth=8` betreft uitsluitend reflectantie.** Een verzoek hiernaar voor `radiance` wordt afgewezen in plaats van
verwerkt: reflectantie heeft een vast bereik van 0..1 dat een 8-bits codering doorstaat, terwijl straling geen
vast bereik heeft, dus om het in 8 bits te passen is een percentieluitrekking per frame nodig waarvan de toewijzing
elk frame verandert — een afbeelding, geen meting.

#### Live reflectantiestream (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` levert 8-bits reflectantie per band als
`multipart/x-mixed-replace` — een M3C als 3-kanaals JPEG, een M3M als grijswaardenbeeld. Queryparameters:
`fps` (standaard 10), `jpeg_q` (standaard 95), `max_h` (0 = native), `band` (één benoemde band, zoals
hierboven), `align` (`1` om te co-registreren — dezelfde regels als de `align` van de SDK), `refl_max` (de 8-bits
coderinglimiet, zoals hierboven). Elk onderdeel bevat
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` en `X-Chloros-Reflectance-Clipped`.
**Delen van verschillende camera’s die dezelfde `X-Chloros-Group-Id` dragen, vormen dezelfde trigger** —
zo worden meerdere streams per camera van één array weer samengevoegd, en dit wordt
aan de serverzijde gegarandeerd omdat array-leden worden geleverd vanuit de gesynchroniseerde framegroep (zie
[Eén trigger per leesbewerking](#one-trigger-per-read--group_id)). Het retourneert `409` wanneer de reflectantie
niet beschikbaar is op het moment van verbinding en beëindigt de stream als deze tijdens het verwerken niet meer beschikbaar is; een vastgelopen
stream betekent dus „geen reflectantie meer“ in plaats van „nog steeds in orde, alleen statisch“.

> **JPEG is verliesgevend — deze pixels zijn een meting.** Compressiefouten komen direct tot uiting in de
> reflectiewaarden en zijn het ergst aan de randen, waar de materiaalgrenzen liggen. De kwaliteit
> is om die reden standaard ingesteld op 95. Als je de getallen intact wilt houden, gebruik dan
> `grab(processing="reflectance")` / `frame_stream(...)`, die hetzelfde resultaat opleveren zonder
> codec in het pad (en `bit_depth=8` als de kosten voor de verbinding, en niet de codec, de beperkende factor zijn).

> **EXIF wordt meegenomen naar de export.** `process()` kopieert het GPS-blok van de bronopname
> **en de bijbehorende ExifIFD** naar elk product, dus exportbestanden bevatten `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` en `CameraSerialNumber`, evenals de
> georeferentie. `FocalLength` is het bestand waarmee Pix4D de grondmonstersafstand berekent — zonder dit bestand
> valt de reconstructie terug op een volkomen verkeerde schaal (in een gemeten geval werd een locatie van 411 m
> omgezet in een locatie van 47,8 km). Het bestand is bewust niet `-all:all`: de structurele tags van IFD0 verstoren
> de LATTICE-uitvoer, en `ExifImageWidth`/`Height` zijn uitgesloten omdat ze de bronopname
> beschrijven in plaats van het geëxporteerde raster.

Subvlaggen voor de opnamefase (van toepassing op de radiometrische niveaus — `radiance`, `reflectance`, `display`):

| Vlag | Standaard | Betekenis |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST-radiometrische schaal. |
| `apply_white_balance` | `True` | WB LUT. DLS-bewust wanneer een DAQ aan de camera is gekoppeld. |
| `apply_index` | `False` | Evaluatie van vegetatie-index. |
| `index_expression` | `None` | Formule overschrijven. Niet-leeg → schakelt index automatisch in. |
| `annotated` | `False` | GUI-decoraties (zebra/raster/peaking) overlappen. Niet beschikbaar voor `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **De retourwaarde is een `chloros_sdk.CapturePaths`, van het type `CapturePathMap`, niet `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` is `Dict[str, Union[str, List[str]]]`: een enkel niveau
> `processing` geeft elk serienummer één pad, terwijl `processing="all"` het de
> **geordende lijst** geeft van elk product dat voor die camera is opgeslagen. `ArrayHandle.capture()` accepteert
> geen `levels=`, en het negeert trefwoordargumenten die het niet kent (`levels`, `daq`,
> `render_index`, `force_daq`), dus geef alleen de hier getoonde door (`aligned=False`
> slaat native bestanden op, net als bij `ArraySession.capture()`); gebruik
> `ArraySession.capture()` voor die besturingselementen. Een live gecombineerde compositie, als er een werd gestreamd, komt binnen onder de extra
> `"combined"`-sleutel in plaats van onder een serienummer. Code die uitgaat van `str` loopt vast op de
> lijstvorm zonder dat een typecontrole hier bezwaar tegen maakt — de annotatie vermeldde `Dict[str, str]`
> een tijdje na de release van de lijstvorm, en daarom bestaat de alias. Normaliseer
> wanneer je de platte vorm wilt:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Wat niet geschreven kon worden: `.failed`.** De map die `capture()` retourneert, is een
> `chloros_sdk.CapturePaths` — een gewone `dict`-subklasse, dus alles hierboven werkt
> ongewijzigd — die ook `.failed` bevat: de bestanden die de backend niet kon schrijven,
> dezelfde vermeldingen als [`CaptureResult.failed`](#bestanden-die-niet-konden-worden-geschreven--captureresultfailed)
> (`serial`, `level`, `error` en `disk_full: True` op een volle schijf). Een gedeeltelijke
> opname geeft nog steeds alle paden weer die zijn geschreven; op een volle schijf is de kaart
> leeg en `.failed` legt uit waarom. `.skipped` geeft de overslagen per camera weer en
> `.group_seq` de gesynchroniseerde groep, en `.alignment` / `.oriented` zijn die
> van de backend, net als bij `CaptureResult`. `capture_smart()` geeft ze weer als
> `result["frames"].failed` en `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Array-uitlijning

`ArrayHandle` stelt het volledige uitlijningsoppervlak bloot. Profielen zijn standaard alleen voor de huidige sessie geldig — roep `export_alignment()` expliciet aan om ze permanent op te slaan.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Uitlijning bij het verbinden

`connect_all(align=...)` kan elke array automatisch uitlijnen bij het verbinden:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Valt terug op `project.json["config"]["auto_align_on_connect"]` wanneer niet gespecificeerd.

### `SensorHandle`

Een opgeslagen lichtsensor (DAQ-U, DAQ-M of DAQ-E), geopend door `connect_all()` in de [DAQ-sensorpool](#daq-sensor-sessions) — dezelfde pool die `connect_daq_sensor()` en `chloros-cli daq pool-connect` gebruiken.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` wacht op het **volgende** spectrum van de sensor — een spectrum dat al in de buffer zit op het moment van de aanroep telt niet mee — en retourneert `(spectrum, is_saturated, integration_time, x, y, z)`, of `None` als er binnen 5 s geen binnenkomt. Een gepauzeerde stream wordt hervat. `spectrum` is uitgedrukt in W/m²/nm wanneer de sensor is gekalibreerd.
- `.sensor` is de [`DAQSensorSession`](#daq-sensor-sessions) van de sensor; voor al het overige: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` en `.settings` zijn de opgeslagen gegevens uit `sensors.json`.

**Wat `connect_all` herstelt.** Elke sensor wordt geopend op het opgeslagen adres — de COM-poort, het Bluetooth-adres of de DAQ-E-host — en, wanneer het project de ID van de sensor heeft geregistreerd, moet het om die sensor gaan: een ander apparaat op dat adres wordt geweigerd en niet aangesloten. De opgeslagen integratietijd, framegemiddelde en automatische belichting worden toegepast, evenals de cap-correctie:

| Opgeslagen cap | Verbonden met |
| --- | --- |
| Een cap-profiel (bijv. `fov_30`) | Dat profiel. |
| Geen gekozen | `sunshine_cosine`, de standaard bij het verbinden, geregistreerd zoals verondersteld (`cap_id_source=auto_default`), dus de verwerking geeft een waarschuwing. De sensoren worden zonder kap geleverd: kies de kap die is gemonteerd. |
| `none` — gekozen als **Geen (sensor zonder kap)** in de GUI | Geen kapcorrectie. |

Een DAQ-E-S heeft geen kapprofiel nodig, ongeacht wat er is opgeslagen: de diffusor maakt deel uit van de kalibratie (zie [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Een project dat door een eerdere Chloros is opgeslagen, registreerde een sensor zonder kap als geen keuze, dus maakt het opnieuw verbinding met `sunshine_cosine`; kies nogmaals **Geen (sensor zonder kap)** in de GUI en sla het project op.

Een sensor die een andere client al in de pool heeft geopend (via de CLI of een ander script) wordt gekoppeld zoals hij is, zonder de instellingen opnieuw toe te passen, en `disconnect_all()` laat deze draaien; een sensor die `connect_all()` heeft geopend, wordt vrijgegeven. Net als bij `connect_daq_sensor()` maakt de Chloros-GUI **geen** deel uit van de pool: sluit een sensor in de GUI af voordat je het project vanuit een script koppelt.**Vereisten en het rapport.** Lichtsensoren hebben een actieve Chloros-backend nodig die is aangemeld bij een betaald Chloros+-abonnement (Copper of hoger), en `connect_all()` start er nooit een. Als er geen actief is, wordt elke opgeslagen sensor vermeld onder `report["sensors"]["errors"]`, met een vermelding dat dit het geval is. Een sensor die de backend niet kan openen, wordt daar ook vermeld, met de reden van de backend; `report["sensors"]["missing"]` blijft leeg voor lichtsensoren.

---

## Directe hardware (zonder backend)

Als je helemaal niet afhankelijk wilt zijn van de backend voor LATTICE-camera’s (CI, headless robots, embedded), importeer dan `lattice_sdk` rechtstreeks — dit wordt opnieuw geëxporteerd door `chloros_sdk`. Let op bij `CAMERA_AVAILABLE`: `lattice_sdk` zit in het SDK-wheel, maar vereist de aanwezigheid van de Arena SDK-runtime en OpenCV (`cv2`), en het wheel installeert geen van beide. Lichtsensoren lopen via de backend — [`connect_daq_sensor()`](#daq-sensor-sessions) of de `proj.sensors` van een project — met één uitzondering: de DAQ-M-bandmagnitude-helper [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), die ook `bleak` nodig heeft.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Voorinstellingen en de trigger

Drie van de vier voorinstellingen werken in **free-run**-modus: de camera belicht continu en een
`capture()` retourneert het volgende frame. `triggered` is de uitzondering — deze zet de
camera in de stand-by voor een hardware-flank op lijn 2, dus er wordt niets vastgelegd totdat er een aankomt.

| Voorinstelling | Trigger | Gebruik bij |
| --- | --- | --- |
| `default` | free-run | algemeen gebruik |
| `high_speed` | free-run | 8-bit, max. 60 fps, korte belichtingstijd |
| `high_quality` | free-run | 12-bit, geen fps-limiet — de gebruikelijke keuze voor foto’s |
| `triggered` | **geactiveerd, lijn 2** | de camera is aangesloten op een M8-synchronisatiekabel en wordt door iets anders geactiveerd |

Als je `triggered` kiest (of zelf `trigger_mode="On"` instelt) zonder dat er iets
op Lijn 2 staat, zal elke `capture()` een time-out geven — en dat is correct, aangezien je
de camera hebt gevraagd om te wachten. De SDK legt dit uit wanneer het gebeurt; zie
[SC_ERR_TIMEOUT tijdens opname](#direct-hardware-backend-free).

> **Opmerking — "GVSP-probe" / `SC_ERR_TIMEOUT -1011`-berichten bij het verbinden zijn geen fouten.**
> Bij het verbinden probeert de SDK**jumboframes** (GVSP-pakketten van 9000 bytes) te onderhandelen voor een hogere doorvoersnelheid. Op een directe point-to-point NIC-verbinding (bijv. een link-local `169.254.x.x`-adres) kan het netwerk doorgaans geen jumboframes verwerken, waardoor deze probe een time-out krijgt en regels zoals de volgende worden gelogd:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Dit is de **beoogde terugvalregeling**: de SDK schakelt automatisch terug naar standaardpakketten van 1500 bytes en de camera blijft normaal verbinding maken (de daaropvolgende `[chunk-enable …]`-regels maken deel uit van de normale verbindingsprocedure). Het vastleggen van beelden werkt nog steeds.
>
> Je kunt deze probe overslaan, maar **het is niet alleen een log-onderdrukker — het schakelt jumboframes uit.** De camera beantwoordt „Don't-Fragment“-pings slechts tot 1500 bytes, hoe goed je netwerk ook is, dus de pingtest alleen kan jumbo nooit detecteren; alleen deze probe kan dat. Schakel deze uit en de camera blijft voor altijd standaardpakketten van 1500 bytes verzenden, op elk netwerk:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Dit is alleen de moeite waard op een netwerk waarvan je *weet* dat het geen jumbo-frames kan verwerken, waar het ongeveer een seconde verbindingstijd per camera bespaart. Aangezien het om een echte afweging gaat en niet om een cosmetische aanpassing, geeft de SDK dit nu aan wanneer je het gebruikt:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Laat het met rust, tenzij je een reden hebt.**Als deze optie ingeschakeld blijft, wordt elke camera één keer per backend-sessie gemeten, en nogmaals wanneer de verbinding verandert (een andere adapter, een andere verbindingssnelheid of een verbinding die is verbroken, zoals bij het wisselen van een switch); bij het opnieuw verbinden wordt het resultaat hergebruikt.**Na een wijziging achter een switch die in de lucht is gebleven, moet je de backend opnieuw opstarten om opnieuw te meten.** Een eerste verbinding kan bij deze controle af en toe enkele minuten duren, en het logboek vermeldt dit wanneer dat het geval is.
>
> Als je de jumbo-doorvoer *wilt*, schakel dan jumbo end-to-end in (NIC MTU 9000 + een switch die deze doorlaat), of zet het vast met `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` wanneer je weet dat de verbinding dit ondersteunt — geef echter de voorkeur aan een `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` per commando boven een permanente instelling, aangezien een vastgelegde grootte de probe overslaat en niet meer aanpast aan het netwerk ervoor. **Elk** apparaat in het pad moet jumbo-pakketten doorlaten — inclusief eventuele PoE-splitters of -injectors, wat de gebruikelijke reden is waarom een anderszins jumbo-geschikte opstelling ze niet kan doorgeven.

> **`SC_ERR_TIMEOUT -1011` tijdens `capture()` / `grab*()` is een ander probleem — dat is een echte fout.**
> De bovenstaande opmerking heeft alleen betrekking op `-1011`, geregistreerd door de**connect-time probe**. Dezelfde foutmelding die voortkomt uit een**capture** betekent dat de camera wel goed is aangesloten, maar geen beelden verstuurt:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> De doorslaggevende aanwijzing is een camera waarvan het *besturings*kanaal in orde is — detectie werkt, instellingen en `[chunk-enable …]`-schrijfbewerkingen slagen allemaal — terwijl *elk* frame een time-out geeft.
>
> **De gebruikelijke oorzaak is dat de camera is ingesteld op een hardwaretrigger.** Bij `trigger_mode="On"` en `trigger_source="Line2"` zendt de camera helemaal niets uit totdat er een elektrische flank op de M8-synchronisatiekabel aankomt. Als er geen kabel is die die lijn aanstuurt, duurt het laden van elk frame oneindig lang. De camera is niet defect en het netwerk werkt prima — hij doet precies wat hem is opgedragen.
>
> `CameraSettings()` en de `default` / `high_speed` / `high_quality`-presets werken in de vrije modus, en een opname die in de time-out gaat terwijl de camera is geactiveerd, geeft een uitleg in plaats van alleen `-1011` weer te geven. `PRESETS["triggered"]` activeert Lijn 2, zoals bedoeld.
>
> Om een camera te dwingen in de vrije loop te werken:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Als er met `trigger_mode="Off"` nog steeds een time-out optreedt, levert de camera daadwerkelijk geen gegevens — stuur ons het logbestand en `ip link show`.

#### Kleurprofielen (RGB-livevoorbeeld) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` bepaalt het kleurprofiel van het scherm voor het **livevoorbeeld** op RGB-camera’s (multispec-camera’s negeren deze instelling):

| Profiel | Betekenis |
| --- | --- |
| `raw` | De radiometrische keten volledig omzeilen. |
| `linear` | DSNU + flat + WB, geen CCM, geen gamma. |
| `natural` | Lineair + gemeten CCM + sRGB-gamma, uitsluitend met de ‘goedkope’ afwerking (chroma-afvlakking + desaturatie van hoge lichten) — de realistische standaardinstelling. |
| `enhanced` | `natural` plus de volledige „hub-parity“-afwerking (defringe, vibrance, CLAHE lokaal contrast). Rijkere weergave tegen ongeveer **het dubbele van de afwerkingskosten per frame**, dus een lagere LIVE-framesnelheid. |
| `custom_temp` | `natural` maar witbalans vastgezet op `custom_cct_k` Kelvin (DLS genegeerd; aan de backend-zijde begrensd op 2000–10000 K). |

Het profiel is een **alleen-voor-live-preview** snelheids-/beeldkwaliteitsregelaar: opgeslagen opnames krijgen altijd de volledige, rijke afwerking, ongeacht het geselecteerde profiel, dus het kiezen van `natural` om frametijd terug te winnen, verlaagt de kwaliteit van wat op de schijf terechtkomt niet. Een onbekend profiel activeert `ValueError`; wanneer een chloros-backend bereikbaar is, wordt de wijziging ook daarheen gepost, zodat het volgende voorbeeldframe dit weergeeft (gebruikers van de directe SDK-versie zonder backend krijgen de instellingenwijziging nog steeds te zien).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Hoogtepunten (RGB-weergave) — `set_highlights`

`LatticeCamera.set_highlights(value)` haalt de helderste, witste pixels van het **scherm** op RGB-camera’s — de oplossing voor een helder wit oppervlak dat wordt weergegeven als een vlakke witte vlek. `value` is `-100`..`0` (begrensd); `0` is uitgeschakeld, en de weergave is dan precies zoals zonder deze instelling. Sterkere instellingen verlagen het wit van het scherm (tot ongeveer 60% bij `-100`) en verbreden de tooncurve, waardoor een oppervlak dat door de automatische helderheid van het voorbeeld naar puur wit werd gedreven, zijn details terugkrijgt.

Het is een weergave-instelling, maar in tegenstelling tot verzadiging en contrast (die alleen het live-voorbeeld wijzigen) verandert het ook wat je opslaat: het live-voorbeeld en opnames op **weergaveniveau** volgen deze instelling, dus die bestanden komen ook donkerder uit (ze hebben de instelling `Chloros:DisplayHighlights`); exporten van straling en reflectie doen dit nooit, en het kan geen sensorgegevens herstellen die zijn afgekapt — verlaag daarvoor de belichting. Multispec- en monocamera’s negeren deze instelling. De waarde wordt opgeslagen in `settings.highlights` en via POST verzonden naar een bereikbare chloros-backend, precies zoals `set_color_profile`; het CLI-equivalent is `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono (M3M)-camera’s en `Calibration`

Een mono **M3M**-camera (`M3M-<lens>-F<wavelength>`) is enkelbandig: één grijswaardenvlak, geen Bayer-mozaïek, geen 3×3 spectrale-crosstalkmatrix. `Calibration` herkent dit en stelt een `is_mono`-vlag beschikbaar. Reflectantie geldt nog steeds als een radiometrische kaart per band (de ontmenging is de identiteitsmatrix), maar multibandberekeningen op één enkele camera leveren zinvolle resultaten op in plaats van onzin:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Om een vegetatie-index op te stellen met monochrome hardware, combineer je meerdere M3M-camera’s bij verschillende golflengten tot een uitgelijnde multiband-stack (zie [Array-uitlijning](#array-alignment)) en bereken je de index over die stack in plaats van op één camera.

#### DAQ-M-bandmagnitudes — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` maakt via Bluetooth verbinding met een DAQ-M via zijn eigen BLE-MAC. Het is de enige klasse voor lichtsensoren die rechtstreeks met de sensor communiceert in plaats van via de backend, dus heeft deze klasse nodig wat de klassen voor directe camera’s ook nodig hebben (`CAMERA_AVAILABLE`: de Arena SDK-runtime en OpenCV) plus `bleak` voor Bluetooth (`pip install "chloros-sdk[camera]"` of `pip install bleak`; zonder dit genereert `DLS` een `DLSError`-fout). Een DAQ-M die elders is aangesloten (de pool van de backend, een project, de GUI) stopt met adverteren, dus maak het daar eerst vrij. **`mac_address` is vereist**: er is geen standaardwaarde, aangezien elke eenheid zijn eigen waarde heeft, en `DLS()` genereert zonder deze waarde een `DLSError` voordat er een verbinding wordt geprobeerd. Zoek het MAC-adres met `discover_daq_sensors(transports=["ble"])` (zie [Discovery](#discovery--finding-an-address-to-connect-with)) of `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integreert een DAQ-M-spectrum (de laatste meting wanneer `spectrum` gelijk is aan `None`; `DLSError` als er nog geen is) via de reacties van de drie kanalen van een camera en retourneert `{"red": …, "green": …, "nir": …}` — de toetsen benoemen de kanalen in die volgorde; bij de curven van een RGB-camera is het derde (`nir`) het blauwe kanaal.

- Zonder `spectral_response` wordt de fleet-curve van het filtermodel gebruikt (de `filter_model` waarmee de `DLS` is gebouwd), die op één exemplaar is gemeten.
- `spectral_response` gebruikt in plaats daarvan de **eigen** gemeten curven van een camera: een dict `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, zoals opgeslagen in het kalibratiepakket van die camera (`color_correction.spectral_response_smoothed` — RGB-camera’s; multispectrale pakketten bevatten geen dergelijke curve). Als het een dict niet kan gebruiken, valt het terug op de fleet-curve.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

De `chloros-cli lattice dls --camera SERIAL` van de CLI doet hetzelfde via de backend: het zoekt de curve van de camera in het geladen of in de cache opgeslagen kalibratiepakket en geeft aan wiens curve is gebruikt. Via HTTP is dat `POST /api/dls/cli-read` met `mac` (verplicht: zonder dit is het antwoord HTTP-statuscode 400, `code` `MAC_REQUIRED`) en `camera_serial` in de body; het `band_magnitudes_source` in het antwoord is `camera <serial>` wanneer de eigen curve van die camera werd gebruikt, of `model <filter>` voor de vlootcurve (een camera zonder curve: geen RGB, of geen pakket geladen of in de cache opgeslagen).

---

## Uitzonderingen

Vang de basisklasse op om „alles wat er mis is gegaan met Chloros“ af te handelen:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` en `ChlorosConfigurationError` zijn attributen van `chloros_sdk` (en van `chloros_sdk.exceptions`), maar niet in `__all__`, dus `from chloros_sdk import *` neemt ze niet mee: importeer ze op naam. De `ChlorosError`-familie wordt afgeleid van `ChlorosLocal` en de daarop gebaseerde functies (`process_folder()`, `process_lattice_capture()`); het smart-connect-oppervlak genereert `ChlorosConnectError`.

Hiërarchie:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## End-to-End-voorbeelden

### 1. Een map verwerken met een aangepaste voortgangsbalk

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live LATTICE-array → Reflectantie + DAQ-referentie

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projectgestuurde opnamecampagne

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Framestream van meerdere camera’s → NumPy-pijplijn

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Headless direct-hardware (zonder backend) opnamescript

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Capaciteitstest vóór het aansluiten van een 4-camera-array

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Equivalent van opname-recept (pure Python)

`run_project_recipe()` voert een CLI-recept uit zoals het is (zie [Projectmap & recepten](#project-folder--recipes)). In Python geschreven zijn de stappen als volgt:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP-server (AI-assistenten)

`chloros-mcp` stelt een AI-assistent die het [Model Context Protocol](https://modelcontextprotocol.io) (MCP) spreekt, zoals Claude Desktop, Claude Code of Cursor, in staat om Chloros aan te sturen: beelden verwerken (een project aanmaken, een map met opnames importeren, instellingen kiezen, verwerken, rapporteren wat de run heeft geëxporteerd) en LATTICE-camera’s, arrays en DAQ-lichtsensoren bedienen (verbinden, voorbeeld bekijken, belichting en kleur instellen, opnamen maken, opnemen, firmware bijwerken). De tools ervan omvatten wat deze SDK doet. Je vraagt het in gewone taal en de assistent roept de tools van de server aan:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

De server is gebaseerd op deze SDK: de verwerkingstools roepen [`ChlorosLocal`](#image-processing--chloroslocal) en de hardwaretools de [smart-connect-aanroepen](#smart-connect-for-lattice-cameras), [DAQ-sessies](#daq-sensor-sessions), `ChlorosProject` en de [camera-hulpprogramma’s](#camera-utilities), die allemaal communiceren met dezelfde backend die de Chloros-app gebruikt. De verzoeken bevatten `X-Chloros-Client: sdk`, zodat de backend de aanmeldings- en abonnementscontroles van de SDK toepast, en de bijbehorende regels van de SDK (Texture Aware wordt bijvoorbeeld geweigerd voor LATTICE-opnames).

### Installeren

De server wordt meegeleverd in het `chloros-sdk`-pakket; er hoeft verder niets te worden geïnstalleerd. De SDK installeert een `chloros-mcp`-commando, dat hetzelfde is als `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** het Chloros-installatieprogramma installeert de SDK via pip in je Python, waardoor `chloros-mcp.exe` in de map `Scripts` van die Python wordt geplaatst.
- **Linux (.deb):** het pakket installeert de SDK voor het systeem `python3` en plaatst het commando `chloros-mcp` in `/usr/bin`, zodat het op de `PATH`. In een virtualenv voegt `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` `chloros-mcp` toe aan de `bin` van de virtualenv.

De server maakt uitsluitend gebruik van de standaardbibliotheek bovenop de SDK (Python 3.7+). Een MCP-client start de server op en communiceert ermee via stdin/stdout; je voert het niet zelf uit, behalve met `--help`.

### Client instellen

**Claude Desktop:**voeg de server toe aan `claude_desktop_config.json` (op Windows: `%APPDATA%\Claude\claude_desktop_config.json`; open dit via**Instellingen > Ontwikkelaar > Configuratie bewerken**), en start vervolgens Claude Desktop opnieuw op:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude-code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** dezelfde `mcpServers`-vermelding als bij Claude Desktop, in `~/.cursor/mcp.json` (elk project; `%USERPROFILE%\.cursor\mcp.json` op Windows) of `.cursor/mcp.json` in één project.

Opties worden in `args` geplaatst, of na het commando voor Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Om een bepaalde Python-SDK te gebruiken in plaats van de `chloros-mcp` op de `PATH`, voer je de module uit met die Python-versie (het volledige pad als `command` als het niet de eerste `python3` op de `PATH` is):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Een client start de server zonder je shell, dus als er wordt gemeld dat de server niet kon worden gestart, staat `chloros-mcp` waarschijnlijk niet op de `PATH` die de client ziet. Geef in plaats daarvan het volledige pad op (`where chloros-mcp` op Windows, `which chloros-mcp` op Linux, waarbij de .deb-bestanden `/usr/bin/chloros-mcp` zijn), of voer de module uit met de Python-versie die de SDK bevat.

### Opties

| Optie | Standaard | Wat het doet |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | De te gebruiken Chloros-backend. |
| `--no-auto-start` | uit | Start nooit een backend: gebruik een backend die al draait, zoals die van de Chloros-app, de Linux-backend-service of een tunnel naar de (zie [Remote-Backend-modus](#remote-backend-mode-pip-only-host-via-tunnel); map-paden moeten dan op beide machines naar dezelfde map verwijzen). |
| `--backend-exe PATH` | waar Chloros het installeert | De backend die moet worden gestart als er geen actief is, voor een Chloros-installatie waar de SDK-map niet naar zoekt. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` of `ERROR`. Logbestanden worden naar stderr gestuurd, wat de meeste clients weergeven in hun MCP-log; stdout bevat alleen protocolberichten. |

### Tools

De argumenten van elke tool zijn die van de SDK-aanroep die deze omhult; de client geeft ze allemaal weer met hun volledige schema. Paden zijn absoluut.

**Verwerking**

| Tool | SDK-aanroep | Wat het doet |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Maakt een project aan en stelt dit in als het geopende project, waarop de verwerkingstools worden toegepast. Een gewone naam maakt de map aan in de standaardprojectmap; een absoluut pad maakt deze daar aan. |
| `open_project` | `ChlorosLocal.open_project` | Opent een opgeslagen project en maakt dit tot het geopende project. De paden van een verplaatst project worden opnieuw gericht naar de map waarin het zich nu bevindt; `relink_folder` (absoluut) geeft de naam aan van de map waarin de andere ontbrekende bestanden zich nu bevinden. Geeft `source_files` en `image_count` terug. Niet gevonden: een foutmelding met de locatie waar ze stonden en de naam `relink_folder`; er is niets geopend. |
| `import_images` | `import_images` | Importeert Survey3-paren `.RAW`+`.JPG`, LATTICE `.tif`-opnames en DAQ `.daq`-opnames; retourneert `count` (afbeeldingen) en `scan_count` (opnames). |
| `configure_processing` | `configure` | Wijzigt de doorgegeven instellingen en geen andere (waarden: [Ondersteunde waarden](#supported-values)); `custom_settings` verzendt een volledige configuratie zoals opgegeven. |
| `get_project_settings` | `get_config` | Leest de opgeslagen verwerkings-, export- en indexinstellingen van het geopende project. |
| `start_processing` | `process(wait=False)` | Begint met het verwerken van het geopende project en keert onmiddellijk terug. |
| `get_processing_status` | `get_status` | Rapporteert `state` (`idle`, `starting`, `processing` of `finished`) en voortgang; zodra deze is voltooid, worden de `summary`, `export_check_failure` van de run gerapporteerd wanneer de exporten niet konden worden geverifieerd, en `image_failure` wanneer afbeeldingen verloren zijn gegaan: de overige producten worden opgeslagen; rapporteer de run als mislukt. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Maakt een project aan voor een map, importeert deze, configureert en start de run in één aanroep (`lattice: true` voor LATTICE-opnames). Terwijl een run bezig is, wordt deze afgewezen voordat er iets wordt aangemaakt, en de backend weigert op dat moment ook een projectwijziging vanuit welke tool dan ook. |
| `export_light_sensor` | `export_light_sensor` | Schrijft gekalibreerde `.daq`- en `.csv`-bestanden voor de lichtsensoropnames van het project opnames van het project zonder beelden te verwerken, waarbij de fabriekskalibratie uit de MAPIR-cloud wordt opgehaald wanneer deze niet in de cache staat (zie [Lichtsensoropnames](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Leest de Chloros-auditmetadata van een verwerkt beeld (bron, kalibratie, toegepaste fasen). |

**Camera's en arrays**

| Tool | SDK-aanroep | Wat het doet |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Wat er momenteel is aangesloten, in elke client, elk met `connected_here`; plus de lopende taken op de server. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Camera’s en lichtsensoren die kunnen worden aangesloten. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | De live-status van één camera, een array (met de bijbehorende capaciteit en status) of een lichtsensor. |
| `analyze_array_network` | `analyze_array_network` | Wat deze host en dit netwerk kunnen ondersteunen voor een voorgestelde array. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Sluit een camera of een gesynchroniseerde array aan (master eerst). |
| `connect_project_devices` | `open_project(...).connect_all` | Sluit elk apparaat aan dat in een opgeslagen project is opgeslagen, inclusief de bijbehorende instellingen. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Maak een camera, array of lichtsensor los (zie Gedrag). `keep_settings: true` op een array slaat de instellingen van elke camera op in zijn eigen geheugen; `clear_settings: true` reset dat geheugen (het resultaat is `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | De apparaten van het project, samen. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Camera-instellingen lezen of schrijven, op basis van de vriendelijke naam of de GenICam-knooppuntnaam. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | De snelheid van een array vastzetten of terugzetten naar de automatische snelheid. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Eén opname (`fastest` en `wait_for_exposure` zijn `capture_fastest` en de opname met vaste belichting). Zonder `output_dir` wordt het opgeslagen in de map van het geopende project`captures`-map van het geopende project, en de bestanden worden aan het project toegevoegd (in de wacht gezet totdat een verwerkingsrun is voltooid); het antwoord `project_add` geeft dit aan. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Continue of intervalopnames, als taak. |
| `start_array_recording` | `ArraySession.record` / `burst` | Een video met gecombineerde index, of een onbewerkte burst, als taak. |
| `build_burst_video` | `ArraySession.build_video` | Een burst opnieuw verwerken tot gekalibreerde video, als taak. |
| `get_job_status` / `stop_job` | | Een taak volgen of stoppen. |
| `get_camera_preview` | | Het live-voorbeeld van elke camera, weergegeven als afbeelding. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Eén frame (of één gesynchroniseerde set) op verwerkingsniveau, zonder op te slaan: statistieken per vlak en een PNG-bestand van elk. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | De uitlijning van een array. |

**Lichtsensoren**

| Tool | SDK-aanroep | Wat het doet |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Sluit een DAQ-U, DAQ-M of DAQ-E aan (`cap_id` stelt de cap-correctie in). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | De meest recente meetwaarden: tijdstempel, verzadiging, spectrumbereik en piek, en het spectrum gemiddeld over golflengtebinnen (`full_spectrum` voor elk punt). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | De stream starten of stoppen. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Een `.daq`-bestand opnemen als een taak; `duration_s` stopt deze vanzelf. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | De cap-correctie (`none` voor een onbedekte sensor). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Welke sensor levert de reflectiewaarde voor elke camera en array. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | De bandensets en de reflectie en indices van de omgeving en het object van twee streaming-sensoren. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | De eigen instellingen van een DAQ-E, opgeslagen in het apparaat. |

**Camerahulpprogramma's** ([Camerahulpprogramma's](#camera-utilities))

| Hulpprogramma | SDK-aanroep | Wat het doet |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` en de bijbehorende setters | De weergavekleur van een RGB-camera: profiel en kleurtemperatuur, verzadiging / contrast, hoge lichten. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Wittebalans in één keer, of automatisch / uit / vaste verhoudingen. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Cameravoorinstellingen. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | De filtermodellen, of één op naam. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Firmwareversies en een flits (standaard een taak). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Stroom- en thermische toestand; slaapstand of actief. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Coëfficiënten van een kalibratiedoel in beeld. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | De netwerkadapters van de camera, een schatting en de oplossing (Alleen voor beheerders). |
| `apply_vignette_correction` | `apply_vignette_correction` | Een map met afbeeldingen corrigeren voor vignettering. |
| `read_dls` | `read_dls` | Eén DLS-meting. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | De hardware en runtime van de host. |

**Transiënte camera's, uitlijningsprofielen en indexen** ([Transiënte arrays, uitlijningsprofielen en indexen](#transient-arrays-alignment-profiles--indices)). Elke tool hier die camera's opent, gebruikt `keep_settings` / `clear_settings` (standaard false): zie [Camerageheugen](#camera-memory--keep_settings--clear_settings).

| Tool | SDK-aanroep | Wat het doet |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Neem één camera op, of stream meerdere synchroon, als een taak (`duration_s` stopt het vanzelf). |
| `capture_cameras_transient` | `capture_cameras_transient` | Eén gesynchroniseerde opname zonder een array aan te sluiten. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | De synchronisatiecontrole van de trigger en welke camera's op de synchronisatielijn zitten. |
| `analyze_array_models` | `analyze_array_models` | Wat een reeks van deze modellen op dit netwerk zou opleveren. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Een uitlijningsprofielbestand berekenen en opslaan (`options` bevat de meer gedetailleerde berekeningsopties). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Een uitgelijnd multibandbeeld, of een reeks daarvan als taak. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | De inhoud van een profielbestand; de volgorde van de banden en de ingeschakelde camera's. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Indexvoorinstellingen en kleurverlopen; een index voor een multiband-TIFF; in één stap vastleggen en indexeren. |

**Tijdsynchronisatie, projectmap en recepten** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projectmap & Recepten](#project-folder--recipes))

| Tool | SDK-aanroep | Wat het doet |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | De status van de PTP-grandmaster, zijn peers en de PTP-status van elke arraycamera. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Start het opnieuw op; stel de prioriteiten in. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | De map waarin nieuwe projecten worden aangemaakt. |
| `run_project_recipe` | `run_project_recipe` | Voer het recept van een opgeslagen project uit als een taak met het bijbehorende logboek. Eerst wordt gecontroleerd: een recept met een fout wordt geweigerd en er wordt niets uitgevoerd; er wordt gewaarschuwd voor niet-aangesloten arrays / sensoren (deze worden niet aangesloten: eerst `connect_project_devices`). `dry_run` keert de controle terug en voert niets uit. |

Er is geen tool om een opgeslagen project te openen voor verwerking: elke verwerkingstaak begint met `create_project` (of `process_folder`).

### Vereisten

- **Chloros geïnstalleerd op dezelfde computer.** De Chloros-backend voert de verwerking uit. De server kan in plaats daarvan gebruikmaken van een backend die via een tunnel bereikbaar is (`--backend-url` met `--no-auto-start`, zie [Opties](#options)).
- **Ingelogd zijn op die computer**, in de Chloros-app of via `chloros-cli login`. De server kan niet voor je inloggen: een tool die wordt aangeroepen terwijl je bent uitgelogd, geeft de melding „Niet ingelogd bij Chloros“ met de reden van de backend, en de assistent laat je dit weten.
- **Een betaald Chloros+-abonnement (Copper of hoger).** De server biedt SDK-toegang, en het gratis Iron-abonnement biedt dit niet: een tool die op het gratis abonnement wordt aangeroepen, antwoordt „Niet beschikbaar in dit Chloros-abonnement” met de reden van de backend.
- Python 3.7 of nieuwer met de SDK, en een MCP-client die lokale (stdio) servers start.

### Werking

- **De backend.** Het starten van de server of het weergeven van de beschikbare tools leidt niet tot het starten van een proces; pas bij de eerste aanroep van een tool wordt verbinding gemaakt. Als de Chloros-app (of de Linux-backenddienst) actief is, gebruikt de server deze backend en laat deze draaien. Als er geen backend draait, start de server de geïnstalleerde backend, zonder venster. Wanneer de server wordt afgesloten (je sluit de client af, of deze start de server opnieuw op), stopt hij de vastgelegde taken en de opnames van de lichtsensor die hij had gestart en maakt hij de aangesloten apparaten vrij; de backend blijft draaien, zodat een nog niet voltooide verwerkingsrun tot het einde wordt voortgezet, en hij stopt zichzelf na 30 minuten als er niets gebruik van maakt. Met `--no-auto-start` start de server nooit een backend: een tool die wordt aangeroepen terwijl er geen actief is, vraagt je om de Chloros-app te openen, en de volgende aanroep nadat je dat hebt gedaan, maakt verbinding.
- **Eén geopend project.** De backend heeft één geopend project, dat wordt gedeeld met de Chloros-app wanneer de app geopend is; werk dus niet aan een ander project in de app terwijl de assistent actief is.
- **Alleen de doorgegeven instellingen veranderen.** `configure_processing` laat elke instelling die niet wordt doorgegeven op de opgeslagen waarde staan (de `configure()` van de SDK zet debayer, vignetcorrectie, reflectiekalibratie, PPK en exportformaat terug naar de standaardinstellingen wanneer deze niet worden doorgegeven; de server vult deze in vanuit het project). Als de opgeslagen instellingen van het project niet kunnen worden gelezen, wordt er niets gewijzigd en wordt dit gemeld, tenzij alle vijf worden opgegeven. `indices` vervangt de indexlijst van het project. Het resultaat geeft weer wat is verzonden als `settings_sent`, en noemt onder `warnings` alle indexen die de banden van de camera niet kunnen vormen.
- **De verwerking verloopt volgens het ‘start-then-poll’-principe.** `start_processing` keert terug zodra de run begint, die vervolgens minuten tot uren duurt, en de assistent volgt dit op met `get_processing_status`. `state` is `idle` (geen verwerking), `starting` (de backend heeft de run nog niet gerapporteerd, tot maximaal 20 s), `processing` of `finished`, met de [samenvatting](#post-run-summary--hints): wat er is geëxporteerd, en aanwijzingen wanneer er minder is geëxporteerd dan verwacht (geen beelden, geen kalibratiedoel gevonden). Producten worden, net als vanuit de app, in de projectmap opgeslagen. `start_processing` wordt geweigerd terwijl een run nog bezig is, en een backend die niet meer reageert, wordt als een fout gemeld, niet als een voltooide run.
- **Wat de assistent ziet.** De waarschuwingen van de SDK verschijnen in het resultaat van de tool onder `warnings`, en fouten (uitgelogd, plan, een onjuist pad) worden weergegeven als fouttekst die de assistent kan lezen en aan jou kan doorgeven. Een lijst met meer dan 20 items wordt ingekort tot de eerste 20, waarbij de volledige lengte wordt weergegeven als `<key>_total`. Paden moeten absoluut zijn.
- **Apparaten worden gedeeld.** Camera’s, arrays en lichtsensoren bevinden zich in de backend, zodat de assistent een apparaat kan aansturen dat is verbonden met de Chloros-app, en wat hij wijzigt, wordt in de app weergegeven. `disconnect_device` geeft weer wat de server heeft aangesloten; een apparaat dat door een andere client is aangesloten (de app, `chloros-cli`, een script, een andere `chloros-mcp`) wordt alleen vrijgegeven met `force: true`, wat de `force_disconnect` van de SDK is, en de weigering vermeldt wie het apparaat heeft aangesloten wanneer de backend dit weet; een apparaat dat de backend helemaal niet in het bezit heeft, wordt gerapporteerd als niet-verbonden (`released: false`, `connected: false`, niets om vrij te geven). Een opname of stream die door de backend wordt uitgevoerd, leidt tot de weergave van `get_job_status` met `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (framegroepen voor een stream met meerdere camera’s) — en een regel met `summary` („220 frames geschreven, 55 weggelaten, 0 mislukt: ...”), terwijl het draait en, na voltooiing, de eindcijfers; de weergave ervan door een andere client (`other_clients_jobs`) bevat hetzelfde (`list_cameras()`, `list_arrays()` en `list_daq_sensors()` bevatten elk apparaat’`connected_by`: `client`, `label`, `name`, `at`). Elke `chloros-mcp` identificeert zichzelf bij de backend als `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, verzonden als `X-Chloros-Client-Label`; elk SDK-script kan dit instellen). Het stoppen van de opname van de lichtsensor van een andere client kost ook `force`.
- **Langdurig werk is een taak.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, en bij elke aanvraag voor vastleggen, oplossen of indexeren van `background`, wordt onmiddellijk een `job_id` geretourneerd; `get_job_status` volgt daarop en `stop_job` beëindigt . Een run gaat door totdat deze wordt gestopt, tenzij je `max_captures`, `duration_s` (of, voor een burst, `max_frames`) opgeeft; `min_free_gb` weigert te starten wanneer de schijf onvoldoende vrije ruimte heeft. Er geldt geen limiet, tenzij je er een instelt. Een opname of stream die de backend uitvoert (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) behoudt het taak-ID van de backend, zodat een andere `chloros-mcp` — een die later is gestart — deze aan de hand van dat ID kan vinden: `get_job_status` volgt deze op (en geeft een overzicht van dergelijke taken zoals `other_clients_jobs`, vanaf `GET /api/camera/cli-jobs`, elk met `started_by`), en `stop_job` stopt deze pas met `force: true`. Elke andere taak (een opnamelop, een array-opname of burst, een achtergrondopname) draait in de `chloros-mcp` die het heeft gestart, en alleen die server weet ervan: een andere die erom vraagt, geeft dit aan. Een gestopte taak leest `stopping` terwijl deze de wachtrij wegschrijft, vervolgens `finished` met de eindtellingen, gedurende 10 minuten en vanaf elke `chloros-mcp`: de server die het heeft gestart, ziet de gedwongen stop van een andere op dezelfde manier. `get_job_status` wacht nooit op een stop: tijdens een stop (van zijn eigen `stop_job`, die van een andere client, of de stop die de camera’s vrijgeeft van een taak die vanzelf is beëindigd) reageert hij onmiddellijk met `stopping` en de live-tellingen, en `stop_job` van een taak die al aan het stoppen is, geeft dit aan. Een beëindigde taak vermeldt de reden in `stop_reason`: de eigen reden (`no_frames`, `disk_full`, `member_missing`, of voor een doorlopende reeks `count`, `duration`, `array_gone`), `limit reached` alleen wanneer de tellingen de `count` (of de `max_frames` van een burst, de`duration_s`) is bereikt, `error`, `stopped` (met `stopped_by`) wanneer een client het heeft gestopt, anders `finished`.
- **Waar opnames naartoe gaan.** Naar de map `captures` van het geopende project, waar de app ze opslaat, tenzij `output_dir` wordt opgegeven.
- **Afbeeldingen.** `get_camera_preview` en `grab_frames` retourneren afbeeldingen die de assistent kan bekijken, naast hun JSON-gegevens. `grab_frames`-afbeeldingen worden qua contrast aangepast voor weergave; de statistieken zijn de werkelijke waarden.
- **Wanneer de server wordt afgesloten**, worden de door hem gestarte taken gestopt en wordt een door hem gestarte opname van de lichtsensor beëindigd. Als de server de backend heeft gestart, maakt hij ook de apparaten vrij waarmee hij verbinding had gemaakt, voordat de backend wordt gestopt; in de backend van de app blijven ze verbonden.
- **Protocol.** Alleen stdio (geen HTTP-eindpunt), alleen tools (geen bronnen of prompts), tekst- en afbeeldingsresultaten, MCP-protocolversies van 05-11-2024 tot 25-11-2025.**Wat ontbreekt.** In- of uitloggen (het account is van jou; de server kan niet inloggen), en het stoppen van een backend die de server niet heeft gestart. Voor al het andere wat deze SDK doet met verwerking en hardware is er een tool. (Het instellen van een DAQ-E – de naam, het authenticatietoken, OTA-wachtwoord, handtekeningvereiste, fabrieksreset — en de directe DAQ-servers zijn uitsluitend die van `chloros-cli`, ook in de SDK: een verkeerde naam of token blokkeert elke Chloros-installatie en sluit deze buiten het apparaat.)

---

## Automatisch starten van backends

De smart-connect-toegangspunten — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` en `discover_daq_sensors` — zijn thin HTTP-clients die ervan uitgaan dat een backend luistert op `127.0.0.1:5000` (de standaard-URL van de smart-connect-interface). Wanneer de GUI of CLI al draait, is er al een actief. Vanuit een kaal script is dat misschien niet het geval — dus **starten deze functies het meegeleverde backend-binaire bestand automatisch** (zonder venster, net zoals `ChlorosLocal` dat doet) vóór hun eerste aanroep, en wachten vervolgens maximaal 60 s totdat het is opgestart. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` en `analyze_array_network()` starten er nooit een.

Regels:

- **Er wordt uitsluitend een lokale URL-link gegenereerd.** Een `backend_url` die verwijst naar `localhost` / `127.0.0.1` / `[::1]` komt in aanmerking; elke andere host wordt beschouwd als de machine van iemand anders en wordt nooit gestart.
- **Een backend die afsluit voordat hij antwoordt, beëindigt het wachten.** De aanroep wordt onmiddellijk afgebroken met de exitcode, het uitgevoerde commando en de laatste regels die de backend heeft weergegeven, in plaats van de 60 seconden af te wachten. Exitcode 98 vormt hierop een uitzondering: een andere backend bezit de poort of het vergrendelingsbestand al (van de Chloros-app, de backend-service of een ander script dat nog aan het opstarten is), dus de aanroep blijft wachten tot die antwoordt. `ChlorosLocal` doet hetzelfde.
- **De backend blijft draaien voor hergebruik** (net als bij de CLI) — er vindt geen impliciete afsluiting plaats wanneer je script wordt beëindigd. Als je het script opnieuw uitvoert, wordt de actieve backend hergebruikt.
- **Schakel dit uit met `auto_start_backend=False`** bij elk van deze aanroepen (bijvoorbeeld wanneer je naar een externe backend verwijst, of wanneer je de levenscyclus van de backend zelf beheert).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Als het meegeleverde binaire bestand niet kan worden gevonden of gestart, genereert de daaropvolgende HTTP-aanroep een actiegerichte, **platformspecifieke** `ChlorosConnectError` in plaats van een standaard ‘verbinding geweigerd’-trace — op Windows verwijst deze je naar de desktop-app of een `chloros-cli`-opdracht; op Linux (zonder GUI) verwijst deze je naar een `chloros-cli`-opdracht of de `.deb`.

---

## Omgeving & Headers

De SDK markeert elke backend-HTTP-aanroep met `X-Chloros-Client: sdk`. De backend past de licentieregels van de SDK/CLI toe (inloggen **en** een betaald Chloros+-abonnement vereist) in plaats van het gratis GUI-traject. Dit wordt automatisch ingesteld tijdens het importeren — je hoeft niets te doen.

De header wordt toegevoegd aan elke `requests` / `urllib`-aanroep waarvan de URL `localhost` of `127.0.0.1` bevat, op elke poort — inclusief je eigen lokale diensten — en vervangt een waarde die je zelf hebt ingesteld. Oproepen naar andere hosts blijven ongewijzigd.

Overschrijf de backend-URL door `backend_url=` door te geven (`ChlorosLocal` accepteert `backend_url=` of `api_url=`; `process_folder()` en `process_lattice_capture()` accepteren alleen `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Een geïnstalleerde backend luistert alleen op loopback, dus een LAN-`backend_url` kan deze niet bereiken; zie [Remote-Backend-modus](#remote-backend-mode-pip-only-host-via-tunnel) voor het tunnelpatroon.)

---

## Versiebeheer en compatibiliteit

- De SDK-versie wordt weergegeven als `chloros_sdk.__version__`.
- De SDK is qua gedrag gekoppeld aan de meegeleverde backend-versie. Het combineren van een oudere SDK met een nieuwere backend werkt meestal (voorwaarts compatibele eindpunten), maar het combineren van een nieuwere SDK met een oudere backend kan `404`-fouten opleveren bij nieuwe eindpunten — upgrade de desktop-app zodat deze overeenkomt.
- De smart-connect-interface (`connect_camera` / `connect_array` / `connect_daq_sensor`) en het eindpunt voor netwerkanalyse retourneren stabiele JSON-schema’s; nieuwe velden worden toegevoegd.

---

## Tips voor probleemoplossing

- **`ChlorosAuthenticationError: Login required`** (afkomstig van `ChlorosLocal`; de Smart-Connect-aanroepen genereren `ChlorosConnectError: Login required. …` om dezelfde reden) → Voer `chloros-cli login EMAIL PASSWORD` één keer uit op deze machine, of meld je aan via de Chloros-desktopapp.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (of `Failed to create project: …` afkomstig van `create_project()`) → Lange paden in Windows zijn op deze computer uitgeschakeld (de standaardinstelling van Windows), waardoor geen enkel pad langer mag zijn dan 260 tekens en producten maximaal ongeveer 130 tekens onder de projectmap staan. Er is niets verwerkt. Het bericht geeft de langste projectmap aan die past: gebruik een kortere, of schakel lange paden in Windows in en start Chloros opnieuw op. Het Windows-installatieprogramma biedt dit aan als „Windows-lange paden inschakelen“, standaard aangevinkt; gebruik anders Groepsbeleid: *Computerconfiguratie > Beheersjablonen > Systeem > Bestandssysteem > Win32-lange paden inschakelen*.
- **`ChlorosConnectError: No Chloros backend is running …`** → De smart-connect roept automatisch een lokale backend op, dus dit verschijnt alleen wanneer het meegeleverde binaire bestand niet kan worden gevonden of gestart (bijv. een pip-only-host zonder desktop-pakket). Het bericht is platformafhankelijk: open op Windows de desktop-app of voer een willekeurige `chloros-cli`-opdracht uit; voer op Linux een `chloros-cli`-commando uit (er is geen GUI) of installeer het `.deb`. Voor een backend op afstand, geef `backend_url=` (en `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** bij het importeren → `lattice_sdk` kon niet worden geladen: de Arena SDK-runtime, of OpenCV (`cv2`), kan in deze Python niet worden geïmporteerd. Het backend-gestuurde oppervlak (`connect_*`, projecteert via een backend, `ChlorosLocal`) werkt nog steeds.
- **Array maakt verbinding met de volledige sensor, maar met een lage framesnelheid**→ Zo is het ontworpen: de automatische modus behoudt de volledige sensor en laat de snelheid dalen tot wat de verbinding aankan. Gebruik `binning=2` / `4` of een kleinere `width` / `height` voor meer frames per seconde, of schakel jumboframes in / upgrade de verbinding. Binning en ROI bieden**geen** oplossing voor geaggregeerde over-subscription (`oversubscribed: true`, fps-velden 0): te veel camera’s voor de verbinding kunnen niet worden opgelost door binning/ROI — verminder het aantal camera’s, schakel jumboframes in of stap over op een snellere NIC (zie [Over-Subscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` meldt dat de ontvangstring van de NIC erg klein is (~0,26 MB) / verbindingspoorten met "FRAMES WILL DROP"** → De ontvangstring van de host-NIC staat op de standaardwaarde (vaak teruggezet naar 32 na een update van het NIC-stuurprogramma). Stel op een Realtek USB 10GbE-adapter `ReceiveBufferLen=256` en `PendingReceives=64` (met verhoogde rechten) in en start vervolgens de backend opnieuw op, zodat de ring opnieuw wordt gelezen. Volledige procedure: [CLI-referentie → Host-NIC instellen en afstemmen](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host loopt vast bij herstarten/afsluiten, later WMI-fouten `Invalid class` / NIC kan niet worden ingeschakeld** → Verouderd USB 10GbE-stuurprogramma veroorzaakt `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Werk het adapterstuurprogramma bij naar een actuele versie (≥ 2026) en pas de instellingen voor de ontvangstring opnieuw toe. Zie [CLI-referentie → Host-NIC instellen en afstemmen](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Array-capture mislukt met HTTP-status 503 "Geen nieuwe gesynchroniseerde framegroep"** → De array heeft binnen de wachttijd geen nieuwe framegroep gepubliceerd. Controleer `arr.status()`: `stalled: True` (of `streaming: False`) waarbij `actual_fps` 0 betekent dat de opnamelus is gestopt met publiceren — verbreek de verbinding met de array en maak opnieuw verbinding. Als `stalled` False is en `actual_fps` groter is dan 0, probeer het dan opnieuw. Direct na `connect_array()` wacht een opname al tot 20 s op de eerste groep.
- **`status()` toont `link_speed_mbps: 4294`** → Windowsk kon de snelheid van de adapter niet lezen; dit is typisch voor USB-ethernetadapters en geen fout. Zie [Verwachte framesnelheden](#expected-frame-rates).
- **`update_firmware()` mislukt met `device never connected back on TCP <port>`** → Een firewall heeft de verbinding van de DAQ-E met deze computer geblokkeerd. Het Windows-installatieprogramma voegt de regel toe (zie [DAQ-E-apparaatinstellingen](#daq-e-device-settings)); bij een andere firewall moet inkomend TCP-verkeer naar de backend vanuit het subnet van de sensor worden toegestaan.
- **Reflectantie geweigerd** → Voor reflectantie op absolute schaal moet een actieve DAQ aan de camera (of array) worden gekoppeld. Koppel deze via de GUI of gebruik `processing="radiance"` (W/m²/sr/nm), waarvoor geen gekoppelde sensor nodig is.
- **`smart=True`-opname duurt langer dan verwacht** → AE-convergentie is afhankelijk van de dynamiek van de scène. Met `ArrayHandle.capture_smart()`, verbreed `exposure_tolerance_pct`, verkort `stability_window_s` of verlaag `settle_timeout_s` voor een snellere, minder gestabiliseerde trigger.

---

## Zie ook

- [CLI-referentie](cli-reference.md) — elke CLI-subopdracht komt overeen met een SDK-aanroep.
- [DAQ-lichtsensoren](https://mapir.gitbook.io/chloros/daq-light-sensors) in de Chloros-handleiding — de sensoren, hun limieten en gekalibreerd bereik, opname, DAQ-E-netwerken.
- [LATTICE-camera's](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) in de Chloros-handleiding — aansluiten, camera-instellingen, opnamemodi, arrays.
- Online documentatie: `https://mapir.gitbook.io/chloros/api-python-sdk`
