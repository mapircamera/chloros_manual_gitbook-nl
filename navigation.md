# GUI: Navigatie

Wanneer je Chloros voor het eerst start, wordt de verwerkingsbackend opgestart. Zodra de backend gereed is, verschijnt het pictogram van het hoofdmenu linksboven <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> en worden de tabbladen ‘Camera’s’ en ‘Lichtsensoren’ in de linkerzijbalk ontgrendeld (tot dan toe zijn ze grijs weergegeven).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Van links naar rechts bevat de bovenste koptekst:

### Hoofdmenu ‘<img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">’

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Vanuit het hoofdmenu kunt u:

* **Nieuw project**— een nieuw project aanmaken. Als u projectsjablonen hebt opgeslagen, verschijnt er een vervolgkeuzemenu**Sjabloon selecteren**, zodat het nieuwe project begint met de instellingen van een sjabloon.
* **Project openen**— een bestaand project openen. De lijst bevat een knop**Projectmap openen** waarmee je de projectmap in je bestandsverkenner opent.
* **Project dupliceren** — kopieer het momenteel geopende project onder een nieuwe naam (er wordt een vrije naam zoals &quot;MijnProject (2)&quot; voorgesteld) en open de kopie. _(zichtbaar nadat een project is geopend)_
* **Bestanden toevoegen** — voeg afzonderlijke afbeeldingsbestanden toe aan het huidige project _(zichtbaar nadat een project is geopend)_
* **Map toevoegen** — voeg een of meer mappen met afbeeldingen toe aan het huidige project _(zichtbaar nadat een project is geopend)_
* **Verwerking starten / Verwerking stoppen** — start of stop de beeldverwerkingspijplijn _(beschikbaar nadat bestanden zijn toegevoegd)_
* **Verbinden met camera** — ga naar het [tabblad Camera&#x27;s](lattice/) om een LATTICE-camera of -array aan te sluiten. Werkt ook zonder dat er een project is geopend.
* **Verbinden met lichtsensor** — ga naar het [tabblad Lichtsensoren](daq/) om een DAQ-lichtsensor aan te sluiten. Werkt ook als er geen project is geopend.

{% hint style="info" %}
**Alleen Windows**: De Chloros Desktop GUI is beschikbaar op Windows. Gebruikers van [Linux](CLI.md) en [Python SDK](api-python-sdk.md) raadplegen voor headless-verwerking.
{% endhint %}

###<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Afspeel-/startknop

Wanneer deze is ingeschakeld, start de knop ‘Verwerking starten’ de beeldverwerkingspijplijn.

###<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Voortgangsbalk<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

In de gratis modus van Chloros, die alle bestanden achtereenvolgens verwerkt, toont de voortgangsbalk 2 fasen: Doeldetectie en Verwerking.

In de betaalde Chloros+-modus, waarin alle bestanden gelijktijdig worden verwerkt, toont de voortgangsbalk 4 fasen: Detecteren, Analyseren, Kalibreren, Exporteren. Als u de muiscursor boven de voortgangsbalk van Chloros+ houdt, verschijnt het uitgebreide paneel met de vier voortgangsbalkfasen, zodat u de voortgang kunt volgen. Als u op de bovenste voortgangsbalk klikt, wordt het uitklapbare paneel vastgezet; klikt u nogmaals, wordt het weer vrijgegeven.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Zijmenu

Het menu in de linkerzijbalk bevat verschillende pictogrammen waarmee u kunt werken, in deze volgorde van boven naar beneden:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projectinstellingen](project-settings/project-settings.md)

Op het tabblad Projectinstellingen kunt u de algemene projectinstellingen en de instellingen voor de projectverwerking aanpassen. Pas deze aan voordat je begint met het verwerken van je bestanden.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Bestandsbrowser

Voeg bestanden/mappen toe aan het project of verwijder ze eruit. Dubbele bestanden worden genegeerd. Vink het vakje in de kolom ‘doel’ aan voor elke doelafbeelding; de verwerking kijkt dan alleen naar de aangevinkte afbeeldingen voor doelen, wat de verwerkingstijd aanzienlijk versnelt. Gebruik de schakelaar „Afbeelding/Metadata“ om te schakelen tussen het bekijken van het miniatuurraster van de geselecteerde afbeelding en een gedetailleerde metadatatabel.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Afbeeldingsviewer](image-viewer-gui/opening-an-image-full-screen.md)

Wanneer u op een afbeelding in de hoofdafbeeldingsviewer klikt, wordt deze op volledig scherm geopend in het tabblad ‘Afbeeldingsviewer’.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Kaartviewer](image-viewer-gui/map-markers.md)

Bekijk uw afbeeldingen op een interactieve 2D-kaart op basis van hun GPS-coördinaten. Ondersteunt Google Maps en ESRI-tegelproviders, waarbij automatisch de beste dienst voor uw locatie wordt geselecteerd. Beweeg de muis over markeringen om miniatuurvoorbeelden van afbeeldingen te zien.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Camera’s](lattice/)

Maak verbinding met LATTICE-camera’s en bedien ze live — één voor één of als gesynchroniseerde opstellingen met meerdere camera’s. Het tabblad toont live voorbeeldtegels met overlays en histogrammen, instellingen per camera en per opstelling, en opname-instellingen waarmee u kunt kiezen welke camera’s en exportformaten „Capture All“ genereert. Beschikbaar zodra de backend gereed is; zie de [LATTICE-sectie](lattice/) voor de volledige handleiding.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Lichtsensoren](daq/)

Sluit DAQ-lichtsensoren aan — DAQ-U (USB), DAQ-M (Bluetooth) en DAQ-E (Ethernet) — en bekijk hun live gekalibreerde spectrumgrafieken in W/m²/nm. Vanaf hier kun je `.daq`-bestanden opnemen in het geopende project, sensoren hernoemen, profielen voor condensatorcorrectie selecteren en de DAQ-E-firmware bijwerken. Beschikbaar zodra de backend gereed is; zie de [DAQ-sectie](daq/) voor de volledige handleiding.

#### Debuglogboek van de <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

Bekijk het logboek voor debug-uitvoer wanneer er problemen optreden. Kopieer of download het logboek en stuur het naar [MAPIR Support](https://www.mapir.camera/community/contact) voor hulp.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Gebruikersaanmelding](chloros+-login.md)

Via de zijbalk voor gebruikersaanmelding kunt u inloggen op uw Chloros+-account om geavanceerde functies te ontgrendelen. U kunt ook de huidige versie van de applicatie bekijken en de taal van de weergegeven tekst in de Chloros-GUI en CLI aanpassen.
