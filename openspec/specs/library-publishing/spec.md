# library-publishing Specification

## Purpose
TBD - created by archiving change commons-extract-standalone-repo. Update Purpose after archive.
## Requirements
### Requirement: Distribution target
`pom.xml` SHALL declare `distributionManagement` with repository id `github` and URL `https://maven.pkg.github.com/kkpa-jbh/jbh-commons-lib`.

#### Scenario: Publish target
- **WHEN** the Maven deploy goal runs with server `github` credentials
- **THEN** the artifact goes to `https://maven.pkg.github.com/kkpa-jbh/jbh-commons-lib`

### Requirement: CI settings with one credential
`.github/maven-settings.xml` SHALL define one server, id `github`, with username `${env.GITHUB_ACTOR}` and password `${env.GITHUB_TOKEN}`. It MUST NOT define `github-packages-read` or use `PACKAGES_READ_TOKEN`.

#### Scenario: Only server github
- **WHEN** `.github/maven-settings.xml` is read
- **THEN** it holds exactly one `<server>` with id `github`

### Requirement: Publish workflow
`.github/workflows/publish.yml` SHALL run on `push` to `main` and on `workflow_dispatch`. It MUST use Java 21 (temurin), run Maven with `-B -s .github/maven-settings.xml -DskipTests` and the deploy goal, and pass only `GITHUB_TOKEN` as secret. Permissions MUST be `contents: read` and `packages: write`.

#### Scenario: Push to main
- **WHEN** a commit lands on `main`
- **THEN** the workflow publishes `com.jbh:jbh-commons-lib:1.0.0-SNAPSHOT` to GitHub Packages

#### Scenario: No extra secret
- **WHEN** the workflow file is read
- **THEN** it does not mention `PACKAGES_READ_TOKEN` or the `github` Maven profile

### Requirement: Repo guidance files
The repo SHALL have `README.md`, `.gitignore` (`target/`, `.DS_Store`, `.idea/`, `*.iml`, `.env`) and `CLAUDE.md`. `CLAUDE.md` MUST keep the pipeline config block and MUST contain a `## System map sync (../jbh-deploy/README.md)` section.

#### Scenario: Sync triggers
- **WHEN** `CLAUDE.md` is read
- **THEN** the sync section lists these triggers: groupId, artifactId or version change; a new dependency on another JBH package; a change of the publish workflow or package URL

