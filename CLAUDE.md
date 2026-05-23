# CLAUDE.md - Noctorya Shopify Dawn Fork (Theme Repo Rules)

NOTE: This file is at the dawn/ (theme repo) root. Claude Code auto-loads it when started with dawn/ as cwd. Paths prefixed with ../ refer to sibling directories inside the NOCTORYA/ parent (01_BLUEPRINT/, 08_DEEP_RESEARCH/, etc.) — this is the standard Phase D workflow where Claude Code launches from dawn/ and reads spec docs from sibling folders.

Auto-loaded by Claude Code when a session opens files in this repository. Permanent
rules apply to ALL theme work.

---

## MANDATORY READING - DO THIS FIRST

Before producing ANY code in this repository, read these files in order:

1. `../01_BLUEPRINT/Noctorya_MasterBlueprint_Part2_Design_Build.md` - design bible.
   Section 11 contains all design tokens. Section 9 contains page-by-page wireframes.
   NEVER invent values; reference this document.
2. `../01_BLUEPRINT/Noctorya_Master_Task_Dispatcher.md` - confirm which Task you are executing.
3. `../08_DEEP_RESEARCH/folder3_design/design_system.md` - complete design system (308 lines).
4. `../08_DEEP_RESEARCH/folder3_design/component_specs.md` - reusable component specifications
   (Navigation, Product Card, Hero, Footer, Cart Drawer, etc.).
5. `../08_DEEP_RESEARCH/folder2_strategy/voice_canon.md` - binding voice rules for any
   customer-facing microcopy (button labels, error messages, empty states, ATC text).
6. `assets/brand-tokens.css` - design tokens encoded as 108 CSS custom properties.

If any of the above is missing or inaccessible, STOP. Report to Eduard. Do not proceed
using only the safety net below.

---

## THEME ARCHITECTURE OVERVIEW

### Dawn fork strategy

Working code lives on `main`. The `live/noctorya` branch is the deploy target.
Develop on `main`, test via `shopify theme dev`, merge to `live/noctorya`, push to Shopify.

### Custom sections (Master Dispatcher Tasks 053-062)

Nine custom sections live in `sections/`. Each has its own CSS file in `assets/`:

- `trust-bar` (4 icons + text row, mobile 2x2 grid)
- `content-teaser` (2-column: painting image + blog excerpt with picker schema)
- `artist-scroll` (horizontal scroll using Dawn `<slider-component>`, Artist Metaobject data)
- `movement-grid` (3x2 grid of movement cards, painting bg + gradient overlay)
- `why-choose-us` (comparison table, accent column highlight)
- `painting-of-the-week` (2-column painting + story, product picker in schema)
- `size-comparison-tool` (reads product metafields width_cm, height_cm; SVG silhouettes)
- `artist-biography` (Artist Metaobject driven; portrait + bio + Quick Facts + paintings grid)
- `links-page` (ultra-light bio link page; system fonts; inline CSS; <2s LCP on 3G)

### Dawn-modified sections (Tasks 063-067)

- `hero` (right-aligned text, "THE ART YOU LIVE WITH")
- `header` (mega menu with featured image, hover-swap pattern)
- `main-product` (sticky gallery left, accordion sections right, app blocks)
- `cart` (drawer desktop, page mobile via JS redirect)
- `main-product` mobile (sticky ATC slide-up drawer)

---

## DESIGN SYSTEM INTEGRATION

### brand-tokens.css

`assets/brand-tokens.css` contains 108 CSS custom properties in 11 sections:
COLORS, TYPOGRAPHY, SPACING, LAYOUT, BORDERS, SHADOWS, ANIMATIONS, BREAKPOINTS,
Z-INDEX SCALE (flagged for future definition), FOCUS INDICATORS, BUTTONS.

A Dawn override block at the bottom redefines Dawn's RGB-triplet color variables
(`--color-background`, `--color-foreground`, etc.) plus Dawn's font family/weight
variables.

### Loading order

In `layout/theme.liquid`, load order is:

1. `assets/base.css` (Dawn baseline + Noctorya global rules: font-smoothing, line-height)
2. `assets/brand-tokens.css` (Noctorya tokens + Dawn overrides)
3. Section-scoped CSS via `{{ 'section-name.css' | asset_url | stylesheet_tag }}` per section.

### Reference pattern

Component CSS references tokens via `var()`:

