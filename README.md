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
├── netlify.toml            # deploy settings and cache headers
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

---

## Deploying

### GitHub Pages

1. Push to a repository
2. **Settings → Pages → Source: Deploy from a branch**
3. Pick `main` and `/ (root)`, then Save

The site appears at `https://<user>.github.io/<repo>/` within a minute or two. `.nojekyll` is already included so Pages serves the files as-is.

### Netlify (current target)

Connect the repository — `netlify.toml` supplies the settings, so there is no build command and the publish directory is the repo root.

**After the first deploy, set the notification email.** Netlify collects submissions whether or not this is configured, but nothing is forwarded until it is:

1. **Site configuration → Forms → Form notifications**
2. **Add notification → Email notification**
3. Form: `contact`, send to `doubleaxis.3d@gmail.com`

Submissions are also kept under **Forms → contact** in the dashboard, which is the place to check if an email ever goes missing.

The free tier covers 100 submissions per month.

### Vercel / Cloudflare Pages

Both will serve the site, but **the contact form only works on Netlify** — it depends on Netlify Forms. On another host the form needs a different backend (Formspree, Basin, or a serverless function).

---

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

## The contact form

Handled by [Netlify Forms](https://docs.netlify.com/forms/setup/) — no backend to run. Netlify detects the form in the deployed HTML at build time, which is why `name="contact"`, `data-netlify="true"` and the hidden `form-name` field all have to be present in the markup as shipped rather than injected by script.

Submissions arrive at **doubleaxis.3d@gmail.com** once the notification is configured (see above).

**How it behaves**

- Name, email, message and the consent checkbox are required. Failing fields are outlined, given an inline message, and the first one takes focus
- Submitting goes over `fetch`, so the visitor stays on the page instead of being sent to a plain success screen
- On success the fields are replaced by a thank-you message
- If the request fails, the button re-enables and a `mailto:` link appears pre-filled with everything they typed, so the message is not lost
- A honeypot field named `company-website` catches naive bots. It is positioned off-screen rather than `display:none`, since some bots skip hidden inputs

**Testing it.** Netlify Forms only runs on a Netlify deploy — the form will not submit from `localhost` or from a Pages deploy. Use a deploy preview to check it end to end.

## Known gaps

See `QA_REPORT.md` for the full audit. Outstanding items:

- No email address, phone number, or social links shown anywhere on the page (the form is the only route in)
- The Work nav item points at the statement section; there are no case studies yet
- The consent text references a Privacy Policy that does not exist
- `<head>` has no description, OG tags, favicon, or canonical URL — links shared to messaging apps show no preview
- Service cards look clickable (pointer cursor, "View →") but do not link anywhere
