# A rakoncátlan időpontfoglaló – hibariport

Repó: https://github.com/And-8-And/umfst-bh02-idopontfoglalo
Készítette: Zsigmond András-Andor, UMFST - ILM2024

## 1. hiba
- **Hiba leírása és reprodukálása:** Ugyanarra az időpontra többször is lehet foglalni, a „már foglalt” hibaüzenet nem jelenik meg. Reprodukálás: két foglalás leadása "09:00" időponttal.
- **Gyökérok (sor + magyarázat):** `app.js`: `return foglalasok.indexOf(ido) !== -1;`. A `foglalasok` tömb objektumokat tartalmaz, az `ido` viszont egy string. Az `indexOf` nem tud objektumokban lévő tulajdonságokra keresni, így mindig `-1`-et (nincs találat) ad vissza.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh02-idopontfoglalo/commit/effc9326cb9deccc03797e010c333a29101f9133 (Kicseréltem az `indexOf`-ot a `some` tömbmetódusra).
- **Hogyan tesztelted (kézi tesztlépések, konzolos ellenőrzés vagy egységteszt):** Kézi teszteléssel: felvittem egy foglalást "09:00"-ra, majd megpróbáltam újra. A felületen megjelent az "Ez az időpont már foglalt." hibaüzenet, és a lista nem bővült új elemmel.
- **Tanulság / érintett alapelv:** Tömbmetódusok megfelelő használata. Objektumokból álló tömbben nem használhatunk egyszerű primitív értéket váró `indexOf`-ot.

## 2. hiba
- **Hiba leírása és reprodukálása:** A napi limit 8, de 9 beteget is be lehet rögzíteni. A 10. betegnél jön csak a hibaüzenet.
- **Gyökérok (sor + magyarázat):** `app.js`: `if (foglalasok.length > MAX_NAPI)`. Ez egy tipikus "off-by-one" hiba. Amikor a 8. beteg bent van, a hossz 8. A 9. hozzáadásakor a feltétel `8 > 8` ami `false`, így a 9. elem is a tömbbe kerül.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh02-idopontfoglalo/commit/cb828842e9fe83c2e234669d3fb811a72326800e (A `>` operátort `>=`-re cseréltem).
- **Hogyan tesztelted (kézi tesztlépések, konzolos ellenőrzés vagy egységteszt):** Kézi teszteléssel létrehoztam 8 foglalást. A 9. leadásakor ellenőriztem, hogy megjelenik-e a "Ma már nincs szabad hely." üzenet, és a foglalás nem kerül a listába.
- **Tanulság / érintett alapelv:** Határérték-ellenőrzés (boundary testing) fontossága és a logikai operátorok pontos alkalmazása.

## 3. hiba
- **Hiba leírása és reprodukálása:** A lista nem rendeződik időrendbe, a későbbi időpontok is bekerülhetnek korábbiak elé.
- **Gyökérok (sor + magyarázat):** `app.js`: `foglalasok.sort(function (a, b) { return a.ido > b.ido; });`. A `sort` egy -1, 0, vagy 1 (negatív, nulla, pozitív szám) típusú visszatérési értéket vár. A `>` boolean-t (true/false) ad vissza, ami kiszámíthatatlan rendezést eredményez böngészőtől függően.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh02-idopontfoglalo/commit/3e1771ed87e30ff8e0b9666a1e6b79cd1fb1441b (A stringek összehasonlítását `a.ido.localeCompare(b.ido)` kifejezésre cseréltem).
- **Hogyan tesztelted (kézi tesztlépések, konzolos ellenőrzés vagy egységteszt):** Hozzáadtam egy "10:00" majd egy "08:30" időpontot. Frissítés és renderelés után az UI-on és a localStorage-ban is a "08:30" volt elöl.
- **Tanulság / érintett alapelv:** A beépített API-k és paramétereik specifikációjának pontos ismerete (MDN dokumentáció olvasása).

