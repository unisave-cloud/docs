# Matchmaker

-  [What is a matchmaker](#what-is-a-matchmaker)
-  [Terminology](#terminology)
-  [Creating a matchmaker](#creating-a-matchmaker)
    - [Dealing with time](#dealing-with-time)
    - [Matching players against AI](#matching-players-against-ai)
    - [Cleaning up old matches](#cleaning-up-old-matches)
-  [Using the matchmaker](#using-the-matchmaker)
    - [Canceling matchmaking](#canceling-matchmaking)
    - [Querying match of a player](#querying-match-of-a-player)
-  [Advanced topics](#advanced-topics)
    - [Naming matchmakers](#naming-matchmakers)
    - [How matchmaker remembers tickets](#how-matchmaker-remembers-tickets)


<a name="what-is-a-matchmaker"></a>
## What is a matchmaker

Matchmaker is a black box, that accepts *players* and groups them into *matches*. For now we don't care how it works, all we need to know is that from the player's perspective the interaction is relatively simple:

1. **Join the matchmaker.** Player provides all additional information, like who he/she wants to play with, etc.
2. **Repeatedly ask the matchmaker, whether the player has been matched.** This process continues, unless the player cancels the waiting or the player is matched.

From the matchmaker's perspective the process is also simple. You define a method called `CreateMatches` that gets a list of waiting players with their additional data and it splits the list up into groups and creates a *match* for each group. You don't need to think about calling this method, it is handled for you.


<a name="terminology"></a>
## Terminology

**MatchEntity** is an ordinary entity, that you define. It represents a single *match*, a single race, battle, fight, ... Owners of this entity are the players participating in the match. The relationship of those players have to be captured by additional entity attributes you specify (who starts, who plays white, ...).

**MatchmakerTicket** is a class, that represents a *ticket*. It represents a single *player*, waiting in the matchmaker. It contains the time, when the player started waiting. It may than contain any additional data you specify about the waiting player (what tank they chose for the battle, what experience they have, what friends they want to play with, etc.).

**MatchmakerFacet** is a facet, that implements the matchmaking logic. Most of the logic here is already implemented for you, all need to do is specify the `CreateMatches` method that performs the actual matching.

**MatchmakerClient** is a *MonoBehaviour* component, that you place into your scene and it is responsible for communicating with the matchmaker. Basically it can start waiting and cancel waiting and then it has two events it can handle: when the player is matched and when the waiting is canceled.


<a name="creating-a-matchmaker"></a>
## Creating a matchmaker

You need to create two files, the matchmaker backend logic and the matchmaker client component. First go to the `Backend` folder, right click and choose `Create > Unisave > Matchmaking > Matchmaker`. Confirm the default file name.

Open the file that has been created and get familiar with it. This default matchmaker matches players with simmilar rating into pairs. The rating is provided in the ticket.

It contains definition of a `MatchEntity` class, with one attribute `FirstMovePlayer`. By default this player is the one with smaller rating of the two.

```cs
public class MatchEntity : Entity
{
    [X] public UnisavePlayer FirstMovePlayer { get; set; }
}
```

Then there's a definition of a `MatchmakerTicket`. You can see the ticket contains `playerRating` field.

```cs
public class MatchmakerTicket : BasicMatchmakerTicket
{
    public int playerRating;
}
```

Finally there's the `MatchmakerFacet` containing the matchmaking logic:

```cs
public class MatchmakerFacet : BasicMatchmakerFacet
    <MatchmakerTicket, MatchEntity>
{
    protected override void PrepareNewTicket(MatchmakerTicket ticket)
    {
        ticket.playerRating = 42; // dummy rating calculation
    }
    
    protected override void CreateMatches(List<MatchmakerTicket> tickets)
    {
        // sort waiting players by their score
        tickets.Sort((a, b) => a.playerRating.CompareTo(b.playerRating));
        
        // while we have enough tickets
        while (tickets.Count >= 2)
        {
            // take first two tickets from the queue
            var selectedTickets = tickets.GetRange(index: 0, count: 2);
            tickets.RemoveRange(index: 0, count: 2);

            // create a match with those tickets
            var match = new MatchEntity {
                FirstMovePlayer = selectedTickets[0].Player // lower score
            };
            
            // launch the match
            SaveAndStartMatch(selectedTickets, match);
        }
    }
}
```

First you see the `PrepareNewTicket` method. It gets called whenever a new ticket is inserted into the matchmaker (when new player starts waiting). Inside this method you can pull additional data from the database and put it into the ticket. You can for example calculate the player rating. You can also perform some validation. If the ticket contains strange values, you can throw an exception to abort the ticket insertion.

> **Note:** If you don't need the `PrepareNewTicket` method, you can remove it entirely.

Then there's the heart of the matchmaker. `CreateMatches` method gets called whenever the matchmaker state changes (new ticket is inserted or some time passes). You have to modify this code to fit your needs. The general shape of this code is:

1. **Look at the list of waiting `tickets`.**
2. **Select tickets from the list, that will form a new match.**
3. **Create a new `MatchEntity` instance and fill it out**, but don't save it, don't assign owners yet.
4. **Call the `SaveAndStartMatch` method.** This method performs the saving and owner assignment for you.
5. **Repeat until there are no compatible tickets.**


<a name="dealing-with-time"></a>
### Dealing with time

When you make decitions on what players to match, it might be useful to know, how long are the players waiting in the matchmaker. For this, you can use two properties of a ticket:

```cs
protected override void CreateMatches(List<MatchmakerTicket> tickets)
{
    MatchmakerTicket ticket = tickets[0];

    // for how many seconds is the player waiting?
    double wfs = ticket.WaitingForSeconds;

    // when did the player start waiting?
    // or rather DateTime.UtcNow value at that time
    DateTime iat = ticket.InsertedAt;
}
```

Now it becomes important to know, when the `CreateMatches` method gets called. Basically it happens each time some waiting client polls the matchmaker for status. For that single client it is every 6 seconds, starting immediately after the player joins the matchmaker.

> **Note:** There's no delay between joining the matchmaker and the first poll. So a player might be matched immediately, when lucky.


<a name="matching-players-against-ai"></a>
### Matching players against AI

When there's not many players online, there might not be enough of them, to fill up an entire match. So when a player waits for too long (say above 30 seconds), they should be matched against an AI.

Detecting a long waiting player is described in the section above, using the `ticket.WaitingForSeconds` property.

All you need to do next is to store the fact in the `MatchEntity`:

```cs
public class MatchEntity : Entity
{
    [X] public bool OpponentIsAI { get; set; }

    [X] public UnisavePlayer FirstMovePlayer { get; set; }
}
```

You set the `OpponentIsAI` to `true`. If the AI is the first one to move, you set `FirstPlayerToMove` to `null`. You might include some AI difficulty here if you want and so on...

Then in your game client, instead of joining a Photon room, you just start a singleplayer game, pretending it's multiplayer.

But again, this depends entirely on your game and is specific to it. You will need to figure out the specifics by yourself.


<a name="cleaning-up-old-matches"></a>
### Cleaning up old matches

It's important that you remove `MatchEntities` that are no longer needed. Luckily the default matchmaker implementation already removes match entities that have been created more than 24 hours ago.

If you want to have matches that live longer than that, or you want to do the garbage collection by yourself, you can override the `CleanUpMatches` method:

```cs
public class MatchmakerFacet : BasicMatchmakerFacet
    <MatchmakerTicket, MatchEntity>
{
    protected override void PrepareNewTicket(MatchmakerTicket ticket)
    {...}
    
    protected override void CreateMatches(List<MatchmakerTicket> tickets)
    {...}

    protected override void CleanUpMatches()
    {
        var matches = GetEntity<TMatchEntity>
            .OfAnyPlayers()
            .GetEnumerable();

        foreach (var match in matches)
        {
            if (match.IsDead) // your custom check here
                match.Delete();
        }
    }
}
```

This method gets called each time some player polls the matchmaker.


<a name="using-the-matchmaker"></a>
## Using the matchmaker

Now that you have a matchmaker created, you need to talk with it from your game client. This talking is performed by a *matchmaker client*, so let's create one.

Go into your `Scripts` folder (or anywhere you put your `.cs` file, that is not the *backend* folder). Right click and choose `Create > Unisave > Matchmaking > MatchmakerClient`. Accept the default file name.

You will see a file with this code:

```cs
using Unisave;
using Unisave.Modules.Matchmaking;
using UnityEngine;

/// <summary>
/// Communicates with the matchmaker
/// </summary>
public class MatchmakerClient : MonoBehaviourBasicMatchmakerClient
    <MatchmakerFacet, MatchmakerTicket, MatchEntity>
{
    void Start()
    {
        StartWaitingForMatch(new MatchmakerTicket {
            // TODO: you can send some additional data to the matchmaker here
            // e.g. chosen tank, color, card deck, friends to play with, ...
        });
    }

    /// <summary>
    /// Called when a match is assigned to this us
    /// </summary>
    /// <param name="match">The match entity that was assigned</param>
    protected override void JoinedMatch(MatchEntity match)
    {
        // TODO: now you can load another scene or something...
        
        Debug.Log("Match has been joined: " + match.ToJson());
        Debug.Log("Are we the first to move: " +
                  (match.FirstMovePlayer == Auth.Player));
    }
}
```

Put this script into your matchmaking scene and modify it to your needs.

> **Note:** Matchmaking scene is sometimes also called a lobby.

You join the matchmaker by calling `StartWaitingForMatch` and providing a new ticket instance. Here is where a ticket is created. In the default implementation this method is called immediately in the `Start` method (because I assume you have a dedicated scene for matchmaking).

Then you define a method `JoinedMatch` that is called when you are assigned to a match. Here you do stuff you need to do: load another scene containing the level, join a Photon room, establish a TCP connection, ...

You can use `match.EntityId`, as an identifier for a room, connection, whatever...


<a name="canceling-matchmaking"></a>
### Canceling matchmaking

When the player no longer wants to wait and clicks some kind of "back" button, you need to call `StopWaitingForMatch()`. This will take some time to process and once it's done, a callback will be run:

```cs
public class MatchmakerClient : MonoBehaviourBasicMatchmakerClient
    <MatchmakerFacet, MatchmakerTicket, MatchEntity>
{
    void Start()
    {...}

    protected override void JoinedMatch(MatchEntity match)
    {...}

    public void OnBackButtonClick()
    {
        StopWaitingForMatch();
    }

    protected override void WaitingCanceled()
    {
        // load some menu scene
    }
}
```

Make sure you do it this way and you don't just leave the scene. The player would remain in the matchmaker and possibly get matched, without knowing it. This would be same as if he/she joined the match and then leaved it immediately.

Also note that just because a player has been matched, doesn't mean they will join the Photon room. They might loose internet connection, so you need to have some timeout and start the match even if some players are missing.


<a name="querying-match-of-a-player"></a>
### Querying match of a player

When the match ends, data about the match is collected by each participating player and then sent to the server for reviewing. You might compare what each player says and go with the majority to prevent cheating. But this is rather advanced and an overkill for a small beginning game. It's entirely ok to just trust what each individual player says.

Then you need to give reward to each player. You have a few possibilities, how to get the `MatchEntity` of a player.

You can just send it to the server directly, but **don't forget to refresh it**:

```cs
public void GiveReward(MatchEntity matchEntity)
{
    // important, the entity might have changed while the match took place
    // - you may even want to lock it in a transaction
    // - or you state that a match entity,
    //  once created is immutable and ignore this
    matchEntity.Refresh();

    // ... perform some rewarding here ...
}
```

You can send just the ID and load it from the database:

```cs
public void GiveReward(string matchEntityId)
{
    var matchEntity = GetEntity<MatchEntity>.Find(matchEntityId);

    // ... perform some rewarding here ...
}
```

Or you can query by the player, **but don't forget to get the latest:**

```cs
using System.Linq;

public void GiveReward()
{
    // get all matches for this player
    var matches = GetEntity<MatchEntity>
        .OfPlayer(Caller)
        .AndOthers()
        .Get();

    // get the most recent match for this player
    var matchEntity = matches
        .OrderByDescending(m => m.CreatedAt)
        .First();

    // ... perform some rewarding here ...
}
```


<a name="advanced-topics"></a>
## Advanced topics


<a name="naming-matchmakers"></a>
### Naming matchmakers

You might need to have multiple matchmakers. Unisave got you covered even in such case. You simply create another matchmaker, just make sure you rename all the classes appropriately to not confuse them. Also make sure you name `MatchmakerFacet` class differently. If you want to keep the class name, you can override the `GetMatchmakerName` method on the matchmaker facet. This method by default returns name of the matchmaker facet type. This name is used for storing the matchmaker state, see below.


<a name="how-matchmaker-remembers-tickets"></a>
### How matchmaker remembers tickets

All the tickets and other additional information is stored in a `BasicMatchmakerEntity` inside the database. This entity has name of the matchmaker it belongs to.

If at any time you rename your matchmaker facet class, the matchmaker name will change. This is not a problem, a new `BasicMatchmakerEntity` will be created, but the previous one will not be deleted.

So if you just stumble upon an entity you don't recognize, maybe it's this matchmaker entity.

You can delete this entity at any time, it will be automatically created when needed again.
