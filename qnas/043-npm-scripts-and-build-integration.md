# npm scripts and build integration - Q&A

This Q&A explains how a project can run Sass through npm scripts and connect compilation to its build workflow. Commands are examples; adjust paths and script names to the project.

## 1. Why use npm scripts for Sass?

A named script gives developers and automation the same documented command. It also makes the project’s Sass version available locally rather than relying on a global executable installed on one machine.

## 2. How do I add Sass to the project?

Install Sass as a development dependency:

~~~sh
npm install --save-dev sass
~~~

This updates the package manifest and lockfile. Use the dependency installation convention and lockfile already selected by the project.

## 3. Why is Sass usually a devDependency?

Sass is generally needed to compile source during development and the build, while the browser receives CSS. If the build runs in a separate stage, that stage still needs the development dependencies installed before Sass compilation.

A project that compiles styles at runtime has a different architecture and should make that dependency choice deliberately.

## 4. Do I need a global Sass installation?

No. npm scripts add local package executables to their command path. After installing Sass in the project, a script can call <code>sass</code> without an absolute path or a global install.

## 5. How can I check which Sass CLI is installed?

Run the project’s local executable through npm:

~~~sh
npx sass --version
~~~

You can also add a script that prints the version. This helps confirm that the project uses its declared Sass dependency rather than an unrelated global installation.

## 6. What does an npm script look like?

A script is a command in the <code>scripts</code> object in <code>package.json</code>:

~~~json
{
  "scripts": {
    "build:css": "sass src/styles/app.scss public/css/app.css"
  }
}
~~~

Run it from the package with <code>npm run build:css</code>.

## 7. How does npm find the sass command?

When npm runs a package script, it adds the local <code>node_modules/.bin</code> directory to the script’s executable path. The script can use <code>sass</code> directly.

This means the executable resolves to the package version installed for that project.

## 8. Where do npm scripts run?

npm scripts run from the package root, regardless of the directory from which <code>npm run</code> was invoked. Write relative input and output paths with that in mind.

If a command needs to behave based on the original invocation directory, npm provides <code>INIT_CWD</code>, but most Sass scripts should use stable project-root paths.

## 9. What is the one-to-one Sass CLI form?

It compiles one input to one output:

~~~sh
sass src/styles/app.scss public/css/app.css
~~~

If the output path is omitted, Sass prints compiled CSS to standard output. A named CSS output is usually more useful for an application build.

## 10. How do I compile multiple entry points?

Pass input-to-output pairs separated by colons:

~~~sh
sass src/styles/public.scss:public/css/public.css src/styles/admin.scss:public/css/admin.css
~~~

This can be stored in an npm script. Each input should be an intended entry point, not an implementation partial.

## 11. How do I compile a Sass directory?

Map an input directory to an output directory:

~~~sh
sass src/themes:public/css/themes
~~~

Sass ignores partials whose names begin with an underscore when compiling a directory. Use this mode only when the non-partial files in that directory are meant to become CSS outputs.

## 12. What should I use as the Sass input?

Use an application entry point that loads its dependencies. Do not point the build at a token partial or component partial unless it is intentionally an independent CSS output.

The entry point tells Sass and future readers what makes up that bundle.

## 13. How do I watch Sass during development?

Use the CLI’s watch flag:

~~~sh
sass --watch src/styles/app.scss:public/css/app.css
~~~

The process stays open and recompiles when watched source files change. Stop it with Ctrl+C. Use a separate npm script for watch mode rather than making the normal build command stay open.

## 14. Why is watch mode not compiling one of my changes?

Check that the edited file is in a directory Sass is watching or in a configured load path. Sass watches directories passed on the command line, parent directories of input files, and load paths; it does not infer arbitrary watch folders from every dependency URL.

For remote filesystems where OS notifications do not work, the CLI has polling mode with <code>--poll</code>.

## 15. What does --update do?

<code>--update</code> compiles only stylesheets whose dependencies changed more recently than their CSS outputs:

~~~sh
sass --update src/styles:public/css
~~~

It is useful with multiple source files, but it does not replace the build system’s dependency graph or package installation.

## 16. What are expanded and compressed CSS?

Expanded output is the default and is easier to inspect. Compressed output removes most whitespace and comments for a smaller file:

~~~sh
sass --style=compressed src/styles/app.scss public/css/app.css
~~~

Choose output style in the appropriate build script. Compression does not change the Sass source or the meaning of the CSS.

## 17. Does Sass generate source maps by default?

Dart Sass generates source maps by default when it emits CSS files. A source map helps browser tools connect generated CSS back to Sass source.

Use <code>--no-source-map</code> when an output should not include a map:

~~~sh
sass --no-source-map src/styles/app.scss public/css/app.css
~~~

Choose based on the debugging and deployment needs of that output.

## 18. How do I make separate development and production scripts?

Give the behaviors distinct names:

~~~json
{
  "scripts": {
    "watch:css": "sass --watch src/styles/app.scss:public/css/app.css",
    "build:css": "sass --style=compressed --no-source-map src/styles/app.scss public/css/app.css"
  }
}
~~~

The example disables source maps for the one-time build. Keep them in the production output if the project’s debugging or monitoring workflow needs them.

## 19. How do I add Sass to a larger npm build?

Call the Sass script from the top-level build:

~~~json
{
  "scripts": {
    "build:css": "sass --style=compressed src/styles/app.scss public/css/app.css",
    "build:js": "node scripts/build-js.mjs",
    "build": "npm run build:css && npm run build:js"
  }
}
~~~

The sequence stops if the Sass command fails, so later steps do not hide a compilation error.

## 20. Does && work on Windows?

Yes, the default Windows command shell used by npm supports <code>&&</code> for sequencing commands. More complex shell syntax, environment-variable assignment, and filesystem commands can differ between shells.

Keep scripts simple and use a small Node script for custom cross-platform behavior when appropriate.

## 21. How do I pass an extra Sass option to a script?

Add <code>--</code> before the extra arguments:

~~~sh
npm run build:css -- --verbose
~~~

npm passes <code>--verbose</code> to the script’s command. Do not put npm’s separator into the stored Sass command itself.

## 22. What do prebuild and postbuild scripts do?

For arbitrary npm script names, matching <code>pre</code> and <code>post</code> scripts can run before and after the named script. For example, <code>npm run build</code> may run <code>prebuild</code>, <code>build</code>, and <code>postbuild</code>.

Use hooks intentionally and keep important build behavior easy to see. Special npm lifecycle scripts have separate rules, so do not treat every similarly named command as interchangeable.

## 23. Why should the lockfile be committed?

The lockfile records the resolved dependency tree used by the project. When the repository tracks npm’s lockfile, commit changes to both the package manifest and lockfile so local installs and automated builds are consistent.

Do not add a second package manager’s lockfile without a migration plan.

## 24. What is npm ci used for?

<code>npm ci</code> installs dependencies from the lockfile in a clean environment. It is commonly used in continuous integration before running the build:

~~~sh
npm ci
npm run build:css
~~~

The CI environment must include Sass during the build stage if the project declares it as a development dependency.

## 25. What if a framework already compiles Sass?

Use the framework’s Sass integration if that is how the project builds its CSS. Do not separately compile the same entry point with the Sass CLI unless the second output is intentional.

Choose one owner for each input entry point to avoid duplicated or inconsistent CSS.

## 26. Should I use the CLI or the Sass JavaScript API?

Use the CLI for straightforward source-to-CSS tasks and npm scripts. The JavaScript API is useful when application code needs to control compilation programmatically, configure importers, or integrate custom behavior.

Choose the simplest interface that meets the build’s needs.

## 27. What is a Sass load path?

A load path adds a directory where Sass can search for modules:

~~~sh
sass --load-path=node_modules src/styles/app.scss public/css/app.css
~~~

Use it to solve a real module-resolution need and document the expected URL format. A load path changes where Sass looks; it does not change the Sass language.

## 28. How can I use Node package URLs?

Dart Sass supports an optional Node package importer in supported versions. The CLI flag is <code>--pkg-importer=node</code>. Use it only when the Sass packages and project URLs are designed for that importer; a regular load path may be a better fit for simpler setups.

Check the CLI documentation for version requirements before using an option in a shared build.

## 29. What happens when Sass compilation fails?

Sass reports the error and returns a failing status. An npm script or CI build should preserve that status so a failed stylesheet build cannot be mistaken for success.

Do not append commands that mask the error or ignore it without a documented reason.

## 30. Should I silence Sass warnings?

Do not silence project warnings just to reduce terminal output. Fix warnings in first-party source and review warnings from dependencies. The CLI’s <code>--quiet-deps</code> option can suppress dependency warnings while retaining warnings from application code.

Use <code>--verbose</code> when you need to see all deprecation warnings.

## 31. How do I choose the CSS output location?

Use a stable path that the application or framework serves or imports, such as <code>public/css/app.css</code>. Confirm that the build writes to the path the page actually loads.

A successful command that writes an unused file is not a complete integration.

## 32. Why should scripts avoid inline environment-variable syntax?

Syntax such as setting an environment variable before a command differs between Windows shells and POSIX shells. For a portable build, use Sass CLI options directly, separate scripts, or a Node script that handles environment configuration.

## 33. Can I commit generated CSS?

That depends on the deployment workflow. Some projects commit generated assets; others build them during deployment. Choose one rule, keep authored Sass separate from generated files, and make sure the deploy process follows the same rule.

## 34. What are common Sass/npm integration mistakes?

- Relying on a global Sass installation instead of the project dependency.
- Compiling a partial instead of the intended entry point.
- Using a broad glob that emits unintended stylesheets.
- Writing output to a path the application never uses.
- Running a watcher in a command intended to finish once.
- Compiling the same input through both the CLI and a bundler.
- Hiding the Sass process’s failing exit status.
- Installing without the lockfile in CI.
- Suppressing all warnings without reviewing them.

## 35. What should I check before calling the integration complete?

Confirm the Sass dependency and lockfile are present, entry points and output paths are correct, scripts are named by purpose, watch mode is separate, build failures remain visible, and the framework or CLI has one clear owner for each Sass entry.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="042-project-structure-and-naming-conventions.md">042 - Project structure and naming conventions - Q&amp;A</a></td><td><a href="../lessons/043-npm-scripts-and-build-integration.md">Lesson</a></td><td><a href="../examples/043-npm-scripts-and-build-integration.md">Example</a></td><td><a href="044-sass-cli-watch-and-build-options.md">044 - Sass CLI, watch mode, and build options - Q&amp;A</a></td></tr>
  </tbody>
</table>
