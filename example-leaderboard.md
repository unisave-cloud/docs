Leaderboard example
===================

The Leaderboard example that comes with the unity asset shows, how you would go about implementing a simple leaderboard with Unisave.

The example contains a game, where the player starts a timer and then they have to click a button as many times as possible. The number of clicks will be the player's score and the leaderboard shows highest scores together with the player names.

The game itself is not wery important, but what is important is the `Leaderboard.cs` script. It contains a method that registers a new record into the leaderboard:

```cs
public void GameHasFinished(string playerName, int score) {...}
```

This methods does the data insertion, sorting and clipping of the 5 best recrods.

One record in the board is represented by this class:

```cs
public class Record
{
    public int score;
    public string playerName;
}
```

> **Tip:** The class is `[Serializable]` just to show up in the unity inspector. Unisave does not require this attribute to be present.

The board as a whole is then just an ordered list of records:

```cs
public List<Record> records = new List<Record>();
```

Now comes the Unisave part. We mark the `records` field and inherit from `UnisaveLocalBehavior`:

```cs
public class Leaderboard : UnisaveLocalBehavior
{
    [SavedAs("leaderboard")]
    [NonNull]
    public List<Record> records = new List<Record>();

    ...
}
```

We also mark the field as `[NonNull]`, because we want to keep the value to be an instance of `List<Record>` even if it somehow happend, that the saved value was a literal `null`.
