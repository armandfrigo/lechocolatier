1. Project Identity
Who: Édouard Morand — master chocolatier, MOF 2015 finalist, DBA, co-founder of Fondation Haute Chocolaterie.
What: His personal consulting brand website — lechocolatiersuisse.ch
Why it exists: To position him as the foremost chocolate consultant in Switzerland — for luxury houses, industrial clients, and premium B2B projects.
Core tension: 20 years working in the shadow of great names. This site is the first time he speaks under his own name. Every line of code should honor that weight.

2. Tech Stack — Non-Negotiable
Framework     : Astro 4.x (static-first, zero JS by default)
Styling       : Tailwind CSS 3.x (utility-only, no component libraries)
Fonts         : Google Fonts CDN — Cormorant Garamond (display) + DM Sans (body)
Images        : Astro <Image /> component only — no raw <img> tags
Forms         : Netlify Forms (no backend, no JS form libraries)
Deployment    : Netlify (or Vercel as fallback)
Language      : TypeScript (.astro files, .ts utilities)
Package mgr   : pnpm
Node          : >=20.x
Forbidden:

❌ React / Vue / Svelte components (unless strictly necessary for interactivity)
❌ CSS frameworks other than Tailwind (no Bootstrap, no Bulma)
❌ Heavy animation libraries (no GSAP, no Framer Motion)
❌ Client-side routing
❌ Any <img> tags without Astro optimization
❌ Inline styles (use Tailwind classes only)
❌ AI-generated placeholder images in production builds


3. Design System
Color Palette
css--color-noir        : #1A0F0A;   /* Deep chocolate black — primary text, headers */
--color-brun        : #2B1B12;   /* Dark chocolate — backgrounds, footers */
--color-marron      : #5A3A2E;   /* Mid chocolate — accents, borders */
--color-caramel     : #C8955A;   /* Warm gold — CTAs, highlights, hover states */
--color-creme       : #FAF6F0;   /* Warm cream — page backgrounds */
--color-blanc       : #FFFFFF;   /* Pure white — cards, modals */
--color-or          : #B8973A;   /* Gold — logo accent, premium markers */
Typography
Display / Hero  : Cormorant Garamond, 300–600 weight, tracking wide
Body / UI       : DM Sans, 300–500 weight
Caption / Meta  : DM Sans, 300, uppercase, letter-spacing: 0.15em
Rules:

Headings in Cormorant Garamond, never DM Sans
Never use system fonts, Inter, Roboto, or Arial
Line-height for body text: 1.7–1.8
Max line length (prose): 65ch

Spacing Philosophy

Generous vertical rhythm — sections breathe (py-24 to py-40 on desktop)
Mobile: py-16 minimum between sections
Content max-width: max-w-6xl (72rem) centered
Text columns: max-w-2xl or max-w-3xl for readability

Motion
css/* Allowed: subtle, CSS-only */
transition-duration: 300ms–500ms
transition-timing:   cubic-bezier(0.25, 0.1, 0.25, 1)
transform:           translateY(0.5rem), opacity(0→1) on scroll reveal

/* Forbidden */
- No rotation animations
- No heavy scale transforms
- No looping animations in hero
- No parallax (hurts performance and mobile UX)
Use the data-animate attribute pattern for scroll reveals:
html<div data-animate="fade-up" data-delay="100">...</div>
Implement with a lightweight IntersectionObserver snippet in a single <script> tag.

4. File Structure
/
├── .github/
│   └── copilot-instructions.md     ← THIS FILE
├── public/
│   ├── fonts/                      (self-hosted fallback if needed)
│   ├── favicon.svg
│   └── og-image.jpg                (1200x630 OG image)
├── src/
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Header.astro
│   │   │   ├── Footer.astro
│   │   │   └── Navigation.astro
│   │   ├── sections/
│   │   │   ├── Hero.astro
│   │   │   ├── About.astro
│   │   │   ├── Services.astro
│   │   │   ├── Expertise.astro
│   │   │   ├── Testimonials.astro
│   │   │   └── Contact.astro
│   │   └── ui/
│   │       ├── Button.astro
│   │       ├── Card.astro
│   │       ├── Divider.astro
│   │       └── Tag.astro
│   ├── content/
│   │   ├── services/               (Astro content collections)
│   │   └── case-studies/
│   ├── layouts/
│   │   ├── BaseLayout.astro
│   │   └── PageLayout.astro
│   ├── pages/
│   │   ├── index.astro             (Home)
│   │   ├── edouard.astro           (About / Biography)
│   │   ├── services/
│   │   │   ├── index.astro
│   │   │   ├── consulting.astro
│   │   │   ├── formations.astro
│   │   │   ├── audit.astro
│   │   │   └── evenementiel.astro
│   │   ├── references.astro        (Case Studies — anonymized)
│   │   └── contact.astro
│   ├── styles/
│   │   └── global.css              (Tailwind @base overrides only)
│   └── utils/
│       ├── seo.ts
│       └── schema.ts               (JSON-LD structured data)
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
└── package.json

