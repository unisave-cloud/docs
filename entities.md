# Entities

- [Introduction](#introduction)
- [Declaration](#declaration)
- [Working with an entity](#working-with-an-entity)
    - [Create](#create)
    - [Update](#update)
    - [Delete](#delete)
    - [Refresh](#refresh)
    - [Entity IDs](#entity-ids)
    - [Find by ID](#find-by-id)
    - [Timestamps](#timestamps)
- [Entity queries](#entity-queries)
    - [Retrieving all entities of a given type](#retrieving-all-entities-of-given-type)
    - [Retrieving the first entity](#retrieving-the-first-entity)
    - [Filter clause](#filter-clause)
- [Singleton entities](#singleton-entities)
- [Entity references](#entity-references)

<!--
- [Transactions](#transactions)
    - [Manual transactions](#manual-transactions)
- [Locks](#locks)
    - [Lock for update](#lock-for-update)
    - [Shared lock](#shared-lock)
    - [Deadlocks](#deadlocks)
-->


<a name="introduction"></a>
## Introduction

An *entity* is a small collection of data that can be stored in the database. It's analogous to a row in a relational database or a document in a NoSQL database, however, it has some additional benefits. Each entity has a type that determines its attributes and each attribute holds the actual data. Entities are designed to interface neatly with the C# source code of your game.


<a name="declaration"></a>
## Declaration

Inside your `Backend/Entities` folder right-click and choose `Create > Unsiave > Entity`. Type in the entity name `PlayerEntity`. A file with the following content will be created:

```cs
using System;
using System.Collections;
using System.Collections.Generic;
using Unisave;
using Unisave.Entities;
using Unisave.Facades;

public class PlayerEntity : Entity
{
    /// <summary>
    /// Replace this with your own entity attributes
    /// </summary>
    public string myAttribute = "Default value";
}
```

The created class describes the structure of a new entity. Each public field or property represents a value that will be stored in the database. The entity is also a regular C# class so you can add methods and private fields to it.

> **Note:** A public property has to have both a public getter and a public setter, otherwise it won't be saved.

> **Note:** You can use either properties or fields, this is up to you.

We can add attributes to the newly created entity to make it represent a player:

```cs
using System;
using System.Collections;
using System.Collections.Generic;
using Unisave;
using Unisave.Entities;
using Unisave.Facades;

public class PlayerEntity : Entity
{
    /// <summary>
    /// Player name displayed to other players
    /// </summary>
    public string name;

    /// <summary>
    /// Number of coins owned
    /// </summary>
    public int coins;

    /// <summary>
    /// When will the premium account expire (or has expired)
    /// </summary>
    public DateTime premiumUntil = DateTime.UtcNow;
}
```


<a name="working-with-an-entity"></a>
## Working with an entity


<a name="create"></a>
### Create

To create a new entity in the database, create new instance of the entity class and save it:

```cs
var player = new PlayerEntity {
    name = "John",
    coins = 200
};

player.Save();
```

> **Warning:** The `Save()` method might only be called from server code (e.g. inside facets).


<a name="update"></a>
### Update

To modify any entity, you modify the corresponding attributes and call save:

```cs
player.premiumUntil = DateTime.UtcNow.AddDays(30);

player.Save();
```


<a name="delete"></a>
### Delete

An entity can be deleted from the database by calling `Delete()` on it:

```cs
someEntity.Delete();
```

The `Delete()` method returns `true` if the entity has been actually deleted and `false` if it wasn't in the database, to begin with (someone might have deleted it before us).

> **Warning:** Make sure you don't use the entity instance after you call `Delete()`. Unisave does not guarantee corectness of the instance state. Simply throw it away.


<a name="refresh"></a>
### Refresh

To pull the latest entity data from the database, use the `Refresh()` method:

```cs
player.Name = "Steve";

player.Refresh();

player.Name; // "John"
```


<a name="entity-ids"></a>
### Entity IDs

Each entity in the database has a unique string identifier. You can obtain this identifier as follows:

```cs
string id = player.EntityId;
```

An entity that hasn't been saved yet has the `EntityId` set to `null` (because it isn't in the database).


<a name="find-by-id"></a>
### Find by ID

We can find an entity in the database by its ID:

```cs
var player = DB.Find<PlayerEntity>(id);
```

If no such entity is found, `null` will be returned.


<a name="timestamps"></a>
### Timestamps

Each entity automatically keeps track of two timestamps: `CreatedAt` and `UpdatedAt`.

```cs
DateTime createdAt = someEntity.CreatedAt;
DateTime updatedAt = someEntity.UpdatedAt;
```

`CreatedAt` is set when the `Save` method is called for the first time.

`UpdatedAt` is set each time you call the `Save` method.

Both values are set to immediate `DateTime.UtcNow`.

> **Note:** When the entity is not yet stored in the database, timestamps have no meaningful value.

> **Note:** It is a good idea to use unisavesal coordinated time (UTC) throughout your game and convert it to local time only when displaying it to the player. This is because your game will most likely be distributed around the world over multiple timezones and this approach makes is easy to deal with.


<a name="entity-queries"></a>
## Entity queries

Entities can be queried from the database by the server-side code using the `GetEntity<T>` facade. Understand that this can not be done on the client-side because the database is not accessible there.


<a name="retrieving-all-entities-of-given-type"></a>
### Retrieving all entities of a given type

The following query returns all the leaderboards:

```cs
List<LeaderboardEntity> leaderboards = DB.TakeAll<LeaderboardEntity>().Get();
```


<a name="retrieving-the-first-entity"></a>
### Retrieving the first entity

When we know, there is only a single result to be returned, we can ask for the only result:

```cs
MatchmakerEntity matchmaker = DB.TakeAll<MatchmakerEntity>().First();
```

If no such entity exists, `null` will be returned.

> **Warning:** Make sure you don't use the `First` method when you suspect that multiple entities might match the query because the entity that will be chosen is picked at random.


<a name="filter-clause"></a>
### Filter clause

We can filter out player with active premium account using the `Filter` clause:

```cs
var premiumPlayers = DB.TakeAll<PlayerEntity>()
    .Filter(p => p.premiumUntil > DateTime.UtcNow)
    .Get();
```

When registering a new player, you might want to check the email address availability:

```cs
using System;
using Unisave.Facets;
using Unisave.Facades;

public class AuthFacet : Facet
{
    /// <summary>
    /// Registers a new player
    /// </summary>
    public bool Register(string email, string password)
    {
        var player = DB.TakeAll<PlayerEntity>()
            .Filter(p => p.email == email)
            .First();
        
        if (player != null)
            return false; // email address already registered
        
        // ... continue with registration ...
    }
}
```

You can filter multiple times by repeating the `Filter` clause:

```cs
var wealthyPremiumPlayers = DB.TakeAll<PlayerEntity>()
    .Filter(p => p.coins > 20_000)
    .Filter(p => p.premiumUntil > DateTime.UtcNow)
    .Get();
```

But the filtration above could also be written using a single clause:

```cs
var wealthyPremiumPlayers = DB.TakeAll<PlayerEntity>()
    .Filter(p => p.coins > 20_000 && p.premiumUntil > DateTime.UtcNow)
    .Get();
```


<a name="singleton-entities"></a>
## Singleton entities

There are some entities that exist only in one instance (one leaderboard, one matchmaker). There is a convenient method `FirstOrCreate` that simplifies working with such entities:

```cs
using System;
using Unisave;
using Unisave.Facets;
using Unisave.Facades;

public class LeaderboardFacet : Facet
{
    /// <summary>
    /// Sends the leaderboard to the game client
    /// </summary>
    public LeaderboardEntity GetLeaderboard()
    {
        return DB.FirstOrCreate<LeaderboardEntity>();
    }

    /// <summary>
    /// Adds a new record to the leaderboard
    /// </summary>
    public void AddRecord(string playerName, int score)
    {
        var leaderboard = DB.FirstOrCreate<LeaderboardEntity>();

        leaderboard.Add(playerName, score); // some smart logic inside

        leaderboard.Save();
    }
}
```

The entity will be created the first time it is needed and then it will only be loaded over and over again.

You might also want to have an entity containing player achievements. One player always has only one achievements entity:

```cs
var achievements = DB.TakeAll<AchievementsEntity>()
    .Filter(a => a.owner == player)
    .FirstOrCreate(a => {
        // here we can initialize the new entity
        a.owner = player;
    });
```


<a name="entity-references"></a>
## Entity references

Entities can contain references to other entities, which lets you build more complex systems. Say a player owns multiple motorbikes - we can capture this fact with references:

```cs
using System;
using System.Collections;
using System.Collections.Generic;
using Unisave;
using Unisave.Entities;
using Unisave.Facades;

public class MotorbikeEntity : Entity
{
    /// <summary>
    /// Motorbike has an owner
    /// </summary>
    public EntityReference<PlayerEntity> owner;

    // ... other attributes ...
}
```

When no owner is assigned, the reference contains `null`. We can create a new motorbike and give it an owner:

```cs
var player = Auth.GetPlayer<PlayerEntity>();

var motorbike = new MotorbikeEntity() {
    owner = player
};
motorbike.Save();
```

Now we can ask for all the motorbikes a player has:

```cs
var motorbikes = DB.TakeAll<MotorbikeEntity>()
    .Filter(m => m.owner == player)
    .Get();
```

Or we can get the owner of a given motorbike:

```cs
PlayerEntity player = motorbike.owner.Find();
```

Again, the owner can be `null` if the reference points nowhere.


<!--

<a name="transactions"></a>
## Transactions

Transaction is a set of database operations that is performed entirely, or not at all. This is useful in that when an exception occurs inside a transaction, the whole transaction can be rolled back.

```cs
var someEntity = SomeEntity.Get();

someEntity.Foo; // 42

DB.Transaction(() => {
    someEntity.Foo = 0;
    
    someEntity.Save();

    throw new Exception();
});

// Foo will still have the value of 42
// even though the whole execution
// crashed after Save was called.
```

You can also pass a return value through a transaction:

```cs
public class SomeFacet : Facet
{
    public int SomeMethod()
    {
        return DB.Transaction(() => {
            
            // ...

            return 42;
        });
    }
}
```

Transactions can even be nested inside each other, although locks persist until the topmost transaction finishes.

Transaction depth can be queried:

```cs
DB.TransactionDepth(); // int
```


<a name="manual-transactions"></a>
### Manual transactions

You can start a new transaction:

```cs
DB.StartTransaction();
```

You can rollback a transaction:

```cs
DB.Rollback();
```

And you can commit a transaction:

```cs
DB.Commit();
```


<a name="locks"></a>
## Locks

When your game grows in popularity, many people will play it simultaneously. This might cause problems when two players try to modify an entity at the same time. This is called a race condition.

Imagine we have a `StatisticsEntity` and each time a player logs in, we increment some counter:

```cs
using System;
using Unisave;

public class StatisticsEntity : Entity
{
    [X] public int LoginCount { get; set; }

    // just a singleton pattern
    // see the section above on singleton entities
    public static StatisticsEntity Get()
    {
        var entity = GetEntity<StatisticsEntity>.First();

        if (entity == null)
        {
            entity = new StatisticsEntity();
            entity.Save();
        }

        return entity;
    }
}

public class SomeFacet : Facet
{
    public void IncrementCounter()
    {
        var entity = StatisticsEntity.Get();

        entity.LoginCount++;

        entity.Save();
    }
}
```

What can happen is that both players load the entity simultaneously with counter value `N`. Both increment the value to `N+1` and then both save `N+1` into the database instead of `N+2`.

> **Note:** This problem concerns primarily *game entities* and *shared entities*. *Player entities* are usually modified by their owner, so there should not be a problem.

> **Note:** Also if you just read an entity, such problem does not bother you. This problem occurs only when modifying data.

What we need is to allow only one player to edit the entity at a time. For that purpouse locks exist.


<a name="lock-for-update"></a>
### Lock for update

Let's modify the example in the following way:

```cs
public class SomeFacet : Facet
{
    public void IncrementCounter()
    {
        var entity = StatisticsEntity.Get();

        DB.Transaction(() => {
            entity.RefreshAndLockForUpdate();
            
            entity.LoginCount++;
            
            entity.Save();
        });
    }
}
```

The `RefreshAndLockForUpdate` method refreshes the entity and locks it so that noone else can access it. The operation is performed after the lock with certainty, that we are the only one acting on the entity. Then the entity is saved and the transaction is committed, which releases the lock.

> **Warning:** Locks can only be applied inside transactions, because the transaction determines their scope. Commit of a transaction releases all locks.

`RefreshAndLockForUpdate` prevents other players from locking or modifying the entity. Non-locking players can however still read the value.

> **Note:** This kind of lock is sometimes also called an exclusive lock.

Usual usecase for this lock is when you modify value of an entity based on it's current value (e.g. incrementing a counter).


<a name="shared-lock"></a>
### Shared lock

A weaker kind of lock is called a shared lock. You can acquire it as follows:

```cs
DB.Transaction(() => {
    someEntity.RefreshAndLockShared();

    // ...
});
```

This lock prevents other players from modifying the entity, but it allows them to lock it with a shared lock as well.


<a name="deadlocks"></a>
### Deadlocks

Deadlock occurs when you have multiple players waiting on locks in a cycle. Such scenario is detected and a `Unisave.Exceptions.DatabaseDeadlockException` is thrown and the transaction is rolled back.

Number of retry attempts can be specified as the second argument to the `Transaction` method.

```cs
DB.Transaction(() => {
    // ...
}, 5);
```

-->
