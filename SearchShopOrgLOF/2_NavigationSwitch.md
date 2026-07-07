# Area 2 — Navigation Switch in Search Component Result View

**Tags:** `@SearchComp` `@LayoutLogic`
**Owner:** Guang

---

## The Dispatch Chain

```
SearchResultsView.resultsComponentsView (ForEach on viewModel.components)   :329
  → SearchComponentType.view (switch — ~65 cases)   SearchComponentType.swift:430
    → BoxComponentView.body (nested ForEach)   BoxComponentView.swift:18
      → SearchComponentType.view (switch — inner dispatch)
        → concrete component view (e.g. SignalsListingCardComponentView)
```

`SearchComponentType.swift` is **auto-generated** (`DO NOT EDIT`). Any change that touches the dispatch switch must go through the Tuist scaffold template — this is a shared concern across all components.

---

## Where Navigation Lives Today

Navigation is wired entirely at the UIKit hosting layer, injected into SwiftUI via `Environment`:

```swift
// SearchResultsViewController+Actions.swift:22–58
view
    .handleSearchNavigateToAction { [weak self] screen in
        self?.handleNavigateToAction(screen)
    }
```

The central switch lives at `SearchResultsViewController+Actions.swift:79–110`:

```swift
private func handleNavigateToAction(_ kind: SearchScreen) {
    switch kind {
    case .shop(let referrer, let isModal): showShop(from: referrer, isModal: isModal)
    case .listing(...):                   showListing(...)
    case .search(...):                    showSearch(...)
    // ...
    }
}
```

Components consume navigation via `@Environment(\.navigateToAction)`.

### Existing shop navigation call sites

Three components already call `.shop(...)` today:

| File | Call | Context |
|------|------|---------|
| `RecommendedShopComponentView.swift:176` | `.shop(viewModel.referrer, isModal: false)` | "Visit shop" button |
| `SignalsListingCardComponentView.swift:236` | `.shop(viewModel.referrer, isModal: true)` | Long-press "Go to shop" |
| `ListingCardVariantFullComponentView.swift:74` | `.shop(viewModel.referrer)` | Shop name tap on full-variant card |

Note: `isModal` is currently dead code in `showShop()` — a comment at line 283 acknowledges this as deferred cleanup from a prior experiment.

---

## Where to Introduce the Navigation Switch

The feature flag variant needs to intercept navigation before the `ForEach` dispatches to existing components. The earliest viable point **without touching the auto-generated file** is:

**Option A — Environment injection at `SearchResultsView` (line 329)**

Wrap the `ForEach` with an additional environment value that signals "shop navigation mode." Components already read from `Environment`; adding one more key is additive and non-breaking.

**Option B — `handleNavigateToAction` in `SearchResultsViewController+Actions.swift` (preferred)**

Add a new `SearchScreen` case or contextual metadata to `SearchReferrer`. This keeps all routing logic in the UIKit coordinator layer — the existing pattern — and requires no changes to the SwiftUI component hierarchy or the auto-generated file.

Option B is preferred: one place to reason about navigation, no SwiftUI component changes required.

---

## Would Moving Logic Lower Cause Delay or Freezing?

No. `SearchScreen` dispatch is synchronous and lightweight. `@Environment` propagation happens at SwiftUI view update time on the main thread. No async work is introduced by adding a navigation switch at any of these layers.
