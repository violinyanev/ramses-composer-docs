<!--
SPDX-License-Identifier: MPL-2.0

This file is part of Ramses Composer
(see https://github.com/bmwcarit/ramses-composer-docs).

This Source Code Form is subject to the terms of the Mozilla Public License, v. 2.0.
If a copy of the MPL was not distributed with this file, You can obtain one at http://mozilla.org/MPL/2.0/.
-->
# Prefabs

![](docs/title.png)

This chapter demonstrates how to create reusable components with _Prefabs_. We want to create a road lined
with street lamps, and we want a light switch to turn them all on and off together. As the lamps are all
identical, we will create a Prefab containing a single lantern and use it six times.

## Prefab concept

Let's do some theory first, we'll see how this works in the example in a second.

A _Prefab_ groups a piece of scene graph and a number of scripts and resources into an encapsulated
entity which can be re-used multiple times. A _Prefab_ contains everything to make its objects visible
as well as the logic and functionality required for dynamic behaviour.

Prefabs are helpful for

* creating re-usable components (think of templates or widgets), and utilizing them multiple times
* structuring your project into modules for better overview and maintainability
* re-using parts of one project in another project as [external references](../../advanced/external_references/README.md)

To use Prefabs you need to create a _PrefabInstance_ object and assign a Prefab to it. The PrefabInstance
can be used in the scene graph just like any other node. All contents of the Prefab appear inside the
PrefabInstance as a copy. If the Prefab is modified, all changes are immeditately applied to all its
PrefabInstances.

As a Prefab is a self-contained entity, its components can not be linked directly to other components.
Likewise, the copied objects inside of a PrefabInstance are read-only and updated from within their Prefab.
So how can we make PrefabInstances individual and interactive?

Lua scripts inside a Prefab which reside at the "top level", i.e. above all other parts of the Prefab, have
special meaning. Other than all internal parts of the Prefab, these scripts are not shared among PrefabInstances
and thus can have different values or links for each instance. These scripts represent the interface of the
Prefab - the properties a Prefab "offers" to whoever instantiates it. Every aspect of the Prefab which you
want to be able to control per-instance needs to be represented by an input parameter in such a script.
The outputs of these interface scripts are linked inside the Prefab, either directly to nodes or to other
Lua scripts used internally.

Note that Prefabs are a powerful feature inside of Ramses Composer for structuring your scene and keeping
in control of complex content. When you export your RAMSES scene, all Prefab hierarchies and instances will
be converted into a single scene graph. Prefab Instances are replaced by simple copies of their current
content. You will find no Prefab or Prefab Instance objects in the RAMSES scene at runtime.

## Setting up the example

We start by setting up the scene as usual: Import the meshes from GLTF, create a material and textures
and configure their uniforms. We have seen this in the __getting started__ chapter.

Here, we are using the meshes from Khronos Group GLTF [lantern example](https://github.com/KhronosGroup/glTF-Sample-Models/tree/master/2.0/Lantern/glTF) (public domain).

There is no need to think about Prefabs at this time yet, any part of the scene graph can be packaged
into a Prefab later. It is also possible to restructure Prefabs at any time.
The [next prefab example](../../advanced/nested_prefabs/README.md) will introduce more complex uses of Prefabs. 

## Creating the Prefab

Now that we can see our single lantern, let's create a Prefab around it.

To make a Prefab you first create an empty Prefab object with _Create Prefab_ in the context menu.
The Prefab will show up in the _Prefabs View_.

![](docs/empty_prefab.png)

You can now move parts of the scene graph into the Prefab by dragging or pasting them. We drag the
_Lantern_ Node in and delete the empty _Lantern.gltf_. You will notice that the lantern disappears
in the preview. Prefabs are not visible as they are just templates. Once we create an instance for
it, we will see our lantern again.

Next we add two scripts to the Prefab. We can create content directly inside the Prefab with the
context menu. There are two scripts:

* [lantern_control.lua](lua/lantern_control.lua) (named _interface_) will be our interface to the outside.
  It has just one parameter _lightSwitch_ and hands it on to the inside. With more complex parameters,
  it could also check the input for validity or transform external units into those used internally.
* [lantern_logic.lua](lua/lantern_logic.lua) (named _logic_) does the actual manipulation of the objects
inside the Prefab. It transforms our boolean light switch into a vector which is linked to the *u_EmissiveFactor*
of the shader to control the light. It also provides some static values for ambient light which are assigned to
all mesh nodes rather than setting those identical values by hand.

![](docs/prefab.png)

For this simple Prefab we could have used just one script for everything, but for more complex uses in real
life it is usually a good idea to separate the outside interface from the actual internal logic in this way.
You might even have multiple scripts on interface level which control different aspects of the Prefab.

## Instancing the Prefab

Now we want to see our lantern again. For this we create a _PrefabInstance_ with the context menu and name it
_lantern1_. Empty PrefabInstances are generic empty containers, so we need to assign the _Prefab Template_
it should use in the properties. After selecting our Lantern Prefab, you should immediately see the PrefabInstance
fill with objects and our lantern appear again in the preview.

![](docs/prefab_instance.png)

Note that all contents of the prefab instance except for the interface scripts are shown as disabled. That is
to remind you that they are just copies of the Prefab. You can examine their properties but you cannot change
the values or the structure here.

Now we create five more PrefabInstances and assign all of them to our lantern prefab. You can manipulate all
your instances just like any other node. So we move them to their positions and rotate those that go on the
other side of the road. When you manually set the _lightSwitch_ parameter on each of the instances, you should
see that particular lantern light up.

![](docs/result.png)

Finally we add the LUA script for the master light switch. *road_control.lua* justs hands on its single parameter
and acts as a sort of user interface of the scene or a substitute for a more complex logic controlling our
lanterns. Just link the interface parameters of all instances to the output of the roadControl script. When
you change its input value now, you should see all lamps coming on.

## Tips and Tricks

Be careful not to confuse PrefabInstances with their Prefabs, because of their identical structure. If you change
the input parameters of a Prefab's interface script, this will **not** effect the existing instances. You can
check the properties inside the Prefab content this way and see whether your scripts produce the intended output,
but there is no visible change in the preview as the Prefabs are abstract, invisible templates. To see an effect
in the preview, you need to change the input parameters of the instances.

Input parameters set on the Prefab interface scripts are copied as default values when creating new instances from
them. So you might safe some work by setting useful default values before creating multiple instances as they will
be initialized with those values from the start.

