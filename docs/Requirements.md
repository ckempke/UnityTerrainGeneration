# Requirements

If the goal is to build something, it helps to know what we're trying to build.   I've called this section "Requirements," but this is an experimental project, so what we're listing here are really "aspirational goals."   Not all of these may be achievable, reasonable, or even technically possible.   But we still want to describe what an "ideal" system would look like.

For the moment, we're going to consider just Geology (that is, just the shape of the terrain, not what's on/in it.

## Target Software Platform

Unity3D initially.   Presumedly many of the concepts would move to other engines (particularly Unreal), but we're going to take advantage of whatever technologies Unity offers us without much concern for whether or not they're portable.

## Scalability

Our first requirement is that this system be scalable to different hardware.    Our specific target is 90/90/90: 90 frames per second 90% of the time on 90th percentile gaming hardware (that is, a PC gaming system which can outperform 90 percent of all PC game systems.)

Why exclude 90% of PCs?   First, because the tail on systems is very long:  [See Steam's System Surveys here](https://store.steampowered.com/hwsurvey/Steam-Hardware-Software-Survey-Welcome-to-Steam) for samples.   The bottom half or so of these systems aren't playing modern games much.  But more importantly:

- That's more or less where my current system sits, so it's the most practical one to target.

- We're building engine-level code rather than a game, so even if this gets widely adopted, there's a long lead time between "now" and when the first games will be available, and the the speed of game systems increases quickly.

However, we'd like the system to scale to lower level hardware; we'd like at least 60fps on current generation Unity-capable consoles or Apple Silicon Macs, and ideally at least 30fps on late model Intel Macs and reasonably capable mobile devices (say, everything iOS and the "top tenth" of Android - basically flagship class devices, not the sub-$250 ones that make up the bulk of the Android world).

## World Size and Variety

Ideally we'd like to be able to generate infinite worlds.   As an initial goal, we'll aim for a generation of a one million square kilometer area (1000x1000 km) as the proof of concept.   Our generated space should contain as high a variety of geomes as possible--at least sea, mountain, hills, plains, and riverbeds.    We will include "sea level" water, but for the moment not worry about higher elevation, emitted, or moving water (any lakes or rivers will be "dry"), since Unity's support for such things is in flux right now.  

The world should be capable of having undercut erosion (i.e. overhangs and natural bridges) and cave systems of arbitrary complexity, but we'll assume that these systems are (to within an order of magnitude or so) similar in number to their real-world counterparts.   That is, such features will be relatively rare; comprising a very small percentage of terrain space.

## Configurability

The generation of worlds should allow the generator to control limits and biases.  For example, there should be options to set the height of the highest mountains and the sea level, determine whether undersea features are present or not, determine the frequency of biomes/geomes in the world, and similar things.    In addition, we'll want to allow the option to "gamify" the environment, things like:

- Number of caves increased, and caves made larger so as to be traversable by walking humans.
- A tendency for caves or spaces behind waterfalls
- An increase in (or even mandate for) natural "switchbacks" and "collapsed slopes" that allow access to most terrain without climbing or high-jumping mechanics (e.g that you can "walk" to most of the world, even mountain peaks and the like).
- Vertically spaced "handholds" for most steep surfaces (for games with climbing mechanics).
- More regions of increased verticality in general (canyons, badlands, cliffs, other eroded or uplifted spaces)
- An increase in "dead end" type terrains:  places with few or even just one point of entry.  For example: deep canyons, calderas accessed through only one path or cave system, mountain valleys, etc.
- Region separation: natural boundaries that separate the world into mutually inaccessible (or difficult to access) regions.    This allows the game to control player progression into "higher level" areas over time.  Such boundaries will suppress the handholds/switchback options in the boundary areas.

All of these should be able to be independently turned on or off; when they're all disabled, the worlds will tend toward the "natural," independent of player accessibility.

