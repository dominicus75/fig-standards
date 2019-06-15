# Önbetöltő (autoloader) szabvány

A csupa nagybetűvel szedett kiemelt kulcsszavak ebben a leírásban az [RFC 2119](../related-rfcs/2119.md) szerint értelmezendők.

## 1. Áttekintés

Ez a PSR az osztályok fájlelérési út szerinti [automatikus betöltését][] írja le. Teljesen interoperábilis és használható bármely más önbetöltő specifikáció mellett, beleértve az elavult [PSR-0][] ajánlást is. Azt is meghatározza, hová kell helyezni azokat a fájlokat, amelyek jelen specifikáció szerint lesznek automatikusan betöltve.

## 2. Specifikáció

1. Az "osztály" kifejezés utalhat osztályokra, interfészekre, trétekre vagy más hasonló struktúrákra.

2. Egy teljesen minősített osztálynévnek a következő sémát kell követnie:

    \<Névtér>(\<Alnévtér>)*\<Osztálynév>

    1. Egy teljesen minősített osztálynévnek rendelkeznie KELL egy legfelső szintű, más néven "gyártó" (vendor) névtérrel.

    2. Egy teljesen minősített osztálynévnek LEHET egy vagy több alnévtere is.

    3. Egy teljesen minősített osztálynevet a tulajdonképpeni osztálynévnek KELL lezárnia.

    4. Az aláhúzás karaktereknek nincs semmilyen speciális jelentése a teljesen minősített osztálynév bármely részében.

    5. Az alfanumerikus karakterek a kis-, és nagybetűk TETSZŐLEGES kombinációjában szerepelhetnek a teljesen minősített osztálynévben.

    6. Valamennyi osztálynévre kis-, és nagybetű-érzékeny módon KELL hivatkozni.

3. Amikor olyan fájl töltődik be, ami megfelel a teljesen minősített osztálynévnek, akkor

    1. az egy vagy több fő-, és alnévtér-név folyamatos sora, amely nem tartalmazza az alapkönyvtárnak megfelelő főnévtér elválasztót (más néven: névtér előtagot) a teljesen minősített osztálynévben.

    2. a névtér előtagot követő alnévtér nevek megfelelnek az alapkönyváron belül található alkönyvtáraknak, mégpedig úgy, hogy a névtér elválasztók (`\`) jelképezik a könyvtár elválasztó karaktereket (a `DIRECTORY_SEPARATOR` beépített konstans értéke). Az alkönyvtárak neveinek betűre egyezniük KELL az alnévterek elnevezésével.

    3. a teljesen minősített név végén szereplő osztálynév megfelel a kért osztályt tartalmazó `.php` kiterjesztésű állománynak. Az állományneveknek betűre egyezniük KELL az osztályok neveivel.

4. A jelen PSR szerinti autoloader megvalósításoknak TILOS kivételt dobni és bármilyen szintű hibát okoznia és NEM AJÁNLOTT értéket visszaadnia.

## 3. Példák

Az alábbi tábla a teljesen minősített osztálynévhez tartozó névtér előtagot, osztálynevet és alapkönyvtárat társítja a megfelelő fájlelérési úthoz.

| Teljesen minősített osztálynév | Névtér előtag     | Alapkönyvtár        | Kapott fájlelérési út
| ----------------------------- |--------------------|--------------------------|-------------------------------------------
| \Acme\Log\Writer\File_Writer  | Acme\Log\Writer    | ./acme-log-writer/lib/   | ./acme-log-writer/lib/File_Writer.php
| \Aura\Web\Response\Status     | Aura\Web           | /path/to/aura-web/src/   | /path/to/aura-web/src/Response/Status.php
| \Symfony\Core\Request         | Symfony\Core       | ./vendor/Symfony/Core/   | ./vendor/Symfony/Core/Request.php
| \Zend\Acl                     | Zend               | /usr/includes/Zend/      | /usr/includes/Zend/Acl.php

[automatikus betöltését]: http://php.net/autoload
[PSR-0]: PSR-0.md
