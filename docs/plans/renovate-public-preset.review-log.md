---
doc: docs/plans/renovate-public-preset.md
reviewed_sha256: 1ffa5404aa6694304646aac34138faffa10c7f6fb38a51bdc01aa5da0cb3a085
rounds: 2
verdict: ready
reviewer: claude
session_ref: a157db78-45ce-4f50-812d-3a6af1108423
unresolved: []
---

# Review log — renovate-public-preset.md

## Round 1 — Claude

VERDICT: ANOTHER-ROUND

**What I examined.** I read the plan `docs/plans/renovate-public-preset.md` against the committed frame: the Standing rules in `AGENTS.md`, plus these sections of `docs/agents/ws-dev-reference.md`: Dependency updates (the Binding rollout invariant is at :753–761), Runner placement (:349–399), GitHub Packages, and Orca worktree checkpoints. I also read:

- **This repo:** the whole tree and the `renovate.json` history. Onboarding came in at `66716b7`/`f018e60`. The merge `d541ac4` left invalid JSON (a missing comma) on `main` from 2026-09-08 until #54 fixed it on 2026-09-13. `requirements.txt` and `gps2mqtt.py`.
- **A local bridge checkout, read only:** `~/Projects/bridge` at `b0523b7` (2026-09-29, possibly stale). I read `renovate-config/default.json`, its README, `scripts/audit-repositories.ts` (the sync-PR path), and how `repo-bootstrap.ts` handles `publicRepo`. Nothing was written.
- **External docs:** Renovate's private-packages page confirms the root cause. It says: "If you have a preset in a private repo but reference ("extend") it from a public repository then it won't work". Its suggested fix is to "break your presets into public and private ones". Mend's credentials docs say org-level secrets "are inherited by all repositories in your organization".

**Limitations.** I could not read rvc2mqtt (no checkout, nothing supplied), current bridge `main`, Mend job logs, or the text of issues #22 and #23. Those come only from the caller's evidence. No `gh` calls were made.

**What holds up.** The diagnosis is right. The chosen design is the clean, durable one:
- One public preset at `wrightstrategy/renovate-config`. That name is also Renovate's conventional preset-repo name.
- Fleet migration through the generator.
- Platform work routed as a PLT request (`ws-dev:repo-request`) and a new repo via `repo-create`.

It rightly rejects making bridge public, making gps2mqtt private, and a hand-copied local config. The secrets claim is also correct. The bridge README says secrets are applied *after* preset resolution, and Mend org secrets reach every repo in the org, so the `{{ secrets.PACKAGES_READ_TOKEN }}` name can live in a public preset. The scope stays tight. The one blocker is rollout order.

### F1 (P1, correctness): The rollout order lets the fleet migration switch Renovate on before each public repo has a required check

- **Requirement violated:** `docs/agents/ws-dev-reference.md:753-761` (Binding): "No automated update ecosystem … activates in a repo until that repo has a stable PR check and that check is required … A repo with no CI gets a check before it gets automation."
- **The plan contradicts itself.** Step 2 says "This gate must be green before Renovate is retried". But Step 1, which comes first, includes "migrate generated consumer configs". That migration *is* the retry.
  - Migration works by opening sync PRs (`audit-repositories.ts:412`). Their body says "Repository CI remains the merge gate", yet gps2mqtt has no CI and no ruleset, so nothing gates them.
  - These PRs are merged routinely: #54 and #55 were merged the day they were opened.
  - Once a sync PR switches `renovate.json` to the resolvable public preset, the next Mend scan (failed repos are scanned daily, per the preset README) activates Renovate on an ungated `main`.
- **rvc2mqtt has no ordering at all.** It only gets "Apply the same CI prerequisite to rvc2mqtt separately" (plan step 3). Step 1's fleet-wide migration reaches it with nothing holding it back.
- **The preset makes this worse.** Its routine group sets `automerge: true` / `platformAutomerge: true` (`default.json:71-88`). In a repo with any CI check that isn't required, Renovate can merge its own non-major waves after those checks pass, without a required gate.
- **Fix: reorder.** Step 2 has no dependency on the platform work. Put the CI job and required check for each affected public repo (gps2mqtt, rvc2mqtt, and any other public consumer the PLT request turns up) first. Then the platform migration can merge freely.
  - If the order stays as written, the plan and the PLT request must say explicitly that the sync PR into any repo without a required check is not merged until that repo's gate exists.
  - The new `renovate-config` repo is itself a registered consumer. Confirm `repo-create`'s required-checks path gives it a required check before its own generated `renovate.json` turns Renovate on.

