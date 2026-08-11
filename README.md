# DOUBLE AXIS

Portfolio site for a visual artist / technical designer. A single static page with a scroll-driven hero, an expanding services accordion, and a contact panel.

No build step, no dependencies — open `index.html` in a browser and it runs.

---

## Structure

```
.
├── index.html              # markup, styles and scripts (67 KB)
├── assets/
│   ├── video/
│   │   └── hero.mp4        # scroll-scrubbed footage, 1440×810 (6.9 MB)
│   ├── fonts/
│   │   ├── anton.woff2
│   │   ├── archivo.woff2
│   │   └── space-grotesk.woff2
│   └── img/
│       ├── hero-poster.jpg
│       ├── statement-bg.webp
│       ├── peek-rabbit.png
│       └── peek-lynx.png
├── .github/workflows/
│   └── pages.yml           # publishes to GitHub Pages on push to main
├── .nojekyll               # keeps GitHub Pages from processing the files
└── LICENSE
```

---

## Running locally

This build loads its assets as separate files, so opening `index.html` straight from disk will not work — Chrome refuses cross-origin requests from `file://`, and the fonts and video come back empty. Serve the folder over HTTP instead:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Any static server works equally well (`npx serve`, `php -S localhost:8000`, etc.).

If you just want to look at the page without a server, use `standalone.html` from the same release — everything is embedded in that one file and it opens by double-clicking. It is for previewing only; deploy the split version, which is what makes the page paint in well under a second.

Note that the form cannot submit from `file://` either — most form services reject requests with no origin. Use the local server to test it.

---

## Deploying

### GitHub Pages

1. Push to a repository
2. **Settings → Pages → Source: GitHub Actions**

The workflow in `.github/workflows/pages.yml` publishes the repo as-is on every push to `main`. The site appears at `https://<user>.github.io/<repo>/`.

Deploying from a branch works too (**Source: Deploy from a branch**, `main` / `/ (root)`) — `.nojekyll` is included so Pages serves the files untouched. The workflow is only there to make deploys visible and repeatable.

Every asset path in the page is relative, so it works whether the site sits at a domain root or under a repo subpath.

**On the video.** `hero.mp4` is 6.9 MB. That is well inside the 100 MB per-file and 1 GB per-repo limits, but Pages has a soft 100 GB/month bandwidth guideline — roughly 14,000 full page loads. Fine for a portfolio; worth watching if the site ever gets busy.

## The contact form

GitHub Pages serves static files and nothing else — there is no server to receive a POST. The submission goes to [Web3Forms](https://web3forms.com), which forwards it as email to **doubleaxis.3d@gmail.com**.

This is already wired up; nothing needs configuring to deploy. The settings sit near the top of the contact-form block in `index.html`:

```js
const FORM_ENDPOINT   = "https://api.web3forms.com/submit";
const FORM_ACCESS_KEY = "46a39b88-4656-46b3-b175-d64aba16ac04";
```

The access key is meant to be public — it only authorises sending to the address it was issued for, so there is nothing to leak by shipping it in client-side code. To point the form at a different inbox, request a new key at web3forms.com and swap this one out.

Each submission is sent as JSON with `replyto` set to whoever filled in the form, so replying from the inbox goes straight back to them.

The free tier covers 250 submissions a month.

**How the form behaves**

- Name, email, message and the consent box are required. Failing fields are outlined and given an inline message, and the first one takes focus
- Submitting goes over `fetch`, so the visitor stays on the page
- On success the fields are replaced by a thank-you note
- If the request fails, the button re-enables and a `mailto:` link appears pre-filled with everything they typed
- If the key is ever cleared out, Send falls back to opening a pre-addressed email rather than becoming a dead button
- A honeypot field named `company-website` is checked before any network call and stripped from the payload

## Notes on the implementation

**Scroll-driven video.** The hero clip is scrubbed by scroll position rather than played. Seeking to an arbitrary frame means decoding from the nearest keyframe, so `hero.mp4` is encoded with a 4-frame GOP — short intervals keep each seek cheap. Re-encoding with a longer GOP will make the scrub stutter badly. The relevant settings:

```
-c:v libx264 -crf 22 -g 4 -keyint_min 4 -sc_threshold 0 -bf 0 -pix_fmt yuv420p -movflags +faststart
```

**Viewport height is pinned in JS.** Mobile browsers change the reported viewport as the URL bar collapses, which would resize the hero's scroll track mid-scroll. The measurement is taken once and only refreshed when the width actually changes.

**The services accordion runs on two axes.** Wide screens expand a column on hover; narrow screens expand a row on tap, with the section height fixed so the page never jumps. The hover rules are scoped to `(hover:hover) and (pointer:fine)` — without that, a tap on a phone re-flowed the stacked cards into four columns.

**Reveal animations fail open.** If scripting is blocked, elements render visible rather than staying masked out.

---

## Browser support

Chrome / Edge 105+, Safari 15.4+, Firefox 121+. The layout depends on `:has()` and `svh` units; older browsers fall back to a static, unanimated version of the same page.

---

## Known gaps

See `QA_REPORT.md` for the full audit. Outstanding items:

- No email address, phone number, or social links shown anywhere on the page (the form is the only route in)
- The Work nav item points at the statement section; there are no case studies yet
- The consent text references a Privacy Policy that does not exist
- `<head>` has no description, OG tags, favicon, or canonical URL — links shared to messaging apps show no preview
- Service cards look clickable (pointer cursor, "View →") but do not link anywhere
