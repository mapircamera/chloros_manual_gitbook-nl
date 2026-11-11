# Chloros gebruiken met AI-assistenten

Deze handleiding is geschreven voor twee doelgroepen: mensen en de AI-assistenten waarmee mensen steeds vaker werken. Op elke pagina staan exacte waarden, standaardinstellingen en kopieer- en plakbare commando's, zodat een assistent (Claude, ChatGPT, Copilot, een programmeeragent, …) in één keer werkende Chloros-automatisering kan schrijven.

Chloros-versie: **

1.3.0**. CLI-/SDK-platforms: Windows 10/11 x64 en Linux (x86_64 / Jetson aarch64).

## Wat je je assistent moet geven

| Bron | URL | Waarvoor dient het |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Machinaal leesbare index van elke pagina in deze handleiding. |
| **Referentie CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Het volledige `chloros-cli`-opdrachtaanbod: elke opdracht, vlag, standaardinstelling, exitcode en regel voor de uitvoermap. Geschreven voor gebruik door LLM. |
| **SDK-referentie** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | De volledige `chloros_sdk` Python-API: klassen, signaturen, uitzonderingen en uitgewerkte voorbeelden. Geschreven voor gebruik door LLM. |
| **Elke pagina als onbewerkte Markdown** | voeg `.md` toe aan de URL van de pagina | bijv. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` geeft de pagina weer als onbewerkte Markdown — ideaal om in een contextvenster te plakken of op te halen via een agent. |

Links in de handleiding: [CLI-referentie](reference/cli-reference.md) · [SDK-referentie](reference/sdk-reference.md).

{% hint style="info" %}
De twee referentiepagina’s staan op zichzelf: een assistent die een van beide heeft gelezen, heeft de rest van de handleiding niet nodig om een correct script te schrijven.
{% endhint %}

## Laat je assistent Chloros rechtstreeks uitvoeren: `chloros-mcp`

Bij alles hierboven moet je assistent een script of een commando *schrijven*. Met `chloros-mcp` stuurt een assistent die het [Model Context Protocol](https://modelcontextprotocol.io) (MCP) spreekt, zoals Claude Desktop, Claude Code of Cursor, Chloros zelf aan. Je vraagt het in gewone taal en het roept de tools van de server aan:

* **Beelden verwerken:** maak een project aan, importeer een map met opnames, kies instellingen, verwerk de beelden en rapporteer wat de run heeft geëxporteerd.
* **Hardware bedienen:** LATTICE-camera’s, arrays en DAQ-lichtsensoren aansluiten, een voorbeeld bekijken, belichting en kleur instellen, opnames maken, registreren en firmware bijwerken.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

De tools van de server omvatten wat de [Python SDK](api-python-sdk.md) doet. Ze worden samen met de SDK geïnstalleerd, dus er hoeft meestal niets extra’s te worden geïnstalleerd: op Windows plaatst het Chloros-installatieprogramma `chloros-mcp.exe` in de `Scripts`-map van je Python, en de Linux-versie `.deb` plaatst `chloros-mcp` in `/usr/bin`.

**Claude-code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**voeg dit toe aan `claude_desktop_config.json` (open dit via**Instellingen > Ontwikkelaar > Configuratie bewerken**) en start vervolgens Claude Desktop opnieuw op.**Cursor** gebruikt dezelfde invoer in `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Als de client aangeeft dat de server niet kon worden gestart, staat `chloros-mcp` waarschijnlijk niet in de `PATH` die hij ziet: geef in plaats daarvan het volledige pad op (`where chloros-mcp` op Windows, `which chloros-mcp` op Linux).

Chloros moet op dezelfde computer zijn geïnstalleerd en je moet daar zijn aangemeld, in de app of via `chloros-cli login`, op een betaald Chloros+-abonnement (Copper of hoger). De server kan niet voor je inloggen.

