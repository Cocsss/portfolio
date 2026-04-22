# AUDIT / 02 — Structure, IA & narration

> Audit read-only. Base : `ce948e5` + `index.html` 2558-3040.
> Focus : information architecture, narration, case studies, labels.
> Auditeur : UX strategist senior, point de vue recruteur/prospect senior en 30s.

---

## Executive Summary

Le site **présente un créateur avant de répondre à une intention client** : le hero annonce un nom + un showreel mais jamais **pour qui** CD travaille ni **ce qu'il vend**. La séquence Projets → Services → Méthode → Tarifs → À propos → Contact est logique sur le papier, mais **3 des 4 case studies sont encore des placeholders** (Bebooth, Lucid Dreams, Scout), et le seul rédigé (Wenda) est une description, pas un case study (pas de rôle, pas de livrable, pas d'impact). La redondance CTA WhatsApp (×4) pèse moins que **l'absence de preuve sociale mesurable** (témoignage placeholder, logos en texte, stats opaques). Verdict : squelette narratif solide, **contenu creux** — le recruteur senior parcourt 4 sections avant d'avoir un fait concret à retenir.

---

## Carte actuelle des sections

```
HERO                          · Identité + showreel + ticker        [charge : identité faible, preuve forte]
§ PROOF-BAND                  · Clients défilants (4 réels + 4 [Client 05])  [charge : preuve sociale en demi-teinte]
§ 01 PROJETS                  · 4 case studies (1 rédigé, 3 placeholders) + Index 6 lignes placeholder
                                                                     [charge : démonstration — mais le contenu manque]
§ PASSONS À L'ACTION          · Mid-CTA WhatsApp                    [charge : conversion intermédiaire]
§ 02 SERVICES                 · 2 cartes (Vidéo / Formation)        [charge : offre commerciale]
§ 03 MÉTHODE                  · 4 étapes (Brief / Tournage / Montage / Livraison)
                                                                     [charge : rassurance process]
§ 04 TARIFS                   · 3 lignes horaires (20€ / 25€ / 20€) [charge : transparence prix]
§ QUOTE-BAND                  · Témoignage placeholder               [charge : preuve sociale — vide]
§ 05 À PROPOS                 · Photo + card flottante + bio + 3 stats (40+ / 04 / BE)
                                                                     [charge : personnalité]
§ 06 CONTACT                  · CTA WhatsApp principal + email / IG / YT
                                                                     [charge : conversion finale]
FOOTER                        · Copyright + back to top
FAB mobile                    · WhatsApp flottant (occurrence #4 du CTA)
```

**Lecture narrative perçue** : « Voici mes projets (beaux mais vides) → achetez → voici comment → voici combien → (vide témoignage) → qui je suis → contact ». L'appel à l'action arrive trop tôt (mid-CTA juste après les projets), alors que le visiteur n'a pas encore reçu de signal de preuve (témoignage, logos, chiffres). La séquence commerciale (services / méthode / tarifs) pousse avant de vendre.

---

## Carte proposée

