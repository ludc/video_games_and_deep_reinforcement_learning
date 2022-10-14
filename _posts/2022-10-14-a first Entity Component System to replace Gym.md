
## What is an ECS?

An **ECS** (Entity-Component System) is a programming paradigm allowing the implementation of **complex dynamical systems**. It is more and more used in the video games industry due to its excellent characteristics:
* It separates **data** (e.g variables describing the state of the system) from **systems** (e.g components implementing the dynamics of the system).
* It allows specifying the dynamics as a computation graph (of systems) that can be dispatched and executed over different parallel architectures with high efficiency. It also allows (just-in-time) compilation.
* It can be computed over a decentralized architecture i.e different systems running on different computers (good point for multiplayer games)
* It is highly modular and facilitates the easy implementation of complex functionalities.

On the drawback side, the main drawback is that it is **not oriented-object** and may necessitate a (small) time to adapt to it. 

Different (excellent) ECS systems have been recently developed, and I wonder they will be more and more present in video games:
* The [DOTS component](https://unity.com/dots) in Unity (C#)
* The [FLECS framework](https://github.com/SanderMertens/flecs) (free and super nice - C and C++)
* The [BEVY Engine](https://bevyengine.org/) (in Rust)

Our objective in this post is to do an ECS in pure python, dedicated to reinforcement learning at scale. 

## Key Principles

The principles underlying ECS are very simple:
* The state of the system (or game) is stored as a set of **Entities**, each entity being composed of different **Components**. Each of these components is actually storing the data describing the state. Said otherwise, information is organized into a three levels tree, the first level being the entities, the second one being the components, and the last one being the data. Having such a data structure is interesting in terms of storage (use of continuous memory, caching, etc…) but this is not the focus of this article. What we can retain here is the simplicity of the structure. The set of all entities is stored in the **World** which is an entity container, usually associated with a **query language** allowing for extracting data. The world is a complete snapshot of the state of the game at time *t*.
* The state (i.e the world) is then updated by executing **Systems**. A system is a function that aims at modifying the entities and components stored in the world. Different philosophies are developed to define the system API. The assumption we are doing in the  blog is that a system is computing new entities (that may overwrite existing ones) or/and new components (that may overwrite existing ones), and are not allowed to directly modify information in the components. Said otherwise, we consider that components are immutable, and entities are modified by overwriting their components. It is not the best choice in terms of computation speed, but it will facilitate the implementations of distributed systems.  Importantly, any system has acces to the whole world information (through the world query language), and thus has complete access to the state of the game at execution time.

## Why is it interesting for Deep (Reinforcement) Learning?

What makes ECS super interesting from the Deep Reinforcement Learning (DRL) point of view is that such a paradigm proposes a clear separation between data and functions, exactly has **we are used to do when manipulating neural networks**. Said otherwise, by using ECS, you can write the dynamics of your system through a computation graph (see Figure 1) of 'pure' functions (in the same spirit than JAX for instance). It opens all the possibilities of provided computation graphs (e.g JIT). More importantly, it allows to **put neural networks** everywhere in the game as we will show in future posts: not only to replace a player (bots), but also to enrich game data, to make/augment rendering, to do level generation, etc… (see Figure 2)

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_fig2.png)
**Figure 1:** *An ECS implements a computation graph whose input is the World composed of Entities and Components.image_caption*

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_fig3.png)
**Figure 2:** *ECS makes easy the use of neural networks to replace one or more systems in the game. It allows to deploy machine learning everywhere.*

# GymECS: A simple ECS to replace gym environments

Now that I have explained the main ideas, let us implement a **simple ECS in python** dedicated to deep (reinforcement) learning research and experimentation. Our objective is to provide a simple framework, close to openAI gym, but ECS oriented and richer that gym which is an agent centric framework. The gymecs system will be game centric and will facilitate the implementation of complex logics and learning systems. Note that, while simple at the beginning, we will show that such an approach allows to develop **realistic 3D games with 3D physics**, but also to make complex computations on GPUs. At the end of the series, our ECS will be a complete python game engine, with the frinedly characteristic to be naturrally compatible with reinforcement learning algorithms. 

In this implementation, I will not focus on the execution speed of the framework, but on the ease to use to test various ideas in deep learning. This ECS is available at [gymecs Github](https://github.com/ludc/gymecs), and will be updated upon publications of blog posts -- one branch for each post.

## Step 1: Component and Entity

A component is a simple container of data. A simple implementation is the following:

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_component.png)
*The Component class*

It makes use of the `setattr`, `hasattr` and `getattr` functions of Python to simplify the usage of the components. 

As an example, a new component can be defined like this:

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_position.png)
*The Position Component*

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_action.png)
*The Action Component*

