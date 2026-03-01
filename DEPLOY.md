# DEPLOY.md — Guide de mise en ligne complet
## Le Chocolatier · lechocolatiersuisse.ch

---

## Prérequis

- Node.js ≥ 20 installé (`node -v`)
- pnpm installé (`npm install -g pnpm`)
- Compte GitHub
- Compte Vercel (gratuit suffit)
- Accès au DNS de `lechocolatiersuisse.ch`

---

## ÉTAPE 1 — Créer le projet Astro

```bash
# 1. Scaffold
pnpm create astro@latest le-chocolatier -- \
  --template minimal \
  --typescript strict \
  --no-install \
  --no-git

cd le-chocolatier

# 2. Dépendances
pnpm install
pnpm add @astrojs/tailwind @astrojs/sitemap tailwindcss
pnpm add -D @astrojs/check typescript sharp

# 3. Vérifier
pnpm run dev
# → http://localhost:4321 doit afficher Astro
```

---

## ÉTAPE 2 — Copier les fichiers de ce repo

```
le-chocolatier/
├── .github/
│   ├── copilot-instructions.md   ← copier depuis ce repo
│   └── workflows/
│       └── deploy.yml            ← copier depuis ce repo
├── public/
│   └── (vide pour l'instant, ajouter favicon.svg)
├── src/
│   └── pages/
│       └── index.astro           ← transposer depuis le prototype HTML
├── SPECS.md                      ← copier depuis ce repo
├── astro.config.mjs              ← voir ci-dessous
├── tailwind.config.mjs           ← voir SPECS.md section 2
└── vercel.json                   ← voir ci-dessous
```

### astro.config.mjs

```javascript
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://lechocolatiersuisse.ch',
  integrations: [
    tailwind({ applyBaseStyles: false }),
    sitemap({
      filter: (page) => !page.includes('/merci'),
      i18n: {
        defaultLocale: 'fr',
        locales: {
          fr: 'fr-CH',
          en: 'en-US',
          de: 'de-CH',
        },
      },
    }),
  ],
  i18n: {
    defaultLocale: 'fr',
    locales: ['fr', 'en', 'de'],
    routing: {
      prefixDefaultLocale: false,
    },
  },
  image: {
    service: { entrypoint: 'astro/assets/services/sharp' },
  },
  compressHTML: true,
});
```

### vercel.json

```json
{
  "buildCommand": "pnpm run build",
  "outputDirectory": "dist",
  "installCommand": "pnpm install",
  "framework": "astro",
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" }
      ]
    }
  ],
  "redirects": [
    {
      "source": "/",
      "has": [{ "type": "host", "value": "www.lechocolatiersuisse.ch" }],
      "destination": "https://lechocolatiersuisse.ch/",
      "permanent": true
    }
  ]
}
```

---

## ÉTAPE 3 — Créer le repo GitHub

```bash
cd le-chocolatier
git init
git add .
git commit -m "feat: initial scaffold"

# Sur github.com → New repository
# Nom : le-chocolatier (private recommandé)

git remote add origin https://github.com/[TON-USERNAME]/le-chocolatier.git
git branch -M main
git push -u origin main
```

---

## ÉTAPE 4 — Connecter Vercel

### Option A : Via l'interface Vercel (recommandé pour démarrer)

1. Aller sur [vercel.com](https://vercel.com) → **Add New → Project**
2. **Import Git Repository** → sélectionner `le-chocolatier`
3. Framework Preset : **Astro** (auto-détecté)
4. Build Command : `pnpm run build`
5. Output Directory : `dist`
6. **Deploy** → premier déploiement en ~2 minutes

### Option B : Via CLI (pour récupérer les IDs pour GitHub Actions)

```bash
# Installer Vercel CLI
npm i -g vercel

# Lier le projet
cd le-chocolatier
vercel link

# → Crée .vercel/project.json avec :
# { "projectId": "prj_xxxx", "orgId": "team_xxxx" }
```

---

## ÉTAPE 5 — Configurer GitHub Actions

### Récupérer les 3 valeurs nécessaires

| Secret | Où le trouver |
|---|---|
| `VERCEL_TOKEN` | vercel.com → Settings → Tokens → Create |
| `VERCEL_ORG_ID` | `.vercel/project.json` → `orgId` |
| `VERCEL_PROJECT_ID` | `.vercel/project.json` → `projectId` |

### Les ajouter dans GitHub

```
Repo GitHub → Settings → Secrets and variables → Actions → New repository secret
```

Ajouter les 3 secrets :
- `VERCEL_TOKEN`
- `VERCEL_ORG_ID`
- `VERCEL_PROJECT_ID`

### Tester le pipeline

```bash
# Faire un commit sur main
git commit --allow-empty -m "test: trigger CI/CD"
git push

# → Aller dans GitHub → Actions → voir le workflow tourner
# → 3 étapes : quality check → build → deploy production
```

---

## ÉTAPE 6 — Domaine personnalisé

### Dans Vercel

1. **Project → Settings → Domains**
2. Ajouter `lechocolatiersuisse.ch`
3. Vercel affiche les DNS records à ajouter

### Chez votre registraire DNS (SWITCH, Infomaniak, Cloudflare...)

```
Type  : A
Name  : @
Value : 76.76.21.21   (IP Vercel)

Type  : CNAME
Name  : www
Value : cname.vercel-dns.com
```

*La propagation DNS prend 5 min à 24h selon le registraire.*

---

## ÉTAPE 7 — Vérifications post-déploiement

```bash
# Lighthouse (depuis Chrome DevTools ou)
npx lighthouse https://lechocolatiersuisse.ch --view

# Structured data
# → https://validator.schema.org → tester l'URL

# OG image
# → https://opengraph.xyz → coller l'URL

# Formulaire de contact
# → Remplir et soumettre → vérifier réception email
# (configurer dans Vercel → Project → Forms → Notification email)
```

---

## Workflow quotidien (une fois tout en place)

```bash
# Nouvelle feature
git checkout -b feature/ma-feature
# ... coder ...
git commit -m "feat: description"
git push

# → GitHub Actions crée automatiquement une URL de preview
# → Vérifier le rendu trilingue

# Merger en production
git checkout main
git merge feature/ma-feature
git push
# → Deploy production automatique
```

---

## Notes sur les images temporaires (avant shooting)

Tant que les vraies photos d'Édouard ne sont pas disponibles, utiliser dans les composants :

```astro
<!-- PLACEHOLDER: remplacer par vraie photo après shooting mars 2026 -->
<div class="photo-placeholder" style="background: #2B1B12; aspect-ratio: 3/4;">
  <span style="color: rgba(200,145,90,0.3); font-size: 0.7rem; letter-spacing: 0.2em;">
    PHOTO À VENIR
  </span>
</div>
```

Pour générer des visuels de chocolat temporaires avec l'API Anthropic (Claude) :
- Utiliser le prototype HTML fourni — l'exploded view SVG est autonome et original
- Les images de service peuvent être des aplats de couleur avec grain texture
- Éviter absolument les images Unsplash/Pexels de "chocolat générique"

---

*Guide v1.0 — Mars 2026 · Le Chocolatier*
