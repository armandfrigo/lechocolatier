# GitHub Copilot — Instructions for Le Chocolatier

> This file lives at `.github/copilot-instructions.md` and is loaded automatically
> by GitHub Copilot in every conversation in this repository.
> **Read it entirely before writing any code, generating any file, or making any suggestion.**

---

## 1. Project Identity

### Who
A master chocolatier operating under the brand name **"Le Chocolatier"** — deliberately anonymous.
The person behind the brand is Édouard Morand: MOF 2015 finalist, DBA, co-founder of Fondation Haute Chocolaterie, 20+ years creating for the most demanding Swiss and international houses. His name is known to peers. On this site, it is never the headline.

### What
`lechocolatiersuisse.ch` — a B2B consulting and creation website. Not a shop. Not a portfolio. A business development tool that generates qualified leads through authority and restraint.

### The strategic positioning
**"L'architecte invisible du chocolat d'exception."**

The editorial angle is the Goldman paradox: the best songs you loved were written by someone whose face you never saw. The best Swiss chocolates you ever tasted may have been conceived, formulated, and put into production by one person — who never appeared on the label. That person now offers their expertise directly.

This positioning must permeate every line of copy, every design decision, every interaction. Understated. Precise. Confident without announcing itself.

### Why discretion is strategic
Édouard operates in a confidential professional ecosystem. His clients — luxury houses, major Swiss chocolatiers, mass retail buyers — are often competitors of each other. Shouting his ambitions publicly would damage existing relationships. The site must attract new clients organically (SEO, word of mouth, referrals) without alerting the existing professional network. **No press release energy. No "I'm launching" energy.**

### What this site is NOT
- Not a personal showcase for Édouard Morand the person
- Not an e-commerce site
- Not a portfolio of named clients
- Not a blog
- Not a media presence

---

## 2. Tech Stack — Non-Negotiable

```
Framework     : Astro 4.x (static-first, zero JS by default)
Styling       : Tailwind CSS 3.x (utility-only, no component libraries)
Fonts         : Google Fonts CDN — Cormorant Garamond (display) + DM Sans (body)
Images        : Astro <Image /> component only — no raw <img> tags
Forms         : Netlify Forms via action POST (no JS, honeypot spam filter)
Deployment    : Vercel (primary) via GitHub Actions CI/CD
CI/CD         : GitHub Actions — see .github/workflows/deploy.yml
Language      : TypeScript (.astro files, .ts utilities)
Package mgr   : pnpm
Node          : >=20.x
i18n          : Astro native i18n — FR / EN / DE from day one
```

**Forbidden — no exceptions:**
- ❌ React / Vue / Svelte components (Astro islands only if strictly required)
- ❌ CSS frameworks other than Tailwind (no Bootstrap, no Bulma, no shadcn)
- ❌ Heavy animation libraries (no GSAP, no Framer Motion, no AOS)
- ❌ Client-side routing
- ❌ Any `<img>` tags without Astro `<Image />` optimization
- ❌ Inline styles (use Tailwind classes only)
- ❌ AI-generated images (Midjourney, DALL·E, Stable Diffusion) in any build
- ❌ Stock photos from Unsplash / Pexels / Shutterstock
- ❌ `any` type in TypeScript
- ❌ Édouard Morand's name as the site headline or hero element
- ❌ Named client references without Édouard's explicit written sign-off

---

## 3. Design System

### Color Palette

```css
/* Core */
--color-noir        : #0D0805;   /* Near-black — hero backgrounds, footer */
--color-brun        : #1E1208;   /* Dark chocolate — dark sections */
--color-marron      : #4A2E20;   /* Mid chocolate — accents, borders */
--color-caramel     : #C8915A;   /* Warm gold — CTAs, highlights, active states */
--color-creme       : #F8F3EB;   /* Warm cream — page backgrounds */
--color-or          : #A87832;   /* Deep gold — hover states, logo */

/* Accent (exploded view / green chocolate shell) */
--color-vert        : #2D5A1B;   /* Dark green shell — exploded view base */
--color-vert-lc     : #4A8A2E;   /* Mid green — eyebrow on dark sections */
```

### Typography

```
Display / Hero  : Cormorant Garamond, 300–400 weight, tracking tight to normal
Italic accent   : Cormorant Garamond italic — used for key words in headlines
Body / UI       : DM Sans, 300–500 weight
Caption / Meta  : DM Sans 300, UPPERCASE, letter-spacing: 0.2–0.35em
CTA / Button    : DM Sans 500, UPPERCASE, letter-spacing: 0.15em
```

