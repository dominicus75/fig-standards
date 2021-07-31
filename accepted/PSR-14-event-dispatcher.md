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

## Események

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

## Megállítható események

A **megállítható esemény** az események egy speciális esete, amely további módszereket
tartalmaz annak megakadályozására, hogy megakadályozza további figyelők meghívását.
Ezt a `StoppableEventInterface` implementálása jelzi.

An Event that implements `StoppableEventInterface` MUST return `true` from `isPropagationStopped()` when
whatever Event it represents has been completed.  It is up to the implementer of the class to
determine when that is.  For example, an Event that is asking for a PSR-7 `RequestInterface` object
to be matched with a corresponding `ResponseInterface` object could have a `setResponse(ResponseInterface $res)`
method for a Listener to call, which causes `isPropagationStopped()` to return `true`.

## Figyelők

A Listener may be any PHP callable.  A Listener MUST have one and only one parameter,
which is the Event to which it responds.  Listeners SHOULD type hint that parameter
as specifically as is relevant for their use case; that is, a Listener MAY type
hint against an interface to indicate it is compatible with any Event type that
implements that interface, or to a specific implementation of that interface.

A Listener SHOULD have a `void` return, and SHOULD type hint that return explicitly.
A Dispatcher MUST ignore return values from Listeners.

A Listener MAY delegate actions to other code. That includes a Listener being a
thin wrapper around an object that runs the actual business logic.

A Listener MAY enqueue information from the Event for later processing by a
secondary process, using cron, a queue server, or similar techniques.  It MAY
serialize the Event object itself to do so; however, care should be taken
that not all Event objects may be safely serializable. A secondary process
MUST assume that any changes it makes to an Event object will NOT propagate to other Listeners.

## Irányító

A Dispatcher is a service object implementing `EventDispatcherInterface`.  It is
responsible for retrieving Listeners from a Listener Provider for the Event dispatched, and invoking each Listener with that Event.

Egy irányítónak:

* MUST call Listeners synchronously in the order they are returned from a ListenerProvider.
* MUST return the same Event object it was passed after it is done invoking Listeners.
* MUST NOT return to the Emitter until all Listeners have executed.

If passed a Stoppable Event, a Dispatcher

* MUST call `isPropagationStopped()` on the Event before each Listener has been called.
If that method returns `true` it MUST return the Event to the Emitter immediately
and MUST NOT call any further Listeners.  This implies that if an Event is passed
to the Dispatcher that always returns `true` from `isPropagationStopped()`, zero listeners will be called.

A Dispatcher SHOULD assume that any Listener returned to it from a Listener
Provider is type-safe.  That is, the Dispatcher SHOULD assume that calling `$listener($event)` will not produce a `TypeError`.

[Promise object]: https://promisesaplus.com/

### Hibakezelés

An Exception or Error thrown by a Listener MUST block the execution of any further Listeners.
An Exception or Error thrown by a Listener MUST be allowed to propagate back up to the Emitter.

A Dispatcher MAY catch a thrown object to log it, allow additional action to be taken, etc.,
but then MUST rethrow the original throwable.

## Figyelő szolgáltató

A Listener Provider is a service object responsible for determining what Listeners
are relevant to and should be called for a given Event.  It may determine both what
Listeners are relevant and the order in which to return them by whatever means it chooses.

That MAY include:

* Allowing for some form of registration mechanism so that implementers may assign a Listener to an Event in a fixed order.
* Deriving a list of applicable Listeners through reflection based on the type and implemented interfaces of the Event.
* Generating a compiled list of Listeners ahead of time that may be consulted at runtime.
* Implementing some form of access control so that certain Listeners will only be called if the current user has a certain permission.
* Extracting some information from an object referenced by the Event, such as an Entity, and calling pre-defined lifecycle
methods on that object.
* Delegating its responsibility to one or more other Listener Providers using some arbitrary logic.

Any combination of the above, or other mechanisms, MAY be used as desired.

Listener Providers SHOULD use the class name of an Event to differentiate one event from another.
They MAY also consider any other information on the event as appropriate.

Listener Providers MUST treat parent types identically to the Event's own type when determining
listener applicability.  In the following case:

```php
class A {}

class B extends A {}

$b = new B();

function listener(A $event): void {};
```

A Listener Provider MUST treat `listener()` as an applicable listener for `$b`, as it is type
compatible, unless some other criteria prevents it from doing so.

## Objektum összeállítás

A Dispatcher SHOULD compose a Listener Provider to determine relevant listeners.  It is
RECOMMENDED that a Listener Provider be implemented as a distinct object from the Dispatcher but that is NOT REQUIRED.

## Interfészek

```php
namespace Psr\EventDispatcher;

/**
 * Defines a dispatcher for events.
 */
interface EventDispatcherInterface
{
    /**
     * Provide all relevant listeners with an event to process.
     *
     * @param object $event
     *   The object to process.
     *
     * @return object
     *   The Event that was passed, now modified by listeners.
     */
    public function dispatch(object $event);
}
```

```php
namespace Psr\EventDispatcher;

/**
 * Mapper from an event to the listeners that are applicable to that event.
 */
interface ListenerProviderInterface
{
    /**
     * @param object $event
     *   An event for which to return the relevant listeners.
     * @return iterable[callable]
     *   An iterable (array, iterator, or generator) of callables.  Each
     *   callable MUST be type-compatible with $event.
     */
    public function getListenersForEvent(object $event) : iterable;
}
```

```php
namespace Psr\EventDispatcher;

/**
 * An Event whose processing may be interrupted when the event has been handled.
 *
 * A Dispatcher implementation MUST check to determine if an Event
 * is marked as stopped after each listener is called.  If it is then it should
 * return immediately without calling any further Listeners.
 */
interface StoppableEventInterface
{
    /**
     * Is propagation stopped?
     *
     * This will typically only be used by the Dispatcher to determine if the
     * previous listener halted propagation.
     *
     * @return bool
     *   True if the Event is complete and no further listeners should be called.
     *   False to continue calling listeners.
     */
    public function isPropagationStopped() : bool;
}
```

[Kezdőlap](../README.md)
