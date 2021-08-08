[Kezdőlap](../README.md)

# HTTP ügyfél

Ez a dokumentum egy olyan közös felületet ír le, amely képes HTTP kéréseket küldeni
és HTTP válaszüzeneteket fogadni.

A csupa nagybetűvel szedett, a követelmények szintjének jelzésére szolgáló kiemelt
kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

## Cél

E PSR célja, hogy lehetővé tegye a fejlesztők számára a HTTP ügyfél implementációktól
független programkönyvtár létrehozását. Ez a könyvtárakat még inkább újrafelhasználhatóvá
teszi az által, hogy csökkenti a függőségek számát és a verzióütközések valószínűségét.

Másodlagos cél, hogy a HTTP ügyfelek a [Liskov helyettesítési elvnek][Liskov] megfelelően
cserélhetőek legyenek. Ez azt jelenti, hogy az összes ügyfélnek azonos módon KELL
viselkednie a HTTP kérés elküldésekor.

## Meghatározások

* **Ügyfél (kliens)** - olyan könyvtár, amely PSR-7 kompatibilis HTTP kérés üzenetek
küldése és HTTP válasz üzenetek hívónak történő visszaadása céljából megvalósítja
ezt a specifikációt.
* **Hívó kód** - bármely kód, amely használja az ügyfél szolgálatait. Nem valósítja
meg a jelen specifikáció interfészeit, de olyan objektumot vesz igénybe, amelyik
implementálja őket (ez az ügyfél vagy kliens).

## Ügyfél

Az ügyfél egy olyan objektum, amely megvalósítja a `ClientInterface`-t.

Az ügyfél által VÁLASZTHATÓ lehetőségek:

* küldhet egy módosított HTTP kérést. Például tömörítheti a kimenő üzenet törzsét.
* módosíthatja a kapott HTTP választ, mielőtt visszadja a hívó kódnak. Például
kitömörítheti a bejövő üzenet törzsét.

Ha az ügyfél a HTTP kérés vagy HTTP válasz módosítását választja, akkor biztosítania
KELL azt, hogy az objektum belsőleg konzisztens maradjon. Például ha az ügyfél
úgy dönt, hogy kitömöríti az üzenet törzsét, akkor el KELL távolítania a `Content-Encoding`
fejlécet és frissítnei a `Content-Length` fejlécet.

Megjegyzendő, hogy ennek eredményeként, mivel a PSR-7 objektumok megváltoztathatatlanok,
a hívó kódnak TILOS feltételezni, hogy a `ClientInterface::sendRequest()` metódusnak
átadott objektum ugyanaz a PHP objektum lesz, amit ténylegesen elküldtek. Például
a kivétel által visszaadott HTTP kérés objektum LEHET, hogy különbözni fog attól,
amit a `sendRequest()` metódus megkapott, ezért a referencia szerinti (`===`)
összehasonlítás nem lehetséges.

Az ügyfél KÖTELES:

* újra összeállítani egy többlépéses HTTP 1xx választ úgy, hogy ha visszaküldésre
kerül a hívó kódnak, érvényes HTTP válasz legyen, 200-as vagy magasabb állapotkóddal.

## Hibakezelés

Az ügyfélnek TILOS hibaállapotként kezelni a jól formázott HTTP kérés és válasz
üzeneteket. Például ha a válasz állapotkódja 400 és 500 közti tartományban van,
akkor ennek TILOS kivételt kiváltania és a választ normál módon KELL visszaküldeni
a hívónak.

Az ügyfélnek egy `Psr\Http\Client\ClientExceptionInterface` példányt KELL dobnia,
ha és csakis akkor, ha egyáltalán nem képes elküldeni a HTTP kérést, vagy ha a HTTP
válasz nem értelmezhető PSR-7 HTTP válasz objektumként.

Ha a kérést nem lehet elküldeni, mert a kérés üzenet nem megfelelően formázott
HTTP kérés, vagy hiányzik belőle néhány kritikus információ (mint például a Host
vagy a HTTP metódus), akkor az ügyfélnek egy `Psr\Http\Client\RequestExceptionInterface`
példányt KELL dobnia.

Ha a kérés hálózati, vagy bármiféle hiba (ideértve az időtúllépést) következtében
nem elküldhető, akkor az ügyfélnek egy `Psr\Http\Client\NetworkExceptionInterface`
példányt KELL dobnia.

Az ügyfélnek LEHET további specifikus kivételeket is dobnia, mint az itt defináltak
(pl. a `TimeOutException` vagy `HostNotFoundException`), feltéve, hogy azok megvalósítják
a fentebb meghatározott interfészt (vagyis a kivétel a `Psr\Http\Client\ClientExceptionInterface`
interfészből vagy valamely leszármazottjából származik).

## Interfészek

### ClientInterface

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;

interface ClientInterface
{
    /**
     * Elküld egy PSR-7 kérést és visszatér egy PSR-7 válasszal.
     *
     * @param RequestInterface $request
     *
     * @return ResponseInterface
     *
     * @throws \Psr\Http\Client\ClientExceptionInterface Ha hiba merül fel a kérés feldolgozása közben.
     */
    public function sendRequest(RequestInterface $request): ResponseInterface;
}
```

### ClientExceptionInterface

```php
namespace Psr\Http\Client;

/**
 * Minden HTTP ügyfélhez kapcsolódó kivételnek meg KELL valósítania ezt az interfészt,
 * vagyis ebből kell leszármaztatni.
 */
interface ClientExceptionInterface extends \Throwable
{
}
```

### RequestExceptionInterface

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;

/**
 * Kivétel sikertelen HTTP kérés esetén.
 *
 * Példák:
 *      - A kérés érvénytelen (pl. hiányzik a HTTP metódus)
 *      - Futásidejű kérés hibák (pl. a kéréstörzs-adatfolyama nem kereshető)
 */
interface RequestExceptionInterface extends ClientExceptionInterface
{
    /**
     * Visszaadja a kérést.
     *
     * A kérés objektum LEHET más, mint a ClientInterface::sendRequest()
     * metódusnak átadott.
     *
     * @return RequestInterface
     */
    public function getRequest(): RequestInterface;
}
```

### NetworkExceptionInterface

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;

/**
 * Akkor dobandó, ha a kérést hálózati problémák miatt nem lehet végrehajtani.
 *
 * Nincs válaszobjektum, mivel ez a kivétel akkor dobandó, ha nem érkezett válasz.
 *
 * Példa: a célgép nevét nem sikerült feloldani vagy a kapcsolat megszakadt.
 */
interface NetworkExceptionInterface extends ClientExceptionInterface
{
    /**
     * Visszaadja a kérést.
     *
     * A kérés objektum LEHET más, mint a ClientInterface::sendRequest()
     * metódusnak átadott.
     *
     * @return RequestInterface
     */
    public function getRequest(): RequestInterface;
}
```

[Liskov]: https://letscode.hu/2016/07/05/betonozas-3-0-liskov-es-haverok/

[Kezdőlap](../README.md)
