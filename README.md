# Spam Protection Plugin

Stop spam bots on the forms of your [October CMS](https://octobercms.com) site — a honeypot, content rules, rate
limiting and single-use tokens, with a log of everything that was blocked. No captcha.

Most spam bots fill in every field they find. The plugin adds an invisible field to your form that a person never
sees, a honeypot, and discards any submission where it carries a value. A second invisible field holds an encrypted
timestamp, so a form submitted faster than a human could fill it is rejected as well.

## Features

* Honeypot — an invisible field only bots fill in, plus a timestamp that catches forms submitted faster than a human
  could type; drop the component into a form and it works
* No captcha and no third-party scripts, so no visitor data leaves your server; the spam log (on by default, kept for
  30 days) stores IP addresses and submitted values, see [Personal data](#personal-data)
* Content rules — blocked keywords (substring or whole word), a cap on links, chosen scripts such as Cyrillic,
  Chinese or Arabic, and the machine-generated strings bots type into name and company fields
* Rate limiting per visitor and page, and a single-use form token that stops a harvested form from being replayed
* Spam log — every blocked submission with its reason, address, browser and the submitted fields, sensitive values
  redacted, pruned on a schedule you choose
* Path exclusions for webhooks and endpoints that must never be inspected, and a CSP nonce for sites with a strict
  Content Security Policy
* Works with Form Builder and any form that posts through the CMS, including AJAX handlers
* Separate permissions for the settings, viewing the log and deleting from it
* Multilingual: English, Polish, German, French, Spanish, Brazilian Portuguese, Italian, Russian, Dutch and Czech
  translations included — more available on request

## Requirements

This plugin requires PHP 8.2 or higher and October CMS 4.0 or higher. Running its test suite and static analysis
needs PHP 8.4.

If you are running October CMS 3.x, use the 1.x version of this plugin.

## Why is this a paid plugin?

Something that is free has little or no perceived value. Users do not commit to free products and only use them until
something else that looks nice and is free comes along. When I invest my time in the development of a new plugin I
commit to supporting and maintaining it. I ask my customers to do the same. I do not make money from this plugin by
advertisements, upgrades or additional services like hosting or setup.

Did you know that 30% of your purchase or donation goes to help fund the October Project?

My plugins take many hours to develop (40-120+) and even more hours to document and maintain. My paid plugins have to
pay for both this time, and the time I am spending on free plugins and less successful paid plugins. This means that it
will take even a successful plugin years to become profitable. Please consider buying an extended license if you want me
to continue to maintain these plugins for the very small fee I ask in return or hire me for adding functionality that
you feel is missing but valuable.

## Like this plugin?

If you like this plugin, give this plugin a like or make a donation with [PayPal](https://www.paypal.me/mplodowski).

## My other plugins

Please check my other [plugins](https://octobercms.com/author/Renatio).

## Support

Please use [GitHub Issues Page](https://github.com/mplodowski/spamprotection-plugin-public/issues) to report any issues
with the plugin. Read the
[upgrade guide](https://github.com/mplodowski/spamprotection-plugin-public/blob/main/UPGRADE.md) before updating.

> Reviews should not be used for getting support or reporting bugs, if you need support please use the Plugin support
> link.

Icon made by [Darius Dan](https://www.flaticon.com/authors/darius-dan)
from [www.flaticon.com](https://www.flaticon.com/).

# Documentation

## Usage

Add the `spamProtection` component to the page, layout or partial that renders your form:

```twig
[spamProtection]
==
```

Then render the component anywhere inside the form tag:

```twig
<form>
    {% component 'spamProtection' %}
</form>
```

Several forms on one page can each render the component; every render gets its own element ids and its own single-use
token. Your form handlers need no changes: a submission caught as spam is stopped before the handler runs, see
[What a blocked visitor sees](#what-a-blocked-visitor-sees).

A bot that strips the hidden fields out of the form gets past every check unless you name the form's handlers on the
component, see [Requiring the honeypot on a form](#requiring-the-honeypot-on-a-form).

## Requiring the honeypot on a form

The middleware cannot tell a protected form whose hidden fields were removed from any other request to the page, so
tell the component which handlers belong to the form:

```ini
[spamProtection]
handlers = "onSend, contactForm::onSubmit"
postback = 1
```

* `handlers` lists the AJAX handlers, separated by commas, that must carry the honeypot fields. A name without a
  component prefix also matches that handler on any component, and the case does not matter.
* `postback` rejects every submission to the page that calls no AJAX handler and lacks the fields, for a form that
  posts to `onStart` or names a `_handler`.

A request without the fields to a listed handler is rejected as `missing_fields` before the handler runs; one with
the fields goes through every check as usual. Other forms and handlers on the page are not affected. This works for
a component declared on the page or the layout, or added in another component's `init()`. A component declared in a
partial is set up only while the page renders, after the handlers have run, so there it has no effect.

## What is protected

The plugin runs as a middleware on the CMS controller. It inspects:

* every `POST`, `PUT`, `PATCH` or `DELETE` request to a CMS page, postbacks and AJAX handlers alike, and a `GET` that
  carries a JSON body. A `_method` field or an `X-HTTP-Method-Override` header does not exempt a `POST`;
* and of those, only submissions that carry the component's fields, unless **Require honeypot on all forms** is on.
  All checks — the honeypot, the single-use token, the content rules and the rate limit — follow this rule, so AJAX
  requests from pages without the component are left alone.

It does not protect:

* a submission without the honeypot fields while **Require honeypot on all forms** is off, unless it targets a
  handler named on the component, see [Requiring the honeypot on a form](#requiring-the-honeypot-on-a-form). A bot
  that removes the hidden inputs reaches any other handler with blocked keywords and without a token, as often as it
  likes;
* requests handled outside the CMS controller: plugin routes (`routes.php`), API endpoints and backend forms;
* `GET` forms that only send a query string, such as a search form;
* paths listed under **Excluded paths**.

**Require honeypot on all forms** closes the first gap for the whole site, but it rejects every CMS submission that
lacks the fields, including search, login, cart or logout requests and AJAX handlers on pages without the component.
List the paths of those forms under **Excluded paths** before turning it on. An exclusion covers the whole path, so a
form without the component that shares a page with a protected one has to render the component as well.

## Settings

Go to **Settings → System → Spam Protection**; the page needs the `Manage spam protection settings` permission.

| Tab | Setting | What it does | Default |
|---|---|---|---|
| General | Enable spam protection | Turns every check off when disabled. | on |
| General | Require honeypot on all forms | Rejects every submission without the honeypot fields, see [What is protected](#what-is-protected). | off |
| General | Excluded paths | Paths that skip every check, see [Excluded paths](#excluded-paths). | empty |
| General | Message for blocked visitors | Shown when a content rule, the rate limit or the token rejects a submission. | built-in, per site language |
| Honeypot | Honeypot field name | Name of the invisible field; must not collide with a field your forms use. | `hp_field` |
| Honeypot | Timestamp field name | Name of the encrypted timestamp field; the token field is this name plus `_token`. | `valid_from` (token: `valid_from_token`) |
| Honeypot | Randomize honeypot field name | Appends a random suffix to the honeypot name on every render. | on |
| Honeypot | Check submission speed | Rejects forms submitted faster than the minimum time. | on |
| Honeypot | Minimum time before submitting (seconds) | 1–3600; 2–3 stops most bots without catching people. | `2` |
| Honeypot | Maximum form age (minutes) | 0–2880; rejects older forms, see [Maximum form age](#maximum-form-age); `0` turns it off. Shown while the speed check is on. | `0` |
| Content | Blocked keywords | Rejects submissions containing any of these words or phrases, case-insensitive. | empty |
| Content | Match whole words only | Blocks `sex` without blocking `Essex`. | off |
| Content | Maximum links allowed | 0–100; `0` allows any number. | `0` |
| Content | Blocked scripts | Rejects submissions containing even one letter of a checked script: Cyrillic, Arabic, Armenian, Bengali, Devanagari, Georgian, Greek, Chinese characters, Hangul, Hebrew, hiragana, katakana, Tamil or Thai. Emoji, digits and punctuation always pass; Greek also blocks single letters such as π or Ω. | none |
| Content | Block random strings | Rejects machine-generated words such as `drLTJlJFcwNYHCFgIjpS`. | off |
| Limits | Maximum submissions per visitor | 0–1000 submissions per visitor and page within the window; `0` turns the limit off. | `0` |
| Limits | Rate limit window (seconds) | 1–86400. | `60` |
| Limits | Single-use form token | Every rendered form carries a signed token accepted once, see [Single-use form token](#single-use-form-token). | off |
| Log | Log blocked submissions | Records every blocked submission in the spam log. | on |
| Log | Keep log entries for (days) | 1–365; older entries are pruned daily. | `30` |

Until the settings are saved for the first time, the protection switch, **Require honeypot on all forms**, the field
names, name randomizing and the speed check come from `config/honeypot.php` and its `HONEYPOT_*` variables, everything
else from the plugin's `config/config.php`. Honeypot keys set in a published `config/renatio/spamprotection/config.php`
take precedence, so delete them to use `HONEYPOT_*`. Both files only provide defaults: once the settings are saved, the
stored values win.

### Content rules

Blocked keywords are matched as substrings anywhere in the text, so `sex` also matches `Essex`. With **Match whole
words only** no letter may touch the keyword on either side; this works with accented letters too, so `gęś` no longer
matches `gęślą`. Keywords are entered as tags, so a keyword may contain spaces but not a comma.

A random string is a word of ten or more letters that switches case at least four times and has at least three odd
pieces, meaning unpronounceable fragments such as `Jl` or capitals running into lowercase such as `ABCdef`. Digits
split words, so CamelCase names such as `YouTubeChannel2` pass, and links and e-mail addresses are skipped, so
**Maximum links allowed** is the control for links.

The content rules read every submitted text field, nested ones included, except the honeypot fields, the CSRF token
and fields whose values are machine-generated by design: captcha responses (`g-recaptcha-response`,
`h-captcha-response`, `cf-turnstile-response`, `frc-captcha-*`), anything named `password` and the ad click
identifiers `gclid`, `fbclid` and `msclkid`.

### Rate limit

The rate limit counts every checked submission by client IP address and page, the page regardless of letter case,
percent-encoding or doubled slashes. It counts per URL, so `/pl/contact` and `/contact` each have their own counter,
and so does every address of a page with URL parameters (`/offer/a`, `/offer/b`). An IPv6 address is counted by its
/64 network, since a single subscriber usually owns the whole range, and an IPv4-mapped IPv6 address as the IPv4
address it carries. Parallel submissions are counted under a cache lock. The limit needs a cache store that persists
between requests; with `array` or `null` it never limits anything.

Behind a load balancer or a proxy such as Cloudflare every visitor arrives from the proxy's address and shares one
counter, so the settings page shows a warning whenever the limit is on and the request that opened it carries a
forwarding header without coming from a trusted proxy. Register the proxy in `app/Provider.php` so Laravel reads the
visitor address from the forwarded headers, and only list addresses you control, since a trusted header can be forged
by anyone who can reach the server directly:

```php
<?php namespace App;

use Illuminate\Http\Request;
use System\Classes\AppBase;

class Provider extends AppBase
{
    public function boot()
    {
        parent::boot();

        Request::setTrustedProxies(
            ['10.0.0.0/8', '173.245.48.0/20'],
            Request::HEADER_X_FORWARDED_FOR | Request::HEADER_X_FORWARDED_PROTO
        );
    }
}
```

`173.245.48.0/20` is one of Cloudflare's ranges; the full list is at https://www.cloudflare.com/ips/. The plugin does
not read `CF-Connecting-IP` or similar headers itself, because a bot that bypasses the CDN could set them to any
address and dodge the limit.

### Maximum form age

The speed check only rejects a form sent too early, so a bot that harvested one form can resubmit it forever. With
**Maximum form age** set, a form whose timestamp is older than that many minutes is rejected as `form_expired`. The age
counts from the moment the form became valid, that is the page load plus the minimum time, and it needs no cache, so it
also works where the single-use token cannot.

A person who leaves the tab open longer is caught too, so an expired form is not dropped silently: the visitor is asked
to reload the page and send it again, in the site's language, even when **Message for blocked visitors** is set. The
typed data stays in the form as described under [What a blocked visitor sees](#what-a-blocked-visitor-sees). Choose a
limit well above the time a visitor needs to fill in your longest form; a few hours is a sensible start. A form that
posts several times from one render, such as a multi-step form, has to be finished within the limit, since
`onRefreshSpamToken` renews only the token.

The timestamp is written into the page when it is rendered, so leave the limit off on pages served from a full-page
cache or a CDN: once the cached copy is older than the limit, every visitor gets an expired form. With the single-use
token on, a form older than 24 hours fails on the token first, so a limit above 1440 minutes adds nothing there.

### Excluded paths

Paths listed under **Excluded paths** skip every check, which is the way to leave a form, a webhook or an endpoint
unprotected, for example `webhooks/*` or `checkout/payment`. Add one path per row, without the domain or query string;
the leading `/` is optional and `/` alone is the home page. Wildcards such as `api/*` are supported, but `*` also spans
`/` and `api/*` does not cover `api` itself. Paths match regardless of case, and on a site with a route prefix such as
`/pl`, `contact` covers `/pl/contact` too. The middleware runs before a page is resolved, so exclusions are matched
against the request path rather than set on the page itself.

## Single-use form token

With **Single-use form token** on, every rendered form carries a signed token in the `valid_from_token` field that is
accepted once and expires after 24 hours, so a bot that harvested one form cannot keep resubmitting it. Like the other
checks it only guards submissions that carry the honeypot fields.

The token is signed with a key kept in the visitor's session, so it is rejected as `token_invalid` when posted
without the session cookie it was issued to: a bot cannot collect tokens in one session and spend them in another.
Signing in keeps it valid, but it ends with the session (`SESSION_LIFETIME`, 120 minutes by default) or on signing
out, so a form left open longer asks the visitor to reload the page. A bot that keeps its cookies can still fetch a
fresh token for every post; that costs it a page load per submission, and the rate limit caps how many it sends.

The token is spent once the submission reaches the page, unless a validation error (a `422` or an AJAX response
listing invalid fields) or a spam check turns it away, so the form stays usable then. Any other failure, such as
`Flash::error()`, `ajax()->error()` or an exception, spends it, because the handler may already have acted. The
plugin cannot tell a validation error thrown by the handler from one thrown afterwards, so a partial or component that
throws one after the handler has run, for example in a partial rendered through `X-AJAX-PARTIALS`, keeps the token
although the handler acted; validate in the handler, not in those partials.

The token needs:

* a cache store that persists between requests and is shared by every web server: `redis`, `database`, `memcached`,
  or `file` on a single server only, where it keeps one file per accepted submission for up to 24 hours. With
  `array`, `null`, `session` or a `failover` store that falls back to one of them the settings page refuses the
  switch; should the store change later, the token is ignored and a warning is logged once an hour;
* an `APP_KEY`, which signs the token; without it the token is ignored the same way;
* the session cookie on the submission, so a form embedded in an iframe on another site, or rendered outside a
  web request, cannot use it;
* pages that are rendered per visitor. Leave the token off on pages served from a full-page cache or a CDN, and do
  not render the component inside a cached partial, since every visitor would receive the same token.

### Refreshing the single-use token

A form that posts several times from one render, such as a multi-step or "send another" form, or one that lets the
visitor retry after an error other than a validation error, has to ask the component for a fresh token between posts.
The `onRefreshSpamToken` handler returns the field name and a new token (or `null` while the setting is off). Call it
with `jax.ajax`, not from the form: only an AJAX request that names the handler and sends no form data or partials
skips the checks. A request carrying the form fields is checked like any submission and fails on the spent token.

```twig
<form data-request="onSubmit" data-request-success="refreshSpamToken(this)">
    {% component 'spamProtection' %}
</form>

<script>
function refreshSpamToken(form) {
    jax.ajax('spamProtection::onRefreshSpamToken', {
        success: function (data) {
            var field = form.querySelector('[name="' + data.spamTokenField + '"]');
            if (field && data.spamToken) {
                field.value = data.spamToken;
            }
        }
    });
}
</script>
```

## Content Security Policy

The honeypot fields are hidden with CSS. Under a Content Security Policy without `'unsafe-inline'` for styles, the
browser drops the inline style and the fields become visible to visitors. Hand the component the nonce your site adds
to the current response and it renders the hiding rule as a `<style nonce="...">` block instead, which the policy
allows. The nonce comes from whatever sets your CSP header, such as a middleware or a CSP package; the plugin does not
create one.

Component properties only accept `{{ name }}` references inside a partial, so either pass the nonce as a partial
variable:

```twig
{% partial 'contact-form' cspNonce=nonce %}
```

```twig
[spamProtection]
nonce = "{{ cspNonce }}"
==
<form>
    {% component 'spamProtection' %}
</form>
```

or set the property from the page life cycle:

```php
function onInit()
{
    $this->spamProtection->setProperty('nonce', $nonce);
}
```

Leave the property empty when no policy is in place.

## What a blocked visitor sees

A submission caught by the honeypot or the speed check gets an empty page, since only bots trip those. These two run
first, so a bot that also sends no token or is over the rate limit still gets the empty page and is logged under the
honeypot reason. Every other rule (blocked keywords, links, scripts, random strings, the maximum form age, the rate
limit and the single-use token) can catch a real person, so the visitor is told the message could not be sent, or for
an expired form or a token from an ended session that the page has to be reloaded:

* an AJAX form gets an error response, so its success handlers do not run and the typed data stays in place. October
  shows the message as a flash with `data-request-flash`, otherwise as an alert;
* a regular form is redirected back with the message in the flash bag and the typed input kept as old input. Render
  `{% flash %}` in your layout to show it. Old input only refills fields the theme renders with `form_value()` or the
  `Form` helpers.

In both cases the form handler does not run. The message is set under **Message for blocked visitors**; when left
empty a built-in one is shown in the language of the site the form was posted to. A custom message is one text for
all sites, so leave it empty on a multilingual installation.

## Spam log

Every blocked submission is recorded under **Settings → Logs → Spam Log** with the reason it was blocked. A content
rule also records what it matched: the keyword as listed in the settings, the link count against the limit, the
blocked script or the random string, together with the field, which the entry preview highlights. A visitor
blocked again for the same reason is logged once a minute, whether or not the rate limit is on, and a visitor over the
rate limit once per its window; the visitor is identified by IP address, or its /64 network for IPv6, and page, as the
rate limit does. Behind a load balancer or CDN configure trusted proxies (see [Rate limit](#rate-limit)), or all
visitors share one entry per reason and page a minute. A failure to write the log is reported to the application log
and never affects the response.

### Reasons

| Reason | Meaning |
|---|---|
| `honeypot_filled` | The hidden honeypot field carried a value. |
| `submitted_too_fast` | The timestamp field was sent before it became valid. |
| `invalid_timestamp` | The timestamp field was tampered with or could not be decrypted. |
| `form_expired` | The timestamp is older than **Maximum form age**. |
| `missing_fields` | The honeypot field or the timestamp field is missing, or the form carried neither while **Require honeypot on all forms** is on or its handler is named on the component. |
| `blocked_keyword` | A blocked keyword was found in the submitted text. |
| `too_many_links` | More links than **Maximum links allowed**. |
| `cyrillic_content` | Cyrillic found while it is checked under **Blocked scripts**. |
| `blocked_script` | Another script checked under **Blocked scripts** found; the match names the script. |
| `random_string` | A machine-generated word found while **Block random strings** is on. |
| `token_missing` | The form carried no single-use token while **Single-use form token** is on. |
| `token_reused` | The single-use token was already spent or is being used by another submission. |
| `token_invalid` | The single-use token was forged, tampered with or is older than 24 hours. |
| `rate_limited` | The visitor exceeded **Maximum submissions per visitor** on that page. |

### Scheduler

Entries older than **Keep log entries for (days)** are removed by a daily scheduled task, so the October scheduler
must be running: https://docs.octobercms.com/4.x/setup/scheduler.html

```bash
* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
```

The **Log** tab of the settings warns when the log still holds entries more than a day past the retention period.

## Permissions

| Permission | Name | Grants |
|---|---|---|
| `renatio.spamprotection.access_settings` | Manage spam protection settings | The **Settings → System → Spam Protection** page. |
| `renatio.spamprotection.access_logs` | View the spam log | The **Settings → Logs → Spam Log** list and entry preview. |
| `renatio.spamprotection.delete_logs` | Delete entries and empty the spam log | Deleting selected entries and the **Empty log** button; needs `access_logs` to reach the list. |

Before 2.2.0 viewing the log also allowed deleting from it. The 2.2.0 migration grants `delete_logs` to every role and
administrator that held `access_logs`; revoke it where viewing should be enough. Super users hold every permission.

## Personal data

For every logged submission the spam log stores:

* the page address, with sensitive query string values redacted, and the request method;
* the visitor's IP address and user agent;
* the reason it was blocked, what the content rule matched and in which field, and the time;
* the submitted fields. Nested fields are listed under their form names, such as `contact[email]`.

Values that look sensitive (passwords, tokens, API keys, one-time codes, card and identity numbers) are replaced with
`[redacted]` before anything is written, and a rule matching in such a field records only the field name. The CSRF
token is dropped entirely. A secret carried in the path of the address, such as `/reset-password/{token}`, is stored as
it is. Values longer than 255 characters and field names longer than 64 are cut, and an entry keeps at most 100 fields
and 10,000 characters, noting how many fields were left out under `[truncated]`.

The log is on by default and entries are kept for 30 days unless you change it. A real person caught by a rule by
mistake is personal data you hold, so mention the log in your privacy policy, keep the retention window as short as
your review process allows, restrict the `View the spam log` permission, or turn logging off. Use **Empty log** on the
log page to delete everything at once.

Outside the log, the rate limit keeps only a hash of the visitor's address and the page in the cache, for the length
of its window, and the single-use token keeps only hashes of the tokens it has accepted. Nothing is sent to third
parties.

## Events

Every rejection fires an event, whether or not logging is on or the entry is throttled, so another plugin can send an
alert or collect statistics:

| Event | Fired when | Payload |
|---|---|---|
| `Spatie\Honeypot\Events\SpamDetectedEvent` | the honeypot, the speed check or missing fields reject a submission | `$request` |
| `Renatio\SpamProtection\Classes\ContentSpamDetectedEvent` | a content rule, the maximum form age, the rate limit or the single-use token rejects a submission | `$request`, `$reason` — one of the [reasons](#reasons), `$rejection` — a `Rejection` with the `detail` a content rule matched and the `field` path it matched in |

`ContentSpamDetectedEvent` extends `SpamDetectedEvent`, but Laravel dispatches by the exact class, so listen to both
to catch every rejection. Keep listeners cheap and non-throwing; an exception aborts the request.

```php
use Illuminate\Support\Facades\Event;
use Renatio\SpamProtection\Classes\ContentSpamDetectedEvent;

Event::listen(ContentSpamDetectedEvent::class, function (ContentSpamDetectedEvent $event) {
    traceLog('Spam blocked: ' . $event->reason . ' from ' . $event->request->ip());
});
```

## Using with Form Builder

The [Form Builder](https://octobercms.com/plugin/renatio-formbuilder) plugin adds the component to every form it
renders, so no theme changes are needed. Both can be combined with reCAPTCHA: the honeypot discards bots before the
form is even validated, and reCAPTCHA covers what slips through.

## Using with an existing form component

To protect the form of your own component, add the Spam Protection component in its `init()` method:

```php
use Renatio\SpamProtection\Components\SpamProtection;

public function init()
{
    $this->addComponent(SpamProtection::class, 'spamProtection', [
        'handlers' => $this->alias . '::onSubmit',
    ]);
}
```

Then render it inside the form in your component's view:

```twig
<form>
    {% component 'spamProtection' %}
</form>
```
