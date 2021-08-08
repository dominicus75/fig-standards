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


| Szám | Szabvány neve                       | Szerkesztő(k)                  | Állapot      | Nyelv  |
|:---:|--------------------------------------|--------------------------------|--------------|--------|
| 0   | [Önbetöltő szabvány][psr0]           | Matthew Weier O'Phinney        | elavult      | magyar |
| 1   | [Alapvető kódolási szabvány][psr1]   | Paul M. Jones                  | elfogadott   | magyar |
| 2   | [Kódstílus útmutató][psr2]           | Paul M. Jones                  | elavult      | magyar |
| 3   | [Naplózó interfész][psr3]            | Jordi Boggiano                 | elfogadott   | magyar |
| 4   | [Önbetöltő szabvány][psr4]           | Paul M. Jones                  | elfogadott   | magyar |
| 6   | [Gyorsítótár interfész][psr6]        | Larry Garfield                 | elfogadott   | magyar |
| 7   | [HTTP-üzenet interfész][psr7]        | Matthew Weier O'Phinney        | elfogadott   | magyar |
| 11  | [Tároló interfész][psr11]            | Matthieu Napoli, David Négrier | elfogadott   | magyar |
| 12  | [Bővített kódstílus útmutató][psr12] | Korvin Szanto                  | elfogadott   | magyar |
| 13  | [Hypermédia hivatkozások][psr13]     | Larry Garfield                 | elfogadott   | magyar |
| 14  | [Eseményirányító (Event Dispatcher)][psr14] | Larry Garfield          | elfogadott   | magyar |
| 15  | [HTTP-kezelők][psr15]                | Woody Gilk                     | elfogadott   | magyar |
| 16  | [Egyszerű gyorsítótár][psr16]        | Paul Dragoonis                 | elfogadott   | magyar |
| 17  | [HTTP-gyárak][psr17]                 | Woody Gilk                     | elfogadott   | magyar |
| 18  | [HTTP-kliens][psr18]                 | Tobias Nyholm                  | elfogadott   | magyar |


[psr0]: accepted/PSR-0.md
[psr1]: accepted/PSR-1-basic-coding-standard.md
[psr2]: accepted/PSR-2-coding-style-guide.md
[psr3]: accepted/PSR-3-logger-interface.md
[psr4]: accepted/PSR-4-autoloader.md
[psr6]: accepted/PSR-6-cache.md
[psr7]: accepted/PSR-7-http-message.md
[psr11]: accepted/PSR-11-container.md
[psr12]: accepted/PSR-12-extended-coding-style-guide.md
[psr13]: accepted/PSR-13-links.md
[psr14]: accepted/PSR-14-event-dispatcher.md
[psr15]: accepted/PSR-15-request-handlers.md
[psr16]: accepted/PSR-16-simple-cache.md
[psr17]: accepted/PSR-17-http-factory.md
[psr18]: accepted/PSR-18-http-client.md


## Kapcsolódó RFC-ajánlások

A fenti listában szereplő PSR-szabványokhoz szervesen kapcsolódó, kivonatosan
(a hivatkozó PSR jobb megértéséhez szükséges mértékben) lefordított RFC ajánlások.

### Az internetes szabványok

Az internetes szabványok leírását az egyedi számmal ellátott RFC (Request For Comments,
kéretik megkritizálni) dokumentumok tartalmazzák. Ezek soha nem módosulnak, az esetleges
hibajavításokat, frissítéseket, az adott szabvány újabb változatát további RFC
dokumentumok tartalmazzák. Egy új RFC elavulttá tehet (obsolete) vagy frissíthet
(update) korábbi RFC-ket.

Az [IETF](https://hu.wikipedia.org/wiki/IETF) által gondozott RFC dokumentumok két
nagyobb alcsoportra oszthatók:
* **BCP** (Best Current Practice - „jelenlegi legjobb gyakorlat”)
* **STD** (Internet Standard - internetes szabvány).

Mindkét alcsoport tagjai külön (BCP-. illetve STD-,) számot kapnak, miközben megtartják
korábbi RFC-számukat is, így egy BCP-, vagy STD-szám akár több RFC-hez is tartozhat
(ezek ugyanis a szabványt jelölik, nem magát a dokumentumot).

A szabványokat 3 érettségi szintbe sorolják:
* **[Proposed Standard (javasolt szabvány)](https://www.rfc-editor.org/standards#PS)**
* **[Draft Standard (szabványtervezet, 2011-től nem használják)](https://www.rfc-editor.org/standards#DS)**
* **[Internet Standard (internet szabvány)](https://www.rfc-editor.org/standards#IS)**


|   RFC száma                      | Típus/szint |      Ajánlás/szabvány tárgya  |
|----------------------------------|:---------------------:|-------------------------------|
| [RFC 2119](related-rfcs/2119.md) | [BCP 14](https://www.rfc-editor.org/info/bcp14) leírás | Kulcsszavak az RFC-kben a követelmények szintjének jelzésére |
| [RFC 3986](related-rfcs/3986.md) | [STD 66](https://www.rfc-editor.org/info/std66) néven internetes szabvány | Egységes Erőforrás-azonosító (URI) |
| [RFC 5234](related-rfcs/5234.md) | [STD 68](https://www.rfc-editor.org/info/std68) néven internetes szabvány | Bővített Backus–Naur Forma (ABNF) |
| [RFC 5424](related-rfcs/5424.md) | Javasolt szabvány | Rendszernapló protokoll |
| [RFC 6570](related-rfcs/6570.md) | Javasolt szabvány | URI-sablonok |

