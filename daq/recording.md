# Opname &amp; het .daq-formaat

Een `.daq`-bestand is het opnameformaat van de lichtsensor van Chloros: een **SQLite-database** met gekalibreerde spectrale frames van één DAQ-sensor. Neem er één op tijdens een opnamesessie en de reflectiepijplijn kan later elk beeld delen door de neerwaartse stralingsintensiteit die op dat exacte moment is gemeten.

## Wat een .daq-bestand bevat

| Eigenschap | Waarde |
| --- | --- |
| Container | SQLite-database, één bestand per sensor per opname |
| Bestandsnaam | Bevat de **sensor-ID**en een**tijdstempel**, bijv. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spectrum per frame | 135 punten, 340–1010 nm in stappen van 5 nm, plus CIE XYZ-tristimulus |
| Eenheden | Gekalibreerde spectrale stralingsintensiteit, **W/m²/nm** (fabriekskalibratiebundel + cap-profiel toegepast) |
| Ingeschreven metadata | Sensor-ID (de sleutel voor het ophalen van de fabriekskalibratie van dat apparaat) en het geldende cap-profiel — zie [Cap-profielen &amp; gekalibreerd bereik](caps-and-range.md) |

Het formaat is identiek voor DAQ-U, DAQ-M en DAQ-E, dus voor de verdere verwerking maakt het niet uit welk transportapparaat de gegevens heeft geregistreerd.

Voor gekalibreerde registratie is de fabriekskalibratiebundel van de sensor vereist. Voor DAQ-U en DAQ-M haalt de backend de bundel op uit de cloud van MAPIR op basis van de sensor-ID (de opname wordt geweigerd als dit niet lukt); DAQ-E-units zijn hiervan vrijgesteld omdat zij hun kalibratie op het apparaat zelf bewaren.

## Opnemen via de GUI

Voor het opnemen via de GUI is een **geopend project** vereist (anders zijn de opnameknoppen uitgeschakeld):

* **Alles opnemen / Alles stoppen** — bovenaan de zijbalk ‘Lichtsensoren’; start of stopt in één keer een `.daq`-opname op elke aangesloten sensor.
* **Opnemen / Opname stoppen** — per sensor, in het instellingenvenster met het tandwielpictogram. Tijdens het opnemen verschijnt een rode „REC“-indicator in de live-informatierijen van de sensor.

Bestanden worden opgeslagen in `<project>/light_sensor/`, en wanneer een opname stopt — hetzij via ‘Stoppen’, ‘Alles stoppen’ of het loskoppelen van een opnamesensor — wordt het voltooide `.daq`-bestand **automatisch toegevoegd aan het geopende project**. Het verschijnt in de bestandslijst van het project zonder dat je het handmatig hoeft toe te voegen, en is dan al klaar voor reflectieverwerking.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Opnemen vanaf de CLI

De CLI neemt op via de sensorpool van de backend (de backend moet actief zijn — deze opdrachten zijn ‘thin’ HTTP-clients):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Haal de `--sensor-id`-waarde op uit `chloros-cli daq pool-list`. Twee standaardinstellingen die de moeite waard zijn om te kennen:

| Optie | Standaard |
| --- | --- |
| `--duration` | `0` — opnemen tot `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` op het bestandssysteem van de **backend**, niet dat van CLI |

Het onderscheid in de uitvoermap is van belang wanneer de CLI is gericht op een backend op een andere machine: het bestand komt terecht op de plek waar de backend draait.

## Opname vanuit Python

`DAQSensorSession` (geretourneerd door `chloros_sdk.connect_daq_sensor()`) geeft dezelfde gepoolde opname weer: `record_start(output_dir=None, device_name=None)` retourneert het bestandspad, `record_stop()` retourneert `{path, rows}`. Zie de [SDK-referentie](../reference/sdk-reference.md) voor de volledige sessie API. De direct-hardware-klassen van SDK (alleen bij installaties op de desktop) schrijven opnames standaard naar `~/Documents/DAQ/`; voor vrijgegeven builds is het hierboven genoemde verzamelpad de ondersteunde route.

## Een .daq-bestand gebruiken tijdens de verwerking

Om reflectantie uit beeldmateriaal te genereren, heeft Chloros de neerwaartse stralingsintensiteit nodig die is afgestemd op elke belichting:

