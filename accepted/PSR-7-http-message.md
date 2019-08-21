[Kezdőlap](../README.md)

# HTTP üzenet interfészek

Ez a dokumentum azokat a közös programozási felületeket írja le, amelyek az ide
vonatkozó [RFC 7230](../related-rfcs/7230.md) és [RFC 7231](../related-rfcs/7231.md)
számú szabványok figyelembevételével ábrázolják a HTTP üzeneteket, különös tekintettel
az [URI](https://hu.wikipedia.org/wiki/URI) komponensre, amelyet az [STD 66](../related-rfcs/3986.md)
internetes szabvány határoz meg.

A HTTP üzenetek a webfejlesztés alappillérei. A böngészők és más HTTP kliens programok
(gyűjtőnéven: [user agent](https://hu.wikipedia.org/wiki/User_agent)), mint a [cURL](https://hu.wikipedia.org/wiki/CURL) egy szabványos HTTP kérelmet (Request)
hoznak létere és küldenek el a webszervernek, amely gondoskodik a szintén szabványos
HTTP válaszról (Response). A szerver oldali kód tehát megkapja a HTTP kérelmet,
majd egy HTTP válasz üzenettel reagál rá.

A HTTP üzenetek általában a végfelhasználóktól származnak, ezért a fejlesztőknek
pontosan kell ismerniük a szerkezetüket és tisztában kell lenniük azzal, hogyan
lehet hozzáférni ezekhez az üzenetekhez, illetve manipulálni őket feladataik
végrehajtása érdekében. Ezen felül azt sem árt tudni, hogy lehet egy kérelmet
létrehozni egy HTTP API számára, vagy kezelni a bejövő kérelmeket.

Minden HTTP kérelemnek van egy meghatározott
[formája](https://hu.wikipedia.org/wiki/HTTP#K%C3%A9r%C3%A9s_(request)):

~~~http
POST /path HTTP/1.1\r\n
Host: example.com\r\n
\r\n
foo=bar&baz=bat\r\n
~~~

A kérelem első sora („request line”) mindig „METÓDUS ERŐFORRÁS VERZIÓ” alakú. Első
helyen a [8 HTTP-metódus](https://hu.wikipedia.org/wiki/HTTP#Met%C3%B3dusok) egyike
szerepel, amely a megadott erőforráson végzendő műveletet határozza meg. Ezt követi
a kérelem célja, vagyis annak az erőforrásnak az azonosítója, amelyre a kérelem
irányul. Ez lehet abszolút URI vagy egy relatív elérési út a kiszolgálón. A sort
az alkalmazott HTTP protokoll verziószáma zárja. Ezt a sort követheti tetszőleges
számú HTTP fejléc sor („header line”) „FEJLÉCNÉV: ÉRTÉK” alakban, majd egy üres sor
után az üzenet törzse (ha van).

A [HTTP válasz](https://hu.wikipedia.org/wiki/HTTP#V%C3%A1lasz_(response)) üzenetek
ehhez hasonló szerkezettel rendelkeznek:

~~~http
HTTP/1.1 200 OK\r\n
Content-Type: text/plain\r\n
\r\n
This is the response body\r\n
~~~

A HTTP válasz első sora az állapotsor („status line”), amely
„VERZIÓ STÁTUSZKÓD INDOKLÁS” alakú. A sort az alkalmazott HTTP protokoll verziója
nyitja, ezt követi a [HTTP-állapotkód](https://hu.wikipedia.org/wiki/HTTP-%C3%A1llapotk%C3%B3dok)
(„status code”), ami egy háromjegyű szám. Az állapotsort az indoklás
(„reason phrase”) zárja, ami egy az állapotkódot magyarázó üzenet valamilyen emberi
nyelven, vagy esetleg angolul. A kérelemhez hasonlóan ezt a sort követheti tetszőleges
számú HTTP fejléc sor „FEJLÉCNÉV: ÉRTÉK” alakban, majd egy üres sor után az üzenet
törzse. A kliens elsősorban az állapotkód, másodsorban a fejléc sorok tartalma
alapján kezeli a választ.

A sorokat mind a kérelem, mind a válasz esetében a `SORVÉG` (CRLF, kocsi vissza + soremelés,
vagyis `\r\n`) karakterpárral kell elválasztani. A fejlécek végét jelző üres sor
csak ezt a két karaktert tartalmazhatja, nem lehet benne szóköz és tabulátor sem.

A jelen dokumentumban leírt interfészek a HTTP üzenetek és alkotórészeik körüli
absztrakciók.

A csupa nagybetűvel szedett, a követelmények szintjének jelzésére szolgáló kiemelt
kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

### Referenciák

- [RFC 2119](../related-rfcs/2119.md)
- [RFC 3986](../related-rfcs/3986.md)
- [RFC 7230](../related-rfcs/7230.md)
- [RFC 7231](../related-rfcs/7231.md)

## 1. Specifikáció

### 1.1 Üzenetek

Egy HTTP üzenet vagy egy kérelem a klienstől a szerver felé, vagy egy válasz a
szervertől a kliensnek. Ennek megfelelően ez a specifikáció a `Psr\Http\Message\RequestInterface` és `Psr\Http\Message\ResponseInterface` interfészeket definiálja a HTTP-üzenetek számára.
Mindkét interfész a `Psr\Http\Message\MessageInterface`-ből származik. Miközben a
`Psr\Http\Message\MessageInterface`-t közvetlenül LEHET implementálni, a megvalósítók
számára AJÁNLOTT a `Psr\Http\Message\RequestInterface` és `Psr\Http\Message\ResponseInterface`
implementálása is.

Innentől kezdve a `Psr\Http\Message` névtér előtag nélkül hivatkozunk ezekre az interfészekre.

### 1.2 HTTP fejlécek

#### A fejléc sorok nevei

Egy HTTP üzenet tartalmazhat tetszőleges számú fejléc sort is, melyek nevei nem kis-,
és nagybetű érzékenyek. A fejlécek olyan osztályokból kerülnek lekérésre, nem kis-,
és nagybetű érzékeny módon, amelyek implementálják a `MessageInterface`-t. Például
a `foo` fejlécet lekérve azonos eredményt fogunk kapni, mint a `FoO` lekérése
esetén. Hasonlóképpen a `Foo` fejléc beállítása felülírja a korábban beállított `foo`
fejléc értéket.

~~~php
$message = $message->withHeader('foo', 'bar');

echo $message->getHeaderLine('foo');
// Kimenet: bar

echo $message->getHeaderLine('FOO');
// Kimenet: bar

$message = $message->withHeader('fOO', 'baz');
echo $message->getHeaderLine('foo');
// Kimenet: baz
~~~

Annak ellenére, hogy a fejléceket nem kis-, és nagybetű érzékeny módon lehet
lekérni, az eredeti írásmódot meg KELL őrizni az implementációban, különösen amikor
`getHeaders()` metódussal kérjük le őket.

A nem megfelelő HTTP alkalmazások függhetnek bizonyos írásmódoktól, így hasznos
lehet, ha a felhasználó képes megszabni a HTTP fejlécek írásmódját, amikor
kérelmet vagy válasz üzenetet hoz létre.

#### Fejlécek többféle értékkel

A többféle értékkel rendelkező fejlécek befogadása érdekében, biztosítandó a kényelmes
munkát a szövegként kezelt fejlécekkel, a fejlécek szövegként vagy tömbként
lekérhetik őket egy `MessageInterface` példányból. A `getHeaderLine()` metódus
alkalmazásával szövegként lekérhető egy fejléc sor, amely vesszővel elválasztva
tartalmazza a megadott fejléchez tartozó összes értéket. A `getHeader()` metódus
ezzel szemben tömbként adja vissza ugyan azt, szintén nem kis-, és nagybetű
érzékeny módon.

~~~php
$message = $message
    ->withHeader('foo', 'bar')
    ->withAddedHeader('foo', 'baz');

$header = $message->getHeaderLine('foo');
// $header tartalma: 'bar, baz'

$header = $message->getHeader('foo');
// ['bar', 'baz']
~~~

Megjegyzés: nem minden fejléc értéket lehetséges vesszővel egymás után fűzve
szövegként visszadni (pl. `Set-Cookie`). Amikor ilyen fejlécekkel dolgozunk,
a `MessageInterface`-alapú osztályoknak AJÁNLOTT támaszkodni a `getHeader()`
metódusra az ilyen többféle értékkel rendelkező fejléc sorok lekérésénél.

#### Gazdagép (Host) fejléc

A kérelmekben a `Host` (gazdagép) fejlécsor jellemzően az URI
[azonos nevű](../related-rfcs/3986.md#322-gazdagép-host) komponensét, valamint a
a TCP kapcsolat létrehozásához használt gazdagépet tükrözi. A HTTP specifikáció
azonban lehetővé teszi, hogy a `Host` fejléc különbözzön ezektől.

Létrehozás közben az implementációknak meg KELL próbálni beállítani a `Host`
fejlécet a beszerzett URI-ből, ha egyébként nem áll rendelkezésre. Ezért a
`RequestInterface::withUri()` metódus alapértelmezés szerint kicseréli a visszaadott
`Host` fejlécet egy olyanra, amelyik megegyezik az `UriInterface` által átadott
`Host` komponenssel.

A metódus második (`$preserveHost`) argumentumaként megadott `true` értékkel lehetséges
megőrizni a `Host` fejléc eredeti állapotát is. Ebben az esetben a visszaadott kérelem
nem fogja felülírni az üzenet `Host` fejlécet, kivéve, ha az üzenet nem tartalmaz
ilyen fejlécet.

Az alábbi táblázat azt mutatja be, hogy a `getHeaderLine('Host')` hogy fog visszatérni
a `withUri()` metódus által visszaadott értékkel, ha a `$preserveHost` paraméter
értéke `true`.

Kérelem Host fejléc<sup>[1](#rhh)</sup> | Kérelem host összetevő<sup>[2](#rhc)</sup> | URI host összetevő<sup>[3](#uhc)</sup> | Eredmény
----------------------------------------|--------------------------------------------|----------------------------------------|--------
''                                      | ''                                         | ''                                     | ''
''                                      | foo.com                                    | ''                                     | foo.com
''                                      | foo.com                                    | bar.com                                | foo.com
foo.com                                 | ''                                         | bar.com                                | foo.com
foo.com                                 | bar.com                                    | baz.com                                | foo.com

- <sup id="rhh">1</sup> A `Host` fejléc értéke a végrehajtás előtt.
- <sup id="rhc">2</sup> Az URI végrehajtás előtt összeállított host összetevője.
- <sup id="uhc">3</sup> Az URI `withUri()` metódussal beinjektált host összetevője.

### 1.3 Adatfolyamok (stream)

A HTTP üzenetek kezdősorból (kérelem vagy állapotsor), fejlécekből és az üzenettörzsből
állnak. Ez utóbbi lehet egészen rövid, de szerfelett nagy is. Mivel az üzenettörzs
teljes egészében a memóriában lakik, ezért karakterláncként való ábrázolása könnyen
több memóriát vehet igénybe, mint szeretnénk. Ezért a kérelem vagy a válasz törzsének
memóriában tárolására tett kísérlet kizárná az olyan implementációkat, amelyek
képesek nagyobb üzenettörzsekkel dolgozni. A `StreamInterface` elrejti a megvalósítás
részleteit, amikor adatot olvasunk az adatfolyamból vagy írunk bele. Olyan helyzetekben,
amikor egy karakterlánc is megfelelő megoldás lenne az üzenettörzs megvalósítására,
a PHP beépített `php://memory` és `php://temp` adatfolyam-burkolóit (wrapper)
lehet alkalmazni.

A `StreamInterface` számos olyan metódust ír le, amely lehetővé teszi az adatfolyamok
olvasását, írását és hatékony bejárást.

Az adatfolyamok képességeit három metódus alkalmazásával lehet feltárni: `isReadable()`,
`isWritable()`, és `isSeekable()`. A kliens kód ezen metódusok segítségével győződhet
meg arról, hogy az adatfolyam valóban rendelkezik az elvárt képességekkel.

Minden `StreamInterface` példány különböző képességekkel rendelkezhet: van, amelyik
csak olvasható, csak írható, de akad olyan is, ami írható-olvasható. Lehetővé
tehetnek tetszőleges véletlen (előre vagy hátra keresve, bármely helyen), vagy kizárólag
szekvenciális (pl. csatlakozó, cső vagy callback-alapú adatfolyam esetén) hozzáférést is.

Végül a `StreamInterface` tartalmaz egy `__toString()` metódust is a tartalom
egyszerű szöveges visszaadására.

A `RequestInterface` és `ResponseInterface`-től eltérően a `StreamInterface` nem
követi az Immutable (változtathatatlan) objektum modellt. Azokban az esetekben,
amikor egy aktuális PHP adatfolyam be van csomagolva, az immutabilitást
lehetetlen megvalósítani, mivel bármelyik kód amelyik interakcióba lép az erőforrással,
potenciálisan megváltoztathatja annak állapotát (ideértve a kurzor pozíciójától
a tartalomig bármit).
A javaslatunk az, hogy az implementációk alkalmazzanak csak olvasható adatfolyamokat
a kiszolgáló oldali kérelmekhez és válaszokhoz. Az ügyfeleknek tisztában kell
lenniük azzal a ténnyel, hogy az adatfolyam-példány megváltoztatható és mint ilyen
az üzenet állapotát is megváltoztathatja. Kétség esetén új adatfolyam-példányt kell
létrehozni és azt az üzenethez csatolni, az állapot érvényesítése érdekében.

### 1.4 A kérelmek célpontjai és az URI-k

Az [RFC 7230](../related-rfcs/7230.md#53--request-target) előírja, hogy a kérelmek
első sorának ("request line") a metódus után tartalmaznia kell egy olyan szegmenst
ami a kérelem célját, vagyis annak az erőforrásnak az azonosítóját tartalmazza,
amelyre a kérelem irányul. A kérelem célját az alább felsorolt formátumokban lehet
ábrázolni:

- **eredeti-formátum**, amely tartalmaz elérési útvonalat (path) és ha van, akkor
  lekérdezési karakterláncot (query); ez az a formátum, amelyre gyakran relatív
  URL-ként hivatkoznak. A TCP protokollon keresztül továbbított üzenetek általában
  ilyen formátumúak; a sémára és a hitelesítésre vonatkozó adatok rendszerint csak
  CGI változók útján jelennek meg.
- **abszolút-formátum**, amely sémát, hitelesítési komponenst ("[user-info@]host[:port]",
  melyben a szögletes zárójelbe tett elemek opcionálisak), útvonalat (ha van),
  lekérdezési karakterláncot (ha van), és részerőforrás azonosítót (ha van) tartalmaz.
  Erre szoktak abszolút URL-ként hivatkozni és ez az egyedüli formátum, ahol az
  URI az [RFC 3986](../related-rfcs/3986.md) szabványban részletezett módon szerepel.
  Ezt a formátumot alkalmazzák a leggyakrabban a HTTP proxiknak küldendő kérelmekben is.
- **hitelesítési-formátum**, amely kizárólag hitelesítési komponenst tartalmaz.
  Ezt túlnyomórészt a CONNECT metódussal küldött kérelmeknél alkalmazzák, a kapcsolat
  felépítéséhez a HTTP kliens és a proxy szerver között.
- **csillag-formátum**, amely kizárólag egy csillag karakterből (`*`) áll és amit
  az OPTIONS metódussal használnak, a webszerver általános képességeinek meghatározására.

A kérelem célján kívül, attól elkülönülve létezik 'tényleges URL' is. Ezt nem
szokták átküldeni a HTTP üzeneten belül, mivel a kérelem által használt
protokoll (http/https), port és gazdagépnév meghatározására való.

A tényleges URL-t az `UriInterface` reprezentálja, modellezve a HTTP és HTTPS URI-t
az [RFC 3986](../related-rfcs/3986.md) internetes szabványnak megfelelően. Az interfész
biztosítja a megfelelő metódusokat az URI különböző szegmenseivel történő interakciókhoz,
elkerülendő az URI ismételt beolvasását. Ezen felül előírja a `__toString()` metódus
megvalósítását is a modellezett URI megfelelő szöveges formátumú megjelenítése
érdekében.

Amikor a `getRequestTarget()` metódus segítségével lekérjük a kérelem célját,
a metódus alapértelmezés szerint egy URI objektumot fog használni, hogy kinyerje
belőle a szükséges összetevőket az _eredeti-formátum_ban történő szöveges
megjelenítésére. Az _eredeti-formátum_ messze a leggyakoribb a kérelem célpontok között.

Ha a végfelhasználó használni kívánja a másik három formátum valamelyikét, vagy
ha kifejezetten felül akarja írni a kérelem célját, ezt a `withRequestTarget()`
metódus segítségével teheti meg. E metódus meghívása nem érinti a `getUri()` metódus
által visszaadott URI-t.

Az alábbi példában a felhasználó egy csillag-formátumú kérelmet szeretne küldeni
a szervernek:

~~~php
$request = $request
    ->withMethod('OPTIONS')
    ->withRequestTarget('*')
    ->withUri(new Uri('https://example.org/'));
~~~

Ez a művelet végül ezt a HTTP kérelmet fogja eredményezni:

~~~http
OPTIONS * HTTP/1.1
~~~

De a HTTP klienskód képes lesz használni a `getUri()` metódus által visszaadott
tényleges URL-t a protokoll, a TCP port és a gazdagép meghatározására.

Egy HTTP klienskódnak figyelmen kívül KELL hagynia a `Uri::getPath()` és `Uri::getQuery()`
metódushívások visszatérési értékét és helyette a `getRequestTarget()` által
visszaadott értéket használni, ami alapértelmezés szerint előbbi kettő összefűzéséből
áll elő.

Azon klienskódoknak amelyek úgy döntenek, hogy nem alkalmaznak legalább egyet a
fent felsorolt 4 kérelem-célpont formátumból, továbbra is alkalmazniuk kell a
`getRequestTarget()` metódust. Ezen a klienseknek vissza KELL utasítani a nem
támogatott kérelem-célpontokat és TILOS visszatérniük a `getUri()` metódusból nyert
értékekkel.

A `RequestInterface` biztosítja a kérelem-célpont kinyeréséhez vagy adott célponttal
való új példány létrehozásához szükséges metódusokat. Alapértelmezés szerint, ha
nincs kifejezett kérelem-célpont megadva az objektumpéldányban, akkor a `getRequestTarget()`
metódus az összeállított URI eredeti-formátum szerinti alakjával fog visszatérni
(vagy egy perjellel "/", ha nincs URI megadva). A `withRequestTarget($requestTarget)`
metódus létrehoz egy új példányt a megadott célponttal és így lehetővé teszi a
fejlesztőknek olyan kérelem üzenetek létrehozását is, amelyek a másik három
kérelem-célpont formátumot (abszolút-, hitelesítési-, és csillag-formátum) jelenítik
meg. Megfelelő használat esetén az összeállított URI példány továbbra is hasznos
lehet, különösen klienskódokban, amelyekben felhasználható a szerverrel való kapcsolat
felépítésére.

### 1.5 Kiszolgáló oldali kérelmek

A `RequestInterface` biztosítja egy HTTP kérelem általános ábrázolását. Azonban
a kiszolgáló oldali üzeneteknek a környezetük jellegéből adódóan további eljárásokra
is szükségük van. Figyelembe kell venniük a [Common Gateway Interface (CGI)](http://webprogramozas.inf.elte.hu/tananyag/wf2/lecke12_lap1.html#hiv9)
protokollt is, közelebbről a PHP CGI absztrakcióit és kiterjesztéseit melyek a
Szerver API-k (SAPI) segítségével érhetők el. A PHP az alábbi szuperglobális
tömbjeivel jelentősen leegyszerűsíti a bejövő adatok rendezését:

- a `$_COOKIE`, biztosítja az egyszerű hozzáférést a deszerializált HTTP sütikhez.
- a `$_GET`, biztosítja az egyszerű hozzáférést a HTTP GET metódussal érkező adatokhoz
  (a kulcs-érték párokra bontott `QUERY_STRING`). A deszerializált adatok a `$_GET`
  tömbben url-dekódolva vannak jelen.
- a `$_POST`, biztosítja az egyszerű hozzáférést a deszerializált, HTTP POST metódussal
  érkező, `application/x-www-form-urlencoded` vagy `multipart/form-data` típusú
  URL-kódolt adatokhoz.
- a `$_FILES`, biztosítja a felhasználók által a HTTP POST metódussal feltöltött
  állományok szerializált metaadatait.
- a `$_SERVER`, hozzáférést biztosít a szerver és a futtatási környezet (CGI/SAPI)
  változóihoz,  fejléceket, útvonalakat és a futó scriptre vonatkozó információkat
  tartalmaz. A QUERY_STRING és REQUEST_URI kulccsal azonosított elemei az [STD
  66](../related-rfcs/3986.md#21-url-kódolás) internetes szabvány szerint
  url-kódolva vannak.

A `ServerRequestInterface` a `RequestInterface` kiterjesztése, amely biztosítja
a különféle szuperglobálisokhoz szükséges absztrakciókat. Ezek segítségével
csökkenthető a hívó kód szuperglobálisokhoz való közvetlen kapcsolódása és
javul a tesztelhetősége.

A kiszolgáló oldali kérelem biztosít egy további "attribute" tulajdonságot is, amely
lehetővé teszi a hívó kódnak, hogy elemezze, lebontsa és párosítsa a kérelmet az
alkalmazás-specifikus szabályokkal (pl. útvonal, séma, gazdagép, stb. illeszkedés).
Mint ilyen, a kiszolgáló oldali kérelem üzenetküldőt is biztosíthat a hívó kódok
között.

### 1.6 Feltöltött állományok

A `ServerRequestInterface` többek közt előírja a `getUploadedFiles()` metódus
megvalósítását is, amely a feltöltött állományok adatait egy normalizált struktúrában
képes visszaadni, ahol minden egyes ágon az `UploadedFileInterface` egy-egy példánya
reprezentálja a feltöltött állományokat.

A `$_FILES` szuperglobális tömb rendelkezik néhány jól ismert hiányossággal a
beérkező fájlok tömbjeinek kezelése kapcsán. Például, ha van egy űrlapunk, amely
egy tömbben küldi el a feltöltött állományokat — ha az input mező neve mondjuk
"files", akkor `files[0]` és `files[1]` kulcs alatt — akkor ezt a PHP az alábbiak
szerint fogja megjeleníteni:

~~~php
array(
  'files' => array(
    'name' => array(
      0 => 'file0.txt',
      1 => 'file1.html',
    ),
    'type' => array(
      0 => 'text/plain',
      1 => 'text/html',
    ),
    /* etc. */
  ),
)
~~~

habár mi inkább valami ilyesmire számítottunk:

~~~php
array(
  'files' => array(
    0 => array(
      'name' => 'file0.txt',
      'type' => 'text/plain',
      /* etc. */
    ),
    1 => array(
      'name' => 'file1.html',
      'type' => 'text/html',
      /* etc. */
    ),
  ),
)
~~~

Az eredmény az, hogy a felhasználóknak részletekbe menően ismerniük kell ezt a
nyelvi eszközt és olyan kódot írni, amely alkalmas a feltöltött adatok kigyűjtésére.

Emellett léteznek olyan forgatókönyvek is, ahol a `$_FILES` tömb üres marad, annak
ellenére, hogy fájlfeltöltés történt. Ilyen eset előfordulhat:

- amikor a feltöltés nem a HTTP `POST` metódussal történik.
- egységteszteléskor.
- amikor nem SAPI környezet, hanem mondjuk [ReactPHP](http://reactphp.org) alatt
  üzemel az alkalmazás.

Ilyen esetekben az adatokat másképpen kell kezelni. Néhány példa erre:

- Egy folyamat elemezheti az üzenet törzsét fájlfeltöltések után kutatva. Ilyen
  esetekben az implementáció dönthet úgy, hogy *nem* írja a feltöltött fájlokat
  az állományrendszerbe, inkább becsomagolja őket egy adatfolyamba a memória és a
  tárhelyfelhasználás csökkentése érdekében.
- Egységtesztelés esetén a különböző forgatókönyvek érvényesítése és ellenőrzése
  érdekében a fejlesztőknek képesnek kell lenniük [csonkolni és/vagy mókolni](https://hu.wikipedia.org/wiki/Mock_objektum) a feltöltött állományok
  metaadatait.

A `getUploadedFiles()` metódus normalizált struktúrát biztosít a felhasználók
számára. Az implementációktól elvárható, hogy:

- összesítsenek minden információt egy adott fájlfeltöltésről és helyezzék el azokat
  egy `Psr\Http\Message\UploadedFileInterface` példányba.
- újraalkossák a feltöltött fa-struktúrát, melyben minden egyes ág egy megfelelő
  `Psr\Http\Message\UploadedFileInterface` példány az adott helyen.

A hivatkozott fa struktúrának utánoznia kell azt az elnevezési szerkezetet, mellyel
az állományokat elküldték.

Egy egyszerű példában ez lehet egy elnevezett űrlap-elem, melyet az alábbi formában
küldenek el:

~~~html
<input type="file" name="avatar" />
~~~

Ebben az esetben a `$_FILES` tömb szerkezete így nézne ki:

~~~php
array(
  'avatar' => array(
    'tmp_name' => 'phpUxcOty',
    'name' => 'my-avatar.png',
    'size' => 90996,
    'type' => 'image/png',
    'error' => 0,
  ),
)
~~~

A `getUploadedFiles()` metódus által visszaadott normalizált forma viszont így fog
festeni:

~~~php
array(
  'avatar' => /* UploadedFileInterface példánya */
)
~~~

Abban az esetben, ha az input mező elnevezésénél az alábbi példához hasonló
tömb-jelölést használunk:

~~~html
<input type="file" name="my-form[details][avatar]" />
~~~

a `$_FILES` a végén ehhez hasonló lesz:

~~~php
array (
  'my-form' => array (
    'name' => array (
      'details' => array (
        'avatar' => 'my-avatar.png',
      ),
    ),
    'type' => array (
      'details' => array (
        'avatar' => 'image/png',
      ),
    ),
    'tmp_name' => array (
      'details' => array (
        'avatar' => 'phpmFLrzD',
      ),
    ),
    'error' => array (
      'details' => array (
        'avatar' => 0,
      ),
    ),
    'size' => array (
      'details' => array (
        'avatar' => 90996,
      ),
    ),
  ),
)
~~~

És az ennek megfelelő fa, amelyet a `getUploadedFiles()` metódus visszaad:

~~~php
array(
  'my-form' => array(
    'details' => array(
      'avatar' => /* az UploadedFileInterface példánya */
    ),
  ),
)
~~~

Némely esetben létrehozhatunk egy tömböt is az állományoknak:

~~~html
Avatar kép feltöltése: <input type="file" name="my-form[details][avatars][]" />
Avatar kép feltöltése: <input type="file" name="my-form[details][avatars][]" />
~~~

(Megeshet, hogy a JavaScript vezérlők további input mezőket is generálnak, hogy
lehetővé tegyék az egyszerre történő többszörös fájlfeltöltést.)

Ilyen esetben a specifikáció megvalósításának össze kell gyűjteni az adott indexhez
minden a fájlhoz kapcsolódó információt. Ennek az az oka, hogy a `$_FILES` tömb
felépítése eltér a normál struktúrától:

~~~php
array (
  'my-form' => array (
    'name' => array (
      'details' => array (
        'avatar' => array (
          0 => 'my-avatar.png',
          1 => 'my-avatar2.png',
          2 => 'my-avatar3.png',
        ),
      ),
    ),
    'type' => array (
      'details' => array (
        'avatar' => array (
          0 => 'image/png',
          1 => 'image/png',
          2 => 'image/png',
        ),
      ),
    ),
    'tmp_name' => array (
      'details' => array (
        'avatar' => array (
          0 => 'phpmFLrzD',
          1 => 'phpV2pBil',
          2 => 'php8RUG8v',
        ),
      ),
    ),
    'error' => array (
      'details' => array (
        'avatar' => array (
          0 => 0,
          1 => 0,
          2 => 0,
        ),
      ),
    ),
    'size' => array (
      'details' => array (
        'avatar' => array (
          0 => 90996,
          1 => 90996,
          3 => 90996,
        ),
      ),
    ),
  ),
)
~~~

A fenti `$_FILES` tömb megfelelne a következő struktúrának, amelyet a `getUploadedFiles()`
metódus ad vissza:

~~~php
array(
  'my-form' => array(
    'details' => array(
      'avatars' => array(
        0 => /* UploadedFileInterface példánya */,
        1 => /* UploadedFileInterface példánya */,
        2 => /* UploadedFileInterface példánya */,
      ),
    ),
  ),
)
~~~

A felhasználók a beágyazott tömb `1` indexű eleméhez a következő kifejezés segítségével
tudnak hozzáférni:

~~~php
$request->getUploadedFiles()['my-form']['details']['avatars'][1];
~~~

Mivel a feltöltött állományok adatai származtatottak (a `$_FILES` tömbből vagy a
kérelem törzséből származnak), ezért az interfész `withUploadedFiles()` metódus
implementálását is megköveteli, lehetővé a normalizáció kiszervezését más
folyamatok részére.

Az eredeti példa esetében a felhasználás a következőhöz lesz hasonló:

~~~php
$file0 = $request->getUploadedFiles()['files'][0];
$file1 = $request->getUploadedFiles()['files'][1];

printf(
  "A %s és %s állományok feltöltve",
  $file0->getClientFilename(),
  $file1->getClientFilename()
);

//Kimenet: "A file0.txt és file1.html állományok feltöltve"
~~~

A jelen ajánlás elismeri, hogy a belőle származó implementációk működhetnek nem SAPI
környezetben is. Ezért az `UploadedFileInterface` olyan metódusokat is előír,
amelyek lehetővé teszik a helyes működést, függetlenül a környezettől. Ilyenek
metódusok:

- a `moveTo($targetPath)` metódus ajánlott és biztonságos alternatívát biztosít a
  `move_uploaded_file()` függvény átmeneti feltöltött fájlon történő meghívásával
  szemben. Az implementációknak azonban a környezet alapján fel kell ismerniük a
  helyes működést.
- a `getStream()` metódus egy `StreamInterface` példánnyal tér vissza. Nem SAPI
  környezetekben az egyik javasolt lehetőség az egyes feltöltött fájlok direkt
  beolvasása helyett a `php://temp` adatfolyam használata; ilyen esetben nincs jelen
  feltöltött állomány. A `getStream()` ezért garantáltan működik minden környezetben.

Példák:

~~~
// Fájl mozgatása a feltöltési könyvtárba
$filename = sprintf(
    '%s.%s',
    create_uuid(),
    pathinfo($file0->getClientFilename(), PATHINFO_EXTENSION)
);
$file0->moveTo(DATA_DIR . '/' . $filename);

// Stream a file to Amazon S3.
// Assume $s3wrapper is a PHP stream that will write to S3, and that
// Psr7StreamWrapper is a class that will decorate a StreamInterface as a PHP
// StreamWrapper.
$stream = new Psr7StreamWrapper($file1->getStream());
stream_copy_to_stream($stream, $s3wrapper);
~~~

## 2. A csomag

A szükséges interfészek és osztályok rendelkezésre állnak a [psr/http-message](https://packagist.org/packages/psr/http-message) csomag részeként.

Telepítése (Composer segítségével, terminálon): `composer require psr/http-message`.

## 3. Interfészek

### 3.1 `Psr\Http\Message\MessageInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * HTTP messages consist of requests from a client to a server and responses
 * from a server to a client. This interface defines the methods common to
 * each.
 *
 * Messages are considered immutable; all methods that might change state MUST
 * be implemented such that they retain the internal state of the current
 * message and return an instance that contains the changed state.
 *
 * @see http://www.ietf.org/rfc/rfc7230.txt
 * @see http://www.ietf.org/rfc/rfc7231.txt
 */
interface MessageInterface
{
    /**
     * Retrieves the HTTP protocol version as a string.
     *
     * The string MUST contain only the HTTP version number (e.g., "1.1", "1.0").
     *
     * @return string HTTP protocol version.
     */
    public function getProtocolVersion();

    /**
     * Return an instance with the specified HTTP protocol version.
     *
     * The version string MUST contain only the HTTP version number (e.g.,
     * "1.1", "1.0").
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * new protocol version.
     *
     * @param string $version HTTP protocol version
     * @return static
     */
    public function withProtocolVersion($version);

    /**
     * Retrieves all message header values.
     *
     * The keys represent the header name as it will be sent over the wire, and
     * each value is an array of strings associated with the header.
     *
     *     // Represent the headers as a string
     *     foreach ($message->getHeaders() as $name => $values) {
     *         echo $name . ': ' . implode(', ', $values);
     *     }
     *
     *     // Emit headers iteratively:
     *     foreach ($message->getHeaders() as $name => $values) {
     *         foreach ($values as $value) {
     *             header(sprintf('%s: %s', $name, $value), false);
     *         }
     *     }
     *
     * While header names are not case-sensitive, getHeaders() will preserve the
     * exact case in which headers were originally specified.
     *
     * @return string[][] Returns an associative array of the message's headers.
     *     Each key MUST be a header name, and each value MUST be an array of
     *     strings for that header.
     */
    public function getHeaders();

    /**
     * Checks if a header exists by the given case-insensitive name.
     *
     * @param string $name Case-insensitive header field name.
     * @return bool Returns true if any header names match the given header
     *     name using a case-insensitive string comparison. Returns false if
     *     no matching header name is found in the message.
     */
    public function hasHeader($name);

    /**
     * Retrieves a message header value by the given case-insensitive name.
     *
     * This method returns an array of all the header values of the given
     * case-insensitive header name.
     *
     * If the header does not appear in the message, this method MUST return an
     * empty array.
     *
     * @param string $name Case-insensitive header field name.
     * @return string[] An array of string values as provided for the given
     *    header. If the header does not appear in the message, this method MUST
     *    return an empty array.
     */
    public function getHeader($name);

    /**
     * Retrieves a comma-separated string of the values for a single header.
     *
     * This method returns all of the header values of the given
     * case-insensitive header name as a string concatenated together using
     * a comma.
     *
     * NOTE: Not all header values may be appropriately represented using
     * comma concatenation. For such headers, use getHeader() instead
     * and supply your own delimiter when concatenating.
     *
     * If the header does not appear in the message, this method MUST return
     * an empty string.
     *
     * @param string $name Case-insensitive header field name.
     * @return string A string of values as provided for the given header
     *    concatenated together using a comma. If the header does not appear in
     *    the message, this method MUST return an empty string.
     */
    public function getHeaderLine($name);

    /**
     * Return an instance with the provided value replacing the specified header.
     *
     * While header names are case-insensitive, the casing of the header will
     * be preserved by this function, and returned from getHeaders().
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * new and/or updated header and value.
     *
     * @param string $name Case-insensitive header field name.
     * @param string|string[] $value Header value(s).
     * @return static
     * @throws \InvalidArgumentException for invalid header names or values.
     */
    public function withHeader($name, $value);

    /**
     * Return an instance with the specified header appended with the given value.
     *
     * Existing values for the specified header will be maintained. The new
     * value(s) will be appended to the existing list. If the header did not
     * exist previously, it will be added.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * new header and/or value.
     *
     * @param string $name Case-insensitive header field name to add.
     * @param string|string[] $value Header value(s).
     * @return static
     * @throws \InvalidArgumentException for invalid header names.
     * @throws \InvalidArgumentException for invalid header values.
     */
    public function withAddedHeader($name, $value);

    /**
     * Return an instance without the specified header.
     *
     * Header resolution MUST be done without case-sensitivity.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that removes
     * the named header.
     *
     * @param string $name Case-insensitive header field name to remove.
     * @return static
     */
    public function withoutHeader($name);

    /**
     * Gets the body of the message.
     *
     * @return StreamInterface Returns the body as a stream.
     */
    public function getBody();

    /**
     * Return an instance with the specified message body.
     *
     * The body MUST be a StreamInterface object.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return a new instance that has the
     * new body stream.
     *
     * @param StreamInterface $body Body.
     * @return static
     * @throws \InvalidArgumentException When the body is not valid.
     */
    public function withBody(StreamInterface $body);
}
~~~

### 3.2 `Psr\Http\Message\RequestInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Representation of an outgoing, client-side request.
 *
 * Per the HTTP specification, this interface includes properties for
 * each of the following:
 *
 * - Protocol version
 * - HTTP method
 * - URI
 * - Headers
 * - Message body
 *
 * During construction, implementations MUST attempt to set the Host header from
 * a provided URI if no Host header is provided.
 *
 * Requests are considered immutable; all methods that might change state MUST
 * be implemented such that they retain the internal state of the current
 * message and return an instance that contains the changed state.
 */
interface RequestInterface extends MessageInterface
{
    /**
     * Retrieves the message's request target.
     *
     * Retrieves the message's request-target either as it will appear (for
     * clients), as it appeared at request (for servers), or as it was
     * specified for the instance (see withRequestTarget()).
     *
     * In most cases, this will be the origin-form of the composed URI,
     * unless a value was provided to the concrete implementation (see
     * withRequestTarget() below).
     *
     * If no URI is available, and no request-target has been specifically
     * provided, this method MUST return the string "/".
     *
     * @return string
     */
    public function getRequestTarget();

    /**
     * Return an instance with the specific request-target.
     *
     * If the request needs a non-origin-form request-target — e.g., for
     * specifying an absolute-form, authority-form, or asterisk-form —
     * this method may be used to create an instance with the specified
     * request-target, verbatim.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * changed request target.
     *
     * @see http://tools.ietf.org/html/rfc7230#section-5.3 (for the various
     *     request-target forms allowed in request messages)
     * @param mixed $requestTarget
     * @return static
     */
    public function withRequestTarget($requestTarget);

    /**
     * Retrieves the HTTP method of the request.
     *
     * @return string Returns the request method.
     */
    public function getMethod();

    /**
     * Return an instance with the provided HTTP method.
     *
     * While HTTP method names are typically all uppercase characters, HTTP
     * method names are case-sensitive and thus implementations SHOULD NOT
     * modify the given string.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * changed request method.
     *
     * @param string $method Case-sensitive method.
     * @return static
     * @throws \InvalidArgumentException for invalid HTTP methods.
     */
    public function withMethod($method);

    /**
     * Retrieves the URI instance.
     *
     * This method MUST return a UriInterface instance.
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @return UriInterface Returns a UriInterface instance
     *     representing the URI of the request.
     */
    public function getUri();

    /**
     * Returns an instance with the provided URI.
     *
     * This method MUST update the Host header of the returned request by
     * default if the URI contains a host component. If the URI does not
     * contain a host component, any pre-existing Host header MUST be carried
     * over to the returned request.
     *
     * You can opt-in to preserving the original state of the Host header by
     * setting `$preserveHost` to `true`. When `$preserveHost` is set to
     * `true`, this method interacts with the Host header in the following ways:
     *
     * - If the Host header is missing or empty, and the new URI contains
     *   a host component, this method MUST update the Host header in the returned
     *   request.
     * - If the Host header is missing or empty, and the new URI does not contain a
     *   host component, this method MUST NOT update the Host header in the returned
     *   request.
     * - If a Host header is present and non-empty, this method MUST NOT update
     *   the Host header in the returned request.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * new UriInterface instance.
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @param UriInterface $uri New request URI to use.
     * @param bool $preserveHost Preserve the original state of the Host header.
     * @return static
     */
    public function withUri(UriInterface $uri, $preserveHost = false);
}
~~~

#### 3.2.1 `Psr\Http\Message\ServerRequestInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Representation of an incoming, server-side HTTP request.
 *
 * Per the HTTP specification, this interface includes properties for
 * each of the following:
 *
 * - Protocol version
 * - HTTP method
 * - URI
 * - Headers
 * - Message body
 *
 * Additionally, it encapsulates all data as it has arrived at the
 * application from the CGI and/or PHP environment, including:
 *
 * - The values represented in $_SERVER.
 * - Any cookies provided (generally via $_COOKIE)
 * - Query string arguments (generally via $_GET, or as parsed via parse_str())
 * - Upload files, if any (as represented by $_FILES)
 * - Deserialized body parameters (generally from $_POST)
 *
 * $_SERVER values MUST be treated as immutable, as they represent application
 * state at the time of request; as such, no methods are provided to allow
 * modification of those values. The other values provide such methods, as they
 * can be restored from $_SERVER or the request body, and may need treatment
 * during the application (e.g., body parameters may be deserialized based on
 * content type).
 *
 * Additionally, this interface recognizes the utility of introspecting a
 * request to derive and match additional parameters (e.g., via URI path
 * matching, decrypting cookie values, deserializing non-form-encoded body
 * content, matching authorization headers to users, etc). These parameters
 * are stored in an "attributes" property.
 *
 * Requests are considered immutable; all methods that might change state MUST
 * be implemented such that they retain the internal state of the current
 * message and return an instance that contains the changed state.
 */
interface ServerRequestInterface extends RequestInterface
{
    /**
     * Retrieve server parameters.
     *
     * Retrieves data related to the incoming request environment,
     * typically derived from PHP's $_SERVER superglobal. The data IS NOT
     * REQUIRED to originate from $_SERVER.
     *
     * @return array
     */
    public function getServerParams();

    /**
     * Retrieve cookies.
     *
     * Retrieves cookies sent by the client to the server.
     *
     * The data MUST be compatible with the structure of the $_COOKIE
     * superglobal.
     *
     * @return array
     */
    public function getCookieParams();

    /**
     * Return an instance with the specified cookies.
     *
     * The data IS NOT REQUIRED to come from the $_COOKIE superglobal, but MUST
     * be compatible with the structure of $_COOKIE. Typically, this data will
     * be injected at instantiation.
     *
     * This method MUST NOT update the related Cookie header of the request
     * instance, nor related values in the server params.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * updated cookie values.
     *
     * @param array $cookies Array of key/value pairs representing cookies.
     * @return static
     */
    public function withCookieParams(array $cookies);

    /**
     * Retrieve query string arguments.
     *
     * Retrieves the deserialized query string arguments, if any.
     *
     * Note: the query params might not be in sync with the URI or server
     * params. If you need to ensure you are only getting the original
     * values, you may need to parse the query string from `getUri()->getQuery()`
     * or from the `QUERY_STRING` server param.
     *
     * @return array
     */
    public function getQueryParams();

    /**
     * Return an instance with the specified query string arguments.
     *
     * These values SHOULD remain immutable over the course of the incoming
     * request. They MAY be injected during instantiation, such as from PHP's
     * $_GET superglobal, or MAY be derived from some other value such as the
     * URI. In cases where the arguments are parsed from the URI, the data
     * MUST be compatible with what PHP's parse_str() would return for
     * purposes of how duplicate query parameters are handled, and how nested
     * sets are handled.
     *
     * Setting query string arguments MUST NOT change the URI stored by the
     * request, nor the values in the server params.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * updated query string arguments.
     *
     * @param array $query Array of query string arguments, typically from
     *     $_GET.
     * @return static
     */
    public function withQueryParams(array $query);

    /**
     * Retrieve normalized file upload data.
     *
     * This method returns upload metadata in a normalized tree, with each leaf
     * an instance of Psr\Http\Message\UploadedFileInterface.
     *
     * These values MAY be prepared from $_FILES or the message body during
     * instantiation, or MAY be injected via withUploadedFiles().
     *
     * @return array An array tree of UploadedFileInterface instances; an empty
     *     array MUST be returned if no data is present.
     */
    public function getUploadedFiles();

    /**
     * Create a new instance with the specified uploaded files.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * updated body parameters.
     *
     * @param array $uploadedFiles An array tree of UploadedFileInterface instances.
     * @return static
     * @throws \InvalidArgumentException if an invalid structure is provided.
     */
    public function withUploadedFiles(array $uploadedFiles);

    /**
     * Retrieve any parameters provided in the request body.
     *
     * If the request Content-Type is either application/x-www-form-urlencoded
     * or multipart/form-data, and the request method is POST, this method MUST
     * return the contents of $_POST.
     *
     * Otherwise, this method may return any results of deserializing
     * the request body content; as parsing returns structured content, the
     * potential types MUST be arrays or objects only. A null value indicates
     * the absence of body content.
     *
     * @return null|array|object The deserialized body parameters, if any.
     *     These will typically be an array or object.
     */
    public function getParsedBody();

    /**
     * Return an instance with the specified body parameters.
     *
     * These MAY be injected during instantiation.
     *
     * If the request Content-Type is either application/x-www-form-urlencoded
     * or multipart/form-data, and the request method is POST, use this method
     * ONLY to inject the contents of $_POST.
     *
     * The data IS NOT REQUIRED to come from $_POST, but MUST be the results of
     * deserializing the request body content. Deserialization/parsing returns
     * structured data, and, as such, this method ONLY accepts arrays or objects,
     * or a null value if nothing was available to parse.
     *
     * As an example, if content negotiation determines that the request data
     * is a JSON payload, this method could be used to create a request
     * instance with the deserialized parameters.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * updated body parameters.
     *
     * @param null|array|object $data The deserialized body data. This will
     *     typically be in an array or object.
     * @return static
     * @throws \InvalidArgumentException if an unsupported argument type is
     *     provided.
     */
    public function withParsedBody($data);

    /**
     * Retrieve attributes derived from the request.
     *
     * The request "attributes" may be used to allow injection of any
     * parameters derived from the request: e.g., the results of path
     * match operations; the results of decrypting cookies; the results of
     * deserializing non-form-encoded message bodies; etc. Attributes
     * will be application and request specific, and CAN be mutable.
     *
     * @return mixed[] Attributes derived from the request.
     */
    public function getAttributes();

    /**
     * Retrieve a single derived request attribute.
     *
     * Retrieves a single derived request attribute as described in
     * getAttributes(). If the attribute has not been previously set, returns
     * the default value as provided.
     *
     * This method obviates the need for a hasAttribute() method, as it allows
     * specifying a default value to return if the attribute is not found.
     *
     * @see getAttributes()
     * @param string $name The attribute name.
     * @param mixed $default Default value to return if the attribute does not exist.
     * @return mixed
     */
    public function getAttribute($name, $default = null);

    /**
     * Return an instance with the specified derived request attribute.
     *
     * This method allows setting a single derived request attribute as
     * described in getAttributes().
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * updated attribute.
     *
     * @see getAttributes()
     * @param string $name The attribute name.
     * @param mixed $value The value of the attribute.
     * @return static
     */
    public function withAttribute($name, $value);

    /**
     * Return an instance that removes the specified derived request attribute.
     *
     * This method allows removing a single derived request attribute as
     * described in getAttributes().
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that removes
     * the attribute.
     *
     * @see getAttributes()
     * @param string $name The attribute name.
     * @return static
     */
    public function withoutAttribute($name);
}
~~~

### 3.3 `Psr\Http\Message\ResponseInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Representation of an outgoing, server-side response.
 *
 * Per the HTTP specification, this interface includes properties for
 * each of the following:
 *
 * - Protocol version
 * - Status code and reason phrase
 * - Headers
 * - Message body
 *
 * Responses are considered immutable; all methods that might change state MUST
 * be implemented such that they retain the internal state of the current
 * message and return an instance that contains the changed state.
 */
interface ResponseInterface extends MessageInterface
{
    /**
     * Gets the response status code.
     *
     * The status code is a 3-digit integer result code of the server's attempt
     * to understand and satisfy the request.
     *
     * @return int Status code.
     */
    public function getStatusCode();

    /**
     * Return an instance with the specified status code and, optionally, reason phrase.
     *
     * If no reason phrase is specified, implementations MAY choose to default
     * to the RFC 7231 or IANA recommended reason phrase for the response's
     * status code.
     *
     * This method MUST be implemented in such a way as to retain the
     * immutability of the message, and MUST return an instance that has the
     * updated status and reason phrase.
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @param int $code The 3-digit integer result code to set.
     * @param string $reasonPhrase The reason phrase to use with the
     *     provided status code; if none is provided, implementations MAY
     *     use the defaults as suggested in the HTTP specification.
     * @return static
     * @throws \InvalidArgumentException For invalid status code arguments.
     */
    public function withStatus($code, $reasonPhrase = '');

    /**
     * Gets the response reason phrase associated with the status code.
     *
     * Because a reason phrase is not a required element in a response
     * status line, the reason phrase value MAY be empty. Implementations MAY
     * choose to return the default RFC 7231 recommended reason phrase (or those
     * listed in the IANA HTTP Status Code Registry) for the response's
     * status code.
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @return string Reason phrase; must return an empty string if none present.
     */
    public function getReasonPhrase();
}
~~~

### 3.4 `Psr\Http\Message\StreamInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Describes a data stream.
 *
 * Typically, an instance will wrap a PHP stream; this interface provides
 * a wrapper around the most common operations, including serialization of
 * the entire stream to a string.
 */
interface StreamInterface
{
    /**
     * Reads all data from the stream into a string, from the beginning to end.
     *
     * This method MUST attempt to seek to the beginning of the stream before
     * reading data and read the stream until the end is reached.
     *
     * Warning: This could attempt to load a large amount of data into memory.
     *
     * This method MUST NOT raise an exception in order to conform with PHP's
     * string casting operations.
     *
     * @see http://php.net/manual/en/language.oop5.magic.php#object.tostring
     * @return string
     */
    public function __toString();

    /**
     * Closes the stream and any underlying resources.
     *
     * @return void
     */
    public function close();

    /**
     * Separates any underlying resources from the stream.
     *
     * After the stream has been detached, the stream is in an unusable state.
     *
     * @return resource|null Underlying PHP stream, if any
     */
    public function detach();

    /**
     * Get the size of the stream if known.
     *
     * @return int|null Returns the size in bytes if known, or null if unknown.
     */
    public function getSize();

    /**
     * Returns the current position of the file read/write pointer
     *
     * @return int Position of the file pointer
     * @throws \RuntimeException on error.
     */
    public function tell();

    /**
     * Returns true if the stream is at the end of the stream.
     *
     * @return bool
     */
    public function eof();

    /**
     * Returns whether or not the stream is seekable.
     *
     * @return bool
     */
    public function isSeekable();

    /**
     * Seek to a position in the stream.
     *
     * @see http://www.php.net/manual/en/function.fseek.php
     * @param int $offset Stream offset
     * @param int $whence Specifies how the cursor position will be calculated
     *     based on the seek offset. Valid values are identical to the built-in
     *     PHP $whence values for `fseek()`.  SEEK_SET: Set position equal to
     *     offset bytes SEEK_CUR: Set position to current location plus offset
     *     SEEK_END: Set position to end-of-stream plus offset.
     * @throws \RuntimeException on failure.
     */
    public function seek($offset, $whence = SEEK_SET);

    /**
     * Seek to the beginning of the stream.
     *
     * If the stream is not seekable, this method will raise an exception;
     * otherwise, it will perform a seek(0).
     *
     * @see seek()
     * @see http://www.php.net/manual/en/function.fseek.php
     * @throws \RuntimeException on failure.
     */
    public function rewind();

    /**
     * Returns whether or not the stream is writable.
     *
     * @return bool
     */
    public function isWritable();

    /**
     * Write data to the stream.
     *
     * @param string $string The string that is to be written.
     * @return int Returns the number of bytes written to the stream.
     * @throws \RuntimeException on failure.
     */
    public function write($string);

    /**
     * Returns whether or not the stream is readable.
     *
     * @return bool
     */
    public function isReadable();

    /**
     * Read data from the stream.
     *
     * @param int $length Read up to $length bytes from the object and return
     *     them. Fewer than $length bytes may be returned if underlying stream
     *     call returns fewer bytes.
     * @return string Returns the data read from the stream, or an empty string
     *     if no bytes are available.
     * @throws \RuntimeException if an error occurs.
     */
    public function read($length);

    /**
     * Returns the remaining contents in a string
     *
     * @return string
     * @throws \RuntimeException if unable to read.
     * @throws \RuntimeException if error occurs while reading.
     */
    public function getContents();

    /**
     * Get stream metadata as an associative array or retrieve a specific key.
     *
     * The keys returned are identical to the keys returned from PHP's
     * stream_get_meta_data() function.
     *
     * @see http://php.net/manual/en/function.stream-get-meta-data.php
     * @param string $key Specific metadata to retrieve.
     * @return array|mixed|null Returns an associative array if no key is
     *     provided. Returns a specific key value if a key is provided and the
     *     value is found, or null if the key is not found.
     */
    public function getMetadata($key = null);
}
~~~

### 3.5 `Psr\Http\Message\UriInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Value object representing a URI.
 *
 * This interface is meant to represent URIs according to RFC 3986 and to
 * provide methods for most common operations. Additional functionality for
 * working with URIs can be provided on top of the interface or externally.
 * Its primary use is for HTTP requests, but may also be used in other
 * contexts.
 *
 * Instances of this interface are considered immutable; all methods that
 * might change state MUST be implemented such that they retain the internal
 * state of the current instance and return an instance that contains the
 * changed state.
 *
 * Typically the Host header will also be present in the request message.
 * For server-side requests, the scheme will typically be discoverable in the
 * server parameters.
 *
 * @see http://tools.ietf.org/html/rfc3986 (the URI specification)
 */
interface UriInterface
{
    /**
     * Retrieve the scheme component of the URI.
     *
     * If no scheme is present, this method MUST return an empty string.
     *
     * The value returned MUST be normalized to lowercase, per RFC 3986
     * Section 3.1.
     *
     * The trailing ":" character is not part of the scheme and MUST NOT be
     * added.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-3.1
     * @return string The URI scheme.
     */
    public function getScheme();

    /**
     * Retrieve the authority component of the URI.
     *
     * If no authority information is present, this method MUST return an empty
     * string.
     *
     * The authority syntax of the URI is:
     *
     * <pre>
     * [user-info@]host[:port]
     * </pre>
     *
     * If the port component is not set or is the standard port for the current
     * scheme, it SHOULD NOT be included.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-3.2
     * @return string The URI authority, in "[user-info@]host[:port]" format.
     */
    public function getAuthority();

    /**
     * Retrieve the user information component of the URI.
     *
     * If no user information is present, this method MUST return an empty
     * string.
     *
     * If a user is present in the URI, this will return that value;
     * additionally, if the password is also present, it will be appended to the
     * user value, with a colon (":") separating the values.
     *
     * The trailing "@" character is not part of the user information and MUST
     * NOT be added.
     *
     * @return string The URI user information, in "username[:password]" format.
     */
    public function getUserInfo();

    /**
     * Retrieve the host component of the URI.
     *
     * If no host is present, this method MUST return an empty string.
     *
     * The value returned MUST be normalized to lowercase, per RFC 3986
     * Section 3.2.2.
     *
     * @see http://tools.ietf.org/html/rfc3986#section-3.2.2
     * @return string The URI host.
     */
    public function getHost();

    /**
     * Retrieve the port component of the URI.
     *
     * If a port is present, and it is non-standard for the current scheme,
     * this method MUST return it as an integer. If the port is the standard port
     * used with the current scheme, this method SHOULD return null.
     *
     * If no port is present, and no scheme is present, this method MUST return
     * a null value.
     *
     * If no port is present, but a scheme is present, this method MAY return
     * the standard port for that scheme, but SHOULD return null.
     *
     * @return null|int The URI port.
     */
    public function getPort();

    /**
     * Retrieve the path component of the URI.
     *
     * The path can either be empty or absolute (starting with a slash) or
     * rootless (not starting with a slash). Implementations MUST support all
     * three syntaxes.
     *
     * Normally, the empty path "" and absolute path "/" are considered equal as
     * defined in RFC 7230 Section 2.7.3. But this method MUST NOT automatically
     * do this normalization because in contexts with a trimmed base path, e.g.
     * the front controller, this difference becomes significant. It's the task
     * of the user to handle both "" and "/".
     *
     * The value returned MUST be percent-encoded, but MUST NOT double-encode
     * any characters. To determine what characters to encode, please refer to
     * RFC 3986, Sections 2 and 3.3.
     *
     * As an example, if the value should include a slash ("/") not intended as
     * delimiter between path segments, that value MUST be passed in encoded
     * form (e.g., "%2F") to the instance.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.3
     * @return string The URI path.
     */
    public function getPath();

    /**
     * Retrieve the query string of the URI.
     *
     * If no query string is present, this method MUST return an empty string.
     *
     * The leading "?" character is not part of the query and MUST NOT be
     * added.
     *
     * The value returned MUST be percent-encoded, but MUST NOT double-encode
     * any characters. To determine what characters to encode, please refer to
     * RFC 3986, Sections 2 and 3.4.
     *
     * As an example, if a value in a key/value pair of the query string should
     * include an ampersand ("&") not intended as a delimiter between values,
     * that value MUST be passed in encoded form (e.g., "%26") to the instance.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.4
     * @return string The URI query string.
     */
    public function getQuery();

    /**
     * Retrieve the fragment component of the URI.
     *
     * If no fragment is present, this method MUST return an empty string.
     *
     * The leading "#" character is not part of the fragment and MUST NOT be
     * added.
     *
     * The value returned MUST be percent-encoded, but MUST NOT double-encode
     * any characters. To determine what characters to encode, please refer to
     * RFC 3986, Sections 2 and 3.5.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.5
     * @return string The URI fragment.
     */
    public function getFragment();

    /**
     * Return an instance with the specified scheme.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified scheme.
     *
     * Implementations MUST support the schemes "http" and "https" case
     * insensitively, and MAY accommodate other schemes if required.
     *
     * An empty scheme is equivalent to removing the scheme.
     *
     * @param string $scheme The scheme to use with the new instance.
     * @return static A new instance with the specified scheme.
     * @throws \InvalidArgumentException for invalid schemes.
     * @throws \InvalidArgumentException for unsupported schemes.
     */
    public function withScheme($scheme);

    /**
     * Return an instance with the specified user information.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified user information.
     *
     * Password is optional, but the user information MUST include the
     * user; an empty string for the user is equivalent to removing user
     * information.
     *
     * @param string $user The user name to use for authority.
     * @param null|string $password The password associated with $user.
     * @return static A new instance with the specified user information.
     */
    public function withUserInfo($user, $password = null);

    /**
     * Return an instance with the specified host.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified host.
     *
     * An empty host value is equivalent to removing the host.
     *
     * @param string $host The hostname to use with the new instance.
     * @return static A new instance with the specified host.
     * @throws \InvalidArgumentException for invalid hostnames.
     */
    public function withHost($host);

    /**
     * Return an instance with the specified port.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified port.
     *
     * Implementations MUST raise an exception for ports outside the
     * established TCP and UDP port ranges.
     *
     * A null value provided for the port is equivalent to removing the port
     * information.
     *
     * @param null|int $port The port to use with the new instance; a null value
     *     removes the port information.
     * @return static A new instance with the specified port.
     * @throws \InvalidArgumentException for invalid ports.
     */
    public function withPort($port);

    /**
     * Return an instance with the specified path.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified path.
     *
     * The path can either be empty or absolute (starting with a slash) or
     * rootless (not starting with a slash). Implementations MUST support all
     * three syntaxes.
     *
     * If an HTTP path is intended to be host-relative rather than path-relative
     * then it must begin with a slash ("/"). HTTP paths not starting with a slash
     * are assumed to be relative to some base path known to the application or
     * consumer.
     *
     * Users can provide both encoded and decoded path characters.
     * Implementations ensure the correct encoding as outlined in getPath().
     *
     * @param string $path The path to use with the new instance.
     * @return static A new instance with the specified path.
     * @throws \InvalidArgumentException for invalid paths.
     */
    public function withPath($path);

    /**
     * Return an instance with the specified query string.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified query string.
     *
     * Users can provide both encoded and decoded query characters.
     * Implementations ensure the correct encoding as outlined in getQuery().
     *
     * An empty query string value is equivalent to removing the query string.
     *
     * @param string $query The query string to use with the new instance.
     * @return static A new instance with the specified query string.
     * @throws \InvalidArgumentException for invalid query strings.
     */
    public function withQuery($query);

    /**
     * Return an instance with the specified URI fragment.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified URI fragment.
     *
     * Users can provide both encoded and decoded fragment characters.
     * Implementations ensure the correct encoding as outlined in getFragment().
     *
     * An empty fragment value is equivalent to removing the fragment.
     *
     * @param string $fragment The fragment to use with the new instance.
     * @return static A new instance with the specified fragment.
     */
    public function withFragment($fragment);

    /**
     * Return the string representation as a URI reference.
     *
     * Depending on which components of the URI are present, the resulting
     * string is either a full URI or relative reference according to RFC 3986,
     * Section 4.1. The method concatenates the various components of the URI,
     * using the appropriate delimiters:
     *
     * - If a scheme is present, it MUST be suffixed by ":".
     * - If an authority is present, it MUST be prefixed by "//".
     * - The path can be concatenated without delimiters. But there are two
     *   cases where the path has to be adjusted to make the URI reference
     *   valid as PHP does not allow to throw an exception in __toString():
     *     - If the path is rootless and an authority is present, the path MUST
     *       be prefixed by "/".
     *     - If the path is starting with more than one "/" and no authority is
     *       present, the starting slashes MUST be reduced to one.
     * - If a query is present, it MUST be prefixed by "?".
     * - If a fragment is present, it MUST be prefixed by "#".
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.1
     * @return string
     */
    public function __toString();
}
~~~

### 3.6 `Psr\Http\Message\UploadedFileInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Value object representing a file uploaded through an HTTP request.
 *
 * Instances of this interface are considered immutable; all methods that
 * might change state MUST be implemented such that they retain the internal
 * state of the current instance and return an instance that contains the
 * changed state.
 */
interface UploadedFileInterface
{
    /**
     * Retrieve a stream representing the uploaded file.
     *
     * This method MUST return a StreamInterface instance, representing the
     * uploaded file. The purpose of this method is to allow utilizing native PHP
     * stream functionality to manipulate the file upload, such as
     * stream_copy_to_stream() (though the result will need to be decorated in a
     * native PHP stream wrapper to work with such functions).
     *
     * If the moveTo() method has been called previously, this method MUST raise
     * an exception.
     *
     * @return StreamInterface Stream representation of the uploaded file.
     * @throws \RuntimeException in cases when no stream is available.
     * @throws \RuntimeException in cases when no stream can be created.
     */
    public function getStream();

    /**
     * Move the uploaded file to a new location.
     *
     * Use this method as an alternative to move_uploaded_file(). This method is
     * guaranteed to work in both SAPI and non-SAPI environments.
     * Implementations must determine which environment they are in, and use the
     * appropriate method (move_uploaded_file(), rename(), or a stream
     * operation) to perform the operation.
     *
     * $targetPath may be an absolute path, or a relative path. If it is a
     * relative path, resolution should be the same as used by PHP's rename()
     * function.
     *
     * The original file or stream MUST be removed on completion.
     *
     * If this method is called more than once, any subsequent calls MUST raise
     * an exception.
     *
     * When used in an SAPI environment where $_FILES is populated, when writing
     * files via moveTo(), is_uploaded_file() and move_uploaded_file() SHOULD be
     * used to ensure permissions and upload status are verified correctly.
     *
     * If you wish to move to a stream, use getStream(), as SAPI operations
     * cannot guarantee writing to stream destinations.
     *
     * @see http://php.net/is_uploaded_file
     * @see http://php.net/move_uploaded_file
     * @param string $targetPath Path to which to move the uploaded file.
     * @throws \InvalidArgumentException if the $targetPath specified is invalid.
     * @throws \RuntimeException on any error during the move operation.
     * @throws \RuntimeException on the second or subsequent call to the method.
     */
    public function moveTo($targetPath);

    /**
     * Retrieve the file size.
     *
     * Implementations SHOULD return the value stored in the "size" key of
     * the file in the $_FILES array if available, as PHP calculates this based
     * on the actual size transmitted.
     *
     * @return int|null The file size in bytes or null if unknown.
     */
    public function getSize();

    /**
     * Retrieve the error associated with the uploaded file.
     *
     * The return value MUST be one of PHP's UPLOAD_ERR_XXX constants.
     *
     * If the file was uploaded successfully, this method MUST return
     * UPLOAD_ERR_OK.
     *
     * Implementations SHOULD return the value stored in the "error" key of
     * the file in the $_FILES array.
     *
     * @see http://php.net/manual/en/features.file-upload.errors.php
     * @return int One of PHP's UPLOAD_ERR_XXX constants.
     */
    public function getError();

    /**
     * Retrieve the filename sent by the client.
     *
     * Do not trust the value returned by this method. A client could send
     * a malicious filename with the intention to corrupt or hack your
     * application.
     *
     * Implementations SHOULD return the value stored in the "name" key of
     * the file in the $_FILES array.
     *
     * @return string|null The filename sent by the client or null if none
     *     was provided.
     */
    public function getClientFilename();

    /**
     * Retrieve the media type sent by the client.
     *
     * Do not trust the value returned by this method. A client could send
     * a malicious media type with the intention to corrupt or hack your
     * application.
     *
     * Implementations SHOULD return the value stored in the "type" key of
     * the file in the $_FILES array.
     *
     * @return string|null The media type sent by the client or null if none
     *     was provided.
     */
    public function getClientMediaType();
}
~~~

[Kezdőlap](../README.md)
