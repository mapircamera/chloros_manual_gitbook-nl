# DAQ-E-netwerk en tijdsynchronisatie

> De fysieke netwerkconfiguratie voor de sensor — bekabeling, PoE, IP-toewijzing en de eigen netwerkinstellingen van het apparaat — staat beschreven in de **[DAQ-gebruikershandleiding](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Deze pagina behandelt de Chloros-kant: aansluiten, tijdsynchronisatie en wat te doen als de detectie geen resultaten oplevert.

De DAQ-E is het Ethernet-lid van de DAQ-familie: gevoed via PoE, gedetecteerd via mDNS (service `_daq-e._tcp`) en adresseerbaar via een hostnaam die is afgeleid van de sensor-ID — `daq-e-<6 hex>.local`, bijv. `daq-e-def330.local`. Op deze pagina wordt beschreven hoe het apparaat gegevens over het netwerk verzendt en hoe het deelneemt aan PTP-tijdsynchronisatie.

## Transportmodi

| Modus | Eindpunt | Gebruikers | Opmerkingen |
| --- | --- | --- | --- |
| **Multicast** (standaard) | UDP `239.10.10.10:5002` | Elk apparaat op hetzelfde LAN ontvangt dezelfde stream | Elk datagram wordt gevalideerd met CRC-16/CCITT |
| **Raw** | TCP-poort `5000` | Precies één client (exclusief) | Direct inzetbaar en byte-compatibel met DAQ-U |

Chloros maakt standaard gebruik van multicast, waardoor de GUI, CLI en SDK allemaal tegelijkertijd één sensor kunnen volgen.

## Netwerkvereisten

* **Hetzelfde broadcastdomein.** De machine waarop Chloros draait, moet zich op hetzelfde L2-netwerksegment bevinden als de sensor — mDNS-detectie gaat niet door routers heen.
* **Firewall.** Op Windows voegt het installatieprogramma de firewallregel **Chloros Backend (UDP-In)**toe voor de backend, in elk netwerkprofiel, die DAQ-E-gegevens (UDP 5002), mDNS (UDP 5353) en PTP (UDP 319/320), en een**Chloros Backend (TCP-In)**-regel, beperkt tot het lokale subnet, voor de verbinding die een DAQ-E tijdens een firmware-update terug naar de computer maakt. Op Linux opent de `.deb` de UDP-poorten in de `ufw` of `firewalld` wanneer die firewall actief is op het moment van installatie. Bij een firewall van een derde partij, of een firewall die later wordt ingeschakeld, moeten dezelfde poorten worden toegestaan.
* **PoE-voeding, geen status-LED.** De DAQ-E heeft geen eigen LED — controleer de stroomtoevoer via de link/PoE-indicator op de switch of de injectorpoort, en wacht na het inschakelen enkele seconden totdat het apparaat is opgestart en verbinding heeft gemaakt met het netwerk.

## Aansluiten

**GUI:** Tabblad Lichtsensoren → Sensor aansluiten → Apparaattype „DAQ-E (Ethernet)“. Het zoeken naar sensoren vindt alleen plaats zolang het aansluitvenster op het scherm staat (mDNS-scan plus een ARP-scan op Windows), en wordt elke 15 seconden herhaald; met de knop Vernieuwen wordt onmiddellijk opnieuw gescand. Gedetecteerde sensoren verschijnen in de vervolgkeuzelijst; de eerste gedetecteerde sensor wordt automatisch geselecteerd.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend actief):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Een DAQ-E die al in de pool zit, wordt zowel aan de hand van de hostnaam als het IP-adres herkend: als je hem opnieuw verbindt via het andere adres, wordt de bestaande poolvermelding weergegeven in plaats van dat het apparaat een tweede keer wordt geopend.

### Hosts met meerdere netwerkkaarten en de eerste verbinding na het opstarten

