# Recipes

Every write below is: dry run, show the plan to the person, explicit yes, then the same call with `apply: true`. "Read" means the matching `get_*` tool, which gives the `token` / `version` to send back.

## Contents
1. Add a new piece to a collection
2. Prepare and release a collection
3. New in: put, remove, reorder
4. Change a price
5. Photos: order, main image, hover, alt text, role
6. Unpublish a sold piece
7. Why is a piece not on the site?
8. A difference or conflict between the store and the dashboard

## 1. Add a new piece to a collection

1. `get_schema` (categories, options, material families, rules).
2. Find the collection: `list_collections` with the designer or collection name. No collection? Create it first (recipe 2); no designer? `save_designer` first. No designer, no collection; no collection, no piece.
3. `create_piece` with `collectionId`, `fields` (title, description, `categoryId`, `priceEur` as typed on the screen, e.g. "4200", excl. VAT, dimensions in cm, weight in kg) and `options` (each option with its values; every combination becomes a variant). The piece is created in **Draft**: nothing goes online.
4. Missing material: `search_materials`, then `create_material` if really absent.
5. Variants: `set_variant` (SKU, stock, own price, materials...) or `add_variant` for one more combination.
6. Photos: adding images through Claude is not available yet (`add_piece_images` says so). Ask the person to drop them on the piece's **Media** in the dashboard, per variant. Then recipe 5 for order, alt and role.
7. When ready and the person has the right: `set_visibility` to `visible`. It is refused, with the reason, if the collection or designer is not published or something is missing (often a photo or a price). No right to publish: `request_publication`.
8. `get_sync_status` on the piece about a minute later.

Vestige piece from a dealer: `create_piece` on the line THE_VESTIGE with the `vintage` block (designer credited, dealer as supplier chosen from the dealers list, condition, provenance), unique piece (stock 1). Its price rule is still open (D63, D71): leave the price to Partnership or an Admin.

## 2. Prepare and release a collection

- **Prepare** (nothing visible): `save_collection` without `collectionId` (name, description, `designerId`, `location` = made in, `type`, SEO) and a **far** `launchDay` (YYYY-MM-DD, Paris; release is at 17:30). Add the pieces (recipe 1) and publish them; they stay invisible until the release date. A collection needs a main image to be published: ask for it on the collection's page in the dashboard.
- **Release**: `get_collection`, then `save_collection` with its `version` and the **real** `launchDay`, `published: true`. On that date the collection and its published pieces appear by themselves, and its pieces enter New in for 8 weeks.
- The dry run says how many pieces are resent to the store (`piecesResent`, 100 at most per change). Over 20, have the person review the list before applying.
- Never create the collection in Shopify: the store's collection is made by the sync.

## 3. New in: put, remove, reorder

- Rule: New in = pieces added by hand, or the pieces of the latest collection releases for 8 weeks, dated by the **collection release date**, never by the day a piece was published. A piece of an old collection published today does not enter New in by itself.
- Put a published piece in: `get_piece`, then `set_new_in` with `on: true`.
- Take it out: `set_new_in` with `on: false`.
- Reorder: no connector tool. The order is set on the dashboard screen **Catalogue › New in**, which also adds and removes. Say so.
- Never touch the New in collection, its tag or its order in Shopify.

## 4. Change a price

1. `get_piece`: read the current EUR price (excl. VAT) of the piece and of each variant (variants can carry their own price), and its USD/GBP shown for information.
2. `set_price` with `priceEur` (piece, or one variant with `variantId`). Dry run first; show old → new EUR and the USD/GBP that will follow.
3. If the price moves by more than 30 %, the tool asks for `confirmPriceJumps: true`: ask the person to confirm the jump explicitly, often it is a typo (4200 vs 42000).
4. USD and GBP follow by themselves (the published exchange rate, rounded) and reach the store and HubSpot in about a minute. Never type them and never edit any price in Shopify.
5. Vestige: the EUR price on the store is the store's own (D63). Send a Vestige price change to an Admin.
6. More than 20 pieces at once (a price list, a rise): one dry run per piece is fine but the person reviews the whole list first; consider asking an Admin.

## 5. Photos: order, main image, hover, alt text, role

- Photos belong to a **variant**. `get_piece` lists each variant's images with their ids.
- **Order and main image**: `order_variant_images` with every image id of that variant, the first one is the main image.
- **Alt text**: `describe_image` (300 characters max; `everywhere: true` for every image of the piece showing the same file).
- **Product or ambiance**: `set_image_role` with `PRODUCT` or `AMBIANCE`.
- **Hover image** (second photo on the cards): no connector tool; it is chosen on the piece's Media in the dashboard. A per-variant hover sent to the store is being built: today the store's hover may not match what the dashboard shows. Say so briefly if asked.
- **Add** photos: on the piece's Media in the dashboard (not yet through Claude).
- **Remove**: `delete_image` dry run, then `confirm: true` only after the person says yes. Refused when the image is used elsewhere or would leave a published piece without a photo.
- Never upload, reorder or delete product media in Shopify: that is what made 73 Vestige photo sets drift in November.

## 6. Unpublish a sold piece

1. `search_pieces` (title, SKU), `get_piece`.
2. `set_visibility` with `to: hidden` (dry run). It says how many pieces leave the store (`leavingStore`).
3. After yes: same call with `apply: true` and `leaving` = that number.
4. If the team prefers the piece to stay visible as sold, the alternative is `set_variant` with `soldOut: true`; ask which they want.
5. Never archive or delete the product in Shopify.

## 7. Why is a piece not on the site?

1. `search_pieces`, then `explain_store_state` on the piece. It says: its status, whether its collection or designer holds it back, what it lacks to be published, its place in the sync queue, the last error, open differences.
2. Explain in one or two sentences, then offer the fix with the matching recipe (publish the collection, add a photo, set a price, wait for the release date...).
3. MS Editions and The Vestige are internal brands: never say "the designer is not published" for them.
4. A failed send: `resend_piece` (dry run, then apply) puts it back in the queue; nothing is forced from here.
5. Use the Shopify connector only to look at how the page renders, never to "fix" the product there.

## 8. A difference or conflict between the store and the dashboard

1. `list_drifts` (for one piece or all) or `get_sync_status`.
2. A change made in Shopify on a field the dashboard did not touch is usually taken back into the dashboard by itself. A product created in Shopify is adopted as a draft. A deletion in Shopify is only reported.
3. A line that is not a conflict (a change taken from Shopify, a product deleted or created there): after the person has looked, `resolve_conflict` with a short `reason` marks it reviewed.
4. A **conflict** (both sides changed the same field: which value to keep) is a human choice made on **Catalogue › Shopify sync** in the dashboard, by someone with the right to publish. The connector cannot choose; show both values if you have them and send the person there.
5. Then fix the value in the dashboard if needed, never by rewriting the product in Shopify.
