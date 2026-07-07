# Area 4 — Navigation Stack Behavior

**Tags:** `@SearchComp` `@APILogic`
**Owner:** Guang

---

## Search → Shop: How It Routes Today

```
SignalsListingCardComponentView / ListingCardVariantFullComponentView
  → @Environment(\.navigateToAction)(.shop(...))
    → SearchResultsViewController+Actions.swift:283  showShop(from:isModal:)
      → traitCollection.navigation.navigate(to: NavigationArea.shop.byShopID(...))
        → ShopNavigation.swift — pushes ShopHomeViewController on current tab nav stack
```

Back-stack is **preserved** on this path — standard push onto the current tab's nav controller.

---

## Listing → Shop: Different Code Path

```
EtsyListingViewController+Routing.swift:42
  → EtsyRouter.pushViewController(wrapper: BOERouteWrapper.shop(...), from: self)
    → BOERoute.shop.routableViewController
      → pushes ShopHomeViewController on the calling VC's nav controller
```

Uses the legacy `EtsyRouter` coordinator, not the modern `EtsyNavigation` system. Back-stack is also preserved (push on the calling VC's nav stack).

### Key differences

| Dimension | Search → Shop | Listing → Shop |
|-----------|---------------|----------------|
| System | EtsyNavigation (modern) | EtsyRouter (legacy coordinator) |
| File | `SearchResultsViewController+Actions.swift:283` | `EtsyListingViewController+Routing.swift:42` |
| `isModal` handling | Dead code — always push | `EtsyRouter.presentViewController` for interstitial autosuggest (modal) |
| Referrer set by | `ShopNavigation` handler | `EtsyRouter.setReferer(on:from:)` |
| About page routing | n/a from search | Uses modern `NavigationArea.shop.aboutByShopID` (home still uses EtsyRouter — inconsistency) |

---

## Back-Stack Loss — Confirmed Risk Scenarios

### 1. New search replaces old search VC
`SearchNavigation.swift:14–28` — `updateSearchNavigationStack()`

Removes the previous `SearchResultsViewController` from the nav stack before pushing a new one. If a user navigated Search → Shop → back → Search Results and then triggers a new search, the old results VC is removed from the stack array.

### 2. Bottom search bar `popToRoot`
`SearchNavigation.swift:213–215`

```swift
self.switchTo(tab: .search, dismissPresented: true)
self.activeViewController?.navigationController?.popToRootViewController(animated: false)
```

Switches tabs and **explicitly destroys** any navigation history on the search tab. A previously pushed Shop Home would be lost.

### 3. Deeplink to shop dismisses modals
`EtsyScreenController+URLHandling.m:750`

```objc
[self.controller dismissViewControllerAnimated:YES completion:nil];
```

Fires before the shop push — any modally-presented flow (e.g. cart) is dismissed before navigating to Shop.

---

## `isModal` Dead Code

`SearchResultsViewController+Actions.swift:283–288` contains an explicit comment:

> "Previous iterations used `isModal` for experiments within modals. To limit the impact of this immediate change, we will defer cleanup of all `isModal` areas to a follow up PR."

The long-press "Go to shop" in search results passes `isModal: true` from `SignalsListingCardComponentView`, but `showShop()` ignores it and always pushes. This may or may not match the intended behavior for the shop navigation variant — worth aligning with design intent before building on top of it.

---

## Does Deeplink Navigation Preserve Back Stack?

Deeplink-triggered shop navigation (`DeeplinkMigration.handle` → `EtsyURLHandler` → `EtsyScreenController.displayShop`) does **not pop to root** — it pushes onto the active tab's current stack. However:

- It first calls `dismissViewControllerAnimated:YES`, dismissing any active modal
- It does not restore the search results stack if the user was in a different context when the deeplink fired

---

## Open Question

When navigating Search → Shop → back, should triggering a new search preserve the shop in the back stack or reset it? The current `updateSearchNavigationStack` implementation always clears the old search VC regardless. This needs a product decision before the navigation routing is finalized.
