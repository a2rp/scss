# 004. Compiling SCSS to CSS — Q&A

## Compilation basics

### 1. What does the Sass compiler do?

It reads a Sass or SCSS entrypoint, loads its dependencies, evaluates Sass features, and writes CSS. Sass variables and nesting are transformed; the browser receives the resulting CSS.

### 2. What is an entrypoint?

An entrypoint is a Sass file that the build command compiles directly into an output stylesheet. It can load other Sass files as dependencies.

### 3. What is a Sass dependency?

A dependency is another Sass file loaded by an entrypoint, commonly with @use. Its Sass members and any CSS it emits participate in compiling that entrypoint.

### 4. What is the basic command for compiling one SCSS file?

Run the local Sass executable with the input first and the output second:

~~~sh
npx sass src/styles.scss dist/styles.css
~~~

### 5. What happens if I omit the output path?

Sass writes the compiled CSS to standard output, usually the terminal. It does not choose an output file automatically in one-to-one mode.

### 6. How does Sass determine the input syntax?

It recognizes .scss as SCSS syntax, .sass as indented Sass syntax, and .css as plain CSS. If the extension is not recognized, input is parsed as SCSS by default unless an option changes that behavior.

### 7. Should I edit the generated CSS?

Usually no. Edit the Sass source and compile it again. A later build can overwrite manual changes to generated output.

## Entrypoints and partials

### 8. How do I compile multiple entrypoints?

Use an input:output pair for each entrypoint:

~~~sh
npx sass src/site.scss:dist/site.css src/admin.scss:dist/admin.css
~~~

Each pair creates its own CSS output.

### 9. Does compiling two entrypoints combine them into one CSS file?

No. Each input:output pair creates a separate output. If one combined stylesheet is needed, create a deliberate entrypoint that loads the necessary Sass modules or use the project’s bundler configuration.

### 10. How do I map a source folder to an output folder?

Use directory input and output paths:

~~~sh
npx sass src/scss:public/css
~~~

Sass compiles the Sass entrypoints in the source tree and mirrors their relative directory structure under the output folder.

### 11. What is a partial?

A partial is generally a Sass file whose name begins with an underscore, such as _tokens.scss. It is intended to be loaded by another stylesheet rather than compiled as a standalone output.

### 12. Does directory compilation create CSS for underscore-prefixed partials?

No. Sass skips partials as standalone outputs in directory mode. An entrypoint can still load them, and their content can affect the entrypoint’s CSS.

### 13. Why did compiling a directory create more CSS files than I expected?

Every non-partial Sass file in the input tree may be treated as an entrypoint. Keep utility files and component dependencies underscore-prefixed when they are not meant to produce standalone output.

## Output formatting and source maps

### 14. What is expanded output?

Expanded output is readable CSS with selectors and declarations formatted on separate lines. It is Dart Sass’s default output style.

### 15. What is compressed output?

Compressed output removes optional whitespace to create compact CSS:

~~~sh
npx sass --style=compressed src/styles.scss dist/styles.css
~~~

It changes formatting; it does not perform every optimization that a production asset pipeline may provide.

### 16. What is a Sass source map?

A source map connects generated CSS back to the Sass files that produced it. Browser developer tools can use it to show original source locations when inspecting CSS.

### 17. Where does Sass write source maps?

When writing CSS files, Dart Sass generates source maps by default. It typically writes a .css.map file beside the CSS and adds a sourceMappingURL comment to the CSS.

### 18. How do I disable source maps in the CLI?

Pass --no-source-map:

~~~sh
npx sass --no-source-map src/styles.scss dist/styles.css
~~~

### 19. Should I use absolute file URLs in a deployed source map?

Usually not. An absolute file URL points to the compiler machine’s filesystem and generally will not resolve on another machine. Relative URLs are the default and are more appropriate for files deployed together.

## Watch, update, and errors

### 20. How do I recompile automatically while editing?

Use watch mode:

~~~sh
npx sass --watch src/scss:public/css
~~~

The command performs an initial compilation and remains active to recompile after changes.

### 21. What should I do if a dependency changes but watch mode does not recompile?

Check whether the dependency’s directory is included among the watched input directories, file parents, or load paths. Add the required directory or load path to the command when needed.

### 22. What does --update do?

It compiles only entrypoints whose sources or dependencies are newer than their corresponding CSS outputs:

~~~sh
npx sass --update src/scss:public/css
~~~

It runs a round of compilation and exits. Watch mode performs an initial update and then stays active.

### 23. Why might an output CSS file contain a compiler error?

Dart Sass can emit error CSS when compilation to a file fails. Read the terminal diagnostic and inspect the source error; the presence of a CSS file does not prove that compilation succeeded.

### 24. What is the right way to locate a compile error?

Read the first terminal error and its file and line. Check the named source and dependencies, correct the Sass, and run the compile command again.

### 25. The CSS was generated, but the browser still looks unchanged. What should I check?

Confirm the HTML loads the expected output path, inspect the network request, and then check whether the selector matches and another CSS rule is not taking precedence.

### 26. Why does directory compilation not automatically combine every file into one stylesheet?

Directory compilation maps Sass files to corresponding CSS files. It is not a bundling command that combines all source entrypoints into one output.

### 27. What is the difference between compiling once and watch mode?

A one-time compile runs and exits. Watch mode compiles once, remains active, and recompiles after relevant file changes.

## Quick review

- **One file:** input path followed by output path.
- **Multiple files:** provide each source:output pair.
- **Directory mode:** map one Sass directory to a CSS output directory.
- **Partial:** an underscore-prefixed Sass dependency, skipped as a directory-mode output.
- **Expanded:** readable default CSS.
- **Compressed:** compact formatting.
- **Source map:** a mapping from generated CSS back to Sass source.
- **Watch:** compile now and keep recompiling after changes.
- **Update:** compile only outputs that are stale.