**Rules:**
- All headings (H1–H3): Cormorant Garamond — never DM Sans
- Italic in Cormorant is a design tool: use it to highlight the essential word in a headline
- Never use system fonts, Inter, Roboto, Arial, or Space Grotesk
- Body line-height: 1.7–1.85
- Max prose line length: 65ch
- Hero H1: `clamp(3.5rem, 8vw, 7.5rem)` — two lines maximum

### Spacing Philosophy

- Sections breathe: `py-24` to `py-40` on desktop, `py-16` on mobile
- Content max-width: `max-w-6xl` (72rem) centered
- Prose columns: `max-w-2xl` or `max-w-3xl`
- Never center-align body text — left-align only (except hero headlines and quote blocks)

### Motion — CSS only, never gratuitous

```
Allowed:
  transition-duration : 300ms–500ms
  transition-timing   : cubic-bezier(0.25, 0.1, 0.25, 1)  [standard luxury ease]
  transition-timing   : cubic-bezier(0.16, 1, 0.3, 1)      [reveal ease — faster out]
  transform           : translateY(8–20px), opacity 0→1 on scroll reveal
  Hero entrance       : staggered fade-up, 200ms delay between elements
  Exploded view       : CSS float animation on SVG layers (keyframes, infinite, gentle)
  Custom cursor       : 10px → 40px on hover, mix-blend-mode multiply

Forbidden:
  - No rotation animations
  - No heavy scale transforms
  - No looping animations in hero (except scroll cue)
  - No parallax
  - No JS animation libraries
```

Scroll reveal pattern — `data-reveal` + `IntersectionObserver` in a single `<script>` tag:
```html
<div data-reveal data-reveal-delay="2">...</div>
```
Delay values: 1 = 100ms, 2 = 200ms, 3 = 300ms, 4 = 400ms.

### Grain texture

Applied as a CSS `::before` pseudo-element using an inline SVG data URI with `feTurbulence`. Opacity 0.04–0.08. Used on hero and all dark sections. Never a background-image file.

---

## 4. i18n — Three Languages from Day One

### Languages

| Code | Locale | Notes |
|---|---|---|
| `fr` | fr-CH | Default. Édouard writes in FR. Master copy. |
| `en` | en-US | Édouard reviews. Key for luxury international clients. |
| `de` | de-CH | Requires professional translation. Critical for Swiss German market. |

### Astro i18n configuration

```javascript
// astro.config.mjs
i18n: {
  defaultLocale: 'fr',
  locales: ['fr', 'en', 'de'],
  routing: { prefixDefaultLocale: false }
}
// → /          = French (default)
// → /en/       = English
// → /de/       = German
```

### Translation file structure

```
src/i18n/
├── fr.ts    ← master copy — all keys defined here first
├── en.ts    ← translated from fr.ts
└── de.ts    ← translated from fr.ts, professional translation required
```

### Rules

- All UI strings in i18n files — no hardcoded copy in components
- Language preference saved to `localStorage` key `'lc-lang'`
- Language switcher visible on all pages (top-right, minimal)
- `<html lang="{currentLocale}">` on every page
- hreflang meta tags on every page
- **German content**: never auto-translate with AI — use placeholder `[DE: traduction requise]` and flag it clearly in code comments until professional translation is provided

---

## 5. File Structure

