# Symfony's cheat sheet

A highly opinionated [Symfony](http://symfony.com/) cheat sheet.

## Security

### Authentication

Are you who you say you are?

* **Firewalls**:
    * listen on `kernel.event`
    * have many listeners
* **Authentication Listeners**:
    * map client data from request to token
    * pass token to authentication manager
    * update state of security context
* **Authentication Managers**:
    * responsible of authenticating the token
    * call the appropriate authentication provider
    * handle exceptions
    * have many authentication providers
* **Authentication Providers**:
    * perform authentication using client data in the token
    * mark the token as authenticated
    * attach the user to the token
    * *may* rely on user provider
* **User Providers**:
    * retrieve the user (from a database, a configuration file, etc)

### References

* [Kris Wallsmith](https://twitter.com/kriswallsmith)' slides: [Love and Loss: a Symfony Security play](http://fr.slideshare.net/kriswallsmith/love-and-loss-a-symfony-security-play)
