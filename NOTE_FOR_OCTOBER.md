# Note for October Ilias (written 2026-08-12, on the GPU box)

You paused the rover work in mid-August. This is everything you need to
restart without re-reading months of history. Read this + the matrix in
[docs/SYSTEM_OVERVIEW.md](docs/SYSTEM_OVERVIEW.md) and you're current.

## FIRST 30 MINUTES — before anything else

1. **Sync this box's commits back to the laptop.** The laptop is the source
   of truth, but August's work (9 commits, `7639395`..`dbd4ba6` on branch
   `rover-pi-integration`) was committed HERE on the GPU box
   (`/media/ilias/bigStorage/projets/gs/ros2_ws_rover`). If you re-export
   from the laptop without pulling these first, you lose them.
2. Skim the capability matrix (bottom of docs/SYSTEM_OVERVIEW.md) — it is
   the honest ledger of what is proven vs merely built.
3. Figures/animations/raw data from all sim tests are mobile-readable at
   github.com/ilisparrow/rover-sim-results (private).

## Where things stand (August summary)

**Proven in sim:** the terrain/ditch stack works end to end. A rover that
used to plan straight across a 0.5 m trench now maps it as a lethal wall
from 6 m away, refuses impossible goals, FINDS a legal crossing when one
exists (the y=+7 bridge) and drives it. Also proven: the 6-DoF ground
truth (z tracks terrain), Voxel-SLAM on real data, the error-snapshot
pipeline, interim NDT relocalization, MARSIM point-realistic lidar.

**The two open sim items (≤1 day on this box):**
- **Heightmap scale bug:** gz normalizes an image heightmap to the image's
  own MAX pixel, not 255. All generated courses are 255/152 = 1.678x
  taller than designed, and sdf_to_pcd.py uses the other convention so
  MARSIM's lidar world is flatter than the physics. Fix make_heightmap.py
  to write full-range images, regenerate PNGs + PCDs, re-verify one drive.
- After that fix: one rerun of the bridge mission on 6-DoF truth (it
  crossed on the planar run; on 6-DoF it ran out of time on the
  accidentally-21° ramp).

## THE FIELD LIST — what you actually paused for

**Gate (bench, ~1 h):** `drive_arbiter` + the `/cmd_vel_safe` chain have
NEVER run on hardware. Bench-test with wheels off the ground before any
field session. If this fails nothing else matters.

**Field day A — the 10 km/h decision (see docs/SPEED_DOSSIER.md for the
full protocol card):** duty sweep 0.2→0.95 on a ≥60 m flat strip, both
directions, full pack, RTK + per-wheel erpm logged. PASS = ≥2.9 m/s at
duty ≤0.90. Then: switch diff_drive to VESC ERPM control (~1736·v), stage
nav2 caps 0.5→1.0→2.0→2.8 with collision-monitor lookahead ≥2.5 s and the
12 m costmap.

**Field day B — sensors:** RTK+LIO/IMU EKF fusion run (code ready, needs
sky view) · measure real track width vs sim's wheel_separation guess
(tune BEFORE touching MPPI) · ground-segmentation visual check ·
camera↔lidar extrinsic: resolve the 16 mm tx disagreement between runs.

**Pi 5:** run rover_terrain on it (CPU headroom unknown), then real-LIO
terrain validation on real ground; expect the conservative 0.10 m step
threshold to over-flag — tune from a real bag.

**Ops loose ends:** snapshot pipeline S3 legs from the credential-holding
machines (one real run) · watcher startup-crash blind spot · process
`snapshot_20260728_232620` on the Pi · delete stale src/SUPER on the Pi ·
wheel odometry from VESC erpm.

## Gotchas that WILL bite you again (all learned the hard way)

- **NEVER call `/world/<n>/set_physics` on a live gz world** — it corrupts
  heightmap collision and the rover wedges itself. RTF caps go into the
  world file pre-launch (`sim_up.sh --rtf=X` does this now).
- MARSIM tops out at ~2.5 lidar frames per WALL second on this box:
  `--rtf=0.3` or LIO wedges permanently (1 s scan-gap gate) and the
  collision monitor stop-goes.
- colcon build on this box MUST source the ros2_extra underlay first, or
  install/setup.bash silently loses nav2:
  `source /opt/ros/jazzy/setup.bash && source /media/ilias/bigStorage/projets/gs/ros2_extra/root/opt/ros/jazzy/setup.bash && colcon build ...`
- pkill/pgrep `-f` self-match: never put a process name AND its spawn in
  one shell line; use the `[x]` char-class trick or sim_kill.sh.
- Sim test harness flags: `--gt-loc` (ground-truth localization — the
  trial corridor degenerates real LIO on purpose), `--terrain`
  (TERRAIN=1: costmaps + collision monitor read /terrain/obstacles).

## The standing design decisions (do not silently change)

- Voxel-SLAM is the target mapper (loop closure). RKO-LIO stays the
  odometry. The NDT map localizer is INTERIM.
- Never substitute a named design choice without asking (yourself, now).
