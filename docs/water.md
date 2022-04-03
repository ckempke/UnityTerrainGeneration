# Water

Ignoring things like precipitation, steam, fog, and ice, liquid water exists in three forms in most open-world games:  oceans, ponds/lakes, and rivers.  The exact form these take will vary, depending on the game and the player's abilities within it.   Often, water is simply a decoration, a sort of scenic, "wall" that acts to contain and direct the player.   In other games, players can swim or dive in the water, making it a full-fledged environment in its own right.   _Subnautica_ and other games in the "diving adventure" genre may take place nearly entirely underwater.

Water is usually transparent to one degree or another, which means that visually it's an _addition_ to another terrain type, not just one of its own.  (It also usually means it's a little more expensive, performance-wise, than other terrains.)

## Common Features

Up until now, our focus has been entirely on world generation and the dynamic loading of terrains.   It has generally _not_ been about the characteristics of the games that might use it: that is, we're generating the world, not defining how the player interacts with it.  That's mostly still going to be true, but water is "special" enough that we'll at least talk about game implementation possibilities.

If water is a barrier to the player--if they cannot enter it at all, or cannot enter it past walking depth--then the implementation is pretty simple:  the player will never "see" from underwater, the edges of the water can be limited like any other terrain boundaries, and all you really need is a good shader for the surface(s); many free and paid ones exist for all of Unity's shader pipelines.   For most such scenarios, the water "surface" can be a simple plane.

But if the player can swim, we're opening up a bunch of new considerations.

### Vision

Generally, we can't see as well or as far underwater as we can above it.  Natural water is often not entirely transparent -- debris, algae, cyanobacteria and the like will physically limit the distance we can see.    This is very, very similar to fog above water, and we can implement it the same way:  distance or volumetric fog, or both.   Land fog is typically white or grey; underwater fog-like effects will usually take on the colors of whatever's causing it:  muddy browns, various greens and blues, red from blood, or whatever.

