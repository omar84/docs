---
layout: page
title: amp-ad
order: 0
---


NOTE: The specification of `amp-ad` is likely to significantly evolve over time. The current approach is designed to bootstrap the format to be able to show ads.

A container to display an ad.

Ads are loaded like all other resources in AMP documents, with a special
custom element called `<amp-ad>`. No ad network provided JavaScript is allowed to run inside the AMP document. Instead the AMP runtime loads an iframe from a
different origin (via iframe sandbox) as the AMP document and executes the ad
network’s JS inside that iframe sandbox.

<!---
Copyright 2015 The AMP HTML Authors. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS-IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

-->

AMP documents only support ads served via HTTPS.

#### Behavior

The `<amp-ad>` requires width and height values to be specified like all
resources in AMP. It requires a `type` argument that select what ad network is displayed. All `data-*` attributes on the tag are automatically passed as arguments to the code that eventually renders the ad. What `data-` attributes are required for a given type of network depends and must be documented with the ad network.
{% highlight html %}
<amp-ad width=300 height=250
    type="a9"
    data-aax_size="300x250"
    data-aax_pubname="test123"
    data-aax_src="302">
</amp-ad>
{% endhighlight %}

#### Styling

`<amp-ad>` elements may not themselves have or be placed in containers that have CSS `position: fixed` set (with the exception of `amp-lightbox`).
This is due to the UX implications of full page overlay ads. It may be considered to allow similar ad formats in the future inside of AMP controlled containers that maintain certain UX invariants.

#### Attributes

**type**

Identifier for the ad network. This selects the template that is used for the ad tag.

**src**

Optional src value for a script tag loaded for this ad network. This can be used with ad networks that require exactly a single script tag to be inserted in the page. The src value must have a prefix that is whitelisted for this ad network.

**data-foo-bar**

Most ad networks require further configuration. This can be passed to the network using HTML `data-` attributes. The parameter names are subject to standard data attribute dash to camel case conversion. E.g. "data-foo-bar" is send to the ad for configuration as "fooBar".

**json**

Optional attribute to pass configuration to the ad as an arbitrarily complex JSON object. The object is passed to the ad as-is with no mangling done on the names.

#### Placeholder

Optionally `amp-ad` supports a child element with the `placeholder` attribute. If supported by the ad network, this element is shown until the ad is available for viewing.
{% highlight html %}
<amp-ad width=300 height=250
    type="foo">
  <div placeholder>Have a great day!</div>
</amp-ad>
{% endhighlight %}

#### No Ad available

- `amp-ad` supports a child element with the `fallback` attribute. If supported by the ad network, this element is shown if no ad is available for this slot.
{% highlight html %}
<amp-ad width=300 height=250
    type="foo">
  <div fallback>Have a great day!</div>
</amp-ad>
{% endhighlight %}

- If there is no fallback element available, the amp-ad tag will be collapsed (set to display: none) if the ad sends a message that the ad slot cannot be filled and AMP determines that this operation can be performed without affecting the user's scroll position.

#### Supported ad networks

- [A9](https://github.com/ampproject/amphtml/blob/master/builtins/../ads/a9.md)
- [AdReactor](https://github.com/ampproject/amphtml/blob/master/builtins/../ads/adreactor.md)
- [AdSense](https://github.com/ampproject/amphtml/blob/master/builtins/../ads/adsense.md)
- [AdTech](https://github.com/ampproject/amphtml/blob/master/builtins/../ads/adtech.md)
- [Doubleclick](https://github.com/ampproject/amphtml/blob/master/builtins/../ads/doubleclick.md)


#### Running ads from a custom domain

AMP supports loading the bootstrap iframe that is used to load ads from a custom domain such as your own domain.

To enable this, copy the file [remote.html](https://github.com/ampproject/amphtml/blob/master/builtins/../3p/remote.html) to your web server. Next up add the following meta tag to your AMP file(s):

{% highlight html %}
<meta name="amp-3p-iframe-src" content="https://assets.your-domain.com/path/to/remote.html">
{% endhighlight %}

The `content` attribute of the meta tag is the absolute URL to your copy of the remote.html file on your web server. This URL must use a "https" schema. It is not allowed to reside on the same origin as your AMP files. E.g. if you host AMP files on "www.example.com", this URL must not be on "www.example.com" but e.g. "something-else.example.com" is OK. See the doc ["Iframe origin policy"](https://github.com/ampproject/amphtml/blob/master/builtins/../spec/amp-iframe-origin-policy.md) for further details on allowed origins for iframes.

##### Enhance incoming ad configuration

This is completely optional: It is sometimes desired to further process the incoming iframe configuration before drawing the ad using AMP's built-in system.

This is supported by passing a callback to the `draw3p` function call in the [remote.html](https://github.com/ampproject/amphtml/blob/master/builtins/../3p/remote.html) file. The callback receives the incoming configuration as first argument and then receives another callback as second argument (Called `done` in the example below). This callback must be called with the updated config in order for ad rendering to proceed.

Example:

```JS
draw3p(function(config, done) {
  config.targeting = Math.random() > 0.5 ? 'sport' : 'fashion';
  // Don't actually call setTimeout here. This should only serve as an
  // example that is OK to call the done callback asynchronously.
  setTimeout(function() {
    done(config);
  }, 100)
});