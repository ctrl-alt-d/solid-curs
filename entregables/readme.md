# Entrebles

Aquest curs consta de 5 hores de treball personal

## SDK Instal·lat

Enganxa el resultat de la comanda `dotnet --list-sdks` per comprovar que tens el dotnet instal·lat. Explica quin mètode d'instal·lació has fet servir.

## Crear solució

Fes una captura per mostrar que saps crear una solució dotnet amb testos. Cal que aparegui la solució, el projecte de test, una classlib i un projecte consola. Suposant que `dherrera` és el teu usuari xtec, el projecte de test s'ha de dir `dherrera.test` i el de consola `dherrera.UI` i la classlib `dherrera.lib`. Utilitza la comanda `tree -I 'bin|obj` a Linux o MacOs o la comanda `Get-ChildItem -Directory -Recurse | Where-Object { $_.Name -notin @('bin', 'obj') }` des de la PowerShell.

## Passar testos

Copia el resultat de `dotnet test` per comprovar que has pogut passar un test.

## attack-calculator-kata

Hem fet aquesta Kata segons demana l'enunciat original, usant la "technique called extract and override". I també l'hem fet usant IoC (Inversion of Code). Repassa mentalment les dues tècniques i explica quina de les dues t'ha agradat més i per què.

## Mocks i NSubstitute

En C# Mockejar una interface amb NSubstitute és més fàcil que mockejar una classe. Explica o posa un exemple on treballar amb interfaces (DIP) ens faciliti els testos unitaris.

