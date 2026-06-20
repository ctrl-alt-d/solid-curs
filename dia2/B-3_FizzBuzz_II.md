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

# Segona part de la pràctica FizzBuzz

Continuem amb el TDD. Ara afegim els casos de negoci: `Fizz`, `Buzz` i `FizzBuzz`.

Partim del punt on `GetLabel(n)` retorna `n.ToString()`.

---

## Prova unitària IV: múltiple de 3 retorna `Fizz`

Afegim aquesta prova a `FizzBuzzTests.cs`:

```csharp
[Fact]
public void GetLabel_3_returns_Fizz()
{
    // Arrange
    var fb = new FizzBuzz();

    // Act
    var actual = fb.GetLabel(3);

    // Assert
    Assert.Equal("Fizz", actual);
}
```
---

## Implementació IV

Canviem `FizzBuzz.cs`:

```csharp
namespace FizzBuzzService;

public class FizzBuzz
{
    public string GetLabel(int n)
    {
        if (n % 3 == 0)
        {
            return "Fizz";
        }

        return n.ToString();
    }
}
```

Executem `dotnet test` i comprovem que tot passa.
---

## Prova unitària V: múltiple de 5 retorna `Buzz`

Afegim una segona prova:

```csharp
[Fact]
public void GetLabel_5_returns_Buzz()
{
    // Arrange
    var fb = new FizzBuzz();

    // Act
    var actual = fb.GetLabel(5);

    // Assert
    Assert.Equal("Buzz", actual);
}
```
---

## Implementació V

Actualitzem `FizzBuzz.cs`:

```csharp
namespace FizzBuzzService;

public class FizzBuzz
{
    public string GetLabel(int n)
    {
        if (n % 3 == 0)
        {
            return "Fizz";
        }

        if (n % 5 == 0)
        {
            return "Buzz";
        }

        return n.ToString();
    }
}
```

Executem `dotnet test`.
---

## Prova unitària VI: múltiple de 3 i 5 retorna `FizzBuzz`

Afegim la prova final d'aquesta fase:

```csharp
[Fact]
public void GetLabel_15_returns_FizzBuzz()
{
    // Arrange
    var fb = new FizzBuzz();

    // Act
    var actual = fb.GetLabel(15);

    // Assert
    Assert.Equal("FizzBuzz", actual);
}
```
---

## Implementació VI

Ara cal comprovar primer el cas combinat:

```csharp
namespace FizzBuzzService;

public class FizzBuzz
{
    public string GetLabel(int n)
    {
        if (n % 3 == 0 && n % 5 == 0)
        {
            return "FizzBuzz";
        }

        if (n % 3 == 0)
        {
            return "Fizz";
        }

        if (n % 5 == 0)
        {
            return "Buzz";
        }

        return n.ToString();
    }
}
```

Executem `dotnet test` per validar-ho tot.
---

## Reflexió

* Què passa si comprovem primer el múltiple de 3 abans del `FizzBuzz`?
* Quin valor aporta fer aquests canvis en passos petits?
* Podem refactoritzar mantenint totes les proves en verd?