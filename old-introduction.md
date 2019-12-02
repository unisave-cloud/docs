# Introduction

Unisave is a free Unity asset and an online service that together solve the problem of player data persistence.

- [Installation](#installation)
- [Unsiave Local](#unisave-local)
- [Unisave Cloud](#unisave-cloud)


<a name="installation"></a>
## Installation

- Open your game in Unity
- Open the asset store window by going to menu `Window > Asset Store`
- Search for `Unisave` and open <a href="https://assetstore.unity.com/packages/slug/142705" target="_blank">this asset</a>.
- Click `Download`, then `Import` and import all the files

Now you are ready to use both *Unisave Local* and *Unisave Cloud*.


<a name="unisave-local"></a>
## Unisave Local

*Unisave Local* is a part of the asset, that saves player data locally, on the player's computer. You only need to import the asset for *Unisave Local* to work.

> Think `PlayerPrefs` on steroids.

Once you import the asset, you can save your data by simply stating a fact:<br>"Save this awesome field under this name."

```cs
using Unisave;

public class PlayerAchievements : UnisaveLocalBehaviour
{
    [SavedAs("achievements")]
    public List<string> takenAchievements = new List<string>();

    [SavedAs("distance-walked")]
    public float walkedDistance = 0.0f;

```

> **Note:** Don't forget on `using Unsiave;` and the `UnisaveLocalBehaviour` parent

*Unisave Local* is completely free, you can use it however you like.

For more information, see the [Unisave Local Documentation](unisave-local).


<a name="unisave-cloud"></a>
## Unisave Cloud

Unisave Cloud is an online service that works together with the asset and provides the following:

- In-game registration and login for players
- Saving player-related data in the cloud (like *Unisave Local*, but online)
<!-- - Moving game-related data to the cloud for later tweaking -->

Even if your game is not multiplayer, you can still synchronize player achievements between devices or monitor player behaviour.

For more information, see the [Unisave Cloud Documentation](unisave-cloud).
