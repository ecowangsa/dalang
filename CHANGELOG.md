# Changelog

All notable changes to the Dalang plugin are documented here. Format loosely
follows [Keep a Changelog](https://keepachangelog.com/); versions use semver.

## [0.1.0] - 2026-05-31 - first plugin release (experimental)

First packaging of Dalang as a Claude Code plugin, distributed from the **`wayang`**
marketplace (`ecowangsa/dalang`). Skill behavior is the SDLC discipline accumulated
across ~6 runs on the author's own projects; this release only **packages** it.

### Added
- Plugin manifest (`.claude-plugin/plugin.json`, name `dalang`, version 0.1.0) and the
  `wayang` marketplace manifest (`.claude-plugin/marketplace.json`) hosting **both**
  `dalang` and `punakawan`. Install: `/plugin marketplace add ecowangsa/dalang` then
  `/plugin install dalang@wayang` (pulls `punakawan` automatically via the same marketplace).
- The skill now lives at `skills/dalang/SKILL.md` (required plugin layout). Typed command
  is namespaced **`/dalang:sdlc`**; description-based triggers fire unchanged.
- Inline **Preflight** section in `SKILL.md`: checks `git`, `superpowers`, `punakawan`,
  `feature-dev`, Node.js/`npx`, Playwright MCP, `/security-review` before Phase 0 and
  **fails loud** instead of degrading silently.
- `README.md` (honest framing + dependency table + install + known limits), this
  `CHANGELOG.md`, and an MIT `LICENSE`.

### Fixed
- **Stale Punakawan invocations** corrected throughout: `/punakawan` ->
  `/punakawan:panel` (the plugin-namespaced command). The Phase-1 threat-model gate that
  seeds Phase-4 SAST traceability had been calling a command that no longer resolves.
- **Dependency mislabels**: `feature-dev:code-architect` marked as a *subagent* (not a
  skill); `/code-review` and `/security-review` marked as *CLI built-ins* (not plugin skills).

### Known limitations (see README "Maturity & honest limits")
- **Zero confirmed external users.** Run only by the author across their own projects.
  External-user validation is an **open item**, not a cleared gate.
- Several rules are **provisional (n=1)** and live on the SKILL.md Watch list, not as
  settled discipline, pending an independent second run.
