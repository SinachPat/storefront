# Implementation Guide — Creator Storefront: Shopping Hub Redesign

Companion to [PRD.md](./PRD.md). This document is written for the design team running the day-to-day work, plus the engineering, content design, and data science partners they sync with. It translates the PRD's requirements into a runnable plan: research protocols, prototype specs, component inventory, API contract questions, experiment mechanics, and rollout sequencing.

---

## 1. How to Use This Document

- **Designers** run §3–6 (research → prototyping → component spec) week to week.
- **Commerce Engineering** partners on §7 (API contract) before concept lock — this is a hard dependency, not a parallel track.
- **Content Design** partners on §8 (empty states, disclosure copy).
- **Data Science** partners on §9 (experiment design) starting in parallel with prototyping, not after.
- **IG Commerce Design Manager** is the weekly review gate described in §11.

---

## 2. Architecture Overview (surfaces & services)

```
Creator Profile
   └── Storefront Entry Point (new tab/affordance)
          └── Storefront Home (one of: Grid / Editorial / Category-filtered / Exploratory)
                 ├── Product Card (grid unit, reused across all layout concepts)
                 ├── Category Filter Bar (conditional — hidden below N products)
                 ├── Product Detail Sheet (lightweight, not a full PDP)
                 │      └── Link-Out CTA → Merchant site (existing checkout, unchanged)
                 └── Empty State (creator-view CTA vs. follower-view absence)

Backing services (existing/partner-owned, not built by this team):
   - Catalog API (Commerce Engineering) — product price/availability/variant metadata
   - Creator Shop Config Service — which products a creator has selected, ordering, grouping
   - Affiliate/Commerce Attribution Service — resolves link-out tracking
   - Trust & Safety Policy Service — restricted-category filtering, disclosure requirements
```

★ Insight ─────────────────────────────────────
Note that the Product Card is deliberately drawn as a single reused unit across all four layout concepts rather than four separate card designs. This is a common trap in "prototype 3–4 concepts" work: teams over-differentiate at the card level when the actual variable under test is *arrangement and framing* (grid density vs. editorial narrative vs. filtered navigation), not the atomic unit itself. Keeping the card constant means the A/B/n research signal you get back is cleaner — you're isolating layout as the variable, not confounding it with card redesign.
─────────────────────────────────────────────────

---

## 3. Research Plan — Diary Studies

**Goal:** Map where mid-tier creators (50K–500K followers) currently lose shoppers in today's fragmented affiliate flow (bio link → Stories → captions), before designing the replacement.

**Participant criteria:**
- 50K–500K followers (mid-tier — large enough for real commerce activity, small enough to lack dedicated commerce ops)
- Currently uses at least one third-party link-in-bio tool
- Mix of content categories (beauty, apparel, home, lifestyle) to avoid over-fitting to one vertical
- Target n: 8–12 creators for diary study depth (qualitative, not statistical)

**Protocol outline:**
1. **Screener** — confirm follower range, current monetization tool, posting cadence.
2. **Diary period** (5–7 days) — creators log every instance they post a product recommendation and how (Story, caption, bio link update), plus any follower DMs asking "where's this from?"
3. **Exit interview** — walk through diary entries together; probe specifically for:
   - Moments they *wanted* to point followers somewhere persistent but couldn't
   - Any manual copy/paste or repeated bio-link updates (a proxy for ops burden)
   - Follower confusion signals (DMs, comments asking for the link again)
