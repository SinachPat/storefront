# PRD — Creator Storefront: Shopping Hub Redesign

| | |
|---|---|
| **Team** | Instagram Creators & Commerce |
| **Tags** | Shopping · Discovery · Native Commerce · Monetization |
| **Status** | Draft v0.1 — for design review |
| **Doc owner** | *[Design lead — fill in]* |
| **Last updated** | 2026-07-17 |
| **Reviewers** | IG Commerce Design Manager, Commerce Engineering, Content Design, Data Science, Trust & Safety/Policy |

**Related docs:** [Implementation Guide](docs/IMPLEMENTATION_GUIDE.md) · [Claude Code Prompt Document](docs/CLAUDE_CODE_PROMPT.md)

---

## 1. TL;DR

Followers currently discover creator-recommended products through a scattered mix of bio links, Story stickers, and caption mentions — a pattern that leaks revenue to third-party link-in-bio tools and erodes buyer trust because there's no consistent way to verify a product is genuinely creator-endorsed. This PRD proposes a **persistent, native storefront surface** living inside a creator's profile that unifies discovery, browsing, and hand-off to purchase, while preserving the editorial feel of the creator's content and meeting Instagram's commerce-safety bar.

This is a **design-and-research-led initiative**: the immediate deliverable is a validated experience (layout concept, information architecture, trust signals, empty states) proven through diary studies and small-market A/B tests — not a full commerce checkout rebuild. Checkout itself continues to hand off to the merchant or Meta's existing commerce checkout, unchanged in this phase.

---

## 2. Problem Statement

**For followers:**
- Product recommendations are ephemeral (Stories expire, captions scroll away) and require leaving the app via an unverified bio link.
- There's no way to tell whether a linked product is a genuine creator pick vs. a generic affiliate redirect, which suppresses purchase confidence.
- High-intent shoppers (arriving specifically to shop) and casual browsers (discovering a product mid-scroll) are served the identical, non-optimized surface — usually a flat list of links.

**For creators:**
- Revenue leaks to third-party link-in-bio tools (Linktree, Beacons, etc.) that sit *outside* Instagram, own the analytics relationship, and take a cut of attention if not revenue.
- Creators have no native curation tools — no way to organize by category, highlight a launch, or retire out-of-stock items — so upkeep is manual and error-prone.
- Monetization features today feel bolted-on rather than a natural extension of a creator's personal brand.

**For Instagram:**
- Commerce activity that happens off-platform is invisible to IG's ranking, ads, and monetization systems — a strategic gap versus platforms with native checkout-to-content integration.

---

## 3. Goals

1. Give followers a **persistent, low-friction browsing surface** for creator-recommended products, discoverable directly from the profile.
2. Make the surface **feel native to the creator's content** — an extension of their voice, not an ad unit — while conforming to IG's visual system.
3. Serve **both casual browsers and high-intent shoppers** from the same surface without forcing either into the other's mental model.
4. Increase **trust and verifiability** of product recommendations (authenticity signals, availability accuracy, transparent pricing).
5. Give creators **lightweight curation controls** with sensible defaults and a graceful empty state for those who haven't set up shop yet.

## 4. Non-Goals (this phase)

