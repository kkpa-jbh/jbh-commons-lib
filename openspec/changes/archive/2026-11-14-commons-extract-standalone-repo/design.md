## Context

`jbh-commons-lib` lives in `jbh-personal-finance/jbh-commons-lib/` as a module with parent
`com.jbh:jbh-personal-finance`. The parent gives it: Java 21, the compiler plugin, surefire
3.5.4, PMD, SpotBugs, and the Jackson and JUnit versions (from the Quarkus BOM).
This change moves the module into its own repo and publishes it. It is a copy plus build and
CI files. There is no logic change and no database change.

## Goals / Non-Goals

**Goals**
- Standalone `pom.xml`: `com.jbh:jbh-commons-lib:1.0.0-SNAPSHOT`, no parent.
- Same behaviour as before: same API, same JPMS exports, same tests, same PMD and SpotBugs checks.
- Publish to GitHub Packages from CI with `GITHUB_TOKEN` only.

**Non-Goals**
- Any edit in `jbh-personal-finance`, `jbh-notification`, `jbh-deploy/README.md`, or the root `kkpa-jbh/CLAUDE.md`.
- New classes, new tests, version upgrades, refactors.

## Responsibilities

none (no new class; no existing class gets a new job; Java files are copied as they are).

## Decisions

### D1. Files copied as-is
Copy `src/main` (12 files, including `module-info.java`) and `src/test` (2 files) from
`jbh-personal-finance/jbh-commons-lib/`. Do not copy `target/` or `.DS_Store`.
Module `jbh.commons` exports `com.jbh.commons.api`, `.util`, `.exception`.

### D2. Pinned versions in the standalone pom
Source of the old values: `jbh-personal-finance/pom.xml` (Quarkus `3.33.4.1`, Java 21,
compiler 3.12.1, surefire 3.5.4, PMD 3.25.0) and the Quarkus BOM file
`~/.m2/repository/io/quarkus/platform/quarkus-bom/3.33.4.1/quarkus-bom-3.33.4.1.pom`.

| Item | Version | How found |
|---|---|---|
| Java | 21 (`maven.compiler.release`) | finance `java.version` |
| maven-compiler-plugin | 3.12.1, `parameters=true`, `enablePreview=false` | finance `pluginManagement` |
| maven-surefire-plugin | 3.5.4, `useModulePath=true` | finance `pluginManagement` |
| maven-pmd-plugin | 3.25.0 | finance property `maven-pmd-plugin.version` |
| spotbugs-maven-plugin | 4.8.6.2, findsecbugs 1.13.0 | finance `pluginManagement` |
| jackson-databind | 2.21.7 | BOM line for `jackson-databind` |
| jackson-annotations | 2.21 (no patch release) | BOM line for `jackson-annotations` |
| junit-jupiter | 6.0.3 | BOM line for `junit-jupiter` |
| archunit-junit5 | 1.3.0 | the old module pom (explicit) |

`jackson-core` (needed by `module-info`) comes with `jackson-databind` 2.21.7.
`junit-platform-engine` is managed to 6.0.3 in the BOM. ArchUnit 1.3.0 would pull an older
engine. So the pom imports `org.junit:junit-bom:6.0.3` in `dependencyManagement`, which keeps
the same engine as in finance. Both exist in `~/.m2`.
Dependency versions use properties (`jackson.version`, `jackson-annotations.version`,
`junit.version`, `archunit.version`). The Quarkus BOM is **not** imported: the lib is
not a Quarkus module, and the import would hide the pinned versions.
Rejected: keep the Quarkus BOM import (large, couples the lib to Quarkus upgrades).

Dropped from the parent because this module does not use them: failsafe, jandex, the Quarkus
plugin, Lombok, logging versions, `DOCKER_API_VERSION` env var, the `**/logging/**` PMD exclude
(no such package here).

### D3. PMD and SpotBugs
PMD: `maven-pmd-plugin` with `<rulesets><ruleset>${project.basedir}/pmd-ruleset.xml</ruleset></rulesets>`,
`failOnViolation=true`, `printFailingErrors=true`, `targetJdk=21`, execution `pmd-check` goal
`check` in phase `verify`. The finance ruleset path was `/pmd-ruleset.xml`; the explicit
`${project.basedir}` path makes `mvn -q pmd:check` work from the repo root.
SpotBugs: `effort=Max`, `threshold=Low`, `failOnError=false`, `excludeFilterFile=spotbugs-exclude.xml`,
`includeFilterFile=spotbugs-include.xml`, findsecbugs 1.13.0, execution `spotbugs-check`
goal `check` in phase `verify`.
Copy `pmd-ruleset.xml`, `spotbugs-exclude.xml`, `spotbugs-include.xml` from `jbh-personal-finance/`
byte for byte. The ruleset name text ("JBH Personal Finance") stays; no edit, so the copy is easy to diff.

