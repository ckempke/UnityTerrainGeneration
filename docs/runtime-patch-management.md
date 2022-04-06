# Runtime Patch Management

Our world generation has two primary parts to it:  the creation of a large world, and the management of that world as the player plays the game.  Both are quite expensive; involving the creation or use of many Unity assets, iterating repeatedly over various attribute maps (height, textures, wind, etc.).  But in one way, they're fundamentally different.

__World Creation__ is done only once, and maybe not even that (the developer may generate and serialize a "fixed" world as part of the development process.)   It might take a long time--minutes, even.   But that creation time is predictable, will likely happen at the beginning of the game when there's other things going on (character creation/naming/selection, lobby stuff, whatever.)

__Patch Management__ on the other hand, is an ongoing process that has to happen at runtime.  It's triggered based in the player's movements around the world, so at times that are relatively unpredictable.   The patch will take some time to generate, and ideally the player should not interact with it before it's ready, but we don't want to "hang" the player waiting for it (at least not after the first initial patch.)

So for patch management, we have three initial requirements:

- Speed:  We should take as little time as possible to make the patch available.   Possibly, that might include making it available before it's completely "done," so long as we know the "fill in" won't be visible or distracting to the player (because of position, field of view, or whatever.)
- Non-disruptive:  Patches being generated should affect gameplay as little as possible--in particular, we don't want to be dropping frames where the player _is_ in order to create where they _might eventually be_.
- Predictive:  Since we'll never be able to make patch creation instantaneous, we should, whenever possible, create them before they're needed, so that by the time the player "reaches" the patch, it's ready and in place.

## Prediction

We'll start with the last category.   There's generally two kinds of movement that players perform in games:   the ordinary "walk around the world at a walk/jog/run pace" kind, and various forms of "fast travel" that we'll lump under "teleportation."

Ordinary travel isn't too bad:   assuming our player can't cross a terrain patch in less time than it takes to make several more, the algorithm is fairly obvious:  Generate the patch the player's in, then the ones immediately adjacent to that, and then the ones immediately adjacent to those...for as far out as resources or game design require.   As long as the patches are "ready" by the time the player gets there, they'll get they appearance of a seamless world.  As they leave each patch and enter another, create the "new" patches that surround them (and maybe delete existing ones that have now become farther away.)

Teleportation is more difficult.    Sometimes it'll be predictable (the user is approaching a portal of some sort that will move them to a known location), but far more often the user will be choosing the destination in some way (e.g. the classic open world fast travel map), and it won't be practical to pre-build patches at every possible destination.

A wait (in the form of a loading screen or something else) may be inevitable when teleporting.  Certain roguelike games have a built-in delay before "word of recall" type spells kick in, to prevent using them to get out of difficult situations, but generally once the teleport has been "triggered," we don't want the player doing anything until it completes.