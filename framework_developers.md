---
jumbotron:
    title: For framework and container developers
    subtitle: Making your project compatible with universal modules!
    image: bridge2.jpg
---

So you are a framework developer or a container developer and are looking to add support for univeral modules in your project? Welcome!

Adding support for service providers
------------------------------------

Essentially, adding support for universal modules means adding support for [container-interop's service providers](http://github.com/container-interop/service-provider).

There are essentially 2 strategies here:

- Add support for service providers in your container yourself,  
- Or rely on a third party container and plug it into your container using the [delegate lookup feature](https://github.com/container-interop/container-interop/blob/master/docs/Delegate-lookup.md).
  [Yaco](https://github.com/thecodingmachine/yaco) can be a good starting point to manage the entries provided by the service providers. It is a compiled container and yet is minimalistic.


The container contains the configuration
----------------------------------------
 
Configuration MUST be available from the container.

Each framework stores configuration in a different place.

- Symfony stores configuration in `app/config/config.yml`.
- Laravel stores configuration in `config/app.php`.
- Your framework may store it in another place...

The only way to make configuration available in a framework agnostic way is to make it available in the container.

This means that `$container->get('DB_HOST')` should return the `DB_HOST` parameter stored in the configuration if it exists.

Many frameworks out there treat configuration and services as 2 different beasts and do not make configuration available in the container.
If you are in this case, you can still use universal module by providing the service providers with an adapter around your container.
The adapter will check both the container and the configuration.

Check the [Symfony adapter](https://github.com/thecodingmachine/service-provider-bridge-bundle/blob/1.0/src/SymfonyContainerAdapter.php) or the [Laravel container adapter](https://github.com/thecodingmachine/laravel-universal-service-provider/blob/1.0/src/LaravelContainerAdapter.php) if you need some examples.


Automatic discovery of service providers
----------------------------------------

Frameworks MUST support service providers discovery using the Puli binding **by default**.

This way, users of your framework will just have to require the universal module in Composer and automatically, your framework will detect and embed the service provider in the container.

You should also offer an option to completely disable Puli.

Of course, users can also manually disable Puli bindings to remove one particular service provider.

Performance optimizations
-------------------------

TODO: explain integration with container-interop/common-factories
