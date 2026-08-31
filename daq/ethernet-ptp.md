# DAQ-E-netwerk en tijdsynchronisatie

> De fysieke netwerkconfiguratie voor de sensor — bekabeling, PoE, IP-toewijzing en de eigen netwerkinstellingen van het apparaat — staat beschreven in de **[DAQ-gebruikershandleiding](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Deze pagina behandelt de Chloros-kant: aansluiting, tijdsynchronisatie en wat te doen als de detectie geen resultaten oplevert.

De DAQ-E is het Ethernet-lid van de DAQ-familie: gevoed via PoE, gedetecteerd via mDNS (service `_daq-e._tcp`) en adresseerbaar via een hostnaam die is afgeleid van de sensor-ID — `daq-e-<6 hex>.local`, bijv. `daq-e-def330.local`. Op deze pagina wordt beschreven hoe het apparaat gegevens over het netwerk verzendt en hoe het deelneemt aan PTP-tijdsynchronisatie.

## Transportmodi

| Modus | Eindpunt | Gebruikers | Opmerkingen |
| --- | --- | --- | --- |
| **Multicast** (standaard) | UDP `239.10.10.10:5002` | Elk apparaat op hetzelfde LAN ontvangt dezelfde stream | Elk datagram wordt gevalideerd met CRC-16/CCITT |
| **Raw** | TCP-poort `5000` | Precies één client (exclusief) | Direct inzetbaar en byte-compatibel met DAQ-U |

Chloros gebruikt standaard multicast, waardoor de GUI, CLI en SDK allemaal tegelijk één sensor kunnen observeren.

## Netwerkvereisten

* **Hetzelfde broadcastdomein.** De machine waarop Chloros draait, moet zich op hetzelfde L2-netwerksegment bevinden als de sensor — mDNS-detectie gaat niet door routers heen.
* **Windows-firewallmelding: accepteer deze.** De eerste keer dat Chloros de multicast-sockets koppelt, vraagt Windows Defender hier eenmalig om toestemming. Als u dit toestaat, worden DAQ-E-gegevens (UDP 5002), mDNS (UDP 5353) en PTP (UDP 319/320) gedekt. Op Linux gebeurt dit zonder melding.
* **PoE-voeding, geen status-LED.** De DAQ-E heeft geen eigen LED — controleer de stroomtoevoer via de link/PoE-indicator op de switch of de injectorpoort, en wacht na het inschakelen enkele seconden totdat het apparaat is opgestart en verbinding heeft gemaakt met het netwerk.

## Verbinding maken

**GUI:** Tabblad Lichtsensoren → Sensor verbinden → Apparaattype &quot;DAQ-E (Ethernet)&quot;. Het zoeken naar sensoren vindt alleen plaats zolang het verbindingsvenster op het scherm staat (mDNS-scan plus een ARP-scan op Windows), en wordt elke 15 seconden herhaald; met de knop Vernieuwen wordt onmiddellijk opnieuw gescand. Gevonden sensoren verschijnen in de vervolgkeuzelijst; de eerste gedetecteerde sensor wordt automatisch geselecteerd.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend actief):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hosts met meerdere netwerkkaarten en de eerste verbinding na het opstarten

Op hosts met meer dan één actieve netwerkinterface kan de **eerste** `pool-connect --eth` na het opstarten leeg zijn, zelfs als de sensor in orde is — de detectieprocedure kan de interface waarop de sensor zich bevindt missen terwijl de ARP-cache nog niet is opgewarmd. De betrouwbare oplossing is om de detectie over te slaan en het adres expliciet door te geven:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accepteert de mDNS-hostnaam of het IP-adres, richt zich altijd op de juiste sensor en is de aanbevolen vorm voor scripts en headless-installaties. Gebruik in de GUI de knop Vernieuwen in het verbindingsvenster en laat een nieuwe scancyclus plaatsvinden.

## Apparaatinstellingen en firmware

De sensor zelf bevat netwerkinstellingen — statisch IP-adres versus DHCP + link-local-adressering, apparaatnaam, automatisch streamen bij het opstarten, OTA-wachtwoord. Deze instellingen aan de apparaatzijde zijn niet beschikbaar als opdrachten in de meegeleverde CLI; ze worden beheerd via de Chloros-GUI waar ze worden weergegeven, of met MAPIR-ondersteuning.

