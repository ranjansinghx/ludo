# Ludo

A single-file, mobile-first Ludo game for the browser. Play pass-and-play on one device, or create/join a room to play online with friends in real time.

No build step, no dependencies to install — the app is `index.html` plus a small manifest/service worker/icons for installability, all servable from any static file host.

## Features

- **Classic Ludo rules** — four colors (red, green, yellow, blue), capturing, safe squares, and the standard "roll a 6 for an extra turn" rule (three sixes in a row forfeits the turn by default — toggleable, see House rules below).
- **Local mode** — 2–4 players pass the phone around on a single device.
- **Online mode** — create a room, share a short join code, and play live with friends on their own devices. Game state syncs automatically between all players.
- **In-game chat** — a lightweight text chat panel for online rooms, with an unread badge, alongside the existing emoji reactions.
- **AFK handling** — an online turn left idle too long is automatically skipped (by the host, or by any other seated player if the host itself has gone quiet); after two skips in a row, that player's pieces are handed to the built-in AI so the game can keep going without them.
- **House rules** — toggle whether three 6s in a row forfeits the turn, set per-game in local setup or per-room by the host when creating an online room.
- **Polished mobile UI** — animated dice rolls, piece movement, captures, and a full-screen app-like layout with safe-area support for notched phones.
- **Installable (PWA)** — "Add to Home Screen" on phones, or install from the browser's install icon on desktop. Runs full-screen like a native app, and local (pass-and-play) mode keeps working offline once installed.
- **Zero installation** — pure HTML/CSS/JS, works straight from a static file host.

## Getting Started

### Play locally
Just open `index.html` in a browser, or serve the folder with any static file server:

```bash
npx serve .
# or
python3 -m http.server 8000
```

Then visit the page and tap **Play on this device** for pass-and-play, or **Play online with friends** for a synced multiplayer game.

### Online multiplayer setup
Online mode syncs game state through a small [Firebase Realtime Database](https://firebase.google.com/docs/database) instance. The app ships pointing at a demo database for convenience — for your own deployment, you should:

1. Create a free Firebase project and enable **Realtime Database**.
2. Set database rules so reads/writes are scoped to a `/ludo_rooms/{code}` path (avoid open read/write on the whole database). At minimum, use something like:

   ```json
   {
     "rules": {
       "ludo_rooms": {
         "$code": {
           ".read": true,
           ".write": true,
           ".validate": "newData.hasChildren(['code','seats','status','version','updatedAt'])"
         }
       },
       "$other": {
         ".read": false,
         ".write": false
       }
     }
   }
   ```

   This still lets anyone who knows (or guesses) a 5‑character room code read/write that one room — there's no login system — but it stops them from reading or wiping any *other* room or any other data in your project. Rooms are meant to be short-lived and low-stakes (a dice game), not a place to store anything sensitive.
3. Replace the `FIREBASE_BASE` constant near the top of the script section in `index.html` with your database's URL.

No API keys or server code are required — the client talks to the Realtime Database's REST API directly over `fetch`.

## Security notes

- **Use your own Firebase project for real deployments.** The `FIREBASE_BASE` this repo ships with points at a shared demo database — fine for trying the app out, but don't rely on it for real games since anyone can point their own copy of this page at it too.
- **There is no server-side move validation.** Any client connected to a room can, in principle, write arbitrary game state (this is the trade-off of a serverless, client-synced design). Don't use this for anything beyond a casual game with people you trust.
- **Player names are escaped before rendering**, so a malicious player name can't inject HTML/JS into other players' screens. Chat messages go through the same escaping.
- **Chat and reactions are unmoderated and visible to anyone with the room code** — same trust model as the rest of the room's synced state. Don't rely on this for anything you wouldn't say in front of a stranger who guessed your code.
- If you deploy on a host that supports custom HTTP headers (Netlify, Cloudflare Pages, Vercel, etc.), consider adding a `_headers` file with a stricter Content-Security-Policy, clickjacking protection, and other hardening headers on top of the `<meta>` CSP already in `index.html`. (This project doesn't ship one yet — the `<meta>` CSP is the only protection in place out of the box.) Hosts that ignore `_headers` (e.g. plain GitHub Pages) still get the `<meta>` CSP, just without `frame-ancestors` (browsers only honor that one via the HTTP header).

## How to Play

1. From the home screen, choose **Play on this device** (local) or **Play online with friends**.
2. **Local:** pick 2–4 colors and player names, then tap **Start Game**. Toggle the "Three 6s forfeits turn" house rule on the setup screen if your group plays it differently.
3. **Online:** the host creates a room (picking seats and house rules) and shares the generated room code; other players join using that code from the lobby screen. The host starts the game once everyone's seated.
4. Roll the dice on your turn and tap a movable piece to move it. Land on an opponent to send their piece home, roll a 6 to get an extra turn, and race all four pieces home to win.
5. In online games, use the emoji reactions or the chat panel to talk to the table. If someone goes quiet, their turn is auto-skipped after a short wait, and after two skips in a row the computer takes over their pieces so the game keeps moving.

## Tech Stack

- Vanilla HTML, CSS, and JavaScript — no frameworks or build tools.
- Google Fonts (Baloo 2, Inter, Space Mono) loaded via CDN.
- Firebase Realtime Database REST API for online room sync.

## Project Structure

```
.
├── index.html            # entire app: markup, styles, and game logic
├── manifest.json         # PWA metadata (name, icons, display mode)
├── sw.js                 # service worker — caches the app shell for install/offline
├── icons/
│   ├── icon-192.png
│   ├── icon-512.png
│   └── icon-maskable-512.png
└── README.md
```

Note: `index.html` still holds all the markup/styles/logic — the other files just add installability (manifest + service worker + icons). Deploy the whole folder together; the manifest, icons, and service worker all need to sit alongside `index.html` at the same path for install to work. See the Security notes above if you want to add a `_headers` file for stricter HTTP headers on hosts that support them.

## Contributing

Issues and pull requests are welcome. Since the game logic lives in one file, please keep changes scoped and well-commented to keep it easy to review.

## License

No license has been specified for this project yet. Add a `LICENSE` file if you'd like to make the terms explicit for others.
