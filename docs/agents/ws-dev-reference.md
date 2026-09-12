<!-- ws:dev-reference source=wrightstrategy/bridge:conventions/dev-reference.md version=1 sha256=e0afdaa533b884ecba9dbc10e2014ce4e64971b19303615b51498b25eaa309db -->
This file is generator-owned by ws-dev repo-bootstrap; do not hand-edit it.

# Coding Universe — reference

**Authority: Binding.**

This document is the **generated/delivered reference payload** for Wright Strategy development
conventions (ADR-016). It is **not** always-resident context: agents and humans **read it on
demand** when a resident tripwire in `conventions/dev.md` (stamped into each repo's `AGENTS.md`
as the `ws:dev` block, and delivered here as `docs/agents/ws-dev-reference.md`) gates a task.

Content below was relocated from the former monolithic `conventions/dev.md`. Prefer the anchors
linked from resident tripwires.

## Standard authority and local departures

**Authority: Binding.**

**Binding** rules cannot be weakened locally; their existing exceptions stay in their owning
policy. **Default** choices apply unless a visible local departure replaces them. A section's
Authority label governs normative prose until the next same-or-higher heading; a labeled
subsection or clause overrides it only within that scope. Examples add no obligations and
MAY/SHOULD remain options/recommendations. Unclassified text retains existing applicability;
never assume unmarked means Default. This does not alter host instructions or user authority.

Use one optional `## Local departures` section in root `AGENTS.md`, outside generated blocks.
Each entry names the Default, scope, replacement and reason, with a canonical source link:

```markdown
## Local departures

- **Default:** [Testing](https://github.com/wrightstrategy/bridge/blob/main/conventions/dev.md#standing-rules), Testing clause.
  **Scope:** Existing Python hook/wiring checks.
  **Replacement:** Keep unittest and standalone runners; the doc-audit package keeps pytest.
  **Reason:** Preserve working stdlib tests without a framework-only migration.
```

Index subtree departures here with their scope and optional nested-guidance link. No section
is needed when there are none. Use the normal repo PR process; Default replacements require
no extra platform approval or issue. Proposed departures remain proposals for human review,
not a new governing frame. Existing valid exceptions can migrate when touched.

Preserve Binding obligations, required checks and actual consumer interfaces. Changing a
Default does not permit hand-editing generated payloads, bypassing a review gate, or breaking
a deployment that relies on the original behavior. Align repo-owned scripts/configuration/docs
when behavior changes; generator changes follow their existing platform ownership path.