**Firmware-updates zijn ingebouwd in de GUI.**Wanneer een aangesloten DAQ-E firmware draait die ouder is dan de image die bij uw Chloros-build is meegeleverd, verschijnt in de sensorrij een oranje**Update beschikbaar**-pictogram en biedt het instellingenvenster met het tandwielpictogram een<version>

knop</version> „Update naar<version>

„. De update wordt in ongeveer 30 seconden via het netwerk geïnstalleerd; de sensor start opnieuw op en maakt automatisch opnieuw verbinding, en bij een onderbroken overdracht blijft de huidige firmware intact.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-tijdsynchronisatie

DAQ-E-firmware v1.2.0+ neemt deel aan IEEE 1588 PTPv2 als een gewone (uitsluitend slave) klok. **De backend van de Chloros-host is de PTP-grandmaster** — elke DAQ-E en elke LATTICE-camera op het LAN is hieraan ondergeschikt in domein 0, waardoor alle tijdstempels van apparaten binnen een tolerantie van ~1 ms blijven. Dankzij die gedeelde klok kunnen de tijdstempels van DAQ-metingen worden afgestemd op de belichtingstijden van de camera’s (zie [Opname &amp; het .daq-formaat](recording.md)).

Controleer de synchronisatie vanuit de CLI:

| Commando | Weergave |
| --- | --- |
| `chloros-cli time-sync status` | Status van de host-grandmaster, BMCA-prioriteiten, klokidentiteit |
| `chloros-cli time-sync peers` | Alle waargenomen slaves (DAQ-E-sensoren + LATTICE-camera&#x27;s) |
| `chloros-cli time-sync cameras` | PTP-status per camera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Het grandmaster-proces opnieuw starten |

In de GUI toont het instellingenvenster van de DAQ-E een live **PTP Sync**-rij met de huidige PTP-status van de sensor.

Details voor gebruikers die strikte afstemming vereisen:

* Elk gestreamd datagram bevat een vlaggenveld; **bit 2 is ingesteld bij frames waarvan de tijdstempel PTP-gesynchroniseerd is**. Pipelines die een strikte afstemming tussen camera en DAQ vereisen, moeten op dat bit filteren.
* Controleer vóór een gesynchroniseerde opname of de sensor voorkomt in `chloros-cli time-sync peers`. (De interne, directe hardwaretooling van MAPIR kan de opname ook afstemmen op PTP-lock met een `--wait-ptp`-vlag die tot 15 seconden wacht totdat de sensor de SLAVE-status bereikt; die tooling maakt geen deel uit van de geleverde CLI.)
* Terwijl PTP actief als slave fungeert, weigert de sensor handmatige klokimpulsen („PTP levert de klok”). Dat is zo ontworpen — vertrouw op PTP.

## Opmerkingen over Linux

* **PTP heeft `libcap2-bin` nodig tijdens de installatie.** De postinst-taak `.deb` verleent `cap_net_bind_service=+ep` rechten op `/usr/lib/chloros/chloros-backend`, zodat het de PTP-poorten 319/320 kan binden zonder root-rechten. Als `libcap2-bin` ontbreekt, wordt die stap overgeslagen en kan PTP niet worden gestart. Oplossing:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Headless Jetson / Raspberry Pi:** bij de eerste installatie wordt de systemd-unit `chloros-backend.service` gegenereerd, maar niet ingeschakeld. Voor continu actieve PTP (en DAQ-beschikbaarheid) zonder de GUI:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Zonder deze unit draait PTP alleen zolang de Chloros-GUI geopend is.

## Probleemoplossing: „Geen DAQ-E-apparaten gevonden“

| Controle | Detail |
| --- | --- |
| Voeding | Geen LED op de sensor — controleer de PoE- en link-indicatoren van de switch/injectorpoort; wacht enkele seconden na het inschakelen |
| Broadcastdomein | Host en sensor bevinden zich op hetzelfde L2-segment; mDNS wordt niet gerouteerd |
| Windows-firewall | Accepteer de Defender-prompt bij de eerste keer opstarten (UDP 5002, 5353, 319/320) |
| Host met meerdere netwerkkaarten | Bij de eerste detectie na het opstarten wordt de sensor mogelijk niet gevonden — maak verbinding via `--eth-host <ip-or-hostname>` |
| Opnieuw scannen via de GUI | Detectie vindt alleen plaats terwijl het verbindingsvenster open is; gebruik ‘Vernieuwen’ |</version>
