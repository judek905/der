# Touchline

A live football website with a free-preview video player, past-game replays, a matchday gallery and premium passes paid by Mobile Money. It is plain HTML, CSS and JavaScript, with no build step and no framework.

> **Status: demo.** Payments are not verified by a server yet. See [Known limitations](#known-limitations) before using it with real money.

## Features

- **Live player** with a scorebug (teams, score, clock) and HLS stream support through hls.js
- **Free preview:** non-premium visitors get 2 minutes of live play, then the player locks and asks them to go premium
- **Fixtures rail:** click a match under "On today" to update the scorebug
- **Past games:** replay cards with competition filter chips. Premium replays show a badge and open the payment dialog for non-premium visitors
- **Matchday gallery:** masonry-style grid with generated placeholder art, so no image link can break
- **Premium plans:** Match day pass, Monthly and Season
- **Payment dialog:** 3 steps (choose pass and enter details, send money with a reference, confirmation)
- **Accessible by default:** keyboard focus outlines, ARIA labels, live regions and reduced-motion support
- **Responsive:** the layout stacks on screens narrower than 900px

## Project structure

```
touchline/
├── index.html   # Page structure, payment dialog, hls.js script tag
├── styles.css   # Theme variables, layout, components
└── script.js    # CONFIG, content data, player, fixtures, replays, payments
```

## Getting started

No installation is needed.

1. Keep the three files in the same folder.
2. Open `index.html` in a browser.

For the most reliable video playback, serve the folder locally instead of opening the file directly:

```bash
python -m http.server 8000
```

Then visit `http://localhost:8000`. The VS Code "Live Server" extension also works.

An internet connection is required for the Google Fonts, the hls.js library (loaded from jsDelivr) and the sample videos.

## Configuration

All content lives at the top of `script.js`.

| Setting | What it does |
|---------|--------------|
| `CONFIG.previewSeconds` | Length of the free live preview in seconds (default `120`) |
| `CONFIG.liveSrc` | Your live stream URL, either HLS (`.m3u8`) or `.mp4`. The default is a public test stream |
| `CONFIG.demoUnlock` | When `true`, pressing "I've paid" unlocks premium without checking payment. Set to `false` once a server verifies payments |
| `CONFIG.payNumber` | Payment number (see the note below) |
| `PLANS` | Pass names, prices (in UGX) and perks |
| `FIXTURES` | Matches shown under "On today" |
| `REPLAYS` | Past games: title, competition, date, duration, premium or free, card colour |
| `SAMPLE_VIDEO` | Video played for every replay. Replace with real match videos |
| `GALLERY` | Gallery captions |

**Payment number:** the number `0748756534` is written directly in `index.html` in three places (the fine print under the plans, the footer, and the payment steps). `CONFIG.payNumber` is not used yet, so to change the number, edit `index.html`.

### Using real images

The gallery and replay thumbnails use the `art()` helper, which draws placeholder pitch graphics. To use real photos, replace the `src` in the gallery and replay templates with your own files, for example `src="photos/goal.jpg"`.

### Changing the look

Colours and fonts are variables at the top of `styles.css`:

```css
:root{
  --turf:#0e3b2e; --night:#08201a; --chalk:#f3f1e7; --gold:#ffd23f; --kit:#e4372b;
}
```

## How premium access works

1. The visitor picks a pass and enters their name and a valid Ugandan mobile number (`07XXXXXXXX`, `2567XXXXXXXX` or `+2567XXXXXXXX`).
2. The dialog shows the amount, the payment number and a unique reference (for example `TLA1B2C3`).
3. After sending the money, the visitor presses **I've paid**.
4. The site saves a premium record in `localStorage` under the key `tl_premium`, with an expiry time:

| Plan | Access length |
|------|---------------|
| Match day pass | 1 day |
| Monthly | 30 days |
| Season | 300 days |

Premium visitors skip the preview lock, see no "Premium" badges and can open every replay.

## Known limitations

- **Payments are not verified.** Pressing "I've paid" unlocks premium on trust. Anyone can unlock the site without paying, and anyone can edit `localStorage` in their browser to do the same.
- **The preview resets on refresh.** The 2-minute counter lives in the page, so reloading it gives a fresh preview.
- **Premium is tied to one browser.** The record is stored locally, so it does not follow the user to another device or after clearing site data.
- **Scores and fixtures are static.** They are hard-coded in `script.js` and do not update automatically.
- **Replays all play the same sample video.**

## Going live with real payments

Money sent to a number cannot be confirmed from the browser. To accept real payments:

1. Add a backend that uses a Mobile Money collection API (MTN MoMo or Airtel Money), or an aggregator such as Flutterwave or Pesapal.
2. Confirm each payment by its reference on the server.
3. Issue a signed premium token or session after confirmation, instead of writing to `localStorage` from the browser.
4. Set `CONFIG.demoUnlock = false`.
5. Check the server side as well, so premium streams are only served to verified users.

## Browser support

Works in current versions of Chrome, Edge, Firefox and Safari. The payment dialog uses the native `<dialog>` element. HLS streams play through hls.js, or natively in Safari.

## Dependencies

- [hls.js](https://github.com/video-dev/hls.js) 1.5.15 (CDN)
- Google Fonts: Big Shoulders Display and Figtree
