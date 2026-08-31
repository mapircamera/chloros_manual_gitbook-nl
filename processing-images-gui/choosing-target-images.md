# Doelafbeeldingen selecteren

Door aan te geven welke afbeeldingen kalibratiedoelen bevatten, weet Chloros precies waar het naar deze doelen moet zoeken. Wanneer er ten minste één afbeelding is aangevinkt in de kolom ‘Doel’, scant Chloros **alleen de aangevinkte afbeeldingen** — het markeren van doelen versnelt dus niet alleen de verwerking, maar voorkomt ook dat landmeetkundige beelden ten onrechte als doel worden aangemerkt.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Waarom doelafbeeldingen markeren?

### Markeren bepaalt de scan

Wanneer u de kolom ‘Doel’ voor specifieke afbeeldingen aanvinkt:

* Chloros scant alleen de aangevinkte afbeeldingen op doelen
* Het detecteren van doelen verloopt veel sneller
* Landmeetkundige afbeeldingen kunnen geen valse doel detecties opleveren

Als er **geen** beelden zijn aangevinkt, valt Chloros terug op het scannen van elk beeld in het project:

* Doeldetectiealgoritmen worden op elk beeld uitgevoerd
* Honderden of duizenden beelden worden onnodig gecontroleerd
* De verwerking duurt aanzienlijk langer, vooral bij grote datasets

{% hint style="success" %}
**Snelheidsverbetering**: Door 2-3 doelbeelden te markeren in een dataset van 500 beelden kan de tijd voor doeldetectie worden teruggebracht van meer dan 30 minuten naar minder dan 1 minuut.
{% endhint %}

***

## Hoe doelafbeeldingen markeren

### Stap 1: Identificeer uw doelafbeeldingen

Bekijk uw geïmporteerde afbeeldingen in de bestandsbrowser en bepaal welke afbeeldingen kalibratiedoelen bevatten.

**Veelvoorkomende scenario’s:*** **Doel vóór opname**: Opgenomen vóór het starten van de sessie
* **Doel na opname**: vastgelegd na afloop van de sessie
* **Doelen in het veld**: doelen die binnen het opnamegebied zijn geplaatst
* **Meerdere doelen**: 2-3 doelafbeeldingen per sessie (aanbevolen)

### Stap 2: Controleer de kolom **Target** <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Voor elke afbeelding die een kalibratiedoel bevat:

1. Zoek de afbeelding in de tabel van de bestandsbrowser
2. Zoek de kolom **Target** (de kolom uiterst rechts)
3. Vink het selectievakje in de kolom **Target** voor die afbeelding aan
4. Herhaal dit voor alle beelden die doelen bevatten

### Stap 3: Controleer je selectie

Controleer vóór de verwerking nogmaals:

* [ ] Alle beelden met kalibratiedoelen zijn aangevinkt
* [ ] Er zijn geen beelden zonder doel per ongeluk aangevinkt
* [ ] De doelen zijn duidelijk zichtbaar in de aangevinkte beelden

***

## LATTICE: Doelen zijn optioneel wanneer een DAQ opneemt

Voor LATTICE-multispectrale camera’s is een kalibratiedoel binnen het beeld **een van de twee** mogelijke reflectantiereferenties:

* **Doel binnen het beeld**: wanneer een gemarkeerde doelafbeelding de kwaliteitscontroles (QA) vChloros doorstaat, wordt het doel de**absolute reflectantiereferentie** voor de beelden eromheen.
* **DAQ-neerwaartse straling**: wanneer er geen doel aanwezig is (of de kwaliteitscontrole mislukt), berekent Chloros de reflectie in plaats daarvan op basis van de neerwaartse stralingsintensiteit van de lichtsensor van de DAQ (ρ = π·L/E). Als een `.daq`- of DAQ-M `.csv`-opname uw opnames dekt, krijgt u gekalibreerde reflectie**zonder ook maar enig doelbeeld**.

