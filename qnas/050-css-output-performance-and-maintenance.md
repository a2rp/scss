# CSS output, performance, and maintenance — Q&A

This Q&A reviews how Sass source becomes browser-delivered CSS and how to measure output before changing it. Browser behavior and tooling evolve, so verify details against the build and browser versions your project uses.

## 1. Does Sass itself run in the browser?

No. Sass is compiled into CSS during a build. The browser receives and applies the generated CSS. Sass compile time can affect builds and developer feedback, while browser performance depends on the resulting CSS and how the page loads it.

## 2. Can Sass source be slow while the browser CSS is fast?

Yes. A large or expensive Sass build can slow development or deployment even if it produces a small stylesheet. Measure compilation and page loading separately.

## 3. What does Dart Sass --style=compressed do?

It removes extra characters where possible and writes compact CSS. It does not remove selectors because a page did not use them, and it does not replace network compression or performance measurement.

## 4. When should I use expanded output?

Expanded output is readable and is the Dart Sass CLI default. It is useful for local development, inspection, and debugging. A production pipeline may generate compressed output or apply its own CSS optimization.

## 5. Does compressed CSS replace gzip or Brotli?

No. Sass compressed output changes the generated CSS text. HTTP compression is a separate delivery layer. Compare the compressed transfer size reported by the server or browser, not only the local file size.

## 6. Does smaller CSS always make a page faster?

Not automatically. Smaller transfers can help, but the effect depends on caching, network conditions, request count, render-blocking resources, page structure, and how much of the CSS is actually used. Measure the same page and conditions before and after changes.

## 7. Does Sass know which selectors the HTML uses?

Generally no. Sass compiles stylesheets; it does not inspect every application route and runtime state to remove selectors that the page does not use. Use coverage or build tooling to investigate unused CSS.

## 8. Does defining a variable or mixin emit CSS?

No, not by itself. Sass emits CSS from style rules and loaded modules that contain CSS. Including a CSS-emitting mixin can generate declarations or rules at each include site.

## 9. Does @use include module CSS more than once?

Within a Sass compilation, a module loaded with <code>@use</code> is only loaded once, and its CSS is included once even if several modules use it. Separate Sass entry points are separate compilations, so a page can still receive duplicate shared CSS from multiple output files.

## 10. Can @use still produce unwanted CSS?

Yes. Loading a module with top-level style rules includes those rules. Keep token and helper modules free of incidental CSS when they should be safe to load without selectors in the output.

## 11. Can mixins create duplicated CSS?

Yes. CSS emitted by a mixin is generated where it is included. Repeated includes may be intentional, but large repeated component output or loading the same component through multiple entry points can increase the delivered CSS. Inspect the compiled output.

## 12. Why is repeated Sass @import a maintenance and size concern?

Legacy Sass <code>@import</code> makes members global and can evaluate and emit a stylesheet each time it is imported. Repeated imports of CSS-emitting files can duplicate output. Sass <code>@import</code> is deprecated; migrate Sass modules to <code>@use</code> and <code>@forward</code>.

## 13. Is Sass @import the same as CSS @import?

No. Sass can resolve Sass module imports during compilation. A plain CSS <code>@import</code> may remain in CSS for the browser to load. Browser CSS imports can add network dependencies and affect when styles are available.

## 14. Should every component have its own CSS file?

No. Splitting can reduce unused CSS for some routes, but it can also add requests, complicate loading, duplicate shared rules, and make cascade order harder to reason about. Choose entry points based on page architecture and measured results.

## 15. What is a Sass entry point?

An entry point is a Sass stylesheet that the build compiles into a CSS artifact used by a page, application, theme, or route group. It loads the modules and styles intended for that output.

## 16. Why can separate entry points duplicate styles?

Each entry point is compiled independently. If both include the same CSS-emitting reset or component module and the page loads both outputs, that shared CSS can be present in both files even though each Sass compilation loaded its module only once.

## 17. What is a partial?

