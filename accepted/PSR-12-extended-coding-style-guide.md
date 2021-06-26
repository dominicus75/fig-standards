[Kezdőlap](../README.md)

# Bővített kódstílus útmutató

A csupa nagybetűvel szedett kiemelt kulcsszavak ebben a leírásban az [RFC 2119]
szerint értelmezendők.

## Áttekintés

Ez a specifikáció kibővíti és leváltja a [PSR-2] kódstílus útmutatót, de ugyanúgy
igényli a [PSR-1] alapvető kódolási szabvány betartását. A [PSR-2] útmutatóhoz
hasonlóan az a célja, hogy csökkentse az értelmezési nehézségeket a különböző
szerzőktől származó kódok olvasása közben. Ennek érdekében a PHP kód formázására
vonatkozó közös szabály-, és elvárásrendszert állít fel.

E PSR egy meghatározott módszert próbál biztosítani, amit a kódstílus-eszközök
implementálhatnak, a projektek pedig deklarálhatják annak betartását, hogy a
fejlesztők könnyen össze tudják kapcsolni a különböző projekteket. Amikor különböző
szerzők többféle projektben működnek együtt, ez elősegítheti, hogy legyen egy
egységesen minden projektben használható útmutató. Ezért ennek az útmutatónak a
haszna nem magukban a szabályokban, hanem azok megosztásában rejlik.

A [PSR-2] ajánlást 2012-ben fogadták el és azóta számos változtatást eszközöltek
a PHP-ban, amelyek kihatással lehetnek a kódstílus útmutatókra. Míg a [PSR-2]
nagyon alaposan leírja az írás idején létező PHP funkcionalitást, az új funkciók
értelmezését nyitva hagyta. Ez a PSR következésképpen a PSR-2 tartalmának tisztázását
és hibáinak javítását kísérli meg egy korszerűbb kontextusban, ahol már új
funkcionalitások is elérhetővé váltak.

### Korábbi programnyelv változatok

Jelen dokumentum egészében bármely utasításokat figyelmen kívül LEHET hagyni, ha
azok nem léteztek a projekt által támogatott PHP változatban.

### Példakód

Az alábbi példa a gyors áttekinthetőség érdekében néhány fontosabb szabályt
foglal össze:

~~~php
<?php

declare(strict_types=1);

namespace Vendor\Package;

use Vendor\Package\{ClassA as A, ClassB, ClassC as C};
use Vendor\Package\SomeNamespace\ClassD as D;

use function Vendor\Package\{functionA, functionB, functionC};

use const Vendor\Package\{ConstantA, ConstantB, ConstantC};

class Foo extends Bar implements FooInterface
{
    public function sampleFunction(int $a, int $b = null): array
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
        // a metódus törzse
    }
}
~~~

## 2. Általános szabályok

### 2.1. Alapvető kódolási szabvány

A kódnak követnie KELL a [PSR-1] alapvető kódolási szabvány összes rendelkezését.

