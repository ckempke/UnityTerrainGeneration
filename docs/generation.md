# Generating Techniques

Terrain generation is hardly uncharted territory.   In fact, this project is meant to be less about breaking new ground than it is integrating and implementing techniques to make a "good game" or "good simulation," ideally one that's easy enough to use that it can be repurposed and modified for other games, other simulations, and other environments.

Let's begin by going over some of the popular techniques.

## Noise

"Noise," in the technical sense we're using here, means a continuous function in one or more dimensions, that exhibit some random or pseudo-random properties.

For one-dimensional noise, think of it as a line that "wanders" between two values (typically either -1 and 1 or else 0 and 1) in an unpredictable fashion.  "Continuous" here means that if you were drawing the line, it could be done without lifting your pencil from the paper.

Of course, computers aren't great at continuous things, and this kind of continuity is broken if you're not thinking of the function as an infinitely-divisible line over real numbers.    In real implementations, this function is "sampled" at discrete, usually fixed intervals.    So long as the intervals are small compared the to average distances over which the function changes value ("period" or the "inverse frequency", if we think of the function as a signal), an illusion of continuity is maintained.   If the interval approaches or becomes longer than the period, the resulting output will show *moire* effects or break down altogether.  This is the same effect that makes fans and propellers appear to slow, then spin backwards when seen on film.

Probably the most famous of the computational noise generation functions is *Perlin noise*.  Developed for the movie *Tron*, it's been around for decades, and won its creator an Oscar for screen effects that used it.

Perlin noise generates "slow," smooth changes in value over the X axis.    These values are typically scaled from their [0,1] range into whatever is useful for a given application, and sampled at different frequencies in order to speed up or slow down the rate of change.

Early in its existence, it was noticed that Perlin noise at certain frequencies tends to look like a natural "ridge line" when viewed edge-on.   This effect is significantly magnified by sampling the Perlin noise function at several different rates (often called "octaves" in the literature), and adding or subtracting the resulting curves.

This effect persists when Perlin noise is extended to two dimensions.   When two-dimensional Perlin noise is generated over a range of (x, y) values, it produces a sort of "lumpy" or "cloudlike" pattern (very visible if you make an image of it, with each pixel given a greyscale value corresponding to the Perlin value at that position).    Again, sampling at different frequencies and adding the results together makes the clouds "fluffier."  

If you take that two-dimensional Perlin bitmap, and treat it as a *heightmap*, where the value indicates the "height" in the third dimension, you get something that often bears a remarkable similarity to natural terrain.    Messing with the scale, frequency, number of octaves, and interpretation can produce a wide variety of terrain effects, from rolling plains through dunes, hills, and mountains.    A huge amount of the "procedurally generated" terrains you see online are created directly from Perlin noise or a similar function.

### The Problems with Perlin Noise

#### Randomness

If the solution were "Perlin noise produces great terrains--we're done," this would be a short document, indeed.   But there are some downsides.

The first is technical.   Perlin noise describes a single, fixed function.   Assuming no bugs, your Perlin noise generator and my Perlin noise generator will generate the same function every time.   If you use it to generate a terrain, it will generate the *same* terrain each run.

Pseudo-random number generation has had this problem for ages, of course, and the solution there is to introduce a *seed* value, effectively a developer-provided starting point for the sequence (often itself derived from some near-random value like the microsection portion of the current clock time).

Perlin noise is unseeded, but we can fake the seed by simply offsetting the input value by a "seed" distance in the X or (X,Y) dimensions, basically just moving the "origin" of the Perlin function somewhere else.

There's a second issue with randomness, although it's not obvious until you look at some Perlin noise results for a while.   There's a distinct "grid" to it--that is, terrain features tend to appear in horizontal and vertical lines.   Whether or not this is a problem depends on the implementation; the "grid" is generally visible only when large amounts of the terrain are visible at once, so for a game where the player spends most of his or her time on the ground, it may never be obvious.

#### Simplex noise

Both of these problems are solved in a more modern noise function, called Simplex (also developed by Ken Perlin).  First, it takes an explicit seed, so you can generate repeatable or non-repeatable output as desired.  Second, it's built over a *hexagonal* structure rather than a gridded one, and with more deviation from those directions.   This generates far fewer visible "lines" in the resulting noise.  Simplex is also somewhat more computationally efficient, so it takes less computer time to generate.

A few specific implementations and uses for Simplex noise were covered by a U.S. patent, but there are a large number of non-infringing libraries and implementations available (e.g. "OpenSimplex"), and in any case the patent expired in early 2022.  Generally speaking, it's almost always better to use simplex noise rather than Perlin.

#### Natural-ish?

The biggest problem with both Perlin and Simplex noise is that while it generates some of the shapes and structures of natural terrain, the terrains described are only superficially "natural."   A geologist looking at noise-generated terrain won't be fooled, and even laypeople will tend to find it "boring" compared to real-world terrains.  "Sharp" features like geologically young mountain peaks, deep canyons, badlands, cliffs, and other features where sudden discontinuities are common aren't produced at all by most noise-based implementations, and require a lot of tweaking to achieve even if you're trying for them.  And of course, in the typical implementations, these are generating height maps over a 2D grid, so there are never any overhangs at all.

The last problem can be addressed in part by moving to *3D* noise, which is how games like Minecraft produce their bridges, overhangs, underground caves, and other features.   But even if you ignore the blocky-ness of Minecraft, its terrains are actually *less* natural looking than 2D implementations:  it covers more of nature, but at the cost of producing a lot of things nature never could.  And this sort of terrain virtually requires an expensive voxel implementation because of it's three-dimensional nature.

In the real world, mountain ranges tend to have foothills because colliding plates ripple like shoving two pieces of gelatin together.   Canyons are carved by rivers, and often still have them in the bottom.  Desert dunes are formed by the effects and direction of wind.   "Sharp" features in climatologically wet areas will tend to crumble and produce debris fields at the bottom.  Cliffs next to oceans or lakes will be undercut over time, resulting in sea caves or just collapse.   Hurricanes and tsunamis re-shape low-lying islands.    Unstable geology falls under gravity, structures become smoother and flatter as they (geologically) age.   Hot spots in the mantle produce "arcs" of volcanic mountains, caulderas, or islands as continental plates move over them.