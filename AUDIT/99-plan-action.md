# AUDIT / 99 — Plan d'action synthétique

> Synthèse des 4 rapports : `01-design.md` (20 issues), `02-structure.md` (11), `03-perf-a11y-seo.md` (15), `04-code.md` (12). Total brut : **58 issues**.
> Arbitré, priorisé, ordonné en commits atomiques.
> Date : 2026-04-22. Branche : `audit/full-refonte-2026`. Commit base : `ce948e5`.
>
> **⚠ Aucune ligne de code n'a été modifiée à ce stade. Ce plan doit être validé avant exécution.**

---

## Executive Summary

Trois grosses lignes de force se dégagent des 4 audits :

1. **Le site perd sa crédibilité en 10 secondes de scroll** à cause de 3-4 signaux visibles : placeholders `[à compléter]` dans 3 case studies sur 4, section Contact verte qui casse l'identité éditoriale, hero sans positionnement ni CTA, témoignage placeholder dans `.quote-band`. Ces 4 issues sont des **bloquantes crédibilité** — à traiter en premier, effort collectif **~2h**.

2. **Les Core Web Vitals et le SEO sont récupérables en 1 journée** : portrait PNG 2.37 Mo → AVIF ~90 Ko, showreel sans poster → poster AVIF pré-généré, pas de schema.org / canonical / sitemap → fichiers à créer. Gain LCP mobile **3.2→1.6 s**, Lighthouse SEO **70→92+**.

