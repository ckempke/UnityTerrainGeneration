# Biomes: Unity Implementation Notes

Aside from the Terrain itself, Unity supports effectively three kinds of add-ons:

## Ordinary Game Objects

You can place game objects manually or by script in the scene; they act as they do in any other scene and get no particular special handling.   We'll use this for most things that are meant to be dynamic:  monsters, treasures, weather effects and the like.    It's also necessary for objects that the player will interact with physically:  for example, large rocks that the player needs to stand on or be blocked by, as opposed to just passing through.

## Details

The _detailmap_ in _TerrainData_ allows you to place simple meshes or billboards into the scene.   They are optimized for efficient drawing (so you can have hundreds or thousands of them), aren't drawn at all beyond a dev-selectable distance from the user, and have no colliders--even if there are colliders attached to the mesh prefabs.   So these are fine for small objects, or for things like tall grass and grain that the player is meant to pass through, but it doesn't work well for things like large boulders, which need to be placed either as trees (if a capsule collider is sufficient) or as ordinary objects.

### Grass Details

Details with a render mode of "Grass" can be moved to the GPU (on all platforms other than WebGL, and custom shaders permitting), and will gain the ability to blow in the wind (but lose the healthy vs. dry coloration options.)

## Trees

Another optimized layer on the TerrainData allows placement of trees.   Tree objects are rendered as meshes near the player and billboards at a distance, with the exact functionality being dependent on the tool that created the tree.  A setting on the terrain data allows trees to generate simple capsule colliders to make the trees "solid" against attempts to walk through them.   Non-tree meshes can be placed as trees so long as they reasonably act like them (in particular, their anchor points are on the bottom) and the simple capsule collider is good enough for interactions.  If they need a real mesh collider (most things will if a player is meant to stand on or against them), they need to be placed as ordinary objects.

Unity supports two basic tree modeling sources (although you can make basically any mesh a "tree" if you want to.).  There's a built-in tree editor that makes "static" trees (no LOD/billboard support, no wind handling) and a third-party tool called SpeedTree which builds more flexible (in both senses of the word) models with wind and distance optimizations.

