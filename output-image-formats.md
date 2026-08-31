---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Uitvoerbeeldformaten

Chloros exporteert bewerkte producten in vier bestandsformaten. Selecteer het formaat in de projectinstellingen (GUI), met `--format` (CLI) of met `export_format` (SDK). De CLI en SDK accepteren de exacte tekenreeksen hieronder.

| Formaatstring | Uitbreiding | Pixeltype | Pixelbereik | Opmerkingen |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(standaard)* | `.tif` | uint16 digitaal getal | 0 – 65535 | Aanbevolen voor fotogrammetrie / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% reflectie. Sommige toepassingen kunnen geen TIFF-bestanden met drijvende-kommagetallen lezen; de bestanden zijn groter. |
| `PNG (8-bit)` | `.png` | uint8 digitaal getal | 0 – 255 | Verliesloze compressie, geschikt voor weergave op het web en visualisatie. |
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

De cameramap is `LATT-<sensor>-<lens>-F<filter>` voor LATTICE en `<model>_<filter>` (bijv. `Survey3N_RGN`) voor Survey3. **Elk geëxporteerd product behoudt de naam van het bronbestand — de map identificeert het product, niet een bestandsextensie.** Zie [Waar de uitvoer terechtkomt](reference/cli-reference.md) in de CLI-referentie voor de volledige regels.

## LATTICE-producten (opname- en exportniveaus)

Eén LATTICE-ruwframe wordt in één keer uitgesplitst naar elk aangevraagd product. Elk producttype heeft zijn eigen schakelaar (GUI-selectievakjes, of CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, standaard allemaal AAN):

| Niveau | Inhoud | Gegevenstype |
| --- | --- | --- |
| `raw` | Bayer-gegevens rechtstreeks van de sensor (monochrome camera’s: de enkele band). De verwerking begint altijd met de ruwe gegevens. | Zoals vastgelegd |
| `debayered` | Lineaire demosaïek — 3-kanaals voor M3C, 1-kanaals grijswaarden voor M3M. | Lineaire DN |
| `radiance` | Absolute spectrale straling uit de volledige radiometrische keten, in **W/m²/sr/nm**. Altijd opgeslagen als 32-bits TIFF (`tiff32/Radiance_Images/`), ongeacht het geselecteerde exportformaat. | float32 |
| `reflectance` | Reflectie ρ, waarbij **DN 32768 = ρ 1,0 (100%)** met ruimte voor ρ 2,0. Geschikt voor Pix4D. | uint16 |
| `preview` | Weergaveklare weergave: RGB = witbalans + gamma; multispectraal = valsekleuruitrekking. | 8-bits weergave |

## Reflectantie-pixelwaarden uitlezen

Reflectantie wordt opgeslagen als een digitaal geheel getal, en **de DN die ρ = 1,0 (100% reflectantie) aangeeft, is afhankelijk van de broncamera**:

| Broncamera | ρ = 1,0 is DN | Hoe te bepalen |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (speling tot ρ 2,0) | De XMP-tag `Chloros:PixelScale=32768` is in het bestand opgenomen. |
| Survey3 | `65535` (afgekapt bij ρ 1,0) | Geen `Chloros:*` XMP-tags — die afwezigheid is het signaal. |

**Lees de `Chloros:PixelScale` XMP-tag en deel erdoor** in plaats van uit te gaan van een constante. De tag is gedefinieerd in het uint16-domein, dus deze blijft `32768` in alle uitvoerformaten die de schaal aanpassen — normaliseer het opgeslagen dtype eerst terug naar uint16 (×257 vanaf 8-bit, ×65535 vanaf float32).

{% hint style="warning" %}
**Eén geval kent per opzet geen schaalfactor.** Wanneer een 8-bits bronopname (BayerRG8) wordt geschreven als 8-bits TIFF, wordt de waarde in de pijplijn begrensd tot 0–255 in plaats van opnieuw geschaald, dus het bestand heeft geen schaal — Chloros laat `Chloros:PixelScale` daar bewust weg. Als de tag ontbreekt in een LATTICE-reflectantiebestand, ga dan niet uit van een schaal; exporteer het bestand in plaats daarvan opnieuw in 16-bit of 32-bit.
{% endhint %}

Zie voor de volledige regels (inclusief de MicaSense-compatibele tags) **&quot;Reflectantiepixels lezen&quot;** in de [CLI-referentie](reference/cli-reference.md).
