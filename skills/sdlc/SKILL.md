---
name: sdlc
description: Use when starting any feature slice or significant code change that benefits from phased SDLC discipline - PM brainstorming, design + threat model, TDD build, QA pyramid (whitebox + E2E), SAST + DAST, docs + human commit gate. Drive this whenever the user says "feature slice", "new slice", "tambah feature", "implement X end-to-end", "build Y", or anything indicating a fresh development cycle on a project, even if they do not mention "dalang" by name. Especially valuable when the change crosses multiple files, touches both backend and UI, has any security surface, or needs visible artifacts (phase log, defect register, traceability) for honest reporting.
---

# dalang - a business-flow SDLC skill

> **dalang** (Jv.): the single puppeteer who runs the entire wayang show - voicing
> every character in turn, ordering the scenes, leading the gamelan. That is exactly
> what this skill is: one model voicing many business roles in turn to carry one
> piece of work from idea to release. It is the honest name, because one dalang
> moving all the puppets is one model wearing all the hats - not a team of
> independent experts.

## What dalang is

dalang is a phased SDLC skill that drives a feature slice from idea to release
through six phases (Discovery -> Design -> Build -> QA -> Security ->
Docs/Release) with iterative loop-backs and hard human commit gates. It is the
executor counterpart to punakawan (advisory). One model voices each business
role in turn - the honest name, because one dalang moving all the puppets is
one model wearing all the hats.

The history of how dalang was promoted from a recipe to a skill (including the
original five-criterion gate, the Punakawan deliberation that introduced and
revised criterion (e), and the load-bearing Bagong dissent) is preserved in
`references/promotion-history.md` for future readers who want to understand
why the skill exists and what disciplines its design encodes.

**How a new lesson enters this skill (two tracks - so this file does not over-fit
one run).** *Default:* an observation folds into SKILL.md only after **>=2 confirmations
across runs**; until then it lives on the **Watch list** (end of file), not as settled
discipline. *Exception:* a **load-bearing catch** - a single run where the missing rule
would have let a real **P0/P1 slip** - may fold from one run via override (the mechanism
that elevated the concurrency probe after run #2). Mark single-run folds with their
provenance so the next reader can calibrate; a self-graded RED->GREEN is *necessary, not
sufficient* - real validation is an independent second run.

## Relationship to punakawan (the boundary that keeps both clean)

dalang and punakawan are **separate citizens** that respect each other's lane:

| | punakawan | dalang |
| --- | --- | --- |
| Nature | advisory - returns a verdict, never executes | executor - produces real code/tests/docs |
| Units | **lenses** (ways of reasoning), never job titles | **business roles** (job titles) |
| Shape | small parallel panel + gate, cap 5 | iterative role pipeline with loop-backs |

Hard rules so neither drifts:
- dalang lives in **its own folder**. It never edits punakawan's files (its
  `skills/panel/SKILL.md` / `roles.md`), and it never reintroduces job-title names
  into punakawan's lens catalog.
