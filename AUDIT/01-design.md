# AUDIT / 01 — Design & identité visuelle

> Phase agent-1 : lecture complète `index.html` (CSS 19-2457 + HTML 2558-3040), read-only.
> Viewport de référence mobile : 390px. Contexte posé par `00-recon.md`.

---

## Executive Summary

Le portfolio a une **armature éditoriale solide** (Dela Gothic + Inter + JetBrains Mono, palette sable/ink/vermillon, numérotation §), mais il **se tire plusieurs balles dans le pied** : accent `#FF4F00` surutilisé (71 occurrences — bordures, puces, underlines, borders, prices, stats, focus, hovers), 3 systèmes d'eyebrow qui ne se parlent pas (`.idx` box noire vs `.block-idx` mono vs `.ps-ed-header .ps-idx` autre box noire plus grosse), un section `#contact` qui casse tout le moodboard en passant au **vert foncé #1F5F42** sans raison narrative, et **trois couches d'overlays visuels** (grain SVG + `#film-grain` + curseur custom `cursor:none`) qui sentent le template abandonné plus que l'identité maîtrisée. L'impression « magazine-print » tient en statique ; elle **se dissout en mouvement**. Priorité : resserrer l'accent, unifier les eyebrows, trancher le vert contact, virer les overlays qui ne servent plus.

---

## Issues numérotées

### D-01 [P0] — Section Contact casse l'identité éditoriale
- **Observation** : `#contact::before` (ligne 1693-1707) force un fond vert `#1F5F42` full-viewport (`width:100vw`), avec gradients verts, alors que les 5 autres sections vivent sur `var(--bg)` sable. Cette décision n'est pas motivée par la narration : on passe d'un univers sable-ink-vermillon à un vert bouteille façon « site WhatsApp officiel » en 1 section. Le choix vient probablement de l'idée « rappel WhatsApp = vert » mais tue l'impression éditoriale cohérente. Le bouton `.contact-wa-btn` (ligne 1814) utilise ensuite `#F0EEE9` sur `#1F5F42`, ce qui est fonctionnel mais accentue la rupture.
- **Impact** : en scroll continu, le visiteur a l'impression que la section Contact est une landing page différente (ou rapatriée d'un autre projet). Rompt la signature « même édition ». Un recruteur qui fait le tour du pot en 30s repère cette dissonance avant tout le reste.
- **Fix concret** : supprimer `#contact::before` (ligne 1694-1707) ; passer tout Contact sur `var(--ink)` full-bleed (comme `.midcta-band` ligne 2324), avec `color:var(--bg)`. Garder le bouton WhatsApp vert `#25D366` **uniquement comme icon-circle interne** (pattern déjà en place dans `.contact-wa-icon` ligne 1822) et le label principal en accent vermillon. Le vert devient un détail, pas un environnement.
- **Effort** : S (suppression + 4 overrides couleur).

### D-02 [P0] — Accent vermillon surutilisé, perd son rôle de ponctuation
- **Observation** : `var(--accent)` apparaît **69 fois** dans le fichier. Inventaire partiel (non-exhaustif) : border-bottom hero-reel (ligne 493), border-right footbanner (1007), border-left cp-frame (1725, 1793), underline hero-meta (481), triangle `▶` des `.idx` (543, 686), arrow `↓` des `.ps-ed-title` (694), hover pcard h3 span (567-568), dot reel-dots (795), color .crow .num (996), hover .crow (1000-1001), hover .prow (919), text `.tariffs-list strong` (1999), color `.tr-price` (2377), `.method-num` (1979), `.method-cell::before` (1975), `.contact-wa-arrow` (1839), `::first-letter` .qb-text (2400), `.hs-num` (1400), `.hs-num::after` (1403), `.apropos-card-initials` (2046), `.apropos-quote em` (2063), etc.
- **Impact** : dans un lexique éditorial, l'accent est un **signe de ponctuation rare** (capitale ornée, jump-line, marque de fin). Ici il sert aussi bien de couleur texte principale (prix, puces de section), que de bordure structurelle, que de couleur hover générique. Résultat : il ne signale plus rien. L'œil s'y habitue en 3s et l'ignore. C'est le signe n°1 d'un template generic-IA.
- **Fix concret** : hiérarchiser les usages. Tier A (ponctuation éditoriale forte) : `▶` dans `.idx`, underline hover sur liens hero-meta, `::first-letter` de `.qb-text`, dot pulsante du hero playtag. Tier B (accent de prix) : `.tr-price`, `.crow .num`. **Interdire** l'accent pour : bordures structurelles (ligne 493, 1007, 1725 — passer à `var(--ink)` ou supprimer), underlines génériques de hover (remplacer par `var(--ink)` 2px), `.apropos-quote em` (passer en italic + ink), `.apropos-card-initials` (passer en `var(--ink)` — la taille fait le travail, pas la couleur). Viser **≤ 25 occurrences** dans le fichier.
- **Effort** : M (audit sélectif + remplacement avec diff ligne-à-ligne).

