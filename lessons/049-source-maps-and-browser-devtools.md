# Source maps and browser developer tools

Sass turns authored stylesheets into CSS that the browser loads. A source map connects locations in that generated CSS back to the Sass source that produced them. Browser developer tools can use that mapping to show source files while you inspect the styles applied to a page.

Source maps help answer two related questions: which Sass rule generated this browser-visible declaration, and why did the browser choose this rule for this element? The first is about the build mapping; the second is about the CSS cascade and runtime state.

## Learning goals

After this lesson, you should be able to:

- Explain how a source map connects generated CSS locations to Sass source.
- Generate and configure source maps with the Dart Sass CLI.
- Understand the different source-map defaults in the Sass JavaScript API.
- Use browser developer tools to inspect Sass-backed rules, computed styles, and override causes.
- Diagnose missing or incorrectly loaded source maps.
- Choose source-map settings that fit local development, CI, and production exposure needs.

## 1. Follow the path from Sass to the browser

A typical stylesheet flow looks like this:

~~~text
SCSS source
   ↓ Dart Sass compilation
generated CSS + source map
   ↓ development server sends CSS and map
browser loads CSS
   ↓ DevTools reads the source map
DevTools connects a CSS declaration to its Sass source
~~~

The browser applies the generated CSS. A source map is metadata for tools; it does not change the cascade or make the browser execute Sass.

A source map associates generated CSS positions with original source positions. It can contain source paths and, optionally, copies of Sass source contents. A generated declaration may map to a rule in a partial because that partial contributed to the compiled stylesheet.

## 2. Generate a source map with the Dart Sass CLI

When the Dart Sass CLI writes CSS to a file, it generates a source map by default. The CSS points to the map, and the map points back to the Sass sources.

For example:

~~~sh
sass src/styles.scss dist/styles.css
~~~

The output commonly includes files like:

~~~text
dist/
  styles.css
  styles.css.map
~~~

The CSS includes a source-mapping URL comment that tells tools where to find the map. Keep the CSS and its map reachable at the paths described by that URL.

To explicitly embed Sass sources in the map:

~~~sh
sass --embed-sources src/styles.scss dist/styles.css
~~~

Embedding makes source contents available through the map, but can increase its size and expose original Sass code to anyone who can retrieve the map. Use it intentionally.

To turn off Sass CLI source-map generation:

~~~sh
sass --no-source-map src/styles.scss dist/styles.css
~~~

Use the option that matches the environment and your build pipeline. If another tool such as a bundler emits the final CSS, configure source maps at that layer as well.

## 3. Source-map defaults differ in the Sass JavaScript API

The Dart Sass CLI and JavaScript API do not have the same default behavior:

- The CLI generates source maps by default when it writes CSS files.
- The JavaScript API's <code>sourceMap</code> option defaults to <code>false</code>.
- When the JavaScript API returns CSS and a source-map object, the caller is responsible for saving the map and linking it from the CSS.

A Node.js build script can request a map:

~~~js
import * as sass from "sass";
import { writeFile } from "node:fs/promises";

const result = sass.compile("src/styles.scss", {
  sourceMap: true,
  sourceMapIncludeSources: true
});

await writeFile("dist/styles.css", result.css);
await writeFile(
  "dist/styles.css.map",
  JSON.stringify(result.sourceMap)
);
~~~

This example requests source contents inside the map. A complete build integration must also add a correct source-mapping URL to the CSS and ensure the referenced source files or embedded source content are available to DevTools. The API deliberately leaves those output-path decisions to the caller.

When using a framework, bundler, or Sass integration, read its source-map settings. Avoid generating multiple independent maps that point to different intermediate versions of the CSS.

## 4. Source paths and source content

A source map needs to describe where its original sources can be found. Depending on how the build is configured, map sources may be relative URLs, absolute file URLs, or paths resolved by a tool-specific root.

Relative source paths are usually more portable when the CSS, map, and sources are served from a known project layout. Absolute <code>file:</code> URLs can work on the machine that created them but generally do not identify files on another developer's computer or a deployed server.

Source content can be available in two ways:

1. The map includes source text, commonly in a <code>sourcesContent</code> field.
2. DevTools requests each Sass source using the paths in the map.

A map with embedded sources can work when original files are not directly served, but it contains their text. A map without embedded sources requires DevTools to be able to retrieve those files. Decide which approach is appropriate for local development and for any publicly reachable deployment.

## 5. Enable CSS source maps in Chrome DevTools

To inspect original Sass sources in Chrome:

1. Open the page and launch DevTools.
2. Open **Settings** and go to **Preferences → Sources**.
3. Enable **CSS source maps**.
4. Inspect an element using the element picker or the Elements panel.
5. In **Elements → Styles**, find the rule and click its source link.
6. DevTools opens the mapped Sass source when the map and source are available.

In the Sources panel, authored files may appear separately from deployed CSS. In the Styles pane, source links beside rules help navigate from the element to the Sass file that contributed the rule.

Browser interfaces can change over time, but the core requirements remain: CSS source maps must be enabled, the CSS must reference the map, and DevTools must be able to load the map and its source content.

## 6. Inspect why a declaration wins or loses

A source map answers where a declaration came from. The Styles and Computed panels answer why the browser applies a particular value.

When a property looks wrong:

1. Select the exact element in the rendered page.
2. In the **Styles** panel, find the property and check whether it is crossed out.
3. Follow the source links to see which stylesheet or Sass source produced each matching rule.
4. Open **Computed** to see the final value after the cascade, inheritance, and active conditions.
5. Expand the property details when available to inspect the rules that contributed or were overridden.
6. Check selector specificity, source order, inheritance, media queries, pseudo-classes, and custom-property values.
7. Toggle a declaration or state in DevTools to test a hypothesis.
8. Make the actual correction in the project's Sass source and rebuild.

Do not confuse a mapped source location with proof that the rule is active. A Sass rule can appear in the map but fail to match the selected element, be overridden, or be disabled by a condition.

## 7. Test states that affect CSS

Styles can differ according to the element, viewport, and interaction state. Inspect the conditions relevant to the problem:

- Hover, focus, and active states.
- Focus-visible behavior using keyboard navigation.
- Responsive breakpoints and device emulation.
- Dark or light theme classes and custom properties.
- Reduced-motion preferences.
- Dynamically added classes or inline styles.
- Inherited values from ancestor elements.

DevTools can force some pseudo-classes and emulate certain media preferences. Use those controls to reproduce a state, then check the actual page behavior too.

## 8. Troubleshoot missing Sass sources

If the Styles panel links only to generated CSS or DevTools cannot open the Sass file, work through this checklist:

1. Confirm that the generated CSS contains a source-mapping URL.
2. Confirm that the map exists at the URL referenced by the CSS.
3. Open **More tools → Developer Resources** and inspect source-map load status.
4. Check the browser console and network panel for 404s, permission errors, or invalid map JSON.
5. Check that the map's source paths resolve from the map location or configured root.
6. Confirm that CSS source maps are enabled in DevTools settings.
7. If source contents are embedded, verify that the map includes the expected source text.
8. Rebuild after changing Sass or build configuration so the CSS and map belong to the same build.
9. Clear stale output or caches if the browser still loads an old map.

A valid map file is not enough if the CSS does not reference it or the browser cannot reach it. Similarly, correct paths cannot help if the deployed CSS and map came from different builds.

## 9. Keep generated output and Sass in sync

A common source-map problem is serving a new CSS file with an old map, or an old CSS file with a map from a different build. This can send DevTools to the wrong line or file.

Use one build process to generate the CSS and its map together. During development, enable watch mode through the project's normal build command and check its output after edits. Avoid manually copying only the CSS file while leaving its map behind.

If a postprocessor or minifier changes CSS after Sass compilation, make sure the tool generates or composes a map that traces through the transformations. A map for an earlier intermediate file may no longer point to the final CSS correctly.

## 10. Understand live DevTools edits

DevTools is useful for temporary experiments: toggle a declaration, change a value, force a pseudo-class, or test a selector. Treat those edits as an investigation, not as a committed source change.

Mapped Sass files may be shown as authored sources, but an edit in DevTools is not a reliable replacement for changing the local Sass project and rebuilding. Confirm the fix in the actual source file, run the normal build, and reload to verify the final output.

## 11. Choose source-map settings by environment

| Environment | Typical choice | Things to check |
| --- | --- | --- |
| Local development | Enable maps for fast source navigation; embed sources only if convenient. | Map freshness, server paths, build speed. |
| Shared test environment | Serve maps only if testers need them and source access is intended. | Access controls, source disclosure, matching build artifacts. |
| Production | Decide explicitly whether maps should be published. | Whether maps expose source text or internal paths, deployment policy, monitoring needs. |
| JavaScript build integration | Enable the API option and have the build tool write/link the map. | API defaults, CSS URL comment, output locations. |

Source maps do not change runtime styles, but public maps can reveal source code and directory structure. Treat them as deployable artifacts and make a deliberate publication choice.

## Key takeaways

- Source maps connect generated CSS positions to the Sass source that contributed them.
- Dart Sass CLI file output includes maps by default; the JavaScript API requires <code>sourceMap: true</code>.
- DevTools needs the CSS map reference and access to the map and its source content.
- Use Styles and Computed together: source maps locate the authored rule, while the cascade determines the final browser value.
- Rebuild CSS and maps together so DevTools does not show stale or mismatched source.
- Decide intentionally whether production deployments should publish maps or embedded Sass sources.

## Official references

- [Dart Sass CLI: source maps](https://sass-lang.com/documentation/cli/dart-sass/#source-maps)
- [Sass JavaScript API: compile options](https://sass-lang.com/documentation/js-api/interfaces/options/)
- [Sass JavaScript API: CompileResult](https://sass-lang.com/documentation/js-api/interfaces/compileresult/)
- [Chrome DevTools: CSS source maps preferences](https://developer.chrome.com/docs/devtools/settings/preferences/)
- [Chrome DevTools: debug authored source with source maps](https://developer.chrome.com/docs/devtools/javascript/source-maps/)
- [Chrome DevTools: view and manually load source maps](https://developer.chrome.com/docs/devtools/developer-resources/)
- [Chrome DevTools: inspect CSS in Styles and Computed](https://developer.chrome.com/docs/devtools/css/reference/)

## Next lesson

Continue with [CSS output, performance, and maintenance](050-css-output-performance-and-maintenance.md).
