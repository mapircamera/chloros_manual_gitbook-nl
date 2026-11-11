# Cap-profielen en gekalibreerd bereik

> De afdekkappen zelf — welke afdekkap bij welke sensor wordt geleverd, hoe ze worden gemonteerd en hun optische gedrag — worden beschreven in de **[DAQ-gebruikershandleiding](https://mapir.gitbook.io/daq)**. Deze pagina gaat over het *aangeven* van de gemonteerde afdekkap aan Chloros
; dit is wat de correctie juist maakt.

De radiometrische fabriekskalibratie van elke DAQ-lichtsensor beschrijft de *kale* sensor. De fysieke kap die over de diffuser is geplaatst, verandert welk licht de sensor opvangt, dus past Chloros
een in de fabriek gemeten **kapcorrectieprofiel** toe bovenop de kalibratiebundel. Het aangeven van de juiste kap is een onderdeel van het verkrijgen van gekalibreerde gegevens — op deze pagina wordt beschreven welke kappen er per model bestaan, hoe je ze moet aangeven en wat het gekalibreerde spectrale bereik van de sensor daadwerkelijk is.

## Beschikbaarheid van kapjes per model

| Kapprofiel (`cap_id`) | Fysiek kapje | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Sunshine cosinus-correctiekap (**standaard op elk model**) | Ja | Ja | Ja |
| `fov_15` / `fov_45` / `fov_90` | FOV-beperkende kegels (15° / 45° / 90°) | Ja | — | Ja |
| `fov_30` / `fov_60` | Kegels voor beperking van het gezichtsveld (30° / 60°) | Ja | — | — |
| `none` | Geen kap gemonteerd | — | — | Ja |

Modelspecifieke opmerkingen:

* **DAQ-M heeft één kapprofiel: `sunshine_cosine`.** &#x27;Bare-plus-Sunshine-kap&#x27; is de productdefinitie, en een &#x27;bare&#x27; DAQ-M heeft geen geometrieprofiel nodig.
* **Een &#x27;bare&#x27; DAQ-U is echt &#x27;bare&#x27;** — deze heeft helemaal geen geometrieprofiel nodig, en daarom bestaat er geen `none`-profiel voor.
* **`none` op een DAQ-E is GEEN no-op.** De verzonken, met glas afgedekte diffusor van de DAQ-E heeft een eigen, daadwerkelijke geometrische correctie, dus „geen kap“ is op dit model zelf een gemeten profiel.
* Een **kale DAQ-E kan bij geen enkele elevatie direct zonlicht meten** — de Sunshine-kap is de standaardconfiguratie. Plan geen buitenwerk met een kale DAQ-E.

In de instellingen per sensor van de GUI (tandwielpictogram op het tabblad ‘Lichtsensoren’) biedt de vervolgkeuzelijst **Cap** ook de optie ‘Geen (onbedekte sensor)’ op de DAQ-U en DAQ-M — bij deze twee modellen betekent ‘onbedekt’ simpelweg dat er geen kapcorrectie wordt toegepast, zoals hierboven vermeld. Kies deze optie alleen wanneer de kap fysiek is verwijderd.

## De kap aangeven — en waarom dit belangrijk is

**De opgegeven `cap_id` moet overeenkomen met de kap die fysiek op de sensor is geplaatst.** Noch de sensor, noch de software kan de gemonteerde kap detecteren. De specificatie is bepalend voor twee zaken:

1. De **live-correctie** die op elk spectrum wordt toegepast.
2. De **kapcode die in elke `.daq`-opname wordt vastgelegd**, waarop de verdere reflectieverwerking vertrouwt.

De Sunshine-kap dempt het licht **ontwerpgewijs met ongeveer een factor 12**, dus als je met de verkeerde kap registreert, worden de spectra met ongeveer die factor verkeerd geschaald. Geef kapwijzigingen onmiddellijk door.

### De kap instellen

GUI: tabblad Lichtsensoren → tandwielpictogram op de sensorrij → vervolgkeuzemenu **Kap**. De standaardinstelling voor elk model is `sunshine_cosine`, een aanname (DAQ-U, DAQ-M en DAQ-E worden zonder cap geleverd, en een opname geeft aan dat de cap werd aangenomen totdat je er een kiest), en de selectie blijft behouden voor het project.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend moet actief zijn):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

De CLI
accepteert syntactisch de volledige `cap_id`-lijst (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); elk profiel wordt bij het verbinden gevalideerd aan de hand van het model van de sensor, dus een onbeschikbare cap-id (bijvoorbeeld een E-only-id op een DAQ-U) leidt tot een duidelijke foutmelding in plaats van een onjuiste correctie. De standaardinstelling van de backend wanneer er niets wordt doorgegeven, is `sunshine_cosine`.