and can be created like this:

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_component_creation.png)
*Creating Components*

Similarly, it is possible to create an Entity class as:
![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_entity.png)
*The Entity class*

and to define new entities types (also called Archetypes in some other ECS libraries):

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_agent.png)
*The Agent Entity*

We can then create the entity describing an agent as:

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_agent_creation.png)
*Creating an agent is very simple...*

Not that components can contain any type of information like big tensors for instance. In order to store our entities, we need to define a **World** class. We propose to just organize the entities in a simple dictionary for sake of simplicity.

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_world.png)
*The World class (see github for full code)*

## Step 2: System

A **System** is just a function that: 
* a) query a world to collect information, 
* b) process this information, and 
* c) create new components/entities to insert in the world (or delete entities). 
  
While it is possible to separate these step explicitly, we propose a very simple implementation as:

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_system.png)

Note that the `reset` function is useful to control the stochasticity of the system (if stochastic) for reproducibility. It allows to have a fully deterministic system.

## Step 3: Game

We have **Component**, **Entity**, **World** and **System**. Let us package all this stuff into a **Game**.


A game is composed of a **World** and **multiple Systems**. Similarly to openAI gym, we consider two methods, `reset` and `step`, but with **one major difference**. The `step` method does not take actions as input. Indeed, in the ECS, the information is stored in the world, so the action taken by any agent at timestep t is stored in the World. Following the [salina](https://arxiv.org/abs/2110.07910) philosophy, our small ECS considers that **everything is a system**. Any agent will be also system (or a combination of systems), producing actions in the World. For Reinforcement Learning, the reward will also be just an information inside the world, and **we do not need to differentiate between reward, action, observation, etc..** It makes everything simpler.


The resulting implementation is the following:

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_game.png)
*The Game class*

Note that for simplicity, we are keeping the render function from gym. But actually, rendering could be handled by a system like any other, but producing pixels.

# Our first Game in gymECS

Now, let see how we can use these abstractions to define our first game. It will be a simple 2D Maze with an agent moving into it (4 discrete actions). 

### Defining Components and Entities

We define all the information that capture the state of our game in components and entities. In our game, it is simply the position and action of the agent, and the map and size of the maze.
![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_game1.png)
*Components and Entities for a simple maze game*

### Defining the needed systems

In our case, we need two systems: the system to control the dynamics of the game, and the system to control the agent (aka the player)

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_game2.png)
*The move agent system*

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_game3.png)
*A random player*

**Note that** there is not difference between the dynamics of the environment or the agent. Again, everything is a system. 

### Packaging into a game

We just need to initialize the game in the `reset` function by creating the initial world. Then the `step` function is just executing the system that models the dynamics of our game. Note that we decide here to keep the player out of the game, but it could be perflectly included as one of the systems of the game. 

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_game4.png)
*The Maze game: the github contains also a rendering function*

### Running the game

Now, we just have to run the game using a simple loop.

![](https://github.com/video_games_and_deep_reinforcement_learning/docs/assets/post1_game5.png)

# Conclusion

ECS systems are wonderful for making AI in games: (more and more) games programmers enjoy them, and they share a lot of similarities with classical Deep learning tools. So they are a **nice way to bridge the two communities** and our blog is dedicated to building such a bridge.

If you look at the previous implementation, as a first advantage, you see the clear separation between the data and the systems. If you are used to manipulating gym environments, I imagine that you understand how it simplifies the understanding of the 'environment'. If you have a gymECS *Game*, then it is easy to identify the relevant information you would need to make your experiments and develop new models: the environment becomes transparent. 
 
In the next posts, I will explain how these simple principles can be used to do reinforcement learning on games much more complex than what we are used to manipulate in academic research. I will slowly make the ECS evolve toward a **3D distributed engine working on GPU** and allowing to implement multiplayer games, but also toward a real **deep reinforcement learning** sandbox. 

Let's keep in touch, feel free to send comments on [Twitter over this post](...)
