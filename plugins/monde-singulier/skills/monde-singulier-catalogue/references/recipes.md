# Recipes

Every write below is: dry run, show the plan to the person, explicit yes, then the same call with `apply: true`. "Read" means the matching `get_*` tool, which gives the `token` / `version` to send back.

## Contents
1. Add a new piece (and publish it)
2. Prepare and release a collection
3. New in: put, remove, reorder
4. Change a price
5. Photos: add, order, main image, hover, alt text, role
6. Options and finishes: rename, add, remove, order
7. French texts: check and correct
8. Unpublish a sold piece
9. Why is a piece not on the site?
10. An extra collection page on the store
11. A difference or conflict between the store and the dashboard

## 1. Add a new piece (and publish it)

1. `get_schema`: categories, option names (`mainOptions`: Size, Base, Upholstery, Material, Dome, Dimmer, Side, Finish, Colour today; any other name such as "Wood" is refused), material families, channels, image rules.
2. Find the collection: `list_collections`. No collection? Create it first (recipe 2); no designer? `save_designer` first.
3. **Vestige only, the dealer first**: `search_dealers`. Found: keep its `id`. Not found: `create_dealer` dry run; show the lookalikes it returns; only when the person says none is the dealer, apply with `notADuplicate: true` (address = pickup address, platform). Needs the right to publish; otherwise Partnership creates it.
4. `create_piece` with `collectionId`, `fields` (English title and description, `categoryId`, `priceEur` excl. VAT as typed, e.g. "4200", dimensions in cm, weight in kg) and `options` (each with its values: 2 options of 3 values make 9 variants). Born in **Draft**. Leave `fields.channels` out to get the screen's defaults (channels the store does not hold are ignored at the send). Vestige: `fields.vintage` with `supplierId`; never ask for pickup address, margin scheme or SKU, they come by themselves.
5. Materials: `search_materials`, `create_material` if really absent, then `set_variant` per variant (SKU, stock, own price, materials...).
6. Photos: recipe 5. A piece needs an image to be published.
7. French: a few minutes later, recipe 7 if the person wants to check it.
8. **Publish**: `publish_pieces` with the piece id(s), no `apply`: the summary (price, images, what blocks, what is only missing) and a confirmation code. Fix what blocks, show the summary, ask once for the batch, then `apply: true` with the code (valid 15 minutes, refused if a piece was saved meanwhile: read the summary again). Without the right to publish: `request_publication`.
9. `get_sync_status` a minute later; if the piece is not on its page, recipe 9.

## 2. Prepare and release a collection

- **Prepare** (nothing visible): `save_collection` without `collectionId` (name, description, `designerId`, `location` = made in, `type`, SEO, image) and a **far** `launchDay` (YYYY-MM-DD, Paris; release at 17:30). Add and publish the pieces (recipe 1): they stay invisible until the date. A collection needs a main image to be published.
- **Release**: `get_collection`, then `save_collection` with its `version`, the **real** `launchDay`, `published: true`. On that date the collection and its published pieces appear, and its pieces enter New in for 8 weeks.
- The dry run says how many pieces are resent (`piecesResent`, 100 at most). Over 20, have the person review the list before applying.
- Never create the collection in Shopify: the store's collection is made by the sync.

## 3. New in: put, remove, reorder

- Rule: pieces added by hand, or the pieces of the latest collection releases for 8 weeks, dated by the **collection release date**. A piece of an old collection published today does not enter by itself.
- Put a published piece in: `set_new_in` with `on: true` (optional `position`, 1 = first). Move it: same call with a new `position`. Take it out: `on: false`.
- Never touch the New in collection, its tag or its order in Shopify.

## 4. Change a price

1. `get_piece`: current EUR price excl. VAT of the piece and of each variant (variants can carry their own).
2. `set_price` with `priceEur` (piece, or one variant with `variantId`). Show old → new EUR and the USD/GBP that will follow.
3. Over 30 % the tool asks for `confirmPriceJumps: true`: ask the person explicitly, often it is a typo (4200 vs 42000).
4. USD and GBP follow by themselves and reach the store and HubSpot in about a minute. Never type them, never edit a price in Shopify.
5. Vestige price: to an Admin. More than 20 pieces at once: the person reviews the whole list first.

## 5. Photos: add, order, main image, hover, alt text, role

- Photos belong to a **variant**; `get_piece` lists each variant's images with their ids.
- **Add** with `add_piece_images` (per variant). A photo reaches the dashboard in only two ways:
  - `{url}`: a public HTTPS address of the image file itself (a CDN, a shared link that downloads the file, no login). Not a web page that shows it.
  - `{upload}`: an upload id, from `upload_images` (a drop box shown in the chat) or, with a terminal (Claude Code, Cowork), `prepare_image_uploads` then a PUT of each file within five minutes. An upload id works once, within the hour.
  - A file attached to the conversation or pasted in the chat cannot be read by any tool: ask for the drop box or for addresses.
  - JPG, PNG or WEBP, 20 MB max, 600 × 600 px min; the server converts and reduces them. At most 5 photos over 3 MB per call (send the rest in another call), 20 images per call and per variant.
  - Give an `alt` per image; `position: 1` makes the first one the main image.
  - A replayed call with the same address is harmless: the answer lists it under `alreadyPresent`.
- **Order and main image**: `order_variant_images` with every image id of the variant, first = main image (store card).
- **Hover**: `set_hover_image` with one of the variant's own images (never the first); `imageId: null` goes back to the second image.
- **Alt text** later: `describe_image` (300 characters; `everywhere: true` for every image with the same file). **Role**: `set_image_role` `PRODUCT` or `AMBIANCE`.
- **Remove**: `delete_image` dry run, then `confirm: true` after a yes.
- Never upload, reorder or delete product media in Shopify.

