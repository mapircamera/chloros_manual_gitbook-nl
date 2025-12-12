---
description: In het laboratorium gemeten panelen die worden gebruikt om vastgelegde gegevens tijdens de nabewerking te kalibreren
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibratiedoelen

MAPIR biedt verschillende kalibratiedoelen voor een reeks toepassingen. De compacte T4-R50, zoals hieronder te zien, bevat 4 panelen waarvan de lichtreflectie van 250 - 2.500 nm is gemeten.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

De T4 diffuse referentiedoelen hebben de volgende reflectiecurven, [gegevens hier downloaden](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectie:: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectie:: 400-1000 nm</p></figcaption></figure>

Als u naar de reflectiegrafiek kijkt, kunt u zien dat de waarden de golflengte (x-as) versus het reflectiepercentage (y-as) zijn. Wanneer we een beeld van het kalibratiedoel vastleggen, creëren we vervolgens een relatie tussen de pixelwaarde en het reflectiepercentage, binnen het spectrum waarvoor elk van de sensorbanden van de camera gevoelig is.

Dit betekent dat u bij elke afbeelding die u met onze camera's vastlegt, een foto kunt gebruiken van onze reflectiedoelen, zoals de [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) of [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), om de afbeeldingen te kalibreren op reflectie. Eenmaal gekalibreerd is elke pixel in de afbeelding gelijk aan het percentage reflectie.

Als u de gekalibreerde afbeeldingen in Chloros als de typische JPG of TIFF uitvoert, wordt het reflectiepercentage berekend door de pixelwaarde te delen door de bitdiepte van het afbeeldingsformaat. Dus voor JPG delen door 255, en voor TIFF delen door 65.535. U kunt ook de uitvoer in PERCENT-indeling in Chloros kiezen, waarna elke pixel een procentuele waarde van 0,0 tot 1,0 heeft (0% tot 100% reflectie). Houd er rekening mee dat sommige afbeeldingstoepassingen de procentuele (zwevende-komma) afbeeldingen niet kunnen accepteren, en dat ze qua opslag groot zijn.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
