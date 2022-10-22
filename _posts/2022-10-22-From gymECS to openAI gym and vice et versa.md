
## (Note from the previous post)

Since the previous post, we have updated the examples providing a maze with goal game (`gymecs_examples/maze/maze_with_goal.py`) but also a multi-agent version of the maze with goal (`gymecs_examples/maze/multiagent_maze_with_goal.py`). Take a look to better understand how to implement more complex systems with ECS.

<p style="text-align: center;"><video autoplay loop src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_multiagent_video.mkv" controls="controls" style="max-width: 640px;">
</video></p>
<p style="text-align: center;">The Multiagent with Goal Game. Multiple (red) players have to go to the (green) goal before the other players.</p>

# Game-centric versus Agent-centric

One main difference between gymECS and openAI gym is that, while `gym` is agent centric and allows to access the state of the world from the agent point of view, `gymECS` is **system centric/game centric** and allows access to any information that describe the state of the world. In that sense, gymECS is much more general than gym since it exposes all the data that describe the complete state of the system at each timestep. One of the good characteristic is that, with gymECS, it is possible to **put neural networks everywhere** in the dynamical system, to replace players, physics, etc..., while gym is limited to focusing on the agent. We will demonstrate the interest of the **system centric** approach in the `to gym` section. But let first start showing how a gym environment can be transformed to and ECS game. 

## From gym to gymECS

Since gymECS is more general than gym, building a bridge from gym to gymECS is easy: **any gym environment** can be easily converted to a **gymECS game**. 

For that, we just need to implement components and entities describing the information computed by the gym environment. 

**Note:** As a first step, we updated the **Component** and **Entity** classes to incorporate `_structureclone` and `_deepclone` methods to facilitate the cloning of data (see `core.py`)

### ECS Components and Entities

To describe the state of a gym environment, we use two entites: `Agent` that contains the observation, reward and action of the agent (the agent-centric information), and `Game` that contains the state of the game, including if the game is finshed or not.

The agent information is defined as follows:

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_agent.png" width="80%"/></p>

<p align='center'> <em>The components and entity to describe an agent in a gym environment</em></p>

For the game state, we will include the `done` information, but also the timestep of the game. Since the ECS exposes all the data, the `gym.Env` is also contained in the `World` as a `GameEnv` component that makes it usable by system.

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_game.png" width="50%"/></p>

<p align='center'> <em>The components and entity to describe a game state</em></p>

### ECS System

To update the state of our game, we need to define a `step` system. This sytem will read the `action` information, and execute it to compute update the world.


<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_system.png" width="150%"/></p>

<p align='center'> <em>The Step system to execute one step of the environment</em></p>

### ECS Game

The resulting game can then be defined as follows. 

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_gamee.png" width="150%"/></p>

<p align='center'> <em>The Game capturing a gym.Env</em></p>

### Playing with the game

To test the game, we need to define a system modeling the player. In our case, it is a simple random player with 2 actions.

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_player.png" width="150%"/></p>

<p align='center'> <em>The Game capturing a gym.Env</em></p>

The final loop to test our game is the following:


<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_loop.png" width="150%"/></p>

<p align='center'> <em>The Game capturing a gym.Env</em></p>

<p style="text-align: center;"><video src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_video1.mkv" controls="controls" style="max-width: 1600px;">
</video></p>


### A few words

It is very simple  to cast any gym environment to a `gymecs.Game` and we provide a generic wrapper. A similar wrapper can be easily made for other interfaces like deepmind lab for instance. gymecs thus provides a unified API for dynamical systems, making my life much easier ! But gymECS can represent dynamical systems much more complex than **agent centric** frameworks. 

## From gymECS to openAI gym

The reverse path from the ECS to gym is the most interesting property. Indeed, as stated before, gymECS is **system centric** while many RL frameworks including openAI gym are **agent centric**. But in a game, we may want to control different stuffs, not only a single agent: a bot, a part of the game logics, multiple bots at once, etc.... gymECS allows to do that, but not openAI gym.


To move from the **system centric** to the **agent centric** point of view (from gymECS to gym), we need to specify what is the `agent` in the game, what are its observations, reward, etc... It means that one gymECS game can be transformed in multiple gym environments depending on what we decide the `agent` to be.

### The Maze Game
To convert our simple maze game to a gym environments, we first define the following class and abstract methods:

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_togym1.png" width="80%"/></p>
<p align='center'> <em>The gym.Env class to capture a game as a gym environments (see complete code in togym.py)</em></p>

Then, a maze game can be matched to a gym environment as follows:

For that, we define the follwing class and abstract methods:
<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_togym2.png" width="100%"/></p>
<p align='center'> <em>Casting our Maze as a gym environment. The observation will be the X,Y position of the agent.</em></p>

To execute this example: `python gymecs\gymecs\togym.py`

### The MultiAgent Maze Game - Single Agent point of view

Let us now take the multi agent maze game as an example. In this game (`gymecs_examples/maze/multiagent_maze_with_goal.py`), there are multiple agents trying to reach the goal. So there are multiple ways to convert this game to a gym environments: maybe we want to focus on one of the agents, maybe with want to learn the multiple agents as one agent at once, etc.... 

First case: we focus on a single agent. In that case, the implementations is made as follows (see `gymecs_examples/maze/gym_multiagemt_maze_with_goal_singleagent_pointofview.py`):

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_ma1.png" width="150%"/></p>
<p align='center'> <em>Casting our Multiagent Maze as a gym environment focusing on a single agent</em></p>

In addition, we have to take care about who is in charge of controlling the other agents. To do that, we can put the dynamics of the other agents directly in the game: 

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_ma2.png" width="150%"/></p>
<p align='center'> <em>Putting othe players dynamics in the game</em></p>

The main function is thus:
<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_ma3.png" width="150%"/></p>
<p align='center'> <em>Putting othe players dynamics in the game</em></p>

### The MultiAgent Maze Game - All Agents point of view

But maybe we want to learn to control all the agents simultaneously in a synchronous way. In that case, we can also adapt the gymECS game to take control of all the agents, providing a vector of actions to the resulting gym environment (see `gymecs_examples/maze/gym_multiagemt_maze_with_goal_allagents_pointofview.py`):

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_ma4.png" width="150%"/></p>
<p align='center'> <em>Putting othe players dynamics in the game</em></p>

The main function is simply:
<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_ma5.png" width="150%"/></p>
<p align='center'> <em>Putting othe players dynamics in the game</em></p>


## Conclusion

Building bridges between existing frameworks and the gymECS one is easy. But, one interesting property is that gymECS is **game centric** and thus much less restricted than openAI gym, letting anyone define on which aspect of the game he/she wants to work on. As defended previously in [the SALINA library](https://arxiv.org/pdf/2110.07910.pdf), reinforcement learning has provided the `environment/agent` formalism which is in fact very restricted. 

I advocate to consider that a dynamical system (or an ECS) is a combination of many multiple dynamics functions, and reinforcement learning is one potential set of algortihms to learn one or multiple of these functions (that are usually called `agent` in RL) while letting the other functions fixed (the `environment`). This can be seen as a useless semantic debate, but actually, I think that considering the objects we manipulate as `dynamic systems/ECS` instead of `agent+environment` opens many different interesting directions. In that view, learning the environment dynamics, the physics, the bot, the agent, the rendering, etc.... is the same, and it makes everything much simpler. 

The next post will be about 3D rendering. 