### D-03 [P0] — Hiérarchie des eyebrows incohérente (3 systèmes coexistent)
- **Observation** : on détecte trois patterns de « petit label mono en haut d'un bloc » qui ne se parlent pas :
  - **Système A** — `.section-head .idx` (ligne 538-543) : box noire `var(--ink)`, texte `var(--bg)`, 18px, `▶` orange, padding 8×16.
  - **Système B** — `.ps-ed-header .ps-idx` (ligne 678-686) : box noire **aussi**, mais **22px**, padding 10×18, `▶` 16px. Plus gros que le §01 de section ? Incohérent hiérarchiquement (le sous-§ ne doit pas écraser le §).
  - **Système C** — `.block-idx` (1948), `.ps-idx` simple (659), `.ss-idx` (2320), `.midcta-idx` (2342), `.hero-eyebrow` (422), `.proof-label` (2300), `.method-num` (1977), `.tr-num` (2366) : tous `JetBrains Mono`, 11-14px, en texte simple sans boîte, avec des letter-spacings allant de `.14em` à `.25em`. On trouve `.18em` (422), `.2em` (659, 2300), `.22em` (538, 659-override, 678, 2320, 2342, 1948, 2366), `.24em` (2455), `.25em` (1978). **Cinq valeurs pour la même fonction**.
- **Impact** : un visiteur ne peut pas distinguer en 1 seconde ce qui est niveau section / niveau projet / niveau sub-block. La grammaire éditoriale s'effondre : on lit 6 « § 01 / … », « § 01.1 / … », « § 02 / … » sans sentir qu'il y a une règle. Un magazine vrai n'a jamais ça.
- **Fix concret** : 
  - **Eyebrow niveau A (section)** = box `var(--ink)` sur bg, 12px mono, `.22em`, padding 6×12. 
  - **Eyebrow niveau B (sous-section / sous-bloc / projet)** = pas de box, mono plain 11px, `.22em`, `var(--muted)`, tiret accent devant (`— §01.1 / …`). 
  - Supprimer `.ps-ed-header .ps-idx` override ligne 678-686 (trop gros) ; fusionner avec niveau B. 
  - Unifier **tous** les letter-spacings eyebrow à `.22em`. Passer à la trappe le `.24em` override ligne 2455.
- **Effort** : M (supprimer 2 définitions CSS, harmoniser les valeurs).

