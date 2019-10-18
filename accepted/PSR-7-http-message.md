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

Minden HTTP kérésnek van egy meghatározott
[formája](https://hu.wikipedia.org/wiki/HTTP#K%C3%A9r%C3%A9s_(request)):

~~~http
POST /path HTTP/1.1\r\n
Host: example.com\r\n
\r\n
foo=bar&baz=bat\r\n
~~~

A kérés első sora („request line”) mindig „METÓDUS ERŐFORRÁS VERZIÓ” alakú. Első
helyen a [8 HTTP-metódus](https://hu.wikipedia.org/wiki/HTTP#Met%C3%B3dusok) egyike
szerepel, amely a megadott erőforráson végzendő műveletet határozza meg. Ezt követi
a kérés célja, vagyis annak az erőforrásnak az azonosítója, amelyre a kérés
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
nyelven, vagy esetleg angolul. A kéréshez hasonlóan ezt a sort követheti tetszőleges
számú HTTP fejléc sor „FEJLÉCNÉV: ÉRTÉK” alakban, majd egy üres sor után az üzenet
törzse. A kliens elsősorban az állapotkód, másodsorban a fejléc sorok tartalma
alapján kezeli a választ.

A sorokat mind a kérés, mind a válasz esetében a `SORVÉG` (CRLF, kocsi vissza + soremelés,
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

Egy HTTP üzenet vagy egy kérés a klienstől a szerver felé, vagy egy válasz a
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
megőrizni a `Host` fejléc eredeti állapotát is. Ebben az esetben a visszaadott kérés
nem fogja felülírni az üzenet `Host` fejlécet, kivéve, ha az üzenet nem tartalmaz
ilyen fejlécet.

Az alábbi táblázat azt mutatja be, hogy a `getHeaderLine('Host')` hogy fog visszatérni
a `withUri()` metódus által visszaadott értékkel, ha a `$preserveHost` paraméter
értéke `true`.

Kérés Host fejléc<sup>\[[1]](#rhh)</sup> | Kérés host összetevő<sup>\[[2]](#rhc)</sup> | URI host összetevő<sup>\[[3]](#uhc)</sup> | Eredmény
----------------------------------------|--------------------------------------------|----------------------------------------|--------
''                                      | ''                                         | ''                                     | ''
''                                      | foo.com                                    | ''                                     | foo.com
''                                      | foo.com                                    | bar.com                                | foo.com
foo.com                                 | ''                                         | bar.com                                | foo.com
foo.com                                 | bar.com                                    | baz.com                                | foo.com

- <span id="rhh">\[1]</span> A `Host` fejléc értéke a végrehajtás előtt.
- <span id="rhc">\[2]</span> Az URI végrehajtás előtt összeállított host összetevője.
- <span id="uhc">\[3]</span> Az URI `withUri()` metódussal beinjektált host összetevője.

### 1.3 Adatfolyamok (stream)

A HTTP üzenetek kezdősorból (kérés vagy állapotsor), fejlécekből és az üzenettörzsből
állnak. Ez utóbbi lehet egészen rövid, de szerfelett nagy is. Mivel az üzenettörzs
teljes egészében a memóriában lakik, ezért karakterláncként való ábrázolása könnyen
több memóriát vehet igénybe, mint szeretnénk. Ezért a kérés vagy a válasz törzsének
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
ami a kérés célját, vagyis annak az erőforrásnak az azonosítóját tartalmazza,
amelyre a kérés irányul. A kérés célját az alább felsorolt formátumokban lehet
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

A kérés célján kívül, attól elkülönülve létezik 'tényleges URL' is. Ezt nem
szokták átküldeni a HTTP üzeneten belül, mivel a kérés által használt
protokoll (http/https), port és gazdagépnév meghatározására való.

A tényleges URL-t az `UriInterface` reprezentálja, modellezve a HTTP és HTTPS URI-t
az [RFC 3986](../related-rfcs/3986.md) internetes szabványnak megfelelően. Az interfész
biztosítja a megfelelő metódusokat az URI különböző szegmenseivel történő interakciókhoz,
elkerülendő az URI ismételt beolvasását. Ezen felül előírja a `__toString()` metódus
megvalósítását is a modellezett URI megfelelő szöveges formátumú megjelenítése
érdekében.

Amikor a `getRequestTarget()` metódus segítségével lekérjük a kérés célját,
a metódus alapértelmezés szerint egy URI objektumot fog használni, hogy kinyerje
belőle a szükséges összetevőket az _eredeti-formátum_ban történő szöveges
megjelenítésére. Az _eredeti-formátum_ messze a leggyakoribb a kérés célpontok között.

Ha a végfelhasználó használni kívánja a másik három formátum valamelyikét, vagy
ha kifejezetten felül akarja írni a kérés célját, ezt a `withRequestTarget()`
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
fent felsorolt 4 kérés-célpont formátumból, továbbra is alkalmazniuk kell a
`getRequestTarget()` metódust. Ezen a klienseknek vissza KELL utasítani a nem
támogatott kérés-célpontokat és TILOS visszatérniük a `getUri()` metódusból nyert
értékekkel.

A `RequestInterface` biztosítja a kérés-célpont kinyeréséhez vagy adott célponttal
való új példány létrehozásához szükséges metódusokat. Alapértelmezés szerint, ha
nincs kifejezett kérés-célpont megadva az objektumpéldányban, akkor a `getRequestTarget()`
metódus az összeállított URI eredeti-formátum szerinti alakjával fog visszatérni
(vagy egy perjellel "/", ha nincs URI megadva). A `withRequestTarget($requestTarget)`
metódus létrehoz egy új példányt a megadott célponttal és így lehetővé teszi a
fejlesztőknek olyan kérés üzenetek létrehozását is, amelyek a másik három
kérés-célpont formátumot (abszolút-, hitelesítési-, és csillag-formátum) jelenítik
meg. Megfelelő használat esetén az összeállított URI példány továbbra is hasznos
lehet, különösen klienskódokban, amelyekben felhasználható a szerverrel való kapcsolat
felépítésére.

### 1.5 Kiszolgáló oldali kérelmek

A `RequestInterface` biztosítja egy HTTP kérés általános ábrázolását. Azonban
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

A kiszolgáló oldali kérés biztosít egy további "attribute" tulajdonságot is, amely
lehetővé teszi a hívó kódnak, hogy elemezze, lebontsa és párosítsa a kérelmet az
alkalmazás-specifikus szabályokkal (pl. útvonal, séma, gazdagép, stb. illeszkedés).
Mint ilyen, a kiszolgáló oldali kérés üzenetküldőt is biztosíthat a hívó kódok
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
kérés törzséből származnak), ezért az interfész `withUploadedFiles()` metódus
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

// Fájl streamelése Amazon S3 tárhely szolgáltatásba.
// Feltéve, hogy a $s3wrapper egy PHP adatfolyam, amit az S3-ba fogunk írni, illetve
// a Psr7StreamWrapper egy osztály, amit egy StreamInterface, mint PHP StreamWrapper
// fog elfedni.
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
 * A HTTP üzenetek a kliens szervernek küldött kéréseit és a szerver ezekre adott
 * válaszait foglalják magukba. Ez az interfész deklarálja azokat a metódusokat,
 * amelyek mindkettőhöz szükségesek.
 *
 * Az üzenetek megváltoztathatatlanok (immutable) ezért az összes olyan metódust, amely
 * megváltoztathatja az objektum állapotát, úgy KELL megvalósítani, hogy megőrizze
 * az aktuális üzenet belső állapotát és egy másik, a megváltozott állapotot
 * tartalmazó objektumpéldánnyal térjen vissza.
 *
 * @see http://www.ietf.org/rfc/rfc7230.txt
 * @see http://www.ietf.org/rfc/rfc7231.txt
 */
interface MessageInterface
{
    /**
     * Karakterláncként adja vissza a HTTP protokoll verziószámát.
     *
     * A visszaadott sztringnek kizárólag a HTTP protokoll verziószámát KELL
     * tartalmaznia (pl., "1.1", "1.0"), semmi egyebet.
     *
     * @return string a HTTP protokoll verziószáma
     */
    public function getProtocolVersion();

    /**
     * A paraméterként kapott HTTP protokoll verziószámot egy új objektumpéldányban
     * adja vissza.
     *
     * A verzió-karakterláncnak kizárólag a HTTP protokoll verziószámát KELL
     * tartalmaznia (pl., "1.1", "1.0").
     *
     * Ezt a metódust oly módon KELL implementálni, hogy az eredeti üzenet
     * változatlanul hagyása mellett olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza az új protokoll verziót.
     *
     * @param string $version a HTTP protokoll verziószáma
     * @return static
     */
    public function withProtocolVersion($version);

    /**
     * Lekérdezi az üzenet összes fejlécének értékét.
     *
     * A kulcsok megfelelnek az elküldött fejlécek neveinek az értékek pedig
     * az egyes fejlécsorok tartalmát adják vissza, szöveges tömb alakjában.
     *
     *    // Fejlécek megjelenítése szövegként
     *     foreach ($message->getHeaders() as $name => $values) {
     *       echo $name . ': ' . implode(', ', $values);
     *     }
     *
     *    // Fejlécek kiküldése iterációval:
     *     foreach ($message->getHeaders() as $name => $values) {
     *       foreach ($values as $value) {
     *         header(sprintf('%s: %s', $name, $value), false);
     *       }
     *     }
     *
     * Ameddig a fejlécek neve nem kis-, és nagybetű érzékeny, a getHeaders()
     * metódus meg fogja őrizni az eredetileg meghatározott írásmódot.
     *
     * @return string[][] az üzenet fejléceit tartalmazó asszociatív tömbbel tér
     *     vissza. A tömböt a fejlécek neveivel KELL indexelni, az
     *     az értékeknek pedig az egyes fejlécsorok tartalmát szöveges formában
     *     visszaadó tömböknek KELL lenniük.
     */
    public function getHeaders();

    /**
     * Ellenőrzi, hogy a kért fejléc létezik-e a megadott nem kis-, és nagybetű
     * érzékeny néven.
     *
     * @param string $name nem kis-, és nagybetű érzékeny fejlécmező név.
     * @return bool true-val tér vissza, ha bármely fejlécnév illeszkedik a nem
     *     kis-, és nagybetű érzékeny összehasonlítás során. False-ot ad vissza, ha
     *     nem található a keresett fejlécnév az üzenetben.
     */
    public function hasHeader($name);

    /**
     * Lekérdezi a nem kis-, és nagybetű érzékeny fejlécnévhez tartozó értéket.
     *
     * Ez a metódus egy olyan tömbbel tér vissza, amely tartalmazza a paraméterként
     * átadott fejlécnévhez tartozó össze értéket.
     *
     * Ha az adott fejléc nem található meg az üzenetben, ennek a metódusnak egy
     * üres tömböt KELL visszaadni.
     *
     * @param string $name nem kis-, és nagybetű érzékeny fejlécnév.
     * @return string[] Egy szöveges értékeket tartalmazó tömb, a megadott fejlécben
     *    megadottak szerint. Ha az adott fejléc nem található meg az üzenetben,
     *    ennek a metódusnak egy üres tömbbel KELL visszatérni.
     */
    public function getHeader($name);

    /**
     * Az adott fejléchez tartozó értékek vesszővel elválasztott felsorolása.
     *
     * Ez a metódus karakterláncként adja vissza a megadott nem kis-, és nagybetű
     * érzékeny fejlécnévhez tartozó értékek vesszővel elválasztott felsorolását.
     *
     * Megjegyzés: nem lehet minden fejléc-értéket megfelelően ábrázolni egy
     * vesszővel összefűzött szöveges listában. Az ilyen fejlécek lekérdezéséhez
     * inkább a getHeader() metódus használandó ehelyett, saját elválasztókarakter
     * megadásával.
     *
     * Ha az adott fejléc nem található meg az üzenetben, ennek a metódusnak egy
     * üres karakterláncot ('') KELL visszaadni.
     *
     * @param string $name nem kis-, és nagybetű érzékeny fejlécnév.
     * @return string Az adott fejléchez tartozó értékek vesszővel elválasztott
     *    felsorolása. Ha az adott fejléc nem található meg az üzenetben, ennek
     *    a metódusnak egy üres karakterláncot KELL visszaadni.
     */
    public function getHeaderLine($name);

    /**
     * A paraméterként kapott fejléc név-érték párt egy új objektumpéldányban
     * adja vissza.
     *
     * Ameddig a fejlécek neve nem kis-, és nagybetű érzékeny, ez a metódus a
     * getHeaders() visszatérési értékeben alkalmazott írásmódot fogja követni.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy az eredeti üzenet
     * változatlanul hagyása mellett olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza az új és/vagy módosított fejlécnevet és a hozzárendelt értéket.
     *
     * @param string $name nem kis-, és nagybetű érzékeny fejlécnév.
     * @param string|string[] $value Fejléc érték(ek).
     * @return static
     * @throws \InvalidArgumentException érvénytelen fejlécnév vagy érték esetnén.
     */
    public function withHeader($name, $value);

    /**
     * Olyan objektumpéldánnyal tér vissza, amely tartalmazza a hozzáadott fejlécet
     * és annak értékét.
     *
     * A megadott fejléc létező értéke megmarad. Az új értéke(ke)t a létező listához
     * fűzi hozzá. Ha a megadott fejléc előtte nem létezett, akkor létrehozza.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy az eredeti üzenet
     * változatlanul hagyása mellett olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza az új fejlécet és/vagy a hozzárendelt értéket.
     *
     * @param string $name a hozzáadandó fejléc nem kis-, és nagybetű érzékeny neve.
     * @param string|string[] $value Fejléc érték(ek).
     * @return static
     * @throws \InvalidArgumentException érvénytelen fejlécnév esetén.
     * @throws \InvalidArgumentException érvénytelen fejléc érték esetén.
     */
    public function withAddedHeader($name, $value);

    /**
     * Visszaad egy objektumpéldányt a megadott fejléc nélkül.
     *
     * A fejléc feloldását kis-, és nagybetű érzékenység nélkül KELL elvégezni.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy az eredeti üzenet
     * változatlanul hagyása mellett olyan objektumpéldánnyal térjen vissza, amely
     * nem tartalmazza a paraméterként kapott fejlécet.
     *
     * @param string $name az eltávolítandó fejléc nem kis-, és nagybetű érzékeny neve.
     * @return static
     */
    public function withoutHeader($name);

    /**
     * Lekérdezi az üzenet törzsét.
     *
     * @return StreamInterface az üzenet StreamInterface-be foglalt törzsével tér vissza.
     */
    public function getBody();

    /**
     * Visszaad egy objektumpéldányt a paraméterként kapott üzenettörzzsel.
     *
     * Az üzenettörzsnek StreamInterface-t megvalósító objektumnak KELL lennie.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy az eredeti üzenet
     * változatlanul hagyása mellett olyan objektumpéldánnyal térjen vissza, amely
     * adatfolyamként (StreamInterface) tartalmazza az új üzenettörzset.
     *
     * @param StreamInterface $body Üzenettörzs.
     * @return static
     * @throws \InvalidArgumentException az esetben, ha az üzenettörzs érvénytelen.
     */
    public function withBody(StreamInterface $body);
}
~~~

### 3.2 `Psr\Http\Message\RequestInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Kimenő, kliensoldali üzenetek ábrázolása.
 *
 * A HTTP specifikációnak megfelelően az interfész az alábbi komponenseket
 * tartalmazza:
 *
 * - Protokoll verzió
 * - HTTP metódus
 * - URI
 * - Fejlécek
 * - Üzenettörzs
 *
 * A létrehozás során az implementációknak meg KELL próbálni beállítani a Gazdagép
 * (Host) fejlécet a rendelkezésre álló URI-ből, ha nincs külön megadva.
 *
 * A kérelmek megváltoztathatatlanok (immutable) ezért az összes olyan metódust, amely
 * megváltoztathatja az objektum állapotát, úgy KELL megvalósítani, hogy megőrizze
 * az aktuális üzenet belső állapotát és egy másik, a megváltozott állapotot
 * tartalmazó objektumpéldánnyal térjen vissza.
 */
interface RequestInterface extends MessageInterface
{
    /**
     * Lekérdezi a kérés célpontját.
     *
     * Lekérdezi a kérés célpontját, akárhogy is jelenik meg (a klienseknek),
     * ahogy (a kiszolgálóknak) megjelent, vagy ahogy az adott objektumpéldányban
     * meg van adva (lásd: withRequestTarget()).
     *
     * A legtöbb esetben ez az összeállított URI eredeti-formátuma lesz, hacsak
     * az érték nem áll rendelkezésre a konkrét implementáció számára (lásd:
     * withRequestTarget() metódus, alább).
     *
     * Ha nincs elérhető URI, sem kérés-célpont megadva, akkor ennek a metódusnak
     * egy perjellel ("/") kell visszatérnie.
     *
     * @return string
     */
    public function getRequestTarget();

    /**
     * A megadott kérés-célponttal rendelkező objektum-példánnyal tér vissza.
     *
     * Ha a kérésnek szüksége van egy nem eredeti-formátumú kérés-célpontra
     *  — pl. abszolút-formátum, hitelesítési-, vagy csillag-formátum meghatározásához —
     * ez a metódus használható egy olyan példány létrehozásához, amely rendelkezik
     * a kért kérés-célponttal.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza az új kérés-célpontot.
     *
     * @see http://tools.ietf.org/html/rfc7230#section-5.3 (a kérés üzenetekben
     *     engedélyezett kérés-célpont formátumokról)
     * @param mixed $requestTarget
     * @return static
     */
    public function withRequestTarget($requestTarget);

    /**
     * Lekérdezi a kérés HTTP metódusát.
     *
     * @return string Szöveges formában adja vissza a kérés HTTP metódusát.
     */
    public function getMethod();

    /**
     * Visszaad egy új példányt a megadott HTTP metódussal.
     *
     * Noha a HTTP metódusok neve jellemzően csupa nagybetűkből áll, a nevek ennek
     * ellenére kis-, és nagybetű érzékenyek és ezért az implementációknak
     * NEM KELLENE módosítani a paraméterként átvett szöveges érték írásmódján.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely tartalmazza
     * az új HTTP metódust.
     *
     * @param string $method kis-, és nagybetű érzékeny metódusnév.
     * @return static
     * @throws \InvalidArgumentException érvénytelen HTTP metódus esetén.
     */
    public function withMethod($method);

    /**
     * Lekéri az URI objektumpéldányt.
     *
     * E metódusnak egy UriInterface példánnyal KELL visszatérnie.
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @return UriInterface A kéréshez tartozó URI-t reprezentáló UriInterface
     *      példánnyal tér vissza.
     */
    public function getUri();

    /**
     * Visszaad egy új objektumpéldányt a megadott URI-vel.
     *
     * Ennek a metódusnak alapértelmezetten frissítenie KELL a gazdagép fejlécet
     * a visszaadott kérésben, ha az URI tartalmaz gazdagépnév komponenst. Ha
     * nem tartalmaz ilyet, akkor minden létező gazdagép fejlécet át KELL vinni
     * a visszaadott kéréspéldányba.
     *
     * A `$preserveHost` paraméter `true`-ra állításával engedélyezhető a gazdagép
     * fejléc eredeti állapotának megőrzése is. Ebben az esetben a jelen metódus
     * az alábbi módokon léphet kölcsönhatásba a gazdagép fejléccel:
     *
     * - Ha a gazdagép fejléc hiányzik vagy üres és az új URI tartalmaz gazdagépnév
     *   komponenst, ennek a metódusnak frissítenie KELL a gazdagép fejlécet a
     *   visszaadott kérésben.
     * - Ha a gazdagép fejléc hiányzik vagy üres és az új URI nem tartalmaz gazdagépnév
     *   komponenst, ennek a metódusnak TILOS frissítenie a gazdagép fejlécet a
     *   visszaadott kérésben.
     * - Ha a gazdagép fejléc létezik és nem üres, ennek a metódusnak TILOS frissítenie
     *   a gazdagép fejlécet a visszaadott kérésben.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely tartalmazza
     * az új UriInterface példányt.
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @param UriInterface $uri Az új kéréshez tartozó URI.
     * @param bool $preserveHost Az eredeti gazdagép fejlécsor állapotának megőrzése.
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
 * Beérkező, kiszolgáló-oldali HTTP kérés ábrázolása.
 *
 * A HTTP specifikációnak megfelelően az interfész az alábbi komponenseket
 * tartalmazza:
 *
 * - Protokoll verzió
 * - HTTP metódus
 * - URI
 * - Fejlécek
 * - Üzenettörzs
 *
 * Emellett becsomagolja az alkalmazáshoz a CGI és/vagy PHP környezettől beérkező
 * összes adatot, ideértve különösen:
 *
 * - A $_SERVER szuper globális által tárolt értékeket
 * - A sütik által (általában a $_COOKIE tömbben) biztosított értékeket
 * - A lekérdezési komponensben átadott argumentumok (általában a $_GET tömbbe
 *   kerülnek, vagy a parse_str() függvény szolgáltatja őket)
 * - Feltöltött állományok, ha vannak (a $_FILES tömb által megjelenítve)
 * - Deszerializált üzenettörzs paraméterek (általában a $_POST tömbből)
 *
 * A $_SERVER értékeit megváltoztathatatlanként KELL kezelni, mivel ezek jelenítik
 * meg az alkalmazás állapotát a kérés beérkezésének időpontjában; ezért egy
 * metódusnak sem szabad lehetővé tenni, hogy módosítsa ezeket az értékeket.
 * A többi érték olyan metódusokat igényel, amelyek helyre tudják állítani őket
 * a $_SERVER tömbből vagy a kérés üzenettörzséből és biztosítják az alkalmazás
 * futása során szükségessé váló eljárásokat (pl. az üzenettörzs paramétereit a
 * tartalomtípus alapján is lehetséges deszerializálni).
 *
 * Ezen felül a jelen interfész felismeri a kérés önellenőrzésének hasznosságát
 * a további paraméterek leszármaztatásában és összeillesztésében (pl. útvonal
 * összehasonlítás URI segítségével, süti értékek visszafejtése, nem űrlap kódolt
 * üzenettörzs deszerializálása, a felhasználók azonosítási fejléceinek ellenőrzése.
 * Mindezek a paraméterek "attributes" néven vannak elraktározva az objektumban.
 *
 * A kérés üzenetek megváltoztathatatlanok (immutable) ezért az összes olyan
 * metódust, amely megváltoztathatja az objektum állapotát, úgy KELL megvalósítani,
 * hogy megőrizze az aktuális üzenet belső állapotát és egy másik, a megváltozott
 * állapotot tartalmazó objektumpéldánnyal térjen vissza.
 */
interface ServerRequestInterface extends RequestInterface
{
    /**
     * A kiszolgáló paramétereinek lekérdezése.
     *
     * A bejövő kérés környezeti adatainak lekérdezése, jellemzően a PHP
     * $_SERVER szuperglobális tömbjéből kinyerve. Az adatokat máshonnan is
     * be LEHET szerezni, nem szükséges feltétlenül a $_SERVER tömbből származniuk.
     *
     * @return array
     */
    public function getServerParams();

    /**
     * Sütiadatok lekérdezése.
     *
     * Lekérdezi azokat a sütiket, amelyeket a kliens küldött a kiszolgálónak.
     *
     * A visszatérési értéknek kompatibilisnek KELL lennie a $_COOKIE
     * szuperglobális tömbbel.
     *
     * @return array
     */
    public function getCookieParams();

    /**
     * Visszaad egy példányt a megadott sütikkel.
     *
     * Az adatokat máshonnan is be LEHET szerezni, nem szükséges feltétlenül a
     * $_COOKIE tömbből származniuk, de kompatibilisnek KELL lenniük vele. Ezeket
     * az adatokat jellemzően a példányosításnál fecskendezik be.
     *
     * Ennek a metódusnak TILOS frissítenie a vonatkozó kéréspéldány Cookie
     * fejlécét, sem a kapcsolódó szerver paramétereket.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza a frissített süti paramétereket.
     *
     * @param array $cookies Sütiket reprezentáló kulcs/érték párokból álló tömb.
     * @return static
     */
    public function withCookieParams(array $cookies);

    /**
     * Lekéri a lekérdezési karakterláncból kinyert változókat.
     *
     * Lekéri a deszerializált lekérdezési karakterlánc változóit, ha vannak.
     *
     * Megjegyzés: előfordulhat, hogy a query paraméterek nincsenek szinkronban
     * az URI vagy a szerver paramétereivel. Ha csupán az eredeti értékek kinyerését
     * kell biztosítani, akkor szükség lehet a lekérdezési karakterlánc elemzésére
     * a `getUri()->getQuery()` visszatérési értékéből, vagy a `QUERY_STRING`
     * környezeti változóból.
     *
     * @return array
     */
    public function getQueryParams();

    /**
     * Visszaad egy új objektumpéldányt a megadott lekérdezési karakterlánc
     * (query string) változókkal.
     *
     * Ezen értékeket AJÁNLOTT változatlanul hagyni a bejövő kérés folyamán.
     * Ezeket be LEHET injektálni példányosítás közben, például a PHP $_GET
     * szuperglobális tömbjéből, vagy származtatni LEHET más értékekből, úgymint
     * az URI. Olyan esetekben, amikor a változók az URI-ből lettek kinyerve, az
     * adatnak kompatibilisnek KELL lennie azzal, amivel a PHP parse_str() függvénye
     * visszatérne, a duplikált lekérdezési paraméterek és beágyazott adatkollekciók
     * kezelése érdekében.
     *
     * A lekérdezési karakterlánc változóinak beállításakor TILOS megváltoztatni
     * a kérés által tárolt URI-t, sem a kiszolgáló paramétereinek értékét.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza a frissített lekérdezési paramétereket (query string).
     *
     * @param array $query A lekérdezési karakterláncot reprezentáló, jellemzően
     *           a $_GET szuperglobálisból nyert kulcs/érték párokból álló tömb.
     * @return static
     */
    public function withQueryParams(array $query);

    /**
     * Normalizált formában lekéri a feltöltött fájlok adatait.
     *
     * Ez a metódus a feltöltött fájlok metaadataival tér vissza, amelyeket egy
     * normalizált fastruktúrában helyez el, ahol minden levél a
     * Psr\Http\Message\UploadedFileInterface egy példánya.
     *
     * Ezeket az értékeket ki LEHET nyerni a $_FILES tömbből vagy az üzenettörzsből
     * példányosításkor vagy be LEHET őket fecskendezni a withUploadedFiles()
     * metóduson keresztül.
     *
     * @return array Egy tömb, mely fastruktúrában tárolja a UploadedFileInterface
     *             példányokat. Üres tömböt KELL visszaadni, ha nincsenek adatok.
     */
    public function getUploadedFiles();

    /**
     * Új példány létrehozása a megadott feltöltött állományokkal.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza a frissített üzenettörzs paramétereket.
     *
     * @param array $uploadedFiles Egy tömb, mely fastruktúrában tárolja a
     *             UploadedFileInterface példányokat.
     * @return static
     * @throws \InvalidArgumentException ha a bemeneti tömb struktúrája érvénytelen.
     */
    public function withUploadedFiles(array $uploadedFiles);

    /**
     * Lekérdezi az üzenettörzsben tárolt paramétereket.
     *
     * Ha a kérés Content-Type fejlécében a típus application/x-www-form-urlencoded
     * vagy multipart/form-data és a HTTP kérés metódusa POST, akkor ennek a
     * metódusnak a $_POST szuperglobális tömb tartalmát KELL visszaadni.
     *
     * Egyébként ez a metódus a kérés üzenettörzsének deszerializált tartalmával
     * térhet vissza; mivel a lekérdezés strukturált tartalmat eredményezhet, a
     * visszatérési érték lehetséges típusának tömbnek vagy objektumnak KELL lennie.
     * A null visszatérési érték a tartalom hiányát jelzi az üzenettörzsben.
     *
     * @return null|array|object A deszerializált üzenettörzs paraméterek, ha vannak.
     *              Ezek jellemzően egy tömbben vagy egy objektumban találhatók.
     */
    public function getParsedBody();

    /**
     * Új példány létrehozása a megadott üzenettörzs paraméterekkel.
     *
     * Ezeket a példányosításkor LEHET beinjektálni.
     *
     * Ha a kérés Content-Type fejlécében a típus application/x-www-form-urlencoded
     * vagy multipart/form-data és a HTTP kérés metódusa POST, akkor ennek a
     * metódusnak a $_POST szuperglobális tömb tartalmát KELL visszaadni.
     *
     * Az adatot ki LEHET nyerni máshonnan is, mint a $_POST tömbből, viszont ez
     * az üzenettörzs deszerializálásának eredménye kell, hogy legyen.
     * A deszerializáció/elemzés strukturált adattal tér vissza, ezért ez a metódus
     * kizárólag tömböket és objektumokat fogad el, esetleg null értéket, ha
     * nincs mit elemezni.
     *
     * Mint például ha a tartalom egyeztetés meghatározza, hogy a kérés adat
     * egy JSON legyen, ez a metódus felhasználható egy új kérés példány
     * létrehozására a deszerializált paraméterekkel.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti
     * üzenet immutabilitását és olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza a frissített üzenettörzs paramétereket.
     *
     * @param null|array|object $data A deszerializált üzenettörzs paraméterek,
     *    ha vannak. Ezek jellemzően egy tömbben vagy egy objektumban találhatók.
     * @return static
     * @throws \InvalidArgumentException nem támogatott argumentumtípus esetén
     */
    public function withParsedBody($data);

    /**
     * Lekérdezi a kérés származtatott tulajdonságait.
     *
     * A kérés "attributes" név alatt tárolt tulajdonságai lehetővé teszik
     * bármilyen a kérésből származtatott paraméter beinjektálását: például,
     * az elérési útvonal összehasonlítás művelet eredményét; a sütik dekódolásának
     * eredményét; a nem Űrlap-kódolt üzenettörzs deszerializálásának eredményét,
     * stb. A tulajdonságok alkalmazás-, és kérésfüggők lesznek, ezen felül
     * LEHETNEK megváltoztathatók is.
     *
     * @return mixed[] A kérésből származtatott attribútumok.
     */
    public function getAttributes();

    /**
     * Lekérdezi a kérés egy megadott származtatott tulajdonságát.
     *
     * Visszaadja a kérés egy adott származtatott tulajdonságát a getAttributes()
     * metódusnál leírt módon. Ha a tulajdonság előzőleg nem volt beállítva, akkor
     * egy alapértelmezett értékkel tér vissza, ha be van állítva ilyen.
     *
     * Ez a metódus fölöslegessé teszi a hasAttribute() metódust azzal, hogy
     * lehetővé teszi az alapértelmezett érték beállítását arra az esetre, ha a
     * kért tulajdonság nem létezik.
     *
     * @see getAttributes()
     * @param string $name A tulajdonság neve.
     * @param mixed $default Alapértelmezett érték, ha a tulajdonság nem létezik.
     * @return mixed
     */
    public function getAttribute($name, $default = null);

    /**
     * Egy olyan új objektumpéldánnyal tér vissza, amely tartalmazza a kérés
     * paraméterként megadott származtatott tulajdonságát.
     *
     * Ez a metódus lehetővé teszi a kérés egyes származtatott tulajdonságának
     * beállítását a getAttributes() metódusnál leírt módon.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza a tulajdonság frissített értékét.
     *
     * @see getAttributes()
     * @param string $name A tulajdonság neve.
     * @param mixed $value A tulajdonság értéke.
     * @return static
     */
    public function withAttribute($name, $value);

    /**
     * Egy olyan új objektumpéldánnyal tér vissza, amely már nem tartalmazza a
     * kérés paraméterként megadott származtatott tulajdonságát.
     *
     * Ez a metódus lehetővé teszi a kérés adott származtatott tulajdonságának
     * eltávolítását a getAttributes() metódusnál leírt módon.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely nem
     * tartalmazza az eltávolított attribútumot.
     *
     * @see getAttributes()
     * @param string $name A tulajdonság neve.
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
 * Kimenő, szerver-oldali üzenet ábrázolása
 *
 * A HTTP specifikációnak megfelelően az interfész az alábbi komponenseket
 * tartalmazza:
 *
 * - Protokoll verzió
 * - Állapotkód és indoklás
 * - Fejlécek
 * - Üzenettörzs
 *
 * A válasz üzenetek megváltoztathatatlanok (immutable) ezért az összes olyan
 * metódust, amely megváltoztathatja az objektum állapotát, úgy KELL megvalósítani,
 * hogy megőrizze az aktuális üzenet belső állapotát és egy másik, a megváltozott
 * állapotot tartalmazó objektumpéldánnyal térjen vissza.
 */
interface ResponseInterface extends MessageInterface
{
    /**
     * Lekéri a válaszüzenet állapotkódját.
     *
     * Az állapotkód egy háromjegyű egész szám, amely jelzi a kiszolgáló kísérletét,
     * hogy megértse és teljesítse a kérelmet.
     *
     * @return int Állapotkód.
     */
    public function getStatusCode();

    /**
     * Visszaad egy új objektumpéldányt a megadott állapotkóddal és opcionálisan
     * megadható szöveges indoklással.
     *
     * Ha nincs indoklás beállítva, akkor az implementációknak LEHET választani
     * az RFC 7231 vagy az IANA által a válaszokhoz ajánlott alapértelmezett
     * állapotkódok közül.
     *
     * Ezt a metódust oly módon KELL implementálni, hogy megőrizze az eredeti üzenet
     * immutabilitását és olyan objektumpéldánnyal térjen vissza, amely
     * tartalmazza a frissített állapotkódot és indoklást.
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @param int $code A beállítandó 3 jegyű állapotkód.
     * @param string $reasonPhrase Az indoklás együtt használható a megadott
     *     állapotkóddal; ha ilyen nincs megadva, akkor az implementációknak LEHET
     *     a HTTP specifikáció által ajánlott, az adott állapotkódhoz tartozó
     *     alapértelmezett indoklást is alkalmazni.
     * @return static
     * @throws \InvalidArgumentException Érvénytelen állapotkód argumentum esetén.
     */
    public function withStatus($code, $reasonPhrase = '');

    /**
     * A kérés állapotkódjához társított indoklás lekérdezése.
     *
     * Mivel az indoklás nem kötelező eleme a válasz állapotsorának, ezért az
     * értéke LEHET akár üres is (''). Az implementációknak LEHET azt is választani,
     * hogy az alapértelmezett RFC 7231 által ajánlott indoklást (melyek felsorolása
     * megtalálható az IANA HTTP állapotkód regiszterében, a mellékelt linken)
     * adják vissza a válasz állapotkódjához.
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @return string Indoklás; üres karakterláncot kell visszaadni, ha nincs ilyen.
     */
    public function getReasonPhrase();
}
~~~

### 3.4 `Psr\Http\Message\StreamInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Leír egy adatfolyamot.
 *
 * Jellemzően egy objektumpéldányba csomagol egy PHP adatfolyamot; ez az interfész
 * biztosítja a wrapper-t a leggyakoribb műveletek körül, ideértve a bejövő
 * adatfolyamok karakterlánccá történő szerializációját is.
 */
interface StreamInterface
{
    /**
     * Beolvassa az összes adatot az adatfolyamból egy karakterláncba, elejétől
     * a végéig.
     *
     * E metódusnak meg KELL kísérelnie megkeresni az adatfolyam kezdetét, az adatok
     * beolvasása előtt és addig olvasni az adatfolyamot, amíg a végére nem ér.
     *
     * Figyelmeztetés: ez nagyobb mennyiségű adatot tölthet a memóriába.
     *
     * Ennek a metódusnak TILOS kivételt dobni azért, hogy megerősítse a PHP
     * típuskonverziót (casting).
     *
     * @see http://php.net/manual/en/language.oop5.magic.php#object.tostring
     * @return string
     */
    public function __toString();

    /**
     * Letárja az adatfolyamot és minden mögöttes erőforrást.
     *
     * @return void
     */
    public function close();

    /**
     * Elkülönít minden mögöttes erőforrást az adatfolyamból.
     *
     * Miután az adatfolyamot elkülönítették, az használhatatlan állapotba kerül.
     *
     * @return resource|null Mögöttes PHP adatfolyam, ha van ilyen
     */
    public function detach();

    /**
     * Lekérdezi az adatfolyam méretét, ha az ismert.
     *
     * @return int|null Az adatfolyam byte-okban megadott méretével tér vissza,
     *                  ha az ismert, ha nem, akkor null-t kell vissza adnia.
     */
    public function getSize();

    /**
     * Visszaadja az írási/olvasási fájl mutató aktuális pozícióját
     *
     * @return int A fájl mutató (pointer) pozíciója
     * @throws \RuntimeException ha hiba történik.
     */
    public function tell();

    /**
     * True értékkel tér vissza, ha az adatfolyam a végéhez ért.
     *
     * @return bool
     */
    public function eof();

    /**
     * Ellenőrzi, hogy az adatfolyam kereshető-e vagy sem.
     *
     * @return bool
     */
    public function isSeekable();

    /**
     * Beállítja a fájlmutatót egy adott pozícióra az adatfolyamon belül.
     *
     * @see http://www.php.net/manual/en/function.fseek.php
     * @param int $offset Adatfolyam eltolás
     * @param int $whence Meghatározza, hogy a kurzor pozíciója hogy legyen
     *     kiszámítva az eltolás (offset) alapján. Az érvényes értékek megegyeznek
     *     a PHP beépített `fseek()`függvényének $whence ($honnan) argumentumának
     *     értékeivel.
     *     SEEK_SET: Állítsa a pozíciót az $offset-ben megadott bájtra
     *     SEEK_CUR: Állítsa a pozíciót a fájlmutató aktuális helye + $offset értékre
     *     SEEK_END: Állítsa a pozíciót az adatfolyam vége + $offset értékre.
     * @throws \RuntimeException kudarc esetén.
     */
    public function seek($offset, $whence = SEEK_SET);

    /**
     * Megkeresi az adatfolyam kezdetét.
     *
     * Ha az adatfolyam nem kereshető, ez a metódus kivételt fog dobni; egyébként
     * visszaállítja a fájlmutatót az adatfolyam elejére (seek(0)).
     *
     * @see seek()
     * @see http://www.php.net/manual/en/function.fseek.php
     * @throws \RuntimeException kudarc esetén.
     */
    public function rewind();

    /**
     * Ellenőrzi, hogy az adatfolyam írható-e vagy sem.
     *
     * @return bool
     */
    public function isWritable();

    /**
     * Szöveges adatot ír az adatfolyamba.
     *
     * @param string $string A karakterlánc, amit írni kell.
     * @return int Az adatfolymba írt bájtok számával tér vissza.
     * @throws \RuntimeException kudarc esetén.
     */
    public function write($string);

    /**
     * Ellenőrzi, hogy az adatfolyam olvasható-e vagy sem.
     *
     * @return bool
     */
    public function isReadable();

    /**
     * Megadott hosszúságú adatot olvas ki az adatfolyamból.
     *
     * @param int $length Kiolvas legfeljebb $length byte adatot az objektumból
     *     és visszaadja azt. Kevesebb, mint $length byte-ot is visszaadhat, ha
     *     a meghívott mögöttes adatfolyam kevesebbet ad vissza.
     * @return string Visszaadja az adatfolyamból kiolvasott adatot, vagy egy
     *     üres karakterláncot, ha nincs elérhető adat.
     * @throws \RuntimeException ha hiba történik.
     */
    public function read($length);

    /**
     * Karakterláncként visszaadja az adatfolyam fennmaradó tartalmát.
     *
     * @return string
     * @throws \RuntimeException ha nem tudja olvasni.
     * @throws \RuntimeException ha hiba történt olvasás közben.
     */
    public function getContents();

    /**
     * Asszociatív tömbként vagy megadott kulcs alapján visszaadja az adatfolyam
     * metaadatait.
     *
     * A visszaadott tömb kulcsai megegyeznek a PHP stream_get_meta_data() függvény
     * által visszaadott kulcsokkal.
     *
     * @see http://php.net/manual/en/function.stream-get-meta-data.php
     * @param string $key Meghatározott metaadat lekérése.
     * @return array|mixed|null Egy asszociatív tömböt ad vissza, ha nincs kulcs
     *     megadva. A megadott kulcshoz tartozó metaadattal tér vissza, ha a kulcs
     *     létezik és van értéke, vagy nullával, ha a kulcs nem található.
     */
    public function getMetadata($key = null);
}
~~~

### 3.5 `Psr\Http\Message\UriInterface`

~~~php
<?php
namespace Psr\Http\Message;

/**
 * Az URI-t reprezentáló értékobjektum.
 *
 * Ez az interfész arra van szánva, hogy az RFC 3986 dokumentumban foglalt STD 66
 * internetes szabvány alapján biztosítsa a leggyakoribb műveletekhez szükséges
 * metódusokat. Az URI-kkel való munkához szükséges további funkcionalitás
 * biztosítható az interfészben leírtakon felül is. Az interfész elsődlegesen a
 * HTTP kérelmekhez használható, de más kontextusban is hasznos lehet.
 *
 * Az ezen interfészt megvalósító osztályok objektumpéldányai megváltoztathatatlanok;
 * minden olyan metódust, amelyik megváltoztathatja az objektum állapotát,
 * úgy KELL megvalósítani, hogy megőrizze az aktuális üzenet belső állapotát és
 * egy másik, a megváltozott állapotot tartalmazó objektumpéldánnyal térjen vissza.
 *
 * Jellemzően a gazdagép (Host) fejléc lesz szintén jelen a kérés üzenetben. A
 * kiszolgáló oldali kérelmekben a séma komponens általában kinyerhető a szerver
 * paramétereiből.
 *
 * @see http://tools.ietf.org/html/rfc3986 (URI specifikáció)
 * @see https://github.com/dominicus75/fig-standards/blob/master/related-rfcs/3986.md
 * (URI specifikáció magyarul)
 */
interface UriInterface
{
    /**
     * Lekéri az URI séma (scheme) komponensét.
     *
     * Ha a séma komponens nincs jelen, ennek a metódusnak üres karakterlánccal
     * KELL visszatérnie.
     *
     * A visszaadott értéket mindig kisbetűs írásmódúra KELL alakítani, az RFC
     * 3986 3.1 fejezetének megfelelően.
     *
     * A sémát lezáró kettőspont (":") határoló karakter, nem része a sémának,
     * ezért TILOS hozzáadni.
     *
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#31-s%C3%A9ma-scheme1
     * @return string Az URI séma komponense.
     */
    public function getScheme();

    /**
     * Lekéri az URI hitelesítési (authority) komponensét.
     *
     * Ha a hitelesítési komponens nincs jelen, ennek a metódusnak üres
     * karakterlánccal KELL visszatérnie.
     *
     * Az URI hitelesítési komponensének szintaxisa:
     *
     * <pre>
     * [user-info@]host[:port]
     * </pre>
     *
     * Ha a port komponens nincs beállítva, vagy a jelenlegi séma szabványos
     * portja az, akkor ezt NEM KELLENE belevenni a kimenetbe.
     *
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#32-hiteles%C3%ADt%C3%A9s-authority
     * @return string Az URI hitelesítési komponense,
     *                "[user-info@]host[:port]" formátumban.
     */
    public function getAuthority();

    /**
     * Lekéri az URI felhasználói információ (userinfo) komponensét.
     *
     * Ha a felhasználói információ komponens nincs jelen, ennek a metódusnak üres
     * karakterlánccal KELL visszatérnie.
     *
     * Ha az URI tartalmaz felhasználói komponenst, ez a metódus visszaadja annak
     * értékét; ezen felül ha a jelszó is meg van adva, az hozzá lesz fűzve a
     * felhasználónévhez, egy kettőspont (":") karakterrel elválasztva tőle.
     *
     * A felhasználói információt lezáró kukac ("@") határoló karakter, nem része
     * a felhasználói információnak, ezért TILOS hozzáadni.
     *
     * @return string Az URI felhasználói információ komponense,
     *                "username[:password]" formátumban.
     */
    public function getUserInfo();

    /**
     * Lekéri az URI gazdagép (Host) komponensét.
     *
     * Ha a gazdagép komponens nincs jelen, ennek a metódusnak üres
     * karakterlánccal KELL visszatérnie.
     *
     * A visszaadott értéket mindig kisbetűs írásmódúra KELL alakítani, az RFC
     * 3986 3.2.2 fejezetének megfelelően.
     *
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#322-gazdag%C3%A9p-host
     * @return string Az URI gazdagép komponense.
     */
    public function getHost();

    /**
     * Lekéri az URI port komponensét.
     *
     * Ha a port jelen van és nem az aktuális HTTP-sémához rendelt szabványos port,
     * akkor ennek a metódusnak integer-ként KELL visszaadnia. Ha a port az
     * aktuális séma szabványos portja, akkor ennek a metódusnak null értékkel
     * KELL visszatérnie.
     *
     * Ugyanígy null értéket KELL visszaadni, ha a port vagy a séma nincs jelen
     * az objektumban.
     *
     * Ha a port nincs jelen, de a séma igen, akkor ennek a metódusnak a sémához
     * tartozó szabványos (alapértelmezett) portot is vissza LEHET adni, vagy
     * esetleg null értéket.
     *
     * @return null|int Az URI port komponense.
     */
    public function getPort();

    /**
     * Lekéri az URI útvonal (path) komponensét.
     *
     * Az útvonal komponens lehet akár üres vagy abszolút (perjellel kezdve) esetleg
     * relatív (perjel nélkül az elején). Az implementációknak támogatniuk KELL
     * mindhárom szintaxist.
     *
     * Általában az üres ("") és az abszolút ("/") útvonal egyenlőnek tekinthető
     * az RFC 7230 ajánlás 2.7.3. fejezete alapján. De ennek a metódusnak TILOS
     * automatikusan megejteni ezt a normalizálást, mert a megtisztított bázis
     * útvonal (base path) kontextusában, pl. a front controller esetében ez a
     * különbség jelentőssé válhat. Ezért a "" és a "/" kezelését a felhasználóra
     * kell hagyni.
     *
     * A visszaadott értéket url-kódolni KELL, de TILOS duplán kódolni bármely
     * karaktert. Azt, hogy mely karaktereket kell url-kódolni, az RFC 3986
     * 2, és 3.3. fejezete határozza meg.
     *
     * Például ha az értéknek perjelet ("/") is tartalmaznia kell, amit nem az
     * útvonal komponens szegmensei közötti határolókarakternek szántak, akkor
     * azt az értéket url-kódolni kell (pl. "%2F") az objektumpéldányban.
     *
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#2-karakterk%C3%A9szlet
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#33-%C3%BAtvonal-path
     * @return string Az URI útvonal komponense.
     */
    public function getPath();

    /**
     * Lekéri az URI lekérdezési (query string) komponensét.
     *
     * Ha a lekérdezési komponens nincs jelen, ennek a metódusnak üres
     * karakterlánccal KELL visszatérnie.
     *
     * A lekérdezési komponenst nyitó kérdőjel ("?") határoló karakter, nem része
     * a komponensnek, ezért TILOS hozzáadni.
     *
     * A visszaadott értéket url-kódolni KELL, de TILOS duplán kódolni bármely
     * karaktert. Azt, hogy mely karaktereket kell url-kódolni, az RFC 3986
     * 2, és 3.4. fejezete határozza meg.
     *
     * Például, ha az érték a lekérdezési komponens egy kulcs/érték párja, ami
     * és jelet ("&") is tartalmaz, amelyet nem az értékek közti határolókarakternek
     * szántak, akkor azt az értéket url-kódolni kell (pl. "%26") az objektumpéldányban.
     *
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#2-karakterk%C3%A9szlet
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#34-lek%C3%A9rdez%C3%A9s-query
     * @return string Az URI lekérdezési komponense.
     */
    public function getQuery();

    /**
     * Lekéri az URI részerőforrás azonosító (fragment) komponensét.
     *
     * Ha a részerőforrás azonosító nincs jelen, ennek a metódusnak üres
     * karakterlánccal KELL visszatérnie.
     *
     * A részerőforrás azonosítót nyitó kettős kereszt ("#") határoló karakter,
     * nem része a komponensnek, ezért TILOS hozzáadni.
     *
     * A visszaadott értéket url-kódolni KELL, de TILOS duplán kódolni bármely
     * karaktert. Azt, hogy mely karaktereket kell url-kódolni, az RFC 3986
     * 2, és 3.5. fejezete határozza meg.
     *
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#2-karakterk%C3%A9szlet
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#35-r%C3%A9szer%C5%91forr%C3%A1s-fragment
     * @return string Az URI részerőforrás azonosító komponense.
     */
    public function getFragment();

    /**
     * Meghatározott séma komponenst tartalmazó új objektumpéldánnyal tér vissza.
     *
     * Ennek a metódusnak meg KELL őriznie az eredeti példány állapotát és
     * olyan objektumpéldánnyal visszatérni, amely tartalmazza a paraméterként
     * átadott sémát.
     *
     * Az implementációknak tekintet nélkül a kis-, és nagybetűs írásmódra,
     * támogatniuk KELL a "http" és a "https" sémákat, de LEHET alkalmazni más
     * sémákat is, ha szükséges.
     *
     * Ha ezt a metódust üresen hagyott séma paraméterrel hívják meg, az a séma
     * komponens eltávolításával egyenértékű.
     *
     * @param string $scheme Az új példányhoz rendelt séma komponens.
     * @return static Az új példány a megadott séma komponenssel.
     * @throws \InvalidArgumentException érvénytelen séma esetén.
     * @throws \InvalidArgumentException nem támogatott séma esetén.
     */
    public function withScheme($scheme);

    /**
     * Meghatározott felhasználói információ komponenst tartalmazó új
     * objektumpéldánnyal tér vissza.
     *
     * Ennek a metódusnak meg KELL őriznie az eredeti példány állapotát és
     * olyan objektumpéldánnyal visszatérni, amely tartalmazza a paraméterként
     * átadott felhasználói információkat.
     *
     * A jelszó elhagyható, viszont a felhasználó információ komponensnek
     * tartalmaznia KELL a felhasználónevet; ha az $user argumentumban üres
     * karakterlánc ("") kerül megadásra, az egyenértékű a felhasználói információ
     * törlésével.
     *
     * @param string $user A hitelesítéshez rendelt felhasználónév.
     * @param null|string $password A $user felhasználóhoz társított jelszó.
     * @return static Az új példány a megadott felhasználói információkkal.
     */
    public function withUserInfo($user, $password = null);

    /**
     * Meghatározott gazdagép komponenst tartalmazó új objektumpéldánnyal
     * tér vissza.
     *
     * Ennek a metódusnak meg KELL őriznie az eredeti példány állapotát és
     * olyan objektumpéldánnyal visszatérni, amely tartalmazza a paraméterként
     * átadott gazdagépnevet.
     *
     * Ha ezt a metódust üresen hagyott gazdagép paraméterrel hívják meg, az a
     * gazdagép komponens eltávolításával egyenértékű.
     *
     * @param string $host Az új példányhoz rendelt gazdagép komponens.
     * @return static Az új példány a megadott gazdagép komponenssel.
     * @throws \InvalidArgumentException érvénytelen gazdagépnév esetén.
     */
    public function withHost($host);

    /**
     * Meghatározott port komponenst tartalmazó új objektumpéldánnyal tér vissza.
     *
     * Ennek a metódusnak meg KELL őriznie az eredeti példány állapotát és
     * olyan objektumpéldánnyal visszatérni, amely tartalmazza a paraméterként
     * átadott portszámot.
     *
     * Az implementációknak kivételt KELL dobniuk, ha a paraméterként átadott port
     * kívül esik a TCP és UDP számára fenntartott port tartományon.
     *
     * Null érték megadása a $port argumentumban egyenértékű a port információ
     * eltávolításával.
     *
     * @param null|int $port Az új példányhoz rendelt port komponens; null
     *                       érték a port információ eltávolításához.
     * @return static Az új példány a megadott port komponenssel.
     * @throws \InvalidArgumentException érvénytelen port argumentum esetén.
     */
    public function withPort($port);

    /**
     * Meghatározott útvonal komponenst tartalmazó új objektumpéldánnyal
     * tér vissza.
     *
     * Ennek a metódusnak meg KELL őriznie az eredeti példány állapotát és
     * olyan objektumpéldánnyal visszatérni, amely tartalmazza a paraméterként
     * átadott elérési útvonalat.
     *
     * Az útvonal komponens lehet akár üres vagy abszolút (perjellel kezdve) esetleg
     * relatív (perjel nélkül az elején). Az implementációknak támogatniuk KELL
     * mindhárom szintaxist.
     *
     * Ha egy HTTP útvonalat inkább a gazdagéphez viszonyítva adnak meg, akkor
     * perjellel ("/") kell kezdődnie. A relatív HTTP útvonalak nem kezdődnek
     * perjellel feltételezve, hogy az alkalmazás vagy a felhasználó számára
     * ismert valamely kiindulási ponthoz (base path) viszonyítjuk őket.
     *
     * A felhasználók url-kódolt vagy dekódolt szöveget is megadhatnak itt.
     * Az implementációk a getPath() metódus leírásában vázoltak szerint
     * biztosíthatják a helyes kódolást.
     *
     * @param string $path Az új példányhoz rendelt útvonal komponens.
     * @return static Az új példány a megadott útvonallal.
     * @throws \InvalidArgumentException érvénytelen útvonal argumentum esetén.
     */
    public function withPath($path);

    /**
     * Meghatározott lekérdezési komponenst tartalmazó új objektumpéldánnyal
     * tér vissza.
     *
     * Ennek a metódusnak meg KELL őriznie az eredeti példány állapotát és
     * olyan objektumpéldánnyal visszatérni, amely tartalmazza a paraméterként
     * átadott lekérdezési karakterláncot.
     *
     * A felhasználók url-kódolt vagy dekódolt szöveget is megadhatnak itt.
     * Az implementációk a getQuery() metódus leírásában vázoltak szerint
     * biztosíthatják a helyes kódolást.
     *
     * Ha ezt a metódust üresen hagyott lekérdezési komponens paraméterrel
     * hívják meg, az a komponens eltávolításával egyenértékű.
     *
     * @param string $query Az új példányhoz rendelt lekérdezési komponens.
     * @return static Az új példány a megadott lekérdezési komponenssel.
     * @throws \InvalidArgumentException érvénytelen lekérdezési komponens esetén.
     */
    public function withQuery($query);

    /**
     * Meghatározott részerőforrás azonosító komponenst tartalmazó új
     * objektumpéldánnyal tér vissza.
     *
     * Ennek a metódusnak meg KELL őriznie az eredeti példány állapotát és
     * olyan objektumpéldánnyal visszatérni, amely tartalmazza a paraméterként
     * átadott részerőforrás azonosítót.
     *
     * A felhasználók url-kódolt vagy dekódolt szöveget is megadhatnak itt.
     * Az implementációk a getFragment() metódus leírásában vázoltak szerint
     * biztosíthatják a helyes kódolást.
     *
     * Ha ezt a metódust üresen hagyott részerőforrás azonosító paraméterrel
     * hívják meg, az a részerőforrás azonosító eltávolításával egyenértékű.
     *
     * @param string $fragment Az új példányhoz rendelt részerőforrás azonosító.
     * @return static Az új példány a megadott részerőforrás azonosítóval.
     */
    public function withFragment($fragment);

    /**
     * Egy karakterláncként ábrázolt URI hivatkozással tér vissza.
     *
     * Attól függően, hogy melyik URI komponens van jelen, a kapott karakterlánc
     * vagy a teljes URI-t magában foglalja, vagy relatív hivatkozást tartalmaz
     * az RFC 3986 4.1. fejezetének megfelelően. A metódus a megfelelő elválasztó
     * karakterek alkalmazásával egymáshoz fűzi az URI különféle komponenseit:
     *
     * - Ha a séma jelen van, azt kettősponttal (":") KELL lezárni (pl. "http:").
     * - Ha a hitelesítési komponens jelen van, az elé kettős perjelet ("//")
     *   KELL tenni (pl. "//password:user").
     * - Az útvonal elválasztó karakter nélkül is hozzáfűzhető az URI már meglévő
     *   részéhez. De létezik két olyan eset is, ahol az útvonalat megfelelően be
     *   kell állítani az érvényes URI hivatkozás létrehozásához, mivel a PHP a
     *   __toString() metódusban nem engedélyezi kivétel dobását:
     *     - Ha az útvonal relatív és a hitelesítési komponens is jelen van, akkor
     *       az útvonalat perjellel ("/") KELL kezdeni.
     *     - Ha az útvonal egynél több perjellel kezdődik és nincs jelen a
     *       hitelesítési komponens, akkor a kezdő dupla perjelet egyre KELL
     *       csökkenteni.
     * - Ha a lekérdezési komponens jelen van, az kérdőjellel ("?") KELL kezdeni
     *   (pl. "?key=value").
     * - Ha a részerőforrás azonosító jelen van, azt kettős kereszttel ("#") KELL
     *   kezdeni (pl. "#fragment").
     *
     * @see https://github.com/dominicus75/fig-standards/blob/master/
     *      related-rfcs/3986.md#41-uri-hivatkoz%C3%A1s
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
 * Egy HTTP kérelmen keresztül feltöltött állományt reprezentáló érték objektum.
 *
 * A jelen interfészt megvalósító objektumok megváltoztathatatlanok (immutable)
 * ezért az összes olyan metódust, amely megváltoztathatja az objektum állapotát,
 * úgy KELL megvalósítani, hogy megőrizze az aktuális üzenet belső állapotát és
 * egy másik, a megváltozott állapotot tartalmazó objektumpéldánnyal térjen vissza.
 */
interface UploadedFileInterface
{
    /**
     * Lekéri a feltöltött állományt reprezentáló adatfolyamot.
     *
     * Ennek a metódusnak egy a feltöltött állományt reprezentáló StreamInterface
     * objektumpéldányt KELL visszaadnia. E metódus célja, hogy lehetővé tegye
     * a natív PHP adatfolyamokhoz kapcsolódó funkcionalitásának, mint például a
     * stream_copy_to_stream() függvény kihasználását az állományfeltöltés
     * manipulálására (bár ahhoz, hogy ezek a függvények működjenek, az eredményt
     * egy natív PHP stream wrapper-be kell csomagolni).
     *
     * Ha korábban meghívásra került a moveTo() metódus, ennek a metódusnak
     * kivételt KELL dobnia.
     *
     * @return StreamInterface A feltöltött állományt reprezentáló adatfolyam.
     * @throws \RuntimeException abban az esetben, amikor nem érhető el adatfolyam.
     * @throws \RuntimeException abban az esetben, amikor nem hozható létre adatfolyam.
     */
    public function getStream();

    /**
     * Áthelyezi a feltöltött állományt egy új helyre.
     *
     * Ez a metódus a move_uploaded_file() alternatívájaként is használható.
     * Garantáltan működni kell SAPI és nem SAPI környezetekben is.
     * Az implementációknak meg kell határozniuk, hogy melyik környezetben vannak
     * és ez alapján alkalmazni a megfelelő metódust (move_uploaded_file(),
     * rename(), vagy valamilyen stream művelet) a művelet végrehajtásához.
     *
     * A $targetPath paraméterben megadható relatív és abszolút útvonal is.
     * Relatív útvonal esetén az eredménynek azonosanak kell lennie a PHP rename()
     * függvényének kimenetével.
     *
     * Az eredeti állományt vagy adatfolyamot az eljárás befejezése után törölni
     * KELL.
     *
     * Ha ez a metódus egynél többször kerül meghívásra, bármely későbbi hívásnak
     * kivételt KELL dobnia.
     *
     * SAPI környezetben való alkalmazás esetén, ahol a $_FILES szuperglobális
     * tömb adatokkal van feltöltve, amikor állományt kell írni a moveTo(),
     * is_uploaded_file() és a move_uploaded_file() függvény segítségével, ezt a
     * metódust AJÁNLOTT a jogosultságok és a feltöltési állapot megfelelő
     * ellenőrzésének biztosítására használni.
     *
     * Adatfolyamba történő átmozgatáshoz ajánlott a getStream() metódust
     * használni, mert a SAPI műveletek nem garantálják az adatfolyamba való írást.
     *
     * @see http://php.net/is_uploaded_file
     * @see http://php.net/move_uploaded_file
     * @param string $targetPath útvonal, ahová a feltöltött állománynak kerülni kell.
     * @throws \InvalidArgumentException ha a $targetPath érvénytelen.
     * @throws \RuntimeException ha bármilyen hiba lép fel áthelyezés közben.
     * @throws \RuntimeException a metódus második vagy sokadik hívása esetén.
     */
    public function moveTo($targetPath);

    /**
     * Lekéri a feltöltött állomány méretét.
     *
     * Az implementációknak (amennyiben elérhető) a $_FILES szuperglobális tömb
     * jelen állományhoz tartozó "size" kulcsa alatt tárolt, a PHP által az átvitt
     * tényleges méret alapján számított értéket KELLENE visszaadni.
     *
     * @return int|null Az állomány mérete byte-ban, vagy null, ha ismeretlen.
     */
    public function getSize();

    /**
     * Lekéri a feltöltött állományhoz kapcsolódó hibákat.
     *
     * A visszatérési értéknek a PHP UPLOAD_ERR_XXX konstansaiban tároltak közül
     * KELL kikerülnie.
     *
     * Ha az állomány feltöltése sikeres volt, akkor ennek a metódusnak az
     * UPLOAD_ERR_OK konstans értékével KELL visszatérnie.
     *
     * Az implementációknak a $_FILES szuperglobális tömb jelen állományhoz
     * tartozó "error" kulcsa alatt tárolt értéket KELLENE visszaadni.
     *
     * @see http://php.net/manual/en/features.file-upload.errors.php
     * @return int Egy szám a PHP UPLOAD_ERR_XXX konstansaiban tároltak közül.
     */
    public function getError();

    /**
     * Lekérdezi a kliens által küldött fájl eredeti nevét.
     *
     * Nem igazán ajánlott megbízni a jelen metódus által visszaadott értékben.
     * A kliens küldhet akár egy rosszindulatú kódot tartalmazó állománynevet is,
     * azzal a szándékkal, hogy meghekkelje az alkalmazást.
     *
     * Az implementációknak a $_FILES szuperglobális tömb jelen állományhoz
     * tartozó "name" kulcsa alatt tárolt értéket KELLENE visszaadni.
     *
     * @return string|null A kliens által küldött fájl eredeti neve, vagy null,
     *                     ha nem érkezett ilyen.
     */
    public function getClientFilename();

    /**
     * Lekérdezi a kliens által küldött média típusát.
     *
     * Nem igazán ajánlott megbízni a jelen metódus által visszaadott értékben.
     * A kliens küldhet akár egy rosszindulatú kódot tartalmazó média típust is,
     * azzal a szándékkal, hogy meghekkelje az alkalmazást.
     *
     * Az implementációknak a $_FILES szuperglobális tömb jelen állományhoz
     * tartozó "type" kulcsa alatt tárolt értéket KELLENE visszaadni.
     *
     * @return string|null A kliens által küldött fájl média típusa, vagy null,
     *                     ha nem érkezett ilyen.
     */
    public function getClientMediaType();
}
~~~

[Kezdőlap](../README.md)
