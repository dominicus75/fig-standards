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

- Every method accepts an array as context data. This is meant to hold any
  extraneous information that does not fit well in a string. The array can
  contain anything. Implementors MUST ensure they treat context data with
  as much lenience as possible. A given value in the context MUST NOT throw
  an exception nor raise any php error, warning or notice.

- If an `Exception` object is passed in the context data, it MUST be in the
  `'exception'` key. Logging exceptions is a common pattern and this allows
  implementors to extract a stack trace from the exception when the log
  backend supports it. Implementors MUST still verify that the `'exception'`
  key is actually an `Exception` before using it as such, as it MAY contain
  anything.

### 1.4 Segítő (helper) osztályok és interfészek

- The `Psr\Log\AbstractLogger` class lets you implement the `LoggerInterface`
  very easily by extending it and implementing the generic `log` method.
  The other eight methods are forwarding the message and context to it.

- Similarly, using the `Psr\Log\LoggerTrait` only requires you to
  implement the generic `log` method. Note that since traits can not implement
  interfaces, in this case you still have to implement `LoggerInterface`.

- The `Psr\Log\NullLogger` is provided together with the interface. It MAY be
  used by users of the interface to provide a fall-back "black hole"
  implementation if no logger is given to them. However, conditional logging
  may be a better approach if context data creation is expensive.

- The `Psr\Log\LoggerAwareInterface` only contains a
  `setLogger(LoggerInterface $logger)` method and can be used by frameworks to
  auto-wire arbitrary instances with a logger.

- The `Psr\Log\LoggerAwareTrait` trait can be used to implement the equivalent
  interface easily in any class. It gives you access to `$this->logger`.

- The `Psr\Log\LogLevel` class holds constants for the eight log levels.

## 2. A csomag

The interfaces and classes described as well as relevant exception classes
and a test suite to verify your implementation are provided as part of the
[psr/log](https://packagist.org/packages/psr/log) package.

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
 * A kontextus tömb tartalmazhat tetszőleges adatot, the only assumption that
 * can be made by implementors is that if an Exception instance is given
 * to produce a stack trace, it MUST be in a key named "exception".
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
