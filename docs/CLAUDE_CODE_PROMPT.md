# Claude Code Prompt Document — Creator Storefront Prototype

Companion to [PRD.md](./PRD.md) and [IMPLEMENTATION_GUIDE.md](./IMPLEMENTATION_GUIDE.md). This document is a **ready-to-paste prompt set** for using Claude Code to build the clickable prototype referenced in the Implementation Guide §4 — the vehicle for diary-study-informed concept testing and swipe-past pressure testing (§5), *not* production Instagram code.

Read this before running anything: the goal is a **research prototype** good enough for moderated usability sessions and stakeholder review, not a shippable feature. Every prompt below is written with that guardrail baked in.

---

## 0. Prerequisites

- Claude Code installed and runnable from this project's directory (`meta-storefront/`).
- Node.js installed (for a Next.js-based prototype — see stack rationale below).
- Access to the **Astryx design system** (`astryx.atmeta.com`) — this project builds on it directly rather than a generic component kit, per direction to build the product with Meta's shared design system.
- No real commerce credentials, no real catalog API access, no real payment integration — this prototype runs entirely on mock data.

## 1. Recommended Stack (and why)

| Choice | Reasoning |
|---|---|
| **Next.js + TypeScript** | Fast scaffold, file-based routing maps cleanly to "profile → storefront tab → product detail," easy to deploy for remote usability sessions |
| **Astryx (`@astryxdesign/core` + a theme package)** | This *is* how Experience Principle #5 ("Respect the visual system," PRD §7) gets implemented concretely — spacing, typography, color, and primitives (Button, Layout, etc.) come from the shared system instead of being approximated in Tailwind |
| **Astryx CLI (`@astryxdesign/cli`)** | Used to discover real components/tokens/templates as we build (`npx astryx component`, `npx astryx docs tokens`) instead of guessing at what's available |
| **Tailwind or StyleX (only where Astryx doesn't cover)** | Astryx supports both as bridges/overrides for one-off layout needs — use them as a supplement to Astryx primitives, not a replacement for them |
| **Mock JSON "catalog API"** | Stands in for Commerce Engineering's real catalog API (PRD §9.3, Implementation Guide §7) — shaped to match the *questions* posed to Commerce Engineering, so swapping in the real API later is a data-layer change, not a UI rewrite |
| **Framer Motion (or CSS transitions)** | Needed for swipe/scroll dwell instrumentation (§5 below) to feel real, not janky |
| **Mobile-first viewport only** | Instagram is a mobile experience; do not build a desktop-first layout and adapt down |

### 1a. Astryx Setup Reference

These are the exact commands the design system's own getting-started docs specify — use them verbatim in Phase 1 rather than improvising an install:

```bash
npm install @astryxdesign/core @astryxdesign/theme-neutral @astryxdesign/cli
npx astryx init
```

Global CSS import (e.g. `globals.css`):
```css
@import '@astryxdesign/core/reset.css';
@import '@astryxdesign/core/astryx.css';
@import '@astryxdesign/theme-neutral/theme.css';
```

Other themes exist (`theme-butter`, `theme-chocolate`, `theme-gothic`, `theme-matcha`, `theme-stone`, `theme-y2k`) — this doc defaults to **`theme-neutral`** as the safest starting point for an image-forward, content-first surface, but that's a guess, not a confirmed brand match. Flag the theme choice to your design lead early; swapping the theme package later is a one-line change, not a rebuild.

Component usage pattern:
```tsx
import {Button} from '@astryxdesign/core/Button';
import {VStack} from '@astryxdesign/core/Layout';

<VStack gap={2}>
  <Button label="Hello Astryx" onClick={() => alert('Hi!')} />
</VStack>
```

CLI discovery commands (have Claude Code run these *before* hand-building any primitive):
```bash
npx astryx component              # list all available components
npx astryx component Button       # a specific component's props & usage
npx astryx docs                   # list documentation topics
npx astryx docs tokens            # spacing, color, radius tokens
npx astryx template --list        # available page templates (check before scaffolding from scratch)
```

★ Insight ─────────────────────────────────────
Notice the setup asks Claude Code to run `npx astryx component` and `npx astryx template --list` *before* building anything — not after. This ordering matters: an agent (or a person) working from a getting-started page alone tends to reach for familiar primitives (a hand-rolled modal, a Tailwind-styled chip) because it doesn't yet know what the design system already provides. Discovery-before-build turns "does Astryx have a Sheet component?" from a guess into a checked fact, and it's the difference between a prototype that actually exercises the shared design system and one that just imports its theme CSS and otherwise ignores it.
─────────────────────────────────────────────────

---

## 2. How to Use This Document

1. Open Claude Code in the `meta-storefront/` directory.
2. Paste the **Context Primer** (§3) first, in its own message — this gives Claude Code the product context so later prompts don't need to re-explain the brief.
3. Run the **Phase prompts** (§4) in order, one at a time. Each phase assumes the previous one is committed/working — don't batch all phases into one giant prompt; you'll get a shallower result and it's harder to course-correct mid-build.
4. After each phase, actually click through the result in a browser before moving on (per this team's own working norms — verify UI changes live, don't just trust the diff).
5. Use the **Acceptance Checklist** (§5) before treating the prototype as ready for a usability session.

---

## 3. Context Primer (paste this first)

```
I'm prototyping a design concept for an Instagram feature: a persistent, native
"storefront" surface on a creator's profile, replacing today's fragmented
affiliate-link experience (bio links, Story stickers, captions).

This is a RESEARCH PROTOTYPE for moderated usability sessions and stakeholder
review — not production code. Please optimize for:
- Fast iteration and clear structure over production hardening
- Mock data instead of real APIs (no real payments, no real auth, no external
  network calls)
- Mobile viewport only (~390px wide), matching the Instagram app
- Visual language: clean, content-forward, minimal chrome — similar to
  Instagram's existing profile/grid aesthetic (generous whitespace, image-forward
  tiles, restrained typography) — not a generic e-commerce template

IMPORTANT: this product is built on Meta's Astryx design system
(@astryxdesign/core), not a generic component kit. Before building any UI
primitive (button, card, sheet/modal, tabs, chip, badge), run
`npx astryx component` (and `npx astryx component <Name>` for specifics) to
check whether Astryx already provides it — use the Astryx primitive if so.
Only fall back to custom composition (Tailwind/StyleX per Astryx's supported
styling approaches) for things Astryx genuinely doesn't cover, like the
storefront-specific layout arrangements themselves.

The prototype needs to support FOUR interchangeable "storefront home" layout
concepts behind a single toggle, so we can A/B compare them in usability
sessions:
1. Grid — dense image-forward tiles for fast scanning/comparison
2. Editorial — creator-authored narrative groupings with copy (e.g. "My
   skincare routine")
3. Category-filtered — grid with a persistent category chip/filter bar
4. Exploratory/hybrid — a featured "spotlight" hero item above a grid

All four share the SAME product card component and the SAME mock catalog data
— only the arrangement/framing differs. Please confirm you understand this
constraint before we start scaffolding, since it affects how you structure
components (shared ProductCard, swappable StorefrontHome layout wrapper).
```

★ Insight ─────────────────────────────────────
This primer explicitly tells Claude Code to confirm understanding of the "shared card, swappable layout" constraint before scaffolding. That's a deliberate technique for agent-directed prompting: when a structural constraint is easy to violate accidentally (an agent might reach for four separate card designs because that's the more obvious reading of "4 concepts"), stating the constraint *and* asking for a confirmation checkpoint catches the misunderstanding before code exists, rather than after a refactor is needed.
─────────────────────────────────────────────────

