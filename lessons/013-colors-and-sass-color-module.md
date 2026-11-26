# 013 - Colors and the Sass Color Module

## Learning goals

After this lesson, you should be able to:

- Write colors using names, hexadecimal values, and CSS color functions.
- Explain alpha transparency and the difference between a color and a quoted color name.
- Recognize legacy RGB/HSL/HWB spaces and modern perceptual spaces such as OKLCH.
- Load and use the sass:color module.
- Read a channel, inspect a color space, and convert a color between spaces.
- Distinguish fixed adjustments, proportional scaling, and setting a channel.
- Mix colors in an explicitly chosen color space.
- Understand gamut mapping, browser fallbacks, and why color calculations do not guarantee accessible contrast.

## 1. Sass colors are typed values

Sass recognizes colors as values. They can be assigned to variables, passed to functions, stored in maps, and used in CSS declarations.

~~~scss
$brand: #2457c5;
$success: seagreen;
$highlight: rgb(245 158 11);

.button {
  color: white;
  background-color: $brand;
  border-color: $highlight;
}
~~~

A color name such as red is parsed as a color value. If you need the literal text "red", quote it:

~~~scss
$danger-color: red;
$danger-label: "red";
~~~

The first variable is a color. The second is a string.

## 2. Common ways to write colors

### Named colors

CSS named colors are convenient for quick examples and small adjustments:

~~~scss
$notice: tomato;
$surface: midnightblue;
~~~

For a design system, tokens with names that describe their role are usually easier to maintain than scattered literal color names.

### Hexadecimal notation

Hex values can use three, four, six, or eight hexadecimal digits:

~~~scss
$short: #36c;
$short-alpha: #36c8;
$full: #3366cc;
$full-alpha: #3366cc88;
~~~

Three digits expand to six; four digits include alpha. Eight digits use the final two hexadecimal digits for alpha.

### RGB

RGB represents red, green, and blue channels. CSS supports modern space-separated syntax and an optional slash-separated alpha channel:

~~~scss
$blue: rgb(51 102 204);
$translucent-blue: rgb(51 102 204 / 65%);
~~~

Legacy comma syntax is also accepted in legacy color functions:

~~~scss
$blue: rgb(51, 102, 204);
$translucent-blue: rgba(51, 102, 204, 0.65);
~~~

Prefer current CSS syntax for new stylesheets unless a project's browser support requires older syntax.

### HSL and HWB

HSL describes hue, saturation, and lightness. HWB describes hue, whiteness, and blackness.

~~~scss
$violet: hsl(270deg 50% 40%);
$warm-gray: hwb(30deg 60% 25%);
~~~

Hue is an angle; the other channels are percentages. These spaces can be helpful for understanding a color, though they are not perceptually uniform.

### Lab, LCH, OKLab, and OKLCH

Modern CSS supports perceptual color spaces. Lab and LCH, and their Oklab and OKLCH variants, represent colors using channels that can be more useful for controlled color adjustments.

~~~scss
$accessible-violet: oklch(45% 0.16 280deg);
$soft-green: lab(75% -30 25);
~~~

OKLCH uses lightness, chroma, and hue:

- Lightness describes how light or dark the color is.
- Chroma describes colorfulness.
- Hue describes the color family around the hue circle.

These values may be less familiar than hex, but they can make it easier to reason about perceptual changes.

### Device-oriented color() syntax

The CSS color() function supports explicit color spaces such as sRGB and Display P3:

~~~scss
$saturated-green: color(display-p3 0.1 0.8 0.25);
~~~

Display P3 can represent colors outside the sRGB gamut. A browser or display that cannot show those colors may render a clipped or mapped result.

## 3. Alpha transparency

Alpha controls a color's opacity. CSS alpha can be written as a decimal from 0 to 1 or as a percentage.

~~~scss
$solid: rgb(36 87 197 / 100%);
$mostly-opaque: rgb(36 87 197 / 80%);
$transparent: rgb(36 87 197 / 0%);
~~~

In hexadecimal notation, the final two digits encode alpha. Alpha belongs to the color value; it is not the same as applying the CSS opacity property to an entire element. The opacity property also affects the element's descendants and rendered content.

~~~scss
.overlay {
  background-color: rgb(0 0 0 / 50%);
}
~~~

## 4. Color spaces and compatibility

A color space defines the channels used to describe a color and how those channel values relate to rendered color.

The traditional CSS color spaces are RGB, HSL, and HWB. Modern CSS also supports spaces such as Lab, LCH, Oklab, OKLCH, XYZ, and device-specific spaces such as Display P3.

Dart Sass 1.79.0 introduced support for modern CSS color spaces and the modern sass:color APIs that work across them. Older Sass implementations and older Dart Sass releases may not understand these functions or syntaxes. Check the compiler version in the project before using modern color features.

