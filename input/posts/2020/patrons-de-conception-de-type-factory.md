Title: Patrons de conception de type Factory
Published: 25/10/20
Tags: [Développement]
---

Dans cet article, nous allons parcourir les trois types de *fabriques* existante. En fonction du contexte et du besoin, elles ont pour principal objectif de réduire la **duplication de code** :

1. [Simple factory](#simple-factory)
2. [Factory method pattern](#factory-method-pattern)
3. [Abstract factory pattern](#abstract-factory-pattern)

Pour chacun de nos exemples, il s'agira de mettre en place une fabrique de création d'un instances de client ou serveur, HTTP ou FTP.

### Simple Factory

Son but est d'encapsuler la création d'objets à un seul endroit. Cependant il ne s'agit pas d'un patron de conception.

Dans l'exemple ci-dessous, nous disposons d'une **fabrique** d'applications utilisant le protocole HTTP.

```csharp
public sealed class HttpApplicationFactory
{
    public static IClientServerApplication CreateHttpApplication(ApplicationType applicationtype)
    {
        switch(applicationtype)
        {
            case ApplicationType.Client:
                return new HttpClient();
            case ApplicationType.Premium:
                return new HttpServer();
            default:
                throw new ArgumentException($"Non managed application type {applicationType}");
        }
    }
}
```

Nous pouvons l'utiliser de la manière suivante :

```csharp
HttpApplicationFactory factory = new HttpApplicationFactory();
IClientServerApplication application = factory.Create(ApplicationType.Client);
```

### Factory method pattern

Il permet la création d'objets, sans avoir à manipuler lors de l'utilisation, la classe concrète de l'objet qui sera créé.

Cela est possible en créant les objets à l'aide d'une **méthode de fabrication (factory method)**, qui peut-être :
* Soit spécifiée dans une interface et implémentée par les classes dépendantes
* Soit implémentée dans une classe de base et surchargée par les classes filles.

```csharp
public abstract class TcpApplicationFactory
{
    // Factory method
    public IClientServerApplication Create(ApplicationType type)
    {
        IClientServerApplication application;
        switch(type)
        {
            case ApplicationType.Client:
                application = CreateClient();
                break;
            case ApplicationType.Server:
                application = CreateServer();
                break;
            default:
                throw new NotImplementedException();
        }

        return application;
    }

    public abstract IClientServerApplication CreateClient();
    public abstract IClientServerApplication CreateServer();
}

public sealed class HttpApplicationFactory : TcpApplicationFactory
{
    public override IClientServerApplication CreateClient()
    {
        // Instanciation of HttpClient concrete class 
        return new HttpClient();
    }

    public override IClientServerApplication CreateServer()
    {
        // Instanciation of HttpServer concrete class
        return new HttpServer();
    }
}

public sealed class FtpApplicationFactory : TcpApplicationFactory
{
    public override IClientServerApplication CreateClient()
    {
        return new FtpClient();
    }

    public override IClientServerApplication CreateServer()
    {
        return new FtpServer();
    }
}
```

Ce qui permet à l'utilisation d'obtenir le code suivant dans le code appelant, et aucune mention de la classe concrète ```FtpClient``` n'y est présente.

```csharp
 IClientServerApplication application = new FtpServerFactory().Create(ApplicationType.Client);
```

Comparé au **Simple Factory**, cela permet repartir les logiques de création de chaque classe au lieu de les concentrer dans une seule et même méthode.

### Abstract factory pattern

La fabrique abstraite fournit un moyen d'**encapsuler un ensemble de fabrique** de la même thématique.

Le code client crée une instance concrète de la fabrique abstraite, puis l'utilise pour créer des objets concrets de la thématique

Le client ne se préoccupe pas de savoir quelle fabrique crée un objet concret, ni de quelle classe concrète est l'objet en question : il n'utilise que les interfaces génériques des objets produits.

```csharp
/// <summary>
/// Abstract factory
/// </summary>
public abstract class TcpApplicationFactory
{
    public abstract IClientServerApplication CreateClient();
    public abstract IClientServerApplication CreateServer();
}

#region Concretes factories
public sealed class FtpApplicationFactory : TcpApplicationFactory
{
    public override IClientServerApplication CreateClient()
    {
        return new FtpClient();
    }

    public override IClientServerApplication CreateServer()
    {
        return new FtpServer();
    }
}

public sealed class HttpApplicationFactory : TcpApplicationFactory
{
    public override IClientServerApplication CreateClient()
    {
        // Instanciation of HttpClient concrete class 
        return new HttpClient();
    }

    public override IClientServerApplication CreateServer()
    {
        // Instanciation of HttpServer concrete class
        return new HttpServer();
    }
}
#endregion

public sealed class Client
{
    private TcpApplicationFactory _factory { get; set; }
    public Client(TcpApplicationFactory factory)
    {
        _factory = factory;
    }

    public IClientServerApplication Create(ApplicationType type)
    {
        IClientServerApplication application;
        switch(type)
        {
            case ApplicationType.Client:
                application = CreateClient();
                break;
            case ApplicationType.Server:
                application = CreateServer();
                break;
            default:
                throw new NotImplementedException();
        }

        return application;
    }
}
```

Voici un exemple d'utilisation :

```csharp
IClientServerApplication application = new Client(new HttpApplicationFactory()).Create(ApplicationType.Client);
```