{% hint style="info" %}
Clientconfiguratie voor elke app, opties, elke tool en hoe een run zich gedraagt: [MCP-server (AI-assistenten)](reference/sdk-reference.md#mcp-server-ai-assistants) in de SDK-referentie.
{% endhint %}

## Prompt-recepten

Kopieer, vul de `<placeholders>` in en plak deze in je assistent.

### 1. Verwerk een vluchtmap naar NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Een map met opnames in batch bekijken

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Een LATTICE-array aansluiten en opnames maken

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. DAQ-spectra van lichtsensoren registreren

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-scripting vanaf de opdrachtregel verloopt altijd via de `daq pool-*`-familie (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andere `daq`-subcommando’s die je assistent mogelijk bedenkt, zijn niet beschikbaar in de meegeleverde builds en worden met een foutmelding afgesloten.
{% endhint %}

## Waarom door AI geschreven scripts goed werken met Chloros

Dit zijn stuk voor stuk echte, geverifieerde gedragingen van Chloros 1.3.0 — ze elimineren de klassieke faalpatronen van door machines geschreven automatisering:

* **Geen ingewikkelde installatieprocedure.**De smart-connect-helpers van de SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) en de verwerkings-entrypoints (`ChlorosLocal`, `process_folder`)**starten de lokale backend automatisch**. Een gegenereerd script heeft geen geopende GUI of handmatig gestarte server nodig — het enige wat nodig is, is dat het desktop-/CLI-pakket is geïnstalleerd.
* **De hele pijplijn bestaat uit één aanroep.** `chloros_sdk.process_folder("path", indices=["NDVI"])` voert import → kalibratie → reflectantie → indexexport van begin tot eind uit. Minder oppervlakte, minder plekken waar een gegenereerd script fout kan gaan.
* **Runs zonder uitvoer voeren zelfdiagnose uit.** Na `process()` wordt het overzicht van de run aan het resultaat toegevoegd, en elke verwerkingshint (bijv. *waarom* een run geen uitvoer opleverde) wordt ook opnieuw gegenereerd als een Python-script `UserWarning` — dus zelfs een script dat het resultaatdict nooit controleert, toont de diagnose.
* **De CLI-uitvoer mislukt opvallend.**Een `chloros-cli process`-run die producten heeft aangevraagd maar er geen heeft geschreven, geeft `Processing finished but wrote no image products.` weer en**stopt met een waarde anders dan nul**, zodat shellscripts en CI dit detecteren met een eenvoudige controle van de exitcode. Succesvolle uitvoeringen rapporteren `Image products written: N`, plus `(plus M clamp masks)` wanneer maskers zijn geschreven. Capture-commando’s noemen eveneens elk bestand dat ze niet konden schrijven en beëindigen met `1`.
* **Relatieve paden betekenen wat een script verwacht.** Bij een lokale backend wordt een relatief uitvoerpad (`-o out/` in de CLI, `output_dir="out/"` in de SDK) omgezet naar de map van waaruit het commando of script wordt uitgevoerd.

Een verschil dat een assistent moet weten: de `process()` van de SDK genereert opzettelijk **geen** foutmelding bij een run zonder resultaat — in plaats daarvan wordt dit gemeld via de samenvatting/tips. Als een Python-pijplijn moet stoppen bij een lege run, controleer dan de samenvatting (recept 2 doet dit).

## Waarschuwingen

* **Chloros+-login vereist.**De CLI en SDK vereisen een**betaald** Chloros+-abonnement, wat server-side wordt afgedwongen: verzoeken mislukken met foutcode `401 AUTH_REQUIRED` wanneer je niet bent ingelogd en met `403 PLAN_UPGRADE_REQUIRED` bij het gratis abonnement. Voer `chloros-cli login` één keer per machine uit voordat je de gegenereerde scripts uitvoert. Zie [Chloros+-aanmelding](chloros+-login.md).
* **Capture-opdrachten sturen echte hardware aan.** `lattice` / `daq` / `project`-opdrachten, de SDK-sessieobjecten en de tools van `chloros-mcp` maken verbinding met, streamen en activeren fysieke camera’s en sensoren. Controleer een gegenereerd script voordat je het voor het eerst uitvoert en voer het uit terwijl je bij de hardware aanwezig bent.
* **Controleer steekproefsgewijs de uitvoer.** Controleer de productmappen en enkele pixelwaarden voordat je de resultaten publiceert. Met name reflectantie-TIFF’s worden per bron geschaald — lees de `Chloros:PixelScale` XMP-tag (LATTICE-bestanden bevatten deze: 32768 = 1,0 reflectantie; Survey3-bestanden niet, en gebruiken 65535 = 1,0) in plaats van uit te gaan van een deler. Beide referentiepagina’s documenteren dit onder “Reflectantiepixels lezen”.
* **Kleine valkuilen die de gegenereerde code in de war brengen:**`pool-record` schrijft naar het bestandssysteem van de**backend-host** (standaard `~/Documents/DAQ Live View/`); op machines met meerdere netwerkinterfaces verdient `daq pool-connect --eth-host <ip-or-hostname>` de voorkeur boven automatische detectie; gebruik `http://127.0.0.1:5000` (nooit `localhost`) overal waar een backend-URL voorkomt; een script dat de uitvoerstructuur van een project doorloopt, moet `clamp_masks/`-mappen overslaan (de `*_clampmask.tif`-bestanden daarin zijn maskers, geen afbeeldingen — zie [Uitvoerbeeldformaten](output-image-formats.md#clamp-masks)); en houd op Windows, waar lange paden zijn uitgeschakeld, projectmappen kort — een projectmap die dieper ligt dan ongeveer 130 tekens wordt geweigerd voordat de verwerking begint.
