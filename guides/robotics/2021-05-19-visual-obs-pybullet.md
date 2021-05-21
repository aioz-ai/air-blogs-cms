---
last_modified_on: "2021-05-19"
title: Visual observations for pybullet agents.
description: Obtaining visual observations for agents in pybullet.
series_position: 1
author_github: https://github.com/trqminh
tags: ["type: tutorial", "level: medium"]
---

Machine learning has been developing significantly, with the approaches to the problem have changed compared to the early days. Indeed, there is a transition from predictive ML to interactive ML or from datasets to simulation systems. The community expects that the simulations can create more comprehensive and general observations; thereby, AGI (Artificial General Intelligence), the ultimate task in Artificial Intelligence, can be solved in the near future. Recently, in 2020, Bullet SDK was rolled out, a physics engine that simulates collision detection and soft and rigid body dynamics. This simulation has an agility to customize a various kinds of task for AI agents to learn. At AIOZ, we are using Pybullet as a simulation environment for robot navigation. Previously, we used Gazebo for this problem, but it has certain limitations. The first is the demanding requirement of computer hardware, and the second is the ability to launch multiple robots at one time. Meanwhile, Pybullet is both lightweight and provides a non-GUI mode that speeds up the training process. Plus, it creates a scalable number of agents for each training.

Nevertheless, there was also a difficulty when we approached using Pybullet for robot navigation. That's how to attach the camera on the agent. Or that is to say, how to obtain visual observations from the agent's perspective. For example, let's load [a race car gym environment in Pybullet](https://github.com/bulletphysics/bullet3/blob/master/examples/pybullet/gym/pybullet_envs/bullet/racecarGymEnv.py). The problem here is how to get an image from this race car's front side at each timestep.

```python
import pybullet as p
import pybullet_data
from racecarGymEnv import RacecarGymEnv

env.reset()
while True:
    env.step(env.action_space.sample())
```

![Fig-1](https://vision.aioz.io/thumbnail/5538a4e65ccd490399e5/1024/TutFig1.png)
*<center>**Figure 1**: Racecar Pybullet.</center>*

There is a solution by putting the camera sensor on the agent when loading the agent's model to Pybullet via the `loadUrdf` function. Accordingly, it needs to connect to ROS to subscribe to images from the camera sensor. However, this approach is quite complicated and requires knowledge of ROS. Furthermore, when switching from Gazebo to Pybullet, we also expect to use this simulator solely for quickly testing models during training.

We come up with a solution that we can utilize the built-in camera of the Pybullet SDK. The simulator provides a camera image that captures a specific point of view whenever calling the function `pybullet.getCameraImage()`. This function receives a parameter about the camera's focusing point, which, as default, is the origin of the coordinate system. Hence, each time step, the problem is to determine this parameter to put the camera's direction from the agent's perspective.

Fortunately, Pybullet also provides a function that computes the orientation, especially the yaw, of objects. Thus we can get the agent's yaw by the code below:

```python
while True:
    agent_pos, agent_orn =\
        p.getBasePositionAndOrientation(env._racecar.racecarUniqueId)
    yaw = p.getEulerFromQuaternion(agent_orn)[-1]
```

![Fig-2](https://vision.aioz.io/thumbnail/c9fcb29eadaf486eb5d0/1024/Fig2TutorialAIOZ.png)
*<center>**Figure 2**: The yaw value of objects. A is the position of the agent; B is the focusing point of camera</center>*

Let take a look at Fig.2, $\alpha$ is the yaw computed from the function and the red arrow is now the direction of the car in the xy-plane. Obviously, the focus point of the camera lies on the red vector. Say, let's decide the focus point $B$ of the camera is $d =5$ far from the agent $A$, $x_B, y_B,z_B$ can be computed as follow:

$$
x_B = x_A + dcos(\alpha)\\y_B = y_A + dsin(\alpha)\\z_B = z_A
$$

With the idea above, it takes no more than 20 lines of code to program. Two parameters that need to pay attention to are `cameraEyePosition` and `cameraTargetPosition`, which are $A$ and $B$ in Fig.2, respectively. The other parameters are regular options of `p.getCameraImage`. The result can be seen in Fig. 3.

```python
obs = env.reset()
distance = 100000
img_w, img_h = 120, 80
while True:
    env.step(env.action_space.sample())
    agent_pos, agent_orn =\
        p.getBasePositionAndOrientation(env._racecar.racecarUniqueId)

		yaw = p.getEulerFromQuaternion(agent_orn)[-1]
    xA, yA, zA = agent_pos
    zA = zA + 0.3 # make the camera a little higher than the robot

    # compute focusing point of the camera
    xB = xA + math.cos(yaw) * distance
    yB = yA + math.sin(yaw) * distance
    zB = zA

    view_matrix = p.computeViewMatrix(
                        cameraEyePosition=[xA, yA, zA],
                        cameraTargetPosition=[xB, yB, zB],
                        cameraUpVector=[0, 0, 1.0]
                    )

    projection_matrix = p.computeProjectionMatrixFOV(
                            fov=90, aspect=1.5, nearVal=0.02, farVal=3.5)

    imgs = p.getCameraImage(img_w, img_h,
                            view_matrix,
                            projection_matrix, shadow=True,
                            renderer=p.ER_BULLET_HARDWARE_OPENGL)
```

![Fig 3](https://vision.aioz.io/f/7bb30d759cca4f9c9af9/?dl=1)
*<center>**Figure 3**: Result of attaching the pybullet camera.</center>*

## Summary
To sum up, in this post, we introduce a problem occurred when using Pybullet for navigation problem, which is the lack of extrinsic observations for the agent, namely the visual one. We also provide a solution to work solely on Pybullet for training navigation models. Hopefully that this tutorial can help.
