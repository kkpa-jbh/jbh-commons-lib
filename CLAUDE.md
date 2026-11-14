# CLAUDE.md

## Overview

`jbh-commons-lib` is a shared technical library (Java 21) for the JBH services.
Coordinates: `com.jbh:jbh-commons-lib:1.0.0-SNAPSHOT`. It is published to GitHub Packages.

**Rules:**
- No business or domain logic.
- No dependency on other JBH code (it would create a build cycle).

JPMS module `jbh.commons` exports `com.jbh.commons.api`, `com.jbh.commons.util`, `com.jbh.commons.exception`.

## Build

- `mvn test`: unit and ArchUnit tests
- `mvn verify`: tests, PMD and SpotBugs
- `mvn -q pmd:check`: complexity check
- `mvn install`: install to the local `~/.m2`

## System map sync (`../jbh-deploy/README.md`)

This repo is one part of the JBH system. The map of how the JBH parts fit together
lives in `../jbh-deploy/README.md` (the `jbh-deploy` repo, a sibling of this one in the `kkpa-jbh` folder).

**Rule:** when a change in this repo affects the system map, update `../jbh-deploy/README.md`
in the same task. Change only the rows or lines that are affected. Then commit it in the
`jbh-deploy` repo (a separate commit there; never one commit across repos).

Triggers for this repo:
- Change the groupId, artifactId or version (this changes the build order)
- Add a dependency on another JBH package (it would create a build cycle)
- Change the publish workflow or the package URL

If you are not sure a change counts, read `../jbh-deploy/README.md` and check whether any line is now wrong.

## Pipeline config (kkpa-orchestrate-change)

- **Modules-Root:** `.` (one module: `commons` = the whole repo, root files included)
- **Doc-Pack:** `CLAUDE.md`
- **Complexity-Check:** `mvn -q pmd:check` (uses `pmd-ruleset.xml`: CyclomaticComplexity, NPathComplexity)
- **Design-Gates:** `database`
- **Git-Strategy:** `auto`
- **Note:** SpotBugs also runs at `verify` (`failOnError=false`: it reports, it does not fail the build)