```css
.product-card__title {
  color: var(--color-text-primary);
  font-family: var(--font-body);
  font-size: var(--fs-body-mobile);
  letter-spacing: var(--ls-body);
}
@media (min-width: 990px) {
  .product-card__title { font-size: var(--fs-body-desktop); }
}
```

NEVER redeclare a token; NEVER hard-code a value that already has a token.

### Z-index policy

Z-index scale: NOT YET DEFINED in design system. Until defined, use explicit
numeric values inline with a comment justifying the stacking context. Escalate
to Eduard when proposing additions to the stacking system.

Scale will be defined when first component requiring stacking context is built
(Tasks 053+: dropdown menu, modal, cart drawer, sticky ATC). NOT proactively
defined ad-hoc.

---

## LIQUID PATTERNS

### capture vs assign

Use `{% capture %}` for multi-step string composition; use `{% assign %}` for
single-value bindings or single-filter operations.

```liquid
{%- capture artist_full_name -%}{{ artist.first_name }} {{ artist.last_name }}{%- endcapture -%}
{%- assign primary_color = settings.color_accent -%}
{%- assign sorted = collection.products | sort: 'price' -%}
```

Avoid `{% assign %}` for multi-step concatenation (works but is harder to read than capture).

### render not include

Liquid `{% include %}` is deprecated. Always use `{% render %}` for snippets.

CORRECT: `{% render 'card-product', product: product, lazy: true %}`
INCORRECT: `{% include 'card-product' %}` (deprecated; may fail in future Shopify versions)

`{% render %}` accepts named parameters and isolates scope. Pass everything the snippet
needs explicitly.

### App blocks in custom sections

When a custom section accepts app blocks, use the `@app` block-type pattern:

```liquid
{%- for block in section.blocks -%}
  {%- case block.type -%}
    {%- when 'text' -%}
      <p>{{ block.settings.body }}</p>
    {%- when '@app' -%}
      {% render block %}
  {%- endcase -%}
{%- endfor -%}
```

Add `{ "type": "@app" }` to the section schema's `blocks` array.

### Schema settings vs theme settings vs metafields

- Schema settings: per-section editable values (visible in Customizer for that section).
  Use for: section title, product picker, blog post picker, background color choice.
- Theme settings (`config/settings_schema.json`): global values applying to all pages.
  Use for: brand logo, primary CTA color override, social media URLs.
- Product metafields: per-product structured data set in Shopify admin.
  Use for: width_cm, height_cm, frame_type, artist reference.
- Section metafields and metaobject references: structured data referenced from custom sections.
  Use for: Artist Metaobject driving artist biography section.

### Section padding mobile pattern

Section padding on mobile is desktop value times 0.75:

```liquid
{%- assign mobile_pad = section.settings.padding_top | times: 0.75 | round: 0 -%}
<style>
  #shopify-section-{{ section.id }} { padding-top: {{ mobile_pad }}px; }
  @media (min-width: 990px) { #shopify-section-{{ section.id }} { padding-top: {{ section.settings.padding_top }}px; } }
</style>
```

This is the ONE allowed inline style pattern (template-driven, per-section).

---

## CSS ARCHITECTURE

### BEM convention

CSS class naming follows BEM (Block - Element - Modifier). Block names use kebab-case;
elements attach to the block with double underscore (`__`); modifiers attach to either
a block or an element with double hyphen (`--`). Names describe ROLE, not appearance.

**Correct patterns**

```css
.product-card                           /* Block - standalone component */
.product-card__title                    /* Element - belongs to .product-card */
.product-card__price                    /* Element - sibling element on same block */
.button--primary                        /* Modifier on a block - variant of .button */
.product-card__title--featured          /* Modifier on an element - state of a title */
.site-header__nav                       /* Hyphenated block + element - both valid */
.cart-drawer__line-item                 /* Multi-word element - kebab-case inside __ */
.button--primary.button--large          /* Composition - two modifiers on one block */
```

Per-example pedagogical notes:

- `.product-card` - Block is a self-contained component. Two-word name in kebab-case;
  no underscores yet because no element or modifier is involved. A block can stand
  alone in HTML without any siblings.
- `.product-card__title` - Double underscore signals "this element BELONGS to the
  block". The block name is repeated in full (never abbreviated); the element name
  is kebab-case after `__` (write `__line-item`, not `__lineItem` and not `__li`).
