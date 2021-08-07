[Kezdőlap](../README.md)

# HTTP gyárak

Ez a dokumentum egy [PSR-7][psr7] szerinti HTTP objektumok létrehozására alkalmas
gyártó objektumot deklaráló közös szabványt ír le.

A PSR-7 ugyanis nem tartalmazott a HTTP objektumok létrehozására vonatkozó ajánlást,
ez pedig nehézségekhez vezet, amikor új HTTP objektumot kell létrehozni olyan
összetevőkön belül, amelyek nem kapcsolódnak a PSR-7 egy konkrét megvalósításához.

A jelen dokumentumban szereplő interfészek olyan metódusokat vázolnak fel, amelyekkel
PSR-7 stílusú objektumokat lehet példányosítani.

A csupa nagybetűvel szedett, a követelmények szintjének jelzésére szolgáló kiemelt
kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

[psr7]: PSR-7-http-message.md/
[rfc2119]: ../related-rfcs/2119.md

## 1. Specifikáció

A HTTP gyártó egy olyan metódus, amellyel új, PSR-7 szerinti HTTP objektumot lehet
létrehozni. A HTTP gyáraknak KÖTELEZŐ megvalósítaniuk ezeket az interfészeket
minden egyes objektumtípusra, amelyet a csomag biztosít.

## 2. Interfaces

A következő interfészeket meg LEHET valósítnai egyetlen közös osztályban, vagy
különálló osztályokban is.

### 2.1 RequestFactoryInterface

Képes PSR-7 stílusú kliensoldali HTTP kérés objektumok létrehozására.


```php
namespace Psr\Http\Message;

use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\UriInterface;

interface RequestFactoryInterface
{
    /**
     * Létrehoz egy új RequestInterface típusú objektumpéldányt.
     *
     * @param string $method A kéréshez társított HTTP metódus.
     * @param UriInterface|string $uri A kéréshez társított URI.
     */
    public function createRequest(string $method, $uri): RequestInterface;
}
```

### 2.2 ResponseFactoryInterface

Képes PSR-7 stílusú HTTP válasz objektumok létrehozására.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\ResponseInterface;

interface ResponseFactoryInterface
{
    /**
     * Létrehoz egy új ResponseInterface típusú objektumpéldányt.
     *
     * @param int $code            HTTP állapotkód. Alapértelmezett a 200.
     * @param string $reasonPhrase Az állapotkódhoz társított szöveges indoklás a
     *                             generált válaszban. Ha nincs megadva, az implementációknak LEHET
     *                             a HTTP specifikáció által ajánlott alapértelmezett
     *                             indiklásokat is használni.
     */
    public function createResponse(int $code = 200, string $reasonPhrase = ''): ResponseInterface;
}
```

### 2.3 ServerRequestFactoryInterface

Képes PSR-7 stílusú szerver request objektumok létrehozására.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\UriInterface;

interface ServerRequestFactoryInterface
{
    /**
     * Létrehoz egy új ServerRequestInterface típusú objektumpéldányt.
     *
     * Megjegyzendő, hogy a szerver paraméterei pontosan a megadott módon lesznek
     * rögzítve, az értékek feldolgozása/elemzése nem történik meg. Különösen
     * arra nem tesznek kísérletet, hogy meghatározzák a HTTP metódust vagy az URI-t,
     * ezeket ugyanis pontosan meg kell adni.
     *
     * @param string $method           A kéréshez tartozó HTTP metódus.
     * @param UriInterface|string $uri A kéréshez tartozó URI.
     * @param array $serverParams      A Server API (SAPI) paramétereinek tömbje, amellyel
     *                                 a létrehozott válasz példányt inicializálni lehet.
     */
    public function createServerRequest(string $method, $uri, array $serverParams = []): ServerRequestInterface;
}
```

### 2.4 StreamFactoryInterface

