# game-AI-navigation
### Author: Daniil Karzanov
This post is my take on training AI for enemies for my Android game. Here, I share some of the results, attempts and challenges while working on it.
Note. The game itself is currently is in closed testing and will be published to Google Play Market for everyone's access shortly. Contact me if you want to become a tester.  Meanwhile, check out the [TRAILER VIDEO](https://youtu.be/o0IWvtH6mIU)!
Let's get to the post.

The goal for the player is to survive as many enemy waves as possible by eliminating the attacking bots. To make the most out of the experience from playing, I want to move from hard-coded logic for the enemy to AI, or more specifically, to a Deep Reinforcement Learning (DRL) policy that governs agent's actions based on the current observations. The agent's observations are mostly perceptive input. I want to incorporate the following logic; AI should learn how to
- move towards the player as it's the only way to attack it
- avoid players flash light. The game happens during the night and the player sees only what he points at
- avoid obstacles and learn how to navigate between them

Inspired by concepts from Hierarchical Reinforcement Learning, we will target each task separately and iteratively move from one task to another by loading the weights of the learned policy.

The game is implemented in [Godot 4 Engine](https://godotengine.org/) and we use ```godot_rl_agents``` package for the communication between the engine and Python code. The model is based on PyTorch and stable_baselines3 libraries.
I tried several models, and PPO seems to work well for this task.

Action Space.  
The agent should return a continuous 2-dimensional vector $a \in \mathbb{R}^2$ the movement direction that is further normalized to a unit vector on the environment side.

Observation Space.
Observation variables include the normalized vector direction and distance to the player. 
The agent also sees x and y coordinates of its current movement direction to help it understand its global rotation and where it is moving
To help it solve the task fo torch light avoidance, I also add the vector coming from the player's torch.

Object and obstacle navigation is perhaps the most challenging task. We include three short rays to track if the agent colliding with an obstacle or with the player. We also have three long rays pointing straight to help the agent see objects in front of it.
<img src="media/player.png" alt="Image 1" width="35%" style="display: inline-block; margin-right: 10px;">
<img src="media/enemy.png" alt="Image 2" width="35%" style="display: inline-block;">

PICTURE HERE

Initially, I tried to have a smaller observation space and combine one-hot encoding for vision and collision rays by overwriting vision by collision and switching an indicator.

Reward. 


We design the training as follows. Each episode we generate a new starting postion and new random position of the obstacles around the player. We vary the number of obstacles from 0 to 3 to create different levels of difficulty. If the agent goes too far away or fails to reach the player within 15 seconds, the episode starts all over again. If the agent is touching the player, the time isn't counting. 


For each element in the game, we define a ```_physics_process``` function that defines the logic at each frame of the game. Here, if the conditions are satisfied, the agent receives a positive reward for reasonable action and a penalty for bad actions. The reward consists of the following small additions.

Positive Rewards.  
To encourage the player to move in the direction of the player, we add a small bonus equal to the reduction of the distance to the player, i.e. ```reward += 0.01 * (old_distance - current_distance)``` if positive.

If the central vision (long) ray, sees the player, we add a small reward ```reward += 1```
If the agent gets unstuck it gets ```0.0001``` each frame. I chose a small value to prevent our agent from getting stuck-unstuck each other frame on purpose to get a reward.
When the middle short ray is colliding with the player, we switch off the long vision rays and switch on an "is_damaging" indicator in the observation space. Each frame, the agent receives ```reward += damage_made * 50```, and once the player's HP is 0 we give  a large bonus ```reward += 300``` and restart the episode.

Penalties.
It is also import to discourage certain actions by adding a negative reward for inappropriate actions.





<img src="media/demo_obstacles.gif" alt="Image 2" width="35%" style="display: inline-block;">

Additional Notes.  
Changing the learning rate to a smaller value helped the agent get stuck in the obstacles fewer times.
We need to make sure that the penalty for leaving is large enough. In some experiments, the agent learns that it is less costly to immediately run to the endzone and finish the game faster and not to lose points colliding with obstacles.
Scaling and normalizing reward to  the range -1, 1 may be one of the next steps.



