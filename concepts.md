# concepts.md — Phase 1 Exploratory Findings

Source pass: README + top-level structure for each candidate, fetched via WebFetch
(GitHub repo pages; raw.githubusercontent.com not needed since pages rendered).
See `sources.md` for exact URLs and fetch status.

Note on access: `paulhoekstra.substack.com/p/agentic-engineering-the-guardrails`
returned **HTTP 403 Forbidden** on every fetch attempt — it appears paywalled/
blocked to automated fetchers. It is **not a GitHub repo** and contributes no
verifiable evidence to this comparison. It is excluded from Phase 2 scoring
and flagged here as inaccessible (see `sources.md`).

---

## New concepts encountered (not in original dimension list)

### 1. Cross-host skill/plugin packaging
- **Repos:** superpowers, claude-code-harness, ship
- **Description:** The harness ships as a portable "skill" or "plugin" bundle
  (`.claude-plugin`, `.cursor-plugin`, `.codex-plugin`, `.kimi-plugin`) rather
  than a standalone runtime — the same skill definitions are loaded by
  multiple *host* coding agents (Claude Code, Cursor, Codex, Gemini CLI, etc).
- **Load-bearing:** Yes for all three — this is their core distribution model.
  It also means dimensions like "tool execution layer" (#2) and language
  quality (#7/#8) are largely inherited from the host agent, not owned by
  the harness itself.

### 2. Review isolation / adversarial verification
- **Repos:** ship (most explicit); claude-code-harness ("Review" as
  independent verification step separate from implementation)
- **Description:** The reviewer phase is mechanically prevented from seeing
  the implementer's context/reasoning, forcing review against the spec
  rather than rubber-stamping. Ship calls this out explicitly ("reviewers
  see no implementation context").
- **Load-bearing:** Yes for ship — central to its pitch ("makes failure modes
  structurally impossible"). Incidental-but-present in claude-code-harness.

### 3. Evidence hierarchy for verification claims
- **Repos:** ship
- **Description:** Verification steps require artifacts (screenshots, logs)
  over the agent's self-reported assertions ("it works").
- **Load-bearing:** Yes for ship's QA/handoff phases; not observed elsewhere.

### 4. File-as-contract / disk-based state
- **Repos:** claude-code-harness (`spec.md`, `Plans.md` as source-of-truth
  contracts), ship ("disk-based state management for resumability")
- **Description:** Distinct from generic "session persistence" (dimension
  #5) — the emphasis is on human-readable, git-trackable files that function
  as a control surface between phases/agents, not just a resume mechanism.
- **Load-bearing:** Yes — both frameworks anchor their multi-phase pipeline
  on these files rather than in-memory or hidden state.

### 5. Git-worktree-per-task isolation
- **Repos:** superpowers
- **Description:** A workflow phase that creates an isolated git worktree
  before development starts, so the working tree for a task never collides
  with other in-flight work.
- **Load-bearing:** Yes — it's phase 2 of the documented 7-phase workflow.

### 6. Hard-enforced RED-GREEN-REFACTOR as a workflow gate
- **Repos:** superpowers
- **Description:** TDD isn't a guideline but a scripted phase the agent must
  pass through (write failing test → make it pass → refactor) before code
  review.
- **Load-bearing:** Yes for superpowers' methodology claim ("works").

### 7. Decoupled automation/control-plane server
- **Repos:** OpenHands
- **Description:** An "Agent Server" (REST API) is separable from the
  interactive chat loop, plus an optional "Automation Server" for
  scheduled/event-triggered runs (Slack, GitHub, Linear triggers). This is a
  distinct concern from "multi-agent orchestration" (#6 original) — it's
  about decoupling *invocation* from *interaction*.
- **Load-bearing:** Yes — central to OpenHands' positioning as a "developer
  control center" rather than a single chat agent.

### 8. Host-agent compatibility tiering
- **Repos:** claude-code-harness ("Supported" vs "Internal-compatible" vs
  "Candidate" support levels for different coding agents)
- **Description:** An explicit compatibility matrix ranking how well the
  harness works on top of each host agent, implying the harness is a layer
  *on top of* other harnesses rather than a peer.
- **Load-bearing:** Incidental — informative for adoption risk, but not
  core to functionality.

### 9. Risk-classified refactor phase
- **Repos:** ship
- **Description:** Refactoring work is explicitly tagged by risk level
  before changes are made, rather than treated uniformly.
- **Load-bearing:** Incidental — a refinement of the verification/guardrails
  dimension (#4 original), not a new axis on its own.

---

## Proposed dimension-list changes for Phase 2

**Additions:**
- **10. Cross-host portability** — does the harness run as a payload inside
  multiple host agents, or is it itself the runtime? (directly affects
  whether #2/#7/#8 are even answerable for that repo)
- **11. Review isolation strength** — is verification done by a context-
  blind/adversarial step, or self-review by the same context?
- **12. Artifact/evidence requirements** — are claims of completion backed
  by required artifacts (logs, screenshots, test output) vs free-text
  self-report?
- **13. Control-plane / invocation decoupling** — can the harness be
  triggered/automated without a live interactive session (webhooks,
  schedules, REST API), separate from multi-agent *routing* (#6)?

**Original dimensions — status after Phase 1:**
- **Confirmed strongly relevant:** #3 Planning & decomposition, #4
  Verification & guardrails, #5 Session persistence (all four repos have
  visible, differentiated mechanisms here).
- **Confirmed relevant but bimodal:** #6 Multi-agent/orchestrator boundary —
  rich and central for OpenHands (Agent Server, multi-backend); essentially
  absent for the three skill-framework repos, which have no orchestrator of
  their own and ride entirely on the host agent's model.
- **Weak / largely N/A for 3 of 4 repos:** #2 Tool execution layer, #7
  TypeScript quality, #8 Python quality, #9 language-specific guidelines —
  superpowers, claude-code-harness, and ship are predominantly Shell/skill-
  markdown packages with no owned language runtime to evaluate; these
  dimensions will mostly score N/A for them, with real signal only from
  OpenHands (Python 64.8% / TypeScript 33.8%) and partially from
  claude-code-harness's minority Go/TS/JS/Python components.
- **Context lifecycle (#1)** — present in all four conceptually (compaction/
  context-rot avoidance), but documented with very different granularity;
  retained as-is for Phase 2.

Proceeding to Phase 2 with dimensions 1–9 (original) + 10–13 (new).
