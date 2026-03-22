# Doelafbeeldingen selecteren

Het markeren van afbeeldingen die kalibratiedoelen bevatten, is een cruciale stap die de verwerkingspijplijn van Chloros aanzienlijk versnelt. Door doelafbeeldingen vooraf te selecteren, hoeft Chloros niet elke afbeelding in uw dataset te scannen op kalibratiedoelen.

## Waarom doelafbeeldingen markeren?

### Verwerkingssnelheid

Zonder het markeren van doelafbeeldingen moet Chloros:

* Elke afzonderlijke afbeelding in uw project scannen
* Doeldetectie-algoritmen uitvoeren op elke afbeelding
* Honderden of duizenden afbeeldingen onnodig controleren

**Resultaat**: De verwerking kan aanzienlijk langer duren, vooral bij grote datasets.

### Met gemarkeerde doelafbeeldingen

Wanneer u de kolom Doel aanvinkt voor specifieke afbeeldingen:

* Scant Chloros alleen de aangevinkelde afbeeldingen op doelen
* Is de doelherkenning veel sneller voltooid
* Wordt de totale verwerkingstijd aanzienlijk verkort

{% hint style="success" %}
**Snelheidsverbetering**: Door 2-3 doelafbeeldingen te markeren in een dataset van 500 afbeeldingen kan de doelherkenningstijd worden teruggebracht van meer dan 30 minuten tot minder dan 1 minuut.
{% endhint %}

***

## Hoe doelafbeeldingen te markeren

### Stap 1: Identificeer uw doelafbeeldingen

Bekijk uw geïmporteerde afbeeldingen in de bestandsbrowser en identificeer welke afbeeldingen kalibratiedoelen bevatten.

**Veelvoorkomende scenario&#x27;s:*** **Doel voor de opname**: vastgelegd vóór het starten van de sessie
* **Doel na de opname**: vastgelegd na het voltooien van de sessie
* **Doelen in het veld**: doelen geplaatst binnen het opnamegebied
* **Meerdere doelen**: 2-3 doelafbeeldingen per sessie (aanbevolen)

### Stap 2: Controleer de kolom Doel

Voor elke afbeelding die een kalibratiedoel bevat:

1. Zoek de afbeelding in de tabel van de bestandsbrowser
2. Zoek de kolom **Doel** (kolom uiterst rechts)
3. Klik op het selectievakje in de kolom Doel voor die afbeelding
4. Herhaal dit voor alle afbeeldingen die doelen bevatten

### Stap 3: Controleer uw selectie

Controleer voor de verwerking nogmaals:

* [ ] Alle afbeeldingen met kalibratiedoelen zijn aangevinkt
* [ ] Er zijn geen afbeeldingen zonder doel per ongeluk aangevinkt
* [ ] Doelen zijn duidelijk zichtbaar in de aangevinkte afbeeldingen

***

## Aanbevolen werkwijzen voor doelafbeeldingen

### Richtlijnen voor het vastleggen van doelen

**Timing:**

* Leg doelafbeeldingen vast vlak voor en tijdens uw opnamesessie
* Onder dezelfde lichtomstandigheden als uw DAQ-lichtsensor
* Leg idealiter zo vaak mogelijk doelafbeeldingen vast voor de beste resultaten. Anders worden de gegevens van de lichtsensor gebruikt om de kalibratie in de loop van de tijd aan te passen.

**Camera-positie:**

* Houd de camera boven het doel, zodat het gecentreerd is en ongeveer 40-60% van het midden van het beeld vult.
* Houd de camera parallel/loodrecht boven het doeloppervlak

**Verlichting:**

* Dezelfde omgevingsverlichting als uw DAQ-lichtsensor
* Vermijd schaduwen op de doeloppervlakken
* Blokkeer uw lichtbron niet met uw lichaam, voertuig of begroeiing
* Bewolkte omstandigheden leveren de meest consistente resultaten op

**Toestand van het doel:**

* Houd de doelpanelen schoon en droog
* Alle 4 panelen moeten duidelijk zichtbaar en onbelemmerd zijn
* Plaats de doelen indien mogelijk loodrecht/nadir ten opzichte van de lichtbron

### Hoeveel doelbeelden?

**Minimum:**1 doelbeeld per sessie.**Aanbevolen:** 3-5 doelbeelden per sessie.**Aanbevolen schema:**

* 3-5 beelden vastgelegd kort nadat de lichtsensor begint met opnemen
* Draai de camera tussen de opnames door voor de beste resultaten
* Optioneel: periodiek halverwege de sessie als de lichtomstandigheden voortdurend veranderen

***

## Werken met meerdere camera&#x27;s

### Opstellingen met twee camera&#x27;s

Als u twee MAPIR-camera&#x27;s tegelijkertijd gebruikt (bijv. Survey3W RGN + Survey3N OCN):

