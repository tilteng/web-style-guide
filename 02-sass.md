# SASS Best Practices

Give color variables generic names

Bad `$tilt-blue; $tilt-almost-black`

Good `$tilt-highlight; $tilt-soft`

# Fonts

Tilt uses Helvetica Neue as its primary font.  This font is a system font in certain Mac-based operating systems ("Helvetica Neue").  For systems that do not have this font installed, we include it as a web font under the terminology "HelveticaNeueRoman" as defined in our CSS with a font-face definition. The proper font fallback order is:

`font-family: "Helvetica Neue", "HelveticaNeueRoman", Helvetica, Arial, sans-serif;`

The justification for this ordering is:

1. "Helvetica Neue" - if this is installed as a system font we will use this.
2. "HelveticaNeueRoman" - this is defined in our CSS using a `@font-face` declaration.  We declarations for 4 different font-weights.  This will be chosen if "Helvetica Neue" is not available as a system font.
3. Helvetica, Arial, sans-serif - these are fallbacks and displaying any of these fonts to the user represents a bug (or a failure to serve the web font properly).  We include these to handle edge cases in a way that is still marginally "on-brand".

Our webfont supports 4 different font-weights for Helvetica Neue, which the design/product team refers to as follows:

* "light" - `font-weight: 300;`
* "normal" or "roman" - `font-weight: 400;`
* "medium" - `font-weight: 500;`
* "bold" - `font-weight: 600;` [1]

You should only use font-weights from the above.  What this means is:

* Don't use `font-weight: bold` in your CSS ever
* Don't use `<strong>` or `<b>` tags without overriding the `font-weight` as default `<strong>` and `<b>` font-weight will be 700, for which we do not have a font-face definition
* Don't use `font-weight` with a different value from the above (300, 400, 500, 600)
* Only use `font-style: normal` - don't use italics
* Don't use `bold-weight` for anything that could potentially display emoji, such as tilt titles, tilter names, comments, group names, etc. Chrome has a known bug that prevents the display of emoji for fonts-weights above 500.


[1] note that this is different from the default `font-weight: bold` which would be `font-weight: 700;`

In general you should never need to override the font-family except for unusual circumstances such as styling an iframe.

## More Information About Fonts

We use font-weight and font-face together to avoid the browser choosing a "faux-bold".

"Faux-bold", in addition to being ugly, will [break the display of colored emoji in Chrome](https://code.google.com/p/chromium/issues/detail?id=441946) (as of July 2015).

We do not presently make usage of italics and there is no italic version of HelveticaNeueRoman currently available in the codebase.  If you use `em` or `i` tags, or set `font-style: italic`, this will result in the browser applying a "faux-italic" version of the webfont, which may not be what you want - watch out!

Displaying a "faux-bold" or "faux-italic" to users should be considered a bug.  Know your fonts!

See also:

* http://www.smashingmagazine.com/2013/02/setting-weights-and-styles-at-font-face-declaration/
* http://alistapart.com/article/say-no-to-faux-bold
