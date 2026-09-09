# Upgrade guide

Versions not listed here need no action. Back up the database before upgrading.

## Upgrading To 2.0.1

Plugin requires October CMS 4.0 or higher.

## Upgrading To 2.1.0

Plugin requires PHP 8.2 or higher. Run `php artisan october:migrate` to create the spam log table.

Settings moved from `config/config.php` to **Settings → System → Spam Protection**; the config file now only seeds
the defaults. Blocked submissions are recorded under **Settings → Logs → Spam Log** and pruned daily, so make sure
the scheduler runs. Two new permissions, `Manage spam protection settings` and `View blocked spam log`, gate the
pages. Content rules, rate limiting, path exclusions and the single-use form token are off by default; the token
needs a cache store that persists between requests.
