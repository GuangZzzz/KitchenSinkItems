# EtsyLens Image Crop — QA Test Plan (SBOE-2830)

- **Branch:** `SBOE-2830/UIKitMod/EtsyLens`
- **PR:** https://github.com/etsy/EtsyApp/pull/35869

**What changed:** EtsyLens now crops camera photos to the visible camera-preview area. Previously the crop was based on the full device screen size.

**Where:** Search → tap the camera / visual-search (EtsyLens) icon → allow camera access.

## Tests

1. **Take a photo** — frame an object, tap the shutter.
   - ✅ The captured/scanned image matches what you saw in the preview.

2. **Pick from photo library** — choose an existing photo instead of shooting.
   - ✅ Keeps its original aspect ratio.

3. **Select a result image** — tap an image so it animates into place.
   - ✅ Lands in the right spot, aligned correctly.

4. **iPad multitasking** (if in scope) — run EtsyLens in Split View / Slide Over and take a photo.
   - ✅ Crop matches the preview area.

5. **Basic regression** — normal flow on iPhone: open EtsyLens → capture → results. Try flash on and off.
   - ✅ Works as before.

## Things to confirm

The captured image is centered and fully framed (not shifted or over-cropped), the image fills the area without stretching or black bars, and the flow completes smoothly.

## Things to beaware

The app will experience crush at certain time in the runloop. Due to older architecture context regarding to concurrency among actor model and AVfoundation. Our team is aware of it. https://etsy.atlassian.net/browse/SBOE-2496
