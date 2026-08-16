# The Spy Who Came in from the Park — Digital Programme (website)

A static website built from the 15-page PDF programme. Every page is the **original artwork,
unaltered** — the site adds navigation, zoom, and a text view on top of it.

```
spy-site/
├── index.html     the whole site (no build step, no dependencies)
├── content.js     transcribed page text + audio setup
├── audio/
│   ├── song02-outside-the-light.m4a   AAC 96k · 2.1 MB · used by default
│   └── song02-outside-the-light.mp3   MP3 128k · 2.8 MB · fallback
└── img/           45 files — 15 pages × 3 versions
    ├── pNN.webp     1080px  · what you see by default
    ├── pNN.jpg      1080px  · fallback for very old browsers
    └── pNN@2x.webp  1654px  · loaded only when you tap to enlarge
```

## What it does

- **Swipe / arrow keys** to turn pages, exactly like the PDF
- **Every page is shown whole** — scaled to fit the screen, never cropped, nothing laid on top
- **Tap any page** to open it full-screen; tap again to zoom to 230%, pinch to scale further
- **Audio on pages 11, 12, 13, 14** — Song 2, Song 7 (across two pages) and Song 8. Each page image is cut in two at a blank
  strip below the header band (y = 298 of 2339) and the player bar is set into the seam,
  styled in the same cream paper with stitched dashes. It covers no words, and tapping play
  changes nothing about the layout. Nothing downloads until play is pressed
  (`preload="none"`), so the audio is free for visitors who never press it.
- **Contents menu** (☰) jumps straight to Cast, Song list, individual songs, etc.
  Pages with a demo are marked ♪ in the menu and gold in the page dots.
- **Deep links**: `…/#p11` opens page 11 directly
- Lazy loading — only the current page ±1 is downloaded, so it opens fast on venue wifi

`content.js` still carries the full transcribed text of every page. Nothing renders it at
the moment, but it is there if you ever want a text/accessibility view back.

## Adding more song demos

Two steps. First cut the page image at a blank horizontal strip and save the two halves
as `pNNa` / `pNNb` (this script picks the cleanest row for you):

```bash
python3 - <<'PY'
from PIL import Image
import numpy as np
PAGE, LO, HI = 12, 0.10, 0.15        # page number, and the band to look for a gap in
src = Image.open(f'/tmp/new/n-{PAGE}.png').convert('RGB'); W,H = src.size
ink = (np.asarray(src.convert('L')) < 190)
CUT = min(range(int(H*LO), int(H*HI)), key=lambda y: ink[y].sum())
print('cut at', CUT, 'ink on that row:', ink[CUT].sum())
for name, part in (('a', src.crop((0,0,W,CUT))), ('b', src.crop((0,CUT,W,H)))):
    r = part.resize((1080, round(part.height*1080/part.width)), Image.LANCZOS)
    r.save(f'img/p{PAGE:02d}{name}.webp','WEBP',quality=68,method=6)
    r.save(f'img/p{PAGE:02d}{name}.jpg','JPEG',quality=78,optimize=True,progressive=True)
    print(f'p{PAGE:02d}{name}', part.size, '->', r.size)
PY
```

Then add `split` + `audio` to that page in `content.js`:

```js
{ n:12, title:"7# Wiping the Floor", nav:"…",
  split:{ a:"p12a", b:"p12b" },
  audio:{
    label:"Song 7 — Wiping the Floor",
    sub:"Demo",
    m4a:"audio/song07.m4a",
    mp3:"audio/song07.mp3"
  } }
```

Also update the two `mkImg(p.split.a, 1080, …)` heights in `index.html` to the printed
slice sizes, so the browser reserves the right space. Converting a WAV:

```bash
ffmpeg -i song.wav -c:a aac  -b:a  96k -movflags +faststart audio/song07.m4a
ffmpeg -i song.wav -c:a libmp3lame -b:a 128k              audio/song07.mp3
```

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
