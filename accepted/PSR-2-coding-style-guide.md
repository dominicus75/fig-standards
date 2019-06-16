# Kódolási stílus útmutató

Ez az útmutató kibővíti a [PSR-1] alapvető kódolási szabványt. Célja, hogy csökkentse az értelmezési nehézségeket a különböző szerzőktől származó kódok olvasása közben. Ennek érdekében a PHP kód formázására vonatkozó közös szabály-, és elvárásrendszert állít fel. Az itt szereplő stilisztikai szabályok a munkacsoport [tagprojektjei](../personel.md) mögött álló fejlesztői közösségekből származnak.

A csupa nagybetűvel szedett kiemelt kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

[PSR-0]: PSR-0.md
[PSR-1]: PSR-1-basic-coding-standard.md

## 1. Áttekintés

- A kódnak követnie KELL a [PSR-1] alapvető kódolási szabványt.

- A kódban 4 szóközt KELL használni a sorok behúzására, nem tabulátort.

- A sorhossz tekintetében NEM SZABAD merev korlátokat felállítani; a rugalmas korlátot 120
  karakternél KELL meghúzni; a soroknak legfeljebb 80 karakter hosszúnak KELLENE lenniük.

- A `namespace` deklarációt üres sornak KELL követni, ahogy a `use` deklarációk blokkját is egy üres sorral KELL elválasztani a kód további részétől.

- Az osztálydeklarációk nyitó (kapcsos-)zárójelét a következő sorban KELL elhelyezni, az osztálytörzset befejező zárójelet pedig a törzs után következő sorban.

- A metódusok nyitó zárójelét szintén új sorba KELL írni, a befejező zárójelnek pedig a törzs utáni sorba KELL kerülni.

- A láthatóságot minden metódusnál és tulajdonságnál be KELL állítani; az `abstract` és `final` kulcsszónak a láthatóság előtt KELL szerepelnie; míg a `static` kulcsszót a láthatósági deklaráció után KELL feltüntetni.

- A vezérlési szerkezetek kulcsszavai után egy szóközt KELL hagyni; viszont metódus-, és függvényhívás után ezt TILOS.

- A vezérlési szerkezetek nyitó zárójelét ugyanabban a sorban KELL elhelyezni, a befejező zárójelnek pedig a törzs utáni sorba KELL kerülni.

- A vezérlési szerkezetek nyitó zárójele után és befejező zárójele előtt NEM LEHET szóköz.

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
        // method body
    }
}
~~~

## 2. Általános szabályok

### 2.1. Alapvető kódolási szabvány

A kódnak követnie KELL a [PSR-1] alapvető kódolási szabvány összes rendelkezését.

### 2.2. Forrásfájlok

Minden PHP fájlban Unix LF (soremelés) karaktereket KELL használni a sorok lezárására.

Minden PHP fájlnak egy üres sorral KELL véget érnie.

A záról `?>` php-címkét el KELL hagyni a kizárólag PHP-kódot tartalmazó fájlokból.

### 2.3. Sorok

A sorhossz tekintetében NEM SZABAD merev korlátokat felállítani.

A rugalmas korlátot 120 karakternél KELL meghúzni; az automatizált stílusellenőrzőknek ennek átlépésekor figyelmeztetést KELL dobniuk, hibát viszont NEM SZABAD.

Az egyes soroknak NEM KELLENE hosszabbnak lennie 80 karakternél; az ennél hosszabbakat AJÁNLOTT felosztani 80 karakternél nem hosszabb sorokra.

A nem üres sorok végére TILOS szóközt tenni.

Az olvashatóság javítása és a kapcsolódó kódblokkok jelzése érdekében üres sorokat LEHET alkalmazni.

Soronként NEM LEHET egynél több utasítás.

### 2.4. Behúzás

A kódban 4 szóközt KELL használni a sorok behúzására, tabulátort TILOS.

> Fontos: a szóközök kizárólagos (tehát nem tabulátorokkal kevert!) használata,
> segít elkerülni a fájl összehasonlítással, javításokkal és egyebekkel kapcsolatos
> problémákat.

### 2.5. Kulcsszavak és a True/False/Null értékek

PHP [keywords] MUST be in lower case.

The PHP constants `true`, `false`, and `null` MUST be in lower case.

[keywords]: http://php.net/manual/en/reserved.keywords.php

## 3. Névtér és `use` deklarációk

When present, there MUST be one blank line after the `namespace` declaration.

When present, all `use` declarations MUST go after the `namespace`
declaration.

There MUST be one `use` keyword per declaration.

There MUST be one blank line after the `use` block.

For example:

~~~php
<?php
namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

// ... additional PHP code ...

~~~

## 4. Osztályok, tulajdonságok és metódusok

The term "class" refers to all classes, interfaces, and traits.

### 4.1. Kiterjeszt és megvalósít kulcsszavak

The `extends` and `implements` keywords MUST be declared on the same line as
the class name.

The opening brace for the class MUST go on its own line; the closing brace
for the class MUST go on the next line after the body.

~~~php
<?php
namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements \ArrayAccess, \Countable
{
    // constants, properties, methods
}
~~~

Lists of `implements` MAY be split across multiple lines, where each
subsequent line is indented once. When doing so, the first item in the list
MUST be on the next line, and there MUST be only one interface per line.

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
    // constants, properties, methods
}
~~~

### 4.2. Tulajdonságok

Visibility MUST be declared on all properties.

The `var` keyword MUST NOT be used to declare a property.

There MUST NOT be more than one property declared per statement.

Property names SHOULD NOT be prefixed with a single underscore to indicate
protected or private visibility.

A property declaration looks like the following.

~~~php
<?php
namespace Vendor\Package;

class ClassName
{
    public $foo = null;
}
~~~

