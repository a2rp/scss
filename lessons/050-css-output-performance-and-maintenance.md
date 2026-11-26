# CSS output, performance, and maintenance

Sass runs during a build and produces CSS. The browser downloads and applies that CSS, so runtime style performance depends on the final CSS and how the page loads it—not on whether the source was written in Sass or plain CSS.

A good stylesheet workflow keeps generated output intentional, measures what the page actually downloads and uses, and removes duplication without making the source harder to maintain.

## Learning goals

After this lesson, you should be able to:

- Explain how Sass source structure affects generated CSS.
- Choose expanded or compressed output for the right stage of a build.
- Measure CSS bytes, requests, blocking behavior, and actual usage.
- Use Coverage as evidence without deleting styles that other routes or states need.
- Avoid duplicate module output and overly broad entry points.
- Balance code splitting, critical CSS, and maintainability.
- Keep a repeatable output and performance review process.

## 1. Separate build-time work from browser work

Sass parsing, evaluation, and compilation happen before the browser uses the stylesheet. Sass build time can affect local feedback and deployment duration. The browser's styling work depends on the CSS it receives and the page it applies it to.

This distinction helps diagnose the correct layer:

- Slow Sass compilation: inspect the build pipeline, dependency graph, and compiler workload.
- Large CSS downloads: inspect generated output, delivery, compression, and which stylesheets are loaded.
- Slow page rendering: inspect render-blocking resources, network timing, and browser performance traces.
- Incorrect appearance: inspect the DOM, cascade, and computed styles.

Changing Sass syntax alone does not make the browser render faster if it produces the same CSS and the same delivery behavior.

## 2. Choose an output style for the build stage

Dart Sass supports <code>expanded</code> and <code>compressed</code> output styles:

- <code>expanded</code> puts selectors and declarations on readable lines. It is the CLI default and is useful during development and review.
- <code>compressed</code> removes extra characters where possible and creates compact output for distribution.

Compile readable CSS:

~~~sh
sass --style=expanded src/main.scss dist/main.css
~~~

Compile compact CSS:

~~~sh
sass --style=compressed src/main.scss dist/main.css
~~~

Sass compression is one part of a delivery pipeline. It does not replace HTTP compression such as Brotli or gzip, and it does not remove CSS merely because a page did not use it. Compare the final transfer size in the environment users receive, not only the size of the Sass source or a locally generated file.

If a bundler or CSS minifier already controls production output, use one clear owner for final CSS optimization rather than running conflicting transformations without measuring the result.

## 3. Know what creates CSS output

Sass variables, functions, and mixins do not automatically create selectors just because they are defined. CSS is emitted by style rules and by loaded modules that contain CSS.

For example, this module only defines a value:

~~~scss
// _tokens.scss
$color-brand: #2457d6;
~~~

Loading it provides the variable without producing a selector:

~~~scss
@use "tokens" as tokens;

.button {
  color: tokens.$color-brand;
}
~~~

A module that contains top-level style rules contributes those rules when loaded. Keep token and helper modules free of incidental CSS when consumers should be able to load them without output.

The module system loads a given module once within a Sass compilation. This prevents the same module's CSS from being re-emitted just because several modules use it. Separate entry points are separate compilations, though: if each output includes the same global CSS and a page loads all those outputs, the browser can still receive duplicates across files.

## 4. Prefer the module system to repeated Sass imports

Legacy Sass <code>@import</code> can evaluate a stylesheet repeatedly and emit its CSS repeatedly when it is imported from multiple places. Sass <code>@import</code> is deprecated; migrate Sass modules to <code>@use</code> and <code>@forward</code>.

The module system also makes dependencies explicit:

~~~scss
// _button.scss
@use "tokens" as tokens;

.button {
  color: tokens.$color-brand;
}
~~~

A Sass <code>@use</code> is handled during compilation. A plain CSS <code>@import</code> can remain in generated CSS for the browser to load. Do not confuse the two: browser CSS imports can create additional network dependencies, while Sass module loading combines Sass modules into compiled output.

## 5. Make entry points intentional

An entry point is a stylesheet that the build compiles into a CSS file for an application, page group, or theme. A small number of intentional entry points can make ownership and loading clear.

For example:

~~~text
src/
  styles/
    app.scss
    admin.scss
    _tokens.scss
    _reset.scss
    components/
      _button.scss
      _dialog.scss
      _navigation.scss
~~~

The Sass CLI skips partials whose names begin with an underscore when compiling a directory. Entry points can use modules to assemble their intended styles:

~~~scss
// app.scss
@use "reset";
@use "components/button";
@use "components/navigation";
~~~

~~~scss
// admin.scss
@use "reset";
@use "components/button";
@use "components/dialog";
~~~

If a page loads both output files, shared CSS such as the reset and button styles may be duplicated across the two separate builds. Decide whether pages should load one entry point, share a separately loaded base file, or use a bundler that extracts common CSS. Measure the result and keep cascade order clear.

Do not split every component into a separate stylesheet by default. More files can reduce unused bytes for some routes, but can also add requests, complexity, and ordering problems. Let the architecture and measurements guide the split.

## 6. Measure the stylesheet users receive

Before optimizing, establish what the browser downloads:

- List stylesheet URLs loaded by the page.
- Check request count and resource timing in the Network panel.
- Compare uncompressed file size with compressed transfer size.
- Check response headers and cache behavior.
- Inspect the page's render-blocking resources.
- Record relevant page-load metrics before and after a change.

