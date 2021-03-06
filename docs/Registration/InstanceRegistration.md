<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table Of Contents

- [Instance Registration](#instance-registration)
  - [Usage in modules](#usage-in-modules)
  - [Configuration](#configuration)
    - [Decoration](#decoration)
    - [Disposal](#disposal)
  - [Example](#example)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Instance Registration

You can mark a field or property with the `[Instance]` attribute:

```csharp
[AttributeUsage(AttributeTargets.Field | AttributeTargets.Property, AllowMultiple = false, Inherited = false)]
public class InstanceAttribute : Attribute
{
    public InstanceAttribute(Options options = Options.Default);

    public Options Options { get; }
}
```

Any field/property so marked will be registered as an instance for the type of the field/property.

How often the field will be accessed is an implementation detail, so the field/property must not change whilst the container is alive.

## Usage in modules

In order for an instance field or property to be exported by a module it must be `public` and `static`. Instance fields and properties in containers can be `private` and `instance` if desired.

## Configuration

The registration can be modified using the `options` parameter. See the [documentation](https://github.com/YairHalberstadt/stronginject/wiki/Registration#options) for more details.

This allows you to register the instance as more than just the type of the the field/property. In particular it is possible to register the instance as it's:

1. Base classes
2. Interfaces
3. If it's an `IFactory<T>` or an `IAsyncFactory<T>`, as `T`, as well as to register `T`'s base classes, interfaces, and factories etc.

### Decoration

By default all instances can be decorated. This can be opted out of by applying `Options.DoNotDecorate`.

### Disposal

Instance fields and properties will not be disposed by StrongInject. This is necessary so that modules containing instance fields and properties can be shared among multiple containers.

However if they are decorated, the decorators will be disposed. If the decorator chooses to delegate disposal to the underlying instance, this means the instance property will be disposed.

For that reason it's best to use the `Options.DoNotDecorate` parameter on disposable instances.

## Example

```csharp
public class A {}
public class B : IDisposable { public void Dispose(){} }
public class C { public C(A a, B b){} }

public class Module
{
    [Instance(Options.DoNotDecorate)] public static readonly B _b = new B(); // Must be public and static. `Options.DoNotDecorate` is best practice as `B` is disposable.
}

[Register(typeof(C))]
public class Container : IContainer<C>
{
    [Instance] private readonly A a; // Can be private and instance.
    public Container(A a) => _a = a;
}
```
