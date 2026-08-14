# Chapter: Sensor fusion (LIO + IMU + wheel odometry)

Does fusing the rover's sensors beat lidar odometry alone? Filter:
`robot_localization` EKF (the rover's own fusion tool), loosely coupled, 2-D
mode. Sensor suite = exactly the rover's: lidar odometry (RKO-LIO, computed
by us from raw lidar+IMU), IMU gyro, wheel odometry TWIST (speed + yaw rate,
never its integrated pose). All estimates computed on our side from raw bag
streams (no dataset odometry products; wheel = the recorded encoder speed
stream, the rawest wheel data any dataset ships).

**Verdict up front: as configured, fusion did NOT beat LIO alone — an
earlier draft of this page claimed it did, based on a scoring window chosen
after seeing the filter's transient. That was wrong. Corrected analysis
below, with the score-vs-window sensitivity shown openly.**

Variants per environment: LIO alone · LIO+wheel+IMU (EKF) · wheel+IMU
dead-reckoning (EKF, no lidar).

---

## 1. Garden, medium (BotanicGarden 1018-13, 201 m — LIO healthy)

![trajectories](fusion_topview_1018_13.png)

| variant | median | p95 | max |
|---|---|---|---|
| **LIO alone** | **0.30** | 0.71 | 0.76 |
| LIO+wheel+IMU | 0.38 | 0.80 | 0.93 |
| wheel+IMU DR | 12.1 | 21.7 | 21.7 |

Fusion adds a little noise, no benefit. (Post-convergence the two are
0.10 vs 0.11 — indistinguishable.) DR alone drifts 6% of distance.

## 2. Garden, the lidar-killer (BotanicGarden 1006-01, 766 m — LIO degraded)

![trajectories](fusion_topview_1006_01.png)

Median (max) error vs how much of the run's start is excluded from scoring —
shown in full because the answer DEPENDS on it, and that dependence is the
finding:

| scored from | LIO alone | LIO+wheel+IMU | wheel+IMU DR |
|---|---|---|---|
| start | **3.35 (7.2)** | 6.80 (81.9) | 65.8 (199) |
| t+30 s | **3.09 (7.5)** | 4.33 (84.9) | 64.0 (194) |
| t+60 s | 2.97 **(8.0)** | 2.30 (89.1) | 53.4 (193) |
| t+120 s | 2.55 **(7.8)** | 1.90 (24.5) | 52.8 (183) |
| t+180 s | 1.81 (5.9) | 1.60 (4.8) | 52.0 (163) |

Read honestly: the fused filter eventually converges to LIO-comparable
medians, but it spends its first minutes tens of metres wrong and keeps
25-90 m excursions long after. A control-loop odometry is judged by its
worst case: LIO's worst moment is 8 m; the filter's is 90 m. **LIO alone
wins.**

## 3. Fast open meadow (TartanDrive 2, 475 m @ 4-8 m/s — LIO broken)

![meadow](../tartandrive2/scene_meadow.jpg)
![trajectories](fusion_topview_td2_meadow.png)

| variant | median | p95 | max |
|---|---|---|---|
| **wheel+IMU DR** | **25.8** | 42.1 | 44.8 |
| LIO+wheel+IMU | 42.7 | 99.8 | 117.4 |
| LIO alone | 57.1 | 145.9 | 169.5 |

Everything fails; the ordering is still informative: with lidar odometry
confidently wrong, blending it in is worse than ignoring it entirely.

---

## Ground detection (Patchwork++ + Kalman terrain grid)

Same pipeline as the rover, on the TartanDrive gentle meadow (309 m):

![accumulated world map](td2_seg_worldmap.png)
![one scan, top + side](td2_seg_scan.png)
![terrain grid](td2_terrain_grid.png)

Meadow corridor drivable-green, vegetation walls red; the side profile shows
the grass canopy handled as ground. Garden equivalent in the
[BotanicGarden chapter](../botanic/README.md). Gotcha: the segmenter assumes
a gravity-aligned cloud — the ATV's 13-degree-tilted lidar had to be leveled
into the IMU frame first.

## Summary report

1. **LIO alone is the best odometry everywhere it functions.** Fusion as
   configured added noise when LIO was healthy, long transients and 25-90 m
   excursions when LIO was degraded, and could not save the run where LIO
   was broken. The architecture keeps RKO-LIO pure in the control loop.
2. **Wheel+IMU dead-reckoning is a real but limited fallback**: 0.4-6% of
   distance on rover-like courses. Enough to stop safely / limp a short
   distance when lidar dies; not navigation. Worth keeping as a monitored
   FALLBACK (a switch when the LIO alarm fires), NOT as a permanent blend.
3. **Open question, explicitly untested**: whether a carefully tuned or
   alarm-gated filter could do better. The claim "fusion helps when LIO
   degrades" is NOT supported by this data as configured — treat any future
   version of that claim as needing new evidence.
4. **Methodology lesson (recorded so it isn't repeated)**: choosing a
   scoring window after seeing the filter's behavior flipped this page's
   conclusion once. Score windows must be fixed before looking, and figures
   and tables must show the same numbers.
5. Tuning facts that remain true and useful: wheel pose must never be fused
   (only twist); a fast wheel stream out-votes slow LIO deltas unless
   covariances compensate; replay tests need sim time; the EKF has a
   multi-minute convergence transient that must be measured, not hidden.
