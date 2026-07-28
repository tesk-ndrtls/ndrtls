# CLAUDE.md — Neanderthals HC (ndrtls.com)

Projektový kontext pre Claude Code. Načíta sa automaticky na začiatku každej session.
Píš so mnou po slovensky.

## Čo to je

Web hokejového klubu **Bratislava Neanderthals**, `ndrtls.com`. Jeden veľký
**single-file HTML** súbor (`index.html`, 5000+ riadkov) — HTML, CSS aj JS
pohromade. Nie je tu build step, framework ani bundler. Edituje sa priamo
`index.html`.

Aktuálna verzia je v badge v pätičke (`<div id="version-badge">vX.Y.Z</div>`).
**Po každej funkčnej zmene ju bumpni** (patch pre malé úpravy, minor pre nové
funkcie) — na to som v chate opakovane zabúdal, preto to sem píšem výslovne.

## Stack

- **Frontend:** čisté HTML/CSS/JS v jednom súbore. Žiadny build.
- **Auth + DB:** Supabase (projekt `vchxjbdrromujmzujduo`), klient `sb`.
- **Hosting:** GitHub Pages.
- **DNS:** Cloudflare.
- **Proxy / server-side:** PHP na WebSupport.sk shared hostingu, doména
  `trenuj.ndrtls.com` (napr. `rahl-proxy.php`, `match-detail-proxy.php`,
  kontaktná notifikácia). PHP + MySQL tam beží mimo tohto repa.

## Ako pracujem (konvencie)

- **Menší je lepší.** Rob cielené zmeny, nie prepisy. Pri úprave jednej veci
  nechaj zvyšok tak.
- **Prosím over syntax** po väčšej zmene (napr. rýchla kontrola, či sa JS dá
  sparsovať) — v chate sme to robili a chytilo to chyby.
- **Slovenské UI.** Všetky texty pre používateľa sú po slovensky, vrátane
  toastov a hlášok.
- **Admin vs. hráč.** Editačné a admin prvky sa zobrazujú len keď
  `currentProfile?.role === 'admin'`. Vždy to skontroluj pri novom UI.
- **Toast na spätnú väzbu:** `toast('Správa ✓','success')` /
  `toast('Chyba…','error')`.
- **Bezpečnosť dát:** používateľský vstup escapuj pred vložením do DOM
  (v kóde je `mdEsc()` a inline `.replace(/</g,'&lt;')` vzory).
- **Supabase zápis:** vzor `sb.from('tabuľka').upsert({...},{onConflict:'stĺpec'})`
  alebo `.update({...}).eq('id',id).select()`. Po write over `data`/`error`.
  Ak meníš schému (nový stĺpec), **napíš mi SQL `alter table`** — migrácie
  v Supabase si spúšťam sám.
- **RLS:** pri nových zápisoch over, že RLS politika dovolí danú rolu
  (časté: admin má `select`/`delete`, ale chýba `update`).

## Kľúčové časti index.html

- **Match detail modal** (`openMatchDetail`, `renderMatchDetail`) — detail
  zápasu: skóre, video (YouTube), timeline gólov/trestov, box score.
  Dáta z `match-detail-proxy.php` (szalh základ) + admin overrides z tabuľky
  `match_details`.
- **Vrstvenie dát v timeline:** (1) szalh proxy → (2) auto-override zo súpisky
  (spárovanie mena s profilom) → (3) admin override z `match_details.overrides`
  (vždy vyhráva). Kľúč akcie: `period_time_type` (napr. `2_31:53_goal`).
- **Admin edit akcie:** ✎ tlačidlo na každej akcii v timeline → inline editor
  (video-čas, číslo, strelec, asistencie). YouTube ID sa edituje pod videom.
  Ukladá `mdPersist()` — mení len dané pole, zvyšok necháva.
- **Prihlášky (admin):** `loadAdminContacts`, tabuľka `contact_requests`.
  Má internú manažérsku poznámku (`admin_note`, `admin_note_by`,
  `admin_note_at`) — `saveContactNote()`.
- **Súpiska / lineup / RSVP:** pracuje s registrovanými profilmi (`profiles`),
  RSVP v tabuľke `rsvp`, výsledky v `game_results`.
- **Pull-to-refresh + scroll-lock:** `initPullToRefresh` (blokovaný pri
  otvorenom menu/modale) a `initScrollLock` (zamkne pozadie keď je otvorená
  vrstva).

## Príbuzné appky (rovnaký ekosystém, iné repá)

Zdieľajú ten istý Supabase projekt a `ndrtls.com` doménu:
- `skuska.ndrtls.com` — Kvašung, appka na skúšky kapely.
- `kvizek.ndrtls.com` — Kvízek, pub-kvíz appka.
- `ourjourney.ndrtls.com` — Naša Cesta, cestovateľské spomienky.

## Git

Používaj git rozumne: pozri `git diff` pred commitom, píš stručné commit
správy (kľudne po slovensky), commitni logické celky. Po zmene v `index.html`
a bumpe verzie je commit + push na GitHub Pages celý deploy.
