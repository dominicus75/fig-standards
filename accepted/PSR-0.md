[Kezdőlap](../README.md)

Önbetöltő (autoloader) szabvány
===============================

> **Elavult** - 2014. október 21-től a PSR-0 ajánlást elavultnak jelölték. Helyette a [PSR-4] szabvány használata ajánlott.

[PSR-4]: PSR-4-autoloader.md

Az alábbiakban ismertetjük azokat a feltétlenül szükséges követelményeket, amelyeket be kell tartani a kölcsönös használhatóság érdekében.

Kötelező
--------

* A teljesen minősített névtereknek és osztályoknak a következő sémát kell követnie: `\<Gyártó_neve>\(<Névtér>\)*<Osztálynév>`
* Minden egyes névtérnek rendelkeznie kell egy legfelsőbb szintű névtérrel ("Gyártó").
* Minden egyes névtér annyi alnévtérrel rendelkezhet, amennyit nem szégyell.
* Minden egyes névtér-elválasztót könyvtár elválasztóvá (`DIRECTORY_SEPARATOR` beépített konstans) kell alakítani, amikor betöltődik.
* Minden aláhúzás (`_`) karaktert az osztály nevében könyvtár elválasztóvá kell konvertálni betöltéskor. Az aláhúzás karakternek ezen felül nincs speciális jelentése a névterekben.
* A teljesen minősített névterek és osztályok neve mindig `.php` végződésű, amikor betöltődnek a fájlrendszerből.
* Kis és nagy betűk bármilyen kombinációban előfordulhatnak a gyártó, a névterek és az osztályok nevében.

Példák
------

* `\Doctrine\Common\IsolatedClassLoader` => `/path/to/project/lib/vendor/Doctrine/Common/IsolatedClassLoader.php`
* `\Symfony\Core\Request` => `/path/to/project/lib/vendor/Symfony/Core/Request.php`
* `\Zend\Acl` => `/path/to/project/lib/vendor/Zend/Acl.php`
* `\Zend\Mail\Message` => `/path/to/project/lib/vendor/Zend/Mail/Message.php`

Aláhúzások a névterekben és osztálynevekben
-------------------------------------------

* `\namespace\package\Class_Name` => `/path/to/project/lib/vendor/namespace/package/Class/Name.php`
* `\namespace\package_name\Class_Name` => `/path/to/project/lib/vendor/namespace/package_name/Class/Name.php`

Az itt meghatározott szabványoknak kellene lenniük a legkisebb közös nevezőnek az autoloader fájdalommentes kölcsönös használhatóságához. Ki is próbálhatjuk, hogy ezeket a szabályokat követve ez a példa SplClassLoader implementáció
miként képes betölteni a PHP 5.3 osztályait.

Példa implementáció
-------------------

Alant egy minta-függvény, amely egyszerűen szemlélteti, hogy a fenti szabályok működnek.

~~~php
<?php

function autoload($className)
{
  $className = ltrim($className, '\\');
  $fileName  = '';
  $namespace = '';
  if ($lastNsPos = strrpos($className, '\\')) {
    $namespace = substr($className, 0, $lastNsPos);
    $className = substr($className, $lastNsPos + 1);
    $fileName  = str_replace('\\', DIRECTORY_SEPARATOR, $namespace) . DIRECTORY_SEPARATOR;
  }
  $fileName .= str_replace('_', DIRECTORY_SEPARATOR, $className) . '.php';

  require $fileName;
}

spl_autoload_register('autoload');

~~~

SplClassLoader megvalósítás
---------------------------

A következő gist egy egyszerű SplClassLoader megvalósítást mutat be, amely gond nélkül képes betölteni azon osztályokat, amelyek elnevezésében követték a fenti kölcsönös használhatósági szabályokat.

* [http://gist.github.com/221634](http://gist.github.com/221634)

[Kezdőlap](../README.md)