5. Pages Specification
5.1 Home (index.astro)
Sections in order:

Hero — Full-viewport, dark background (#1A0F0A or #2B1B12), large Cormorant headline, single CTA
ValueProp — 3 columns: "Créateur", "Consultant", "Formateur" — icon + text
About — Short intro on Édouard, photo (real, professional), link to full biography
Services — 4 service cards: Consulting opérationnel / Formations / Audit stratégique / Expériences
Expertise — Key figures: 20+ ans | MOF 2015 Finaliste | 600–700t | Bean-to-bar | DBA
Testimonials — 2–3 testimonials (anonymized, with role/sector)
Contact CTA — Full-width section, dark background, single button

Hero rules:

No video background
No auto-playing media
Headline: max 2 lines, large (text-6xl md:text-8xl)
Sub-headline: 1 sentence maximum
CTA: "Parlons de votre projet →" → links to /contact
One secondary link: "Découvrir le parcours" → links to /edouard

5.2 About (edouard.astro)
Full biography of Édouard Morand. Key content beats (in order):

Portrait photo (professional, not AI-generated)
"Vingt ans dans l'ombre des grands. Aujourd'hui, à la lumière."
Career narrative: Stettler → Hôtel du Cap-Eden-Roc → MOF 2015 finalist
Expertise pillars: Bean-to-bar / Luxury creation / Industrial scale
Awards & distinctions (MOF finaliste 2015, DBA, Fondation Haute Chocolaterie)
Philosophy quote in large Cormorant type

5.3 Services (services/index.astro + sub-pages)
Each service page must include:

Title + subtitle
What it is (2–3 paragraphs)
Who it's for (target client)
What you get (deliverables)
A real-world example (anonymized if confidential)
CTA: "Discutons de votre projet"

5.4 References (references.astro)
Anonymized case studies. Format per reference:
Sector  : Maison horlogère suisse (Vallée de Joux)
Project : Création d'une collection chocolat-caviar pour événement de lancement
Scale   : Édition limitée, 500 pièces
Result  : [Outcome in 1 sentence]
Never name clients without explicit written authorization from Édouard.
5.5 Contact (contact.astro)

Netlify Form (no JS required)
Fields: Nom / Entreprise / Email / Type de projet (select) / Message
Type de projet options: Consulting opérationnel | Formation | Audit | Expérience & Événementiel | Autre
After submit → redirect to /merci page (create it)
Physical: Aubonne, Suisse
Email: info@le-chocolatier.ch
Instagram: @lechocolatiersuisse
No phone number unless Édouard provides one


6. SEO & Performance Rules
Meta & Open Graph
Every page must use the seo.ts utility and include:
typescript// Required for every page
title: string          // "Page Name — Édouard Morand | Le Chocolatier"
description: string    // 150–160 chars, includes "chocolatier suisse", "consultant"
ogImage: string        // absolute URL, 1200x630
canonical: string      // absolute URL
lang: "fr"
Structured Data (JSON-LD)
Implement in schema.ts and inject in BaseLayout:

Person schema for Édouard Morand
ProfessionalService schema for each service
LocalBusiness for the contact page (Aubonne, CH)

Performance Targets
MetricTargetLighthouse Performance≥ 95Lighthouse Accessibility≥ 95Lighthouse SEO100Lighthouse Best Practices100LCP< 2.5sCLS< 0.1FID / INP< 100ms
Image Rules

Every image: WebP format, responsive sizes attribute, explicit width + height
Hero image: loading="eager" + fetchpriority="high"
All other images: loading="lazy"
Alt text: descriptive, never empty, never "image" or filename
No images > 200kb after optimization


7. Accessibility (WCAG 2.1 AA)

Every interactive element: visible focus ring (focus-visible:ring-2 ring-offset-2)
Color contrast: minimum 4.5:1 for normal text, 3:1 for large text
All images: meaningful alt attributes
Navigation: keyboard-navigable, skip-to-main link
Forms: <label> for every input, error messages associated via aria-describedby
No ARIA attributes added speculatively — only when they solve a real accessibility problem


8. Content & Tone Guidelines
Voice

French (fr-CH), formal but warm — never corporate-cold
Luxury register: understated confidence, no superlatives
Short sentences. White space in prose is intentional.
Avoid: "passion", "excellence", "unique" (overused in luxury copy)
Prefer: precision, justesse, savoir-faire, rigueur, exigence

What to Never Write

❌ No mention of competitors or other chocolatiers by name
❌ No price lists or rate cards on public pages
❌ No client names without Édouard's authorization
❌ No fabricated testimonials or invented awards
❌ No AI-generated biographical details not confirmed by Édouard

Confirmed Facts (use freely)

MOF 2015 Finalist (Chocolatier-Confiseur)
Worked at Chocolaterie Stettler (Geneva)
Worked at Hôtel du Cap-Eden-Roc (Antibes)
Collaborated with Jaeger-LeCoultre (chocolate creation)
Co-founder, Fondation Haute Chocolaterie
DBA (Doctorat en Business Administration)
Direct sourcing from Cameroon plantations
Production scale: 600–700 tonnes finished product / year
Creates for leading Swiss chocolate houses (unnamed)
Luxury commissions including chocolate × caviar creations
Based in Aubonne, Vaud, Switzerland


9. Git & Code Conventions
Branch Strategy
main          → production (Netlify deploys from here)
develop       → integration branch
feature/*     → feature branches (e.g. feature/hero-section)
fix/*         → bug fixes
content/*     → copy and content updates only
Commit Messages (Conventional Commits)
feat(hero): add animated headline reveal on scroll
fix(nav): correct mobile menu close on route change
content(about): update Édouard biography section
style(colors): align caramel accent across all CTAs
perf(images): convert hero to WebP, add responsive sizes
a11y(contact): add aria-describedby to form error messages
Code Style

TypeScript strict mode: "strict": true in tsconfig
No any types
Astro components: Props interface defined at top of <script> block
Tailwind: no @apply in components — inline classes only
No !important anywhere
CSS custom properties for all brand colors (defined in global.css)


10. Copilot Agent — Workflow Rules
When Copilot Agent is working on a task, it must:

Read this file first before generating any code
Ask before naming clients — never add client or brand names not in section 8
Never use placeholder images from picsum, unsplash, or AI generators — use local public/images/placeholder-[context].jpg paths instead, with a comment <!-- REPLACE: professional photo of [description] -->
Check Lighthouse budget — flag any suggestion that would add > 10kb of JS
Prefer static over dynamic — if something can be static HTML, make it static
Mobile-first always — write mobile CSS first, desktop as md: / lg: overrides
One component, one file — no component should exceed 150 lines of Astro markup
Always add structured data when creating or editing page files
Never modify copilot-instructions.md unless explicitly asked by Édouard


11. Environment & Secrets
# .env.local (never commit)
PUBLIC_SITE_URL=https://lechocolatiersuisse.ch
CONTACT_EMAIL=info@le-chocolatier.ch

# No API keys required for MVP
# Netlify handles form processing server-side

12. Definition of Done
A feature or page is complete when:

 Renders correctly on mobile (375px), tablet (768px), desktop (1440px)
 Lighthouse score ≥ 95 performance on mobile (tested with pnpm run lighthouse)
 All images use <Image /> with alt text, width, height
 No TypeScript errors (pnpm run typecheck)
 No Tailwind classes outside of component files
 French copy reviewed and no lorem ipsum remaining
 Structured data (JSON-LD) present on the page
 Accessible: keyboard-navigable, contrast passes
 PR description explains what changed and why


Last updated: March 2026 — Édouard Morand / Le Chocolatier Suisse
Maintained by: the development team — do not edit without Édouard's approval