---
name: screenshot
description: Take screenshots of websites, desktop windows, or specific screen regions. Use when the user wants to capture an image of a web page, application window, desktop, or screen area. Trigger on "screenshot", "capture screen", "take a picture of", "screen grab", "snapshot", "capture the page", or when needing visual documentation of UI state.
---

# Screenshot

Capture screenshots of websites, windows, or the desktop. Three approaches depending on the target.

## 1. Website Screenshots (Built-in Browser)

Use the Playwright browser tools when a URL is involved:

1. Navigate: `playwright_browser_navigate` to the URL
2. Wait for content: `playwright_browser_wait_for` (text or time)
3. Screenshot: `playwright_browser_take_screenshot`
   - `scale: "css"` for standard resolution
   - `scale: "device"` for high-DPI
   - `fullPage: true` for entire scrollable page
   - `filename` to save to a specific path

For specific elements, pass `element` and `target` to capture just that region.

## 2. Desktop / Window Screenshots (Python + Pillow)

Pillow's `ImageGrab` captures the screen on Windows with no extra dependencies.

### Full Screen
```python
from PIL import ImageGrab

img = ImageGrab.grab()  # entire virtual screen (all monitors)
img.save("screenshot.png")

# Single monitor only
img = ImageGrab.grab(bbox=(0, 0, 1920, 1080))
img.save("monitor1.png")
```

### Region Capture
```python
from PIL import ImageGrab

# bbox = (left, top, right, bottom)
img = ImageGrab.grab(bbox=(100, 100, 800, 600))
img.save("region.png")
```

### Specific Window (via PyWin32)
```python
import win32gui
import win32ui
import win32con
from PIL import ImageGrab

def screenshot_window(title_substring):
    # Find window by title
    hwnd = win32gui.FindWindow(None, None)

    def enum_handler(h, _):
        if title_substring.lower() in win32gui.GetWindowText(h).lower():
            return h
        return True

    win32gui.EnumWindows(enum_handler, None)
    # Bring to front and capture
    win32gui.SetForegroundWindow(hwnd)
    rect = win32gui.GetWindowRect(hwnd)
    img = ImageGrab.grab(bbox=rect)
    return img
```

### Delayed Screenshot (for capturing menus/transitions)
```python
import time
from PIL import ImageGrab

time.sleep(2)  # give user time to switch windows / open menu
ImageGrab.grab().save("delayed.png")
```

## 3. Batch / Repeated Capture
```python
import time
from PIL import ImageGrab

for i in range(5):
    ImageGrab.grab().save(f"capture_{i:03d}.png")
    time.sleep(1)
```

## Saving Tips

- Use `.png` for UI screenshots (lossless)
- Use `.jpg` for photos (smaller files)
- Use `.webp` for web-optimized images
- Create a `screenshots/` directory for organized output
- Include timestamp in filename: `screenshot_20240115_103022.png`

## Quick One-Liners (PowerShell)

```powershell
# Full screen via .NET (no Python needed)
Add-Type -AssemblyName System.Windows.Forms,System.Drawing
$bounds = [System.Windows.Forms.Screen]::PrimaryScreen.Bounds
$bitmap = New-Object System.Drawing.Bitmap($bounds.Width, $bounds.Height)
$graphics = [System.Drawing.Graphics]::FromImage($bitmap)
$graphics.CopyFromScreen($bounds.Location, [System.Drawing.Point]::Empty, $bounds.Size)
$bitmap.Save("C:\temp\screenshot.png")
$graphics.Dispose(); $bitmap.Dispose()
```

## Workflow

1. Determine target: website, window, region, or full desktop
2. Choose approach: browser tools (URLs) / Python (desktop) / PowerShell (quick)
3. Capture and save
4. Report the file path to the user
