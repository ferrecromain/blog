Title: Utiliser la validation de modèle d'ASP
Published: 15/05/20
Tags: [Développement, ASP]
---

### Validation de propriétés

Pour effectuer une validation sur une propriété, nous pouvons utiliser les attributs de validations. En voici certains parmi les plus utilisés fournit par .NET :

* [Required]
* [StringLength]
* [Range]
* [RegularExpression]
* [MinLength] et [MaxLength]

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
    public string City { get; set; }
    [Required]
    public DateTime BirthDay { get; set; }
}
```

Une fois l'application lancée, nous lui soumettons le corps de requête suivant :
```json
{
  "firstName": "",
  "lastName": "",
  "homePhone": "toto",
  "city": "string"
}
```

La réponse serait alors la suivante :
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

Nous pouvons remarquer que le champs ```BirthDay``` n'a pas été renseigné, et ce bien que nous lui ayons appliqué l'attribut ```[Required]```.

Ceci est du au fait que ```DateTime``` est un type valeur, qu'il aura forcément une valeur par défaut lors de la création du modèle, et donc que l'information n'est pas considéré comme *manquante*.

# Validation de classe

En reutilisant notre modèle ```User```, nous souhaitons exprimer le fait qu'au moins un numéro de téléphone valide soit renseigné, et que s'il le sont tous les deux qu'ils doivent être différent.

Pour cela nous lui implémentons l'interface ```IValidatableObject```, et définissons la logique de validation suivante :

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

        if(HomePhone != null && CellPhone !=null && HomePhone == CellPhone)
        {
            yield return new ValidationResult($"{nameof(HomePhone)} and {nameof(CellPhone)} cannot be the same", new [] {nameof(HomePhone), nameof(CellPhone)});
        }
    }
}
```