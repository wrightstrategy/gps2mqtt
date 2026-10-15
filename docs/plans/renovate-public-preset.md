# Restore Renovate for public Wright Strategy repositories

Authored-By: Codex
Date: 2026-10-14
Status: approved by owner on 2026-10-14; step 1 implemented in PR #112, pending merge

## Problem and evidence

Renovate opened [gps2mqtt#22](https://github.com/wrightstrategy/gps2mqtt/issues/22) on 2026-09-08 with `Cannot find preset's package (github>wrightstrategy/bridge//renovate-config/default.json)`. The path in `renovate.json` matches the existing file and the example in bridge's preset README. `gps2mqtt` is public; `bridge` is private. Renovate's [private preset guidance](https://docs.renovatebot.com/getting-started/private-packages/) explicitly says a public repository cannot extend a preset in a private repository. `rvc2mqtt` has the same public/private combination and an equivalent error in its issue #23. The inaccessible preset is the root cause; changing the JSON path or retrying the scan cannot fix it.

This repository also has no `.github/workflows`, branch protection, or rulesets. The binding dependency-update policy in `docs/agents/ws-dev-reference.md` requires a stable, required PR check *before* automated dependency updates activate. Renovate's onboarding PR was merged on 2026-09-08 without that prerequisite. No post-onboarding dependency PR appears in this repository. The issue's exact error is historical; a fresh Renovate run must confirm resolution after the migration. The current shared preset excludes Renovate's `pip_requirements` manager, so this repo's sole manifest, `requirements.txt`, will not produce dependency PRs even after preset access is repaired.

## Decision

Keep `gps2mqtt` public and preserve one fleet policy. Move the shared Renovate preset to a dedicated **public** organization repository, `wrightstrategy/renovate-config`, after checking that the preset contains no secret values or material that cannot be public. The existing `{{ secrets.PACKAGES_READ_TOKEN }}` reference is a name, not a credential; its value stays in Mend. Make that public repository the single maintained source of the preset. Update bridge's generator, validation, and documentation to point the fleet at the new location; migrate existing consumers through the generator. Do not leave two manually maintained copies.

This is a platform change. From this gps2mqtt worktree, request it through one unassigned PLT Linear issue under `ws-dev:repo-request`; the platform owner executes it under bridge's own gates. A new repository goes through `ws-dev:repo-create`, not manual creation. Track the gps2mqtt CI/config work in a separate SWW Linear issue, with the PLT dependency recorded; rvc2mqtt needs its own SWW carrier. Renovate's bot-created GitHub issues remain diagnostic evidence, not agent-work trackers.

## Rollout

1. **Safety gates first:** Add an unconditional pull-request CI job in gps2mqtt that runs `python -m py_compile gps2mqtt.py`. The script reads local configuration at import time and depends on gpsd's `gps` module, so an import smoke test is not a suitable first gate; installing the current unpinned requirements adds network failure modes without serving a syntax check. Use a GitHub-hosted runner for public fork PRs, SHA-pin third-party actions, and set `permissions: { contents: read }`. Verify the job runs on every PR, including a docs-only change, then make that exact check required on `main` with the established owner-final ruleset posture. Do the equivalent in rvc2mqtt and any other public consumers found during the PLT audit **before any sync PR switches their config to a resolvable preset**. Adding a required check before its workflow works would freeze merges.
2. **Platform migration:** Create the public preset repository through the governed repo-creation flow, including its own required CI check before Renovate activation. Transfer the preset and suitable tests/docs from bridge, inspecting both `default.json` and README for credential values, internal operating details, and links to private docs before publication. Validate public resolution without authentication. Replace bridge's old preset with a small forwarding preset extending the public one, so private consumers keep working while only one full policy is maintained. Change bridge's generator and checks to use the new URL, update the owning living documentation and decisions entry, and migrate generated consumer configs without deleting permitted local rules. Hold any public consumer's sync PR until its required check from step 1 is active. Remove the forwarder only after migration is verified.
3. **Local verification:** Consume the generator's new thin `renovate.json`, keeping the fleet policy central and any local exceptions described. Run local config validation and confirm the public preset resolves. Request a fresh Mend scan and inspect its job log or a current dashboard update showing successful preset resolution and applicable manager detection; absence of a new error alone is insufficient. If an update PR exists, confirm its required check runs and gates merge. Close #22 and the equivalent rvc2mqtt issue only after each repository's scan is verified; do not wait for an upstream release just to manufacture an update PR.

## Tradeoffs and limits

- A public preset lets public and private repositories inherit one policy and avoids publishing the app's source from a different visibility than today. It exposes the *policy text* to the public; review that text before publishing. Secret **values** stay outside git.
- A dedicated repository and fleet migration cost more than copying JSON into gps2mqtt, but prevent policy drift and preserve the generated-config contract. Making bridge public would expose the entire platform control plane; making gps2mqtt private would remove its public availability.
- A minimal required check is a merge guard, not proof that the GPS device, MQTT, and Traccar integration works. Do not claim runtime coverage from it. Stronger app tests can be added when those behaviors are changed.
- This fixes the configuration error and permits Renovate to manage the new workflow's Actions, but it does **not** update `gps3`, `paho-mqtt`, or `haversine` from `requirements.txt`. Bringing those Python packages under automation requires a separate decision: migrate the app to a supported PEP 621/uv manifest and lockfile, or change fleet manager coverage. That dependency-maintenance gap should be tracked separately rather than hidden by closing the config error.

## Approval boundary

The owner approved this plan on 2026-10-14. Step 1 is implemented in PR #112, pending merge; the remaining steps follow the platform and local PR, review, documentation, and deployment gates.
