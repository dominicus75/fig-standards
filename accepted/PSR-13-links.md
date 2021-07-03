[Kezdőlap](../README.md)

# Hivatkozásmeghatározó interfészek

A **hipermédia**<sup id="1">[[1]](#note1)</sup> hivatkozások az internet egyre fontosabb részévé válnak, mind HTML, mind
valamilyen API kontextusban. Ennek ellenére még nem létezik egy közös hipermédia
formátum, sem a formátumok közötti kapcsolat ábrázolására szolgáló általános módszer.

A jelen specifikáció célja, hogy a PHP fejlesztők számára egyszerű, általánosan
alkalmazható, a használt szerializációs formátumtól független módszert biztosítson
a hipermédia hivatkozások ábrázolására. Ez lehetővé teszi a rendszer számára, hogy
a választ hipermédia hivatkozásokkal együtt szerializálja, egy vagy több fizikai
formátumban, függetlenül a hivatkozások formátumától.

A csupa nagybetűvel szedett, a követelmények szintjének jelzésére szolgáló kiemelt
kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

### Referenciák

- [RFC 2119](../related-rfcs/2119.md)
- [RFC 4287](https://tools.ietf.org/html/rfc4287)
- [RFC 5988](https://tools.ietf.org/html/rfc5988)
- [RFC 6570](https://tools.ietf.org/html/rfc6570)
- [IANA Link Relations Registry](http://www.iana.org/assignments/link-relations/link-relations.xhtml)
- [Microformats Relations List](http://microformats.org/wiki/existing-rel-values#HTML5_link_type_extensions)

## 1. Specifikáció

### 1.1 Alapvető hivatkozások<sup id="2">[[2]](#note2)</sup>

Egy hipermédia hivatkozás minimálisan az alábbiakból áll:
- a hivatkozott célerőforrást reperezentáló URI
- a forrás és a célerőforrás viszonyát meghatározó kapcsolat.

Az alkalmazott formátumtól függően egy hivatkozásnak számos egyéb tulajdonsága is lehet.
Mivel a további attribútumok nem univerzálisak és nincsenek jól szabványosítva, ezért
ez a specifikáció sem foglalkozik velük.

Jelen specifikáció alkalmazásában az alábbi fogalmak jelentése a következő:

*    **Megvalósító objektum** - olyan objektum, amely megvalósítja a jelen specifikációban
meghatározott interfészek egyikét.

*    **Szerializáló** - egy függvénykönyvtár vagy más rendszer, amely egy vagy több
Link objektumot vesz át, majd egy olyan megadott formára alakítja, amely külső
adattárolóra lementhető, és amelyből az eredeti állapot később helyreállítható.

### 1.2 Tulajdonságok

Az URI és a kapcsolat mellett minden hivatkozásnak nulla vagy több további tulajdonsága
LEHET. Az itt engedélyezett értékekről nincs formális nyilvántartás, azok érvényessége
a kontextustól és gyakran az alkalmazott szerializálási formátumtól is függ. Az
általánosan támogatott tulajdonságok közé tartozik a `hreflang` (*a linkelt dokumentum
nyelve, kétbetűs nyelvkóddal ábrázolva*), `title`, vagy a `type` (*a hivatkozott
dokumentum média típusa*).

A szerializálóknak el LEHET hagyni a hivatkozás objektum tulajdonságait, ha az
alkalmazott szerializációs formátum ezt megköveteli. Ellenben a szerializálóknak
kódolni KELLENE az összes rendelkezésre álló attribútumot a felhasználói kiterjesztés
lehetővé tétele érdekében, hacsak a szerializációs formátum meghatározása ezt meg
nem akadályozza.

Némely tulajdonság (általában a `hreflang`) egynél többször is megjelenhet a kontextusában.
Ebből kifolyólag egy tulajdonság értéke LEHET tömb is. A szerializálóknak ezt a tömböt
bármilyen formátumban LEHET kódolni, amely megfelel a szerializált formátumnak
(szóközzel vagy vesszővel elválasztott lista, stb.). Ha a megadott tulajdonságnak
az adott kontextusban nem lehet több értéke, akkor a szerializálóknak az első értéket
KELL használniuk, az utána következő értékek figyelmenkivül hagyásával.

Ha a tulajdonság értéke logikai igaz (`true`), akkor a szerializálóknak LEHET a
megfelelő rövidített formát is használni, ha a szerializálási formátum ezt lehetővé
teszi. Például a HTML szabvány megengedi, hogy azon attribútumoknak, amelyek jelenlétének
logikai jelntése van (ilyen pl. a `required` vagy a `multiple` tulajdonság az
űrlapoknál), ne legyen értéke (maga az tulajdonság jelenléte hordozza az értéket).
Ez a szabály csak és kizárólag azon esetekre érvényes, ha a tulajdonság értéke kimondott
logikai igaz (`true`), de nem vonatkozik a PHP más, igazként is értelmezhető adattípusára
(pl. az `integer` típusú `1`).

Ha a tulajdonság értéke logikai hamis (`false`), akkor a szerializálóknak AJÁNLOTT
teljesen figyelmenkivül hagyni azt, kivéve, ha ez megváltoztatná az eredmény szematikai
jelentését. Ez a szabály csak és kizárólag azon esetekre érvényes, ha a tulajdonság
értéke kimondott logikai hamis (`false`), de nem vonatkozik a PHP más, hamisként
is értelmezhető adattípusára (pl. az `integer` típusú `0`).

### 1.3 Kapcsolatok

A hivatkozások kapcsolatai karakterláncként vannak meghatározva, ami vagy egy
szimpla kulcsszó (publikus kapcsolat esetén), vagy egy abszolút URI (privát
kapcsolat esetén).

Abban az esetben, ha a kapcsolat egy kulcsszóval van meghatározva, annak
olyannak KELLENE lenni, ami szerepel az [IANA nyilvántartásában](http://www.iana.org/assignments/link-relations/link-relations.xhtml).
Opcionálisan a [microformats.org nyilvántartása](http://microformats.org/wiki/existing-rel-values)
is használható, de ez nem minden kontextusban érvényes.

Azt a kapcsolatot, amelyet a fenti vagy más hasonló publikus nyilvántartásokban
szereplő kulcsszavakkal nem lehet leírni, privátnak kell tekinteni, ami egy
adott alkalmazásra vagy használati esetre jellemző. Az ilyen kapcsolatoknál
abszolút URI-t KELL megadni.

### 1.4 Hivatkozás sablonok<sup id="3">[[3]](#note3)</sup>

Az [RFC 6570](https://tools.ietf.org/html/rfc6570) meghatározza az URI-sablonok
formátumát, ami lényegében egy olyan minta, amelyet az ügyfél eszköz által megadott
értékekkel kell feltölteni. Némely hipermédia formátum támogatja a hivatkozás sablonokat,
míg mások esetleg nem. Egyes hipermédia formátumok egyedi módon jelzik, hogy a
hivatkozás sablont tartalmaz. A sablonokat nem támogató formátumokat kezelő
szerializálóknak figyelmen kívül KELL hagyniuk a sablonos hivatkozásokat, ha
ilyennel találkoznak.

### 1.5 Fejleszthető szolgáltatók

Bizonyos esetben a hivatkozás szolgáltatónak (Link Provider) szüksége lehet a képességre,
hogy további hivatkozásokat tudjon hozzáadni. Más esetekben futásidőben a hivatkozás
szolgáltató szükségképpen csak olvasható, más adatforrásból származó hivatkozásokkal.
Ezokból kifolyólag a módosítható providerek másodlagos interfészek, amelyek implementálása
opcionális.

Ráadásul néhány Link Provider objektum, mint például a [PSR-7](https://github.com/dominicus75/fig-standards/blob/master/accepted/PSR-7-http-message.md#33-psrhttpmessageresponseinterface) Response objektum megváltoztathatalannak (immutable) van tervezve. Ez azt jelenti,
hogy a linkek hozzáadásra irányuló metódusok ezekkel inkompatibilisek lennének.
Ebből kifolyólag az `EvolvableLinkProviderInterface` metódusai megkövetelik egy új
objektumpéldány visszaadását, amely megegyezik a kiindulási objektummal, azt leszámítva,
hogy tartalmaz egy további Link objektumot is (illetve a `removeLink()` esetén
eggyel kevesebbet tartalmaz, mint az eredeti).

### 1.6 Fejleszthető hivatkozás objektumok

A Link objektumok a legtöbb esetben értékobjektumok<sup id="4">[[4]](#note4)</sup>. Ezért hasznos lehet képessé
tenni őket a PSR-7 értékobjektumokhoz hasonló fejlődésre. Ezen okból kifolyólag
a jelen szabvány leír egy további `EvolvableLinkInterface`-t is, ami olyan metódusokat
biztosít, amelyek lehetővé teszik új objektumpéldányok létrehozást az eredeti
megváltoztatása nélkül. Ugyanez a modell van használatban a PSR-7 szabványban is,
ami a PHP által alkalmazott másolás íráskor (copy on write, COW) technikának hála
még mindig CPU-, és memória-hatékony.

A sablonokat tartalmazó értékekhez nincs kidolgozandó metódus, mivel egy hivatkozás
sablonos értéke kizárólag a `href` tulajdonság értékén alapul. Ezért TILOS önállóan
beállítani, e helyett azt kell megállapítani, hogy a `href` értéke tartalmaz-e az
[RFC 6570](https://tools.ietf.org/html/rfc6570) által meghatározott URI-sablont,
vagy nem.

## 2. A csomag

A leírt interfészek és osztályok a [psr/link](https://packagist.org/packages/psr/link)
csomag részei.

## 3. Interfészek

### 3.1 `Psr\Link\LinkInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * Csak olvasható Link objektum.
 */
interface LinkInterface
{
    /**
     * Visszatér a hivatkozás célpontjával.
     *
     * A hivatkozás céljának az alábbiak egyikének kell lennie:
     * - egy abszolút URI, az RFC 5988 meghatározása szerint.
     * - relatív URI, az RFC 5988 meghatározása szerint. Feltételezzük, hogy a relatív
     *   hivatkozás bázisát (a base html tag href tulajdonságában beállított érték)
     *   a kliens a kontextus alapján felismeri.
     * - URI sablon az RFC 6570 meghatározása szerint.
     *
     * Ha URI-sablont adunk vissza, akkor az isTemplated() metódusnak true értékkel
     * KELL visszatérnie.
     *
     * @return string
     */
    public function getHref();

    /**
     * Megállapítja, hogy a hivatkozás tartalmaz-e URI-sablont.
     *
     * @return bool
     *  True, ha az objektum URI-sablont tartalmaz, egyébként False.
     */
    public function isTemplated();

    /**
     * Visszadja a hivatkozás kapcsolattípusát/típusait.
     *
     * Nulla vagy több, a hivatkozáshoz tartozó kapcsolattípust ad vissza egy
     * string értékeket tartalmazó tömbben.
     *
     * @return string[]
     */
    public function getRels();

    /**
     * Visszaadja az URI tulajdonságainak listáját.
     *
     * @return array
     *  A tulajdonságok listája kulcs-érték párok formájában, ahol a kulcs típusa
     *  string, az érték pedig a PHP valamely elemi adattípusa, vagy string-tömb.
     *  Ha nem található egyetlen tulajdonság sem, akkor üres tömböt KELL visszaadni.
     */
    public function getAttributes();
}
~~~

### 3.2 `Psr\Link\EvolvableLinkInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * Egy fejleszthető hivatkozás értékobjektum.
 */
interface EvolvableLinkInterface extends LinkInterface
{
    /**
     * Visszaad egy új példányt a megadott href tulajdonsággal.
     *
     * @param string $href
     *   A belefoglalandó href értéknek az alábbiak egyikének kell lennie:
     *    - egy abszolút URI, az RFC 5988 meghatározása szerint.
     *    - relatív URI, az RFC 5988 meghatározása szerint. Feltételezzük, hogy a relatív
     *      hivatkozás bázisát (a base html tag href tulajdonságában beállított érték)
     *      a kliens a kontextus alapján felismeri.
     *    - URI sablon az RFC 6570 meghatározása szerint.
     *    - Egy __toString() metódust megvalósító objektum, amely a fenti értékek
     *      egyikét állítja elő.
     *
     * Egy megvalósító könyvtárnak az átadott objektumot rögtön karakterláncá
     * KELLENE alakítania, ahelyet, hogy később visszadná.
     *
     * @return static
     */
    public function withHref($href);

    /**
     * Egy olyan objektumpéldányt ad vissza, amelyben a megadott kapcsolat szerepel.
     *
     * Ha a megadott kapcsolat már létezik, akkor ennek a metódusnak hiba nélkül
     * KELL visszatérnie, a nélkül, hogy a már létező kapcsolatot ismét hozzáadta volna.
     *
     * @param string $rel
     *   A hozzáadanadó kapcsolat értéke.
     * @return static
     */
    public function withRel($rel);

    /**
     * Visszaad egy olyan objektumpéldányt, amely a megadott kapcsolatot már
     * nem tartalmazza.
     *
     * Ha a megadott kapcsolat már nem létezik, akkor ennek a metódusnak hiba nélkül
     * KELL visszatérnie.
     *
     * @param string $rel
     *   Az eltávolítandó kapcsolat értéke.
     * @return static
     */
    public function withoutRel($rel);

    /**
     * Visszaad egy új objektumpéldányt a megadott tulajdonság hozzáadásával.
     *
     * Ha a megadott tulajdonság már létezik, akkor felülírja azt az új értékkel.
     *
     * @param string $attribute
     *   Beillesztendő tulajdonság neve.
     * @param string $value
     *   A tulajdonság beállítandó értéke.
     * @return static
     */
    public function withAttribute($attribute, $value);

    /**
     * Visszaad egy új objektumpéldányt a megadott tulajdonság nélkül.
     *
     * Ha a megadott tulajdonság már nem létezik, akkor ennek a metódusnak hiba nélkül
     * KELL visszatérnie.
     *
     * @param string $attribute
     *   Az eltávolítandó tulajdonság.
     * @return static
     */
    public function withoutAttribute($attribute);
}
~~~

### 3.2 `Psr\Link\LinkProviderInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * Egy hivatkozás szolgáltató objektum.
 */
interface LinkProviderInterface
{
    /**
     * LinkInterface objektumokat tartalmazó bejárható adatszerkezetet ad vissza.
     *
     * A bejárható adatszerkezet lehet tömb, vagy a PHP \Traversable interfészt
     * megvalósító bármilyen objektum. Ha egy LinkInterface objektum sem áll rendelkezésre,
     * akkor ennek a metódusnak egy üres tömböt vagy \Traversable objektumot KELL
     * visszadnia.
     *
     * @return LinkInterface[]|\Traversable
     */
    public function getLinks();

    /**
     * LinkInterface objektumokat tartalmazó bejárható adatszerkezetet ad vissza,
     * amely tartalmazza a megadott kapcsolatot.
     *
     * A bejárható adatszerkezet lehet tömb, vagy a PHP \Traversable interfészt
     * megvalósító bármilyen objektum. Ha egy olyan LinkInterface objektum sem áll
     * rendelkezésre, amely tartalmazza a megadott kapcsolatot, akkor ennek a metódusnak
     * egy üres tömböt vagy \Traversable objektumot KELL visszadnia.
     *
     * @return LinkInterface[]|\Traversable
     */
    public function getLinksByRel($rel);
}
~~~

### 3.3 `Psr\Link\EvolvableLinkProviderInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * Egy fejleszthető hivatkozás szolgáltató értékobjektum.
 */
interface EvolvableLinkProviderInterface extends LinkProviderInterface
{
    /**
     * Visszaad egy olyan objektumpéldányt, amely a megadott LinkInterface objektumot
     * is tartalmazza.
     *
     * Ha a megadott hivatkozás már létezik, akkor ennek a metódusnak hiba nélkül
     * KELL visszatérnie. A LinkInterface objektum akkor létezik, ha a $link azonos
     * (===) a gyüjteményben már megtalálható egyik LinkInterface objektummal.
     *
     * @param LinkInterface $link
     *   Egy hivatkozás objektum, amelyet hozzá kell adni a gyüjteményhez.
     * @return static
     */
    public function withLink(LinkInterface $link);

    /**
     * Visszaad egy olyan objektumpéldányt, amely a megadott LinkInterface objektumot
     * már nem tartalmazza.
     *
     * Ha a megadott hivatkozás nem létezik, akkor ennek a metódusnak hiba nélkül
     * KELL visszatérnie. A LinkInterface objektum akkor létezik, ha a $link azonos
     * (===) a gyüjteményben már megtalálható egyik LinkInterface objektummal.
     *
     * @param LinkInterface $link
     *   Egy hivatkozás objektum, amelyet el kell távolítani a gyüjteményből.
     * @return static
     */
    public function withoutLink(LinkInterface $link);
}
~~~

### A fordító jegyzetei:

* <span id="note1">[[1]](#1)</span> *A **hipermédia** a hipertext fogalmát úgy terjeszti
  ki, hogy az egyes csomópontokban nemcsak újabb szöveget (szöveges dokumentumot),
  hanem tetszőleges más médiaelemet, képet, hangot, videót is találhatunk. A szakirodalomban
  ma már ritkán különböztetik meg a hipertext és hipermédia fogalmát, gyakran szinonim
  kifejezésként alkalmazzák őket. A hipertext és hipermédia lényeges tulajdonsága
  a nemlineáris információláncolás. Ha az egyes elemek bizonyos jelentésbeli összefüggéseik
  mentén össze vannak kapcsolva, akkor ezt a nemlineáris összefüggésrendszert nevezzük
  hipertextnek, illetve bizonyos esetekben hipermédiának. A hipermédia-rendszer a
  hipertext és a multimédia közös halmazát alkotja, magában egyesítvén a hipertext
  kapcsolódási és a multimédia összetett médiarendszerét. A hipermédia-rendszer az
  információk nemlineáris láncolatából áll. Az információegységek minden kapcsolata
  hivatkozásokkal valósul meg. Egyszerű módon integrál összetett médiumokat.*
  (Forgó Sándor, Lengyelné Molnár Tünde: [Multimédiafejlesztés](https://regi.tankonyvtar.hu/hu/tartalom/tamop412A/2011-0021_47_multimediafejlesztes/2210_a_hipertext_s_a_hipermdia.html) című könyve nyomán)
* <span id="note2">[[2]](#2)</span> *A linkek azok a szervező eszközök a hipertexten
  belül, melyek megteremtik a kapcsolatot a tetszőleges szempont alapján összetartozó,
  egymással asszociálható információs egységek, csomópontok vagy csomópontrészek
  között. A linkek lehetnek egyirányúak, amikor csak azt tudjuk megmondani, egy adott
  dokumentumból hová mutatnak linkek. A kétirányú linkekben az is megállapítható,
  mely csomópontokból mutatnak linkek agy adott dokumentum valamely pontjára. A linkek
  jelölhetnek valamilyen szemantikus kapcsolatot két dokumentum között, s a kapcsolat
  jellege szerint a linkek tipizálhatók. A web-terminológiában a linkek referenciák
  (egy cím), melyek a web valamely erőforrására mutatnak. Ezek az erőforrások lehetnek
  egy HTML oldal, egy kép, hangfájl, videó stb.*
  (Forgó Sándor, Lengyelné Molnár Tünde: [Multimédiafejlesztés](https://regi.tankonyvtar.hu/hu/tartalom/tamop412A/2011-0021_47_multimediafejlesztes/2210_a_hipertext_s_a_hipermdia.html) című könyve nyomán)
* <span id="note3">[[3]](#3)</span> *Az URI-sablonok nyomtatható unicode karakterekből
  álló karakterláncok, amelyek nulla vagy több beágyazott változó-kifejezést tartalmaznak.
  Minden kifejezést kapcsoszárójelek (`{` és `}`) határolnak. A részletes szintaxist
  a vonatkozó RFC [2. fejezete](https://datatracker.ietf.org/doc/html/rfc6570#section-2) tárgyalja*.
* <span id="note4">[[4]](#4)</span> *Az értékobjektum (vagy adatátviteli objektum, DTO)
  egy egyszerű (primitív) értéket reprezántáló kisebb, jellemzően immutable-típusú objektum.
  Értékobjektumok esetében két objektum egyenlősége nem azok identitásán, hanem a
  bennük tárolt érték megegyezőségén alapszik.*

[Kezdőlap](../README.md)