```
/
├── .github/
│   ├── copilot-instructions.md     ← THIS FILE
│   └── workflows/
│       └── deploy.yml              ← GitHub Actions → Vercel
├── public/
│   ├── images/
│   │   └── placeholder/            ← design placeholders only (no stock, no AI)
│   │       └── .gitkeep
│   ├── favicon.svg
│   └── og-image.jpg                ← 1200×630, created manually (no AI)
├── src/
│   ├── i18n/
│   │   ├── fr.ts                   ← master translation file
│   │   ├── en.ts
│   │   └── de.ts
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Header.astro        ← transparent → sticky on scroll, trilingue
│   │   │   ├── Footer.astro
│   │   │   └── LanguageSwitcher.astro
│   │   ├── sections/
│   │   │   ├── Hero.astro          ← "L'architecte invisible du chocolat"
│   │   │   ├── Stats.astro         ← 5 key figures band
│   │   │   ├── ExplodedView.astro  ← SVG demi-sphère animée
│   │   │   ├── Services.astro      ← 4 service cards grid
│   │   │   ├── Expertise.astro     ← 2-column, timeline right
│   │   │   ├── Quote.astro         ← full-width philosophy quote
│   │   │   └── Contact.astro       ← form + info 2-column
│   │   └── ui/
│   │       ├── Button.astro
│   │       ├── ServiceCard.astro
│   │       ├── StatItem.astro
│   │       └── CaseStudyCard.astro
│   ├── content/
│   │   ├── config.ts               ← Astro content collections schema
│   │   ├── services/               ← .md files per service × 3 languages
│   │   └── case-studies/           ← anonymized references
│   ├── layouts/
│   │   ├── BaseLayout.astro        ← <html>, <head>, JSON-LD, hreflang
│   │   └── PageLayout.astro        ← header + footer wrapper
│   ├── pages/
│   │   ├── index.astro             ← Home (FR default)
│   │   ├── en/
│   │   │   └── index.astro         ← Home EN
│   │   ├── de/
│   │   │   └── index.astro         ← Home DE
│   │   ├── services/
│   │   │   ├── index.astro
│   │   │   ├── consulting.astro
│   │   │   ├── formations.astro
│   │   │   ├── audit.astro
│   │   │   └── evenementiel.astro
│   │   ├── references.astro        ← anonymized case studies
│   │   ├── contact.astro
│   │   └── merci.astro             ← post-form redirect (excluded from sitemap)
│   ├── styles/
│   │   └── global.css              ← Tailwind directives + CSS custom properties
│   └── utils/
│       ├── seo.ts                  ← SEO props builder
│       ├── schema.ts               ← JSON-LD structured data
│       └── i18n.ts                 ← getLocale, t() helper
├── vercel.json                     ← headers, redirects (www → apex)
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
├── SPECS.md
└── package.json
```

**Note on `/edouard` page**: There is **no public `/edouard` biography page** in the MVP. Édouard's identity is not the site's headline. His expertise is present throughout all pages implicitly. A brief, discreet "about" section may exist on the homepage — linking nowhere. Revisit in v2 if Édouard decides to become more visible.

---

## 6. Key Components

### 6.1 ExplodedView.astro — The Signature Visual

The exploded view of a green-painted chocolate demi-sphere is the site's most distinctive visual element. It must be built as a pure SVG with CSS animations — no JS, no canvas, no WebGL.

```
Layers (top → bottom, each floating independently):
  1. Green painted shell — upper hemisphere
     - Radial gradient: #7DB85A → #4A8A2E → #2D5A1B → #1A3A0D
     - Marble swirl overlay (SVG path, low opacity)
     - Specular highlight (white radial, top-left)
     - CSS animation: translateY 0 → -18px, 4s ease-in-out infinite

  2. Ganache disc
     - Dark brown radial gradient
     - CSS animation: translateY 0 → -10px, 4s ease-in-out 0.3s infinite

  3. Praliné / interior disc
     - Warm cream/caramel gradient, subtle texture lines
     - CSS animation: translateY 0 → -14px, 4s ease-in-out 0.15s infinite

  4. Dark chocolate shell — lower bowl
     - Deep chocolate radial gradient
     - Inner cavity highlight (arc path)
     - Static (grounding element)

Labels: absolute positioned, connected by 1px caramel lines
  - "Coque peinte · Beurre de cacao"
  - "Ganache grand cru"
  - "Praliné maison"
  - "Coque chocolat noir"
  Labels hidden on mobile (< 768px)

Section background: --color-noir (#0D0805)
Dimension reference lines: dashed caramel, 15% opacity
Floating particles: 5–6 dots, caramel + green, low opacity
```

**This component is a business statement, not decoration.** It communicates instantly: there is a master craftsperson behind this site who understands chocolate at a molecular level.

### 6.2 Hero.astro

```
Headline (FR): "L'architecte / invisible / du chocolat."
Headline (EN): "The invisible / architect / of chocolate."
Headline (DE): "Der unsichtbare / Architekt / der Schokolade."

"invisible" / "architect" / "unsichtbare" → Cormorant italic, color: caramel

Pre-headline: location + positioning tagline (DM Sans, caramel, uppercase, tracking-widest)
Sub-headline: 1 sentence, DM Sans 300, rgba(white, 0.55), max 540px
CTA primary: "Parlons de votre projet" → #contact
CTA ghost: "Découvrir l'expertise" → #services (with → arrow)
Scroll cue: animated vertical line + "Défiler" label

Background: --color-brun (#1E1208) + grain texture overlay
No photo, no video, no image in hero
```

