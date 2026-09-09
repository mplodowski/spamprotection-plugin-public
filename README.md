# Spam Protection Plugin

Stop spam bots on every form of your [October CMS](https://octobercms.com) site — a honeypot, content
rules, rate limiting and single-use tokens, with a log of everything that was blocked. No captcha.

When adding a form to a public site, there's a risk that spam bots will try to submit it with fake values. Luckily, the majority of these bots are pretty dumb. You can thwart most of them by adding an invisible field to your form that should never contain a value when submitted. Such a field is called a honeypot. These spam bots will just fill all fields, including the honeypot.

When a submission comes in with a filled honeypot field, this plugin will discard that request. On top of that this plugin also checks how long it took to submit the form. This is done using a timestamp in another invisible field. If the form was submitted in a ridiculously short time, the anti-spam will also be triggered.

After installing this plugin, all you need to do is to add the component to your form.

## Features

* Honeypot — an invisible field only bots fill in, plus a timestamp that catches forms submitted faster than a human
  could type; drop the component into a form and it works
* No captcha, nothing for visitors to solve, no third-party scripts and no personal data sent anywhere, so it is GDPR
  friendly out of the box
* Content rules — blocked keywords (substring or whole word), a cap on links, Cyrillic script and the machine-generated
  strings bots type into name and company fields
* Rate limiting per visitor and page, and a single-use form token that stops a harvested form from being replayed
* Spam log — every blocked submission with its reason, address, browser and the submitted fields, sensitive values
  redacted, pruned on a schedule you choose
* Path exclusions for webhooks and endpoints that must never be inspected, and a CSP nonce for sites with a strict
  Content Security Policy
* Works with Form Builder and any form that posts through the CMS, including AJAX handlers
* Multilingual: English, Polish, German, French, Spanish, Brazilian Portuguese, Italian, Russian, Dutch and Czech
  translations included — more available on request

## Requirements

This plugin requires PHP 8.2 or higher and October CMS 4.0 or higher. Running its test suite and static analysis
needs PHP 8.4.

## Why is this a paid plugin?

Something that is free has little or no perceived value. Users do not commit to free products and only use them until
something else that looks nice and is free comes along. When I invest my time in the development of a new plugin I commit to
supporting and maintaining it. I ask my customers to do the same. I do not make money from this plugin by
advertisements, upgrades or additional services like hosting or setup.

Did you know that 30% of your purchase or donation goes to help fund the October Project?

My plugins take many hours to develop (40-120+) and even more hours to document and maintain. My paid plugins have to
pay for both this time, and the time I am spending on free plugins and less successful paid plugins. This means that it
will take even a successful plugin years to become profitable. Please consider buying an extended license if you want me
to continue to maintain these plugins for the very small fee I ask in return or hire me for adding functionality that
you feel is missing but valuable.

## Like this plugin?

If you like this plugin, give this plugin a Like or Make donation with [PayPal](https://www.paypal.me/mplodowski).

## My other plugins

Please check my other [plugins](https://octobercms.com/author/Renatio).

## Support

Please use [GitHub Issues Page](https://github.com/mplodowski/spamprotection-plugin-public/issues) to report any issues
with plugin. Read the [upgrade guide](https://github.com/mplodowski/spamprotection-plugin-public/blob/main/UPGRADE.md)
before updating.

> Reviews should not be used for getting support or reporting bugs, if you need support please use the Plugin support
> link.

Icon made by [Darius Dan](https://www.flaticon.com/authors/darius-dan)
from [www.flaticon.com](https://www.flaticon.com/).

# Documentation

## Usage

After installing this plugin, all you need to do is to add the component to your form.

Add `spamProtection` component on frontend page or layout.

```
[spamProtection]
==
```

Then just place component tag anywhere inside your form tag.

```
<form>
    {% component 'spamProtection' %}
</form>
```

That's all you need to protect yourself from spam bots. Everything will work just like before, but when a spam bot fills the form, the form will not be sent.

## Settings

Go to `Settings -> System -> Spam Protection` to configure the plugin.

* **General** — turn protection on or off, and decide whether submissions without
  honeypot fields count as spam. Only enable the latter once every frontend form
  renders the component.
* **Fields** — the honeypot and timestamp field names. They must not collide with
  fields your forms actually use. Randomizing the honeypot name appends a random
  suffix on every render, which makes the field harder for bots to learn.
* **Timing** — reject forms submitted faster than a human could fill them, and how
  many seconds that means. The **single-use form token** goes further: every
  rendered form carries a signed token that is accepted once and expires after
  24 hours, so a bot that harvested one form cannot keep resubmitting it. Like
  the other checks it guards submissions that carry the honeypot fields. The
  token is only spent when the submission gets through, so a validation error or
  a honeypot rejection leaves the form usable. It needs a cache store that
  persists between requests and is shared by every web server (`redis`,
  `database`, `memcached`; `file` only on a single server); with `array` or
  `null` the settings page refuses the switch, and should the store change later
  the token is ignored and a warning is logged. Leave it off on pages served from a full-page
  cache or a CDN, where every visitor would receive the same token. A form that
  posts several times from one render, such as a multi-step or "send another"
  form, has to [refresh the token](#refreshing-the-single-use-token) between
  posts, since only the first one gets through.
* **Content** — discard submissions containing blocked keywords, more links than
  you allow, Cyrillic script, or a random string such as `drLTJlJFcwNYHCFgIjpS`
  (a word of ten or more letters that switches case at least three times and contains adjacent capitals or digits), and cap how many submissions one visitor may send
  in a time window. Every rule is off by default.
* **Log** — whether blocked submissions are recorded, and for how long.

Paths listed under **Excluded paths** skip every check, which is the way to leave a
form or an endpoint unprotected. Add one path per row; wildcards such as `api/*`
are supported. The
honeypot middleware runs before a page is resolved, so exclusions are matched
against the request path rather than set on the page itself.

## Refreshing the single-use token

A rendered form carries one token and the token is spent by the first submission
that gets through. When the form stays on the page and posts again, ask the
component for a fresh token first. The `onRefreshSpamToken` handler returns the
field name and a new token (or `null` while the setting is off). Call it with
`jax.ajax`, not from the form: an AJAX request that names the handler runs nothing
but that handler, so it is let through without any check even with the spent
token still in the form.

```html
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

Call it the same way when a step of a multi-step form has been accepted. A
submission that fails validation keeps its token, so nothing needs refreshing then.

## Content Security Policy

If your site sends a Content Security Policy, hand the component the nonce of the
current response and it renders the hiding rule as a `<style nonce="...">` block
instead of an inline style attribute, which a nonce cannot whitelist.

Component properties only accept `{{ name }}` references inside a partial, so
either pass the nonce as a partial variable:

```
{% partial 'contact-form' cspNonce=nonce %}
```

```
[spamProtection]
nonce = "{{ cspNonce }}"
==
<form>
    {% component 'spamProtection' %}
</form>
```

or set the property from the page life cycle, using whatever produces the nonce on your site:

```
function onInit()
{
    $this->spamProtection->setProperty('nonce', $nonce);
}
```

Leave the property empty when no policy is in place.

## How the checks are applied

The middleware inspects every `POST` request handled by the CMS, including AJAX
handlers. All checks, the honeypot as well as the content rules and the rate
limit, only run on submissions that carry the component's fields, unless
**Require honeypot on all forms** is on. AJAX requests from pages without the
component are therefore left alone, and a bot that drops the fields entirely is
only caught once that setting is on.

The rate limit counts by client IP address. Behind a load balancer or a proxy such
as Cloudflare every visitor arrives from the proxy's address and shares one
counter, so the settings page shows a warning whenever the limit is on and the
request that opened it carries a forwarding header without coming from a
trusted proxy. Register the proxy in
`app/Provider.php` so Laravel reads the visitor address from the forwarded
headers, and only list addresses you control, since a trusted header can be
forged by anyone who can reach the server directly:

```php
use Illuminate\Http\Request;

public function boot()
{
    parent::boot();

    Request::setTrustedProxies(
        ['10.0.0.0/8', '173.245.48.0/20'],
        Request::HEADER_X_FORWARDED_FOR | Request::HEADER_X_FORWARDED_PROTO
    );
}
```

The plugin does not read `CF-Connecting-IP` or similar headers itself, because a
bot that bypasses the CDN could set them to any address and dodge the limit.

Blocked keywords are matched as substrings anywhere in the text, so `sex` also
matches `Essex`. Turn on **Match whole words only** to require that no letter
touches the keyword on either side; this works with accented letters too, so
`gęś` no longer matches `gęślą`. Keywords are entered as tags, so a keyword may
contain spaces but not a comma.

The content rules read every submitted text field, nested ones included, except
the honeypot fields, the CSRF token and the fields whose values are
machine-generated by design: captcha responses (`g-recaptcha-response`,
`h-captcha-response`, `cf-turnstile-response`, `frc-captcha-*`), anything named
`password` and the ad click identifiers `gclid`, `fbclid` and `msclkid`.

Requests handled outside the CMS controller, such as plugin routes, are not covered.

## Spam log

Every blocked submission is recorded under `Settings -> Logs -> Spam Log`, together
with the reason it was blocked, the originating IP address and the submitted
fields. Values that look sensitive (passwords, tokens, card numbers) are replaced
with `[redacted]` before anything is written, and the CSRF token is dropped
entirely. Values longer than 255 characters are cut. A failure to write the log
is reported to the application log and never affects the response.

Entries older than the configured retention window are removed by the daily
schedule, so make sure the October scheduler is running:

```
* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
```

### Reasons

Reason | Meaning
------ | -------
`honeypot_filled` | The hidden honeypot field carried a value.
`submitted_too_fast` | The timestamp field was sent before it became valid, or it was tampered with.
`missing_fields` | The form carried no honeypot fields while **Require honeypot on all forms** is on.
`blocked_keyword` | A blocked keyword was found in the submitted text.
`too_many_links` | More links than **Maximum links allowed**.
`cyrillic_content` | Cyrillic script found while **Block Cyrillic characters** is on.
`random_string` | A word switching letter case repeatedly, with adjacent capitals or digits, while **Block random strings** is on.
`token_missing` | The form carried no single-use token while **Single-use form token** is on.
`token_reused` | The single-use token was already spent or is older than 24 hours.
`rate_limited` | The visitor exceeded **Maximum submissions per visitor** on that page.

### Personal data

The log stores the visitor's IP address, user agent and the submitted fields, which
may contain personal data of a real person caught by a rule by mistake. Keep the
retention window as short as your review process allows, restrict the
`View blocked spam log` permission, or turn logging off entirely. Use
**Empty log** on the log page to delete everything at once.

## Excluding an endpoint

Every `POST` handled by the CMS goes through the middleware, including your own
AJAX handlers and pages that receive webhooks. When such an endpoint must not be
inspected, add its path under **Excluded paths**, for example `webhooks/*` or
`checkout/payment`.

## Using with Form Builder

The [Form Builder](https://octobercms.com/plugin/renatio-formbuilder) plugin adds
the component to every form it renders, so no theme changes are needed. Both can
be combined with reCAPTCHA: the honeypot discards bots before the form is even
validated, and reCAPTCHA covers what slips through.

## Using with existing form component

When you have your custom component form and want to add Spam protection component all you need to do is add it in your component `init()` function.

```
use Renatio\SpamProtection\Components\SpamProtection;

public function init()
{
    $this->addComponent(SpamProtection::class, 'spamProtection', []);
}
```

Then add it to your view component:

```
<form>
    {% component 'spamProtection' %}
</form>
```