- dalang **calls** `/punakawan:panel` as an *advisory gate* at a hard fork the controller
  cannot adjudicate solo - a design or security choice, a **loop-back fix that crosses
  the slice boundary**, or a process/governance call. **But skip trivial / one-dimensional
  forks** (punakawan's own rule): the panel earns its cost only on a real multi-tradeoff
  decision, and over-invoking it trains the gate to be ignored. Calling it does not change
  how punakawan works.
- punakawan stays lens-based and advisory; dalang stays role-based and executive.

## The flow is iterative, not a one-way pipeline

Phases have a default order but **gates can bounce work back**. That is what makes it
professional rather than waterfall:

- QA or Security finds a **defect** -> back to **DEV** (do not proceed).
- **UAT / End User** rejects -> back to **PM / Design** (matching the spec is not the
  same as matching what was wanted).
- A **threat model** opens a new risk -> back to **Architecture**.

**A loop-back fix is itself a fork when it crosses the slice boundary.** The loop-back rule
(any open P0/P1 -> back to DEV) says *when* to bounce, not that you may silently expand the
blast radius. If clearing a finding means editing a file **outside the slice's planned set**
(shared / pre-existing infra) **or** making a real design choice, **STOP and escalate first** -
human re-scope and/or `/punakawan:panel` - before writing the fix. This discipline lives here, not in
the user's CLAUDE.md. (field-notes: two Slice-B P1 fixes hit this.)

**Tailor to change size.** A one-line fix does not need the full pipeline. Two lanes:
- **lite** - PM (clarify) -> DEV (TDD) -> static review -> verification -> commit gate.
- **full** - every phase below, including security shift-left and the QA pyramid.

## Preflight - confirm the toolchain before Phase 0 (run once, fail loud)

dalang's phases map onto external skills, plugins, MCP servers, and CLIs. A missing one
mostly **degrades silently** - the worst failure for a skill whose whole value is *honest
reporting* (it would file artifacts claiming coverage a missing stage never gave). So
**before Phase 0, check the toolchain and surface what's absent with the exact install
command - never silently skip a stage.**

| Need | Kind | If absent |
| --- | --- | --- |
| `git` | CLI | the sandbox / branch / diff / commit-gate **safety spine** breaks - **stop** |
| `superpowers` | plugin | Phase 0 `brainstorming`, Phase 2 `subagent-driven-development` + `test-driven-development`, Phase 5 `verification-before-completion` + `finishing-a-development-branch` are all gone - **stop** |
| `punakawan` | plugin | Phase 0 contested-direction gate + **Phase 1 threat model** (the source of the T1/T2 MUSTs Phase 4 must close) cannot run - **stop** on any security-touching slice |
| `feature-dev` | plugin | Phase 1 `code-architect` (a subagent) absent - architecture is improvised |
| Node.js / `npx` | runtime | **both** MCP servers fail to spawn -> Phase 3 E2E, Phase 4 browser-DAST, Phase 5 `context7` silently become **no-ops** (web slices) |
| Playwright MCP | MCP | no browser E2E / DAST (off-web slices retarget - see Phase 3 / 4) |
| `/security-review` | CLI built-in | present without any plugin; it is the Phase 4 SAST driver |
| `python3`, `curl`, `gh`, `context7`, `pandoc` | misc | degrade gracefully (md2docx fallback, protocol-DAST, PR path, lib-docs) |

Print the missing set + its install commands at run start; only then enter Phase 0. (This
is an inline check, **not** a separate skill - a doctor-skill that is itself missing helps
no one.) Rows whose consequence says **stop** are hard-required (do not proceed without
them - the `punakawan` stop is conditional on a security-touching slice); everything else
degrades gracefully. **Exact install commands live in the plugin README.**

## Role -> skill / tool mapping

Each role is marked **STAGE** (does real work via a skill/tool) or **GATE** (a cheap
checkpoint - human, inline, or advisory; *not* a spawned subagent).

### Phase 0 - Discovery and Intent
| Role | Type | Maps to | Output |
| --- | --- | --- | --- |
| Sponsor / CEO | GATE | human; `/punakawan:panel` if direction is contested | goal, success metrics, scope guardrails |
| PM / BA | STAGE | `brainstorming` | requirements, user stories, acceptance criteria |

**State-trigger discipline.** When a requirement defines a UI/UX state (empty,
no-results, error, loading), the acceptance criterion must answer **when does
this state actually trigger** given the backend's real semantics, not just
**is this state implemented**. A state that is defined but unreachable in
practice is theater; a state whose trigger depends on a backend change must
either include that change in the slice or be explicitly deferred. This rule
emerged from a real catch at E2E in a run where a "no-results" message was
shipped but never reached because the search endpoint always returned top-k.

**The Sponsor/CEO gate is cheap but NOT a rubber-stamp - produce a 3-line artifact** (or
it gets skipped): **(1) Goal** - one sentence; **(2) Success metrics** - >=1 *measurable*
check, before->after where it applies; **(3) Scope guardrails** - what's in, and explicitly
what's *out / untouched*. Contested direction -> `/punakawan:panel`. (field-notes: skipped on a backend slice; on a UI slice the artifact anchored sign-off.)

### Phase 1 - Design and Architecture
| Role | Type | Maps to | Output |
| --- | --- | --- | --- |
| SA / Architect | STAGE | `feature-dev:code-architect` *(subagent, not a skill)* + `writing-plans` (skill) | design + implementation plan |
| Security (shift-left) | GATE | `/punakawan:panel --preset safeguard` (threat / obligation / operability) | threat model + **numbered MUSTs (T1, T2…)** folded into the plan |
| Plan sign-off | GATE | human approves the plan | go / no-go |

### Phase 2 - Build
| Role | Type | Maps to | Output |
| --- | --- | --- | --- |
| DEV | STAGE | `subagent-driven-development` + `test-driven-development` | code + unit tests (whitebox) |

### Phase 3 - QA / Test (the test pyramid)
| Layer | Type | Maps to | Output |
| --- | --- | --- | --- |
| Whitebox (unit + integration) | STAGE | `test-driven-development` + coverage | green internal tests |
| Blackbox (functional + API / contract) | STAGE | functional / contract tests against the interface (no internals) | green behavior tests |
| E2E / UI | STAGE | **Playwright MCP** (`browser_navigate` / `browser_click` / `browser_snapshot`) | smoke + user-flow regression |
| Static review | **STAGE** | `requesting-code-review` (skill) or the `/code-review` built-in (multi-dimensional for multi-file / security slices) | review findings addressed |

**Code review is a STAGE, not a cheap gate, for any multi-file or security-touching slice.**
A single inline glance under-reads a 15+-file or credential-touching change. Run it as a
*multi-dimensional* review - a reviewer per dimension (DI / seams, correctness, security
surface, tests + spec-conformance) - then **verify the findings** (below). The `GATE != subagent`
rule below bans inflating headcount by *persona / job-title*, **not** the review *dimensions* a
STAGE legitimately fans out. (Lite lane: one cheap single-pass review. field-notes: a 7-dimension
review caught two P1 seam defects a green suite missed.)

**Platform-tooling pre-check for E2E.** Browser-wrapping shells (Tauri, Electron) need a
native-shell WebDriver driver (`tauri-driver`, `electron-chromedriver`); where it is
platform-unsupported, write specs with a conditional preflight skip, document the gap in
the Phase 3 report, and *do not fake-green the role*. (field-notes: E2E platform-tooling.)

**When the slice has no UI / no HTTP surface (off-web), E2E and Blackbox retarget - they
don't vanish.** *(Provisional, n=1 - comic-web Slice B; confirm on a 2nd off-web slice.
Mirrors the non-web-DAST note in Phase 4.)* For a CLI / job / library / internal-service
slice: **E2E** = drive the real entrypoint end-to-end against real (test) infra and assert
the *observable outcome* - exit code, DB / queue state, emitted events, logs - not a browser
flow; **Blackbox** = test the *contract* of that interface (CLI args / flags / exit-codes /
stdout, or the public API/method signature + return), internals kept black-box. Whitebox,
static-review, and SAST are unchanged.

