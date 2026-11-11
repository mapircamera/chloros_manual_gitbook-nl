# Werkprocessen voor reflectantie

Een DAQ-lichtsensor zet radiometrische beelden om in reflectantie. Er zijn twee verschillende werkprocessen:

1. **Enkele sensor** — één DAQ meet de neerwaartse stralingsintensiteit terwijl een camera opnamen maakt, en Chloros deelt de stralingsintensiteit van de camera door die referentiewaarde.
2. **Twee sensoren** — twee DAQ-sensoren, waarvan één naar de hemel kijkt en één naar een object, produceren een live spectrale reflectanciekromme zonder dat er een camera aan te pas komt.

## Eén sensor + camera (neerwaartse referentie)

De DAQ fungeert als de neerwaartse lichtsensor (DLS): de camera meet de opwaartse straling **L**(W/m²/sr/nm), de DAQ meet de neerwaartse stralingsintensiteit**E** (W/m²/nm) en Chloros berekent de reflectantie per band als:

> ρ = π · L / E

De DAQ-meting is altijd **qua tijdstempel afgestemd op de belichting** — daarom delen de DAQ en de camera’s een PTP-gesynchroniseerde klok (zie [DAQ-E Networking & Time Sync](ethernet-ptp.md)). Pas de Sunshine-cosinuskap aan voor buitenwerk en declareer deze correct; de kapdeclaratie schaalt E direct (zie [Cap Profiles & Calibrated Range](caps-and-range.md)). Houd bij kwantitatief werk rekening met de instrumentkarakteristiek: de kwantitatieve stralingsintensiteit is gebaseerd op een gemiddelde van ten minste 15 seconden aan metingen.

### Live-opname

Koppel de DAQ aan een camera in het tabblad ‘Camera’s’: het instellingenpaneel van elke camera heeft een **Lichtsensor**-dropdownmenu waarin alle aangesloten DAQ’s (DAQ-U/M/E) uit het tabblad ‘Lichtsensoren’ worden weergegeven; bij een gesynchroniseerde reeks wordt de selectie van de lichtsensor voor de hele reeks doorgegeven aan elk onderdeel (afzonderlijke camera’s kunnen dit nog steeds overschrijven). Zodra de koppeling is gemaakt, worden de spectra van de sensor ingevoerd in het DLS-slot van de camera en worden de reflectie-exportwaarden gedeeld door de bijbehorende meting. In een array gebruikt een onderdeel met een eigen sensor zijn eigen meetwaarde — voor zijn banden in de live composiet, zijn exportgegevens en zijn automatische belichting — en gebruiken de andere onderdelen die van de array.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Twee belangrijke gedragingen om te weten:

* **Geen DAQ gekoppeld → reflectantie wordt geweigerd, niet gesimuleerd.** *Chloros
* wijst het reflectantieproduct af en registreert de reden voor het overslaan, in plaats van stilletjes een minderwaardig product terug te geven.
* **De gebruikte meetwaarde blijft behouden.** Voor elk reflectieframe wordt de daadwerkelijk toegepaste DAQ-meting opgeslagen als een `.daq`-bestand in de map `daq/` van de opnames, zodat de opname later opnieuw kan worden verwerkt ([Opname & het .daq-formaat](recording.md)). Een `.daq.cameras.json` ernaast geeft de namen aan van de camera’s die het beschrijft, en bij herverwerking wordt het alleen gekoppeld aan de beelden van die camera’s.

### Verwerking van opgenomen beeldmateriaal

Voor verwerking na de vlucht neemt u tijdens de sessie een `.daq` op en bewaart u dit samen met de beelden — de verwerkingspijplijn koppelt de naar beneden gerichte straling automatisch aan de bijbehorende tijdstempels en haalt bij het eerste gebruik eventuele ontbrekende fabriekskalibratie op uit de cloud van MAPIR. GUI-opnames worden automatisch aan het geopende project toegevoegd zodra ze stoppen.

De reflectantiereferentie kan tijdens de verwerking worden geselecteerd — `--reflectance-source` op `chloros-cli process`, of de instelling voor de reflectantiebron in de projectinstellingen van de GUI:

| Waarde | Gedrag |
| --- | --- |
| `auto` (standaard) | Een QA-goedgekeurd kalibratiedoel binnen het beeld is de absolute referentie; DAQ-downwelling (ρ = π·L/E) is de terugvaloptie. Een doel dat de kwaliteitscontrole (QA) doorstaat met een vlag (een paneel is gevallen of een band is defect) krijgt niet automatisch de voorkeur: de ketens ‘doel’, ‘doel+DAQ’ en ‘alleen DAQ’ worden getest in volgorde van hun gemeten score op de buiten beschouwing gelaten panelen. |
| `daq` | DAQ-bepalend |
| `target` | Strikt in-frame doel; geen DAQ-vervanging |

