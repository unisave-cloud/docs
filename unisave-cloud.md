# Unisave Cloud

- [Video](#video)
- [Introduction](#introduction)
- [Registration & Login](#registration-and-login)
- [Saving data](#saving-data)
- [Logout](#logout)


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

After you registered, you need to create a game and set up a connection from your Unity project. The process is explained in this short [cloud connection manual](cloud-connection).

<a name="registration-and-login"></a>
## Registration & Login

Before you can save any data, you need to know to which player the data belongs. So we need to provide the player with a registration and a login form.

This boring user interface is best to keep in a separate scene. Now create this `LoginScene`.

To save you time, Unisave asset comes with a login form saved as a prefab in `Unisave/Components/Auth/Login or Register Panel.prefab`.

But it's a user interface, so we first need to create a canvas. Right-click the `Hierarchy` window, and create `UI > Canvas`. Now drag the prefab from the file explorer into the canvas game object.

<img src="img/unisave-cloud_hierarchy.png">

You should be able to see the form in the scene view. Next we need to tell the panel, which scene should be loaded after a player logs in.

Select the `Login or Register Panel` and scroll down the `Inspector` window. You should see a `Login Or Register Controller`. This controller has a field called `Scene Name To Load`. Here you put the path to the next scene:

<img src="img/unisave-cloud_next-scene.png">


<a name="saving-data"></a>
## Saving data

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


<a name="logout"></a>
## Logout

And when you're done, you logout the player and go back to the login scene.

```cs
UnisaveCloud.Logout();

SceneManager.LoadSceneAsync("Scenes/LoginScene");
```

## TODOcument

> *Unisave Cloud* documentation is being worked on, so it doesn't cover everything yet.

- local testing player
    - starting a logged-in-only scene (e.g. `GarageScene`) in Unity editor logs in a fake local player
    - this player can be also accessed by logging in with email `'local'` and any password
- continuous saving
    - yes, it's saves the data in the background periodically every 5 min
- unexpected logout handling
    - e.g. network disconnect
- custom login and registration logic
    - error messages, password strength validation
    - autoregistration
