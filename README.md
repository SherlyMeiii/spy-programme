# The Spy Who Came in from the Park — Digital Programme (website)

A static website built from the 15-page PDF programme. Every page is the **original artwork,
unaltered** — the site adds navigation, zoom, and a text view on top of it.

```
spy-site/
├── index.html     the whole site (no build step, no dependencies)
├── content.js     transcribed page text, used for the "Text" view
└── img/           45 files — 15 pages × 3 versions
    ├── pNN.webp     1080px  · what you see by default
    ├── pNN.jpg      1080px  · fallback for very old browsers
    └── pNN@2x.webp  1654px  · loaded only when you tap to enlarge
```

## What it does

- **Swipe / arrow keys** to turn pages, exactly like the PDF
- **Tap any page** to open it full-screen; tap again to zoom to 230%, pinch to scale further
- **Contents menu** (☰) jumps straight to Cast, Song list, individual songs, etc.
- **"Text" button** re-flows the transcribed text into a single readable column —
  built for the two-column lyric pages, which are hard to read on a phone.
  The cover and the two photo-gallery pages stay as images in either mode.
- **Deep links**: `…/#p11` opens page 11 directly
- Lazy loading — only the current page ±1 is downloaded, so it opens fast on venue wifi

## Preview locally

Double-clicking `index.html` works in most browsers. If images don't appear
(some browsers block local file reads), run a tiny server instead:

```bash
cd spy-site
python3 -m http.server 8000
# then open http://localhost:8000
```

## Publish it

Any static host works — there's no server code.

- **Netlify Drop** — drag the `spy-site` folder onto https://app.netlify.com/drop. Free, instant URL.
- **Cloudflare Pages / Vercel** — same idea, connect a repo or drag the folder.
- **GitHub Pages** — push the folder contents to a repo, enable Pages in Settings.

Once you have the URL, generate a QR code pointing at it for the venue.
A short custom domain (e.g. `spyinthepark.com`) makes the QR simpler and scans faster.

## Editing the text

`content.js` holds the transcription for each page. It's plain HTML inside a JS array —
fix a typo there and it updates the text view. The page images are untouched by this.

## Regenerating images from a new PDF

```bash
pdftoppm -png -r 200 programme.pdf /tmp/hi
python3 - <<'PY'
from PIL import Image; import glob
for i, s in enumerate(sorted(glob.glob('/tmp/hi-*.png')), 1):
    im = Image.open(s).convert('RGB')
    r = im.resize((1080, round(im.height*1080/im.width)), Image.LANCZOS)
    r.save(f'img/p{i:02d}.webp', 'WEBP', quality=68, method=6)
    r.save(f'img/p{i:02d}.jpg', 'JPEG', quality=78, optimize=True, progressive=True)
    im.save(f'img/p{i:02d}@2x.webp', 'WEBP', quality=76, method=6)
PY
```

If the page count changes, update the array in `content.js` to match.
