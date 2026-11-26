# CSS output, performance, and maintenance — Examples

This exercise creates two Sass entry points, compares readable and compressed CSS output, and investigates unused CSS with Chrome DevTools. It also demonstrates why coverage data should lead to a review rather than automatic deletion.

## 1. Create the sample project

~~~text
css-output-demo/
  index.html
  src/
    _tokens.scss
    _base.scss
    app.scss
    admin.scss
    components/
      _card.scss
      _admin-panel.scss
  dist/
    app.css          generated
    admin.css        generated
    app.min.css      generated
~~~

The <code>dist</code> files are build output. Do not edit them by hand; change Sass source and rebuild.

## 2. Define Sass-only tokens

Create <code>src/_tokens.scss</code>:

~~~scss
$color-ink: #202534;
$color-muted: #596579;
$color-surface: #f4f6fa;
$color-brand: #2457d6;
$space-card: 1rem;
$radius-card: 0.5rem;
~~~

This module defines values but has no style rules, so loading it by itself does not emit CSS.

## 3. Add shared base styles

Create <code>src/_base.scss</code>:

~~~scss
@use "tokens" as tokens;

:root {
  color: tokens.$color-ink;
  background-color: tokens.$color-surface;
  font-family: system-ui, sans-serif;
}

body {
  margin: 0;
}

button,
input {
  font: inherit;
}
~~~

This module emits global CSS. Any entry point that loads it includes these styles in that entry point's CSS output.

## 4. Add a card component

Create <code>src/components/_card.scss</code>:

~~~scss
@use "../tokens" as tokens;

.card {
  padding: tokens.$space-card;
  border: 1px solid #d6dce7;
  border-radius: tokens.$radius-card;
  background-color: white;

  &__title {
    margin-block: 0 0.5rem;
    color: tokens.$color-ink;
  }

  &__description {
    margin: 0;
    color: tokens.$color-muted;
  }
}

/* Used only on routes that render an open dialog. */
.dialog[open] {
  border: 1px solid #d6dce7;
  border-radius: tokens.$radius-card;
  padding: tokens.$space-card;
}
~~~

The dialog rule is included in the app output even if the demo page below does not contain a dialog. Coverage will identify it as unobserved in that session.

## 5. Add admin-only styles

Create <code>src/components/_admin-panel.scss</code>:

~~~scss
@use "../tokens" as tokens;

.admin-panel {
  max-width: 48rem;
  margin-inline: auto;
  padding: 1.5rem;
  border-top: 4px solid tokens.$color-brand;
  background-color: white;
}
~~~

This module belongs to the admin entry point. A public page that does not need an admin panel should not load its CSS.

## 6. Create intentional Sass entry points

Create <code>src/app.scss</code>:

~~~scss
@use "base";
@use "components/card";
~~~

Create <code>src/admin.scss</code>:

~~~scss
@use "base";
@use "components/admin-panel";
~~~

Each output contains its intended styles. The shared base rules appear once in each independently compiled file. If one page loads both files, the base rules will be duplicated across the requests. In this sample, the public page loads only <code>app.css</code> and an admin page would load only <code>admin.css</code>.

Compile both entry points:

~~~sh
sass --style=expanded src/app.scss:dist/app.css src/admin.scss:dist/admin.css
~~~

The Sass CLI skips partial files with underscore-prefixed names when compiling a directory. Keeping explicit entry points makes it clear which styles should become standalone CSS files.

## 7. Add a page that uses the card

Create <code>index.html</code>:

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>CSS output demo</title>
    <link rel="stylesheet" href="dist/app.css">
  </head>
  <body>
    <main>
      <article class="card">
        <h1 class="card__title">A small card</h1>
        <p class="card__description">
          Inspect the styles and coverage for this page.
        </p>
      </article>
    </main>
  </body>
</html>
~~~

Serve the page through the project's local web server and open it in Chrome. Avoid loading both <code>app.css</code> and <code>admin.css</code> on this page.

## 8. Compare expanded and compressed builds

Generate readable output for development and compact output for comparison:

~~~sh
sass --style=expanded src/app.scss dist/app.css
sass --style=compressed src/app.scss dist/app.min.css
~~~

In PowerShell, compare the raw generated file sizes:

~~~powershell
Get-Item distapp.css,distapp.min.css | Select-Object Name,Length
~~~

The compressed file should usually be smaller as text, but that is not the final network measurement. HTTP compression, cache headers, and server configuration affect the transferred size. In Chrome DevTools, use the Network panel's Size information to inspect resource and transfer sizes for the actual page.

