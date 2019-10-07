# Air Learning
Aerial robotics is a cross-layer, interdisciplinary field. **Air Learning** is an effort to bridge seemingly disparate fields.

Designing an autonomous robot to perform a task involves interactions between various boundaries spanning from modeling the environment down to the choice of onboard computer platform available in the robot. Our goal through building Air Learning is to provide researchers with a cross-domain infrastructure that allows them to holistically study and evaluate reinforcement learning algorithms for autonomous aerial machines. Air Learning is built on top of several open-source tools such as Microsoft AirSim, OpenAI Gym, Stable-Baselines, and Epic games Unreal Engine.

We depict the main components of the Air Learning infrastructure below:
![](https://github.com/harvard-edge/airlearning/blob/master/docs/images/airl-infrastructure.png)

Key features in this version of Air Learning includes:

* A photorealistic, configurable, and random environment generator based unreal game engine for facilitating domain randomization for the aerial robot.

* We use Microsoft's AirSim plugin in our configurable environment generator for aerial robot model and physics.  We augment the AirSim plugin by having an energy model for the aerial robot.

* OpenAI gym interface to environment generator to train a variety of reinforcement learning algorithms.

* A tight coupling between the parameters of environment generator and reinforcement learning to provide infrastructure to train aerial robot using curriculum learning.

* Quality of Flight metrics to evaluate the performance of reinforcement learning.

* Benchmarking the performance of reinforcement learning algorithm on different onboard computer platforms using hardware-in-the-loop methodology, thus allowing to study the problem holistically.

* Source seeking application 

## Air Learning Environment Generator
Learning algorithms are data hungry, and the availability of high-quality data is vital for the learning process. Also, an environment that is good to learn from should include different scenarios that are challenging for the robot. By adding these challenging situations, they learn to solve those challenges. For instance, for teaching a robot to navigate obstacles, the data set should have a wide variety of obstacles (materials, textures, speeds, etc.) during the training process.

We designed an environment generator specifically targeted for autonomous UAVs. Air Learning's environment generator creates high fidelity photo-realistic environments for the UAVs to fly in. The environment generator is built on top of UE4 and uses the AirSim UE4 plugin for the UAV model and flight physics. The environment generator with the AirSim plugin is exposed as OpenAI gym interface. 

The environment generator has different configuration knobs for generating challenging environments. The configuration knobs available in the current version can be classified into two categories. The first category includes the parameters that can be controlled via a game configuration file. The second category consists of the parameters that can be controlled outside the game configuration file. The full list of settings/parameters that we can control are shown in the tabulated [here](https://github.com/harvard-edge/airlearning-ue4):
 

### Demonstration of Air Learning Environment Generator
Here is a simple demo to show how we can use Air Learning environment generator to convert a UE4 mesh into a dynamic obstacle.

<p align="center">
<img src= "https://github.com/harvard-edge/airlearning/blob/master/docs/images/environment_generator.gif" >
</p>


Applying different materials to the same obstacles:

![](https://github.com/harvard-edge/airlearning/blob/master/docs/images/materials.jpg)


For information on installing Air Learning environment generator, check the following [repository](https://github.com/harvard-edge/airlearning-ue4/tree/b4f27ea457936609745ddad1191ab8c54f8799ac) or follow the air learning-ue4 sub-module in this repository.

## Air Learning RL
Deep reinforcement learning is still a nascent field that is rapidly evolving. Hence, there is significant infrastructure overhead to integrate a simulator and evaluate new deep reinforcement learning algorithms for UAVs.

So, we expose our random environment generator and AirSim UE4 plugin as an OpenAI gym interface and integrate its popular reinforcement learning framework with stable baselines (which is based on OpenAI baselines) and Keras-RL. To expose our random environment generator into an OpenAI gym interface, we extend the work of AirGym to add support for environment randomization, a wide range of sensors (Depth image, Inertial Measurement Unit (IMU) data, RGB image, etc.) from AirSim and support exploring multimodal policies.

Currently, we support two reinforcement learning algorithms one for discrete actions control and one for continuous action control:
* Deep Q-Networks (DQN)
* Proximal Policy Optimization (PPO)


Using Air Learning, we can train different reinforcement learning algorithms. Here is the video demonstration of using Air Learning environment generator used in RL training.

<p align="center">
<img align= "center" src="https://github.com/harvard-edge/airlearning-ue4/blob/master/Images/training_gif.gif">
</p>



## Hardware Evaluation

Often aerial roboticists port the algorithm onto UAVs to validate the functionality of the algorithms. These UAVs can be custom built or commercially available off-the-shelf (COTS) UAVs but mostly have fixed hardware that can be used as onboard compute. A critical shortcoming of this approach is that the roboticist cannot experiment with hardware changes. More powerful hardware may (or may not) unlock additional capabilities during flight, but there is no way to know until the hardware is available on a real UAV so that the roboticist can physically experiment with the platform. 

Reasons for wanting to do such exploration includes understanding the computational requirements of the system, quantifying the energy consumption implications as a result of interactions between the algorithm and the hardware, and so forth. Such evaluation is crucial to determine whether an algorithm is, in fact, feasible when ported to a real UAV with a specific hardware configuration and battery constraints.

For instance, a Parrot Bepop comes with a P7 dual-core CPU Cortex A9 and a Quad core GPU. It is not possible to fly the UAV assuming a different piece of hardware, such as the NVIDIA Xavier processor that is significantly more powerful; at the time of this writing, there is no COTS UAV that contains the Xavier platform. So, one would have to wait until a commercially viable platform is available. However, using Air Learning, one can experiment how the UAV would behave with a Xavier since the UAV is flying virtually. 

<p align="center">
<img align= "center" src="https://github.com/harvard-edge/airlearning/blob/master/docs/images/hil.jpg" width="500" >
</p>

Air Learning uses Hardware-in-the-loop (HIL) methodology for system evaluation. A HIL simulation combines the benefits of the real design and the simulation by allowing them to interact with one another, as shown in the figure above. There are three core components in Air Learning HIL methodology: 
* A high-end desktop that simulates a virtual environment flying the UAV.

* An embedded system that runs the operating system, the deep reinforcement learning algorithms, policies, and associated software stack  
* A flight controller that controls the flight of the UAV in the simulated environment.

## Quality of Flight Metrics
The success rate has been the sole criteria for evaluating reinforcement learning algorithms so far. It kind of makes sense for domains like computer games. However, for applying the success of these learning algorithms to a mobile robot (like UAVs), success rate alone seldom captures the efficacy of the algorithm. For instance, UAVs are heavily constrained by the compute capability as well as the amount of energy available onboard. To give a very high-level perspective, In self-driving cars, the compute prototypes alone consume about [2.5kW of power](https://www.wired.com/story/self-driving-cars-power-consumption-nvidia-chip/) which translate to reduced range of the car. In contrast to the car, the situation is extremely dire for UAVs. For example:

<p align="center">
<img align= "center" src="https://github.com/harvard-edge/airlearning/blob/master/docs/images/energy-significance.png" width="500" >
</p>

The energy delta between a car and UAVs (High-end) is about 1000x. Hence an algorithm designed specifically for UAVs has to account for these constrains early on in the design phase. Hence when we developed Air Learning, we baked other metrics (beyond success rate) that quantifies the overall quality of flight. In this version of Air Learning, we have the following quality of flight metrics:
* Success rate
* Energy per mission
* Distance Travelled
* Flight time

## How to get it

Currently, Air Learning is tested on Windows 10. For Linux users, please stay tuned!

There are two parts to Air Learning. 
* The Air Learning environment generator ([Installation instruction](https://github.com/harvard-edge/airlearning-ue4))
* Air Learning Reinforcement Learning Training ([Installation instruction](https://github.com/harvard-edge/airlearning-rl))

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

Air Learning aims in softening the boundaries between Robotics, Reinforcement learning, Controls, and System architecture by providing a cross-disciplinary infrastructure for researchers in this domain. If like us, you also believe in looking at the problem holistically, please reach out to us. Also, we welcome any contributions that make this tool better or easy to use.

### Contributors
* Srivatsan Krishnan (Harvard University)
* Behzad Boroujerdian (The University of Texas at Austin)
* William Fu (Harvard University)
* Aleksandra Faust (Robotics at Google)
* Vijay Janapa Reddi (Harvard University)
* Bardienus Duisterhof (TU Delft/ Harvard University)

### Contact (Maintainers)
* Srivatsan Krishnan (srivatsan@seas.harvard.edu)
* Bardienus Duisterhof (b.p.duisterhof@gmail.com) 
