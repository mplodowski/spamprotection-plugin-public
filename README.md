# Spam Protection Plugin

Preventing spam submitted through forms.

When adding a form to a public site, there's a risk that Spam bots will try to submit it with fake values. Luckily, the majority of these bots are pretty dumb. You can thwart most of them by adding an invisible field to your form that should never contain a value when submitted. Such a field is called a honeypot. These Spam bots will just fill all fields, including the honeypot.

When a submission comes in with a filled honeypot field, this plugin will discard that request. On top of that this plugin also checks how long it took to submit the form. This is done using a timestamp in another invisible field. If the form was submitted in a ridiculously short time, the anti Spam will also be triggered.

After installing this plugin, all you need to do is to add the component to your form.

## Features

* Protect all your frontend forms from Spam bots
* Simple `Google reCaptcha` replacement
* GDPR compliance
* Work out of the box

## Why is this a paid plugin?

Something that is free has little or no perceived value. Users do not commit to free products and only use them until
something else looks nice and is free comes along. When I invest my time in the development of a new plugin I commit to
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
with plugin.

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

That's all you need to protect yourself from Spam bots. Everything will work just like before, but when Spam bot fill the form, the form will not be sent.

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
