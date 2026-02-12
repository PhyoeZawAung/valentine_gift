# Our Love Story – Valentine Surprise Web App

A small, mobile-first Valentine surprise web app that walks through a love letter, shared memories, photo moments, and a final “Will you be my Valentine?” question.

Built with **HTML5**, **Tailwind CSS (CDN)**, and **vanilla JavaScript**.

---

## Structure

Project root (inside `valentine-app`):

- `index.html` – main single-page app (all screens and layout)
- `css/styles.css` – custom animations and visual polish on top of Tailwind
- `js/app.js` – screen flow, typewriter effect, timeline, gallery, final surprise
- `assets/images/` – your photos go here
- `assets/music/` – your background music file(s)
- `assets/video/` – optional recorded video message
- `package.json` – minimal metadata if you want to deploy with static hosts / tooling

---

## Personalizing the Experience

### 1. Love Letter Text

Open `js/app.js` and edit:

```js
const loveLetterParagraphs = [ /* ... */ ];
```

Write 2–4 short paragraphs that feel like a real letter from you to her. Line breaks (`\n\n`) create new paragraphs.

### 2. Story Timeline

In `js/app.js`, edit `timelineItems`:

- **icon** – small emoji (`♥`, `✨`, `🌸`…)
- **date** – e.g. `"June 15, 2022"` or `"Our first trip"`
- **title** – short label for the moment
- **description** – 1–2 sentimental lines

### 3. Photo Gallery

1. Copy 6–10 of your favorite photos into `assets/images/` and rename them (e.g. `photo-1.jpg`, `photo-paris.jpg`).
2. In `js/app.js`, update `galleryPhotos` with:
   - correct `src` (e.g. `"assets/images/paris.jpg"`)
   - a sweet `caption`
   - a short `alt` description

Tapping a photo opens a **full-screen modal** with the caption.

### 4. Background Music

1. Place an MP3 in `assets/music/`, for example `song.mp3`.
2. In `index.html`, update:

```html
<audio id="bg-music" src="assets/music/romantic-track.mp3" ...></audio>
```

Change the `src` to your file name.

Your girlfriend can toggle music with the **Music On/Off** pill in the top-right corner.

### 5. Optional Video Message

1. Put a short video (`.mp4`) into `assets/video/`, e.g. `message.mp4`.
2. In `index.html`, inside the final question overlay, there is:

```html
<video id="final-video" class="... hidden" src="assets/video/message.mp4" ...></video>
```

- Update the `src` if needed.
- Remove the `hidden` class if you want it visible by default.

### 6. Days Together Counter

In `js/app.js`, set your real date:

```js
const ANNIVERSARY_DATE = new Date("YYYY-MM-DD");
```

This shows the number of days together on the final screen.

---

## Screens & Flow

1. **Welcome**
   - Gradient background, animated envelope, “I made something for you”.
   - Tap anywhere → Envelope animation.
2. **Envelope Opening**
   - 2–3s opening animation with floating hearts.
   - Auto-transition → Love letter.
3. **Love Letter**
   - Typewriter text effect with a blinking cursor.
   - “Continue” button unlocks after text finishes.
4. **Our Story Timeline**
   - Vertical cards with icons, date, title, description.
   - Soft staggered fade-in.
5. **Memory Photo Gallery**
   - Grid of rounded photo cards, subtle hover/press effects.
   - Tapping opens a full-screen modal with caption.
6. **Final Surprise**
   - Centered romantic text + days-together counter.
   - “One last thing…” → final overlay.
   - Final overlay shows “Will you be my Valentine?” with two buttons:
     - **Yes ❤️**
     - **Always yes**
   - Clicking either:
     - Shows a heartfelt message.
     - Triggers a soft **heart-rain** animation (floating hearts).

---

## Running Locally

You can open `index.html` directly in a browser, but using a tiny local server is better (especially for some browsers’ media rules).

From inside `valentine-app`:

```bash
cd valentine-app
npx serve .
```

Or use any static HTTP server you like.

---

## Deployment

This is a plain static site (HTML + CSS + JS), so you can deploy almost anywhere.

### GitHub Pages

1. Create a new GitHub repository.
2. Add the contents of `valentine-app`.
3. Push to the `main` branch.
4. In repo settings → **Pages** → deploy from `/(root)` or `/docs` (depending on your setup).
5. GitHub will give you a URL you can share.

### Netlify

1. Go to Netlify, create a new site from Git.
2. Select your repository.
3. Set the build command to **none** (it’s static) and publish directory to the folder containing `index.html` (e.g. root or `valentine-app`).
4. Netlify gives you a shareable URL (you can customize the subdomain).

---

## Tips for Maximum Emotional Impact

- Replace all placeholder text with **very specific memories**.
- Choose photos that show:
  - her smile
  - one silly moment
  - one soft, quiet moment
  - one “big” memory (trip, date, etc.)
- Test it on your own phone to:
  - check font sizes
  - verify music plays after a user tap
  - make sure images load quickly (compress if needed)

Then just send her the link and let the app do its magic. 💗

