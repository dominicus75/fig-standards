# Alapvető kódolási szabvány

Ez a PSR-ajánlás azokat a szabványos kódolási konvenciókat tartalmazza, amelyek a megosztott PHP-kódok magas szintű együttműködéséhez szükségesek.

A csupa nagybetűvel szedett kiemelt kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

[PSR-0]: PSR-0.md
[PSR-4]: PSR-4-autoloader.md

## 1. Áttekintés

- A forrásfájlokban kizárólag `<?php` és `<?=` címkéket KELL használni.

- A PHP forrásfájlokat kizárólag UTF-8 kódolással KELL elkészíteni, [BOM-kódolás](https://hu.wikipedia.org/wiki/UTF-8#BOM) nélkül.

- A forrásfájloknak AJÁNLOTT kizárólag entitásokat (osztályok, függvények, állandók, stb.) deklarálni, *vagy* mellékhatásokat kiváltani (pl. kimenetet generálása, .ini beállítások megváltoztatása, stb.), de NEM KELLENE mindkettőt csinálni egyszerre.

- Az osztályok és névterek elnevezésénél a következő ajánlásokat KELL figyelembe venni: [[PSR-0], [PSR-4]].

- Az osztályok elnevezésében a `StudlyCaps` vagy `PascalCase` stílust KELL követni.

- Az osztályállandók elnevezésének kizárólag nagybetűt és (szóelválasztóként) aláhúzás karaktert KELL tartalmaznia.

- A metódusok/függvények elnevezésében a `camelCase` stílust KELL alkalmazni.

## 2. Files

### 2.1. PHP címkék

A PHP kódban kizárólag a hosszú `<?php ?>` címkék és a rövid-echoként is emlegetett `<?= ?>` címkéket KELL alkalmazni; bármilyen más címkevariáció alkalmazása TILOS.

### 2.2. Karakterkódolás

A PHP kódban kizárólag BOM nélküli UTF-8 kódolást KELL használni.

### 2.3. Mellékhatások

A file SHOULD declare new symbols (classes, functions, constants,
etc.) and cause no other side effects, or it SHOULD execute logic with side
effects, but SHOULD NOT do both.

The phrase "side effects" means execution of logic not directly related to
declaring classes, functions, constants, etc., *merely from including the
file*.

"Side effects" include but are not limited to: generating output, explicit
use of `require` or `include`, connecting to external services, modifying ini
settings, emitting errors or exceptions, modifying global or static variables,
reading from or writing to a file, and so on.

The following is an example of a file with both declarations and side effects;
i.e, an example of what to avoid:

~~~php
<?php
// side effect: change ini settings
ini_set('error_reporting', E_ALL);

// side effect: loads a file
include "file.php";

// side effect: generates output
echo "<html>\n";

// declaration
function foo()
{
    // function body
}
~~~

The following example is of a file that contains declarations without side
effects; i.e., an example of what to emulate:

~~~php
<?php
// declaration
function foo()
{
    // function body
}

// conditional declaration is *not* a side effect
if (! function_exists('bar')) {
    function bar()
    {
        // function body
    }
}
~~~

## 3. Névtér és osztálynevek

Namespaces and classes MUST follow an "autoloading" PSR: [[PSR-0], [PSR-4]].

This means each class is in a file by itself, and is in a namespace of at
least one level: a top-level vendor name.

Class names MUST be declared in `StudlyCaps`.

Code written for PHP 5.3 and after MUST use formal namespaces.

For example:

~~~php
<?php
// PHP 5.3 and later:
namespace Vendor\Model;

class Foo
{
}
~~~

Code written for 5.2.x and before SHOULD use the pseudo-namespacing convention
of `Vendor_` prefixes on class names.

~~~php
<?php
// PHP 5.2.x and earlier:
class Vendor_Model_Foo
{
}
~~~

## 4. Osztályállandók, tulajdonságok és metódusok

The term "class" refers to all classes, interfaces, and traits.

### 4.1. Állandók

Class constants MUST be declared in all upper case with underscore separators.
For example:

~~~php
<?php
namespace Vendor\Model;

class Foo
{
    const VERSION = '1.0';
    const DATE_APPROVED = '2012-06-01';
}
~~~

### 4.2. Tulajdonságok

This guide intentionally avoids any recommendation regarding the use of
`$StudlyCaps`, `$camelCase`, or `$under_score` property names.

Whatever naming convention is used SHOULD be applied consistently within a
reasonable scope. That scope may be vendor-level, package-level, class-level,
or method-level.

### 4.3. Metódusok

Method names MUST be declared in `camelCase()`.
