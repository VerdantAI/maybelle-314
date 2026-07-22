# Agent Control Surface — Research Findings

## Intro

This document records research for the `investigate-agent-control-surface` spike: how to
design the control surface that coding agents (Codex, Claude) and humans use to drive,
build, and validate Maybelle 314 on the source/development system, while keeping the
performance Pi safe. It is docs-only; no code, CLI, MCP server, or skill is implemented
here.

The subject is a control-surface *shape* — most likely a conventional CLI with
machine-readable output, optionally fronted later by an MCP adapter and/or repo-local
agent skills. Findings below are grounded in current standards and official documentation,
with a permissive-licensing lens on any recommended tooling.

**Method:** web research (WebSearch + WebFetch) across four clusters — agent-friendly CLI
design, JSON Schema / machine-readable I/O, MCP server patterns, and Claude Code
skills / `AGENTS.md` conventions.

**Retrieval date:** 2026-07-22.

**Evidence tiers used below:**
- **Standard** — formal or de-facto specification (POSIX, BSD `sysexits.h`, `NO_COLOR`, JSON Schema, MCP spec, RFC).
- **Convention** — documented in respected cross-industry guides (clig.dev, 12-Factor CLI, GitHub CLI, official Anthropic/MCP docs).
- **Opinion/emerging** — 2025–2026 practitioner writing on "CLIs/tools for AI agents"; useful but not standardized.

---

## Findings by Requirement

### Requirement: Agent workflow inventory

*Classify Maybelle workflows and mark each source-system-only, Pi-safe, or hardware-affecting.*
This requirement is answered from the project's own `proposal.md`, `design.md`, and
`README.md` (not web research); it is recorded here for completeness of the deliverable.

| Workflow | Primary inputs | Classification | Risk tier |
|---|---|---|---|
| DAW export inspection (Bitwig Type-1 SMF notes+velocity; Ardour TBD) | MIDI files, DAW exports | Source-system-only | Read-only |
| Song-bundle generation (MIDI + metadata manifest) | DAW exports, schemas | Source-system-only | Reversible file-write |
| ES-9 profile validation | Recorded ES-9 profile, schema | Hardware-adjacent (validated against recorded/mock fixture) | Read-only |
| Pi port-topology report | Live Pi USB/port enumeration | Pi-safe (diagnostic) | Read-only |
| Configuration review / validation | Runtime config files, schema | Source-system-only | Read-only |
| Fixture generation (deterministic test music) | Generators, schemas | Source-system-only | Reversible file-write |
| Test execution | Repo test suite, fixtures | Source-system-only | Read-only |
| LFO/modulation waveform + Assimil8or preset prep (offline tooling) | AKWF-style shapes, DAW bounces | Source-system-only | Reversible file-write |
| ES-9 config upload | ES-9 device | Hardware-affecting | Hardware write — gate or exclude |
| Arm/trigger live playback on Pi | Live rack | Hardware-affecting | Live performance control — exclude |

This maps directly onto the design's two-host model: the **source/development machine**
gets the full agent surface; the **performance Pi** (potentially wired to a live rack)
exposes at most read-only diagnostics.

### Requirement: Structured CLI assessment

*Evaluate a structured CLI as the baseline agent surface and define its conventions.*

