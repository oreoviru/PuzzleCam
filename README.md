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

## **ALGORITHMS & HOW THEY WORK**

PuzzleCam uses a pipeline of specialized algorithms combining deep learning, geometric heuristics, probability, image processing, and numerical physics:

```
Webcam Frame
    │
    ▼
MediaPipe CNN (HandLandmarker) ──► 21 Normalized 3D Keypoints per Hand
    │
    ▼
Geometric Heuristics ────────────► Pinch Detection & Fist Detection
    │
    ▼
Dual-Hand Bounding Box ──────────► Captures Cropped Photo Area
    │
    ▼
Image Filters ───────────────────► Luminance + Box-Muller Gaussian Noise
    │
    ▼
Fisher-Yates Shuffle ────────────► Randomizes 3x3 Tile Placements
    │
    ▼
Drag, Snap & Displacement ───────► Distance-based Snapping & Cubic Ease-Out Displacement
    │
    ▼
Euler Physics Simulation ────────► 36-Piece Radial Shatter Animation on Save
```

### 1. Computer Vision & Landmark Detection (MediaPipe Pipeline)
- **Model**: Google MediaPipe `HandLandmarker` (two-stage pipeline: BlazePalm detector + 3D hand landmark model) executed on WebAssembly (WASM) with GPU/CPU delegates.
- **How it works**:
  1. **Palm Detection**: An initial single-shot detector identifies palm bounding boxes across the full video frame.
  2. **Landmark Regression**: Crops around each detected palm and feeds the sub-image into a neural network to regress the $(x, y, z)$ coordinates of **21 3D hand landmarks** (wrist, knuckles/MCP joints, PIP joints, DIP joints, and fingertips).
  3. Coordinates are normalized to $[0, 1]$ relative to webcam dimensions.

### 2. Geometric Gesture Recognition
Converts raw landmark coordinates into user interactions using spatial heuristics:

- **Pinch Detection (`isPinching`)**:
  Computes the 2D Euclidean distance between the thumb tip (Landmark 4) and index fingertip (Landmark 8):
  $$\text{dist} = \sqrt{(x_{\text{thumb}} - x_{\text{index}})^2 + (y_{\text{thumb}} - y_{\text{index}})^2}$$
  A pinch is active when $\text{dist} < 0.055$.

- **Fist Detection (`isFist`)**:
  Calculates whether each fingertip (index, middle, ring, pinky) is closer to the wrist (Landmark 0) than its respective metacarpophalangeal (MCP) knuckle joint:
  $$\text{dist}(\text{Tip}, \text{Wrist}) < \text{dist}(\text{MCP}, \text{Wrist})$$
  When at least 4 fingers are curled, a fist is detected. A debounce counter (`FIST_HOLD_FRAMES = 12`) ensures the fist must be held intentionally.

- **Dual-Hand Framing Box (`computeHandFrame`)**:
  Constructs an Axis-Aligned Bounding Box (AABB) using the index fingertips of both hands as opposite diagonal anchors:
  $$x_{\min} = \min(x_1, x_2) - \text{padding}, \quad x_{\max} = \max(x_1, x_2) + \text{padding}$$
  $$y_{\min} = \min(y_1, y_2) - \text{padding}, \quad y_{\max} = \max(y_1, y_2) + \text{padding}$$
  Clamped to canvas bounds to define the snapshot crop area.

### 3. Tile Shuffling (Fisher-Yates / Knuth Algorithm)
- **Where**: `shuffle(arr)` in `app.js`
- **How it works**:
  Iterates backwards from the last array index ($i = N - 1$ down to 1), choosing an unbiased random index $j \in [0, i]$ and swapping the elements at $i$ and $j$:
  ```javascript
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  ```
- **Time Complexity**: $O(N)$ with $O(1)$ auxiliary space. Guarantees every permutation is equally likely ($1 / N!$).

### 4. Photobooth Filter (Luminance & Box-Muller Gaussian Noise)
- **ITU-R BT.601 Luminance Formula**:
  $$\text{gray} = 0.299 R + 0.587 G + 0.114 B$$
- **Contrast & Brightness Adjustment**:
  $$v = \text{gray} \cdot \alpha + \beta \quad (\alpha = 1.3, \, \beta = 10)$$
- **Box-Muller Transform for Analog Film Grain**:
  Converts two independent uniform random variables $U_1, U_2 \sim U(0, 1)$ into a standard normal (Gaussian) distribution:
  $$Z_0 = \sqrt{-2 \ln(U_1)} \cos(2\pi U_2)$$
  $$\text{noise} = Z_0 \cdot \sigma \quad (\sigma = 15)$$
  $$v_{\text{final}} = \min(255, \max(0, v + \text{noise}))$$
  This creates realistic film grain rather than digital artifact noise.

### 5. Puzzle Mechanics (Hit-Testing, Snapping & Displacement)
- **Nearest Neighbor Hit-Testing (`findNearestPiece`)**:
  Finds the closest tile to the index fingertip within an activation radius ($0.75 \times \max(\text{width}, \text{height})$).
- **Distance-Based Snap (`isNearOwnCell`)**:
  Calculates whether a tile is within snapping tolerance of its designated home cell:
  $$\Delta = \sqrt{(x - x_{\text{target}})^2 + (y - y_{\text{target}})^2} < \min(\text{tileW}, \text{tileH}) \times 0.45$$
- **Cell Occupancy & Conflict Displacement (`displaceCellOccupant`)**:
  If tile $A$ is dropped or snapped into an already occupied grid cell:
  1. Identifies the conflicting occupant tile via AABB overlap.
  2. Queries the $3 \times 3$ grid to locate all currently free/unoccupied slots.
  3. Displaces the occupant to an open slot with slight jitter.
  4. Animates displacement smoothly using a cubic ease-out curve:
     $$\text{eased}(t) = 1 - (1 - t)^3 \quad \text{for } t \in [0, 1]$$

### 6. Shatter Effect (Semi-Implicit Euler Physics Simulation)
- **Where**: `startShatter` & `updateAndDrawShatter` in `app.js`
- **How it works**:
  1. Slices the solved canvas into a $6 \times 6$ grid (36 fragments).
  2. Assigns each fragment an initial radial velocity vector outward from the board's center:
     $$\vec{d} = (x_{\text{frag}} - x_{\text{center}}, \, y_{\text{frag}} - y_{\text{center}})$$
     $$\vec{v}_0 = \frac{\vec{d}}{\|\vec{d}\|} \cdot \text{speed} + \vec{v}_{\text{jitter}}$$
  3. Simulates projectile physics every frame ($\Delta t \approx 1/60\text{s}$) using Euler integration:
     $$x_{t + \Delta t} = x_t + v_x \Delta t$$
     $$y_{t + \Delta t} = y_t + v_y \Delta t$$
     $$v_{y, t + \Delta t} = v_{y, t} + g \Delta t \quad (g = 220 \dots 300)$$
     $$\theta_{t + \Delta t} = \theta_t + \omega \Delta t$$
  4. Fades fragment opacity and scale over 850ms before finalizing the capture into the photo strip.

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
