---
layout: post
title: "Dependency Injection/IoC"
date: 2017-06-26 09:15:00 +05:30
tags: [patterns]
---
## Dependency Injection Pattern

*Dependency Injection* pattern describes a mechanism to develop a **loosely coupled** code reducing the tight coupling between code and alows an effective way to introduce changes in code without adding complexity.

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

With above code we are bound to face following issues:
* Client class needs to be aware of "Service" class.
* Any changes in Service method may result in changing the class consuming the it.
* The code is almost impossible to testable.
* It is difficult to switch out Service class and replace it with new implementation.

Let's refactor the above code:
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
With the code modification:
* We have made sure Client class only reers to the signature of IService rather than the implementation.
* Code is not flexiable enough, we can switch the concreate implementation of IService and Client does not have to change.
* Code has become testable.