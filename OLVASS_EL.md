# STRNG Fitness – tagsági app, v12

Ez a csomag az új app forrását, a kész helyi változatot és az opcionális Supabase-hátteret tartalmazza. A GitHubon jelenleg futó oldalt önmagában a ZIP letöltése nem módosítja.

## Mi működik a csomagban?

- Heti naptár, naponta több óra és óratípus, edzésterv/WOD, edzői alternatívák.
- Kiemelt, lejáró közlemények; az aznapi lemondott óra automatikus piros figyelmeztetést kap.
- Órák és közlemények piszkozata, magyar idő szerinti közzététel; a korábbi közzétett változat az időzítésig elérhető.
- Hetek másolása, mentett órasablonok, vendég-előnézet, opcionális meglévő foglalási link.
- Tervezett részvétel (nem foglalás), változások olvasási visszaigazolása, edzés utáni visszajelzés, alternatíva naplózása.
- Megmaradó 5/3/1 AMRAP-rögzítés és becsült 1RM-grafikon, előző eredmény, eredményjavítás.
- Hat hét, heti két futás. A korábbi terv intervallumos és könnyű napjainak átdolgozása, nem hivatalos Hinshaw-program.
- Külön futás rögzítése dátum/táv/idő/megjegyzés mezőkkel, javítás, tempó és tényleges heti össztáv.
- Mentett eredmények visszaállítható törlése, szigorúan ellenőrzött import, fájlexport, offline alkalmazáskeret.

## Felhővel válik közössé

A `config.json` szándékosan üres. Ebben a változatban a mentés HELYI: az admin bárki által kipróbálható, és az itt közzétett terv nem jut el más készülékre. Így ne add ki bérletesek éles appjaként.

A felhő bekötésével működik:

- e-mailes egyszer használatos kód, személyes fiók;
- valódi, szerveren ellenőrzött admin és aktív bérletes hozzáférés;
- közös edzésterv, időzített publikálás és vendégenként elkülönített napló;
- offline mentés után szinkronizálás, verzióütközés jelzése;
- vendéglista, aktivitás, főgyakorlat-eredmények, jelentkezők, visszajelzések és olvasottság.

Az értesítések az appban jelennek meg; telefonra küldött push és automatikus tájékoztató e-mail nincs bekötve. A külön futás nem teljesít programozott edzést. Az órára készülés nem bizonyít jelenlétet; a vendég saját részvétel-visszajelzése számít az összesítőben.

## Egyszeri indítás – Geri számára

1. Hozz létre egy saját Supabase-projektet. A szolgáltató fiókja és esetleges díja a tiéd; ebben a munkában nem jött létre előfizetés.
2. A projekt SQL Editor felületén másold be és futtasd a `backend/schema.sql` tartalmát. A fájl végén lévő adminmintába írd be a SAJÁT belépési e-mail-címedet, és azt a két sort külön futtasd le (a `--` megjegyzésjel nélkül). Csak itt adható adminjog.
3. Authentication / Email Templates: a Magic Link levélhez használd a `backend/login-email.html` mintát. A `{{ .Token }}` helyén érkezik a kód. Ezért nem kell jelszó-visszaállítás.
4. Állíts be saját SMTP-levélküldőt a Supabase-ben, hitelesített feladóval. Az alap küldő nem általános vendégküldésre való. A feladó domainjéhez a szolgáltató által adott DNS-rekordokat kell felvenni. A küldő titkos kulcsa csak a szolgáltatói beállításokba kerülhet.
5. GitHub repository / Settings / Secrets and variables / Actions / Variables: add meg a `STRNG_SUPABASE_URL` és a `STRNG_SUPABASE_ANON_KEY` értéket. Utóbbi a NYILVÁNOS publishable/anon kulcs, soha nem service_role vagy secret kulcs.
6. Actions / Build cloud-ready app / Run workflow. A végén letölthető a `strng-cloud-ready` csomag. Ennek fájljai kerüljenek az app GitHub Pages által kiszolgált mappájába. Ez a munkafolyamat önmagában NEM publikál, nem változtatja meg a jelenlegi tárhelybeállítást.
7. Lépj be a saját e-mailes kódoddal. Admin / Bérletes hozzáférés: név, e-mail és bérlet utolsó napja. A vendég ugyanazzal az e-maillel lépjen be.
8. Saját próba vendégfiókkal ellenőrizd a kód megérkezését, a jogosultságokat, egy edzés mentését és egy másik eszközön visszatöltését. Ezek éles szolgáltatói próbája ebben a munkában még nem történt meg.

