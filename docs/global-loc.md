# Global Locality

At the global scale, we're concerned about shapes of things on scales ranging from several kilometers to continental.  Our primary concern here is to build the _Global Terrain Template_, a height map on a one-vertex-per-patch scale that will act as the backbone structure for the local maps that follow.

Once we've done that, we can use these geological structures to create additional templates for things like moisture, temperature, wind, weather, and biomes.    If we wish, we can also use them to generate political divisions for games and simulations.

## Global Terrain Template

The GTT will be sampled by the local patch generation maps to provide the rough structure that underlies that map.   For example, the GTT "cell" on a mountainside will have a fairly steep slope to it, the sampling insures that the terrain patch generated for that cell provides that slope.   The GTT essentially provides the contract for the local patches that are created.

More details on this process are found in the [Local Locality](local-loc.md) section.   Here, we're concerned with the generation of the global terrain template itself.

The template really need only be a height map; a two dimensional array of floats whose dimensions are basically just the dimensions of our world divided by the patch size.   But, especially while experimenting, it's nice to be able to "see" that height map as whole, so we'll generate a Unity Terrain object from it that we can inspect in the scene window while the game is running.    Using a Terrain for this map also let's us build the local stuff first if we want, since we can simply use Unity's terrain editing tools to make our "template" manually.

The downside of this is that we're limited by the maximum terrain size of 4096x4096 samples, but we can live with that until we're confident in our algorithms; at 1km patches, that's a sixteen million kilometer area for us to work with.   Still, we need to be aware of scale:  for game scales, that's way more than enough, but if our application needs real-world spaces, the surface of the Earth is something like 40 times that size.   Which means our algorithms need to accept pretty significant scaling and still work--either by actual scaling up to larger maps, or by allowing recursive decomposition to make higher resolution ones.

### Step 1: Generate Region Map

As always for Voronoi maps, we'll generate a bunch of random points.   True random tends to make "clumpy" points.  I'm OK with that, but if you want them a bit more evenly spread out, you can use *Lloyd relaxation* a couple times to move them about; you can find code for it in the Delaunay library in the [references](references.md).

We're not going to just store the points "raw," though.  We're eventually going to want to mark them up in various ways, so we'll build a structure containing a point and use that.

All we're looking for here is a sort of general elevation level.   We force areas we want to be underwater to be low (usually edges, unless your map allows the player to reach/cross the edge), and just rough out a shape for the rest of it.

The algorithm, in pseudocode, is basically this:

```pseudocode
Generate Random Voronoi Regions;
Assign every pixel in the heightmap a region.
Set the level of any region that touches an edge to zero;
Maybe set a small number of other regions to zero to encourage bays, inland seas, etc.
Set the level of every other region to at most one one level higher than any region it touches.
Map the "level" to a set of heights
```

There are a lot of magic numbers in this implementation:  how many Voronoi regions to generate, actual dimensions, how many "inland" points to push down, and the like.   It also takes a fair amount of time.   On my fairly high-end system (for 2022), it takes almost a minute and numerous iterations over each pixel to produce the map.  According to the profiler, the bulk of the time is in the initial assignment of each pixel of the heightmap to it's respective region.   With a geometry-based approach and a good flood-fill algorithm, that could be dramatically improved.

But once it's done, we'll have something like this:

![Image of "terraced" height levels in Unity Terrain.](media/gen-terraces.png)

That's a 2048x2048 patch terrain with 500 Voronoi regions and five randomly suppressed points.  (Only three are visible, one was cropped in on the bottom, and another was probably an edge region that was already zero).

Were not trying to do anything here except get the rough shape of a continent or two.  In particular, those high and low points aren't really meant to be mountain ranges and valleys, we're going to compress this down into something much smoother and flatter in a moment.

But before we do that, let's play around in the editor to demonstrate how we can use this to generate rough landforms.  Here's another map:

![level 0](media/vor-sea-level0.png)

Here's that same map with a blue plane added above the first tier:

![Level 1](media/vor-sea-level1.png)

Using this, we get one large landmass with some small (well, relatively) inland "lakes."   Raise the level one more:

![Level 2](media/vor-sea-level2.png)

Our "lakes" are getting bigger, and while we've still got only a single continent, it's clearly starting to break into regions.   One more level:

![Level 3](media/vor-sea-level3.png)

Our lakes have become bays, and the "continent" has a more interesting shape.   Still another level:

![Level4](media/vor-sea-level4.png)

We've got two continents now (and an ocean-to-land ratio similar to Earth).   If we go another level:

![Level5](media/vor-sea-level5.png)

Our "continents" have become small islands in a vast sea.

Which of these (if any) is the ideal world for our game depends on the goals of the game, and in particular on whether or not sea travel (or undersea travel) is an important gameplay element.  If not, it might make more sense to increase the number of both terraces and "pushdown" lakes to break up the land "earlier" in the sea level rise path.   Here's a map with 700 regions and fifteen pushdowns, with the sea level at layer 2:

![700-15-2](media/vor-700-15-2.png)

..or layer 3:

![700-15-3](media/vor-700-15-3.png)

Either would make an interesting world for exploration.
