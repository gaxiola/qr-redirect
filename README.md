# QR Redirect

A tiny GitHub Pages site that turns short IDs into redirects, so QR codes you
print into scrapbooks keep working even when you move the underlying video.

## How it works

```
[QR code]  →  https://you.github.io/qr-redirect/?id=birthday-2024
                              │
                              ▼
                       reads links.json
                              │
                              ▼
              https://drive.google.com/...  (or any URL — LAN, NAS, etc.)
```

The QR encodes a stable URL on your GitHub Pages site. The actual destination
lives in `links.json` and you can change it any time without re-printing.

## One-time setup

1. **Create a public repo** on your personal GitHub (e.g. `qr-redirect`).
2. Upload these four files: `index.html`, `admin.html`, `links.json`, `README.md`.
3. In the repo → **Settings → Pages**, set:
   - Source: **Deploy from a branch**
   - Branch: **main**, folder: **/ (root)**
4. Wait ~1 minute. Your site will be at:
   `https://<your-username>.github.io/qr-redirect/`
5. Open `…/admin.html`, paste your base URL and repo URL once. They're stored
   in your browser only.

## Adding or updating a link

1. Open `…/admin.html` and click **Edit links.json on GitHub**
   (or browse to `links.json` in the repo and click the pencil icon).
2. Add or change an entry:
   ```json
   {
     "birthday-2024": "https://drive.google.com/file/d/ABC123/view",
     "first-steps": "http://192.168.1.50:8080/videos/first-steps.mp4"
   }
   ```
3. Commit. The site picks up the change within a minute.

**Tip:** keep IDs short and sortable (`birthday-2024`, `xmas-2023-tree`) so
they're easy to type and read.

## Generating a QR

1. Open `…/admin.html`.
2. Find the link in the list → **Download QR PNG**.
3. Print it into your scrapbook.

## Notes

- **LAN URLs** (`http://192.168.x.x/...`) only resolve when the phone scanning
  the QR is on that LAN. The redirect itself still works from anywhere —
  only the *final hop* is LAN-restricted.
- **Google Drive direct links:** for videos, use the standard share URL
  (`https://drive.google.com/file/d/FILE_ID/view`). Make sure the file is set
  to "Anyone with the link can view."
- `links.json` is public. Don't put anything secret in either the ID or the
  target URL — anyone who guesses an ID can follow it.
- The page does a 250 ms pause before redirecting so QR scanner apps have time
  to hand off cleanly to the browser. Tweak in `index.html` if you want.
