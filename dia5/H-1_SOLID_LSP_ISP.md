# SOLID: LSP i ISP (Liskov Substitution Principle + Interface Segregation Principle)

## Objectiu de la sessió

Entendre els principis LSP i ISP dins de SOLID, detectar dissenys que semblen correctes però trenquen el comportament esperat, i refactoritzar-los amb exemples pràctics en C#.

---

## 1) Punt de partida: no n'hi ha prou amb "tenir interfícies"

Moltes vegades pensem que, si fem servir herència o interfícies, ja estem dissenyant bé. Però no sempre és així.

Podem tenir:

- una classe filla que no es comporta com el client espera
- una interfície massa gran que obliga implementacions a simular mètodes que no necessiten

Això és exactament el que volen evitar:

- **LSP**: que una substitució no trenqui el comportament
- **ISP**: que els clients no depenguin de mètodes que no fan servir

---

## 2) Context: LSP i ISP dins SOLID

Recordatori ràpid:

- S: Single Responsibility Principle
- O: Open/Closed Principle
- L: **Liskov Substitution Principle**
- I: **Interface Segregation Principle**
- D: Dependency Inversion Principle

Si SRP ens ajuda a separar responsabilitats i OCP a créixer sense tocar massa codi, LSP i ISP ens ajuden a mantenir abstraccions sanes.

---

## 3) LSP - Liskov Substitution Principle

### Concepte

Una classe derivada ha de poder substituir la seva classe base sense trencar el programa.

Traducció pràctica:

- si el client espera un contracte
- qualsevol implementació d'aquell contracte ha de complir-lo
- no pot sorprendre el client amb errors inesperats o comportaments incompatibles

Barbara Liskov ho va formular el 1987. La idea no és només "compila perquè hereta", sinó "manté les mateixes garanties de comportament".

### Senyals que estem trencant LSP

- la classe derivada llença `NotSupportedException` en mètodes heretats
- el client ha de preguntar "quina implementació ets realment?"
- una subclasse imposa més restriccions que la classe base
- una subclasse canvia el significat d'un mètode

---

## 4) Exemple C# que trenca LSP

Exemple clàssic i molt pedagògic:

```csharp
public class Bird
{
	public virtual void Fly()
	{
		Console.WriteLine("Flying");
	}
}

public class Sparrow : Bird
{
}

public class Penguin : Bird
{
	public override void Fly()
	{
		throw new NotSupportedException("A penguin cannot fly");
	}
}
```

I ara un client:

```csharp
public class BirdTrainer
{
	public void MakeBirdFly(Bird bird)
	{
		bird.Fly();
	}
}
```

Problema:

- `BirdTrainer` assumeix que qualsevol `Bird` pot volar
- això és cert per `Sparrow`, però fals per `Penguin`
- per tant, `Penguin` no és substituïble per `Bird` en aquest contracte

Aquest és el punt important: el problema no és el pingüí, sinó l'abstracció mal dissenyada.

---

## 5) Exemple C# aplicant LSP

En lloc de posar el vol a la classe base, separem millor el comportament:

```csharp
public abstract class Bird
{
	public abstract void Move();
}

public interface IFlyingBird
{
	void Fly();
}

public class Sparrow : Bird, IFlyingBird
{
	public override void Move()
	{
		Console.WriteLine("Hopping");
	}

	public void Fly()
	{
		Console.WriteLine("Flying");
	}
}

public class Penguin : Bird
{
	public override void Move()
	{
		Console.WriteLine("Swimming or waddling");
	}
}

public class BirdTrainer
{
	public void TrainFlyingBird(IFlyingBird bird)
	{
		bird.Fly();
	}
}
```

Ara sí:

- `Bird` representa allò que totes les aus comparteixen
- `IFlyingBird` representa només les aus que volen
- el client depèn del contracte correcte

LSP no demana forçar jerarquies; demana dissenyar contractes coherents.

---

## 6) Què implica realment complir LSP?

No és només una qüestió de signatures. També afecta:

- **precondicions**: la subclasse no ha d'exigir més que la base
- **postcondicions**: la subclasse no ha de garantir menys que la base
- **invariants**: la subclasse ha de respectar les regles bàsiques del tipus

Exemple intuïtiu:

- si una interfície diu "desa aquest fitxer"
- una implementació que de vegades falla perquè és "només lectura" sense avisar al contracte
- probablement no és una bona substitució