Dit automatische gedrag is de standaardinstelling. In de CLI / SDK komt dit overeen met `--reflectance-source auto`; u kunt ook `target` (strikt — geen DAQ-vervanging) of `daq` (DAQ-autoritair) forceren. Zie de [CLI-referentie](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**LATTICE-doelgeometrieën**: naast de klassieke paneeldetectie die wordt gebruikt voor Survey3, ondersteunt de LATTICE-verwerking**met ArUco gemarkeerde doelen**,**doelen met vaste ROI**en**strookdoelen**, die per project worden geconfigureerd. Per eenheid kunnen**gemeten** reflectiescans van doelen worden aangeleverd op serienummer (CLI: `--target-reflectance-dir`, één `<serial>.csv` per doeleenheid), met de nominale T3/T4P-spectra als back-up.

{% hint style="info" %}
**F988-module**: De reflectantie van de F988 wordt gekalibreerd met behulp van een reflectantiepaneel ter plaatse: de band ligt buiten het gekalibreerde bereik van de DAQ-lichtsensor, dus Chloros past uw meest recente paneelopname toe en behoudt deze tussen de paneelmetingen door. Als een F988-module uitsluitend met DAQ wordt verwerkt, wijst Chloros de op DAQ gebaseerde reflectantie voor die band af (oversla-reden `dls-uncalibrated-band-988`) — de paneelworkflow is de ondersteunde methode.
{% endhint %}

***

## Aanbevolen werkwijzen voor doelafbeeldingen

### Richtlijnen voor het vastleggen van doelafbeeldingen

**Timing:**

* Leg doelafbeeldingen vast vlak voor en gedurende uw opnamesessie
* Onder dezelfde lichtomstandigheden als uw DAQ-lichtsensor
* Maak idealiter zo vaak mogelijk doelafbeeldingen voor het beste resultaat. Anders worden de gegevens van de lichtsensor gebruikt om de kalibratie in de loop van de tijd aan te passen.

**Camera-positie:**

* Houd de camera boven het doel, zodat het gecentreerd is en ongeveer 40-60% van het midden van het beeld vult.
* Houd de camera parallel aan of loodrecht boven het doeloppervlak

**Verlichting:**

* Dezelfde omgevingsverlichting als die van uw DAQ-lichtsensor
* Vermijd schaduwen op de doeloppervlakken
* Blokkeer uw lichtbron niet met uw lichaam, voertuig of begroeiing
* Bewolkte omstandigheden leveren de meest consistente resultaten op

**Toestand van het doel:**

* Houd de doelpanelen schoon en droog
* Alle panelen van je doel (bijv. alle 4 op een T4) moeten duidelijk zichtbaar en onbelemmerd zijn
* Plaats doelen indien mogelijk loodrecht/nadir ten opzichte van de lichtbron

### Hoeveel doelbeelden?

**Minimum:**1 doelbeeld per sessie.**Aanbevolen:** 3-5 doelbeelden per sessie.**Aanbevolen schema:**

* 3-5 beelden vastgelegd kort nadat de lichtsensor begint met opnemen
* Draai de camera tussen de opnames door voor de beste resultaten
* Optioneel: tussentijds tijdens de sessie als de lichtomstandigheden voortdurend veranderen

***

## Werken met meerdere camera’s

### Opstellingen met twee camera’s

Als je twee MAPIR-camera’s tegelijkertijd gebruikt (bijv. Survey3W RGN + Survey3N OCN):

1. Maak doelafbeeldingen met **beide camera’s** tegelijkertijd
2. Gebruik voor beide camera’s **hetzelfde fysieke doel**

3. Markeer de doelbeelden voor**beide cameratypen** in de bestandsbrowser
4. Chloros gebruikt de juiste doelen voor de kalibratie van elke camera

### Kolom ‘Cameramodel’

De kolom **Cameramodel** helpt bij het identificeren welke beelden afkomstig zijn van welke camera:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* enz.

Gebruik deze kolom om te controleren of je voor elk cameratype in je project doelen hebt gemarkeerd.

***

## Instellingen voor doelherkenning

### De detectiegevoeligheid aanpassen

Als Chloros uw doelen niet correct detecteert, pas dan deze instellingen aan in [Projectinstellingen](adjusting-project-settings.md):**Minimaal kalibratiegebied (px):*** **Standaard**: 25 pixels
* **Verhoog** deze waarde als er valse detecties optreden bij kleine artefacten
* **Verlaag** deze waarde als doelen niet worden gedetecteerd**Minimale doelclustering (0-100):*** **Standaard**: 60
* **Verhoog** als doelen in meerdere detecties worden opgesplitst
* **Verlaag** als doelen met kleurvariatie niet volledig worden gedetecteerd

{% hint style="info" %}
**Tip voor CLI**: `chloros-cli process` ondersteunt dezelfde instellingen (`--min-target-size`, `--target-clustering`), en de vlag `--target`/`--targets` markeert een volledige invoermap als uitsluitend voor het doelpaneel. Zie de [CLI-referentie](../reference/cli-reference.md).
{% endhint %}

***

## Veelvoorkomende problemen met doelafbeeldingen

### Probleem: geen doelen gedetecteerd

**Mogelijke oorzaken:**

* Doelafbeeldingen zijn niet gemarkeerd in de bestandsbrowser
* Doel is te klein in beeld (&lt; 30% van de afbeelding)
* Slechte belichting (schaduwen, schittering)
* Instellingen voor doeldetectie zijn te streng

**Oplossingen:**

1. Controleer of de kolom ‘Doel’ is aangevinkt bij de juiste afbeeldingen
2. Controleer de kwaliteit van de doelafbeeldingen in het voorbeeld
3. Leg de doelen opnieuw vast als de kwaliteit slecht is
4. Pas de instellingen voor doelherkenning indien nodig aan

### Probleem: valse doelherkenningen

**Mogelijke oorzaken:**

* Witte gebouwen, voertuigen of bodembedekking die ten onrechte als doelen worden herkend
* Heldere vlekken in de vegetatie
* Detectiegevoeligheid te laag

**Oplossingen:**

1. Markeer alleen daadwerkelijke doelafbeeldingen — alleen aangevinkte afbeeldingen worden gescand
2. Vergroot het minimale kalibratiesteekproefgebied
3. Verhoog de minimale waarde voor doelclustering
4. Zorg ervoor dat doelafbeeldingen alleen het doel tonen (minimale achtergrondruis)

***

## Controlechecklist

Controleer vóór het starten van de verwerking uw selectie van doelafbeeldingen:

* [ ] Minimaal 1 doelafbeelding gemarkeerd per sessie (of, voor LATTICE, een `.daq`/`.csv`-opname die de sessie dekt)
* [ ] De selectievakjes in de doelkolom zijn aangevinkt voor alle doelafbeeldingen
* [ ] Doelafbeeldingen zijn vastgelegd binnen hetzelfde tijdsbestek als het onderzoek
* [ ] Doelen zijn duidelijk zichtbaar in het voorbeeldvenster wanneer erop wordt geklikt
* [ ] Alle kalibratiepanelen zijn zichtbaar in elke doelafbeelding
* [ ] Geen schaduwen of obstructies op de doelen
* [ ] Bij gebruik van twee camera’s: doelen gemarkeerd voor beide cameratypen

***

## Verwerking zonder doelen

### LATTICE: met een DAQ-opname

Als een DAQ-lichtsensor de neerwaartse stralingsintensiteit heeft geregistreerd tijdens uw LATTICE-opnames, is er geen doel nodig:

1. Importeer het bestand `.daq` (of DAQ-M `.csv`) met de beelden
2. Laat het vakje ‘Target’ (Doel) leeg
3. De reflectie wordt automatisch berekend op basis van de DAQ-referentie voor neerwaartse straling
4. Voor stralingsintensiteit is nooit een doel of een DAQ nodig — deze wordt uitsluitend afgeleid uit de fabrieksmatige radiometrische kalibratie van de camera

### Verwerking zonder enige referentie

Je kunt ook verwerken zonder doelen en zonder een DAQ:

1. Laat alle selectievakjes in de kolom ‘Doel’ uitgeschakeld
2. **Schakel** „Reflectanciekalibratie / witbalans“ uit in de projectinstellingen — de doelherkenning wordt dan volledig overgeslagen
3. Vignetteringscorrectie wordt nog steeds toegepast
4. De uitvoer wordt niet gekalibreerd voor absolute reflectantie (LATTICE multispectraal exporteert nog steeds debayered-, preview- en radiance-producten)

{% hint style="warning" %}
**Niet aanbevolen voor wetenschappelijk werk met de Survey3**: Zonder reflectiekalibratie geven de pixelwaarden van Survey3 alleen relatieve helderheid weer, geen wetenschappelijke reflectiemetingen. Gebruik kalibratiedoelen (of, voor LATTICE, een DAQ-lichtsensor) voor nauwkeurige, herhaalbare resultaten.
{% endhint %}

***

## Volgende stappen

Zodra u uw doelafbeeldingen hebt gemarkeerd:

1. **Controleer uw instellingen** - Zie [Projectinstellingen aanpassen](adjusting-project-settings.md)
2. **Start de verwerking** – Zie [De verwerking starten](starting-the-processing.md)
3. **Volg de voortgang** – Zie [De verwerking volgen](monitoring-the-processing.md)

Voor meer informatie over kalibratiedoelen zelf, zie [Kalibratiedoelen](../calibration-targets.md).
