# PixelForge

> **Professional Image Processing Studio** — Developed by 1CA

PixelForge is a desktop image processing application built with Python and PyQt6. It provides a clean, dark-themed UI for compressing, denoising, and watermarking images — with real-time previews and non-destructive output so your originals are never touched.

---

## Description

Most image processing tools are either bloated creative suites that charge a monthly fee, or barebones command-line utilities that require memorizing a dozen flags. PixelForge sits in the middle — a focused, professional desktop app that does three things exceptionally well: **compression**, **noise reduction**, and **watermarking**.

It was built for photographers, designers, content creators, and developers who regularly need to process images without opening a heavyweight editor. Every decision in the design prioritises speed and clarity. You open an image, toggle the tool you need, adjust one or two sliders, and save — the whole workflow takes seconds.

### Why PixelForge?

**It stays out of your way.**
The three tool panels are collapsed by default. You only expand what you actually need, so the interface never feels cluttered regardless of the task at hand.

**It tells you what will happen before it happens.**
The compression panel runs a live in-memory encode on every slider move, showing you the exact output file size and dimensions in real time — no guessing, no trial and error. The noise reduction panel renders a live preview directly on the canvas as you drag the strength slider, so you can find the right balance visually before committing.

**It never touches your originals.**
Every processed file is saved as a new file with a clear `_PixelForge` suffix. Your source image is always preserved exactly as it was.

**It runs entirely offline.**
No subscriptions, no cloud uploads, no API keys. Everything runs locally on your machine using OpenCV and Pillow — two of the most battle-tested image processing libraries in the Python ecosystem.

**It is keyboard and drag-drop friendly.**
Drop an image straight onto the canvas. Use `Ctrl+O` to open, `Esc` to clear. The floating load button sits centred at the bottom of the canvas so it is always reachable.

### What it is not

PixelForge is not a photo editor. It does not do colour grading, layer compositing, retouching, or generative effects. It is a precision utility — small, fast, and purpose-built for the three workflows it covers.

---

## Screenshots

> _Drop a screenshot of the app here once running._

---

## Features

### 🗜 Image Compression
- Adjustable quality slider (1–100%) with **live size estimation** as you drag
- Optional **target resize** — set a custom width and/or height (maintains aspect ratio if only one axis is set)
- Real-time stats badge shows original vs. estimated output dimensions and file size before you even click Save
- Saves as optimized JPEG

### ✦ Noise Reduction
- Edge-preserving **bilateral filter** (powered by OpenCV)
- Adjustable strength slider (5–150)
- **Live canvas preview** updates automatically as you drag the slider — no need to save first
- Runs in a background thread so the UI stays fully responsive

### © Copyright Watermark
- Custom text input for your copyright line
- Five placement positions: Bottom Right, Bottom Left, Top Right, Top Left, Center
- Adjustable opacity slider (0–100%)
- Auto-scaled font size proportional to image width for readability on any resolution
- Subtle drop shadow for legibility on both dark and light backgrounds

### 🎨 General UX
- **Drag & drop** — drop any image directly onto the canvas to load it
- **Floating action button** (📂) centred at the bottom of the canvas for quick image loading
- **Collapsible tool panels** — toggle each tool ON/OFF independently; only activate what you need
- **Single global save location** — set one output folder in the header bar, used by all three tools
- Non-destructive output — processed files are saved as new files (e.g. `photo_compress_PixelForge.jpg`), originals are never modified
- Image metadata strip in the header (filename, dimensions, file size)
- Keyboard shortcuts: `Ctrl+O` to open, `Esc` to clear canvas
- Smooth splash screen on launch

---

## Project Structure

```
PixelForge/
├── main.py          # Entry point — splash screen & app bootstrap
├── ui_main.py       # Main window, all UI layout and interaction logic
├── processor.py     # Background worker threads (processing + live preview)
├── styles.py        # Global Qt stylesheet (dark charcoal palette)
└── README.md
```

