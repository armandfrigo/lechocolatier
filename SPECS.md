# SPECS.md — Le Chocolatier
## lechocolatiersuisse.ch — Spécifications techniques v2.0

> **Contrat de build.** À lire intégralement avant chaque session de développement.
> Complète `.github/copilot-instructions.md` avec le détail des composants, données et décisions d'architecture.
> En cas de contradiction entre ce fichier et `copilot-instructions.md`, `copilot-instructions.md` prévaut.

---

## Table des matières

1. [Vision & positionnement](#1-vision--positionnement)
2. [Architecture technique](#2-architecture-technique)
3. [i18n — Trois langues dès le MVP](#3-i18n--trois-langues-dès-le-mvp)
4. [Design tokens](#4-design-tokens)
5. [Composants UI — détail](#5-composants-ui--détail)
6. [Pages — structure complète](#6-pages--structure-complète)
7. [Content collections](#7-content-collections)
8. [SEO & Structured Data](#8-seo--structured-data)
9. [Formulaire de contact](#9-formulaire-de-contact)
10. [Stratégie images — phase pré-shooting](#10-stratégie-images--phase-pré-shooting)
11. [CI/CD & Déploiement Vercel](#11-cicd--déploiement-vercel)
12. [Performance & build](#12-performance--build)
13. [Checklist de lancement](#13-checklist-de-lancement)

---

## 1. Vision & positionnement

### Angle éditorial fondateur

> **"L'architecte invisible du chocolat d'exception."**

La référence culturelle : Jean-Jacques Goldman. Celui qui a écrit les plus grandes chansons de Céline Dion, Johnny Hallyday, sans jamais en faire le centre de sa propre communication. Derrière les créations que vous connaissez, il y avait une seule plume. Ce site dit : cette plume existe dans le chocolat, et elle est disponible pour votre projet.

### Ce que le site doit accomplir

**Objectif principal** : convertir des visiteurs qualifiés en prospects qui prennent contact.

**Cibles** (par ordre de priorité) :
1. Responsables achats et directeurs de maisons de luxe (horlogerie, joaillerie, hôtellerie 5★)
2. Chocolatiers artisanaux suisses cherchant à externaliser R&D ou mise en production
3. Acheteurs grande distribution cherchant un sourcing direct et une capacité de production à l'échelle
4. Directeurs de communication luxe cherchant des expériences sensorielles originales

**Positionnement discret** : Édouard ne veut pas annoncer ses ambitions à ses concurrents et clients actuels. Le site doit attirer par l'autorité et la qualité, pas par le bruit. Aucune énergie "lancement startup". Aucun nom de client sans autorisation écrite.

### KPIs (à 6 mois)

| Métrique | Cible |
|---|---|
| Taux de conversion visiteur → formulaire | ≥ 2% |
| Durée moyenne de session | ≥ 2:30 min |
| Position SEO "consultant chocolat suisse" | Top 3 |
| Position SEO "chocolate consulting Switzerland" | Top 5 |
| Lighthouse Performance mobile | ≥ 95 |

---

## 2. Architecture technique

### Stack & justifications

```
Astro 4.x       Static-first, zéro JS par défaut, i18n natif, intégration Tailwind native.
                Idéal pour un site vitrine où la performance est non-négociable.

Tailwind 3.x    Utility-first. Design system via config. Zéro CSS mort au build.
                Pas de composants pré-stylisés (pas de shadcn, pas de daisyUI).

TypeScript      Strict mode. Props typées partout. Aucun `any`.

Vercel          CDN mondial. Previews automatiques sur PR. Domaine custom gratuit.
                Formulaires gérés via Netlify Forms (POST action) — Vercel n'a pas
                de gestion native des formulaires statiques.

GitHub Actions  CI/CD : quality check → build → deploy. Voir deploy.yml.

pnpm            Gestionnaire de paquets. Lockfile committé.
```

### Configuration Astro (`astro.config.mjs`)

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
        locales: { fr: 'fr-CH', en: 'en-US', de: 'de-CH' },
      },
    }),
  ],
  i18n: {
    defaultLocale: 'fr',
    locales: ['fr', 'en', 'de'],
    routing: { prefixDefaultLocale: false },
  },
  image: {
    service: { entrypoint: 'astro/assets/services/sharp' },
  },
  compressHTML: true,
  build: { inlineStylesheets: 'auto' },
});
```

**Routing i18n résultant :**
```
/             → Français (défaut)
/en/          → English
/de/          → Deutsch
/services/    → FR
/en/services/ → EN
/de/services/ → DE
```

### Configuration Tailwind (`tailwind.config.mjs`)

```javascript
import { fontFamily } from 'tailwindcss/defaultTheme';

export default {
  content: ['./src/**/*.{astro,html,js,ts,mdx}'],
  theme: {
    extend: {
      colors: {
        noir:    '#0D0805',
        brun:    '#1E1208',
        marron:  '#4A2E20',
        caramel: '#C8915A',
        creme:   '#F8F3EB',
        or:      '#A87832',
        vert:    '#2D5A1B',
        'vert-lc': '#4A8A2E',
      },
      fontFamily: {
        display: ['"Cormorant Garamond"', ...fontFamily.serif],
        body:    ['"DM Sans"', ...fontFamily.sans],
      },
      fontSize: {
        'display-xl': ['clamp(3.5rem, 8vw, 7.5rem)', { lineHeight: '1.0', letterSpacing: '-0.02em' }],
        'display-lg': ['clamp(2.5rem, 5vw, 4.5rem)', { lineHeight: '1.1', letterSpacing: '-0.01em' }],
        'display-md': ['clamp(1.5rem, 3vw, 2.5rem)', { lineHeight: '1.2' }],
      },
      spacing: {
        'section-sm': '4rem',
        'section-md': '6rem',
        'section-lg': '10rem',
      },
      maxWidth: {
        prose:   '65ch',
        content: '72rem',
      },
    },
  },
};
```

---

## 3. i18n — Trois langues dès le MVP

### Principe

Toutes les chaînes de texte UI sont dans `src/i18n/{lang}.ts`. Aucun texte hardcodé dans les composants `.astro`.

### Structure des fichiers de traduction

```typescript
// src/i18n/fr.ts — MASTER COPY — toutes les clés définies ici d'abord
export const fr = {
  nav: {
    expertise:   "L'expertise",
    services:    "Services",
    references:  "Références",
    contact:     "Contact",
    cta:         "Votre projet",
  },
  hero: {
    eyebrow:     "Aubonne, Suisse · Consulting en chocolaterie haut de gamme",
    headline:    "L'architecte invisible du chocolat.",
    // "invisible" → italic caramel in rendering
    sub:         "Derrière certaines des créations les plus remarquées du chocolat suisse, il y avait une seule expertise. Elle est désormais disponible pour votre projet.",
    ctaPrimary:  "Parlons de votre projet",
    ctaGhost:    "Découvrir l'expertise",
    scroll:      "Défiler",
  },
  stats: {
    exp:     { value: "20+",      label: "Années",   sub: "de chocolaterie haut de gamme" },
    mof:     { value: "MOF",      label: "Finaliste 2015", sub: "Meilleur Ouvrier de France" },
    prod:    { value: "700t",     label: "Production", sub: "Produit fini / an · sourcing direct" },
    btb:     { value: "B→B",      label: "Bean-to-Bar", sub: "De la fève à la tablette" },
    dba:     { value: "DBA",      label: "Doctorat", sub: "Business Administration" },
  },
  // ... (toutes sections)
  contact: {
    eyebrow:     "Votre projet",
    headline:    "Parlons de ce que vous voulez bâtir.",
    subtext:     "Chaque projet est unique. Prenez contact pour un premier échange confidentiel.",
    labelName:   "Nom complet",
    labelCo:     "Entreprise",
    labelEmail:  "Email",
    labelType:   "Type de projet",
    labelMsg:    "Message",
    submit:      "Envoyer",
    address:     "Aubonne, Vaud — Suisse",
    types: [
      { value: "consulting",  label: "Consulting opérationnel" },
      { value: "luxury",      label: "Création luxe & sur mesure" },
      { value: "audit",       label: "Audit stratégique" },
      { value: "formation",   label: "Formation" },
      { value: "other",       label: "Autre" },
    ],
  },
  footer: {
    copy: "© 2026 Le Chocolatier · Aubonne, Suisse",
    instagram: "Instagram @lechocolatiersuisse",
  },
};
```

### Helper i18n (`src/utils/i18n.ts`)

```typescript
import { fr } from '../i18n/fr';
import { en } from '../i18n/en';
import { de } from '../i18n/de';

const translations = { fr, en, de };

export type Locale = 'fr' | 'en' | 'de';

export function getTranslations(locale: Locale) {
  return translations[locale] ?? translations.fr;
}
```

### Règle German placeholder

Tant que la traduction professionnelle allemande n'est pas disponible :
```typescript
// src/i18n/de.ts
// [DE: traduction professionnelle requise — NE PAS utiliser auto-translate IA]
export const de = {
  nav: {
    expertise:  "Expertise",           // safe — same word
    services:   "Leistungen",          // safe — standard
    references: "Referenzen",          // safe — standard
    contact:    "Kontakt",             // safe — standard
    cta:        "[DE: traduction requise]",
  },
  // ...
};
```

---

## 4. Design tokens

### Typographie — règles d'application

| Usage | Font | Weight | Size | Transform |
|---|---|---|---|---|
| Hero H1 | Cormorant Garamond | 300 | display-xl | none |
| Section H2 | Cormorant Garamond | 400 | display-lg | none |
| Card H3 | Cormorant Garamond | 600 | display-md | none |
| Italic accent (headlines) | Cormorant Garamond | 300 italic | inherited | none |
| Body prose | DM Sans | 300 | 1.125rem | none |
| Navigation | DM Sans | 400 | 0.875rem | UPPERCASE |
| Caption / meta / eyebrow | DM Sans | 300 | 0.65–0.75rem | UPPERCASE, tracking: 0.2–0.35em |
| CTA button | DM Sans | 500 | 0.7rem | UPPERCASE, tracking: 0.15em |

### CSS custom properties (global.css)

```css
:root {
  --color-noir    : #0D0805;
  --color-brun    : #1E1208;
  --color-marron  : #4A2E20;
  --color-caramel : #C8915A;
  --color-creme   : #F8F3EB;
  --color-or      : #A87832;
  --color-vert    : #2D5A1B;
  --color-vert-lc : #4A8A2E;

  --ease-luxury : cubic-bezier(0.25, 0.1, 0.25, 1);
  --ease-reveal : cubic-bezier(0.16, 1, 0.3, 1);

  --font-display : 'Cormorant Garamond', Georgia, serif;
  --font-body    : 'DM Sans', system-ui, sans-serif;
}
```

### Hiérarchie des couleurs

```
Background page          → --color-creme   (#F8F3EB)
Background hero          → --color-brun    (#1E1208)
Background dark sections → --color-noir    (#0D0805)
Background footer        → --color-noir    (#0D0805)
Text principal           → --color-noir    (#0D0805)
Text sur fond sombre     → rgba(#F8F3EB, 1) → rgba(white, 0.55) pour secondaire
Accent / CTA             → --color-caramel (#C8915A)
Hover CTA                → --color-or      (#A87832)
Italic headline accent   → --color-caramel (#C8915A)
Border / séparateur      → --color-marron  (#4A2E20) à 15–30% opacity
Eyebrow dark sections    → --color-vert-lc (#4A8A2E)
```

---

## 5. Composants UI — détail

### 5.1 Button.astro

```
Variants  : primary | secondary | ghost
Sizes     : sm | md | lg
States    : default | hover | focus-visible | disabled

primary   → bg-caramel text-noir      | hover: bg-or
secondary → border border-caramel text-caramel | hover: bg-caramel/10
ghost     → text-caramel              | hover: underline + arrow translateX

Props:
  href?     : string   — renders <a>
  type?     : 'button' | 'submit' | 'reset'
  variant   : 'primary' | 'secondary' | 'ghost'   (default: 'primary')
  size      : 'sm' | 'md' | 'lg'                  (default: 'md')
  class?    : string   — Tailwind passthrough
  arrow?    : boolean  — appends → with hover translate animation
```

### 5.2 ExplodedView.astro — Composant signature

Le visuel le plus distinctif du site. SVG pur + animations CSS. Zéro JavaScript.

```
Dimensions SVG : viewBox="0 0 420 480"
Wrapper        : position relative, 420×480px desktop, 320×380px mobile

Couches (de haut en bas, chacune avec animation CSS indépendante) :

1. Coque verte supérieure (hémisphère)
   - Path en demi-ellipse : M 110 120 Q 110 20 210 20 Q 310 20 310 120 Z
   - Fill : radialGradient #7DB85A → #4A8A2E → #2D5A1B → #1A3A0D (cx:35%, cy:30%)
   - Overlay marbre : 2 path SVG stroke, rgba(vert clair, 0.2–0.3), stroke-width 8–12
   - Highlight spéculaire : ellipse radialGradient blanc → transparent (top-left)
   - Rim : ellipse stroke rgba(vert, 0.4), stroke-width 1.5
   - Base plate : ellipse fill #1d4010
   - Animation : floatShell — translateY 0 → -18px, 4s ease-in-out infinite
   - Filter : feDropShadow rgba(vert, 0.4)

2. Disque Ganache
   - Ellipse cx:210 cy:220 rx:90 ry:13
   - Fill : radialGradient brun foncé (#5C3020 → #2A1008)
   - Animation : floatGanache — translateY 0 → -10px, 4s ease-in-out 0.3s infinite

3. Disque Praliné / intérieur
   - Ellipse cx:210 cy:255 rx:82 ry:12
   - Fill : radialGradient crème/caramel (#F0DFC0 → #D4A96A → #8B5E28)
   - Lignes de texture : 1 path arc stroke rgba(caramel, 0.4)
   - Animation : floatInterior — translateY 0 → -14px, 4s ease-in-out 0.15s infinite

4. Coque chocolat noir inférieure (bol)
   - Path : M 110 320 Q 110 420 210 420 Q 310 420 310 320 Z
   - Fill : radialGradient (#6B3D22 → #3D1E0A → #1E0D05)
   - Rim : ellipse stroke rgba(brun, 0.5)
   - Highlight intérieur : path arc stroke rgba(brun clair, 0.3)
   - Statique — élément d'ancrage visuel

Éléments décoratifs :
   - 5–6 particules circulaires (r:1–2), caramel et vert, opacity 0.2–0.4
   - Ligne de dimension gauche : stroke-dasharray 4,4, caramel 15% opacity
   - Lignes de connexion labels : stroke caramel 0.6px, opacity 0.4–0.6

Labels (absolus, desktop uniquement — hidden < 768px) :
   - Position : right side, connected by 1px caramel line
   - Font : DM Sans 0.6rem, uppercase, letter-spacing 0.2em, color caramel
   - "Coque peinte · Beurre de cacao"  → top ~55px
   - "Ganache grand cru"               → top ~175px
   - "Praliné maison"                  → top ~265px
   - "Coque chocolat noir"             → top ~360px (left side)
   - Trilingues via [data-lang] switching

Section wrapper :
   - Background : --color-noir (#0D0805)
   - Radial glow : rgba(vert, 0.06), ellipse 60% 50%
   - Grain texture overlay
   - Layout : 2 colonnes (texte gauche / SVG droite) → 1 colonne sur mobile
```

### 5.3 Hero.astro

```
Background  : --color-brun (#1E1208) + grain texture (CSS ::before)
Radial glow : rgba(caramel, 0.08), top-right, 70vw × 70vw
Layout      : flex column, justify-center, padding-top 8rem, min-h-screen

Éléments (top → bottom) avec animation CSS staggerée :
  1. Pre-headline  — DM Sans 0.65rem uppercase tracking-[0.35em] text-caramel
                     opacity-0 → 1, translateY 20px → 0, delay 200ms
  2. H1            — Cormorant 300, display-xl, text-white, max-w-[900px]
                     "invisible" / "architect" en italic caramel
                     opacity-0 → 1, translateY 30px → 0, delay 400ms
  3. Sub-headline  — DM Sans 300 1.05rem, rgba(white, 0.55), max-w-[540px]
                     opacity-0 → 1, translateY 20px → 0, delay 600ms
  4. CTA pair      — opacity-0 → 1, translateY 20px → 0, delay 800ms
                     [btn-primary: "Parlons de votre projet"] [btn-ghost: "Découvrir l'expertise →"]
  5. Scroll cue    — opacity-0 → 1, delay 1200ms
                     "Défiler" label + 1px vertical line, 60px, gradient caramel → transparent
                     animation scrollPulse 2s ease-in-out infinite

Règles absolues :
  - Aucune image (ni photo, ni vidéo, ni IA)
  - Aucun élément lourd
  - LCP < 2.5s : seul le texte charge dans le hero
```

### 5.4 Stats.astro

```
Container   : background creme, border-top + border-bottom caramel/12
Layout      : flex row, space-between, max-w-content, gap-8
              overflow-x auto sur mobile (scroll horizontal)

5 items :
  { value: "20+",   label: "Années",        sub: "de chocolaterie haut de gamme" }
  { value: "MOF",   label: "Finaliste 2015", sub: "Meilleur Ouvrier de France" }
  { value: "700t",  label: "Production",    sub: "Produit fini / an · sourcing direct" }
  { value: "B→B",   label: "Bean-to-Bar",   sub: "De la fève à la tablette" }
  { value: "DBA",   label: "Doctorat",      sub: "Business Administration" }

Typographie :
  value  → Cormorant Garamond, 2.5rem, text-caramel
  label  → DM Sans 400, 0.65rem, uppercase, tracking-[0.15em]
  sub    → DM Sans 300, 0.75rem, text-noir/50

Séparateurs : 1px vertical, marron 15% opacity, hidden sur mobile
Scroll reveal avec data-reveal + delay 0–4
```

### 5.5 Header.astro

```
Position : fixed, top 0, full-width, z-99
Initial  : transparent (sur hero sombre)
Scrolled (> 80px) :
  - background rgba(13,8,5, 0.95)
  - backdrop-filter blur(20px)
  - border-bottom 1px rgba(caramel, 0.15)
  - transition all 500ms var(--ease-luxury)

Contenu :
  Gauche : logo "Le Chocolatier" (Cormorant, 1.1rem, uppercase, tracking-[0.25em])
           "Le" blanc, "Chocolatier" caramel
  Centre : nav links (DM Sans 0.65rem, uppercase, tracking-[0.2em])
           couleur rgba(white, 0.6) → caramel au hover
  Droite : [LanguageSwitcher] + [CTA button ghost → primary au scroll]

Mobile (< 768px) :
  - Logo + hamburger uniquement
  - Tap hamburger → overlay full-screen background noir
  - Links en vertical, large, Cormorant display
  - Close via ✕ ou tap extérieur
```

### 5.6 ServiceCard.astro

```
Props:
  number   : string  ("01" | "02" | "03" | "04")
  title    : string
  tagline  : string  (italic, caramel)
  text     : string  (2–3 sentences)
  points   : string[]  (3 items max)
  href?    : string

Layout : 2×2 grid desktop, 1 col mobile
Style  : background creme, border caramel/12
         Numéro giant en arrière-plan : Cormorant 5rem, marron 8% opacity

Interactions :
  ::before pseudo : 3px left border, height 0 → 100% au hover (transition 400ms)
  hover background : rgba(caramel, 0.04)
```

### 5.7 CaseStudyCard.astro (Références anonymisées)

```
Props:
  sector   : string  ("Maison horlogère suisse — Vallée de Joux")
  project  : string  (description 1 ligne)
  scale    : string  (volume / durée / portée)
  result   : string  (outcome 1 phrase)
  tags     : string[]
  year     : number

Règle absolue : JAMAIS de nom de client réel
Disclaimer de page :
  "Par respect pour la confidentialité de nos partenaires, les noms et certains
   détails des projets sont volontairement omis. Les résultats sont authentiques."
```

### 5.8 LanguageSwitcher.astro

```
Style   : 3 boutons texte inline (FR · EN · DE)
          DM Sans 0.65rem, uppercase, tracking-[0.2em]
          Inactif : rgba(white, 0.5) sur fond sombre / rgba(noir, 0.4) sur fond clair
          Actif + hover : text-caramel
          Séparateur · entre les boutons

Behavior :
  - onClick → setLang(code) → localStorage.setItem('lc-lang', code)
  - Persiste entre sessions
  - Met à jour html[lang], tous les [data-lang] éléments
  - URL ne change pas (switching client-side) — évite la complexité de routing
    Note : pour le SEO multilingue, les pages /en/ et /de/ existent en dur côté serveur
```

---

## 6. Pages — structure complète

### 6.1 `index.astro` — Accueil

**Sections dans l'ordre :**
```
1. Hero             (plein écran sombre — "L'architecte invisible")
2. Stats            (5 chiffres, fond creme)
3. ExplodedView     (plein écran sombre — SVG demi-sphère verte animée)
4. Services         (4 cartes 2×2, fond creme)
5. Quote            (plein écran sombre — citation philosophique)
6. Expertise        (2 colonnes : texte gauche / liste droite)
7. Contact CTA      (plein écran sombre — bouton)
```

**Meta FR :**
```
title       : "Le Chocolatier — Consulting en chocolaterie haut de gamme, Suisse"
description : "Expertise en création chocolatière, consulting industriel et créations de luxe. Derrière les plus grandes maisons suisses. Aubonne, Vaud."
```

**Alternance sombre/clair des sections :**
```
Hero          → sombre (#1E1208)
Stats         → clair  (#F8F3EB)
ExplodedView  → sombre (#0D0805)
Services      → clair  (#F8F3EB)
Quote         → sombre (#1E1208)
Expertise     → clair  (#F8F3EB)
Contact CTA   → sombre (#0D0805)
```

### 6.2 `en/index.astro` et `de/index.astro`

Même structure que `index.astro`, avec `lang` passé aux composants pour sélection i18n.

### 6.3 `services/index.astro`

```
Vue d'ensemble des 4 services.
Header section + grille 2×2 + CTA global.
Lien vers sous-pages détaillées.
```

### 6.4 Services — sous-pages (consulting / formations / audit / evenementiel)

Chaque sous-page suit ce template :
```
1. Page hero (titre + tagline + fond sombre)
2. Description (ce que c'est — 2–3 paragraphes)
3. Pour qui (profil client cible)
4. Livrables (ce que vous recevez — liste)
5. Cas type anonymisé
6. CTA : "Discutons de votre projet"
```

**Données par service :**

```
consulting.astro
  Titre    : "Consulting Opérationnel"
  Tagline  : "Du concept à la mise en production — sans lacune."
  Cible    : chocolatiers artisanaux, marques qui veulent externaliser R&D+production
  Capacité : sourcing direct Cameroun, volumes 600–700t/an grande distribution
  Bean-to-bar : maîtrise complète du process

formations.astro
  Titre    : "Formations Technologie & Produits"
  Tagline  : "Transmettre ce que vingt ans n'enseignent pas dans les écoles."
  Modules  : Bean-to-bar (2j), Chocolat & confiserie haut de gamme (1–3j), entreprise sur mesure
  Langues  : Français, English on request

audit.astro
  Titre    : "Audit Stratégique"
  Tagline  : "Voir ce que vous ne voyez plus — parce que vous êtes dedans."
  Périmètre: installations + RH + flux production + gamme
  Livrable : Rapport écrit + session restitution demi-journée

evenementiel.astro
  Titre    : "Expériences & Créations sur Mesure"
  Tagline  : "Quand le chocolat devient l'événement lui-même."
  Offres   :
    - Collections privées pour maisons de luxe (horlogerie, joaillerie)
    - Créations chocolat × caviar (commandes exclusives haute valeur)
    - Ateliers hôteliers & VIP (5★, Suisse et international)
    - Cadeaux d'entreprise haute valeur perçue
    - Dîners de prestige / expériences sensorielles
```

### 6.5 `references.astro`

```
Layout : grille de 4–6 cartes CaseStudyCard
Header : titre + disclaimer confidentialité

Références à créer (à valider avec Édouard) :
  1. Secteur : Maison horlogère, Vallée de Joux
     Projet  : Collection chocolat pour lancement de montre
     Échelle : Édition limitée, ~500 pièces
     Tags    : Luxe, Événementiel

  2. Secteur : Hôtellerie 5★, Suisse romande
     Projet  : Programme mensuel de création chocolatière VIP
     Échelle : Collections saisonnières récurrentes
     Tags    : Hôtellerie, Création, Récurrent

  3. Secteur : Grande distribution, France + Suisse
     Projet  : Développement gamme bean-to-bar avec sourcing direct
     Échelle : 400–600t produit fini / an
     Tags    : Industriel, Bean-to-bar, Sourcing

  4. Secteur : Maison de confiserie suisse (non nommée)
     Projet  : R&D et mise en production d'une gamme signature
     Tags    : Consulting, R&D, Production

⚠️ Toutes ces références sont à faire valider par Édouard AVANT mise en ligne
```

### 6.6 `contact.astro`

```
Layout : 2 colonnes desktop (infos gauche / formulaire droite) → 1 col mobile

Colonne gauche :
  - Eyebrow : "Votre projet"
  - H2 : "Parlons de ce que vous voulez bâtir."
  - Subtext : 2 lignes
  - Adresse : Aubonne, Vaud, Suisse
  - Email : info@le-chocolatier.ch
  - Instagram : @lechocolatiersuisse

Formulaire (voir section 9)
```

### 6.7 `merci.astro`

```
Background sombre (#0D0805)
Contenu centré :
  - Icône ✓ SVG, stroke caramel
  - H1 (Cormorant) : "Votre message est bien reçu."
  - Texte : "Vous recevrez une réponse personnelle dans les 48 heures."
  - Lien → /  (btn-ghost)

⚠️ EXCLURE du sitemap dans astro.config.mjs
⚠️ Ajouter <meta name="robots" content="noindex"> dans le head
```

---

## 7. Content Collections

### `src/content/config.ts`

```typescript
import { defineCollection, z } from 'astro:content';

const services = defineCollection({
  type: 'content',
  schema: z.object({
    title:        z.string(),
    titleEn:      z.string(),
    titleDe:      z.string(),
    tagline:      z.string(),
    taglineEn:    z.string(),
    taglineDe:    z.string(),
    description:  z.string().max(160),
    icon:         z.string(),
    order:        z.number(),
    featured:     z.boolean().default(false),
  }),
});

const caseStudies = defineCollection({
  type: 'content',
  schema: z.object({
    sector:    z.string(),
    project:   z.string(),
    scale:     z.string(),
    result:    z.string(),
    tags:      z.array(z.string()),
    year:      z.number().optional(),
    featured:  z.boolean().default(false),
    validated: z.boolean().default(false), // ⚠️ must be true before publishing
    // NEVER add: clientName, exactLocation, identifiableDetails
  }),
});

export const collections = { services, caseStudies };
```

---

## 8. SEO & Structured Data

### 8.1 SEO par page — template

```typescript
// src/utils/seo.ts
export interface SEOProps {
  title:       string;
  description: string;
  canonical?:  string;
  ogImage?:    string;
  noindex?:    boolean;
  locale?:     'fr-CH' | 'en-US' | 'de-CH';
  alternates?: { lang: string; url: string }[];
}

export function buildSEO(props: SEOProps, siteUrl: string) {
  return {
    ...props,
    fullTitle: `${props.title} — Le Chocolatier`,
    canonical:  props.canonical ?? siteUrl,
    ogImage:    props.ogImage   ?? `${siteUrl}/og-image.jpg`,
    locale:     props.locale    ?? 'fr-CH',
  };
}
```

### 8.2 JSON-LD (src/utils/schema.ts)

```typescript
export const organizationSchema = {
  "@context": "https://schema.org",
  "@type": "ProfessionalService",
  "name": "Le Chocolatier",
  "url": "https://lechocolatiersuisse.ch",
  "email": "info@le-chocolatier.ch",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "Aubonne",
    "addressRegion": "Vaud",
    "addressCountry": "CH"
  },
  "areaServed": ["CH", "FR", "BE", "LU", "DE", "AT"],
  "serviceType": [
    "Consulting en chocolaterie",
    "Formation chocolatier bean-to-bar",
    "Audit stratégique chocolaterie",
    "Création chocolat luxe",
    "Sourcing cacao direct"
  ],
  "sameAs": ["https://www.instagram.com/lechocolatiersuisse/"]
};

// Note : le schema Person avec le nom d'Édouard est volontairement omis du MVP
// par cohérence avec la stratégie de discrétion. À ajouter si/quand Édouard
// décide de rendre son nom public sur le site.
```

### 8.3 hreflang (BaseLayout.astro)

```astro
<link rel="alternate" hreflang="fr-CH" href="https://lechocolatiersuisse.ch{path}" />
<link rel="alternate" hreflang="en-US" href="https://lechocolatiersuisse.ch/en{path}" />
<link rel="alternate" hreflang="de-CH" href="https://lechocolatiersuisse.ch/de{path}" />
<link rel="alternate" hreflang="x-default" href="https://lechocolatiersuisse.ch{path}" />
```

### 8.4 Mots clés SEO prioritaires

| Intention | Mot clé | Page cible | Priorité |
|---|---|---|---|
| Service FR | "consultant chocolatier suisse" | / et /services | ★★★ |
| Service FR | "consulting chocolaterie suisse" | /services/consulting | ★★★ |
| Service FR | "formation bean to bar suisse" | /services/formations | ★★ |
| Luxe FR | "création chocolat luxe suisse" | /services/evenementiel | ★★ |
| Service EN | "chocolate consulting Switzerland" | /en/ | ★★★ |
| Service EN | "bean to bar consultant Switzerland" | /en/services/ | ★★ |
| Service DE | "Schokoladen Consultant Schweiz" | /de/ | ★★ |
| Local | "chocolatier consultant Vaud" | /contact | ★ |

---

## 9. Formulaire de contact

### Configuration (Netlify Forms via Vercel)

Vercel ne gère pas les formulaires statiques nativement. Le formulaire est traité par Netlify Forms via un POST classique. Astro génère du HTML statique ; Netlify intercepte le POST.

**Alternative si Netlify Forms non disponible** : utiliser [Formspree](https://formspree.io) (gratuit jusqu'à 50 soumissions/mois) avec la même approche POST sans JS.

```html
<!-- contact.astro -->
<form
  name="contact-lc"
  method="POST"
  data-netlify="true"
  data-netlify-honeypot="bot-field"
  action="/merci"
>
  <input type="hidden" name="form-name" value="contact-lc" />
  <input type="hidden" name="lang" value={currentLocale} />
  <p style="display:none"><input name="bot-field" /></p>

  <!-- Nom -->
  <label for="name">{t.contact.labelName} *</label>
  <input id="name" name="name" type="text" required autocomplete="name" />

  <!-- Entreprise -->
  <label for="company">{t.contact.labelCo}</label>
  <input id="company" name="company" type="text" autocomplete="organization" />

  <!-- Email -->
  <label for="email">Email *</label>
  <input id="email" name="email" type="email" required autocomplete="email" />

  <!-- Type de projet -->
  <label for="type">{t.contact.labelType} *</label>
  <select id="type" name="type" required>
    <option value="" disabled selected>—</option>
    {t.contact.types.map(opt => (
      <option value={opt.value}>{opt.label}</option>
    ))}
  </select>

  <!-- Message -->
  <label for="message">{t.contact.labelMsg} *</label>
  <textarea id="message" name="message" rows="5" required></textarea>

  <button type="submit">{t.contact.submit} →</button>
</form>
```

### Notification email

Configurer dans Netlify → Forms → contact-lc → Notifications :
- To: `info@le-chocolatier.ch`
- Subject: `[Le Chocolatier] Nouvelle demande — {{type}}`

---

## 10. Stratégie images — phase pré-shooting

### Contexte

Le shooting photographique avec Édouard est prévu **fin mars 2026**. Jusqu'à cette date, aucune image réelle n'est disponible.

### Ce qui est utilisé en attendant

| Élément | Approche |
|---|---|
| Hero | Aucune image — fond sombre + grain texture CSS |
| Section sombres | Gradients radials + grain CSS |
| ExplodedView | SVG généré en code — aucun fichier image |
| Cards services | Fond creme plein — numéro géant en arrière-plan |
| About / expertise | Placeholder block coloré avec commentaire `<!-- REPLACE -->` |

### Nomenclature placeholders

```html
<!-- REPLACE: photo professionnelle main d'Édouard travaillant le chocolat -->
<div class="aspect-[4/3] bg-brun/40 flex items-center justify-center">
  <span class="text-caramel/30 text-xs uppercase tracking-widest">
    Photo · Mars 2026
  </span>
</div>
```

### Livrables attendus du photographe

```
Priorité 1 (hero + about) :
  - Mains au travail : beurre de cacao, pinceau sur moule, travail tablette
  - Formats : 1920w + 800w + 400w, WebP

Priorité 2 (sections services) :
  - Ambiance atelier : lumière naturelle, outils professionnels
  - Détails produits : bonbons peints en cours, coque au démoulage
  - Formats : 1200w + 600w, WebP

Priorité 3 (cards + OG) :
  - Beauty shots produits finis (comme les demi-sphères vertes de la référence)
  - OG image 1200×630 : composition logo + produit
```

### Conversion après shooting

```bash
# Conversion WebP après réception des photos
for f in public/images/raw/*.jpg; do
  cwebp -q 82 "$f" -o "${f%.jpg}.webp"
done
```

---

## 11. CI/CD & Déploiement Vercel

### Workflow GitHub Actions (`.github/workflows/deploy.yml`)

```yaml
name: CI/CD → Vercel

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
        with: { version: 8 }
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm run typecheck
      - run: pnpm run build

  deploy-preview:
    needs: quality
    if: github.event_name == 'pull_request' || github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
        with: { version: 8 }
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}

  deploy-production:
    needs: quality
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://lechocolatiersuisse.ch
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
        with: { version: 8 }
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### DNS (chez le registraire de lechocolatiersuisse.ch)

```
Type A     : @     → 76.76.21.21
Type CNAME : www   → cname.vercel-dns.com
Type CNAME : preprod → [désactiver ou rediriger vers apex]
```

### Scripts pnpm

```json
{
  "scripts": {
    "dev":       "astro dev",
    "build":     "astro build",
    "preview":   "astro preview",
    "typecheck": "astro check && tsc --noEmit",
    "lint":      "eslint src --ext .ts,.astro",
    "lighthouse": "npx lighthouse http://localhost:4321 --output=html --view"
  }
}
```

---

## 12. Performance & build

### Targets Lighthouse (mobile)

| Métrique | Cible |
|---|---|
| Performance | ≥ 95 |
| Accessibility | ≥ 95 |
| SEO | 100 |
| Best Practices | 100 |
| LCP | < 2.5s |
| CLS | < 0.1 |
| INP | < 100ms |

### Images — conventions

```
Format     : WebP exclusivement pour les images de contenu
             SVG pour icônes et éléments décoratifs
             PNG uniquement si transparence requise et SVG impossible

Nommage    : {section}-{description}-{width}w.webp
             hero-mains-chocolat-1920w.webp
             service-consulting-600w.webp

Tailles    :
  Hero     : 1920w + 800w (lazy: non, fetchpriority: high)
  Sections : 1200w + 600w (lazy: oui)
  Cards    : 600w + 300w  (lazy: oui)
  OG       : 1200×630 (unique, public/og-image.jpg)

Budget     : Aucune image > 200kb après compression
```

---

## 13. Checklist de lancement

### Contenu & branding
- [ ] Toutes les traductions DE validées par traducteur professionnel
- [ ] Références validées par Édouard (validated: true dans frontmatter)
- [ ] Placeholder `<!-- REPLACE -->` comptabilisés et documentés
- [ ] Aucun lorem ipsum dans le build
- [ ] Angle éditorial "architecte invisible" cohérent sur toutes les pages
- [ ] Aucun nom de client non autorisé

### Technique
- [ ] `pnpm run build` sans erreur TypeScript
- [ ] `pnpm run typecheck` propre
- [ ] Formulaire testé (soumission → email reçu par info@le-chocolatier.ch)
- [ ] Page `/merci` fonctionnelle et exclue du sitemap
- [ ] hreflang correct sur toutes les pages
- [ ] Sitemap.xml généré, validé, soumis à Search Console
- [ ] robots.txt correct (`/merci` exclu)

### Performance
- [ ] Lighthouse Performance ≥ 95 mobile
- [ ] Lighthouse Accessibility ≥ 95
- [ ] Lighthouse SEO = 100
- [ ] Core Web Vitals : LCP < 2.5s, CLS < 0.1
- [ ] Aucune image > 200kb

### SEO
- [ ] JSON-LD validé sur schema.org/validator
- [ ] OG image vérifiée (opengraph.xyz)
- [ ] Google Search Console configuré

### Légal (obligations suisses)
- [ ] Mentions légales (page dédiée ou footer étendu)
- [ ] Politique de confidentialité (collecte email via formulaire)
- [ ] Pas d'analytics au lancement si pas de bannière cookie

### DNS & infra
- [ ] lechocolatiersuisse.ch → Vercel (DNS propagé)
- [ ] HTTPS actif (Let's Encrypt via Vercel)
- [ ] `www.` → apex : 301 configuré dans vercel.json
- [ ] `preprod.` → désactivé ou redirigé
- [ ] GitHub secrets VERCEL_TOKEN / ORG_ID / PROJECT_ID en place
- [ ] Premier deploy production via GitHub Actions confirmé

### Post-shooting (mars 2026)
- [ ] Remplacer tous les placeholders par vraies photos
- [ ] Optimiser en WebP, vérifier budget < 200kb par image
- [ ] Lighthouse re-passé après intégration photos
- [ ] OG image mise à jour avec vraie photo produit

---

*SPECS v2.0 — Mars 2026*
*Intègre toutes les décisions prises en session : positionnement Goldman, discrétion stratégique, i18n FR/EN/DE dès le MVP, déploiement Vercel + GitHub Actions, ExplodedView SVG, stratégie images pré-shooting, suppression de la page /edouard en façade.*
*Propriété de Le Chocolatier — Ne pas distribuer sans autorisation.*