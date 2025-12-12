---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispectrale indexformules

De onderstaande indexformules maken gebruik van een combinatie van Survey3 filter gemiddelde transmissiebereiken:

<table><thead><tr><th align="center">Survey3 Filterkleur</th><th width="196.199951171875" align="center">Survey3-filternaam</th><th width="159.800048828125" align="center">Transmissiebereik (FWHM)</th><th align="center">Gemiddelde transmissie</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Wanneer deze formules worden gebruikt, kan de naam eindigen op &quot;\_1&quot; of &quot;\_2&quot;, wat overeenkomt met het NIR-filter dat werd gebruikt, namelijk NIR1 of NIR2.

***

## EVI - Verbeterde vegetatie-index

Deze index is oorspronkelijk ontwikkeld voor gebruik met MODIS-gegevens als verbetering ten opzichte van NDVI door het vegetatiesignaal in gebieden met een hoge bladoppervlakte-index (LAI) te optimaliseren. Hij is het meest bruikbaar in gebieden met een hoge LAI waar NDVI verzadigd kan raken. Hij maakt gebruik van het blauwe reflectiegebied om de signalen van de bodemachtergrond te corrigeren en atmosferische invloeden, waaronder aerosolverstrooiing, te verminderen.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-waarden moeten voor vegetatiepixels tussen 0 en 1 liggen. Heldere elementen zoals wolken en witte gebouwen, samen met donkere elementen zoals water, kunnen leiden tot abnormale pixelwaarden in een EVI-afbeelding. Voordat u een EVI-afbeelding maakt, moet u wolken en heldere kenmerken uit de reflectieafbeelding maskeren en optioneel de pixelwaarden van 0 tot 1 drempelen.

_Referentie: Huete, A., et al. &quot;Overzicht van de radiometrische en biofysische prestaties van de MODIS-vegetatie-indexen.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Bosbedekkingsindex 1

Deze index onderscheidt boskruinen van andere soorten vegetatie met behulp van multispectrale reflectiebeelden die een rode randband bevatten.

$$
FCI1 = Red * RedEdge
$$

Beboste gebieden hebben lagere FCI1-waarden vanwege de lagere reflectie van bomen en de aanwezigheid van schaduwen binnen het bladerdak.

_Referentie: Becker, Sarah J., Craig S.T. Daughtry en Andrew L. Russ. &quot;Robuuste bosbedekkingsindexen voor multispectrale beelden.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Bosbedekkingsindex 2

Deze index onderscheidt boskruinen van andere soorten vegetatie met behulp van multispectrale reflectiebeelden zonder rode randband.

$$
FCI2 = Red * NIR
$$

Beboste gebieden hebben lagere FCI2-waarden vanwege de lagere reflectie van bomen en de aanwezigheid van schaduwen binnen het bladerdak.

_Referentie: Becker, Sarah J., Craig S.T. Daughtry en Andrew L. Russ. &quot;Robuuste bosbedekkingsindexen voor multispectrale beelden.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Index voor wereldwijde milieumonitoring

Deze niet-lineaire vegetatie-index wordt gebruikt voor wereldwijde milieumonitoring op basis van satellietbeelden en probeert atmosferische effecten te corrigeren. Hij is vergelijkbaar met NDVI, maar is minder gevoelig voor atmosferische effecten. Hij wordt beïnvloed door kale grond en wordt daarom niet aanbevolen voor gebruik in gebieden met schaarse of matig dichte vegetatie.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Waar:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referentie: Pinty, B., en M. Verstraete. GEMI: een niet-lineaire index voor het monitoren van wereldwijde vegetatie vanuit satellieten. Vegetation 101 (1992): 15-20._

***

## GARI - Green Atmosferisch resistente index

Deze index is gevoeliger voor een breed scala aan chlorofylconcentraties en minder gevoelig voor atmosferische effecten dan NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

De gamma-constante is een wegingsfunctie die afhankelijk is van de aerosolcondities in de atmosfeer. ENVI gebruikt een waarde van 1,7, wat de aanbevolen waarde is van Gitelson, Kaufman en Merzylak (1996, pagina 296).

_Referentie: Gitelson, A., Y. Kaufman en M. Merzylak. &quot;Gebruik van een Green-kanaal bij teledetectie van wereldwijde vegetatie vanuit EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Chlorofylindex

Deze index wordt gebruikt om het chlorofylgehalte in bladeren van een breed scala aan plantensoorten te schatten.

$$
GCI = {NIR \over Green} - 1
$$

Door een breed NIR en groene golflengten te hebben, kan het chlorofylgehalte beter worden voorspeld, terwijl er meer gevoeligheid en een hogere signaal-ruisverhouding mogelijk is.

_Referentie: Gitelson, A., Y. Gritz en M. Merzlyak. &quot;Relaties tussen het chlorofylgehalte in bladeren en spectrale reflectie en algoritmen voor niet-destructieve chlorofylbeoordeling in bladeren van hogere planten.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Bladindex

