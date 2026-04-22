# AUDIT / 04 — Qualité de code & maintenabilité

> Audit read-only. Périmètre : `index.html` (3498 lignes / 155 Ko). CSS lignes 19 → 2456, HTML 2457 → 3040, JS 3081 → 3496.
> Date : 2026-04-22. Commit base : `a7024cc`.

---

## Executive Summary

Le fichier tient debout mais porte ~6 mois de dette stratifiée non nettoyée. **~900 lignes de CSS (≈37 % du bloc `<style>`) ciblent des classes qui n'existent plus dans le HTML** : une refonte « v1 » (cartes asymétriques `.pcard`, vidéos `.vcard`, liste `.crow`, bandeau `.footbanner`) a été remplacée par la v2 éditoriale (`.ps-ed*`, `.contact-grid--simple`, `.apropos-*`) sans supprimer l'ancien. Les 4 projets sont **hardcodés HTML** (~45 lignes chacun, ~180 lignes totales) avec copies-conformes du markup carousel — chaque ajout/modif = 5 endroits à toucher sans filet. 3 placeholders `[à compléter]` sont en clair dans le DOM en prod. Les 3 fichiers `index copie*.html` gitignorés sont l'artefact direct de ce manque de structure data-driven : tu versionnes par copie parce que tu n'as pas de source de vérité. La bonne nouvelle : l'architecture JS est propre (7 IIFE thématiques + 1 factory `initStackCarousel`), les tokens CSS existent (`:root` bien défini), les media queries sont peu nombreuses (5 breakpoints cohérents : 600/720/768/900/960/1100). Le coût du cleanup est **S à M**, pas L.

---

## Cartographie des dettes

### Dette CSS

| Catégorie | Nb estimé | Lignes impactées | Sévérité |
|---|---|---|---|
| Sélecteurs définis / jamais présents dans le HTML | **12 familles** (≈ 44 sélecteurs) | ~900 | **P0** |
| Duplications littérales (même sélecteur redéclaré) | **9** | ~20 | P1 |
| Overrides « patch » en fin de fichier (lignes 2437-2455) | 5 règles | 20 | P1 |
| Magic numbers non extraits en `--var` | ~40 occurrences | diffus | P2 |
| Media queries éparpillées (mêmes breakpoints répétés) | `max-width:600` ×6, `max-width:900` ×5, `max-width:960` ×3 | — | P2 |
| Règles contradictoires laissées sur place (`.ps-divider` défini puis `display:none`) | 3 | 15 | P1 |
| Utilisation répétée du trio `font-family:var(--font-mono);font-size:11px;letter-spacing:.18em;text-transform:uppercase` | **≥ 8 duplications verbales** (eyebrow / meta / kicker) | ~25 | P2 |

### Dette JS

| Pattern | Localisation | Sévérité |
|---|---|---|
| `initStackCarousel('carousel-scout')` appelé mais `#carousel-scout` n'existe pas dans le DOM (Scout = vidéo horizontale unique, pas un carousel) | ligne 3189 | **P0** |
| Bloc « Wide carousel — Bebooth » (lignes 3232-3257) cherche `.wide-card`, `.wide-dot`, `.wide-count`, `.wide-prev`, `.wide-next` dans `#carousel-bebooth` — **aucun de ces sélecteurs n'existe dans le HTML** (Bebooth utilise le carousel stack standard) | lignes 3232-3257 | **P0** |
| `document.querySelectorAll('.stats .n, .apropos-stats .n')` cherche `.stats` qui n'existe plus (résidu d'un ancien hero stats) — heureusement `.apropos-stats .n` matche, donc aucun bug, juste du bruit | lignes 3275, 3299 | P2 |
| `document.querySelectorAll('.cta,.hero-meta a,.menu-btn')` cherche `.cta` qui n'existe plus dans le HTML | ligne 3482 | P2 |
| Styles inline empilés dans une boucle JS (`link.style.cssText='display:inline-flex;gap:16px;…'`) dans le bloc « Next project links » au lieu d'une classe | lignes 3338-3341 | P2 |
| `const CHARS` répété 2 fois (deux blocs scramble) | lignes 3424 et 3443 | P2 |

### Dette structure fichier

- **Mono-fichier** : défendable pour un one-pager vanilla sans build. Le coût réel n'est pas le mono-fichier, c'est l'**absence de source de vérité pour les projets**.
- **Commentaires section CSS** : présents et `/* ===== Section ===== */` cohérents (68 blocs balisés). Bonne hygiène. Par contre, les **ordres thématiques ont dérivé** : la refonte v2 (`/* ===== À propos — photo + floating ID card ===== */` ligne 2405) vit loin de la v1 (`/* ===== À propos section ===== */` ligne 2002). Deux sections À propos dans le même fichier à 400 lignes d'écart.
- **Patch tardif** (lignes 2437-2455 `Editorial micro-moves`) : 5 overrides suffixés qui devraient être mergés dans leurs sections d'origine. C'est le smell classique du « j'ajoute à la fin pour être sûr que ça gagne en cascade ».
- **Fichier monolithique** : le CSS n'est pas minifié → 87 Ko envoyés en clair à chaque visite (sera abordé par l'agent perf, mais c'est aussi un sujet de code).
- **Fichiers parasites** gitignored mais visibles dans le workspace : `index copie.html`, `index copie 2.html`, `index copie 3.html`, `portfolio (2).html`, `handbrake-presets-cd.json`. **Symptôme direct de l'absence de data-driven** : tu dupliques le fichier pour garder une version de secours avant d'éditer à la main.

