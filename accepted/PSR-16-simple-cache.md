[Kezdőlap](../README.md)

# Gyorsítótár programkönyvtárak közös interfésze

Ez a dokumentum egy egyszerű, de bővíthető felületet ír le a gyorsítótár elem és
a gyorsítótár illesztő számára.

A csupa nagybetűvel szedett, a követelmények szintjének jelzésére szolgáló kiemelt
kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

A végső megvalósítók TETSZŐLEGESEN díszíthetik az objektumokat a javasoltnál több
funkcionalitással, de először a előírt interfészeket/funkcionalitást KELL megvalósítaniuk.

## 1. Specifikáció

### 1.1 Bevezetés

A [gyorsítótárazás/cache](https://www.letscode.hu/2015/02/05/cache-avagy-dugikeszletek)
bevett mód bármely projekt teljesítménynövelésére, a különböző gyorsítótár
megoldások számos keretrendszer és függvénykönyvtár legáltalánosabb szolgáltatásai.

Az interoperábilitás ezen a szinten azt jelenti, hogy a programkönyvtárak dobhatják
saját gyorsítótár implementációjukat és könnyedén támaszkodhatnak a keretrendszerre
vagy más gyorsítótárnak szánt könyvtárra.

A PSR-6 már megoldotta ezt a problémát, de meglehetősen formális és bőbeszédű módon,
még a legegyszerűbb használati esetknél is. Jelen PSR célja, hogy egyszerűbb,
áramvonalasabb felületet építsen a gyakori esetekhez. Független a PSR-6 szabványtól,
de úgy tervezték, hogy a PSR-6-al való kompatibilitás a lehető legegyszerűbb legyen.

### 1.2 Alapfogalmak

A klienskód, a szolgáltató programkönyvtár, az élettartam, a lejárat és a kulcs
fogalmának meghatározása megegyezik a PSR-6 szabványban foglaltakkal.

*    **Hívó vagy kliens kód** - Az a programkönyvtár vagy kód, amelynek ténylegesen
gyorsítótár szolgáltatásra van szüksége. Ez a kód fogja használni az ezen szabványos
interfészt megvalósító gyorsítótár szolgáltatást, de egyébként nem lesz tudomása
eme szolgáltatások megvalósításáról.

*    **Megvalósító vagy szolgáltató programkönyvtár** - Ez a programkönyvtár felelős
a jelen szabvány megvalósításáért és azért, hogy biztosítsa a gyorsítótárazó szolgáltatást
a hívó vagy kliens kód számára. A szolgáltató programkönyvtárnak KELL biztosítania
azokat az osztályokat, amelyek implementálják a `Psr\SimpleCache\CacheInterface`
interfészt. A szolgáltató programkönyvtáraknak támogatniuk KELL legalább a TTL
(élettartam, részletek alább) funkcionalitást.

*    **Élettartam** - Az egyes elemek élettartama (TTL) az az időtartam, amíg az
adott elem tárolva van a gyorsítótárban és stabilnak tekinthető (nem változik).
Az élettartamot általában egy másodperceket reprezentáló egész számmal szokás
megadni, vagy egy rögzített időtartamot tároló
[DateInterval](https://www.php.net/manual/en/class.dateinterval.php) objektummal.

*    **Lejárat** - Az a tényleges időpont, amikor egy elem lejár. Ezt rendszerint
úgy számítják ki, hogy az elem gyorsítótárba helyezésének időpontjához hozzáadják
az élettartamot (TTL), de kifejezetten beállítható a DateTime objektummal is.

Ha egy elem élettartamát 300 másodpercre állítjuk és 1:30:00 időpontban helyezzük a
gyorsítótárba, akkor 1:35:00-kor fog lejárni.

A szolgáltató programkönyvtáraknak LEHET egy elemet lejárttá tenni a kért lejárati
idő előtt is, de a lejárati idő elérése után lejártként KELL kezelniük minden elemet.
Ha a hívó kód kéri, hogy mentsen egy konkrét elemet, de nincs meghatározva lejárati
idő, illetve az vagy az élettartam értéke `null`, akkor a szolgáltató programkönyvtárnak
LEHET egy alapértelmezett időtartamot alkalmazni. Ha nincs beállítva alapértelmezett
időtartam, akkor a szolgáltató programkönyvtárnak ezt úgy kell értelmezni, hogy
az adott elem gyorsítótárazására irányuló kérelem örök időkre szól, vagy legalábbis
addig, ameddig a mögöttes implementáció támogatja.

Ha negatív vagy nulla lejárati idő van megadva, akkor az elemet (ha létezik) törlöni
KELL a gyorsítótárból, mert már lejárt.

*    **Kulcs** - Legalább egy karakterből álló szöveg, amely egyedileg azonosítja
a gyorstárazott elemet. A szolgáltató programkönyvtáraknak támogatniuk KELL az
alfanumerikus karaktereket, aláhúzást és pontot bármilyen sorrendben tartalmazó,
UTF-8 kódolású kulcsokat, melyek hossza nem haladja meg a 64 karaktert.
A szolgáltató programkönyvtárak TETSZŐLEGESEN támogathatnak olyan kulcsokat is,
amelyek a felsoroltakon kívül más karaktereket is tartalmaznak, nem UTF-8 a kódolásuk,
vagy hosszabbak 64 karakternél, de csak a minimálisan elvárt követelmények teljesítésén felül.
A szolgáltató programkönyvtárak felelősek szöveges kulcsaik megfelelő védőkarakterezéséért
(szép magyar szóval: eszképelés), de képesnek KELL lenniük arra is, hogy
visszaadják az eredeti, módosítatlan szöveges kulcsot. A következő karakterek fenn
vannak tartva a szabvány jövendő kiterjesztései céljaira, ezért a szolgáltató
programkönyvtáraknak NEM SZABAD támogatni a használatukat: `{}()/\@:`

*    **Gyorsítótár (Cache)** - olyan objektum, amely implemetálja a `Psr\SimpleCache\CacheInterface`
interfészt.

*    **Gyorsítótár hiányok** - A gyorsítótár hiány null értékkel fog visszatérni
és ezért jelzi, hogy a `null` érték tárolása nem lehetséges. Ez a fő eltérés a PSR-6
feltételeihez képest.

### 1.3 Gyorsítótár

Az implementációknak LEHET biztosítani egy olyan mechanizmust, amely segítségével
a felhasználó beállíthatja az alapértelmezett lejárati időt, ha ez nincs beállítva
az adott gyorsítótár elemhez. Ha nincs a felhasználó által megadott alapértelmezett
lejárati idő, akkor az implementációnak KELL meghatároznia a mögöttes implementáció
által megengedett értéket. Ha a mögöttes implemetáció nem támogatja a lejárati időt,
akkor a felhasználó által beállított lejárati időt csendben figyelmenkívül KELL hagyni.

### 1.4 Adat

A szolgáltató programkönyvtáraknak támogatniuk KELL az összes
[szerializálható](http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=10#section_3_19)
PHP adattípust, különösen a következőket:

*    **Karakterlánc** (string) - Tetszőleges méretű karakterláncok, bármely PHP-kompatibilis
karakterkódolással.
*    **Egész szám** (integer) - Bármilyen méretű, akár 64 biten ábrázolt a PHP által
támogatott előjeles egész szám.
*    **Lebegőpontos szám** (float) - Tizedes ponttal jelölt előjeles tört szám.
*    **Logikai érték** (boolean) - Igaz (true) és hamis (false).
*    **Null** - Null érték (bár nem lesz megkülönböztethető a gyorsítótár hiánytól).
*    **Arrays** - Indexed, associative and multidimensional arrays of arbitrary depth.
*    **Tömb** - Numerikusan vagy szövegesen indexelt, többdimenziós tömb, tetszőleges mélységben.
*    **Objektum** - Bármely objektum, amely támogatja a veszteségmentes szerializációt
és deszerializációt, oly módon, hogy `$objektum == unserialize(serialize($objektum))`
kifejezés igaz legyen. Az objektumoknak ki LEHET használni a PHP [Serializable interfésze](https://www.php.net/manual/en/class.serializable.php)
vagy a `__sleep()` illetve `__wakeup()` mágikus metódusok, esetleg más nyelvi eszközök
által kínált funkcionalitást.

Minden a szolgáltató programkönyvtárnak átadott adatot úgy KELL visszaadni, ahogy
átadásra került, különös tekintettel a változók típusára. Tehát ha a visszatérési
érték ugyan 5, de a típusa szöveg, annak ellenére, hogy egész számként volt eltárolva,
akkor az hibának számít. A szolgáltató programkönyvtáraknak LEHET alkalmazni a PHP
[serialize()](https://www.php.net/manual/en/function.serialize.php) és [unserialize()](https://www.php.net/manual/en/function.unserialize.php) függvényeit,
de nem kötelesek erre. A kompatibilitás érdekében egyszerűen csak alapértékként
kell használni az elfogadható objektum értékeket.

Ha bármilyen okból nem lehetséges a mentett érték pontos visszaadása, akkor a
jelen PSR-t megvalósító programkönyvtáraknak inkább gyorsítótár-hiánnyal KELL
visszatérniük, mint az érvénytelen adattal.

## 2. Interfészek

### 2.1 CacheInterface

A gyorsítótár interfész meghatározza a gyorsítótár-bejegyzések gyűjteményén végezhető
legalapvetőbb műveleteket, melyek magában foglalják az olvasást, írást és az elemek
törlését.

Továbbá rendelkezik a gyorsítótár bejegyzések többrészes gyüjteményét is kezelő
metódusokkal, mint például a több gyorstár-bejegyzést egyszerre író, olvasó vagy
törlő metódusok. Ez akkor hasznos, amikor sok cache olvasást/írást kell végrehajtani,
mert lehetővé teszi hogy a műveleteket a gyorsítótár-kiszolgáló egyetlen hívásával
lehessen végrehajtani, jelentősen lecsökkentve a várakozási időt.

A CacheInterface egy példánya megfelel egy gyorsítótár-elem gyüjteménynek, egykulcsos
névtérrel és egyenértékű a PSR-6 [Cache\CacheItemPoolInterface](PSR-6-cache.md#cacheitempoolinterface)-el.
A különböző CacheInterface példányoknak LEHET ugyanaz az adatforrásuk, viszont
logikailag függetlennek KELL lenniük egymástól.


~~~php
<?php

namespace Psr\SimpleCache;

interface CacheInterface
{
    /**
     * Lekér egy értéket a gyorsítótárból.
     *
     * @param string $key     Az elemet a cache-ben azonosító egyedi kulcs.
     * @param mixed  $default Alapértelmezett visszatérési érték, ha a kulcs nem létezik.
     *
     * @return mixed Az elem értéke a cache-ből, vagy $default gyorsítótár-hiány esetén.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   Akkor KELL dobnia, ha a $key string nem érvényes érték.
     */
    public function get($key, $default = null);

    /**
     * Adatok megőrzése a cache-ben, opcionális lejárati idővel (TTL) rendelkező egyedi kulccsal.
     *
     * @param string                 $key   A tárolandó elem egyedi kulcsa.
     * @param mixed                  $value A tárolandó elem értéke. Szerializálhatónak kell lennie.
     * @param null|int|\DateInterval $ttl   Opcionális. Az elem lejárati ideje. Ha nincs érték megadva és
     *                                      az illesztő támogatja a lejárati időt, akkor a könyvtár beállíthat
     *                                      egy alapértelmezett értéket hozzá, vagy az illesztő
     *                                      gondoskodik róla.
     *
     * @return bool True siker, false kudarc estenén.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   Akkor KELL dobnia, ha a $key string nem érvényes érték.
     */
    public function set($key, $value, $ttl = null);

    /**
     * A megadott egyedi kulccsal azonosított elem törlése a gyorsítótárból.
     *
     * @param string $key A törlendő gyorsítótár-elem egyedi kulcsa.
     *
     * @return bool True, ha az elem eltávolítása sikeres volt. False, ha valami hiba történt.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   Akkor KELL dobnia, ha a $key string nem érvényes érték.
     */
    public function delete($key);

    /**
     * Az egész gyorsítótár törlése.
     *
     * @return bool True siker, false kudarc estenén.
     */
    public function clear();

    /**
     * Több gyorsítótár-elem egyidejű lekérdezése a kulcsaik által.
     *
     * @param iterable $keys    Egy műveletben beszrzendő kulcsok listája.
     * @param mixed    $default Alapértelmezett visszatérési érték, ha a ok nem léteznek.
     *
     * @return iterable Egy kulcs => érték párokból álló lista. A nem létező, vagy
     *   elavult gyorsítótár-kulcsok értéke $default lesz.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   Akkor KELL eldobni, ha $keys típusa nem tömb, sem Traversable,
     *   vagy ha a $keys bármelyik eleme nem érvényes érték.
     */
    public function getMultiple($keys, $default = null);

    /**
     * Kulcs => érték párok megőrzése a cache-ben, opcionális lejárati idővel (TTL).
     *
     * @param iterable               $values Egy kulcs => érték párokból álló lista a többszörös beállítási művelethez.
     * @param null|int|\DateInterval $ttl    Opcionális. Az elem lejárati ideje. Ha nincs érték megadva és
     *                                       az illesztő támogatja a lejárati időt, akkor a könyvtár beállíthat
     *                                       egy alapértelmezett értéket hozzá, vagy az illesztő
     *                                       gondoskodik róla.
     *
     * @return bool True siker, false kudarc estenén.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   Akkor KELL eldobni, ha $values típusa nem tömb, sem Traversable,
     *   vagy ha a $values bármelyik eleme nem érvényes érték.
     */
    public function setMultiple($values, $ttl = null);

    /**
     * Több cache-elem egyetlen műveletben történő törlése.
     *
     * @param iterable $keys A törlendő elemek kulcsainak szöveges alapú listája.
     *
     * @return bool True, ha az elemek eltávolítása sikeres volt. False, ha valami hiba történt.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   Akkor KELL eldobni, ha $keys típusa nem tömb, sem Traversable,
     *   vagy ha a $keys bármelyik eleme nem érvényes érték.
     */
    public function deleteMultiple($keys);

    /**
     * Meghatározza, hogy egy elem jelen van-e a gyorsítótárban.
     *
     * Megjegyzés: Ajánlatos, hogy a has() csak cache előmelegítésére legyen használva
     * és ne alkalmazzuk éles alakalmazásban a get/set műveletekhez. Mivel ez a metódus
     * amikor true-val tér vissza, majd közvetlenül ez után egy másik script esetleg
     * eltávolítja az imént megtalált elemet, ez olyan versenyhelyzethez vezet, ami
     * az alkalmazás állapotát elavultá teheti.
     *
     * @param string $key A keresett elem kulcsa.
     *
     * @return bool
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   Akkor KELL dobnia, ha a $key string nem érvényes érték.
     */
    public function has($key);
}
~~~

### 2.2 CacheException

~~~php

<?php

namespace Psr\SimpleCache;

/**
 * Az összes olyan kivételtípus őse, amelyet a megvalósító könyvtár dobhat.
 */
interface CacheException
{
}
~~~

### 2.3 InvalidArgumentException

~~~php
<?php

namespace Psr\SimpleCache;

/**
 * Kivétel interfész érvénytelen gyorsítótár paraméterekhez.
 *
 * Amikor egy érvénytelen argumentum kerül átadásra, a jelen interfészt megvalósítóknak
 * ilyen kivételt kell dobni.
 */
interface InvalidArgumentException extends CacheException
{
}
~~~

[Kezdőlap](../README.md)
