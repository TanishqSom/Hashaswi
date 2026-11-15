# GRAPH-UAV RESEARCH PLATFORM

## Complete Project, Installation, Operation and Research Guide

**Project bundle:** Graph-UAV Research\
**Primary simulation:** PX4 SITL + Gazebo Harmonic\
**Robotics middleware:** ROS 2 Jazzy\
**Primary development environment:** Ubuntu 24.04\
**Intended reader:** A person receiving only this SSD and this README\
**Document status:** Handover guide based on the project configuration
and implementation recorded at the time of packaging.

------------------------------------------------------------------------

# 1. What this project is

Graph-UAV is a research and development platform for autonomous
exploration by unmanned aerial vehicles (UAVs) in GPS-denied or
GPS-degraded environments.

The current development system combines:

1.  **PX4 Autopilot SITL** for flight-control simulation.
2.  **Gazebo Harmonic** for the simulated world, vehicle, sensors and
    physics.
3.  **ROS 2 Jazzy** for communication between mapping, graph,
    exploration and vehicle-control software.
4.  **A 2D LiDAR occupancy mapper** for building a representation of
    observed free and occupied space.
5.  **Frontier detection and safe-approach selection** for identifying
    boundaries between explored and unknown space and selecting
    candidate approach locations.
6.  **A topological graph library** (`graph_uav_core_v0_2`) for graph
    construction, analysis, reliability modelling and planning.
7.  **RViz2** for visual inspection of maps, frontiers, graph structures
    and vehicle-related data.
8.  **Experiment logging** for collecting numerical evidence from
    simulation runs.

The research direction is to move from a single UAV mapping pipeline to
a multi-UAV system in which vehicles explore an environment, build and
share topological information, reason about uncertain or blocked routes,
and navigate without using simulator ground-truth pose as an input to
the navigation system.

## 1.1 Current implementation status --- read this carefully

This SSD is a **source and project handover**, not a preinstalled,
turnkey computer image.

The project has working source components and a previously exercised
simulation pipeline. However, the following distinctions matter:

  -----------------------------------------------------------------------
  Capability                          Recorded state at handover
  ----------------------------------- -----------------------------------
  Gazebo research arena               Created

  Custom X500 research model          Created

  2D LiDAR and sensor definitions     Present in the custom model/world

  PX4 SITL integration                Configured; must be rebuilt and
                                      verified on the receiving PC

  ROS 2 PX4 message package           Source is provided separately as
                                      `px4_msgs`

  Gazebo-to-ROS bridge                Previously used successfully

  Occupancy mapping                   Implemented and previously
                                      processed LiDAR scans

  Frontier detection                  Implemented

  Safe approach candidate selection   Implemented

  Topological graph updates           Implemented

  Graph metrics                       Implemented

  CSV experiment logging              Implemented

  RViz map/marker visualization       Configurable; TF and frame
                                      compatibility need checking

  Autonomous frontier-to-flight       Not yet established as a completed,
  controller                          validated component

  Three-UAV coordinated exploration   Research objective; not a completed
                                      validated experiment

  GPS-denied navigation with no       Research objective; not yet
  ground-truth pose leakage           validated
  -----------------------------------------------------------------------

**Do not describe stationary mapping as a completed autonomous
exploration experiment.** A completed exploration experiment must
include actual commanded UAV movement, observed movement, map updates
and recorded measurements.

The earlier system showed approximately 9.85% map coverage during a
stationary mapping run. This is a historical observation, not a
guaranteed startup result or a performance claim.

------------------------------------------------------------------------

# 2. SSD directory layout

The recommended SSD layout is:

``` text
GRAPH_UAV_RESEARCH/
├── README.md
│
├── graph_uav_ws/
│   ├── src/
│   │   └── graph_uav_ros/
│   │       └── graph_uav_mapper/
│   ├── graph_uav_core_v0_2/
│   ├── simulation/
│   │   └── worlds/
│   │       └── permutes_research_arena_v2.sdf
│   ├── experiments/
│   │   └── experiment_01/
│   │       └── single_uav_baseline.csv
│   └── config/
│
├── PX4_CUSTOM/
│   ├── airframes/
│   │   └── 4013_gz_x500_lidar_2d
│   ├── models/
│   │   └── gz_x500_research/
│   └── worlds/
│       └── permutes_research_arena_v2.sdf
│
├── px4_msgs/
│
└── environment.txt
```

The actual SSD may have a slightly different layout. Inspect the files
rather than assuming every optional directory exists.

## 2.1 What each component does

### `graph_uav_ws/`

The main ROS 2 project workspace. It contains the ROS package source,
simulation world, graph library, experiment records and project
configuration.

### `graph_uav_core_v0_2/`

The project's original graph mathematics library. Treat this as the
authority for graph operations. Do not casually rewrite or duplicate its
mathematical implementations in the ROS nodes.

### `src/graph_uav_ros/graph_uav_mapper/`

The ROS 2 package containing the occupancy mapping and frontier-related
nodes.

The previously used executables are:

-   `occupancy_mapper_node`
-   `frontier_node`

### `simulation/worlds/permutes_research_arena_v2.sdf`

The Gazebo research arena. The recorded world is approximately 40 m × 30
m and contains perimeter boundaries, internal dividers, obstacles,
colored zones, junction markers and deployment pads.

### `PX4_CUSTOM/`

Custom PX4 airframe/model/world assets. These must be merged into the
matching paths in the PX4 source tree. They are not a complete PX4
installation.

### `px4_msgs/`