---

## Issues numérotées

### C-01 [P0] — Dead CSS : 12 familles orphelines depuis la refonte éditoriale v2

**Observation.** Les classes suivantes sont définies dans le CSS mais **n'existent plus dans le HTML** (greps ci-dessous confirmés à la ligne près) :

| Famille | Lignes CSS | Preuve d'absence HTML |
|---|---|---|
| `.pcard` + `.pcard.p1` → `.p4` (grille projets v1 asymétrique) | 561-596, 579-587 (glass), 1062, 1329 | `grep 'class="[^"]*\bpcard\b[^"]*"' index.html` → **0 match** |
| `.pgrid` (grid wrapper v1) | 557, 1061, 1328 | `grep 'class="[^"]*\bpgrid\b[^"]*"'` → **0 match** |
| `.vcard` + `.vcard.v1` → `.v7` + `.t1` → `.t7` (grille vidéos) | 944-986, 1065, 1355 | `grep 'class="[^"]*\bvcard\b[^"]*"'` → **0 match** |
| `.vgrid` (grid vidéos) | 943, 1064, 1354 | `grep 'class="[^"]*\bvgrid\b[^"]*"'` → **0 match** |
| `.crow` (lignes contact style tabulaire v1) | 990-1001, 1066, 1359-1363 | `grep 'class="[^"]*\bcrow\b[^"]*"'` → **0 match** |
| `.contact-list` (wrapper v1) | 989 | `grep 'class="[^"]*\bcontact-list\b[^"]*"'` → **0 match** |
| `.footbanner` + `.footbanner .fb-cap*` | 1003-1015, 1364-1365 | `grep 'class="[^"]*\bfootbanner\b[^"]*"'` → **0 match** |
| `.ps-block--below`, `.ps-below-top`, `.ps-below-tag`, `.ps-below-sub`, `.ps-below-desc`, `.ps-carousel--wide`, `.wide-stage`, `.wide-card`, `.wide-bar`, `.wide-dots`, `.wide-dot`, `.wide-navs`, `.wide-nav`, `.wide-count` (carousel Bebooth « wide » alternatif) | 826-892 | `grep 'class="[^"]*\b(ps-block--below\|wide-card\|ps-below-top)\b'` → **0 match** |
| `.ps-wide-layout`, `.ps-text--wide` (layout projet stacked alternatif) | 668-670 | `grep 'class="[^"]*\bps-wide-layout\b'` → **0 match** |
| `.wenda-bg`, `.wenda-grid`, `.wenda-carousel`, `.wenda-title`, `.wenda-sub`, `.wenda-p1`, `.wenda-p2` (mobile v1) | 1338-1346 | `grep 'class="[^"]*\bwenda\b[^"]*"'` → **0 match** |
| `.contact-portrait`, `.cp-frame`, `.cp-crosshair`, `.cp-mark`, `.cp-tag`, `.cp-avail`, `.cp-avail-dot`, `.contact-wa-*` (+ tout le thème vert contact v1) | 1692-1870, 1874-1938 (≈ 240 lignes) | `grep 'class="[^"]*\bcontact-portrait\b'` → **0 match**. Note : `#contact::before` (fond vert) ligne 1693 s'applique toujours au HTML actuel, mais ~150 lignes de `.cp-*` sont orphelines. |
| `.contact-link` / `.contact-lede` / `.contact-wa-btn` | encore utilisés (OK — à conserver) | — |
| `.topnav` (nav desktop v1) | 398-401 | `grep 'class="[^"]*\btopnav\b'` → **0 match** (remplacé par `.section-nav`) |
| `.avail-badge` / `.avail-dot` | 307-330 | `grep 'class="[^"]*\bavail-badge\b'` → **0 match** |
| `.brand-status` / `.brand-logo` / `.logo-wrap` / `.logo-img` / `.eyelid*` (ancien logo œil animé) | 360-395, 405 | aucun `<svg class="logo-svg">` ou `.eyelid-*` dans le HTML |

**Impact.**
- ~900 lignes de CSS indexé et envoyé au navigateur à chaque visite (poids pur).
- Friction lecture pour le développeur : quand tu ouvres le fichier pour un fix, tu passes devant 4 blocs « projects grid » / 2 « contact » / 2 « À propos » et dois deviner lequel est actif.
- **Risque de régression** : si tu copies du code d'un `index copie*.html` vers `index.html` en pensant réactiver un pattern, tu vas composer avec des cascades obsolètes.

