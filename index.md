# A New Python API for Webots Robotic Simulations
## Prof. Justin C. Fisher, Southern Methodist University

## Overview.
1. Webots is a popular open-source package for 3D robotics simulations.  It can also be used as a 3D interactive environment for other physics-based modeling, virtual reality, teaching or games. (See the first video below for an intro to Webots!)
2. A new Python API for Webots is presented that is more efficient and provides a more intuitive, easily usable, and "pythonic" interface than the simple Python API that has historically been included with Webots.  (See the bullet-list below for some of the features and conveniences of this new API.)
3. The history of this new API is presented including its development for an undergraduate Cognitive Science course called Minds, Brains, and Robotics.  (See the second video below of a fun robot race!)
4. Some design decisions are discussed, which likely parallel challenges that other developers face.
5. Some readability metrics are provided to support the claim that the new API is much more readable than the old.

## Quick Links.

**Download Webots:**  https://cyberbotics.com/

**The new Python API:**  https://github.com/Justin-Fisher/new_python_api_for_webots

## 1. About Webots

Webots uses the Open Dynamics Engine (ODE), which allows physical simulations of Newtonian bodies, collisions, joints, springs, friction, and fluid dynamics.
Webots provides the means to simulate a wide variety of robot components, including motors, actuators, wheels, treads, grippers, light sensors, ultrasound sensors, pressure sensors, range finders, radar, lidar, and cameras (with many of these sensors drawing their inputs from GPU processing of the simulation).
A typical simulation will involve one or more robots, each with somewhere between 3 and 30 moving parts (though more would be possible), each running its own controller program to process information taken in by its sensors to determine what control signals to send to its devices.
A simulated world typically involves a ground surface (which may be a sloping polygon mesh) and dozens of walls, obstacles, and/or other objects, which may be stationary or moving in the physics simulation.

<p align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/O7U3sX_ubGc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

## 2. Presenting a New Python API for Webots

In qualitative terms, the old API feels like one is awkwardly using Python to call C and C++ functions, whereas the new API feels much simpler, much easier, and like it is fully intended for Python.

* Unlike the old API, the new API contains helpful Python type annotations and docstrings.
* Webots employs many vectors, e.g., for 3D positions, 4D rotations, and RGB colors.  The old API typically treats these as lists or integers (24-bit colors).  In the new API these are Vector objects, with conveniently addressable components (e.g. `vector.x` or `color.red`), convenient helper methods like `vector.magnitude` and `vector.unit_vector`, and overloaded vector arithmetic operations, akin to (and interoperable with) NumPy arrays.
* The new API also provides easy interfacing between high-resolution Webots sensors (like cameras and Lidar) and Numpy arrays, to make it much more convenient to use Webots with popular Python packages like Numpy, Scipy, PIL or OpenCV.  For example, converting a Webots camera image to a NumPy array is now as simple as `camera.array` and this now allows the array to share memory with the camera, making this extremely fast regardless of image size.
* The old API often requires that all function parameters be given explicitly in every call, whereas the new API gives many parameters commonly used default values, allowing them often to be omitted, and keyword arguments to be used where needed.
* Most attributes are now accessible (and alterable, when applicable) by pythonic properties like `motor.velocity`.
* Many devices now have Python methods like `__bool__` overloaded in intuitive ways.  E.g., you can now use `if bumper` to detect if a bumper has been pressed, rather than the old `if bumper.getValue()`.
* Pythonic container-like interfaces are now provided.  You may now use `for target in radar` to iterate through the various targets a radar device has detected or `for packet in receiver` to iterate through communication packets that a receiver device has received (and it now automatically handles a wide variety of Python objects, not just strings).
* The old API requires supervisor controllers to use a wide variety of separate functions to traverse and interact with the simulation’s scene tree, including different functions for different VRML datatypes (like `SFVec3f` or `MFInt32`). The new API automatically handles these datatypes and translates intuitive Python syntax (like dot-notation and square-bracket indexing) to the Webots equivalents.  E.g., you can now move a particular crate 1 meter in the x direction using a command like `world.CRATES[3].translation += [1,0,0]`. Under the old API, this would require numerous function calls (calling `getNodeFromDef` to find the CRATES node, `getMFNode` to find the child with index 3, `getSFField` to find its translation field, and `getSFVec3f` to retrieve that field’s value, then some list manipulation to alter the x-component of that value, and finally a call to `setSFVec3f` to set the new value).

