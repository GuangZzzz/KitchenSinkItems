# Area 3 — Reuse of Shop Component Items

**Tags:** `@SearchComp` `@LayoutLogic` `@APILogic`
**Owner:** Martin

---

## Already Search-Native — Zero Adaptation

### `RecommendedShopComponentView`
`Features/SearchComponents/Sources/Components/RecommendedShop/RecommendedShopComponentView.swift:14`

Already a first-class `SearchComponentType` case (`.recommendedShop`, type string `"recommended_shop"`). Fully wired into the search component decoder and `SearchLayoutView`.

**Renders:** shop avatar, name, owner, location, star rating, listing card strip, "Visit shop" CTA.

**Sizing:** Two modes controlled by `RecommendedShopComponentModel.layout`:
- `.scroll` — standalone card, full-minus-padding wide
- `.wrap` — carousel peek card at 75% width

**Input model:** `RecommendedShopComponentModel` — decoded directly from the search API response. Fields: `shop_id`, `shop_name`, `shop_average_rating`, `shop_total_rating_count`, `deepLink`, `icon.url`, `shop_location`, `shop_owner`, `display_listings: [SearchComponentType]`.

**Adaptation needed:** None from the client. The backend just needs to include `"recommended_shop"` in the search response payload for the relevant endpoint.

---

## Best for a Compact List-Style Shop Row — Low Adaptation

### `ShopAutosuggestItemComponentView`
`Features/SearchComponents/Sources/Components/ShopAutosuggestItem/ShopAutosuggestItemComponentView.swift:11`

Already lives in `SearchComponents`, already conforms to `SearchComponentView` / `SearchComponentViewModel` / `SearchComponentModel`. Currently appears only in the autosuggest overlay.

**Renders:** 30pt thumbnail, shop name, star-seller icon, rating, owner name, follow/purchased badge.

**Sizing:** Intrinsic height list row, full-width.

**Input model:** `ShopAutosuggestItemComponentViewModel` — fields: `name`, `owner`, `imageUrl`, `url`, `shopAverageRating`, `shopRatingCount`, `isFavorited`, `isPreviouslyPurchased`, `isStarSeller`. Maps cleanly to any shop search result.

**Adaptation needed:** Add it as a standalone `SearchComponentType` case for main results (small scaffolding task). Add a tappable container — autosuggest currently handles the tap outside this view.

---

## Reusable Sub-Component

### `ShopReviewInfoView`
`Features/FavoritesUI/Sources/Shared/Views/FavoriteShops/Version2-Q2-2025/ShopReviewInfoView.swift:9`

Generic info row with no strong Favorites coupling beyond a simple availability enum.

**Renders:** optional avatar, shop name, coupon/vacation text, star rating with review count.

**Input model:** `ShopReviewInfoView.Model` — `name`, `numberOfReviews`, `stars`, `avatarURL`, `couponDescription`, `availability`.

**Adaptation needed:** Low — could be extracted to a shared location and used anywhere.

---

## Not Recommended for Search Reuse

| Component | Location | Reason |
|-----------|----------|--------|
| `FavoriteShopWideCardView` | `FavoritesUI/` | 4-image strip, no rating shown, favorites-oriented layout |
| `FavoriteShopGridCardView` | `FavoritesUI/` | Model has favorites-specific fields (`couponDescription`, item count) |
| `DealsUI/RecommendedShopView` | `DealsUI/` | GRT tracking, Deals environment deps, `FollowShopToggle` coupling |
| `DealsUI/ShopCardView` | `DealsUI/` | Fixed 288pt card, sale copy — not suitable without full redesign |
| `ShopTile` (Profile) | `Profile/` | Profile-specific navigation and analytics events |
| `GroupGridView` | `ListingScreenUI/` | Listing grid module, not a shop card |

---

## API / View Model Changes Needed

`RecommendedShopComponentModel` already decodes from the search API with no changes required.

If reusing `ShopAutosuggestItemComponentView` in main results, its model fields (`name`, `owner`, `imageUrl`, `shopAverageRating`, `shopRatingCount`, `isFavorited`, `isStarSeller`) map cleanly to any shop search result — **no new API work** assuming those fields are present in the response payload.
