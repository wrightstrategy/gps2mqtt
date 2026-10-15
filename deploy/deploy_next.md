# Pending deployment steps

## Switch gps2mqtt to the public Renovate preset (SWW-237, PR #114)

- When: publish the public preset before merging PR #114; verify Renovate after merge.
- Tiers: GitHub and Mend Renovate Cloud.
- Do: merge the reviewed `wrightstrategy/renovate-presets` publication PR #3. Confirm its `default.json` is publicly readable. Then merge PR #114 and request a fresh Renovate scan from the Dependency Dashboard.
- Verify: `https://raw.githubusercontent.com/wrightstrategy/renovate-presets/main/default.json` returns 200 without authentication, the `main` ruleset requires the GitHub Actions `verify` check, and the new Mend scan resolves the preset without a configuration error. Close gps2mqtt#22 only after the scan evidence.
- Why: a public repository cannot extend Bridge's private preset, and switching the consumer before its public preset exists would leave Renovate stopped.
