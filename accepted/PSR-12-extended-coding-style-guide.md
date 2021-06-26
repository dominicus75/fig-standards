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

namespace Szallito\Csomag;

use Szallito\Csomag\{OsztalyA as A, OsztalyB, OsztalyC as C};
use Szallito\Csomag\ValamiNevter\OsztalyD as D;
use Szallito\Csomag\MasikNevter\OsztalyE as E;

use function Szallito\Csomag\{functionA, functionB, functionC};

use const Szallito\Csomag\{CONSTANT_A, CONSTANT_B, CONSTANT_C};

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

A PSR-1 ajánlásban szereplő `StudlyCaps` kifejezés alatt mindig [PascalCase-t](https://hu.wikipedia.org/wiki/CamelCase)
KELL érteni, ahol minden egyes szó (a camelCase írásmódtól eltérően ideértve a
legelső szót is) nagy kezdőbetűvel írandó.

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

## 3. Névterek, deklarációs és importáló utasítások

A PHP állományok fejléce több különböző blokkot foglalhat magába. Ha van fejléc,
akkor minden alábbi blokkot egy üres sorral KELL elválasztani egymástól, de magának
a blokknak NEM SZABAD üres sort tartalmaznia. Minden blokknak az alábbi sorrendben
KELL szerepelnie a fejlécben, a nem releváns blokkok azonban elhagyhatók.

* Nyitó `<?php` cimke.
* Állomány-szintű dokumentációs blokk.
* Egy vagy több deklarációs utasítás.
* Az állomány névtér-deklarációja.
* Egy vagy több osztály-importáló utasítás a `use` kulcsszó használatával.
* Egy vagy több függvény-importáló utasítás a `use` kulcsszó használatával.
* Egy vagy több konstans-importáló utasítás a `use` kulcsszó használatával.
* Az állomány kódjának további része.

Ha az állomány vegyesen tartalmaz PHP és HTML kódot, a fenti szekciók bármelyike
továbbra is használható. Ez esetben ezeknek az állomány elején KELL helyet foglalniuk,
még akkor is, ha a kód további része PHP záró-tagot is magában foglal, a PHP és HTML
kódrészeket elválasztandó.

Ha a nyitó `<?php` tag az állomány első sorában van elhelyezve, akkor ennek az ő
saját sorának KELL lennie, ide más kód nem írható, kivéve ha a fájl HTML kódot is
tartalmaz.

Az importáló utasításoknak soha NEM SZABAD vissza perjellel (backslash) kezdődni,
mivel mindig teljesen minősítettnek kell lenniük.

Példa a fenti szabályok alkalmazásra:

~~~php
<?php

/**
 * Ez a fájl kódstílus példákat tartalmaz.
 */

declare(strict_types=1);

namespace Szallito\Csomag;

use Szallito\Csomag\{OsztalyA as A, OsztalyB, OsztalyC as C};
use Szallito\Csomag\ValamiNevter\OsztalyD as D;
use Szallito\Csomag\MasikNevter\OsztalyE as E;

use function Szallito\Csomag\{functionA, functionB, functionC};
use function MasikSzallito\Csomag\functionD;

use const Szallito\Csomag\{CONSTANT_A, CONSTANT_B, CONSTANT_C};
use const MasikSzallito\Csomag\CONSTANT_D;

/**
 * FooBar egy minta osztály.
 */
class FooBar
{
    // ... további PHP kód ...
}

~~~

(A szállító és a csomag nevén felüli) kettőnél több névtér használata TILOS, tehát
a `Szállító\Csomag\Névtér\Alnévtér` sémánál mélyebb tagolás nem engedélyezett. A
következő példakód a lehetséges legnagyobb névtér-összetettséget szemlélteti:

~~~php
<?php

use Szallito\Csomag\Nevter\{
    AlnevterEgy\OsztalyA,
    AlnevterKetto\OsztalyB,
    AlnevterHarom\OsztalyY,
    OsztalyZ,
};
~~~

A következő kód már nem engedélyezett mélységű névteret tartalmaz:

~~~php
<?php

use Szallito\Csomag\Nevter\{
    AlNevter\MasikNevter\OsztalyA,
    MasikAlnevter\OsztalyB,
    OsztalyZ,
};
~~~

Ha egy HTML kódot is tartalmazó PHP állományban szeretnénk deklarálni a szigorú
módot (strict types), akkor a deklarációnak a fájl első sorában KELL elhelyezkedni,
a nyitó és záró PHP-tag között, ahogy az alábbi példakódban is látható:

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

A deklarációs utasításnak TILOS szóközt tartalmaznia, ellenben pontosan így KELL
kinéznie: `declare(strict_types=1)` (az utasítás végén az opcionális pontosvesszővel).

Engedélyezett a deklarációs utasítások blokkba foglalása is, ezeket az alább látható
módon KELL megformázni, ügyelve a zárójelek elhelyezkedésére és a térközökre.

~~~php
declare(ticks=1) {
    // some code
}
~~~

## 4. Osztályok, tulajdonságok és metódusok

Az "osztály" kifejezés ebben a dokumentumban egyaránt vonatkozik osztályokra,
interfészekre vagy trait-ekre.

A záró kapcsos zárójelekkel azonos sorban TILOS bármi mást (megjegyzéseket, utasításokat,
stb.) elhelyezni.

Új osztály példányosításakor a kerek zárójeleket akkor is ki KELL tenni, ha nem adunk
át semmilyen paramétert a konstruktornak.

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

namespace Szallito\Csomag;

use IzeOsztaly;
use BigyoOsztaly as Bigyo;
use MasikSzallito\MasikCsomag\BigyoOsztaly;

class OsztalyNev extends SzuloOsztaly implements \ArrayAccess, \Countable
{
    // állandók, tulajdonságok, metódusok
}
~~~

A megvalósított interfészek (`implements`), illetve a szülő osztályok (`extends`)
listáját több sorba is szét LEHET tördelni, ahol minden egyes sort egyszeres behúzással
kell kezdeni. Ennél a megoldásnál a lista első elemének a következő sorba KELL kerülnie
és soronként csak egyetlen interfészt/ősosztályt KELL feltüntetni.

~~~php
<?php

namespace Szallito\Csomag;

use IzeOsztaly;
use BigyoOsztaly as Bigyo;
use MasikSzallito\MasikCsomag\BigyoOsztaly;

class OsztalyNev extends SzuloOsztaly implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
    // állandók, tulajdonságok, metódusok
}
~~~

