#  Background

This paper is an investigation into natural terrain generation in the Unity game engine, meaning environments that conform to the expected geological, botanical, and other attributes of the “real world” (or whatever world’s expectations we’re trying to render) without being directly sampled from it.

Terrain generation is applicable to a number of fields, but primarily computer games and simulations.   It’s often desirable to have “real-world-like,” yet still fictitious settings for player interaction, or be able to generate “probable” terrains for unexplored areas that we wish to answer questions about (or land a spacecraft on).

The exact usage often doesn’t matter—and Unity’s primary use is for games—so we’ll use “game” as a catch-all term for the rest of this discussion.

## Geology, Biome, and Water

A "Terrain" as we customarily think of it is made up of at least two very high-level properties.

First, we have the "Geology," which determines the physical structure of the "ground" in the terrain.  Examples include "mountains", "hills", "plains", and more exotic structures like
canyons, badlands, moors, buttes, plateaus, land bridges, caves, etc.   We can even stretch the word “natural” to the breaking point to include things like Avatar’s air-floating islands for magical worlds. In Unity's built-in terrain objects, geology is represented by the components related to the mesh: the mesh itself, as well as colliders, heightmaps, nav agents, and the like.

Second, we have the "Biome," which is associated with the (generally) biological details of an area. Examples here would be "swamp", "sea", "lake", "desert", "barren", "forest," and many more.   Unity's terrain objects include these as the "trees" and "details" components.    Note that there’s some fuzziness about “biological” here.   Things like small rock formations, scattered stones, stalactites and stalagmites, and other “small mineralogical” things aren’t biological, but they’d generally be handled in a manner similar to the locating of plants, and hence lumped into the biome.

It's common in games today to combine the two (usually by including geology as part of the biome), and certainly they're often overlapping concepts.  But keeping them separate helps is in a couple of ways. First, they're handled at very different parts of the terrain generation process.  Second, being able to combine and apply different combinations lets us re-use components to create flexibility.   For example, a relatively flat terrain might be a swamp, desert, moor, plain, wheatfield, or several other things depending on which biome it’s paired with.  Similarly, a high-altitude biome might be reasonably applied to multiple “mountain” geologies and even things like high meadows.

“Water” (or “hydrology”) is a weird component which could be considered either geology or biome, both, or entirely independent. How a game or simulation deals with it will largely depend on whether water is a "decoration" or an interactive environment of its own.   Unity itself shares this ambiguity, and has generally just given up on implementing water as a standard part of the engine in the early-2020’s versions of the engine.   We’ll consider it separately just because many implementations are likely to.

## Unity Terrain GameObjects

The Unity engine provides a high-level game object called “Terrain.”    Unity Terrain objects consist of both geology (represented as a heightmap), and biome (represented as a “trees” collection and a “details” collection).   It does not handle water directly (and since about 2019 Unity has not included the “Standard Assets” collections that used to provide the easiest implementation), so water features are imposed as separate game objects, usually as one or more geometric volumes or a simple “surfaces” at a particular altitude, depending on the needs of the game. 


