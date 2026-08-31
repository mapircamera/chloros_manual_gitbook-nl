---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Ondersteunde camera’s

Chloros verwerkt beeldmateriaal van twee MAPIR-cameraseries op **alle platforms** (Windows, Linux amd64 en Linux arm64/Jetson):

* **Survey3** — Survey3W (breed) en Survey3N (smal) camera’s. Invoer: `RAW+JPG`.
* **LATTICE**— M3C- en M3M-multispectrale cameramodules. Invoer: opnames van `.tif`/`.tiff`. LATTICE-camera’s kunnen ook**live worden bediend** vanuit Chloros — via het tabblad ‘Camera’s’ in de GUI (Windows) of `chloros-cli lattice` / de Python SDK (Windows en Linux) — inclusief gesynchroniseerde multi-camera-opstellingen. Zie de [LATTICE-handleiding](lattice/).

De verwerkingspijplijn accepteert ook `.dng`-invoerbestanden.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Fabrikant</th><th width="250">Cameramodel</th><th width="138">Filtermodel</th><th width="187">Beeldtype</th></tr></thead><tbody><tr><td><strong>Fabrikant</strong></td><td><strong>Cameramodel</strong></td><td><strong>Filtermodel</strong></td><td><strong>Beeldtype</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

De LATTICE-lijn is een modulair multispectraal camerasysteem dat is gebaseerd op de Sony IMX265 global-shutter-sensor (3,1 MP, 3,45 µm pixels). Elke camera slaat zijn identiteit op als een modelstring:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros geeft deze weer met het voorvoegsel `LATT-` (bijvoorbeeld `LATT-M3M-L41-F550`), en de modelstring stuurt alles stroomafwaarts aan — sensorprofiel, bandindeling en kalibratie worden automatisch bepaald; er hoeft per camera niets te worden geconfigureerd. Het lensnummer is het **horizontale gezichtsveld in graden**: `L41` = smal 41°, `L87` = breed 87°.

Er zijn twee sensorconfiguraties:

| Configuratie | Sensor      | Filtertype                           | Banden per camera                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer-kleur | Drievoudige banddoorlaat                       | 3 spectrale banden uit één enkele belichting                                 |
| **M3M**       | Monochroom  | Enkel smalband-interferentiefilter | 1 gekalibreerde band — combineer meerdere M3M-camera’s voor vegetatie-indexen |

### M3C (Bayer) filteropties

| Filter | Banden (naam @ midden nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### M3M (mono) filtercatalogus — 23 SKU&#x27;s

Het F-getal is het SKU-label; de gemeten bandbreedte (ingestempeld op elk gekalibreerd exportproduct) is de filterscan per partij:

| SKU    | Centrum (nm, gemeten) | FWHM randen (nm) | Breedte (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominaal)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominaal)           | 867–893         | 26         |
| F905   | — (nominaal)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_&quot;De bandranden worden gemeten als de waarden voor de volledige breedte bij de helft van het maximum uit de filterscans per partij van MAPIR — dezelfde waarden die Chloros in elke gekalibreerde export verwerkt.&quot;_ &quot;— (nominaal)&quot; = nog geen lot-scan; voor die SKU’s is het vermelde middelpunt het SKU-nummer en is de breedte het cijfer van de fabrikant.

† &quot;De reflectie van F988 wordt gekalibreerd met behulp van een reflectiepaneel in de scène: de band ligt buiten het gekalibreerde bereik van de DAQ-lichtsensor, dus Chloros past je meest recente paneelopname toe en behoudt deze tussen de paneelmetingen door.&quot; Zie [Kalibratiedoelen](calibration-targets.md).

Zie de [LATTICE-handleiding](lattice/) voor live camerabesturing, arrays, netwerkconfiguratie en de radiometrische verwerkingsketen.
