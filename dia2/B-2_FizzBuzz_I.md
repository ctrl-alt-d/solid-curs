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

# Kata TDD FizzBuzz

Anem a provar si ens agrada o no el TDD. Farem la Kata FizzBuzz, molt comeguda per a iniciar-se en TDD.

Mostrar els 100 primer nombres enters ... però:

* Si el nombre és múltiple de 3 posem `Fizz`
* Si el nombre és múltiple de 3 posem `Buzz`
* Si el nombre és múltiple de 3 i de 5 posem `FizzBuzz`

---

Exemple:

```
1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
...
```

---

## Creem entorn

Seguiu aquests passos des de la línia de comandes per crear la solució i afegir els projectes (classe i proves):

```bash
dotnet new sln -o fizzbuzz
cd fizzbuzz

dotnet new classlib -o FizzBuzzService
dotnet sln add ./FizzBuzzService/FizzBuzzService.csproj

dotnet new xunit -o FizzBuzzService.Tests
dotnet add ./FizzBuzzService.Tests/FizzBuzzService.Tests.csproj reference ./FizzBuzzService/FizzBuzzService.csproj
dotnet sln add ./FizzBuzzService.Tests/FizzBuzzService.Tests.csproj
```

---

## Canviem el nom als fitxers .cs

* Canviem el nom de `Class1.cs` a `FizzBuzz.cs` 
* Canviem el nom de `UnitTest1.cs` a `FizzBuzzTests.cs` 

---

## Estructura final esperada

```
fizzbuzz/
├── fizzbuzz.sln
├── FizzBuzzService/
│   ├── FizzBuzzService.csproj
│   └── FizzBuzz.cs
└── FizzBuzzService.Tests/
    ├── FizzBuzzService.Tests.csproj
    └── FizzBuzzTests.cs
```

---

# Comencem el TDD

Com que fem TDD, comencem escrivint el test, un cop el test falli o no compili farem la implementació per tal que el test passi.

---

## Proves unitàries I

Crearem proves amb xUnit a `FizzBuzzService.Tests`. Exemple de `FizzBuzzTests.cs`:

```csharp
using Xunit;
using FizzBuzzService;

namespace FizzBuzzService.Tests;

public class FizzBuzzTests
{
    [Fact]
    public void PucInstanciarFizzBuzz()
    {
        // Arrange

        // Act
        var fb = new FizzBuzz(); // La classe encara no existeix

        // Assert
        
    }
}
```

Executa el test i no complilarà. Fem la implementació.

---

## Implementació: `FizzBuzzService` I

Crearem una classe senzilla que exposi la lògica per obtenir l'etiqueta d'un nombre. Crear el fitxer `FizzBuzz.cs` dins `FizzBuzzService` amb aquest contingut:

```csharp
namespace FizzBuzzService;

public class FizzBuzz
{
}
```

---

## Proves unitàries II

Anem a cridar el mètode `GetLabel`:

```csharp
using Xunit;
using FizzBuzzService;

namespace FizzBuzzService.Tests;

public class FizzBuzzTests
{
    [Fact]
    public void GetLabel_member_exists_text()
    {
        // Arrange
        var fb = new FizzBuzz();

        // Act
        var actual = fb.GetLabel(1);

        // Assert
        Assert.Equal(1, actual);
    }
}
```

Executa el test i no compilarà. Fem la implementació.

---

## Implementació: `FizzBuzzService` II

Crearem una classe senzilla que exposi la lògica per obtenir l'etiqueta d'un nombre. Crear el fitxer `FizzBuzz.cs` dins `FizzBuzzService` amb aquest contingut:

```csharp
namespace FizzBuzzService;

public class FizzBuzz
{
    public string GetLabel(int n)
    {
        throw new NotImplementedException();
    }
}
```

Tornem a executar les proves unitàries, ara passaran.

---

## Proves unitàries III

Anem a cridar el mètode `GetLabel`:

```csharp
using Xunit;
using FizzBuzzService;

namespace FizzBuzzService.Tests;

public class FizzBuzzTests
{
    [Fact]
    public void GetLabel_1_returns_!()
    {
        // Arrange
        var fb = new FizzBuzz();

        // Act
        var actual = fb.GetLabel(1);

        // Assert
        Assert.Equal(1, actual);
    }
}
```

---

## Implementació: `FizzBuzzService` III

Crearem una classe senzilla que exposi la lògica per obtenir l'etiqueta d'un nombre. Crear el fitxer `FizzBuzz.cs` dins `FizzBuzzService` amb aquest contingut:

```csharp
namespace FizzBuzzService;

public class FizzBuzz
{
    public string GetLabel(int n)
    {
        return n.ToString();
    }
}
```

---

## Reflexió

* Per què estem fent passes tant petites?
* Hem de conservar tots els tests que estem escrivint?
* Per què escrivim codi que sabem que no compila?
