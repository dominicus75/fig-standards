[Kezdőlap](../README.md)

# HTTP szerver kéréskezelők

Ez a dokumentáció közös interfészeket ír le a HTTP szerver kéréskezelők („request handlers”)
és a HTTP szerver köztesréteg („middleware”) számára, amelyek a  [PSR-7][psr7] vagy
az ezt helyettesítő PSR-ekben leírt HTTP üzeneteket.

A HTTP kéréskezelők alapvető részei bármely webalkalmazásnak. A szerveroldali kód
fogad egy kérés üzenetet, feldolgozza és válaszüzenetet állít elő. A HTTP köztesréteg
segítségével eltávolíthatjuk a gyakori kérések és válaszok feldolgozását az
alkalmazásrétegből.

A jelen dokumentumban leírt interfészek a kéréskezelők és a köztesréteg absztrakcióját
tartalmazzák.

*Megjegyzés: a kéréskezelőkre és a köztesrétegre való minden hivatkozás a **szerver kérések**
feldolgozására vonatkozik.*

A csupa nagybetűvel szedett, a követelmények szintjének jelzésére szolgáló kiemelt
kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

[psr7]: PSR-7-http-message.md/
[rfc2119]: ../related-rfcs/2119.md

### Referenciák

- [PSR-7][psr7]
- [RFC 2119][rfc2119]

## 1. Specifikáció

### 1.1 Kéréskezelők

A kéréskezelő olyan egyedi komponens, amely feldolgozza a HTTP kérést és a PSR-7
szabványban leírtak szerint létrehozza a HTTP válasz üzenetet.

A kéréskezelőnek LEHET kivételt is dobnia, ha a kérés körülményei meggátolják a
válasz előállításában. A kivétel típusa nincs meghatározva.

A jelen szabványt használó kéréskezelőknek KÖTELEZŐ megvalósítani a következő
interfészt:

- `Psr\Http\Server\RequestHandlerInterface`

### 1.2 Köztesréteg

A köztesréteg komponens olyan egyedi komponens, amely gyakran más köztesréteg
összetevőkkel együtt vesz részt a bejövő kérések feldolgozásában és a PSR-7 által
meghatározott válasz létrehozásában.

A köztesréteg komponensnek LEHET HTTP választ létrehozni és visszaadni anélkül,
hogy ezt tovább delegálná a kéréskezelőnek, ha elegendő feltétel teljesül.

A jelen szabványt használó köztesrétegnek KÖTELEZŐ megvalósítani a következő
interfészt:

- `Psr\Http\Server\MiddlewareInterface`

### 1.3 Válaszüzenet generálása

AJÁNLOTT, hogy bármely köztesréteg vagy kéréskezelő, amely HTTP választ állít elő,
vagy a PSR-7 `ResponseInterface` egy prototípusát, vagy egy ilyet generálni képes
gyártó példányt állítson össze, egy adott HTTP üzenet implementációtól való
függés megelőzése érdekében.

### 1.4 Kivételkezelés

AJÁNLOTT, hogy bármely köztesréteget használó alkalmazás tartalmazzon egy olyan
komponenst is, amelyik elkapja a kivételeket és HTTP válaszokká alakítja őket.
Ezen köztesrétegnek KELLENE az első futtatott összetevőnek lennie, amelynek meg
kell szakítania minden további feldolgozást, bizosítandó, hogy a HTTP válasz mindig
létrejön.

## 2. Interfészek

### 2.1 Psr\Http\Server\RequestHandlerInterface

Az alábbi interfészt a kéréskezelőknek KELL megvalósítani.

```php
namespace Psr\Http\Server;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

/**
 * Kezeli a szerver kéréseket és létrehozza a HTTP választ.
 *
 * Egy HTTP kéréskezelő feldolgozza a HTTP kérést és létrehozza a HTTP választ.
 */
interface RequestHandlerInterface
{
    /**
     * Kezel egy kérést és előállítja a választ.
     *
     * Hívhat további együttműködő kódokat is a válasz előállításához.
     */
    public function handle(ServerRequestInterface $request): ResponseInterface;
}
```

### 2.2 Psr\Http\Server\MiddlewareInterface

Az alábbi interfészt a kompatibilis köztesréteg komponensekkel KELL megvalósítani.

```php
namespace Psr\Http\Server;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

/**
 * Résztvesz a szerver kérés és válasz előállításában.
 *
 * A HTTP köztesréteg komponens részt vesz a HTTP üzenet feldolgozásában:
 * a kérésnek eleget téve előállítja a válasz üzenetet vagy továbbküldi a kérést
 * a soronkövetkező köztesrétegnek és esetleg reagál annak válaszára.
 */
interface MiddlewareInterface
{
    /**
     * Bejövő szerver kérés feldolgozása.
     *
     * Feldolgoz egy bejövő szerver kérést, azért, hogy előállítsa a választ.
     * Ha képtelen maga választ előállítani, delegálhatja a kérést egy másik
     * kéréskezelőnek, hogy az dolgozza fel.
     */
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface;
}
```

[Kezdőlap](../README.md)