ROS 2 message definitions corresponding to the PX4 1.17 message
interface. The previously used branch was `release/1.17`.

### `experiments/`

CSV and other research records. Preserve original files. Do not
overwrite baseline data when running a new trial.

------------------------------------------------------------------------

# 3. Hardware and software requirements

## 3.1 Recommended operating system

The recorded development environment was:

-   Ubuntu 24.04
-   ROS 2 Jazzy
-   Gazebo Harmonic / Gazebo Sim 8
-   Python 3.12
-   PX4 SITL, based on PX4 v1.17 development source

The source PX4 revision should be read from `environment.txt` if
present. The recorded development revision was:

``` text
v1.17.0-alpha1-1346-gbe631ed584
commit be631ed584
```

Use the exact recorded commit when possible. A different PX4 revision
may change model integration, airframe startup, uXRCE-DDS behavior or
message compatibility.

## 3.2 Hardware

The original development PC had approximately:

-   6 CPU cores
-   16 GB RAM
-   4 GB swap

A receiving PC should ideally have a comparable or better CPU and memory
configuration. A discrete GPU is helpful for Gazebo and RViz but is not
necessarily mandatory for every headless workflow.

Ensure the machine has working graphics drivers and can open Gazebo and
RViz.

## 3.3 Required software components

Install or build the following on the receiving PC:

1.  Ubuntu 24.04.
2.  ROS 2 Jazzy.
3.  Gazebo Harmonic / Gazebo Sim 8.
4.  ROS-Gazebo bridge compatible with Jazzy and Gazebo Harmonic.
5.  PX4 Autopilot source at the recorded revision.
6.  PX4 build dependencies and simulator dependencies.
7.  Micro XRCE-DDS Agent.
8.  `px4_msgs` from the matching release branch.
9.  Python dependencies used by the project.
10. ROS 2 build tools, including `colcon` and `rosdep`.

Do not assume that copying source files installs these dependencies.

------------------------------------------------------------------------

# 4. Prepare the receiving Ubuntu PC

The commands below assume Ubuntu 24.04 and a fresh machine.

## 4.1 Update the OS

``` bash
sudo apt update
sudo apt upgrade
sudo apt install -y git git-lfs curl wget unzip zip rsync build-essential \
  cmake ninja-build python3-pip python3-venv python3-dev \
  python3-colcon-common-extensions python3-rosdep python3-vcstool
```

If `rosdep` has not been initialized on this PC:

``` bash
sudo rosdep init
rosdep update
```

If `sudo rosdep init` says it is already initialized, continue.

## 4.2 Install ROS 2 Jazzy

Follow the official ROS 2 Jazzy installation instructions for Ubuntu
24.04:

https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html

After installation:

``` bash
source /opt/ros/jazzy/setup.bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
```

Check:

``` bash
ros2 --help
```

## 4.3 Install Gazebo Harmonic and ROS-Gazebo integration

Follow the official Gazebo Harmonic installation instructions and
install the ROS-Gazebo bridge package compatible with ROS 2 Jazzy.

The bridge executable used by this project is:

``` bash
ros2 run ros_gz_bridge parameter_bridge
```

Check that it exists:

``` bash
ros2 pkg prefix ros_gz_bridge
```

Check Gazebo:

``` bash
gz sim --versions
```

The project was developed against Gazebo Sim 8 / Harmonic. Do not
silently substitute another Gazebo major version.

## 4.4 Install PX4

Obtain the PX4 source repository and check out the exact commit recorded
in `environment.txt`.

The previously used repository location was:

``` text
~/PX4-Autopilot
```

The recorded commit was:

``` text
be631ed584
```

Example:

``` bash
cd ~
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
cd PX4-Autopilot
git checkout be631ed584
git submodule update --init --recursive
```

If the SSD contains a different exact commit in `environment.txt`, use
that value instead.

Install PX4's Ubuntu dependencies using the setup instructions/scripts
supplied by the checked-out PX4 revision. Then build the Gazebo SITL
target using the PX4 documentation for that revision.

**Important:** The custom airframe and model files must be merged into
this PX4 checkout before building/running the custom simulation.

## 4.5 Merge the custom PX4 assets

The SSD's `PX4_CUSTOM/` directory is an organized handover copy. It may
not use exactly the same relative paths as the original PX4 tree.

Find the matching directories in the PX4 checkout:

``` bash
cd ~/PX4-Autopilot

find ROMFS -type d -path '*airframes*'
find Tools/simulation/gz -maxdepth 3 -type d
```

Copy the custom airframe into the corresponding POSIX airframes
directory, preserving the filename:

``` text
4013_gz_x500_lidar_2d
```

Copy the custom model directory:

``` text
gz_x500_research/
```

into the matching Gazebo models directory used by the checked-out PX4
version.

Copy the custom world to the matching PX4 Gazebo worlds directory if the
PX4 launch configuration expects it there.

Do not overwrite standard PX4 model files. If a destination file already
exists, compare it first:

``` bash
diff -u source_file destination_file
```

The project previously used:

``` bash
export PX4_SYS_AUTOSTART=4013
export PX4_SIM_MODEL=gz_x500_research
export PX4_GZ_WORLD=permutes_research_arena_v2
```

These variables depend on the custom assets being installed in the
locations expected by PX4.

## 4.6 Build and prepare `px4_msgs`

The original message workspace was:

``` text
~/ros2_px4_ws
```

The source package was:

``` text
~/ros2_px4_ws/src/px4_msgs
```

The recorded branch was:

``` text
release/1.17
```

On the receiving PC:

``` bash
mkdir -p ~/ros2_px4_ws/src
cp -a /path/to/SSD/px4_msgs ~/ros2_px4_ws/src/
```

Then:

``` bash
cd ~/ros2_px4_ws
source /opt/ros/jazzy/setup.bash
colcon build --packages-select px4_msgs
```

Verify:

``` bash
source ~/ros2_px4_ws/install/setup.bash
ros2 pkg prefix px4_msgs
ros2 interface show px4_msgs/msg/TrajectorySetpoint
```

The TrajectorySetpoint message uses PX4's NED convention for position,
velocity and acceleration fields. Do not assume ROS ENU coordinates can
be copied directly into PX4 setpoints.

------------------------------------------------------------------------

# 5. Restore the Graph-UAV workspace

Copy the workspace from the SSD to the home directory:

``` bash
cp -a /path/to/SSD/graph_uav_ws ~/graph_uav_ws
```

If the destination already contains a `~/graph_uav_ws`, back it up
before replacing anything.

Install package dependencies:

``` bash
source /opt/ros/jazzy/setup.bash
cd ~/graph_uav_ws
rosdep install --from-paths src --ignore-src -r -y
```

Build the ROS package:

``` bash
cd ~/graph_uav_ws
source /opt/ros/jazzy/setup.bash
colcon build --packages-select graph_uav_mapper --symlink-install
```

The graph library is located at:

``` text
~/graph_uav_ws/graph_uav_core_v0_2
```

The original development setup added it to `PYTHONPATH`:

``` bash
export PYTHONPATH="$HOME/graph_uav_ws/graph_uav_core_v0_2:$PYTHONPATH"
```

Add that line to `~/.bashrc` if the ROS package needs to import the
library:

``` bash
echo 'export PYTHONPATH="$HOME/graph_uav_ws/graph_uav_core_v0_2:$PYTHONPATH"' >> ~/.bashrc
```

Open a new terminal or source the file:

``` bash
source ~/.bashrc
```

If imports fail, inspect the package's `package.xml`, `setup.py`,
`setup.cfg` and Python imports. Do not install arbitrary newer package
versions without checking the project's requirements.

The original environment had a noted compatibility concern: Ubuntu's
packaged NetworkX was 2.8.8 while the original library requirements
indicated NetworkX \>= 3.2. Check the actual requirements bundled on the
SSD and install a compatible version in an isolated environment if
needed.

------------------------------------------------------------------------

# 6. Shell setup used by the project

For ROS terminals, source the environments in this order:

``` bash
source /opt/ros/jazzy/setup.bash
source ~/ros2_px4_ws/install/setup.bash
source ~/graph_uav_ws/install/setup.bash
export PYTHONPATH="$HOME/graph_uav_ws/graph_uav_core_v0_2:$PYTHONPATH"
```

The order matters because the ROS package depends on ROS 2 and PX4
message definitions.

You may add these commands to a helper script or source them manually in
each terminal.

------------------------------------------------------------------------

# 7. Launch the simulation

The following is the previously used startup sequence. Run each
component in a separate terminal. Wait for each process to initialize
before starting the next component.

## Terminal 1 --- Gazebo research arena

``` bash
cd ~/PX4-Autopilot

source build/px4_sitl_default/rootfs/gz_env.sh

export GZ_SIM_SYSTEM_PLUGIN_PATH="/usr/lib/x86_64-linux-gnu/gz-sim-8/plugins:$GZ_SIM_SYSTEM_PLUGIN_PATH"

gz sim -r ~/graph_uav_ws/simulation/worlds/permutes_research_arena_v2.sdf
```

The world should load with the research arena and its static geometry.

## Terminal 2 --- PX4 SITL

``` bash
cd ~/PX4-Autopilot

source build/px4_sitl_default/rootfs/gz_env.sh

export GZ_SIM_SYSTEM_PLUGIN_PATH="/usr/lib/x86_64-linux-gnu/gz-sim-8/plugins:$GZ_SIM_SYSTEM_PLUGIN_PATH"

export PX4_GZ_WORLD=permutes_research_arena_v2
export PX4_SYS_AUTOSTART=4013
export PX4_SIM_MODEL=gz_x500_research
export PX4_GZ_MODEL_POSE="-14,-11.8,1.5,0,0,0"

./build/px4_sitl_default/bin/px4
```

The recorded spawn pose was approximately:

-   X = -14 m
-   Y = -11.8 m
-   Z = 1.5 m

The actual pose must be checked in the running simulator. Do not assume
the model spawned correctly merely because PX4 started.

## Terminal 3 --- Micro XRCE-DDS Agent

``` bash
MicroXRCEAgent udp4 -p 8888
```

This provides the DDS agent endpoint used by the PX4 uXRCE-DDS client.

## Terminal 4 --- Gazebo to ROS 2 bridge

``` bash
source /opt/ros/jazzy/setup.bash

ros2 run ros_gz_bridge parameter_bridge \
'/clock@rosgraph_msgs/msg/Clock[gz.msgs.Clock' \
'/world/permutes_research_arena_v2/model/x500_research_0/link/link/sensor/lidar_2d_v2/scan@sensor_msgs/msg/LaserScan[gz.msgs.LaserScan' \
'/model/x500_research_0/odometry_with_covariance@nav_msgs/msg/Odometry[gz.msgs.OdometryWithCovariance'
```

This bridge command was previously used for:

-   Simulation clock.
-   The custom 2D LiDAR scan.
-   Gazebo odometry with covariance.

