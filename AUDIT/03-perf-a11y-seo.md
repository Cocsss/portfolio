# AUDIT / 03 — Performance, A11y & SEO

> Read-only. Estimations argumentées (pas de Lighthouse réel lancé). Mobile first (390 px, 4G lente ≈ 1.6 Mbps down, RTT 150 ms, iPhone SE 2020 CPU throttle 4×).
> Cibles Core Web Vitals (2026) : **LCP < 2,5 s**, **CLS < 0,1**, **INP < 200 ms**.

---

## Executive Summary

Le site est techniquement propre pour un single-file vanilla (pas de framework, pas de dépendance JS, landmarks HTML5 en place, `prefers-reduced-motion` respecté, `:focus-visible` défini). **Trois gros écueils** plombent les CWV et le SEO : (1) le portrait PNG de **2,37 Mo** chargé en `<img>` simple, sans AVIF, sans `srcset`, sans `loading="lazy"` — il n'est pas LCP mais pèse dans le bundle mobile ; (2) le **showreel vidéo en autoplay** sur le hero devient le LCP probable et tire le LCP vers 3-4 s sur 4G (aucun poster, `preload="metadata"` correct mais le `<video>` est toujours le candidat LCP) ; (3) **aucun balisage schema.org**, pas de `sitemap.xml`, pas de `robots.txt`, pas de `canonical`, pas d'`og:image` — le site est SEO-aveugle alors que le contenu est qualifié géographiquement (Bruxelles, Premiere Pro). Secondaires : le **film-loader bloque visuellement 3,4 s avant le hero** (impact perçu fort même si le LCP du DOM est déjà peint dessous), **contraste `--accent` sur `--bg` en light à 3,4:1** (fail AA body mais pass AA large et UI), et le **`<h1>` est vide sans JS** (chars injectés en JS, seul l'`aria-label` sauve la sémantique). Verdict : **P0 corrigeables en 1-2 jours, gain estimé 1,5-2,5 s sur LCP mobile + Lighthouse SEO 60 → 90+**.

---

## Poids estimé du site

Le bundle *first load* servi à un visiteur mobile neuf, sans cache, inclut :

| Ressource | Taille | Charge | Note |
|---|---|---|---|
| `index.html` (HTML + CSS inline + JS inline, non-minifié, non-gzippé) | **155 Ko** brut / **~35 Ko** gzippé | Bloquante (parser HTML) | CSS inline ≈ 60 Ko brut, JS inline ≈ 18 Ko brut, HTML utile ≈ 77 Ko |
| `assets/portrait.png` 1400×2100 | **2 375 Ko** | Non-bloquante mais **eager** (pas de `loading="lazy"`) | Section §05 hors fold — le preloader scanner HTML le tire quand même car `<img>` simple |
| Google Fonts CSS (`css2?...`) | ~2,5 Ko gzippé | Bloquante (render-blocking `<link rel="stylesheet">`) | Déclare @font-face avec `unicode-range` latin + latin-ext + vietnamese |
| Dela Gothic One `woff2` (1 poids, latin) | ~24 Ko | Critical (h1, h2, h3) | 1 fichier latin (display) |
| Inter `woff2` × 4 poids × latin | ~4 × 18 = **72 Ko** | Critical (body) | 300/400/500/600 séparés, `display=swap` fourni par Google |
| Inter latin-ext (fallback pour é/à/…) | ~4 × 22 = **88 Ko** | Critical (FR) | Google sert latin et latin-ext comme @font-face distincts — les accents français déclenchent le téléchargement |
| JetBrains Mono `woff2` × 2 poids × 2 subsets | ~4 × 22 = **88 Ko** | Eager (eyebrow, idx, meta) | 400/500 + latin/latin-ext |
| `email-decode.min.js` (Cloudflare) | ~1,5 Ko gzippé | Fin de body, non-critique | Charge depuis `/cdn-cgi/scripts/5c5dd728/...` même origine |
| Showreel 2026 (R2) premier segment | ~80-200 Ko metadata (HTTP 206) | `preload="metadata"` déclenche téléchargement du moov | Le reste arrive au play |
| 10 vidéos carousels `preload="metadata"` | 10 × ~30 Ko = ~300 Ko | Parallèle | `preload="metadata"` × 10 = 10 HTTP requests moov box simultanés |
| Favicon | manquant (404) | — | Aucun `<link rel="icon">` déclaré |

**Total bundle critique mobile (LCP)** : ~35 (HTML gz) + 2 375 (portrait) + ~200 (fonts latin only si JS fail, le preloader ne lazy-load pas) + ~150 (showreel metadata) = **~2,7 Mo avant interactivité**.

Sur 4G lente (1,6 Mbps ≈ 200 Ko/s download), portrait seul = **~12 s de bande passante**. Il ne bloque pas le rendu mais consomme les fenêtres TCP du showreel et des fonts.

---

## Issues numérotées par axe

### Performance

#### PERF-01 [P0] — Portrait PNG 2,37 Mo, sans format moderne ni lazy
- **Observation** : `index.html:2956` → `<img src="assets/portrait.png" alt="" />`. PNG 2486723 octets, 1400×2100 probable (asset découpé §119016c). Pas de `<picture>`, pas de `srcset`, pas de `loading="lazy"`, pas de `decoding="async"`. La section `#apropos` est hors-fold §05, donc éligible au *lazy loading* natif.
- **Impact estimé CWV** : sur 4G lente, 2,37 Mo = ~12 s de bande passante concurrente au LCP. Sans être le LCP lui-même, il vole du bandwidth aux assets critiques (showreel, fonts latin-ext). Gain Lighthouse perf : **+10 à +18 points**. Économie bundle : **≈ 2,2 Mo** sur mobile après conversion AVIF @ q60.
- **Fix concret** :
  ```bash
  # Générer un AVIF carré + un JPEG fallback + un 1x / 2x
  cwebp -q 78 portrait.png -o portrait.webp       # ~180 Ko
  avifenc --min 24 --max 30 -s 6 portrait.png portrait.avif  # ~90 Ko
  # ou, plus simple, un seul outil Sharp :
  npx sharp-cli -i portrait.png -o portrait-1x.avif --avif quality=55
  npx sharp-cli -i portrait.png -o portrait-2x.avif --avif quality=55 resize=1400
  ```
  HTML cible :
  ```html
  <picture>
    <source type="image/avif"
            srcset="assets/portrait-700.avif 700w, assets/portrait-1400.avif 1400w"
            sizes="(max-width: 900px) 90vw, 560px">
    <source type="image/webp"
            srcset="assets/portrait-700.webp 700w, assets/portrait-1400.webp 1400w"
            sizes="(max-width: 900px) 90vw, 560px">
    <img src="assets/portrait-1400.jpg" alt="" width="1400" height="2100"
         loading="lazy" decoding="async">
  </picture>
  ```
  Noter `width`/`height` pour éviter CLS au moment où l'image paint.
- **Effort** : S (1h — export des variantes + remplacement 1 `<img>`).

#### PERF-02 [P0] — Showreel vidéo `<video autoplay>` sans poster, candidat LCP mobile
- **Observation** : `index.html:2608-2611`. `<video class="hero-reel-video" src="...showreel-2026.mp4" autoplay muted loop playsinline preload="metadata">`. Aucun `poster=""` — donc pendant les ~800-1500 ms qui séparent `metadata` loaded du premier frame rendu, le conteneur montre le dégradé CSS `linear-gradient(135deg, #0a0a09 0%, #1a1816 40%, #252220 70%, #0f0e0c 100%)` (`index.html:490`). Sur mobile en 21:9, le reel est un grand rectangle ~390×167 = **65 000 px²**, candidat LCP potentiel si la `.hero-name` char-by-char anime ses spans avec `translateY(110%)` + opacity 0 (`index.html:458-462`) — les lettres ne sont visibles qu'après `3,4 s + animation 1,1 s = ~4,5 s`. Donc **le LCP est soit le dégradé du `.hero-reel`, soit le first frame du showreel, soit le `.hero-eyebrow` mono**, pas le h1.
- **Impact estimé CWV** : LCP sur 4G + CPU 4× ≈ **3,2-4,1 s** (acceptable mais dégradé). Sans poster, il y a un flash noir → dégradé → vidéo qui pénalise la perception. Avec poster AVIF pré-généré (1 frame à 2 s de timecode, ~35 Ko), le LCP candidat devient l'image poster elle-même, paintable en ≤ 1,8 s.
- **Fix concret** :
  ```bash
  # Extraire la frame 2s du showreel en AVIF
  ffmpeg -ss 2 -i showreel-2026.mp4 -frames:v 1 -vf scale=1440:-1 showreel-poster.jpg
  avifenc --min 28 --max 34 -s 6 showreel-poster.jpg showreel-poster.avif  # ~30-50 Ko
  ```
  HTML :
  ```html
  <video class="hero-reel-video"
         src="https://videos.corentindeville.com/showreel-2026.mp4"
         poster="https://videos.corentindeville.com/showreel-poster.avif"
         autoplay muted loop playsinline preload="metadata"
         width="1440" height="617"
         fetchpriority="high">
  </video>
  ```
  Ajouter `fetchpriority="high"` n'envoie pas de Priority Hint sur `<video>` lui-même (géré par browser heuristic), mais le `poster=""` va chez `<img>` qui respecte le hint. Upload le poster sur R2, même bucket.
- **Effort** : S (30 min — ffmpeg + avifenc + upload R2 + 1 attribut HTML).

#### PERF-03 [P1] — 10 vidéos `<video preload="metadata">` en parallèle dans les carousels projets
- **Observation** : `index.html:2670, 2678, 2686, 2719, 2727, 2735, 2768, 2776, 2784, 2815`. 4 carousels × ~3 clips = 10 `<video>` tous avec `autoplay muted loop preload="metadata"`. Même hors-fold, le navigateur télécharge les *moov box* des 10 (≈ 30-80 Ko chacun = **300-800 Ko gaspillés** avant que l'utilisateur scrolle). Sur iPhone bas de gamme, **10 décodeurs vidéo actifs simultanés = RAM saturée** + `autoplay` qui se déclenche dès entrée viewport → saccades INP.
- **Impact estimé CWV** : INP +50 à +150 ms pendant scroll §01 Projets (décodage hardware concurrent). Bundle +300-800 Ko.
- **Fix concret** : passer à `preload="none"` sur les vidéos des carousels non-actifs, et utiliser IntersectionObserver pour `load()`/`play()` au premier scroll vers la section. Pattern :
  ```html
  <video src="..." preload="none" muted loop playsinline
         data-autoplay-on-visible></video>
  ```
  ```js
  const vio = new IntersectionObserver(entries => {
    entries.forEach(e => {
      const v = e.target;
      if (e.isIntersecting) { v.play().catch(()=>{}); } else { v.pause(); }
    });
  }, { threshold: 0.25 });
  document.querySelectorAll('video[data-autoplay-on-visible]').forEach(v => vio.observe(v));
  ```
  Garder `autoplay` uniquement sur le showreel hero.
- **Effort** : M (1-2h — retrait de tous les `autoplay` + 1 IntersectionObserver + `preload="none"`).

#### PERF-04 [P1] — Google Fonts en mode CSS externe `&display=swap`
- **Observation** : `index.html:17`. 3 familles × N poids + latin + latin-ext (car FR à accents) = **~10-12 requêtes woff2 séparées**, ~270 Ko total. Avec `preconnect` mais sans `<link rel="preload">` sur les fichiers `.woff2` critiques (Dela Gothic et Inter 400/500). Résultat : FOUT de ~500 ms (fallback `Impact`/`Arial`) pendant que les display s'installent.
- **Impact estimé CWV** : CLS ~0,02-0,05 (Dela Gothic a des métriques très différentes de Impact → reflow h1). LCP +200-500 ms sur le texte. Gain après self-host : **-150 à -400 ms LCP**.
- **Fix concret** (2 options) :
  - **Option A (minimale)** : ajouter `preload` sur Dela Gothic latin + Inter 400 latin :
    ```html
    <link rel="preload" as="font" type="font/woff2" crossorigin
      href="https://fonts.gstatic.com/s/delagothicone/v15/...latin.woff2">
    <link rel="preload" as="font" type="font/woff2" crossorigin
      href="https://fonts.gstatic.com/s/inter/v18/...latin.woff2">
    ```
    URL exactes à extraire depuis le CSS Google (ouvrir l'URL ligne 17, copier les `src:url(...)` latin).
  - **Option B (meilleure)** : self-host avec `google-webfonts-helper` (https://gwfh.mranftl.com/fonts), servir 3 fichiers woff2 latin + latin-ext depuis `/assets/fonts/`, CSS @font-face inline avec `font-display: swap` et `unicode-range`. Économie : 2 requêtes DNS (`fonts.googleapis.com`, `fonts.gstatic.com`), 1 CSS round-trip, ~20-40 Ko headers, **-300 ms LCP réaliste**.
- **Effort** : S pour option A (10 min), M pour option B (1h — télécharger + inliner @font-face).

#### PERF-05 [P1] — Film-loader 3,4 s bloque la perception du hero
- **Observation** : `index.html:210-305`. `.film-loader` `position:fixed; z-index:6000` avec `animation: filmReveal .65s ... 3.4s forwards`. Donc le contenu est masqué par un overlay noir pendant **3,4 s + 0,65 s de transition = 4 s** avant que le hero soit visible. Le JS `buildHero()` démarre les char animations à `delay = 3500 ms`. Même si le LCP technique mesure le paint sous l'overlay (le browser voit le DOM derrière), **l'utilisateur perçoit un splash noir de 4 s** — mauvais sur le ressenti, et potentiellement détecté comme LCP sur le `.film-loader` background si c'est la plus grande zone peinte (100vw × 100vh = souvent oui).
- **Impact estimé CWV** : LCP perçu +2 à +3 s. Si Chrome mesure l'overlay noir comme LCP (largest contentful = viewport entier en `#0a0a09`), alors **LCP = 3,4 s minimum** et le site échoue CWV d'office. Note : Chrome ignore souvent les éléments full-viewport single-color — à tester dans un vrai Lighthouse.
- **Fix concret** :
  - Réduire `3.4s` à `1.2-1.6s` (perception brief, pas un splash).
  - OU rendre le film-loader conditionnel : l'afficher uniquement au premier chargement (cookie / sessionStorage) pour éviter qu'il bloque les retours. Exemple :
    ```js
    if (!sessionStorage.getItem('loaderSeen')) {
      document.body.classList.add('first-visit');
      sessionStorage.setItem('loaderSeen','1');
    } else {
      document.getElementById('filmLoader').style.display = 'none';
    }
    ```
  - Respecter `prefers-reduced-motion` : déjà fait en CSS (ligne 2242-2250) mais **le film-loader n'est pas masqué** explicitement — ajouter :
    ```css
    @media (prefers-reduced-motion: reduce){
      .film-loader{display:none !important}
    }
    ```
- **Effort** : S (15 min).

#### PERF-06 [P1] — Scroll handlers multiples sans throttle, parallax showreel + scrollspy + scrollbar
- **Observation** :
  - `index.html:3225` — scrollspy `update()` appelé sur chaque `scroll` (passive OK, mais `update()` itère `sections.length = 6` + getBoundingClientRect via `offsetTop`).
  - `index.html:3311` — scroll progress bar `update()` chaque scroll.
  - `index.html:3468-3478` — parallax showreel avec rAF + ticking flag (OK, le seul correctement throttled).
  - `index.html:3225` scrollspy n'est PAS dans rAF. Il tourne à 60+ Hz sur mobile avec scroll inertiel.
- **Impact estimé CWV** : INP 60-120 ms pendant scrolls longs mobiles. Léger mais mesurable.
- **Fix concret** : envelopper scrollspy dans rAF + ticking flag, comme le parallax :
  ```js
  let nav_ticking = false;
  window.addEventListener('scroll', () => {
    if (nav_ticking) return;
    requestAnimationFrame(() => { update(); nav_ticking = false; });
    nav_ticking = true;
  }, {passive:true});
  ```
- **Effort** : S (5 min).

#### PERF-07 [P2] — CSS et JS inline non-minifiés
- **Observation** : `<style>` 2438 lignes / ~60 Ko + `<script>` ~18 Ko, tous deux en clair. Cloudflare Pages gzippe à la sortie (bon), mais on peut gagner ~20 % additionnels en minifiant avant commit.
- **Impact estimé CWV** : HTML total 155 Ko → ~120 Ko après minification CSS/JS → ~28 Ko gzippé au lieu de ~35 Ko. Gain LCP : ~50-100 ms sur 4G.
- **Fix concret** : build step optionnel (go à l'encontre du "no-build"). Alternative : outil one-shot manuel pré-commit :
  ```bash
  npx html-minifier-terser index.html -o index.min.html \
    --minify-css --minify-js --collapse-whitespace --remove-comments
  ```
  Mais conflit avec l'édition directe. À arbitrer — je **recommande de garder non-minifié** tant qu'il n'y a pas de CI.
- **Effort** : M si on introduit une étape (contre la contrainte no-build). **Skip par défaut.**

---

### Accessibilité

#### A11Y-01 [P0] — `<h1>` vide sans JavaScript
- **Observation** : `index.html:2585-2588`. Le `<h1 class="hero-name" aria-label="Corentin Devillé">` contient 2 `<span class="word" data-word="Corentin|Devillé">` **vides**, remplis en JS ligne 3093-3109 (`buildHero()`). Si JS fail/block (Noscript, CSP stricte, JS-désactivé, ou simplement erreur d'une ligne plus haut qui avorte le script), le h1 n'a **aucun texte** hormis l'astérisque SVG décoratif. L'`aria-label` sauve les screen readers mais **Google crawle `innerText` et indexera `""` comme h1 — SEO double peine**.
- **Impact estimé** : Lighthouse a11y n'échoue pas (`aria-label` suffit pour AXE), mais le SEO indexe un h1 vide. Failure logique, pas mécanique.
- **Fix concret** : mettre le texte en dur dans les `<span class="word">` et laisser JS wrapper les caractères à l'exécution sans vider le data-word. Ex :
  ```html
  <h1 class="hero-name" id="heroName">
    <span class="word" data-word="Corentin">Corentin</span>
    <span class="word" data-word="Devillé">Devillé</span>
    <span class="hero-star" aria-hidden="true">...</span>
  </h1>
  ```
  Dans le JS, avant d'injecter les `.char`, faire `w.textContent = ''` (ce qu'il fait déjà implicitement en ne lisant que `dataset.word`). Donc juste ajouter le texte enfant. Retirer l'`aria-label` devient optionnel.
- **Effort** : XS (2 min).

#### A11Y-02 [P0] — Contraste `--accent #FF4F00` sur `--bg #F0EEE9` = 3,4:1 (fail WCAG AA body text)
- **Observation** : `--accent #FF4F00` (vermillon) utilisé comme couleur de texte dans plusieurs endroits où le texte est **body < 18 pt** :
  - `index.html:402` `.topbar a:hover{color:var(--accent)}` — taille 14 px (fail si utilisé en normal-weight small)
  - `index.html:483` `.hero-meta a:hover{color:var(--accent)}` — font-size 14 px letter-spacing .14em (fail borderline)
  - `index.html:1859` — inconnu à ce stade, mais les `--accent` appliqués sur fond `--bg` dans un `font-family:var(--font-display)` passeront (large text)
  - Ratio calculé : `#FF4F00` (luminance relative ≈ 0,265) vs `#F0EEE9` (luminance ≈ 0,895) → **(0,895 + 0,05) / (0,265 + 0,05) = 3,00**. Re-vérif précise : **3,03:1**. Fail AA 4,5:1 body, pass AA 3:1 large (≥18 pt ou ≥14 pt bold), pass UI non-text 3:1.
- **Impact estimé** : Lighthouse a11y signalera 2-4 contrast failures. WCAG AA compliance formelle en échec sur body-sized text en orange. Le dark mode sauve la mise (`#FF4F00` sur `#0F0F0E` = ~4,58:1, pass AA body).
- **Fix concret** : deux options :
  1. **Réserver l'accent orange au "large text" et aux décorations** (icônes ▶, borders, backgrounds, SVG fills). Changer `:hover{color:var(--accent)}` en `:hover{color:var(--ink); text-decoration-color:var(--accent)}` ou `:hover{background:color-mix(in srgb, var(--accent) 12%, transparent)}`.
  2. **Foncer l'accent** pour le light mode uniquement : `--accent-text: #D83F00` (ratio ~4,6:1). Utiliser `--accent` pour les visuels, `--accent-text` pour les `color:`. En dark mode, `--accent-text = var(--accent)`.
     ```css
     :root { --accent: #FF4F00; --accent-text: #CC3D00; }
     body[data-theme="dark"] { --accent-text: #FF7A3D; }
     ```
- **Effort** : S (15 min — ajouter variable + search/replace des `color:var(--accent)` où c'est du body text).

#### A11Y-03 [P1] — Menu hamburger sans `aria-expanded` ni `aria-controls`
- **Observation** : `index.html:2576` → `<button class="menu-btn" aria-label="Menu">`. Le bouton ouvre `#mobMenu` (`index.html:2480`) via toggle classe `.open` sans informer l'AT de l'état (open/closed) ni de la cible contrôlée. Le `<button class="mm-close" id="mmClose">` ferme, mais aucun piège de focus n'est géré : au `open`, le focus reste sur le bouton hamburger derrière l'overlay plein écran.
- **Impact estimé** : WCAG 4.1.2 (Name, Role, Value) — borderline pass/fail selon l'interprétation. WCAG 2.1.2 (No Keyboard Trap) pass si Tab sort de l'overlay, mais WCAG 2.4.3 (Focus Order) en échec car l'ordre de focus ne suit pas l'ouverture.
- **Fix concret** :
  ```html
  <button class="menu-btn" aria-label="Menu" aria-expanded="false" aria-controls="mobMenu">
    <span></span><span></span>
  </button>
  ```
  ```js
  btn.addEventListener('click', () => {
    const isOpen = menu.classList.toggle('open');
    btn.setAttribute('aria-expanded', String(isOpen));
    if (isOpen) { close.focus(); document.body.style.overflow='hidden'; }
    else { btn.focus(); document.body.style.overflow=''; }
  });
  // Escape ferme
  document.addEventListener('keydown', e => {
    if (e.key === 'Escape' && menu.classList.contains('open')) {
      menu.classList.remove('open');
      btn.setAttribute('aria-expanded','false');
      btn.focus();
    }
  });
  ```
- **Effort** : S (15 min).

#### A11Y-04 [P1] — Carousels projets : 4 blocs `.reel-nav` avec focus management inexistant
- **Observation** : `index.html:2693, 2694, 2742, 2743, 2791, 2792` × 3 carousels = 6 boutons prev/next + dots (`.reel-dot`). Les boutons ont `aria-label="Précédent"` / `aria-label="Suivant"` (bon), mais :
  - Les `.reel-dot` ne sont pas des `<button>` mais des `<span>` → pas focusables clavier, pas de rôle. `index.html:2695` : `<span class="reel-dot active"></span>` × 3.
  - Aucun `aria-current="true"` sur le dot actif ni aucune annonce du slide courant.
  - Aucun `role="region" aria-roledescription="carousel" aria-label="Projets Wenda"` sur le conteneur `.ps-carousel`.
- **Impact estimé** : WCAG 2.1.1 (Keyboard) fail sur les dots, 4.1.2 borderline. Perception : un user clavier ne peut pas cibler un slide par dot, il doit spammer prev/next.
- **Fix concret** :
  ```html
  <div class="ps-carousel" id="carousel-wenda"
       role="region" aria-roledescription="carousel" aria-label="Les filles de Wenda — 3 reels">
    ...
    <div class="reel-dots" role="tablist">
      <button class="reel-dot active" role="tab" aria-selected="true" aria-label="Reel 1"></button>
      <button class="reel-dot"        role="tab" aria-selected="false" aria-label="Reel 2"></button>
      <button class="reel-dot"        role="tab" aria-selected="false" aria-label="Reel 3"></button>
    </div>
  </div>
  ```
  JS update : `dots.forEach((d,i)=>d.setAttribute('aria-selected', String(i===active)));`
- **Effort** : S (30 min — changer span→button × 12 dots + rendu JS).

#### A11Y-05 [P2] — `aside class="apropos-visual" aria-hidden="true"` cache le portrait aux AT
- **Observation** : `index.html:2954`. L'aside contient le portrait + la carte ID "Corentin Devillé, Vidéaste, Formateur, Bruxelles". Le `aria-hidden="true"` sur le conteneur masque **toute l'ID card** aux screen readers, y compris le texte "CD / Corentin Devillé / Vidéaste / Formateur / Bruxelles". Le texte du bio à droite (`.apropos-text`) est redondant avec ces infos, donc ce n'est **pas un bug bloquant**, mais c'est un choix à documenter.
- **Impact estimé** : aucun fail formel WCAG (contenu redondant ailleurs), mais la logique "décorative" est trompeuse — la carte ID contient de l'info sémantique unique (tag "ID · 001", BE / 2026) qui n'est pas ailleurs.
- **Fix concret** : soit retirer `aria-hidden` et laisser l'image `alt=""` (décorative) tandis que la carte ID est annoncée, soit assumer le choix et le justifier. Recommandé : retirer `aria-hidden="true"` de l'aside.
- **Effort** : XS (1 min).

#### A11Y-06 [P2] — `prefers-reduced-motion` ne désactive pas le morphing text, le scramble, le ticker
- **Observation** : `index.html:2242-2250` désactive `animation-duration`/`transition-duration` pour tous les éléments (`*`), mais :
  - Le morphing `scrambleTo()` (JS `index.html:3419-3439`) utilise `setInterval` à 35 ms pour réécrire `textContent` — aucune animation CSS impliquée, **donc non-désactivé par la media query**.
  - Le scramble des titres de section (JS `3441-3461`) idem.
  - Le ticker (CSS `@keyframes tickerScroll`) devrait être coupé par la media query mais les 3 carousels autoplay-vidéo continuent de tourner (vidéos = pas des animations CSS).
- **Impact estimé** : WCAG 2.3.3 (Animation from Interactions) borderline. L'utilisateur avec vestibular disorder voit toujours le morphing "Vidéaste → Formateur → …".
- **Fix concret** :
  ```js
  const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (!reduceMotion) {
    setInterval(()=>{ idx=(idx+1)%words.length; scrambleTo(words[idx]); }, 2800);
  } else {
    el.textContent = words[0]; // poser la valeur initiale seulement
  }
  ```
  Même traitement pour scramble titres et pour forcer `pause()` sur tous les `<video autoplay>` :
  ```js
  if (reduceMotion) document.querySelectorAll('video[autoplay]').forEach(v=>{v.pause();v.removeAttribute('autoplay');});
  ```
- **Effort** : S (10 min).

#### A11Y-07 [P2] — `role="marquee"` absent, ticker correctement `aria-hidden="true"`
- **Observation** : `index.html:2629` — `<div class="ticker-wrap ticker--in-hero" aria-hidden="true">`. Bon choix (contenu décoratif répétitif). **Ceci n'est pas un bug**, c'est une mention positive : le `role="marquee"` est déprécié et l'`aria-hidden` est la bonne solution.
- **Action** : aucune. Noté pour "Ce qu'on garde".

---

### SEO

#### SEO-01 [P0] — Aucun balisage Schema.org (Person / LocalBusiness / Service / VideoObject)
- **Observation** : Recherche `application/ld+json` → 0 hit. Recherche `schema.org` → 0 hit. Le site vend des services vidéo à Bruxelles sans aucun signal structuré → Google ne génère aucun Rich Result, aucun Knowledge Panel.
- **Impact estimé SEO** : manque d'éligibilité aux Rich Results. Cas concret : "vidéaste Bruxelles" → pas de carte Google My Business renforcée, pas de listing Services, pas de Freshness signal. **Lighthouse SEO** : -10 à -15 points.
- **Fix concret** : injecter un `<script type="application/ld+json">` en fin de `<head>` :
  ```html
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Person",
    "name": "Corentin Devillé",
    "alternateName": "CD Studio",
    "jobTitle": "Vidéaste & Formateur Premiere Pro",
    "url": "https://corentindeville.com",
    "image": "https://corentindeville.com/assets/portrait.png",
    "sameAs": [
      "https://www.instagram.com/corentin.deville",
      "https://www.youtube.com/@corentindeville"
    ],
    "address": {
      "@type": "PostalAddress",
      "addressLocality": "Bruxelles",
      "addressCountry": "BE"
    },
    "knowsAbout": ["Premiere Pro","Montage vidéo","Tournage","Drone","Aftermovie","Brand film"],
    "makesOffer": [
      {"@type":"Offer","itemOffered":{"@type":"Service","name":"Montage vidéo"},"priceSpecification":{"@type":"UnitPriceSpecification","price":"20","priceCurrency":"EUR","unitCode":"HUR"}},
      {"@type":"Offer","itemOffered":{"@type":"Service","name":"Tournage & drone"},"priceSpecification":{"@type":"UnitPriceSpecification","price":"25","priceCurrency":"EUR","unitCode":"HUR"}},
      {"@type":"Offer","itemOffered":{"@type":"Service","name":"Formation Premiere Pro"},"priceSpecification":{"@type":"UnitPriceSpecification","price":"20","priceCurrency":"EUR","unitCode":"HUR"}}
    ]
  }
  </script>
  ```
  Optionnel : ajouter un `VideoObject` par showreel, mais risque de duplicate si déjà sur YouTube.
- **Effort** : S (20 min — rédiger le JSON + valider sur https://validator.schema.org).

#### SEO-02 [P0] — Pas de `<link rel="canonical">`, pas d'`og:image`, pas d'`og:url`
- **Observation** : `index.html:7-11`. Présents : `og:title`, `og:description`, `og:type`, `og:locale`, `twitter:card=summary_large_image`. **Manquants** : `og:image` (obligatoire pour un `summary_large_image` Twitter), `og:url`, `canonical`. Résultat : partage WhatsApp / LinkedIn → card vide ou tombe en fallback scrape du h1 (vide, cf A11Y-01 → double pénalité).
- **Impact estimé SEO** : partages sociaux cassés. Crawl : Google peut indexer des variantes `www.` vs apex, `https://` vs `http://` → dilution PageRank.
- **Fix concret** :
  ```html
  <link rel="canonical" href="https://corentindeville.com/">
  <meta property="og:url" content="https://corentindeville.com/">
  <meta property="og:image" content="https://corentindeville.com/assets/og-cover-1200x630.jpg">
  <meta property="og:image:width" content="1200">
  <meta property="og:image:height" content="630">
  <meta property="og:image:alt" content="Portfolio Corentin Devillé — Vidéaste &amp; Formateur Premiere Pro">
  <meta name="twitter:image" content="https://corentindeville.com/assets/og-cover-1200x630.jpg">
  <meta name="twitter:title" content="Corentin Devillé — Vidéaste & Formateur Premiere Pro">
  <meta name="twitter:description" content="Tournage, montage, drone et cours Premiere Pro à Bruxelles.">
  ```
  Générer `og-cover-1200x630.jpg` : un export Figma/Photoshop 1200×630 avec nom + job + site URL, ~80-150 Ko en JPEG q82.
- **Effort** : S (15 min HTML + 30 min design OG image = total 45 min).

#### SEO-03 [P0] — `sitemap.xml` et `robots.txt` absents
- **Observation** : `ls /Users/cocs/Desktop/PORTFOLIO/` → aucun. Cloudflare Pages sert 404 sur `/robots.txt` → Google scrape avec politique par défaut, mais manque une directive claire + sitemap.
- **Impact estimé SEO** : sur un one-pager, l'impact est limité (une seule URL), mais Google Search Console se plaindra. Crawl budget non-optimisé pour les moteurs secondaires (Bing, DuckDuckGo).
- **Fix concret** : créer deux fichiers à la racine :

  `robots.txt` :
  ```
  User-agent: *
  Allow: /
  Sitemap: https://corentindeville.com/sitemap.xml
  ```
  `sitemap.xml` :
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    <url>
      <loc>https://corentindeville.com/</loc>
      <lastmod>2026-04-22</lastmod>
      <changefreq>monthly</changefreq>
      <priority>1.0</priority>
    </url>
  </urlset>
  ```
  Cloudflare Pages les sert automatiquement depuis `/public/` (ou racine si pas de build).
- **Effort** : XS (5 min).

#### SEO-04 [P1] — Hiérarchie Hn : `<h2 class="midcta-title">` rompt la séquence section→h2
- **Observation** : `index.html:2848` → `<h2 class="midcta-title">Un projet en tête ?</h2>` dans une `.midcta-band` qui est une *bande éditoriale* et non une section principale numérotée. Les 6 sections numérotées (`#projets`, `#services`, `#processus`, `#tarifs`, `#apropos`, `#contact`) ont chacune leur `<h2>`. Le midcta-band ajoute un 7e `<h2>` non-référencé dans la nav. Déjà flaggé dans `00-recon.md`.
- **Impact estimé SEO** : Google accepte plusieurs h2 par page, mais l'outline est un peu bruité. Non-bloquant.
- **Fix concret** : descendre en `<h3 class="midcta-title">` (cohérent avec le niveau "bande éditoriale" vs section principale), OU garder `<h2>` et assumer l'emphase. Je recommande **`<h3>`** pour cohérence outline.
- **Effort** : XS (30 s).

#### SEO-05 [P1] — Email obfusqué Cloudflare : `[email protected]` en DOM server-side
- **Observation** : `index.html:3012, 2534` → `<span class="__cf_email__" data-cfemail="...">[email&#160;protected]</span>`. Googlebot voit `[email protected]` comme texte avant exécution JS (il exécute du JS depuis 2019, donc il décode en général, mais la fiabilité n'est pas à 100%). Décision de sécurité acceptable contre harvest.
- **Impact estimé SEO** : marginal. L'email n'est de toute façon pas un signal SEO fort. L'obfusc handicape surtout les anciens crawlers.
- **Fix concret** (optionnel, trade-off) :
  - **Garder** si spam email est réel (✓ recommandé).
  - **Remplacer** par une fonction JS locale plus légère qui décode au `DOMContentLoaded`, évitant la dépendance `email-decode.min.js` : économie ~2 Ko + 1 requête. Exemple :
    ```html
    <a data-m="1">Email</a>
    <script>
      document.querySelectorAll('[data-m]').forEach(a => {
        a.href = 'mailto:' + ['corentin', 'corentindeville.com'].join('@');
        a.textContent = ['corentin', 'corentindeville.com'].join('@');
      });
    </script>
    ```
  - Sinon, accepter le choix actuel.
- **Effort** : S (10 min) si on remplace ; sinon rien.

#### SEO-06 [P2] — Favicon absent
- **Observation** : aucun `<link rel="icon">` dans `<head>`. Cloudflare renvoie 404 sur `/favicon.ico`. Chrome met un fallback grisé dans les tabs.
- **Impact estimé SEO** : nul direct, mais mauvais UX brand. Aussi utilisé par Google SERP (favicon snippet).
- **Fix concret** :
  ```html
  <link rel="icon" type="image/svg+xml" href="/favicon.svg">
  <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32.png">
  <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
  ```
  SVG mono-trait "CD" dans `--accent`.
- **Effort** : S (15 min).

---

## Cibles CWV réalistes après fix

Scénario mesure : Moto G4 simulé Chrome DevTools, 4G lente (1,6 Mbps / 150 ms RTT), cache vide, viewport 390×844.

| Métrique | Actuel estimé | Après P0 | Après P0 + P1 |
|---|---|---|---|
| **LCP** | 3,2 - 4,5 s (film-loader overlay ou showreel frame) | **2,1 - 2,6 s** (poster AVIF showreel) | **1,6 - 2,1 s** (+ font preload + loader court) |
| **CLS** | ~0,05 - 0,08 (font swap + reveal animations) | ~0,03 - 0,05 (img sized) | **< 0,02** |
| **INP** | 180 - 260 ms (10 vidéos en parallel scroll) | 130 - 180 ms (videos lazy) | **< 120 ms** (scrollspy rAF) |
| **Bundle total mobile** | ~2,7 Mo | ~0,4 Mo (portrait AVIF + videos deferred) | ~0,35 Mo (fonts self-host optionnel) |
| **Lighthouse Perf** | ~55-65 | ~80-85 | **90+** |
| **Lighthouse A11y** | ~88 (contrast + h1 vide + menu aria) | ~95 | **98+** |
| **Lighthouse SEO** | ~70 (pas schema, pas canonical, pas og:image) | **92-95** | 95+ |
| **Lighthouse Best Practices** | ~92 (favicon manquant + mixed preload) | 95 | 100 |

Cibles 2026 atteintes après P0 uniquement : **LCP ✓, CLS ✓, INP borderline**. Après P0 + P1 : **tout vert**.

---

## Ce qu'on garde

Éléments déjà bien faits — à ne pas casser dans la refonte :

1. **`html lang="fr"`** — déclaré correctement (`index.html:2`).
2. **`<meta name="viewport">`** propre sans `user-scalable=no` — accessibilité préservée.
3. **`<meta name="theme-color">`** — bien, harmonise Safari iOS / Chrome Android.
4. **Landmarks HTML5 complets** — `<header>` / `<nav>` / `<main>` / `<section>` (toutes numérotées avec `id`) / `<footer>` / `<aside>`. Bon choix.
5. **`aria-label` sur toutes les sections "band"** (proof, midcta, quote) et sur les boutons icon-only (menu, eye bubble, nav reel prev/next, FAB WhatsApp).
6. **`:focus-visible` défini** avec `outline:2px solid var(--accent);outline-offset:3px` (`index.html:2253`) — satisfait WCAG 2.4.7.
7. **`@media (prefers-reduced-motion: reduce)`** implémenté (`index.html:2242-2250`) — manque juste l'extension JS (A11Y-06) et la désactivation du film-loader.
8. **`preconnect` vers fonts.googleapis.com et fonts.gstatic.com** (`index.html:15-16`) avec `crossorigin` correct sur gstatic.
9. **`preload="metadata"` sur les vidéos** — pas le défaut `"auto"` qui téléchargerait tout. Bien (même si on peut pousser à `"none"` pour les carousels — PERF-03).
10. **Aucune dépendance JS tierce** hormis `email-decode.min.js` fourni par CF — bundle JS app à ~18 Ko non-minifié, très raisonnable.
11. **Ticker `aria-hidden="true"`** — correct, contenu décoratif répétitif non-lu par AT (pas de `role="marquee"` déprécié).
12. **Dark mode via `[data-theme]`** — séparation propre, contrastes OK en dark (muted et accent passent AA).
13. **Pas de `<table>` pour le layout, pas de `<br>` abusif** hors eyebrow/meta — HTML sémantique globalement propre.
14. **Vidéos hébergées R2** (`videos.corentindeville.com`) — HTTP/2, pas de coût bandwidth Cloudflare Pages, et pas d'iframe YouTube — bonne décision perf/privacy.
15. **Single-file vanilla** — cache HTML très simple, pas de hydration, pas de runtime framework. **On garde.**
