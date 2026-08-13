# Odometry/SLAM benchmark — RKO-LIO vs BIEVR vs Voxel-SLAM

All runs on identical raw data (Livox AVIA + its BMI088 IMU only) from the
BotanicGarden dataset — real vegetation, survey-grade ground truth (cm-level,
not GPS). Machine: AMD Ryzen 5 2600 (6C/12T), 64 GB, no GPU used.

## Accuracy (median / p95 / max error vs ground truth, metres)

| sequence | RKO-LIO | BIEVR | Voxel-SLAM | Fast-LIO2 (paper) |
|---|---|---|---|---|
| 1018-00, 115 m, woods/grass | 0.35 / 0.58 / 0.69 | 0.14 / 0.34 / 0.37 | **0.09 / 0.16 / 0.25** | 2.15 |
| 1018-13, 201 m, mixed | 0.30 / 0.71 / 0.76 | **0.22 / 0.64 / 0.65** | 0.25 / 0.70 / 0.71 | 2.39 |
| 1006-01, 766 m, dense — the "lidar killer" | 3.35 / 7.05 / 7.23 | 2.09 / 3.95 / 4.17 | **0.60 / 2.04 / 2.14** | 39.7 (collapsed) |

Failures across everything: RKO-LIO 0, BIEVR 0, Voxel-SLAM one self-recovered
reset (on the killer course). All three beat the published baseline by 10-60x.

## Compute (sequence 1018-13, 198 s of data)

| system | how it ran | speed vs real-time | CPU avg (p95) | peak RAM |
|---|---|---|---|---|
| RKO-LIO | offline, max speed | **9.0x** | 2.6 cores (5.3) while sprinting -> ~0.3 core-equivalent at real-time pace | **0.19 GB** |
| BIEVR | offline, max speed | 4.5x | 6.3 cores (7.5) while sprinting -> ~1.4 core-equivalent at real-time pace | 1.15 GB |
| Voxel-SLAM | live replay 1.0x | keeps up | 1.4 cores (1.9) | 2.13 GB |
| Voxel-SLAM | live replay 2.0x | keeps up, 0 resets | 1.6 cores (2.6) | 2.14 GB |

Reading the compute table honestly:
- RKO-LIO is in a different weight class: ~10x cheaper than either SLAM in
  CPU and RAM. That is what a Pi 5 budget looks like.
- Voxel-SLAM at real-time is lighter than expected (1.4 cores) and has >=2x
  headroom on this desktop — but 2.1 GB RAM and growing with map size, plus
  Ceres/GTSAM bursts, on a Pi 5 that also runs nav2 + terrain is unproven.
- These are desktop cores; Pi 5 cores are several times weaker. Numbers here
  bound the answer, they do not settle it.

## Relocalization (leak-free: map from 1018-00, queries from 1018-13)

| system | result |
|---|---|
| Voxel-SLAM | relocalized: 11 cross-session matches, 0.18 m consistency |
| BIEVR | failed (ScanContext seeds all rejected by ICP verification) |
| NDT on RKO map | 1/8 random queries fixed; yaw-search returns confident WRONG poses in vegetation — keep off |

## Verdict

- **Accuracy:** Voxel-SLAM wins overall (dominant where it matters most — the
  hard sequence); BIEVR consistently second, best on the medium sequence.
- **Robustness:** all three survived everything; only Voxel-SLAM ever reset
  (once, self-recovered).
- **Compute:** RKO-LIO by a mile.
- **Repositioning:** Voxel-SLAM is the only one that works cross-session in
  vegetation.
- **Architecture consequence:** RKO-LIO stays the control-loop odometry
  (cheap, continuous, never jumps); Voxel-SLAM is the mapper + repositioning
  layer. Replacing RKO-LIO with Voxel-SLAM alone would hinge on (a) zero
  resets in our own forest and (b) fitting 2+ GB / 1.5+ desktop cores into
  the Pi budget — (b) looks unlikely; a compute test on the Pi settles it.

Top views: [1018-00](botanic_1018_00_topview.png) ·
[1018-13](botanic_1018_13_topview.png) ·
[1006-01](botanic_1006_01_topview.png)
