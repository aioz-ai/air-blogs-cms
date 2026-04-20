---
last_modified_on: "2026-03-28"
title: AeroScene: Progressive Scene Synthesis for Aerial Robotics
description: Scene Generation, Aerial Robotics, Diffusion model
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


In the previous section, we observed that AeroScene surpasses other approaches in the scene generation task. We now delve into the details of the AeroScene dataset and examine its applications in navigation and interaction tasks for aerial robotics.




## AeroScene Dataset Statistic and Labels
Using our proposed method, we create the AeroScene dataset with more than 1000 scenes, which are specifically curated to emphasize multi-scale hierarchical structures in indoor and outdoor environments. All scenes are embedded into NVIDIA Isaac Sim to facilitate downstream drone-related tasks. Scene creation began with initializing empty indoor (e.g., rooms, offices) and outdoor (e.g., parks, urban areas) settings, where base layouts were formed by importing pre-built assets such as walls, floors, and terrain. Objects were then placed in a hierarchical manner: coarse-scale structural elements like buildings, walls, or terrains were positioned first to define the overall structure, followed by fine-scale details such as utensils, decorations, or debris, which were arranged relative to the larger elements to preserve logical groupings and spatial relationships. Finally, all objects were annotated with bounding boxes, orientations, scales, and semantic categories, while hierarchies explicitly linked fine objects to their coarse parents. In addition, interaction areas suitable for drone landing were annotated to support downstream robotics tasks. Specifically, all scenes are stored within a consistent schema: each object has $\texttt{id}$, $\texttt{scene\_id}$, $\texttt{parent\_id}$, $\texttt{category\_id}\,\in\{1,\dots,C\}, \texttt{bbox}$ given as $(\mathbf{p},\mathbf{q},\mathbf{s})$ where $\mathbf{p}\in\mathbb{R}^3$ (center), $\mathbf{q}\in\mathbb{R}^4$ (quaternion), $\mathbf{s}\in\mathbb{R}^3$ (local extents), plus precomputed $\texttt{bbox\_corners}$ for fast IoU tests, a $\texttt{domain}$ flag (indoor/outdoor), and optional $\texttt{attributes}$; interaction areas (e.g., landing zones) are stored as polygonal regions with centroid and radius. The dataset statistical information can be found in Table 3.


<style>
      table {
    margin-left: auto;
    margin-right: auto;
    /* Optional: Set a specific width so there is available space to center within its container */
    /* width: 80%; */ 
  }
  .center-content th,
  .center-content td {
    text-align: center; /* Centers text horizontally */
    vertical-align: middle; /* Centers text vertically */
  }
  .bold-text {
    font-weight: bold; /* or use a numeric value like 700 */
  }
  .italic-text {
    font-style: italic;
  table td:nth-child(3) {
    text-align: center;
}
  }
</style>
<table>
    <tr>
        <td class=bold-text>Criteria</td>
        <td class=bold-text align="center">Train</td>
        <td class=bold-text align="center">Test</td>
        <td class=bold-text align="center">Total</td>
    </tr>
    <tr>
        <td class=italic-text>#Scenes</td>
        <td align="center">812</td>
        <td align="center">204</td>
        <td align="center">1016</td>
    </tr>
    <tr>
        <td class=italic-text>#Objects</td>
        <td align="center">122,356</td>
        <td align="center">37,654</td>
        <td align="center">160,010</td>
    </tr>
    <tr>
        <td class=italic-text>Avg. objects/scene</td>
        <td align="center">149</td>
        <td align="center">152</td>
        <td align="center">149</td>
    </tr>
    <tr>
        <td class=italic-text>Avg. objs / coarse-scale level</td>
        <td align="center">35</td>
        <td align="center">32</td>
        <td align="center">34</td>
    </tr>
    <tr>
        <td class=italic-text>Avg. objs / fine-scale level</td>
        <td align="center">111</td>
        <td align="center">114</td>
        <td align="center">112</td>
    </tr>
    <tr>
        <td class=italic-text>Avg. Landing Areas for Drones</td>
        <td align="center">55</td>
        <td align="center">53</td>
        <td align="center">54</td>
    </tr>
    <tr>
        <td>Small Drone (Avg.)</td>
        <td align="center">42</td>
        <td align="center">43</td>
        <td align="center">42</td>
    </tr>
    <tr>
        <td>Medium Drone(Avg.)</td>
        <td align="center">11</td>
        <td align="center">13</td>
        <td align="center">12</td>
    </tr>
    <tr>
        <td>Large Drone(Avg.)</td>
        <td align="center">4</td>
        <td align="center">4</td>
        <td align="center">5</td>
    </tr>
    <tr>
        <td class=italic-text>#Categories (coarse-scale)</td>
        <td class="bold-text" align="center" colspan="3">23</td>
    </tr>
    </tr>
    <tr>
        <td class=italic-text>#Categories (fine-scale)</td>
        <td class="bold-text" align="center" colspan="3">47</td>
    </tr>
