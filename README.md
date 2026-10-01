# PuzzleCam — Gesture Capture

Hand-gesture controlled photobooth app that runs entirely in the browser. No installation, no backend, and no dependencies to install.

---

## **DESCRIPTION**

PuzzleCam captures a photo using your hands as a "frame", converts it into a 3x3 puzzle with a black-and-white photobooth effect, and allows you to solve it using pinch gestures. Once completed, it is saved into a downloadable photo strip.

---

## **SYSTEM REQUIREMENTS**

- **Browser:** Chrome or Edge (recommended), Firefox
- **Hardware:** Webcam
- **Internet connection:** Required to load the MediaPipe model (~10MB, first time only)
- **Local server:** Required to run the app (cannot be opened directly as a file)

---

## **INSTALLATION & SETUP**

### 1. Clone the repository

```bash
git clone https://github.com/mishu006/Puzzle.git
cd Puzzle
```

### 2. Start a local server

The app uses ES modules and webcam access, so it must run over HTTP.

You can run:
```bash
python3 -m http.server 8000
```
Or install the [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) extension in VS Code and click **Go Live**.

### 3. Open in your browser

```
http://localhost:8000
```

Allow camera access when prompted by your browser.

---

## **PROJECT STRUCTURE**

```
Puzzle/
├── index.html        # Main app entry point
├── app.js            # Core logic (tracking, puzzle, gallery)
├── css/
│   └── styles.css    # Styles and layout
└── .gitignore
```

---

## **CONTROL GESTURES**

| Gesture | Action |
|---|---|
| Both hands pinching | Freeze frame area and start countdown |
| One hand pinching a piece | Drag the puzzle piece |
| Closed fist (hold) | Save completed puzzle / Reset board |

---

## **APPLICATION FLOW**

1. Show both hands to the camera and pinch to define the capture frame.
2. Hold the pinch during countdown — the photo is taken automatically.
3. The photo is split into a 3x3 puzzle with a black-and-white photobooth filter.
4. Rearrange the pieces using pinch gestures.
5. When completed, close your fist to save to the strip with a shatter animation.
6. Download the full strip once you have 3 saved puzzles.

---

## **TECH STACK**

- **[MediaPipe Tasks Vision](https://developers.google.com/mediapipe)** `v0.10.14` — hand landmark detection
- **Canvas 2D API** — rendering, puzzle pieces, photobooth effect
- **JavaScript (ES Modules)** — vanilla, no frameworks
- **CSS Custom Properties** — theming and layout

All external dependencies are loaded via CDN. No extra installation required.

---

## **TROUBLESHOOTING GUIDE**

### **The camera does not turn on**

Ensure no other application (Teams, Zoom, Discord, etc.) is currently using the camera in the background.

### **The app does not load the model**

Check your internet connection. The MediaPipe model (~10MB) is downloaded from `storage.googleapis.com` and the runtime from `cdn.jsdelivr.net`. If any of these domains are blocked on your network, the app cannot start.

### **The app shows a black screen**

Ensure you are opening the app via a local HTTP server, not directly as a file from your file explorer.

### **Pinch gesture is not detected**

Ensure you have good lighting and both hands are clearly visible to the camera. Bring your thumb and index finger closer together until the yellow dot on the screen activates.

---

## **BROWSER COMPATIBILITY**

| Browser | Support |
|---|---|
| Chrome / Edge | Recommended |
| Firefox | Supported |
| Safari | Limited (may require additional permissions) |
| Mobile | Limited (desktop recommended) |

---

## **LICENSE**

MIT — free to use, modify, and share.
