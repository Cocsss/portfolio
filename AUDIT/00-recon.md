# AUDIT / 00 — Reconnaissance

> Phase préliminaire de l'orchestrateur. Read-only, zéro modification.
> Commit base : `ce948e5` (main). Branche audit : `audit/full-refonte-2026`.
> Date : 2026-04-22.

---

## 1. Stack détecté

| Élément | Valeur |
|---|---|
| Type | **Static site — vanilla HTML/CSS/JS, single-file** |
| Framework | Aucun (pas de React/Astro/Next/Vue) |
| Build | Aucun (pas de bundler, pas de CI build, pas de package.json) |
| Fichier principal | `index.html` — 3498 lignes, 155 Ko |
| CSS | Inline `<style>` lignes 19 → 2457 (2438 lignes) |
| JS | Inline `<script>` à la fin (~450 lignes) |
| Fonts | Google Fonts — 3 familles : `Dela Gothic One`, `Inter` (300/400/500/600), `JetBrains Mono` (400/500) |
| Hébergement | Cloudflare Pages (site) + Cloudflare R2 (vidéos via `videos.corentindeville.com`) |
| Domaine | `corentindeville.com` |
| Déploiement | Auto sur `git push origin main` |
| Images hors-vidéo | `assets/portrait.png` (2.37 Mo — le SEUL raster statique ; le reste est SVG inline ou posters vidéo R2) |
| Écosystème de fichiers | Plusieurs backups `index copie*.html` (3 variantes) + `portfolio (2).html` (référence) — tous gitignored |

---

## 2. Sections actuelles (dans l'ordre du DOM)

| Ligne | Élément | ID / classe | Rôle présumé |
|---|---|---|---|
| 2558 | `<header class="topbar">` | — | Nav sticky avec 6 entrées (Projets / Services / Méthode / Tarifs / À propos / Contact) + toggle thème |
| 2482 | `<nav class="mob-menu">` | `mob-menu` | Menu mobile plein écran, mêmes 6 entrées |
| 2582 | `<section class="hero wrap">` | `#top` | Nom CD + morph job + meta + showreel vidéo autoplay + ticker (désormais intérieur) + scroll hint |
| 2637 | `<section class="proof-band">` | — | Bandeau full-bleed défilant — logos clients en texte display |
| 2650 | `<section class="section wrap">` | `#projets` | §01 — 4 case studies (Wenda / Bebooth / Lucid Dreams / Scout) + Selected Work Index 6 lignes placeholder |
| 2845 | `<section class="midcta-band">` | — | Full-bleed noir, titre monument + bouton WhatsApp accent |
| 2855 | `<section class="section wrap">` | `#services` | §02 — 2 cartes services (Vidéo / Formation) |
| 2883 | `<section class="section wrap">` | `#processus` | §03 — 4 étapes méthode (Briefing / Tournage / Montage / Livraison) |
| 2914 | `<section class="section wrap">` | `#tarifs` | §04 — 3 lignes tarifs (Montage 20€ / Tournage 25€ / Formation 20€) |
| 2937 | `<section class="quote-band">` | — | Full-bleed centré, citation témoignage (placeholder) |
| 2945 | `<section class="section wrap">` | `#apropos` | §05 — Photo + carte ID-CD flottante + quote + body + 3 stats (40+ / 04 / BE) |
| 2989 | `<section class="section wrap">` | `#contact` | §06 — CTA WhatsApp principal + liens mail / Instagram / YouTube, single-column centré |
| 3030 | `<footer class="site-footer">` | — | Liens sociaux + bouton « Haut ↑ » |
| 3036 | `<a class="mwa-fab">` | — | Bouton flottant WhatsApp mobile (FAB) |

**Total** : 6 sections numérotées + 3 bandes éditoriales (proof / midcta / quote) + hero + footer = 11 blocs scroll.

### Hiérarchie Hn détectée (brute)

- 1 × `h1` : `.hero-name` (Corentin Devillé, bonne hygiène)
- 7 × `h2` : Projets / Services / Méthode / Tarifs / À propos / Contact (les 6 sections) + `midcta-title` « Un projet en tête ? »
- 6 × `h3` : 4 titres projet + « Autres travaux » (Selected Work) + 2 cartes service
- 1 × `h4` : `.tweaks h4` (panneau de réglages thème ?)

> **Flag** : la bande mid-CTA utilise `<h2>` alors qu'elle n'est pas une section principale structurante — double emploi hiérarchique à trancher.

---

## 3. Cinq points de friction évidents à l'œil nu

Ce sont les choses qu'un senior repère en < 30s de scroll du code et du live.

