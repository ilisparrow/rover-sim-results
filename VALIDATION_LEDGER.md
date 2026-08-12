# Validation ledger — what is proven where, and what each item still needs

Companion to the capability matrix in [SYSTEM_OVERVIEW.md](SYSTEM_OVERVIEW.md).
The matrix says WHAT is verified; this ledger says AT WHICH LEVEL and what the
NEXT level requires. Update in the same commit as the work (house rule).

Validation levels, in order:
- **SIM** — passes in Gazebo/MARSIM on this box
- **BENCH** — rover powered, wheels off the ground
- **MOVING** — rover driving, anywhere (parking lot / indoors)
- **FIELD** — outside: terrain, sky view for RTK, real distances
- (a dash — means the level is not applicable)

| Item | SIM | BENCH | MOVING | FIELD | Next step |
|---|---|---|---|---|---|
| drive_arbiter + /cmd_vel_safe chain | ✅ | ❌ needed | ❌ | ❌ | bench first — gate to everything |
| Terrain/ditch stack (GT localization) | ✅ 2026-08-01/02 | — | — | ❌ | superseded by real-LIO row below |
| Terrain/ditch stack (real LIO) | ✅ 2026-08-12: proving mission 3/3 PASS, drift ≤0.11 m, bridge crossed both ways | — | ❌ Pi 5 CPU | ❌ tune step_max on real ground | hardware next |
| 6-DoF sim ground truth | ✅ 2026-08-02 | — | — | — | sim-only tool, done |
| Heightmap scale fix | ✅ 2026-08-12 (e1afcf9) | — | — | — | done |
| Bridge crossing on 6-DoF truth | ✅ 2026-08-12: crossed both ways (maxx 22.0; goal-2 stopwatch expired pre-arrival); clean PASS on the real-LIO proving run | — | — | — | done |
| Voxel-SLAM hybrid (RKO-LIO prior) | 🟡 mechanism proven (85 degenerate scans ridden, 0 resets); accuracy-neutral on plane-rich ground (0.31 vs 0.20 m shape err); DECISIVE plane-poor A/B pending (trunks-only bag) | — | ❌ | ❌ forest session | forest-loop bag A/B |
| Voxel-SLAM loop closure / reposition | ✅ real bag | — | — | ❌ live on rover | unchanged by hybrid (verify) |
| LIO fitness monitor | ✅ live + validated: conditioning correlates +0.52 with true error over 3883 scans (proving bag); alarm threshold still needs a degenerate-run bag | — | ❌ | ❌ thresholds on real data | degenerate-bag threshold |
| Wheel odometry (VESC erpm) | — code done (cd6cc30) | ❌ erpm sanity | ❌ taped-line calibrate | ❌ effective_track vs RTK | bench first |
| Snapshot watcher startup blind spot | 🔧 code done (liveness guard, cd6cc30) | ❌ kill-a-node test | — | — | bench test |
| Snapshot S3 legs | — | — | — | ❌ from robot | one real run (needs creds machines) |
| 10 km/h chain (duty sweep → ERPM → nav2 caps) | ✅ analysis | ❌ | ❌ | ❌ THE field day | docs/SPEED_DOSSIER.md protocol |
| EKF RTK+LIO fusion | ✅ sim | — | — | ❌ sky view | field day B |
| Track width measurement | — | — | ❌ yaw-rate test | — | before MPPI retune |
| Camera↔lidar extrinsic 16 mm dispute | — | ❌ re-calib | — | ❌ | field day B |
| Bank leg of trial course | 🔧 queued (item 5) | — | — | — | sim, east spawn |

## Log

- 2026-08-12 (later): proving mission 3/3 PASS under real LIO — terrain
  stack sim-validated end to end. Fitness monitor validated (cond ↔ error
  +0.52). Hybrid mechanism proven, decisive A/B queued on a plane-poor bag.
  Bank leg still queued. New gotcha: MARSIM frame ceiling scales with PCD
  size (proving world needs --rtf=0.15).
- 2026-08-12: ledger created. Items 1-5 of the no-rover work plan started
  (this session). Levels for finished work backfilled from the matrix.
