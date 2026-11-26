# npm scripts and build integration — Examples

These examples use the Sass CLI installed in the project’s npm dependencies. Run npm commands from the package root unless a project script documents otherwise.

## 1. Add Sass to a project

~~~sh
npm install --save-dev sass
~~~

npm records Sass in <code>package.json</code> and updates the lockfile. The exact installed version is recorded by the package manager; do not copy a version number from an unrelated project.

## 2. Define a one-time CSS build script

Add a script to <code>package.json</code>:

~~~json
{
  "scripts": {
    "build:css": "sass src/styles/app.scss public/css/app.css"
  }
}
~~~

Run it:

~~~sh
npm run build:css
~~~

npm makes the local Sass executable available to the script. No global Sass install is needed.

## 3. Compile an entry point directly

~~~sh
sass src/styles/app.scss public/css/app.css
~~~

Expected result: Sass reads the app entry point, follows its module dependencies, writes CSS to <code>public/css/app.css</code>, and exits.

The entry point might load modules like this:

~~~scss
// src/styles/app.scss
@use "foundations/reset";
@use "components/button";
@use "components/card";
~~~

## 4. Separate the watch command from the build

~~~json
{
  "scripts": {
    "build:css": "sass src/styles/app.scss public/css/app.css",
    "watch:css": "sass --watch src/styles/app.scss:public/css/app.css"
  }
}
~~~

Use <code>npm run build:css</code> for a one-time compilation. Use <code>npm run watch:css</code> during development; stop the foreground watcher with Ctrl+C.

## 5. Compile public and admin bundles

~~~json
{
  "scripts": {
    "build:css": "sass src/styles/public.scss:public/css/public.css src/styles/admin.scss:public/css/admin.css"
  }
}
~~~

Sass compiles each input-to-output pair. Keep every Sass file in this command an intentional application entry point.

## 6. Compile a directory of themes

Source tree:

~~~text
src/
  themes/
    light.scss
    dark.scss
    _shared-tokens.scss
~~~

Command:

~~~sh
sass src/themes:public/css/themes
~~~

Sass maps non-partial source files to CSS outputs and ignores names beginning with an underscore when compiling the directory. Use this approach only when each non-partial file is intended to produce CSS.

## 7. Create expanded and compressed scripts

~~~json
{
  "scripts": {
    "build:css:debug": "sass --style=expanded src/styles/app.scss public/css/app.css",
    "build:css": "sass --style=compressed src/styles/app.scss public/css/app.css"
  }
}
~~~

Expanded CSS is easier to inspect. Compressed CSS removes formatting whitespace for a release output.

## 8. Keep or disable source maps intentionally

Dart Sass generates source maps by default when writing CSS:

~~~sh
sass src/styles/app.scss public/css/app.css
~~~

To disable a source map for a chosen output:

~~~sh
sass --no-source-map --style=compressed src/styles/app.scss public/css/app.css
~~~

A matching package script might be:

~~~json
{
  "scripts": {
    "build:css": "sass --no-source-map --style=compressed src/styles/app.scss public/css/app.css"
  }
}
~~~

Keep source maps in development if browser debugging needs to trace CSS back to Sass.

## 9. Add a load path

~~~json
{
  "scripts": {
    "build:css": "sass --load-path=node_modules src/styles/app.scss public/css/app.css"
  }
}
~~~

This adds <code>node_modules</code> as a Sass load path. Use it only when a dependency’s Sass files need to be resolved from that path. Keep the module URLs clear and documented.

## 10. Compile only changed stylesheets

~~~json
{
  "scripts": {
    "update:css": "sass --update src/styles:public/css"
  }
}
~~~

<code>--update</code> is useful when a directory contains several Sass entry files. Sass updates outputs whose inputs or dependencies have changed.

## 11. Compose CSS and JavaScript builds

~~~json
{
  "scripts": {
    "build:css": "sass --style=compressed src/styles/app.scss public/css/app.css",
    "build:js": "node scripts/build-js.mjs",
    "build": "npm run build:css && npm run build:js"
  }
}
~~~

The JavaScript command is illustrative. With <code>&&</code>, the next command runs only if the previous one succeeds. A Sass error therefore fails the top-level build.

## 12. Install from the lockfile in CI

For a project that tracks <code>package-lock.json</code>:

~~~sh
npm ci
npm run build:css
~~~