A kódkérés új e-mail-címhez is létrehozhat hitelesítési fiókot, de bérletes jogosultságot soha nem ad automatikusan. A projektben az e-mail-provider legyen engedélyezve, a küldési limiteket a vendégszámhoz állítsd. A belépőkód csak a kért e-mail-címhez használható.

A régi névtelen naplók nem kerülnek automatikusan az új fiókba. Beállítások / Korábbi, névtelen napló átvétele: a vendég külön megerősíti, hogy az övé az adott telefonon lévő napló. A régi helyi példány megmarad.

## Körlevél

A legegyszerűbb javasolt felállás: Supabase + saját SMTP a belépési kódokhoz; a meglévő Systeme.io a körlevelekhez.

Admin / Körlevél előkészítése: megírható és letölthető a szöveg. A felhős vendéglista frissítése után CSV-be exportálhatók a hozzájárulást jelző, aktív tagok. A rendszer nem küld levelet és nem szinkronizál automatikusan a Systeme.io-val. A leiratkozásokat a levélküldőben kell tiszteletben tartani; importnál nem szabad újra feliratkoztatni a korábban leiratkozottakat. Az appban visszavont hozzájárulást is vezesd át a listában. A bérlet nem jelent automatikus hírlevél-feliratkozást.

Systeme.io-ban külön STRNG Fitness címke/lista, hitelesített domain és feladó, tesztlevél, majd címkére szűrt Newsletter küldés vagy időzítés szükséges. Az üzemeltetési értesítéseket (pl. elmaradó óra) külön kezeld a marketingtől.

## Technikai működés és korlátok

- A forrás: `src/base.html`, `src/club.js`, `src/club.css`. `npm run build` újragenerálja az `index.html` fájlt és a CSP-hash-t. A kész index közvetlen szerkesztése a következő buildnél elveszne.
- `npm ci --ignore-scripts`, majd `npm test`: helyi UI/mentési tesztek, valós PostgreSQL-motoron futó adat-hozzáférési tesztek (PGlite, a Supabase auth azonosítóit utánzó tesztkörnyezet).
- Az adatbázistáblák közvetlen hozzáférése tiltott, az RPC-k hitelesített felhasználót és szerepkört ellenőriznek. Minden másik vendég naplója a vendég számára elérhetetlen.
- Minden felhős módosítás revíziót ellenőriz. Ütközéskor nincs csendes felülírás. A helyi napló exportálható, majd a felhőbeli betölthető; kézi összevetés szükséges lehet.
- A lejáró bérletet a szerver magyar idő szerint ellenőrzi. Korábban letöltött adatok offline másolata technikailag megmaradhat az eszközön; ezt távolról nem lehet biztosan visszavonni.
- A Supabase session a saját böngésző tárolójában van. Közös eszközön ki kell jelentkezni. A böngészőadatok törlése a még nem szinkronizált helyi naplót törölheti.
- Az edzői publikáláshoz internet kell. A helyi piszkozat módú szerkesztő adatait exportálhatod, majd a bekötött adminban importálhatod.
- Automatikus fizetés/bérletrendszer-integráció, push-értesítés és levélküldő-szinkron nincs ebben a verzióban. A bérleteket az adminban kézzel vezeted.
- A csomag nem tartalmazza az elvetett Mayhem home átdolgozásokat, valódi vendégadatokat vagy tesztórákat.
- Éles üzem előtt a saját adatkezelési tájékoztatódat igazítsd a tényleges szolgáltatókhoz és a vendég–edző adatmegosztáshoz.

## Hivatalos útmutatók

- https://supabase.com/docs/guides/auth/auth-email-passwordless
- https://supabase.com/docs/guides/auth/auth-smtp
- https://supabase.com/docs/guides/database/functions
- https://help.systeme.io/article/358-how-to-send-or-schedule-a-newsletter
- https://help.systeme.io/article/316-how-to-authenticate-your-personal-domain-name