### F2 (P2, correctness): Step 3's closure criterion may not be satisfiable for gps2mqtt

- **The criterion:** "Confirm a normal dependency PR can be created and is held by the required check before merging", with #22 closed "only after each repo is verified".
- **Why it may never be met:**
  - The preset's `enabledManagers` is `["bun","npm","pep621","github-actions","dockerfile","custom.regex"]` (`default.json:10`). That excludes `pip_requirements`, and gps2mqtt's only manifest is `requirements.txt`, so Renovate finds no Python dependencies at all.
  - The only update candidates are the Actions in the workflow Step 2 creates. Those will be SHA-pinned at current releases, and a new release also has to clear `minimumReleaseAge: 1 day`.
  - So a dependency PR may not exist for weeks, and closing #22 would wait on an arbitrary upstream release.
- **It is also redundant.** Step 2 already proves the gate: it requires the check to run on a docs-only PR and then makes that check required.
- **Fix:** close on what the plan already names. Use the Mend job log or dependency-dashboard update showing the preset resolved and managers detected. Add "if an update PR exists, confirm it waits on the required check" as an opportunistic check, not a precondition.

### F3 (P3, correctness): Step 2 doesn't give enough detail to execute for a public repo's required check

- **Runner tier is unstated.** The reference sends fork-PR code from public repos to GitHub-hosted runners, with the reason documented beside the `ubuntu-*` line (:370). It also binds public repos away from the `Homelab-runners` group (:374). An implementer applying the private-CI default `[self-hosted, ci]` would get this wrong. Name GitHub-hosted, plus the usual SHA pins and `permissions: contents: read`.
- **The "stable source check" is unnamed, and the obvious one fails:**
  - `gps2mqtt.py` does `from gps import *` and calls `gps(mode=WATCH_ENABLE | WATCH_NEWSTYLE)` (:117). That is gpsd's `gps` module, not the declared `gps3`.
  - The script also reads `gps2mqtt.ini` at module scope.
  - So an import smoke test fails. A syntax-level check (e.g. `python -m py_compile gps2mqtt.py`) is what actually works.
- **"Installs the declared Python requirements" doesn't earn its place.** Those requirements are unpinned, and one of them is unused by the script. Installing them adds a PyPI network dependency to a required check without feeding the check. Drop it unless the chosen check uses it.

### F4 (P3, simplification): Make the old bridge preset a forwarder instead of a second copy

"Keep the old bridge preset available until consumers have migrated" leaves two copies during the transition, which works against "Do not leave two manually maintained copies."

Instead, replace bridge's `renovate-config/default.json` with `{"extends": ["github>wrightstrategy/renovate-config"]}` (or the explicit path) at cutover:
- There is one source from day one.
- Private consumers keep working untouched.
- The migration loses its time pressure.
- Removing the forwarder becomes the single cleanup step.

### F5 (P3, correctness): The plan names a Linear issue only for the platform work

The Binding Work-management rule is "Track agent work in Linear, never GitHub issues; link PRs to Linear". The gps2mqtt (and rvc2mqtt) CI-and-config work needs its own SWW Linear issue, blocked by the PLT request, so the cross-repo dependency is visible. #22 is Renovate's own bot issue, not a tracking issue. One sentence fixes this.

### S1: After the fix, Renovate still won't manage gps2mqtt's Python dependencies

Because `pip_requirements` is not in `enabledManagers`, "restoring Renovate" here means it manages only the CI workflow's Actions. Moving to `pyproject.toml` with `uv.lock`, the Python Default, would put the dependencies under `pep621` and lockfile maintenance. That adds capability the plan never claimed, so it's the owner's call. The Tradeoffs section should still state the gap so approval rests on accurate expectations.

### S2: The publication review should cover the README, not just `default.json`

The bridge README that the plan would transfer contains:
- an `op://Agent Scott Shared/gh-pat.renovate-mend` 1Password item path;
- internal Mend operating notes;
- a relative link into private bridge docs (`../docs/specs/bridge-consolidation-hld.md`).

`default.json` also carries a bridge-only `customManagers` regex that could stay in bridge's local config. None of this is a credential. The plan's "material that cannot be public" review is the right place for these, and the owner decides.

