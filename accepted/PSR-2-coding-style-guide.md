[Kezdőlap](../README.md)

# Kódolási stílus útmutató

Ez az útmutató kibővíti a [PSR-1] alapvető kódolási szabványt. Célja, hogy csökkentse
az értelmezési nehézségeket a különböző szerzőktől származó kódok olvasása közben.
Ennek érdekében a PHP kód formázására vonatkozó közös szabály-, és elvárásrendszert állít fel.
Az itt szereplő stilisztikai szabályok a munkacsoport [tagprojektjei](../personel.md)
mögött álló fejlesztői közösségekből származnak.

A csupa nagybetűvel szedett kiemelt kulcsszavak ebben a leírásban
az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

[PSR-0]: PSR-0.md
[PSR-1]: PSR-1-basic-coding-standard.md

## 1. Áttekintés

- A kódnak követnie KELL a [PSR-1] alapvető kódolási szabványt.

- A kódban 4 szóközt KELL használni a sorok behúzására, nem tabulátort.

- A sorhossz tekintetében NEM SZABAD merev korlátokat felállítani; a rugalmas korlátot
120 karakternél KELL meghúzni; a soroknak legfeljebb 80 karakter hosszúnak KELLENE lenniük.

- A `namespace` deklarációt üres sornak KELL követni, ahogy a `use` operátorok
blokkját is egy üres sorral KELL elválasztani a kód további részétől.

- Az osztálydeklarációk nyitó kapcsos zárójelét a következő sorban KELL elhelyezni,
az osztálytörzset befejező zárójelet pedig a törzs után következő sorban.

- A metódusok nyitó zárójelét szintén új sorba KELL írni, a befejező zárójelnek
pedig a törzs utáni sorba KELL kerülni.

- A láthatóságot minden metódusnál és tulajdonságnál be KELL állítani;
az `abstract` és `final` kulcsszónak a láthatóság előtt KELL szerepelnie; míg
a `static` kulcsszót a láthatósági deklaráció után KELL feltüntetni.

- A vezérlési szerkezetek kulcsszavai után egy szóközt KELL hagyni; viszont metódus-,
és függvényhívás után ezt TILOS.

- A vezérlési szerkezetek törzsét határoló kapcsos zárójel nyitó elemét ugyanabban
a sorban KELL elhelyezni, a záró elemnek pedig a törzs utáni sorba KELL kerülni.

- A vezérlési szerkezetnek átadandó paramétereket tartalmazó kerek zárójelek nyitó
eleme után és befejező eleme előtt NEM LEHET szóköz.

### 1.1. Példakód

Az alábbi példa a gyors áttekinthetőség érdekében néhány fontosabb szabályt foglal össze:

~~~php
<?php
namespace Vendor\Package;

use FooInterface;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class Foo extends Bar implements FooInterface
{
    public function sampleMethod($a, $b = null)
    {
        if ($a === $b) {
            bar();
        } elseif ($a > $b) {
            $foo->bar($arg1);
        } else {
            BazClass::bar($arg2, $arg3);
        }
    }

    final public static function bar()
    {
        // függvénytörzs
    }
}
~~~

## 2. Általános szabályok

### 2.1. Alapvető kódolási szabvány

A kódnak követnie KELL a [PSR-1] alapvető kódolási szabvány összes rendelkezését.

### 2.2. Forrásfájlok

Minden PHP fájlban Unix LF (soremelés, `\n`) karaktereket KELL használni a sorok lezárására.

Minden PHP fájlnak egy üres sorral KELL véget érnie.

A záró `?>` php-címkét el KELL hagyni a kizárólag PHP-kódot tartalmazó fájlokból.

### 2.3. Sorok

A sorhossz tekintetében NEM SZABAD merev korlátokat felállítani.

A rugalmas korlátot 120 karakternél KELL meghúzni; az automatizált
stílusellenőrzőknek ennek átlépésekor figyelmeztetést KELL dobniuk, hibát viszont NEM SZABAD.

Az egyes soroknak NEM KELLENE hosszabbnak lennie 80 karakternél; az ennél
hosszabbakat AJÁNLOTT felosztani 80 karakternél nem hosszabb sorokra.

A nem üres sorok végére TILOS szóközt tenni.

Az olvashatóság javítása és a kapcsolódó kódblokkok jelzése érdekében üres sorokat LEHET alkalmazni.

Soronként NEM LEHET egynél több utasítás.

### 2.4. Behúzás

A kódban 4 szóközt KELL használni a sorok behúzására, tabulátort TILOS.