### 4.3. Metódusok

Visibility MUST be declared on all methods.

Method names SHOULD NOT be prefixed with a single underscore to indicate
protected or private visibility.

Method names MUST NOT be declared with a space after the method name. The
opening brace MUST go on its own line, and the closing brace MUST go on the
next line following the body. There MUST NOT be a space after the opening
parenthesis, and there MUST NOT be a space before the closing parenthesis.

A method declaration looks like the following. Note the placement of
parentheses, commas, spaces, and braces:

~~~php
<?php
namespace Vendor\Package;

class ClassName
{
    public function fooBarBaz($arg1, &$arg2, $arg3 = [])
    {
        // method body
    }
}
~~~

### 4.4. Metódus paraméterek

In the argument list, there MUST NOT be a space before each comma, and there
MUST be one space after each comma.

Method arguments with default values MUST go at the end of the argument
list.

~~~php
<?php
namespace Vendor\Package;

class ClassName
{
    public function foo($arg1, &$arg2, $arg3 = [])
    {
        // method body
    }
}
~~~

Argument lists MAY be split across multiple lines, where each subsequent line
is indented once. When doing so, the first item in the list MUST be on the
next line, and there MUST be only one argument per line.

When the argument list is split across multiple lines, the closing parenthesis
and opening brace MUST be placed together on their own line with one space
between them.

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
        // method body
    }
}
~~~

### 4.5. `abstract`, `final`, és `static`

When present, the `abstract` and `final` declarations MUST precede the
visibility declaration.

When present, the `static` declaration MUST come after the visibility
declaration.

~~~php
<?php
namespace Vendor\Package;

abstract class ClassName
{
    protected static $foo;

    abstract protected function zim();

    final public static function bar()
    {
        // method body
    }
}
~~~

### 4.6. Metódus-, és függvényhívás

When making a method or function call, there MUST NOT be a space between the
method or function name and the opening parenthesis, there MUST NOT be a space
after the opening parenthesis, and there MUST NOT be a space before the
closing parenthesis. In the argument list, there MUST NOT be a space before
each comma, and there MUST be one space after each comma.

~~~php
<?php
bar();
$foo->bar($arg1);
Foo::bar($arg2, $arg3);
~~~

Argument lists MAY be split across multiple lines, where each subsequent line
is indented once. When doing so, the first item in the list MUST be on the
next line, and there MUST be only one argument per line.

~~~php
<?php
$foo->bar(
    $longArgument,
    $longerArgument,
    $muchLongerArgument
);
~~~

## 5. Vezérlési szerkezetek

The general style rules for control structures are as follows:

- There MUST be one space after the control structure keyword
- There MUST NOT be a space after the opening parenthesis
- There MUST NOT be a space before the closing parenthesis
- There MUST be one space between the closing parenthesis and the opening
  brace
- The structure body MUST be indented once
- The closing brace MUST be on the next line after the body

The body of each structure MUST be enclosed by braces. This standardizes how
the structures look, and reduces the likelihood of introducing errors as new
lines get added to the body.

### 5.1. `if`, `elseif`, `else`

An `if` structure looks like the following. Note the placement of parentheses,
spaces, and braces; and that `else` and `elseif` are on the same line as the
closing brace from the earlier body.

~~~php
<?php
if ($expr1) {
    // if body
} elseif ($expr2) {
    // elseif body
} else {
    // else body;
}
~~~

The keyword `elseif` SHOULD be used instead of `else if` so that all control
keywords look like single words.

### 5.2. `switch`, `case`

A `switch` structure looks like the following. Note the placement of
parentheses, spaces, and braces. The `case` statement MUST be indented once
from `switch`, and the `break` keyword (or other terminating keyword) MUST be
indented at the same level as the `case` body. There MUST be a comment such as
`// no break` when fall-through is intentional in a non-empty `case` body.

~~~php
<?php
switch ($expr) {
    case 0:
        echo 'First case, with a break';
        break;
    case 1:
        echo 'Second case, which falls through';
        // no break
    case 2:
    case 3:
    case 4:
        echo 'Third case, return instead of break';
        return;
    default:
        echo 'Default case';
        break;
}
~~~

### 5.3. `while`, `do while`

A `while` statement looks like the following. Note the placement of
parentheses, spaces, and braces.

~~~php
<?php
while ($expr) {
    // structure body
}
~~~

Similarly, a `do while` statement looks like the following. Note the placement
of parentheses, spaces, and braces.

~~~php
<?php
do {
    // structure body;
} while ($expr);
~~~

### 5.4. `for`

A `for` statement looks like the following. Note the placement of parentheses,
spaces, and braces.

~~~php
<?php
for ($i = 0; $i < 10; $i++) {
    // for body
}
~~~

### 5.5. `foreach`

A `foreach` statement looks like the following. Note the placement of
parentheses, spaces, and braces.

~~~php
<?php
foreach ($iterable as $key => $value) {
    // foreach body
}
~~~

### 5.6. `try`, `catch`

A `try catch` block looks like the following. Note the placement of
parentheses, spaces, and braces.

~~~php
<?php
try {
    // try body
} catch (FirstExceptionType $e) {
    // catch body
} catch (OtherExceptionType $e) {
    // catch body
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
    // body
};

$closureWithArgsAndVars = function ($arg1, $arg2) use ($var1, $var2) {
    // body
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
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) {
    // body
};

$noArgs_longVars = function () use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
    // body
};

$longArgs_longVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
    // body
};

$longArgs_shortVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use ($var1) {
    // body
};

$shortArgs_longVars = function ($arg) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
    // body
};
~~~

Note that the formatting rules also apply when the closure is used directly
in a function or method call as an argument.

~~~php
<?php
$foo->bar(
    $arg1,
    function ($arg2) use ($var1) {
        // body
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
