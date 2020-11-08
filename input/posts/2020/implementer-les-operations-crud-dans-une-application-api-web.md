Title: Implémenter les opérations CRUD dans une application API web Restful
Published: 23/05/20
Tags: [ASP.NET web API]
---

Lorsque nous créons un projet d'application web API Restful, il est important de respecter un certain nombre de **contraintes**.

Nous allons partir d'un modèle de domaine ```UserModel``` avec lequel nous dédierons un contrôleur ```UsersController``` où nous implémenterons nos diverses actions [CRUD](https://fr.wikipedia.org/wiki/CRUD).

Les **ressources** (données échangées entre client/serveur) seront **représentées** (format de présentation) au format JSON.

Dans les actions de notre contrôleur, nous utilisons la logique de liaison de données (mapping), entre nos DTMs (Data Transfert Model) et notre modèle de domaine. Chaque opération à son propre DTM.

En outre chacune d'entre elles sera documenté à l'aide de Swagger.

### Implémentation du contrôleur

Le contrôleur regroupe un ensemble d'actions pour agir généralement sur une collection de données particulière.

```csharp
/// <summary>
/// Operations about users
/// </summary>
[ApiController]
[Route("/api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IQueryRepository<UserModel> _userQueryRepository;
    private readonly ICommandRepository<UserModel> _userCommandRepository;

    public UsersController
        (
            IQueryRepository<UserModel> userQueryRepository,
            ICommandRepository<UserModel> userCommandRepository
        )
    {
        _userQueryRepository = userQueryRepository;
        _userCommandRepository = userCommandRepository;
    }
}
```

Passons en revue certains éléments :
* ```ControllerBase``` : Classe sur laquelle dérivent les contrôleurs de notre application. <br />
A ne pas confondre avec la classe ```Controller``` qui hérite de ```ControllerBase``` et ajoute la prise en charge des vues, servant aux applications ASP.NET MVC.
* ```ApiController``` : Attribut permettant d'activer des comportements spécifiques, tel que :
    * Réponse ```400 Bad Request``` en cas d'exception levée lors de l'exécution d'une action.
    * Attributs de source de liaison (```FromBody```, ```FromRoute```...).
    * Obligation d'appliquer l'attribut ```Route``` sur le contrôleur.
* ```Route``` : Attribut permettant de lier un contrôleur à un *modèle de route* (ici ```/api/[controller]```)
* ```UsersController``` : Classe de contrôleur de notre application, son constructeur sert à l'injection des dépendances nécessaires (classes de services). <br/>
Son nom est soumis à deux contraintes :
    * Il est necesairement suffixé par ```Controller```.
    * Il doit necessairement prendre le pluriel. 

### Implémentation des actions

#### Lecture

Le verbe HTTP ```GET``` sert à l'obtention de ressources.

Nous pouvons exposer les données liées à notre modèle par les deux points de terminaisons ci-dessous :

```csharp
/// <summary>
/// Get a single user
/// </summary>
/// <param name="id">User identifier</param>
/// <response code="200">The matching user</response>
/// <response code="404">Requested user was not found</response>
/// <response code="400">Request cannot be completed, check output for more details</response>
[HttpGet("{id}")]
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async IActionResult GetAsync(int id)
{
    UserModel user = await _userQueryRepository.GetAsync(id);

    if(user == null)
        return NotFound(NotFoundStrings.EntityNotFound(id));
    
    return Ok(new UserGetDto(user));
}

/// <summary>
/// Get all available users
/// </summary>
/// <response code="200">Collection of users</response>
/// <response code="400">Request cannot be completed, check output for more details</response>
[HttpGet]
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async IActionResult GetAsync()
{
    IEnumerable<UserGetDto> users = await _userQueryRepository.GetAsync();
    return Ok(users.Select(m => new UserGetDto(m));
}
```

#### Création

Le verbe HTTP ```POST``` sert à la création de nouvelles ressources.

En cas de succès, nous renvoyons le statut ```201 Created``` et renseignons l'entête HTTP ```location``` par l'**URI** vers la ressource créée (ref : [RFC2616](https://tools.ietf.org/html/rfc2616#section-9.5)).

```csharp
/// <summary>
/// Create a new user
/// </summary>
/// <param name="dto">User informations</param>
/// <response code="201">User has been created successfuly</response>
/// <response code="400">Request cannot be completed, check output for more details</response>
[HttpPost]
[ProducesResponseType(StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async IActionResult PostAsync(UserPostDto dto)
{
    UserModel model = dto.ToModel();
    model = await _userCommandRepository.CreateAsync(model);
    return CreatedAtAction(nameof(Get), new { id = model.Id }, new UserGetDto(model));
}
```

#### Mise à jour

Le verbe HTTP ```PUT``` sert à la modification complète de ressources. 

Si nous souhaitons ne modifier qu'une seule propriété nous devons tout de même renseigner les autres car sinon elles seront remplies avec des valeurs par défaut.

En cas de succès, il est recommandé de renvoyer un corps de réponse vide avec le code HTTP ```204 No Content```. Si toutefois nous devions retourner des informations alors nous utiliserions le code HTTP ```200 Ok```.

```csharp
/// <summary>
/// Update an existing user
/// </summary>
/// <param name="dto">User informations</param>
/// <param name="id">User identifier</param>
/// <response code="201">User has been updated successfuly</response>
/// <response code="404">Requested user was not found</response>
/// <response code="400">Request cannot be completed, check output for more details</response>
[HttpPut("{id}"})]
[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async IActionResult PutAsync(int id, UserPostDto dto)
{
    if(user == null)
        return NotFound(NotFoundStrings.EntityNotFound(id));
    UserModel model = dto.ToModel();
    model = await _userCommandRepository.Update(model);
    return CreatedAtAction(nameof(Get), new { id = model.Id }, new UserGetDto(model));
}
```

#### Suppression

Le verbe HTTP ```DELETE``` sert à la suppression de ressources existantes.

En cas de succès, il est recommandé de renvoyer un corps de réponse vide avec le code HTTP ```204 No Content```. Si toutefois nous devions retourner des informations alors nous utiliserions le code HTTP ```200 Ok```.

```csharp
/// <summary>
/// Delete an existing user
/// </summary>
/// <param name="id">User identifier</param>
/// <response code="204">User has been deleted successfuly</response>
/// <response code="404">Requested user was not found</response>
/// <response code="400">Request cannot be completed, check output for more details</response>
[HttpDelete("{id}")]
[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async IActionResult DeleteAsync(int id)
{
    UserModel user = await _userQueryRepository.GetAsync(id);

    if(user == null)
        return NotFound(NotFoundStrings.EntityNotFound(id));
    
    await _userQueryRepository.DeleteAsync(id);
    return NoContent();
}
```