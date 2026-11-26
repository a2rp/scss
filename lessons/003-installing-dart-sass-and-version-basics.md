# 003. Installing Dart Sass and Version Basics

## Learning goals

After this lesson, you should be able to:

- Identify Dart Sass as the primary Sass implementation and distinguish it from older implementations.
- Choose between a project-local npm installation, a standalone CLI installation, and the Sass embedded package.
- Install Sass in a project and verify which version is actually running.
- Read common semantic version constraints in package.json.
- Explain why a lockfile matters for repeatable installations.
- Upgrade Sass carefully and respond to deprecation warnings.

## Sass implementations

Sass is a language with implementations that compile it. The implementation matters because implementations can differ in supported features, maintenance status, interfaces, and performance.

### Dart Sass

Dart Sass is the primary Sass implementation. It receives new Sass features and is the recommended starting point for new projects.

Dart Sass can be used through:

- A standalone command-line executable.
- The npm package named sass, which provides a JavaScript distribution of Dart Sass.
- The npm package named sass-embedded, which communicates with the Dart Sass executable through the Embedded Sass protocol.

The npm package named sass is convenient and works across platforms supported by Node.js. The sass-embedded package can offer faster compilation by using the Dart VM, but it has a narrower platform support range. For ordinary learning projects, the sass package is a simple default.

### Node Sass and LibSass

The old node-sass package used LibSass, a separate C/C++ implementation. LibSass and Ruby Sass are no longer active implementations to choose for new projects. Existing projects may still depend on node-sass, but new projects should use Dart Sass.

A package name can be confusing: node-sass and sass are not interchangeable package names. Do not install node-sass for a new Sass project.

## Choose an installation style

### Project-local npm installation

For a project that is managed with npm, install Sass as a development dependency:

~~~sh
npm install --save-dev sass
~~~

The shorter equivalent is:

~~~sh
npm install -D sass
~~~

A local installation records Sass in package.json and the resolved package in package-lock.json. Anyone installing the project can then use the same declared dependency setup.

Run the local compiler through npm’s executable lookup:

~~~sh
npx sass --version
~~~

Compile a file with the same local executable:

~~~sh
npx sass src/styles.scss dist/styles.css
~~~

A project-local install is usually the best choice when the Sass version should travel with the project.

### Standalone command-line installation

The Sass team also provides standalone packages for common operating systems and CPU architectures. After downloading and extracting the matching package, run its Sass executable. On Windows the executable is named sass.bat. Add its directory to PATH if you want to run sass from any terminal location.

Verify the executable:

~~~sh
sass --version
~~~

A standalone install is useful when you need a system-wide compiler or do not have a Node-based project. Be aware that a globally available sass command may be a different version from a project-local Sass dependency. In a repository, prefer the project’s local executable when checking or compiling that repository.

### sass-embedded

A JavaScript tool or build system can use sass-embedded when it supports the Embedded Sass protocol. It may be faster for larger builds, but it is not necessary for most beginner projects. Follow the integration tool’s documentation before replacing sass with sass-embedded.

Do not install both packages without a reason. A build tool should use the implementation its configuration and platform support are intended for.

## Verify the tools and dependency

For an npm project, check Node.js and npm first:

~~~sh
node --version
npm --version
~~~

Then install and inspect Sass:

~~~sh
npm install -D sass
npx sass --version
npm ls sass
~~~

The commands answer different questions:

- node --version prints the active Node.js version.
- npm --version prints the active npm version.
- npx sass --version runs the Sass executable selected from the project environment.
- npm ls sass shows the installed Sass dependency in the current project.

If the global sass command and npx sass --version print different versions, the project-local command is the relevant one for that repository.

## Keep the dependency files

A small npm Sass project commonly has:

~~~text
project/
├── package.json
├── package-lock.json
├── src/
│   └── styles.scss
└── dist/
    └── styles.css
~~~

The package.json file records the requested dependency range and project scripts. The package-lock.json file records the resolved dependency tree used by npm.

Commit both package.json and package-lock.json for an npm application or learning project. Do not commit node_modules; it can be recreated from the dependency files.

For a clean install based on the lockfile, use:

~~~sh
npm ci
~~~

npm ci installs the dependency versions recorded in package-lock.json and fails if that lockfile disagrees with package.json. It is commonly used in continuous integration and clean build environments.

## Semantic version basics

A standard stable version has three numeric parts:

~~~text
major.minor.patch
1.108.2
~~~

The parts communicate the scale of a release:

- **Major**: may contain breaking changes.
- **Minor**: adds functionality while aiming to remain compatible within that major line.
- **Patch**: contains compatible fixes.

A version selector in package.json may allow more than one version. For example:

~~~json
{
  "devDependencies": {
    "sass": "^1.108.2"
  }
}
~~~

Common selectors include:

