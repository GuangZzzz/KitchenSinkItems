# VisualSearchAnchorListingCard — QA Steps (UIKit Modernization)

- **Branch:** `UIkitMod/UIScreen/SearchDiscoveryComponentView`
- **PR:** *(pending)*

**What changed:** `centerOffset` now uses the card's measured container width instead of `UIScreen.main.bounds.width`.

**Where:** Search → camera icon → take or pick a photo → anchor card loads at the top of results.

## Steps

1. Visual search on iPhone (single window).
   - [ ] Listing image animates from the horizontal center of the card to its resting position on the left.
   - [ ] Title, price, and rating fade in after the slide.

2. Visual search on iPad in Split View (~50% width).
   - [ ] Image starts centered within the panel.
   - [ ] Animation lands without overshooting or clipping.

3. Tap the anchor card after it settles.
   - [ ] Navigates to the listing detail page.
