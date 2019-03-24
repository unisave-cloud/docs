Unisave Cloud
=============

Unisave Cloud builds on top of Unisave Local and allows you to save data in the cloud.

> **Note:** Unisave Cloud is currently under development, so the unity asset does not contain any code for it and the documentation below is rather sparse.

Before you can access data, you have to login a player:

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
UnisaveCloud.Logout(); // and then load the login scene
```