---

## 4. Phase Prompts

### Phase 1 — Scaffold

```
Scaffold a new Next.js (App Router) + TypeScript project in this directory.

Install and initialize the Astryx design system:
  npm install @astryxdesign/core @astryxdesign/theme-neutral @astryxdesign/cli
  npx astryx init
Then read whatever agent-facing docs `astryx init` generates before writing
any components — those are the source of truth for conventions, not this
prompt. Import the theme in globals.css:
  @import '@astryxdesign/core/reset.css';
  @import '@astryxdesign/core/astryx.css';
  @import '@astryxdesign/theme-neutral/theme.css';

Before scaffolding page structure, run `npx astryx template --list` to check
whether an existing page template (e.g. a profile or commerce template) is a
better starting point than a blank page.

Set up a mobile-first layout shell (max-width mobile viewport, centered on
larger screens) simulating an Instagram profile page at route
`/profile/[handle]`, using Astryx layout primitives (e.g. VStack/HStack, or
whatever `npx astryx component` shows for layout) rather than raw flexbox
divs.

Include a basic profile header (avatar, handle, follower count, bio text —
all mock data) and a tab bar with three tabs: Posts, Reels, Shop — check
`npx astryx component` for an existing Tabs component before building one.
Posts and Reels can render placeholder grids for now — our focus is the
Shop tab.
```

