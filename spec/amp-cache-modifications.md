# AMP cache modifications best practices

These are guidelines for what AMP cache implementations should look like. Some items are required for overall security of the platform while others are suggestions for performance improvements. All modifications are made to both AMP and AMP4ADS documents except where noted.

### HTML Sanitization

The AMP Cache parses and re-serializes all documents to remove any ambiguities in parsing the document which might result in subtly different parses in different browsers.

#### All HTML comments are stripped

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<foo><!-- comment --></foo>` | `<foo></foo>` |

</details>

#### Tag and attribute names are lowercased

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<P DATA-FOO=BAR>` | `<p data-foo=BAR>` |

</details>

#### Attribute values are consistently quoted and escaped

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<p data-foo=’< >’>` | `<p data-foo="&lt; &gt">` |

</details>

#### All tags are closed, except for HTML5 void elements

[Void elements](https://www.w3.org/TR/html5/syntax.html#void-elements)  are tags that have no end tag and also no contents. All other tags are closed.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<foo><bar></foo>`<br>`<br/>` | `<foo><bar></bar></foo>`<br>`<br>` |

</details>

#### Whitespace inside tags is stripped

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<p    data-foo=bar    >` | `<p data-foo=bar>` |

</details>

#### Text is escaped

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `3 < 4` | `3 &lt; 4` |

</details>

#### Encoded text characters are simplified, using UTF-8 equivalent characters

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `&nbsp;` | `"\u00A0"` |
| `&#x61;` | `a` |
| `&#00000000000039;` | ` &#39;` |

</details>

#### Move elements after `<body>`, which are only allowed in `<body>`, into the `<body>`. This includes text.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<body></body><div>foo</div>text` | `<body><div>foo</div>text</body>` |

</details>


### URL Rewrites

The AMP Cache rewrites URLs found in the AMP HTML for two purposes. One is to rebase relative URLs found in the document so that the URL remains the same when loaded from the AMP Cache. The other reason is to improve performance by selecting a different equivalent resource. This includes rewriting image and font URLs to use a cached copy and rewriting AMP javascript URLs to use a copy with longer cache lifetimes.

#### All relative `href` , `src` and `data-iframe-src` URLs are rewritten as absolute URLs

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<a href=foo.html target=_top>Lorem ipsum</a>` | `<a href=https://example.com/foo.html target=_top>Lorem ipsum</a>` |
| `<amp-list src="list.json" ...>...</amp-list>` | `<amp-list src="https://example.com/list.json" ...>...</amp-list>` |
| `<amp-install-serviceworker data-iframe-src="sw.html"...></amp-install-serviceworker>` | `<amp-install-serviceworker data-iframe-src="https://example.com/sw.html"...></amp-install-serviceworker>` |

</details>

#### All relative `form[action]` and `form[action-xhr]` URLs are rewritten as absolute URLs

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<form action=/subscribe>...</form>` | `<form action=https://example.com/subscribe>...</form>` |

</details>

#### All relative `@font-face` CSS URLs are rewritten as absolute URLs

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `@font-face {`<br>`font-family: Foo;`<br>`src: url(font.woff);`<br>`}` | `@font-face {`<br>`font-family: Foo;`<br>`src: url(https://example.com/font.woff);`<br>`}` |

</details>

#### All image URLs are rewritten as AMP cache URLs except those in `amp-mustache` `template`

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<amp-img src=https://example.com/foo.png></amp-img>` | `<amp-img src=/i/s/foo.png></amp-img>` |
| `<amp-img srcset="https://example.com/bar.png 1080w, https://example.com/bar-400.png 400w">`| `<amp-img src="/i/s/bar.png 1080w, /i/s/bar-400.png 400w">` |
| `<amp-anim src=foo.gif></amp-anim>` | `<amp-anim src=/i/s/foo.gif></amp-anim>` |
| `<amp-video poster=bar.png>` | `<amp-video poster=/i/s/bar.png>` |

</details>

#### Anchor tags must have a target of `_blank` or `_top`

*Condition*:
If `<a>` tag does not have attribute `target=_blank` or `target=_top` then add `target=_top`. All other `target` values are rewritten to `_top`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<a href=https://example.com/foo.html>Lorem ipsum</a>` | `<a href=https://example.com/foo.html target=_top>Lorem ipsum</a>` |
| `<a href=https://example.com/bar.html target=_blank>Lorem ipsum</a>` | `<a href=https://example.com/bar.html target=_blank>Lorem ipsum</a>` |
| `<a href=https://example.com/baz.html target=window>Lorem ipsum</a>` | `<a href=https://example.com/baz.html target=_top>Lorem ipsum</a>` |

