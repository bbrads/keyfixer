# KeyFixer landing pages

Rank-and-rent auto locksmith pages. Single-file HTML, no build step.
Cloudflare Pages deploys `main` on push; `foo.html` serves at `/foo`.

| File | Template | Live path | Region / operator |
|---|---|---|---|
| `index.html` | v1: call + Growform form | `/` | Belfast (master, do not fork casually) |
| `l3.html` | v2: call-only | `/l3` | Kent trial (operator 3) |

URL scheme: operator-coded paths (`/l3`, `/l4`, ...). Never name the
county in the URL; a `/kent` path reads as a national brand to a
local caller. Root stays Belfast.

Numbers: pages carry whatever number Bradley routes. Kent currently
runs the shared 444 number (forwarded operator-side). The click-to-call
conversion label is identical on every page by design; do not mint new
labels per page.

## Stamping a new city (v2, from `l3.html`)

Copy `l3.html` to `lN.html`, then work through the slots. Every
replacement should be made with an assert-guarded script (count the
expected occurrences first; abort on mismatch), not hand edits.

1. **City name** — replace `Kent` everywhere (~17 hits): `<title>`,
   meta description, `og:title`/`og:description`, LocalBusiness schema
   (`description`, `areaServed`), hero pill `Covering: X & Surrounding`,
   H1 `...in X?`, hero bullet `Based in X`, areas `h2`, services `h2`
   `...More Across X`, owner eyebrow + paragraph, FAQ answers
   (`within the hour anywhere in X`, price answer
   `a more competitive price in X`), panel coverage line
   `Wherever you are in X, I come to you.`
2. **Towns** — meta description town list, areas paragraph
   (`I cover A, B, C, ..., and everywhere in between.`), schema
   `areaServed` array. Use real towns from the operator's patch.
3. **Number** — replace the E.164 (`+44...`, covers 10 `tel:` links +
   schema `telephone`), the spaced 38px display number, and the bare
   display strings in button/contact lines. Zero placeholders may
   survive; grep the old number after.
4. **Map** — `images/map-<city>.webp` 644x362 + retina 1288x724,
   `cwebp -q 90` (never compress harder; blurry maps got rejected).
   Update `src`, `srcset`, `width`/`height`, alt text.
5. **Reviews** — from an Apify Google-reviews dataset of the operator:
   - anonymise: locksmith names become "he"; recombine reviewer
     first/last names across the pool so none trace back, keeping
     gender and culture coherent (Polish first + Polish last, etc.);
     business reviewers get a generic trade name.
   - localise: swap any source-town mentions to patch towns.
   - order: price/speed proof in slots 1-4, no two same-genre reviews
     adjacent (e.g. commercial van jobs), nothing off-topic (keys only).
   - dates: fixed ladder labels by position (`1 day ago` ... `8 months
     ago`), never recomputed.
   - counts: badges + schema `reviewCount` come from the dataset's
     `reviewsCount`, not the number of cards shown.
6. **Verify before ads** (browser, not eyeball): every `tel:` href
   carries the new number; clicking each tel element pushes exactly one
   `conversion` event with label `AW-18305418863/j5mgCMmc8NEcEO-M2phE`;
   below-fold images all loaded ~2.5s after load with no scrolling;
   clean mobile render at 390px (panel lines one-per-line, tick block
   flush with the coverage line).

## v1 extras (form pages)

`index.html` adds the Growform embed: client-ID slot in the embed
snippet, cache-warm chain (client script -> bundle -> fonts ->
Cloudinary step images), instant static facade of step 1 whose tap
replays onto the booted form, same-origin resize shim. Fork it only
for operators who work text leads.

## Invariants (both templates)

- **Tracking**: one Google tag (`AW-18305418863`), one delegated
  document-level click listener on `a[href^="tel:"]`, direct
  `gtag('event','conversion')`, **no preventDefault** (iOS never runs
  `event_callback` on tel links). Console logs `[click-to-call]`.
- **Performance**: hero image inlined as a data URI, Fira fonts
  self-hosted in `fonts/`, all third-party (Clarity `xmahcaqgcw`,
  Meta Pixel) behind the deferred gate (first interaction / real
  deviceorientation / 6s idle-quiet / 15s ceiling), below-fold images
  flip lazy->eager 1.2s after load. PSI mobile 100 on clean runs.
- **Copy**: first-person locksmith voice, no em dashes, no exclamation
  marks, Oxford commas, no premium positioning, no hedges ("usually",
  "just in case"), affirmative claims over denials, shop language
  banned ("open now" -> "I'm in the van and ready to take your call").
