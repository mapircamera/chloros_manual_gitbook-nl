---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibratiedoelen

MAPIR biedt diverse kalibratiedoelen voor een breed scala aan toepassingen. Het hieronder afgebeelde compacte T4-R50-model bevat 4 panelen waarvan de lichtreflectie is gemeten in het bereik van 250 tot 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>De T4 diffuse referentiedoelen hebben de volgende reflectiecurves, [gegevens hier downloaden](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectie :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-reflectie :: 400-1000 nm</p></figcaption></figure>De T4P diffuse referentiedoelen hebben de volgende reflectiecurves, [gegevens hier downloaden](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflectie :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflectie :: 400-1000 nm</p></figcaption></figure>Als u naar de reflectiegrafiek kijkt, ziet u dat de waarden de golflengte (x-as) versus het reflectiepercentage (y-as) weergeven. Wanneer we een afbeelding van het kalibratiedoel vastleggen, creëren we vervolgens een relatie tussen de pixelwaarde en het reflectiepercentage, binnen het spectrum waarvoor elk van de sensorbanden van de camera gevoelig is.

Dit betekent dat u bij elke foto die u met onze camera&#x27;s maakt, een foto van onze reflectiedoelen kunt gebruiken, zoals de [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) of [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), om de beelden te kalibreren voor reflectie. Na kalibratie is elke pixel in de afbeelding gelijk aan het reflectiepercentage.

Als u de gekalibreerde beelden in Chloros uitvoert als de gebruikelijke JPG of TIFF, wordt het reflectiepercentage berekend door de pixelwaarde te delen door de bitdiepte van het beeldformaat. Dus voor JPG deelt u door 255, en voor TIFF deelt u door 65.535. U kunt ook kiezen voor de PERCENT-uitvoer in Chloros, waarbij elke pixel een procentuele waarde heeft tussen 0,0 en 1,0 (0% tot 100% reflectie). Houd er wel rekening mee dat sommige beeldbewerkingsprogramma&#x27;s geen afbeeldingen met procentuele waarden (drijvende komma) ondersteunen en dat deze bestanden veel opslagruimte in beslag nemen.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