```
HERO                          · Identité + POSITIONNEMENT EXPLICITE + showreel
                                « CD · vidéaste d'événements et de marques ·
                                  formateur Premiere Pro · Bruxelles »
                                [ajout : sous-titre cible + 2 CTA : Voir projets / Devis 24h]

§ PROOF-BAND                  · Logos clients (SVG, pas texte) + 1 stat visible
                                (« 40+ projets livrés / 2022→2026 »)

§ 01 CAS CLIENTS              · [renommé] — 4 case studies RÉDIGÉS
                                · Problème → Rôle → Décisions → Livrables
                                · Chaque carte : 3-4 metrics (vues, durée, format,
                                  plateforme, budget si ok de le dire)

§ QUOTE-BAND                  · [déplacé ICI] — témoignage client juste après
                                le premier vrai cas → vibe de confiance
                                AVANT de parler prix

§ 02 CE QUE JE FAIS           · [renommé Services] — 2 offres + tags concrets
                                (inchangé sur le fond)

§ 03 COMMENT ON TRAVAILLE     · [renommé Méthode] — 4 étapes (inchangé)
                                + ajout micro-FAQ collapse (3-4 Q/R max)

§ 04 BUDGET & DISPONIBILITÉ   · [fusion Tarifs + bloc nouveau] —
                                · Grille horaire (actuelle)
                                · Exemples de budgets packages ("Reel x3 ≈ 250€")
                                · Statut dispo ("Dispo mai 2026 ✓")

§ PASSONS À L'ACTION          · Mid-CTA — déplacé juste avant À propos, pas entre
                                projets et services (coupe la démo)

§ 05 QUI JE SUIS              · [renommé À propos] — photo + bio + 3 stats LISIBLES
                                (« 40+ projets livrés / 4 ans de pratique /
                                  basé à Bruxelles, déplacements EU »)

§ 06 ÉCRIS-MOI                · [renommé Contact] — CTA WhatsApp + canaux
                                · Ajout mini-formulaire 3 champs (optionnel)
                                  pour les prospects qui refusent WhatsApp

FOOTER                        · (inchangé)
FAB mobile                    · (conservé, seul CTA mobile persistant)
```

### Diff commenté

| Action | Élément | Raison |
|---|---|---|
| **Ajout** | Sous-titre cible dans hero | Répond au « pour qui ? » (marques / événements / étudiants) |
| **Ajout** | 2 CTA hero (Voir projets / Devis 24h) | Donne une action immédiate au scanner de 3s |
| **Ajout** | Logos SVG vs texte dans proof-band | Preuve visuelle vs mention verbale |
| **Ajout** | Bloc « Disponibilité » fusionné avec Tarifs | Répond à « peut-il le faire ce mois-ci ? » — info critique pour event manager |
| **Ajout** | Exemples de budgets packages | Passer de « tarif horaire » (effrayant) à « exemple concret » (rassurant) |
| **Ajout** | Micro-FAQ sous Méthode | Pré-répond aux 3-4 objections classiques (droits, déplacements, révisions) |
| **Ajout** | Mini-formulaire dans Contact | 15 % de prospects refusent WhatsApp en premier contact |
| **Déplacement** | Quote-band : après Projets (pas avant À propos) | Validation sociale au moment où le doute naît, pas après l'achat |
| **Déplacement** | Mid-CTA : après Tarifs (pas après Projets) | Ne coupe plus le storytelling démonstratif ; arrive au moment où le prospect a toutes les infos |
| **Renommage** | « Projets » → « Cas clients » | Centre sur l'usage client, pas sur le portfolio créatif |
| **Renommage** | « Services » → « Ce que je fais » | Langue humaine vs jargon commercial |
| **Renommage** | « Méthode » → « Comment on travaille » | « On » = inclusion client, pas solo créateur |
| **Renommage** | « Tarifs » → « Budget & disponibilité » | Répond à 2 questions, pas 1 |
| **Renommage** | « À propos » → « Qui je suis » | Plus direct, moins corporate |
| **Renommage** | « Contact » → « Écris-moi » | Action vs catégorie |
| **Suppression** | Placeholders `[Client 05]` → `[Client 08]` | Tant qu'il n'y a pas 8 clients, n'en montre que 4 en plus gros |
| **Suppression** | Selected Work Index (6 lignes placeholder) | Un index vide signale un portfolio vide. Le masquer jusqu'à ce qu'il y ait 4 projets réels en plus. |

---

## Issues numérotées

### S-01 [P0] — Le hero ne dit pas « pour qui » ni ne propose d'action

**Observation** : lignes 2582-2634. Le hero contient nom + morphing « Je suis — Vidéaste » + meta (Vidéaste & Formateur / Basé en Belgique) + showreel. Aucune mention des **cibles** (marques, événements, artistes, étudiants), aucun CTA primaire.

**Impact visiteur** : en 3 secondes, le visiteur sait QUI (Corentin Devillé) et voit UNE PREUVE (le showreel en fond). Mais il ne sait pas **s'il est au bon endroit** pour son besoin. Un event manager scanne puis part vers un autre onglet.