Képes PSR-7 stílusú adatfolyamok létrehozására HTTP kérés és válasz objektumok számára.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\StreamInterface;

interface StreamFactoryInterface
{
    /**
     * String típusú bemenetből létrehoz egy új StreamInterface típusú objektumpéldányt.
     *
     * Az adatfolyamot ideiglenes erőforrással KELLENE létrehozni.
     *
     * @param string $content Szöveges tartalom, amellyel az adatfolyam megtöltendő.
     */
    public function createStream(string $content = ''): StreamInterface;

    /**
     * Létező állományból létrehoz egy új StreamInterface típusú objektumpéldányt.
     *
     * Az állományt a megadott módon KELL megnyitni, ami lehet bármilyen mód, amit
     * a `fopen()` függvény támogat.
     *
     * A `$filename` LEHET bármely string, amit támogat a `fopen()` függvény.
     *
     * @param string $filename Az adatfolyam alapjául szolgáló állománynév vagy adatfolyam URI.
     * @param string $mode     A mód, amellyel a mögöttes file/adatfolyam megnyitandó.
     *
     * @throws \RuntimeException Ha az állomány nem nyitható meg.
     * @throws \InvalidArgumentException Ha a megadott mód érvénytelen.
     */
    public function createStreamFromFile(string $filename, string $mode = 'r'): StreamInterface;

    /**
     * Létező erőforrásból létrehoz egy új StreamInterface típusú objektumpéldányt.
     *
     * At adatfolyamnak olvashatónak KELL lennie és az sem árt, ha írható.
     *
     * @link https://www.php.net/manual/en/language.types.resource.php
     * @param resource $resource Az adatfolyam alapjául szolgáló PHP resource.
     */
    public function createStreamFromResource($resource): StreamInterface;
}
```

A jelen felület megvalósításainak ideiglenes adatfolyamot KELLENE alkalmazi, amikor
string típusú bemenetből hoznak létre új erőforrást. Az AJÁNLOTT módszer ez:

```php
$resource = fopen('php://temp', 'r+');
```

### 2.5 UploadedFileFactoryInterface

Képes adatfolyamok létrehozására feltöltött file objektumok számára.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\StreamInterface;
use Psr\Http\Message\UploadedFileInterface;

interface UploadedFileFactoryInterface
{
    /**
     * Léterhoz egy új UploadedFileInterface típusú objektumpéldányt.
     *
     * Ha a méret nincs megadva, akkor az adatfolyam méretéből lesz megállapítva.
     *
     * @link http://php.net/manual/features.file-upload.post-method.php
     * @link http://php.net/manual/features.file-upload.errors.php
     *
     * @param StreamInterface $stream  A feltöltött file tartalmát reprezentáló,
     *                                 alapul szolgáló adatfolyam.
     * @param int $size                Az állomány mérete byte-ban megadva.
     * @param int $error               PHP file feltöltés hiba.
     * @param string $clientFilename   A kliens által megadott állománynév, ha van.
     * @param string $clientMediaType  A kliens által megadott médiatípus, ha van.
     *
     * @throws \InvalidArgumentException Ha a file erőforrás nem olvasható.
     */
    public function createUploadedFile(
        StreamInterface $stream,
        int $size = null,
        int $error = \UPLOAD_ERR_OK,
        string $clientFilename = null,
        string $clientMediaType = null
    ): UploadedFileInterface;
}
```

### 2.6 UriFactoryInterface

Képes PSR-7 stílusú URI objektumok létrehozására kliens és szerver kérések számára.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\UriInterface;

interface UriFactoryInterface
{
    /**
     * Létrehoz egy új UriInterface típusú objektumpéldányt.
     *
     * @param string $uri Az értelmezendő URI.
     *
     * @throws \InvalidArgumentException Ha a megadott URI nem értelmezhető.
     */
    public function createUri(string $uri = '') : UriInterface;
}
```

[Kezdőlap](../README.md)