> Fontos: a szóközök kizárólagos (tehát nem tabulátorokkal kevert!) használata,
> segít elkerülni a fájl összehasonlítással, javításokkal és egyebekkel kapcsolatos
> problémákat.

### 2.5. Kulcsszavak és a true/false/null értékek

A PHP [kulcsszavakat] kisbetűvel KELL írni.

A `true`, `false`, és `null` értékeket szintén kisbetűvel KELL írni.

[kulcsszavakat]: http://php.net/manual/en/reserved.keywords.php

## 3. Névtér deklarációk és a `use` operátor

A `namespace` deklarációt üres sornak KELL követni.

A `use` operátoroknak a névtér deklaráció után KELL következnie.

A `use` operátornak csak egyszer KELL szerepelnie deklarációnként.

A `use` operátorok blokkját egy üres sorral KELL elválasztani a kód további részétől.

Példa a fenti szabályok alkalmazásra:

~~~php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

// ... további PHP kód ...

~~~

## 4. Osztályok, tulajdonságok és metódusok

Az "osztály" kifejezés ebben a dokumentumban egyaránt vonatkozik osztályokra,
interfészekre vagy trait-ekre.

### 4.1. Az `extends` és az `implements` kulcsszavak

Az `extends` és az `implements` kulcsszavaknak az osztálynévvel egy sorban KELL lenniük.

Az osztálydeklarációk nyitó (kapcsos-)zárójelét külön sorban KELL elhelyezni
(a deklaráció után), az osztálytörzset befejező zárójelet pedig a törzs utáni sorban.

~~~php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements \ArrayAccess, \Countable
{
    // állandók, tulajdonságok, metódusok
}
~~~

A megvalósított interfészek listáját (`implements`) több sorba is szét LEHET tördelni,
ahol minden egyes sort egyszeres behúzással kell kezdeni. Ha ezt a megoldást válasszuk,
akkor a lista első elemének a következő sorba KELL kerülnie és soronként csak
egyetlen interfészt KELL feltüntetni, ahogy a következő példában is látható:

~~~php
<?php
namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
    // állandók, tulajdonságok, metódusok
}
~~~

### 4.2. Tulajdonságok

A láthatóságot minden objektumtulajdonsághoz meg KELL adni.

Az elavult, de kompatibilitási okokból még támogatott `var` kulcsszót TILOS használni
az objektumtulajdonságok deklarálásánál.

Utasításonként egynél több tulajdonságot TILOS deklarálni.

Az objektumtulajdonságok láthatósági szintjét (`private` vagy `protected`)
NEM AJÁNLOTT aláhúzás karakterrel jelezni a tulajdonságnév elején.

Az előadottak tükrében egy objektumtulajdonság-deklarációnak valahogy így kellene kinéznie:

~~~php
<?php

namespace Vendor\Package;

class ClassName
{
    public $foo = null;
}
~~~

### 4.3. Metódusok

A láthatóságot minden objektum metódushoz meg KELL adni.

A metódusok láthatósági szintjét (`private` vagy `protected`)
NEM AJÁNLOTT aláhúzás karakterrel jelezni a metódusnév elején.

A metódusnév után TILOS szóközt tenni. A metódus esetleges paramétereit tartalmazó
kerek zárójel nyitó eleme után és záró eleme előtt NEM LEHET szóköz. Az osztálydeklarációk
nyitó kapcsos zárójelét külön sorban KELL elhelyezni (a deklaráció után),
az osztálytörzset befejező kapcsos zárójelet pedig a törzs utáni sorban.

A fentiek alapján egy metódus deklarációnak a következőképpen kellene kinéznie.
Különösen figyeljünk oda a kerek, kapcsos és szögletes zárójelek, vesszők és
szóközök elhelyezésére:

~~~php
<?php
namespace Vendor\Package;

class ClassName
{
    public function fooBarBaz($arg1, &$arg2, $arg3 = [])
    {
        // függvénytörzs
    }
}
~~~

### 4.4. Metódus paraméterek

A metódusok paraméterlistájában az egyes paraméterek utáni vessző elé TILOS
szóközt tenni, a vessző után ellenben SZÜKSÉGES.

Az alapértelmezett értéket tartalmazó metódus paramétereknek a lista végén KELL elhelyezkedni.

~~~php
<?php
namespace Vendor\Package;

class ClassName
{
    public function foo($arg1, &$arg2, $arg3 = [])
    {
        // függvénytörzs
    }
}
~~~

A metódus paraméterlistáját több sorba is szét LEHET tördelni, ahol minden egyes sort
egyszeres behúzással kell kezdeni. Ha ezt a megoldást válasszuk, akkor a lista
első elemének a következő sorba KELL kerülnie és soronként csak egyetlen paramétert
KELL feltüntetni.

Ha a paraméterlistát külön sorokba tördeljük, akkor a listát lezáró kerek zárójelet
és a függvénytörzs kezdetét jelző nyitó kapcsos zárójelet külön sorba KELL írni,
szóközzel elválasztva, ahogy az alábbi példában is látható:

~~~php
<?php
namespace Vendor\Package;

class ClassName
{
    public function aVeryLongMethodName(
        ClassTypeHint $arg1,
        &$arg2,
        array $arg3 = []
    ) {
        // függvénytörzs
    }
}
~~~

### 4.5. `abstract`, `final`, és `static`

Ha szükség van rájuk, akkor az `abstract` és a `final` kulcsszavaknak meg KELL
előzniük a láthatósági beállítást a deklarációkban.

Ellenben a `static` kulcsszónak a láthatósági deklaráció után KELL állnia.

~~~php
<?php
namespace Vendor\Package;

abstract class ClassName
{
    protected static $foo;

    abstract protected function zim();

    final public static function bar()
    {
        // függvénytörzs
    }
}
~~~

### 4.6. Metódus-, és függvényhívás

Metódus-, vagy függvényhívás esetén a hívott metódus vagy függvény neve és a paraméterlista
kezdő zárójele közé - a 4.3. pontban megénekelt metódus-deklarációhoz hasonlóan - TILOS
szóközt tenni, ugyanúgy a paraméterlistát határoló kerek zárójel nyitó eleme után és
záró eleme előtt NEM LEHET szóköz. A meghívott metódus paraméterlistájában az egyes
paraméterek utáni vessző elé TILOS szóközt tenni, a vessző után ellenben SZÜKSÉGES.


~~~php
<?php
bar();
$foo->bar($arg1);
Foo::bar($arg2, $arg3);
~~~

A hívandó metódus vagy függvény paraméterlistáját több sorba is szét LEHET tördelni,
ahol minden egyes sort egyszeres behúzással kell kezdeni. Ha ezt a megoldást válasszuk,
akkor a lista első elemének a következő sorba KELL kerülnie és soronként csak egyetlen
argumentumot KELL feltüntetni.

~~~php
<?php
$foo->bar(
    $hosszuArgumentum,
    $hosszabbArgumentum,
    $iszonyuHosszuArgumentum
);
~~~

## 5. Vezérlési szerkezetek

A vezérlési szerkezetekre vonatkozó általános szabályok a következők:

- A vezérlési szerkezet kulcsszava után egy szóközt KELL tenni
- A vezérlési szerkezetnek átadandó paramétereket tartalmazó kerek zárójelek nyitó
eleme után és befejező eleme előtt NEM LEHET szóköz.
- A paraméterlistát lezáró kerek zárójelet és a függvénytörzs kezdetét jelző nyitó
kapcsos zárójelet szóközzel KELL elválasztani
- A vezérlési szerkezet törzsét egyszeres behúzással KELL tagolni
- A törzset lezáró kapcsos zárójelnek a törzset követő sorban KELL lennie

Minden egyes vezérlési szerkezet törzsét kapcsos zárójelekkel KELL határolni.
Ez egységesíti a szerkezetek megjelenését, új sorok törzshöz adásakor pedig
csökkenti a hibák felbukkanásának valószínűségét.

### 5.1. `if`, `elseif`, `else`

Egy `if` elágazásnak az alábbi példakódhoz hasonlóan kell kinéznie. Különösen
figyeljünk oda a kerek, kapcsos és szögletes zárójelek, vesszők és szóközök elhelyezésére;
valamint arra, hogy az `else` és `elseif` kulcsszavak ugyan abban a sorban legyenek, mint
 az őket megelőző kifejezés törzsét lezáró kapcsos zárójel.

~~~php
<?php
if ($expr1) {
    // if törzs
} elseif ($expr2) {
    // elseif törzs
} else {
    // else törzs;
}
~~~

Az `elseif` kulcsszót KELLENE használni az `else if` alak helyett, hogy minden vezérlő
kulcsszó egységesen egybe legyen írva.

### 5.2. A `switch` szerkezet és a `case` elágazás

A `switch` szerkezetnek az alábbi példakódhoz hasonlóan kell kinéznie. Különösen
figyeljünk oda a kerek és kapcsos zárójelek, szóközök elhelyezésére.
A `case` elágazást a `switch`-hez viszonyítva egy behúzással beljebb KELL kezdeni,
a `break` kulcsszót (vagy más lezáró kulcsszót) a `case` törzsével azonos szinten
 KELL elhelyezni. Amikor egy nem üres `case`-törzsben a továbblépés szándékos, akkor
 olyan megjegyzéseket KELL elhelyezni, mint az alábbi példában is szereplő
