# Performance

We interrupt our regularly scheduled program for some notes on performance, because if you've been trying to duplicate my code and algorithms (or actually USING my code and algorithms; contact me if you want the current source), you're probably getting slammed by it about now.

## Unity's expectations

Although built on C# and .NET, Unity's docs don't generally make much mention of the standard multithreading/asynchronous models.

First, be aware that despite the absolute silence from the docs, *async* and *await* work just fine in Unity.   If you're already familiar with them, they may be all you need.    To this, Unity adds at least three different models for unblocking the all-important main thread:   Coroutines, Jobs/Entity Component System, and Shaders.

Shaders are different in kind from the others: they move work from the CPU to the GPU, and are particularly useful for things where you're performing essentially the same calculation over many, many entities:  computing colors by applying a function to every pixel every frame, computing waves by calculating distortion maps, moving entities by applying a function to ever vertex, etc.   They're not a generic mechanism for doing "background" work; rather they solve certain common problems _extremely_ efficiently.    So we're not going to talk about them right now.

To begin with, Unity's main unit of computation is the _frame_.  That's a little unusual if you come from a non-game development world, but it makes sense for games.   Everything in _MonoBehaviour_ is built around execution in a specific order, generally once per displayed frame.    For most code that's in one of the _Update()_ or similarly-named methods that get executed every frame (or at least on a cadence similar to frames, like _FixedUpdate()_).

Unity is also very, very main-thread focused.   Almost none of it's API can be called anywhere except the main thread (at least not successfully). 

All of this leads to a model where Unity expects work to be done in very small chunks, per frame, on a single thread.    So nothing that goes in the _MonoBehaviour_ implementations should take more than about 1/60 of a second _at worst_, and modern games are aiming for more than twice that frame rate.

With my "straightforward" implementations, generating a new terrain patch can take up to about a second, depending on size, and we're still just generating shape.    Generating the initial world's Global Terrain Template can take _minutes_.

Both work, even when done in _MonoBehaviour_ contexts.   But I'm stretching the definition of "work" here -- the game basically freezes at startup until the GTT is complete, and freezes for a second or two every time a new terrain patch (or more than one) needs to be created.   It's fine for this sort of initial research and experimentation, but it would be unacceptable as shipping code.    And it's going to be unacceptable for us, pretty soon -- this is only going to get worse as we add functionality.

This is a good time to start working on performance, because if we're doing something _wrong_ in a way that can't be moved to a more performant mechanism easily, we're just making more work for ourselves the longer we wait.

## First things first

Does our code really need to take so long?   I'm a big believer in straightforward, naïve algorithms for experimenting.   Simple code means fewer bugs and more time to spend on the fun stuff.

As an example, my Voronoi diagram code (to assign each pixel a region) basically looks like this:

```pseudocode
for each x position
   for each z position
       for each voronoi point
            {
            calculate the distance from (x,z) to that point
            if it's the smallest we've seen so far, remember it as "smallest"
            }
set the region for (x,z) to smallest
```

That works great, always gives the right answer, and is really simple to write; the actual code's only a little longer than the pseudocode here.    But _Vector2.Distance()_ is actually pretty expensive, and we're creating vectors constantly from both the test point at the Voronoi points in order to call it -- that allocation is moderately expensive, too.

Here's a different way to do it:

```pseudocode
radius = 0
repeat until no pixels are assigned during a pass (or equivalently, all points are "finished")
{
	for each voronoi point that's not "finished"
	   {
	   Find every pixel in a "ring" around the point at radius
	   if that pixel hasn't been assigned yet, assign it to this voronoi point
	   if all of them were already assigned, this voronoi point is "finished"
	   }
	increase radius by 1
}
```

This never calls _Distance()_ (or makes any vectors) at all.   It's not a perfect algorithm; if two points are expanding into space and meet in the same iteration, the lower numbered voronoi point will always "get" that pixel, so there's a very slight bias to the lower-numbered points.   The "ring" calculation is complex and needs to not miss any pixels, so the actual implementation replaces it with an expanding square instead of a circle, which is much easier to calculate--and has even more small bias in favor of the lower-numbered points.   But we're using this to make a rough map in the first place, and the Voronoi point positions are randomized in space, so those errors aren't a problem for this use.

And while it may not be obvious from the pseudocode, the second algorithm is much, much faster than the first.   For 400 Voronoi points on my system, it lowers the cost of the GTT from 2.5 minutes to 18 seconds.    And the benefits get higher as you increase the number of Voronoi points:  it takes the new algorithm about 25 seconds to do 900 points, but over ten minutes for the first one.

Huge gains like this aren't actually that uncommon in code like games where you're often iterating repeatedly over huge numbers of pixels.   So it's worth a look to find out if there's an optimization somewhere that can bring your computational costs down.    The profiler is a good place to look for this; it's a bit complex to use in Unity, but it's time well spent.   You're going to need it, eventually.

## Two kinds of Asynchronous

Even after our optimizations, though, we've still got several hundred or thousand times too much work to do in a frame, and it doesn't seem likely that any amount of algorithmic tweaking is going to bring us down into the right range.     So we need to look at ways to keep our game performant while still getting that work done.

The two major tasks we've concentrated on thus far are fundamentally different from each other:

__Global Terrain Template__ generation is done once at startup, and it needs to be completely finished before the rest of the game can do _anything_.   Until there's a world that exists, there's nothing for the player to explore.   So some amount of time needs to pass here.   We can toss up a "Loading..." screen and just let GTT generation hang the game on the first frame until it's done, but it would be better to let the player do something else (customize their avatar, watch a cut scene, configure a network match, whatever) while it's happening, and just not let them go "in world" until we're ready.   On subsequent runs loaded from a saved game, the GTT will presumably be loaded as part of the game data and not need to be regenerated at all.

A few __Terrain Patches__ need to be present at startup, too -- at least the one the player is standing on.  But mostly, these will be generated as the player moves around the world, in sets of three to five (or so, depending on generation radius) at a time.   After the initial ones, Terrain Patches are generated while the player is still a full patch or more away from them.  The player can only move so fast, so if these take a few hundred--or even a few thousand--frames to be generated, we're still good.    These will be an ongoing cost as the player plays the game; they will likely still be generating new patches a hundred hours into the game.

So our goal in both places is to spread the computation out over many frames, without preventing the other things that are going on in our game from happening.    And there are essentially two ways to do that:

- We could do a little bit of work each frame, and then give up ("yield") the rest of that frame's computational time to the rest of the game.   In this model, everything would stay on the main thread, both our computation and the rest of the game's could use Unity APIs whenever they wanted, and the time taken would be determined by the total amount of computation and the single-threaded performance of the hardware.    This is the model implemented by the simple _async/await_ without Tasks, and by Unity's Coroutines.
- We could offload the work entirely to another thread or process, which on most modern systems means moving them to another real or virtual/hyperthreaded CPU core.   This will be faster in absolute terms for both the expensive computation and the rest of the game -- they're effectively not competing for resources at all, and don't even need to interact until the expensive computation is done and the results are ready.    Unity APIs would be mostly unavailable to the expensive computation, which often isn't as much of a penalty as it seems, since these are generally mathematical and data-structure oriented.   This is the model implemented by _async/await_ combined with _Task.Run()_, and by the Unity Jobs System.     There's also a variant on this where you run essentially the same code simultaneously on many threads/cores with different inputs (say, generating 50 terrains at once) using _ParallelForTask_, but such routines are usually better suited to shaders on modern systems, and scale poorly between devices with varying numbers of cores.