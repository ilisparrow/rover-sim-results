# Chapter: BotanicGarden (real garden vegetation)

## 1. What it looks like

Real botanic garden (Tongji Univ., Shanghai): thick woods, grasslands,
riversides, narrow trails. Recorded with the rover's EXACT sensor pair —
Livox AVIA lidar + its built-in BMI088 IMU.

![sample scenes: camera (top), their 3D map (middle), semantic labels (bottom)](botanic_sample_scenes.png)
![the recording robot — AVIA + BMI088 on the sensor tower](botanic_robot.png)

Ground truth: the whole garden surveyed with a tripod laser scanner (11 mm
global accuracy); robot poses from lidar localization in that map,
cross-checked to 0.6–2.3 cm. NOT GPS — GPS dies under this canopy.

## 2. Estimated trajectories vs ground truth

All systems ran on identical raw data (lidar + IMU only). Black = survey
ground truth; colored = each system, rigidly aligned.

**1018-00 — 115 m loop, woods/grass (easy):**
![1018-00](botanic_1018_00_topview.png)

**1018-13 — 201 m, mixed (medium):**
![1018-13](botanic_1018_13_topview.png)

**1006-01 — 766 m, dense vegetation (the lidar-killer: published Fast-LIO2
collapses here with 39.7 m):**
![1006-01](botanic_1006_01_topview.png)

## 3. Results table

Median / p95 / max trajectory error vs ground truth (m):

| sequence | RKO-LIO | BIEVR | Voxel-SLAM | Fast-LIO2 (paper) |
|---|---|---|---|---|
| 1018-00 (115 m) | 0.35 / 0.58 / 0.69 | 0.14 / 0.34 / 0.37 | **0.09 / 0.16 / 0.25** | 2.15 |
| 1018-13 (201 m) | 0.30 / 0.71 / 0.76 | **0.22 / 0.64 / 0.65** | 0.25 / 0.70 / 0.71 | 2.39 |
| 1006-01 (766 m) | 3.35 / 7.05 / 7.23 | 2.09 / 3.95 / 4.17 | **0.60 / 2.04 / 2.14** | 39.7 |

Failures: RKO-LIO 0 · BIEVR 0 · Voxel-SLAM 1 self-recovered reset (1006-01).

Compute on the same data (desktop Ryzen 5 2600): RKO-LIO 9× real-time /
0.19 GB · BIEVR 4.5× / 1.15 GB · Voxel-SLAM keeps up at 2× replay / 2.13 GB.

Additional experiments on this dataset (details in [BENCHMARK.md](BENCHMARK.md)):
- Hybrid Voxel-SLAM + RKO-prior: no improvement (3 resets vs 1 stock).
- Leak-free relocalization (map from 1018-00, queries from 1018-13):
  Voxel-SLAM ✅ (11 matches, 0.18 m) · BIEVR ❌ (all seeds rejected) ·
  NDT localizer ⚠️ 1/8 + honest no-fix; yaw-search returns confident WRONG
  poses in vegetation — keep off.
- Ground segmentation (Patchwork++ + terrain grid) on real vegetation:

![ground segmentation](terrain_seg_worldmap.png)
![one scan](terrain_seg_scan.png)
![terrain grid](terrain_grid.png)

## 4. Conclusion

In vegetation, everything works and Voxel-SLAM wins: best accuracy on 2 of 3
sequences (dominant on the hardest), the only system that relocalizes
cross-session, one benign reset in 1.1 km total. BIEVR is a consistent
second. RKO-LIO is 3–6× less accurate but 10× cheaper and never misbehaves.
Both sim-predicted failure modes (BIEVR divergence, Voxel reset storm) did
NOT appear on real vegetation — but see the TartanDrive 2 chapter: they are
real on other terrain.
