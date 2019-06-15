> **Mottó gyanánt:** *"Noha a PHP-FIG egyik szabványa sem mentes a hibáktól, sőt néha fájó hülyeségeket tartalmaznak, a munkájuk mégis komoly előrelépést jelent a különböző frameworkök, függvénykönyvtárak közötti adatátadás tekintetében, és sokkal átláthatóbbá teszi a kódot."* Pásztor János: [Félreértett programnyelvek - PHP](https://www.refaktor.hu/felreertett-programnyelvek-php/)


PHP Keretrendszerközi Együttműködési Munkacsoport
================================================

Immár tizedik évét taposó [munkacsoportunk](personnel.md) létrehozását az az ötlet inspirálta, hogy az egyes nagyobb projektek (jellemzően keretrendszerek) fejlesztői számára olyan fórumot biztosítsunk, ahol felismerhetik projektjeik esetleges közös pontjait és megállapodhatnak a közös munka módozataiban. Fő célunk a munkánk közmegegyezésen alapuló összehangolása, de tudatában vagyunk annak, hogy a PHP-közösség többi része is figyelemmel kíséri a tevékenységünk. Ha mások is elfogadják és alkalmazzák amit itt közösen létrehozunk, annak csak örülni tudunk, de nem ez az elsődleges célunk.

Honosítás
---------

Az összes PSR-szabványleírás brit (en_GB) vagy amerikai (en_US) angol nyelven íródott (néhány ettől esetleg eltérhet, de az egyes specifikációkon belül ragaszkodunk a következetességhez). A PHP FIG nem kínál hivatalos fordításokat más nyelvekre, viszont bárki szabadon lefordíthatja e szabványokat a saját nyelvére, összhangban a [licenc-feltételekkel](LICENSE.md).


# PHP szabvány ajánlások

A PSR munkafolyamat-szabályzat szerint minden egyes PSR-ajánlás rendelkezik egy a munkafolyamat állapotát tükröző státusszal. Minden egyes beérkező javaslatnak először egy belépő szavazáson kell átesnie, s ha ezt az akadályt sikeresen veszi, akkor *"vázlat"* címkével ellátva megjelenik az alábbi listán. Ha a PSR már mint  *"elfogadott"* van megjelölve, az később még változhat (idővel lehet pl. *"elavult"*, lásd. psr-0; illetve *"gazdátlan"*, mint a PSR-8). Egy *"vázlat"*, mielőtt *"elfogadott"* állapotba kerülne, még lényeges változásokon mehet keresztül, ellenben a *"felülvizsgált"* ajánlások csak kisebb eltéréseket mutathatnak az *"elfogadott"* állapothoz képest.

A jelen fordítás csak a már elfogadott (esetleg már elavult) ajánlásokat tartalmazza, a vázlatokat és a gazdátlanná vált dokumentumokat nem, ezek az [eredeti dokumentációban](https://github.com/php-fig/fig-standards) találhatók meg, angol nyelven.


| Szám | Szabvány neve                                | Szerkesztő(k)                     | Állapot     |
|:---:|--------------------------------------|--------------------------------|------------|
| 0   | [Önbetöltő szabvány][psr0]         | Matthew Weier O'Phinney        | elavult |
| 1   | [Alapvető kódolási szabvány][psr1]        | Paul M. Jones                  | elfogadott   |
| 2   | [Kódstílus útmutató][psr2]           | Paul M. Jones                  | elfogadott   |
| 3   | [Naplózó interfész][psr3]             | Jordi Boggiano                 | elfogadott   |
| 4   | [Önbetöltő szabvány][psr4]         | Paul M. Jones                  | elfogadott   |
| 6   | [Gyorsítótár interfész][psr6]            | Larry Garfield                 | elfogadott   |
| 7   | [HTTP-üzenet interfész][psr7]       | Matthew Weier O'Phinney        | elfogadott   |
| 11  | [Tároló interfész][psr11]         | Matthieu Napoli, David Négrier | elfogadott   |
| 13  | [Hypermédia hivatkozások][psr13]            | Larry Garfield                 | elfogadott   |
| 14  | [Esemény-kezelő][psr14]            | Larry Garfield                 | elfogadott   |
| 15  | [HTTP-kezelők][psr15]               | Woody Gilk                     | elfogadott   |
| 16  | [Egyszerű gyorsítótár][psr16]                | Paul Dragoonis                 | elfogadott   |
| 17  | [HTTP-gyárak][psr17]              | Woody Gilk                     | elfogadott   |
| 18  | [HTTP-kliens][psr18]                 | Tobias Nyholm                  | elfogadott   |


[psr0]: accepted/PSR-0.md
[psr1]: accepted/PSR-1-basic-coding-standard.md
[psr2]: accepted/PSR-2-coding-style-guide.md
[psr3]: accepted/PSR-3-logger-interface.md
[psr4]: accepted/PSR-4-autoloader.md
[psr6]: accepted/PSR-6-cache.md
[psr7]: accepted/PSR-7-http-message.md
[psr11]: accepted/PSR-11-container.md
[psr13]: accepted/PSR-13-links.md
[psr14]: accepted/PSR-14-event-dispatcher.md
[psr15]: accepted/PSR-15-request-handlers.md
[psr16]: accepted/PSR-16-simple-cache.md
[psr17]: accepted/PSR-17-http-factory.md
[psr18]: accepted/PSR-18-http-client.md