Zie [Kalibratiedoelen](../calibration-targets.md) voor doelworkflows en het [hoofdstuk over LATTICE](../lattice/README.md) en de [CLI-referentie](../reference/cli-reference.md) voor de volledige verwerkingspijplijn. Gebruik bij het inlezen van geëxporteerde reflectantiepixels de aangegeven schaal (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — zie [Uitvoerbeeldformaten](../output-image-formats.md).

### Banden buiten het gekalibreerde bereik van de DAQ

Het radiometrisch gekalibreerde bereik van de DAQ is ~374–974 nm. Chloros weigert DAQ-gebaseerde reflectantie voor elke cameraband waarvan minder dan de helft van het spectrale gewicht binnen dat bereik valt, en rapporteert als reden voor het overslaan `dls-uncalibrated-band-<nm>`. Van de leverbare SKU’s heeft dit alleen gevolgen voor de F988: de reflectie van de F988 wordt gekalibreerd met behulp van een reflectiepanel in de scène; de band ligt buiten het gekalibreerde bereik van de lichtsensor van de DAQ, dus Chloros past uw meest recente panelopname toe en houdt deze vast tussen de panelmetingen door. Als een F988-camera uitsluitend in DAQ-modus wordt gebruikt, weigert Chloros de op DAQ gebaseerde reflectie voor die band met overslaagreden `dls-uncalibrated-band-988` — de paneelworkflow is de ondersteunde methode.

## Dubbele sensor (omgevingslicht + object)

Twee DAQ-sensoren — elk willekeurig paar, ongeacht het transport — leveren een live reflectiespectrum zonder camera: één sensor is gericht op de hemel (**Omgevingslichtbron**), één op het object (**Objectscanner**), en Chloros berekent per golflengte:

> R(λ) = object(λ) / omgevingslicht(λ)

(nul wanneer omgevingslicht ≤ 0).

### In de GUI

Als beide sensoren zijn aangesloten in het tabblad **Lichtsensoren**, open je het venster voor het toevoegen van sensoren (de „+“-knop op een grafiektegel in rasterweergave) en kies je**Omgevingslicht + Object combineren**. Selecteer de twee sensoren in de vervolgkeuzelijsten**Omgevingslichtbron**en**Objectscanner**en klik op**Maken**. De groep verschijnt als een eigen grafiek en als een rij in de zijbalk met een groene**REF**-badge.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Onder de reflectiegrafiek (lijstweergave) berekent een live **vegetatie-indxtabel** indices op basis van de curve, waarbij gebruik wordt gemaakt van bandcentra bij blauw 450 / groen 550 / rood 670 / NIR 800 nm. Op verhoudingen gebaseerde indices die de absolute schaal opheffen (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) worden altijd weergegeven; indices die absolute reflectie vereisen (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) verschijnen alleen wanneer beide sensoren stroomgekalibreerde modellen zijn.

### Schijnbare versus relatieve reflectantie — de naamgevingsregel

Chloros benoemt de output van de dubbele sensor op basis van wat het sensorpaar daadwerkelijk kan claimen:

| Sensorpaar | Label |
| --- | --- |
| Beide sensoren gekalibreerd — fabrieksbundel geladen | **Schijnbare reflectantie** |
| Een van beide sensoren niet gekalibreerd | **Relatieve reflectie** |

Alle drie de modellen zijn radiometrisch: zodra de fabriekskalibratiebundel van een sensor is geladen, zijn de spectra absolute waarden in W/m²/nm, dus een paar gekalibreerde sensoren levert een verhouding op tot een absolute schijnbare reflectie — het transport bepaalt dit niet. Een sensor die nog steeds ruwe tellingen streamt (geen bundel bereikbaar) degradeert het resultaat tot een relatieve curve (de spectrale vorm blijft geldig). Beide sensoren moeten correct gedeclareerde limieten hebben ([Limietprofielen & gekalibreerd bereik](caps-and-range.md)).

### Vanuit Python


Er is geen specifieke functie voor twee sensoren in de gezamenlijke SDK-interface: open twee sessies met `chloros_sdk.connect_daq_sensor()` en bereken zelf de verhouding tussen hun `latest()`-spectra, waarbij je dezelfde naamgevingsconventie hanteert. (Er bestaat ook een opnametool voor twee sensoren op de interne, directe hardware-interface van MAPIR, die ter volledigheid wordt vermeld in de [CLI-referentie](../reference/cli-reference.md) — deze maakt geen deel uit van de meegeleverde CLI; de hierboven beschreven GUI-workflow is de ondersteunde werkwijze.)
