# SASS Best Practices

Give color variables generic names

Bad `$tilt-blue; $tilt-almost-black`

Good `$tilt-highlight; $tilt-soft`

# Fonts

Tilt uses HelveticaNeueRoman as its primary font-family.  This is a non-free webfont and is defined in our CSS with a font-face definition. The proper font fallback order is:

`font-family: "HelveticaNeueRoman", Helvetica, Arial, sans-serif;`

We presently use 4 different font-weights for HelveticaNeueRoman, which the design/product team refers to as follows:

* "light" - `font-weight: 300;`
* "normal" or "roman" - `font-weight: 400;`
* "medium" - `font-weight: 500;`
* "bold" - `font-weight: 600;` [1]

[1] note that this is different from the default `font-weight: bold` which would be `font-weight: 700;`

In general you should never need to override the font-family except for unusual circumstances such as styling an iframe.

## More Information About Fonts

We use font-weight and font-face together to avoid the browser choosing a "faux-bold".

"Faux-bold", in addition to being ugly, will [break the display of colored emoji in Chrome](https://code.google.com/p/chromium/issues/detail?id=441946) (as of July 2015).

We do not presently make usage of italics and there is no italic version of HelveticaNeueRoman currently available in the codebase.

See also:

* http://www.smashingmagazine.com/2013/02/setting-weights-and-styles-at-font-face-declaration/
* http://alistapart.com/article/say-no-to-faux-bold