</details>

### Insert and Rewrite Tags

#### Insert `<link rel=icon>`

When a given AMP document does not have a favicon present, insert one. Inserted tag is of the form `<link href={document_protocol}://{document_domain}/favicon.ico rel=icon>`.

*Condition*:
No `<link>` tag present with attribute `rel` equal to any of the following: `icon`, `icon shortcut`, `shortcut icon`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<link href=https://example.com/favicon.ico rel=icon>`<br>`</head>` |
| `<head>`<br>`...`<br>`<link href=https://example.com/favicon.ico rel="icon shortcut">`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<link href=https://example.com/favicon.ico rel="icon shortcut">`<br>`...`<br>`</head>` |

</details>

#### Rewrite `<link rel=manifest>` to `<link rel=origin-manifest>`

*Condition*:
`<link rel=manifest>` tag present in the document.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<link rel=manifest>`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<link rel=origin-manifest>`<br>`...`<br>`</head>` |

</details>

#### Insert `<meta content=always name=referrer>` [required]

If the document was fetched from HTTP origins and does not have a meta referrer tag then insert one.

*Condition*:
No `<meta name=referrer ...>` tag present and document was fetched from HTTP and not HTTPS.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<meta content=always name=referrer>`<br>`</head>` |

</details>

#### Insert `<meta content=noindex name=robots>` [required]

AMP Cache pages should not show up in search result pages. The cache also uses [`robots.txt`](https://cdn.ampproject.org/robots.txt) to enforce this.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<meta content=noindex name=robots>`<br>`</head>` |

</details>


### Remove Tags and Attributes

#### Remove `<link>` resource hints

The AMP Cache removes any resource hints in the original document.

*Condition*
Any `<link>` tag present with attribute `rel` equal to any of the following:
 - `dns-prefetch`
 - `preconnect`
 - `prefetch`
 - `preload`
 - `prerender`

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<link rel=dns-prefetch href=https://example.com>`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`</head>` |

</details>

#### Remove non-whitelisted `<meta>` tags

*Condition*:
Remove any `<meta>` tags except for those that:
 - do not have attributes `content`, `itemprop`, `name` and `property`
 - have attribute `http-equiv`
 - have attribute `name` with prefix `amp-`
 - have attribute `name` with prefix `dc.`
 - have attribute `name` with prefix `i-amp-` [temporary, will be removed at a future date]
 - have attribute `name` with prefix `i-amphtml-`
 - have attribute `name` with prefix `twitter:`
 - have attribute `name=apple-itunes-app`
 - have attribute `name=copyright`
 - have attribute `name=referrer` [note: this may be inserted by AMP Cache]
 - have attribute `name=robots` [note: this is inserted by AMP Cache]
 - have attribute `name=viewport`
 - have attribute `property` with prefix "al:"
 - have attribute `property` with prefix "fb:"
 - have attribute `property` with prefix "og:"

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<meta charset=utf-8>`<br>`<meta http-equiv=content-language content=en>`<br>`<meta name=description content="An example AMP page">`<br>`<meta name=twitter:title content="AMP Example">` | `<meta charset=utf-8>`<br>`<meta http-equiv=content-language content=en>`<br>`<meta name=twitter:title content="AMP Example">` |

</details>

#### Remove `amp-live-list` children based on `amp_latest_update_time` parameter

This is discussed in detail at [Server side filtering for `amp-live-list`](https://github.com/ampproject/amphtml/blob/master/extensions/amp-live-list/amp-live-list-server-side-filtering.md)

#### Remove `amp-access-hide` sections when `amp-access` JSON is `"type": "server"`

If the document is using `amp-access` type of `server` then the AMP Cache removes any section with the attribute `amp-access-hide` from the document.

*Condition*:
`<script id=amp-access type=application/json>...</script>` contains `"type": "server"`

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<script id=amp-access type=application/json>`<br>`{`<br>`...`<br>`"type": "server"`<br>`}`<br>`</script>`<br>`...`<br>`<div amp-access-hide ...>`<br>`Content`<br>`</div>` | `<script id=amp-access type=application/json>`<br>`{`<br>`...`<br>`"type": "server"`<br>`}`<br>`</script>`<br>`...` |

