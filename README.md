# Nidhi @ 40 — Wishes & Live Wall

A tiny, static microsite to collect birthday wishes and favorite memories and display them on a live wall. No build step, no dependencies — just HTML/CSS/JS plus a Google Apps Script backend.

## What’s here
- `index.html`: Simple form to submit a wish (max 800 chars), name, and consent. Persists the name in `localStorage`. Shows a confetti animation on success.
- `livewall.html`: Auto‑refreshing grid that fetches consented notes every 4 seconds and renders them for display (e.g., on a projector). Each note includes a local ❤️ like counter and a 🗑️ delete button to hide it. Escapes HTML to prevent XSS.

Both files POST/GET to the same `ENDPOINT_URL` (a Google Apps Script Web App).

## Quick start
- Open `index.html` locally to submit a test message.
- Open `livewall.html` to see the live wall update.

Tip: If you fork or reuse this, update the `ENDPOINT_URL` in both files.

## Backend (Google Apps Script)
The site expects a Web App endpoint that:
- Accepts `POST` with body: `{ message: string, name: string, consent: boolean }`
- Returns JSON: `{ ok: true }` (or `{ ok: false }` on error)
- Serves `GET ?mode=list` and returns `{ ok: true, items: Array<{message,name,consent}> }`

This frontend avoids CORS preflight by not setting a `Content-Type` header when posting. Ensure your Apps Script reads raw `text/plain` and parses JSON.

Example Apps Script (Sheet‑backed):
```js
// Create a Google Sheet with header row: message | name | consent | ts
// Set its ID below and deploy this script as a Web App
// (Execute as: Me; Who has access: Anyone with the link)
const SHEET_ID = 'PUT_YOUR_SHEET_ID_HERE';

function doPost(e) {
  try {
    const raw = e && e.postData && e.postData.contents;
    const { message, name, consent } = JSON.parse(raw || '{}');
    if (!message || !name || message.length > 800) return json({ ok: false });
    const sh = SpreadsheetApp.openById(SHEET_ID).getSheets()[0];
    sh.appendRow([message, name, !!consent, new Date()]);
    return json({ ok: true });
  } catch (err) {
    return json({ ok: false });
  }
}

function doGet(e) {
  const mode = (e && e.parameter && e.parameter.mode) || '';
  if (mode !== 'list') return json({ ok: true, items: [] });
  const sh = SpreadsheetApp.openById(SHEET_ID).getSheets()[0];
  const values = sh.getDataRange().getValues();
  const items = values.slice(1).map(r => ({
    message: String(r[0] || ''),
    name: String(r[1] || ''),
    consent: Boolean(r[2]),
    ts: r[3]
  }));
  return json({ ok: true, items });
}

function json(obj) {
  return ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
```

After deploying, copy the Web App URL and set it as `ENDPOINT_URL` in both HTML files.

## Deploy (GitHub Pages)
- Push this repo to GitHub (done).
- In the repo Settings → Pages, set Source to `main` (root).
- Your site will be available at `https://<your-username>.github.io/<repo>/`.
- Open `index.html` to submit and `livewall.html` to display.

## Notes
- Privacy: Only messages with `consent` render on the wall; storage is in your Google Sheet.
- Safety: The wall escapes HTML before rendering; keep doing server‑side validation in Apps Script as needed.
- Customization: Update colors, copy, and limits inline in the two HTML files.
- Likes/deletes are stored in the viewer's browser only; they don't modify the underlying data.

