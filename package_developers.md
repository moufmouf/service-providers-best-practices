---
jumbotron:
    title: For package developers
    subtitle: Master universal module development!
    image: elephant2.jpeg
---

So you have developed a package, and you plan on making it available to many frameworks out there. Welcome!

Universal modules are Composer packages
---------------------------------------

This might be obvious, but let's say it anyway. Universal modules MUST be Composer packages.

The `composer.json` file of your package MUST require `container-interop/service-provider`:

```json
{
  "require": {
    "container-interop/service-provider": "^1.0"
  }
}
```


Separate package or bundled package?
------------------------------------

The first question you will have to answer is: *do I want the universal module to be part of my main package or is it an additional package?*

We are ok with both solutions. If you provide the *universal module* as an extra package next to your main package, please name it with a `-universal-module` suffix.
For instance, if your module is named `foo/bar`, the *universal module* should be `foo/bar-universal-module`.


Implementing the interface
--------------------------

A universal module contains at least one service provider.

This service provider MUST implement the [`Interop\Container\ServiceProvider`](https://github.com/container-interop/service-provider/blob/master/src/ServiceProvider.php) interface.

Constructor parameters
----------------------

The service provider MUST NOT have a constructor with compulsory parameters. It is ok if the service provider has a constructor with some optional parameters.

<div class="alert alert-info">
The service provider should be usable <em>out of the box</em>, without having to pass arguments to the container.
It must not need any kind of configuration to be useful (except configuration provided directly in the container). It is important to remember that containers are expected to
automatically detect your service provider (using Puli) and create an instance of your service provider (passing no parameter).
</div>

Constructor parameters are ok if they are altering the behaviour of a service provider (for instance by modifying the default name of the provided services).


Fetching configuration
----------------------

Configuration MUST be fetched from the container. Each framework stores configuration in a different place.

- Symfony stores configuration in `app/config/config.yml`.
- Laravel stores configuration in `config/app.php`.
- Other frameworks will store configuration somewhere else...

As far as you are concerned, this is not an issue. As long as you are using a compatible framework, **configuration can be accessed from the container**.

So if your service provider needs a `LOGFILE` configuration parameter, you can simply use `$container->get('LOGFILE')`.

Of course, you will need to let the developer know you are expecting such a parameter to exist. More on this in the documentation chapter below.


<div class="alert alert-danger">What you should NOT do:</div>

```php
class DbConnectionProvider implements ServiceProvider {

    private $dbHost;
    private $dbUser;
    private $dbPassword;
    
    /**
     * Such a constructor is NOT ok. You are forcing the developer to pass parameters in the constructor
     * arguments, hence defeating the purpose of the dependency-injection container.
     */
    public function __construct(string $dbHost, string $dbUser, string $dbPassword)
    {
        $this->dbHost = $dbHost;
        $this->dbUser = $dbUser;
        $this->dbPassword = $dbPassword;
    }

    public function getServices()
    {
        return [
            DbConnection::class => [ self::class, 'createConnection' ]
        ];
    }
    
    public static function createConnection()
    {
        return new DbConnection($this->dbHost, $this->dbUser, $this->dbPassword);
    }
}
```

<div class="alert alert-success">What you should do instead:</div>

```php
class DbConnectionProvider implements ServiceProvider {

    public function getServices()
    {
        return [
            DbConnection::class => [ self::class, 'createConnection' ]
        ];
    }
    
    public static function createConnection(ContainerInterface $container)
    {
        // This is ok: configuration is fetched from the container.
        return new DbConnection(
            $container->get('my.package.dbhost'),
            $container->get('my.package.dbuser'), 
            $container->get('my.package.dbpassword')
        );
    }
}
```



TODO

explain why (Puli must be able to create service provider automatically + a service provider must be usable out of the box without requiring any parameter in the constructor)





For package developers
=======================


- explain how to document a package
- naming convention
- no compulsory parameters in the constructor
- fetching dependencies
- explain Puli integration, how to bind a package
- performance best practices
