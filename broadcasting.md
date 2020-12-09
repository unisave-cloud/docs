# Broadcasting

TOC


## Introduction

Broadcasting is a system, that lets you send messages from the server to clients. It gives you a way to notify clients when something interesting happens on the server. It can also be used as a backbone for turn-based games.

Your game client subscribes to a number of channels. Channel is a pipe through which messages flow from the server to subscribed clients. Your server-side code can then send messages into channels. It's just like YouTube.


## Defining a channel

At the core of the broadcasting system are channels. Each channel has a unique string identifier, but you will represent it by a class inside your `Backend` folder:

```cs
using Unisave.Broadcasting;

public class ChatRoomChannel : BroadcastingChannel
{
    public SpecificChannel WithParameters(string roomName)
    {
        return SpecificChannel.From<ChatRoomChannel>(roomName);
    }
}
```

The class above represent many channels, but all of them repesent some chatting room. The method `WithParameters` creates one such specific channel.

You can add your own methods that create specific channels, with arguments that fit your need. You can, for example, create a notification channel for each player, where the player will receive notifications about friend requests:

```cs
using Unisave.Broadcasting;

public class FriendRequestsChannel : BroadcastingChannel
{
    public SpecificChannel OfPlayer(PlayerEntity player)
    {
        return SpecificChannel.From<FriendRequestsChannel>(
            player.EntityId
        );
    }
}
```

Or you can create a global channel where the server periodically broadcasts the number of active players:

```cs
using Unisave.Broadcasting;

public class ActivePlayersChannel : BroadcastingChannel
{
    public SpecificChannel WithoutParameters()
    {
        return SpecificChannel.From<ActivePlayersChannel>();
    }
}
```

All you have to do is provide a list of string parameters to the specific channel cosntructor.

> **Note:** The string identifier of a channel, with the parameters has to be at most 255 characters, so be careful with the parameters. The string identifier has the following structure:<br>
> `Namespace.ChannelClass["parameter1", "another param"]`<br>
> (this identifier is 53 characters)


## Defining messages

Now that you have channels, you need to define the messages that will be sent through them. Messages are defined similarly to entities:

```cs
using Unisave.Broadcasting;

// some player sent a chat message
public class ChatMessage : BroadcastingMessage
{
    public string playerName;
    public string message;
}

// some player joined the chat room
public class PlayerJoinedMessage : BroadcastingMessage
{
    public string playerName;
}
```


## Sending messages

You can send messages anywhere from your server-side code using the `Broadcast` facade. Here is an example, how an authenticated player would send a chat message to a chat room:

```cs
using Unisave;
using Unisave.Facades;
using Unisave.Facets;

public class ChatFacet : Facet
{
    public void SendMessage(string room, string msg)
    {
        // get the authenticated player
        var player = Auth.GetPlayer<PlayerEntity>();

        // send the message into the channel
        Broadcast.Channel<ChatRoomChannel>()
            .WithParameters(room)
            .Send(new ChatMessage {
                playerName = player.name,
                message = msg
            });
    }

    ...
}
```

Note that the `WithParameters` is one of the methods you've defined in your channel.


## Subscribing to a channel

The last thing that remains is actually receiving the messages on the client. But before the client can receive any messages it has to first subscribe to a channel. Channel subscriptions are initiated by the client but performed by the server:

1) The client asks the server to subscribe to a channel, by calling a facet method.
2) The server performs the subscription, which is represented by a `ChannelSubscription` object.
3) The server sends the subscription representation back to the client.
4) The client consumes the subscription and thus begins to receive messages from the subscribed channel.

Let's first show the steps 1 and 4, that are performed by the client:

```cs
using System.Collections;
using Unisave.Broadcasting;
using Unisave.Facades;
using UnityEngine;

public class ChatController : UnisaveBroadcastingClient
{
    private async void OnEnable()
    {
        // ask server for a subscription to a chat room
        var subscription = await OnFacet<ChatFacet>
            .CallAsync<ChannelSubscription>(
                nameof(ChatFacet.JoinRoom),
                "some-room-name"
            );
        
        // route messages by type to corresponding methods
        FromSubscription(subscription)
            .Forward<ChatMessage>(ChatMessageReceived)
            .Forward<PlayerJoinedMessage>(PlayerJoined)
            .ElseLogWarning();
    }

    void ChatMessageReceived(ChatMessage msg)
    {
        // [John]: Hello people!
        Debug.Log($"[{msg.playerName}]: {msg.message}");
    }

    void PlayerJoined(PlayerJoinedMessage msg)
    {
        // John joined the room
        Debug.Log($"{msg.playerName} joined the room");
    }
}
```

You create a script that inherits `UnisaveBroadcastingClient`, which in turn inherits `MonoBehaviour`. Then you specify the `OnEnable` method to register any subscriptions that are needed. The `OnDisable` is already defined and handles unsubscribing, so you don't have to care about that. So from the outside the component just magically receives messages and you can activate and deactivate it any way you want.

The server-side logic that handles the subscription is more interesting:

```cs
using Unisave;
using Unisave.Facades;
using Unisave.Facets;

public class ChatFacet : Facet
{
    ...

    public ChannelSubscription JoinRoom(string room)
    {
        // get the authenticated player
        var player = Auth.GetPlayer<PlayerEntity>();

        // verify the player can access the channel
        if (player.isBanned)
            throw new AuthException();

        // subscribe the client into the channel
        var subscription = Broadcast
            .Channel<ChatRoomChannel>()
            .WithParameters(room)
            .CreateSubscription();
        
        // new player in the room broadcast
        Broadcast.Channel<ChatRoomChannel>()
            .WithParameters(room)
            .Send(new PlayerJoinedMessage {
                playerName = player.name
            });

        return subscription;
    }
}
```

There's a lot to unpack. Firstly, the facet can contain any logic you want so you can easily check that the player has permissions to actually subscribe to the channel. It also means that the server can in-theory choose the channel the client will subscribe to, it doesn't need to be the client's choice.

The part where the subscription happens is the method call `.CreateSubscription()`. It can be accessed in the same fashion to the `.Send(...)` method seen before. Again, the `.WithParameters(...)` method is one of the methods you defined for the channel `ChatRoomChannel`.

Now that the subscription has been created, the client will receive any messages sent to the channel from now on. Even though it hasn't yet consumed the subscription. The messages are remembered and will be received the moment the subscription gets consumed. This means we can send a message to everyone in the channel (including our just subscribed client), that a new player has joined.

Lastly we return the subscription from the facet so that the client can consume it.


## Losing connection and other edge-cases

TL;DR; The connection is automatically re-established and no messages should get lost (they will be re-played).

TODO
