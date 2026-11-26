# Browser support and progressive enhancement

Browsers do not all implement every web feature at the same time, and a supported feature can still have differences or bugs across versions. A robust Sass project begins with the experience users need, keeps important content available in a simple baseline, and adds newer styling when the browser can use it.

Sass is a build-time language. It organizes source and compiles it into CSS; the browser applies that generated CSS. Sass syntax such as variables and mixins does not, by itself, make a browser support a CSS feature or create a fallback for one.

## Learning goals

After this lesson, you should be able to:

- Define a browser support policy from audience and product requirements.
- Use compatibility data and Baseline as inputs to that policy.
- Write fallback CSS that remains useful when an enhancement is unsupported.
- Use <code>@supports</code> feature queries for CSS enhancements.
- Explain what a feature query can and cannot prove.
- Distinguish Sass compilation from browser compatibility transforms.
- Build and review a practical test matrix.

## 1. Define the support target

There is no universal list of browsers that every project must support. Start by identifying the browsers, versions, devices, and embedded web views your users actually rely on. Product requirements, analytics, support commitments, security policy, and organizational needs can all affect the target.

Write a short support policy that answers:

- Which browser families and operating systems are in scope?
- How many recent releases must work?
- Are embedded web views or managed enterprise browsers included?
- Which assistive technologies and input modes must be considered?
- Which features are essential, and which are optional enhancements?
- What is the expected behavior when a feature is missing?

Treat this as a product decision and review it when the user base or supported platforms change. Do not silently assume that every visitor has the newest desktop browser.

## 2. Use compatibility data as evidence

Check the documentation for each feature that matters. MDN pages include browser compatibility information for CSS properties, values, selectors, and at-rules. Check the exact part of a feature you plan to use: support for a property does not necessarily mean that every value or related behavior is supported.

Baseline is another useful summary of availability across a defined set of popular browsers. Its categories help describe whether a feature is widely available, newly available, or has limited availability. Baseline is a starting point for a support decision, not a guarantee that the feature works for every older browser, web view, assistive technology, device constraint, or user need.

For an important feature, record:

1. The exact property, value, selector, or API.
2. The browsers and minimum versions in your support policy.
3. The compatibility data and date you checked it.
4. Whether the feature is required for the content to work.
5. The fallback if the feature is missing.
6. How you will test both the fallback and enhanced paths.

This is more reliable than deciding from a browser name alone or copying a compatibility table from an old article.

## 3. Start with a useful baseline

Progressive enhancement means the core content and operation work with broadly supported technology, while newer features improve the experience where available. The baseline does not need to look identical in every browser, but it should remain understandable and usable.

For example, a panel can use a simple stacked layout first, then a grid enhancement:

~~~scss
.feature-list {
  display: flex;
  flex-direction: column;
}

.feature-list > * + * {
  margin-block-start: 1rem;
}

@supports (display: grid) {
  .feature-list {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }

  .feature-list > * + * {
    margin-block-start: 0;
  }
}
~~~

Browsers that support the declaration apply the grid rule. Browsers that do not recognize the feature query ignore the conditional block and keep the baseline layout. Each item remains available in document order either way.

A fallback does not have to imitate every detail of the enhanced layout. It needs to preserve content, meaning, and necessary operation.

## 4. Understand declaration fallbacks and the cascade

For a simple property or value, a previous declaration can act as a fallback. A browser that does not understand a declaration ignores it and continues parsing:

~~~scss
.banner {
  background-color: #312e81;
  background-image: linear-gradient(120deg, #312e81, #7c3aed);
}
~~~

A browser that does not support the gradient retains the solid background color. The fallback should appear first, with the enhancement after it.

This pattern is useful when a declaration can be safely ignored. It is not enough when the newer feature changes layout, requires several declarations, or needs a different interaction model. For those cases, keep a complete baseline and use a feature query around the enhancement.

Do not put a new declaration before its fallback. A later fallback would overwrite the supported enhancement in the normal cascade.

## 5. Use <code>@supports</code> to test CSS features

The <code>@supports</code> at-rule checks whether the browser recognizes a CSS declaration condition. Use a complete property-value pair:

~~~scss
.card-list {
  display: flex;
  flex-direction: column;
}

@supports (display: grid) {
  .card-list {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
  }
}
~~~

You can combine conditions with <code>and</code>, <code>or</code>, and <code>not</code>:

~~~scss
@supports (display: grid) and (grid-template-columns: subgrid) {
  .nested-layout {
    display: grid;
    grid-template-columns: subgrid;
  }
}
~~~

For a selector feature, use a selector query when appropriate:

~~~scss
@supports selector(.card:has(img)) {
  .card:has(img) {
    border-color: #7c3aed;
  }
}
~~~

A feature query is most useful when the browser can apply one clear enhanced set of rules and the baseline remains available outside it.

### A feature query has limits

<code>@supports</code> tests whether a browser accepts the declaration or selector syntax. It does not prove the feature is completely implemented, bug-free, or behaving as your design requires. It cannot test every partial implementation or browser defect.

Be precise about what you test. For example, <code>@supports (gap: 1rem)</code> says that the declaration is accepted. It may not establish that <code>gap</code> works in every layout context you plan to use. Check compatibility data and test the actual combination, such as grid gap versus flex gap, against your target browsers.

Do not use a feature query as a substitute for visual and functional testing.

## 6. Handle selector support carefully

An unsupported selector in a comma-separated selector list can cause the browser to discard the whole rule. Keep baseline selectors separate from advanced selectors:

~~~scss
.button,
.link-button {
  color: #312e81;
}

@supports selector(:focus-visible) {
  .button:focus-visible,
  .link-button:focus-visible {
    outline: 3px solid #7c3aed;
    outline-offset: 3px;
  }
}
~~~

The baseline styling still applies where the more specific selector is not supported. A feature query can guard the enhanced focus treatment, but always preserve a visible and useful focus indicator in the target browsers.

Selector lists have nuanced parsing behavior, and some newer functions are intentionally designed to tolerate unsupported selectors inside them. Do not assume all selector lists fail or recover in the same way; check the syntax and compatibility of the exact selector you use.

## 7. Keep Sass and browser support separate

Sass variables, maps, mixins, and modules are evaluated while compiling. Sass can emit an <code>@supports</code> rule, but the browser decides whether the condition matches when it loads the CSS.

A Sass variable can organize a build-time decision:

~~~scss
$enable-grid-enhancement: true;

@if $enable-grid-enhancement {
  @supports (display: grid) {
    .feature-list {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
    }
  }
}
~~~

This Sass condition decides whether the query is included in the output at all. It does not detect the user's browser during compilation. The <code>@supports</code> inside the emitted CSS is what checks the browser.

Likewise, Sass does not automatically translate every new CSS feature into an older equivalent. If a project needs syntax transformation or vendor prefixing, configure the relevant build tool against the project's browser target. Inspect the generated CSS to confirm what the tool actually emitted. Avoid copying old prefixes by hand without a current compatibility reason.

## 8. Use JavaScript feature detection for JavaScript features

CSS support should usually be detected with CSS feature queries. For a JavaScript API or a JavaScript-dependent behavior, feature-detect the API or capability before relying on it.

For example, the browser exposes <code>CSS.supports()</code> to query a CSS declaration from JavaScript:

~~~js
if (
  typeof CSS !== "undefined" &&
  typeof CSS.supports === "function" &&
  CSS.supports("display", "grid")
) {
  document.documentElement.classList.add("supports-grid");
}
~~~

For browser APIs, check for the API or method your code needs, then provide a meaningful fallback. Do not infer capabilities by parsing the user-agent string unless a narrowly justified platform-specific workaround requires it. User-agent strings can be incomplete, changed, or shared across browsers that have different capabilities.

Keep the no-JavaScript experience in mind too. If the page's core content is only inserted after a script runs, CSS compatibility alone will not make that content available.

## 9. Avoid brittle prefix and browser-name strategies

Vendor-prefixed CSS exists in legacy stylesheets and for some special cases, but prefixes are not a general compatibility plan. Prefer standards-based CSS, compatibility data, and an explicit target policy. If a current target genuinely requires a prefix, make the build process reproducible and keep the standard declaration after the prefixed declaration where both are needed.

Avoid rules that say “if browser X, use this layout” when the real question is whether a capability is available. Feature detection is usually more direct: it tests the capability the style depends on instead of guessing from browser identity.

## 10. Test realistic browsers and modes

It is not practical to test every possible browser, release, device, operating system, and assistive technology combination. Select a test set based on the support policy and risk.

A practical review may include:

- Current stable versions of the major browsers in scope.
- The oldest supported version or a representative older environment.
- Mobile Safari and mobile Chrome when mobile browsers are in scope.
- Any required embedded web view or managed browser.
- Keyboard-only operation and visible focus.
- A screen reader for important navigation and content flows.
- Narrow and wide viewports, zoom, and text enlargement.
- The fallback path with an enhancement disabled or unavailable.
- The enhanced path in a browser that supports the feature.

Prioritize critical user journeys, forms, navigation, dialogs, and layouts where unsupported CSS could hide or overlap content. When a visual difference is harmless, document the intended fallback instead of spending effort to make every pixel identical.

## 11. Inspect the CSS the browser receives

The browser does not receive Sass variables or mixin calls. It receives compiled CSS, and production tooling may transform or bundle that output further.

When investigating a compatibility issue:

1. Reproduce it in a browser included in the support policy.
2. Inspect the loaded stylesheet and computed styles in developer tools.
3. Confirm the Sass build ran and that the intended CSS was emitted.
4. Check whether the relevant condition matched.
5. Compare the feature against current compatibility information.
6. Verify that the fallback rule is present and not overridden.
7. Reduce the issue to the smallest relevant selector or declaration.

This separates a Sass compilation problem, a build pipeline problem, a cascade problem, and a browser capability or implementation problem.

## 12. Practice task

Create a feature card layout with a stacked baseline and a grid enhancement behind <code>@supports (display: grid)</code>. Add a decorative gradient with a solid-color fallback. Then add one newer selector behind a selector feature query.

Write a short browser support policy for the sample. Identify the exact feature conditions, decide what remains usable without them, and create a small test matrix. Review both the baseline and enhanced layout in the target browsers. Finally, inspect the generated CSS and confirm that Sass compilation did not silently remove or rewrite a rule you depend on.

## Summary

Compatibility work begins with a user-focused support policy. Use compatibility data and Baseline as evidence, not as a replacement for testing. Keep important content and operation in a broadly supported baseline, then use <code>@supports</code> to add CSS enhancements where appropriate.

Feature queries test whether syntax is accepted; they cannot guarantee a bug-free implementation. Sass organizes and compiles styles, while browser support decisions happen in the browser and any compatibility transforms happen in separate build steps. Test the compiled output and the user journeys that matter.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="051-accessible-motion-and-responsive-patterns.md">051 - Accessible motion and responsive patterns</a></td><td><a href="../qnas/052-browser-support-and-progressive-enhancement.md">Q&amp;A</a></td><td><a href="../examples/052-browser-support-and-progressive-enhancement.md">Example</a></td><td><a href="053-capstone-build-a-reusable-scss-design-system.md">053 - Capstone: build a reusable SCSS design system</a></td></tr>
  </tbody>
</table>
