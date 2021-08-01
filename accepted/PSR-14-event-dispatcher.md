[Kezdőlap](../README.md)

# Eseményirányító (Event Dispatcher)

Az eseményirányítás egy gyakori és jól tesztelt mehanizmus, ami lehetővé teszi
a fejlesztők számára, hogy könnyen és következetesen logikát fecskendezzenek
be egy alkalmazásba.

Jelen PSR célja az, hogy egy közös mechanizmust hozzon létre annak érdekében,
hogy a függvénykönyvtárakat és komponenseket szabadabban felhasználhassák
eseményalapú kiterjesztésre, valamint különféle alkalmazások és keretrendszerek
közötti együttműködésre.

A csupa nagybetűvel szedett, a követelmények szintjének jelzésére szolgáló kiemelt
kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

## Cél

Közös interfészeket biztosítani az események kezeléséhez és irányításához, melyek
lehetővé teszik a fejlesztők számára, hogy olyan függvénykönyvtárakat hozzanak
létre, amelyek azonos módon képesek számos keretrendszerrel vagy más függvénykönyvtárral
interkciót folytatni.

Néhány példa:

* Biztonsági keretrendszer, amely megakadályozza, hogy a felhasználók olyan
adatokat mentsenek vagy érjenek el, amihez nincs jogosultságuk.
* Közös teljes oldal gyorstárazási rendszer.
* Programkönyvtárak, amelyek más könyvtárak képességeit terjesztik ki, függetlenül
attól, hogy ezek mely keretrendszerbe vannak integrálva.
* Naplozó csomag az alkalmazáson belül végrehajtott összes művelet követésére.

## Meghatározások

* **Esemény (Event)** - egy *Küldő (Emitter)* által létrehozott üzenet, ami tetszőleges
PHP objektum lehet.
* **Figyelő (Listener)** - bármely olyan meghívható PHP-kód, amely egy esemény
átadását várja. Nulla vagy több eseményfigyelő is feliratkozhat ugyanarra az eseményre.
A figyelőnek LEHET más aszinkron viselkedést is előidézni.
* **Küldő (Emitter)** - bármely tetszőleges kód, ami eseményt szeretne küldeni.
"Hívó kódként" is ismert. Használati esetre és nem adatszerkezetre utal.
* **Irányító (Dispatcher)** - egy olyan szolgáltatás-objektum (service), amelyet egy
küldő ad meg egy eseményobjektum számára. Az irányító felelős azért, hogy biztosítsa
az események eljuttatását az összes érintett figyelőnek, de az érintett figyelők
meghatározását egy Figyelő szolgáltató (Listener Provider) objektumra KELL bízni.
* **Figyelő szolgáltató (Listener Provider)** - felelős annak meghatározásáért, hogy
mely figyelők érintettek egy adott eseményben, viszont magát a figyelőt TILOS meghívnia.
A figyelő szolgáltató nulla vagy több releváns figyelőt adhat meg.

## Események (Events)

Az események olyan objekjtumok, amelyek kommunikációs egységként működnek a küldő
és a figyelő között.

Az eseményobjektum LEHET változtatható, ha a használati eset arra hívja a figyelőt,
hogy információt küldjön vissza a küldőnek. Ha azonban nincs szükség ilyen kétirányú
kommnikációra, akkor AJÁNLOTT, hogy az eseményodjektum megváltoztathatatlanként
(immutable) legyen definiálva; azaz az objektum állapotát megváltoztató metódusok
nélkül.

A megvalósítóknak vállalniuk KELL, hogy ugyanazt az objektumot az összes figyelőnek
el lehessen küldeni.

AJÁNLOTT, de NEM SZÜKSÉGES, hogy az eseményobjektumok támogassák a veszteségmentes
szerializációt és deszerializációt; az `$event == unserialize(serialize($event))`
kódnak ezért `true` értéket KELLENE visszadnia. Az objektumoknak LEHET alkalmazni
a PHP `Serializable` interfészét, a `__sleep()` vagy a `__wakeup()` mágikus
metódusokat, illetve más hasonló nyelvi elemet.

## Megállítható események (Stoppable Events)

A **megállítható esemény** az események egy speciális esete, amely további módszereket
tartalmaz annak megakadályozására, hogy további figyelők kerüljenek meghívásra.
Ezt a `StoppableEventInterface` implementálása jelzi.