### 4.2 Trait-ek használata

Ha `use` kulcsszót a trait-et használó osztály belsejében használjuk, akkor
az osztálytörzs kezdetét jelző nyitó kapcsos zárójel utáni sorba KELL elhelyezni.

~~~php
<?php

namespace Szallito\Csomag;

use Szallito\Csomag\ElsoTrait;

class OsztalyNev
{
    use FirstTrait;
}
~~~

Az osztályba importált minden egyes trait-et külön sorban KELL feltüntetni (soronként
egyet), illetve mindegyiknek rendelkeznie KELL saját `use` importáló utasítással.

~~~php
<?php

namespace Szallito\Csomag;

use Szallito\Csomag\ElsoTrait;
use Szallito\Csomag\MasodikTrait;
use Szallito\Csomag\HarmadikTrait;

class OsztalyNev
{
    use ElsoTrait;
    use MasodikTrait;
    use HarmadikTrait;
}
~~~

Amikor az osztály törzse a `use` importáló utasításon kívül mást nem tartalmaz,
akkor a lezáró kapcsos zárójelnek az importáló utasítást követő sorban KELL lennie.

~~~php
<?php

namespace Szallito\Csomag;

use Szallito\Csomag\ElsoTrait;

class OsztalyNev
{
    use ElsoTrait;
}
~~~

Egyébként a `use` importáló utasítást üres sornak KELL követnie:

~~~php
<?php

namespace Szallito\Csomag;

use Szallito\Csomag\ElsoTrait;

class OsztalyNev
{
    use ElsoTrait;

    private $property;
}
~~~

Az `insteadof` és az `as` operátor használata esetén ezeket a következők szerint
kell használni, ügyelve a behúzásra, a térközökre és az új sorokra.

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

### 4.3 Tulajdonságok és állandók

A láthatóságot minden objektumtulajdonsághoz meg KELL adni.

A láthatóságot minden állandóhoz is be KELL állítani, abban az esetben, ha a
projektben alkalmazott PHP változat támogatja az állandók láthatósági szintjének
megadását (PHP 7.1 vagy újabb verzió).

Az elavult, de kompatibilitási okokból még támogatott `var` kulcsszót TILOS használni
az objektumtulajdonságok deklarálásánál.

Utasításonként egynél több tulajdonságot TILOS deklarálni.

Az objektumtulajdonságok láthatósági szintjét (`private` vagy `protected`)
TILOS aláhúzás karakterrel jelezni a tulajdonságnév elején.

