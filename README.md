# Rover sim results — ditch & terrain A/B test (2026-08-01)

## TL;DR

- **Without the terrain node:** the planner drew a path straight across the
  0.5 m trench — a hole returns no lidar points, so the costmap sees nothing
  there. The rover never even reached it: it froze halfway up the 13° ramp,
  because ground points that the segmenter mislabels on slopes looked like a
  wall to the collision monitor.
- **With the terrain node:** the rover climbed the ramp, the trench showed up
  in the costmap as a solid no-go wall from 6 m away, and nav2 never drove
  across it. Goals at 12 m and 17 m (1.3 m before the edge) both succeeded.
- Commits in the rover repo: `eb2d278`, `f5c9ef3`, `7639395`.

## 1 · The test course

A gentle 13° ramp, a 0.5 m trench crossing the whole map at x≈19, and a
40–50° bank at x≈30. Sparse trees, start pad at the origin.

![course](figures/terrain_trial_course.png)

## 2 · Baseline — blind to the ditch, stuck on the ramp

The blue plan goes straight over the trench. Orange is what the rover actually
did: froze at x=8.3 on the ramp (slope points leaking through ground
segmentation sat in the collision monitor's height band and read as an
obstacle dead ahead).

![baseline](figures/terrain_ab_baseline.png)

## 3 · Terrain node on — ditch marked, never crossed

The dark red band is the trench as nav2 now sees it. Goals at 12 m and 17 m
passed; for the goal beyond the trench the rover searched along the wall but
never drove over the edge (max x = 17.2 vs lip at 18.3).

![terrain](figures/terrain_ab_terrain.png)

## 4 · Local costmap, side by side

Top: baseline 6×6 m window — empty ahead, nothing to stop for. Bottom:
terrain mode's 12×12 m window with the trench wall solidly marked.

![localcost](figures/terrain_ab_localcost.png)

## The "latched ditch" fix, in plain words

The node flags a ditch when it sees ground cells right next to cells it has
**never** gotten a lidar return from — a hole gives you nothing back, so
"known ground next to a blind spot" is the signature of a ditch. To avoid
false alarms, that check only ran in a band 3–8 m from the robot.

**The bug:** that band moves *with the robot*. Drive toward the trench and the
trench leaves the band — the node re-checked, the rule no longer applied, and
it erased its own warning right as the rover got close. Up close you can't
re-detect it either: the trench's near wall hides the floor from the lidar,
so you still get no returns — no new information at all. Version 1 drove
straight into the trench because of this.

**The fix:** once a cell is flagged as ditch, it stays flagged ("latched") as
long as those neighboring cells are still blind spots. The flag clears only
when actual returns arrive from there and show the ground is fine.
Warnings expire on evidence, not on distance.
