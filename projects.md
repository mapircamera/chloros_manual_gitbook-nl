# GUI: Projecten

Met Chloros kun je projecten aanmaken die je later weer kunt openen. Een project is een gewone map (in je projectmap) die het volgende bevat:

* `project.json` — projectinstellingen, bestandslijst en weergavevoorkeuren
* `cameras.json` — camera’s en arrays die waren aangesloten terwijl het project geopend was, inclusief hun instellingen
* `sensors.json` — DAQ-lichtsensoren die waren aangesloten terwijl het project openstond, plus koppelingen tussen camera’s en sensoren
* je opnames, `.daq`-opnames en mappen met verwerkte output

Er is geen eigen projectbestandsformaat — de map en de daarin opgenomen JSON-bestanden vormen het project, waardoor projecten ook eenvoudig kunnen worden gekopieerd, gearchiveerd en vanaf de [CLI](CLI.md) of [Python SDK](api-python-sdk.md).

## Nieuw project

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Selecteer „Nieuw project“ in het hoofdmenu en voer een unieke naam voor je project in.

Als je projectsjablonen hebt opgeslagen, verschijnt er onder het naamveld een vervolgkeuzemenu **Sjabloon selecteren** — als je er een kiest, wordt het nieuwe project gestart op basis van de instellingen van dat sjabloon. Sjablonen worden opgeslagen via [Projectinstellingen](project-settings/project-settings.md): voer een naam in het veld „Naam projectsjabloon opslaan“ in en klik op het opslagpictogram.

## Project openen

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>Onder „Project openen“ worden alle projecten in je projectmap weergegeven, met onderaan <strong>de optie „Projectmap openen“</strong></p></figcaption></figure>Selecteer „Project openen“ om een lijst met bestaande projecten in de projectmap te bekijken. Als er geen projecten zijn, wordt het secundaire zijmenu niet geopend. Op de bovenstaande foto ziet u enkele via de GUI aangemaakte projecten (t1, t2, t3). De DATE\_TIME-projecten zijn aangemaakt door CLI met behulp van het standaard naamgevingsschema voor projecten. Als u op een projectnaam klikt, wordt het project geopend.

Als je op de knop &quot;Projectmap openen&quot; klikt, wordt de bestandsverkenner van je computer geopend op het projectpad. Je kunt het projectpad aanpassen in de [Projectinstellingen](project-settings/project-settings.md).

Als een van de bronafbeeldingsbestanden van het project is verplaatst of verwijderd sinds het voor het laatst werd geopend, toont Chloros een dialoogvenster met een lijst van de exacte bestanden die ontbreken, in plaats van een leeg raster te openen.

## Project dupliceren

Beschikbaar zodra een project is geopend. Selecteer &quot;Project dupliceren&quot; om het huidige project onder een nieuwe naam te kopiëren — Chloros stelt de volgende beschikbare naam voor (bijv. &quot;MijnProject (2)&quot;) — en het duplicaat wordt onmiddellijk geopend.

## Bestanden toevoegen

Nadat een project is geopend, selecteert u „Bestanden toevoegen” in het hoofdmenu om afzonderlijke afbeeldingsbestanden aan het huidige project toe te voegen. Dit komt overeen met de toevoegfunctie van de bestandsbrowser, maar is voor het gemak rechtstreeks toegankelijk vanuit het hoofdmenu.

## Map toevoegen

Nadat een project is geopend, selecteer je &quot;Map toevoegen&quot; in het hoofdmenu om mappen met afbeeldingen aan het huidige project toe te voegen. Je kunt in één keer meerdere mappen selecteren. Dubbele bestanden worden genegeerd.

## Verwerking starten / stoppen

Nadat bestanden aan een project zijn toegevoegd, wordt ‘Verwerking starten’ beschikbaar in het hoofdmenu. Dit is hetzelfde als klikken op de knop Afspelen/Starten in de bovenste koptekst. Tijdens de verwerking verandert het menu-item in ‘Verwerking stoppen’, zodat u de verwerkingsstroom kunt onderbreken.

## Verbinden met camera / Verbinden met lichtsensor

Onderaan het hoofdmenu bevinden zich twee hardwaresnelkoppelingen, die beschikbaar zijn ongeacht of er een project openstaat:

* **Verbinden met camera** — opent het [tabblad Camera’s](lattice/) om een LATTICE-camera of -array aan te sluiten.
* **Verbinden met lichtsensor** — opent het [tabblad Lichtsensoren](daq/) om een DAQ-lichtsensor aan te sluiten.

Als je hardware aansluit terwijl er een project openstaat, wordt dit in het project opgeslagen (zie hieronder). Zonder project zijn verbindingen alleen geldig voor de huidige sessie.

{% hint style="info" %}
De menu-items **Bestanden toevoegen**,**Map toevoegen**en**Verwerking starten/stoppen**zijn alleen zichtbaar of ingeschakeld wanneer er een project is geopend en er bestanden zijn toegevoegd. Ze bieden snelle toegang tot acties die ook beschikbaar zijn via de zijbalk**Bestandsbrowser** en de knoppen in de koptekst.
{% endhint %}

## Projecten onthouden je hardware

Nieuw in 1.2.0: een project onthoudt de hardware die je aansluit zolang het geopend is. Van camera’s en arrays (met hun instellingen per camera, namen, kleuren en rasterindeling) wordt een momentopname gemaakt in `cameras.json`, en van lichtsensoren (met namen, kleuren en camerakoppelingen) in `sensors.json` — automatisch, terwijl je aan het werk bent.

Wanneer je een project **opnieuw opent**, maakt Chloros niet onmiddellijk verbinding met de hardware. Elk onderdeel maakt opnieuw verbinding zodra je het tabblad bezoekt waartoe het behoort:

* Als je het tabblad **Camera’s** opent, worden de opgeslagen camera’s en arrays opnieuw verbonden en worden hun opgeslagen instellingen opnieuw toegepast.
* Als je het tabblad **Lichtsensoren** opent, worden de opgeslagen DAQ-sensoren opnieuw aangesloten.

Op deze manier zorgt het openen van een project, alleen om afbeeldingen te bekijken of te exporteren, er nooit voor dat camera’s in de streammodus worden gezet. Als een opgeslagen apparaat niet kan worden gevonden wanneer het bijbehorende tabblad wordt geopend, geeft een dialoogvenster aan welke apparaten niet beschikbaar zijn, zodat je ze opnieuw kunt aansluiten of verwijderen.

## DAQ-opnames en .daq-bestanden in een project

* `.daq`-opnames die zijn gemaakt terwijl het project geopend is (vanuit het tabblad **Lichtsensoren**of tijdens opnames) worden**automatisch aan het project toegevoegd**.
* Geïmporteerde `.daq`-bestanden en alle projectopnames worden weergegeven in het gedeelte **DAQ-lichtsensor** van [Projectinstellingen](project-settings/project-settings.md), elk met hun eigen cap-correctieprofiel.
* Tijdens de verwerking leveren de `.daq`-bestanden van het project neerwaartse verlichting voor reflectantieproducten — zie [Uitvoerbeeldformaten](output-image-formats.md).

## Een opgeslagen project zonder GUI uitvoeren

Een opgeslagen project kan zonder de GUI worden uitgevoerd:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` werkt op basis van een projectmap-pad — zie de [CLI-referentie](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` retourneert een project-handle; `connect_all()` zet elke opgeslagen camera en sensor online met de opgeslagen instellingen — zie de [SDK-referentie](reference/sdk-reference.md).