The exact Gazebo topic names depend on the loaded model and world. If a
topic does not exist, inspect the actual topics:

``` bash
gz topic -l
```

Do not change the topic string arbitrarily. Match the bridge to the
actual Gazebo topic and message type.

## Terminal 5 --- Occupancy mapper

``` bash
source /opt/ros/jazzy/setup.bash
source ~/graph_uav_ws/install/setup.bash

ros2 run graph_uav_mapper occupancy_mapper_node \
  --ros-args -p use_sim_time:=true
```

The mapper consumes LiDAR and odometry-related data and publishes the
occupancy map and mapping status.

## Terminal 6 --- Frontier and graph node

``` bash
source /opt/ros/jazzy/setup.bash
source ~/graph_uav_ws/install/setup.bash

ros2 run graph_uav_mapper frontier_node \
  --ros-args \
  -p use_sim_time:=true \
  -p uav_id:=uav_01 \
  -p min_cluster_size:=5
```

This node performs frontier processing, safe approach candidate
selection, graph updates, graph metrics and experiment logging,
according to the current package implementation.

## Terminal 7 --- RViz2

``` bash
source /opt/ros/jazzy/setup.bash
source ~/graph_uav_ws/install/setup.bash

rviz2
```

Configure RViz as described in Section 9.

------------------------------------------------------------------------

# 8. Verify each layer before proceeding

Do not assume the system is working because all processes opened. Verify
message flow.

## 8.1 Gazebo topics

``` bash
gz topic -l | grep x500_research_0
```

Previously observed topics included:

``` text
/model/x500_research_0/odometry
/model/x500_research_0/odometry_with_covariance
/model/x500_research_0/pose
```

Sensor topics included:

``` text
/world/permutes_research_arena_v2/model/x500_research_0/link/base_link/sensor/imu_sensor/imu
/world/permutes_research_arena_v2/model/x500_research_0/link/base_link/sensor/air_pressure_sensor/air_pressure
/world/permutes_research_arena_v2/model/x500_research_0/link/base_link/sensor/magnetometer_sensor/magnetometer
/world/permutes_research_arena_v2/model/x500_research_0/link/base_link/sensor/navsat_sensor/navsat
```

The LiDAR topic previously used was:

``` text
/world/permutes_research_arena_v2/model/x500_research_0/link/link/sensor/lidar_2d_v2/scan
```

Topic existence does not prove that messages are being published. Test
message flow with:

``` bash
timeout 5s gz topic -e -t /world/permutes_research_arena_v2/model/x500_research_0/link/base_link/sensor/imu_sensor/imu
```

## 8.2 ROS 2 topic discovery

``` bash
source /opt/ros/jazzy/setup.bash
source ~/ros2_px4_ws/install/setup.bash
source ~/graph_uav_ws/install/setup.bash

ros2 topic list
```

Check the graph and mapper topics:

``` bash
ros2 topic list -t | grep graph_uav
```

Check PX4 topics:

``` bash
ros2 topic list | grep fmu
```

## 8.3 Check LiDAR bridge

``` bash
ros2 topic list -t | grep scan
```

Use the actual bridged scan topic returned by this command:

``` bash
ros2 topic hz /scan
```

If the topic is not `/scan`, replace it with the actual name.

## 8.4 Check occupancy map

``` bash
ros2 topic echo /graph_uav/map --once
```

Check its rate:

``` bash
ros2 topic hz /graph_uav/map
```

## 8.5 Check frontier and graph topics

``` bash
ros2 topic list -t | grep graph_uav
```

Previously used topics included:

``` text
/graph_uav/map
/graph_uav/frontiers
/graph_uav/graph_metrics
/graph_uav/topological_graph/nodes
/graph_uav/topological_graph/edges
/graph_uav/selected_frontier
```

Inspect messages:

``` bash
ros2 topic echo /graph_uav/frontiers --once
ros2 topic echo /graph_uav/graph_metrics
ros2 topic echo /graph_uav/selected_frontier
```

The node and edge visualization topics may be `Marker` or `MarkerArray`.
Always inspect their actual types:

``` bash
ros2 topic type /graph_uav/topological_graph/nodes
ros2 topic type /graph_uav/topological_graph/edges
```

## 8.6 Check PX4 local-position validity

``` bash
ros2 topic echo /fmu/out/vehicle_local_position_v1
```

Also check the message rate:

``` bash
ros2 topic hz /fmu/out/vehicle_local_position_v1
```

The earlier development system reported states including:

``` text
xy_valid=false
z_valid=true
v_xy_valid=false
heading_good_for_control=false
dead_reckoning=true
```

These flags indicate that the PX4 estimator was not ready for normal
horizontal position control at that time.

**Do not arm or command autonomous flight while position/heading
validity is false. Do not bypass arming checks or disable estimator
checks to make the vehicle move.** Diagnose sensor data, timestamps,
bridge connectivity, estimator configuration and frame alignment first.

------------------------------------------------------------------------

# 9. RViz2 visualization setup

The intended RViz view is a top-down view of the arena, occupancy map,
frontier candidates, safe approach points, selected frontier,
topological graph and vehicle position.

## 9.1 Global settings

Set:

  Setting         Value
  --------------- ------------------------
  Fixed Frame     `x500_research_0/odom`
  View Type       `TopDownOrtho`
  Initial Scale   Approximately `35`
  Target X        `0`
  Target Y        `0`

The Fixed Frame must actually exist in the available transform tree or
be compatible with the message frames. The project previously had
incomplete TF availability, so if RViz reports `No transform`, inspect
frame IDs rather than repeatedly changing the Fixed Frame.