### D-04 [P1] — Le cursor custom `cursor:none` casse le feel premium sur tous les devices touch/hybrid
- **Observation** : ligne 2164-2165 force `cursor:none` sur `body` et sur `a, button, .pcard, .vcard, .crow`. Un curseur custom `.cur-dot` + `.cur-ring` est positionné en JS. Sur écrans touch ou sur un trackpad qui masque le curseur à l'arrêt, il y a des ratés connus (ring fantôme au load, décalage de quelques frames au hover-change, vanilla JS qui ne synchronise pas avec les transitions CSS). Ligne 2263 désactive ces éléments sur ≤ 900px, mais **la règle `cursor:none` sur tous les children `a/button` reste active**. Sur iPad ou sur trackpad macOS en mode touch (qui arrive en 2026), ça produit un clic sans feedback visuel.
- **Impact** : perception premium fragile. Un curseur custom est un choix qui doit être *parfait* — sinon il sabote le message. Aujourd'hui il est juste « présent ». Sur l'audit client (présentation devant un recruteur qui se penche sur l'écran en mode observer-mouse-only), le ring a un lag perceptible.
- **Fix concret** : soit on assume à 100 % (ajouter `hover:hover) and (pointer:fine)` comme gate media-query autour de toute la logique cursor, fixer le lag JS en passant sur `transform` + `will-change` strict, et **tester à 60 FPS sur un MacBook M1 sans dev tools**), soit on supprime. Recommandation : **supprimer** (virer lignes 2164-2186 + les divs `.cur-dot` `.cur-ring` lignes 2550-2551). L'identité éditoriale ne dépend pas d'un curseur custom, et le gain en robustesse vaut la perte.
- **Effort** : S (si suppression) / M (si hardening).

### D-05 [P1] — Trois overlays grain superposés (.grain + #film-grain + possible gridbg)
- **Observation** : trois systèmes coexistent : 
  - `.grain` ligne 195-200 : SVG data-URL feTurbulence, opacity `.40`, mix-blend-mode screen, animation flicker 0.12s.
  - `#film-grain` ligne 2141-2161 : SVG `feTurbulence` externe, opacity `.045`, mix-blend-mode overlay, animation grainShift 0.08s.
  - `.gridbg` ligne 70-79 : quadrillage de fond 96px à opacité .6.
- **Impact** : sur mobile le JS désactive `.grain` et `.blobs` (ligne 2232-2233), mais `#film-grain` reste actif. Résultat : deux grains qui tournent en boucle. Sur écran Retina cela crée un moirage subtil quand on scrolle vite, et c'est **surtout** un coût GPU inutile (deux `feTurbulence` + animation rapide = repaints). Esthétiquement, deux grains qui ne s'accordent pas en fréquence (0.72 vs 0.85) produisent un bruit « sale » plus qu'un grain cinéma propre.
- **Fix concret** : garder un seul grain. Recommandation : `#film-grain` (opacity `.045`, overlay) — plus subtil, plus cinéma. Supprimer `.grain` (ligne 194-208) et son div (ligne 2556). Maintenir le gate `prefers-reduced-motion` déjà en place (2249).
- **Effort** : S.

### D-06 [P1] — Rythme vertical inter-sections : 4 densités qui ne convergent pas
- **Observation** : la page alterne :
  - `.section.wrap` — `padding:96px var(--page-pad) 80px` (ligne 531-532)
  - `.ps-ed` — `padding:80px 0` (674)
  - `.ps-block + .ps-block` — `margin-top:140px` (override ligne 2451, sinon 96px ligne 600)
  - `.midcta-band` — `padding:100px var(--page-pad) ; margin:96px 0 0` (2326)
  - `.quote-band` — `padding:120px var(--page-pad)` (2390)
  - `.proof-band` — `padding:28px 0` (2296) — **2× plus dense** que le reste par design (ok)
  - `.sub-block` — `margin-top:72px` (1942)
  - `.section-sub` — `margin-top:120px` (2317)
  - `.apropos-stats` — `margin-top:18px` (2068)
  Sur mobile ligne 1308 : `.section, .section.wrap` tombent à `padding:64px var(--page-pad) 44px`, mais `.midcta-band` garde son propre breakpoint (2355) à `56px`, et `.quote-band` (2403) à `64px`. Pas de commune mesure.
- **Impact** : en lecture fluide mobile, les transitions section-à-section se ressentent irrégulières. On a l'impression que la maquette a été faite par composant plutôt que par page. Un éditorial papier joue sur **un seul rythme + exceptions justifiées** (bandes = plus grandes, index = plus petits). Ici l'équipe a multiplié les valeurs ad-hoc.
- **Fix concret** : définir 3 tokens d'espacement dans `:root` :
  ```
  --gap-section: clamp(64px, 9vw, 120px);
  --gap-band:    clamp(56px, 8vw, 100px);
  --gap-sub:     clamp(40px, 5vw, 72px);
  ```
  Remplacer partout les valeurs dures par ces tokens. Forcer `.section + .section` `margin-top` nul (padding interne suffit). Les bandes (`midcta`, `quote`) utilisent `--gap-band` à 100 %. Les sous-blocs (`sub-block`, `section-sub`) utilisent `--gap-sub`.
- **Effort** : M (ajout 3 vars + remplacement méthodique).

### D-07 [P1] — Distinction h2 vs h3 ambiguë dans Projets
- **Observation** : `.section-head h2` (ligne 544-548) pèse `clamp(96px, 16vw, 240px)`, font-display. `.ps-ed-title` qui est un `<h3>` (ligne 2662, 2711, 2760, 2809) pèse `clamp(4.5rem, 10vw, 10rem)` = ~72 à 160px (lignes 687-691). Ratio desktop : 240/160 = 1.5× seulement. Ratio mobile (13vw vs 13vw) : quasi-identique. Les « Projets » (h2) et « Les filles de Wenda » (h3) ont des **poids visuels comparables en 13-16vw**.
- **Impact** : la règle « 5 niveaux distinguables en 1s » dit que h2 doit manifestement dominer h3. Ici, sur mobile spécifiquement, le titre de section et les titres de case study sont quasi-interchangeables. Le visiteur perd la trace du « où suis-je ? ».
- **Fix concret** : baisser `.ps-ed-title` à `clamp(40px, 8vw, 120px)` (lignes 689, 747) et laisser `h2` de section à son clamp actuel. Augmenter aussi le `.section-head h2` mobile (ligne 1318) de `clamp(38px, 13vw, 64px)` à `clamp(44px, 14vw, 80px)` pour reprendre de l'air. Ratio cible : h2 ≥ 1.8× h3 à tous les viewports.
- **Effort** : S (2-3 edits).

### D-08 [P1] — Portrait `assets/portrait.png` ne parle pas avec son cadre `.apropos-photo`
- **Observation** : HTML ligne 2955-2957 → `<img src="assets/portrait.png" alt="" />` dans `.apropos-photo` (ligne 2407-2414). CSS : `aspect-ratio:3/4`, `object-fit:cover`, `object-position:center top`, `filter:grayscale(.08)`. Et une carte ID `apropos-card--float` (ligne 2415) est positionnée en absolu `top:-24px;right:-24px`. L'image a `alt=""` (décorative) et le `.apropos-visual` est marqué `aria-hidden="true"` (ligne 2954). L'image perd son statut narratif — mais en plus, la carte flottante recouvre 210×280px du coin haut-droit **du portrait**, donc cache du visage potentiellement.
- **Impact** : recruteur qui va sur §05 « À propos » voit un portrait mais sans point focal clair, avec un rectangle texte qui flotte dessus. On ne sait pas si c'est un overlay éditorial délibéré (magazine) ou une glitch. Le `filter:grayscale(.08)` est un choix de style mais si la source a déjà été graded, il écrase les tons chair. Plus : `aria-hidden` en coupe l'accès pour tout utilisateur de screen reader et la photo devient purement ornementale — contraire à ce qu'un portfolio devrait faire (je suis cette personne, j'assume mon visage).
- **Fix concret** : (a) supprimer `aria-hidden="true"` ligne 2954, ajouter un `alt="Portrait de Corentin Devillé, vidéaste, en studio."` au `<img>`. (b) Positionner la card flottante en **bas-droite** (`bottom:-24px;right:-24px` ligne 2416) plutôt qu'en haut, pour libérer le visage. (c) Monter le `aspect-ratio` à `4/5` pour un cadrage plus portrait magazine (ligne 2408). (d) Retirer le `filter:grayscale` ligne 2413 — ou le pousser à 1 (noir & blanc assumé) ; `.08` est une demi-mesure qui pollue. Cohérent avec identité éditoriale → choisir le N&B franc.
- **Effort** : S.

