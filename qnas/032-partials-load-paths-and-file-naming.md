# 032 — Partials, Load Paths, and File Naming: Q&A

## Partials and file names

### 1. What is a Sass partial?

A partial is a Sass stylesheet conventionally named with a leading underscore because it is meant to be loaded by another stylesheet, not used as a standalone CSS entrypoint.

~~~text
_tokens.scss
_button.scss
~~~

### 2. Why does a partial filename start with an underscore?

The underscore tells Sass tools that the file is a partial and should not be compiled as an independent stylesheet. It can still be loaded by another Sass module.

### 3. Does a partial disappear from the compiled CSS?

No. If a loaded partial contains CSS rules, those rules can appear in the compiled output. The underscore affects standalone compilation behavior, not whether the module participates when another file loads it.

### 4. Is a partial the same thing as a private module?

No. A partial is a file naming convention. Sass member privacy is controlled by member names that begin with a hyphen or underscore. A partial can contain public members, private members, CSS, or any combination of them.

### 5. Do I include the underscore when writing @use?

Usually not. If the file is _tokens.scss, write:

~~~scss
@use "tokens";
~~~

### 6. Do I include the file extension in a module URL?

Usually not. For example, @use "tokens" can resolve to _tokens.scss. Omitting the extension also keeps the URL independent from the Sass syntax used by the file.

### 7. Can a non-partial stylesheet be loaded with @use?

Yes. @use can load a Sass stylesheet whether or not its filename starts with an underscore. Use the underscore convention to distinguish reusable modules from standalone entrypoints.

### 8. What should I name a reusable module?

Choose a descriptive name based on its responsibility, such as _colors.scss, _spacing.scss, _button.scss, or _form-field.scss. Avoid generic names that make the module hard to find.

### 9. Is the underscore in _tokens.scss related to underscores inside Sass member names?

They are separate conventions. The leading underscore in a filename marks a partial. A leading underscore in a variable, mixin, or function name marks that Sass member as private.

### 10. What is an _index.scss file for?

An _index.scss file can serve as a directory entrypoint. If a folder named tokens contains _index.scss, Sass can load that entrypoint using the folder URL:

~~~scss
@use "tokens";
~~~

### 11. Can _index.scss forward the directory's other modules?

Yes. For example:

~~~scss
// tokens/_index.scss
@forward "colors";
@forward "spacing";
~~~

Consumers can then load the directory entrypoint instead of referring to each partial individually.

## Module URLs and path resolution

### 12. What is a Sass module URL?

It is the URL string in a rule such as @use or @forward that tells Sass which stylesheet to load. It follows Sass's module resolution rules; it is not simply a copied operating-system path.

### 13. Are relative module URLs supported?

Yes. Sass resolves a module URL relative to the stylesheet that contains the @use or @forward rule. You do not need to write ./ for every relative URL.

### 14. How do I load a module in a sibling directory?

Use a relative URL with forward slashes:

~~~scss
// src/scss/components/_button.scss
@use "../tokens/colors";
~~~

The URL is relative to _button.scss, not to the terminal's current directory.

### 15. Should Sass URLs use backslashes on Windows?

No. Use forward slashes in Sass module URLs on every operating system:

~~~scss
@use "components/form-field";
~~~

A backslash may be interpreted as an escape and can make a URL unreliable.

### 16. Are module URLs case-sensitive?

Sass treats module URLs as case-sensitive. Use the exact spelling and capitalization of the file and directories so the project also works on case-sensitive filesystems.

### 17. Why does my stylesheet work on Windows but fail on Linux?

Common causes include a backslash in a Sass URL or capitalization that does not exactly match a filename. A case-insensitive filesystem can hide those mismatches locally.

### 18. What is a load path?

A load path is an additional directory Sass searches for modules when resolving a URL. It lets files in that directory be loaded without writing a long relative path from every consumer.

### 19. How do I add a load path with the Dart Sass CLI?

Use --load-path or its shorter form -I:

~~~bash
npx sass --load-path=src/scss src/scss/app.scss dist/app.css
~~~

