# Chloros+ Inloggen

## Inloggen via de GUI

Via het zijbalkmenu van de gebruiker <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> kun je inloggen op je Chloros+-account en extra functies ontgrendelen.

**Je hoeft slechts één keer per computer in te loggen.** De GUI, CLI en Python SDK delen dezelfde gecachete sessie — als u zich aanmeldt via de desktop-GUI, worden ook de CLI en SDK op die computer geactiveerd (en omgekeerd via `chloros-cli login`).

Wanneer u bent ingelogd, worden uw accountgegevens weergegeven:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Abonnementsniveaus

| Abonnement | `plan_id` | Type |
| --- | --- | --- |
| Iron | `0` | Gratis |
| Copper | `1` | Betaald (Chloros+) |
| Bronze | `2` | Betaald (Chloros+) |
| Silver | `3` | Betaald (Chloros+) |
| Goud | `4` | Betaald (Chloros+) |

Zie [abonnementen en prijzen](https://cloud.mapir.camera/pricing) voor wat elk betaald abonnement omvat.

### Toegang tot CLI/SDK vereist een betaald abonnement

Toegang tot CLI en Python SDK vereist **een willekeurig betaald Chloros+-abonnement (Copper of hoger)**. Dit wordt**server-side** afgedwongen — elk CLI/SDK-verzoek moet zowel een actieve sessie als een betaald abonnement bevatten:

| HTTP-status | `error_code` | Betekenis | Oplossing |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Niet aangemeld op deze machine | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Ingelogd, maar het abonnement is te laag (gratis Iron-niveau) | Upgrade naar een betaald Chloros+-abonnement |

`chloros-cli status` blijft bereikbaar in het gratis abonnement, zodat je altijd je huidige abonnement kunt zien en waarom de toegang wordt geweigerd.

### Limieten voor aangesloten hardware per abonnement

Elk abonnement heeft een limiet voor het aantal LATTICE-camera’s en DAQ-lichtsensoren dat tegelijkertijd live kan worden aangesloten:

| Abonnement | LATTICE-camera’s | DAQ-lichtsensoren |
| --- | --- | --- |
| Iron (gratis / niet ingelogd) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI-aanmelding

Meld je aan met je Chloros+-inloggegevens om CLI-verwerking in te schakelen. Op Linux (zonder GUI) is dit de enige manier om je licentie te activeren.

**Syntaxis:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-gebruikers**: De Python SDK biedt ook een programmatische `logout()`-methode om in de cache opgeslagen inloggegevens te wissen. Zie de [SDK-referentie](reference/sdk-reference.md) voor meer informatie.
{% endhint %}

**Voorbeeld:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Speciale tekens**: Gebruik enkele aanhalingstekens rond wachtwoorden die tekens bevatten zoals `$`, `!` of spaties.
{% endhint %}

**Uitvoer:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Opslag van inloggegevens

In de cache opgeslagen inloggegevens en configuratie worden op **alle platforms** opgeslagen in de map `.chloros` van je gebruikershomedirectory:

| Platform | Pad naar cache met inloggegevens |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Als u een nieuwere versie over een bestaande versie installeert, blijven uw inloggegevens behouden, zowel op Windows (het installatieprogramma) als op Linux (de map `.deb`, inclusief `chloros-cli update`). Als u Chloros verwijdert — of het Linux-pakket verwijdert — wordt de in de cache opgeslagen aanmelding gewist, zodat u zich na een nieuwe installatie opnieuw moet aanmelden.

### Vervaldatum van het abonnement en offline respijtperiode

De vervaldatum van het abonnement in de GUI geeft aan wanneer uw licentie ongeldig wordt. Bij terugkerende maandabonnementen is de vervaldatum aan het einde van de maand; bij jaarabonnementen is dit een jaar nadat u het abonnement bent begonnen.

Chloros valideert uw licentie online, maar offline werken wordt ondersteund binnen een respijtperiode:

* Succesvolle servervalidaties worden **5 minuten** in de cache opgeslagen, dus bij normaal gebruik zijn er maar heel weinig licentieaanvragen nodig.
* Een ondertekende, aan de machine gebonden licentiecache dekt langere offline periodes: **30 dagen voor maandabonnementen**en**tot de vervaldatum van je abonnement (maximaal 365 dagen) voor jaarabonnementen**.
* Wanneer de respijtperiode verstrijkt, wordt het abonnement omgezet naar het gratis Iron-niveau totdat de computer één keer verbinding kan maken met de licentieserver; de toegang wordt hervat bij de volgende succesvolle controle.

### Apparaatlimiet

Elk Chloros+-abonnement biedt een verschillend aantal geregistreerde apparaten. Elk apparaat waarop u inlogt met een Chloros+-account telt mee voor uw aantal geregistreerde apparaten. U kunt een apparaat hernoemen en verwijderen op uw MAPIR Cloud-accountpagina.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros-abonnement</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOUD</th></tr></thead><tbody><tr><td align="right">Ondersteunde apparaten</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Het exacte aantal toegestane apparaten voor uw account wordt weergegeven op uw MAPIR Cloud-accountpagina. Als u zich afmeldt op een apparaat, wordt de slot voor dat apparaat vrijgegeven. Een apparaat dat al is geregistreerd, kan altijd weer inloggen, zelfs als het account de limiet voor het aantal apparaten heeft bereikt.