- `.product-card__price` - Sibling element. Multiple elements share the same block
  prefix; each describes a distinct part. The element name describes function, not
  position (write `__price`, not `__bottom-text`).
- `.button--primary` - Double hyphen signals "this is a VARIANT of the block". The
  block name is preserved in full; the modifier describes the variant by purpose
  (`primary`, `secondary`, `ghost`), not appearance (`red`, `bold`, `big`).
- `.product-card__title--featured` - Modifier attached to an ELEMENT, not the block.
  Triple-segment (`block__element--modifier`) is the legitimate maximum nesting depth
  in BEM. Anything deeper is a sign you need a new block.
- `.site-header__nav` - Hyphens are allowed INSIDE a block name (`site-header`); the
  element separator is still double underscore. Do not confuse `-` (word separator)
  with `__` (block-element separator).
- `.cart-drawer__line-item` - Multi-word elements also use kebab-case after the `__`.
  The `__` only separates block from element; the element name itself stays
  kebab-case for any internal word breaks.
- `.button--primary.button--large` - Composition via two modifiers stacked on the
  same element in HTML. Each modifier is applied as a SEPARATE class; you NEVER
  write `.button--primary--large` (which would imply "primary-large" is a single
  modifier name). In HTML:
  `<button class="button button--primary button--large">SHOP</button>`.

**Incorrect patterns**

```css
.productCardTitle                       /* camelCase - violates kebab-case rule */
.product_card_title                     /* Single underscores - ambiguous, not BEM */
.product-card-title                     /* Single hyphen between block and element */
.product-card__title__featured          /* Double element nesting - forbidden */
.product-card--title                    /* Modifier syntax used for an element */
.red-title                              /* Color in name - not semantic */
.title-featured                         /* Modifier without parent block - dangling */
.button--primary--large                 /* Two modifiers fused - should be separate */
```

Per-example pedagogical notes:

- `.productCardTitle` - camelCase violates BEM's kebab-case rule. Switch to
  `.product-card__title`. CamelCase is a JavaScript identifier convention, not a CSS
  class convention in this repo.
- `.product_card_title` - Single underscores are ambiguous between word-separator
  and element-separator. BEM uses kebab-case (`-`) for words and double underscore
  (`__`) for the block-element boundary. Pick one role per delimiter and stick to it.
- `.product-card-title` - Single hyphen between `card` and `title` could mean either
  "the title of a card" or "a card-title component in its own right". BEM removes
  that ambiguity with `__`. Switch to `.product-card__title`.
- `.product-card__title__featured` - Elements DO NOT nest in BEM. If you find
  yourself writing `__x__y`, the deeper segment is either a modifier
  (`--featured`) or a separate block (`.featured-badge`) that happens to live
  inside the card.
- `.product-card--title` - Modifiers are for VARIANTS, not for elements. The `--`
  syntax describes "a variant of the parent thing"; an element is a different
  concept and uses `__`. Switch to `.product-card__title`.
- `.red-title` - Names should describe ROLE, not APPEARANCE. If the design rebrands
  and red becomes silver, the class name is now wrong. Use
  `.product-card__title--featured` (purpose) or `.product-card__title--accent`
  (semantic emphasis) so the class survives a visual refresh.
- `.title-featured` - Modifiers are valid only when attached to a block or element
  they modify. A free-floating modifier has no parent and cannot exist alone.
  Either attach it (`.product-card__title--featured`) or promote the concept to a
  block in its own right (`.featured-callout`).
- `.button--primary--large` - Two modifiers cannot be fused with `--`. Each modifier
  is independent; in HTML, apply them as two separate classes:
  `<button class="button button--primary button--large">`. In CSS, target each
  modifier with its own selector. Fusing them creates exponential class growth as
  variants multiply.

**Edge case: utility classes are NOT BEM**

Single-purpose utility classes (`.u-hidden`, `.text-center`, `.mt-md`, `.is-active`,
`.has-error`) live in a separate namespace and DO NOT follow BEM. They are
intentionally short, prefixed (`u-`, `is-`, `has-`) or descriptive
(`text-center`), and applied directly in HTML alongside BEM block classes:

```html
<article class="product-card is-loading">...</article>
<p class="product-card__price text-center">...</p>
```