## 9.2 Displays to add

Use **Add → By display type**.

  ---------------------------------------------------------------------------------------------------
  Display name            RViz type               Topic
  ----------------------- ----------------------- ---------------------------------------------------
  Occupancy Map           Map                     `/graph_uav/map`

  Exploration Frontiers   MarkerArray             `/graph_uav/frontiers`

  Selected Frontier       Marker                  `/graph_uav/selected_frontier`

  Graph Vertices          Marker or MarkerArray   `/graph_uav/topological_graph/nodes`

  Graph Edges             Marker or MarkerArray   `/graph_uav/topological_graph/edges`

  Safe Approaches         MarkerArray             Find actual safe-approach topic

  UAV Odometry            Odometry                `/model/x500_research_0/odometry_with_covariance`

  Grid                    Grid                    No topic
  ---------------------------------------------------------------------------------------------------

The graph display type must match the actual message type. Check:

``` bash
ros2 topic list -t | grep graph_uav
```

Find safe-approach topic:

``` bash
ros2 topic list -t | grep -E 'safe|approach'
```

## 9.3 Occupancy map

Suggested settings:

-   Topic: `/graph_uav/map`
-   Alpha: approximately `0.8`
-   Color Scheme: `map`
-   Enabled: true

The map uses a recorded resolution of 0.10 m and a nominal 40 m × 30 m
area.

## 9.4 Save RViz configuration

Create a configuration directory:

``` bash
mkdir -p ~/graph_uav_ws/config
```

In RViz, select **File → Save Config As** and save:

``` text
/home/$USER/graph_uav_ws/config/graph_uav_research.rviz
```

Launch it later with:

``` bash
rviz2 -d ~/graph_uav_ws/config/graph_uav_research.rviz
```

If the SSD does not already contain a saved `.rviz` file, the recipient
must configure and save it once.

------------------------------------------------------------------------

# 10. Mapping and graph architecture

## 10.1 Occupancy mapping

The mapper builds a 2D occupancy grid from LiDAR observations.

Recorded map configuration:

  Parameter       Recorded value
  ------------- ----------------
  Width                     40 m
  Height                    30 m
  Resolution              0.10 m
  Grid width           400 cells
  Grid height          300 cells
  Origin          (-20 m, -15 m)

The mapper uses log-odds updates. The recorded values were:

-   Free-space update: -0.4
-   Occupied-space update: +0.85
-   Log-odds clamp: \[-5, +5\]

The occupancy output convention was:

    Value Meaning
  ------- -----------
       -1 Unknown
        0 Free
       50 Uncertain
      100 Occupied

The implementation distinguishes observed cells from unobserved cells.
Range handling includes finite returns, maximum-range clearing, positive
infinity clearing, invalid NaN/negative-infinity rejection and
minimum-range rejection.

## 10.2 Frontier detection

A frontier is a boundary between known free space and unknown space.

The frontier detector uses connected-component/BFS-style clustering. The
recorded configuration used:

-   4/8 connectivity support in the detector implementation.
-   Minimum cluster size of 5.
-   Information gain based on adjacent unknown cells.

Frontiers are candidate exploration regions. A frontier is not
automatically a safe flight target.

## 10.3 Safe approach selection

The safe approach component uses obstacle clearance and reachability
checks. The recorded implementation uses a distance transform and
BFS-style reachability checks, excludes unknown cells from traversal and
prevents diagonal corner-cutting.

A previous implementation could seed reachability from a nearby safe
cell when the robot's grid cell was unknown. This is a
preview/initialization behavior and is not proof that a real vehicle can
travel safely from its exact current position.

## 10.4 Topological graph

The graph represents important locations and traversable connections
rather than every occupancy-grid cell.

The graph is represented as:

\[ G_t=(V_t,E_t,W_t,P_t) \]

where:

-   (V_t): vertices.
-   (E_t): edges.
-   (W_t): edge costs.
-   (P_t): reliability information.

The current frontier node adds graph vertices near selected safe
approach locations and connects nearby vertices when distance and
occupancy-grid corridor checks pass.

Recorded graph IDs used a quantized coordinate scheme similar to:

``` text
uav_01_x<quantized_x>_y<quantized_y>
```

The earlier edge construction was not fully dynamic: edges were added
once and were not always reevaluated after subsequent map changes. Treat
this as a known limitation when interpreting graph metrics.

## 10.5 Graph metrics

The graph analyzer includes operations such as:

-   Connected components.
-   Connectivity.
-   Articulation points.
-   Bridges.
-   Vertex and edge connectivity.
-   Centrality.
-   Graph Laplacian.
-   Algebraic connectivity.

A graph with multiple connected components has algebraic connectivity
(`\lambda`{=tex}\_2=0) under the usual Laplacian convention. This is a
useful diagnostic, not necessarily a software error.

------------------------------------------------------------------------

# 11. Graph library: mathematical functionality

The original library `graph_uav_core_v0_2` is the authoritative
implementation of the graph mathematics.

The following is a conceptual guide to the recorded API. Confirm exact
function signatures against the source files in the SSD before extending
or modifying the library.

## 11.1 Core graph

The graph supports vertices and edges with metadata.

Recorded usage patterns:

``` python
TopologicalGraph.add_vertex(vertex, **attrs)

TopologicalGraph.add_edge(
    u,
    v,
    cost,
    reliability_by_state=None,
    available=True,
    validated=False,
    **metadata
)
```

Blocked edges can be retained in the graph while marked unavailable:

``` python
set_obstruction(u, v, obstructed)
```

Retaining blocked edges preserves the history/identity of a route while
making it unavailable for planning.

## 11.2 Environment-conditioned reliability

The model represents environment states (z_j), probabilities (q_j), and
edge success probabilities (p_e(z_j)).

Unconditional edge success probability:

\[ P(S_e)=`\sum`{=tex}\_j q_j p_e(z_j) \]

For a path (`\pi`{=tex}), under conditional independence of edge
successes given a shared environment state:

\[ Rel(`\pi`{=tex})=`\sum`{=tex}*j
q_j`\prod`{=tex}*{e`\in`{=tex}`\pi`{=tex}}p_e(z_j) \]

This shared-environment formulation matters. Multiplying unconditional
edge reliabilities can be incorrect when edges share environmental
conditions.

The environment probabilities must be supported by data or an explicit
experimental assumption. Do not invent values and present them as
measured probabilities.

## 11.3 Traversal success and localization

The recorded empirical traversal success definition uses the conjunction
of:

-   Edge clear.
-   Vehicle arrived.
-   Localization error remained within a specified tolerance.

The localization chain rule is an exact identity:

\[ P(A_e`\cap `{=tex}L_e`\mid `{=tex}Z) =
P(A_e`\mid `{=tex}Z)P(L_e`\mid `{=tex}A_e,Z) \]

This does not require an independence assumption.

## 11.4 Reliability estimation

The library includes a trial-based reliability estimator. The recorded
API included:

``` python
ReliabilityEstimator.estimate_from_trials(
    trials,
    environment_state,
    confidence=0.95
)
```

and:

``` python
ReliabilityEstimator.update_graph_edge(...)
```

The trial structure was recorded as:

``` python
TraversalTrial(
    environment_state,
    edge_clear,
    arrived,
    localization_errors_m,
    max_localization_error_m
)
```

The estimator uses empirical trials and a Wilson confidence interval.
Blocked trials are excluded from conditional traversal-success
estimation. An edge can be treated as validated when its lower
confidence bound meets the chosen minimum reliability threshold.

The threshold must be selected and justified by the experiment. Do not
label an edge validated merely because a point estimate is high.

## 11.5 Reliability-constrained planning

The recorded planner API was:

``` python
GraphPlanner.reliability_constrained_shortest_path(
    source,
    target,
    minimum_reliability,
    max_candidates=10000,
    risk_weight=0
)
```

The planner searches for a low-cost path subject to a reliability
constraint. Candidate ordering uses a score involving edge cost and a
risk term based on (-`\log `{=tex}`\bar `{=tex}p_e), followed by an
exact shared-environment reliability check.

A bounded candidate search has an important limitation: failure to find
a path within the candidate cap does not prove that no feasible path
exists.

## 11.6 Graph exploration

The recorded `GraphExplorer` functionality includes:

-   Frontier vertices on the graph.
-   Information-gain score.
-   Edge coverage route.

Information gain score:

\[ score=`\frac{information\_gain}{\max(travel\_cost,10^{-9})}`{=tex} \]

The recorded API included:

``` python
GraphExplorer.information_gain_score(info_gain, travel_cost)

GraphExplorer.frontier_vertices(
    known_vertices,
    candidate_frontiers
)
```

The edge coverage routine uses an unweighted CPP-style/Eulerization
approach and requires a connected graph. Do not describe it as a
weighted Chinese Postman Problem solver.

## 11.7 Features not to claim as implemented

The following were explicitly not part of the original library
implementation at the recorded handover point:

-   A general robustness index.
-   Hamiltonian optimization.
-   Yen's algorithm with disjoint-path guarantees.
-   Markov localization-drift model.
-   Bayesian updating of reliability.
-   Weighted Chinese Postman Problem.
-   Globally optimal reliability-constrained planner.

These may be future research extensions, but must not be presented as
completed code.

------------------------------------------------------------------------

# 12. Experiment programme

The planned research programme consists of seven experiments. The system
should collect logs and evidence for each experiment.

  -------------------------------------------------------------------------
                           No. Experiment            Main objective
  ---------------------------- --------------------- ----------------------
                             1 Single-UAV mapping    Demonstrate mapping
                               and exploration       while a UAV actually
                                                     moves to explore
                                                     frontiers

                             2 Multi-UAV frontier    Allocate exploration
                               allocation            targets among multiple
                                                     UAVs

                             3 Shared topological    Build and compare a
                               graph construction    shared graph from
                                                     multiple UAV
                                                     observations

                             4 Communication loss    Study graph
                               and graph             consistency and
                               synchronization       synchronization after
                                                     communication
                                                     interruptions

                             5 Edge reliability      Estimate traversal
                               estimation            success and confidence
                                                     from repeated trials

                             6 Reliability-aware     Compare route cost and
                               path planning         reliability under
                                                     constraints

                             7 GPS-denied navigation Navigate using
                               without ground-truth  permitted
                               pose leakage          onboard/localization
                                                     inputs while ground
                                                     truth is reserved for
                                                     evaluation
  -------------------------------------------------------------------------

## 12.1 Experiment 1 --- single-UAV mapping and exploration

A valid E1 run should include:

1.  Initial vehicle state and starting pose.
2.  LiDAR mapping.
3.  Frontier detection.
4.  Safe approach selection.
5.  A navigation command to a selected target.
6.  Actual vehicle movement.
7.  Observed arrival or failure.
8.  Updated map and graph.
9.  CSV logging.
10. A repeatable run configuration.

