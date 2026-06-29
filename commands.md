# commands.md — Command Surface Maps

Sources for every command below are inline; `[inferred]` marks anything
read from source code or filenames rather than from docs/README usage
sections, per task constraints. The Substack URL is not a repo and has no
command surface.

---

### Superpowers command surface

No standalone CLI binary ships with this repo — Superpowers is a
context-injection/skill layer activated inside a host agent session.

**Runtime invocation (per-host install, from README usage section)**
```
/plugin install superpowers@claude-plugins-official   — Claude Code install
agy plugin install https://github.com/obra/superpowers — Antigravity install
/plugins → search "superpowers" → Install Plugin       — Codex CLI install
/add-plugin superpowers                                 — Cursor install
gemini extensions install https://github.com/obra/superpowers — Gemini CLI install
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace — GitHub Copilot CLI install
/plugins install https://github.com/obra/superpowers    — Kimi Code install
pi install git:github.com/obra/superpowers              — Pi install
```
Source: [README.md](https://raw.githubusercontent.com/obra/superpowers/main/README.md), [README.kimi.md](https://raw.githubusercontent.com/obra/superpowers/main/docs/README.kimi.md), [README.opencode.md](https://raw.githubusercontent.com/obra/superpowers/main/docs/README.opencode.md) (OpenCode requires manual `opencode.json` plugin entry instead of an install command)

**Deprecated slash commands** (removed v5.1.0 — kept for historical map)
```
/brainstorm     — superseded by the `brainstorming` skill triggered conversationally
/execute-plan   — superseded by `executing-plans` skill
/write-plan     — superseded by `writing-plans` skill
```
Source: [RELEASE-NOTES.md](https://raw.githubusercontent.com/obra/superpowers/main/RELEASE-NOTES.md)

**Skills (conversationally triggered, not typed commands)** — `skills/`
```
brainstorming, dispatching-parallel-agents, executing-plans,
finishing-a-development-branch, receiving-code-review,
requesting-code-review, subagent-driven-development,
systematic-debugging, test-driven-development, using-git-worktrees,
using-superpowers, verification-before-completion, writing-plans,
writing-skills
```
Source: [skills/ listing](https://github.com/obra/superpowers/tree/main/skills)

**Hooks (session lifecycle, not user-invoked)**
```
SessionStart (matcher: startup|clear|compact) → hooks/session-start[-codex]
```
Source: [hooks.json](https://raw.githubusercontent.com/obra/superpowers/main/hooks/hooks.json)

**Maintainer-only scripts** `[inferred]`
```
scripts/bump-version.sh
scripts/lint-shell.sh
scripts/sync-to-codex-plugin.sh
```
Source: [scripts/ listing](https://github.com/obra/superpowers/tree/main/scripts) — internal tooling, not user-facing

**Observations**
- Entirely install-command + conversational-skill surface; no flags, no
  subcommand hierarchy, no config-set verbs at all.
- The complete absence of a `harness run`/`harness resume`/`harness config`
  surface is itself a signal: this confirms dimension 13 (control-plane
  decoupling) score of 1 — there is no invocation path outside a live
  session.
- Per-host install syntax varies wildly (slash command, CLI subcommand,
  JSON config edit, custom `pi install` verb) — no unified CLI vocabulary
  across hosts, by design (it inherits whatever vocabulary the host uses).

---

### claude-code-harness command surface

**Runtime invocation (skills = slash commands)**
```
/harness-plan                                  — draft/update spec.md + Plans.md
/work [all] [task-number|range] [--codex]
      [--parallel N] [--no-commit] [--resume id]
      [--breezing] [--auto-mode] [--tdd-bypass]  — execute approved tasks
/harness-review [code|plan|scope] [--quick]
      [--codex-closeout] [--dual] [--team-debate]
      [--security] [--ui-rubric]                 — independent verification
/harness-sync [--snapshot] [--no-retro]
      [--plan NAME]                              — reconcile Plans.md vs git vs traces
/release [patch|minor|major] [--dry-run]          — versioned, gated release
/breezing [--codex]                               — shorthand for parallel team mode
```
Source: [harness-work/SKILL.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/skills/harness-work/SKILL.md), [harness-review/SKILL.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/skills/harness-review/SKILL.md), [harness-sync/SKILL.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/skills/harness-sync/SKILL.md), [harness-release/SKILL.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/skills/harness-release/SKILL.md), [CLAUDE-commands.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/CLAUDE-commands.md)

**Configuration**
```
/harness-setup                — initial project setup
/validate                     — validate plugin/config integrity
/reload-plugins                — reload skill/plugin definitions
harness.toml                   — permission tiers, network-egress denylist,
                                  TDD-enforcement toggle, self-review rules
.claude-code-harness.config.yaml — review.codex.enabled, breezing.default_parallel,
                                  advisor.{claude_model,codex_model}, monitor.plans_drift
```
Source: [docs/CLAUDE-commands.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/CLAUDE-commands.md), [harness.toml](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/harness.toml), [.claude-code-harness.config.yaml](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/.claude-code-harness.config.yaml)

**Tool management / memory**
```
/remember                     — write to .claude/memory/
/handoff-to-cursor             — cross-host handoff helper
```
Source: [docs/CLAUDE-commands.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/docs/CLAUDE-commands.md)

**Go binary (`bin/harness`) subcommands** `[inferred from go/DESIGN.md, not raw .go source]`
```
harness hook {pretool|posttool|permission|session-start|session-end|stop|
  pre-compact|post-compact|task-completed|task-created|permission-denied|
  teammate-idle|notification|config-change|user-prompt|todo-sync|
  subagent-start|subagent-stop|setup|instructions-loaded|worktree-create|
  worktree-remove|cwd-changed|file-changed|elicitation|elicitation-result|
  post-tool-failure}            — ~28 hook event routes, stdin→route→stdout
harness effort <task-desc>      — task effort estimation
harness plans {sync|update}     — Plans.md reconciliation
harness version
harness worker {auto-test|ci-check}
```
Source: [go/DESIGN.md](https://raw.githubusercontent.com/Chachamaru127/claude-code-harness/main/go/DESIGN.md)

**Observations**
- The `pre-compact`/`post-compact` hook pair is a direct architectural tell
  for dimension 1 (context lifecycle) — explicit compaction awareness baked
  into the command surface itself, not just documented behavior.
- `--dual` / `--team-debate` flags on `/harness-review` reveal that
  adversarial review is opt-in, not default — corroborating the
  "collaborative self-review, not adversarial" framing found in docs.
- `harness.toml`'s TDD-enforcement toggle defaulting to **off** is notable:
  the harness ships the capability but doesn't force it by default.
- This is the only one of the three skill-frameworks with a real compiled
  binary (`bin/harness`) underneath the slash-command surface, giving it a
  CI-usable (`harness worker ci-check`) headless command absent from
  superpowers/ship.

---

### Ship command surface

**Runtime invocation (skills = slash commands)**
```
/ship:use-ship <goal>      — routing entrypoint, picks skill/phase/full pipeline
/ship:auto                 — full pipeline: design→dev→e2e→review→qa→refactor→handoff
/ship:design                — spec+plan with adversarial peer challenge (6 phases)
/ship:dev                   — implementation w/ peer cross-validation, wave parallelism
/ship:e2e                   — codifies acceptance criteria as executable tests
/ship:review                 — defect-only code review, P1/P2/P3 severity
/ship:qa [--recheck]         — exploratory runtime testing
/ship:refactor                — risk-classified, behavior-preserving cleanup
/ship:handoff                 — commit/push/PR, CI fix loop (max 3 rounds)
/ship:arch-design              — system architecture planning (pre-design)
/ship:write-docs               — structured documentation generation
/ship:visual-design             — frontend/UI design-system creation
```
Source: [skills/use-ship/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/use-ship/SKILL.md), [skills/design/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/design/SKILL.md), [skills/dev/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/dev/SKILL.md), [skills/e2e/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/e2e/SKILL.md), [skills/review/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/review/SKILL.md), [skills/qa/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/qa/SKILL.md), [skills/handoff/SKILL.md](https://raw.githubusercontent.com/heliohq/ship/main/skills/handoff/SKILL.md), [docs/skills.md](https://raw.githubusercontent.com/heliohq/ship/main/docs/skills.md)

**Configuration / install**
```
/plugin marketplace add heliohq/ship   — Claude Code install
(Codex plugin sidebar)                  — Codex install
/add-plugin ship                        — Cursor install
.mcp.json: { codex: { command: codex, args: [mcp-server] } } — optional external verifier
```
Source: [README.md](https://raw.githubusercontent.com/heliohq/ship/main/README.md), [.mcp.json](https://raw.githubusercontent.com/heliohq/ship/main/.mcp.json)

**Hooks (session lifecycle, not user-invoked)**
```
SessionStart  → scripts/session-start.sh   — inject routing hint only
PreToolUse    → scripts/phase-guardrail.sh — blocks cross-phase file access
Stop          → scripts/stop-gate.sh       — external verifier renders
                                              TASK_COMPLETE/INCOMPLETE/BLOCKED
```
Source: [hooks/hooks.json](https://raw.githubusercontent.com/heliohq/ship/main/hooks/hooks.json)

**Scripts** `[inferred from filenames; only 3 of 7 contents independently fetched]`
```
scripts/auto-orchestrate.sh
scripts/generate-docs-index.sh
scripts/path-bootstrap.sh
scripts/phase-guardrail.sh      (content verified)
scripts/pr-readiness.sh
scripts/session-start.sh         (content verified)
scripts/stop-gate.sh             (content verified)
```
Source: [scripts/ listing](https://github.com/heliohq/ship/tree/main/scripts)

**Observations**
- `/ship:qa --recheck` is the only flag-bearing command found — the rest of
  the surface is bare slash-commands, contrasting with claude-code-harness's
  heavily flagged `/work`/`/harness-review`.
- The hook-enforced phase-guardrail (blocking QA from reading `review.md`)
  is invisible at the command-surface level — it's a silent constraint, not
  a documented flag — meaning users discover review isolation only by
  reading source, not by inspecting available commands.
- No standalone CLI/binary; no headless/CI-invocable command exists,
  consistent with dimension 13 score of 2 (the Stop-gate's shell-out to an
  external verifier is the closest thing to a non-interactive invocation).

---

### OpenHands command surface

**Runtime invocation**
```
openhands                              — interactive TUI (default; per-action confirm)
openhands -t "task"                    — inline task, non-interactive-ish
openhands -f instructions.md           — task from file
openhands --headless -t "task"         — non-interactive/CI mode
openhands --resume                     — resume most recent conversation
openhands --resume <id>                — resume specific conversation
openhands --resume --last              — resume last
openhands --always-approve / --yolo    — auto-approve all actions
openhands --llm-approve                — LLM-based security analyzer approval mode
openhands --json                       — JSON output for scripted use
openhands --override-with-envs         — apply LLM_MODEL/LLM_API_KEY/LLM_BASE_URL env vars
openhands --no-condense                — disable context condensation [inferred — bug report #8630, not formal docs]
openhands acp                          — Agent Client Protocol server (IDE integrations)
openhands web                          — browser-based UI for the CLI
openhands serve                        — full Docker-backed GUI server
openhands cloud -t "task"              — run task on OpenHands Cloud
```
Source: [OpenHands-CLI README](https://github.com/OpenHands/OpenHands-CLI/blob/main/README.md), [issue #8630](https://github.com/OpenHands/OpenHands/issues/8630) (for `--no-condense`)

**Configuration**
```
openhands login                        — authenticate with OpenHands Cloud
```
Source: [OpenHands-CLI README](https://github.com/OpenHands/OpenHands-CLI/blob/main/README.md)

**Tool management**
```
openhands mcp list                     — list configured MCP servers
openhands mcp add <name>               — add an MCP server
openhands mcp enable <name>            — enable an MCP server
openhands mcp disable <name>           — disable an MCP server
```
Source: [OpenHands-CLI README](https://github.com/OpenHands/OpenHands-CLI/blob/main/README.md)

**In-session slash commands (inside the TUI)**
```
/help                                  — list all commands
/init                                  — initialize repo exploration + project docs
```
Source: [OpenHands-CLI README](https://github.com/OpenHands/OpenHands-CLI/blob/main/README.md)

**Installation**
```
uv tool install openhands --python 3.12
curl -fsSL https://install.openhands.dev/install.sh | sh
npm install -g @openhands/agent-canvas   then  agent-canvas
docker run ...  (volume-mounted, see README)
```
Source: [README.md](https://raw.githubusercontent.com/OpenHands/OpenHands/main/README.md), [OpenHands-CLI README](https://github.com/OpenHands/OpenHands-CLI/blob/main/README.md)

**Automation Server (no CLI command — webhook/cron-triggered, separate control plane)**
```
Phase 1: cron-triggered runs
Phase 2: GitHub event-triggered runs
Phase 3: generic webhook-triggered runs (GitHub/GitLab/Slack/Datadog)
```
Source: [issue #13275](https://github.com/OpenHands/OpenHands/issues/13275) `[inferred from RFC issue, feature in phased rollout — not yet a stable documented command surface]`

**Observations**
- `--resume`/`--resume <id>`/`--resume --last` is the most granular resume
  command surface of the four — directly evidencing the strong session-
  persistence score (dimension 5).
- The Automation Server's triggers (cron/webhook) are **not CLI commands at
  all** — they're a separate REST/event control plane, which is exactly the
  "control-plane decoupling" concept flagged in `concepts.md` (#7) and the
  reason OpenHands scores 3 on dimension 13 while the others score 1–2.
  This category — invocation without any CLI command whatsoever — has no
  equivalent in superpowers, claude-code-harness, or ship.
- `mcp add/list/enable/disable` is the only explicit "tool management"
  command category found across all four harnesses — the other three
  manage tools implicitly via host-agent config files, not their own CLI
  verbs.
- `--llm-approve` vs `--always-approve`/`--yolo` directly exposes the
  human-in-loop vs autonomy tradeoff as a first-class flag, more explicit
  than any guardrail-related flag in the other three harnesses.

---

## Cross-harness command category table

| Category | superpowers | claude-code-harness | ship | OpenHands |
|---|---|---|---|---|
| Standalone CLI binary | ✗ | ✓ (`bin/harness`, Go) | ✗ | ✓ (`openhands`, separate repo) |
| Slash-command surface (host-injected) | ✓ (skills, no flags) | ✓ (5 verbs, heavily flagged) | ✓ (12 verbs, mostly flag-free) | ✓ (`/help`, `/init` — minimal, TUI-internal) |
| Per-host install commands | ✓ (9 hosts, 9 distinct syntaxes) | ✓ (tiered: supported/internal-compatible/candidate) | ✓ (3 hosts: Claude Code/Codex/Cursor) | N/A — OpenHands is the host |
| Resume / session continuation flag | ✗ (file convention only, no flag) | ✓ (`--resume id` on `/work`) | ✗ (state file only, no resume flag) | ✓ (`--resume`, `--resume <id>`, `--resume --last`) |
| Explicit compaction-lifecycle command/hook | ✓ (SessionStart matcher incl. `compact`) | ✓ (`harness hook pre-compact`/`post-compact`) | ✗ (SessionStart injects hint only, no compact-specific hook) | ✓ (condenser + `--no-condense` flag) |
| Tool/MCP management subcommand | ✗ | ✗ (config file only) | ✗ (config file only) | ✓ (`mcp list/add/enable/disable`) |
| Headless/CI-invocable command | ✗ | ✓ (`harness worker ci-check`) | ✗ | ✓ (`--headless`) |
| Human approval / autonomy flag | ✗ (no flag — convention only) | ✗ (config-file toggle, no CLI flag) | ✗ (Stop-gate verdict, no user flag) | ✓ (`--always-approve`/`--yolo`/`--llm-approve`) |
| Release/versioning command | ✗ | ✓ (`/release patch\|minor\|major --dry-run`) | ✗ (`/ship:handoff` covers PR/CI, not versioning) | N/A (handled by repo's own release workflows, not a user command) |
| Adversarial/dual-review flag | ✗ (default isolated, no flag needed) | ✓ (`--dual`, `--team-debate`) | ✗ (isolation is default, not flag-gated) | ✗ (PR Review Action is a separate CI job, not a CLI flag) |
| Automation/webhook control plane (non-CLI) | ✗ | ✗ | ✗ | ✓ (Automation Server: cron/webhook-triggered) |
| Config-set style command | ✗ (file edit only) | ✓ (`/harness-setup`, `/validate`, `/reload-plugins`) | ✗ (file edit only) | ✓ (`openhands login`) |

**Missing-category observations:**
- **No harness offers a config-set CLI verb** in the literal `harness config set <key>` sense from the task's template — all four either use slash-commands that wrap config files, or edit YAML/TOML/JSON files directly. This category from the original template (PHASE 3 spec) is **absent across the board**.
- **Tool/MCP management as a first-class CLI category exists only in OpenHands** — the three skill-frameworks treat tools as inherited from the host, never their own subcommand surface.
- **Automation/control-plane-without-a-CLI-command is unique to OpenHands** and is the strongest command-surface signal of the "control-plane decoupling" concept proposed in `concepts.md`.
- **Adversarial-review-as-a-flag exists only in claude-code-harness** (`--dual`/`--team-debate`) — superpowers and ship treat isolation as a structural default rather than an opt-in toggle, which is arguably a stronger design (no risk of forgetting to pass the flag).