The Sass development dependency must be installed in the build environment before compiling. The deployed application can then serve the generated CSS without running Sass in the browser.

## 13. Pass a Sass CLI option through npm

Stored script:

~~~json
{
  "scripts": {
    "build:css": "sass src/styles/app.scss public/css/app.css"
  }
}
~~~

Run with an extra flag:

~~~sh
npm run build:css -- --verbose
~~~

The separator <code>--</code> tells npm to pass the following option to the script command.

## 14. Add a top-level development script

~~~json
{
  "scripts": {
    "watch:css": "sass --watch src/styles/app.scss:public/css/app.css",
    "dev": "npm run watch:css"
  }
}
~~~

In a project that also watches JavaScript or starts a server, use the framework’s documented process runner or a cross-platform helper to manage multiple long-running tasks. Do not put shell-specific background syntax into a shared script without checking every supported environment.

## 15. Use a bundler as the Sass owner

If the app’s bundler imports Sass directly, an npm script may call only that build:

~~~json
{
  "scripts": {
    "build": "vite build"
  }
}
~~~

In this approach, the bundler owns Sass compilation for its imported entry points. Do not also run a separate Sass CLI command for those same entries unless the second output is intentional.

The tool name is an example; use the project’s actual bundler and its current Sass integration.

## 16. Keep a directory build from emitting partials

Example:

~~~text
src/styles/
  app.scss
  _tokens.scss
  _reset.scss
  components/
    _button.scss
~~~

Running:

~~~sh
sass src/styles:public/css
~~~

does not make each underscore-prefixed partial a standalone CSS output. If only <code>app.scss</code> should produce CSS, compile that file directly to make the build intent explicit.

## 17. Verify the output path is served

Script:

~~~json
{
  "scripts": {
    "build:css": "sass src/styles/app.scss public/assets/app.css"
  }
}
~~~

HTML or the framework should reference the same output:

~~~html
<link rel="stylesheet" href="/assets/app.css">
~~~

A build command can succeed even if the website never loads the generated file. Keep the output path and the application’s CSS reference aligned.

## 18. Avoid depending on a global executable

Fragile setup: a developer has Sass installed globally, but the project does not declare it. The build can then behave differently on another machine.

Project setup:

~~~sh
npm install --save-dev sass
npm run build:css
~~~

The project dependency and npm script document the required compiler and command.

## 19. Keep generated output out of Sass source folders

~~~text
src/
  styles/
    app.scss
    components/
      _button.scss
public/
  css/
    app.css
    app.css.map
~~~

The CLI command can write to <code>public/css</code> while the hand-maintained Sass stays under <code>src/styles</code>. Decide whether output files are committed based on the project’s deployment workflow.

## 20. Example package scripts for a small standalone project

~~~json
{
  "name": "site-styles",
  "private": true,
  "scripts": {
    "build:css": "sass --style=compressed src/styles/app.scss public/css/app.css",
    "watch:css": "sass --watch src/styles/app.scss:public/css/app.css",
    "build": "npm run build:css"
  }
}
~~~

Install Sass separately with npm so the manifest and lockfile contain the package-managed dependency. The build script compiles once; the watch script is for local development.

## 21. Troubleshooting command paths

If Sass reports that it cannot find an input file:

1. Run the command from the package root.
2. Check the path and filename casing.
3. Confirm that the input is the application entry point.
4. Check that the output directory is valid for the project.
5. Verify Sass is installed in the package with <code>npm install</code> or <code>npm ci</code>.

If Sass cannot resolve a module used by the entry point, check its relative URL and any configured load paths.

## 22. Practice tasks

1. Add Sass to a small npm project and create a <code>build:css</code> script.
2. Add a separate watch script and run it while changing a partial.
3. Compile two named entry points to two different output files.
4. Choose whether source maps should be emitted for local builds and release builds.
5. Add a Sass build to a larger <code>build</code> script without masking failures.
6. Find the CSS output path referenced by the application and make the npm script write there.
7. If a bundler already owns Sass compilation, remove any redundant CLI compile for the same entry point.

## Official references

- [Install Sass](https://sass-lang.com/install/)
- [Dart Sass CLI](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass JavaScript package](https://sass-lang.com/dart-sass/)
- [npm scripts](https://docs.npmjs.com/using-npm/scripts/)
- [npm package.json scripts field](https://docs.npmjs.com/cli/v11/configuring-npm/package-json#scripts)
