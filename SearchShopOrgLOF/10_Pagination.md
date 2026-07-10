# Area 5 — Pagination

**Tags:** `@SearchComp` `@APILogic`
**Status:** Pending API team decision on endpoint strategy

---

## Existing Pagination Call Stack

```
ScrollView scroll (80% threshold)
  → PaginationModifier.onScrollGeometryChange  SearchResultsView.swift:574
      visibleRect.maxY > contentSize.height - containerSize.height * 0.8
  → .pagination callback closure              SearchResultsView.swift:170
      guard !isPaginating && viewState != .loading && footerError == nil
      viewModel.isPaginating = true
      DispatchQueue.main.async { paginationCallback() }
  → SearchResultsViewController.paginate()   SearchResultsViewController.swift:381
      paginationDebouncer.perform(with: true)
  → Debouncer closure after 0.1s             SearchResultsViewController.swift:111
      guard resource.loadState == .loaded
      resource.load()
  → EtsyResource.load()                      EtsyResource.m:309
      updateOffset()  →  offset += lastResults.count
                      or  nextPath cursor (MMX path)
      NSURLSession HTTP GET
  → HTTP response parsed
      results accumulated: results += newResults
      nextPath updated for next cursor
      dispatch to main queue
  → Success callback                         SearchResultsViewController.swift:251
      viewModel.isPaginating = false
      viewModel.canLoadMore = resource.canLoadMore()
      viewModel.append(components: new)
  → SearchResultsViewModel.append()          SearchResultsViewModel.swift:190
      self.components.append(contentsOf: components)
      @Observable invalidation → SwiftUI re-render
  → SearchResultsView ForEach re-renders     SearchResultsView.swift:338
      new cards appended to grid
```

### Pagination Footer States

| State | UI shown |
|-------|----------|
| `footerError == .retryable` | Error view with Retry button |
| `footerError == .terminal` | Permanent error, no retry |
| `footerError == nil && canLoadMore == true` | Skeleton rows or spinner |
| `canLoadMore == false` | Nothing (pagination exhausted) |

### Two Offset Strategies in EtsyResource

| Strategy | Trigger | Mechanism |
|----------|---------|-----------|
| Legacy offset | Non-MMX path | `offset += lastResults.count` per page |
| MMX cursor | `respectNextPathForLoadMore = true` | `nextPath` URL from response header/envelope |

---

## What Is Reusable

**`PaginationModifier` / `.pagination()` extension — fully portable.**

Zero search-specific imports. Can be attached to any SwiftUI `ScrollView`:

```swift
scrollView.pagination(callback: { viewModel.loadMore() })
```

**Everything below it is not portable.** `EtsyResource` (ObjC), `SearchResultsViewController`, and `SearchResultsManager` are tightly coupled to `SearchComponentType`, `EtsySearchWithAds`, and the search filter/sort state. Extracting them for a shop module costs more than building fresh.

**Modern Swift alternative already exists:** `ShopHomeSearchViewModel` + `ShopHomeSearchRepository` in `Features/ShopHomeUI/Sources/`. Uses `async/await`, `Task` cancellation, `offset + feedPaginationKey` cursor, and `canLoadMore = listings.count < totalCount`. This is the correct template for any new paginated module.

---

## Client Effort Comparison

### Scenario A — Shop module embedded in existing search stream

The server includes shop components inline in the search result stream. The existing `EtsyResource` → `viewModel.append(components:)` pipeline handles all pagination automatically.

**Pagination client effort: zero.**

Shop components arriving on any page get appended like any other `SearchComponentType`. No new pagination code required.

| Item | Effort |
|------|--------|
| New component (Model, ViewModel, View, Component bridge) | ~1 day |
| Register new type string in SearchComponentType (Tuist scaffold) | ~2 hours |
| Pagination | Zero |
| **Total** | **~1 day** |

**Constraints:**
- Server controls when/where shops appear in the stream — client cannot request more shops independently
- If design requires shops at fixed intervals, that is a server-side concern

---

### Scenario B — Dedicated shop search endpoint

A separate API call returns only shop results. The shop module manages its own pagination independently of the search listing stream.

**Pagination client effort: ~1.5 days.**

Copy-adapt `ShopHomeSearchViewModel` + `ShopHomeSearchRepository` as the template.

| Item | Effort |
|------|--------|
| `ShopSearchRepository` protocol + live impl | ~0.5 day |
| `ShopSearchViewModel` (offset, canLoadMore, loadMore()) | ~0.5 day |
| Wire `PaginationModifier` scroll trigger | ~1 hour |
| Coordinate two paginators on same screen | Medium complexity — see below |
| New component UI | ~1 day |
| **Total** | **~2.5 days** |

**Coordination problem:** If the shop module lives inside the search results `ScrollView`, one scroll view drives two independent paginators. The 80% threshold fires against the full search list height, not the shop section. A separate trigger (`.onAppear` on the last visible shop card) is needed to fire shop pagination independently.

**Additional risks:**
- Two in-flight network requests on the same screen — need independent loading and error states
- What if the shop request fails but the search request succeeds? Requires explicit error handling per paginator

---

## Side-by-Side Summary

| | Scenario A (inline in search stream) | Scenario B (dedicated endpoint) |
|---|---|---|
| Pagination client work | Zero | ~1.5 days |
| Total client work | ~1 day | ~2.5 days |
| Independent shop load-more | Not possible | Yes |
| Coordination risk | None | Medium |
| Fits existing architecture | Perfectly | Requires new layer |
| Server flexibility required | Server controls position | Client controls layout |

---

## Open Question

**Does the shop result set need to be independently paginated?**

- If no — shops appear at server-controlled positions in the search stream → **Scenario A**, no pagination work on client
- If yes — a dedicated "Shops" section that loads more shops as the user scrolls, decoupled from listing results → **Scenario B**, new pagination stack required

This is the question to resolve with the API team before scoping pagination work.

---

## Update — 2026-07-09

**Decision: Scenario A confirmed.**

API team confirmed shop results will be delivered through the existing search stream.

Two additional constraints clarified by product:

- **Shops-only results** — the response contains only shop components on this surface, no mixed listing cards. The `SearchComponentType` `ForEach` dispatch will only encounter shop-type components.
- **Sufficient result volume** — enough shops exist in the result set that exhausting pagination is not a concern.

With these in place the previously noted risks are resolved. Server-controlled ordering is acceptable because there are no mixed component types to interleave with. Client pagination work is **zero** — the existing stack handles everything unchanged.
