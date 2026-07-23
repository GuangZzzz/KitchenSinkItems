# UIScreen.main Modernization — Search Scope

| # | File | Line(s) | Pattern | Change |
|---|---|---|---|---|
| 1 | `Features/SearchUI/.../EtsyLensV2ViewModel.swift` | 195 | Pattern 2 · bounds · non-view (ViewModel) | D&F on `capturePhoto()` and `handleGalleryImage(_:)`; add `bounds:` inline to private `handleCapturedImage` |
| 2 | `Features/SearchComponents/.../SearchDiscoveryComponentView.swift` | 15 | Pattern 2 · bounds · SwiftUI | TODO only (GeometryReader adoption changes layout semantics) |
| 3 | `Features/SearchComponents/.../VisualSearchAnchorListingCardComponentView.swift` | 41 | Pattern 2 · bounds · SwiftUI | `@State containerWidth` + `.onGeometryChange` on body |
| 4 | `buyonetsy/Search/Legacy/SearchResultsContainerViewController.swift` | 250, 269, 279 | Pattern 2 · bounds · UIViewController | `UIScreen.main.bounds.width` → `view.bounds.width` |
| 5 | `buyonetsy/Search/Legacy/.../SearchTypeSelectionViewController.swift` | 24 | Pattern 2 · bounds · UIViewController | `UIScreen.main.bounds.width` → `view.bounds.width` |
| 6 | `buyonetsy/Search/Visual Search/VisualSearchViewController+Navigation.swift` | 84 | Pattern 3 · coordinateSpace | Add `windowScene.screen` guard, use `screen.coordinateSpace` |
| 7 | `buyonetsy/Search/Visual Search/EtsyLens/EtsyLensViewController+Navigation.swift` | 30 | Pattern 3 · coordinateSpace | Same |
| 8 | `buyonetsy/Search/Visual Search/EtsyLens/EtsyLensV2ViewController.swift` | 304 | Pattern 3 · coordinateSpace | Same |
| 9 | `buyonetsy/Search/Components/SearchResultsViewController+Actions.swift` | 200 | Pattern 3 · coordinateSpace | Same |
| 10 | `buyonetsy/Search/Visual Search/EtsyLens/EtsyLensV2ResultsViewController.swift` | 369 | Pattern 3 · coordinateSpace | Same |
