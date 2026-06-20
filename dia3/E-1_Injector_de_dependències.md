

---
## Injector de dependències a .NET

Ja hem vist què és **IoC** i per què **DIP** ens ajuda a desacoblar el codi. Ara toca la part pràctica: com ho resol .NET amb el seu injector de dependències.

La idea és senzilla: en lloc de crear objectes amb `new` dins de cada classe, registrem els serveis en un contenidor i deixem que .NET els construeixi quan calgui.

## Què és el contenidor?

El contenidor de DI de .NET és una llista de serveis registrats. Aquests serveis es guarden dins d'un `IServiceCollection`.

```csharp
var services = new ServiceCollection();
```

Sobre aquesta col·lecció hi afegim els serveis que volem utilitzar:

```csharp
services.AddTransient<IEmailSender, SmtpEmailSender>();
services.AddScoped<IOrderRepository, OrderRepository>();
services.AddSingleton<IClock, SystemClock>();
```

Més tard, el contenidor crea un `IServiceProvider`, que és qui sap instanciar els objectes i resoldre les seves dependències.

## Els tres temps de vida

Quan registrem un servei, hem de decidir quant dura la seva instància.

### `AddTransient`

Crea una instància nova cada vegada que es demana el servei.

Útil per serveis petits i sense estat compartit.

```csharp
services.AddTransient<IEmailSender, SmtpEmailSender>();
```

Cada petició és una nova instància de la classe.

### `AddScoped`

Crea una instància per cada *scope*.

En aplicacions web, normalment un *scope* equival a una petició HTTP. Això vol dir que dins d'una mateixa petició es reutilitza la mateixa instància.

```csharp
services.AddScoped<IOrderRepository, OrderRepository>();
```

Aquest és un dels casos més típics en MVC o Web API, sobretot per repositoris i serveis que treballen amb dades de la petició.

### `AddSingleton`

Crea una sola instància per a tota l'aplicació.

```csharp
services.AddSingleton<IClock, SystemClock>();
```

És útil per serveis sense estat o per recursos costosos que volem compartir.

Compte: un `singleton` ha de ser segur en entorns amb concurrència, perquè el faran servir moltes peticions alhora.

## Exemple complet

Imagina aquestes classes:

```csharp
public interface IEmailSender
{
    void Send(string to, string body);
}

public class SmtpEmailSender : IEmailSender
{
    public void Send(string to, string body)
    {
        // enviar correu
    }
}

public class NotificationService
{
    private readonly IEmailSender _emailSender;

    public NotificationService(IEmailSender emailSender)
    {
        _emailSender = emailSender;
    }

    public void NotifyUser(string userEmail)
    {
        _emailSender.Send(userEmail, "Hola!");
    }
}
```

I el registre seria:

```csharp
// Declarem la col·lecció de "serveis" injectables
var services = new ServiceCollection();

services.AddTransient<IEmailSender, SmtpEmailSender>();
services.AddScoped<NotificationService>();

// Creem el proveidor de "serveis"
var serviceProvider = services.BuildServiceProvider();

// Accedim a un "servei" usant el proveidor
// Fixat que `NotificationService` rep al constructor un `IEmailSender``
// El serviceProvider s'ocuparà de crear l' `IEmailSender` segons el
// temps de vida que haguem definit.
var notifier = serviceProvider.GetRequiredService<NotificationService>();
```

El més important no és memoritzar la sintaxi, sinó veure la idea: la classe només declara què necessita, i **el contenidor s'encarrega de construir-ho**.

## Per què encaixa amb DIP?

Amb DIP, la classe depèn d'abstraccions, no d'implementacions concretes.

Aquí passa exactament això: `NotificationService` no coneix `SmtpEmailSender`, només coneix `IEmailSender`.

Això fa el codi més fàcil de provar, de canviar i de llegir.

## I MVC ja ho porta preparat?

Sí. A les aplicacions ASP.NET Core MVC, el contenidor ja ve preparat de sèrie.

Quan l'aplicació arrenca, `Program.cs` registra els serveis al `builder.Services`, que és un `IServiceCollection` ja preparat pel framework.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();
// Nosaltres podem afegir tots els serveis que volguem.
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

var app = builder.Build();
```

Això vol dir que controllers, serveis, repositoris i altres classes poden rebre dependències pel constructor sense que nosaltres hàgim de crear manualment cada objecte.

En MVC, el controlador també forma part d'aquest sistema:

```csharp
public class OrdersController : Controller
{
    private readonly IOrderRepository _repository;

    public OrdersController(IOrderRepository repository)
    {
        _repository = repository;
    }
}
```

El framework construeix el controlador i li injecta el que necessita.

## Service Locator: el que no hem de fer

Per acabar, un avís important. Si dins d'una classe injectem `IServiceProvider` i comencem a fer `GetRequiredService<T>()`, ja no estem fent DI de forma clara: estem fent servir un **Service Locator**.

```csharp
public class NotificationService
{
    private readonly IServiceProvider _serviceProvider;

    public NotificationService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public void Notify(string to, string body)
    {
        var sender = _serviceProvider.GetRequiredService<IEmailSender>();
        sender.Send(to, body);
    }
}
```

Això amaga les dependències reals i complica els tests. Si una classe necessita un servei, és millor declarar-lo al constructor directament.

## Resum ràpid

- `IServiceCollection` és on registrem els serveis.
- `AddTransient` crea una instància nova cada vegada.
- `AddScoped` reutilitza la mateixa instància dins del mateix *scope*.
- `AddSingleton` crea una sola instància per a tota l'aplicació.
- ASP.NET Core MVC ja porta aquest sistema preparat.
- La classe ha de rebre les dependències, no buscar-les per dins.

Més endavant ho aplicarem en una pràctica amb [solid-raigX](https://github.com/ctrl-alt-d/solid-raigX).

---