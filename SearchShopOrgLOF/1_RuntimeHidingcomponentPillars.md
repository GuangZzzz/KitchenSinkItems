# Area 1 — Runtime Hiding: Pilters

**Tags:** `@SearchComp` `@LayoutLogic`
**Owner:** Guang & Martin

---

## Terminology

The codebase term is **pilter** (pill + filter), not "pillar." Used consistently across all Swift sources, Obj-C headers, and Collage color tokens (`clg_color_pilter_border`, `clg_color_pilter_border_selected`).

---

## What Pilters Are

Pilters are `RefinementPillComponentModel` instances rendered as `CollageSelectableChip` views in a horizontally scrollable row above the listing grid, hosted in `SearchResultsHeaderV2View`.

Data flow:

```
API response → SearchResultsViewController:274
  → SearchResultsManager.pilterComponents: [SearchFilterComponentType]   (@Observable)
    → SearchResultsHeaderV2ViewModel.pilterComponents (passthrough)
      → SearchResultsHeaderV2View — `if !viewModel.pilterComponents.isEmpty` guard
```

---

## Current Hiding Mechanism — 100% Server-Driven

There is no client-side control over pilter visibility today. The view branch is:

```swift
// SearchResultsHeaderV2View.swift:36
if !viewModel.pilterComponents.isEmpty {
    ScrollView(.horizontal, showsIndicators: false) {
        HStack(spacing: .clgDimensionPalGrid100) {
            ForEach(viewModel.pilterComponents) { component in
                component.view
            }
        }
    }
}
```

Server sends `pilter_components: []` → row collapses instantly, no animation.

---

## Flash / Layout Jump — Confirmed High Risk

When the API response arrives and `pilterComponents` transitions from `[]` → non-empty:

- The `ScrollView` block renders suddenly
- The sort row and listing grid **snap downward**
- No `withAnimation {}` or `.transition()` wraps this branch

The sticky overlay path (`SearchResultsView.swift:188`) does have an animated transition:

```swift
.transition(.offset(x: 0, y: -20).combined(with: .opacity))
```

But that only applies to the floating overlay triggered on scroll-up — **not** to the initial inline appearance of the pilter row.

**Secondary risk:** `piltersOffset` is measured asynchronously via `GeometryReader` and starts at `0.0`. There is a brief window after first render where scroll logic can misfire based on a stale offset value, potentially causing a double-render moment.

---

## Do We Need a New API Endpoint?

| Option | Mechanism | API change? |
|--------|-----------|-------------|
| **Server-side** | Server returns `pilter_components: []` for shop search variant | Yes — new query param or response variant so server knows to omit pilters |
| **Client-side** | Filter in `SearchResultsManager` or `SearchResultsHeaderV2ViewModel` behind a feature flag | No — data is fetched but discarded on the client |

**Recommended path:**
- **Short-term:** client-side filter in `SearchResultsHeaderV2ViewModel.pilterComponents` gated on the feature flag — minimal blast radius, no API dependency
- **Long-term:** server omits pilters via the existing empty array or a new field, avoiding unnecessary data transfer

---

## Fixing the Layout Jump

Two changes — one at the data layer, one at the view layer:

```swift
// 1. Wherever pilterComponents is written in SearchResultsManager
withAnimation(.easeOut(duration: 0.2)) {
    self.pilterComponents = incoming
}

// 2. SearchResultsHeaderV2View.swift:36 — add a transition to the branch
if !viewModel.pilterComponents.isEmpty {
    ScrollView(.horizontal, showsIndicators: false) { ... }
        .transition(.move(edge: .top).combined(with: .opacity))
}
```

This gives the inline pilter row the same treatment the sticky overlay already has.

---

## Open Question

Is the hide condition tied to a specific tab within search results (e.g. a "Shops" tab), or does the entire search results screen suppress pilters when the shop navigation variant is active?

This determines whether VM-level filtering is sufficient or whether the server needs to be aware of the variant context.
