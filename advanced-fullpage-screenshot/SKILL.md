---
name: advanced-fullpage-screenshot
description: Captures a full-page screenshot of a URL using CSS translateY to scroll through the page viewport by viewport, then stitches the captures into a single PNG. Includes a pre-scroll pass to trigger lazy-loaded content, freezes GIFs/CSS animations/videos before capture to prevent mid-animation frames, handles devicePixelRatio for retina displays, and crops the final viewport to avoid blank overflow. Use this when the user asks for a fullpage or full-page screenshot of a URL.
---

**Usage:** `/advanced-fullpage-screenshot <url> [--no-freeze]`

- `--no-freeze` — skip the freeze-page step (disables GIF/animation/video freezing)

---

## Steps

### 1. Ensure Pillow is installed
```bash
python3 -c "import PIL" 2>/dev/null || pip3 install Pillow
```

### 2. Prepare output directory
Ensure `tmp/screenshots/` exists relative to the current working directory:
```bash
mkdir -p tmp/screenshots
```

### 3. Open the URL
Use `new_page` with the URL from `$ARGUMENTS`.

### 4. Wait for page load
Poll via `evaluate_script` until `document.readyState` returns `'complete'`:
```js
() => document.readyState
```

### 5. Save initial page state
Run via `evaluate_script` and store as `initialState`:
```js
() => ({
    scrollX: window.scrollX,
    scrollY: window.scrollY,
    overflowBody: document.body.style.overflow,
    overflowDocument: document.documentElement.style.overflow,
    transform: document.body.style.transform
})
```

### 6. Hide scrollbar
Only set `overflow: hidden` on `body` — do NOT set it on `documentElement`. Setting it on `html` clips translated body content and produces blank viewports.
```js
() => {
    document.body.style.overflow = 'hidden';
}
```

### 7. Get page dimensions
Run via `evaluate_script` and store all values:
```js
() => ({
    viewportHeight: window.innerHeight,
    viewportWidth: window.innerWidth,
    fullpageHeight: Math.max(
        document.body.offsetHeight, document.body.scrollHeight,
        document.documentElement.offsetHeight, document.documentElement.scrollHeight
    ),
    fullpageWidth: Math.max(document.body.offsetWidth, document.documentElement.offsetWidth),
    devicePixelRatio: window.devicePixelRatio
})
```

Compute: `totalPages = Math.ceil(fullpageHeight / viewportHeight)`

Cap: if `totalPages > 15`, set `totalPages = 15` and note that the page was capped.

### 8. Pre-scroll pass (trigger lazy-loaded content)

Scroll through the entire page using `window.scroll` so off-screen images and sections load before capture. Use a 300ms wait between each step via `wait_for` (or a short `sleep` if unavailable).

For `i` from `1` to `totalPages - 1`, run via `evaluate_script`:
```js
// pixels = viewportHeight * i
() => { window.scroll(0, {pixels}); }
```

After the last step, scroll back to top and wait 500ms for content to settle:
```js
() => { window.scrollTo(0, 0); }
```

Re-query page dimensions after the pre-scroll pass — lazy content may have increased `fullpageHeight`. Update `fullpageHeight` and recompute `totalPages` if changed.

### 9. Freeze page (default ON, skip if `--no-freeze` passed)

Run via `evaluate_script` to freeze GIFs, CSS animations, and videos at their current frame before capture:

```js
() => (()=>{let e=0;let t=function(t){var a=/^(?!data:).*\.gif/i.test(t.src);if(a){void 0;e++}return a};let a=function(e){var t=document.createElement("canvas");var a=t.width=e.width;var r=t.height=e.height;t.getContext("2d").drawImage(e,0,0,a,r);try{e.src=t.toDataURL("image/gif")}catch(a){for(var i=0,n;n=e.attributes[i];i++);e.parentNode.replaceChild(t,e)}};let r=function(){[].slice.apply(document.images).filter(t).map(a);return e};let i=function(){var e=document.getElementsByTagName("*");let t=0;for(var a=0;a<e.length;a++){var r=e[a];try{var i=window.getComputedStyle(r,null).getPropertyValue("animation-duration");if(parseInt(i)>0){r.style.animationDuration="0s";t++;void 0}}catch(e){}}return t};let n=function(){if(window.jQuery){void 0;jQuery.fx.off=true}};let o=function(e){return e.tagName+(e.id?"#"+e.id:e.className?"."+e.className:"")};let c=function(){const e=document.querySelectorAll("video");let t=0;if(e.length)e.forEach((e=>{if(e.currentTime!==0){t++;if(!e.loop&&e.duration<10){void 0;e.pause();e.autoplay=false;e.currentTime=e.duration}else{void 0;e.pause();e.autoplay=false;e.currentTime=0}}}));return t};const l=()=>{const e=document.querySelectorAll("*");let t=0;e.forEach((e=>{const a=window.getComputedStyle(e).getPropertyValue("background-image");if(a.includes(".gif"))try{const r=new Image;r.src=a.replace(/url\(['"]?/g,"").replace(/['"]?\)/g,"");const i=document.createElement("canvas");i.width=r.width;i.height=r.height;const n=i.getContext("2d");n.drawImage(r,0,0,r.width,r.height);const o=i.toDataURL("image/gif");if(!r.width||!r.height)throw Error('Issue handling background gif — likely chrome issue from enabling "disable cache".))');e.style.backgroundImage=`url(${o})`;void 0;t++}catch(e){void 0}}));return t};n();return{totalFrozen:{gifs:r(),bgGifs:l(),animations:i(),videos:c()}}})()
```

Log the returned `totalFrozen` counts for reference.

### 10. Single-page fast path

If `totalPages == 1`: skip all translate logic. Take one screenshot directly:
- `filePath`: `tmp/screenshots/.tmp_vp_0.png`

Then jump to step 12 (Restore). Pass `remainder = 0` to the stitch script (no crop needed).

### 11. Multi-page: CSS translate scroll and capture loop

Only run this step if `totalPages > 1`.

For `pageIndex` from `0` to `totalPages - 1`:

**Scroll:**
- `pageIndex == 0`: do NOT apply any transform — the page is already at the top after the pre-scroll pass. Take the screenshot at the natural state.
- `pageIndex > 0`: substitute `pixels = viewportHeight * pageIndex`

```js
// pageIndex N > 0 only (substitute actual pixel value):
() => { document.body.style.transform = 'translateY(-{pixels}px)'; }
```

**Capture** the viewport using `take_screenshot` with:
- `filePath`: `tmp/screenshots/.tmp_vp_{pageIndex}.png` (use absolute path resolved from `os.getcwd()`)

### 12. Restore page state

Substitute the actual saved `initialState` values into this script:
```js
() => {
    document.body.style.transform = '{initialState.transform}';
    document.body.style.overflow = '{initialState.overflowBody}';
    window.scrollTo({initialState.scrollX}, {initialState.scrollY});
}
```

### 13. Stitch images with Pillow

Generate the output filename by sanitizing the URL to alphanumeric+hyphens and appending a timestamp (`YYYYMMDD-HHMMSS`). Example: `example-com-20260401-143022.png`.

Substitute the actual runtime values and run inline via heredoc — no file write needed:

```bash
python3 << 'EOF'
from PIL import Image
import os

base_dir  = os.path.abspath("tmp/screenshots")
tmp_dir   = base_dir
n_pages   = {totalPages}
vh        = {viewportHeight}
fph       = {fullpageHeight}
dpr       = {devicePixelRatio}
out_path  = os.path.join(base_dir, "{outputFilename}")

paths  = [os.path.join(tmp_dir, f'.tmp_vp_{i}.png') for i in range(n_pages)]
images = [Image.open(p) for p in paths]

remainder = fph % vh
if remainder != 0:
    crop_px = int(round((vh - remainder) * dpr))
    last = images[-1]
    images[-1] = last.crop((0, 0, last.width, last.height - crop_px))

total_height = sum(img.height for img in images)
max_width    = max(img.width  for img in images)

canvas = Image.new('RGB', (max_width, total_height))
y = 0
for img in images:
    canvas.paste(img, (0, y))
    y += img.height

canvas.save(out_path)
print(f'Saved: {out_path} ({max_width}x{total_height}px)')

for p in paths:
    os.remove(p)
EOF
```

### 14. Report
Output the final image path and dimensions to the user.

### 15. Close the browser tab
After reporting, close the tab used for capture using `close_page`, unless additional screenshots are queued in the same session.
