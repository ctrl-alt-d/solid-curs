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

# Construint una solució (amb proves)

Repassem com crear una solució que inclou proves. Ho farem des de la línia de comandes. Microsoft té un document (en castellà) semblant al que farem: [Pruebas unitarias de C# en .NET mediante dotnet test y xUnit](https://learn.microsoft.com/es-es/dotnet/core/testing/unit-testing-csharp-with-xunit)

---

## 1. Crear la solució i entrar al directori
Aquestes comandes creen la solució nova i ens situen dins la carpeta de treball.
```bash
dotnet new sln -o Calculadora
cd Calculadora
```

---

## 2. Crear el projecte principal (biblioteca)
Aquí generem la llibreria on implementarem la lògica de la calculadora.
```bash
dotnet new classlib -o ServeiCalculadora
```

---

## 3. Afegir el projecte principal a la solució
Afegim el projecte principal al fitxer de solució per poder-lo gestionar conjuntament.
```bash
dotnet sln add ./ServeiCalculadora/ServeiCalculadora.csproj
```

---

## 4. Crear el projecte de proves amb xUnit
Creem un projecte de proves unitàries amb xUnit per validar el comportament del codi.
```bash
dotnet new xunit -o ServeiCalculadora.Proves
```

---

## 5. Referenciar el projecte principal des del projecte de proves
Enllacem el projecte de proves amb el projecte principal per poder provar-ne les classes.
```bash
dotnet add ./ServeiCalculadora.Proves/ServeiCalculadora.Proves.csproj reference ./ServeiCalculadora/ServeiCalculadora.csproj
```

---

## 6. Afegir el projecte de proves a la solució
Incloem també el projecte de proves dins la solució perquè s'executi amb la resta.
```bash
dotnet sln add ./ServeiCalculadora.Proves/ServeiCalculadora.Proves.csproj
```

---

## 7. Executar les proves
Amb aquesta ordre llancem tots els tests de la solució i comprovem si passen.
```bash
dotnet test
```

---

## Estructura final esperada
Aquesta és la disposició de carpetes i fitxers que hauríem de tenir en aquest punt.
```bash
Calculadora/
├── Calculadora.slnx
├── ServeiCalculadora/
│   ├── ServeiCalculadora.csproj
│   └── Class1.cs
└── ServeiCalculadora.Proves/
    ├── ServeiCalculadora.Proves.csproj
    └── UnitTest1.cs
```
---

## Creem una funció de suma:

Canviem el nom de `Class1.cs` a `ServeiCalculadora.cs` i posem el següent contingut

---

```c#
namespace ServeiCalculadora;
public class ServeiCalculadora
{
    public int Suma(int sumand1, int sumand2)
    {
        return sumand1 + sumand2;
    }
}
```

Comprovem que tot compila

---

## Fem la prova unitària per comprovar que la funció funciona:

Li canviem el nom a `UnitTest1.cs` a `ProvesServeiCalculadora.cs`

---

```c#
using Xunit;
using ServeiCalculadora;

namespace ServeiCalculadora.Proves;

public class ProvesServeiCalculadora
{
    [Fact]
    public void LaCalculadoraSapSumarDosNombres()
    {
        // Arrange
        ServeiCalculadora servei = new();

        // Act
        int result = servei.Suma(2, 2);

        // Assert
        Assert.Equal(4, result);
    }
}
```

---


Executem la prova unitària: `dotnet test` també des de l'IDE.

> [!TIP]
> Fixa't que no hem fet interfície d'usuari per provar la funcionalitat que hem creat.

---

## Creant IServeiCalculadora

Veurem més endavant la importància de les `interfaces`. De moment, fem el pas bàsic: separar el contracte (interfície) de la implementació (classe).

---

### 1. Creem la interfície

Al projecte `ServeiCalculadora`, creem un fitxer nou: `IServeiCalculadora.cs`.

Aquest codi defineix el contracte mínim que haurà de complir qualsevol servei de calculadora.

```c#
namespace ServeiCalculadora;

public interface IServeiCalculadora
{
    int Suma(int sumand1, int sumand2);
}
```

La interfície defineix **què** ha de fer el servei, però no **com** ho fa.

---

### 2. Fem que el servei compleixi la interfície

Editem `ServeiCalculadora.cs` perquè implementi `IServeiCalculadora`:

Aquí adaptem la classe perquè segueixi el contracte de la interfície.

```c#
namespace ServeiCalculadora;

public class ServeiCalculadora : IServeiCalculadora
{
    public int Suma(int sumand1, int sumand2)
    {
        return sumand1 + sumand2;
    }
}
```

Ara la classe queda obligada a implementar tots els mètodes definits a la interfície.

---

### 3. Al test, fem servir la interfície en lloc de la classe

Al test, podem seguir creant la implementació real, però tipant la variable com a interfície:

---

En aquesta prova usem la interfície com a tipus per reduir l'acoblament amb la classe concreta.

```c#
using Xunit;
using ServeiCalculadora;

namespace ServeiCalculadora.Proves;

public class ProvesServeiCalculadora
{
    [Fact]
    public void LaCalculadoraSapSumarDosNombres()
    {
        // Arrange
        IServeiCalculadora servei = new ServeiCalculadora();

        // Act
        int result = servei.Suma(2, 2);

        // Assert
        Assert.Equal(4, result);
    }
}
```
---

Això ens ajuda a desacoblar el test de la implementació concreta. Ara no li veiem utilitat, però més endavant veurem els avantatges de treballar amb la interface.

Executem de nou les proves amb `dotnet test`.

