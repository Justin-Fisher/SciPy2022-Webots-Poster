# A New Python API for Webots Robotic Simulations
## Prof. Justin C. Fisher, Southern Methodist University

Webots is a popular open-source package for 3D robotics simulations.
It can also be used as a 3D interactive environment for other physics-based modeling, virtual reality, teaching or games. Webots has provided a simple API allowing Python programs to control robots and/or the simulated world, but this API is inefficient and does not provide many "pythonic" conveniences.
A new Python API for Webots is presented that is more efficient and provides a more intuitive, easily usable, and "pythonic" interface.

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
For example, an early activity in this class involves building Braitenburg-style vehicles [Bra01]_ that use light sensors and cameras to detect a lamp carried by a hovering drone, as well as ultrasound and touch sensors to detect obstables.
Using these sensors, the robots navigate towards the lamp in a cluttered playground sandbox that includes sloping sand, an exterior wall, and various obstacles including a puddle of water and platforms from which robots may fall.

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
In the new API these have quite uniformly been changed to Python properties, so these purposes are now accomplished with :code:`motor.velocity` and :code:`motor.velocity = new_velocity`.

Reduction of wordiness and punctuation helps to make programs easier to read and to understand, and it reduces the cognitive load on coders.
However, there are also drawbacks.

One drawback is that properties can give the mistaken impression that some attributes are computationally cheap to get or set.
In cases where this impression would be misleading, more traditional method calls were retained and/or the comparative expense of the operation was clearly documented.

Two other drawbacks are related.
One is that inviting ordinary users to assign properties to API objects might lead them to assign other attributes that could cause problems.
Since Python lacks true privacy protections, it has always faced this sort of worry, but this worry becomes even worse when users start to feel familiar moving beyond just using defined methods to interact with an object.

Relatedly, Python debugging provides direct feedback in cases where a user misspells `motor.setFoo(v)` but not when someone mispells 'motor.foo = v`.  If a user inadvertently types `motor.setFool(v)` they will get an `AttributeError` noting that `motor` lacks a `setFool` attribute.
But if a user inadvertently types `motor.fool = v`, then Python will silently create a new `.fool` attribute for `motor` and the user will often have no idea what has gone wrong.

These two drawbacks both involve users setting an attribute they shouldn't: either an attribute that has another purpose, or one that doesn't.
Defenses against the first include "hiding" important attributes behind a leading "_", or protecting them with a Python property, which can also help provide useful doc-strings.
Unfortunately it's much harder to protect against misspellings in this piece-meal fashion.

This led to the decision to have robot devices like motors and cameras employ a blanket `__setattr__` that will generate warnings if non-property attributes of devices are set from outside the module.
So the user who inadvertently types `motor.fool = v` will immediately be warned of their mistake.
This does incur a performance cost, but that cost is often worthwhile when it saves development time and frustration.
For cases when performance is crucial, and/or a user wants to live dangerously and meddle inside API objects, this layer of protection can be deactivated.

An alternative approach, suggested by Matthew Feickert, would have been to use `__slots__` rather than an ordinary `__dict__` to store device attributes, which would also have the effect of raising an error if users attempt to modify unexpected attributes.  Not having a `__dict__` can make it harder to do some things like cached properties and multiple inheritance.  But in cases where such issues don't arise or can be worked around, readers facing similar challenges may find `__slots__` to be a preferable solution.

### 4.2 Backwards Compatibility.
The new API offers many new ways of doing things, many of which would seem "better" by most metrics, with the main drawback being just that they differ from old ways.
The possibility of making a clean break from the old API was considered, but that would stop old code from working, alienate veteran users, and risk causing a schism akin to the deep one that arose between Python 2 and Python 3 communities when Python 3 opted against backwards compatibility.

Another option would have been to refrain from adding a "new-and-better" feature to avoid introducing redundancies or backward incompatibilities.
But that has obvious drawbacks too.