**Fix concret.** Supprimer ces 12 familles d'un bloc. Grep de vérification avant suppression :
```bash
for cls in pcard vcard pgrid vgrid crow contact-list footbanner ps-block--below wide-card ps-wide-layout wenda-bg cp-frame topnav avail-badge brand-status eyelid; do
  echo "=== $cls ==="; grep -c "class=\"[^\"]*\\b$cls\\b" index.html
done
```
Chaque classe dont le count est 0 → supprime toutes ses règles CSS. Gain estimé : ~35 % du bloc `<style>`.

**Effort.** **M** (2h avec tests visuels sur mobile/desktop).

---

### C-02 [P0] — Dead JS : 2 IIFE entières ciblent des sélecteurs inexistants

**Observation.**
1. **Ligne 3189** : `['carousel-wenda','carousel-bebooth','carousel-lucid','carousel-scout'].forEach(initStackCarousel);` — `#carousel-scout` n'existe pas dans le HTML. Le scout est une vidéo horizontale unique dans `.scout-dual`. La factory bail out proprement (`if (!root) return;`), donc pas de crash, mais c'est un appel mort.
2. **Lignes 3232-3257** : le bloc « Wide carousel — Bebooth » cherche `#carousel-bebooth` puis `.wide-card`, `.wide-dot`, `.wide-count`, `.wide-prev`, `.wide-next`. Ces classes ont été supprimées du HTML quand Bebooth est passé au carousel stack standard. Le `const cards = [...root.querySelectorAll('.wide-card')]` retourne `[]` → `n = 0` → la fonction tourne dans le vide. **26 lignes de JS mortes.**
3. **Ligne 3275, 3299** : `'.stats .n, .apropos-stats .n'` — `.stats` n'existe plus (résidu d'un hero stats v0). Heureusement le second sélecteur matche.
4. **Ligne 3482** : `'.cta,.hero-meta a,.menu-btn'` — `.cta` n'existe plus dans le HTML.

**Impact.** Code indexé, parseurs/linters bruités, et surtout **signal trompeur** : si demain quelqu'un lit `['carousel-wenda','carousel-bebooth','carousel-lucid','carousel-scout']` il va penser que Scout a un carousel → faux contrat mental.

**Fix concret.**
```js
// Avant
['carousel-wenda','carousel-bebooth','carousel-lucid','carousel-scout'].forEach(initStackCarousel);

// Après
['carousel-wenda','carousel-bebooth','carousel-lucid'].forEach(initStackCarousel);
```
Supprimer entièrement le bloc `WIDE CAROUSEL — Bebooth` (lignes 3232-3257). Nettoyer les deux sélecteurs morts dans les querySelectorAll ligne 3275/3299/3482.

**Effort.** **S** (10 min).

---

### C-03 [P0] — Projets hardcodés : 4 × ~45 lignes de HTML dupliqué, shape copy-paste

**Observation.** Les 4 blocs projets (Wenda 2658-2702, Bebooth 2707-2751, Lucid 2756-2800, Scout 2805-2825) partagent la même forme : `.ps-block.ps-ed > .ps-bg > .ps-ed-header (idx+h3+meta) > .ps-ed-media (carousel ou scout-dual) > .ps-ed-desc (2×p)`. Seules varient :
- ID (`data-ps-id` + `id="carousel-X"`)
- Numéro section (`§ 01.1 / …`)
- Titre, catégorie meta (5 spans)
- 3 URLs vidéos (format 9:16 ou 16:9)
- 2 paragraphes description

Le markup carousel à 3 vidéos est **répété à l'identique 3 fois** dans Wenda, Bebooth, Lucid — ~25 lignes chacune. Scout casse le pattern (une seule vidéo full-width, pas de carousel) mais la coquille reste identique.

**Impact.**
- Ajouter un 5e projet = copier 45 lignes puis patcher 7-8 chaînes → **4 occasions de typo** (id mismatch entre `data-ps-id` et `id="carousel-X"` → JS ne bind pas le carousel).
- Corriger un bug de structure (ex : ajouter un bouton « play » à chaque carousel) = le faire 3 fois.
- **Les placeholders `[à compléter]` en HTML** (lignes 2748, 2749, 2759, 2761, 2797, 2798, 2822, 2823) sont indexés par Google et visibles en prod. Aucun fallback, aucun flag. C'est précisément le genre de chose que data-driven résout (un champ `description: null` peut afficher un placeholder stylé « Projet bientôt détaillé » au lieu du littéral `[à compléter]`).
- **Les 3 `index copie*.html` gitignorés** sont la conséquence directe : chaque édition risquée d'un projet = tu duplique le fichier avant.

**Fix concret.** Voir section **Data-driven — recommandation arbitrée** plus bas. Verdict court : **oui, passer à un `const PROJECTS = [...]` + template literal rendu en inline script vaut la peine à 4 projets**, précisément parce que l'état actuel produit déjà 3 fichiers de backup.

**Effort.** **M** (4-6h incluant migration des 4 projets + tests visuels + debug du binding `initStackCarousel` après rendu dynamique).

---

### C-04 [P1] — `.apropos-card::before` défini deux fois (une override en fin de fichier)

