# MoreChain

Marketing site for **MoreChain**, which builds custom operational systems
(automation, internal tools, process redesign) for owner-run businesses and
prices itself as a share of the verified gain.

Built to the brief in `HANDOFF - morechain.md`. Live at
<https://liav-chanoch.github.io/morechain/>.

> **Copy status.** All screens now carry the approved strings, supplied in
> `FIXES - make chain.md` and copied across verbatim. Screen 02's four step
> bodies are deliberately 76 to 86 characters so the strip reads as equal
> weight; keep any rewrite inside that range. The prototypes themselves
> (`MoreChain Site v5.dc.html`, `MoreChain Mobile.dc.html`) are still not on
> this machine, so they remain the tie-breaker for anything not covered.

## Why it exists

Two audiences land here: an owner who wants to know where their operation is
running below what it could, and a buyer who has to understand a fee model
unusual enough to need explaining before it can be trusted. The site is one
continuous argument, hero through contact, which is why it is a linear card
stack rather than a nav-driven brochure.

## Content rules, non-negotiable

| Rule | Detail |
| --- | --- |
| No em dash | Not in copy, headings, labels, alt text, comments or commit messages. Use a comma, a colon, a period or parentheses. |
| Optimisation, never repair | The client is not broken, they have untapped headroom. Banned: fix, repair, broken, leak, bleeding money, problem, damage, what is wrong. Preferred: headroom, upside, opportunity, gain, capture, unlock, optimise, improve, compounding. Internal code identifiers may use any name. |
| Custom-built throughline | Every screen carries the claim that systems are built from scratch for that one operation. |

## Reference screens

`reference/` in the handoff carries six PNGs captured from the approved
prototype at 924x540. They are the source of truth for **proportion, hierarchy
and spacing**, not for pixel values: every size in the prototype is a `clamp()`
with a `vh` term, so the captures are deliberately short and wide. Match the
ratios, in particular that the text column ends before the chain lane at about
86% of the width, the section label sits alone above a hairline, and the
content block is centred in the space left below it.

## Structure

Six full-viewport cards. Each pins to the top while the next slides over it;
the covered card scales down and darkens.

| # | id | Label | Purpose |
|---|----|-------|---------|
| 0 | `p-0` | (hero) | Positioning and primary CTA |
| 1 | `p-1` | 01 / WHAT WE DO | The service, four offerings |
| 2 | `p-2` | 02 / HOW IT WORKS | Four-step engagement and risk reversal |
| 3 | `p-3` | 03 / WHO WE ARE | Credibility, three pillars |
| 4 | `p-4` | 04 / OUR CLIENTS | Sector chips and six client names |
| 5 | `p-5` | 05 / START WITH THE FREE PART | Contact and footer |

Fixed chrome sits above the cards: logo pill, "Book discovery" pill, the live
readout (UNCAPTURED / CAPTURED / CHAIN plus a status word), step buttons, and a
numbered rail on the right edge that fades in once the hero has been left.

## The chain

A single fixed `<canvas>` drawing 13 record nodes on a hairline spine. Three
nodes (3, 7, 10) start open. Particles travel down the spine and spill sideways
at an open node. A green seal front sweeps the chain; behind it nodes are
sealed and nothing more spills.

| Concern | Rule |
| --- | --- |
| Node states | Untouched: slate stroke, no fill. Open: amber stroke plus a dashed amber connector to the next node. Sealed: green stroke, faint green fill. Previously-open once sealed: mint stroke, mint fill, and a solid mint square centred inside it. That last distinction matters to the client. |
| Glow | A two pass stroke (5px at low alpha, then 1.1px opaque), never `ctx.shadowBlur`. Per-node shadowBlur every frame was a measured performance problem. |
| Hero | Self running. An 8.4s loop: sweep 5.2s on a cubic ease, hold 2s, release 0.7s. Full opacity, vertically inset to clear the CTA above and the step buttons below. |
| Handover | On scroll it cross-fades to a scroll-driven value with a retained baseline, `lerp(auto, RETAIN + (1 - RETAIN) * scrollProgress, dockProgress)`, `RETAIN` 0.30, so it never resets to zero. |
| Docked | Canvas at 0.30 opacity (0.10 on mobile), chain grown to about 1.02 of viewport height, sitting near the right edge so card copy reads over it. |
| Spill | Bounded to about 3.2 node heights and faded over exactly that distance, so particles never rain across the viewport. |
| Reduced motion | One static frame per scroll position, no particles. |

## Interaction