A stationary mapping session is useful for checking the sensor and
mapper pipeline, but is not a completed autonomous exploration trial.

## 12.2 Experiment 2 --- multi-UAV frontier allocation

The intended extension is to spawn multiple vehicles, maintain per-UAV
state and allocate candidate frontiers without assigning the same target
unnecessarily.

Measure allocation quality, travel cost, duplicate exploration, coverage
and time to completion.

## 12.3 Experiment 3 --- shared graph

Combine local graph observations into a shared graph. Evaluate graph
overlap, duplicate vertices, edge consistency, connected components and
graph growth.

## 12.4 Experiment 4 --- communication loss

Introduce controlled communication outages. Record local graph changes,
synchronization delay, conflicts and consistency after reconnection.

## 12.5 Experiment 5 --- edge reliability

Perform repeated traversal trials in defined environment conditions.
Record edge-clear state, arrival, localization error, trial outcome and
confidence interval.

Do not fabricate environment-state probabilities or success rates.

## 12.6 Experiment 6 --- reliability-aware planning

Compare shortest-cost routes with routes satisfying a minimum
reliability threshold. Record path length/cost, computed reliability,
selected route, rejected candidates and search limits.

## 12.7 Experiment 7 --- GPS-denied navigation

Ground-truth pose may be used as an evaluation reference, but must not
leak into the mapping, localization or navigation decision pipeline.

Record the pose source for every relevant node. Separate:

-   Navigation estimate.
-   Simulator ground-truth reference.
-   Evaluation error.

The earlier mapper reported `pose_source=simulator_ground_truth`. This
must be changed or isolated before claiming Experiment 7.

------------------------------------------------------------------------

# 13. Experiment logging

The recorded CSV path was:

``` text
~/graph_uav_ws/experiments/experiment_01/single_uav_baseline.csv
```

The recorded columns included:

``` text
simulation_time_s
elapsed_time_s
uav_id
coverage_pct
frontier_count
safe_approach_count
selected_x
selected_y
information_gain
travel_cost_m
exploration_score
cumulative_distance_m
vertices
edges
components
```

Inspect the log:

``` bash
tail -20 ~/graph_uav_ws/experiments/experiment_01/single_uav_baseline.csv
```

Watch live:

``` bash
tail -f ~/graph_uav_ws/experiments/experiment_01/single_uav_baseline.csv
```

The earlier stationary run showed cumulative distance of zero. This is
consistent with no vehicle movement and must not be presented as
autonomous exploration.

The earlier `safe_approach_count` was very large (over 11,000 in a
sample). Verify whether the field records all safe grid cells rather
than a count of distinct approach candidates before using it in a paper.

Keep a separate CSV for each independent run. Record configuration,
seed, software commit, world, UAV count and experiment conditions.

------------------------------------------------------------------------

# 14. Autonomous flight architecture --- planned next stage

The intended control architecture separates high-level exploration from
low-level flight control.

### High-level ROS 2 exploration system

Responsible for:

-   Occupancy map.
-   Frontier detection.
-   Safe approach selection.
-   Graph updates.
-   Graph-based route planning.
-   Target selection.
-   Research logging.

### PX4 flight controller

Responsible for:

-   Attitude and rate control.
-   Position/velocity control.
-   Motor control.
-   Flight modes.
-   Failsafes.

### Planned navigation controller

A separate ROS 2 node is intended to:

1.  Subscribe to the selected frontier.
2.  Check target validity and safety.
3.  Convert map coordinates to PX4 local NED coordinates.
4.  Publish Offboard heartbeat continuously.
5.  Publish trajectory setpoints.
6.  Monitor vehicle state and arrival.
7.  Handle timeout, hold and return/land behavior.
8.  Log commanded and observed positions.

This controller must not be assumed to exist merely because the mapper
and frontier node run. Inspect the source tree for the actual controller
implementation before trying to launch it.

PX4 Offboard control requires a continuing setpoint/heartbeat stream and
valid estimator state. Use the PX4 documentation for the exact
checked-out version.

**Safety:** This is simulation research. Do not transfer commands to a
physical aircraft without a separate hardware safety review, geofence,
failsafe plan, manual override, controlled test area and validation.

------------------------------------------------------------------------

# 15. Troubleshooting

## 15.1 `ros2: command not found`

Source ROS:

``` bash
source /opt/ros/jazzy/setup.bash
```

## 15.2 `Package 'graph_uav_mapper' not found`

Build and source the workspace:

``` bash
cd ~/graph_uav_ws
source /opt/ros/jazzy/setup.bash
colcon build --packages-select graph_uav_mapper --symlink-install
source install/setup.bash
```

## 15.3 `ModuleNotFoundError` for graph library

``` bash
export PYTHONPATH="$HOME/graph_uav_ws/graph_uav_core_v0_2:$PYTHONPATH"
```

Check whether the module exists in the library and verify its import
name.

## 15.4 `px4_msgs` not found

``` bash
source /opt/ros/jazzy/setup.bash
source ~/ros2_px4_ws/install/setup.bash
ros2 pkg prefix px4_msgs
```

If missing, build the `px4_msgs` workspace.

## 15.5 Gazebo topic exists but no messages

A listed topic is not proof of active publication.

Check:

``` bash
gz topic -l
timeout 5s gz topic -e -t ACTUAL_TOPIC_NAME
```

Check whether Gazebo is running and unpaused. Confirm the topic belongs
to the expected world/model instance.

## 15.6 ROS topic exists but no messages

Check the publisher and rate:

``` bash
ros2 topic info -v ACTUAL_TOPIC
ros2 topic hz ACTUAL_TOPIC
```

