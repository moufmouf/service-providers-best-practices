---
jumbotron:
    title: Deploy your services in any PHP framework
    subtitle: Finally, frameworks are playing together!
    image: elephant.jpeg
---


**Universal modules** are PHP packages built to be framework agnostic. They integrate and run easily with any package.
 
Every modern PHP framework comes with a *container*. The container provides the framework with configured *instances* of services.
If you are a package developer and you need to integrate with a particular framework, you usually need to write a *meta-package* that explains to the framework how to configure its container. If you want to support 5 frameworks, you have to write 5 modules. On the opposite side, as a user, if the package author did not write a meta-package for your particular framework, it gets harder to use his package.

That's where **universal modules** come into play. Universal modules have the ability to put *services* into any framework's container.

How is this different from container-interop/service-provider?
--------------------------------------------------------------

**Universal modules** are built on top of [container-interop's service providers](http://github.com/container-interop/service-provider).

The `container-interop/service-provider` project contains the base `ServiceProvider` interface allowing packages to provide services to all compatible frameworks. The interface is meant to be a standard that can be adopted easily. By design, the `container-interop/service-provider` project does not enforce anything regarding naming, performances, etc...

This `universal modules` norm offers an additional set of rules that are more opinionated and extend the standard.

For instance, universal modules have:

- A strict naming policy for services
- Puli integration: they are automatically detected by your framework

This website also provides a set of advices for universal module writers allowing them to:

- write modules that are truly interoperable
- write service providers that have the best possible performance