| Selector | General meaning |
| --- | --- |
| 1.108.2 | Request exactly this version. |
| ~1.108.2 | Allow compatible patch updates in the 1.108 line. |
| ^1.108.2 | Allow compatible updates that do not cross the next major version. |

For stable 1.x releases, the caret range shown above allows versions from 1.108.2 up to, but not including, 2.0.0. The exact resolved version is recorded in package-lock.json. This means the selector and lockfile serve related but different roles: package.json states what range is acceptable, while the lockfile records what this installation resolved.

Pre-release versions such as alpha, beta, or release-candidate builds may follow different matching rules and can change before a stable release. Do not use them by accident in a production dependency.

## Updating Sass safely

A careful update process is:

1. Check the current project version with npx sass --version or npm ls sass.
2. Read the Sass release notes and any breaking-change notes relevant to the new version.
3. Update Sass using the project’s package manager.
4. Review changes to package.json and package-lock.json.
5. Run the project’s Sass build command.
6. Read warnings and fix deprecated syntax instead of hiding warnings without a migration plan.
7. Commit the source and dependency-file changes together.

For example, to install a newer version allowed by the current package.json range, run:

~~~sh
npm update sass
~~~

To request a particular version explicitly, use:

~~~sh
npm install -D sass@1.108.2
~~~

The version in that command is only an example. Choose a version that is currently supported by the project and verify the result after installation.

Do not assume that upgrading a package means the browser will change behavior in the same way as a Sass language change. Sass compiles at build time; the browser receives the resulting CSS. A compiler update can change diagnostics or output details, so review the generated CSS when a change is important to your project.

## Deprecation warnings and compatibility

Dart Sass uses deprecation warnings to give projects time to move away from older syntax and APIs. A warning does not always mean compilation failed, but it does identify work that may be needed before a later release.

When a warning appears:

1. Identify whether it comes from your Sass source or from a dependency/build tool.
2. Read the warning’s migration guidance.
3. Update your code or the responsible dependency where possible.
4. Recompile and confirm the warning is resolved.

Some older JavaScript integrations call Sass’s legacy JavaScript API. The modern API replaced that API, and Dart Sass warns when the legacy API is used. If the warning comes from a bundler, check whether the bundler supports the modern API and upgrade or configure it according to its documentation. Avoid changing application code to silence a warning that is actually produced inside a build tool.

Sass also deprecates language features over time. For example, the old Sass @import rule is deprecated in favor of the module system, which uses @use and @forward. The dedicated module and migration lessons cover that change in detail.

## Common setup problems

### The sass command is not found

With a local npm install, use npx sass or a package script. A global command requires a global installation or a standalone executable directory on PATH.

### A different Sass version runs than expected

Run npx sass --version from the project directory. A global Sass executable may not match the local package.

### Sass is missing after cloning the project

Install dependencies from the repository root with npm ci when package-lock.json is present, or npm install when setting up or updating dependencies.

### The lockfile changes unexpectedly

Review the package manager command and the diff. An install or update may resolve newer versions allowed by the ranges in package.json. Keep dependency changes intentional and commit the lockfile with package.json.

### A build tool reports an API or compatibility warning

Check the tool’s supported Sass integration and version. The warning may come from the tool rather than from the project’s SCSS.

### A compile error appears after upgrading

Read the first Sass error and its file and line number. Check the Sass release notes and migration guidance. Fix one issue at a time, then rebuild.

## Key takeaways

- Dart Sass is the primary implementation and the recommended choice for new projects.
- The npm package sass is an easy project-local default; sass-embedded is an alternative when the tool and platform support it.
- Avoid starting new projects with node-sass or LibSass.
- Check the project-local version with npx sass --version.
- package.json records a dependency range; package-lock.json records the resolved installation.
- Use a lockfile-based install to make project setup repeatable.
- Treat deprecation warnings as migration information and verify compatibility when upgrading.

## Practice

1. In a new npm folder, install Sass as a development dependency.
2. Record the output of node --version, npm --version, npx sass --version, and npm ls sass.
3. Inspect package.json and package-lock.json. Identify the declared selector and resolved Sass version.
4. Run npm ci after deleting node_modules, then verify the Sass version again.
5. Compare the meaning of an exact version, a tilde range, and a caret range.
6. If a Sass warning appears, identify whether it comes from your stylesheet or a tool dependency.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="002-how-sass-fits-css-and-a-web-build.md">002 - How Sass Fits CSS and a Web Build</a></td><td><a href="../qnas/003-installing-dart-sass-and-version-basics.md">Q&amp;A</a></td><td><a href="../examples/003-installing-dart-sass-and-version-basics.md">Example</a></td><td><a href="004-compiling-scss-to-css.md">004 - Compiling SCSS to CSS</a></td></tr>
  </tbody>
</table>
