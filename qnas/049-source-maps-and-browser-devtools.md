# Source maps and browser developer tools — Q&A

This Q&A explains how Sass source maps connect generated CSS to source files and how browser developer tools help inspect the final cascade. Sass and browser tooling can change, so check current options for the versions used by your project.

## 1. What is a source map?

A source map is metadata that relates positions in generated CSS to positions in the original Sass files. Browser tools can use it to show an authored source location while the browser continues to use the generated CSS.

## 2. Does the browser run Sass because a source map exists?

No. The browser loads and applies CSS. The source map helps developer tools navigate between generated CSS and its source; it does not change how the page renders.

## 3. Does Dart Sass generate source maps by default?

The Dart Sass CLI generates source maps by default when it writes CSS files. It can be disabled with <code>--no-source-map</code>. Build integrations may add their own source-map settings, so check the actual pipeline too.

## 4. Does the Sass JavaScript API generate maps by default?

No. The JavaScript API's <code>sourceMap</code> option defaults to <code>false</code>. Set <code>sourceMap: true</code> when compiling if the build needs a map.

## 5. What does a CSS source-mapping URL do?

It tells tools where to find the map associated with the CSS. If the URL is missing, points to the wrong location, or cannot be loaded, DevTools may show only generated CSS even when a map file exists elsewhere.

## 6. Where does Sass write the map file?

With CLI file output, Sass normally writes a map alongside the CSS and adds a reference to it. The exact output layout depends on the input and output paths. Check the generated CSS comment and filesystem instead of assuming a path.

## 7. What is the difference between embedding sources and embedding a source map?

Embedding sources places Sass source text inside a source map. Embedding the source map places the map data inside the CSS instead of in a separate map file. These are separate choices and can affect artifact size and source exposure.

## 8. What does --embed-sources do?

It includes the contents of source Sass files in the generated source map. This can help DevTools when it cannot retrieve original source files separately, but it increases map size and can disclose Sass source to anyone who can access the map.

## 9. What does --embed-source-map do?

It embeds source-map data in the generated CSS rather than writing a separate map file and referring to it. This is distinct from embedding source file contents inside the map.

## 10. What does --no-source-map do?

It tells the Dart Sass CLI not to generate a source map. Use it only when the project does not need Sass source navigation in that build. If a bundler generates the final CSS, configure maps at the bundler layer too.

## 11. How does DevTools find the original Sass file?

DevTools reads the map reference from the generated CSS, loads the map, and then uses the source paths and any embedded source contents to show authored files. The map, source paths, server, and browser settings must all line up.

## 12. How do I enable CSS source maps in Chrome?

Open DevTools Settings, go to Preferences → Sources, and enable CSS source maps. Then inspect an element and follow the source link beside a matching rule in Elements → Styles.

## 13. What is the Authored section in the Sources panel?

When source maps load, DevTools can group the original files separately from deployed CSS. The authored source tree helps you navigate to Sass files that contributed to the stylesheet.

## 14. What do Styles and Computed show?

Styles lists rules that match the selected element, including overridden declarations and links to their sources. Computed shows the final property values after the browser's cascade and other active conditions.

## 15. If DevTools opens a Sass source, does that mean its rule is applied?

No. A map indicates where generated code came from. A selector may not match the selected element, or its declaration may be overridden, disabled, inherited differently, or excluded by a media query or pseudo-class state.

## 16. Why does DevTools show generated CSS instead of Sass?

Common causes include disabled CSS source maps, a missing source-mapping URL, a map that cannot be fetched, invalid map data, or source paths that do not resolve. Check the browser console, network panel, and Developer Resources source-map status.

## 17. How can I check whether a map loaded?

In Chrome DevTools, open **More tools → Developer Resources** and inspect the source-map load status. Also check the Console and Network panels for failed requests or invalid map data.

## 18. Why might a map load but its Sass source not appear?