</table>
<p align="center">
    <b>Table 3:</b>
    <i> Statistics of AeroScene dataset</i> 
</p>

## AeroScene for Navigation and Interaction Tasks

To demonstrate the practical use of the AeroScene dataset, we define a unified aerial robotics task that combines *long-range navigation* and *close-range physical interaction* within a single scene. In this setup, an aerial robot starts at a designated location and must autonomously navigate to a *pre-annotated interaction area* before performing a controlled landing or perching maneuver. This task demonstrates how AeroScene’s realistic and richly annotated environments can evaluate both high-level planning and fine-grained physical interaction capabilities under diverse conditions.

### Task Design  
The drone task is divided into two sequential phases: 
- *Navigation Phase*: The drone uses global semantic information and dynamics constraints to plan a trajectory to the target area. A geometric controller executes this path, ensuring stable navigation through complex environments. 
- *Interaction Phase*: Once near the target, the drone switches to local sensing. Point-cloud data is analyzed to identify surface normals, slope, and clearance, allowing the system to select a safe landing or perching zone. The drone then performs a precise descent and touchdown.

### Example Scenario 
Figure 6 illustrates a representative mission: navigating a generated urban-style scene to land on the red roof of the Weefit building in the scene. In this example, the environment contains detailed objects and varying elevations, requiring the drone to plan a path and then transition to close-proximity perception for accurate landing. This scenario, combined with AeroScene’s dataset generation pipeline, demonstrates the usefulness of our work in benchmarking aerial robotics algorithms in perception, mapping, trajectory planning, and interaction control.

### Scene Utility and Data Generation
AeroScene’s hierarchical object labeling, annotated landing zones, and physics-aware surfaces create challenging, realistic environments for aerial autonomy and manipulation research. Each trial not only evaluates planning and control strategies but also generates a rich dataset for future development. For every trajectory, we record:

- **Visual Data:** RGB and depth streams from simulated onboard cameras.
- **State Estimates:** Ground-truth absolute position, orientation, and velocity.
- **Inertial Measurements:** IMU sensor readings for accelerations and angular rates.
- **Control Signals:** Low-level motor or actuator commands used during trajectory execution.
- **Planned and Executed Trajectories:** Waypoints and actual flight paths for benchmarking performance.

In total, we record each scene 300 planned trajectories for each scene. Small- and medium-sized drone platforms, i.e., 3DR Iris and AscTec Hummingbird, were used to test the unified navigation and interaction pipeline. Across all trials, the system achieved an overall success rate of 91\%, demonstrating the utility of AeroScene as a challenging yet tractable benchmark for aerial robotics research. Details are summarized in Table 4.

<table>
    <tr>
        <td class="bold-text">Metric</td>
        <td class="bold-text">Value</td>
    </tr>
    <tr>
        <td>Drone Platforms</td>
        <td>2</td>
    </tr>
    <tr>
        <td>Trajectories per Scene</td>
        <td>300</td>
    </tr>
    <tr>
        <td>Overall Success Rate</td>
        <td class="bold-text">91%</td>
    </tr>
</table>
<p align="center">
    <b>Table 4:</b>
    <i> Evaluation summary of navigation and interaction tasks on the AeroScene dataset.</i> 
</p>


![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1774625586/dronetask_cup6sa.png)
*<center>**Figure 6**:  Generated navigation and interaction trajectories for the example mission: landing on the red roof of the Weefit building. Green lines represent feasible navigation and interaction trajectories, and red lines denote failed attempts. Sampled point clouds are displayed within the blue box, with red dots indicating failure landing points.</center>*


## Limitations 
The proposed AeroScene, while effective in demonstrating hierarchical scene synthesis, has certain limitations. Current experiments are mainly conducted in simulation, which may not fully represent the diversity and complexity of real-world aerial environments (such as wind conditions). In addition, the framework focuses on static scene layouts, without explicitly modeling temporal dynamics or handling uncertainty from dynamic environments. Therefore, an interesting future work is to extend the synthesis process to generate dynamic elements. Furthermore, performing sim-to-real validation on real aerial robots would be an interesting direction for future work.

## Conclusion 
We introduced **AeroScene**, a hierarchical diffusion framework for 3D scene synthesis that combines hierarchy-scale tokenization, multi-branch feature extraction, and a cross-scale attention with gradient-based guidance objectives. Our approach enables structured reasoning across global layouts and local details while enforcing physical and semantic plausibility. Using our method, we generate a large-scale benchmark of 3D environments for drone interaction. 

We hope this blog provides you with valuable insights into scene generation for drones.
</CodeExplanation>