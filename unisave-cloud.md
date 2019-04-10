Unisave Cloud
=============

Unisave Cloud builds on top of Unisave Local and allows you to save data in the cloud.

> **Note:** Unisave Cloud is currently under development, so players must be registered manually just for testing purpouses.

First you need to create account in the online service and then set some properties in the unity asset configuration tab. See the [cloud connection manual](cloud-connection.md) for details.

Now inside your game, you first need your player to log in:

```cs
public class MyLoginController : MonoBehaviour, ILoginCallback
{
    public InputField email;
    public InputField password;

    public void OnLoginButtonClick()
    {
        UnisaveCloud.Login(this, email.text, password.text);
    }

    public void LoginSucceeded()
    {
        // load the next scene
    }

    public void LoginFailed(LoginFailure failure)
    {
        // login did not succeed, display failure.message
    }
}
```

Then in the logged-in scene you can access the data:

> Note that this is practically identical to Unisave Local. The only difference being the parent class `UnisaveCloudBehaviour`.

```cs
public class PlayerStatistics : UnisaveCloudBehaviour
{
    [SavedAs("statistics.races.total")]
    public int racesTotal = 0;

    [SavedAs("statistics.races.won")]
    public int racesWon = 0;
}
```

And when you're done, you logout the player:

```cs
UnisaveCloud.Logout();
```
