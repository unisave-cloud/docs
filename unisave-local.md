Unisave Local
=============

- [Introduction](#introduction)
- [SavedAs attribute](#saved-as)
- [Loading and saving](#loading-and-saving)
- [Default value](#default-value)
- [NonNull attribute](#non-null)


<a name="introduction"></a>
## Introduction

Unisave Local solves the problem of player data saving in an elegant manner. Let's analyze the example provided on the title page:

```cs
using Unisave;

public class PlayerStatistics : UnisaveLocalBehaviour
{
    [SavedAs("statistics.races.total")]
    public int racesTotal = 0;

    [SavedAs("statistics.races.won")]
    public int racesWon = 0;

```

So we have some kind of racing game and we want to store some statistics. We created the `PlayerStatistics` script, that will be inserted into every scene that needs to display, or modify the statistics. Now it's up to the script to handle the saving.

Normally we would use `PlayerPrefs` to store the values and it would require quite a bit of coding. Now we can just state a fact: "Save this field under this name." Such an approach is substantially more self-explanatory. We don't care how it's done, just that it works.


<a name="saved-as"></a>
## `SavedAs` attribute

This attribute tells Unisave which fields to save and how to call them. It can be attached to any public, non-static field of your script.

The only argument is the key, under which the value will be stored.

> **Note:** The key should be unique within the game, meaning there should not be two scripts (or two simultaneous instances of one script) with the same key on some fields. If that was to happen, the system would not work properly.

The attribute does not perform the loading and saving procedure. That is enabled by inheriting from `UnisaveLocalBehaviour`.

> **Note:** There is the `UnisaveLocalBehaviour` for local saving and the `UnisaveCloudBehaviour` for cloud saving. For more info about the cloud saving see the related documentation page.


<a name="loading-and-saving"></a>
## Loading and saving

The data is loaded during `Awake()` so it's already prepared for you in the `Start()` method. Saving takes place in the `OnDestroy()` method. This logic is implemented in the `UnisaveLocalBehaviour` class.

If you want to perform loading and saving at a different moment, you can define it yourself:

```cs
using Unisave;

public class MyScript : MonoBehaviour
{
    void OnEnable()
    {
        UnisaveLocal.Load(this);
    }

    void OnDisable()
    {
        UnisaveLocal.Save(this);
    }
}
```

Note that we no longer inherit from the `UnisaveLocalBehaviour`. The `UnisaveLocalBehaviour` class is only meant as a shortcut for convenience, but it's by no means mandatory.

Now we have an idea of what's actually going on below the surface. We call the methods `UnisaveLocal.Load` and `UnisaveLocal.Save` and they look at our script, find all fields marked with `[SavedAs(...)]` and perform the necessary action.


<a name="default-value"></a>
## Default value

So each script is immediately loaded as soon as it's instantiated. But what if there is no value saved to load? Well then we simply provide the default value as we are used to:

```cs
[SavedAs("my-field")]
public string myField = "My default value";
```

How cool is that? It looks just like casual C# code, but it's actually much deeper.

> **Note:** Don't forget that the default value will be used the first time your user launches your game.

Also if you omit the default value, then C# automatically assumes the default value to be `null` or `0`. So having no default value is not an option.


<a name="non-null"></a>
## `NonNull` attribute

Sometimes you have fields, that look like this:

```cs
[SavedAs("names")]
public List<string> names = new List<string>();
```

Then you use the field somewhere in your code:

```cs
names.Sort();
```

But what if the field happens to be `null`? Well, then you get a `NullReferenceException` and your game blows up.

It might happen, that the saved value is `null`. Maybe due to some mistake from previous run of your game. If you mark the field with the `[NonNull]` attribute, then Unisave will be cautious when loading and if it loads a `null` value, then the value will be ignored and the default value will be used instead.

```cs
[SavedAs("names")]
[NonNull]
public List<string> names = new List<string>();
```

Also you will get a warning message if you try to save `null` into such field.

This attribute does not solve mistakes, but helps with their detection and tries to reduce their destructive effects. It's better to empty a list than to blow up the game.
