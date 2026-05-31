# Dalang - a business-flow SDLC skill for Claude Code

> **dalang** (Javanese): the single puppeteer who runs the entire *wayang* show -
> voicing every character in turn, ordering the scenes, leading the gamelan.

**Dalang** is a [Claude Code](https://claude.com/claude-code) plugin that drives one
feature slice from **idea to release** through six SDLC phases - Discovery -> Design ->
Build -> QA -> Security -> Docs/Release - with iterative loop-backs and **hard human
commit gates**. It is the *executor* counterpart to its sibling
[**Punakawan**](https://github.com/ecowangsa/punakawan) (the *advisor*).

> [!IMPORTANT]
> **One model, many hats - not a team of independent reviewers.** Dalang voices each
> business role (PM, Architect, DEV, QA, Security, TW) *in turn* with one model. The
> value is the **checklist, the phase gates, and the honest artifacts** - never the
> illusion of independent experts. A "review" and the "developer" it reviews are the
> same weights wearing different prompts. Dalang says so out loud, on purpose.

> [!WARNING]
> **Status: experimental (v0.1.x).** Dalang has been run ~6 times by its author across
> their *own* projects. **Zero confirmed external users.** Several rules are provisional
> (n=1, single self-graded run) and labeled as such. Treat it as a sharp checklist that
> is still earning its evidence - not battle-tested discipline. See
> [Maturity & honest limits](#maturity--honest-limits).

---

## Install

```text
/plugin marketplace add ecowangsa/dalang
/plugin install dalang@wayang
```

Installing `dalang@wayang` **automatically pulls `punakawan@wayang`** (a declared
same-marketplace dependency). The repo `ecowangsa/dalang` *is* the `wayang` marketplace
catalog and hosts both puppets; you can also `/plugin install punakawan@wayang` on its own.

> [!NOTE]
> **Already use Punakawan, or have a local copy of either?** This `wayang` catalog is the
> single source for both puppets - add **only** `ecowangsa/dalang`.
> - If you previously ran `/plugin marketplace add ecowangsa/punakawan`, remove that
>   marketplace first - it also names itself `wayang`, and two marketplaces with the same
>   name collide. Punakawan installs from *this* catalog as `punakawan@wayang`.
> - If you have an old hand-installed copy under `~/.claude/skills/dalang` or
>   `~/.claude/skills/punakawan`, remove it so it doesn't shadow or compete with the plugin.

Dalang then leans on several **other** plugins and tools that it does **not** auto-install
(they live in a different marketplace, or are CLI/runtime, which a plugin cannot declare
as dependencies). Install these yourself - see the next section.

---

## Prerequisites (what Dalang needs to actually run)

Dalang's phases map onto external skills, plugins, MCP servers, and CLIs. A missing one
mostly **degrades silently** - which is the worst failure mode for a tool whose whole
value is *honest reporting*. So Dalang runs an **inline preflight before Phase 0** and
tells you what is missing. Set these up first:

| Dependency | Kind | Required? | Install / source | If absent |
| --- | --- | --- | --- | --- |
| `git` | CLI | **Required** | system / Xcode CLT / `brew`/`apt` | The sandbox / branch / diff / commit-gate **safety spine** breaks. Dalang cannot guarantee a reversible run. |
| **superpowers** | plugin | **Required** | `/plugin install superpowers@claude-plugins-official` | Phase 0 `brainstorming`, Phase 2 `subagent-driven-development` + `test-driven-development`, Phase 5 `verification-before-completion` + `finishing-a-development-branch` all vanish - most of the execution spine. |
| **punakawan** | plugin | **Required**¹ | auto-installed with `dalang@wayang` | Phase-0 contested-direction gate + **Phase-1 threat model** (the source of the `T1, T2…` MUSTs that Phase-4 SAST must close) cannot run. The shift-left security spine collapses. |
| **feature-dev** | plugin | **Required**² | `/plugin install feature-dev@claude-plugins-official` | Phase-1 `code-architect` (a *subagent*) is absent; architecture is improvised. |
| **Node.js / `npx`** | runtime | Required for web slices | [nodejs.org](https://nodejs.org) / `nvm` / `brew` | **Both** MCP servers spawn via `npx`; without Node, Phase-3 E2E, Phase-4 browser-DAST, and Phase-5 `context7` silently become **no-ops**. First run also needs network. |
| **Playwright MCP** | MCP | Required for web E2E/DAST | `/plugin install playwright@claude-plugins-official` | No browser E2E or browser-level security probes. (Off-web slices retarget - see SKILL.md Phase 3/4.) |
| `/security-review` | CLI built-in | **Required** | ships with Claude Code (no plugin) | The Phase-4 SAST driver. |
| `/code-review` | CLI built-in / `requesting-code-review` (superpowers) | Required | built-in, or via superpowers | Phase-3 static-review STAGE. One of the two covers it. |
| **context7 MCP** | MCP | Optional | `/plugin install context7@claude-plugins-official` | Only Phase-5 TW loses live library-doc lookup (TW is itself opt-in). |
| `python3` (>=3.9) | runtime | Optional | system / `brew` | Only the bundled `md2docx.py` (zero-dep Markdown->.docx for Word shops); falls back to `pandoc`/markdown. |
| `curl` | CLI | Optional | system | Protocol-level DAST probes (request smuggling / SSRF) Chromium can't issue. Web slices only. |
| `gh` | CLI | Optional | `brew install gh` / `apt` | Only the "open a PR" path at the release gate. |
| `pandoc` / `python-docx` | CLI/lib | Optional | `brew`/`pip` | Richer `.docx` than the bundled fallback. |

¹ Auto-installed as a `wayang` same-marketplace dependency - you should not need to do
anything. ² From the official marketplace; cross-marketplace deps are not auto-installed
by design (no silent third-party installs).

**One-time setup for a clean machine:**

```text
# the official marketplace (if not already added)
/plugin marketplace add anthropics/claude-plugins-official
/plugin install superpowers@claude-plugins-official
/plugin install feature-dev@claude-plugins-official
/plugin install playwright@claude-plugins-official
/plugin install context7@claude-plugins-official   # optional

# the wayang marketplace (dalang + punakawan)
/plugin marketplace add ecowangsa/dalang
/plugin install dalang@wayang                       # pulls punakawan automatically
```

…and have `git` + Node.js on PATH.

---

## How to use it

Dalang fires on intent, not just its name. Start a feature slice and say things like
*"build X end-to-end"*, *"new slice"*, *"tambah feature"*, *"implement Y"* - Dalang
engages. You can also invoke it explicitly:

```text
/dalang:sdlc <describe the feature slice>
```

It then runs the preflight, walks the phases, parks at **human gates** (plan sign-off,
UAT, commit/release) for your go/no-go, and **loops back** when QA/Security finds a
defect or UAT rejects the result. Nothing is committed, pushed, or deployed without an
explicit human gate.

---

## The six phases (what each does)

| Phase | Roles | What happens |
| --- | --- | --- |
| **0 - Discovery & Intent** | Sponsor/CEO (gate), PM/BA | A 3-line Sponsor artifact (goal / measurable success / scope guardrails) + requirements & acceptance criteria via `brainstorming`. |
| **1 - Design & Architecture** | SA/Architect, Security (shift-left), Plan sign-off | `code-architect` + `writing-plans` produce the plan; `/punakawan:panel --preset safeguard` produces a threat model with **numbered MUSTs (T1, T2…)**; human signs off. |
| **2 - Build** | DEV | `subagent-driven-development` + `test-driven-development` - code + whitebox tests. |
| **3 - QA / Test pyramid** | Whitebox, Blackbox, E2E/UI, Static review | Unit/integration + contract + Playwright E2E + a **multi-dimensional** code review. |
| **4 - Security verification** | SAST, concurrency/seam probes, DAST | `/security-review` on the diff (every Phase-1 MUST-ID closed as a register row) + dynamic probes. |
| **5 - Docs & Release** | TW (opt-in), Verification, UAT, Commit/Release | Verification must be green; UAT checks *wanted* vs *built*; **hard human commit gate**. |

Findings are **adversarially verified** before they gate, severities use one P0-P3 scale,
and any open P0/P1 bounces back to DEV before the commit gate.

### Artifacts you get

Five small live tables, designed for an in-session run (no fake sprints/velocity):
**phase log, run-board, defect register, loop-back log, traceability**. They make the run
replayable and the reporting honest.

---

## Relationship to Punakawan

Two **separate citizens** of one `wayang` ecosystem:

| | Punakawan | Dalang |
| --- | --- | --- |
| Nature | **advisory** - returns a verdict, never executes | **executor** - produces real code/tests/docs |
| Units | **lenses** (ways of reasoning) | **business roles** (job titles) |
| Shape | small parallel panel + gate (cap 5) | iterative role pipeline with loop-backs |

Dalang **calls** `/punakawan:panel` as an advisory gate at a hard fork it cannot
adjudicate alone (a design/security choice, a loop-back fix that crosses the slice
boundary, a governance call) - but skips trivial forks. They never edit each other's files.

---

## Maturity & honest limits

This is the part most tools omit. Dalang states it plainly:

- **Lineage:** promoted from an internal recipe to a skill on 2026-05-29 after a Punakawan
  deliberation; packaged as this plugin on 2026-05-31. Run across ~6 slices on the author's
  own projects (Nusa, comic-web, Valkyrie).
- **Zero confirmed external users.** No one outside the author has run a full-lane slice on
  a project the author has never seen. The original promotion criterion that required a
  *real external pull* was softened under author-internal conditions (documented, with the
  dissent, in `skills/dalang/references/promotion-history.md`). **External-user validation
  is an open item, not a cleared gate.** If you run Dalang on your own project, telling us
  what broke is the single most valuable contribution.
- **Provisional (n=1) rules** live on the SKILL.md *Watch list*, not in the settled body -
  e.g. off-web E2E/Blackbox retargeting, DAST-without-a-web-surface, the pre-Build
  foundation health precheck. They are useful defaults, not law, until a second independent
  run confirms them.
- **Honest automation limit:** deep pentest (Burp/ZAP, fuzzing) is *pointed to*, not
  automated. What Dalang automates is `/security-review` (static) + light Playwright/`curl`
  probing (dynamic).

---

## Contributing / validation wanted

The most useful thing you can do: **run a real slice and report back.** Open an issue with
what phase you reached, what broke, and what the preflight said. That is the external pull
this tool's own discipline says it still needs.

## License

MIT - see [LICENSE](LICENSE). © 2026 Ecowangsa.
