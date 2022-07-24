# Regional Locality

Our final "locality" resolution is "regional."  This is a surprisingly "busy" level of detail in landforms.

At the one extreme, any geometry features larger than maybe five or ten kilometers can be encoded directly into the global template.   Its height resolution is at the tile level, representing somewhere between 256 meters and a couple kilometers of space per height value (depending on the "Terrain Tile Size" in the world params).

On the other hand features of only a few meters can be represented entirely at the terrain patch level, so that's where we do texturing, trees, rocks, buildings, world items, etc.

But there are a lot of features in the natural world that are larger than a tile (or at least larger than our smallest potential tiles) and yet too small to accurately model at the world level.   Things like small hills, ridges, buttes, plateaus, canyons, lakes, etc need a larger canvas. 

There's another category, too:  things that would fit in a tile, but which are troublesome if they overlap the edges:  a great example are the erosion "scoops" we talked about earlier.   If there's a scoop at the edge of one tile but not the adjacent one, we get a very sharp and linear "stitching" that results in the seam lines being very obvious.

For some features, we can just force them to lie fully inside the terrain.   That works for very small things like rocks and trees (where even if they overlap a bit, it isn't visible), but as we mentioned in a previous section, doing it for large features will result in a potentially obvious "grid" of places where features aren't.

This undesired gridding is itself fractal:  we handle it at the global level by forcing the edges of the world map to water.  That's an OK "hack" at that level, _until_ we start trying to make even larger worlds by positioning adjacent "worlds" next to each other, at which point—again—the "grid" if seas breaking up landmasses might become too obvious, although the player will likely never see enough of the world to notice the pattern if you don't show them a "map" or similar bird's eye view.

We're going to have the same issue at the regional level:  if we generate "regions" (of whatever size) in a grid, and don't let large features overlap the grid, the player may or may not be well-travelled enough to notice.   But we can solve this problem with a little ingenuity, and for these mid-sized regions, we should.

## What's a Region?

For our purposes, a "region" is a grid of space larger than a tile and smaller than the world.   It needs to be large enough to handle our largest reasonably detailed features, but small enough to be manageable.  We'll start with a region of 20 tiles on each edge.

Now here's the trick for de-gridding:  we'll position the origin of one of these regions at every _tenth_ tile.   That means that the regions will overlap:  any given tile will be in two regions:  one with it's origin at `tilenum % 20`, and one with it's origin at ten tiles to one side of that in each axis.   We can draw features in any region, while staying away from the edges of that region.  The grid will be hidden by the _overlapping_ regions, whose edges will be in the center of the first one. 

When we generate each patch, therefore, the terrain height levels there becomes the sum of:

- The levels of the terrain patch itself
- The levels of the 20,40,60,80... grid region part that overlaps it, and
- The levels of the 10,30,50,70... grid region part that overlaps it.

In theory, this should solve the stitching problem:  features that overlap tile boundaries will now be drawn into one of the _regions_, and the adjacent tile will pick up its "portion" of the feature from there, too.   So we'll only get very severe edge discrepancies when we _want_ them (e.g. when an actual cliff runs along the edge of a tile).

These regions don't necessarily need to be kept at the same resolution as tiles, since to draw the player's current patch and the few surrounding it could require keeping as many as 4 of these 20x20 regions in memory at once.

For example, if our local tiles are 1024x1024 meters, we could have as many as:

1024 x 1024 x (20 x 20 sized region) * 4 regions * 4 bytes/height = ~6.7GB.

If we're willing to live with 1/4 that resolution at the regional level, we can reduce that to:

256 x 256 x (20 x 20 sized region) * 4 regions * 4 bytes/height = ~420 MB

That's pretty reasonable for holding in memory except on mobile, (and on mobile the tiles will probably be smaller, anyway).

