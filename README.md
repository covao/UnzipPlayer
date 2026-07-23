# UnzipPlayer

## Overview

UnzipPlayer is a browser-based media player that extracts and plays audio/video files directly from ZIP archives

## Features

- Single HTML file — no build step, no dependencies to install
- Drag-and-drop ZIP loading with hot-swap replacement
- Encrypted ZIP support (AES-256 / ZipCrypto) with password prompt
- In-memory extraction — files never touch disk
- Built-in ID3 tag parser (v1, v2.2, v2.3, v2.4) with Shift_JIS / UTF-8 / Latin1 auto-detection
- Metadata columns: Title, Artist, Album, Duration
- Folder tree sidebar with recursive playlist view
- Shuffle and repeat playback modes
- Keyboard shortcuts (Space = play/pause, Ctrl+Arrow = prev/next)
- URL parameter loading (`?zip=` or `?file=`)
- Iframe embed mode (`?embed=1`) with a compact 2-row player layout (title / controls) and auto-play of the first track
- Hash-based command API (`#cmd=...`) to control playback from a parent page without reloading the iframe
- Dark theme UI with list and grid views
- Works on `file://` protocol — no local server needed

## Requirements

- A modern web browser (Chrome, Firefox, Edge, Safari)
- No server, no Node.js, no Python — just a browser

## Quick Start

1. Download `unzipplayer.html`
2. Open it in your browser
3. Drag a ZIP file containing `.mp3` or `.mp4` files onto the page

That's it.

## Installation

No installation required. It's a single HTML file.


## Uninstallation

Delete the file.

## Usage

### Basic

Open `unzipplayer.html` in a browser and drop a ZIP file onto the page. Files are extracted into memory and displayed in a file manager UI. Click any file to play it.

### Encrypted ZIP

If the ZIP is password-protected, a dialog will prompt for the password. Enter it and click **Unlock**. If the password is wrong, you can retry.

### URL Parameters

Load media directly via URL query parameters:

```
unzipplayer.html?zip=https://example.com/music.zip
unzipplayer.html?file=https://example.com/song1.mp3&file=https://example.com/song2.mp3
```

> **Note:** The remote server must allow CORS for URL loading to work.

### Iframe Embed Mode

Add `embed=1` (or `embed=true`) to the URL to switch to a compact player suited for embedding in an `<iframe>`. In this mode the header, folder sidebar, and file browser are hidden, and the player bar is rendered as a **2-row layout**: the track title on the first row and the playback controls (shuffle / prev / play-pause / next / repeat / seek bar / volume) on the second row. If a ZIP or file is supplied via `?zip=` / `?file=`, the first track starts playing automatically.

```html
<iframe
  src="unzipplayer.html?embed=1&zip=https://example.com/music.zip"
  width="360" height="140" frameborder="0" allow="autoplay">
</iframe>
```

You can combine `embed=1` with `zip=` or `file=` params, or leave them out and drop a ZIP onto the embedded player manually.

### Hash Command API (no-reload remote control)

Once media is loaded, playback can be controlled from a parent page (e.g. the page hosting the `<iframe>`) by changing the player's URL hash. This does **not** reload the page — it's handled via the `hashchange` event, so it works well for iframe embeds.

| Command | Hash format | Description |
|---|---|---|
| Play | `#cmd=play` | Resume the current track (or start the first track if nothing has played yet) |
| Stop | `#cmd=stop` | Pause playback and reset the current track's position to 0:00 |
| Play at time | `#cmd=playat&path=<file path>&t=<seconds>` | Start the given track and seek to time `t` immediately |
| Fade out & stop | `#cmd=fadeout` | Fade the current track's volume out over 1.5s, then pause and reset to 0:00 |
| Fade in & play | `#cmd=fadein&path=<file path>&t=<seconds>` | Start the given track at time `t` with volume at 0, then fade in to full volume over 1.5s |

`path` is the file's path exactly as stored inside the ZIP (e.g. `Album/Track1.mp3`), URL-encoded. `t` is a time in seconds (decimals allowed, e.g. `42.5`).

Example, from the parent page hosting the iframe:

```js
const player = document.getElementById('playerFrame').contentWindow;
// Play a specific track from 30 seconds in
player.location.hash = '#cmd=playat&path=' + encodeURIComponent('Album/Track1.mp3') + '&t=30';
// Later, fade it out and stop
player.location.hash = '#cmd=fadeout';
```

> **Note:** After each command is processed, the player automatically clears its own hash, so the same command (e.g. `#cmd=stop`) can be sent repeatedly and will still trigger every time.

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Space | Play / Pause |
| Ctrl + → | Next track |
| Ctrl + ← | Previous track |

### Supported Formats

| Type | Extensions |
|------|-----------|
| Audio | `.mp3` `.wav` `.ogg` `.aac` `.flac` `.m4a` `.wma` |
| Video | `.mp4` `.webm` `.mkv` `.avi` `.mov` `.m4v` `.ogv` |
| Archive | `.zip` (standard and encrypted) |

