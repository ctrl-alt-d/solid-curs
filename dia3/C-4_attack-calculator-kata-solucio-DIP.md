### Codi refactoritzat usant DIP (Inversió de dependències)

Creem la interface del dau i la seva implementació:

```c#
    public interface IDice
    {
        int Roll();
    }

    public class D20Dice : IDice
    {
        private readonly Random _random = new Random();

        public int Roll()
        {
            return _random.Next(1, 21);
        }
    }
```

Ara, `` rep per injecció (Inversió de Dependències) la interface del dau:


```c#
    public class AttackCalculator
    {
        private readonly IDice _dice;

        public AttackCalculator(IDice dice) // <--- Aquí
        {
            _dice = dice;
        }

        public int CalculateDamage(Character atk, Character def)
        {
            var dice = _dice.Roll();
            var attackRoll = atk.Force + dice;

            if (IsCriticalFailure(dice))
            {
                return 0;
            }

            if (!Hits(attackRoll, def.armorClass))
            {
                return 0;
            }

            if (IsCriticalHit(dice))
            {
                return atk.damageDealt * 2;
            }

            return atk.damageDealt;
        }

        private static bool Hits(int attackRoll, int armorClass)
        {
            return attackRoll > armorClass;
        }

        private static bool IsCriticalFailure(int dice)
        {
            return dice == 1;
        }

        private static bool IsCriticalHit(int dice)
        {
            return dice == 20;
        }
    }
}
```

Per fer els testos, creem un doble de test del dau (stub):

```csharp
namespace Game.tests.StubClasses;

public class DauTrucat(int numero) : IDice
{
    public int LlençaDau()
    {
        return numero;
    }
}
```

D'aquesta manera podem fer que el dau "caigui" sempre de la cara que ens interessa:

```csharp
    private static (AttackCalculator, Character, Character) ArrageObjects(int dau)
    {
        var dauTrucat = new DauTrucat(dau);
        var calc = new AttackCalculator(dauTrucat);

                var atk = new Character(
            armorClass: NoIMPORTA,
            weaponDamage: NoIMPORTA,
            race: "Atacant",
            force: 5);
        var def = new Character(
            armorClass: 100,
            weaponDamage: NoIMPORTA,
            race: "Defensor",
            force: NoIMPORTA);


        return (calc, atk, def);
    }
```