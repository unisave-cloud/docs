# Authentication

- [Player entity](#player-entity)
- [Auth facet](#auth-facet)
- [Authenticated player](#authenticated-player)
- [Guarding facets](#guarding-facets)
- [Security](#security)
    - [Password hashing](#password-hashing)
    - [Sensitive data leakage](#sensitive-data-leakage)
- [Custom authentication](#custom-authentication)


<a name="player-entity"></a>
## Player entity

You have to create an entity that will represent your players whenever you want to authenticate your players. Unisave provides a nice template to get you started quickly. Go to your `Backend` folder, right-click, select `Create > Unisave > Auth > PlayerEntity`, and hit enter. The following file will be created:

```cs
using System;
using Unisave;
using Unisave.Entities;

public class PlayerEntity : Entity
{
    /// <summary>
    /// Email of the player
    /// </summary>
    public string email;

    /// <summary>
    /// Hashed password
    /// </summary>
    public string password;
}
```

You may add any additional fields you want, like `name`, `coins`, `experience`, ...


<a name="auth-facet"></a>
## Auth facet

Auth facet contains logic regarding player registration, login, and logout. Again, Unisave has a reasonable template for you. Go to your `Backend` folder, right-click, select `Create > Unisave > Auth > AuthFacet`, and hit enter. The following file will be created:

```cs
using System;
using Unisave.Facades;
using Unisave.Facets;
using Unisave.Utils;

public class AuthFacet : Facet
{
    public bool Login(string email, string password)
    {
        var player = FindPlayer(email);

        if (player == null)
            return false;

        if (!Hash.Check(password, player.password))
            return false;

        Auth.Login(player);
        return true;
    }

    public bool Logout()
    {
        bool wasLoggedIn = Auth.Check();
        Auth.Logout();
        return wasLoggedIn;
    }
    
    public RegistrationResult Register(string email, string password)
    {
        // ... validation code ...
        // return RegistrationResult.InvalidEmail;
        // return RegistrationResult.WeakPassword;
        // return RegistrationResult.EmailTaken;
        
        // register
        var player = new PlayerEntity {
            email = normalizedEmail,
            password = Hash.Make(password)
        };
        player.Save();
        
        // login
        Auth.Login(player);
        
        return RegistrationResult.Ok;
    }

    public enum RegistrationResult { /* ... */ }
    
    // ... helper methods ...
}
```

The facet performs player registration by an email and a password. You can easily add player nicknames to the registration logic. The entire facet is built on top of the `Auth` facade about which you can read in the [custom authentication](#custom-authentication) section.

Feel free to read the entire template and modify the code to fit your needs. The facet normalizes email addresses (meaning they are turned to lower-case and trimmed). This makes the login logic seem case-insensitive even though it isn't.

> **Note:** The facet uses a case-sensitive email comparison to make sure a database index can be used for the lookup. This speeds up the login significantly.

> **Note:** The lookup is performed twice - once with the email the player typed in and the second time with its lowercase variant. This ensures that an email address with upper-case characters in the database can still be logged into (for backward compatibility and general fool-proofing).


<a name="authenticated-player"></a>
## Authenticated player

You can ask for the currently authenticated player using the `Auth` facade:

```cs
using Unisave.Facades;
using Unisave.Facets;

public class WhoIsFacet : Facet
{
    public void WhoIsLoggedIn()
    {
        var player = Auth.GetPlayer<PlayerEntity>();

        if (player == null)
            Log.Info("Nobody is logged in.");
        else
            Log.Info(player.email + " is logged in.");
    }
}
```

You can also only check whether there is anyone logged in:

```cs
bool someoneIsLoggedIn = Auth.Check();
```


<a name="guarding-facets"></a>
## Guarding facets

There are many facets that should only be accessed by an authenticated player. You can enforce this condition by specifying an authentication middleware for the facet:

```cs
using Unisave.Facades;
using Unisave.Facets;
using Unisave.Authentication.Middleware;

[Middleware(typeof(Authenticate))] // <-- this line
public class PlayerFacet : Facet
{
    public void ChangePlayerName(string newName)
    {
        var player = Auth.GetPlayer<PlayerEntity>();

        // player won't be null here because of the middleware check

        player.name = newName;
        player.Save();
    }
}
```

The middleware declaration `[Middleware(typeof(Authenticate))]` can also be applied to a single method only.

A `Unisave.Authentication.AuthException` will be raised whenever a non-authenticated game client tries to call a guarded facet method.


<a name="security"></a>
## Security


<a name="password-hashing"></a>
### Password hashing

When tinkering with the `AuthFacet` make sure you hash passwords before storing them. Hashing is an important technique that makes sure you don't store your player's passwords as-is. A hashed password is almost impossible to turn back to the original password so if you were to accidentally leak your database, your player's passwords wouldn't be compromised.

Unisave provides utility functions in `Unisave.Utils.Hash` class that can help you with hashing:

```cs
// create hash (during registration)
string hashedPassword = Hash.Make("password");

// compare value against a hash (during login)
string providedPassword = "password";
bool matches = Hash.Check(providedPassword, hashedPassword); // true
```

> **Tip:** You can read more about hashing at https://crackstation.net/hashing-security.htm


<a name="sensitive-data-leakage"></a>
### Sensitive data leakage

The `PlayerEntity` might contain sensitive data like an email address, real name, password hash, access tokens, etc. Leaking this information might give malicious actor access to the player's account and steal their identity. Because of this, you should be very wary about returning player entities from facets.

Say you implement a matchmaker. Many players participate in a single match. Each player needs to know the nicknames of all the other players. If you were to download entire player entities you would gain access not only to the nicknames but also to all the sensitive information. Make sure you extract only the information you actually need and nothing more.

Never trust your game client. It could have been disassembled and modified. It can call facets in a different order than you've anticipated. Trust only the server-side code (facets) - it cannot be tampered with.

Even if a player would download their own player entity. They could have a virus-infected computer. They could have a cracked version of your game. Sensitive information should never leave the server.


<a name="custom-authentication"></a>
## Custom authentication

You can build any custom authentication system on top of the `Auth` facade. The `Auth` facade simply remembers the authenticated player during the playing session.

There are a few methods used to query the currently authenticated player:

```cs
// Tests whether somebody is logged in.
bool someoneIsLoggedIn = Auth.Check();

// Gets ID of the entity representing the authenticated player.
// Null if nobody logged in.
string playerEntityId = Auth.Id();

// Gets the entity representing the authenticated player.
// Returns null if nobody logged in.
PlayerEntity player = Auth.GetPlayer<PlayerEntity>();
```

Then there are two methods that make a player logged in or logged out:

```cs
// Takes in the entity representing a player.
Auth.Login(player);

// Logs out the authenticated player.
// Does nothing if nobody authenticated.
Auth.Logout();
```
