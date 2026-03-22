# De verwerking afronden

Zodra Chloros de verwerking heeft voltooid, is het tijd om de resultaten te bekijken, de uitvoerkwaliteit te controleren en de verwerkte afbeeldingen klaar te maken voor gebruik in uw workflow. Op deze pagina vindt u een overzicht van de laatste stappen en de volgende acties.

## Indicatie dat de verwerking is voltooid

Wanneer de verwerking succesvol is voltooid, ziet u verschillende indicatoren:

* ✅ **Voortgangsbalk**: bereikt 100% voltooiing
* ✅ **Foutopsporingslogboek**: toont het bericht &quot;Verwerking voltooid&quot;
* ✅ **Startknop**: wordt weer ingeschakeld (klaar voor de volgende verwerkingsrun)
* ✅ **Uitvoerbestanden**: alle verwerkte afbeeldingen zijn opgeslagen in de submap van het cameramodel***

## Uw verwerkte afbeeldingen vinden

### De uitvoermap openen

1. Klik op het **hoofdmenu** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (linksboven)
2. Selecteer **&quot;Projectmap openen&quot;**

3. Uw bestandsverkenner opent de projectmap
4. Zoek uw project op naam

***

## Verwerkte afbeeldingen bekijken

### Snel voorbeeld in Bestandsverkenner

**Windows ingebouwd voorbeeld:**

1. Navigeer naar de submap van het cameramodel
2. Selecteer een afbeeldingsbestand
3. Het voorbeeld verschijnt in het voorbeeldvenster van Windows Explorer
4. Gebruik de pijltjestoetsen om door de afbeeldingen te bladeren

### Voorbeeld in externe afbeeldingsviewers

**Aanbevolen viewers:*** **QGIS** - Gratis GIS-software (het beste voor georefereerde multispectrale analyse)
* **IrfanView** - Snelle, lichtgewicht afbeeldingsviewer (ondersteunt TIFF)
* **Adobe Photoshop** - Professionele bewerking (ondersteuning voor TIFF)
* **GIMP** - Gratis alternatief voor Photoshop
* **Windows Photos** - Basisweergave (ondersteunt mogelijk geen 16-bits TIFF)

### Voorbeeldweergave in de Chloros-afbeeldingsviewer

Gebruik de ingebouwde afbeeldingsviewer van Chloros voor geavanceerde visualisatie:

1. Klik op een miniatuurafbeelding in de bestandsbrowser
2. De afbeelding wordt geopend in het hoofdvenster voor voorbeeldweergave
3. Klik op het tabblad **Afbeeldingsviewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> in de linkerzijbalk
4. Gebruik [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) voor interactieve analyse

Zie [Afbeeldingsviewer](../image-viewer-gui/opening-an-image-full-screen.md) voor gedetailleerde instructies.

***

## Het debuglogboek bekijken

### Controleer op waarschuwingen of fouten

1. Open het tabblad **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> tabblad
2. Blader door de berichten
3. Zoek naar gele waarschuwingen of rode fouten
4. Bekijk eventuele opgemerkte problemen
5. Neem contact op met MAPIR-ondersteuning voor hulp

### Het logboek opslaan

Om een verslag van de verwerking bij te houden of om naar MAPIR-ondersteuning te sturen:

1. Klik op de knop **&quot;Kopiëren&quot;**of**&quot;Downloaden&quot;**

2. Sla het op als tekstbestand in de projectmap
3. Voeg het toe aan de projectdocumentatie
4. Stuur het naar de ondersteuning van MAPIR als er problemen zijn opgetreden

***

## Veelvoorkomende uitvoerproblemen en oplossingen

### Probleem: Ontbrekende uitvoerbestanden

**Mogelijke oorzaken:**

* Bestanden voldeden niet aan de verwerkingscriteria
* Alleen-doel-afbeeldingen (uitgesloten van export)
* Schijfruimte raakte op tijdens export
* Bestandsbeschadiging tijdens verwerking

**Oplossingen:**

1. Controleer het foutlogboek op oversla- of foutmeldingen
2. Controleer of er voldoende schijfruimte was
3. Tel de bestanden: dit moet overeenkomen met (oorspronkelijk aantal - doel aantal) × (indexen + 1)
4. Importeer ontbrekende bestanden opnieuw en verwerk ze opnieuw