| File | Responsibility |
|---|---|
| `main.py` | `QApplication` setup, `SplashScreen` widget, timer-driven loading sequence |
| `ui_main.py` | `MainWindow`, `ToolPanel` card widget, `ToggleButton`, all layout builders |
| `processor.py` | `ImageProcessorWorker` (compress / denoise / watermark), `LivePreviewWorker` (debounced preview) |
| `styles.py` | Complete `STYLESHEET` string — deep charcoal palette, all widget overrides |

---

## Requirements

- **Python** 3.9 or later
- **PyQt6** — GUI framework
- **Pillow** — image open / save / watermark rendering
- **OpenCV** (`opencv-python`) — bilateral filter for noise reduction and live preview
- **NumPy** — array bridge between OpenCV and Qt

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/f0rtron/pixelforge.git
cd pixelforge
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install PyQt6 Pillow opencv-python numpy
```

### 4. Run the app

```bash
python main.py
```

---

## Usage

1. **Load an image** — drag and drop onto the canvas, or click the `📂` floating button (or press `Ctrl+O`)
2. **Choose a save folder** _(optional)_ — click `📁` in the header bar. If left unset, output files are saved next to the source image
3. **Toggle a tool ON** — click the `OFF` pill on any tool panel header to expand it
4. **Adjust settings** — use the sliders and inputs; compression stats and noise previews update in real time
5. **Click the action button** — `Compress & Save`, `Apply & Save`, or `Apply Watermark`
6. The processed image loads instantly in the canvas and the metadata strip updates

> Tools are independent — you can run compression, then denoise the result, then watermark it, all in the same session.

---

## How It Works

### Threading model
All heavy processing runs in a `QThread` subclass (`ImageProcessorWorker`) so the UI never freezes. Progress is reported back to the main thread via Qt signals (`progress`, `finished`, `error`, `stats`).

The live noise preview uses a separate lightweight `LivePreviewWorker` thread that:
1. Downscales the image to a max width of 640 px for speed
2. Applies a bilateral filter at the current slider value
3. Emits the result as a NumPy BGR array
4. The main thread converts it to a `QPixmap` and renders it in the canvas

A 380 ms debounce timer prevents the preview from firing on every pixel of slider movement.

### Compression stats
The real-time stats badge performs an **in-memory JPEG encode** (using a `BytesIO` buffer) on every slider change — no file is written to disk. This gives an accurate byte-level size estimate instantly.

### Output naming
Processed files follow the pattern:
```
{original_name}_{task}_PixelForge.jpg
```
For example: `sunset_compress_PixelForge.jpg`, `portrait_denoise_PixelForge.jpg`

---

## Supported Formats

| Input | Output |
|---|---|
| `.jpg` / `.jpeg` | `.jpg` |
| `.png` | `.jpg` |
| `.bmp` | `.jpg` (compression / watermark) or original ext (denoise) |
| `.webp` | `.jpg` |

> Transparent images (RGBA / palette mode) are automatically flattened to RGB before JPEG export.

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+O` | Open image file dialog |
| `Esc` | Clear the canvas |

---

## Dependencies & Licenses

| Package | License |
|---|---|
| [PyQt6](https://pypi.org/project/PyQt6/) | GPL v3 / Commercial |
| [Pillow](https://python-pillow.org/) | HPND (PIL License) |
| [opencv-python](https://pypi.org/project/opencv-python/) | MIT |
| [NumPy](https://numpy.org/) | BSD 3-Clause |

> **Note on PyQt6 licensing:** PyQt6 is licensed under the GPL v3. If you intend to distribute PixelForge commercially or as a closed-source product, you will need a commercial PyQt6 licence from Riverbank Computing, or consider switching to PySide6 (LGPL).

---

## Contributing

Pull requests are welcome. For significant changes, please open an issue first to discuss what you'd like to change.

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add some feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a pull request

---

## Roadmap

- [ ] Batch processing — process multiple images at once
- [ ] PNG export option for lossless output
- [ ] Brightness / contrast / saturation adjustments
- [ ] Undo / redo history
- [ ] Preset saving (save your favourite compression + resize settings)
- [ ] Drag-and-drop reordering of operations

---

## License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  <strong>PixelForge</strong> · Developed by <strong>1CA</strong>
</p>