- Rebuilding native checkout or payments — the storefront browses and hands off; it does not replace existing checkout infra.
- Creator-side inventory management or fulfillment tooling.
- Cross-creator marketplace / search-wide shopping discovery (this is scoped to a *single creator's* profile surface).
- Live shopping / shoppable video as a first-class format (may be a V2 layout concept, not in initial 3–4 concepts).
- International tax/duty handling beyond what the existing merchant checkout already does.

---

## 5. Users & Personas

| Persona | Description | Primary need |
|---|---|---|
| **Mid-tier creator** (50K–500K followers) | Posts consistently, monetizes via affiliate/brand deals, not big enough for a dedicated commerce team | Native tool that replaces link-in-bio without extra ops burden |
| **Casual browser** | Lands on a creator's profile for content, not shopping; may swipe past a shop tab if it interrupts their scan of the profile | Low-commitment glanceability; easy to ignore or dip into |
| **High-intent shopper** | Arrived specifically to find "that thing they saw in a Reel" | Fast path to the specific product — search/filter, not just a wall of grid tiles |

★ Insight ─────────────────────────────────────
The brief's mention of "0.5s swipe-past behavior patterns from Feed research" is the crux of why this needs two supported browsing modes instead of one. Feed research shorthand for "swipe-past" is usually the threshold at which a user's scroll velocity signals disengagement rather than evaluation — under ~500ms dwell, the content didn't register as worth stopping for. A storefront entry point that fails this test on a *profile* (a much higher-intent surface than Feed) would still get scrolled past by casual browsers, so the design has to win attention within that same window while not overwhelming someone who *is* a high-intent shopper and wants density and filtering immediately after.
─────────────────────────────────────────────────

## 6. Success Metrics

**Primary (North Star candidates):**
- ↑ Creator-linked product purchases per active shopping session (session = any session with a storefront view)
- ↓ Drop-off rate at the link-out step (affiliate/storefront → merchant site)

**Secondary:**
- ↑ Creator NPS specifically on monetization features (tracked via existing creator NPS instrument, filtered to monetization question set)
- ↑ Storefront tab engagement rate (visits to a creator's storefront / profile visits)
- ↑ Return-visit rate to a creator's storefront within 7 days

**Guardrails (must not regress):**
- Profile visit → content engagement rate (storefront must not cannibalize core content consumption)
- Time-to-first-post-view on profile (storefront entry point must not slow down the primary content experience)
- Creator setup completion rate (empty state must not create abandonment)

---

## 7. Experience Principles

1. **Content-first, commerce-second.** The storefront reads as a curated extension of what the creator already posts, not a distinct "store app" bolted onto the profile.
2. **One surface, two speeds.** Casual browsing (scan, swipe, discover) and high-intent shopping (search, filter, compare) are both first-class, not one retrofitted onto the other.
3. **Verifiable by design.** Every product surfaced carries a visible, consistent signal that it's the creator's own curation — not a resolved question the user has to take on faith.
4. **Fail gracefully to zero.** A creator with no shop configured should still produce a coherent, on-brand empty state — never a broken or blank tab.
5. **Respect the visual system.** Typography, spacing, motion, and iconography inherit IG's design tokens; this is a new surface, not a new skin. Concretely, the product is built on **Astryx**, Meta's shared design system (`astryx.atmeta.com`), rather than a one-off component set — see the Implementation Guide §6a and the Claude Code prompt document for how this is enforced at build time.

---

## 8. Scope & Phasing

| Phase | Scope | Exit criteria |
|---|---|---|
| **Discovery** | Diary studies with mid-tier creators; funnel mapping | Documented drop-off map across current affiliate flow |
| **Concept (V0)** | 3–4 prototyped layout concepts (grid, editorial, category-filtered, + 1 exploratory) pressure-tested for swipe-past behavior | One concept (or hybrid) selected with directional research support |
| **MVP (V1)** | Single validated layout, core product card, category filter, empty state, trust badge, link-out flow, baseline instrumentation | Shipped to small-market A/B test |
| **V1.1** | A/B-informed iteration (layout tuning, filter refinement, creator curation controls) | Metrics hit target thresholds (see §6) in ≥1 test market |
| **V2 (future)** | Broader rollout, additional layout personalization, richer catalog metadata (bundles, variants at scale), possible live/video-shoppable format | Out of scope for this PRD |

---

## 9. Functional Requirements

### 9.1 Storefront Entry Point
- New persistent tab/entry point on the creator's profile (alongside Posts/Reels/Tagged), visible only when a creator has commerce access enabled.
- Entry point must render *something* meaningful within the swipe-past attention window — a thumbnail preview or product count, not just a label.
- Entry point is omitted entirely (not shown as disabled/greyed) for creators without commerce access, to avoid dead-end taps.

### 9.2 Storefront Home — Layout Concepts
Prototype and pressure-test at least these concepts:
1. **Grid** — dense, image-forward tiles; optimized for scan speed and high-intent comparison shopping.
2. **Editorial** — creator-authored groupings ("My skincare routine," "What I wore this week") with narrative copy; optimized for casual browsing and brand voice.
3. **Category-filtered** — persistent filter/chip bar (e.g., Beauty, Home, Apparel) atop a grid; optimized for return visitors who shop this creator repeatedly.
4. **Exploratory concept** (to be defined with research input — candidates: "spotlight/featured drop" hero format, or a hybrid editorial-grid).

Each concept must support: product image, price, availability state, and a save/bookmark affordance (if within platform patterns).

### 9.3 Product Card
- Required metadata: image, title, price, availability (in stock / low stock / sold out), and — where the catalog API supports it — variant summary (e.g., "3 colors").
- Explicit **authenticity/curation signal** (e.g., "Selected by [creator]") distinct from generic sponsored-content disclosure.
- Tapping a card opens a lightweight product detail view (not a full PDP rebuild) with a clear CTA to continue to the merchant.

### 9.4 Category & Filter
- Filter bar reflects categories the creator (or catalog metadata) defines; must degrade gracefully when a creator has too few products to warrant filtering (hide the bar rather than show a single-option filter).

### 9.5 Empty State (no shop configured)
- Must be co-designed with Content Design; tone should invite setup without shaming inaction.
- Distinguish two empty states: (a) creator has no commerce access at all — surface is simply absent; (b) creator has access but hasn't configured products — visible CTA to set up, shown only when *viewing your own profile as the creator*, not to followers.

### 9.6 Link-Out & Attribution
- Every outbound tap to a merchant site is instrumented for funnel measurement (see §14).
- Link-out should carry a lightweight interstitial or same-tab transition consistent with existing IG external-link patterns — not introduce a new pattern without Legal/Policy review.

### 9.7 Creator Curation Tools (lightweight, this phase)
- Reorder/group products, hide out-of-stock items, and toggle which layout concept is active (if more than one ships to V1).
- No inventory/fulfillment management — that remains the merchant/catalog system of record.

---

## 10. Non-Functional Requirements

- **Performance:** Storefront tab must reach first meaningful paint fast enough to survive the swipe-past window; treat this as a hard perf budget, not an aspiration.
- **Accessibility:** Product cards, filters, and CTAs meet IG's existing a11y bar (contrast, tap target size, screen-reader labeling of price/availability).
- **Localization:** Price/availability formatting must respect locale from day one, since A/B tests run in multiple small markets.
- **Privacy:** No new PII collection beyond what's needed for funnel instrumentation (§14); link-out events should not leak follower identity to merchant endpoints beyond existing affiliate-link behavior.

---

## 11. Commerce Safety & Policy Requirements

- All products must resolve through the existing verified catalog/merchant pipeline — no freeform/unverified URLs, to preserve the "verifiable by design" principle (§7.3).
- Sponsored/affiliate disclosure requirements (per Meta commerce policy) must be visually distinct from the "Selected by creator" curation signal — these communicate different things and must not be visually conflated.
- Restricted categories (per existing Commerce policy) must be filterable out of the catalog API response before they ever reach the client.

---

## 12. Research Plan (summary — see Implementation Guide for protocol detail)

- Diary studies with mid-tier creators (50K–500K) mapping current funnel drop-off, prior to any prototyping.
- Concept pressure-testing against Feed's swipe-past behavioral benchmark.
- Content Design collaboration on empty-state tone/copy testing.

## 13. Experimentation Plan (summary)

- Small-market A/B test of the selected V1 concept vs. current bio-link status quo.
- Primary readout: browse-to-purchase conversion, link-out drop-off, creator NPS (monetization subset).

## 14. Analytics & Instrumentation (minimum event set)

| Event | Fires when | Key properties |
|---|---|---|
| `storefront_entry_viewed` | Entry point renders on profile | creator_id, has_commerce_access |
| `storefront_entry_tapped` | User taps into storefront | dwell_time_since_render (for swipe-past analysis) |
| `storefront_home_viewed` | Storefront home renders | layout_concept_variant |
| `product_card_viewed` | Card enters viewport | product_id, availability_state |
| `product_card_tapped` | Card tapped | product_id, position_in_layout |
| `filter_applied` | Category filter used | category, result_count |
| `linkout_tapped` | User taps through to merchant | product_id, price, source_surface |
| `linkout_completed` *(if measurable)* | Merchant confirms session landed | product_id |
| `empty_state_viewed` / `empty_state_cta_tapped` | Creator-side empty state shown/acted on | creator_id |

---

## 15. Cross-Functional Dependencies

| Partner | Dependency |
|---|---|
| **Commerce Engineering** | Catalog API contract — confirm what metadata (price, availability, variants) can be surfaced and at what freshness/latency |
| **Content Design** | Empty-state copy, disclosure language, tone of curation signal |
| **Data Science** | A/B test design, sample size/power for small-market tests, NPS instrumentation tie-in |
| **Trust & Safety / Policy** | Sponsored-disclosure compliance, restricted-category filtering, link-out interstitial pattern approval |
| **IG Commerce Design Manager** | Weekly directional critique and go/no-go on concept selection |

## 16. Timeline & Milestones *(illustrative — align with team's actual sprint calendar)*

| Week(s) | Milestone |
|---|---|
| 1–2 | Diary studies complete; funnel drop-off map delivered |
| 3–4 | 3–4 layout concepts prototyped; swipe-past pressure testing |
| 5 | Concept selection; weekly design review sign-off |
| 6–7 | MVP build in partnership with Commerce Engineering; empty-state copy finalized with Content Design |
| 8 | Small-market A/B launch |
| 9–10 | Readout, iterate on V1.1 |

## 17. Risks & Open Questions

| Risk | Mitigation |
|---|---|
| Catalog API can't deliver real-time availability at needed freshness | Confirm constraint with Commerce Engineering *before* concept lock (§9.2 depends on this) |
| Storefront tab cannibalizes content engagement (guardrail regression) | Monitor guardrail metrics from day one of A/B; kill-switch to revert entry point |
| Creators perceive curation tools as extra ops burden, suppressing adoption | Diary-study this explicitly; default to zero-config sensible states |
| Disclosure/curation-signal visual conflict raises policy flags | Loop in Trust & Safety/Policy during concept phase, not after |

**Open questions:**
- Which single layout concept (or hybrid) becomes the V1 default if research is split between casual-browser and high-intent-shopper optimized concepts?
- Does the "Selected by creator" signal require any backend verification, or is it creator-asserted?
- What's the fallback experience on markets/creators without full catalog API coverage?

---

## 18. Appendix — Glossary

- **Link-in-bio tool:** Third-party service (e.g., Linktree) creators use today to aggregate outbound links in their bio, since Instagram historically supports only one bio link.
- **Swipe-past behavior:** Feed-research shorthand for content that fails to hold attention within ~0.5s of scroll dwell — used here as a benchmark for storefront-entry-point legibility.
- **Catalog API:** Commerce Engineering's system of record for product metadata (price, availability, variants) surfaced into the storefront.
- **Link-out:** The moment a user leaves the native storefront surface for a merchant's external checkout.
- **Astryx:** Meta's internal shared design system (`astryx.atmeta.com`) — provides the component primitives, theming, and design tokens this surface is built on, per Experience Principle #5.