A típusdeklaráció és a tulajdonság neve közé egy szóközt KELL tenni.

Az előadottak tükrében egy objektumtulajdonság-deklarációnak valahogy így kellene
kinéznie:

~~~php
<?php

namespace Szallito\Csomag;

class OsztalyNev
{
    public $foo = null;
    public static int $bar = 0;
}
~~~

### 4.4 Metódusok és függvények

A láthatóságot minden metódushoz meg KELL adni.

A metódusok láthatósági szintjét (`private` vagy `protected`) TILOS aláhúzás karakterrel
jelezni a metódusnév elején.

A metódus-, és függvénynév után TILOS szóközt tenni. A metódus esetleges paramétereit
tartalmazó kerek zárójel nyitó eleme után és záró eleme előtt NEM LEHET szóköz. A
függvénytörzset nyitó kapcsos zárójelét külön sorban KELL elhelyezni (a fejléc után),
a függvénytörzset befejező kapcsos zárójelet pedig a törzs utáni sorban.

A fentiek alapján egy metódus deklarációnak a következőképpen kellene kinéznie.
Különösen figyeljünk oda a kerek, kapcsos és szögletes zárójelek, vesszők és
szóközök elhelyezésére:

~~~php
<?php

namespace Szallito\Csomag;

class OsztalyNev
{
    public function fooBarBaz($arg1, &$arg2, $arg3 = [])
    {
        // metódus törzs
    }
}
~~~

Egy függvény deklarációnak pedig a következőképpen kellene kinéznie. Különösen
figyeljünk oda a kerek, kapcsos és szögletes zárójelek, vesszők és szóközök
elhelyezésére:

~~~php
<?php

function fooBarBaz($arg1, &$arg2, $arg3 = [])
{
    // function body
}
~~~

### 4.5 Metódusok és függvények paraméterei

A metódusok paraméterlistájában az egyes paraméterek utáni vessző elé TILOS
szóközt tenni, a vessző után ellenben SZÜKSÉGES.

Az alapértelmezett értéket tartalmazó metódus paramétereknek a lista végén KELL
elhelyezkedni.

~~~php
<?php

namespace Szallito\Csomag;

class OsztalyNev
{
    public function foo(int $arg1, &$arg2, $arg3 = [])
    {
        // metódus törzs
    }
}
~~~

A metódus paraméterlistáját több sorba is szét LEHET tördelni, ahol minden egyes sort
egyszeres behúzással kell kezdeni. Ennél a megoldásnál a lista első elemének a
következő sorba KELL kerülnie és soronként csak egyetlen paramétert KELL feltüntetni.

Ha a paraméterlistát külön sorokba tördeljük, akkor a listát lezáró kerek zárójelet
és a függvénytörzs kezdetét jelző nyitó kapcsos zárójelet egy sorba KELL írni,
szóközzel elválasztva, ahogy az alábbi példában is látható:

~~~php
<?php

namespace Szallito\Csomag;

class OsztalyNev
{
    public function aVeryLongMethodName(
        ClassTypeHint $arg1,
        &$arg2,
        array $arg3 = []
    ) {
        // metódus törzs
    }
}
~~~

Ha a visszatérési érték típusát is meg kell adni, akkor az ezt jelző kettőspont és
a típusdeklaráció közé egy szóközt KELL tenni. A kettőspontnak és a típusdeklarációnak
a paraméterlistával azonos sorban KELL elhelyezkedni, úgy, hogy a paraméterlistát
lezáró kerek zárójel és a kettőspont közé nem kell szóközt tenni.

~~~php
<?php

declare(strict_types=1);

namespace Szallito\Csomag;

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

Nullable típusdeklaráció esetén a kérdőjel és a visszatérési típus közé TILOS szóközt
tenni.

~~~php
<?php

declare(strict_types=1);

namespace Szallito\Csomag;

class ReturnTypeVariations
{
    public function functionName(?string $arg1, ?int &$arg2): ?string
    {
        return 'foo';
    }
}
~~~

Ha az argumentum előtt referencia-operátort (`&`) használunk, akkor TILOS szóközt
tenni utána, ahogy az előző példában is láthattuk.

A változó hosszúságú paraméterlistát jelző hármaspont operátor (variadic oparator)
és a paraméter neve közé TILOS szóközt tenni:

```php
public function process(string $algorithm, ...$parts)
{
    // processing
}
```

