# Multi-camera-opstellingen

Een LATTICE **opstelling**bestaat uit twee of meer LATTICE-camera’s die als één gesynchroniseerde eenheid zijn aangesloten. Eén camera fungeert als**master**: deze stuurt een hardware-GPIO-triggerpuls naar een gedeelde synchronisatielijn (standaard**Line2**), zodat elk onderdeel op hetzelfde moment een opname maakt. Chloros voegt PTP-tijdsynchronisatie toe, een live preview (tegels per camera of één uitgelijnde multiband-compositie) en gesynchroniseerde opname — elke opnamecyclus levert één **framegroep** op waarin alle camera’s dezelfde tijdstempel en frame-ID delen (weergegeven als `fid:N` in de opname-uitvoer).

Arrays zijn de manier waarop monochrome (M3M) camera’s vegetatie-indexen produceren — één camera levert één band, en de array lijnt deze uit tot een multiband-stack. Zie [Mono-camera’s en vegetatie-indexen](mono-indices.md).

Er zijn drie gelijkwaardige manieren om een array aan te sluiten, en bij alle drie wordt dezelfde „smart-prep“-workflow uitgevoerd:

| Oppervlak | Instappunt |
| --- | --- |
| GUI | Tabblad „Camera’s“ → **Array aansluiten** (blauwe knop) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (eerste serienummer = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (eerste serienummer = master) |

Smart-prep voert het volgende uit: een netwerkcapaciteitstest (ICMP DF-ping + GVSP-test), selectie van het synchronisatieniveau, een projectie van de framesnelheid bij de gevraagde framegrootte, de beslissing voor gepakt 12-bits transport (zie [Gepakt 12-bits transport](#packed-12-bit-transport)), PTP inschakelen, automatische keuze van het pixelformaat per camera, het instellen van de automatische belichting op basis van de opgeslagen status van elke camera, en GPIO-triggerconfiguratie op Line2.

**Standaard volledige sensor (nieuw in 1.3.0).** Connect behoudt de door u gevraagde framegrootte — de volledige 2048×1536-sensor, zonder binning en zonder bijsnijden, tenzij u anders kiest — en laat de beeldsnelheid dalen tot wat de host en het netwerk kunnen leveren. Het voert nooit zelfstandig binning of bijsnijden uit (in versie 1.2.0 werd binning automatisch geactiveerd bij minder dan 1,5 fps). Binning of een kleinere ROI is aan jou: kies er een in het Connect-dialoogvenster en het ‘Projected Outcome’ toont de framesnelheid die dit oplevert; `array-connect` geeft de snellere opties weer met hun verwachte snelheden.

{% hint style="info" %}
Camera’s moeten bereikbaar zijn via de verbinding voordat dit allemaal werkt — zie [Camera’s aansluiten](connecting.md) voor detectie, adressering en het downloaden van de kalibratie bij de eerste verbinding. Bij opstellingen met meerdere camera’s zijn de instellingen voor de ontvangstring van de host-NIC net zo belangrijk als de verbindingssnelheid; de volledige tabel met symptomen en oplossingen staat in de [CLI-referentie § Host-NIC-configuratie en -afstemming](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Het dialoogvenster ‘Array Connect’

Tabblad ‘Camera’s’ → **Connect Array**opent een wizard in drie stappen:**Selecteren → Weergavemodus → Instellingen**.

### Stap 1 — Master en slaves selecteren



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->

Het dialoogvenster scant het netwerk zodra het wordt geopend („Netwerk wordt gescand...“), en controleert vervolgens de synchronisatiekabels („Synchronisatiekabels worden gecontroleerd...“) om te bepalen welke camera’s op de M8-synchronisatielijn zijn aangesloten. Je hebt minimaal **2 camera’s** nodig om een array samen te stellen.

De M8-synchronisatiekabels komen samen op één knooppunt, dus **elke aangesloten camera kan master zijn**. De test vult de rollen vooraf in — de RGB-camera (`-FRGB`) als master wanneer deze is aangesloten, anders de eerste aangesloten camera, en de overige aangesloten camera’s als slaves — en geeft een van de volgende drie meldingen weer:

| Melding | Betekenis |
| --- | --- |
| „Synchronisatiekabel gevonden op {n} camera’s. Elk van deze kan master zijn; de RGB-camera (anders de eerste) is vooraf geselecteerd.“ (groen) | De probe heeft de camera’s op de synchronisatielijn gevonden; de selectievakjes voor master en slave zijn al aangevinkt. |
| „Geen master gedetecteerd – controleer GPIO-kabel“ (oranje) | Geen enkele camera heeft een triggerpuls waargenomen; controleer de synchronisatiebekabeling. De rollen zijn nog steeds vooraf ingevuld (de RGB-camera, of anders de eerste camera in de lijst, als master) en je kunt deze wijzigen. |
| "Geen synchronisatiekabel: {seriële nummers}" (oranje) | Bij de vermelde camera’s is geen synchronisatiekabel aangesloten. |

De cameratabel heeft de kolommen **Camera / Seriële poort / IP / Master (radio) / Slave (selectievakje)**. De rijen worden weergegeven in**cameravolgorde** — M3C (Bayer)-camera’s vóór M3M (mono); onder de M3C-camera’s: FRGB, FRGN, FOCN en vervolgens FNGB; M3M-camera’s op filtergolflengte, van laag naar hoog (F450 vóór F808). De cameralijst in de zijbalk en de standaardindeling van het raster volgen dezelfde volgorde.

* Kies precies **één master**en**één of meer slaves**. Als je nogmaals op het selectievakje van de huidige master klikt, wordt deze gewist. Het kiezen van een andere master gebeurt met één klik: de nieuwe master verlaat de slaves en de vorige master wordt een slave (tenzij deze geen synchronisatiekabel heeft).
* Een camera met de markering **"Geen synchronisatiekabel"** kan nooit als slave worden geselecteerd — een slave zonder triggerbedrading zou eeuwig op de synchronisatielijn wachten en een dood signaal afgeven. Sluit die camera in plaats daarvan aan als een standalone camera.
* Camera’s die al stand-alone zijn aangesloten, worden *niet* uitgeschakeld: bij het aansluiten in een array wordt de stand-alone-sessie beëindigd en wordt de camera opnieuw opgenomen in de array.

**Volgende: Weergavemodus →**wordt beschikbaar zodra een master en ten minste één slave zijn gekozen.**Opnieuw scannen** voert de detectie en de bedradingscontrole opnieuw uit.

{% hint style="warning" %}
**Annuleren** is uitgeschakeld terwijl een scan of test gaande is — annuleren tijdens de test kan de camera-SDK-crash veroorzaken op LATTICE-camerafirmware. Wacht tot het draaiende pictogram is voltooid.
{% endhint %}

### Stap 2 — Weergavemodus

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| Modus | Wat je krijgt |
| --- | --- |
| **Afzonderlijke camera’s** | Eén live-tegel per camera, die allemaal tegelijk worden geactiveerd zodat de beelden synchroon blijven. Elke camera behoudt zijn eigen kleur en instellingen. |
| **Gecombineerde camera’s** *(standaard)* | Eén tegel die de uitgelijnde multiband NDVI-/indexcompositie weergeeft. Camera’s delen de kleur van de array. |

De weergavemodus verandert alleen de presentatie van het livevoorbeeld — het opnamegedrag is in beide gevallen hetzelfde.

### Stap 3 — Array-instellingen en het verwachte resultaat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Bij het openen van dit scherm vraagt Chloros de backend om een **aanbeveling**en past deze toe: de**volledige sensor — geen binning, geen ROI-uitsnede — zolang de framesnelheid dit aankan**, ongeacht de snelheid die de host en de verbinding leveren. Alleen wanneer geen enkele snelheid de volledige sensor aankan, verkleint de aanbeveling het beeld, en dan geeft het de voorkeur aan binning boven ROI-uitsnijden, omdat binning het volledige gezichtsveld behoudt. Vanaf dat punt ruil je resolutie in voor beeldsnelheid. Bij elke wijziging die je aanbrengt, wordt de analyse live opnieuw uitgevoerd en wordt het paneel**Verwacht resultaat** aan de rechterkant bijgewerkt.

Linkerkolom — instellingen:

| Instelling | Keuzes | Standaard | Opmerkingen |
| --- | --- | --- | --- |
| **ROI (gezichtsveld)** | Volledig (2048×1536) / Half (1024×768) / Kwart (512×384) | Volledig | Sensoruitsnede: halve/kwart uitsnede naar een kleiner gebied met de oorspronkelijke pixelafstand. |
| **Binning** | 1× / 2× (som 2×2) / 4× (som 4×4) | 1× | Hardware-binning: 2×2 = volledig gezichtsveld tegen een kwart van de bandbreedtekosten; 4×4 = volledig gezichtsveld tegen 1/16. Verborgen als de camera's geen binning ondersteunen. |
| **Beeld via de kabel** (uitlezing) | — | — | De breedte × hoogte na binning die daadwerkelijk via de kabel wordt verzonden, afgerond op veelvouden van 16 (minimaal 64). |
| **Triggerfrequentie** | 0,5–60 fps, stap 0,1 | leeg = auto | De triggerfrequentie van de master. Laat leeg om Chloros dit te laten afleiden. |
| **Bandbreedte**| 20–2000 MB/s, stap 10 | leeg = auto | Hoeveel de host daadwerkelijk aankan, in MB/s —**het enige getal waarop de toewijzing van de hele array afhangt.** Wordt automatisch gedetecteerd via de netwerkadapter. Verlaag deze waarde als de array beschadigde frames meldt: de gedetecteerde waarde overschat USB-adapters en gedeelde switches. Als je deze waarde wijzigt, wordt de projectie live opnieuw uitgevoerd. Het veld wordt geopend met het budget waarmee de laatste array was aangesloten (opgeslagen met het project), dus een budget dat je eenmaal hebt gecorrigeerd, blijft behouden. |

Het selectievakje **Pin-resolutie** uit versie 1.2.0 is verdwenen: bij het aansluiten wordt altijd de door jou gekozen framegrootte behouden en wordt in plaats daarvan de framesnelheid verlaagd.

Rechterkolom — **Geprojecteerd resultaat**:

* **Synchronisatieniveau** — "Gelijktijdige opname" (groen), "Gelijktijdige opname (FTD-versprongen uitzending)" (groen), "Versprongen opname (100 ms drift)" (oranje), of "Configuratie te groot" (rood). Chloros kiest het niveau: gelijktijdige opname wanneer de bandbreedte kan worden gedeeld met een snelheid waarop de camera’s kunnen worden afgestemd (de framesnelheid daalt naarmate het aandeel van elke camera afneemt), en gespreide opname alleen wanneer dat niet mogelijk is — zodat een array verbinding maakt op elk netwerk, met de framesnelheid die dat netwerk toestaat.
* **fps-projectie** — de framesnelheid waarmee de array naar verwachting zal draaien. Bij gelijktijdige opname is dit één getal: de snelheid komt eerst, en de automatische belichting vult de periode die deze oplevert, waarna de versterking wordt verhoogd, zodat een donkere scène deze niet verlaagt (zie [Hoe automatische belichting belichtingstijd en versterking verdeelt](camera-settings.md)). Bij gespreide opname wordt dit weergegeven als een bereik („donker → licht“), omdat de belichtingstijd van elke camera de spreiding in een donkere scène verlengt.
* **NIC-lijn** — verbindingssnelheid en continue doorvoersnelheid („NIC {mbps} Mbps · continu {N} MB/s“).
* **Sim-emit burst-controle** — kan de NIC-ring van de host één gelijktijdige burst van alle camera’s ontvangen (“Sim-emit burst: X MB · bruikbare NIC-ring: Y MB ✓/✗”).
* **Controle van het netwerkbudget** — totale vraag in stabiele toestand versus de botsingsveilige netwerkcapaciteit („Netwerkbudget: {vraag} MB/s gevraagd door {n} camera’s · plafond {plafond} MB/s ✓/✗ overbelegd”).
* **"Max. aantal camera’s op deze verbinding: {n} — bepaald door de minimale bandbreedte per camera, dus binning verhoogt dit aantal niet."** — wordt weergegeven wanneer je dicht bij (of boven) het maximum aantal camera’s zit.
* **"BEELDEN ZULLEN VERLOREN GAAN bij deze instellingen."**— rode waarschuwing met de reden van de backend, plus een lijst met blokkerende factoren en blauwe**oplossingssuggesties** („Om deze reeks op het netwerk te laten passen“ / „Om gelijktijdige opname mogelijk te maken“).**Toepassen & Verbinden** is geblokkeerd totdat er een prognose is, en het label geeft aan waarom het wordt geweigerd:

| Knoplabel | Betekenis | Wat daadwerkelijk helpt |
| --- | --- | --- |
| "Bezig met analyseren..." | Analyse nog bezig. | Wacht. |
| **"Te veel camera’s voor dit netwerk”**| De verbinding kan de array niet eens met één camera tegelijk aan (aggregatietest mislukt): veel meer camera’s dan de verbinding aankan, of andere camera’s die erop streamen. | Minder camera’s, end-to-end jumboframes of een snellere NIC.**Een kleinere ROI helpt NIET** — zie hieronder. |
| **"Verminder ROI om in te schakelen"** | Frames zouden bij deze instellingen verloren gaan (burst/ring-controle mislukt). | Verminder de ROI, verhoog de binning of repareer de ontvangstring van de NIC. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Tijdens het verbinden kan er een groen **kalibratiedownloadvenster** verschijnen met een voortgangsbalk per seriële poort: de eerste keer dat een camera op een machine wordt aangesloten, haalt Chloros het fabriekskalibratiepakket van ~3,8 MB via GigE uit de camera (ongeveer 70 seconden per camera). Bij camera’s die al in de cache staan, wordt dit venster nooit weergegeven; bij meer dan zes camera’s scrollen de rijen. Zie [Camera’s aansluiten](connecting.md).

### Gecomprimeerde 12-bits overdracht

12-bits formaten (`BayerRG12`, `Mono12`) worden normaal gesproken als 16-bits woorden verzonden. ****Gecomprimeerd**transport verzendt dezelfde 12 bits in 1,5 byte per pixel en decomprimeert deze op de host, waardoor het verzenden van elk frame 25 % minder tijd kost — bij volledige sensorbetrouwing levert dit ongeveer een derde meer frames per seconde op.**De opgeslagen gegevens zijn identiek.** Array Connect bepaalt dit automatisch (`auto`): het pakt alleen in wanneer dat nuttig is en de host voldoende CPU-capaciteit heeft voor het uitpakken. De GUI heeft hiervoor geen schakelaar; gebruik vanuit de CLI `array-connect --packed {auto,on,off}` (standaard `auto`). 8-bits en 16-bits formaten worden nooit gecomprimeerd.

## Bandbreedte: hoeveel camera’s passen er?

Wat een array aankan, hangt af van de verbinding, niet vChloros, dus de planningscijfers staan in de hardwarehandleiding: **[Array Bandwidth Planning](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Wat Chloros hiermee doet: het verbindingsvenster voert een netwerkproef uit, schat de haalbare framesnelheid en kiest een niveau dat past. Een array maakt verbinding op elk netwerk: op een trage verbinding wordt elke camera aangepast aan zijn aandeel van de verbinding en daalt de framesnelheid, in plaats van dat de array wordt geweigerd of dat de frames worden verkleind. Alleen een array waarvan de verbinding niet eens één camera tegelijk aankan, wordt geweigerd (**Te veel camera’s voor dit netwerk**) in plaats van dat er stilletjes pakketten worden weggegooid — zie het hierboven beschreven paneel met de geprojecteerde uitkomst.

## Wanneer frames ontbreken

Een camera kan om twee totaal verschillende redenen ontbreken in een gepubliceerde groep,
en deze vereisen tegengestelde oplossingen. Chloros houdt ze apart bij in plaats van één
"onvolledig" getal te rapporteren dat geen van beide noemt:

| Wat is er gebeurd | Wat betekent het | Waar moet je kijken |
| --- | --- | --- |
| **Beschadigd**— het frame is aangekomen maar was structureel defect | GVSP-pakketverlies op het netwerkpad | Het**bandbreedtebudget**, de NIC-ontvangstring, jumboframes, de switch |
| **Nooit aangekomen**— er kwam helemaal geen frame binnen | De camera heeft niet geactiveerd, of er is niets uitgezonden | De**M8-synchronisatiekabel**, de synchronisatielijn, of alle leden zijn ingeschakeld |

De verdeling wordt elke 10 seconden opnieuw geëvalueerd terwijl de array streamt. Boven de 5 % wordt dit
gelogd met vermelding van beide getallen, en elke beschadigde buffer wordt gemeld zodra dit
voor het eerst per camera gebeurt, waarna de gegevens eenmaal per minuut worden samengevat zodat een lange sessie leesbaar blijft.

**Corrupte frames met nul ‘nooit aangekomen’ betekent dat de trigger en kabelsynchronisatie perfect zijn**en dat elk verloren frame op het netwerkpad ligt. De oplossing is om het**Wire Budget** te verlagen en
opnieuw verbinding te maken.

{% hint style="warning" %}
**Het verlagen van de triggerfrequentie helpt niet bij beschadigde frames.** De pakketpacing
van de camera wordt eenmalig vastgelegd bij het tot stand brengen van de verbinding. Het verlagen van de triggerfrequentie verandert hoe vaak een burst
plaatsvindt, niet hoe snel de burst zelf over de kabel gaat. Op een gemeten opstelling met 4 camera’s
veranderde een 5×-verlaging van de triggerfrequentie niets, terwijl het verlagen van het wire budget van 240 naar
200 MB/s bracht hetzelfde systeem van 10,4 % beschadigde frames naar nul.
{% endhint %}

Een actieve array kan zichzelf niet opnieuw plannen — verbreek de verbinding en maak opnieuw verbinding, zodat de verbindingstijd-
kiezer kan werken op basis van het nieuwe budget.

### USB-netwerkadapters hebben een limiet

Een USB-ethernetadapter geeft zijn *ethernet*-verbindingssnelheid aan, maar wat hij daadwerkelijk
kan volhouden, wordt beperkt door de USB-bus en het bijbehorende stuurprogramma. Een USB 10GbE-dongle kreeg vroeger
een doorvoersnelheid van ongeveer 1000 MB/s toegeschreven — een getal dat nog nooit was gemeten — en het afstemmen van
vier camera’s op die fictieve speling leidde tot 6–18 % beschadigde frames, terwijl de array
nog steeds een gezonde beoogde framesnelheid rapporteerde. Via USB aangesloten adapters zijn nu begrensd op
**200 MB/s** op Windows en NVIDIA Jetson, en **400 MB/s** op Linux x86_64. De limiet is een absoluut getal in plaats van een percentage, omdat de bus de beperkende factor is:
een USB 1 GbE-adapter haalt ongeveer 80 MB/s en ondervindt hier geen hinder van.

Als je host daadwerkelijk sneller is dan de limiet, verhoog dan **Wire Budget** om dit aan te geven.

## PTP-tijdsynchronisatie

De *synchronisatie* van frames vindt plaats via de hardwaretrigger; **PTP** (IEEE 1588 PTPv2) zorgt voor vergelijkbare *tijdstempels* op elk apparaat. Dit is standaard ingeschakeld bij het aansluiten van de array:

* De **Chloros-hostbackend draait de PTP-grandmaster**. LATTICE-camera’s en DAQ-E-lichtsensoren werken hieraan als slave in domein 0, zodat beeldtijdstempels en DAQ-spectra op één klok uitkomen (~1 ms).
* `--no-ptp` (CLI) schakelt het uit voor werk op de testbank — tijdstempels tussen camera's zijn dan **niet** vergelijkbaar.
* Controleer de synchronisatiestatus via de CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Het tabblad ‘Camera’s’ zelf heeft geen PTP-indicator; de synchronisatie-informatie per camera die daar wordt weergegeven, bestaat uit de alleen-lezen **Rol**(Master/Slave),**Synchronisatielijn** en het niveau van de mogelijkheden van de array. De PTP-status van DAQ-E wordt weergegeven in de sensordetails op het tabblad ‘Lichtsensoren’.

## De live-weergave

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

van de array Het hoofdweergavegebied biedt twee lay-outs (schakelbaar in de bovenste balk): **rasterweergave**(elke tegel is een cel; verslepen om de volgorde te wijzigen wanneer het rasterhangslot is ontgrendeld) en**lijstweergave**(rijen over de volledige breedte: een**gecombineerde**array is één rij — de samengestelde tegel — terwijl een**afzonderlijke**array**elk lid een eigen rij**geeft, met een eigen livefeed en een eigen kalibratiedoel-tabel; onder de arrays staat één losstaande camera — de actieve). De schuifregelaar**Feed Zoom** past de grootte van de tegels aan; bij een celbreedte van minder dan 200 px worden de naam- en fps-overlays automatisch verborgen. De aangesloten camera’s verschijnen in de zijbalk en in de standaardindeling van het raster in de volgorde van de camera’s (M3C vóór M3M, FRGB eerst, mono’s op golflengte); het ontgrendelen van het raster en slepen heeft nog steeds voorrang op de rastervolgorde.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**De afzonderlijke modus** toont één tegel per camera. Elke tegel toont:

* de cameranaam (linksboven),
* een **fps-weergave** (rechtsboven) — dit is de *werkelijke opnamesnelheid* van de camera zoals gerapporteerd door de backend, niet de pollfrequentie van het voorbeeld (het livevoorbeeld is begrensd op 30 fps, ongeacht de opnamesnelheid),
* een statuspunt — groen (streaming) / oranje (bezig met laden) / rood (fout),
* een **spinner voor verouderde beelden** wanneer er gedurende 2 s geen nieuw beeld is binnengekomen — normaal gedurende ~5 s na het verbinden of verbreken van de verbinding, terwijl de backend het bandbreedtebudget over de camera’s herverdeelt.**Gecombineerde modus**toont één samengestelde tegel: de backend voert debayering uit, schaalt, lijnt uit, verwijdert ruis, converteert naar straling per band (plus DLS-reflectie wanneer een lichtsensor is gekoppeld), evalueert de indexuitdrukking van de array, past de LUT toe en streamt het resultaat als MJPEG. Totdat het eerste uitgelijnde frame wordt weergegeven, geeft de tegel de status weer: „Array wordt voorbereid…“, "Uitlijning kalibreren…", "Wachten op eerste frame…", of — als de automatische uitlijningspogingen van de tegel (ongeveer een minuut) zijn uitgeput — "Uitlijning vereist" met een**Uitlijning kalibreren**-knop.

De fps-weergave van de array-tegel geeft ook de status van de stream weer:**"Instellingen toepassen…"**terwijl een wijziging in de instellingen (formaat, binning, ROI) de camera’s opnieuw opstart,**"Vastgelopen"**wanneer de array actief is maar geen framegroepen meer levert, en**"Gestopt"**wanneer de streaming is gestopt — sluit de array in dat geval opnieuw aan. Wanneer de spectrale ontmenging een band in de composiet op een groot deel van de pixels (25 % of meer) moest afknijpen, toont de tegel**"Bandgegevens met lage betrouwbaarheid · N%"**: indexwaarden die op die band zijn gebaseerd, zijn twijfelachtig (meestal heeft één band te weinig licht ontvangen).

Handige feiten over de gecombineerde modus:

* De composiet is geregistreerd ten opzichte van het frame van de **master**-camera. AE-ROI-richting en spotmeting op de composiet zijn exact voor de master en bij benadering voor de slaves; gebruik**Split View** (array-instellingen → "Show member cameras") voor pixel-exacte tegels per camera zonder extra cameraverbindingen te openen.
* **Display Layers**(array-instellingen; standaard uitgeschakeld) laat je een voorgrond- en achtergrondlaag kiezen — elke lidcamera of**Index**. Met voorgrond = Index tonen pixels buiten de LUT Min/Max de achtergrondlaag.
* **Renderresolutie** (standaard 720p) bepaalt de hoogte van de livestream *en* de exportgrootte van de opgeslagen composiet. Beelden per camera worden altijd in volledige resolutie geëxporteerd.
* De uitlijning wordt per sessie berekend en nooit opgeslagen. Chloros lijnt **elke array automatisch uit na het verbinden, in beide weergavemodi**, zodra de automatische belichting van de leden is gestabiliseerd (het blijft dit tot 5 minuten lang proberen). Het gedeelte**Uitlijning**in het instellingenvenster van de array toont de RMS-afwijkingen en de knop**Uitlijning opnieuw kalibreren** — ook voor afzonderlijke arrays die geen samengestelde tegel hebben.

## Opnemen: monitoring versus analyse

De opnameoppervlakken van de array zijn duidelijk onderverdeeld in **monitoring-kwaliteit**(vastleggen wat je ziet) en**analyse-kwaliteit** (ruwe gegevens vastleggen, later kalibreren):

| Workflow | Kwaliteit | Wat wordt opgeslagen | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Opname**(stilstaande beelden) | Analyse | Eén gesynchroniseerde framegroep per passage; bestanden per camera op elk geselecteerd exportniveau (ruw/debayered/straling/reflectie/voorbeeld/index) + `.daq` sidecar |**Knop **Alles vastleggen** + Instellingen voor vastleggen | `lattice array-capture` |
| **Indexvideo opnemen** | Monitoring | De weergegeven live samengestelde indexcompositie — 8-bit, voorbeeldresolutie, LUT ingebakken. Vanuit de CLI, zonder geopende GUI-weergave, rendert de backend de NDVI-waarde van de array (of `--expr`) zodra de array is uitgelijnd; een opname waarbij geen frame wordt geschreven, mislukt | ● Indexvideo opnemen (gecombineerde arrays) | `lattice array-record` |
| **Ruwe burst → video samenstellen**| Analyse | Ruwe sensorframes op volledige opnamesnelheid + manifest + `.daq`, vervolgens offline reconstructie tot gekalibreerde stralings-/reflectantie-/indexvideo, qua tijd afgestemd op DAQ-metingen | ⦿ Ruwe burst opnemen →**Video samenstellen** | `lattice array-burst` → `lattice array-build-video` |

Vuistregel: als de pixels *metingen* moeten opleveren, gebruik dan ‘capture’ of ‘burst’ (analysekwaliteit); als je alleen maar wilt *bekijken of demonstreren* wat de array heeft waargenomen, neem dan de indexvideo op (monitoringkwaliteit).

### Opname-instellingen (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Het tandwieltje naast **Alles vastleggen** opent het venster ‘Opname-instellingen’ (hiervoor moet er een project openstaan — opnames worden daarin opgeslagen):

* **Opnamemodus**:**Single**(één keer) /**Continuous**(achter elkaar; begrensd door een aantal opnames, standaard 1, of een duur, standaard 10 s) /**Interval** (time-lapse: N opnames om de X interval voor Y in totaal, standaard 1 om de 5 s gedurende 1 minuut).
* **Exporttypes per camera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alles wat van toepassing is, staat standaard aan. Radiance/Reflectance zijn verborgen voor camera’s met een RGB-filter; **Reflectance verschijnt alleen wanneer de camera een DAQ-lichtsensor heeft** (een eigen sensor of een sensor die is overgenomen van de array); voor Index is een geconfigureerde indexuitdrukking vereist.
* **Uitgelijnd**(per array, standaard**ingeschakeld**): past de exportbestanden van de leden aan het uitlijningsprofiel van de array aan, zodat de exportbestanden pixel-uitgelijnd zijn. Raw blijft altijd onvervormd, maar bevat de transformatie in de metagegevens. Bestanden van array-leden worden opgeslagen in de oorspronkelijke oriëntatie van elke camera (eventuele instellingen voor spiegelen/roteren worden in het bestand vastgelegd, maar niet toegepast).
* **Snelste opname** (schakelaar): alleen raw + de toegewezen DAQ-waarde + de gratis samengestelde index, waarbij de kalibratieberekeningen tijdens het vastleggen worden overgeslagen voor maximale snelheid — reconstrueer straling/reflectie/index later vanuit het opgeslagen `.daq`.
* Selecties blijven behouden in het project. Verborgen of gepauzeerde camera’s worden overgeslagen.
* **Continu** op arrays draait binnen de Chloros-backend (nieuw in 1.3.0): wanneer elke camera in de lijst tot een array behoort, voert de backend de opnamelus zelf uit, waarbij één groep wordt weggeschreven terwijl de volgende wordt geselecteerd, zonder verzoek per opname. Een volle schijf stopt de uitvoering („Gestopt: schijf vol. N opgeslagen, F mislukt“). Zie [Opname-instellingen en -modi](capture.md#capture-modes) voor de opmerking van Microsoft Defender over hoge opnamesnelheden.

De equivalente CLI-opdracht (zelfde backend-eindpunt, dezelfde semantiek):

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

TIFF-compressie voor opnames is `deflate` (verliesvrij, standaard) of `none` — volledige vlaggentabellen, de indeling van de opnamemap en regels voor herverwerking staan in de [CLI-referentie](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Een DAQ-lichtsensor koppelen

Voor reflectie- en belichtingsgecorrigeerde voorbeelden zijn gegevens over neerwaarts vallend licht nodig van een DAQ-sensor (aangesloten via het tabblad **Lichtsensoren**):

* De **array-rij**in de zijbalk toont een**"DAQ · aan/uit"-knop** — *aan* wanneer een lichtsensor op array-niveau is ingesteld **of** wanneer een van de aangesloten camera’s een eigen sensor heeft; de tooltip geeft precies aan welke sensor welke camera van gegevens voorziet.
* Wijs de instelling toe voor de hele array in de array-instellingen → **Omgevingslichtsensor**→ vervolgkeuzelijst**Lichtsensor**. De selectie blijft behouden in het project, wordt doorgegeven aan elke camera in de array, en individuele camera’s kunnen deze nog steeds overschrijven met hun eigen sensor.
* De statusregel eronder geeft de actuele status weer: **Uit**→ "Wachten op eerste spectrum…" →**"Actief — alle camera’s in de reeks zijn verlichtingsgecorrigeerd"** → of, als er in de afgelopen 3 seconden geen nieuw spectrum is binnengekomen, een melding dat de gegevens verouderd zijn — de laatste meting blijft dan in gebruik (metingen vervallen nooit in het opnamepad).

Als er een sensor is toegewezen: het exporttype ‘Reflectance’ wordt beschikbaar, live-voorbeelden zijn belichtingsgecorrigeerd, voorspellende automatische belichting kan het spectrum gebruiken en elke reflectantie-opname schrijft de daadwerkelijk gebruikte DAQ-meting als een **`.daq`-bestand** in de `daq/`-map, zodat de opname later opnieuw kan worden verwerkt. Daarnaast geeft een `.daq.cameras.json`-bestand de namen aan van de camera’s waarvan de beelden door die meting worden beschreven; bij herverwerking koppelt Chloros de meting uitsluitend aan beelden van die camera’s.

## `array-connect` CLI-opties

| Vlag | Standaard | Beschrijving |
| --- | --- | --- |
| `--serials SN1,SN2,…` | alle LATTICE-camera’s automatisch detecteren (minimaal 2 nodig) | **Het eerste serienummer is de MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synchronisatielijn. |
| `--target-fps F` | auto | Vuurfrequentie van de master-trigger. |
| `--binning {1,2,4}` | auto | Hardware-binning. ‘Auto’ staat voor de volledige sensor (geen binning) wanneer een framesnelheid dit ondersteunt; ‘connect’ past nooit binning toe om de snelheid hoog te houden. |
| `--packed {auto,on,off}` | `auto` | [Gecomprimeerd 12-bits transport](#packed-12-bit-transport): `auto` comprimeert alleen wanneer dit nuttig is en de host het decomprimeren aankan. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Handmatige overschrijving van de synchronisatieniveaukeuze. |
| `--wire-ceiling-mbps MB_PER_S` | automatisch gedetecteerd | Bandbreedtebudget van de host in MB/s — de CLI-weergave van het veld **Wire Budget**. Verlaag deze waarde als de array beschadigde frames meldt. Wordt samen met het project opgeslagen, zodat deze bij een latere herverbinding wordt hersteld. |
| `--no-recommend` | uit | Sla de netwerkanalysestap over. |
| `--no-ptp` | uit | Schakel PTP uit (tijdstempels tussen camera’s zijn dan niet vergelijkbaar). |

`lattice array-list`, `array-status` en `array-disconnect` beheren de permanente sessie. De volledige referentie van de subcommando’s, inclusief uitlijning (`align-calibrate` / `align-apply`) en de netwerktools, staat in de [CLI-referentie § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); de SDK-equivalenten (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) staan in de [SDK-referentie](../reference/sdk-reference.md). Vanuit Python is het draadbudget `connect_array(..., wire_ceiling_mbps=120)`, en de indeling in ‘live corrupt’ en ‘never-arrived’ staat op [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
