# VisualSearchAnchorListingCard — QA Steps (UIKit Modernization)

- **Branch:** `UIkitMod/UIScreen/SearchDiscoveryComponentView`
- **PR:** https://github.com/etsy/EtsyApp/pull/35964

**change log:**  `centerOffset` is caculated from its container width instead of `UIScreen.main.bounds.width`.


UI flow: anchor card loads at the top of screen with animation slide from left to right and text blur to clear.

Search → camera icon → take or pick a photo -^
Tap in any listing card -> tap find similar icon -^

The area we want to do more testing is resizing while animation, cycle back with design team for this interaction. 

## Steps

1. Visual search on iPhone (single window).
   - [ ] Listing image animates from the horizontal center of the card to its resting position on the left.
   - [ ] Title, price, and rating fade in after the slide.

2. Visual search on iPad.
   - [ ] Image starts centered within the panel.
   - [ ] Animation lands.

3. Tap the anchor card after it settles.
   - [ ] Navigates to the listing detail page.
  