Sources:
- [Renovate — Private package support](https://docs.renovatebot.com/getting-started/private-packages/)
- [Renovate — Mend-hosted credentials](https://docs.renovatebot.com/mend-hosted/credentials/)
- [Renovate — Using secrets with Mend cloud Apps](https://docs.renovatebot.com/mend-hosted/app-secrets)

## Round 1 — author disposition

- F1: fixed — safety gates now precede public-consumer migration.
- F2: fixed — scan evidence replaces a hypothetical update PR as closure criterion.
- F3: fixed — the runner, permissions, and exact check are named; unused install removed.
- F4: fixed — bridge's preset becomes a forwarder during migration.
- F5: fixed — separate SWW Linear carriers and PLT dependency are specified.
- S1: noted for human — Python requirements remain outside Renovate manager coverage.
- S2: noted for human — publication inspection covers README and preset.

## Round 2 — Claude

VERDICT: READY

**What I examined this round.** I re-read all of the revised `docs/plans/renovate-public-preset.md` (lines 1–34, still untracked in the worktree). I checked each change against the round-1 evidence: the Binding rollout invariant in `docs/agents/ws-dev-reference.md:753-761`, runner placement (:349–399), the shared preset's `enabledManagers` and automerge settings (`~/Projects/bridge/renovate-config/default.json`, read-only local checkout from 2026-09-29), and the sync-PR path in bridge's `scripts/audit-repositories.ts`. I took the author's dispositions as claims and checked each against the text; none was accepted on assertion. As in round 1, I could not see rvc2mqtt's tree or Mend's logs. No conclusion below depends on them.

**Prior findings**

- **F1: resolved.** The rollout now starts with "Safety gates first" (step 1, line 21). gps2mqtt gets an unconditional PR check made required on `main`. rvc2mqtt and any other public consumer the PLT audit finds get the same "before any sync PR switches their config to a resolvable preset".
  - Step 2 (line 22) adds the matching hold: "Hold any public consumer's sync PR until its required check from step 1 is active."
  - The new preset repo must have "its own required CI check before Renovate activation".
  - The forwarder doesn't reopen the problem. Public consumers still point at private bridge until their held sync PR merges, so it can't switch Renovate on early. Private consumers are already active.
  - The order now matches the Binding invariant, and the earlier contradiction is gone.
- **F2: resolved.** Step 3 (line 23) now closes on a Mend job log or dashboard showing the preset resolved and managers detected. The update-PR check only applies "If an update PR exists", and the plan says plainly not to wait for an upstream release to create one.
- **F3: resolved.** Step 1 now names:
  - the exact check, `python -m py_compile gps2mqtt.py`, with an accurate reason for not using an import smoke test (config is read at import time and the script needs gpsd's `gps` module);
  - a GitHub-hosted runner, giving the public-fork-PR reason the reference asks for;
  - SHA-pinned actions and `permissions: { contents: read }`.

  The unpinned requirements install is gone, with a reason given.
- **F4: resolved.** At cutover, bridge's old preset becomes a forwarder that extends the public one, and it is removed after migration is verified. That fits the Decision's "Do not leave two manually maintained copies."
- **F5: resolved.** Line 17 adds separate SWW Linear issues for gps2mqtt and rvc2mqtt, records the dependency on the PLT request, and treats Renovate's bot-created GitHub issues as diagnostic evidence only.
- **S1: noted.** The Python dependency gap is now stated twice: in Problem (line 11) and as its own tradeoff (line 30), to be tracked separately. That's honest, and it adds no mechanism to the plan.
- **S2: noted.** Step 2 now has publication review cover both `default.json` and the README, looking for credential values, internal operating details, and links to private docs.

**Accretion check.** The plan grew only where a finding called for it. The new text is an ordering change, one hold sentence, a concrete check command, a forwarder that replaces a temporary second copy, and two honest statements of limits. No new gates, mechanisms or normative caps were added without a correctness reason, so the drift tripwire is not triggered. The design is unchanged and still the cleanest durable option: one public preset, migrated through the generator, with the platform work requested through PLT.

**New findings.** None. The changed text is consistent with itself, with the Decision and Approval boundary sections, and with the Binding rules on required checks, public isolation and work management. The plan is ready for the owner's approval decision. It still authorizes no implementation, as its Approval boundary says.

## Round 2 — author disposition

- All prior findings resolved; no new findings. Exact document hash verified before recording.