As another example illustrating how much easier the new API is to use, here are two lines from Webots' sample `supervisor_draw_trail`, as it would appear in the old Python API.

```python
  root_children = sup.getField(sup.getRoot(), "children")
  root_children.importMFNodeFromString(-1, trail_plan)
```

And here is how that looks written in the new controller:

```python
  world.children.append(trail_plan)
```

The new API is mostly backwards-compatible with the old Python Webots API, and provides an option to display deprecation warnings with helpful advice for changing to the new API.

The new Python API is planned for inclusion in an upcoming Webots release, to replace the old one.
In the meantime, an early-access version is available, distributed under Apache 2.0 licence, the same permissibe open-source license that Webots is distributed under.

## 3. History and Motivation.

Much of this new API was developed by the author in the course of teaching an interdisciplinary Southern Methodist University undergraduate Cognitive Science course entitled Minds, Brains and Robotics.
Before the Covid pandemic, this course had involved lab activities where students build and program physical robots.
The pandemic forced these activities to become virtual.  Fortunately, Webots simulations actually have many advantages over physical robots, including not requiring any specialized hardware (beyond a decent personal computer), making much more interesting uses of altitude rather than having the robots confined to a safely flat surface, allowing robots to engage in dangerous or destructive activities that would be risky or expensive with physical hardware, allowing a much broader array of sensors including high-resolution cameras, and enabling full-fledged neural network and computational vision simulations.

For example, an early activity in this class involves building Braitenburg-style vehicles that use light sensors and cameras to detect a lamp carried by a hovering drone, as well as ultrasound and touch sensors to detect obstables.
Using these sensors, the robots navigate towards the lamp in a cluttered playground sandbox that includes sloping sand, an exterior wall, and various obstacles including a puddle of water and platforms from which robots may fall.

<video src="https://user-images.githubusercontent.com/19636109/176779241-cf439202-3339-45b2-a16b-4b076bd3eaa6.mp4" controls="controls" style = "max-width: 80%">
</video>

This interdisciplinary class draws students with diverse backgrounds, and programming skills.
Accomodating those with fewer skills required simplifying many of the complexities of the old Webots API.
It also required setting up tools to use Webots "supervisor" powers to help manipulate the simulated world, e.g. to provide students easier customization options for their robots.
The old Webots API makes the use of such supervisor powers tedious and difficult, even for experienced coders, so this practically required developing new tools to streamline the process.
These factors led to the development of an interface that would be much easier for novice students to adapt to, and that would make it much easier for an experienced programmer to make much use of supervisor powers to manipulate the simulated world.

Discussion of this with the core Webots development team then led to the decision to incorporate these improvements into Webots, where they can be of benefit to a much broader community.

## 4. Design Decisions.

This section discusses some design decisions that arose in developing this API, and discusses the factors that drove these decisions.
This may help give the reader a better understanding of this API, and also of relevant considerations that would arise in many other development scenarios.

### 4.1. Shifting from functions to properties.
The old Python API for Webots consists largely of methods like `motor.getVelocity()` and `motor.setVelocity(new_velocity)`.
In the new API these have quite uniformly been changed to Python properties, so these purposes are now accomplished with `motor.velocity` and `motor.velocity = new_velocity`.