### Phase 4 - Security verification (the role that was missing)
| Activity | Type | Maps to | Output |
| --- | --- | --- | --- |
| SAST / secure code review | STAGE | `/security-review` (CLI built-in, not a plugin skill) on the diff (see local-repo fallback below) | injection / authz / secret / crypto findings |
| Concurrency probe | STAGE | included in SAST when the slice adds any **multi-write code path** (a new endpoint or call site that mutates shared state, esp. through a sync route hitting FastAPI's threadpool) | races, lost updates, inconsistent reads |
| Seam-zone probe | STAGE | included in SAST whenever the slice (a) **deviates from the spec's implementation choice** (e.g. regex instead of the spec'd parser library), or (b) introduces a new boundary between layers (cache + policy, parser + orchestration, etc.). Build-time review cannot see these seams because they are *inter-stage* by construction. | misuse-of-deviation findings, content-spoofing, parser-ambiguity edge cases |
| DAST / pentest | STAGE* | **Playwright MCP** for browser-level probes (XSS, IDOR, authz bypass) **+ `curl` / raw HTTP for protocol-level probes** + OWASP checklist | dynamic findings |

`*` **Honest limit.** Deep pentest (Burp / ZAP, fuzzing) is manual / tooling outside
Claude. The recipe *points* to it; it does not pretend to automate it. What is
automatable here: `security-review` (static) plus light Playwright probing (dynamic).

**Local-repo fallback for SAST.** When `security-review` breaks on a fresh local repo
without a remote (it hard-diffs against `origin/HEAD`), dispatch a SAST subagent over the
branch diff (`git diff main..feature-branch`) - same checklist, manual driver. (field-notes:
local-repo fallback.)

**Why `curl` alongside Playwright for DAST.** Chromium blocks HTTP/1.1 request
streaming and rejects some header manipulation; SSRF probes via chunked uploads,
explicit `Content-Length` bypass attempts, or `Transfer-Encoding: chunked` need a
non-browser client. Pair Playwright (DOM/UI surface) with `curl` (protocol surface).

**DAST when there is no browser / HTTP surface** - *provisional (n=1); full guidance + why
it is provisional are in the **Watch list** at the end of this file.* Gist: when a slice
exposes no web surface (backend probes, CLI, a library, an encrypted column), retarget DAST
to the runtime that exists and **prove the security claim with a real command, not by
reasoning**. The honest-limit note above still applies.

**Bundled-sidecar staleness pre-check for DAST.** If the slice ships a bundled separate
process (Tauri sidecar, Electron subprocess, serverless layer), the dev runner may not
rebuild it on source change - DAST against a stale bundle gives *false-positive* P0/P1.
Before any DAST conclusion, verify the bundle matches the source tree (grep a recent
identifier, compare mtimes, or rebuild). (field-notes: bundled-sidecar staleness.)

### Phase 5 - Docs and Release
| Role | Type | Maps to | Output |
| --- | --- | --- | --- |
| TW | STAGE *(optional, offered)* | **offer** "produce the release/stakeholder doc now?" - usually deferred until *all* modules' UAT is done; format asked per project; `context7` for library docs | documentation in the project's format (when accepted) |
| Verification | GATE | `verification-before-completion` must be green | passing evidence |
| UAT / End User | GATE | acceptance vs original intent | accept / loop back |
| Commit / Release | GATE | **hard human gate + worktree/branch sandbox**; `finishing-a-development-branch` | confirmed release |

**TW is opt-in and usually end-of-arc - offer it, don't auto-run it.** Documentation is
typically written **once, after *all* modules' UAT** (not per slice), so **ask** rather than
auto-produce: *"Produce the release/stakeholder doc now? It's usually done after the whole
feature's UAT."* (Matches dalang's offer-first discipline.) When accepted,
the deliverable is the stakeholder/release doc (**distinct** from the markdown run-artifacts,
which stay markdown), in the **project's delivery format - ask which** (markdown / .docx / other).
For a **.docx (Word)** shop the bundled **`scripts/md2docx.py`** makes a *valid* `.docx` from a
Markdown subset with **zero deps** (python stdlib; verified `Microsoft Word 2007+` on py3.9) -
or `pandoc` / `python-docx` if the project has them. The format is a per-project choice, not a
universal rule; dalang just ships the zero-dep .docx capability for when it's wanted.

**UAT vs intent - check what was *wanted*, not just the spec (concrete checklist).** This is
the only gate guarding *wanted != built*; green tests + spec-conformance are necessary, not
sufficient. Run it explicitly: **(a)** does the result match the **Phase-0 goal + the user's
own words** (not just the plan)? **(b)** are the **defined UI/UX states actually reachable**
(empty / error / loading - see the state-trigger discipline)? **(c)** for a UI change,
**observe the real artifact** - screenshot / E2E the rendered result against the design
intent and confirm no console errors (do not infer from tests alone); **(d)** an *intent*
mismatch loops back to **PM / Design** (distinct from a defect, which loops to DEV). (field-notes: a UI UAT caught a system-adaptive default, an absent empty-state, and a dark-mode contrast issue.)

## Outputs the runner gets (the honest professional artifacts)

dalang executes in **one session** (minutes), so it produces engineering artifacts
and traceability - **not** a calendar, sprint cadence, or velocity (theater for an
in-session agent). Produce five small live tables - **phase log, run-board, defect
register, loop-back log, traceability** - treating the phase log as first-class (a
reader can replay the run from it). **Column formats: `references/artifacts.md`.**

