# Het tabblad DAQ in Chloros

Het tabblad DAQ — met de naam **Lichtsensoren** in de zijbalk van Chloros — is het live bedieningspaneel voor [DAQ-U-, DAQ-M- en DAQ-E-lichtsensoren](README.md): sluit sensoren aan via elk willekeurig transportprotocol, bekijk gekalibreerde spectra in realtime, bereken live reflectie met een sensorpaar en sla `.daq`-bestanden rechtstreeks op in je project.

Het tabblad wordt beschikbaar zodra de Chloros-backend volledig is opgestart. De grafieken op het tabblad worden gevoed door de DAQ-service van Chloros via een liveverbinding die automatisch opnieuw verbinding maakt (met een backoff van 2–10 s) als deze wordt onderbroken; zolang de service niet bereikbaar is, staat in de statusregel van een sensor **Geen server**.

De lay-out bestaat uit een **sensorzijbalk**(één rij per aangesloten sensor) plus een**grafiekgebied** (één grafiektegel per sensor of groep).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Een sensor aansluiten

Klik op **Sensor aansluiten** bovenaan de zijbalk. Het aansluitvenster wordt geopend in het hoofdgedeelte (of als een overlay wanneer u een andere sensor toevoegt — in dat geval verschijnt er een knop Annuleren).

| Bedieningselement | Gedrag |
| --- | --- |
| **Apparaattype** | `DAQ-U (USB)` (standaard), `DAQ-M (Bluetooth)` of `DAQ-E (Ethernet)`. Bij het wisselen wordt het scannen voor het nieuw geselecteerde transport opnieuw gestart. |
| **Poort / BLE-apparaat / Hostnaam / IP** | Geeft een lijst weer van gedetecteerde apparaten als `device - description`; het eerste item dat als sensor wordt herkend, wordt automatisch geselecteerd. Tijdens het scannen wordt `Scanning...` (USB), `Scanning (N)...` met een aftelling van 8 seconden (BLE) of `Discovering ethernet sensors (N)...` met een aftelling van 5 seconden (Ethernet). Bij geen resultaten wordt `No ports` / `No BLE devices` / `No ethernet sensors found` weergegeven. |
| **↻ Vernieuwen** | Scant het geselecteerde transport onmiddellijk opnieuw (uitgeschakeld tijdens een BLE-/Ethernet-scan). |
| **Verbinden** | Wordt ingeschakeld zodra een apparaat is geselecteerd; het label verandert in `Connecting...` terwijl de verbinding tot stand wordt gebracht. |

Het zoeken naar apparaten vindt alleen plaats **terwijl het verbindingsvenster op het scherm staat** en wordt elke 15 seconden herhaald, uitsluitend voor het geselecteerde transport — het openen van het tabblad alleen is niet voldoende om te scannen. Bij een mislukking verschijnt in het venster: *&quot;Verbinding mislukt. Probeer de sensor los te koppelen en opnieuw aan te sluiten, en klik vervolgens opnieuw op Verbinden.&quot;*

De zijbalk wordt automatisch geopend wanneer je eerste sensor verbinding maakt.

{% hint style="info" %}
**Wordt de DAQ-E niet weergegeven?** De DAQ-E heeft geen status-LED — controleer de PoE/link-indicator op de switch of injectorpoort waarop hij is aangesloten, en wacht na het inschakelen enkele seconden totdat hij is opgestart. De Chloros-machine moet zich in hetzelfde broadcastdomein bevinden (mDNS gaat niet door routers heen). Accepteer op Windows de Defender-firewallprompt de eerste keer dat Chloros zijn multicast-sockets koppelt (mDNS UDP 5353, DAQ-E-gegevens UDP 5002, PTP UDP 319/320). Twee DAQ-E-units op één LAN worden afzonderlijk gedetecteerd, elk onder hun eigen `daq-e-<id>.local`-hostnaam.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Apparaattype biedt DAQ-U (USB), DAQ-M (Bluetooth) en DAQ-E (Ethernet)</figcaption></figure>***

