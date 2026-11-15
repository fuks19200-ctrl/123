# Bloom — homepage

A single-page, scroll-driven landing page. Three background videos are scrubbed by the
scroll position and painted onto a canvas, with glass cards and text fading in over them.

Plain HTML, CSS and JavaScript in one file. No framework, no build step, no dependencies.

## Files

```
index.html              the whole page (markup, styles, script)
favicon.svg
videos/                 1920×1080 background videos (desktop)
videos/mobile/          1280×720 versions, used on screens up to 820px wide
posters/                still frames, shown while the videos load
media/og.jpg            1200×630 preview image for link sharing
```

## Run it locally

Serve the folder over HTTP; opening `index.html` straight from disk is not reliable for video.

```
python3 -m http.server 8000
```

Then open http://localhost:8000 in a browser. Any static server works (`npx serve`, VS Code Live Server, etc.).

## Deploy

Upload the folder as-is to any static host: Vercel, Netlify, Cloudflare Pages, GitHub Pages, S3.

## Before you publish

- **Share preview:** `og:image` and `twitter:image` point to `media/og.jpg`. Social networks
  need an absolute URL, so change both to `https://your-domain/media/og.jpg` once you have a
  domain. You can also add `<link rel="canonical" href="https://your-domain/">`.
- **Menu:** the menu items (Atelier, Collections, Rituals, About, Contact) are placeholders
  with no link. Add an `href` to each `<a>`, in both the desktop and the mobile menu, when
  those pages exist.
- **Email form:** front-end only. It shows a success message but sends nothing. Connect it to
  your mailing-list provider in the submit handler at the bottom of the script.
- **Fonts** load from Google Fonts (Instrument Serif, Manrope, Space Grotesk, JetBrains Mono).

## Replacing the videos

Every scroll step seeks the video, so the videos must have a keyframe on **every** frame.
A normal export has one keyframe every few seconds, and the scroll will stutter badly,
especially on phones. Re-encode any new video like this (ffmpeg):

```
# desktop
ffmpeg -i input.mp4 -an -c:v libx264 -pix_fmt yuv420p -preset slow -crf 26 \
  -g 1 -keyint_min 1 -sc_threshold 0 -movflags +faststart videos/bloom-bg-1.mp4

# mobile
ffmpeg -i input.mp4 -an -vf scale=1280:-2 -c:v libx264 -pix_fmt yuv420p -preset slow -crf 26 \
  -g 1 -keyint_min 1 -sc_threshold 0 -movflags +faststart videos/mobile/bloom-bg-1.mp4
```

Keep the posters in sync: `posters/bg-N.jpg` is a 1280×720 still from the start of video N
(it is painted until the video is ready).

```
ffmpeg -ss 0.2 -i videos/bloom-bg-1.mp4 -frames:v 1 -vf scale=1280:-2 -q:v 3 posters/bg-1.jpg
```

## Tuning

At the top of the script:

- `SCROLL_EASE` (0.08): how softly the video follows the scroll. Lower is smoother and slower.
- `SECTION_1_END` / `SECTION_2_END`: where one video hands over to the next.
- The page is 500 viewport-heights tall (`.scroll-container`), which sets the total scroll length.

The videos have no audio track and play muted and inline, so browser autoplay rules do not
block them. On iOS the page briefly plays and pauses each video to make it drawable.
