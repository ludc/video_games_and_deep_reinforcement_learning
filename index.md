<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/robot.png" width="100%"/></p>

My objective is to publish a series of posts explaining how ECS can be leveraged **to bridge video game programming and deep (reinforcement) learning**. The articles are structured to progressively demonstrate how an ECS can be developed 'from scratch' in python and integrated with classical DRL tools, resulting in a **powerful learning platform** associated to **complex and beautiful games**. 

The associated source code is [https://github.com/ludc/gymecs](https://github.com/ludc/gymecs) - Each blog post is associated with a branch of the github repository. 

### The author

 I am a deep (reinforcement) learning professor from Sorbonne University on a sabbatical. I worked at Criteo and spent almost 4 years at Meta (FAIR) doing academic research but also concrete projects to democratize the use of RL techniques (e.g have a look at [salina](https://github.com/facebookresearch/salina) which is a wonderful  RL library :) ). I recently joined Ubisoft, going back to my roots since I did AI research and computer science mainly motivated by video games when I was younger. I am right now a research scientist at La Forge.

* [Google Scholar Page](https://scholar.google.com/citations?user=9PLqulwAAAAJ&hl=fr)
* [Twitter Profile](https://twitter.com/LudovicDenoyer)

**This blog only expresses my own opinions and my personal thoughts** 


## Actual Posts

* **14th of October, 2022:** [Defining a pure python ECS, why is it good for Deep Reinforcement Learning?](https://ludc.github.io/video_games_and_deep_reinforcement_learning/2022/10/14/a-first-Entity-Component-System-to-replace-Gym.html)

In this post, we give the basics of Entity Component Systems and show a simple implementation in Python. We then develop a simple 2D maze game as an example of use.

<p style="text-align: center;"><video autoplay loop src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/index_maze.mkv" controls="controls" style="max-width: 640px;">
</video></p>
<p style="text-align: center;">The developped Maze Game</p>

## Future Posts

* **October, 2022 (to come):** [Building bridges between gymecs and openAI Gym]()

In this post, we show how are ECS can be casted as a gym environment, and vice-et-versa. We also discuss the advantage of the ECS w.r.t other deep RL environments libraries aka our ECS is **game-centrc** while classical libraries are **agent-centric**. `gymecs` is thus much more flexible and general. 

<p style="text-align: center;"><video autoplay loop src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post2_video.mkv" controls="controls" style="max-width: 640px;">
</video></p>
<p style="text-align: center;">openAI gym as a game in our ECS</p>

* **October, 2022 (to come):** [Adding 3D rendering with raylib]()

We explain how ECS can be easily extended to handle 3D rendering (using the raylib library). 

<p style="text-align: center;"><video autoplay loop src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/index_raylib.mkv" controls="controls" style="max-width: 640px;">
</video></p>
<p style="text-align: center;">The previous maze with 3D rendering</p>

* **October, 2022 (to come):** [Using GPU for fast computations in the ECS]()

gymecs allows  to naturrally integrate processings mmades on GPU to drastically speed-up computations and speed-up game dynamics. We show how classical GPU librairies can be used in the ECS and provide an example using JAX.

* **October, 2022 (to come):** [Integrating 3D Physics in the game]()

We explain how 3D physics can be integrated in the ECS by using the **Physx** library that works on multiple CPUs and GPUs. 

<p style="text-align: center;"><video autoplay loop src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/index_physx.mkv" controls="controls" style="max-width: 640px;">
</video></p>
<p style="text-align: center;">The use of physx in the ECS</p>


