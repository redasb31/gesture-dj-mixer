# Gesture DJ Mixer (Hand-Controlled Web DJ)

A browser-based DJ mixer controlled entirely with your hands using **MediaPipe Hands** (computer vision) and the **Web Audio API** (audio engine).  
No build tools required: open a local server, allow camera access, and start mixing.

---

## Demo Features

- **5 loop tracks** (Kick, Bass, Hats, Chord, Vocals)
- **Hand-gesture controls** for:
  - Track mute/unmute
  - Track selection
  - Filter (low-pass) control
  - FX send (delay) control
- **Minimal UI** with live meters and selection status
- **Client-side only** (HTML/CSS/JS)

---

## Tech Stack / Libraries

This project is pure front-end and runs fully in the browser.

### Computer Vision (Hand Tracking)
Loaded from CDN:
- `@mediapipe/hands`
- `@mediapipe/camera_utils`
- `@mediapipe/drawing_utils`

### Audio
Built-in Web APIs (no external audio libraries):
- **Web Audio API**: `AudioContext`, `GainNode`, `BiquadFilterNode`, `DelayNode`
- `fetch()` + `decodeAudioData()` for loading and decoding audio loops

### UI
- Plain HTML + CSS + JavaScript (single `index.html`)

---

## Repository Structure

Recommended structure:

```text
gesture-dj-mixer/
├─ index.html
└─ assets/
   ├─ kick.mp3
   ├─ bass.mp3
   ├─ hats.mp3
   ├─ chord.mp3
   └─ vocals.mp3
```

> The application expects the audio loops under `assets/` with **exactly** these filenames.

---

## How to Run Locally

Camera access generally requires **HTTPS** or **localhost**. Do not open the file directly with `file://`.

### Option A: Python (recommended)
From the project folder:

```bash
python3 -m http.server 8000
```

Then open:
- `http://localhost:8000`

### Option B: Node (http-server)
```bash
npm i -g http-server
http-server -p 8000
```

---

## Hand Controls

The app uses **two hands**:

### Left Hand — Toggle Tracks (Mute/Unmute)
Each finger toggles one track (thumb → pinky).  
To toggle, hold the finger open briefly (debounced at ~300ms to avoid accidental triggers).

| Left-hand finger | Track |
|---|---|
| Thumb | KICK |
| Index | BASS |
| Middle | HATS |
| Ring | CHORD |
| Pinky | VOCALS |

Notes:
- Tracks start **muted** (`enabled = false`) until you toggle them on.
- If you struggle with accidental toggles, make sure only the intended finger is clearly extended.

### Right Hand — Select Track + Control Filter
**Track selection** is done by opening **one finger** on the right hand.  
The first detected open finger (thumb→pinky order) becomes the selected deck.

| Right-hand finger (open) | Selected track |
|---|---|
| Thumb | KICK |
| Index | BASS |
| Middle | HATS |
| Ring | CHORD |
| Pinky | VOCALS |

**Filter control (selected track):**
- Move your **right wrist up/down**:
  - Higher wrist → more open filter (brighter)
  - Lower wrist → more closed filter (darker)

Practical tip:
- For predictable selection, keep **only one finger** open on the right hand.  
  If multiple fingers are open, the app will select the earliest one in the list (thumb wins over index, etc.).

### Left Hand Height — FX Send (Selected Track)
- The **left wrist up/down** controls **FX send (delay amount)** on the currently selected track:
  - Higher wrist → more delay send
  - Lower wrist → less delay send

---

## Audio / FX Details

- Each track chain:
  - `BufferSource → Lowpass Filter → Gain → Master`
  - plus a send to a **global delay** FX bus
- The delay is configured as a tempo-ish echo (`delayTime ≈ 0.375s`) with feedback + filtering for darker repeats.