</details>

#### Remove attribute `nonce`

*Condition*:
Remove `nonce` from every tag except for those that are only inserted by the AMP cache `<meta content=NONCE name=i-amphtml-access-state>` and `<meta content=NONCE name=i-amp-access-state>`

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<script async custom-element=amp-youtube nonce=cryptohash src=https://cdn.ampproject.org/v0/amp-youtube-0.1.js></script>` | `<script async custom-element=amp-youtube src=https://cdn.ampproject.org/v0/amp-youtube-0.1.js></script>` |

</details>


### Optimizations

These are modifications that either reduce the byte size of the document or decreases the time to render. An AMP cache is not required to implement these.

#### The AMP engine javascript URL is rewritten to most recent stable version

If possible, rewrite to use the stable version. Otherwise use the unversioned path. The stable version takes the form `<script async src=https://cdn.ampproject.org/rtv/{version}/v0.js></script>`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<script async src=https://cdn.ampproject.org/v0.js></script>` | `<script async src=https://cdn.ampproject.org/rtv/031485231782273/v0.js></script>` |

</details>

#### Insert `<link href=https://fonts.gstatic.com rel="dns-prefetch preconnect">`
The AMP Cache adds prefetch hint tags for browsers to assist in loading resources earlier and thus speed up page loads.

*Condition*:
Has a stylesheet of the form: `<link href=https://fonts.googleapis.com/... rel=stylesheet>`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<link href=https://fonts.googleapis.com/css?family=Lato rel=stylesheet>`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<link href=https://fonts.googleapis.com/css?family=Lato rel=stylesheet>`<br>`<link href=https://fonts.gstatic.com rel="dns-prefetch preconnect">`<br>`...`<br>`</head>` |

</details>

#### Prioritize AMP engine javascript and other render blocking scripts in `<head>`

