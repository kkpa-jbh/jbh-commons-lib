# CLAUDE.md

## Pipeline config (kkpa-orchestrate-change)

- **Modules-Root:** `.` (one module: `commons` = the whole repo, root files included)
- **Doc-Pack:** `CLAUDE.md`
- **Complexity-Check:** `mvn -q pmd:check` (uses `pmd-ruleset.xml`: CyclomaticComplexity, NPathComplexity)
- **Design-Gates:** `database`
- **Git-Strategy:** `auto`
