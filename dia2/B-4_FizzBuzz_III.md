---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
style: |
  pre,
  pre code {
    font-family: Consolas, Monaco, 'Courier New', monospace;
    font-size: 14px;
  }
---

# Tercera part de la pràctica FizzBuzz

En aquesta part farem 3 coses:

* Refactoritzar `GetLabel` mantenint els tests en verd.
* Crear un mètode per obtenir els `N` primers valors de FizzBuzz.
* Crear una UI de consola (`FizzBuzzUI`) que pregunti fins a quin nombre volem mostrar.

---

## 1. Refactorització de `GetLabel`

Partim d'una implementació funcional. Ara la fem més llegible sense canviar el comportament.

### Objectiu de la refactorització

* Evitar repetir comprovacions amb `%`.
* Donar un nom semàntic a la regla de "múltiple de".
* Mantenir l'ordre correcte: primer `FizzBuzz`, després `Fizz`, després `Buzz`.

---
### Implementació refactoritzada

```csharp
namespace FizzBuzzService;

public class FizzBuzz
{
    public string GetLabel(int n)
    {
        bool esFizz = EsMultipleDe(n, 3);
        bool esBuzz = EsMultipleDe(n, 5);

        if (esFizz && esBuzz)
        {
            return "FizzBuzz";
        }

        if (esFizz)
        {
            return "Fizz";
        }

        if (esBuzz)
        {
            return "Buzz";
        }

        return n.ToString();
    }

    private static bool EsMultipleDe(int n, int divisor)
    {
        return n % divisor == 0;
    }
}
```

---
## 2. Mètode per mostrar els N primers valors de FizzBuzz

Ara afegim un mètode nou al mateix servei per generar la seqüència de l'1 a `n`.

```csharp
namespace FizzBuzzService;

public class FizzBuzz
{
    public string GetLabel(int n)
    {
        bool esFizz = EsMultipleDe(n, 3);
        bool esBuzz = EsMultipleDe(n, 5);

        if (esFizz && esBuzz)
        {
            return "FizzBuzz";
        }

        if (esFizz)
        {
            return "Fizz";
        }

        if (esBuzz)
        {
            return "Buzz";
        }

        return n.ToString();
    }

    public IEnumerable<string> GetFirstNLabels(int n)
    {
        for (int i = 1; i <= n; i++)
        {
            yield return GetLabel(i);
        }
    }

    private static bool EsMultipleDe(int n, int divisor)
    {
        return n % divisor == 0;
    }
}
```

---
## 3. Tests del nou mètode

Podem afegir proves simples a `FizzBuzzTests.cs`:

```csharp
[Fact]
public void GetFirstNLabels_5_returns_expected_sequence()
{
    // Arrange
    var fb = new FizzBuzz();

    // Act
    var actual = fb.GetFirstNLabels(5).ToArray();

    // Assert
    Assert.Equal(new[] { "1", "2", "Fizz", "4", "Buzz" }, actual);
}

[Fact]
public void GetFirstNLabels_15_last_is_fizzbuzz()
{
    // Arrange
    var fb = new FizzBuzz();

    // Act
    var actual = fb.GetFirstNLabels(15).Last();

    // Assert
    Assert.Equal("FizzBuzz", actual);
}
```

---

## 4. Creació de `FizzBuzzUI` (projecte consola)

Creem un projecte de consola nou dins la solució:

```bash
dotnet new console --use-program-main -o FizzBuzzUI
dotnet sln add ./FizzBuzzUI/FizzBuzzUI.csproj
dotnet add ./FizzBuzzUI/FizzBuzzUI.csproj reference ./FizzBuzzService/FizzBuzzService.csproj
```
---

## 5. Implementació de la UI

A `FizzBuzzUI/Program.cs`, fem que l'usuari indiqui fins a quin nombre vol mostrar:

```csharp
using FizzBuzzService;

namespace FizzBuzzUI;

internal class Program
{
    private static void Main(string[] args)
    {
        Console.Write("Fins a quin nombre vols mostrar FizzBuzz? ");
        string? input = Console.ReadLine();

        if (!int.TryParse(input, out int limit) || limit <= 0)
        {
            Console.WriteLine("Introdueix un enter positiu.");
            return;
        }

        var fizzBuzz = new FizzBuzz();

        foreach (var value in fizzBuzz.GetFirstNLabels(limit))
        {
            Console.WriteLine(value);
        }
    }
}
```
---

Executem:

```bash
dotnet run --project ./FizzBuzzUI/FizzBuzzUI.csproj
```
---

## Reflexió final

* Hem separat la lògica (`FizzBuzzService`) de la presentació (`FizzBuzzUI`).
* Hem refactoritzat sense canviar comportament gràcies als tests.
* Ara és fàcil fer noves interfícies (web, API, etc.) reutilitzant el servei.
