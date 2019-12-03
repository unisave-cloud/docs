# Introduction

- [Server logic](#server-logic)
- [Calling server methods](#calling-server-methods)
- [Data](#data)
- [Where to go next](#where-to-go-next)

Unisave is a service providing backend for games, targeting Unity engine and C# language.


<a name="server-logic"></a>
## Server logic

No matter what backend you are building, you need to have server-side logic. This logic is grouped into so-called *facets*.

Facet is the face of your backend server. It is what your game client can call remotely.

Below is a `HomeFacet` that returns data about the logged-in player when a home scene is loaded:

```cs
using System;
using System.Collections;
using System.Collections.Generic;
using Unisave;

public class HomeFacet : Facet
{
    /// <summary>
    /// Returns information about the logged-in player
    /// </summary>
    public PlayerEntity GetPlayerEntity()
    {
        // pull data from the database
        PlayerEntity entity = GetEntity<PlayerEntity>
            .OfPlayer(this.Caller)
            .First();

        // send data back to the caller
        return entity;
    }
}
```


<a name="calling-server-methods"></a>
## Calling server methods

Of course, there needs to be some `MonoBehaviour` in the home scene that actually calls the facet method:

```cs
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unisave;

public class HomeSceneController : MonoBehaviour
{
    void Start()
    {
        OnFacet<HomeFacet>.Call(nameof(HomeFacet.GetPlayerEntity))
            .Then(PlayerInfoObtained)
            .Done();
    }

    void PlayerInfoObtained(PlayerEntity playerEntity)
    {
        Debug.Log("Player: " + playerEntity.Nickname);
        Debug.Log("Coins: " + playerEntity.Coins);
    }
}
```


<a name="data"></a>
## Data

What is this `PlayerEntity`? An *entity* is a collection of data that can be stored in the database.

This `PlayerEntity` you've seen is defined as follows:

```cs
using System;
using System.Collections;
using System.Collections.Generic;
using Unisave;

public class PlayerEntity : Entity
{
    /// <summary>
    /// Name displayed to other players
    /// </summary>
    [X] public string Nickname { get; set; }

    /// <summary>
    /// Number of coins owned
    /// </summary>
    [X] public int Coins { get; set; }
}
```

Entities can be created, saved, modified and deleted by the server code. They can also be sent to the client and read.


<a name="where-to-go-next"></a>
## Where to go next

Now you need to know, where to put this code, how to get it to the server and how to run it. This is described in the [next section on workflow](workflow).
