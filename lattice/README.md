# LATTICE-camera&#x27;s

LATTICE is het modulaire multispectrale camerasysteem van MAPIR voor beeldvorming in de landbouw en de wetenschap. Elke LATTICE-camera is gebaseerd op de Sony IMX265 global-shutter-sensor (**3,1 MP, 3,45 µm pixels**) en maakt via Ethernet verbinding als een**GigE Vision**-apparaat.

Chloros 1.2.0 bestuurt LATTICE-camera’s in realtime — detectie, live preview, opname en gesynchroniseerde opstellingen met meerdere camera’s — vanaf drie interfaces:

| Interface    | Waar                                                          | Platforms                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Tabblad **Camera’s** in de zijbalk van Chloros                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice`-opdrachtfamilie                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Op zoek naar de hardware?**Cameramodules, lenzen, filters en banden, frames en bevestigingsmateriaal, kabels, PoE- en triggerbekabeling worden beschreven in de [**LATTICE-gebruikershandleiding**](https://mapir.gitbook.io/lattice-camera). Dit hoofdstuk behandelt het aansturen van de camera’s vanuit Chloros.

LATTICE-opnames zijn standaard `.tif`/`.tiff`-bestanden, en Chloros verwerkt deze altijd uitgaande van de ruwe opname. Zie de [CLI-referentie](../reference/cli-reference.md) en de [SDK-referentie](../reference/sdk-reference.md) voor het volledige commando en het API-oppervlak.

## Twee sensorconfiguraties

| Configuratie | Sensor       | Filter                                | Wat één camera levert                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-kleur | drievoudig banddoorlaatfilter                |**Drie gekalibreerde banden uit één enkele belichting**                 |
| **M3M**| Monochroom   | enkel smalband-interferentiefilter |**Eén gekalibreerde band**; combineer meerdere M3M-camera’s voor indices |

Omdat een M3M-camera achter één enkel filter monochroom is, krijgt elke band zijn eigen belichting. Een M3C-camera bestrijkt alle drie zijn banden met één sensorbelichting.

## Modelstrings en naamgeving

Elke camera slaat zijn identiteit op in de GenICam `DeviceUserID` als een modelstring:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros geeft deze weer met het voorvoegsel `LATT-` (bijvoorbeeld `LATT-M3M-L87-F450`). Dezelfde tekenreeks ‘`LATT-…`’ wordt bij elke export in de EXIF-tag ‘`Model`’ geschreven en wordt in verwerkte projecten gebruikt als de naam van de uitvoermap van de camera.

| Component | Waarden                                                   | Betekenis                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayer-kleur / monochroom                                                                          |
| Lens      | `L41` / `L87`                                            | Het getal is het **horizontale gezichtsveld in graden**: L41 = smal (41°), L87 = breed (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) of `F<nm>` (M3M) | Zie [Filters &amp; spectrale banden](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

De modelstring bepaalt alles wat daarna volgt: Chloros bepaalt het sensorprofiel, de bandindeling en de fabriekskalibratie op basis van `DeviceUserID` + `DeviceSerialNumber`. Er hoeft per camera niets te worden geconfigureerd — zie [Camera’s aansluiten](connecting.md).

## Filters en banden

Bandcentra, FWHM-randen en de volledige M3M-catalogus met 23 SKU’s zijn productspecificaties en staan daarom in de hardwarehandleiding: [**Filters &amp; spectrale banden**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Wat aan de softwarekant van belang is: de filtercode in de modelstring bepaalt welke producten Chloros kan samenstellen. RGB-filtercamera’s (`FRGB`) leveren uitsluitend debayered en preview-producten — straling en reflectie per band zijn niet relevant voor een breedbandsensor, dus Chloros slaat deze over en geeft dit aan. Elk ander filter levert de volledige keten van stralingsintensiteit → reflectie → index op.

## Radiometrische kalibratie in een oogopslag

Elke LATTICE-camera wordt in de fabriek individueel gekalibreerd aan de hand van een naar NIST herleidbare keten en wordt geleverd met een certificaat per camera. Wat dit precies omvat, hoe het wordt gemeten en welke nauwkeurigheid je kunt opgeven, staat in de hardwarehandleiding: [**Radiometrische fabriekskalibratie**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Wat de software betreft, is het van belang dat Chloros de juiste kalibratie vaststelt wanneer een camera wordt aangesloten en de toegepaste coëfficiënten bij elke export vastlegt — zie [Camera’s aansluiten](connecting.md).

## In dit hoofdstuk

* [Camera’s aansluiten](connecting.md) — automatische detectie, het aansluitvenster in de GUI, CLI/SDK-equivalenten, en hoe de fabriekskalibratie wordt bepaald (pakket op de camera versus cloud) wanneer een camera verbinding maakt.

Verdere LATTICE-onderwerpen — camera-instellingen en livebesturing, opnamemodi, opstellingen met meerdere camera’s, en mono (M3M)-verwerking en indexen — worden behandeld in aparte hoofdstukken van deze handleiding, en de volledige lijst met commando’s is te vinden in de [CLI-referentie](../reference/cli-reference.md) en [SDK-referentie](../reference/sdk-reference.md).
