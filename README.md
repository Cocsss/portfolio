# Portfolio — CD Studio

Site portfolio de Corentin Devillé / CD Studio, hébergé gratuitement sur Cloudflare Pages, vidéos servies direct depuis Cloudflare R2.

## Stack

- **Site** : `index.html` (vanilla HTML/CSS/JS, zéro dépendance)
- **Hébergement site** : Cloudflare Pages (gratuit, BP illimitée)
- **Hébergement vidéo** : Cloudflare R2 (10 Go gratuits, egress gratuit)
- **Domaine** : Cloudflare Registrar (à prix coûtant)

## Architecture des vidéos

Les vidéos sont servies en **direct** dans le HTML via `<video>` natif :

```html
<video autoplay muted loop playsinline preload="metadata"
       poster="assets/bebooth-poster.jpg">
  <source src="https://videos.tondomaine.com/bebooth.mp4" type="video/mp4">
</video>
```

Aucune iframe, aucun player tiers, aucun logo, aucune pub.

Le sous-domaine `videos.tondomaine.com` pointe vers le bucket R2.

## Déploiement — étapes

Voir `DEPLOY.md` pour la procédure complète.

## Structure

```
.
├── index.html              Site complet
├── assets/                 Images fixes (portrait, logos, posters)
├── media/
│   ├── videos/             (gitignoré — vit sur R2, pas ici)
│   ├── photos/             Photos utilisées dans le site
│   └── thumbnails/         Vignettes projets
└── README.md
```
