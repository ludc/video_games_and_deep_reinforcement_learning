# Using GPU to speed-up computatons in GymECS

We have previously shown how to define a simple maze game with `gymECS`. If we execute the maze, we can compute how many frames per seconds our system is able to achieve (witout rendering).

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post3_fps1.png" width="150%"/></p>
<p align='center'> <em>The simple maze is executed at 65000 frames per seconds (on my computer)</em></p>

This is mainly due to the fact that the computation is made on a single CPU -- knowing that python is not known to be efficient in terms of computation speed. 

To speed-up the processing, it is possible to rely on GPU librairies directly in the definition of our entities, components and systems. Note that this is the strategy also adopted in different frameworks like [BRAX](https://github.com/google/brax). 

In our ECS, the use of GPU can be made for different usage:
* Parallelizing multiple games as a single game (what we show here)
* Representing multiple agents in a single game
* Making part of the computations in the game (for instance particles)

gymECS allows to put GPU computation anywhere in the game. 

## Parallelizing the maze game

As an example, let us show how we can parallelize the execution of thousands of mazes using [JAX](https://github.com/google/jax).

### Entities and Components
First of all, let us redefine our components and entities, such that the world will contain information about **N** mazes instead of a single one. For instance, the position of the agents will now be a **N x 2** JAX tensor corresponding to the coordinates of the **N** agents playing the **N** games. Similarly, the maze map will be a **N x Width x Height** tensor describing the map of **N** mazes instead of one. 

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post3_ec.png" width="50%"/></p>
<p align='center'> <em>The entities and components in JAX representing N games</em></p>

### Systems

Then, we need to redefine our systems to model the dynamics of **N** games simultaneously. This can be done by using the `lax` functions provided by `jax`.  Without going into details, let us rewrite the moving system.

First of all, we define single functions to model the movements of a single agent with JAX tensors:

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post3_j1.png" width="60%"/></p>
<p align='center'> <em>Elementary functions in JAX</em></p>

Then, thanks to JAX, these functions can be automatically **batched** to operate on **N** agents simultaneously. Moreover, we can use the **Just-in-time compilation** features to compute a very fast system:

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post3_j3.png" width="150%"/></p>
<p align='center'> <em>Elementary functions in JAX</em></p>

The remaining of the code is very similar the the previous one (see `gymecs_examples/gpumax/jax_maze.py`)

### Running the game on GPU. What is the speedup? 

Now that the game has been implemented in JAX, I let you see how many frames per seconds you can obtain..... But it is now dozens of millions !!!!

# Conclusion

Of course, the drainECS framework is not as fast as a pure C++ or Rust ECS since it is coded in python. But on the other side, it can benefit of the large scale computing tools recently developped in the deep learning community to achieve very high performance but executing computations on GPU. 

When developping real games, the usual assumption is to use the GPU **solely for rendering**, and bringing classical game logics on GPU is not desirable. So, for developping classical games, the python ECS is certainly not the best choice. But many games could beneficiate of GPU computations: don't you remember the first versions of civilization when you had to wait dozens of seconds between each turn for the game to compute all opponents moves ? And now imagine a civilization on steroids, with millions of  agents  computed on GPU, even just in python ! 

























