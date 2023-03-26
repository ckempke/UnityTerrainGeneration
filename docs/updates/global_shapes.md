---
title: "Revisiting Global Shape"
author: Chris Kempke
tags:
  - heightmap
  - realism
...

So we've been living with a fair number of limitations in our terrains.   It's time to loop back and see if we can eliminate some of these problems.   But first, let's enumerate them and see if we can understand their root causes.

## Beaches

We'll start with one of the more straightforward issues:   the interface between land and sea isn't very good.   We get a lot of this sort of thing:

(Image)

Effectively multi-kilometer stretches of beach with shallow tide pools, followed by a very, very slow deepening into the ocean proper.    The opposite effect occurs inland; the land rises very slowly from the sea, so slowly that any terrain feature that descends below the base terrain height is likely to "fill" with water, sometimes even dozens or hundreds of kilometers inland.

There exist places like this in the real world, but they're not the norm.   Generally, beaches extend less than a kilometer inland from the water edge, and the "tide pool" zone is at most a few dozen meters.   In other places, there are almost no beaches at all; just steep cliffs that fall off directly to the sea.  

Once offshore, what happens depends on the coast's relationship to continental plates; either a gradual deepening or an abrupt falloff to great depths within a few kilometers of short.  In either case, the effect is not nearly so gradial as we've generated thus far.

The hardest part here is the "cliff" scenario:  viewed from above, a cliff is a rough, irregular line of effectively no width.   Our heightmap-based terrain is capable of representing that (although the fixed position of the coordinates means that the cliff will need to be at least one meter further out at the bottom than it is at the top).   But in the other direction, those cliffs can run for hundreds of kilometers, sometimes interrupted by small stretches of more traditional beaches.   Our current model isn't very good at generating sharp discontinuities that extend to the edges of individual patches or beyond, and our current use of heavy smoothing will tend to "round" such cliffs in any case.  (And of course, undercuts and sea caves aren't possible at all using simple height maps.)

## Stitching

Our stitching model still has bugs:  When irregular features intersect patch edges, you get ugly and unnaturally-straight "cliffs."

(Image)

There are still visible (and depending on your model, traversable) gaps at some of these discontinuities, particularly at corners, probably caused by the stitching algorithm being applied more than once to one side of a patch.  The corners of patches also tend to produce weird discontinuities.   All of these issues are largely hidden in nearly flat terrains, but it's difficult to traverse any distance in hilly or mountainous ones without encountering them; they're certainly not stable enough to build games around.

Ideally, we'd like to eliminate stitching altogether:  doing this would mean that any two adjacent patches would need to generate identical edges: each with no more information about the other patch than it's four corner heights and coordinates.   That's not as impossible as it sounds (for example, Perlin-class noise requires nothing more than coordinates across the entire floating-point space), but it's going to require some more complex algorithms than we've been using, thus far.

## Pointiness

While we're addressing stitching, it's probably time to get rid of the polygonal base shapes of our mountains.   We've been using erosion to cover it up somewhat, but it's still really obvious:

(Image)

The erosion is also causing us stitching problems (see above), so relying on it to do such heavy lifting is likely to cause us problems later.

With a very few exceptions, our mountain problem is planarity.  We've covered earlier that mountains aren't cones, but they're not pyramids, either.   Across dozens of tiles, the general shape of our ranges is fine, but once we get down to that local space, they look far too geometrical.  It's especially obvious when (like the image above), we view them from a middle distance away.

Mathematically, what we're doing when we generate our local terrains is *surface subdivision.*  We're currently using the most primitive (and cheapest) form: a linear subdivision that we then perturb with a little perlin noise and then erode the heck out of.    But there are other implementations:  non-rational B-splines (NURBS), various other Bezier subdivisions, Catmull-Clark division, etc.    Most (really, all) of these produce much "rounder" division shapes.     Incautious use of these could turn our mountains back into cones again, but they offer a fairly powerful toolset for generating a less "flat" base terrain that we can then apply our other tools to.

## Range Generation

At an even more global scale, we've still got a couple issues with our mountian range generation, that are likely to leak into other aspects of our terrain gen.   One is a simple bug:  if you generate a few hundred terrains and look at them, you'll notice that the mountain ranges tend to be clustered toward the origin; they're not evenly distributed across the surface.

The other is less subtle:  the ranges tend to be very directional along compass lines: north-south or east-west.   From far enough away, these are basically straight lines.   That's a feature of the very simple random walk algorithm we used; it tends to "correct" deviations from a straight line pretty quickly; too quickly, in this case.   It also has really only eight cardinal directions, it's not very good at movine south-southeast, for example.  Since we're using that same walk for our canyons (and we're going to use it again soon, for additional erosive features, rivers, and roads), all of those features are also going to be more directional than we actually want them.

## Scale

Finally (at least for this batch), we have a subtle scale issue.   We do things like mountain generation, smoothing, feature application, etc. across the global height map: the scale here is effectively "patches."   So a mountain range might be 200 patches long and ten wide, our canyons similarly smoothed and places based on coordinates that are basically patches.

The problem is, patch size is a variable in our generation:  they could be 256 meters, 512 meters, a kilometer, or something else on an edge.   And our mountain ranges and such are built based on them.  The net effect is that a world with 256-meter patches will have mountain ranges a quarter as wide and four times as steep as a world with kilometer patches.

This also makes it harder to see issues in implementation:  if we're using one size patch in our testing, we might not notice things becoming unnaturally steep or shallow for other patch sizes.   And we'd like the sizes of things like mountains to be either a real-world scale (for simulations), or some explicitly chosen variation on the real world (for games), not dependent on what resolution we use for their subdivisions.

## OK, Let's get fixing

That's probably enough issues for us to tackle as a next step. so let's address them.s
