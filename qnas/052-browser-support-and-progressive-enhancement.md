# Browser support and progressive enhancement — Q&A

These answers cover browser targeting, CSS fallbacks, feature queries, and the role of Sass in a compatibility workflow. Browser support changes as browsers release updates, so check current compatibility information for the exact feature and target versions.

## 1. What does browser support mean?

Browser support describes whether a browser version recognizes and implements a web feature. It can vary by property, value, selector, API, platform, and implementation details. “Supports CSS Grid” is less precise than checking the particular Grid behavior the layout uses.

## 2. What is a browser support policy?

It is a project decision that names the browsers, versions, devices, and environments the product will support, plus what the product should do when a feature is missing. Base the policy on audience data and product requirements, and review it when those change.

## 3. Do all projects need to support every old browser?

No universal browser list fits every product. A public service, an internal enterprise tool, and a personal portfolio can have different user needs and support commitments. Decide explicitly and document the acceptable fallback for browsers outside the preferred target.

## 4. What is progressive enhancement?

Progressive enhancement starts with core content and operation that work using a broadly supported baseline. It then adds styling or behavior where the browser supports it. The fallback may look simpler, but the important information and actions remain available.

## 5. Does progressive enhancement mean the page must look identical everywhere?

No. A fallback can use a simpler layout or decoration. It should preserve meaning, important content, and required operation. Avoid making the only copy of information, form control, or navigation depend on an optional CSS enhancement.

## 6. What is Baseline?

Baseline summarizes the availability of web platform features across a defined set of popular browsers. Its categories help show whether a feature is widely available, newly available, or has limited availability. It is useful evidence for a support decision, but it is not a substitute for checking your users, older releases, web views, accessibility, or the actual implementation.

## 7. If a feature is Baseline widely available, can I skip testing?

No. Baseline does not guarantee that a feature works in every browser release or every environment, nor that the feature is accessible or appropriate for your design. Test important user journeys and any combinations that matter, especially if the feature is essential.

## 8. What does <code>@supports</code> test?

A CSS feature query tests whether the browser accepts a CSS declaration or supported selector condition. For example, <code>@supports (display: grid)</code> checks the declaration condition for Grid. It lets CSS apply an enhancement conditionally without using JavaScript.

## 9. Can <code>@supports</code> prove a feature is bug-free?

No. It tests whether a browser recognizes the condition, not whether every part of the feature is correct, complete, or free from bugs. It cannot detect all partial implementations or browser defects. Check compatibility data and test the exact behavior in real target browsers.

## 10. What is the difference between <code>@media</code> and <code>@supports</code>?

A media query tests the rendering environment, such as viewport size, orientation, or a user preference. A feature query tests whether the browser supports a CSS feature. A responsive layout may use both: a media query for available viewport width and a feature query for an optional layout capability.

## 11. Why should a fallback appear before an enhancement?

CSS declarations later in the cascade can override earlier ones when both are valid. Put a broadly supported value first and the enhancement after it:

~~~scss
.hero {
  background-color: #312e81;
  background-image: linear-gradient(120deg, #312e81, #7c3aed);
}
~~~

A browser that rejects the gradient declaration keeps the solid background.

## 12. When is a declaration fallback enough?

It works when an unsupported declaration can simply be ignored without breaking the rest of the layout. A solid color before a gradient is a simple example. If a newer feature changes structure or requires several coordinated rules, provide a complete baseline and guard the enhanced rules with a feature query.

## 13. Can I put a fallback and enhancement in one selector?

Yes. A common pattern is to write the fallback first and the enhanced declaration afterward. For a group of related layout rules, it is usually easier to review if the baseline and enhanced block are clearly separated.

## 14. Is this a good Grid fallback?

~~~scss
.feature-list {
  display: flex;
  flex-direction: column;
}

@supports (display: grid) {
  .feature-list {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
~~~

It can be a useful baseline and enhancement, but check every declaration in the baseline too. For example, older browser versions may support Flexbox but not <code>gap</code> in Flexbox. If spacing is required, use another baseline spacing method or limit the support target accordingly.

## 15. What is a selector feature query?

It asks whether the browser accepts a selector syntax, for example <code>@supports selector(:focus-visible)</code>. It can help guard an enhancement that uses a newer selector. Keep the baseline selector and visible focus behavior outside the query so unsupported syntax does not remove the only useful styling.

## 16. Can one unsupported selector break a comma-separated selector list?

In some traditional selector lists, an invalid selector can cause the entire rule to be discarded. Selector parsing has evolved, and some newer functions have forgiving selector-list behavior, so the exact syntax matters. When uncertain, separate the baseline from the advanced selector or guard the advanced rule with a feature query.

## 17. Does Sass make modern CSS work in older browsers?

No. Sass compiles Sass source into CSS. It does not make a browser understand a CSS feature or create the design fallback automatically. Write fallback CSS, use feature queries, and configure separate build tools for transformations if the support policy requires them.

## 18. Does Sass decide whether the user's browser supports a feature?

No. Sass runs during the build and emits CSS. A Sass variable or <code>@if</code> chooses what source gets emitted at build time. The browser evaluates a compiled CSS <code>@supports</code> query later when it loads the stylesheet.

## 19. Does Sass automatically add vendor prefixes?

Do not assume it does. Sass compiles Sass syntax and expressions; prefixing is a separate compatibility transformation when a project needs it. If the support target requires prefixes, configure the build tooling and browser target deliberately, then inspect the emitted CSS.

## 20. Should I copy every vendor-prefixed property from an old code sample?

No. Prefixes are not a complete compatibility strategy and many historical prefixes are obsolete. Check current compatibility information first. If a prefix is genuinely needed for your target, keep the standard declaration too and prefer a reproducible build rule over undocumented manual copies.

## 21. Should I detect a browser by its name or user-agent string?

Usually not. Browser names do not directly describe the capability a stylesheet needs, and user-agent strings can be changed or shared. Prefer a CSS feature query for CSS support and feature detection for a JavaScript API. Use a browser-specific workaround only when a known implementation bug justifies it.

## 22. How should I feature-detect a JavaScript API?

Check for the API or method that your code will call, then provide a useful fallback. Do not call a feature-detection method before checking that the method exists in the target environment:

~~~js
const supportsGrid =
  typeof CSS !== "undefined" &&
  typeof CSS.supports === "function" &&
  CSS.supports("display", "grid");
~~~

For non-CSS APIs, check the relevant property or method rather than inferring support from the browser brand.

## 23. What should I do if an unsupported enhancement is essential?

If the feature is required for the content or task, a decorative fallback is not enough. Choose a solution that works in the declared support target, add an appropriate polyfill or alternate implementation if suitable, or revise the support policy transparently. Do not ship an essential interaction that silently disappears.

## 24. How do I choose browsers for a test matrix?

Start with the support policy and audience. Include representative current browsers and platforms in scope, plus the oldest supported version or a suitable older environment. Add mobile browsers, embedded web views, assistive technologies, or managed browsers when users rely on them. Prioritize high-risk flows instead of trying to test every imaginable combination.

## 25. What paths should a compatibility review test?

Review both the fallback and enhanced path. Verify that content and controls remain available, no layout overflows, focus is visible, and forms and navigation still work. Also test realistic viewport sizes, zoom, text enlargement, and the input modes required by the product.

## 26. What should I inspect when the Sass looks right but the browser does not?

Inspect the CSS the browser actually loaded. Confirm the compiler emitted the rule, any post-processing step preserved or transformed it as expected, the feature query matches, and the cascade did not override the declaration. Then check the browser compatibility data and reproduce the behavior in the target environment.

## 27. Does a working fallback remove the need to keep the CSS maintainable?

No. Keep the baseline and enhancements close enough to review together, use Sass modules and mixins where they clarify shared policy, and inspect the compiled CSS when abstractions become complex. A compatibility workaround should have a reason, a target, and a way to remove or review it later.

## References

- [MDN: supporting older browsers](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Supporting_Older_Browsers)
- [MDN: using feature queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Conditional_rules/Using_feature_queries)
- [MDN: <code>@supports</code> at-rule](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/%40supports)
- [MDN: CSS error handling and vendor prefixes](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Syntax/Error_handling)
- [MDN: Baseline compatibility](https://developer.mozilla.org/en-US/docs/Glossary/Baseline/Compatibility)
- [MDN: implementing feature detection](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing/Feature_detection)
- [MDN: cross-browser testing](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing/Introduction)
- [Sass: CSS at-rules](https://sass-lang.com/documentation/at-rules/css/)
- [Sass: Dart Sass CLI](https://sass-lang.com/documentation/cli/)
- [Next: Capstone — build a reusable SCSS design system](../lessons/053-capstone-build-a-reusable-scss-design-system.md)