Ha a referencia-, és a hármaspont operátort kombináljuk, akkor szintén TILOS szóközt
tenni a két operátor közé:

```php
public function process(string $algorithm, &...$parts)
{
    // processing
}
```

### 4.6 `abstract`, `final`, és `static` kulcsszavak

Ha szükség van rájuk, akkor az `abstract` és a `final` kulcsszavaknak meg KELL
előzniük a láthatósági beállítást a deklarációkban.

Ellenben a `static` kulcsszónak a láthatósági deklaráció után KELL állnia.

~~~php
<?php

namespace Szallito\Csomag;

abstract class OsztalyNev
{
    protected static $foo;

    abstract protected function zim();

    final public static function bar()
    {
        // metódus törzs
    }
}
~~~

### 4.7 Metódus-, és függvényhívás

Metódus-, vagy függvényhívás esetén a hívott metódus vagy függvény neve és a paraméterlista
kezdő zárójele közé - a 4.4. pontban megénekelt metódus-deklarációhoz hasonlóan - TILOS
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
ahol minden egyes sort egyszeres behúzással kell kezdeni. Ennél a megoldásnál
a lista első elemének a következő sorba KELL kerülnie és soronként csak egyetlen
argumentumot KELL feltüntetni. Ha egy egyedüli paraméter több sort foglal el (ahogy
a névtelen függvények vagy a tömbök esetében), az nem jelenti paraméterlista
széttördelését.

~~~php
<?php

