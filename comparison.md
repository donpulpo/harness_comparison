# comparison.md — Agent Harness Comparison

Scope note: 4 of 5 candidates are GitHub repos and are scored below.
`paulhoekstra.substack.com/p/agentic-engineering-the-guardrails` returned
HTTP 403 on every fetch attempt (paywalled/blocked) and is not a repo —
excluded from scoring. See `sources.md`.

Dimension list used: original 9 + Phase 1 additions 10–13 (see `concepts.md`).
Score legend: **3** = strong, explicit, verified in source. **2** = present
but partial/indirect. **1** = minimal/incidental. **N/A** = no evidence
found, or dimension doesn't apply to this repo's nature.

---

## Repo vitals

| Repo | Stars/Forks | Last release | Commits | Open issues | CI | Abandoned? |
|---|---|---|---|---|---|---|
| obra/superpowers | ~240k★/21.3k (unverified — see caveat) | v6.0.3, 2026-06-18 | — | 139 | None found (`.github/workflows` 404) | No — active, weekly/monthly releases |
| Chachamaru127/claude-code-harness | 2.9k★/277 | v4.16.4, 2026-06-28 | 1,445 | 1 | `validate-plugin.yml`, `opencode-compat.yml`, `codeql.yml`, `release.yml`, `scorecard.yml` (ubuntu-latest only, no OS/lang matrix) | No — near-daily releases |
| heliohq/ship | 82★/6 | No tagged releases; last commit ~2026-05-30 | 193 | 0 | None found (`.github/workflows` 404) | No — bursty active dev, but pre-1.0/no releases |
| OpenHands/OpenHands | 78.7k★/10k | cloud 1.40.0, 2026-06-26; core 1.8.0, 2026-06-10 | — | 120 | `py-tests.yml` (Python 3.12 only, single version), `fe-unit-tests.yml`, `fe-e2e-tests.yml`, `lint.yml`, release workflows | No — bi-weekly/monthly cadence |

**Caveat on superpowers' star count:** the research agent reported ~240k
stars corroborated by star-history.com and trendshift.io snippets, but this
is an order of magnitude larger than comparable skill-framework repos and
could not be cross-checked against the GitHub API (403 unauthenticated).
Treat this figure as **unverified** — flagged rather than asserted as fact.

---

## Scored dimensions