Sass CLI file builds generate source maps by default. The map is a separate artifact and is not an additional stylesheet link in this HTML. If the production artifact policy disables maps, use <code>--no-source-map</code> consistently when producing the output you measure.

## 9. Inspect loaded stylesheets in Network

With DevTools open:

1. Select **Network** and enable **Disable cache** for a controlled local comparison.
2. Reload the page.
3. Filter the resource list to CSS.
4. Check which stylesheet URLs loaded and how many requests were made.
5. Inspect resource size, transferred size, timing, and cache status.
6. Confirm this page loads <code>app.css</code> but not <code>admin.css</code>.
7. Compare the same page and environment before and after a CSS change.

Disabling cache is useful for a cold-load comparison. Also test normal cache behavior because repeat visits may reuse stylesheets. Keep the test conditions the same when comparing results.

## 10. Find unused CSS with Coverage

In Chrome DevTools:

1. Open the Command Menu and search for **Show Coverage**.
2. Start instrumenting coverage and reload the page.
3. Interact with the page while recording.
4. Select <code>app.css</code> to see the line-level usage report.
5. Find the <code>.dialog[open]</code> rule. It may appear unused because this page did not render a dialog.
6. Open a route or state that does render a dialog and record coverage again.

Coverage reflects only resources and states observed during the recorded session. The dialog may be required elsewhere. Search the application for its markup and class usage, test every route and state that needs it, and remove the CSS only if the project confirms it is unnecessary.

## 11. Investigate CSS that is truly unused

When a selector looks unused in several representative recordings:

1. Search templates, components, and dynamically generated class names for references.
2. Check route-specific, responsive, theme, hover, focus, open, and error states.
3. Confirm no content is loaded lazily after the recording ends.
4. Map the rule back to Sass source and identify the module that emits it.
5. Remove or split the source rule only after confirming its ownership and use.
6. Rebuild and record the same routes and interactions again.
7. Check both the output-size change and the rendered behavior.

A repeated unused report is stronger evidence than a single session, but review the application architecture before deleting a shared or state-dependent rule.

## 12. Check for duplicate output

If multiple CSS files are loaded, inspect their compiled output for repeated base or component rules. In the sample, both entry points include <code>_base.scss</code>; that is useful if each route group loads only one output, but duplicates base declarations if the same page loads both.

Possible fixes include:

- Load only the entry point needed by that page.
- Move shared CSS into one separately loaded base stylesheet.
- Use the project's bundler to extract common CSS when it supports this workflow.
- Reorganize component output so each page group loads the styles it needs.

After changing entry points, inspect stylesheet order and computed styles because output splitting can change cascade behavior.

## 13. Keep the development and production commands explicit

Example package scripts:

~~~json
{
  "scripts": {
    "build:css:dev": "sass --style=expanded src/app.scss dist/app.css",
    "build:css:prod": "sass --style=compressed --no-source-map src/app.scss dist/app.min.css"
  }
}
~~~

Merge these scripts into the project's existing manifest and use its installed Sass version. The development command keeps readable CSS and the CLI's default map behavior. The production command requests compressed CSS and disables Sass source maps for that artifact. If a bundler owns final CSS output, configure the settings there instead.

Do not rely on a local development output as proof of production size. Measure the file and transfer behavior produced by the actual production pipeline.

## Practice checklist

- Compile the app and admin entry points and inspect both outputs.
- Confirm partial files are not emitted as standalone CSS.
- Compare expanded and compressed raw sizes.
- Inspect the page's CSS requests and transfer sizes in Network.
- Use Coverage on the demo page, then visit a dialog state before drawing conclusions.
- Check whether independently compiled entry points duplicate shared CSS when loaded together.
- Rebuild after changes and recheck output, page behavior, and measurements.

## Official references

- [Dart Sass CLI: output styles and compilation](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass: @use and module output](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: @import and migration](https://sass-lang.com/documentation/at-rules/import/)
- [Chrome DevTools: Coverage](https://developer.chrome.com/docs/devtools/coverage/)
- [Chrome DevTools: CSS inspection and Coverage](https://developer.chrome.com/docs/devtools/css/reference/)
- [web.dev: the critical rendering path](https://web.dev/learn/performance/understanding-the-critical-path)
- [Chrome Lighthouse: render-blocking resources](https://developer.chrome.com/docs/lighthouse/performance/render-blocking/)
