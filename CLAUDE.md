# custos-control — CLAUDE.md

Pixhawk bridge and mission management for the Custos drone stack.

Workspace-wide rules and state caveats live at `../CLAUDE.md` (= `/space/drone/CLAUDE.md`). If you are in a standalone clone, the one rule below is the one most likely to be violated when working in this repo.

## The one rule

**The Pixhawk is the only hardware surface owned here; everything else reads/writes `custos_control_msgs`.** If you reach into another repo to control flight (or have another repo reach into yours for a Pixhawk action), the boundary is wrong — model the action as a `custos_control_msgs` service or action instead. Mission management consumes `custos_navigation_msgs` (planned trajectories) and emits `custos_control_msgs`; it does not depend directly on the navigation planner package.

## State of this repo

- Single package: `custos_control_pixhawk` (`package.xml`, `CMakeLists.txt`, empty `src/`). No real bridge implementation yet.
- **Choice still open:** uXRCE-DDS vs MAVROS as the Pixhawk transport. The package skeleton declares neither dependency; pick one before adding code and capture the decision as an ADR.
- **Brace-expansion litter:** there is a literal directory named `{.github` alongside the real `.github/`. Do not stage it.

## Cross-repo edges

- **Depends on:** `rclcpp`, `rclcpp_lifecycle`, `lifecycle_msgs`, `custos_common_msgs`, `custos_control_msgs`.
- **Depended on by (runtime, by topic):** the planner in `custos-navigation` emits trajectories that this repo consumes; mission state and command acknowledgements flow back as `custos_control_msgs`.
- **Hardware:** Pixhawk autopilot via uXRCE-DDS or MAVROS (TBD).

## Repo-specific hard rules

- **No direct dependency on `custos-navigation`.** All cross-repo flight intent flows through `custos_navigation_msgs` and `custos_control_msgs` topics. The planner package and the bridge package do not link against each other.
- **Mission management lives here, not in navigation.** Mission state machines, flight modes, safety interlocks, and pre-flight checks are control concerns — they own the Pixhawk. Navigation is path-shaped; control is intent-shaped.
- **Choose the Pixhawk transport intentionally.** uXRCE-DDS keeps everything in ROS2 idioms; MAVROS bridges to the larger MAVLink ecosystem. Either is reasonable; pick once and document with an ADR. Mixing both at the same layer is the failure mode.
- **No NOVATEK dependency, ever.** Even though the control stack runs alongside the wrapper on real hardware, control code never imports wrapper headers (ADR 0010).

## Build / test cheat sheet

```bash
colcon build --packages-select custos_control_pixhawk
colcon test --packages-select custos_control_pixhawk

# Run against simulated Pixhawk SITL or hardware.
ros2 launch custos_control_pixhawk pixhawk.launch.py   # TODO: launch file unwritten
```

## Pointers specific to this repo

- Wire types: `custos-interfaces/custos_control_msgs/`
- Cross-repo CI policy: ADR 0009
- Mission/planner boundary: planning record §"Repo list" (control owns mission; navigation owns path)

> TODO(post-first-commit): delete the `{.github` litter directory before the initial commit.
> TODO(post-active): write an ADR locking uXRCE-DDS vs MAVROS, then implement the bridge.
> TODO(post-active): document Pixhawk firmware version compatibility once tested against real hardware.
