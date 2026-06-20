# IoC (Inversió del Control)

## Objectiu de la sessió

Entendre què és la Inversió del Control (IoC), per què la necessitem en disseny de software i com la materialitzem habitualment a .NET amb Dependency Injection (DI).

---

## 1) Definició d'IoC

**Inversió del Control (IoC)** és un principi de disseny on una part del control del flux o de la creació de dependències deixa d'estar dins de la classe de negoci i passa a un mecanisme extern.

Dit simple: en lloc de dir "jo em construeixo les meves dependències", la classe diu "algú m'ha de donar el que necessito".

### Definició curta per recordar

> IoC = el control (de creació, configuració o execució) es desplaça fora del component.

---

## 2) Per què ens importa?

Sense IoC, les classes acostumen a quedar **fortament acoblades**.

Quan una classe crea directament les seves dependències:

- costa canviar implementacions
- costa fer tests (mocks, stubs, fakes)
- augmenta el cost de manteniment
- es viola la idea de "programar contra abstraccions"

Amb IoC:

- depenem d'interfícies
- el wiring es fa fora (composition root)
- el codi és més testejable i extensible

---

## 3) IoC vs DI

- **IoC** és el principi general.
- **DI (Dependency Injection)** és una tècnica concreta per aplicar IoC. (No confondre amb `DIP`, el principi d'inversió de dependències)

És a dir, DI és una forma habitual de fer IoC, però no l'única.

---

## 4) Exemple en C#: sense IoC (acoblament fort)

```csharp
public class EmailSender
{
	public void Send(string to, string body)
	{
		Console.WriteLine($"Sending email to {to}: {body}");
	}
}

public class NotificationService
{
	private readonly EmailSender _emailSender;

	public NotificationService()
	{
		// Dependència creada internament: alt acoblament
		_emailSender = new EmailSender();
	}

	public void Notify(string userEmail, string message)
	{
		_emailSender.Send(userEmail, message);
	}
}
```

Problema: `NotificationService` està lligat a `EmailSender` concret.

---

## 5) Exemple en C#: amb IoC via DI

```csharp
public interface IMessageSender
{
	void Send(string to, string body);
}

public class EmailSender : IMessageSender
{
	public void Send(string to, string body)
	{
		Console.WriteLine($"Sending email to {to}: {body}");
	}
}

public class SmsSender : IMessageSender
{
	public void Send(string to, string body)
	{
		Console.WriteLine($"Sending SMS to {to}: {body}");
	}
}

public class NotificationService
{
	private readonly IMessageSender _messageSender;

	// La dependència entra des de fora (constructor injection)
	public NotificationService(IMessageSender messageSender)
	{
		_messageSender = messageSender;
	}

	public void Notify(string userContact, string message)
	{
		_messageSender.Send(userContact, message);
	}
}
```

Ara `NotificationService` no sap si envia email, SMS o qualsevol altra cosa.
Només sap que existeix el contracte `IMessageSender`.

---

## 6) Dotnet i el ServiceCollection

En aplicacions .NET modernes, el wiring acostuma a fer-se mitjançant un `ServiceCollection`, en parlarem més endavant.

---

## 8) Relació amb SOLID

IoC està molt connectat amb:

- **DIP (Dependency Inversion Principle)**
- **OCP (Open/Closed Principle)**
- **SRP (Single Responsibility Principle)**

Quan apliques IoC, sovint acabes millorant automàticament aquests principis perquè separa responsabilitats i redueix dependències concretes.



## Articles relacionats

- [Inversion of control (Wikipedia)](https://en.wikipedia.org/wiki/Inversion_of_control)
- [Dependency injection (Wikipedia)](https://en.wikipedia.org/wiki/Dependency_injection)
- [Dependency injection in ASP.NET Core (Microsoft)](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-10.0)
- [Dependency injection in ASP.NET Core (Microsoft)](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-10.0)