### D-09 [P1] — Hero mobile : `font-size: clamp(36px, 13vw, 180px)` coupe trop tôt à 390px
- **Observation** : `.hero-name` ligne 435 : `font-size: clamp(36px, 13vw, 180px)`. À 390px de viewport, 13vw = 50.7px. Le `min` clamp (36px) ne se déclenche que sous 277px de viewport — donc il ne sert à rien sur devices modernes. Mais 50.7px pour un nom-titre Dela Gothic One qui est censé écraser le fold mobile, c'est **timide**. Par comparaison, `.section-head h2` mobile (ligne 1318) est à `clamp(38px, 13vw, 64px)` — donc le titre de section est à 50.7px aussi. **Le nom de l'auteur est de la même taille que le h2 de section.**
- **Impact** : sur mobile, le hero ne remplit pas le fold comme il le devrait. Le nom se lit, oui, mais il ne transmet pas l'autorité. L'effet magazine « couverture monumentale » s'effondre.
- **Fix concret** : passer à `clamp(48px, 16vw, 200px)` ligne 435. À 390px → 62.4px. Ligne 1252 (mobile fix hero) : aucune modif nécessaire, le clamp porte. Vérifier que `.hero-name .word` avec `white-space:nowrap` (ligne 1253, 2223) n'entraîne pas d'overflow horizontal — tester à 320px (iPhone SE 1st gen encore utilisé). Si débordement, ajouter `word-break:break-word` en fallback.
- **Effort** : S.

### D-10 [P1] — `.quote-band` : `::first-letter` en accent + placeholder littéral
- **Observation** : ligne 2400 → `.qb-text::first-letter{color:var(--accent)}`. HTML ligne 2939 → `<blockquote class="qb-text">«&nbsp;[Témoignage client — 1 à 2 phrases à compléter par Corentin].&nbsp;»</blockquote>`. Donc la première lettre colorée est… **le chevron `«`**, puis un non-breaking space, puis un placeholder entre crochets. Résultat : en production actuelle, l'accent décoratif tombe sur un guillemet — jamais sur une initiale narrative.
- **Impact** : la touche éditoriale « capitale ornée » est perdue. En plus, le placeholder `[Témoignage client — à compléter]` est en prod (cf. F4 du recon). Un œil averti voit tout de suite que le bloc est faux.
- **Fix concret** : (a) retirer `::first-letter` ligne 2400 et déplacer l'accent sur une grande guillemet dessinée (`.qb-text::before{content:"“";…}` en `font-family:var(--font-display);font-size:2em;color:var(--accent);` — hors flux). (b) *Hors scope design mais à tagger à l'agent 2* : remplacer le placeholder par un vrai témoignage ou masquer la section jusqu'à rédaction.
- **Effort** : S.