4. **Funnel synthesis** — produce a drop-off map: content post → follower intent → link discovery → link-out → (unknown: purchase, since that's off-platform today).

**Deliverable:** A funnel drop-off map (artifact, not just notes) that becomes the baseline the V1 concept is measured against.

---

## 4. Prototyping Plan — Layout Concepts

Build to **clickable-prototype fidelity**, not static comps — the swipe-past pressure test in §5 requires real scroll/tap behavior, not a flat mock.

| Concept | Prototype scope | Primary hypothesis under test |
|---|---|---|
| **Grid** | Full scroll, tap-through to product detail sheet | Casual browsers scan faster; high-intent shoppers compare faster |
| **Editorial** | 2–3 authored groupings with narrative copy, scroll + tap-through | Casual browsers dwell longer / trust signal is stronger |
| **Category-filtered** | Grid + working filter chip bar (client-side mock data is fine) | Return visitors complete tasks faster ("find me the skincare item again") |
| **Exploratory (4th concept)** | Define with research input after diary studies — do not lock before §3 synthesis | TBD — likely a hybrid or "featured drop" hero pattern |

**Build vehicle:** See [CLAUDE_CODE_PROMPT.md](./CLAUDE_CODE_PROMPT.md) for a ready-to-use prompt set that scaffolds all four concepts as a single clickable web prototype (mobile-viewport web app), suitable for moderated usability sessions and for the swipe-past instrumentation described below. The prototype is built on **Astryx** (`astryx.atmeta.com`), Meta's shared design system, rather than a generic component kit — see §6a below.

---

## 5. Pressure-Testing Swipe-Past Behavior

**What "swipe-past" means here:** Feed research treats sub-0.5s scroll dwell as a signal the user disengaged rather than evaluated. Applied to a profile surface, this becomes: *does the storefront entry point communicate enough (via thumbnail, product count, or motion) to survive a fast scroll-through of the profile without registering as "just another tab"?*

**Method:**
1. Instrument the prototype (see CLAUDE_CODE_PROMPT.md's analytics stub) to log dwell time on the entry point before a tap or scroll-past.
2. Run moderated sessions where participants are given a *content* task first ("find a recent post about X") — not a shopping task — so the storefront entry point is encountered incidentally, matching real casual-browser behavior.
3. Compare dwell-before-decision across the 3–4 concepts' entry-point treatments (thumbnail preview vs. label-only vs. product-count badge).
4. Flag any concept where >50% of sessions register sub-0.5s dwell *and* zero recall of the entry point in post-session debrief — that's a failing entry-point treatment regardless of how good the storefront home itself is.

**Why this matters more here than in Feed:** a profile is a higher-intent surface than Feed to begin with, so a storefront entry point that still gets swiped past is a stronger negative signal — it's failing on friendlier ground.

---

## 6. Component Inventory (design spec handoff)

| Component | Notes | Astryx-first? |
|---|---|---|
| Storefront tab entry | Must render meaningful content pre-tap (thumbnail/count), not just a static label | Check for a Tabs primitive |
| Storefront home container | Swappable per layout concept; shares scroll/nav chrome with Posts/Reels tabs for visual consistency | Layout primitives (VStack/HStack or equivalent) |
| Product card | Image, title, price, availability state, variant summary (if catalog supports), "Selected by [creator]" signal | Likely custom composition — check for a Card primitive to build on first |
| Category filter/chip bar | Conditionally rendered — hide below a minimum product count threshold (define with research, suggest 6) | Check for Chip/Tag primitive |
| Product detail sheet | Lightweight modal/sheet, not full PDP; single clear link-out CTA | Check for Sheet/Drawer/Modal primitive |
| Empty state (creator view) | CTA to set up shop; only shown to the profile owner | Button primitive; check for an EmptyState pattern |
| Empty state (follower view) | Storefront entry point simply absent — no dead-end tap target | N/A |
| Trust/curation badge | Visually distinct from sponsored-disclosure badge (policy requirement, PRD §11) | Check for Badge/Tag primitive |
| Loading skeleton | Needed given perf budget constraint (PRD §10) — masks catalog API latency | Check for Skeleton primitive |

### 6a. Design System Compliance (Astryx)

This product is built directly on **Astryx** (`astryx.atmeta.com`), Meta's shared design system — this is the concrete implementation of Experience Principle #5 in the PRD ("Respect the visual system"), not just a stylistic preference.

- **Packages:** `@astryxdesign/core`, a theme package (`@astryxdesign/theme-*`), and `@astryxdesign/cli` for discovery.
- **Discovery before build:** run `npx astryx component` (list) and `npx astryx component <Name>` (detail) before hand-building any primitive in the table above; run `npx astryx template --list` before scaffolding a page from scratch.
- **Styling for anything Astryx doesn't cover:** Astryx supports plain CSS, a Tailwind bridge, and StyleX (`xstyle` prop) — use one of these as a supplement for storefront-specific layout arrangement, not as a substitute for an existing Astryx primitive.
- **Theme selection is an open decision, not a default to accept blindly:** seven themes exist (`neutral`, `butter`, `chocolate`, `gothic`, `matcha`, `stone`, `y2k`). The Claude Code prompt document defaults to `theme-neutral` as a safe starting point for prototyping, but the design lead should confirm which theme (if any beyond neutral) is the sanctioned choice for IG Commerce surfaces before this goes past the concept-testing phase.
- **Token reference:** `npx astryx docs tokens` surfaces the canonical spacing/color/radius scale — use this instead of eyeballing values when a layout concept needs a one-off adjustment.

---

## 7. Catalog API Contract — Questions for Commerce Engineering

Resolve these **before** concept lock, since they constrain which layout concepts are even viable:

1. What product metadata fields does the catalog API guarantee (price, availability, variant count/summary) vs. what's optional/creator-dependent?
2. What's the freshness/latency guarantee on availability state — is "sold out" ever stale enough to embarrass a creator?
3. Is there a minimum/maximum product count per creator shop, and does that inform the "hide filter bar below N products" threshold (§6)?
4. Can the API pre-filter restricted categories server-side (policy requirement, PRD §11), or does the client need to filter?
5. What's the attribution mechanism for link-out — does Commerce Engineering already emit a resolvable event, or does this team need to instrument it independently (PRD §14)?

---

## 8. Content Design Collaboration — Empty States & Disclosure

- **Creator-side empty state copy:** invite setup without implying the creator is behind or doing something wrong — this is a common failure mode in "get started" CTAs that this project should explicitly test against (per PRD §6 guardrail on creator setup completion rate).
- **Curation signal copy:** "Selected by [creator]" (or equivalent) must read as distinct from sponsored/affiliate disclosure language — coordinate directly with Policy so Content Design isn't guessing at compliance language.
- **Two empty states, not one:** confirm with Content Design that copy differs meaningfully for (a) creator-viewing-own-profile vs. (b) the surface being absent entirely for followers when no shop exists — these are different moments, not the same string reused.

---

## 9. Experimentation Plan — Small-Market A/B Test

| Parameter | Recommendation |
|---|---|
| **Test cell** | Selected V1 concept (storefront) vs. control (status quo bio-link/Stories flow) |
| **Markets** | 2–3 small markets, chosen with Data Science for existing commerce-instrumentation coverage |
| **Primary metrics** | Browse-to-purchase conversion (PRD §6 primary), link-out drop-off rate |
| **Secondary metrics** | Creator NPS (monetization subset), storefront return-visit rate |
| **Guardrails** | Profile → content engagement rate, time-to-first-post-view (PRD §6 guardrails) |
| **Duration** | Sized by Data Science for statistical power on browse-to-purchase conversion — do not shortcut this; commerce conversion events are typically low-volume relative to content engagement events |
| **Kill-switch criteria** | Any guardrail regression beyond agreed threshold triggers rollback, independent of primary-metric performance |

---

## 10. Rollout Sequencing

1. **Internal alpha** — dogfood on internal/test creator accounts only.
2. **Limited creator cohort** — opt-in mid-tier creators from the diary study pool (they already understand the "why").
3. **Small-market A/B** — per §9.
4. **Readout → V1.1 iteration** — layout tuning, filter threshold tuning, curation-tool refinement based on findings.
5. **Broader rollout** — out of scope for this PRD; requires its own go/no-go review.

---

## 11. Review Cadence & Stakeholder Map

- **Weekly design review** with IG Commerce Design Manager and broader product team — bring prototype state, research findings-to-date, and open decisions; this is a directional-critique forum, not a status readout.
- **Ad hoc syncs** with Commerce Engineering whenever a catalog API assumption changes (§7) — these are blocking dependencies, escalate immediately rather than batching into the weekly review.
- **Content Design pairing** — working session cadence (suggest 2x/week during empty-state copy development) rather than async handoff, given tone is easy to get subtly wrong.

---

## 12. Risks Carried From PRD (engineering/process lens)

| Risk | Implementation-level mitigation |
|---|---|
| Catalog API latency breaks perf budget | Loading skeleton (§6) + confirm freshness SLA (§7) before committing to real-time availability display |
| Swipe-past testing shows entry point fails across all concepts | Treat entry-point treatment as its own variable to iterate, independent of which storefront-home layout wins |
| A/B test underpowered due to low commerce-event volume | Loop in Data Science at prototyping stage (§9), not after MVP is built, to size test duration realistically |
