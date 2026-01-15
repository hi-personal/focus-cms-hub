# Focus CMS – Technikai áttekintés

## Projektirány és nyitottság

A Focus CMS egy működő, valós használatban lévő rendszer.
Mivel egyre többen értékelik a letisztult és egyszerű megoldásokat, felmerült bennem annak a lehetősége, hogy a projekt később nyitott formában is elérhetővé váljon.

Az elképzelés szerint a kód **MIT licenc** alatt kerülhetne publikálásra.
Amennyiben kialakul egy kisebb fejlesztői kör, a Focus CMS egy mások számára is hasznos, örömteli alternatívává válhatna.

Ezt megelőzően azonban szükséges:
- a kódbázis átvizsgálása,
- annak eldöntése, hogy alkalmas-e közös alapnak,
- és hogy nem tartalmaz-e olyan strukturális vagy logikai hibákat,
  amelyek eleve kizárnák az érdemi együttműködést.

A nyitás előtt különösen értékes segítség lenne az alábbi területeken:
- automatizált tesztek írása,
- biztonsági auditálás,
- hosszabb távú kódoptimalizálás.

---

## Architektúra – alapelvek

A Focus CMS **monolitikus felépítésű**, de **moduláris szemléletű** rendszer.

Minden funkció közvetlenül a rendszer részeként van megvalósítva.
A cél nem egy „plugin-halmozó” megközelítés (WordPress-szerű svájcibicska), hanem egy letisztult, integrált megoldás.

A filozófia inkább egy *Apple-szerű* irányt követ:
- kevesebb funkció,
- de stabil,
- átgondolt,
- hosszú távon fenntartható működés.

---

## Sablonarchitektúra és bővíthetőség

A sablonok nem pusztán megjelenítési rétegek.

### Telepítés és struktúra
- A sablonok Composerrel telepíthetők
- Egyedi modul-telepítő csomagon keresztül
- Saját mappába kerülnek, elkülönülve az alkalmazás core-jától

A sablonfejlesztés **teljesen független** az alkalmazás core kódjától.

---

### Service Provider alapú működés

Minden sablon saját Service Providerrel rendelkezik
(pl. `Themes\FocusDefaultTheme\Providers` namespace alatt).

Ennek köszönhetően a sablon:
- nem csak Blade nézeteket tartalmaz,
- hanem dedikált PHP osztályokat is,
- amelyek a layoutok, komponensek és kapcsolódó logika kezeléséért felelnek.

```php
\Themes\FocusDefaultTheme\Classes\Layouts\Components\PublicDefault::class => 'public-default',
\Themes\FocusDefaultTheme\Classes\Layouts\Components\MaintenanceDefault::class => 'maintenance-default',
```

Ezáltal:
- a layoutok Blade komponensként használhatók (`<x-public-default>`)
- minden layout mögött **opcionálisan**, egy konkrét PHP osztály áll
- a nézetkezelés strukturált, bővíthető és jól követhető
- a sablon nemcsak megjelenítést, hanem logikai réteget is hordoz
- a sablon lokálisan fejleszthető, majd saját GIT repóból telepíthető composerrel

---

## Funkcionális bővítés sablon szinten

A saját Service Provider révén a sablon önállóan képes bővíteni az alkalmazás működését:

- saját route-ok definiálása
- saját kontrollerek létrehozása
- egyedi middleware-ek regisztrálása
- helper-ek és service bindingok hozzáadása

Mindez úgy történik, hogy a core kód módosítása **nem szükséges**.

Ebben a modellben a sablon (**opcionálisan!**) nem „skin”, hanem egy **funkcionális modul**, saját felelősségi körrel.

---

## Független sablonfejlesztés (külön GIT + symlink)

A sablonok külön Git repositoryban fejleszthetők:
- önálló verziókezeléssel
- saját kiadási ciklussal
- az alkalmazás kódbázisától függetlenül

Lokális fejlesztés során a sablon symlinkelve van az alkalmazás megfelelő mappájába, így:
- azonnali visszajelzés érhető el módosításkor
- nincs szükség folyamatos újratelepítésre
- a lokális és éles működés megegyezik

---

## Admin és frontend teljes szétválasztása

A frontend sablon teljes mértékben független az admin felülettől:

- külön nézetstruktúra
- külön CSS és JavaScript állományok
- külön Vite / NPM build folyamat
- nincs közös asset vagy UI függőség

Az admin és a frontend:
- nem osztozik nézeteken
- nem osztozik stílusokon
- nem osztozik JavaScript logikán

Ez biztosítja, hogy az admin stabil és kiszámítható maradjon,
miközben a frontend szabadon kísérletezhet UI/UX szinten,
és elkerülhető minden „keresztszennyezés”.

---

## Build rendszer és eszközök

- Tailwind CSS alapú admin felület
- Frontend sablonok szintén Tailwindre épülnek
- Alpine.js használata
- jQuery kompatibilisen integrálva
- NPM-mel modulárisan telepített csomagok

### Build folyamatok
- `npm build --all` → alkalmazás + sablonok
- `npm build app` → csak az alkalmazás
- sablonok cseréje NPM + Vite konfiguráción keresztül

---

## Funkciók (nem teljes lista)

- Webes admin felület
- CLI parancsok (pl. sablonváltás, karbantartás)
- Brevo (email küldés)
- Google reCAPTCHA + Cloudflare védelem
- Kétfaktoros hitelesítés (email, Authy kompatibilis)
- Taxonómiák:
  - kategóriák (hierarchikus)
  - címkék (nem hierarchikus)
  - konfigurálható bővíthetőség
- Oldalak és bejegyzések
- Widget rendszer:
  - sablonban definiált widget helyek
  - adminból shortcode-dal beilleszthető
  - tartalomba ágyazható, teljes értékű postként
- Markdown alapú tartalomkezelés
- Saját kép- és fájlkezelő:
  - mobiloptimalizált
  - albumon belüli sorrendezés
- Kódrészletek szintaxiskiemeléssel
- Belépési értesítések emailben
- Jelenleg jellemzően egyfelhasználós használatban van (nálam)
- Az architektúra többfelhasználós működésre is fel van készítve
- Adatbázis többfelhasználós működésre tervezve
- CLI-ből több user létrehozható

---

## Fejlesztési és üzemeltetési modell

- Lokális fejlesztés localhoston
- Docker alapú virtualizáció
- Apache + PHP
- Node.js
- MariaDB

Az Apache + PHP és a Node.js konténerek a konténereken kívüli fájlrendszerre mutatnak,
így a fejlesztés (pl. VS Code) közvetlenül a host rendszeren történik.

A konténerek közös hálózaton futnak.

### Tipikus parancsok

Apache konténer:
- `php artisan serve`
- `composer update`

Node konténer:
- `npm update`
- `npm run build`

Host:
- Git műveletek

### Telepítés és frissítés

- privát Git repository
- szerveren SSH-n keresztül:
  - `git pull`
  - `composer update`
  - `php artisan migrate`
  - `php artisan optimize:clear`
  - `php artisan optimize`
