# standalone-library Specification

## Purpose
TBD - created by archiving change commons-extract-standalone-repo. Update Purpose after archive.
## Requirements
### Requirement: Standalone Maven build
The repo SHALL build with `mvn` from its root with no parent POM. The coordinates MUST be `com.jbh:jbh-commons-lib:1.0.0-SNAPSHOT`, packaging `jar`. All versions MUST be explicit in `pom.xml`.

#### Scenario: Build without a parent
- **WHEN** `mvn verify` runs from the repo root on a machine without the finance repo
- **THEN** the build resolves all dependencies and plugins and ends with `BUILD SUCCESS`

#### Scenario: No parent element
- **WHEN** `pom.xml` is read
- **THEN** it has no `<parent>` element

### Requirement: Code moved unchanged
The Java sources, `module-info.java` and the two existing tests SHALL be identical to those in `jbh-personal-finance/jbh-commons-lib/`. Packages MUST stay `com.jbh.commons.*`. The JPMS module MUST stay `jbh.commons` and export `com.jbh.commons.api`, `com.jbh.commons.util` and `com.jbh.commons.exception`.

#### Scenario: Existing tests pass
- **WHEN** `mvn test` runs
- **THEN** `ApiResponseTest` (5 tests) and `CommonsArchitectureTest` (4 ArchUnit rules) pass, and nothing else runs

#### Scenario: Same file content
- **WHEN** each file under `src/` is compared with its source file in finance
- **THEN** the content is identical

### Requirement: Static analysis kept
The build SHALL run PMD (`maven-pmd-plugin` 3.25.0, `pmd-ruleset.xml`, `failOnViolation=true`, execution `pmd-check`) and SpotBugs (`spotbugs-maven-plugin` 4.8.6.2 with findsecbugs 1.13.0, `failOnError=false`, execution `spotbugs-check`) in the `verify` phase. The rule files MUST be copies of the finance files.

#### Scenario: Complexity check from the root
- **WHEN** `mvn -q pmd:check` runs from the repo root
- **THEN** it finds the ruleset and ends without violations

#### Scenario: Verify runs both tools
- **WHEN** `mvn verify` runs
- **THEN** `pmd-check` and `spotbugs-check` both run after the tests

### Requirement: No other JBH dependency
The library MUST contain no business or domain logic. It MUST NOT depend on any other JBH package. Runtime dependencies MUST be only Jackson annotations and databind.

#### Scenario: Dependency list
- **WHEN** `mvn dependency:tree` is read
- **THEN** the only `com.jbh` artifact is this one