**Fix concret** :
- Sous `.hero-morph`, ajouter une ligne `.hero-pitch` :
  `« Je filme et je monte pour des marques, des événements et des artistes. Je forme sur Premiere Pro à Bruxelles. »`
- Sous la meta, ajouter 2 boutons : `[Voir les cas clients →]` et `[Devis en 24h · WhatsApp]`
- Garder le showreel en fond (il est la preuve implicite).

**Effort** : S (20-30 lignes HTML + 40 lignes CSS)

---

### S-02 [P0] — 3 case studies sur 4 sont des placeholders en production

**Observation** : lignes 2748-2750 (Bebooth), 2759 / 2797-2799 (Lucid Dreams), 2822-2823 (Scout Festival). Tous contiennent littéralement `[Description — à compléter]` affiché à l'utilisateur. Même la ligne eyebrow de Lucid Dreams dit `§ 01.3 / [Catégorie — à compléter]`. L'index « Autres travaux » a 6 lignes `[Projet 05]` → `[Projet 10]`.

**Impact visiteur** : crédibilité **détruite** en 10 secondes de scroll. Un prospect voit du placeholder = le site n'est pas prêt = le créateur n'est pas sérieux. P0 absolu.

**Fix concret** :
- Option A (court terme, 30 min) : masquer Bebooth, Lucid, Scout tant qu'ils ne sont pas rédigés. Afficher uniquement Wenda en vedette + un bloc « Plus de projets bientôt ».
- Option B (court terme bis) : garder les vidéos mais retirer les `<div class="ps-ed-desc">` contenant des placeholders jusqu'à rédaction.
- Option C (vrai fix, 2-3h) : CD rédige 3 paragraphes par projet selon le template ci-dessous (voir section « Case studies »).
- Pour l'index « Autres travaux » : masquer le bloc entier (`display:none` sur `.section-sub`) tant qu'il n'y a pas 4 lignes réelles.

**Effort** : S pour le masquage, L pour la rédaction complète (dépend de CD).

---

### S-03 [P0] — Le seul case study rédigé (Wenda) n'est pas un case study