**Observation.**
```css
/* Ligne 2019-2028 */
.apropos-card::before{
  content:"";position:absolute;inset:0;z-index:0;pointer-events:none;
  background-image: linear-gradient(…);
  background-size:48px 48px;
  opacity:.35;
  mask-image: radial-gradient(…);
}

/* Ligne 2453 (section "Editorial micro-moves") */
.apropos-card::before{opacity:.12}
```
La seconde ne fait que corriger `opacity`. Deux règles pour un seul ajustement → le lecteur doit scroller 430 lignes pour voir la valeur finale.

**Impact.** Friction lecture. Inoffensif en termes de rendu. Typique du patch-à-la-fin.

**Fix concret.** Merger :
```css
.apropos-card::before{
  /* … */
  opacity:.12;    /* ex .35 — adouci dans refonte v2 pour laisser respirer la photo */
}
```

**Effort.** S (2 min).

---

### C-05 [P1] — `.ps-divider` : 15 lignes de CSS + 3 `<div class="ps-divider">` dans le HTML, puis `display:none` global en fin de fichier

**Observation.**
```css
/* Ligne 735-758 : définition complète (15 lignes, y compris mobile) */
.ps-divider{ max-width:1400px; margin:80px auto; display:flex; … }
.ps-divider span{padding:10px 22px;background:var(--ink);color:var(--bg)}
.ps-divider::before,.ps-divider::after{content:"";flex:1;height:2px;background:var(--ink)}
.ps-divider::before,.ps-divider::after{content:"";flex:1;height:1px;background:var(--rule)}  /* ← override intra-bloc */

/* Ligne 2450 */
.ps-divider{display:none}
```
Et dans le HTML lignes 2704, 2753, 2802 :
```html
<div class="ps-divider"><span>— 02 —</span></div>
```

Donc : **CSS défini + HTML généré + masqué en toute fin.** Le `<div>` est dans le DOM (coût de parsing, indexation). Et ligne 742-743 on voit même **deux définitions consécutives contradictoires** de `::before,::after` (`height:2px` puis `height:1px`) — la 2e gagne en cascade, la 1re est morte.

**Impact.** 3 `<div>` inutiles dans le DOM, 15 lignes de CSS exécutées pour rien, un bug latent si quelqu'un retire le `display:none` ligne 2450.

**Fix concret.** Supprimer les 3 `<div class="ps-divider">` du HTML + supprimer toutes les règles `.ps-divider*` du CSS.

**Effort.** S (5 min).

---

### C-06 [P1] — `.topbar .brand`, `.hero-reel .ph-label`, `.apropos-visual` : chacun défini deux fois

**Observation.**
- `.topbar .brand` : ligne 349 (`display:flex;gap:10px;align-items:center;white-space:nowrap`) puis ligne 357 (`display:flex;gap:12px;align-items:center`). La 2e override juste `gap`, mais répète 2 propriétés identiques.
- `.hero-reel .ph-label` : ligne 504-508 (bloc complet) puis ligne 509 (`color:#F0EEE9`) puis ligne 512 (`z-index:2;color:#F0EEE9` via liste) puis ligne 513 (`.hero-reel .ph-label .big{color:#F0EEE9}`). **3 redéfinitions de `color:#F0EEE9` sur 10 lignes.**
- `.apropos-visual` : ligne 2006 (`position:relative`) et ligne 2406 (`position:relative;display:flex;justify-content:center;align-items:flex-start`). La 2e englobe la 1re ligne 400 plus loin.

**Impact.** Pur bruit. Le 2e intervient à chaque fois dans un bloc de refonte posé à côté au lieu de merger. Zéro bug, juste friction.

**Fix concret.** Merger chacune dans une seule déclaration complète.

**Effort.** S (5 min total).

---

### C-07 [P1] — Patch block en fin de `<style>` (lignes 2437-2455)

**Observation.** Bloc commenté `/* ===== Editorial micro-moves ===== */` qui contient 5 mini-règles :
```css
.ps-ed-meta span{background:transparent!important;color:var(--ink);padding:0}
.ps-ed-meta span + span::before{ content:"·";color:var(--accent);margin:0 16px;position:static;font-weight:600; }
.ps-ed-header{position:relative}
.ps-ed-header::before{ content:""; position:absolute; top:-16px; left:0; width:48px; height:2px; background:var(--accent); }
.ps-divider{display:none}
.ps-block + .ps-block{margin-top:140px}
.apropos-card::before{opacity:.12}
.ps-idx{letter-spacing:.24em}
```
- 3 de ces règles **overrident** des valeurs définies 1600 lignes plus haut (ligne 706-712 pour `.ps-ed-meta`, ligne 677-685 pour `.ps-ed-header`).
- `!important` sur `.ps-ed-meta span` signale que l'auteur a rencontré une cascade qu'il n'a pas voulu déméler.
- `.ps-divider{display:none}` est le 3e signal de la dette C-05.

**Impact.** Le lecteur doit scroller tout le fichier pour voir les valeurs finales. `!important` = dette qui se propage.

**Fix concret.** Migrer chacune de ces 8 règles dans leur section d'origine, supprimer le bloc final. Retirer `!important` après avoir vérifié qu'aucune cascade de `data-mode="glass"` n'écrase — pour `.ps-ed-meta span` c'est ok.

**Effort.** S (15 min).

---