Op hosts met meer dan één actieve netwerkinterface kan de **eerste** `pool-connect --eth` na het opstarten leeg zijn, zelfs als de sensor in orde is — de detectieprocedure kan de interface waarop de sensor zich bevindt missen terwijl de ARP-cache nog niet is opgewarmd. De betrouwbare oplossing is om de detectie over te slaan en het adres expliciet door te geven:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accepteert de mDNS-hostnaam of het IP-adres, richt zich altijd op de juiste sensor en is de aanbevolen vorm voor scripts en headless-installaties. Gebruik in de GUI de knop Vernieuwen in het verbindingsvenster en laat een nieuwe scancyclus plaatsvinden.

## Apparaatinstellingen en firmware

De sensor zelf bewaart zijn netwerkinstellingen: DHCP (de fabrieksinstelling, die terugvalt op een 169.254.x.x link-local-adres wanneer geen DHCP-server reageert) of een statisch adres. Ook wordt bijgehouden of de sensor bij het opstarten begint met streamen. Wijzig deze instellingen in de GUI (de tandwielinstellingen van de sensor, **Netwerk**:**Wijzigen…**,**Streamen bij het opstarten**,**Sensor opnieuw opstarten**; zie [Het tabblad DAQ](gui.md)), of via de CLI en SDK met `pool-set-network`, `pool-set-auto-stream` en `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; zie de [CLI-referentie](../reference/cli-reference.md#daq-e-device-settings)). De sensor start opnieuw op om een netwerkwijziging door te voeren, waarna Chloros opnieuw verbinding maakt. Er moet een statisch adres aanwezig zijn op het netwerk van de computer waarop Chloros draait: Chloros weigert een adres dat het na de herstart niet kon bereiken. De ID van de sensor en de hostnaam `daq-e-<id>.local` veranderen nooit. Nieuw in 1.3.0.

**Firmware-updates zijn geïntegreerd in de GUI.** Wanneer een aangesloten DAQ-E firmware draait die ouder is dan de image die bij uw Chloros-build is meegeleverd, verschijnt er in de sensorrij een oranje **Update beschikbaar**-pictogram en biedt het instellingenvenster met het tandwielpictogram een <version>knop</version>

"Bijwerken naar <version>". De update wordt in ongeveer 30 seconden via het netwerk geïnstalleerd; de sensor start opnieuw op en maakt automatisch opnieuw verbinding, en bij een onderbroken overdracht blijft de huidige firmware intact. Voer vanuit de CLI of SDK, en op Linux (dat geen GUI heeft), `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`) uit.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-tijdsynchronisatie

DAQ-E-firmware v1.2.0+ neemt deel aan IEEE 1588 PTPv2 als een gewone (alleen-slave) klok. **De backend van de Chloros-host is de PTP-grandmaster** — elke DAQ-E en elke LATTICE-camera op het LAN is hieraan ondergeschikt in domein 0, waardoor alle tijdstempels van apparaten binnen een tolerantie van ~1 ms blijven. Dankzij die gedeelde klok kunnen de tijdstempels van DAQ-metingen worden gekoppeld aan de belichtingstijden van de camera’s (zie [Opname & het .daq-formaat](recording.md)).

Controleer de synchronisatie via de CLI:

| Commando | Toont |
| --- | --- |
| `chloros-cli time-sync status` | Status van de host-grandmaster, BMCA-prioriteiten, klokidentiteit en `health`: `OK`, of `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` met een `health_detail` die aangeeft waarom zijn pakketten het netwerk niet bereiken |
| `chloros-cli time-sync peers` | Alle waargenomen slaves (DAQ-E-sensoren + LATTICE-camera’s) |
| `chloros-cli time-sync cameras` | PTP-status per camera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Het grandmaster-proces opnieuw starten |

In de GUI toont het instellingenvenster van de DAQ-E een live **PTP Sync**-rij met de huidige PTP-status van de sensor.**Een DAQ-E waarbij PTP is uitgeschakeld, blijft uitgeschakeld.** Het apparaat bewaart de PTP-instelling (aan/uit) in zijn eigen geheugen, dus een apparaat waarvan PTP was uitgeschakeld, blijft uitgeschakeld na het opnieuw aansluiten of het uit- en inschakelen van de stroom, en de tijdstempels worden niet via PTP gesynchroniseerd. `chloros-cli daq pool-list` markeert een dergelijk apparaat (`!! PTP is switched OFF on this unit …`) en geeft de opdracht weer waarmee dit kan worden verholpen:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros zet een apparaat nooit vanzelf weer aan, omdat de uit-stand opzettelijk kan zijn.

Details voor gebruikers die strikte afstemming vereisen:

* Elk gestreamd datagram bevat een vlaggenveld; **bit 2 is ingesteld op frames waarvan de tijdstempel PTP-gesynchroniseerd is**. Pipelines die strikte afstemming van camera’s en DAQ’s vereisen, moeten op dat bit filteren.
* Controleer vóór een gesynchroniseerde opname of de sensor voorkomt in `chloros-cli time-sync peers` en of `chloros-cli daq pool-list` de PTP niet als uitgeschakeld markeert. (De interne, direct op de hardware gerichte tooling van MAPIR kan de opname ook afstemmen op PTP-lock met een `--wait-ptp`-vlag die tot 15 s wacht tot de sensor de SLAVE-status bereikt; die tooling maakt geen deel uit van de meegeleverde CLI.)
* Terwijl PTP actief in de slave-modus is, weigert de sensor handmatige klokimpulsen („PTP levert de klok“). Dat is zo ontworpen — vertrouw op PTP.

## Opmerkingen voor Linux

* **PTP-poorten zonder root.** PTP bindt aan UDP 319/320. De vlag `.deb` schrijft `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), zodat de backend deze poorten bindt terwijl deze onder uw gebruikersaccount draait. Vanaf versie 1.3.0 stelt het pakket geen bestandscapaciteit (`setcap`) in op de backend en is er geen `libcap2-bin` nodig; voeg zelf geen capaciteit toe, want dan start de uitgepakte backend niet op. Als PTP poort 319 niet kan binden, raadpleeg dan [Linux-installatie](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Headless Jetson / Linux-host:** het pakket installeert de backend-services, maar schakelt er geen enkele in. Voor een continu actieve PTP (en DAQ-beschikbaarheid) zonder dat iemand een commando uitvoert, schakel je de service in als je eigen gebruiker:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Zonder dit draait PTP alleen zolang de backend door iets is gestart — een `chloros-cli`-commando of een SDK-script. Zie [Linux-installatie](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) voor de root-account `chloros-backend.service` en hoe u hieruit kunt overschakelen.

## Probleemoplossing: „Geen DAQ-E-apparaten gevonden”

| Controle | Detail |
| --- | --- |
| Stroom | Geen LED op de sensor — controleer de PoE- en link-indicatoren van de switch/injectorpoort; wacht enkele seconden na het inschakelen |
| Broadcastdomein | Host en sensor bevinden zich op hetzelfde L2-segment; mDNS wordt niet gerouteerd |
| Firewall | Windows: de regel **Chloros Backend (UDP-In)** van het installatieprogramma laat de DAQ-E-stream (UDP 5002), mDNS (5353) en PTP (319/320) door; een firewall van een derde partij moet hetzelfde toestaan. Linux: de `.deb` opent deze poorten in `ufw` / `firewalld` alleen als de firewall actief was tijdens de installatie — sta ze zelf toe in een later ingeschakelde firewall |
| Host met meerdere netwerkkaarten | Discovery doorzoekt mDNS met een ARP-fallback; `--eth-host <ip-or-hostname>` slaat discovery over en maakt direct verbinding |
| Opnieuw scannen via de GUI | Discovery wordt alleen uitgevoerd terwijl het verbindingsvenster open is; gebruik ‘Vernieuwen’ |
