[Kezdőlap](../README.md)

# Gyorsítótár interfész

A [gyorsítótárazás/cache](https://www.letscode.hu/2015/02/05/cache-avagy-dugikeszletek)
bevett mód bármely projekt teljesítménynövelésére, a különböző gyorsítótár
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
támogatniuk KELL legalább a TTL (élettartam, részletek alább) funkcionalitást.

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
megtalálható a gyorsítótárban a kért kulcs alatt és ez az elem még nem járt le,
vagy nem érvénytelen bármilyen más okból. A hívó programkönyvtáraknak erről
AJÁNLOTT meggyőződni az `isHit()` metódus meghívásával, minden `get()` hívás esetén.

*    **Hiány** - A hiány (miss) meglepő módon az imént tárgyalt találat ellentéte.
Akkor áll elő, ha a hívó vagy kliens kód kér egy olyan egyedi kulccsal azonosított elemet,
amely vagy egyáltalán nem található a gyorsítótárban az adott kulcs alatt, vagy
létezik ugyan, de már lejárt, esetleg bármilyen más okból érvénytelen. A lejárt vagy
érvénytelen elemeket minden esetben gyorsítótár hiányként KELL figyelembe venni.

*    **Halasztott** - Egy halasztott gyorsítótár-mentés azt jelzi, hogy egy elem
nem áll rendelkezésre azonnal a gyűjtőben. A gyűjtő objektumnak LEHET késleltetni
egy ilyen halasztott cache-elemet azért, hogy kihasználhassa a tárolómotorok által
támogatott tömeges műveletek előnyeit. A gyűjtőnek biztosítania KELL, hogy bármely
halasztott cache-elem végig megmaradjon és az adat ne vesszen el, illetve álljon
rendelkezésre mindaddig, amíg a hívó kód ezt kéri. Amikor a kliens kód meghívja a
`commit()` metódust, minden fennálló halasztott elemet meg KELL tartani.
Egy szolgáltató programkönyvtár TETSZŐLEGESEN alkalmazhat bármilyen megfelelő
logikát arra, hogy meghatározza, mikor maradjanak meg a halasztott elemek. Ilyen
megoldás lehet egy objektum destruktor, vagy annak kikötése, hogy maradjon meg
minden mentéskor, illetve egy időkorlát, esetleg az elemek maximális számának
ellenőrzése, vagy bármely más alkalmas logika. A halasztott gyorsítótár elemek
iránti kérelmeknek vissza KELL adniuk a halasztott, de még fenntartott elemet.

## Adat

A szolgáltató programkönyvtáraknak támogatniuk KELL az összes [szerializálható](http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=10#section_3_19)
PHP adattípust, különösen a következőket:

*    **Karakterlánc** (string) - Tetszőleges méretű karakterláncok, bármely PHP-kompatibilis
karakterkódolással.
*    **Egész szám** (integer) - Bármilyen méretű, akár 64 biten ábrázolt a PHP által támogatott egész szám.
*    **Lebegőpontos szám** (float) - Tizedes ponttal jelölt tört szám.
*    **Logikai érték** (boolean) - Igaz (true) és hamis (false).
*    **Null** - Tényleges null érték.
*    **Tömb** - Numerikusan vagy szövegesen indexelt, többdimenziós tömb, tetszőleges mélységben.
*    **Objektum** - Bármely objektum, amely támogatja a veszteségmentes szerializációt
és deszerializációt, oly módon, hogy `$objektum == unserialize(serialize($objektum))`
kifejezés igaz legyen. Az objektumoknak ki LEHET használni a PHP [Serializable interfésze](https://www.php.net/manual/en/class.serializable.php) vagy a `__sleep()`
illetve `__wakeup()` mágikus metódusok, esetleg más nyelvi eszközök által kínált
funkcionalitást.

Minden a szolgáltató programkönyvtárnak átadott adatot úgy KELL visszaadni, ahogy
átadásra került, különös tekintettel a változók típusára. Tehát ha a visszatérési
érték ugyan 5, de a típusa szöveg, annak ellenére, hogy egész számként volt eltárolva,
akkor az hibának számít. A szolgáltató programkönyvtáraknak LEHET alkalmazni a PHP
[serialize()](https://www.php.net/manual/en/function.serialize.php) és [unserialize()](https://www.php.net/manual/en/function.unserialize.php) függvényeit,
de nem kötelesek erre. A kompatibilitás érdekében egyszerűen csak alapértékként
kell használni az elfogadható objektum értékeket.

Ha bármilyen okból nem lehetséges a mentett érték pontos visszaadása, akkor a
jelen PSR-t megvalósító programkönyvtáraknak inkább gyorsítótár-hiánnyal (miss -
lásd fentebb) KELL visszatérniük, mint az érvénytelen adattal.

## Kulcsfogalmak

### Gyűjtő

A gyűjtő (pool) a gyorsítótár rendszerekben tárolt elemek gyűjteménye, egy minden
elemet magában foglaló logikai tároló. Minden gyorsítótárazható elem Item objektumként
lekérhető belőle és a gyorstárazott objektumokkal zajló összes interakció
a gyűjtőn keresztül történik.

### Elemek

Egy elem (item) reprezentálja az egyes kulcs/érték párokat a gyűjtőn belül. A kulcs
az elem elsődleges egyedi azonosítója, ezért állandónak KELL lennie. Az érték ezzel
szemben bármikor TETSZŐLEGESEN változtatható.

## Hibakezelés

Amíg a gyorsítótárazás gyakran az alkalmazás teljesítményének fontos része,
soha nem lehet kritikus része a funkcionalitásának. Ennél fogva egy gyorsítótárban
felmerülő hibának NEM KELLENE az egész alkalmazást működésképtelenné tenni. Ennek okán
a jelen PSR-t megvalósító szolgáltató programkönyvtáraknak TILOS a jelen interfészben
meghatározottakon kívül más kivételt dobni, ezen felül el KELLENE kapniuk bármilyen
hibát vagy kivételt, amit egy mögöttes adattároló kiválthat. A szolgáltató
programkönyvtáraknak adott esetben AJÁNLOTT naplózni vagy más módon jelezni az
ilyen hibákat a rendszer-adminisztrátor számára.

Ha a hívó kód kérelmére egy vagy több elemet törölni, vagy az egész gyűjtőt
nullázni kell, akkor NEM SZABAD hibaként figyelembe venni azt, hogy a keresett kulcs
nem létezik. A végeredmény így is ugyan az lesz (a kulcs nem létezik vagy a gyűjtő üres),
a kérés teljesül, ezért ez nem tekinthető hibának.

## Interfészek

### CacheItemInterface

A CacheItemInterface egy elemet ír le a gyorsítótár rendszeren belül. Minden egyes
Item objektumot egy egyedi kulccsal KELL társítani, amit a megvalósító rendszer
szerint lehet beállítani és jellemzően egy Cache\CacheItemPoolInterface objektum
segítségével kerül átadásra.

A Cache\CacheItemInterface objektum magában foglalja a cache-elemek tárolását és
visszakeresését. Minden egyes Cache\CacheItemInterface objektum a
Cache\CacheItemPoolInterface segítségével jön létre, ami felelős minden szükséges
beállításért, továbbá az objektum egyedi kulcshoz társításáért.
A Cache\CacheItemInterface objektumoknak képesnek KELL lenni tárolni és visszaadni
bármilyen, a jelen dokumentum Adat-szekciójában meghatározott PHP-adattípust.

Maguknak a hívó vagy kliens kódoknak NEM SZABAD példányosítani az Item objektumot,
csak kérhetik ezt a gyűjtő (Pool) objektumtól a `getItem()` metóduson keresztül. A hívó
kódoknak NEM KELLENE azt feltételezni, hogy a szolgáltató programkönyvtár által
létrehozott Item objektum kompatibilis más hasonló programkönyvtárak gyűjtő (Pool)
objektumaival.


~~~php
<?php

namespace Psr\Cache;

/**
 * A CacheItemInterface leír egy felületet, amelyen keresztül a gyorsítótárban lévő
 * objektumok kommunikálhatnak egymással.
 */
interface CacheItemInterface
{
    /**
     * Az aktuális cache-elem kulcsával tér vissza.
     *
     * A szolgáltató/megvalósító könyvtár által betöltött kulcs, aminek rendelkezésre
     * kell állnia a magasabb szintű hívók számára is, ha szükséges.
     *
     * @return string - A szöveges kulcs ehhez a cache-elemhez.
     *
     */
    public function getKey();

    /**
     *
     * Visszaadja a jelen objektum kulcsához társított elem értékét a gyorstárból.
     *
     * A visszaadott értéknek meg kell egyezni a set() metódussal beállított értékkel.
     *
     * Ha az isHit() false-al tér vissza, akkor ennek a metódusnak null-al KELL.
     * Megjegyzendő, hogy a null megengedetten tárolt érték, így az isHit()
     * metódusnak különbséget KELLENE tenni a között, hogy null értéket talált,
     * vagy egyáltalán nem talált értéket a kért kulcs alatt.
     *
     * @return mixed Jelen cache elem kulcsának megfelelő érték, vagy null, ha ilyen nincs.
     *
     */
    public function get();

    /**
     * Megerősíti, hogy a gyorsítótár-lekérdezés találatot eredményezett.
     *
     * Megjegyzés: ennek a metódusnak nem szabad versenyhelyzetet eredményeznie
     * az isHit() és a get() hívások között.
     * @see https://hu.wikipedia.org/wiki/Versenyhelyzet#A_sz%C3%A1m%C3%ADt%C3%A1stechnik%C3%A1ban
     *
     * @return bool - true, ha a kérelem cache-találatot eredményezett, egyébként false.
     *
     */
    public function isHit();

    /**
     * Beállítja a jelen gyorsítótár-elem által reprezentált értéket.
     *
     * A $value argumentum lehet bármilyen PHP-val szerializálható elem, akkor is
     * ha a szerializáló metódus a megvalósító programkönyvtárban marad.
     *
     * @param mixed $value - a szerializálható érték, amit tárolni szeretnénk
     *
     * @return static - a hivatkozott objektum
     *
     */
    public function set($value);

    /**
     * Beállítja a jelen gyorsítótár-elemhez tartozó lejárati időpontot.
     *
     * @param \DateTimeInterface|null $expiration
     *   Az az időpont, ami után az elemet lejártként KELL figyelembe venni.
     *   Ha kifejezetten null kerül átadásra, akkor az alapértelmezett értéket LEHET
     *   használni. Ha nincs beállítva, akkor az értéket tartósan tárolni kell,
     *   vagy legalább addig, amíg az adott implementáció lehetővé teszi.
     *
     * @return static - a hívott objektum.
     *
     */
    public function expiresAt($expiration);

    /**
     * Beállítja a jelen gyorsítótár-elemhez tartozó lejárati időtartamot.
     *
     * @param int|\DateInterval|null $time
     *   Az a jelen pillanattól számított időtartam, miután az elemet lejártként
     *   KELL figyelembe venni. Ez alatt az az egész számmal kifejezett időparaméter
     *   értendő, amely azt az időt jelöli másodpercekben, amíg az elem lejár.
     *   Ha kifejezetten null kerül átadásra, akkor az alapértelmezett értéket LEHET
     *   használni. Ha nincs beállítva, akkor az értéket tartósan tárolni kell,
     *   vagy legalább addig, amíg az adott implementáció lehetővé teszi.
     *
     * @return static - a hívott objektum
     *
     */
    public function expiresAfter($time);

}
~~~

### CacheItemPoolInterface

A Cache\CacheItemPoolInterface elsődleges célja az, hogy fogadja a hívó kód által
küldött kulcsot és visszaadja a hozzá tartozó Cache\CacheItemInterface objektumot.
Ez a teljes gyorsítótár-kollekcióval történő interakció elsődleges pontja is.
A gyűjtő (Pool) objektumok összes konfigurációja és inicializációja a megvalósító
programkönyvtárra marad.

~~~php
<?php

namespace Psr\Cache;

/**
 * A CacheItemPoolInterface feladata, hogy CacheItemInterface objektumokat hozzon létre.
 */
interface CacheItemPoolInterface
{
    /**
     * Egy CacheItemInterface-t megvalósító objektummal tér vissza, amelyet a
     * paraméterként várt egyedi kulcs azonosít.
     *
     * Ennek a metódusnak mindig egy CacheItemInterface objektummal kell visszatérni,
     * még abban az esetben is, ha gyorsítótár-hiány (miss, lásd fent) áll fent.
     * Éppen ezért TILOS null értéket visszaadnia.
     *
     * @param string $key - a kért gyorsítótár-elemet azonosító egyedi kulcs.
     *
     * @throws InvalidArgumentException - ha a $key paraméter értéke érvénytelen,
     * akkor egy \Psr\Cache\InvalidArgumentException típusú kivételt KELL dobni.
     *
     * @return CacheItemInterface - a megfelelő cache-elem.
     *
     */
    public function getItem($key);

    /**
     * A gyorsítótár elemeinek bejárható adatkollekciójával tér vissza.
     * @see https://www.php.net/manual/en/class.traversable.php
     *
     * @param string[] $keys - a lekérendő elemek kulcsainak indexelt tömbje.
     *
     * @throws InvalidArgumentException - ha bármelyik kulcs értéke a paraméterként
     * átadott $keys tömbben érvénytelen, akkor egy \Psr\Cache\InvalidArgumentException
     * típusú kivételt KELL dobni.
     *
     * @return array|\Traversable
     *   A gyorsítótár elemeinek bejárható adatkollekciója, amelyet az egyes elemek
     *   gyorsítótár-kulcsai indexelnek. A gyorsítótár-elem minden kulccsal vissza
     *   fog térni, akkor is, ha kulcs nem található a tárban. Ha azonban nincsenek
     *   megadva kulcsok, akkor egy üres tömböt vagy bejárható objektumot KELL
     *   visszaadni helyette.
     */
    public function getItems(array $keys = array());

    /**
     * Megerősíti, hogy a gyorsítótár tartalmazza-e a kért elemet.
     *
     * Megjegyzés: a teljesítmény növelése érdekében ez a metódus, ha LEHET,
     * mellőzze a gyorstárazott érték visszaadását. Ez ugyanis versenyhelyzetet
     * eredményezhet CacheItemInterface::get() metódussal. Az ilyen szituációk
     * elkerülése érdekében ellenőrzésre csak a CacheItemInterface::isHit()
     * metódust használjuk.
     * @see https://hu.wikipedia.org/wiki/Versenyhelyzet#A_sz%C3%A1m%C3%ADt%C3%A1stechnik%C3%A1ban
     *
     * @param string $key - a kulcs, aminek a létezését vizsgáljuk
     *
     * @throws InvalidArgumentException - ha a $key paraméter értéke érvénytelen,
     * akkor egy \Psr\Cache\InvalidArgumentException típusú kivételt KELL dobni.
     *
     * @return bool - true, ha az elem létezik a gyorstárban, egyébként false
     *
     */
    public function hasItem($key);

    /**
     * Törli az összes elemet a gyűjtőből.
     *
     * @return bool - true, ha a gyűjtő sikeresen kiürítve, false, ha hiba történt.
     *
     */
    public function clear();

    /**
     * Eltávolítja a kért elemet a gyűjtőből.
     *
     * @param string $key - a törlendő elem kulcsa.
     *
     * @throws InvalidArgumentException - ha a $key paraméter értéke érvénytelen,
     * akkor egy \Psr\Cache\InvalidArgumentException típusú kivételt KELL dobni.
     *
     * @return bool - true, ha a kért elem sikeresen eltávolítva, false, ha hiba történt.
     *
     */
    public function deleteItem($key);

    /**
     * Eltávolít több elemet a gyűjtőből.
     *
     * @param string[] $keys - a gyűjtőből eltávolítandó kulcsok tömbbe foglalt felsorolása.
     *
     * @throws InvalidArgumentException - ha bármelyik kulcs értéke a paraméterként
     * átadott $keys tömbben érvénytelen, akkor egy \Psr\Cache\InvalidArgumentException
     * típusú kivételt KELL dobni.
     *
     * @return bool - true, ha a kért elemek sikeresen el lettek távolítva, false, ha hiba történt.
     *
     */
    public function deleteItems(array $keys);

    /**
     * A gyorsítótár-elem azonnali tartós tárolása.
     *
     * @param CacheItemInterface $item - a mentendő gyorsítótár-elem.
     *
     * @return bool - true, ha az elem sikeresen mentve, false, ha hiba történt.
     *
     */
    public function save(CacheItemInterface $item);

    /**
     * Gyorsítótár elem beállítása halasztott mentésre.
     *
     * @param CacheItemInterface $item - a mentendő gyorsítótár-elem.
     *
     * @return bool - false, ha az elem nem állítható sorba, vagy ha az eltárolást
     *   már sikertelenül megkísérelték. Egyébként true.
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * Bármely halasztott gyorsítótár-elem tartós tárolása.
     *
     * @return bool - true, ha az összes még el nem mentett elem mentése sikeres volt,
     *   vagy ha nem voltak ilyen elemek. Egyébként false.
     */
    public function commit();
}
~~~

### CacheException

Ezt az Exception interfészt kritikus hibák kezelésére szánták, különös tekintettel,
- de nem korlátozva kizárólagosan - a *cache setup* hibákra, mint a gyorsítótár szerver
vagy a megadott hitelesítő adatok hibái.

A jelen PSR-t implementáló programkönyvtáraknak meg KELL valósítani ezt az interfészt.

~~~php
<?php

namespace Psr\Cache;

/**
 * Általános kivétel interfész a jelen PSR-t implementáló programkönyvtárak által
 * dobott összes kivételhez.
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
 * Kivétel interfész az érvénytelen gyorsítótár argumentumok kezelésére.
 *
 * Bármikor egy érvénytelen argumentum kerül átadásra egy metódusnak, annak
 * olyan kivételt kell dobnia, amelyik implementálja az alábbi
 * Psr\Cache\InvalidArgumentException interfészt.
 */
interface InvalidArgumentException extends CacheException
{
}
~~~

[Kezdőlap](../README.md)
