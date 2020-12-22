# Data Serialization

- [Available types](#available-types)
- [Planned types](#planned-types)
- [How it works](#how-it-works)
- [What doesn't work](#what-doesnt-work)


> **Outdated documentation page:**<br>
> This documentation page is outdated and needs rewriting. For example polymorphism already works. If you have a question about a specific value you want to serialize, just ask it in the discord server.

<!--
    TODO: describe, how one can use the serializer if they need to send
    data somewhere (e.g. via http client)
-->


<a name="available-types"></a>
## Available types

Here is a list of things that can be saved by Unisave:

**Primitives**

- `null`
- `long`, `int`, `short`, `byte`
- `bool`
- `double`, `float`
- `string`

**Geometry**

- `Vector2`, `Vector3`
- `Vector2Int`, `Vector3Int`

**Arrays**

- `byte[]`, `Vector3[]`, ...

**Collections**

- `List<T>`, `Dictionary<TK, TV>`

**Enums**

```cs
enum MyEnum
{
    One,
    Two = 2,
    Three = 3
}
```

**Your own classes**

```cs
class Motorbike
{
    public string name;
    public int fuel;
    public string driverName;
    public float power;
}
```

Unisave will serialize all non-static fields (both public and private).

~~The type must have a parameterless constructor in order to be deserialized.~~

~~The `[Serializable]` attribute is not needed.~~ Edit: The attribute needs to be serialized properly, planned feature.


<a name="planned-types"></a>
## Planned types

**Geometry**

- `Transform`, `Matrix4x4`, `Quaternion`, `Vector4`


<a name="how-it-works"></a>
## How it works

The data is stored in the form of JSON. The format works well with hierarchical data and is human-reader friendly. But it lacks type information which it both a problem and an advantage.

> **Credit:** Unisave uses [LightJson](https://github.com/MarcosLopezC/LightJson), which is a wonderful lightweight library for working with JSON in C#

When saving a field, Unisave looks at the type of the field and it will use this type for serialization ~~regardless of type of the actual value~~.


<a name="what-doesnt-work"></a>
## What doesn't work

**Saving meshes, game objects, prefabs, textures**

There are assets claiming they can save these kinds of things. While it might be possible, it's not really a good idea. When you save a tree, it might go through, but when you save a forest, your disk and network usage goes through the roof and you are saving the same data multiple times.

Instead save a list of tree positions and then make tree instances after loading the list.

*But I want to save a prefab online to be able to change it later.*<br>
Unisave does not support this kind of usage. And think about it twice - do you really ever need to change just a single little thing, without changing anything else, any kind of script? It's a better idea to just release a next version of your game.

**High volume data (virtual worlds)**

Unisave does not target this usage, because such data is much more complex to work with and so cannot be easily generalized across mutiple games. At least some kind of chunking is needed, but exact solution differs from game to game.

**Cyclic dependencies**

The benefit is low and the complexity high. It's not worth implementing, because the entire structure of the serialized data would have to be different and not as easily readable (by a human).

~~**Polymorphism**~~

~~For polymorphism to work I would need to save the type together with the data itself (because I need to know the type during loading). This might be done for standard types, but it would be more difficult for user-defined types.~~

~~Also saving type information couples the database with your game very tightly making it difficult to change the data structure later.~~
