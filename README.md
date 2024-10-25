# game-AI-navigation
### Author: Daniil Karzanov
This post is my take on training AI for enemies for my Android game. Here, I share some of the results, attempts and challenges while working on it.
Note. The game itself is currently is in closed testing and will be published to Google Play Market for everyone's access shortly. Contact me if you want to become a tester.  Meanwhile, check out the [TRAILER VIDEO](https://youtu.be/o0IWvtH6mIU)!
Let's get to the post.

The goal for the player is to survive as many enemy waves as possible by eliminating the attacking bots. To make the most out of the experience from playing, I want to move from hard-coded logic for the enemy to AI, or more specifically, to a Deep Reinforcement Learning (DRL) policy that governs agents actions based on the current observations. The agent's observations are mostly perceptive input. I want to incorporate the following logic; AI should learn how to
- move towards player as it's the only way to attack it
- avoid palyers flash light. The game happens udring the night and the player sees only what he points at
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

Object and obstacle navigation is perhaps the most challenging task. We include three short rays to track if the agent colliding with an obstacle or with the player. We also have three long rays poiting straight to help the agent see objects in front of it.
<img src="media/player.png" alt="Image 1" width="35%" style="display: inline-block; margin-right: 10px;">
<img src="media/enemy.png" alt="Image 2" width="35%" style="display: inline-block;">

PICTURE HERE

TODO: about same vector for collision and vision

Reward. 
For each element in game, we define a ```_physics_process``` function that defines the logic at each frame of the game. Here, if the conditions are satisfied, the agent receives a positive reward for a reasonable action and a penalty for bad actions.
x



