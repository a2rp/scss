# 013 — Colors and the Sass Color Module: Examples

Most modern color-module examples in this file require Dart Sass 1.79.0 or later. Browser support for modern CSS color syntax is separate from Sass compiler support. Add fallback declarations when the project needs them.

## 1. Define colors in several CSS formats

Sass recognizes standard color names, hex values, and CSS color functions.

~~~scss
$named: royalblue;
$short-hex: #36c;
$hex-with-alpha: #3366cc99;
$rgb: rgb(51 102 204);
$hsl: hsl(220deg 60% 50%);
$oklch: oklch(52% 0.18 260deg);
$p3: color(display-p3 0.1 0.3 0.9);

.palette-sample {
  color: $named;
  background-color: $short-hex;
  border-color: $rgb;
}
~~~

Sass treats these as colors, not strings.

## 2. Keep a fallback before modern color syntax

A browser that does not understand the later declaration keeps the earlier supported value.

~~~scss
.button {
  background-color: #3159a5;
  background-color: oklch(48% 0.15 260deg);
}
~~~

Compiled CSS:

~~~css
.button {
  background-color: #3159a5;
  background-color: oklch(48% 0.15 260deg);
}
~~~

The later declaration takes effect in browsers that support OKLCH.

## 3. Compare color alpha with element opacity

Alpha affects a color. The opacity property affects the whole rendered element and its children.

~~~scss
.overlay {
  background-color: rgb(0 0 0 / 55%);
}

.disabled-control {
  opacity: 0.55;
}
~~~

Use an alpha color when only a background or border should be translucent. Use opacity when the entire element should be translucent.

## 4. Inspect a color's space and channels

color.space() identifies a color's space. color.channel() reads a channel. Channel names are quoted; space names are unquoted.

~~~scss
@use "sass:color";

$brand: #2457c5;

@debug color.space($brand); // rgb
@debug color.channel($brand, "red", $space: rgb);
@debug color.channel($brand, "hue", $space: oklch);
@debug color.channel($brand, "lightness", $space: oklch);
~~~

The channel values are printed by the compiler. Hue is an angle and lightness is a percentage.

## 5. Convert a color to another space

to-space() changes how the color is represented.

~~~scss
@use "sass:color";

$brand: #2457c5;
$brand-in-oklch: color.to-space($brand, oklch);
$brand-in-p3: color.to-space($brand, display-p3);

@debug color.space($brand); // rgb
@debug color.space($brand-in-oklch); // oklch
@debug color.space($brand-in-p3); // display-p3
~~~

Conversion does not necessarily change the perceived color.

## 6. Apply a fixed adjustment with color.adjust()

adjust() adds or subtracts a fixed amount from a channel.

~~~scss
@use "sass:color";

$brand: #2457c5;
$fixed-lighter: color.adjust($brand, $lightness: 8%, $space: oklch);
$rotated-hue: color.adjust($brand, $hue: 12deg, $space: oklch);
$less-transparent: color.adjust($brand, $alpha: -10%, $space: oklch);

.preview {
  color: $fixed-lighter;
  border-color: $rotated-hue;
  background-color: $less-transparent;
}
~~~

Choose this when the same fixed channel change is intended.

## 7. Scale channels proportionally with color.scale()

scale() moves a percentage of the remaining distance toward a channel limit.

~~~scss
@use "sass:color";

$brand: #2457c5;
$relative-lighter: color.scale($brand, $lightness: 18%, $space: oklch);
$relative-darker: color.scale($brand, $lightness: -18%, $space: oklch);
$less-vivid: color.scale($brand, $chroma: -30%, $space: oklch);

.preview {
  color: $relative-lighter;
  background-color: $relative-darker;
  border-color: $less-vivid;
}
~~~

scale() percentages range from -100% to 100%. Positive values move toward the channel maximum and negative values move toward its minimum.

## 8. Set an exact channel with color.change()

change() sets a channel to the specified value.

~~~scss
@use "sass:color";

$brand: #2457c5;
$fixed-lightness: color.change($brand, $lightness: 55%, $space: oklch);

@debug color.channel($fixed-lightness, "lightness", $space: oklch);
~~~

Use this when the exact channel value matters. Inspect the result if the supplied channel may be outside the expected range.

## 9. Mix two colors in a chosen space

The method controls the color space used for interpolation.

~~~scss
@use "sass:color";

$blue: #2457c5;
$coral: #e65a3b;

$rgb-mix: color.mix($blue, $coral, $weight: 35%, $method: rgb);
$oklch-mix: color.mix($blue, $coral, $weight: 35%, $method: oklch);

.rgb-result {
  border-color: $rgb-mix;
}

.oklch-result {
  border-color: $oklch-mix;
}
~~~

The two intermediate results may differ because the interpolation spaces differ. The weight is the contribution from the first color.

## 10. Compare Sass mixing and CSS color-mix()

Sass color.mix() computes a color at build time. CSS color-mix() stays in the stylesheet for the browser to calculate.

~~~scss
@use "sass:color";

$brand: #2457c5;
$compile-time-mix: color.mix($brand, white, $method: oklch);

.sass-mix {
  background-color: $compile-time-mix;
}