### Probleem: Donkere of lichte randen (vignettering nog steeds zichtbaar)

**Mogelijke oorzaken:**

* Vignetteringscorrectie uitgeschakeld
* Camera/lens staat niet in de Chloros-profieldatabase
* Extreme vignettering die de correctiemogelijkheden te boven gaat

**Oplossingen:**

1. Controleer of vignetteringscorrectie is ingeschakeld in Projectinstellingen
2. Controleer of het cameramodel correct is gedetecteerd
3. Neem contact op met de ondersteuning van MAPIR als de vignettering aanhoudt

### Probleem: Onjuiste kleuren of waarden

**Mogelijke oorzaken:**

* Geen kalibratiedoelen gedetecteerd
* Verkeerd kalibratiedoelmodel geselecteerd
* Reflectiekalibratie uitgeschakeld
* Doelafbeeldingen van slechte kwaliteit

**Oplossingen:**

1. Controleer of reflectiekalibratie is ingeschakeld
2. Controleer de berichten &quot;Doel gevonden&quot; in het foutopsporingslogboek
3. Controleer de kwaliteit van de doelafbeeldingen
4. Voer de verwerking opnieuw uit met de juiste doelen gemarkeerd

### Probleem: NDVI-waarden lijken onjuist

**Verwachte NDVI-bereiken:*** **Water, rotsen, grond**: -0,1 tot 0,2
* **Schaarse/ongezonde vegetatie**: 0,2 tot 0,4
* **Matige vegetatie**: 0,4 tot 0,6
* **Gezonde, dichte vegetatie**: 0,6 tot 0,9**Als waarden buiten deze bereiken vallen:**

1. Controleer of de reflectiekalibratie is toegepast
2. Controleer of het logboek van de lichtsensor is opgenomen
3. Controleer of de kalibratiedoelen zijn gedetecteerd
4. Zorg ervoor dat het juiste cameramodel is gedetecteerd
5. Controleer het tijdstip en de omstandigheden van de opname van de doelafbeelding

***

## Uw verwerkte afbeeldingen gebruiken

### Voor fotogrammetrie / het maken van orthomosaïeken

**Aanbevolen workflow:**

1.**Importeer gekalibreerde reflectantiebeelden** in fotogrammetriesoftware:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Bewaar EXIF-metadata**: Zorg ervoor dat GPS-gegevens behouden blijven voor geotagging
3. **Gekalibreerde workflows**: Gebruik reflectantiebeelden voor wetenschappelijke nauwkeurigheid
4. **Verwerk indexmozaïeken**: Maak NDVI orthomosaïeken van afzonderlijke indexbeelden
5. **Exporteer georefereerde GeoTIFF**: Voor gebruik in GIS-toepassingen

### Voor GIS-analyse

**Aanbevolen workflow:**

1.**Laad in QGIS, ArcGIS of vergelijkbare software**

2.**Gebruik 16-bits TIFF** reflectiebeelden voor multibandanalyse
3. **Gebruik indexbeelden** (NDVI, NDRE) als kant-en-klare vegetatielagen
4. **Rastercalculator**: Combineer banden voor aangepaste analyse
5. **Exporteren**: maak classificatiekaarten, veranderingdetectie en kaarten van de vegetatiegezondheid

### Voor directe analyse / rapportage

**Aanbevolen workflow:**

1.**Gebruik indexbeelden met LUT-kleuren** voor visuele rapporten
2. **Haal statistieken op**: gemiddelde NDVI per veld/perceel
3. **Tijdreeks**: vergelijk indices over meerdere sessies
4. **Genereer rapporten**: voeg kaarten, statistieken en visualisaties toe***

## Archivering en back-up

### Aanbevolen back-upstrategie

**Wat op te slaan:*** ✅ **Originele RAW/JPG-beelden** - Archiveer op een aparte schijf/in de cloud
* ✅ **Verwerkte output** - Bewaar gekalibreerde afbeeldingen en indices
* ✅ **Projectbestand** - Bevat alle instellingen voor herverwerking indien nodig
* ✅ **Debuglog** - Documenteert verwerkingsdetails
* ✅ **Kalibratiedoelafbeeldingen** - Voor verificatie en herverwerking**Aanbevelingen voor opslag:*** **Directe back-up**: Externe harde schijf
* **Langetermijnarchief**: Cloudopslag (Google Drive, Dropbox, enz.)
* **Kritieke gegevens**: Bewaar 2-3 kopieën op verschillende locaties***

