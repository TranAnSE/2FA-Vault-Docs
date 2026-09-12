# Upgrade

## Self-hosted server

Update the source code in `/var/www/2FA-Vault` (see [Get your 2FA-Vault copy](/getting-started/installation/self-hosted-server/#get-your-2FA-Vault-copy))

!!!warning
Do not change the `/var/www/2FA-Vault/storage` directory nor your `/var/www/2FA-Vault/database/database.sqlite` file (when using SQLite)
!!!

Depending on how you update your files, you could have to set the permissions again:

```sh
chown -R www-data:www-data /var/www/2FA-Vault
chmod -R 775 /var/www/2FA-Vault
```

Update the dependencies by running:

```sh
composer install

# or if you didn't add composer to your system PATH
php composer.phar install
```

Then start the installation wizard by running:

```sh
php artisan 2FA-Vault:install
```

The wizard automatically runs the following commands:

```sh
php artisan cache:clear
php artisan config:clear
php artisan migrate
php artisan passport:install
php artisan config:cache
php artisan route:cache
```

---

## Docker

!!!warning When using SQLite
At the very least, backup your `database.sqlite` file to avoid bad surprises!
!!!

The Docker image [!badge 2FA-Vault/2FA-Vault] is built on every commit pushed to the `master` branch.  
You can therefore pull the image with `docker pull 2FA-Vault/2FA-Vault` and restart the container to update it.

You can also use tagged images, see [Docker Hub tags](https://hub.docker.com/r/2FA-Vault/2FA-Vault/tags?page=1&ordering=last_updated), which are produced on Github releases.

---

## YunoHost

1. Open the __:icon-sync: System update__ manager from the Yunohost Admin
2. In the __:icon-package: Applications__ section, click the 2FA-Vault [!button variant="success" icon="plus" corners="round" text="Upgrade" size="s"] button
3. Wait for the installer to complete its job

You can also upgrade using the YunoHost command-line:

```bash
yunohost app upgrade 2FA-Vault
```

---

## Upstream v8 sync

This release syncs 2FA-Vault with upstream 2FAuth v8. Read the notes below before upgrading, then apply the upgrade as usual — the install wizard runs `php artisan migrate` for you.

### Database migration (Passport 13)

The upgrade ships a migration that adapts the `oauth_clients` table to Laravel Passport 13: it backfills the new `grant_types` column and drops the legacy columns and the `oauth_personal_access_clients` table.

- Run `php artisan migrate` as part of the upgrade (the install wizard does this automatically).
- Existing **Personal Access Tokens keep working** — client IDs are preserved. As a precaution, try one of your existing PATs after upgrading (e.g. from the browser extension or CLI) and review/rotate tokens in _Settings > OAUTH_ if anything misbehaves.

### Behavior changes

- **Email notifications no longer break requests.** When mail is not configured or a message cannot be sent, the app now logs a warning (with a hint about the `MAIL_*` variables) instead of failing the request. The admin test email still surfaces errors on purpose — use it to validate your [mail configuration](/getting-started/config/env-vars/#email-setting).
- **Administrators are subject to SSO-only restrictions.** The implicit policy bypass for admins was removed: when _Use SSO only_ is enabled, the PAT and WebAuthn restrictions now apply to administrator accounts as well. The admin panel itself is unchanged.
- **Shared account secrets are policy-gated in the API.** The `withSecret` query parameter on `twofaccounts` endpoints only returns secret fields to users holding the _read secret_ permission (the account owner). Team members using shared accounts no longer receive secret material.
- **Extension tip:** the browser extension (1.3.2) now strips a trailing slash from the server URL at setup. Older versions require the URL without a trailing slash.

### New features

- **Group chips** — groups can be pinned as chips on the main accounts view (per-group _Show in chips_ flag), with the [USE_GROUP_CHIPS](/getting-started/config/user-preferences/#use_group_chips) and [SHOW_VIRTUAL_CHIPS](/getting-started/config/user-preferences/#show_virtual_chips) user preferences to control the feature.
- **PAT expiry display** — the token list in _Settings > OAUTH_ now shows each token's expiration date (new tokens expire after 90 days).
