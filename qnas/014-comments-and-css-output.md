# 014 - Comments and Generated CSS Output: Q&A

## Comments in SCSS

### 1. What are the two main comment forms in SCSS?

SCSS supports silent comments beginning with // and block comments written as /* ... */. They behave differently in generated CSS.

### 2. What is a silent comment?

A // comment continues to the end of its line and is never emitted into CSS. It is useful for notes that belong only in the Sass source.

### 3. What is a loud comment?

A block comment written as /* ... */ in a statement position is normally emitted as a CSS comment in expanded output.

### 4. Do all block comments remain in compressed CSS?

No. Dart Sass removes ordinary block comments in compressed mode to reduce the output size.

### 5. How do I preserve a comment in compressed CSS?

Start the comment with /*! instead of /*. Sass preserves that comment in compressed output. Use this for required notices such as license information or attribution.

### 6. Should every source comment use /*!?

No. Use it only when the comment needs to ship in CSS. It remains in the production output and adds bytes, so use // for source-only notes.

### 7. Can a block comment contain Sass interpolation?

Yes. A loud comment can contain interpolation. Sass evaluates the expression before emitting the CSS comment.

### 8. Are interpolated values emitted from a silent comment?

No. Silent comments are discarded, so their contents and interpolation do not appear in the generated stylesheet.

### 9. Can comments appear between selector parts?

Yes. SCSS allows comments where CSS allows whitespace. Sass removes the comment while preserving the selector's token separation. This is legal but can reduce readability.

### 10. What are SassDoc comments?

They are documentation comments written with three slashes directly above a Sass API item. SassDoc tooling can parse their Markdown and annotations. They are silent and are not emitted as CSS.

### 11. Do comments change the CSS behavior?

Comments themselves do not change the intended declarations. However, a comment that is emitted becomes part of the CSS file, and malformed syntax around comments can still cause a compilation error.

### 12. Should private notes or secrets be placed in a loud comment?

No. A loud comment can be published in generated CSS, especially with a /*! prefix. Do not put secrets or private notes in content that may ship.

## Generated CSS

### 13. What does Sass generate from an SCSS file?

Sass evaluates Sass-specific syntax and writes CSS. Variables become their values, mixins expand into declarations, loops generate repeated rules, and nested selectors become ordinary CSS selectors.

### 14. Does the browser receive the original SCSS?

No. The browser receives CSS. Source maps can help developer tools connect generated CSS back to the original Sass files.

### 15. How do I compile one SCSS file to one CSS file?

Use the Dart Sass CLI with an input path and an output path:

~~~sh
sass src/styles.scss public/styles.css
~~~

### 16. What happens if I omit the output path?

Sass prints the generated CSS to the terminal instead of writing it to a CSS file.

### 17. What does expanded output mean?

Expanded output is the default Dart Sass style. It formats selectors and declarations on separate lines, which is easier to read and debug.

### 18. What does compressed output mean?

Compressed output removes as many extra characters as possible and places the stylesheet on one line. It is compact for delivery and harder to inspect manually.

### 19. How do I request each output style?

Pass --style=expanded or --style=compressed to the Sass CLI.

### 20. Does compressed output change the design?

It should not change the intended CSS behavior. It changes formatting and removes optional comments. Compare the resulting declarations if a build appears different.

### 21. Where should I fix a problem I see in generated CSS?

Fix the SCSS source and compile again. Manual changes to generated CSS can be overwritten by the next build.

### 22. What is one-to-one Sass CLI mode?

It compiles one Sass input file to one output location. The output can be a file or, when omitted, the terminal.

### 23. Can Sass compile a directory of files?

Yes. Dart Sass supports many-to-many mode, which maps Sass inputs or directories to CSS outputs. Files whose names begin with an underscore are partials and are skipped when compiling a directory.

## Source maps

### 24. What is a source map?

A source map records how generated CSS corresponds to the Sass files that produced it. Browser developer tools can use it to show the original Sass source while inspecting styles.

### 25. Does Dart Sass create source maps by default?

Yes. Dart Sass generates source maps by default for CSS files it emits.

### 26. How can I turn off source-map generation?

Pass --no-source-map to the Sass CLI.

### 27. What is the difference between relative and absolute source-map URLs?

Relative URLs refer to Sass source files relative to the map's location, making the output portable when the directory structure is preserved. Absolute URLs refer to file locations on the compiling machine and are generally not portable.

### 28. What does --embed-sources do?

It embeds the contents of contributing Sass files into the source map. This can make the map larger, but it makes the source available wherever that map is served.

### 29. What does --embed-source-map do?

It embeds the map contents in the generated CSS instead of writing a separate linked map file.

### 30. Should source maps be published with production CSS?

That depends on the project. Source maps are useful for debugging, but they may expose source paths or, when sources are embedded, the Sass contents. Choose and configure them intentionally.

## Encoding and debugging

### 31. What does --no-charset do?

It prevents Dart Sass from adding a UTF-8 charset marker or byte-order mark for non-ASCII output. Use it only when it fits the project's serving and encoding setup.

### 32. Why does a generated CSS file contain a sourceMappingURL comment?

That comment connects the CSS file to its source map. It is tooling metadata used by developer tools, not a Sass source comment.

### 33. Why is a block comment missing after compilation?

Check the selected output style. Ordinary block comments are removed in compressed mode. Use /*! for a notice that must remain.

### 34. Why is my // note not visible in browser developer tools?

Silent comments are removed during compilation. Use an emitted block comment only when the note belongs in generated CSS.

### 35. How can I inspect Sass output during development?

Compile with expanded style, open the generated CSS, and use source maps in browser developer tools. Keep the source SCSS as the file you edit.

### 36. What is the key rule for comments and CSS output?

Decide whether each comment belongs only in source or must be published. Then choose output formatting and source-map settings deliberately for the build target.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="013-colors-and-sass-color-module.md">013 - Colors and the Sass Color Module: Q&amp;A</a></td><td><a href="../lessons/014-comments-and-css-output.md">Lesson</a></td><td><a href="../examples/014-comments-and-css-output.md">Example</a></td><td><a href="015-nesting-rules-and-declarations.md">015 - Nesting Rules and Declarations: Q&amp;A</a></td></tr>
  </tbody>
</table>