.browser-mix {
  background-color: color-mix(in oklch, #2457c5, white);
}
~~~

Use Sass mixing for generated static tokens. Use CSS color-mix() when runtime CSS behavior is useful and supported by the target browsers.

## 11. Create color variations in a token map

A map can keep color values named by their UI role. color functions can derive related values from one base token.

~~~scss
@use "sass:color";
@use "sass:map";

$tokens: (
  "brand": #2457c5,
  "surface": #ffffff,
  "text": #172033
);

$brand: map.get($tokens, "brand");
$brand-hover: color.scale($brand, $lightness: 12%, $space: oklch);
$brand-muted: color.scale($brand, $chroma: -35%, $space: oklch);

.button {
  color: map.get($tokens, "surface");
  background-color: $brand;
}

.button:hover {
  background-color: $brand-hover;
}

.button:disabled {
  background-color: $brand-muted;
}
~~~

Check the resulting foreground and background colors in the actual design. These calculations do not determine accessible contrast automatically.

## 12. Generate custom properties from a color map

Create runtime CSS tokens from Sass data.

~~~scss
@use "sass:map";

$colors: (
  "brand": #2457c5,
  "surface": #ffffff,
  "text": #172033
);

:root {
  @each $name, $value in $colors {
    --color-#{$name}: #{$value};
  }
}
~~~

Compiled CSS:

~~~css
:root {
  --color-brand: #2457c5;
  --color-surface: #ffffff;
  --color-text: #172033;
}
~~~

Interpolation places Sass values into CSS custom property values.

## 13. Check visual color equality across spaces

Two values may use different representations while describing the same rendered color.

~~~scss
@use "sass:color";

$brand: #2457c5;
$brand-oklch: color.to-space($brand, oklch);

@debug color.same($brand, $brand-oklch); // true
~~~

Use color.same() when visual equivalence matters. The ordinary equality operator compares Sass values according to Sass's value rules.

## 14. Map a color into a target gamut

Use to-gamut() when the output needs to fit a particular color space.

~~~scss
@use "sass:color";

$wide-color: oklch(60% 0.3 30deg);

$mapped-color: color.to-gamut(
  $wide-color,
  $space: rgb,
  $method: local-minde
);

$clipped-color: color.to-gamut(
  $wide-color,
  $space: rgb,
  $method: clip
);

@debug $mapped-color;
@debug $clipped-color;
~~~

local-minde reduces chroma to find a perceptually similar in-gamut color. clip clamps channels to the target range. The results may differ.

## 15. Detect a missing channel

Modern CSS color syntax can mark a channel as none. channel() reads zero for a missing channel, so use is-missing() when you need to know whether it was absent.

~~~scss
@use "sass:color";

$partially-known: rgb(100 none 200);

@debug color.is-missing($partially-known, "green"); // true
@debug color.channel($partially-known, "green", $space: rgb); // 0
~~~

Missing channels are an advanced feature; ordinary application tokens usually use explicit channel values.

## 16. Use an OKLCH color as a token

A design system can store modern color values directly as tokens.

~~~scss
$brand: oklch(48% 0.16 260deg);
$surface: oklch(99% 0.01 260deg);
$text: oklch(22% 0.03 260deg);

.card {
  color: $text;
  background-color: $surface;
  border-color: color.scale($brand, $lightness: 55%, $space: oklch);
}
~~~

For projects supporting older browsers, provide compatible fallback declarations before modern values.

## 17. Practice tasks

1. Define a brand color and inspect its RGB and OKLCH channels.
2. Convert a hex color to OKLCH and inspect its space.
3. Produce one fixed lightness adjustment and one proportional lightness scale.
4. Set an exact lightness value.
5. Mix two colors in RGB and OKLCH, then compare the results.
6. Add a fallback before an OKLCH declaration.
7. Create a translucent background without fading the text above it.
8. Map a color into the RGB gamut using both supported mapping methods.
9. Check whether a modern color has a missing channel.
10. Compare a hex color to its converted representation using color.same().

### Sample answers

~~~scss
@use "sass:color";

$brand: #2457c5;
$accent: #e65a3b;

$brand-in-oklch: color.to-space($brand, oklch);
$brand-space: color.space($brand-in-oklch);
$brand-red: color.channel($brand, "red", $space: rgb);
$brand-lightness: color.channel($brand, "lightness", $space: oklch);

$fixed: color.adjust($brand, $lightness: 8%, $space: oklch);
$scaled: color.scale($brand, $lightness: 18%, $space: oklch);
$exact: color.change($brand, $lightness: 55%, $space: oklch);
$rgb-mix: color.mix($brand, $accent, $method: rgb);
$oklch-mix: color.mix($brand, $accent, $method: oklch);

.demo {
  background-color: #3159a5;
  background-color: oklch(48% 0.15 260deg);
  box-shadow: 0 0 0 0.25rem rgb(36 87 197 / 25%);
  border-color: $scaled;
}

$wide-color: oklch(60% 0.3 30deg);
$mapped: color.to-gamut($wide-color, $space: rgb, $method: local-minde);
$clipped: color.to-gamut($wide-color, $space: rgb, $method: clip);

$unknown-channel: rgb(100 none 200);
$has-missing-green: color.is-missing($unknown-channel, "green");
$same-color: color.same($brand, color.to-space($brand, oklch));

@debug $brand-space;
@debug $brand-red;
@debug $brand-lightness;
@debug $fixed;
@debug $exact;
@debug $rgb-mix;
@debug $oklch-mix;
@debug $mapped;
@debug $clipped;
@debug $has-missing-green;
@debug $same-color;
~~~

The first debug values identify the converted space and requested channels. The final equality check is true because the conversion preserves the color.

## Further reading

- [Sass colors](https://sass-lang.com/documentation/values/colors/)
- [sass:color module](https://sass-lang.com/documentation/modules/color/)
- [CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/)