Deze index is oorspronkelijk ontworpen voor gebruik met een digitale RGB-camera om de graanbedekking te meten, waarbij de rode, groene en blauwe digitale getallen (DN&#x27;s) variëren van 0 tot 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-waarden variëren van -1 tot +1. Negatieve waarden staan voor bodem en niet-levende elementen, terwijl positieve waarden staan voor groene bladeren en stengels.

_Referentie: Louhaichi, M., M. Borman en D. Johnson. &quot;Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.&quot; Geocarto International 16, nr. 1 (2001): 65-70._

***

## GNDVI - Green Genormaliseerde vegetatie-index

Deze index is vergelijkbaar met NDVI, behalve dat hij het groene spectrum van 540 tot 570 nm meet in plaats van het rode spectrum. Deze index is gevoeliger voor chlorofylconcentratie dan NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referentie: Gitelson, A., en M. Merzlyak. &quot;Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Geoptimaliseerde bodemgecorrigeerde vegetatie-index

Deze index is oorspronkelijk ontworpen met kleur-infraroodfotografie om de stikstofbehoefte voor maïs te voorspellen. Hij is vergelijkbaar met OSAVI, maar vervangt de groene band door rood.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referentie: Sripada, R., et al. &quot;Bepaling van de stikstofbehoefte voor maïs tijdens het seizoen met behulp van luchtfoto&#x27;s in kleur-infrarood.&quot; Proefschrift, North Carolina State University, 2005._

***

## GRVI - Green Ratio Vegetatie-index

Deze index is gevoelig voor fotosynthesesnelheden in boskruinen, aangezien groene en rode reflecties sterk worden beïnvloed door veranderingen in bladpigmenten.

$$
GRVI = {NIR \over Green }
$$

_Referentie: Sripada, R., et al. &quot;Luchtfoto&#x27;s met infraroodkleuren voor het bepalen van de stikstofbehoefte van maïs in het vroege seizoen.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Bodemgecorrigeerde vegetatie-index

Deze index is oorspronkelijk ontworpen met kleur-infraroodfotografie om de stikstofbehoefte voor maïs te voorspellen. Hij is vergelijkbaar met SAVI, maar vervangt de groene band door rood.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referentie: Sripada, R., et al. &quot;Bepaling van de stikstofbehoefte voor maïs tijdens het seizoen met behulp van luchtfotografie in kleur-infrarood.&quot; Proefschrift, North Carolina State University, 2005._

***

## LAI - Bladoppervlakte-index

Deze index wordt gebruikt om de bladbedekking te schatten en de groei en opbrengst van gewassen te voorspellen. ENVI berekent groene LAI met behulp van de volgende empirische formule van Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Waarbij EVI is:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hoge LAI-waarden variëren doorgaans van ongeveer 0 tot 3,5. Wanneer de scène echter wolken en andere heldere elementen bevat die verzadigde pixels produceren, kunnen de LAI-waarden hoger zijn dan 3,5. Idealiter maskeert u wolken en heldere elementen uit uw scène voordat u een LAI-afbeelding maakt.

_Referentie: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde en A. Thomsen. &quot;Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.&quot; Remote Sensing of Environment 81, nr. 2-3 (2002): 179-193._

***

## LCI - Bladchlorofylindex

Deze index wordt gebruikt om het chlorofylgehalte in hogere planten te schatten, die gevoelig zijn voor variaties in reflectie als gevolg van chlorofylabsorptie.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referentie: Datt, B. &quot;Remote Sensing of Water Content in Eucalyptus Leaves.&quot; Journal of Plant Physiology 154, nr. 1 (1999): 30-36._

***

## MNLI - Gemodificeerde niet-lineaire index

Deze index is een verbetering van de niet-lineaire index (NLI) waarin de bodemgecorrigeerde vegetatie-index (SAVI) is opgenomen om rekening te houden met de bodemachtergrond. ENVI gebruikt een waarde van 0,5 voor de correctiefactor voor de achtergrond van het bladerdak (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referentie: Yang, Z., P. Willis en R. Mueller. &quot;Impact van Band-Ratio Enhanced AWIFS Image op de nauwkeurigheid van gewasclassificatie.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 - Gemodificeerde bodemgecorrigeerde vegetatie-index 2

Deze index is een eenvoudigere versie van de MSAVI-index, voorgesteld door Qi, et al (1994), die een verbetering is ten opzichte van de Soil Adjusted Vegetation Index (SAVI). Het vermindert bodemruis en vergroot het dynamische bereik van het vegetatiesignaal. MSAVI2 is gebaseerd op een inductieve methode die geen constante _L_-waarde gebruikt (zoals bij SAVI) om gezonde vegetatie te benadrukken.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referentie: Qi, J., A. Chehbouni, A. Huete, Y. Kerr en S. Sorooshian. &quot;A Modified Soil Adjusted Vegetation Index.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE- Genormaliseerd verschil RedEdge

Deze index is vergelijkbaar met NDVI, maar vergelijkt het contrast tussen NIR en RedEdge in plaats van Red, waardoor vegetatiestress vaak eerder wordt gedetecteerd.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Genormaliseerde vegetatie-index

Deze index is een maatstaf voor gezonde, groene vegetatie. De combinatie van de genormaliseerde verschilformule en het gebruik van de gebieden met de hoogste absorptie en reflectie van chlorofyl maken deze index robuust onder uiteenlopende omstandigheden. Hij kan echter verzadigd raken in dichte vegetatie wanneer LAI hoog wordt.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

De waarde van deze index varieert van -1 tot 1. Het gebruikelijke bereik voor groene vegetatie is 0,2 tot 0,8.

_Referentie: Rouse, J., R. Haas, J. Schell en D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI - Niet-lineaire index

Deze index gaat ervan uit dat de relatie tussen veel vegetatie-indexen en biofysische parameters aan het oppervlak niet-lineair is. Hij lineariseert relaties met oppervlakteparameters die de neiging hebben niet-lineair te zijn.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referentie: Goel, N., en W. Qin. &quot;Invloeden van de architectuur van het bladerdak op relaties tussen verschillende vegetatie-indexen en LAI en Fpar: een computersimulatie.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Geoptimaliseerde bodemgecorrigeerde vegetatie-index

Deze index is gebaseerd op de bodemgecorrigeerde vegetatie-index (SAVI). Hij gebruikt een standaardwaarde van 0,16 voor de correctiefactor voor de achtergrond van het bladerdak. Rondeaux (1996) stelde vast dat deze waarde een grotere bodemvariatie oplevert dan SAVI voor lage vegetatiebedekking, terwijl hij een grotere gevoeligheid vertoont voor vegetatiebedekking van meer dan 50%. Deze index kan het best worden gebruikt in gebieden met relatief dunne vegetatie waar de bodem door het bladerdak heen zichtbaar is.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referentie: Rondeaux, G., M. Steven en F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Renormalized Difference Vegetation Index

Deze index maakt gebruik van het verschil tussen nabij-infrarode en rode golflengten, samen met de NDVI, om gezonde vegetatie te benadrukken. Hij is ongevoelig voor de effecten van bodem en de geometrie van de zon.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referentie: Roujean, J., en F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Bodemgecorrigeerde vegetatie-index

Deze index is vergelijkbaar met NDVI, maar onderdrukt de effecten van bodempixels. Hij maakt gebruik van een aanpassingsfactor voor de achtergrond van het bladerdak, _L_, die een functie is van de vegetatiedichtheid en vaak voorkennis van de hoeveelheid vegetatie vereist. Huete (1988) stelt een optimale waarde van _L_=0,5 voor om rekening te houden met eerste-orde variaties in de bodemachtergrond. Deze index kan het best worden gebruikt in gebieden met relatief dunne vegetatie waar de bodem door het bladerdak heen zichtbaar is.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referentie: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Transformed Difference Vegetation Index

Deze index is nuttig voor het monitoren van de vegetatiebedekking in stedelijke omgevingen. Hij raakt niet verzadigd zoals NDVI en SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referentie: Bannari, A., H. Asalhi en P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping&quot; In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - Visible Atmospherically Resistant Index

Deze index is gebaseerd op de ARVI en wordt gebruikt om het aandeel van de vegetatie in een scène te schatten met een lage gevoeligheid voor atmosferische effecten.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referentie: Gitelson, A., et al. &quot;Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Vegetatie-index met breed dynamisch bereik

Deze index is vergelijkbaar met NDVI, maar maakt gebruik van een wegingscoëfficiënt (_a_) om het verschil tussen de bijdragen van de nabij-infrarood- en rode signalen aan de NDVI te verminderen. De WDRVI is bijzonder effectief in scènes met een matige tot hoge vegetatiedichtheid wanneer NDVI hoger is dan 0,6. NDVI heeft de neiging af te vlakken wanneer het vegetatieaandeel en de bladoppervlakte-index (LAI) toenemen, terwijl de WDRVI gevoeliger is voor een breder scala aan vegetatieaandelen en voor veranderingen in LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

De wegingscoëfficiënt (_a_) kan variëren van 0,1 tot 0,2. Een waarde van 0,2 wordt aanbevolen door Henebry, Viña en Gitelson (2004).

_Referenties_

_Gitelson, A. &quot;Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.&quot; Journal of Plant Physiology 161, nr. 2 (2004): 165-173._

_Henebry, G., A. Viña en A. Gitelson. &quot;The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.&quot; Gap Analysis Bulletin 12: 50-56._
