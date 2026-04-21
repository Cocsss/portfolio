# Déploiement — guide pas à pas

## Ce que tu vas faire

1. Créer un compte Cloudflare + un compte GitHub
2. Pousser le code sur GitHub
3. Connecter GitHub → Cloudflare Pages (site live)
4. Acheter ton domaine chez Cloudflare
5. Créer le bucket R2 pour les vidéos
6. Uploader tes vidéos
7. Remplacer les placeholders dans `index.html`

**Temps total : ~30 minutes la première fois.**
**Coût : ~9,80 €/an (juste le domaine).**

---

## 1. Comptes — 5 minutes

### GitHub
1. Va sur https://github.com/signup — inscris-toi (gratuit)
2. Vérifie ton email

### Cloudflare
1. Va sur https://dash.cloudflare.com/sign-up — inscris-toi
2. Vérifie ton email

---

## 2. Initialiser le repo Git et le pousser sur GitHub

Ouvre le Terminal (Applications → Utilitaires → Terminal sur Mac), puis :

```bash
cd ~/Desktop/PORTFOLIO
git init -b main
git add .
git commit -m "Initial commit"
```

Sur GitHub :
1. Clique sur le `+` en haut à droite → **New repository**
2. Nom : `portfolio` (ou ce que tu veux)
3. Laisse tout en **Public** (requis pour Pages free)
4. **NE coche PAS** "Add README" ni ".gitignore" (on les a déjà)
5. Clique **Create repository**

GitHub te montre ensuite les commandes à coller. Ce sera du genre :

```bash
git remote add origin https://github.com/TON-PSEUDO/portfolio.git
git branch -M main
git push -u origin main
```

Colle-les dans ton Terminal. Ton code est maintenant sur GitHub.

---

## 3. Déployer sur Cloudflare Pages

1. Dashboard Cloudflare → **Workers & Pages** (dans le menu de gauche)
2. Clique **Create application** → onglet **Pages** → **Connect to Git**
3. Autorise Cloudflare à accéder à tes repos GitHub
4. Sélectionne `portfolio`
5. Dans la config de build :
   - **Framework preset** : None
   - **Build command** : *(laisser vide)*
   - **Build output directory** : `/`
6. Clique **Save and Deploy**
7. 30 secondes plus tard, ton site est live sur `portfolio-xxx.pages.dev`

---

## 4. Acheter ton domaine

1. Dashboard Cloudflare → **Domain Registration** → **Register Domains**
2. Tape ton nom voulu (ex: `cdstudio.com`, `corentindeville.com`)
3. Ajoute au panier → paie (~9,80 € pour un `.com`)
4. Le domaine apparaît dans **Websites** dans le menu

### Brancher le domaine au site

1. Retourne dans **Workers & Pages** → ton projet `portfolio`
2. Onglet **Custom domains** → **Set up a custom domain**
3. Tape `tondomaine.com` → Continue → Activate
4. Cloudflare configure tout automatiquement (DNS + SSL)
5. Attends 1-5 minutes → ton site est sur `https://tondomaine.com`

Ajoute aussi `www.tondomaine.com` pour que les deux marchent.

---

## 5. Créer le bucket R2 pour les vidéos

### Activer R2 (première fois seulement)

1. Dashboard Cloudflare → **R2** dans le menu
2. Si c'est la première fois, Cloudflare te demande d'ajouter une **carte bancaire** même pour le plan gratuit (anti-abus). La carte ne sera **pas débitée** tant que tu restes sous 10 Go.
3. Accepte et continue

### Créer le bucket

1. **Create bucket** → nom : `cd-videos` (ou autre)
2. Location : Automatic
3. **Create**

### Rendre le bucket accessible en public via un sous-domaine

1. Dans ton bucket → onglet **Settings**
2. Section **Public access** → **Custom Domains** → **Connect Domain**
3. Tape `videos.tondomaine.com` (sous-domaine libre)
4. Cloudflare configure le DNS automatiquement → SSL actif en 1-2 minutes

---

## 6. Uploader tes vidéos

1. Dans ton bucket R2 → **Objects** → bouton **Upload**
2. Drag & drop tes `.mp4` directement dans la zone
3. Chaque fichier uploadé est dispo à `https://videos.tondomaine.com/nomdufichier.mp4`

**Important** :
- Nomme les fichiers en minuscules sans espaces ni accents : `bebooth-reel.mp4`, `aftermovie-event.mp4`, etc.
- Tu peux uploader des dossiers entiers d'un coup

---

## 7. Brancher les vidéos dans `index.html`

Ouvre `index.html` et cherche les placeholders de vidéos.

### Exemple — un reel auto-play

Remplace une balise `<div class="reel-ph">...</div>` ou équivalent par :

```html
<video class="reel" autoplay muted loop playsinline preload="metadata"
       poster="assets/bebooth-poster.jpg">
  <source src="https://videos.tondomaine.com/bebooth-reel.mp4" type="video/mp4">
</video>
```

### Exemple — un aftermovie avec contrôles

```html
<video class="aftermovie" controls preload="metadata"
       poster="assets/aftermovie-poster.jpg">
  <source src="https://videos.tondomaine.com/aftermovie-event.mp4" type="video/mp4">
  Ton navigateur ne supporte pas la vidéo HTML5.
</video>
```

### Pousser les changements sur le site

```bash
cd ~/Desktop/PORTFOLIO
git add index.html
git commit -m "Branche vidéos R2"
git push
```

Cloudflare Pages redéploie automatiquement en 30 secondes. Ton site est à jour.

---

## Workflow pour la suite

À chaque modification :

```bash
cd ~/Desktop/PORTFOLIO
git add .
git commit -m "message descriptif"
git push
```

→ site redéployé auto.

Pour ajouter une vidéo : upload dans R2 + lien dans `index.html` + push.

---

## Commandes utiles

```bash
# Voir les fichiers modifiés
git status

# Voir les changements
git diff

# Annuler un fichier non-pushé
git checkout -- fichier.html

# Historique
git log --oneline
```

---

## Dépannage

- **"Pages build failed"** → vérifie que le build command est vide et l'output directory est `/`
- **Domaine n'est pas accessible** → attendre 5-10 min pour la propagation DNS
- **Vidéo ne se charge pas** → vérifie que le bucket est en "Public access" et le custom domain bien configuré. Teste l'URL directement dans un onglet.
- **R2 facturé** → vérifie ton usage dans R2 → Overview. Sous 10 Go = gratuit.

---

## Coût final

| Poste | Coût/an |
|---|---|
| Domaine `.com` (Cloudflare Registrar) | ~9,80 € |
| Cloudflare Pages | 0 € |
| Cloudflare R2 (sous 10 Go) | 0 € |
| SSL, CDN, DNS | 0 € |
| **Total** | **~9,80 €/an** |
