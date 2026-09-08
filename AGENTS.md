<!-- ws:dev begin — generated; do not hand-edit -->
<!-- ws:dev source=wrightstrategy/bridge:conventions/dev.md version=1 sha256=1d090e9ce3487a30d646d17501de22122475060d051c23d418feee09aa2eeea1 -->
# Coding Universe

Shared conventions for Wright Strategy development repositories. This resident block is
**defaults and tripwires only** (ADR-028). Reference payload:
`docs/agents/ws-dev-reference.md`. Workflow policy lives in the skill that owns the workflow.

## Standing defaults

- **Mindset:** GitOps — git is the source of truth; declarative over manual.
- **Communication:** write for the operator, who has not loaded your skills or vocabulary.
  Lead with what changed and what you need from them; a term of art appears only beside its
  plain meaning. If a report needs "now in English, what did you do?", it failed.
- **Ordinary tier (ADR-026):** write implementation yourself, in-session; a different
  model family reviews it before the human merge gate. Parallelism or isolation, when a task
  earns it, runs on Orca orchestration inside a ready runtime — never a hidden host subagent
  or a silent executor CLI. Workers never dispatch onward and are released before teardown.
  ADRs, specs, and plans you always write yourself.
- **Review gates:** no implementation PR meets the human gate without `ws-dev:pr-review`;
  no finished spec or plan without `ws-dev:doc-review`. One exemption: **Tiny** work
  (trivial, behavior-preserving — CI is the check) may skip the agent review; when in
  doubt it is not Tiny. Every departure exists only on the gate record — a silent skip is
  never a waiver. Gate policy, waiver classes, and declared post-verdict deltas live in those
  skills.
- **Work management (ADR-029):** org work lives in Linear (`ws-dev:linear`) — planned work files
  complete into Backlog; only bugs get priority. Blocked until it lands? file to Todo +
  `agent-waiting`, saying what is blocked. A mid-task **discovery is a hypothesis, not work**:
  file it to **Triage** with the discovery template + your exponential estimate; the nightly
  pass adjudicates — never promote your own finding to Backlog
  (`docs/standards/work-management.md#agent-discovery-triage`). Route by the repository's
  `linear_team`; multi-repo teams require its `repo:` label. Repos with
  `change_control = "platform"` use **PLT** and `ws-dev:repo-request`. Never a GitHub issue — GitHub keeps code, PRs, CI; link
  PRs to their Linear issue so state syncs on merge.
- **Secrets:** never in git — commit `.env.example`, keep values in 1Password / SOPS / GitHub
  Environments. Apps fail fast on missing config; prefer OIDC or short-lived tokens.
- **Python:** always `uv`, never pip; commit `uv.lock`; never commit `.venv/`.
- **TypeScript:** Bun, never Node, for agent and scripting code; commit `bun.lock`;
  `"strict": true`.
- **Commits:** Conventional Commits, imperative subject <70 chars, body says *why*. Agent
  commits carry `Authored-By: Claude|Codex|Grok` (reconciliation: `Reconciled-By:`);
  `Co-Authored-By:` only for an agent that wrote shipped bytes.
- **Deploy steps:** what a change needs outside the image (env var, migration, job, cutover
  order, out-of-app step) is an entry in `deploy/deploy_next.md` in the same PR, edited in
  place when a later change supersedes it — never a commit trailer, which cannot be
  corrected once written.
- **Testing:** `bun test` / `pytest`; test names describe behavior; real dependencies over
  mocks that can drift.
- **CI runners:** private-repo CI defaults to `runs-on: [self-hosted, ci]` (fresh isolated
  VM, no LAN route); GitHub-hosted is a documented exception.
- **Runtime:** one foreground process, stdout/stderr logs, clean `SIGTERM`; HTTP services
  expose `/healthz` and `/readyz`; migrations are explicit, idempotent deploy steps.
- **Docs:** living docs are the only authority and update with implementation; a decision
  lands as a living-doc edit plus a dated `docs/decisions.md` entry in the same PR (ADRs are
  a frozen archive — never write or supersede one). Before finishing: `ws-dev:doc-audit`, or
  state `docs not needed: <reason>`.
- **Releases:** Python: python-semantic-release, tag-only, dispatched; JS/TS: Changesets;
  release-please is deprecated. Notes let a deployer bump a pin without reading source:
  the filed deploy entries lead, then breaking changes and the image index digest.
- **Platform repos (ADR-029):** from outside the platform stream never write `bridge`, `homelab`,
  `dotfiles`, or the homelab cluster — no clone, PR, `kubectl`, SSH; file PLT. Inside it, write
  all three.
- **Instruction files:** `AGENTS.md` is each repo's canonical guidance; `CLAUDE.md` is the
  one-line `@AGENTS.md` stub. Repo-specific guidance stays outside the generated block;
  scaffold with `repo-bootstrap`.

## Tripwires

| Before you… | Read |
|---|---|
| write GitHub Actions beyond runner tier, `timeout-minutes`, one-run-per-change | `docs/agents/ws-dev-reference.md#github-actions` |
| write any `concurrency:` stanza, merge-queue config, or deploy workflow | `docs/agents/ws-dev-reference.md#concurrency-and-merge-queues` |
| write Dockerfiles, container CI, or release-promote flows | `docs/agents/ws-dev-reference.md#container-images--ci` |
| ship images or touch secret-scan gates | `docs/agents/ws-dev-reference.md#supply-chain-integrity` |
| edit Dependabot/Renovate config, registries, or osv-scanner | `docs/agents/ws-dev-reference.md#dependency-updates` |
| write a deploy step, or rotate/file the deploy-steps file | `docs/agents/ws-dev-reference.md#deploy-steps` |
| set up release automation, cut a release, or hotfix production | `docs/agents/ws-dev-reference.md#versioning--releases` |
| touch GitHub Packages auth (a 403 on "correct" config is this) | `docs/agents/ws-dev-reference.md#github-packages` |
| adopt graphify or change graph commit/ignore rules | `docs/agents/ws-dev-reference.md#knowledge-graph-graphify` |
| build or modify a web app | `wrightstrategy/web-ui` — shared UI spine, `create-app`, earn-its-place |
| parallelize or isolate work in an Orca runtime | `docs/agents/ws-dev-reference.md#parallel-and-isolated-work` |
| work in an Orca worktree | `docs/agents/ws-dev-reference.md#orca-worktree-checkpoints` |
| record, waive, or depart from any review gate | `ws-dev:pr-review` / `ws-dev:doc-review` |
| need filing/routing rules beyond the two destinations | `docs/standards/work-management.md` (in `wrightstrategy/bridge`) |
<!-- ws:dev end -->

<!-- ws:system-context begin — generated from wrightstrategy/bridge registry.toml; do not hand-edit -->
This repo is: GPSD location telemetry publisher for MQTT and Traccar.
It **consumes**: nothing yet. It's **consumed by**: nothing yet.
Full map: `wrightstrategy/bridge` → `ARCHITECTURE.md`.
<!-- ws:system-context end -->
