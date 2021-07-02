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

A Link objektumok a legtöbb esetben értékobjektumok. Ezért hasznos lehet képessé
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
 * A readable link object.
 */
interface LinkInterface
{
    /**
     * Returns the target of the link.
     *
     * The target link must be one of:
     * - An absolute URI, as defined by RFC 5988.
     * - A relative URI, as defined by RFC 5988. The base of the relative link
     *     is assumed to be known based on context by the client.
     * - A URI template as defined by RFC 6570.
     *
     * If a URI template is returned, isTemplated() MUST return True.
     *
     * @return string
     */
    public function getHref();

    /**
     * Returns whether or not this is a templated link.
     *
     * @return bool
     *   True if this link object is templated, False otherwise.
     */
    public function isTemplated();

    /**
     * Returns the relationship type(s) of the link.
     *
     * This method returns 0 or more relationship types for a link, expressed
     * as an array of strings.
     *
     * @return string[]
     */
    public function getRels();

    /**
     * Returns a list of attributes that describe the target URI.
     *
     * @return array
     *   A key-value list of attributes, where the key is a string and the value
     *  is either a PHP primitive or an array of PHP strings. If no values are
     *  found an empty array MUST be returned.
     */
    public function getAttributes();
}
~~~

### 3.2 `Psr\Link\EvolvableLinkInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * An evolvable link value object.
 */
interface EvolvableLinkInterface extends LinkInterface
{
    /**
     * Returns an instance with the specified href.
     *
     * @param string $href
     *   The href value to include. It must be one of:
     *     - An absolute URI, as defined by RFC 5988.
     *     - A relative URI, as defined by RFC 5988. The base of the relative link
     *       is assumed to be known based on context by the client.
     *     - A URI template as defined by RFC 6570.
     *     - An object implementing __toString() that produces one of the above
     *       values.
     *
     * An implementing library SHOULD evaluate a passed object to a string
     * immediately rather than waiting for it to be returned later.
     *
     * @return static
     */
    public function withHref($href);

    /**
     * Returns an instance with the specified relationship included.
     *
     * If the specified rel is already present, this method MUST return
     * normally without errors, but without adding the rel a second time.
     *
     * @param string $rel
     *   The relationship value to add.
     * @return static
     */
    public function withRel($rel);

    /**
     * Returns an instance with the specified relationship excluded.
     *
     * If the specified rel is already not present, this method MUST return
     * normally without errors.
     *
     * @param string $rel
     *   The relationship value to exclude.
     * @return static
     */
    public function withoutRel($rel);

    /**
     * Returns an instance with the specified attribute added.
     *
     * If the specified attribute is already present, it will be overwritten
     * with the new value.
     *
     * @param string $attribute
     *   The attribute to include.
     * @param string $value
     *   The value of the attribute to set.
     * @return static
     */
    public function withAttribute($attribute, $value);

    /**
     * Returns an instance with the specified attribute excluded.
     *
     * If the specified attribute is not present, this method MUST return
     * normally without errors.
     *
     * @param string $attribute
     *   The attribute to remove.
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
 * A link provider object.
 */
interface LinkProviderInterface
{
    /**
     * Returns an iterable of LinkInterface objects.
     *
     * The iterable may be an array or any PHP \Traversable object. If no links
     * are available, an empty array or \Traversable MUST be returned.
     *
     * @return LinkInterface[]|\Traversable
     */
    public function getLinks();

    /**
     * Returns an iterable of LinkInterface objects that have a specific relationship.
     *
     * The iterable may be an array or any PHP \Traversable object. If no links
     * with that relationship are available, an empty array or \Traversable MUST be returned.
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
 * An evolvable link provider value object.
 */
interface EvolvableLinkProviderInterface extends LinkProviderInterface
{
    /**
     * Returns an instance with the specified link included.
     *
     * If the specified link is already present, this method MUST return normally
     * without errors. The link is present if $link is === identical to a link
     * object already in the collection.
     *
     * @param LinkInterface $link
     *   A link object that should be included in this collection.
     * @return static
     */
    public function withLink(LinkInterface $link);

    /**
     * Returns an instance with the specified link removed.
     *
     * If the specified link is not present, this method MUST return normally
     * without errors. The link is present if $link is === identical to a link
     * object already in the collection.
     *
     * @param LinkInterface $link
     *   The link to remove.
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
  a vonatkozó RFC [2. fejezete](https://datatracker.ietf.org/doc/html/rfc6570#section-2) tárgyalja.

[Kezdőlap](../README.md)
