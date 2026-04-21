# AiDN Agency — site vitrine

Site statique mono-page déployé sur Vercel à l'adresse [www.aidnagency.fr](https://www.aidnagency.fr/).

## Structure

```
aidn.ai-main/
├── index.html                       Page d'accueil (contenu principal)
├── 404.html                         Page d'erreur 404 custom
├── mentions-legales.html            Mentions légales (LCEN)
├── politique-confidentialite.html   Politique de confidentialité (RGPD)
│
├── vercel.json                      Configuration Vercel (headers, cache)
├── robots.txt                       Directive crawlers
├── sitemap.xml                      Plan du site pour Google
├── site.webmanifest                 Manifest PWA (Android, iOS)
├── favicon.ico                      Favicon multi-résolution (16/32/48)
│
├── assets/
│   ├── icons/                       Icônes exportées depuis "Signe A - Noir"
│   │   ├── favicon-16.png           Onglet HiDPI
│   │   ├── favicon-32.png           Onglet standard
│   │   ├── favicon-48.png           Windows taskbar
│   │   ├── apple-touch-icon.png     iOS homescreen 180×180
│   │   ├── icon-192.png             Android / PWA
│   │   └── icon-512.png             Android splash / maskable
│   └── images/
│       └── logo.png                 Logo horizontal local (fallback)
│
└── README.md                        Ce fichier
```

## Stack

- HTML5 + CSS pur + JavaScript vanilla — **zéro dépendance**
- Polices Google Fonts : Inter
- Images & CDN : Cloudinary (avec `f_auto,q_auto` → WebP/AVIF automatique)
- Analytics : Vercel Web Analytics + Speed Insights (sans cookies)

## Déploiement

### Via Vercel CLI
```bash
vercel --prod
```

### Via Git
Push sur la branche connectée au projet Vercel, le déploiement est automatique.

### Domaine
- Production : `www.aidnagency.fr`
- Redirection à configurer : `aidnagency.fr` → `www.aidnagency.fr` (308) via Dashboard Vercel → Domains

## Configuration Vercel (`vercel.json`)

- **cleanUrls** : `/mentions-legales` au lieu de `/mentions-legales.html`
- **Headers sécurité** : HSTS, X-Frame-Options, Referrer-Policy, Permissions-Policy
- **Cache** : 1 an pour images et fonts, 30 jours pour CSS/JS, no-cache pour HTML

## SEO

- `sitemap.xml` déclare 3 URLs (accueil + 2 pages légales)
- `robots.txt` référence le sitemap
- JSON-LD Organization dans l'`index.html`
- Open Graph + Twitter Card avec image 1200×630 via Cloudinary

## Accessibilité

- Skip link clavier
- Landmark `<main>`, `<nav>`, `<footer>`
- `aria-label` / `aria-expanded` / `aria-controls` sur les éléments interactifs
- `prefers-reduced-motion` désactive les animations pour les utilisateurs sensibles
- Focus visible avec outline personnalisé

## ⚠️ À compléter avant mise en ligne publique

- [ ] Remplacer le numéro WhatsApp placeholder `+33 6 12 34 56 78` (8 occurrences dans `index.html`)
- [ ] Compléter les champs `[À COMPLÉTER]` dans `mentions-legales.html` (raison sociale, SIRET, RCS, adresse, etc.)
- [ ] Compléter les champs `[À COMPLÉTER]` dans `politique-confidentialite.html` (responsable du traitement)
- [ ] Remplacer les images placeholder `placehold.co` des 4 modals projet dans le JavaScript de `index.html`
- [ ] Activer Vercel Web Analytics dans le dashboard du projet

## Scripts utiles

```bash
# Tester en local (http://localhost:8000)
python3 -m http.server 8000

# Vérifier les liens cassés
grep -rn 'href="[^"#]' index.html | grep -v "http"

# Voir la taille totale du déploiement
du -sh .
```
