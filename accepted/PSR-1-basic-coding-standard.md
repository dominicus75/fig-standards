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

## 2. Forrásfájlok

### 2.1. PHP címkék

A PHP kódban kizárólag a hosszú `<?php ?>` illetve a rövid-echoként is emlegetett `<?= ?>` címkéket KELL alkalmazni; bármilyen más címkevariáció alkalmazása TILOS.

### 2.2. Karakterkódolás

A PHP kódban kizárólag BOM nélküli UTF-8 kódolást KELL használni.

### 2.3. Mellékhatások

Egy forrásfájlnak csak új adatszerkezeteket és működési logikát (osztályok, függvények, állandók,
stb.) KELLENE deklarálni és nem okozhat más mellékhatást, illetve nem változtathatja meg az alkalmazás állapotát; vagy a csak működési logikát KELLENE végrehajtania mellékhatásokat is kiváltva, de NEM KELLENE mindkettőt egyszerre.

A "mellékhatások" kifejezés olyan logika *pusztán a tartalmazó fájl betöltődéséből fakadó* végrehajtását jelenti, amely nem közvetlenül az osztályok, függvények, állandók és egyebek deklarálásához kapcsolódik.

A "mellékhatások" közé tartozik: a kimenet generálása, a `require` vagy `include` használata, kapcsolódás külső szolgáltatásokhoz, az ini beállítások módosítása, hibák és kivételek dobása, globális és statikus változók módosítása, állományok beolvasása és írása, stb.

Az alábbiakban egy olyan kód látható, amelyben ugyanazon fájlban találhatunk deklarációkat és mellékhatást kiváltó utasításokat, tehát egy példát arra, hogy mit kell kerülnünk:

~~~php
<?php
// mellékhatás: php.ini beállítás megváltoztatása
ini_set('error_reporting', E_ALL);

// mellékhatás: állomány betöltése
include "file.php";

// mellékhatás: kimenet generálása
echo "<html>\n";

// függvény deklaráció
function foo()
{
    // function body
}
~~~

A következő példakód egy olyan fájlt mutat be, amely mellékhatások nélkül tartalmazza a függvény deklarációt, vagyis a követendő példát szemlélteti:

~~~php
<?php

// függvény deklaráció
function foo()
{
    // függvénytörzs
}

// egy feltételes deklaráció, ami nem minősül mellékhatásnak
if (! function_exists('bar')) {
    function bar()
    {
        // függvénytörzs
    }
}
~~~

## 3. Névtér és osztálynevek

Az osztályok és névterek elnevezésénél a következő ajánlásokat KELL figyelembe venni: [[PSR-0], [PSR-4]]

Ez azt jelenti, hogy minden osztálynak (interfésznek, trait-nek) önmagában egy külön fájlban kell lennie, és legalább egy névtérben kell helyet foglalnia, ez a kötelezően elvárt legfelsőbb szintű névtér pedig a szállító (vendor) egyedi neve.

Az osztályok elnevezésében a `StudlyCaps` vagy `PascalCase` stílust KELL követni. Ez azt jelenti, hogy a névnek nagy betűvel kell kezdődnie és több szóból álló elnevezés esetén minden szót nagy kezdőbetűvel (és nem aláhúzás karakterrel) választunk el a többitől, valahogy így: `OsztalyNeveAmiCsinalValamit`.

A PHP 5.3 vagy újabb változataiban írt kódokban formális névtereket KELL használni.

Példakód:

~~~php
<?php
// PHP 5.3 vagy újabb:
namespace Vendor\Model;

class Foo
{
}
~~~

A PHP 5.2.x vagy annál régebbi változatiban írt kódban AJÁNLOTT az ál/pszeudo-névtér konvenció alkalmazása, ahol a szállító neve előtagként (`Vendor_`) kapcsolódik az osztály nevéhez, aláhúzás karakterrel elválasztva.

~~~php
<?php
// PHP 5.2.x vagy régebbi:
class Vendor_Model_Foo
{
}
~~~

## 4. Osztályállandók, tulajdonságok és metódusok

Az "osztály" kifejezés jelen dokumentumban az összes osztályra, interfészre, trait-re vonatkozik.

### 4.1. Állandók

Az osztályállandók elnevezésének kizárólag nagybetűt és (szóelválasztóként) aláhúzás karaktert KELL tartalmaznia, mint az alábbi példában:

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

Jelen dokumentum szándékosan kerüli, hogy bármilyen ajánlást fogalmazzon meg a `$StudlyCaps`, `$camelCase`, vagy `$under_score` stílusú tulajdonságnevekkel kapcsolatban.

Akármelyik elnevezési konvenciót lehet alkalmazni, de arra KELLENE törekedni, hogy ezt észszerű határokon belül következetesen tegyük. A hatókör, amelyen belül egységesen egy elnevezési konvencióhoz KELLENE ragaszkodni lehet szállítói-, csomag-, osztály-, vagy metódus szint.

### 4.3. Metódusok

A metódusok/függvények elnevezésében a `camelCase` stílust KELL alkalmazni. Ez azt jelenti, hogy a névnek kisbetűvel kell kezdődnie és több szóból álló elnevezés esetén minden szót nagy kezdőbetűvel (és nem aláhúzás karakterrel) választunk el a többitől, valahogy így: `valamitCsinaloFuggveny()`.
