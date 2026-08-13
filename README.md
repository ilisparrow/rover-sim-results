# Rover evaluation results

Chapters — one per dataset, each with: what it looks like, estimated
trajectories vs ground truth, results table, conclusion.

| chapter | terrain | truth quality |
|---|---|---|
| [BotanicGarden](figures/botanic/README.md) | real garden vegetation (rover's exact sensors) | survey-grade, cm |
| [TartanDrive 2](figures/tartandrive2/BENCHMARK.md) | fast open dirt, meadows, trails (Livox Mid-70) | RTK-fused GPS, dm |
| [Simulation campaigns](figures/) | Gazebo/MARSIM terrain + ditch + odometry tests | exact sim truth |

Cross-dataset verdict so far: RKO-LIO = control-loop odometry everywhere
(never jumps, 10× cheaper); Voxel-SLAM = mapper + repositioning where
vegetation/structure exists (reset-storms on fast featureless ground);
BIEVR = strong second in vegetation, needs at-rest boot; wheel-odom EKF =
the open-ground insurance.
