# Nidhi @ 40 — Wishes, RSVP & Live Wall

A tiny, static microsite to collect birthday wishes, RSVPs, and favorite memories and display them on a live wall. No build step, no dependencies — just HTML/CSS/JS. **Wishes + Live Wall** use a Google Apps Script backend. **RSVP** is handled via **Netlify Forms** at `/rsvp/`.

## What’s here
- `index.html` — Simple form to submit a wish (max 800 chars), name, and consent. Persists the name in `localStorage`. Shows a confetti animation on success.
- `rsvp/` — Netlify-backed RSVP form that collects **family name**, **number of adults**, **number of kids**, plus optional **email** and **phone**; then shows a thank-you page (see routing notes below).
- `livewall.html` — Auto-refreshing grid that fetches consented notes every 4 seconds and renders them for display (e.g., on a projector). Each note includes a local ❤️ like counter and a 🗑️ delete button. Delete buttons appear for notes submitted from the same browser or, if you open the page with `?admin=1`, for every note. Deleted notes stay hidden after refresh. Escapes HTML to prevent XSS.

**Configuration notes**
- Wishes/Live Wall POST/GET to Google Apps Script Web Apps. Update the `ENDPOINT_URL` in those files to point to your own backend.
- The RSVP form submits directly to Netlify (no custom backend). After the first submission, view entries in **Netlify → Forms → rsvp**. You can enable email notifications there.

## Quick start
- Open `index.html` locally to submit a test message (requires your Apps Script endpoint).
- Open `rsvp/` to send yourself a test RSVP.
- Open `livewall.html` to see the live wall update (requires your Apps Script endpoint).

Tip: If you fork or reuse this, update the `ENDPOINT_URL` in the **non-RSVP** files.

## Netlify routing (RSVP)
You can use either a pretty URL folder or a flat file for the thank-you page:

**Option A (pretty URL)**  
- Thank-you page at `/rsvp/thanks/index.html`  
- Form action: `action="/rsvp/thanks/"`  
- Add redirects to ensure trailing slashes resolve:
```toml
[build]
  publish = "."

[[redirects]]
  from = "/rsvp"
  to = "/rsvp/"
  status = 301

[[redirects]]
  from = "/rsvp/thanks"
  to = "/rsvp/thanks/"
  status = 301