### D-11 [P2] — Triple bordure accent (bordure 4-6px ink-gauche/droite) crée un pattern « print » qui fatigue
- **Observation** : trois blocs utilisent le même trick de bordure colorée épaisse comme signature :
  - `.hero-reel` — `border-bottom: 4px solid var(--accent)` (493)
  - `.footbanner` — `border-right: 6px solid var(--accent)` (1007)
  - `.cp-frame` — `border-left: 6px solid var(--accent)` (1725) — section contact
  Chacun sur un côté différent, épaisseurs différentes (4 vs 6), aucune règle.
- **Impact** : c'est un tic visuel. En statique ça passe ; en défilement ça se transforme en « chaque bloc a son petit trait orange d'un côté au hasard ». Rien ne structure l'ensemble — c'est décoratif, pas identitaire.
- **Fix concret** : choisir **une seule** position (recommandation : border-left 4px uniform) et l'appliquer systématiquement à **deux** blocs max (ex: `.hero-reel` + `.cp-frame`). Supprimer la bordure de `.footbanner` ligne 1007. Uniformiser l'épaisseur à 4px.
- **Effort** : S.

### D-12 [P2] — Hover `.pcard` underline 6px : trop épais pour un portfolio éditorial
- **Observation** : ligne 567-568 → `.pcard .lbl h3 span{background-image:linear-gradient(var(--accent),var(--accent));background-size:0% 6px;…}` animation de soulignement accent 6px d'épaisseur. À 42px de font-size (max clamp), 6px = 14 % de la hauteur de capitale. C'est un soulignement « gras » pour une identité qui se réclame sobre.
- **Impact** : sur l'ancien pattern `.pgrid` (non utilisé dans la v2 courante mais le CSS est toujours là, mort en live, vif à la ressuscitation). Si on revient aux `pcard`, l'underline 6px vermillon fait « fin d'année 2015 » plus que « édition 2026 ».
- **Fix concret** : passer à 2px ink (`linear-gradient(var(--ink), var(--ink))`, `background-size:0% 2px`). Alternative plus éditoriale : pas d'underline, rotation du `.arr` de 45° + couleur ink→accent. Vérifier si le CSS `.pcard` est encore appelé dans le HTML v2 (grep `class="pcard` ne revient que dans définitions) — **sinon supprimer tout le bloc ligne 557-597** comme dead code.
- **Effort** : S (si kill).

### D-13 [P2] — `.rec-dot-solo` animé rouge `#E43D12` : couleur non déclarée dans le système
- **Observation** : ligne 358 → `background:#E43D12;flex-shrink:0;animation:recBlink 2.4s ease-in-out infinite;box-shadow:0 0 0 4px rgba(228,61,18,.18)`. Le `#E43D12` est un rouge plus sombre que `--accent: #FF4F00`. Il apparaît sans être défini dans `:root`. Le `Tweaks` panel propose « Ember #E43D12 » comme variante d'accent (ligne 3060), mais il est hardcodé dans le dot topbar.
- **Impact** : incohérence de couleur pure. Un visiteur attentif qui compare le dot topbar et un triangle `▶` de section voit deux rouges légèrement différents. En dark mode, aucun override — le dot reste rouge froid.
- **Fix concret** : soit unifier sur `var(--accent)` (ligne 358), soit ajouter `--accent-deep:#E43D12` dans `:root` (ligne 20) et l'utiliser. Idem ligne 522 (`.playtag .dot` utilise déjà `--accent` — cohérent). Privilégier option 1 (l'ink + un accent unique).
- **Effort** : S.

### D-14 [P2] — Morph hero `Je suis — Vidéaste` : `min-width:10ch` crée un trou visuel
- **Observation** : lignes 2209-2219 (et HTML ligne 2589). `.morph-word` avec `min-width:10ch` pour stabiliser le layout pendant le morphing. Sur desktop, la plupart des rôles cyclés (Vidéaste, Monteur, Formateur, Tourneur?) font 7-10 caractères. Avec `min-width:10ch`, le label « Vidéaste » (8 chars) laisse 2ch de vide à droite. Sur mobile, `.morph-word` est remplacé par `min-width:8ch` (ligne 1256), même effet.
- **Impact** : œil perçoit un espace bizarre derrière le mot, surtout quand il morphe vers un mot plus court. Mineur mais signale que le composant n'est pas fini.
- **Fix concret** : supprimer `min-width`, remplacer par un wrapper flex avec `min-width:max(…)` calculé dynamiquement en JS au load (on mesure chaque mot, on prend le plus large, on fixe en px). Fallback CSS pur : viser `min-width:9ch` ET `text-align:left` + inline-block pour que le trou soit **après** le mot et intègre visuellement mieux. Recommandation no-JS : `min-width:9ch` (au lieu de 10), ça réduit le trou sur « Vidéaste ».
- **Effort** : S.