**Where the artifacts live - offer it ONCE at run start (cross-repo safety).** First **detect any
existing convention** - if the repo already uses `docs/superpowers/runs/` or `docs/dalang/`,
continue it (don't fragment). Then **offer the destination**: *"Run history in `docs/dalang/<slice>/`
(default), this repo's existing `docs/...`, or another path - e.g. a separate / central docs repo
for cross-repo work?"* Confirm once with a sensible default (don't re-ask per artifact); record the
chosen path in the phase log so the run stays replayable.

Behavioral rules that ride on the artifacts:
- **Run-board:** `awaiting-gate` is the terminal state of a *healthy* run parked at a
  human gate - keep it distinct from `blocked` (stuck on an unresolved defect/dependency).
- **Defect register:** one **P0/P1/P2/P3** scale (not a parallel Crit/High/Med/Low) so the
  loop-back rule stays crisp. **Any open P0/P1 bounces back to DEV before the commit gate.**
  Record clean checks too (severity `-`, status `closed - no finding`) - negative results
  aren't lost.

### Verify findings before they gate

A finding from review / SAST / DAST is a **claim, not yet a fact**. Before its severity drives
a loop-back or blocks the commit gate, **adversarially verify each finding**: an *independent
skeptic, refute-by-default*, reads the actual code and rules it **confirmed / refuted /
severity-adjusted**, grounded in file:line. This kills false positives and stops a
confident-but-wrong severity from gating the run; record each verdict + reasoning in the
register. (Skip on the lite lane. field-notes: a Phase-4 pass correctly refuted four "injection"
findings on operator-set CLI values - no boundary crossed.)

### Loop-back log (what bounced, and why)
Loop-backs are what make this iterative rather than waterfall, but a status snapshot
hides them - record each bounce as a first-class event (format in `references/artifacts.md`).

**When you loop back to fix a finding** (it is Phase-2 work, held to Phase-2 discipline):
- **Red-first + test-honesty.** Write the regression test first and *watch it fail*; confirm it
  goes red when the defect is present (a test that passes pre-fix proves nothing).
- **Delta-SAST the fix, not just the suite.** A fix is new code on (often shared) surface; before
  re-clearing the gate re-run SAST over the **fix diff** (`git diff` of the loop-back commits),
  not just the full re-run. A green suite is necessary, not sufficient. (field-notes: a guard test
  passed for the wrong reason; delta-SAST caught a P2 test-honesty defect.)

### Traceability
Map each PM acceptance criterion to the test(s) that cover it and the result; a failing
row points to its defect ID. Format in `references/artifacts.md`.

## Discipline / honesty rules (carried from the Punakawan verdict)

- **GATE != subagent.** Gate rows are cheap human/inline/advisory checks. Do not
  spawn a subagent per *job title / persona* - collapse roles to the mapped skill calls.
  This bans **headcount theater**, not the multiple review *dimensions* a STAGE (code
  review, SAST) legitimately fans out, nor the *independent skeptic* that verifies a
  finding - those are real work, not personas.
- **Many hats, one head.** 8+ roles are one model conditioned on different personas,
  not independent reviewers. The value is the *checklist and the gates*, not headcount.
- **Security is shift-left, shift-right, and *traced*.** Once at design (threat model ->
  numbered MUSTs T1…), once at verification (SAST + DAST). **Phase-4 SAST must close every
  MUST-ID** - each surfaces as a defect-register row (verified-clean or a finding), so a
  design-time security requirement can't silently go unverified (the security analog of
  requirement->test traceability). Never only at the end.
- **Execution writes to a sandbox.** All work happens on a worktree/branch; a hard
  human gate precedes any commit, push, or deploy. A bad run is `git reset`-able.
- **Scale to the change.** Use the lite lane for small fixes; reserve the full
  pipeline (E2E + pentest) for changes that warrant it.

## Watch list (provisional - n=1, awaiting a 2nd confirmation)

