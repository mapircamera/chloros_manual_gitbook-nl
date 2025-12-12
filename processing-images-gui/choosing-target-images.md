# Doelafbeeldingen selecteren

Het markeren van afbeeldingen die kalibratiedoelen bevatten, is een cruciale stap die de verwerkingspijplijn van Chloros aanzienlijk versnelt. Door vooraf doelafbeeldingen te selecteren, hoeft Chloros niet elke afbeelding in uw dataset te scannen op kalibratiedoelen.

## Waarom doelafbeeldingen markeren?

### Verwerkingssnelheid

Zonder doelafbeeldingen te markeren, moet Chloros:

* Elke afbeelding in uw project scannen
* Doeldetectiealgoritmen op elke afbeelding uitvoeren
* Honderden of duizenden afbeeldingen onnodig controleren

**Resultaat**: De verwerking kan aanzienlijk langer duren, vooral bij grote datasets.

### Met gemarkeerde doelafbeeldingen

Wanneer u de kolom Doel voor specifieke afbeeldingen aanvinkt:

* scant Chloros alleen de aangevinkte afbeeldingen op doelen
* wordt de doelherkenning veel sneller voltooid
* wordt de totale verwerkingstijd aanzienlijk verkort

{% hint style=&quot;success&quot; %}
**Snelheidsverbetering**: Door 2-3 doelafbeeldingen te markeren in een dataset van 500 afbeeldingen kan de detectietijd van doelen worden teruggebracht van meer dan 30 minuten tot minder dan 1 minuut.
{% endhint %}

***

## Doelafbeeldingen markeren

### Stap 1: Identificeer uw doelafbeeldingen

Bekijk uw geïmporteerde afbeeldingen in de bestandsbrowser en identificeer welke afbeeldingen kalibratiedoelen bevatten.

**Veelvoorkomende scenario&#x27;s:**

* **Doel vóór de opname**: opgenomen vóór het starten van de sessie
* **Doel na de opname**: opgenomen na het voltooien van de sessie
* **Doelen in het veld**: doelen die binnen het opnamegebied zijn geplaatst
* **Meerdere doelen**: 2-3 doelafbeeldingen per sessie (aanbevolen)

### Stap 2: Controleer de doelkolom

Voor elke afbeelding die een kalibratiedoel bevat:

1. Zoek de afbeelding in de tabel van de bestandsbrowser.
2. Zoek de kolom **Doel** (kolom uiterst rechts).
3. Klik op het selectievakje in de kolom Doel voor die afbeelding.
4. Herhaal dit voor alle afbeeldingen die doelen bevatten.

### Stap 3: Controleer uw selectie

Controleer voor de verwerking nogmaals:

* [ ] Alle afbeeldingen met kalibratiedoelen zijn aangevinkt.
* [ ] Er zijn geen afbeeldingen zonder doel per ongeluk aangevinkt.
* [ ] De doelen zijn duidelijk zichtbaar in de aangevinkte afbeeldingen.

***

## Best practices voor doelafbeeldingen

### Richtlijnen voor het vastleggen van doelen

**Timing:**

* Leg doelafbeeldingen direct voor en tijdens uw opnamesessie vast.
* Onder dezelfde lichtomstandigheden als uw DAQ-lichtsensor.
* Maak voor het beste resultaat zo vaak mogelijk doelafbeeldingen. Anders worden de gegevens van de lichtsensor gebruikt om de kalibratie in de loop van de tijd aan te passen.

**Camera-positie:**

* Houd de camera boven het doel, zodat deze gecentreerd is en ongeveer 40-60% van het midden van de afbeelding vult.
* Houd de camera parallel/nadir ten opzichte van het doeloppervlak

**Verlichting:**

* Dezelfde omgevingsverlichting als uw DAQ-lichtsensor
* Vermijd schaduwen op de doeloppervlakken
* Blokkeer uw lichtbron niet met uw lichaam, voertuig of vegetatie
* Bewolkte omstandigheden leveren de meest consistente resultaten op

**Doelvoorwaarden:**

* Houd de doelpanelen schoon en droog
* Alle 4 panelen moeten duidelijk zichtbaar en onbelemmerd zijn
* Doelen zo mogelijk loodrecht/nadir op de lichtbron

### Hoeveel doelafbeeldingen?

**Minimaal:** 1 doelafbeelding per sessie. **Aanbevolen:** 3-5 doelafbeeldingen per sessie.

**Aanbevolen schema:**

* 3-5 afbeeldingen vastgelegd kort nadat de lichtsensor begint met opnemen
* Draai de camera tussen opnames om voor het beste resultaat
* Optioneel: periodiek halverwege de sessie als de lichtomstandigheden voortdurend veranderen

***

## Werken met meerdere camera&#x27;s

### Opstellingen met twee camera&#x27;s

