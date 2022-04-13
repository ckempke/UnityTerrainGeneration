# Unity MonoBehaviour vs. ScriptableObject vs. Standard Classes

In my first few weeks and months of learning Unity, I made a lot of mistakes and had a lot of misunderstandings about what base classes worked best to create my "things" on. 

This doesn't have anything to do with Procedural Terrain Generation (and in fact, if you're reading a multi-thousand-word rambling treatise on PTG, you're probably past these learning steps).    But my reading indicates there's still a _ton_ of misunderstandings about these out there, and the winds of GoogleBing can bring all sorts of readers to these shores.

So let's review.

## What are we talking about, anyway?

Unity's scripting environment is a C# framework (actually several) that provides APIs useful for game development and related tasks.    Like most frameworks, it provides "root classes" that your own classes can derive from.

When implementing a class to provide functionality in a Unity app, we generally have three choices:

- Inherit from **UnityEngine.MonoBehaviour**
- Inherit from **UnityEngine.ScriptableObject**
- Don't inherit from either (just use a "standard" C# class).

As far as the language is concerned, these are all the same:  **MonoBehaviour** and **ScriptableObject** are just C# classes that can be inherited from, like any other.    But they are used quite differently.

### MonoBehavior / GameObject

Classes derived from **MonoBehaviour** create _components_ which are added to a Unity **GameObject**.  (At a conceptual level, a GameObject is basically just an empty container for it's components.  By default, they contain nothing at all except a transform, and even that's a component.) They can define what the GameObject _is_ (Mesh, Terrain), or just add capabilities and properties to it (transforms, navagents, colliders).  There are two reasons to use a MonoBehavior:

- For any entity that has "physical" presence in a scene.  This is by far the most common usage of **MonoBehavior**.  Anything you add into the Hierarchy window is a **GameObject**, usually with **MonoBehaviour** components on it.   If your player can look at it, or through it, or more generally anything for which its _position_ in the virtual world (or screen) is important, use **MonoBehaviour**.  For example, a monster, camera, terrain, treasure chest, tree, mesh, UI Button, etc.   
- For any non-physical entity that needs the "heartbeat" of frames or the lifecycle of scenes.   If you want your object to be created when a scene loads and deleted when it goes away, or if you need to do something at regular intervals guided by _Update()_ or its ilk, use a **MonoBehaviour**.  Examples here tend to be called "managers" and implement things like network connections, quest and dialog systems, spawners and deleters of game entities, and other things that do work every frame (or every few frames) and come and go with parts of the world.

**GameObjects** (and therefore their associated **MonoBehaviours**) are created with scene entry and deleted on its exit.  Conceptually, they exist as part of the world that's represented by each scene.

It's possible (through the *DontDestroyOnAwake()* mechanism) for a game object and its **MonoBehaviours** to survive between scene changes---and there are even some good reasons why you might do so---but generally needing to do this is a sign that your design could use some work.   For example, if your managers can save and load their data, as they're destroyed and recreated, they can just be included in the scenes they're needed for and left out of others.    Or even more commonly, they would be better represented as one of the other two class types.

**MonoBehavior** interacts with the Unity Editor; adding a **MonoBehaviour** script to a **GameObject** lets you edit that behavior's public and \[SerializableField\] properties from the Unity Inspector window.   This alone is not sufficient to require **MonoBehaviour**, since **ScriptableObject** also allows inspector editing.

### ScriptableObject

**ScriptableObjects** are assets, like files, materials, images, sounds, prefabs, and the like.   They have no inherent presence in the game world, and are not tied to the lifecycle of scenes.   They are, by definition, persistent unless explicitly destroyed.   

Reasons to choose **ScriptableObject:**

- One or more instances of your class needs to exist (or at least be available) throughout the lifetime of the application.  Managers that don't need integration with the *Awake/Start/Update/FixedUpdate* sort of cycle are good candidates for **ScriptableObjects**.
- You want to allow devs or artists to create and edit them using the Inspector.  Adding the \[CreateAssetMenu\] attribute to the class will allow them to be created from the Asset menu or by right-clicking in the Project window, like any other asset.
- To define a specific set of values for entities, as a sort of "template" for instantiating later (think of it as a much more generic version of a template, or a database---see below.)

Like **MonoBehaviour**, **ScriptableObject** allows you to edit public and serializable properties from the Unity Inspector window.

## Example

### First Implementation

That last case may require an example.  Let's say we're making an RPG game with monsters: Orcs and Kobolds.   We'll need to be able to place or spawn these monsters into the scene, so they're going to be MonoBehaviors attached to a gameObject.

```C#
public class Orc : MonoBehaviour {
  public int maxHitpoints = 10;
  public int hitPoints = 10;
  public int damageDone = 5;
  public GameObject prefab;
  ... the rest of the stuff it does..
}

public class Kobold : MonoBehaviour {
  public int maxHitpoints = 3;
  public int hitPoints = 3;
  public int damageDone = 1;
  public GameObject prefab;
  ... the rest of the stuff it does..
}
```

This works, and for a small enough game, it might actually work fine.   But it has some problems:

- Almost all of the code is duplicated, except for some constant values
- If we create a new "Goblin" monster, we need to copy paste and create a new class (and since it's in code, a dev needs to do it).   That might become unweildy in an RPG with 50 or 60 different kinds of monsters.

### More Flexible Implementation

One of the uses of ScriptableObject is to allow a "template" to be created as an asset, with no new code necessary after the initial creation.    So, consider this:

```C#
[CreateAssetMenu(fileName = "MonsterSpeciesData", menuName = "MyCoolGame/MonsterSpecies")]
public class MonsterSpecies : ScriptableObject {
  public string name;
  public int maxHitpoints;
  public int damageDone;
  public GameObject prefab;
}
```

Now any time a developer, artist, or other team member wants to add a new monster type, all they need to do is right-click in the Assets folder, choose "MonsterSpecies" from the "MyCoolGame" submenu, name the asset, and define the values for that monster.  Notice that "hitpoints" isn't defined here, since it's an attribute of a particular monster, rather than all of the monsters of a given species (if you damage one orc, the others remain undamaged.)

Then, we can just have a generic "Monster" class that references it:

```C#
public class Monster : MonoBehavior {
  public MonsterSpecies monsterSpecies;
  public int hitPoints;
 	... generic monster-y behaviours ...
}
```

This ability to treat "Monster Species" as an asset, the same way you'd define a Material asset for painting a mesh, makes adding new species of monsters require no code at all, just connecting stuff up in the editor, which frees up your devs to do more development work, and enables your artists and game designers to do their work without needing the devs.

This same sort of thing used to be done using a database (and if you've got very large amounts of data, the database is still the better solution), but for small-to-medium size datasets, defining assets is easier for everybody.

Important:  Note that like all assets, ScriptableObjects created while the game is running in the editor will be removed when you end the game.   This makes it easy to "try out" a new monster race even while the game is running, but also means you'll lose your work if you forget and build an asset meant to be permanent during a run. 

Having just one MonoBehaviour class for all monsters may or may not be sufficient, depending on whether the behaviors of those monsters differ in ways that need to be expressed in code (for example, bipeds vs. quadrupeds vs. snakes may be sufficiently different in motion that actual subclasses make sense), but you'll need many fewer of them.    (And you might reduce further by defining "Locomotion" ScriptableObjects for various locomotion methods, and make them available as assets, too.)

## Generic Class

If you've got an entity that doesn't require any particular integration of Unity mechanisms at all -- say something purely mathematical, or "business logic", or AI algorithms, you can just use an ordinary C# class.  Nothing in Unity _requires_ you to derive your classes from **MonoBehaviour** or **ScriptableObject** if you don't need the capabilities they offer.   Such classes are available only from code; they have no presence in the Unity Scene Hierarchy, Project Hierarchy (except for the script files that they're defined in), or Editor.    They must be instantiated and destroyed by code, and cannot expose properties to the Inspector.

Which sounds like a lot of limitations, but really, that's just _code,_ of the kind you've probably been writing for years.  It works just fine in Unity, it just doesn't do anything special there.  It also describes basically all third-party C# libraries and APIs, unless they are specifically designed for Unity.

# Conceptually, Assets are Databases

The title of this section isn't literally true:  assets are serialized and deserialized by Unity in various ways, but they aren't actually backed by a traditional database, so far as I know.    But if your programming background is from the database world, it's a powerful metaphor for understanding the Asset library, and **ScriptableObject's** role in it.

Different databases use slightly different terminology, but generally speaking a database _table_ defines a group of related *fields* ("_columns_" in some terminology) and their types; strongly analogous to a struct or class that contains only properties and variables.   Tables are types: they define the structure and layout of data, they aren't the data itself.

A database _row_ or _entry_ is the instantiation of that: the specific set of values for one entity.   Rows are variables (often constants); they define the values for one instance of the type defined by their table.

At this very basic level, you can think of the project's Asset folder as a database:  the _types_ of assets available are the tables.  This includes the ones Unity gives you:  Material, Prefab, Textures, Sprytes, Images, and so on and on.   It also includes any that you define yourself via **ScriptableObject**.   A **ScriptableObject** class definition is effectively a database table definition.

When you actually right-click and create an asset of any type, you're conceptually adding an entry/row to that table.

Assets are traditionally used for compile-time databases:  usually constant elements that define the available options built into a game:   The materials things are made of, the logos displayed in your UIi, the kinds of monsters, the kinds of treasures, the kinds of items that players can pick up, towns and dungeons in your world, etc.    They usually don't change, or at least don't change much, as the game executes.

Assets are typically _not_ used for the sort of updated-at-runtime databases you might use to store information that changes at runtime:  character names and avatars, the "picked up" state of items, the character's actual inventory and equipment.    That sort of dynamically changing information tends to go into a save file or traditional database.    

There's no technical limitation here:  you can create and modify assets at runtime, you _could_ use them as your save game storage.  But doing so makes it very hard to "reset" the game to starting conditions, and it's not a best practice.

*This isn't a perfect analogy:  **ScriptableObjects** can add behaviors---not just properties and variables, and sometimes you'll have **ScriptableObjects** that are entirely code and define no traditional fields at all.   This often isn't true of more traditional databases.  Many don't allow "executable" entries in their tables (in many traditional usages, this could be a security nightmare), nor the inheritance **ScriptableObject** gets from its C# nature.    And most databases provide querying and combining operators that aren't needed in the ScriptableObject case because C# or .NET provide equivalents.*