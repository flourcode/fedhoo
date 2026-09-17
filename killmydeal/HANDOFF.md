# killmydeal.com — handoff notes

For whoever maintains this next: Mark, a developer, or an AI assistant handed
this folder. There is one file. There is no server, no database, no account
system, no API key and no build step anywhere in this project.

## What the site is

A single HTML page holding two things:

1. **The tool.** Five questions about a federal pursuit — customer, money,
   power, path, now — answered YES / SORT OF / NO, scored in the browser, with
   a verdict and the question the seller's manager is most likely to ask.
2. **The content.** An explanation of the five signals, the failure patterns in
   pipeline reviews, a section for managers, Mark's bio and consulting offer,
   and an FAQ. This half exists to be found by search.

The audience is federal sellers and their managers, in the ten minutes before a
pipeline review, usually on a phone.

## The one rule

**Everything lives in `index.html`.** Markup, styles and script are in that one
file on purpose: it can be dropped on any static host, opened from a thumb
drive, or emailed to somebody. If you find yourself adding a build step, ask
whether the thing you are adding is worth losing that.

## How it works

No framework, no dependencies, no network calls at runtime. The only external
request the page makes is to Google Fonts for Roboto.

**Scoring** (`function score()`): each answer maps to a number — YES 92,
SORT OF 50, NO 8 — weighted money 26, power 22, customer 20, path 16, now 16.
Three caps then apply, each of which also prints its reason on screen:

| Condition | Effect |
| --- | --- |
| money = NO | total capped at 55 |
| power = NO | total capped at 60 |
| customer = NO | total capped at 45 |

Thresholds produce the verdict: 75+ HEALTHY, 55–74 STABLE, 35–54 ON LIFE
SUPPORT, under 35 DEAD ON ARRIVAL. The medical metaphor is deliberate and the
set should stay coherent — an earlier draft used GRILL IT for the second tier,
which collided with the GRILL ME button two inches below it.

**Everything else is a lookup table** keyed to the weakest answer: the "your
boss will ask…" line (`ATTACK`), the question in BE READY FOR and the three
Boss Mode questions (`GRILL`), and the actions in DO THIS FIRST (`FIX`). The
ordered list under DO THIS FIRST is every pillar not answered YES, weakest
first, capped at three.

**There is deliberately no AI.** An earlier version sent the five answers to a
language model for coaching. It produced confident, specific, wrong advice —
task orders and recompetes that no one had mentioned — because five words is
not enough context. Confident wrong advice is worse than none here, since the
entire promise is that every line is defensible. If someone revisits this,
that is the bar to clear.

## Privacy, which is a feature and not a footnote

Nothing the user enters is stored or transmitted. No localStorage, no cookies,
no analytics on answers, no server. The tool never asks for the customer, the
agency, the contract number, the partner or the dollar value, which is what
makes it usable by sellers whose employers restrict where pipeline detail can
go. Several lines of copy say so explicitly.

**If you add analytics, a save feature, a CRM integration or an AI call, the
privacy claims in the copy stop being true.** Change the copy in the same
commit or do not make the change. The claims appear in: the line under the
home screen button, the FAQ ("Does anything I enter leave my device?"), and
the site footer.

## Where each kind of change lives

| Change | Where |
| --- | --- |
| Question wording or answer options | `const P = [...]` near the top of the script |
| Weights, caps, thresholds | `function score()` |
| "Your boss will ask…" lines | `ATTACK` inside `score()` |
| Boss Mode questions, BE READY FOR | `const GRILL` |
| DO THIS FIRST actions | `const FIX` |
| Mark's block after a result | `function mountMark()` |
| SEO copy, FAQ, bio, offer | the `<section class="band">` blocks in the HTML |
| Structured data | the `application/ld+json` block in `<head>` |
| Colours, type, spacing | the `:root` tokens at the top of `<style>` |

## Design system

Material 3 (m3.material.io), baseline purple scheme, implemented by hand: colour
roles as CSS custom properties, 16px card corners and fully-rounded buttons,
state layers at M3's hover and press opacities, emphasized easing for motion,
a 3px focus ring at 2px offset, `prefers-reduced-motion` honoured, and a full
dark scheme under `prefers-color-scheme: dark`.