Als u twee MAPIR-camera&#x27;s tegelijkertijd gebruikt (bijv. Survey3W RGN + Survey3N OCN):

1. Leg doelbeelden vast met **beide camera&#x27;s** tegelijkertijd.
2. Gebruik **hetzelfde fysieke doel** voor beide camera&#x27;s.
3. Markeer doelbeelden voor **beide cameratypen** in de bestandsbrowser.
4. Chloros gebruikt de juiste doelen voor de kalibratie van elke camera.

### Kolom Cameramodel

De kolom **Cameramodel** helpt bij het identificeren van welke beelden van welke camera afkomstig zijn:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* enz.

Gebruik deze kolom om te controleren of u doelen voor elk cameratype in uw project hebt gemarkeerd.

***

## Instellingen voor doelherkenning

### De detectiegevoeligheid aanpassen

Als Chloros uw doelen niet correct detecteert, pas dan deze instellingen aan in [Projectinstellingen](adjusting-project-settings.md):

**Minimaal kalibratiemonstergebied:**

* **Standaard**: 25 pixels
* **Verhoog** als u valse detecties krijgt bij kleine artefacten
* **Verlaag** als doelen niet worden gedetecteerd

**Minimale doelclustering:**

* **Standaard**: 60
* **Verhoog** als doelen worden opgesplitst in meerdere detecties
* **Verlaag** als doelen met kleurvariatie niet volledig worden gedetecteerd

***

## Veelvoorkomende problemen met doelafbeeldingen

### Probleem: geen doelen gedetecteerd

**Mogelijke oorzaken:**

* Doelafbeeldingen zijn niet gemarkeerd in de bestandsbrowser
* Doel is te klein in het kader (&lt; 30% van de afbeelding)
* Slechte belichting (schaduwen, schittering)
* Instellingen voor doelwitdetectie te streng

**Oplossingen:**

1. Controleer of de kolom Doelwit is aangevinkt voor de juiste afbeeldingen
2. Controleer de kwaliteit van de doelwitafbeelding in het voorbeeld
3. Leg de doelwitten opnieuw vast als de kwaliteit slecht is
4. Pas indien nodig de instellingen voor doelwitdetectie aan

### Probleem: valse doelwitdetecties

**Mogelijke oorzaken:**

* Witte gebouwen, voertuigen of bodembedekking die voor doelen worden aangezien
* Heldere vlekken in de vegetatie
* Detectiegevoeligheid te laag

**Oplossingen:**

1. Markeer alleen daadwerkelijke doelafbeeldingen om het detectiebereik te beperken
2. Vergroot het minimale kalibratiemonstergebied
3. Verhoog de minimale doelclusteringwaarde
4. Zorg ervoor dat doelafbeeldingen alleen het doel tonen (minimale achtergrondruis)

***

## Verificatiechecklist

Controleer voordat u met de verwerking begint uw selectie van doelafbeeldingen:

* [ ] Minstens 1 doelafbeelding gemarkeerd per sessie
* [ ] De selectievakjes in de doelkolom zijn aangevinkt voor alle doelafbeeldingen
* [ ] Doelafbeeldingen vastgelegd binnen hetzelfde tijdsbestek als het onderzoek
* [ ] Doelen duidelijk zichtbaar in het voorbeeld wanneer erop wordt geklikt
* [ ] Alle 4 kalibratiepanelen zichtbaar in elke doelafbeelding
* [ ] Geen schaduwen of obstructies op doelen
* [ ] Voor dubbele camera: doelen gemarkeerd voor beide cameratypes

***

## Doelvrije verwerking

### Verwerking zonder kalibratiedoelen

Hoewel dit niet wordt aanbevolen voor wetenschappelijk werk, kunt u zonder doelen verwerken:

1. Laat alle selectievakjes in de kolom Doel leeg.
2. **Schakel** &quot;Reflectiekalibratie&quot; uit in Projectinstellingen.
3. Vignettecorrectie wordt nog steeds toegepast
4. De uitvoer wordt niet gekalibreerd voor absolute reflectie

{% hint style=&quot;warning&quot; %}
**Niet aanbevolen**: zonder reflectiekalibratie geven pixelwaarden alleen de relatieve helderheid weer, niet de wetenschappelijke reflectiemetingen. Gebruik kalibratiedoelen voor nauwkeurige, herhaalbare resultaten.
{% endhint %}

***

## Volgende stappen

Nadat u uw doelafbeeldingen hebt gemarkeerd:

1. **Controleer uw instellingen** - Zie [Projectinstellingen aanpassen](adjusting-project-settings.md)
2. **Start de verwerking** - Zie [De verwerking starten](starting-the-processing.md)
3. **Controleer de voortgang** - Zie [De verwerking controleren](monitoring-the-processing.md)

Voor meer informatie over kalibratiedoelen zelf, zie [Kalibratiedoelen](../calibration-targets.md).
