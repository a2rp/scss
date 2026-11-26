# Source maps and browser developer tools — Examples

This exercise compiles a small Sass component, inspects its mapped source in Chrome DevTools, and traces an overridden declaration back to the rule that wins. It also shows how to check source-map availability and disable maps in a build that should not publish them.

## 1. Create the sample files

~~~text
sass-source-map-demo/
  index.html
  src/
    _tokens.scss
    _button.scss
    main.scss
  dist/
    main.css          generated
    main.css.map      generated
~~~

Use your project's normal local web server. Serve the project over a local HTTP origin and open <code>index.html</code> through that server so the browser can request the generated CSS, map, and Sass sources.

## 2. Define a token partial

Create <code>src/_tokens.scss</code>:

~~~scss
$color-brand: #2457d6;
$color-text: #202534;
$space-button: 0.75rem 1rem;
$radius-button: 0.375rem;
~~~

The underscore makes this a Sass partial. Sass can load it as a module without producing a separate CSS file for the partial.

## 3. Write the component in a partial

Create <code>src/_button.scss</code>:

~~~scss
@use "tokens" as tokens;

.button {
  padding: tokens.$space-button;
  border: 0;
  border-radius: tokens.$radius-button;
  color: white;
  background-color: tokens.$color-brand;
  font: inherit;
  cursor: pointer;

  &:hover {
    background-color: #1746bb;
  }

  &:focus-visible {
    outline: 3px solid currentColor;
    outline-offset: 3px;
  }
}
~~~

The base rule and nested states are Sass source locations that can appear in the generated map.

## 4. Create an entry point with an intentional override

Create <code>src/main.scss</code>:

~~~scss
@use "button";

.page {
  padding: 2rem;

  .button {
    color: #202534;
  }
}
~~~

The nested <code>.page .button</code> selector has greater specificity than <code>.button</code>. This intentionally overrides the white text color so you can inspect which declaration wins in DevTools.

## 5. Add the page

Create <code>index.html</code>:

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Sass source-map demo</title>
    <link rel="stylesheet" href="dist/main.css">
  </head>
  <body>
    <main class="page">
      <h1>Source maps and DevTools</h1>
      <button class="button" type="button">Inspect this button</button>
    </main>
  </body>
</html>
~~~

## 6. Compile Sass and generate the map

Run Dart Sass from the project root:

~~~sh
sass src/main.scss dist/main.css
~~~

For CLI file output, Dart Sass generates a source map by default. Look for:

~~~text
dist/
  main.css
  main.css.map
~~~

Open the end of <code>dist/main.css</code> and confirm it has a source-mapping URL comment that points to <code>main.css.map</code>. The map should list the original Sass files that contributed to the CSS.

To keep recompiling while editing, use watch mode:

~~~sh
sass --watch src/main.scss:dist/main.css
~~~

Leave the watch process running while you make changes. Confirm Sass reports a successful rebuild before refreshing the page, so the CSS and map belong to the same compilation.

## 7. Open mapped Sass in Chrome

1. Open the locally served page in Chrome.
2. Open DevTools and enable CSS source maps in **Settings → Preferences → Sources**.
3. Use the element picker to select the button.
4. In **Elements → Styles**, locate the <code>color</code> declaration.
5. Click its source link to open the mapped Sass file.
6. Follow the source links for the base button rule and the page-specific override.

The Styles panel should show both matching rules. The white <code>color</code> from <code>_button.scss</code> is crossed out, while the darker value from <code>main.scss</code> wins because the nested selector is more specific.

This demonstrates the two parts of the investigation: the source map identifies Sass locations, while the browser cascade determines the final computed value.

## 8. Inspect the computed value and test a fix

With the button selected, open the **Computed** panel and inspect <code>color</code>. Expand the property details if available to see the rules that contributed the value.

Temporarily disable the winning declaration in Styles. The button text should return to white. This is a browser experiment; it does not change the Sass file on disk.

Make the actual fix in <code>src/main.scss</code>. For example, remove the accidental nested override:

~~~scss
@use "button";

.page {
  padding: 2rem;
}
~~~