## Volgende verwerkingsruns

### Projectinstellingen hergebruiken

Als u in de toekomst vergelijkbare datasets gaat verwerken:

1. **Sla de projectsjabloon op** (indien nog niet gedaan)
2. **Maak een nieuw project** aan met behulp van de opgeslagen sjabloon
3. **Importeer nieuwe afbeeldingen**

4.**Verwerk**met identieke instellingen voor consistentie

### Batchverwerking van meerdere sessies

Voor meerdere sessies/datasets:**Optie 1: GUI - Meerdere projecten**

* Maak voor elke sessie een apart project aan
* Gebruik consistente sjablooninstellingen
* Verwerk ze één voor één

**Optie 2: Chloros CLI (alleen Chloros+)**

* Automatiseer batchverwerking
* Verwerk meerdere mappen met scripts
* Zie [CLI-documentatie](../CLI.md)

**Optie 3: Python SDK (alleen Chloros+)**

* Programmatische besturing
* Integratie met analysepijplijnen
* Zie [API-documentatie](../api-python-sdk.md)

***

## Problemen met nabewerking oplossen

### Opnieuw verwerken met andere instellingen

Als de resultaten niet naar wens zijn:

1. Bewaar de originele afbeeldingen (verwijder ze nooit)
2. Open hetzelfde project in Chloros
3. Pas de instellingen aan in het paneel Projectinstellingen
4. Verwerk opnieuw - de uitvoer overschrijft de vorige resultaten

### Een subset van afbeeldingen verwerken

Om alleen specifieke afbeeldingen opnieuw te verwerken:

1. Maak een nieuw project aan
2. Importeer alleen de afbeeldingen die opnieuw moeten worden verwerkt
3. Gebruik dezelfde instellingensjabloon
4. Verwerk de kleinere dataset

### Hulp krijgen

Als u problemen ondervindt:

* 📧 **E-mail**: info@mapir.camera (voeg het foutopsporingslogboek toe)
* 🌐 **Ondersteuning**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Veelgestelde vragen](../faq.md)
* 📖 **Documentatie**: [Chloros Handleiding](../)***

## Samenvatting: Volledige workflow

U hebt nu de volledige Chloros-verwerkingsworkflow voltooid:

1. ✅ **Project aangemaakt** - Zie [Projecten](../projects.md)
2. ✅ **Bestanden toegevoegd** - Zie [Bestanden toevoegen](adding-files-to-a-project.md)
3. ✅ **Instellingen aangepast** - Zie [Projectinstellingen aanpassen](adjusting-project-settings.md)
4. ✅ **Doelen gemarkeerd** - Zie [Doelafbeeldingen kiezen](choosing-target-images.md)
5. ✅ **Verwerking gestart** - Zie [De verwerking starten](starting-the-processing.md)
6. ✅ **Voortgang gecontroleerd** - Zie [De verwerking controleren](monitoring-the-processing.md)
7. ✅ **Resultaten bekeken** - Deze pagina**Uw gekalibreerde, reflectiegecorrigeerde multispectrale beelden zijn klaar voor analyse!**

***

## Aanvullende bronnen

### Geavanceerde functies

* [**Beeldviewer**](../image-viewer-gui/opening-an-image-full-screen.md) - Interactieve visualisatie en analyse
* [**Index/LUT-sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Aangepaste index-tests
* [**Multispectrale indexformules**](../project-settings/multispectral-index-formulas.md) - Volledige indexreferentie

### Automatisering &amp; Integratie

* [**CLI Documentatie**](../CLI.md) - Batchverwerking via de opdrachtregel
* [**Python SDK**](../api-python-sdk.md) - Programmatische automatisering
* [**Chloros+ Functies**](../#chloros) - Geavanceerde verwerkingsmogelijkheden

### Ondersteuning &amp; Leren

* [**Veelgestelde vragen**](../faq.md) - Antwoorden op veelgestelde vragen
* [**Kalibratiedoelen**](../calibration-targets.md) - Inzicht in reflectiekalibratie
* [**Ondersteunde camera&#x27;s**](../supported-cameras.md) - Compatibele hardware
