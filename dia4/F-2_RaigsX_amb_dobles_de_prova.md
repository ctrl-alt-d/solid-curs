# Documentació de com fer la kata: Raigs X amb dobles de prova

## 1) Preparació

Clona el repositori i entra al projecte:

```bash
git clone https://github.com/ctrl-alt-d/solid-raigX
cd solid-raigX
```

Instal·la les dependències de testing al projecte de tests:

```bash
cd tests/ControladoraRaigX.Tests
dotnet add package NSubstitute
dotnet add package FluentAssertions
```

## 2) Estructura dels tests

En aquesta kata seguim el patró AAA:

- Arrange: preparem l'escenari (dobles, valors i precondicions)
- Act: executem la funcionalitat
- Assert: comprovem comportament (resultat, excepcions o col·laboracions)

Fem servir:

- xUnit per definir tests (`[Fact]`, `[Theory]`, `[InlineData]`)
- NSubstitute per crear dobles (`Substitute.For<IMaquina>()`)
- FluentAssertions per validar excepcions de forma llegible

## 3) Explicació test a test

### Test 1: SiTotEsCorrecteSAplicaLaRadiacio

Objectiu: validar el [camí feliç](https://en.wikipedia.org/wiki/Happy_path).

Guia:

- Arrange: crea un doble de `IMaquina` i fes que la comprovació de sistemes retorni `true`.
- Act: crida `AplicaRadiacio(...)` des de la controladora amb valors vàlids.
- Assert: verifica que el mètode de la màquina s'ha cridat exactament una vegada.

Fragment orientatiu (no complet):

```csharp
maquina.ComprovaTotsElsSistemesActius().Returns(true);
// ...
await maquina.Received(1).AplicaRadiació(...);
```

Què s'està comprovant:

- La màquina informa que està operativa (`Returns(true)`)
- La controladora executa l'acció
- El mètode de la màquina `AplicaRadiació(...)` es crida exactament 1 cop

Per què és important: aquest test valida la col·laboració entre `Controladora` i `IMaquina`, no la implementació interna de la màquina.

### Test 2: SiLaMaquinaNoEstaOkLaControladoraLLançaUnaExcepcioInoAplicaRadiació

Objectiu: validar la regla de seguretat quan la màquina no està en condicions.

Guia:

- Arrange: configura la màquina per retornar `false` a la comprovació de sistemes.
- Act: desa la crida a `AplicaRadiacio(...)` en una funció async (no l'executis directament en l'assert).
- Assert: comprova que es llança `MaquinaNoOkException`.

Fragment orientatiu (no complet):

```csharp
var action = async () => await controladora.AplicaRadiació(...);
await action.Should().ThrowAsync<MaquinaNoOkException>();
```

Què s'està comprovant:

- Si els sistemes no estan actius (`Returns(false)`), no es pot aplicar radiació
- La controladora llança `MaquinaNoOkException`

Recomanació: afegeix verificació de no-col·laboració per blindar més el comportament.

```csharp
await maquina.DidNotReceive().AplicaRadiació(...);
```

### Test 3: SiElsParametresNoSónValidsLLançaUnaExcepcio

Objectiu: validar entrada invàlida amb proves parametritzades.

Guia:

- Defineix un `[Theory]` amb diversos `[InlineData]` per cobrir combinacions invàlides.
- Mantingues la màquina en estat vàlid (`true`) per assegurar que l'error provingui només dels paràmetres.
- Reutilitza el patró `action` + `ThrowAsync<...>()`.

Fragment orientatiu (no complet):

```csharp
[InlineData(-1, 10)]
[InlineData(10, -1)]
// ...
await action.Should().ThrowAsync<ParametresNoValidsException>();
```

Què s'està comprovant:

- Qualsevol potència o durada negativa és invàlida
- La controladora protegeix el sistema llançant `ParametresNoValidsException`
- Amb `[Theory]` i `[InlineData]` evitem duplicar tres tests gairebé idèntics

### Test 4: NoEsPotSuperarElLlindarDeDurada

Objectiu: validar que la controladora imposa un màxim de durada.

Guia:

- Arrange: deixa la màquina en estat operatiu.
- Act: crida la controladora amb una entrada que et permeti observar el límit de durada.
- Assert: verifica que la durada enviada a la màquina compleix la regla del llindar (no cal comprovar un valor exacte).

Fragment orientatiu (no complet):

```csharp
await maquina
    .Received(1)
    .AplicaRadiació(Arg.Any<int>(), Arg.Is<int>(x => x <= 5000), Arg.Any<CancellationToken>());
```

Què s'està comprovant:

- Encara que li passis una combinació alta, la durada enviada a la màquina no supera el límit
- `Arg.Is<int>(x => x <= 5000)` valida la regla funcional sense acoblar-se a un valor exacte

### Test 5 (proposat): Les crides es fan en ordre

Objectiu: validar que abans d'aplicar radiació es comprova que tot està OK.

Guia:

- Arrange: crea la `Controladora` amb una màquina substituïda.
- Act: executa `AplicaRadiació(...)` amb valors vàlids.
- Assert: usa `Received.InOrder(...)` per exigir l'ordre:
    1. primer `ComprovaTotsElsSistemesActius()`
    2. després `AplicaRadiació(...)`

Fragment orientatiu (no complet):

```csharp
Received.InOrder(() => {
        maquina.ComprovaTotsElsSistemesActius();
        maquina.AplicaRadiació(Arg.Any<int>(), Arg.Any<int>(), Arg.Any<CancellationToken>());
});
```

Documentació oficial de NSubstitute (`Received.InOrder`):

- https://nsubstitute.github.io/help/received-in-order/

## 4) Esquema de dependències (ASCII)

```text
            depen de         implementa
[Controladora] ----> [IMaquina] <--- [Maquina]
```

Lectura ràpida:

- La `Controladora` coneix només `IMaquina` (abstracció).
- `Maquina` és la implementació concreta de `IMaquina`.

## 5) Bones pràctiques que es veuen aquí

- Un test, una responsabilitat
- Noms de test descriptius (escenari + resultat esperat)
- Separar clarament Arrange/Act/Assert
- Verificar excepcions amb FluentAssertions i col·laboracions amb NSubstitute
- Fer servir `[Theory]` quan només canvien les dades

## 6) Resum ràpid

Amb aquesta bateria estem cobrint:

- Camí feliç
- Regles de seguretat
- Validació de paràmetres
- Regles de llindar

No només comprovem resultats: comprovem comportaments i contractes entre objectes, que és justament el valor dels dobles de prova.