| Concern | Rule |
| --- | --- |
| Scrolling | Wheel and trackpad scroll the page normally. |
| One entry point | Logo, header CTA, both hero CTAs, step buttons and every rail marker call the same `goTo(index)`. |
| `goTo` never reads `offsetTop` | The cards are sticky, so a stuck card reports its stuck offset rather than its layout position, which silently breaks every backward jump. The target comes from stack geometry instead. |
| Tween | Own rAF tween, ease-in-out cubic, `min(900, 260 + distance * 0.35)` ms, guarded by a 110ms `setTimeout` that jumps straight to the target if the first frame has not fired. Native smooth scrolling proved unreliable in embedded views: the easing is a bonus, the landing is guaranteed. |
| Snap | Mandatory scroll snap on mobile stands down for the duration of a programmatic tween, since the two fight each other. |
| Keyboard | PageUp and PageDown move one screen. Home and End jump to the ends. |

## Traps this build already hit

1. **A canvas is a replaced element.** `position:fixed; inset:0` with `width:auto` resolves to its intrinsic (backing store) size, not the viewport, so the chain rendered at DPR times the viewport and landed off screen. It needs explicit `width:100%; height:100%`.
2. **Connectors cannot be grid children of the tile row.** With `repeat(4, minmax(0,1fr))` the three connectors each consumed a column and wrapped the row into two. The row is flex.
3. `min-height:0` on every grid inside a card, because the cards are `overflow:hidden` flex columns and a child grid taller than its space will not shrink.
4. No `backdrop-filter` over the animating canvas. Every canvas repaint re-runs every blur. The chrome uses solid `rgba(7,11,14,0.93)`.
5. No animated `filter: brightness` on full-viewport layers. Each card has an absolutely positioned dark overlay whose `opacity` is animated so it composites.
6. Nothing is measured per frame. References are cached at init, card coverage comes from `scrollY` arithmetic rather than `getBoundingClientRect`, style writes are skipped when the value has not changed, and DPR is capped at 1.5.
7. `minmax(0, 1fr)` everywhere, since `1fr` is `minmax(auto, 1fr)` and one long unbreakable string blows out its track.
8. Client names are `white-space:nowrap`. `dereh-haski.co.il` breaking at its own hyphen misreads the client's name.

## Stack

| Layer | Choice | Why |
| --- | --- | --- |
| Markup, style, script | Single `index.html`, inline CSS and JS | No build step, no dependency surface |
| Fonts | Archivo 400/500/600/700 and IBM Plex Mono 400/500 | Currently loaded from Google Fonts. The brief asks for self-hosting; see Open items. |
| Chain | Inline `<canvas>` 2D | 13 nodes, particles, seal front |
| Icons | Hand-traced vector `M` plus generated PNGs | `assets/favicon.svg` carries the Archivo 700 `M` as an exact polygon, so it needs no font |
| Hosting | GitHub Pages, branch `main`, root `/` | Same pattern that serves liav-chanoch.com |
| SEO | `rel=canonical`, JSON-LD `ProfessionalService`, `sitemap.xml`, `robots.txt` | Staging paths are disallowed in robots and carry `noindex` |

## Run locally

```bash
python3 -m http.server 8000 --directory .
```

Then open <http://localhost:8000>. Check both sides of the 760px breakpoint:
below it the rail and readout change shape and the tile row becomes a 2x2 grid.

## Staging

`/v2/` is a sandbox for trying changes without touching production. It is
tagged `noindex, nofollow` and disallowed in `robots.txt`, and it references
shared assets at `../assets/`, so promoting it later is an HTML swap rather
than an asset migration.

## Environment variables

| Variable | Required | Purpose |
| --- | --- | --- |
| _(none)_ | | The site is fully static with no runtime configuration. |

`.env.example` is committed as the template for anything added later. All
`.env*` variants are gitignored.

## Open items

Waiting on the client, per section 10 of the brief:

- Exact copy for screens 01 to 05, from `MoreChain Site v5.dc.html`
- The three case studies for the parked SELECTED SYSTEMS screen
- A real client quote with name, role and company, plus the two counters
- Logo files for the six clients, to replace the text names
- The real LinkedIn URL (currently a dead anchor)
- Confirmation that `hello@morechain.co` is live

Waiting on a decision here:

- **Self-hosted fonts.** The brief asks for Archivo and IBM Plex Mono to be
  self-hosted. That means downloading woff2 files into `assets/fonts/`, which
  needs an explicit go-ahead. Until then they load from Google Fonts with
  `display=swap`.