### Phase 2 — Mock Catalog Data Layer

```
Create a mock catalog data layer at `lib/catalog.ts` shaped to mirror the
real constraints we expect from Commerce Engineering's catalog API:
- Product: id, title, image URL (use placeholder images), price, currency,
  availability ("in_stock" | "low_stock" | "sold_out"), variantSummary
  (optional, e.g. "3 colors"), category, isCuratorSelected (boolean),
  isFeatured (boolean, for the spotlight/hero concept)
- Creator shop config: which product IDs are included, their order, and
  category groupings

Generate ~24 mock products across 3-4 categories (e.g. Beauty, Apparel,
Home) so we have enough density to test the category-filtered concept
meaningfully. Also create one "empty" creator config (zero products) to
support the empty-state work in a later phase.
```

### Phase 3 — Shared Product Card + Detail Sheet

```
Build a single reusable ProductCard component used across all four layout
concepts. It must display: image, title, price, availability state (visually
distinct treatment for sold_out — not just a text label swap), variant
summary if present, and a "Selected by [creator]" curation badge that is
VISUALLY DISTINCT from a generic sponsored-disclosure badge (also render a
small "Paid partnership" badge on ~30% of mock products so we can see both
badges coexist without visual conflict).

Tapping a card opens a bottom sheet — run `npx astryx component` first to
check for an existing Sheet, Drawer, or Modal primitive and use it; only
hand-build the sheet behavior if Astryx has no equivalent. It should show a
larger image, full details, and a single primary CTA button "View on
[merchant]" (use Astryx's Button) that — since this is a prototype — just
logs a `linkout_tapped` event to the console with product id, price, and
source layout concept, rather than navigating anywhere real.
```

### Phase 4 — Storefront Home: Four Layout Concepts

```
Build the Shop tab as a StorefrontHome component that accepts a
`layoutConcept` prop: "grid" | "editorial" | "categoryFiltered" | "spotlight".
Implement all four using the SAME ProductCard from Phase 3:

1. grid: dense responsive grid, 2-3 columns depending on viewport, ordered by
   the creator config's product order
2. editorial: 2-3 named groupings (e.g. "My skincare routine") each rendering
   a horizontal scroll row of cards with a short narrative caption above
3. categoryFiltered: a horizontally scrollable chip/filter bar pinned at the
   top of the grid; selecting a chip filters the grid client-side; HIDE the
   filter bar entirely if the creator config has fewer than 6 products
4. spotlight: one large "featured" card (product where isFeatured is true)
   as a hero above a standard grid for the remaining products

Add a dev-only toggle (floating button or URL query param
`?layout=grid|editorial|categoryFiltered|spotlight`) to switch between
concepts live, since we'll use this for side-by-side usability comparisons.
```

### Phase 5 — Empty States

```
Implement two distinct empty states for the Shop tab, per our PRD:
1. Follower view: if the creator config has zero products AND the viewer is
   not the creator, don't show the Shop tab at all — remove it from the tab
   bar entirely (no dead-end tap target).
2. Creator (self) view: if viewing your own profile (simulate this with a
   `?viewAs=creator` query param) and there are zero products, show the Shop
   tab with an inviting empty state and a "Set up your shop" CTA button
   (non-functional in this prototype — just needs to be visually complete for
   research purposes; add a TODO comment noting Content Design owns final
   copy).
```

### Phase 6 — Swipe-Past Instrumentation