Instead, a compromise was typically adopted: to provide both the "new-and-better" way and the "worse-old" way.
This redundancy was eased by shifting from `getFoo` / :code:`setFoo` methods to properties, and from `CamelCase` to pythonic :code:`snake_case`, which reduced the number of name collisions between old and new.
Employing the "worse-old" way leads to a deprecation warning that includes helpful advice regarding shifting to the "new-and-better" way of doing things.
This may help users to transition more gradually to the new ways, or they can shut these warnings off to help preserve good will, and hopefully avoid a schism.

### 4.3 Separating `robot` and `world`.
===============================================
In Webots there is a distinction between "ordinary robots" whose capabilities are generally limited to using the robot's own devices, and "supervisor robots" who share those capabilities, but also have virtual omniscience and omnipotence over most aspects of the simulated world.
In the old API, supervisor controller programs import a `Supervisor` subclass of `Robot`, but typically still call this unusually powerful robot `robot`, which has led to many confusions.

In the new API these two sorts of powers are strictly separated.
Importing `robot` provides an object that can be used to control the devices in the robot itself.
Importing `world` provides an object that can be used to observe and enact changes anywhere in the simulated world (presuming that the controller has such permissions, of course).
In many use cases, supervisor robots don't actually have bodies and devices of their own, and just use their supervisor powers incorporeally, so all they will need is `world`.
In the case where a robot's controller wants to exert both forms of control, it can import both `robot` to control its own body, and `world` to control the rest of the world.

This distinction helps to make things more intuitively clear.
It also frees `world` from having all the properties and methods that `robot` has, which in turn reduces the risk of name-collisions as `world` takes on the role of serving as the root of the proxy scene tree.
In the new API, `world.children` refers to the `children` field of the root of the scene tree which contains (almost) all of the simulated world, `world.WorldInfo` refers to one of these children, a `WorldInfo` node, and `world.ROBOT2` dynamically returns a node within the world whose Webots DEF-name is "ROBOT2".
These uses of `world` would have been much less intuitive if users thought of `world` as being a special sort of robot, rather than as being their handle on controlling the simulated world.
Other sorts of supervisor functionality also are very intuitively associated with `world`, like `world.save(filename)` to save the state of the simulated world, or `world.mode = 'PAUSE'`.

Having `world.attributes` dynamically fetch nodes and fields from the scene tree did come with some drawbacks.
There is a risk of name-collisions, though these are rare since Webots field-names are known in advance, and nodes are typically sought by ALL-CAPS DEF-names, which won't collide with `world` 's lower-case and MixedCase attributes.
Linters like MyPy and PyCharm also cannot anticipate such dynamic references, which is unfortunate, but does not stop such dynamic references from being extremely useful.


:
:
:




[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Justin-Fisher/SciPy2022-Webots-Poster/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.


## New Python API for Webots
Justin C. Fisher

### Apache License.

Copyright 2022. Justin Fisher.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

### Warning and thanks.

This new Python API is still partly under construction and likely 
still contains many bugs. Thanks in advance for any help you can 
give in finding and quashing them. And thanks also for any other 
helpful feedback or suggestions you might give. -Justin Fisher

### Contents

The enclosed Webots project includes an early version of the 
new python api (in the folder `new_python_api`) and a number 
of sample worlds and controllers that show the new api in action 
and could serve as a model for making your own.

This readme file contains an overview of the new API, instructions 
for using the early-access version of it, some tips for users of  
the old API regarding what has changed, and some known issues. 

### Overview.

Webots has historically provided a simple Python API, allowing Python 
programs to control individual robots or the simulated world. This 
Python API was a thin wrapper over a C++ API, which itself was 
a wrapper over Webots’ core C API.  These nested layers of 
API-wrapping were inefficient. Furthermore, this API was not 
very “pythonic” and did not provide many of the conveniences that 
help to make development in Python be fast, intuitive, and easy to 
learn.  This new Python API more efficiently interfaces directly 
with the Webots C API and provides a more intuitive, easily usable, 
and “pythonic” interface for controlling Webots robots and simulations.

In qualitative terms, the old API felt like you were awkwardly 
using Python to call C and C++ functions, whereas the new API 
feels much simpler, much easier, and like it was fully intended 
for Python.  Here is a representative (but far from comprehensive) 
list of examples:  

* Unlike the old API, the new API contains helpful Python type 
annotations and docstrings.
* Webots employs many vectors, e.g., for 3D positions, 4D rotations, 
and RGB colors.  The old API typically treated these as lists or 
integers (24-bit colors).  In the new API these are Vector objects, 
with conveniently addressable components (e.g. `vector.x` or 
`color.red`) and overloaded vector arithmetic operations, akin 
to (and interoperable with) Numpy arrays.  The new API also provides
easy interfacing between high-resolution Webots sensors 
(like cameras and Lidar) and Numpy arrays, to make it much more 
convenient to use Webots with popular python packages 
like Numpy, Scipy, or OpenCV.
* The old API often required that all function parameters be given 
explicitly in every call, whereas the new API gives many parameters 
commonly used default values, allowing them often to be omitted, 
and keyword arguments to be used where needed. 
* Most attributes are now accessible (and alterable, when applicable)
by pythonic properties like `motor.velocity`.  
* Many devices now have python methods like `__bool__` 
overloaded in intuitive ways.  E.g., you can now use `if bumper` to 
detect if a bumper has been pressed, rather than the 
old `if bumper.getValue()`.
* Pythonic container-like interfaces are now provided.  You may now 
use `for target in radar` to iterate through the various targets a 
radar device has detected or `for packet in receiver` to iterate 
through communication packets that a receiver device has received 
(and it now automatically handles a wide variety of python objects, 
not just strings).
* The old API required supervisor controllers to use a wide variety 
of separate functions to traverse and interact with the simulation’s
scene tree, including different functions for different VRML 
datatypes (like `SFVec3f` or `MFInt32`). The new API automatically 
invisibly handles these datatypes and translates intuitive python 
syntax (like dot-notation and square-bracket indexing) to the Webots 
equivalents.  E.g., you can now move a particular crate 1 meter in 
the x direction using a command like 
`world.CRATES[3].translation += [1,0,0]`. Under the old API, this 
would have required numerous function calls 
(calling `getNodeFromDef` to find the CRATES node, `getMFNode` to 
find the child with index 3, `getSFField` to find its translation 
field, and `getSFVec3f` to retrieve that field’s value, then some 
list manipulation to alter the x-component of that value, and 
finally a call to `setSFVec3f` to set the new value).  

The new API is mostly backwards-compatible with the old Python 
Webots API, and provides an option to display deprecation warnings 
with helpful advice for changing to the new API.

### Early Access Installation 
This of course requires Python.  I've tried not to use any features from 
later than Python 3.6, but have done most of the testing in Python 3.9,
so would generally recommend Python 3.9 or higher. 

This of course also requires Webots.  A few recent bugfixes are relevant, 
especially if you want to make heavy use of the new supervisor/`world`, 
so it may be best to use a recent nightly build of Webots.  But for most 
purposes, the most recent stable build should probably be fine.

Eventually the relevant files for the new python API will be situated 
within the Webots installation and will automatically be available for 
import, much as the old python `controller` was.  To test an 
early-access version of this new controller, you needn't do a 
special Webots installation.  Instead you can use an ordinary Webots 
installation and just situate the relevant files for the new Python API
somewhere where your Python controller can import them.

The enclosed Webots project situates the relevant files within 
a `new_python_api` folder at the top level alongside the `worlds` 
folder, and then each particular python controller is accompanied 
by a `runtime.ini` indicating that it may import
from `../../new_python_api`, `:`-separated from the paths for any 
other folders you'd like the controller to also be able to import 
from.  E.g., the following `runtime.ini` also allows controllers 
to import from the project's `libraries` folder.

Here is the relevant `runtime.ini` line:
```
PYTHONPATH = $(PYTHONPATH):../../libraries:../../new_python_api
```

To test python controllers within a copy of this project, all you'll 
need is a copy of that `runtime.ini` alongside your controller.  

To test the new API within another project, you'll need to put
a copy of the contents of `new_python_api` somewhere your 
controller can find it, e.g. by copying this folder to the top 
level of your project alongside your `worlds` folder, and ensuring 
that your own `runtime.ini` includes that folder, or simply pasting 
those contents directly alongside your controller in its own folder 
(fine for simple trials, but won't scale well to multiple controllers.)

### Recommended Interactive Development Environment (IDE).

It is strongly encouraged to use a sophisticated IDE like PyCharm, 
as it will be able to give you type-hinting, auto-completion, and 
and mouse-over doc-strings.  You will probably need to go to project 
settings and ensure that PyCharm knows to look for the new api 
files wherever you opted to situate them.

The new python API includes thorough inline documentation
in its source code.  Eventually the Webots online docs' Python tabs 
will be updated to include this information too. But for early access, 
you'll likely want to rely a lot upon mouseover 
doc-strings, auto-completion hints, perusing the heavily-commented 
source files, looking at included sample controllers, and/or asking me.  

### Importing the new controller.

For ordinary non-supervisor robot controllers, the import is simply:

```python
import robot
```

After this the `robot` object will work quite similarly to the `robot`  
that users of the old API often made by first importing `Robot` from 
`controller` and then setting `robot = Robot()`.  (Eventually, the 
new API will include versions of `controller` and `Robot` that work 
like this for backwards-compatibility, but early-access version does 
not to avoid potential conflicts with the `controller` that will 
be present in current Webots installations.)

The `robot` module gives you access to robot devices, and other 
sorts of robot functionality.  E.g., the following simple controller
would make a motor attempt to oscillate its position.

```python
import math, robot
motor1 = robot.Motor("motor1")
while robot.step():
    motor1.target_position = math.sin(robot.time)
```

If you want to use supervisor functionality, that is now located 
entirely within the `world` module.

```python
import world
world: world.WorldModule  # this explicit type declaration helps PyCharm give better hints
```

(The second line is optional, but will help PyCharm's linter recognize 
that `world` inherits many `Node` methods and properties via `WorldModule`. 
I will eventually take manual steps to make more of these appear 
automatically which will make this explicit declaration less useful.)

Note that `world` provides only supervisor functionality, so 
if you want a controller to both exercise god-like supervisor 
powers over the world, and mere robot-like control over its 
own devices, you will need to import both `world` and `robot`.

The `world` module allows you to interact with the Webots scene 
tree as though it were composed of ordinary Python objects, 
without needing to bother with calling any functions to find nodes 
or fields, nor to hassle with VRML types like `SFVec3f` or `MFInt32` 
when interacting with the fields.

For comparison, here are two lines from Webots' sample 
`supervisor_draw_trail`, as it would appear in the old
Python controller.

```python
root_children_field = supervisor.getField(supervisor.getRoot(), "children")
root_children_field.importMFNodeFromString(-1, trail_string)
```
And here is how that looks written in the new controller:
```python
world.children.append(trail_string)
```
The `world` module also provides access to other supervisor 
functionality, and to some functionality shared with `robot` 
that is also likely to be used by supervisors, 
like `.step()` and `.time`.

```python
world.mode = "PLAY"
while world.step():
    if world.time > 10: break
world.save("myfile")
```

### Cheat-sheet for differences from the old API

#### General `robot` / `world`.

`object.getFoo()` has typically been changed to `object.foo` 

`object.setFoo(new_value)` has typically been changed to 
`object.foo = new_value`.  Take care with your spelling, as it is 
dangerously easy to silently create some new attribute rather than 
the one you intended to set. 

Any method name written in `camelCase` is now probably rewritten 
in Pythonic `snake_case` and/or given a more convenient name.

The old versions (like `.getFoo`, `.setFoo` and `.camelCase`) 
will generally still work, but will print a one-time (per run) 
warning about being deprecated, together with a suggested translation. 
(There'll also soon be an option in `settings` to hide such warning spam.)

So, in general, you could just follow the docs for the old API 
and get something that will probably work, with deprecation 
warnings pointing you towards newer, and typically better, 
ways of doing things.

The `settings` module contains some (and eventually will contain more) 
settings, many of which let you turn off a new feature for backwards 
compatibility. To alter settings, you would typically 
`import settings` and then assign `settings.setting_name = new_value`.  
Depending on the setting, you may need to do this before importing 
`robot` or `world`.

Anything that returned a vector-like list of floats now returns a 
some form of `Vector` object, whose components may be accessed via
`.x`, `.y` and `.z` (or for Colors, things like `.red`).  Vector 
objects support vector-arithmetic, like `2*v1 + v2` and provide 
convenience functions like `v1.magnitude` or `v2.unit_vector`.
If you ever want to create a vector, you can import from `vectors`
or use `robot.Vector` or `robot.Color`.

Most old functions that had many arguments that you had to list in 
the right order now have sensible default values, and let 
you use keywords to specify just what arguments you need to have 
differ from defaults. A huge offender was the old 
supervisor `setLabel` which you'll find to have been tremendously 
improved in `world.Label`.

In the spirit of Python duck-typing, most of the new API's functions 
aren't very picky regarding what sort of arguments you pass them.  
E.g. functions that expect a color will accept a `Color` vector, 
or a tuple/list of RGB(A) values ranging 0-1, or a hex integer 
like 0xFF00FF.  In general, if you think it would make good sense 
for a function to accept an argument, there's a good chance it can, 
and you can check the type-hinting for confirmation.

#### Robot Sensors.

All sensors (and anything else with a sensor-like sampling period) 
are now automatically enabled upon first access with the simulation's
basic timestep as their sampling period.  You may alter this with 
`sensor.sampling = new_period` or disable a sensor with 
`sensor.sampling = None`.

Most sensors' values can be accessed via `sensor.value` (with the
main exceptions being sensors that can provide a variety of 
different values).  However, most sensors are now "surrogates" 
for their own `.value`, meaning that you can typically use the sensor 
itself in commands where you would have used its `.value`.  
E.g. you can use `if light_sensor > 50` or `if bumper` or 
`estimated_velocity += accelerometer * robot.timestep_sec` 
or `gps.x` or `for target in radar`.  The main reason to use 
`.value` is if you want to store a snapshot of the sensor's
current value for later comparison, because at later timesteps, 
the sensor itself will have forgotten its old value and become 
a surrogate for its new value.  (For high bandwidth sensors like 
Cameras, RangeFinders and Lidar, .value shares memory with the 
simulation to allow for faster reading, but this means the 
.value will be valid only for this timestep. For these, use 
the device's .copy() method to store a lasting value, if you need it.)

#### Supervisors / `world`

For supervisors, `world.DEFNAME` will find the node with that 
DEF-name descended from the root `world`.  You can similarly 
initiate such a search from any other node, and it will recursively 
seek downwards, skipping levels when appropriate (unlike Webots' 
built-in dot-pathing which requires that you explicitly mention every 
successive level after jumping to the first). You may also search
for NodeTypes, e.g. `world.ROBOT1.Camera` or device names, e.g. 
`world.ROBOT1.camera`.  ALL-CAPS DEF-names should not collide 
with other names, but other names could.  You can also use 
`world.Node(identifier)` to find nodes, where the identifier 
could be a DEF-name, NodeType, device name, Device object, or 
unique ID integer.

Nodes' fields are accessible as `node.fieldname`. 
Referring to an SF (single) field will automatically return its
value. E.g. `world.ROBOT1.translation` returns a 3D Vector.  
SF fields are also settable, e.g. `world.ROBOT1.translation = (0,0,0)`.
Referring to an MF (multi) field will return a container object 
that lets you interact with that field as though it were 
a Python list.  E.g., `world.children[0]` returns the first child 
of the root `world`, `for node in world.children` iterates over 
all these children, and `world.children.append(new_node)` 
would import a new node at the end of this list.  If you treat 
MF Fields like python lists, they'll probably do exactly what you expect.
(If accessing fields via `node.fieldname` ever doesn't work, e.g. 
due to some name collision, you may also access, and iterate 
over fields, using `node.fields` or `node.proto_fields`)

The new API makes it quite easy to traverse fields or the tree, e.g. 
with `for f in node.fields` or `for node in world.descendants`.

The `world` module generally does a good job at caching a 
"proxy scene tree" to avoid slow repeat-lookups through the C-API. 
It's often advisable to just access things through this proxy tree, 
rather than storing your own local references to node or especially
fields.  The `world` module's caching is good enough that there's 
generally not much speed to be gained by storing your 
own local references. Personal convenience may still justify 
creating a local reference to a stable node that you'll often use, e.g. 
by setting `bot2 = world.ROBOT2`. However, (unlike the old API) 
there usually is no point in storing local references to most fields 
(i.e. usually better to use `bot2.translation` than to store 
`trans_field = bot2.fields.translation` and then refer to 
`trans_field.value`). If you'll do much deleting of nodes from the 
scene tree, then storing your own local references to nodes/fields 
will risk accidentally using one that has become outdated, which may 
crash Webots, whereas if you refer to things through the proxy scene 
tree, it automatically updates its references
(in response to changes that this supervisor itself has made -- 
if other supervisors make changes, things get a lot
messier, and you may run into some bugs deep in Webots that make 
keeping track of such changes practically impossible).

You can import a node to an SF field simply by assigning it a 
value.  E.g., `joint.endPoint = "wheel.wbo"` or 
`joint.endPoint = my_VRML_string`.  For an MF field, you would 
add a node much as you would for a python list, with `.append`, 
`.insert` or slicing, e.g. `n.children[0:2]=[string1,string2,string3]`. 
In addition to accepting .wbo filenames and VRML strings, these 
import commands will also accept existing nodes to be copied, 
or you can use `world.plan` to create planned versions of nodes
using python syntax rather than VRML, which makes it easy to 
incorporate dynamic variables into plans, or to dynamically alter 
plans, e.g., between successive imports.  
Here's a simple example from the sample `supervisor_draw_trail.py`.

```python
plan = world.plan  # for easier repeated reference
trail_plan = plan.IndexedLineSet(DEF = "TRAIL_LINE_SET",
                                 material = plan.Material(diffuseColor=TRAIL_COLOR, emissiveColor=TRAIL_COLOR),
                                 coord = plan.Coordinate(point=[initial_pos] * (TRAIL_LENGTH+1)),
                                 coordIndex = [0] * (TRAIL_LENGTH+2)
                                )
world.children.append(trail_plan)
```
### Known Issues.

#### World / Supervisor.

The `world` module is quite complete and well tested.  

Pycharm's linter isn't good at recognizing that `world` will 
inherit methods from `WorldModule` and hence from `Node`. 
I have jury-rigged a way to make some commonly used 
ones automatically appear in the linter, but have not yet done this for 
all of them, and this work-around won't show docstrings for all 
properties.  I encourage early-access users to explicitly declare 
`world: 'world.WorldModule'` after you import it to get better linting.  

Pycharm's linter will complain about many uses of the proxy scene 
tree.  E.g. it won't know that `world.ROBOT1` will dynamically 
create a Node representing the scene tree node with DEF-name `ROBOT1`, 
so will instead red-underline this and fail to give helpful 
linting advice.  A partial work-around is to explicitly type-declare 
what type of object you expect such references to produce.
(Commonly used fields, like `children` and `translation` have already 
been type-declared for you.)

#### Robot module.

The `robot` module is near-complete and largely works, but is
not yet thoroughly tested.

Basic functionality like motors and simple sensors generally 
work (though likely with a few bugs).

High bandwidth devices are not entirely implemented. Ordinary 
camera and rangefinder and lidar images and pointclouds should 
work.  I expect that camera recognition objects won't work 
entirely. Coming soon!

`robot.keyboard` is fairly well tested. I haven't had a joystick 
to test but it shares core functionality with keyboard, so hopefully will 
work.  `robot.mouse` is untested and likely doesn't work entirely.

If you use `robot.Device()` (or the deprecated `robot.getDevice()`)
to create devices, Pycharm's linter won't help you much for them, 
since it won't know which type of Device will be returned.
It's generally better to use more specific constructors like 
`robot.Motor()`.  If you detect devices dynamically, e.g. with 
`robot.devices[0]` you will also need an explicit type declaration 
to get full linting help.