A PSR-1 ajánlásban szereplő `StudlyCaps` kifejezés alatt mindig[PascalCase-t](https://hu.wikipedia.org/wiki/CamelCase) KELL érteni, ahol minden
egyes szó (a camelCase írásmódtól eltérően ideértve a legelső szót is) nagy
kezdőbetűvel írandó.

### 2.2 Állományok

Minden PHP fájlban Unix LF (soremelés, `\n`) karaktereket KELL használni a sorok
lezárására.

Minden PHP fájlnak egy üres sorral KELL véget érnie, amelyet egy LF (soremelés,
`\n`) karakter zár le.

A záró `?>` php-címkét el KELL hagyni a kizárólag PHP-kódot tartalmazó fájlokból.

### 2.3 Sorok

A sorhossz tekintetében NEM SZABAD merev korlátokat felállítani.

A rugalmas korlátot 120 karakternél KELL meghúzni.

Az egyes soroknak NEM KELLENE hosszabbnak lennie 80 karakternél; az ennél
hosszabbakat AJÁNLOTT felosztani 80 karakternél nem hosszabb egymást követő sorokra.

A nem üres sorok végére TILOS szóközt tenni.

Az olvashatóság javítása és a kapcsolódó kódblokkok jelzése érdekében üres sorokat
LEHET alkalmazni.

Soronként NEM LEHET egynél több utasítás.

### 2.4 Behúzás

A kódban szintenként 4 szóközt KELL használni a sorok behúzására, tabulátort TILOS.

### 2.5 Kulcsszavak és adattípusok

A fenntartott PHP [kulcsszavak]at és [adattípusok]at kisbetűvel KELL írni. Ez
vonatkozik az eljövendő PHP változatokban bevezetendő új kulcsszavakra és
típusokra is, melyeket szintén kisbetűvel KELL írni.

Az adattípusok nevének rövidített alakját KELL alkalmazni, például `boolean`
helyett `bool`, `integer` helyett `int`, stb.

## 3. Declare Statements, Namespace, and Import Statements

The header of a PHP file may consist of a number of different blocks. If present,
each of the blocks below MUST be separated by a single blank line, and MUST NOT contain
a blank line. Each block MUST be in the order listed below, although blocks that are
not relevant may be omitted.

* Opening `<?php` tag.
* File-level docblock.
* One or more declare statements.
* The namespace declaration of the file.
* One or more class-based `use` import statements.
* One or more function-based `use` import statements.
* One or more constant-based `use` import statements.
* The remainder of the code in the file.

When a file contains a mix of HTML and PHP, any of the above sections may still
be used. If so, they MUST be present at the top of the file, even if the
remainder of the code consists of a closing PHP tag and then a mixture of HTML and
PHP.

When the opening `<?php` tag is on the first line of the file, it MUST be on its
own line with no other statements unless it is a file containing markup outside of PHP
opening and closing tags.

Import statements MUST never begin with a leading backslash as they
must always be fully qualified.

Példa a fenti szabályok alkalmazásra:

~~~php
<?php

/**
 * Ez a fájl kódstílus példákat tartalmaz.
 */

declare(strict_types=1);

namespace Vendor\Package;

use Vendor\Package\{ClassA as A, ClassB, ClassC as C};
use Vendor\Package\SomeNamespace\ClassD as D;
use Vendor\Package\AnotherNamespace\ClassE as E;

use function Vendor\Package\{functionA, functionB, functionC};
use function Another\Vendor\functionD;

use const Vendor\Package\{CONSTANT_A, CONSTANT_B, CONSTANT_C};
use const Another\Vendor\CONSTANT_D;

/**
 * FooBar egy minta osztály.
 */
class FooBar
{
    // ... további PHP kód ...
}

~~~

Compound namespaces with a depth of more than two MUST NOT be used. Therefore the
following is the maximum compounding depth allowed:

~~~php
<?php

use Vendor\Package\SomeNamespace\{
    SubnamespaceOne\ClassA,
    SubnamespaceOne\ClassB,
    SubnamespaceTwo\ClassY,
    ClassZ,
};
~~~

And the following would not be allowed:

~~~php
<?php

use Vendor\Package\SomeNamespace\{
    SubnamespaceOne\AnotherNamespace\ClassA,
    SubnamespaceOne\ClassB,
    ClassZ,
};
~~~

When wishing to declare strict types in files containing markup outside PHP
opening and closing tags, the declaration MUST be on the first line of the file
and include an opening PHP tag, the strict types declaration and closing tag.

For example:
~~~php
<?php declare(strict_types=1) ?>
<html>
<body>
    <?php
        // ... additional PHP code ...
    ?>
</body>
</html>
~~~

Declare statements MUST contain no spaces and MUST be exactly `declare(strict_types=1)`
(with an optional semi-colon terminator).

Block declare statements are allowed and MUST be formatted as below. Note position of
braces and spacing:

~~~php
declare(ticks=1) {
    // some code
}
~~~

## 4. Osztályok, tulajdonságok és metódusok

Az "osztály" kifejezés ebben a dokumentumban egyaránt vonatkozik osztályokra,
interfészekre vagy trait-ekre.

Any closing brace MUST NOT be followed by any comment or statement on the
same line.

When instantiating a new class, parentheses MUST always be present even when
there are no arguments passed to the constructor.

~~~php
new Foo();
~~~

### 4.1 Az `extends` és az `implements` kulcsszavak

Az `extends` és az `implements` kulcsszavaknak az osztálynévvel egy sorban KELL
lenniük.

Az osztálydeklarációk nyitó kapcsos zárójelét külön sorban KELL elhelyezni
(az osztályfejléc után), az osztálytörzset befejező zárójelet pedig a törzs utáni
sorban (BSD kódolási stílus).

A nyitó kapcsos zárójeleknek egy saját sorban KELL lenniük, amely előtt és után
TILOS üres sort hagyni.

A záró kapcsos zárójeleknek egy saját sorban KELL lenniük, amely előtt és után
TILOS üres sort hagyni.

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

Lists of `implements` and, in the case of interfaces, `extends` MAY be split
across multiple lines, where each subsequent line is indented once. When doing
so, the first item in the list MUST be on the next line, and there MUST be only
one interface per line.

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

### 4.2 Using traits

The `use` keyword used inside the classes to implement traits MUST be
declared on the next line after the opening brace.

~~~php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;
}
~~~

