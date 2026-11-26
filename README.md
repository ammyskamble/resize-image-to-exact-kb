# Resize Image to Exact KB (Browser Canvas)

A tiny, dependency-free example of the two things upload forms actually demand and most resize code ignores:

1. **Exact pixel dimensions** — 200×230 means 200×230.
2. **A file-size band, not just a ceiling** — many portals reject files that are too *small* as well as too big (a signature might have to be 10–20 KB, a thumb impression 20–50 KB).

Everything runs locally with the Canvas API and `toBlob` — no server, no upload, the file never leaves the device.

## The core trick: binary-search the JPEG quality

JPEG file size rises monotonically with the `toBlob` quality parameter, so landing inside a KB band is a textbook binary search:

```js
async function encodeInBand(canvas, minBytes, maxBytes) {
  let lo = 0.05, hi = 1.0, best = null;
  for (let i = 0; i < 7; i++) {
    const q = (lo + hi) / 2;
    const blob = await new Promise(r => canvas.toBlob(r, 'image/jpeg', q));
    if (blob.size > maxBytes) {
      hi = q;              // too heavy — lower the quality
    } else if (blob.size < minBytes) {
      lo = q;              // too light — raise it, remember this candidate
      best = blob;
    } else {
      return blob;         // inside the band
    }
  }
  return best;             // closest attempt that stays under the ceiling
}
```

## Try it

Open `index.html` in any browser (or serve the folder with any static server). Pick an image, set the pixel size and the KB band, and download the result. Defaults are 560×160 px and 20–50 KB.

One honest limitation: a mostly-white image on a tiny canvas has a natural size ceiling — at 140×60 px even quality 1.0 may only produce a few KB, because there is very little information to encode. No re-encode of the same pixels can beat that; it is why "file too small" rejections exist.

## In production

This pipeline powers the photo and signature presets in [SignResize](https://signresize.in) — a free tool that resizes images to the exact pixel and KB specs exam and job portals require, 100% in the browser.

## License

MIT