* **Bewaar het `.daq`-bestand samen met het beeldmateriaal.**Tijdens de verwerking bepaalt de pijplijn automatisch de**naar tijdstempel afgestemde neerwaartse straling** op basis van een opgenomen `.daq` (elk DAQ-model) — of op basis van een DAQ-M-native `.csv` — die naast de beelden wordt aangetroffen. GUI-opnames voldoen hier automatisch aan, aangezien ze direct na het stoppen aan het project worden toegevoegd.
* **Kalibratie wordt op verzoek opgehaald.** Als een fabriekskalibratiebundel per camera of per DAQ nog niet lokaal in de cache is opgeslagen, haalt Chloros deze bij het eerste gebruik automatisch op uit de cloud van MAPIR (eenmalig internetverbinding vereist; opgeslagen in de cache onder `~/.chloros/`).
* **Live-opnames schrijven hun eigen sidecar.** Voor elk live vastgelegd reflectiebeeld wordt de daadwerkelijk gebruikte DAQ-meting opgeslagen als een `.daq`-sidecar naast de beelden, zodat de opname later opnieuw kan worden verwerkt zonder de originele opname.

## De stralingsintensiteit terugkrijgen

Bij het verwerken van een project worden ook alle lichtsensoropnames die het bevat, geëxporteerd naar een
`Light Sensor/`-map naast de beeldproducten. Hiervoor zijn **geen** beelden nodig: een
lichtsensor die afzonderlijk is ingezet, vormt een volledige opname, en een map die alleen `.daq`-
bestanden bevat, is een geldige invoer. Het rapport geeft aan hoeveel lichtsensorproducten er zijn opgeslagen.

| Product | Wat het is |
| --- | --- |
| `<name>_calibrated.daq` | Een opnieuw te verwerken archief volgens hetzelfde schema als een live-opname, waarin nu de kalibratiebundel wordt vermeld waarmee het is geproduceerd. Het opnieuw importeren ervan leidt **niet** tot een tweede kalibratie. |
| `<name>_calibrated.csv` | Spectrale stralingsintensiteit in W/m²/nm op het eigen golflengteraster van de sensor, één rij per meting, plus fotometrische kolommen: totaal vermogen, fotopische en scotopische lux, PPFD met de verdeling in blauw/groen/rood, en piekgolflengte. |

Een DAQ-U of DAQ-M waarvan de kalibratiebundel niet kan worden opgehaald — u bent offline, of
er is geen kalibratie voor die sensor in het bestand — wordt **met opgave van reden overgeslagen** en nooit weggeschreven
als een „gekalibreerd“ bestand met ruwe telwaarden. Maak verbinding met het internet en voer de bewerking opnieuw uit. Een DAQ-E
heeft zijn eigen kalibratie, dus deze is alleen nodig wanneer het apparaat niet is aangesloten en
er lokaal niets in de cache is opgeslagen.

### DAQ-A: ruwe tellingen, en waarom dat het juiste antwoord is

De **DAQ-A** dateert van vóór het systeem met kalibratiebundels per seriële poort en heeft geen bundel om
op te halen. Dat is geen vergissing: een DAQ-A wordt in het veld gekalibreerd aan de hand van een
reflectiedoel, en bij kalibratie op basis van een doel is alleen de *relatieve*
respons van de sensor nodig — en dat is precies wat de ruwe telwaarden zijn. Chloros kalibreert er tegenwoordig mee.

Een DAQ-A-opname kan dus wel worden geëxporteerd, maar onder een andere naam:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, niet `_calibrated` — een andere bestandsnaam in plaats van een vlag in het bestand zelf,
omdat de claim intact moet blijven wanneer het bestand als naakte naam per e-mail wordt doorgestuurd. De `.csv`
header vermeldt `raw spectral sensor counts (NOT irradiance)` en waarschuwt dat de waarden
vergelijkbaar zijn **binnen** het bestand en niet tussen sensoren onderling. De kolommen die alleen
betekenis hebben voor werkelijke stralingsintensiteit — totaal vermogen, lux, PPFD — worden leeg gelaten in plaats van
berekend op basis van tellingen.

Oudere DAQ-A-SD-opnames (schema v1.01 / v1.02) registreren alleen de schrijftijd van het bestand, geen
tijdstempel per meting. Chloros zal beelden hier niet mee matchen — het koppelen van een frame aan een
schrijftijd zou onjuist zijn zonder dat het er ooit verkeerd uitziet — maar de export leest ze prima en
het CSV geeft aan op welke klok het staat.

Voor het volledige verhaal over reflectantie — één sensor met een camera, en twee sensoren voor omgeving/object — zie [Reflectantie-workflows](reflectance.md).