A Sass compiler accepting a declaration does not guarantee that every browser or display can render it. Use a fallback when the project's browser support requires one.

~~~scss
.callout {
  background-color: #3159a5;
  background-color: oklch(48% 0.15 260deg);
}
~~~

Browsers that support the second value use it. Browsers that do not support it retain the earlier fallback declaration.

## 5. Load the sass:color module

Use the built-in module with @use:

~~~scss
@use "sass:color";

$brand: #2457c5;
$dark-brand: color.scale($brand, $lightness: -12%, $space: oklch);
~~~

The color namespace identifies Sass color operations. The modern module functions can work across supported color spaces.

## 6. Inspect a color's space and channels

color.space() returns the name of a color's space. color.channel() reads one channel.

~~~scss
@use "sass:color";

$brand: #2457c5;

@debug color.space($brand); // rgb
@debug color.channel($brand, "red", $space: rgb);
@debug color.channel($brand, "hue", $space: oklch);
@debug color.channel($brand, "lightness", $space: oklch);
~~~

Channel names are quoted strings. Color-space names are unquoted Sass identifiers.

Channels may have units: hue values are angles, and channels such as HSL lightness or OKLCH lightness are percentages. Read the channel in the space that gives the measurement you intend.

## 7. Convert a color with color.to-space()

color.to-space() converts a color to another color space:

~~~scss
@use "sass:color";

$brand: #2457c5;
$brand-oklch: color.to-space($brand, oklch);
$brand-p3: color.to-space($brand, display-p3);

@debug color.space($brand-oklch); // oklch
@debug color.space($brand-p3); // display-p3
~~~

Conversion changes the representation of the color. It does not necessarily change how the color looks.

Most other Sass color functions return a color in the same space as the input color. color.to-space() is the explicit conversion function.

## 8. Adjust, scale, and change are different operations

The color module provides distinct ways to modify channels. Choose the function that matches your intent.

### color.adjust(): add or subtract a fixed amount

adjust() moves one or more channels by a fixed amount:

~~~scss
@use "sass:color";

$brand: #2457c5;
$brighter: color.adjust($brand, $lightness: 8%, $space: oklch);
$less-blue: color.adjust($brand, $hue: -8deg, $space: oklch);
~~~

A fixed change can reach a channel's limit. Sass clamps channels where the color-space rules require it.

### color.scale(): move proportionally toward a limit

scale() moves a channel a percentage of the remaining distance toward its minimum or maximum:

~~~scss
@use "sass:color";

$brand: #2457c5;
$soft-brand: color.scale($brand, $lightness: 18%, $space: oklch);
$deeper-brand: color.scale($brand, $lightness: -18%, $space: oklch);
~~~

This often produces more useful relative changes than adding the same fixed amount to colors that started at different channel values. Scaling percentages are between -100% and 100%.

### color.change(): set a channel to an exact value

change() sets a channel instead of adding or scaling it:

~~~scss
@use "sass:color";

$brand: #2457c5;
$fixed-lightness: color.change($brand, $lightness: 55%, $space: oklch);
~~~

Use it when a specific channel value is required. Unlike adjust(), change() does not clamp out-of-range channels automatically.

All three functions return the result in the original color's space. Pass $space explicitly to make the calculation space clear, especially for colors written in legacy RGB/HSL/HWB spaces.

## 9. Choose a perceptual space deliberately

When adjusting a color for perceived lightness or saturation, OKLCH can be a useful working space:

~~~scss
@use "sass:color";

$base: #3159a5;
$hover: color.scale($base, $lightness: 12%, $space: oklch);
$muted: color.scale($base, $chroma: -30%, $space: oklch);
~~~

The original colors remain represented in their input space after these operations, while Sass computes the requested change in OKLCH.

Color perception still depends on context, surrounding colors, display capabilities, and the design goal. Always inspect the compiled result in the intended UI.

## 10. Mix colors with color.mix()

color.mix() combines two colors. For predictable results across color spaces, specify the interpolation method:

~~~scss
@use "sass:color";

$brand: #2457c5;
$accent: #e65a3b;

$blend: color.mix($brand, $accent, $weight: 35%, $method: oklch);
~~~

The weight controls how much of the first color contributes to the mix. With a weight of 35%, the first color contributes 35% and the second contributes the remainder.

The selected color space affects the interpolation path. Mixing in RGB and mixing in OKLCH can produce different intermediate colors.

~~~scss
@use "sass:color";