### F1 — Aucune preuve de résultat dans les case studies
Les 4 projets ont un bloc descriptif mais **zéro métrique, zéro livrable chiffré, zéro rôle explicite**. Placeholders `[à compléter]` sur 3 des 4 projets (Bebooth, Lucid Dreams, Scout). Même Wenda (le seul texte rédigé) n'a pas de structure problème → rôle → décisions → impact. **Un recruteur ne peut rien retenir.**

### F2 — Poids du portrait PNG : 2.37 Mo pour un asset qui sert d'accent dans À propos
Le portrait actuel est un PNG transparent 1400×2100 non converti en AVIF/WebP, servi sans `srcset`/`sizes` et sans `loading="lazy"`. Sur mobile 4G, c'est ~2s de téléchargement tout seul — il est hors-fold (§05), donc éligible lazy.

### F3 — Fichier monolithique de 155 Ko, 3500 lignes, zéro séparation
Tout vit dans `index.html` : CSS + JS + HTML. Pas un péché en soi pour un one-pager, mais :
- Aucune `data-*` structure pour les projets → ajouter un projet = dupliquer ~60 lignes de HTML
- Le CSS n'est pas minifié (envoyé en clair à chaque visite)
- Les 3 backups `index copie*.html` prouvent qu'on édite ce fichier à la main, à l'aveugle

### F4 — Preuve sociale inexistante (logos, témoignage, stats)
- `.proof-band` contient 4 noms réels (Wenda, Bebooth, Scout, Lucid) + 4 placeholders `[Client 05]`→`[Client 08]` en **texte display**, pas de logos SVG
- `.quote-band` a un placeholder littéral `[Témoignage client — à compléter]`
- Les 3 stats À propos (40+ / 04 / BE) sont opaques — que signifie « 04 ans » ? Projets livrés = 40+ est crédible mais non vérifié

### F5 — Numérotation §01/§02… inconsistante inter-sections
La nav liste 6 entrées numérotées, mais :
- La bande mid-CTA a son propre label `§ Passons à l'action` (sans numéro) qui rompt la séquence
- La bande quote-band n'a pas de label du tout
- Les `ps-idx` dans Projets utilisent un sous-niveau `§ 01.1 / §01.2…` (cohérent) mais **Bebooth a `§ 01.3 / [Catégorie — à compléter]`** (placeholder visible en prod)
- La proof-band a `§ Clients — Sélection` (sans numéro) — même traitement que midcta-band, ok mais non documenté

---

## 4. Arbre des composants clés (logique, pas physique)

Le site est single-file, donc il n'y a pas de composants au sens React. J'extrais les **primitives CSS réutilisables** effectivement utilisées dans le HTML :

```
PRIMITIVES DE LAYOUT
├── .section.wrap            — section standard (padding page + max-width)
├── .section-head            — en-tête section (.idx + h2 + .sub)
├── .wrap                    — max-width container
└── .reveal                  — classe d'animation scroll (IntersectionObserver)

PRIMITIVES ÉDITORIALES (bandes full-bleed)
├── .proof-band              — scroller horizontal
├── .midcta-band             — CTA noir pleine largeur
└── .quote-band              — témoignage centré

PRIMITIVES DE PROJETS
├── .ps-ed                   — wrapper d'un case study éditorial
│   ├── .ps-ed-header        — eyebrow + title
│   ├── .ps-ed-meta          — métadonnées inline
│   ├── .ps-ed-desc          — paragraphes description
│   └── .ps-ed-media         — conteneur vidéo/image
├── .ps-idx                  — numérotation projet
├── .ps-divider              — hidden (display:none via refonte v2)
└── .ps-carousel             — carousel 3D (défini en CSS, peu utilisé)

SELECTED WORK INDEX
├── .plist                   — conteneur liste
└── .prow (.r1…r6)           — ligne projet (num / name / cat / preview / arrow)

TARIFS (nouveau v2)
├── .tariffs-edit
├── .tr-row
├── .tr-left / .tr-right
└── .tr-num / .tr-name / .tr-price / .tr-unit

À PROPOS
├── .apropos-grid            — layout 5fr/6fr
├── .apropos-visual          — col visuelle (photo + carte)
├── .apropos-photo           — conteneur photo
├── .apropos-card            — carte ID-CD
│   └── .apropos-card--float — variante flottante sur photo
└── .apropos-stats           — 3 stats

CONTACT
├── .contact-grid--simple    — layout single-col v2
├── .contact-wa-btn          — CTA WhatsApp principal
└── .contact-link            — lien social secondaire

HERO
├── .hero-name               — h1 animé char-par-char (.word / .char)
├── .hero-star               — astérisque accent SVG
├── .hero-morph              — « Je suis — [job] » morphing
├── .hero-meta               — 3 colonnes meta
├── .hero-reel               — conteneur showreel vidéo
└── .hero-scroll-hint        — indicateur scroll

NAVIGATION
├── .topbar                  — header sticky
├── .section-nav             — liens nav desktop
├── .menu-btn                — hamburger mobile
├── .mob-menu                — overlay mobile
└── .mm-link                 — lien mobile numéroté

ANIMATIONS / UI UTIL
├── .film-loader             — intro animée C→D→œil (~2s)
├── .eye-bubble              — logo œil CD split-hover
├── .ticker-wrap             — défileur (.ticker-track + .sep)
│   └── .ticker--in-hero     — modifier (refonte v2)
├── .stripes                 — lignes décoratives sur reel
├── .playtag                 — badge « Showreel 2026 » sur reel
└── .mwa-fab                 — bouton flottant WhatsApp mobile
```

