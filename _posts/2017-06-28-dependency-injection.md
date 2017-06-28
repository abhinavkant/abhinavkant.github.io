---
layout: post
title: "Inversion of Control and Dependency Injection"
date: 2017-06-28 06:00:00 +05:30
tags: [patterns]
comments: true
---
> Inversion of control is a common characteristic of frameworks, so saying that these lightweight containers are special because they use inversion of control is like saying my car is special because it has wheels.

> Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various IoC advocates we settled on the name Dependency Injection.

ref: https://martinfowler.com/articles/injection.html#InversionOfControl

So the idea is to create a **loosely coupled** code. Design the code such that
* separating concerns
* binary references of its dependents has been reduced.
* code maintainability without adding complexity.
* increase testability.

Example of a tight coupled code

```csharp
public class Service
{
    public void Serve()
    {
        Console.WriteLine("Service Invoked");
    }
}

public class Client
{
    public Service _service;

    public Client()
    {
        _service = new Service();
    }

    public void StartService()
    {
        Console.WriteLine("Service Started");
        _service.Serve();
        //To Do: Some Stuff
    }
}

class Program
{
    static void Main(string[] args)
    {
        var c = new Client();
        c.Start();
    }
}
```

Above code we are bound to face issues:
* Client class needs to be aware of "Service" class.
* Any changes in Service method may result in changing the class consuming the it.
* The code is almost impossible to testable.
* It is difficult to switch out Service class and replace it with new implementation.

Let's refactor the code:

```csharp
public interface IService
{
    void Serve();
}

public class Service : IService
{
    public void Serve()
    {
        Console.WriteLine("Service Invoked");
    }
}

public class Client
{
    private readonly IService _service;

    public Client(IService service)
    {
        _service = service;
    }

    public void Start()
    {
        Console.WriteLine("Service Started");
        this._service.Serve();
        //To Do: Some Stuff
    }
}

class Program
{
    static void Main(string[] args)
    {
        var c = new Client(new Service());
        c.Start();
    }
}
```

Now lets see what have we achieved:
* We have made sure Client class only refers to the signature of IService rather than the concrete implementation.
* Code is more flexible, we can switch the implementation of IService when Client class is initialized a
* Client does not have to change.
* Code is testable.

## Ways to achieve IoC/DI:

### Constructor Injection

```csharp
public interface IService
{
    void Serve();
}

public class Service : IService
{
    public void Serve()
    {
        Console.WriteLine("Service Called");
        //To Do: Some Stuff
    }
}

public class Client
{
    private IService _service;

    public Client(IService service)
    {
        this._service = service;
    }

    public void Start()
    {
        Console.WriteLine("Service Started");
        this._service.Serve();
        //To Do: Some Stuff
    }
}
class Program
{
    static void Main(string[] args)
    {
        var client = new Client(new Service());
        client.Start();

        Console.ReadKey();
    }
}
```
### Property Injection

```csharp
public interface IService
{
    void Serve();
}

public class Service : IService
{
    public void Serve()
    {
        Console.WriteLine("Service Called");
        //To Do: Some Stuff
    }
}

public class Client
{
    private IService _service;

    public IService Service
    {
        set
        {
            this._service = value;
        }
    }

    public void Start()
    {
        Console.WriteLine("Service Started");
        this._service.Serve();
        //To Do: Some Stuff
    }
}

class Program
{
    static void Main(string[] args)
    {
        Client client = new Client();
        client.Service = new Service();
        client.Start();

        Console.ReadKey();
    }
}
```

### Method Injection

```csharp
public interface IService
{
    void Serve();
}

public class Service : IService
{
    public void Serve()
    {
        Console.WriteLine("Service Called");
        //To Do: Some Stuff
    }
}

public interface IClient
{
    void Start();
}

public class Client
{
    private IService _service;

    public IService Service
    {
        set
        {
            this._service = value;
        }
    }

    public void Start()
    {
        Console.WriteLine("Service Started");
        this._service.Serve();
        //To Do: Some Stuff
    }
}
class Program
{
    static void Main(string[] args)
    {
        Client client = new Client();
        client.Service = new Service();
        client.Start();

        Console.ReadKey();
    }
}
```

The most convenient way to achieve DI/IoC is to use constructor injection
##  IoC/DI Container

A framework to create dependencies and inject them automatically when required. It automatically creates objects based on request and inject them when required. DI Container helps us to manage dependencies with in the application in a simple and easy way.

### IoC/DI Frameworks:
* Autofac

    1. Register the instance

    ```csharp 
    var builder = new ContainerBuilder();
    builder.RegisterType<Service>().As<IService>();
    builder.RegisterType<Client>().As<IClient>();
    var container = builder.Build();
    ```

    2. Resolve and instance

    ```csharp
    var client = resolver.Resolve<IClient>();
    client.Start();
    ```

* StructureMap
* Unity
* NInject