Python
SDK
: geef de cap bij het verbinden een naam met `connect_daq_sensor(cap_id="sunshine_cosine")`, of wijzig deze tijdens de uitvoering met `DAQSensorSession.set_cap("fov_45")`; zie de [SDK
-referentie](../reference/sdk-reference.md#daq-sensor-sessions).

Geavanceerd: profielen worden meegeleverd in de Chloros
-installatie onder `daq/cap_profiles/<u|m|e>/<cap_id>.json` en kunnen per gebruiker worden overschreven via `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Los van de limieten krijgen sensoren die nog nooit opnieuw zijn gekalibreerd automatisch een kleine, uit de vloot afgeleide verfijning van de donkere offset — hier is geen actie van de gebruiker voor nodig.

## DAQ-E-S: vaste diffuser en kanteling

De **DAQ-E-S** is een DAQ-E met twee verschillen die voor Chloros
van belang zijn. Al het andere (detectie, verbinding, PTP, opname, apparaatinstellingen) werkt precies hetzelfde als bij een DAQ-E.

* **De diffusor is vast gemonteerd, dus er is geen cap-correctie nodig.** De diffuser zat al op het apparaat toen de fabriekskalibratie werd gemeten, zodat die kalibratie al de werkelijke neerwaartse stralingsintensiteit weergeeft. Chloros
past er geen cap-profiel op toe, wat er ook wordt gevraagd: een cap die ervoor is bedoeld, wordt genegeerd met een melding, de opnames geven aan dat de cap afkomstig is van het model, en de waarschuwing voor een veronderstelde cap wordt hiervoor nooit geactiveerd. In Projectinstellingen → DAQ-lichtsensor wordt in de rij geen cap weergegeven, en de projectbrede cap-overschrijving is hier niet van toepassing. Pas geen cap handmatig toe op de gegevens: de Sunshine-correctie zou dan ongeveer 11× te hoog uitvallen.
* **Het apparaat is voorzien van een 3-assige versnellingsmeter.** De cosinusrespons van de diffusor is alleen geldig zolang deze horizontaal staat; een gekantelde sensor rapporteert dus een verkeerde stralingsintensiteit zonder andere symptomen. Elke DAQ-E-S-meting bevat daarom de kantel-, rol- en pitchhoeken, en een `.daq` die hiermee is geregistreerd, slaat deze op naast elk spectrum (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), zodat je een wolk kunt onderscheiden van een verschoven montuur.

Zie [DAQ-E-S in de CLI
-referentie](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor) voor alle houdingsvelden en hoe een fout in de houdingsstream wordt gemeld.

## Prestaties van de Sunshine Cap (de buitenconfiguratie)

Cijfers waarop je procedures kunt baseren:

| Eigenschap | Waarde |
| --- | --- |
| Gezichtsveld | 180° hemisferisch |
| Cosinusresponsfout | ≤ ±4 % tot 60° invalshoek; ≤ ±4,5 % tot 70° |
| Grens voor lage zonstand | Niet aanbevolen bij een zonnehoogte van minder dan ~15° |
| Demping | ~12× (volgens ontwerp) |
| Herhaalbaarheid bij hermontage van de kap | ≈ 1,5 % |
| Kwantitatieve stralingsintensiteit | Gemiddelde van **≥ 15 s** aan metingen (kenmerk van het instrument, geen defect) |

Gebruik voor elke kwantitatieve stralingsintensiteit — inclusief reflectiewaarden — een gemiddelde van ten minste 15 seconden aan metingen in plaats van één enkel beeld.

## Gekalibreerd spectraal bereik

| Eigenschap | Waarde |
| --- | --- |
| Spectrale bemonstering | 340–1010 nm in stappen van 5 nm (135 punten) |
| Radiometrisch gekalibreerd bereik | **~374–974 nm** (afgedwongen in de software) |

De sensor rapporteert het volledige raster van 340–1010 nm, maar de naar NIST herleidbare radiometrische versterking beslaat ~374–974 nm. Chloros
**weigert de absolute-reflectantie-indeling** voor elke cameraband waarvan minder dan de helft van het spectrale gewicht binnen dat bereik valt, en rapporteert de overslaagreden `dls-uncalibrated-band-<nm>` in plaats van een ongekalibreerd product te genereren. Van de leverbare camera-SKU’s valt alleen het F988-filter buiten dit bereik; hiervoor wordt in plaats daarvan de workflow met reflectantiepanelen gebruikt — zie [Reflectantie-workflows](reflectance.md).

Zie het [DAQ-overzicht](README.md) voor sensormodellen, transporten en sensor-ID’s. Zie [Opname en het .daq-formaat](recording.md) voor informatie over hoe de cap-stamp tijdens de verwerking wordt verbruikt.
