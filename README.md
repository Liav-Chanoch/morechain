# MoreChain

Marketing site for **MoreChain** — a systems consultancy that finds where an
operation is bleeding money, builds the automation/software that stops it, and
prices itself as a share of the savings rather than as an hourly rate.

The commercial model the site has to communicate:

1. **Discovery** — pain points identified, at no cost to the client.
2. **Quantify** — how much money a tailor-made system would actually save.
3. **Price** — the system is priced as a percentage of that saved money.
4. **Retainer** — the client pays monthly, only after the cost efficiency is proven.

> Copy is currently lorem ipsum placeholder. Structure, layout, interaction and
> SEO scaffold are real; wording is not.

## Why it exists

Two audiences land here: an operator who wants to know what breaks in their
business and what it costs, and a buyer who needs to understand a pricing model
that is unusual enough to need explaining before it can be trusted. The site is
one continuous argument from "here is the leak" to "here is why you pay nothing
until it's proven", which is why it is a linear deck rather than a nav-driven
brochure site.

## Architecture

A single-page **card-stack deck**: one `.deck` container holding N
full-viewport `.panel` elements stacked by z-index, plus persistent chrome
rendered *outside* the deck so it survives navigation, plus two independent
overlay layers for case detail.

The parts that are load-bearing and easy to break:

| Concern | Rule |
| --- | --- |
| State | One `current` integer. One `goTo(i)`. One `render()`. No second code path decides what is showing. |
| Wheel | Fires the instant the delta threshold is crossed. Locks afterwards, and **returns before touching `lockUntil`** while locked so events can't extend it. Lock length scales with gesture speed. |
| Touch | Checks the current panel's own scroller for remaining room in that direction *before* treating a swipe as panel navigation. |
| Reduced motion | Same `render()`, same classes; CSS branches transform → opacity. |
| Mobile chrome | Below `860px` the floating dots/arrows/label are replaced by an opaque bottom bar, themed per panel via CSS custom properties set from a JS theme table. |
| Panel height | Content bands are sized off `svh`, never `dvh` — mobile Safari re-expands its toolbar after a scroll settles and `dvh`-sized content gets stranded behind the bar. |
| First paint | Every panel has a **static** CSS default matching its initial JS state, so the highest-z-index panel can't paint on top before the script runs. |
| Overlays | Case list → overlay 1 (summary, slides in) → overlay 2 (narrower, stacked, full detail). Closing 2 returns to 1. |
| Overlay layout | Desktop two columns, **top-aligned** (centering a tall media element against short text reads as a crop). Mobile single column with the media forced between the two text blocks via `display:contents` + `order` — no DOM reordering, desktop untouched. |
| Media | Screenshots get a CSS-drawn frame. Already-chromed sources use `data-frame="plain"` so they aren't double-bezelled. No image at all → the column collapses; no empty placeholder box. |

## Stack

| Layer | Choice | Why |
| --- | --- | --- |
| Markup/style/script | Single `index.html`, inline CSS + JS | No build step, no dependency surface, one file to reason about |
| Fonts | Google Fonts via `<link>` — Bricolage Grotesque, Space Grotesk, JetBrains Mono | Display / body / mono, loaded in one request |
| Graphics | Inline SVG (the chain motif) | Scales, themes with `currentColor`, no binaries |
| Hosting | GitHub Pages, branch `main`, root `/` | Same pattern that serves liav-chanoch.com |
| SEO | `rel=canonical`, JSON-LD `ProfessionalService`, `sitemap.xml`, `robots.txt` | Staging paths are disallowed in robots *and* carry `noindex` |

## Run locally

```bash
python3 -m http.server 8000 --directory .
```

Then open <http://localhost:8000>. Test both sides of the `860px` breakpoint —
the mobile bottom bar and the desktop floating chrome are entirely different
components, so a change to one is not a change to the other.

## Staging

`/v2/` is a sandbox for trying changes without touching production. It is
tagged `noindex, nofollow` and disallowed in `robots.txt`. Any asset it needs is
referenced at `../assets/...` — the same file production uses, never a copy — so
promoting a sandbox later is an HTML swap, not an asset migration.

## Environment variables

| Variable | Required | Purpose |
| --- | --- | --- |
| _(none)_ | — | The site is fully static with no runtime configuration. |

`.env.example` is committed as the template for anything added later (a form
endpoint, an analytics id). All `.env*` variants are gitignored.