### C-08 [P1] — Magic numbers sur les animations et transitions

**Observation.** 59 `cubic-bezier(…)` différents dans le fichier. 68 `clamp(…)`. Plusieurs values fréquentes mais non tokenisées :
- `cubic-bezier(.22,1,.36,1)` apparaît ~18 fois (ease modern cross-feature)
- `cubic-bezier(.2,.8,.2,1)` ~6 fois (reveal animations)
- `cubic-bezier(.65,0,.25,1)` (film-loader) unique, ok
- Transitions typées mais non nommées : `transition:.3s`, `transition:.35s`, `transition:.4s`, `transition:.45s`, `transition:.55s` coexistent sans système
- Durées animations : `60s` (proof-band), `28s` (ticker), `8s` (star spin), `7s` (eye blink), `5s` (eyelid v1, dead mais présent) — difficile de coordonner un ralentissement global

Les valeurs sont aussi hardcoded dans :
- `padding:28px 24px 28px 0` (method-cell) — une combinaison bespoke
- `margin:0 0 40px` (`.ps-ed-title`)
- `gap:80px` (`.ps-grid`), `gap:72px` (`.apropos-grid`), `gap:40px` (`.ps-grid` mobile)

**Impact.** Pour adoucir/durcir globalement l'animation, il faut toucher 18 endroits. Pour changer le rythme des sections, idem.

**Fix concret.** Ajouter dans `:root` :
```css
:root{
  /* … existants … */
  --ease-out:     cubic-bezier(.22,1,.36,1);
  --ease-in-out:  cubic-bezier(.2,.8,.2,1);
  --dur-fast:     .25s;
  --dur-med:      .4s;
  --dur-slow:     .6s;

  --gap-section:  80px;
  --gap-grid:     72px;
  --gap-tight:    24px;
}
```
Puis rechercher/remplacer les ~18 occurrences. Ce n'est pas urgent mais ça rendra la prochaine refonte visuelle trivial.

**Effort.** M (1-2h, à faire en même temps que le cleanup dead CSS).

---

### C-09 [P2] — Médias queries éparpillées : mêmes breakpoints recollés à différents endroits

**Observation.** Breakpoints utilisés (extrait) :
- `max-width:600` : lignes 889, 894, 1188, 1521, 1673, 1874, 2097, 2221, 2276, 2286, 2383 (×11)
- `max-width:900` : lignes 1869, 2085, 2227, 2231, 2263 (×5 dont 2 identiques consécutifs)
- `max-width:960` : lignes 816, 888, 1058 (×3)
- `max-width:768` : lignes 744, 1427 (×2)
- `max-width:1100` : lignes 356, 1474, 2266 (×3)

Cela fait **~25 blocs `@media`** là où 5 regroupements suffiraient. En plus, lignes 2227 et 2231 : **deux blocs `@media(max-width:900px)` consécutifs** non fusionnés.

**Impact.** Pour modifier le layout mobile, tu ouvres 11 blocs. Impossible de « voir d'un coup » ce qui se passe à 600px.

**Fix concret.** Regrouper par breakpoint (un gros `@media (max-width:600px) { … }` par breakpoint, ou à défaut, nommer les breakpoints avec un commentaire cohérent). Ne pas chercher la perfection : fusionner les 2 blocs `@media(max-width:900px)` adjacents (lignes 2227 et 2231) est un win de 2 minutes.

**Effort.** S pour la fusion ponctuelle ; M si regroupement global.

---

### C-10 [P2] — Commentaires éparpillés, certains obsolètes ou trompeurs

**Observation.** Bonne hygiène globale (68 balises `/* ===== Section ===== */`) mais dérives :
- Ligne 894 : `/* ===== Mobile for project sections ===== */@media (max-width: 600px){` — commentaire collé à la règle, visuellement illisible.
- Ligne 1328-1329 : le commentaire `/* Projects grid */` introduit des règles `.pcard.p1, .pcard.p2, .pcard.p3, .pcard.p4` qui sont dead code depuis la v2 → commentaire **obsolète car il documente un composant disparu**.
- Ligne 1989 : `/* ===== Tariffs list (generalised, no longer scoped to .info-body) ===== */` — note historique utile, mais `.info-body` n'existe nulle part, donc la parenthèse n'aide plus personne.
- Ligne 2251 : `/* Hide tweaks on small screens */` suivi d'aucune règle (bloc vide).
- Ligne 3328 : `/* ========== MOUSE-REACTIVE HERO TITLE — disabled ========== */` puis… rien. Le comment lui-même est muet mais vestige d'un bloc retiré.

**Impact.** Minor. Un relecteur passe 5s sur chaque pour comprendre si c'est un TODO ou du vestige.

**Fix concret.** Nettoyer au fil du cleanup C-01.

**Effort.** S (à faire pendant C-01).

---

### C-11 [P2] — Méthode (4 étapes) et Tarifs (3 lignes) hardcodés : cohérence avec le data-driven des projets

**Observation.** Si les projets passent en data-driven (voir arbitrage plus bas), les 4 étapes méthode (lignes 2890-2909) et les 3 tarifs (2921-2932) suivent la même structure copy-paste. Cohérence de traitement : soit tout reste HTML, soit tout passe par un `const` + render.

