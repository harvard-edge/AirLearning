# Air Learning
Aerial robotics is a cross-layer and interdisciplinary field. Designing an autonomous aerial robot to perform a task involves interactions between various boundaries spanning from modeling the environment down to the choice of onboard computer platform available in the robot. Our goal through building Air Learning is to provide researchers with a cross-domain infrastructure that allows them to holistically study and evaluate reinforcement learning algorithms for aerial autonomous machines. Air Learning is built on top of several open-source tools such as Microsoft AirSim, OpenAI Gym, Stable-Baselines,and Epic games Unreal Engine.

The main components of Air Learning infratructure is depicted below:
![](https://github.com/harvard-edge/airlearning/blob/master/docs/images/airl-infrastructure.png)

## Air Learning Environment Generator
Learning algorithms are data hungry, and the availability of high-quality data is vital for the learning process. Also, an environment that is good to learn from should include different scenarios that are challenging for the robot. By adding these challenging situations, they learn to solve those challenges. For instance, for teaching a robot to navigate obstacles, the data set should have a wide variety of obstacles (materials, textures, speeds, etc.) during the training process.

We designed an environment generator specifically targeted for autonomous UAVs. Air Learning's environment generator creates high fidelity photo-realistic environments for the UAVs to fly in. The environment generator is built on top of UE4 and uses the AirSim UE4 plugin for the UAV model and flight physics. The environment generator with the AirSim plugin is exposed as OpenAI gym interface. 

The environment generator has different configuration knobs for generating challenging environments. The configuration knobs available in the current version can be classified into two categories. The first category includes the parameters that can be controlled via a game configuration file. The second category consists of the parameters that can be controlled outside the game configuration file. The full list of parameters that can be controlled are shown in the table below:

| Parameter           	| Format                  	| Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 	|
|---------------------	|-------------------------	|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| Arena Size          	| [length, width, height] 	| The Arena Size is the total volume available in the environment. It is represented by [length, width, height] tuple. A large arena size means the UAV has to cover more distance in reaching the goal which directly impacts its energy and mission success.  The arena can be customized by adding materials, which we describe in the "materials" section.                                                                                                                                                              	|
| Wall Colors         	| [R, G, B]               	| The Wall Color parameter can be used to set the wall colors of the arena. The parameter takes [R, G, B] tuple as input. By setting different values of [R, G, B], any color in the visible spectrum can be applied to the walls. The neural network policies show sensitivity towards different colors and varying these color during training can help the policy to generalize well.                                                                                                                                      	|
| # Static Obstacles  	| Scalar Integer          	| The # Static Obstacles is a parameter that describes the total number of static objects that is spawned in the environment. Using this parameter, we can generate environments ranging from very dense to very sparse obstacles. Depending upon the value of this parameter, the navigation complexity can be easy or difficult. A large number of obstacles increases the collision probability and can be used for stressing the efficacy of reinforcement learning algorithms.                                           	|
| # Dynamic Obstacles 	| Scalar Integer          	| The # Dynamic Obstacles is a parameter that describes the total number of obstacles that can move in the environment.                                                                                                                                                                                                                                                                                                                                                                                                       	|
| Seed                	| Scalar Integer          	| The Seed parameter is used for randomizing the different parameters in the environment. By setting the same 'Seed' value, we can reproduce (and randomize) the environment (obstacle position, goal position, etc.).                                                                                                                                                                                                                                                                                                        	|
| Minimum Distance    	| Scalar Integer          	| The Minimum distance is a parameter that controls the minimum distance between two static objects in the arena. This parameter in conjunction with # Static Obstacles is what determines congestion.                                                                                                                                                                                                                                                                                                                        	|
| Goal Position       	| [X, Y, Z]               	| The Goal Position is a parameter that specifies the destination coordinate that the UAV must reach. The Goal Position coordinates should always be inside the arena, and there is error checking for input errors. Similar to # Static Obstacles, it increases task complexity.                                                                                                                                                                                                                                             	|
| Velocity            	| [Vmax, Vmin]            	| The Velocity parameter is a tuple of the form [V_min, V_max] that works with # Dynamic Obstacles. The environment generator randomly chooses a value from this range for the velocity of a dynamic obstacle. This coupled with the # Dynamic Obstacles helps control how dynamic and challenging the environment is for the aerial robot.                                                                                                                                                                                   	|
| Asset               	| [folder name]           	| An Asset in Air Learning is a mesh in UE4. Any asset that is available in the project can be used as a static obstacle, dynamic obstacle, or both. At simulation startup, Air Learning uses these assets as either a static or dynamic obstacle. The number of assets that will be spawned in the arena will be equal to the # Static Obstacle and # Dynamic Obstacle parameter. By having the ability to spawn any asset as an obstacle, the UAV agent can generalize to avoid collision with different types of obstacle. 	|
| Materials           	| [folder name]           	| A Material is a UE4 asset that can be applied to meshes to control the visual look of the scene. Material is usually made of multiple textures to create a particular visual effect for the asset. At simulation startup, Air Learning environment generator applies materials to matching assets.                                                                                                                                                                                                                          	|
| Textures            	| [folder name]           	| A Texture is an image that is used on an UE4 asset. They are mapped to the surfaces of any given asset. At startup, the environment generator applies textures to matching assets. Textures and materials help the training algorithm capture different object features, which is important to help the algorithm generalize.                                                                                                                                                                                               	|

### Demonstration of Air Learning Environment Generator
Here is an simple demo to show how we can use Air Learning environment generator to convert a UE4 mesh into a dynamic obstacle.

<p align="center">
<img src= "https://github.com/harvard-edge/airlearning/blob/master/docs/images/environment_generator.gif" >
</p>


Applying different materials to the same obstalces:

![](https://github.com/harvard-edge/airlearning/blob/master/docs/images/materials.jpg)


For information on installing Air Learning environment generator, check the following [repository](https://github.com/harvard-edge/airlearning-ue4/tree/b4f27ea457936609745ddad1191ab8c54f8799ac) or follow the airlearning-ue4 sub-module in this repository.

## Air Learning RL
Deep reinforcement learning is still a nascent field that is rapidly evolving. Hence, there is significant infrastructure overhead to integrate a simulator and evaluate new deep reinforcement learning algorithms for UAVs.

So, we expose our random environment generator and AirSim UE4 plugin as an OpenAI gym interface and integrate it popular reinforcement learning framework with stable baselines (which is based on OpenAI baselines) and Keras-RL. To expose our random environment generator into an OpenAI gym interface, we extend the work of AirGym to add support for environment randomization, a wide range of sensors (Depth image, Inertial Measurement Unit (IMU) data, RGB image, etc.) from AirSim and support exploring multimodal policies.

Currently we support two reinforcement learning algorithms one for discrete actions control and one for continuous action control:
* Deep Q-Networks (DQN)
* Proximal Policy Optimization (PPO)


Using Air Learning we can use train different reinforcement learning algorithms. Here is the video demonstration of using Air Learning environment generator used in RL training.

<p align="center">
<img align= "center" src="https://github.com/harvard-edge/airlearning-ue4/blob/master/Images/training_gif.gif">
</p>



## Hardware Evaluation

Often aerial roboticists port the algorithm onto UAVs to validate the functionality of the algorithms. These UAVs can be custom built or commercially available off-the-shelf (COTS) UAVs but mostly have fixed hardware that can be used as onboard compute. A critical shortcoming of this approach is that the roboticist cannot experiment with hardware changes. More powerful hardware may (or may not) unlock additional capabilities during flight, but there is no way to know until the hardware is available on a real UAV so that the roboticist can physically experiment with the platform. 

Reasons for wanting to do such exploration includes understanding the computational requirements of the system, quantifying the energy consumption implications as a result of interactions between the algorithm and the hardware, and so forth. Such evaluation is crucial to determine whether an algorithm is, in fact, feasible when ported to a real UAV with a specific hardware configuration and battery constraints.

For instance, a Parrot Bepop comes with a P7 dual-core CPU Cortex A9 and a Quad core GPU. It is not possible to fly the UAV assuming a different piece of hardware, such as the NVIDIA Xavier processor that is significantly more powerful; at the time of this writing there is no COTS UAV that contains the Xavier platform. So, one would have to wait until a commercially viable platform is available. However, using Air Learning, one can experiment how the UAV would behave with a Xavier since the UAV is flying virtually. 

<p align="center">
<img align= "center" src="https://github.com/harvard-edge/airlearning/blob/master/docs/images/hil.jpg" width="500" >
</p>

Air Learning uses Hardware-in-the-loop (HIL) methodology for system evaluation. A HIL simulation combines the benefits of the real design and the simulation by allowing them to interact with one another as shown in the figure above. There are three core components in Air Learning HIL methodology: 
* A high-end desktop that simulates a virtual environment flying the UAV.

* An embedded system that runs the operating system, the deep reinforcement learning algorithms, policies and associated software stack  
* A flight controller that controls the flight of the UAV in the simulated environment.

## How to get it

Currently Air Learning is tested on Windows 10. For Linux users, please stay tuned!

There are two parts to Air Learning. 
* The Air Learning environment generator ([Installation instruction](https://github.com/harvard-edge/airlearning-ue4))
* Air Learninng Reinforcement Learning Training ([Installation instruction](https://github.com/harvard-edge/airlearning-rl))

## Participate

### Paper
More technical information and insights on using Air Learning can be found [here](https://arxiv.org/pdf/1906.00421.pdf):

```
@article{krishnan2019air,
  title={Air Learning: An AI Research Platform for Algorithm-Hardware Benchmarking of Autonomous Aerial Robots},
  author={Krishnan, Srivatsan and Borojerdian, Behzad and Fu, William and Faust, Aleksandra and Reddi, Vijay Janapa},
  journal={arXiv preprint arXiv:1906.00421},
  year={2019}
}
```

### Contribute

Air Learning aims in softening the bounderies between Robotics, Reinforcement learning, Controls, and System architecture by providing a cross-disciplinary infrastructure for researchers in these domain. If like us, you also believe in looking at the problem holistically please reach out to us. Also we welcome any contributions that makes this tool better or easy to use.

### Contributors
* Srivatsan Krishnan (Harvard University)
* Behzad Boroujerdian (The University of Texas at Austin)
* William Fu ( Harvard university)
* Aleksandra Faust (Robotics at Google)
* Vijay Janapa Reddi (Harvard University)

### Contact (Maintainers)
* Srivatsan Krishnan (srivatsan@seas.harvard.edu)

