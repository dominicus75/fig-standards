[Kezdőlap](../README.md)

# Gyorsítótár interfész

A [gyorsítótárazás/cache](https://www.letscode.hu/2015/02/05/cache-avagy-dugikeszletek)
bevett mód bármely projekt teljesítménynövelésének, a különböző gyorsítótár
megoldások számos keretrendszer és függvénykönyvtár legáltalánosabb szolgáltatásai.
Ez olyan helyzetet eredményezett, ahol számos projekt görgeti maga előtt saját,
a funkcionalitás változatos szintjeit megtestesítő cache függvénykönyvtárait.
Ezen különbözőségek okán a fejlesztőknek több rendszert is el kellett sajátítaniuk,
hogy biztosítsák azt a funkcionalitást, amelyre szükségük volt. E mellett maguknak
a gyorsítótárazó függvénykönyvtárak fejlesztőinek is szembesülniük kellett a
választással a között, hogy kizárólag korlátozott számú keretrendszert támogassanak
vagy hozzanak létre nagy számú adapter osztályt, hogy munkájukat minél szélesebb
körben alkalmazhassák.

A gyorsítótárazó rendszerek számára készített közös programozási felület megoldhatja
ezt a problémát. A különböző programkönyvtárak és keretrendszerek fejlesztői
számíthatnak arra, hogy a gyorsítótárazó rendszerek az elvárt módon fognak működni,
míg ezen rendszerek fejlesztőinek csupán egyetlen programozási felületet kell
megvalósítani, adapterek sokaságának megírása helyett.

A csupa nagybetűvel szedett kiemelt kulcsszavak ebben a leírásban az
[RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

## Cél

Ezen PSR célja, hogy lehetővé tegye a fejlesztők számára olyan gyorsítótárazó
programkönyvtárak készítését, amelyek egyedi fejlesztés nélkül integrálhatók a már
létező keretrendszerekbe és projektekbe.

## Alapfogalmak

*    **Hívó vagy kliens kód** - Az a programkönyvtár vagy kód, amelynek ténylegesen
gyorsítótár szolgáltatásra van szüksége. Ez a kód fogja használni az ezen szabványos
interfészt megvalósító gyorsítótár szolgáltatást, de egyébként nem lesz tudomása
eme szolgáltatások megvalósításáról.

*    **Megvalósító vagy szolgáltató programkönyvtár** - Ez a programkönyvtár felelős
a jelen szabvány megvalósításáért és azért, hogy biztosítsa a gyorsítótárazó szolgáltatást
a hívó vagy kliens kód számára. A szolgáltató programkönyvtárnak KELL biztosítania
azokat az osztályokat, amelyek implementálják a `Cache\CacheItemPoolInterface` és
a `Cache\CacheItemInterface` interfészeket. A szolgáltató programkönyvtáraknak
támogatniuk KELL legalább a TTL (lejárati idő, részletek alább) funkcionalitást.

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

*    **Találat** - Találatról (hit) akkor beszélünk a gyorsítótárakkal kapcsolatban,
amikor a hívó vagy kliens kód kér egy olyan egyedi kulccsal azonosított elemet, amely
megtalálható a gyorsítótárban a kért kulcs alatt, illetve, ha ez az elem még nem
járt le, vagy nem érvénytelen bármilyen más okból. A hívó programkönyvtáraknak erről
AJÁNLOTT meggyőződni az `isHit()` metódus meghívásával, minden `get()` hívás esetén.

*    **Hiány** - A hiány (miss) meglepő módon az imént tárgyalt találat (hit) ellentéte.
Akkor áll elő, ha a hívó vagy kliens kód kér egy olyan egyedi kulccsal azonosított elemet,
amely vagy egyáltalán nem található a gyorsítótárban az adott kulcs alatt, vagy
létezik ugyan, de már lejárt, esetleg bármilyen más okból érvénytelen. A lejárt vagy
érvénytelen elemeket minden esetben gyorsítótár hiányként KELL figyelembe venni.

*    **Késleltetett** - A deferred cache save indicates that a cache item may not be
persisted immediately by the pool. A Pool object MAY delay persisting a deferred
cache item in order to take advantage of bulk-set operations supported by some
storage engines. A Pool MUST ensure that any deferred cache items are eventually
persisted and data is not lost, and MAY persist them before a Calling Library
requests that they be persisted. When a Calling Library invokes the commit()
method all outstanding deferred items MUST be persisted. An Implementing Library
MAY use whatever logic is appropriate to determine when to persist deferred
items, such as an object destructor, persisting all on save(), a timeout or
max-items check or any other appropriate logic. Requests for a cache item that
has been deferred MUST return the deferred but not-yet-persisted item.

## Adat

A szolgáltató programkönyvtáraknak támogatniuk KELL az összes [szerializálható](http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=10#section_3_19)
PHP adattípust, különösen a következőket:

*    **Karakterlánc** (string) - Tetszőleges méretű karakterláncok, bármely PHP-kompatibilis
karakterkódolással.
*    **Egész szám** (integer) - All integers of any size supported by PHP, up to 64-bit signed.
*    **Lebegőpontos szám** (float) - Tizedes ponttal jelölt tört szám.
*    **Logikai érték** (boolean) - Igaz (true) és hamis (false).
*    **Null** - Tényleges null érték.
*    **Tömb** - Numerikusan vagy szövegesen indexelt, többdimenziós tömb, tetszőleges mélységben.
*    **Objektum** - Any object that supports lossless serialization and
deserialization such that $o == unserialize(serialize($o)). Objects MAY
leverage PHP's Serializable interface, `__sleep()` or `__wakeup()` magic methods,
or similar language functionality if appropriate.

All data passed into the Implementing Library MUST be returned exactly as
passed. That includes the variable type. That is, it is an error to return
(string) 5 if (int) 5 was the value saved.  Implementing Libraries MAY use PHP's
serialize()/unserialize() functions internally but are not required to do so.
Compatibility with them is simply used as a baseline for acceptable object values.

If it is not possible to return the exact saved value for any reason, implementing
libraries MUST respond with a cache miss rather than corrupted data.

## Kulcsfogalmak

### Gyűjtő (pool)

The Pool represents a collection of items in a caching system. The pool is
a logical Repository of all items it contains.  All cacheable items are retrieved
from the Pool as an Item object, and all interaction with the whole universe of
cached objects happens through the Pool.

### Elemek (items)

An Item represents a single key/value pair within a Pool. The key is the primary
unique identifier for an Item and MUST be immutable. The Value MAY be changed
at any time.

## Hibakezelés

While caching is often an important part of application performance, it should never
be a critical part of application functionality. Thus, an error in a cache system SHOULD NOT
result in application failure.  For that reason, Implementing Libraries MUST NOT
throw exceptions other than those defined by the interface, and SHOULD trap any errors
or exceptions triggered by an underlying data store and not allow them to bubble.

An Implementing Library SHOULD log such errors or otherwise report them to an
administrator as appropriate.

If a Calling Library requests that one or more Items be deleted, or that a pool be cleared,
it MUST NOT be considered an error condition if the specified key does not exist. The
post-condition is the same (the key does not exist, or the pool is empty), thus there is
no error condition.

## Interfészek

### CacheItemInterface

CacheItemInterface defines an item inside a cache system.  Each Item object
MUST be associated with a specific key, which can be set according to the
implementing system and is typically passed by the Cache\CacheItemPoolInterface
object.

The Cache\CacheItemInterface object encapsulates the storage and retrieval of
cache items. Each Cache\CacheItemInterface is generated by a
Cache\CacheItemPoolInterface object, which is responsible for any required
setup as well as associating the object with a unique Key.
Cache\CacheItemInterface objects MUST be able to store and retrieve any type of
PHP value defined in the Data section of this document.

Calling Libraries MUST NOT instantiate Item objects themselves. They may only
be requested from a Pool object via the getItem() method.  Calling Libraries
SHOULD NOT assume that an Item created by one Implementing Library is
compatible with a Pool from another Implementing Library.

~~~php
<?php

namespace Psr\Cache;

/**
 * CacheItemInterface defines an interface for interacting with objects inside a cache.
 */
interface CacheItemInterface
{
    /**
     * Returns the key for the current cache item.
     *
     * The key is loaded by the Implementing Library, but should be available to
     * the higher level callers when needed.
     *
     * @return string
     *   The key string for this cache item.
     */
    public function getKey();

    /**
     * Retrieves the value of the item from the cache associated with this object's key.
     *
     * The value returned must be identical to the value originally stored by set().
     *
     * If isHit() returns false, this method MUST return null. Note that null
     * is a legitimate cached value, so the isHit() method SHOULD be used to
     * differentiate between "null value was found" and "no value was found."
     *
     * @return mixed
     *   The value corresponding to this cache item's key, or null if not found.
     */
    public function get();

    /**
     * Confirms if the cache item lookup resulted in a cache hit.
     *
     * Note: This method MUST NOT have a race condition between calling isHit()
     * and calling get().
     *
     * @return bool
     *   True if the request resulted in a cache hit. False otherwise.
     */
    public function isHit();

    /**
     * Sets the value represented by this cache item.
     *
     * The $value argument may be any item that can be serialized by PHP,
     * although the method of serialization is left up to the Implementing
     * Library.
     *
     * @param mixed $value
     *   The serializable value to be stored.
     *
     * @return static
     *   The invoked object.
     */
    public function set($value);

    /**
     * Sets the expiration time for this cache item.
     *
     * @param \DateTimeInterface|null $expiration
     *   The point in time after which the item MUST be considered expired.
     *   If null is passed explicitly, a default value MAY be used. If none is set,
     *   the value should be stored permanently or for as long as the
     *   implementation allows.
     *
     * @return static
     *   The called object.
     */
    public function expiresAt($expiration);

    /**
     * Sets the expiration time for this cache item.
     *
     * @param int|\DateInterval|null $time
     *   The period of time from the present after which the item MUST be considered
     *   expired. An integer parameter is understood to be the time in seconds until
     *   expiration. If null is passed explicitly, a default value MAY be used.
     *   If none is set, the value should be stored permanently or for as long as the
     *   implementation allows.
     *
     * @return static
     *   The called object.
     */
    public function expiresAfter($time);

}
~~~

### CacheItemPoolInterface

The primary purpose of Cache\CacheItemPoolInterface is to accept a key from the
Calling Library and return the associated Cache\CacheItemInterface object.
It is also the primary point of interaction with the entire cache collection.
All configuration and initialization of the Pool is left up to an Implementing
Library.

~~~php
<?php

namespace Psr\Cache;

/**
 * CacheItemPoolInterface generates CacheItemInterface objects.
 */
interface CacheItemPoolInterface
{
    /**
     * Returns a Cache Item representing the specified key.
     *
     * This method must always return a CacheItemInterface object, even in case of
     * a cache miss. It MUST NOT return null.
     *
     * @param string $key
     *   The key for which to return the corresponding Cache Item.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return CacheItemInterface
     *   The corresponding Cache Item.
     */
    public function getItem($key);

    /**
     * Returns a traversable set of cache items.
     *
     * @param string[] $keys
     *   An indexed array of keys of items to retrieve.
     *
     * @throws InvalidArgumentException
     *   If any of the keys in $keys are not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return array|\Traversable
     *   A traversable collection of Cache Items keyed by the cache keys of
     *   each item. A Cache item will be returned for each key, even if that
     *   key is not found. However, if no keys are specified then an empty
     *   traversable MUST be returned instead.
     */
    public function getItems(array $keys = array());

    /**
     * Confirms if the cache contains specified cache item.
     *
     * Note: This method MAY avoid retrieving the cached value for performance reasons.
     * This could result in a race condition with CacheItemInterface::get(). To avoid
     * such situation use CacheItemInterface::isHit() instead.
     *
     * @param string $key
     *   The key for which to check existence.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if item exists in the cache, false otherwise.
     */
    public function hasItem($key);

    /**
     * Deletes all items in the pool.
     *
     * @return bool
     *   True if the pool was successfully cleared. False if there was an error.
     */
    public function clear();

    /**
     * Removes the item from the pool.
     *
     * @param string $key
     *   The key to delete.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if the item was successfully removed. False if there was an error.
     */
    public function deleteItem($key);

    /**
     * Removes multiple items from the pool.
     *
     * @param string[] $keys
     *   An array of keys that should be removed from the pool.

     * @throws InvalidArgumentException
     *   If any of the keys in $keys are not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if the items were successfully removed. False if there was an error.
     */
    public function deleteItems(array $keys);

    /**
     * Persists a cache item immediately.
     *
     * @param CacheItemInterface $item
     *   The cache item to save.
     *
     * @return bool
     *   True if the item was successfully persisted. False if there was an error.
     */
    public function save(CacheItemInterface $item);

    /**
     * Sets a cache item to be persisted later.
     *
     * @param CacheItemInterface $item
     *   The cache item to save.
     *
     * @return bool
     *   False if the item could not be queued or if a commit was attempted and failed. True otherwise.
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * Persists any deferred cache items.
     *
     * @return bool
     *   True if all not-yet-saved items were successfully saved or there were none. False otherwise.
     */
    public function commit();
}
~~~

### CacheException

This exception interface is intended for use when critical errors occur,
including but not limited to *cache setup* such as connecting to a cache server
or invalid credentials supplied.

Any exception thrown by an Implementing Library MUST implement this interface.

~~~php
<?php

namespace Psr\Cache;

/**
 * Exception interface for all exceptions thrown by an Implementing Library.
 */
interface CacheException
{
}
~~~

### InvalidArgumentException

~~~php
<?php

namespace Psr\Cache;

/**
 * Exception interface for invalid cache arguments.
 *
 * Any time an invalid argument is passed into a method it must throw an
 * exception class which implements Psr\Cache\InvalidArgumentException.
 */
interface InvalidArgumentException extends CacheException
{
}
~~~

[Kezdőlap](../README.md)
