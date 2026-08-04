# Search Layout Sizing — QA Test Guide (SBOE-2883)

- **Branch:** `SBOE-2883/UIKitModernization/SearchResultsContainerViewController-SearchTypeSelectionViewController`
- **PR:** https://github.com/etsy/EtsyApp/pull/36076

**What changed:** `UIScreen.main.bounds.width` replaced with context-aware window/view width in two search VCs. Affects layout sizing calculations for the search results header and the "Search by" domain switcher sheet.

## SearchResultsContainerViewController

**Where:** Search → type any query → submit → search results screen.

1. **Results header renders correctly** — header stack sits flush above results with no overlap or gap below it.
2. **iPad Split View / Stage Manager** — open search in a split window. Header height should fit the narrower window, not the full screen width.

## SearchTypeSelectionViewController

**Where:** The domain switcher is rarely visible. It appears in the search bar during transitions between search contexts where no back arrow occupies the left slot. When visible, it shows as a small icon (bag = Items, shop = Shops) + chevron left of the text field.

1. **Sheet sizes to content** — tap the domain button → bottom sheet slides up and snaps to exactly the height of its options (Items, Shops). No clipping, no dead space.
2. **iPad Split View / Stage Manager** — same check in a narrower window.
