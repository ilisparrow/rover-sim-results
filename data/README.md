# Raw run captures (numpy .npz)

Each file is one full run, captured live by a subscriber node. Load with
`numpy.load(path)`. Keys:

- `gt`  — N x 4 array: [sim_time_s, x, y, z] ground-truth pose (~30 Hz)
- `lio` — same layout, the localization the stack believed
- `plan` — last nav2 global plan, M x 2 [x, y]
- `final_local_data` / `final_local_meta` — last local costmap
  (`meta` = [resolution, origin_x, origin_y, width, height]); same for
  `final_global_*`
- `snap8_*`, `snap14_*`, `snap17_*` — costmap + plan snapshots frozen when
  the rover first crossed x = 8 / 14 / 17 m
- `terrain_grid` — terrain-node classified cells, K x 4
  [x, y, ground_z, class] (0 free, 1 slope, 2 step, 3 ditch)

Files:
- `baseline_run.npz` — stock pipeline: plan crosses the invisible ditch,
  rover froze at x=8.3 on the ramp
- `terrain_v1_drove_into_trench.npz` — terrain node BEFORE the latched
  ditch fix: warning erased itself on approach, rover entered the trench
- `mission_bridged_course.npz` — the validated mission: goals at 12 m and
  22 m (via the y=+7 bridge) PASS; every figure in this repo renders from
  this file