The map may contain source paths that the browser cannot fetch, or the map may omit embedded source contents. Confirm that the paths are correct relative to the map and that the web server exposes the source files, or deliberately include source contents when appropriate.

## 19. Why do mapped line numbers look wrong?

The CSS and map may come from different builds, a later transformation may have changed CSS without composing maps, or the browser may be using a stale cached artifact. Rebuild and deploy CSS and maps together, and ensure each later transform preserves or composes source maps.

## 20. What if PostCSS, a bundler, or a minifier runs after Sass?

The final tool should produce a map that traces from its output back through earlier transformations to Sass source. If a transform drops or fails to compose the map, DevTools may map to an intermediate file or the wrong location.

## 21. Why are absolute file URLs risky for shared builds?

An absolute <code>file:</code> URL points to a path on the machine that generated the artifact. It usually does not identify the same file on another developer's computer or a deployed website. Relative paths are often more portable when configured correctly.

## 22. Are production source maps safe to publish?

That depends on the application's deployment and source-disclosure policy. Maps can expose source text when embedded and may reveal filenames or directory structure even without embedded sources. Decide deliberately whether to publish them and restrict access if needed.

## 23. Does source-map generation affect CSS behavior?

No. Source maps are debugging metadata and do not alter the CSS cascade or runtime styling. They can add build artifacts or comments and may affect the size of a distributed package if maps are published.

## 24. Can I edit Sass in DevTools?

DevTools can be used for temporary experiments, but a displayed authored source is not the same as a committed project change. Make the final correction in the local Sass file, run the normal build, and reload to confirm the compiled output.

## 25. How do I debug a property with the wrong computed value?

Select the exact element, inspect the property in Styles, follow source links, and check Computed. Look for crossed-out declarations, specificity, source order, inheritance, media queries, pseudo-classes, and custom properties. Then verify the fix in the Sass source and rebuild.

## 26. Should I turn on source maps in production?

Choose based on debugging needs and source disclosure policy. A team may publish maps, keep them in a private error-monitoring service, or omit them from public artifacts. The choice is a deployment decision; Sass does not know which policy your application needs.

## 27. What should a Sass JavaScript build do with CompileResult.sourceMap?

When <code>sourceMap: true</code> is enabled, the compile result includes a source-map object. The build caller must write it to the intended location and add the correct source-mapping URL to the CSS. The Sass API leaves this output relationship to the caller.

## 28. What should I check when the browser uses stale mapped source?

Confirm that the server is returning the latest CSS and map, that the CSS references the intended map, and that caches are not serving older files. Rebuild and reload with the project’s normal cache-busting or development-server workflow.

## 29. Can source maps help with CSS inside a component framework?

Yes, if the framework or bundler emits and preserves CSS source maps, and the browser can load them. The Sass map must be composed through later transforms so final output still maps back to the intended Sass source.

## 30. What is a reliable source-map debugging workflow?

Enable CSS maps, inspect the target element, follow its rule source, compare Styles and Computed, check Developer Resources if the map is missing, verify paths and build freshness, make the fix in Sass, rebuild CSS and maps together, and reload to confirm the result.

## Official references

- [Dart Sass CLI: source maps](https://sass-lang.com/documentation/cli/dart-sass/#source-maps)
- [Sass JavaScript API: compile options](https://sass-lang.com/documentation/js-api/interfaces/options/)
- [Sass JavaScript API: CompileResult](https://sass-lang.com/documentation/js-api/interfaces/compileresult/)
- [Chrome DevTools: CSS source maps preferences](https://developer.chrome.com/docs/devtools/settings/preferences/)
- [Chrome DevTools: debug authored source with source maps](https://developer.chrome.com/docs/devtools/javascript/source-maps/)
- [Chrome DevTools: view and manually load source maps](https://developer.chrome.com/docs/devtools/developer-resources/)
- [Chrome DevTools: inspect CSS in Styles and Computed](https://developer.chrome.com/docs/devtools/css/reference/)
