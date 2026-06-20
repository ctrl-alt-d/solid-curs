---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
---

# Dobles de Proves

## Per què existeixen els dobles de prova

Quan testem una classe de manera **aïllada**, sovint necessitem dependències que:

- Accedeixin a una base de dades real
- Cridin un servei extern
- Tinguin comportaments difícils de provocar en un test

Per evitar-ho, substituïm les dependències reals per **dobles de prova**.

---

## Tipus de dobles (Robert C. Martin)

Robert C Martin, al seu llibre *"La artesanía del código limpio"*, defineix cinc tipus:

| Tipus | Propòsit |
|-------|----------|
| **Dummy** | Omple una signatura però mai s'usa |
| **Stub** | Retorna valors prefixats |
| **Spy** | Registra les crides que rep |
| **Mock** | Verifica que es cridà d'una manera concreta |
| **Fake** | Implementació simplificada que realment funciona |

---

## El problema: crear classes manualment és pesat

Imagina que tenim aquesta interfície:

```csharp
public interface IEmailSender
{
    void Send(string to, string subject, string body);
}
```

Sense cap llibreria, caldria crear una classe nova per a cada escenari:

```csharp
public class FakeEmailSender : IEmailSender
{
    public List<string> SentTo { get; } = new();

    public void Send(string to, string subject, string body)
        => SentTo.Add(to);
}
```

Per a cada interfície, per a cada escenari → **moltíssimes classes**.

---

## La solució: llibreria especialitzada


En el nostre cas farem servir **NSubstitute**, que és una llibreria especialitzada en dobles de prova per a .NET.

Permet crear substituts de qualsevol interfície **en una sola línia**, sense necessitat de definir cap classe nova.

```bash
dotnet add package NSubstitute
```

> 📦 NuGet: `NSubstitute`
> 🔗 https://nsubstitute.github.io

---

## Afegir NSubstitute al projecte de proves

```bash
dotnet add ./MeuProjecte.Tests/MeuProjecte.Tests.csproj package NSubstitute
```

I a la part superior del fitxer de test:

```csharp
using NSubstitute;
```

---

## Dummy amb NSubstitute

Un **Dummy** és un objecte que passem per complir una signatura però que mai s'usarà realment.

```csharp
[Fact]
public void CrearUsuari_SenseEnviarEmail_FuncioOk()
{
    // Arrange
    var emailSender = Substitute.For<IEmailSender>(); // dummy: no importa el que fa

    var servei = new ServeiUsuaris(emailSender);

    // Act
    var resultat = servei.CrearUsuari("anna@exemple.cat");

    // Assert
    Assert.True(resultat);
}
```

---

## Stub amb NSubstitute

Un **Stub** retorna valors predefinits perquè el nostre codi pugui avançar.

```csharp
public interface IRepoProductes
{
    Producte? ObtenirPer(int id);
}

[Fact]
public void ObtenirPreu_ProducteExistent_RetornaPreu()
{
    // Arrange
    var repo = Substitute.For<IRepoProductes>();
    repo.ObtenirPer(42).Returns(new Producte { Id = 42, Preu = 9.99m });

    var servei = new ServeiProductes(repo);

    // Act
    var preu = servei.ObtenirPreu(42);

    // Assert
    Assert.Equal(9.99m, preu);
}
```

---

## Spy amb NSubstitute

Un **Spy** registra les crides que ha rebut perquè puguem verificar-les després.

```csharp
[Fact]
public void ComprarProducte_ProducteExistent_EnviaEmail()
{
    // Arrange
    var emailSender = Substitute.For<IEmailSender>();
    var servei = new ServeiComandes(emailSender);

    // Act
    servei.Comprar("anna@exemple.cat", 42);

    // Assert — verifiquem que es va cridar Send amb el destinatari correcte
    emailSender.Received(1).Send("anna@exemple.cat", Arg.Any<string>(), Arg.Any<string>());
}
```

`Received(n)` és la forma que té NSubstitute de fer d'espia.

---

## Mock amb NSubstitute

Un **Mock** és un spy que també configura expectatives. Amb NSubstitute, `Received` ja cobreix aquest rol:

```csharp
[Fact]
public void ProcessarComanda_SempreNotificaAdmin()
{
    // Arrange
    var notificador = Substitute.For<INotificador>();
    var servei = new ServeiComandes(notificador);

    // Act
    servei.Processar(new Comanda { Id = 1 });

    // Assert — el mock verifica la interacció exacta
    notificador.Received(1).Notifica(
        Arg.Is<string>(s => s.Contains("admin")),
        Arg.Any<string>()
    );
}
```

---

## Fake amb NSubstitute

Un **Fake** té una implementació real simplificada. NSubstitute no és la millor eina aquí — normalment es crea a mà o es fa servir una BD en memòria.

```csharp
// Fake típic: implementació en memòria
public class FakeRepoUsuaris : IRepoUsuaris
{
    private readonly List<Usuari> _usuaris = new();

    public void Afegir(Usuari u) => _usuaris.Add(u);
    public Usuari? ObtenirPer(int id) => _usuaris.FirstOrDefault(u => u.Id == id);
}
```

> Els Fakes s'usen quan la lògica del col·laborador és prou complexa com per merèixer ser simulada de debò.

---

## Resum: quin tipus usar?

| Necessito... | Tipus | Amb NSubstitute? |
|---|---|---|
| Complir una signatura sense usar-la | Dummy | ✅ `Substitute.For<T>()` |
| Que retorni un valor fix | Stub | ✅ `.Returns(...)` |
| Verificar que es va cridar | Spy / Mock | ✅ `.Received(n)` |
| Una implementació real lleugera | Fake | ⚠️ Millor a mà |

---

## Beneficis de NSubstitute

- ✅ **Sense classes noves**: un `Substitute.For<T>()` substitueix qualsevol interfície
- ✅ **Tests més nets**: tot queda al fitxer de test, sense fitxers auxiliars
- ✅ **Expressiu**: la sintaxi `Received`, `Returns`, `Arg.Any` llegeix gairebé com prosa
- ✅ **Segur en refactor**: si canvia la interfície, el compilador detecta els substituts trencats

---

## Exemple complet

```csharp
public interface ILogger { void Log(string msg); }
public class Calculadora
{
    private readonly ILogger _logger;
    public Calculadora(ILogger logger) => _logger = logger;
    public int Suma(int a, int b)
    {
        _logger.Log($"Sumant {a} + {b}");
        return a + b;
    }
}

[Fact]
public void Suma_RegistraLaOperacio()
{
    var logger = Substitute.For<ILogger>();      // crea el doble
    var calc = new Calculadora(logger);

    var resultat = calc.Suma(3, 4);

    Assert.Equal(7, resultat);
    logger.Received(1).Log(Arg.Is<string>(s => s.Contains("3") && s.Contains("4")));
}
```
