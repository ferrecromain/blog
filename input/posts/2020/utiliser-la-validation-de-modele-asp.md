Title: Utiliser la validation de modèle d'ASP
Published: 15/05/20
Tags: [ASP.NET]
---

Dans une application Web, il est **primordial** de vérifier les informations soumise par le client vers le serveur, afin de satisfaire les **exigences métier** et garantir la **cohérence des données**.

Le **framework ASP** dispose de son propre système de validation. Dans la pipeline d'execution d'une requête HTTP, celui ci est executé après la [liaison de données](https://docs.microsoft.com/fr-fr/aspnet/core/mvc/models/model-binding) et avant l'action d'un contrôleur.

### Validation de propriété

Pour effectuer une validation sur une propriété, nous utilisons les **attributs de validations**.  [Parmi ceux disponible dans .NET](https://docs.microsoft.com/fr-fr/dotnet/api/system.componentmodel.dataannotations), en voici quelques-uns fréquement utilisés :

Nom | Description | Exemple
--- | --- | ---
Required | La valeur est requise | ```[Required]```
StringLength | La longueur de chaîne est limité à *n* caractères | ```[StringLength(20)]```
Range | La valeur doit être comprise dans l'intervalle | ```[Range(1, int.MaxValue)]```
RegularExpression | La valeur doit satisfaire l'expression régulière | ```[RegularExpression(@"[A-Za-z0-9]+")```
MinLength et MaxLength | La valeur doit être au moins ou au plus égale à *n* | ```[MinLength(4)]```

Prenons un exemple, et définissons un modèle de transfert pour un utilisateur souhaitant mettre à jour ses informations :

```csharp
public class User
{
    [Required]
    public string FirstName { get; set; }
    [Required]
    public string LastName { get; set; }
    [Phone, Required]
    public string CellPhone { get; set; }
    [Phone]
    public string HomePhone { get; set; }
    [Required]
    public DateTime BirthDay { get; set; }
}
```

Une fois l'application lancée, nous lui soumettons le corps de requête suivant :
```json
{
  "firstName": "",
  "lastName": "",
  "homePhone": "toto"
}
```

En retour nous obtenons la réponse :
```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "|5679ad87-4b71d1b069bc6745.",
  "errors": {
    "LastName": [
      "The LastName field is required."
    ],
    "CellPhone": [
      "The CellPhone field is required."
    ],
    "FirstName": [
      "The FirstName field is required."
    ],
    "HomePhone": [
      "The HomePhone field is not a valid phone number."
    ]
  }
}
```

Nous pouvons remarquer que le champs ```BirthDay``` a passé la validation, et ce bien que nous lui ayons appliqué l'attribut ```[Required]``` mais pas fournit de valeur.

Ceci est du au fait que ```DateTime``` est un [type valeur](https://docs.microsoft.com/fr-fr/dotnet/csharp/language-reference/builtin-types/value-types)et donc qu'il aura forcément une valeur par défaut lors de la création du modèle. Voila pourquoi l'information n'est pas considérée comme *manquante*.

Un solution consiste à rendre le champs nullable

### Validation de classe

En reutilisant notre modèle ```User```, nous souhaitons exprimer le fait qu'au moins un numéro de téléphone valide soit renseigné.

Comme cette logique de validation implique **plus d'une propriété**, nous ne pouvons nous servir des attributs de validations. 

Au lieu de cela, nous allons utiliser l'interface [IValidatableObject](https://docs.microsoft.com/fr-fr/dotnet/api/system.componentmodel.dataannotations.ivalidatableobject) et l'implémenter de la manière suivante sur notre modèle :

```csharp
public class User : IValidatableObject
{
  ...
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {        
        if(HomePhone == null && CellPhone == null)
        {
            yield return new ValidationResult("At least one phone number is required", new [] {nameof(HomePhone), nameof(CellPhone)});
        }
    }
}
```

En soumettons la requête :
```json
{
  "firstName": "Jean",
  "lastName": "Dupont",
}
```

Nous obtenons la réponse attendue :
```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "|8a63c595-4030754928e01558.",
  "errors": {
    "CellPhone": [
      "At least one phone number is required"
    ],
    "HomePhone": [
      "At least one phone number is required"
    ]
  }
}
```

### Définir ses propres attributs de validations

Il est fort probable au cours du développement de notre application webAPI que nous ayons besoin d'avantages de logique de validations de propriété. 

Pour ce faire, nous pouvons concevoirs nos propres classes héritant de ```ValidationAttribute```, qui seront utilisés comme **attributs**. 

Voici quelques exemples assez parlant :

```csharp
using System.Collections;
using System.ComponentModel.DataAnnotations;
using System;

/// <summary>
/// Require the value is not null or the enumeration empty.
/// </summary>
[AttributeUsage(AttributeTargets.Property | AttributeTargets.Field | AttributeTargets.Parameter, AllowMultiple = false)]
public sealed class NotNullOrEmptyEnumerable : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        string error = $"'{validationContext.MemberName}' cannot be null or empty";

        IEnumerable enumerable = value as IEnumerable;

        if (enumerable != null && enumerable.GetEnumerator().MoveNext())
        {
            return ValidationResult.Success;
        }
        
        return new ValidationResult(error);
    }
}
```

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;

/// <summary>
/// Require the value is not null or the enumeration does not contains duplicate integer
/// </summary>
[AttributeUsage(AttributeTargets.Property | AttributeTargets.Field | AttributeTargets.Parameter, AllowMultiple = false)]
public class NoDuplicateInteger : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        var collection = value as ICollection<int>;
        if (collection == null)
            return new ValidationResult($"Parameter '{validationContext.MemberName}' is not a collection of integers");
        if (collection.Count == collection.Distinct().Count())
        {
            return ValidationResult.Success;
        }
        return new ValidationResult($"List '{validationContext.MemberName}' must not contains duplicated integers.");
    }
}
```