A stylesheet linked in the document head is generally render-blocking while the browser downloads and processes it. Removing unused CSS, reducing unnecessary requests, and ensuring cacheable assets can help the critical path. Avoid adding complex asynchronous-loading patterns without checking for flash of unstyled content and accessibility issues.

Inlining critical CSS is an advanced technique. It can reduce a blocking fetch for a small set of essential styles, but it can duplicate rules, complicate caching, and create visual bugs if the extracted subset is incomplete. Use tooling and verify the rendered result if the project needs it.

## 7. Use Coverage to find candidates, not automatic deletions

Chrome DevTools Coverage can report used and unused CSS for the page load and interactions recorded in a session:

1. Open DevTools and the Command Menu.
2. Search for **Show Coverage**.
3. Start coverage and reload the page.
4. Exercise the page and its important interactions.
5. Inspect the CSS resource and line-level usage report.
6. Map candidate rules back to source and investigate before removing them.

Coverage is a snapshot of the URLs, routes, viewport, and interactions exercised during the recording. A rule may appear unused because it is needed on another route, at another breakpoint, after a user action, or in a state the session did not visit.

Use the report to create a review list. Search the application for selector usage, check routes and component states, and verify the result after removal. Never delete all red-marked CSS solely on the basis of one visit.

## 8. Avoid output growth from repeated component emission

A mixin that emits selectors generates CSS each time it is included:

~~~scss
@mixin button-base {
  padding: 0.75rem 1rem;
  border-radius: 0.375rem;
}

.primary-button {
  @include button-base;
}

.secondary-button {
  @include button-base;
}
~~~

This is often appropriate because each selector needs declarations in the output. But including a large component stylesheet through multiple entry points can create duplicate CSS. Check the compiled output when adding mixins, CSS-generating modules, or repeated includes.

Keep reusable Sass-only helpers separate from modules that emit global or component CSS. A clear boundary makes it easier to predict the output of loading a module.

## 9. Keep selectors and nesting maintainable

The browser ultimately receives expanded selectors. Deep Sass nesting can generate long, specific selectors that are harder to override and maintain. It is not automatically a performance problem; inspect its generated CSS and focus on real complexity and cascade behavior.

Prefer a direct component selector:

~~~scss
.search-panel {
  padding: 1rem;

  &__input {
    width: 100%;
  }
}
~~~

Avoid tying a component to every ancestor in a page:

~~~scss
.page {
  .content {
    .sidebar {
      .search-panel {
        .search-panel__input {
          width: 100%;
        }
      }
    }
  }
}
~~~

Keep the component's intended API stable and avoid unnecessary global dependencies. Use browser profiling when there is evidence of costly style recalculation rather than relying on selector folklore.

## 10. Be careful with compressed output and debugging

Compressed output is harder to inspect manually. Preserve readable Sass source maps and development output where they help the team, and generate the production artifact through the configured build pipeline.

If production source maps are enabled, decide whether they should be public. A map can reveal source paths, and a map with embedded source content contains the original Sass. Follow the project's deployment policy.

Do not edit generated CSS as the permanent fix. Change the source Sass, rebuild, and verify the generated file and page again.

## 11. Treat CSS layering and order as part of output

Combining stylesheets can change source order. Code splitting can also change which CSS arrives first. Even if all declarations are present, the result may differ because of the cascade, layers, specificity, or media conditions.

Before combining, splitting, or reordering CSS:

- Check stylesheet link order and generated order.
- Review cascade layer declarations if the project uses layers.
- Check duplicate declarations and overrides.
- Test responsive and interaction states.
- Confirm that lazy-loaded styles do not arrive after a visible component flashes incorrectly.

Output optimization should preserve behavior, not only reduce bytes.

## 12. A practical CSS output and performance workflow

1. Identify the page, route, device, and state that need investigation.
2. Inspect the final CSS files the browser downloads.
3. Record file sizes, transfer sizes, requests, and render-blocking timing.
4. Use Coverage while exercising representative routes and interactions.
5. Trace candidate rules to Sass source and understand why they exist.
6. Check whether modules, mixins, or separate entry points emit duplicates.
7. Make one targeted source or delivery change.
8. Rebuild with the production configuration.
9. Compare the same network and performance measurements.
10. Verify visual behavior, cascade order, and all relevant page states.
11. Document the build and loading boundary so future changes preserve it.

## Key takeaways

- Sass affects browser performance through the CSS it emits and how that CSS is delivered.
- Use readable expanded output for development and compressed output when appropriate for production; measure final transfer size.
- <code>@use</code> loads a module once per compilation, while repeated legacy Sass <code>@import</code> can duplicate output.
- Multiple independent entry points can still duplicate shared CSS if a page loads all of them.
- Coverage reveals only what was used during the recorded session; verify other routes and states before removing code.
- Optimize based on measured output and behavior, and re-check the cascade after changing CSS order or delivery.

## Official references

- [Dart Sass CLI: output styles and compilation](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass: @use and module output](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: @import and migration](https://sass-lang.com/documentation/at-rules/import/)
- [Chrome DevTools: Coverage](https://developer.chrome.com/docs/devtools/coverage/)
- [Chrome DevTools: CSS inspection and Coverage](https://developer.chrome.com/docs/devtools/css/reference/)
- [web.dev: the critical rendering path](https://web.dev/learn/performance/understanding-the-critical-path)
- [Chrome Lighthouse: render-blocking resources](https://developer.chrome.com/docs/lighthouse/performance/render-blocking/)

## Next lesson

Continue with [Accessible motion and responsive patterns](051-accessible-motion-and-responsive-patterns.md).
