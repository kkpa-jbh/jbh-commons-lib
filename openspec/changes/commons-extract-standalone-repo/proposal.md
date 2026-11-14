## Why

`jbh-commons-lib` is a module of the `jbh-personal-finance` multi-module build. The new
`jbh-notification` service and other repos need it as a normal dependency. They cannot
build it from the finance repo. It must be its own repo, published to GitHub Packages
(JBH-74, section 1).

## What Changes

- Copy the sources, tests and `module-info.java` of `jbh-personal-finance/jbh-commons-lib/` into this repo root. Same packages `com.jbh.commons.*`. No logic change.
- Make `pom.xml` standalone: no `<parent>`. Coordinates `com.jbh:jbh-commons-lib:1.0.0-SNAPSHOT`. Pin the Java, compiler, surefire, Jackson and JUnit versions the parent gave before.
- Copy PMD and SpotBugs from the finance parent POM, with `pmd-ruleset.xml`, `spotbugs-exclude.xml` and `spotbugs-include.xml`.
- Add `distributionManagement` (id `github`, URL `https://maven.pkg.github.com/kkpa-jbh/jbh-commons-lib`).
- Add `.github/maven-settings.xml` and `.github/workflows/publish.yml` (only `GITHUB_TOKEN`).
- Add `README.md`, `.gitignore`, and complete `CLAUDE.md` (keep the pipeline config block; add the System map sync rule).

## Capabilities

### New Capabilities
- `standalone-library`: the repo builds, tests and passes static analysis on its own, with no parent POM, and keeps the same public API and JPMS exports.
- `library-publishing`: CI publishes `com.jbh:jbh-commons-lib:1.0.0-SNAPSHOT` to GitHub Packages with `GITHUB_TOKEN` only.

### Modified Capabilities
(none)

## Impact

- Only this repo changes. No database. No new classes.
- Later sections of JBH-74 make other repos read this package.

## Not included

These are other sections of JBH-74, outside this change:
- Any edit in `jbh-personal-finance` (removing the module, switching to the package).
- Any edit in `jbh-notification`.
- `jbh-deploy/README.md` (system map) and the root `kkpa-jbh/CLAUDE.md`.
- New unit tests. The system plan excludes them.
