# Multi-camera-opstellingen

Een LATTICE **opstelling**bestaat uit twee of meer LATTICE-camera’s die als één gesynchroniseerde eenheid zijn aangesloten. Eén camera is de**master**: deze zendt een hardware GPIO-triggerpuls uit via een gedeelde synchronisatielijn (standaard**Line2**), zodat alle camera’s op hetzelfde moment een opname maken. Chloros voegt PTP-tijdsynchronisatie toe, een live preview (tegelbeelden per camera of één uitgelijnd multiband-composietbeeld), en gesynchroniseerde opname — elke opnamecyclus levert één**framegroep** op waarin alle camera’s dezelfde tijdstempel en frame-ID delen (weergegeven als `fid:N` in de opname-uitvoer).

Arrays zijn de manier waarop mono (M3M)-camera’s vegetatie-indexen produceren — één camera levert één band, en de array lijnt deze uit tot een multiband-stack. Zie [Mono-camera’s &amp; vegetatie-indexen](mono-indices.md).

Er zijn drie gelijkwaardige manieren om een array aan te sluiten, en bij alle drie wordt dezelfde &quot;smart-prep&quot;-workflow uitgevoerd:

| Oppervlak | Ingangspunt |
| --- | --- |
| GUI | Tabblad Camera&#x27;s → **Array aansluiten** (blauwe knop) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (eerste serienummer = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (eerste serienummer = master) |

Smart-prep voert, in volgorde, het volgende uit: een netwerkcapaciteitstest (ICMP DF-ping + GVSP-test), selectie van het synchronisatieniveau, automatisch verkleinen van de framegrootte om deze aan te passen aan de verbinding, PTP inschakelen, automatische keuze van het pixelformaat per camera, automatische instelling van de belichting op basis van de opgeslagen status van elke camera, en configuratie van de GPIO-trigger op Line2.

