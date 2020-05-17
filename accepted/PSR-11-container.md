# Tároló interfész

Ez a dokumentum egy általános interfészt határoz meg a függőségbefecskendező tárolók
 ([Dependency Injection Container](https://www.letscode.hu/2016/05/09/tiszta-kod-7-resz-mi-a-fene-az-a-dependency-injection-container/) - a továbbiakban: **DIC** vagy **DI-tároló**) számára.

A `ContainerInterface` célja, hogy szabványosítsa a keretrendszerek és függvénykönyvtárak
számára azt, hogyan használják a DI-tárolót a különféle objektumok és paraméterek
(a továbbiakban: **bejegyzések**) eléréséhez.

A csupa nagybetűvel szedett, a követelmények szintjének jelzésére szolgáló kiemelt
kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

A `megvalósító` kifejezés ebben a dokumentumban arra a kódra értendő, ami megvalósítja a
`ContainerInterface` előírásait, egy függőségbefecskendezéssel kapcsolatos függvénykönyvtárban
vagy keretrendszerben. A DI-tárolókat alkalmazó (meghívó) kódokra a jelen dokumentum
`felhasználó`-ként hivatkozik.

## 1. Specifikáció

### 1.1 Alapok

#### 1.1.1 Bejegyzésazonosítók

Egy bejegyzésazonosító lehet bármilyen a PHP által elfogadott, legalább egy karakter
hosszúságú karakterlánc, ami egyedileg azonosít egy elemet a DI-tárolóban. A
bejegyzésazonosító alapján a klienskódnak NEM KELLENE feltételezni, hogy annak felépítése
hordoz bármiféle jelentést.

#### 1.1.2 Olvasás a DI-tárolóból

- A `Psr\Container\ContainerInterface` két metódust ír elő: `get` és `has`.

- A `get` egy kötelező paramétert vár: egy bejegyzésazonosítót, amelynek karakterláncnak
  KELL lennie. A `get` visszatérhet bármivel (egy *vegyes* értékkel), vagy dobhat
  egy `NotFoundExceptionInterface` típusú kivételt, ha az azonosító nem található a DI-tárolóban.
  A `get` metódus két egymást követő, ugyanazon azonosítóval történő meghívásának
  ugyanazt a visszatérési értéket KELLENE produkálnia. Viszont a `megvalósító`
  kialakításától és/vagy a `felhasználó` konfigurációjától függően a `get` metódus
  különböző értékekkel is visszatérhet, ezért a `felhasználónak` NEM KELLENE támaszkodnia
  arra, hogy ugyanazt az értéket kapja két egymást követő hívás eredményéül.

- A `has` metódus egy egyedi paramétert vár: egy bejegyzésazonosítót, amelynek karakterláncnak
  KELL lennie. A `has` metódusnak `true` értékkel KELL visszatérnie, ha a
  bejegyzésazonosító ismert a DI-tároló számára, illetve `false` értékkel, ha nem.
  Amikor a `has($id)` `false` értéket ad vissza, akkor a `get($id)` egy `NotFoundExceptionInterface`
  típusú kivételt KELL dobnia.

### 1.2 Kivételek

A DI-tároló által közvetlenül dobható kivételeknek a
[`Psr\Container\ContainerExceptionInterface`](#container-exception) által reprezentált
típusokat KELLENE megvalósítani.

A `get` metódus nem létező azonosítóval történő meghívása esetén egy
[`Psr\Container\NotFoundExceptionInterface`](#not-found-exception) típusú kivételt
KELL dobni.

### 1.3 Ajánlott használat

A felhasználónak NEM AJÁNLOTT átadni DI-tárolót egy objektumba, mert így a fogadó
objektum *a saját függőségeit* fogja önmagába fecskendezni. Ez azt jelentené, hogy a
DI-tárolót [Service Locator](https://hu.wikipedia.org/wiki/Szolg%C3%A1ltat%C3%A1slok%C3%A1tor)
gyanánt alkalmazzuk, ami általában kerülendő.

## 2. A csomag tartalma

Az interfészek és osztályok által leírt és releváns kivételek a
[psr/container](https://packagist.org/packages/psr/container) csomag részét képezik.

Azon csomagoknak amelyek megvalósítják a PSR Container szabványt, ezt ajánlott külön
jelezniük is: `psr/container-implementation` `1.0.0`.

## 3. Interfészek

<a name="container-interface"></a>
### 3.1. `Psr\Container\ContainerInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * Egy olyan tároló interfészt ír le, amely deklarálja a saját bejegyzéseinek
 * olvasásához szükséges metódusokat.
 */
interface ContainerInterface
{
    /**
     * A megadott azonosító alapján megkeres és visszaad egy bejegyzést a tárolóból.
     *
     * @param string $id A keresett bejegyzés azonosítója.
     *
     * @throws NotFoundExceptionInterface  Nem található bejegyzés **ezzel** az azonosítóval.
     * @throws ContainerExceptionInterface A bejegyzés visszaadása során fellépett hiba esetén.
     *
     * @return mixed Bejegyzés.
     */
    public function get($id);

    /**
     * True értékkel tér vissza, ha a paraméterül kapott azonosítóhoz tartozik
     * bejegyzés, egyébként false-t ad vissza.
     *
     * Ha a has($id) true-val tér vissza, az még nem jelenti azt, hogy a `get($id)`
     * esetleg nem fog kivételt dobni, csak azt, hogy `NotFoundExceptionInterface`
     * típusút biztosan nem.
     *
     * @param string $id A keresett bejegyzés azonosítója.
     *
     * @return bool
     */
    public function has($id);
}
~~~

<a name="container-exception"></a>
### 3.2. `Psr\Container\ContainerExceptionInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * Alap interfész, amely egy általános kivételt reprezentál a tárolóban.
 */
interface ContainerExceptionInterface
{
}
~~~

<a name="not-found-exception"></a>
### 3.3. `Psr\Container\NotFoundExceptionInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * A kért bejegyzés nem található a tárolóban.
 */
interface NotFoundExceptionInterface extends ContainerExceptionInterface
{
}
~~~

### Jelen szabványt megvalósító projektek:
- [Acclimate](https://github.com/jeremeamia/acclimate-container)
- [Aura.DI](https://github.com/auraphp/Aura.Di)
- [dcp-di](https://github.com/estelsmith/dcp-di)
- [League Container](https://github.com/thephpleague/container)
- [Mouf](http://mouf-php.com)
- [Njasm Container](https://github.com/njasm/container)
- [PHP-DI](http://php-di.org)
- [PimpleInterop](https://github.com/moufmouf/pimple-interop)
- [XStatic](https://github.com/jeremeamia/xstatic)
- [Zend ServiceManager](https://github.com/zendframework/zend-servicemanager)