`// no break`.

~~~php
<?php
switch ($expr) {
    case 0:
        echo 'Első eset, megszakítással';
        break;
    case 1:
        echo 'Második eset, megszakítás nélküli továbblépéssel';
        // no break
    case 2:
    case 3:
    case 4:
        echo 'Harmadik eset, megszakítás helyett visszatéréssel';
        return;
    default:
        echo 'Alapértelmezett eset';
        break;
}
~~~

### 5.3. A `while` és a `do while` ciklusok

A `while` ciklusnak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek, szóközök megfelelő elhelyezésére.

~~~php
<?php
while ($expr) {
    // ciklusmag
}
~~~

A `do while` ciklus nem sokban különbözik a `while` ciklustól, így a rá vonatkozó
szabályok is hasonlóak. Figyeljünk oda itt is a kerek és kapcsos zárójelek és
szóközök megfelelő elhelyezésére.

~~~php
<?php
do {
    // ciklusmag;
} while ($expr);
~~~

### 5.4. A `for` ciklus

A `for` ciklusnak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php
for ($i = 0; $i < 10; $i++) {
    // ciklusmag
}
~~~

### 5.5. `foreach`

A `foreach` ciklusnak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php
foreach ($iterable as $key => $value) {
    // utasítások
}
~~~

### 5.6. `try`, `catch`

A `try catch` blokknak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php
try {
    // try törzs
} catch (FirstExceptionType $e) {
    // catch törzs
} catch (OtherExceptionType $e) {
    // catch törzs
}
~~~

## 6. Closure-ök (névtelen függvények)

Closures MUST be declared with a space after the `function` keyword, and a
space before and after the `use` keyword.

The opening brace MUST go on the same line, and the closing brace MUST go on
the next line following the body.

There MUST NOT be a space after the opening parenthesis of the argument list
or variable list, and there MUST NOT be a space before the closing parenthesis
of the argument list or variable list.

In the argument list and variable list, there MUST NOT be a space before each
comma, and there MUST be one space after each comma.

Closure arguments with default values MUST go at the end of the argument
list.

A closure declaration looks like the following. Note the placement of
parentheses, commas, spaces, and braces:

~~~php
<?php
$closureWithArgs = function ($arg1, $arg2) {
    // törzs
};

$closureWithArgsAndVars = function ($arg1, $arg2) use ($var1, $var2) {
    // törzs
};
~~~

Argument lists and variable lists MAY be split across multiple lines, where
each subsequent line is indented once. When doing so, the first item in the
list MUST be on the next line, and there MUST be only one argument or variable
per line.

When the ending list (whether of arguments or variables) is split across
multiple lines, the closing parenthesis and opening brace MUST be placed
together on their own line with one space between them.

The following are examples of closures with and without argument lists and
variable lists split across multiple lines.

~~~php
<?php
$longArgs_noVars = function (
    $hosszuArgumentum,
    $hosszabbArgumentum,
    $iszonyuHosszuArgumentum
) {
    // törzs
};

$noArgs_longVars = function () use (
    $hosszuValtozo,
    $hosszabbValtozo,
    $borzasztoanHosszuValtozo
) {
    // törzs
};

$longArgs_longVars = function (
    $hosszuArgumentum,
    $hosszabbArgumentum,
    $iszonyuHosszuArgumentum
) use (
    $hosszuValtozo,
    $hosszabbValtozo,
    $borzasztoanHosszuValtozo
) {
    // törzs
};

$longArgs_shortVars = function (
    $hosszuArgumentum,
    $hosszabbArgumentum,
    $iszonyuHosszuArgumentum
) use ($var1) {
    // törzs
};

$shortArgs_longVars = function ($arg) use (
    $hosszuValtozo,
    $hosszabbValtozo,
    $borzasztoanHosszuValtozo
) {
    // törzs
};
~~~

Note that the formatting rules also apply when the closure is used directly
in a function or method call as an argument.

~~~php
<?php
$foo->bar(
    $arg1,
    function ($arg2) use ($var1) {
        // törzs
    },
    $arg3
);
~~~

## 7. Összegzés

There are many elements of style and practice intentionally omitted by this
guide. These include but are not limited to:

- Declaration of global variables and global constants

- Declaration of functions

- Operators and assignment

- Inter-line alignment

- Comments and documentation blocks

- Class name prefixes and suffixes

- Best practices

Future recommendations MAY revise and extend this guide to address those or
other elements of style and practice.

[Kezdőlap](../README.md)
