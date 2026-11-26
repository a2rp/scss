# npm scripts and build integration

npm scripts give a project a consistent way to install Sass, compile stylesheets, watch for edits, and connect Sass compilation to a larger build. The goal is to make the command clear and repeatable for developers and automation.

This lesson uses the Sass command-line interface (CLI) from the npm package. A framework or bundler may already compile Sass; choose one owner for the compilation step so the same entry point is not processed twice by accident.

## Learning goals

After this lesson, you should be able to:

- Add Sass as a project dependency and invoke its CLI through npm.
- Define readable development and production scripts in <code>package.json</code>.
- Compile one entry point or several entry points to CSS outputs.
- Use watch mode, source maps, output styles, and load paths intentionally.
- Chain Sass into a larger build without hiding failures.
- Keep scripts portable and distinguish build configuration from Sass source.

## 1. Install Sass in the project

For a Node-based project, add the Sass package as a development dependency:

~~~sh
npm install --save-dev sass
~~~

This records Sass in <code>package.json</code> and updates the lockfile. Team members and CI can install the recorded dependency from the project rather than relying on a Sass executable installed globally on one machine.

Check the installed CLI from the project root:

~~~sh
npx sass --version
~~~

When npm runs a package script, it adds the project’s local executable directory to the command search path. That lets a script invoke <code>sass</code> directly.

## 2. Add focused scripts to package.json

A minimal package manifest might contain:

~~~json
{
  "scripts": {
    "build:css": "sass src/styles/app.scss public/css/app.css",
    "watch:css": "sass --watch src/styles/app.scss:public/css/app.css"
  },
  "devDependencies": {
    "sass": "^1.0.0"
  }
}
~~~

The version range above is only illustrative; use the version installed and recorded by your package manager. Keep the lockfile in sync with <code>package.json</code>.

Run the commands with:

~~~sh
npm run build:css
npm run watch:css
~~~

The build command compiles once and exits. The watch command remains open and recompiles when source files change. Stop it with Ctrl+C.

## 3. Know how npm finds the command

The npm script can call <code>sass</code> because npm places local package executables on the script’s path. A script generally does not need an absolute path to <code>node_modules/.bin/sass</code> and should not require the Sass CLI to be installed globally.

This improves consistency across machines: the command uses the Sass version declared for this project.

## 4. Compile one entry point

Sass can compile a single source file to a CSS file:

~~~sh
sass src/styles/app.scss public/css/app.css
~~~

This is one-to-one mode. If no output path is supplied, Sass writes the CSS to the terminal. For an application build, an explicit output path is usually easier to integrate and review.

The file passed as input should be an application entry point that loads the partials it needs.

## 5. Compile multiple entry points

The CLI can map several input files to separate outputs:

~~~sh
sass src/styles/public.scss:public/css/public.css src/styles/admin.scss:public/css/admin.css
~~~

It can also compile a directory of Sass files to a directory of CSS files:

~~~sh
sass src/themes:public/css/themes
~~~

When compiling a directory, Sass ignores partial files whose names begin with an underscore. Choose directory compilation only when each non-partial file is meant to become an output stylesheet.

## 6. Development and production output

Dart Sass writes expanded CSS by default, which is convenient to inspect. Use compressed output for a production build:

~~~sh
sass --style=compressed src/styles/app.scss public/css/app.css
~~~

A project can define separate scripts:

~~~json
{
  "scripts": {
    "build:css": "sass --style=compressed src/styles/app.scss public/css/app.css",
    "build:css:debug": "sass --style=expanded src/styles/app.scss public/css/app.css"
  }
}
~~~

Use a named script rather than making developers remember output flags. Keep the source entry point and destination consistent between the scripts.

## 7. Source maps

Dart Sass generates source maps by default when it writes CSS files. A source map connects compiled CSS back to its Sass sources, which is useful during local browser debugging.

For an output that should not include a separate source map, pass <code>--no-source-map</code>:

~~~sh
sass --no-source-map --style=compressed src/styles/app.scss public/css/app.css
~~~

Choose this per build target. Do not disable source maps automatically in every environment if developers rely on them.

## 8. Watch the source during development

Use watch mode to keep Sass running while files change:

~~~sh
sass --watch src/styles/app.scss:public/css/app.css
~~~

Add a script for it:

~~~json
{
  "scripts": {
    "watch:css": "sass --watch src/styles/app.scss:public/css/app.css"
  }
}
~~~

Sass watches the input’s parent directory and load paths. Put dependencies under the watched source tree or specify a suitable load path so edits are detected. If file notifications do not work on a remote filesystem, Sass also offers polling mode with <code>--poll</code>.

Watch mode is a foreground development command, not a production build step.

## 9. Compile only changed outputs with --update

The <code>--update</code> option skips stylesheets whose source and dependencies have not changed since the output was generated:

~~~sh
sass --update src/styles:public/css
~~~

This is useful for a directory with multiple entry files. It does not replace a project’s build graph or dependency manager; it is an optimization in Sass’s CLI.

## 10. Organize build scripts by purpose

Use script names that tell the user what the command does:

~~~json
{
  "scripts": {
    "build:css": "sass --style=compressed --no-source-map src/styles/app.scss public/css/app.css",
    "watch:css": "sass --watch src/styles/app.scss:public/css/app.css",
    "build": "npm run build:css"
  }
}
~~~

A larger project can extend the top-level build:

~~~json
{
  "scripts": {
    "build:css": "sass --style=compressed src/styles/app.scss public/css/app.css",
    "build:js": "node scripts/build-js.mjs",
    "build": "npm run build:css && npm run build:js"
  }
}
~~~

The <code>&&</code> sequence runs the second command only if the first succeeds. This lets a Sass compile error fail the overall build rather than being hidden by a later command.

Do not create many aliases for the same command. Prefer a small set of scripts that match the development and deployment workflow.

## 11. Understand npm script execution

Run a named script with <code>npm run name</code>. npm runs scripts from the package root and provides local dependencies’ executable paths.

Arguments can be passed to the script after <code>--</code>:

~~~sh
npm run build:css -- --verbose
~~~

This appends <code>--verbose</code> to the Sass command. npm has special lifecycle behavior for some script names, and matching <code>pre</code> and <code>post</code> scripts may also run around a named script. Use those hooks intentionally rather than hiding important build steps in them.

## 12. Chain build steps without swallowing errors

For a simple cross-platform sequence, npm scripts can call other npm scripts:

~~~json
{
  "scripts": {
    "clean:css": "node scripts/clean-css.mjs",
    "build:css": "sass --style=compressed src/styles/app.scss public/css/app.css",
    "build": "npm run clean:css && npm run build:css"
  }
}
~~~

The clean script in this example is illustrative. Keep cleanup scoped to the generated CSS directory and make it safe for the project. Do not use a shell-specific command if the project needs scripts to work on both Windows and Unix-like systems.

A build should return a nonzero status when Sass fails. Avoid appending commands that mask Sass’s exit status.

## 13. Add load paths only when needed

A load path tells Sass where else to look for module URLs:

~~~sh
sass --load-path=node_modules src/styles/app.scss public/css/app.css
~~~

In a script:

~~~json
{
  "scripts": {
    "build:css": "sass --load-path=node_modules src/styles/app.scss public/css/app.css"
  }
}
~~~

Prefer explicit module URLs and the package’s supported Sass integration when possible. Add a load path to solve a real resolution problem, and document it so imports are not mysterious.

Dart Sass also supports a Node package importer in supported versions. Use it only when the project and its dependencies are set up for package URLs; a load path is not the same thing as changing the module URL syntax.

## 14. Integrate Sass with a framework or bundler

Some frameworks compile Sass as part of their own build pipeline. In that case, follow the framework’s documented Sass integration and avoid separately compiling the same entry point unless the output is intentionally separate.

A project may use either of these approaches:

- **Standalone CLI:** npm script compiles Sass entry points directly to CSS.
- **Bundler integration:** the application build imports Sass and the bundler handles compilation.

Keep one clearly documented owner for each Sass entry point. Running both paths against the same source can produce duplicate CSS, unused outputs, or inconsistent settings.

## 15. Separate development commands from release builds

Development commonly uses watch mode and source maps. A release build commonly compiles once, may use compressed CSS, and may choose not to publish source maps.

~~~json
{
  "scripts": {
    "dev:css": "sass --watch src/styles/app.scss:public/css/app.css",
    "build:css": "sass --style=compressed --no-source-map src/styles/app.scss public/css/app.css"
  }
}
~~~

The exact choice depends on deployment and debugging needs. Make the difference visible in the script names rather than depending on undocumented shell flags.