Observations from a *single* run that did **not** meet the load-bearing-catch bar (no
P0/P1 would have slipped without them) - so per the "two tracks" rule near the top they
are recorded here, **not** folded as settled discipline, until a 2nd independent run
confirms them. Treat them as useful defaults, not law; promote to the body on confirmation.

- **DAST when there is no browser / HTTP surface** *(comic-web Slice B).* Many slices
  (backend probes, CLI commands, a library, a new encrypted column) expose no web surface,
  so Playwright/curl have nothing to drive. DAST does not vanish - it *retargets* to the
  runtime that exists: invoke the CLI / probe under real conditions (missing / malformed /
  valid input, unreachable upstream) and **prove the security claim with a real command,
  not by reasoning** - e.g. write a row through the app then read the *raw* DB column to
  confirm ciphertext-at-rest with plaintext absent; grep logs/stdout for leaked secrets;
  confirm the path fails closed. *Provisional because in its origin run the agent improvised
  this fine without the rule - a guidance gap, not a missed catch.*

- **Off-web E2E / Blackbox retarget** *(comic-web Slice B; n=1).* The Phase-3 off-web retarget
  note is **provisional** - the contrasting slice was on-web, so it did not confirm the off-web
  case. Promote once a 2nd off-web slice (CLI / job / library) confirms it.

*(Promoted to the body this run: Sponsor-gate output template, UAT-vs-intent checklist, TW
deliverable+format, code-review-as-STAGE - each on a 2nd contrasting confirmation. Threat-model
-> SAST traceability folded too, but as a **structural completion** of the existing
requirement->test traceability discipline, n=1 - watch for a real MUST-slip to confirm its weight.)*

- **Pre-Build foundation / health precheck** *(comic-web landing QA run; n=1).* Before Phase 3
  QA / E2E, verify the app is actually runnable + has the data the slice needs: app boots over
  *real HTTP* (not just the CLI kernel - stale opcache/config-cache 500'd every route incl
  `/up` until `optimize:clear` + container restart), the build toolchain is reachable (node was
  host-only, not in the app container), the web server is healthy (nginx was unhealthy -> use
  `php artisan serve`), and there's enough seed data to exercise content states (empty dev DB +
  SQLite test-DB missing content tables both blocked real verification). dalang already has
  E2E platform-tooling + bundled-sidecar prechecks; this is the broader "does it even run, with
  data?" gate. Confirm on a 2nd run before folding.

- **A Phase-1 threat model can RE-SCOPE the slice, not only add MUSTs** *(Enkari Plan C; n=1).*
  The safeguard panel may conclude the slice's **core approach** is unsafe and bounce all the way
  back to re-defining what the slice *is* - not just appending T-numbered requirements. In Plan C
  the panel found the planned containment (an agent with `acceptEdits` "isolated" on a git branch)
  was not containment at all (a branch isolates the commit graph, not Bash/filesystem/network), and
  the slice was re-scoped from "run the agent that edits" to a **read-only `plan`-mode MVP**, with
  the dangerous write capability deferred to a *follow-up slice that re-runs the safeguard panel
  first*. Treat such a re-scope as the threat-model gate **working**, record it as a Phase-1->
  Architecture loop-back (human re-scope), and split rather than push the dangerous capability
  through. *Already implied by "a threat model opens a new risk -> back to Architecture"; the new
  nuance is that the right resolution is often a smaller, safer slice + a deferred one, not a pile
  of mitigations bolted onto the original.* Promote on a 2nd run where a panel re-scopes a slice.

- **Spike the real external contract before TDD-ing the parser** *(Enkari Plan C; n=1).* When a
  slice integrates an external tool's stream/output (a CLI, an API), make the **first Build task a
  SPIKE** that captures the *real* schema from the live tool and then TDD against captured fixtures -
  never against an assumed shape. The Plan C spike of `claude --output-format stream-json` found
  load-bearing surprises an assumed schema would have gotten wrong: huge hook-noise lines (validating
  the parser's byte-cap), and the requested `--model` alias differing from the model actually billed
  in the events (so the UI must read the model from the events, not assume the alias). A self-graded
  parser built on guessed shapes is *necessary, not sufficient* - the captured fixture is the
  validation. Confirm on a 2nd external-integration slice before folding.