$rgb-blend: color.mix(#2457c5, #e65a3b, $method: rgb);
$oklch-blend: color.mix(#2457c5, #e65a3b, $method: oklch);
~~~

For polar spaces such as OKLCH, Sass also supports hue interpolation methods. Use them only when the desired path around the hue circle is intentional.

## 11. Compare colors with color.same()

The equality operator compares Sass values, including their representation. color.same() checks whether two colors visually represent the same color even if they use different spaces.

~~~scss
@use "sass:color";

$brand: #2457c5;
$brand-oklch: color.to-space($brand, oklch);

@debug $brand == $brand-oklch;
@debug color.same($brand, $brand-oklch); // true
~~~

Use color.same() for visual equivalence across color-space representations.

## 12. Gamut and out-of-gamut colors

A color space has a gamut: the range of colors it can represent. A color can be valid Sass data and still be outside the gamut of a target color space or display.

Sass can preserve out-of-gamut channel values. CSS or a display may clip or map those values during rendering. Gamut mapping attempts to preserve a visually similar color within the requested target gamut.

~~~scss
@use "sass:color";

$wide-gamut-color: oklch(60% 0.3 30deg);
$safe-for-rgb-gamut: color.to-gamut(
  $wide-gamut-color,
  $space: rgb,
  $method: local-minde
);
~~~

The local-minde method is intended to find a perceptually similar in-gamut color. The clip method simply clips channels to the target range and can change appearance more noticeably.

Use gamut mapping when a color must fit a target gamut. It is not necessary to convert every ordinary sRGB color.

## 13. Missing and powerless channels

Modern CSS color syntax can use none for a missing channel. A missing channel is not the same idea as a channel whose value is zero.

~~~scss
$gray: hsl(none 0% 50%);
~~~

In a gray HSL color, hue does not affect the rendered color because saturation is zero; that channel is powerless. In other cases a channel may be explicitly missing.

color.channel() returns zero for a missing channel. Use color.is-missing() when the distinction matters:

~~~scss
@use "sass:color";

$unknown-green: rgb(100 none 200);

@debug color.is-missing($unknown-green, "green"); // true
@debug color.channel($unknown-green, "green", $space: rgb); // 0
~~~

This is an advanced CSS color feature. Most everyday color tokens do not need missing channels.

## 14. Legacy helpers and safer modern choices

Sass has historical global functions such as lighten(), darken(), saturate(), and transparentize(). Their fixed channel changes can hit a boundary sooner than intended, and many are limited to legacy color spaces.

For new code, prefer explicit module functions:

~~~scss
@use "sass:color";

$brand: #2457c5;

$lighter: color.scale($brand, $lightness: 15%, $space: oklch);
$darker: color.scale($brand, $lightness: -15%, $space: oklch);
$more-transparent: color.scale($brand, $alpha: -25%, $space: oklch);
~~~

Use adjust() when adding or subtracting a fixed channel amount. Use scale() when moving proportionally toward a channel limit. Use change() when setting an exact channel value.

## 15. Color calculations do not guarantee accessible contrast

A lighter color is not automatically readable against every background. Hue, chroma, background, font size, and display conditions affect the final result.

Use Sass color functions to create and organize values, then evaluate foreground/background contrast for the actual interface and its accessibility requirements. Do not assume that a generated hover, disabled, or text color automatically meets contrast standards.

## 16. Common mistakes

### Quoting the color-space name

Color-space arguments are unquoted:

~~~scss
@use "sass:color";

$converted: color.to-space(#2457c5, oklch);
~~~

### Leaving the channel name unquoted

Channel names are quoted:

~~~scss
@use "sass:color";

$lightness: color.channel(#2457c5, "lightness", $space: oklch);
~~~

### Confusing adjust() with scale()

adjust() adds a fixed amount. scale() moves a percentage of the remaining distance toward a limit. They produce different outcomes.

### Assuming changing a color space always changes its appearance

to-space() changes representation. It does not necessarily change the visual color.

### Ignoring compiler or browser support

Modern color syntax and color-space APIs require sufficiently recent Dart Sass and browser support. Use a fallback declaration when needed.

## 17. Practice checklist

Before moving on, make sure you can answer these questions:

1. Which hexadecimal digits represent alpha in an eight-digit hex color?
2. What is the difference between RGB and HSL channel descriptions?
3. How do color.adjust(), color.scale(), and color.change() differ?
4. Why should color.mix() specify a method for modern color calculations?
5. Which arguments to color.channel() are quoted?
6. What does color.to-space() change?
7. What is an out-of-gamut color?
8. Why is a lighter color not guaranteed to have accessible contrast?
9. When should you provide a fallback color declaration?
10. What does a missing channel mean?
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="012-maps-and-map-functions.md">012 - Maps and Map Functions</a></td><td><a href="../qnas/013-colors-and-sass-color-module.md">Q&amp;A</a></td><td><a href="../examples/013-colors-and-sass-color-module.md">Example</a></td><td><a href="014-comments-and-css-output.md">014 - Comments and Generated CSS Output</a></td></tr>
  </tbody>
</table>
