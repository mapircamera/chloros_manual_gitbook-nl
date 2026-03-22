# GUI: Navigatie

Wanneer je Chloros en Chloros (Browser) voor het eerst start, wordt de backend opgestart. Zodra deze gereed is, verschijnt het pictogram van het hoofdmenu linksboven <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Van links naar rechts bevat de bovenste koptekst:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hoofdmenu

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Vanuit het hoofdmenu kunt u:

* **Nieuw project** — een nieuw project aanmaken
* **Project openen** — een bestaand project openen
* **Projectmap openen** — de projectmap openen in uw bestandsverkenner
* **Bestanden toevoegen** — afzonderlijke afbeeldingsbestanden toevoegen aan het huidige project _(zichtbaar nadat een project is geopend)_
* **Map toevoegen** — een map met afbeeldingen toevoegen aan het huidige project _(zichtbaar nadat een project is geopend)_
* **Verwerking starten / Verwerking stoppen** — de afbeeldingsverwerkingspijplijn starten of stoppen _(ingeschakeld nadat bestanden zijn toegevoegd)_

{% hint style="info" %}
**Alleen Windows**: De Chloros Desktop GUI is beschikbaar op Windows. Gebruikers van Linux zouden de [CLI](CLI.md) en [Python SDK](api-python-sdk.md) documentatie raadplegen voor headless verwerking.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Afspelen/Startknop

Wanneer ingeschakeld, start de knop voor het starten van de verwerking de pijplijn voor beeldverwerking.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Voortgangsbalk <img src=".gitbook/assets/image (5).png" alt="" data-size="line">In de gratis Chloros-modus, die alle bestanden achtereenvolgens verwerkt, toont de voortgangsbalk 2 fasen: Doel detecteren en Verwerken.

In de betaalde Chloros+-modus, die alle bestanden gelijktijdig verwerkt, toont de voortgangsbalk 4 fasen: Detecteren, Analyseren, Kalibreren, Exporteren. Als u de muiscursor boven de voortgangsbalk van Chloros+ houdt, verschijnt er een uitklapbaar paneel met de 4 voortgangsbalkfasen, zodat u de voortgang kunt volgen. Als u op de bovenste voortgangsbalk klikt, wordt het uitklapbare paneel vastgezet; als u nogmaals klikt, wordt het weer vrijgegeven.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Zijmenu

Het menu in de linkerzijbalk bevat verschillende pictogrammen waarmee u kunt werken:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projectinstellingen](project-settings/project-settings.md)

Op het tabblad Projectinstellingen kunt u de algemene projectinstellingen en de verwerkingsinstellingen voor het project aanpassen. Pas deze aan voordat u begint met het verwerken van uw bestanden.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Bestandsbrowser

Voeg bestanden/mappen toe aan het project of verwijder ze. Dubbele bestanden worden genegeerd. Vink het vakje in de doelkolom aan voor elke doelafbeelding, en de verwerking zal alleen gekeken naar aangevinkte afbeeldingen voor doelen, wat uw verwerkingstijd aanzienlijk versnelt. Gebruik de schakelaar Afbeelding/Metadata om te schakelen tussen het bekijken van het miniatuurraster van de geselecteerde afbeelding en een gedetailleerde metadatatabel.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Afbeeldingsviewer](image-viewer-gui/opening-an-image-full-screen.md)

Wanneer u op een afbeelding klikt in de hoofdafbeeldingsviewer, wordt deze op volledig scherm geopend in het tabblad Afbeeldingsviewer.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Kaart](image-viewer-gui/map-markers.md)

Bekijk uw afbeeldingen op een interactieve 2D-kaart op basis van hun GPS-coördinaten. Ondersteunt Google Maps en ESRI-tegelproviders, waarbij automatisch de beste dienst voor uw locatie wordt geselecteerd. Beweeg de muis over markeringen om miniatuurvoorbeelden van afbeeldingen te zien.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Foutlogboek

Bekijk het logboek voor foutmeldingen wanneer er problemen optreden. Kopieer/download het logboek en stuur het naar [MAPIR Support](https://www.mapir.camera/community/contact) voor hulp.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Gebruikersaanmelding](chloros+-login.md)

Via de zijbalk voor gebruikersaanmelding kunt u inloggen op uw Chloros+-account om geavanceerde functies te ontgrendelen. U kunt ook de huidige versie van de applicatie bekijken en de taal van de weergegeven tekst in de Chloros-GUI en CLI aanpassen.
