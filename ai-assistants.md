# Chloros gebruiken met AI-assistenten

Deze handleiding is bedoeld voor twee doelgroepen: mensen en de AI-assistenten waarmee mensen steeds vaker werken. Op elke pagina staan exacte waarden, standaardinstellingen en kopieer-en-plakbare commando’s, zodat een assistent (Claude, ChatGPT, Copilot, een programmeeragent, …) in één keer werkende Chloros-automatisering kan schrijven.

Chloros-versie: **

1.2.0**. CLI/SDK-platforms: Windows 10/11 x64 en Linux (x86_64 / Jetson aarch64).

## Wat moet je aan je assistent geven?

| Bron | URL | Waarvoor dient het? |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Machinaal leesbare index van elke pagina in deze handleiding. |
| **CLI-referentie** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Het volledige `chloros-cli`-opdrachtoverzicht: elke opdracht, vlag, standaardinstelling, exitcode en regel voor de uitvoermap. Geschreven voor gebruik door LLM. |
| **SDK-referentie** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | De volledige `chloros_sdk` Python API: klassen, signaturen, uitzonderingen en uitgewerkte voorbeelden. Geschreven voor gebruik door LLM-gebruikers. |
| **Elke pagina als onbewerkte Markdown** | voeg `.md` toe aan de pagina URL | bijv. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` geeft de pagina weer als ruwe Markdown — ideaal om in een contextvenster te plakken of op te halen via een agent. |

Links in de handleiding: [CLI Referentie](reference/cli-reference.md) · [SDK Referentie](reference/sdk-reference.md).

{% hint style="info" %}
De twee referentiepagina’s staan op zichzelf: een assistent die een van beide heeft gelezen, heeft de rest van de handleiding niet nodig om een correct script te schrijven.
{% endhint %}

## Promptrecepten

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

### 2. Een map met opnames in batchbewaking zetten

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
DAQ-scripting vanaf de opdrachtregel verloopt altijd via de `daq pool-*`-reeks (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andere `daq`-subcommando’s die je assistent zou kunnen bedenken, zijn niet beschikbaar in de geleverde builds en resulteren in een foutmelding.
{% endhint %}

## Waarom door AI geschreven scripts goed werken met Chloros

Dit zijn allemaal echte, geverifieerde gedragingen van Chloros 1.2.0 — ze elimineren de klassieke faalpatronen van door machines geschreven automatisering:

* **Geen ingewikkelde instellingen.**De ‘smart-connect’-hulpprogramma’s van SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) en de verwerkingsinstap punten (`ChlorosLocal`, `process_folder`)**starten de lokale backend automatisch op**. Een gegenereerd script heeft geen geopende GUI of handmatig gestarte server nodig — het enige wat nodig is, is dat het desktop/CLI-pakket is geïnstalleerd.
* **De hele pijplijn bestaat uit één aanroep.** `chloros_sdk.process_folder("path", indices=["NDVI"])` voert import → kalibratie → reflectantie → indexexport van begin tot eind uit. Minder oppervlakte, minder plaatsen waar een gegenereerd script fout kan gaan.
* **Runs zonder uitvoer voeren een zelfdiagnose uit.** Na `process()` wordt de samenvatting van de run aan het resultaat toegevoegd, en elke verwerkingstip (bijv. *waarom* een run geen output opleverde) wordt ook opnieuw verstuurd als een Python `UserWarning` — dus zelfs een script dat het resultaatdict nooit controleert, geeft de diagnose weer.
* **De CLI mislukt op opvallende wijze.**Een `chloros-cli process`-run die producten heeft aangevraagd maar er geen heeft geschreven, geeft `Processing finished but wrote no image products.` weer en**stopt met een waarde anders dan nul**, zodat shellscripts en CI dit detecteren met een eenvoudige controle van de exitcode. Succesvolle uitvoeringen melden `Image products written: N`.

Een asymmetrie die een assistent moet weten: de `process()` van de SDK genereert opzettelijk **geen** foutmelding bij een run zonder producten — in plaats daarvan rapporteert deze via de samenvatting/tips. Als een Python-pijplijn moet stoppen bij een lege run, controleer dan de samenvatting (recept 2 doet dit).

## Waarschuwingen

* **Chloros+ aanmelding vereist.**De CLI en SDK vereisen een**betaalde** Chloros+-abonnement, dat server-side wordt afgedwongen: verzoeken mislukken met `401 AUTH_REQUIRED` als je niet bent ingelogd en met `403 PLAN_UPGRADE_REQUIRED` op het gratis abonnement. Voer `chloros-cli login` één keer per machine uit voordat je de gegenereerde scripts uitvoert. Zie [Chloros+ Inloggen](chloros+-login.md).
* **Capture-opdrachten sturen echte hardware aan.** De commando’s `lattice` / `daq` / `project` en de sessieobjecten SDK maken verbinding met, streamen en activeren fysieke camera’s en sensoren. Controleer een gegenereerd script voordat het voor het eerst wordt uitgevoerd, en voer het uit terwijl er iemand bij de hardware aanwezig is.
* **Controleer steekproefsgewijs de uitvoer.** Controleer de productmappen en enkele pixelwaarden voordat u de resultaten publiceert. Met name TIFF-bestanden met reflectiegegevens worden per bron geschaald — raadpleeg de `Chloros:PixelScale` XMP-tag (LATTICE: 32768 = reflectie 1,0; Survey3: 65535) in plaats van een deler aan te nemen. Beide referentiepagina’s documenteren dit onder “Reflectiepixels lezen”.
* **Kleine valkuilen die de gegenereerde code in de war brengen:**`pool-record` schrijft naar het bestandssysteem van de**backend-host** (standaard `~/Documents/DAQ Live View/`); op machines met meerdere netwerkinterfaces verdient `daq pool-connect --eth-host <ip-or-hostname>` de voorkeur boven automatische detectie; en gebruik `http://127.0.0.1:5000` (nooit `localhost`) overal waar een backend URL voorkomt.
