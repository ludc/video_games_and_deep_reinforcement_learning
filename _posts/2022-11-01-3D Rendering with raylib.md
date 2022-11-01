# 3D Rendering with raylib

 In `gymECS`, rendering is ensured by a **system** like any other ones, which will parse the `World` and execute rendering commands based on the entities and components. From the user point of view, rendering is thus controlled just by manipulating data that are in the world. 
 
 In a general way, ECS makes easy the implementation of different rendering engines on top of similar game logics. In the following, I show how we can use the [RayLib](https://www.raylib.com/) librairy, and its [Python binding](https://electronstudio.github.io/raylib-python-cffi) to make 3D rendering. 

**Note about speed**: Raylib is a library that uses function calls to execute rendering operations, which may be slow in python due to the generation of too many calls (and python is bad when calling functions). We could make use of better 3D rendering librairies (like [Panda3D](https://www.panda3d.org/)) to speed-up our rendering. For instance *Panda3D* is using python to define a rendering structure (scene graph) which is then executed directly in C, avoiding to make too many python calls at each frame.

Implementation is available in:
* `gymecs/raylib.py` for the core components, entities and systems
* `gymecs_examples/raylibmaze` for the multi agent maze


## Principles

In order to use raylib as a rendering library, the simplest way is to define new components dedicated to rendering. When these components will be attached to particular entities, then the rendering will be automatically executed. 

### Components

The first step is to define the **RayLibDrawComponent** that is the component in charge of drawing. This component is the basic class for any rendering component.

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post4_i1.png" width="150%"/></p>
<p align='center'> <em>The drawing component</em></p>

This component contains two fields:
* `activated` that will tell the system to draw or not (allowing us to desactivate the drawing whenever we want)
* `draw_layer` that is used to define in which order the components will be drawn. Indeed, the components with low values of `draw_layer` will be executed first, allowing to manage the drawing priorities (for instane, the camera has to be drawn at the beginning of each frame rendering)

This component also contains a `_draw` method that will be executed by the drawing system at execution. (Note that, since we have updated the `Component` and `Entity` classes, by conventions, components methods have to be prefixed by `_` to avoid to be considered as data)

Now, we can define different components corresponding to different assets to draw. For instance, if one want to draw a rectangle, we can define the **RayLibRectangle** component as:

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post4_i2.png" width="100%"/></p>
<p align='center'> <em>The rectangle component</em></p>

In this case, the rectangle will be drawn at a fixed position. If we want to attach this rectangle to a moving object, we can define another component:

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post4_i3.png" width="100%"/></p>
<p align='center'> <em>The rectangle component attached to an entity position</em></p>

This **RayLibRectangle_EntityPosition** component is reading the drawing position directly into the entity it is attached to (instead of a fixed position), and will thus *follow* the entity movements. 

A similar effort can be put to define components to draw the walls or the ground of the maze (see `gymecs_examples/raylibmaze`)


#### Background and Camera

To configure the 3D scene, we also need to define the background and camera into the scene. This is done by defining two other components:

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post4_i4.png" width="100%"/></p>
<p align='center'> <em>The background and camera components</em></p>

### System

In order to render the scene, we need to define a new system called **RayLibSystem** that will be in charge of executing the drawing. This sytem is generic to any game using raylib rendering. **Note that** by choosing not to execute this system in a game, you will have a game running 'headless' while executing this system at each step will execute the rendering. It is thus easy to execute the game a **super speed** by desactivating the rendering. 

The **RayLibSystem** proceeds in three steps:
* first, it identifies the entities that have rendering components. (For that, we have implemented a new method **get_components_by_type** in **World**)
* second, it orders the components by `draw_layer` 
* last, it executes the components in the right order, only if they are `activated`

The resulting code is the following:

<p style="text-align: center;"><img  src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post4_i5.png" width="100%"/></p>
<p align='center'> <em>The rendering system</em></p>

**And that's it !!!!* As you can see, adding a 3D rendering layer is very simple !!

**Important**: Since the 3D rendering has to be made at a given framerate, the **RayLibSystem** needs the use of a `_game_dt` arguments at execution time. 

## The 3D Multiagent Maze game

(see `gymecs_examples/raylibmaze/multiagent_maze_with_goal.py`)

To illustrate our new components and systems, we have updated the very simple maze game presented during the first post. To make it 3D, the modifications are the following (see `gymecs_examples/raylib_maze/maze.py`):
* We have added rendering components to the `Agent`, `Goal` and `Maze` in the game
* We have added the `RayLibSystem` in the `Game` class and we execute this system at each step
* In addition, we have added a `ControlCamera` system to control the camera

The modifications are **very few lines** to make our maze 3D. Of course, the rendering made here is quite simple, but can be easily modified to reach a good level.

<p style="text-align: center;"><video autoplay loop src="https://github.com/ludc/video_games_and_deep_reinforcement_learning/raw/main/docs/assets/post4_videomaze.mkv" controls="controls" style="max-width: 640px;">
</video></p>
<p style="text-align: center;">The Maze Game in 3D</p>


# Conclusion

I have illustrated the basic principles to use ECS for 3D rendering. Making more complex and beautiful games is now just a matter of coding and deploying the right components. Again, the clear advantage of ECS is the seperation between the game data (the **World**) and the systems, rendering being just a system like the others. Disconnecting rendering can be made by just not executing this system, while activating rendering can be made by just adding some components in the game data. Next step will be to implement realistic 3D physics... See you in a next post.










