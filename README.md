Unisave
=======

Saving player progress in the cloud made super easy:

```cs
using Unisave;

public class PlayerStatistics : UnisaveCloudBehaviour
{
    [SavedAs("statistics.races.total")]
    public int racesTotal = 0;

    [SavedAs("statistics.races.won")]
    public int racesWon = 0;

```

It's like PlayFab, but fun to use.

Do you prefer having your feet on the ground instead of floating in the cloud? Unisave can of course save your data locally as well.


## Introduction

Unisave is a free Unity asset and an online service that together solve the problem of player data persistence.

The goal of Unisave is to be very easy to learn and understand. You should be able to use it even if you are still just learning to code.

Unisave consists of two parts (but they are both used in the same way):

- Unisave Local
- Unisave Cloud


### Unisave Local

Unisave Local saves the data locally, on the player's computer. This requires you only to import the [free asset](#) into your game. No registration or cloud stuff here.

Then you just make your favourite `MonoBehaviour` scripts inherit from `UnisaveLocalBehaviour` and tag favourite fields with the `[SavedAs("some-key")]` attribute.

More information can be found in the documentation below.


### Unisave Cloud

Unisave Cloud is an online service that works together with the asset and provides the following:

- Player registration and email verification
- Player login into your game
- Saving player-related data in the cloud (like Unisave Local, but online)
- Moving game-related data to the cloud for later tweaking

Even if your game is not multiplayer, you can still synchronize player achievements between devices or monitor player progress through the game.

> **Note:** Unisave Cloud service is currently under development and is not yet available. Unisave Local has been released early on as a source of feedback from you, developers. Thank you.


## What can be saved

- Basic types, like `int`, `float`, `string`, ...
- Unity math types `Vector2`, `Vector3`, `Vector3Int`, ...
- Unity transform `Transform` (very soon)
- Collections `List<T>`, `Dictionary<string, T>`
- Arrays `byte[]`, `Vector3[]` (very soon)
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


## Documentation

- [Unisave Local](#Unisave-Local)
- [Unisave Cloud](#Unisave-Cloud)
- [Examples](#)
    - Local
        - [Leaderboard](#)

