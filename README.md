# Sign-Speech

# Universal Accessibility Platform

A single-page web app for **learning American Sign Language (ASL)** and **communicating with it through a webcam**. It recognises fingerspelled letters, everyday signs and drawn letters, then types them into a message box and speaks them aloud.

Everything lives in one self-contained `index.html` file (about 600 KB, with all 46 reference pictures embedded). Hand tracking runs in the browser, and no video leaves the device.

---

## Table of contents

- [Features](#features)
- [Quick start](#quick-start)
- [How to use each mode](#how-to-use-each-mode)
- [Routes](#routes)
- [How it works](#how-it-works)
- [Accessibility](#accessibility)
- [Privacy and data storage](#privacy-and-data-storage)
- [Browser support and requirements](#browser-support-and-requirements)
- [Project structure](#project-structure)
- [Customising and extending](#customising-and-extending)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Credits](#credits)
- [License](#license)

---

## Features

The home page offers two paths: **Learn sign language** and **Talk using sign language**.

### Learn sign language

- A picture dictionary of the **ASL alphabet (A–Z)** and **20 everyday signs**.
- Each entry has a photo, a plain-language description of how to make it, and a tip.
- Search across names and descriptions.
- A **Practise with the camera** button jumps straight into the matching talk mode.

### Talk using sign language

| Mode | What it does |
| --- | --- |
| **Fingerspelling** | Show a letter with your hand. Hold it steady until the progress bar fills and the letter is typed and spoken. |
| **Basic signs** | Sign a word such as *yes*, *please* or *help*. The word is typed and spoken when recognised. |
| **Draw a letter** | Write a capital letter with a finger, mouse or pen on a board, or with your index finger in the air. |
| **Draw in the air** | Point with your index finger to draw over the camera picture, with several tools and colours. |
| **My own gestures** *(optional)* | Record your own hand movements and assign each a phrase to type and speak. |

### Shared tools

- **Message bar** with a text area, *Speak all*, *Space*, *Repeat letter* (for double letters such as LL), backspace and *Clear*.
- **Voice** output through the browser's speech synthesis, with a mute toggle.
- **Teach the camera** your own version of a letter or sign.
- **Three text sizes**, and haptic feedback on phones that support vibration.

### The 20 everyday signs

`yes` · `no` · `please` · `thank you` · `sorry` · `help` · `more` · `all done` · `find` · `want` · `open` · `sit` · `sleep` · `play` · `eat` · `drink` · `hurt` · `restroom` · `stop` · `go`

---

## Quick start

There is nothing to install or build. You only need to serve the file, because browsers only allow camera access on **HTTPS or `localhost`**.

```bash
# from the folder containing index.html
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

Any static host works for deployment (GitHub Pages, Netlify, Cloudflare Pages, Vercel, an S3 bucket). The host must serve over HTTPS.

> Opening `index.html` directly with a `file://` URL will show the Learn dictionary, but the camera and the model loader may be blocked. Use a local server.

---

## How to use each mode

The camera stays **off** until you press **Start**. Each camera screen has Start and Stop buttons.

### Fingerspelling

1. Open **Talk → Fingerspelling** and press **Start**.
2. Face your palm toward the camera and make a letter. Hold it until the bar fills.
3. Change your hand shape before the next letter.

Letter-specific notes:

- **Space**: an open hand with fingers spread, held steady.
- **J and Z**: these move. Make the I (for J) or the pointing hand (for Z) and draw the letter slowly in the air, in about a second.
- **G and H**: turn your hand sideways.
- **P and Q**: point your fingers down.
- **Hold time** can be set to Slow (1.25 s), Normal (0.85 s) or Fast (0.55 s).
- If a letter keeps being mixed up, select it in the guide, make it your way and press **Teach the camera my letter**. It records about two seconds of your hand and also matches against it from then on. **Reset** clears this.

### Basic signs

1. Open **Talk → Basic signs** and press **Start**.
2. Stand back so the camera sees your hands **and face**. Several signs are defined relative to the mouth, chin or chest.
3. Sign at a steady pace. For two-hand signs, keep both hands in view.
4. If a sign is not recognised, select it and press **Teach the camera my version of this sign**. Record it two or three times.

### Draw a letter

- Draw **big, block capital letters** on the board with a finger, mouse or pen. Multi-stroke letters work in any stroke order.
- When you pause (Short 0.7 s, Normal 1.1 s or Long 1.7 s), the letter is typed and spoken. A live guess appears as you draw.
- To write **several letters at once**, draw them left to right with a small gap between letters. This can be switched off.
- **In the air**: press **Start camera**, point with your index finger and hold it still until the ring turns green, then draw. Curl or lower your finger to lift the pen. Hold an open hand still for one second to add a space.
- If a letter is wrong, tap the correct one. It is typed in place of the wrong one and the app **learns your handwriting** for next time. **Forget my corrections** clears this.
- **Ask before typing** shows the top candidates and waits for you to choose (Enter picks the first).
- **Air pen starts after** (Quick / Normal / Careful) sets how long your finger must be still before drawing begins.

### Draw in the air

- Point with your index finger and curl the other fingers to draw. Open your hand, make a fist or lower your hand to lift the pen.
- **Tools**: Pen, Marker, Rainbow, Line, Box, Circle and Eraser.
- **Colours**: 8 swatches. **Pen size**: Thin, Medium, Thick or Extra thick.
- **Undo / Redo / Clear**. Holding an open hand still for 1.5 seconds also clears the picture, and Undo brings it back.
- **Plain board** hides the camera so you draw on a solid background.
- **Save picture** downloads `air-drawing.png` (1280×960, mirrored to match what you saw).

### My own gestures

1. Open **Talk → record your own hand movements** (a link at the bottom of the Talk page) and press **Start**.
2. Enter a **Name** and **What it says**.
3. Press **Record example** at least twice. A held pose is stored as a pose and a moving gesture as a movement.
4. Press **Save gesture**, then turn **Shortcut mode** on. Matching gestures type and speak their phrase.

---

## Routes

Navigation uses hash routes, so any screen can be linked to directly and the Back button works.

| Hash | Screen |
| --- | --- |
| `#/` | Home |
| `#/learn` | Dictionary |
| `#/talk` | Choose a talk mode |
| `#/talk/spell` | Fingerspelling |
| `#/talk/signs` | Basic signs |
| `#/talk/write` | Draw a letter |
| `#/talk/draw` | Draw in the air |
| `#/talk/mine` | My own gestures |

Changing screens stops the camera and resets the recognisers.

---

## How it works

### Hand and face tracking

The app uses [MediaPipe Tasks Vision](https://developers.google.com/mediapipe) (`@mediapipe/tasks-vision` 0.10.14, loaded from jsDelivr):

- **Hand Landmarker** (up to 2 hands) runs on every screen that uses the camera.
- **Face Landmarker** (1 face) is loaded only for **Basic signs** and **My own gestures**, which need mouth and chin positions.
- It tries the **GPU** delegate first and falls back to **CPU**.
- The camera stream is 640×480, front-facing, with no audio. Both models are downloaded the first time they are needed.

### Recognition engine

All recognition is hand-written JavaScript on top of the 21 landmarks per hand. There is no trained classifier for the letters or signs themselves.

1. **Canonical hand frame.** Every hand is mirrored to look like a right hand, rotated so the wrist-to-middle-knuckle line points up, and scaled by palm length. Comparisons therefore do not depend on distance, rotation or handedness.
2. **Synthetic letter templates.** Hand shapes are defined as finger bend angles plus a thumb target and a preferred orientation, then turned into landmarks with simple forward kinematics. This produces a template bank with several rolls per letter.
3. **Static letter classifier.** A weighted landmark distance with an orientation penalty (which separates look-alikes such as U/H, G/Q and K/P). A small **rule layer** corrects the letters that are easily confused (E, X, G, Q, P).
4. **Voting and hold-to-add.** A majority vote over the last 8 frames (at least 70 % agreement) prevents one bad frame from changing the answer. A letter is committed once it is held for the chosen time.
5. **J and Z.** The fingertip path is recorded, simplified and checked against the expected shape. Z is three strokes (across, diagonally back and down, across). J is a downward stroke with a hook.
6. **Basic signs.** Rule-based scoring over roughly the last 1.7 seconds. It uses hand state (open, fist, pointing, gathered fingertips, and so on), movement (direction reversals, circles, lifts, hands moving together or apart) and position relative to the face and chest. A sign fires at a score of 0.8 or more, then rests for 1.8 seconds so it is not typed twice.
7. **Personal gestures and sign recordings.** Hand shape and movement are turned into 88-dimension frame features, resampled to 30 frames, and compared with **dynamic time warping**. A segmenter finds where a gesture starts and ends.
8. **Drawn letters.** Strokes are resampled to 32 points, normalised, and matched against glyph templates with a point-cloud matcher. This is stroke-order and stroke-count independent. Strokes that overlap sideways are grouped into one letter. User corrections are added as extra templates (up to 150 are kept).

### Learning your style

Teaching a letter or sign, recording a gesture and correcting a drawn letter all add examples to the matchers. They are stored in the browser (see below).

---

## Accessibility

- **Text size**: three levels (base 16, 19 or 23 px), available on every page and remembered between visits.
- **Screen readers**: status messages use `aria-live`, controls have labels, toggles use `aria-pressed`, and typed letters are announced through a live region.
- **Keyboard**: visible focus rings, native buttons and links, and `Ctrl/Cmd + Enter` to speak the message.
- **Targets**: buttons and inputs are at least 44–48 px tall.
- **Reduced motion**: the animated background and transitions are turned off when `prefers-reduced-motion` is set.
- **Contrast and theme**: a dark colour scheme with high-contrast text.
- **Mobile**: responsive layout, safe-area insets, and haptic feedback on recognition where supported.
- **No-camera alternatives**: the dictionary and the **Draw a letter** board work without a camera, using touch, mouse or pen.

---

## Privacy and data storage

- The camera is only used after you press **Start** and is released when you press **Stop** or leave the screen.
- Hand and face tracking run **on your device**. The app does not upload video or images.
- Your preferences and personal training data are stored only in your browser's `localStorage`:

| Key | Contents |
| --- | --- |
| `st.size` | Text size |
| `st.mute` | Voice muted or not |
| `st.hold` | Fingerspelling hold time |
| `st.user` | Your taught letters |
| `st.sx` | Your recordings of basic signs |
| `st.gest` | Your custom gestures |
| `st.wuser` | Your drawn-letter corrections |
| `st.wdelay`, `st.warm` | Draw-a-letter timing settings |
| `st.wask`, `st.wmulti` | Draw-a-letter toggles |

To remove everything, clear the site's data in your browser. The in-app **Reset**, **Forget my corrections** and **Delete** buttons clear individual items.

**Network requests made by the page**

| Host | Purpose |
| --- | --- |
| `cdn.jsdelivr.net` | MediaPipe Tasks Vision library and WASM files |
| `storage.googleapis.com` | Hand and face landmark model files |
| `fonts.googleapis.com`, `fonts.gstatic.com` | Inter and Instrument Serif web fonts (system fallbacks are used if blocked) |

---

## Browser support and requirements

- A modern browser with **ES modules and dynamic `import()`**, **WebAssembly**, `getUserMedia` and (for voice) the **Web Speech API**. Current Chrome, Edge, Safari and Firefox should work, though speech voices and performance vary by platform.
- **HTTPS or `localhost`** for camera access.
- **An internet connection** the first time you use a camera mode, to fetch the MediaPipe library and models.
- A **front-facing webcam**. Laptops and phones both work.
- Voice output depends on the voices installed on your device. If there is no speech support, text is still typed.

---

## Project structure

Everything is in `index.html`:

```
index.html
├── <style>          Theme, layout, components (CSS variables at :root)
├── <body>           Views: #v-home, #v-learn, #v-talk, #v-cam,
│                           #v-draw, #v-write, #v-mine, plus the shared message bar
└── <script module>
    ├── 1  canon()                 Canonical hand frame
    ├── 2  SHAPES, makeHand()      Synthetic hands and letter definitions
    ├── 3  classify(), ruleLetter  Static letter classifier + rule layer
    ├── 4  isJ(), isZ()            Moving letters
    ├── 5  Speller, Vote           Hold-to-add spelling
    ├── 6  hs(), judge(),          Basic signs (rule based)
    │      SignWatcher
    ├── 7  feat(), dtw(),          Personal gestures (DTW templates)
    │      Segmenter
    ├── 8  GLY, recognizeLetter()  Drawn-letter point-cloud matcher
    ├── IMG                        46 embedded JPEGs (A–Z and 20 signs)
    ├── content                    LETTER_HOW, SIGN_HOW, SIGN_NAME, …
    ├── UI                         Dictionary, guide, message bar, toasts
    ├── camera + tracking          MediaPipe loading, frame loop
    ├── draw in the air            PAD state, tools, save to PNG
    ├── draw a letter              WR state, ink board, learning
    └── routing                    Hash router
```

The recognition engine section is commented as pure code with no browser APIs, which was tested in Node and inlined into the page by a build step. That build script and its tests are not part of this file. If you keep a separate source copy, edit that and rebuild. Otherwise edit the inline script directly.

---

## Customising and extending

| I want to… | Where to change it |
| --- | --- |
| Change colours or fonts | CSS variables at the top of `<style>` (`--c1`, `--c2`, `--c3`, `--bg`, `--ink`, `--serif`, `--sans`) |
| Edit a letter or sign description | `LETTER_HOW`, `LETTER_TIP`, `SIGN_HOW`, `SIGN_NAME` |
| Replace a picture | The matching key in the `IMG` object (`A`–`Z`, `s_yes`, `s_thank_you`, …) |
| Adjust letter hold times | `holdMs` (`slow`, `normal`, `fast`) |
| Tune letter strictness | The threshold `T` in `classify(c, T = .34)` |
| Tune sign sensitivity | The `>= .8` trigger and `rest` time in `SignWatcher.push` |
| Add or fix a drawn-letter shape | Add a `gl('X', …strokes)` line in section 8 |
| Add a pen colour or tool | `PEN` and `TOOLS`, then handle it in `renderPad()` |
| Add a new letter hand shape | An entry in `SHAPES` with finger bends, a thumb target and an orientation |

**Adding a new sign** means updating several places:

1. Add its id to `SIGNS`, and its labels to `SIGN_TEXT` and `SIGN_NAME`.
2. Add `SIGN_HOW[id]` (description and tip) and an image under `IMG['s_<id>']`.
3. Add a scoring rule for it in `judge()` that returns a value from 0 to 1.

---

## Troubleshooting

**"The camera was blocked or not found."**
Allow camera access for the site in your browser. Check that no other app is using the camera, and that the page is served over HTTPS or `localhost`.

**"Could not load the tracking model…"**
The MediaPipe library or model files could not be downloaded. Check your connection and whether your network or an extension is blocking `cdn.jsdelivr.net` or `storage.googleapis.com`.

**A letter is mistyped or never recognised.**
Use good, even lighting and a plain background. Keep your palm toward the camera with the hand upright (except G, H, P and Q). Hold the shape steadily. Use **Teach the camera my letter** to add your own version.

**A sign is not recognised.**
Stand back so both hands and your face are in view. Slow down a little and keep a steady pace. Record your own version two or three times with **Teach the camera my version of this sign**.

**Drawn letters come out wrong.**
Draw larger, in block capitals, with a clear gap between letters. Tap the correct letter when it is wrong so it learns your handwriting.

**There is no voice.**
Check that the **Voice** button is not muted and that your device has a speech voice installed. *Speak all* always speaks, even when muted.

**It runs slowly.**
Close other tabs, and try a browser with GPU/WebGL support. The app falls back to CPU if the GPU delegate fails.

---

## Limitations

- **ASL only.** Other sign languages are not covered.
- **Fingerspelling is not the same as ASL.** Real conversation relies on grammar, facial expression and a much larger vocabulary. This app covers the alphabet and 20 signs and should be treated as a learning and communication aid, not a replacement for a qualified interpreter or instructor.
- **Heuristic recognition.** Accuracy depends on lighting, camera angle, distance and individual hand shape. Teaching your own examples helps.
- **Block capitals only** in Draw a letter. There are no digits, lowercase letters or punctuation.
- **Single signer.** Face and hand tracking assume one face (and up to two hands).
- **Network needed on first use** of a camera mode.

---

## Credits

- Hand and face tracking: [MediaPipe Tasks Vision](https://developers.google.com/mediapipe) by Google (Apache 2.0).
- Typefaces: [Inter](https://rsms.me/inter/) and [Instrument Serif](https://fonts.google.com/specimen/Instrument+Serif), served by Google Fonts (SIL Open Font License).
