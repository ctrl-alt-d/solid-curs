### `AttackCalculator`

Per solucionar la Kata amb extract amb replace el que hem fet és crear un mètode `virtual` (que permet ser sobre escrita) que ens dona el resultat del dau:

```csharp
        protected virtual int LlençaDau()
        {
            return random.Next(1, 20);
        }
```

Per fer els testos sobreescrivim aquest mètode per tal de 'trucar el dau':

```csharp
namespace Game.tests;

public class AttackCalculatorTestable(int numero): AttackCalculator
{
    protected override int LlençaDau()
    {
        return numero;
    }
}
```

Als testos, l'únic que hem de fer és dir quin número volem que surti quan llencem el dau:

```csharp
    [Fact]
    public void TincProuForçaPeroElDauSurtUn1ElDanyEs0()
    {          
        // arrange
        var calc = new AttackCalculatorTestable(1);
        var atk = new Character(
            armorClass: NoIMPORTA,
            weaponDamage: NoIMPORTA,
            race: "Atacant",
            force: 5);
        var def = new Character(
            armorClass: 2,
            weaponDamage: NoIMPORTA,
            race: "Defensor",
            force: NoIMPORTA);

        // act
        var resultat = calc.CalculateDamage(atk, def);

        // assert
        Assert.Equal(0, resultat);
    }
```


Resolució amb extract and replace: [github.com/ctrl-alt-d/solid-attack-calculator-kata](https://github.com/ctrl-alt-d/solid-attack-calculator-kata)