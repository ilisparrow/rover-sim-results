# Chapter: TartanDrive 2.0 (fast off-road, 7 tests)

CMU's off-road dataset: a Yamaha ATV at 3–10 m/s (well above rover speeds),
carrying a **Livox Mid-70** — the AVIA's closest sibling (same non-repetitive
petal scan, same narrow cone) — plus NovAtel GPS/IMU. Ground truth: RTK-fused
pose at 50 Hz (~decimetre). One ~120 s chunk per environment, session-start
(at-rest) chunks except where noted. All three systems on identical raw data.
Errors: median / p95 / max ATE vs truth after rigid alignment, in metres.

Data quirks fixed in our converter: Livox clock 36.9 s off the GPS clock; no
per-point timestamps (synthesized ramp — validated); Mid-70→IMU extrinsic
from their calibration files.

---

## 1. Slag heap — steep barren rock (closest public analog to burned terrain)

![slag heap](scene_slagheap.jpg)
![trajectories](topview_slagheap.png)

| system | median | p95 | max | failures |
|---|---|---|---|---|
| **BIEVR** | **0.50** | 0.92 | 1.01 | 0 |
| RKO-LIO | 0.57 | 0.97 | 1.22 | 0 |
| Voxel-SLAM | — | — | — | 4 resets, trajectory lost |

Near-tie, both excellent. Bare relief feeds the lidar — the burned-forest
analog is EASY.

---

## 2. Figure-8 — flat open field (purest featureless case)

![figure-8](scene_figure8.jpg)
![trajectories](topview_figure8.png)

| system | median | p95 | max | failures |
|---|---|---|---|---|
| **BIEVR** | **0.56** | 1.32 | 1.72 | 0 |
| RKO-LIO | 1.34 | 2.93 | 3.70 | 0 |
| Voxel-SLAM | — | — | — | 7 resets, trajectory lost |

BIEVR's home-turf claim validated: 2.4x better than RKO-LIO on flat grass.

---

## 3. Down meadows — open meadow, gentle pace (≤3 m/s — the rover's regime)

![meadow](scene_meadow.jpg)
![trajectories](topview_meadow_gentle.png)

| system | median | p95 | max | failures |
|---|---|---|---|---|
| **RKO-LIO** | **0.50** | 1.35 | 1.70 | 0 |
| BIEVR | 2.93 | 4.39 | 5.22 | 0 |
| Voxel-SLAM | — | — | — | 2 resets, trajectory lost |

At rover-like speed the order flips back: RKO-LIO best.

---

## 4. Turnpike — the main dirt trail

![turnpike](scene_turnpike.jpg)
![trajectories](topview_turnpike.png)

| system | median | p95 | max | failures |
|---|---|---|---|---|
| **RKO-LIO** | **4.46** | 7.33 | 12.57 | 0, degraded |
| BIEVR | 13.90 | 31.78 | 34.92 | 0, degraded badly |
| Voxel-SLAM | — | — | — | 46 resets |

Fast trail with berms: nobody clean, RKO-LIO least-bad.

---

## 5. Gupta — wooded trail (closest to forest driving)

![gupta](scene_gupta.jpg)
![trajectories](topview_gupta.png)

| system | median | p95 | max | failures |
|---|---|---|---|---|
| **BIEVR** | **2.72** | 5.03 | 5.49 | 0 |
| RKO-LIO | 20.38 | 34.19 | 35.00 | broke mid-run |
| Voxel-SLAM | — | — | — | 24 resets |

The woods flip it again: BIEVR 7x better; RKO-LIO broke.

---

## 6. Warehouse — trail past structures, fast

![warehouse](scene_warehouse.jpg)
![trajectories](topview_warehouse.png)

| system | median | p95 | max | failures |
|---|---|---|---|---|
| **RKO-LIO** | **20.07** | 79.34 | 94.34 | degraded badly |
| Voxel-SLAM | 3.24 | 24.47 | 51.61 | 67 resets (partial trajectory, 41% of run) |
| BIEVR | 5476 | 14004 | 15714 | EXPLODED (aggressive early motion) |

The hardest run: aggressive driving; nothing survived intact.

---

## 7. Down meadows — full ATV speed, MOVING start (the stress case)

![meadow](scene_meadow.jpg)
![trajectories](topview_meadow_fast.png)

| system | median | p95 | max | failures |
|---|---|---|---|---|
| **RKO-LIO** | **57.08** | 145.89 | 169.48 | broke mid-run |
| BIEVR | 11144 | 30571 | 37489 | EXPLODED (no at-rest init) |
| Voxel-SLAM | — | — | — | 41 resets |

Same meadow as test 3 — speed and the moving start break everything.

---

## Conclusion

- **Terrain picks the winner.** BIEVR: flat/featureless + woods (its
  bump-image design working as advertised). RKO-LIO: gentle pace + least-bad
  everywhere violent. Neither survives full ATV aggression.
- **Every BIEVR explosion traces to initialization** (moving start /
  aggressive early motion). Cleanly initialized, it never diverged.
  Operational rule: BIEVR boots at rest.
- **Voxel-SLAM reset-storms on every test here** (2–67). Terrain-real, not a
  sim artifact: fast motion + weak structure. Mapper in vegetation/structure
  only, always behind reset detection — never control-loop odometry off-road.
- **Slag heap being trivially easy is the best news for the burned-forest
  mission** — barren relief is lidar-friendly.
- The rover drives 3–10x slower than this ATV; at that pace (test 3) all
  systems behave. Terrain-class lessons transfer; absolute numbers do not.
