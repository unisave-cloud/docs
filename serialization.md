# Data Serialization

### What can be saved

- Basic types, like `int`, `float`, `string`, ...
- Unity math types `Vector2`, `Vector3`, `Vector3Int`, ...
- Unity transform `Transform` (soon)
- Collections `List<T>`, `Dictionary<string, T>`
- Arrays `byte[]`, `Vector3[]` (soon)
- Your own classes:

```cs
class Motorbike
{
    public string name;
    public int fuel;
    public string driverName;
    public float power;
}
```

<!--
    What doesn!t work
    - meshes, game objects, prefabs, textures
    - high volume data
    - cyclic dependencies
    - polymorphism
-->