* Advantages: Reduction of wordiness and punctuation helps to make programs easier to read and to understand, and it reduces the cognitive load on coders.
* Drawback #1.  Properties can give the mistaken impression that some attributes are computationally cheap to get or set. In cases where this impression would be misleading, more traditional method calls were retained and/or the comparative expense of the operation was clearly documented.
* Drawback #2. Inviting ordinary users to assign properties to API objects might lead them to assign other attributes that could cause problems.
* Drawback #3. Python debugging provides direct feedback in cases where a user misspells `motor.setFoo(v)` but not when someone mispells 'motor.foo = v`.  If a user inadvertently types `motor.setFool(v)` they will get an `AttributeError` noting that `motor` lacks a `setFool` attribute.  But if a user inadvertently types `motor.fool = v`, then Python will silently create a new `.fool` attribute for `motor` and the user will often have no idea what has gone wrong.

Our solution: Robot devices like motors and cameras employ a blanket `__setattr__` that will generate warnings if non-property attributes of devices are set from outside the module.

An alternative solution:  use `__slots__` rather than an ordinary `__dict__` to store device attributes, which would also have the effect of raising an error if users attempt to modify unexpected attributes.  Not having a `__dict__` can make it harder to do some things like cached properties and multiple inheritance.  But in cases where such issues don't arise or can be worked around, readers facing similar challenges may find `__slots__` to be a preferable solution.

### 4.2 Backwards Compatibility.
The new API offers many new ways of doing things, many of which would seem "better" by most metrics, with the main drawback being just that they differ from old ways.  We considered three options.

1. **Make a clean break?** The possibility of making a clean break from the old API was considered, but that would stop old code from working, alienate veteran users, and risk causing a schism akin to the deep one that arose between Python 2 and Python 3 communities when Python 3 opted against backwards compatibility.

2. **Just stick with the "worse old" ways?** This has obvious drawbacks.

3. **Compromise?** We usually opted to support both, but the "worse old" way generates (optional) deprecation warnings that provide guidance for shifting to the "new and better way". This involves (temporary) redundancy but hopefully will help ease the transition.

### 4.3 Separating `robot` and `world`.
===============================================
In Webots there is a distinction between "ordinary robots" whose capabilities are generally limited to using the robot's own devices, and "supervisor robots" who share those capabilities, but also have virtual omniscience and omnipotence over most aspects of the simulated world.
In the old API, supervisor controller programs import a `Supervisor` subclass of `Robot`, but typically still call this unusually powerful robot `robot`, which has led to many confusions.

In the new API these two sorts of powers are strictly separated.

* Importing `robot` provides an object that can be used to control the devices in the robot itself.
* Importing `world` provides an object that can be used to observe and enact changes anywhere in the simulated world (presuming that the controller has such permissions, of course).

In the case where a robot's controller wants to exert both forms of control, it can import both `robot` to control its own body, and `world` to control the rest of the world.

This distinction helps to make things more intuitively clear.
It also frees `world` from having all the properties and methods that `robot` has, which in turn reduces the risk of name-collisions as `world` takes on the role of serving as the root of the proxy scene tree.
In the new API, `world.children` refers to the `children` field of the root of the scene tree which contains (almost) all of the simulated world, `world.WorldInfo` refers to one of these children, a `WorldInfo` node, and `world.ROBOT2` dynamically returns a node within the world whose Webots DEF-name is "ROBOT2".
Other sorts of supervisor functionality also are very intuitively associated with `world`, like `world.save(filename)` to save the state of the simulated world, or `world.mode = 'PAUSE'`.

Having `world.attributes` dynamically fetch nodes and fields from the scene tree did come with some drawbacks.
There is a risk of name-collisions, though these are rare since Webots field-names are known in advance, and nodes are typically sought by ALL-CAPS DEF-names, which won't collide with `world` 's lower-case and MixedCase attributes.
Linters like MyPy and PyCharm also cannot anticipate such dynamic references, which is unfortunate, but does not stop such dynamic references from being extremely useful.

## 5. Readability Metrics


:
:
:



