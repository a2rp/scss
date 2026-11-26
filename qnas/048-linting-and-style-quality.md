# Linting and stylesheet quality - Q&A

This Q&A explains how to build a practical SCSS quality workflow with Stylelint, Prettier, Dart Sass, and browser inspection. Tool support can change across releases, so use compatible package versions and check the documentation for the versions pinned by your project.

## 1. What is linting?

Linting checks source files against a parser and a set of configured rules. For stylesheets, rules can identify syntax errors, suspicious constructs, or conventions that the project wants to keep consistent.

## 2. What does Stylelint do?

Stylelint parses stylesheet source and reports problems from its enabled rules. With a SCSS-aware syntax and configuration, it can lint SCSS source and apply some automatic fixes.

## 3. Does Stylelint compile SCSS?

No. Stylelint parses and checks source; it is not the Sass compiler. Run Dart Sass separately to verify that Sass compiles and produces CSS.

## 4. What configuration should I start with for SCSS?

The Stylelint project recommends the SCSS community configuration <code>stylelint-config-standard-scss</code>. It includes SCSS syntax support and SCSS-specific rules, giving a project a baseline that can be customized.

## 5. Do I need to configure a parser when using the SCSS standard config?

Usually the shared config provides the SCSS parser and related rules. Check its documentation for the versions you install. If you configure Stylelint manually or lint another syntax, you may need to specify the appropriate custom syntax.

## 6. Does an SCSS parser also parse indented .sass files?

Do not assume so. <code>.scss</code> uses braces and semicolons; indented <code>.sass</code> has a different syntax. Configure a parser such as <code>postcss-sass</code> for those file patterns, or use a shared config designed for that syntax.

## 7. How do I run Stylelint on SCSS files?

Use a quoted glob, for example:

~~~sh
npx stylelint "src/**/*.scss"
~~~

Update the path to match the project's source layout. Quoting the glob lets Stylelint handle file matching consistently across shells.

## 8. Why should I quote a glob?

Shells handle wildcard expansion differently. Quoting the pattern passes it to Stylelint so its CLI can resolve files according to the tool's glob behavior.

## 9. What is the difference between Stylelint and Prettier?

Stylelint checks configured code-quality and convention rules, and it can fix some violations. Prettier formats supported source into a consistent layout. Formatting cleanly does not prove that code is correct, and linting does not replace formatting.

## 10. Does a successful lint prove the Sass build works?

No. A linter and a compiler perform different checks. Run Dart Sass as part of the workflow to catch Sass compilation errors and deprecation messages.

## 11. Does passing lint prove the page looks right?

No. A linter cannot fully evaluate the rendered page, the cascade, loaded assets, user interactions, or all browser states. Inspect the result in the browser and review the design behavior.

## 12. What does Prettier's printWidth option do?

It guides how Prettier wraps code. It is not a strict maximum line length. Choose a value that suits the project and use the same configuration in editors and CI.

## 13. Should I configure both Stylelint and Prettier?

Yes, when the project wants both rule checking and consistent formatting. Keep their responsibilities clear and avoid rule configurations that fight the chosen formatter.

## 14. What files should Stylelint lint?

Lint authored stylesheet source that the project maintains, such as SCSS files under <code>src</code>. Include plain CSS only if it shares a compatible configuration. Use a separate syntax configuration for indented Sass or other stylesheet languages.

## 15. Should generated CSS be linted with the SCSS source command?

Usually keep generated output out of the source lint glob. Generated files can be huge and are produced from source that should be checked before compilation. If a project has a separate reason to lint generated CSS, give it an explicit command and configuration.

## 16. What is .stylelintignore for?

It lists paths Stylelint should skip, such as generated bundles, vendor code, or coverage output. Keep ignore rules narrow so maintained source does not accidentally fall outside linting.

## 17. Does ignoring a legacy folder fix its lint problems?

No. It only hides those files from the command. For a large existing codebase, use a planned migration or baseline strategy instead of excluding authored source indefinitely.

## 18. What does Stylelint --fix do?

It applies fixes for violations where a rule supports a safe automatic fix. It does not fix every lint problem or determine whether the stylesheet's behavior is correct.

## 19. Should I review files after running --fix?

Yes. Review the complete diff, especially when many files changed. If the fix is broad or mechanical, keeping that cleanup separate from feature changes can make review easier.

## 20. Can Prettier and Stylelint both modify the same file?

They can. Configure a consistent workflow, run them in a predictable order, and review the final diff. Avoid enabling conflicting formatting rules that repeatedly rewrite the same code.

## 21. How should I handle a valid exception to a rule?

Use the narrowest relevant Stylelint disable comment, name the rule, and explain the reason. Revisit the exception when the code changes. Do not disable all rules for an entire file to avoid one local warning.

## 22. What are reportDescriptionlessDisables and reportNeedlessDisables for?

They help identify disable comments without an explanation and comments that no longer suppress an active lint problem. Related reporting can also identify invalid rule names. These checks keep exceptions limited and maintainable.

## 23. Should I change the shared config or add a disable comment?

Change the shared config when a rule is broadly wrong for the project's documented convention. Use a local disable only for an isolated exception. Keeping shared decisions in one config makes them visible and consistent.

## 24. Is deeper Sass nesting always wrong?

No. Nesting can make a direct relationship clear, but deep nesting often produces tightly coupled selectors with higher specificity. A nesting-depth rule can flag code for review; the right limit depends on the project.

## 25. Can linting verify accessibility?

Linting can catch some patterns when suitable rules are configured, but it cannot prove that a whole interface is accessible. Review focus visibility, contrast, responsive behavior, motion preferences, and actual interaction in context.

## 26. Should I use a property-order plugin?

Only if a consistent declaration order helps the team. Property order is often a convention rather than a correctness issue. If a plugin automatically reorders declarations, check that the change preserves intent and does not obscure cascade-sensitive code.

## 27. What should a CI stylesheet workflow run?

A typical pipeline checks formatting, lints authored source, and compiles Sass. Each command should use the project's locked dependencies and return a failure status when the check fails.

## 28. Should CI run Prettier with --write?

Usually use <code>prettier --check</code> in CI so it reports files that need formatting without silently changing the workspace. Developers can run <code>--write</code> locally and commit the resulting change.

## 29. Why might Stylelint pass without checking any files?

The glob may match no files because the path or extension is wrong. Review the command and consider how the CLI should behave for empty input. Do not let a no-file match appear to be a meaningful lint pass.

## 30. What should I do when Stylelint reports a syntax error?

Read the file location and parser message. Check for unclosed braces, brackets, or strings and verify that the correct syntax parser is configured for the file. A syntax parser error is different from an ordinary style-rule violation.

## 31. How should I add linting to an existing project with many violations?

Start by choosing a compatible shared config, run it on the current source, and review categories of findings. Fix the most valuable issues and agree on how legacy code will be brought under the rules. Avoid hiding all existing files behind a permanent ignore pattern.

## 32. What is the complete stylesheet quality loop?

Format authored source, lint it with the project's shared rules, compile with the project's Sass version, inspect rendered behavior in the browser, and review the changes. Each step checks a different part of the result.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="047-warnings-errors-and-debugging.md">047 - Warnings, errors, and debugging - Q&amp;A</a></td><td><a href="../lessons/048-linting-and-style-quality.md">Lesson</a></td><td><a href="../examples/048-linting-and-style-quality.md">Example</a></td><td><a href="049-source-maps-and-browser-devtools.md">049 - Source maps and browser developer tools - Q&amp;A</a></td></tr>
  </tbody>
</table>
