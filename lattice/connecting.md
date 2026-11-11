# Camera’s aansluiten

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Het tabblad &#x27;Camera&#x27;s&#x27; voordat er iets is aangesloten</p></figcaption></figure>

Chloros
detecteert LATTICE-camera&#x27;s automatisch op de verbinding — via het tabblad &#x27;Camera&#x27;s&#x27; in de GUI, via `chloros-cli lattice` of via de Python
SDK
. De modelstring van de camera bepaalt alles wat daarna volgt: Chloros
bepaalt het sensorprofiel, de bandindeling en de fabriekskalibratie op basis van de `DeviceUserID` + `DeviceSerialNumber` van de camera, dus **er hoeft per camera niets te worden geconfigureerd**.

Zorg ervoor dat het hostnetwerk is ingesteld voordat je verbinding maakt — link-local-adressering, jumboframes en, voor arrays, de instellingen voor de ontvangstbuffer van de netwerkkaart. Dat is de configuratie aan de hardwarekant en staat beschreven in de LATTICE-handleiding: [**Netwerkconfiguratie**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Verbinding maken via de GUI

Open het tabblad **Camera&#x27;s** in de zijbalk van Chloros
(de hardwaretabbladen verschijnen zodra de backend is opgestart), of gebruik het hoofdmenu → **Verbinden met camera**. Beide openen het dialoogvenster**Camera(&#x27;s) verbinden**.

### Het dialoogvenster **Camera(&#x27;s) verbinden**Het dialoogvenster scant het netwerk zodra het wordt geopend („Netwerk wordt gescand...“) en geeft een lijst weer van alle camera’s die het vindt. Elke rij toont het**model**van de camera (bijv. `LATT-M3M-L41-F550`), het**serienummer**en het**IP-adres**.

* **Klik op een rij om deze te selecteren**(groene markering). U kunt**meerdere camera’s** selecteren en deze in één keer verbinden — Chloros
verbindt ze achtereenvolgens.
* Rijen met het label **&quot;Verbonden&quot;** zijn al verbonden en kunnen niet opnieuw worden geselecteerd.
* Rijen met het label **&quot;In array&quot;** behoren tot een momenteel verbonden camera-array. Verbreek eerst de verbinding met de reeks om die camera afzonderlijk te gebruiken.
* **Verbinden** — verbindt de geselecteerde camera(&#x27;s); de knop toont een aantal, bijvoorbeeld &quot;Verbinden (3)&quot;, wanneer er meer dan één is geselecteerd.
* **Opnieuw scannen** — voert de zoekactie opnieuw uit.
* **Sluiten** — sluit het dialoogvenster.
* Als de scan zonder resultaten wordt voltooid, geeft het dialoogvenster **&quot;Geen camera&#x27;s gevonden op het netwerk&quot;** weer — zie [Probleemoplossing](connecting.md#troubleshooting) hieronder.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Het dialoogvenster Camera(&#x27;s) verbinden — hier weergegeven zonder camera&#x27;s op het netwerk</p></figcaption></figure>

### Eerste verbinding: kalibratiepakket downloaden

De **eerste keer** dat een bepaalde camera op een computer wordt aangesloten, haalt Chloros
het fabriekskalibratiepakket van de camera (\~3,8 MB) via GigE rechtstreeks van de camera zelf op. Terwijl dit proces loopt, toont het dialoogvenster een groen paneel met de tekst **&quot;Kalibratiegegevens downloaden van camera&quot;**en een voortgangsbalk per serienummer — reken op ongeveer**70 seconden** per camera. Bij meer dan zes camera’s scrollen de voortgangsregels binnen het paneel. Het pakket wordt in de cache van de host opgeslagen, zodat bij latere aansluitingen van dezelfde camera het downloaden volledig wordt overgeslagen (en het venster nooit wordt weergegeven).

### Systeem analyseren

De knop **Systeem analyseren** in het dialoogvenster onderzoekt de host en het netwerk (het label luidt „Analyseren...” terwijl het proces loopt) en genereert een diagnostisch rapport:

* **Host** — CPU-kernen en RAM; GPU-naam en geheugen, of „GPU: geen gedetecteerd”.
* **Netwerkinterfaces** — de naam van elke netwerkkaart, verbindingssnelheid, MTU (met een &quot;jumbo&quot;-tag indien actief), up/down-status en of deze op een USB-bus is aangesloten.
* **Camera’s**— serienummer, model, IP-adres en**op welke netwerkkaart elke camera is aangesloten**.
* **Prestaties** — huidige versus ideale fps per camera voor het pixelformaat, met een groene regel „Potentieel: N× verbetering mogelijk” wanneer de ideale waarde hoger is dan de huidige.
* **Waarschuwingen en genummerde aanbevelingen** — of „Systeem ziet er goed uit voor het huidige aantal camera’s.” wanneer er niets te verhelpen valt.

Voer dit uit wanneer detectie of streaming zich onverwacht gedraagt — het identificeert de meeste problemen aan de NIC-zijde (verkeerde MTU, camera op de verkeerde interface, beperkingen van de USB-adapter) zonder het dialoogvenster te verlaten.

### Een array aansluiten

Om twee of meer camera’s als een **gesynchroniseerde array**aan te sluiten, gebruik je in plaats daarvan de wizard voor het aansluiten van een array (**Camera-array aansluiten**): deze leidt je door de selectie van master/slave (vooraf ingevuld door een synchronisatiekabelproef — elke aangesloten camera kan master zijn, en de RGB
-camera `-FRGB` is vooraf geselecteerd wanneer deze is aangesloten), de keuze van de weergavemodus (afzonderlijke versus gecombineerde tegels) en een scène met array-instellingen met een live weergave van de haalbare fps en kabelbandbreedte voordat u de instellingen definitief maakt. De wizard en array-workflows worden behandeld in het hoofdstuk over multi-camera-arrays van deze handleiding; het CLI
-equivalent is de &quot;LATTICE Camera First-Connect Workflow&quot; in de [CLI
-referentie](../reference/cli-reference.md).

## Verbinding maken via de CLI
en SDK
Toegang via de

CLI
en SDK
vereist een betaald Chloros
+-abonnement en dat u bent ingelogd; dit wordt serverzijde afgedwongen (`401 AUTH_REQUIRED` wanneer u niet bent ingelogd, `403 PLAN_UPGRADE_REQUIRED` op het gratis abonnement).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Volledige handtekeningen, opties en vastleggingsworkflows: [CLI
-referentie](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK
-referentie](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Hoe kalibratie wordt afgehandeld bij het verbinden

Elke LATTICE-camera heeft zijn fabriekskalibratiepakket **op de camera** staan, en Chloros
controleert ook de cloud van MAPIR
wanneer de camera verbinding maakt:

| Situatie   | Wat Chloros
gebruikt                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| De**nieuwste kalibratie die voor dat serienummer is gepubliceerd** — de kopie in de cloud heeft voorrang op de kopie op de camera. Een camera die door MAPIR
opnieuw is gekalibreerd of bijgewerkt, werkt zichzelf daarom automatisch bij; er is geen actie van de gebruiker nodig. |
| **Offline**| Het**pakket op de camera**, zoals het is. Volledig offline workflows blijven werken; ze nemen gewoon geen nieuwere kalibraties over totdat de camera eenmaal online is (of in de fabrieksinstellingen is teruggezet).                                                  |

Op het moment van opname worden de daadwerkelijk toegepaste coëfficiënten **vastgelegd in de XMP-metadata van elke afbeelding**. Een latere kalibratie-update wijzigt nooit stilletjes beelden die je al hebt vastgelegd — bij het opnieuw verwerken van een oude opname worden de coëfficiënten gebruikt die in de XMP zijn vastgelegd, niet de meest recente versie van vandaag.

## Probleemoplossing

* **&quot;Geen camera&#x27;s gevonden op het netwerk&quot;**— controleer de link-local-configuratie in [Netwerkconfiguratie](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statisch host-netwerkkaartadres `169.254.x.x/16`, camera&#x27;s op dezelfde link, geen DHCP/gateway verwacht. Gebruik vervolgens**Systeem analyseren**in het verbindingsvenster om te controleren op welke netwerkkaart elke camera wel (of niet) zichtbaar is.**Scannen opnieuw** na elke wijziging in de bekabeling of netwerkkaart.
* **Een opstelling die eerder werkte, wil geen verbinding maken** (array-paneelpoorten met `FRAMES WILL DROP` / `Reduce ROI to enable`) — een update van het netwerkkaartstuurprogramma heeft de instellingen van de ontvangstring ongemerkt gereset. Pas deze opnieuw toe of voer `chloros-cli lattice network --fix` uit vanaf een terminal met beheerdersrechten; zie [Netwerkinstellingen](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Een camera geeft „In Array“ weer** — deze maakt deel uit van een verbonden array-sessie. Verbreek de verbinding met de array om de camera zelfstandig te gebruiken.
