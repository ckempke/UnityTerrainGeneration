#  Heightmaps vs. Voxels

For our discussion, assume a three -dimensional, three axis coordinate space, with mutually orthogonal axes X, Y, and Z.    The Y axis will represent height (i.e., it’s the “up and down” axis), and X and Z represent horizontal positions.    There are numerous different representations that meet these requirements (e.g., left-handed and right-handed coordinate systems); the specific one being used doesn’t matter so long as consistency is maintained across the system (or accounted for:  some of Unity’s “map” types invert the (X, Z) values from others).

## Heightmaps

A “heightmap” is a two-dimensional array of height values (sometimes represented as a greyscale image where dark areas are ‘low’ and light areas are ‘high’) that describe the height of the geology at each (x, z) coordinate in space.    This is a reasonably compact way of storing elevation/height values; is uniform in sampling; usually compresses well; and is extremely simple to implement, explain, and use.      They also lend themselves well to machine generation; there are many simple and well-known algorithms for achieving natural-looking terrain with them.    Heightmaps (aka “elevation maps”) also exist for a wide variety of real-world scientific purposes where their overhang limitations are irrelevant, so there are a numerous sources that will let you “sample” a portion of the real world to generate realistic looking (because it’s actually real) locations for a game.

The downside of heightmaps is that they represent only a single ground level per (x, z) location.   This means they cannot be used to represent any structure that has multiple potential “ground” surfaces at a single (x,z) point:  most notably caves, but also bridges, tunnels, or even simple overhanging or undercut surfaces.   (It also wouldn’t represent building interiors, but human-constructed structures are almost always superimposed on the geology with game objects rather than being part of them, so this limitation usually doesn’t matter).

Whether or not this is a problem depends on the game.   If the game does not require such structures, heightmaps are a good choice.  Such games are numerous, and players often won’t notice the absence of overhangs among all the other details of a good-looking world.   It’s also common to simply not model interior spaces as part of the terrain at all, and simply transition the player to a new scene representing the interior space (e.g., Skyrim ).

 If there are relatively few “overhanging” objects, they can be added to a heightmap-based world by using mesh game objects to create the overhanging parts, and adding them to the world like any other object.    Conversely, recent versions of Unity allow heightmaps to define “holes” in them, to which a custom mesh to represent, say, a cave interior can be “attached.”   Both of these mechanisms are reasonably complex to implement, and while they could be automated, in practice usually require a trained modelling artist to design each instance.   Less troublesome, but still a consideration is that the “appended” structure (the meshes that describe either the overhanging object or the cave) are different in nature from the terrain object itself, and need to be handled separately for things like collision, walkability, and navigation agent (or other AI) availability.

This last point is especially important for games that implement player deformation of the terrain, usually in the form of digging.    It’s possible to limit the player to only single-height deformations (a standard “bowl-shaped” crater created by an explosion would be representable in a height map, for example).    Even digging can be limited in this way, so long as digging always removes all terrain above the dug point.   The recent (as of this writing) game Valheim does this reasonably successfully because of its generally shallow digging mechanism, but it does produce some odd behaviors on steep surfaces.

For games where digging is a primary game element (e.g. the player may be removing large hunks of the world for building, resource gathering, area accessibility, or other purposes), the limitations of heightmaps can be considerable, as the player expects to be able to cut horizontal holes in vertical surfaces.

Similarly, in games where the player can add to the surface, the inability to create overhangs will be immediately apparently.

## Voxels

If a “pixel” is a two dimensional “pixel element,” a voxel is its three-dimensional counterpart: a “volume element.”   Voxel terrains involve representing the environment as a three-dimensional array of “blocks”, in which each block is either present or absent.       This allows three dimensional structures that have as many potential “ground heights” as they have blocks, and trivially represents things such as caves and overhangs.

The canonical voxel world game is Minecraft .   The Minecraft world is made up of cubical blocks about half the height of the player, and effectively any block can be removed or replaced to solve the game’s puzzles and build whatever the player wishes.

Compared to what we think of as pixels, Minecraft’s voxels are very, very large.   That’s typical of voxel based games, because representing any significant amount of terrain with sub-millimeter sized voxels would require prohibitively large amounts of memory.   In the block-based world of Minecraft, the large block sizes are a design benefit; much of the game’s distinctive style and “feel” comes from its blocky nature.

If we wish to represent more natural-looking voxel terrain, there are obstacles that need to be overcome.     The most obvious one is the blocky nature of the terrain.

Since the voxel “blocks” need to be converted to a mesh for display anyway, we can use this conversion as an opportunity to ‘smooth’ the blocks into something less chunky.    There are well-known methods to do this, in particular the marching cubes or marching tetrahedrons algorithms.    These generate meshes more complex than the pure cubes of the underlying voxel representation, but they’re relatively inexpensive to implement because they’re primarily lookup-table based.   At a very high level, each vertex and face of a block is replaced by a new face based on the present/not present state of the bock itself and the neighboring blocks in the direction of the vertex or face.