$foo->bar(
    $hosszuArgumentum,
    $hosszabbArgumentum,
    $iszonyuHosszuArgumentum
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
- A törzsnek a nyitó zárójel után következő sorban KELL kezdődnie
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

A `while` ciklusnak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek, szóközök megfelelő elhelyezésére.

~~~php
<?php

while ($expr) {
    // ciklusmag
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
    // ciklusmag
}
~~~

A `do while` ciklus küllemben nem sokban különbözik a `while` ciklustól, így a
rá vonatkozó szabályok is hasonlóak. Figyeljünk oda itt is a kerek és kapcsos
zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php

do {
    // ciklusmag;
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
    // ciklusmag;
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
    // ciklusmag
}
~~~

A zárójelben lévő kifejezéseket fel LEHET osztani több sorra is, ahol az egyes
sorokat legalább egyszeres behúzással kell kezdeni. Ebben az esetben az első
feltételnek a következő sorban KELL helyet foglalnia. A befejező kerek zárójelnek
és a törzset nyitó kapcsos zárójelnek egyazon sorban KELL lennie, egy szóközzel
elválasztva.

~~~php
<?php

for (
    $i = 0;
    $i < 10;
    $i++
) {
    // ciklusmag
}
~~~

### 5.5 `foreach`

A `foreach` ciklusnak a következő példakódhoz hasonlóan kell kinéznie. Figyeljünk
oda a kerek és kapcsos zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php

foreach ($iterable as $key => $value) {
    // ciklusmag
}
~~~

### 5.6 `try`, `catch`, `finally`

A `try-catch-finally` blokknak a következő példakódhoz hasonlóan kell kinéznie.
Figyeljünk oda a kerek és kapcsos zárójelek és szóközök megfelelő elhelyezésére.

~~~php
<?php

try {
    // try törzs
} catch (FirstThrowableType $e) {
    // catch törzs
} catch (OtherThrowableType | AnotherThrowableType $e) {
    // catch törzs
} finally {
    // finally törzs
}
~~~

## 6. Operátorok

A stílusszabályok az operandusok száma szerint vannak csoportosítva.

Ha a szóköz használata engedélyezett az operátor körül, akkor akár többet is
LEHET használni belőle, ha ez javítja az olvashatóságot.

### 6.1. Egyoperandusú operátorok

A léptető (inkrementáló/dekrementáló) operátoroknál TILOS szóközt tenni az
operátor és az operandus közé.

~~~php

$i++;
++$j;
~~~

A típuskonvertáló operátoroknál TILOS szóközt alkalmazni a zárójelen belül:

~~~php

$intValue = (int) $input;
~~~

### 6.2. Kétoperandusú operátorok

Minden kétoperandusú ([aritmetikai], [összehasonlító], [értékadó], [bitorientált],
[logikai], [karakterlánc], és [típus]) operátort legalább egy szóköznek KELL megelőzni
és követni:

~~~php
if ($a === $b) {
    $foo = $bar ?? $a ?? $b;
} elseif ($a > $b) {
    $foo = $a + $b * $c;
}
~~~

### 6.3. Feltételes (háromoperandusú, ternális) operátor

A feltételes, más néven ternális operátorban szereplő kérdőjel és kettőspont
karaktert legalább egy szóköznek KELL megelőzni és követni:

~~~php

$variable = $foo ? 'foo' : 'bar';
~~~

Ha a feltételes operátor középső operandusát elhagyják ("Elvis operátor"), az
operátornak ugyanazt a stílusszabályt KELL követnie, mint más bináris
[összehasonlító] operátoroknak:

~~~php

$variable = $foo ?: 'bar';
~~~

## 7. Névtelen függvények

*A névtelen függvény (más néven closure) olyan névtelen függvény, ami a paraméterlistában
átadott argumentumok mellett hozzáférhet a hatókörén kívül létrehozott azon változókhoz
is, amelyek a függvénydeklaráció záradékában (a `use` kulcsszó utáni kerek zárójelben)
vannak felsorolva.*

Névtelen függvény létrehozásánál a `function` kulcsszó után szóközt KELL
tenni, ahogy a záradékot jelölő `use` kulcsszó előtt és után is.

A függvénytörzset megnyitó kapcsos zárójelnek (az 5. fejezetben taglalt vezérlési
szerkezetekhez hasonlóan) egy sorban KELL lennie a függvényfejléccel, a törzset
lezáró kapcsos zárójelnek viszont a törzs utáni sorban KELL lennie.

A névtelen függvény paraméter-, és váltózólistáját tartalmazó kerek
zárójelek nyitó eleme után és záró eleme előtt NEM SZABAD szóközt hagyni.

A névtelen függvény paraméter-, és váltózólistájában az egyes változónevek
utáni vessző elé TILOS szóközt tenni, a vessző után ellenben SZÜKSÉGES.

Az alapértelmezett értéket tartalmazó paramétereknek a lista végére KELL kerülni.

Ha van visszatérési érték, annak ugyan azokat a szabályokat KELL követnie, mint
amelyek a normál függvényekre és metódusokra érvényesek. Ha jelen van a `use` kulcsszó,
akkor a visszatérési érték típusát bevezető kettőspontnak a `use`-záradék végét jelző
zárójel után KELL következnie, úgy, hogy a zárójel és a kettőspont között nem
lehet szóköz.

Egy névtelen függvény deklarációjának az alábbi kódhoz hasonlóan kell
kinéznie. Figyeljünk oda a kerek és kapcsos zárójelek és szóközök megfelelő
elhelyezésére:

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

A névtelen függvény paraméter-, és váltózólistáját több sorba is szét
LEHET tördelni, ahol minden egyes sort egyszeres behúzással kell kezdeni. Ennél
a megoldásnál a lista első elemének a következő sorba KELL kerülnie és soronként
csak egyetlen paramétert/változót KELL feltüntetni.

Ha a paraméter-, és váltózólistát külön sorokba tördeljük, akkor a listát lezáró
kerek zárójelet és a függvénytörzs kezdetét jelző nyitó kapcsos zárójelet azonos
sorba KELL írni, szóközzel elválasztva.

A következő példakódok olyan névtelen függvény-deklarációkat mutatnak be, amelyek
a paraméterlista mellett rendelkezhetnek változólistával (záradék) is, s ezek a
listák több sorba vannak tördelve, a fenti szabályok szerint.

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

Ne feledjük, hogy a formázási szabályok akkor is érvényesek, ha a névtelen
függvény egy másik függvény vagy metódus argumentumaként kerül meghívásra, mint az
alábbi példában:

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

## 8. Névtelen osztályok

A névtelen osztályoknak ugyanazon fentebb kifejtett irányelveket KELL követniük,
mint a névtelen függvényeknek.

~~~php
<?php

$instance = new class {};
~~~

A nyitó kapcsos zárójel abban az esetben LEHET ugyan abban a sorban, mint a `class`
kulcsszó, ha az `implements` után felsorolt interfészek egy sorban is
elférnek (nincs sortörés). Ha az interfészek listája több sorban helyezkedik el,
akkor a nyitó kapcsos zárójelnek az utolsó felsorolt interfész után következő
sorban KELL lennie.

~~~php
<?php

// Zárójel ugyanabban a sorban
$instance = new class extends \Foo implements \HandleableInterface {
    // Class content
};

// Zárójel a következő sorban
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
[RFC 2119]: ../related-rfcs/2119.md
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