A Sass partial conventionally has a filename beginning with an underscore, such as <code>_tokens.scss</code>. It is intended to be loaded by another Sass file. The Dart Sass CLI ignores partial files when compiling a whole directory, so they do not become standalone CSS outputs by default.

## 18. What is render-blocking CSS?

Stylesheets are render-blocking by default when the browser needs to download and process them before rendering the page's content with its styles. Reduce unnecessary CSS and requests on the critical path, but verify the visual result when changing load behavior.

## 19. Should I always inline critical CSS?

No. Inlining is an advanced technique. It may reduce a blocking fetch for essential styles, but it can increase HTML size, duplicate cached rules, and cause a flash of unstyled or incorrectly styled content if extraction is incomplete. Use it only with a measured need and careful verification.

## 20. What does Chrome Coverage show?

Coverage records which portions of loaded CSS and JavaScript were used during a page load and the interactions recorded in that session. It provides a report and line-level view that can help identify candidates for further investigation.

## 21. Does a red or unused Coverage line mean I can delete it?

No. It means the code was not observed as used in that recording. It may be needed on another route, viewport, theme, interaction state, or dynamically rendered component. Test representative conditions and search for usage before removing it.

## 22. How can I make Coverage results more useful?

Record relevant routes and states, reload with instrumentation, and interact with the page. Exercise responsive layouts, menus, dialogs, focus, themes, and lazy-loaded UI that matter to the application. Coverage is still evidence from a set of observed sessions, not a proof for all users.

## 23. What should I measure before optimizing CSS?

Record the stylesheet URLs, request count, transfer size, cache behavior, render-blocking timing, and relevant page performance metrics. Use the same route, device, network conditions, and test steps for the comparison.

## 24. Should I optimize selector performance by flattening every selector?

Not by default. Deep or highly specific selectors can make the cascade harder to maintain, but do not assume every nested selector is a runtime bottleneck. Inspect generated CSS and use browser profiling when there is evidence of costly style recalculation.

## 25. How can I keep output from growing over time?

Keep entry points intentional, separate Sass-only helpers from CSS-emitting modules, review repeated mixin inclusion, migrate repeated legacy imports, use coverage to find candidates, and compare production output across changes.

## 26. Is source-map size part of production CSS size?

A separate source map is a separate file and is not the CSS transfer size unless it is embedded in the CSS. Maps can still be included in a deployment and may expose source paths or source contents. Decide whether to publish them as part of the artifact policy.

## 27. What if a CSS change makes the page look different after splitting files?

Check link order, cascade layers, specificity, media conditions, and whether a stylesheet now loads later or not at all. Code splitting and concatenation can change the order and availability of rules, so verify all affected states.

## 28. Should I remove unused CSS from a third-party library directly?

Usually avoid editing generated or vendored files by hand. Prefer importing only supported components, configuring the library, using build-time extraction when appropriate, or updating the dependency. Keep license and update behavior in mind.

## 29. How do I compare Sass outputs?

Compile before and after with the same version and production options. Compare generated selectors and declarations, raw file size, transferred size in the browser, and the page's load and visual behavior. A smaller file is not useful if it removes required states or changes the cascade.

## 30. What is a safe CSS optimization loop?

Measure the real page, inspect its downloaded CSS, use Coverage across representative routes and states, trace candidate rules back to source, make one targeted change, rebuild production output, compare the same metrics, and verify visual and interaction behavior.

## Official references

- [Dart Sass CLI: output styles and compilation](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass: @use and module output](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: @import and migration](https://sass-lang.com/documentation/at-rules/import/)
- [Chrome DevTools: Coverage](https://developer.chrome.com/docs/devtools/coverage/)
- [Chrome DevTools: CSS inspection and Coverage](https://developer.chrome.com/docs/devtools/css/reference/)
- [web.dev: the critical rendering path](https://web.dev/learn/performance/understanding-the-critical-path)
- [Chrome Lighthouse: render-blocking resources](https://developer.chrome.com/docs/lighthouse/performance/render-blocking/)