**Impact.** Pas critique. 4 étapes qui changent rarement ≠ 10+ projets qui changent souvent. Mais si tu passes au data-driven pour les projets, ajouter `const METHOD = [4 étapes]` et `const TARIFS = [3 lignes]` est trivial.

**Fix concret.** Voir arbitrage final.

**Effort.** S (si déjà engagé dans le template literal pattern).

---

### C-12 [P2] — Selected Work Index (`.plist` / `.prow r1-r6`) : 6 lignes placeholder hardcodées et classes `.r1-.r6` qui ne servent qu'à pinter la couleur de la preview

**Observation.** Lignes 2834-2839, 6 lignes placeholder `[Projet 05]` → `[Projet 10]`. Chacune porte une classe `.r1`, `.r2`, … `.r6` dont le SEUL but est de styler `.prow.r1 .preview{background:linear-gradient(…)}` (lignes 928-933).

**Impact.**
- Le `.preview` est en plus masqué `display:none` dès `max-width:960`, donc la classe ne sert à rien en mobile.
- Les 6 lignes de texte `[Projet 05]… [Projet 10]` sont crawlées par Google.
- Ajouter un 7e projet = ajouter `.prow.r7 .preview{background:…}` + une ligne de CSS pour chaque slot supplémentaire. **Scaling faible.**

**Fix concret.**
- Option A (quick) : inline le dégradé via `style=""` sur chaque `.prow`, supprimer les 6 classes `.r1-.r6` en CSS.
- Option B (propre) : data-driven avec `const OTHER_WORK = [{num, name, cat, gradient}, …]` et render JS.

**Effort.** S.

---

## Refactors recommandés par ordre d'impact

1. **C-01 + C-02 : Purge du dead code CSS + JS** — Impact **élevé** (lisibilité, poids, signal), effort **M** (2h). Commence ici : le reste devient plus lisible une fois que les 900 lignes mortes sont parties.
2. **C-03 + recommandation data-driven** — Impact **élevé** (supprime le besoin des 3 `index copie*.html`, rend le placeholder `[à compléter]` gérable), effort **M** (4-6h). Prochaine étape naturelle après le cleanup.
3. **C-05 + C-07 : Merger les patchs en fin de fichier + supprimer `.ps-divider`** — Impact **moyen**, effort **S** (20 min).
4. **C-08 : Tokeniser ease/durée/gaps dans `:root`** — Impact **moyen** (prépare le terrain pour refontes futures), effort **M** (1-2h). À faire pendant C-01.
5. **C-04 + C-06 : Dédoublonner `.apropos-card::before`, `.topbar .brand`, `.hero-reel .ph-label`** — Impact **faible**, effort **S** (10 min). Quick win.
6. **C-09 : Regrouper les `@media`** — Impact **faible** (lisibilité), effort **M**. À faire en dernier, et seulement si tu comptes refaire une passe d'ajustements mobiles.
7. **C-10, C-11, C-12** — Hygiène, à traiter opportunément.

---

## Data-driven — recommandation arbitrée

### Verdict : **OUI, extraire les 4 projets dans un `const PROJECTS = [...]` + render via template literal en inline script.**

### Arbitrage

**À 4 projets** on pourrait arguer que 4×45 lignes = 180 lignes reste maintenable. C'est vrai si les projets sont statiques. **Ils ne le sont pas** :
- 3 des 4 projets portent `[à compléter]` en prod → tu VAS les éditer dans les semaines à venir.
- Le fait qu'il y ait 3 `index copie*.html` dans le workspace prouve que chaque édition te fait peur au point que tu préfères dupliquer le fichier en backup. C'est une friction réelle, mesurée.
- Le format DOM des projets a déjà été refondu 2 fois (v1 `.ps-block` + `.ps-grid` + `.ps-carousel`, puis v2 `.ps-ed*`). À la 3e refonte, tu voudras éditer **un template**, pas 4 copies.
- La grille `.plist` prévoit **10 projets** au total (05 → 10). Si 6 autres viennent, on passe à 10×45 = 450 lignes HTML. Là le HTML devient ingérable sans structure.

**À 4 projets + 4 étapes + 3 tarifs + 1 testimonial + 6 lignes index**, la valeur totale en data extractible est ~250 lignes. C'est le bon seuil pour un refactor data-driven léger.

### Proposition concrète (compatible no-build, vanilla, zéro dépendance)

Ajouter en début de `<script>` (avant les IIFE actuelles) :