{% hint style="info" %}
Camera’s moeten bereikbaar zijn via de verbinding voordat dit allemaal werkt — zie [Camera’s aansluiten](connecting.md) voor detectie, adressering en het downloaden van de kalibratie bij de eerste verbinding. Bij opstellingen met meerdere camera’s zijn de instellingen van de ontvangstring van de host-NIC net zo belangrijk als de verbindingssnelheid; de volledige tabel met symptomen en oplossingen staat in de [CLI Referentie § Instelling en afstemming van de host-NIC](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Het dialoogvenster ‘Array Connect’

Tabblad ‘Camera’s’ → **Connect Array**opent een wizard in drie stappen:**Selecteren → Weergavemodus → Instellingen**.

### Stap 1 — Selecteer master en slaves

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

Het dialoogvenster scant het netwerk zodra het wordt geopend („Netwerk wordt gescand...“), en controleert vervolgens de GPIO-triggerbedrading („GPIO-bedrading wordt gecontroleerd...“). Je hebt minimaal **2 camera’s** nodig om een array samen te stellen.

De bedradingscontrole vult de rolselectie waar mogelijk automatisch in en geeft een van de volgende drie meldingen weer:

| Melding | Betekenis |
| --- | --- |
| &quot;GPIO-master gedetecteerd — selecties vooraf ingevuld&quot; (groen) | De test heeft de triggertopologie gevonden; de selectievakjes voor master en slave zijn al aangevinkt. |
| &quot;Geen master gedetecteerd — controleer GPIO-kabel&quot; (oranje) | Geen enkele camera heeft een triggerpuls waargenomen; controleer de synchronisatiebekabeling. Je kunt de rollen nog steeds handmatig kiezen. |
| &quot;Geen synchronisatiekabel: {serienummers}&quot; (oranje) | De vermelde camera’s hebben geen synchronisatiekabel aangesloten. |

De cameratabel heeft de kolommen **Camera / Serienummer / IP / Master (radio) / Slave (selectievakje)**:

* Kies precies **één master**en**één of meer slaves**. Als je nogmaals op de radio van de huidige master klikt, wordt deze weer uitgeschakeld.
* Een camera met de markering **&quot;Geen synchronisatiekabel&quot;** kan nooit als slave worden geselecteerd — een slave zonder triggerbedrading zou voor altijd op de synchronisatielijn blijven wachten en een dood beeld leveren. Sluit die camera in plaats daarvan aan als een standalone camera.
* Camera’s die al standalone zijn aangesloten, worden *niet* uitgeschakeld: ‘array connect’ beëindigt de standalone-sessie en opent de camera opnieuw binnen de array.

**Volgende: Weergavemodus →**wordt beschikbaar zodra een master en ten minste één slave zijn gekozen.**Opnieuw scannen** voert de detectie en de bedradingscontrole opnieuw uit.

{% hint style="warning" %}
**Annuleren** is uitgeschakeld terwijl een scan of test wordt uitgevoerd — annuleren tijdens de test kan de camera SDK met LATTICE-camerafirmware laten crashen. Wacht tot het draaiende pictogram is voltooid.
{% endhint %}

### Stap 2 — Weergavemodus

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Modus | Wat je krijgt |
| --- | --- |
| **Afzonderlijke camera’s** | Eén live-tegel per camera, die allemaal tegelijk worden geactiveerd zodat de beelden synchroon blijven. Elke camera behoudt zijn eigen kleur en instellingen. |
| **Gecombineerde camera’s** *(standaard)* | Eén tegel die de uitgelijnde multiband NDVI/index-compositie weergeeft. Camera’s delen de kleur van de array. |

De weergavemodus verandert alleen de presentatie van het livevoorbeeld — het opnamegedrag is in beide gevallen hetzelfde.

### Stap 3 — Array-instellingen en het geprojecteerde resultaat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Bij het openen van deze scène vraagt Chloros de backend om een **aanbeveling**en past automatisch een combinatie van ROI en binning toe die past bij de ontvangstring van je NIC (het geeft de voorkeur aan binning boven ROI-bijsnijden, omdat binning het volledige gezichtsveld behoudt). Bij elke wijziging die u aanbrengt, wordt de analyse live opnieuw uitgevoerd en wordt het paneel**Verwacht resultaat** aan de rechterkant bijgewerkt.

Linkerkolom — instellingen:

| Instelling | Keuzes | Standaard | Opmerkingen |
| --- | --- | --- | --- |
| **ROI (gezichtsveld)** | Volledig (2048×1536) / Half (1024×768) / Kwart (512×384) | Volledig | Sensoruitsnede: halve/kwart uitsnede naar een kleiner gebied met de oorspronkelijke pixelafstand. |
| **Binning** | 1× / 2× (som 2×2) / 4× (som 4×4) | 1× | Hardware-binning: 2×2 = volledig gezichtsveld tegen een kwart van de draadkosten; 4×4 = volledig gezichtsveld tegen 1/16. Verborgen als de camera&#x27;s geen binning ondersteunen. |
| **Beeld aan de kabelzijde** (uitlezing) | — | — | De breedte × hoogte na binning die daadwerkelijk via de kabel wordt verzonden, afgerond op veelvouden van 16 (minimaal 64). |
| **Pinresolutie**| selectievakje | uit | Chloros verhoogt normaal gesproken automatisch de binning bij het aansluiten wanneer de geprojecteerde snelheid onder**1,5 fps**. Door binning behoudt u de door u gekozen framegrootte en wordt de lagere snelheid geaccepteerd — waardoor een configuratie met te hoge eisen leidt tot een harde verbindingsweigering in plaats van een automatische afname van de snelheid. |
| **Triggerfrequentie** | 0,5–60 fps, stap 0,1 | leeg = auto | De triggerfrequentie van de master. Laat leeg om Chloros deze te laten afleiden. |
| **Wire Budget**| 20–2000 MB/s, stap 10 | leeg = auto | Hoeveel de host daadwerkelijk aankan, in MB/s —**het enige getal waaraan de hele array-toewijzing afhangt.** Automatisch gedetecteerd via de netwerkadapter. Verlaag deze waarde als de array beschadigde frames meldt: de gedetecteerde waarde overschat USB-adapters en gedeelde switches. Als je deze waarde wijzigt, wordt de projectie live opnieuw uitgevoerd. |

Rechterkolom — **Geprojecteerd resultaat**:

* **Synchronisatieniveau** — „Gelijktijdige opname“ (groen), „Gelijktijdige opname (FTD-versprongen uitzending)“ (groen), „Versprongen opname (100 ms drift)“ (oranje) of „Configuratie te groot“ (rood).
* **fps-projectie** — weergegeven als een bereik („zwak → sterk”), omdat de snelheid van een gesynchroniseerde reeks wordt bepaald door de belichtingstijd van de traagste camera.
* **NIC-regel** — verbindingssnelheid en continue doorvoersnelheid („NIC {mbps} Mbps · continu {N} MB/s”).
* **Sim-emit burst-controle** — kan de NIC-ring van de host één gelijktijdige burst van alle camera’s verwerken („Sim-emit burst: X MB · NIC-ring bruikbaar: Y MB ✓/✗”).
* **Controle van het kabelbudget** — totale vraag in stabiele toestand versus de botsingsveilige kabelcapaciteit („Kabelbudget: {vraag} MB/s gevraagd door {n} camera’s · plafond {plafond} MB/s ✓/✗ overbelegd“).
* **&quot;Max. aantal camera’s op deze verbinding: {n} — bepaald door de minimale bandbreedte per camera, dus binning verhoogt dit aantal niet.&quot;** — wordt weergegeven wanneer je dicht bij (of boven) het maximum aantal camera’s zit.
* **&quot;BIJ DEZE INSTELLINGEN ZULLEN FRAMES VERLOREN GAAN.&quot;**— rode waarschuwing met de reden van de backend, plus een lijst met blokkerende factoren en blauwe**oplossingssuggesties** (&quot;Om deze reeks op het netwerk te laten passen&quot; / &quot;Om gelijktijdige opname mogelijk te maken&quot;).**Toepassen &amp; Verbinden** is geblokkeerd totdat er een prognose beschikbaar is, en het label geeft aan waarom het wordt geweigerd:

| Knoplabel | Betekenis | Wat helpt daadwerkelijk |
| --- | --- | --- |
| &quot;Bezig met analyseren...&quot; | Analyse nog bezig. | Wacht. |
| **&quot;Te veel camera’s voor dit netwerk&quot;**| De array belast de verbinding te zwaar (aggregatecontrole mislukt). | Minder camera’s, end-to-end jumbo frames of een snellere NIC.**Een kleinere ROI helpt NIET** — zie hieronder. |
| **&quot;Verminder ROI om in te schakelen&quot;** | Frames zouden bij deze instellingen verloren gaan (burst/ring-controle mislukt). | Verminder ROI, verhoog de binning of repareer de ontvangstring van de NIC. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Tijdens het verbinden kan er een groen **kalibratiedownloadvenster** verschijnen met een voortgangsbalk per seriële poort: de eerste keer dat een camera op een machine wordt aangesloten, haalt Chloros het fabriekskalibratiepakket van ongeveer 3,8 MB via GigE op van de camera (ongeveer 70 seconden per camera). Camera’s die al in de cache staan, tonen dit paneel nooit. Zie [Camera’s aansluiten](connecting.md).

## Bandbreedte: hoeveel camera’s passen er

Wat een array aankan, is een eigenschap van de kabel, niet van Chloros, dus de planningscijfers staan in de hardwarehandleiding: **[Planning van array-bandbreedte](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Wat Chloros hiermee doet: het verbindingsvenster voert een netwerkcontrole uit, berekent de haalbare framesnelheid en kiest een niveau dat hierop aansluit. Als de array de kabel overbelast, weigert het verbinding te maken in plaats van stilletjes pakketten te laten vallen — zie het hierboven beschreven paneel met de verwachte uitkomst.

## Wanneer frames ontbreken

Een camera kan om twee totaal verschillende redenen ontbreken in een gepubliceerde groep,
en deze vereisen tegengestelde oplossingen. Chloros telt ze afzonderlijk in plaats van één
&quot;onvolledig&quot; getal te rapporteren dat geen van beide noemt:

| Wat is er gebeurd | Wat betekent het | Waar moet je kijken |
| --- | --- | --- |
| **Corrupt**— het frame is aangekomen maar was structureel beschadigd | GVSP-pakketverlies op het netwerkpad | Het**bandbreedtebudget**, de ontvangstring van de NIC, jumboframes, de switch |
| **Nooit aangekomen**— er kwam helemaal geen frame binnen | De camera is niet geactiveerd, of er is niets vanuit de camera verzonden | De**M8-synchronisatiekabel**, de synchronisatielijn, of alle apparaten zijn ingeschakeld |

De verdeling wordt elke 10 seconden opnieuw geëvalueerd terwijl de array streamt. Boven de 5 % wordt dit
gelogd met vermelding van beide getallen, en elke beschadigde buffer wordt gemeld zodra dit
voor het eerst per camera gebeurt, waarna de gegevens eenmaal per minuut worden samengevat zodat een lange sessie leesbaar blijft.

**Corrupte frames met nul „nooit aangekomen“ betekenen dat de trigger en de kabelsynchronisatie perfect zijn**en dat elk verloren frame op het netwerkpad ligt. De oplossing is om het**Wire Budget** te verlagen en
opnieuw verbinding te maken.

{% hint style="warning" %}
**Het verlagen van de triggerfrequentie helpt niet bij beschadigde frames.** De pakketpacing
van de camera wordt eenmalig vastgelegd bij het tot stand brengen van de verbinding. Het verlagen van de triggerfrequentie verandert hoe vaak een burst
plaatsvindt, niet hoe snel de burst zelf over de kabel gaat. Op een gemeten opstelling met 4 camera’s
veranderde een halvering van de triggerfrequentie niets, terwijl het verlagen van het wire budget van 240 naar
200 MB/s bij dezelfde opstelling het percentage beschadigde frames terugbracht van 10,4 % naar nul.
{% endhint %}

Een actieve array kan zichzelf niet opnieuw plannen — verbreek de verbinding en maak opnieuw verbinding, zodat de connect-time
picker kan werken op basis van het nieuwe budget.

### USB-netwerkadapters zijn begrensd op 200 MB/s

Een USB-ethernetadapter geeft de *ethernet*-verbindingssnelheid aan, maar wat deze daadwerkelijk
kan volhouden, wordt beperkt door de USB-bus en het bijbehorende stuurprogramma. Aan een USB 10GbE-dongle werd vroeger
een doorvoersnelheid van ongeveer 1000 MB/s toegeschreven — een getal dat nog nooit was gemeten — en het afstemmen van
vier camera’s op basis van die denkbeeldige speling leidde tot 6–18 % beschadigde frames, terwijl de opstelling
nog steeds een gezonde doelframesnelheid rapporteerde. Via USB aangesloten adapters zijn nu begrensd op
**200 MB/s**. De limiet is een absoluut getal en geen percentage, omdat de bus de beperkende factor is:
een USB 1 GbE-adapter haalt ongeveer 80 MB/s en ondervindt hier geen hinder van.

Als je host daadwerkelijk sneller is dan de limiet, verhoog dan **Wire Budget** om dit aan te geven.

## PTP-tijdsynchronisatie

De *synchronisatie* van frames vindt plaats via de hardwaretrigger; **PTP** (IEEE 1588 PTPv2) zorgt voor vergelijkbare *tijdstempels* op elk apparaat. Dit is standaard ingeschakeld bij het aansluiten van de array:

* De **Chloros-host-backend draait de PTP-grandmaster**. LATTICE-camera’s en DAQ-E-lichtsensoren werken hieraan als slave in domein 0, zodat beeldtijdstempels en DAQ-spectra op één klok uitkomen (~1 ms).
* `--no-ptp` (CLI) schakelt dit uit voor werk op de testbank — tijdstempels tussen verschillende camera’s zijn dan **niet** vergelijkbaar.
* Controleer de synchronisatiestatus met de CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Het tabblad ‘Camera’s’ zelf heeft geen PTP-indicator; de synchronisatiegegevens per camera die daar worden weergegeven, zijn de alleen-lezen **Rol**(Master/Slave),**Synchronisatielijn** en het niveau van de mogelijkheden van de array. De PTP-status van DAQ-E wordt weergegeven in de sensordetails op het tabblad ‘Lichtsensoren’.

## De live-weergave

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

van de array Het hoofdweergavegebied biedt twee lay-outs (schakelbaar in de bovenste balk): **rasterweergave**(elke tegel is een cel; versleep om de volgorde te wijzigen wanneer het rasterhangslot is ontgrendeld) en**lijstweergave**(arrays over de volledige breedte bovenaan, één actieve camera daaronder). Met de schuifregelaar**Feed Zoom** kunt u de grootte van de tegels aanpassen; bij een celbreedte van minder dan 200 px worden de naam- en fps-overlays automatisch verborgen.

In de **afzonderlijke modus** wordt één tegel per camera weergegeven. Elke tegel toont:

* de cameranaam (linksboven),
* een **fps-weergave** (rechtsboven) — dit is de *werkelijke opnamesnelheid* van de camera zoals gerapporteerd door de backend, niet de poll-snelheid van het voorbeeld (het live-voorbeeld is begrensd op 30 fps, ongeacht de opnamesnelheid),
* een statuspunt — groen (streaming) / oranje (bezig met laden) / rood (fout),
* een **spinner voor verouderde beelden** wanneer er gedurende 2 s geen nieuw beeld is binnengekomen — normaal gedurende ~5 s na het tot stand brengen of verbreken van een verbinding, terwijl de backend de bandbreedteverdeling tussen de camera&#x27;s opnieuw in evenwicht brengt.**Gecombineerde modus**toont één samengestelde tegel: de backend voert debayering, schaalverandering, uitlijning en ruisonderdrukking uit, converteert naar straling per band (plus DLS-reflectantie wanneer een lichtsensor is gekoppeld), evalueert de indexuitdrukking van de array, past de LUT toe en streamt het resultaat als MJPEG. Totdat het eerste uitgelijnde frame wordt weergegeven, geeft de tegel de status weer: „Array wordt voorbereid…“, &quot;Uitlijning kalibreren…&quot;, &quot;Wachten op eerste frame…&quot;, of — als de tijd voor automatische uitlijningspogingen (~30 s) is verstreken — &quot;Uitlijning vereist&quot; met een knop**Uitlijning kalibreren**.

Handige feiten over de gecombineerde modus:

* De composiet wordt geregistreerd ten opzichte van het frame van de **master**-camera. AE-ROI-focus en spotmeting op de composiet zijn exact voor de master en bij benadering voor de slaves; gebruik**Split View** (array-instellingen → „Show member cameras”) voor pixel-exacte tegels per camera zonder extra cameraverbindingen te openen.
* Met **Display Layers**(array-instellingen; standaard uitgeschakeld) kun je een voorgrond- en achtergrondlaag kiezen — elke lidcamera of**Index**. Als de voorgrond = Index is, tonen pixels buiten de LUT-min/max de achtergrondlaag.
* **Renderresolutie** (standaard 720p) stelt de hoogte van de livestream *en* de exportgrootte van de opgeslagen compositie in. Beelden per camera worden altijd in volledige resolutie geëxporteerd.
* De uitlijning wordt per sessie berekend en nooit opgeslagen — zie het gedeelte over uitlijning in het venster met array-instellingen voor RMS-residuen en de knop *Opnieuw kalibreren*.

## Opnemen: monitoring versus analyse

De opnamevlakken van de array zijn duidelijk onderverdeeld in **monitoring-kwaliteit**(opnemen wat je ziet) en**analyse-kwaliteit** (ruwe gegevens opnemen, later kalibreren):

| Workflow | Kwaliteit | Wat er wordt opgeslagen | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Opname**(foto’s) | Analyse | Eén gesynchroniseerde framegroep per doorloop; bestanden per camera op elk geselecteerd exportniveau (ruw/debayered/straling/reflectie/voorvertoning/index) + `.daq` sidecar | Knop**Alles vastleggen** + Opname-instellingen | `lattice array-capture` |
| **Indexvideo opnemen** | Monitoring | De weergegeven live samengestelde indexcompositie — 8-bit, previewresolutie, LUT ingebakken; de livestream moet geopend zijn | ● Indexvideo opnemen (gecombineerde arrays) | `lattice array-record` |
| **Ruwe burst → video samenstellen**| Analyse | Ruwe sensorframes met volledige opnamesnelheid + manifest + `.daq`, vervolgens offline reconstructie tot gekalibreerde stralings-/reflectantie-/indexvideo, tijdsafgestemd op DAQ-metingen | ⦿ Ruwe burst opnemen →**Video samenstellen** | `lattice array-burst` → `lattice array-build-video` |

Vuistregel: als de pixels *metingen* opleveren, gebruik dan ‘capture’ of ‘burst’ (analysekwaliteit); als je alleen maar wilt *bekijken of demonstreren* wat de sensor heeft waargenomen, neem dan de indexvideo op (monitoringkwaliteit).

### Opname-instellingen (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Het tandwiel naast **Alles vastleggen** opent het venster ‘Opname-instellingen’ (hiervoor moet er een project openstaan — opnames worden daarin opgeslagen):

* **Opnamemodus**:**Enkel**(één doorloop) /**Continu**(achter elkaar; begrensd door een opnametotaal, standaard 1, of een duur, standaard 10 s) /**Interval** (timelapse: N opnames om de X interval voor Y totaal, standaard 1 om de 5 s gedurende 1 minuut).
* **Exporttypes per camera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alles wat van toepassing is, staat standaard ingeschakeld. Radiance/Reflectance zijn verborgen voor RGB-filtercamera’s;**Reflectance verschijnt alleen wanneer de camera een DAQ-lichtsensor heeft** (een eigen sensor of overgenomen van de array); Index vereist een geconfigureerde indexuitdrukking.
* **Uitgelijnd**(per array, standaard**ingeschakeld**): past de exporten van de elementen aan het uitlijningsprofiel van de array aan, zodat de exporten pixel-geregistreerd zijn. Raw blijft altijd ongewrapt, maar bevat de transformatie in de metagegevens.
* **Snelste opname** (schakelaar): alleen raw + de toegewezen DAQ-meting + de gratis gecombineerde indexcompositie, waarbij de kalibratieberekeningen op het moment van opname worden overgeslagen voor maximale snelheid — radiance/reflectantie/index later opnieuw opbouwen vanuit de opgeslagen `.daq`.
* Selecties blijven behouden in het project. Verborgen of gepauzeerde camera’s worden overgeslagen.

Het equivalente CLI (zelfde backend-eindpunt, dezelfde semantiek):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

TIFF-compressie voor opnames is `deflate` (verliesvrij, standaard) of `none` — volledige vlaggenlijsten, de indeling van de opnamemap en regels voor herverwerking staan in de [CLI-referentie](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Een DAQ-lichtsensor koppelen

Voor weergaven met reflectie- en verlichtingscorrectie zijn gegevens over neerwaarts vallend licht nodig van een DAQ-sensor (aangesloten via het tabblad **Lichtsensoren**):

* De **array-rij**in de zijbalk toont een**&quot;DAQ · aan/uit&quot;-knop** — *aan* wanneer een lichtsensor op array-niveau is ingesteld **of** wanneer een van de camera’s in de array een eigen sensor heeft; de tooltip geeft precies aan welke sensor welke camera van gegevens voorziet.
* Wijs een array-brede instelling toe in de array-instellingen → **Omgevingslichtsensor**→ vervolgkeuzemenu**Lichtsensor**. De selectie blijft behouden voor het project, wordt doorgegeven aan elke camera in de array, en individuele camera’s kunnen deze nog steeds overschrijven met hun eigen sensor.
* De statusregel eronder geeft de actuele status weer: **Uit**→ &quot;Wachten op eerste spectrum…&quot; →**&quot;Actief — alle camera’s in de array zijn belichtingsgecorrigeerd&quot;** → of, als er in de afgelopen 3 s geen nieuw spectrum is binnengekomen, een melding dat de gegevens verouderd zijn — de laatste meting blijft dan in gebruik (metingen vervallen nooit in het opnamepad).

Als er een sensor is toegewezen: het exporttype ‘Reflectantie’ wordt beschikbaar, live-voorbeelden zijn belichtingsgecorrigeerd, voorspellende automatische belichting kan het spectrum gebruiken en bij elke reflectantie-opname wordt de daadwerkelijk gebruikte DAQ-meting als een **`.daq` sidecar** naast de beelden, zodat de opname later opnieuw kan worden verwerkt.

## `array-connect` CLI-opties

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--serials SN1,SN2,…` | alle LATTICE-camera’s automatisch detecteren (vereist ≥2) | **De eerste seriële poort is de MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synchronisatielijn. |
| `--target-fps F` | auto | Vuurfrequentie van de mastertrigger. |
| `--binning {1,2,4}` | auto | Hardware-binning. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Handmatige overschrijving van de synchronisatieniveaukiezer. |
| `--wire-ceiling-mbps MB_PER_S` | automatisch gedetecteerd | Host-draadbudget in MB/s — de CLI-variant van het veld **Wire Budget**. Verlaag deze waarde als de array beschadigde frames meldt. Wordt samen met het project opgeslagen, zodat deze bij een latere herverbinding wordt hersteld. |
| `--no-recommend` | uit | Sla de netwerkanalysestap over. |
| `--no-ptp` | uit | Schakel PTP uit (tijdstempels van verschillende camera&#x27;s zijn dan niet vergelijkbaar). |

`lattice array-list`, `array-status` en `array-disconnect` beheren de permanente sessie. De volledige referentie van subcommando&#x27;s, inclusief uitlijning (`align-calibrate` / `align-apply`) en de netwerktools, staat in de [CLI-referentie § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); de SDK-equivalenten (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) staan in de [SDK-referentie](../reference/sdk-reference.md). Vanaf Python is het draadbudget `connect_array(..., wire_ceiling_mbps=120)`, en de splitsing tussen ‘live corrupt’ en ‘never-arrived’ staat op [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