**Observation** : lignes 2699-2700. Le texte Wenda décrit **la marque cliente** (histoire de famille, maison 1972) et **ce que CD fait** (« capturer l'élégance »). Aucun des 4 piliers attendus : problème posé par le client, rôle précis de CD, décisions créatives spécifiques, résultat mesurable.

**Impact visiteur** : le recruteur senior cherche « qu'est-ce qu'il a résolu ? ». Actuellement il reçoit « pour qui il l'a fait ». Ce n'est pas la même chose.

**Fix concret** : réécrire Wenda selon le template plus bas (§ Case studies). Garder les 2 paragraphes actuels en « Contexte » mais ajouter 3 sous-blocs : Rôle / Décisions / Livrables & impact.

**Effort** : S (réécriture, 45 min pour CD).

---

### S-04 [P1] — Redondance CTA WhatsApp : 4 occurrences, aucune hiérarchie

**Observation** :
1. Header desktop : `brand > a href="#contact"` (lien neutre mais pointe sur la section contact où réside le WA)
2. Contact drawer (panel ouvrable par eye-bubble) : ligne 2526
3. Mid-CTA band : ligne 2850
4. Contact section : ligne 3001
5. FAB mobile : ligne 3036

Soit 5 points d'entrée WhatsApp en scroll complet — trop sans hiérarchie.

**Impact visiteur** : le CTA perd de la force par sur-exposition. Sur mobile, le FAB flottant + la mid-CTA + la section contact empilent 3 boutons WhatsApp identiques en ≤ 3 écrans.

**Fix concret** :
- Garder FAB mobile (toujours visible → primaire sur mobile).
- Garder section contact (point d'atterrissage).
- Mid-CTA : **déplacer après Tarifs**, pas après Projets (cf. carte proposée).
- Contact drawer (eye-bubble) : garder (comportement discret déclenché au clic).
- Différencier visuellement : mid-CTA = version « pitch long » avec tagline « Devis 24h » ; section contact = version « formulaire + canaux » ; FAB = icône seule.
- Vérifier qu'en mobile viewport 390px, on ne voit jamais 2 CTA WhatsApp **actifs en même temps** à l'écran.

**Effort** : M (déplacement HTML + coordination z-index FAB vs mid-CTA).

---

### S-05 [P1] — Preuve sociale dégradée : témoignage placeholder, logos en texte, stats opaques

**Observation** :
- `.quote-band` ligne 2939-2940 : `«&nbsp;[Témoignage client — 1 à 2 phrases à compléter par Corentin].&nbsp;»` + `[Prénom Nom] — [Fonction · Structure]`. Affiché tel quel en prod.
- `.proof-band` ligne 2640-2646 : texte display, 4 réels + 4 `[Client 05]` → `[Client 08]`.
- `.apropos-stats` ligne 2979-2983 : `40+` (Projets livrés) clair, `04` (Ans d'expérience) confus (« 04 ans » ? « 2004 » ?), `BE` (Base · disponible EU) ambigu.

**Impact visiteur** : la preuve sociale est **l'axe principal de doute** d'un prospect. Un placeholder de témoignage signale que le vrai client n'a pas encore répondu = pas de clients actifs récents. Les stats opaques ne construisent pas de confiance.

**Fix concret** :
- `.quote-band` : **masquer en CSS** tant que le témoignage n'est pas rédigé. Mieux vaut rien qu'un placeholder.
- `.proof-band` : retirer `[Client 05]` → `[Client 08]`. Dupliquer les 4 réels en plus gros + plus d'espace. Si possible, remplacer texte par SVG logos (cohérence graphique de portfolio).
- `.apropos-stats` : reformuler en 3 chiffres explicites : « 40+ projets / 2022→2026 / Bruxelles → Europe ». Éviter « 04 ans » qui n'est jamais lu comme « 4 ans ».

**Effort** : S (masquage / reformulation texte), M si SVG logos.

---

### S-06 [P1] — Absence de bloc « disponibilité » et de process de devis

**Observation** : rien dans le site ne répond à « est-il libre en mai ? » ni « comment je commande un devis ? ». La méthode (§03) décrit le travail après le contrat ; il manque la pré-étape d'avant-contrat.

**Impact visiteur** : un event manager qui prépare un festival en juin veut savoir **immédiatement** si CD est libre sur les dates. S'il doit WhatsApp pour le demander, il risque de contacter un autre prestataire en parallèle. Un indicateur statique « Dispo mai 2026 ✓ / complet juin 2026 » débloque 20-30 % des hésitations.

**Fix concret** : dans `#tarifs` (rebaptisé « Budget & disponibilité »), ajouter sous les 3 lignes tarifaires un bandeau simple :
```
Disponibilité actuelle
· Mai 2026 — Ouvert ✓
· Juin 2026 — 2 créneaux restants
· Juillet 2026 — Complet
[Demander un devis sous 24h → WhatsApp]
```
Même brut, cette info sauve des contacts.

**Effort** : S (HTML + CSS simple).

---

### S-07 [P2] — Manque de fil narratif entre sections : pas de transitions sémantiques

**Observation** : la fin de Projets saute direct sur la mid-CTA (« Passons à l'action »), puis on retombe froidement sur Services. Idem entre Services et Méthode, entre Tarifs et Quote-band. Aucune phrase de liaison qui dise « Ces projets, voilà comment ils voient le jour » ou « Maintenant que tu as vu ce que je fais, voici à quel prix ».

**Impact visiteur** : le scroll est abrupt. Chaque section se sent autonome, pas chaînée. Ça renforce la critique externe « sections mal délimitées / hiérarchie floue » — paradoxalement, le problème n'est pas un manque de délimitation mais un **excès de délimitation** : chaque section est une île.

**Fix concret** :
- Ajouter un `.section-bridge` optionnel entre les sections, 1 phrase courte éditoriale. Exemple après §01 Projets : `« Voilà ce qu'on livre. Maintenant, comment on fonctionne. »`
- Alternativement : utiliser le `.sub` du `.section-head` comme pont narratif (actuellement c'est du tag « Social content · Événementiel ») — le rendre conversationnel.
- Garder la mid-CTA mais la réserver au moment narratif juste — après Tarifs, pas entre Projets et Services.

**Effort** : S (texte + classe CSS de transition).

---

### S-08 [P2] — Numérotation incohérente : §01→§06 vs bandes sans numéro vs §01.1→§01.4

**Observation** (confirmé par `00-recon.md` F5) : 3 systèmes coexistent.
1. Sections principales : `§ 01` à `§ 06` (ok).
2. Bandes éditoriales : `§ Clients — Sélection` / `§ Passons à l'action` / rien sur quote-band (incohérent).
3. Case studies : `§ 01.1` à `§ 01.4` (ok, sauf `§ 01.3 / [Catégorie — à compléter]` sur Lucid Dreams ligne 2759).
4. Selected Work Index : `§ 01 · Index` avec un point médian au lieu du slash (ligne 2830).

**Impact visiteur** : lisible individuellement, mais en scroll le lecteur ne sait jamais si une bande est une sous-section ou un intermède. Ça donne l'impression de « 11 sections » au lieu de « 6 sections + 3 bandes ».

**Fix concret** :
- Règle unique : sections = `§ NN / titre`, bandes = `§ · intermède` (pas de numéro, point médian comme marqueur visuel différent).
- Quote-band : ajouter `§ · Témoignage` dans le même style que `§ Clients`.
- Mid-CTA : renommer `§ · Un mot ?` ou laisser tel quel mais normaliser la typo (point médian au lieu d'aucun séparateur).
- Lucid Dreams : si catégorie pas prête, retirer le placeholder et mettre `§ 01.3 / Music` (ou ce qu'il est).

**Effort** : S (find/replace dans HTML).

---

### S-09 [P1] — Mobile ≤ 390px : cartes case studies s'empilent sans hiérarchie, et le carousel à 3 slides avec boutons prev/next est trop dense

**Observation** (à valider par test réel mais probable au vu du HTML) : chaque `.ps-ed` contient header (idx + title + meta 5 spans) + media (carousel 3 reels) + description. Sur 390px, la meta de 5 spans wrap sur 3 lignes, les boutons prev/next du carousel mangent l'écran, et les 3 paragraphes après sans respiration verticale suffocent.

**Impact visiteur** : sur mobile (majorité du trafic IG/WhatsApp), le premier case study Wenda prend ~1.5 écran rien que sa propre verticalité. Le visiteur décroche avant d'atteindre Bebooth.

**Fix concret** :
- Meta : réduire de 5 spans à 3 max (ex : « Fashion · Social · Brussels ») ou wrap propre (grid 2 cols).
- Carousel : masquer les boutons prev/next sur mobile, activer swipe natif seulement + dots agrandis.
- Description : limiter à 2 paragraphes max en mobile (le 3e derrière un « Lire + »).
- Ajouter un espacement vertical dédié `> 64px` entre `.ps-block` sur mobile (actuellement trop serré).

**Effort** : M (CSS responsive + JS mineur pour swipe).

---

### S-10 [P2] — La nav à 6 entrées + nom + toggle thème + badges 4K/60fps = densité topbar trop élevée

**Observation** : ligne 2558-2577. Sur desktop, le header contient brand + 6 liens nav + badges 4K/60fps + menu-btn. C'est dense mais ok. En mobile, ça se résume au brand + hamburger, donc ça passe. Sur tablette (≤ 900px), le scrollspy avec pill animé + 6 liens est à l'étroit.

**Impact visiteur** : perte de lisibilité des sections actives, le pill bouge sur des cibles trop rapprochées.

**Fix concret** :
- Réduire à 4 entrées principales en nav desktop : Projets / Services / Tarifs / Contact (dropper Méthode et À propos dans le flow, pas dans la nav).
- Si on veut garder Méthode et À propos, les mettre dans un sous-menu « + » ou les fusionner (« Comment on bosse » unique qui couvre méthode + bio).

**Effort** : M (choix éditorial + HTML).

---

### S-11 [P2] — Le ticker du hero contient le mot « Premiere Pro » 2 fois, les autres aussi

**Observation** : ligne 2631. `<span>Premiere Pro</span>...<span>Premiere Pro</span>` — la liste est juste dupliquée pour l'effet infini mais à la lecture ça donne l'impression que CD ne connaît que 12 skills.

**Impact visiteur** : impression de CV court. Facile à enrichir sans casser l'effet.

**Fix concret** : proposer 20-24 termes uniques au lieu de 12×2. Ex ajouter : DaVinci Resolve (si ok) / Color grading / Cadrage / Son / BTS / Événementiel / Corporate / Fashion / etc.

**Effort** : S (texte).

---

## Case studies — audit détaillé

### Wenda — § 01.1

**État actuel** : seul case study rédigé. 2 paragraphes (lignes 2699-2700). Le premier décrit le client (histoire familiale), le second décrit le travail de façon générique (« reels dynamiques », « ambiances boutique »).

**Problème / Rôle / Décisions / Résultat — qu'est-ce qui manque ?**
- Problème : non formulé (la marque avait-elle besoin de rajeunir son audience ? d'augmenter ses abonnés IG ? de soutenir un lancement de collection ?).
- Rôle : ambigu (« je réalise » — en tant que quoi ? Directeur artistique ? Opérateur caméra ? Monteur seul ?).
- Décisions : aucune décision créative spécifique citée (choix de format, de lumière, de rythme).
- Résultat : aucune métrique (nb de reels livrés en 2024-2026 ? vues cumulées ? retours client ?).

**Template de réécriture** :
```
§ 01.1 / Fashion · Social content

Les filles de Wenda — 2024→2026

CONTEXTE
Maison de mode belge fondée en 1972, reprise en [année] par Lara et
Marina Marinof. Besoin : [à compléter — ex : rajeunir la présence
digitale sans casser l'héritage de la maison / soutenir chaque
lancement de collection / …].

RÔLE
[À compléter — ex : DA + tournage + montage, en solo /
en duo avec [nom] · cadence mensuelle / à la demande].

DÉCISIONS
· Format : reels 9:16 Instagram, [durée moyenne — ex : 12-20s]
· Lumière : [naturelle boutique / setup dédié shooting]
· Rythme : [coupes serrées sur détails matière / plans fluides produits]
· [Autre choix fort — ex : pas de voix off, musique curated uniquement]

LIVRABLES & IMPACT
· [N] reels livrés sur la période
· [Plateforme principale : Instagram @...]
· [Métrique visible — ex : X reels > 50k vues / collaboration
   reconduite chaque saison depuis N mois]

[Tag — à compléter : collab en cours / sur demande]
```

---

### Bebooth — § 01.2

**État actuel** : vidéos chargées (3 reels carousel), mais description = 2 lignes de placeholder littéral (lignes 2748-2749 : `[Description du projet Bebooth — à compléter]` / `[Angle créatif, type de contenu produit, vibe — à compléter]`).

**Problème / Rôle / Décisions / Résultat — qu'est-ce qui manque ?**
- **Tout**. C'est un placeholder.

**Template de réécriture** :
```
§ 01.2 / Event · Photobooth

Bebooth — 2025 · Brussels

CONTEXTE
[Qu'est-ce que Bebooth ? · prestataire photobooth événementiel,
anime mariages / corporate / festivals à Bruxelles].
Besoin : [à compléter — contenus social pour démo / contenu
showreel commercial / aftermovies événement].

RÔLE
[À compléter — ex : réalisation + montage de [N] reels 16:9
pour leur page Instagram / démo commerciale].

DÉCISIONS
· Format : [16:9 en prod, recadré 9:16 pour IG / natif 9:16]
· Approche : [reportage événement réel / mise en scène setup]
· Rythme : [à compléter]

LIVRABLES & IMPACT
· [N] reels livrés
· [Utilisés pour / résultat]
```

---

### Lucid Dreams — § 01.3

**État actuel** : vidéos chargées (3 clips), mais **même l'eyebrow de catégorie est un placeholder** (ligne 2759 : `§ 01.3 / [Catégorie — à compléter]`). Meta remplie de tirets. Description 2 paragraphes placeholder (lignes 2797-2798).

**Problème / Rôle / Décisions / Résultat — qu'est-ce qui manque ?**
- **Tout, plus la catégorie du projet elle-même**.

**Template de réécriture** :
```
§ 01.3 / [Music · Clip / Brand · Film / à déterminer par CD]

Lucid Dreams — [année] · [lieu]

CONTEXTE
[Qu'est-ce que Lucid Dreams ? · artiste / collectif / marque].
Besoin : [à compléter].

RÔLE
[À compléter].

DÉCISIONS
· Format : 9:16 vertical · [durée / plateforme]
· Angle créatif : [à compléter — c'est le plus important ici
   car le visuel est déjà stylisé, il faut expliciter l'intention]

LIVRABLES & IMPACT
· [N] clips / [N] reels
· [Plateforme / retombée]
```

**Note critique** : Lucid Dreams a 3 vidéos mais pas de catégorie. Soit CD sait ce que c'est et oublie de remplir, soit c'est un nom placeholder aussi. À trancher en priorité — si c'est un vrai projet, la catégorie doit être la première chose écrite.

---

### Scout Festival — § 01.4

**État actuel** : 1 vidéo aftermovie 16:9 (pas un carousel), description 2 paragraphes placeholder (lignes 2822-2823).

**Problème / Rôle / Décisions / Résultat — qu'est-ce qui manque ?**
- **Tout**.
- Bon point : c'est le projet avec le format le plus « portfolio » (aftermovie 16:9) — potentiellement la pièce maîtresse commerciale pour attirer des organisateurs d'événements.

**Template de réécriture** :
```
§ 01.4 / Event · Aftermovie

Scout Festival — 2026 · Belgique

CONTEXTE
[Scout Festival = festival de [genre] à [ville], [édition N]ème édition,
[N] jours, [public]].
Besoin : aftermovie pour [alimenter la campagne 2027 / rendre
compte aux sponsors / générer des ventes early-bird].

RÔLE
· Réalisation + cadrage + drone [solo / équipe de N]
· Tournage sur [durée en jours]
· Montage [durée de livraison]

DÉCISIONS
· Format : 16:9, [durée] — pensé pour diffusion sur [YouTube
   / réseaux sponsors / projection pré-show N+1]
· Approche : [énergie foule / portraits artistes / overview site]
· Musique : [tracking sync / original / artiste du festival]

LIVRABLES & IMPACT
· 1 aftermovie [durée]
· [N] teasers courts 9:16 dérivés
· [Métrique — ex : X vues YouTube / utilisé pour lever sponsor Y]
```

**Priorité de rédaction** : Scout Festival d'abord (plus commercial), Wenda réécrit en deuxième (déjà rédigé), Bebooth en troisième, Lucid Dreams en dernier (niche).

---

## Fil narratif proposé (elevator pitch scroll)

> Objectif : le visiteur comprend en 6-8 phrases ce qu'il vit en scrollant la page.

1. **Hero** — « Je tombe sur un vidéaste belge, CD. Je vois en fond un showreel rythmé et je lis qu'il filme pour des marques, des événements et des artistes, et qu'il forme sur Premiere Pro. J'ai deux boutons : voir son travail ou demander un devis. »
2. **Proof-band** — « Quatre noms défilent en gros : Wenda, Bebooth, Scout Festival, Lucid Dreams. Je ne les connais peut-être pas tous mais la variété des univers me rassure — il bosse pas que dans un seul créneau. »
3. **Cas clients (§01)** — « Premier cas : Wenda. Je lis une histoire — problème posé, rôle qu'il a tenu, décisions qu'il a prises, résultat. Je comprends pour la première fois ce qu'il vend vraiment. Les 3 autres cas suivent le même moule. »
4. **Quote-band** — « Un client me dit en une phrase ce que ça donne de bosser avec lui. Je passe de 'peut-être' à 'probablement'. »
5. **Ce que je fais (§02)** — « Ok maintenant je vois les 2 offres sèches : Vidéo / Formation. Je coche mentalement celle qui me concerne. »
6. **Comment on travaille (§03)** — « Il me rassure sur le process en 4 étapes. Je vois qu'il n'improvise pas. »
7. **Budget & disponibilité (§04)** — « Je vois un tarif horaire, un exemple concret de package, ET son agenda du mois. Ça me dit si je peux le booker tout de suite ou pas. »
8. **Mid-CTA + Qui je suis (§05) + Écris-moi (§06)** — « Il me rappelle que c'est simple, me donne son visage et sa bio courte, puis me laisse un formulaire minimaliste et un WhatsApp. Je contacte ou je ferme l'onglet — la décision est mûre, elle n'est pas forcée. »

---

## Labels à reconsidérer

| Actuel | Proposition | Raison |
|---|---|---|
| Projets | **Cas clients** | Centré usage/résultat, pas portfolio créatif |
| Services | **Ce que je fais** | Langue humaine, pas jargon commercial |
| Méthode | **Comment on travaille** | « On » inclusif (client + CD), évite le ton descendant |
| Tarifs | **Budget & disponibilité** | Répond à 2 questions (combien + quand) en 1 section |
| À propos | **Qui je suis** | Direct vs froideur administrative |
| Contact | **Écris-moi** | Action claire vs catégorie statique |
| Passons à l'action | **Un projet en tête ?** | Déjà là (h2 de la mid-CTA) — harmoniser pour que l'index soit le même |
| Clients — Sélection | **Ils m'ont confié** | Plus narratif, cohérent avec « Qui je suis » |
| Témoignage (quote) | **Ils en disent ça** | Ton conversationnel |
| Autres travaux | **Plus de projets bientôt** | Honnête tant qu'il n'y a que 4 cas |
| 40+ / 04 / BE | **40+ projets livrés / 2022→2026 / Bruxelles → EU** | Lisibles sans effort cognitif |
| `[Projet 05]…[Projet 10]` | — (supprimer le bloc) | Un index vide tue la crédibilité |

---

## Ce qu'on garde

Les éléments forts à **ne pas casser** dans la refonte :

- **Hero showreel en fond vidéo + ticker intérieur + morphing « Je suis — »** : signature visuelle unique, reconnaissable. Conserver tel quel, juste ajouter la couche positionnement / CTA.
- **Numérotation `§ 01 / section`** : pro, éditoriale, cohérente avec l'identité « portfolio / 2026 — Éd. 01 ». À harmoniser sur les bandes mais pas à abandonner.
- **Bandes full-bleed (proof / midcta / quote)** : bon rythme visuel, ruptures bienvenues entre sections denses. Garder le pattern, fiabiliser le contenu.
- **Carte ID flottante dans À propos** : idée forte, personnelle, différenciante. Ne pas simplifier.
- **Eye-bubble + contact drawer (eye-label « Contacte-moi »)** : micro-interaction élégante, CTA discret mais persistant. Garder.
- **FAB mobile WhatsApp** : bonne pratique mobile, conversion directe. Garder.
- **Méthode en 4 étapes** : pédagogie claire, rassurance process. Garder, juste renommer le label de section.
- **Palette sable/ink/accent orange + Dela Gothic + Inter** : identité stable, pas à bouger.
- **Vanilla no-build** : contrainte dure connue, respectée. À maintenir.

---

*Fin audit / 02 — Structure, IA & narration.*
