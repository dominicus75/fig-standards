[Kezdőlap](../README.md)

Naplózó Interfész
=================

Ez a dokumentum egy közös programozási felületet ír le a naplózó függvénykönyvtárak számára.

A fő cél az, hogy lehetővé váljon a különböző függvénykönyvtárak számára egy
`Psr\Log\LoggerInterface`-típusú objektum fogadása és ennek segítségével az egyszerű
és univerzális naplózás. Az egyedi igényeket kielégítő keretrendszerek és
tartalomkezelők (CMS) céljainak megfelelően ki is LEHET terjeszteni az interfészt,
de ezeknek a változatoknak továbbra is kompatibilisnek kell maradni ezzel a
dokumentummal. Ez biztosítja azt, hogy a harmadik féltől származó, szintén jelen
PSR-t megvalósító függvénykönyvtárak is képesek legyenek írni egy központi
alkalmazás-naplóba.

A csupa nagybetűvel szedett kiemelt kulcsszavak ebben a leírásban az
[RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

A `megvalósító` alatt ebben a dokumentumban azt a személyt értjük, aki implementálja
a `LoggerInterface`-t egy naplózáshoz kapcsolódó függvénykönyvtárban, vagy
keretrendszerben. Az így elkészült naplózókat használókra a szöveg `felhasználóként`
hivatkozik.

## 1. Specifikáció

### 1.1 Alapok

- A `LoggerInterface` nyolc olyan metódust ír elő, amelyek az [RFC 5424][] nyolc
  szintjének (debug, info, notice, warning, error, critical, alert, emergency)
  megfelelően naplózzák az eseményeket.

- A kilencedik, `log` névre hallgató metódus első paraméterként a naplózási
  szintre kíváncsi. Meghívva ezt a metódust a 8 naplózási szint egyikét reprezentáló
  konstanssal, azonos eredményt KELL kapni, mint a naplózási szint-specifikus
  metódus hívása esetén. Ha a jelen specifikáció által nem definiált, vagy az
  ezt implementáló osztály által nem ismert naplózási szinttel kerül meghívásra
  a metódus, akkor egy `Psr\Log\InvalidArgumentException`-típusú kivételt KELL
  dobnia. A felhasználóknak NEM KELLENE egyéni naplózási szinteket használni,
  anélkül, hogy biztosan tudnák, hogy az aktuális implementáció támogatja-e azt.

[RFC 5424]: ../related-rfcs/2119.md

### 1.2 Az üzenet

- Mindegyik metódus képes fogadni egyszerű szövegként átadott üzenetet vagy olyan objektumot,
  amelyik rendelkezik [`__toString()`](http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=10#section_8_2) metódussal. A megvalósítóknak LEHET különlegesen kezelni az átadott objektumokat.
  Ha nem ez a helyzet, akkor a szöveggé KELL konvertálni őket.

- Az üzenet szövegében LEHET helyőrzőket is elhelyezni, amelyeket a megvalósítónak
  ki LEHET cserélni a kontextus tömbből származó értékekre.

  A helyőrzők neveinek meg KELL felelniük a kontextus tömb kulcsainak.

  A helyőrzők neveit kapcsos zárójellel (`{}`) KELL határolni. A határoló karakterek és a
  helyőrzőnév között NEM LEHET semmilyen térköz (whitespace) karakter.

  A helyőrzők neveinek kizárólag az angol ábécé kis-, és nagybetűit, számokat, aláhúzás
  karaktereket és pontot KELLENE tartalmazni. Más karakterek használata a helyőrzők
  specifikációjának esetleges későbbi módosításától függ.

  A megvalósítóknak LEHET helyőrzőket használni különféle védő(escape)karakter-stratégiák
  megvalósításához és a naplók megjelenítéshez kapcsolódó formázásához.
  A felhasználóknak viszont NEM KELLENE védőkarakterrel előre ellátott helyőrzőket
  használni, mivel nem tudhatják, hogy milyen kontextusban lesz az adat megjelenítve.

  Az alábbiakban egy példa implementáció látható, amely helyőrző közbeszúrást
  valósít meg:

  ~~~php
  <?php

  /**
   * A kontextus tömb elemeinek közbeszúrása az üzenetben található helyőrzők helyére.
   */
  function interpolate($message, array $context = array())
  {
      // Csere-tömb létrehozása a kontextus tömb kapcsos zárójelbe tett kulcsaival indexelve
      $replace = array();
      foreach ($context as $key => $val) {
          // Ellenőrzi, hogy az érték átkonvertálható-e szöveggé
          if (!is_array($val) && (!is_object($val) || method_exists($val, '__toString'))) {
              $replace['{' . $key . '}'] = $val;
          }
      }

      // a csere-tömb értékeinek beszúrása az üzenetbe, majd visszatérés a kész szöveggel
      return strtr($message, $replace);
  }

  // üzenet kapcsos zárójelekkel határolt helyőrzőnévvel
  $message = "{username} felhasználó létrehozva";

  // a kontextus tömb a helyőrző nevekhez rendelt csere értékekkel
  $context = array('username' => 'bolivar');

  // "bolivar felhasználó létrehozva" szöveg kiíratása
  echo interpolate($message, $context);
  ~~~

### 1.3 Kontextus

- Minden metódus egy tömböt vár kontextus adat gyanánt. Ez a tömb tartalmazhat
  bármilyen külső információt, akár olyat is, ami nem illeszkedik jól
  egy karakterláncba. A megvalósítóknak biztosítaniuk KELL azt, hogy a
  kontextusadatokat minél kisebb engedékenységgel kezeljék. A kontextus tömbben
  átadott értékeknek TILOS kivételt dobniuk, PHP hibát (error), figyelmeztetést
  (warning) vagy értesítést (notice) kiváltania.

- Ha egy `Exception` objektum kerül átadásra a kontextus adatok között, akkor annak
  a kontextus tömb `'exception'` kulcsa alatt KELL helyet foglalnia.
  A kivételek (exception) naplózása egy olyan közös minta alapján történik, amely
  lehetővé teszi a megvalósítók számára, hogy veremkivonatot (stack trace) hozzanak
  létre a kapott `Exception` objektumból, ha a napló háttérprogramja támogatja ezt.
  A megvalósítóknak használatbavétel előtt mindig ellenőrizni KELL azt, hogy a
  kontextus tömb `'exception'` kulcsa alatt valóban egy `Exception` objektum található,
  mivel LEHET benne bármilyen típusú adat.

### 1.4 Segítő (helper) osztályok és interfészek

- A `Psr\Log\AbstractLogger` osztály kiterjesztésével és az általános `log` metódus
  implementálásával lehetővé válik a `LoggerInterface` nagyon könnyű megvalósítása.
  A másik nyolc metódus segítségével továbbítható az üzenet és a kontextus tömb.

- Hasonlóképpen a `Psr\Log\LoggerTrait` csak az általános `log` metódus
  implementálását követeli meg. Ne feledjük, hogy a trait nem tud interfészeket
  implementálni, ezért mindig meg kell valósítani a `LoggerInterface`-t is.

- A `Psr\Log\AbstractLogger` osztályt kiterjesztő `Psr\Log\NullLogger` biztosítja
  mindkettőt, interfésszel együtt. Ezt a "black hole" technikát biztosító interfészt
  alkalmazó felhasználóknak LEHET használni, ha nem áll rendelkezésre egy naplózó.
  A feltételes naplózás viszont jobb megközelítés LEHET, ha a kontextus adatok
  létrehozása túl nagy ráfordítással járna.

- A `Psr\Log\LoggerAwareInterface` csupán egy `setLogger(LoggerInterface $logger)`
  metódus megvalósítását írja elő, amelyet a keretrendszerek tudnak használni arra,
  hogy automatikusan összedrótozzanak tetszőleges objektumpéldányokat a naplózóval.

- A `Psr\Log\LoggerAwareTrait` trait segítségével az azonos nevű interfész könnyedén
  megvalósítható bármely osztályban.

- A `Psr\Log\LogLevel` osztály az RFC 5424 nyolc szintjének megfelelően tartalmazza
  a naplózási szinteket.

## 2. A csomag

A szükséges interfészek és osztályok, továbbá a vonatkozó kivétel (Exception) osztályok leírásai,
valamint az egyes implementációk ellenőrzést lehetővé tevő teszt eszközök rendelkezésre állnak
a [psr/log](https://packagist.org/packages/psr/log) csomag részeként.

Telepítése (Composer segítségével, terminálon): `composer require psr/log`.

Példa a PSR-3 megvalósítására: [Monolog](https://github.com/Seldaek/monolog/blob/master/src/Monolog/Logger.php).

## 3. `Psr\Log\LoggerInterface`

~~~php
<?php

namespace Psr\Log;

/**
 * Leír egy naplózó példányt.
 *
 * Az üzenetnek szövegnek KELL lennie, vagy olyan objektumnak, amely
 * implementálja a __toString() mágikus metódust.
 *
 * Az üzenet szövegében LEHET helyőrzőket elhelyezni, a következő formában: {foo} ahol foo
 * ki lesz cserélve a kontextus tömb "foo" indexű elemével.
 *
 * A kontextus tömb tartalmazhat tetszőleges adatot, az egyetlen kikötés a
 * a megvalósítók számára az, hogy ha egy Exception példány van hivatva
 * a veremkivonat (stack trace) létrehozására, annak a kontextus tömb "exception"
 * kulcsa alatt KELL lakni.
 *
 */
interface LoggerInterface
{
    /**
     * A rendszer használhatatlan.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function emergency($message, array $context = array());

    /**
     * Azonnali beavatkozást igényel.
     *
     * Például: a teljes webhely leállt, az adatbázis elérhetetlen, stb.
     * Akár SMS-riasztást is ki kell váltania, hogy felébressze az illetékest.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function alert($message, array $context = array());

    /**
     * Kritikus körülmények.
     *
     * Például: az alkalmazás egyik komponense nem elérhető, váratlan (nem elkapott,
     * kezeletlen) kivétel.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function critical($message, array $context = array());

    /**
     * Futásidejű hibák, amelyek nem igényelnek azonnali beavatkozást, de
     * ajánlott őket ellenőrizni és naplózni.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function error($message, array $context = array());

    /**
     * Hibának nem minősülő rendkívüli események.
     *
     * Például: Elavult API használata, egy API nem megfelelő használata,
     * nem feltétlenül rossz, de nem kívánatos dolgok.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function warning($message, array $context = array());

    /**
     * Normális, de speciális kezelést igényló esemény.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function notice($message, array $context = array());

    /**
     * Érdekes események.
     *
     * Például: felhasználó bejelentkezik.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function info($message, array $context = array());

    /**
     * Részletes hibakeresési információk.
     *
     * @param string $message
     * @param array $context
     * @return void
     */
    public function debug($message, array $context = array());

    /**
     * Naplózás tetszőlegesen választott szinttel.
     *
     * @param mixed $level
     * @param string $message
     * @param array $context
     * @return void
     */
    public function log($level, $message, array $context = array());
}
~~~

## 4. `Psr\Log\LoggerAwareInterface`

~~~php
<?php

namespace Psr\Log;

/**
 * Leír egy naplózó-felismerő példányt.
 */
interface LoggerAwareInterface
{
    /**
     * Elhelyez egy LoggerInterface-t megvalósító
     * naplózó-objektumpéldányt az implementáló objektumban.
     *
     * @param LoggerInterface $logger
     * @return void
     */
    public function setLogger(LoggerInterface $logger);
}
~~~

## 5. `Psr\Log\LogLevel`

~~~php
<?php

namespace Psr\Log;

/**
 * Naplózási szintek leírása.
 */
class LogLevel
{
    const EMERGENCY = 'emergency';
    const ALERT     = 'alert';
    const CRITICAL  = 'critical';
    const ERROR     = 'error';
    const WARNING   = 'warning';
    const NOTICE    = 'notice';
    const INFO      = 'info';
    const DEBUG     = 'debug';
}
~~~

[Kezdőlap](../README.md)
