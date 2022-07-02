# Programmer accessibility

All of this work isn't of much value if it can't be used in a real game or simulation, so we're at the point where we need to think about how our eventual (developer) users are going to want to access our world tools.

## Use Cases

Specifically, here are the things we anticipate that our users will want to do, and the tools that either exist or we need to write in order to enable them.   Since "anticipate" is infamously subject to being wrong, I've been building a game (or at least the shell of one) to go with this.   That gives me at least an indication that we've covered enough API space for at least one user.

### Points of Interest 

Since we're not generating anything other than the terrain itself, the game dev is probably going to want to supplement the generation with some of their own:  in particular, placing cities, towns, dungeons, airports, or whatever their particular game requires as points of interest.

As with the terrain generation itself, this will likely need to be done at several levels of detail.   A million square kilometer region of the United States would likely contain thousands of municipalities of various sizes (most of which a player will never see), each of which with between dozens and tens of thousands of individual buildings.     Pre-baking all of that would (as per our usual refrain) be prohibitively large -- much larger than is going to be storable on anything other than a server farm.

On the other hand, the biggest "cities" are likely to be anchors for things like major roads.   And while cities don't _cause_ rivers, they tend to locate along them, so depending on how you build things, one will dictate some of the positions of the other.  (I'm going to use "cites" here as a shorthand for any collection of people, from a hamlet up to a real city.)

Statista.com has some data available online about the sizes of "incorporated places" in the US in 2019.  I don't have the rights to reproduce the graphs and such here, but the takeaway is that there were ten cities of a million people or more in 2019, and at each size class below that ("size class" being about a halving of the city size), the number of them is roughly 2.5 times the number of the larger class.   This pattern continues until you get to the smallest (under 10K people) cities, at which point it jumps more 10x over the "10-25K people" category.    Cities of fewer than 10K people make up about 80% of all incorporated places.      The same pattern applies to more densely populated countries like India or China, although the actual numbers will of course vary.

That's a modern, industrial, high-population world.    During the time of, say, Parthia, Kushan, the Roman Empire, and the Han Dynasty (the original "Silk Road" empires) the populations would be much more "clustered," but also much smaller (The population of the entire planet during that period was considerably smaller than the US alone, today.).  Games set in fantasy worlds probably have an even smaller population:  the infamous eight-person "city," one of which has rats in the cellar that need killing.

From the caller's level, there are basically two points of access to the world for city generation:  

1. The entire world at the terrain template level of detail (e.g. one "height" value per terrain tile, with the associated generation parameters like wind, moisture, land/sea, steepness, etc.)
2. The terrain patch handler, at patch load time.

### World Generation

Of course, the first thing they'll want to do is actually generate a world.   By making the World Generation Params a scriptable object, they can configure it in the editor to their liking, and then make a call to **GenerateWorldAsync**() to invoke it, mostly on a background thread.

It's not clear that we need to give them further hooks here: all they really care about is "when is it done," which can be handled by a simple _await_.   Once that's completed, all of the construction arrays (heights, moisture, etc) and the terrain template itself will be available for them to place whatever they need.

In my test game, I take this opportunity to generate a player starting city, a dozen or so "large" city locations, a few ports, and a few "fortresses."   

All of these are done by walking the terrain template and looking for specific criteria:

- Starting village: I look for a "flat" area that's close (a few tiles) to a "steep" one.  This puts the player near two distinct "geomes" right at the beginning.  Given the size of the world, significant travel might otherwise be necessary to actually see any variable base topography.
- For cities, I look for "flat" areas that aren't particularly wet or dry.  (I'd also add "riverside", except that as of this writing, rivers aren't implemented, yet.).  Basically boring plains areas where we can spread out, since I intend to put large cities at these locations.
- For ports, I look for land that's within two tiles of sea.
- For fortresses, I look for steep terrain (given the fantasy genre, "mountain fortresses" are a standard trope.)

This sort of "criteria-based location finder" is going to be generally useful to devs, so I'll likely move the "Location Manager" object that's currently into my game into the world generation library.

The habitations created here will be serialized and stored, to sort of act as "permanent beacons" in the world.

### Patch Loading

Even with the permanent locations chosen, that constitutes a few motes of humanity(?) in a vast wilderness.   Most games are going to want more civilization than that, and for the smaller cities (as well as more quest-focussed stuff like ruins, dungeons, wayside inns, etc.), we'll need to generate them at patch load time.

These will likely be transient.  Depending on the game, they may only get serialized once they become "important" in some way (the player visits them, a quest or map references them, etc.).  

Alternatively, we can use a seed based on the terrain tile to generate the locations in (and around, see below) that tile.  This allows us to always generate the same "random" locations for the tile, and serialization of the actual locations becomes unnecessary (changes made to them will still need to be saved.)
