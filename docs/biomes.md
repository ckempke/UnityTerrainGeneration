# Biomes

In our parlance, a *biome* is the physical characteristics of a terrain that aren't its basic shape.   Practically speaking, that is:

- The base textures of the ground (TerrainLayer objects)
- Imposed structure ("shape" that's specific to this biome, like pools in a swamp, spires in a badland, dunes in a desert, cave entrances, flattened regions for buildings, etc.), particularly at the local or regional level.
- Vegetation (grasses, bushes, trees) and small mineralogical (rocks, stalactites) details
- Water and water behavior.

Buildings and other sentient-being-constructed things may also apply here.   Let's ignore that for the moment, since it rapidly leaves "terrain generation" and heads toward "game design."

We're also going to ignore water-mostly.   Some of it we can treat like any other biome, placing corals, rocks, kelp "trees," and the like.   But water has all sorts of other gameplay effects (gravity, visual distance, opacity) and visual characteristics (surface appearance, caustics, light filtration) that are going to merit their own treatment later.

## Overlap

Early versions of Minecraft used to have fairly strict delineations between biomes:  you'd be in the desert, and a dozen steps later you'd be in the plains.  There was a little bit of "mixing" right at the edges, but for the most part where one biome ended, another would begin.

Real worlds aren't like that (and as Minecraft has evolved, it's started blending, too).  Changes between biomes are often so gradual that a traveler doesn't really notice them at all.  And even in the middle of, say, an arid grassland, there may be the occasional patch of scrub forest from the a neighboring slightly wetter biome.   There are also places where biome changes _are_ sudden, usually because of the presence of either drastic elevation changes or water:  a desert oasis or the green vegetation around a river or pond in an otherwise dry landscape, for example, or the even starker "tree line" on mountains above which nothing grows.

Part of this is just thinking of "biomes" as being discrete things: no two deserts are the same, and no two places in the _same_ desert are generally the same, either.   Biomes, in the computer generation sense, are a shorthand meant to increase practicality, at some cost to realism.   But even so, our model shouldn't think of a place as having "a biome", but rather being affected by "this set of biomes" to different degrees.