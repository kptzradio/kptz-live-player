# KPTZ Live Player

Pop-out HTML5 audio player for [KPTZ 91.9 FM](https://kptz.org), Port Townsend WA.

Displays live stream audio with real-time now-playing data (track, artist, album art, show name/time) pulled from Spinitron.

---

## Files

| File | Purpose |
|------|---------|
| `player.html` | The pop-out player. Hosted on GitHub Pages. |
| `worker.js` | Cloudflare Worker — CORS proxy for Spinitron API. |
| `wix-embed.html` | Snippet to paste into a Wix HTML iframe embed element. |

---

## Setup

### 1. Deploy the Cloudflare Worker (do this first)

The Spinitron v2 API blocks direct browser requests (no CORS headers), so a server-side proxy is required. A Cloudflare Worker on the free tier handles this cleanly and keeps your API key out of the client code.

1. Log in to [Cloudflare](https://dash.cloudflare.com)
2. Go to **Workers & Pages** → **Create Worker**
3. Name it something like `kptz-spinitron`
4. Replace the default code with the contents of `worker.js`
5. Click **Deploy**
6. Go to **Settings → Variables → Secrets** and add:
   - Name: `SPINITRON_API_KEY`
   - Value: your key from Spinitron Admin → Automation & API
7. Note your Worker URL — it will look like:
   `https://kptz-spinitron.YOUR-SUBDOMAIN.workers.dev`

### 2. Configure player.html

Open `player.html` and find the `CONFIG` block near the top of the `<script>` section:

```javascript
const CONFIG = {
  streamUrl: 'https://kptz.streamguys1.com/live-aac',
  proxyBase: 'https://YOUR-WORKER.workers.dev',   // ← paste your Worker URL here
  pollInterval: 30000,
};
```

Replace `YOUR-WORKER.workers.dev` with your actual Worker URL.

**Logo image:** The `<img>` tag in the header currently points to the logo on your Wix static CDN. If you want a local copy, drop a `logo.png` into this repo and change the `src` to `./logo.png`.

### 3. Deploy to GitHub Pages

1. Create a new GitHub repository (e.g. `kptz-player`)
2. Push `player.html` (and optionally `logo.png`) to the `main` branch
3. Go to **Settings → Pages**
4. Set Source to **Deploy from a branch**, branch `main`, folder `/ (root)`
5. Save — GitHub will give you a URL like:
   `https://YOUR-USERNAME.github.io/kptz-player/`

Your player will be live at:
`https://YOUR-USERNAME.github.io/kptz-player/player.html`

### 4. Add the launch button to Wix

1. Open `wix-embed.html` and update the `playerUrl` variable to match your GitHub Pages URL from step 3
2. In Wix Editor, add **Add Element → Embed → HTML iframe**
3. Paste the entire contents of `wix-embed.html` into the code editor
4. Size the embed element to approximately **160px wide × 48px tall**
5. Publish your Wix site

---

## How it works

```
Visitor clicks "Listen Live" button (Wix embed)
    ↓
window.open() opens player.html in a named pop-out window
    ↓
player.html loads; visitor clicks Play
    ↓
HTML5 <audio> tag connects to StreamGuys stream URL directly
    ↓
JavaScript polls Cloudflare Worker every 30 seconds
    ↓
Worker forwards request to Spinitron API with Bearer auth
    ↓
Spinitron returns current spin JSON
    ↓
Player updates track title, artist, album, art, show info
```

The pop-out window has a fixed name (`kptz-player`), so clicking the
"Listen Live" button again focuses the existing window rather than
opening a second one — the stream keeps playing uninterrupted.

---

## Spinitron API fields used

| Field | Source | Used for |
|-------|--------|---------|
| `spin.song` | `/spins` | Track title |
| `spin.artist` | `/spins` | Artist name |
| `spin.release` | `/spins` | Album name |
| `spin.image` | `/spins` | Album art URL |
| `spin.playlist_id` | `/spins` | Links to playlist for show lookup |
| `playlist.show_id` | `/playlists/:id` | Links to show |
| `playlist.start/end` | `/playlists/:id` | Show air times |
| `show.title` | `/shows/:id` | Program name |

---

## Customisation notes

- **Show name display:** The player conditionally shows the show name and time block only when a `show_id` is present in the playlist. If there's no show attached (automation / freeform), that section stays hidden.
- **Progress bar:** Intentionally omitted — there's nothing to scrub on a live stream.
- **Window size:** Adjust `width` and `height` in `wix-embed.html` to taste.
- **Poll interval:** 30 seconds is Spinitron's recommended minimum. Don't go lower.