Do NOT retrofit BEM onto utility names (`.product-card__u-hidden` is wrong on both
counts). Do NOT mix BEM and utility conventions in the same class name. Keep the
two namespaces separate by convention; this repo's utility namespace is documented
in `assets/base.css` and `assets/brand-tokens.css` comments.

### Mobile-first @media pattern

Default styles target mobile (0-749px); tablet and desktop layered via min-width:

```css
.product-card__title { font-size: var(--fs-body-mobile); }
@media (min-width: 750px) { .product-card__title { font-size: var(--fs-body-tablet); } }
@media (min-width: 990px) { .product-card__title { font-size: var(--fs-body-desktop); } }
```

CSS custom properties cannot be used inside @media thresholds; duplicate the literal
`750px` and `990px` values per Dawn standard.

### Component CSS scoping

One CSS file per top-level section or snippet; file name matches the Liquid file.
Example: `sections/trust-bar.liquid` -> `assets/trust-bar.css`.

Load via `{{ 'trust-bar.css' | asset_url | stylesheet_tag }}` inside the section.
Component CSS files start with the block class as scope: `.trust-bar`, `.trust-bar__icon`,
`.trust-bar__text`.

### Dawn variable override pattern

Dawn references colors via `rgba(var(--color-foreground), 1)`. The override block in
brand-tokens.css redefines `--color-foreground` to Noctorya values:

```css
:root, .color-scheme-1 {
  --color-foreground: 240, 240, 240; /* #F0F0F0 */
}
```

NEVER override Dawn variables outside brand-tokens.css. NEVER hard-code Dawn's
expected RGB-triplet format in component CSS; use Noctorya tokens
(`var(--color-text-primary)`) which already encode the hex value.

### App override pattern

Third-party app CSS lives in `assets/custom-app-overrides.css`. Apps default to light
theme; overrides force dark theme. This is the ONE file where `!important` is allowed
(third-party namespace exception). See Blueprint Part 2 Section 12 for Judge.me,
Picture It AR, Wishlist Hero, Selleasy overrides.

---

## ACCESSIBILITY

### Semantic HTML

Use semantic elements first; add ARIA only when semantics are insufficient:

- `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`
- `<button>` for interactive triggers (NEVER `<div onclick>`)
- `<a>` for navigation (NEVER `<div>` with JS click handler for links)
- `<form>` with labeled inputs (`<label for="id">`)

### WCAG 2.4.7 focus indicators

ALL interactive elements must have visible focus. Both `outline` AND `box-shadow`
required (Windows High Contrast Mode suppresses shadows):

```css
:focus-visible {
  outline: var(--focus-outline-width) solid var(--focus-outline-color);
  outline-offset: var(--focus-outline-offset);
  box-shadow: 0 0 0 var(--focus-ring-width) var(--focus-ring-color);
}
```

NEVER use `outline: none` without immediate replacement. Tokens encode WCAG-compliant values.

### Keyboard navigation

- All interactive elements reachable via Tab.
- Tab order matches visual order (do NOT use `tabindex` to reorder).
- Esc closes modals/drawers.
- Enter and Space activate buttons.
- Arrow keys navigate within a custom component (carousel, accordion) when component owns focus.

### Touch targets

Minimum 48x48 px on mobile interactive elements (link, button, icon button).
Below 48px, add invisible padding via `:before` or padding utility.

### Color contrast

WCAG AA validated combinations documented in `assets/brand-tokens.css` comments and
in `design_system.md` Section 1 WCAG Contrast Verification table. Do not pair tokens
outside the validated set.

### Alt text and labels

- Image alt text: 80-125 chars, factual, includes artist + title + product type.
- Decorative images: `alt=""` (empty, not omitted).
- Form labels: visible `<label>` preferred; `aria-label` only when visible label is impossible.

---

## PERFORMANCE

### Image loading

- Hero: `loading="eager"`, `fetchpriority="high"`.
- Collection grid: first 4 products `loading="eager"`; rest `loading="lazy"`.
- All other images: `loading="lazy"`.
- Use Shopify's `image_url` filter with sized variants; never load 2048px originals where smaller fits.

### Script loading

- `defer` for scripts that need DOM ready but no execution priority.
- `async` for independent third-party scripts (analytics, pixels).
- NO synchronous third-party scripts.
- Always use `{{ 'foo.js' | asset_url | script_tag, defer: true }}` pattern.

### Font preload

