# SOLID: SRP i OCP (Single Responsibility Principle + Open/Closed Principle)

## Objectiu de la sessió

Entendre els principis SRP i OCP dins de SOLID, detectar quan els estem trencant i aplicar-los amb exemples reals en C#.

---

## 1) Punt de partida: codi que sembla correcte, però es trenca fàcilment

Imaginem una classe que calcula imports, imprimeix factures i desa dades:

```csharp
public class InvoiceService
{
	public decimal CalculateTotal(List<decimal> lines)
	{
		return lines.Sum();
	}

	public void PrintInvoice(decimal total)
	{
		Console.WriteLine($"Total invoice: {total}");
	}

	public void SaveToDatabase(decimal total)
	{
		// Simulació de persistència
		Console.WriteLine($"Saving {total} to DB");
	}
}
```

- Aquesta classe té una única raó de canvi o en té diverses? (format impresió, càlcul, tipus de persistència)

Aquest tipus de codi ens serveix per entendre SRP i OCP en pràctica.

---

## 2) Context: SRP i OCP dins SOLID

Recordatori ràpid:

- S: **Single Responsibility Principle**
- O: **Open/Closed Principle**
- L: Liskov Substitution Principle
- I: Interface Segregation Principle
- D: Dependency Inversion Principle

En aquesta sessió ens centrem en els dos primers perquè són la base per reduir acoblament i fer canvis amb menys regressions.

---

## 3) SRP - Single Responsibility Principle

### Concepte

Una classe ha de tenir **una sola responsabilitat**. Dit d'una altra manera: ha de tenir **una sola raó de canvi**.

### Què vol dir "responsabilitat"?

No és "fer una sola línia" ni "ser petita". És encapsular una preocupació coherent del negoci o del sistema.

Exemples de responsabilitats diferents:

- regles de negoci
- persistència
- format de sortida
- comunicació amb serveis externs

Quan barregem aquestes preocupacions en una sola classe, qualsevol canvi impacta massa codi.

---

## 4) Exemple C# que trenca SRP

```csharp
public class UserManager
{
	public void Register(string email)
	{
		if (!email.Contains("@"))
			throw new ArgumentException("Email invàlid");

		// Persistència
		Console.WriteLine($"Insert user {email} in DB");

		// Infraestructura de notificació
		Console.WriteLine($"Send welcome email to {email}");
	}
}
```

Problemes:

- barreja validació, persistència i notificació
- si canvia el repositori, toquem la mateixa classe
- si canvia el sistema d'email, també
- test unitari més difícil perquè hi ha massa efectes col·laterals

---

## 5) Exemple C# aplicant SRP

```csharp
public class UserValidator
{
	public void EnsureValidEmail(string email)
	{
		if (string.IsNullOrWhiteSpace(email) || !email.Contains("@"))
			throw new ArgumentException("Email invàlid");
	}
}

public interface IUserRepository
{
	void Save(string email);
}

public class SqlUserRepository : IUserRepository
{
	public void Save(string email)
	{
		Console.WriteLine($"Insert user {email} in DB");
	}
}

public interface IWelcomeNotifier
{
	void Send(string email);
}

public class EmailWelcomeNotifier : IWelcomeNotifier
{
	public void Send(string email)
	{
		Console.WriteLine($"Send welcome email to {email}");
	}
}

public class UserRegistrationService(
	UserValidator validator,
	IUserRepository repository,
	IWelcomeNotifier notifier)
{
	public void Register(string email)
	{
		validator.EnsureValidEmail(email);
		repository.Save(email);
		notifier.Send(email);
	}
}
```

Ara cada peça té una responsabilitat clara.

---

## 6) OCP - Open/Closed Principle

### Definició curta

Els components han d'estar:

- **oberts a extensió**
- **tancats a modificació**

Idea pràctica: quan arriba un nou requisit, volem afegir comportament nou sense tocar massa codi existent que ja funciona.

### Evolució del concepte: de Meyer al polimorfisme

Segons Wikipedia, el terme OCP el va introduir Bertrand Meyer a *Object-Oriented Software Construction* (1988).

En la formulació original de Meyer, la idea clau era:

- un mòdul és *tancat* quan té una interfície estable i es pot reutilitzar
- un mòdul és *obert* quan el podem estendre, principalment via herència