## 6. Options and finishes: rename, add, remove, order

- Read the piece (`get_piece`: options, values, variants).
- **Rename** an option or a value: `edit_options` with `rename_option` (e.g. Material → Finish) or `rename_value` (e.g. "Oak" → "Natural oak"). A label change only: variants, prices and photos stay; the store's variant titles follow. Several operations go in one call, in order; after a `rename_option` the later operations use the NEW name. The French name is retranslated unless written by hand.
- **Add** an option: `edit_options` `add_option` with its first value (existing variants take it). A colour swatch: `set_colour` on a value of the option named Colour (the store shows no swatch). One more combination: `add_variant`.
- **Remove** a value: `remove_option_value`; the dry run lists the variants archived (any history, photos included) or deleted; show it, then `confirm: true`. Remove an option: first its other values, then `remove_option` (never the last option).
- **Order** the values: `order_option_values`. It sets the order in the dashboard only; **the store does not follow it yet**. Say so to the person.

## 7. French texts: check and correct

1. `get_french_texts` with `kind` (`"product"` for a piece, `"designer"`, `"collection"`) and the id: each text with its key, English, French, status (automatic, hand-written, missing). `translation` says whether one is waiting or failed.
2. A wrong French: `set_french_texts` with the key and the French to keep; dry run shows English beside French. Written texts become hand-written: the machine no longer overwrites them.
3. A French SEO title needs the English one first: write it with `update_piece` (`fields.seoTitle`) / `save_collection` / `save_designer`, wait for the translation, then correct.
4. The title of a piece is not translated (no key). Use only keys `get_french_texts` gave; never invent one.
5. To let the machine translate a text again: `hand_back_french_texts` (overwrites what was typed).
6. Wrong English: correct the English (it is the source); the French follows unless hand-written.

## 8. Unpublish a sold piece

1. `search_pieces`, `get_piece`.
2. `set_visibility` with `to: hidden` (dry run): it says how many pieces leave the store. After a yes, apply with `leaving` = that number.
3. If the team prefers it visible as sold: `set_variant` with `soldOut: true` (or stock 0). Ask which.
4. Never archive, delete or change the product in Shopify (except the emergency rule, `escalate.md`).

## 9. Why is a piece not on the site?

1. `search_pieces`, then `explain_store_state`: status, collection or designer holding it back, channels, what it lacks, queue, last error, differences.
2. Explain in one or two sentences, then the fix: publish the collection, add a photo, set a price, wait for the release date, put it on its channels (`update_piece`, `fields.channels`, codes from `get_schema`)...
3. MS Editions and The Vestige are internal brands: never "the designer is not published" for them.
4. A failed send: `resend_piece`. Nothing is forced from here.
5. The Shopify connector may look at the page, never "fix" the product.

## 10. An extra collection page on the store

A page that is not a designer, a collection of the dashboard or The Vestige (a theme, an edit, a gift selection) can be made in Shopify: the dashboard does not manage such pages. In this order:

1. `list_store_collections` (unmanaged ones by default): what already exists, and whether any shows `orphanReservedTag: true`.
2. Choose a **free tag**: no comma, no space at the ends, and none reserved. Reserved (refused, the dashboard computes and rewrites them on every send): prefixes `cat:`, `type:`, `col:`, `designer:`, `year:`, `decade:`, `vx-d-`, `vx-o-`, `vx-m-`, `vx-cat-`, `vx-dec-`, `vx-fam-`; exact tags `created-for-ms`, `in-stock`, `unique-piece`, `limited-edition`, `new-in`, `showroom-rivoli`, `ms-editions`, `the vestige`, `vx`, `vx-1stdibs`, and `trade`. Example: `walnut-edit-2026`.
3. In Shopify (Shopify connector or admin): create an **automatic** collection whose rule is "product tag equals" that free tag, with its title, text and image. Never a manual collection, never a rule on a reserved tag.
4. Put the tag on the pieces with the **dashboard**: `get_piece`, then `update_piece` with `fields.extraTags` = the piece's current free tags plus the new one (the list sent replaces them). Dry run, yes, apply; one piece per call. Never tag the products in Shopify: the next send would remove the tag.
5. A minute later, check the page with the Shopify connector (collection products) and `list_store_collections`: the new collection must be listed as `source: "unmanaged"` with `orphanReservedTag: false` and the expected product count. If it shows `orphanReservedTag: true`, its rule is on a reserved tag: change the rule to the free tag.
6. To take a piece out, `update_piece` without that tag. Menu or page links to the collection: Shopify connector.

## 11. A difference or conflict between the store and the dashboard

1. `list_drifts` (one piece or all), `get_sync_status`, or `store_drift_report` for the store side (`orphanReservedTagCollections`, `bornInShopify`, `staleOpenAlerts`). Lines counted in `adoptedUnderReview` are not stale: do not `resolve_conflict` them, they protect the images of pieces adopted from the store.
2. A change made in Shopify on a field the dashboard did not touch is usually taken back by itself. A product created in Shopify waits for a person. A deletion in Shopify is only reported.
3. A line that is not a conflict: after the person has looked, `resolve_conflict` with a short `reason`.
4. A **conflict** (both sides changed the same field) is a human choice on **Catalogue › Shopify sync**, by someone with the right to publish. The connector cannot choose; show both values if you have them and send the person there.
5. Then fix the value in the dashboard if needed, never by rewriting the product in Shopify.
