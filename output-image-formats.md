---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Uitvoerbeeldformaten

Chloros exporteert verwerkte producten in vier bestandsformaten. Selecteer het formaat in Projectinstellingen (GUI), met `--format` (CLI) of met `export_format` (SDK). De CLI en SDK accepteren de onderstaande exacte tekenreeksen.

| Formaatstring | Extensie | Pixeltype | Pixelbereik | Opmerkingen |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(standaard)* | `.tif` | uint16 digitaal getal | 0 – 65535 | Aanbevolen voor fotogrammetrie / GIS. |
| `TIFF (8-bit)` | `.tif` | uint8 digitaal getal | 0 – 255 | Voor opnames die volledig uit 8-bits bestaan en alleen de 8-bits formaten bieden: Chloros voert de bitdiepte niet op. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% reflectie. Sommige toepassingen kunnen TIFF-bestanden met drijvende-kommagetallen niet lezen; de bestanden zijn groter. |
| `PNG (8-bit)` | `.png` | uint8 digitaal getal | 0 – 255 | Compressie zonder kwaliteitsverlies, geschikt voor weergave op het web en visualisatie. |
| `JPG (8-bit)` | `.jpg` | uint8 digitaal getal | 0 – 255 | Compressie met verlies, kleinste bestanden. |

## Waar de uitvoerbestanden worden opgeslagen

De bestanden worden opgeslagen in de projectmap, gegroepeerd per camera en vervolgens per bestandsformaat:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

De cameramap is `LATT-<sensor>-<lens>-F<filter>` voor LATTICE en `<model>_<filter>` (bijv. `Survey3N_RGN`) voor Survey3. **Elk geëxporteerd product behoudt de naam van het bronbestand — de map identificeert het product, niet een bestandsextensie.** Zie [Waar de uitvoerbestanden worden opgeslagen](reference/cli-reference.md) in de CLI-referentie voor de volledige regels.

## LATTICE-producten (vastleg- en exportniveaus)

Eén LATTICE-ruwframe wordt in één doorloop opgesplitst in alle aangevraagde producten. Elk producttype heeft zijn eigen schakelaar (GUI-selectievakjes, of CLI-opdrachten `--debayered` / `--preview` / `--radiance` / `--reflectance`, standaard allemaal ingeschakeld):

| Niveau | Inhoud | Gegevenstype |
| --- | --- | --- |
| `raw` | Bayer-gegevens rechtstreeks van de sensor (monochrome camera’s: de enkele band). De verwerking begint altijd met de ruwe gegevens. | Zoals vastgelegd |
| `debayered` | Lineaire demosaïc — 3-kanaals voor M3C, 1-kanaals grijswaarden voor M3M: de donkere offset van het kalibratiepakket wordt verwijderd, gevolgd door een randbewuste demosaïc, zonder lineariteitscorrectie, flat field of unmix. Een `process`-export schrijft hetzelfde bestand op dat bij een opname op dit niveau wordt opgeslagen, in elke verwerkingsmodus. | Lineaire DN |
| `radiance` | Absolute spectrale straling uit de volledige radiometrische keten, in **W/m²/sr/nm**. Wordt altijd opgeslagen als 32-bits TIFF (`tiff32/Radiance_Images/`), ongeacht het geselecteerde exportformaat. | float32 |
| `reflectance` | Reflectie ρ, waarbij **DN 32768 = ρ 1,0 (100%)** met ruimte tot ρ 2,0. Geschikt voor Pix4D. | uint16 |
| `preview` | Weergaveklare render: RGB-waarden = witbalans + gamma; multispectraal = valsekleurenuitrekking. | 8-bits weergave |

### Oriëntatie

`raw`- en `debayered`-bestanden behouden altijd de **oorspronkelijke** oriëntatie van de sensor; de instelling voor spiegelen/roteren van de camera wordt alleen toegepast op de andere producten (preview, straling, reflectie, index). Bij array-opnames blijven de bestanden van elk element in de oorspronkelijke oriëntatie, tenzij u om oriëntatie vraagt (`--orient` in de CLI, `orient=True` in de SDK). Elk bestand registreert de oriëntatie die bij de pixels hoort in XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — en `ChlorosOrientationApplied` geeft aan of de pixels zelf deze informatie bevatten (`false` bij raw- en debayered-bestanden). De Chloros-beeldviewer past raw- en debayered-bestanden aan op basis van dat record, uitsluitend voor weergave; de bestanden zelf worden nooit herschreven. Een bestand zonder record (geschreven vóór versie 1.3.0 of door een ander programma) wordt weergegeven zoals het is opgeslagen.

## Clamp-maskers

De stralings- en reflectantieproducten van een multispectrale M3C-camera ondergaan een 3×3-band-unmix. Wanneer de unmix in een band een negatieve waarde oplevert, wordt die waarde afgekapt tot nul (liveweergaven kappen op precies dezelfde manier af). Nieuw in 1.3.0: een product waarin ten minste één pixel is afgekapt, krijgt een bijbehorend **clamp-masker** dat precies aangeeft welke pixels en banden dit betreft:

* **Waar:**in een map met de naam `clamp_masks/`**naast** de productmappen, nooit erin: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, en `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Wat:** een `uint8`-TIFF-bestand met één band op het eigen pixelraster van het product (inclusief oriëntatie, ROI, binning en array-uitlijning). Bit 0 (`mask & 1`) is ingesteld waar de band in het R-kanaal van het product is afgekapt, bit 1 (`& 2`) het G-kanaal, bit 2 (`& 4`) het B-kanaal. Het aandeel van geknipte pixels per band wordt ook in het product vastgelegd als `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` en `Chloros:BandBUnmixClampFraction`.
* **Hoe dit in het product wordt aangegeven:** de XMP-tag `Chloros:UnmixClampMask` bevat het pad naar het masker ten opzichte van de map van het product (`../clamp_masks/<product folder>/<name>_clampmask.tif`, altijd gescheiden door `/`), `none` wanneer er geen pixel is afgekapt (er wordt geen masker geschreven), of `unavailable` wanneer er geen masker kon worden geschreven. Een product zonder tag heeft de unmix nooit uitgevoerd (RGB- en monochrome camera’s, raw- en debayered-niveaus).

Omdat geen enkele productmap een masker bevat, kun je een productmap zoals `Reflectance_Calibrated_Images/` in zijn geheel, inclusief submappen, doorgeven aan Pix4D, Metashape of andere fotogrammetriesoftware — geef de productmap door, niet de bovenliggende formaatmap. De eigen importfunctie van Chloros slaat clamp-maskers over. De CLI-opdracht telt ze apart: `Image products written: N (plus M clamp masks)`.

## Indexproducten

Elke index komt terecht in een eigen `<INDEX>_Index_Images/`-map (bijv. `NDVI_Index_Images/`).

* **Integerformaten**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) slaan een**lineaire uitrekking per beeld** op, geen −1…1-toewijzing: DN 0 is de laagste indexwaarde van dat frame en de volledige schaal de hoogste. Het bestand bevat `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` en `Chloros:IndexMax`, dus ook `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Omdat elk frame zijn eigen eindpunten heeft, moet je frames vergelijken op basis van waarden, niet op basis van DN.
* **`TIFF (32-bit, Percent)`** slaat de indexwaarden zelf op als float32, zonder herschaling — de beste keuze voor kwantitatief werk gedurende een vlucht.

Nieuw in 1.3.0: een indexproduct beschrijft **zichzelf** in plaats van de tags van de ruwe brongegevens te herhalen. De GPS- en cameratags worden gekopieerd van het product waarop het is berekend (het reflectantieproduct, indien aanwezig), en vervolgens:

| Tag | Waarde |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (zodat het bestand nooit opnieuw als opname wordt geïmporteerd) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / de gebruikte formule |
| `Chloros:IndexLevel` | `reflectance` of `radiance` — waarop de index is gebaseerd (ontbreekt als geen van beide van toepassing is) |
| `Chloros:IlluminationInvariant` | `True` voor een index berekend op basis van reflectie, `False` op basis van straling of sensorwaarden |
| `Chloros:ChlorosPipelineVersion` | de Chloros-versie waarmee de index is geschreven |

`Chloros:PixelScale` is verwijderd uit indexproducten — gebruik `IndexMin` / `IndexMax` hierboven. Het volledige terugleesrecept staat onder „Een raster met gehele getallen is een contrastuitrekking per afbeelding“ in de [CLI-referentie](reference/cli-reference.md).

## Reflectiewaarden van pixels uitlezen

Reflectantie wordt opgeslagen als een digitaal geheel getal, en **het DN-getal dat ρ = 1,0 (100% reflectantie) aangeeft, is afhankelijk van de broncamera**:

| Broncamera | ρ = 1,0 is DN | Hoe te bepalen |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (speling tot ρ 2,0) | XMP-tag `Chloros:PixelScale=32768` is in het bestand opgenomen. |
| Survey3 | `65535` (afgekapt bij ρ 1,0) | Geen `Chloros:PixelScale`-tag — die afwezigheid is het teken. (Producten van Survey3 bevatten wel andere `Chloros:*`-verwerkingstags, zoals `Chloros:CalibrationApplied`, in elke verwerkingsmodus vanaf versie 1.3.0.) |

**Lees de `Chloros:PixelScale` XMP-tag en deel erdoor** in plaats van uit te gaan van een constante. De tag is gedefinieerd in het uint16-domein, dus hij blijft `32768` in alle uitvoerformaten die herschalen — normaliseer het opgeslagen dtype eerst terug naar uint16 (×257 vanaf 8-bit, ×65535 vanaf float32).

{% hint style="warning" %}
**8-bits opnames.** De exportgegevens van een 8-bits opname zijn 8-bits. De reflectantie ervan wordt opnieuw geschaald en voorzien van de tag `Chloros:PixelScale=32768`, zodat de schaal behouden blijft; 8-bits DN-producten en reflectantiebestanden die vóór versie 1.3.0 zijn geschreven, worden begrensd tot 0–255 en bevatten geen schaaltag. Als de tag ontbreekt in een LATTICE-reflectantiebestand, ga dan niet uit van een schaal; exporteer het bestand in plaats daarvan opnieuw in 16-bits of 32-bits.
{% endhint %}

Zie voor de volledige regels (inclusief de MicaSense-compatibele tags) **&quot;Reflectantiepixels lezen&quot;** in de [CLI-referentie](reference/cli-reference.md).
