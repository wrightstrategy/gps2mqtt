<!-- ws:dev begin — generated; do not hand-edit -->
<!-- ws:dev source=wrightstrategy/bridge:conventions/dev.md version=1 sha256=c2147c827077531164a708db761306f50689047725d48254b635a07e7b246bb8 -->
# Coding Universe

Shared Wright Strategy rules: **Binding** boundaries cannot be weakened locally;
**Default** choices apply unless replaced in root `AGENTS.md` → `Local departures`.
Record the default, scope, replacement, and reason outside generated blocks. Definitions
and format: `docs/agents/ws-dev-reference.md#standard-authority-and-local-departures`.
Workflow policy stays in its owning skill; local departures never waive gates or checks.

<a id="standing-defaults"></a>

## Standing rules

- **Default — Mindset:** GitOps — git is the source of truth; declarative over manual.
- **Default — Communication:** write for the operator. Lead with what changed and what you
  need from them; explain terms beside their plain meaning.
- **Default — Ordinary work:** write implementation yourself, in-session.
- **Binding — Agent roles:** primary authors write plans/specs; earned parallelism or isolation
  uses Orca orchestration in a ready runtime, never hidden host subagents or silent executor
  CLIs. Workers never dispatch onward and are released before teardown. Shipped work meets
  the owning review gate however it was produced.
- **Binding — Review gates:** use `ws-dev:pr-review` for implementation PRs and
  `ws-dev:doc-review` for finished specs/plans before the human gate. Tiny work
  (trivial, behavior-preserving) and other exceptions use only the owning skill's recorded
  paths. Every departure belongs on the gate record; a silent skip is never a waiver.
- **Binding — Work management:** before filing development work, use `ws-dev:linear` and
  its canonical `docs/standards/work-management.md` filing contract in bridge. Platform
  requests use `ws-dev:repo-request`. Track agent work in Linear, never GitHub issues;
  link PRs to Linear.
- **Binding — Secrets:** never in git.
- **Default — Configuration:** commit `.env.example`; keep values in 1Password / SOPS /
  GitHub Environments. Fail fast on missing config; prefer OIDC or short-lived tokens,
  subject to applicable security/authentication contracts.
- **Default — Python:** use `uv`, not pip; commit `uv.lock`; never commit `.venv/`.
- **Default — TypeScript:** Bun for agent/scripting code; commit `bun.lock`; `"strict": true`.
- **Default — Commits:** Conventional Commits, imperative subject <70 chars; body says why.
- **Binding — Attribution:** agent commits carry truthful `Authored-By: Claude|Codex|Grok`
  (reconciliation: `Reconciled-By:`); `Co-Authored-By:` only for authors of shipped bytes.
- **Binding — Deploy steps:** record required external actions (env, migration, job, cutover,
  out-of-app step) in `deploy/deploy_next.md` in the same PR; edit superseded entries in place.
  Follow the reference's existing migration scope for repos still on `Deploy:` trailers.
- **Default — Testing:** `bun test` / `pytest`; behavioral test names; real dependencies over
  mocks that can drift. Required checks and verification obligations remain in force.
- **Default — CI runners:** private CI uses `runs-on: [self-hosted, ci]` (fresh VM, no LAN);
  document GitHub-hosted exceptions. Generated workflows remain generator-owned.
- **Default — Runtime:** one foreground process, stdout/stderr logs, clean `SIGTERM`;
  HTTP `/healthz` + `/readyz`; explicit, idempotent migrations. Preserve actual consumer contracts.
- **Binding — Docs:** reconcile living/reference docs with changes. Decisions land in their
  living home plus dated `docs/decisions.md` entries; ADRs are frozen (never extend or supersede).
  Finish with `ws-dev:doc-audit` or `docs not needed: <reason>`.
- **Default — Releases:** Python: python-semantic-release, tag-only, dispatched; JS/TS:
  Changesets. release-please is deprecated; migrate when release tooling is touched.
- **Binding — Release information:** accurate deployment changes, breaking changes and image
  index digest let a deployer bump a pin without reading source. Preserve published interfaces.
- **Binding — Platform:** outside the platform stream never write `bridge`, `homelab`,
  `dotfiles`, or the cluster (no clone, PR, `kubectl`, SSH); file PLT. Inside it, write all three
  under the target repo's own gates. Existing owner-authorized exceptions retain their scope.
- **Binding — Instruction files:** `AGENTS.md` is canonical; `CLAUDE.md` is the one-line
  `@AGENTS.md` adapter. Generated content changes at its source, via `repo-bootstrap`.
- **Default — Local guidance:** smallest useful layout outside generated blocks.
- **Binding — Graphify:** for adopters, PRs never change `graphify-out/`; regeneration and
  committed cleanup belong to the existing main-side job.

## Tripwires

Read the applicable source; its rule labels govern. These pointers add no waiver path.

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