Icons are **inline SVG**, not the Material Symbols font. The font was tried and
removed: when the request fails — common on government and contractor networks
— every icon renders as the literal word `check_circle`. Do not reintroduce the
icon font.

## SEO

- One `<h1>`, rendered as **static HTML**, not by JavaScript. The tool captures
  that markup into `INTRO_HTML` at load and restores it when the user returns
  home, so crawlers and no-JS visitors get real content and there is never a
  second `<h1>`.
- Canonical URL, description, OpenGraph and Twitter tags in `<head>`.
- JSON-LD: `WebApplication`, `Person`, `FAQPage` (six questions, eligible for
  rich results), `HowTo` (the five steps).
- Roughly 1,000 words of substantive content, written for sellers rather than
  for a keyword. Keep it that way; thin SEO filler would undercut the tool.

`robots.txt`, `sitemap.xml` and `card.jpg` (the 1200×630 share image) ship
alongside `index.html`.
The OG card is 1200×630 and was generated by screenshotting an HTML file at that
size, so it matches the app's palette and type. To change it, edit that markup
and re-screenshot rather than hand-editing a PNG. Internal sharing on Teams and
Slack is the main way this spreads between reps, so the card matters more than
it looks.

## Things that will break if you are careless

- **Class name collisions.** `.mark` was already the pillar checkmark style
  when the bio block reused it, which silently forced the block to 40px wide.
  The bio block is `.markblk` for that reason.
- **Images that fail to load.** `mark.jpg` sits beside the HTML. If it is
  missing, its `onerror` adds `.nophoto` so the grid collapses to one column
  instead of squeezing the text into a 120px lane. Keep that handler.
- **Long button labels.** Two CTA labels have already overflowed on a phone.
  The buttons are `white-space: nowrap` with ellipsis and a 160px flex basis,
  but short labels are the real fix.
- **Duplicate script tags.** When this page was assembled from the tool-only
  version, the script was included twice and every `const` collided. There
  should be exactly two `<script>` elements: the JSON-LD block and the app.

## Deploying

Copy `index.html`, `card.jpg`, `robots.txt` and `sitemap.xml` to the web
root. Put `mark.jpg` (square, ideally 240×240 or larger) beside them. That's the
whole deploy. Any static host works — S3 and
CloudFront, Amplify, Netlify, GitHub Pages.

The footer and `window.KMD_BUILD` carry a build stamp. Bump it when you change
the file; it is how you confirm what is actually deployed, which has already
saved one debugging session.

Current build: **2026-09-17.1930**

## Related properties

- **fedhoo.com** — Mark's federal market research tool, built on USASpending
  and SAM data. A trimmed version of Kill My Deal also lives at
  `fedhoo.com/killmydeal/`; if you change the scoring here, change it there or
  retire that copy.
- **Calendly** — `calendly.com/markflournoy/chat-with-mark`, a free 30-minute
  conversation. Every link is UTM-tagged `utm_source=killmydeal` so bookings
  from this page are distinguishable in Calendly.
- **LinkedIn** — `linkedin.com/in/markflournoy`. Deliberately the *secondary*
  outlined button and the Calendly the primary filled one: a stressed rep will
  send a one-line DM long before booking a meeting, so the low-commitment path
  has to be visible. The copy asks for "a sanitized one-liner about your weakest
  pillar," which respects their operational security and gives them something
  specific to type. The link also appears in the static bio CTA row, the site
  footer, and `sameAs` in the structured data.

## Local testing

Open `index.html` in a browser. That's it. Worth checking after any change:

1. Answer all five questions; a verdict appears.
2. GRILL ME asks three questions and reaches a closing verdict.
3. The logo returns you home with answers cleared, and there is still exactly
   one `<h1>` on the page.
4. DevTools → Network shows **no requests** except the font.
5. At 390px wide, nothing overflows horizontally and no button clips its label.
