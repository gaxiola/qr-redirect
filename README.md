# QR Redirect

A tiny static site that turns short IDs into redirects, so QR codes you print
into scrapbooks keep working even when you move the underlying video.

## How it works

```
[QR code]  →  https://your-domain/?id=birthday-2024
                              │
                              ▼
                       reads links.json
                              │
                              ▼
              https://drive.google.com/...  (or any URL — LAN, NAS, etc.)
```

The QR encodes a stable URL on your site. The actual destination lives in
`links.json` and you can change it any time without re-printing.

## Hosting

Works on either:

- **GitHub Pages** (public repo required on the free plan).
- **Cloudflare Pages** (recommended if you want auth) — connect your GitHub
  repo, builds run on every push, and you can put **Cloudflare Access** in
  front so only your family can reach the site.

## One-time setup

1. **Push these four files** to a public repo (`index.html`, `admin.html`, `links.json`, `README.md`).
2. **Enable Pages** (Settings → Pages on GitHub, or connect repo in Cloudflare Pages dashboard).
3. **Open `…/admin.html`** and fill in Settings:
   - **Base URL**: your deployed URL (e.g. `https://qr.example.com/`)
   - **Repo URL**: `https://github.com/yourusername/qr-redirect`
   - **Branch**: usually `main`
   - **GitHub token**: see below

### Creating the GitHub token

The admin page reads/writes `links.json` via the GitHub API. You need a
fine-grained Personal Access Token:

1. <https://github.com/settings/personal-access-tokens/new>
2. **Repository access**: Only select repositories → pick your `qr-redirect` repo
3. **Repository permissions** → **Contents**: Read and write
4. Generate, copy, paste into admin page → Save

The token is stored only in your browser's localStorage. Behind Cloudflare
Access, that browser is already gated to authenticated family members.

## Daily use

Open `…/admin.html`:

- **Add link**: type an ID (or click *Generate random* for an opaque one),
  paste the target URL, click *Add link*. Commits to GitHub automatically.
- **Edit link**: click *Edit* on any row, change ID and/or target, *Save*.
- **Delete link**: *Delete* on any row → confirm.
- **Generate QR**: each row has *Download QR* (PNG, print-ready).

After any change, GitHub Pages redeploys in ~1 min; Cloudflare Pages in ~30 s.
The admin page itself always reads via the GitHub API, so you see your changes
immediately — only the *redirect page* lags by the deploy time.

## Encryption (default)

`links.json` is **encrypted at rest** with AES-GCM, key derived via PBKDF2-SHA256
(250 000 iterations). Anyone fetching `links.json` from your GitHub Pages site
sees only `{v, salt, iv, ct}` — random bytes.

Each generated QR URL includes the passphrase in its fragment:

```
https://you.github.io/qr-redirect/?id=abc123#k=PASSPHRASE
```

URL fragments are never sent to servers, so the passphrase never appears in
GitHub Pages access logs or any CDN. Family members scanning a QR don't need
to type anything — the redirect page reads the fragment, decrypts in the
browser, and forwards them.

### Setup

1. Open the admin page → **Settings → Encryption passphrase**.
2. Click **Generate** to make a strong random passphrase, then **Save**.
3. **Copy the passphrase somewhere safe** (password manager, locked note).
   If you lose it and clear this browser, you can't manage the file or print
   new QRs. Existing printed QRs will still work because the passphrase is
   inside them.

### Threat model

- ✅ Random visitor / bot reads `links.json` → ciphertext, useless.
- ✅ GitHub repo source is public → still just ciphertext.
- ✅ AI crawlers index the site → nothing meaningful to learn.
- ❌ Anyone who has a printed QR can decode the QR image and extract the
  passphrase, which decrypts the whole map. The QR's physical possession is
  effectively the auth.
- ❌ Git history retains any plaintext `links.json` ever committed. The repo
  was emptied to `{}` before encryption, so you're clean — **never commit
  plaintext entries again**.

## Notes

- **LAN URLs** (`http://192.168.x.x/...`) only resolve when the scanning phone
  is on that LAN. The redirect itself works from anywhere — only the final
  hop is LAN-restricted.
- **Google Drive direct links**: use the standard share URL
  (`https://drive.google.com/file/d/FILE_ID/view`) and set the file to
  "Anyone with the link can view."
- **Changing passphrase**: not supported in-place. To rotate, decrypt locally,
  clear the passphrase in admin, set a new one, then re-add each link
  (existing printed QRs will stop working — they encode the old passphrase).
