---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibratiedoelen

MAPIR biedt diverse kalibratiedoelen voor een breed scala aan toepassingen. Het hieronder afgebeelde compacte T4-R50-model bevat 4 panelen waarvan de lichtreflectie is gemeten in het bereik van 250 tot 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>De T4 diffuse referentiedoelwitten hebben de volgende reflectiecurves, [gegevens hier downloaden](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4-reflectie :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-reflectie :: 400-1.000 nm</p></figcaption></figure>De T4P diffuse referentiedoelen hebben de volgende reflectanciekrommen, [gegevens hier downloaden](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflectantie :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflectie :: 400-1000 nm</p></figcaption></figure>Als je naar de reflectiegrafiek kijkt, zie je dat de waarden de golflengte (x-as) weergeven tegenover het reflectiepercentage (y-as). Wanneer we een foto maken van het kalibratiedoel, leggen we vervolgens een verband tussen de pixelwaarde en het reflectiepercentage, binnen het spectrum waarvoor elk van de sensorbanden van de camera gevoelig is.

Dit betekent dat je bij elke foto die je met onze camera’s maakt, een foto van onze reflectiedoelwitten kunt gebruiken, zoals de [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) of [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), kunt gebruiken om de beelden te kalibreren op reflectie. Na kalibratie is elke pixel in het beeld gelijk aan het reflectiepercentage.

Voor **Survey3** geldt het volgende: als je de gekalibreerde afbeeldingen in Chloros uitvoert als een standaard JPG of TIFF, wordt het reflectiepercentage berekend door de pixelwaarde te delen door de bitdiepte van het afbeeldingsformaat. Dus voor JPG deel je door 255 en voor TIFF deel je door 65.535. Je kunt ook kiezen voor de PERCENT-uitvoer in Chloros, waarna elke pixel een procentuele waarde tussen 0,0 en 1,0 zal hebben (0% tot 100% reflectie). Houd er wel rekening mee dat sommige beeldbewerkingsprogramma’s geen afbeeldingen in procenten (drijvende komma) kunnen verwerken, en dat deze qua opslagruimte veel ruimte innemen.

{% hint style="info" %}
**De LATTICE-reflectie maakt gebruik van een andere pixelschaal.** De LATTICE-reflectie wordt opgeslagen met DN 32768 = 100% reflectie (niet 65535), en elk bestand bevat een XMP-tag (`Chloros:PixelScale`) waarin de schaal wordt vermeld. Lees de tag en deel door deze waarde in plaats van uit te gaan van een constante — zie [Uitvoerbeeldformaten](output-image-formats.md).
{% endhint %}

## Kalibratiedoelen met LATTICE-camera’s

Bij LATTICE-camera’s is een kalibratiedoel **optioneel** voor reflectie: Chloros kan in plaats daarvan de reflectie koppelen aan de neerwaartse stralingsintensiteit gemeten door een DAQ-lichtsensor (ρ = π·L/E). De referentie wordt gekozen met de instelling voor de reflectiebron (Projectinstellingen in de GUI; `--reflectance-source` in de CLI; `reflectance_source` in de SDK):

| Waarde | Gedrag |
| --- | --- |
| `auto` *(standaard)* | Een QA-goedgekeurd doel binnen het beeld is de **absolute referentie**; wanneer er geen doel aanwezig is of de QA mislukt, valt Chloros terug op de DAQ-downwelling-drempel. |
| `target` | Strikt alleen doel — geen DAQ-vervanging. |
| `daq` | DAQ-bepalend — de neerwaartse meting is altijd de referentie. |

Aanvullend doelgedrag voor LATTICE:

* **Doelgeometrieën** — Panelen gemarkeerd met ArUco, panelen met vaste ROI en strookdoelen worden allemaal ondersteund; de geometrie is afkomstig uit de doelconfiguratie van het project.
* **Per eenheid gemeten doelgegevens** — `--target-reflectance-dir DIR` verwijst naar een map met per eenheid gemeten reflectiescans van doelen (`<serial>.csv`, opgezocht aan de hand van het serienummer/QR-code van de doeleenheid). Bij een misser valt Chloros terug op de nominale T3/T4P-spectra.
* **Tijdelijke verankering** — een gedetecteerd doel kalibreert de frames eromheen en wordt tussen doelwaarnemingen vastgehouden.

De volledige vlagsemantiek en voorbeelden staan in de [CLI-referentie](reference/cli-reference.md) (zie „Exportschakelaars per product”).

### F988

&quot;De reflectantie van F988 wordt gekalibreerd met behulp van een reflectantiepaneel in de scène: de band ligt buiten het gekalibreerde bereik van de DAQ-lichtsensor, dus Chloros past je meest recente paneelopname toe en behoudt deze tussen de paneelwaarnemingen.&quot;

Als F988 wordt uitgevoerd met uitsluitend DAQ-kalibratie, wijst Chloros de op DAQ gebaseerde reflectantie voor die band af en geeft aan waarom (oversla reden `dls-uncalibrated-band-988`); de paneelworkflow is de ondersteunde methode.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
