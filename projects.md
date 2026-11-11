# GUI: Projecten Met

Chloros kun je projecten aanmaken die je later weer kunt openen. Een project is een gewone map (in je projectmap) die het volgende bevat:

* `project.json` — projectinstellingen, bestandslijst en weergavevoorkeuren
* `cameras.json` — camera’s en arrays die waren aangesloten terwijl het project geopend was, met hun instellingen
* `sensors.json` — DAQ-lichtsensoren die waren aangesloten terwijl het project open was, plus koppelingen tussen camera’s en sensoren
* je opnames, `.daq`-opnames en mappen met verwerkte uitvoer

Er is geen eigen projectbestandsformaat — de map en de JSON-bestanden daarin vormen het project, waardoor projecten ook eenvoudig kunnen worden gekopieerd, gearchiveerd en aangestuurd via de [CLI](CLI.md) of de [Python SDK](api-python-sdk.md).

Tenzij je dit wijzigt in [Projectinstellingen](project-settings/project-settings.md), is de projectmap `Documents\MAPIR\Chloros Projects` in je map ‘Documenten’ (`~/Documents/MAPIR/Chloros Projects` op Linux).

{% hint style="warning" %}
**Windows: houd projectmappen kort, tenzij ‘lange paden’ is ingeschakeld.**Windows wordt geleverd met ‘lange paden’ uitgeschakeld, en dan mag geen enkel bestandspad langer zijn dan 260 tekens. Verwerkte producten worden tot ongeveer 130 tekens onder de projectmap opgeslagen, dus vanaf versie 1.3.0 weigert Chloros een projectmap die te diep ligt — wanneer je deze aanmaakt of opent, of wanneer je de verwerking start — in plaats van het programma uit te voeren en niets op te slaan. Het bericht geeft de langste projectmap aan die nog past. Gebruik een kortere projectmap, of schakel lange paden in Windows in (de optie**Enable Windows long paths** van het installatieprogramma, standaard aangevinkt) en start Chloros opnieuw op. Zie [De verwerking starten](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Nieuw project

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Selecteer „Nieuw project“ in het hoofdmenu en voer een unieke naam voor je project in.

Als je projectsjablonen hebt opgeslagen, verschijnt er onder het naamveld een vervolgkeuzelijst **Sjabloon selecteren** — als je er een kiest, wordt het nieuwe project gestart op basis van de instellingen van dat sjabloon. Sjablonen worden opgeslagen via [Projectinstellingen](project-settings/project-settings.md): voer een naam in het veld „Naam projectsjabloon opslaan“ in en klik op het opslagpictogram.

## Project openen

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>Onder „Project openen“ worden alle projecten in je projectmap weergegeven, met <strong>Projectmap openen</strong> onderaan</p></figcaption></figure>

Selecteer „Project openen“ om een lijst met bestaande projecten in de projectmap te bekijken. Als er geen projecten zijn, wordt het secundaire zijmenu niet geopend. Op de bovenstaande foto ziet u enkele via de GUI aangemaakte projecten (t1, t2, t3) in de lijst staan. De DATE\_TIME-projecten zijn aangemaakt door de CLI, waarbij het standaard naamgevingsschema voor projecten is gebruikt. Als je op een projectnaam klikt, wordt het project geopend.

Als je op de knop „Projectmap openen“ klikt, wordt de bestandsverkenner van je computer geopend op het projectpad. U kunt het projectpad aanpassen in de [Projectinstellingen](project-settings/project-settings.md).

Als een van de bronafbeeldingsbestanden van het project is verplaatst of verwijderd sinds het project voor het laatst werd geopend, toont Chloros een dialoogvenster met een lijst van de bestanden die precies ontbreken, in plaats van een leeg raster te openen.

## Project dupliceren

Beschikbaar zodra een project is geopend. Selecteer &#x27;Project dupliceren&#x27; om het huidige project onder een nieuwe naam te kopiëren — Chloros stelt de volgende beschikbare naam voor (bijv. &#x27;MijnProject (2)&#x27;) — en het duplicaat wordt onmiddellijk geopend.

## Bestanden toevoegen

Nadat een project is geopend, selecteert u „Bestanden toevoegen” in het hoofdmenu om afzonderlijke afbeeldingsbestanden aan het huidige project toe te voegen. Dit komt overeen met de toevoegfunctie van de bestandsbrowser, maar is voor het gemak rechtstreeks toegankelijk vanuit het hoofdmenu.

## Map toevoegen

Nadat een project is geopend, selecteer je &quot;Map toevoegen&quot; in het hoofdmenu om mappen met afbeeldingen aan het huidige project toe te voegen. Je kunt in één keer meerdere mappen selecteren. Dubbele bestanden worden genegeerd.

## Verwerking starten / stoppen

Nadat bestanden aan een project zijn toegevoegd, wordt „Verwerking starten“ beschikbaar in het hoofdmenu. Dit is dezelfde actie als het klikken op de knop Afspelen/Starten in de bovenste koptekst. Tijdens de verwerking verandert het menu-item in „Verwerking stoppen“, zodat u de verwerkingsstroom kunt onderbreken.

## Verbinden met camera / Verbinden met lichtsensor

Onderaan het hoofdmenu bevinden zich twee hardware-snelkoppelingen, die beschikbaar zijn ongeacht of er een project openstaat:

* **Verbinden met camera** — opent het [tabblad Camera’s](lattice/) om een LATTICE-camera of -array aan te sluiten.
* **Verbinden met lichtsensor** — opent het [tabblad Lichtsensoren](daq/) om een DAQ-lichtsensor aan te sluiten.

Als je hardware aansluit terwijl er een project openstaat, wordt dit in het project opgeslagen (zie hieronder). Zonder project zijn verbindingen alleen geldig voor de huidige sessie.

{% hint style="info" %}
De menu-items **Bestanden toevoegen**,**Map toevoegen**en**Verwerking starten/stoppen** zijn alleen zichtbaar of ingeschakeld wanneer er een project is geopend en er bestanden zijn toegevoegd. Ze bieden snelle toegang tot acties die ook beschikbaar zijn via de zijbalk van de bestandsbrowser en de knoppen in de koptekst.
{% endhint %}

## Projecten onthouden je hardware

Nieuw in 1.2.0: een project onthoudt de hardware die je aansluit zolang het geopend is. Van camera’s en arrays (met hun instellingen per camera, namen, kleuren en rasterindeling) wordt automatisch een momentopname gemaakt in `cameras.json`, en van lichtsensoren (met namen, kleuren en camerakoppelingen) in `sensors.json` — automatisch, terwijl je aan het werk bent.

Wanneer je een project **opnieuw opent**, raakt Chloros de hardware niet onmiddellijk aan. Elk onderdeel maakt opnieuw verbinding zodra je het tabblad bezoekt waartoe het behoort:

* Als je het tabblad **Camera’s** opent, worden de opgeslagen camera’s en arrays opnieuw verbonden en worden hun opgeslagen instellingen opnieuw toegepast.
* Als je het tabblad **Lichtsensoren**opent, worden de opgeslagen DAQ-sensoren opnieuw verbonden en worden hun opgeslagen instellingen opnieuw toegepast, inclusief de kap. Een sensor die in het Cap-menu is ingesteld op**Geen (kale sensor)**, blijft onbedekt. Een project dat is opgeslagen door een eerdere versie van Chloros heeft een kale sensor geregistreerd als geen keuze, dus deze wordt weer weergegeven met de cosinuscorrectie voor zonlicht; stel deze opnieuw in op Geen en sla het project op.

Op deze manier worden camera’s nooit in de streamingmodus gezet wanneer je een project opent om alleen maar door beelden te bladeren of deze te exporteren. Als een opgeslagen apparaat niet kan worden gevonden wanneer het tabblad wordt geopend, geeft een dialoogvenster aan welke apparaten niet beschikbaar zijn, zodat je ze opnieuw kunt verbinden of verwijderen.

## DAQ-opnames en .daq-bestanden in een project

* `.daq`-opnames die zijn gemaakt terwijl het project openstaat (vanaf het tabblad ‘Lichtsensoren’ of tijdens opnames) worden **automatisch aan het project toegevoegd**.
* Geïmporteerde `.daq`-bestanden en alle projectopnames worden weergegeven in het gedeelte **DAQ-lichtsensor** van [Projectinstellingen](project-settings/project-settings.md), elk met hun eigen cap-correctieprofiel.
* Tijdens de verwerking leveren de `.daq`-bestanden van het project neerwaartse verlichting voor reflectantieproducten — zie [Uitvoerbeeldformaten](output-image-formats.md).

## Een opgeslagen project zonder GUI uitvoeren

Een opgeslagen project kan zonder de GUI worden uitgevoerd:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` werkt op basis van een projectmap-pad — zie de [CLI-referentie](reference/cli-reference.md).
* **de SDK**: `chloros_sdk.open_project(path)` retourneert een projecthandle; `connect_all()` zet elke opgeslagen camera, array en lichtsensor online met de opgeslagen instellingen (lichtsensoren via een actieve Chloros-backend) — zie de [SDK-referentie](reference/sdk-reference.md).