### D-15 [P2] — `.avail-badge` vert `#2A7F5C` : cohérence avec le Contact vert ?
- **Observation** : lignes 307-324 définissent un badge « Available for work » en vert `#2A7F5C` avec dot pulsant. **Mais le badge n'apparaît pas dans le HTML v2** (aucune occurrence dans les lignes 2558-3040). Cependant une variante `.cp-avail` (ligne 1795) réutilise le même lexique dot-vert dans la section Contact verte.
- **Impact** : code mort élégant, mais si `.avail-badge` est ressuscité, il entrerait en conflit avec le vert contact + le vert WhatsApp icon. Trois verts différents dans la même page (#2A7F5C, #1F5F42, #25D366). Soit on mutualise, soit on supprime.
- **Fix concret** : (a) si `.avail-badge` est mort, supprimer lignes 307-330. (b) Sinon l'utiliser dans le hero (remettre ligne 2582 après le h1) comme signal « dispo 2026 », mais alors aligner **tous** les verts sur `--accent-ok: #2A7F5C` déclaré `:root`. À trancher côté direction éditoriale (pas ici).
- **Effort** : S (suppression) / M (standardisation).

### D-16 [P2] — `.tweaks` panel en prod : baisse la perception premium
- **Observation** : ligne 1034-1055 + HTML 3042-3079. Panneau de réglages flottant en bas-droite (thème / mode / accent / grille). Il est `display:none` par défaut et activé par `.on`. Risque : s'il est accessible par trigger (keyboard shortcut? scroll magic?), un visiteur peut le voir. Un recruteur qui voit « Tweaks — Ink / Ember / Moss / Ultramarine » comprend immédiatement que le site a été bricolé à plat plutôt que design-décidé.
- **Impact** : mineur si bien caché, critique si exposé. Signal « WIP » en direct.
- **Fix concret** : soit le panneau est strictement dev-only → le gater sur `localhost` ou derrière un flag `?tweaks=1` dans l'URL ; soit le supprimer du HTML livré (ligne 3042-3079) et garder les overrides CSS comme théorie. Recommandation : retirer le bloc HTML en prod, conserver le CSS (pour variantes futures).
- **Effort** : S.

### D-17 [P2] — Double spec-badge « 4K / 60fps » dans topbar est un détail techno-bro
- **Observation** : HTML 2572-2575 → `<span class="spec-badge">4K</span><span class="spec-badge">60fps</span>` dans la topbar, en face des liens nav. Ligne 351-355 : petits badges mono avec bordure. Affichés seulement desktop (≥ 1100px) ligne 356.
- **Impact** : dans un portfolio qui se veut éditorial-sobre, afficher 4K/60fps dans la topbar est l'exact opposé du positionnement. Ça dit « vidéaste techno » pas « vidéaste narrateur ». Positionne CD comme wedding-camera, pas comme auteur.
- **Fix concret** : supprimer ces deux spans (HTML ligne 2572-2575) + la règle CSS `.spec-badge` (ligne 351-355) + `.topbar-spec` (350). Si on veut un signal stat dans la topbar, remplacer par **une** information narrative (ex: `<span class="spec-badge">Éd. 01 / 2026</span>` — cohérent avec le hero-eyebrow ligne 2584).
- **Effort** : S.

### D-18 [P2] — Letter-spacing du display Dela Gothic One incohérent (5 valeurs)
- **Observation** : Dela Gothic One a des letter-spacings différents d'un titre à l'autre :
  - `.hero-name` : `-.015em` (435)
  - `.section-head h2` : `-.035em` (546)
  - `.ps-ed-title` : `-.025em` (689)
  - `.ps-title` (ancien) : `-.015em` (802)
  - `.midcta-title` : `-.02em` (2343)
  - `.qb-text` : `-.015em` (2397)
  - `.apropos-quote` : `-.015em` (2060)
  - `.apropos-card-initials` : `-.04em` (2045)
  - `.mob-menu nav a` : `-.02em` (1122)
- **Impact** : à l'œil expert, certains titres « respirent » et d'autres sont serrés. Le grand `h2` à `-.035em` est trop serré à 240px (les glyphes se chevauchent presque sur certains couples comme `ri`, `ro`). Dela Gothic a un design déjà tight — `-.035em` est agressif.
- **Fix concret** : règle simple : `-.02em` pour tous les titres ≤ 80px, `-.015em` pour tous les titres > 80px (paradoxalement Dela a besoin de plus d'air en grand). Réécrire les 8 valeurs ci-dessus à cette règle. La `.apropos-card-initials` à -.04em peut rester (cas spécial : 168px monolithique « CD »).
- **Effort** : S.

### D-19 [P2] — `.proof-band` logos en texte display : crédibilité limitée
- **Observation** : ligne 2637-2647. La bande défilante « clients » contient `<span>Wenda</span><span>Bebooth</span>…<span>[Client 05]</span>…<span>[Client 08]</span>` rendus en `var(--font-display)` 22-34px avec opacity .55. Pas de logos SVG, pas de contraste, les placeholders `[Client 05-08]` apparaissent visuellement. Noté aussi dans F4 du recon.
- **Impact** : une bande clients est une **preuve de confiance visuelle**. En texte, c'est aussi crédible qu'une liste Wikipedia. Les placeholders en crochets confirment qu'on est en démo. L'intention est la bonne (signal social proof) mais l'exécution tue l'effet.
- **Fix concret** : (a) si pas de logos disponibles, masquer la proof-band (display:none) jusqu'à avoir ≥ 4 logos SVG réels. (b) Si on garde la version texte, au moins supprimer les 4 placeholders (lignes 2641-2645) et dupliquer `Wenda · Bebooth · Scout · Lucid` autant de fois que nécessaire pour remplir le track. (c) Ajouter une ligne de hiérarchie : la `.proof-label` « § Clients — Sélection » en `var(--accent)` pour marquer la bande comme légende forte (ligne 2300-2305 → changer `color:var(--muted)` en `color:var(--accent)` avec `font-weight:600`).
- **Effort** : S.

### D-20 [P2] — Focus visible `:focus-visible` limité à un outline + border-radius 2px
- **Observation** : ligne 2253-2254 → `:focus-visible{outline:2px solid var(--accent);outline-offset:3px;border-radius:2px}`. L'outline 2px en `var(--accent)` (vermillon) sur le fond sable `#F0EEE9` donne un contraste de **~4.8:1** (correct AA) mais sur le fond vert contact `#1F5F42`, le même vermillon sur vert donne **~2.9:1** (sous AA). Et les `crow/prow` avec leur `cursor:none` (qui masque le curseur) combinés avec le focus outline deviennent la *seule* indication visuelle — or le border-radius 2px n'a pas de sens pour des blocs qui eux-mêmes n'ont pas de radius.
- **Impact** : contraste AA cassé en section Contact (lié à D-01). Indicateur visuel de focus pas optimisé pour clavier-only users ni pour démos.
- **Fix concret** : (a) si D-01 est appliqué (Contact sur ink), les 2px accent sur ink donnent **~5.7:1** → AA OK. (b) augmenter outline à 3px pour des boutons/CTA principaux (ex: `.contact-wa-btn:focus-visible` override) où le retour clavier doit être franc. (c) supprimer `border-radius:2px` du focus par défaut — il fait doubler les bords sur les éléments rectangulaires.
- **Effort** : S.

---

## Recommandations globales

1. **Convergence vers un seul dialecte éditorial.** Aujourd'hui le site parle 3 dialectes : sobre-sable en Hero / Projets / Services / Méthode / Tarifs / À propos ; radical-WhatsApp-vert en Contact ; ink-bandeau en `.midcta-band` / `.mob-menu`. On a deux moodboards dans une même page. Trancher : 85 % sable + 15 % ink (pour les bandes de respiration) + 0 % vert (le WhatsApp reste un icon-accent, pas un fond).

2. **Tokeniser le vocabulaire.** Six types de valeurs actuellement hardcodées devraient vivre dans `:root` : `--gap-section`, `--gap-band`, `--gap-sub`, `--eyebrow-size`, `--eyebrow-spacing`, `--rule-weight`. Ça force la cohérence et facilite chaque futur ajustement. Le site est single-file no-build — parfait pour des CSS vars propres.

3. **Règle des 3 accents max par écran.** Définir une doctrine : par fold visible (≤ 1 viewport), on autorise **3 apparitions** de `var(--accent)` maximum. Aujourd'hui certaines sections en ont 6-8 (ex: Services = numéros + lignes + hover tags + link underlines = 5-7 touches). Forcer la discipline par audit visuel fold-par-fold.

4. **Supprimer ce qui a l'air d'une feature et qui n'en est pas.** Curseur custom, double grain, film-loader 2 secondes, tweaks panel, morph hero, eye-bubble split-hover, easter egg `.cdv-egg`. Chacun pris isolément est un choix défendable. Accumulés, ils signalent « un dev qui n'a pas su trancher ». Garder au maximum **2** micro-signatures (recommandation : film-loader + eye-bubble, les deux liés au logo-oeil). Supprimer les autres ou les gater derrière un easter egg maîtrisé.

5. **Mobile-first rigoureux, pas mobile-ajusté.** De nombreux overrides mobile (1187-1372, 2221-2289) ressemblent à des patchs ajoutés au fil des tests. Rédiger le CSS **en partant de 390px** et ajouter les media-queries `min-width` progressivement. Ça simplifie la cascade et évite les `!important` (on en compte 14 dans le fichier, ex: 2232-2240, 2249-2250).

---

## 3 références de portfolios exemplaires

- **[Robin Mastromarino — robinmastromarino.com](https://robinmastromarino.com/)** — Dir. créatif FR. Structure 100 % sobre-éditoriale (sable/ink uniquement, zéro couleur accent), typographie serif + mono comme seul système, scroll qui respire à 120-160px entre chaque bloc. Ce que CD peut capter : **la discipline du mono-accent (ici absent)** et l'emploi d'une seule règle de rythme verticale.

- **[Studio Feixen — feixen.ch](https://www.feixen.ch/)** — Studio suisse dir. par Felix Pfäffli. Identité magazine-print agressive, palette restreinte par projet, numérotation §§ utilisée **sérieusement** (01/02/03 comme véritable index de travaux, pas comme ornement). Ce que CD peut capter : **les eyebrows qui signalent la fonction narrative** plutôt que la décoration. La typographie s'efface pour laisser parler les projets.

- **[Locomotive — locomotive.ca](https://locomotive.ca/)** — Agence MTL. Excellent cas d'étude pour un vidéaste : grands titres monumentaux, vidéos pleine largeur, carousels discrets, mais **un seul accent** (bleu électrique) utilisé ~4 fois par page max. Ce que CD peut capter : **comment une identité peut avoir 1 accent fort sans le noyer** — chaque apparition compte, chaque apparition a un rôle (CTA principal, puce d'index, indicateur de progression).

---

## Ce qui marche DÉJÀ et qu'il faut protéger

1. **Le triptyque typographique Dela Gothic One / Inter / JetBrains Mono est cohérent** et ambitieux. Le display est distinctif (Dela Gothic a une personnalité — pas un Inter-XXL de plus), le mono JetBrains est reconnaissable mais pas cliché-dev. Ne pas migrer vers un Neue Haas ou un GT America par « sécurité ». C'est la signature la plus forte du site.

2. **Le morph du `.hero-name` lettre-par-lettre avec `.word.overflow:hidden` + `.char.translateY(110%)`** (lignes 439, 458-468) est bien exécuté et sert l'identité sans artifice. À conserver tel quel. Seule modif tolérable : le timing de 3500ms post-film-loader (JS ligne 3097) — ne pas allonger.

3. **Le système d'entry animation `.reveal` avec IntersectionObserver et 3 delays (d1/d2/d3)** (ligne 1026-1031) est sobre, performant, réutilisable. À conserver. Ne pas le transformer en scroll-driven-animation CSS (trop de support-navigateur variable en 2026 encore).

4. **Les `.method-cell::before` — trait accent 2px × 48px en top-left de chaque cellule** (ligne 1974-1976) est un bon exemple de ponctuation éditoriale juste. À protéger et à étendre (cf D-03 : faire porter ce trait par toutes les eyebrows de niveau B).

5. **Le `.ps-ed-meta` avec séparateur `·` accent entre spans** (lignes 711-712, 2439-2442) — bonne lecture de metadata magazine. Même combat : protéger ET répliquer.

6. **L'architecture full-bleed via 3 bandes éditoriales (proof / midcta / quote)** est une structure pertinente pour rythmer le scroll. Le ratio 6 sections + 3 bandes = 1 bande toutes les 2 sections ≈ narrativement juste. Préserver cette cadence même quand on corrigera le contenu des placeholders.

7. **`prefers-reduced-motion` respecté** (ligne 2242-2250) : le site désactive correctement animations/transitions pour les utilisateurs concernés. Rare, précieux, ne pas oublier de maintenir ce gate à chaque nouvelle animation ajoutée.

8. **Le responsive topbar** (ligne 1192-1248) est bien pensé : brand-dot + name collé en haut-gauche sur mobile, hamburger 36×36 coin haut-droit, fond transparent → verre dépoli au scroll. Sobre et moderne. Ne pas transformer ça en « burger menu coloré ».

Fin de l'audit design.
