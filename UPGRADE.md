# Upgrade guide

Versions not listed here need no action. Back up the database before upgrading.

## Upgrading To 2.0.1

Plugin requires October CMS 4.0 or higher. On October CMS 3, stay on 1.x.

## Upgrading To 2.1.0

Plugin requires PHP 8.2 or higher. Run `php artisan october:migrate` and make sure the scheduler runs: the new spam log
is on by default, stores IP addresses and is pruned daily. Settings now live under **Settings → System → Spam
Protection**; before 2.2.0 they ignored `HONEYPOT_*` variables and a published `config/honeypot.php`, so check the
values there. Grant the new `Manage spam protection settings` and `View the spam log` permissions where needed.

## Upgrading To 2.2.0

Run `php artisan october:migrate`.

- Deleting entries and emptying the spam log need the new `Delete entries and empty the spam log` permission; the
  migration grants it to everyone who could view the log, so revoke it where viewing should be enough.
- Blocked visitors get a message instead of an empty page, so render `{% flash %}` in the layout.
- List the handlers of each protected form in the component's `handlers` property, or turn on `postback`; otherwise a
  submission without the honeypot fields skips every check. See the README.
- The single-use token is bound to the session, so forms rendered before the upgrade ask the visitor to reload. A form
  that posts more than once fetches a fresh token with `onRefreshSpamToken`. The token is ignored on a `session` store
  and on a `failover` store that falls back to a non-persistent one.
- The default honeypot name is `hp_field` and the minimum time 2 seconds, also with `HONEYPOT_SECONDS=1`. Save the old
  values in the settings to keep them, and clear cached pages.
- Until the settings are first saved, `HONEYPOT_*` and `config/honeypot.php` apply; delete honeypot keys from a
  published `config/renatio/spamprotection/config.php` to use them. A `HONEYPOT_NAME` that is not a valid field name
  blocks the first backend save until corrected.
- `with_csp` is replaced by the component's `nonce` property.
- Copy a theme override of the component partial again: element ids now use `{{ honeypotId }}`, and the component's
  variables exist only while it renders.
- **Block Cyrillic characters** is replaced by **Blocked scripts**; a switch that was on shows up as Cyrillic checked.
  Code reading `block_cyrillic` should call `Settings::blockedScripts()`.
- The middleware calls `ContentInspector::rejection()`, so a subclass overriding only `inspect()` is no longer used.
  `collectText()`, `matchesBlockedKeyword()`, `exceedsLinkLimit()`, `containsBlockedCyrillic()` and
  `containsBlockedRandomString()` are gone; move overrides to their successors, which take the submitted fields and
  return a `Rejection` (`findBlockedScript()` replaces `containsBlockedCyrillic()`).
