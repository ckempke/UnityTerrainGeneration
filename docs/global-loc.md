# Global Locality

At the global scale, we're concerned about shapes of things on scales ranging from several kilometers to continental.  Our primary concern here is to build the _Global Terrain Template_, a height map on a one-vertex-per-patch scale that will act as the backbone structure for the local maps that follow.

Once we've done that, we can use these geological structures to create additional templates for things like moisture, temperature, wind, weather, and biomes.    If we wish, we can also use them to generate political divisions for games and simulations.

## Global Terrain Template

The GTT will be sampled by the local patch generation maps to provide the rough structure that underlies that map.   For example, the GTT "cell" on a mountainside will have a fairly steep slope to it, the sampling insures that the terrain patch generated for that cell provides that slope.   The GTT essentially provides the contract for the local patches that are created.

More details on this process are found in the [Local Locality](local-loc.md) section.   Here, we're concerned with the generation of the global terrain template itself.

The template really need only be a height map; a two dimensional array of floats whose dimensions are basically just the dimensions of our world divided by the patch size.   But, especially while experimenting, it's nice to be able to "see" that height map as whole, so we'll generate a Unity Terrain object from it that we can inspect in the scene window while the game is running.    Using a Terrain for this map also let's us build the local stuff first if we want, since we can simply use Unity's terrain editing tools to make our "template" manually.

The downside of this is that we're limited by the maximum terrain size of 4096x4096 samples, but we can live with that until we're confident in our algorithms; at 1km patches, that's a sixteen million kilometer area for us to work with.   Still, we need to be aware of scale:  for game scales, that's way more than enough, but if our application needs real-world spaces, the surface of the Earth is something like 40 times that size.   Which means our algorithms need to accept pretty significant scaling and still work--either by actual scaling up to larger maps, or by allowing recursive decomposition to make higher resolution ones.

We're going to be using Voronoi maps to local features here, so we'll need some code to generate them.   That sounds like _way_ too much work for me, so we'll use []