The AMP Cache places the [AMP engine javascript](https://cdn.ampproject.org/v0.js) as the second child of `<head>` right after `<meta charset=utf-8>`. It then emits any other render blocking custom-element script tags followed by the remaining custom-element `<script>` tags in the document. Render blocking custom-element `<script>` tags are listed in [SERVICES at render-delaying-services.js](https://github.com/ampproject/amphtml/blob/master/src/render-delaying-services.js#L28).

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async src=https://cdn.ampproject.org/v0.js></script>`<br>`<meta charset=utf-8>`<br>`...`<br>`</head>` | `<head>`<br>`<meta charset=utf-8>`<br>`<script async src=https://cdn.ampproject.org/v0.js></script>`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`...`<br>`</head>` |

</details>

#### Remove duplicate custom-element extensions in `<head>`

If a custom-element `<script>` tag is included more than once, the AMP Cache removes all but one.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`...`<br>`</head>` |

</details>

#### Remove unused custom-element extensions in `<head>` [WIP]

This is currently a work in progress.

If a custom-element `<script>` tag is included in `<head>` but not used in `<body>` then remove it. There are several exceptions to this listed under *Condition*.

*Condition*:
Remove unused custom-element extensions with the following exceptions:
 - Do not remove any custom-element extensions if `<amp-live-list>` is present within the document
 - Do not remove any of the following custom-element extensions:
  - amp-access
  - amp-access-laterpay
  - amp-analytics
  - amp-auto-ads
  - amp-dynamic-css-classes
  - amp-form
  - amp-share-tracking

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-analytics src=https://cdn.ampproject.org/v0/amp-analytics-0.1.js></script>`<br>`<script async custom-element=amp-youtube src=https://cdn.ampproject.org/v0/amp-youtube-0.1.js></script>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`<amp-youtube ...></amp-youtube>`<br>`...`<br>`</body>` | `<head>`<br>`...`<br>`<script async custom-element=amp-analytics src=https://cdn.ampproject.org/v0/amp-analytics-0.1.js></script>`<br>`<script async custom-element=amp-youtube src=https://cdn.ampproject.org/v0/amp-youtube-0.1.js></script>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`<amp-youtube ...></amp-youtube>`<br>`...`<br>`</body>` |

</details>

#### Remove `<script type=application/ld+json>...</script>`

Remove JSON-based linked data from the document.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<script async src=https://cdn.ampproject.org/v0.js></script>`<br>`<script type=application/ld+json>`<br>`{`<br>`"@context": "http://schema.org",`<br>`"@type": "Person",`<br>`"name": "Lorem Ipsum",`<br>`}`<br>`</script>`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<script async src=https://cdn.ampproject.org/v0.js></script>`<br>`...`<br>`</head>` |

</details>

#### Remove insignificant whitespace in `<head>`

Remove whitespace in `<head>` except for tags that should preserve whitespace.

*Condition*:
Remove whitespace except for within these tags:
 - `<script>`
 - `<style>`

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<meta charset=utf-8>`<br>`<style amp-custom>`<br>`body {`<br>`background-color: white;`<br>`}`<br>`</style>`<br>`...`<br>`</head>` | `<head>...<meta charset=utf-8><style amp-custom>`<br>`body {`<br>`background-color: white;`<br>`}`<br>`</style>...</head>` |

</details>

#### Remove unnecessary attribute value quotes in entire document [WIP]

This is currently a work in progress for AMP documents and implemented for AMP4ADS documents.

Remove quotes from around an attribute’s value unless the attribute’s value has an ASCII character in the set { 0x20(space), 0x22("), 0x27('), 0x3E(>), 0x60(\`) }.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<meta charset="utf-8">`<br>`<script async src="https://cdn.ampproject.org/v0.js"></script>`<br>`<link rel="icon shortcut" href="https://example.com/favicon.ico">`<br>`...`<br>`</head>` | `<head>`<br>`...`<br>`<meta charset=utf-8>`<br>`<script async src=https://cdn.ampproject.org/v0.js></script>`<br>`<link rel="icon shortcut" href=https://example.com/favicon.ico>`<br>`...`<br>`</head>` |

</details>


### Additional Modifications for AMP for Ads (AMP4ADS) Documents

These are AMP4ADS specific modifications and not implemented for AMP documents.

#### Prioritize AMP4ADS engine javascript in `<head>`

The AMP Cache places the AMP4ADS engine javascript as the second child of `<head>` right after `<meta charset=utf-8>`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async src=https://cdn.ampproject.org/amp4ads-v0.js></script>`<br>`<meta charset=utf-8>`<br>`...`<br>`</head>` | `<head>`<br>`<meta charset=utf-8>`<br>`<script async src=https://cdn.ampproject.org/amp4ads-v0.js></script>`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`...`<br>`</head>` |

</details>

#### Record UTF-16 offsets for AMP4ADS engine and custom-element extensions in added JSON

AMP4ADS requires providing the start and end position of the block of `<script>`
tags that represent the AMP4ADS engine and custom-element extensions. These
string offsets are in UTF-16 encoding lengths. This data is provided in the
`amp-ad-metadata` JSON as `ampRuntimeUtf16CharOffsets`. The `amp-ad-metadata`
JSON is appended to the end of the `<body>`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`<meta charset=utf-8>`<br>`<script async src=https://cdn.ampproject.org/amp4ads-v0.js></script>`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`</body>` | `<head>`<br>`<meta charset=utf-8>`<br>`<script async src=https://cdn.ampproject.org/amp4ads-v0.js></script>`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`<script type=application/json amp-ad-metadata>`<br>`{`<br>`"ampRuntimeUtf16CharOffsets" : [ 55, 337 ]`<br>`}`<br>`</script>`<br>`</body>` |

</details>

#### Record UTF-16 offsets for `amp-access` and `amp-analytics` JSON in added JSON

AMP4ADS requires providing the start and end position of `amp-access` and
`amp-analytics` JSON. These string offsets are in UTF-16 encoding lengths. This
data is provided in the `amp-ad-metadata` JSON as `jsonUtf16CharOffsets`. The
`amp-ad-metadata` JSON is appended to the end of the `<body>`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<script id=amp-access type=application/json>`<br>`{`<br>`...`<br>`}`<br>`</script>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`<amp-analytics>`<br>`<script type=application/json>`<br>`{`<br>`...`<br>`}`<br>`</script>`<br>`</amp-analytics>`<br>`...`<br>`</body>` | `<head>`<br>`...`<br>`<script id=amp-access type=application/json>`<br>`{`<br>`...`<br>`}`<br>`</script>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`<amp-analytics>`<br>`<script type=application/json>`<br>`{`<br>`...`<br>`}`<br>`</script>`<br>`</amp-analytics>`<br>`...`<br>`<script type=application/json amp-ad-metadata>`<br>`{`<br>`"jsonUtf16CharOffsets" : {`<br>`"amp-access": [ 12, 92 ],`<br>`"amp-analytics": [ 105, 175],`<br>`}`<br>`}`<br>`</script>`<br>`</body>` |

</details>

#### Record UTF-16 offsets for CSS `body` selectors in added JSON

AMP4ADS requires providing the start and end position of CSS `body` selectors in
`<style amp-custom>`. These string offsets are in UTF-16 encoding lengths.  This
data is provided in the `amp-ad-metadata` JSON as `cssReplacementRanges`. The
`amp-ad-metadata` JSON is appended to the end of the `<body>`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<style amp-custom>`<br>`body { background-color: #eee; }`<br>`...`<br>`@media screen and (min-width:480px) {`<br>`body { color: #ccc; }`<br>`}`<br>`</style>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`</body>` | `<head>`<br>`...`<br>`<style amp-custom>`<br>`body { background-color: #eee; }`<br>`...`<br>`@media screen and (min-width:480px) {`<br>`body { color: #ccc; }`<br>`}`<br>`</style>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`<script type=application/json amp-ad-metadata>`<br>`{`<br>`"cssReplacementRanges" : [`<br>`[ 59, 63 ],`<br>`[ 139, 143 ],`<br>`}`<br>`</script>`<br>`</body>` |

</details>


#### Record custom-element extension in an added JSON `<script>`

AMP4ADS requires providing additional metadata of what custom-element extensions
are included in the AMP4ADS document. This data is provided in the
`amp-ad-metadata` JSON as `customElementExtensions` and `extensions`. The
`amp-ad-metadata` JSON is appended to the end of the `<body>`.

`customElementExtensions` is just a list of the custom-element extension names
in the document. This is now deprecated but still used.

`extensions` is a list of objects containing `custom-element` and `src`
attributes, which are the name of the custom-element and the location of the
extension respectively.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`</body>` | `<head>`<br>`...`<br>`<script async custom-element=amp-accordion src=https://cdn.ampproject.org/v0/amp-accordion-0.1.js></script>`<br>`<script async custom-element=amp-instagram src=https://cdn.ampproject.org/v0/amp-instagram-0.1.js></script>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`<script type=application/json amp-ad-metadata>`<br>`{`<br>`"customElementExtensions" :`<br>`[ "amp-accordion", "amp-instagram" ],`<br>`"extensions" : [`<br>`{`<br>`"custom-element" : "amp-accordion",`<br>`"src" : "https://cdn.ampproject.org/v0/amp-accordion-0.1.js"`<br>`},`<br>`{`<br>`"custom-element" : "amp-instagram",`<br>`"src" : "https://cdn.ampproject.org/v0/amp-instagram-0.1.js"`<br>`}]`<br>`}`<br>`</script>`<br>`</body>` |

</details>

#### Record custom fonts in an added JSON `<script>`

AMP4ADS requires providing additional metadata of what custom fonts (`href` and
if present `media`) were included in the AMP4ADS document. This data is provided
in the `amp-ad-metadata` JSON as `customStylesheets`. The `amp-ad-metadata` JSON
is appended to the end of the `<body>`.

<details>
<summary>example</summary>

| before | after |
| --- | --- |
| `<head>`<br>`...`<br>`<link href=https://fonts.googleapis.com/css?family=Foo rel=stylesheet>`<br>`<link href=https://fonts.googleapis.com/css?family=Bar rel=stylesheet media=print>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`</body>` | `<head>`<br>`...`<br>`<link href=https://fonts.googleapis.com/css?family=Foo rel=stylesheet>`<br>`<link href=https://fonts.googleapis.com/css?family=Bar rel=stylesheet media=print>`<br>`...`<br>`</head>`<br>`<body>`<br>`...`<br>`<script type=application/json amp-ad-metadata>`<br>`{`<br>`"customStylesheets" : [`<br>`{`<br>`"href" : "https://fonts.googleapis.com/css?family=Foo"`<br>`},`<br>`{`<br>`"href" : "https://fonts.googleapis.com/css?family=Bar",`<br>`"media" : "print"`<br>`}`<br>`}`<br>`</script>`<br>`</body>` |

</details>