Quan LSP falla, el codi client comença a omplir-se de condicionals, validacions especials o excepcions defensives.

---

## 7) ISP - Interface Segregation Principle

### Concepte

Els clients no haurien de dependre d'interfícies que no fan servir.

Traducció pràctica:

- millor diverses interfícies petites i coherents
- que no pas una interfície enorme amb mètodes per a tothom

Una interfície massa grossa crea acoblament innecessari i implementacions artificials.

### Senyals que estem trencant ISP

- una classe implementa mètodes buits
- apareixen `NotSupportedException` perquè "aquesta implementació no fa això"
- un canvi en un mètode afecta classes que ni tan sols el fan servir
- la interfície barreja responsabilitats diferents

---

## 8) Exemple C# que trenca ISP

```csharp
public interface IMultiFunctionDevice
{
	void Print(string document);
	void Scan(string document);
	void Fax(string document, string phoneNumber);
}

public class OfficePrinter : IMultiFunctionDevice
{
	public void Print(string document)
	{
		Console.WriteLine($"Printing: {document}");
	}

	public void Scan(string document)
	{
		Console.WriteLine($"Scanning: {document}");
	}

	public void Fax(string document, string phoneNumber)
	{
		Console.WriteLine($"Faxing {document} to {phoneNumber}");
	}
}

public class BasicPrinter : IMultiFunctionDevice
{
	public void Print(string document)
	{
		Console.WriteLine($"Printing: {document}");
	}

	public void Scan(string document)
	{
		throw new NotSupportedException();
	}

	public void Fax(string document, string phoneNumber)
	{
		throw new NotSupportedException();
	}
}
```

Problemes:

- `BasicPrinter` només sap imprimir
- tot i així la forcem a dependre d'escaneig i fax
- la interfície no representa una sola necessitat coherent

---

## 9) Exemple C# aplicant ISP

```csharp
public interface IPrinter
{
	void Print(string document);
}

public interface IScanner
{
	void Scan(string document);
}

public interface IFaxSender
{
	void Fax(string document, string phoneNumber);
}

public class BasicPrinter : IPrinter
{
	public void Print(string document)
	{
		Console.WriteLine($"Printing: {document}");
	}
}

public class OfficePrinter : IPrinter, IScanner, IFaxSender
{
	public void Print(string document)
	{
		Console.WriteLine($"Printing: {document}");
	}

	public void Scan(string document)
	{
		Console.WriteLine($"Scanning: {document}");
	}

	public void Fax(string document, string phoneNumber)
	{
		Console.WriteLine($"Faxing {document} to {phoneNumber}");
	}
}
```

Ara cada client pot dependre només del que realment necessita.

Exemple:

```csharp
public class PrintService(IPrinter printer)
{
	public void PrintInvoice(string document)
	{
		printer.Print(document);
	}
}
```

`PrintService` no necessita saber res d'escàners ni de faxos.

---

## 10) Relació entre LSP i ISP

Sovint aquests dos principis fallen junts.

Per exemple:

- una interfície massa grossa força implementacions artificials
- aquestes implementacions acaben llençant excepcions o fent coses rares
- això acaba trencant la substitució

És a dir:

- **ISP** ajuda a definir contractes petits i coherents
- **LSP** ajuda a garantir que les implementacions respecten aquests contractes

Un bon disseny d'interfícies facilita molt complir LSP.

---

## 11) Pistes pràctiques per detectar problemes

Pregunta't:

- aquesta subclasse compleix realment el que promet la base?
- algun client ha de defensar-se segons la implementació concreta?
- aquesta interfície té mètodes que alguns clients o classes no necessiten?
- estic fent servir `NotSupportedException` com a símptoma d'un contracte mal definit?

Si la resposta és sí, probablement tens una pista clara de LSP o ISP mal aplicat.

---

## 12) Resum final

- **LSP**: una implementació ha de poder substituir l'abstracció sense trencar el comportament esperat
- **ISP**: cap client ha de dependre de mètodes que no fa servir

Aplicats bé:

- redueixen sorpreses en temps d'execució
- simplifiquen els clients
- milloren testabilitat i mantenibilitat
- fan que les abstraccions siguin realment útils

En resum: no n'hi ha prou amb "tenir herència" o "tenir interfícies". Cal que el contracte sigui petit, coherent i respectat per totes les implementacions.
