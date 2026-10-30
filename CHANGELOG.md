# Changelog

All notable changes to this module are documented here. This project adheres to
[Semantic Versioning](https://semver.org/).

## 2.1.0 - 2026-10-27

### Added
- **OpenStreetMap again**: the pickup map follows the provider chosen in the store locator
  (*Map Settings → Map Provider*: Google Maps or OpenStreetMap). OpenStreetMap needs no key and uses the
  Leaflet bundled in the store locator (no CDN). It had been dropped in 2.0.0.

### Changed
- Requires `threecommerce/m2-storelocator` ^1.2 (`Helper\Data::getMapConfig()` / `isMapAvailable()`).
- With OpenStreetMap the map is shown even without any Google API key.

## 2.0.0 - 2026-10-14

### Changed (breaking)
- Store source is now the **Threecommerce Store Locator** (`threecommerce/m2-storelocator` ^1.1,
  `Threecommerce_Storelocator`) instead of the unpublished `mage-os/module-store-locator`
  (`MageOS_StoreLocator`). Composer requirement, `module.xml` sequence, collections and models all
  switched; no MageOS class is referenced anymore.
- The per-store "pickup enabled" flag moved to the `tc_pickup_enabled` column of
  `3com_storelocator` (declarative schema of this module). Stores must be re-flagged for pickup in
  the new store locator. `quote.tc_pickup_store_id` / `sales_order.tc_pickup_store_id` are now
  `int unsigned` like the store locator id.
- Admin: the store locator form is a tabbed widget form (not a ui_component), so the flag is a new
  **Store Pickup** tab added via layout (`threecommerce_storelocator_storelocator_edit`) and saved by
  an adminhtml observer on `threecommerce_storelocator_save_before`; a **Pickup** column is added to
  the store grid via a plugin. The store locator module is not modified.
- Map: Google Maps only, with the store locator configuration (API key, Map ID → Advanced Markers,
  zoom), via the new shared `js/pickup-map.js`. OpenStreetMap/Leaflet support and the module's own
  CSP whitelist were removed (the store locator already whitelists Google Maps).
- License aligned to `proprietary`; author e-mail `scrivici@threecommerce.it`; hardcoded
  `version` removed from `composer.json`.

### Added
- **Breeze checkout support** (Threecommerce One Step Checkout >= 1.2): list/map store picker in the
  `onestepcheckout.breeze.shipping.extra` container, "Pickup store" line in the summary recap
  (`onestepcheckout.breeze.recap.extra`), vanilla `js/breeze-pickup.js` driven by the checkout DOM
  events `osc:shipping-method-change` and `osc:before-place-order` (blocks the order without a
  store and re-saves the choice right before payment).
- Base stylesheet `css/pickup.css` (neutral, `--osc-*` variables): on Breeze it follows the
  checkout "Use Built-in Styles" option, on Hyvä it is always loaded.
- The success-page "ready for pickup at" box now renders on both Breeze and Hyvä.

### Fixed
- Hyvä: removed the inline `<style>` from the picker template (CSP) and the duplicate `x-init`
  (Alpine 3 already calls `init()`), which registered the listeners twice.
- Saving the chosen store updates only `quote.tc_pickup_store_id` instead of re-saving the whole
  quote, so it no longer overwrites address/shipping data saved concurrently by the checkout.
- The order guard also rejects stores that were disabled or un-flagged after being chosen.
- Breeze: picker, list/map and summary line are hidden also with an inline `display: none`, so a
  theme that sets `display` on `.osc-breeze__recap-block` cannot show them when the built-in
  stylesheet is off.

## 1.0.0 - 2026-06-23

First release. In-store pickup shipping method for Mage-OS / Magento 2, built as an
**add-on to `MageOS_StoreLocator`** (which is never modified — all extension is additive).

### Shipping method
- New `storepickup` shipping carrier ("Store Pickup"), configurable handling fee (free by
  default), under **Stores → Configuration → Sales → Shipping Methods**. When active it
  surfaces as a shipping method at checkout.

### Pickup-enabled stores
- Per-store `tc_pickup_enabled` flag added to the Store Locator table via additive declarative
  schema.
- "Available for Pickup" field merged into the Store Locator admin store **form** and a "Pickup"
  column merged into the store **listing** (via same-name `ui_component` merge — no core edit).

### Checkout (Threecommerce One Step Checkout, Hyvä)
- In-checkout **store picker** shown when the pickup method is selected: choose a store from a
  **list** or a **map**, reusing the Store Locator map engine and the configured map provider
  (Google Maps / OpenStreetMap), API key and zoom.
- Chosen store is persisted on the quote (AJAX) and shown in the **order-summary recap**.
- Integration is decoupled (layout-injected child blocks + generic `getChildHtml` hooks); the
  checkout works unchanged when this module is absent.

### Order, invoice, shipment, e-mail
- Chosen pickup store carried from quote to order (`tc_pickup_store_id`).
- The order's shipping address is set to the pickup store, and the shipping description names
  the store — so the store is visible in the order/invoice/shipment **e-mails** and the orders
  grid with no template overrides.
- "Store Pickup" panel on the admin **order**, **invoice**, **shipment** and **credit memo**
  views.
- "Ready for pickup at …" confirmation on the checkout success page.
- Hard guard: placing a pickup order without a selected store is rejected.