# SPECS.md — Le Chocolatier / Édouard Morand
## Site web lechocolatiersuisse.ch — Spécifications techniques complètes

> Ce document est le contrat de build du site.
> Il doit être lu en intégralité avant toute session de développement.
> Il complète `.github/copilot-instructions.md` avec le détail des composants, des données et des décisions d'architecture.

---

## Table des matières

1. [Vision produit](#1-vision-produit)
2. [Architecture technique](#2-architecture-technique)
3. [Design tokens](#3-design-tokens)
4. [Composants UI](#4-composants-ui)
5. [Pages — détail complet](#5-pages--détail-complet)
6. [Content collections](#6-content-collections)
7. [SEO & Structured Data](#7-seo--structured-data)
8. [Formulaire de contact](#8-formulaire-de-contact)
9. [Performance & build](#9-performance--build)
10. [Internationalisation](#10-internationalisation)
11. [Checklist de lancement](#11-checklist-de-lancement)

---

## 1. Vision produit

### Objectif principal
Convertir des visiteurs qualifiés (directeurs de maisons de luxe, chocolatiers artisanaux, responsables achats grande distribution) en **prospects qui prennent contact**.

### Objectifs secondaires
- Établir Édouard Morand comme la référence suisse du consulting chocolat haut de gamme
- Démontrer la double capacité : **artisanat exceptionnel** ET **échelle industrielle**
- Construire une légitimité SEO sur les requêtes B2B francophones et anglophones

### KPIs attendus (à 6 mois)
| Métrique | Cible |
|---|---|
| Taux de conversion visiteur → formulaire | ≥ 2% |
| Durée moyenne de session | ≥ 2:30 min |
| Pages par session | ≥ 2.5 |
| Position SEO "consultant chocolat suisse" | Top 3 |
| Lighthouse Performance (mobile) | ≥ 95 |

---

## 2. Architecture technique

### Stack décision

```
Astro 4.x       Choisi pour : génération statique, zero-JS par défaut,
                 intégration Tailwind native, performance maximale

Tailwind 3.x    Choisi pour : utility-first, pas de CSS mort,
                 design system via config, pas de composants pré-stylisés

TypeScript      Obligatoire : toutes les props typées, aucun `any`

Netlify         Choisi pour : formulaires gérés server-side sans backend,
                 déploiement continu depuis GitHub, edge CDN mondial
```

### Configuration Astro

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://lechocolatiersuisse.ch',
  integrations: [
    tailwind({ applyBaseStyles: false }),
    sitemap({
      filter: (page) => !page.includes('/merci'),
    }),
  ],
  image: {
    service: { entrypoint: 'astro/assets/services/sharp' },
  },
  compressHTML: true,
  build: {
    inlineStylesheets: 'auto',
  },
});
```

### Configuration Tailwind

```javascript
// tailwind.config.mjs
import { fontFamily } from 'tailwindcss/defaultTheme';

export default {
  content: ['./src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue}'],
  theme: {
    extend: {
      colors: {
        noir:    '#1A0F0A',
        brun:    '#2B1B12',
        marron:  '#5A3A2E',
        caramel: '#C8955A',
        creme:   '#FAF6F0',
        or:      '#B8973A',
      },
      fontFamily: {
        display: ['"Cormorant Garamond"', ...fontFamily.serif],
        body:    ['"DM Sans"', ...fontFamily.sans],
      },
      fontSize: {
        'display-xl': ['clamp(3rem, 8vw, 7rem)', { lineHeight: '1.05', letterSpacing: '-0.02em' }],
        'display-lg': ['clamp(2rem, 5vw, 4.5rem)', { lineHeight: '1.1', letterSpacing: '-0.01em' }],
        'display-md': ['clamp(1.5rem, 3vw, 2.5rem)', { lineHeight: '1.2' }],
      },
      spacing: {
        'section-sm': '4rem',
        'section-md': '6rem',
        'section-lg': '10rem',
      },
      maxWidth: {
        prose: '65ch',
        content: '72rem',
      },
    },
  },
  plugins: [],
};
```

---

## 3. Design tokens

### Typographie — règles d'application

| Usage | Font | Weight | Size | Transform |
|---|---|---|---|---|
| Hero H1 | Cormorant Garamond | 300 | display-xl | none |
| Section H2 | Cormorant Garamond | 400 | display-lg | none |
| Card H3 | Cormorant Garamond | 600 | display-md | none |
| Body prose | DM Sans | 300 | 1.125rem | none |
| Navigation | DM Sans | 400 | 0.875rem | UPPERCASE |
| Caption / meta | DM Sans | 300 | 0.75rem | UPPERCASE, tracking-widest |
| CTA button | DM Sans | 500 | 0.875rem | UPPERCASE, tracking-wider |

### Espace — Grid system

```
Mobile  (< 768px)  : 1 colonne, padding horizontal 1.5rem
Tablet  (768–1024) : 2 colonnes, padding horizontal 2.5rem  
Desktop (> 1024px) : jusqu'à 4 colonnes, max-w-content centré
```

### Hiérarchie des couleurs

```
Background page         : --color-creme   (#FAF6F0)
Background dark section : --color-brun    (#2B1B12)
Background footer       : --color-noir    (#1A0F0A)
Text principal          : --color-noir    (#1A0F0A)
Text sur fond sombre    : --color-creme   (#FAF6F0)
Accent / CTA            : --color-caramel (#C8955A)
Accent hover CTA        : --color-or      (#B8973A)
Border / séparateur     : --color-marron  (#5A3A2E) à 30% opacity
```

---

## 4. Composants UI

### 4.1 Button.astro

```
Variants  : primary | secondary | ghost
Sizes     : sm | md | lg
States    : default | hover | focus | disabled

Primary   : bg-caramel text-creme → hover:bg-or
Secondary : border border-caramel text-caramel → hover:bg-caramel/10
Ghost     : text-caramel → hover:underline

Props:
  href?     : string  (renders as <a>)
  type?     : 'button' | 'submit' | 'reset'
  variant?  : 'primary' | 'secondary' | 'ghost'
  size?     : 'sm' | 'md' | 'lg'
  class?    : string (passthrough)
```

### 4.2 Card.astro

```
Usage : Service cards, reference cards
Layout: icon (top) + title + text + optional link

Props:
  title    : string
  text     : string
  icon?    : string (SVG path or icon name)
  href?    : string
  tag?     : string (e.g. "Consulting", "Formation")
```

### 4.3 Header.astro

```
Behavior:
  - Transparent on hero (dark pages)
  - White/creme with shadow on scroll (light pages)
  - Sticky positioning
  - Mobile: hamburger menu → full-screen overlay

Nav links:
  - L'art du chocolat → /#expertise
  - Édouard Morand    → /edouard
  - Services          → /services
  - Références        → /references
  - Contact           → /contact

CTA in header: "Parlons de votre projet" (Button ghost → primary on scroll)
```

### 4.4 Hero.astro (Homepage)

```
Layout  : Full viewport height (min-h-screen)
BG      : Dark (#1A0F0A) with subtle grain texture overlay (CSS)
Content : Centered, max-w-4xl

Elements (top → bottom):
  1. Pre-headline  : "Édouard Morand" — DM Sans, uppercase, tracking-widest, text-caramel
  2. Headline      : "L'art du chocolat. / À un autre niveau." — Cormorant, display-xl, text-creme
  3. Sub-headline  : 1 sentence — DM Sans 300, text-creme/70
  4. CTA pair      : [Parlons de votre projet →] [Découvrir le parcours]
  5. Scroll cue    : Animated chevron or line

Animation (CSS only):
  - All elements start opacity-0, translateY(1rem)
  - Stagger: 0ms, 200ms, 400ms, 600ms delay
  - Duration: 800ms ease-out
```

### 4.5 Stats.astro (Chiffres clés)

```
5 stats horizontaux (scroll horizontal sur mobile)

Data:
  { value: "20+",       label: "années d'expérience",         sub: "en chocolaterie haut de gamme" }
  { value: "MOF 2015",  label: "Finaliste",                   sub: "Meilleur Ouvrier de France" }
  { value: "600–700t",  label: "de produit fini / an",        sub: "sourcing direct Cameroun" }
  { value: "Bean-to-Bar", label: "maîtrise complète",         sub: "de la fève à la tablette" }
  { value: "DBA",       label: "Doctorat en Business Adm.",   sub: "stratégie + savoir-faire" }

Style:
  - value    : Cormorant Garamond, display-md, text-caramel
  - label    : DM Sans 400, text-noir/creme selon fond
  - sub      : DM Sans 300, small, text-noir/50
  - Séparateur vertical léger entre stats (hidden sur mobile)
```

### 4.6 ServiceCard.astro

```
4 cartes pour les services principaux

Props:
  title    : string
  tagline  : string (courte, italique)
  points   : string[] (3 bullet points max)
  href     : string
  icon     : SVG inline

Layout: 2×2 sur desktop, 1 colonne sur mobile
Style: Fond creme, border caramel/20, hover → shadow-lg + border-caramel
```

### 4.7 Testimonial.astro

```
Quote block anonymisé

Props:
  quote    : string
  role     : string  (e.g. "Directeur Général, Maison de haute joaillerie, Genève")
  context? : string  (e.g. "Projet de consulting opérationnel 2023")

Style:
  - Grand guillemet Cormorant en arrière-plan (text-caramel/15, text-9xl)
  - Quote en Cormorant italic, text-xl
  - Role en DM Sans uppercase caption
```

---

## 5. Pages — détail complet

### 5.1 `index.astro` — Accueil

```
Sections dans l'ordre :
  1. Hero             (plein écran, sombre)
  2. Stats            (fond creme, 5 chiffres)
  3. Intro About      (2 colonnes: texte gauche + photo droite)
  4. Services Grid    (4 cartes)
  5. Luxury Banner    (full-width, sombre, citation Édouard)
  6. Testimonials     (2 quotes côte à côte)
  7. Contact CTA      (sombre, centré, bouton)

Meta:
  title       : "Le Chocolatier — Édouard Morand | Consultant Chocolat Haut de Gamme, Suisse"
  description : "Édouard Morand, finaliste MOF 2015, consultant en chocolaterie haut de gamme basé en Suisse. Création sur mesure, consulting industriel, formations et expériences de luxe."
```

### 5.2 `edouard.astro` — Biographie

```
Sections:
  1. Page Hero       (titre + sous-titre, fond sombre)
  2. Portrait        (photo + biographie longue, alternance gauche/droite)
  3. Parcours        (Timeline visuelle: Stettler → Cap-Eden-Roc → MOF → Jaeger → Fondation)
  4. Distinctions    (cards: MOF 2015 / DBA / Fondation Haute Chocolaterie)
  5. Philosophie     (grande citation Cormorant, fond caramel/10)
  6. CTA             (→ /contact)

Timeline data:
  [
    { year: "2003–", event: "Formation & premières expériences en France et Suisse" },
    { year: "~2010", event: "Chocolaterie Stettler, Genève — création de gammes" },
    { year: "~2012", event: "Hôtel du Cap-Eden-Roc, Antibes — chef chocolatier" },
    { year: "2015",  event: "Finaliste Meilleur Ouvrier de France — Chocolatier-Confiseur" },
    { year: "~2018", event: "Collaboration Jaeger-LeCoultre — création exclusive chocolat" },
    { year: "~2020", event: "Co-fondateur, Fondation Haute Chocolaterie" },
    { year: "2024",  event: "Doctorat en Business Administration (DBA)" },
    { year: "2026",  event: "Lancement de Le Chocolatier — sous son nom" },
  ]

⚠️ Dates approximatives : à valider par Édouard avant mise en ligne
```

### 5.3 `services/index.astro`

```
Vue d'ensemble des 4 services avec lien vers sous-pages.
Header section + grille 2×2 + CTA global.
```

### 5.4 `services/consulting.astro`

```
Titre     : "Consulting Opérationnel"
Tagline   : "Du concept à la mise en production — sans lacune."

Contenu:
  - Ce que c'est (3 paragraphes)
  - Pour qui (maisons de luxe, chocolatiers artisanaux, industriels)
  - Livrables : cahier des charges / recettes / process de production / suivi lancement
  - Capacités : jusqu'à 600–700t produit fini / sourcing direct Cameroun / bean-to-bar
  - Cas type (anonymisé)
  - CTA contact
```

### 5.5 `services/formations.astro`

```
Titre     : "Formations Technologie & Produits"
Tagline   : "Transmettre ce que vingt ans n'enseignent pas dans les écoles."

Modules proposés:
  - Bean-to-bar : de la fève à la tablette (2 jours)
  - Chocolat & confiserie haute gamme (1–3 jours)
  - Accompagnement personnalisé en entreprise
  - Formation équipe de production

Format : présentiel, Suisse, possibilité de déplacement
Langue : Français (anglais sur demande)
```

### 5.6 `services/audit.astro`

```
Titre     : "Audit Stratégique"
Tagline   : "Voir ce que vous ne voyez plus — parce que vous êtes dedans."

Périmètre:
  - Analyse des installations (équipements, flux, capacités)
  - Évaluation ressources humaines et compétences
  - Organisation et flux de production
  - Revue de gamme (positionnement, marges, cohérence)
  - Rapport écrit + recommandations priorisées

Livrable : Rapport structuré + session de restitution (demi-journée)
```

### 5.7 `services/evenementiel.astro`

```
Titre     : "Expériences & Évènementiel"
Tagline   : "Quand le chocolat devient l'événement lui-même."

Offres:
  - Collections privées pour maisons de luxe (horlogerie, joaillerie, mode)
  - Création chocolat × caviar (commandes exclusives)
  - Ateliers hôteliers & VIP (5 étoiles, Suisse et international)
  - Cadeaux d'entreprise haute valeur perçue
  - Dîners de prestige / expériences sensorielles

Tone: Ce n'est pas du chocolat offert. C'est une signature.
```

### 5.8 `references.astro`

```
Format: grille de 4–6 références anonymisées

Structure par référence:
  Secteur   : (e.g. "Maison horlogère, Vallée de Joux")
  Projet    : (description 1 ligne)
  Échelle   : (volume, durée, portée)
  Résultat  : (1 phrase outcome)

Disclaimer en bas de page:
  "Par respect pour la confidentialité de nos clients, les noms et certains
   détails des projets ont été modifiés. Les résultats présentés sont authentiques."
```

### 5.9 `contact.astro`

```
Layout : 2 colonnes (desktop) — infos gauche / formulaire droite

Colonne gauche:
  - Titre : "Parlons de votre projet"
  - Texte : 2–3 lignes d'introduction
  - Adresse : Aubonne, Vaud, Suisse
  - Email : info@le-chocolatier.ch
  - Instagram : @lechocolatiersuisse

Formulaire (Netlify):
  - name="contact" (Netlify Forms identifier)
  - Nom complet* (text, required)
  - Entreprise (text)
  - Email* (email, required)
  - Type de projet* (select, required):
      Consulting opérationnel
      Formation
      Audit stratégique
      Expérience & Évènementiel
      Création sur mesure / Luxe
      Autre
  - Message* (textarea, 6 rows, required)
  - Honeypot: <input name="bot-field" style="display:none" />
  - Submit: "Envoyer votre demande →"

Post-submit: redirect → /merci
```

### 5.10 `merci.astro`

```
Page simple, fond sombre:
  - Icône ✓ (SVG, couleur caramel)
  - Titre : "Votre message est bien reçu."
  - Texte : "Édouard vous répondra personnellement dans les 48 heures."
  - Lien retour → /
  
⚠️ Exclure du sitemap (dans astro.config.mjs)
```

---

## 6. Content Collections

### 6.1 Services (`src/content/services/`)

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const services = defineCollection({
  type: 'content',
  schema: z.object({
    title:       z.string(),
    tagline:     z.string(),
    description: z.string().max(160),
    icon:        z.string(),  // SVG filename in /public/icons/
    order:       z.number(),
    featured:    z.boolean().default(false),
    cta:         z.string().default('Discutons de votre projet'),
  }),
});
```

### 6.2 Case Studies (`src/content/case-studies/`)

```typescript
const caseStudies = defineCollection({
  type: 'content',
  schema: z.object({
    sector:    z.string(),
    project:   z.string(),
    scale:     z.string(),
    result:    z.string(),
    tags:      z.array(z.string()),
    year:      z.number(),
    featured:  z.boolean().default(false),
    // Never include: client name, exact location, identifiable details
  }),
});
```

---

## 7. SEO & Structured Data

### 7.1 `src/utils/seo.ts`

```typescript
interface SEOProps {
  title:        string;
  description:  string;
  canonical?:   string;
  ogImage?:     string;
  noindex?:     boolean;
}

export function buildSEO(props: SEOProps, siteUrl: string): SEOProps & { fullTitle: string } {
  return {
    ...props,
    fullTitle: `${props.title} — Édouard Morand | Le Chocolatier`,
    canonical: props.canonical ?? siteUrl,
    ogImage:   props.ogImage ?? `${siteUrl}/og-image.jpg`,
  };
}
```

### 7.2 `src/utils/schema.ts` — JSON-LD

```typescript
export const personSchema = {
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Édouard Morand",
  "jobTitle": "Maître Chocolatier & Consultant",
  "description": "Finaliste MOF 2015, DBA, consultant en chocolaterie haut de gamme.",
  "url": "https://lechocolatiersuisse.ch",
  "sameAs": ["https://www.instagram.com/lechocolatiersuisse/"],
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "Aubonne",
    "addressRegion": "Vaud",
    "addressCountry": "CH"
  }
};

export const organizationSchema = {
  "@context": "https://schema.org",
  "@type": "ProfessionalService",
  "name": "Le Chocolatier — Édouard Morand",
  "url": "https://lechocolatiersuisse.ch",
  "telephone": null,  // À renseigner si Édouard le souhaite
  "email": "info@le-chocolatier.ch",
  "areaServed": ["CH", "FR", "BE", "LU"],
  "serviceType": [
    "Consulting en chocolaterie",
    "Formation chocolatier",
    "Audit stratégique",
    "Création chocolat luxe"
  ]
};
```

### 7.3 Targets SEO — mots clés prioritaires

| Intention | Mot clé | Page cible |
|---|---|---|
| Branded | "Édouard Morand chocolatier" | /edouard |
| Service | "consultant chocolatier suisse" | /services |
| Service | "formation bean to bar suisse" | /services/formations |
| Luxe | "création chocolat luxe suisse" | /services/evenementiel |
| Local | "chocolatier consultant Vaud" | /contact |
| Anglais | "chocolate consulting Switzerland" | / (meta EN) |

---

## 8. Formulaire de contact

### Configuration Netlify Forms

```html
<!-- Dans contact.astro -->
<form
  name="contact"
  method="POST"
  data-netlify="true"
  data-netlify-honeypot="bot-field"
  action="/merci"
>
  <input type="hidden" name="form-name" value="contact" />
  <p style="display:none">
    <label>Bot field: <input name="bot-field" /></label>
  </p>
  <!-- champs ici -->
</form>
```

### Notifications Netlify

Configurer dans Netlify UI → Forms → Notifications:
- Email de notification: `info@le-chocolatier.ch`
- Sujet: `Nouvelle demande — [type de projet]`

---

## 9. Performance & Build

### Scripts pnpm

```json
{
  "scripts": {
    "dev":        "astro dev",
    "build":      "astro build",
    "preview":    "astro preview",
    "typecheck":  "astro check && tsc --noEmit",
    "lighthouse": "npx lighthouse http://localhost:4321 --output=html --view",
    "lint":       "eslint src --ext .ts,.astro"
  }
}
```

### Optimisation images — conventions

```
Nommage     : [section]-[description]-[width]w.jpg
              hero-atelier-chocolat-1920w.webp
              edouard-portrait-800w.webp
              service-consulting-600w.webp

Formats     : WebP pour toutes les images de contenu
              SVG pour icônes et décoratifs
              PNG uniquement pour logos si transparence requise

Tailles     :
  Hero      : 1920×1080 (desktop), 800×600 (mobile)
  Portrait  : 800×1000 (portrait)
  Cards     : 600×400
  OG Image  : 1200×630 (unique, public/og-image.jpg)
```

---

## 10. Internationalisation

**MVP : Français uniquement (fr-CH)**

`<html lang="fr">` sur toutes les pages.

Préparer pour l'anglais en v2:
- Stocker toutes les chaînes de texte UI dans `src/i18n/fr.ts`
- Pas de routing i18n dans le MVP — une seule langue

---

## 11. Checklist de lancement

### Contenu
- [ ] Photos professionnelles d'Édouard livrées et optimisées
- [ ] Biographie validée par Édouard (dates timeline confirmées)
- [ ] Au minimum 2 références / case studies rédigées et approuvées
- [ ] Aucun lorem ipsum restant dans le build
- [ ] Toutes les images `<!-- REPLACE -->` remplacées par vraies photos

### Technique
- [ ] `astro build` sans erreur ni warning TypeScript
- [ ] Formulaire Netlify testé en staging (soumission reçue par email)
- [ ] Page `/merci` fonctionnelle après soumission
- [ ] Sitemap.xml généré et valide
- [ ] robots.txt correct (`/merci` exclu)
- [ ] OG image vérifiée avec [opengraph.xyz](https://opengraph.xyz)

### Performance
- [ ] Lighthouse Performance ≥ 95 (mobile)
- [ ] Lighthouse Accessibility ≥ 95
- [ ] Lighthouse SEO = 100
- [ ] Lighthouse Best Practices = 100
- [ ] Aucune image > 200kb
- [ ] Core Web Vitals : LCP < 2.5s, CLS < 0.1

### SEO
- [ ] Structured data validée sur [schema.org validator](https://validator.schema.org)
- [ ] Google Search Console configuré
- [ ] Sitemap soumis à Google Search Console

### Légal
- [ ] Mentions légales (page dédiée ou footer) — obligation suisse
- [ ] Politique de confidentialité (formulaire de contact = collecte de données)
- [ ] Consentement cookies si analytics ajoutés

### DNS & Déploiement
- [ ] Domaine `lechocolatiersuisse.ch` pointé vers Netlify
- [ ] HTTPS actif (Netlify Let's Encrypt automatique)
- [ ] Redirections : `www.` → apex, `preprod.` → désactivé
- [ ] Headers de sécurité configurés dans `netlify.toml`

---

```toml
# netlify.toml
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options         = "DENY"
    X-XSS-Protection        = "1; mode=block"
    X-Content-Type-Options  = "nosniff"
    Referrer-Policy         = "strict-origin-when-cross-origin"
    Permissions-Policy      = "camera=(), microphone=(), geolocation=()"

[[redirects]]
  from   = "https://www.lechocolatiersuisse.ch/*"
  to     = "https://lechocolatiersuisse.ch/:splat"
  status = 301
  force  = true
```

---

*Specs v1.0 — Mars 2026*
*Propriété d'Édouard Morand / Le Chocolatier Suisse*
*Ne pas distribuer sans autorisation*
