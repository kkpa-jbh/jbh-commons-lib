## 1. Copy the code (no dependencies)

- [x] 1.1 Copy `jbh-personal-finance/jbh-commons-lib/src/` to `src/` (12 main files incl. `module-info.java`, 2 test files). No edit. No `target/`, no `.DS_Store`. Check with `diff -r` (design D1).

## 2. Build and quality config (no dependencies; touches only root config files)

- [x] 2.1 Write the standalone `pom.xml`: coordinates, properties, dependencies, compiler and surefire config, `junit-bom` import, no `<parent>` (design D2).
- [x] 2.2 Add PMD and SpotBugs plugin config and executions to `pom.xml` (design D3). Depends on 2.1.
- [x] 2.3 Copy `pmd-ruleset.xml`, `spotbugs-exclude.xml`, `spotbugs-include.xml` from `jbh-personal-finance/` unchanged (design D3).
- [x] 2.4 Add `distributionManagement` to `pom.xml` (design D4). Depends on 2.1.

## 3. CI files (no dependencies; touches only `.github/`)

- [x] 3.1 Create `.github/maven-settings.xml` with the one server `github` (design D4).
- [x] 3.2 Create `.github/workflows/publish.yml` (design D4).

## 4. Docs and repo files (no dependencies; touches only root docs)

- [x] 4.1 Create `.gitignore` (design D5).
- [x] 4.2 Create `README.md` (design D5).
- [x] 4.3 Complete `CLAUDE.md`: keep the pipeline block, add SpotBugs note, overview, build commands, JPMS exports, rules, System map sync section (design D5).

## 5. Verify (depends on 1, 2, 3, 4)

- [x] 5.1 The leader runs `mvn test`, `mvn -q pmd:check`, `mvn verify`, `mvn install` and the file checks of the design test contract. Compare with the expected results there.

Size estimate: 24 files, ~1020 changed lines (tests and docs included; 14 copied Java files ~515 lines, config files ~500 lines)