### 20. Can I set more than one load path?

Yes. Pass the option more than once:

~~~bash
npx sass \
  --load-path=src/scss \
  --load-path=node_modules/library/scss \
  src/scss/app.scss dist/app.css
~~~

### 21. Which path does Sass check first: relative URL or load path?

Sass checks relative to the current stylesheet first. It checks load paths only when no matching relative module is found.

### 22. Which of two load paths wins?

When no relative module matches, earlier load paths take precedence over later ones. Put the preferred path first and avoid duplicates that make resolution surprising.

### 23. Can a relative file shadow a load-path module?

Yes. If the current stylesheet's relative directory contains a matching module, Sass uses it before checking configured load paths.

### 24. Are load paths relative to the Sass file or the terminal?

The URL lookup happens from the stylesheet first, then from configured load paths. The paths you pass to the CLI are configured from the command invocation and should be written according to the working directory and build setup.

### 25. Does adding a load path change where Sass looks for relative modules?

No. Relative lookup still comes first. A load path adds another place to search; it does not replace relative resolution.

## Compilation and troubleshooting

### 26. Which files should be entrypoints?

Use non-partial stylesheets for files you intend to compile into CSS, such as app.scss, admin.scss, or print.scss. Use partials for modules that are loaded by those entrypoints.

### 27. What happens when I compile a directory?

In Sass CLI directory mode, Sass compiles non-partial Sass files in the source directory into corresponding outputs. Partials are intended to be loaded and are not emitted as independent output files. Other build tools may use their own file discovery rules.

### 28. Why is Sass unable to find my partial?

Check the file name, relative path, configured load paths, URL capitalization, and slash direction. Confirm that the file exists and that you did not include the underscore in the URL by mistake.

### 29. Why is a different version of my module loading?

Check for a relative file with the same URL first. If none exists, inspect load paths in order; an earlier matching path takes priority over later ones.

### 30. What should I check if a directory URL fails?

Confirm the directory has an _index.scss or _index.sass file. Then check the index file's own @use and @forward URLs.

### 31. Can I use an absolute Windows path in @use?

Avoid doing that. Module URLs should be portable Sass URLs, not machine-specific filesystem paths. Use relative URLs or configure a load path.

### 32. Should every directory have an _index.scss?

No. Add an index when a directory needs a deliberate entrypoint that groups its public modules. A small folder with one consumer may be clearer without an extra barrel.

### 33. How can I avoid load-path collisions?

Keep load paths narrow, use descriptive filenames and directory names, and avoid storing unrelated modules with identical URLs. Remember that a relative match always takes precedence.

### 34. What is the difference between a file path and a module URL?

A file path identifies a location in the operating system's filesystem. A module URL is written in Sass and resolved by Sass's importer. For portable code, write Sass URLs with forward slashes and use relative URLs or configured load paths.

### 35. Do partials have to contain only variables and mixins?

No. A partial can contain any Sass supported by the language, including CSS rules. Decide whether its CSS should be emitted when the module is loaded.

### 36. What is a good quick rule for naming?

Name standalone compilation inputs without a leading underscore. Name reusable Sass modules with a leading underscore, and describe their responsibility in the rest of the filename.

## Quick reference

~~~text
File:                    Module URL:
_tokens.scss             @use "tokens";
components/_button.scss  @use "components/button";
tokens/_index.scss       @use "tokens";
~~~

~~~bash
npx sass src/scss/app.scss dist/app.css
npx sass --load-path=src/scss src/scss/app.scss dist/app.css
npx sass --load-path=src/scss --load-path=node_modules/library/scss src/scss/app.scss dist/app.css
~~~

## Related material

- [Lesson: Partials, load paths, and file naming](../lessons/032-partials-load-paths-and-file-naming.md)
- [Example: Partials, load paths, and file naming](../examples/032-partials-load-paths-and-file-naming.md)
- [Sass: @use and module resolution](https://sass-lang.com/documentation/at-rules/use/)
- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/dart-sass/)
