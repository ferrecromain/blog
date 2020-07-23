Title: Implementer les opérations CRUD dans une application API web Restful
Published: 23/05/20
Tags: [ASP.NET web API]
---

Lorsque nous créons un projet d'application web API Restful, il est important de respecter un certains nombre de **contraintes**.

Nous allons partir d'un modèle de domaine ```UserModel``` avec lequel nous dédierons un contrôleur ```UsersController``` où nous implémenterons nos diverses actions **CRUD**.

Les **ressources** (données échangées entre client/serveur) seront **représentées** (format de présentation) au format JSON.

Dans les actions de notre contrôleur, nous utilisons la logique de liaison de données (mapping), entre nos DTMs (Data Transfert Model) et notre modèle de domaine. Chaque opération à son propre DTM.

En outre chacune d'entre elles sera documenté à l'aide de Swagger.

### Lecture

Le verbe HTTP ```GET``` sert à l'obtention de ressources.

Nous pouvons exposer les données liées à notre modèle par les deux points de terminaisons ci dessous :

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
public IActionResult Get(int id)
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
public async IActionResult Get()
{
    IEnumerable<UserGetDto> users = await _userQueryRepository.GetAsync();
    return Ok(users.Select(m => new UserGetDto(m));
}
```

### Création

Le verbe HTTP ```POST``` sert à la création de nouvelles ressources.

En cas de succés, nous renvoyons le status ```201 Created``` et renseignons l'entête HTTP ```location``` par l'**URI** vers la ressource créée (ref : [RFC2616](https://tools.ietf.org/html/rfc2616#section-9.5)).

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
public async IActionResult Post(UserPostDto dto)
{
    UserModel model = dto.ToModel();
    model = await _userCommandRepository.CreateAsync(model);
    return CreatedAtAction(nameof(Get), new { id = model.Id }, new UserGetDto(model));
}
```

### Mise à jour

Le verbe HTTP ```PUT``` sert à la modification complête de ressources.

Si nous souhaitons ne modifier qu'une seule propriété nous devons tout de même renseigner les autres, ou elle seront remplies avec des valeurs par défaut suivant leur type.

En cas de succés, il est recommandé de renvoyer un corps de réponse vide avec le code HTTP ```204 No Content```. Si toutefois nous devions retourner des informations alors nous utiliserions le code HTTP ```200 Ok```.

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
public async IActionResult Post(int id, UserPostDto dto)
{
    if(user == null)
        return NotFound(NotFoundStrings.EntityNotFound(id));
    UserModel model = dto.ToModel();
    model = await _userCommandRepository.Update(model);
    return CreatedAtAction(nameof(Get), new { id = model.Id }, new UserGetDto(model));
}
```

### Suppression

Le verbe HTTP ```DELETE``` sert à la suppression de ressources existantes.

En cas de succés, il est recommandé de renvoyer un corps de réponse vide avec le code HTTP ```204 No Content```. Si toutefois nous devions retourner des informations alors nous utiliserions le code HTTP ```200 Ok```.

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
public async IActionResult Delete(int id)
{
    UserModel user = await _userQueryRepository.GetAsync(id);

    if(user == null)
        return NotFound(NotFoundStrings.EntityNotFound(id));
    
    await _userQueryRepository.DeleteAsync(id);
    return NoContent();
}
```