Each individual trait that is imported into a class MUST be included
one-per-line and each inclusion MUST have its own `use` import statement.

~~~php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;
use Vendor\Package\SecondTrait;
use Vendor\Package\ThirdTrait;

class ClassName
{
    use FirstTrait;
    use SecondTrait;
    use ThirdTrait;
}
~~~

When the class has nothing after the `use` import statement, the class
closing brace MUST be on the next line after the `use` import statement.

~~~php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;
}
~~~

Otherwise, it MUST have a blank line after the `use` import statement.

~~~php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;

    private $property;
}
~~~

When using the `insteadof` and `as` operators they must be used as follows taking
note of indentation, spacing, and new lines.

~~~php
<?php

class Talker
{
    use A, B, C {
        B::smallTalk insteadof A;
        A::bigTalk insteadof C;
        C::mediumTalk as FooBar;
    }
}
~~~

### 4.3 Properties and Constants

Visibility MUST be declared on all properties.

Visibility MUST be declared on all constants if your project PHP minimum
version supports constant visibilities (PHP 7.1 or later).

The `var` keyword MUST NOT be used to declare a property.

There MUST NOT be more than one property declared per statement.

Property names MUST NOT be prefixed with a single underscore to indicate
protected or private visibility. That is, an underscore prefix explicitly has
no meaning.

There MUST be a space between type declaration and property name.

A property declaration looks like the following:

~~~php
<?php

namespace Vendor\Package;

class ClassName
{
    public $foo = null;
    public static int $bar = 0;
}
~~~

### 4.4 Methods and Functions

Visibility MUST be declared on all methods.

Method names MUST NOT be prefixed with a single underscore to indicate
protected or private visibility. That is, an underscore prefix explicitly has
no meaning.

Method and function names MUST NOT be declared with space after the method name. The
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

A function declaration looks like the following. Note the placement of
parentheses, commas, spaces, and braces:

~~~php
<?php

function fooBarBaz($arg1, &$arg2, $arg3 = [])
{
    // function body
}
~~~

### 4.5 Method and Function Arguments

In the argument list, there MUST NOT be a space before each comma, and there
MUST be one space after each comma.

Method and function arguments with default values MUST go at the end of the argument
list.

~~~php
<?php

namespace Vendor\Package;

class ClassName
{
    public function foo(int $arg1, &$arg2, $arg3 = [])
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

When you have a return type declaration present, there MUST be one space after
the colon followed by the type declaration. The colon and declaration MUST be
on the same line as the argument list closing parenthesis with no spaces between
the two characters.

~~~php
<?php

declare(strict_types=1);

namespace Vendor\Package;

class ReturnTypeVariations
{
    public function functionName(int $arg1, $arg2): string
    {
        return 'foo';
    }