In `layout/theme.liquid` `<head>`, preload exactly 2 fonts: `inter-400.woff2` +
`bodoni-moda-500.woff2` via `<link rel="preload" as="font" type="font/woff2" ... crossorigin>`.
Other weights load via `@font-face` with `font-display: swap` (heading) or
`font-display: optional` (body).

### /links page exception

`templates/page.links.json` and its section use system fonts ONLY (no WOFF2 load),
inline CSS in `<style>`, no external stylesheet requests, no analytics on initial load.
Target: <2s LCP on throttled 3G.

### Lighthouse target

Mobile score >= 60 (Master Dispatcher Task 071). Test on throttled 3G via Lighthouse
mobile preset before merging to `live/noctorya`.

---

## VOICE CANON FOR MICROCOPY

All customer-facing text in this theme follows the Noctorya voice canon (anonymous critic writing for Eleanor, the buyer persona)
(`../08_DEEP_RESEARCH/folder2_strategy/voice_canon.md`).

### High-level rules

- Cold, factual, manifesto-like. Gallery curator authority.
- NO marketing buzzwords (haunting, captivating, breathtaking, stunning, iconic, masterpiece).
  See voice_canon.md for the full banned list.
- ALL CAPS for headings and CTAs only; never for emphasis in body text.
- Title format: `[Artist Last Name]: [Painting Title]`.
- URL slug format: `artist-lastname-painting-name`.

### Pronoun matrix for theme microcopy

- Body copy (descriptions, paragraphs, longform): no first/second person, no we/our.
- CTAs and button labels: second-person allowed ("SHOP NOW", "ADD TO CART").
- Headings: second-person allowed for brand-voice slogans ("YOUR WALLS DESERVE MORE",
  "THE ART YOU LIVE WITH").
- Error/empty states: second-person allowed but kept terse.

When in doubt about microcopy, consult `voice_canon.md` Section "Banned vocabulary"
and "Signature traits".

---

## WORKFLOW

### Plan-before-code (mandatory for multi-file changes)

Before writing any code that touches more than one file:

1. State a plan: list the files you will modify, the changes per file, and any risks.
   Reference Blueprint Part 2 section numbers.
2. Wait for Eduard to approve or adjust.
3. Only after approval: execute the plan.

This prevents context drift across files and missed design intent.

### Schema-first approach

When building a new section:

1. Draft the schema JSON first (settings + blocks + presets).
2. Have Eduard verify the editable surface matches the Customizer need.
3. Then write the Liquid markup that consumes those settings.
4. Then write the CSS file scoped to the block class.
5. Then add the section to the home page JSON template.

### Mobile-first testing

After every significant change, verify:

1. 375px width layout renders cleanly (mobile).
2. 750px width layout renders cleanly (tablet).
3. 1280px width layout renders cleanly (desktop).
4. Dark theme colors applied (no light backgrounds visible).
5. Focus indicators visible on all interactive elements (keyboard nav test).
6. Bodoni Moda hairlines render cleanly on Windows at 28-36px (worst case for thin serifs).
7. Console error free.

Test devices per Blueprint Part 2 Section 15: Windows 1080p laptop (Chrome+Edge),
iPhone Safari, Android mid-range Chrome, macOS Safari, Firefox macOS.

### Liquid validation

Run `shopify theme check` before committing. Fix all errors and warnings before merging.

### Context drift recovery

If Claude Code starts violating rules from this file (wrong colors, light backgrounds,
Google Fonts CDN), run `/memory` to inspect loaded context, then manually correct or
re-reference this file.

---

### `settings_data.json` dual-write protocol

`config/settings_data.json` is touched by TWO write surfaces:

1. Claude Code (this repo's git lane) — modifies via `Write` tool
2. Shopify Theme Editor (Eduard-side, dev store admin UI) — modifies server-side, syncs back to repo

Race condition pattern: if both surfaces write between syncs, the second write silently overwrites the first.

Required hygiene before ANY commit touching settings_data.json:

1. `git pull --rebase` to fetch latest Shopify-side writes
2. Verify settings_data.json in working tree is current
3. Edit + commit + push immediately

If a Theme Editor edit landed between your `git pull` and `git push`, re-pull and reconcile manually. Source: Chiarivista AI cross-audit feedback 2026-05-23.

## ACCEPTANCE CHECKLIST PER SECTION TYPE

### Custom section (Tasks 053-062)

- [ ] sections/[name].liquid created with full schema (settings + blocks + presets)
- [ ] assets/[name].css scoped to block class, references brand-tokens.css only, BEM naming
- [ ] Mobile-first @media (default mobile, then 750px, then 990px)
- [ ] Accessibility: semantic HTML, focus-visible, touch targets 48px+ mobile
- [ ] Performance: lazy-load non-critical images, defer/async scripts
- [ ] Voice canon: customer-facing strings pass voice_canon.md test
- [ ] Tested at 375 / 750 / 1280px widths, console error free
- [ ] PROGRESS SNAPSHOT included at end of session report (per Global Instructions Task State Awareness)

### Dawn-modified section (Tasks 063-067)

- [ ] Original Dawn file preserved via git (rollback possible)
- [ ] Modifications scoped to documented surface
- [ ] CSS overrides in component-scoped file (not in base.css)
- [ ] All Noctorya tokens used; zero hard-coded values
- [ ] Tested in Shopify theme editor at 375 / 750 / 1280px; Customizer still functional
- [ ] PROGRESS SNAPSHOT included at end of session report

### App integration CSS

- [ ] CSS lives in assets/custom-app-overrides.css (the ONE !important-allowed file)
- [ ] Overrides scoped to app-specific selectors (e.g., .jdgm-*, .picture-it-*)
- [ ] References brand-tokens via var() where applicable
- [ ] Tested with app installed in dev store
- [ ] Pandectes does not block the app's required scripts
- [ ] PROGRESS SNAPSHOT included at end of session report

### Snippet (snippets/)

- [ ] snippets/[name].liquid created
- [ ] Parameters passed explicitly via {% render 'name', param: value %}
- [ ] No hard-coded values; uses caller-passed parameters
- [ ] Documented at top of file: parameters expected and their types
- [ ] PROGRESS SNAPSHOT included at end of session report

### Template (templates/)

- [ ] templates/[name].json created (Dawn uses JSON templates)
- [ ] References sections by name (not inline section content)
- [ ] Customizer-friendly section ordering
- [ ] Tested in theme editor
- [ ] PROGRESS SNAPSHOT included at end of session report

---

## NEVER DO IN THIS REPO

- NEVER use Bodoni Moda for body text. Headings >= 24px at weight 500+ only.
- NEVER use light backgrounds. Noctorya is dark from the ground up.
- NEVER use Google Fonts CDN. Self-hosted WOFF2 only.
- NEVER use jQuery. Dawn is vanilla JS.
- NEVER use SCSS, LESS, or any CSS preprocessor. Pure CSS only.
- NEVER use localStorage or sessionStorage in Liquid context (unsupported in Shopify).
- NEVER use Google Tag Manager. Use native Google/Meta/TikTok channels.
- NEVER use emoji in code, comments, or filenames.
- NEVER use !important outside assets/custom-app-overrides.css.
- NEVER invent color, spacing, or typography values. Reference brand-tokens.css.
- NEVER use em dashes in customer-facing microcopy. Use period, comma, or colon.
- NEVER use dropdowns for product variants. Pills with dimensions AND price only.
- NEVER pre-select a size on variant selectors.
- NEVER skip WCAG focus indicators to save time.
- NEVER enable cart drawer on mobile <990px. Cart drawer is desktop-only;
  mobile redirects to /cart page via JS.
- NEVER modify Dawn core files unless the task explicitly says "modify Dawn X".
- NEVER use {% include %} (deprecated). Use {% render %}.
- NEVER add external CDN dependencies without checking the Blueprint app stack.
- NEVER add files outside the theme directory unless the task explicitly requests.
- NEVER use Title Case or ALL CAPS for code identifiers (file names, classes, IDs).
- NEVER hard-code RGB-triplet values for Dawn variables; let brand-tokens.css define them.

---

## WHEN STUCK

- Reference the specific Blueprint Part 2 section number in your question to Eduard.
- Check `assets/brand-tokens.css` for available tokens before requesting new values.
- Check `../08_DEEP_RESEARCH/folder3_design/component_specs.md` for component-level rules.
- If a Master Task Dispatcher task is ambiguous, ask before guessing.
- If voice_canon.md conflicts with this file: voice_canon wins for microcopy; this
  file wins for code structure. Report the conflict so this file can be updated.

---

End of theme repo rules.
