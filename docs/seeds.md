# Determinism and Seeding

We have a large number of algorithms dependent on random numbers in our generation code---in reality, probably more of it's random than not.   That's great for generating varied terrain, but it becomes a problem when we need to reproduce that terrain exactly.

If a player leaves a terrain patch, then turns around and re-enters it, it would be weird if the mountains moved.   They might not notice trees being in different places, or the distribution of rocks changing, but anything bigger is going to be a problem.

If our game is multiplayer, then even the trees and the rocks need to be in the same places, or two players in the same place won't be seeing the same things.

The solution, if you'll recall back to your introductory programming courses, is seeded pseudo-random number generators.   The _seed_ initial value, which you provide, will cause the PRNG to produce the same sequence of "random" numbers in the same order.

If you're like me, you probably spent most of the time in those early courses trying to _defeat_ the seed and get different results each time (a common solution was to use the current wall clock time as the seed), but now we really want that determinism.

While doing the initial development for algorithms, though, it's often valuable to have new results every time (this gives is a sense of what the "outlier" possibilities are for our algorithms, e.g. occasionally generating a world that's entirely underwater).   And as algorithms change their consumption of random numbers, even using the same seed won't generate the same results, so we'll typically add seeding a little later in the development process.

## Determinism

Ultimately, we want all of our algorithms to be _deterministic_.  That is, we can rely on them producing the same results when given the same inputs, every time, on every computer.    That may seem a touch weird for worlds which are entirely defined by randomness, but we need it for reproducibility.

That determinism need to exist across the entire world, at every scale.   And we have the additional constraint that we generate our world, at runtime, _as needed_.   We won't always be generating terrain patches in the same order, for example.   Even if the player moves along the same path, the background patch creation may complete in different orders based on available CPU cores and other minutia, and most players aren't going to just traverse the same path over and over.

So it's not enough to have one seed.   Every task needs to have its own seed available (although it might share it with other tasks), and it needs to be able to determine and use that seed in a deterministic fashion, as well.

But let's back up a bit.  We can achieve determinism in several ways:

### Single Generation

If something is generated only once, then stored and reloaded, the determinism problem becomes much easier.    We'll do this, for example, for our Global Terrain Template.  Once it's generated, we'll write it to disk, and reload it when we reload our game.   If the seed is unrecoverable after generation, that's not all that important, because we have the results.

_Minecraft_ uses something similar for chunks that have been modified: when the player gets far enough away, those chunks are stored, and the stored version is reloaded in preference to a generated one when the player returns.   Unmodified chunks could be done the same way, but it's often more efficient to just re-create them.

More generally, there are all sorts of things for which "store and load" is a better solution:  monsters, treasure, resource nodes, etc.   These things will change as the player interacts with them, and shouldn't generally "reset" to their initial configuration every time the player re-enters their vicinity.    Even if this sort of stuff "respawns" after a time, most games will want to do that under their own control, not as a side effect of the player's movement.     They're also---compared to terrains, anyway---very cheap to store.  Keeping the location, type, and current hit points of a monster might take only a few dozen bytes.   Items and resource node states are even lighter weight.   It's not uncommon for save game files for even large worlds to be in the few tens of megabytes in size.

For all of these sorts of scenarios, the solution is generate once (possibly on the server, for multiplayer games), store, and reload as necessary.

### Positional Generation

Next up is our old favorite: generate based on position.    Perlin/Simplex noise works this way -- the functions are continuous across the entire number space, and we just sample from it based on our position.   This works well for things that are in fixed locations relative to some coordinate system:   our terrain patches, for example, can use their global coordinates as inputs.  Within a terrain, the position of details objects, rocks, trees, and the like can be relative to the terrain's coordinates, the world's coordinates, or both.

This feature of the Perlin-class noise algorithms is why they're used so ubiquitously in terrain generation.  Not only do you get the same results when you feed in the same positions, but those results are continuous across _adjoining_ positions.  So anything generated from P-noise should line up across terrain patches without requiring extensive stitching.

### Seeds

Nearly all computer pseudorandom generators these days are seeded, so for simple stuff like *RandomRange*() and the like, we're covered.   Perlin noise is _not_ a seeded algorithm--the function always produces the same values for a given position--but we can use a large offset to work as a surrogate seed.   Simplex noise _does_ allow a seed value (and has other advantages), so we'll prefer that, anyway.

But being able to seed isn't the whole solution.   Even with a seed, if we want to generate the same sequence, we need to generate exactly the same number of values after that seed, in the same order.    Which means if we, say, generate the global terrain, then patches A, B, C, D, and E without re-seeding, we always have to generate those same patches in that same order every time.

Alternatively (and more practically), we need to _reseed_ our generators every time we start a new task.  We could use the global template's seed to generate a seed for every patch, but it's probably easier just to use some inherent property like the patch's position as the seed value.

Seeded algorithms don't necessarily have the continuity-at-edges feature of Perlin noise, so if we use them to create macroscopic features, we need to be aware of how they ineract at the edges.   For a lot of stuff (placing rocks, trees, puddles, etc.) continuity at edges doesn't matter, and seeds are great for that.