    public function anotherFunction(
        string $foo,
        string $bar,
        int $baz
    ): string {
        return 'foo';
    }
}
~~~

In nullable type declarations, there MUST NOT be a space between the question mark
and the type.

~~~php
<?php

declare(strict_types=1);

namespace Vendor\Package;

class ReturnTypeVariations
{
    public function functionName(?string $arg1, ?int &$arg2): ?string
    {
        return 'foo';
    }
}
~~~

When using the reference operator `&` before an argument, there MUST NOT be
a space after it, like in the previous example.

There MUST NOT be a space between the variadic three dot operator and the argument
name:

```php
public function process(string $algorithm, ...$parts)
{
    // processing
}
```

When combining both the reference operator and the variadic three dot operator,
there MUST NOT be any space between the two of them:

```php
public function process(string $algorithm, &...$parts)
{
    // processing
}
```

### 4.6 `abstract`, `final`, and `static`

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

### 4.7 Method and Function Calls

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
next line, and there MUST be only one argument per line. A single argument being
split across multiple lines (as might be the case with an anonymous function or
array) does not constitute splitting the argument list itself.

~~~php
<?php

$foo->bar(
    $longArgument,
    $longerArgument,
    $muchLongerArgument
);
~~~

~~~php
<?php

somefunction($foo, $bar, [
  // ...
], $baz);

$app->get('/hello/{name}', function ($name) use ($app) {
    return 'Hello ' . $app->escape($name);
});
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
figyeljünk oda a kerek, kapcsos és szögletes zárójelek, vesszők és szóközök
elhelyezésére; valamint arra, hogy az `else` és `elseif` kulcsszavak ugyan abban
a sorban legyenek, mint az őket megelőző kifejezés törzsét lezáró kapcsos zárójel.

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

Az `elseif` kulcsszót KELLENE használni az `else if` alak helyett, hogy minden
vezérlő kulcsszó egységesen egybe legyen írva.

A zárójelben lévő kifejezéseket fel LEHET osztani több sorra is, ahol az egyes
sorokat legalább egyszeres behúzással kell kezdeni. Ebben az esetben az első
feltételnek a következő sorban KELL helyet foglalnia. A befejező kerek zárójelnek
és a törzset nyitó kapcsos zárójelnek egyazon sorban KELL lennie, egy szóközzel
elválasztva. A feltételek közötti esetleges logikai operátoroknak következetesen
mindig vagy az adott sor elején, vagy a végén kell lenniük, ezt nem ajánlott
keverni.

Expressions in parentheses MAY be split across multiple lines, where each
subsequent line is indented at least once. When doing so, the first condition
MUST be on the next line. The closing parenthesis and opening brace MUST be
placed together on their own line with one space between them. Boolean
operators between conditions MUST always be at the beginning or at the end of
the line, not a mix of both.

~~~php
<?php

if (
    $expr1
    && $expr2
) {
    // if body
} elseif (
    $expr3
    && $expr4
) {
    // elseif body
}
~~~

### 5.2. A `switch` szerkezet és a `case` elágazás

A `switch` szerkezetnek az alábbi példakódhoz hasonlóan kell kinéznie. Különösen
figyeljünk oda a kerek és kapcsos zárójelek, szóközök elhelyezésére.
A `case` elágazást a `switch`-hez viszonyítva egy behúzással beljebb KELL kezdeni,
a `break` kulcsszót (vagy más lezáró kulcsszót) a `case` törzsével azonos szinten
KELL elhelyezni. Amikor egy nem üres `case`-törzsben a továbblépés szándékos,
akkor olyan megjegyzéseket KELL elhelyezni, mint az alábbi példában is szereplő
`// no break`.

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

A zárójelben lévő kifejezéseket fel LEHET osztani több sorra is, ahol az egyes
sorokat legalább egyszeres behúzással kell kezdeni. Ebben az esetben az első
feltételnek a következő sorban KELL helyet foglalnia. A befejező kerek zárójelnek
és a törzset nyitó kapcsos zárójelnek egyazon sorban KELL lennie, egy szóközzel
elválasztva. A feltételek közötti esetleges logikai operátoroknak következetesen
mindig vagy az adott sor elején, vagy a végén kell lenniük, ezt nem ajánlott
keverni.

~~~php
<?php

switch (
    $expr1
    && $expr2
) {
    // structure body
}
~~~

### 5.3 `while`, `do while`

A `while` statement looks like the following. Note the placement of
parentheses, spaces, and braces.

~~~php
<?php

while ($expr) {
    // structure body
}
~~~

A zárójelben lévő kifejezéseket fel LEHET osztani több sorra is, ahol az egyes
sorokat legalább egyszeres behúzással kell kezdeni. Ebben az esetben az első
feltételnek a következő sorban KELL helyet foglalnia. A befejező kerek zárójelnek
és a törzset nyitó kapcsos zárójelnek egyazon sorban KELL lennie, egy szóközzel
elválasztva. A feltételek közötti esetleges logikai operátoroknak következetesen
mindig vagy az adott sor elején, vagy a végén kell lenniük, ezt nem ajánlott
keverni.

~~~php
<?php

while (
    $expr1
    && $expr2
) {
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

A zárójelben lévő kifejezéseket fel LEHET osztani több sorra is, ahol az egyes
sorokat legalább egyszeres behúzással kell kezdeni. Ebben az esetben az első
feltételnek a következő sorban KELL helyet foglalnia. A feltételek közötti
esetleges logikai operátoroknak következetesen mindig vagy az adott sor elején,
vagy a végén kell lenniük, ezt nem ajánlott keverni.

~~~php
<?php

do {
    // structure body;
} while (
    $expr1
    && $expr2
);
~~~

### 5.4 `for`

A `for` ciklusnak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php

for ($i = 0; $i < 10; $i++) {
    // for body
}
~~~

A zárójelben lévő kifejezéseket fel LEHET osztani több sorra is, ahol az egyes
sorokat legalább egyszeres behúzással kell kezdeni. Ebben az esetben az első
feltételnek a következő sorban KELL helyet foglalnia. A feltételek közötti
esetleges logikai operátoroknak következetesen mindig vagy az adott sor elején,
vagy a végén kell lenniük, ezt nem ajánlott keverni.

~~~php
<?php

for (
    $i = 0;
    $i < 10;
    $i++
) {
    // for body
}
~~~

### 5.5 `foreach`

A `foreach` ciklusnak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php

foreach ($iterable as $key => $value) {
    // foreach body
}
~~~

### 5.6 `try`, `catch`, `finally`

A `try-catch-finally` blokknak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php

try {
    // try body
} catch (FirstThrowableType $e) {
    // catch body
} catch (OtherThrowableType | AnotherThrowableType $e) {
    // catch body
} finally {
    // finally body
}
~~~

## 6. Operátorok

Style rules for operators are grouped by arity (the number of operands they take).

When space is permitted around an operator, multiple spaces MAY be
used for readability purposes.

All operators not described here are left undefined.

### 6.1. Egyoperandusú operátorok

The increment/decrement operators MUST NOT have any space between
the operator and operand.
~~~php
$i++;
++$j;
~~~

Type casting operators MUST NOT have any space within the parentheses:
~~~php
$intValue = (int) $input;
~~~

### 6.2. Kétoperandusú operátorok

Minden kétoperandusú ([aritmetikai], [összehasonlító], [értékadó], [bitorientált],
[logikai], [karakterlánc], és [típus]) operátort MUST be preceded and
followed by at least one space:

~~~php
if ($a === $b) {
    $foo = $bar ?? $a ?? $b;
} elseif ($a > $b) {
    $foo = $a + $b * $c;
}
~~~

### 6.3. Feltételes (háromoperandusú, ternális) operátor

The conditional operator, also known simply as the ternary operator, MUST be
preceded and followed by at least one space around both the `?`
and `:` characters:
~~~php
$variable = $foo ? 'foo' : 'bar';
~~~

When the middle operand of the conditional operator is omitted, the operator
MUST follow the same style rules as other binary [comparison] operators:

~~~php
$variable = $foo ?: 'bar';
~~~

## 7. Closures

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

If a return type is present, it MUST follow the same rules as with normal
functions and methods; if the `use` keyword is present, the colon MUST follow
the `use` list closing parentheses with no spaces between the two characters.

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

$closureWithArgsVarsAndReturn = function ($arg1, $arg2) use ($var1, $var2): bool {
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

## 8. Névtelen osztályok

Anonymous Classes MUST follow the same guidelines and principles as closures
in the above section.

~~~php
<?php

$instance = new class {};
~~~

The opening brace MAY be on the same line as the `class` keyword so long as
the list of `implements` interfaces does not wrap. If the list of interfaces
wraps, the brace MUST be placed on the line immediately following the last
interface.

~~~php
<?php

// Brace on the same line
$instance = new class extends \Foo implements \HandleableInterface {
    // Class content
};

// Brace on the next line
$instance = new class extends \Foo implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
    // Class content
};
~~~

[PSR-1]: PSR-1-basic-coding-standard.md
[PSR-2]: PSR-2-coding-style-guide.md
[RFC 2119] ../related-rfcs/2119.md
[kulcsszavak]: http://php.net/manual/en/reserved.keywords.php
[adattípusok]: http://php.net/manual/en/reserved.other-reserved-words.php
[aritmetikai]: http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=3#section_7_2
[értékadó]: http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=3#section_7_4
[összehasonlító]: http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=3#section_7_1
[bitorientált]: http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=3#section_7_8
[logikai]: http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=3#section_7_9
[karakterlánc]: http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=3#section_7_3
[típus]: http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=10#section_3_11

[Kezdőlap](../README.md)
