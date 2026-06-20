# SOLID: DIP (Dependency Inversion Principle)

## Objectiu de la sessió

Entendre el principi DIP dins de SOLID, veure quin problema resol en el dia a dia i preparar el terreny per a la sessió següent sobre IoC.

---

## 1) Punt de partida: codi fortament acoblat

Abans de parlar de SOLID, interfícies o contenidors, mira aquest exemple:

```csharp
public class OrderService
{
	private MySQLRepository repository = new MySQLRepository();

	public void Save(Order order)
	{
		repository.Save(order);
	}
}
```

Preguntes per reflexionar:

- Què passa si volem canviar MySQL per MongoDB?
- Com fem tests sense base de dades real?
- Qui decideix quina implementació utilitzar?

Aquí la dependència està "enganxada" al codi. Aquest és exactament el tipus de problema que DIP vol resoldre.

---

## 2) Context: on encaixa DIP dins SOLID

Recordatori ràpid de SOLID:

- S: Single Responsibility Principle
- O: Open/Closed Principle
- L: Liskov Substitution Principle
- I: Interface Segregation Principle
- D: **Dependency Inversion Principle**

El DIP tanca el bloc SOLID amb una idea clau: la direcció de les dependències importa tant com la lògica de negoci.

---

## 3) Definició formal de DIP

Definició clàssica (Robert C. Martin):

1. Els mòduls d'alt nivell no han de dependre de mòduls de baix nivell. Tots dos han de dependre d'abstraccions.
2. Les abstraccions no han de dependre dels detalls. Els detalls han de dependre de les abstraccions.

### Traducció pràctica

La teva lògica de negoci (alt nivell) no hauria de conèixer classes concretes d'infraestructura (baix nivell), com ara SMTP, SQL Server, Stripe, fitxers o APIs concretes. Alt nivell ha de saber el què (ex: notificar, persistir, ...) però no el com (ex: mail, sms, MySql, Mongo, ...)

---

## 4) Per què DIP és tan important?

Quan no apliquem DIP ens podem trobar amb aquestes problemàtiques:

- el codi queda fortament acoblat
- els canvis d'infraestructura trenquen negoci
- els tests són cars i fràgils
- costa reutilitzar la lògica en altres contextos

Quan sí apliquem DIP:

- el domini queda més net i estable
- podem canviar infraestructura sense tocar el core
- testing més fàcil amb dobles (mocks, fakes, stubs)
- millor mantenibilitat a mig i llarg termini

---

## 5) Exemple C# sense DIP (acoblament directe)

Nota: això és una simplificació per entendre el concepte.

```csharp
public class SqlUserRepository
{
	public string GetById(int id)
	{
		return $"User {id} from SQL";
	}
}

public class UserReportService
{
	private readonly SqlUserRepository _repository;

	public UserReportService()
	{
		// La capa de negoci crea directament infraestructura
		_repository = new SqlUserRepository();
	}

	public string BuildReport(int userId)
	{
		var user = _repository.GetById(userId);
		return $"Report for: {user}";
	}
}
```

Problemes:

- `UserReportService` depèn d'un detall concret (`SqlUserRepository`)
- si passes a Mongo o API externa, has de tocar la classe de negoci
- testejar aquesta classe és més complicat

---

## 6) Exemple C# aplicant DIP (dependència sobre abstracció)

Nota: això és una simplificació per entendre el concepte.

```csharp
public interface IUserRepository
{
	string GetById(int id);
}

public class SqlUserRepository : IUserRepository
{
	public string GetById(int id)
	{
		return $"User {id} from SQL";
	}
}

public class InMemoryUserRepository : IUserRepository
{
	public string GetById(int id)
	{
		return $"User {id} from memory";
	}
}

public class UserReportService
{
	private readonly IUserRepository _repository;

	public UserReportService(IUserRepository repository)
	{
		_repository = repository;
	}

	public string BuildReport(int userId)
	{
		var user = _repository.GetById(userId);
		return $"Report for: {user}";
	}
}
```

Ara el servei de negoci depèn d'un contracte (`IUserRepository`), no del detall concret.

---

## 7) Qui depèn de qui? (visió arquitectònica)

Sense DIP:

- Domini -> Infraestructura

Amb DIP:

- Domini -> Abstracció
- Infraestructura -> Abstracció

La direcció és més saludable perquè el domini és la part més estable i els detalls (DB, APIs, proveïdors) són la part que canvia sovint.

---

## 8) Relació entre DIP, IoC i DI

Veiem que podem relacionar DIP amb altres conceptes d'arquitectura:

- **DIP** diu la regla de disseny: depèn d'abstraccions.
- **IoC** (inversió del control) diu qui controla la creació/execució: el control es desplaça fora del component.
- **DI** (Injecció de Dependències) és la tècnica habitual per implementar IoC i facilitar DIP.

Resum curt:

> DIP és el "què" arquitectònic.
> IoC/DI són el "com" pragmàtic en codi.

---

## 9) Exemple C# amb injecció per constructor (pont cap IoC)

Nota: respàs, és una simplificació.

```csharp
public interface IPriceProvider
{
	decimal GetPrice(string sku);
}

public class ApiPriceProvider : IPriceProvider
{
	public decimal GetPrice(string sku)
	{
		return 99.95m;
	}
}

public class PricingService
{
	private readonly IPriceProvider _priceProvider;

	public PricingService(IPriceProvider priceProvider)
	{
		_priceProvider = priceProvider;
	}

	public decimal GetFinalPrice(string sku)
	{
		var basePrice = _priceProvider.GetPrice(sku);
		return basePrice * 1.21m; // IVA d'exemple
	}
}
```

Aquí ja estem aplicant DIP i, alhora, deixem la porta oberta a IoC/DI container.

---

## 10) Errors habituals amb DIP

1. Crear interfícies inútils "per tot" sense cap motiu de variació.
2. Posar abstraccions a la capa equivocada (depura la direcció de dependències).
3. Confondre DIP amb "afegir moltes classes".
4. Fer Service Locator i amagar dependències.

DIP no és burocràcia: és reduir acoblament on hi ha risc real de canvi.

---

## 11) Quan aplicar-ho amb més intensitat

Situacions on DIP dona molt valor:

- accessos a dades
- integracions externes (pagaments, email, APIs)
- unit tests del domini
- sistemes amb canvis freqüents de proveïdor o tecnologia

Pot ser overkill en scripts petits o prototips molt curts.

---

## 12) Articles relacionats

- [SOLID (Wikipedia)](https://en.wikipedia.org/wiki/SOLID)
- [Dependency inversion principle (Wikipedia)](https://en.wikipedia.org/wiki/Dependency_inversion_principle)
- [The Dependency Inversion Principle (Robert C. Martin)](https://www.objectmentor.com/resources/articles/dip.pdf)
- [Robert C. Martin - Design Principles and Design Patterns (paper)](http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf) [arxiu](https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf)
- [Dependency inversion (Microsoft)](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#dependency-inversion)