```js
/* ========== CONTENT DATA ========== */
const PROJECTS = [
  {
    id: 'wenda',
    num: '01.1',
    category: 'Mode — Social content',
    title: 'Les filles de Wenda',
    client: 'Les Filles de Wenda',
    tag: 'Fashion — Social content',
    location: 'Waterloo / Brussels',
    year: '2024 — 2026',
    format: '9:16',
    media: {
      kind: 'carousel-vertical',
      videos: [
        'https://videos.corentindeville.com/wenda-reel-1.mp4',
        'https://videos.corentindeville.com/wenda-reel-2.mp4',
        'https://videos.corentindeville.com/wenda-reel-3.mp4',
      ],
    },
    description: [
      "Je réalise les vidéos et contenus visuels pour Les Filles de Wenda, maison de mode belge fondée en 1972 par Wenda d'Ursel et Pierre Marinof, aujourd'hui reprise par leurs filles Lara et Marina Marinof. Une histoire de famille, de transmission et de style.",
      "Mon travail : capturer l'élégance intemporelle et la modernité des collections à travers des reels dynamiques pour Instagram — mises en scène des nouvelles pièces, ambiances boutique, moments de shooting.",
    ],
  },
  {
    id: 'bebooth',
    num: '01.2',
    category: 'Événementiel — Photobooth',
    title: 'Bebooth',
    client: 'Bebooth',
    tag: 'Event — Photobooth',
    location: 'Brussels',
    year: '2025',
    format: '16:9',
    media: {
      kind: 'carousel-horizontal',
      videos: [
        'https://videos.corentindeville.com/bebooth-reel-1.mp4',
        'https://videos.corentindeville.com/bebooth-reel-2.mp4',
        'https://videos.corentindeville.com/bebooth-reel-3.mp4',
      ],
    },
    description: null,  // placeholder géré par le render, pas de texte hardcodé
  },
  {
    id: 'lucid',
    num: '01.3',
    category: null,
    title: 'Lucid Dreams',
    /* … */
    description: null,
  },
  {
    id: 'scout',
    num: '01.4',
    category: 'Événementiel — Aftermovie',
    title: 'Scout Festival',
    client: 'Scout Festival',
    tag: 'Event — Aftermovie',
    location: 'Belgique',
    year: '2026',
    format: '16:9',
    media: {
      kind: 'single-horizontal',
      video: 'https://videos.corentindeville.com/aftermovie-scout.mp4',
    },
    description: null,
  },
];

function renderProject(p){
  const metaSpans = [p.client, p.tag, p.location, p.year, p.format]
    .filter(Boolean).map(s => `<span>${s}</span>`).join('');

  const mediaHTML = p.media.kind === 'single-horizontal'
    ? `<div class="scout-dual"><div class="scout-horiz"><video src="${p.media.video}" autoplay muted loop playsinline preload="metadata" style="width:100%;height:100%;object-fit:cover;display:block;background:#0e0d0b"></video></div></div>`
    : `<div class="ps-carousel${p.media.kind === 'carousel-horizontal' ? ' ps-carousel--big' : ''}" id="carousel-${p.id}"${p.media.kind === 'carousel-horizontal' ? ' data-format="horizontal"' : ''}>
        <div class="reel-stage"${p.media.kind === 'carousel-horizontal' ? ' data-format="horizontal"' : ''}>
          ${p.media.videos.map(src => `
            <div class="reel-card">
              <div class="thumb" style="position:absolute;inset:0;background:#0e0d0b">
                <video src="${src}" autoplay muted loop playsinline preload="metadata" style="width:100%;height:100%;object-fit:cover;display:block"></video>
              </div>
              <div class="reel-hl"></div>
            </div>`).join('')}
        </div>
        <button class="reel-nav reel-prev" aria-label="Précédent">‹</button>
        <button class="reel-nav reel-next" aria-label="Suivant">›</button>
        <div class="reel-dots">${p.media.videos.map((_, i) => `<span class="reel-dot${i === 0 ? ' active' : ''}"></span>`).join('')}</div>
      </div>`;

  const descHTML = p.description
    ? p.description.map(para => `<p class="ps-p">${para}</p>`).join('')
    : `<p class="ps-p ps-p--placeholder">Projet bientôt détaillé.</p>`; // placeholder stylé, pas de [à compléter] crawlé

  return `
    <article class="ps-block ps-ed reveal" data-ps-id="${p.id}">
      <div class="ps-bg"></div>
      <div class="ps-ed-header">
        <div class="ps-idx">§ ${p.num}${p.category ? ' / ' + p.category : ''}</div>
        <h3 class="ps-ed-title">${p.title}</h3>
        <div class="ps-ed-meta">${metaSpans}</div>
      </div>
      <div class="ps-ed-media">${mediaHTML}</div>
      <div class="ps-ed-desc">${descHTML}</div>
    </article>`;
}

// Render once, before IntersectionObserver and carousel init run
const projectsContainer = document.getElementById('projects-container');
if (projectsContainer) {
  projectsContainer.innerHTML = PROJECTS.map(renderProject).join('');
}
```

Puis dans le HTML :
```html
<section class="section wrap" id="projets" …>
  <div class="section-head reveal">…</div>
  <div id="projects-container"></div>
  <!-- Selected Work Index (peut aussi être data-driven) -->
</section>
```