### D4. Publishing
`distributionManagement` repository: id `github`, name `GitHub kkpa-jbh Maven Packages`, URL
`https://maven.pkg.github.com/kkpa-jbh/jbh-commons-lib`. No `github` profile: the lib reads no
other JBH package (`jbh-gateway-client` needs the profile only to read contracts).
`.github/maven-settings.xml`: one server `github`, `${env.GITHUB_ACTOR}` and `${env.GITHUB_TOKEN}`.
`.github/workflows/publish.yml`: mirror of `jbh-gateway-client`: triggers `workflow_dispatch` and
push to `main`; permissions `contents: read`, `packages: write`; checkout v4, setup-java v4 (Java 21,
temurin, cache maven); one step running Maven with `-B -s .github/maven-settings.xml -DskipTests`
and the deploy goal, env `GITHUB_TOKEN` only. Drop `-U` and `-Pgithub` (no SNAPSHOT of another repo to refresh).
Tests are skipped in CI publish like in `jbh-gateway-client`.

### D5. CLAUDE.md, README.md, .gitignore
- `CLAUDE.md`: keep the `## Pipeline config (kkpa-orchestrate-change)` block unchanged. Add one
  note line that SpotBugs also runs at `verify` (`failOnError=false`). Add: project overview
  (Java 21 shared technical library, no business logic), build commands (`mvn test`, `mvn verify`,
  `mvn -q pmd:check`, `mvn install`), JPMS exports, the rule "no business or domain logic and no
  dependency on other JBH code", and `## System map sync (../jbh-deploy/README.md)` shaped like
  the one in `jbh-gateway-client/CLAUDE.md`. Triggers: groupId, artifactId or version change; a new
  dependency on another JBH package (it would create a build cycle); a change of the publish
  workflow or package URL.
- `README.md`: short. What it is, coordinates, exports, build, install and use as a dependency
  (CI needs the `github-packages-read` repository and a read token), publish.
- `.gitignore`: `target/`, `.DS_Store`, `.idea/`, `*.iml`, `.env`.

## Test contract

Test-First blueprint does not apply: the system plan excludes new unit tests and no class is
new, so there is no blueprint group in tasks.md.

The two existing tests move unchanged. Expected results after the move, run from the repo root:

| Command | Expected |
|---|---|
| `mvn test` | `BUILD SUCCESS`. `ApiResponseTest`: 5 tests pass. `CommonsArchitectureTest`: 4 ArchUnit rules pass. Total: 9 tests, 0 failures, 0 skipped. Same counts as in finance. |
| `mvn -q pmd:check` | exit code 0, no output (no violations; the ruleset file is found). |
| `mvn verify` | `BUILD SUCCESS`. `pmd-check` and `spotbugs-check` run after the tests. SpotBugs may print warnings but must not fail the build. |
| `mvn install` then `ls ~/.m2/repository/com/jbh/jbh-commons-lib/1.0.0-SNAPSHOT/` | `jbh-commons-lib-1.0.0-SNAPSHOT.jar` and `.pom` exist. The jar holds `module-info.class` and `com/jbh/commons/**`. |
| `mvn dependency:tree` | compile scope: only `jackson-databind`, `jackson-core` 2.21.7 and `jackson-annotations` 2.21. No other `com.jbh` artifact. |

File checks (the leader or the user run them):
- `diff -r` of `src/` against `jbh-personal-finance/jbh-commons-lib/src/` prints nothing.
- `diff` of the three quality files against the finance originals prints nothing.
- `pom.xml` has no `<parent>`.
- `.github/maven-settings.xml` has one `<server>`, id `github`. `publish.yml` does not contain `PACKAGES_READ_TOKEN`.

The publish workflow cannot be checked locally. It is checked by the first run on `main` after
the push (not in this change).

## Database design

none. No table, column, index, constraint, view or migration.

## Concurrency

none. This is a build and packaging change. No shared state, no async code, no read-modify-write.
The CI workflow runs one job.

## Risks / Trade-offs

- **JUnit 6 with ArchUnit 1.3.0**: finance runs this pair already. The `junit-bom` import keeps the platform engine at 6.0.3. If the build shows an engine mismatch, report it to the leader; do not change the tests.
- **PMD ruleset path**: the old value `/pmd-ruleset.xml` is replaced by `${project.basedir}/pmd-ruleset.xml`. If `pmd:check` does not find the file, the leader reports it back.
- **Offline `~/.m2`**: all pinned versions exist in `~/.m2`, except plugins not yet used here (SpotBugs, PMD) which finance already downloads.

## Migration Plan

1. Land this repo with the copy. Nothing in finance changes yet.
2. The user pushes `main` and the branch; the first CI run publishes the 1.0.0-SNAPSHOT package.
3. Later sections of JBH-74 switch the other repos to the package. Rollback: delete this repo's
   package; finance still has its own module until its section removes it.