## 4. hiba
- **Hiba leírása és reprodukálása:** XSS hiba. Ha a beteg neve HTML kód (pl. az `<img...>` példa), az beépül a DOM-ba és lefut, így megváltoztatva az oldal kinézetét.
- **Gyökérok (sor + magyarázat):** `app.js` (24. sor környéke): `+ f.nev +`. A felhasználó által megadott nyers bemenet (név) kódolás (escaping) nélkül, direktben kerül az `innerHTML`-be, így a böngésző futtatható HTML/JS kódként értelmezi.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh02-idopontfoglalo/commit/0d0da95ca06f105027045f6730ce7269b01b01fd (Beszúrás előtt egy `.replace()` lánccal ártalmatlanítottam a `<` és `>` karaktereket, HTML entitásokká alakítva őket).
- **Hogyan tesztelted (kézi tesztlépések, konzolos ellenőrzés vagy egységteszt):** Beírtam a megadott `<img src=x onerror="document.body.style.background='red'">` tesztkódot betegnévként. A háttér nem lett piros, a lista elemében pedig pontosan a kód szövege jelent meg.
- **Tanulság / érintett alapelv:** Biztonság (Security) / Bemeneti adatok validálása és tisztítása. Sose bízzunk meg a felhasználói adatban (Never trust user input).

## 5. hiba
- **Hiba leírása és reprodukálása:** A "Törlés" gombot nem lehet a billentyűzet Tab gombjával kijelölni és Enterrel aktiválni.
- **Gyökérok (sor + magyarázat):** `app.js` (25. sor): `<span class="torol" ...>`. A `<span>` egy inline szöveges elem, ami szemantikailag nem interaktív. Nem kap automatikusan fókuszt, és az Enter lenyomása nem generál `click` eseményt rajta.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh02-idopontfoglalo/commit/61b60b4bdb01a7c33e46d8db97b9387106a707e3 (A `<span>` elemet átírtam `<button type="button">` elemre az `app.js`-ben).
- **Hogyan tesztelted (kézi tesztlépések, konzolos ellenőrzés vagy egységteszt):** Az egér használata nélkül, kizárólag a Tab billentyűvel végiglépkedtem az oldalon. A fókusz megállt a Törlés gombon, majd az Enter lenyomására sikeresen törlődött az elem.
- **Tanulság / érintett alapelv:** Webes akadálymentesítés (Accessibility / a11y) és a szemantikus HTML alapelvek betartása (az interakciókhoz gombot vagy linket használjunk).

## 6. hiba
- **Hiba leírása és reprodukálása:** Sok foglalás esetén (50+) a renderelés jelentősen belassul, hosszú feladat (long task) jelenik meg a Performance profilozóban.
- **Gyökérok (sor + magyarázat):** `app.js` (24. sor): `lista.innerHTML += ...`. A ciklus minden egyes lépésénél módosítjuk a DOM-ot. Az `innerHTML` olvasása és írása ciklusban állandó újra-renderelésre (layout thrashing) kényszeríti a böngészőt.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh02-idopontfoglalo/commit/aa0ac3f0ac33a5f41ef24741ca10f1cc3f461df6 (A generált HTML stringeket egy memóriában lévő változóban fűztem össze a cikluson belül, és a cikluson kívül csak egyszer állítottam be a `lista.innerHTML`-t).
- **Hogyan tesztelted (kézi tesztlépések, konzolos ellenőrzés vagy egységteszt):** Egy kis szkripttel beletettem 100 foglalást a localStorage-ba, majd a böngésző devtools Performance fülén mértem az oldalbetöltést. A "render" függvény futási ideje töredékére csökkent.
- **Tanulság / érintett alapelv:** Teljesítményoptimalizálás (Performance). A DOM manipuláció drága művelet, minimalizálni kell az írások számát, különösen ciklusokon belül.

## Összegzés
- Mit tanultam: A kód működése mögötti apró buktatókat, mint az `indexOf` korlátait referenciatípusoknál, a boundary-check (határérték) pontosságát, a biztonságos DOM manipulációt és az akadálymentesítés alapjait.
- Melyik alapelv sérült a leggyakrabban: A programozási jó gyakorlatok (Best Practices) és az adatkezelési szabályok megsértése – a típusok nem megfelelő ismerete és a böngésző (DOM) költséges operációinak ciklusba szervezése volt a legtöbb probléma okozója.