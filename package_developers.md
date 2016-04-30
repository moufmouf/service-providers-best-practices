---
jumbotron:
    title: For package developers
    subtitle: Master universal module development!
    image: elephant2.jpeg
---
So you have developed a package, and you want to make it available to many frameworks out there. Welcome!

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

Puli integration
----------------

Universal modules MUST provide a Puli binding for the service providers they are providing.

This way, users of your *universal module* will just have to require the module in Composer and automatically, your service provider will be detected and your service provider entries will be available in the container.

Not used to Puli? Here is a crash course: 

In `composer.json`, add the `puli/cli` and `puli/composer-plugin` in `require-dev`:

```sh
composer require --dev puli/cli
composer require --dev puli/composer-plugin
```

Then, simply use Puli's `bind` command:

```sh
vendor/bin/puli bind --class Acme\\Foo\\MyServiceProvider container-interop/service-provider
```

That's it! Do not forget to commit the new `puli.json` file in your package repository!

Naming convention (objects)
---------------------------

If your service provider is creating a **service** (a class that is most of the time meant to be instantiated only once), the name of your service should be the fully qualified name of the class.

For instance, a database connection or a logger that logs to a file are services. Most of the time, you only need one database connection, or one log file. Of course, you can create later more instances of the same class using your container, but the service provider will help you get started with a sensible default of one instance.


<div class="alert alert-danger">This is NOT ok:</div>

```php
class MyServiceProvider implements ServiceProvider {

    public function getServices()
    {
        return [
            // The 'myModuleService' is not matching the fully qualified class name of 'MyService'.
            'myModuleService' => function() {
                return new MyService();
            }
        ];
    }
}
```

<div class="alert alert-success">This is ok:</div>

```php
class MyServiceProvider implements ServiceProvider {

    public function getServices()
    {
        return [
            MyService::class => function() {
                return new MyService();
            }
        ];
    }
}
```

<div class="alert alert-info">Note: it is important to respect this convention, as a courtesy to users of <em>autowiring</em> containers. Those containers will automatically try to inject in containers instances whose name is the name of the type-hinted class/interface. If you do not follow this naming convention, you are preventing the autowiring capability of these containers to work out of the box.</div>

If your service is providing a default implementation for a well-known interface, you can also create an alias on the fully qualified name of the interface.

<div class="alert alert-danger">This is NOT ok:</div>

class MyLoggerProvider implements ServiceProvider {

    public function getServices()
    {
        return [
            // Let's assume FileLogger implements PSR-3's LoggerInterface
            // The FileLogger should NOT be connected to the LoggerInterface directly.
            // The LoggerInterface can be overloaded by another service provider and your FileLogger entry will be lost forever.
            LoggerInterface::class => function(ContainerInterface $container) {
                return new FileLogger($this->container->get('LOGFILE'));
            }
        ];
    }
}


<div class="alert alert-success">This is ok:</div>

```php
class MyLoggerProvider implements ServiceProvider {

    public function getServices()
    {
        return [
            // Let's assume FileLogger implements PSR-3's LoggerInterface
            FileLogger::class => function(ContainerInterface $container) {
                return new FileLogger($this->container->get('LOGFILE'));
            },
            // Then our logger can claim the default instance by creating an alias on the interface name
            LoggerInterface::class => new Alias(FileLogger::class)
        ];
    }
}
```

Note: other service providers may also claim the same interface. The last service provided registered will "win".
But since the FileLogger is the only one to claim its own class name, the user can later decide to use this specific FileLogger using the `FileLogger::class` entry.

Finally, your service provider may create many instances of the same class. This is particularly true if your service provider is providing objects that are not services.

For instance, a service provider could decide many file loggers (one log file for errors, one for debug). Or another service provider might decide to offer menu items to be put in a menu...

<div class="alert alert-info">If many provided objects share the same class, the name of the object should be <em>namespaced</em> using the Composer package name, in order to avoid naming collisions.</div>

So if your package is creating 2 file loggers and none is generic enough to claim the `FileLogger::class` entry, then a good name for those logger would be:

- `vendor_name/package_name:errorLogger`
- `vendor_name/package_name:debugLogger`



Naming convention (configuration/parameters)
--------------------------------------------

<div class="alert alert-info">If your service provider requires parameters, it should fetch those from the container.
It is a good idea to expect namespaced parameters AND to create an alias from the namespaced parameter to the non namespaced parameter.</div>

Ok, this is probably a little confusing, so let's take an example.

If your service provider expects a `LOGFILE` parameter, and another service provider expects a `LOGFILE` parameter too, there is no way you can feed 2 different `LOGFILE` parameters to your 2 service providers.

Instead, your service provider should rely on a parameter named "[vendor_name].[package_name].LOGFILE".

Of course, having to provide a parameter named "my_company.my_package.LOGFILE" if a bit tiresome. So your service provider could add an alias to the more simple "LOGFILE" this way:

```php
class MyLoggerProvider implements ServiceProvider {

    public function getServices()
    {
        return [
            FileLogger::class => function(ContainerInterface $container) {
                return new FileLogger($this->container->get('my_company.mypackage.LOGFILE'));
            },
            'my_company.mypackage.LOGFILE' => new Alias('LOGFILE')
        ];
    }
}
```

So now, the user can simply provide a `LOGFILE` parameter in its configuration. But if for some reason, the parameter `LOGFILE` is also used by another service provider and the user wants to feed 2 different values to the 2 service providers, the user can instead provide a `my_company.mypackage.LOGFILE`. This parameter will override the alias defined in the service provider. Shazam! We have a simple solution, yet flexible in case things get more complicated.


Performance best practices
--------------------------

TODO






For package developers
=======================


- explain how to document a package
- naming convention
- no compulsory parameters in the constructor
- fetching dependencies
- explain Puli integration, how to bind a package
- performance best practices
