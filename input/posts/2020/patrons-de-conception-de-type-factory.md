Title: Patrons de conception de type Factory
Published: 25/10/20
Tags: [Développement]
---

### Simple Factory

Son but est d'encapsuler la création d'objets à un seul endroit. Il ne s'agit pas à proprement parler d'un patron de conception.

Dans l'exemple ci dessous, nous disposons d'une **fabrique** de différents types de compte.

```csharp
public sealed class AccountFactory
{
    public static IAccount CreateAccount(AccountType type, AccountCategory category, string email)
    {
        switch(accountType)
        {
            case AccountType.Basic:
                return new BasicAccount(email, category);
            case AccountType.Premium:
                return new PremiumAccount(email, category);
            case AccountType.Ultimate:
                return new UltimateAccount(email, category);
            default:
                throw new ArgumentException($"Type de compte '{accountType}' non géré");
        }
    }
}
```

Nous pouvons l'utiliser de la manière suivante :
```csharp
try
{
    AccountType accountType = AccountType.Basic;
    AccountCategory accountCategory = AccountCategory.Professional
    IAccount account = AccountFactory.CreateAccount(accountType, accountCategory, "johndoe@domain.tld");
    Console.WriteLine($"Instanciation d'un nouveau compte de type '{accountType}' (classe : '{account.GetType().Name}') réussi");
}
catch(Exception ex)
{
    Console.WriteLine(ex.Message);
}
```

```console
Instanciation d'un nouveau compte de type 'Basic' (classe : 'BasicAccount') réussi
```

A noter qu'au lieu d'instancier directement nos modèles, nous pouvons les récuperer par une méthode de service ou d'appel à une base de données.

### Factory method pattern

Il permet la création d'objets sans utiliser la classe concrete de l'objet qui sera créé.

Cela est possible en créant les objets à l'aide d'une **méthode de fabrication (factory method)**, qui peut-être :
* Soit spécifiée dans une interface et implémentée par les classes dépendantes
* Soit implémentée dans une classe de base et surchargée par les classes filles.

```csharp
public abstract class AccountFactory
{
    // Factory method
    public IAccount GenerateAccount(string email, AccountType type)
    {
        IAccount account;
        switch(type)
        {
            case AccountType.Basic:
                account = CreateBasicAccount(email);
                break;
            case AccountType.Premium:
                account = CreatePremiumAccount(email);
                break;
            case AccountType.Ultimate:
                account = CreateUltimateAccount(email);
                break;
            default:
                throw new NotImplementedException();
        }

        return account;
    }

    public abstract IAccount CreateBasicAccount(string email);
    public abstract IAccount CreatePremiumAccount(string email);
    public abstract IAccount CreateUltimateAccount(string email);
}

public sealed class ProfessionalAccountFactory : AccountFactory
{
    public override IAccount CreateBasicAccount(string email)
    {
        // Instanciation of BasicAccount concrete class 
        return new BasicAccount()
        { 
            Email = email, 
            Category = AccountCategory.Professional 
        };
    }

    public override IAccount CreatePremiumAccount(string email)
    {
        // Instanciation of PremiumAccount concrete class
        return new PremiumAccount() 
        { 
            Email = email, 
            Category = AccountCategory.Professional 
        };
    }

    public override IAccount CreateUltimateAccount(string email)
    {
        // Instanciation of PremiumAccount concrete class
        return new UltimateAccount() 
        { 
            Email = email, 
            Category = AccountCategory.Professional 
        };
    }
}

public sealed class PersonalAccountFactory : AccountFactory
{
    public override IAccount CreateBasicAccount(string email)
    {
        return new BasicAccount()
        { 
            Email = email, 
            Category = AccountCategory.Personal 
        };
    }

    public override IAccount CreatePremiumAccount(string email)
    {
        return new PremiumAccount() 
        { 
            Email = email, 
            Category = AccountCategory.Personal 
        };
    }

    public override IAccount CreateUltimateAccount(string email)
    {
        throw new NotImplementedException();
    }
}
```

Ce qui permet à l'utilisation d'obtenir le code suivant dans le code appellant, et aucune mention de la classe concrête ```BasicAccount``` n'y est présente.

```csharp
 IAccount account = new ProfessionalAccountFactory().GenerateAccount("johndoe@domain.tld", AccountType.Basic);
Console.WriteLine($"Création du compte nom : {account.Email} et catégorie : {account.Category}");
```

Comparé au **Simple Factory**, celà permet repartir les logiques de création de chaque classes au lieu de les concentrer dans une seule et même méthode.

### Abstract factory pattern

La fabrique abstraite fournit un moyen d'encapsuler un ensemble de fabrique de la même thématique.

Le code client crée une instance concrète de la fabrique abstraite, puis l'utilise pour créer des objets concrets de la thématique

Le client ne se préoccupe pas de savoir quelle fabrique crée un objet concret, ni de quelle classe concrète est l'objet en question : il n'utilise que les interfaces génériques des objets produits.

```csharp
/// <summary>
/// Abstract factory
/// </summary>
public abstract class AccountFactory
{
    public abstract IAccount CreateBasicAccount(string email);
    public abstract IAccount CreatePremiumAccount(string email);
    public abstract IAccount CreateUltimateAccount(string email);
}

#region Concretes factories
public sealed class ProfessionalAccountFactory : AccountFactory
{
    public override IAccount CreateBasicAccount(string email)
    {
        return new BasicAccount()
        { 
            Email = email, 
            Category = AccountCategory.Professional 
        };
    }

    public override IAccount CreatePremiumAccount(string email)
    {
        return new PremiumAccount() 
        { 
            Email = email, 
            Category = AccountCategory.Professional 
        };
    }

    public override IAccount CreateUltimateAccount(string email)
    {
        return new UltimateAccount() 
        { 
            Email = email, 
            Category = AccountCategory.Professional 
        };
    }
}

public sealed class PersonalAccountFactory : AccountFactory
{
    public override IAccount CreateBasicAccount(string email)
    {
        return new BasicAccount()
        { 
            Email = email, 
            Category = AccountCategory.Personal 
        };
    }

    public override IAccount CreatePremiumAccount(string email)
    {
        return new PremiumAccount() 
        { 
            Email = email, 
            Category = AccountCategory.Personal 
        };
    }

    public override IAccount CreateUltimateAccount(string email)
    {
        throw new NotImplementedException();
    }
}
#endregion

public sealed class Client
{
    private AccountFactory _factory { get; set; }
    public Client(AccountFactory factory)
    {
        _factory = factory;
    }

    public IAccount CreateAccount(string email, AccountType type)
    {
        IAccount account;
        switch(type)
        {
            case AccountType.Basic:
                account = _factory.CreateBasicAccount(email);
                break;
            case AccountType.Premium:
                account = _factory.CreatePremiumAccount(email);
                break;
            case AccountType.Ultimate:
                account = _factory.CreateUltimateAccount(email);
                break;
            default:
                throw new NotImplementedException();
        }

        return account;
    }
}
```

Voici un exemple d'utilisation :

```csharp
IAccount account = new Client(new ProfessionalAccountFactory())
    .CreateAccount("johndoe@domain.tld", AccountType.Basic);
Console.WriteLine($"Création du compte nom : {account.Email} et catégorie : {account.Category}");
```