## De sensorzijbalk

Elke aangesloten sensor krijgt een rij (plus één rij per Ambient+Object-groep). Rijen kunnen door slepen worden herschikt, en hun volgorde bepaalt ook de volgorde van de grafiektegels. Klik op een rij om die sensor/groep tot de actieve grafiek in de lijstweergave te maken.

| Element | Betekenis |
| --- | --- |
| Gekleurde linkerrand | De kleur van de grafiek van de sensor. |
| Transportbadge | `DAQ-U` / `DAQ-M` / `DAQ-E`, of een groene `REF`-badge voor een Ambient+Object-reflectiegroep. |
| Apparaatnaam | Standaard ingesteld op het serienummer van de sensor (de vaste identificatie voor kalibratie, `.daq`-bestandsnamen en importafstemming); aangepaste namen blijven per project behouden. |
| **Gekalibreerd**-pillen (groen) | Wordt weergegeven wanneer het fabriekskalibratiebundel van de sensor is geladen, d.w.z. dat spectra echte W/m²/nm zijn. |
| **Update beschikbaar**-knop (oranje, alleen DAQ-E) | De huidige firmware is ouder dan de image die bij deze Chloros-build is meegeleverd. Tijdens een update wordt de voortgang in realtime weergegeven (`Flashing… N%`, `Restarting sensor…`, vervolgens `Updated X → Y` of `Failed`). |
| Oog | Schakelt de zichtbaarheid van deze sensor op de grafiek in of uit. |
| Tandwiel | Opent het instellingenvenster per sensor (hieronder). |
| ✕ (rood) | Verbreekt de verbinding met de sensor, of verwijdert een Ambient+Object-groep. |

Boven de rijen bevinden zich twee knoppen:

* **Sensor verbinden** — opent het verbindingsvenster (wordt tijdens het verbinden omgedoopt tot `Connecting...`).
* **Alles opnemen / Alles stoppen**— start of stopt een `.daq`-opname op**elke**aangesloten sensor. Vereist ten minste één sensor**en een geopend project** (tooltip: „Open een project om op te nemen“); de knop wordt rood zolang er een opname loopt.

In de lege toestand staat er „Geen sensoren aangesloten“.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Instellingen per sensor (het tandwielvenster)

Open dit via het tandwielpictogram in een sensorrij. Inhoud in volgorde:

* **Informatierijen** — Apparaattype (DAQ-U/M/E), Verbinding (`Serial (USB)` / `Bluetooth` / `Ethernet`), poort (COM-poort, BLE-adres of host) en serienummer.
* **Kalibratierapport: downloaden** — haalt het NIST-traceerbare kalibratiecertificaat (PDF) van dit apparaat op en opent dit in uw PDF-viewer. Beschikbaar zodra het seriële nummer bekend is; het certificaat wordt bij de eerste verbinding in de cache opgeslagen.
* **Apparaatnaam** — klik op het potlood om de naam te wijzigen; blijft per project behouden.
* **Kleur grafieklijn** — kleurenstaal; blijft per project behouden.
* **Integratietijd (ms)**— schuifbalk + getal,**1–500 ms**, standaard**32 ms**. Uitgeschakeld wanneer AE is ingeschakeld.
* **Frame-gemiddelde**— schuifbalk + getal,**1–50 frames**, standaard**20**.
* **AE: AAN/UIT**— schakelaar voor automatische belichting;**standaard AAN** bij het verbinden. Schakel deze uit om de integratietijd handmatig in te stellen.
* **Streaming stoppen / Streaming starten** — de livestream pauzeren of hervatten.
* **Opnemen / Opname stoppen** — `.daq`-opname per sensor (vereist een geopend project).
* **Cap** — het cap-correctieprofiel (volgende sectie).
* **Live-informatierijen** — Integratietijd (ms), FPS, Samples, Opname (rood `REC` of `Off`) en Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Alleen DAQ-E: rijen voor netwerk, firmware en PTP