**Couplage CSS/HTML** : ~70 classes de layout/composant ; ~20 % semblent définies sans être utilisées dans le HTML actuel (dead CSS à confirmer par l'agent 4).

---

## 5. JS inline — capacités actives

Observé au passage dans le `<script>` final :

- IntersectionObserver `.reveal` → animation d'entrée
- Compteur numérique `.stats .n, .apropos-stats .n` (count-up)
- Scrollspy dynamique (lit `nav a[data-target]`)
- Morphing du job title hero
- Film-loader intro (séquence C/D/œil)
- Toggle thème (light/dark via `[data-theme]`)
- Eye-bubble split-on-hover

**Aucune dépendance externe JS.** Tout est vanilla. Bonne note préventive.

---

## 6. Email obfusqué Cloudflare

Les `href="mailto:…"` sont remplacés par des liens `/cdn-cgi/l/email-protection#…` et décodés par un script Cloudflare servi depuis `/cdn-cgi/scripts/5c5dd728/cloudflare-static/email-decode.min.js`. Ça fonctionne, mais ça **bloque le crawl SEO de l'adresse** et ajoute un script externe. À évaluer par l'agent perf.

---

## 7. Ce que l'orchestrateur sait déjà des commits récents

| Commit | Sujet | Note |
|---|---|---|
| `ce948e5` | refactor(structure): full editorial overhaul — 6-section flow with proof/quote/midcta bands | Refonte v2 qui vient d'être poussée. Base de l'audit. |
| `1843d49` | fix(hero): close stray wrapper div | Équilibrage div. |
| `6ab6579` | refactor(structure): promote About/Tarifs/Process out of Infos accordion | Étape précédente. |
| `119016c` | feat(contact): replace portrait with cutout + tune CSS for transparency | Portrait découpé. |
| `dfc2be3` | style(contact): swap colored aura for neutral modern grid mat | Style grille. |

---

## 8. Ce que les 4 agents vont maintenant regarder

- **Agent 1 — Design** : typographie, rythme, palette, contraste, cohérence visuelle, signature. Base : tout le bloc CSS 19-2457 + inspection du HTML rendu.
- **Agent 2 — Structure/IA** : narration, ordre, nommage, case studies, fil. Base : structure DOM 2558-3040 + contenu textuel.
- **Agent 3 — Perf/A11y/SEO** : poids, fonts, images, landmarks, alt, focus, head, schema. Base : `<head>` 2-18, assets, scripts, CSS critique.
- **Agent 4 — Code** : monolithe, data-driven, dead code, duplications, maintenabilité. Base : arbre fichiers + `index.html` complet.

---

## 9. Contraintes connues à rappeler aux agents

1. **Le site doit rester no-build** : pas de Next/Vite/Astro suggérés comme « il faudrait migrer ». Améliorations incrémentales sur l'existant.
2. **Vidéos restent sur R2** — ne pas proposer d'iframe YouTube/Vimeo.
3. **Pas de dépendance JS ajoutée sans justification écrite**.
4. **Identité sobre + éditoriale** : palette contenue (bg sable `#F0EEE9`, ink noir, accent orange `--accent`). Pas de pivot « brutaliste coloré » ou « glassmorphism ».
5. **Mobile first** : chaque issue doit être testable en priorité sur viewport ≤ 390 px.
6. **Pas de refonte pour le plaisir** : chaque changement = issue identifiée.

Fin de la reconnaissance. Lancement des 4 agents en parallèle à la prochaine étape.