Save the file, wait for the Sass watcher to compile successfully, and refresh the page. Reopen DevTools and confirm that the source map now leads to the updated source and the computed text color is white.

## 9. Check source-map loading when Sass is missing

If DevTools shows only generated CSS:

1. Confirm CSS source maps are enabled in DevTools settings.
2. Open <code>dist/main.css</code> and verify the source-mapping URL comment exists.
3. Confirm <code>dist/main.css.map</code> exists and is valid JSON.
4. In DevTools, open **More tools → Developer Resources** and inspect the map's load status.
5. Check the Console and Network panels for missing map or source requests.
6. Check that map source paths resolve from the map's location.
7. Recompile after source changes, then reload the page to avoid stale CSS or maps.

A map may load while a Sass source does not. If the map does not embed source contents, DevTools needs to retrieve the source paths listed inside it.

## 10. Optionally embed Sass source contents

To include source text in the CLI-generated map, compile with:

~~~sh
sass --embed-sources src/main.scss dist/main.css
~~~

DevTools can then use the included sources even when the original files are not separately served. The map becomes larger and contains the Sass source code, so use this only when that trade-off is acceptable.

To embed map data inside the CSS instead of using a separate map file, Dart Sass also provides <code>--embed-source-map</code>. This is separate from <code>--embed-sources</code>: one embeds the map in CSS, and the other embeds source contents in the map.

## 11. Disable source maps for a selected output

If a particular build should not create a map:

~~~sh
sass --no-source-map src/main.scss dist/main.css
~~~

After compiling, confirm that the generated output no longer references a Sass map and that the map file is not being copied from a stale prior build. A separate bundler may still generate a map, so check the complete pipeline and deployment artifacts.

Source maps do not change page styling. Decide whether to publish them based on debugging needs and source-disclosure policy.

## 12. Inspect a source map as a file

Open <code>dist/main.css.map</code> as JSON. Common fields include:

- <code>version</code>: the source-map format version.
- <code>sources</code>: the original source paths.
- <code>names</code>: optional symbol names used by the map.
- <code>mappings</code>: encoded generated-to-original position mappings.
- <code>sourcesContent</code>: optional embedded source text.

The mapping data is encoded for tools rather than intended for manual editing. Use DevTools or a source-map viewer to navigate mappings; inspect the JSON mainly to diagnose paths and whether source text is embedded.

## 13. Debug an incorrect mapped location

If clicking a rule opens an unexpected Sass file or line:

1. Confirm the browser loaded the latest CSS.
2. Confirm that CSS references the expected map filename.
3. Check whether the map is stale or was generated before a Sass edit.
4. Check whether another tool transforms CSS after Sass and whether it composes source maps.
5. Confirm the map source path and source root resolve correctly.
6. Rebuild CSS and map together and reload the page.

Do not manually edit the map to compensate for a broken build pipeline. Fix the generation or map-composition step that produced the mismatch.

## Practice checklist

- Compile the sample and find the emitted CSS map.
- Open the button in DevTools and trace its rules to both Sass files.
- Use Styles and Computed to identify why the text color is overridden.
- Remove the override in Sass, rebuild, and confirm the corrected computed value.
- Inspect Developer Resources if a map or original file does not load.
- Try <code>--embed-sources</code> and compare the map contents and size.
- Try <code>--no-source-map</code> and confirm the output no longer references a map.

## Official references

- [Dart Sass CLI: source maps](https://sass-lang.com/documentation/cli/dart-sass/#source-maps)
- [Sass JavaScript API: compile options](https://sass-lang.com/documentation/js-api/interfaces/options/)
- [Sass JavaScript API: CompileResult](https://sass-lang.com/documentation/js-api/interfaces/compileresult/)
- [Chrome DevTools: CSS source maps preferences](https://developer.chrome.com/docs/devtools/settings/preferences/)
- [Chrome DevTools: debug authored source with source maps](https://developer.chrome.com/docs/devtools/javascript/source-maps/)
- [Chrome DevTools: view and manually load source maps](https://developer.chrome.com/docs/devtools/developer-resources/)
- [Chrome DevTools: inspect CSS in Styles and Computed](https://developer.chrome.com/docs/devtools/css/reference/)