Confirm that the Gazebo bridge is still running and its Gazebo message
type matches the actual topic type.

## 15.7 PX4 DDS topics missing

Check:

-   PX4 is running.
-   `MicroXRCEAgent udp4 -p 8888` is running.
-   PX4 uXRCE-DDS client is configured for the correct agent IP/port.
-   `px4_msgs` matches the PX4 source version.
-   ROS terminals have sourced `px4_msgs`.

## 15.8 PX4 estimator says `xy_valid=false`

Do not arm. Inspect actual IMU, magnetometer, barometer and other
required sensor messages, timestamps, PX4/Gazebo sensor integration,
timesync and estimator status. Do not disable estimator checks as a
workaround.

## 15.9 RViz `No transform`

Inspect message frames:

``` bash
ros2 topic echo /model/x500_research_0/odometry_with_covariance --once
ros2 run tf2_tools view_frames
```

The frame selected as Fixed Frame must be connected to the displayed
message frame through TF, or the display must use a compatible frame.

## 15.10 Graph shows disconnected components

This may be a real property of the current graph or a consequence of
incomplete graph edge creation. Check:

-   Map connectivity.
-   Occupancy corridor test.
-   Edge availability.
-   Graph vertex positions.
-   Whether edges are being reevaluated after map updates.

Do not force graph connectivity simply to make the visualization look
connected.

------------------------------------------------------------------------

# 16. Reproducibility and research integrity

For every experiment, preserve:

-   PX4 commit.
-   ROS package source revision.
-   Graph library version/source.
-   Gazebo world file.
-   Custom model and airframe.
-   ROS parameters.
-   PX4 parameters.
-   UAV spawn positions.
-   Environment configuration.
-   Random seeds, if used.
-   Start/end timestamps.
-   CSV logs.
-   ROS bag or other raw sensor logs, if collected.
-   Screenshots or RViz recordings.
-   Failure records.

Never overwrite the original baseline CSV. Create a new run file, for
example:

``` text
experiment_01_run_001.csv
experiment_01_run_002.csv
```

Record failed trials too. A failed traversal is evidence and may be
necessary for reliability estimation.

Clearly distinguish:

-   Implemented.
-   Executed.
-   Tested.
-   Validated.
-   Not implemented.
-   Planned.

A function passing a unit test does not prove that the full UAV system
works in simulation. A simulation result does not prove physical flight
performance.

------------------------------------------------------------------------

# 17. Useful official documentation

Use documentation corresponding to the installed software versions.

-   ROS 2 Jazzy: https://docs.ros.org/en/jazzy/
-   ROS 2 Jazzy installation:
    https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html
-   Gazebo documentation: https://gazebosim.org/docs/
-   PX4 documentation: https://docs.px4.io/
-   PX4 ROS 2 integration: https://docs.px4.io/main/en/ros2/
-   RViz2: https://docs.ros.org/en/jazzy/p/rviz2/

When documentation for `main` differs from the installed PX4 revision,
prefer the documentation/source matching the checked-out revision.

------------------------------------------------------------------------

# 18. First-run checklist for the receiving researcher

Follow this order. Do not start by trying to fly the UAV.

-   [ ] Confirm Ubuntu 24.04.
-   [ ] Install ROS 2 Jazzy.
-   [ ] Install Gazebo Harmonic and `ros_gz_bridge`.
-   [ ] Install PX4 prerequisites.
-   [ ] Check out the recorded PX4 commit.
-   [ ] Merge custom PX4 airframe/model/world files.
-   [ ] Build PX4 SITL.
-   [ ] Build `px4_msgs` release/1.17.
-   [ ] Copy `graph_uav_ws` into the home directory.
-   [ ] Install ROS package dependencies.
-   [ ] Set the graph library `PYTHONPATH`.
-   [ ] Build `graph_uav_mapper`.
-   [ ] Start Gazebo.
-   [ ] Start PX4.
-   [ ] Start Micro XRCE-DDS Agent.
-   [ ] Start the Gazebo ROS bridge.
-   [ ] Start occupancy mapper.
-   [ ] Start frontier/graph node.
-   [ ] Verify map messages.
-   [ ] Verify frontier messages.
-   [ ] Verify graph messages and metrics.
-   [ ] Open RViz and configure displays.
-   [ ] Save RViz configuration.
-   [ ] Verify PX4 sensor data and estimator validity.
-   [ ] Only then begin controlled navigation-controller development.

------------------------------------------------------------------------

# 19. Project handover summary

The core contribution of this project is the integration of
perception-derived mapping with a topological graph representation
intended for autonomous UAV exploration and reliability-aware planning.

The occupancy grid captures local spatial evidence. Frontier detection
identifies unexplored boundaries. Safe approach selection filters
candidate locations using clearance and reachability checks. The graph
library represents navigable structure and provides graph analysis,
reliability estimation and constrained planning functions. PX4 is
intended to execute flight commands, while ROS 2 coordinates the
research pipeline and Gazebo provides the simulated environment.

The project is deliberately structured so that graph mathematics,
mapping, exploration logic and flight control remain distinct
components. This makes it possible to test each layer independently and
collect evidence for the seven-experiment programme.

**The receiving researcher should first reproduce the mapping and graph
pipeline, then validate estimator and coordinate frames, and only then
implement and test autonomous movement.** Multi-UAV coordination and
GPS-denied navigation are subsequent research milestones, not
capabilities to assume from a successful startup.

------------------------------------------------------------------------

**End of README**
