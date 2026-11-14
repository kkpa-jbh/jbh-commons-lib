# jbh-commons-lib

Shared technical library for JBH services (Java 21). It has no business or domain logic.
It does not depend on any other JBH code.

## Coordinates

```xml
<dependency>
  <groupId>com.jbh</groupId>
  <artifactId>jbh-commons-lib</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</dependency>
```

## JPMS module `jbh.commons`

Exports: `com.jbh.commons.api`, `com.jbh.commons.util`, `com.jbh.commons.exception`.

## Build

```
mvn test            # unit and ArchUnit tests
mvn -q pmd:check    # complexity rules (pmd-ruleset.xml)
mvn verify          # tests + PMD + SpotBugs
mvn install         # install to the local ~/.m2
```

## Use as a dependency

The package is on GitHub Packages: `https://maven.pkg.github.com/kkpa-jbh/jbh-commons-lib`.
In CI, add a repository with id `github-packages-read` for this URL. Give it a token
with `read:packages` in your Maven settings. Locally, run `mvn install` here first.

## Publish

The workflow `.github/workflows/publish.yml` deploys the package on every push to `main`
(or by hand with `workflow_dispatch`). It uses only `GITHUB_TOKEN`.