1. Leg doelbeelden vast met **beide camera&#x27;s** tegelijkertijd
2. Gebruik **hetzelfde fysieke doel** voor beide camera&#x27;s
3. Markeer doelbeelden voor **beide cameratypes** in de bestandsbrowser
4. Chloros gebruikt de juiste doelen voor de kalibratie van elke camera

### Kolom Cameramodel

De kolom **Cameramodel** helpt u te bepalen welke beelden van welke camera afkomstig zijn:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* enz.

Gebruik deze kolom om te controleren of u doelen hebt gemarkeerd voor elk cameratype in uw project.

***

## Instellingen voor doelherkenning

### De detectiegevoeligheid aanpassen

Als Chloros uw doelen niet correct detecteert, pas dan deze instellingen aan in [Projectinstellingen](adjusting-project-settings.md):**Minimaal kalibratiesamplegebied:*** **Standaard**: 25 pixels
* **Verhoog** als er valse detecties optreden bij kleine artefacten
* **Verlaag** als doelen niet worden gedetecteerd**Minimale doelclustering:*** **Standaard**: 60
* **Verhoog** als doelen in meerdere detecties worden opgesplitst
* **Verlaag** als doelen met kleurvariatie niet volledig worden gedetecteerd***

## Veelvoorkomende problemen met doelafbeeldingen

### Probleem: geen doelen gedetecteerd

**Mogelijke oorzaken:**

* Doelafbeeldingen niet gemarkeerd in Bestandsbrowser
* Doel te klein in beeld (&lt; 30% van de afbeelding)
* Slechte belichting (schaduwen, schittering)
* Instellingen voor doelherkenning te streng

**Oplossingen:**

1. Controleer of de kolom &#x27;Doel&#x27; is aangevinkt voor de juiste afbeeldingen
2. Controleer de kwaliteit van de doelafbeelding in het voorbeeld
3. Leg doelen opnieuw vast als de kwaliteit slecht is
4. Pas de instellingen voor doelherkenning indien nodig aan

### Probleem: valse doelherkenningen

**Mogelijke oorzaken:**

* Witte gebouwen, voertuigen of bodembedekking die voor doelen worden aangezien
* Heldere plekken in de vegetatie
* Te lage detectiegevoeligheid

**Oplossingen:**

1. Markeer alleen daadwerkelijke doelafbeeldingen om het detectiebereik te beperken
2. Vergroot het minimale kalibratiesteekproefgebied
3. Verhoog de minimale waarde voor doelclustering
4. Zorg ervoor dat doelafbeeldingen alleen het doel tonen (minimale achtergrondruis)

***

## Verificatiechecklist

Controleer uw selectie van doelafbeeldingen voordat u met de verwerking begint:

* [ ] Minimaal 1 doelafbeelding gemarkeerd per sessie
* [ ] De selectievakjes in de kolom &#x27;Doel&#x27; zijn aangevinkt voor alle doelafbeeldingen
* [ ] Doelafbeeldingen zijn vastgelegd binnen hetzelfde tijdsbestek als het onderzoek
* [ ] Doelen zijn duidelijk zichtbaar in het voorbeeld wanneer erop wordt geklikt
* [ ] Alle 4 kalibratiepanelen zijn zichtbaar in elke doelafbeelding
* [ ] Geen schaduwen of obstructies op doelen
* [ ] Voor dubbele camera: Doelen gemarkeerd voor beide cameratypes

***

## Verwerking zonder doelen

### Verwerking zonder kalibratiedoelen

Hoewel dit niet wordt aanbevolen voor wetenschappelijk werk, kunt u verwerken zonder doelen:

1. Laat alle selectievakjes in de kolom Doel uitgeschakeld
2. **Schakel** &quot;Reflectiekalibratie&quot; uit in Projectinstellingen
3. Vignettecorrectie wordt nog steeds toegepast
4. De uitvoer wordt niet gekalibreerd voor absolute reflectie

{% hint style="warning" %}
**Niet aanbevolen**: Zonder reflectiekalibratie geven pixelwaarden alleen de relatieve helderheid weer, geen wetenschappelijke reflectiemetingen. Gebruik kalibratiedoelen voor nauwkeurige, herhaalbare resultaten.
{% endhint %}

***

## Volgende stappen

Zodra u uw doelafbeeldingen hebt gemarkeerd:

1. **Controleer uw instellingen** - Zie [Projectinstellingen aanpassen](adjusting-project-settings.md)
2. **Start de verwerking** - Zie [De verwerking starten](starting-the-processing.md)
3. **Volg de voortgang** - Zie [De verwerking volgen](monitoring-the-processing.md)

Voor meer informatie over kalibratiedoelen zelf, zie [Kalibratiedoelen](../calibration-targets.md).