The freshness audit displays these declarations and source locations, including current and
existing-update outcomes, without changing freshness or check results. New sync PR bodies
include them. Primary agents include applicable committed and proposed departures in review
evidence. Reviewers cite Binding rules as Binding and assess Default replacements on their
merits; a displaced preference alone cannot block them. The semantic doc-audit verifies the
claims against behavior. Full authority:
[bridge's standard authority](https://github.com/wrightstrategy/bridge/blob/main/docs/standards/standard-authority.md).

## Parallel and isolated work

**Authority: Binding.**

**Default — Ordinary work:** implementation is written in-session by the agent holding the
conversation.

**Binding — Orchestration and roles:** plans/specs remain primary-authored. When a task
genuinely earns parallelism or isolation — independent slices that can run concurrently, a
long batch, or work that must not touch the session checkout — run it through **Orca
orchestration** inside a ready runtime (`orca status --json`): visible `worker-start` /
`check --wait`, one worktree per worker, results verified by the session agent with real
commands, never trusted from a worker's self-report. Orca's shipped skills carry the
invocation mechanics; this payload does not duplicate them. Workers never dispatch onward.
However the bytes were produced, they meet the same cross-family review gate
(`ws-dev:pr-review`) before the human merge gate.

### Worker release before teardown

**Authority: Binding.**

The installed, version-matched Orca orchestration skill (`orca skills get orchestration`) and
command help are authoritative for lifecycle command spellings. After every accepted
`worker_done`, succeeded or failed, the coordinator settles the terminal's next owner before
acknowledging another delivery, waiting again, or ending the turn. Choose exactly one:

```text
orca orchestration worker-start --task <next-task-id> --terminal <handle> --json
orca orchestration worker-retain --dispatch <dispatch-id> --json
orca orchestration worker-release --dispatch <dispatch-id> --json
```

Reuse transfers the exact terminal to an immediate follow-up dispatch. Retention records an
operator-requested exception; it is not an implicit default. Otherwise release the worker.
Never keep a completed worker live merely to inspect its output: release preserves the archive,
which remains readable with `orca orchestration worker-read --dispatch <dispatch-id> --json`.

Release is post-completion settlement, not cancellation. Never release in reaction to a timeout,
heartbeat, status, question, escalation, TUI-idle state, or a stale or rejected `worker_done`.
For `release_pending` or `release_unknown`, follow the receipt's recovery action; never substitute
`orca terminal close`. Use `worker-stop` or `worker-abandon` only as conditional recovery when
`worker-show` proves a failed, stopped, or unknown dispatch, and do not treat an unknown state as
disposable without operator confirmation. Never run `orca orchestration reset` while any dispatch
is active.

A worktree or terminal that owns an active dispatch is never removed: reuse, retain, or release
the worker first, then perform any teardown. Worktree and branch removal remain human actions;
nothing automates those destructive steps. When a terminal predates its dispatch — for example,
the agent terminal launched by `orca worktree create --agent` — `worker-release` returns
`retained` with reason `external_terminal` and takes no process action. The release still settles
the dispatch and remains required; the operator can then remove the worktree.

The [Orca worktree checkpoints](#orca-worktree-checkpoints) govern card state: release the worker
before the card is completed or the worktree is removed, while `completed` itself remains reserved
for after merge.


## Orca worktree checkpoints

**Authority: Default.**

The installed, version-matched Orca CLI (`orca skills get orca-cli` and
`orca worktree --help`) is authoritative for command and status spellings; the examples below
show the supported surface when this convention was recorded.

Create an Orca worktree from its Linear issue so the suggested branch, issue context, and links
carry into the workspace:

```text
orca worktree create --linear-issue <ID-or-URL> ...
```

Link an existing worktree when necessary:

```text
orca worktree set --worktree active --linear-issue <ID> --json
```

At each meaningful transition — implementation complete, checks passing, PR opened, waiting on
review, blocked, and done — update both the worktree comment and board card status. Read the
current comment first and preserve operator-written context by appending or amending it; never
clobber it:

```text
orca worktree current --json
orca worktree set --worktree active --comment "<short current text>" --workspace-status <status> --json
```

Use `in-progress` while implementing, `in-review` once the PR is open and awaiting review, and
`completed` only after merge. Keep comments short and current; a blocked comment says what is
blocked and on whom. Orca metadata updates are best-effort and never block the work itself.


## Repository documentation — full narrative

**Authority: Binding.**

**Every repository must document the system it owns.** At minimum, its `README.md` explains the
repository's purpose, setup, and primary usage, and its `AGENTS.md` maps agents to the current
sources of truth. When behavior, interfaces, configuration, architecture, schemas, or operations
are non-trivial, maintain the corresponding living/reference documentation; do not leave that
truth only in code, issue threads, or historical plans.

- Documentation is part of the change. Update affected living/reference docs in the same change
  that changes the implementation.
- Preserve historical documents as history. The ADR mechanism is retired (documentation
  standard §6): every `docs/adr/` directory is a frozen archive — never write, supersede, or
  re-status an ADR. A decision lands as an edit to the owning living document plus a dated
  entry in the repo's `docs/decisions.md`, in the same PR that implements it.
- Before finishing implementation work, invoke **`ws-dev:doc-audit`**. The skill contains the
  organization documentation standard and semantic reconciliation workflow. Update the affected
  docs, or report `docs not needed: <specific reason>` when the change genuinely leaves documented
  behavior and interfaces unchanged.

This is a local agent responsibility, not a documentation-specific CI or credentials requirement.


## Work management — leaving the graph honest

**Authority: Binding.**

Work filing and planning live in **Linear** (ADR-029; rulebook:
`wrightstrategy/bridge` → `docs/standards/work-management.md`). There is no planning agent:
under One Front Door the plan is *authored* at a planning session, not inferred from metadata,
so the committed projects in target-date order are the plan and Linear's own views are the
plate. The Chief-of-Staff snapshot layer that used to compute one is retired (ADR-030).

Keeping work state honest is still part of finishing the work, not overhead — it is what the
operator and every future reader actually read: link PRs to their Linear issue so state syncs
on merge, assign the carrier, close shipped work with a one-line outcome (never a session log —
point-in-time narrative later contradicts the graph), and record real blocking relations, never
invented ones. Issue **state** has one writable home — Linear — so don't restate it in markdown.

The GitHub-era planning vocabulary — milestones, the label allowlist, the plate-rank formula
(bridge#485) — is retired outright. Timing lives only on projects; issues carry no milestone
or horizon.


## GitHub Actions

**Authority: Binding.**

- Pin every third-party Action and reusable workflow by **full commit SHA**. Keep the human
  version in a comment if helpful (for example, `# docker/build-push-action v7.2.0`) and let
  Dependabot/Renovate raise update PRs.
- Set `permissions: { contents: read }` at workflow or job scope, then grant only the extra
  scopes a job needs (`packages: write`, `id-token: write`, `attestations: write`, etc.).
- **Default — Credential choice:** prefer OIDC or GitHub App tokens over long-lived credentials.
- **Binding — Build secrets:** never pass secrets through Docker build args; use secret mounts.
- Treat `pull_request_target` as privileged: do not check out or execute untrusted PR code in
  that context.
- **Default — Node runtime maintenance:** GitHub periodically forces JS actions onto a
  newer Node major (e.g. Node 20 → 24, forced 2026-06-16) and emits deprecation warnings before.
  Get ahead of it: keep Renovate raising action-bump PRs, and to validate/opt-in early set
  `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true` (or the then-current equivalent) as a workflow-level
  `env:`. Drop the override once every pinned action is native to the new runtime.
- When a release/automation token must reach an **org-owned** repo, use a **GitHub App
  installation token** (`actions/create-github-app-token` with the org App's client ID/private-key),
  not a fine-grained PAT — a PAT scoped to a personal account 404s on org repos, and App tokens
  also trigger downstream workflows (unlike the default `GITHUB_TOKEN`).

### Concurrency and merge queues

**Authority: Binding.**

A concurrency group still runs at most one job or workflow at a time. How many runs may *wait*
behind that one is configurable with `queue`:

- `single` (default): at most one pending run. A third arrival cancels and replaces the pending
  run even when `cancel-in-progress` is `false`. The cancellation flag protects the running run;
  it does not make a shared group a durable queue.
- `max`: up to 100 pending runs, processed FIFO by the time each started waiting — not by
  dispatch time or commit order. When the queue is full, further arrivals are canceled. GitHub
  still does not guarantee that an older commit lands first. `queue: max` cannot be combined with
  `cancel-in-progress: true`; that combination is a workflow validation error.

Keep the one-run-per-change rule: required checks run on the PR head, and the default-branch push
does not repeat those same build/test/scan jobs. For workflows that produce an artifact from each
shipped commit, combine a per-ref PR group with cancellation and a per-SHA default-branch group:

```yaml
concurrency:
  group: <name>-${{ github.event_name == 'pull_request' && github.ref || github.sha }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
```

A new PR push then supersedes the older run. Every default-branch commit instead has a unique
group, so its run can be neither cancelled while running nor displaced while pending. This matters
when graphify refresh, a release-PR tool such as the deprecated release-please, or a docs/lockfile
bot pushes an automation commit just after a merge; a saturated runner pool makes it likely that the merge build is still pending when the
automation run arrives. The observed result was a release tag pointing at a commit for which no
image had been built. Unique per-SHA groups do not need `queue: max`; PR cancellation cannot
combine with it.

Default-branch deploy workflows need a different concurrency choice. A per-SHA group combined with
a commit-pinned deploy lets runs overlap — and with two ready deploy slots, actually execute in
parallel — and finish out of order, so the target can settle on an older commit. Do not use that
combination.

Serialize the apply with a constant group keyed by the mutation target, keep the backlog, and
resolve the tip only after the run enters that group:

```yaml
concurrency:
  group: deploy-<target>
  queue: max
  cancel-in-progress: false
```

Those are three separate properties, not one knob:

- **Serialization.** The group runs one apply at a time. Put that in the workflow; do not fake it
  with a one-runner pool.
- **Queue retention.** `queue: max` keeps intermediate deploys. `queue: single` still drops the
  older pending run.
- **Tip-at-execution.** Checkout and apply the default branch's tip after entering the group
  (`ref: ${{ github.event_name == 'push' && 'main' || github.ref }}` is the Homelab form;
  `workflow_dispatch` stays pinned to the selected ref). Queue admission order is not commit
  order; a commit-pinned run can still land an older SHA last.

`wrightstrategy/homelab`'s `deploy-compose.yml` and `deploy-configs.yml` (homelab#1891) are the
worked example: two ready deploy slots, one constant `queue: max` group per target, tip resolved
inside that group.

When a repository enables a merge queue, every workflow that produces a required status context
must also run for queue entries:

```yaml
on:
  pull_request:
  merge_group: { types: [checks_requested] }
```

Otherwise the required context never arrives and the queue cannot complete the entry. Queue runs
must not supersede one another: cancellation of a required check fails that entry and makes the
queue thrash instead of drain. Gate cancellation to `pull_request` and keep the group unique per
entry; `github.ref` is unique for each `merge_group` entry (and `github.sha` in the general recipe
above is unique too). Do not key the group only with `github.event.pull_request.number`: that value
is empty for `merge_group`, collapsing every queue entry into one shared group.

### Repository settings

**Authority: Binding.**

- **Default — Branch cleanup:** delete head branch on merge is on for every org repo (`delete_branch_on_merge: true`).
  Merged PR branches must not accumulate. `repo-create` enables it on every new repo; existing
  repos must match. Toggle: `gh repo edit <owner>/<repo> --delete-branch-on-merge`.
- **Human repository access** follows the repository access standard
  (`wrightstrategy/bridge` → `docs/standards/repository-access.md`): any repo that ships a
  marketplace-distributed plugin must be readable by every human in `roster.toml`; other private
  repos default to invite-only (the org-wide `default_repository_permission: none` remains
  binding). Agents propose
  grants; owners apply them.

### Actions static analysis (zizmor)

**Authority: Binding.**

- Every registered **private** app repo runs **zizmor** on `.github/workflows/` (generated
  workflow + policy via repo-bootstrap `--payloads` / the central audit fan-out).
- Policy encodes the first-party moving-ref rule (same as prose above):
  `wrightstrategy/bridge/*` and `wrightstrategy/.github/*` → **ref-pin** (`@vX` for CI standards,
  `@main` for the org caller stub); everything else → **hash-pin**. Default
  zizmor (blanket hash-pin) fails these moving first-party refs — never ship the linter without
  this policy (bridge#145, quiltshowcase#388).
- Payload exclusions are **per-payload**: homelab is excluded from the generated **Renovate**
  payload because it owns that configuration locally, but still receives the **zizmor** payload
  like every other registered private repo
  so pin/checksum bumps in the generator fan out (bridge#262).

### Runner placement & the minute budget

**Authority: Default.**

GitHub-hosted minutes are a **fixed monthly budget** (the org allowance, currently 3,000 Linux
min), not a free resource — and GitHub **bills every job rounded up to a whole minute**, so many
short, parallel, or scheduled jobs add up far faster than wall-clock suggests. **Self-hosted
runners are unlimited and free.** Spend the budget deliberately.
- **The isolated CI VM pool** (`runs-on: [self-hosted, ci]`) serves tests, lint, SAST, secret
  scans, service containers, PR image builds, trusted branch builds, releases, and scheduled
  maintenance. Each job gets a fresh Ubuntu 24.04 VM with local Docker, normal passwordless
  `sudo`, and public internet access, then the VM is destroyed. It has no route to the LAN,
  WireGuard, k3s, Nimitz, or 1Password and no standing credentials. `setup-*` actions,
  `apt-get`, Docker actions, and `services:` work exactly as on `ubuntu-latest`; do not add
  runner-specific workarounds or maintain a custom CI image. The shared release workflow
  already selects the CI pool.
- **Binding — Deployment runners:** jobs that must reach the homelab LAN or QuiltShowcase over WireGuard
  use the `Homelab-deploy` runner group plus the appropriate label (`homelab` or
  `quiltshowcase`). Do not target a deployment runner by label alone — an unrelated runner can
  share that label. Never store deploy credentials in the VM image or runner configuration;
  inject them from GitHub secrets for the individual job.
- **GitHub-hosted is the explicit exception:** untrusted/fork PR code from public repositories,
  the organization-wide agent workflow, external deadman monitoring that must survive Nimitz
  failure, and operating-system matrices that require GitHub macOS/Windows images. Document the
  reason beside every remaining `ubuntu-*`, `macos-*`, or `windows-*` assignment.
- **Binding — Public isolation:** public repositories must not use the `Homelab-runners` group.
- **Binding — Runner ownership:** the Nimitz VM definitions, network isolation,
  registration-token minter, lifecycle service, monitoring, and operations runbook live in
  `wrightstrategy/homelab`. Application repos select a trust tier; they do not manage runner hosts.
- **Default — Cost control:** path-gate expensive non-required jobs (image builds, e2e).
- **Binding — Required checks:** never path-gate a required status check: a skipped required check hangs the PR at "Expected". Gate
  the job with a computed `needs.<detector>.outputs` condition instead of a top-level `paths:`
  filter when the workflow also hosts a required check.
- **Scheduled workflows** run on self-hosted, at the cadence actually needed (a liveness probe
  every few hours, not every few minutes).
- **A minute spike during heavy development is a smell, not a cost of doing business.** It means
  the pipeline does too much per change (redundant jobs, no `cancel-in-progress`, an unbounded
  matrix, an image rebuilt on both PR and merge) — fix the pipeline, don't absorb the overage.
  When a month trends over, read per-repo usage with the **parameterized** billing endpoint —
  `GET /organizations/<org>/settings/billing/usage?year=<Y>&month=<M>` — and attack the top repo
  first. Three traps (observed 2026-07, `wrightstrategy/bridge#118`): **(1)** the
  unparameterized endpoint returns coarse rollup rows whose `repositoryName` is an arbitrary
  stamp, not real attribution — always pass `year`/`month` (observed behavior, not a documented
  GitHub contract; re-verify against the rollup if numbers look off); **(2)** filter
  `unitType == "Minutes"` — `product == "actions"` also carries `GigabyteHours` storage rows
  that corrupt a minutes sum; **(3)** don't filter to `sku == "Actions Linux"` — Windows/macOS
  SKUs bill at 2×/10× multipliers and silently vanish from a Linux-only view. The 3,000-minute
  allowance is applied as a **monetary discount**, so summed `netAmount > 0` — not a minutes
  comparison — is the exact "we are being charged" signal.


## Versioning & releases

**Authority: Default.**

**Standard by language.** Python repos release with **python-semantic-release** in tag-only mode,
run from a dispatched workflow. JS/TS repos release with **Changesets**. **release-please is
deprecated:** no new adoption; a repo still on it migrates the next time its release tooling is
touched. Known remaining consumers are bridge's own path-scoped `ci/` driver and the reusable
`release.yml` workflow it publishes; they follow the same rule. Design record and pilot:
`wrightstrategy/quiltshowcase` `docs/project/release-operating-model-spec.md` (decisions D8, D14,
D20, D21); QuiltShowcase piloted the model before it became the convention.

Common to every tool, with each obligation labeled:
- **Default — Version source:** the version lives in one place (`pyproject.toml` / `package.json`) and code reads it
  dynamically — `importlib.metadata.version(...)` in Python, an import from `package.json` in
  JS/TS — never a hardcoded version literal in source or tests, or the first bump breaks the build.
- **Binding — Release tags:** nothing tags by hand. The release workflow is the only thing that creates a `v*` tag; a
  hand-pushed tag bypasses the notes, the digest append, and the readiness check.
- **Binding — Image identity:** the image lane is unchanged: `container-build` release mode resolves the `sha-<full>` digest the
  branch build already pushed for the tagged commit and promotes `:X.Y.Z` / `:X.Y`. Build once, retag.

### Python: python-semantic-release, tag-only, from a dispatched workflow

**Authority: Default.**

- **Why this shape.** Tag-only mode tags a commit the branch already built and validated, so
  build-once-retag needs no release commit. There is no release PR, so no CI round per refresh
  (QuiltShowcase measured 15.5% of its CI runs going to release-PR refreshes before it batched them).
  Config is Python-native, a `[tool.semantic_release]` table in `pyproject.toml`. `remote.type`
  supports `github` and `gitea`, so the setup survives a Forgejo move with one value changed.
  Versions are computed from the tags reachable in the branch's own history, which is exactly what a
  `release/X.Y` maintenance lane needs. Explicit forcing flags (`--minor` / `--major`) cover the
  numbering edge case below.
- **The workflow.** `workflow_dispatch` on `main` or a `release/X.Y` branch with inputs `dry_run`
  (default **true**: print the next version and the notes, change nothing — an accidental click does
  nothing), `bump` (`auto` | `minor` | `major`), and an optional `prerelease` token. A real cut
  computes the version from Conventional Commits since the last tag in the branch's history, tags the
  branch tip without adding a commit, pushes the tag, and creates the GitHub Release with generated
  notes. The existing release lane then runs unchanged.
- **Binding — Constraints when using this release lane:** pin python-semantic-release by exact version in the workflow (for example
  `uvx python-semantic-release==X.Y.Z`); a floating version is a supply-chain and behavior drift.
  Push the tag with the org bot **App token**, never `GITHUB_TOKEN`: a tag pushed with the default
  token does not trigger the downstream release workflow (the OPS-1 lesson).
- **Config (`pyproject.toml`):** tag format `v{version}`; the conventional-commit parser; no version
  files (see the dynamic-read rule above); branch patterns for `main` and `release/.*`;
  `remote.type = "github"`. `CHANGELOG.md` is retired — the Release body is the changelog, and the
  committed file was what forced the release commit. **The tool's defaults do not produce the
  version consequences below; set them explicitly.** The conventional parser's default
  `patch_tags` is `["fix", "perf"]`, and the default changelog excludes nothing, so the table below
  requires `[tool.semantic_release.commit_parser_options]` with `minor_tags = ["feat"]` and
  `patch_tags = ["fix"]`, and a release-notes **template** that omits the non-releasing types from
  the grouped change sections — by template logic or by `changelog.exclude_commit_patterns`, either
  is fine. (Until 2026-09 this section forbade the latter, because dropping a `chore:` commit would
  also have dropped any `Deploy:` trailer it carried. Deploy steps are a file now — see § Deploy
  steps — so that hazard is gone.) Verified against python-semantic-release 10.6.2; re-check the
  parser defaults when bumping the pin.
- **Version consequence.** `fix:` → patch, `feat:` → minor, `!` or a `BREAKING CHANGE:` footer →
  major. `build`, `chore`, `ci`, `docs`, `perf`, `refactor`, `style`, and `test` neither release nor
  appear in the notes (`perf` is deliberately not a release: nothing a customer sees changed). `fix:`
  means a defect in *released* behavior; corrections to a feature that has not shipped stay inside
  that feature's own commits, or every release's notes are padded with fixes to things no customer
  saw. `!` is reserved for operator-breaking changes (a required new env var, a manual migration
  step, a changed deploy order) and the body says what the deployer does.
- **The one numbering edge case.** A maintenance line has cut `2.1.1` while `main` has seen only
  `fix:` commits since `2.1.0`, so `main`'s derived next version is also `2.1.1`. The dry run shows
  it. Rule: while a maintenance line is open, `main`'s next release is at least the next minor —
  pass `bump: minor`.

### JS/TS: Changesets

**Authority: Default.**

- Each PR that changes published behavior adds a changeset file; the Changesets action maintains the
  version PR and, on merge, tags and publishes. The version lives in `package.json` only, read
  dynamically. The release lane (build once on `main`, retag on `v*`) is the same as for Python.

### Maintenance-branch lane: `release/X.Y`

**Authority: Default.**

- **Create lazily, from the production pin.** `release/X.Y` exists only when production needs a fix
  before the next minor. Read what production runs (the deployed image pin names the version and
  digest), then create the branch from that tag. Never pre-create: staging is evaluation, not a
  promise to promote, and production may sit on 2.1 while `main` reaches 2.5.
- **Fix there first.** Branch from `release/X.Y`, commit as `fix(scope): …`, PR into `release/X.Y`
  under the same gate as `main`. Merge, then cut from the maintenance branch (`Cut release` on
  `release/X.Y`); staging deploys it; run the promotion check; pin it in production. The fix ships
  alone — `main`'s unfinished work cannot follow it.
- **Forward-port in the same session.** Cherry-pick the fix commit onto a branch from `main` and open
  a PR. Never merge in the other direction. A hotfix migration keeps its file name so `main`'s
  migration-leaf check forces the merge migration. If a newer cut is waiting on staging and you still
  intend to ship it, cherry-pick onto its `release/X.Y` too and cut a patch; if you intend to skip it,
  do nothing. This is the step people forget, which is why it is a rule.
- **What the release lane needs, per repo.** `container-build` release mode is already
  branch-agnostic — it resolves `sha-<full>` for the tagged commit regardless of which branch built
  it — and the reusable workflow does not change for this. The repo-owned pieces: run the branch-mode
  image build on `release/**` applying only the mandatory commit lookup tag (no `:edge` / `:next`, no
  deploy); extend the PR-and-required-checks ruleset to `refs/heads/release/*` with branch creation
  exempt from required checks so a branch can be created from a tag; add `release/**` to any push
  triggers that gate `main` (migration-leaf, zizmor); and make the repo's readiness check accept a
  build from `main` **or** from the `release/*` branch that contains the tagged commit.
- **Binding — Consumer pins:** stop promoting `:latest` once a repo can have two release lines: it can move backward, and
  nothing should consume it — consumers pin a digest.
- **Binding — Rollback identity:** pin an older released tag and digest. It never uses a branch; images and tags are
  permanent. **Cleanup:** a `release/X.Y` older than the production pin is flagged for a human
  (planning brief), never auto-deleted; deletion loses nothing because the tags keep every commit.
- Repo-local procedures live in QuiltShowcase's `release-ops` skill (`cut` with dry run first,
  `hotfix start`, `hotfix forward-port`, `promote-check`, `branches`). It lifts into the ws-dev plugin
  when a second repo needs it (earn-its-place); until then, copy the procedure, not the code.


## Deploy steps

**Authority: Binding.**

**The record is a file, not commit history.** Anything a change needs outside the image — a new,
renamed, or removed environment variable; a migration that needs ordering or a hand step; a
scheduled job; an ordered cutover; an out-of-app step such as a DNS record, a webhook subscription,
or a provider dashboard setting — is an entry in `deploy/deploy_next.md`, added in the same PR as
the change that needs it.

The org used a `Deploy:` commit trailer until 2026-09. It failed for a structural reason, not a
QuiltShowcase-specific one: **git history is append-only.** When a later commit changed what an
earlier one had asked for, the result was two trailers of equal authority with nothing encoding
which superseded which — and no place to record that a step had actually been run on a tier.
QuiltShowcase's sweep of 182 commits found a trailer naming a command that no longer existed and
two pairs that contradicted each other. A file in the working tree is mutable, so a correction is
an edit and the contradiction cannot exist. That property is why this generalizes to every repo,
including repos far smaller than the one that found it.

### Writing an entry

**Authority: Binding.**

- **Read the pending entries before adding one.** If your change alters or cancels an entry already
  there, **edit that entry in place** and say what changed. Never leave two entries that disagree —
  that is the exact failure this convention exists to prevent.
- **Automatic work is not an entry.** Whatever the container or the deploy already does on every
  tier — migrations, schedule sync, health-check provisioning — needs no entry. An entry is for
  what a person or a separate system must do.
- **Names, never values.** Keys, hostnames, and command names belong in an entry; secrets and
  per-tier values do not (resident **Binding — Secrets**).
- **Default — Entry shape:** QuiltShowcase's example:

  ```
  ## Short title (ISSUE-nnn, PR #nnn)
  - When: before the image goes live | with the image | after the image is live
  - Tiers: next, staging, production
  - Do: the exact command or the exact manual step
  - Verify: the command that proves it, or "manual"
  - Why: one sentence
  ```

### Making it stick: gate on the paths that imply a step

**Authority: Default.**

Documentation does not enforce this; CI does. Name the handful of paths in your repo that almost
always imply an out-of-app step — the env-var example file, the webhook handler table, the feature
switch registry, the container entrypoint — and fail a PR that touches one of them without touching
`deploy/deploy_next.md`. Keep the workflow paths-filtered so it is skipped everywhere else.
QuiltShowcase's `deploy-steps.yml` is the worked example. A repo whose out-of-app steps are rare
enough that no path predicts them does not need the gate; it still keeps the file.

The authoring side needs no skill: the entry shape above plus this gate is the whole rule.

### Rotation: file on the release, never on completion

**Authority: Binding.**

The pending file rotates when the code **ships**, not when its steps are finished.

- **A repo that cuts releases** files the pending file as `deploy/deploy_vX.Y.Z.md` during the cut,
  then starts a fresh empty pending file. The version number is the index: a promotion reads every
  versioned file between the tier's current pin and the candidate, in order.
- **A single-tier repo that deploys straight from `main`** rotates on the deploy instead, by date or
  deploy number. Same rule, different name.

Rotating on "the steps are all done" is wrong twice over. *Done* is per tier — done on staging is
not done on production, so staging finishing a step would rename the file out from under the
production promotion that still needs to read it. And a file named for its completion has no
orderable identity, so nothing can compute which files a promotion still has to cross.

**Default — Completion notation:** record completion inside the file as
`Done: <tier> <date> (<who>)` on the entry.

**Binding — Filed record integrity:** a filed version file is otherwise edited only to strike
an entry that a later version withdrew, naming that version. Rollback reads the files backwards.

### What scales with tiers, and what does not

**Authority: Binding.**

The file and edit-in-place are Binding; the entry shape is Default. Versioned files at the
cut, `Done:` marks, and the cross-version promotion read are machinery for *promotion*: they earn
their place when a repo has more than one tier or a release cut, and not before. A single-tier repo
with neither keeps one `deploy/deploy_next.md`, rotates it on deploy, and stops there. Do not
install a promotion model in a repo that has nothing to promote.

### Release notes

**Authority: Binding.**

- **Release notes lead with a "Deployment changes" section built from the entries filed at the
  cut**, then changes grouped by type and scope, and the release lane appends the image index digest
  to the Release body. This is what makes the resident "a deployer bumps a pin without reading
  source" rule mechanical: a deployer reads the top of the Release and the digest at the bottom,
  nothing else. The section is the contract, not the template — a repo on another release tool
  renders the same section by whatever means it has.
- The tagged tree still holds the same entries under the pending name, so **a tag alone is a
  complete record** even before the filed copy is read.
- **Default — Promotion diagnostics:** a promotion check may diff environment key names across
  tiers and print the filed entries between the production pin and candidate. QuiltShowcase's
  `promote-check` is the worked example.
- **Binding — Diagnostic secrecy:** names only; values never leave their tier.

### The executing side stays repo-local

**Authority: Default.**

Reading the files a promotion crosses, running the steps in order, and writing the `Done:` marks is
a procedure with real branching, and it is skill-shaped — QuiltShowcase carries it as its repo-local
`release-ops` skill.

**Binding — Shared skill ownership:** it is deliberately not a `ws-dev:` skill yet: that
procedure is the most repo-specific part of this convention (image pins, tier names, that repo's own workflows), and
promotion into the shared plugin is gated on a demonstrated second consumer (earn-its-place).
Revisit when a second repo has built one.

### Repos still carrying `Deploy:` trailers

**Authority: Binding.**

Nothing here is retroactive and no migration is required. Repos still using the trailer keep
working under this existing migration exception; the file is required for new work. `ci/scripts/homelab-bump.ts` still emits a trailer on
the automated homelab pin bump — that is homelab's call to change, not a bridge edit.

## Container images & CI

**Authority: Binding.**

Registry: `ghcr.io/wrightstrategy/<repo>`. Builds run in GitHub Actions. The cluster is amd64.
These build/release standards are encoded as **versioned reusable workflows** in
`wrightstrategy/bridge` (`container-build`, `release`, `osv-scan`, `lockfile-refresh`, and
`homelab-bump`, introduced in v2.1.0). Their
canonical sources live under `ci/` and are generated into the root workflow namespace consumed as
`wrightstrategy/bridge/.github/workflows/<name>.yml@vX`.

**Default — Workflow choice:** consume the shared workflow rather than re-implementing it per
repo. Its published interfaces remain Binding when consumed.

**Binding — Automatic pin update contract:**
The `homelab-bump` workflow automatically commits enrolled image pins after a successful
stable release. Homelab owns its target mapping and activation policy; app callers supply
the release tag and App configuration. Its contract is `wrightstrategy/bridge` →
`ci/docs/homelab-bump.md`. Pin the published workflow commit SHA; it is consumed directly
from git, not from the separately downloadable ws-dev plugin archive.
- **Default — Image construction:** multi-stage builds, language-native locked dependencies,
  `EXPOSE` the service port, tight `.dockerignore`, and the uv Docker example's cache/layering
  recipes for Python.
- **Binding — Image security and deployment contract:** digest-pin the base image, run as a
  non-root user, bundle owned migration tooling/scripts, and wire required platform probes.
- **Default — Platforms:** `linux/amd64` by default; add `linux/arm64` only when a target host needs it
  (one-line `platforms:` change).
- **Default — Image metadata:** stamp images with `docker/metadata-action` (`org.opencontainers.image.*` labels).
- **Build once, then promote.** *Branch builds create releasable artifacts; release builds only
  promote them.* An image is built, scanned, attested, and pushed **exactly once** — on the `main`
  push — and a `v*` tag **relabels that same digest** (no rebuild), so the released image is
  byte-identical to what was tested and deployed, and re-running a tag is idempotent.
- **Trigger matrix:**
  - PR → test gate (lint + tests) **and** build the image, but **never push** (validates the
    Dockerfile early).
  - push to `main` → after the gate, build + scan + push **by digest** (provenance + SBOM), then
    promote a **mandatory commit lookup tag** (`sha-<full-commit-sha>`) + a rolling `:edge`. This
    is the **only** place an image is built.
  - `v*` tag (from the repo's release workflow — see Versioning & releases — never by hand) →
    **retag-only**: resolve the digest the branch build pushed for the tagged commit, promote
    `:X.Y.Z` / `:X.Y` (and `:latest` only while the repo has a single release line) for a stable
    release (only `:X.Y.Z-<pre>` for a prerelease), and
    **append the `sha256:` index digest to the GitHub Release body**. No rebuild, no re-scan. A
    `v*` tag **must** point at a commit a branch build (`main` or `release/**`) already built —
    releasing a never-built commit is a hard error (so tag-only repos add a default-branch build).
- **Binding — Build verification:** test-gate every build; never publish a release whose image
  build failed.
- **Default — Test dependencies:** use real integration dependencies where cheap, rather than
  drift-prone mocks.


## Supply-chain integrity

**Authority: Binding.**

Required for every released image, and works regardless of repo visibility:
- **OCI provenance + SBOM attestations** via `docker/build-push-action` (`provenance: mode=max`,
  `sbom: true`) — BuildKit stores them with the image in the registry; no GitHub-plan
  dependency. Because `mode=max` can expose build-arg values, keep secrets out of build args.
- **Vulnerability scan gate** (`aquasecurity/trivy-action`) on the built image; fail on
  **fixable** `CRITICAL,HIGH` (`ignore-unfixed: true`). No-fix-available base-image OS CVEs
  aren't actionable and would otherwise block every release as new ones are disclosed — gate on
  what you can actually remediate. Allowlist a specific fixable CVE only with written justification.
- **Secret scanning** — every repo, regardless of visibility, runs **gitleaks** in two places:
  a **pre-commit hook** (blocks a secret at commit time — the earliest catch and the free analog
  to push protection) and a **CI gate** on PRs + the default branch. Use `gitleaks git` mode
  (scans committed **history**, not the working dir) so a developer's gitignored local `.env`
  stays out of scope; pair with branch protection so nothing reaches the default branch without
  the gate. Prefer this OSS path over **GitHub Secret Protection** (GHAS secret-scanning SKU,
  ~$19/active-committer/month on private repos) — don't pay for what gitleaks does for free on a
  small private repo. Run the **pinned** gitleaks binary/container directly (avoid
  `gitleaks/gitleaks-action`, which wants a free-but-annoying org `GITLEAKS_LICENSE`). Optionally
  add a **scheduled** `trufflehog` deep scan for **verified** detection (it calls provider APIs to
  confirm a found credential is actually live). On adoption, run a one-time full-history baseline
  scan first to confirm nothing has already leaked.

**Default — Optional stronger guarantees**, when available (do not block on them):

- **Default — GitHub artifact attestations** (`actions/attest`) for image digests and release artifacts.
  Private/internal repos require GitHub Enterprise Cloud; public repos work on current plans.
- **Default — Keyless image signing** with cosign (Sigstore, OIDC), always signing the **digest**, never
  a tag. Public-good Sigstore works for private repos but records signing identity metadata in
  the public Rekor log; decide per repo whether that is acceptable.
- If signing/attestation is unavailable, the baseline remains OCI provenance + SBOM + Trivy.


## Dependency updates

**Authority: Binding.**

Automated dependency updates are standardized org-wide by ADR-020. The policy is **five layers,
ordered by value not effort**, and Renovate is the sole version-update engine. Every registered
repository carries a thin `renovate.json` generated by `repo-bootstrap`; it extends the shared
`renovate-config/default.json` preset so fleet policy has one inheritable source of truth.

**Manager coverage follows repository contents.** The shared preset enables Bun, npm, PEP 621,
GitHub Actions, Dockerfile, and the fleet's narrow custom-regex manager. Renovate discovers the
applicable managers from each repository rather than requiring generated lockfile-specific entries.
The preset groups routine non-major updates separately from major updates, keeps GitHub Actions
SHA-pinned except for the deliberate first-party moving-ref policy, and authenticates to
`npm.pkg.github.com` through its centrally managed `PACKAGES_READ_TOKEN` host rule. A repository may
add only described `packageRules` and `ignoreDeps` in its generated thin config; the generator
preserves that documented local seam and rejects mutations to the shared core. A `description` may
be a string **or an array of strings**, matching Renovate's own schema — Renovate renders the array
as separate lines in the update PR.

**Carry or refuse — never drop.** Once a `renovate.json` carries the generated core, regeneration
either carries every local key forward or **refuses**, naming the offending field and writing
nothing (`renovate.json: packageRules[1] needs a non-empty description …`). It never removes
repo-owned content silently: a dropped `packageRules` entry is dependency policy deleted inside a
routine audit PR, which is how ChargeAlert briefly lost its security-override exclusion (PLT-350).
Fix the named field and re-run sync; if a repository genuinely needs a key outside the seam, file
PLT rather than hand-editing around the generator. A file that does *not* carry the generated core —
foreign config, or JSON that does not parse — has no recognizable seam to protect, so `sync` adopts
the canonical payload as the documented repair.

**The rollout invariant.** Value order is not activation order. **No automated update ecosystem —
`github-actions` included — activates in a repo until that repo has a stable PR check and that check
is required** (layer 1). The fleet posture is **owner-final** (ratified 2026-08-30): CI is required
and no automation can bypass it, but `enforce_admins` stays false / the org-admin ruleset bypass
stays open — the owner's merge button is the human gate, not a hole in it. Enabling update PRs into an ungated default
branch recreates the incident this policy exists to prevent (a dependency bump merged past no gate);
requiring a check that does not exist or is flaky freezes merges instead. A repo with no CI gets a
check before it gets automation. Layer 1 before layer 4, per repo.

**The five layers** (and what shipped):

1. **Required status checks on the default branch, owner-final** (`enforce_admins: false`; the
   only ruleset bypass is the owning admin — the live fleet posture since ADR-006 slice 3,
   preserved by ADR-007/008 and what the scaffolder creates). Require a context that runs
   unconditionally on `pull_request` — verified statically at source, not from past runs; a
   path-filtered context qualifies only if the union of its lanes covers every PR. *Applied to all
   active repos*; homelab additionally keeps `enforce_admins: true` by its own choice, with cost
   control as a per-step classifier rather than a skipped required check.
2. **Dependabot alerts + dependency graph org-wide**, via a security configuration ("Wright Strategy
   baseline"), enforced and default for new repos. It must **not** enable secret scanning (the paid
   GHAS SKU the org declines in favor of gitleaks) or `dependabot_security_updates` (update
   automation, forbidden before layer 1). *Applied.*
3. **Renovate access to private org packages.** The shared preset's `hostRules` consumes the
   centrally managed **`PACKAGES_READ_TOKEN`** for `npm.pkg.github.com`. *Applied* — and sufficient
   for every lookup, but see [GitHub Packages](#github-packages) before diagnosing a `401`: a Bun
   workspace whose private dependency sits in a nested package file does not get this credential
   into its lockfile install, for reasons upstream of the preset.
4. **Version updates through the shared Renovate preset and generated thin config.**
   `repo-bootstrap` owns `renovate.json`; the preset owns fleet manager coverage, grouping, pinning,
   and central package authentication. Graphify resolves latest stable directly in its refresh
   job and has no dedicated Renovate pin manager. The preset is activated fleet-wide;
   homelab keeps its locally owned
   Renovate configuration and is excluded only from the generated Renovate payload.
5. **`osv-scanner` in CI on the `bun.lock` repos, pinned.** GitHub's dependency graph has no
   resolution path for `bun.lock` (JavaScript is "graph jobs: NO"), so a vulnerability present **only
   transitively** in a bun repo goes unreported; direct-dependency alerts still fire, and uv repos are
   graph-covered and get **no** scanner. The **`osv-scan` reusable workflow** lives in `bridge`
   (`.github/workflows/osv-scan.yml`, consumed `uses: …@vX`); it runs the version-pinned,
   checksum-verified `osv-scanner` binary on self-hosted CI — **not** the vendor reusable workflow,
   which uploads SARIF to code scanning (a GHAS SKU the org declines) and pins `ubuntu-latest`.
   Per-repo rollout is **baseline before enforcement**: scan and report (`fail-on-vuln: false`) →
   triage the backlog → record accepted findings as reviewed, time-bounded `osv-scanner.toml`
   suppressions (`[[IgnoredVulns]]` with `ignoreUntil`) → flip `fail-on-vuln: true` and make the
   check required. `bun audit` is rejected as the primary gate (npm-registry resolution at run time;
   `--ignore` with no expiry or justification trail). *Live consumer set:* web-ui, chargealert,
   sentinel, scuttlebutt (four; ADR-004 named six — proofyard and agent-ops are archived and drop
   from the active set).

Renovate raises update PRs but does not replace vulnerability scanning, so it is not a layer-5
substitute. Dependabot alerts and the dependency graph remain enabled as security features, while
`osv-scanner` covers the documented Bun transitive-dependency gap.

**Check any future mandated toolchain against GitHub's dependency-graph support *empirically, before*
standardizing it** — not from the docs support-table, which a reading got wrong twice. Enable the
graph, pull the repo's SBOM, compare against the lockfile, and check whether known transitives of a
direct dependency appear. Bun and uv were both adopted on ergonomics and both inherited this coverage
question; only a probe of the live graph answers it.


## Web UI — kit dependency mechanics

**Authority: Default.**

Building or modifying a web app? Use the shared UI spine — don't reinvent it.
- Repo: `~/Projects/web-ui` (GitHub: `wrightstrategy/web-ui`). Stack: Bun + SvelteKit + Svelte 5.
- **Binding — Design ownership when consuming the kit:** it is the source of truth for its design standards: `@wrightstrategy/ui` (semantic tokens +
  components), the `create-app` scaffolder, the canonical SvelteKit template, and the design
  canvas (`design/v1.0/` — when canvas and code diverge, the canvas wins).
- Scaffold new apps with its `create-app`; follow its page recipes and AppShell / PageHeader / token
  conventions (see its `skills/` and `docs/`).
- **Binding — Shared promotion:** earn-its-place: don't promote a component into the kit before it has 2-app reuse; app-local
  styles stay app-local.
- **Binding — Package interface when consuming the kit:** the kit is `@wrightstrategy/ui` (renamed from `@wright/ui` in June 2026; GitHub Packages requires
  the npm scope to equal the owning org). How you depend on it is not a preference — it follows from
  where your app lives:
  - **Inside the web-ui workspace:** `workspace:*`.
  - **Editable local dev outside it:** the scaffolder writes a relative `file:` path. This does not
    exist inside a CI runner or Docker build — swap it before your first CI run.
  - **Cross-repo / CI / Docker (the real deployment shape):** `"@wrightstrategy/ui": "^1"` from
    GitHub Packages — it is a **private** package, so see *Consuming private GitHub Packages* below.


<a id="consuming-private-github-packages"></a>

## GitHub Packages

**Authority: Binding.**

`@wrightstrategy/ui` is published **private** to `npm.pkg.github.com` (ADR-003). Any repo moved to
this posture inherits the same rules — and the same trap.

**A token is necessary but never sufficient.** GitHub Packages requires auth to pull *regardless of
package visibility* (unlike `ghcr.io`, which serves public images anonymously). Every path below
needs the committed `.npmrc` scope route — no secret literal, so it is safe to commit:

```ini
@wrightstrategy:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```

Beyond that the consumer paths differ — do not assume the Actions recipe covers them:

| Path | Credential | Also needs |
|---|---|---|
| **Actions job** | `NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}` | job `permissions:` with **both** `contents: read` and `packages: read` (job-level `permissions` zeroes every omitted scope, so listing only `packages: read` breaks `actions/checkout`) |
| **Local dev / Docker** | PAT **(classic)** with `read:packages`, from 1Password | Nothing to declare — there is no workflow permission here. The PAT's **owner** must also have read access to the package; the scope alone still 403s. Fine-grained PATs are **not** accepted by this registry, despite the org's general preference for newer token types. Docker mounts it as a BuildKit secret, never a build-arg |
| **Renovate** | `PACKAGES_READ_TOKEN`, managed in Mend Renovate Cloud | The shared preset's `hostRules` entry for `npm.pkg.github.com`; repositories must not duplicate this in local `renovate.json`. Sufficient for lookups everywhere, but **not** for the lockfile install in a Bun workspace — see below |

**The Renovate row has a second trap, and its symptom is a `401`.** Renovate's Bun artifact updater
appends the `hostRules` credential to the `.npmrc` beside the *changed package file*, then runs
`bun install` beside the *lockfile*. Those are one directory in a single-package repo and the
install is authenticated. In a Bun **workspace** whose private dependency is declared in a nested
package file they differ: the token lands where Bun never looks, Bun falls back to the committed
root `.npmrc`, and `${NODE_AUTH_TOKEN}` expands to nothing outside Actions. The update lands with a
missing or partial `bun.lock` and `error: GET https://npm.pkg.github.com/... - 401`.

This is upstream [renovatebot/renovate#43255](https://github.com/renovatebot/renovate/issues/43255).
An unresolvable Mend secret cannot produce it — Renovate abandons the whole repository run rather
than emitting a PR — but resolution is not validity either, since an expired PAT substitutes
cleanly and then `401`s. The discriminator is **which package file the failing wave changed**, not
when it failed: a nested one is this defect whatever the credential's state, because that install
never receives the credential; a root-only failure does receive it, so that one is a credential
question to test directly. Having placed it, do not duplicate `hostRules` or add a CI job that
rewrites `bun.lock` on Renovate branches; regenerate the lockfile by hand on the affected PR.
Mechanics, blast radius and the interim procedure are in the [shared preset
README](https://github.com/wrightstrategy/bridge/blob/main/renovate-config/README.md#private-github-packages).

**And in every case, for a private package:**
- **The package must grant the consuming repo Read** — package settings → ***Manage Actions access*** →
   *Add Repository* → Read. **This step is UI-only: there is no REST API for it** (every endpoint
   404s), so it cannot be scripted or included in a rollout PR. It is the step people forget, and
   its symptom is a `403` on a job whose config looks perfectly correct.

   > **The trap:** that settings page has **two visually identical sections**, each with its own
   > green *Add Repository* button — **Manage Actions access** and **Manage Codespaces access**.
   > CI needs **Actions**. Filling in Codespaces looks completely correct and changes nothing; the
   > 403 persists with no hint that you granted the wrong thing. We lost time to exactly this.
   > A third section, *Manage access* → "Inherit access from source repository", governs **user and
   > team** access — it is unrelated to Actions, and toggling it does not fix a 403. Leave it on.

**Publishing a repo's package private is therefore a two-actor operation**: the automation can
delete/republish, but a human must add the grants, and consumers are **down between those two
moments**. Sequence deliberately, and grant *before* announcing the change is complete.

**Do not edit anything inside the package directory in the same change that republishes it.**
`README.md` ships in the npm tarball *regardless of the `files` allowlist*, so touching it changes
the tarball's integrity hash — and every consumer's `bun.lock` pins that hash plus a
content-addressed download URL. A changed hash breaks `bun install --frozen-lockfile` everywhere at
once. `bun pm pack` is byte-deterministic, so a same-source republish reproduces the artifact
exactly; verify it rather than assume:
```bash
bun pm pack --destination /tmp/v && \
  echo "sha512-$(openssl dgst -sha512 -binary /tmp/v/*.tgz | openssl base64 -A)"
# must equal the sha512 consumers already pin in bun.lock
```
If it does **not** match, that is a stop signal — roll back or bump the version. Never regenerate
consumer lockfiles under an unchanged version: one version naming two artifacts produces
cache-dependent builds. `web-ui`'s `publish-ui.yml` enforces this with a required
`expect_integrity` dispatch input; copy that pattern rather than trusting an operator to check.

**Public→private is one-way.** A public GitHub Packages package cannot be made private — visibility
is **per-package, not per-version** — so the only path is delete + republish, and **deletion removes
every version at once**.

Before planning on it, inventory what you must restore:
- **Deletion is refused if _any_ version exceeds 5,000 downloads**, and the REST API exposes no
  download count. Confirm the Delete control is actually present in package settings first; its
  absence is the only reliable signal.
- **Republish every version consumers still resolve, not just the newest.** `@wrightstrategy/ui` had
  exactly one version, which made this trivial; that is not the general case. A package with history
  needs each still-referenced version re-published, and each one's integrity re-verified against what
  its consumers pin. If you cannot reproduce them all, do not delete.


## Knowledge graph (graphify)

**Authority: Binding.**

Repos that use **graphify** (`safishamsi/graphify`, CLI `graphify`) build a knowledge graph
at `graphify-out/`. Adoption is **explicit**: a repo is a graphify adopter when it is flagged
`graphify = true` in `registry.toml` (the audit delivers the graphify payload only to marked
repos). `graphify-out/` is a **`main`-owned** artifact — the model and its rationale are
ADR-013 (`docs/adr/013-own-graphify-out-on-main-via-a-post-merge-regen-job-prs-never-modify-it.md`).
The PR-side graph-**staleness** gate is retired (a branch's committed graph is expected to trail
`main`, so it no longer fails PRs). The generated adopter payload completes ADR-013 Phase 2: a
post-merge job installs the latest stable `graphifyy[sql]` distribution, regenerates and commits
`graphify-out/` on `main` (AST-only, free), while required PR CI rejects any diff under
`graphify-out/**`. The job uses `uv tool install --upgrade --prerelease disallow 'graphifyy[sql]'` and logs
`graphify --version`. Upgrades include stable major releases without a Graphify version-bump PR;
the version log is available for the Actions log-retention period, not durable graph provenance.
The SQL extra is part of the canonical runtime contract: without it graphify
classifies SQL but cannot produce the AST hashes required by the manifest integrity gate. The
workflow uses the org bot App's ruleset bypass and is delivered only after a repo is deliberately
marked and its protection is ready.

The PLT-202 capability evaluation does not add routine wiki generation or a persistent MCP
service to the adopted workflow. Use source traversal for configuration that the installed
release does not represent; CLI queries remain the normal graph interface. On-demand wiki
exports and temporary stdio MCP remain available without a fleet service commitment. A
bounded homelab YAML topology experiment is tracked separately in PLT-374; it does not
authorize a maintained fork or expansion of committed graph coverage.

Phase 2 canonical activation is complete: the guidance workflow checks out the PR merge result plus
its base parent (`fetch-depth: 2`) and the pinned checker enforces that adopter PRs do not modify
`graphify-out/**`. Existing adopters receive that canonical workflow on the next central audit
fan-out. For a new adopter with an existing committed graph, land the adopter payload before setting
`graphify = true`; this is an explicit migration and may need an owner-run `--graphify-payload` PR
because central bootstrap cannot select an unmarked repo. Merging that payload triggers the
protected-main refresh, which performs any tracked-artifact cleanup outside the PR. A repository
with no committed graph uses the canonical audit path: establish the ruleset/bypass and confirm the org
Actions secret `BOT_APP_PRIVATE_KEY` covers the repo, merge `graphify = true`, then merge the
generated payload PR. Consumer CI cannot read the central marker and still treats that payload PR
as unadopted; merging it triggers the first main-side refresh, which creates and validates the
initial **structural/AST-only** graph before the bot commits it.
Standardize the committed set the same way in every repo:

- **`.gitignore` — commit the map, drop local-only state.** `graphify-out/` is meant to be
  committed so teammates and agents start from the same map. The generated payload appends an
  explicit allowlist for the five committed artifacts before excluding the per-run API cost,
  semantic-extraction cache, and browser view. That allowlist deliberately overrides an older
  whole-directory `graphify-out/` ignore; without it a zero-graph refresh can generate valid files
  and then silently see nothing to commit. The workflow and central checker fail loud if any
  committed artifact is still ignored.
  ```
  # abbreviated — repo-bootstrap generates the complete ordered block
  !/graphify-out/
  /graphify-out/*
  !/graphify-out/graph.json
  !/graphify-out/manifest.json
  /graphify-out/cost.json
  /graphify-out/cache/
  /graphify-out/graph.html
  ```
  Do not remove a previously committed path in a PR: the generated protected-main refresh removes
  every tracked path outside the canonical five-file set after the payload merges. This keeps the
  cleanup inside the same privileged ownership boundary as regeneration.

- **`.graphifyignore` — merged with `.gitignore`, scopes what gets indexed.** Uses gitignore syntax
  (including `!` negation). Since graphify **0.8.43** (`safishamsi/graphify` #1363, a security fix)
  the two files are **merged**, not either-or: `.gitignore` is evaluated first and `.graphifyignore`
  last, and `.graphifyignore` **can only ever exclude more — it never re-includes** a path
  `.gitignore` already dropped. So it holds the graphify-specific *extras* to skip on top of
  `.gitignore`; you do **not** restate everything. (Older guidance said adding one makes it "take
  over" so it "must list everything" — that was true before 0.8.43 and is now wrong; see
  `docs/design-notes/graphify-best-practices-research.md`.) Index real source + the markdown
  design/architecture docs corpus (specs, living/reference runbooks, the ADR *index*); exclude,
  on top of `.gitignore`:
  - vendored/generated noise — `venv/`, `node_modules/`, build output, collected static, `media/`
  - graphify's own output — `graphify-out/`
  - dead/legacy code that's been superseded
  - stale point-in-time notes (session logs) — they contradict later decisions and poison the graph
  - decided ADR bodies — `docs/adr/[0-9][0-9][0-9]-*.md` (the journal; keep `docs/adr/README.md`)
  - binary assets graphify can't parse — `*.png`/`*.jpg`/`*.svg`/`*.pdf`/`*.xlsx`/`*.docx`/…

- **Keeping it current is the post-merge job's role, not a per-PR step.** Under the `main`-owned
  model you do **not** regenerate and commit `graphify-out/` on a branch to satisfy a gate — that
  freshness is owned on `main` (ADR-013). A PR should leave the inherited `graphify-out/` untouched.
  When you want a **branch-current** view locally, run `graphify update .` (AST-only, free) to
  refresh your working copy, query it, and leave it **uncommitted** — do not include it in the PR.
  A generated `.githooks/pre-commit` guard provides an earlier local warning when the clone has
  `core.hooksPath=.githooks`; that Git config is clone-local and cannot be activated by merging the
  hook. Required PR CI is therefore always the enforcement of record. The post-merge job validates
  `graph.json` as a node-link graph and applies the same manifest integrity contract as PR CI: the
  manifest must be an object keyed by safe corpus paths, every value must be an object, and every
  `ast_hash` must be 32 lowercase hexadecimal characters. Validation happens before staging or
  pushing, then the job converges after the last source merge. The job sets
  `GRAPHIFY_NO_BACKUP=1` because Git history already preserves
  every prior graph; otherwise graphify snapshots a curated graph into a dated
  `graphify-out/YYYY-MM-DD/` directory before overwriting it. The privileged main-side job also
  removes any already-tracked graphify path outside the canonical five-file set, which is the only
  cleanup path consistent with the invariant that PRs never modify `graphify-out/**`. graphify
  refuses to overwrite a committed graph when the regenerated graph has fewer nodes. In the clean,
  stateless post-merge checkout, the workflow retries any failed ordinary refresh once with
  `--force`; graphify's failure text is not a stable interface, while `--force` accepts a shrink
  without bypassing corrupt-graph guards. A failed retry remains fatal. Every accepted reduction
  emits an Actions warning, writes the old and new counts to the job summary, and includes the node
  delta in the graph commit subject. If a refresh is wedged, run **Graphify graph refresh** from the
  Actions UI with the boolean `force` input enabled. That recovery still reports any node reduction
  and does not change ADR-013 ownership: PRs must continue to leave `graphify-out/**` untouched.