Egy eseményobjektumban, amely implementálja a `StoppableEventInterface`-t, a
`isPropagationStopped()` metódusnak KÖTELEZŐ `true` értéket visszaadni, amikor bármilyen,
általa képviselt esemény befejeződött. Az osztály megvalósítójától függ, hogy ez
mikor történik meg. Például egy olyan esemény, amelyik egy PSR-7 típusú `RequestInterface`
objektumot vár, hogy megfeleltesse a megfelelő `ResponseInterface` objektummal,
rendelkezhet egy `setResponse(ResponseInterface $res)` metódussal, amit a figyelő
meghívhat, ez esetben a `isPropagationStopped()` metódus `true` értékkel tér vissza.

## Figyelők (Listeners)

Figyelő lehet bármilyen meghívható PHP-kód (callable). A figyelőnek egy és csakis egy
paraméterrel KELL rendelkeznie, ez az az esemény, amire válaszol. A figyelőknek
AJÁNLOTT type hint-elni azt a paramétert, amelyik az ő használati esetükre vonatkozik,
vagyis egy figyelőnek LEHET type hint-elni egy interfészt azért, hogy jelezze:
kompatibilis valamennyi eseménytípussal, amelyik implementálja a jelzett interfészt,
vagy az interfész egy konkrét megvalósításával.

A figyelőknek AJÁNLOTT érték visszaadása nélkül visszatérni (`void`) és ezt AJÁNLOTT
kifejezetten jelezniük is a metódus szignatúrájában. Az irányítónak viszont
KÖTELEZŐ figyelmenkivül hagynia a figyelő visszatérési értékét.

Egy figyelőnek LEHET továbbdelegálnia bármely tevékenységet más kódnak. Ez magában
foglalja azt is, hogy a figyelő tulajdonképpen egy burkoló (wrapper) azon objektum
körül, amelyik a tényleges üzleti logikát tartalmazza.

Egy figyelőnek LEHET későbbi, másodlagos eljárással történő feldolgozás céljából
besorolni az információkat. LEHET az eseményobjektumot magát szerializálni is, ügyelve
arra, hogy nem minden eseményobjektum szerializálható biztonságosan. A másodlagos
eljárásnak KÖTELEZŐ biztosítani, hogy az eseményobjektumon általa eszközölt bármilyen
változás nem fog továbbterjedni más figyelőkre.

## Irányító (Dispatcher)

Az irányító egy szolgáltatás objektum, amely implemetálja az `EventDispatcherInterface`
interfészt. Ez felelős az eseménynek megfelelő figyelők lekéréséért a figyelő szolgáltató
(Listener Provider) objektumtól és minden egyes figyelő meghívásáért az adott eseménnyel.

Egy irányítónak:

* abban a sorrendben KELL meghívnia a figyelőket, ahogy a figyelő szolgáltatótól
megkapja őket.
* ugyanazzal az eseményobjektummal KELL visszatérnie, amelyet a figyelő meghívása
után megkapott.
* TILOS visszatérnie a küldőhöz, amíg az összes figyelő le nem futott.

Ha egy leállítható esemény át lett adva a figyelőnek, akkor annak

* KÖTELEZŐ meghívnia az `isPropagationStopped()` metódust az eseményen, mielőtt
bármely figyelőt is meghívna. Ha ez a metódus `true` értékkel tér vissza, akkor
azonnal vissza KELL adnia az eseményt a küldőnek és TILOS más további eseményfigyelőt
meghívnia. Ez azt jelenti, hogy ha egy esemény el lett küldve az irányítónak, akkor
annak mindig `true` értéket kell visszadni az `isPropagationStopped()` metódusból,
valamint pontosan nulla figyelő fog meghívódni.

Az irányítónak feltételeznie KELLENE, hogy a figyelő szolgáltató által visszaadott
bármely figyelő típusbiztos. Vagyis, az irányítónak azt KELLENE feltételeznie, hogy
a `$listener($event)` kód nem eredményez `TypeError`-t.

### Hibakezelés

A figyelő által dobott kivételnek vagy hibának blokkolnia KELL minden további figyelő
végrehajtását. A figyelő által dobott kivételnek vagy hibának engednie KELL, hogy
visszaterjedjen a kibocsátóhoz.

Az irányítónak naplózás vagy más további művelet céljából LEHET dobott hibát vagy
kivételt elkapnia, de ez után KÖTELEZŐ azt változatlan formában továbbdobnia.

## Figyelő szolgáltató (Listener Provider)

