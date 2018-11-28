# Setting up CARLA simulator environment for Reinforcement Learning

 **Table of Contents**

   * [Setting up CARLA simulator environment for Reinforcement Learning](#setting-up-carla-simulator-environment-for-reinforcement-learning)
      * [Introduction](#introduction)
      * [Requirements](#requirements)
      * [Setting up the CARLA Path](#setting-up-the-carla-path)
      * [Getting the required files for RL](#getting-the-required-files-for-rl)
      * [Playing with the Environment](#playing-with-the-environment)
         * [To create an CARLA environment](#to-create-an-carla-environment)
         * [Resetting the environment](#resetting-the-environment)
         * [Taking an action](#taking-an-action)
         * [Reading values after taking an action](#reading-values-after-taking-an-action)
         * [Rendering the game after each action](#rendering-the-game-after-each-action)
      * [Testing CARLA game as a human](#testing-carla-game-as-a-human)


### Introduction
If you didn't know, **[CARLA is an open-source simulator for autonomous driving research.](https://github.com/carla-simulator/carla "CARLA is an open-source simulator for autonomous driving research.")**

It can be used as an environment for training [ADAS](https://en.wikipedia.org/wiki/Advanced_driver-assistance_systems "ADAS"), and also for Reinforcement Learning.

This guide will help you set up the CARLA environment for RL. Most of my code here is inspired from [Intel Coach](https://github.com/NervanaSystems/coach "Intel Coach")'s setup of CARLA. I thought it'd be helpful to have a separte guide for this, to implement our own RL algorithms on top of it, instead of relying on Nervana Coach.

## Requirements

- Download the[ latest CARLA release from here](https://github.com/carla-simulator/carla/releases " latest CARLA release from here").   
(As of the time of writing, CARLA is in Experimental stage for Windows OS)   
(Tested using CARLA 0.8.0)
- Any Debian-based OS (Preferably Ubuntu 16.04 or later)
- Python 3.x installed
- To install python packages:
    `pip install -r requirements.txt`

## Setting up the CARLA Path

After downloading the release version, place in any accessible directory, preferably something like `/home/username/CARLA` or whatever.

Now open up your terminal, enter **`nano ~/.bashrc`** and include the PATH of the CARLA environment like:

```bash
export CARLA_ROOT=/home/username/CARLA
```

## Getting the required files for RL

Just clone (or fork) this repo by
```
git clone https://github.com/GokulNC/Setting-Up-CARLA-RL
```

All the required files for Environment's RL interface is present in the `Environment` directory (which you need not worry about)
*Note*: Most of the files are obtained from Intel Coach's interface for RL, with modifications from my side.

## Playing with the Environment

The environment interface provided here is more or less similar to that of [OpenAI Gym](https://github.com/openai/gym) for standardization purpose ;)

### To create an CARLA environment
```python
from Environment.carla_environment_wrapper import CarlaEnvironmentWrapper as CarlaEnv

env = CarlaEnv()  # To create an env
```

### Resetting the environment
```python
# returns the initial output values (as described in sections below)
initial_observation = env.reset()
```

### Taking an action

```python
observation, reward, done, info = env.step(action_idx)
```

where **`action_idx`** is the discretized value of action corresponding to a specific action.

As of now, there are 9 discretized values, each corresponding to different actions as defined in  **`self.actions`** of `carla_environment_wrapper.py` like

```python
actions = {0: [0., 0.],
					1: [0., -self.steering_strength],
					2: [0., self.steering_strength],
					3: [self.gas_strength, 0.],
					4: [-self.brake_strength, 0],
					5: [self.gas_strength, -self.steering_strength],
					6: [self.gas_strength, self.steering_strength],
					7: [-self.brake_strength, -self.steering_strength],
					8: [-self.brake_strength, self.steering_strength]}

actions_description = ['NO-OP', 'TURN_LEFT', 'TURN_RIGHT', 'GAS', 'BRAKE',
									'GAS_AND_TURN_LEFT', 'GAS_AND_TURN_RIGHT',
									'BRAKE_AND_TURN_LEFT', 'BRAKE_AND_TURN_RIGHT']
```

(Feel free to modify it as you see fit)

### Values returned from env.step() (after taking an action)

```python
# observation   :   observation after taking the action

# To get RGB image from the observation:
state = observation['rgb_image']
# TODO: In future, will add supoort for LiDAR sensors, etc. as required

# reward       :   immediate reward after taking the action

# done          :   boolean True/False indicating if episode is finished
#                       (collision has occured or time limit exceeded)

# info          :   information about the action taken & consequences
# To get the id of the last action taken
last_action_idx = info['action']
# more info will be added later
```

### Rendering the game after each action
CARLA [automatically renders everything](https://github.com/carla-simulator/carla/issues/286) as you play (take actions/pass controls). So no need of explicitly rendering.   
If you need to render the camera view,
```python
env = CarlaEnv(is_render_enabled=True)  # To create an env

# To render after each action:
env.render()
```

### Saving screenshots
```python
env = CarlaEnv(save_screens=True)  # To create an env

# To save after each action:
env.save_screenshots()
```

## Testing CARLA game as a human

I have included a file **`human_play.py`** which you can run by
```
python human_play.py
```

and play the game manually to get an understanding of it. (Make sure the focus is on the terminal window)   
Use the arrow keys to play (`Up` to accelerate, `Down` to brake, `Left/Right` to steer)

## Extras:

- You can change resolution of server window, render window and other configs in `Environment/carla_config.py`
- You can get the following outputs, instead of just RGB image:
  - For Segmentated output: `env = CarlaEnv(cameras=['SemanticSegmentation'])` and `segmented_output = observation['segmented_image']`
  - For depth output: `env = CarlaEnv(cameras=['Depth'])` and `depth_map = observation['depth_map']`
  - (Note: You can also use a combination of everything. For RGB output, `cameras=['SceneFinal']`)   
(To play with your own cameras, feel free to modify things as described [here in docs](https://carla.readthedocs.io/en/latest/cameras_and_sensors/))


## TODO in future:

- As of now, the CarlaEnvironmentWrapper supports both continous & hardcoded discretized values. I think discretized action values can be removed
- Make it Gym compliant (for benchmarks)

Feel free to contribute!
