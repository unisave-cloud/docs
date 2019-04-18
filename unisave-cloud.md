# Unisave Cloud

- [Video](#video)
- [Introduction](#introduction)
- [Registration & Login](#registration-and-login)
- [Loading and saving](#loading-and-saving)
- [Logout](#logout)
- [Local testing player](#local-testing-player)
- [Unexpected logout](#unexpected-logout)
- [Custom login and registration](#custom-login-and-registration)


<a name="video"></a>
## Video

<div style="position: relative">
    <div style="padding-top: 56.25%"></div>
    <iframe
    style="position: absolute; top: 0; left: 0; height: 100%"
    width="100%"
    height="auto"
    src="https://www.youtube.com/embed/HQIbqRsSIYQ"
    frameborder="0"
    allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen
    ></iframe>
</div>


<a name="introduction"></a>
## Introduction

Unisave Cloud builds on top of Unisave Local and allows you to save data in the cloud.

But before you can use it, you need to have a developer account on the Unisave website. The service is free for up to 100 registered players, so you can freely try it out.

After you registered yourself, you need to create a game and set up a connection from your Unity project. The process is explained in this short [cloud connection manual](cloud-connection).

<a name="registration-and-login"></a>
## Registration & Login

Before you can save any data, you need to know to which player the data belongs. So you need to provide the player with a registration and a login form.

This boring user interface is best to keep in a separate scene. Now let's create this scne and name it `LoginScene`, for example.

To save you time, Unisave asset comes with a login form, saved as a prefab in `Unisave/Components/Auth/Login or Register Panel.prefab`.

But it's a piece of user interface, so you first need to create a canvas. Right-click the `Hierarchy` window and create `UI > Canvas`. Now drag the prefab from the file explorer into the canvas game object.

<img src="img/unisave-cloud_hierarchy.png">

You should be able to see the form in the scene view. Next you need to tell the panel, which scene should be loaded after a player logs in.

Select the `Login or Register Panel` and scroll down the `Inspector` window. You should see a `Login Or Register Controller`. This controller has a field named `Scene Name To Load`. Here you put the path to the next scene:

<img src="img/unisave-cloud_next-scene.png">


<a name="loading-and-saving"></a>
## Loading and saving

Now inside the `GarageScene` you can save data in a familiar manner:

```cs
public class PlayerStatistics : UnisaveCloudBehaviour
{
    [SavedAs("statistics-races-total")]
    public int racesTotal = 0;

    [SavedAs("statistics-races-won")]
    public int racesWon = 0;
}
```

> Note that this is practically identical to *Unisave Local*. The only difference being the parent class `UnisaveCloudBehaviour`.

Loading and saving again happen during `Awake` and `OnDestroy`, however the saving does not go directly to the server. The data is kept inside a cache. This cache is initialized during player login and then sent to the server on logout. So make sure you call logout, when the player wants to exit from your game.

However to prevent data loss during unexpected circumstances, Unisave also performs saving periodically every 5 minutes, if a player is logged in.

> **Note:** This fake loading and saving to the cache, is actually called *Distribution* and *Collection* in the code.


### Explicit saving

Simmilar to the *Unisave Local* case, you can override the time of saving and loading.

Calling `UnisaveCloud.Load(behaviour)` distributes the data from cache into your behaviour script.

Calling `UnisaveCloud.Save()` however saves **all data from all scripts at the same time and sends it to the server**.

*Unisave Cloud* remembers, where the data has been distributed, so the saving method does not require a behaviour script as an argument.

So saving all changes after a notable event can be achieved quite easily:

```cs
class GarageController : MonoBehaviour
{
    public void NewEngineHasBeenPurchased()
    {
        // save everyting in all existing behaviour scripts
        // and send it to the server
        UnisaveCloud.Save();
    }
}
```


<a name="logout"></a>
## Logout

And when you're done, you logout the player and go back to the login scene.

```cs
UnisaveCloud.Logout();

SceneManager.LoadSceneAsync("Scenes/LoginScene");
```

<a name="local-testing-player"></a>
## Local testing player

When you are developing your game, you don't want to login every single time you launch your game. When you start a scene, that expects a user to be logged in, then a fake player is logged in.

Data of this player is saved locally, so it works just like *Unisave Local*, however all the cloud-related stuff like periodic saving still takes place.

If you start your game on the login scene, but you still want to login as the local testing player, you just login with email `local` and any password.

Local testing player can only be accessed, when launching your game inside the editor. After the game is built, the `local` email will be concidered as any other email and launching a login-only scene without having a player logged in will result in an error.


<a name="unexpected-logout"></a>
## Unexpected logout

> **Warning:** Currently not implemented, planned soon.

Sometimes network connection breaks and the player data cannot be saved to the server. When this happens, an event is triggered that your game needs to handle and it needs to forcefuly logout the player.

There will be a small data loss, because the progress made since the last save cannot be saved anymore. But that's precisely why you need to force a logout - to make sure the data loss won't be any greater.

To handle this scenario, you just need to subscribe to the event:

```cs
class MyScript : MonoBehaviour
{
    /*
        PLANNED FEATURE!
        The exact syntax may be different when implemented.
     */

    void Start()
    {
        // register a method to be called
        // in case of an unexpected logout
        UnisaveCloud.UnexpectedLogout += this.OnUnexpectedLogout;
    }

    void OnUnexpectedLogout()
    {
        // load the login scene with a "connection lost" message
        // for example something like this:
        MyLoginSceneController.LaunchMode = LaunchMode.ConnectionLost;
        SceneManager.LoadSceneAsync("Scenes/LoginScene");

        // you actually don't need to call logout,
        // because it's called automatically
        //UnisaveCloud.Logout();
    }
}
```


<a name="custom-login-and-registration"></a>
## Custom login and registration

The `Login or Register Panel` prefab that comes with the asset can be modified to fit the graphical style of your game. But if you want to customize the login or registration process, you can do that easily.

> **Example:** A form, that automatically registers a player if the email is unknown.

> **Example:** Separating registration logic to a different scene, or disabling it all together.

> **Example:** Enforcing password strength or removing the password confirmation field.


### Registration

```cs
using Unisave;

public class RegistrationController : MonoBehaviour, IRegistrationCallback
{
    public void OnTheButtonClick()
    {
        // this = IRegistrationCallback instance
        UnisaveCloud.Register(this, "email@example.com", "password");
    }

    // required by IRegistrationCallback
    public void RegistrationSucceeded()
    {
        // login now?
    }

    // required by IRegistrationCallback
    public void RegistrationFailed(RegistrationFailure failure)
    {
        Debug.Log(failure.message);
    }
}
```


### Login

```cs
using Unisave;

public class LoginController : MonoBehaviour, ILoginCallback
{
    public void OnTheButtonClick()
    {
        // this = ILoginCallback instance
        UnisaveCloud.Login(this, "email@example.com", "password");
    }

    // required by ILoginCallback
    public void LoginSucceeded()
    {
        // login now?
    }

    // required by ILoginCallback
    public void LoginFailed(LoginFailure failure)
    {
        Debug.Log(failure.message);
    }
}
```
