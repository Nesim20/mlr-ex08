# Leaf classifier — live camera classification in the browser

**Live page: <https://nesim20.github.io/mlr-ex08/>**

Point your phone's rear camera at a leaf and the four class probabilities update
live. Everything runs on the device: the frames never leave the browser.

A single `index.html` with plain JavaScript. `leaf_model.onnx` is loaded with
onnxruntime-web from a CDN; there is no build step and no server-side code.

## The model

| | |
|---|---|
| file | `leaf_model.onnx` (used byte-for-byte as provided, 6.1 MB) |
| input | `image`, `float32`, shape `1 × 3 × 96 × 96` |
| output | `logits`, `float32`, shape `1 × 4` — **raw logits, not probabilities** |
| classes | `apple`, `cherry`, `chestnut`, `maple` (in `model_info.json` order) |
| `input_size` | 96 |
| `mean` | 0.485, 0.456, 0.406 |
| `std` | 0.229, 0.224, 0.225 |

Preprocessing follows `model_info.json` exactly, in this order:

1. take the **central square** of the frame (`min(width, height)`, centred);
2. resize that square to **96 × 96**;
3. scale RGB pixels to **0–1** (divide by 255);
4. standardise each channel with `mean` / `std`;
5. build a channel-first `Float32Array` of shape `1 × 3 × 96 × 96`.

The output tensor is named `logits` and holds unbounded values (for the check
image: `-5.84, 9.25, -1.92, -4.37`), so **softmax is applied in the page** before
anything is displayed.

## Camera and fallback

* **Rear camera** — `getUserMedia({ video: { facingMode: { ideal: 'environment' } } })`,
  with progressively looser fallbacks if a device refuses those constraints. The
  stream is attached to a `playsinline` `<video>`; a `requestAnimationFrame` loop
  grabs frames (at most one inference in flight, ~11 fps cap) and repaints the bars.
* **Photo upload** — the fallback, for desktop, for a denied camera permission, or
  for a leaf you photographed earlier. Same preprocessing path, same model call.

`getUserMedia` needs a secure context, so the camera only works over HTTPS
(the GitHub Pages address) or `localhost` — not from `file://`. The upload
fallback works everywhere.

## Self-check on the page

The page classifies `check_image.jpg` on load and prints its numbers next to
`check_expected.json`:

| class | page | expected |
|---|---|---|
| apple | 2.804e-7 | 2.804e-7 |
| cherry | 0.999984 | 0.999985 |
| chestnut | 1.412e-5 | 1.412e-5 |
| maple | 1.224e-6 | 1.224e-6 |

Max absolute difference measured in Chrome: **1.25e-7** (tolerance 1e-4), top
class `cherry`. The same image run through Python + `onnxruntime` reproduces the
expected file to 3.5e-11, so the residual is float32 / JPEG-decoder noise, not a
preprocessing difference.

## Notes worth knowing

* **onnxruntime-web is pinned to 1.21.0** on purpose. The model is an opset-20
  export with `ir_version 10`; onnxruntime-web 1.17 and older refuse it (1.16 fails
  with an opaque emscripten abort, and the model is not modified to accommodate
  that). Versions from 1.19 ship only the *threaded* wasm build, which still
  initialises single-threaded when `SharedArrayBuffer` is missing — and it is
  missing on GitHub Pages, because Pages cannot send the COOP/COEP headers
  `SharedArrayBuffer` requires. Verified with `crossOriginIsolated === false`.
* **The QR code is generated in the browser** by a byte-mode QR encoder written in
  this file (Reed–Solomon over GF(256), mask selection by penalty score). No
  external QR service is contacted — the URL never leaves the page. It encodes the
  deployed address above, so scanning it with a phone opens the page.
* iOS Safari needs a tap before a camera will start, hence the start button;
  `playsinline` keeps the stream in the page rather than fullscreen.

## Files

```
index.html          the whole app: UI, preprocessing, inference, QR encoder
leaf_model.onnx     the classifier, unmodified
model_info.json     classes, input size, mean, std (read by the page at runtime)
check_image.jpg     verification image
check_expected.json expected probabilities for that image
```