The research strongly supports making a conventional CLI the primary control surface. The
most authoritative single reference is the **Command Line Interface Guidelines**
([clig.dev](https://clig.dev/), [repo](https://github.com/cli-guidelines/cli-guidelines)),
supplemented by [12-Factor CLI Apps](https://jdxcode.medium.com/12-factor-cli-apps-dd3c227a0e46)
and the [GitHub CLI manual](https://cli.github.com/manual/).

**Command shape (Convention).**
- **Noun-verb subcommands** are the dominant pattern; `noun verb` is more common than `verb noun`, e.g. `gh repo create`, `gh pr list` ([clig.dev](https://clig.dev/), [GitHub CLI](https://cli.github.com/manual/)). For Maybelle this suggests `maybelle bundle validate`, `maybelle profile check`, `maybelle topology report`, `maybelle fixture generate`, `maybelle config validate`.
- Be **consistent** across subcommands (same flag names, output shape) and avoid ambiguous/similar names such as `update` vs `upgrade` ([clig.dev](https://clig.dev/)).
- **Help text**: support `-h`/`--help` on every command and subcommand, show help when a required-argument command is run bare, and **lead with examples** ([clig.dev](https://clig.dev/)).

**`--json` and machine-readable output (Standard + Convention).**
- The core stream contract is a Unix **Standard**: primary/machine-readable output to **stdout**; logs, progress, and errors to **stderr** — so warnings survive `stdout` redirection and piping works ([clig.dev](https://clig.dev/), [12-Factor CLI](https://jdxcode.medium.com/12-factor-cli-apps-dd3c227a0e46)).
- Provide a **`--json`** flag for structured output; JSON "affords more structure … for complex data structures" ([clig.dev](https://clig.dev/)). Offer **`--plain`** tabular text when human formatting would break `grep`/`awk` parsing ([clig.dev](https://clig.dev/)).
- Switch human-vs-machine behavior on **TTY detection**: when stdout is not an interactive terminal, drop colors/animations/progress bars ([clig.dev](https://clig.dev/), [12-Factor CLI](https://jdxcode.medium.com/12-factor-cli-apps-dd3c227a0e46)).
- Honor **`NO_COLOR`** (Standard): if the env var is present and non-empty, suppress ANSI color; also on `--no-color`, non-TTY, or `TERM=dumb` ([no-color.org](https://no-color.org/), [clig.dev](https://clig.dev/)).
- Support **`-`** for stdin/stdout so commands compose in pipelines without temp files ([clig.dev](https://clig.dev/)).

**Deterministic exit codes (Standard, with a caveat).**
- Baseline: **0 = success, non-zero = failure**, mapping the most important failure modes to distinct codes ([clig.dev](https://clig.dev/), [GNU Bash — Exit Status](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)).
- **BSD `sysexits.h`** (values 64–78: `EX_USAGE`=64, `EX_DATAERR`=65, `EX_NOINPUT`=66, `EX_UNAVAILABLE`=69, `EX_SOFTWARE`=70, `EX_TEMPFAIL`=75, `EX_CONFIG`=78, etc.) is the closest thing to a standard code-range convention, starting at 64 to avoid clashes ([man7 sysexits.h](https://www.man7.org/linux//man-pages/man3/sysexits.h.3head.html), [FreeBSD sysexits(3)](https://man.freebsd.org/cgi/man.cgi?sektion=3&query=sysexits)). **Caveat:** the interface is officially **deprecated** ("Its use is discouraged") ([Ubuntu sysexits.h](https://manpages.ubuntu.com/manpages/noble/man3/sysexits.h.3head.html)) — treat it as useful vocabulary, not a mandate.
- **Reserved shell codes to avoid** (Standard): `126` (not executable), `127` (command not found), `128+N` (killed by signal N, e.g. `130` = SIGINT), `255` (out of range) ([TLDP Exit Codes](https://tldp.org/LDP/abs/html/exitcodes.html), [GNU Bash](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)).
- Stable exit codes are what let scripts and agents branch on success/failure and decide retry-vs-don't-retry deterministically ([clig.dev](https://clig.dev/)).

**Commands runnable without hardware.** All read-only/inspection/validation and reversible
file-generation workflows above (bundle inspect/generate, config validate, fixture generate,
test execution, ES-9 profile validation *against a recorded profile*) run on the
source/development machine with **no rack attached**, using recorded or mocked fixtures.
Only ES-9 config upload and live playback need real hardware, and those are gated/excluded
(see Safety model). This satisfies the requirement's "identify which commands agents can run
safely without direct hardware access."

**Why this specifically helps agents (Emerging/opinion).** Each CLI invocation is an atomic
operation with an exit code, stdout, and stderr — "exactly the deterministic loop agents
need for self-correction"
([dev.to](https://dev.to/thedailyagent/building-production-grade-tools-for-ai-agents-what-works-after-100-deployments-20om)).
Structured output makes results parseable
([Designing CLI Tools for AI Agents](https://archit15singh.github.io/posts/2026-02-28-designing-cli-tools-for-ai-agents/)),
and the **never-require-an-interactive-prompt** rule (below) is doubly critical because an
agent cannot answer an unexpected prompt and will hang ([12-Factor CLI](https://jdxcode.medium.com/12-factor-cli-apps-dd3c227a0e46)).

### Requirement: MCP feasibility assessment

*Map candidate MCP resources/tools/prompts to CLI/core operations; record what is useful only after schemas/CLI stabilize.*

**MCP primitives (Standard).** MCP servers expose three primitives with a clear control model
([Understanding MCP servers](https://modelcontextprotocol.io/docs/learn/server-concepts),
[Architecture overview](https://modelcontextprotocol.io/docs/learn/architecture)):
- **Tools** — executable functions the model can call and decides when to use (model-controlled; may require user consent).
- **Resources** — passive, read-only data sources for context, each with a URI + MIME type (application-controlled).
- **Prompts** — reusable instruction templates / workflows, explicitly user-invoked (typically surfaced as slash commands).

**Candidate mapping for Maybelle** (adopt only after the CLI/schemas stabilize):

| MCP primitive | Maybelle candidates |
|---|---|
| Resources | JSON Schemas, song-bundle manifests, ES-9 profiles, topology reports, validation results, logs, fixtures |
| Tools | Thin wrappers over stable core ops: `validate_bundle`, `check_es9_profile`, `report_topology`, `validate_config`, `generate_fixture` (read-only/reversible only) |
| Prompts | "Validate this song bundle", "Prepare an ES-9 patch report", "Diagnose Pi port topology" |

**MCP as an adapter, not the engine (Convention/recommendation).** The dominant best
practice is to keep the MCP layer thin and put behavior in a **core library the CLI already
exercises**, so behavior stays testable without the protocol and a second implementation
doesn't drift ([philschmid MCP best practices](https://www.philschmid.de/mcp-best-practices),
[MCP vs CLI+Skills trade-offs](https://medium.com/@akshaychame2/mcp-vs-cli-vs-cli-skills-trade-offs-use-cases-and-best-practices-49b9cfd7a556)).
Design MCP tools around **agent outcomes**, ~5–15 per server, namespaced, with rich
descriptions and paginated returns — not a 1:1 endpoint dump
([Anthropic: Writing tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents)).

**Transports (Standard).** MCP defines two: **stdio** (local subprocess; "clients SHOULD
support stdio whenever possible") and **Streamable HTTP** (remote, multi-client, supports
HTTP auth) ([Transports spec](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)).
For a local dev tool, **stdio** is the right default (no network surface). If Streamable HTTP
is ever used, the spec **mandates** validating the `Origin` header, binding to `127.0.0.1`
when local, and authenticating all connections ([Transports spec](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)).

**When MCP earns its keep vs. cost (Convention).** MCP is worth adopting for cross-host/org
sharing, stateful sessions, or governance/audit needs; its costs are a separate codebase to
maintain and significant token overhead (full tool schemas injected per session — stacking
several servers can exceed 150k tokens) ([Firecrawl MCP vs CLI](https://www.firecrawl.dev/blog/mcp-vs-cli),
[MindStudio token costs](https://www.mindstudio.ai/blog/mcp-vs-cli-ai-agents-token-costs-when-to-use)).
Because the adapter is thin, starting CLI-only and adding MCP later is a reversible decision.
This directly answers the requirement's "record which MCP capabilities are useful only after
schemas and CLI commands are stable": **all of them** — MCP tools should wrap stabilized core
ops, and resources should serve stabilized schema/manifest formats.

**Maturity & licensing.** MCP is governed under the Linux Foundation with two stabilized
transports, a dated-spec cadence, official SDKs, and an Inspector tool
([MCP org](https://github.com/modelcontextprotocol)). Licensing is **permissive**: spec is
MIT→Apache-2.0, Python SDK MIT, TypeScript SDK MIT→Apache-2.0
([python-sdk LICENSE](https://github.com/modelcontextprotocol/python-sdk/blob/main/LICENSE),
[typescript-sdk LICENSE](https://github.com/modelcontextprotocol/typescript-sdk/blob/main/LICENSE)) —
compatible with Maybelle's MIT-compatible policy.

### Requirement: Safety boundary model

*Classify each proposed agent operation and specify required safeguards.*

The design's five-tier classification aligns with documented CLI safety practice and MCP
tool-annotation semantics.

| Tier | Examples | Required safeguards |
|---|---|---|
| Read-only | inspect bundle, validate config, report topology, check ES-9 profile | Broadly available; no confirmation |
| Reversible file-write | generate bundle, generate fixture, write LFO/preset | **Default dry-run + diff**; write only on explicit flag |
| Hardware-adjacent validation | ES-9 profile checks against recorded/mock fixture | Read-only; run against fixtures, never touch live device |
| Hardware write | upload ES-9 config | **Explicit human confirmation**, source-system only, never autonomous |
| Live performance control | arm/trigger playback on Pi | **Excluded** from agent tooling entirely |

**Safeguard mechanics (Convention).**
- **`--dry-run`**: "describe the changes that would occur" without running them (e.g. `git add`, `rsync`) ([clig.dev](https://clig.dev/)).
- **Tiered confirmation**: mild changes may skip confirmation; moderate (deletes, remote/hardware changes) should prompt and offer dry-run; severe should require typing a non-trivial confirmation, with `--confirm="<name>"` for scripted use ([clig.dev](https://clig.dev/)).
- **Never *require* an interactive prompt** — always allow override via `-f`/`--force` or `--yes` so automation (and agents) don't hang ([12-Factor CLI](https://jdxcode.medium.com/12-factor-cli-apps-dd3c227a0e46), [clig.dev](https://clig.dev/)). The safe posture for hardware writes is therefore *explicit opt-in flag + confirmation string*, not an unskippable prompt.

**MCP tool annotations as a risk vocabulary (Standard, with caveat).** If MCP is adopted,
every tool should carry the four hints — `readOnlyHint`, `destructiveHint`,
`idempotentHint`, `openWorldHint` — so hosts can auto-approve reads and gate writes. Defaults
are pessimistic (an unannotated tool is treated destructive + open-world + non-idempotent)
([MCP: Tool Annotations](https://blog.modelcontextprotocol.io/posts/2026-03-16-tool-annotations/)).
**Caveat (Standard):** clients "MUST treat [annotations] as untrusted unless they come from a
trusted server" — annotations are hints for UX, not security enforcement; real safety comes
from confirmation, least privilege, and sandboxing
([MCP: Tool Annotations](https://blog.modelcontextprotocol.io/posts/2026-03-16-tool-annotations/),
[OWASP MCP Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html)).

**Two-host boundary & audit.** Keep hardware-write and live-control operations
source-system-only and out of any Pi-side surface; the Pi exposes at most read-only
diagnostics. Log agent-triggered operations (activity logs are a documented MCP consent
mechanism ([server-concepts](https://modelcontextprotocol.io/docs/learn/server-concepts))),
and prefer **deterministic, diffable** output so every agent action is human-inspectable —
which the JSON-Schema/canonical-output findings below make concrete.

### Requirement: Repository guidance strategy

*Identify which guidance files/skills should exist, what each contains, how to avoid bloat, and how to validate.*

**File set (Convention, official docs).**
- **`AGENTS.md`** is the cross-tool open standard ("a README for agents") stewarded by the Agentic AI Foundation under the Linux Foundation, supported by Codex, Claude, Gemini CLI, Copilot, Cursor, and 20+ others; plain Markdown, no required fields, repo root, closest file wins in monorepos ([agents.md](https://agents.md)).
- **`CLAUDE.md`** — Claude Code reads `CLAUDE.md`, **not** `AGENTS.md`. The documented interop pattern is a `CLAUDE.md` that imports `@AGENTS.md` (or a symlink) so both tools share one source of truth ([Claude Code memory](https://code.claude.com/docs/en/memory)). **Recommendation for Maybelle:** author one `AGENTS.md` and a thin `CLAUDE.md` that imports it — single source, cross-agent coverage.
- **Slash commands / skills** for repeatable workflows (see next requirement).

**Content (official Include/Exclude guidance).** Include: bash commands agents can't guess
(validation/test runners), code-style rules that differ from defaults, testing instructions,
repo/PR etiquette, architecture map, artifact locations (where schemas/bundles/profiles/
fixtures live), and the **unsafe-operations list** (hardware writes, live control). Exclude:
anything derivable from reading code, standard language conventions, detailed API docs (link
instead), frequently-changing info, file-by-file descriptions
([Claude Code best practices](https://code.claude.com/docs/en/best-practices)).

**Avoiding context bloat (official).** Target **under ~200 lines** for `CLAUDE.md`; "bloated
CLAUDE.md files cause Claude to ignore your actual instructions." For each line ask "would
removing this cause a mistake?" — if not, cut it. Move multi-step procedures into skills
(load on demand) or path-scoped rules; note `@path` imports organize but do **not** reduce
loaded context ([Claude Code memory](https://code.claude.com/docs/en/memory),
[best practices](https://code.claude.com/docs/en/best-practices)).

**Validation (official).** Treat guidance "like code": review when things go wrong, prune
regularly, and test by observing whether agent behavior actually shifts
([best practices](https://code.claude.com/docs/en/best-practices)). Give agents a **way to
verify** their work (test suite / build exit code / linter returning pass/fail) so they close
their own loop ([best practices](https://code.claude.com/docs/en/best-practices)). For
requirements that must hold every time, use a **hook** (deterministic), not advisory prose.
This satisfies "how it should be validated against real agent workflows."

### Requirement: Published skill assessment

*Compare repo-local skills, published reusable skills, MCP prompts, and plain docs; identify appropriate workflows, dependencies, exclusions, and publishing requirements.*

**What skills are (official).** A Claude Code **Agent Skill** is a directory with a
`SKILL.md` (YAML frontmatter — `name` ≤64 chars lowercase/hyphen, `description` ≤1024 chars
stating *what* and *when* — plus Markdown body). Skills use **progressive disclosure**:
name+description (~100 tokens) always loaded; body (<5k tokens) loaded on trigger; bundled
resources read on demand ([skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview),
[Claude Code skills](https://code.claude.com/docs/en/skills)). Notably, **custom slash
commands have been merged into skills** — a `.claude/commands/deploy.md` and a
`.claude/skills/deploy/SKILL.md` both create `/deploy`; skills are the recommended format and
add supporting files, invocation control, and autonomous invocation
([Claude Code skills](https://code.claude.com/docs/en/skills)).

**Comparison of options for Maybelle:**

| Option | Fit for Maybelle | Notes |
|---|---|---|
| **Repo-local skills / commands** (`.claude/skills/`, `.claude/commands/`) | **Best first step** | Committed to VCS, versioned with the code, invoked via `/name`; can pre-approve exact commands via `allowed-tools`. Use `disable-model-invocation: true` for side-effecting workflows so only the human triggers them. |
| **MCP prompts** | Later, if MCP adopted | Same workflow templates, but only worth it once an MCP server exists. |
| **Plain docs / `AGENTS.md`** | Always | Baseline; cheapest; cross-agent. |
| **Published reusable skills** (plugins/marketplace) | **Defer** | Extra maintenance, versioning, and safety surface; do only after CLI/schemas/safety stabilize. |

**Appropriate skill workflows** (repo-local first): "validate this song bundle", "prepare an
ES-9 patch report", "generate test fixtures", "diagnose Pi topology". Each depends on the
corresponding **stable CLI command + JSON Schema**. Hardware-write and live-control workflows
must be **excluded** from any skill (or gated behind human confirmation), matching the safety
tiers.

**Publishing requirements if/when a skill is published (official).** Portability: skills
follow the [Agent Skills open standard](https://agentskills.io) but **do not sync across
surfaces** (claude.ai, API, Claude Code are managed separately). Distribution is via
**plugins + marketplaces** (`marketplace.json` in `.claude-plugin/`, hosted on any git host;
users add with `/plugin marketplace add owner/repo`), which provide discovery, version
tracking, and updates ([plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)).
Evaluate before publishing with the official `skill-creator` A/B eval loop (with-skill vs
without-skill, ≥3 scenarios, baseline comparison) ([Claude Code skills](https://code.claude.com/docs/en/skills),
[skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)).

**Safety for distributed skills (official, strongly worded).** "Use Skills only from trusted
sources"; a malicious skill "can direct Claude to invoke tools or execute code in ways that
don't match the Skill's stated purpose." Audit all bundled files; treat external-URL-fetching
skills as especially risky; "treat like installing software." In Claude Code, a project
skill's `allowed-tools` grants take effect only after the workspace-trust dialog — review
project skills before trusting a repo
([skills overview — security](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
For Maybelle, which touches real hardware, this reinforces: **do not publish a reusable skill
until CLI schemas and the safety model are stable, and exclude all hardware-affecting
operations from any published artifact.** This satisfies the requirement's versioning /
compatibility / installation / testing / maintenance points.

### Requirement: Agent-readiness recommendation

See the **RECOMMENDATION** section below, which names the first implementation path and lists
unresolved spikes before non-read-only agent operations are allowed.

---

## Cross-cutting: machine-readable I/O contract (supports CLI + MCP + skills)

These findings underpin every requirement above and make agent output deterministic and
human-inspectable.

- **JSON Schema 2020-12** is the current dialect; always set `"$schema": "https://json-schema.org/draft/2020-12/schema"`. Publish schemas for both command inputs and outputs so agents can self-validate against one source of truth ([json-schema.org spec](https://json-schema.org/specification), [getting started](https://json-schema.org/learn/getting-started-step-by-step)). Spec is **BSD-3-Clause / AFL-3.0** — permissive ([LICENSE](https://github.com/json-schema-org/json-schema-spec/blob/main/LICENSE)). MCP itself standardizes on JSON Schema 2020-12 ([MCP SEP-1613](https://modelcontextprotocol.io/seps/1613-establish-json-schema-2020-12-as-default-dialect-f)), so one schema set serves both CLI and MCP.
- **JSON Lines (JSONL/NDJSON)** for streams of records — one JSON value per `\n` line, UTF-8, no BOM, `.jsonl` — versus a single JSON document for one cohesive result ([jsonlines.org](https://jsonlines.org/)). Useful for validation results and topology enumerations.
- **Stable, machine-readable errors**: emit structured error objects with a **stable error code/identifier** (the RFC 9457 `type`/code pattern — HTTP-scoped but the reusable convention) so agents branch deterministically, distinct from the process exit code ([RFC 9457](https://www.rfc-editor.org/info/rfc9457/)).
- **Versioning**: apply **SemVer 2.0.0** to the output contract — MAJOR for breaking field/type changes — and embed a `schema_version` field; below `1.0.0` the contract is explicitly unstable ([semver.org](https://semver.org/), CC-BY-3.0). Back it with contract tests.
- **Determinism**: sort keys recursively (`json.dumps(..., sort_keys=True)`) for diffable/reproducible output; use **RFC 8785 JCS** only if byte-identical canonical output is ever needed ([RFC 8785](https://www.rfc-editor.org/info/rfc8785/)).
- **Validation tooling (permissive)**: `jsonschema` (Python, **MIT**) and `Ajv` (JS, **MIT**), both supporting 2020-12 ([python-jsonschema](https://github.com/python-jsonschema/jsonschema), [Ajv](https://github.com/ajv-validator/ajv)).

**Gap flagged:** there is **no CLI-specific standard** for machine-readable output or errors
analogous to jsonlines.org or RFC 9457. The established practice is to *compose* the
general-purpose standards above (JSON Schema + JSONL + RFC 9457-style error objects + SemVer +
canonical JSON).

---

## RECOMMENDATION

**Recommended first implementation path: CLI + repo guidance (`AGENTS.md`/`CLAUDE.md`) +
repo-local skills, over a shared core library. Defer MCP and any published skill.**

This matches the spike's design decisions and the weight of the research: every major coding
agent can run local commands and parse structured output, and a CLI stays useful to humans and
CI with no agent-specific runtime.

**Architecture.** Implement behavior in a **core library**; the **CLI** is a thin façade over
it, and any future **MCP server** is a second thin façade over the *same* core. This keeps
behavior testable without either surface and prevents drift
([philschmid](https://www.philschmid.de/mcp-best-practices),
[MCP vs CLI+Skills](https://medium.com/@akshaychame2/mcp-vs-cli-vs-cli-skills-trade-offs-use-cases-and-best-practices-49b9cfd7a556)).

**Top conventions to adopt:**
1. **Noun-verb subcommands** with consistent flags and examples-first help ([clig.dev](https://clig.dev/)).
2. **stdout = data, stderr = messages**; TTY-aware; honor `NO_COLOR`; support `-` for stdin/stdout (Standard) ([clig.dev](https://clig.dev/), [no-color.org](https://no-color.org/)).
3. **`--json`** structured output (plus `--plain`), governed by a **published JSON Schema 2020-12**; **JSONL** for record streams; **structured errors with stable codes**; **SemVer'd** output contract with an embedded `schema_version` ([json-schema.org](https://json-schema.org/specification), [jsonlines.org](https://jsonlines.org/), [semver.org](https://semver.org/)).
4. **Deterministic exit codes** — 0/non-zero, distinct codes per failure mode, avoid reserved shell codes; `sysexits.h` as vocabulary only (deprecated) ([clig.dev](https://clig.dev/), [man7](https://www.man7.org/linux//man-pages/man3/sysexits.h.3head.html)).
5. **Risk-tiered safety**: read-only broadly available; file-writes **dry-run + diff by default**; hardware writes **explicit opt-in flag + confirmation string, source-system only**; live control **excluded**. **Never require an unskippable prompt** ([clig.dev](https://clig.dev/), [12-Factor CLI](https://jdxcode.medium.com/12-factor-cli-apps-dd3c227a0e46)).
6. **Minimal guidance**: one `AGENTS.md` + a thin importing `CLAUDE.md` (<200 lines), listing commands, artifact locations, validation/test runners, and the unsafe-operations list; validated by observing agent behavior and pruning regularly ([agents.md](https://agents.md), [Claude Code memory](https://code.claude.com/docs/en/memory)).
7. **Repo-local skills/commands** for the common workflows, with `disable-model-invocation: true` on side-effecting ones ([Claude Code skills](https://code.claude.com/docs/en/skills)).

**How it serves both agents and humans.** Deterministic exit codes + structured stdout +
schema'd output give agents the atomic, parseable, self-correcting loop they need; the same
CLI, help text, and JSON are directly usable by humans and CI. Dry-run/diff defaults and
diffable canonical output make every agent action human-inspectable before it touches
anything real. The thin-façade core means MCP and published skills remain reversible,
incremental additions rather than rewrites.

**Tooling licensing:** all recommended standards/tooling are permissive — JSON Schema
(BSD-3/AFL-3), `jsonschema`/`Ajv` (MIT), MCP SDKs (MIT/Apache-2.0), SemVer (CC-BY-3.0),
`AGENTS.md`/MCP under the Linux Foundation — consistent with Maybelle's MIT-compatible policy.

---

## Open / needs follow-up

- **Base-platform/runtime dependency.** The concrete command set, package placement (main app
  vs separate dev-tool package vs MCP server), and the language of the core library depend on
  the `decide-base-platform` outcome (Python-centered runtime is the current bias, ES-9 I/O
  spike will drive it). *Blocked on `decide-base-platform`.*
- **Song-bundle & manifest schema.** JSON Schemas for bundle/manifest, ES-9 profile, topology
  report, and fixtures can't be finalized until the bundle format is decided
  (`decide-base-platform`) and DAW-interchange research lands.
- **ES-9 profile capture format.** What a "recorded ES-9 profile" fixture contains, and how
  agents validate against it without hardware, depends on the ES-9 I/O stack decision and the
  ES-9 profiles spike. *Blocked on ES-9 profile spike.*
- **Pi-side surface.** Whether the performance Pi exposes *any* agent-accessible (read-only)
  diagnostic surface, or remains fully source-system-only, is a deployment decision to confirm
  once the Pi image/runtime is chosen.
- **Hardware-write confirmation mechanics.** The exact confirmation model for ES-9 config
  upload (typed confirmation string, out-of-band approval, or exclusion) needs a dedicated
  safety spike before any non-read-only hardware operation is exposed.
- **Published-skill decision.** Deferred by design; revisit only after CLI commands, schemas,
  fixtures, and the safety model are stable, then run the `skill-creator` eval loop before
  publishing.
- **No CLI-specific I/O standard.** Because none exists, the composed convention (JSON Schema +
  JSONL + RFC 9457-style errors + SemVer) should be pinned in a short internal spec before
  first implementation so agents and humans share one contract.

### Unresolved spikes before non-read-only agent operations are allowed
1. Finalize bundle/manifest/profile/topology/fixture **JSON Schemas** (needs base-platform + bundle-format decisions).
2. Define the **hardware-write confirmation model** and audit-logging format.
3. Decide the **Pi-side surface** (read-only diagnostics vs none).
4. Pin the **machine-readable I/O contract** (schema dialect, error codes, versioning) as an internal spec.
5. Choose **package placement** and core-library language (follows base-platform).