Als anys 90, el principi es va reinterpretar en una versió més "polimòrfica" (popularitzada, entre d'altres, per Robert C. Martin, 1996):

- tanquem l'abstracció (contracte)
- obrim noves implementacions darrere aquesta abstracció
- canviem comportaments per substitució polimòrfica, no tocant la classe client

Per això, en pràctica moderna, OCP sovint es materialitza amb interfícies + injecció de dependències + estratègies/factories (com l'exemple de `IDiscountPolicy` d'aquest document), més que no pas amb herència de classes concretes.

---

## 7) Exemple C# que trenca OCP

```csharp
public class DiscountCalculator
{
	public decimal ApplyDiscount(string customerType, decimal amount)
	{
		if (customerType == "Regular")
			return amount;

		if (customerType == "Premium")
			return amount * 0.90m;

		if (customerType == "Vip")
			return amount * 0.80m;

		throw new ArgumentException("Tipus desconegut");
	}
}
```

Cada nou tipus de client obliga a modificar la mateixa classe, re-testar-ho tot i augmentar risc de regressió.

---

## 8) Exemple C# aplicant OCP amb factory

En el *Theatrical-Players-Refactoring-Kata*, Fowler fa servir una idea molt semblant: centralitzar la selecció del comportament en una factory i deixar la lògica de negoci treballar amb l'abstracció. La classe de negoci no ha de saber com es decideix quina política toca; només ha de saber que existeix una estratègia adequada per a cada tipus.

```csharp
public interface IDiscountPolicy
{
	bool CanApply(string customerType);
	decimal Apply(decimal amount);
}

public class RegularDiscountPolicy : IDiscountPolicy
{
	public bool CanApply(string customerType) => customerType == "Regular";
	public decimal Apply(decimal amount) => amount;
}

public class PremiumDiscountPolicy : IDiscountPolicy
{
	public bool CanApply(string customerType) => customerType == "Premium";
	public decimal Apply(decimal amount) => amount * 0.90m;
}

public class VipDiscountPolicy : IDiscountPolicy
{
	public bool CanApply(string customerType) => customerType == "Vip";
	public decimal Apply(decimal amount) => amount * 0.80m;
}

public interface IDiscountPolicyFactory
{
	IDiscountPolicy? GetFor(string customerType);
}

public class DiscountPolicyFactory(IEnumerable<IDiscountPolicy> policies) : IDiscountPolicyFactory
{
	public IDiscountPolicy? GetFor(string customerType)
	{
		var policy = policies.FirstOrDefault(p => p.CanApply(customerType));

		if (policy is null)
			throw new ArgumentException("Tipus desconegut");

		return policy;
	}
}

public class DiscountCalculator(IDiscountPolicyFactory policyFactory)
{
	public decimal ApplyDiscount(string customerType, decimal amount)
	{
		var policy = policyFactory.GetFor(customerType);
		return policy.Apply(amount);
	}
}
```

Per afegir un nou tipus (`Student`, per exemple), creem una classe nova que implementi `IDiscountPolicy` i la factory la podrà resoldre sense tocar la lògica de `DiscountCalculator`.

Aquí `IEnumerable<IDiscountPolicy>` representa que la factory rep per injecció totes les polítiques registrades. No crea la col·lecció a mà: el contenidor d'IoC li passa les implementacions disponibles i la factory selecciona la que toca. Podem veure a la documentació de [Service registration methods](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-10.0&utm_source=chatgpt.com#service-registration-methods) com l'injector de dependències suporta aquesta configuració. Hi ha llibreries com `Scrutor` que ens poden ajudar a injectar totes aquestes classes a la vegada:

```csharp
services.Scan(scan => scan
    .FromAssemblyOf<IDiscountPolicy>()
    .AddClasses(c => c.AssignableTo<IDiscountPolicy>())
    .As<IDiscountPolicy>()
    .WithTransientLifetime());
```

En aquest exemple hem afegit una factory que seria qui decidiria quin, de entre tots els descomptes, apliquem (per exemple el que més beneficia al client):

- la factory decideix quin comportament toca
- el calculator només aplica el comportament

Si la resolució de la política creix molt, aquesta factory és el lloc natural per encapsular-la.

Si vols una analogia encara més propera al kata de Fowler, pensa en això com la diferència entre:

- decidir quin càlcul toca segons el tipus
- executar el càlcul una vegada ja tens la peça correcta

Aquest és exactament el tipus de separació que fa el codi més fàcil d'estendre sense rebentar el que ja funciona.

---

## 9) Relació entre SRP i OCP

- SRP ens ajuda a separar responsabilitats perquè el codi sigui entenedor i canviable.
- OCP aprofita aquesta separació per poder afegir funcionalitat amb menys risc.

En molts casos, si SRP falla, OCP també es fa difícil d'aplicar.

---

## 10) Cal tenir en compte

1. SRP no vol dir "una classe de 20 línies". Vol dir una raó de canvi.
2. Quan trobem molts d'`if`/`switch` en lògica (descomptes, regles, formats, integracions) és un senyal que hem de valorar fer refactorització.
3. No cal crear jerarquies i interfícies innecessàries quan no hi ha cap variació real.
4. Trobar un punt d'equilibri: aplicar OCP abans que el codi ja estigui molt acoblat i, alhora, no fer sobreenginyeria.

---

## 11) Detectar quan cal refactoritzar

Abans de tancar una classe, pregunta't:

- Aquesta classe té més d'una raó de canvi?
- Si afegeixo una variant nova, he de modificar codi antic o només afegir-ne de nou?
- Estic barrejant negoci amb detalls d'infraestructura?

Si la resposta és "sí" sovint, probablement estàs trencant SRP o OCP.

---

## 12) Articles relacionats

- [SOLID (Wikipedia)](https://en.wikipedia.org/wiki/SOLID)
- [Open-closed principle (Wikipedia)](https://en.wikipedia.org/wiki/Open-closed_principle)
- [Single-responsibility principle (Wikipedia)](https://en.wikipedia.org/wiki/Single-responsibility_principle)
- [Object-Oriented Software Construction - Bertrand Meyer](https://en.wikipedia.org/wiki/Object-Oriented_Software_Construction)
- [Design Principles and Design Patterns (Robert C. Martin)](http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf) [arxiu](https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf)