Even in the absence of particles, human vision is blurred underwater.  This is because our eyes evolved for the the refraction at the surface of the eye to be with the eye itself and _air_.  (For the opposite reason, many amphibious creatures can't see well in air, because their eyes expect an eyeball/water boundary.).  Water bends light more dramatically than air, and provides more pressure on the eyeball itself.   The refraction is expecially significant if you're looking from the air _into_ water in which something is partially submerged.  Even in relatively shallow water, distances and even true positions are difficult to judge.

Ray tracing systems can work with refraction directly. Most other games tend to just ignore its effects (or "roll them in" to the fog, or assume the player is wearing a mask or goggles), because they're difficult to implement and generally deleterious to game play.  It's very unusual for a game to implement surface refraction, for example.

Air and water are both generally moving, but the effect is considerably more obvious in water.  This causes changes in density that both visibly affect refraction and eyeball pressure.  The resulting "wavy" distortion is easily simulated with post-processing if you want it.

Finally, water isn't completely transparent.  As you descend, less light from the surface reaches you, and the darker the surroundings will be.  In the real world, light is very dim by 200 meters down, and even in the best conditions completely dark by 1000 meters down.   Whether or not this matters to your game depends on whether there's any ability of your characters reaching such depths.  Note that this only applies to sunlight:  emissive surfaces (e.g. underwater lava), bioluminescence, and player-carried illumination can all bring light to the depths.

Generally, these effects are placed based on the _camera_ position, not those of the player's eyes (except in first-person where the two are coincident).   This can cause a very common effect where by placing the camera right at the surface of the water, you can often "see" underwater using the above-water lighting and visibility settings.   Solving this is extremely expensive; even AAA games tend to just live with it.

_Note: Want to solve it?  Calculate the line where the surface plane intersects the "near" plane of the camera frustrum.   Create a second camera at the same position.  Apply the underwater settings to the second camera, and render to a texture that is invisible to the underwater camera, but rendered on a plane perpendicular to the above-water camera with its top at the waterline.  Expect a drop in frame rates, since you've effectively just doubled the amount of stuff you're rendering._

### Armature Position

Generally there are three basic character postures for our player's avatar model:

__Walking__: In shallow water, they will walk, run, or stand just like they do on land.  (If they're doing it _exactly_ the same as they do on land--i.e. on top of the water itself--remember to turn off or remove the mesh colider for the water surface object!)

__Swimming:__ The normal standing posture will continue until the player reaches some depth, typically either the waist or the shoulders (shoulders is more realistic), at which point they will pivot to a "swimming" posture.   This is prone along the surface of the water.  Hands and legs tend to stay near the body or each other except during strokes.   Typical swim animations are either the Australian crawl, some sort of dog paddle, or a breaststroke.  The latter two have the advantage that the eyes remain forward looking, and are much more likely in a world where swimming is a very occasional occurrence, rather than something that people would train in.   Swimming animations tend to mostly ignore the up-down motion of real swimming in favor of "sticking" to the water surface.   It's just as two-dimensional as walking/running, just in a different positure, and with greater penetration of the "ground."

__Diving/Underwater__: In some games (for example, _Valheim_), that's all the swimming that's allowed.   But most games in which water is a traversable environment also allow underwater movement.   This is typically initiated by a sort of head-down dive.  Once the body is below water, orientation becomes somewhat arbitrary; the swimmer is effectively in a zero-gravity environment.  The arms and legs tend to be spread further away from the body and used for larger, more sweeping motions.  For humans, direction changes are fairly rapid (like a figure skater pivoting), but orientation changes (where the spine and head point, relative to "up") are quite slow (probably because they require movements that make no sense on land).

These three positions are mutually exclusive; probably different animation layers or controllers altogether.

### Surface Distortion

Except in exceptionally windless conditions, water is almost never perfectly flat.  Moving water never is.  Larger bodies of water are also subject to waves and tides from non-wind sources.   While games often present a sort of "foam" effect at the edges of water, this is relatively uncommon in the real world, especially in freshwater.    Water that moves rapidly over uneven surfaces will incorporate air and become whiter and more opaque.   Waves that reach a certain height and shape will "break," again incorportating air and scattering droplets.  Disturbed water will make characteristic expanding "rings" around the point of disturbance.

All of this is hard to model, and modelling it completely accurately in anything like realtime is well beyond desktop computers.   But it can be faked reasonably well, particularly using GPU shaders.  These are typically of two types:  reflection shaders distort the visual aspect of water; waves and patterns on the water being achieved through (in effect) moving normal maps, where changing the direction of reflections at each point can give a very convicing illusion of an uneven surface.  This illusion falls apart at shorelines, though, where the unmoving "edges" of the water destroy the effect.   Reflection shaded water doesn't lap onto beaches, or change it's depth.

Surface displacement shaders actually move the vertices of the water, generating "actual" waves.  As such, they will rise and fall above the "beach" textures sufficiently to model actual incoming waves.    These displacements are also much more convincing when viewed from almost parallel to the water line, because they're actually moving.

Available shaders can do either, and often both, of these effects.  Assuming sufficient GPU resources, they're also pretty cheap and provide reasonably good-looking water.   Want to roll your own?  There are dozens of tutorials out there on the Internet.

_Note: I talk about water surface "planes" a lot in this section, but because our players may be able to see the surface from underneath, the implementation actually uses a very thin solid (a squished cube), instead, so that it has both a top and bottom side.   You can also turn on the flag to make plane meshes visible from both sides, but I've had mixed luck with shaders handling that. 

# Seas

The first case is the (literally) biggest:  the oceans and seas that border the landmasses of our world.  While not strictly physically accurate, we can just represent this as a single "sea level" plane that covers the entire map.

This does have the downside that we can't have "dry" areas of land that are below sea level (see the page on mountain generation for some cases where that causes issues), but living with that limitation is a small price to pay for the simplicity of a single sea-level plane.

...or is it?

In all but the simplest graphical environments (maybe a very low-poly game), our water is likely to have one or more shaders on it.  That makes it relatively expensive to render compared to our other polygons, and it's not necessary for the vast majority of non-underwater terrains---basically just the seashore and the ocean.    Even if it's not drawn, though, the engine needs to spend effort culling it.

Very large polygons (and ours might be hundreds of thousands of meters along an edge) are also a pain in other ways.   Shaders and textures may or may not tile correctly on them (or require so many tilings they exceed the capabilities of the system), they block views in the scene editor while we're debugging.   They're just not on the same scale as most anything else in the world, which means they're a special case everywhere.

And then there's that pesky "sea is everywhere" problem.  It's relatively easy for us to solve in our world generation (we simply don't generate parts that are under that layer unless we want them), but what if the player is allowed to manipulate the terrain?  Or if we have caves or other underground structures?   Do they fill with water wherever they're below sea level?

We may or may not care.   If our game doesn't need any of that stuff, and is OK with the performance, the single-sea-plane thing might work fine.   If not, we can just generate a local, patch-sized sea plane for those spots we want to have ocean in.  As long as we're careful about tiling so that the edges meet correctly, we should get a decent sea from that.

We could also keep the sea plane, but just make the parts of that are under land 100% transparent.   That's harder to do correctly (again, since the scales are so different), but it's a simpler conceptual model.

## Lakes, ponds, swamps, puddles

Water (usually freshwater) accumulates above sea level in various forms.  Some, like puddles, are very transient and perhaps best handled as part of precipitation shaders.    But for larger bodies---particularly ones deep enough for a player to swim or submerge---we'll need to handle them sort of like little oceans, with some exceptions.    

There are some really deep lakes in the world: Crater Lake is over five hundred meters, and Lake Baikal is more than three times that depth.   But generally speaking, lakes aren't as deep as oceans, and we can probably accept not having depth-based light fade for simpler implementations.

Generally, we'll want to define a simple "volume" to contain the water.  The top of this volume will define the water's surface, the bottom will be below the maximum water depth, and the other edges will define the horizontal limits in (x, z) space.  In most cases, we can probably live with a scaled cube.  Recent versions of _Minecraft_ refer to such structures as _aquifers_, a name that's good enough that I'll steal it.   Unlike real aquifers, these are generally no larger than necessary to contain their body of water.

For something like a swamp, we might want our aquifer to cover the entire area of the terrain (and maybe adjoining ones, as well).  For more contained bodies of water, we can make them smaller, and even place them with the body.