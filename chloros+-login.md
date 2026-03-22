# Chloros+ Inloggen

## Chloros en Chloros (browser) Inloggen

Via het <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> zijbalkmenu kunt u inloggen op uw Chloros+-account en extra functies ontgrendelen.

Wanneer u bent ingelogd, worden uw accountgegevens weergegeven:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Inloggen

Log in met uw Chloros+ inloggegevens om CLI-verwerking in te schakelen. Op Linux (zonder GUI) is dit de enige manier om uw licentie te activeren.

**Syntaxis:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-gebruikers**: De Python SDK biedt ook een programmatische `logout()`-methode om in de cache opgeslagen inloggegevens te wissen. Zie de [Python SDK documentatie](api-python-sdk.md#logout) voor details.
{% endhint %}

**Voorbeeld:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Speciale tekens**: Gebruik enkele aanhalingstekens rond wachtwoorden die tekens zoals `$`, `!` of spaties bevatten.
{% endhint %}

**Uitvoer:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Opslag van inloggegevens

In de cache opgeslagen inloggegevens worden opgeslagen op een platformspecifieke locatie:

| Platform | Pad naar cache met inloggegevens |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Vervaldatum van het abonnement

De vervaldatum van het abonnement in de GUI geeft aan wanneer uw licentie ongeldig wordt. Voor terugkerende maandelijkse abonnementen is de vervaldatum aan het einde van de maand. Voor jaarabonnementen is dit een jaar nadat u het abonnement bent gestart. De licentiecontrole vereist een maandelijkse internetverbinding om te verifiëren, met een respijtperiode van 30 dagen.

### Apparaatlimiet

Elk Chloros+-abonnement biedt een verschillend aantal geregistreerde apparaten. Elk apparaat waarop u inlogt met een Chloros+-account telt mee voor uw aantal geregistreerde apparaten. U kunt een apparaat hernoemen en verwijderen op de pagina van uw MAPIR Cloud-account.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-abonnement</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOUD</th></tr></thead><tbody><tr><td align="right">Ondersteunde apparaten</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
