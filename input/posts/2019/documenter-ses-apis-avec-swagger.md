Title: Documenter ses APIs avec Swagger
Published: 06/05/20
Tags: [Développement, ASP]
---

Swagger est une collection d'outils permettant de concevoir, générer et **documenter** des API REST. Il respose sur la spécification *OpenAPI* et est implémenté en ASP.NET Core à travers *SwashBuckle*.

### Installer et configurer SwashBuckle

Installons le paquet nuget [Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/) dans le projet d'application Web API que nous souhaitons documenter.

Ensuite dans le fichier ```startup.cs```, ajoutons dans la collection de services (méthode ```ConfigureService```) le générateur Swagger :

```csharp
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1.0", new OpenApiInfo { Title = "Mon application WebAPI", Version = "v1.0" });

    // Indiquer à Swagger l'emplacement de la documentation XML.
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    c.IncludeXmlComments(xmlPath);
});
```

Puis activons les intergiciels (méthode ```Configure```) pour dans l'ordre :

1. Servir le point de terminaison Swagger contenant la spécification OpenAPI au format JSON.
2. Servir l'application web Swagger UI.

```csharp
app.UseSwagger();
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Mon application WebAPI v1.0");
});
```

Enfin dans le fichier de configuration du projet, nous souhaitons pouvoir générer le fichier ```/bin/<NomDuProjet>.xml``` contenant l'ensemble de la documentation XML et ne pas être averti des membres publiques non documentés

```xml
<PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <NoWarn>CS1591</NoWarn>
</PropertyGroup>
```

A présent si nous démarrons notre projet, et que nous souhaitons accéder à l'adresse ```/swagger/v1/swagger.json``` précedemment définie, nous obtiendrons la spécification OpenAPI courante de notre application.

### Documenter ses points de terminaisons

Dans le contrôleur d'exemple ci dessous, un jeu d'attributs et de commentaires XML ont été utilisés afin de le documenter ainsi que son point de terminaison

```csharp
/// <summary>
/// Operations about felines
/// </summary>
[ApiController]
[Route("[controller]")]
[Produces(MediaTypeNames.Application.Json)]
public class FelinesController : ControllerBase
{
    /// <summary>
    /// Get felines that match the pattern
    /// </summary>
    /// <param name="pattern">Search pattern </param>
    /// <returns>The matching felines</returns>
    /// <response code="200">Returns the matching felines</response>
    /// <response code="400">Request cannot be completed, check output for more details</response>
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [HttpGet("contains/{pattern}")]
    public IEnumerable<string> Search([Required, StringLength(5)] string pattern)
    {
        var words = new List<string>() {"Tigre", "Guépard", "Jaguar"};
        return words.Where(p => p.Contains(pattern));
    }
}
```

A partir du code précedent, **SwaggerUI** sera en mesure de nous founir la documentation suivante à l'adresse ```/swagger/index.html``` :

![Documentation de l'API via Swagger UI](/media/development/swagger/swagger-ui.png)