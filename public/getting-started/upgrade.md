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

## Release 1.4.0 (backup snapshots, restore versioning, team roles)

Release 1.4.0 ships three feature areas: **server-side backup snapshots with restore** (dry-run diff, merge and destructive replace restores), **per-team custom roles with a permission matrix**, and the **CLI 0.3.0** (local vault unlock + offline OTP for E2EE vaults, opt-in keychain caching, HOTP counter sync). Read the notes below, then apply the upgrade as usual.

### Database migrations (2 new)

`php artisan migrate` (the install wizard does this automatically) runs:

1. `team_users.role` and `team_invitations.role` widen from the 4-value enum to a 50-char string (custom role slugs).
2. A new `team_roles` table is created and the 4 system presets are seeded for **every existing team** — existing teams behave exactly as before the upgrade.

!!!warning Take a DB backup before migrating
The enum-to-string widen runs one `ALTER TABLE` per table. On large `team_users` tables the ALTER takes a metadata lock for its duration — schedule accordingly, and **take a database backup before upgrading** (also required for a clean downgrade, see below).
!!!

### Behavior notes

- **Team authorization is now permission-based.** The preset roles reproduce the previous behavior exactly, including quirks (viewers may still share their *own* accounts). Team owners can add custom roles in _Teams → team → Roles & permissions_. Activity-log **export** stays owner-only.
- **Snapshot restores are safety-gated.** A destructive (replace) restore always requires a dry-run diff first, a single-use confirmation token (10 min TTL), and the typed word `RESTORE`; an automatic pre-restore snapshot is kept. On **multi-node deployments the cache store must be shared between nodes** (`redis`/`database`) — restore tokens live in the cache.
- **The CLI remembers derived keys only on request** (`2fav get <service> --remember`) — keychain entries expire after 24h by default and never store the master password itself.

### Downgrade story (rolling back to 1.3.x)

- The migrations' `down()` **remaps any custom role slug to `member`** before re-narrowing the role columns to the old enum — a rollback never fails or blanks roles, but custom role definitions are lost (they live in the dropped `team_roles` table).
- The snapshot store **survives a downgrade but becomes inert**: the `backup_snapshots` rows and their encrypted files remain on the `snapshots` disk and can be removed later with `php artisan snapshots:prune`.
- Snapshots are encrypted with the application key: a past or future **APP_KEY rotation invalidates historical snapshots** (they report `app_key_rotated` and cannot be restored).
- Team members assigned a custom role fall back to `member` permissions after a downgrade (custom slugs resolve to nothing on 1.3.x, which is the safe direction).

---

## Laravel 13 platform upgrade (release 1.3.3)

Release 1.3.3 is a **platform-only release**: it moves the app from Laravel 12 to Laravel 13 (with WebAuthn library v5 and PHPUnit 13) and ships zero user-facing features. Read the notes below, then apply the upgrade as usual.

### Mandatory after updating: clear every cache

The CSRF middleware class was renamed in this release. If a stale route or config cache survives the update, the app can fail to boot. Run this **before** the install wizard:

```sh
php artisan config:clear && php artisan route:clear && php artisan view:clear && php artisan cache:clear
```

### Config additions (if you maintain a customized config)

- `config/cache.php` gains a `storage` store and a `serializable_classes` allow-list (`Illuminate\Support\Collection` + `Illuminate\Support\LazyCollection`). The allow-list keeps cached settings working under Laravel 13's unserialize gate — do not replace it with `false`.
- `config/session.php` gains `'serialization' => 'json'` (see the session advisory below).
- `config/database.php` refreshes the MySQL SSL option to the PHP 8.4 `Pdo\Mysql` constants and reads the Postgres `sslmode` from the `DB_SSLMODE` env var.

### Session serialization: one-time logout

Session payloads now serialize as JSON (upstream default). Existing PHP-serialized sessions are discarded, so **every user is logged out once** after the upgrade. If a deployment cannot afford that, keep the old behavior by setting `'serialization' => 'php'` in `config/session.php` (not recommended: JSON avoids PHP object-injection gadget chains if `APP_KEY` ever leaks).

### Personal access tokens: expected no-op, verify anyway

Token minting and verification are untouched by the framework bump — existing PATs keep working. The install wizard now also runs `2fauth:fix-passport-key-permissions` (Docker entrypoint runs it on boot as well), which tightens the `storage/oauth-*.key` file permissions for rootless Docker deployments. As a precaution, verify one of your existing PATs after upgrading (e.g. from the browser extension or CLI) and review/rotate tokens in _Settings > OAUTH_ if anything misbehaves.

### WebAuthn library archived upstream

The `laragear/webauthn` package (v5) that powers passwordless login was **archived by its author in May 2026** — it receives no further security fixes. This release is its last stop in 2FA-Vault. A migration to Laravel Passkeys is planned as a follow-up release; WebAuthn login continues to work unchanged in the meantime.

### CSRF rename — no client impact

`App\Http\Middleware\VerifyCsrfToken` was renamed to `App\Http\Middleware\PreventRequestForgery` (upstream parity). Behavior is unchanged for browsers, the browser extension, and the CLI: first-party web forms keep using the CSRF token, `/api/v1` endpoints stay CSRF-free (Bearer authentication), and Laravel 13's new `Sec-Fetch-Site` check only *relaxes* protection (same-origin browser requests may skip the token check).

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