### 1. Context lifecycle
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 3 | SessionStart hook on `startup\|clear\|compact` re-injects bootstrap skill content after every reset; v6.0.0 moved to file-handoff instead of pasted text to cut token use | [hooks.json](https://raw.githubusercontent.com/obra/superpowers/main/hooks/hooks.json), [RELEASE-NOTES.md](https://raw.githubusercontent.com/obra/superpowers/main/RELEASE-NOTES.md) |
| claude-code-harness | 2 | Three "cognitive-load surfaces" (Plan Brief, Progress Tracker, Acceptance Demo) compress context; explicit `pre-compact`/`post-compact` hook subcommands save WIP state | [cognitive-load-surfaces.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/cognitive-load-surfaces.md), [go/DESIGN.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/go/DESIGN.md) |
| ship | 3 | SessionStart hook injects only a minimal routing hint, deliberately not full pipeline state, to avoid turning every session into a Ship session; phase-isolation hook restricts what each phase can read | [design/002](https://raw.githubusercontent.com/heliohq/ship/main/docs/design/002-session-context-injection.md), [phase-guardrail.sh](https://raw.githubusercontent.com/heliohq/ship/main/scripts/phase-guardrail.sh) |
| OpenHands | 3 | Pluggable Condenser architecture; append-only EventStore, condensation inserts summarizing markers rather than deleting; known overflow-loop bug w/ `--no-condense` escape hatch | [condenser docs](https://docs.openhands.dev/sdk/arch/condenser), [issue #8630](https://github.com/OpenHands/OpenHands/issues/8630) |

### 2. Tool execution layer
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 2 | No MCP/owned tool layer — maps onto each host's native tool-calling via 3 documented "integration shapes" (shell hook / in-process plugin / manifest context file) | [porting-to-a-new-harness.md](https://raw.githubusercontent.com/obra/superpowers/main/docs/porting-to-a-new-harness.md) |
| claude-code-harness | 2 | Native Claude Code hook/JSON-stdin model via single Go binary (~28 subcommands); MCP used only for optional `harness-mem` and Codex second-opinion review | [go/DESIGN.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/go/DESIGN.md), [ARCHITECTURE.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/ARCHITECTURE.md) |
| ship | 2 | Rides host's native tools + 3 Claude Code hooks (SessionStart/PreToolUse/Stop); 1 MCP server registered (`codex` as external verifier) | [hooks.json](https://raw.githubusercontent.com/heliohq/ship/main/hooks/hooks.json), [.mcp.json](https://raw.githubusercontent.com/heliohq/ship/main/.mcp.json) |
| OpenHands | 3 | Native MCP support (`mcp_config` on `AgentBase`) plus first-party Pydantic-typed tool package (`TerminalTool`, `FileEditorTool`, `TaskTrackerTool`); CLI `mcp add/list/enable/disable` | [agent/base.py](https://github.com/OpenHands/software-agent-sdk/blob/main/openhands-sdk/openhands/sdk/agent/base.py), [OpenHands-CLI README](https://github.com/OpenHands/OpenHands-CLI/blob/main/README.md) |

### 3. Planning & decomposition
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 3 | 7-phase methodology (brainstorm→worktree→plan→implement→TDD→review→finish); `writing-plans` mandates 2–5 min tasks, no placeholders; `subagent-driven-development` runs fresh-implementer-per-task + reviewer + progress ledger loop | [writing-plans/SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/writing-plans/SKILL.md), [subagent-driven-development/SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/subagent-driven-development/SKILL.md) |
| claude-code-harness | 3 | Explicit initializer→worker→reviewer→advisor roles; auto solo/parallel/"breezing" team mode by task count; `Plans.md` is 5-column task SSOT with DoD/Depends columns | [harness-work/SKILL.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/skills/harness-work/SKILL.md) |
| ship | 3 | `/ship:design` 6-phase pipeline with parallel independent investigator + capped adversarial spec resolution; `/ship:dev` decomposes into dependency-graphed parallel "waves" | [design/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/design/SKILL.md), [dev/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/dev/SKILL.md) |
| OpenHands | 3 | Dedicated `TaskTrackerTool` (Pydantic `TaskItem`, todo/in_progress/done, persistent `TASKS.json`), description explicitly steers use for multi-phase vs atomic work | [task_tracker/definition.py](https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/openhands-tools/openhands/tools/task_tracker/definition.py) |

### 4. Verification & guardrails
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 3 | `verification-before-completion` bans completion claims without fresh-run proof+output+exit-code; hard RED-GREEN-REFACTOR with verify-RED/verify-GREEN steps; 3-failed-attempts escalation in `systematic-debugging` | [verification-before-completion/SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/verification-before-completion/SKILL.md) |
| claude-code-harness | 3 | TDD red→green→refactor loop, retry capped at `max_iterations` (default 3); PreToolUse rules R01–R15 block secrets/protected paths/test tampering; dedicated tampering-detector script | [posttooluse-tampering-detector.sh](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/scripts/posttooluse-tampering-detector.sh) |
| ship | 3 | Multi-layered: peer review w/ capped fix rounds, independent E2E codification, defect-only severity-tagged review, exploratory QA forbidding "should work," plus a `Stop` hook that calls an external verifier before letting session exit | [stop-gate.sh](https://raw.githubusercontent.com/heliohq/ship/main/scripts/stop-gate.sh), [qa/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/qa/SKILL.md) |
| OpenHands | 3 | `ConfirmationPolicy` + `LLMSecurityAnalyzer` tagging risk per tool call, halts for human approval; separate "Critic" scores completion vs threshold and triggers automatic retries | [critic_example.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/34_critic_example.py) |

### 5. Session persistence
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 2 | File-based: plans/specs under `docs/superpowers/`, progress ledger in `.superpowers/sdd/`; `executing-plans` designed for cross-session resumption; no formal checkpoint/run-ID system | [executing-plans/SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/executing-plans/SKILL.md), [RELEASE-NOTES.md](https://raw.githubusercontent.com/obra/superpowers/main/RELEASE-NOTES.md) |
| claude-code-harness | 3 | `.claude/memory/` durable+git-shareable (decisions/patterns/session-log/context.json) vs `.claude/state/` local runtime; optional companion `harness-mem` MCP server (SQLite) gives true cross-session continuity | [MEMORY_POLICY.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/MEMORY_POLICY.md), [harness-mem](https://github.com/Chachamaru127/harness-mem) |
| ship | 2 | Lightweight `.ship/ship-auto.local.md` state file (orchestrator-exclusive write); durable artifacts as markdown under `docs/ship/<task-id>/`; explicitly rejects a separate "memory store" | [design/004](https://raw.githubusercontent.com/heliohq/ship/main/docs/design/004-stage-driven-workflow.md) |
| OpenHands | 3 | `persistence_dir` with `base_state.json` + numbered append-only `events/` JSON files; resumes by replay; CLI `--resume`, `--resume <id>`, `--resume --last` | [conversation/state.py](https://github.com/OpenHands/software-agent-sdk/blob/main/openhands-sdk/openhands/sdk/conversation/state.py), [OpenHands-CLI README](https://github.com/OpenHands/OpenHands-CLI/blob/main/README.md) |

### 6. Multi-agent / orchestrator boundary
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 2 | Model-agnostic by design (runs unmodified across host harnesses/models); subagent dispatch requires explicit model selection but no cost/routing guidance documented; no orchestrator of its own | [subagent-driven-development/SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/subagent-driven-development/SKILL.md) |
| claude-code-harness | 3 | Deliberately multi-model with config-driven routing: role tiers (lite/standard/deep/review/advisor/release) map to specific models per host, implemented in `model-routing.sh` | [model-routing-policy.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/model-routing-policy.md) |
| ship | 3 | Explicitly model-agnostic ("optimize for autonomy... not a named model version"); recommends reviewer be a different provider for independence; external verifier abstracts over codex/claude/custom | [use-ship/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/use-ship/SKILL.md) |
| OpenHands | 3 | Model-agnostic via LiteLLM (100+ providers); clear harness/orchestrator split: Agent Server (single-host REST API) vs Automation Server (event/schedule layer) vs Agent Canvas (frontend client) | [issue #13275](https://github.com/OpenHands/OpenHands/issues/13275) |

### 7. TypeScript quality
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 2 | Only 2.6% of repo; single file `.pi/extensions/superpowers.ts` uses ESM, typed `ExtensionAPI` import, type-guard narrowing; but sync file reads + silent error caching; no typed tool schemas end-to-end | [superpowers.ts](https://raw.githubusercontent.com/obra/superpowers/main/.pi/extensions/superpowers.ts) |
| claude-code-harness | 1 | 7.6% of repo per linguist, but no `.ts` source files could be located by directory browsing (bin/, scripts/, hooks/, skills/, templates/, monitors/ all checked); third-party blog claim of a "TS guardrail engine" contradicted by actual Bash-based tampering detector | [posttooluse-tampering-detector.sh](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/scripts/posttooluse-tampering-detector.sh) (negative evidence) |
| ship | N/A | No `.ts`/`.tsx` files found in repo (Shell/HTML/Go-Template only per linguist, confirmed by file-tree search) | [repo file-tree search](https://github.com/heliohq/ship/find/main) |
| OpenHands | 2 | Real source read (`sidebar.tsx`): modern ESM, TanStack Query hooks, i18next, React Router — but sampled component has no explicit prop/type annotations; typing concentrated at hook/API layer, not uniformly enforced; separate typed `@openhands/ui` library w/ declaration generation | [sidebar.tsx](https://raw.githubusercontent.com/OpenHands/OpenHands/main/frontend/src/components/features/sidebar/sidebar.tsx) |

### 8. Python quality
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 1 (mostly N/A) | 1.9% of repo; substantive Python lives in a separate `superpowers-evals` repo (not in main tree), using uv/ruff/ty per pre-commit config, but not independently inspectable | [.pre-commit-config.yaml](https://raw.githubusercontent.com/obra/superpowers/main/.pre-commit-config.yaml) |
| claude-code-harness | 1 | 1.4% of repo; two real files reviewed (`check-release-version-sync.py`, `final-scan-redaction.py`) — dataclasses, type hints, CLI flags, exit codes, but pure CI/utility scripts: no Pydantic, no async, no packaging | [check-release-version-sync.py](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/scripts/check-release-version-sync.py) |
| ship | N/A | No Python files found in repo (confirmed by file-tree search) | [repo file-tree search](https://github.com/heliohq/ship/find/main) |
| OpenHands | 3 | Real source read (`agent/base.py`): abstract Pydantic `BaseModel`, custom validators/serializers, sync+async `step()`/`astep()` parity, `py.typed` marker present; heavy async stack (FastAPI/aiohttp/sqlalchemy[asyncio]/asyncpg) | [agent/base.py](https://github.com/OpenHands/software-agent-sdk/blob/main/openhands-sdk/openhands/sdk/agent/base.py) |

### 9. Language-specific guidelines
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 1 (N/A-leaning) | Skills explicitly designed to be technology-agnostic; domain/language-specific skills pushed to separate plugins by policy, not core | [writing-skills/SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/writing-skills/SKILL.md) |
| claude-code-harness | 1 | Localization covers only human UI language (EN/JA), not target programming language; onboarding docs are generic; host-specific (Cursor/Codex) guides exist but not language-specific ones | [i18n-language-contract.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/i18n-language-contract.md) |
| ship | 1 | No per-language quickstarts; e2e skill auto-detects/scaffolds by tech stack rather than offering authored per-language guides | [e2e/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/e2e/SKILL.md) |
| OpenHands | 2 | Clear Python SDK quickstart with numbered example scripts; no equivalent dedicated TypeScript SDK quickstart found — TS guidance only implicit in frontend dev setup | (search snippet) docs.openhands.dev/sdk/getting-started |

### 10. Cross-host portability
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 3 | Standout feature: formal porting contract (shared content + thin per-harness tool-mapping layer + acceptance test) across 10 host harnesses, each with its own plugin manifest dir | [porting-to-a-new-harness.md](https://raw.githubusercontent.com/obra/superpowers/main/docs/porting-to-a-new-harness.md) |
| claude-code-harness | 2 | Honest graded tiers: Claude Code = supported, Codex CLI/OpenCode/Cursor = internal-compatible, Codex app/Copilot CLI = candidate; explicitly notes enforcement strength differs per host | [tool-capability-matrix.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/tool-capability-matrix.md) |
| ship | 3 | Parallel `.claude-plugin/`/`.codex-plugin/`/`.cursor-plugin/` dirs sharing `skills/`/`hooks/`; documented Codex-specific manifest workaround for a hooks-field validation rejection | [design/003](https://raw.githubusercontent.com/heliohq/ship/main/docs/design/003-codex-plugin-packaging.md) |
| OpenHands | 3 (reversed direction) | OpenHands is itself the host/runtime — Agent Canvas can drive *other* agents (Claude Code, Codex, Gemini CLI) as ACP subprocess backends, the inverse of being embedded inside them | docs.openhands.dev/openhands/usage/agent-canvas/acp-agents (search snippet) |

### 11. Review isolation strength
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 3 | Review dispatched to a fresh general-purpose subagent with no access to session history, diff-only context via `review-package`; collaborative not adversarial, but real context-blindness | [requesting-code-review/SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/requesting-code-review/SKILL.md) |
| claude-code-harness | 2 | Reviewer agent hard-restricted to Read/Grep/Glob (no Write/Edit/Bash); but project's own docs explicitly frame this as "collaborative self-review, not adversarial"; optional dual/team-debate modes add cross-checking but aren't default | [harness-review-operating-model.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/harness-review-operating-model.md) |
| ship | 3 | Mechanically enforced via PreToolUse hook: QA phase is actively blocked from reading `review.md`/`plan.md` ("independence violation") and from writing source code — verified in source, not just instructed | [phase-guardrail.sh](https://raw.githubusercontent.com/heliohq/ship/main/scripts/phase-guardrail.sh) |
| OpenHands | 2 | Official PR Review Action runs in a fresh sandboxed context with its own LLM key — not self-review — but a single automated pass, not described as adversarial/red-team by design | [PR Review Action](https://github.com/marketplace/actions/openhands-pr-review-action) |

### 12. Artifact/evidence requirements
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 3 | Hard gate: fresh-command + full-output + exit-code required before any completion claim; rejects "linter passed ≠ build succeeded"; contributor process requires session transcripts for new harness ports | [verification-before-completion/SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/verification-before-completion/SKILL.md) |
| claude-code-harness | 3 | Release preflight requires clean tree + changelog entry + zero mirror-drift (actually re-running validators) + CI status; review results persist to JSON and gate commits | [release-preflight.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/release-preflight.md) |
| ship | 3 | Documented evidence hierarchy: L1 proof (screenshots/curl/logs) required, L2 indicators ("tests passed") insufficient, L3 reasoning ("should work") auto-fails | [README.md](https://raw.githubusercontent.com/heliohq/ship/main/README.md) |
| OpenHands | 2 | PR template + CI workflow enforces human-asserted "tested" checkbox for contributions; Critic/threshold mechanism is the closest runtime analog, but no general runtime requirement that the agent itself produce artifacts before claiming completion | [AGENTS.md](https://raw.githubusercontent.com/OpenHands/OpenHands/main/AGENTS.md) |

### 13. Control-plane / invocation decoupling
| Repo | Score | Note | Source |
|---|---|---|---|
| superpowers | 1 | No standalone CLI/daemon/webhook/API; activates only via SessionStart hooks or in-session skill calls inside a host harness | [scripts/ listing](https://github.com/obra/superpowers/tree/main/scripts) (negative evidence) |
| claude-code-harness | 2 | Go binary itself is headless/stateless (usable in CI), but higher-level loop orchestration is explicitly confined to a single interactive session per project docs; cross-session auto-reentry deferred to a future phase | [long-running-harness.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/long-running-harness.md) |
| ship | 2 | No standalone CLI/API/webhook trigger; everything invoked inside a host session via hooks; Stop-gate can shell out to a non-interactive verifier call as a partial decoupling primitive | [stop-gate.sh](https://raw.githubusercontent.com/heliohq/ship/main/scripts/stop-gate.sh) |
| OpenHands | 3 | Automation Server is purpose-built for this: webhook-triggered (GitHub/GitLab/Slack/Datadog/cron), queue-based, no live session needed; also headless CLI (`--headless -t "task"`) and GitHub Action resolver | [issue #13275](https://github.com/OpenHands/OpenHands/issues/13275) |

---

## Language deep-dive

**TypeScript.** Only OpenHands has a real, substantial TypeScript surface (frontend + standalone `@openhands/ui` library with declaration generation); even there, type coverage is uneven at the component level. Superpowers' TS footprint is a single, small, reasonably-typed extension file. claude-code-harness's reported 7.6% TS could not be located in source — treat as unverified. Ship has no TypeScript at all. **None of the four ship a typed tool-schema pipeline end-to-end** — OpenHands gets closest via Pydantic-typed tools on the Python side, not TS.

**Python.** OpenHands is the only repo with a real Python *runtime* (Pydantic-based agent core, async-first, `py.typed` distributed). Superpowers and claude-code-harness use Python only for narrow CI/tooling scripts (redaction scanning, version-sync checks) — competent but not idiomatic "framework" Python (no Pydantic/async/packaging). Ship has no Python at all.

**Implication:** dimensions 7–9 are only meaningfully differentiating for OpenHands. For the other three, the harness's own implementation language is irrelevant to its value — they are markdown/shell skill packages riding on a host agent's runtime, so "TS/Python quality" should really be asked of the *host* (Claude Code, Codex, etc.), not the harness.

---

## Shortlist recommendations

- **Want a single self-hosted product with its own runtime, multi-backend control, and automation/webhook triggers:** OpenHands. It's the only candidate with real, verifiable evidence across all 13 dimensions and the only one with a genuine TS+Python codebase to evaluate.
- **Want the most rigorous, mechanically-enforced quality gate on top of an existing coding agent, and don't mind no Python/TS app code to inspect:** ship — its review-isolation claim is the only one of the three skill-frameworks independently verified at the tool-permission level (PreToolUse hook actually blocks cross-phase file reads), and its evidence-hierarchy is the most explicit anti-hallucination control found.
- **Want the broadest cross-host reach with a battle-tested verification discipline (and don't need built-in memory/persistence machinery):** superpowers — strongest portability story (10 hosts, formal porting contract) and the most explicit "no completion claims without fresh evidence" skill, but weakest on session persistence and has zero control-plane/automation story.
- **Want config-driven multi-model routing and the most evidence-gated release process:** claude-code-harness — the only one of the three skill-frameworks with first-class, declarative model routing per role and a release pipeline that re-validates mirror drift rather than trusting status, but its own docs honestly disclose that review is "collaborative, not adversarial" and that cross-session automation isn't shipped yet.
- **Caution:** superpowers' reported star count (~240k) is unverified and should be re-checked against an authenticated GitHub API call before being cited externally.
