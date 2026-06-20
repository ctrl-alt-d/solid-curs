
# SOLID: introducció i context històric

Aquest document és una **introducció general**. Més endavant, treballarem cada principi per separat amb exemples i exercicis.

---

## 1) Què és SOLID

SOLID és un acrònim de cinc principis de disseny orientat a objectes pensats per reduir acoblament, facilitar manteniment i fer el codi més adaptable al canvi.

- S: Single Responsibility Principle (SRP)
- O: Open/Closed Principle (OCP)
- L: Liskov Substitution Principle (LSP)
- I: Interface Segregation Principle (ISP)
- D: Dependency Inversion Principle (DIP)

---

## 2) D'on ve SOLID?

### Context de l'època (finals 80 - 90 - inicis 2000)

Durant l'expansió de l'OOP (especialment amb C++ i després Java/C#), molts equips van detectar un patró repetit:

- dissenys rígids
- classes massa grans
- jerarquies d'herència fràgils
- dificultat per testejar
- por a refactoritzar

És a dir, el software funcionava al principi, però es degradava amb el temps (el que Robert C. Martin anomena *software rot*).

### Software rot

El **software rot** (degradació del programari) descriu el procés pel qual una base de codi es deteriora gradualment amb el temps, encara que continuï funcionant correctament.

No es tracta d'un error concret, sinó de l'acumulació de petites decisions que fan que el sistema sigui cada vegada més difícil d'entendre, modificar i mantenir.

#### Símptomes habituals

- Funcions cada cop més llargues.
- Noms de variables i mètodes poc descriptius.
- Codi duplicat.
- Comentaris desactualitzats.
- Dependències complexes entre mòduls.
- Correccions ràpides (*quick fixes*) que es converteixen en permanents.
- Por a modificar una part del sistema perquè podria trencar-ne una altra.

#### Relació amb el deute tècnic

El software rot està relacionat amb el *technical debt*, però no són exactament el mateix concepte:

- **Technical debt**: decisions conscients que prioritzen la velocitat sobre la qualitat.
- **Software rot**: degradació acumulada que apareix amb el temps, sigui conscient o no.

#### La Boy Scout Rule

Segons Robert C. Martin (*Uncle Bob*), una de les millors maneres d'evitar la degradació del codi és aplicar la *Boy Scout Rule*:

> Leave the campground cleaner than you found it.

En termes pràctics:

- Renomenar una variable confusa.
- Eliminar codi duplicat.
- Simplificar una condició complexa.
- Afegir un test que falta.

Les millores petites i constants acostumen a ser més efectives que les grans refactoritzacions ocasionals.

#### Exemple

Codi inicial:

```csharp
public decimal CalculateTotal(Order order)
{
    return order.Items.Sum(item => item.Price);
}
```

Després de mesos de canvis i correccions:

```csharp
public decimal CalculateTotal(Order order)
{
    decimal total = 0;

    foreach (var item in order.Items)
    {
        total += item.Price;

        if (item.Type == 3)
        {
            total *= 0.95m;
        }

        if (order.Customer.Country == "ES")
        {
            // TODO: Revisar IVA
            total *= 1.21m;
        }

        // Fix incidència #1847
        if (item.Price > 1000)
        {
            total -= 10;
        }
    }

    return total;
}
```

Tot i que el codi continua funcionant, ara és més difícil d'entendre, provar i modificar.

#### Com detectar-lo

Una pregunta útil és:

> Aquest codi és més fàcil o més difícil de modificar que fa sis mesos?

Si cada canvi requereix més temps, més proves manuals i genera més incertesa, és probable que la base de codi estigui patint software rot.

#### Conseqüències

Quan el software rot s'acumula:

- La velocitat de desenvolupament disminueix.
- Els errors són més freqüents.
- Les refactoritzacions es tornen més costoses.
- L'onboarding de nous desenvolupadors és més lent.
- Cada funcionalitat nova requereix més esforç que l'anterior.

Per aquest motiu, mantenir el codi net no és una activitat puntual, sinó una responsabilitat contínua de l'equip de desenvolupament.

### Qui va inventar els conceptes?

Important: **SOLID no neix d'un sol dia ni d'una sola persona**. L'acrònim agrupa idees que venen de diverses fonts.

- **OCP**: formulat per Bertrand Meyer a *Object-Oriented Software Construction* (1988).
- **LSP**: formulat per Barbara Liskov (amb formalització clàssica amb Jeannette Wing, 1994, *A Behavioral Notion of Subtyping*).
- **SRP, ISP i DIP**: desenvolupats i popularitzats per Robert C. Martin als anys 90 (articles d'Object Mentor / C++ Report; després en llibres).

### Qui va encunyar l'acrònim SOLID?

Les cinc idees van ser reunides i difoses per Robert C. Martin; l'acrònim **SOLID** s'atribueix habitualment a **Michael Feathers** (cap al 2004).

---

## 3) On es van publicar i consolidar?

Fites habitualment citades:

- 1988: Bertrand Meyer, *Object-Oriented Software Construction* (OCP).
- 1994: Liskov & Wing, *A Behavioral Notion of Subtyping* (LSP).
- anys 90: articles SRP/ISP/DIP de Robert C. Martin (Object Mentor, C++ Report, després arxivats).
- 2000: Robert C. Martin, *Design Principles and Design Patterns* (paper on ja apareix el paquet de principis en clau anti-*software rot*).
- 2003: Robert C. Martin, *Agile Software Development, Principles, Patterns, and Practices* (consolidació didàctica).

---

## 4) Quins problemes intenta evitar SOLID?

SOLID és una resposta pràctica a problemes estructurals molt comuns:

1. **Rigidesa**: petits canvis impliquen tocar massa codi.
2. **Fragilitat**: arregles una cosa i se'n trenquen altres.
3. **Immobilitat**: costa reutilitzar mòduls en altres contextos.
4. **Viscositat**: és més fàcil "fer un pedaç" que fer el disseny correcte.
5. **Complexitat accidental**: creix el soroll de codi no essencial.

En resum, SOLID tracta d'evitar que el cost de canvi exploti amb el temps.

---

## 5) SOLID està superat avui?

Resposta curta: **no està superat, però tampoc és suficient per si sol**.

### El que continua vigent

- Encara és una molt bona base per dissenyar codi mantenible.
- Encara redueix acoblament i millora testabilitat.
- Encara ajuda molt en codi orientat a domini.

### El que s'ha afegit amb els anys

Avui el disseny també incorpora:

- DDD (Domain-Driven Design)
- Arquitectura Hexagonal / Ports & Adapters
- Clean Architecture
- CQRS/Event-Driven en certs contextos
- enfocaments funcionals i data-oriented segons problema

La lectura moderna és: **SOLID és una base local de disseny de codi**, mentre que aquests altres enfocaments amplien la visió a nivell d'arquitectura, límits de context i operació del sistema.

---

## 6) Relació amb altres conceptes d'arquitectura

SOLID connecta de forma natural amb:

- **Separation of Concerns**: separar responsabilitats i decisions de canvi.
- **Low Coupling / High Cohesion**: menys dependències entre mòduls, més coherència interna.
- **Encapsulació**: ocultar detalls d'implementació.
- **Inversió de dependències i IoC/DI**: especialment en el principi DIP.
- **Boundary design** (Ports/Adapters): el domini depèn de contractes, no de tecnologia.

Per això SOLID encaixa tan bé en arquitectures modernes en capes o hexagonals.

---

## 7) Precaució: mals usos habituals

Aplicar SOLID "de manera mecànica" també crea problemes:

- massa abstraccions innecessàries
- interfícies prematures
- dissenys hiperfragmentats
- sobreenginyeria en projectes simples

Principi pràctic: aplica SOLID **allà on hi ha pressió real de canvi**, no com a ritual.

---

## Articles i fonts recomanades

- [SOLID (Wikipedia)](https://en.wikipedia.org/wiki/SOLID)
- [Dependency inversion (Microsoft Learn)](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#dependency-inversion)
- [Robert C. Martin - Design Principles and Design Patterns (paper)](http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf) [arxiu](https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf)
- [Open-Closed Principle (Wikipedia)](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)
- [Liskov Substitution Principle (Wikipedia)](https://en.wikipedia.org/wiki/Liskov_substitution_principle)
- [A Behavioral Notion of Subtyping (Liskov/Wing, 1994 - DOI)](https://dl.acm.org/doi/10.1145/197320.197383)
- [Object-Oriented Software Construction (Bertrand Meyer)](https://en.wikipedia.org/wiki/Object-Oriented_Software_Construction)