### Pourquoi c'est raisonnable ici et non over-engineering
1. **Zéro dépendance ajoutée** — template literal natif.
2. **Zéro build step** — le rendu se fait au premier tick après parsing, avant l'IntersectionObserver.
3. **Compatible avec Cloudflare Pages** — c'est du HTML/JS statique.
4. **Edit friction divisée par 5** — ajouter un projet = objet JS de 10 lignes. Changer un URL = 1 ligne. Corriger un typo description = 1 ligne. Plus besoin des `index copie*.html`.
5. **Placeholders gérés proprement** — un `description: null` affiche un placeholder stylé, pas `[à compléter]` crawlé.
6. **Carousels continuent de fonctionner** — `initStackCarousel` est appelé APRÈS `innerHTML`, les IDs matchent toujours (`carousel-${p.id}`).

### Coûts à anticiper
- **SEO / crawl** : le contenu n'est plus dans le HTML statique initial, il est injecté au parsing. **Pour Google c'est OK** (Googlebot exécute le JS depuis 2019), mais si tu vises aussi des crawlers plus anciens (Bing partiel, LinkedIn preview) il peut y avoir un manque de texte sur le snippet. Parade : garder les `<h3>` des projets dans le HTML statique + injecter le reste. Alternative : noscript fallback avec la liste textuelle.
- **Réordonnement des IIFE** : `['carousel-wenda','carousel-bebooth','carousel-lucid'].forEach(initStackCarousel)` doit s'exécuter **après** `innerHTML`. Solution : wrap dans un `requestAnimationFrame` ou DOMContentLoaded (déjà implicite vu que le `<script>` est en bas de `<body>`).

### Portée recommandée du refactor data-driven
- **Phase 1 (obligatoire si on fait le refactor)** : `PROJECTS[]` + render. Résout 90 % du problème.
- **Phase 2 (optionnelle)** : `METHOD_STEPS[]`, `TARIFS[]`, `OTHER_WORK[]` (les 6 lignes placeholder Selected Work) — cohérence de traitement, marginal.
- **Phase 3 (non recommandée)** : externaliser en `projects.json` fetch. Inutile pour 4 projets, rajoute un round-trip réseau.

---

## Ce qu'on garde

Beaucoup de choses sont **déjà bien faites**, et méritent d'être préservées pendant le cleanup :

1. **Tokens CSS dans `:root`** (lignes 20-35) — `--bg`, `--ink`, `--accent`, `--rule`, `--muted`, `--font-display/body/mono`, `--page-pad`, `--grid-size`. Bonne base, juste à étendre (voir C-08).
2. **Dark mode via attribute selector** (`body[data-theme="dark"]` lignes 38-51) — pattern solide, compatible avec le Tweaks panel.
3. **`initStackCarousel` comme factory** (lignes 3162-3187) — c'est du **bon JS** : une fonction, N éléments. Pas de duplication. Une fois Scout retiré de la liste d'appel, c'est clean.
4. **IntersectionObserver** pour reveal et pour le contact band (`in-contact` class) — pattern vanilla moderne et performant.
5. **Scrollspy modulaire** (bloc lignes 3191-3230) — lit `data-target` sur les liens, ne duplique pas la liste des sections. Ça scale si tu ajoutes une section.
6. **Tweaks panel + `applyTweaks`** — bien séparé, bon contrat de données via `TWEAKS` object avec marqueur `EDITMODE-BEGIN/END`. À conserver tel quel.
7. **Structure des `/* ===== Section ===== */`** — ~68 balises cohérentes, on voit d'où on est dans le fichier. Continuer ce style après cleanup.
8. **Zéro dépendance JS** — règle d'or à maintenir. Le refactor data-driven proposé ne la viole pas.
9. **`@media (prefers-reduced-motion: reduce)`** (ligne 2242) — accessibilité bien pensée, à garder.
10. **Landmarks HTML** (`<header>`, `<main>`, `<section aria-label="…">`, `<footer>`) — bon réflexe structurel.

### Fichiers parasites à nettoyer (hors `index.html`)

- `index copie.html` (125 Ko, 21 Apr 14:16)
- `index copie 2.html` (135 Ko, 21 Apr 16:48)
- `index copie 3.html` (142 Ko, 21 Apr 23:10)
- `portfolio (2).html` (120 Ko, 21 Apr 12:31)

Ces 4 fichiers sont gitignored (donc pas de problème git) mais **encombrent le workspace** et, surtout, **prouvent structurellement le besoin du refactor C-03** : tu duplique le fichier parce que tu n'as pas de source de vérité pour tes projets. Une fois `PROJECTS[]` en place, ces backups deviendront superflus.

**Recommandation** : après avoir migré en data-driven (C-03) et vérifié le live, déplacer ces 4 fichiers hors du workspace (dans un `.trash-2026-04/` sur le Desktop par ex, ou simplement les supprimer puisqu'ils sont gitignored et non nécessaires au déploiement). Garder 1 seul snapshot si tu tiens à un plan B — pas 4.

Le `handbrake-presets-cd.json` (15 Ko) n'est pas parasite : c'est un preset HandBrake utilisé pour encoder les vidéos R2. À garder mais peut-être à déplacer dans `/assets/` ou un futur `/tools/`.

---

**Fin de l'audit.** Trois P0 (C-01, C-02, C-03) couvrent 80 % de la dette. Effort total estimé **8-10h** pour un cleanup propre. Le site marche aujourd'hui, mais chaque ajout de projet va devenir plus pénible tant que C-03 n'est pas traité.