* **Hostnaam / IP** — het huidige adres van het apparaat.
* **Firmware** — actuele firmwareversie, plus een actiecel: er<version\>

verschijnt</version\>

een<version\>

knop</version\>

**Update naar \<version\>** wanneer deze Chloros-build een nieuwere DAQ-E-firmware-image bevat. De update wordt in ongeveer 30 seconden via het netwerk geïnstalleerd; de sensor start opnieuw op en maakt automatisch opnieuw verbinding, en bij een onderbroken overdracht blijft de huidige firmware intact. De voortgang wordt live weergegeven (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), en het veld geeft `Up to date` weer wanneer dit actueel is.
* **PTP-synchronisatie** — de actuele PTP-status (valt terug op `unknown`). DAQ-E-firmware v1.2.0+ neemt deel aan IEEE 1588 PTPv2 als een klok die uitsluitend als slave fungeert; de backend van de host met Chloros is de PTP-grandmaster, en elke DAQ-E- en LATTICE-camera op het LAN is hieraan ondergeschikt in domein 0, waarbij de tijdstempels binnen ongeveer 1 ms worden gehouden.

Voor een Ambient+Object-groep toont het tandwielmenu alleen de bronsensoren van de groep, de apparaatnaam en de kleur van de grafieklijn.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Kapselectie

De vervolgkeuzelijst **Cap** geeft aan Chloros door welke fysieke kap over de diffuser van de sensor is geplaatst, en past het in de fabriek gemeten correctieprofiel van die kap toe op elk spectrum. De keuzemogelijkheden zijn afhankelijk van het model:

| Model | Kapkeuzes |
| --- | --- |
| DAQ-U | Geen (blote sensor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (cosinuscorrector) |
| DAQ-M | Geen (blote sensor), Sunshine (cosinuscorrector) |
| DAQ-E | Geen (blote sensor), FOV 15°, FOV 45°, FOV 90°, Sunshine (cosinuscorrector) |

**De standaardinstelling voor elk model is Zonneschijn (cosinuscorrectie)** — MAPIR levert elke DAQ met de Sunshine-kap geïnstalleerd, en dit is de standaardconfiguratie voor buitengebruik: een 180° hemisferisch gezichtsveld met een cosinusfout van ≤ ±4 % tot 60° en ≤ ±4,5 % tot 70° (niet aanbevolen bij een zonnehoogte van minder dan ~15°), met een ontwerpgebonden demping (~12×). Uw keuze blijft in het project opgeslagen.

{% hint style="warning" %}
**De selectie van de kap moet overeenkomen met de fysieke kap.**Noch de sensor, noch de software kan detecteren welke kap is gemonteerd. De selectie bepaalt zowel de realtime correctie als de stempel die in elk `.daq`-bestand wordt geschreven — met de ~12×-demping van de Sunshine-kap zorgt een niet-aangegeven kapwisseling ervoor dat spectra met ongeveer die factor verkeerd worden gecorrigeerd. (Het verwijderen en opnieuw plaatsen van dezelfde kap leidt tot een herhaling van ongeveer 1,5 %.) Kies alleen**Geen (blote sensor)** wanneer de kap fysiek is verwijderd; op een DAQ-E past „Geen“ nog steeds een fabrieksgeometrieprofiel toe voor de verzonken glazen diffuser — het is geen „no-op“ — en een blote DAQ-E is een benchconfiguratie, geen ondersteunde veldconfiguratie.
{% endhint %}

{% hint style="info" %}
Upgraden vanaf een eerdere handleiding: de schakelaar „Sunshine Diffuser Installed” aan de browserzijde uit versie 1.1.0 is verdwenen. De afhandeling van de kap gebeurt nu via dit kapprofiel per sensor, dat aan de serverzijde wordt toegepast.
{% endhint %}

***

## Het grafiekgebied

Een vaststaande balk bovenaan bevat een **schakelaar voor lijst- ⇄ rasterweergave**en een schuifregelaar voor**grafiekzoom** (tegelgrootte 200–2000 px). De weergave schakelt automatisch over naar rasterweergave wanneer er meer dan één grafiekgroep is, en terug naar lijstweergave bij één of minder. De weergavemodus en grafiekgrootte blijven per project behouden.

De **spectrumgrafiek** voor elke sensor toont:

* **X-as** — Golflengte (nm). Het sensorraster is 340–1010 nm met stappen van 5 nm (135 punten), geïnterpoleerd tot 1 nm voor weergave.
* **Y-as** — Vermogen (W/m²), met een automatisch SI-voorvoegsel (m/µ/n) gekozen op basis van de piek. Spectra zijn radiometrisch gekalibreerde spectrale stralingsintensiteit (W/m²/nm) op alle drie de transporten.
* Een regenboogkleurige spectrale vulling onder een enkele curve; meerdere sensoren op één grafiek worden weergegeven als gekleurde lijnen met gedimde vullingen.
* **Muisaanwijzer**— een verticale cursor met golflengte en waarde per sensor;**verslepen** om in te zoomen (er verschijnt een uitzoomknop wanneer ingezoomd).
* Een **+**-knop (alleen in rasterweergave) om een sensor aan deze grafiek toe te voegen of een groep aan te maken (zie hieronder).
* De apparaatnaam gecentreerd bovenaan, en een draaiknop totdat het eerste frame binnenkomt.

**Verzadiging** wordt niet op de grafiek zelf aangegeven: een verzadigde sensor toont rode `SATURATED`-statustekst en een rode `Saturated: Yes`-rij in de live-gegevenstabel. Verlaag de integratietijd of schakel AE opnieuw in om dit te verhelpen.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Live-gegevenstabel (lijstweergave)

Onder de grafiek in lijstweergave, elke 500 ms vernieuwd:

* **Alle modellen**: Lichtkleurstaal (sRGB vanuit CIE XYZ), Verzadigd (Ja/Nee), CIE 1931 X/Y/Z, Chromaticiteit x/y, CIE u′/v′, CCT (K), CRI (Ra), Dominante golflengte (nm), Piekgolflengte (nm), Excitatiepuurheid, Duv, CIE L\*/a\*/b\* en Munsell H/V/C.
* **Alleen gekalibreerde sensoren**(elk van de DAQ-U / DAQ-M / DAQ-E zodra het fabriekskalibratiebundel is geladen — het groene**Gekalibreerd**-pictogram in de sensorrij is de herkenning): Totaal vermogen (W/m²), fotopische lux (lx), scotopische lux (lx), S/P-verhouding, PPFD plus PPFD Red/Green/Blue (µmol/m²/s), en de opische stralingsintensiteiten — S-kegel, melanopisch, rhodopisch, M-kegel, L-kegel (allemaal W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Reflectiegroepen (Omgeving + Object)

Twee aangesloten sensoren kunnen worden gecombineerd tot een live weergave van de reflectie — zonder camera:

1. Klik in de rasterweergave op **+**op een grafiektegel en kies**Omgevingslicht + Object combineren**.
2. Kies een **Omgevingslichtbron**-sensor en een**Objectscanner**-sensor (twee verschillende sensoren) en klik vervolgens op**Aanmaken**.

Chloros berekent R(λ) = object(λ) / omgevingslicht(λ) per golflengte op basis van de twee live streams (0 wanneer omgevingslicht ≤ 0). Het label van de groep volgt de kalibratieklasse van de sensoren:

* Beide sensoren gekalibreerd (bundel geladen) → **&quot;Schijnbare reflectie&quot;**.
* Een van beide sensoren niet gekalibreerd → **&quot;Relatieve reflectie&quot;**.

De groep verschijnt als een groene `REF`-rij in de zijbalk en in een eigen grafiek (regenboogvulling, waarden bij aanwijzen tot op 4 decimalen, zoomen door te slepen).

Het **+**-menu biedt ook**Nieuwe sensor toevoegen** met drie plaatsingsopties: *Nieuwe sensor combineren* (toevoegen aan deze grafiek), *Bestaande sensor hierheen verplaatsen* of *Nieuwe sensor bekijken* (eigen grafiek).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Vegetatie-index tabel

In de lijstweergave staat onder de grafiek van een reflectiegroep een vegetatie-indxtabel, berekend op basis van de live-reflectie bij de bandcentra **blauw 450 / groen 550 / rood 670 / NIR 800 nm** (waarden tot op 4 decimalen, `---` wanneer niet berekenbaar; plaats de muisaanwijzer op een indexnaam voor de volledige naam):

* **Altijd weergegeven** (schaalonafhankelijk, elke sensorgecombinaatie): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Alleen wanneer beide sensoren vermogensgekalibreerd zijn** (beide bundels geladen): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## `.daq`-bestanden opnemen

* Voor het opnemen is een **geopend project** vereist — anders zijn zowel de knop ‘Alles opnemen’ (zijbalk) als de opnameknop per sensor uitgeschakeld.
* Bestanden worden opgeslagen als **`<project folder>/light_sensor/`**; bestandsnamen bevatten het sensor-ID en een tijdstempel, en de apparaatnaam wordt samen met de opname opgeslagen.
* Wanneer een opname stopt (via ‘Stop’, ‘Alles stoppen’ of een verbroken verbinding tijdens het opnemen), wordt het voltooide `.daq`-bestand **automatisch aan het geopende project toegevoegd** — het verschijnt in de bestandslijst van het project zonder dat het handmatig hoeft te worden toegevoegd, klaar om te dienen als downwelling-gegevens voor [reflectieverwerking](README.md).
* Tijdens het opnemen verschijnt er een rode `REC`-indicator in de live-rijen van het instellingenvenster.

Voor kwantitatieve stralingsintensiteitswaarden moet je minimaal 15 seconden aan gegevens middelen — dit is een kenmerk van het instrument, geen defect.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Lay-outs met meerdere sensoren en projectbehoud

* Combineer meerdere sensoren in één grafiek (gedeelde assen), houd afzonderlijke grafieken bij (automatische rasterindeling), verplaats sensoren tussen grafieken, herschik rijen/tegels door ze te verslepen en verberg afzonderlijke sensoren met de oogschakelaar.
* Per project blijven de volgende gegevens van Chloros behouden: apparaatnamen, grafiekkleuren, grafiekgrootte, weergavemodus en de instellingen van elke sensor (integratietijd, framegemiddelde, AE-status, cap-selectie).
* **Bij het opnieuw openen van een project worden de sensoren automatisch opnieuw verbonden** op basis van het adres — COM-poort voor DAQ-U, BLE-apparaat voor DAQ-M, mDNS-hostnaam voor DAQ-E (wordt omgezet, zelfs als het IP-adres van het apparaat is gewijzigd) — en past het opgeslagen cap-profiel, het framegemiddelde, de AE-status en de handmatige integratietijd van elke sensor opnieuw toe.***

## Camera koppelen (DLS)

Er hoeft niets te worden gekoppeld. In tegenstelling tot DLS-workflows voor drones, waarbij een lichtsensor vooraf aan een camera wordt gekoppeld, koppelt Chloros DAQ-gegevens achteraf aan beeldmateriaal: tijdens het importeren/verwerken worden de metingen van `.daq` geïnterpoleerd naar de belichtingstijdstempel van elke opname. Neem op met een willekeurige aangesloten sensor (de `.daq` wordt automatisch aan het project toegevoegd), en de reflectieverwerking zoekt de juiste meetwaarden op basis van de tijd — zie [DAQ-lichtsensoren](README.md) voor informatie over hoe de neerwaartse gegevens worden gebruikt.</version\>