```
Add lightweight analytics instrumentation to support swipe-past pressure
testing (per our Implementation Guide §5):
- Log a `storefront_entry_viewed` event with a timestamp when the Shop tab
  entry point enters the viewport
- Track dwell time between that render and either (a) a tap into the tab, or
  (b) the user scrolling/navigating away — log `storefront_entry_tapped`
  (with dwell_ms) or `storefront_entry_swiped_past` (with dwell_ms) accordingly
- Also instrument `product_card_viewed`, `product_card_tapped`, and
  `filter_applied` per the event table in our PRD §14

For this prototype, just console.log these events with a consistent
`[analytics]` prefix and a JSON payload — no real analytics backend needed.
Make the event names/shapes easy to find (one file, e.g. `lib/analytics.ts`)
since we'll want to swap in real instrumentation later without hunting
through components.
```

### Phase 7 — Polish Pass

```
Do a polish pass focused on what matters for usability testing fidelity:
- Loading skeletons for product cards (simulate a brief mock API delay, e.g.
  300-500ms, so the skeleton is actually visible and testable)
- Basic responsive check at common mobile widths (360px, 390px, 428px)
- Keyboard/screen-reader basics on the product detail sheet (focus trap,
  accessible labels on price/availability) — this is a research prototype,
  not shipping code, but sessions may include accessibility-focused
  participants
- Make sure the layout-concept toggle from Phase 4 doesn't leak into what
  participants see — it should be easy for a moderator to hide/remove before
  a session
```

---

## 5. Guardrails (apply to every phase)

- **Astryx-first** — check `npx astryx component` before hand-building any UI primitive (button, card shell, sheet, tabs, chip, badge, skeleton). Custom composition is for storefront-specific arrangement (the four layout concepts), not for things the design system already provides.
- **No real network calls** — all data is local mock data; if Claude Code suggests wiring up a real API, redirect it back to the mock layer.
- **No real auth/accounts** — the `?viewAs=creator` query param is a deliberate stand-in, not a real auth system.
- **No payment or checkout flow** — link-out is a logged event, not a real navigation.
- **Mobile-first, not responsive-desktop-first** — this prototype exists to be tested on a phone-sized viewport; don't let a "make it responsive" instinct produce a desktop-first layout that happens to shrink.
- **Shared ProductCard across all four concepts** — if a later prompt drifts toward four separate card implementations, stop and re-consolidate; this was called out as a constraint in the Context Primer for a reason (see the Insight in §3).
- **Iterative, but not throwaway** — because this is built on the real Astryx components/theme rather than an approximated design system, treat the component layer with more care than a typical disposable prototype; the mock *data layer* (catalog, analytics backend) is what's expected to be rebuilt for production, not the UI's use of Astryx.

---

## 6. Acceptance Checklist (before running a usability session)

- [ ] All four layout concepts render from the same mock dataset and same ProductCard
- [ ] Layout toggle works and is hideable/removable for session recordings
- [ ] Follower-view empty state fully hides the Shop tab (no dead-end tap)
- [ ] Creator-view empty state shows a complete, on-brand CTA
- [ ] Curation badge and sponsored-disclosure badge are visually distinct and both render correctly when a product has both
- [ ] Console-logged analytics events fire correctly for entry dwell time, card views/taps, and filter usage
- [ ] Category filter bar correctly hides itself below the 6-product threshold
- [ ] Sold-out products are visually distinct, not just text-labeled
- [ ] Verified in-browser at 360px, 390px, and 428px viewport widths
- [ ] No real network/auth/payment calls anywhere in the prototype
- [ ] UI is built on Astryx components/theme (not ad-hoc Tailwind/hand-rolled primitives) wherever an Astryx equivalent exists — spot-check by grepping for raw `<div className=` where a Button/Card/Sheet/Tabs primitive should have been used instead
- [ ] Theme choice (`theme-neutral` by default) has been sanity-checked against the design lead's intent, not left as an unreviewed default

---

## 7. After the Prototype: Handoff Note

Once a concept (or hybrid) is validated per the Implementation Guide's research plan, treat this prototype as a **partial** starting point, not a throwaway: because it's built on real Astryx components and theming rather than an approximated design system, its UI layer is closer to production-viable than a typical research prototype. What still needs replacing for production is the *data layer* — the mock catalog and analytics stub — once the real contract with Commerce Engineering (Implementation Guide §7) is resolved. Don't assume the whole thing gets rewritten; audit component-by-component what can carry forward.
