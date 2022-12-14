---
title: "First thoughts on grass"
author: Chris Kempke
tags:
  - grass
...

# Grass and Ground Cover

> Note:  It's been a while since I've put anything here.   I wasn't kidnapped by aliens (that I know of); rather I have been spending much of my time on an actual game that uses this stuff.   That's currently at a place where I can start turning my attention back to the world generation itself.

We discussed ground cover a little bit in the Biomes section, but it's time to come back and look at it more deeply.

In the real world, we have grass.   It hasn't always been that way:  prehistoric dinosaurs didn't eat grass, because it hadn't evolved yet.  But these days, it's pretty ubiquitous, and even the modern dinosaurs sometimes eat it, or at least its seeds.  And where there isn't grass, there's often some other ground cover: ferns, leaves, heather, clover, or potentially hundreds of other species.

For our purposes, grass is part of the "biome," which means that it's probably a `GenerationDependentObject`, and will be selected for or against based on its relationship with the individual terrain patch.

Primarily, of course, that's moisture.   Grass grows faster (though not necessarily taller) as moisture availability increases; that's why many people water their lawns.   But wind can remove ground cover; either directly or by removing the topsoil it depends on.   With the exceptions of mosses and lichens, little ground cover grows on rocks or other exposed stone, and in places with relatively loose soil, it won't grow well on steep slopes, either.

These aren't all-or-nothing.   Over much of the moisture range, changes aren't represented by "thinner" ground cover so much as different species of it.    Even some fairly arid areas have some cover.    For moisture changes that are climatic or seasonal rather than locational, the color and thickness of the ground cover will change throughout the year even if the species remains the same.    Aside from the turning of leaves of deciduous trees in the fall, the color and health of ground cover is often the best way to identify the season or recent weather of an area by sight.

The terrain maps we generated much earlier have information for wind, water, and general erosion; the height maps can give us both local and more regional slope information.  So, given a sufficient number of meshes and textures, we should be able to generate `GenerationDependentObject` specifications for them to allow for us to select them at runtime.

## Implementation

As discussed back in the Biomes introduction, Unity gives us at least three ways to put stuff "on" a terrain:

- We can just place prefabs there like any other element of a scene.
- We can place the elements as "trees" in the tree layers.
- We can place the elements as "details" in the detail layers.

Each of these has some pros and cons.    Placing prefabs individually is the most flexible, but also the most expensive, performance-wise.  Trees can use wind zones and levels of detail, but are relatively limited in number.   Detail layers allow many more placements, but cannot usr LOD at all, and cut off completely after a certain distance from the player.

Within the detail layer, there are a number of sub-options, basically allowing us to choose billboarded textures, actual meshes, and various steps in between.   These are effectively tradeoffs of performance against quality. 

At the current moment, in late 2022, we're probably going to want to use detail-based ground cover, or else ground cover placed as "trees."

### Grass Shaders

But there's another option:   As GPUs become more powerful and ubiquitous, we can let them render the grass entirely on their own through the use of shaders.

> "Shaders" is a sort of generic, catch-all term for units of functionality that run on the GPU.  There are all sorts of different kinds:  vertex shaders move the actual vertices of models around on the GPU, allowing us to, say, upload a model once to the video card, then animate it from there using just shaders.   Texture/pixel shaders generate the pixel colors at each pixel based on the underlying meshes and input textures: all rendering in Unity is through shaders, for example.  There's one attached to every material.    There are a number of other types, as well.

Ideally, we could write a grass shader that let us specify things like the physical characteristics of the ground cover (height, thickness, "bendability", color, etc.) and just "paint" that material onto the terrain like any other texture, and the GPU would just go off and make some grass for us.

These can generate impressive amounts of grass that can do cool things like blow in the wind, automatically adjust LOD for view distances, bend away from a creature traveling through it, or even leave trails behind, and at frame rates that even simple textured quads can't hope to compete with.

You can find numerous examples of that online and in the Unity Asset Store.

But...there are catches.

The first catch is that the easiest way to do this is with a **geometry shader.** Geometry shaders, as their name implies, create geometry (in this case, whatever triangles make up each blade of grass) during the render pipeline's execution, and draw it based on information passed to the shader.

There are a number of such examples present online, and some of them are quite impressive in their functionality.

But geometry shaders are a dead technology.  They're relatively memory inefficient even in the GL world where they're most used, and mobile platforms often have trouble rendering them.    Shader experts have been recommending moving away from them for some time, and important ears are listening:  the Apple platforms (based on the Metal rendering engine) don't support geometry shaders at all, and they're discouraged on other modern platforms that use OpenGL or Vulkan.

All is not lost:   The replacement for geometry shaders is just to do the same things in a general-purpose **compute shader**, available on all your favorite platforms.   Compute shaders, as the name implies, allow us to do general purpose computing on the GPU (particularly for highly parallel algorithms, where you're doing the "same thing" across thousands or millions of pixels/objects/textures/whatever).  They're one of the underlying technologies behind DOTS and similar efforts.

So the trick here is to just create a compute shader that does what the geometry shader did:  generate some additional meshes each frame, and draw them on the GPU when requested.

There aren't very many examples of these around, and with good reason.  Computer shaders are (for this task, at least), much more complicated to write than most other shader types, requiring a level of expertise that's not as common as the other kinds (and that yours truly doesn't currently possess).   And of course, they scale in functionality with the graphics hardware they're running on; particularly on Windows and Linux, that's a fair amount of variation that needs to be supported.

But as these become better understood and more ubiquitously available, migrating to grass shaders seems like a no-brainer option.

### Hybrid

However, grass shaders are likely to remain most ideally suited to actual grass: long blades of relatively simple shapes that can be described by a few splines or polygons.   More elaborate forms of ground cover—such as ferns, flowers, and branching-stem plants—will be more difficult to render.  It's not impossible:  many of the "vegetation systems" today render their plants based on things like Lindenmayer systems (https://en.wikipedia.org/wiki/L-system), which are recursive visual "grammars" that can produce remarkably natural-looking plants.   But these are still more complex, and arbitrarily-recursive algorithms tend not to have good shader solutions, anyway.

So ideally we'd be able to use a hybrid version:  generate our base grasses with a good, general-as-possible grass shader, and supplement the other ground covers using the other options available.