### 6.3 Stats.astro

Five statistics in a horizontal band on `--color-creme` background:

```
20+      → années d'expérience / en chocolaterie haut de gamme
MOF 2015 → Finaliste / Meilleur Ouvrier de France
700t     → Production / Produit fini / an · sourcing direct Cameroun
B→B      → Bean-to-Bar / De la fève à la tablette
DBA      → Doctorat / Business Administration
```

Values in Cormorant Garamond, caramel. Horizontal scroll on mobile.

### 6.4 Header.astro

```
Navigation links (FR | EN | DE variants):
  - L'expertise / Expertise / Expertise    → #expertise
  - Services / Services / Leistungen       → #services (or /services)
  - Références / References / Referenzen  → /references
  - Contact / Contact / Kontakt           → #contact

CTA: "Votre projet / Your project / Ihr Projekt" → #contact

Behavior:
  - Transparent on dark hero
  - background: rgba(13,8,5,0.95) + backdrop-filter blur(20px) after 80px scroll
  - Border-bottom: rgba(caramel, 0.15) when scrolled
  - Language switcher: top-right, always visible
  - Mobile: hamburger → full-screen overlay, dark background
```

---

## 7. Image Strategy — Pre-Shooting Phase

**The photography shoot is scheduled for late March 2026.** Until then:

### What to use
- The `ExplodedView.astro` SVG component (autonomous, no image file)
- CSS grain textures (inline SVG data URI)
- Dark color fields with subtle gradients
- Geometric/abstract placeholder blocks with `<!-- REPLACE: [description] -->` comment