3. **La dette technique est localisée et pas étalée** : ~900 lignes de CSS mortes (12 familles orphelines de la refonte v1), ~40 lignes de JS mortes (carousel Scout qui n'existe pas + bloc wide-carousel Bebooth obsolète), 4 fichiers `index copie*.html` parasites. Cleanup : **2-3 h** pour gagner ~35 % de poids CSS et supprimer les signaux trompeurs.

**Cibles après exécution P0 + P1 :** LCP < 2.1 s, CLS < 0.02, Lighthouse Perf 90+, A11y 98+, SEO 95+, plus un portfolio **lisible, crédible et prêt à prospecter** pour mai 2026.

---

## Cross-référence : issues qui apparaissent dans ≥ 2 rapports

Ces convergences renforcent la priorité.

| Convergence | Rapports | Action unifiée |
|---|---|---|
| **Placeholders `[à compléter]` visibles en prod** | S-02 (Structure), C-03 (Code), D-19 (Design sur proof-band), D-10 (Design sur quote-band), F4 (Recon) | **Masquage systématique** tant que contenu non rédigé (P0-1) |
| **Portrait PNG poids + cadrage + alt** | PERF-01 (Perf), D-08 (Design), A11Y-05 (A11y) | **Pipeline unifié** : AVIF + srcset + alt narratif + retirer aria-hidden (P0-5) |
| **Accent `--accent` `#FF4F00` surutilisé ET contraste fail AA** | D-02 (Design 69 occurrences), A11Y-02 (contraste 3.03:1 body text) | **Réserver accent aux décorations + large text**, créer `--accent-text` foncé (P0-7) |
| **Numérotation inconsistante** | F5 (Recon), D-03 (Design eyebrows), S-08 (Structure `§` / `·`) | **Règle unique** : sections = `§ NN / titre`, bandes = pas de numéro (P1) |
| **Dead code** | C-01 + C-02 (Code), D-15 (Design `.avail-badge`), D-12 (Design `.pcard`) | **Purge** CSS + JS + HTML inutilisés (P0-8) |
| **Hero ne positionne pas + CTA absents** | S-01 (Structure), D-09 (Design taille trop timide mobile) | **Ajout pitch + 2 CTA + agrandir `.hero-name` mobile** (P0-3) |

---

## Contradictions entre rapports — résolues

### C1. Agent 1 (D-02) veut moins d'accent, Agent 2 ne se positionne pas dessus
**Résolution :** pas de contradiction réelle. On applique D-02 (hiérarchiser usages de `--accent`, viser ≤ 25 occurrences). Cohérent avec A11Y-02 (contraste) qui pousse dans le même sens.

### C2. Agent 2 propose de renommer les labels de section (Projets → Cas clients, Services → Ce que je fais, etc.). Les autres rapports ne disent rien.
**Résolution :** c'est un changement **éditorial**, pas technique. Impact UX réel mais non-bloquant. → **Question d'arbitrage Q3** posée au user.

### C3. Agent 2 propose de déplacer `.quote-band` (après Projets, pas avant À propos) et `.midcta-band` (après Tarifs, pas après Projets). Les autres rapports la maintiennent en place.
**Résolution :** Agent 2 argumente narrativement (confiance avant prix, CTA après info). Raisonnement solide mais nécessite réaccord du user car impact majeur sur le fil de lecture. → **Questions d'arbitrage Q1 + Q2**.

### C4. Agent 4 (C-03) recommande un refactor data-driven `const PROJECTS = [...]`. Les autres n'y touchent pas.
**Résolution :** refactor M (4-6h). Pas P0 absolu mais la présence de 4 fichiers `index copie*.html` en workspace est un symptôme direct. → **Question d'arbitrage Q4** posée au user : on data-drive ou on reste HTML hardcoded ?

### C5. Agent 1 (D-16) recommande de retirer le panneau `.tweaks` de prod. Agent 4 le liste dans "ce qu'on garde" (bien séparé, bon contrat).
**Résolution :** Agent 1 parle de sa présence visuelle en prod, Agent 4 parle de sa propreté de code. Les deux ont raison. → **Gater l'accès au panel derrière `?tweaks=1` URL param** ou `localhost`. C'est la synthèse correcte.

---

## TOP 10 P0 — issues bloquantes, à traiter en premier

> Format : `[P0-n] ISSUE-ID — titre` · owner agent · effort · fix résumé · dépend de

### P0-1 — Placeholders `[à compléter]` visibles en prod (4 endroits)
**Source** : S-02 (Structure) + C-03 (Code) + D-10 (quote-band) + D-19 (proof-band).
**Effort** : S (30-45 min).
**Fix** :
- `.ps-ed-desc` de Bebooth / Lucid / Scout → retirer les `<p>[à compléter]</p>`, afficher un simple « Projet bientôt détaillé » stylé OU masquer la `ps-block` entière.
- `.quote-band` → masquer en CSS (`display:none`) tant que le témoignage n'est pas rédigé.
- `.proof-band` → supprimer les 4 `<span>[Client 05-08]</span>`, dupliquer les 4 réels (Wenda · Bebooth · Scout · Lucid) en plus gros.
- Selected Work Index (6 lignes `[Projet 05-10]`) → masquer le bloc.
- Eyebrow Lucid Dreams `§ 01.3 / [Catégorie — à compléter]` → retirer le placeholder ou mettre `§ 01.3 / Music · Clip`.

**Impact visiteur** : crédibilité +++ en < 30 s de scroll. C'est **le** gain visible n°1.

---

### P0-2 — Section Contact : retirer le fond vert, revenir à l'identité éditoriale
**Source** : D-01 (Design).
**Effort** : S (30 min).
**Fix** :
- Supprimer `#contact::before` (lignes 1693-1707 du CSS) et toutes les règles liées au fond vert `#1F5F42`.
- Passer `#contact` soit sur `var(--bg)` sable (unifié avec les 5 autres sections), soit sur `var(--ink)` full-bleed (comme `.midcta-band`) pour rupture visuelle mais cohérente.
- Garder le bouton WhatsApp vert `#25D366` **uniquement comme icon-circle interne** (déjà le cas dans `.contact-wa-icon`).
- Tester que le `:focus-visible` passe AA sur le nouveau fond (cf A11Y).

**Recommandation** : passer sur `var(--ink)` — cohérent avec mid-CTA, donne un point d'arrivée dramatique en fin de page.

---

### P0-3 — Hero : ajouter positionnement (1 phrase) + 2 CTA
**Source** : S-01 (Structure) + D-09 (taille mobile).
**Effort** : S (45 min HTML + CSS).
**Fix** :
- Sous `.hero-morph`, ajouter `.hero-pitch` : *« Je filme et je monte pour des marques, des événements et des artistes. Je forme sur Premiere Pro à Bruxelles. »*
- Sous la `.hero-meta`, ajouter 2 boutons : `[Voir les cas clients →]` (ancre `#projets`) et `[Devis en 24h · WhatsApp]` (lien WhatsApp).
- Agrandir `.hero-name` mobile : `clamp(36px, 13vw, 180px)` → `clamp(48px, 16vw, 200px)` (D-09).

**Impact** : en 3 s, le visiteur sait pour qui CD travaille ET a une action claire.

---

### P0-4 — Case studies : masquage temporaire + réécriture Wenda (seul projet rédigé)
**Source** : S-02 + S-03 (Structure).
**Effort** : S (masquage 30 min) + L (réécriture dépend de CD, 45 min par projet × 4 hors scope de cette phase).
**Fix** :
- **Phase immédiate (scope audit)** : masquer les `.ps-ed-desc` vides (Bebooth/Lucid/Scout), retirer placeholders, garder les vidéos + les titres.
- **Phase hors scope (à faire par CD après audit)** : réécrire Wenda en case study complet (Contexte / Rôle / Décisions / Livrables & impact) selon le template fourni dans `02-structure.md`.

**Note critique** : ce P0 est partiellement bloqué par la rédaction CD. La partie **technique** (masquer les placeholders) est faisable tout de suite. Le **contenu** est un devoir à réaliser.

---

### P0-5 — Portrait : AVIF + srcset + lazy + alt narratif + cadrage carte
**Source** : PERF-01 (Perf) + D-08 (Design) + A11Y-05 (A11y).
**Effort** : S (1h — shell + HTML).
**Fix unifié** :
```bash
# Générer variantes
npx sharp-cli -i assets/portrait.png -o assets/portrait-700.avif --avif quality=55 resize=700
npx sharp-cli -i assets/portrait.png -o assets/portrait-1400.avif --avif quality=55 resize=1400
npx sharp-cli -i assets/portrait.png -o assets/portrait-700.webp --webp quality=78 resize=700
npx sharp-cli -i assets/portrait.png -o assets/portrait-1400.webp --webp quality=78 resize=1400
npx sharp-cli -i assets/portrait.png -o assets/portrait-1400.jpg --jpeg quality=82 resize=1400
```
```html
<picture>
  <source type="image/avif" srcset="assets/portrait-700.avif 700w, assets/portrait-1400.avif 1400w"
          sizes="(max-width: 900px) 90vw, 560px">
  <source type="image/webp" srcset="assets/portrait-700.webp 700w, assets/portrait-1400.webp 1400w"
          sizes="(max-width: 900px) 90vw, 560px">
  <img src="assets/portrait-1400.jpg"
       alt="Portrait de Corentin Devillé, vidéaste, en studio."
       width="1400" height="2100" loading="lazy" decoding="async">
</picture>
```
- Retirer `aria-hidden="true"` sur `.apropos-visual`.
- Déplacer `.apropos-card--float` de `top:-24px;right:-24px` à `bottom:-24px;right:-24px` pour libérer le visage.
- Retirer le `filter:grayscale(.08)` (soit 0, soit franc 100% en N&B — pas de demi-mesure).

**Gain** : 2.37 Mo → ~90 Ko sur mobile = **-2.28 Mo** sur bundle LCP-concurrent.

---

### P0-6 — Showreel hero : poster AVIF + dimensions explicites
**Source** : PERF-02 (Perf).
**Effort** : S (30 min).
**Fix** :
```bash
ffmpeg -ss 2 -i showreel-2026.mp4 -frames:v 1 -vf scale=1440:-1 showreel-poster.jpg
npx sharp-cli -i showreel-poster.jpg -o showreel-poster.avif --avif quality=50
# Upload showreel-poster.avif sur R2 (même bucket)
```
```html
<video class="hero-reel-video"
       src="https://videos.corentindeville.com/showreel-2026.mp4"
       poster="https://videos.corentindeville.com/showreel-poster.avif"
       autoplay muted loop playsinline preload="metadata"
       width="1440" height="617"></video>
```

**Gain** : LCP mobile passe de 3.2-4.1 s à 2.1-2.6 s (poster devient candidat LCP paint-able en ≤ 1.8 s).

---

### P0-7 — SEO : schema.org + canonical + og:image + sitemap + robots (bundle)
**Source** : SEO-01 + SEO-02 + SEO-03 (Perf/SEO).
**Effort** : S (1h30 — JSON-LD + meta + 2 fichiers + 1 OG image 1200×630).
**Fix** :
- Ajouter `<script type="application/ld+json">` Person + makesOffer (3 services) en fin de `<head>`.
- Ajouter dans `<head>` : `<link rel="canonical">`, `<meta property="og:url">`, `<meta property="og:image">` (+ `og:image:width/height/alt`), `<meta name="twitter:image">`, `<meta name="twitter:title">`, `<meta name="twitter:description">`.
- Créer `og-cover-1200x630.jpg` : export depuis un Figma/Photoshop avec nom + job + URL (à produire par CD ou proposer un layout à générer).
- Créer `robots.txt` et `sitemap.xml` à la racine (contenu exact dans `03-perf-a11y-seo.md` §SEO-03).
- Ajouter favicon : `<link rel="icon" type="image/svg+xml" href="/favicon.svg">` + mono-trait `CD` en `--accent`.

**Gain** : Lighthouse SEO 70 → 92+, partages sociaux WhatsApp/LinkedIn affichent enfin une preview.

---

### P0-8 — A11y : h1 textuel en dur + contraste accent + menu aria-expanded (bundle)
**Source** : A11Y-01 + A11Y-02 + A11Y-03 (A11y).
**Effort** : S (45 min total).
**Fix** :
- **h1** : mettre `"Corentin"` et `"Devillé"` en dur dans les `<span class="word">`, garder le JS qui les split en `.char` mais lire `textContent` au lieu de `dataset.word`. Retirer `aria-label` (devenu redondant).
- **Contraste** : créer `--accent-text` = `#CC3D00` (ratio 4.6:1 body) en light, = `#FF7A3D` en dark. Remplacer `color:var(--accent)` par `color:var(--accent-text)` dans tout body-text (~5 occurrences). **Garder** `var(--accent)` pour backgrounds, borders, icônes, large-text.
- **Menu hamburger** : ajouter `aria-expanded="false"` + `aria-controls="mobMenu"`, JS qui met à jour, gestion focus à l'ouverture/fermeture + Escape qui ferme.

**Gain** : Lighthouse A11y 88 → 95+, WCAG 2.1 AA compliance formelle.

---

### P0-9 — Dead CSS : purge des 12 familles orphelines (~900 lignes)
**Source** : C-01 (Code) + D-12 + D-15 + D-17 (Design — `.pcard`, `.avail-badge`, `.spec-badge` morts).
**Effort** : M (2h avec tests visuels mobile + desktop).
**Fix** :
- Script de vérification avant suppression (commande fournie dans `04-code.md` §C-01).
- Supprimer : `.pcard*` / `.pgrid` / `.vcard*` / `.vgrid` / `.crow*` / `.contact-list` / `.footbanner*` / `.ps-block--below*` / `.wide-*` / `.ps-wide-layout` / `.wenda-*` (mobile v1) / `.cp-*` (contact portrait v1) / `.topnav` / `.avail-badge*` / `.brand-status` / `.eyelid*` / `.logo-wrap`.
- Conserver `#contact::before` pour l'instant si P0-2 ne l'a pas encore retiré (ordre d'exécution à prévoir).
- Supprimer aussi `.ps-divider` CSS + HTML (C-05).
- Nettoyer le bloc `/* Editorial micro-moves */` en fin de fichier (C-07) : merger chaque règle dans sa section d'origine.

**Gain** : -35 % du bloc `<style>` envoyé en clair (~22 Ko brut économisés, ~6 Ko gzippé).

---

### P0-10 — Dead JS : supprimer 2 IIFE + 2 sélecteurs morts
**Source** : C-02 (Code).
**Effort** : S (10 min).
**Fix** :
- Retirer `carousel-scout` de la liste `['carousel-wenda','carousel-bebooth','carousel-lucid','carousel-scout'].forEach(initStackCarousel)` (ligne 3189).
- Supprimer intégralement le bloc `WIDE CAROUSEL — Bebooth` (lignes 3232-3257, 26 lignes mortes).
- Dans `querySelectorAll('.stats .n, .apropos-stats .n')` : retirer `.stats .n`.
- Dans `querySelectorAll('.cta,.hero-meta a,.menu-btn')` : retirer `.cta`.

**Gain** : code signal propre, évite les malentendus pour le futur Corentin qui relirait.

---

## Récapitulatif P0 — tableau synthétique

| # | Issue | Agent | Effort | Temps estimé | Gain principal |
|---|---|---|---|---|---|
| P0-1 | Placeholders masqués | Structure + Code | S | 45 min | Crédibilité visible |
| P0-2 | Contact sans fond vert | Design | S | 30 min | Identité cohérente |
| P0-3 | Hero pitch + CTA | Structure + Design | S | 45 min | Positionnement clair |
| P0-4 | Case studies masqués + Wenda | Structure | S (+L hors scope) | 30 min + LIVRABLE CD |
| P0-5 | Portrait AVIF + cadrage + alt | Perf + Design + A11y | S | 1 h |  -2.28 Mo bundle |
| P0-6 | Showreel poster AVIF | Perf | S | 30 min | LCP -1 à -1.5 s |
| P0-7 | SEO bundle (schema+canonical+og+sitemap+robots+favicon) | Perf/SEO | S | 1 h 30 | SEO 70→92 |
| P0-8 | A11y bundle (h1+contraste+menu-aria) | Perf/A11y | S | 45 min | A11y 88→95 |
| P0-9 | Dead CSS purge | Code + Design | M | 2 h | -35 % CSS |
| P0-10 | Dead JS purge | Code | S | 10 min | Code signal propre |

**Total P0 tech : ~8-9 h de travail actif** (hors rédaction Wenda par CD).

---

## P1 — à traiter juste après P0, groupés par thème

### Thème A — Accent & hiérarchie typographique (Design)
- **D-02** — Hiérarchiser les usages de `var(--accent)` : Tier A (ponctuation forte : ▶, first-letter, dot pulsante), Tier B (prix/stats), **interdire** bordures structurelles + underlines hover génériques. Viser ≤ 25 occurrences. [M, 2h]
- **D-03** — Unifier les eyebrows : niveau A = box ink 12px mono `.22em`, niveau B = mono plain 11px `.22em` muted avec tiret accent. Supprimer override `.ps-ed-header .ps-idx` (trop gros). [M, 1h30]
- **D-07** — Augmenter le ratio h2/h3 : `.ps-ed-title` passer à `clamp(40px, 8vw, 120px)`, maintenir `.section-head h2`. [S, 15 min]
- **D-18** — Unifier letter-spacing Dela Gothic : `-.02em` si ≤80px, `-.015em` si >80px. [S, 10 min]

### Thème B — Rythme vertical & tokens (Design + Code)
- **D-06** — Tokeniser les espacements : `--gap-section`, `--gap-band`, `--gap-sub` dans `:root`. Remplacer ~15 valeurs hardcodées. [M, 1h]
- **C-08** — Tokeniser `--ease-out`, `--ease-in-out`, `--dur-fast/med/slow`, `--gap-*`. Search/replace ~18 occurrences. [M, 1-2h] — **à faire dans le même commit que D-06** pour cohérence.

### Thème C — Perf avancée
- **PERF-03** — Vidéos carousels en `preload="none"` + IntersectionObserver pour `play()`/`pause()`. Réduire `autoplay` au seul showreel hero. [M, 1-2h]
- **PERF-04** — Option A : preload Dela Gothic + Inter 400 latin en `<link rel="preload">`. Option B (plus tard) : self-host fonts. [S, 10 min pour A]
- **PERF-05** — Film-loader : réduire 3.4 s → 1.4 s OU le gater par `sessionStorage` (1 affichage par session). Ajouter `display:none` sous `prefers-reduced-motion`. [S, 15 min]
- **PERF-06** — Wrapper scrollspy `update()` dans rAF + ticking flag (pattern déjà utilisé par le parallax). [S, 5 min]

### Thème D — A11y avancée
- **A11Y-04** — Carousels : transformer les `<span class="reel-dot">` en `<button>` focusables, ajouter `role="tablist"`, `aria-selected`, `role="region"` + `aria-roledescription="carousel"`. [S, 30 min]
- **A11Y-06** — Désactiver le scramble/morphing/ticker JS sous `prefers-reduced-motion` (media query JS, pas seulement CSS). [S, 10 min]

### Thème E — Narration & fil éditorial (Structure)
- **S-04** — Différencier visuellement les 5 CTA WhatsApp (mid-CTA = pitch long, contact section = formulaire+canaux, FAB = icon seul, hero = à ajouter en P0-3, drawer = garder). [M, 1h — à coordonner avec P0-3]
- **S-05** — Reformuler les 3 stats `.apropos-stats` : « 40+ projets livrés » / « 2022→2026 » / « Bruxelles → EU ». Supprimer le confus « 04 ». [S, 15 min]
- **S-06** — Ajouter un bloc « Disponibilité » sous Tarifs : liste 3 mois prochains + statut (Ouvert / 2 créneaux / Complet). [S, 45 min]
- **S-09** — Mobile `.ps-ed` : réduire meta de 5 à 3 spans, masquer boutons prev/next carousel, limiter description à 2 paragraphes. [M, 1h]

### Thème F — Code maintenance
- **C-04** — Merger `.apropos-card::before` dupliqué (lignes 2019 + 2453). [S, 2 min]
- **C-05** — Supprimer `.ps-divider` (CSS + 3 divs HTML). [S, 5 min] — **inclus dans P0-9**.
- **C-06** — Dédoublonner `.topbar .brand`, `.hero-reel .ph-label`, `.apropos-visual`. [S, 5 min]
- **C-07** — Migrer les 8 règles du bloc `/* Editorial micro-moves */` dans leurs sections d'origine, retirer `!important`. [S, 15 min] — **inclus dans P0-9**.

### Thème G — SEO avancé
- **SEO-04** — Descendre `<h2 class="midcta-title">` en `<h3>` (bande éditoriale ≠ section principale). [XS, 30 s]

---

## P2 — hygiène, à traiter opportunément

Regroupés rapidement car peu critiques :
- **D-04** — Retirer le curseur custom `cursor:none` (robustesse premium). [S]
- **D-05** — Garder un seul grain (`#film-grain`), supprimer `.grain` et son div. [S]
- **D-11** — Uniformiser les bordures accent épaisses (4px border-left max, sur 2 blocs). [S]
- **D-13** — Unifier `#E43D12` dot topbar sur `var(--accent)`. [S]
- **D-14** — Hero `.morph-word` : `min-width:9ch` au lieu de 10. [S]
- **D-16** — Tweaks panel : gater derrière `?tweaks=1` URL param (ne pas supprimer le code, juste le HTML livré). [S]
- **D-17** — Retirer badges `4K / 60fps` topbar (techno-bro) OU remplacer par `Éd. 01 / 2026`. [S]
- **D-20** — Ajuster `:focus-visible` : retirer `border-radius:2px`, tester 3px sur CTA principaux. [S]
- **S-07** — Ajouter transitions narratives entre sections (`.section-bridge` 1 phrase). [S, éditorial]
- **S-08** — Normaliser numérotation `§` (sections = `§ NN / titre`, bandes = `§ · intermède`). [S]
- **S-10** — Réduire nav desktop à 4 entrées (Projets / Services / Tarifs / Contact) ? [M, arbitrage éditorial]
- **S-11** — Enrichir ticker hero : passer de 12×2 duplications à 20-24 termes uniques. [S]
- **A11Y-07** — RAS (déjà bien fait, mention positive).
- **SEO-05** — Garder l'email obfusqué Cloudflare (trade-off anti-spam OK). RAS.
- **SEO-06** — Inclus dans P0-7 (favicon).
- **PERF-07** — **Skip** : minification HTML incompatible avec contrainte no-build.
- **C-09** — Regrouper les `@media` éparpillés. [M] — à faire lors d'une futur refonte mobile.
- **C-10** — Nettoyer commentaires obsolètes. [S] — à faire pendant P0-9.
- **C-11** — Data-driven Méthode + Tarifs si on data-drive les projets. [S] — dépend de Q4.
- **C-12** — Selected Work Index : inline-style gradient au lieu de classes `.r1-.r6`. [S] — inclus dans P0-9 si data-drive.

---

## Questions d'arbitrage pour toi — AVANT exécution

Je te demande de trancher ces 6 points. Certains changent la structure narrative, donc je ne les implémente pas sans GO explicite.

### Q1 — Déplacer `.quote-band` : on la met **après Projets** (Agent 2) ou on la laisse **avant À propos** (structure actuelle) ?

**Argument Agent 2 pour déplacer** : validation sociale au moment où le doute naît (juste après avoir vu les projets), pas après l'achat.
**Argument pour conserver** : positionnement actuel = transition douce entre la partie commerciale (Tarifs) et la partie identité (À propos).
**Ma recommandation** : **déplacer après Projets**, mais seulement une fois qu'un vrai témoignage est rédigé. En attendant, la `.quote-band` est masquée (P0-1) donc le débat est théorique jusqu'à rédaction.

### Q2 — Déplacer `.midcta-band` : on la met **après Tarifs** (Agent 2) ou on la laisse **entre Projets et Services** (structure actuelle) ?

**Argument Agent 2 pour déplacer** : ne coupe plus la démo (Projets → Services doit être fluide), CTA arrive quand le prospect a toutes les infos (projets + services + méthode + tarifs).
**Argument pour conserver** : rythme actuel (bande de respiration au milieu) fonctionne visuellement.
**Ma recommandation** : **déplacer après Tarifs** — cohérent avec Q1, et l'argument narratif est solide.

### Q3 — Renommer les labels de section ?

Proposition Agent 2 :
| Actuel | Proposé | Ma reco |
|---|---|---|
| Projets | Cas clients | OUI — centre sur usage/résultat |
| Services | Ce que je fais | OUI — langue humaine |
| Méthode | Comment on travaille | OUI — « on » inclusif |
| Tarifs | Budget & disponibilité | OUI — répond à 2 questions en 1 |
| À propos | Qui je suis | Optionnel — « À propos » reste très lisible |
| Contact | Écris-moi | Optionnel — « Contact » reste clair |

**Ma recommandation** : appliquer les 4 premiers (gain narratif net), laisser « À propos » et « Contact » tels quels (risque de sur-traduire le ton). Mais c'est **ton portfolio** — trancher.

### Q4 — On data-drive les projets en `const PROJECTS = [...]` + template literal (C-03) ?

**Pour** : édition future divisée par 5, supprime le besoin des `index copie*.html`, ajouter un projet = 1 objet JS de 10 lignes.
**Contre** : 4-6h de refactor, introduction de rendu dynamique (compatible Google depuis 2019, mais pas tous les crawlers).
**Ma recommandation** : **OUI mais uniquement si tu prévois d'ajouter ≥ 2 projets dans les 3 prochains mois**. Sinon, 4 projets hardcodés = acceptable, et on fait le cleanup (P0-9 + P0-10) sans aller plus loin.

### Q5 — Case studies en attente : on masque OU on affiche un placeholder stylé ?

Pour Bebooth / Lucid / Scout (description non rédigée) :
- **Option A** : masquer toute la `.ps-block` → on voit 1 projet Wenda seul. Honnête mais squelettique.
- **Option B** : garder la `.ps-block` avec vidéos + titre, masquer uniquement la `.ps-ed-desc` (pas de texte). 3/4 projets apparaissent visuellement sans placeholder.
- **Option C** (si Q4 = OUI) : afficher un placeholder stylé « Projet bientôt détaillé. » au lieu du texte `[à compléter]`.

**Ma recommandation** : **Option B** (masque desc, garde visuels). Vu que Scout + Bebooth + Lucid ont déjà des vidéos qui montrent le travail, on ne perd rien. À toi de rédiger quand tu pourras.

### Q6 — Disponibilité mensuelle (S-06) : on l'ajoute ?

**Pour** : débloque 20-30 % des prospects qui veulent savoir si CD est libre avant de contacter.
**Contre** : c'est une donnée qui **doit être tenue à jour** (sinon pire que rien). Si tu n'as pas la discipline de la MAJ mensuelle, skip.
**Ma recommandation** : ajouter seulement si tu t'engages à une MAJ mensuelle. Sinon, skip.

---

## Ordre d'exécution proposé (atomic commits)

> Chaque ligne = un commit. Commits ordonnés pour max visible impact + minimum de rework.

| # | Commit | Scope | Dépend de |
|---|---|---|---|
| 1 | `chore(audit): add AUDIT/ folder with recon + 4 reports + plan-action` | Docs | — |
| 2 | `fix(content): hide placeholders in case studies, quote-band, proof-band, selected work index` | P0-1 + P0-4 partiel | — |
| 3 | `refactor(contact): drop green backdrop, unify on ink full-bleed` | P0-2 | — |
| 4 | `feat(hero): add positioning line + 2 CTAs, bump mobile name size` | P0-3 + D-09 | — |
| 5 | `perf(assets): portrait → AVIF/WebP/JPEG variants + srcset + lazy, drop aria-hidden, narrative alt` | P0-5 | assets/ prêts |
| 6 | `perf(hero): showreel poster AVIF + explicit dimensions` | P0-6 | R2 poster uploadé |
| 7 | `feat(seo): add JSON-LD Person, canonical, og:image, twitter card, favicon + sitemap.xml + robots.txt` | P0-7 | og-cover prêt |
| 8 | `fix(a11y): h1 static text, --accent-text var, menu aria-expanded + focus management` | P0-8 | — |
| 9 | `refactor(css): purge dead classes (.pcard .vcard .cp-* .wide-* .footbanner etc) + merge editorial patch block` | P0-9 + C-04 + C-05 + C-06 + C-07 | P0-2 appliqué (pour `#contact::before`) |
| 10 | `refactor(js): drop dead carousel-scout + wide-bebooth IIFE + stale selectors` | P0-10 | — |
| 11 | (optionnel Q4) `refactor(data): extract PROJECTS[] + render via template literal` | C-03 | Q4 = OUI |
| 12 | `style(design): scope --accent to decorations/large-text, unify eyebrows + letter-spacings` | P1 thème A | — |
| 13 | `style(rhythm): tokenize --gap-section/band/sub + --ease-* + --dur-*` | P1 thème B | — |
| 14 | `perf(video): carousel videos preload=none + IntersectionObserver play/pause` | P1 thème C · PERF-03 | — |
| 15 | `perf(fonts): preload Dela Gothic + Inter 400 latin (option A)` | P1 thème C · PERF-04 | — |
| 16 | `perf(loader): film-loader 3.4s → 1.4s + session-gated + reduced-motion` | P1 thème C · PERF-05 | — |
| 17 | `fix(a11y): reel-dots as buttons, carousel aria + scroll-reduce JS` | P1 thème D | — |
| 18 | (si Q3 = OUI) `refactor(copy): rename section labels (Projets→Cas clients etc)` | P1 thème E | — |
| 19 | (si Q1+Q2 = OUI) `refactor(structure): reorder quote-band + midcta-band` | P1 thème E | — |
| 20 | `chore(cleanup): move index copie*.html outside workspace, document workflow` | Hygiène | C-03 si fait |

**Total commits principaux** : **10 obligatoires (P0)** + **5-7 secondaires (P1)** + **2-3 conditionnels (Q3/Q4/Q1+Q2)**.

---

## Ce qu'on NE TOUCHE PAS — décisions explicites

> Pour éviter toute dérive pendant l'exécution.

1. **Le stack vanilla / no-build** — aucune migration vers Astro/Next/Vite/Hugo. Contrainte dure, respectée par tous les agents.
2. **Les vidéos sur R2** — on ne les migre pas vers YouTube/Vimeo/Mux. R2 fonctionne, pas touche.
3. **Les 3 familles de fonts** (Dela Gothic One / Inter / JetBrains Mono) — signature forte. On peut les preload ou les self-host, mais pas les changer.
4. **La palette `--bg` sable + `--ink` noir + `--accent` vermillon** — identité stable. Seul le *scope* de `--accent` change (P1 thème A).
5. **Le `.film-loader` intro** — garder l'effet, juste le raccourcir (PERF-05). Micro-signature liée au logo œil.
6. **Le morph `.hero-name` char-by-char** — bon composant, à conserver tel quel. Seul l'input devient du texte statique (P0-8 A11Y-01) au lieu de `dataset.word`.
7. **L'IntersectionObserver `.reveal`** — pattern performant, à préserver.
8. **Le carousel Wenda/Bebooth/Lucid en 3D stack** — composant signature, on ne le remplace pas.
9. **Le FAB WhatsApp mobile** — bonne pratique conversion, à garder.
10. **La carte ID-CD flottante dans À propos** — idée forte, à protéger. Juste repositionnée (bottom au lieu de top) dans P0-5.
11. **Le dark mode via `[data-theme]`** — séparation propre, à conserver.
12. **Les landmarks HTML5 + `prefers-reduced-motion`** — base a11y solide, à préserver.
13. **Le `.tweaks` panel** — gardé en code, gated derrière URL param en prod (D-16).
14. **L'obfuscation email Cloudflare** — trade-off anti-spam OK (SEO-05).

---

## Cibles post-exécution — à valider en QA

### Métriques techniques (après P0 + P1)

| Métrique | Avant | Après P0 | Après P0+P1 | Méthode de mesure |
|---|---|---|---|---|
| LCP mobile (4G lente) | 3.2 - 4.5 s | 2.1 - 2.6 s | **1.6 - 2.1 s** | Chrome DevTools Moto G4 emulation |
| CLS | ~0.05 - 0.08 | ~0.03 - 0.05 | **< 0.02** | Lighthouse |
| INP | 180 - 260 ms | 130 - 180 ms | **< 120 ms** | Lighthouse |
| Bundle total mobile | ~2.7 Mo | ~0.4 Mo | **~0.35 Mo** | Network tab |
| Lighthouse Perf | 55-65 | 80-85 | **90+** | Lighthouse mobile |
| Lighthouse A11y | 88 | 95 | **98+** | Lighthouse |
| Lighthouse SEO | 70 | 92-95 | **95+** | Lighthouse |
| Lighthouse Best Practices | 92 | 95 | **100** | Lighthouse |

### Métriques éditoriales (à valider à l'œil)

- Aucun `[à compléter]` / `[Client 0X]` / `[Projet 0X]` visible en prod.
- Hero annonce clairement pour qui CD travaille (lecture en ≤ 3 s).
- Contact sobre, cohérent avec le reste, pas de rupture verte.
- Portrait en AVIF, carte flottante qui ne cache pas le visage.
- Identité monochrome + 1 accent (vermillon) utilisé comme ponctuation, pas comme ambiance.

### Métriques de code

- `grep -c "class=\"[^\"]*\bpcard\b" index.html` → **0**
- `grep -c "class=\"[^\"]*\bvcard\b" index.html` → **0**
- `grep -c "class=\"[^\"]*\bwide-card\b" index.html` → **0**
- Pas de `!important` (hors reset) dans le CSS.
- Pas de `[à compléter]` dans le HTML.

---

## Plan par phases (post-GO)

### Phase EXÉCUTION — P0 (jour 1)
Commits 1 → 10. Environ 8-9 h de travail actif. Tous les P0 tech traités. Site crédible, CWV acceptables, SEO propre.

### Phase EXÉCUTION — P1 (jour 2)
Commits 12 → 17 (+ 18/19 si Q3/Q1/Q2 = OUI). Environ 6-8 h. Finitions design, perf avancée, a11y avancée.

### Phase QA (fin jour 2)
Agent critic final qui vérifie :
- P0 tous résolus (grep check)
- CWV estimés cibles atteintes
- Aucun placeholder en prod
- `AUDIT/CHANGELOG.md` rédigé avec before/after + métriques + issues non-traitées et leurs raisons
- Commit final `chore(audit): close audit with CHANGELOG + metrics`

---

## Demande de GO explicite

Je **ne touche rien** tant que tu n'as pas répondu aux 6 questions d'arbitrage ci-dessus **et** donné un GO explicite.

**Réponds au minimum à :**
- Q1 (quote-band) : déplacer OUI / garder
- Q2 (mid-CTA) : déplacer OUI / garder
- Q3 (renommage labels) : 4 premiers OUI / tous / aucun
- Q4 (data-drive projets) : OUI / NON
- Q5 (cas studies en attente) : option A / B / C
- Q6 (disponibilité mensuelle) : OUI / NON

**Puis un GO** pour démarrer la Phase EXÉCUTION. Je travaillerai en atomic commits dans l'ordre proposé, je te signalerai chaque commit pushé, et j'arrête sur demande.

— *Fin du plan d'action.*