## 16. Keep package metadata and lockfiles consistent

Installing Sass with npm updates the package manifest and lockfile. Commit both when the repository commits its dependency lockfile, so local and automated installations use the project’s declared dependency resolution.

For a clean automated install, npm projects commonly use:

~~~sh
npm ci
npm run build:css
~~~

Use the package manager and lockfile already selected by the project. Do not mix npm, pnpm, and Yarn lockfiles without an explicit migration plan.

## 17. Handle Sass errors and warnings

If Sass cannot compile an entry point, the command exits with an error. Keep the Sass command’s failure visible to the enclosing npm script or build system.

Sass may emit warnings for deprecated features or explicit <code>@warn</code> statements. Fix warnings in project code and review dependency warnings. Do not silence all warnings merely to make build output look clean.

For command-line diagnostics, the CLI provides options such as <code>--verbose</code> and <code>--quiet-deps</code>. Use quiet dependency output carefully; it should not conceal warnings from first-party source.

## 18. Use a predictable output path

Choose a stable destination such as <code>public/css/app.css</code> or the framework’s configured assets directory. Make sure the application serves or imports that exact output.

Keep Sass source maps and CSS outputs together according to the project’s asset workflow. A build that succeeds but writes to an unused path is not integrated.

## 19. Make scripts portable

npm runs scripts through a shell, and shell details differ across operating systems. Prefer portable Sass CLI arguments and npm-to-npm script composition. Avoid depending on inline environment-variable syntax or commands that exist only in one shell.

If a custom operation is needed, a small Node script is often more portable than a long shell command. Keep the npm script itself readable.

## 20. Build integration checklist

Before considering Sass integrated, confirm that:

- Sass is declared in the project dependencies.
- The project has one explicit command for each intended output.
- Each command compiles the right entry point and writes to a served or imported path.
- Partials are not accidentally compiled as standalone outputs.
- Development watch and release build behavior are distinct.
- Source-map behavior is deliberate.
- Sass errors fail the enclosing build.
- The build pipeline has one owner for each Sass entry point.
- The package manifest and lockfile are consistent.
- Another developer can run the commands from the project root.

## Common mistakes

### Relying on a global Sass installation

A global CLI can differ from the project’s declared version. Install Sass in the project and invoke it through npm scripts.

### Using the wrong input path

A script can exit successfully while generating CSS somewhere the app never loads. Verify both the input entry point and output destination.

### Compiling partials as separate outputs

Use explicit entry files or a directory mapping that ignores partials. Avoid broad globs that emit a CSS file for every source module.

### Starting watch mode during a release build

Watch mode does not represent a one-time production build. Keep separate named scripts for development and release output.

### Running two Sass pipelines on the same entry point

If a bundler already compiles Sass, a second CLI compilation can create duplicate or inconsistent output. Choose one owner for that entry point.

### Hiding a failed Sass command

When chaining scripts, use a failure-sensitive sequence such as <code>&&</code> and do not ignore the Sass process status.

### Disabling source maps without considering development

Source maps help connect browser CSS to Sass files. Disable them only for outputs where that tradeoff is intended.

## Quick reference

| Need | Example |
| --- | --- |
| Install Sass locally | <code>npm install --save-dev sass</code> |
| Compile one entry | <code>sass src/styles/app.scss public/css/app.css</code> |
| Watch one entry | <code>sass --watch src/styles/app.scss:public/css/app.css</code> |
| Compile compressed CSS | <code>sass --style=compressed input.scss output.css</code> |
| Disable source maps | <code>sass --no-source-map input.scss output.css</code> |
| Compile only changed files | <code>sass --update src/styles:public/css</code> |
| Run a project script | <code>npm run build:css</code> |
| Pass an extra CLI option | <code>npm run build:css -- --verbose</code> |

## Next lesson

Continue with [Sass CLI, watch mode, and build options](044-sass-cli-watch-and-build-options.md).
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="042-project-structure-and-naming-conventions.md">042 - Project structure and naming conventions</a></td><td><a href="../qnas/043-npm-scripts-and-build-integration.md">Q&amp;A</a></td><td><a href="../examples/043-npm-scripts-and-build-integration.md">Example</a></td><td><a href="044-sass-cli-watch-and-build-options.md">044 - Sass CLI, watch mode, and build options</a></td></tr>
  </tbody>
</table>
