# References

I'm not the first, or the fiftieth, or the five hundredth person to walk this path.  Here are references to others who have gone before or after me.  (Yeah, yeah, I'm casual about my references.   No AP style here; this isn't being submitted for publication.   If you care about the details, go look at them!) 

## Concepts

[Polygonal Map Generation for Games](http://www-cs-students.stanford.edu/~amitp/game-programming/polygon-map-generation/) uses Voronoi diagrams to "lay out" biomes and elevations before filling them in with other techniques.   This is a fairly old and frequently-referenced article written by Amit Patel.

Andy Lo took that same technique a lot further, attempting to literally implement the stresses of continental plate interaction.  [Procedural Terrain Generation With Voronoi Diagrams](https://squeakyspacebar.github.io/2017/07/12/Procedural-Map-Generation-With-Voronoi-Diagrams.html).

Some insights into the troubles with spherical worlds from Andy Gainey:  [Procedural Planet Generation](https://experilous.com/1/blog/post/procedural-planet-generation)

A whole blog detailing one developer's descent into this rabbit hole:  [Fantasy Maps for fun and glory](https://azgaar.wordpress.com/).

Along the same lines, someone playing with not just the physical maps, but their political divisions, as well: [Generating fantasy maps](https://mewo2.com/notes/terrain/).

A little tutorial on shader-based mountain generation, by Paul Neale on [Youtube](https://youtu.be/1Ko5YZhb5-k).  This is using different software (3DS Max), bends the definition of "procedural," and it's ultimately different from the way I generated them, but it's a powerful technique.

## Implementations

An implementation of the Marching Cubes algorithm using Unity.   There are several of these available in your favorite search engine, but this ones had one of the most functional implementations as of the time I was looking.   I was originally planning on extending it for this project, but ultimately had sufficiently different requirements that it wasn't a good fit.  This is a github project reference, so it's code, not so much text.  [Marching-Cubes-Terrain](https://github.com/Eldemarkki/Marching-Cubes-Terrain)

Voronoi diagram implementations differ greatly based on what you intend to do with the results.    If you can iterate over pixels, it's pretty much trivial to generate a bitmap representation.   If you can cheaply repeat distance checks over a lot of vertices simultaneously, implementation is fairly simple (this is why Voronoi noise is such a popular shader effect for anything that has a sort of "viscous liquid boiling" effect - lava, mud pits, cauldrons, etc.).    But if you need the actual structures in geometric form: i.e. a list of vector edges, it becomes quite a bit harder.   As developers, we like it when other people do the hard stuff for us, as in this implementation by someone with the handle "PouletFrit": [GitHub implementation](https://github.com/PouletFrit/csDelaunay), [Description with some sample "how to use code"](https://forum.unity.com/threads/delaunay-voronoi-diagram-library-for-unity.248962/).

A bunch of techniques from Jacob Olsen for simulating erosion:  https://web.mit.edu/cesium/Public/terrain.pdf

Penny de Byl has a several courses on procedurally generating various things, the terrain one is here (Udemy, possibly available elsewhere, as well):  https://www.udemy.com/course/procedural-terrain-generation-with-unity/

