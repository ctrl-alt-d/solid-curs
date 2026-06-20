# Pràctica: injectar serveis en una aplicació MVC

## Objectiu

Integrar un servei extern dins una app ASP.NET Core MVC fent servir injecció de dependències (DI), i entendre com canviar d'implementació sense tocar el controlador ni la vista.

---

## Context

Partim del repositori [ctrl-alt-d/solid-di-example](https://github.com/ctrl-alt-d/solid-di-example), que inclou:

- `FotoService.Abstractions`: l'abstracció `IFotoDogService`.
- `FotoServiceHardCoded`: implementació simple (retorna una URL fixa).
- `FotoServiceDogCeo`: implementació real que consulta una API externa.
- `FotoServiceDogCeo.IntegrationTests`: proves d'integració de la implementació `DogCeo`.

---

## Passes de la pràctica

### 1. Clonar el repositori

```bash
git clone https://github.com/ctrl-alt-d/solid-di-example
cd solid-di-example
```

### 2. Crear el projecte MVC i afegir-lo a la solució

```bash
dotnet new mvc -o web
dotnet sln add web
cd web
```

### 3. Afegir referències als projectes de servei

```bash
dotnet add reference ../FotoService.Abstractions
dotnet add reference ../FotoServiceHardCoded
dotnet add reference ../FotoServiceDogCeo
```

### 4. Registrar el servei al contenidor DI

Fitxer: `web/Program.cs`

Afegim els `using` necessaris:

```csharp
using FotoService.Abstractions;
using FotoServiceDogCeo;
```

Registrem el servei:

```csharp
builder.Services.AddControllersWithViews();
builder.Services.AddTransient<IFotoDogService, FotoDogCeoService>();
builder.Services.AddHttpClient();
```

> `FotoDogCeoService` necessita `IHttpClientFactory`, per això cal `AddHttpClient()`.

Si vols provar la versió hardcoded, canvia `FotoDogCeoService` per la implementació concreta que hi hagi dins del projecte `FotoServiceHardCoded`.

### 5. Injectar la dependència al controlador

Fitxer: `web/Controllers/HomeController.cs`

```csharp
using FotoService.Abstractions;
using Microsoft.AspNetCore.Mvc;
using web.Models;

namespace web.Controllers;

public class HomeController(IFotoDogService fotoDogService) : Controller
{
    public async Task<IActionResult> Index()
    {
        ViewBag.UrlFotoGos = await fotoDogService.GetRandomDogImageUrlAsync();
        return View();
    }
}
```

### 6. Mostrar la foto a la vista

Fitxer: `web/Views/Home/Index.cshtml`

```cshtml
@{
    ViewData["Title"] = "Home Page";
}

<div class="text-center">
    <h1 class="display-4">Welcome</h1>

    <img src="@ViewBag.UrlFotoGos" alt="Foto aleatòria d'un gos" />

    <p>
        Learn about
        <a href="https://learn.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.
    </p>
</div>
```

### 7. Executar l'aplicació

```bash
dotnet run
```

Obre l'URL que et mostri la consola (habitualment `https://localhost:xxxx`) i comprova que es pinta una foto.

---

## Resultat esperat

- El controlador no sap d'on surt la foto: només coneix `IFotoDogService`.
- Canviar d'implementació és només canviar el registre a `Program.cs`.
- La UI continua igual encara que canviï el proveïdor de dades.

---

## Errors habituals

1. No registrar el servei al DI: error de resolució de dependències en arrencar.
2. Oblidar `AddHttpClient()` amb `FotoDogCeoService`: Ens donarà error en temps d'execució perquè no sabrà instanciar les dependències (`FotoDogCeoService` té com a dependència `IHttpClientFactory`).

---

## Relació amb SOLID

- **DIP (Dependency Inversion Principle):** el controlador depèn d'una abstracció (`IFotoDogService`).
- **OCP (Open/Closed Principle):** podem afegir noves implementacions sense modificar el controlador.
- **SRP (Single Responsibility Principle):** cada peça té una responsabilitat clara (controller, servei i vista), cosa que facilita manteniment i proves.

---

## Punt important

Hem pogut provar que `FotoDogCeoService` funciona correctament sense haver d'engegar l'aplicació web, mitjançant els testos d'integració.