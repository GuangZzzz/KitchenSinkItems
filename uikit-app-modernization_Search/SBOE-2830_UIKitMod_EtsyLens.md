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

---

# Visual Search coordinateSpace Modernization — QA Steps

- **Branch:** `EtsyLensViewController+Navigation`
- **PR:** https://github.com/etsy/EtsyApp/pull/36106

**What changed:** `UIScreen.main.coordinateSpace` replaced with `window.windowScene?.screen.coordinateSpace` in four visual search view controllers. The listing zoom transition now uses the window's scene screen rather than the global main screen.

**Where:** Search → camera icon → take or pick a photo → tap a listing from results.

## Steps

1. Visual search on iPhone (single window) — tap a listing from results.
   - [ ] Zoom transition animation lands correctly on the listing detail page.

2. Visual search on iPad in Split View (~50% width) — tap a listing.
   - [ ] Transition animation is scoped to the panel, not the full screen.

3. Tap the anchor card after it settles.
   - [ ] Navigates to the listing detail page.
