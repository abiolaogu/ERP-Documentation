# Sovereign Next16 Enforcement Tranche

Date: 2026-03-01
Workspace: /Users/AbiolaOgunsakin1/ERP

## Completed in this tranche

1. Operational defaults switched to Next16 scaffold
- `Makefile` now prioritizes `apps/sovereign-next16` in:
  - `dev-module`
  - `frontend`
  - `frontend-install`
  - `frontend-build`
- `status` output now includes a dedicated `Next16` column.

2. New Next16 GA gate added
- Script: `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/ga-next16-cutover-check.sh`
- Make target: `make ga-next16`
- Included in full gate run: `make ga-all`

3. Predeploy checks expanded
- `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/ga-predeploy-check.sh` now requires:
  - `.workik/config.yml`
  - `apps/sovereign-next16/package.json`
  - `docs/sovereign/SOVEREIGN_NEXT16_WORKIK_MIGRATION_2026-03-01.md`
- Added `Phase 3B` baseline execution of the Next16 gate.

4. README standardization completed portfolio-wide
- Marker applied in all repos: `<!-- SOVEREIGN_NEXT16_STANDARD_2026_03 -->`
- Coverage: `24/24` repository READMEs.
- Supporting scripts:
  - `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/apply-next16-readme-standard.sh`
  - `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/repair-next16-readme-section.sh`

5. Gate-script reliability fix
- Corrected npm invocation semantics from `npm run <script> --if-present` to `npm run --if-present <script>` in gate scripts.

6. Legacy frontend manifest retirement completed
- Script: `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/retire-legacy-frontend-manifests.sh`
- Result: legacy `web/apps/web/frontend` manifests are forwarding shims to `apps/sovereign-next16`.
- Debt enforcement in cutover gate is now enabled (`Debt Enforce: 1`).

7. Strict enforcement hardening completed
- Gate scripts now execute app-local npm operations with workspace isolation:
  - `npm install --workspaces=false`
  - `npm run --workspaces=false --if-present <script>`
- Updated files:
  - `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/ga-next16-cutover-check.sh`
  - `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/ga-predeploy-check.sh`
- Legacy shim package names are now path-unique to avoid duplicate workspace-name collisions:
  - `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/retire-legacy-frontend-manifests.sh`
- Strict cutover run completed:
  - Command: `GA_NEXT16_BUILD_ENFORCE=1 make ga-next16`
  - Outcome: `GA_NEXT16_STATUS=PASS`
  - Validation artifact:
    - `/Users/AbiolaOgunsakin1/ERP/Documentation/SOVEREIGN_NEXT16_STRICT_ENFORCEMENT_20260301T225856Z.md`

## Current validated status

From `/Users/AbiolaOgunsakin1/ERP/Documentation/SOVEREIGN_NEXT16_CUTOVER_STATUS.md`:

- Repositories checked: `24`
- Next16 scaffold present: `24/24`
- Workik Next16+Shadcn config present: `24/24`
- Next16 apps clean of Refine/AntD: `24/24`
- Transitional legacy frontend debt (legacy manifests with Refine/AntD): `0/24`

## Completion Status

- Frontend standardization and legacy manifest retirement tranche is complete.
- Strict Next16 cutover enforcement is complete and green (`GA_NEXT16_BUILD_ENFORCE=1`).