A figyelő szolgáltató egy szolgáltató-objektum, ami annak meghatározásáért felelős,
hogy adott eseményhez mely eseményfigyelőt kell meghívni. Meghatározhatja, hogy
mely figyelőket és milyen sorrendben kell meghívni. Ebbe a következőket LEHET
beleérteni:

* A regisztrációs mechanizmus bizonyos formáinak engedélyezése, hogy a megvalósítók
rögzített sorrendben rendelhessenek figyelőket az eseményekhez.
* Az alkalmazható eseményfigyelők listájának összeállítása a megvalósított esemény
interfész és az esemény típusa alapján.
* Az eseményfigyelők listájának idő előtti összeállítása, melyből futásidőben lehet
tájékozódni.
* A hozzáférés-szabályozás valamilyen formájának megvalósítása azért, hogy bizonyos
figyelők csak akkor legyenek meghívhatók, ha az aktuális felhasználó rendelkezik
a szükséges jogosultsággal.
* Néhány információ kinyerése az esemény által hivatkozott objektumból, mint például
egy entitás és előre meghatározott életciklus metódusok meghívása azon az objektumon.
* A felelősség átruházása egy vagy több másik figyelő szolgáltató objektumra,
tetszőleges logikát alkalmazva.

A fentiek bármely kombinációja vagy akár más mechanizmus is TETSZŐLEGESEN alkalmazható.

A figyelő szolgáltatóknak AJÁNLOTT használni egy esemény osztálynevét, hogy meg
tudják különböztetni azt a többitől. A figyelő szolgáltatóknak adott esetben figyelembe
LEHET venni más információt is az eseményről.

A figyelő szolgáltatóknak az esemény saját típusával azonos módon KELL kezelni a
szülő típusokat is, amikor meghatározzák a figyelő alkalmazhatóságát. Mint a következő
esetben:

```php
class A {}

class B extends A {}

$b = new B();

function listener(A $event): void {};
```

A figyelő szolgáltatóknak KÖTELEZŐ `listener()`-t `$b` számára alkalmazható. típus
kompatibilis figyelőként kezelni, hacsak valami más kritérim meg nem akadályozza ezt.

## Objektum összeállítás

Az irányítónak AJÁNLOTT összeállítania egy figyelő szolgáltatót a releváns figyelők
meghatározására. AJÁNLOTT továbbá, hogy a figyelő szolgáltatót az irányítótól
különböző objektumként valósítsák meg, de ez nem feltétlenül szükséges.

## Interfészek

```php
namespace Psr\EventDispatcher;

/**
 * Definiál egy irányítót az eseményekhez.
 */
interface EventDispatcherInterface
{
    /**
     * Feldolgozható eseményt biztosít minden releváns figyelőnek.
     *
     * @param object $event
     *   A feldolgozandó eseményobjektum.
     *
     * @return object
     *   Az átadott esemény, a figyelők által módosítva.
     */
    public function dispatch(object $event);
}
```

```php
namespace Psr\EventDispatcher;

/**
 * Feltérképezi az adott eseményhez alkalmazható figyelőket.
 */
interface ListenerProviderInterface
{
    /**
     * @param object $event
     *   egy esemény, amelyre megkapjuk az érintett figyelőket.
     * @return iterable[callable]
     *   Egy bejárható/iterable (array, iterator, vagy generator) adatszerkezet,
     *   amely callable típusú elemeket tartalmaz. Minden hívható elemnek típus
     *   kompatibilisnek KELL lennie az $event-el.
     */
    public function getListenersForEvent(object $event) : iterable;
}
```

```php
namespace Psr\EventDispatcher;

/**
 * Esemény, amelynek feldolgozása félbeszakadhat, amikor az eseményt lekezelték.
 *
 * Az irányító megvalósításának ellenőrizni KELL, hogy egy esemény megállíthatónak
 * van-e jelölve, miután minden figyelőt meghívtak. Ha igen, akkor rögtön visszatérhet
 * anélkül, hogy bármely további figyelőt meghívna.
 */
interface StoppableEventInterface
{
    /**
     * A továbbadás megállt?
     *
     * Ezt az irányító jellemzően csak akkor fogja használni, ha az előző figyelő
     * megállította az esemény továbbadását.
     *
     * @return bool
     *   true, ha az esemény készen van és nincs további hívandó eseményfigyelő.
     *   false, ha folytatni kell a figyelők meghívását.
     */
    public function isPropagationStopped() : bool;
}
```

[Kezdőlap](../README.md)
