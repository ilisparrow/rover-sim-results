# BotanicGarden evaluation (real vegetation, survey ground truth)

Real garden dataset (Tongji Univ., RA-L 2024), recorded with the rover's exact
sensors: Livox AVIA lidar + its built-in BMI088 IMU. Ground truth: the whole
garden was surveyed with a tripod laser scanner (11 mm global accuracy); robot
poses come from lidar localization in that map, cross-checked to 0.6-2.3 cm.
NOT GPS — GPS dies under canopy; this doesn't.

All methods ran on identical raw data (lidar + IMU only, no cameras, no wheel
odometry). Errors are absolute trajectory error vs ground truth after rigid
alignment. Yardstick: the dataset paper's own Fast-LIO2 result on the same
AVIA lidar.

## Sequence 1018-00 — 115 m loop, woods/grass (easy)

| method | median | p95 | max | failures |
|---|---|---|---|---|
| Voxel-SLAM | **0.091 m** | 0.162 | 0.252 | 0 resets |
| BIEVR | 0.140 m | 0.341 | 0.374 | 0 |
| RKO-LIO | 0.345 m | 0.576 | 0.694 | 0 |
| Fast-LIO2 (paper) | 2.15 m ATE | | | |

![1018-00 top view](botanic_1018_00_topview.png)

## Sequence 1006-01 — 766 m, dense vegetation (the lidar-killer)

The sequence where published lidar-inertial odometry collapses: Fast-LIO2
scored 39.7 m ATE here (10x worse than on any other sequence).

| method | median | p95 | max | failures |
|---|---|---|---|---|
| Voxel-SLAM | **0.598 m** | 2.039 | 2.141 | 1 reset, self-recovered |
| BIEVR | 2.086 m | 3.945 | 4.167 | 0, never diverged |
| RKO-LIO | 3.346 m | 7.048 | 7.230 | 0, graceful degradation |
| Fast-LIO2 (paper) | 39.7 m ATE | | | collapsed |

![1006-01 top view](botanic_1006_01_topview.png)

## Takeaways so far

- Voxel-SLAM's sim reset-storm (35 resets / 109 m in the synthetic trunk
  forest) does NOT reproduce on real vegetation: 0 resets on 115 m, 1
  recoverable reset on 766 m. Real vegetation gives its plane detector more
  to hold onto than bare synthetic trunks.
- BIEVR's sim divergence on featureless ground also does not reproduce: on
  the real degenerate course it beat RKO-LIO and never diverged.
- Both sim failure modes were real but pessimistic. Real-world ranking:
  Voxel-SLAM > BIEVR > RKO-LIO on accuracy; all three survive.

## The dataset

![scenes](botanic_sample_scenes.png)
![robot](botanic_robot.png)

Pending: hybrid (Voxel-SLAM fed with RKO-LIO prior), leak-free
relocalization test (map from 1018-00, query scans from 1018-13), ground
segmentation on real vegetation.

## Combination test: Voxel-SLAM + RKO-LIO prior (1006-01)

No improvement: 3 resets with the prior vs 1 without (its resets originate in
the re-initialization path the prior cannot reach — same diagnosis as in sim).
Recommendation unchanged: run Voxel-SLAM stock as the mapper, detect resets,
keep RKO-LIO as the control-loop odometry.

## Relocalization, leak-free (map from 1018-00, query scans from 1018-13)

Same garden, different recording — no query scan ever touched the map build.

| system | mechanism | result |
|---|---|---|
| Voxel-SLAM | 3D descriptors + pose graph | **Relocalized**: 11 cross-session matches, 0.18 m median consistency vs GT |
| BIEVR | ScanContext + ICP verify | **Failed**: candidates found, all rejected by ICP (fitness 1.9-8.7); status LOST |
| RKO-LIO map + NDT (ours) | NDT from an initial guess | 1/8 random mid-drive queries fixed (0.7 m); 7/8 honest "no fix". WARNING: with yaw-search on it returns confident WRONG poses (wrong basin scores 0.07, true pose 0.36 — vegetation aliases). Keep yaw-search OFF in vegetation. |

## Ground segmentation on real vegetation (1018-13)

Patchwork++ + the Kalman terrain grid, fed raw garden scans + RKO odometry:

![accumulated](terrain_seg_worldmap.png)
![one scan](terrain_seg_scan.png)
![terrain grid](terrain_grid.png)

Reading: the walkable path network comes out green and continuous; trees and
bushes red; the terrain grid flags hazard cells at vegetation edges. All
qualitative (the dataset has no drivability labels) but the separation is
clean on real-world grass, slopes and foliage.