### What is strictly forbidden
- ❌ AI-generated images (Midjourney, DALL·E, Stable Diffusion, ChatGPT image)
- ❌ Stock photos (Unsplash, Pexels, Shutterstock)
- ❌ Random chocolate photos from the internet
- ❌ The uploaded reference photo of the green bonbons (that is Édouard's actual work — do not use without permission, do not redistribute)

### Post-shooting
Expected deliverables from photographer (March 2026):
- Édouard's hands at work (no face required — consistent with discretion)
- Detail shots: painted shells, ganache, molds, tempering chocolate
- Atelier ambiance
- Product beauty shots of finished bonbons
- All: RAW + WebP 1920w + 800w + 600w versions

---

## 8. Content & Tone Guidelines

### Brand voice

**Primary language**: French (fr-CH) — master copy written here first.
**Register**: Luxury B2B. Understated. Precise. Never cold.
**Rhythm**: Short sentences separated by white space. Prose, not bullet lists.

**Use:**
- justesse, rigueur, maîtrise, exigence, savoir-faire
- "derrière les plus grandes maisons" — the shadow/architect angle
- Numbers as proof (700t, 20 ans, MOF)

**Never use:**
- passion, excellence, unique, révolutionnaire, incroyable
- Superlatives without evidence
- "Je" or first person (the brand speaks, not the person)
- Any language that announces ambition loudly

### What to never write

- ❌ Édouard Morand's name as the hero headline
- ❌ Named client references without explicit written authorization
- ❌ Price lists or day rates on any public page
- ❌ Fabricated testimonials or invented awards
- ❌ Biographical details not confirmed (see confirmed facts below)
- ❌ Any copy that reveals competitive strategy

### Confirmed facts — use freely in copy

| Fact | Notes |
|---|---|
| MOF 2015 Finalist (Chocolatier-Confiseur) | Confirmed |
| Chocolaterie Stettler, Geneva | Confirmed |
| Hôtel du Cap-Eden-Roc, Antibes | Confirmed |
| Jaeger-LeCoultre collaboration | Confirmed |
| Co-founder, Fondation Haute Chocolaterie | Confirmed |
| DBA (Doctorat en Business Administration) | Confirmed |
| Direct sourcing, Cameroon plantations | Confirmed |
| 600–700 tonnes finished product / year | Confirmed |
| Creates for leading Swiss chocolate houses (unnamed) | Confirmed — never name them |
| Luxury commissions: chocolate × caviar | Confirmed |
| Based in Aubonne, Vaud, Switzerland | Confirmed |
| Instagram perso: @edouard.morand | Do not link from site (personal) |
| Instagram pro: @lechocolatiersuisse | Link in footer |

### Tone per language

| Language | Register | Key watch-out |
|---|---|---|
| FR | Refined, slightly literary | Never corporate, never startup |
| EN | Precise, minimal | Avoid Americanisms — use British spelling where ambiguous |
| DE | Formal (Sie form), precise | Professional translation required — never use AI auto-translate |

---

## 9. Deployment Architecture

```
GitHub (main branch)
    ↓ push
GitHub Actions (.github/workflows/deploy.yml)
    ↓ pnpm install → typecheck → build
Vercel (production)
    ↓
lechocolatiersuisse.ch
```

### Branch strategy

```
main        → production auto-deploy
develop     → staging / preview deploy
feature/*   → feature branches → PR to develop
fix/*       → hotfix branches → PR to main
content/*   → copy and i18n updates only
```

### GitHub secrets required

```
VERCEL_TOKEN        → vercel.com → Account Settings → Tokens
VERCEL_ORG_ID       → .vercel/project.json after `vercel link`
VERCEL_PROJECT_ID   → .vercel/project.json after `vercel link`
```

### vercel.json

```json
{
  "buildCommand": "pnpm run build",
  "outputDirectory": "dist",
  "framework": "astro",
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options",        "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy",        "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy",     "value": "camera=(), microphone=(), geolocation=()" }
      ]
    }
  ],
  "redirects": [
    {
      "source": "/",
      "has": [{ "type": "host", "value": "www.lechocolatiersuisse.ch" }],
      "destination": "https://lechocolatiersuisse.ch/",
      "permanent": true
    },
    {
      "source": "/",
      "has": [{ "type": "host", "value": "preprod.lechocolatiersuisse.ch" }],
      "destination": "https://lechocolatiersuisse.ch/",
      "permanent": true
    }
  ]
}
```

---

## 10. Commit Conventions (Conventional Commits)

```
feat(hero):         add animated headline entrance
feat(i18n):         add German translations for services section
fix(nav):           correct mobile menu z-index on iOS Safari
content(fr):        update consulting service copy
content(de):        [DE PLACEHOLDER] awaiting professional translation
style(colors):      align caramel token across all hover states
perf(images):       convert hero placeholder to WebP
a11y(form):         add aria-describedby to email error message
chore(deps):        upgrade Astro to 4.x
```

---

## 11. Copilot Agent — Mandatory Workflow

When working on any task in this repo, Copilot must:

1. **Read this file first** — entirely, before generating a single line of code
2. **Check brand identity** — would this output embarrass a world-class chocolate professional? If unsure, do less
3. **Language check** — every user-facing string must exist in `fr.ts`, `en.ts`, and `de.ts`. German can be `[DE: traduction requise]` placeholder but must exist
4. **Image discipline** — never reference external image URLs, stock photos, or AI-generated images. Use `public/images/placeholder/` paths with `<!-- REPLACE: [what goes here] -->` comments
5. **JS budget** — flag any addition that would add more than 10kb of JS. The site should be near-zero JS
6. **Static first** — if it can be static HTML, make it static
7. **Mobile first** — write mobile CSS first, desktop as `md:` or `lg:` overrides
8. **Component size** — no single `.astro` component > 150 lines of markup
9. **No client names** — never add named client references not explicitly in Section 8 of this file
10. **No Édouard as hero** — his name can appear in meta descriptions and structured data, never as the page headline
11. **Do not modify this file** unless explicitly instructed by the project owner

---

## 12. Definition of Done

A page or component is complete when all boxes are checked:

- [ ] Renders correctly: mobile 375px, tablet 768px, desktop 1440px
- [ ] Lighthouse Performance ≥ 95 (mobile)
- [ ] Lighthouse Accessibility ≥ 95
- [ ] All images via `<Image />` with `alt`, `width`, `height`
- [ ] All `<!-- REPLACE -->` placeholders documented
- [ ] No TypeScript errors (`pnpm run typecheck`)
- [ ] All UI strings in `fr.ts` / `en.ts` / `de.ts`
- [ ] No hardcoded copy in `.astro` components
- [ ] Structured data (JSON-LD) present on page-level files
- [ ] Keyboard-navigable, focus rings visible
- [ ] PR description: what changed, why, which pages affected

---

*Last updated: March 2026 — v2.0*
*Reflects all decisions made through conversation: branding discretion, Vercel deployment, i18n FR/EN/DE, exploded view, Goldman positioning, pre-shooting image strategy.*
*Do not edit without the project owner's approval.*