When flat shaded, the resulting terrains would still not be mistaken for entirely natural.   Again, this could be an advantage:  System Era Softworks’s Astroneer  uses this sort of “smoothed voxel” terrain as a stylistic choice; it perfectly matches the cartoony, low-polygon aesthetic of the rest of the game’s elements.    More sophisticated shading and texturing algorithms can mask the polygonization of the terrain and make it appear realistic. 

A bigger problem is memory, or its related sibling, resolution.    

A default size terrain in Unity has 512 edges in each direction, for a heightmap (which records vertex heights, not planar heights) of 513x513 heights.   Assuming each height element is a 32-bit float, the heightmap can be represented in a little over 8MB of memory.   Not tiny by any stretch, but quite manageable for any modern computer.    The mesh generated from that heightmap will be larger:  each vertex of the resulting mesh having a position in 3D space, it’s going to be a minimum of about 24MB, and probably more once you include texture UV’s, normal maps, and all the other aesthetic niceties of the modern rendering engine.     But even if it approaches 100MB, that represents (at the commonly used scale of one Unity unit per “real world” meter) about a quarter of a square kilometer of space for the player to move around in; extending that to vision distances of maybe a couple kilometers square would still fit comfortably in two or three gigabytes of memory; even less with level of detail and mipmap optimizations.

So what about the voxel world?   If we assume our voxels are one Unity unit on an edge, we need 512 x 512 x N of them, where N is the “height range” of the terrain: the distance between the lowest and highest.  Since typical implementations use a square grid, if we assume a 512 block height range, we get about 135 million blocks.   If our terrain consists of a single bit of memory for each block (present or not present), that’s about 16MB of data for the voxel representation.   Each block when converted to a mesh will have as many as 8 vertices using the blocky Minecraft representation, or up to about 16 using the marching cubes implementation.   At 3x32 bits for each of those, we’re up to more than 1.5 gigabytes just for the basic mesh, and again some multiple of that for the other miscellanea.

That’s a naïve implementation—most or all of those vertices are shared with other blocks, and the majority of vertices won’t be present at all in non-pathological terrains because they’re inside solid areas or in the “air” and omitted altogether.   Ultimately, a voxel-based representation of the same terrain that’s produced from a heightmap will generate identical or near-identical meshes, albeit without taking advantage of any of the voxel’s overhang advantages.   But achieving these niceties takes computing time.

At a very high level, the fundamental takeaway is this:  despite their three-dimensional appearance, heightmaps are fundamentally a two-dimensional “space” (they represent a single, non-overlapping surface), and voxel terrains are fundamentally a three-dimensional one.   And the number of dimensions maps directly to the amount of computation for things like mesh generation, texturing, rendering, storage, etc.

When we get down to grinding code; heightmap computations (for “whatever”) tend to be a set of nested for loops:

```clike
For x in 0…xsize
   For z in 0…zsize {…}
```

Voxel implementations, on the other hand, tend to be nested three levels deep:

```clike
For x in 0…xsize
   For y in 0…ysize 
      For z in 0…zsize {…}
```

This extra dimension buys us the advantages of a true three-dimensional representation, but at a significant computational cost (effectively a multiplier by the size of the extra dimension).

There are a lot of ways to reduce the cost of voxel terrain operations.   Since the process tends to be of the “do some relatively simple thing on all these positions” type, large gains can be achieved on modern systems by moving some of the work to the GPU via various shaders.   But even after optimizations and across a wide range of algorithms, the extra dimension multiplier remains.

For that reason, voxel worlds tend to be generated and rendered in relatively small “chunks.”    While a Unity terrain object can easily handle a square 512 Unity units on an edge (and on beefy hardware, 4K x 4x or larger terrain objects), a 512 x 512 x 512 voxel cube would bring even a high-end 2021-era CPU to its knees.    More typical sizes are 16x16x16 (or if on GPU, 32x32x32) chunks, several of which are generated and placed next to each other to build the terrain.    The total number of chunks is then adjusted to computational capability of the hardware.

A reference Unity implementation (largely unoptimized, and not using compute shaders) I use can generate roughly a 5x5 set of chunks in realtime at decent frame rates on a midrange Intel Macintosh, or about a 7x7 set of chunks on a high end gaming PC.    Many versions of Minecraft allow you to set the number of rendered chunks in the settings, as well, to play around with what numbers a decent implementation can provide.

This breakdown into chunks provides an additional benefit.   For most circumstances, most of the time, a player avatar standing on the surface of a 512x512x512 chunk wouldn’t be able to see the vast majority of the voxels, anyway – they’d be “underground.”   There’s also no point in rendering large numbers of voxels that are all “air.”     The breakdown into smaller chunks means that we’re rendering only the voxels that are near the surface of the terrain (or just the ones in the player’s vicinity, if they’re underground).   This could result in the player being able to see a full 512x512 (or greater) terrain space, not realizing that they’re standing on a thin “shell” only one or two chunks thick.  We only need to render the deeper chunks when at least one voxel in them becomes visible.

Is that enough?   It depends on the game world, environment, and the complexity of the rest of the terrain generation.    This is where we need to start talking about size.

