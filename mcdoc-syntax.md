# Mcdoc Syntax
Mcdoc works off of an idea of "dispatchers". The basic syntax is as follows:
```mcdoc
dispatch minecraft:<registry>[<id>] to struct <human readable name> {
	<keys and values>
}
```
replacing each of the angle-bracket-surrounded pieces of text with something appropriate.

For example, if I had an item called `foo` that had a byte nbt tag:
```mcdoc
dispatch minecraft:item[foo] to struct MyFooItem {
	...super::ItemBase, // Ignore this for now
	foo: byte,
}
```

It's even simpler for defining a struct that describes your nbt storage:
```mcdoc
dispatch minecraft:storage["namespace:name"] to struct MyStorage {
    my_value: byte,
}
```
This describes a simple storage at the resource location `namespace:name` that looks like this:

```nbt
{
    my_value: 1b, // or any other byte value
}
```
Since you're dispatching to the `minecraft:storage` registry, Spyglass will now autocomplete types inside your
function files in your datapack that reference the storage at `namespace:name`. This is a really great way to
catch errors before they happen ingame!

## More Types
Let's show a more advanced example that introduces some new types:
```mcdoc
use ::java::util::text::Text // Types can be imported from the vanilla mcdoc

dispatch minecraft:storage["namespace:name"] to struct MyStorage {
    a: byte,
    b: string,
    c: int[],         // a primitive array - in nbt, looks like [I; 2,3,4]
    d: [double],      // a standard list - in nbt, looks like [1d,2d,3d]
    e: MyNamedStruct, // you can create other structs in your files and reference them
    f: struct {       // or just define them inline
        g: 1L,        // A constant that must exactly match (in this case, a long)
    },
    ...ParentStruct,  // Inlines all of the fields from ParentStruct into this struct - this defines a `foo` key.
    h: Text,          // This imported type is a string or text component
}

struct MyNamedStruct {
    /// This is a doc comment - note the triple slash. It will appear in the hover text when you hover over `x`.
    x: byte @ 5..12,      // A byte that must be in the range [5,12]
    y: [struct {          // A list of structs (or, analogously, nbt compounds)
        z: string @ ..16, // A string that must be at most 16 characters
    }]
}

struct ParentStruct {
    foo: (string | byte) // A union type - a string *or* a byte
}
```
Hopefully all of that is _fairly_ sensible. To work out where you can import things from like the `Text` in that example,
there's unfortunately not really a better way than just looking through the vanilla-mcdoc repo, probably specifically [the `util` folder](https://github.com/SpyglassMC/vanilla-mcdoc/tree/main/java/util). 

However, importing types isn't something you're likely to use all that much - the only other one that you're likely to use is probably
`::java::world::item::ItemStack`, which is an ItemStack like `{id: "apple", components: {...}}`

## Attributes
You can optionally specify an _attribute_ before the type of a field, like this:
```mcdoc
struct Foo {
    item: #[id="item"] string, // Restricts the value of `item` to be a valid Minecraft item id
}
```
Any registry name works inside `id`, like `block`, `function`, `loot_table`, `dimension`, `entity_type`, etc, etc.

The other attribute you may be interested in is `uuid`:
```mcdoc
struct Bar {
    my_uuid: [#uuid] int[] @ 4, // Specifies that the list of four ints is a uuid.
}
```